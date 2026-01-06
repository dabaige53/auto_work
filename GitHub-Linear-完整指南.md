# GitHub + Linear 自动化工作流完整指南

**目标**：GitHub issue → Linear 自动同步 → 本地开发 → PR → Linear 自动更新 → 完成  
**时间**：整个配置约 15-20 分钟，之后每次开发只需 2 分钟

---ecutvtzw/dat-4-tesdt
123
## 📋 目录

1. [前置要求](#前置要求)
2. [第一部分：GitHub 配置](#第一部分github-配置)
3. [第二部分：Linear 配置](#第二部分linear-配置)123
4. [第三部分：本地 Git 配置](#第三部分本地-git-配置)
5. [第四部分：实际操作流程](#第四部分实际操作流程-一步步来)
6. [第五部分：故障排查](#第五部分故障排查)

---

## 前置要求

- ✅ 有 GitHub 账户和至少一个仓库（用于测试）
- ✅ 有 Linear 账户和一个 Team
- ✅ 对 Git 基本命令有了解
- ✅ 本地安装了 Git 和编辑器

**测试仓库推荐**：新建一个空仓库或用现有项目都可以，这份指南会用 `my-awesome-project` 作例子。

---

# 第一部分：GitHub 配置

## 步骤 1.1：确认 GitHub 仓库设置

打开你的 GitHub 仓库主页，检查：

1. **仓库名字**：记住它（例如 `my-awesome-project`）
2. **链接**：记住完整 URL（例如 `https://github.com/yourname/my-awesome-project`）
3. **Settings → General**：
   - ✅ 确认仓库是 Public 或你有访问权（这样 Linear 能读到 PR）

## 步骤 1.2：启用 GitHub Issues（如果要用）

1. 仓库 **Settings** → 左侧菜单 **Features**
2. 确保 ✅ **Issues** 是打开的
3. 确保 ✅ **Pull Requests** 是打开的

（通常默认都开启，无需改动）

## 步骤 1.3：生成个人访问令牌（Personal Access Token，仅本地需要）

虽然 Linear 连接 GitHub 不需要这个，但为了本地推送方便，建议配置：

1. GitHub 右上角头像 → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. 点 **Generate new token (classic)**
3. 填写：
   - **Note**：`linear-github-automation`
   - **Expiration**：90 days（或你偏好的期限）
   - **Select scopes**：勾选以下权限
     - ✅ `repo` (完整仓库访问)
     - ✅ `workflow` (GitHub Actions)
     - ✅ `admin:repo_hook` (管理 webhook)

4. 点 **Generate token**
5. **复制令牌**并保存到本地密码管理器（之后只能看一次）

## 步骤 1.4：配置本地 Git 认证（可选，推荐用 SSH）

如果你还没配置过 GitHub 认证，三选一：

### 方案 A：SSH（推荐）
```bash
# 生成 SSH 密钥
ssh-keygen -t ed25519 -C "your_email@example.com"
# 全部回车（使用默认位置和空密码）

# 复制公钥
cat ~/.ssh/id_ed25519.pub

# 粘贴到 GitHub Settings → SSH and GPG keys → New SSH key
```

### 方案 B：HTTPS + Token（如果不会 SSH）
```bash
# 配置 Git 使用 token（MacOS/Linux）
git config --global credential.helper osxkeychain

# 或 Windows
git config --global credential.helper wincred

# 首次推送时会提示输入用户名和密码
# 用户名：你的 GitHub 用户名
# 密码：粘贴上面生成的 Personal Access Token
```

---

# 第二部分：Linear 配置

## 步骤 2.1：访问 Linear 并进入设置

1. 打开 [https://linear.app](https://linear.app)
2. 登录你的账户
3. 选择或创建一个 **Team**（如果还没有）
   - 左侧菜单 → 点击 Team 名字
   - 如果没有，创建一个新的（Team name 可以随便取）

## 步骤 2.2：连接 GitHub

### 2.2.1 进入 GitHub 集成页面

1. 点击左上角 **Linear logo** → **Settings**（或 Team Settings）
2. 左侧菜单 → **Integrations**
3. 找到 **GitHub** → 点击 **Install** 或 **Configure**

### 2.2.2 授权 GitHub App

1. 会跳转到 GitHub 的授权页面
2. 看到 "OpenLinear" 或 "Linear" 的 GitHub App 授权请求
3. 点击 **Authorize** 或 **Install**
4. 选择要授权的仓库：
   - **建议**：先只授权你的测试仓库
   - 点 **Only select repositories**
   - 搜索并选中 `my-awesome-project`
5. 点 **Install** → 回到 Linear

### 2.2.3 确认连接成功

Linear 的 Integrations 页面应该显示：
```
✅ GitHub
   Connected
   Authorized repositories: my-awesome-project
```

## 步骤 2.3：配置 PR 自动化规则（最关键）

这一步决定了"PR 合并时 Linear 自动更新"是否工作。

1. Linear 左侧菜单 → **Settings** → **Workspace** 或 **Team**
2. 找到 **Workflow** 或 **Automation**
3. 向下滚动找到 **GitHub Integration** 部分

### 查找或配置这些规则：

| 规则 | 触发条件 | Linear 状态 | 是否启用 |
|------|---------|-----------|--------|
| PR Opened | GitHub PR 被创建 | In Progress | ✅ 启用 |
| PR Review Requested | 请求审核 | In Review | ✅ 启用 |
| PR Ready to Merge | 可合并（CI 通过） | Ready to Merge | ✅ 启用 |
| PR Merged | PR 已合并 | Done | ✅ 启用 |

**确保这些都是绿色对勾 ✅**

如果没看到这些选项：
- 可能你的 Linear 版本较旧，跳过此步（手动也能用）
- 或者去 **Integrations** 页面找 GitHub 的详细配置按钮

## 步骤 2.4：配置 Issue 同步（可选）

如果你想：**GitHub issue 创建 → 自动在 Linear 创建 issue**

1. 在 GitHub Integration 页面，找到 **Issues Sync**
2. 开启 **Sync GitHub issues to Linear**
3. 选择同步方向：
   - **One-way: GitHub → Linear**（推荐，GitHub 为主）
   - **Two-way**（如果 Linear 和 GitHub 都要用）

4. 指定哪些 Label 要同步（例如 `bug`, `feature`）

## 步骤 2.5：配置 Commit 链接（可选但有用）

让 GitHub commit message 中的 `ABC-123` 自动链接到 Linear issue：

1. GitHub Integration 页面 → **Commit Linking**
2. 开启 **Link commits to issues**
3. 选择 commit message 格式：
   - `ABC-123: message`（推荐）
   - `[ABC-123] message`
   - `message (#ABC-123)`

之后你 commit 时就写：
```bash
git commit -m "ABC-123: fix user auth bug"
```

---

# 第三部分：本地 Git 配置

## 步骤 3.1：配置全局 Git 用户信息

打开终端/命令行，运行：

```bash
# 替换成你的 GitHub 用户名和邮箱
git config --global user.name "Your GitHub Username"
git config --global user.email "your.email@example.com"

# 验证是否设置成功
git config --global user.name
git config --global user.email
```

## 步骤 3.2：克隆仓库到本地

```bash
# 替换成你的仓库地址
git clone https://github.com/yourname/my-awesome-project.git

# 进入仓库目录
cd my-awesome-project

# 确认分支
git branch
# 应该看到 * main (或 master)
```

## 步骤 3.3：设置上游分支（推荐但可选）

如果这是团队仓库，建议配置上游：

```bash
# 查看当前 remote
git remote -v
# 应该看到 origin 指向你的仓库

# 如果要添加上游（例如团队的中央仓库）
git remote add upstream https://github.com/teamname/my-awesome-project.git
```

---

# 第四部分：实际操作流程 一步步来

## 🎯 场景：修复一个 bug

### 步骤 4.1：在 Linear 中创建或打开 Issue

1. 打开 Linear
2. 你的 Team → **Issues**
3. 创建新 Issue 或打开现有的

**填写内容**：
- **Title**：`Fix user login redirect bug`
- **Description**：
  ```
  当用户登录后，重定向到首页而不是上次访问的页面。
  应该保存 referrer 并在登录后跳转回去。
  
  Steps to reproduce:
  1. 访问 /about
  2. 点击登录
  3. 登录成功后被重定向到首页
  ```
- **Priority**：High（或你的优先级）
- **Assignees**：选择你自己
- **Status**：新创建默认是 "Backlog" 或 "Todo"，保持不变

4. 点 **Create Issue**

**记住 Issue ID**（例如 `AUTH-42`）

### 步骤 4.2：从 Linear 生成分支名（关键！）

1. 打开你刚创建的 issue（例如 `AUTH-42 Fix user login redirect bug`）
2. 右上角或右侧边栏，找到 **"Copy branch name"** 按钮
   - 有时候在 "..." 菜单里
   - 或直接在右侧 "Development" 部分有个按钮
3. 点击 → **复制分支名到剪贴板**

应该复制到：
```
AUTH-42-fix-user-login-redirect-bug
```

**如果没看到这个按钮**：
- 手动组织分支名格式：`ABC-123-describe-the-issue`
- 例如：`AUTH-42-fix-login-redirect`

### 步骤 4.3：本地创建分支并开发

打开终端，运行：

```bash
# 1️⃣ 确保在主分支上，并同步最新代码
git checkout main
git pull origin main

# 2️⃣ 创建新分支（粘贴 Linear 复制的分支名）
git checkout -b AUTH-42-fix-user-login-redirect-bug

# 3️⃣ 验证分支创建成功
git branch
# 应该看到 * AUTH-42-fix-user-login-redirect-bug
```

**此时，Linear 中的 issue 状态可能自动变为 "In Progress"**（取决于你的配置）

### 步骤 4.4：编写代码

用你的编辑器修改代码。例如编辑 `src/auth.js`：

```javascript
// src/auth.js
export function login(username, password) {
  // ... 验证逻辑 ...
  
  // ✨ 新增：保存当前 URL
  const referrer = sessionStorage.getItem('referrer') || '/';
  
  // 登录成功后重定向
  window.location.href = referrer;
  
  // ✨ 清除 referrer
  sessionStorage.removeItem('referrer');
}
```

### 步骤 4.5：提交代码

```bash
# 1️⃣ 查看修改
git status
# 应该看到 modified: src/auth.js

# 2️⃣ 添加文件到暂存区
git add src/auth.js
# 或添加所有修改
git add .

# 3️⃣ 提交
# ⭐ 重要：commit message 最好包含 Linear issue ID
git commit -m "AUTH-42: fix user login redirect to referrer"

# 或按下列格式（任选其一都可以）
git commit -m "[AUTH-42] Fix user login redirect bug"
git commit -m "fix: user login redirect - AUTH-42"

# 4️⃣ 推送到 GitHub
git push origin AUTH-42-fix-user-login-redirect-bug
```

**推送后，GitHub 和 Linear 应该自动建立链接**

### 步骤 4.6：在 GitHub 创建 Pull Request

#### 方案 A：GitHub Web UI（推荐新手）

1. 打开 GitHub 仓库主页
2. 应该看到一条黄色提示：
   ```
   AUTH-42-fix-user-login-redirect-bug had recent pushes
   [Compare & pull request]
   ```
3. 点 **Compare & pull request**

#### 方案 B：手动创建

1. GitHub 仓库 → **Pull requests** 标签
2. 点 **New pull request**
3. 选择：
   - **base**：`main`（主分支）
   - **compare**：`AUTH-42-fix-user-login-redirect-bug`（你的分支）
4. 点 **Create pull request**

#### 填写 PR 信息

```markdown
## 标题
Fix user login redirect bug

## 描述
Closes #AUTH-42

### 问题
用户登录后被重定向到首页，而不是他们原来访问的页面。

### 解决方案
- 在页面加载时，将当前 URL 保存到 sessionStorage
- 登录成功后，重定向到保存的 URL 而不是硬编码的首页

### 类型
- [x] Bug fix
- [ ] Feature
- [ ] Breaking change

### 测试
- [x] 在 Chrome 中测试
- [x] 在 Firefox 中测试
- [ ] 在 Safari 中测试

### Checklist
- [x] 代码遵循项目风格
- [x] 已自测
- [x] 添加/更新了注释
- [ ] 更新了文档
```

**关键一行**（Linear 需要识别）：
```
Closes #AUTH-42
或
Closes linear: AUTH-42
```

5. 点 **Create pull request**

**此时，Linear 中的 issue 应该自动变为 "In Progress" 或显示关联 PR**

### 步骤 4.7：代码审核

#### 在 GitHub 上：

1. 等待团队成员审核（或自己审核）
2. 如果有修改意见，GitHub 会显示在 PR 里
3. 本地修改代码，重新 add → commit → push 到**同一分支**
   ```bash
   # 修改代码后
   git add .
   git commit -m "AUTH-42: address review feedback"
   git push origin AUTH-42-fix-user-login-redirect-bug
   ```
4. 重新请求审核或等待自动审核通过

#### 在 Linear 上同步：

- GitHub PR 的评论会显示在 Linear issue 的 **Activity** 里
- Linear issue 的评论也会同步到 GitHub PR

### 步骤 4.8：合并 PR

当代码审核通过，CI 检查全部绿色 ✅ 后：

#### 方案 A：GitHub Web UI（推荐）

1. GitHub PR 页面滚到底部
2. 如果显示 "All checks passed"，看到绿色 ✅
3. 点 **Merge pull request**
4. 选择合并方式（通常默认 "Create a merge commit"）
5. 点 **Confirm merge**

#### 方案 B：本地合并（可选）

```bash
# 切换回 main
git checkout main

# 同步远程最新
git pull origin main

# 合并你的分支
git merge --no-ff AUTH-42-fix-user-login-redirect-bug

# 推送
git push origin main

# 删除本地分支（可选）
git branch -d AUTH-42-fix-user-login-redirect-bug

# 删除远程分支（推荐，保持整洁）
git push origin --delete AUTH-42-fix-user-login-redirect-bug
```

**合并后，Linear 中的 issue 应该自动变为 "Done"** ✅

### 步骤 4.9：验证完成

1. **在 Linear 中**：
   - 打开 issue `AUTH-42`
   - 应该看到状态是 **"Done"**（绿色）
   - 右侧应该显示关联的 PR 链接

2. **在 GitHub 中**：
   - PR 应该显示为 "Merged"
   - 分支应该被删除或标记为"已删除"

3. **在本地**：
   ```bash
   git checkout main
   git pull origin main
   
   # 验证你的修改已经在 main 分支上
   git log --oneline | head -5
   # 应该看到 "AUTH-42: fix user login redirect..." 的 commit
   ```

---

# 第五部分：故障排查

## 问题 1：PR 合并后 Linear 没有自动变成 Done

### 原因检查清单

1. **PR 没有正确链接 Linear issue**
   - 检查：GitHub PR 页面右侧，应该有 "Linked issues" 或 "AUTH-42" 的显示
   - 修复：在 PR 描述里确保有一行：
     ```
     Closes linear: AUTH-42
     或
     Closes #AUTH-42
     ```

2. **CI/CD 检查失败**
   - GitHub PR 页面看 "Checks" 部分是否全部 ✅ 通过
   - 如果有红叉 ❌，Linear 不会自动更新
   - 修复：修改代码使 CI 通过

3. **Linear 的自动化规则没启用**
   - 检查：Linear Settings → Workflow → 确保 "PR Merged → Done" 规则是启用的
   - 修复：手动打开规则开关

4. **GitHub App 授权权限不足**
   - 检查：Linear Settings → Integrations → GitHub → 看是否显示 "Connected" 
   - 修复：重新授权，选择"所有仓库"或明确选中你的仓库

### 临时解决方案

如果自动化不工作，手动更新：
```bash
# 在 GitHub PR 里评论
comment: "fixes #AUTH-42"

# 或手动在 Linear 里改状态
Linear issue → 右上角 Status → 选 "Done"
```

---

## 问题 2：本地分支和 GitHub 远程分支不同步

### 症状
```
git push 时出现错误：
fatal: The current branch AUTH-42-... has no upstream branch
```

### 解决方案

```bash
# 方案 A：第一次推送时指定上游
git push -u origin AUTH-42-fix-user-login-redirect-bug

# 方案 B：如果已经推送了，关联上游
git branch -u origin/AUTH-42-fix-user-login-redirect-bug

# 方案 C：全局配置（推荐）
git config --global push.default current
# 之后 git push 会自动推送到同名远程分支
```

---

## 问题 3：忘记创建分支，直接在 main 上修改了代码

### 补救步骤

```bash
# 1️⃣ 查看你的修改
git status

# 2️⃣ 创建新分支（会包含当前的修改）
git checkout -b AUTH-42-fix-user-login-redirect-bug

# 3️⃣ 推送这个新分支
git push -u origin AUTH-42-fix-user-login-redirect-bug

# 4️⃣ 创建 PR（按步骤 4.6）

# 5️⃣ main 分支应该没有变化（因为你没有 push 到 main）
# 如果 main 已经被污染了，联系团队负责人恢复
```

---

## 问题 4：Linear issue 没有自动识别我的分支

### 原因

- 分支名不符合 `TEAM-NUMBER-description` 格式
- Linear 团队 Team ID 设置不对

### 修复

1. **检查 Linear Team ID**：
   - Linear Settings → General → 找 "Team ID"（例如 `AUTH`）

2. **确保分支名格式**：
   ```bash
   # ✅ 正确
   AUTH-42-fix-user-login-redirect-bug
   [TEAMID]-[NUMBER]-[description]
   
   # ❌ 错误
   fix-user-login-redirect-bug
   42-fix-user-login-redirect-bug
   ```

3. **手动创建分支**（如果复制不到）：
   ```bash
   git checkout -b AUTH-42-fix-user-login-redirect-bug
   ```

---

## 问题 5：GitHub 和 Linear 连接断开了

### 排查步骤

1. **检查 Linear 的 GitHub App 状态**：
   - Linear Settings → Integrations → GitHub
   - 应该显示 "✅ Connected"
   - 如果是 "❌ Disconnected"，点 "Reconnect"

2. **重新授权**：
   ```
   1. Linear Integrations → GitHub → 点菜单 → "Disconnect"
   2. 等 10 秒
   3. 点 "Install" 或 "Connect"
   4. 授权 GitHub App
   5. 完成
   ```

3. **检查 GitHub 端的 App 授权**：
   - GitHub Settings → Applications → Authorized OAuth Apps
   - 找 "Linear" → 确保权限还在
   - 如果被删除了，Linear 会自动提示重新连接

---

## 问题 6：PR merge 了但想撤回（revert）

### 如果还没推送到 main

```bash
git reset --soft HEAD~1
# 修改代码
git add .
git commit -m "新的 commit"
git push
```

### 如果已经 merge 到 main

```bash
# 创建一个新分支用于恢复
git checkout main
git pull origin main

# revert（创建一个新 commit 来撤销之前的 commit）
git revert <commit-hash>
# commit hash 可以从 git log 里找到

git push origin main

# 在 Linear 里手动改状态回 "In Progress" 或 "Todo"
```

---

# 完整速查表

## 一次完整工作流的命令

```bash
# 📍 第 1 步：准备
git checkout main
git pull origin main

# 📍 第 2 步：创建分支（复制 Linear 的分支名）
git checkout -b AUTH-42-fix-user-login-redirect-bug

# 📍 第 3 步：开发（修改你的代码）
# ... 编辑文件 ...

# 📍 第 4 步：提交
git add .
git commit -m "AUTH-42: fix user login redirect bug"

# 📍 第 5 步：推送
git push -u origin AUTH-42-fix-user-login-redirect-bug

# 📍 第 6 步：创建 PR（GitHub 网页操作）
# 访问 GitHub 仓库，点 "Compare & pull request"
# 标题：Fix user login redirect bug
# 描述：Closes #AUTH-42

# 📍 第 7 步：审核和修改（如需要）
# ... 审核反馈 → 修改代码 ...
# git add . && git commit -m "AUTH-42: address review" && git push

# 📍 第 8 步：合并（GitHub 网页操作）
# PR 页面 → 点 "Merge pull request"

# 📍 第 9 步：清理（可选）
git checkout main
git pull origin main
git branch -d AUTH-42-fix-user-login-redirect-bug
git push origin --delete AUTH-42-fix-user-login-redirect-bug
```

## 常用命令速查

| 操作 | 命令 |
|------|------|
| 查看所有分支 | `git branch -a` |
| 删除本地分支 | `git branch -d branch-name` |
| 删除远程分支 | `git push origin --delete branch-name` |
| 查看最近提交 | `git log --oneline -10` |
| 查看分支差异 | `git diff main..branch-name` |
| 中止当前修改 | `git reset --hard HEAD` |
| 重新获取远程更新 | `git fetch origin` |
| 同步 main 分支最新代码 | `git fetch origin && git rebase origin/main` |

---

# ✅ 配置完成检查清单

复制下面的清单，完成所有项目：

```
[ ] 1. GitHub 账户和仓库已创建
[ ] 2. GitHub SSH 或 HTTPS 认证已配置
[ ] 3. Linear 账户和 Team 已创建
[ ] 4. Linear GitHub App 已安装并授权
[ ] 5. Linear PR 自动化规则已启用（4 条规则都是 ✅）
[ ] 6. 本地 Git 用户信息已配置
[ ] 7. 仓库已克隆到本地
[ ] 8. 用一个测试 issue 跑过一遍完整流程
[ ] 9. 验证 PR 合并后 Linear issue 自动变成 Done
[ ] 10. 保存此文档和你的 team ID、仓库地址
```

---

# 🚀 进阶技巧（可选）

## 技巧 1：Git Alias（节省打字）

```bash
# 创建别名
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.ps push
git config --global alias.pl pull
git config --global alias.log 'log --oneline --graph'

# 使用
git co -b AUTH-42-xxx  # 代替 git checkout -b
git cm -m "message"    # 代替 git commit -m
git ps origin xxx      # 代替 git push origin xxx
```

## 技巧 2：GitHub CLI 加速（可选，更高级）

如果你想从命令行直接创建 PR（而不用网页）：

```bash
# 安装 GitHub CLI
# MacOS: brew install gh
# Ubuntu: sudo apt install gh
# Windows: choco install gh

# 登录
gh auth login

# 创建 PR（自动）
gh pr create --title "Fix user login redirect bug" \
             --body "Closes #AUTH-42" \
             --base main
```

## 技巧 3：Commit 之前自动检查

```bash
# 创建 pre-commit hook（可选）
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
# 检查是否 commit message 包含 issue ID
COMMIT_MSG=$(cat $1)
if ! [[ $COMMIT_MSG =~ ^[A-Z]+-[0-9]+ ]]; then
    echo "❌ Commit message 应该以 ISSUE-123 开头"
    exit 1
fi
EOF

chmod +x .git/hooks/pre-commit
```

---

# 📞 需要帮助？

如果遇到问题，按以下顺序排查：

1. **查看错误信息** → 复制粘贴到 [Google](https://google.com) 或 [Stack Overflow](https://stackoverflow.com)
2. **检查此文档** → 在第五部分"故障排查"找答案
3. **GitHub 官方文档** → https://docs.github.com
4. **Linear 官方文档** → https://linear.app/docs

---

**最后一句话**：完全配置好后，你只需要记住这 3 件事：
1. 从 Linear 复制分支名
2. 本地开发 → push
3. GitHub 创建 PR → merge

剩下的全部自动化！

---

*文档最后更新：2026 年 1 月 6 日*
