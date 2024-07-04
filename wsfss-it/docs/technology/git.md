---
comments: true
---

`git` 是一个分布式版本控制系统，用于敏捷高效地处理项目变更。

## Git 常用命令

### 初始化

```linenums="1"
git init # 初始化本地git仓库
git config --global user.name "your name" # 配置用户名
git config --global user.email "your@email.com" # 配置邮件
git remote add origin https://github.com/your/repo.git # 添加远程仓库关联
git config --global --unset http.proxy # 不使用代理
git clone https://github.com/your/repo.git # 克隆远程仓库
```

### 代码提交

```linenums="1"
git add . # 添加所有文件
git commit -m "commit message" # 提交
git push origin master # 推送到远程仓库 master 分支
```

### 分支管理

```linenums="1"
git branch # 查看当前分支
git branch -a # 查看所有分支
git branch -D branch-name # 删除本地分支
git push origin --delete branch-name # 删除远程分支
git status # 查看分支状态（是否修改）
git checkout -b branch-name # 创建新分支
git checkout branch-name # 切换到分支
git merge master # 合并本地 master 分支代码到当前分支
git merge origin/master # 合并远程 master 分支代码到当前分支
git branch -m master master_other_name  # 本地分支改名
git fetch # 获取所有远程分支（不更新本地分支，另需merge）
git fetch --prune # 获取所有原创分支并清除服务器上已删掉的分支
git show-branch # 图示当前分支历史
```

### 日志查看

```linenums="1"
git log # 查看提交日志
git log -1 # 显示1行日志 -n为n行
git log --stat # 显示提交日志及相关变动文件
git show 6e048c19 # 显示某个提交的详细内容 6e048c19 为提交记录ID
git log v2.0 # 显示版本标签v2.0的日志
```

### Tag

```linenums="1"
git tag # 显示已存在的tag
git tag -a v1.0 -m "v1.0" # 创建版本标签
git push origin v1.0 # 推送某个标签
git show v1.0 # 显示v1.0的日志及详细内容
git push --tags  # 把所有tag推送到远程仓库
```

### 显示变更

```linenums="1"
git diff # 显示工作区和暂存区差异
git diff --cached # 显示暂存区和HEAD差异（还未commit
git diff HEAD^ # 比较与上一个版本的差异
git diff origin/master..master # 比较远程分支master上有本地分支master上没有的
git diff origin/master..master --stat # 只显示差异的文件，不显示具体内容
```

### 代码回退

```
git stash # 暂存当前修改，并且将当前代码切换到HEAD提交上
git stash list # 显示暂存列表
git show HEAD # 显示HEAD提交日志
git show HEAD^  # 显示HEAD上一个版本的提交日志 ^^为上两个版本 ^5为上5个版本
git reset --hard HEAD # 将当前版本重置为HEAD（通常用于merge失败回退）
git reset --hard commit-id # 将当前版本重置为指定的提交版本
```

## 应用举例

### 代码冲突处理

当你的开发进行到一半，但是代码还不想进行提交，然后需要同步去关联远端代码时，如果你本地的代码和远端代码没有冲突时，可以直接通过 `git pull` 解决，但是如果可能发生冲突怎么办，直接 `git pull` 会拒绝覆盖当前的修改。这个时候，你可能需要 `git pull --rebase`。

有时候 `git pull --rebase` 可能不起作用，这个时候，你可能需要用到 `git stash`。`git stash` 将本地的修改保存起来，并且将当前代码切换到 HEAD 提交上，然后进行 `git pull`，再 pop 出本地代码：

```linenums="1"
git stash
git pull
git stash pop
```