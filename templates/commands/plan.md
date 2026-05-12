---
description: 强制以"只读 + 列计划"的方式分析任务，不修改任何文件
allowed-tools: Read, Grep, Glob, Bash(git status), Bash(git diff:*), Bash(git log:*), Bash(ls:*), Bash(rg:*)
argument-hint: [任务描述]
---

请进入"规划模式"完成以下任务：**$ARGUMENTS**

## 工作要求

1. **只读不写**：本次禁止使用任何会修改文件、改变 git 状态、或运行带副作用的命令
2. 先快速浏览相关代码（用 Read / Grep / Glob），定位涉及的文件
3. 列出**实施计划**，包含：
   - 需要修改 / 新增的文件清单（含路径）
   - 每个文件的核心改动点（用 1~3 句话说清）
   - 风险点 / 不确定的地方
   - 验证方式（要跑哪些测试 / 命令）
4. 如果发现任务描述不清晰或存在歧义，**先列出问题**让我回答，再出计划

## 输出格式

```markdown
## 计划

### 涉及文件
- `path/a.ts` — 干什么
- `path/b.vue` — 干什么

### 步骤
1. ...
2. ...

### 风险 / 待确认
- ...

### 验证
- `pnpm typecheck`
- `pnpm test xxx`
```
