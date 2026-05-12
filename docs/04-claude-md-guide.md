# CLAUDE.md 写作指南

`CLAUDE.md` 是 Claude 在每次会话开始时自动读取的"项目说明书"。写得好 = Claude 立刻像老员工；写得烂 = Claude 像新人乱猜。

## 层级与加载顺序

Claude 会**自上而下**读以下文件并合并（下层覆盖/补充上层）：

1. `~/.claude/CLAUDE.md` — 全局规则
2. 项目根 `CLAUDE.md` — 项目级
3. 当前工作目录到根之间各级 `CLAUDE.md` — 子目录级（适合 monorepo）

在 monorepo 中：根 `CLAUDE.md` 写跨包通用；`packages/web/CLAUDE.md` 写前端特有；`packages/api/CLAUDE.md` 写后端特有。

## 黄金原则

1. **短而准** — 200~400 行最佳，超过 600 行就该拆分或精简
2. **写真实存在的东西** — 命令、目录、脚本必须能跑通；写错的命令比不写更糟
3. **写禁忌比写要求更有效** — Claude 默认会做"合理的事"，你只需告诉它哪里有雷
4. **业务术语必写** — Claude 不知道你公司的"工单 / SKU / 风控"具体指什么
5. **用 `#` 持续迭代** — 每次发现 Claude 犯的错，都让它写进来

## 推荐结构（前端 / TS 项目）

```markdown
# <项目名>

<一句话项目简介>

## 技术栈

- 框架：Vue 3 / Nuxt 3
- 语言：TypeScript（strict）
- 包管理：pnpm@9
- 样式：UnoCSS
- 测试：Vitest + Playwright
- Node：>= 20

## 常用命令

| 命令 | 作用 |
|------|------|
| `pnpm dev` | 启动开发服务器（端口 3000）|
| `pnpm test` | 跑单元测试 |
| `pnpm test:e2e` | 跑 Playwright |
| `pnpm lint` | ESLint + 自动修复 |
| `pnpm typecheck` | vue-tsc 类型检查 |
| `pnpm build` | 生产构建 |

> 提交前必须跑 `pnpm lint && pnpm typecheck && pnpm test`

## 目录结构

- `app/` Nuxt 应用代码
- `app/components/` 复用组件，按业务域分子目录
- `app/composables/` 组合式函数，文件名以 `use` 开头
- `server/` Nitro 服务端代码
- `shared/` 前后端共享类型与工具
- `tests/` Playwright e2e
- `dist/` 构建产物，**不要手动编辑**

## 代码风格

- 单文件组件优先 `<script setup lang="ts">`
- 不写 Options API
- 组件名 PascalCase；composable kebab 文件名 + camelCase 导出
- 优先 VueUse，避免自己造轮子
- 状态管理用 Pinia，禁止全局可变单例

## 提交规范

- Conventional Commits：`feat: ...` / `fix: ...` / `chore: ...` / `refactor: ...`
- 一个 commit 一个目的，避免大杂烩
- PR 标题就是首个 commit 的标题

## 业务术语

- **工单（Ticket）**：客服系统中的客户请求记录
- **SKU**：商品最小销售单位，全大写 `SKU` 不写 `Sku`
- **风控（RiskControl）**：交易反欺诈模块，缩写 `RC`

## 禁忌（重要）

- ❌ 不要 `git push --force` 到 `main` / `develop`
- ❌ 不要直接修改 `dist/`、`node_modules/`、`.nuxt/`
- ❌ 不要删除或跳过测试来让 CI 通过
- ❌ 不要引入新依赖前不告知（先讨论）
- ❌ 不要把密钥写进代码或 `.env`，使用 `.env.local`
- ❌ 不要用 `any` 绕过类型问题（除非加 `// FIXME` 注释）

## 工作流偏好

- 复杂改动先进 Plan Mode 给我看计划
- 大于 3 个文件的改动写完后请自己跑 `pnpm lint` 与 `pnpm typecheck`
- 默认使用中文回复
```

## 反面教材

❌ **太长太空**：写了 1000 行哲学，没一条具体命令
❌ **命令对不上**：写 `npm run test`，但项目实际是 `pnpm test:unit`
❌ **业务术语缺失**：Claude 把"工单"理解成 GitHub Issue
❌ **大量 markdown 装饰**：花哨的 emoji 和分割线吃 token 不办事
❌ **复制别人的不改**：里面还残留着别人的项目名

## 子目录 CLAUDE.md 的用法

monorepo 里，**根 CLAUDE.md 短**，子目录 CLAUDE.md 写细节：

```
CLAUDE.md                    # 5 块通用约定 + 包列表 + 全局命令
packages/web/CLAUDE.md       # 前端：Vue 风格、组件目录、UI 库
packages/api/CLAUDE.md       # 后端：路由约定、数据库 schema、错误码
packages/sdk/CLAUDE.md       # SDK：发布流程、版本策略
```

Claude 进入子目录时会自动加载对应的 `CLAUDE.md`，**只在需要时**消耗那部分上下文。

## 持续迭代的姿势

- 每周 review 一次 `CLAUDE.md`，把过时的删掉
- 用 `#` 即时沉淀新约定
- PR 模板里加一句"如果改动了开发约定，请同步更新 CLAUDE.md"
- 把 `CLAUDE.md` 当成代码看待：会过期，需要维护
