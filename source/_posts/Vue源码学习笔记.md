---
title: Vue源码学习笔记
date: 2019-07-04 11:33:21
tags: Vue源码
categories: Vue
---


> Vue源码学习笔记
> author: @TiffanysBear



## 准备工作
前序了解

### Flow 静态类型检查工具

类型推断：通过变量的使用上下文来推断出变量类型，然后根据这些推断来检查类型。

```javascript

/*@flow*/

function split(str) {
  return str.split(' ')
}

split(11)


```

类型注释：事先注释好我们期待的类型，Flow 会基于这些注释来判断。

```javascript

/*@flow*/


function add(x: number, y: number): number {
  return x + y
}

add('Hello', 11)


```

[点击了解更多Flow相关特性](https://flow.org/en/docs/types/)

<!-- more -->

## 源码目录设计
Vue.js 的源码都在 src 目录下，其目录结构如下。

```javascript
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码

```

### compiler
compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。

编译的工作可以在构建时做（借助 webpack、vue-loader 等辅助插件）；也可以在运行时做，使用包含构建功能的 Vue.js。显然，编译是一项耗性能的工作，所以更推荐前者——离线编译。

### core
core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。

### platform
Vue.js 是一个跨平台的 MVVM 框架，它可以跑在 web 上，也可以配合 weex 跑在 native 客户端上。platform 是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和 weex 上的 Vue.js。


### server
Vue.js 2.0 支持了服务端渲染，所有服务端渲染相关的逻辑都在这个目录下。注意：这部分代码是跑在服务端的 Node.js，不要和跑在浏览器端的 Vue.js 混为一谈。

服务端渲染主要的工作是把组件渲染为服务器端的 HTML 字符串，将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。

### sfc
通常我们开发 Vue.js 都会借助 webpack 构建， 然后通过 .vue 单文件来编写组件。

这个目录下的代码逻辑会把 .vue 文件内容解析成一个 JavaScript 的对象。

### shared
Vue.js 会定义一些工具方法，这里定义的工具方法都是会被浏览器端的 Vue.js 和服务端的 Vue.js 所共享的。


## Vue入口文件

Vue入口文件目录 vue/src/core/instance/index.js 


```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue

```

采用的是ES5的写法，并不是ES6的Class写法的优点，是因为：

1、使用混入Mixin的方式传入Vue，为Vue的原型prototype上增加方法。class难以实现这种方法
2、此种方式将代码模块合理划分，将扩展分散到多个模块中去实现，使得代码文件不会过于庞大，便于维护和管理。这个编程技巧以后可以用于代码开发实现中。


通过Mixin增加的原型方法：


```
initMixin(Vue) // _init
stateMixin(Vue) // $set、$delete、$watch
eventsMixin(Vue) // $on、$once、$off、$emit
lifecycleMixin(Vue) // _update、$forceUpdate、$destroy、
renderMixin(Vue) // $nextTick、_render 


```




### initGlobalAPI

在 vue/src/core/index.js 中，调用的initGlobalAPI(Vue)，是为Vue增加静态方法的，

在路径 vue/src/core/global-api/ 目录下的文件中，都是给Vue添加的静态方法


比如：

```javascript
Vue.use // 使用plugin
Vue.extend
Vue.mixin 
Vue.component 
Vue.directive 
Vue.filter

```



## new Vue 做了什么

从入口的文件看来，通过new关键字初始化，调用了

```javascript
// src/core/instance/index.js

this._init(options)

```

然后从Mixin增加的原型方法看，initMixin(Vue)，调用的是为Vue增加的原型方法_init

```javascript
// src/core/instance/init.js

Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    ....
    ....
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    
    ....
    ....
    
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
}


```


从上面的函数名看来，new vue所做的事情，就像一个流程图一样展开了，分别是

- 合并配置
- 初始化生命周期
- 初始化事件中心
- 初始化渲染
- 调用beforeCreate钩子函数
- init injections and reactivity（这个阶段属性都已注入绑定，而且被$watch变成reactivity，但是$el还是没有生成，也就是DOM没有生成）
- 初始化state状态（初始化了data、props、computed、watcher）
- 调用created钩子函数。

在初始化的最后，检测到如果有 el 属性，则调用 vm.$mount 方法挂载 vm，挂载的目标就是把模板渲染成最终的 DOM。

Vue代码初始化的主线逻辑非常分明，使得逻辑和流程非常清楚，这种编程方法值得学习。


## Vue实例挂载

实例挂载主要是$mount方法的实现，在 ` src/platforms/web/entry-runtime-with-compiler.js ` & `src/platforms/web/runtime/index.js ` 等文件中都有对Vue.prototype.$mount的定义:

```javascript
// vue/platforms/web/entry-runtime-with-compiler.js

Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}


```

$mount方法进来会先进行缓存，之后再进行覆盖重写，再重写的方法里面会调用之前缓存的mount方法，这种做法是因为，多个平台platform的mount方法不同，在入口处进行重写，使后续的多入口能够复用公用定义的mount方法（原先原型上的 $mount 方法在 src/platform/web/runtime/index.js 中定义）。

在$mount方法中，会先判断options中 el 是否存在，再判断 render （有template存在的条件下也需要有render函数），之后再是再判断template，会对template做一定的校验，最后使用 `compileToFunctions` 将template转化为` render ` 和 `staticRenderFns`.

compileToFunctions编译过程就放在下面文章中再详细解释。


mountComponent方法定义在 `src/core/instance/lifecycle.js`中，


```
// src/core/instance/lifecycle.js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted && !vm._isDestroyed) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}

```


从上面的代码可以看到，mountComponent 核心就是先实例化一个`渲染Watcher(字段isRenderWatcher)`，在它的回调函数中会调用 updateComponent 方法，在此方法中调用 vm._render 方法先生成虚拟 Node，最终调用 vm._update 更新 DOM。

Watcher 在这里起到两个作用，一个是初始化的时候会执行回调函数，另一个是当 vm 实例中的监测的数据发生变化的时候执行回调函数，这块儿我们会在之后的章节中介绍。

函数最后判断为根节点的时候设置 vm._isMounted 为 true， 表示这个实例已经挂载了，同时执行 mounted 钩子函数。 这里注意 vm.$vnode 表示 Vue 实例的父虚拟 Node，所以它为 Null 则表示当前是根 Vue 的实例。

因此接下来分析的重点在于：`vm._update` 和 `m._render`

## _render


## _update

















