---
title: UE4-Editor扩展
date: 2019-09-10 14:44:32
permalink: ue4-editor-extension

categories:
- UnrealEngine4

tags:
- UE4
- C++

---
今天这篇文章要写UE4的编辑器扩展内容的，我还没学会，有时间的话一定要整理一下。

<!--more-->
# Overvirew

在用了Unity的各种技巧之后，再回到UE4的时候不禁觉得要是UE4也有这些方便的扩展功能就好了。Unity中的自定义Inspector扩展就很方便，还有DebugMenu等等。

- [Customizing detail panels](https://wiki.unrealengine.com/Customizing_detail_panels)

- [UE4 Blutilityによるお手軽なエディター拡張](http://unrealengine.hatenablog.com/entry/2016/10/05/215937)


# 添加ModuleEd
*ModuleEd*是一种统称，实际上的意思是给工程添加一个Editor扩展功能的Module，比如说你的工程名字叫`TestHello`，那么为了清楚起见就把这个Editor扩展的Module起名为`TestHelloEd`。
这其实就跟UE4的命名差不多，UE4也有一个Module叫`UnrealEd`，功能都是相同的。

至于为什么要添加一个新的Module而不是创建一个Plugin，虽然也有个人喜好，但是我们希望在最后打包工程文件的时候，这个针对UE4Editor的功能内容和代码并不像一起打包的。使用Module只好就可以非常简单的控制打包选项了。但是如果你的这个功能希望在许多工程文件中进行使用，那还是Plugin的方式更适合一些。

在此基础之上，使用的时候就要注意我们可以使用`TestHelloEd`的内容参照`TestHello`的内容，反过来的参照就尽量避免了。

关于如何添加一个Module的方法参考文章：
- [Creating an Editor Module](https://unrealcommunity.wiki/creating-an-editor-module-x64nt5g3)

大致的创建顺序：
1. 仿照Source文件夹下原本的工程Module添加一个新的Module
  ```
  // 工程的感觉大概像这样
  -/Source
  --/TestHello
  ---/..
  ---TestHello.Build.cs
  ---TestHello.h
  ---TestHello.cpp
  --/TestHelloEd
  ---/..
  ---TestHelloEd.Build.cs
  ---TestHelloEd.h
  ---TestHelloEd.cpp
  --TestHello.Target.cs
  --TestHelloEditor.Target.cs
  ```
2. 对新创建的TestHelloEd.Build.cs文件进行配置
  ```
  using UnrealBuildTool;

  public class TestHelloEd : ModuleRules
  {
    public TestHelloEd(ReadOnlyTargetRules Target) : base(Target)
    {
        PublicDependencyModuleNames.AddRange(
            new string[]
            {
                "Core",
                "CoreUObject",
                "Engine",
                "InputCore",
                "TestHello"     // <-添加项目依赖
            });
        PrivateDependencyModuleNames.AddRange(
            new string[]
            {
            });
        PublicIncludePaths.AddRange(
            new string[]
            {
                "TestHelloEd"
            });
        PrivateIncludePaths.AddRange(
            new string[]
            {
                “UnrealEd”,
            });
        PrivateIncludePathModuleNames.AddRange(
            new string[]
            {
            });
        DynamicallyLoadedModuleNames.AddRange(
            new string[]
            {
            });
    }
  }
  ```
3. 对新创建的TestHelloEd.h/TestHelloEd.cpp文件进行配置
  其实目前为止没有什么需求，只要修正一些编译错误就可以了
  ```
  // TestHelloEd.h
  #pragma once
  #include "Engine.h"
  #include "Modules/ModuleInterface.h"
  #include "Modules/ModuleManager.h"
  #include "UnrealEd.h"

  DECLARE_LOG_CATEGORY_EXTERN(TestHelloEd, All, All)

  class FTestHelloEdModule: public IModuleInterface
  {
  public:
      virtual void StartupModule() override;
      virtual void ShutdownModule() override;

  };

  // TestHelloEd.cpp
  #include "TestHelloEd.h"
  #include "Modules/ModuleManager.h"
  #include "Modules/ModuleInterface.h"

  /*
  At the top of the file use the IMPLEMENT_GAME_MODULE macro, the first argument is the name of the module class you created in the header file, the second argument is the name of the module as you declared it in the uproject file.
  */
  IMPLEMENT_GAME_MODULE(FTestHelloEdModule, TestHelloEd);

  DEFINE_LOG_CATEGORY(TestHelloEd)

  #define LOCTEXT_NAMESPACE "TestHelloEd"

  void FTestHelloEdModule::StartupModule()
  {
      UE_LOG(TestHelloEd, Warning, TEXT("TestHelloEd: Log Started"));
  }

  void FTestHelloEdModule::ShutdownModule()
  {
      UE_LOG(TestHelloEd, Warning, TEXT("TestHelloEd: Log Ended"));
  }

  #undef LOCTEXT_NAMESPACE
  ```

4. 配置工程Source下的Target.cs文件
  大概像下面这样，添加我们新增的Module就可以
  ```
  // TestHello.Target.cs
  using UnrealBuildTool;
  using System.Collections.Generic;

  public class TestHelloTarget : TargetRules
  {
      public TestHelloTarget(TargetInfo Target) : base(Target)
      {
          Type = TargetType.Game;
          DefaultBuildSettings = BuildSettingsVersion.V2;
          ExtraModuleNames.Add("TestHello");

          // Editor的扩展所以只在Editor的情况下才编译这个Module
          if (Target.Configuration != UnrealTargetConfiguration.Shipping)
          {
              ExtraModuleNames.AddRange(new string[] {"TestHelloEd"});
          }
      }
  }

  // TestHelloEditor.Target.cs
  using UnrealBuildTool;
  using System.Collections.Generic;

  public class TestHelloEditorTarget : TargetRules
  {
  	public TestHelloEditorTarget(TargetInfo Target) : base(Target)
  	{
  		Type = TargetType.Editor;
  		DefaultBuildSettings = BuildSettingsVersion.V2;

  		ExtraModuleNames.AddRange(new string[] { "TestHello", "TestHelloEd" });
  	}
  }
  ```
5. 最后，添加Module到.uproject文件中
  跟项目工程放在一起的感觉
  ```
  {
    "FileVersion": 3,
    "EngineAssociation": "4.7",
    "Category": "",
    "Description": "",
    "Modules": [
        {
            "Name": "TestHello",
            "Type": "Runtime",
            "LoadingPhase": "Default"
        },
        {
            "Name": "TestHelloEd",
            "Type": "Editor",
            "LoadingPhase":  "PostEngineInit"
        }
    ]
  }
  ```

由于我验证的版本是UE5EA2，如果你去`Tools->Debug->Modules`下面能找到你创建的新的Module，那么就成功了。

# Editor扩展，添加自定义界面
在UE5中添加自定义的窗口方便定义一些Debug功能，想快速入手基本上逃不过EditorUtilityWidget这个东西。

参考文章:
- [Using tool menus for editor extension](https://unrealcommunity.wiki/using-tool-menus-for-editor-extension-ipmtgt9o)
-[Did you know UnrealEngine 4.26 brought in the ability to extend editor menus and toolbars using Blueprints?](https://twitter.com/milkyengineer/status/1379644279480446982)
-[How to Make Tools in UE4](https://lxjk.github.io/2019/10/01/How-to-Make-Tools-in-U-E.html#_setup_editor_module)
-[【UE4】Editor Utility Widgetについてのあれこれ](http://anapurna.co.jp/ue4/641/)
