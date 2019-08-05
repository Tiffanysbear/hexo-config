---
title: 移动端1px解决方案
date: 2019-05-21 20:47:19
tags: 移动端1px的实现
categories: JavaScript
---

> 移动端1px的实现
> author: @TiffanysBear

## 背景
在移动端，css中的1px并等于移动设备的1px，因为手机屏幕有不同的像素密度。window中的devicePixelRatio就是反应css中像素与真实像素的比例，也就是设备物理像素和设备独立像素的比例，也就是 devicePixelRatio = 物理像素 / 独立像素。所以造成了通过css设置1px，在移动端屏幕上会变粗。
### 解决方案一：使用伪类缩放
使用伪类缩放需要主要的是：<br>

 1. 设置全边框的时候，box-sizing要设置为border-box，否则伪元素上下左右各会多1px
 2. 需要将父元素设置为relative
 3. 注意 transform 的起点，上边距要用左上角，下边距用左下角

<!-- more -->

0.5px下边框
```css 
/* 下边框 */
.one-px-border2:after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    border-bottom: 1px solid red;
    transform: scaleY(.5);
    transform-origin: left bottom;
}
```
0.5px上边框
```css 
 /* 上边框 */
.one-px-border1:before {
    content: "";
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    border-top: 1px solid red;
    transform: scaleY(.5);
    transform-origin: left top;
}
```

0.5px全边框
```css 
.one-px-border3:after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    border: 1px solid red;
    transform-origin: left bottom;
    width: 200%;
    height: 200%;
    transform: scale(.5);
    /* box-sizing要设置为border-box，否则伪元素上下左右各会多1px */
    box-sizing: border-box;
    /* 设置圆角border等 */
    border-radius: 10px;
}
```

### 解决方案二：使用 -webkit-min-device-pixel-ratio
> The -webkit-device-pixel-ratio CSS media query. Use this to specify the screen densities for which this style sheet is to be used. The corresponding value should be either "0.75", "1", or "1.5", to indicate that the styles are for devices with low density, medium density, or high density screens, respectively. For example: The hdpi.css stylesheet is only used for devices with a screen pixel ration of 1.5, which is the high density pixel ratio.

使用less对常用的1px进行封装：
```javascript
// 高清屏 1px
@lignt-gray-color: #e7e7e7;
// 设置圆角（如果圆角大于0，则添加圆角的代码）
.border-radius(@borderRadius) when (unit(@borderRadius) > 0) {
    border-radius: @borderRadius;
    @media (-webkit-min-device-pixel-ratio: 2) {
        &:before {
            border-radius: unit(unit(@borderRadius) * 2, px);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.5) {
        &:before {
            border-radius: unit(unit(@borderRadius) * 2.5, px);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.75) {
        &:before {
            border-radius: unit(unit(@borderRadius) * 2.75, px);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 3) {
        &:before {
            border-radius: unit(unit(@borderRadius) * 3, px);
        }
    }
}

// border函数
.border(
    @borderWidth: 1px; 
    @borderStyle: solid; 
    @borderColor: @lignt-gray-color; 
    @borderRadius: 0) {
    position: relative;
    &:before {
        content: '';
        position: absolute;
        width: 98%;
        height: 98%;
        top: 0;
        left: 0;
        transform-origin: left top;
        -webkit-transform-origin: left top;
        box-sizing: border-box;
        pointer-events: none;
    }
    @media (-webkit-min-device-pixel-ratio: 2) {
        &:before {
            width: 198%;
            height: 198%;
            -webkit-transform: scale(.5);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.5) {
        &:before {
            width: 248%;
            height: 248%;
            -webkit-transform: scale(.4);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.75) {
        &:before {
            width: 273%;
            height: 273%;
            -webkit-transform: scale(1 / 2.75);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 3) {
        &:before {
            width: 298%;
            height: 298%;
            transform: scale(1 / 3);
            -webkit-transform: scale(1 / 3);
        }
    }
    .border-radius(@borderRadius);
    &:before {
        border-width: @borderWidth;
        border-style: @borderStyle;
        border-color: @borderColor;
    }
}

// 全边框函数及规则
.border-all(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth; @borderStyle; @borderColor; @borderRadius);
}

// 上边框函数及规则
.border-top(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth 0 0 0; @borderStyle; @borderColor; @borderRadius);
}

// 右边框函数及规则
.border-right(
	@borderWidth: 1px; 
	@borderStyle: solid;
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 @borderWidth 0 0; @borderStyle; @borderColor; @borderRadius);
}

// 下边框函数及规则
.border-bottom(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 0 @borderWidth 0; @borderStyle; @borderColor; @borderRadius);
}

// 左边框函数及规则
.border-left(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 0 0 @borderWidth; @borderStyle; @borderColor; @borderRadius);
}

// 上-右边框函数及规则
.border-tr(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth @borderWidth 0 0; @borderStyle; @borderColor; @borderRadius);
}

// 上-下边框函数及规则
.border-tb(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth 0; @borderStyle; @borderColor; @borderRadius);
}

// 上-左边框函数及规则
.border-tl(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth 0 0 @borderWidth; @borderStyle; @borderColor; @borderRadius);
}

// 右-下边框函数及规则
.border-rb(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 @borderWidth @borderWidth 0; @borderStyle; @borderColor; @borderRadius);
}

// 右-左边框函数及规则a
.border-rl(
	@borderWidth: 1px; 
	@borderStyle: solid;
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 @borderWidth; @borderStyle; @borderColor; @borderRadius);
}

// 下-左边框函数及规则
.border-bl(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(0 0 @borderWidth @borderWidth; @borderStyle; @borderColor; @borderRadius);
}


```

使用时：

```html
.one-px-border {
	// 使用less函数.border
	.border(1px, solid, red);
	// 使用less函数.border-radius
	.border-radius(20px);
}
```

### 解决方案三：使用图片
切图，使用图片结合border-image进行css样式设置
```html
.border-bottom-1px {
	border-width: 0 0 1px 0;
	border-image: url(linenew.png) 0 0 2 0 stretch;
}
```
缺点：不够灵活，换颜色需要换图片

### 解决方案四：使用box-shadow模拟

```css
.box-shadow-1px {
    width: 100px;
    height: 100px;
    box-shadow: inset 0px -1px 1px -1px red;
} 
```

