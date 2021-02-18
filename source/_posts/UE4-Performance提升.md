---
title: UE4-Performance提升
date: 2019-03-19 19:47:59
permalink: ue4-performance

categories:
- UnrealEngine4

tags:
- UE4
- Performance

description: 整理一些UE4的优化手段和知识，一些文章需要总结的内容。（未整理完毕）
---
主要记录一些能够查看UE4的性能工具和提升性能的一些手段记录。

<!--more-->

我在知乎看见一篇文章整理的非常好：

- [Performance and Profiling(UE4优化)](https://zhuanlan.zhihu.com/p/36434616)
- [Unreal 4 性能入门指南](https://zhuanlan.zhihu.com/p/36851846)

根据这个再根据自己在工作中遇到的问题进行进一步细致的总结吧。

发现了BasePass的消耗很高，查一下。
# 1. Optimization

关于一些渲染流程的说明：

- [CEDEC2016: Unreal Engine 4 のレンダリングフロー総おさらい](https://www.slideshare.net/EpicGamesJapan/cedec2016-unreal-engine-4)

上面的文章我之前有总结过一些，但是并没有完全理解。

## 1.1 Emulate HMD performance
下面有提到VRSDK的设置（v-sync）会使得帧数锁在90fps那里，如果小于90fps就会直接降到45fps，不能够正确的测试处理时间。

我们可以模拟VR的绘制：
- **Launch witch -emulatestereo**
- **Set resolution 2160x1200**
- **Set r.screenpercentage 140**

具体的做法如下：
1. 在*Editor Preferences > Play > Play in Standalone Game > Additional Launch Parameters*中填入 `-emulatestereo`
2. Start with Standalone Mode and Set Resolution to `r.SetRes 2160x1200` or `r.SetRes 2160x1200f`
3. `r.screenpercentage 140`

## 1.2 Ready Profiling
- Play in Standalone
- Make sure the Editor is set to NOT update in realtime
- Minimize the Editor
- Make sure to turn off Frame Rate Smoothing[Project Settings]
- Turn off VSync[r.vsync 0]

使用`r.ScreenPercentage 10`命令，如果程序突然加速，说明性能瓶颈在GPU上。

引用来自下面的视频：
- [UE4 Performance and Profiling](https://www.youtube.com/watch?v=hcxetY8g_fs)

关于开始优化前的准备问题，流程可以参考一下[[CEDEC2017] UE4プロファイリングツール総おさらい（グラフィクス編）](https://www.slideshare.net/EpicGamesJapan/cedec2017-ue4) 里面的内容。每次最好快速浏览一下。

然后是在准备Profile一定要先**烘焙光照**，要不然我真不知道在优化个什么鬼。因为在这个地方踩坑了。有位大佬在项目工程里放了一个范围超级大的[Lightmass Importance Volume](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/Lightmass/Basics/index.html) 导致UE4在烘焙的时候总是让**Swarm**一直卡在ExportScene阶段。(这里参考了[Why is Swarm taking forever to export scene?](https://answers.unrealengine.com/questions/726808/why-is-swarm-taking-forever-to-export-scene.html))




## 1.3 VR Instanced Stereo
这个可以在
**Edit > Project Settings > Rendering > VR**

中开启。

目的是为了让双眼的画面同时渲染。减少Draw Call。

于是我试了试，并没有变快..反而感觉慢了，因为瓶颈本来不是CPU么。使用应该有条件的。

- [VR Instanced Stereo](https://docs.unrealengine.com/en-us/Platforms/VR/VRPerformance)

## 1.4 Rendering Pipeline
刚刚好，趁着这个机会，顺便把渲染流程给描述出来，也就是每个阶段做了什么，既然要优化，就要知道做了设么，什么顺序做的。

至于流程的话参考了[CEDEC2016 Unreal Engine 4 のレンダリングフロー総おさらい]()这篇文章。

关于这篇文章详细的解说下面PrePass章节部分有链接。

这个网站相当的
- [UNREAL ART OPTIMIZATION](https://unrealartoptimization.github.io/)

官方网站的[GPU Profiling](https://docs.unrealengine.com/en-US/Engine/Performance/GPU/index.html)

### 1.4.1 Z PrePass 
#### PrePass DDM_AllOpaque(Force by DBuffer)

~~这个PrePass是不是上面的*Z PrePass*还有待确认，因为我是在**GPUProfiler**中看到的这个，也许是Z PrePass的一部分。~~

我看到的不是单单的**PrePass**，而是**PrePass DDM_AllOpaque(Forced by DBuffer)**。

~~这个Pass是用来先行计算**Depth-Buffer**的，在BasePass把数据写G-Buffer之前，把深度通过测试的像素留下，达到减少处理的目的，但是不是一定会减少。~~

~~这个可以ON/OFF，也可以开启之后指定物体经不过经过该处理。~~

~~所以说PrePass的目的是为了减少BasePass的处理负荷。~~

我在项目中发现的特点是：在所有的View0/View1中的处理中，还是**Dynamic**的时间最长。上面也有提到当动态的顶点，skeleton动画比较多的时候就会这样，可以试试关掉PrePass或者把顶点动画的物体不经过该Pass处理会不会减轻一些负担。但是在此之前，我应该学会如何使用Profile工具。

我来更新了，这个地方果然是我瞎说的。之前说的都划掉，先来说前半部分。

**PrePass DDM_AllOpaque**是什么？

在这篇文章里有简短的提到[[UE4]GPU Visualizer (GPU Profiling) Specification](https://dawnarc.com/2018/12/ue4gpu-visualizer-gpu-profiling-specification/):

> **PrePass DDM_…**
> 
> Sense
>
> - Early rendering of depth(Z) from non-translucent meshed.
> - Required by DBuffer decals.
> - May be used by occlusion culling.
> - Engine -> Rendering -> Optimizations -> Early Z-pass.
>
> Cost affected by
> 
> - Triangle count of opaque objects.
> - Depending on Early Z settings: Overdraw and complexity of Masked materials.

好，是这个东西，但是为什么要做这个处理不知道。再来看后半部分。

**Forced by DBuffer**是什么？

英文意思很好理解，但是**DBuffer**是个什么我总是找不到。最后还是在上面（或者下面？）提到的那个神一样的SlideShare的介绍文章[［CEDEC 2016］UE4を扱うアーティストがつまづき易いポイントはここだ。Epic Gamesが解説する注意点と回避法](https://www.4gamer.net/games/210/G021013/20160830082/)中找到了类似的定义：

> DBuffer DecalsまたはDeferred Decalとは，デカールをG-Bufferとは別の，「D-Buffer」と呼ばれるデカール専用特殊バッファに描画しておき，G-Bufferを用いてのライティングやシェーディングは，D-Bufferの内容も反映しながら行うという流れになる。デカールの描画結果がG-Bufferに統合されてしまうと，これが原因で前出のような問題が出てしまう。これを回避するために専用バッファを用意したというわけだ。

> 　ただ，D-Bufferの利用に加えて，描画系も今までより処理のパスが増えるため，描画負荷が高くなるのがネックといったところ。
> 　負荷は高くなるが正確なデカール表現が行えるDBuffer Decalsを導入すべきなのか。あるいは別の表現を選んでDBuffer Decalを避けるかは，よく検討して決めたほうがいいようだ。

具体的意思要专门展开关于**DBuffer Decal**的文章，总而言之就是当你决定使用**DBuffer Decals**的时候，并在（Project Setting/Rendering/Lighting中）开启了DBuffer Decals功能时候，那么上面的**PrePass DDM_AllOpaque**就会被强制执行。由于某种原因（文章中表示是Pre-Lighting阶段的描画需要）后续在别的文章中展开。

上面关于**PrePass DDM_...**的内容貌似也在[UE4 Graphics Profiling: All Categories Guide (Rendering Passes)](https://www.youtube.com/watch?v=C3lumWdwHmA&list=PLF8ktr3i-U4A7vuQ6TXPr3f-bhmy6xM3S&index=5)中有提及，这是是我之前就收藏过的，但是我并没有很仔细的看。。。

关于**Deferred Decal**的讨论，本来想要在其它章节写，但是暂时先整理一下, [[UE4] Deferred Decal](http://monsho.blog63.fc2.com/blog-entry-139.html)
中提到的几个设定：

```
[SystemSettings]
r.EarlyZPassMovable=1
r.EarlyZPass=2
r.DBuffer=1
```

我本来是想找是谁打开了DBuffer的设定，结果发现DBuffer是默认引擎开启的？！至于上面的问题，由于Dbuffer而强制计算的PrePass的操作，在我关掉DBuffer之后还是存在。

关于UE4中的[Unreal Engine 4 Console Variables and Commands](http://www.kosmokleaner.de/ownsoft/UE4CVarBrowser.html)

|Name|Help|
|---|---|
|r.EarlyZPass|Whether to use a depth only pass to initialize Z culling for the base pass. Cannot be changed at runtime.<br>Note: also look at r.EarlyZPassMovable<br>0: off<br>1: only if not masked, and only if large on the screen<br>2: all opaque (including masked)<br>x: use built in heuristic (default is 3)|
|r.EarlyZPassMovable|Whether to render movable objects into the depth only pass. Movable objects are typically not good occluders so this defaults to off.<br>Note: also look at r.EarlyZPass|
|r.DBuffer|Enables DBuffer decal material blend modes.<br>DBuffer decals are rendered before the base pass, allowing them to affect static lighting and skylighting correctly.<br>When enabled, a full prepass will be forced which adds CPU / GPU cost. Several texture lookups will be done in the base pass to fetch the decal properties, which adds pixel work.<br>0: off<br>1: on (default)|


这里插播一条[Decal performance question](https://answers.unrealengine.com/questions/154220/decal-performance-question.html)：
> This is from the docs:
>
> Performance
>
> The mesh complexity of the objects affected by the decal is not affecting the performance. The decal performance depends on the shader complexity and the shader box size on the screen.
>
> We can further improve the per decal performance. Ideally the bounding box of the decal is small to get better per pixel performance. This can be done manually. An automated method is possible but a good designer can also adjust the placement to improve performance further.
>
> Current limitations
>
> We currently only support deferred decals and they only work on static objects. Normal blending is currently not wrapping around the object. Mip map computation is not done yet so you might see 2x2 block artifacts on object borders. Streaming is not yet hooked up so make sure the texture is not streamed. Masking decals (not affecting other object) is not fully implemented.


### 1.4.3 Base Pass
关于base pass的总结：

- [UNREAL ART OPTIMIZATION - Base Pass](https://unrealartoptimization.github.io/book/profiling/passes-base/)

    **Responsible for**
        
        • Rendering final attributes of Opaque or Masked materials to the G-Buffer
        
        • Reading static lighting and saving it to G-Buffer
        
        • Applying DBuffer decals

        • Applying fog

        • Calculating final velocity (from packed 3D velocity)

        • In forward renderer: dynamic lighting

    **Cost affected for**
        
        • Rendering resolution

        • Shader complexity

        • Number of objects

        • Number of decals

        • Triangle count


在basepass阶段做了许多工作，其中**Shader Complexity** 是影响性能的一个很重要的因素。


#### 1.4.3.1 Optimization
如何优化这个问题首先要知道那个地方需要优化。
- Shader Complexity: 在view-mode下可以查看shader的复杂度。
- Stat: 在Material Editor里面有stat window查看pass的数量
- Rendering Resolution: 可以查看和影响G-Buffer和其他贴图的质量。
    - `stat RHI`: 查看G-Buffer的内存占用

英文文章读起来不如中文的快，那么容易理解，但是有些话翻译成中文的话不知道为什么就变了味道。还是多读几遍人家的文章吧。

#### 1.4.3.2 GPU Visualizer

##### BasePass-Dynamic
>  <span style="color:red">BasePass 0 = Opaque Meshes.</span>

>  <span style="color:red">BasePass 1 = Alpha Masked Opaque Meshes for Z-depth.</span>
 
>  <span style="color:red">BasePass Dynamic = Animated Vertices such as Skeletal, GeoCache(Alembic),etc.</span>

在上面我只看到了**Dynamic**，而其他的是**Static EBassPassDrawListType=0**和**Static EBasePassDrawListType=1**意思是不是一样的我也不确定。

于是我查了查**EBasePassDrawListType**
```C++
// Source UnrealEngine4源码：Runtime\Render\Private\BasePassRendering.cpp
EBasePassDrawListType DrawType = EBasePass_Default;

// The definition of the type
enum EBasePassDrawListType
{
    EBasePass_Default = 0,
    EBasePass_Masked,
    EBasePass_MAX
}
```

看这个声明貌似上面的是对的。

还有一个我有点在意的是在VR的模式下会出现两种
- View0
- View1

VR眼镜不是有两个镜片么，就对应了两个摄像机。

所以**Dynamic**处理占用时间，是因为**顶点动画**比较多。

### 1.4.4 Pre-Lighting

### 1.4.5 Lighting
- [Lighting Passes](https://unrealartoptimization.github.io/book/profiling/passes-lighting/)

#### Unwrapping UVs for Lightmaps
- [UNREAL ENGINE - Unwrapping UVs for Lightmaps](https://docs.unrealengine.com/en-US/Engine/Content/Types/StaticMeshes/LightmapUnwrapping/index.html)


#### 1.4.5.1 GPU Visualizer
这个是在**ProfileGPU**命令中出现的GPU Visualizer视图中的光照部分GPU消耗情况。

这里需要澄清的是，我按照渲染流程顺序题的标题，但是这个GPU Visualizer中的内容未必就是这个阶段做的事情，也许的确是这个阶段做的，但是我不知道。我只是看见名字相同，就整理到一起罢了。

层级关系：

- Lights
    - DirectLighting
        - NonShadowLightings
        - IndirectLighting
        - ShadowLights

大致上是这样的。

##### ShadowLights -> ... -> ShadowProjectionOnOpaque
这个项目下处理占用了好多的时间。这是在一个Stationary类型的Spot光源下，产生的大量的计算。关键字有*PerObject*和*PerShadow*。

一个Spot光源的出现在直射光的项目下面我就已经很吃惊了。顾名思义，这应该是阴影投影在不透明物体上的处理。当然直射光被设置为StationaryLight，需要动，所以每一帧更新都要重新计算阴影位置。



### 1.4.6 Reflection
- [UNREAL ENGINE - Reflections](https://docs.unrealengine.com/en-US/Resources/Showcases/Reflections/index.html)

#### Reflection Environment
- [UNIREAL ENGINE - Reflection Environment](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/ReflectionEnvironment/index.html)

#### Planar Reflection
- [UNREAL ENGINE - Planar Reflection](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/PlanarReflections/index.html)


### 1.4.7 Translucency
按照处理顺序的话透明处理绝对不是那么靠前，之后应该调整一下顺序。

#### Separate Translucency
把半透明的处理写到其他的Buffer中。

**Project Settings > Engine > Rendering > Translucency**

我似乎有看过一点这方面的文章。

- `r.SeparateTranslucencyScreenPercentage XX`: 指定该buffer的解像度
- `r.SeparateTranslucencyAutoDownsample`: 自动降低解像度


### 1.4.8 PostProcess

### 1.4.9 Shadow
阴影这一部分的内容有很多，顺序也需要之后调整。

#### Fake Shadow
多在角色模型中使用，比如人物脚下一团黑影代表着阴影那样。

#### Capsule Shadow
- SkeletalMesh: Capsule Direct(Indirect) Shadow 等等




## 1.5 Command Introduction
记录一些常用的命令和里面的参数解读。

### stat SceneRendering

#### RenderQueryResult
关于这个参数，我也不太完全确定，在网上搜了也没有什么具体的答案。搜来搜去看到了**OcclusionCulling**感觉很像，但是按照这个方向去优化了一下试了试并不是很理想，貌似不是并不是一个概念，而且OcclusionCulling这个参数在另外一个命令，貌似是*stat InitView*中有貌似。

看到有人理解的是：

**RenderQueryResult是GPU完成整个一帧的渲染最后要做的事情。这个条目耗费时间很长说明GPU在这一帧内没有空闲，一直在工作。**

感觉还说的过去。

# 2. Bluerint优化
像Blueprint和C++是由CPU处理的**Game**阶段的处理。

## Tick Event
如果不需要的话，将Actor中的Tick Enable 设置为Off。亦或者调整Tick Interval的数值减少每一帧的调用。

## Spawn
Spawn处理不一定非要在一帧之内完成，可以分散到下下帧，由于间隔时间很短所以不会暴露什么。

但是分帧完成一个动作我还没见过这种操作，之后有时间找找看吧。

## Nativizing Blueprint
这个我也不太懂，真要用的时候再看吧。
- [Nativizing Blueprint](https://docs.unrealengine.com/en-US/Engine/Blueprints/TechnicalGuide/NativizingBlueprints)

# 3. Landscape Optimization
自作大型的场景的时候就会用到Landscape，但是问题是在Statistics中查看的结果，这个东西的无论是顶点数还是资源大小还是光照贴图的大小都是场景里耗费最高的。

关于Landscape的制作也不是随意的。

- [Introduction to Landscapes - #6 Unreal Engine 4 Level Design Tutorial Series](https://www.youtube.com/watch?v=OWOlYoI-3tY)

这个视频涉及到了如何合理的制作Landscape，还有其他的系列视频关于如何使用Landscape的工具的。

再就是官方文档了：
- [Landscape Technical Guide](https://docs.unrealengine.com/en-us/Engine/Landscape/TechnicalGuide)

解释了各种参数的意义和推荐设置。

还有就是一些具体细节的调整：
1. Navigation不用的话关掉
2. Collision不用的话关掉
3. LOD设置调低



# 4. HTC Vive
**Vive的刷新率只有90Hz。**

这是什么概念？每秒刷新90帧的能力，这跟游戏能够达到的帧率有很大不同。

1. 这里就有疑问了，如果游戏的帧率高于显示器（VR）的刷新率会怎么样。

<span style="color:red">**会造成撕裂(tearing)。**</span>

GPU活儿干多了，比如说应该把一帧的图像交给显示器，结果交付了一帧还多的数据（显示器来取数据的速度慢了一拍，本应该取走的一帧数据被显卡二次更新中途，）。产生了画面撕裂。

**V-Sync**被用来解决这个问题，垂直同步(Vertical Synchronization)通过建立一个不让在显示器刷新前将后备缓冲中的画面拷贝到显示缓冲中的规定来解决这个问(有条件的双倍缓冲)。如果FPS高于刷新率的话没有问题，后备缓冲的更新完成后，系统处于等待状态。当显示器刷新后，后备缓存拷入显示缓存，显卡则可以在后备缓存中描画新的画面。

2. 游戏的帧率低于显示器（VR）的刷新率会怎样

如果打开了垂直同步，那么帧数直接减半。理论上讲，双缓冲的VSync，FPS将是一组不连续的整数，其等于**刷新率/n**。

这就是为什么我总是看Vive的帧率被锁在45帧的原因。

那么进入正题，UE4中的**Smoothed Frame Rate Range**有什么用。

## Smoothed Frame Rate Range

- [What exactly does "Smooth Frame Rate" do?](https://answers.unrealengine.com/questions/90743/what-exactly-does-smooth-frame-rate-do.html)

> **With Frame Rate Smoothing, the application is determining what range is acceptable for frame rate wandering,so you can cap your frame rate to between Min and Max allowable frame rates.Since this is application based,it will make these changes before any hardware vsync changes.**

举例来说，如果设置了Max60f，Min40f的话，当超过60帧会保持在60帧，当低于60帧而高于40帧的时候会保持帧数，但是低于40帧的时候就会降到30帧。

得益于现在的显卡很多具有自适应的垂直同步功能。

当然上面的内容只是我看文章得到的，并没有实际试验过。

然后吧，我读到了一篇文章。
- [Unreal* Engine 4 VR应用的CPU性能优化和差异化：第一部分](https://software.intel.com/zh-cn/articles/cpu-performance-optimization-and-differentiation-for-unreal-engine-4-vr-application-part1)

这里面提到了**另外，因为VR是强制开启垂直同步的，所以只要一帧的渲染时间超过11.1ms，即使只超过0.1ms，也会导致一帧需要花两个完整的垂直同步周期完成，使得VR应用很容易因为场景稍微改变而出现性能大降的情形。这时候可以用“–emulatestereo”指令，同时把分辨率(resolution)设为2160x1200，屏幕百分比(screenpercentage)设为140，就可以在没有接VR头显及关闭垂直同步的情况下分析性能。**

好的。

# 5. UE4中的一些概念

## Instanced Static Mesh
关于这个东西，我纠结了好久。我是从**foliage**这个东西接触到它的。因为foliage刷出来的东西就是这个类型，生成一些随机的树啊花啊草啊就很方便，但是同时带来了性能消耗。

- [Foliage Instanced Meshes](https://docs.unrealengine.com/en-us/Engine/Foliage)

为什么说带来了性能消耗呢，其实除了方便之外还有其他的好处。
- 减少了**Draw Call**

据我的理解：**一个Actor的渲染对于CPU来说就得生成一个draw call**，所以庞大的Actor的数量会拖CPU的后腿，减少draw call是优化性能的方向之一。

但是与此同时，一个draw call的数据不充分就导致GPU做额外的工作。也就是我遇见的InstancedStaticMesh这个东西产生的影响。

### 剔除
有两种剔除方式：
- Frustum Culling
- Occlusion Culling

对于InstancedStaticMesh来说，第一种视椎体剔除是可以降看不见的部分剔除的。而第二种遮挡剔除，对于它来说就完全不起作用了，反而加重了GPU的负担。

我这么认为的证据是，在我分析一个用满是foliage制作的大场景中，我试着：

1. ToggleDebugCamera: 命令行打开Debug摄像机，找到想要看的位置
2. FreezeRendering:

我忘记上面的命令是在哪里看到的了，在我暂停了渲染之后移动摄影机之后发现，在摄像机之外的内容被剔除掉但是在摄像机之内的所有InstancedStaticMash都还在。即下面的文章中提到的：
- [Unreal* Engine 4 Optimization Tutorial, Part 2](https://software.intel.com/en-us/articles/unreal-engine-4-optimization-tutorial-part-2)

> One thing to know about instanced static meshes is that if any part of the mesh is rendered, the whole of the collection is rendered. This wastes potential throughput if any part is drawn off camera. It’s recommended to keep a single set of instanced meshes in a smaller area; for example, a pile of stone or trash bags, a stack of boxes, and distant modular buildings.

如果将Foliage的部分做的很大，那么所有的内容都会被渲染，哪怕是一小部分进到了摄像机的视野里。

还要一种方式是在Editor中，不是运行的状态，输入命令：
- r.VisualOccludedPrimitives

可以实时的查看被遮挡的物体的轮廓，上面的例子中，并看不见InstancedStaticMesh被遮挡的轮廓，因为他们是一体的，只要一部分出现在了视野里就会被渲染。

这里我突然就产生了对*遮挡*这个功能的疑惑，遮挡剔除这个部分对Instanced类型的物体做的不是很好，或者说是根本无能为力。按照我之前的理解，对看不见的片元，CPU和GPU不会去渲染才对，但是结果是，遇见这种类型的，都会被渲染，看不见的片元被渲染浪费了大量的GPU的能力和处理时间。**GPU并没有认为这是一个物体，把出现在视野内的片元渲染，看不见的片元舍弃，而是全部听从了CPU的命令进行全部的渲染。**

这个是我之前的理解。

但是对于视野内的物体，遮挡剔除就完美的降被遮挡的非Instanced物体给剔除掉了，这是事实。

**遮挡剔除究竟是CPU的工作还是GPU的工作？**

### 关于OcclusionCulling的一些问题
- [UE4のOcclusion Cullingで良く聞かれる質問1: Occlusion Culling自身の処理負荷を減らしたい - だらけ者だらけ](http://darakemonodarake.hatenablog.jp/entry/20181204-01-OcclusionCulling)
- [UE4のOcclusion Cullingで良く聞かれる質問2 Occlusion Cullingによりオブジェクトが1フレーム消失することがある - だらけ者だらけ](http://darakemonodarake.hatenablog.jp/entry/20181204-02-OcclusionCulling)

上面是关于遮挡剔除的一些问题的文章。重点是在第二篇。

这里提到了一些遮挡剔除的特性：

当摄像机的移动超过了一定的阈值OcclusionCulling就会Off
- `CameraRotaionThreshold(Default 45.0)`
- `CameraTranslationThreshold(Default 1000)`

另外，`r.AllowOcclusionQueries` 的ON/OFF 可以手动切换。

也可以通过扩大Occlusion的Bound来提前渲染object

- `OCCLUSIN_SLOP`
- `r.ExpandAllOcclusionTestedBBoxedAmount`

进入摄像机视野的一瞬间扩大OcclusionBound：
- `r.ExpandNewlyOcclusionTestedBBoxsAmount(Default=0.0f)`



### 简单的Profiling
- stat SceneRendering
- stat InitView
- Stat SceneUpdate

命令应该别的地方有讲过。但是这里我注意到的是在`stat InitView`命令里，有一个处理占了我很多时间

### Render Query Result
查了也不是很明白

> RenderQuery Result is when the render thread stalls waiting for the GPU to finish the Occlusion Query, and return the results to the render thread, so that it knows what to render.

> At the same time, the game thread is stalled waiting for the render thread.

> This can be turned on or off with the console command

> r.AllowOcclusionQueries

> 0 - off 1 - on

看了也不太懂系列。

什么是**Occlusion Query**?

- [GPU Gems- Chapter 29. Efficient Occlusion Culling](http://developer.download.nvidia.com/books/HTML/gpugems/gpugems_ch29.html)

我下了PDF，可以看。

### 如何正确的使用

- [UE4 Optimization : Instancing](https://www.youtube.com/watch?v=oMIbV2rQO4k)

我觉得这个人的每个视频都值得我刷几遍。

### Culling Distance
1. Foliage Culling Distance
2. Culling Distance Volumn

# UE4 Performance and Profiling
实际上准确的标题名字应该是
- [UE4 Performance and Profiling | Unreal Dev Day Montreal 2017 | Unreal Engine](https://www.youtube.com/watch?v=hcxetY8g_fs)

这其实是一个视频，我尝试的边看边做笔记，结果足足整理了三页，虽然是就是抄的英文，等之后有时间再记录下来吧。

现在要将这些内容整理一下，同时也温习一下视频的内容。

## 1.CPU/GPU Profiler
我觉得有必要区分这两种Profiler，因为我最近接触的总是GPUProfiler，搞得我都不知道CPU要怎么Profiling了。

## 2.Profiling in a Build

算是准备优化之前的准备工作吧。

1. Minimize the noise that can interface with profiling
    - Turn off everything you are not using
    - Turn off v-sync `r.vsync 0`

2. Turn off Framerate Smoothing

3. Make a Test build
    - Testing in a Development build inflates the Draw thread with noise

尽可能的关闭噪声（noise），前两项是必须要做的，但是第三项，我也不太清楚我理解对不对，开发的时候使用的是**Development mode**，所以尽可能的减少噪音就直接build工程（即**Shipping mode**）来optimization。

但是吧，还能不能用*stat*一类的命令来debug了我还真没试过。

## 3.Profiling from within the Editor
可能就是上面的补充吧，其实我也是这么做的。在Editor中Optimization肯定会有noise。

前面也有提到具体怎么做(虽然是VR的，但是应该都一样)这里就简单抄一下。
1. Play in Standalone
2. Make sure the Editor is set to NOT update i realtime
3. Minimize the Editor
    - VR > Editor Preference > Play > should minimize Editor on VRPIE

4. Make sure to turn off Frame Rate Smoothing
5. Turn off VSync

## 4.General Process

1. Identify the bottleneck
    - Game Thread
    - Render Thread
        - CPU
        - GPU

    - Often jumps back and forth as you optimize
    - Use `r.ScreenPercentage 10` to quickly check if you are GPU bound

最后一条很有用。

### Game Thread
- Code or Blueprint

### CPU Render
- Object count,draw calls,culling

### GPU Render
- Shaders, overdraw,light

## 5.Measuring in Milliseconds
- Use *<span style="color:green">stat unit</span>*,not just *<span style="color:green">stat fps</span>*
    - Largest number shows you the likely bottleneck

- Milliseconds per frame
    - <span style="color:grey">Frame:</span> total time to finish each frame
    - <span style="color:red">Game:</span> C++ or BP gameplay operations
    - <span style="color:blue">Draw:</span> CPU render time
    - <span style="color:purple">GPU:</span> GPU render time
- You can also use *<span style="color:green">stat unitGraph</span>*,whitch shows a line graph playback.
    - Mostly useful for spotting repeating hitches

### ScreenPercentage
前面有提到使用这个命令来模拟VR的性能，还有就是使用这个命令来快速确认游戏的性能瓶颈是不是GPU。

- Mostly useful to measure problems unrelated to Game Thread
- Use *stat unit* to show milliseconds
- Use *<span style="color:green">r.ScreenPercentage 10</span>*
    - Or any number smaller than 100
    - Reduces number od pixels sent to the GPU
    - If things get faster,you were GPU bound
    - If they dont get faster,you were CPU bound

## 6.Show Flags
One of the simplest ways to look for problems is to turn off partsof your scene.

Helps know when to look into reducing
- LODs
- Less translucency
- Adjust lighting

*<span style="color:green">show assetType</span>* or *<span style="color:green">showFlag.assetType 0-1</span>*

- Staticmeshes
- Skeletalmeshes
- Particles
- Lighting
- Transluncency
- Reflectionenvironment
- Many more listed in docs

## 7.Diagnostic Tools-Realtime stats and view modes

### Stat commands
- *<span style="color:green">stat fps</span>*
- *<span style="color:green">stat unit</span>*
- *<span style="color:green">stat scenerendering</span>*
- *<span style="color:green">stat gpu</span>*
- *<span style="color:green">stat engine</span>*
- *<span style="color:green">stat streaming</span>*
- *<span style="color:green">stat emitters</span>*
- *<span style="color:green">stat lighting</span>*

<span style="color:green"></span>

#### Stat SceneRendering
- Only place to see draw calls
    - Draw call is a single request to GPU to draw something
    - Prime candidate for CPU slowdown on lower-end machines and also on mobile(less of a concern with Metal and Vulkan)

- Also good palce to see time for:
    - Shadows
    - Decals
    - Post Processing
    - Lighting

这个命令挺重要的。

#### Stat GPU
- Relatively new 4.15
- Realtime readout from GPU
- Gives highlights, but not details
     - Makes i very good to quickly target trouble spots
    
- Use the full GPU profiler if you want to target individual things
    - Example:if you want to find specific lights that are casting shadows

### Optimization View Modes

#### Shader Complexity
- Show how much your shhaders are costing on the GPU
- Good way to see overdraw issues
    - Overdraw is when a pixel must be drawn multiple times
    - One of the most common content issues for optimization

- Graph at the bottom shows where the pixel and vertex shaders are in terms of performance
- If you see a lot of red and white,reconsider your approach

下面的话来自于官方文档里的
- [View Modes](https://docs.unrealengine.com/en-us/Engine/UI/LevelEditor/Viewports/ViewModes)

> Shader Complixty Mode is used to visualize the number of shader instructions being used t calculate each pixel ofour scene.It i a generally a good indicating of how performance-friendly your scene will be. In general, it is used to test overall performance for your base scene, as well as to optimize particle effects, which tend to cause performance spikes with a large amount of overdraw for a short period of time.




#### Quad Overdraw
- Helps show how you are using your polygon count on the screen
- Can help show where meshes should be LOD-ed down
    - Too much green shows areas that should be simplified
    - Anything more than green is starting to get costly, commonly translucency overdraw

- Very useful for MSAA on Forward Rendering,as the number of poly edges dramatically affects performance

#### Quad Overdraw in-depth
- Your GPU breaks the view up into quads
    - 2*2 groups of pixels
    - This is more efficient than performing all operations on all pixels

- Very small, or very long, thin geometry wastes pixels
    - Regular, large polygons make the best use of pixel quads, best use of GPU

- Model with regular trangles and LDD aggressively

> When you are looking at an opaque object on the deferred render, and you see a lot of green ,that means all 4 pixels of that quad had to be recalculated over and over.

> you should probably be using lower LODs.

*毫无意义的分割线

#### Shader Complexity + Quad Overdraw
- Combines two powerful view modes into one
- USeful to get an idea of expensive shader anf geometry at a glance
- You will still need the individual settings to help diagnose specifics

#### Liht Complexity
- Visualizes the cost of scene lighting
- As lights overlap, the colors shift from cool to warm to white
- Only shows cost of lighting, not shadowing
- Obviously, white is bad
- Great way to see where you should be lowing light radii
- By flipping this on and off, you can quickly see if the cost of any given light is "worth it"

#### Lightmap Density
- Shows the density of texels for lightmap purposes
- Color shifts from cool to warm an density increases
    - Most things can be blue
    - Shadow maps don't often need to be very high res

- Keep this as low as possible
    - Cost adds up quickly

#### Stationary Light Overlap
- Only a maxium of 4 stationary lights can affect any given object
- Beyound that,any other lights fall back to Movable(fully dynamic)
- This view mode helps track down where that might be happening
- Reminder to keep lighth radii as small as you can get it
- Do you have a stationary sun?
    - Congratulations! That's one of the four lights!

#### LOD Coloration
- This mode shows the current mesh LODs in use by color coding them
- Very fast way to through ypur scene and verify that things are LODing when they should be
- Interestingly, mode clearly shows that the trees are not LODing at all in this project
    - Was able to diagnose frame drop instantly using this mode

## 8.Profilling Tools

### CPU Profiling
- Integrated tool to take apart a segment od your gameplay and see wat's happening on each tick
    - Very useful way to profile Blueprint performance

- Measures a segment of time
    - Within that segment, can look a individual frames or averages

- Requires two special Stat Commands
    - *<span style="color:green">stat startfile</span>* & *<span style="color:green">stat stopfile</span>*
    - Tese generate a log file between the interval of the commands
    - Profiller allows deep analysis that log

- Step down into world tick and see individual Blueprint functions
- Can be used for CPU(Game and Draw) and GPU

捕获下来的日志可以在UE4的**Session Frontend**中展开分析。

### GPU Profiling
- Three method to profile GPU functions
    - <span stype="color:green">stat GPU</span> command in tne viewport
    - Recorded file log in the **Session Frontend**
    - GPU Profiler
        - Can dump out to either the log or its own UI

- Great way to visualize the cost of:
    - Base pass
    - Lighting
    - Shadows
    - Post processing

### Tracking Slow Frame
- *<span style="color:green">stat dumpHitches</span>*
    - The command is used to dump any hitched over a given time in milliseconds out to the log
    - Use command *<span style="color:green">t.hitchThreshhold 0.xx</span>* to set value (0.05 is default)

- *<span style="color:green">memReport -full</span>*
    - Full breakdown of how memory is being used
    - There's a great blog post on how this works

虽然我还没有尝试过但是我可以使用上面的命令来找到游戏运行时候突然消耗非常高的那一帧究竟做了什么。

### startFPSChart and stopFPSChart
- You can use the commands *<span style="color:green">startFPSChart/span>* and *<span style="color:green">stopFPSChart/span>* to create a diagram of framerates over time

- You can call these at start and end of a Level Sequence to automatically read out the frame rates along a given course, as defined by a cinematic

这里使用完这个命令之后会输出日志文件，需要使用别的软件把日志文件转化为图表。我用了GoogleDrive上的Excel做的。

## 9.Blueprint Optimizations - Or:Keeping the Kids from Eating the Crayons

- Blueprints make it easy for folks to assemble gamepaly logic
- Best results often come with engineer mentorship
- Common challenge
    - Reliance on Tick functionlity
    - Over-use of expensive functions(iterating on many objects)
    - Abuse of hard reference

### Reliance on Ticking Blueprints
- Tick means should on every frame
- Blueprints should almost never need Tick
    - Remember to uncheck *Enable Actor Tick* in Class Defaults!
        - This is on by default so that the Tick event will work

- Alternatives to Tick
    - Timers
    - Timelines
    - Manually enabling/disabling Tick on demand
- Make sure to adjust Tick Frequency
    - 0.0= every frame

### Expensive Functionality
- Some functions are inherently expensive
    - Get All Actors of Class
    - Spawn
    - Anything that needs to iterate over a large group of objects or properties
- Try not to use these if at all possiple
    - If you are doing it to get a reference, consider having the referenced class pass itself up so the referencing object does not need to query
    - Use TSets instead of arrays
- If you must use them, do so as seldom as possible
    - Perferably only once,such as at Begin Play
- Heavy ConstructionScripts can murder spawn times.
    - Consider placing in the level beforehand

### Hard References in Blueprint
- It is very easy for Blueprints to generate references to each other
- When you load a Blueprint, every other Blueprint it references must be loaded
    - And the Blueprints referenced by those
    - And so on,and so on..
- This will not slow dowwn in-game performance, but it can eat away at memory and load times
- Some studios have thought the Editor just ran slow
    - Turns out they were loading moost(ot all) of their game on startup

- Avoiding hard references:
    - Avoiding casting operations unless you are certain you need them and know that it won't cause issue
        - For instance, if a Pickup class can only interact with the player, it might be fine to have it cast to Player
        - But having the Player class references every other type of pickup and interactive object in the game, you will likely see problems

    - Instead, use Blueprint Interfaces
    - Try to get into the mentailty of not needin a very specific reference type
        - Send your messages via an interface to a more generic class
        - If they land on something inplementing the interface, grate!
        - If not,no big deal

### Other Blueprint Optimizations
- Avoiding doing too much of any one thing(like with any scripting language)
    - Too much functionality in a sngle class
        - Break things up
        - Use a class hierarchy
            - But on that note, also avoid...
    - Class hierarchies that are too deep
    - Too many components within a class
    - Too much high-end math
        - Use the Math Expression node- it's optimized to speed things up
    - When all else fails for BP performance: **GO NATIVE**!
        - At Epic, many of our Blueprints derive from generic C++ classes
        - Yours should, too!
        - Keep all the heavy functionality in code, leave the lighter stuff for Blueprint

### What Actors are Ticking?
- Did you lose track of what's ticking? Use *<span style="color:green">dumpticks</span>*
- Dumps a list of all ticking Actors out to the log, telling you how many tick functions are called
- Also shows how many enabled and disabled ticking Actors are in the scene

## 10.Draw Thread Optimization

### CPU Rendering Considerations
- Bottlenecks at the Draw thread are often caused by doing too much:
    - Too many draw calls
    - Occlusion queries - see above
    - Simulating too many particles
    - Adding too many lights - often hits the GPU harder
- Generally the best way to speed up the Draw thread is to do less
- Find every way you can to put fewer things on the screen
- Generally this means either being very clever with content or using the integrated tools within UE4 start combining objects

### Actor Merge Tool
- Located under *Window > Developer Tools*
- Combines selection of meshes in to new asset, replacing originals
- Can also combine Material via Simplygon
- Works best with many meshes having the same Material

> The Actor Merge tool works best with many meshed that have the same materials as possible.If you try to combine 20 meshed and each has its own Material every materail,you are not benefiting from the tool because every material is going to make a draw call anyway.

### Instanced Static Meshes
- Mechanism for generating multiple instances of a given mesh, with each considered part of the same mesh object.
- Can only be created throuth code or Blueprint at this time, often via the Construction Script
- Very easy to create a Blueprint set that helps generate this
    - Placement Blueprint that is used to preview where mesh will be
    - Radius based ISM Blueprint that gathers transforms from Preview BPs and populates the instances with itself.
    - Be careful of Editor Utility class BPs-they're Editor only!
- Also consider *Hierarchical Instanced Static Meshes*
    - Handle their own occlusion/visibility

### Hierarchical LOD
- Hierarchical LODs allow multiple meshed to be combined and then reduced as a single mesh
- Will also combine textures into atlases. reducing overall Material demands
- Very useful for buildings and cities, groups of large meshes that need to be viewed at extreme distance
- Requires Simplygon implementation

## 11.GPU Optimizations - What to do about all those pixels

### Vertex Shader Optimizatin
- Be careful how much you make use of World Position Offset
    - Often cheaper than the alternative methods of vertex animation
- Vertex color can eventually get costly
    - On Paragon, we ended up stripping it and adding it back per instance

### Pixel Shader Optimizations
<span style="color:red">Pixel Shader Don'ts</span>
- Too much math
- Too many textures
- Too many procedural functions
    - noise
- Too many Material layers
- Reliance on lfs(if statements)
    - Both sides have to calculate

<span style="color:green">Pixel Shader Dos</span>
- Use textures for lookups instead of mesh
- Compress greyscale maps into single textures
- Minimize Layer usage
- Use Switch Parameters to turn off what you don't need

### Material Instruction Counts
- Always pay attention to Material instruction counts
- Caution: the number indicated is not accurate until you click Apply
    - Sometimes it's best to re-compile the Material just to be safe

### Dealing with Overdraw
- Overdraw is one of the leading causes of GPU slogging
- Minimize the geometry area for overdraw
    - Adding vertices is almost aways cheaper than relying in overdraw
    - For example, on A Boy and His Kite,we ended up cutting the grass planes to almost exactly match the outline of the grass texture alpha
- Make use of Particle Cutout property
    - This is found under the Cascade Required Moudle
    - Feed it a texture, it automatically snips the spite
    - Also works in subUVs,with a different cutout for every frame

### Managing Texture Resolution
- Author texture at whatever resolution you like,but keep in mind you may not always use full resolution
- Use the Texture Streaming view to see what level of mips you're using for any given texture
- You can use the Statistics panel set to Texture Staticstics to see what levl of mips you are using at current levels
- Then use the Texture Editor to force mip bias
- Or better yet,reimport at lower resolution

### Lighting Considerations
- Dynamic lights are expensive (but somewhat cheaper in deferred)
    - Small,unshadowed lights are the cheapest!
    - You can have lots of these

- Minimize number of dynamic lights
- Minimize number of things dynamic lights have to effect
- Minimize dynamic light radii -tighter is better
- Cast shadows from as few dynamic lights as possible
    - Dynamic shadow casting lights are the most expensive in UE
- Watch out for Stationary Light Overlap
    - The fallback to dynamic lighting is extremely expensive

- Bake whenever you can
- Don't assume you need dynamic lights
- Use Mesh Distance Field shadows at distance
- Watch out for dense shado cascades
- Many artifacts are cleaned up with Shadow Bias,but be gentle
- Keep Lightmap Res as low as you can
    - Use the view mode,keep it blue as much as possible

- Avoid Light Function unless you really need them
    - Consider IES profiles, but understand they also have a cost
- Lit translucency gets expensive, use with caution
- Cull shadows early (at close distance as possible)
- Cull dynamic lights as early as possible
- Spot lights are cheaper than Point lights
- Don't be afraid to completely fake shadows
    - We do this a lot, especially for VR

### Replication Optimization
- Common problems for networking:
    - Doing too much
    - Doing it too much
- Replicate as little as you can, as seldom as you can
- Use <span style="color:green">net.*</span>  commands to check what's going on
    - Must be run on the sever
        - Use cheat net.* to run the command on the server from the client
    - Use net.DumpRelevantActors to see what is currently replicating
        - This command features some improvements as of 4.19
    - There are a lot of these net.* commands - check online for full list

*毫无意义的分割线
### Network Relevancy View Mode(4.19)
...

我已经不知道他在讲什么了。。

## 12.Content Streaming

### Texture Streaming
- Textures streaming into and out of your scene at inopportune times cause visible pops
- As of 4.15 we have some tools for texture streaming diagnostics
    - Texture streaming view mode
        - Primitive Distance Accuracy
        - Mesh Densities Accuracy
        - Material Texture Scales Accuracy
        - Required Texture Resolution
    - <span style="color:green">stat streaming</span>

#### Primitive Distance Accuracy
- Visialization system for texture streaming
- Enable users to see what mips the system things they should be using, allowing for intelligent mip limits
    - <span style="color:red">Red = 2 or more mips too few</span>
    - <span style="color:orange">Orange = 1 mip too few</span>
    - <span style="color:blue">White = the right degree of mips</span>
    - <span style="color:cyan">Cyan = 1 mip too many</span>
    - <span style="color:green">Green = 2 or more mips too many</span>

- This setting can be adjusted using the *<span style="color:green">StreamingDistanceMultiplier</span>* property

#### Mesh UV Densities Accuracy
- This uses the density of a mesh's UVs
- Visualizes how those UV densities are distributing to densities are contributing to streaming data
- Use the same paradigm as Primitive Distance Accuracy
    - <span style="color:red">Red = 2 or more mips too few</span>
    - <span style="color:orange">Orange = 1 mip too few</span>
    - <span style="color:blue">White = the right degree of mips</span>
    - <span style="color:cyan">Cyan = 1 mip too many</span>
    - <span style="color:green">Green = 2 or more mips too many</span>

- Fixing this requires the UVs on each mesh to be adjuested

#### Material Texture Scales Accuracy
- This view mode samples all textures and feeds back the worst culprits for over-streaming and under-streaming
- Data is based on streaming affected by textures that have had their UVs scaled

- Helps diagnose streaming errors caused by UV scaling

#### Required Texture Resolution
- This mode shows te required resolution for the given texture, indicating how many mips under or over it is
- Helps show the delta between the ideal resolution for the texture-what the GPU wants to show-and what is the GPU wants to show- and what is currenrly avaliable
    - <span style="color:red">Red = 2 or more mips too few</span>
    - <span style="color:orange">Orange = 1 mip too few</span>
    - <span style="color:blue">White = the right degree of mips</span>
    - <span style="color:cyan">Cyan = 1 mip too many</span>
    - <span style="color:green">Green = 2 or more mips too many</span>

#### Stat Streaming
- Realtime metrics on texture streaming memory usage
- Breaks down texure streaming memory into 3 pools
    - Texture
    - Streaming
    - Wanted

#### Level Streaming
- Level streaming is an ideal way to control what content is in use in your game
- What you currently need is streamed in, what you don't is streamed out
- Be careful how much you stream at once!
    - You may need negate some of the benefit if you have over-referenced your content within code or Blueprint

#### Bonus: Level Streaming as Collaboration
- Level Streaming is also the primary way for level artists to work together
- Different aspects are separated into different levels
    - Not just different physical zones
#### World Composition
- Specialized streaming system designed for large worlds
- Will not work with old-school level streaming volimes
- But WILL work with Blueprint streaming
    - Pro Tips: You can very easily make a Blueprint that functions just like a Level Streaming volume and does exactly the same thing, only better.


# UE4中的各种特性的优化

## Collision Optimization
关于碰撞体的优化，第一印象都是去掉碰撞体，就像我之前操作的那样，去
- ViewMode -> Visbility Collision

中找到可以看见到的但是没有用到的碰撞体。

然而我偶然看见一个工具，叫CollisionAnalyzer，貌似很有用处。

### Collision Analyzer
位置位于
- Window -> Developer Tools -> Collision Analyzer



- [[UE4] コリジョンのデバッグ―Collision Analyzerを使おう！](http://historia.co.jp/archives/594/)



我突然意识到对于碰撞体的优化，应该是对CPU的处理的优化吧，毕竟碰撞处理的话都是逻辑一样的计算，但是我也不太确定啊。


## Profile Data Visualizer
这篇文章真的是太长了，尽管我有在排版上面下了功夫，但还是显得很乱。在渲染的各个阶段我应该也有写出这个**GPUProfile**的一些部分的内容，但是这个Profile中的一些内容我并不知道该怎么分类，于是我便想把这些内容总结到一起。首先是这个Profile的整体结构。

- ProfileDataVisualizer(FRAME)
    - Scene
        - UpdateSceneObjectData
        - UpdateGlobalDistanceVolume
    - SlateUI
    - FRAME Leaf Evnets


上面是这个视图的结构，因为是VR，有些内容会连续重复出现两次，应该就是里面的View0和View1的区别吧，两个镜片，即摄影机的话有些操作就必须做两次。有些内容我应该有在这篇文章的其他部分有展开就直接省略。

下面就对其中总是耗费很多时间的项目进行一些调查。

### Update Global Distance Volume
参考了这个问题[Update Global Distance Field Volume taking longer than usual](https://answers.unrealengine.com/questions/467114/update-global-distance-field-volume-taking-longer.html)

> The global distance field is updated if any of the features using it are enabled:
>
> - Distance field particle collision
>
> - DistanceToNearestSurface material node
>
> - Shadow casting movable skylight
>
> It also updates if Ray Traced Distance field shadows are enabled, but that's a bug. You can workaround it by forcing global distance fields off with 'r.AOGlobalDistanceField 0'.
>
> I'll assume you are actually using a feature that requires it. The global distance field is a cache around the camera that has to update if the camera moves a lot, or if you have a moving static mesh which has bAffectDistanceFieldLighting enabled. The bigger the static mesh, the more expensive the update will be. Use 'r.AOGlobalDistanceFieldLogModifiedPrimitives 1' to track down which objects it is and disable bAffectDistanceFieldLighting on them.

在视图中仔细观察它的结构：
- UpdateGlobalDistanceFieldVolume
    - CacheType MostlyStaic Clipmap0
    - CacheType MostlyStaic Clipmap1
    - CacheType MostlyStaic Clipmap2
    - CacheType MostlyStaic Clipmap3
    - CacheType Movable Clipmap0
        - GridCull
        - TileCullAndComposite 128x128x128
        - CompositeHeightfelds
    - CacheType Movable Clipmap1
    - CacheType Movable Clipmap2
    - CacheType Movable Clipmap3

在上面的回答的追加内容中：
> So I put a skylight back in, set it from moveable to stationary and it has solved the performance issue. Could this potentially be a but as you told me Shadow Casting Moving Skylights cause it to be updated, however it appears that any moving skylight causes the update to the global distance field. Thanks for the hint that lead to the answer :)

有提到SkyLight的光是Stationary类型的话，会缓解一下这个处理的时间。

于是我试着找会影响这个操作的几个概念都是什么意思，算是拓展。

#### Distance field particle collision
参考了[Using Particle Collision Mode for Distance Fields](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/MeshDistanceFields/HowTo/DFHT_3/index.html)

感觉很有用。

#### RayTraced Distance Field Soft Shadows
参考了[RayTraced Distance Field Soft Shadows](https://docs.unrealengine.com/ja/Engine/Rendering/LightingAndShadows/RayTracedDistanceFieldShadowing/index.html)





# LOD Optimization (未整理)
- [UE4 LODs for Optimization -Beginner-](https://www.slideshare.net/com044/lods-for-optimization-beginner)



# GPU Performance for Game Artists
这是上面的视频里大力推荐的一篇文章。有时间我也得看一下，去理解一下整理一下。

- [GPU Performance for Game Artists](www.fragmentbuffer.com/gpu-performance-for-game-artists/)

# Tech Art Aid videos on Youtube
这个在油管上的视频我一直在看了，在这儿记录一下网址：

- [UE4 Graphics Profiling](https://www.youtube.com/playlist?list=PLF8ktr3i-U4A7vuQ6TXPr3f-bhmy6xM3S)

# Optimization需要之后整理
关于Fortnite的一些优化方法，我觉得应该能学到一些什么：
- [Optimizing UE4 for Fortnite: Battle Royale - Part 1 | GDC 2018 | Unreal Engine](https://www.youtube.com/watch?v=KHWquMYtji0)

几篇感觉很不错的关于优化的知乎文章:
- [[GDC16] Optimizing the Graphics Pipeline with Compute](https://zhuanlan.zhihu.com/p/33881861)

上面文章里的PDF我已经下载下来了。老地方找。

- [[Siggraph15] GPU-Driven Rendering Pipelines](https://zhuanlan.zhihu.com/p/33881505)

- [Performance and Profiling（UE4优化）](https://zhuanlan.zhihu.com/p/36434616)

我感觉下面的知识我至少应该了解，这里贴上一些链接，以便自己找：
- [Precomputed Visibility Volume](https://docs.unrealengine.com/en-us/Engine/Rendering/VisibilityCulling/PrecomputedVisibilityVolume)

- [Proxy Geometry Overview](https://docs.unrealengine.com/en-US/Engine/Proxy-Geometry-Tool/Proxy-Geometry-Overview)

- [PRECOMPUTED VISIBILITY VOLUMES](http://timhobsonue4.snappages.com/culling-precomputed-visibility-volumes)

- [Software Occlusion Culling](https://software.intel.com/en-us/articles/software-occlusion-culling)

这个带有中文的教程我觉得非常好有时间把这个整理成一个系列吧:
- [Unreal Engine 4 优化教程第三部分](https://software.intel.com/zh-cn/articles/unreal-engine-4-optimization-tutorial-part-3)

- [Unreal* Engine 4 VR应用的CPU性能优化和差异化：第一部分](https://software.intel.com/zh-cn/articles/cpu-performance-optimization-and-differentiation-for-unreal-engine-4-vr-application-part1)

- [英特尔软件工程师帮助实施 Unreal Engine* 4.19 优化](https://software.intel.com/zh-cn/articles/intel-software-engineers-assist-with-unreal-engine-419-optimizations)


- [Epic Landscape Production](https://80.lv/articles/epic-landscape-production/)

这篇我应该有记录但是为了以防万一再记录一次。
- [BasePass](https://unrealartoptimization.github.io/book/profiling/passes-base/)

一篇关于VR的优化文章：
- [UE4での描画最適化について](https://framesynthesis.jp/tech/unrealengine/performance/)

比较详细的分解UE4的渲染Pass的内容的文章：
- [UE4の描画パスについて Ver 4.18.1](http://monsho.hatenablog.com/entry/2017/12/16/012502)

UE4的的官方视频关于Profiling和Optimization的，由于是2019年年末的视频，我觉得有必要学习一下
- [Profiling and Optimization in UE4 | Unreal Indie Dev Days 2019 | Unreal Engine](https://www.youtube.com/watch?v=EbXakIuZPFo)




