---
title: FIS 插件机制
date: 2019-08-26 10:25:22
tags: FIS
categories: 插件
---

> FIS 插件机制
> author: [@TiffanysBear](https://tiffanysbear.github.io/)


当我们使用 FIS 插件的时候，有没有想过自己也开发一个基于 FIS 的插件，参与 FIS 打包编译的整个流程；那么问题就来了：

* FIS 的编译过程运行原理是怎样的呢？
* FIS 编译打包的过程有哪些？
* 怎么参与FIS 的打包编译过程？
* 怎么实现一个基于FIS的插件？
* FIS 是怎么引入自定义插件的？


基于以下的问题，从原理再进行慢慢分析，了解 FIS 编译的基本流程和原理，以及如何自己自定义一个 FIS 插件。


## 编译过程运行原理

fis的编译过程可以分为两个阶段： 单文件编译 和 打包。处理流程如下图，图片来自 FIS 官网：

![](https://github.com/Tiffanysbear/accumulation/raw/master/image/fis-1.png)

<!--more-->

### 单文件编译过程
从图上可以看出，单文件编译过程都是通过pipe管道进行的，并且在最初都建立有缓存，以提升编译效率，在单文件的处理过程中，又主要分为了以下的几个步骤：

> 
* parser(编译器)：将其他语言编译为标准js、css，比如将前端模板、coffee-script编译为js，将less、sass编译为css。
* preprocessor(标准预处理器)：在fis进行标准化处理之前进行某些修改，比如 支持image-set语法的预处理插件
* standard(标准化处理)：前面两项处理会将文件处理为标准的js、css、html语法，fis内核的标准化处理过程对这些语言进行 三种语言能力 扩展处理。这也就意味着，使用less、coffee等语法在fis系统中一样具备 资源定位、内容嵌入，依赖声明 的能力。该过程 不可扩展。
* postprocessor(标准后处理器)：对文件进行标准化之后的处理，比如利用依赖声明能力实现的 js包装器插件，可以获取js文件的依赖关系，并添加define包装。
* lint(可选)(校验器)：代码校验阶段，使用 fis release命令的 --lint 参数会调用该过程。
* test(可选)(测试器)：自动测试阶段，使用 fis release命令的 --test 参数会调用该过程。
* optimize(可选)(优化器)：代码优化阶段，使用 fis release命令的 --optimize 参数会调用该过程。fis内置的fis-optimizer-uglify-js插件和fis-optimizer-clean-css插件都是这类扩展。


### 打包过程

如果是文件的简单合并，可以使用 __inline 进行简单的内容嵌入，如果嵌入的内容中需要实时嵌入动态变量，可以考虑使用 bdtmpl 进行前端模块的编译和转换。

打包的原理是通过 FIS 的pack 配置，对文件资源进行合并等操作，最后产出关于文件打包信息到 map.json 文件中，并产生相应的打包文件。所以 FIS 的打包结果并 不会再嵌入到某个文件内，而是利用map.json中的数据进行运行时打包信息查询。

一、 在fis-conf.js中配置:

```
fis.config.merge({
    pack : {
        'aio.js' : ['a.js', 'b.js', 'c.js']
    }
});
```


二、 执行命令 fis release --pack --dest ./output

三、 进入output目录，查看map.json文件，得到内容：

```

{
    "res" : {
        "a.js" : {
            "uri" : "/a.js",
            "type" : "js",
            "pkg" : "p0"
        },
        "b.js" : {
            "uri" : "/b.js",
            "type" : "js",
            "pkg" : "p0"
        },
        "c.js" : {
            "uri" : "/c.js",
            "type" : "js",
            "pkg" : "p0"
        }
    },
    "pkg" : {
        "p0" : {
            "uri" : "/aio.js",
            "type" : "js",
            "has" : ["a.js", "b.js", "c.js"]
        }
    }
}
```

四、 将map.json交给某个前端或后端框架，当运行时需要“a.js”资源的时候，该框架应该读取map.json的信息，并根据一定的策略决定是否应该返回“a.js”资源所标记的“p0”包的uri。


因此也可以看出，FIS 团队强调的一点，打包只是资源的备份。

fis系统的打包过程提供了4个可扩展的处理过程，它们是：

1. prepackager(打包预处理器)：在打包前进行资源预处理。
2. packager(打包处理器)：对资源进行打包。默认的打包器就是收集资源表，建立map.json的过程
3. spriter(csssprite处理器)：对css进行sprites化处理
4. postpackager(打包后处理器)：打包之后对文件进行处理，通常用来将map.json转换成其他语言的文件，比如php


## 插件调用机制


fis的插件也是一个npm包，利用fis.require函数来加载。当我们在fis系统中加载一个插件的时候，会利用 nodejs的require向上查找机制 从 fis-kernel 模块出发，向上查找所需模块。

fis插件系统巧妙的利用了nodejs的require机制来实现其扩展机制。这意味着，要想扩展fis可以有 三种途径 ：

1、使用fis的用户，自己需要某种插件，可以在fis安装目录的 同级，安装自己扩展的插件。比如： ` npm install -g fis `   `npm install -g fis-parser-coffee-script`
2、使用 FIS 内置的插件，目前已经内置的插件包括：

* fis-kernel：fis编译机制内核
* fis-command-release：fis release命令的提供者，处理编译过程，并提供文件监听、自动上传等功能
* fis-command-install：fis install命令的提供者，用于从fis仓库下载组件、配置、框架、素材等资源
* fis-command-server：fis server命令的提供者，用于开启一个本地php-cgi服务器，对项目进行预览、调试。
* fis-optimizer-uglify-js：fis的优化插件，调用uglify-js对文件内容进行js压缩。
* fis-optimizer-clean-css：fis的优化插件，调用clean-css对文件内容进行css压缩。
* fis-postprocessor-jswrapper：fis的后处理器插件，用于对js文件进行包装，支持amd的define包装或者匿名自执行函数包装。


3、开发一个依赖于fis模块的npm包，并在这个包里定制所需要的插件。这种方式与上一条类似，也是将插件安装在fis的同级目录下。


### 可扩展时机
在整个编译流程可以扩展的点有以下，也就说说我们自己自定义的插件可以在下列的时机进行自己需求的定制，通过回调获取该阶段编译的结果，进行自定义配置。
编译阶段：

* parser
* preprocessor
* postprocessor
* lint
* test

打包阶段：

* prepackager
* packager
* spriter
* postpackager

### 自定义插件
自定义插件需要的，需要封装一个npm包，结合上面的可扩展时机，命名规则一般为：fis-[需要插入的时机名称]-[自定义插件名]，例如：fis-parse-my-css;

#### 编译阶段插件

1、在自定义插件的index.js中：

```
/*
 * fis
 * http://fis.baidu.com/
 */

'use strict';

var sass = require('node-sass');

module.exports = function(content, file, settings) {
    // content: 内容
    // file: 文件
    // settings: 现在的配置
    var opts = fis.util.clone(settings);
    opts.data = content;
    return sass.renderSync(opts);
};


```


2、插件配置调用
在fis-config.js中调用格式如下：

```
// vi fis-conf.js
// 文件后缀 .scss 的调用插件 my-sass 进行解析
fis.config.set('modules.parser.scss', 'my-sass');
fis.config.set('settings.parser.my-sass', {
    // my-sass 的配置
});
fis.config.set('roadmap.ext.scss', 'css'); // 由于 scss 文件最终会编译成 css，设置最终产出文件后缀为 css


```


3、发布npm包，这个可以参考我之前写过的一个文档，[从0到1发布一个npm包](https://juejin.im/post/5d5a4284518825388716035f)


#### 打包阶段插件

1、插件接口如此：

```
/**
 * 打包阶段插件接口
 * @param  {Object} ret      一个包含处理后源码的结构
 * @param  {Object} conf     一般不需要关心，自动打包配置文件
 * @param  {Object} settings 插件配置属性
 * @param  {Object} opt      命令行参数
 * @return {undefined}          
 */
module.exports = function (ret, conf, settings, opt) {
    // ret.src 所有的源码，结构是 {'<subpath>': <File 对象>}
    // ret.ids 所有源码列表，结构是 {'<id>': <File 对象>}
    // ret.map 如果是 spriter、postpackager 这时候已经能得到打包结果了，
    // 可以修改静态资源列表或者其他
}

```

以prepackager插件为例。prepackager即打包前需要对文件做某些处理，比如想在所有的html注释里面插入编译时间。

2、插件开发

我们为这个插件取名叫 append-build-time

```
<npm/global/path>/fis-prepackager-append-build-time
<npm/global/path>/fis-prepackager-append-build-time/index.js

```

在其 index.js 中：

```
module.exports = function(ret, conf, settings, opt) {
    fis.util.map(ret.src, function(subpath, file) {
        if (file.isHtmlLike) {
            var content = file.getContent();
            content += '<!-- build '+ (new Date())+'-->';
            file.setContent(content);
        }
    });
};
```

3、配置使用

```
// vi fis-conf.js
fis.config.set('modules.prepackager', 'append-build-time'); // packager阶段插件处理所有文件，所以不需要给某一类后缀的文件设置。
fis.config.set('settings.prepackager.append-build-time', {
    // settings
})
```

4、发布npm包，这个可以参考我之前写过的一个文档，[从0到1发布一个npm包](https://juejin.im/post/5d5a4284518825388716035f)


#### 小提示：
当然为了更快速的搞定一些小需求，可以把插件功能直接写到配置文件 fis-conf.js 中；

```
// vi fis-conf.js
fis.config.set('modules.postprocessor.js', function (content) {
    return content += '\n// build time: ' + Date.now();
});
```

注意：配置使用插件时，同一个扩展点可以配置多个插件，比如；

```
// 调用 fis-prepackager-a, fis-prepackager-b ...插件
fis.config.set('modules.prepackager', 'a,b,c,d');
// or
fis.config.set('modules.prepackager', ['a', 'b', 'c', 'd']);
// or
fis.config.set('modules.prepackager', [function () {}, function () {}])
```


具体的原理和封装步骤就讲到这么多，具体封装解决方案可以见 FIS 的官网 —— [封装解决方案](http://fex-team.github.io/fis-site/docs/dev/solution.html)


本文参考文档：

1、[插件扩展点列表](http://fex-team.github.io/fis-site/docs/more/extension-point.html) http://fex-team.github.io/fis-site/docs/more/extension-point.html
2、[插件调用机制](http://fex-team.github.io/fis-site/docs/more/how-plugin-works.html) http://fex-team.github.io/fis-site/docs/more/how-plugin-works.html
3、[编译过程运行原理](http://fex-team.github.io/fis-site/docs/more/fis-base.html) http://fex-team.github.io/fis-site/docs/more/fis-base.html


我的博客即将同步至腾讯云+社区，邀请大家一同入驻：https://cloud.tencent.com/developer/support-plan?invite_code=jam988nsd8ol



