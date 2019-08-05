---
title: getBoundingClientRect使用指南
date: 2019-05-22 14:22:10
tags: getBoundingClientRect
categories: JavaScript
---



> getBoundingClientRect使用指南
> author: @TiffanysBear

主要介绍getBoundingClientRect的基本属性，以及具体的使用场景和一些需要注意的问题。

### getBoundingClientRect

```
Element.getBoundingClientRect()

```

含义：
方法返回元素的大小及其相对于视口的位置。

值： 
返回值是一个 DOMRect 对象，这个对象是由该元素的 getClientRects() 方法返回的一组矩形的集合, 即：是与该元素相关的CSS 边框集合。

<!-- more -->

属性值：

* top: 元素上边距离页面上边的距离
* left: 元素右边距离页面左边的距离
* right: 元素右边距离页面左边的距离
* bottom: 元素下边距离页面上边的距离
* width: 元素宽度
* height: 元素高度


![图片一](https://github.com/Tiffanysbear/accumulation/raw/master/image/gbcr.png)

注意：
如果所有的元素边框都是空边框，那么这个矩形给该元素返回的 width、height 值为0，left、top值为第一个css盒子（按内容顺序）的top-left值。

当计算边界矩形时，会考虑视口区域（或其他可滚动元素）内的滚动操作，也就是说，当滚动位置发生了改变，top和left属性值就会随之立即发生变化（因此，它们的值是相对于视口的，而不是绝对的）。如果你需要获得相对于整个网页左上角定位的属性值，那么只要给top、left属性值加上当前的滚动位置（通过window.scrollX和window.scrollY），这样就可以获取与当前的滚动位置无关的值。

![图片二](https://github.com/Tiffanysbear/accumulation/raw/master/image/gbcr-2.png)


如图所示：
当页面的元素在浏览器的左上角时，得到的top和left值为负值，right和bottom值为正值。



### 应用场景一
#### 1、获取dom元素相对于网页左上角定位的距离
以前的写法是通过offsetParent找到元素到定位父级元素，直至递归到顶级元素body或html。

```javascript
// 获取dom元素相对于网页左上角定位的距离
function offset(el) {
    var top = 0;
    var left = 0;
    // 获取元素的位置还有getBoundingClientRect的方法
    // 从网上得知offset的兼容较差而且设置translate3D的y轴值给元素定位了y轴的距离后
    //会出现offsetTop为0
    do {
        top += el.offsetTop;
        left += el.offsetLeft;
    }
    while(el = el.offsetParent);// 存在兼容性问题，需要兼容
    
    
    return {
        top: top,
        left: left
    }
}
```

测试代码如下：

```javascript
var odiv = document.getElementsByClassName('markdown-body');
offset(a[0]); // {top: 271, left: 136}

```

现在根据getBoundingClientRect这个api，可以写成这样：

```javascript
var positionX = this.getBoundingClientRect().left + document.documentElement.scrollLeft;
var positionY = this.getBoundingClientRect().top + document.documentElement.scrollLeft;

```

### 应用场景二
#### 2、判断元素是否在可视区域内

```javascript
function isElView(el) {
    var top = el.getBoundingClientRect().top; // 元素顶端到可见区域顶端的距离
    var bottom = el.getBoundingClientRect().bottom; // 元素底部端到可见区域顶端的距离
    var se = document.documentElement.clientHeight; // 浏览器可见区域高度。
    
    if (top < se && bottom > 0) {
        return true;
    }
    else if (top >= se || bottom <= 0) {
        // 不可见
    }
    return false;
}


```


### 缺点

这个属性频繁计算会引发页面的重绘，可能会对页面的性能造成影响。






