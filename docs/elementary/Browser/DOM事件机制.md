---
sidebar_position: 2
description: 'DOM事件级别以及事件流'
---

# DOM 事件机制

## DOM 级别

DOM 级别可以分成 4 个级别：DOM0、DOM1、DOM2、DOM3。

DOM 级别来源于 W3C 标准。  
1998 年 W3C 发布了 DOM 级别 1 标准。它含有文档导航和处理功能。  
2000 年发布了 DOM 级别 2 标准。DOM2 级是 DOM1 级的扩展，为 DOM 提供了更多的特性，并支持了 CSS。  
2004 年发布了 DOM 级别 3 标准，进一步增加了 DOM 的功能，支持将 DOM 转换成 XML。  
实际上没有 DOM0 这个东西。DOM0 一般指的是 DOM1 出现之前的技术的定义。

## DOM 事件级别

DOM 事件可以分为 DOM0 级事件处理、DOM2 级事件处理、DOM3 级事件处理。分别对应 DOM 级别的不同阶段。

DOM1 级没有事件的相关内容，因此没有 DOM1 级事件处理。

### DOM0 级事件处理

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

### DOM2 级事件处理

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

### DOM3 级事件

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

## DOM 事件流

### 为什么要有事件流

假如在一个 Button 上注册了一个 Click 事件，在其父 DIV 上注册了另一个 Click 事件，那么当我点击 Button 时，两个事件发生的先后顺序是什么？这需要一种约定来规范事件执行的顺序。这就是**事件流**。

在 DOM0 年代，微软和网景这两个主要的浏览器厂商走了两种不同的路线。

- 微软的 IE9 以及更早的浏览器使用的是**事件冒泡**，先从具体的接收元素，然后逐步向上传播到不具体的元素。
- 网景采用的是**事件捕获**，先由不具体的元素接收事件，最具体的节点最后才接收到事件。

在后来 W3C 指定的标准中，**同时采用**了这两种方案。

### DOM 事件模型

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

## 事件代理（事件委托）

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

## Event 对象常见的方法和属性

Event 接口表示在 DOM 中发生的任何事件。

由于示例代码过长，这边不再赘述，可以在[这里](https://juejin.im/post/5bd2e5f8e51d4524640e1304#heading-10)查看具体的示例代码。下面只列出大纲结论性内容。

- `event.preventDefault()`：如果调用这个方法，默认事件行为将不再触发。
- `event.stopPropagation()`：方法阻止事件冒泡到父元素，阻止任何父事件处理程序被执行.
- `event.stopImmediatePropagation ()`：既能阻止事件向父元素冒泡，也能阻止元素同事件类型的其它监听器被触发。
- `event.target`指向引起触发事件的元素，是**事件的真正发出者**。
- `event.currentTarget`是事件绑定的元素，是**监听事件者**。

## 参考出处

1. [W3C DOM 活动](https://www.w3school.com.cn/w3c/w3c_dom.asp)
2. [刘倩文-DOM 级别详解](https://www.cnblogs.com/lqw007/articles/9639090.html)
3. [DOM 标准规范](http://c.biancheng.net/view/5887.html)
4. [浪里行舟-DOM 事件机制](https://juejin.im/post/5bd2e5f8e51d4524640e1304)
5. [伪硬核玩家-深入理解 DOM 事件机制](https://juejin.im/post/5c71e80d51882562547bb0ce)
