---
title: C++中的各种文件操作
date: 2019-03-15 12:42:01

categories:
- C++
- C

tags:
- C++
- C
- 编程
---

说到LOG的制作那么文件读写就肯定有大费周折了，当然不是我大费周折，我用的人家现成的东西。但是读起来其实挺费劲的，谁让我这么菜呢。

<!--more-->

## 打开文件

### _sopen_s, _wsopen_s
打开文件以供共享。

参考官方文档：
- [_sopen_s,_wsopen_s](https://docs.microsoft.com/zh-cn/cpp/c-runtime-library/reference/sopen-s-wsopen-s?view=vs-2017)

## 操作文件

### _lseek
将文件指针移动到指定位置

官方文档：
- [_lseek, _lseeki64](https://docs.microsoft.com/zh-cn/cpp/c-runtime-library/reference/lseek-lseeki64?view=vs-2017)

### _write
将数据写入文件

官方文档：
- [_write](https://docs.microsoft.com/zh-cn/cpp/c-runtime-library/reference/write?view=vs-2017)

## 关闭文件

### _close
关闭文件。

关于这些函数的用法在我最近读的plog里面都有，有空再仔细整理一下吧。

还有一些其他的用法调查一下就知道了，想了想花时间写一下也好但是收益不大，暂时先记下吧。

## errno

