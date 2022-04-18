---
title: 'UE4:Error_Message整理'
date: 2018-08-16 17:28:14
categories:
- UnrealEngine4
tags:
- UE4
- UE4Error
---
整理在学习UE4的过程中遇到的一些BUG。

<!--more-->



# UE4中遇见的报错
旨在记录UE4学习过程中遇见的各种报错。
## class关键字的使用

`syntax error:missing ';' before '*'` 这样完全不知到为什么的报错。

参考链接：
- [syntax error: missing ';' before '*' (First Person Shooter C++ Tutorial)](https://answers.unrealengine.com/questions/701590/syntax-error-missing-before-first-person-shooter-c.html)

加了一个关键字`class`就解决了。

在UE4C++里面，ForwardDeclaration是非常常见的，要铭记于心才行。
关于ForwardDeclaration:
- [What are Forward declarations in C++](https://www.geeksforgeeks.org/what-are-forward-declarations-in-c/)

## bUsedWithInstancedStaticMeshes
时间太过于久远再加上自己的懒惰，这个错误到底是在说什么我也不清楚了。


`LogMaterial: Warning: Material /Game/MagicCircle/Materials/MT_Smoke.MT_Smoke missing bUsedWithInstancedStaticMeshes=True! Default Material will be used in game.`

## 包含头文件的问题
就像下面的信息描述的，我使用了一个`TWeakObjectPtr`来进行值的初始化，于是便报错了，各种调查结果显示，是赋值的值的类型的头文件没有好好的包含进去，以至于找不到该类型。

`error C2664: 'FWeakObjectPtr &FWeakObjectPtr::operator =(const FWeakObjectPtr &)': cannot convert argument 1 from 'T *' to 'const UObject *'`

![Error description](Annotation 2020-07-16 181212.jpg)

参考的链接：[TWeakObjectPtr with Custom Character Class](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/42246-tweakobjectptr-with-custom-character-class)



`error C2440: 'initializing' : cannot convert from 'initializer-list'`

关于这个错误类型，涉及到一些初始化操作，但我的情况下还是头文件没有包含进去的问题，真是头大。


## unresolved eternal symbol类型
有的时候会在错误信息中包含上述关键词的错误

`error Link2019: unresolved external symbol 函数名` 这样的错误是找到了函数的声明但是却找不到函数的定义。

一般需要检查Build.cs文件中的依赖项是否添加。

亦或者出现在利用另一个**Module**中的函数的时候，找不到函数的实现，出现上面的错误编译不通过。实际上调用的函数是虚函数的话就没有问题，关键是不是虚函数的情况。

所以问我怎么办我也不太清楚，这一部分好好参考如何添加Module或者ActionRPG的LoadScreen部分的源码好好实现应该就没有问题了。

## 蓝图的编译错误
`Can’t parse default value ‘none’ for ...`这样的错误，发生在变量的地方，一把都是Array的情况，就是明明没有问题，编译就是通不过。

搜了一下发现UE4.26版本会经常出现这个问题，我也的确用的就是UE4.26。解决方案就是:
- 向这个Array里添加一个元素
- 移除这个元素
- 再编译保存，编译错误就会消失

出现这个问题的原因：
> It has kept the original value of “none” instead of updating it to be an “Empty Array”
>
> To fix you need to modify the default value and save.

参考资料：
- [ Can’t parse default value ‘none’ for Cameras?](https://forums.unrealengine.com/t/cant-parse-default-value-none-for-cameras/478596)

# 别人的劳动成果
另外贴一个别人整理的一些报错的文章
- [UE4 C++ビルドエラー対応覚書](https://finap.hateblo.jp/entry/2019/12/06/000000)
