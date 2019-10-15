---
title: 《CSS世界》第六章 流的破坏与保护总结
date: 2018-10-11 11:19:26
tags: 布局
categories: HTML && CSS
---

> 《CSS世界》第六章 流的破坏与保护总结
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

## float属性

### float的本质与特性
> 1.包裹性（包含包裹和自适应）

> 2.块状化并格式化上下文

> 3.破坏文档流

> 4.没有任何margin合并

### float的作用机制
> float属性使父元素高度塌陷，为了实现文字环绕效果。高度塌陷是为了让跟随的内容可以和浮动元素在一个水平线上。（行框盒子在正常定位状态下只会跟随浮动元素，不会发生重叠）

### clear属性
> clear属性让自身不能和前面的浮动元素相邻。clear属性只有块级元素才有效。

> 如果clear: both;元素前面的元素就是float元素，则margin-top负值即使设置成-9999px，也没有任何效果；

> clear: both; 后面的元素依旧可能会发生文字环绕现象。

<!-- more -->

## BFC

### BFC定义

> 通过一些特定的手段形成的封闭空间，即BFC元素内部不会影响外部的元素。可以用来防止margin重叠，清楚浮动防止父元素高度坍塌。

#### 触发BFC条件

- <html>根元素；
- float的值不为none；
- overflow的值为auto、scroll或hidden；
- display的值为table-cell、table-caption和inline-block中任何一个；
- position的值不为relative和static；

#### 各个BFC优缺点
- float，浮动元素本身BFC化，然而浮动元素有破坏性和包裹性，失去了元素本身的流体自适应行。
- position: absolute; 脱离了文档流。
- overflow: hidden; 容器盒子外的元素可能会被隐藏掉。
- display: inline-block; IE6、7下完美，因为即BFC化又保持block也行，保留了流体特性。但在其他浏览器下会让元素尺寸包裹收缩。

#### overflow
> overflow裁剪的边界是border box的内边缘，而非padding box的内边缘。

> overflow-x和overflow-y属性中一个值设置为visible而另一个设置为scroll、auto或者hidden，则visible的样式表现会如同auto。除非 overflow-x和overflow-y 属性值都是visible，否则会当成auto来解析。

```
// 单行文字省略号
.ell {
	text-overflow: ellipsis;
	white-space: nowarp;
	overflow: hidden;
}

// 多行文字省略号
.ell-rows-2 {
	display: -webkit-box;
	-webkit-box-orient: vertical;
	-webkit-line-clamp: 2;
}
```
##### overflow与锚点定位
锚点定位行为的触发条件

- URL地址中的锚链与锚点元素对应（a标签以及name属性）并有交互行为
- 可focus的锚点元素处于focus状态

> 锚点定位的本质通过改变**容器**滚动高度或者宽度实现的。锚点定位发生在普通容器元素上，定位行为是由内而外的。

> 设置了overflow: hidden;的元素也是可以滚动的，只是滚动条不见了而已。

## 绝对定位

### 绝对定位特性
- 块状化
- 破坏性
- 块状格式上下文
- 包裹性、自适应性

### absolute的包含块

- 根元素被称为“初始包含块”，其尺寸等同于浏览器可是窗口的大小。
- 对于其他元素，如果该元素的position是relative或者static。则“包含块”由最近的块容器祖先盒的content box边界形成。
- 如果元素position: fixed;则包含块是初始包含块。
- 如果元素position: absolute，则包含块由最近的position不为static的祖先元素建立，由该祖先的padding box边界形成。

### 无依赖absolute定位的相对特性
> absolute是非常独立的CSS属性值，其样式和行为表现不依赖其他任何CSS属性就可以完成。单独设置position: absolute; 本质上是相对定位，只不过不占据CSS流的尺寸空间而已。

> 可以利用该特性实现“图片和文字水平对齐”，“表单提示词”等效果。

### absolute与text-align
> text-align会改变absolute元素的位置，本质是“幽灵空白节点”和“无依赖绝对定位”共同作用的结果，具体就是由于绝对定位元素不占据CSS流中的尺寸空间，表现为一个“空白节点”，这时text-align使该节点居中，因此效果就是绝对定位元素偏右了。**注意，只有原本是内联水平的元素才有这种情况**

### absolute与overflow
> 绝对定位元素不总是被父级overflow属性剪裁，尤其当overflow在绝对定位元素及其包含块之间的时候。

### absolute与clip
> 当position: fixed;overflow属性就不可用了，除非根元素。这时可以使用clip来进行剪裁。

```
// 这种方式既满足视觉上的隐藏，屏幕阅读器等辅助设备又支持很好（display: none;不识别或者不可focus）。移动端中可以使用透明度为0.
.clip {
	position: absolute;
	clip: rect(0 0 0 0);
}
```

> clip隐藏仅仅决定了那部分可见，非可见部分不响应点击事件等；虽然视觉上隐藏了，但是元素的尺寸还是不变的，在IE、firfox中抹掉了不可见区域对布局的影响，chrome没有这种问题。

### absolute的流体特性
> 当对立方向同时发生定位时，表现为格式化宽度，自适应包含块的padding box。具备自适应性。

> margin: auto;可以让绝对定位元素居中。条件是对立方向同时发生定位。

## 相对定位
> 相对定位的left/top等的百分比是相对于包含块计算的，而不是相对自身。

> 对立方向同时发生定位时，只有一个方向的定位属性起作用。

## 固定定位
> 与无依赖绝对定位相同，也存在无依赖固定定位。