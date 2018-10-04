---
title: 设置Atom的包管理器——apm的代理
date: 2017-03-11 18:27:15
categories: Technology
tags:
  - Atom
---

Atom是一款非常流行的文本编辑器。借由庞大的插件支持，用户可以把它改造成适合自己的开发环境使用。但是由于众所周知的原因，在某地要更新插件并不容易。有时会刷不出来可更新插件的列表，即使能够刷出来，更新也可能因为各种网络原因而失败。解决这一问题，为Atom的更新服务挂上代理是一个不错的解决方案。
<!--more-->

## apm——Atom Package Manager
Atom的插件更新功能由apm提供。apm内部封装了一个npm。熟悉JS的同学会对npm非常的熟悉。apm更新插件时会先发送请求到atom.io/api/packages获取所有安装的包的最新信息。如果发现有更新的版本，apm会利用npm获取最新的包。所以，如果你刷不出来列表，多半是因为无法连接到atom.io。刷出来列表了，但是更新时出现网络错误，可能是因为无法连接到npm的源。

## 解决方案
### 为apm挂上代理
首先我们来解决第一个问题，访问atom.io不畅。这一点可以通过给apm挂上代理解决。apm内置了设置代理的功能，详细可以参见[apm - Atom Package Manager][cb459c6c]。简单而言，在命令行/终端窗口执行下列命令：

    apm config set https-proxy http://your-proxy-address:port

将上面的`your-proxy-address`替换为你的代理服务器地址，`port`替换为你的代理服务器端口即可。

执行下列命令，看`https-proxy`是否出现在输出中，可以证实设置是生效的：

    apm config list

网上有教程说要设置`http-proxy`的，通过`apm upgrade --verbose`查看请求之后可以看到请求并未经过代理。

#### 对于SS代理的解决方案
SS代理在这里是无法直接使用的。因为SS本身并不是一个http代理。所以，我们可以用polipo将SS代理转化为一个http代理。这一方面可以参见[SS的官方文档][a0834d6e]。

### 给apm换一个软件源
Atom的插件实际上在npm上，npm的官方源在国内访问起来是非常缓慢的。但是，国内有许多镜像源可以使用，如淘宝源(http://registry.npm.taobao.org/)，CNPM（http://r.cnpmjs.org）。

要设置apm使用的软件源很简单，执行下列命令：

    apm config set registry npm_mirror_url

将上面的`npm_mirror_url`替换为你想要使用的镜像源。如要使用淘宝源，即可以使用以下命令。

    apm config set registry http://registry.npm.taobao.org


至此，我们已经完成了整个更新流程的“优化”。atom应该可以正常更新了。

  [a0834d6e]: https://github.com/shadowsocks/shadowsocks/wiki/Convert-Shadowsocks-into-an-HTTP-proxy "Convert Shadowsocks into an HTTP proxy"
  [cb459c6c]: https://github.com/atom/apm#using-a-proxy "apm - Atom Package Manager"
