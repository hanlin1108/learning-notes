# 连接 Claude Code 到 GitHub

> 目标：让 Claude Code 在本地干活，改完直接推到 GitHub。
> 只讲 20% 最核心的内容，一看就懂。

---

## 一、它到底是怎么工作的？

Claude Code 本身**没有**连 GitHub 的魔法，它只做两件事：

1. **在你的电脑上改代码**（读文件、写文件）
2. **帮你执行 git 命令**（`git add`, `git commit`, `git push`）

所以「连到 GitHub」= **让 git 能推到 GitHub** + **让 Claude Code 有权限执行这些命令**。

一句话：**Claude Code 是你的代劳 git 助手。**

---

## 二、一次性准备（3 步，做完不用再做）

### 1. 装好 git 并配置身份
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

### 2. 认证 GitHub（推荐用 `gh` CLI，最省事）
```bash
brew install gh            # Mac
gh auth login              # 按提示选 GitHub.com → HTTPS → 浏览器登录
```
✅ 登录后，`git push` 和创建 PR 都免密。

### 3. 本地仓库关联到 GitHub
有两种情况：

**A. 已有 GitHub 仓库** → 克隆到本地
```bash
git clone https://github.com/你的用户名/仓库名.git
cd 仓库名
```

**B. 本地已有项目** → 推到一个新仓库
```bash
cd 项目文件夹
git init
gh repo create            # 一行命令创建 GitHub 仓库并关联
```

---

## 三、核心工作流（90% 的场景就这一个）

在仓库目录下打开 Claude Code，然后：

```
1. 直接描述你要做什么："帮我在 xxx.md 里加一节关于 yyy 的内容"
2. Claude Code 改完文件后，告诉它："提交并推送到 GitHub"
3. 它会执行 git add / commit / push，第一次会请求权限 → 同意即可
```

**就这么简单。** 你不用自己敲 git 命令。

---

## 四、三种最常见的场景（照着抄）

### 场景 1：小改动，直接推到 main
> 适用：个人笔记、dotfiles、单人项目

告诉 Claude Code：
> "帮我修改 README，加一个"关于我"的部分，然后提交并推送到 GitHub。"

### 场景 2：走 PR 流程（团队项目 / 正规做法）
告诉 Claude Code：
> "开一个新分支叫 `feature/add-login`，实现登录功能，commit 后推送，并用 gh 创建 PR。"

Claude Code 会：
```bash
git switch -c feature/add-login
# ... 改代码 ...
git add . && git commit -m "..."
git push -u origin feature/add-login
gh pr create --title "..." --body "..."
```

### 场景 3：从 GitHub 拉最新改动
> 每天开工第一件事

告诉 Claude Code：
> "先 pull 最新的 main，再帮我做 xxx。"

---

## 五、提示词模板（收藏备用）

| 目的 | 跟 Claude Code 说 |
|---|---|
| 保存并推送 | "commit 并 push 到 GitHub" |
| 起新分支干活 | "开一个新分支 `xxx`，做完后推送并创建 PR" |
| 同步远程 | "先 `git pull` 再开始" |
| 查看状态 | "现在仓库是什么状态？有没有未提交的改动？" |
| 撤销本次改动 | "还没 push 的话，撤销最后一次 commit" |

---

## 六、最容易踩的 3 个坑

### 1. ❌ `.env` / API key 被推上去
**一定**先建 `.gitignore`，并告诉 Claude Code "永远不要 commit 敏感文件"。

### 2. ❌ 在错误的分支上干活
开工前先问一句："现在在哪个分支？"

### 3. ❌ 权限没开，push 失败
第一次 push 时 Claude Code 会请求运行 git 命令的权限 → **允许**。
如果频繁打断，可在 `.claude/settings.json` 把 git 常用命令加入白名单。

---

## 七、心智模型（一张图）

```
你              Claude Code           本地 git           GitHub
 │  描述需求   →    │                     │                │
 │                  │  改文件              │                │
 │                  │  ──────────────→    │                │
 │  "推到 GitHub" → │                     │                │
 │                  │  git add/commit/push│                │
 │                  │  ──────────────→    │  ───────────→  │
 │                  │                     │                │ ✅
```

**核心洞察**：Claude Code 不直接连 GitHub，它只是在本地**替你敲 git 命令**。所以只要你本地的 git 能推到 GitHub，Claude Code 就能。

---

## 八、30 秒自检清单

- [ ] `git config --global user.email` 配置了吗？
- [ ] `gh auth status` 显示已登录？
- [ ] 仓库里 `git remote -v` 指向对的 GitHub 地址？
- [ ] `.gitignore` 拦住了 `.env` 和大文件？

全打勾 → 直接让 Claude Code 开始干活。

---

*核心就这么多。剩下的等你需要时再查 `github_notes.md`。*

---

## Method A: Use Claude Code Locally with GitHub (Simplest — Recommended for Beginners)

**One-time setup:**
```bash
gh auth login
```

**Daily use:**
```bash
cd your-project
claude
```

Then describe your task directly, including any GitHub actions. Claude Code will handle `git add`, `git commit`, `git push`, and `gh pr create` automatically. It will ask for your confirmation before touching the repository.

**Example prompts:**
- "Add a login page on the `feature/login` branch, commit and push to origin when done."
- "Summarize the last 5 commits on the main branch."
- "Fix the bug in issue #23, run the tests, then open a PR."

---

## Method B: GitHub App — Let Claude Work Directly on GitHub

### Step 1: Make sure Claude Code is up to date
```bash
claude update
claude --version   # Must be ≥ 1.0.44
```

### Step 2: Log in with your Max account
```bash
claude
```
Inside Claude Code, type `/status` to check the current account. If it's not Max, type `/logout` then `/login` and sign in with your Max account.

> **Warning:** If you have `ANTHROPIC_API_KEY` set as an environment variable, Claude Code will use the API key instead of your Max subscription — this will incur API charges.
> Check with: `echo $ANTHROPIC_API_KEY`
> If it has a value, remove it from `~/.zshrc` / `~/.bashrc` and restart your terminal.

### Step 3: Navigate to your project
```bash
cd your-project-path
claude
```

### Step 4: Install the GitHub App
```
/install-github-app
```
