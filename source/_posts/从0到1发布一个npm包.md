---
title: 从0到1发布一个npm包
date: 2019-08-15 17:19:37
tags: npm
categories: JavaScript
---

> 从0到1发布一个npm包
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

最近在项目业务中有遇到一些问题，一些通用的方法或者封装的模块在PC、WAP甚至是APP中都需要使用，但是对于业务的PC、WAP、APP往往是不同的业务、不同的代码库中，尽管已经将公用的组件和方法抽离到各自公共common中，但是各个大业务大方向上的公用封装依然不能满足需求。

比如一个计算文档类型大小的方法，可能都同时存在于各个业务的common中，假设是有3处代码库中均有；如果此时的需求是将文档类型或者大小的方法进行一些修改，增加一种文档类型或者减少一种文档类型，那咱们是否是需要去共同修改上面的3处方法。这样做，很不利于代码的维护，浪费人力，增加了代码工作量。

那么，如何做到管理一些公共依赖的基础模块代码呢？这时候，封装发布一个npm包进行统一管理就是一个很好的办法了。

先po一下我在写这篇文章时，根据以下的步骤发布的一个简单封装的npm包以及github地址，大家可以先看：

[npm包：page-performance-monitor](https://www.npmjs.com/package/page-performance-monitor)
[github地址：page-performance-monitor，欢迎 star、issue](https://github.com/Tiffanysbear/page-performance-monitor)


下面，就从0开始讲起，如何从0到1发布一个npm包。先介绍一下什么是npm~

## npm

npm 是JavaScript 世界的包管理工具，并且是Node.js 平台的默认包管理工具。通过npm 可以安装、共享、分发代码，管理项目依赖关系。

[官网地址](https://www.npmjs.com/)

比如有一些非常通用的公用方法，抽象封装，剔除一些冗余的业务需求，可以封装在一个npm包中，提供给相应的多个业务去使用。

那么接下来就列举一下封装一个简单的封装步骤；

## 发布步骤

以我之前的博客中列举的页面性能监控工具performance为例，具体的[performance介绍](https://juejin.im/post/5d53a1056fb9a06b1d213ac7)可以点击链接，做一个简单的封装，满足基本的业务上的打点统计需求即可；后面也会讲到后续如何去封装一个高质量的npm包，比如加上一些example、测试test、完善README.md等，逐步去完善。大概是有以下几个步骤：

1、新建项目，准备需要发布的代码
2、准备package.json
3、注册npm账号、并登录
4、发布

其实发布的过程并不难，要发布一个好的质量高的npm包往往是取决于要封装的代码、以及对代码单测覆盖、demo案例、README介绍等

### 准备项目：

开始准备的步骤，从一个最基础的项目新建开始，都是在Mac的Linux环境上进行：


```
// 新建项目文件夹
 mkdir page-performance
 
 // 初始化npm，初始化package.json
 npm init
 
 // 准备好封装代码
 // 一般源码是放在src，通过其他打包工具生成的一般是在dist目录或者build目录
 mkdir src
 
 // 可以将自己需要的代码往src中添加了
 // 假设我们只需要发布一个index.js就好
 // ......
```


### 发布一个最简单的npm包：
1、先去[官网](https://www.npmjs.com/)注册一个账号，填写好账号、密码、邮箱
2、然后登录npm账号 `npm login`，如果你们公司有自己的默认npm仓库或者使用的淘宝镜像，注意需要指定一下仓库地址；`npm login --registry=https://registry.npmjs.org`

```
# 会依次让你输入用户名、密码、和邮箱
Username:  
Password:
Email: (this IS public) 
```

3、发布包 `npm publish --registry=https://registry.npmjs.org`

会提示+ page-performance-monitor@1.0.0 你的包名字和版本，那么说明就发布好了。

我在发布的时候遇到了两个小问题，记录一下，如果你们也有相同的问题，可以使用下面的解决办法：
1). 提示 publish Failed PUT 403


you must verify your email before publishing a new package: https://www.npmjs.com/email-edit : page-performance-monitor


![](https://github.com/Tiffanysbear/accumulation/raw/master/image/npm-1.png)

之前登录的邮箱需要验证，去注册邮箱中找到npm发的邮件，点击验证一下就行.

2）第二个问题是：You do not have permission to publish "page-performance". Are you logged in as the correct user? : page-performance
![](https://github.com/Tiffanysbear/accumulation/raw/master/image/npm-1.png)

提示是说你没有权限发布这个包，其实是因为你的这个包名字和已有的重复了，需要在 package.json 里面换一个包名就行。

到这里，一个简单的npm包就封装好了，如何确认自己的包确认好了呢？去官网的搜索框输入你的包名[搜一下](https://www.npmjs.com/package/page-performance-monitor)，找到你的就ok啦~


到这步，你就会发布一个简单的npm包啦，如果只是一个很小的需求的化，就完全够用了；但是如果想要发布一个质量好有各种小标签logo的，那么就需要如下的步骤进行一下优化。

### 优化npm包：

<!-- more -->

#### 1、代码环境依赖-线上线下环境

如果项目在线上线下使用的配置都不同的化，可以通过命令输入的不同，区分是debug模式还是生产production模式。

` process.env.NODE_ENV === 'production' `

在相应的package.json中的配置中，就需要加上 ` npm run build --mode production` 来进行区分。

#### 2、配置打包编译
好的一个npm包，往往需要不同的产出模式，比如利于script标签使用的iife模式，或者是采用amd、cmd等的打包方式进行export；或者需要采用babel进行转义，增加polyfill；或者你需要增加demo，为demo输出不同的样例，都需要使用配置打包编译。

目前常见的打包编译工具有webpack、rollup、fis、gulp等工具，相信也非常熟悉了；因为我的这个只是个简单的检测页面性能的工具方法，采用较为简单的适合工具库类型打包的rollup进行打包编译。

rollup.config.js配置如下：

```
/**
 * @file: rollup.config.js
 * @author: Tiffany
 */
// Rollup plugins
import resolve from 'rollup-plugin-node-resolve';
import commonjs from '@baidu/rollup-plugin-commonjs';
import babel from 'rollup-plugin-babel';
import uglify from 'rollup-plugin-uglify-es';
export default [
    {
        input: 'src/index.js',
        output: {
            file: 'dist/index.js',
            format: 'umd',
            name: 'Perf',
            legacy: true,
            strict: false,
            sourceMap: true
        },
        plugins: [
            resolve(),
            commonjs(),
            babel({
                runtimeHelpers: true,
                exclude: 'node_modules/**'
            }),
            uglify()
        ]
    }
];

```

配合babel的配置，如下：

```
{
    "presets": [
        [
            "latest",
            {
                "es2015": {
                    "modules": false
                }
            }
        ]
    ],
    "plugins": [
        "external-helpers"
    ]
}
```

然后就可以根据自己的需求，选择打包format的模式，产出自己需要的结果。大家也可以根据自己的项目需求、大小等，进行配置。

#### 3、增加单测

现在前端单测的库有很多，在这里就不再赘述；在这里采用的是 mocha + chai 断言库，因为这个库是运行在浏览器端，需要依赖于 JSDOM 中的 window 对象，因为采用了 JSDOM 库来实现 DOM 对象等的构建以及全局变量 window 的加入，以下是具体的配置：

```
// test/index.test.js

/**
 * @file: index.test.js
 * @author: zhoufang04
 * @description: mocha + chai test
 */

const expect = require('chai').expect;
const {JSDOM} = require('jsdom');
const perf = require('../dist/index.js');
const {window} = new JSDOM(`<!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width,initial-scale=1.0,minimum-scale=1.0, maximum-scale=1.0,user-scalable=no">
        <meta name="author" content="test">
        <title>performance test</title>
    </head>
    <body>
        <div id="values"></div>
        <div id="app"></div>
    </body>
    </html>`);

global.window = window;

describe('页面性能测试', function () {
    it('加载完成返回数据为对象', function () {
        expect(perf.getPerformanceTiming()).to.be.an('object');
    });
    it('返回耗时', function () {
        expect(perf.getPerformanceTiming().duration).to.be.an('number');
    });
    it('返回ttfb耗时', function () {
        expect(perf.getPerformanceTiming().ttfb).to.be.an('number');
    });
    it('返回requestTime耗时', function () {
        expect(perf.getPerformanceTiming().requestTime).to.be.an('number');
    });
});


```

运行`node ./node_modules/mocha/bin/mocha`，效果如下图：

![](https://github.com/Tiffanysbear/accumulation/raw/master/image/npm-3.png)


需要注意的是，本地node版本太低可能会导致mocha会有报错，这时候采用 nvm 升级一下node版本，再次运行就行。

#### 4、增加Example

增加example文件夹，里面可以通过对这个包的使用，增加一些Demo案例，让别人能更好的知道怎么使用这个库。


#### 5、完善README.md

在项目文件中增加README.md，提供使用方法、demo、注意事项等信息，方便别人使用，更容易让人明白。

可以看下在 `page-performance-monitor` 这个库中，我这边写的README.md，[点击链接可查看](https://github.com/Tiffanysbear/page-performance-monitor)



## 总结
上面的步骤就是如何从0到1封装的一个npm包，可以封装一个简单的适于业务快速开发的，也可以封装一个高质量封装一起使用；可以根据自己的业务需求、时间成本等自行选择。
















