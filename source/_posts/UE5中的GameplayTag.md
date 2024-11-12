---
title: UE5中的GameplayTag
date: 2024-01-03 20:49:43
permalink: UE5-gameplayTag
categories:
- UE5
tags:
- UnrealEngine4
- UnrealEngine5
- GameplayTag
---

简单记录一下UE中GameplayTag的各项用法。
<!--more-->

# Overview

# GameplayTag

## GameplayTag使用上的小技巧

### GameplayTag数组
记得有GameplayTagContainer的存在。

### Tag过滤器
当GameplayTag的数量个分类很多的时候，过滤器就重要了
- UPORPERTY的情况-> meta=(Categories="TagRooter")
  - FGameplayTag有效
  - FGameplayTagContainer有效
- UFUNCTION的情况-> meta=(GameplayTagFilter="TagRooter")

- [[UE4]GameplayTagをフィルタリングして検索効率アップ](https://historia.co.jp/archives/17266/)
