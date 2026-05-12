# 全局规则（适用于所有项目）

> 本文件应放置在 `~/.claude/CLAUDE.md`，对所有项目生效。
> 项目特定的规则请放到项目根目录的 `CLAUDE.md`，会覆盖/补充这里的内容。

## 沟通

- **默认使用中文回复**（除非项目 CLAUDE.md 要求英文或代码注释场景）
- 简短、直接，不啰嗦套话；不要"You're absolutely right!" 之类的开场白
- 不确定时先问，不要瞎猜
- 引用文件用 markdown 反引号 `path/to/file.ts`

## 工作流

- 复杂任务（涉及多文件 / 不熟悉的代码）**先进入 Plan Mode** 给出计划，等我确认再实施
- 实施完成后，如果有 lint / typecheck / test 命令，**主动运行并报告结果**
- 改动后展示 `git diff` 摘要，方便我 review
- 一次只做一件事；不要顺手"优化"无关代码

## 编码偏好

- 默认包管理器：**pnpm**（如项目用别的包管理器，以项目 CLAUDE.md 为准）
- 不引入新依赖前先告知，并说明为什么 / 体积如何
- 不写无意义注释；保留已有注释，除非我要求改
- 命名清晰胜过简短
- 优先使用语言/框架内置能力，再考虑三方库

## Git

- 提交信息使用 [Conventional Commits](https://www.conventionalcommits.org/)：`feat: ...` / `fix: ...` / `chore: ...` / `refactor: ...` / `docs: ...` / `test: ...`
- 一个 commit 一个目的
- **禁止** `git commit -am`、`git add .`（未审先提）
- **禁止** `git push --force` 到任何共享分支
- 提交前确认 `git status` 没有意外文件

## Git 凭证（重要：节省 token）

GitHub 凭证已经在我本机一次性配好（SSH 或 keychain）。**你不要尝试以下任何操作**：

- ❌ `git config` 修改任何 git 配置
- ❌ `gh auth login` / `gh auth status` 重做 GitHub 授权
- ❌ `security find-internet-password` 等任何 macOS Keychain 操作
- ❌ `export GIT_ASKPASS=...` 或设置 `GIT_TERMINAL_PROMPT=0`
- ❌ 修改 `~/.gitconfig`、`~/.netrc`、`~/.ssh/config`

如果 `git push` / `git pull` / `git fetch` 因为凭证报错（例如 `Permission denied (publickey)`、`could not read Username`、`Authentication failed`），**直接把原始错误打印给我，停手，不要尝试任何修复**。我会自己处理凭证问题。

非凭证类的 git 报错（merge 冲突、non-fast-forward、有未提交文件等）可以正常处理。

## 测试

- **不要删除或跳过测试**来让流水线通过
- 修 bug 时优先添加 / 更新覆盖该 bug 的测试
- 测试失败先分析根因，不要靠改测试断言来"修复"

## 安全

- 不提交密钥、token、`.env`
- 危险命令（`rm -rf`、`DROP TABLE`、`--force`）必须我明确要求才能执行
- 不擅自联网下载未知脚本执行

## 不要做的事

- ❌ 不要为了"提高代码质量"重构无关文件
- ❌ 不要在没有运行的情况下声称"测试通过"
- ❌ 不要伪造代码示例或 API（不知道就说不知道）
- ❌ 不要长篇大论；保持简洁，能用列表就用列表
