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

# 一些常见的修饰符

## explicit 禁止隐式类型转换
标题只是最常用的用法，参考下面网址的描述:
1. 指定构造函数或转换函数 (C++11 起)或推导指引 (C++17 起)为显式，即它不能用于**隐式转换**和**复制初始化**。
2. `explicit`说明符可以与常量表达式一同使用。当且仅当该常量表达式求值为`true`时函数为显式。
(C++20 起)

explicit 说明符只可出现于在**类定义之内的构造函数**或**转换函数**(C++11 起)的*声明说明符序列* 中。

※*声明时不带函数说明符 explicit 的拥有单个无默认值形参的 (C++11 前)构造函数被称作转换构造函数。*
※*构造函数（除了复制/移动）和用户定义转换函数都可以是函数模板；explicit 的含义不变。*

上面的概念还是很复杂，直接把下面网址上的示例代码复制过来了：
```
struct A
{
    A(int) { }      // 转换构造函数
    A(int, int) { } // 转换构造函数 (C++11)
    operator bool() const { return true; }
};

struct B
{
    explicit B(int) { }
    explicit B(int, int) { }
    explicit operator bool() const { return true; }
};

int main()
{
    A a1 = 1;      // OK：复制初始化选择 A::A(int)
    A a2(2);       // OK：直接初始化选择 A::A(int)
    A a3 {4, 5};   // OK：直接列表初始化选择 A::A(int, int)
    A a4 = {4, 5}; // OK：复制列表初始化选择 A::A(int, int)
    A a5 = (A)1;   // OK：显式转型进行 static_cast
    if (a1) ;      // OK：A::operator bool()
    bool na1 = a1; // OK：复制初始化选择 A::operator bool()
    bool na2 = static_cast<bool>(a1); // OK：static_cast 进行直接初始化

//  B b1 = 1;      // 错误：复制初始化不考虑 B::B(int)
    B b2(2);       // OK：直接初始化选择 B::B(int)
    B b3 {4, 5};   // OK：直接列表初始化选择 B::B(int, int)
//  B b4 = {4, 5}; // 错误：复制列表初始化不考虑 B::B(int,int)
    B b5 = (B)1;   // OK：显式转型进行 static_cast
    if (b2) ;      // OK：B::operator bool()
//  bool nb1 = b2; // 错误：复制初始化不考虑 B::operator bool()
    bool nb2 = static_cast<bool>(b2); // OK：static_cast 进行直接初始化
}
```
我在EffectC++那个书里也有看到过这个关键字的使用说明，说不定以后有时间会整理那本书的内容作为笔记。

- [explicit说明符](https://zh.cppreference.com/w/cpp/language/explicit)

等待整理：

偶然看见的:

- [explicitとvolatileキーワード(C++)](http://c-crad.wktk.so/td/?p=250)

- [Typedefの考え方](https://qiita.com/aminevsky/items/82ecce1d6d8b42d65533)

除了关键字，符号也算
- [*p++のお話(インクリメント演算子って不思議だね](https://qiita.com/1024chon/items/f17ee5afc6644cfd33f1)

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
