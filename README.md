# Claude Code Best Practices

一份"拿来即用"的 Claude Code 学习与配置仓库：覆盖全局配置、新项目上手 SOP、高效使用技巧，并提供可直接拷贝到任何 TypeScript / 前端项目的模板。

> 本仓库不会自动修改你的 `~/.claude/` 配置；`templates/` 下的文件由你按需拷贝到全局或项目里。

## 目录

```
.
├── docs/                              学习笔记 & SOP
│   ├── 01-global-setup.md             全局配置与工作流
│   ├── 02-new-project-sop.md          新项目上手 SOP（强烈建议先看）
│   ├── 03-efficient-usage.md          高效使用技巧
│   ├── 04-claude-md-guide.md          CLAUDE.md 写作指南
│   ├── 05-troubleshooting.md          常见问题与排查
│   └── 06-github-credentials.md       GitHub 推送凭证一次性配置
└── templates/                         可拷贝模板
    ├── CLAUDE.md                      项目级 CLAUDE.md（前端 / TS 倾向）
    ├── global-CLAUDE.md               全局 ~/.claude/CLAUDE.md
    ├── settings.json                  项目级 .claude/settings.json
    ├── settings.global.json           全局 ~/.claude/settings.json
    ├── commands/                      自定义 slash commands
    ├── agents/                        子代理（subagents）
    └── hooks/                         hooks 示例
```

## 快速开始

### 1. 应用全局配置（一次性）

```bash
mkdir -p ~/.claude
cp templates/global-CLAUDE.md   ~/.claude/CLAUDE.md
cp templates/settings.global.json ~/.claude/settings.json
# 全局 slash commands & agents（可选）
cp -R templates/commands ~/.claude/commands
cp -R templates/agents   ~/.claude/agents
```

> **国内环境必看**：模板默认通过 **MiniMax 开放平台**（国内版 `minimaxi.com`）调用 `MiniMax-M2.7` 模型，而不是 Anthropic 官方 API。拷贝后需把 `~/.claude/settings.json` 里的 `ANTHROPIC_AUTH_TOKEN` 替换成你的真实 Key，并新建 `~/.claude.json` 加 `{ "hasCompletedOnboarding": true }`。详细步骤见 [`docs/01-global-setup.md` 的 MiniMax 接入小节](docs/01-global-setup.md#使用-minimax-m27国内环境必读)。

### 2. 新项目第一天的标准动作

1. 在项目根目录运行 `claude` 进入交互，执行 `/init` 让 Claude 生成 `CLAUDE.md` 草稿
2. 用 `templates/CLAUDE.md` 对照补全（技术栈、常用命令、风格、禁忌）
3. 拷贝项目级配置：

   ```bash
   mkdir -p .claude
   cp ../calude-code-best-practices/templates/settings.json .claude/settings.json
   cp -R ../calude-code-best-practices/templates/commands .claude/commands
   cp -R ../calude-code-best-practices/templates/agents   .claude/agents
   cp -R ../calude-code-best-practices/templates/hooks    .claude/hooks
   ```

4. 把 `CLAUDE.md` 与 `.claude/`（除本地敏感项外）纳入 git，团队共享

完整 SOP 见 [`docs/02-new-project-sop.md`](docs/02-new-project-sop.md)。

## 学习路径推荐

1. **先读** [`docs/02-new-project-sop.md`](docs/02-new-project-sop.md)：解决"新项目第一步做什么"
2. **再读** [`docs/04-claude-md-guide.md`](docs/04-claude-md-guide.md)：把 `CLAUDE.md` 写好
3. **再读** [`docs/01-global-setup.md`](docs/01-global-setup.md)：建立你专属的全局配置
4. **进阶** [`docs/03-efficient-usage.md`](docs/03-efficient-usage.md)：Plan Mode、subagents、并行
5. **遇到问题** [`docs/05-troubleshooting.md`](docs/05-troubleshooting.md)