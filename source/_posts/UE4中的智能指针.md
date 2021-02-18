---
title: UE4中的智能指针
date: 2018-07-31 19:59:19
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
对于内存使用的了解，就不得不了解指针。UE4拥有跟C++类似的智能指针，在这里对虚幻4的智能指针库的内容进行一些总结跟提炼，同时也需要对C++的智能指针进行深入的了解。

<!--more-->

# UE4的智能指针
智能指针并不能使用`UPROPERTY()`，`TSharedRef`,`TSharedPtr`,`TWeakPtr`等等。

上一次更新这篇文章是两年前，都两年了我还没深入了解UE4的智能指针，我也不知道我都干了些什么。

我应该对UE4的智能指针进行分类探讨，根据自己遇到过的情况。

## TWeakPtr


## TWeakObjectPtr
关于`TWeakPtr`和`TweakObjectPtr`之间的区别：

**TWeakObjectPtr**是针对`UObjects`的弱指针引用，**TWeakPtr**则是针对除了`UObjects`的弱指针引用。
> TWeakObjectPtr is for weak pointers to UObjects, TWeakPtr for pointers to everything else.
> 
> Since UObjects are garbage collected and shared pointers are reference counted, we cannot have the same weak pointer type for all, unfortunately.

- [Difference between TWeakPtr and TWeakObjectPtr?](https://answers.unrealengine.com/questions/298868/difference-between-tweakptr-and-tweakobjectptr.html)


参考链接：
- [Unreal Smart Pointer Library](https://api.unrealengine.com/INT/Programming/UnrealArchitecture/SmartPointerLibrary/index.html) 

# C++的智能指针
指针的使用伴随着内存泄漏(memory leak)的问题，可能会发生内存泄漏的情况有：
- new或者malloc出来的内存因为程序员的疏忽忘记释放
- 程序运行发生错误(throw)，未能执行内存释放程序

所以不是说只要程序员足够谨慎就能够避免指针造成的内存泄漏的问题。

## C++11中的智能指针
主要在用的智能指针有：unique_ptr, shared_ptr, weak_ptr。

这3种指针组件就是采用了boost里的智能指针方案。很多有用过boost智能指针的朋友，很容易地就能发现它们之间的关间：
|std|boost|功能说明|
|----|----|----|
|unique_ptr|scoped_ptr|独占指针对象，并保证指针所指对象生命周期与其一致|
|shared_ptr|shared_ptr|可共享指针对象，可以赋值给shared_ptr或weak_ptr。<br>指针所指对象在所有的相关联的shared_ptr生命周期结束时结束，是强引用。|
|weak_ptr|weak_ptr|它不能决定所指对象的生命周期，引用所指对象时，需要lock()成shared_ptr才能使用。|

参考链接：
- [C++11中的智能指针](https://my.oschina.net/hevakelcj/blog/465978)

## 三种智能指针的特性用法

参考链接：
- [C++11及C++14标准的智能指针](https://blog.csdn.net/haolexiao/article/details/56773039)

### weak_ptr
`std::weak_ptr`是一个很好的解决**悬空指针**问题的方式。使用原生指针（raw pointers）的话不知道现在所引用的资源是否已经被释放。而使
用`std::shared_ptr`来管理的话，`std::weak_ptr`只管使用，而不关心资源的使用情况，反正也不管理指向的资源。

因为本身`std::weak_ptr`并不能直接引用到对象，不会影响对象的自动释放，不会影响对象的引用计数，需要使用`lock()`来升级到`std::shared_ptr`来进行操作。

参考资料：
- [C++ weak pointer](https://my.oschina.net/u/2368952/blog/416606)
- [When is std::weak_ptr useful?](https://stackoverflow.com/questions/12030650/when-is-stdweak-ptr-useful)

## 比起直接使用new优先使用std::make_unique和std::make_shared

参考链接：
- [Item 21: 比起直接使用new优先使用std::make_unique和std::make_shared](https://www.cnblogs.com/boydfd/p/5146432.html)


