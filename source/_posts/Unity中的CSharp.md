---
title: Unity中的CSharp
permalink: csharp-in-unity
date: 2020-04-16 17:10:21
categories:
- Unity
- CSharp
tags:
- csharp
---
跟CSharp打交道也很久了，但是有很多的东西我都从来未接触过，甚至是一些比较基础的用法我都不知道。这篇文章虽然带有Unity，但主要还是为了记录CSharp的使用，嘛，毕竟我主要的CSharp的使用场景就是Unity。

<!--more-->

在另外的一篇文章，**Unity知识点记录**中也有一些零星的CSharp使用的记载，但是Unity相关的内容比较多。
### C#的一些基础用法
#### 函数传参时new一个参数
函数传参的时候，这个参数未必是上面已经预备好的，而是想要一个新的参数或者类，

```csharp
// sample code
public class Data()
{
    int a;
    flaot b;
}

public void main()
{
    Data data = new Data();
    SampleFunction(data);
}
```

上面这种最常见的用法。但是当我们`Data`的类并未提前预备好，我们就必须new一个。
```csharp
public void main()
{
    SampleFunction(new Data());
}
```

那么这里问题又来了，我们想new一个Data类之后顺便还想把变量值加入，这个时候想到的就是为Data类添加一个构造函数。

但是实际上我读大佬的代码的时候，就不需要：
```csharp
public void main()
{
    SampleFunction(new Data(){ a = sample_a, b = sample_b});
}
```

这样就不需要额外创建一个构造函数了，说实话，这应该是非常基础的用法，但是我完全没有印象，就连写的途中，变量赋值之间用的是逗号而不是分号而纠结了一会儿，还以为是VS中的自动补齐工具出问题了。

记住，应该会经常使用的，很方便的写法。感觉有时间需要了解一下这个写法的机制。

<span style="color:red">重温了这篇文章发现自己最近刚好用到这个地方，但是很巧我就没有这么用，还是傻傻的创建了构造函数，我真是最近散漫惯了。</span>

#### 获取一个枚举类型里的所有类型
需求就是像标题所说，至于为什么要这么做，就应该会有很多理由，比如用枚举设定事件类型的时候想要知道这些类型有那些或者有多少。

就用`Enum.GetNames(Type)`这个函数来获取。

- [Enum.GetNames(Type) 方法](https://docs.microsoft.com/zh-cn/dotnet/api/system.enum.getnames?view=netframework-4.8)

但其实这里还有更重要的地方，那就是CSharp中的**属性**，在别的文章中更详细的展开了，这里想只贴上实现的代码，以供日后参考。



