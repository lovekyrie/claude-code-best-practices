# <项目名>

<一句话项目简介，例如：面向 B 端客服的工单系统前端>

## 技术栈

- 框架：<Vue 3 / Nuxt 3 / React / Next.js>
- 语言：TypeScript（strict 开启）
- 包管理：pnpm@<版本>
- 样式：<UnoCSS / Tailwind / SCSS>
- 测试：Vitest + Playwright
- Node：>= 20

## 常用命令

| 命令 | 作用 |
|------|------|
| `pnpm install` | 安装依赖 |
| `pnpm dev` | 启动开发服务器 |
| `pnpm test` | 跑单元测试（Vitest） |
| `pnpm test:e2e` | 跑端到端测试（Playwright） |
| `pnpm lint` | ESLint 检查并自动修复 |
| `pnpm typecheck` | 类型检查 |
| `pnpm build` | 生产构建 |

> 提交前必须本地通过 `pnpm lint && pnpm typecheck && pnpm test`

## 目录结构

- `src/` 应用源码
- `src/components/` 复用组件，按业务域分子目录
- `src/composables/` 组合式函数，文件名以 `use` 开头
- `src/pages/` 路由页面
- `src/api/` API 客户端封装
- `src/types/` 共享类型
- `tests/` 端到端测试
- `dist/` 构建产物，**不要手动编辑**

## 代码风格

- 单文件组件优先 `<script setup lang="ts">`
- 不使用 Options API
- 组件名 PascalCase；文件名与组件名一致
- 优先 [VueUse](https://vueuse.org/) / 标准库，避免造轮子
- 状态管理用 Pinia；禁止全局可变单例
- 函数 / 变量 camelCase；常量 UPPER_SNAKE_CASE
- 不使用 `any`，确需绕过加 `// FIXME(<负责人>): <原因>`

## 提交规范

- Conventional Commits：`feat(scope): ...` / `fix(scope): ...` / `refactor` / `chore` / `docs` / `test`
- 一个 commit 一个目的
- PR 标题与首个 commit 一致

## 业务术语

- **<术语 1>**：<解释>
- **<术语 2>**：<解释>

## 禁忌

- ❌ `git push --force` 到 `main` / `develop`
- ❌ 直接修改 `dist/` / `node_modules/` / 自动生成的目录
- ❌ 删除或 skip 测试以让 CI 通过
- ❌ 引入新依赖前不告知（请先讨论必要性与体积）
- ❌ 提交密钥 / `.env`（用 `.env.local`，已 ignore）
- ❌ 使用 `any` 绕过类型问题（除非加 FIXME 注释）

## 工作流偏好

- 复杂改动先进入 Plan Mode 给我看计划再实施
- 改动 > 3 个文件后请自行运行 `pnpm lint` 与 `pnpm typecheck`，把报错修了再回报
- 默认中文回复
- 不主动添加注释 / JSDoc，除非我要求
- 不擅自重构无关代码
