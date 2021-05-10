---
title: C++中的函数指针理解
date: 2021-01-10 23:20:23
permalink: c++-function-pointer
categories:
- C++
tags:
- C++
- UE4
---
学会如何辨别和使用函数指针，还有与模板之间的应用。
<!--more-->

当遇见需要大面积使用某种操作，大体上需要的操作都是一样的，但是总有一些数据类型或者条件有些细微的不同，让我不能简单实现批次处理，但是一个一个进行复制粘贴调整费时还费力，而且犯错的几率非常高，这个时候求助大佬的时候，简单的使用了C++的template，函数指针之类的操作完成了这个需求，看得我心痒的不行。

关于模板的内容我记得还在博客的别的文章中总结学习过，然而事实证明不是真正的使用就根本学不会。

这次准备实战进行学习。

首先是基础。

# C++中使用std::function
这个地方多亏了一篇文章的总结，十分感谢：
- [std::functionの使い方 in C++](https://qiita.com/ymd_/items/27009e3e6d7a73653fab)

首先是`std::function`的功能是可以包裹(Wrap)多数种类的C++函数类型方便进行统一的处理。这里多数种类的函数包括
- 函数
- Lambda表达式
- 函数Object
- 类的成员函数

这里就引出了我之前没怎么意识到的问题，那就是，**什么是函数？**

如果说上面的函数种类都被称为函数的话，那么**什么是函数指针？**

这是我在使用的过程中一直没有搞清楚的地方。希望在整理这篇文章的过程中会对这个概念有个完全的理解。

## Lambda表达式
关于Lambda表达式有很多内容，之后慢慢更新，这里先参照我看的资料记住Lambda表达式的基础用法。
```c++
[キャプチャー](引数)->返り値の型{関数の内容}(実行時の引数);
```

翻译过来就是
```c++
[Capture](参数)-> 返回值类型 {函数体的内容} (运行时参数)
```

这里Capture中的内容可以是`&`和`=`，还有什么都不写。`&`就是通常的引用的意思，`=`就是复制来自Scope外的参数。**至于什么都不写的情况下，是哪种方式我暂时不太清楚**。

甚至可以这样写：`[]{}()`省略参数和返回值。

再就是最后的`(运行时参数)` 是个什么使用方法我还暂时不清楚，没遇到过。
难不成只是函数的`()`操作符，就像是`Get(运行时参数)`，这里就是`Lambda表达式(运行时参数)`这种感觉。

## std::function使用方法
```c++
std::function< 戻り値の方(引数の型) > object = 関数or ラムダ式 or 関数オブイェクト orクラスのメンバ関数;
object(引数);で利用できる
```

翻译过来就是：
```c++
std::function< 返回值类型 (参数类型) > object = 函数 or Lambda表达式 or 函数Object or 类成员函数;

object(参数); // 函数调用
```

### 普通的函数
使用`std::function`来操作最简单最常见的函数：
```c++
void print_number(const int i)
{
    std::cout << i << std::endl;
}

int mai()
{
    std::function< void(int) > f_func = print_number;
    f_func(100);
}
```
这个是最简单也是最好理解的用法。

### Lambda表达式
举个使用`std::function`操作lambda表达式的例子：
```c++
void print_number(const int i)
{
    std::cout << i << std::endl;
}

int main()
{
    std::function< void (int) > f_lambda = [=](int i){ print_number(i);};

    f_lambda(100);
}
```

### bind
这个这个使用方式是我几乎没有见到和使用的，记着没有坏处，应该是把函数的地址和参数地址绑定到一起？
```c++
void print_number(const int i)
{
    std::cout << i << std::endl;
}

int main()
{
    std::function < void() > f_bind = std::bind(print_number, 100);

    f_bind();
}
```
这个函数绑定感觉有意思。

### 类成员函数
一种可以表示类的成员函数的方法
```c++
struct Foo
{
    Foo(const int n) : i_(n) {}
    void print_add(const int n) const { std::cout << i_ + n << std::endl; }
    int i_;
}

int main()
{
    std::function< void(const Foo&, int) > f_member = &Foo::print_add;
    Foo foo(1);

    f_member(foo, 100);
}
```

### 函数Object
这个方法是我第一见到，这个概念说实话我也是第一次遇到，先上用法：
```c++
struct PrintFunctor
{
    void operator()(int i)
    {
        std::cout<< i << std::endl;
    }
}

int main()
{
    std::function< void(int) > f_func_obj = PrintFunctor();

    f_func_obj(100);
}
```

这个的使用是我再请教大佬的是时候遇到的，再学会使用之前，需要搞懂**函数Object**这个概念。














