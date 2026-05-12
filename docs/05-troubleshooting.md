# 常见问题与排查

## 1. 上下文爆炸 / 响应变慢

**症状**：会话越来越慢，Claude 开始忘记前面说过的事。

**排查**：

- `/cost` 看 token 用量
- 如果接近模型上限：`/compact` 压缩，或直接 `/clear` 重开

**预防**：

- 每个新任务都 `/clear`
- 大任务用 subagent 外包探索/审查
- 关掉用不到的 MCP（`/mcp` 看清单）

## 2. Claude 似乎没读 CLAUDE.md

**症状**：明明 `CLAUDE.md` 里写了"用 pnpm"，它还是跑 `npm install`。

**排查**：

- `CLAUDE.md` 是否在项目**根目录**？
- 文件名大小写：必须是 `CLAUDE.md`（全大写）
- 是不是从子目录启动 `claude` 而忘了根目录的规则太弱？把关键规则放根目录
- 是不是规则太长被淹没？精简到 < 400 行，关键禁忌放显眼位置

**验证**：在 Claude 里问 `请总结你看到的 CLAUDE.md 内容`，看它是否引用得上。

## 3. 权限报错 / Bash 命令被拒

**症状**：`Permission denied for tool Bash(...)`。

**处理**：

- 临时允许：在弹出的 prompt 选 "Yes"
- 长期允许：编辑 `.claude/settings.json` 的 `permissions.allow`，加 `"Bash(pnpm test:*)"` 这种带通配符的条目
- `/permissions` 交互式管理

> 不要图省事 allow `Bash(*)`。最少 deny 列表必须有 `rm -rf`、`git push --force`、`sudo`。

## 4. Hooks 不触发

**排查清单**：

- `.claude/settings.json` 的 `hooks` 字段拼写是否正确（`PreToolUse` / `PostToolUse` / `Stop` 等）
- `matcher` 是否匹配你期望的工具名（`Bash` / `Edit` / `Write`）
- hook 命令本身能在终端独立跑通吗？
- 启动 Claude 时加 `--debug` 看日志

## 5. MCP 服务器连不上

**排查**：

- `/mcp` 查看状态，红色就是挂了
- 直接在终端跑 MCP 启动命令看错误（如 `npx -y @playwright/mcp@latest`）
- 网络 / 代理 / Node 版本问题
- 重启：先 `/mcp` 看清单，再 `claude mcp restart <name>`

## 6. Claude 在 Plan Mode 里偷偷改了文件

**不应该发生**。如果发生：

- 确认你确实在 Plan Mode（左下角应有标识）
- 升级 Claude Code 到最新版
- 报告 bug

## 7. 子代理结果带不回来

**症状**：调子代理后主会话什么都没拿到。

**原因**：子代理的 prompt 没要求"返回结构化结论"。在子代理 `.md` 里明确：

```markdown
## Output

完成后必须返回一份 markdown 清单：
- 发现的问题（按严重度）
- 推荐改动（含文件路径与行号）
```

## 8. Git 操作被拦 / commit 没带 Co-author

- 检查全局 `settings.json` 的 `includeCoAuthoredBy`
- `git commit` 被拦：在 allow 加 `"Bash(git commit:*)"`
- 不想让 Claude 直接 push：`deny` 里加 `"Bash(git push:*)"`，让人工执行

## 9. 在 monorepo 里 Claude 总跑错包

**解决**：

- 子目录加 `CLAUDE.md` 说明"本目录是 X 包，命令是 `pnpm --filter web ...`"
- 根 `CLAUDE.md` 列出所有包及其作用
- 大任务时直接 `cd packages/web && claude` 在子目录启动

## 10. 我需要回滚 Claude 的改动

- 改动还没确认：`Esc Esc` 回到上一条用户消息
- 改动已写入文件：`git diff` 看，`git checkout -- <file>` 撤销
- 整段会话不要了：直接 `/clear` + `git reset --hard`

> **重要**：Claude 改文件前最好确保 git 工作树干净，方便随时 `git diff` / `git checkout`。
