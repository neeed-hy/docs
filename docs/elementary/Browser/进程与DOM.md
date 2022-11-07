---
sidebar_position: 1
---

# 浏览器基础知识-进程与 DOM 事件

[[toc]]

## 浏览器进程与线程

### 基础概念

> 进程是 cpu 资源分配的最小单位（是能拥有资源和独立运行的最小单位）;
>
> 线程是 cpu 调度的最小单位（线程是建立在进程的基础上的一次程序运行单位，一个进程中可以有多个线程）。

用比喻的说明方式：

> 进程是工厂。每个工厂有他的独立资源。工厂之间相互独立。
>
> 线程是工厂中的工人，多个工人协作完成任务。工厂内可能有一个或多个工人，工人之间共享工作空间。

完善一下概念：

> 工厂资源 ---> 内存
>
> 工厂之间独立 ---> 进程间独立
>
> 厂内有一个或多个工人 ---> 一个进程由一个或多个线程组成
>
> 多个工人协作完成任务 ---> 多线程在进程中协作完成任务
>
> 工人之间共享空间 ---> 同一进程下的线程共享内存空间

### 浏览器的多进程

浏览器是一个多进程的软件。它包含的**主要**进程有：

- Browser 进程。

  Browser 进程是浏览器的主进程。只有一个。作用有：

  - 负责浏览器界面显示与交互。如前进后退。
  - 负责各个页面的管理，创建和销毁其他进程。
  - 绘制渲染进程中生成的位图。

- 网络进程

  主要负责页面的网络资源加载，之前是作为一个模块运行在浏览器进程里面的，直至最近才独立出来，成为一个单独的进程。

- 第三方插件进程

  每种类型的插件对应一个进程。

- GPU 进程

  最多一个。用于 3D 绘制等场景。

- Renderer 进程

  Renderer 进程是浏览器的渲染进程。也称为浏览器内核。  
  默认每个 Tab 页面一个进程，互相之间不影响。  
  主要执行任务有页面渲染、脚本执行、事件处理等。  
  Renderer 进程内部是多线程的。

基本上说，**浏览器中每打开了一个网页就相当于起了一个新的进程**。

在一些特殊情况下浏览器会将多个进程合并，比如开了多个空白页，这些空白页就会被合并成了一个进程。

浏览器这种多进程机制的优点在于：

- 单个页面崩溃不会影响到其他页面
- 第三方插件崩溃不会影响浏览器
- 提升算力、提升稳定性

### Renderer 进程（渲染进程）

对于前端来说，最重要的是浏览器中的渲染进程。页面的渲染、脚本的执行等重要工作全部在渲染进程内执行。

渲染进程是多线程的。它的重要线程有：

- GUI 渲染线程

  - 负责渲染浏览器界面。  
    解析 HTML、CSS，构建 DOM 树、CSSOM 树、Render Tree 渲染树，进行布局、绘制（指绘制到**内存**中，绘制到显示器上是 Browser 进程的工作）。
  - 当界面需要进行重绘 Repaint 或回流 Reflow 时，该线程就会执行。
  - **GUI 渲染线程和 JS 引擎线程是互斥的**！  
    当 JS 引擎执行时，GUI 渲染线程会被挂起，直到**JS 引擎结束执行，变为空闲**时才会继续。

- JS 引擎线程

  - 也称 JS 内核。负责处理执行 Javascript 脚本程序。最著名的 JS 引擎是 V8 引擎。
  - JS 引擎一直等待着任务队列中任务的到来，然后加以处理。**一个 Tab 页**（也就是一个 Renderer 渲染进程）**中只有一个 JS 引擎线程**在运行 JS 程序。
  - **GUI 渲染线程和 JS 引擎线程是互斥的**！  
    如果 JS 引擎执行时间过长，会造成页面渲染的阻塞。

- 事件触发线程

  - 归属于浏览器而不是 JS 引擎。用来控制事件循环。
  - 当 JS 引擎执行代码块如`setTimeout()`（或浏览器内核的其他线程，如 ajax 请求）时，会将对应任务添加到事件线程中。
  - 当对应的事件符合触发条件被触发时，该线程会把时间添加到待处理队列的队尾，等待 JS 引擎的处理。
  - 由于 JS 是单线程的，待处理队列中的时间都要排队等待 JS 引擎处理。

- 定时触发器线程

  - 传说中`setTimeout()`、`setInterval()`所在的线程。浏览器的这种定时计数器并不是 JS 引擎计数的，而是单独开了一个线程。
  - 计时完毕后，将任务添加到事件队列中，等待 JS 引擎执行。
  - > W3C 在 HTML 标准中规定，规定要求 setTimeout 中低于 4ms 的时间间隔算为 4ms。

- 异步 HTTP 请求线程

  - 在 XMLHttpRequest 在连接后是通过浏览器新开一个线程请求

  - 将检测到状态变更时，如果设置有回调函数，异步线程就产生状态变更事件，将这个回调再放入事件队列中。再由 JavaScript 引擎执行。

### Renderer 进程中线程之间的一些关系

#### GUI 渲染线程和 JS 引擎线程互斥

因为 JS 可以操作 DOM，所以如果 JS 线程和 GUI 线程同时运行，那么渲染线程前后获得的元素数据就可能不一致了。

为了防止渲染出现不可预期的结果，浏览器设置 GUI 渲染线程和 JS 引擎线程互斥。

#### JS 阻塞页面加载

因为上面的互斥关系，JS 如果执行过长就会阻塞页面渲染。

因此要避免 JS 执行的时间过长，不要在 JS 中进行巨量的计算。

#### WebWorker，JS 的伪“多线程”

HTML5 中新增加的特性。

- 创建 Worker 时，JS 引擎向浏览器申请开一个子线程（子线程是浏览器开的，完全受主线程控制，而且不能操作 DOM）

- JS 引擎线程与 worker 线程间通过特定的方式通信（postMessage API，需要通过序列化对象来与线程交互特定的数据）

所以，如果有非常耗时的工作，请单独开一个 Worker 线程，这样里面不管如何翻天覆地都不会影响 JS 引擎主线程，只待计算出结果后，将结果通信给主线程即可。

而且注意下，JS 引擎是单线程的，这一点的本质仍然未改变，Worker 可以理解是浏览器给 JS 引擎开的外挂，专门用来解决那些大量计算问题。

#### WebWorker 与 SharedWorker

既然都到了这里，就再提一下 SharedWorker（避免后续将这两个概念搞混）

- WebWorker 只属于某个页面，不会和其他页面的 Render 进程（浏览器内核进程）共享

  - 所以 Chrome 在 Render 进程中（每一个 Tab 页就是一个 render 进程）创建一个新的线程来运行 Worker 中的 JavaScript 程序。

- SharedWorker 是浏览器所有页面共享的，不能采用与 Worker 同样的方式实现，因为它不隶属于某个 Render 进程，可以为多个 Render 进程共享使用

  - 所以 Chrome 浏览器为 SharedWorker 单独创建一个进程来运行 JavaScript 程序，在浏览器中每个相同的 JavaScript 只存在一个 SharedWorker 进程，不管它被创建多少次。

WebWorker 与 SharedWorker 本质上就是进程和线程的区别。SharedWorker 由独立的进程管理，WebWorker 只是属于 render 进程下的一个线程

### Browser 进程和 Renderer 进程间的通信

- Browser 进程收到用户请求，首先获取页面内容（如下载资源），然后将该任务通过 RendererHost 接口传递给 Renderer 进程。
- Renderer 进程的 Renderer 接口收到消息，简单解释后交给渲染线程开始渲染：
  - 渲染线程接收请求，加载网页并渲染网页。期间可能需要 Browser 进程帮助下载资源、GPU 进程帮助渲染
  - JS 线程负责执行脚本。在执行的过程中可能会操作 DOM，因此和 GUI 渲染线程互斥。
  - 最后 Renderer 进程将渲染结果传输给 Browser 进程。
- Browser 进程接收渲染结果并绘制。

## DOM 事件机制

### DOM 级别

DOM 级别可以分成 4 个级别：DOM0、DOM1、DOM2、DOM3。

DOM 级别来源于 W3C 标准。  
1998 年 W3C 发布了 DOM 级别 1 标准。它含有文档导航和处理功能。  
2000 年发布了 DOM 级别 2 标准。DOM2 级是 DOM1 级的扩展，为 DOM 提供了更多的特性，并支持了 CSS。  
2004 年发布了 DOM 级别 3 标准，进一步增加了 DOM 的功能，支持将 DOM 转换成 XML。  
实际上没有 DOM0 这个东西。DOM0 一般指的是 DOM1 出现之前的技术的定义。

### DOM 事件级别

DOM 事件可以分为 DOM0 级事件处理、DOM2 级事件处理、DOM3 级事件处理。分别对应 DOM 级别的不同阶段。

DOM1 级没有事件的相关内容，因此没有 DOM1 级事件处理。

#### DOM0 级事件处理

请看代码示例：

```html
<button id="btn" type="button"></button>

<script>
  var btn = document.getElementById('btn')
  btn.onclick = function () {
    alert('Hello World')
  }
  // btn.onclick = null; 解绑事件
</script>
```

上面就是典型的 DOM 0 级的事件处理的步骤：先找到 DOM 节点，然后把处理函数赋值给该节点对象的事件属性。

DOM0 级事件处理方式的缺点在于他无法给一个 DOM 绑定多个同类型事件。示例代码如下：

```js
var btn = document.getElementById('btn')

btn.onclick = function () {
  alert('Hello World')
}
btn.onclick = function () {
  alert('这句话才会被真正的执行')
}
```

#### DOM2 级事件处理

DOM2 级事件弥补了 DOM0 级事件的缺陷，可以同时给一个 DOM 绑定多个同类型事件了。

DOM2 级事件新增了 **addEventListener** 和 **removeEventListener** 两个方法，用来绑定/解绑事件。两个方法的用法如下：

```js
target.addEventListener(event-name, callback[, useCapture]);
target.removeEventListener(event-name, callback[, useCapture]);
```

两个函数都接受三个参数：

- event-name：事件名称
- callback：事件处理函数
- useCapture: 在冒泡阶段还是捕获阶段执行。默认是 false，代表事件句柄在冒泡阶段执行。

具体的例子请看下面的代码：

```html
<button type="button" id="btn">点我试试</button>

<script>
  var btn = document.getElementById('btn')

  function fn() {
    alert('Hello World')
  }
  function fn2() {
    alert('Hello World2')
  }
  btn.addEventListener('click', fn, false)
  btn.addEventListener('click', fn2, false)
  // 解绑事件，代码如下
  // btn.removeEventListener('click', fn, false);
</script>
```

注意： IE9 以下不支持 addEventListener 和 removeEventListener 这两个方法。

#### DOM3 级事件

在 DOM 2 级事件的基础上添加了更多的事件类型。

UI 事件，当用户与页面上的元素交互时触发，如：load、scroll  
焦点事件，当元素获得或失去焦点时触发，如：blur、focus  
鼠标事件，当用户通过鼠标在页面执行操作时触发如：dblclick、mouseup  
滚轮事件，当使用鼠标滚轮或类似设备时触发，如：mousewheel  
文本事件，当在文档中输入文本时触发，如：textInput  
键盘事件，当用户通过键盘在页面上执行操作时触发，如：keydown、keypress  
合成事件，当为 IME（输入法编辑器）输入字符时触发，如：compositionstart  
变动事件，当底层 DOM 结构发生变化时触发，如：DOMsubtreeModified  
同时 DOM3 级事件也允许使用者自定义一些事件。

### DOM 事件流

#### 为什么要有事件流

假如在一个 Button 上注册了一个 Click 事件，在其父 DIV 上注册了另一个 Click 事件，那么当我点击 Button 时，两个事件发生的先后顺序是什么？这需要一种约定来规范事件执行的顺序。这就是**事件流**。

在 DOM0 年代，微软和网景这两个主要的浏览器厂商走了两种不同的路线。

- 微软的 IE9 以及更早的浏览器使用的是**事件冒泡**，先从具体的接收元素，然后逐步向上传播到不具体的元素。
- 网景采用的是**事件捕获**，先由不具体的元素接收事件，最具体的节点最后才接收到事件。

在后来 W3C 指定的标准中，**同时采用**了这两种方案。

#### DOM 事件模型

DOM 事件模型分为捕获和冒泡。一个事件发生后，会在父元素和子元素之间传播。传播分为三个阶段：

- 捕获阶段：事件从 window 对象自上而下向目标节点传播的阶段；

- 目标阶段：真正的目标节点正在处理事件的阶段；

- 冒泡阶段：事件从目标节点自下而上向 window 对象传播的阶段。

请看示例代码。下面的代码分别为三个嵌套的 div 注册了点击事件。  
那么，点击最下层的儿子 div 时打印的结果是什么？

```html
<div id="grandfather1">
  爷爷
  <div id="parent1">
    父亲
    <div id="child1">儿子</div>
  </div>
</div>
<script>
  var grandfather1 = document.getElementById('grandfather1'),
    parent1 = document.getElementById('parent1'),
    child1 = document.getElementById('child1')

  grandfather1.addEventListener(
    'click',
    function fn1() {
      console.log('爷爷')
    },
    true //true代表捕获阶段执行
  )
  parent1.addEventListener(
    'click',
    function fn1() {
      console.log('爸爸')
    },
    false //false代表冒泡阶段执行
  )
  child1.addEventListener(
    'click',
    function fn1() {
      console.log('儿子')
    },
    true //true代表捕获阶段执行
  )
</script>
```

答案是：

```html
爷爷--儿子--爸爸
```

因为向“爷爷 div”注册的事件，他的 addEventListener 的第三个函数是 true；而向“爸爸 div”注册的事件，他的 addEventListener 的第三个函数是 false。  
因此在捕获的过程中首先输出“爷爷”，再输出最下层的“儿子”，最后在冒泡阶段输出“爸爸”。

请注意：addEventListener 方法**默认在冒泡阶段执行**。

### 事件代理（事件委托）

由于事件会在冒泡阶段向上传播到父节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件。这种方法叫做事件的代理（delegation）。

它具有下面两个优点：

- 减少内存消耗，提高性能。

  假设有一个列表，列表之中有大量的列表项，我们需要在点击每个列表项的时候响应一个事件。

  ```html
  <ul id="list">
    <li>item 1</li>
    <li>item 2</li>
    <li>item 3</li>
    ......
    <li>item n</li>
  </ul>
  ```

  如果给每一个 li 都注册事件，那么会有很大的内存消耗。借助事件代理，可以把事件注册在父组件 ul 上。

- 动态绑定事件

  在很多时候，我们需要通过用户操作动态的增删列表项元素，如果一开始给每个子元素绑定事件，那么在列表发生变化时，就需要重新给新增的元素绑定事件，给即将删去的元素解绑事件，如果用事件代理就会省去很多这样麻烦。

### Event 对象常见的方法和属性

Event 接口表示在 DOM 中发生的任何事件。

由于示例代码过长，这边不再赘述，可以在[这里](https://juejin.im/post/5bd2e5f8e51d4524640e1304#heading-10)查看具体的示例代码。下面只列出大纲结论性内容。

- `event.preventDefault()`：如果调用这个方法，默认事件行为将不再触发。
- `event.stopPropagation()`：方法阻止事件冒泡到父元素，阻止任何父事件处理程序被执行.
- `event.stopImmediatePropagation ()`：既能阻止事件向父元素冒泡，也能阻止元素同事件类型的其它监听器被触发。
- `event.target`指向引起触发事件的元素，是**事件的真正发出者**。
- `event.currentTarget`是事件绑定的元素，是**监听事件者**。

## 跨域

## 防抖与节流

## 参考出处

1. [yck 小册](https://juejin.im/book/5bdc715fe51d454e755f75ef/section/5bdc71fbf265da6128599324)
2. [神三元-浏览器灵魂之问](https://juejin.im/post/5df5bcea6fb9a016091def69#heading-0)
3. [阮一峰-进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)
4. [dailc-从浏览器多进程到 JS 单线程，JS 运行机制最全面的一次梳理](https://juejin.im/post/5a6547d0f265da3e283a1df7)
5. [从 chrome 浏览器说说进行与线程](https://segmentfault.com/a/1190000020526789?_ea=20309599)
6. [W3C DOM 活动](https://www.w3school.com.cn/w3c/w3c_dom.asp)
7. [刘倩文-DOM 级别详解](https://www.cnblogs.com/lqw007/articles/9639090.html)
8. [DOM 标准规范](http://c.biancheng.net/view/5887.html)
9. [浪里行舟-DOM 事件机制](https://juejin.im/post/5bd2e5f8e51d4524640e1304)
10. [伪硬核玩家-深入理解 DOM 事件机制](https://juejin.im/post/5c71e80d51882562547bb0ce)
11. [浪里行舟-九种跨域方式实现原理（完整版）](https://juejin.im/post/5c23993de51d457b8c1f4ee1)
