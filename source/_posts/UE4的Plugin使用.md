---
title: UE4的Plugin使用
date: 2018-08-02 10:39:13
categories:
- UnrealEngine4
tags:
- UE4
- Plugins
---
制作UE4的插件相当于给UE4引擎添加新的功能模块。UE4的功能模块组成是由Module组成的，关于Module具体是什么，中文翻译就是模块，自己的理解中可以说是文件的单位了。

新建的插件代表着跟UE4本来的功能模块不是从属于一个Module，所以需要为自己制作的插件制作一个属性为Public的公开接口以供UE4引擎调用。

<!--more-->

# 什么是Module？
看了一眼这篇文章的创建时间是2018年，而2022年我又来更新了。

在创建Plugin之前，应该了解一下Module这个概念，当时的我也是赶鸭子上架，Module是什么都不太清楚。
在自己的项目中可以创建很多个Module，Plugin里面也可以创建很多个Module。至于什么时候应该用Plugin什么时候该直接在自己的项目里创建Module，这区别我在UE4的Editor扩展里面有提到过一些。

关于Module的基础概念：
- [UE4 Modulles](https://docs.google.com/presentation/d/1rSFFQk7RxNAHevROfVvUNviUfIntLkO_HpdvzHLkNEs/view#slide=id.g6e0e4b3bcf_5_213)

偶然发现的这个文档，总结了一些，算是很珍贵的资源了。


# Plugin的Public公开权限
一般一个Module中不想公开的源文件都会设置为Private权限，不允许外界的Module访问。要把权限公开，使得其他的模块能够访问的话需要以下两步。

## 头文件的位置
- 头文件(.h)放到[<ModuleDirectory>\Source\Public\]文件夹中去
- cpp实现文件放到[<ModuleDirectory>\Source\Private\]文件夹中去

## 添加Export用的宏
在类的声明中添加一个宏：`<大写字母的Module名字>_API`

例如：
```C++
UCLASS()
class SAYHELLO_API USayHelloFunction : public UBlueprintFunctionLibrary{
    GENERATED_BODY()
}
```
这样一个名为`SayHello`的Module的class的权限就变成公开的了。

Tips：
- 上述的`SAYHELLO_API`的定义文件位置在`Intermediate/Build/Win64/UE4Editor/Development/SayHello/Definitions.SayHello.h`。里面定义了`DLLEXPORT`,`DLLIMPORT`。
- UE会按照Module的单位生成DLL，`<ModuleName>_API`在自身的Module中会指定DLLEXPORT，在其他的Module中会指定DLLIMPORT。（啥意思？书上就写了这么多。。。）


参考资料：
- [Unreal Engine 4 Documentation - Plugins](http://api.unrealengine.com/INT/Programming/Plugins/index.html)
- [Unreal Engine 4 C++ 插件介绍](https://blog.csdn.net/qq_20309931/article/details/53584075)
- [ue4插件开发](https://blog.csdn.net/yangxuan0261/article/details/52098104)

# Plugins
顺着文章名字找过来，结果发现内容跟我想象的不太一样。那么就分为两部分，上面是介绍UE4插件的使用方式。这一部分是介绍UE4中用到的插件。


## Impostor Bake
- [Impostor Baker for UE4](https://80.lv/articles/impostor-baker-for-ue4/)

- [Octahedral Impostors](https://shaderbits.com/blog/octahedral-impostors/)
- [Github](https://github.com/ictusbrucks/ImpostorBaker)

这个插件在工程中使用过不过不太清楚是做什么的，不是我使用的部分，只是看到了。（需要整理）
