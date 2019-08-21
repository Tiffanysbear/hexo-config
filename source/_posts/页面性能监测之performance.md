---
title: 页面性能监测之performance
date: 2019-08-14 11:03:34
tags: 性能监控
categories: JavaScript
---


> 页面性能监测之performance
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

最近，需要对业务上的一些性能做一些优化，比如降低首屏时间、减少核心按钮可操作时间等的一些操作；在这之前，需要建立的就是数据监控的准线，也就是说一开始的页面首屏数据是怎样的，优化之后的数据是怎样，需要有一个对比效果。此时，performance 这个API就非常合适了。

### performance

> Performance 接口可以获取到当前页面中与性能相关的信息。它是 High Resolution Time API 的一部分，同时也融合了 Performance Timeline API、Navigation Timing API、 User Timing API 和 Resource Timing API。

> 该类型的对象可以通过调用只读属性 Window.performance 来获得。
> 
> [参考链接](https://developer.mozilla.org/zh-CN/docs/Web/API/Performance
) https://developer.mozilla.org/zh-CN/docs/Web/API/Performance


### performance.timing对象

performance对象是全局的，它的timing属性是一个对象，它包含了各种与浏览器性能有关的时间数据，提供浏览器处理网页各个阶段的耗时。偷一个图~
![](https://github.com/Tiffanysbear/accumulation/raw/master/image/perf-1.png)

performance.timing对象包含下列属性（全部只读）：

* navigationStart：当前浏览器窗口的前一个网页关闭，发生unload事件时的Unix毫秒时间戳。如果没有前一个网页，则等于fetchStart属性。

* unloadEventStart：如果前一个网页与当前网页属于同一个域名，则返回前一个网页的unload事件发生时的Unix毫秒时间戳。如果没有前一个网页，或者之前的网页跳转不是在同一个域名内，则返回值为0。

* unloadEventEnd：如果前一个网页与当前网页属于同一个域名，则返回前一个网页unload事件的回调函数结束时的Unix毫秒时间戳。如果没有前一个网页，或者之前的网页跳转不是在同一个域名内，则返回值为0。

* redirectStart：返回第一个HTTP跳转开始时的Unix毫秒时间戳。如果没有跳转，或者不是同一个域名内部的跳转，则返回值为0。

* redirectEnd：返回最后一个HTTP跳转结束时（即跳转回应的最后一个字节接受完成时）的Unix毫秒时间戳。如果没有跳转，或者不是同一个域名内部的跳转，则返回值为0。

* fetchStart：返回浏览器准备使用HTTP请求读取文档时的Unix毫秒时间戳。该事件在网页查询本地缓存之前发生。

* domainLookupStart：返回域名查询开始时的Unix毫秒时间戳。如果使用持久连接，或者信息是从本地缓存获取的，则返回值等同于fetchStart属性的值。

* domainLookupEnd：返回域名查询结束时的Unix毫秒时间戳。如果使用持久连接，或者信息是从本地缓存获取的，则返回值等同于fetchStart属性的值。

* connectStart：返回HTTP请求开始向服务器发送时的Unix毫秒时间戳。如果使用持久连接（persistent connection），则返回值等同于fetchStart属性的值。

* connectEnd：返回浏览器与服务器之间的连接建立时的Unix毫秒时间戳。如果建立的是持久连接，则返回值等同于fetchStart属性的值。连接建立指的是所有握手和认证过程全部结束。

* secureConnectionStart：返回浏览器与服务器开始安全链接的握手时的Unix毫秒时间戳。如果当前网页不要求安全连接，则返回0。

* requestStart：返回浏览器向服务器发出HTTP请求时（或开始读取本地缓存时）的Unix毫秒时间戳。

* responseStart：返回浏览器从服务器收到（或从本地缓存读取）第一个字节时的Unix毫秒时间戳。

* responseEnd：返回浏览器从服务器收到（或从本地缓存读取）最后一个字节时（如果在此之前HTTP连接已经关闭，则返回关闭时）的Unix毫秒时间戳。

* domLoading：返回当前网页DOM结构开始解析时（即Document.readyState属性变为“loading”、相应的readystatechange事件触发时）的Unix毫秒时间戳。

* domInteractive：返回当前网页DOM结构结束解析、开始加载内嵌资源时（即Document.readyState属性变为“interactive”、相应的readystatechange事件触发时）的Unix毫秒时间戳。

* domContentLoadedEventStart：返回当前网页DOMContentLoaded事件发生时（即DOM结构解析完毕、所有脚本开始运行时）的Unix毫秒时间戳。

* domContentLoadedEventEnd：返回当前网页所有需要执行的脚本执行完成时的Unix毫秒时间戳。

* domComplete：返回当前网页DOM结构生成时（即Document.readyState属性变为“complete”，以及相应的readystatechange事件发生时）的Unix毫秒时间戳。

* loadEventStart：返回当前网页load事件的回调函数开始时的Unix毫秒时间戳。如果该事件还没有发生，返回0。

* loadEventEnd：返回当前网页load事件的回调函数运行结束时的Unix毫秒时间戳。如果该事件还没有发生，返回0。

<!-- more -->

### performance API

#### 1、performance.now()
performance.now()方法返回当前网页自从performance.timing.navigationStart到当前时间之间的毫秒数。

```
performance.now()

// Date.now() 近似等于 (performance.timing.navigationStart + performance.now())

```


#### 2、performance.mark()
该方法是做一个标记mark，结合measures方法，可以计算两个标记之间间隔的时间差；因此可以直接依据业务上的不同，计算两个业务逻辑之间的距离。

```
// 以一个标志开始。
performance.mark('mySetTimeout-start');

// 等待一些时间。
setTimeout(function() {
  // 标志时间的结束。
  performance.mark('mySetTimeout-end');

  // 测量两个不同的标志。
  performance.measure(
    'mySetTimeout',
    'mySetTimeout-start',
    'mySetTimeout-end'
  );

  // 获取所有的测量输出。
  // 在这个例子中只有一个。
  var measures = performance.getEntriesByName('mySetTimeout');
  var measure = measures[0];
  console.log('setTimeout milliseconds:', measure.duration)

  // 清除存储的标志位
  performance.clearMarks();
  performance.clearMeasures();
}, 1000);

```

#### 3、performance.getEntries()
浏览器获取网页时，会对网页中每一个对象（脚本文件、样式表、图片文件等等）发出一个HTTP请求。performance.getEntries方法以数组形式，返回这些请求的时间统计信息，有多少个请求，返回数组就会有多少个成员。

由于该方法与浏览器处理网页的过程相关，所以只能在浏览器中使用。

![](https://github.com/Tiffanysbear/accumulation/raw/master/image/perf-0.png)


#### 4、performance.navigation对象

（1）performance.navigation.type: 该属性返回一个整数值，表示网页的加载来源

> 0：网页通过点击链接、地址栏输入、表单提交、脚本操作等方式加载，相当于常数performance.navigation.TYPE_NAVIGATENEXT。

> 1：网页通过“重新加载”按钮或者location.reload()方法加载，相当于常数performance.navigation.TYPE_RELOAD。

> 2：网页通过“前进”或“后退”按钮加载，相当于常数performance.navigation.TYPE_BACK_FORWARD。

> 255：任何其他来源的加载，相当于常数performance.navigation.TYPE_UNDEFINED。


（2）performance.navigation.redirectCount: 该属性表示当前网页经过了多少次重定向跳转。


### 总结

因此根据图上的解释，封装了一个计算页面性能监控的基于performance的函数，用于返回性能数据。
可以根据自己的需求，在适合的时机执行函数，得到你需要的间隔时间duration。


```

/**
 * @file: performance.js
 * @author: Tiffany
 * @description: 页面性能统计
 */


var getPerformanceTiming = function () {
    var performance = window.performance;

    if (!performance) {
        // 当前浏览器不支持 performance
        return {msg: 'not suport performance'};
    }
    var t = performance.timing || {};

    var ns = t.navigationStart;
    var times = {
        // 间隔时间浏览器打开页面的耗时，
        // 在首屏时间点执行这段函数呢，那就是首屏的耗时；
        // 可以根据自己的业务需求，进行执行
        duration: new Date().getTime() - ns,
        // 页面渲染出第一个字符的耗时
        ttfb: t.responseStart - ns,
        // 响应结束到开始请求的时间，
        // 可以参考静态资源的加载时间是否过长，是否能有优化的时间点
        requestTime: t.responseEnd - t.requestStart
    };
   
   
    return times;
};

module.exports = {
    getPerformanceTiming: getPerformanceTiming
};


```





















