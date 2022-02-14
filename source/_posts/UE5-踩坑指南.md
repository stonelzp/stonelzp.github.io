---
title: UE5-踩坑指南
date: 2022-01-28 13:07:32
permalink: ue5-the-fucking-moment
categories:
- UnrealEngine5
tags:
- UE5
---

记录一些在UE5中遇见的BUG，目前是UE5EA阶段。

<!--more-->

# 报错记录

## error CS0579
> DotNetCore\obj\Release\netcoreapp2.0\.NETCoreApp,Version=v2.0.AssemblyAttributes.cs(4,12): error CS0579: global::System.Runtime.Versioning.TargetFrameworkAttribute' 属性が重複しています。

在UE4版本升为UE5的时候，准备打开UE5的工程文件的时候就打不开了，VS的报错就像上面的错误内容。
按照上面报错的路径，去引擎源码的文件夹下找到这个文件，由于这个文件是自动生成的文件，删除再重新打开VS编译就解决了。

参考文章:
- [【C#】.NETFramework,Version=v4.0.AssemblyAttributes.csでコンパイルエラーが発生する](http://nanoappli.com/blog/archives/2040)
