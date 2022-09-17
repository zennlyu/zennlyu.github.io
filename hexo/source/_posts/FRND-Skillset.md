---
title: Front-End Skill Set
categories: [devtools]
tags: [random-notes]
---

## Front-End Skill Set

### HTML/CSS 基础

> 这一部分的知识点大都偏记忆型，较为简单。虽不需要特地花时间去背，但需注重积累，建议在实战开发过程中，养成随时理解、随手记录的习惯。

1. HTML 各类标签（块级、行内元素的定义以及二者相互转换几乎是每场技术一面必考问题）
   - HTML5中新增的语义化标签、客户端存储方案、websocket等
2. CSS各类常用的属性是必要的
   - 浮动、清除浮动以及定位更是必考点。
   - CSS盒模型（盒模型的类型、切换模式、计算不同盒模型下盒子宽高）
   - CSS各类选择器（各个选择器优先级顺序、多个选择器组合时优先级比重的计算）
   - CSS中的伪类与伪元素在实际开发中使用频率很高（比如鼠标在标签上的四种状态，利用伪元素在元素前后添加一些不在DOM tree的元素等）
   - CSS3 的新特性
   - 常见布局（盒子页面居中、两栏、三栏自适应布局、BFC布局、Flex布局、table布局、grid布局、移动端布局（rem、流式、自适应、响应式）。这些布局实现方法有很多种，比如三栏布局可用圣杯布局、双飞翼布局、Flex布局、table布局等多种方法实现，故学习时要掌握多种实现方法，同时要加以自己的理解灵活变通。BFC布局可用来清除浮动、解决margin坍塌重叠等问题，几乎是第一轮技术面必考问题。Flex布局原理与应用也是技术面中的高频考点，可以直接看阮一峰老师的[Flex布局教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)。）

### JavaScript/TypeScript

> 建议根据《JavaScript高级程序设计》进行系统学习。

1. 基础知识超高频考点有数据类型以及判断方法、闭包、块级作用域、函数提升与变量提升、原型链、JS继承、变量深浅拷贝等等

2. JS 高级特性：闭包将涉及到内存泄漏问题与垃圾回收机制

3. JS是单线程的语言，浏览器和Node.js定义了各自的Event Loop（事件循环机制）则是用来解决异步问题。将程序分为“主线程（执行栈）”与“Event Loop线程”，“主线程”自上而下依次执行同步任务，“Event Loop线程”将异步任务推入宏任务队列与微任务队列去执行，在下面这段代码是面试中关于这类问题的经典考题，其中包含了同步、异步任务，几个输出的先后顺序是怎样的。

   ```javascript
   setTimeout(function(){
       console.log('1')
   });
   new Promise(function(resolve){
       console.log('2');
       resolve();
   }).then(function(){
       console.log('3')
   });
   console.log('4');
   // 2,4,3,1
   // 首先进行任务划分，
   // 同步任务：new Promise()、console.log('4')；
   // 宏任务: setTimeout();
   // 微任务：Promise().then()；
   // Event Loop依次将同步任务推入执行栈并执行，当遇到宏任务或微任务时，推到宏任务或微任务队列中，同步队列执行完毕，会去微队列取任务，直到微队列清空，再去宏队列取任务执行。故此段程序执行顺序为：new Promise()、console.log('4')、Promise().then()、setTimeout()。
   ```

4. JS 事件机制 (原生事件绑定、事件冒泡、事件委托、事件监听、阻止默认事件触发) - 见《JS 高级程序设计》第13章

### ES6 新特性

> [ES6 标准](http://caibaojian.com/es6/generator.html)

1. 箭头函数：箭头函数与普通函数在原理与使用上有哪些区别呢？

2. Promise相关

   Promise的实现原理以及代码。用 Promise 函数实现 sleep 函数是滴滴二面代码题之一

3. let/const/var

4. async/await

5. 前端模块化

6. ES6 与 ES5 对比性学习：如何用ES5来实现ES6中的Class功能？

   

### 浏览器





### 框架相关

### 可视化技术/图形学

### 设计模式与工程化

Webpack 原理，常见 loader

MVC，MVVM

### 计算机网络

OSI七层模型、各层中的传输协议、TCP/UDP区别、TCP三次握手四次挥手、HTTP/HTTPS区别、HTTP各版本、HTTP报文结构等等。

浏览器的网络攻防问题千万不可忽视，常见的攻击如CSRF、XSS、SQL注入的攻击原理与途径，应对的防御措施都需要熟悉与理解，多看一些常见攻击案例，如经典的CSRF银行案例，不仅有助于理解，在面试时也能通过举例阐述得更加清楚。

### 数据结构与算法

1. 数组、链表、队列、栈、Set、Map、哈希表
2. JS代码实现数据结构
3. 排序算法与一些其他算法题。快速排序、归并排序、堆排序、冒泡排序、插入排序、选择排序、希尔排序、桶排序、基数排序、Timsort这十种，

### 前端必撕代码

深拷贝、防抖节流、手写Promise、Promise实现Sleep函数、原型链、CSS画三角形、自适应布局、JS继承、数组去重、Ajax请求过程等。

```javascript
#纯CSS画三角形
.triangle{
    width: 0;
    height: 0;
    border: 10px solid transparent;
    border-top-color: #44a5fc;
    border-right-color: #44a5fc;
    transform: skew(20deg);  #用来画钝角，使元素在水平方向和垂直方向同时倾斜（X轴和Y轴同时倾斜）
    position: absolute; #定位调节三角形位置
    left: 116px;
    top: 26px;
}
//Promise实现Sleep函数
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}
async function test() {
  console.log(new Date());
  await sleep(3000);
  console.log(new Date());
}
test();
console.log('continue execute！');
```



### 常考智力题

25匹马赛跑、烧绳子、药丸质量检测、3L和5L杯子倒出4L的水等等
