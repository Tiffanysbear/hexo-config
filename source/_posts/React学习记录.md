---
title: React学习记录
date: 2019-06-18 11:49:06
tags: React
categories: React
---

> React学习记录
> 学习基础React基本API，深化react的细节，学会思考和提问。
> author: @TiffanyBear




### content
1、React DOM 在渲染所有输入内容之前，默认会进行转义。它可以确保在你的应用中，永远不会注入那些并非自己明确编写的内容。所有的内容在渲染之前都被转换成了字符串。这样可以有效地防止 XSS（cross-site-scripting, 跨站脚本）攻击。


2、JSX 表示对象
Babel 会把 JSX 转译成一个名为 React.createElement() 函数调用。react元素

<!-- more -->

3、使用String字符串化

```
<div> My JavaScript variable is {String(myVariable)}. </div> 

```

4、“纯函数”，因为该函数不会尝试更改入参，且多次调用下相同的入参始终返回相同的结果。

5、React 非常灵活，但它也有一个严格的规则：
<font color="red">所有 React 组件都必须像纯函数一样保护它们的 props 不被更改。</font>

问题：需要严格保护props的原因是什么？


6、尽管 this.props 和 this.state 是 React 本身设置的，且都拥有特殊的含义，但是其实你可以向 class 中随意添加不参与数据流（比如计时器 ID）的额外字段。

7、State 的更新可能是异步的。出于性能考虑，React 可能会把多个 setState() 调用合并成一个调用。
因为 this.props 和 this.state 可能会异步更新，所以你不要依赖他们的值来更新下一个状态。

例如，此代码可能会无法更新计数器：

```javascript
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

要解决这个问题，可以让 setState() 接收一个函数而不是一个对象。这个函数用上一个 state 作为第一个参数，将此次更新被应用时的 props 做为第二个参数：

```javascript
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

8、数据是向下流动的

9、事件处理

* React 事件的命名采用小驼峰式（camelCase），而不是纯小写。
* 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串。


```javascript
<button onClick={activateLasers}>
  Activate Lasers
</button>

```

在 React 中另一个不同点是你不能通过返回 false 的方式阻止默认行为。你必须显式的使用 preventDefault；

```javascript
function ActionLink() {
  function handleClick(e) {
    e.preventDefault();
    console.log('The link was clicked.');
  }

  return (
    <a href="#" onClick={handleClick}>
      Click me
    </a>
  );
}

```


10、JSX 回调函数中的 this，在 JavaScript 中，class 的方法默认不会绑定 this。如果你忘记绑定 this.handleClick 并把它传入了 onClick，当你调用这个函数的时候 this 的值为 undefined。

```javascript
// 为了在回调中使用 `this`，这个绑定是必不可少的
this.handleClick = this.handleClick.bind(this);

// or 在模板中
<button onClick={this.handleClick.bind(this)}>
    Click me
</button>

```


11、阻止组件渲染
可以让 render 方法直接返回 null，而不进行任何渲染。在组件的 render 方法中返回 null 并不会影响组件的生命周期。依旧会按照生命周期执行相应的函数方法。

12、key值：

* 帮助 React 识别哪些元素改变了，比如被添加或删除，不建议使用索引来用作 key 值，如果列表项目的顺序可能会变化。正确的key 应该在数组的上下文中被指定。一个好的经验法则是：在 map() 方法中的元素需要设置 key 属性。
* 数组元素中使用的 key 在其兄弟节点之间应该是独一无二的。然而，它们不需要是全局唯一的。当我们生成两个不同的数组时，我们可以使用相同的 key 值。
* key 会传递信息给 React ，但不会传递给你的组件。如果你的组件中需要使用 key 属性的值，请用其他属性名显式传递这个值


13、状态提升
通常，多个组件需要反映相同的变化数据，这时我们建议将共享状态提升到最近的共同父组件中去。


14、React ref
引用

15、错误边界
部分 UI 的 JavaScript 错误不应该导致整个应用崩溃，为了解决这个问题，React 16 引入了一个新的概念 —— 错误边界。

错误边界是一种 React 组件，这种组件可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI，而不是渲染那些崩溃了的子组件树。错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误。

注意
错误边界无法捕获以下场景中产生的错误：

* 事件处理（了解更多）
* 异步代码（例如 setTimeout 或 requestAnimationFrame 回调函数）
* 服务端渲染
* 它自身抛出来的错误（并非它的子组件）
* 
如果一个 class 组件中定义了 static getDerivedStateFromError() 或 componentDidCatch() 这两个生命周期方法中的任意一个（或两个）时，那么它就变成一个错误边界。当抛出错误后，请使用 static getDerivedStateFromError() 渲染备用 UI ，使用 componentDidCatch() 打印错误信息。

[例子Demo](https://zh-hans.reactjs.org/docs/error-boundaries.html)




### 思考问题：
1、需要严格保护props的原因是什么？





