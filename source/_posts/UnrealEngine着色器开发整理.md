---
title: UnrealEngine着色器开发整理
date: 2018-07-25 13:49:32
categories:
- 工具
tags:
- UE4
---
整理自己在学习Unreal Engine 4着色器过程中遇到的问题和知识点。

<!--more-->

## Unreal Engine 4设定
### 1. UE4的内置材质Shader函数库位置
UE4Shader的编写入门反而相对比较容易，使用自带的各种函数库拉拉线竟然就可以完成。函数库的位置都在:

位置：
- `C:\Program Files\Epic Games\UE_4.19\Engine\Shaders\Private` UE4的安装文件夹中

### 2. Material.cpp文件位置
- `C:\Program Files\Epic Games\UE_4.19\Engine\Source\Runtime\Engine\Private\Materials`
