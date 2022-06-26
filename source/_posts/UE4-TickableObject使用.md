---
title: UE4-TickableObject使用
date: 2022-06-26 23:13:19
permalink: how-to-make-a-tickable-object
categories:
- UnrealEngine4
tags:
- UE4
---

UE4提供了非常方便的Tick函数，但是在一些情况下一些Tick函数不会默认存在，则需要自己添加。

<!--more-->
# Overview
这篇文章学习理解如何在UObject中添加Tick函数。

- [How to make a Tickable Object](https://benui.ca/unreal/tickable-object/)

# FTickableGameObject

**MyTickableThing.h**
```
#pragma once

#include "CoreMinimal.h"
#include "Tickable.h"

class FMyTickableThing : public FTickableGameObject
{
public:
	// FTickableGameObject Begin
	virtual void Tick( float DeltaTime ) override;
	virtual ETickableTickType GetTickableTickType() const override
	{
		return ETickableTickType::Always;
	}
	virtual TStatId GetStatId() const override
	{
		RETURN_QUICK_DECLARE_CYCLE_STAT( FMyTickableThing, STATGROUP_Tickables );
	}
	virtual bool IsTickableWhenPaused() const
	{
		return true;
	}
	virtual bool IsTickableInEditor() const
	{
		return false;
	}
	// FTickableGameObject End


private:
	// The last frame number we were ticked.
	// We don't want to tick multiple times per frame
	uint32 LastFrameNumberWeTicked = INDEX_NONE;
};
```

**MyTickableThing.cpp**
```
#include "MyTickableThing.h"

void FMyTickableThing::Tick( float DeltaTime )
{
	if ( LastFrameNumberWeTicked == GFrameCounter )
		return;

	// Do our tick
	// ...

	LastFrameNumberWeTicked = GFrameCounter;
}
```

当然这种写法并不是只局限于UObject，自己定义的结构体或者类都可以使用这个方式。
