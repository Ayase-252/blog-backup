---
title: 重新学习正则表达式——从理论角度
tags:
  - Regular Expression
categories:
  - Computer Science
mathjax: true
date: 2019-10-16 01:07:23
---


正则表达式是一种描述*句法规则*的语言。在学习正则表达式的时候，
直接从某种程序语言的正则表达式 API 入手的我总感觉正则表达式非常复杂，
难以掌握。因此，我决定从另外一种角度重新审视一下正则表达式。

本文将会从正则表达式的数学原理出发，
看正则表达式如何仅仅从用 3 种基本运算就能够表示各种各样的句法规则。

<!--more-->

通过本文，你将会了解：

- 从数学角度重新定义正则表达式；

- 正则表达式的基本运算只有 3 种；

- 正则表达式如何通过这 3 种正则运算扩展；

- 如何构造一个复杂的正则表达式，如验证电子邮件地址的正则表达式。

本文假定读者有一些集合论的基础，特别需要了解集合的并的概念。

## 前言

经常，我们需要去提取一些符合一定规则的信息。例如，我们的手机号由 11 位数字组成，
11 位数字就是一个规则，或者说叫做模式（Pattern）。
我们可以很容易地理解 11 位数字是什么。但是，计算机却不会简单地理解这一概念。
因此，我们需要构造一种方法，最好是有明确规则的方法。使用这种方法，
让计算机知道我们所需要查找的字符串的模式，然后从原始字符串中把所需的信息提取出来。

美国数学家
[Stephen Cole Kleene](https://en.wikipedia.org/wiki/Stephen_Cole_Kleene)
教授就发明了这样一种方法——正则表达式。

## 前置概念

为了研究正则表达式，我们需要先定义一些前置概念：

1. _字母表_（Alphabet）：一个*符号*的有限集合。典型的字母表有像英文字母表 a-z，
   数字字母表 0-9。当然不限定是上述两类。任意符号的有限集合都是字母表。

2. _字符串_（String）：一个由从字母表中抽出的符号组成的有限序列，
   如 abc 就是定义在英文字母表上的字符串。我们用$|s|$表示一个字符串的长度，
   并且定义空字符串$|\epsilon|$是长度为 0 的字符串，即$|\epsilon| = 0$。

3. _语言_（Language）：某个字母表上字符串组成的
   [可数集](https://en.wikipedia.org/wiki/Countable_set)。
   如 $L=\\{0,1,2,3,4,5,6,7,8,9\\}$组成了一种语言，这种语言只包含位数为 1 的数字。
   定义只包含空字符串的语言为$\emptyset$。

可以看到，语言是一个字符串组成的集合，
我们的目的就是去找到一种方式描述这种语言中字符串的模式。

在开始正则表达式的探索之前，我们先定义一个字符串之间*拼接*的操作。这个操作很简单，
如果字符串$s$与字符串$t$进行拼接，会得到字符串$st$。假如$s=\text{cat}$、
$t=\text{house}$，那么$st=\text{cathouse}$。
为了更加简便地表示同一个字符串之间的拼接操作，这里定义一个类似数学上指数的操作，
令$s$为字符串，那么定义$s^n=\underbrace{s \ldots s}_{n个}$。
可以简单理解为$s$的$n$次重复。

## 基本运算

有了前置概念之后，我们将会定义 3 个语言之间的基本运算，有了这 3 个运算
，我们可以使用一些基本的语言来表达出更加高级的语言。
比如使用数字作为基本语言，表达出 11 位手机号。下面$L$，$M$均表示一种语言。
这三种运算分别是：

1. $L$与$M$的并：$L\cup M=\\{s|s \in L 或 s \in M\\}$

2. $L$与$M$的拼接：$LM = \\{st | s \in L 且 t \in M\\}$

3. $L$与$M$的 Kleene 闭包:$L^*=\cup_{i=0}^{\infty}L^i$

其中$\cup{i=0}^{\infty}L^i=L^0 \cup L^1 \cup \ldots$，$L^n = \underbrace{L \ldots L}_{n个}$。

通过语言之间的并操作，我们可以通过两个基本语言扩展成一个范围更大的语言。
如使用一个包含大写字母与小写字母的语言$L=\\{A,\ldots,Z,a, \ldots, z\\}$与
包含数字的语言$D=\\{0,\ldots,9\\}$，
通过并操作我们可以获得一个既包含大小写字母也包含数字的语言
$L\cup D = \\{A, \ldots, Z, a, \ldots, z,0,\ldots,9\\}$。

接下来是语言之间的拼接操作。从定义可以看到，拼接操作产生了类似
[笛卡尔积](https://en.wikipedia.org/wiki/Cartesian_product)的效果。
拼接操作可以极大地扩展语言。例如$LM$就产生了一个长度为 2 的字符串，
其中第 1 位是字母，第 2 位是数字。显然，字符串$\text{a1} \in LM$。

最后一种操作是 Kleene 闭包，这个操作可以将语言自身从 0 次重复（空语言$\emptyset$）
到无限次重复产生的所有语言并起来。如$D^\*=D^0 \cup D^1 \cup D^2, \ldots$，
$D^0$是一个只包含的空字符串的语言，$D^1$是一个只包含 1 位数字的语言，
$D^2$是一个只包含 2 位数字的语言，一直到$D^{\infty}$，
因此$D^\*$表示所有正整数加上空字符串。

### 通过基本运算来表示另外一种语言

有了上一节提到的 3 种运算，再定义一些基本语言，
我们就可以用这些运算来表示另外一种符合某种模式的语言。为了简单起见，
我们这一节还是使用前面定义的两种基本语言，
字母语言$L=\\{A,\ldots,Z,a, \ldots, z\\}$，数字语言$D=\\{0,\ldots,9\\}$。

现在我们想表示一个即包含字母也包含数字的语言，我们该怎么表示呢？
显然，我们可以使用$L\cup D$。为了简化表述，我们接下来用语言中字符串的模式来表示语言本身。

接下来，我们想要表示一个包含长度为 2 的仅包含字母的字符串，该怎么表示？显然，
使用一次拼接就可以$L^2=LL$。长度增加到 3 ？再加一次拼接，使用$L^3$就可以表示。
那么增加到$n$位？那么我们就拼接$n-1$次，使用$L^n$。最后，
我们想表示一个不固定长度仅包含字母的字符串（可以包含空字符串），使用 Kleene 闭包$D^*$即可
。使用这些技巧，我们可以表示$n$次重复的字符串。

最后，我们把这些运算结合起来，可以表示模式更加复杂的语言：

- $L(L \cup D)^3$：以字母开头的一个长度为 4 的字符串，后 3 位可以由数字与字母组成；

- $DD^*$：所有正整数（由于与一个$D$进行了拼接，不含空字符串）；

- $L^3 \cup L^4 \cup L^5$：由 3 到 5 个字母组成；

- $\emptyset \cup L^1$：由 0 到 1 个字母组成。

下面有一个小问题，如果要表达一个以两个字母开头，两个数字结尾的字符串？

如果对正则表达式的语法比较熟悉的同学可能已经发现了，
在正则表达式中对应上述几种常见模式的简写。

## 正则表达式

到这里，我们已经掌握了正则表达式背后最基本的数学原理了。没错，就那么简单。
接下来，为了进一步形式化我们上面用集合语言表达的想法，就得出了正则表达式，
它包含一套运算，以及一套优先级的定义，使得我们可以简化在大部分情况下需要的括号。

### 运算

基本运算还是上面提到的 3 种，但是变换了符号：

1. 并：$\(r\)|(s)=L\(r\) \cup L(s)$

2. 拼接：$\(r\)(s)=L\(r\)L(s)$

3. Kleene 闭包：$\(r\)^\*=(L\(r\))^\*$

4. $\(r\) = L\(r\)$

第 4 种运算的出现是为了让某种语言的表示与语言本身分开，
如我们可以使用`\d`代表一个数字语言$D$。这么的话就有`(\d) = L(\d) = D`

### 运算优先级

通过设定合适的优先级，可以免去大部分表示优先级的括号的需要，我们设定的优先级如下：

1. 一元运算符 Kleene 闭包$*$具有最高的优先级，具有左结合性；

2. 拼接运算符有第二高的优先级，具有左结合性；

3. 并操作符$|$具有最低的优先级，具有左结合性；

遵循上述运算优先级，我们可以把$(a)|((b)^*\(c\))$简化为$a|bc$。

到这里，我们已经完全掌握了如何使用正则表达式去表示某种模式。

## 实际编程语言中正则表达式的使用

为了便于使用，编程语言中的正则表达式对两个方面进行了扩展。第一，
为常用的模式定义了新的运算；第二，为一些常用的基本语言提供了简便表示方法。
下面以 JavaScript 为例：

新的运算包括：

1. 含空集的不定长度字符串$LL^*$，我们将它表示为$L^+$，也被称为正闭包（Positive Closure）。

2. 长度在一定范围内的字符串$L\\{m,n\\}=\cup_{i=m}^{n}L^i$

3. 可选字符串，空字符串或者长度为 1 的字符串$L?=\emptyset | L$

新的常见基本语言表示方法：

1. 通配`.`，任意除了`\n`、`\r`、`\u2028`（[LINE SEPARATOR](https://codepoints.net/U+2028)）、
   `\u2029`([PARAGRAPH SEPARATOR](https://codepoints.net/U+2029))之外的字符。

2. 字符集`[abc]`：`[abc]`是$a|b|c$的简便写法。在字符集中通过`-`字符可以指定一个字符范围。

3. 数字`\d`：`[0-9]`

4. 数字、大小写以及下划线`\w`：`[a-zA-Z0-9_]`

5. 取反`[^abc]`

## Demo：验证电子邮件地址

作为练习，我们用正则表达式来符合 RFC-5321 规定的电子邮件地址，进而验证一个地址是否符合标准。

根据[RFC-5321(Simple Mail Transfer Protocol)](https://tools.ietf.org/html/rfc5321)：
合法的电子邮件地址的格式为`<local-part>@<domain>`，这里使用`<name-regexp>`
表示一个子正则表达式，定义如下：

```text
# local-part可以是两种字符串<Dot-String>或者<Quoted-String>之一
<local-part> = <Dot-string>|<Quoted-string>

# Dot-String必须由<Atom>开头，可以用.进行分隔，但是.不能是最后一位
# 如user.name.就不行
<Dot-string> = <Atom>(<Atom>\.)*

# Auto是长度为1以上由<atext>组成的字符串
<Atom> = <atext>{1,}
<atext> = [a-zA-Z0-9!#$%&'*+-/=?^_`{}\|~]

# Quoted-string必须由两个引号"括起来，其中的内容是<QcontentSMTP>*，如"hello world"
<Quoted-string> = <DQUOTE><QcontentSMTP>*<DQUOTE>

# QContentSTMP可以是两种字符串之一，<qtextSTMP>或者<quoted-pairSTMP>
<QContentSTMP> = <qtextSTMP>|<quoted-pairSTMP>

# quoted-pairSTMP是斜杠加上斜杠加任何可显示ASCII字符，如/r、/n、/i等
<quoted-pairSTMP> = /[\x20-\x7e]

# qtextSTMP是除了引号"、反斜杠\之外的所有可显示字符
<qtextSTMP> = [\x20-\x21\x23-\x5b\x5d-\x7e]

<DQUOTE> = "

# domain必须由<sub-domain>开头，允许后面加.<sub-domain>，如github.com
<domain> = <sub-domain>(\.<sub-domain>)*

# sub-domain必须有一个<Let-dig>开头，后面接一个<Ldh-str>
<sub-domain> = <Let-dig><Ldh-str>?

<Let-dig> = [a-zA-Z0-9]

# Ldh-str是一个不能用连字号-结尾的字符串，综合来说意味着sub-domain可以
# 包含连字符但是不能是开头和结尾的位置。
<Ldh-str> = [a-zA-Z0-9-]*<Let-dig>
```

按照标准，我们可以逐步从子正则表达式入手，逐步地拼接成更加高级的子表达式，
直至形成最终的表达式。
由于这个正则表达式非常复杂，我们使用模板字符串来一步一步的拼接形成最后的表达式，代码如下:

```javascript
const atext = `[a-zA-Z0-9!#$%&'*+-/=?^_\`{}\|~]`
const atom = `${atext}{1,}`
// 由于运算优先级的原因，添加给atom一个non-capture group，下面也有很多同样的用法
const dotString = `${atom}(?:${atom}\\.)*`

const dQuote = '"'
const qtextSTMP = `[\\x20-\\x21\\x23-\\x5b\\x5d-\\x7e]`
const quotedPairSTMP = `\\/[\\x20-\\x7e]`
const qcontentSMTP = `(?:${qtextSTMP})|(?:${quotedPairSTMP})`
const quotedString = `${dQuote}(?:${qcontentSMTP})*${dQuote}`
const localPart = `(?:${dotString})|(?:${quotedString})`

const letDig = `[a-zA-Z0-9]`
const ldhStr = `[a-zA-Z0-9-]*${letDig}`
const subDomain = `${letDig}(?:${ldhStr})?`
const domain = `(?:${subDomain})(?:\\.${subDomain})*`

const verifyEmail = emailAddr => {
  // 因为正则太复杂，因此使用构造函数形式，以便使用模板字符串
  return new RegExp(`(?:${localPart})@(?:${domain})`).test(emailAddr)
}

console.log(verifyEmail('@h-2.com')) // false
console.log(verifyEmail('a_user@h-2.com')) // true
console.log(verifyEmail('a_user@.com')) // false
console.log(verifyEmail('a_user@com')) //true
console.log(verifyEmail('a_user@')) // true
console.log(verifyEmail('"quoted user"@ayase.moe')) // true
console.log(verifyEmail('"quoted_with_blackslash/uuser"@ayase.moe')) // true
```

现在我们已经写好了一个可能是正则表达式中最难的一个——验证邮箱地址的正则。

## 总结

在本文中，我们从几个基本概念——字母表、字符串以及语言出发，
介绍了构成正则表达式的 3 种基本操作——并、拼接与 Kleene 闭包。
我们使用了这 3 种基本操作，从字母语言与数字语言出发，
表达了一些具有复杂模式的语言。

通过重新定义基本运算以及运算优先级正式化了正则表达式，
并且探讨了正则表达式的一些扩展。最后我们使用了 JS 的正则表达式，
实现了一个符合 RFC-5321 的电子邮件地址验证器。
