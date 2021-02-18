---
title: Unity-Shader学习
date: 2019-06-28 19:07:05
permalink:  unity-shader-learning

categories:
- Unity
tags:
- Shader
---
关于Unity的Shader，能说的内容很多，因为我是真的不懂。之前用了好长时间看了关于Unity的书，能看懂一些内容了，但是还远不到立马就能写的程度。这次把自己所见所得写下来。

<!--more-->

# PostProcessing
不知道该起什么名字为好。

## Post Processing Stack v2
这个是一个Unity中方便添加PostProcessing特效的插件。

对这个插件有一个大体上的了解可以参照这篇文章：
- [Unity ルックデヴ講座 Post Processing Stack v2編](https://www.gaprot.jp/gaprot-x-chogiken/xr/unity-lookdev-post-processing-stack-v2)

但是当你想要添加自己的自定义特效的话，就不够用了。
- [Post Processing Stack v2にカスタムエフェクトを追加する方法](http://pond-comfat.hatenablog.com/entry/2019/01/23/031855)

使用这个插件添加自定义Post Processing Shader的话，就要写点代码了，不光是Shader的。


# Renderer Pipeline

## LWRP
- [【 Unity道場 1月 ~LWRPとシェーダー~】軽量レンダーパイプライン、Light Weight Renderer Pipeline…とは](https://www.slideshare.net/UnityTechnologiesJapan/190130unitydojyo-tatsuhiko-yamamura)

## Scriptable Render Pipeline
- [Unity Scriptable Render Pipeline](https://github.com/Unity-Technologies/ScriptableRenderPipeline)


# 使用PostProcessingStack的实例

## 使用PostProcess制作受伤效果
感觉上就像是受到伤害的时候摄像机中心到边缘向红色渐变的效果。

首先是我使用的Unity版本是**Unity2019.2.12f1**。

参考的就除了上面贴出来的链接之外，还有最重要的官方文档了[PostProcessing-Writing Custom Effects](https://github.com/Unity-Technologies/PostProcessing/wiki/Writing-Custom-Effects)

> Custom effects need a minimum of two files: a C# and a HLSL source files (note that HLSL gets cross-compiled to GLSL, Metal and others API by Unity so it doesn't mean it's restricted to DirectX).

### C#
```C#
using System;
using UnityEngine;
using UnityEngine.Rendering.PostProcessing;
 
[Serializable]
[PostProcess(typeof(GrayscaleRenderer), PostProcessEvent.AfterStack, "Custom/Grayscale")]
public sealed class Grayscale : PostProcessEffectSettings
{
    [Range(0f, 1f), Tooltip("Grayscale effect intensity.")]
    public FloatParameter blend = new FloatParameter { value = 0.5f };
}
 
public sealed class GrayscaleRenderer : PostProcessEffectRenderer<Grayscale>
{
    public override void Render(PostProcessRenderContext context)
    {
        var sheet = context.propertySheets.Get(Shader.Find("Hidden/Custom/Grayscale"));
        sheet.properties.SetFloat("_Blend", settings.blend);
        context.command.BlitFullscreenTriangle(context.source, context.destination, sheet, 0);
    }
}
```

> **Important: this code has to be stored in a file named Grayscale.cs. Because of how serialization works in Unity, you have to make sure that the file is named after your settings class name or it won't be serialized properly.**

这里要注意的是脚本名字要一致。

#### Setting
> The settings class holds the data for our effect. These are all the user-facing fields you'll see in the volume inspector.

之前用的时候就会觉得为什么还要做个这个类，原来是为了保存为PostProcess准备的预设数据，所以有`Serializable`属性。

```C#
[Serializable]
[PostProcess(typeof(GrayscaleRenderer), PostProcessEvent.AfterStack, "Custom/Grayscale")]
public sealed class Grayscale : PostProcessEffectSettings
{
    [Range(0f, 1f), Tooltip("Grayscale effect intensity.")]
    public FloatParameter blend = new FloatParameter { value = 0.5f };
}
```

......我觉得官网说的比我清楚多了，干脆贴上英文得了。

看英文，看英文。[Writing Custom Effects](https://github.com/Unity-Technologies/PostProcessing/wiki/Writing-Custom-Effects)

#### Renderer
> Our renderer extends PostProcessEffectRenderer<T>, with T being the settings type to attach to this renderer.

```C#
public sealed class GrayscaleRenderer : PostProcessEffectRenderer<Grayscale>
{
    public override void Render(PostProcessRenderContext context)
    {
        var sheet = context.propertySheets.Get(Shader.Find("Hidden/Custom/Grayscale"));
        sheet.properties.SetFloat("_Blend", settings.blend);
        context.command.BlitFullscreenTriangle(context.source, context.destination, sheet, 0);
    }
}
```
看文档看文档。

### Shader
进入Shader的书写。

```
Shader "Hidden/Custom/Grayscale"
{
    HLSLINCLUDE

        #include "Packages/com.unity.postprocessing/PostProcessing/Shaders/StdLib.hlsl"

        TEXTURE2D_SAMPLER2D(_MainTex, sampler_MainTex);
        float _Blend;

        float4 Frag(VaryingsDefault i) : SV_Target
        {
            float4 color = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.texcoord);
            float luminance = dot(color.rgb, float3(0.2126729, 0.7151522, 0.0721750));
            color.rgb = lerp(color.rgb, luminance.xxx, _Blend.xxx);
            return color;
        }

    ENDHLSL

    SubShader
    {
        Cull Off ZWrite Off ZTest Always

        Pass
        {
            HLSLPROGRAM

                #pragma vertex VertDefault
                #pragma fragment Frag

            ENDHLSL
        }
    }
}
```
看文档，看文档。**关于shader里面的内容还有好多我不懂的**。
### Effect ordering
### Custom editor
### Additional notes

### 特效制作
所以上面的使用说明看完之后，就是实现部分了。不知不觉就把上面的部分写到这一章节离了= =。

因为上面的实现都差不多，只记下Shader部分的：

```
Shader "Hidden/Custom/Damage"
{
    HLSLINCLUDE

#include "Packages/com.unity.postprocessing/PostProcessing/Shaders/StdLib.hlsl"

    TEXTURE2D_SAMPLER2D(_MainTex, sampler_MainTex);
    float _Range;
    float4 _Color;

    float4 Frag(VaryingsDefault i) : SV_Target
    {
        float2 uv = i.texcoord;
        float4 color = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv);
        float radiusFactor = (1 - _Range) * (1 - _Range) / 4;
        float radius = saturate(((uv.x - 0.5) * (uv.x - 0.5) + (uv.y - 0.5) * (uv.y - 0.5)) / radiusFactor);
        return saturate(color * (1 - radius) + _Color * radius);
    }

    ENDHLSL

    SubShader
    {
        Cull Off ZWrite Off ZTest Always

        Pass
        {
            HLSLPROGRAM

                #pragma vertex VertDefault
                #pragma fragment Frag

            ENDHLSL
        }
    }
}

```


这些都是要做的准备工作，最后要把这个PostProcess应用到Camer上。

- Camera上添加PostProcessLayer组件，指定PostProcessEffect的layer
- New一个GameObject，添加PostProcessVolume组件，可以是子物体
- GameObject的Layer需要指定上面提到的layer

这些是有可能会忘的，其他的就很简单了。别忘了New一个新的Profile。参数的话可以参照[Class PostProcessVolume](https://docs.unity3d.com/Packages/com.unity.postprocessing@2.0/api/UnityEngine.Rendering.PostProcessing.PostProcessVolume.html)



## 使用PostPerocess制作向纯色渐变的效果
说起来上面的效果也只是这个效果的拓展，这个效果是我从巨佬那里偷学来的。








# 关于Shader的一些文章

- [Cg Programming/Unity/Translucent Bodies](https://en.wikibooks.org/wiki/Cg_Programming/Unity/Translucent_Bodies)

- [Toon Shader](https://roystan.net/articles/toon-shader.html)

- [【Unity】良い感じに見える（屋内向け）ライティングの設定手順](http://tsubakit1.hateblo.jp/entry/2018/01/04/235520)

- [Unity Post Processing Stackで作る光芒エフェクト](https://ics.media/entry/19728/)

- [【Unity】【シェーダ】ブルームを独自実装する](http://light11.hatenadiary.com/entry/2018/03/15/000022)

原先在知乎上看到的文章，这回有大图看：
- [Unite 2018 | 《崩坏3》：在Unity中实现高品质的卡通渲染（上）](https://connect.unity.com/p/unite-2018-beng-pi-3-zai-unityzhong-shi-xian-gao-pin-zhi-de-qia-tong-xuan-ran-shang)

- [Dark and Stormy](http://vfxmike.blogspot.com/2018/07/dark-and-stormy.html)

一个关于Shader的WIKI，据说关于Unity的Shader讲得很详细。（但是貌似不翻墙看不到。）
- [Cg Programming](https://en.wikibooks.org/wiki/Cg_Programming)
