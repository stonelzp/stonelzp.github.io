---
title: UE5中的INTERFACE
date: 2024-01-03 20:58:37
permalink: UE5-how-to-use-Interface
categories:
- UE5
tags:
- UnrealEngine4
- UnrealEngine5
---

UE中的接口内容整理到这里。
<!--more-->

# Overview

# UInterface

### UE4中的Interface
关于UE4或者说C++的一些接口的用法进行一些记录。原本是ActionRPG中的Module开始调查，就先从Module的接口开始说起，不过除了调查这个是什么的接口，更重要的是了解接口的使用方式，不仅仅是UE4中的有重要作用的接口。

C++的接口的一些特性使用方法更为重要。

#### IModuleInterface

#### UInterface
关于UE4中的`UINTERFACE()`使用的频率十分之高，熟悉之后就会发现接口这个功能或者说书习惯显得十分重要。

这一部分是跟ActionRPG项目没有关系的整理：
- [UE4/C++ Interface の作り方](https://usagi.hatenablog.jp/entry/2017/07/04/042417)
- [【UE4】C++でInterfaceを使う方法](https://qiita.com/bigengelt/items/12dabebc1bf646f4be13)

上面的文章相当的有用。从零创建一个UE4可以使用的UInterface的方法。

最省力的方式是用UE4Editor里面新建C++的方式，下拉到最后一行找到**Unreal Interface**，直接创建一个接口就可以了。
如何添加接口的函数就是一个难点了。

创建接口的函数就有些看自己的习惯了

##### 后续希望蓝图实现的接口函数
也是最常见的一种方式
```
// === MyInterface.h ===
// 手間暇(1): UINTERFACE で UInterface を public 継承した U プリフィックスのクラスを定義する
UINTERFACE(BlueprintType)
class MyApp_API UMyInterface: public UInterface
{
  // 要注意: ここでもこちらは U プリフィックス版を使う
  GENERATED_UINTERFACE_BODY()
};

// 手間暇(2): UE4のクラス定義マクロを付けずに先に定義した U プリフィックスのクラスの I プリフィックス版のクラスを定義する
class MyApp_API IMyInterface
{
  // 要注意: ここでもこちらは I プリフィックス版を使う
  GENERATED_IINTERFACE_BODY()
public:
  // 手間暇(3) Blueprint へ関数を公開するめんどくさいマクロをだらだら書く（めんどくさいでござる＠ｗ＠）
  UFUNCTION( BlueprintNativeEvent, BlueprintCallable, Category = "MyInterface" )
  float GetMyValue();
};
```
上面的代码直接从上面的文章链接中复制过来的。
可以看到接口函数的声明添加了`BlueprintNativeEvent`这个Meta修饰符，当然上面的代码不是使用上面说的最省力的方式的代码，这里`UMyInterface`里的UClass的meta修饰符使用了`BlueprintType`，UE4默认创建的话是`MinimalAPI`。

至于使用哪种的话，`MinimalAPI`Meta修饰符好像是为了把UClass类**类型**的情报显示给其它module的，由于只是导出类型信息给其它module，可以减少编译时间，我也不知道该不该使用这个。不过想要在BP中使用的话`BlueprintType`肯定是需要的哇。

然后是实现
```
// === MyInterface.cpp ===
// 手間暇(4): GENERATED_UINTERFACE_BODY が宣言だけ自動的に生成する U プリフィックス版の方の ctor を定義する
UMyInterface::UMyInterface( const class FObjectInitializer& ObjectInitializer ): Super( ObjectInitializer ) { }

// 手間暇(5): ここでようやく C++ 的な virtual-override における virtual 宣言された基底クラスのメンバー関数を定義
// Note: どこで↓の宣言に相当する virtual retur_type FunctionName_Implementation(); が行われているのかは後述。
float GetMyValue_Implementation() { return 3.14159265358979f; }
```

这里需要注意的是，我们在定义UInterface的时候使用的是`GENERATED_IINTERFACE_BODY`这个宏，所以我们需要再添加一个`FObjectInitializer`参数的构造函数定义，貌似不加的话会报错（我记不太清了如果说错了对不起）。构造函数的声明不显式声明没问题，宏那里帮忙做了好像。

最后是接口的使用:
```
// === MyImplementation.h ===
// 手間暇(6): インターフェースを実装する UE4 オブジェクトクラスでの override 実装の宣言
float GetMyValue_Implementation() override;

// === MyImplementation.cpp
// 手間暇(7): インターフェースを実装する UE4 オブジェクトクラスでの override 実装の定義
float MyImplementation::GetMyValue_Implementation() { return 1.41421356f; }
```

需要注意的是声明的是`GetMyValue`而Override的则是`GetMyValue_Implementation`，这就是UE里的黑魔法了，记住就行。

相比上面的写法我更喜欢下面的方法
##### 只想C++实现的接口函数
```
#pragma once

#include "CoreMinimal.h"

#include "UObject/Interface.h"
#include "GRAIDebugMoveToInterface.generated.h"

UINTERFACE(MinimalAPI, meta = (CannotImplementInterfaceInBlueprint))
class UMyInterface : public UInterface
{
	GENERATED_BODY()
};

class GOLEMRAIDERS_API IMyInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
    virtual void DebugGetMyValue() {} // 纯虚函数应该不支持
};
```
像上面的虚函数写法就很简洁，而且cpp文件什么都不需要了。
`GENERATED_BODY`帮我们把构造函数的定义也一起做了。

接口的使用就是
```
// === MyImplementation.cpp
virtual void DebugGetMyValue() override;
```

我感觉这种写法就是披着UE4C++皮的很普通的C++的接口使用方式，泛用性不是很高，至少像再UE4中实现的话还是用第一种方式。

##### 蓝图UINTERFACE的使用
当使用BluepintNativeEvent创建好了Interface之后就是使用了，导入头文件什么的就不说了

假定我们现在有一个实现了这个接口的实例Actor:`SampleActor`,调用这个接口的操作
```
if(SampleActor->GetClass()->ImplementsInterface(UMyInterface::StaticClass()))
{
    IMyInterface::Execute_DebugMyValue(SampleActor);
}
```

调用函数的方法就很简单。


##### UINTERFACE修饰符


