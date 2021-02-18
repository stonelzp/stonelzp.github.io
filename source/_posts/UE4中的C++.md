---
title: UE4中的C++
date: 2018-08-01 12:27:59
permalink: UE4-C++
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
UE C++拥有着自己的库，当然也可以使用C++的标准库(STL)，但是不同的地方还应该好好记载下来，同时也是对C++的一种复习。

<!--more-->

# 1.值类型？指针？
在标准C++中，类的声明可以
```C++
ClassExample ObjName;

/或者是

ClassExample* ObjPtr;
```
而在UE4中为了统一值类型与指针的规则，将这种类的声明全部使用指针类型，不使用值类型。
```C++
UObject* o;
```

# 2.UE4生成类对象实例(Instance)
直接上例子：
```C++
//声明
UMyClass* MyClass;

//生成实例
MyClass = NewObject<UMyClass>();

//或者
MyClass = NewObject<UMyClass>(Owner);
```
Tips:
- 在构造函数中不能使用`NewObject<T>`生成其他的对实例。会使程序Crash。如果想生成其他的Object的话使用`FObjectInitializer::CreateDefaultSubobject<T>`函数
    ```C++
     ASomeActor::ASomeActor(const FObjectInitializer & ObjectInitializer) : Super(ObjectInitializer){
        SampleActor = ObjectInitialize.CreateDefaultSubobject<ASampleActor>(this, TEXT("SampleActor"));
     }
    ```

## UE中的构造函数
UE4中添加构造函数的话有三种方法：
- 没有构造函数
- 声明默认的构造函数
- 带有`const FObjectInitializer& ObjectInitializer`参数的构造函数

使用方法的话就像这样：
```c++
// .h
class AMyActor: public AActor
{
    GENERATED_BODY()
public:
    AMyActor(onst FObjectInitializer& ObjectInitializer);
}

// .cpp
AMyActor::AMyActor(const FObjectInitializer& ObjectInitializer) 
    : Super(ObjectInitializer)
{

}
```

至于为什么要使用`FobjectInitializer`参数的构造函数的原因我现在还不是非常清楚。



# 3.Actor的实例化


# 4.Actor的Component实例化

## 1.在UE4C++中试着添加component

### 1.添加SkeletalMeshComponent
或许有一天我会再次遇见这个问题，就是想要用C++来写全部的东西。比如说在C++中为一个Actor添加ActorComponent（这个ActorComponent就是要写的C++本体了），这是很顺利。然后再为这个Actor添加一个SkeletalMeshComponent组件。

问题就出现在这里。

我天真的以为在这个ActorComponent中使用`this->GetOwner()->GetRootComponent()`然后创建组件就能成功了。

```C++
// 其他的省略
this->GetOwner()->GetRootComponent()->CreateDefaultSubject<USkeletalMeshComponent>(TEXT("SkeletalMeshName"));
```

然后UE4Editor就驾崩了。

`CreateDefaultSubject`直接用就行，不用加前面的一串限定，我还不知原因。把前面删了之后就不会崩溃了。而且去View试图中找Actor下面也确实有生成的SkeletalMeshComponent了。

于是我就SetSkeletalMesh之后发现，明明SkeletalMesh上面有值，也就是正确赋值了，Mesh不显示。

对比了一下终于发现这种方法生成的SkeletalMeshComponent，后面带了一个**inherited**的标识。

去搜了一下**inherited skeletal mesh component doesnt show mesh**

天杀的这种UE4程序级别的BUG也能被我碰到。都2019年了还没解决么。看样子是15年提的问题。

也许会有别的方式来为一个Actor在C++中创建SkeletalMeshComponent，但是我没有找到。想了想，还是直接用Blueprint吧，当然直接让Actor直接Add一个Component就能解决然后在C++中保留这个组件的ref就行，但是这样的话不是纯C++的话，为什么直接效率一些直接用BP吧。

为什么我C++中创建就是*inherited*的对象？是创建的姿势不对么...

我也试过用`NewObject<USkeletalMeshComponent>`来创建过，但可能是因为代码在构造函数里，还没运行，只是编译一下

UE4Editor就驾崩了。

### 添加各种Component
看啊，我上面提问的弱智的问题。UE4中的Component的创建自有其方式，好好学着就好了。

在这里我想记录一下关于Actor的**RootComponent**的问题。

- [Cannot attach actors without Root Components](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/15790-cannot-attach-actors-without-root-components)

> An Actor without a root component has no position.
> 
> If you are building an Actor through blueprints, it has a "dummy" Scene Component by default that exists to give it a position. You can add other components below that, or replace it as the root component.
> 
> In the C++ case, the default Actor does not provide this root component. In your class constructor you will need to create some form of Scene Component and assign it to the RootComponent variable. The simplest form of this would be:
> 
> RootComponent = PCIP.CreateDefaultSubobject<USceneComponent>(this, TEXT("SceneComponent"));
> 
> If you wanted to add a Mesh component you could instead create that. Looking at StaticMeshActor.cpp should give some insight in how to do this for a more complex (and visual) component type.
> 
> If your goal is to build up a number of related static meshes, or what not in a hierarchy, I'd probably recommend looking at building that out in blueprints rather than C++ as it is much easier to make proper asset references and relationships in your hierarchy there.

于是我就找了一下StaticMEshActor中的实现看看：
```c++
AStaticMeshActor::AStaticMeshActor(const FObjectInitializer& ObjectInitializer)
        : Super(ObjectInitializer)
{
        SetCanBeDamaged(false);

        StaticMeshComponent = CreateDefaultSubobject<UStaticMeshComponent>(StaticMeshComponentName);
        StaticMeshComponent->SetCollisionProfileName(UCollisionProfile::BlockAll_ProfileName);
        StaticMeshComponent->Mobility = EComponentMobility::Static;
        StaticMeshComponent->SetGenerateOverlapEvents(false);
        StaticMeshComponent->bUseDefaultCollision = true;

// BEGIN ENLIGHTEN MODS
        StaticMeshComponent->EnlightenLightingModeOverride = ELM_ContributeLightmap;
// END ENLIGHTEN MODS

        RootComponent = StaticMeshComponent;
        
        // Only actors that are literally static mesh actors can be placed in clusters, native subclasses or BP subclasses are not safe by default
        bCanBeInCluster = (GetClass() == AStaticMeshActor::StaticClass());
}
```

关于RootComponent的描述在Actor中是这样的：
```c++
protected:
        /** The component that defines the transform (location, rotation, scale) of this Actor in the world, all other components must be attached to this one somehow */
        UPROPERTY(BlueprintGetter=K2_GetRootComponent, Category="Utilities|Transformation")
        USceneComponent* RootComponent;
```

以下是我的猜测：

一般情况下都是被推荐存在一个RootComponent的，而且从Actor的源码来看已经预备好了一个直接用就可以了。但是当你不使用RootComponent的时候会把第一个创建的Component赋值给这个预备好的Rootomponent? 我也不确定，只是就发生的现象而言。

至于使用的话参照上面的就好了。

那段引用中的英文很重要，改天我多看几遍。



# 5.从Content(Asset)中加载Object对象数据

# 6.UE4中的容器



# 7.UE4C++中的一些小知识点
## 1.从SkeletalMeshComponent得到AnimationBlueprint
```C++
// USkeletalMeshComponent::GetAnimInstance()
USkeletalMeshComponent * SkeMC = SkeMC_ptr;
UAnimInstance* animIns = SkeMC->GetAnimInstance();

// 拓展
// Copy one anim instance to another charactor
ACharactor* ch = /*...*/;
UAnimInstance * mi = GetMesh()->GetAnimInstance();
ch->GetMesh()->SetAnimInstanceClass(mi->GetClass());
```

## 2.运行时取得SkeletalMesh的Bone情报
取得Bone情报能做什么呢？

### 1.Get the name of all bones
参考：
- [Skeletal mesh overlap returns null bone](https://answers.unrealengine.com/questions/491046/skeletal-mesh-overlap-returns-null-bone.html)

```C++
    USkeletalMeshComponent* SkelMeshComp = Cast<USkeletalMeshComponent>(GetMesh());
    if (SkelMeshComp) {
        TArray<FName> BoneNames;
        SkelMeshComp->GetBoneNames(BoneNames);
        for (int i = 0; i < BoneNames.Num(); i++) {
            FBodyInstance* BodyInst = SkelMeshComp->GetBodyInstance(BoneNames[i]);
            if (BodyInst) {
                UPhysicalMaterial* PM = BodyInst->GetSimplePhysicalMaterial();
                EPhysicalSurface SurfaceType = UPhysicalMaterial::DetermineSurfaceType(PM);
                if (SurfaceType_Default != SurfaceType) {
                    PhysicBoneNames.Add(BoneNames[i]);
                }
            }
        }
    }
```
### 2.Get Socket Location
搜了半天，又是Bone又是FbodyInstance什么的，结果直接获取bone的世界坐标的话有这个函数。

- [Get Socket Location](http://api.unrealengine.com/INT/BlueprintAPI/Utilities/Transformation/GetSocketLocation/index.html)

C++和Blueprint都能用，自己以前也有试过在Skeletal加Socket，连自己加的Socket都能直接获取，方便了。


### 3.Animation切换
就我自己所接触的动画处理而言，将动画的主体的状态跟动画的播放是分开的，也就是说**存在一帧的延迟**。

我不知道真正的游戏开发中的情况是什么样子的，不真正的去看人家的源码我是不会明白的......

记录一下要点。

- 状态的切换和真正动画的切换存在一帧的延迟
- 真正动画的切换在UE4中有一个cross-fade支持

为什么要注意这个是因为动画中如果出现位移，而真正的位移跟动画应该是分开处理的，动画中有位移的话就得根据这段位移进行实际位移的调整来适应下一段动画。不然的话就会产生残留帧（产生跳帧的感觉），因此对于动画的调整就要精确到每一帧的地步，超级麻烦。

Unity和UE4都一样，为Animation的切换提供的过渡的功能，有的时候反而起了反作用。

但是吧，这种只是强行修复位移的效果终究不如使用真正流畅的动画素材要来的好，可能的话，还是制作正确的动画比较合理（拜托美工啦）。


## 3.Handling animation in C++
把UE4中的关于Animation的处理，交给C++来做。就结论来说就是：做个C++类的AnimInstance以供AnimationBlueprint来继承。

我特么...暂时没有找到非要这样的做的理由。

- [Handling animations in C++](https://orfeasel.com/handling-animations-in-c/)
- [Animation Blueprint, Implement Custom C++ Logic Via Tick Updates](https://wiki.unrealengine.com/Animation_Blueprint,_Implement_Custom_C%2B%2B_Logic_Via_Tick_Updates)
- [Animation Blueprint, Set Custom Variables Via C++](https://wiki.unrealengine.com/Animation_Blueprint,_Set_Custom_Variables_Via_C%2B%2B)

上面的教程和Wiki手把手教学，教你如何使用C++操作动画，但是并不是全部C++，动画跳转还是用的AnimationBlueprint。都已经用了AnimBP了......

## (1) 暂停动画
我尝试着找到停止AnimationInstance的tick函数的方法，但是失败了，后来发现在**AnimationBlueprint**附着的**SkeletalMesh**组件上有下面两个属性

- Pause Anims
- Global Anim Rate Scale

控制动画播放和暂停，或者动画的播放速度。

## 4.使用C++制作Blueprint的Node
- [Blueprints, Creating C++ Functions as new Blueprint Nodes](https://wiki.unrealengine.com/Blueprints,_Creating_C%2B%2B_Functions_as_new_Blueprint_Nodes)

我有试着写过全局的输出LOG的函数，应该多少有这篇文章的内容，这篇应该介绍的很多，有时间整理一下。

## 5.在C++中生成BlueprintClass实例
为什么要做这么蛋疼的事情?但也是的确事出有因，用蓝图写一些组件然后赋值真的太方便了啊...C++里写的话就得各种找组件然后Load各种素材找路径，在蓝图里一拽就解决了。

这里主要是利用了[SpawnActor](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Actors/Spawning/index.html)这个类。

- [UE4: C++ コード から C++ クラスまたは Blueprint のアクターを FindObject して SpawnActor する方法](https://usagi.hatenablog.jp/entry/2017/10/17/123000)

上面这篇文章写得很好了，就是风格太花里胡哨了，看着有些累眼睛。

为了能先得到要实例化的BlueprintClass，我没有选择使用从ContentBrower里面Load素材的方法。而是选择了最方便的方法，当然上面的文章里也有提到。

```C++
UPROPERTY(EditAnywhere, Category = "PawnBlueprintClass")
    TSubclassOf<class AActor> SunDevilBPClass;
```

公开一个类模板的属性，取得类模板。（前提是BlueprintClass继承了AActor类，当然这里填上你继承的类类型就好了）公开了这个属性之后，就可以在Editor的Detail面板里把想要实例化的蓝图类选进去。

还有一篇文章，或许也能作参考，但是我没有试过，之后有时间的话可以试一下。

- [C++でのブループリントクラス取得方法につきまして](https://answers.unrealengine.com/questions/725873/c%E3%81%A6%E3%81%AE%E3%83%95%E3%83%AB%E3%83%BC%E3%83%95%E3%83%AA%E3%83%B3%E3%83%88%E3%82%AF%E3%83%A9%E3%82%B9%E5%8F%96%E5%BE%97%E6%96%B9%E6%B3%95%E3%81%AB%E3%81%A4%E3%81%8D%E3%81%BE%E3%81%97%E3%81%A6.html)


### FActorSpawnParameters

### TSubclassOf
- [TSubclassOf](https://api.unrealengine.com/INT/API/Runtime/CoreUObject/Templates/TSubclassOf/index.html)

上面用到的一个类，感觉挺重要的，需要总结一下这个类相关的东西。


### 疑问
我会选择使用C++来生成蓝图类的原因除了我制作了一个C++管理类来生成蓝图类大量实例之外，还有就是对蓝图类初始化抱有疑问。

1. 蓝图类的constructor中的内容，在把蓝图拖入场景这个操作中，会执行几次？
2. 假设这个蓝图类有一个C++的父类，同样是上面的操作，父类的构造函数会执行几次？

我分别用输出log的方法输出一些内容，得到的结果很让人困惑。蓝图中的constructor跟真正意义上的构造函数又有不同貌似。然而让我不可思议的是，问题2中得到的结果是两次。拖到场景中生成了一个实例，但是父类的构造函数调用了两次。

要说不可思议问题1的constructor貌似执行了很多次（2次以上）。


## 6.深入理解UE4中的UClass
UE4中会经常看到`UCLASS()`宏，摘自官网的描述：
> The UCLASS macro gives the UObject a reference to a UCLASS that describes its Unreal-based type. Each UCLASS maintains one Object called the 'Class Default Object', or CDO for short. The CDO is essentially a default 'template' Object, generated by the class constructor and unmodified thereafter. Both the UCLASS and the CDO can be retrieved for a given Object instance, though they should generally be considered read-only. The UCLASS for an Object instance can be accessed at any time using the GetClass() function.

- [Objects](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/Objects/index.html)

想上面描述的那样，知道UCLASS宏维护了一个CDO的东西。

###  GetClass() vs StaticClass()
- [GetClass() vs StaticClass()](https://answers.unrealengine.com/questions/346164/view.html)




# 8.UE4C++中遇到的一些问题调查

## 1.Pointer to incomplete class type is not allowed
在我使用一些组件指针的时候，想要得到组件的名称，使用`->`来调用函数的时候就出现了这个问题。

[Pointer to incomplete class type is not allowed](https://stackoverflow.com/questions/12027656/pointer-to-incomplete-class-type-is-not-allowed)

上面的链接有解释。

**An "incomplete class" is one declared but not defined.E.g.**

最终的解决是，好好的把相应的头文件包含进去。不是说可以声明指针就代表这个指针就能调用这个指针代表的类中所有公开函数了。话说，没有好好的包含人家的头文件却能在文件中没问题的声明这个类型的指针，对我来说真是邪门了。UE4真是搞不懂了。

## 2.C++ Cross Reference
即可以解释上面我遇见的问题的原因。包含指针声明不代表编译器能找到指针指向内容的定义。

头文件的交叉引用，我觉得我本来是遇不见的，或者说遇见了也会不屑一顾，不会去使用的。

但是凡事都有例外不是。

基本来说，程序中出现使用交叉引用的话，就是程序设计出现了重大的问题，基本上需要重新设计了。但是我非要用这种交叉引用是有原因的...当然因为我水平不行是最大的原因。

要怎么解决这个问题呢？

- [C++交叉引用问题](https://blog.csdn.net/github_30605157/article/details/52063056)
- [Resolve build errors due to circular dependency amongst classes](https://stackoverflow.com/questions/625799/resolve-build-errors-due-to-circular-dependency-amongst-classes)

上面的两个文章链接，告诉我怎么解决交叉引用的问题。总结来说就是使用**Forward Declaration**。

我的解决方案就是对于相互引用的两个类`A`，`B`。分别置于不同文件`A.h`,`A.cpp`,`B.h`,`B.cpp`。我们的希望A中包含B的实例，但同时B中包含A的指针引用。

则此时对B来说，我就是在头文件中引用了A的指针，但是不实际使用A的内容（指针调用），于是就在`B.h`中添加`class A`的声明，但是不导入A的头文件。在`B.cpp`中引入A的头文件对指针加以使用。这样编译器就明白我到底在捣鼓什么了。

- [C++中前置声明的应用与陷阱](https://blog.csdn.net/yunyun1886358/article/details/5672574)

上面的文章解释了前置声明的原理，感觉讲的非常好。
