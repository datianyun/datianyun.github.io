---
layout: blog
title: this
summary: javascript原理系列
---

# {{ page.title }}

2014-04-22 北京 

## 说明

刚在微博上看了一个帖子，javascript里this的面试题，发现很久不看又给忘了，找了篇别人写的文章确实挺好的，转帖过来边写边学习下。 

#定义

This是执行上下文的一个属性：

`````
activeExecutionContext = {
  VO: {...},
  this: thisValue
};

`````
This与上下文的可执行代码类型有关，其值在进入上下文阶段就确定了，并且在执行代码阶段是不能改变的。

下面就来详细对其作个介绍。


#全局代码中This的值

这种情况下，一切都变得非常简单，this的值总是全局对象本身;因此，可以间接地获取引用：

`````javascript
// 显式定义全局对象的属性
this.a = 10; // global.a = 10
alert(a); // 10
 
// 通过赋值给不受限的标识符来进行隐式定义
b = 20;
alert(this.b); // 20
 
// 通过变量声明来进行隐式定义
// 因为全局上下文中的变量对象就是全局对象本身
var c = 30;
alert(this.c); // 30
`````

#函数代码中This的值

当this在函数代码中的时候，事情就变得有趣多了。这种情况下是最复杂的，并且会引发很多的问题。

函数代码中this值的第一个特性（同时也是最主要的特性）就是：它并非静态的绑定在函数上。

正如此前提到的，this的值是在进入上下文的阶段确定的，并且在函数代码中的话，其值每次都会大不相同。

然而，一旦进入执行代码阶段，其值就不能改变了。比方说，要想给this赋一个新的值是不可能的，因为this根本就不是变量（相反的，在Python语言中，它显示定义的self对象是可以在运行时随意更改的）：

`````javascript
var foo = {x: 10};
 
var bar = {
  x: 20,
  test: function () {
 
    alert(this === bar); // true
    alert(this.x); // 20
 
    this = foo; // error, 不能更改this的值
 
    alert(this.x); // 如果没有错误，则其值为10而不是20
 
  }
 
};
 
// 在进入上下文的时候，this的值就确定了是“bar”对象
// 至于为什么，会在后面作详细介绍
 
bar.test(); // true, 20
 
foo.test = bar.test;
 
// 但是，这个时候，this的值又会变成“foo”
// 纵然我们调用的是同一个函数
 
foo.test(); // false, 10

`````

因此，在函数代码中影响this值的因素是有很多的。

首先，在一般的函数调用中，this的值是由激活上下文代码的调用者决定的，比如说，调用函数的外层上下文。this的值是由调用表达式的形式决定的。

理解并谨记这一点是非常必要的，有利于在任何上下文中都能准确的确定this的值。

影响调用上下文中的this的值的只有可能是调用表达式的形式，也就是调用函数的方式。 （一些关于JavaScript的文章和书籍中指出的“this的值取决于函数的定义方式，如果是全局函数，则this的值就会设置为全局对象，如果是某个对象的方法，则this的值就会设置为该对象”——这纯属扯淡，根本就是在误人子弟）。 正如此前大家看到的，纵然是全局函数，this的值也会随着函数调用方式的不同而不同：
`````javascript
function foo() {
  alert(this);
}
 
foo(); // global
 
alert(foo === foo.prototype.constructor); // true
 
// 然而，同样的函数，以另外一种调用方式的话，this的值就不同了
 
foo.prototype.constructor(); // foo.prototype

`````

调用一个对象的某个方法的时候，this的值也有可能不是该对象的：

`````javascript
var foo = {
  bar: function () {
    alert(this);
    alert(this === foo);
  }
};
 
foo.bar(); // foo, true
 
var exampleFunc = foo.bar;
 
alert(exampleFunc === foo.bar); // true
 
// 同样地，相同的函数以不同的调用方式，this的值也就不同了
 
exampleFunc(); // global, false

`````

那么，究竟调用表达式的方式是如何影响this的值的呢？为了完全搞明白这其中的奥妙，首先，这里有必要先介绍一种内部类型——引用类型（the Reference type）。

#引用类型

引用类型的值可以用伪代码表示为一个拥有两个属性的对象——base属性（属性所属的对象）以及该base对象中的propertyName属性：

`````javascript
var valueOfReferenceType = {
  base: ,
  propertyName: 
};
`````
引用类型的值只有可能是以下两种情况：

1.当处理一个标识符的时候
2.或者进行属性访问的时候

标识符其实就是变量名，函数名，函数参数名以及全局对象的未受限的属性。如下所示：

````javascript
var foo = 10;
function bar() {}

````

