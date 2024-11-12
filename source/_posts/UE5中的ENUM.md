---
title: UE5中的ENUM
date: 2024-01-03 21:09:26
permalink: ue5-how-to-use-enum
categories:
- UE5
tags:
- UnrealEngine4
- UnrealEngint5
---

UE中的枚举使用方法整理
<!--more-->

# Overview

# UENUM
- [UE4 C++とUnreal C++の列挙型の扱い](http://papersloth.hatenablog.com/entry/2017/07/07/005225)
**CppExampleEnum.h**
```C++
#pragma once

UENUM(BlueprintType)
enum class ECppExampleEnum : uint8 {
    None = 0,
    Foo,
    Bar
};
```
对于`enum class ECppExampleEnum : uint8`这种写法有些迷惑。
- `class`是为了使枚举类型更安全。为什么安全，参考下面的链接。之后整理。
- `uint8`是为了指定枚举器的基础类型。

参考链接:
- [C++11的enum class & enum struct和enum](https://blog.csdn.net/sanoseiichirou/article/details/50180533#2-enum-class-%E5%92%8C-enum-struct)
- [C\C++中的整形提升](https://blog.csdn.net/sanoseiichirou/article/details/50171727)
- [C++标准文档-n2347](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2347.pdf)


# UENUM的一些基础的使用
## Convert to Stirng
直接取自引擎代码
```
	if (const UEnum* EnumPtr = FindObject<UEnum>(nullptr, TEXT("/Script/PCG.EPCGMedadataCompareOperation"), true))
	{
		return FName(FString("Compare: ") + EnumPtr->GetNameStringByValue(static_cast<int>(Operation)));
	}
	else
	{
		return NAME_None;
	}
```

## For-Loop ENUM_RANGE_BY_COUNT()
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


