---
title: 《你不知道的JavaScript》读书笔记——对象那些事
date: 2020-06-10 15:55:44
tags: [JavaScript, 你不知道的JavaScript, 对象]
categories: 读书笔记
---

## null是object吗

众所周知，JavaScript中有6种基础类型：

* string
* number
* boolean
* null
* undefined
* object

这6种基本类型本身并不是对象，但在执行`typeof null`的时候，会返回`object`，以至于初学者时常将null当作了一种对象类型。实际上，null本身是基本类型，`typeof null === 'object'`这是语言本身的bug。

在JavaScript中，二进制前三位都为0的时候会判断为object类型，然而null的二进制形式全是0，自然前三位也是0，所以执行typeof的时候返回了object.

## 字面量和构造函数生成的基本类型有区别吗

考虑如下代码：

```javascript
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

var strObject = new String( "I am a string" ); typeof strObject; // "object"
strObject instanceof String; // true

// 检查 sub-type 对象
Object.prototype.toString.call( strObject ); // [object String]
```

对于上面代码，我们有两个字符串`strPrimitive`和`strObject`，分别通过字面量和构造函数的方式生成。

从上面的结果可以看出，`strPrimitive`是一个基本类型，并不是一个对象，和`strObject`执行`typeof`的结果并不相同。

在必要的时候，语言会自动把字符串字面量转换成一个String对象，因此我们可以直接在字符串字面量上访问属性和方法。

## 对象的属性名永远是字符串

在对象中，属性名永远都是字符串。如果你使用string(字面量)以外的其他值作为属性名，那它首先会被转换为一个字符串。即使是数字也不例外，虽然在数组下标中使用的的确是数字，但是在对象属性名中数字会被转换成字符串，所以当心不要搞混对象和数组中数字的用法：

```javascript
var myObject = { };
myObject[true] = "foo";
myObject[3] = "bar";
myObject[myObject] = "baz";

myObject["true"]; // "foo"
myObject["3"]; // "bar"
myObject["[object Object]"]; // "baz"
```