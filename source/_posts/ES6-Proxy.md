---
title: ES6-Proxy
date: 2019-05-22 11:17:14
tags: ES6
categories: ES6
---

## Proxy

### 1、使用proxy扩展构造函数
> Proxy参数说明：
> target：
> 用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）。
> handler：
> 一个对象，其属性是当执行一个操作时定义代理的行为的函数。

<!-- more -->

```javascript
function extend(sup,base) {
  var descriptor = Object.getOwnPropertyDescriptor(
    base.prototype,"constructor"
  );
  base.prototype = Object.create(sup.prototype);
  var handler = {
    construct: function(target, args) {
      var obj = Object.create(base.prototype);
      this.apply(target,obj,args);// 13行、14行
      return obj;
    },
    apply: function(target, that, args) {
      sup.apply(that,args);
      base.apply(that,args);
    }
  };
  var proxy = new Proxy(base,handler);
  descriptor.value = proxy;
  Object.defineProperty(base.prototype, "constructor", descriptor);
  return proxy;
}

var Person = function(name){
  this.name = name
};

var Boy = extend(Person, function(name, age) {
  this.age = age;
});

Boy.prototype.sex = "M";

var Peter = new Boy("Peter", 13); // new时开始执行handler的constructor
console.log(Peter.sex);  // "M"
console.log(Peter.name); // "Peter"
console.log(Peter.age);  // 13
```


> Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。
> apply() 方法调用一个具有给定this值的函数，以及作为一个数组（或类似数组对象）提供的参数。

因此，根据proxy的特性，可以实现拦截器的功能，其中，proxy支持的拦截操作一共有13种，如下2所示。

### 支持的操作
#### Proxy支持的拦截操作有以下13种

> - get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
> - set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
> - has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
> - deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
> - ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
> - getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
> - defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey,
> propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
> - preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
> - getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
> - isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
> - setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
> - apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object,
> ...args)、proxy.apply(...)。
> - construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。

具体的拦截方法的详细使用，[点击查看13种拦截方法详解](http://es6.ruanyifeng.com/#docs/proxy)。