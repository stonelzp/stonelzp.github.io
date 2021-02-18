---
title: UE4光照学习调查笔记
date: 2019-03-15 17:56:21

categories:
- UnrealEngine4

tags:
- UE4
---
提高VR帧数的工作，需要对各个流程有所了解。光照，渲染等等，这篇文章就是用来记录我在试图理解光照流程中遇到的知识点。

<!--more-->

# 光照
在UE4的官方文档中有很多说明，我只记录一些我可能会忘记的。

我在别的优化文章里面多少有提到关于光照的内容。

## Light Mobility
- [Light Mobility](https://docs.unrealengine.com/en-us/Engine/Rendering/LightingAndShadows/LightMobility)
### Stationary lights
这种光源是位置不变但是它的brightness和color等想要变化的那种光源。

> However,it should be noted that runtime changes to brightness only affect the direct lighting.Indirect (bounced) lighting, since it is pre-calculated by lightmass, will not change.

这里有三个需要我理解的地方。
- Direct Lighting ( √ )
- Indirect (bounced) Light
- Lightmass

所有来自于Stationary Light的indirect lighting (应该是间接光照吧) 和间接阴影(indirect shadowing)会存储在Lightmap（光照贴图）中。直接光照阴影会存储在阴影贴图中(Shadowmap)。

> All of the indirect lighting and shadowing from Stationary Lights is stored within the lightmap.Direct shadows are stored within the Shadowmap.These lights make use of Distance Field Shadows, meaning that their shadows can remain crisp even fairly low Lightmap Resolution on lit objects.

当使用了低质量lightmap解决方案的时候，因为Stationary lights直射光产生的直接阴影是存储在Shadowmap中的，所以仍然还可以产生比较清晰的阴影。

这里需要深入理解的是：
- Shadowmap
- Lightmap

#### Direct Lighting
> The direct lighting of stationary lights is rendered dynamically using deferred shading.This allows the brightness and color tobe changeable at runtime, along with a light function or IES profile.

应该就是直接光源，stationary light的直接光照是使用延迟渲染。

这里我应该理解的是
- Deferred shading(该回忆了)
- IES profile

#### Direct Shadowing
> Realtime shadowing of light sources has a major performance cost. A fully dynamic light with shadows will often cost twenty times(20x) as much to render than a dynamic light without shadows. For this reason, stationary light have the ability to have static shadowing on static object, but with some limitations.

关于阴影，光照的实时阴影是影响性能的一个主要参数。stationary light能产生静态阴影，在静态物体上。但是有限制。

##### Static Shadowing

**On Opaque**

> Lightmass generate distance field shadow maps for stationary lights on static objects during the lighting rebuild. Distance field shadow maps provide very accurate shadow transition even at low resolution, and with very little runtime cost. Like lightmaps, distance field shadow maps require uniquely unwrapped UVs on all *StaticMeshes* using static lighting.

当stationary light对不透明静态物体生成阴影的时候，在lighting rebuild阶段生成distance field shadow maps 。

> Lighting must be built to display distance field shadows, otherwise whole scene dynamic shadows will be used for previewing.

这里我并没有找到有能够显示**distance field shadow**的编译选项。

> Only 4 or fewer overlaping stationary lights can have static shadowing, because the lights must be assigned to different channels of a shadowmap texture. This is a graph coloring problem, so there are often fewer than 4 overlapping allowed due to topology. 

> Shadowing cannot affect the overlap test, so **the sunlight typically requires a channel from the entire level it is in, even the underground areas.**
这一句话应该是说阴影不会影响overlap测试，所以即使是在地下世界里的场景，有sunlight的情况下都要给sunlight预留一个channel。

就像上面所说的，尽量避免stationary lighting的overlapping。在**StationaryLightOverlap**视图中，可以查看stationary lighting的重合程度。如果光源的图标变成了红色的X就表明它不能容纳更多的channel了。

**On Translucency**

> Translucency also receives shadowing very cheaply with Stationary lights - Lightmass precomputes a shadow depth map from static geometry which is applied to transluency at runtime. This form of shadowing is fairly coarse only captures shadowing on the scale of metres.

不透明的物体也可以接收来自Stationary Light的光照阴影，代价也不是很高。但是这种阴影很粗糙。
- Shadow depth map

> The resolution of the static shadow depth map is controlled by StaticShadowDepthMapTransitionSampleDistanceX and StaticShadowDepthMapTransitionSampleDistanceY in BaseLightmass.ini, with a default setting of 100 meaning one texel every meter.

##### Dynamic Shadowing
> Dynamic objects(like StaticMeshComponents and SkeletalMeshComponement with Mobility set to Movable) must integrate into the static shadowing of the world from distance field shadowmaps. This is accomplished with *Per Object* shadows. Each movable object creates two dynamic shadows from a stationary light : a shadow to handle the static world casting onto the object, and a shadow to handle the object casting onto the world.

动态物体（movable）必须整合到来自**distance field shadowmap**的世界静态阴影(**The static shadowing of the world**) 中去,这个经由**Per Object shadows**来完成(每一个物体的阴影计算？)。每一个标记为movable的物体会创建两个来自StationaryLight的动态阴影，一个用来处理静态世界在该物体上的阴影，另外一个用来处理该物体在世界里面的阴影。

这意味着当动态物体的数量很多的时候，使用动态光是更好的选择。

> *Per Object* shadows used by movable components apply a shadowmap to the object's bounds, therefore the bounds must be accurate. For Skeletal meshes this means they should have a physics assets. For partial systems, any fixed bounding box must be large enough to contain all particles.

**Direction light dynamic shadowing**

**Directiong Stationary Lights**比较特殊，因为它通过*Cascaded Shadow Maps*(级联阴影贴图)和*static shadowing*支持整个场景的阴影的生成。当场景中存在许多*animating foliage*会很有用；当在player的周围存在许多动态的阴影，但是并不想花费太大的代价去使用许多cascades覆盖很大的视野范围。动态阴影将会随着距离的提升一点点渐隐到静态阴影中，这样的过度通常难以区分。

可以改变 *DirectionalLightStationary* 的**Dynamic Shadow Distance StationaryLight** 的范围来控制过渡的距离范围。

这里我不理解的概念：
- Cascaded Shadow Maps
- Animating foliage

> Movable components will still create PerObjects shadows even when using Cascaded Shadow Maps on a directional light.This behavior is useful with small Dynamic Shadow Distance, but incurs unnecessary cost with larger distance. To disable PerObject shadows and save performance, disable *Use Inset Shadows For Movable Objects* on the light.

#### Indirect Lighting
和静态光照一样，StationaryLight将间接光照存储在光照贴图内（lightmap）。间接光照不能像直接光照那样运行中改变光照的brightness和color。这意味着即使在**light build**中将**Visible**取消，间接光照还是会被存储在光照贴图当中。

**IndirectLightingIntensity**的光照属性可以被用来控制大小和开关，在光照的编译阶段。

> However there is a post process volumn setting called **IndirectLightingIntensity** which lets you scale the contribution of the lightmap for all light, which can be changed at runtime from a blueprint.

上面的很好理解就不解释了。


#### Use Area Shadows for Stationary Lights
**Directional Light**可以打开这个属性，但是要保证这个直射光照的**Mobility**是**Stationary**。

> When the *Use Area Shadows for Stationary Lights* option is enabled,the Stationary Light will use area shadows for the precomputed shadow maps. Area shadows are shadows that get softer the further they are from caster.

> Note that Area Shadows will only work on Stationary Lights and you might have to increase some objects lightmap resolution to get the same shadow quality and sharpness.

总的来说就是这个属性会让阴影变得比较柔软，更贴近现实一些。

### Movable Lights
动态光照不会把数据烘焙到光照贴图里，并且没有任何间接关照（currently）。

#### 阴影
阴影是动态光照里最昂贵的花销之一，光照内的mesh的图元数量和三角形数量是影响性能的指标。

> Movable lights setup to cast shadows use whole scene dynamic shadows, which have a significant performance cost.The performance cost comes primarily from the number meshes affected by the light,and the triangle count of those meshes.

#### Shadow Map Caching

1. 选中所有产生动态阴影的动态光
2. 确保**Mobility**是**Movable**，并且**Cast Shadows**属性选中
3. **Backtick( ` )** 打开console输入 *<span style="color:green">Stat Shadowrendering</span>* 查看实时动态阴影的花销
4. 再次打开console，键入 *<span style="color:green">Sr.Shadow.CacheWholeSceneShadow 0</span>*关掉**dynamic shadow caching**
5. 注意**CallCount** 和 **InclusiveAug**
6. 再打开console，键入 *<span style="color:green">r.Shadow.CacheWholeSceneShadow 1</span>* 打开**dynamic shadow caching**

- you can control the maximum amount memory used by the Shadow Map Cache using **<span style="color:green">r.Shadow.WholeScenceShadowCacheMb</span>**

##### Limitations
关于使用限制：
- By default,caching can only happen when an object meets the following requirements:
    - Primitives have their mobility **Mobility** set **Static** or **Stationary**.
    - Materials used in the level do not use **World Position Offset**.
    - Light need to be either a **Point** or **Spot** light, have its **Mobility** set to **Movable**, and have **Shadow Casting** enabled.
    - Lights have to remain at one location.
    - Material that use animated **Tessellation** or **Pixel Depth Offset** can cause artifacts as their shadow depths are cached.

最后一条不太理解，需要继续调查：
- Tessellation
- Pixel Depth Offset

## Actor Mobility
这里插一条概念，关于Actor的Mobility属性。

- [Actor Mobility](https://docs.unrealengine.com/en-us/Engine/Actors/Mobility)

文档上有写，所以我就少写一点。

当我知道了StationaryLight的属性的影响的时候，主要会让我产生混乱的是当StaticMesh的mobility属性是Stationary的话怎么办

> For Static Mesh Actors,this means that they can be changed but not moved.They do not contribute to pre-calculated lightmaps using Lightmass and are lit like Movable Actor when lit by a Static or Stationary Light.However,when lit by a Movable Light, they will use a Cached Shadow Map to reuse for the next frame when the lighting is not moving.

大体上就是StationaryLight下的StationaryMobility的StaticMeshActor,没什么太大的意义。



## Lightmass
- [Lightmass Global Illumination](https://docs.unrealengine.com/en-us/Engine/Rendering/LightingAndShadows/Lightmass)

Lightmass生成像区域阴影，漫反射交互等等的复杂光照交互的光照贴图。通常被用来预计算一部分的静态光照和StationaryLight的光照。

## Feature for Static and Stationary lights

### Diffuse Interreflection

**Diffuse interreflection** is by far the most visually important global illumination lighting effect.Light bounces by default with Lightmass,and the BaseColor term of your material controls how much light(and what color)bounces in all directions. This effect is sometimes called Color Bleeding.Diffse Interreflection is incoming light reflecting equally in all directions,which means  that it is not affected by the viewing direction or position.

### Indirect Lighting Cache
Lightmass为静态物体生成间接光照的光照贴图，动态物体也同样需要，解决方案是使用**Indirect Lighting Cache**。但是从UE4 4.18版本之后默认实现就被**Volumetric Lightmap** 取代了。

### Volumetic Lightmaps
- [Volumetric Lightmaps](https://docs.unrealengine.com/en-US/Engine/Rendering/LightingAndShadows/VolumetricLightmaps)

在光照的编译阶段所有的点预计算的光照会被存储在Volumetric Lightmap中，之后在实时运算中被用来dynamic objects的间接光照插值运算。

工作方式：
- 在光照的编译阶段，Lightmass会在整个场景中放置**lighting samples** 并且计算它们的间接光照。
- 当开始渲染动态物体的时候，**Volumetric Lightmap**会以插值的方式为每一个被渲染的像素提供预先计算好的间接光照。
- if no built lighting is available(meaning the object is new or has moved too much), lighting is interpolated to each pixel from the Vloumetric Lightmap for **Static** objects until lighting is rebuild.

上面的那句话我不太理解就直接抄了英文。

#### Enabling Volumetric Lightmap Visualization
在view视图中找到：

**Show > Visualize > Volumetric Lightmap**

可以查看，貌似必须是编译好光照之后才能看的见。

有着越远离几何体密度越低的特点。

这里提到了**Lightmass Important Volume**这个蛮重要的东西。

## Stationary Light的阴影
- [Stationary Light の影について](http://darakemonodarake.hatenablog.jp/entry/2015/12/16/UE4/Stationary)

参照上面的文章进行的一些总结。

### Static Mesh Shadow
-  **Stationary Light的阴影贴图是由各自的StaticMesh保持的**
    - 这个可以在Stat的NumSM/TextureSM条目里查看
    - ULightComponent::ReassignStationaryLightChannels

- **Stationary Light的ShadowMap是经由G-Buffer之后的光照计算阶段生成的**
    - 再稍微具体一点来说就是：StaticMesh在LightBuilt之后，有了StationaryLight阴影贴图，然后在BasePass阶段，把这个ShadowMaps的值拷贝到G-Buffer中的一个PrecomputedShadowFactors的float4类型的值中去，在写进G-Buffer中之后，在光照计算阶段，光照通过读取自己持有的Index对应的PrecomputedShadowFactor的值来计算像素的阴影。

### Movable Object
动态物体在StationaryLight的阴影又是什么样子的？

先是以点光源为例，DirectionalLight使用的是CascadeShadow，有一些不同。

- StationaryLight为每一个MovableMesh计算阴影贴图，并把ShadowMap的数据存储到一张巨大的阴影贴图中。
- 之后逐个对其所有的Mesh的渲染进行阴影的计算。（实际的渲染阶段的阴影计算？）


总结为：

**Movable Mesh的Stationary Light阴影的计算Cost = 光照范围内的Movable Mesh数 * 每一个物体阴影的生成和渲染cost（PerObject？之前貌似在哪里见过）**

Stationary Light的阴影，每一个Movable Mesh都会被计算。

像是废话。

Stationary Light的阴影，对于Movable Mesh而言，它会一个一个的进行阴影贴图的计算，然后一个一个的进行渲染。

用实际的数据来展现就是：

在<span style="color:green">ProfileGPU</span>命令打开的Profiler中的这一项：
- **ShadowDepthFromOpaqueProjected**

可以看到Movable Mesh在Stationary Light下的每一个阴影贴图计算的时间，一般都是0.01我感觉。

终于知道这个是什么意思了。

还有这一项：

- **ShadowProjectionOnOpaque**

可以看到对于每一个Movable Mesh，Stationary Light的实际的阴影计算所消耗的时间。

（我觉得我应该去复习一下渲染流程了。）

这就会在StationaryLight的光照范围内加入大量的动态物体会使得处理时间变长的原因。

**而另一方面，Movable Light对于Movable Mesh阴影的处理不是以每一个动态物体为单位而是以光照为单位，对这个光照范围内的所有物体打包计算。**

因此当Movable物体很多的时候，动态光照的方案要优于StationaryLight方案。

### Directional Light的阴影
上面的都是对点光源和聚光灯的光源来说的不包含直射光。

Directional Light基本上是用Cascade Shadow Map进行的。级联阴影贴图。

在ProfileGPU中的具体项目是：
- ShadowDepthFromOpaqueProjected
    - WholeScene split1
    - WholeScene split2
    - WholeScene split3
    - ..(maybe)

有一个让处在Cascade Shadow以外的Object以点光源一样进行阴影计算的方式：
- Cascade Shadow Map > Inset Shadow For Movable Objects

这意思是CascadeShadow是有范围的？

通过调整 *Dynamic Shadow Distance Stationary Light* 的距离可以让物体不使用Cascade Shadow。

总结：

- **Static Mesh：Stationary Light事先计算阴影，由各静态mesh保持，当开始渲染的时候直接读取值处理负荷很小。**

- **Movable Mesh：Stationary Light对每一个动态Mesh计算阴影，当动态物体很多的时候，优先选择Movable Light。**











## Lighting needs to be rebuilt
光照烘焙，一般的情况下这个提示是要烘焙光照，但是我遇到了烘焙光照之后还有这个提示的问题。是在我删除掉场景里的所有FoliageStaticMesh之后。

### DumpUnbuildLightInteractions
在Command Line中输入上述的命令，可以在OutputMessage里面输出需要烘焙的对象信息。








# Light Map
光照贴图这个部分能说的东西实在时候太多了，要理解的东西也非常多，**LightMap**具体是个什么东西，用来存储什么数据？搜LightMap的时候又会出现其他的概念，**LightMass**又是什么？

LightMass在上面也有提到一点。

## Unwrapping UVs for Lightmaps
当试着搜LightMap的时候，就会出现这篇官方文档
-[Unwrapping UVs for Lightmaps](https://docs.unrealengine.com/en-US/Engine/Content/Types/StaticMeshes/LightmapUnwrapping)


## Lightmass
这个有我下载的PDF，有时间总结一下。
- [Lightmassの仕組み ~Lightmap編~ (Epic Games Japan- 篠山範明)]()

这篇文章提到的：

**LightMap和ShadowMap是以Actor为单位各自保持的。**

可以在Statistics中的StaticMeshLighting info中看到Actor的各自持有的**TextureLM**和**TextureSM**。






# LightingTroubleshootingGuide
- [LightTroubleshootingGuide](https://wiki.unrealengine.com/LightingTroubleshootingGuide#Lightmap_Resolution_.2F_Shadow_Quality)


