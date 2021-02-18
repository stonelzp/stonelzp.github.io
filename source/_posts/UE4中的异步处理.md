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
