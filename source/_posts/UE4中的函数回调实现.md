---
title: UE4中的函数回调实现
date: 2018-07-31 15:26:25
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
回调函数的含义，实际上我也不太清楚，搜回调函数得到的关键字有很多：闭包，代理，委托，Wrapper，等等。但有一点我很清楚，这些我都不太懂。

说是在不同的语言中有不同的叫法。其本质上就是一个函数指针，而在汇编层面，就是子程序代码的首地址。每一段编译器先放一个占位符，最后放入实际的值。

<!--more-->
上面的话是[Unreal用到一些编程技巧](http://sourcecodereadingofunreal.readthedocs.io/en/latest/content/programmingSkill.html)里的内容。

# 先理解什么是"函数回调"

> A "callback" is any function that is called by another function whitch takes the first function as a parameter. A lot of the time, a "callback" is a function that is called when something happens. That something can be called an "event" in programmer-speak.

摘自Stack Overflow：
- [How to explain callbacks in plain english? How are they different from calling one function from another function?](https://stackoverflow.com/questions/9596276/how-to-explain-callbacks-in-plain-english-how-are-they-different-from-calling-o)

实际的应用场景就类似：**类A需要类B做一件事情，类B做完之后需要告知类A已经做完了这件事情，至于告知的具体方法(函数的具体实现)类A并没有事先告诉类B(编译期间)，等类B做完看一下类A留下的便条(绑定的事件)，根据信条里面的内容(函数的具体实现)来通知类A。**

用简单的代码来表示CallBack就是：

```C++
void A(){
    printf("B work has done.")
}

void B( void (A_prt) (void)){

}
```

学了C++的虚函数之后，理解起来就是快 =。=

# UE4中的函数回调(CallBack)
C++中应该有许多种方式的回调，UE C++(UnrealEngine中的C++)则是使用的**DELEGATE**和**EVENT**来实现。

关于UE C++的代理，存在着以下的几种方式：
- 静态的Single-cast Delegates
- Dynamic Single-cast Delegates
- 静态的Multi-cast Delegates
- Dynamic Multi-cast Delegates

这几种代理的实现有什么不同需要后续整理，可以参考：
- [What difference betweens delegates?](https://answers.unrealengine.com/questions/533323/what-difference-between-delegates.html)

有的时候需要很好的利用一下UE4的[官方论坛](https://answers.unrealengine.com/index.html)，像是Stack Overflow一样。

## Dynamic Multi-cast Delegates
Dynamic Multi-cast Delegates是唯一的一种可以和UE4的Blueprint联动的代理实现方式。

**Dynamic Multi-cast Delegates的UE C++中的声明**
```C++
//File: CallbackExample.h
//Class: ACallbackExample

DECLARE_DYNAMIC_MULTICAST_DELEGATE(FZeroInputDelegate);    //没有参数的声明方法
UPROPERTY(BlueprintAssignable, Category="UE C++ Book")    //BlueprintAssignable属性使得这个代理在Blueprint中也取得到。 ->此处在真正的工程中不应该写注释，会出编译问题
FZeroInputDelegate TheZeroInputDelegate;

DECLEAR_DYNAMIC_MULTICAST_DELEGATE_TwoParams(FTwoInputsDelegate, float, FloatVal, int32, IntVal);    //两个参数的声明方法
UPROPERTY(BlueprintAssignable, Category="UE C++ Book")
FTwoInputsDelegate TheTwoInputsDelegate;
```

Tips:
- 参数最大允许有8个。参数的声明类似上述两个参数的声明，先是类型后是参数名。
- 参数不同的代理声明只需要将上述的数字换成相应参数的英文就可以，比如说八个参数的情况声明代理的语句就会变成：`DELEAR_DYNAMIC_MULTICAST_DELEGATE_EightParams`

这样制作的代理，可以在UE的Blueprint中获得到这个代理（似乎是作为Event）并进行实现了。别忘了指定Event与Target。即把这个代理委托给一个类的实例(Instance)。在Blueprint中似乎是以`Event`而存在的，Blueprint中实现的操作是把这个调用的Event登录到这个代理上去。这样在UE C++中调用这个代理，也会调用Blueprint中的事件，执行这个事件的实现部分。

**调用Blueprint中的代理实现**
```C++
// File: CallbackExample.cpp
// Class: ACallbackExample

auto ACallbackExample::ExecuteDelegate(const float FloatVal, const int32 IntVal) -> void {
    TheZeroInputDelegate.Broadcast();
    TheTwoInputsDelegate.Broadcast(FloatVal, IntVal);
}
```
官方参考链接：
- [Dynamic Delegates](https://docs.unrealengine.com/en-us/Programming/UnrealArchitecture/Delegates/Dynamic)
- [Events](https://docs.unrealengine.com/en-us/Programming/UnrealArchitecture/Delegates/Events)
## Blueprint Event与Blueprint Function
关于事件与函数的区别，没有返回值的被叫做事件，有返回值的叫做函数。是否真实需要验证。在UE C++中可以登录Blueprint的事件，通过给UPROPERTY宏添加属性来完成。话说回来这个是函数，应该使用UFUNCTION宏才对，不知道为什么书上这么说。

用于事件声明的属性有两种，区别的方式是是否在UE C++中有默认的实现。
- `BlueprintImplementableEvent`:没有默认实现
- `BlueprintNativeEvent`:拥有默认实现

**Blueprint Event,Blueprint Function在UE C++中的声明**
```C++
// File: CallbackExample.h
// Class: ACallbackExample

// BP Event
UFUNCTION(BlueprintImplementableEvent, Category="UE C++ Book")
void FloatInputEvent(const float FloatVal);

UFUNCTION(BlueprintNativeEvent, Category="UE C++ Book")
void VectorInputEvent(const FVector& VecValue);

// BP Function
UFUNCTION(BlueprintImplementableEvent, Category="UE C++ Book")
float IntInputFunction(const int32 IntInput);

UFUNCTION(BlueprintNativeEvent, Category="UE C++ Book")
TArray<float> VecArrayInputFuncion(const TArray<FVector>& VecValues);
```

**Event的默认实现**
```C++
// File: CallbackExample.cpp
// Class: ACallbackExample

//static FVector TheVector事先声明的属性

auto ACallbackExample::VectorInputEvent_Implementation(const FVector& VecValue)->void{
    TheVector = VecValue;
}

auto ACallbackExample::VecArrayInputFuncion_Implementation(const TArray<FVector>& VecValues)-> TArray<float>{
    TArray<float> Result;
    for (const auto& Val : VecValue)
        Result.Emplace(FVector::Dist(TheVector, Val));

    return Result;
}
```

以上的代码实装完成之后，继承了上面的`CallbackExample`类的Blueprint就可以在Blueprint Editor中对上述UE C++中的事件与函数进行重写了。

以上，是对UE中的代理与事件，在可用范围内的总结与实现。但是对于在什么情况下使用这一点上仍然有许多疑问。

---
2020/06/09 更新

不知道我当时写下这篇文章的时候为什么没有对上面的`_Implementation`后缀产生大大的问号，定义了一个函数之后，函数的实现部分的函数名字竟然需要添加后缀，让人大呼这是什么抛瓦。

应为时间有限，就快速记录一下。

像上面那样函数的定义和实现都是在同一个文件的`.h`,`.cpp`中实现的，就是说：
```c++
// SampleA.h
UFUNCTION(BlueprintNativeEvent, Category="Execute")
void Execute(int value) const;

// SampleA.cpp
void SampleA::Execute_Implementation(int value) const{ }
// auto SampleA::Execute_Implementation(int value) const{// default Implementation }   // 类型推断还能这么用的吗
```
这样的是好像也没有什么太大的问题，好像也很容易理解，但是当这个类似于事件的提供移到子类的时候，就有些让人摸不到头脑了。

```c++
// SampleA.h
UFUNCTION(BlueprintNativeEvent, Category="Execute")
void Execute(int value) const;

// SampleA.cpp
// 这里应该没有实现，但是这个函数是有默认实现的，没错，蓝图那边有默认实现，如果我们没有这个C++的_Implementation实现，那么就是蓝图的默认实现被调用，但是我们对C++的_Implementation进行了实现，那么就是C++这部分实现会被调用。

// SampleB.h
class SampleB : public SampleA
{
    // 没有这个子类的声明会报错
    virtual void Execute_Implementation(ine value) const override;

}

// SampleB.cpp
void SampleB::Execute_Implementation(ine value) const{ // 函数实现}
```
如果是继承来的`Execute`就需要这样的样子，总之就是，带有
- UFUNCTION(BlueprintImplementableEvent, Category = "Foo")
- UFUNCTION(BlueprintNativeEvent, Category = "Foo")

的可能都要这么实现。

- [What's the deal with appending '_Implementation' to functions?](https://www.epicgames.com/unrealtournament/forums/unreal-tournament-development/ut-development-programming/11600-what-s-the-deal-with-appending-_implementation-to-functions)

> FUNCTION_Implementation is for the UnrealEngine framework. Calling FUNCTION will also call the _Implementation part. FUNCTION is only a wrapper generated by the UHT. CPP includes the generated classes/headers.

---
## UE C++的函数回调实现
说实话，函数回调跟代理的概念我还是没太理清。在许多的实际应用上，有些操作你叫它代理也行，叫它函数回调也行。用自己的感受就好比是参考系不同一样。就像上面举的函数回调的例子来说：

函数回调就是类A想要在类B在运行中达成某种条件的时候调用类A提供的函数。这个时候类A是主参考系。

代理就是类B在运行中达成某种条件的时候想要通知类A，但是不知道方法，而且觉得这个方法还是由类A决定为好，于是就提供给类A一个纸条(代理)，类A要实现代理内容。这个时候类B就是主参考系了。

我是这样理解的，不知道是否正确。但是在UE4之中delegate的出现比较多，之后就用代理来称呼了。

在Blueprint和AnimationBlueprint中可以很容易的实现代理，利用Blueprint自带的**Event Dispatchers**就可以快速的实现。函数绑定的时候需要注意的是要Cast正确的对象。

在UE C++实现代理就稍微有些复杂。但还是很简单的。

## 一些代理之间的区别
在对UE4的代理越来越多的使用之后，对代理这个概念有了深入的了解，或者说是开始一点点接受并认同**代理(delegate)**这个概念的存在了。

关于UE4中的各种类型的代理准备在上面逐一记录使用方法(TODO)，这里主要是用来记录我调查了某种代理之间的使用区别。

### AddDynamic & AddUniqueDynamic
据网上的资料所说，`AddUniqueDynamic`类型的代理绑定了函数之后，相同对象的相同函数被重复绑定也只会被调用一次，而`AddDynamic`则是会被调用绑定次数。

> Consider the following fabricated situations :
>
> Situation 1 :
> Tank->OnTankDeath.AddDynamic(this, &ATankPlayerController::OnTankDeath);
> Tank->OnTankDeath.AddDynamic(this, &ATankPlayerController::OnTankDeath);
>
> Situation 2 :
> Tank->OnTankDeath.AddUniqueDynamic(this, &ATankPlayerController::OnTankDeath);
> Tank->OnTankDeath.AddUniqueDynamic(this, &ATankPlayerController::OnTankDeath);
>
> If you were to then call : OnTankDeath.Broadcast();
>
> For situation 1 , ATankPlayerController::OnTankDeath would be invoked or called twice.
> For situation 2 , ATankPlayerController::OnTankDeath would be invoked or called only once.
>
> Essentially AddUniqueDynamic adds the dynamic delegate only if it isnt already present in the > list of invocable dynamic delegates associated with OnTankDeath.

- [Difference between AddDynamic & AddUniqueDynamic?](https://community.gamedev.tv/t/difference-between-adddynamic-adduniquedynamic/37003)


# 为什么要使用Delegate和Event？
关于代理的实现，就算明白了也需要知道需要在什么情况下使用代理，否则没有意义。

## 关于Event
在UE4中的Blueprint中应没有委托这一说，全部是以Event的名字来称呼的。

## Delegate的使用情况

### 推测1
面向对象的说法只是一种理想的情况，总会有想要实现别人功能的情况。比如说ClassA想要实现一个功能，但是明显这个功能是由ClassB负责的部分，要是自己来实现的话不好，所以自己的话，声明一个Delegate，想用的时候就把这个广播出去(Broadcast)，实现了这接口的内容会被调用。

只言片语：
> 现在我要对一系列数据进行排序，而排序算法可能比较复杂，我不会自己写，我想调用Array.Sort方法，微软为我们提供了快速排序算法。
> 但是这里有一个问题——我要实现自定义排序规则，比如对于字符串，默认的是按字母顺序，但现在我想这样排序：
>
> 按字符串长度排序，只有当长度不同时，再按字母排序。
>
> 显然，微软不可能提供这样“个性”的排序方法，那是不是说，就必须让我们自己去写快速排序算法呢？
> 不需要！
> 我们只需要使用委托，就能实现这个要求：
> string[]strs="I like C# very much".Split();
> Array.Sort(strs,Rule);
> int void Rule(string first,string second)
> {
> return first.Length==second.Length?first.CompareTo(second):first.Length.CompareTo(second.Length);
> }
>
> 显然，我并不需要知道快速排序算法的逻辑，我只需要告之排序规则，就实现了我的个性排序。
>
> 试问：如果没有委托，你如何解决这个问题？

# C++中的代理实现
代理应该涉及了许多知识，完全理解需要后续的更新整理。


参考链接：
- [C++中实现委托（Delegate）](http://www.cppblog.com/huangwei1024/archive/2010/11/17/133870.html)
- [C++实现Delegate Event实例(例子、example、sample)](http://aigo.iteye.com/blog/2301010)
- [C++委托实现(函数指针，function+bind，委托模式)](https://blog.csdn.net/lijun538/article/details/51277199)
- [高效C++委托的原理](https://blog.csdn.net/cyxisgreat/article/details/7506672)
