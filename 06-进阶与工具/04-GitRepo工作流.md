# Git / Repo 工作流

## 概述

Git 是内核开发的基础工具。Repo 是 Google 为 Android 多仓库管理开发的工具，广泛应用于高通 Android BSP 开发。

> 高通工作场景：JD 要求 "Familiarity with Git, repo, and baseline upgrade, commit upstream"。你每天都会使用 git 和 repo 管理内核代码、处理基线升级。

## Git 高级用法

### 内核开发常用 Git 操作

```bash
# 配置
git config --global user.name "Your Name"
git config --global user.email "your.email@qualcomm.com"
git config --global core.editor "vim"

# 查看远程分支
git remote -v
git branch -a

# 基于远程分支创建本地分支
git checkout -b my-feature origin/master

# 交互式 rebase（整理提交历史）
git rebase -i HEAD~5
# 常用命令：
#   pick    保留该提交
#   squash  合并到前一个提交
#   fixup   合并到前一个提交（丢弃提交信息）
#   reword  修改提交信息
#   edit    修改提交内容

# 修补最近的提交（不修改信息）
git add fixed-file.c
git commit --amend --no-edit

# 从上游 rebase（内核开发推荐方式）
git fetch origin master
git rebase origin/master
# 不要使用 git pull（会创建 merge commit）
```

### Cherry-pick — 选择性合入

```bash
# 从其他分支 cherry-pick 特定的提交
git cherry-pick <commit-hash>

# cherry-pick 一系列提交
git cherry-pick <start>..<end>

# 解决冲突后用
git cherry-pick --continue
# 或取消
git cherry-pick --abort

# 高通场景：从老内核 pick 驱动补丁到新内核
# 或者从 linux-next pick 修复补丁
```

### Bisect — 二分查找引入 bug 的提交

```bash
# 场景：新内核 wifi 坏了，找到是哪个提交导致的
git bisect start
git bisect bad HEAD           # 当前版本有问题
git bisect good v5.10         # v5.10 是好的

# Git 会 checkout 一个中间版本
# 测试后标记：
git bisect good   # 如果这个版本没问题
git bisect bad    # 如果这个版本有问题

# 重复直到找到第一个 bad commit
git bisect reset  # 结束 bisect 会话
```

### Stash — 临时保存

```bash
# 保存当前未提交的修改
git stash save "WIP: fixing I2C probe issue"
git stash list
git stash pop
git stash drop stash@{0}

# 使用场景：需要切换到其他分支修 bug
```

## Repo 工具

Repo 是 Google 开发的多仓库管理工具，Android 内核有数百个 git 仓库：

```bash
# 初始化 Repo
repo init -u https://android.googlesource.com/platform/manifest \
          -b android-14.0.0_r30

# 同步所有仓库
repo sync -j$(nproc)    # -j 并行同步

# 查看所有仓库状态
repo status

# 在所有仓库中执行命令
repo forall -c "git status"
repo forall -c "git log --oneline -3"

# 创建 repo 主题分支（跨仓库创建同名分支）
repo start my-feature --all

# 查看 repo 分支信息
repo info

# 上传代码到 Gerrit
repo upload

# 丢弃所有修改
repo forall -c "git checkout . && git clean -fd"
```

## 基线升级（Baseline Upgrade）

基线升级是高通内核开发中频繁的操作：将 Android 通用内核（ACK）的更新合并到高通平台内核中。

### 基本流程

```bash
# 场景：将 android-5.15 的最新补丁合并到 qcom 内核

# 1. 添加 Android 内核远程仓库
cd kernel/msm-5.15
git remote add ack https://android.googlesource.com/kernel/common
git fetch ack android-5.15-2024-01

# 2. 基于当前主干创建合并分支
git checkout -b merge-ack-jan2024 origin/master

# 3. 尝试合并
git merge ack/android-5.15-2024-01

# 4. 解决冲突
# 常见冲突类型：
#   - 文件位置不同（高通改了路径）
#   - 同一处代码的修改不同
#   - 高通的 hunk 与 ACK 的 hunk 冲突
git mergetool  # 使用可视化工具
# 或手动编辑冲突文件
git add <resolved-file>
git merge --continue

# 5. 构建测试
make qcom_defconfig
make -j$(nproc) Image dtbs
```

### 冲突解决技巧

```bash
# 查看冲突文件列表
git diff --name-only --diff-filter=U

# 查看某个冲突的详情
git diff HEAD...MERGE_HEAD -- drivers/i2c/busses/i2c-qcom-geni.c

# 用 "ours" 或 "theirs" 策略
# ours = 当前分支（高通侧）
# theirs = 合并目标（ACK 侧）
git checkout --ours drivers/i2c/busses/i2c-qcom-geni.c
git checkout --theirs drivers/i2c/busses/i2c-qcom-geni.c

# 查看合并三方差异
git log --oneline HEAD..MERGE_HEAD  # ACK 有哪些新提交？
```

## Git 内核开发最佳实践

```bash
# 1. 每个提交只做一件事（原子提交）
# 2. 提交信息清晰（说明 WHY 而不是 WHAT）
# 3. 在功能分支上开发，不要在主分支上
# 4. Rebase 而不是 Merge 来同步上游
# 5. 经常和上游同步，避免大量冲突

# 检查即将发送的补丁
git log --oneline origin/master..HEAD

# 检查提交中的修改文件
git diff --stat origin/master..HEAD

# 查看当前分支和上游的差异
git rebase -i origin/master  # 整理提交
```

## 实操练习

### 1. 模拟基线升级冲突

```bash
# 创建练习场景
mkdir git-merge-practice && cd git-merge-practice
git init

# 创建 "上游" 分支
git checkout -b upstream
echo "line1\nline2\nline3" > driver.c
git add driver.c && git commit -m "initial driver"

# 创建 "高通" 分支
git checkout -b qcom
echo "line1\nline2\nline3\nqcom_debug\n" >> driver.c
git add driver.c && git commit -m "qcom: add debug"

# 回到 upstream 并修改同样文件
git checkout upstream
echo "line1\nline2\nline3\nupstream_fix\n" >> driver.c
git add driver.c && git commit -m "fix: upstream fix"

# 尝试合并（会产生冲突）
git checkout qcom
git merge upstream

# 练习解决冲突
```

### 2. Repo 操作练习

```bash
# 创建一个练习 repo 环境
mkdir repo-practice && cd repo-practice

# 模拟 repo 多仓库
git init kernel/msm-5.15
git init vendor/qcom
git init device/lemon

# repo status 的手动版本
for dir in kernel/msm-5.15 vendor/qcom device/lemon; do
    echo "=== $dir ==="
    (cd $dir && git status -s)
done
```

## 真实 GitHub 资源

| 仓库 | 说明 |
|------|------|
| [git/git](https://github.com/git/git) | Git 源码 |
| [android/tools-repo](https://android.googlesource.com/tools/repo/) | Repo 工具源码 |
| [git-scm.com](https://git-scm.com/doc) | Git 官方文档 |
| [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials) | Git 教程 |
| [ohmygit](https://github.com/git-learning-game/oh-my-git) | Git 游戏化学习 |

## 面试考点

1. **git rebase 和 git merge 的区别？** rebase 重写提交历史，merge 创建合并提交。内核开发推荐 rebase
2. **git bisect 的原理？** 二分查找，每次将搜索范围减半，O(log n) 次操作
3. **repo 工具的主要作用？** 同时管理多个 git 仓库，保持它们的分支同步
4. **基线升级时的冲突来源？** 高通和 ACK 各自修改了同一段代码、文件位置移动、格式不同
5. **git pull --rebase 的好处？** 避免多余的 merge commit，保持提交历史线性
