---
title: 'Assembly-CSharp中的程序集'
permalink: assembly-csharp
date: 2020-01-17 19:15:37
categories:
- C#
tags:
- C#
- 笔记
---
在接触Unity的过程中，一直不清楚`Assembly`这个单词到底是什么意思，中文翻译是程序集，但是具体是个什么东西一直没有一个准确的概念。这次准备搞懂它并且对**反射**这个概念进行一次透彻的了解。

<!--more-->

# Assembly-程序集
- [.NET 中的程序集](https://docs.microsoft.com/zh-cn/dotnet/standard/assembly/)官方文档说明
> 程序集构成了 .NET 应用程序的部署、版本控制、重用、激活范围和安全权限的基本单元。 程序集是为协同工作而生成的类型和资源的集合，这些类型和资源构成了一个逻辑功能单元。 程序集采用可执行文件 (.exe) 或动态链接库文件 (.dll) 的形式，是 .NET 应用程序的构建基块 。 它们向公共语言运行时提供了注意类型实现代码所需的信息。

就像一个应用程序一样，在Windows中这个应用程序会有`exe`文件和若干`dll`文件。还有一些其他的我不知道的文件，这些都是程序集的一部分。这就是一个程序集，准确与否我也不确定。反正上面的我理解就是这个意思。

关于Assembly的一些要点的话在上面的链接中可以确认，有时间再整理一下吧。

# Reflection-C#中的反射机制
- [反射 (C#)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/reflection)
官网的说明跟没有说明一样，还是得参考其他的文章的使用。因为具体要怎么使用，为什么要使用反射对我来说还是一个疑问，使用反射有什么好处呢？

- [C#反射机制](https://zhuanlan.zhihu.com/p/41282759)

> 反射是.NET中的重要机制,通过反射,可以在运行时获得程序或程序集中每一个类型(包括类、结构、委托、接口和枚举等)的成员和成员的信息。有了反射,即可对每一个类型了如指掌。另外我还可以直接创建对象,即使这个对象的类型在编译时还不知道。

运行时能够获取程序或程序集中的的成员情报。利用这些情报可以做很多的事情。

话是这么说我还不太清楚直接使用DLL中的情报有什么不同。就我的经验是不知道DLL中的开发的接口的时候也是无从下手，是不是使用反射就可以某种意义上了解DLL的实现和开放的接口或者函数，利于处理。当然EXE也是一样。

## 反射的使用

[System.Reflection 命名空间](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection?view=netframework-4.8)

> System.Reflection 命名空间包含通过检查托管代码中程序集、模块、成员、参数和其他实体的元数据来检索其相关信息的类型。 这些类型还可用于操作加载类型的实例，例如挂钩事件或调用方法。 若要动态创建类型，请使用 [System.Reflection.Emit](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit?view=netframework-4.8) 命名空间。




### 反射相关类

#### System.Type类
通过这个类可以访问任何给定数据类型的信息。

**获取给定类型的Type引用有3种常用方式**
- 使用C# `typeof`运算符
    ```csharp
    Type t = typeof(string);
    ```
- 使用对象`GetType()`方法
    ```csharp
    string s = "grayworm";
    Type t = s.GetType();
    ```
- 还可以调用Type类的静态方法`GetType()`
    ```csharp
    Type t = Type.GetType("System.String");
    ```
官方文档说明：[Type 类](https://docs.microsoft.com/zh-cn/dotnet/api/system.type?view=netframework-4.8)

#### System.Reflection.Assembly类
它可以用于访问给定程序集的信息，或者把这个程序集加载到程序中。

官方文档：[Assembly类](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.assembly?view=netframework-4.8)

在官方文档中似乎例举了一些使用方法，暂时我并没有什么需要使用这个反射机制。但是貌似Unity中也会使用这个特性。下次当我决定使用一些DLL等的外部链接库的时候再好好研究并实现这些个特性。

不是往后推，后面对于自己的项目肯定会加入一个Log收集的动态链接库，那个时候一定要把两种链接方式都总结一下。
