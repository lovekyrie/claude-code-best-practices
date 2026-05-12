# 高效使用技巧

让 Claude Code 真正"省时间"的实战技巧。按使用频率排序。

## 1. Plan Mode：复杂任务必先用

`Shift+Tab` 切到 **Plan Mode**，Claude **只读不写**，专注列计划。

适用：

- 改动跨多个文件 / 多个模块
- 不确定改动范围
- 需要先理解陌生代码

姿势：

```
[Plan Mode]
> 我想给 X 功能加权限校验，请先阅读相关代码并给出实施计划
```

满意后切回 Default Mode 让它执行；不满意就在 Plan Mode 里继续讨论，**避免把上下文浪费在错误实现上**。

## 2. `/clear` 与 `/compact`：上下文管理

- **`/clear`**：彻底清空。**每个新任务都应该 `/clear`**，否则旧任务的代码、错误、tokens 会拖慢和误导。
- **`/compact`**：压缩当前上下文为摘要。任务长但还没结束、不想丢失记忆时用。
- **新开终端**：偶尔上下文已经被严重污染，直接退出重开比 `/compact` 更干净。

经验法则：会话超过 ~30 轮 / 上下文 > 60% 就该考虑 compact 或 clear。

## 3. Subagents：把脏活外包

主会话保持干净，把"探索性 / 重复性 / 大量读文件"的活交给子代理：

- 让 `code-reviewer` 跑完整审查 → 只把审查结论带回主会话
- 让 `test-runner` 在独立上下文里反复试错跑测试
- 让 "搜索代理" 翻遍仓库找用法 → 只回传相关文件清单

子代理的上下文**独立**，不会吃主会话的 token。

模板见 [`../templates/agents/`](../templates/agents/)。

## 4. 并行：git worktree + 多 Claude

同一个仓库开多个 worktree，每个 worktree 一个 Claude 终端，做不同任务：

```bash
git worktree add ../proj-feat-a feat/a
git worktree add ../proj-fix-b  fix/b
# 终端 1: cd ../proj-feat-a && claude
# 终端 2: cd ../proj-fix-b  && claude
```

避免单个 Claude 在大仓库里"什么都做"，token 爆炸还容易串味。

## 5. 截图与图像驱动开发

直接把截图拖进 Claude 终端（或 `Cmd+V`）：

- "这个按钮位置不对，照截图调整 CSS"
- "Storybook 报错截图，看看哪里挂了"
- "Figma 截图，照这个实现组件"

配合 Playwright MCP 可以闭环："改完后自己截图给我看"。

## 6. 让 Claude 自检自修

代码写完别立刻收工，追加一句：

> 请运行 `pnpm lint` 和 `pnpm test`，把失败的修了再回报

或者在 hooks 里配 PostToolUse 自动跑 lint（见 [`../templates/hooks/example-hooks.json`](../templates/hooks/example-hooks.json)）。

## 7. Token / 成本优化

- **`/clear` 是最便宜的优化**：80% 的 token 浪费来自上下文堆积
- **大文件别整文件读**：让 Claude 用 grep / 行范围读
- **少贴大段代码到聊天**：让它自己 Read，比你贴上去更省
- **关掉用不到的 MCP**：MCP 服务器的 schema 全程占上下文
- **`/cost` 经常看一眼**：发现某次会话突然飙升就停下来分析

## 8. 让 Claude 记住经验：`#` 命令

发现 Claude 反复犯同一个错（比如老想跑 `npm` 而不是 `pnpm`），输入：

```
# 本项目统一使用 pnpm，不要使用 npm 或 yarn
```

Claude 会问你写到全局还是项目 `CLAUDE.md`，选好即可。**这是把"调教"沉淀成长期记忆的最低成本方式**。

## 9. 危险操作的防线

- 不要 allow `Bash(*)`，至少加上：

  ```json
  "deny": ["Bash(rm -rf *)", "Bash(git push --force*)", "Bash(sudo *)"]
  ```

- 重大破坏性操作（删库、改 main 分支）走 hooks PreToolUse 拦截
- 用 git worktree 隔离实验，不在主分支上让 Claude 乱试

## 10. Plan → Code → Review 三段式

复杂任务推荐分 3 步、每步 `/clear` 一次或换子代理：

1. **Plan Mode** 出计划，人工 review 计划
2. **Default Mode** 实施
3. **`code-reviewer` subagent** 独立审查 diff

每段上下文独立、目标单一，质量明显提升。
