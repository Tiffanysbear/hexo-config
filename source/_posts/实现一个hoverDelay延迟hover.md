---
title: 实现一个hoverDelay延迟hover
date: 2019-07-30 20:22:41
tags: 问题收集和解决
categories: JavaScript
---



> 实现一个hoverDelay延迟hover
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

### 需求背景
经常在页面开发中，需要使用hover事件来触发相应的网络请求或页面DOM元素显示切换，需要考虑的问题就有了：

* hover动作非常快，如果一hover就请求，会造成多余请求的浪费，造成后端接口不必要的压力
* 如何判断这个用户hover是想做一定的操作，而不是鼠标误触
* 构造这个hover延迟的时候，怎样封装才能通用使用

先来看一下效果演示：

<iframe height="257" style="width: 100%;" scrolling="no" title="Vue.js | Mouseover &amp; Mouseleave" src="//codepen.io/AAA_TTT/embed/VorrpN/?height=257&theme-id=0&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/AAA_TTT/pen/VorrpN/'>Vue.js | Mouseover &amp; Mouseleave</a> by AAA_TTT
  (<a href='https://codepen.io/AAA_TTT'>@AAA_TTT</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

### 问题思考
基于上述的问题，思考是如下：

* 当用户hover停留在某一DOM元素上一定时长时，比如500ms，才认为这个用户是实际想要做某种操作，这时候在实际的进行相应的网络请求或页面DOM元素显示切换
* 如果在500ms之前就移开，就算是用户误触误滑，不做任何处理
* 构造hover通用封装时，采用jQuery的插件开发的方式，形成通用的解决方案

<!-- more -->

### 代码封装

基于jQuery的插件系统，实现的hoverDelay插件方法；在每次事件之前，清空所有的计时器，重新设置延时定时器，则进行相应操作。

```
/**
 * @file: 鼠标滑动延迟执行的JQUERY扩展方法
 * @author: TiffanysBear
 *
 */
$.fn.hoverDelay = function (options) {
    var defaults = {
        hoverDuring: 200,
        outDuring: 200,
        hoverEvent: function () {
            $.noop();
        },
        outEvent: function () {
            $.noop();
        }
    };
    var sets = $.extend(defaults, options || {});
    var hoverTimer;
    var outTimer;
    return $(this).each(function () {
        $(this).hover(function () {
            clearTimeout(outTimer);
            hoverTimer = setTimeout(sets.hoverEvent, sets.hoverDuring);
        }, function () {
            clearTimeout(hoverTimer);
            outTimer = setTimeout(sets.outEvent, sets.outDuring);
        });
    });
};

```


### 代码使用
因为该方法是放在jQuery的原型方法上，因此所有jQuery对象都有这个方法可以使用。

```
$(this).hoverDelay({
    hoverDuring: 500,
    outDuring: 300,
    hoverEvent: function () {
        
    },
    outEvent: function () {
        
    }
});

```


### 后续思考

类似与Vue这种类似于MVVM框架的项目应该如何做hoverDelay呢？原理也是一致的；但是在细节的处理上有些不同，通过Vue绑定的 `mouseover`、`mouseleave`对定时器进行设置和清理也能实现需求。

html结构：

```
.<div id="mouse">
  <a
    v-on:mouseover="mouseover"
    v-on:mouseleave="mouseleave">
    {{message}}
  </a>
</div>

```

css样式：

```
body {
  
  background: #333;
  
  #mouse {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: block;
    width: 280px;
    height: 50px;
    margin: 0 auto;
    line-height: 50px;
    text-align: center;
    color: #fff;
    background: #007db9;

    a {
      display: block;
      width: 100%;
      height: 100%;
      cursor: pointer;
    }
  }
}

```


JS代码：

```
new Vue({
  el: '#mouse',
  data: {
    message: 'Hover Me!' ,
    timer: null,
    hoverEnterTime: 500,
    hoverLeaveTime: 300
  }, 
  methods: {
    mouseover: function(){
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.message = 'Good!'
      }, this.hoverEnterTime);
    },    
    mouseleave: function(){
      clearTimeout(this.timer);
      this.timer = setTimeout(() => {
        this.message = 'Hover Me!!'
      }, this.hoverEnterTime);
    }
  }
})


```

[代码效果和功能演示](https://codepen.io/AAA_TTT/pen/BXmwVg)：



<iframe height="265" style="width: 100%;" scrolling="no" title="Vue.js | Mouseover &amp; Mouseleave" src="https://codepen.io/AAA_TTT/embed/VorrpN/?height=265&theme-id=0&default-tab=js" frameborder="no" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href='https://codepen.io/AAA_TTT/pen/VorrpN/'>Vue.js | Mouseover &amp; Mouseleave</a> by AAA_TTT
  (<a href='https://codepen.io/AAA_TTT'>@AAA_TTT</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>









