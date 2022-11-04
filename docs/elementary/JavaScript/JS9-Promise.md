# JS 基础--Promise 专题

[[toc]]

## Promise 凭借什么消灭了回调地狱

Promise 利用了三大技术手段来解决回调地狱：

- 回调函数延迟绑定
- 返回值穿透

  如果向 `then` 方法传递的参数不是一个函数，那么会将其解释为 `then(null)`,这个 `then` 方法不会对上一个 `then` 方法传过来的 promise 进行任何处理，直接传给下一个 `then` 方法。这就是**值穿透**。

  ```js
  var promise = new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("haha");
    }, 1000);
  });
  //  第一个then方法的参数不是函数，直接跳过
  promise.then("hehe").then((res) => {
    console.log(res);
  });
  //  haha
  ```

- 错误冒泡

  在链式调用 then 方法时，每个 then 都有可能会产生错误，分别处理会让代码变得非常臃肿。Promise 采用错误冒泡的处理方式，可以在链式调用的最后 catch 所有的错误，进行一站式处理，使代码变的简洁。

  ```js
  readFilePromise("1.json")
    .then((data) => {
      return readFilePromise("2.json");
    })
    .then((data) => {
      return readFilePromise("3.json");
    })
    .then((data) => {
      return readFilePromise("4.json");
    })
    .catch((err) => {
      // xxx
    });
  ```

## 为什么 Promise 要引入微任务

问题本身就是如何处理回调的问题。有三种方法：

- 使用同步回调，直到异步任务进行完，再进行后面的任务。
- 使用异步回调，将回调函数放在进行宏任务队列的队尾。
- 使用异步回调，将回调函数放到当前宏任务中的最后面。

第一种显然不可取，整个 JS 的执行都会被阻塞。

采用第二种方法，在宏任务队列很长的情况下，回调要等很久的队。

采用第三种方法，能够保证实时性。最为合适。

因此 Promise 利用了微任务。

## Promises/A+规范

具体规范可以阅读[Promises/A+规范]这篇文章。

整体文章比较长，但是核心规则在于：

> 1. Promise 本质是一个状态机，且状态只能为以下三种：Pending（等待态）、Fulfilled（执行态）、Rejected（拒绝态），状态的变更是单向的，只能从 Pending -> Fulfilled 或 Pending -> Rejected，状态变更不可逆
> 2. 一个 Promise 必须具有 then 方法。then 方法接收两个可选参数，分别对应状态改变时触发的回调。then 方法返回一个 promise。then 方法可以被同一个 promise 调用多次。

## 手写 Promise

首先搭建大体框架：

```js
const PENDING = "pending";
const RESOLVED = "resolved";
const REJECTED = "rejected";

function MyPromise(fn) {
  const that = this;
  that.state = PENDING;
  that.value = null;
  that.resolvedCallbacks = [];
  that.rejectedCallbacks = [];
  // 待完善 resolve 和 reject 函数
  // 待完善执行 fn 函数

  function resolve(value) {
    if (that.state === PENDING) {
      that.state = RESOLVED;
      that.value = value;
      that.resolvedCallbacks.map((cb) => cb(that.value));
    }
  }

  function reject(value) {
    if (that.state === PENDING) {
      that.state = REJECTED;
      that.value = value;
      that.rejectedCallbacks.map((cb) => cb(that.value));
    }
  }

  try {
    fn(resolve, reject);
  } catch (e) {
    reject(e);
  }
}

MyPromise.prototype.then = function (onFulfilled, onRejected) {
  const that = this;
  onFulfilled = typeof onFulfilled === "function" ? onFulfilled : (v) => v;
  onRejected =
    typeof onRejected === "function"
      ? onRejected
      : (r) => {
          throw r;
        };
  if (that.state === PENDING) {
    that.resolvedCallbacks.push(onFulfilled);
    that.rejectedCallbacks.push(onRejected);
  }
  if (that.state === RESOLVED) {
    onFulfilled(that.value);
  }
  if (that.state === REJECTED) {
    onRejected(that.value);
  }
};
```

## 参考出处

1. [Promises/A+规范]
2. [神三元-原生 JS(下篇)](https://juejin.im/post/5dd8b3a851882572f56b578f#heading-30)
3. [yck 小册](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5be1a7e451882516bc477978)
4. [雪天\_-解读 Promise 内部实现原理](https://juejin.im/post/5a30193051882503dc53af3c)
5. [写代码像蔡徐抻-异步编程二三事](https://juejin.im/post/5e3b9ae26fb9a07ca714a5cc)
6. [nswbmw-深入 Promise(一)](https://zhuanlan.zhihu.com/p/25178630)
7. [nswbmw-深入 Promise(二)](https://zhuanlan.zhihu.com/p/25198178)
8. [xieranmaya-剖析 Promise 内部结构](https://github.com/xieranmaya/blog/issues/3)
9. [Carus-最最最详细的手写 Promise 教程](https://juejin.im/post/5b2f02cd5188252b937548ab)

[promises/a+规范]: https://www.ituring.com.cn/article/66566
