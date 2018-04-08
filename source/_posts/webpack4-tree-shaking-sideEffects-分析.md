---
layout: webpack4
title: webpack4 tree shaking && sideEffects 分析
date: 2018-04-03 17:38:38
tags:
---
### 1. 前言
早在webpack 2.0的时候， webpack就支持了tree shaking的优化，在webpack 4.0 发布的post中，作者提到新的webpack对编译后的bundle.js的体积增加了新的优化策略：sideEffects: false，这部分内容在webpack 4.0的文档中的tree shaking一节出现。
在这之前，对于项目中引用的第三方依赖库，tree shaking就是一个鸡肋，因为有"side effects"， 特别是经过babel转译之后的代码；并且webpack并没有对代码进行逻辑分析的能力或者说UglifyJS没有流程分析的能力，其实现在的webpack4也没有做到(n a 100% ESM module world, identifying side effects is straightforward. However, we aren't there just yet),
目前sideEffects: false 更像是一种中间状态， 这个选项告诉webpack，这个库是没有"副作用"的，可以大胆的进行代码移除（这里应该是webpack大胆的进行标记，由UglifyJS进行移除）。
关于webpack tree shaking的原理以及副作用的分析，有一篇[文章](https://segmentfault.com/a/1190000012794598)写得很好，我就不重复造轮子了，本篇文章带来两个例子，来看下效果。

用到的编译工具(对于本地做这些测试验证，很好用): [webpack-taste](https://github.com/qiaoqiaowang/webpack-taste)

### 2. Webpack tree Shakigng
 webpack 4增加了 mode参数，在mode 为production的时候，会自动引入UglifyJS，tree shaking才会生效。可以将上述编译工具clone下来，修改webpack.config.js文件中的mode参数为production.
 
 
##### src/math.js

	export function square(x) {
		return x * x;
	}

	export function cube(x) {
		return x * x * x;
	}

	export function list(x) {
		return x;
	}
	
	
###### src/index.js

	import {square} from "./math";

	var a = 3;
	console.log(square(a));
	
##### dist/bundle.js

	!function(e){var r={};function t(n){if(r[n])return r[n].exports;var o=r[n]={i:n,l:!1,exports:{}};return e[n].call(o.exports,o,o.exports,t),o.l=!0,o.exports}t.m=e,t.c=r,t.d=function(e,r,n){t.o(e,r)||Object.defineProperty(e,r,{configurable:!1,enumerable:!0,get:n})},t.r=function(e){Object.defineProperty(e,"__esModule",{value:!0})},t.n=function(e){var r=e&&e.__esModule?function(){return e.default}:function(){return e};return t.d(r,"a",r),r},t.o=function(e,r){return Object.prototype.hasOwnProperty.call(e,r)},t.p="",t(t.s=0)}([function(e,r,t){"use strict";t.r(r);var n;console.log((n=3)*n)}]);
	
	
### 3. sideEffects: false

在webpack-taste目录安装lodash依赖。

##### src/index.js
	
	import {xor} from "lodash-es";
	console.log(xor([2,1], [4,2]));
	
##### have sideEffects (将lodash-es的package.json文件中的sideEffects: false 配置项去掉)

	build complete!
	Hash: 005759b9aa686de0b61f
	Version: webpack 4.4.1
	Time: 5483ms
	Built at: 2018-4-8 14:39:08
		Asset      Size  Chunks             Chunk Names
	bundle.js  86.1 KiB       0  [emitted]  main
	Entrypoint main = bundle.js
	[0] F:/webpack/node_modules/lodash-es/_root.js 298 bytes {0} [built]
	[1] F:/webpack/node_modules/lodash-es/isBuffer.js 1.09 KiB {0} [built]
	[2] F:/webpack/node_modules/lodash-es/_nodeUtil.js 763 bytes {0} [built]
	[3] F:/webpack/node_modules/lodash-es/stubFalse.js 278 bytes {0} [built]
	[4] F:/webpack/node_modules/lodash-es/_cloneBuffer.js 1.03 KiB {0} [built]
	[5] F:/webpack/node_modules/lodash-es/_freeGlobal.js 171 bytes {0} [built]
	[6] (webpack)/buildin/harmony-module.js 597 bytes {0} [built]
	[7] ./src/index.js + 632 modules 605 KiB {0} [built]
       | F:/webpack/node_modules/lodash-es/zipObjectDeep.js 641 bytes [built]
       | ./src/index.js 92 bytes [built]
       | F:/webpack/node_modules/lodash-es/filter.js 1.47 KiB [built]
       | F:/webpack/node_modules/lodash-es/lodash.default.js 20.1 KiB [built]
       | F:/webpack/node_modules/lodash-es/isObjectLike.js 612 bytes [built]
       | F:/webpack/node_modules/lodash-es/isSymbol.js 680 bytes [built]
       | F:/webpack/node_modules/lodash-es/zipWith.js 958 bytes [built]
       | F:/webpack/node_modules/lodash-es/lodash.js 16.8 KiB [built]
       | F:/webpack/node_modules/lodash-es/isArray.js 486 bytes [built]
       | F:/webpack/node_modules/lodash-es/zipObject.js 662 bytes [built]
       | F:/webpack/node_modules/lodash-es/zip.js 607 bytes [built]
       | F:/webpack/node_modules/lodash-es/add.js 467 bytes [built]
       | F:/webpack/node_modules/lodash-es/isObject.js 731 bytes [built]
       | F:/webpack/node_modules/lodash-es/toNumber.js 1.53 KiB [built]
       | F:/webpack/node_modules/lodash-es/toFinite.js 866 bytes [built]
       |     + 618 hidden modules
    [8] (webpack)/buildin/global.js 509 bytes {0} [built]
	
##### sideEffects false

	build complete!
	Hash: ae732b3b3d0539147658
	Version: webpack 4.4.1
	Time: 1942ms
	Built at: 2018-4-8 14:41:27
		Asset      Size  Chunks             Chunk Names
	bundle.js  8.37 KiB       0  [emitted]  main
	Entrypoint main = bundle.js
	[0] ./src/index.js + 77 modules 45.5 KiB {0} [built]
       | F:/webpack/node_modules/lodash-es/_overRest.js 1.07 KiB [built]
       | ./src/index.js 92 bytes [built]
       | F:/webpack/node_modules/lodash-es/_arrayFilter.js 630 bytes [built]
       | F:/webpack/node_modules/lodash-es/isArrayLikeObject.js 740 bytes [built
	]
       | F:/webpack/node_modules/lodash-es/_baseXor.js 1.07 KiB [built]
       | F:/webpack/node_modules/lodash-es/_baseRest.js 557 bytes [built]
       | F:/webpack/node_modules/lodash-es/_baseFlatten.js 1.17 KiB [built]
       | F:/webpack/node_modules/lodash-es/xor.js 809 bytes [built]
       | F:/webpack/node_modules/lodash-es/isArrayLike.js 828 bytes [built]
       | F:/webpack/node_modules/lodash-es/_setToString.js 390 bytes [built]
       | F:/webpack/node_modules/lodash-es/_baseUniq.js 1.86 KiB [built]
       | F:/webpack/node_modules/lodash-es/isObjectLike.js 612 bytes [built]
       | F:/webpack/node_modules/lodash-es/_baseDifference.js 1.87 KiB [built]
       | F:/webpack/node_modules/lodash-es/identity.js 368 bytes [built]
       | F:/webpack/node_modules/lodash-es/_SetCache.js 630 bytes [built]
       |     + 63 hidden modules
	[1] (webpack)/buildin/global.js 509 bytes {0} [built]
    [2] F:/webpack/node_modules/lodash-es/_freeGlobal.js 171 bytes {0} [built]
    [3] (webpack)/buildin/harmony-module.js 597 bytes [built]
    [4] F:/webpack/node_modules/lodash-es/zipWith.js 958 bytes [built]
    [5] F:/webpack/node_modules/lodash-es/zipObjectDeep.js 641 bytes [built]
    [6] F:/webpack/node_modules/lodash-es/zipObject.js 662 bytes [built]
    [7] F:/webpack/node_modules/lodash-es/zip.js 607 bytes [built]
    [8] F:/webpack/node_modules/lodash-es/xorWith.js 1.19 KiB [built]
    [9] F:/webpack/node_modules/lodash-es/xorBy.js 1.27 KiB [built]
	[10] F:/webpack/node_modules/lodash-es/wrapperValue.js 453 bytes [built]
    [11] F:/webpack/node_modules/lodash-es/wrapperReverse.js 1020 bytes [built]
    [12] F:/webpack/node_modules/lodash-es/wrapperLodash.js 6.78 KiB [built]
    [13] F:/webpack/node_modules/lodash-es/wrapperChain.js 704 bytes [built]
    [14] F:/webpack/node_modules/lodash-es/wrapperAt.js 1.31 KiB [built]
    + 549 hidden modules

最后生成的bundle.js 相差甚远(86.1 KiB vs 8.37 KiB)。最新版的lodash-es， RxJS 都已支持sideEffects: false。希望越来越多的库文件都能进行此项优化！
