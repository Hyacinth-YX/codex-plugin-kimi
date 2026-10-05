# Codex Plugin for Kimi Code

在 Kimi Code CLI 中直接使用 OpenAI Codex：做代码评审、对抗性评审，或把调试/实现任务委派给 Codex。

本插件移植自 [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)（Claude Code 版，Apache-2.0）。与原版一样，它通过本机的 **Codex app server**（`codex app-server`，JSON-RPC 协议）深度集成，复用你本地的 Codex 安装、登录态和配置——不是简单的 CLI 包装。

## 功能一览

- `/codex:review` — 对当前改动做只读 Codex 评审
- `/codex:adversarial-review` — 可指定关注点的对抗性评审（质疑设计与取舍）
- `/codex:rescue` — 把任务委派给 Codex（调 bug、做修复、继续之前的 Codex 线程）
- `/codex:status` / `/codex:result` / `/codex:cancel` — 后台任务管理
- `/codex:setup` — 检查安装与登录状态，管理 review gate
- `codex-rescue` 子代理 — 主 Agent 可自动把重型任务分派给 Codex
- 可选的 **review gate**：每次 Kimi 回合结束前自动让 Codex 评审本次改动，发现问题则阻止结束

## 环境要求

- **Node.js 18.18+**
- **Codex CLI** 并已登录（ChatGPT 订阅含 Free 档，或 OpenAI API key）：

```bash
npm install -g @openai/codex
codex login
```

用量会计入你的 Codex 额度。

## 安装

在 Kimi Code 中执行（路径替换为本仓库实际位置）：

```
/plugins install /path/to/codex-plugin-kimi
/reload
/codex:setup
```

也可以直接从 GitHub 安装：`/plugins install https://github.com/Hyacinth-YX/codex-plugin-kimi.git`。

`/codex:setup` 会报告：Node 版本、Codex CLI 版本、登录状态、app server 运行时是否可用。看到 `"ready": true` 即就绪。

## 使用方法

### 评审当前改动

```
/codex:review
/codex:review --base main          # 评审当前分支相对 main 的改动
/codex:review --background         # 后台运行（多文件改动建议后台）
```

评审是只读的，不会改任何文件。

### 对抗性评审（质疑方案本身）

```
/codex:adversarial-review
/codex:adversarial-review --base main 重点质疑这里的缓存和重试设计是否合理
/codex:adversarial-review --background 检查并发竞态，质疑整体方案
```

与 `/codex:review` 的区别：可以在 flags 之后附加自由文本作为评审重点。

### 把任务委派给 Codex

```
/codex:rescue 调查为什么测试突然挂了
/codex:rescue 用最小的安全补丁修复这个失败的测试
/codex:rescue --resume 应用上次运行给出的第一个修复建议
/codex:rescue --model gpt-5.4-mini --effort medium 调查这个 flaky 的集成测试
/codex:rescue --background 调查这个回归问题
```

也可以直接用自然语言，主 Agent 会通过 `codex-rescue` 子代理自动分派：

```
让 Codex 重新设计数据库连接层，提高容错性。
```

说明：

- 不传 `--model` / `--effort` 时由 Codex 自己选默认值；`spark` 会映射为 `gpt-5.3-codex-spark`
- 不带 `--resume` / `--fresh` 时，如果本仓库存在可续接的 Codex 线程，插件会询问你是继续还是新开
- 耗时任务建议 `--background`，之后用 `/codex:status` 和 `/codex:result` 查看

### 后台任务管理

```
/codex:status              # 列出本仓库的运行中/近期任务
/codex:status task-abc123  # 查看单个任务进度
/codex:result              # 查看最近完成任务的结果（含 Codex session ID）
/codex:result task-abc123
/codex:cancel task-abc123  # 取消后台任务
```

拿到 Codex session ID 后，可以在 Codex 里直接打开同一线程继续：

```bash
codex resume <session-id>
```

### Review gate（可选，谨慎开启）

```
/codex:setup --enable-review-gate
/codex:setup --disable-review-gate
```

开启后，插件通过 `Stop` hook 在每次 Kimi 回合结束时，针对本次改动跑一次 Codex 评审；若发现问题会阻止回合结束，让 Kimi 先修复。

> 注意：review gate 可能造成 Kimi/Codex 长时间循环，快速消耗额度，请只在能盯着的会话中开启。另外 Kimi hook 超时上限为 600 秒，单次 gate 评审超过 8 分钟会被判超时放行（fail-open）。

## 与 Claude Code 原版的差异

| 功能 | 状态 | 说明 |
| --- | --- | --- |
| review / adversarial-review / rescue / status / result / cancel / setup | ✅ 完整移植 | 与原版相同的 app-server 集成 |
| review gate（Stop hook） | ✅ 移植 | 阻塞协议改为 Kimi 的 exit code 2 + stderr；单次评审限时 8 分钟 |
| 后台任务 / 会话生命周期 hook | ✅ 移植 | SessionStart / SessionEnd 事件均支持 |
| `/codex:transfer` | ❌ 移除 | 依赖 Claude Code 的会话记录格式（`~/.claude/projects/*.jsonl`），Kimi 会话格式不同，无法直接导入 Codex |
| 按会话隔离任务列表 | ⚠️ 降级 | 原版通过 `CLAUDE_ENV_FILE` 给会话注入环境变量来标记任务归属；Kimi 无此机制，任务不按会话过滤，`/codex:status` 显示本仓库全部任务 |

## Codex 配置

插件复用你现有的 Codex 配置：

- 用户级：`~/.codex/config.toml`
- 项目级：`<项目根>/.codex/config.toml`（需项目被信任）

例如让某个项目固定使用 `gpt-5.4-mini` + `high` 推理强度：

```toml
model = "gpt-5.4-mini"
model_reasoning_effort = "high"
```

更多配置项见 [Codex 文档](https://developers.openai.com/codex/)。

## 目录结构

```
codex-plugin-kimi/
├── kimi.plugin.json          # Kimi 插件 manifest（命令/代理/技能/hooks 声明）
├── commands/                 # 斜杠命令（/codex:review 等）
├── agents/codex-rescue.md    # 委派任务的子代理
├── skills/                   # 运行时约定、GPT-5.4 prompt 指南、结果呈现规范
├── prompts/                  # 评审 prompt 模板
├── schemas/                  # 评审输出 schema
└── scripts/                  # app-server JSON-RPC 客户端与任务运行时（Node.js，与原版一致）
```

## 常见问题

**需要单独的 Codex 账号吗？**
不需要。插件使用本机 Codex CLI 的登录态。已登录则开箱即用；未登录先 `codex login`。

**`/codex:review` 卡住或很慢？**
多文件评审本来就慢，建议 `--background`。后台任务的状态和日志由插件的共享运行时管理，`/codex:status` 可随时查看。

**会话结束后后台任务会怎样？**
`SessionEnd` hook 会清理本会话启动的运行中任务并关闭共享运行时；已完成的任务结果仍保留，可在下次会话用 `/codex:status` / `/codex:result` 查看。

## 致谢与许可

上游项目 [openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc)，Apache-2.0（见 `LICENSE` / `NOTICE`）。本仓库为向 Kimi Code CLI 的移植版。
