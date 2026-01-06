# GitHub + Linear 协作工作流

## 1. 工作流概述
通过 GitHub CLI 与 Linear 集成，实现从 Issue 认领、本地开发到 PR 自动关联及任务状态同步的完整闭环。123

## 2. 前置条件
- 已克隆仓库：`dabaige53/auto_work`
- 已安装并登录 GitHub CLI (`gh auth login`)
- 已连接 Linear GitHub App
- Linear 团队 ID：`DATA`（Issue 格式为 DATA-X）
123
## 3. 日常开发流程

### 第一步：同步代码
在开始新任务前，确保本地主分支是最新的。
```bash
git checkout main
git pull origin main
```

### 第二步：创建分支
分支命名规范：`DATA-<Issue编号>-<简短描述>`
建议从 Linear 复制分支名或按照以下格式手动创建。
```bash
git checkout -b DATA-123-add-user-auth
```

### 第三步：提交变更
Commit 消息规范：建议在消息中包含 Issue ID 以便追踪。
```bash
git add .
git commit -m "DATA-123: 实现用户认证模块"
```

### 第四步：推送分支
```bash
git push -u origin DATA-123-add-user-auth
```

### 第五步：创建 Pull Request
使用 `gh` 命令行创建 PR。在描述中包含 `Closes DATA-123` 即可触发自动化状态变更。
```bash
gh pr create --title "DATA-123: 实现用户认证模块" --body "## 摘要
实现基本的用户认证功能。

## 相关 Issue
Closes DATA-123" --base main
```

### 第六步：合并 PR
当 PR 通过评审且 CI 通过后，合并 PR。
```bash
gh pr merge --merge --delete-branch
```
*注：合并后 Linear 中的对应 Issue 将自动转为 "Done" 状态。*

## 4. 快速命令速查表

| 操作         | 命令                                      |
| :----------- | :---------------------------------------- |
| 同步主分支   | `git pull origin main`                    |
| 创建新分支   | `git checkout -b DATA-XXX-feature-name`   |
| 提交代码     | `git commit -m "DATA-XXX: description"`   |
| 推送代码     | `git push -u origin <branch-name>`        |
| 创建 PR      | `gh pr create --title "..." --body "..."` |
| 合并 PR      | `gh pr merge --merge --delete-branch`     |
| 查看 PR 状态 | `gh pr status`                            |

## 5. 常见问题解答 (FAQ)

**Q: 为什么合并 PR 后 Linear 任务状态没有更新？**
A: 请确保 PR 的描述中包含 `Closes DATA-XXX` 或 `Fixes DATA-XXX` 关键字，且 Linear GitHub App 已在仓库中启用。

**Q: 分支名一定要包含 DATA-XXX 吗？**
A: 是的，分支名包含 Issue ID 是 Linear 与 GitHub 建立链接的关键方式。

**Q: 如何在本地快速查看我提交的 PR？**
A: 使用命令 `gh pr list --author "@me"`。
