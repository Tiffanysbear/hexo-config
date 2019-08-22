---
title: 重新理解前端系列 — AMD、CMD
date: 2019-08-21 11:27:29
tags: AMD、CMD
categories: JavaScript
---

> 重新理解前端系列 — AMD、CMD
> author: [@TiffanysBear](https://tiffanysbear.github.io/)

本文主要是针对之前一些熟悉的前端概念，再次回顾的时候，结合自己的开发经验和使用，进行再次理解。经过了开发和线上使用之后，会有更为深刻的印象。对比requirejs源码分析，实现一个模块加载器，需要考虑哪些问题。


### 起源
其实对于AMD和CMD的不同，之前一直是拘泥在使用上的不同。没有深刻的认识为什么会有不同，其实主要是因为浏览器端和 Node 端不同性能特点和瓶颈带来的不同。

早期的js模块化主要用于浏览器端，主要的需求和瓶颈在于带宽，需要将js从服务端下载下来，从而带来的网络性能开销，因此主要是满足对于作用域、按需加载的需求。因此AMD（异步模块定义）的出现，适合浏览器端环境。

而后出现Node之后，主要的性能开销不再是网络性能，磁盘的读写和开销可以忽略不计；CMD的出现更符合Node 对于CommonJS的定义和理解，在运行时进行加载，引入时只是产生引用指向关系。

因此两者产生了不同的使用特点，在出现循环引用时，就产生了不同的现象。以下是针对 requirejs 源码部分的解读。如果有问题，欢迎提问纠正。

#### 1、动态加载一个js模块的方法，怎么保证异步和回调的执行

一先开始是需要判断环境，浏览器环境和webworker环境；如果是浏览器环境，通过`document.createElement ` 创建script标签，使用async属性使js能进行异步加载, IE等不兼容async字段的，通过监听 load 、 onreadystatechange 事件执行回调，监听脚本加载完成。

```
req.createNode = function (config, moduleName, url) {
    var node = config.xhtml ?
        document.createElementNS('http://www.w3.org/1999/xhtml', 'html:script') :
        document.createElement('script');
    node.type = config.scriptType || 'text/javascript';
    node.charset = 'utf-8';
    node.async = true; //创建script标签添加了async属性
    return node;
};
req.load = function (context, moduleName, url) { //用来进行js模块加载的方法
    var config = (context && context.config) || {},
    	node;
    if (isBrowser) { //在浏览器中加载js文件
    
        node = req.createNode(config, moduleName, url); //创建一个script标签
        
        node.setAttribute('data-requirecontext', context.contextName); //requirecontext默认为'_'
        node.setAttribute('data-requiremodule', moduleName); //当前模块名
        
        if (node.attachEvent &&
            !(node.attachEvent.toString && node.attachEvent.toString().indexOf('[native code') < 0) &&
            !isOpera) {
            
            useInteractive = true;
            
            node.attachEvent('onreadystatechange', context.onScriptLoad);
        } else {
            node.addEventListener('load', context.onScriptLoad, false);
            node.addEventListener('error', context.onScriptError, false);
        }
        node.src = url;
        
        if (config.onNodeCreated) { //script标签创建时的回调
            config.onNodeCreated(node, config, moduleName, url);
        }
        
        currentlyAddingScript = node;
        if (baseElement) { //将script标签添加到页面中
            head.insertBefore(node, baseElement);
        } else {
            head.appendChild(node);
        }
        currentlyAddingScript = null;
        
        return node;
    } else if (isWebWorker) { //在webWorker环境中
    	try {
            setTimeout(function () { }, 0);
            importScripts(url); //webWorker中使用importScripts来加载脚本
            
            context.completeLoad(moduleName);
    	} catch (e) { //加载失败
            context.onError(makeError('importscripts',
                'importScripts failed for ' +
                moduleName + ' at ' + url,
                e,
                [moduleName]));
    	}
    }
};


```


#### 2、怎么判断去加载js，怎么保证加载的顺序

通过 setTimeout 放入下一个队列中，保证加载顺序

```
//通过setTimeout的方式加载依赖，放入下一个队列，保证加载顺序
context.nextTick(function () {
	//Some defines could have been added since the
	//require call, collect them.
	intakeDefines();

	requireMod = getModule(makeModuleMap(null, relMap));

	//Store if map config should be applied to this require
	//call for dependencies.
	requireMod.skipMap = options.skipMap;

	requireMod.init(deps, callback, errback, {
		enabled: true
	});

	checkLoaded();
});
```

#### 3、require中的js文件是怎么判断已经loaded，怎么保证加载数据的数量是正确的？

依赖数量，是通过 depCount 来计算的，通过循环遍历，统计具体的依赖数量；


```
// ...
enable: function () {
	enabledRegistry[this.map.id] = this;
	this.enabled = true;

	//Set flag mentioning that the module is enabling,
	//so that immediate calls to the defined callbacks
	//for dependencies do not trigger inadvertent load
	//with the depCount still being zero.
	this.enabling = true;

	//enable每一个依赖
	each(this.depMaps, bind(this, function (depMap, i) {
		var id, mod, handler;

		if (typeof depMap === 'string') {
			//Dependency needs to be converted to a depMap
			//and wired up to this module.
			depMap = makeModuleMap(depMap,
				(this.map.isDefine ? this.map : this.map.parentMap),
				false,
				!this.skipMap);
			this.depMaps[i] = depMap; //获取的依赖映射

			handler = getOwn(handlers, depMap.id);

			if (handler) {
				this.depExports[i] = handler(this);
				return;
			}

			this.depCount += 1; //依赖项+1

			on(depMap, 'defined', bind(this, function (depExports) {
				if (this.undefed) {
					return;
				}
				this.defineDep(i, depExports); //加载完毕的依赖模块放入depExports中，通过apply方式传入require定义的函数中
				this.check();
			})); //绑定defined事件，同时将dep添加到registry中

			if (this.errback) {
				on(depMap, 'error', bind(this, this.errback));
			} else if (this.events.error) {
				// No direct errback on this module, but something
				// else is listening for errors, so be sure to
				// propagate the error correctly.
				on(depMap, 'error', bind(this, function (err) {
					this.emit('error', err);
				}));
			}
		}

		id = depMap.id;
		mod = registry[id];

		//跳过一些特殊模块，比如：'require', 'exports', 'module'
		//Also, don't call enable if it is already enabled,
		//important in circular dependency cases.
		if (!hasProp(handlers, id) && mod && !mod.enabled) {
			context.enable(depMap, this); //加载依赖
		}
	}));

	//Enable each plugin that is used in
	//a dependency
	eachProp(this.pluginMaps, bind(this, function (pluginMap) {
		var mod = getOwn(registry, pluginMap.id);
		if (mod && !mod.enabled) {
			context.enable(pluginMap, this);
		}
	}));

	this.enabling = false;

	this.check();
},
```

判断单个文件加载成功，是通过 checkLoaded 每间隔 50s 做一次轮询进行判断，变量 inCheckLoaded 作为标识；下面是 checkLoaded 函数：

```
function checkLoaded() {
	var err, usingPathFallback,
		waitInterval = config.waitSeconds * 1000,
		//It is possible to disable the wait interval by using waitSeconds of 0.
		expired = waitInterval && (context.startTime + waitInterval) < new Date().getTime(),
		noLoads = [],
		reqCalls = [],
		stillLoading = false,
		needCycleCheck = true;

	//Do not bother if this call was a result of a cycle break.
	if (inCheckLoaded) {
		return;
	}

	inCheckLoaded = true;

	//Figure out the state of all the modules.
	eachProp(enabledRegistry, function (mod) {
		var map = mod.map,
			modId = map.id;

		//Skip things that are not enabled or in error state.
		if (!mod.enabled) {
			return;
		}

		if (!map.isDefine) {
			reqCalls.push(mod);
		}

		if (!mod.error) {
			//If the module should be executed, and it has not
			//been inited and time is up, remember it.
			if (!mod.inited && expired) {
				if (hasPathFallback(modId)) {
					usingPathFallback = true;
					stillLoading = true;
				} else {
					noLoads.push(modId);
					removeScript(modId);
				}
			} else if (!mod.inited && mod.fetched && map.isDefine) {
				stillLoading = true;
				if (!map.prefix) {
					//No reason to keep looking for unfinished
					//loading. If the only stillLoading is a
					//plugin resource though, keep going,
					//because it may be that a plugin resource
					//is waiting on a non-plugin cycle.
					return (needCycleCheck = false);
				}
			}
		}
	});

	if (expired && noLoads.length) {
		//If wait time expired, throw error of unloaded modules.
		err = makeError('timeout', 'Load timeout for modules: ' + noLoads, null, noLoads);
		err.contextName = context.contextName;
		return onError(err);
	}

	//Not expired, check for a cycle.
	if (needCycleCheck) {
		each(reqCalls, function (mod) {
			breakCycle(mod, {}, {});
		});
	}

	//If still waiting on loads, and the waiting load is something
	//other than a plugin resource, or there are still outstanding
	//scripts, then just try back later.
	if ((!expired || usingPathFallback) && stillLoading) {
		//Something is still waiting to load. Wait for it, but only
		//if a timeout is not already in effect.
		if ((isBrowser || isWebWorker) && !checkLoadedTimeoutId) {
			checkLoadedTimeoutId = setTimeout(function () {
				checkLoadedTimeoutId = 0;
				checkLoaded();
			}, 50);
		}
	}

	inCheckLoaded = false;
}
```


#### 4、如果有循环引用，怎么判断出的，怎么解决的
这部分暂且还有点疑惑，先mark一下，之后再理解；

看到有个 breakCycle 函数，执行条件是 needCycleCheck 为 true，但是当 `!mod.inited && mod.fetched && map.isDefine` 模块未被初始化完成，但是已经获取过定义过之后，且 在 map.prefix 有前缀，会启动 breakCycle 检查；至于为什么要这么做，只能猜测是为了到模块require时循环引用打破轮询查询加载状态等待的问题，现在先留一个疑问。

```
function breakCycle(mod, traced, processed) {
	var id = mod.map.id;

	if (mod.error) {
		mod.emit('error', mod.error);
	} else {
		traced[id] = true;
		each(mod.depMaps, function (depMap, i) {
			var depId = depMap.id,
				dep = getOwn(registry, depId);

			//Only force things that have not completed
			//being defined, so still in the registry,
			//and only if it has not been matched up
			//in the module already.
			if (dep && !mod.depMatched[i] && !processed[depId]) {
				if (getOwn(traced, depId)) {
					mod.defineDep(i, defined[depId]);
					mod.check(); //pass false?
				} else {
					breakCycle(dep, traced, processed);
				}
			}
		});
		processed[id] = true;
	}
}
```


但是在CommonJs中时，存在依赖的情况下，因为存在的只是引用，代码执行是在实际调用时才发生，在文件的开头和结尾也会有变量标识是否加载完成。一旦某个模块出现循环依赖加载
，就只输出已经执行到的部分，还未执行的部分不会输出。

在ES6模块加载的循环加载情况下，ES6是动态引用的，不存在缓存值问题，而且模块里面的变量绑定所在的模块；不关心是否发生了循环加载，只是生成一个指向被加载模块的引用，需要开发者自己来保证真正取值的时候能够取到值。















