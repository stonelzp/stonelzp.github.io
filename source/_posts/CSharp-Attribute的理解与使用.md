---
title: CSharp-Attribute的理解与使用
permalink: csharp-attribute
date: 2020-05-04 06:54:22
categories:
- C#
tags:
- C#
- Unity
---
这次要学习的是CSharp中的属性(Attribute)，经常听见，也在Unity中使用过，却不知其为何物。**属性**是能够为类或者成员添加额外的信息的东西。

<!--more-->

首先是入门文章的链接：
- [属性](https://ufcpp.net/study/csharp/sp_attribute.html)

总结加上自己的实现，也可以说是汉化？这篇文章的最后更新是2011年，比较老了，还是要自己检验一下，而且还加深记忆。

# 属性(attribute)是什么
**属性**是用来为类或者成员添加额外信息的一种东西。像是类中的修饰符`public`,`private`都可以视为一种属性。

对于C++来说，像这种为类或成员添加额外信息的情况，需要我们对C++这门语言进行扩展，重新定义编译器。但是CSharp就不需要这么麻烦。也就是说，通过使用库中提供的属性，又或者是自己添加的属性，对编译器发出指令，可以留下使用者想要留下的信息。
> C++ などの既存の言語では、このような追加情報を定義する場合、 言語仕様自体を拡張し、新たにコンパイラを作り直す必要がありました。 それに対し、C# では自分で属性を定義し、クラスやメンバーに付加することが出来ます。 すなわち、ライブラリで提供されている属性や自作した属性を用いることで、 コンパイラに対する指示を行ったり、クラスの利用者に対する情報を残すことが出来ます。

使用属性的场合：
- 条件分支编译，向编译器发出指令的时候使用（Conditional和Obsolete等）
- 作者的信息以*meta data*的形式插入到程序当中（AssemblyTitle等）
- 利用**反射**，程序运行时取出想要的属性情报

Point：
- CSharp的话，可以自由的为类或者成员添加自定义的属性。
- 一部分的属性，可以作为对编译器或者VisualStudio发出指令的方式使用。
    - [Obsolete] class OldClass {} … 古いバージョンとの互換性のためだけに残してるけど、このクラスはもう使わないで。
    - [EditorBrowsable] T Property; … Visual Studio の IntelliSense（などの、開発ツールの補完機能）で表示するかどうかを設定します。

上面的两个例子我不太理解，遇见再说吧。

## 属性的使用
使用`[]`来为类或成员添加属性
```csharp
[属性名(属性パラメータ)]
メンバーの定義
```
属性名的最后要以`Attribute`结尾，就像既存的`ObsoleteAttribute`,`ConditionalAttribute`那样。但是在CSharp中使用的情况，省略掉也没有关系，`Obsolete`,`Conditional`这样也是可以在CSharp中使用的。


<span style:color="red">暂时先整理到这里，其它的地方积压的有些多了。之后再回来整理。</span>
