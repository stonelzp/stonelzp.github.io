---
title: 我所理解的Lambda表达式
date: 2019-05-27 11:31:14
permalink: the-lambda-what-i-understand

categories:
- C#

tags:
- C#
- lambda

---
纠结了好久，最终决定把lambda这个部分从Unity中分离出来单独总结。理解并能够使用Lambda表达式是这篇文章的目的。除此之外如果有关于lambda的拓展用法亦或者好的使用案例一并在这里记录。

<!--more-->

首先是Lambda的基础概念

# Lambda

## lambda expressions是什么

> A lambda expression is a block of code (an expression or a statement block) that is treated as an object. It can be passed as an argument to methods, and it can also be returned by method calls.

来自 [Lambda expressions (C# Programming Guide)](https://docs.microsoft.com/ja-jp/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)

匿名函数，也可以这样叫，将函数作为对象处理，用`{}`包含。因为最近一年之和C++打交道，C#的语言机制究竟是什么样子的完全不记得了，关于C#的编译执行过程应该与C++区别并整理到别的文章去。

但是就C++的感觉来说，lambda给我的印象就是函数指针。但是内部的处理我还是不清楚(无论是C#还是C++)，比如说函数指针(lambda)的调用代价是什么样子的，跟普通的函数调用有什么区别等等。

但是这些先放一边，将lambda视为一个对象，可以作为**参数①**，可以作为**返回函数调用②**。

## lambda怎么用
既然要使用lambda就要知道它的使用方式是什么样子的。
- [私はこうしてLINQ・ラムダ式を理解できた（入門）](https://qiita.com/Aki_mintproject/items/a70c33911ad96e5d8188)

这篇涉及了一些Linq的东西，讲的会比较多一些，但是很好的涵盖的lambda的继承用法。

- [C# 今更ですが、ラムダ式](https://qiita.com/rawr/items/11790e9ea08a29d028a4)

这篇讲的内容比较少但是对于基础讲的很具体。

### lambda文法



### 什么时候用lambda

