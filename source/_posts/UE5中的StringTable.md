---
title: UE5中的StringTable
date: 2024-01-03 20:29:24
permalink: ue5-stringtable
categories:
- UE5
tags:
- UnrealEngine5
- UnrealEngine4
- StringTable
- Localization
---

今后使用UE5的版本进行验证，所以UE4的内容验证或许会有些怠慢。这次主要是介绍StringTable的相关用法。

<!--more-->

# Overview
针对本地化功能提供的文本数据表。

# StringTable
- [StringTables-Centralize text into a table that can be referenced throughout your application.](https://docs.unrealengine.com/5.3/en-US/using-string-tables-for-text-in-unreal-engine/)


# 实用Code

## StringTable资源引用
UE中提供了`UStringTable`这样的资源允许我们在Editor中持有该资源的引用。
```
UPROPERTY(EditDefaultsOnly)
UStringTable* SampleStringTable;
```

## StringTable中是否存在键值
方便使用的函数：
```
bool UStringTableHelperTool::HasKey(UStringTable* InStringTable, const FString& InKey)
{
    if(IsValid(InStringTable))
    {
        FString OutKey;
        if(InStringTable->GetMutableStringTable()->GetSourceString(InKey, OutKey))
        {
            return true;
        }
    }

    return false;
}
```

## StringTable中的读写
向StringTable中写入数据
```
void UStringTableHelperTool::UpdateStringTable(UStringTable* InStringTable, const FSting& InKey, const FString& InValue)
{
    if(!HasKey(InStringTable, InKey))
    {
        InStringTable->Modify();
        InStringTable->GetMutableStringTable()->SetSourceString(InKey, InValue);
    }
    else
    {
        // TODO: 如果存在相同的键值，则覆盖值
    }
}
```