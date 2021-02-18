---
title: new与malloc的区别
date: 2019-03-12 13:18:31

categories:
- C++
- C

tags:
- C++
- C
- 编程
---

new/delete与malloc/free显而易见的区别是，前者会调用对象的构造/析构函数，而后者不会。

<!--more-->

再仔细说明就是new的操作是新建*对象* ，而malloc只是分配一块内存而已。

在Stack Overflow上有更为详尽的说明，有时间的话好好整理一下。
- [What is the difference between new/delete and malloc/free?](https://stackoverflow.com/questions/240212/what-is-the-difference-between-new-delete-and-malloc-free)
