---
title: 更快的字符串复制方法
tags:
  - Algorithm
categories:
  - Technique
date: 2019-10-14 00:31:38
mathjax: true
---

今天遇到了一个看似 trival，但是深挖进去却很有趣的问题——如何更快地复制一个字符串。

<!--more-->

复制字符串问题，就是实现下面这个函数`repeatStr`。
这个函数接收`source`, `times`两个参数，返回重复`times`的`source`字符串，
如`repeatStr('abc', 2)`就应该返回`abcabc`。

```javascript
function repeatStr(source, times) {}
```

## 迭代实现

这个问题初看起来非常简单，我看到这个问题的第一眼就想到了迭代解法。
问题要求我重复多少次，我就循环多少次嘛。Easy Question。实现如下:

```javascript
function repeatStrByIteration(source, times) {
  let res = ''
  for (let i = 0; i < times; i++) {
    res += source
  }
  return res
}
```

一切完美。但是......它还可以优化吗？如果想让它跑得更快该怎么办？
（我：嗯？还有跑得更快的方法？）

## 二分复制

其实真的还有跑得更快的复制实现。但是我们需要转换一下解决问题的思路。

在复制中，费时间的操作是字符串之间的加法，因为`string`是不可变类型，
每一次改变它都需要创建一个新的`string`对象出来。
所以我们优化的方向是如何尽可能地减少两个字符串相加操作。

在迭代实现中，对于$times = n$，我们需要相加`n`次。
联想到二分查找，一个很自然的想法就是——我们可以将目标字符串*“对折”*起来相加，
比如，我们要重复 10 次`a`，可以通过`'aaaaa' + 'aaaaa'`得到最后的字符串，
这里需要 1 次加法，然后为了构造`aaaaa`，我们可以通过`aa+aaa`，需要 1 次加法，
`aa`可以分解为`a+a`，需要 1 次加法，最后构造`aaa`，可以分解为`a+aa`，需要 1 次加法。
可以看到，构造整个目标字符串`aaaaaaaaaa`所需要的加法次数，从 10 次下降到了 4 次。

可以通过二叉树的性质证明我们的算法的时间复杂度是$O(\log n)$的（二叉树的高度）。
实现如下：

```javascript
function repeatStrByBinaryJoin(source, times) {
  // 两个边界情况
  if (times === 0) {
    return ''
  } else if (times === 1) {
    return str
  } else {
    // 通过Math.floor(times/2)来对折，如果是偶数，floor(times/2) === times/2
    // 这样我们直接用加号拼起来，如果是奇数，times = floor(times / 2) + [floor(times/2) + 1]
    // 需要多拼接一个source字符串
    let res = binaryRepeatStr(source, Math.floor(times / 2))
    res += res
    if (times % 2 !== 0) {
      res += source
    }
    return res
  }
}
```

## 性能测试

我们在 node 环境下进行测试。我们使用两个函数执行复制一千万次`abc`，测试函数脚本如下：

```javascript
const { performance } = require('perf_hooks')

function measurePerf(func) {
  const startTime = performance.now()
  func()
  return performance.now() - startTime
}

function repeatByIteration(source, times) {
  let res = ''
  for (let i = 0; i < times; i++) {
    res += source
  }
  return res
}

function repeatByBinaryJoin(source, times) {
  if (times === 0) {
    return ''
  } else if (times === 1) {
    return source
  } else {
    let res = repeatByBinaryJoin(source, Math.floor(times / 2))
    res += res
    if (times % 2 !== 0) {
      res += source
    }
    return res
  }
}

const strToRepeat = 'abc'
const repeatTimes = 10000000

console.log(
  `repeat by iteration: ${measurePerf(() =>
    repeatByIteration(strToRepeat, repeatTimes)
  )}ms`
)

console.log(
  `repeat by binary join: ${measurePerf(() =>
    repeatByBinaryJoin(strToRepeat, repeatTimes)
  )}ms`
)
```

输出如下:

```bash
repeat by iteration: 2006.9261000156403ms
repeat by binary join: 0.3501009941101074ms
```

可以看到我们的二分复制版本相较于迭代版本从时间上来看性能有了巨大的提升。

## 总结

通过本文，我们使用二分技巧成功地将复制字符串这个工作的复杂度从$O(n)$降低到了$O(\log n)$。
