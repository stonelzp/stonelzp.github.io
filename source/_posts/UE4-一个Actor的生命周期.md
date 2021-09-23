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

# 记录一些使用

## Actor的生成

### SpawnActor
在C++中如何进行一个Actor的生成
- [UE4: C++ コード から C++ クラスまたは Blueprint のアクターを FindObject して SpawnActor する方法](https://usagi.hatenablog.jp/entry/2017/10/17/123000)

#### SpawnActor生成对象是c++的情况
`FActorSpawnParameters`这个必要参数的各个属性含义还不明白
```
FActorSpawnParameters params;

// 必要に応じてパラメーターを設定する。
params.bAllowDuringConstructionScript = true;
params.bDeferConstruction = false;
params.bNoFail = true;
params.Instigator = this;
params.Name = { };
params.ObjectFlags = EObjectFlags::RF_NoFlags;
params.OverrideLevel = nullptr;
params.Owner = this;
params.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;
params.Template = nullptr;
```
生成：
```
// 事前に用意した params と必要な SpawnActor のオーバーロードを使う。
// ASomethingActor は必要に応じて #include "SomethingActor.h" など参照可能にしておく。
auto something =
  GetWorld()->SpawnActor< ASomethingActor >
  ( GetActorLocation()
  , GetActorRotation()
  , params
  );
```

#### SpawnActor生成对象是Blueprint的情况
跟上面的文章有些不一样的地方是构造函数里并不会把蓝图类的路径写死到程序里，这并不利于代码的维护

首先是指定我们需要生成的蓝图类
```
protected:
  // 例えば目的の Blurprint クラスが AStaticMeshActor ならそのオブジェクトの SpawnActor 用に参照させるオブジェクトの保持はこんな感じ。
  // （よりゆるくは AActor* でもよい。）
  UPROPERTY(EditDefaultOnly)
  TSubclassOf< class AStaticMeshActor > something_object;
```

对指定的蓝图类进行生成
```
// 仕込みで用意しておいた something_object を使う SpawnActor のオーバーロードを使う。
auto something =
  GetWorld()->SpawnActor< AStaticMeshActor>
  ( something_object // <-- ここで UClass* なオブジェクトを渡すために事前の仕込みが必要
  , GetActorLocation()
  , GetActorRotation()
  , params
  );
```

也不是说上面的文章就是不对的，只是个人的使用习惯而已。

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

感觉就是因为这个属性才会用到**DeferredSpawn**，经常在Blueprint中会有`SpawnActorByClass`的时候，当我们在C++使用了上面的修饰符和写法之后，在这个函数的里面就会出现"Exposed“属性。

所以就像上面说的那样，允许在BP的Construction之前做一些设置。

嘛先记住吧总归是有好处的。

- [UE4 C++でのExposeOnSpawnについて](https://papersloth.hatenablog.com/entry/2018/04/13/232533)

按照上面的文章内容所说，在蓝图中设置的时候除了**Expose on Spawn**需要check之外，**instance editable**一项也需要被check，理由暂时不太清楚。
