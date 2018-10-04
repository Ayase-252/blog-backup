---
title: 在Windows中配置兼容于VS Code的GPG签名服务
tags:
  - Visual Studio Code
  - GPG
  - Git
categories: Technology
date: 2018-05-04 15:36:29
---


在Git中，可以通过GPG加密的方式保证在commit传输的过程中不受篡改。在Windows中，可以用VS Code自带的版本控制提交带GPG签名的commit。但是一遇到对应的GPG密钥带有密码的时候，VS Code自带的版本控制（准确地来说是所调用的git程序）就无法对commit进行签名。具体会提示`gpg: cannot open tty 'no tty': No such file or directory`。原因在于`gpg`程序无法获取密码以进行签名。由于文档的不足，我尝试了很多方法去解决这一问题。最近看到[#17014](https://github.com/Microsoft/vscode/issues/17014)之后，突然了解了需要`gpg-agent`才能使GPG签名功能正常使用。本文将会描述如何在Windows中做出这些改变。
<!--more-->

## 准备工作
本文将不会描述如何使用`gpg`，如果想了解这一方面的信息的话，推荐阅读[GPG入门教程](http://www.ruanyifeng.com/blog/2013/07/gpg.html)或[git文档-签署工作](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E7%AD%BE%E7%BD%B2%E5%B7%A5%E4%BD%9C)
由于cygwin中的`gnupg`不自带`gpg-agent`，我们需要一个自带`gpg-agent`的`gpg`版本。在Windows平台下，流行的选择是[gpg4win](https://www.gpg4win.org/)。下载安装包，然后安装在任意目录之下。但是此时，签署功能仍然无法工作，一是因为`gpg-agent`未打开，二是因为`git`不知道新的`gpg`在哪，仍然会尝试使用旧的`gpg`导致问题。

## 配置
首先我们要在新的`gpg`下新建密钥，此部分请参见之前提到的教程，这里不在阐述。之后，在你最喜欢的命令行工具中执行

```shell
gpg-agent
```

以开启gpg-agent。

最后，我们要通知`git`新的`gpg`的位置，执行

```shell
git config --global gpg.program "your/path/to/gpg"
```

此时，签署功能应该可以正常使用了。
