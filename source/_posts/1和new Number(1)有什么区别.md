---
title: 1和new Number(1)有什么区别
date: 2019-06-06 15:43:31
tags: 包装对象
categories: JavaScript
---

> 1和new Number(1)有什么区别
> author: @Tiffanysbear 

总结，两者的区别就是原始类型和包装对象的区别。

### 什么是包装对象

对象Number、String、Boolean分别对应数字、字符串、布尔值，可以通过这三个对象把原始类型的值变成（包装成）对象:

```
var v1 = new Number(123);
var v2 = new String('abc');
var v3 = new Boolean(true);

```

我们来看下实际的v1、v2、v3是什么呢？

```
typeof v1;// "object"
typeof v2;// "object"
typeof v3;// "object"

v1 === 123; // false
v1 == 123; // true


```

可以理解的是，v1此时是对象，===比较的是内存地址，因此跟数字Number 123不相等；可是为什么v1 == 123得到的值会是true呢？

<!-- more -->

























