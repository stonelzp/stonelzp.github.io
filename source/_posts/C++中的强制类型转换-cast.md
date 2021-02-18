---
title: C++中的强制类型转换-cast
date: 2019-02-18 19:20:58
categories:
- C++
tags:
- C++
- 编程
---

实在是遇见太多次了，遇见了还看不懂，再不整理就过分了。关于C++中的强制类型转换问题。这次的主角是:`stati_cast`,`dynamic_cast`,`const_cast`,`reinterpret_cast`。

<!--more-->


## static_cast
提笔要写，先看别人的总结
- [When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?](https://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used)

这里我遇见的情况就是static_cast的重要的用法。继承类型的向下类型转换。下面来看看我遇见的神仙代码：
```C++
template<class T>
class Singleton : NonCopyable
{
public:
	Singleton()
	{
		assert(!m_instance);
		m_instance = static_cast<T*>(this);
	}

	~Singleton()
	{
		assert(m_instance);
		m_instance = 0;
	}

	static T* getInstance()
	{
		return m_instance;
	}

private:
	static T* m_instance;
};

template<class T>
T* Singleton<T>::m_instance = NULL;
```
一个单例模板类，读代码的时候，按照这个模板被实例化的是他的子类，顺便这个子类所具有的instance的引用就变成它自己了。我什么时候也能写出这么优秀的代码。

还有其他的强制类型转换，遇见的时候具体分析吧。

## const_cast
在做各种数据类型匹配的时候，遇到了这样的问题：
```C++
// UE4 C++
FString text = "111";
TCHAR* t_text = *text;
```
上面是想把FString类型转化为TCHAR*类型的数据。

但是上面的会报错，原因是无法将`const THCAR*`转化为`TCHAR*`。那就直接加上关键字const就解决了。但是问题是这个`t_text`是我设置的即将要传入另外一个函数的参数，它要是设置为常量类型的话就没法传参了。

把常量指针变成普通的指针，就是我接触到`const_cast`的起因。但是关于它的使用，可不是一句话就能总结的。

> const_cast实现的原因在于C++对于指针的转换是任意的，它不会检查类型，任何指针之间都可以进行互相转换。

### 去const限定
```C++
const TCHAR* t_text = "1234";
TCHAR* text = const_cast<TCHAR*>(t_text);
```

将常量指针转化为了正常指针，但是正常情况下不会做这种弱智操作。正常声明就行了，但是有的时候要传参的情况传的不是常量怎么办，那只好强制转换了。

**去const限定的操作绝对不是为了修改它的内容**，既然声明了常量还要修改那为什么还要声明为常量。既然声明了常量就要贯彻到底。**绝对不对const数据进行重新赋值**。

根据别人的文章内容，强行修改常量的值会产生*未定义行为（Undefined Behavior）*,这种行为由编译器决定如何处理。
