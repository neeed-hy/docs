---
sidebar_position: 9
description: 'Loader 基础'
---

# Loader 基础

## 写在前面

如何扩展 Webpack？有两种主流方式：

- 第一种是 Loader，他主要的功能是将资源内容翻译成 Webpack 能够理解和处理的 JavaScript 代码；
- 第二种是 Plugin，他可以深度接入 Webpack 构建的过程，乃至于重塑构建的逻辑。

这其中，Loader 是职责更为单一、结构更为简单的一种。

为什么 Webpack 要设计出 Loader 这一扩展方式？本质上是因为实际上的资源文件格式太多，不可能一一穷举，因此 Webpack 选择将**解析资源**的能力开放出去，将**读文件**和**处理文件**逻辑解耦，Webpack 内部只进行 JavaScript 代码的处理。

这就是 Loader 的作用：对特定资源的解析。

实现上，Loader 通常是一种 mapping 函数形式，接收原始代码内容，返回翻译结果，类似于：

```js
module.exports = function (source) {
  // 执行各种代码计算
  return modifySource
}
```

在 Webpack 进入到构建阶段以后，首先会通过 IO 接口读取文件内容，之后调用 LoadRunner 将文件内容以 `source` 参数的形式传递给 Loader 数组，`source` 数据可能在 Loader 数组中经过多次形态转换，最终以 JavaScript 代码的形式提交给 Webpack 主流程，这就实现了特定格式文件=> JavaScript 代码的翻译过程。

Loader 中执行的各种资源内容转译操作通常都是 CPU 密集型 ，这对于单线程的 JavaScript 来说有不小的性能压力。为此，Webpack 默认会缓存 Loader 的执行结果直到资源或资源依赖发生变化。

## Loader 简单示例
