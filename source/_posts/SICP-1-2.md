---
title: SICP 1.2节读书笔记
date: 2016-10-19 20:41:33
categories: "Note"
tags:
  - SICP
---
Procedure and Process
<!--more-->

## 1.2.1 线性递归处理（Linear Recursion Process）与迭代处理（Iteration Process）
### 线性递归处理
递归处理(Recursive Process)是指由一些推迟的操作（Defered operation）构成的处理链。
这些操作的结果无法依赖现有的信息得到，只能推迟到信息充足时才能够从递归的内部往外计算。如，
计算阶乘时采用以下迭代式：
\begin{equation}
n!=n(n-1)!
\end{equation}

在Scheme中定义函数如下：

``` lisp
(def (factorial x)
     (if (= x 1)
         1
         (* x (factorial (- x 1)))
      )
)
```

如果要执行`(factorial 4)`，它的执行情况如下：
```lisp
(* 4 (factorial 3))
(* 4 (* 3 (factorial 2)))
(* 4 (* 3 (* 2 (factorial 1))))
(* 4 (* 3 (* 2 1)))
(* 4 (* 3 2))
(* 4 6)
24
```

在得到结果的过程中，可以看到在头3步的递归调用。在这个过程中，解释器在后台自动记录了整
个计算过程。在通常的编程语言中，这个计算过程的中间变量将会被记录在栈上，这一片内存由系统自
动管理。所需要内存量随调用深度线性增长。所以这个处理过程又被称为**线性递归处理**。

### 迭代处理
迭代处理（Iterative Process）是通过*操作所有的状态*，逐步迭代得到最终结果。使用迭代
处理需要能够找出完成这一函数的所有状态。以计算加法为例：
\begin{equation}
c=a+b=(a-1)+(b+1)=\dots=(a-a)+(b+a)=0+b+a
\end{equation}

在Scheme中定义函数如下：
```lisp
(define (dec x) (- x 1))
(define (inc x) (+ x 1))
(define (+ a b)
     (if (= a 0)
         b
         (+ (dec a) (inc b))
     )
)
```

如果要执行`(+ 3 1)`，其执行过程如下：
```lisp
(+ 2 2)
(+ 3 1)
(+ 4 0)
4
```

显然在这种模式下，解释器不必要去保存中间变量，直接将最后的结果作为结果就行。理想中，这种
调用过程是不会产生额外的内存开销的。但是在通常的编程语言，如C中，系统要维护调用栈，即使
是这种写法，仍然会产生额外的内存开销。此时，可以通过将递归改写成由循环免去调用过程的代价。
这种技法叫*尾递归*。Scheme在解释器层面进行了尾递归优化，所以在上述执行过程中是没有额外内
存开销的。
