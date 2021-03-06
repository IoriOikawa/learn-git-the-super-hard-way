# 第0章：创建工作环境

## 基础知识

- Git在硬盘上维护软件配置信息。具体形式是文件夹（Git repo）。
- 为了方便用户编辑，每一个repo都可以选配至多一个worktree。
- 为了方便多个repo之间共享数据，一个repo可以放弃对象和引用信息的所有权，将其全权托管给另一个repo。
  - 很多人认为一个repo可以有多个worktree，严格意义上讲其实是多个repo都链接到了同一个repo，每一个repo贡献一个worktree，看起来是一个大repo有多个worktree的样子

## 创建Git repo并选配worktree

### Lv0

```bash
# Git repo以.git结尾只是惯例
mkdir the-repo.git
# 所有的Git repo都必须包括objects文件夹，用来保存对象
mkdir the-repo.git/objects
# 所有的Git repo都必须包括refs文件夹，用来保存引用
mkdir the-repo.git/refs
# 所有的Git repo都必须包括HEAD
# 但是HEAD指向的目标不一定需要真实存在
# 在初始化时指向refs/heads/master只是惯例
echo 'ref: refs/heads/master' > the-repo.git/HEAD
```

至此一个最简单的Git repo创建完毕，采用`git symbolic-ref`（Lv2）检验是否创建成功：
```bash
git --git-dir=the-repo.git symbolic-ref HEAD
# refs/heads/master
```

现在添加worktree：
```bash
# 默认worktree不需要进行任何复杂操作
# 任意文件夹都可以被视作worktree
mkdir default-tree
```

采用`git status --porcelain`（Lv2）和`git status`（Lv3）检验是否创建成功：
```bash
git --git-dir=the-repo.git --work-tree=default-tree status --porcelain
# 没有输出表明worktree是干净的
git --git-dir=the-repo.git --work-tree=default-tree status
# On branch master
#
# No commits yet
#
# nothing to commit (create/copy files and use "git add" to track)
```

为了简化命令行调用方式，在worktree下添加.git文件：
```bash
echo "gitdir: $(pwd)/the-repo.git" > default-tree/.git
```

现在一切已经就绪，采用`git status`（Lv3）检查：
```bash
cd default-tree
git status
# On branch master
#
# No commits yet
#
# nothing to commit (create/copy files and use "git add" to track)
```

#### 合并repo和worktree

如果希望彻底避免绝对路径，可以把repo放在worktree里面：
```bash
rm default-tree/.git
mv the-repo.git default-tree/.git
```

采用`git status`（Lv3）检查：
```bash
cd default-tree
git status
# On branch master
#
# No commits yet
#
# nothing to commit (create/copy files and use "git add" to track)
```

### Lv3

日常创建repo、不选配worktree：
```bash
rm -rf the-repo.git
git init --bare the-repo.git
# Initialized empty Git repository in /root/the-repo.git/
```

日常创建repo、选配worktree、把repo放在worktree里面：
```bash
rm -rf default-tree
git init default-tree
# Initialized empty Git repository in /root/default-tree/.git/
```

日常创建repo、选配worktree但不把repo放在worktree里面：
```bash
rm -rf the-repo.git default-tree
git init --separate-git-dir the-repo.git default-tree
# Initialized empty Git repository in /root/the-repo.git/
```

## 添加新repo并链接到原repo，以实现“一个repo多个worktree”

### Lv0

```bash
git init --bare the-repo.git
# Reinitialized existing Git repository in /root/the-repo.git/
# 惯例是将小repo放在这个位置：
mkdir -p the-repo.git/worktrees/another/
# 为了和大repo建立起联系，创建commondir文件：
echo '../..' > the-repo.git/worktrees/another/commondir
# 小repo也是repo，必须要有HEAD
# 注意：让不同worktree的HEAD指向同一个引用会导致一个worktree的修改影响另一个
# 这虽然合法但是违背了worktree创立的初衷
echo 'ref: refs/heads/another' > the-repo.git/worktrees/another/HEAD
```

至此一个最简单的小repo创建完毕，采用`git symbolic-ref`（Lv2）检验是否创建成功：
```bash
# 特别注意此处的git-dir已经发生变化
git --git-dir=the-repo.git/worktrees/another symbolic-ref HEAD
# refs/heads/another
```

和普通repo一样，添加worktree非常简单：
```bash
mkdir another-tree
```

采用`git status`（Lv3）检验是否创建成功：
```bash
git --git-dir=the-repo.git/worktrees/another --work-tree=another-tree status
# On branch another
#
# No commits yet
#
# nothing to commit (create/copy files and use "git add" to track)
```

给小repo简化命令行调用方式完全相同：
```bash
echo "gitdir: $(pwd)/the-repo.git/worktrees/another" > another-tree/.git
```

为了在`git worktree list`（Lv3）中查看worktree，在repo中登记一下：
```bash
echo "$(pwd)/another-tree/.git" > the-repo.git/worktrees/another/gitdir
# 注意此处git-dir写大repo小repo都能得到一样的结果
git --git-dir=the-repo.git worktree list
# /root/the-repo.git  (bare)
# /root/another-tree  0000000 [another]
git --git-dir=the-repo.git/worktrees/another worktree list
# /root/the-repo.git  (bare)
# /root/another-tree  0000000 [another]
```

### Lv3

```bash
# 由于git worktree add会验证commit-ish是否有效，
# 所以在我们创造出任何一个有效引用之前，该命令无法正常运行
# git --git-dir=the-repo.git worktree add --no-checkout third-tree <commit-ish>
```

## 删除小repo

### Lv0

```bash
# 直接删掉repo
rm -rf the-repo.git/worktrees/another
# 这个其实可以不删，不过留着容易让人误会
rm -f another-tree/.git
```

### Lv3

```bash
# 删掉the-repo.git/worktrees/another/gitdir所指向的对象
rm -f another-tree/.git
# 主动让git检验各个worktree是否存在；
# 在发现the-repo.git/worktrees/another的worktree已经找不到了之后，
# 它会主动删掉对应的小repo（由于是小repo，所以基本不会损失什么数据）：
# 注意：此处填写小repo，甚至填写另一个小repo都是可以的
git --git-dir=the-repo.git worktree prune
```

## clone: 利用模板创建repo

（参见第5章）

### `git clone --mirror`

- Lv2
```bash
rm -rf copy.git
git init --bare copy.git
# Initialized empty Git repository in /root/copy.git/
tee -a ./copy.git/config <<EOF
[remote "origin"]
  url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
  fetch = +refs/*:refs/*
  mirror = true
EOF
# [remote "origin"]
#   url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
#   fetch = +refs/*:refs/*
#   mirror = true
git --git-dir=copy.git fetch origin refs/heads/master:refs/heads/master
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
git --git-dir=copy.git fetch origin refs/heads/dev:refs/heads/dev
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
git --git-dir=copy.git symbolic-ref HEAD refs/heads/master
```

- Lv3
```bash
rm -rf copy.git
git clone --mirror git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git copy.git
# Cloning into bare repository 'copy.git'...
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
```

### `git clone --bare`

与`--mirror`类似，但是四不像（没有config）

- Lv2
```bash
rm -rf copy.git
git init --bare copy.git
# Initialized empty Git repository in /root/copy.git/
tee -a ./copy.git/config <<EOF
[remote "origin"]
  url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
EOF
# [remote "origin"]
#   url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
git --git-dir=copy.git fetch origin refs/heads/master:refs/heads/master
git --git-dir=copy.git fetch origin refs/heads/dev:refs/heads/dev
git --git-dir=copy.git symbolic-ref HEAD refs/heads/master
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
```

- Lv3
```bash
rm -rf copy.git
git clone --bare git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git copy.git
# Cloning into bare repository 'copy.git'...
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
```

### `git clone`

- Lv2
```bash
rm -rf copy-wt
git init copy-wt
# Initialized empty Git repository in /root/copy-wt/.git/
tee -a ./copy-wt/.git/config <<EOF
[remote "origin"]
  url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
  fetch = +refs/*:refs/*
[branch "dev"]
  remote = origin
  merge = refs/heads/dev
EOF
# [remote "origin"]
#   url = git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git
#   fetch = +refs/*:refs/*
# [branch "dev"]
#   remote = origin
#   merge = refs/heads/dev
git --git-dir=copy-wt/.git fetch origin
git --git-dir=copy-wt/.git update-ref refs/heads/dev refs/remotes/origin/dev
git --git-dir=copy-wt/.git symbolic-ref HEAD refs/heads/dev
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
# fatal: refs/remotes/origin/dev: not a valid SHA1
# 如果指定了--no-checkout，省略这一行
git --git-dir=copy-wt/.git --work-tree=copy-wt checkout-index -fua
```

- Lv3
```bash
rm -rf copy-wt
git clone --branch dev git@github.com:b1f6c1c4/learn-git-the-super-hard-way.git copy-wt
# Cloning into 'copy-wt'...
# error: cannot run ssh: No such file or directory
# fatal: unable to fork
```

## 总结

（以下均为Lv3）
- 创建空repo
  - `git init --bare <repo>`
  - `git init --separate-git-dir <repo> <worktree>`
  - `git init <worktree>` - repo在`<worktree>/.git`
- 从模板创建
  - `git clone --mirror <url> <repo>`
  - `git clone --bare <url> <repo>`
  - `git clone [--no-checkout] [--branch <ref>] [--separate-git-dir <repo>] <url> <worktree>`
- “单”repo多worktree
  - `git worktree list`
  - `git worktree add`
  - `git worktree prune`

## 扩展阅读

[gitrepository-layout](https://git-scm.com/docs/gitrepository-layout)

