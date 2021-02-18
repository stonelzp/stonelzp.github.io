---
title: C++-制作一个静态的单一实例
date: 2019-03-01 16:29:22
categories:
- C++
tags:
- C++
- 编程
---

制作单一实例，就会想到Singleton，单例模式。在C++里面如何制作一个唯一的一个实例，我只是有一个想法，然后付诸实践。也许这个想法或许是错的。但是重要的是我将这个想法付诸实践的过程，这个过程中我明白了许多知识点，只是不想忘记而已。

<!--more-->

## 指定需要单例化的类
首先要选一个适合的类进行单一实例化。举例来说我现在正在制作一个日志系统，故而我希望有一个唯一的实例**Logger**来管理关于日志的一切。所以我声明了以下：

```C++
// Logger.h
class Logger
{
    Logger();
    ~Logger();
}

// Logger.cpp
#include "Logger.h"

Logger::Logger(){}

Logger::~Logger(){}
```

像这样子。

## 制作一个单例模板
关于这个单例模板其实我也不太清楚具体是为什么而制作的，只是我在学习[plog]()这个日志库的时候看到就学了。首先它是一个模板，怎么用为什么要这么用，需要我理解模板的使用。

```C++
template<class T>
class Singleton :NonCopyable
{
public:
    Singleton()
    {
        if(m_instance == NULL)
        {
        m_instance = static_cast<T*>(this);
        }
    }

    ~Singleton()
    {
        if(m_instance != NULL)
        {
            m_instance = 0;
        }
    }

    static T* getInstance()
    {
        return m_instance;
    }

private:
    static T* m_instance;
}

template<class T>
T* Singleton<T>::m_instance = NULL;
```

先把出现的`NonCopyable`类的出现放在一边，讨论一下这个模板。

这个模板有一个私有静态属性，类型为**T**，而且在构造函数里面进行了像下类型转换，使用`static_cast，这个算是骚操作么，我也不太清楚。还有注意取得这个属性的指针函数是静态的，也就是有准备把这个单一实例声明为静态的。

总结一句话来说就是我们得到使用了这个模板并实例化成功的静态实例的指针引用。

### 引用
这里让我有些不舒服的是**引用**这个词，按照我所理解的，引用和指针并不是一样的：

**引用（reference）（&）就像能自动的被编译器间接引用的常量型指针。**

关于这一部分我应该是需要用大量的时间去理解的。在*C++编程思想第一卷第十一章的-引用和拷贝构造函数*里面有较为详细的说明。之后也会有提到。

### 静态类成员
关于静态类成员，[Static Members of a C++ Class](https://www.tutorialspoint.com/cplusplus/cpp_static_members.htm) 里面的说明应该很详细了。

要点在于:
- 静态类成员只有一份拷贝，无论类被实例化了多少次。
- 静态类函数只能调用静态类成员。
- 可以使用类公开的静态成员和静态函数，甚至类没有被实例化。
- 类静态成员没有this指针，可以用<类名>::<静态成员>来获取。
- 静态类成员必须在类外初始化

这个是我制作的单例模式的关键理解部分。

## 关于NonCopyable类
这个类可谓是突然出现在我的面前，一查却发现大有来历。就是一个防止类被复制的类。
```C++
class NonCopyable
{
protected:
    NonCopyable(){}

private:
    NonCopyable(const NonCopyable&);
    NonCopyable& operator=(const NonCopyable&);
}
```

这个地方涉及了许多拷贝构造函数相关的东西，*C++编程思想第一卷第十一章-引用和拷贝构造函数*的内容，之后需要好好理解。

## 静态实例实装
经过上面的洗礼，我试着写成下面这样：
```C++
class Logger : public Singleton<Logger>
{
private:
    static Logger logger;

private:
    Logger();
    Logger(const Logger&);
    ~Logger();
}

Logger Logger::logger = Logger();
```

我在类里面声明了一个自身类的静态实例。加上继承来的静态方法可以取到这个静态实例。然后定义了它。

这里有一个问题就是，当我在类外面定义logger的时候，`Logger Logger::logger = Logger();`,编译报错了，因为当时我没有加上`Logger(const Logger&);`这句话。

这句话是什么呢，貌似就是所谓的**拷贝构造函数**。并且被设置成了私有的。

