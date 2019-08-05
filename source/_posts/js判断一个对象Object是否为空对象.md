---
title: js判断一个对象Object是否为空对象
date: 2019-05-21 20:48:38
tags: js判断空对象的方法
categories: JavaScript
---

>  js判断空对象的方法
>  判断一个js对象是否是空对象isEmptyObject
>  author: @TiffanysBear

### 方法一：使用for...in遍历
```javascript
var isEmptyObject = function () {
	for (var i in this) {
		return false;
	}
	return true;
}
// 尽量不要使用object.prototype直接进行修改
// 否则会为继承时生成的对象新增不必要的可枚举属性
// 同时可被for-in枚举到
Object.defineProperty(Object.prototype, 'isEmptyObject', {
	writable: false,
	configurable: false,
	enumerable: false,
	value: isEmptyObject
});
```

<!-- more -->

### 方法二：使用JSON.stringify方法
```javascript
var isEmptyObject = function () {
	return JSON.stringify(obj) === '{}';
}

Object.defineProperty(Object.prototype, 'isEmptyObject', {
	writable: false,
	configurable: false,
	enumerable: false,
	value: isEmptyObject
});
```
### 方法三：使用ES6的Object.keys

```javascript
var isEmptyObject = function () {
	return Object.keys(a).length === 0;
}

Object.defineProperty(Object.prototype, 'isEmptyObject', {
	writable: false,
	configurable: false,
	enumerable: false,
	value: isEmptyObject
});
```
如果不支持Object.keys，采用如下的polyfill：

```javascript
if (!Object.keys) {
  Object.keys = (function () {
    var hasOwnProperty = Object.prototype.hasOwnProperty,
        hasDontEnumBug = !({toString: null}).propertyIsEnumerable('toString'),
        dontEnums = [
          'toString',
          'toLocaleString',
          'valueOf',
          'hasOwnProperty',
          'isPrototypeOf',
          'propertyIsEnumerable',
          'constructor'
        ],
        dontEnumsLength = dontEnums.length;

    return function (obj) {
      if (typeof obj !== 'object' && typeof obj !== 'function' || obj === null) {
	      throw new TypeError('Object.keys called on non-object');
      }

      var result = [];

      for (var prop in obj) {
        if (hasOwnProperty.call(obj, prop)) result.push(prop);
      }

      if (hasDontEnumBug) {
        for (var i=0; i < dontEnumsLength; i++) {
          if (hasOwnProperty.call(obj, dontEnums[i])) result.push(dontEnums[i]);
        }
      }
      return result;
    }
  })()
};

```