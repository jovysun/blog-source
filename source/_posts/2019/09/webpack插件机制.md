---
title: webpack插件机制
permalink: webpack-plugin-mechanism
tags:
  - webpack
categories:
  - 工具
date: 2019-09-25 11:45:04
updated: 2019-09-25 11:45:04
---

# 概述
想要编写一个webpack的插件demo很简单，只要按照官方给的范式就好。想要实际编写一个生产用的插件，那就得深入了解插件机制。例如，apply方法干嘛用的，有哪些周期钩子，同步异步的钩子怎么触及（tap）。
<!-- more -->

# 详述
## apply方法

一个框架（对象）想要具有良好的扩展性，通常都会暴露一个接口，通过这个入口方法来整合第三方提供的功能模块，这就是插件系统。例如 jQuery 提供的`extend`方法，Vue 提供的`install`方法，webpack 提供的`apply`方法。因此，编写 webpack 插件需要定义 apply 方法。

## tapable
插件不同于 loader，loader 是针对具体模块的处理，而插件是针对 webpack 整个编译过程，因此，我们需要 webpack 的生命周期函数，通常也叫钩子 hooks。webpack 中为插件提供钩子机制的是一个抽离出来的核心工具库 tapable，它提供了抽象类Tapble和各种钩子类：

```js
// node_modules/tapable/README.md
const {
  SyncHook,
  SyncBailHook,
  SyncWaterfallHook,
  SyncLoopHook,
  AsyncParallelHook,
  AsyncParallelBailHook,
  AsyncSeriesHook,
  AsyncSeriesBailHook,
  AsyncSeriesWaterfallHook
} = require('tapable');
```
### 钩子类使用示例

定一个类，申明具体类型的钩子实例：
``` js
class Car {
	constructor() {
		this.hooks = {
			accelerate: new SyncHook(["newSpeed"]),
			brake: new SyncHook(),
			calculateRoutes: new AsyncParallelHook(["source", "target", "routesList"])
		};
	}

	/* ... */
}
```
同步钩子使用`tap`方法添加消费者：
``` js
const myCar = new Car();

// Use the tap method to add a consument
myCar.hooks.brake.tap("WarningLampPlugin", () => warningLamp.on());

// 接受传参
myCar.hooks.accelerate.tap("LoggerPlugin", newSpeed => console.log(`Accelerating to ${newSpeed}`));
```
异步钩子，除了用`tap`还可以用`tapPromise`和`tapAsync`方法添加消费者：
``` js
myCar.hooks.calculateRoutes.tapPromise("GoogleMapsPlugin", (source, target, routesList) => {
	// return a promise
	return google.maps.findRoute(source, target).then(route => {
		routesList.add(route);
	});
});
myCar.hooks.calculateRoutes.tapAsync("BingMapsPlugin", (source, target, routesList, callback) => {
	bing.findRoute(source, target, (err, route) => {
		if(err) return callback(err);
		routesList.add(route);
		// call the callback
		callback();
	});
});

// You can still use sync plugins
myCar.hooks.calculateRoutes.tap("CachedRoutesPlugin", (source, target, routesList) => {
	const cachedRoute = cache.get(source, target);
	if(cachedRoute)
		routesList.add(cachedRoute);
})
```
声明这些钩子的类可以用`call`、`promise`或者`callAsync`调用他们：

``` js
class Car {
	/* ... */

	setSpeed(newSpeed) {
		this.hooks.accelerate.call(newSpeed);
	}

	useNavigationSystemPromise(source, target) {
		const routesList = new List();
		return this.hooks.calculateRoutes.promise(source, target, routesList).then(() => {
			return routesList.getRoutes();
		});
	}

	useNavigationSystemAsync(source, target, callback) {
		const routesList = new List();
		this.hooks.calculateRoutes.callAsync(source, target, routesList, err => {
			if(err) return callback(err);
			callback(null, routesList.getRoutes());
		});
	}
}
```
[更多介绍](https://github.com/webpack/tapable)

## Compiler

Compiler 模块是 webpack 的支柱引擎，它通过 CLI 或 Node API 传递的所有选项，创建出一个 compilation 实例。它扩展(extend)自 Tapable 类，以便注册和调用插件。大多数面向用户的插件首先会在 Compiler 上注册。

compiler 对象是 webpack 的编译器对象，compiler 对象会在启动 webpack 的时候被一次性的初始化，compiler 对象中包含了所有 webpack 可自定义操作的配置，例如 loader 的配置，plugin 的配置，entry 的配置等各种原始 webpack 配置等，在 webpack 插件中的自定义子编译流程中，我们肯定会用到 compiler 对象中的相关配置信息，我们相当于可以通过 compiler 对象拿到 webpack 的主环境所有的信息。

Compiler 对象扩展自 Tapable 类，提供了编译器的整个[生命周期钩子](https://webpack.docschina.org/api/compiler-hooks/)，常用的有`done`（编译(compilation)完成）、`emit`（生成资源到 output 目录之前）等。

## Compilation

Compilation 模块会被 Compiler 用来创建新的编译（或新的构建）。compilation 实例能够访问所有的模块和它们的依赖（大部分是循环依赖）。它会对应用程序的依赖图中所有模块进行逐个编译(literal compilation)。在编译阶段，模块会被加载(loaded)、封存(sealed)、优化(optimized)、分块(chunked)、哈希(hashed)和重新创建(restored)。

Compilation 类扩展(extend)自 Tapable，提供了一次编译的[生命周期钩子](https://webpack.docschina.org/api/compilation-hooks/)。

## 事件钩子添加消费者

对于各种钩子的触及，有三个 API：同步的`tap`，异步的`tapAsync`和`tapPromise`。使用示例如下：

### tap

```js
class HelloCompilationPlugin {
  apply(compiler) {
    // tap(触及) 到 compilation hook，而在 callback 回调时，会将 compilation 对象作为参数，
    compiler.hooks.compilation.tap('HelloCompilationPlugin', compilation => {
      // 现在，通过 compilation 对象，我们可以 tap(触及) 到各种可用的 hooks 了
      compilation.hooks.optimize.tap('HelloCompilationPlugin', () => {
        console.log('正在优化资源。');
      });
    });
  }
}

module.exports = HelloCompilationPlugin;
```

### tapAsync

在我们使用 tapAsync 方法 tap 插件时，我们需要调用 callback，此 callback 将作为最后一个参数传入函数。

```js
class HelloAsyncPlugin {
  apply(compiler) {
    compiler.hooks.emit.tapAsync(
      'HelloAsyncPlugin',
      (compilation, callback) => {
        // 做一些异步的事情……
        setTimeout(function() {
          console.log('Done with async work...');
          callback();
        }, 1000);
      }
    );
  }
}

module.exports = HelloAsyncPlugin;
```

### tapPromise

在我们使用 tapPromise 方法 tap 插件时，我们需要返回一个 promise，此 promise 将在我们的异步任务完成时 resolve。

```js
class HelloAsyncPlugin {
  apply(compiler) {
    compiler.hooks.emit.tapPromise('HelloAsyncPlugin', compilation => {
      // 返回一个 Promise，在我们的异步任务完成时 resolve……
      return new Promise((resolve, reject) => {
        setTimeout(function() {
          console.log('异步工作完成……');
          resolve();
        }, 1000);
      });
    });
  }
}

module.exports = HelloAsyncPlugin;
```

[更多介绍](https://webpack.docschina.org/contribute/writing-a-plugin/#%E5%BC%82%E6%AD%A5%E4%BA%8B%E4%BB%B6%E9%92%A9%E5%AD%90)

# 参考
