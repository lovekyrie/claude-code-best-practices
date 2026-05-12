# GitHub 推送凭证：一次性配置

> **核心理念**：凭证管理是一次性、有副作用、需要人工交互的事。**绝对不要让 Claude Code 帮你做**，它会兜圈子、消耗大量 token、并且每条命令都要你授权。
> 配好一次后，Claude 只需要 `git push`，永远成功。

## 推荐：SSH 方式（最快、最稳）

### 1. 生成 SSH key（如果没有）

```bash
ls -la ~/.ssh/id_ed25519.pub 2>/dev/null && echo "已存在" || \
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/id_ed25519 -N ""
```

### 2. 把公钥加到 GitHub

```bash
pbcopy < ~/.ssh/id_ed25519.pub
echo "公钥已复制到剪贴板，去 https://github.com/settings/keys 粘贴添加"
open https://github.com/settings/keys
```

### 3. 验证

```bash
ssh -T git@github.com
# 看到 "Hi <username>! You've successfully authenticated" 就成功
```

### 4. 把已有的 HTTPS 远端切换为 SSH

```bash
# 在每个项目里跑一次
git remote set-url origin git@github.com:<owner>/<repo>.git
git remote -v   # 确认变成 git@github.com:...
```

新克隆仓库时直接用 SSH URL：

```bash
git clone git@github.com:<owner>/<repo>.git
```

## 备选：HTTPS + macOS Keychain

如果你必须用 HTTPS（公司网络等限制 SSH）：

### 1. 装 GitHub CLI 并登录（一次）

```bash
brew install gh
gh auth login    # 选 HTTPS，按提示登录浏览器授权
```

`gh auth login` 会自动把凭证存进 macOS Keychain，**之后 `git push` 不会再问密码**。

### 2. 验证

```bash
gh auth status   # 看到 "Logged in to github.com" 就成功
git ls-remote origin   # 应该不弹任何密码框
```

## 配好之后：让 Claude 只跑 `git push`

`global-CLAUDE.md` 里已经加了规则：**Claude 不准碰任何 git 配置、credential、keychain、ASKPASS**。如果 `git push` 失败，它会直接把错误抛给你，由你人工处理。

`settings.global.json` 的 `deny` 也加了硬拦截，下面这些命令 Claude 永远跑不了：

- `Bash(git config --global:*)` — 修改全局 git 配置
- `Bash(security:*)` — 操作 macOS Keychain
- `Bash(gh auth:*)` — 重做 GitHub 授权
- `Bash(export GIT_ASKPASS*)` / `Bash(GIT_ASKPASS=*)` — 绕开凭证机制
- `Bash(GIT_TERMINAL_PROMPT=*)` — 同上

## 常见报错对照表

| 报错 | 真正原因 | 处理 |
|------|---------|------|
| `Permission denied (publickey)` | SSH key 没加到 GitHub | 跑上面"推荐方式"第 2 步 |
| `fatal: could not read Username` | HTTPS 没凭证 | 跑 `gh auth login` |
| `remote: Repository not found` | 仓库名错 / 没权限 | 自己检查 `git remote -v` 和 GitHub 上的权限 |
| `! [rejected] non-fast-forward` | 远端有新提交 | `git pull --rebase` 后再 push |

⚠️ **任何凭证类报错，让 Claude 帮你 `git pull --rebase` 是 OK 的，但凡涉及 config / credential / keychain 的步骤，自己来。**
