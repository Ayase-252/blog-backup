---
title: 前端自动化测试：Why and How
categories:
  - Development
tags:
  - Testing Automation
---

在规范的软件开发流程中，对于软件的测试是必不可少的。
对软件的测试贯穿于软件的开发阶段以及验收阶段。在开发阶段，
工程师们会分析需求，设计软件架构并且在开发过程中针对每一个模块编写*单元测试*，
对模块组成的系统编写*集成测试*。在软件验收阶段，
测试工程师们可能会执行全系统范围的测试。

## 为什么不要手动测试

广义而言，测试也包含手动测试（manual testing），
如在开发中手动打开某个页面验证功能是否正常。但是在本文中，
测试这个概念指的是自动化测试（automated testing），
即可以通过命令自动执行的测试。

手动测试在软件开发过程中是不可靠的。有以下几点理由：

首先，手动测试的效率很差。
例如针对一个页面进行功能测试。当对代码进行更改，
编译器需要重新编译模块，
最好情况下会通过模块热替换（Hot Module Replacement, HMR）替换掉浏览器中对应的模块，
不太好的情况下，需要我们手动地刷新页面。视项目大小，这两个操作的时间数量级在几秒到几十秒之间。
之后，我们会手动的与页面进行交互，检查效果是否符合需求。视需求的复杂程度，
检查的时间数量级大概也在几秒到几十秒之间。将这些时间加起来，
手动测试**一个**测试用例的时间数量级在几秒到几分钟。
（如果工程师精力不集中，所需要的时间会更多）。
而如果测试是自动化的，也许测试过程中也需要重新编译模块，时间数量级在几秒到几十秒。
但是，针对每一个测试用例，验证的时间可能只需要几十毫秒。这样，
在几个测试用例的数量级下，自动化测试所需要的时间会显著低于手动测试的时间，

其次，手动测试的可重复性很差。有人会说，重复一遍手动测试的流程不就可以了吗？
但是，_人可能是可靠性中最弱的一环_。人的记忆力是很差的，
除了事先制定严格的测试程序，并以检查单的形式把测试程序固化下来，
人是无法精确地重复之前每一项手动测试的。显然，
我们不会在开发过程中制定严格的测试程序，我们只会关注于手头上正在开发的功能点，
并且将 99%的注意力放在验证当前的功能点上。其他的功能可能因为当前的功能点失效，
但是很有可能无法及时发现而形成新引入的 bug。

最后，手动测试覆盖的范围有限。由于手动测试效率低，换而言之手动测试的成本很高，
对于需要在多平台运行的程序，特别是 Web 应用，手动测试往往只能覆盖几个典型的平台。
自动化测试成本很低，而且在需要的情况下可以利用多台机器并行运行进一步提高效率，
可以在很短时间之内在多个平台进行验证。当然，这需要测试平台对自动化测试框架的支持。

因此，手动测试只能保证**当前需求在几个特定平台上的正确性**。当软件不断进化，
功能不断发生变更，手动测试无法保证**需求在整个软件周期内的正确性**。