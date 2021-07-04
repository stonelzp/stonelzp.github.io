---
title: UE4中的异步处理
date: 2018-08-01 15:13:28
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
能够熟练的使用UE4中的异步处理应该能够很好的利用程序运行的资源，和计算。除此之外还有并行的处理。目的是要理清UE4中的线程的同步异步，并行操作和标准C++中的线程同步异步，并行操作。

<!--more-->

# UE4的异步处理的实现
首先从UE4的异步处理实现的部分，当然现在还没有用过，也没有整理~

时间有限，暂时记录下关键字：
- UE C++ ThreadPool
- 异步处理相关函数：
    - Async
        - Lambda记法
    - AsyncTask
    - ParallelFor
    ```
    //函数的位置
    Engine/Source/Runtme/Cre/Pblic/Async/Async.h
    Engine/Source/Runtme/Cre/Pblic/Async/ParallelFor.h
    ```
- 异步辅助API
    - FScopeLock

## 资源的排他处理(FScopeLock)
虽然我上面的关于UE4多线程的使用处理都还没有完成，但是我先遇到了多线程的资源竞争问题，如果说内存上的一块资源是线程共享的，那么多线程操作的时候不加上锁的话数据就会变得很奇怪，老生常谈了。

UE4貌似提供了一个快速为数据加锁的实现方式：**FScopeLock**

使用方法也是很简单
1. 声明一个`CriticalSection`，这又是一个相当重要的概念，但不是UE4专有的。
  ```
  private：
    FCriticalSection Mutex;
  ```
2. 在访问资源的函数内的开头使用`FScopeLock`进行处理
  ```
  // 参考各种参考资料的时候发现有好多种写法
  // Way1：最直接也是参考引擎源码的写法
  func()
  {
    FScopeLock ScopeLock(&Mutex); // 运行到大括号外资源互斥就会失效
    // FScopeLock ScopeLock(Mutex) // 或者这种写法，但是UE4引擎中没有看到这种值传递的方式
    ...
  }
  // Engine\Source\Runtime\CoreUObject\Private\UObject\Obj.cpp 中的源码
  bool IsExcluded(UClass* InClass)
  {
    // ...
    FScopeLock ScopeLock(&ExclusionListCrit); // CriticalSection声明的都一样的方式
    // ...
  }

  // Way2:
  func()
  {
    Mutex.Lock();
    // 这里的资源是线程互斥的

    Mutex.UnLock();
  }
  ```

关于线程资源排他(互斥)处理的方式还有其它方式，`FThreadSafeCounter`,`FThreadSafeBool`等，虽然我完全是第一次听说。

参考资料：
- [UE4 非同期処理についてのメモ](https://qiita.com/unknown_ds/items/72c910888877995b3a85)
- [MultiThreading and synchronization Guide](https://michaeljcole.github.io/wiki.unrealengine.com/MultiThreading_and_synchronization_Guide/)

上面文章基本没看...需要找时间验证多线程处理的实现。
不过说实话，这里的资源互斥处理，我只是照着写，也不知道该怎么验证。

# 拓展内容

## CriticalSection
关于这个概念需要找一篇文章好好理解了。
