---
title: UE4-一个Actor的生命周期
permalink: ue4-actor-lifecycle
date: 2020-09-28 18:41:50
categories:
- UnrealEngine4
tags:
- UE4
---

关于Actor的生命周期，与很多初始化的操作相关联，准备整理到这篇文章中。

<!--more-->

# Actor Lifecycle
- [Actor Lifecycle - What actually happens when an Actor is loaded or spawned, and eventually dies.](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Actors/ActorLifecycle/index.html)

首先是官方文档。

# 遇到的一些问题

### SpawnActorDeferred
遇到了一个Actor的生成问题，不过关于物体的动态生成一直都是一个非常重要的问题。

首先看一下GASDocumentation中的一段代码：
```c++
// .h
    UPROPERTY(BlueprintReadWrite, EditAnywhere, Meta = (ExposeOnSpawn = true))
    float Range;

    UPROPERTY(BlueprintReadWrite, Meta = (ExposeOnSpawn = true))
    FGameplayEffectSpecHandle DamageEffectSpecHandle;

// .cpp
    AGDProjectile* Projectile = GetWorld()->SpawnActorDeferred<AGDProjectile>(ProjectileClass, MuzzleTransform, GetOwningActorFromActorInfo(), Hero, ESpawnActorCollisionHandlingMethod::AlwaysSpawn);
    Projectile->DamageEffectSpecHandle = DamageEffectSpecHandle;
    Projectile->Range = Range;
    Projectile->FinishSpawning(MuzzleTransform);
```

#### Deferred Spawn
这种生成Actor的方法被这样称为

> An Actor can be Deferred Spawned by having any properties set to "Expose on Spawn."
> 
> 1. **SpawnActorDeferred** - meant to spawn procedural Actors, allows additional setup before Blueprint construction script.
> 
> 2. Everything in SpawnActor occurs, but after PostActorCreated the following occurs:
> 
>       1. Do setup / call various "initialization functions" with a valid but incomplete Actor instance.
>       2. **FinishSpawningActor** - called to Finalize the Actor, picks up at ExecuteConstruction in the Spawn Actor line.


就像上面描述的和那样的使用方法。

#### Expose on Spawn
上面的例子提到了了这个属性修饰符，在Blueprint中也能找到这个条目。

感觉就是因为这个属性才会用到**DeferredSpaen**，经常在Blueprint中会有`SpawnActorByClass`的时候，当我们在C++使用了上面的修饰符和写法之后，在这个函数的里面就会出现"Exposed“属性。

所以就像上面说的那样，允许在BP的Construction之前做一些设置。

嘛先记住吧总归是有好处的。

- [UE4 C++でのExposeOnSpawnについて](https://papersloth.hatenablog.com/entry/2018/04/13/232533)

按照上面的文章内容所说，在蓝图中设置的时候除了**Expose on Spawn**需要check之外，**instance editable**一项也需要被check，理由暂时不太清楚。




