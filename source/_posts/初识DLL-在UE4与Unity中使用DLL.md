---
title: 初识DLL-在UE4与Unity中使用DLL
date: 2019-01-28 16:02:05
permalink: use-dll-in-ue4-unity

categories:
- C++
tags:
- UE4
- Unity

description: 关于DLL的制作与使用，和一些注意事项。与之相关的一些知识点。

---

记录自己逐步认识和掌握DLL的过程。包含了创建DLL，在UE4中使用DLL，在Unity中使用DLL。

<!--more-->

# 新建DLL工程
先新建一个简单的DLL工程，然后在里面添加内容，方法可以参照：
- [Linking Dlls](https://wiki.unrealengine.com/Linking_Dlls)

上面的链接已经失效了，UE4的Wiki一升级就完蛋。还好能搜到文章的映像，好心人还是有的。
- [Linking Dlls](https://nerivec.github.io/old-ue4-wiki/pages/linking-dlls.html)

我觉得我还是好好的记录一下为好...

使用VS新建一个动态链接库的解决方案，就是工程
![创建动态链接库](CreateDLL_Setp1.jpg)

至于它默认的建立的几个文件说实话我也不是特别了解，反正只是按照教程我自己新建了一个新的文件`LogFuncExport`的 .h .cpp文件。
![新建一个我们用来导出函数的类文件](CreateDLL_Setp2.jpg)

然后我们复制粘贴测试代码，然后用`Release`和`X64`模式编译项目，在我们的项目文件夹下的**x64/Release**下找到我们用来测试用的dll文件。

至于测试代码就顺带贴一下，顺带一提我用的是VS2019，UE4的版本是4.25。头文件别忘了根据场景替换。
<details>
  <summary>点击展开测试用的代码</summary>
  ```c++
  // .h文件
  #pragma once  

#define DLL_EXPORT __declspec(dllexport)  //shortens __declspec(dllexport) to DLL_EXPORT

#ifdef __cplusplus    //if C++ is used convert it to C to prevent C++'s name mangling of method names
extern "C"
{
#endif

  bool DLL_EXPORT getInvertedBool(bool boolState);
  int DLL_EXPORT getIntPlusPlus(int lastInt);
  float DLL_EXPORT getCircleArea(float radius);
  char DLL_EXPORT *getCharArray(char* parameterText);
  float DLL_EXPORT *getVector4( float x, float y, float z, float w);

#ifdef __cplusplus
}
#endif
  ```

  ```c++
  // .cpp文件
  #pragma once

#include "string.h"
#include "CreateAndLinkDLLFile.h"

//Exported method that invertes a given boolean.
bool getInvertedBool(bool boolState)
{
  return bool(!boolState);
}

//Exported method that iterates a given int value.
int getIntPlusPlus(int lastInt)
{
  return int(++lastInt);
}

//Exported method that calculates the are of a circle by a given radius.
float getCircleArea(float radius)
{
  return float(3.1416f * (radius * radius));
}

//Exported method that adds a parameter text to an additional text and returns them combined.
char *getCharArray(char* parameterText)
{
  const char* additionalText = " world!";
  
  if (strlen(parameterText) + strlen(additionalText) + 1 > 256)
  {
    return (char*)"Error: Maximum size of the char array is 256 chars.";
  }

  char combinedText[256] = "";
  
  strcpy_s( combinedText, 256, parameterText);
  strcat_s( combinedText, 256, additionalText);
  
  return ( char* )combinedText;
}

//Exported method that adds a vector4 to a given vector4 and returns the sum.
float *getVector4( float x, float y, float z, float w )
{
  float* modifiedVector4 = new float[4];
  
  modifiedVector4[0] = x + 1.0F;
  modifiedVector4[1] = y + 2.0F;
  modifiedVector4[2] = z + 3.0F;
  modifiedVector4[3] = w + 4.0F;

  return ( float* )modifiedVector4;
}
  ```

</details>


# 导出函数
添加了具体的库的功能之后，需要将函数导出供外部使用。
```C++
#pragma once

#define DLL_EXPORT __declspec(dllexport)

#ifdef __cplusplus
extern "C"
{
#endif
    int DLL_EXPORT ExportFuncName(){ return 0;}
    // ......

#ifdef __cplusplus
}
#endif
```
像这样，上述的函数就被导出，能够被外部调用了。

## 导出函数的时候遇到的问题
当我在UE4中使用inline这个关键字的时候，给我报错了。

我在DLL导出函数的时候，导出的函数前面加了**inline**关键字，所以把导出函数的定义跟声明都写在了头文件里。发生了什么呢？

- 在UE4中我获取到了**DLL Plugin**中的dll库，得到了DLL的Handle，但是准备使用这个Handel取出里面的函数的时候，取出来的是空。函数名什么的都是正确的情况下。

发生上述的情况下我试着去掉DLL中的导出函数的`inline`声明，声明和定义分开在头文件和cpp文件，就解决了。

回归第一句的事实，貌似UE4C++中对于inline关键字是不支持的。哪怕是动态库中导出来的函数。

# UE4Editor中调用DLL
在文章最开始的地方提到的链接已经说明的很清楚了。

要点在于:
- 把制作好的DLL放到特定的文件夹中，一般是Plugin文件夹里面建立一个Plugin。
- 创建一个类继承`BlueprintFunctionLibrary`这样蓝图也能使用
- 声明的函数要是静态的（static）
- 调用DLL中的函数的步骤是:
    - 定义一个函数指针用来接收DLL中导出的函数(Use `typedef` to declare a method to store the DLL method)
    - 声明一个Handle来保持与DLL的连接（`void* v_dllHandle`)
    - 都不为空的时候调用导出来的函数。

```C++
typedef int(*_getFunc)(int a, int b);
_getFunc m_getFuncFromDll;

void * v_dllHandle;

void ULOGBlueprintFunctionLibrary::DLLFunc(int a, int b）
{
    FString dllfilepath = FPaths::Combine(*FPaths::ProjectPluginsDir(), TEXT("DLLLibrary"), TEXT("DLLName.dll"));
    if(FPaths::FileExists(dllfilepath))
    {
        v_dllHandle = FPlatformProcess::GetDllHandle(*dllfilepath);

        if(v_dllHandle != NULL)
        {
            FString procName = "FunctionName";
            m_getFuncFromDll = (_getFunc)FPlatformProcess::GetDllExport(v_dllHandle, *procName);

            if(m_getFuncFromDll != NULL)
            {
                m_getFuncFromDll(a,b);
            }
        }
    }
}
```

使用的方法就像上述。

# DEBUG DLL
关于DLL的DEBUG的问题，直接在工程里面DEBUG的方法我没有头绪。目前的方法是：

选中Build的模式为`Release，x64 `然后编译。在工程文件夹中找到编译完毕的dll文件导入到调用的工程里面进行调用测试。

# UE4中打包第三方库
你看上面使用动态链接库的方法多么简单啊，只要找到*dll*就可以对这个库进行使用了。按照上面的方法，无论在UE4Editor上运行几次，你都会满意的没有话说。

是的，在UE4的Editor上运行的话。

当你觉得差不多了咱们打包一下游戏吧。

*File -> Package Project -> Windows ->Windows(64-bit)*

打完包了都不知道发什么了什么，知道发现dll的内容根本没法调用。是的，上面的一顿操作，当你打包整个工程之后会发现，准备好的dll并没有被一起打包进工程里。

原因就在于UE4并没有提供自动打包第三方库的功能，需要手动打包。敢情上面的dll的一顿操作只是针对Editor模式好用，当你准备打包应用程序的时候就不行了。算了，不生气。

## UE4的编译工具(Build Tools)
- [Build Tools](https://docs.unrealengine.com/en-US/Programming/BuildTools/index.html)

为什么UE4不能把第三方库内容自动打包呢？这就要从UE4引擎的编译模式来说起了。

从官网的内容来看的话，BuildTools分为三个Topics

### 1.UnrealBuildTool
> UnrealBuildTool (UBT) is a custom tool that manages the process of building Unreal Engine 4 (UE4) source code across a variety of build configurations. Read  BuildConfiguration.cs to explore various user-configurable build options.

这个应该非常重要的概念了，也经常见到这个工具，但是具体的内容日后再挖。

- [Understanding Unreal Build Tool](https://ericlemes.com/2018/11/23/understanding-unreal-build-tool/)
#### Topics1： Targets
- [Targets](https://docs.unrealengine.com/en-US/Programming/BuildTools/UnrealBuildTool/TargetFiles/index.html)

UnrealBuildTool(以后简称UBT)


#### Topics2: Modules
- [Modules](https://docs.unrealengine.com/en-US/Programming/BuildTools/UnrealBuildTool/ModuleFiles/index.html)
- [UE4 モジュールについて](http://historia.co.jp/archives/3097/)
- [UE4 Moduleについて](http://papersloth.hatenablog.com/entry/2018/03/05/221735)
- [一つの UE4 プロジェクトで複数のゲームモジュールを扱う](https://qiita.com/go_astrayer/items/5d001a9cde182488f9f3)
- [【UE4】モジュール追加](https://www.romsuke.com/entry/2019/01/10/212704)
- [ModuleRules(XXX.build.csファイル)について](https://qiita.com/takayashiki2/items/db995c558024c3db8223)




#### Topics3: Build Configuration

#### Topics4: IWYU

#### Topics5: Project Files for IDEs

#### Topics6: Versioning of Binaries

### 2.UnrealHeadTool

### 3.AutomationTool

## 打包第三方库
上面的是对于UE4编译系统的理解，而真正需要做的是具体是什么呢?

针对之前的内容我大致的总结了一下，慢慢补充:

**将DLL（第三方库）和项目一起打包**
1. `.uplugin`中的`"Modules"`模块的`"Type"`从`"Developer"`更改为`"Runtime"`
    1. 就我自己的观察结果来看，这个属性不更改的话，打包之后（Package化）连Plugin的尸体都不会给留，在打包后的工程中找不到想要的DLL
2. 指定平台编译（不是必要条件但是要是用错了平台的话肯定是要吃苦头的）
    ```
    // uplugin文件
    "Modules": [
    {
    "Name": "USBCamDirectShow",
    "Type": "Runtime",
    "LoadingPhase": "Default",
    "WhitelistPlatforms": ["Win64", "Win32"]　←これを追加
    }
    ]
    ```
3. 工程文件结构补充(Module分布),Win64平台
    ```
    Project
       |- Binaries
            |- ...
       |- Plugins
            |- PluginNameA
                   |- Binaries
                       |- ThirdParty
                           |- PluginNameALibrary
                       |- Win64
                   |- Source
                      |- PluginNameA
                            |- Private
                            |- Public
                            |- PluginNameA.Build.cs
                      |- ThirdParty
                           |- PluginNameALibrary
                                |- PluginNameALibrary.Build.cs
                                |- PluginNameALibrary.tps
                                |- LibrarySource
                   |- PluginNameA.uplugin
                   |- ...
       |- Source
           |- Project
               |- Project.h
               |- Project.cpp
               |- ProjectBuild.cs
               |- ...
           |- Project.Target.cs
           |- ProjectEditor.Target.cs
       |- Project.sln
       |- Project.uproject
       |- ...
    ```
    我只列出了比较重要的内容，对于理解打包第三方库来说的。相比于默认的（创建工程的时候）文件结构来说，唯一不同的地方是在`Plugins`的文件夹下面的插件中的`Binaries`和`Source`文件夹出现了`ThirdParty`文件夹。当然这个文件夹也不是我创建的。那么我也不是凭空就知道正确的结构。要点就在于UE4提供了第三方库的插件使用模板。
    1. 有效使用UE4提供的第三方库的插件使用模板。可以参考下面的第二个链接。`Edit -> Plugins -> NewPlugin`打开创建新的插件窗口。拉到最下边有一个**Third Party Library**的模板，创建使用。当然这还只是开始。不过这个时候已经有了雏形，大体的插件文件结构已经构建完成，不需要再去额外的创建文件，只要适当的修改文件就好了。
    2. 使用这个方法创建了第三方库之后你会发现，在你的Plugin的*Source/ThirdParty/PluginNameLibrary*下面就有了一个Solution，完全可以直接在这个Solution中制作想要的第三方库。
    3. 但是由于生成的Solution的名字固定，我选择删除了这个Solution，把上面自己新建的Solution给复制进去了。

4. 确保第三方库（DLL）的路径。下面是第三方库的编译设置文件(Build.cs)中的内容
    ```c#
    // 位于上述目录结构中的PluginAnameLibrary.Build.cs
    // Fill out your copyright notice in the Description page of Project Settings.

    using System.IO;
    using UnrealBuildTool;

    public class test1Library : ModuleRules
    {
        Type = ModuleType.External;

        if (Target.Platform == UnrealTargetPlatform.Win64)
        {
            // Add the import library
            PublicLibraryPaths.Add(Path.Combine(ModuleDirectory, "x64", "Release"));
            PublicAdditionalLibraries.Add("ExampleLibrary.lib");

            // Delay-load the DLL, so we can load it from the right place first
            PublicDelayLoadDLLs.Add("ExampleLibrary.dll");
            // RuntimeDependencies.Add(new RuntimeDependency(Path.Combine(ModuleDirectory,  "x64", "Release", "ExampleLibrary.dll")));
            // RuntimeDependencies.Add(dll_path);
            string dll_runtimePath = Path.Combine(ModuleDirectory, "..", "..", "..", "Binaries", "ThirdParty", "test1Library", "Win64", "ExampleLibrary.dll");
            RuntimeDependencies.Add(dll_runtimePath);
        }
        else if(Target.Platform == UnrealTargetPlatform.Mac)
        {
            PublicDelayLoadDLLs.Add(Path.Combine(ModuleDirectory, "Mac", "Release", "libExampleLibrary.dylib"));
        }
    }
    ```
    第一眼看上去全是懵的，这都是啥？直到现在我也没有能全理解。但是大体的内容我猜的就是保证第三方库的位置。按照代码中的路径依次填充准备好的dll文件，虽然lib文件也设置了，但是估计也不会用到，用的时候再说，此次首要目的是打包dll文件。至于其他的平台的设置，根据要求把。这里重要的是加入了`RuntimeDependencies`这个依赖项。
    1. 这个依赖项的内容或许会在上面的**UnrealBuildTool**章节中详细展开，或者下面的**各种验证**章节中展开。
    2. 这一部分主要是对dll进行配置，添加依赖项，保证dll在打包过后依然存在，同时也可以在这个**Module**(Module的概念参考上面，我的理解就是**每一个Module都需要一个Build.cs文件**)里加入dll的源码或者dll的源文件，但是如果加了源文件并且使用了源码的话，那么使用dll的意义应该也不存在了吧。

5. 插件源码配置。这个部分是对dll的内容进行调用的，或者说为dll的调用提供接口。
    ```c#
    // 文件结构
    Source
       |- PluginModule
             |- Private
             |- Public
             |- PluginModule.Build.cs
       |- ThirdParty
    ```
6. 如何制作一个扩展插件。这个需求我不知道要如何来描述为好，总之就是上面的插件Module为我们提供了调用接口后，我要如何在另外一个Module中调用这个接口。即如何在工程的主Module中使用扩展的插件Module中的接口。
    - [An Introduction to UE4 Plugins](https://wiki.unrealengine.com/An_Introduction_to_UE4_Plugins)
    
    这个是一个神一样的Wiki，手把手教你制作一个plugin或者说是扩展的Module。跟这篇我完全不知道在讲什么的[Linking Static Libraries Using The Build System](https://wiki.unrealengine.com/Linking_Static_Libraries_Using_The_Build_System)的这篇Wiki比起来真是一个天上一个地上。不少老外都在diss这篇。

当我差不多理解了上面的内容，也成功的让我的dll在打包后也成功输出了。但我还是想吐槽一句，UE4 的编译系统（UBT）这个东西让我这个初学者感觉到了恶心。为什么，为什么我找不到这个东西具体的Manual？还要去看各种UE4引擎的源码去猜呢？



- [【UE4】プラグインのTypeをRuntimeにしたのにShippingでパッケージ化するとロードされないとき](https://tubezgames.com/2017/01/ue4-plugin-build/)
- [［UE4］プラグインをパッケージ化しよう！](http://historia.co.jp/archives/6717/)





### 各种验证，各种现象

#### RunTimeDependencies.Add(String path)
这个东西加进去的，首先要保证路径正确由函数名字就可以明白这个是为dll库添加运行时依赖的，也就是确保运行时按照这个路径下能找到dll文件。但是这里有需要注意的问题。这个依赖项的作用，据我简单实验观察得到的结果是
- **被路径指定的文件在打包后会被一起打包**

这非常重要。因为非常头疼的事情就是在打包的时候，dll并没有被带上。自己copy进去？我没有实验过不知道好不好用，但是肯定是会被打的。

比如说上面的例子中按照该路径添加了这个依赖，那么在打包后，这个路径就会被保存下来，当然文件也会存在，而之后的代码中按照原本的路径来取得dll也是没有问题的。

那么这里就有了路径留哪个文件夹的问题。**Binaries？Source？**

个人倾向于Binaries文件夹，毕竟dll算是2进制文件，放到这个文件夹里不会产生歧义。但是第三方库的生成往往不是UE4的工作，而是VS中编译好（参考上面）生成。我只好在binaries文件夹的相应路径中复制一份。（Source里更倾向于放置源码，放进去源码的话貌似也可以使用，毕竟都一样嘛）

**这个依赖非常关键，它让你的dll在工程打包的时候能够一起被打包。**

在我写下这么多文字的时候，就隐隐约约的察觉到了一个问题。我是因为写好的插件直接就放在了一个叫`Plugins`的文件夹里面了，与这个插件相对的Build.cs文件是一点都没有写。上面提到的又是扩展插件又是Module的，在一个Module里面提供dll的调用接口，为什么我现在工作的Module里不能提供？

肯定是可以的。原因应该就是我没有给自己的Plugin文件写任何的编译依赖，导致没有没打包。若是被打包进工程，能找到dll的话肯定能使用了啊。

关键就是上面的这个依赖项的添加。


- [Packaging Plugin with third party Dll](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1364966-packaging-plugin-with-third-party-dll)
- [How do I add thirdparty library?](https://answers.unrealengine.com/questions/218616/how-do-i-add-thirdparty-library.html)
- [How do you statically link an external DLL/dylib to your project?](https://answers.unrealengine.com/questions/197667/how-do-you-statically-link-an-external-dlldylib-to.html)
- [UNREAL ENGINE: INCLUDING A THIRD-PARTY LIBRARY (ON THE EXAMPLE OF THE POINT CLOUD LIBRARY AND BOOST) [TUTORIAL] [DOWNLOAD]](http://www.valentinkraft.de/including-the-point-cloud-library-into-unreal-tutorial/)
- [Adding third party DLL path for plugin](https://answers.unrealengine.com/questions/427772/adding-dll-path-for-plugin.html?sort=oldest)

为了搞懂如何打包第三方库，以及做了各种的实验所参考的文章。
- [UE4: バイナリー配布や実行時リンク向けのライブラリーを UE4 プロジェクトへ組み入れる方法を mecab で解説](https://usagi.hatenablog.jp/entry/2018/03/14/082509)
- [Using thirdparty libraries in our UE4 mobile/desktop project](https://www.parallelcube.com/2018/03/01/using-thirdparty-libraries-in-our-ue4-mobile-project/)
- [UE4のカスタムプラグイン作成時におけるThirdPartyライブラリの取り込み方](https://qiita.com/T_Sumisaki/items/b5f01fc7c61330c0a38f)
- [Plug-ins & Third-Party SDKs in UE4](https://www.slideshare.net/GerkeMaxPreussner/plugins-thirdparty-sdks-in-ue4)
- [UE4でのPlugin開発記録メモ(FMessageDialogでダイアログ表示編)](https://qiita.com/BlackMa9/items/38701d6b4122b966975c)


还有就是感谢UE4的源码了，我翻了许多源码内的各种插件的使用，终于脑海里有了大概的印象。


#### IpluginManager::Get()
这个类是干什么的以后补上，我见到的是用它来取得Plugins的路径：
```c++
FString BaseDir = IPluginManager::Get().FindPlugin("PluginName")->GetBaseDir();

```

需要注意的是使用这个类的时候除了要补上头文件`#include IPluginManager.h`之外，还要把依赖加进Module里面。
```c#
PrivateDependencyModuleNames.AddRange(new string[] { "Projects" });
```

要不然项目编译不会通过。
- [IPluginManager::Get() Linker error - Not finding implementation](https://answers.unrealengine.com/questions/745481/ipluginmanagerget-linker-error-not-finding-impleme.html)

#### C# SYstem.IO.SetAttributes
- [File.SetAttributes(String, FileAttributes) Method](https://docs.microsoft.com/ja-jp/dotnet/api/system.io.file.setattributes?view=netframework-4.8)


#### Json数据解析
- [Parsing Json files](https://orfeasel.com/parsing-json-files/)


#### 各种路径获取
- [Packaged Game Paths, Obtain Directories Based on Executable Location](https://wiki.unrealengine.com/Packaged_Game_Paths,_Obtain_Directories_Based_on_Executable_Location)

##### 获取绝对路径
```C++
FString RelativePath = FPaths::GameContentDir();
 
 FString FullPath = IFileManager::Get().ConvertToAbsolutePathForExternalAppForRead(*RelativePath);
```
- [Getting full path of project directory](https://answers.unrealengine.com/questions/580005/getting-full-path-of-project-directory.html)

#### 文件夹创建
- [File Management, Create Folders, Delete Files, and More](https://wiki.unrealengine.com/File_Management,_Create_Folders,_Delete_Files,_and_More)



# 其他
## DLL中头文件的引入
在使用**plog** 源码的时候尝试引入头文件的地方：
```C++
// for example
#include <plog/Util.h>
```
上面的引入会出错，而把尖括号换成双引号后就好了。

关于这两者的区别：
- 用尖括号来指定文件时，预处理器是以特定的方式来寻找文件，一般是环境中或编译器命令行指定的某种寻找路径。这种设置寻找路径的机制随机器，操作系统，C++实现的不同而不同，要视具体的情况而定。
- 用双引号来指定文件时，预处理器是以“由实现定义的的方式”来寻找文件。它通常是从当前的目录开始寻找，如果没有找到，那么include命令就按与尖括号同样的方式开始寻找。

要说DLL不支持尖括号的查找也不是，内置的一些库还是可以使用尖括号来include的，难不成自己添加的文件就要使用双引号吗？

另外关于头文件返回上一级路径的写法：
```C++
#include "../Util.h"
```

## 函数名字前面加上&

> In C++, when the ref-sign(&) is used before the function name in the declaration of a function it is associated with the return value of the function and means that the function will return by reference.

- [Use of '&' operator before a function name in C++](https://stackoverflow.com/questions/23776784/use-of-operator-before-a-function-name-in-c)

## C++类构造函数初始化列表
- [Constructor member initializer lists](https://www.learncpp.com/cpp-tutorial/8-5a-constructor-member-initializer-lists/)

可以参照以上的文章，主要是可以在声明的同时初始化，在构造函数里面的知只是赋值操作，不是所谓的初始化。

比如说类中想要拥有const类型的变量的话，可以利用构造函数的初始化函数列表来给类中常量赋初值。

## C++中的函数指针

## C++运算符重载

## C++中的流
要把流与字符串分开来看，流是对象，可以用来处理字符串。

### stringstream

### st(), c_str()函数

## 深入理解char*与char[]的差别

## 关于类中静态成员的理解
这里需要强调理解的是，类中静态成员的存储位置是静态存储区，只有一个拷贝，无论类被实例化了多少个，静态成员只有一个，还有一些其他的重要的使用方式，之后整理。

## extern "C"
- [よくみる extern "C" {} と __cplusplus - はこべにっき](https://hakobe932.hatenablog.com/entry/20090104/1231073299)

## mutex::lock
- [mutex::lock - cpprefjp C++日本語リファレンス](https://cpprefjp.github.io/reference/mutex/mutex/lock.html)


