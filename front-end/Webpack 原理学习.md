# Webpack 原理学习

## 源码常用函数

`__webpack_require__` 加载文件

```javascript
// The require function
function __webpack_require__(moduleId) {
 		// Check if module is in cache
 		if(installedModules[moduleId]) {
 			return installedModules[moduleId].exports;
 		}
 		// Create a new module (and put it into the cache)
 		var module = installedModules[moduleId] = {
 			i: moduleId,
 			l: false,
 			exports: {}
 		};
 		// Execute the module function
 		modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
 		// Flag the module as loaded
 		module.l = true;
 		// Return the exports of the module
 		return module.exports;
 	}
```



`__webpack_require__.r` 给exports标注为ES6模块(一旦用了import/export default)

```javascript
 	// define __esModule on exports
__webpack_require__.r = function(exports) {
 		if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
 			Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
 		}
 		Object.defineProperty(exports, '__esModule', { value: true });
 	};
```



`__webpack_require__.d` 给export加上某个属性，并且配置enumerable和get

```javascript
// define getter function for harmony exports
__webpack_require__.d = function(exports, name, getter) {
 		if(!__webpack_require__.o(exports, name)) {
 			Object.defineProperty(exports, name, { enumerable: true, get: getter });
 		}
 	};
```

`__webpack_require__.o` 查看某个元素是否存在

```javascript
// Object.prototype.hasOwnProperty.call
__webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
```

`__webpack_require__.n` 获取default export，兼容ES6和commonjs

```javascript
// getDefaultExport function for compatibility with non-harmony modules
__webpack_require__.n = function(module) {
 		var getter = module && module.__esModule ?
 			function getDefault() { return module['default']; } :
 			function getModuleExports() { return module; };
 		__webpack_require__.d(getter, 'a', getter);
 		return getter;
};
```

`__webpack_require__.e` 用来加载额外的代码文件，并且用一个promise数组保存当前脚本是否运行完成

`webpackJsonpCallback` 搭配一起使用，确保额外代码文件已经download成功，并且加入了`modules` 变量中

```javascript
// This file contains only the entry chunk.
 	// The chunk loading function for additional chunks
 	__webpack_require__.e = function requireEnsure(chunkId) {
 		var promises = [];
 		// JSONP chunk loading for javascript
 		var installedChunkData = installedChunks[chunkId];
 		if(installedChunkData !== 0) { // 0 means "already installed".
 			// a Promise means "currently loading".
 			if(installedChunkData) {
 				promises.push(installedChunkData[2]);
 			} else {
 				// setup Promise in chunk cache
 				var promise = new Promise(function(resolve, reject) {
 					installedChunkData = installedChunks[chunkId] = [resolve, reject];
 				});
 				promises.push(installedChunkData[2] = promise);
 				// start chunk loading
 				var script = document.createElement('script');
 				var onScriptComplete;
 				script.charset = 'utf-8';
 				script.timeout = 120;
 				if (__webpack_require__.nc) {
 					script.setAttribute("nonce", __webpack_require__.nc);
 				}
 				script.src = jsonpScriptSrc(chunkId);
 				document.head.appendChild(script);
 			}
 		}
 		return Promise.all(promises);
 	};
```



`__webpack_require__.t` 运行代码块，并且根据一些判断条件返回不同的value

```javascript
// create a fake namespace object
// mode & 1: value is a module id, require it
// mode & 2: merge all properties of value into the ns
// mode & 4: return value when already ns object
// mode & 8|1: behave like require
__webpack_require__.t = function(value, mode) {
  // 直接加载value
  if(mode & 1) value = __webpack_require__(value);
  // 已经加载了value，不需要包装成es模块，直接返回
  if(mode & 8) return value;
  // 如果已经包装成为es模块，或者本身是个es模块，直接返回
  if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
  // 把value包装成一个es模块并且返回
  var ns = Object.create(null);
  __webpack_require__.r(ns);
  Object.defineProperty(ns, 'default', { enumerable: true, value: value });
  if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
  return ns;
};
```



## 异步加载(懒加载)

```javascript
import(/* webpackChunkName: "title" */ './titile').then()
```

如果代码中遇到了import，会自动进行代码分割，打包后会把title文件自动变成单独的文件。

```
__webpack_require__("./index.js") -> __webpack_require__.e("title") 定义promise并且从服务器下载title文件 -> webpackJsonpCallback 把title文件内容加载到modules里面，并且promise状态变成resolve -> 运行then之后的函数 __webpack_require__ 运行title文件
```



## Webpack编译流程

- 从webpack.config.js中获取options
- 通过options初始化Compiler
- 加载所有配置文件
- 执行Compiler对象的run方法开始执行编译
- 确定入口文件：根据配置中的entry找到所有入口文件
- 从入口文件出发，通过options里面的loader对模块进行编译(fs.readFileSync)，通过递归编译所有模块，得到modules
- 输入资源：根据入口和模块之间关系，组成一个个包含多个模块的chunk，得到chunks
- 再把每个chunk转换成一个单独的文件加入到输出列表中，得到asserts
- 确定好输入内容之后，根据配置options中的路径和文件名，把asserts中的内容写入到系统中(fs.writeFileSync)



## Loader

- WebPack会根据配置床技爱你NormalModuleFactory，他可以用来创建NormalModule
- 在工厂创建NormalModule实例之前还要通过loader的resolve来解析loader路径
- 在NormalModule实例创建之后，则会通过build方法来进行模块的构建，构建模块第一步就是用loader来加载并且处理模块内容。而loader-runner这个库就是webpack中loader的运行器
- 最后loader将处理完的模块内容输出，进入后续的编译流程

### babel-loader简单实现

```javascript
const babel = require("@babel/core");

// loader在执行的时候this会指向loaderContext对象，而callback就是在这个对象里的
function loader(source) {
  const options = {
    presets: ["@babel/preset-env"], // 配置预设，这是一个插件包
    sourceMap: true
  }
  // babel提供了ast那么webpack就不会自己再去生成ast了
  const { code, ast, map } = babel.transform(source,options)

  //如果只返回单个source，直接返回即可
  //如果需要返回多个内容，需要用到callback
  return this.callback(null, code, map, ast);
}

module.exports = loader
```

- 多个loader可以组成一个loader chain
- compiler需要得到最后一个loader产生的处理结果，这个处理结果应该是string或者buffer(被转换成一个string)

### Loader-runner实现

`loader runner` 是一个执行loader链条的模块 ，loader的类型：pre normal inline post

- 不管rules中的顺序如何排列，永远是都是post+inline+normal+pre
- 如果类型一样，先下后上，先右后左 

内联方式

- 使用 `!` 前缀，将禁用所有已配置的 normal loader(普通 loader) 

```javascript
import Styles from '!style-loader!css-loader?modules!./styles.css';
```

- 使用 `!!` 前缀，将禁用所有已配置的 loader（preLoader, loader, postLoader）

```javascript
import Styles from '!!style-loader!css-loader?modules!./styles.css';
```

- 使用 `-!` 前缀，将禁用所有已配置的 preLoader 和 loader，但是不禁用 postLoaders

```javascript
import Styles from '-!style-loader!css-loader?modules!./styles.css';
```

```javascript
// 实现runner.js
/**
1. 如何加载loaders
2. loaders的顺序
3. 运行loader-runner
*/
const path = require("path");
const fs = require("fs");
const {runLoaders} = require("loader-runner")
const rules = [
  {
    test: /\.js$/,
    use: ["normal-loader1", "normal-loader2"]
  },
  {
    test: /\.js$/,
    use: ["pre-loader1", "pre-loader2"],
    enforce: "pre"
  },
  {
    test: /\.js$/,
    use: ["post-loader1", "post-loader2"],
    enforce: "post"
  }
];
const requrest = "inline-loader1!inline-loader2!./index.js";
let inlineLoaders = requrest.split("!");
const resource = inlineLoaders.pop();
const loaderDir = path.resolve(__dirname, "loaders")
const resovleLoaders = loader => path.resolve(loaderDir, loader);
const resolveResource = path.resolve(__dirname, resource);
let prevLoaders = [];
let normalLoaders = [];
let postLoaders = []

for(let rule of rules) {
  if(rule.enforce === "pre") {
    prevLoaders.push(...rule.use);
  }else if(rule.enforce === "post") {
    postLoaders.push(...rule.use);
  }else {
    normalLoaders.push(...rule.use);
  }
}

prevLoaders = prevLoaders.map(resovleLoaders);
postLoaders = postLoaders.map(resovleLoaders);
normalLoaders = normalLoaders.map(resovleLoaders);
inlineLoaders = inlineLoaders.map(resovleLoaders);

const loaders = [...postLoaders, ...inlineLoaders, ...normalLoaders, ...prevLoaders];
console.log(loaders);

runLoaders({
  loaders,
  resource: resolveResource,
  readResource: fs.readFile.bind(fs)
}, (err, data) => {
  console.log(data);
})
```

```javascript
// 实现loader-runner，直接看源码容易理解
// https://github.com/webpack/loader-runner/blob/master/lib/LoaderRunner.js 
/*
主要搞清楚两个点：
1. pitch和loader的运行顺序是什么样的
2. pitch和loader同步和异步运行的区别
**/
```

### 自定义Loader

#### file-loader

加载图片的作用 `/\.(jpg|png|gif|bmp)$/` 

#### url-loader

对`file-loader` 的上层封装，如果limit=8*1024表示小于8kb的图片以base64内联到文件中，如果大于8kb使用file-loader引入图片。如果不设置limit那么都以base64内联到文件内。

好处是可以减少一个图片的HTTP请求。

#### style-loader

把样式文件变成一个style标签插入到html文件中

#### css-loader

在js中遇到了require/import这种标志符，就需要css-loader来识别，将内容导出来

#### less-loader

把less编译成css样式

### loader连用问题

- 最后一个loader(最左侧)才能用module.exports或者export default导出，否则会和其他最后一个loader冲突。

- 两个最左侧loader如何连用？

最左侧的loader放弃normal，在pitch中写相关代码并返回，当需要用到倒数第二个loader的地方使用require内联loader。这样到回到webpack中解析的时候发现需要require那么会根据require中的代码重新进入loader解析。

- remainingRquest data这些参数有什么用

获取还没使用过的loader

- pitch有什么用？什么时候用？

比如最左侧loader连用，可以通过return直接调转loader顺序。

## SourceMap

`source-map` 产生map文件

`eval` 使用eval包裹模块代码

`cheap` debugger或者报错不包含列信息、也不包含loader的source map（只有从loader解析出来的代码和webpack解析出来的代码映射）

`module` 包含loader的sourcemap(jsx -> js    bable sourcemap)

`inline` 将.map作为DataURL嵌入打包文件，不单独生成.map文件

### 最佳实践

#### 开发环境

我们在开发环境对SourceMap的要求是：快(eval)、信息全(module)，由于代码为压缩，不用在意代码列信息(cheap)，所以推荐开发环境配置`cheap-module-eval-source-map`

#### 生产环境

一般情况下，并不希望线上环境的代码被看到，所以不应该提供sourceMap给浏览器，但是我们有时候需要在线上环境做定位错误信息，这时候可以用`hidden-source-map` , 这样可以打包sourceMap文件，同时又可以在打包文件里 隐藏sourcemap注释



### 通过sourceMap调试代码

#### 生产环境调试

webpack打包仍然生成sourceMap，但是map文件挑出放到本地服务器，而不含map文件(无sourceMapURL)部署到服务器，借助第三方软件(fiddler)，将浏览器对map文件的请求拦截到本地服务器，就可以实现本地sourceMap调试

#### 测试环境调试

原理和生产环境调试类似，测试环境可以在打包文件后面加上sourceMapURL(例如说本地某个服务器)，这样debugger的时候map文件会映射到到所设置的URL上。然后将map文件放到本地启动一个服务器，这样测试环境就会映射到该服务器上了。

# Webpack的使用

## loader

- loader的特点：希望单一，每个loader有自己单一的特点，多个loader连续使用。比如css-loader是解析require/import引入的css，而style-loader是插入css到html中
- loader的用法：

```javascript
{
  test: /\.css$/,
  use: ["style-loader", "css-loader"]
}

{
  test: /\.js$/,
  use: [
    {
      loader: "babel-loader",
      options: {}
    }
  ]
}
```

- loader顺序默认从右向左执行，从下往上执行