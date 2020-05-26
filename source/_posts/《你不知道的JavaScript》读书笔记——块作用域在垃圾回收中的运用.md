---
title: 《你不知道的JavaScript》读书笔记——块作用域在垃圾回收中的运用
date: 2020-05-26 18:48:55
tags: [JavaScript, 你不知道的JavaScript, 作用域]
categories: 读书笔记
---

ES6中引入了`let`和`const`关键字，可以实现块作用域了。对于块作用域的优势，一直只知道能够不污染作用域，阅读《你不知道的JavaScript》时才发现在垃圾回收中也能有很大优势。

考虑以下代码：

```javascript
function process(data) {
// 在这里做点有趣的事情
}
var someReallyBigData = { .. };
process( someReallyBigData );
var btn = document.getElementById( "my_button" );
      btn.addEventListener( "click", function click(evt) {
          console.log("button clicked");
}, /*capturingPhase=*/false );
```

click函数的点击回调并不需要`someReallyBigData`变量。理论上这意味着当`process(..)`执行后，在内存中占用大量空间的数据结构就可以被垃圾回收了。但是，由于`click`函数形成了一个覆盖整个作用域的闭包，JavaScript引擎极有可能依然保存着这个结构(取决于具体 实现)。

块作用域可以打消这种顾虑，可以让引擎清楚地知道没有必要继续保存`someReallyBigData`了:
```javascript
function process(data) {
// 在这里做点有趣的事情
}
// 在这个块中定义的内容可以销毁了! {
let someReallyBigData = { .. }; process( someReallyBigData );
}
var btn = document.getElementById( "my_button" );
     btn.addEventListener( "click", function click(evt){
         console.log("button clicked");
}, /*capturingPhase=*/false );
```