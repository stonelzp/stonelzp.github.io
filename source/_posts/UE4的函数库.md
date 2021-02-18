---
title: UE4的函数库
date: 2018-08-02 17:47:44
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
在UE4中总会遇见一些不知道是做什么的函数，这篇文章的目的是整理自己遇到的UE4的函数和类，和弄清函数和类的时候遇到的一些问题的解决。

<!--more-->

# UE4中的类

## UTexture2D

### TextureCompressionSettings

### TextureMipGenSettings

### UpdateResource() Functio作用

## UTextureRenderTaget2D
Ureal Engine 4 Documentation:
- [UTextureRenderTarget2D](http://api.unrealengine.com/INT/API/Runtime/Engine/Engine/UTextureRenderTarget2D/index.html)

可以用来存储一个2DTexture数据的类，拥有着许多的成员，文件的位置：
- C:\Program Files\Epic Games\UE_4.19\Engine\Source\Runtime\Engine\Classes\Engine\TextureRenderTarget2D.h
- C:\Program Files\Epic Games\UE_4.19\Engine\Source\Runtime\Engine\Private\TextureRenderTarget2D.cpp

关于如何实例化这个类，我在网上并没有找到类似的实现，但是在UE4的Blueprint中可以找到一个名为`Create Rendr Target 2D`的node函数。试着找了一下这个节点的实现函数
- C:\Program Files\Epic Games\UE_4.19\Engine\Source\Runtime\Engine\Private\KismetRendngLibrary.cpp

其中有如下的实现代码：
```C++
UTextureRenderTarget3D* UKismetRenderingLibrary::CreateRenderTarget2D(UObject* WorldContextObject, int32 Width, int32 Height, ETextureRenderTargetFormat Format)
{
    UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject, EGetWorldErrorMode::LogAndReturnNull);

    if (Width > 0 && Height > 0 && World && FApp::CanEverRender())
    {
        UTextureRenderTarget2D* NewRenderTarget2D = NewObject<UTextureRenderTarget2D>(WorldContextObject);
        check(NewRenderTarget2D);
        NewRenderTarget2D->RenderTargetFormat = Format;
        NewRenderTarget2D->InitAutoFormat(Width, Height); 
        NewRenderTarget2D->UpdateResourceImmediate(true);

        return NewRenderTarget2D; 
    }

    return nullptr;
}
```
或许可以给与一些参照。

## UMaterialInstanceDynamic
Ureal Engine 4 Documentation:
- [UMaterialInstanceDynamic](https://api.unrealengine.com/INT/API/Runtime/Engine/Materials/UMaterialInstanceDynamic/index.html)

这个类的作用按照字面意思来推测是用来创建一个动态的材质实例，在UE4的Blueprint中也有相应的节点函数：`CreateDynamicMaterialInstance`。函数位置
- C:\Program Files\Epic Games\UE_4.19\Engine\Source\Runtime\Engine\Private\KismetMaterialLibrary.cpp

其中的实现代码：
```C++
class UMaterialInstanceDynamic* UKismetMaterialLibrary::CreateDynamicMaterialInstance(UObject* WorldContextObject, class UMaterialInterface* Parent)
{
    UMaterialInstanceDynamic* NewMID = nullptr;

    if (Parent)
    {

        // MIDs need to be created within a persistent object if in the construction script (or blutility) or else they will not be saved.
        // If this MID is created at runtime then put it in the transient package
        UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject, EGetWorldErrorMode::ReturnNull);
        UObject* MIDOuter = (World && (World->bIsRunningConstructionScript  || !World->IsGameWorld()) ? WorldContextObject : nullptr);
        NewMID = UMaterialInstanceDynamic::Create(Parent, MIDOuter);
        if (MIDOuter == nullptr)
        {
            NewMID->SetFlags(RF_Transient);
        }
    }

    return NewMID;
}
```


可以为该材质添加值的函数实现：
```C++
void UKismetMaterialLibrary::SetScalarParameterValue(UObject* WorldContextObject, UMaterialParameterCollection* Collection, FName ParameterName, float ParameterValue)
{
    if (Collection)
    {
        if (UWorld* World = GEngine->GetWorldFromContextObject(WorldContextObject, EGetWorldErrorMode::LogAndReturnNull))
        {
            UMaterialParameterCollectionInstance* Instance = World->GetParameterCollectionInstance(Collection);

            const bool bFoundParameter = Instance->SetScalarParameterValue(ParameterName, ParameterValue);

            if (!bFoundParameter && !Instance->bLoggedMissingParameterWarning)
            {
                FFormatNamedArguments Arguments;
                Arguments.Add(TEXT("ParamName"), FText::FromName(ParameterName));
                FMessageLog("PIE").Warning()
                    ->AddToken(FTextToken::Create(LOCTEXT("SetScalarParamOn", "SetScalarParameterValue called on")))
                    ->AddToken(FUObjectToken::Create(Collection))
                    ->AddToken(FTextToken::Create(FText::Format(LOCTEXT("WithInvalidParam", "with invalid ParameterName '{ParamName}'. This is likely due to a Blueprint error."), Arguments)));
                Instance->bLoggedMissingParameterWarning = true;
            }
        }
    }
}
```

使用方法是在C++中声明一个材质
```C++
UMaterialInstanceDynamic* mMaterial;

//将mMaterial的材质实例通过Blueprint传递过来

mMaterial->SetScalarParamaterValue("TextureWidth",512);

//这样便可以将换递过来的Blueprint中的名为`TextureWidth`(if exist)的Parameter赋值为512了
```

但还是有许多疑问。


# UE4中的函数

## check()

参考链接：
- [When should I use Check()?](https://answers.unrealengine.com/questions/418935/when-should-i-use-check.html?sort=oldest)

## AddInstance()

# UE4中的一些类型

## TextureAddress

在`Texture.h`中看到了这个属性，不太清楚是什么属性，就查了一下。貌似是一种纹理寻址模式。因为有赋值为`T_Clamp`,Clamp让我有些回想起来在Unity中设置UV的时候有repeat跟clamp等等选项来着，需要调查一下。

- [D3D11_TEXTURE_ADDRESS_MODE(纹理寻址模式)](https://blog.csdn.net/gggg_ggg/article/details/40374029)

## TextureFilter

## SRGB是什么
参考链接：
- [sRGB - how to be?](https://forums.unrealengine.com/development-discussion/content-creation/91012-frustration-black-materials-from-my-textures/page2)
- [【图形学】我理解的伽马校正（Gamma Correction）](https://blog.csdn.net/candycat1992/article/details/46228771)
