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

# UEC++中的Delegate实现
上面的内容对于UE4/UE5中的代理（Delegate）这个概念有些大概的了解了。但是再UnrealEngine中，提供了一些类型的代理，什么时间用什么样的代理如果不深入理解的话，虽然可以一招鲜吃遍天但是未必太单调了。
而且为什么需要根据场景选择不同的代理的理由会在后面进行说明和佐证。

首先是官方文档：
- [Delegates - Data types that reference and execute member functions on C++ Objects](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/)

文档的干货说明：
- **Delegates can call member functions on C++ objects in a generic, type-safe way.** ：代理的调用时线程安全的。
- **Delegates are safe to copy. whenever possible, pass delegates by reference.** ：代理可以复制，代价是需要再Heap中申请内存，所以尽可能的使用引用。


UE引擎中提供了大致三种类型的代理：
- Single
- Multicast
  - Events
- Dynamic(UObject,serializable)

**UE的代理都是对Object的弱引用。**Dynamic的代理不敢确定。

代理的作用域(Scope)可以存在于:
- Global Scope
- namespace
- class

下面就对这三种代理类型进行说明。

## Delegates - Single
Single，也是通常的代理类型。通常的代理类型可以拥有以下几种特征：
- **Functions returning a value**：Delegate可以拥有返回值
- **Functions declared as `const`**：Delegate的对象可以是`const`函数
- **Up to four "payload" variables**：至多4个"payload"变量
- **Up to eight function parameters**：支持最多8个函数参数

### Single代理的声明
一般的写法(参考官方文档)：

| Function signature | Declaration macro |
| ---- | ---- |
| void Function() | DECLARE_DELEGATE(DelegateName) |
| void Function(Param1) | DECLARE_DELEGATE_OneParam(DelegateName, Param1Type) |
| void Function(Param1, Param2) | DECLARE_DELEGATE_TwoParams(DelegateName, Param1Type, Param2Type) |
| void Function(Param1, Param2, ...) | DECLARE_DELEGATE_<Num>Params(DelegateName, Param1Type, Param2Type, ...) |
| <RetValType> Function() | DECLARE_DELEGATE_RetVal(RetValType, DelegateName) |
| <RetValType> Function(Param1) | DECLARE_DELEGATE_RetVal_OneParam(RetValType, DelegateName, Param1Type) |
| <RetValType> Function(Param1, Param2) | DECLARE_DELEGATE_RetVal_TwoParams(RetValType, DelegateName, Param1Type, Param2Type) |
| <RetValType> Function(Param1, Param2, ...) | DECLARE_DELEGATE_RetVal_<Num>Params(RetValType, DelegateName, Param1Type, Param2Type, ...) |

关于Delegate的一个就是属性修饰符，我真的是没见过，只在官方文档里见到了
```
UDELEGATE(BlueprintAuthorityOnly)
DECLARE_DYNAMIC_MULTICAST_DELEGATE_FourParams(FInstigatedAnyDamageSignature, float, Damage, const UDamageType*, DamageType, AActor*, DamagedActor, AActor*, DamageCauser);
```
这个`UDELEGATE`有时间我得调查一下。

### Single代理的绑定

| Function | Description |
| ---- | ---- |
| Bind | Binds to an existing delegate object. |
| BindStatic | Binds a raw C++ pointer global function delegate. |
| BindRaw | Binds a raw C++ pointer delegate. Since raw pointers do not use any sort of reference, calling `Execute` or `ExecuteIfBound` after deleting the target object is unsafe. |
| BindLambda | Binds a functor. This is generally used for lambda functions. |
| BindSP | Binds a shared pointer-based member function delegate. Shared pointer delegates keep a weak reference to your object. You can use `ExecuteIfBound` to call them. |
| BindUObject | Binds a `UObject` member function delegate. `UObject` delegates keep a weak reference to the `UObject` you target. You can use `ExecuteIfBound` to call them. |
| UnBind | Unbinds this delegate. |

使用Delegate的例子<span style="color: red">（随时补全）</span>：


关于Delegate的**PayLoad Data**，这又是一个我未曾见过的写法，也就是说在Bind的阶段可以传递Payload数据
> When binding to a delegate, you can pass payload data along. These are arbitrary variables that will be passed directly to any bound function when it is invoked. This is really useful as it allows you to store parameters within the delegate itself at bind-time. All delegate types (except for "dynamic") supports payload variables automatically.
>
> This example passes two custom variables, a bool and an int32 to a delegate. Then when the delegate is invoked, these parameters will be passed to your bound function. The extra variable arguments must always be accepted after the delegate type parameter arguments.
> ```
> MyDelegate.BindRaw( &MyFunction, true, 20 );
> ```

我暂时没有找到具体的使用案例<span style="color: red">（随时补全）</span>。

### Single代理执行
由于对未绑定的代理执行`Execute()`可能会造成未定义的程序行为（比如说未被初始化的成员变量被访问到），需要在使用之前`IsBound()`进行判断。
对于没有返回值的代理可以使用`ExecuteIfBound()`执行。
**代理的使用的时候需要注意避免对未初始化的变量进行访问。**

| Execution Functions | Description |
| ---- | ---- |
| Execute | Executes a delegate without checking its bindings |
| ExecuteIfBound | Checks that a delegate is bound, and if so, calls Execute |
| IsBound | Checks whether or not a delegate is bound, often before code that includes an `Execute` call |

## Delegate - Multicast
官方文档
- [Multi-cast Delegates - Delegates that can be bound to multiple functions and execute them all at once.](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Multicast/)

**Multicast类型的代理不可使用返回值**，特性大致上都与Single类型相同。使用上通常是为了方便传递一个代理的集合。

### Multicast代理声明
Multicast类型分为两种，基于Dynamic类型的特殊性，分为第三类型的代理，将在下一个章节部分进行说明。

| Declaration Macro | Description |
| ---- | ---- |
| DECLARE_MULTICAST_DELEGATE( DelegateName ) | Creates a multi-cast delegate. |
| DECLARE_MULTICAST_DELEGATE_<Num>Params( DelegateName, Param1Type, Param2Type, ... ) | Creates a multi-cast delegate with N parameters. |

### Multicast类型代理的绑定
Multicast的绑定语义上类似与数组。

| Function | Description |
| ---- | ---- |
| Add() | Adds a function delegate to this multi-cast delegate's invocation list. |
| AddStatic() | Adds a raw C++ pointer global function delegate. |
| AddRaw() | Adds a raw C++ pointer delegate. Raw pointer does not use any sort of reference, so may be unsafe to call if the object was deleted out from underneath your delegate. Be careful when calling Execute()! |
| AddSP() | Adds a shared pointer-based (fast, not thread-safe) member function delegate. Shared pointer delegates keep a weak reference to your object. |
| AddUObject() | Adds a UObject-based member function delegate. UObject delegates keep a weak reference to your object. |
| Remove() | Removes a function from this multi-cast delegate's invocation list (performance is O(N)). Note that the order of the delegates may not be preserved! |
| RemoveAll() | Removes all functions from this multi-cast delegate's invocation list that are bound to the specified UserObject. Note that the order of the delegates may not be preserved! |

关于`RemoveAll()`这个函数在使用**Raw delegate**有一点需要注意：
> `RemoveAll()` will remove all registered delegates bound to the provided pointer. Keep in mind that Raw delegates that are not bound to an object pointer will not be removed by this function!

### Multicast类型代理执行
我们可以安全的使用`Broadcast()`执行这个类型的代理，即使它没有任何的绑定。Multicast代理的执行顺序是没有保障的，不要想当然的认为是绑定的顺序。

| Declaration Macro | Description |
| ---- | ---- |
| Broadcast() | Broadcasts this delegate to all bound objects, except to those that may have expired. |

关于官方文档的：
> The only time you need to be careful is if you are using a delegate to initialize output variables, which is generally very bad to do.

这句话我很在意，但是我又搞不明白他说的具体是什么样的情况。

## Delegate - Events
官方文档
- [Events - Delegates that can be bound to multiple functions and execute them all at once.](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Events/)

Events类型的代理是一种类型特化的Multicast代理类型。也就是说只有该代理声明的类的类型才能使用该代理。这减少了该对象暴露在外面世界的时候内部的一些比较敏感的代理实现不会被指定类之外的对象调用。

### Events类型代理声明
`OwningType`的声明就是指定该代理从属的类，只有指定的类才能调用`Broadcast()`。

| Declaration Macro | Description |
| ---- | ---- |
| DECLARE_EVENT( OwningType, EventName ) | Creates an event. |
| DECLARE_EVENT_OneParam( OwningType, EventName, Param1Type ) | Creates an event with one parameter. |
| DECLARE_EVENT_TwoParams( OwningType, EventName, Param1Type, Param2Type ) | Creates an event with two parameters. |
| DECLARE_EVENT_<Num>Params( OwningType, EventName, Param1Type, Param2Type, ... ) | Creates an event with N parameters. |

### Events类型代理绑定
与Multicast类型的代理绑定一样。此处省略。

### Events类型代理执行
与Multicast类型的代理执行一样。此处省略。



## Delegate - Dynamic
官方文档
- [Dynamic Delegates - Delegates that can be serialized and support reflection.](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/Delegates/Dynamic/)

Dynamic类型的代理数据会被序列化(serialized)，可以通过函数名找到。也就是说它们的执行效率要比普通的代理慢许多。

本质上Dynamic代理类型不是独立于Single/Multicast代理类型的第三个类型，而是基于其特殊性放在第三个位来说。
Single/Multicast类型代理分别有着**General**与**Dynamic**的差别。

### Dynamic类型代理的声明
官方文档的写法貌似是Dynamic类型代理不支持返回值的写法，但是我也不完全确定。

| Declaration Macro | Description |
| ---- | ---- |
| DECLARE_DYNAMIC_DELEGATE( DelegateName ) | Creates a dynamic delegate. |
| DECLARE_DELEGATE_<Num>Params( DelegateName , Param1Type, Param2Type, ...) | Creates a dynamic delegate with N parameters. |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE( DelegateName ) | Creates a dynamic multi-cast delegate. |
| DECLARE_DYNAMIC_MULTICAST_DELEGATE_<Num>Params( DelegateName, Param1Type, Param2Type, ... ) | Create a dynamic multi-cast delegate with N parameters |

### Dynamic类型代理的绑定

| Helper Macro | Description |
| ---- | ---- |
| BindDynamic( UserObject, FuncName ) | Helper macro for calling BindDynamic() on dynamic delegates. Automatically generates the function name string. |
| AddDynamic( UserObject, FuncName ) | Helper macro for calling AddDynamic() on dynamic multi-cast delegates. Automatically generates the function name string. |
| RemoveDynamic( UserObject, FuncName ) | Helper macro for calling RemoveDynamic() on dynamic multi-cast delegates. Automatically generates the function name string. |

### Dynamic类型代理的执行
执行方式与各自Single/Multicast类型代理的执行方式相同。此处省略。

关于各项代理的源码实现可以直接参考引擎的源码：
UE4/UE5：`Engine\Source\Runtime\Core\Public\Delegates\DelegateSignatureImpl.inl`

## Delegate的Sample实现
未必避免理解变得困难，于是决定没有把各自代理的例子放在上面章节，而是独立设置了一个章节。

### Events代理的sample
这一段是来自官网上的
```
public:
/** Broadcasts whenever the layer changes */
DECLARE_EVENT( FLayerViewModel, FChangedEvent )
FChangedEvent& OnChanged() { return ChangedEvent; }
```

```
private:
/** Broadcasts whenever the layer changes */
FChangedEvent ChangedEvent;
```

官网这里需要注意的是下面的一个TIP：
> Accessors for events should follow an OnXXX pattern instead of the usual GetXXX pattern.

我一直觉得把代理的变量直接放在Public或者Protected有一些抵触，刚好以后就参考这种写法了。

#### 抽象Events代理
关于这个**Inherited Abstract Event**也是官方文档的内容

**Base Class Implementation:**
```
/** Register/Unregister a callback for when assets are added to the registry */
DECLARE_EVENT_OneParam( IAssetRegistry, FAssetAddedEvent, const FAssetData&);
virtual FAseetAddedEvent& OnAssetAdded() = 0;
```

**Derived Class Implementation:**
```
DECLARE_DERIVED_EVENT( FAssetRegistry, IAssetRegistry::FAssetAddedEvent, FAssetAddedEvent);
virtual FAssetAddedEvent& OnAssetAdded() override { return AssetAddedEvent; }
```
在派生类中声明派生的Events代理的时候，貌似函数的语言不需要重复定义了(我理解的函数参数的数量和类型)，使用`DECLARE_DERIVED_EVENT`宏就足够了，该宏的第三个参数是定义该Events代理的新名字，一般跟父类的名字相同就可以了。

这种写法的话我能想到的就是开放子类也可使用这个Events代理，只不过需要自己实现罢了。
（只不过需要自己实现罢了，可以约束子类一定要实现这个代理？）


参考链接：
- [C++中实现委托（Delegate）](http://www.cppblog.com/huangwei1024/archive/2010/11/17/133870.html)
- [C++实现Delegate Event实例(例子、example、sample)](http://aigo.iteye.com/blog/2301010)
- [C++委托实现(函数指针，function+bind，委托模式)](https://blog.csdn.net/lijun538/article/details/51277199)
- [高效C++委托的原理](https://blog.csdn.net/cyxisgreat/article/details/7506672)
