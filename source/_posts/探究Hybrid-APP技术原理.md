---
title: 探究Hybrid-APP技术原理
date: 2019-05-24 15:35:04
tags: Hybrid-APP技术
categories: Hybrid-APP
---

> 探究Hybrid-APP技术原理
> author: [@TiffanysBear](https://tiffanysbear.github.io/)


## 背景

随着Web技术的发展和移动互联网的发展，Hybrid技术已经成为一种前端开发的主流技术方案。那什么是Hybrid App呢？

> Hybrid App（混合模式移动应用）是指介于web-app、native-app这两者之间的app，兼具" Native App良好用户交互体验的优势 "和" Web App跨平台开发的优势 "。
>  

总的来说，就是既具有APP的体验和性能，又具有Web灵活的开发模式和跨平台开发能力。

<!-- more -->

## 现有的技术方案

1、H5 + JSBridge，通过JSBridge完成H5和Native的通信，赋予H5一定的端能力。是一种基于WebView UI的解决方案。

2、React-Native，进一步通过JSbridge将js解析为虚拟DOM传递到Native，并使用原生进行渲染。

3、小程序解决方案，采用双线程的渲染机制，将渲染层WebView和逻辑层JavaScriptCore形成独立的模块，通过Native进行通信（setData），逻辑层的网络请求也会由Native进行转发。在UI方面，采用的是WebView和原生相结合的方式。


## 技术原理

本文将从jsbridge的原理、实现、双向通信、接入方式和H5的嵌入方式进行详细阐述。

### jsbridge的原理

客户端能对WebView中请求进行拦截，都有相应的API：

Android:

```javascript
// Android: shouldoverrideurlloading 
public boolean shouldOverrideUrlLoading(WebView view, String url){
	//读取到url后自行进行分析处理
	
	//如果返回false，则WebView处理链接url，如果返回true，代表WebView根据程序来执行url
	return true;
}

```

IOS:

```javascript
// IOS: shouldStartLoadWithRequest 
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType {
    NSURL *url = [request URL];
     
    NSString *requestString = [[request URL] absoluteString];
    //获取url scheme后自行进行处理
			

```

因此，在页面中可以通过iframe加载src的方式触发相应的捕获函数，在捕获函数中可以对url中的参数进行解析；此外，Android还可以通过重写OnJSPrompt方法，对调用Prompt进行拦截，同样能实现通信的目的。

示例：
调起ios端：

```javascript
function iosInvoke(scheme) {
    var elem = document.createElement('iframe');
    var body = document.body || document.getElementsByTagName('body')[0];
    elem.style.display = 'none';
    elem.src = scheme;
    body.appendChild(elem);
    setTimeout(function () {
        body.removeChild(elem);
        elem = null;
    }, 0);
}

```

调起android端：

```javascript
function androidInvoke(scheme) {
    var androidJsBridge = window.Bdbox_android_jsbridge;
    if (androidJsBridge && androidJsBridge.dispatch) {
        androidJsBridge.dispatch(scheme);
    } else {
        var re = window.prompt('BdboxApp:' + JSON.stringify({
            obj: 'Bdbox_android_jsbridge',
            func: 'dispatch',
            args: [scheme]
        }));
        return re;
    }
}
```

### 协议制定URL Scheme

URL Scheme是什么

> 由于苹果的app都是在沙盒中，相互是不能访问数据的。但是苹果还是给出了一个可以在app之间跳转的方法：URL Scheme。简单的说，URL Scheme就是一个可以让app相互之间可以跳转的协议。每个app的URL Scheme都是不一样的，如果存在一样的URL Scheme，那么系统就会响应先安装那个app的URL Scheme，因为后安装的app的URL Scheme被覆盖掉了，是不能被调用的。

设置URL Scheme

> xxxapp://communication?args=xx




### 如何进行双向通信
双向通信主要是H5和Native的双向通信过程以及参数传递、回调执行。

#### H5通知Native：
H5通知Native的方式主要有：

* 调用prompt/console/alert，调用时进行参数传递，端进行拦截重写
* URL Scheme跳转拦截，将参数放在请求URL上，详细的文章介绍 [URL Scheme](https://www.jianshu.com/p/eed01a661186)
* API挂载，通过Navtive获取js执行环境，将相应的api挂载在js上，供h5调用


#### Native通知H5：

* 回调机制，在向Native传递信息时，将回调函数也传递，Native在调用完成后，使用js执行环境执行回调函数

### 接入方式

* jsbridge的接入，端方面的jsbridge和h5方面的jsbridge

### 嵌入方式
h5的嵌入方式:

* 直接代码，直接将H5代码css、html、js放入端目录下，以file协议的方式访问，优点是访问快速，缺点是迭代不方便。
* 线上地址，以http协议访问，使用webview打开url形式，相较于代码嵌入的方式来说，速度比较慢，依赖网络传输速度；优点是迭代快速
























