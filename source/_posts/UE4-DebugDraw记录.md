---
title: UE4-DebugDraw记录
date: 2022-06-27 23:14:50
permalink: how-to-debug-draw
categories:
- UnrealEngine4
tags:
- UE4
---

UE4本身提供了很多方便Debug的显示功能，在这里做一些使用记录。

<!--more-->

# Overview


# UKismetSystemLibrary::DrawDebug系列
`UKismetSystemLibrary::DrawDebug`这个函数库里面提供了很多开箱即用的函数，很方便也是使用频度很高的功能。

## DrawDebugSphere
使用方法（复制粘贴过来的）：
```
#include "Runtime/Engine/Classes/Kismet/GameplayStatics.h"
#include "Runtime/Engine/Classes/Kismet/KismetSystemLibrary.h"

// 適当なアクターに実装
void AMyPawn::Tick( float DeltaTime )
{
  Super::Tick( DeltaTime );

  // 2.5 秒ごとに表示を on/off する細工
  static float tt = 0;
  tt = FMath::Fmod( tt + DeltaTime, 5.0f );
  if ( tt < 2.5f )
  {
    // アクターの位置に赤い球を描画
    UKismetSystemLibrary::DrawDebugSphere( GetWorld(), GetActorLocation(), 100.0f, 12, FLinearColor::Red, 0.0f, 3.0f );
    // プレイヤーの位置を取得（本題と直接は関係しない）
    auto l0 = UGameplayStatics::GetPlayerPawn( GetWorld(), 0 )->GetActorLocation();
    // プレイヤーの位置からこのアクターの位置へ黄色の矢印を描画
    UKismetSystemLibrary::DrawDebugArrow( GetWorld(), l0, GetActorLocation(), 100.0f, FLinearColor::Yellow, 0, 3.0f );
  }
}

// プレイヤーのアクターに実装
void TestCharacter::Tick( float DeltaTime )
{
  Super::Tick( DeltaTime );

  // プレイヤーの位置に "PLAYER[0]" を緑色で描画
  UKismetSystemLibrary::DrawDebugString( GetWorld(), GetActorLocation(), TEXT( "PLAYER[0]" ), nullptr, FLinearColor::Green, 0 );
}
```

- [UE4: 点や線を簡易的に3D空間内へ描画する方法、あるいは UKismetSystemLibrary::DrawDebug 系の紹介](https://usagi.hatenablog.jp/entry/2017/12/21/083000)
- [Draw 3D Debug Points, Lines, and Spheres: Visualize Your Algorithm in Action](https://michaeljcole.github.io/wiki.unrealengine.com/Draw_3D_Debug_Points,_Lines,_and_Spheres:_Visualize_Your_Algorithm_in_Action/#Draw_Point)

# Debug HUD系列
之前有碰见过很有趣的HUD，就是DebugCamera用的HUD表示，感觉很适合常驻表示一些信息，有时间可以总结一下。
