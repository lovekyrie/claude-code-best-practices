---
description: 安全重构：先列计划与风险，再小步实施并保持测试通过
allowed-tools: Read, Grep, Glob, Edit, Bash(pnpm test:*), Bash(pnpm lint:*), Bash(pnpm typecheck), Bash(git status), Bash(git diff:*)
argument-hint: [重构目标，例如：把 useUser 拆成 useUserState + useUserActions]
---

请安全重构：**$ARGUMENTS**

## 阶段 1：理解与计划（不动代码）

1. Read 涉及的文件
2. Grep 找到所有调用点
3. 输出**重构计划**：
   - 改动文件清单
   - 调用点清单（影响面）
   - 拆解为 N 个**小步骤**，每一步都能让测试继续通过
   - 风险点与回滚方式

⏸️ **等我确认计划后再进入阶段 2**

## 阶段 2：小步实施

按计划逐步执行，每步完成后：

1. 运行 `pnpm typecheck`
2. 运行相关测试（不要全跑，只跑受影响的）
3. 报告 diff 摘要，再进入下一步

如果中途发现计划有问题，**停下来汇报**，不要硬改。

## 阶段 3：收尾

1. 跑全套：`pnpm lint && pnpm typecheck && pnpm test`
2. 用 `git diff --stat` 给出改动总览
3. 提示"可以 review 后再 commit"

## 严禁

- ❌ 同时进行重构 + 改行为（功能改动单独一次提交）
- ❌ 跨越多个模块的"大爆炸"重构，一次提交里包不下
- ❌ 重构期间删除/弱化测试
