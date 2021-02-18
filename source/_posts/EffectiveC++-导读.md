---
title: EffectC++-导读
date: 2019-01-07 11:21:44
categories:
- Effective C++
tags:
- C++
- 笔记
- 编程
---

Effective C++ 的导读章节的内容整理，知识点备忘吧。应该会将以后的章节归档到一起。

<!--more-->

## 术语（Terminology)
关于*声明*与*定义*应该有了初步的认识，在其他的C++学习章节有提到过这之间的区分。

> 定义式(definition)的任务是提供编译器一些声明式所遗漏的细节。对对象而言，定义式是编译器为此编译器拨内存的起点。

这里有一个对于初始化(initialization)的default构造函数的认知误区：
- default构造函数是一个可被调用而不带任何实参
- 这样的构造函数要不没有参数，要不就是每个参数都有缺省值

**不是说构造函数就是没有参数的。**

### explicit关键字
应用场景：
```C++
class B{
public:
    explicit B(int x = 0, bool b = true);
};

class C{
public:
    explicit C(inx x);
};
```

在构造函数的前面加上explicit关键字可以防止被用来执行*隐式类型转换(implicit type conversions)*，但是仍可以被用来进行*显式类型转换(explicit type cnversions)*.

比如说传参的时候，参数应该是一个类型B的对象
```C++
void doSomething(B bObject);

B bObj1;

doSomething(28);   // 错误，int跟B之间没有隐式类型转换

doSomething(B(28));  // 正确，有显式转型，即cast
```

虽然上述的使用是我之前知道的但是不知道为什么是正确的。为了防止构造函数被隐式类型转换，把构造函数声明为explicit是一个好的选择。

### copy构造函数和copy assignment操作符
- copy构造函数被用来“以同型对象初始化自我对象”
- copy assignment操作符被用来“从另一个同型对象中拷贝其值到自我对象”

```C++
class Widget {
public:
    Widget();                                 // 默认构造函数
    Widget(const Widget& rhs);                // copy构造函数
    Widget& operator=(const Widget& rhs);     // copy assignment操作符
    ...
};

Widget w1;         // 调用默认构造函数
Widget w2 = w1;    // 调用copy构造函数
w1 = w2;           // 调用copy assignment操作符
```

上述的代码应该足够说明用法，但是需要注意的是看见`=`的时候要注意：
```C++
Widget w3 = w2;    // 调用copy构造函数
```

> 幸运的是“copy构造函数”很容易个“copy赋值”有所区别。如果一个对象被定义（例如以上语句中的w3)，一定会有一个构造函数被调用，不可能调用赋值操作。如果没有新对象被定义（例如前述的“w1=w2”语句），就不会有构造函数被调用，那么当然是赋值操作被调用。


### STL


### TR1和Boost
[Boost](https://www.boost.org)
