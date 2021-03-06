# 第8章：检索与查看历史

## 检查分支的commit

- Lv2

若要根据parent关系，列出一个引用的所有commit，只需使用`git rev-list <commit-ish>`。
加上`-v`可以列出更多内容。
然而这并不能满足日常工作需求：
没有语法高亮、没有具体修改内容、没有非线性关系、
没有自动分页、没有tag。
所以Lv2的`git rev-list`命令并没有什么实际意义。

- Lv3

Lv3的`git log`的基本语法是`git log <commit-ish> [-- <path>]`。
但是依然没有好到哪里去。
上古命令`git whatchanged`约等于`git log --raw`，也不能满足所有需求。
我们必须添加一些参数，最好封装成Lv4。
根据实际使用需求，分成4种情况：

### 查看当前分支的简要历史

- Lv4: `git lg`

```sh
git log --color --graph --pretty=tformat:'%Cred%h%Creset -%C(magenta)%d %Cgreen(%aI)%Creset %s %C(bold blue)%G?<%an>%Creset' --abbrev-commit
```

### 查看整个repo的简要历史

- Lv3: `git show-branch -a`

```bash
(git log --graph --pretty=tformat:'h -%d (%aI) %s <%an>' --abbrev-commit --all)
# * h - (HEAD -> br4) (2018-01-01T00:00:06+08:00) commit G <b1f6c1c4>
# * h - (2018-01-01T00:00:04+08:00) commit E <b1f6c1c4>
# * h - (2018-01-01T00:00:01+08:00) commit B <b1f6c1c4>
# * h - (2018-01-01T00:00:00+08:00) commit A <b1f6c1c4>
# * h - (br1) (2018-01-01T00:00:07+08:00) commit H <b1f6c1c4>
# | * h - (br3) (2018-01-01T00:00:05+08:00) commit F <b1f6c1c4>
# | * h - (2018-01-01T00:00:04+08:00) commit E <b1f6c1c4>
# | | * h - (br2) (2018-01-01T00:00:03+08:00) commit D <b1f6c1c4>
# | |/  
# |/|   
# * | h - (2018-01-01T00:00:02+08:00) commit C <b1f6c1c4>
# |/  
# * h - (2018-01-01T00:00:01+08:00) commit B <b1f6c1c4>
# * h - (2018-01-01T00:00:00+08:00) commit A <b1f6c1c4>
git show-branch -a
# ! [br1] commit H
#  ! [br2] commit D
#   ! [br3] commit F
#    * [br4] commit G
# ----
#    * [br4] commit G
#    * [br4^] commit E
#    * [br4~2] commit B
#    * [br4~3] commit A
# +    [br1] commit H
#   +  [br3] commit F
#   +  [br3^] commit E
#  +   [br2] commit D
# ++   [br1^] commit C
# +++  [br1~2] commit B
# +++  [br1~3] commit A
```

- Lv4: `git la`

```sh
git log --color --graph --pretty=tformat:'%Cred%h%Creset -%C(magenta)%d %Cgreen(%aI)%Creset %s %C(bold blue)%G?<%an>%Creset' --abbrev-commit --all
```

### 查看当前分支的历史文件修改摘要

- Lv4: `git ls`

```sh
git log --color --graph --pretty=tformat:'%Cred%h%Creset -%C(magenta)%d %Cgreen(%aI)%Creset %s %C(bold blue)%G?<%an>%Creset' --abbrev-commit --decorate --numstat
```

### 查看当前分支的历史文件修改详情

- Lv4: `git lf`

`git log -p`

### 统计commit

- Lv3: `git shortlog --all [-s]`

## 检查某个文件的历史

有两种视角：

### 分支视角：列出当前分支中和该文件有关的所有commit

- Lv4: `git lf [--follow] -- <path>`

添加`--follow`可以兼容文件重命名。

### 内容视角：对文件的每一行列出哪个commit修改了它

- Lv3: `git blame -n -- <path>`
换一种输出格式：`git annotate -- <path>`

## 寻找文件

### 在index中寻找

- Lv4: `git find`

`!git ls-files -- :/ | grep --color=always`

### 在index和所有submodule的index中寻找

参见第9章。

- Lv4: `git finds`

`!(git ls-files; git submodule foreach --recursive "git ls-files") | awk -F '/' "{ if (match(\$0, /^Entering '(.*)'\$/, ary)) { P=ary[1]; sep=\"/\" } else { printf \"%s%s%s\\n\",P,sep,\$0; } }" | grep --color=always`

## 搜索关键词

### 在worktree中搜索

注意：只能搜索在index中有对应项的文件。

- Lv1

`git ls-files -s -- <path> | awk '{ print $2; }' | xargs -0 grep [-i] [-w] [-P] <regex>`

- Lv3

`git grep [-i] [-w] [-P] <regex> -- <path>`

### 在index中搜索

- Lv1

`git ls-files -s -- <path> | awk '{ print $2; }' | xargs -n 1 bash -c 'git cat-file blob $1 | grep [-i] [-w] [-P] <regex>' ''`

- Lv3

`git grep --cached [-i] [-w] [-P] <regex> -- <path>`

### 在tree中搜索

- Lv1

`git ls-tree -r <tree-ish> -- <path> | awk '{ print $3; }' | xargs -r -n 1 bash -c 'git cat-file blob $1 | grep [-i] [-w] [-P] <regex>' ''`

- Lv3

`git grep [-i] [-w] [-P] <regex> <tree-ish> -- <path>`

### 在当前分支中搜索

注意：一般正常人都会先在HEAD中搜索，找不到了再尝试在当前分支中搜索。
因此我们可以认为关键词一定会在`git lf`中出现至少两次（一次添加一次删除）。

- Lv2: `git grep <regex> $(git rev-list HEAD)`
- Lv3: `git lf` 然后使用pager的搜索功能
- Lv3: `git log -G <regex>`

### 在整个repo的所有所有引用的最新commit中搜索

- Lv2: `git grep <regex> $(git rev-parse --all)`

### 在整个repo的所有历史中搜索

过于暴力，走投无路了再用这个：

- Lv2: `git grep <regex> $(git rev-list --all)`

### 在worktree和所有submodule的index中搜索

参见第9章。

- Lv4: `git greps [-i] [-w] [-P] <regex>`

`!bash -c '(git --no-pager grep --color=always "$@" -- :/; git submodule foreach --recursive "$(printf "%s || true" "$(printf "%q " git --no-pager greps --color=always "$@")" )") | less' ''`

## 总结

- 列出commit
  - Lv2
    - `git rev-list [-v] <commit-ish>`
  - Lv3
    - `git log` / `git whatchanged`
    - `git show-branch` - 另一种方式查看整个repo的历史
    - `git shortlog --all [-s]` - 统计commit
  - Lv4
    - `git lg` - HEAD的简要历史
    - `git la` - 整个repo的简要历史
    - `git ls` - HEAD的文件修改摘要
    - `git lf` - HEAD的文件修改详情
- 检查文件的历史
  - Lv3
    - `git blame -n -- <path>` - 对每一行找出最近一次修改它的commit
    - `git annotate -- <path>` - 同上，换一种输出格式
  - Lv4
    - `git lf [--follow] -- <path>` - 列出相关commit
- 寻找文件
  - Lv4
    - `git find`
    - `git finds`
- 搜索关键词
  - Lv3
    - `git grep [-i] [-w] [-P] <regex> -- <path>` - 在worktree中搜索
    - `git grep --cached [-i] [-w] [-P] <regex> -- <path>` - 在index中搜索
    - `git grep [-i] [-w] [-P] <regex> <tree-ish> -- <path>` - 在tree中搜索
    - `git log -G <regex>` - 在HEAD的历史中搜索
    - `git grep <regex> $(git rev-list --all)` - 在整个repo中搜索
  - Lv4
    - `git greps [-i] [-w] [-P] <regex>`
