---
title: EffectiveC++-Chapter01-Accustoming-Yourself-to-C++
date: 2019-01-10 12:16:23
categories:
- Effective C++
tags:
- C++
- 笔记
- 编程
---
Effective C++的第一章内容总结。

<!--more-->

## 条款01：View C++ as a federation of languages.(视C++为一个语言联邦)
- **C**
- **Object-Oriented C++**
- **Templete C++**
- **STL**

> 例如对内置（也就是C-like）类型而言*pass-by-value*通常比*pass-by-reference*高效，但当你从**C part of C++**移往**Object-Oriented C++**，由于用户自定义的（user-defined)构造函数和析构函数的存在，*pass-by-reference-to-const*往往更好。运用Template C++时尤其如此，因为彼时你甚至不知道所处理的对象的类型。然而一旦跨入STL你就会了解，迭代器和函数对象都是在C指针之上塑造出来的，所以对STL的迭代器和函数对象而言，旧式的*C pass-by-value*守则再次适用（参数传递方式的选择细节请见条款20）。

## 条款02：Prefer consts,enums,and inlines to #define.(尽量以const,enum,inline替换#define)
```C++
class GamePlayer {
private:
    enum { NumTurns = 5};  // "the enum hack"补偿作法。以保证NumTurns在编译期间有正确的值

    int scores[NumTurns];  // 本是定义的地方，NumTurns未必会有初值，编译器也许会报错
}

```

> 基于数个理由enum hack值得我们认识。第一，enum hack的行为某方面说较像#define而不像const，有时候这正是你想要的。例如取一个const的地址是合法的，但取一个enum的地址就不合法而取一个#define的地址通常也不合法。
>
> 认识enum hack的第二个理由纯粹是为了实用主义。许多代码用了它，所以看到它是你必须认识它。事实上，“enum hack”是*template metaprogramming*(模板元编程)的基础技术。

```C++
#define CALL_WITH_MAX(a, b) f((a) > f(b) ? (a) : (b))

// 推荐的写法
template<typename T>
inline void callwithmax(const T& a, const T& b)
{
    f(a > b ? a : b);
}
```
使用template inline函数（见条款30）也可以获得类似宏带来的效率以及一般函数所有可预料行为和类型安全性（type safety）。

请记住：
- 对于单纯常量，最好以const对象或enums替换#define
- 对于形似函数的宏（macs），最好改用inline函数替换#define

## 条款03：Use const whenever possible(尽可能使用const)

