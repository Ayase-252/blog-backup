---
title: 如何拯救坏掉的Git Repository——记一次git“事故”
categories:
  - Programming
tags:
  - Git
  - 版本控制
---

1 月的北京风和日丽。我做好了一个需求，估摸着该提 PR 合并到主仓库了。
于是，我准备和主仓库`master`分支`rebase`一下，然后整理一下 commit 记录。临近下班了，
又要面对一天中最困难的问题——晚上吃什么？楼下那家 KFC 的菜单已经吃完一轮了。
突然之间，我隐隐感觉有一点不太对劲。一看`git log`。Oh shit!!

<!--more-->

## 我 TM rebase 反了

我惊出了一身冷汗，瞬间没心思考虑晚饭问题了。看了一眼本地`master`分支，
最后一条 commit 记录的 SHA 和远程`master`最后一条 commit 的 SHA 不一样。诶？
远程仓库的 merge commit 全不见了。然后一看我刚刚敲的命令：

```bash
git checkout master
git pull upstream master
git rebase feature

git checkout feature
git rebase master
```

Oh, shit!我在干什么？我以前听说过搞乱 Git 仓库是很麻烦的问题。此时无数想法从脑中闪过：
难道 feature 要重写吗？好几百行啊，而且 feature 已经快到 deadline 了。我慌了，真的慌了。
还是先喝口水冷静下来吧。

冷静下来之后，我想我为了面试看过一点点 Git 的基本原理。从原理入手，
一点点小心地修正记录的话应该是可以修好的。此时最重要的是理解现在仓库的历史。Don't Panic。

## 分析 commit 历史

在这个项目中，生产代码单独是一个 repo。开发的时候，开发者`fork`生产代码的 repo，
然后`clone`自己`fork`的 repo。在本地，按照惯例，我自己`fork`的 repo 是`origin`，
远程生产代码的 repo 是`upstream`。开发新功能的时候，我从本地的`master`分支
`checkout`一个特性分支`feature`。在开发了几天之后，远程生产代码已经合并了数个 PR。
在我的愚蠢操作之前，整个项目的分支情况是下面这样的：

```plain
    C5---C6---C7---C8 feature
    /
  C1--C2--M2--C3--M3 upstream/master
  |
  origin/master
  master
```

其中`Mx`是 merge commit。在愚蠢的操作之后，由于`rebase`默认丢弃掉将要`rebase`分支的
merge commit，项目的分支情况变成了：

```plain
                        C5'--C6'--C7'--C8' feature
                      /
                C2'--C3' master
                /
origin/master C1--C2--M2--C3--M3 upstream/master
```

知道病因之后，似乎还有救的样子。首先，我们可以在`feature`分支中以`C1`为起点、
剔除掉`C2'`、`C3'`。这样，`feature`就恢复了之前的状态。然后，
我们将`master`分支回退到`origin/master`。这样，`master`分支也恢复了。
最后做一遍正常的`rebase`操作应该就行了。未来实现这个方案，
我们可以使用两把非常好用的“手术刀”——`reset`、还有`rebase`本身，在这个情况下可以使用。

## 手术过程

首先我们做第一步，以某个 commit 为起点，剔除掉一些 commit，
我们可以使用`git rebase -i start_commit_sha`命令。`-i`参数表示`interactive`。
执行之后，git 会调用一个文本编辑器打开一个文件，
里面的内容是从起点之后的第一个 commit 到最后一个 commit 的所有 commit。commit 可以被更改、
压缩(squash/fix up)、删除。要删除某个 commit，只需要在编辑器中把该行删掉即可。
保存退出之后，git 会根据这个文件的内容进行`rebase`操作。

删掉`C2'`、`C3'`之后，项目结构变成：

```plain
                C2'--C3' master
                /
origin/master C1--C2--M2--C3--M3 upstream/master
                \
                C5--C6--C7--C8 feature
```

然后，**在`master`分支**中使用`git reset --hard commit_sha`。
这个命令的语义是将*HEAD*（分支的指针）、工作目录、暂存区重置到`commit_sha`指向的 commit。
这里[官方文档](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E7%BD%AE%E6%8F%AD%E5%AF%86)解释得很清楚。

此时项目结构变为：

```plain
              master
              |
origin/master C1--C2--M2--C3--M3 upstream/master
                \
                C5--C6--C7--C8 feature
```

和没经过愚蠢操作的项目结构是一样的。此时只要来一遍正常操作就行了。

## 后记

解决 git 仓库被意外破坏的问题，最重要的是**要冷静**。只有冷静下来，
才能够准确地分析出现在 git 仓库的状况。然后看情况使用对应的工具操作。
git 其实内置了很强大的对 commit 历史进行操作的工具，所以大多数的 git 操作失误是不用重写的。

当然，最重要的是**操作 git 仓库的时候要专心**。敲命令真的很容易错的 orz。
