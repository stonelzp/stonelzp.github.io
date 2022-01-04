---
title: UE4-对工程进行DEBUG的正确姿势
permalink: ue4-debug-your-project
date: 2020-11-09 13:06:08
categories:
- UnrealEngine4
tags:
- UE4

---
这篇文章目的是用来记录我在调查对UE4工程进行DEBUG需要的知识点的一些记录。一开始应该会非常杂乱，等有了更多的精力再好好整理吧。

<!--more-->

# 项目的编译选项
在使用VS对工程进行编译的时候就会看见**SolutionConfigurations**这个下拉菜单有许多选项，不知道选哪一个就会很迷茫。其实这个东西早就应该在学习UE4当时深入理解一下了。

说是深入理解，其实也就是好好的读一下官方文档。
- [Build Configurations Reference](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DevelopmentSetup/BuildConfigurations/)

UE4中的编译选项包含两个关键字，第一个关键字代表了**Build Configuration-State**，第二个关键字代表了**Build Configuration-Target**。
直接用我们最常见的两个编译选项来说就是：
- Debug Editor ： 会将UE4引擎源码和项目里面的DebugSymbols全部包含在内供我们Debug
  - Debug -State，可以对项目或者工程进行Debug的状态
  - Editor -Target，可以在UE4的Editor中打开我们的项目
- DebugGame Editor ： UE4的引擎源码编译会被优化，只有项目(game code)可以被很好的Debug
  - DebugGame -State，可以Debug game code
  - Editor -Target，可以在UE4的Editor中打开我们的项目

UE4默认的是Development Editor的编译方式，怎么说呢，Editor用起来变快了一点？但是开发的话经常Debug就不是那么友好了。单纯的把自己的Code内容编译好再反映到Editor上的话，这个应该是最适合的了，如果说UE4源码本身的编译时间不是那么的长的话，倒可以频繁的切换到这个模式试试呢。


# 保留项目中的设定

## 对 ini 文件的读写
UE4中的基本上所有的设定都是保存在ini文件中的。而我今天就要了解如何在UE4的Editor-ProjectSettings中对ini文件进行读写操作。

首先我们需要的是一个**config object**，然后把保存着设定数据的对象暴露给UE4的Editor方便其与接口对接进行操作，而不是直接对ini文件进行读写。
<details>
    <summary> Creating you settings object (Click to expand) </summary>
    ```c++
    // Copyright 2015 Moritz Wundke. All Rights Reserved.
    // Released under MIT.

    #pragma once

    #include "CustomGameSettings.generated.h"

    /**
     * Setting object used to hold both config settings and editable ones in one place
     * To ensure the settings are saved to the specified config file make sure to add
     * props using the globalconfig or config meta.
     */
    UCLASS(config = Game, defaultconfig)
    class UCustomGameSettings : public UObject
    {
        GENERATED_BODY()

    public:
        UCustomGameSettings(const FObjectInitializer& ObjectInitializer);

        /** Sample bool property */
        UPROPERTY(EditAnywhere, config, Category = Custom)
        bool bSampleBool;

        /** Sample float property that requires a restart */
        UPROPERTY(EditAnywhere, config, Category = Custom, meta = (ConfigRestartRequired = true))
        float SampleFloatRequireRestart;

        /** Sample string list */
        UPROPERTY(config, EditAnywhere, Category = Custom)
        TArray<FString> SampleStringList;

        /** Or add min, max or clamp values to the settings */
        UPROPERTY(config, EditAnywhere, Category = Custom, meta = (UIMin = 1, ClampMin = 1))
        int32 ClampedIntSetting;

        /** We can even use asset references */
        UPROPERTY(config, EditAnywhere, Category = Materials, meta = (AllowedClasses = "MaterialInterface"))
        FStringAssetReference StringMaterialAssetReference;

    };
    ```
</details>

这里重要的有两处：
- `UCLASS(config=Game, defaultconfig)` : 表明这是一个Config对象的UObject，存储的设定(ini)文件在默认的位置。
- `UPROPERTY(config)` : 将属性暴露给Editor的关键字，想要保存在ini文件中的属性一定要带上这个属性修饰符。

之后就是空的构造函数的实现，这里直接省略了代码。


接下来是将我们准备好的**config object**登录到我们想要是有的Module上。这会让我们的configobject适用于任何的Module甚至是Plugins。这需要我们继承**IModuleInterface**接口并实现。由于plugins和工程的Module一般都是会继承这个接口，所以我们不需要刻意去实现，只要利用已经存在的`StartupModule`函数就足够了。
<details>
    <summary> Registering the object with our module (click to expand) </summary>
    ```c++
    // Copyright 2015 Moritz Wundke.All Rights Reserved.
    // Released under MIT.

    #include "CustomSettings.h"

    // Settings
    #include "CustomGameSettings.h"
    #include "ISettingsModule.h"
    #include "ISettingsSection.h"
    #include "ISettingsContainer.h"

    #define LOCTEXT_NAMESPACE "CustomSettings"

    class FCustomSettingsModule : public FDefaultGameModuleImpl
    {
        virtual void StartupModule() override
        {
                RegisterSettings();
        }

        virtual void ShutdownModule() override
        {
                if (UObjectInitialized())
                {
                        UnregisterSettings();
                }
        }

        virtual bool SupportsDynamicReloading() override
        {
                return true;
        }

    private:

        // Callback for when the settings were saved.
        bool HandleSettingsSaved()
        {
                UCustomGameSettings* Settings = GetMutableDefault<UCustomGameSettings>();
                bool ResaveSettings = false;

                // You can put any validation code in here and resave the settings in case an invalid
                // value has been entered

                if (ResaveSettings)
                {
                        Settings->SaveConfig();
                }

                return true;
        }

        void RegisterSettings()
        {
                // Registering some settings is just a matter of exposing the default UObject of
                // your desired class, feel free to add here all those settings you want to expose
                // to your LDs or artists.

                if (ISettingsModule* SettingsModule = FModuleManager::GetModulePtr<ISettingsModule>("Settings"))
                {
                        // Create the new category
                        ISettingsContainerPtr SettingsContainer = SettingsModule->GetContainer("Project");

                        SettingsContainer->DescribeCategory("CustomSettings",
                                LOCTEXT("RuntimeWDCategoryName", "CustomSettings"),
                                LOCTEXT("RuntimeWDCategoryDescription", "Game configuration for the CustomSettings game module"));

                        // Register the settings
                        ISettingsSectionPtr SettingsSection = SettingsModule->RegisterSettings("Project", "CustomSettings", "General",
                                LOCTEXT("RuntimeGeneralSettingsName", "General"),
                                LOCTEXT("RuntimeGeneralSettingsDescription", "Base configuration for our game module"),
                                GetMutableDefault<UCustomGameSettings>()
                                );

                        // Register the save handler to your settings, you might want to use it to
                        // validate those or just act to settings changes.
                        if (SettingsSection.IsValid())
                        {
                                SettingsSection->OnModified().BindRaw(this, &FCustomSettingsModule::HandleSettingsSaved);
                        }
                }
        }

        void UnregisterSettings()
        {
                // Ensure to unregister all of your registered settings here, hot-reload would
                // otherwise yield unexpected results.

                if (ISettingsModule* SettingsModule = FModuleManager::GetModulePtr<ISettingsModule>("Settings"))
                {
                        SettingsModule->UnregisterSettings("Project", "CustomSettings", "General");
                }
        }
    };

    IMPLEMENT_PRIMARY_GAME_MODULE( FCustomSettingsModule, CustomSettings, "CustomSettings" );

    #undef LOCTEXT_NAMESPACE
    ```
</details>

像上面这样我们就在UE4的`Project Settings`就能看到我们新添加的Category：CustomSettings，
![Register the settings](Screenshot 2020-11-14 150422.jpg)

从函数的名称可以猜测，UE4的ProjectSettings的界面分成了Container，Category，Section。

在我参照的文章的最后提到了一个最简单的向ProjectSettings中追加新的Category的方法。就是直接使用`UDeveloperSettings`类，这样默认的就是直接在ProjectSetting中的Game的Section中出现我们新的项目
<details>
    <summary> Using Auto-discovery Settings : Click to expand </summary>
    ```c++
    UCLASS(config=Game, defaultconfig, meta=(DisplayName="My Settings"))
    class LOADINGSCREEN_API UMyDeveloperSettings : public UDeveloperSettings
    {
        GENERATED_UCLASS_BODY()

    public:

        // Add all your properties here as we did before
    };
    ```
</details>

这样我们就可以使用UE4准备的接口进行差不多全部的设定的读写了。但是对UE4是如何管理ini文件的流程完全没有深入的了解。

### Config=ConfigName
关于UCLASS的限定修饰符**config**
- [Class Specifiers](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Reference/Classes/Specifiers/index.html)

> Config=ConfigName
>
> Indicates that this class is allowed to store data in a configuration file (.ini). If there are any class properties declared with the config or globalconfig Specifiers, this Specifier causes those properties to be stored in the named configuration file. This Specifier is propagated to all child classes and cannot be negated, but child classes can change the config file by re-declaring the config Specifier and providing a different ConfigName. Common ConfigName values are "Engine", "Editor", "Input", and "Game".

经过验证，UCLASS中的**config**修饰符决定了ini文件的名称，**defaultconfig**决定了ini文件的路径。关于还有没有其他的关键字我不清楚。应该还有。
- `config=Game, defaultconfig` : 会存在*ProjectName/Config/DefaultGame.ini*设定文件中。

同理将Game替换为上面的"Engine", "Editor", "Input"，就会保存在各种的*DefaultEngine.ini*，*DefaultEditor.ini*，*DefaultInput.ini*文件中。

但是于此同时在*Project/Saved/Config/Windows*文件夹下也会相应的生成*Game.ini*，*Engine.ini*， *Editor.ini*，*Input.ini*文件，这涉及到了一些更复杂的机制。Game，Engine，Editor这些都是在创建项目的时候UE4为我们自动创建的，也是基本上常用的设定文件。但是当我们决定使用我们自己的设定文件的时候，比如说这样写：
- `config=DebugSettings, defaultconfig`

那么我们会发现在*Project/Config*的默认的设定文件文件夹中出现了*DefaultDebugSettings.ini*设定文件，同时在Saved的文件中的设定文件中的出现了*DebugSettings.ini*文件夹。这就涉及到了UE4的ini设定文件的管理问题了，说实话我只是观察到了现象，对于源码并没有进行深入的了解。

当我们不使用*defaultconfig*修饰符的话，我们的*Project/Config*文件夹下就不会生成设定文件，只有Saved文件夹下会生成相应的ini文件。

除了藉由UE4的Editor的SettingsModule对ini文件进行读写之外，还可以使用提供的函数。
- SaveConfig
- LoadConfig

关于自定义读写和对UE4的设定文件的更详细的说明，我发现了一篇文章：
- [UE4 Config配置文件详解（2017.4.1更新）](https://blog.csdn.net/u012999985/article/details/52801264)

这篇文章说的额非常详细，也涉及到了我在简略的读这一部分代码的时候遇到的一些类方法的说明，有时间的话可以把这篇文章的内容摘抄一下嘿嘿。

### 从命令行对设定进行重写
ini文件设定是可以在UE4启动的时候用命令行的形式对其进行重新赋值的，就像
```
-ini:inifile:[section1]:parameter1=value1,[section2]:parameter2=value2,...
```
- inifile : 是**configfile**的名字，应该就是UCLASS修饰符的`config=ConfigName`中的ConfigName，比如说DefaultEngine.ini的情况就是`Engine`。
- section : 是ini文件中的section名字，类似于[/Script/Engine.Engine]
- parameter1=value1 : 重新值的方式

由于我并没有直接验证过，估计要以后使用了才会知道，只记录下我找的资料链接
- [第006回 コマンドラインからConfig設定を上書きする時の注意点](https://www.cc2.co.jp/blog/?p=21872)


### LOCTEXT
硬要说的话是上面的代码中：`LOCTEXT_NAMESPACE`的作用，这涉及到了UE4的LocalizationSystem的问题，我也是完全不知道啊，反正遇到的上面的问题就是在我使用`LOCTEXT`的时候，没有在开始定义那个NAMESPACE的时候就会报错。

简单的贴一下链接：
- [#define LOCTEXT_NAMESPACE "Something"](https://answers.unrealengine.com/questions/849059/view.html?sort=oldest)
- [Text Localization](https://docs.unrealengine.com/en-US/Gameplay/Localization/Formatting/index.html)

涉及到了本地化的内容，感觉超级重要，先记录下来链接之后一定要另外整理：
- [UE4のローカライズ機能紹介 (UE4 Localization Deep Dive)](https://www.slideshare.net/EpicGamesJapan/ue4-ue4-localization-deep-dive-191115517)

### 参考资料
基本上都是参照这一篇的内容。
- [Custom Settings](https://nerivec.github.io/old-ue4-wiki/pages/customsettings.html)
- [Configuration Files](https://docs.unrealengine.com/en-US/Programming/Basics/ConfigurationFiles/index.html)

## 对ini文件的深度读写
上面的内容只是单纯的介绍了使用UE4的Editor来对ini文件进行读写，由于很多读写的细节被封装到了Editor当中，使用自制的Debug工具的时候少不了对ini文件的读写。所以这一部分用来记录我调查的内容。

关于ImGui的内容在另一篇文章。

关于如何做到自己对ini文件读写，首先我无意间看到这篇文章：
- [設定ファイルのヒミツ - Happy My Life](https://blog.cnu.jp/2015/12/19/hidden-parameter/)

这之后应该是对源码的解析，和自己使用ini文件的案例的分析和记录。




# Asset加载的实时观测
最近在调查如何在Realtime情况下观测UE4中Assets的Load和UnLoad，好像调查方向有点走偏了。

## Asset Registry
在我苦苦追寻有没有什么函数回调可以追踪UE4中的Asset的加载的时候找到了这个Module，就一些人所说，
- [Track assets loaded inside the game](https://answers.unrealengine.com/questions/812628/track-assets-loaded-inside-the-game.html)

在`FAssetRegistryModule`和`IAssetRegistry`API源码中可以寻找答案，更准确的说是`OnInMemoryAsset`事件。

结果是否答案在它们身上我也不是很清楚，因为我对UE4本身的资源管理不是了解的缘故，我也不敢随便下结论。首先是**AssetRegistry**这个是什么开始。

> The Asset Registry is an editor subsystem which gathers information about unloaded assets asynchronously as the editor loads. This information is stored in memory so the editor can create lists of assets without loading them. The information is authoritative and is kept up to date automatically as assets are changed in memory or files are changed on disk. The Content Browser is the primary consumer for this system, but it may be used anywhere in editor code.

- [Asset Registry](https://docs.unrealengine.com/en-US/Programming/Assets/Registry/index.html)

首先是官网的解读，AssetRegistry是Editor的子系统，用来收集各种Asset的加载信息。这些情报被加载在内存当中，Editor会根据这些情报管理那些加载和未被加载的Asset。而AssetRegistry的主要消费对象是Editor的**Content Browser**。

其实调查到这里我就觉得有点偏了，因为这个感觉主要面向对象是UE4的Editor，想要自己制作Debugger的话，不知道能不能用啊。

```c++
FAssetRegistryModule& AssetRegistryModule = FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
TArray<FAssetData> AssetData;
const UClass* Class = UStaticMesh::StaticClass();
AssetRegistryModule.Get().GetAssetsByClass(Class, AssetData);
```
很遗憾官网上的这段代码并不能执行，函数的参数类型对不上。


下面这段至少是我在UE4.25版本的时候执行可以顺利运行的代码。
```c++
FAssetRegistryModule& AssetRegistryModule = FModuleManager::LoadModuleChecked<FAssetRegistryModule>("AssetRegistry");
        TArray<FAssetData> AssetDatas;
        /*
        // const FName ClassName =
        AssetRegistryModule.Get().GetAssetsByClass("StaticMesh", AssetDatas);
        if(AssetDatas.Num() > 0)
        {
                for (FAssetData Element : AssetDatas)
                {
                        UE_LOG(LogTemp, Log, TEXT("%s"), *(Element.AssetName.ToString()));
                        // if(Element.IsAssetLoaded())
                }
        }
*/
        AssetRegistryModule.Get().GetAllAssets(AssetDatas);
                if(AssetDatas.Num() > 0)
        {
                for (FAssetData Element : AssetDatas)
                {
                        // UE_LOG(LogTemp, Log, TEXT("%s"), *(Element.AssetName.ToString()));
                        if(Element.IsAssetLoaded())
                        {
                                UE_LOG(LogTemp, Log, TEXT("%s"), *(Element.AssetName.ToString()));
                        }
                }
        }
```

这里注目的是两个函数：
- `GetAllAssets(AssetDatas)`: 直接查看代码的话，这个函数会返回在这个Registry中所有的Assets，但是比较慢，推荐使用另一个有Filter版本的函数。
- `IsAssetLoaded()` : 检查Asset是否被加载。

还有就是判断一个Asset是否已经被加载的判断是已被加载的Asset是可以被`FindObject`到的，但也是我读了代码的一点推测而已。

关于AssetRegistry更加详细的解释，我发现了这样的文章：
- [UE4 AssetRegistry分析](https://zhuanlan.zhihu.com/p/76964514)

再就是官方文档和源码了，由于时间关系，先理解到这里。

关于AssetRegistry更加准确，或说是更好的概括的一个答案：
- [What is Asset Registry used for?](https://answers.unrealengine.com/questions/877592/what-is-asset-registry-used-for.html)

> To understand what asset registry is you need to understand what asset is in technical term.
>
> Asset is a UObject that can be dumped down to file (uasset package) and can be loaded back to the memory from that file and in is mainly used to store game resources, when you load them they are avable in memory as UObject objects like UBlueprint, UTexture2D, USkeletalMesh. USoundWave and so on. Every type of asset you see in content browser has corresponding class and each asset you see in "content Browser" is a UObject that is in memory or can be loaded in memory.
>
> As assets are UObjects normally you would seek them out from reflection system. But because assets exists also on files and don't need to be loaded to memory all the time when they are unused (they would just waste memory space), there need to be object that needs to keep track of them regardless if they are loaded or not. Searching them in reflection system is pointless if they are not loaded first. And that what AsssetRegistry is. it allows you to list out assets, get there regestry entry (FAssetData) and load them up, it also a to more optimized way to seek assets that are loaded already, as well as edit there registry information. And yes "Content Browser" in reality is Asset Registry explorer and it mainly use AssetRegistry to list and edit assets in there. It also provides event delegates which let you track live any changes done to asset registry.
>
> Best way to just look in API refrence to see what function it gives:
>
> https://api.unrealengine.com/INT/API/Runtime/AssetRegistry/IAssetRegistry/index.html
>
> You can always access from anywhere via:
>
>  ```
>  AssetRegistryModule.Get()->...
>  ```
> Assets in registry are stored in paths (which you see as folder structure in content browser), if you want to request specific asset you need to know path for it (you can get it by right clicking asset and clicking Copy Reference). but you can also whole sets of assets using diffrent identificators. for example you can get all assets of specific class (class name as name if the asset class without U prefix:
>
> https://api.unrealengine.com/INT/API/Runtime/AssetRegistry/IAssetRegistry/GetAssetsByClass/index.html
>
> ```
>  TArray<FAssetData> Assets;
>  AssetRegistryModule.Get()->GetAssetsByClass(FName("Texture2D"),Assets,true);
> ```
> And oyu will get registry entires of assets in form of array of FAssetData structures:
>
> https://api.unrealengine.com/INT/API/Runtime/AssetRegistry/FAssetData/index.html
>
> And you can load and get asset UObject it self by calling GetAsset()
>
> LoadObject is a function that loads UObject form the file, it function is useful if you want to load asset (or not) that is outside of asset registry.
>
> Find Object on other hand finds assets that is already loaded in memory.



## Memreport命令抓取Asset的加载情况
这个命令的的得知来自于
- [デバッグとメモリーの最適化](https://www.unrealengine.com/ja/blog/debugging-and-optimizing-memory)

在工程的命令行直接输入`Memrepory`，则会瞬间抓取这一帧的内存加载情况，结果会输出到你的*YourGame/Saved/Profiling/MemReports*文件夹中。

`Memreport -Full`命令则会输出更详细的内容。BaseEngine.ini中的[MemReportCommands] Section下（带有-Full的情况下则是[MemReportFullCommands]）的所有命令都会在Memreport的命令执行后被执行

还有一些其他的命令等待我去发现。

### 抓取数据
应用的启动参数(Standalone模式启动的额外参数)指定`Loadtime`
```
-trace=Loadtime
```

想要获取更多情报的话，CPUEvent或者I/O事件的情报，添加更多参数
```
-trace=Loadtime,File,Cpu -statnamedevents
```

至于其他的命令，遇见在做记录吧。

## Unreal Insight - Asset Loading Insight
最近研究如何使用UnrealInsights吃了一点闭门羹，隐藏的有点深啊，各种使用方法。这次参照的是UnealEngineJP的官方视频：
- [UE4.25 Update - Unreal Insights -](https://www.youtube.com/watch?v=NhehQI98lis&app=desktop)

只参照了AssetLoadingInsights的部分。

**Package(Shipping以外)可以使用。**

## Unreal Insights - Memory Innsights
UE4.26版本增加了Beta特性MemoryInsinsights的功能。

使用方法很普通的UnrealInsights差不多，以Standalone模式启动，添加额外的启动命令行参数
```c++
-trace=memory
```
参考资料：
- [[UE4] Memory Insightsを使用したメモリトラッキング](https://qiita.com/EGJ-Ken_Kuwano/items/83425f450f6da100a9d0)


## Low-Level Memory Tracker
据说是从4.18版本实装的一个可以追踪内存使用情况的工具。

> LLM supports PlayStation 4 and XboxOne. Windows is supported as an experimental feature.

先简单介绍一下用法。
- 打开LLM：Editor的话以Standalone模式，添加`-llm`启动参数
- 使用命令：`Stat LLM`, `Stat LLMFull`等命令就可以查看内存使用情况了。

但是乍看一下输出的内存使用情况就很粗略，还有就是可以输出到CSV文件中查看，我暂时还不清楚使用CSV是否更好一些。

### LLMTAGSETS
这个功能是**Experimenttal**的功能，但是感觉的确是我想用的，便试了一下但是并不顺利。

首先由于还是实验性的功能，据其它的文章所说，我们需要开启这个功能：
```c++
// 在 Engine\Source\Runtime\Core\Public\HAL\LowLevelMemTracker.h 中可以找到
    // using asset tagging requires a significantly higher number of per-thread tags, so make it optional
    // even if this is on, we still need to run with -llmtagsets=assets because of the sheer number of stat ids it makes
    // LLM Assets can be viewed in game using 'Stat LLMAssets'
    #define LLM_ALLOW_ASSETS_TAGS 0
```

源码是0，所以我改成了1，重新编译之后就发生了错误，原因是：`FStatGroup_STATGROUP_LLMAssets: undeclared identifier`。总而言之调查结果就是在相同路径下的`LowLevelMemStats.h`文件中的内容出了问题，因为当中的所有代码都没有被编译进去。

原因是一开始的编译条件中有一个**WITH_ENGINE**的宏定义值为0，为什么啊，我又搜了一下这个是什么
- [Macro defined by UBT in UE4](https://imzlp.me/posts/5214/)

也就在这种文章中有所提及，貌似是涉及到UE4的UBT的配置问题，但是我对UBT也不是特别了解。




### 参考资料
- [Low-Level Memory Tracracker](https://docs.unrealengine.com/en-US/Programming/Development/Tools/LowLevelMemoryTracker/index.html)
- [UE4 Low Level Memory Tracker 使用](https://zhuanlan.zhihu.com/p/78005333)
- [[UE4] LLM (Low Level Memory Tracker)を使用したメモリトラッキング](https://qiita.com/donbutsu17/items/dd410cd6ee53b0b348ca)


# 一些优化的小技巧
在对UE4的逐渐学习理解过程中，会遇见一些常见的优化小技巧。

## Uin8变量的使用
在UE4的源码中经常会见到一些看起来比较违和的变量声明：
```c++
uint8 bCanBeDamaged : 1;
```
之前我什么都不懂，以为只是一种布尔型变量的初始化，上面的就是初始化为true的意思，实际上就是我真的什么都不懂。

按照别人的解答，这样的写法，首先是UE4默认的布尔类型变量的声明是以小写字母`b`开始的，而数字`1`的含义则是只使用1bit位。这是节省内存的表现，相比于布尔类型的数据。这是它们的唯一的不同。

为了方便理解原文，贴上原答案：
> All booleans in Unreal are prefixed with b. If you see a uint with the b prefix it is declared as a bit field like `uint8 bCanBeDamaged:1` The 1 in the end indicates that it only uses 1 bit. `bCanBeDamaged == true` is perfectly fine to do even though it is a uint8. The reason a bit field is used instead of a bool is to save memory and the only difference between a uint8 bit field and a bool is the way it is declared.

参考链接：
- [Uint8 Confusion .. Why not bool?](https://answers.unrealengine.com/questions/888947/uint8-confusion-why-not-bool.html)

## WITH_EDITOR and WITH_EDITORONLY_DATA
~在对项目工程进行DEBUG的时候免不了添加一些DEBUG函数，但是这些个DEBUG函数也只是会在UE4的Editor上使用，这种依存于UE4Editor的DEBUG代码，就可以利用标题的宏，当最终进行shipping的编译时，这些代码都不会被编译(应该)。~

刨除上面我自己的肤浅的理解，关于这两个宏的使用在网上调查了一下也没有完全明白这两个宏使用的区别，使用条件还是不清楚，还是不要使用的好。

```
// CoreMiscDefines.h
#ifndef WITH_EDITORONLY_DATA
	#if !PLATFORM_CAN_SUPPORT_EDITORONLY_DATA || UE_SERVER || PLATFORM_IOS
		#define WITH_EDITORONLY_DATA	0
	#else
		#define WITH_EDITORONLY_DATA	1
	#endif
#endif

// PCH.Engin.h
#define WITH_EDITOR 1
```

还是老老实实的用下面的这个包含DEBUG的代码吧。

```
#if UE_BUILD_DEBUG || UE_BUILD_DEVELOPMENT
#endif
```
