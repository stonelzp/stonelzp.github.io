---
title: UE4中的Meta修饰符
date: 2021-05-10 12:21:00
permalink: ue4-meta-data-specifiers
categories:
- UnrealEngine4
tags:
- UE4
---
UE4中的**MetadataSpecifiers**是个很邪门的东西，一开始的时候根本不知道是什么东西怎么用，等到对UE4C++有了较为深入的了解还有引擎的源码读多了之后，才知道这个东西虽然不用也行但是用了有的时候会更方便一些。与之同样重要或者说更为重要的是**PropertySpecifers**这个概念。

<!--more-->

# MetadataSpecifiers
> When declaring classes, interfaces, structs, enums, enum values, functions, or properties, you can add Metadata Specifiers to control how they interact with various aspects of the engine and editor. Each type of data structure or member has its own list of Metadata Specifiers.

上面是官方文档[Metadata Specifiers](https://docs.unrealengine.com/en-US/ProgrammingAndScripting/GameplayArchitecture/Metadata/index.html)的描述，当我声明UE4的数据结构的时候，基本上都可以添加Meta修饰符来实现一些动作，每种数据结构和成员都有它自己的一个Metadata修饰符的列表，而我的目的就是把自己遇到过的Meta修饰符整理，记录下使用方法。

需要注意的是，Metadata修饰符之存在于Editor部分，不要把Metadata修饰符的使用范围扩大到游戏中的逻辑，比如说游戏运行是变量数据的读取。仅仅是方便我们在UE4的Editor中开发的存在。

## 遇到的一些使用

### Meta = (EditCondition = "变量名")
这里的变量名大多是布尔类型。用途是控制Detail面板上的变量能否修改。

```
UPROPERTY(EditAnywhere, BlueprintReadWirte)
uint8 bCondition : 1;

UPROPERTY(EditAnywhere, BlueprintReadWirte, meta = (EditCondition = "bCondition"))
float SettingsValue;
```

这样Detail面板上的`SettingsValue`的值就会根据`bCondition`的值而允许修改或者不允许修改。

`meta = (EditCondition = "!bCondition")`这样的写法也是可以的。

- 参考文章：[【UE4】詳細パネルでの編集可・不可を制御する](https://qiita.com/Dv7Pavilion/items/6f86134587b3ad6ff396)
