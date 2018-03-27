---
title: CommonJs的require和es6 import的区别
date: 2018-03-23 16:55:07
tags:
---

### 1. ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量
  这点是阮一峰老师讲解Module的语法时提到的，我的理解是，因为ES6 module要尽量静态化，所以import文件时，文件路径不支持变量形式，但是CommonJs 和 AMD模块，都是支持变量形式的，
  类似于: require(__dirname + "/a/b"); 所以需要在运行时才能知道其依赖关系。而正因为ES6 是静态化的，所以在编译时就可以确定依赖关系，这也是webpack tree shaking可以针对ES6 module 做代码优化的原因。
  
<!-- more -->
### 2. ES6 Module 中导入模块的属性或者方法是强绑定的，包括基础类型；而 CommonJS 则是普通的值传递或者引用传递

  这里拿一个例子来解释：
	
	//obj.js
	var obj = {};
	exports.a = 0;
	setTimeout(function(){
		obj.a = 1;
		exports.a++;
		console.log("after reassignment");
		console.log(obj);
		console.log(a);
	}, 500)
	
	
	//index.js
	var {obj, a} = require("./obj.js");
	console.log(obj.a);
	console.log(a);
	setTimeout(function(){
		console.log(obj.a);
		console.log(a);
	}, 1000);
	
	
上面这段代码会依次输出什么？

	undefined                                       index.js
	0                                               index.js
	after reassignment:                             obj.js
	{a: 1}                                          obj.js
	1                                               obj.js
	1                                               index.js
	0                                               index.js
	
如果将require改成import呢？

	//index.js
	import {obj, a} from "./obj.js";
	console.log(obj.a);
	console.log(a);
	setTimeout(function(){
		console.log(obj.a);
		console.log(a);
	}, 1000);
	
	undefined                                       index.js
	0                                               index.js
	after reassignment:                             obj.js
	{a: 1}                                          obj.js
	1                                               obj.js
	1                                               index.js
	1                                               index.js
	

进一步探究一下，webpack是如何针对require, import做的区分呢？查看下编译后的文件就见分晓。(只将核心代码放在这里，去掉了一些注释等)

	//require编译后的index.js模块
	var _require = __webpack_require__(/*! ./obj.js */ "./src/obj.js"),
    obj = _require.obj,
    a = _require.a;//a获取了_require.a，是一个局部变量，即使_require.a有变化，这里的a的值也不会有变化
	console.log(obj.a);
	console.log(a);
	setTimeout(function () {
		console.log(obj.a);
		console.log(a);
    }, 1000);//# sourceURL=webpack:///./src/index.js?");
	
	
	//import编译后的index.js模块
	var _obj = __webpack_require__(/*! ./obj.js */ "./src/obj.js");
	console.log(_obj.obj.a);
	console.log(_obj.a);
	setTimeout(function () {
		console.log(_obj.obj.a);
		console.log(_obj.a);//此处引用的是对象_obj.a字段的值，_obj.a变化，这里自然是可以拿到最新值的
    }, 1000);//# sourceURL=webpack:///./src/index.js?");

	