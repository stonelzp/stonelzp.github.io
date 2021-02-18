---
title: C++中的常见关键字记录
date: 2019-05-22 20:04:00
permalink: c++-modifier-keyword

categories:
- C++

tags:
- C++
- 编程

description: 经常会遇见一些不常遇见的修饰符，记录在这里。

---

之后需要整理并理解的:

- [explicit说明符](https://zh.cppreference.com/w/cpp/language/explicit)

偶然看见的:

- [explicitとvolatileキーワード(C++)](http://c-crad.wktk.so/td/?p=250)

- [Typedefの考え方](https://qiita.com/aminevsky/items/82ecce1d6d8b42d65533)

除了关键字，符号也算
- [*p++のお話(インクリメント演算子って不思議だね](https://qiita.com/1024chon/items/f17ee5afc6644cfd33f1)

哇这篇文章根本没怎么更新呢..

这一天我在公司的项目中看到了大佬使用了`*&`这个运算符，没用过这啊，看看什么意思
- [Meaning of *& and **& in C++](https://stackoverflow.com/questions/5789806/meaning-of-and-in-c)


# 指针与引用

关于指针和引用这一块儿我不明白的地方有很多，可以简单验证的方法有很多，我就是懒。

## * 与 & 的区别
- [What's the difference between * and & in C?](https://stackoverflow.com/questions/28778625/whats-the-difference-between-and-in-c/28778902)
这里是这两种关键字出现在函数的参数修饰符的位置。关于作为函数的参数，有什么区别的问题：
> `funct(int a)`
> 
> Creates a copy of a
>
> `funct(int * a)` 
> 
> Takes a pointer to an int as input. But makes a copy of the pointer.
> 
> `funct(int& a)`
> 
> Takes an int, but by reference. a is now the exact same int that was given. Not a copy. Not a pointer.

还有其他的答案也很好，有一些我不知道的知识点，之后整理。

