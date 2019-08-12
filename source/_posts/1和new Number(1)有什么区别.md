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

那这就是包装对象在使用时的问题了。再来理解一下什么是原始类型。

### 什么是原始类型

比如123这类就是原始类型，原始类型并不是一个对象，因此并没有对象具有的一些属性和方法；但是为什么能调用(123).toFixed()这些方法呢？

原因就是这些方法都是像包装对象"借用"来的，toFixed方法是在Number对象原型上的方法。

```
(123).toFixed === Number.prototype.toFixed // true
"123".indexOf === String.prototype.indexOf // true
```


### JS求值

JS在求值运算时，总是会求出原始资料的值，而不是用对象。如下面的例子：

```
var a = new Number(122);
var b = a + 33; // 155
typeof b; // number
```

但是要注意 `new Boolean` 的用法，只有当 new Boolean 的参数值为 null 或者 undefined 时，求值转换的原始资料的值才是false，其他情况都是true；

```
!!(new Boolean(false)) // true
```

所以尽量不要使用 new Boolean 这个包装对象进行赋值，否则会产生一些误会。


### 运算时调用 valueOf 和 toString 的优先级

先说下结论：

> 1、进行对象转换时（alert(e2)）,优先调用 toString 方法，如没有重写 toString 将调用 valueOf 方法，如果两方法都不没有重写，但按 Object 的 toString 输出。
> 2、进行强转字符串类型时将优先调用 toString 方法，强转为数字时优先调用 valueOf。
> 3、在有运算操作符的情况下，valueOf的优先级高于toString。

以下是三个例子

第一个：

```
let e2 = {
    n : 2,
    toString : function (){
        console.log('this is toString')
        return this.n
    },
    valueOf : function(){
        console.log('this is valueOf')
        return this.n*2
    }
}
alert(e2) //  2  this is toString
alert(+e2)  // 4 this is valueOf
alert(''+e2) // 4 this is valueOf
alert(String(e2)) // 2 this is toString
alert(Number(e2)) // 4 this is valueOf
alert(e2 == '4') // true  this is valueOf
alert(e2 === 4) //false ===操作符不进行隐式转换


```


第二个：

```
let e3 = {
    n : 2,
    toString : function (){
        console.log('this is toString')
        return this.n
    }
}
alert(e3) //  2  this is toString
alert(+e3)  // 2 this is toString
alert(''+e3) // 2 this is toString
alert(String(e3)) // 2 this is toString
alert(Number(e3)) // 2 this is toString
alert(e3 == '2') // true  this is toString
alert(e3 === 2) //false  ===操作符不进行隐式转换
```


第三个：

```
Object.prototype.toString = null; 
let e4 = {
    n : 2,
    valueOf : function(){
        console.log('this is valueOf')
        return this.n*2
    }
}
alert(e4) //  4 this is valueOf
alert(+e4)  // 4 this is valueOf
alert(''+e4) // 4 this is valueOf
alert(String(e4)) // 4 this is valueOf
alert(Number(e4)) // 4 this is valueOf
alert(e4 == '4') // true  this is valueOf
alert(e4 === 4) //false  ===操作符不进行隐式转换

```













