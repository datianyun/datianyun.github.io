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

这类代码是在“程序”级别上被处理的：比如，加载一个外部的js文件或者内联的js代码（被包含在<script></script>标签内）。全局代码不包含任何函数体内的代码。

在初始化的时候（程序开始），ECStack如下所示：

`````
ECStack = [
    globalContext
];

`````

#函数代码

一旦控制器进入函数代码（各类函数），就会有新的元素会被压栈到ECStack。要注意的是：实体函数代码并不包括内部函数的代码。如下所示，我们调用一个函数，该函数递归调用自己一次：

`````
(function foo(bar){
    if (bar){

    return;

}

foo(true);
})();

`````

之后，ECStack就被修改成如下所示:


`````
//首先激活foo函数
ECStack = [
     functionContext
    globalContext
];
//递归激活foo函数
ECStack = [
     functionContext - recursively
     functionContext
    globalContext
];

`````

每次函数返回，退出当前活动的执行上下文时，ECStack就会被执行对应的退栈操作——先进后出——和传统的栈实现一致。同样的，当抛出未捕获的异常时，也会退出一个或者多个执行上下文，ECStack也会做相应的退栈操作。待这些代码完成之后，ECStack中就只剩下一个执行上下文（globalContext）——直到整个程序结束。

#Eval代码

说到eval代码就比较有意思了。这里要提到一个叫做调用上下文的概念，比如：调用eval函数时候的上下文，就是一个调用上下文，eval函数中执行的动作（例如：变量声明或者函数声明）会影响整个调用上下文：

`````
eval(‘var x = 10’);
(function foo(){
    eval(‘ var y = 20’);
})();
alert(x); // 10
alert(y); // ”y” is not defined

`````

ECStack会被修改为：

`````
ECStack = [
    globalContext
];
//eval(‘var x = 10’);
ECStack.push(
    evalContext,
    callingContext: globalContext
);

// eval exited context
ECStack.pop();

//foo function call
ECStack.push( functionContext);

//eval(‘ var y = 20’);
ECStack.push(
    evalContext,
    callingContext:  functionContext
);

//return from eval
ECStack.pop();

//return from foo
ECStack.pop();

`````

在1.7以上版本SpiderMonkey的实现中（Firefox，Thunderbird浏览器内置的JS引擎），允许在调用eval函数的时候，将调用上下文作为第二个参数传递给eval函数。因此，如果传入的调用上下文存在的话，就有可能会影响该上下文中原有的私有变量（在该上下文中声明的变量）：

````
function foo(){
    var x = 1;
    return function() { alert(x); }
};

var bar = foo();

bar(); // 1
eval(‘x = 2’, bar); //传递上下文，影响了内部变量“var x”
bar(); // 2

````