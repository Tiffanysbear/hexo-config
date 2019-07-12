---
title: JavaScript编写高质量的68个有效方法
date: 2019-05-21 20:54:50
tags: Effective JavaScript
categories: Javascript
---
> Effective JavaScript编写高质量的68个有效方法
> author：@Tiffanysbear 
> description: 读书笔记


### 第一条：了解你使用的javascript
1. 严格模式`use strict`

* 不允许重定义arguments变量
* 只有在脚本或函数的顶部才生效
* 不要将进行严格模式检查的代码和非严格模式的代码进行打包压缩
* 可通过立即调用函数隔离严格与非严格区域，单独隔离作用域
* 编写库时，开启严格检查，兼容性更强


```javascript
(function () {
	"use strict";
	function f() {
		// code
	}
})();
```

<!-- more -->

### 第二条：理解js浮点数
1. 位运算会将数字转换为32位大端的2的补码表示的整数，8表示为0000 0000 0000 0000 0000 0000 0000 1000，可通过(8).toString(2) // '1000'，指定基数进行转换
2. 0.1 + 0.2 不等于0.3
3. 整数运算，计算只适用于$$(-2)^{53} - 2^{53}$$

### 第三条：隐式转换

1. 3 + true // 4
2. "2" + 3 // "23"
3. 2 + "3" // "23"
4. 1 + "2" + 3 // "123"
5. "17" * 3 // 51
6. x === NaN // false
7. 对象的隐式转换，会调用自身的toString()转换为字符串或者valueOf()方法转换为数字，问题就在于调用这两个方法的优先级。具有vauleOf方法的对象应该实现toString方法，返回一个valueOf方法产生的数字字符串表示。
    
    * 存在toString方法，就调用toString方法
    * 存在valueOf方法，就调用valueOf方法
    * 同时存在toString、valueOf方法，优先调用valueOf方法

    
8. 真值转换，js中有7个假值：false、0、-0、""、NaN、null、undefined
9. 检测一个值是否是未定义的值，应该使用typeof或者与undefined进行比较，而不是直接if(!x)


















