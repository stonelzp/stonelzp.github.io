---
title: UE4-Packaging设置的注意点
date: 2022-04-30 18:17:43
permalink: ue4-packaging-settings
categories:
- UnrealEngine4
tags:
- UE4
- Packaging
---

记录一下UnrealEngine在打包的时候一些设定的注意点。

<!--more-->
# Overview
目前对于UE的打包经历的不是很多，关于UE4的打包设定也只是使用了最简单的设置，或者默认设置。关于各自设置也应该深入了解一下了。

其实在项目开发的时候自己验证的东西放到哪里确实有些头疼。但是本质上还是我对UE4的Packaing不甚了解。哪一部分会被打包，哪一部分会被摘除，在项目的最后摘除所有的内容虽然可以，但是如果平常开发的时候工作的环境很不整洁的话，后面的工作可能会变成不可能完成的工作量了。

关于C++的代码部分，如果是在Editor部分的话放到项目的Editor的部分Module中就可以。可是uasset之类的二进制文件我就有些迷茫了。

# Developers Folder
- [Developers Folder](https://docs.unrealengine.com/4.27/en-US/Basics/ContentBrowser/UserGuide/DevelopersFolder/)

上面是官网上对DevelopersFolder的介绍。这个文件下的东西完全应该放置属于自己的东西，使用的原则有以下几个：
1. 在这个文件夹下只放置自己的东西
2. 外部不可以参照任何该文件夹下的任何资源
3. 该文件夹下的资源不可以参照除这个文件夹下之外的资源，如果需要的话把资源复制到该文件夹下

第三步执行起来说实话有些困难。

如果之后我对UE的打包方式有了更好的理解，明确了那个部分不会被打包到游戏中的话，或许会有更好的办法。

# Packaging
暂未整理。


参考资料：
- [UE4 リリース向けパッケージを作る際にやるべきこと](https://unrealengine.hatenablog.com/entry/2015/08/14/211733)
- [UE4 一番お手軽にパッケージング時のサイズを最適化する方法](https://unrealengine.hatenablog.com/entry/2015/10/13/220000)
- [Tinyなパッケージに挑戦してみよう](https://wakanya.hatenablog.com/entry/TinyPackage)
- [UE4.27.2 プラグイン、パッケージ化時容量測定結果](https://qiita.com/O_Y_G/items/b74f54205cfe4c6cf8f0)
- [Assets Cleaner - Project Cleaning Tool](https://www.unrealengine.com/marketplace/ja/product/assets-cleaner-project-cleaning-tool)

上面的那个付费的AssetsCleaning工具有可能会用到。
