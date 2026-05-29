# 提示词最佳实践（Prompting Guide）

> 本文基于 [Claude Code 官方最佳实践](https://code.claude.com/docs/zh-CN/best-practices)，把"怎么写提示词"这件事单独拎出来讲。每条规则都给出**之前 → 之后**的示范，对照着抄即可显著提升 Claude 的输出质量。

核心约束只有一条：**Claude 的 context window 填得越快，性能掉得越快**。所有提示词技巧的本质都是「在有限上下文里，让 Claude 一次就把活干对」。

## 目录

1. [给 Claude 一种验证其工作的方式](#1-给-claude-一种验证其工作的方式)
2. [先探索，再规划，最后编码（四段式提示）](#2-先探索再规划最后编码四段式提示)
3. [在提示中提供具体的上下文](#3-在提示中提供具体的上下文)
4. [提供丰富的内容：@、截图、URL、管道](#4-提供丰富的内容截图url管道)
5. [让 Claude 采访你（大型功能用 AskUserQuestion）](#5-让-claude-采访你大型功能用-askuserquestion)
6. [紧密反馈循环：Esc / 撤销 / rewind](#6-紧密反馈循环esc--撤销--rewind)
7. [用 subagents 做调查与验证](#7-用-subagents-做调查与验证)
8. [Writer / Reviewer 双会话模式](#8-writer--reviewer-双会话模式)
9. [跨文件扇出（批量改）](#9-跨文件扇出批量改)
10. [常见反模式与修复](#10-常见反模式与修复)

---

## 1. 给 Claude 一种验证其工作的方式

> 「这是你能做的最高杠杆的事情」—— 官网原话

Claude 能跑测试、看截图、比对输出时，质量会陡升。否则它就会给你一个"看起来对、其实跑不通"的东西。

| 场景 | ❌ 之前 | ✅ 之后 |
| --- | --- | --- |
| 指定测试用例 | "实现一个验证邮箱的函数" | "写一个 `validateEmail` 函数。示例测试：`user@example.com` 返回 true、`invalid` 返回 false、`user@.com` 返回 false。实现后跑测试，全绿再回报" |
| UI 视觉验证 | "把仪表盘做得好看点" | "[拖入截图] 照这个设计实现。完成后用 Playwright MCP 截图，与原设计对比，列出差异并修复" |
| 根因修复 | "构建失败" | "构建失败，错误如下：[粘贴错误]。修复它并验证 build 成功。**解决根因，不要把错误吞掉**" |

**项目里的等价写法**（前端 TS 项目）：

```text
> 实现 src/utils/price.ts 的 formatPrice 函数：
> - 输入 number，输出带千分位的字符串（含两位小数）
> - 0 → "0.00"，1234.5 → "1,234.50"，负数前加 "-"
> 写完后运行 pnpm test src/utils/price.test.ts，失败就修到全过
```

> 经验：**没法验证 = 不要发布**。能配 hook 自动跑 lint / test 就配上（见 [`../templates/hooks/`](../templates/hooks/)）。

---

## 2. 先探索，再规划，最后编码（四段式提示）

复杂任务别一句话扔过去，按四段写四条提示，每段切换合适的模式。

### 阶段 1 — 探索（Plan Mode，只读不写）

`Shift+Tab` 切到 Plan Mode：

```text
> 阅读 /src/auth 目录，搞清楚我们的 session 和登录是怎么实现的。
> 顺便看看环境变量里的 secret 是怎么管理的。
> 这一步只读不写，看完告诉我你的理解。
```

### 阶段 2 — 规划

仍在 Plan Mode：

```text
> 我想加 Google OAuth 登录。
> 需要改哪些文件？session 流程怎么走？给我一份详细实施计划。
```

> 满意按 `Ctrl+G` 可在编辑器里直接改计划；不满意继续在 Plan Mode 讨论。**别让计划没敲定就开写**。

### 阶段 3 — 实施（Default Mode）

```text
> 按你刚才的计划实现 OAuth 流程。
> 给 callback handler 写测试，跑 pnpm test，挂掉就修，直到全绿。
```

### 阶段 4 — 提交

```text
> 用一条描述性的 commit message 提交，然后开 PR。
```

> **小任务别套这个流程**：改个错别字、加一行日志、重命名变量 —— 直接让 Claude 干。能用一句话描述 diff 就跳过 Plan Mode。

---

## 3. 在提示中提供具体的上下文

提示越精确，返工越少。Claude 能推断意图，但**读不了你的心**。

| 策略 | ❌ 之前 | ✅ 之后 |
| --- | --- | --- |
| **限定范围** | "为 foo.py 加测试" | "为 `foo.py` 写测试，覆盖用户已注销的边界情况。**不要用 mock**" |
| **指向来源** | "为什么 ExecutionFactory 的 API 这么奇怪？" | "看 `ExecutionFactory` 的 git 历史，总结这个 API 是怎么演化成现在这样的" |
| **参考现有模式** | "加一个日历小组件" | "看主页上现有 widget 怎么实现的（`HotDogWidget.php` 是个好例子）。**照这个模式**实现日历小组件：用户能选月份、左右翻页换年份。**不要引入新依赖**，只用代码库里已有的库" |
| **描述症状** | "修登录 bug" | "用户反馈 session 超时后登录失败。看 `src/auth/` 的认证流程，特别是 token 刷新。**先写一个能重现 bug 的失败测试，再修**" |

**反例 → 正例**（中文项目场景）：

```text
❌  把列表页搞快点
✅  src/pages/order/list.vue 在 1000 条数据时滚动卡顿。
    用 Chrome Performance 录一段火焰图给我分析瓶颈，
    然后改用 vue-virtual-scroller，保持现有 props 不变，
    改完用同样数据跑一次确认 FPS > 50。
```

> 例外：**探索阶段**模糊提示反而有用 —— 比如「这个文件你会怎么改进？」能挖出你想不到的角度。

---

## 4. 提供丰富的内容：@、截图、URL、管道

不要用文字描述代码位置，让 Claude **直接拿到原料**：

| 方式 | 用法 |
| --- | --- |
| `@` 引用文件 | `修改 @src/api/user.ts，给 getUser 加缓存` |
| 拖入 / 粘贴图片 | 直接 `Cmd+V` 截图到终端，"照这个 UI 实现" |
| URL 给文档 | `按 https://xxx/api-doc 的规范实现` （记得 `/permissions` 白名单常用域） |
| 管道喂数据 | `cat error.log \| claude -p "分析这段日志，定位崩溃点"` |
| 让它自己取 | "用 `gh issue view 1234` 拉 issue 详情，再开始动手" |

> 反面教材：把整个文件粘进聊天框。**让 Claude 自己 Read 比你贴更省 token**，还能保证拿到最新版本。

---

## 5. 让 Claude 采访你（大型功能用 AskUserQuestion）

大功能容易"想得不全就开写"，让 Claude 反过来问你：

```text
> 我想做 [一句话描述].
> 用 AskUserQuestion 工具详细采访我：
> 包括技术实现、UI/UX、边界情况、顾虑、取舍。
> 不要问显而易见的问题，挖那些我可能没想到的硬骨头。
> 一直问到覆盖完整为止，然后把完整 spec 写到 SPEC.md。
```

采访完 → **新开一个会话**执行 SPEC.md。新会话上下文干净、目标单一，效果远好于一个会话从头干到尾。

---

## 6. 紧密反馈循环：Esc / 撤销 / rewind

发现 Claude 走偏，**立刻**改正，别等它写完。

| 操作 | 作用 |
| --- | --- |
| `Esc` | 中途打断，context 保留，可继续重定向 |
| `Esc Esc` 或 `/rewind` | 回到之前任意 checkpoint（对话/代码/两者） |
| "撤销那个" | 让 Claude 还原它刚才的改动 |
| `/clear` | 任务无关时彻底清空上下文 |

**规则**：在同一会话里对同一问题改正超过 **两次**，说明 context 已被失败方案污染 —— `/clear`，把你学到的东西写进**新一轮**提示词里重开，几乎一定更快。

```text
❌  长会话 + 不停改正
✅  /clear → 用更具体的提示重开
```

---

## 7. 用 subagents 做调查与验证

Context 是稀缺资源，把"读一堆文件"的脏活外包出去：

```text
> 用 subagent 调查我们的认证系统怎么处理 token 刷新，
> 以及有没有可复用的 OAuth 工具，给我一个摘要清单。
```

实现完用另一个 subagent 复审：

```text
> 用 subagent 审查刚才这段代码的边界情况和并发问题。
```

子代理在**独立的 context** 里跑，回来只给你结论 —— 主会话保持干净。

---

## 8. Writer / Reviewer 双会话模式

新鲜的 context 做 code review 更客观（Claude 不会偏袒自己刚写的代码）：

| 会话 A（Writer） | 会话 B（Reviewer） |
| --- | --- |
| `给我们的 API 加一个速率限制中间件` | |
| | `审查 @src/middleware/rateLimiter.ts：边界情况、竞态、是否符合现有中间件模式` |
| `这是审查反馈：[贴 B 的输出]。逐条修复。` | |

> 同样适用 TDD：A 写测试，B 写实现让测试过。配合 `git worktree` 物理隔离更稳。

---

## 9. 跨文件扇出（批量改）

大规模迁移 / 改造别在一个会话里串行干，用脚本扇出：

```bash
for file in $(cat files.txt); do
  claude -p "把 $file 从 React 改写成 Vue 3 <script setup>。只回 OK 或 FAIL。" \
    --allowedTools "Edit,Bash(git commit *)"
done
```

姿势：

1. 先让 Claude 列出所有要改的文件
2. 在 **2-3 个文件**上试，调好提示
3. 用 `--allowedTools` 严格限定权限（无人值守时尤其重要）
4. 加 `--permission-mode auto` 让分类器在后台拦危险操作
5. 开发期带 `--verbose` 调试，上生产去掉

```bash
claude --permission-mode auto -p "fix all lint errors"
```

---

## 10. 常见反模式与修复

官网原文列了 5 个，照抄并附项目里的解法：

| 反模式 | 表现 | 修复 |
| --- | --- | --- |
| **厨房水槽会话** | 一个会话里跨多个无关任务 | 任务切换前 `/clear` |
| **反复改正** | 同一问题改两次还不对 | 第二次失败立刻 `/clear`，用学到的东西重写提示 |
| **过度膨胀的 CLAUDE.md** | 规则太多被忽略 | 无情修剪，能用 hook 强制的就不写在 CLAUDE.md（见 [04-claude-md-guide.md](04-claude-md-guide.md)） |
| **信任未验证** | 看起来对，跑起来挂 | 永远要求 Claude 提供验证（测试 / 截图 / 命令输出） |
| **无限探索** | "调查一下 X" → 读了 200 个文件 | 加范围限定，或丢给 subagent |

---

## 速查：一句"模板提示词"

复杂任务直接套这个骨架：

```text
> [目标] 我要在 [@文件 / 模块] 做 [一句话描述变更]。
>
> [约束]
> - 沿用现有模式：参考 @path/to/example.ts
> - 不引入新依赖
> - 风格遵循 CLAUDE.md
>
> [验证]
> 完成后跑 pnpm lint && pnpm test，失败修到全过。
>
> [输出]
> 改完用一句话总结改了什么、为什么这么改。
```

> 这一个模板能覆盖 80% 日常任务。剩下 20% 复杂的，回到 [§2 四段式](#2-先探索再规划最后编码四段式提示)。

---

## 延伸阅读

- 官网原文：<https://code.claude.com/docs/zh-CN/best-practices>
- 上下文窗口可视化：<https://code.claude.com/zh-CN/context-window>
- 减少 token 用量：<https://code.claude.com/zh-CN/costs#reduce-token-usage>
- 权限模式：<https://code.claude.com/zh-CN/permission-modes>
