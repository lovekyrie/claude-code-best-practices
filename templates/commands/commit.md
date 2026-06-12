---
description: 按 Conventional Commits 生成提交信息并提交
allowed-tools: Bash(git status), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git commit:*), Bash(git push:*)
argument-hint: [可选：附加说明 / scope 提示]
---

请帮我提交当前改动。附加说明：**${ARGUMENTS:-无}**

## 步骤

1. 运行 `git status` 看清要提交什么
2. 运行 `git diff --staged`（如已 stage）或 `git diff`（未 stage）理解改动
3. 如果改动跨多个独立目的，**告诉我应该拆成几个 commit**，不要硬塞一个
4. 选择合适的 type（`feat` / `fix` / `chore` / `refactor` / `docs` / `test` / `style` / `perf` / `ci`）
5. 如果改动局限于某个模块，加 scope，例如 `feat(auth): ...`
6. 标题 ≤ 72 字符，命令式语态（"add"/"fix" 而不是 "added"/"fixes"）
7. 必要时附 body（空一行后写），说明"为什么"而非"做了什么"

## 提交前检查

- [ ] 没有混入意外文件（`.env`、`.DS_Store`、构建产物）
- [ ] 没有调试日志 / `console.log`
- [ ] 改动符合本次提交主题，没夹带私货

## 执行

确认信息后用 `git commit -m "<title>" -m "<body>"`（如有 body），不要使用 `-am`。

## 禁止

- ❌ 不要 `git add .`，请显式 `git add <file>` 或先与我确认
- ❌ 不要包含 `Co-authored-by: Claude`，除非项目要求
