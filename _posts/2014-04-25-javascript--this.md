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


中间过程中，对应的引用类型的值如下所示：


````javascript
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
var barReference = {
  base: global,
  propertyName: 'bar'
};

````

要从引用类型的值中获取一个对象实际的值需要GetValue方法，该方法用伪代码可以描述成如下形式：

````javascript

function GetValue(value) {
 
  if (Type(value) != Reference) {
    return value;
  }
 
  var base = GetBase(value);
 
  if (base === null) {
    throw new ReferenceError;
  }
 
  return base.[[Get]](GetPropertyName(value));
 
}

````

````javascript
GetValue(fooReference); // 10
GetValue(barReference); // function object "bar"

````

对于属性访问来说，有两种方式： 点符号（这时属性名是正确的标识符并且提前已经知道了）或者中括号符号：

````javascript
foo.bar();
foo['bar']();

````

中间过程中，得到如下的引用类型的值：



````javascript
var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
 
GetValue(fooBarReference); // function object "bar"

````

问题又来了，引用类型的值又是如何影响函数上下文中this的值的呢？——非常重要。这也是本文的重点。总的来说，决定函数上下文中this的值的规则如下所示：


````
函数上下文中this的值是函数调用者提供并且由当前调用表达式的形式而定的。 如果在调用括号()的左边，有引用类型的值，那么this的值就会设置为该引用类型值的base对象。 所有其他情况下（非引用类型），this的值总是null。然而，由于null对于this来说没有任何意义，因此会隐式转换为全局对象。

````

#引用类型以及null（this的值）

有这么一种情况下，当调用表达式左侧是引用类型的值，但是this的值却是null，最终变为全局对象。 发生这种情况的条件是当引用类型值的base对象恰好为活跃对象。

当内部子函数在父函数中被调用的时候就会发生这种情况。正如第二章介绍的， 局部变量，内部函数以及函数的形参都会存储在指定函数的活跃对象中：

````javascript
function foo() {
  function bar() {
    alert(this); // global
  }
  bar(); // 和AO.bar()是一样的
}

````

活跃对象总是会返回this值为——null（用伪代码来表示，AO.bar()就相当于null.bar()）。然后，如此前描述的，this的值最终会由null变为全局对象。

当函数调用包含在with语句的代码块中，并且with对象包含一个函数属性的时候，就会出现例外的情况。with语句会将该对象添加到作用域链的最前面，在活跃对象的之前。 相应地，在引用类型的值（标识符或者属性访问）的情况下，base对象就不再是活跃对象了，而是with语句的对象。另外，值得一提的是，它不仅仅只针对内部函数，全局函数也是如此， 原因就是with对象掩盖了作用域链中更高层的对象（全局对象或者活跃对象）：

````javascript
var x = 10;
 
with ({
 
  foo: function () {
    alert(this.x);
  },
  x: 20
 
}) {
 
  foo(); // 20
 
}
 
// because
 
var  fooReference = {
  base: __withObject,
  propertyName: 'foo'
};

````

当调用的函数恰好是catch从句的参数时，情况也是类似的：在这种情况下，catch对象也会添加到作用域链的最前面，在活跃对象和全局对象之前。 然而，这个行为在ECMA-262-3中被指出是个bug，并且已经在ECMA-262-5中修正了；因此，在这种情况下，this的值应该设置为全局对象，而不是catch对象。

````javascript
try {
  throw function () {
    alert(this);
  };
} catch (e) {
  e(); // __catchObject - in ES3, global - fixed in ES5
}
 
// on idea
 
var eReference = {
  base: __catchObject,
  propertyName: 'e'
};
 
// 然而，既然这是个bug
// 那就应该强制设置为全局对象
// null => global
 
var eReference = {
  base: global,
  propertyName: 'e'
};

````

同样的情况还会在递归调用一个非匿名函数的时候发生（函数相关的内容会在第五章作相应的介绍）。在第一次函数调用的时候，base对象是外层的活跃对象（或者全局对象）， 在接下来的递归调用的时候——base对象应当是一个存储了可选的函数表达式名字的特殊对象，然而，事实却是，在这种情况下，this的值永远都是全局对象：

````javascript

(function foo(bar) {
 
  alert(this);
 
  !bar && foo(1); // "should" be special object, but always (correct) global
 
})(); // global

````
#当函数作为构造器被调用时this的值

这里要介绍的是函数上下文中关于this值的另外一种情况——当函数作为构造器被调用的时候：

````javascript

function A() {
  alert(this); // newly created object, below - "a" object
  this.x = 10;
}
 
var a = new A();
alert(a.x); // 10

````

#手动设置函数调用时this的值

Function.prototype上定义了两个方法（因此，它们对所有函数而言都是可访问的），允许手动指定函数调用时this的值。这两个方法是：.apply和.call； 它们都接受第一个参数作为调用上下文中this的值。而它们的不同点其实无关紧要：对于.apply来说，第二个参数接受数组类型（或者是类数组的对象，比如arguments）, 而.call方法接受任意多的参数。这两个方法只有第一个参数是必要的——this的值。

````javascript
var b = 10;
 
function a(c) {
  alert(this.b);
  alert(c);
}
 
a(20); // this === global, this.b == 10, c == 20
 
a.call({b: 20}, 30); // this === {b: 20}, this.b == 20, c == 30
a.apply({b: 30}, [40]) // this === {b: 30}, this.b == 30, c == 40

````


