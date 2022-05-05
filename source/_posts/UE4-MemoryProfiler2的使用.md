---
title: UE4-MemoryProfiler2的使用
date: 2022-04-18 12:02:27
permalink:
categories:
- UnrealEngine4
tags:
- UE4
- MemoryProfiler2
---

是时候调查内存的使用情况了！这篇文章主要是介绍MemoryProfiler2的工具的使用。看看使用这个工具是如何查看UE4的内存使用情况。

<!--more-->

# Overview
主要是介绍Memory Profiler这个工具的使用。但是在介绍这个工具之前，如果只是想要简单快速的查看内存的使用，推荐使用`MemReport -Full`这个命令直接输出到文件。但是输出来的文件没有可视化的图表，看起来有些费眼睛。

还有一些其它的查看内存的工具放在最后说明。

# Memory Profiler2

## 找到二进制启动文件
工欲善其事，必先利其器。首先我们需要知道如何启动MemoryProfiler2这个工具。

首先去`\Engine\Programs\MemoryProfiler2\Binaries\MemoryProfiler2`这个路径下看有没有这个启动文件。这里我默认都是使用的自定义引擎，不是Epic的Launcher上的引擎。

如果在编译UE4引擎源码的时候直接选的`Build Solution`，基本上会编译所有的内容，Memory Profiler2也会跟着被编译，所以会直接发现它的二进制启动文件。如果只编译了UE4的Solution的话，在上面的路径中就找不到启动器了，需要我们自己编译。

去`Engine/Source/Programs/MemoryProfiler2/MemoryProfiler2.sln`这个路径，打开工程文件，右键Solution选择编译(Build),然后就会在上面的路径中找到编译完成的二进制启动文件了。

## 开启MemoryProfile设定
在Window64的情况下进行设定(复制粘贴的代码)：
```
public class TP_424Target : TargetRules
{
    public TP_424Target(TargetInfo Target) : base(Target)
    {
        Type = TargetType.Game;
        DefaultBuildSettings = BuildSettingsVersion.V2;

        // 添加这一段到[Project].Target.cs文件
        if (Target.Platform == UnrealTargetPlatform.Win64)
        {
            bUseMallocProfiler = true;
        }
        ExtraModuleNames.AddRange( new string[] { "TP_424" } );
    }
}
```

还有另外两种方法，作为记录用也介绍一下，不作推荐。

1. 去`\Engine\Saved\UnrealBuildTool\BuildConfiguration.xml`里面开启
```
<?xml version="1.0" encoding="utf-8" ?>
<Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
  <BuildConfiguration>
      <bUseMallocProfiler>true</bUseMallocProfiler>
  </BuildConfiguration>
</Configuration>
```

2. 去`\Engine\Source\Programs\UnrealBuildTool\Configuration\TargetRules.cs`里面开启
```

        /// <summary>
        /// If true, then enable memory profiling in the build (defines USE_MALLOC_PROFILER=1 and forces bOmitFramePointers=false).
        /// </summary>
        [XmlConfigFile(Category = "BuildConfiguration")]
        public bool bUseMallocProfiler = true;
```

这两种的实现方式都是一样的，选其一就可以。但是由于需要重新编译引擎源码，外加启动时会连Editor的资源一同算进去（据说是这样），我一开始尝试的就是这个方式，总之就是启动的时间超级长，文件的容量也很快变得非常大，故不做推荐。


## Package设定
为了准确测量内存的使用量，将工程打包，一般都会选择大部分Debug功能都被摘除了的Test的BuildSolution进行打包，这个是最接近Shipping模式的BuildSolution。

**别忘记Package项目设定下`Include Debug Files`要选上。**

## Profile
打包完毕之后启动打包项目。



## Profile分析
关于为什么要使用Test打包：
- [[UE4] ビルドコンフィギュレーション Test構成とは](https://qiita.com/donbutsu17/items/401c1fa1037a99e14b03)

参考资料：
- [[UE4] Malloc Profiler/Memory Profiler2を使用したメモリトラッキング](https://qiita.com/donbutsu17/items/a72a282587390f43d12d#41-memory-profiler2%E3%81%AE%E8%B5%B7%E5%8B%95%E7%A2%BA%E8%AA%8D)

# 其它内存调查工具

- [Low-Level Memory Tracker - Going over how to use the Low-Level Memory Tracker in your Unreal Engine projects.](https://docs.unrealengine.com/4.26/en-US/ProductionPipelines/DevelopmentSetup/Tools/LowLevelMemoryTracker/)

## MemReport
关于如何使用这个命令
- [[UE4]Memreportを活用しよう！](https://historia.co.jp/archives/25131/)

命令的项目解读(有时间需要整理):
- [[UE4] Objコマンドによるオブジェクト解析](https://qiita.com/donbutsu17/items/dd9e00bee27d6868ed3d)
- [デバッグとメモリーの最適化](https://www.unrealengine.com/ja/blog/debugging-and-optimizing-memory?lang=ja)



## Memory Insights

- [[UE4] Memory Insightsを使用したメモリトラッキング](https://qiita.com/EGJ-Ken_Kuwano/items/83425f450f6da100a9d0)
