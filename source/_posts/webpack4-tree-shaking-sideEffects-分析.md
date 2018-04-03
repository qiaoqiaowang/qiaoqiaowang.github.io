---
layout: webpack4
title: webpack4 tree shaking && sideEffects 分析
date: 2018-04-03 17:38:38
tags:
---
### 1. 前言
早在webpack 2.0的时候， webpack就支持了tree shaking的优化，在webpack 4.0 发布的post中，作者提到新的webpack对编译后的bundle.js的体积增加了新的优化策略：sideEffects: false，那这个和webpack原有的tree shaking又有什么关系呢？
以下通过例子来解释下什么叫 side effect, 和 tree shaking的关系，进而解释webpack 4.0为什么支持了sideEffects:false options。 （或者直接跳到结论部分）