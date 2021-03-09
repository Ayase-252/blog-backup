---
title: 基于文件夹的个性化 git 配置方法
tags:
  - git
categories:
  - 通用技术
date: 2021-03-09 20:44:44
---


在一些时候，我们希望 git 有不同的配置。
比如自己的开源或者私人项目用一套 git 配置，其中 `user.name` 是 `xyz`，
`user.email` 是 `xyz@abc.com`
公司项目用另外一套 git 配置，`user.name` 是 `Real Name`、
`user.email` 是 `realname@corp.com`。

`git config` 支持系统层级 `--system`、
用户层级 `--global` 与仓库层级（无选项）的配置。但是，
对于大量项目，手动地通过 `git config` 指定未免过于繁琐。
本文介绍了一种通过修改 git 的配置文件 `.gitconfig`，使用
[`[includeIf]`](https://git-scm.com/docs/git-config#_conditional_includes)
对**某个文件夹**下的所有 git 项目指定 git 配置的方法。

<!--more-->

## git 配置文件层级

在 git 中，有三个层级的配置文件：

- 系统层级: `/etc/gitconfig`，作用于系统中所有用户的 git 配置；
- 用户层级: `$HOME/.gitconfig`，作用于用户的 git 配置；
- 项目层级: `.git/config`，作用于项目中。

如果有相同的配置，按照 项目 > 用户 > 系统 的优先级获取配置。

## `[includeIf]`

从 git 2.13.0 开始，git 配置文件开始支持
[Conditional Includes](https://git-scm.com/docs/git-config#_conditional_includes)
的配置。通过设置 `includeIf.<condition>.path`，可以向命中 `condition` 的
git 仓库引入 `path` 指向的一个 git 配置文件中配置。

`[includeIf]` 的语法如下，`<keyword>` 为关键词，`<data>` 是与关键词关联的数据，
具体意义由关键词决定。

```gitconfig
[includeIf "<keyword>:<data>"]
path = path/to/gitconfig
```

其中支持的 keyword 有：

- `gitdir`: 其中 `<data>` 是一个 [glob pattern](https://en.wikipedia.org/wiki/Glob_(programming))
如果代码仓库的`.git`目录匹配 `<data>` 指定的 glob pattern，那么条件命中；
- `gitdir/i`：`gitdir`的大小写不敏感版本。
- `onbranch`：其中 `<data>` 是匹配分支名的一个glob pattern。
  假如代码仓库中分支名匹配 `<data>`，那么条件命中。

就我们的需求，使用 `gitdir` 完全可以。

## 例子

假设在家用工作电脑上，我们默认开发的是个人项目。有时为了应对紧急需求，
会将公司项目 clone 到电脑中，统一放置放到 `~/corp-projects/` 目录下面。
个人项目与公司项目的差异点在：第一、使用的邮箱名不同，
个人项目会使用个人邮箱，公司项目使用公司邮箱；第二，
公司项目可能需要 VPN 接入才能够存取代码库。
我们首选使用，用户层级的 git 配置文件。

```bash
vim ~/.gitconfig
```

在最后添加一个 conditional include:

```gitconfig
# ~/corp-projects/ 下面的所有仓库引入 `~/crop-projects/.gitconfig` 中的配置
[includeIf "gitdir:~/corp-projects/"]
path = ~/corp-projects/.gitconfig
```

最后创建公司项目统一的配置文件：

```bash
vim ~/corp-projects/.gitconfig
```

```gitconfig
[user]
name = <Your Name>
email = <Your Email>

[http]
# 代理地址，如果公司项目需要代理才能够存取，填写此项；如果不需要，则不用这一行
proxy = <Proxy URL>
```
