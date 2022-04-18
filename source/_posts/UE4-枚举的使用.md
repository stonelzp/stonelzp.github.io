---
title: UE4-枚举的使用
date: 2019-09-10 14:58:41
permalink: ue4-enum
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
介绍关于枚举的使用。

<!--more-->

# Overview

- [UE4 C++とUnreal C++の列挙型の扱い](http://papersloth.hatenablog.com/entry/2017/07/07/005225)

# UE为UENUM提供的一些宏的记录

## ENUM_RANGE_BY_COUNT()
为定义的枚举类型添加一个For-Loop方便使用的宏。
声明：
```
#include "Misc/EnumRange.h"

UENUM()
enum class EElementType : uint8
{
	Earth,
	Fire,
	Wind,
	Water,
	Count	UMETA(Hidden)
};

ENUM_RANGE_BY_COUNT(EElementType, EElementType::Count);
```

使用：
```
for (EElementType ElementType : TEnumRange<EElementType>())
{
	SettingArray.Add(LoadInitialSettings(ElementType));
}
```

参考文章：
- [【UE4】UEnumで範囲ベースforを行うメモ](https://qiita.com/Rinderon/items/253e780fe4bd1297abb8)
