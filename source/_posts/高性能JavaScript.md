---
title: 高性能JavaScript
date: 2019-07-11 10:41:09
tags: 高性能JavaScript
categories: Javascript

---


> 高性能JavaScript
> author: @TiffanysBear

从《高性能JavaScript》一书中的整理笔记：

1、将经常使用的对象成员、数组项、和域外变量存入局部变量

原因：数据存储位置对大地代码整体性能会产生重要的影响，直接变量和局部变量的访问速度快于数组和对象成员。因为局部变量位于作用域链的第一个对象中，全局变量位于作用域链的最后一环。变量在作用域链的位置越深，访问的时间就越长。

```javascipt
var doc = document;
var db = doc.body;
var odiv = doc.getElementById('div1'); 

```
 

2、避免使用with表达式，因为他改变了运行期上下文的作用域链。

3、同理with，也要注意使用try-catch，因为catch也会改变运行期上下文的作用域链。

4、嵌套成员变量会造成重大的性能影响，尽量少用。

5、DOM操作量化问题：

```javascript

// 在循坏中更新页面，问题所在：每次循环都对DOM元素访问了两次
// 一次是读取document.getElementById('here').innerHTML的内容
// 一次是修改它。
function changeDOM() {
    for (var i=0; i < 15000; i++) {
        document.getElementById('here').innerHTML += 'a';
    }

}    
// 改变方法，使用局部变量存好改变量，在循环结束时一并修改
function changeDOM() {
    var content =''；
    for (var i=0; i < 15000; i++) {
        content += 'a';
    }
    document.getElementById('here').innerHTML += content;
}
    // 关于js字符串拼接的性能优化问题
    // js的处理机制是：新建一个临时字符串，将新字符串赋值为 content + 'a'
    // 然后返回这个新字符串并同时销毁原始字符串
    // 导致字符串的连接效率较低的重要原因不仅在于对于新的临时变量的不断创建
    // 还有js的垃圾回收机制下不断在对象创建期间回收，导致的效率低下
    // 提高效率的办法是用数组的join函数：
        function changeDOM() {
          var content =[]；
          for (var i=0; i < 15000; i++) {
              content.push('a');
          }
         document.getElementById('here').innerHTML += content.join('');
      }
    // 但是同时也要注意，后来的大部分浏览器都对“+”的连接字符串做了优化
    // 由于SpiderMonkey等引擎对字符串的“+”运算做了优化，结果使用Array.join的效率反而不如直接用"+"！
    // 因此建议是：在IE7以下，使用join，在新浏览器下，除了变量缓存外，不需要做别的优化
　　　　　　
```

<!-- more -->

6、克隆已有的DOM元素，即element.cloneNode(),比起新建节点来说，即element.createElement()，会快一点，但是性能提高不是很大。

7、遍历数组明显快于同样大小和内容的HTML集合

8、 for循环时，HTML某元素集合的长度不建议直接作为循环终止条件，最好将集合的长度赋给一个变量，然后使用变量作为循环终止条件；

原因：当每次迭代过程访问集合的length时，它导致集合器更新，在所有的浏览器上都会产生明显的性能损失。

9、需要考虑实际情况的优化，根据7，可以将集合中的元素通过for循坏赋值到数组中，访问数组的数组快于集合。但是要注意对于复制的开销是否值得。

```javascript
function toArray(collection) {
    var arr = [];
    var clen = collection.length;
    for (var i= 0; i < clen; i++) {
        arr[i] = collection[i];
    }
}

```

10、获取DOM节点，使用nextSibling方式与childNodes方式，在不同的浏览器中，这两种方法的时间基本相等。但是在IE中，nextSibling比childNodes好，IE6下，nextSibling比对手快16倍，在IE7下，快105倍。因此，在老的IE中性能严苛的使用条件下，用nextSibling较好。

11、querySelectorAll()可以联合查询，即querySelectorAll(‘div .warning，div .notice’),在各大浏览器中支持也挺好的，还可以过滤很多非元素节点；

这个网站是：[canIuse](http://caniuse.com/)，可以检查HTML、CSS元素在各大浏览器的兼容情况，一个很有用的网站！


12、重绘和重排版；

重绘：不需要改变元素的长度和宽度，不影响DOM的几何属性；

重排版：影响了几何属性，需要重新计算元素的几何属性，而且其他元素的几何属性有可能也会受影响。浏览器会在重排版过程中，重新绘制屏幕上受影响的部分。

获取布局信息的操作将导致刷新队列的动作，如使用：`offsetTop`、`offsetLeft`、`offsetWidth`、`offsetHeight`、`scrollTop`、`scrollLeft`、`scrollWidth`、`clientTop`、`clientLeft`、`clientHeight`、`geteComputedStyle()`(在IE中此函数成为currentStyle)；浏览器此时不得不进行渲染队列中带改变的项目，并重新排版以返回正确值。

解决办法：

* 通过延迟访问布局信息避免重排版。

* 整体修改cssText的css代码，而不是分开访问，修改cssText的属性

```js
// 访问了4次DOM，第二次开始重排列并强迫渲染队列执行
var el = document.getElementById('div1');
el.style.borderLeft = '1px';
el.style.borderRight = '2px';
el.style.padding = '5px';
// 改进：改变合并，通过cssText实现
var el = document.getElementById('div1');
el.cssText += 'border-left = 1px; border-right = 2px; padding = 5px;';

```

* 改变css类名来实现样式改变

* 当对DOM元素进行多次修改时，可以通过以下的步骤减少重绘和重排版的次数：

(注意：此过程引发两次重排版，第一次引发一次，第三次引发一次。如果没有此步骤的话，每次对第二步的改变都有可能带来重排版。)

从文档流中摘除该元素，摘除该元素的方法有：
a、对其应用多重改变
b、将元素带回文档中
c、使其隐藏，进行修改后在显示
d、使用文档片段创建子树，在将他拷贝进文档

```js
var doc = document;
// 创建文档子树
var frag = doc.createDocumentFragment();
// 自定义函数，将修改内容data赋给文档片段frag，具体过程忽略
appendDataToElement(frag,data);
// 注意：添加时实际添加的是文档片段的子节点群，而不是frag自己，只会引发一次重排版
doc.getElementById('div1').appendChild(frag);
 
```

创建一个节点的副本，在副本上进行修改，再让复制节点覆盖原先节点

```js
// 创建一个节点的副本，在副本上进行修改，再让复制节点覆盖原先节点
var oldNode = document.getElementById('old');
var clone = old.cloneNode();
appendDataToElement(clone, data);
oldNode.replaceChild(clone, oldNode);

```

ps：推荐第二种，因为其涉及最少数量的操作和重排列。


14、减少对布局信息的查询次数，查询时将他赋值给局部变量参与计算。

例子，在元素网右下方不断平移时，在timeout中可以写：


```js
var current = myElement.offsetLeft;
current++;
myElement.style.left = current + 'px';
myElement.style.top =  current + 'px';
if (current > 500) {
// stop animation
}
// 拒绝下面的写法，每次移动都会查询一次偏移量，导致浏览器刷新渲染队列，非常耗时

myElement.style.left = myElement.offsetLeft + 'px';
myElement.style.top =  myElement.offsetLeft + 'px';
if (myElement.offsetLeft > 500) {
 // stop animation
}

```
15、大量的元素使用：hover之后，页面性能将降低，特别是IE8中。因此强烈建议，在数据量很大的表格中，减少鼠标在表上移动效果，减少高亮行的显示，使用高亮是个慢速过程CPU使用率会提高到80%-90%，尽量避免使用这种效果。

16、事件托管

讲到事件托管，首先我们来看一看冒泡机制：

DOM2级事件规定事件包括三个阶段： ① 事件捕获阶段 ② 处于目标阶段 ③ 事件冒泡阶段

![](https://www.w3.org/TR/DOM-Level-3-Events/images/eventflow.svg)


图片引用来源：http://www.w3.org/TR/DOM-Level-3-Events/#event-flow

如下图的实验结果可以知道，当我们点击了inner之后，捕获和冒泡结果如上图的规律相同；



因此，因为每一个元素有一个或多个事件句柄与之相连时，可能会影响性能，毕竟连接每一个句柄都是有代价的，所以我们采用事件托管技术，在一个包装元素上挂接一个句柄，用于处理子元素发生的所有事件。

下面我们以如下的dom结构为例：

假如有一个ul，下面有很多个li：

```html
<div>
    <ul id="ulList">
        <li id="item1"></li>
        <li id="item2"></li>
        <li id="item3"></li>
        <li id="item4"></li>
        <li id="item5"></li>
    </ul>
</div>
```
当某个Li被点击的时候需要触发相应的处理事件。我们通常的写法，是为每个Li都添加onClick的事件监听。

```js
function addListenersLi(liNode) {
    liNode.onclick = function clickHandler(){}
}

window.onload = function(){
    var ulNode = document.getElementById("ulList");
    var liNodes = ulNode.getElementByTagName("li");
    for(var i=0, l = liNodes.length; i < l; i++){
        addListeners4Li(liNodes[i]);
    }   
}
```

如果li足够多，或者对于li的操作特别频繁，为每一个li绑定一个点击事件将会特别影响性能，因为在此期间，你需要访问和修改更多的DOM节点，事件的绑定过程发生在onload事件中，绑定本身也非常耗时；同时，浏览器需要保存每个句柄的记录，很占用内存。重点是有些绑定了还不一定会用着，并不是100%的按钮或链接都会被点到的哟！

因此，采用事件托管更为高效，当事件被抛到更上层的父节点的时候，我们通过检查事件的目标对象（target）来判断并获取事件源Li。下面的代码可以完成我们想要的效果：

```js
var oul = document.getElementById('ulList');
oul.addEventListener('click',function(e){
    var e = e || window.event;
    var target = e.target || e.srcElement;
 
    console.log(target.nodeName);
    if (target.nodeName == 'LI') {
        // 事件真正的处理程序
        alert(target.id);
        e.preventDefault();
        e.stopPropagation();
    }
    else {
        console.log(target.nodeName);
    }

});
```

























