---
title: 简述Python中的Decorator
categories: "Techno"
tags:
  - Python
  - Decorator
---
在使用很多库时，包括Python的标准库时，有时会遇到这样的写法。以Python的`unittest`标准库
为例（代码中`import`语句均省略）：

    class AUnittest(Testcase):
        @skip('test is disabled.')
        def test_a_skipped_unittest(self):
            #do test

上面的代码中，`skip`就是一个装饰器。装饰器使用一种特殊的语法与函数或者类结合，即
`@decorator(*args, **kwargs)`的方式。本例中，很显然，使用skip装饰器使得单元测试框架跳
过了`test_a_skipped_unittest`。
<!-- more -->

## 概述
Decorator（装饰器）分为两种。function decorator和class decorator。顾名思义，前者装饰
的是函数，而后者装饰的是类。

decorator实际上一个语法糖。其作用的实质是在运行时，Python自动进行被装饰的函数或者类的名
字与装饰器的绑定。decorator的产生背景与灵感可以参见[PEP 318](https://www.python.org/dev/peps/pep-0318/)。

所有的decorator本质上都是一个callable。这一点可以在下文中的代码等价形式中看出来。

## Function Decorator
function decorator是Python中最早的decorator的形式。它能够很好的解释decorator的机制，
所以首先考虑这一形式。用以下代码为例：

    @func_decorator(*fd_args)
    def function(*args):
      # do something

Python将会自动在被装饰函数的def语句之后，绑定装饰器与被绑定函数其等价的代码如下：

    def function(*args):
      # do something
    function = func_decorator(function, *fd_args)

可以看出，function的引用实际上指向的是`func_decorator(function, *fd_args)`的返回值。
如果我们用一个wrapper function去包装本来function，然后再将wrapper function返回出来。
这样我们就得到了一个“装饰过”的function。之后对function的调用，实际上是调用了一个
`func_decorator`返回出来的一个新的callable。值得注意的是，decorator在整个执行过程中只
调用了**一次**，即在def语句之后，进行绑定时调用decorator。

用一个比较实际的例子，考虑下面的代码：

    def print_decorator(function):
      # define a wrapper to function decorated.
      def function_wrapper(*args):
        print('function is called.')
        function(*args)
      return function_wrapper

    @print_decorator
    def add(x, y):
      print(x + y)

    add(1, 2) # print function is called
              # then print 3

上述执行过程实际上是`add`函数被自动绑定到了`function_wrapper(*args)`，最后的`add(1, 2)`
实际执行的是：

    function_wrapper(*args):
      print('function is called.')
      add(*args)

可以看到，这里的`print_decorator`给原有的`add`函数增添了一个每当`add`调用，在terminal
里打出一行`function is called.`的功能。这与logger的功能很像，是一个很普遍的需求。

总结起来，此处的function decorator拦截了直接的函数调用，允许在decorator中对被装饰的函数
进行一些预先的处理，如在调用前打一行log等。值得注意的是，decorator本身是一个callable
object。
