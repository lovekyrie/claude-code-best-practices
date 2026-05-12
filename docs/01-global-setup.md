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

去 [MiniMax 开放平台 - 充值与 Token](https://platform.minimaxi.com/user-center/payment/token-plan) 申请并复制 API Key（形如 `sk-cp-...`）。

### 2. 把 Key 写进 `~/.zshrc`（一次性，永久生效）

> **核心思路**：Key 只放在 shell 环境变量里，模板文件永远是占位符 / 不含 Key。这样：
> - `cp templates/settings.global.json ~/.claude/settings.json` 之后**无需任何编辑**
> - 重装 / 换电脑 / 同步模板时不会反复填 Key
> - 模板可以安全提交到 git

```bash
cat >> ~/.zshrc <<'EOF'

# === Claude Code via MiniMax ===
export ANTHROPIC_AUTH_TOKEN="sk-cp-你的真实key"
export MINIMAX_API_KEY="sk-cp-你的真实key"   # MCP 用，可与上一行同值
EOF

source ~/.zshrc
```

> bash 用户改 `~/.bashrc`；fish 改 `~/.config/fish/config.fish`（`set -x ANTHROPIC_AUTH_TOKEN ...`）。

### 3. 拷贝模板（开箱即用）

```bash
mkdir -p ~/.claude
cp templates/settings.global.json ~/.claude/settings.json
```

`templates/settings.global.json` 里**没有** `ANTHROPIC_AUTH_TOKEN` 字段，Claude Code 启动时会自动从 shell env 读取。

> 如果你**之前** shell 里 `export` 过 Anthropic 官方的 token，必须先清掉再重设：
> ```bash
> unset ANTHROPIC_AUTH_TOKEN ANTHROPIC_BASE_URL
> # 同时把 ~/.zshrc 里旧的 export 行删除
> ```

### 4. 跳过 onboarding

新建或编辑 `~/.claude.json`，加上：

```json
{ "hasCompletedOnboarding": true }
```

### 5. 装 MiniMax MCP（图片理解 + 网络搜索）

[`../templates/mcp/minimax.json`](../templates/mcp/minimax.json) 模板里也**没有 Key**，靠 `uvx` 子进程继承 shell env。

```bash
# 一键安装到 user scope（所有项目可用）
claude mcp add -s user MiniMax \
  --env MINIMAX_API_HOST=https://api.minimaxi.com \
  -- uvx minimax-coding-plan-mcp -y
```

> 注意命令里也**没传** `MINIMAX_API_KEY`，因为 `~/.zshrc` 里已 export，`uvx` 启动子进程时会自动继承。

### 6. 验证

```bash
claude
```

进入后：

- `/status` 应显示 `ANTHROPIC_BASE_URL` 指向 `api.minimaxi.com/anthropic`
- `/model` 应显示 `MiniMax-M2.7`
- `/mcp` 应看到 `MiniMax` 服务器（绿色），含 `understand_image` / `web_search` 两个工具

### 国际版（如果你用 `minimax.io`）

把 `~/.zshrc` 里 `MINIMAX_API_KEY` 不变，但相应替换 host：

```json
"ANTHROPIC_BASE_URL": "https://api.minimax.io/anthropic"
```

MCP 安装命令的 `MINIMAX_API_HOST` 改为 `https://api.minimax.io`。

### 安全要点

✅ **真实 Key 只在 `~/.zshrc`**（本机私有，不进任何仓库）
✅ **模板里永远不含 Key**（已经做到）
❌ **不要**把 `~/.claude/settings.json` 或 `~/.claude.json` 复制进任何 git 仓库的 `templates/`

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
