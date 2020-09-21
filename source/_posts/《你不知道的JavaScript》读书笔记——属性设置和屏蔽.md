---
title: 《你不知道的JavaScript》读书笔记——属性设置和屏蔽
date: 2020-09-21 10:45:17
tags: [JavaScript, 你不知道的JavaScript, 原型链]
categories: 读书笔记
---

在JavaScrit中，给一个对象设置属性并不仅仅是添加一个新属性或者修改已有的属性值。

```javascript
myObject.foo = "bar";
```

如果`myObject`对象中包含名为`foo`的普通数据访问属性，这条赋值语句只会修改已有的属性值。

如果`foo`不是直接存在于`myObject`中，`[[Prototype]]`链就会被遍历，类似`[[Get]]`操作。
如果原型链上找不到`foo`，`foo`就会被直接添加到`myObject`上。

然而，如果`foo`存在于原型链上层，赋值语句`myObject.foo = "bar"`的行为就会有些不同。

如果属性名`foo`既出现在`myObject`中也出现在`myObject`的`[[Prototype]]`链上层，那 么就会发生屏蔽。`myObject`中包含的`foo`属性会屏蔽原型链上层的所有`foo`属性，因为`myObject.foo`总是会选择原型链中最底层的`foo`属性。

屏蔽比我们想象中更加复杂。下面我们分析一下如果`foo`不直接存在于`myObject`中而是存在于原型链上层时`myObject.foo = "bar"`会出现的三种情况。

1. 如果在`[[Prototype]]`链上层存在名为`foo`的普通数据访问属性并且没有被标记为只读(writable:false)，那就会直接在`myObject`中添加一个名为`foo`的新属性，它是屏蔽属性。
2. 如果在`[[Prototype]]`链上层存在`foo`，但是它被标记为只读(writable:false)，那么 无法修改已有属性或者在`myObject`上创建屏蔽属性。如果运行在严格模式下，代码会 抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在`[[Prototype]]`链上层存在`foo`并且它是一个setter，那就一定会 调用这个setter。`foo`不会被添加到(或者说屏蔽于)`myObject`，也不会重新定义`foo`这个 setter。

大多数开发者都认为如果向`[[Prototype]]`链上层已经存在的属性(`[[Put]]`)赋值，就一 定会触发屏蔽，但是如你所见，三种情况中只有一种(第一种)是这样的。

如果你希望在第二种和第三种情况下也屏蔽 foo，那就不能使用`=`操作符来赋值，而是使用`Object.defineProperty(..)`来向`myObject`添加 foo。

第二种情况可能是最令人意外的，只读属性会阻止`[[Prototype]]`链下层 隐式创建(屏蔽)同名属性。这样做主要是为了模拟类属性的继承。你可以把原型链上层的`foo`看作是父类中的属性，它会被`myObject`继承(复制)，这样一来`myObject`中的`foo`属性也是只读，所以无法创建。但是一定 要注意，实际上并不会发生类似的继承复制。这看 起来有点奇怪，`myObject`对象竟然会因为其他对象中有一个只读`foo`就不能包含`foo`属性。更奇怪的是，这个限制只存在于`=`赋值中，使用`Object. defineProperty(..)`并不会受到影响。