---
title: 利用Haproxy进行SS代理中继
tags:
  - Haproxy
  - Proxy Relay
date: 2017-02-01 15:46:07
categories: Tutorial
---


在使用国外的代理服务器时，常常会因为各种不稳定因素导致在直连时连接的丢包率特别高。对于一些对丢包率特别敏感的游戏（没错，说的就是舰队Collection），使用丢包率高的线路进行游戏是一件非常痛苦的事情。有的时候，游戏限定某个地区的IP才能够接入（没错，说的还是舰队Collection），所以必须使用架设在那个地区的代理服务器A。这两项加起来就有点麻烦了。但是，如果我们能够找到一个中继服务器B，从本地直连到中继服务器B的丢包率非常低，而从中继服务器B到目标服务器A的丢包率也非常低，这样我们就能够形成一条丢包率较低的线路。这样一条线路有效地提升了那些丢包率敏感的游戏体验。

    本地<->中继服务器B<->代理服务器A

在这里，我们在中继服务器B上使用`Haproxy`将请求转发到目标服务器A上。注意`Haproxy`只支持TCP连接的转发。

<!--more-->

## 基本步骤
### 安装Haproxy
Debian/Ubuntu

    sudo apt-get -y install haproxy

### 配置Haproxy
使用文本编辑软件打开`/etc/haproxy/haproxy.conf`，注意提供管理员权限，用以下内容替换原来的内容：

    global
        ulimit-n  51200

    defaults
    	log	global
    	mode	tcp
    	option	dontlognull
            timeout connect 5000
            timeout client  50000
            timeout server  50000

    frontend ss-in
        bind *:relay_server_port
        default_backend ss-out

    backend ss-out
        server server1 proxy_server_ip:proxy_server_port maxconn 20480

指定一个端口为`relay_server_port`，如`8100`等，这个端口将会之后用于**本地与中继服务器B**的连接。将`proxy_server_ip`替换为被中继的**代理服务器A**的IP地址，`proxy_server_port`设置为被中继的**代理服务器B**的监听端口。

### 启动Haproxy
运行
    sudo /etc/init.d/haproxy start

### 设置本地客户端
命令行
    sslocal -s relay_server_ip -p relay_server_port -l local_port -k password -m encryption_method

其中`relay_server_ip`是中继服务器B的IP，`relay_server_port`是上文提到的中继服务器转发的端口，`password`与`encryption_method`分别为访问代理服务器的密码与加密方法，与之前的设置相同。

## 原理
`Haproxy`是一款HTTP/TCP负载均衡器，核心功能就是将前端的大流量请求，分流到后端的各个服务器中。原理与我们要实现的代理中继非常类似。`Haproxy`监听特定端口的请求，然后将这个请求转发到后台的某一台服务器的端口上。这里`Haproxy`监听的端口就是设置的中继服务器的端口，“后台服务器”就是我们原来的SS代理服务器。


## 参考资料
1. [Shadowsocks利用 HaProxy 实现中继(中转/端口转发)][81ad36de]
2. [使用 HAProxy 加速 Shadowsocks][d51f8f2c]

  [81ad36de]: https://doub.io/ss-jc29/ "Shadowsocks利用 HaProxy 实现中继(中转/端口转发)"
  [d51f8f2c]: http://kaywu.xyz/2016/06/19/Shadowsocks-HAProxy/ "使用 HAProxy 加速 Shadowsocks"

## 修改日志
2017/11/9 修正启动Haproxy的命令
