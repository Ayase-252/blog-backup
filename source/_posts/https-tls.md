---
title: 一文弄懂HTTPS（II）-TLS/SSL协议握手
tags:
  - Web Security
  - HTTPS
  - Computer Network
categories: Technology
mathjax: true
date: 2018-11-15 23:31:56
---


{% post_link https-cryto-basic 上一篇文章 %}，我们介绍了有关HTTPS的一些密码学的基础，介绍了对称加密、非对称加密算法以及Diffie Hellman密钥交换协议等。这一篇文章，我们会介绍如何建立一个HTTPS连接，更准确的说，如何通过SSL/TLS协议建立起安全的连接。

<!--more-->

## TLS/SSL协议

回到HTTPS为什么能够安全地传输数据这一问题。我们曾经回答过，数据在传输过程中被加密了。但是其实，HTTPS还是那个HTTP协议，只不过在传输层与HTTP之间加上了一层安全协议，TLS/SSL。HTTPS实质上是下面的架构。

```text
HTTP
----
SSL/TLS
----
TCP
----
...
```

那么，TLS/SSL又是什么东西呢？TLS（Transport Layer Security，传输层安全性协议）以及其前身SSL（Secure Sockets Layer，安全套接层）是一种安全协议，为其上层协议提供在不安全信道上的安全传输服务。TLS与其上层协议没有耦合，因此其上层协议不仅仅可以是HTTP，还可以是邮箱服务等。

TLS发展到现在已经到了TLS1.3版本，有兴趣的话可以查阅相关的历史沿革。

## TLS握手协议

我们常说的HTTPS握手的过程实质上是TLS握手协议执行的过程。TLS采用C/S模型，即客户端服务器模型，下面我们也会使用客户端、服务器这样的术语。

注意TLS协议握手与密钥交换协议的目的是一致的，都是为了客户端与服务器之间能够**共享一个密钥**。

下面我们来简介一下过程：

```text
Client                          Server
  | ----> (1) Client Hello ---->   |
  | <---- (2) Server Hello <----   |
  | <---- (3) Certificate  <----   |
  | <---- (4) Server Key Exchange  |
  | ----- (5) Server Hello Done    |
  | ----> (6) Client Key Exchange  |
  ---------------------------------- Part. 1
  | ----> (7) Change Cipher Spec   |
  | ----> (8) Finish       ---->   |
  | <---- (9) Change Cipher Spec   |
  | <---- (10) Finish      -----   |
  ---------------------------------- Part. 2
        Shakehand Finish
```

握手可以分为两个部分，第一部分是客户端和服务器之间协商建立其安全通信的过程，第二部分是客户端与服务器之间就协商建立好安全通信进行测试的过程。这里我们先讲第一个部分。

### Part.1 密钥交换与密钥协商

如前面密码学基础讲过，要实现通信双方的“悄悄话”，使用对称加密的方式，双方需要就密钥达成统一。那么，在现实中如何做到呢？这就要用到上面Part.1的握手过程了。接下来的内容，初次接触TLS的同学可能会很迷惑，因为会有很多新名词出现。不过不用担心，我会在结尾总结为什么会有这些名词的出现。好，那我们开始HTTPS探险的第一步！

#### Client Hello

如HTTP一样，TLS握手的第一步也是由客户端发起的，这一步向服务器表明想要建立安全连接的意愿，并且向服务器提供一些必要的信息，如接下来连接可以使用的密码套件（这里的密码指Cipher，是指一种用于**加密**及**解密**的算法，而不是我们通常意义上说的Password），以及用于生成主密钥的**客户端随机数1**。这一步报文携带的信息有：

| 字段                 | 用处                                                         |
| -------------------- | ------------------------------------------------------------ |
| `Version`            | 使用的TLS的版本号                                            |
| `Random`             | **客户端随机数1**，用于生成主密钥                            |
| `SessionId`          | 会话ID，用处在后面会讲                                       |
| `Cipher Suite`       | 客户端可以使用的密码套件，按优先级排序。一个密码套件包含**密钥交换协议**，**对称加密算法**，**消息认证码（MAC）算法**，**伪随机函数**。为什么是这四种后面会提到 |
| `Compression Method` | 数据压缩方法（由于存在安全漏洞，在TLS1.3中被禁用）           |
| `Extensions`         | 一些扩展信息                                                 |

当服务器收到客户端的Client Hello信息之后，服务器就要忙起来了。为了建立起TLS连接，服务器需要向客户端通知：（1）就TLS版本，密码套件达成一致；（2）交换密钥使用的非对称算法的公钥及证书；（3）密钥交换协议需要的参数。

### Server Hello

Server Hello信息主要表明了服务器在TLS版本，密码套件上的选择。同时服务器会向客户端发送**服务器随机数2**。在这一步，报文携带的信息有：

| 字段                 | 用处                                               |
| -------------------- | -------------------------------------------------- |
| `Version`            | 使用的TLS的版本号                                  |
| `Random`             | **服务端随机数2**，用于生成主密钥                  |
| `SessionId`          | 会话ID，用处后面会讲                               |
| `Cipher Suite`       | 服务器选择的密码套件，接下来的通信将会遵循这一套件 |
| `Compression Method` | 数据压缩方法（存在安全漏洞，在TLS1.3中被禁用）     |
| `Extensions`         | 一些扩展信息                                       |

### Certificate

然后，服务器需要做的是把证书传递给客户端。在前一篇文章中，我们讲到了D-H密钥交换中中间人攻击。为了防止中间人攻击，需要保证非对称加密的公钥确实来自通信的一方。因此，Web建立起了PKI（Public Key Infrastructure，公开密钥基础建设）。也许以后我们会详细说这个主题。现在可以先这么想：

在现实生活中有公证处这一个机构，负责认证提交文件的有效性，认证之后公证处会颁发一份公证书，表明文件是真实有效的。接下来，出于对公证处这一个机构的信任，我们会信任带有公证书的这一份文件的内容。

假设在网络世界中有公证处这一个机构（其实是有的，数字证书认证机构，Certificate Authority, CA），我们向CA申请认证我们的公私钥对，如果认证通过的话，公证处会向我们颁发一个证书（Certification）。出于对CA的信任，我们认为证书认证过的公钥是正确的。

其实两者说的是一回事，想想申请公证和申请证书都是需要钱的。。

以本网站的证书举例（使用了Cloudflare CDN加速），我们来看看证书里包含什么信息（用PC访问的可以直接点地址栏右面的锁标记）：
![Certification image 1](certificate-1.png)
![Certification image 2](certificate-2.png)
![Certification image 3](certificate-3.png)

这些字段的用处如下表所示：

| 字段         | 用处                                                         |
| ------------ | ------------------------------------------------------------ |
| 版本         | 数字证书符合的版本（[X.509](https://zh.wikipedia.org/wiki/X.509)） |
| 序列号       | 证书颁发机构给这份证书的序列号                               |
| 签名算法     | 对证书的重点内容进行非对称加密加密，生成签名使用的算法，用于验证证书确实由颁发者背书 |
| 签名哈希算法 | 对证书内容进行哈希，生成指纹使用的算法，用于验证证书是否完整无误 |
| 颁发者       | 证书的颁发机构                                               |
| 有效期       | 证书的有效期                                                 |
| 使用者       | 谁可以使用这份证书                                           |
| 公钥         | 这份证书认证的公钥以及非对称加密算法                         |
| 公钥参数     | 非对称加密算法采用的其他参数                                 |
| 指纹         | 对证书内容按签名哈希算法进行哈希得到的值                     |

值得注意的是，证书体系中有一条信任链。层级最高的为根CA，它们的证书已经被提前预置到世界上所有的电脑中，它们可以通过签发证书授权中间CA向其他客户颁发证书。如果服务器的证书由中间CA颁发，在这个阶段，服务器需要按照信任链的顺序向客户端传送从服务器证书到离CA最近的一个中间CA的证书。如信任链是`CA -> ICA1 -> ICA2 ->...-> ICA n -> Server Cert`，那么服务器需要传送`Server Cert, ICA n, ..., ICA 2 -> ICA 1`。然后浏览器通过预置在电脑上的根CA的证书验证整条信任链是否完好。

### Server Key Exchange

说了很多证书方面的东西，假设证书有效，现在客户端已经知道了如何使用非对称加密向服务器传送数据了。接下来，根据协定好的密钥交换协议，服务器**可能**需要向客户端传送**密钥交换协议**所需要的参数。这一步是**可选**的。以D-H密钥交换协议为例，密钥交换双方需要对$p$, $a$达成一致（不熟悉的可以看{% post_link https-cryto-basic 这里 %}），在这一步可以将$p$, $q$，以及$a^{k_2} \mod p$一起发送出去。

### Server Hello Done

这一个报文表示Server Hello整个过程的完成。客户端可以根据收到的信息计算Pre-master secret（预主密钥）。

### 计算预主密钥

预主密钥的生成方式与**密钥交换协议**有关，如果采用D-H密钥交换，客户端可以产生随机数$k_1$，然后计算$a^(k_1) \mod p$作为预主密钥，并用证书中的非对称加密方法及其公钥加密。

### Client Key Exchange

将上面加密好的预主密钥通过Client Key Exchange报文发送给服务器。服务器使用其私钥解密，就能够得到预主密钥。

自此第一部分密钥协商双方交互的部分就告一段落了。接下来，客户端与服务器会使用接收到的信息计算正式通信时**对称加密算法**的密钥（主密钥的计算）。

### 计算主密钥

我们先来统计一下在上面的交换中服务器和客户端就什么值达成了一致：
1. `ClientHello.random`：由客户端在`Client Hello`报文中发送给服务器，公开信息；
2. `ServerHello.random`：由服务器在`Server Hello`报文中发送给客户端，公开信息；
3. `Pre-master secret`：预主密钥，非公开信息；

计算主密钥的方式由RFC 5356规定：

```text
master_secret = PRF（pre_master_secret，“master secret”，ClientHello.random + ServerHello.random）[0..47];
```

其中，`PRF`是协议约定的**伪随机函数**，其中`pre_master_secret`是非公开信息，因此中间人无法知道主密钥的存在，但是通信双方是可以产生同一把主密钥的。

到此，可能会有同学认为我们艰苦的工作就告一段落了。但是事实上没有。我们还会根据主密钥产生四把密钥。分别为：

- 客户端写入加密密钥：客户端用来加密数据，服务器用来解密数据。
- 服务器写入加密密钥：服务器用来加密数据，客户端用来解密数据。
- 客户端写入MAC密钥：客户端用来创建MAC，服务器用来验证MAC。
- 服务器写入MAC密钥：服务器用来创建MAC，客户端用来验证MAC。

WHY？？？？

#### 为什么双方要使用不同的内容加密密钥？

由于在一些对称加密算法中，使用相同的密钥加密并不安全，这个[回答](https://crypto.stackexchange.com/questions/2878/separate-read-and-write-keys-in-tls-key-material)解释了在使用RC4加密算法时，使用相同密钥加密的情况下，中间人可以通过对双方流量的监听获得明文的例子。

#### MAC密钥又是什么？

TLS进行了复杂的加密，但是到目前为止，只保证了**中间人不能够窃听消息**，但是不能保证**中间人不能够篡改消息**。

MAC（Message Authentication Code，消息认证码）机制便提供了这样一个保证。消息验证码的原理其实和通信时使用的校验和差不多。在数据发送前，一方会将**明文**，使用密码套件的**消息认证码算法**计算好MAC，然后将其包含在加密数据中。另外一方接收到报文先将密文和MAC分开，对密文使用**对称加密算法**解密成明文，再用**消息认证码算法**计算MAC，如果MAC匹配的化就认为报文是完好的。

自此，我们已经生成好了密码套件中所有算法需要的参数，为了保证万无一失，通信双方需要进行一次通信测试。

## Part.2 通信测试

通信测试很简单，客户端先产生一个验证数据，用**对称加密算法**加密验证数据。首先通过`Change Cipher Spec`报文通知服务器接下来的报文会用协商好的密码套件加密，然后将加密好的验证数据通过`Finish`报文发送给服务器。服务器接受到验证数据，解密后用自己的密钥加密。先发送`Change Cipher Spec`报文通知客户端接下来的报文会用协商好的密码套件加密，然后将加密过的验证数据通过`Finish`报文回传给客户端。客户端解密后对比是否是原来的报文，是的话就意味着整个加密体系没有问题。握手过程结束。

接下来，通信双方转到通常模式，可以尽情地在不安全的空间说“悄悄话”了~

## TLS会话复用

（我一看现在已经写了5k多字了，有点累了。。）TLS握手的过程非常冗长，涉及的运算量也比较大，对服务器会造成很大的压力。那么有什么办法减少握手的次数呢？

在`Client Hello`报文以及`Server Hello`报文中有一个`sessionId`字段，不知道大家在读了那么多文字之后有没有印象。对于前端的同学，`session`应该是一个很熟悉的词，它可以帮助我们保存用户登录的状态。那么我们是否可以借用`session`的思路，把一个`session`所需的参数在服务器里存储起来，把对应的`sessionId`发送给客户端。双方约定`session`失效的时间。当客户端在约定的时间之内使用`sessionId`与服务器通信，因为双方都保存着加密必要的参数，因此不需要复杂的密钥协商及交换过程就可以通信。这样大大减轻了服务器在握手上的压力。

但是这又带来了另外一个问题，保存大量的`session`需要大量的硬盘空间。为了解决这一个问题出现了`Session Ticket`。服务器将加密的必要参数通过保密的安全密钥加密发送给客户端，客户端加密过的参数。要通信时，在`Client Hello`报文中将`Session Ticket`带上。如果服务器能够成功解密的话就可以跳过密钥协商环节直接通信。这样就不用在服务器上存储大量`session`的信息。（把资源负担留给用户，23333）。

## 密码套件

现在，我们来总结一下密码套件中这些东西的作用：
1. 密钥交换协议：规定通信双方如何交换通信时使用的对称加密算法的密钥。影响`Server Key Exchange`与`Client Key Exchange`阶段；
2. 对称加密算法：规定通信双方在通信时如何加密信息。影响通信测试以及正常通信；
3. 消息认证码算法：规定如何对内容进行哈希，生成MAC。用于保证信息在传输过程中不受篡改；
4. 伪随机算法：用于生成主密钥及相关密钥；

## 总结

TSL/SSL是一个在应用层之下，传输层之上保证应用层协议的数据能够通过不安全信道传输的安全协议。本文介绍了双方如何通过TSL握手建立安全通信的过程。

但是写道这，感觉还是有一些东西没有讲清楚，如证书验证的过程。这就留一个坑先吧，我们下期再见。


## 参考资料
1. [SSL/TLS协议详解(下)——TLS握手协议](https://xz.aliyun.com/t/2531)