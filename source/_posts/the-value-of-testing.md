---
title: 测试的价值
tags:
  - Automated Testing
categories: Thinking
date: 2021-06-08 16:59:10
---


今天，我打算开一个新的栏目——[Thinking](/categories/Thinking)
来记录在生活中的一些思考。栏目中的文章主题不限于技术，
包含生活中的方方面面。（其实是因为长文太难写了，草稿箱里可能已经坑了20来篇了
（笑）。

那么栏目的第一篇，我打算聊一聊「自动化测试」。

<!--more-->

很多研发会觉得写测试用例是非常累赘的事情。有些会认为，
业务变化太快，维护测试用例费时不讨好；有些会认为，公司配备了完整的 QA 团队，
有问题让 QA 人工测试就好，没必要让研发写自动化测试等等；还有可能，
业务需求开发时间被压缩，觉得“额外”再写一份测试用例降低工作效率等等。

在日常工作中，至今为止接触的数十个项目中，
只有少数一两个项目在 CI 的要求下保证了一定的单元测试覆盖率（如果我没记错的话 50% ）。
其它项目基本上没有自动化测试用例的存在。毫不令人意外的，项目经常出现的现象有：

- 实现一个不相关的需求，导致线上其它不相关功能出现 bug；
- 为了不改动旧代码逻辑，大量复制粘贴代码，
  仔细一看，其中的实现逻辑大同小异，如就差一个判断等；
- 没人完全知道一个函数所有的输入输出的意义，类型与范围；
- 大量的全局变量；
- 大家都知道代码逻辑很烂，但是没有勇气重构去改良它们；
- 测试提出问题之后，需要在多处埋点 `console.log` 去寻找问题的根源在哪，
  耗费大量时间；

如果自动化测试是低效的，那么与之相反，没有自动化测试应该会体现出高效。
但是在实际中，没有自动化测试的项目中，反而也体现了低效与风险。
仔细思考的话，是否对于*自动化测试*的看法是有误的吗？是否自动化测试真的会对
项目的*开发效率*、*调试效率*、*可维护性*产生负面影响？🤔

## 自动化测试与开发效率

拒绝自动化测试的一大原因是开发*害怕*实施自动化测试会影响开发效率。毕竟，
研发除了业务代码之外，还要多维护一份测试用例代码。很多人下意识的会同意这种说法。

但是，实际上真的是这样吗？回想实际开发的过程，在完成一个需求的时候，
最频繁的操作会是什么？对于我而言会是*验证*操作。比如修改一处代码，
然后满怀希望的等待编译（可能耗时 10s 左右），然后在浏览器上模拟用户操作，
看程序的反应。这一套操作下来，个人估计至少需要 30s 左右。所以*人工验证*需要
30s 来一个功能点是否正常工作。人工验证是繁琐与无趣的，中间可能走神，
这样需要时间会更长。

如果换做用自动化测试的话，一个单元测试耗费的时间是毫秒级的，
一个 E2E 测试用例的时间大概在 1s 左右。在一次人工验证的时间中，
通过自动化测试我们能验证多少功能点呢？只跑 E2E 测试，
我们都至少能够验证 30 几个功能点。
况且 E2E 测试是自动化测试中最耗费时间的一类测试用例了，
自动化测试的效率对于人工验证的效率是数量级上的提升的。
另外，很多框架为了最小化繁琐操作，都自带了 `watch` 功能。
当检测到代码改变，自动运行相关的测试用例。基本上，
在修改代码后的**几秒**之内就能够知道代码修改**对整个项目的影响**。
这是人工验证无法做到的。

另外，自动化测试会使开发在整个研发的工作流中，不用在工具之间切来切去。
只用在代码编辑器与命令行中就可以知道修改代码的结果。
这会使开发更加专注于编写逻辑，不受“上下文”切换的影响。

自动化测试将开发中最频繁操作的效率提升了一个数量级。
付出的是仅仅几分钟时间设计一个测试用例。

## 自动化测试与调试效率

在调试中最难的问题莫过于找到问题的原因。如果没有测试用例的话，
我们可能会通过在有疑问的地方打日志的方式来找出问题。
对于一个较大型的程序而言，涉及到的代码可能横跨几个文件，好几个方法，
打日志的方法非常盲目，属于最后的手段。

但是如果有测试用例覆盖的话，DEBUG 会容易很多。
如果程序出现了问题，而同时某个测试用例没有通过。
可以直接怀疑相关的代码出现了问题。
修正代码使测试用例通过，bug 就修完了。
不再有到处打 `console.log` 来寻找问题在哪的操作。

即使，当所有测试用例都通过，程序仍然出现了 bug。这说明当前测试用例没有完全覆盖需·求。
此时，我们用 bug 的复现方法编写一个一定会失败的测试用例。
然后调整代码使这个测试用例通过。这样，不仅在调试过程中借助了自动化工具提升了体验，
而且更重要的是，测试用例可以保证这个 bug 永远不会再次出现。

## 自动化测试与可维护性

代码的可维护性与很多因素有关。如架构设计的合理性，模块之间是否存在耦合等等。
由于业务多变难以预测，几乎很少能够*一次性*的写出既符合需求又易于扩展的代码。
因此，在开发过程中，不断对代码进行[重构][]，
优化代码结构是提高项目可维护性的必经之路。

从定义而言，*重构*要求代码的改动不能改变软件的功能。然而，现实并不是乌托邦。
有时，代码几经接手，原需求已经不可考；有时，代码充斥着[坏味道][]，
我们已经无法理解代码的职责是什么。在这种情况下，
我们无法保证代码能够被安全重构。

重构对于项目健康而言是必要的，那么如何保证在安全的情况下重构呢？
——完整的测试用例。测试用例就像合同一样，保证了一个单元的输入输出的正确性。
单元的用户不关心单元内部的实现逻辑，只需要单元的输入输出符合预期即可。
如果一个单元有完整的测试用例覆盖的话，我们在单元内部任意调整代码结构，
只要能够保证测试通过，我们都能够认为重构是安全的。因此，Martin Fowler
在[《重构——改善既有代码的设计》](https://book.douban.com/subject/33400354/)
中提到了单元测试是重构的基础。

## 总结

自动化测试是提高软件设计质量与可靠性的良好手段。在现代的软件开发中，
软件代码与测试用例代码是同样重要的。在大型开源项目中，如 Node.js 中，
实现任何新特性或者修复 bug 都会被[**要求附上**](https://github.com/nodejs/node/blob/master/doc/guides/contributing/pull-requests.md#step-6-test)相应的测试用例。
但是在实际工作中，推行自动化测试也许遇到一些阻力。希望借由本文能够消除一些对单元测试的误解。

[重构]: https://en.wikipedia.org/wiki/Code_refactoring
[坏味道]: https://blog.codinghorror.com/code-smells/
