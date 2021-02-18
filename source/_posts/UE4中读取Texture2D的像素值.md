---
title: UE4中读取Texture2D的像素值
date: 2018-08-03 14:52:11
cateories:
- UnrealEngine4
tags:
- UE4
---
目的是在UE中获取到一张Texture2D图片的所有像素值，并将这些像素值进行某些处理并保存到另外一个Texture中。

<!--more-->

# 第一步：获取到Texture2D资源

Unreal Engine 4 Documentation
- [Referencing Assets](http://api.unrealengine.com/INT/Programming/Assets/ReferencingAssets/index.html)

对于高速化来说异步加载资源也是好的解决方案

Ureal Engine 4 Documentation
- [Asynchronous Asset Loading](https://docs.unrealengine.com/en-us/Programming/Assets/AsyncLoading)


# 参考的文章
- [Accessing pixel values of Texture2D](https://answers.unrealengine.com/questions/25594/accessing-pixel-values-of-texture2d.html)
    - >First you need to understand that a texture is normally, a sum of multiple images called MipMaps. MipMaps are down-scaled versions of your images, always in steps of power of 2, so the original image, is, say, 512x512 - this would be the MipMap "0", then the engine generates the MipMap "1" which is 256x256 and then MipMap "2" which is 128x128, this continues on down to 16x16 I think. The farther away the texture is rendered, the smaller version of the image is used. This means that you need to access the mipmap 0 of your texture.

# 对上面的答案进行了非常好的总结的文章
- [UE4 – Reading the pixels from a UTexture2D](http://isaratech.com/ue4-reading-the-pixels-from-a-utexture2d/)

- [Reading data from UTexture2D](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/84701-reading-data-from-utexture2d)

# 之后附上完整的代码


# UE4中的资源管理
- [Asset Management](https://api.unrealengine.com/INT/Engine/Basics/AssetsAndPackages/AssetManagement/index.html)

# 拓展知识
在代码中看到了`static_cast`这个语句，竟然不知道是做什么的。查了一下
- [C++中static_cast, dynamic_cast, const_cast用法/使用情况及区别解析](https://blog.csdn.net/bzhxuexi/article/details/17021559)

之后要好好整理一下。
