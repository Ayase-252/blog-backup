---
title: "MATLAB:糟糕的程序语言"
categories: Technology
tags:
  - MATLAB
  - Programming Language
date: 2017-10-26 22:47:20
---

MATLAB 因为其“标准库”具有丰富的功能（包括内置的函数以及工具箱），能够很容易地实现工程中的很多算法，因此一直是学术界事实上的仿真软件的标准。关于他的便利性，有一次我甚至在不知道 FFT 的细节的情况下，依靠查阅文档在 30 分钟之内写出了一个使用 FFT 对信号进行频域分析的小程序，包含数据输入和数据可视化。包含超过一半的注释，代码不到 100 行。可以理解为什么在学术界以及工程界，MATLAB 能够得到近似必修课一样的待遇。

但是，在需要在 MATLAB 中构建一个稍微复杂一点的系统的时候呢？MATLAB 会迅速地变得糟糕起来。有时甚至需要非常反直觉的 trick 才能够实现一些编程中的 essential practice。一方面，MATLAB 本身语言上还是有极大的缺陷。一方面 MATLAB 文档只是 how to do 而缺少 why to do，使得我们很难去抓住高级用法的本质，更谈不上用到 MATLAB 设计者想象中预想的最佳实现。

<!--more-->

## 反直觉的函数以及类文件限制

如很多编程语言一样，函数是 MATLAB 中主要的代码重用的方法之一。但是反直觉的是，MATLAB 在一个`.m`文件中只允许暴露一个函数给外部使用。这样迫使使用者要么将可能非常复杂的功能实现在一个“主函数”中，而将其中的逻辑分散在其中不能被外部调用的“子函数”中；要么将所有的“子函数”写成单独`.m`文件。前者，能够复用的只是实现特定功能的主函数，而不能够复用子函数。设想如果我们有两个神经网络，它们可能用到同一种 activation function，如`ReLu`。那么为了复用`ReLu`，我们只能选择将`ReLu`单独抽出来作为`.m`文件。这无疑增加了要管理的文件数量，并且最后我们会发现这些`.m`文件并不在同一个逻辑抽象的层级如在同一个文件夹中，包含`cnn.m`，`rnn.m`，`relu.m`。这样就会非常令人困惑，加大维护难度。当然，通过恰当的将这些文件归类到文件夹中可以稍微减轻不适感。但是作为一门编程语言而言，这样的限制还是相当怪异的。

但是，比起类，函数的文件限制造成影响要温柔许多。是的，**一个`.m`文件只能包含一个类，而且是同名的类**。设想，JavaScript 中一个`.js`只能声明一个`class`（ES6）,Python 中一个`.py`文件只能声明一个`class`。再设想，C++中一个`.hpp`文件中只能包含一个 class 的声明。在所有宣称支持 OOP 的编程语言中，对类施加这种限制的语言恐怕只有 MATLAB 了。这样带来的后果是，就算两个类有非常密切的关系，如一个类是另外一个类的选项的 enumeration class，我们都必须将这些类分开为两个文件，而且根本没有任何的 workaround。这样一来，类文件的层级就会变得非常混乱。

## 缺少模块作用域或者命名空间

在许多现代的编程语言之中，为了在大型项目中避免命名冲突，或多或少都会引入模块作用域或者命名空间的概念。如 Python，一个`.py`文件就是一个 module。在其他文件中要使用到这个文件的一些函数或者类的时候，要显式地`import xxx`到另外的文件中。这样减少了命名冲突的可能。但是，MATLAB 却不然。所有在 MATLAB 的 Path 中的函数都在同一个作用域中——全局作用域。这些函数可能来自 MATLAB 的内置功能、安装的工具箱、第三方库以及自己编写的代码。有兴趣的话，在 Command Window 输入`is`然后再使用 tab 键自动补全，看看下拉菜单里这些函数名。根本不知道哪个函数来自于哪个功能。这个缺陷导致最严重的后果就是命名时冲突的可能性大大增加。由于 MATLAB 涵盖很多工程领域的一些算法，如果当我们不知道有这样一个算法或者想动手实现一个相同的算法时，命名相同的可能性就会非常的大。而在 MATLAB 中是允许后来的名字去覆盖前面的名字的，此时造成的 BUG 就会非常难以察觉。MATLAB 所提供的工具箱中有很多重名的函数，如`write`。在一次想实现 TCP socket 通信时，就出现了这样一个 BUG。`write(tcpclient, data)`是向建立好的 socket 写入数据的函数。但是，实际调用的时候，MATLAB 却提示参数错误。在一段非常令人懊恼的 DEBUG 过程之后，我终于发现出问题的`write`函数并不是来自于想要的工具箱。通过手动的调整 pathdef.m 文件才解决了这一 BUG。

![若干重复的write函数](/images/2017/10/20171025222801.png)

## 混乱的命名规则

一个项目必然要遵循一定的命名规则。如果所有的命名都是遵循一致的命名规则的时候，我们就可以通过命名规则去猜测实现某一功能的函数的名字。但是 MATLAB 再次令人失望了。在短短的以`is`开头能够搜索到的函数，我们就能够看到至少三种命名规则的存在：驼峰命名法（`is2dDataArray`）、下划线间隔的全小写命名（`is_simulink_handle`）以及不用下划线间隔的全小写命名（`ischar`）。当初学某种语言的时候，我们会模仿着其标准库去制定命名规则。但是在这里，我们能够选择什么命名规则呢？所有都可以吧。当第三方开发者再向这个本来就缺少隔离的，杂乱无比的全局作用域中添加函数时，简直就是一种灾难。这时，MATLAB 文档就凸显了其重要性。我们只能够对功能进行描述去 Google 出文档，才能够知道对应的函数是什么。而一门优秀的语言不是这样的。如 JavaScript 中对`Array`进行遍历操作。JavaScript 可以使用`Array.forEach(callback)`对一个数组进行遍历，我们绝对不会在写代码的时候怀疑这个函数到底是`foreach`、`for_each`、`ForEach`等等。

总而言之，混乱的命名规则进一步使得本来就不怎么干净的作用域变得混乱无比。加大了开发的难度。

## Vector? Array? Cell Array?

数组(array)是一种常见的数据结构，几乎在所有的程序中都会或多或少的用到它。无论是静态语言中，如 C++中的原生数组`int arr[10]`，或者是其标准库中的`Vector<t>`，或者是动态语言中，如 Python 中的 List `l = [1, 2, 3]`。它们的意义和操作都非常的清晰。当选择用什么东西去实现“数组”的时候，我们根本不会有什么其他的想法。但是，MATLAB 又给我们上了一课。

MATLAB 中存在像数组的对象不止一种，如`array`、`cell array`、`character array`等。第一种是矩阵的一个特殊形式——“一维矩阵”，在数学意义上叫做向量，为了避免歧义，第一种数组从现在开始用`vector`来表示。这种数组只能用于存储数。这很容易理解，毕竟它来自于 MATLAB 的杀手功能——矩阵运算。当我们仅仅需要存储数的时候，`vector`可以是一种选择。它具备在运行时调整大小的能力，是一个完全的“动态数组”的实现。当我们很开心地用上了`vector`却发现我们需要去实现一个二维动态数组的时候，问题就出现了。

如我们有两个数组`a = [1 2 3]`，`b = [3 2]`。`a`记录 A 端口的历史数据，`b`记录 B 端口的历史数据。由于不同步的原因，A 和 B 不会同时收到数据。而我们预测到之后有增加端口的需求，总不能够`c`、`d`、`e`这样的命名下去吧。我们很容易想到用一个以端口号索引的二维数组`history`，调用`history[1]`就可以获取第 1 个端口的历史数据。这时如果是各个端口历史记录个数不相同的情况下时，MATLAB 就会报出与维数相关的错误了。因为如果用到了二维数组，此时概念就变成了`matrice`（矩阵），它在数学意义上是一个$m \times n$的数字阵列。任何一行的数字个数不相等是不能够定义一个矩阵的。

此时`cell array`就会变成一个很好的选择。它和 Python 中的 List 非常的相像。具有动态调整大小的能力，而且不关心里面所存元素的数据类型。可以说，`cell array`才是最贴近作为数据结构的`array`的概念。所以，个人推荐在需要使用到数组这一数据结构的时候都使用`cell array`实现。

但是接下来麻烦才刚刚开始。在解决一些 BUG 的时候，会发现`cell array`居然会有两种访问的方式，小括号的下标运算符`[]`以及大括号的下标运算符`{}`，而且它们的作用截然不同。

    >> history={{1,2},{3,5,2}}

    history =

    1×2 cell array

      {1×2 cell}    {1×3 cell}

    >> history{1}

    ans =

    1×2 cell array

      [1]    [2]

    >> history(1)

    ans =

    cell

      {1×2 cell}

你没有看错，当你尝试去访问第一个端口的历史数据时，`history{1}`得到是一个`1x2 cell array`，而`history(1)`得到的是一个`cell`，它的元素是一个`1x2`的`cell array`。这意味与`history{1}`的等价写法是

    tempArray = history(1);
    tempArray{1}

那么可能有人会问`history(1){1}`不就等价了吗？不，MATLAB 会提示你`Error: ()-indexing must appear last in an index expression.` Index 这一类操作符还有顺序限制？这也是在写本文的时候才发现的问题。难道不是`history(1)`先得到一个`cell`，我们再用`{1}`去 index 第一个元素吗?想不通啊。这样就会出现一类 BUG，你以为得到了值，但是其实你并没有真正得到你想要的东西。这种 BUG 会非常令人困惑，如果不通过单步调试是找不出来的。

## 总结

不可否认 MATLAB 以及其工具箱是非常强大的。对于一些利用微分方程等就能够进行简单建模的任务，MATLAB 是极佳的选择。但是，当这门脱胎于数值计算任务的语言用于构建较为复杂的“软件”的时候，其自身的局限就会被放大，甚至造成非常不好的编程体验。这就是为什么我称它是糟糕的“编程语言”的原因。
