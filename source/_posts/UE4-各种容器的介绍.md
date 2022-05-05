---
title: UE4-各种容器的介绍
date: 2020-12-01 21:23:02
permalink: ue4-container
categories:
- UnrealEngine4
tags:
- UE4
---

UE4中提供了许多自定义的容器，比如说`TArray`，`TMap`等等，这篇文章主要想记录这些容器的使用。


<!--more-->

## TMap的使用
关于使用其实搜一下就好了，使用方法也很简单。

只是我遇到了一个问题就是TMap需要Key和Value，这个时候Key的类型需要是常用的类型，如果是自定义的类型则会出现编译错误，而我们需要把自定义的类型的Hash类型定义好。具体我也不是特别清楚

下面是从ActionRPG项目里复制过来的代码，如何使用参照代码就OK了。
**
```c++
/** Struct representing a slot for an item, shown in the UI */
USTRUCT(BlueprintType)
struct ACTIONRPG_API FRPGItemSlot
{
    GENERATED_BODY()

    /** Constructor, -1 means an invalid slot */
    FRPGItemSlot()
        : SlotNumber(-1)
    {}

    FRPGItemSlot(const FPrimaryAssetType& InItemType, int32 InSlotNumber)
        : ItemType(InItemType)
        , SlotNumber(InSlotNumber)
    {}

    /** The type of items that can go in this slot */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Item)
    FPrimaryAssetType ItemType;

    /** The number of this slot, 0 indexed */
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = Item)
    int32 SlotNumber;

    /** Equality operators */
    bool operator==(const FRPGItemSlot& Other) const
    {
        return ItemType == Other.ItemType && SlotNumber == Other.SlotNumber;
    }
    bool operator!=(const FRPGItemSlot& Other) const
    {
        return !(*this == Other);
    }

    /** Implemented so it can be used in Maps/Sets */
    friend inline uint32 GetTypeHash(const FRPGItemSlot& Key)
    {
        uint32 Hash = 0;

        Hash = HashCombine(Hash, GetTypeHash(Key.ItemType));
        Hash = HashCombine(Hash, (uint32)Key.SlotNumber);
        return Hash;
    }

    /** Returns true if slot is valid */
    bool IsValid() const
    {
        return ItemType.IsValid() && SlotNumber >= 0;
    }
};
```

**FGameplayAbilitySpecHandle：**
```c++
/** Handle that points to a specific granted ability. These are globally unique */
USTRUCT(BlueprintType)
struct FGameplayAbilitySpecHandle
{
    GENERATED_USTRUCT_BODY()

    FGameplayAbilitySpecHandle()
        : Handle(INDEX_NONE)
    {
    }

    /** True if GenerateNewHandle was called on this handle */
    bool IsValid() const
    {
        return Handle != INDEX_NONE;
    }

    /** Sets this to a valid handle */
    void GenerateNewHandle();

    bool operator==(const FGameplayAbilitySpecHandle& Other) const
    {
        return Handle == Other.Handle;
    }

    bool operator!=(const FGameplayAbilitySpecHandle& Other) const
    {
        return Handle != Other.Handle;
    }

    friend uint32 GetTypeHash(const FGameplayAbilitySpecHandle& SpecHandle)
    {
        return ::GetTypeHash(SpecHandle.Handle);
    }

    FString ToString() const
    {
        return IsValid() ? FString::FromInt(Handle) : TEXT("Invalid");
    }

private:

    UPROPERTY()
    int32 Handle;
};
```

### 关于TMap中的值传问题
经过验证，至少在Struct的使用上，TMap的使用是值传递，而不是引用传递。比方说：
```c++
USTRUCT()
struct FTest{
    int a = 0;
}

FTest test1 = FTest();

TMap<int32, FTest> TestMap;
TestMap.Add(1, test1);

test1.a = 100;

print(test1.a);
print(TestMap.Find(1));
```
上面的代码是我随意瞎写的，就是体现个意思，通过输出结果可以发现，`Add`函数被调用之后使用了值传递，而不是引用传递。至少结构体是这样的。

## TArray的使用
UE4中数组，记录一些简单的使用方法
```
TArray<int32> IntArray;
InArray.Init(10, 5);
// InArray == [10, 10, 10, 10, 10]  意外的有些时候会派上用场，需要记住

TArray<FString> StrArr;
StrArr.Add    (TEXT("Hello"));
StrArr.Emplace(TEXT("World"));
// StrArr == ["Hello","World"]  Emplace也是经常会使用到的，需要记住
```

这一部分意外的有点多，等到遇到Array操作有些困扰的时候需要查阅：
- [TArray: Arrays in Unreal Engine](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/TArrays/)


### Add_GetRef
要知道有这么一个函数
- [TArray::Add_GetRef - Adds a new item to the end of the array, possibly reallocating the whole array to fit.](https://docs.unrealengine.com/4.26/en-US/API/Runtime/Core/Containers/TArray/Add_GetRef/1/)
