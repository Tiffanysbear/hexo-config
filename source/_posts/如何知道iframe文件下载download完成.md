---
title: 如何知道iframe文件下载download完成
date: 2019-07-30 11:20:15
tags: 问题收集和解决
categories: JavaScript
---


> 如何知道iframe文件下载download完成
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

### 问题
当使用iframe作为文件下载的载体时，如何知道文件已经下载完毕。现有的iframe的onLoad方法具有兼容性问题，在chrome、IE下无法监听onLoad事件监听文件下载完毕，因为onLoad事件本身也是对iframe中的html结构的加载进度监听。

```
 var url = 'http://www.example.com/file.zip';
 var iframe = document.createElement('iframe');
 iframe.src = url;
 iframe.style.display = 'none';
 iframe.onload = function() {
     console.debug('start downloading...');
     document.body.removeAttribute(iframe);
 }
 document.body.appendChild(iframe);

```


当chrome、IE下时，如果HTTP文件头中包含Content-disposition: attachment；即下载文件的链接的话，不会触发这个事件onLoad事件。

这里说一下Content-disposition：

> Content-disposition 是 MIME 协议的扩展，MIME 协议指示 MIME 用户代理如何显示附加的文件。Content-disposition其实可以控制用户请求所得的内容存为一个文件的时候提供一个默认的文件名，文件直接在浏览器上显示或者在访问时弹出文件下载对话框。
> Content-Disposition为属性名disposition-type是以什么方式下载，如attachment为以附件方式下载disposition-parm为默认保存时的文件名服务端向客户端游览器发送文件时，如果是浏览器支持的文件类型，一般会默认使用浏览器打开，比如txt、jpg等，会直接在浏览器中显示
> 
> 注意事项：
> 1.当代码里面使用Content-Disposition来确保浏览器弹出下载对话框的时候。 response.addHeader('Content-Disposition', 'attachment');一定要确保没有做过关于禁止浏览器缓存的操作。
代码如下:
response.setHeader('Pragma', 'No-cache');
response.setHeader('Cache-Control', 'No-cache');
response.setDateHeader('Expires', 0);
> 不然会发现下载功能在opera和firefox里面好好的没问题，在IE下面就是不行


<!--more-->

### 解决思路一：利用cookie

后端将文件下载进度放在cookie中，通过轮询cookie的方式，对文件下载进度进行获取，判断文件是否已经下载完毕。
缺陷：
1、需要后端配合
2、如果客户端禁用了cookie，则该方案完全失效；在无痕浏览模式下，读取cookie，甚至代码报错。


### 解决思路二：添加header配置

```
 // 不让浏览器自动检测文件类型
 // 说明资料：http://drops.wooyun.org/tips/1166
 response.addHeader('X-Content-Type-Options', 'nosniff');
 // 提示浏览器不让其在frame或iframe中加载资源的文件内容
 // https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options
 response.addHeader('X-Frame-Options', 'deny');

```

但是在chorome v58版本将header的X-Frame-Options设为deny会报错。并且下载的时候网络连接会出现失败。


### 解决思路三：轮询监听readyState

定时器轮询监听readyState的状态，如果是 complete 或者 interactive 说明文件加载完成。

```
var timer = setInterval(function () {
    iframe = document.getElementById('iframedownload');
    var iframeDoc = iframe.contentDocument || iframe.contentWindow.document;
    // Check if loading is complete
    if (iframeDoc.readyState == 'complete' || iframeDoc.readyState == 'interactive') {
        loadingOff();
        clearInterval(timer);
        return;
    }
}, 4000);

```

该种方法比较好，因为不需要后端进行配合，且不依赖与cookie等变量带来的问题，且能实现我们的需求。
















