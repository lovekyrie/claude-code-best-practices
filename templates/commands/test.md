---
description: 检测项目测试命令、运行、并定位修复失败用例
allowed-tools: Read, Grep, Glob, Edit, Bash(pnpm test:*), Bash(pnpm run:*), Bash(npm test:*), Bash(npm run:*), Bash(yarn test:*)
argument-hint: [可选：要跑的测试文件 / 用例名]
---

请运行测试并修复失败用例。范围：**${ARGUMENTS:-全部测试}**

## 步骤

1. 读 `package.json`，确认测试命令（`pnpm test` / `pnpm test:unit` / `pnpm test:e2e`）
2. 运行测试
3. 如果全部通过：报告耗时与用例数，结束
4. 如果有失败：
   1. 列出失败用例清单
   2. 对每个失败：先**分析根因**（看测试代码 + 被测代码 + 错误信息）
   3. 区分"代码 bug"还是"测试过时"
   4. 如果是代码 bug：修代码，**不要改测试断言**
   5. 如果是测试过时：说明原因后再改测试
5. 修复后重新运行，确保全部通过
6. 报告最终结果与改动文件清单

## 严禁

- ❌ 用 `.skip` / `.todo` 跳过失败测试
- ❌ 注释掉断言
- ❌ 把期望值改成实际值"让它通过"
- ❌ 不分析就改测试

## 输出

```markdown
## 测试结果
- 总数 / 通过 / 失败 / 跳过：...
- 耗时：...

## 修复
- `path/to/file.ts:42` — <根因> → <如何修>
```
