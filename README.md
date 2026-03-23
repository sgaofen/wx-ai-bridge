# cli-in-wechat

在微信中运行 Claude Code、Codex CLI、Gemini CLI —— 通过微信 ClawBot 官方插件的 iLink Bot API 实现。

```
╔══════════════════════════════════════╗
║       cli-in-wechat  v0.1.0        ║
║  Claude Code / Codex / Gemini CLI  ║
╚══════════════════════════════════════╝
```

## 它是什么

一个运行在你电脑上的桥接服务。微信是遥控器，电脑是执行端。

```
微信 ClawBot (手机)
    ↕  iLink Bot API — 微信官方消息通道 (不封号)
桥接服务 (你的电脑)
    ↕  child_process.spawn
claude -p / codex exec / gemini -p (本地 CLI)
```

你在微信里发消息，服务调你电脑上的 CLI 工具执行，结果发回微信。

## 功能

- **三大 CLI 工具**: Claude Code、Codex CLI、Gemini CLI，通过 `@` 前缀随时切换
- **最高权限默认开启**: Claude `--dangerously-skip-permissions`、Codex `--yolo`、Gemini `--approval-mode yolo`
- **会话续接**: 连续对话自动通过 `--resume` 保持上下文
- **完整 `/` 命令体系**: 40+ 命令覆盖三个 CLI 的所有核心 flag
- **工具接力**: `>>` 传递上条结果，`@tool1>tool2` 链式调用
- **typing 指示器**: 处理中显示"正在输入"
- **微信官方通道**: 使用 iLink Bot API (ClawBot 插件)，不封号

## 安装

### 前置要求

- **Node.js** >= 18
- **微信** iOS 8.0.70+，已启用 ClawBot 插件 (我 → 设置 → 插件)
- 至少一个 CLI 工具:

```bash
# Claude Code (推荐)
npm install -g @anthropic-ai/claude-code

# Codex CLI
npm install -g @openai/codex

# Gemini CLI
npm install -g @google/gemini-cli
```

### 安装并运行

```bash
git clone https://github.com/sgaofen/cli-in-wechat.git
cd cli-in-wechat
npm install
npm run dev
```

首次运行会在终端显示 QR 码，用微信扫码登录 ClawBot。凭据保存在 `~/.wx-ai-bridge/`，后续自动复用。

### CLI 工具认证

每个工具需要单独登录:

```bash
# Claude Code — 用你的 Anthropic 订阅账号
claude

# Codex — 用 ChatGPT 账号
codex

# Gemini — 设置 API key 或 OAuth
export GEMINI_API_KEY="your-key"
# 或直接运行 gemini 触发浏览器 OAuth
gemini
```

## 使用方法

### 发消息

| 输入 | 行为 |
|---|---|
| 直接打字 | 发给上次使用的工具 (默认 Claude) |
| `@claude 写排序算法` | 用 Claude Code |
| `@codex fix the bug` | 用 Codex CLI |
| `@gemini 解释代码` | 用 Gemini CLI |

切换工具后，后续消息默认发给该工具，不用每次都加 `@`。

### 工具接力

```
@claude 分析这个项目的架构
>> @codex 根据上面的分析，重构代码    ← Claude 的输出作为 Codex 的上下文
>> 继续优化                           ← 继续用 Codex，带上之前的结果
@claude>codex 先分析再修复            ← 链式: Claude分析 → Codex执行
```

### 完整命令列表

#### 设置类

| 命令 | 作用 | 对应 CLI flag |
|---|---|---|
| `/status` | 查看所有当前设置 | — |
| `/model <名>` | 切换模型 (`reset`=默认) | `--model` |
| `/mode <auto\|safe\|plan>` | 权限模式 | Claude: `--permission-mode` / Codex: `--sandbox` / Gemini: `--approval-mode` |
| `/effort <low\|med\|high\|max>` | 思考深度 | Claude: `--effort` |
| `/turns <数字>` | 最大 agent 轮次 | Claude: `--max-turns` |
| `/budget <美元>` | API 预算 (`off`=无限) | Claude: `--max-budget-usd` |
| `/dir <路径>` | 切换工作目录 | `cwd` |
| `/system <提示词>` | 追加系统提示 (`clear`=清除) | Claude: `--append-system-prompt` |
| `/tools <列表>` | 允许的工具 (逗号分隔) | Claude: `--allowedTools` |
| `/notool <列表>` | 禁用的工具 | Claude: `--disallowedTools` |
| `/verbose` | 切换详细输出 | Claude: `--verbose` |
| `/bare` | 切换 bare 模式 (跳过配置加载) | Claude: `--bare` |
| `/adddir <路径>` | 添加额外目录访问 | Claude/Codex: `--add-dir` |
| `/name <名>` | 会话命名 | Claude: `--name` |
| `/sandbox <ro\|write\|full\|off>` | 沙箱级别 | Codex: `--sandbox` |
| `/search` | 切换 web 搜索 | Codex: `--search` |
| `/ephemeral` | 切换临时模式 | Codex: `--ephemeral` |
| `/profile <名>` | 加载配置 profile | Codex: `--profile` |
| `/approval <模式>` | 审批模式 | Gemini: `--approval-mode` |
| `/include <目录>` | 添加上下文目录 | Gemini: `--include-directories` |
| `/ext <名\|none>` | 指定 extensions | Gemini: `-e` |

#### 操作类

| 命令 | 作用 | 实现方式 |
|---|---|---|
| `/diff [说明]` | 查看 git 差异 | 发 prompt 给当前工具 |
| `/commit [说明]` | 创建 git 提交 | 发 prompt 给当前工具 |
| `/review [说明]` | 代码审查 | 发 prompt 给当前工具 |
| `/plan [描述]` | 制定计划 / 切换 plan 模式 | 无参数=切模式，有参数=发 prompt |
| `/init` | 创建项目配置文件 | 发 prompt (CLAUDE.md/AGENTS.md/GEMINI.md) |
| `/files` | 列出目录结构 | 发 prompt 给当前工具 |
| `/compact` | 压缩上下文 | 清除 session (重新开始) |
| `/stats` | 查看上次回复信息 | 本地记录 |

#### 会话类

| 命令 | 作用 |
|---|---|
| `/new` | 新会话 (清除所有工具 session) |
| `/clear` | 清除所有会话和历史 |
| `/cancel` | 取消正在运行的任务 |
| `/fork` | 分支当前会话 |
| `/resume` | 查看保存的会话 ID |

#### 快捷命令

| 命令 | 等效 |
|---|---|
| `/yolo` | `/mode auto` + `/effort max` |
| `/fast` | `/effort low` |
| `/reset` | 重置所有设置为默认 |
| `/cc` | 切到 Claude Code |
| `/cx` | 切到 Codex CLI |
| `/gm` | 切到 Gemini CLI |

#### 终端专属 (微信中不可用)

以下命令仅在本地终端有效，在微信中会提示：

`/vim` `/theme` `/color` `/terminal-setup` `/keybindings` `/chrome` `/ide` `/stickers` `/mobile` `/login` `/logout` `/doctor` `/upgrade` `/exit` `/quit` 等

## 权限模式

`/mode` 命令对应三个工具的不同 flag:

| 模式 | Claude Code | Codex CLI | Gemini CLI |
|---|---|---|---|
| `auto` (默认) | `--dangerously-skip-permissions` | `--yolo` | `--approval-mode yolo` |
| `safe` | 默认权限 | `--full-auto` | `--approval-mode default` |
| `plan` | `--permission-mode plan` | `--sandbox read-only` | `--approval-mode plan` |

## 配置

配置文件: `~/.wx-ai-bridge/config.json`

```jsonc
{
  "defaultTool": "claude",         // 默认工具
  "workDir": "/Users/you",         // CLI 工作目录
  "cliTimeout": 300000,            // 超时 (ms)
  "allowedUsers": [],              // 允许的微信用户 ID (空=不限)
  "tools": {                       // 每个工具的额外 CLI 参数
    "claude": { "args": ["--max-turns", "50"] },
    "codex": { "args": ["--add-dir", "/tmp"] }
  }
}
```

### 数据存储

```
~/.wx-ai-bridge/
├── config.json           # 配置
├── credentials.json      # 微信 iLink 登录凭据
├── poll_cursor.txt       # 消息轮询游标
└── sessions/
    └── sessions.json     # 用户设置 + 会话 ID
```

## 架构

```
src/
├── index.ts              # 入口
├── config.ts             # 配置管理
├── ilink/
│   ├── types.ts          # iLink 协议类型
│   ├── auth.ts           # QR 扫码登录
│   └── client.ts         # 长轮询 + 发消息 + typing
├── adapters/
│   ├── base.ts           # CLIAdapter 接口 + UserSettings
│   ├── claude.ts         # claude -p --output-format json
│   ├── codex.ts          # codex exec --yolo
│   ├── gemini.ts         # gemini -p --approval-mode yolo
│   ├── aider.ts          # aider -m --yes-always
│   └── registry.ts       # 自动检测已安装工具
└── bridge/
    ├── session.ts        # 会话 + 设置持久化
    ├── formatter.ts      # 响应格式化
    └── router.ts         # @ 路由 + / 命令 + >> 接力 + 链式调用
```

### 添加新 CLI 工具

实现 `CLIAdapter` 接口 (~50行), 在 `registry.ts` 中注册:

```typescript
export class MyToolAdapter implements CLIAdapter {
  readonly name = 'mytool';
  readonly displayName = 'My Tool';
  readonly command = 'mytool';
  // ... implement isAvailable() and execute()
}
```

然后 `@mytool` 即可使用。

## 微信 iLink Bot API

本项目使用微信 2026 年 3 月推出的 ClawBot 插件提供的官方 iLink Bot API:

- 域名: `ilinkai.weixin.qq.com` (腾讯官方)
- 认证: QR 扫码 → Bearer token
- 收消息: HTTP 长轮询 (35s)
- 发消息: POST + `context_token`
- **官方通道，不封号**

## License

MIT
