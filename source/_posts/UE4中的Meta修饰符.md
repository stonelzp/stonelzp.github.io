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

### meta=(WorldContext="WorldContextObject", CallableWithoutWorldContext)
这个我用来定义BlueprintCallable函数时调用省略`WorldContextObject`。

想要记录下这个meta的原因是，我想得到当前调用函数所在的Blueprint的名字。在C++中直接使用`__Function__`写个宏就可以直接得到调用函数的名字，但是在Blueprint中，没有宏当参数这么便利的方法，得顺便把self当参数传进去。

这就导致我想输出蓝图名字的时候，无论如何都得额外做一个把self传进去的操作。

但是有一天我发现，为什么UE4自带的`Print`函数就没有传这样的参，而且还把调用蓝图的名字输出来了。看代码就知道了。

在下面的源代码中可以找到UE4自带的Print函数声明定义：
- `Engine\Source\Runtime\Engine\Classes\Kismet\KismetSystemLibrary.h`
- `Engine\Source\Runtime\Engine\Private\KismetSystemLibrary.cpp`

会找到下面的声明：
```C++
UFUNCTION(BlueprintCallable, meta=(WorldContext="WorldContextObject", CallableWithoutWorldContext, Keywords = "log print", AdvancedDisplay = "2", DevelopmentOnly), Category="Utilities|String")
static void PrintString(UObject* WorldContextObject, const FString& InString = FString(TEXT("Hello")), bool bPrintToScreen = true, bool bPrintToLog = true, FLinearColor TextColor = FLinearColor(0.0, 0.66, 1.0), float Duration = 2.f);
```

重点是下面:
**WorldContext="WorldContextObject", CallableWithoutWorldContext** 这部分。

受这个启发，这个让我们可以不用传入额外的参数（self），并且使用`GetNameSafe`函数得到名字。

**Tips:**
1. Keywords关键字可以设置搜索的关键字。细节上面的官方文档中有。
2. `GEngine->AddOnScreenDebugMessage((uint64)-1, Duration, TextColor.ToFColor(true), FinalDisplayString);`向屏幕输出LOG，来自上面`PrintString()`函数的实现。


# Class Specifier
除了一些meta限定，其它的地方UE4也提供了许多修饰符。这个是**UCLASS**定义的时候使用的修饰符，可以定义UCLASS在Engine和Editor中的一些行为。
- [Class Specifiers](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/GameplayArchitecture/Classes/Specifiers/)

## 一些我用过的

### hidecategories=("category1|category2")
官网上也这样写
```
HideCategories=(Category1, Category2, ...)
```

作用就是可以隐藏定义的Category，由于这个修饰符是会影响到子类的，所以被我用来在子类的蓝图中隐藏父类中的Category。

# Property Specifiers
这一部分用来记录`UPROPERTY()`，修饰属性的各种修饰符。

## 一些我用过的

### Transient
根据官网的描述：
> Property is transient, meaing it will not be saved or loaded. Properties tagged this way will be zero-filled at loat time.

```c++
    UPROPERTY(Transient)
    int32 ValueX;
```

说实话，在没有对UE4的多人模式进行学习的时候真是一头雾水。被标记了这个属性修饰符的变量，意味着变量存储的是一个暂时的值，我们并不想永久的保存它，比如说这个变量保存的是其它类中的内容，就好像是弱指针一样，只是暂时的保留这个变量的值。

还有一个作用就是，标记了这个修饰符的变量，它不会被Serialize(见UE4的Net Serialize)。
