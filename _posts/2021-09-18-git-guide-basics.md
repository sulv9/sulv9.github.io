---
title: Git指南-基础篇
date: 2021-09-18 14:14:43 +/-TTT
categories: [Tech, Git]
tags: [git]
---

# 为什么要用Git？

- 方便多人远程协作开发
- 教学相长，在review别人代码的同时也能被别人发现自己代码里的问题，提升代码水平
- 可以让你放心大胆地开发，随时可以回退版本或者新建分支

# Git常用指令指南

## **新建代码仓库**

- `git init`  初始化Git仓库

- `git clone [url]`  克隆远程仓库代码到本地 

## **Git config相关指令**

- `git config [--global] user.name "[name]"`

- `git config [--global] user.email "[email address]"`

分别配置提交代码时用户的姓名和电子邮箱，其中如果加上`--global`表示全局配置，不加则表示仅作用于当前仓库。

## **Git Branch相关指令**

- `git branch`  显示所有本地分支列表

- `git branch -r`  显示所有远程分支列表

- `git branch -a`  显示所有本地和远程分支列表

- `git branch -m [branch-name]`  修改当前分支的名字

- `git branch [branch-name]`  新建一个分支，且当前分支不会自动切换

- `git branch -d [branch-name]`  删除指定的本地分支

- `git branch -dr [remote/branch]`  删除指定的远程分支，远程仓库地址+'/'+远程分支名

- `git checkout [branch-name]`  切换到指定分支

- `git checkout -b [branch-name]`  创建并自动切换到指定分支

## 增加/删除与提交

- `git add .`  添加当前目录下的所有文件到缓存区
- `git rm [file-name]`  将文件从工作区中移除并重新放入暂存区

- `git mv [file-original] [file-renamed]`  改名文件，并将其放入暂存区
- `git commit -m"message"`  提交暂存区到仓库区
- `git commit -v`  显示工作区文件所有的diff信息

## 远程同步

- `git remote add [short-name] [url]`  添加一个远程仓库，[short-name]为别名，[url]为远程仓库地址
- `git pull origin [branch-name] --rebase`  拉取远程分支的最新代码，并执行变基操作。

- `git push [remote] [branch-name]`  推送代码至远程仓库的指定分支上，`[remote]`为远程仓库地址，`[branch-name]`为远程仓库的分支名，这里**需要远程仓库存在该分支**才能执行成功。

- `git push -u origin [branch-name]`  同样是推送当前分支代码到远程仓库的指定分支上，但`-u`指令指定了默认主机，以后可以直接使用`git push`进行推送，而且如果此时远程仓库不存在该分支会自动创建。

- `git push --set-upstream [remote] [name-of-your-branch-name]`  推送当前分支到远程仓库的相应分支，`[remote]`表示远程仓库地址，第二参数是你的分支名，该指令会自动在远程仓库创建一个新的分支。
- `git push --force-with-lease`  强行推送代码，但不会覆盖，操作安全。

## **查看信息**

- `git status`  显示有变更的文件

- `git log --oneline`  简洁查看当前版本的历史记录

- `git reflog`  显示当前分支的操作记录

- `git diff`  显示工作区与暂存区的差异

## 撤销

- `git reset --hard <hash>`  重置暂存区与工作区代码到指定的某次提交

- `git stash`  将工作区与暂存区的内容保存至堆栈中

- `git stash pop`  将堆栈中保存的内容移至当前分支

  git stash使用可以参考这个博客：[git stash详解_stone_yw的博客-CSDN博客_git stash](https://blog.csdn.net/stone_yw/article/details/80795669)

## Git tag相关指令

### 展示标签

- `git tag`  显示所有的标签
- `git show [tag-name]`  查看标签的具体信息

### 创建标签

- `git tag [tag-name] `  创建一个轻量级标签
- `git tag -a [tag-name] -m "message"`  创建一个完整的标签
- `git tag -a [tag-name] [commit-hash] -m "message"`  为指定的commit打上标签

### 远程推送

- `git push origin [tag-name]`  推送单个标签至远程仓库（注意，正常push时不会自动提交标签）
- `git push origin --tags`  推送全部标签至远程仓库

### 删除标签

- `git tag -d [tag-name]`  删除本地标签
- `git push [remote] :refs/tags/[tagname]`  删除远程分支标签

### 切换标签

- `git checkout [tag-name]`  切换HEAD指向目标tag对应的commit
- `git checkout -b [new-branch-name] [tag-name]`  从指定tag对应的commit切出一条新的分支

### Git tag vs branch

#### Same

标签和分支都是作用于一个commit的，它们都是用来方便帮助我们不用去记忆commit的hash值的。

#### Different

当你执行`git tag`时，相当于为当前最新的commit打上了标记，这个标记是刻在该commit上的，除非该commit被删除，否则标记不会被改变。但是`git branch`就不太一样，branch始终会记录你的最新提交，每当有新的commit提交时branch都会跟着变动。

> The difference between tags and branches are that **a branch always points to the top of a development line and will change when a new commit is pushed whereas a tag will not change**. Thus tags are more useful to "tag" a specific version and the tag will then always stay on that version and usually not be changed.
>
> 标签和分支之间的区别在于，分支始终指向开发线的顶部，并且会在推送新提交时更改，而标签不会更改。因此，标签对于“标记”特定版本更有用，并且标签将始终保留在该版本上并且通常不会更改。
>
> (摘自[wikibooks-git](https://en.wikibooks.org/wiki/Git/Advanced#Tags_vs_Branches))

#### Practice

假如项目上线发布了v1.0版本，然后你负责继续开发之后的版本。在你将v1.1版本的代码写完后打算开始v1.2版本的开发，但是当你把v1.2版本开发到一半时突然v1.1版本被发现有bug，并且此时v1.1版本需要上线而v1.2版本的新功能暂时不上线，那么你就可以先在当前代码处打上一个v1.2.1标签，然后checkout回v1.1标签版本修复bug，修复完成后推送至远端分支，并在切换回v1.2.1版本后拉取最新分支。

有些人可能会问，你这么麻烦还不如直接新建v1.1和v1.2两个分支进行开发得了，但是一个项目的版本可能会非常多，例如v1.1.2-rc.3，v2.1.3-a.1等等非常多的版本，每个版本都新建一个分支来保存状态的话岂不是太多太乱了。

其实`tag`就是commit的简易别名，正常情况下假如你要切换到某次commit的代码状态时需要它的hash值，但打上标签后你只需要使用标签名即可方便切换。

参考文章：

[常用 Git 命令清单 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

[Git - 打标签 (git-scm.com)](https://git-scm.com/book/zh/v2/Git-基础-打标签)

[Git/Advanced - Wikibooks, open books for an open world](https://en.wikibooks.org/wiki/Git/Advanced)

# Git开发规范指南

## Commit message格式

格式：`git commit -m "[type]Write your messages here."`

其中`[type]`有下面几种类型：

```
feat：新功能（feature）
fix：修补bug
optimize：优化
style： 格式（不影响代码运行的变动）
refactor：重构（即不是新增功能，也不是修改bug的代码变动）
```

## Github开发规范

- 不能直接在`main/develop`分支上进行开发
- 在提交PR时尽量选择第三个选项`Rebase and merge`，提完PR后将自己的远程分支删除
- 每次更新一个小版本之后要记得打上tag
