# 全局配置与工作流

适用于"所有项目"的一次性配置，集中放在 `~/.claude/`。

## `~/.claude/` 目录结构

```
~/.claude/
├── CLAUDE.md              全局规则（中文回复、提交规范、通用编码偏好等）
├── settings.json          全局权限、env、模型选择
├── commands/              全局 slash commands（所有项目可用）
│   └── *.md
├── agents/                全局子代理
│   └── *.md
└── projects/              Claude 自动写入的会话历史（不要手动改）
```

> 项目级配置放在仓库的 `.claude/`，会与全局配置**合并**：项目级 > 全局级。

## 全局 CLAUDE.md 该写什么

只写**真正适用于所有项目**的规则。例如：

- 默认中文回答（除非项目要求英文）
- 提交规范：Conventional Commits，禁止 `git commit -am` 的偷懒模式
- 不擅自删除/跳过测试
- 写代码前先 Plan Mode 确认理解，再实施
- 默认使用 `pnpm`（如果你统一用 pnpm）
- 注释保持原状，除非要求修改

模板见 [`../templates/global-CLAUDE.md`](../templates/global-CLAUDE.md)。

> ⚠️ 不要把项目特定的目录、命令、业务术语写进全局 CLAUDE.md，会污染所有项目。

## 全局 settings.json

模板见 [`../templates/settings.global.json`](../templates/settings.global.json)，包含：

- `permissions.allow`：所有项目都安全的命令（`Read`、`git status`、`git diff`、`git log`、`pnpm install`、`pnpm test*`、`pnpm lint*`）
- `permissions.deny`：危险命令统一拦（`rm -rf /`、`git push --force*`、`:(){`、`sudo *`）
- `env`：默认环境变量（含模型路由，见下节 MiniMax 接入）
- `includeCoAuthoredBy`：是否在 commit 里加 `Co-authored-by: Claude`

## 使用 MiniMax-M2.7（国内环境必读）

由于国内访问 Anthropic 官方 API 受限，本仓库默认把模板配成走 **MiniMax 开放平台**（国内版 `minimaxi.com`），调用 Claude Code 时实际使用的是 `MiniMax-M2.7` 模型。

### 1. 获取 API Key

去 [MiniMax 开放平台 - 充值与 Token](https://platform.minimaxi.com/user-center/payment/token-plan) 申请并复制 API Key。

### 2. 配置

[`../templates/settings.global.json`](../templates/settings.global.json) 中的 `env` 已经写好默认值，**只需把 `ANTHROPIC_AUTH_TOKEN` 替换为你的真实 Key**：

```json
"env": {
  "ANTHROPIC_BASE_URL": "https://api.minimaxi.com/anthropic",
  "ANTHROPIC_AUTH_TOKEN": "REPLACE_WITH_YOUR_MINIMAX_API_KEY",
  "API_TIMEOUT_MS": "3000000",
  "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": "1",
  "ANTHROPIC_MODEL": "MiniMax-M2.7",
  "ANTHROPIC_DEFAULT_SONNET_MODEL": "MiniMax-M2.7",
  "ANTHROPIC_DEFAULT_OPUS_MODEL": "MiniMax-M2.7",
  "ANTHROPIC_DEFAULT_HAIKU_MODEL": "MiniMax-M2.7"
}
```

> 国际版（`minimax.io`）对应 BASE_URL 为 `https://api.minimax.io/anthropic`，按需替换。

### 3. 清理旧的 Anthropic 环境变量（关键）

shell 环境变量优先级**高于** `settings.json`，必须先清掉，否则配置不生效：

```bash
unset ANTHROPIC_AUTH_TOKEN
unset ANTHROPIC_BASE_URL
# 如果 ~/.zshrc 或 ~/.bashrc 里有 export，请同步删除
```

### 4. 跳过 onboarding

新增 `~/.claude.json`（如已存在则编辑），加：

```json
{ "hasCompletedOnboarding": true }
```

### 5. 验证

```bash
claude
```

进入后：

- `/status` 应显示 `ANTHROPIC_BASE_URL` 指向 `api.minimaxi.com/anthropic`
- `/model` 应显示 `MiniMax-M2.7`

### 安全提醒

⚠️ **不要把 `ANTHROPIC_AUTH_TOKEN` 真实值提交到 git**。推荐做法二选一：

- **方案 A（推荐）**：把 token 放到 `~/.zshrc` 里 `export ANTHROPIC_AUTH_TOKEN=...`，删除 `settings.json` 里这一行；shell env 优先级更高
- **方案 B**：保留在 `~/.claude/settings.json`（这个文件本就是本机私有，不会进任何仓库）；但**不要**把它拷到团队共享仓库

本仓库 `templates/settings.global.json` 里写的是 `REPLACE_WITH_YOUR_MINIMAX_API_KEY` 占位符，安全可提交。

## 全局 slash commands

把"任何项目都用得上"的命令放全局，例如：

- `/plan` — 强制进入 Plan Mode 思考
- `/review` — 通用代码审查
- `/commit` — 按 Conventional Commits 生成提交信息
- `/test` — 找测试命令、运行、修复失败用例
- `/refactor` — 安全重构（先列计划再动手）

模板见 [`../templates/commands/`](../templates/commands/)。

## 全局 subagents

适合放全局的子代理：

- `code-reviewer` — 项目无关的严格审查
- `test-runner` — 检测测试框架并执行

项目特定的 agent（如"审查 Vue 组件 a11y"）放项目级 `.claude/agents/`。

## MCP：全局还是项目？

| 场景 | 建议 |
|------|------|
| 你所有项目都用 Playwright | 全局装 |
| 只在某个项目用某数据库 | 项目级装 |
| Figma / Notion 等账号绑定型 | 全局装 |

```bash
# 全局
claude mcp add --scope user playwright -- npx -y @playwright/mcp@latest
# 项目级（默认）
claude mcp add db -- npx -y @some/db-mcp
```

## 终端集成与速查

- `claude` — 启动
- `Shift+Tab` — 切换 Default / Auto-Accept / Plan Mode
- `Esc` — 打断
- `Esc Esc` — 回滚到上一条用户消息
- `/clear` — 清空当前上下文（开始新任务时必用）
- `/compact` — 压缩当前上下文（保留摘要）
- `/config` — 查看/修改配置
- `/permissions` — 管理权限
- `/cost` — 查看本会话开销
- `/mcp` — 查看 MCP 状态
- `#` 开头的消息 — 写入 CLAUDE.md（让 Claude 记住）

## 推荐的工作流姿势

1. **新任务必 `/clear`**：避免上一个任务的上下文污染
2. **复杂任务先 Plan Mode**：让 Claude 列计划再实施
3. **大重构用 git worktree + 多个 Claude 实例**：并行而不打架
4. **让 Claude 自检**：写完代码后让它跑 lint/test 再回报
5. **用 `#` 沉淀经验**：发现 Claude 反复犯同一个错，用 `#` 让它写进 CLAUDE.md
