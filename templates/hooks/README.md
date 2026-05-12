# Hooks 示例

[`example-hooks.json`](example-hooks.json) 演示三类常用 hook，把其中的 `hooks` 字段合并到你项目的 `.claude/settings.json` 即可。

## 1. PostToolUse — 自动 lint 修复

`Edit` / `Write` 工具调用之后，自动跑 `pnpm lint --fix`。

```json
{
  "matcher": "Edit|Write",
  "hooks": [
    { "type": "command", "command": "pnpm lint --fix --quiet || true" }
  ]
}
```

- `|| true` 保证 lint 失败不会阻塞会话
- 按需替换为 `pnpm format`、`biome check --apply`、`prettier --write` 等

## 2. PreToolUse — 拦截危险 bash

在 Bash 工具执行前，用一段 Node 脚本判断是否危险，危险则 `exit 2` 阻止调用并把 stderr 反馈给 Claude。

```json
{
  "matcher": "Bash",
  "hooks": [
    {
      "type": "command",
      "command": "node -e \"const cmd=process.env.CLAUDE_TOOL_INPUT||''; const bad=[/rm\\s+-rf\\s+\\//, /git\\s+push\\s+--force/, /git\\s+push\\s+-f\\b/, /:\\(\\)\\s*\\{/, /sudo\\s+/, /mkfs/, /dd\\s+if=/]; if(bad.some(r=>r.test(cmd))){console.error('Blocked dangerous command: '+cmd); process.exit(2);}\""
    }
  ]
}
```

> 这是兜底防线；同时也应在 `permissions.deny` 里维护黑名单。

## 3. Stop — 会话停止时提醒

```json
{
  "hooks": [
    { "type": "command", "command": "echo '⚠️  Claude 已停止，运行 git status / git diff 后再决定是否提交。'" }
  ]
}
```

## 调试

- 启动时 `claude --debug` 看 hook 触发日志
- 手动跑 hook 命令，确认在你的 shell 下能成功执行
- `matcher` 支持正则字符串（`Edit|Write` 表示匹配 `Edit` 或 `Write` 工具）
