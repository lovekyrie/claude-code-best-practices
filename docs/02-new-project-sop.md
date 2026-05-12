# 新项目上手 SOP

> 直接回答你的问题：**是的，拿到新项目第一步通常就是建 `CLAUDE.md`** —— 推荐先用 `/init` 生成草稿再人工补全。但这只是第 1 步，下面是完整的 8 步标准动作。

## TL;DR — 新项目第一天清单

```text
[ ] 1. 进入项目根目录，运行 claude，执行 /init           （让 Claude 自动扫描代码生成 CLAUDE.md）
[ ] 2. 人工审阅 CLAUDE.md，补齐：技术栈 / 命令 / 风格 / 禁忌 / 业务术语
[ ] 3. 创建 .claude/settings.json，配置权限白名单与 hooks
[ ] 4. 按需添加 .claude/commands/ 项目专属 slash 命令
[ ] 5. 按需添加 .claude/agents/   子代理（如严格 reviewer）
[ ] 6. 配置 MCP（Playwright / 数据库只读 / Figma 等）
[ ] 7. 用 Plan Mode 跑一次 "请阅读代码并总结架构"，验证 Claude 真的理解项目
[ ] 8. 把 CLAUDE.md + .claude/（脱敏后）纳入 git，团队共享
```

---

## 第 1 步：`/init` 生成 CLAUDE.md 草稿

```bash
cd path/to/new-project
claude
> /init
```

Claude 会扫描仓库结构、`package.json`、`README` 等，生成一份 `CLAUDE.md` 草稿。

**为什么先用 `/init` 而不是手写**：

- 自动识别真实存在的脚本和目录，避免你写错命令名
- 给你一个起点，比从零开始快得多

**草稿一定要人工审阅**：自动生成的内容经常过于啰嗦或不够准确。

## 第 2 步：补全 CLAUDE.md

参考 [`templates/CLAUDE.md`](../templates/CLAUDE.md)，至少包含 6 块：

1. **项目概述**：一句话说清这是什么、给谁用
2. **技术栈**：框架、语言版本、包管理器（`pnpm` / `npm` / `yarn`）
3. **常用命令**：`pnpm dev`、`pnpm test`、`pnpm lint`、`pnpm build`、部署
4. **目录结构**：哪些目录放什么、哪些目录别动
5. **代码风格**：命名约定、提交规范、is否使用 TS strict、ESLint 规则要点
6. **禁忌（最重要）**：不要直接 push 到 main、不要改 `dist/`、不要删测试、不要在没有跑测试时 commit

详细写作指南见 [`04-claude-md-guide.md`](04-claude-md-guide.md)。

## 第 3 步：`.claude/settings.json` 权限与 hooks

拷贝 [`templates/settings.json`](../templates/settings.json) 作为起点：

```bash
mkdir -p .claude
cp ../calude-code-best-practices/templates/settings.json .claude/settings.json
```

关键配置：

- `permissions.allow`：白名单常用安全命令（`pnpm *`、`git status`、`git diff`、`Read`）
- `permissions.deny`：黑名单危险命令（`rm -rf /`、`git push --force`、`pnpm publish`）
- `hooks`：自动 lint、阻止危险 bash（见 [`templates/hooks/example-hooks.json`](../templates/hooks/example-hooks.json)）

> 不要把含密钥的 env 提交到 git。把 `.claude/settings.local.json` 加到 `.gitignore`，团队共享只放 `.claude/settings.json`。

## 第 4 步：项目专属 slash 命令

项目里高频重复的操作就做成命令。例如：

- `/deploy` — 部署到预览环境
- `/migrate` — 跑数据库迁移
- `/seed` — 重置开发库
- `/storybook` — 启动 Storybook 并截图

模板见 [`templates/commands/`](../templates/commands/)，复制到 `.claude/commands/<name>.md` 即可使用。

## 第 5 步：子代理（subagents）

子代理 = 用独立上下文执行特定子任务，不污染主会话。常见：

- `code-reviewer` — 严格审查 PR
- `test-runner` — 跑测试并修复
- `doc-writer` — 维护 README/CHANGELOG

模板见 [`templates/agents/`](../templates/agents/)。

## 第 6 步：MCP 服务器

按需启用，不要全开（启动慢且占上下文）：

- **Playwright MCP** — 让 Claude 操作真实浏览器，截图驱动开发
- **Database MCP（只读）** — 让 Claude 查 schema 和样本数据
- **Figma MCP** — 从设计稿生成代码

```bash
# 项目级
claude mcp add playwright -- npx -y @playwright/mcp@latest
```

## 第 7 步：Plan Mode 验证

进入 Claude，按 `Shift+Tab` 切到 Plan Mode，提问：

> 请阅读这个仓库，输出：1) 架构概览 2) 关键模块职责 3) 你认为 CLAUDE.md 还缺什么

如果 Claude 的总结明显错位，说明 `CLAUDE.md` 还需要补充。

## 第 8 步：纳入 git

```bash
git add CLAUDE.md .claude/settings.json .claude/commands .claude/agents .claude/hooks
git commit -m "chore: add Claude Code config"
```

`.gitignore` 建议加：

```gitignore
.claude/settings.local.json
.claude/.session/
```

---

## 常见误区

- ❌ **直接复制别人的 CLAUDE.md 不改** — 命令名/目录对不上反而误导 Claude
- ❌ **CLAUDE.md 写成长篇大论** — 越长越容易被忽略；保持精炼，按需拆到子目录的 `CLAUDE.md`
- ❌ **不配 hooks 直接 allow `Bash(*)`** — 危险；至少 deny `rm -rf` 和 `--force`
- ❌ **第一天就装一堆 MCP** — 启动慢、上下文吃紧；用到再加
