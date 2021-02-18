---
title: 'UE4-ConstructorHelpers::FObjectFinder'
date: 2018-08-10 15:38:38
categories:
- UnrealEngine4
tags:
- UE4
- 编程
---
先来说一说这个问题的起源吧。最近想做一个类似于火焰的特效，是由大量的烟雾粒子组成的那种类似于烟雾的火焰。在制作大量烟雾粒子之前首先要制作出第一个粒子。

<!--more-->

制作一个粒子表现的话首先要先实例化一个Instance，这个实例首先依附在一个Acor上。当然，像Unity的脚本函数一样，在Scene中制作一个Actor然后把脚本当做一个ActorComponent的方案是可行的。然后在脚本中使用`this->GetOwner()`即可以获取到这个Actor。可以使用`this->GetOwner()->GetName()`来获取到Actor的名字。

参考链接：
- [Get actor from component in c++?](https://answers.unrealengine.com/questions/300035/get-actor-from-component-in-c.html)

在这里遇见了一个问题。就是在使用UE的`UE_LOG`打印输出的时候发生了类型错误。
- 在`UE_LOG`中使用的都是基础类型，`%d,%s`等等。
- `GetName()`函数返回的变量类型则是UE的`FString`类型

解决例子：
```c++
// Actor component .cpp file
UE_LOG(LogTemp, Log, TEXT("show value: : %s", *(this->GetOwner()->GetName()));
```
解决案参考：
- [Log issue (passing a FString)](https://answers.unrealengine.com/questions/1903/log-issue-passing-a-fstring.html)
- [Logs, Printing Messages To Yourself During Runtime](https://wiki.unrealengine.com/Logs,_Printing_Messages_To_Yourself_During_Runtime#Log_an_FString.2CInt.2CFloat)

能够获取到Parent的Actor就可以根据自身的Actor来制作InstanceStaticMeshComponent了。

## InstanceStaticMeshComponent


使用方法参照：
- [Using Instanced Static Meshes in C++?](https://answers.unrealengine.com/questions/464719/using-instanced-static-meshes-in-c.html)

下一步就是发生问题的地方，为制作好的InstanceStaticMeshComponent添加StaticMesh。

上网查查资料就有，这篇文章就行
- [QUESTION Apply Static Mesh to StaticMesh component](https://answers.unrealengine.com/questions/13555/question-apply-static-mesh-to-staticmesh-component.html)

照着做本来不会出错的但是我一运行，UE4肯定Crash。

原因就在于题目所提到的关于`ContructorHelpers::FObjectFinder`的使用上。**总结来说就是这个函数只能运行在类的构造函数中，或者构造函数里调用的函数中。**

解决文章：
- [How to use ConstructorHelpers::FObjectFinder?](https://answers.unrealengine.com/questions/246199/how-to-use-constructorhelpersfobjectfinder.html)

这个回答中有提到。之后再整理。
