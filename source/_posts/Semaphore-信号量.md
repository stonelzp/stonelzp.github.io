---
title: Semaphore-信号量
date: 2018-06-28 00:12:02
categories:
- 编程
tags:
- C#
---
Semaphore,中文叫信号量，日语叫セマフォ（= .=）。经常会在多线程的编程中用到。信号量说简单点就是为了线程同步，或者说是为了限制线程能运行的数量。

仔细说明一下就是，信号量会在内部维护一个计数器，当一个线程调用了这个信号量，计数器就会减1，直到计数器减为0，调用这个信号量的线程将会被阻塞，直到有别的线程释放掉一个信号量使其计数器加1。

<!--more-->
那么问题就来了，这个信号量维护的这个计数器应该是对线程的死锁有所防护的，也就是说同一时间只有一个线程能过获取这个信号量，而且当线程获取信号量的时候对信号量中的计数器进行减操作是具有**原子性**的操作。对于这个计数器的保存位置应该深入调查一下。

实例代码：
```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;

namespace Semaphore1
{
    class Program
    {
        //我设置一个最大允许5个线程允许的信号量
        //并将它的计数器的初始值设为0
        //这就是说除了调用该信号量的线程都将被阻塞
        static Semaphore semaphore = new Semaphore(0, 5);

        static void Main(string[] args)
        {
            for (int i = 1; i <= 5; i++)
            {
                Thread thread = new Thread(new ParameterizedThreadStart(work));

                thread.Start(i);
            }

            Thread.Sleep(1000);
            Console.WriteLine("Main thread over!");

            //释放信号量，将初始值设回5，你可以将
            //将这个函数看成你给它传的是多少值，计数器
            //就会加多少回去，Release()相当于是Release(1)
            semaphore.Release(5);
        }

        static void work(object obj)
        {
            semaphore.WaitOne();

            Console.WriteLine("Thread {0} start!",obj);

            semaphore.Release();
        }
    }
}


```
