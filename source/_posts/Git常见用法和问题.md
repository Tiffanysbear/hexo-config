---
title: Git常见用法和问题
date: 2019-08-05 17:30:16
tags:  Computer Skills
categories: Computer Skills
---


> Git常见用法和问题
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

现在git已经成为各个大厂代码管理的基本工具了，相信对于常用的一些git操作指令已经很熟悉了，先讲一些常见的使用吧。
首先是了解下git的概念，工作区、暂存区、远程仓库。

![](https://github.com/Tiffanysbear/accumulation/raw/master/image/git-demo.png)



### 常见用法

1、将计划改动加入到缓存区
增加某个文件：`git add <filename>`
增加全部文件：`git add .`

2、提交commit
`git commit -m 'comment message'`

3、推送改动
`git push origin master`

4、分支相关操作
切分支：`git checkout -b <branch-name>`
切回master分支：`git checkout master`

5、合并更新
本地同步： `git pull`
合并：`git merge <branch>`
查看改动文件: `git diff <source_branch> <target_branch>`

6、标签
创建标签：`git tag 1.0.0 <commit_id>` , 其中commit_id是通过 `git log` 获取的。


### 问题及解决

1、从工作区撤销
当看到自己的改动并不是自己想要修改时，且从未加入过暂存区，使用 `git checkout -- <filename>`, 撤销自己的改动。

2、从暂存区撤销
如果自己已经操作过 `git add <filename>`，需要从暂存区撤销，使用 `git reset HEAD <file>...` 对文件进行撤销回工作区

3、出现冲突conflict
当执行 `git merge` 操作时，可能会出现冲突，通过去代码中将冲突部分修改之后，再进行提交即可。

<!-- more -->

### 稍微高级的用法

#### git rebase 变基

1、`git rebase`和`git merge`操作都是用来进行分支合并的，可以通过git rebase将一个分支变基到master分支。

`git fetch origin`
`git rebase origin/your_branch_name`

如果有冲突的话，就进行冲突修改的操作：
`git add -u`
`git rebase --continue`

或者
`git rebase --abort` 取消变基操作，并将分支切回git rebase 之前的状态。

或者
`git rebase --skip` 忽略该提交，这样有问题的提交所引入的变化就不会被添加到历史中。

最后进行push操作就行。



#### git stash 保存临时修改

工作时，有时候在某个分支上还在修改，并不想`commit`，但是就必须要切换分支；这时候，git不会让你切换分支，会提示你还有尚未保存的修改；这时候，使用 `git stash` 就会将其保存到未完成的修改栈中，暂存你的工作状态。

`git stash` 这样的话，工作目录就是干净的，就可以自由的切换分支；
`git stash save 'describe message'` 暂存并添加描述信息
`git stash clear` 清除所有的stash信息
`git stash list` 查看暂存的所有变更
`git stash pop` 将最近一次的暂存弹出

如果不小心将自己的git stash的内容全部删除的话，需要采用下面的步骤进行恢复：

1、`git fsck --lost-found` 首先可以通过这个命令，查找删除的记录

![](https://github.com/Tiffanysbear/accumulation/raw/master/image/git-2.png)


stash信息中相对的是，dangling commit相关的提交信息;

或者使用命令： `git log --graph --oneline --decorate  $( git fsck --no-reflog | awk '/dangling commit/ {print $3}' )`


![](https://github.com/Tiffanysbear/accumulation/raw/master/image/git-3.png)


图表化会看得更清楚些。

2、`git show <id>` 可以查看该id下面具体内容，可以看到进行改动的代码。

3、使用 `git merge <id>` 或者 `git stash apply <id>`  恢复删除的代码。

#### 或者使用另外一个办法：

* git reflog 可以查看所有分支的所有操作记录（包括已经被删除的 commit 记录和 reset 的操作);例如执行 git reset --hard HEAD~1，退回到上一个版本，用git log则是看不出来被删除的commitid，用git reflog则可以看到被删除的commitid。
* git log -g 找到最近一次提交的commit记录,并记下commit id
* git branch newbranch commit_id，生成一个newbranch新分支，切到newbranch分支


#### 克隆某条特定远程分支

`git remote add -t -f origin`


#### cherry-pick
如果只想将远程仓库中一个特定的提交合并到自己分支中，使用 `git cherry-pick` 选择指定的 SHA 值的提交，然后将其合并到当前分支中。


#### 应用来自于不相关的本地仓库补丁

如果需要将另一个不相关的本地仓库的提交补丁应用到当前仓库，使用：
`git --git-dir=/.git format-patch -k -1 --stdout  | git am -3 -k`


#### 忽略某个文件的变更
对于某些环境相关的配置文件，可能每次环境变化都不同，但是每次合并都需要修改；下面这个命令可以永久性告诉git不要管某个本地文件。

` git update-index --assume-unchanged `


#### 清理
Git提示 "untracked working tree files" 会 "overwritten by checkout"。需要通过清理来使工作树保持整洁。

```
git clean -f     # remove untracked files
git clean -fd    # remove untracked files/directories
git clean -nfd   # list all files/directories that would be removed

```

#### 将项目文件打成 tar 包，并且排除 .git 目录

使用tar、zip来打包项目文件时，需要剔除.git目录：

` tar cJf .tar.xz / --exclude-vcs `


#### 安装 git blame 相关插件

如果使用的是 VSCode 的话，可以安装 git blame 这个插件，可以找到每一行代码的作者。
















