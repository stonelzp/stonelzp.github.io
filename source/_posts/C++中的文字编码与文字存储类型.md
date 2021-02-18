---
title: C++中的文字编码与文字存储类型
date: 2019-02-14 15:25:34
categories:
- C++
tags:
- C++
---

在准备把函数名字输出的时候遇到了输出乱码的问题。
```C++
TCHAR* text = "DLL内容テスト。";
UE_LOG(LogTemp, Log, TEXT("TEXT: %s"), text);
```

<!--more-->

在我写这段代码之前我连`TCHAR`是什么类型都不知道，经过一番的调查，关于文字编码和文字的数据存储类型，了解到了很多。

## 源码的存储格式
关于源码的存储格式，在我现在的地方，使用的大多是日语系统因此在什么都不动的情况下使用的是日语的默认 **Japanese(Shift-JIS)** 格式。至于怎么确认存储格式的话，在一些编辑器中应该就可以查看。

- 在Visual Studio 2017中改变文件存储格式
    - File -> Save as -> 下面Save旁边的箭头 -> save with encoding -> replace -> Encoding

一般比较喜欢的就是**Unicode(UTF-8)** 就是了，毕竟兼容了英文字母之外的字符。

比如最前面把函数名字输出却遇到了乱码的的方，如果将源码存储为UTF-8格式，就不会出现乱码了。

## C++中的字符存储类型
关于字符的存储问题，首先要分清楚各种字符的存储类型。
#### char
普通的8位字节类型。
#### wchar_t
宽字符类型，表示范围要远大于char类型，表示类型有16位与32位，具体环境具体判断。Unicode编码字符一般以wchar_t类型存储。

为了让编译器识别Unicode字符串，必须在前面加一个**L** :
```C++
wchar_t * text = L"这是中文字符";
```
看到这个`L`，让我想起了现在看的一个日志开源库里的一段糟心的代码：
```C++
#ifdef _WIN32
#   define _PLOG_NSTR(x) L##x
#   define PLOG_NSTR(x)  _PLOG_NSTR(x)
#else
#   define PLOG_NSTR(x)  x
#endif


// Usage
PLOG_NSTR("context");
```
简单来说就是一个把字符串添加L的宏，刚开始看到的时候糟心的不行。这段内容使用Unicode编码的意思，但是这里有一个问题：
- 添加了L也就是说是使用Unicode编码，使用wchar_t数据类型存储。接着上述例子，`PLOG_NSTR`宏接收的如果不是简单的英文字母而是汉字或者日语，简单地输出会正确的输出吗？
- 此时源码的存储格式是否支持Unicode编码会产生影响吗？
    - Shift-JIS格式的源码配合英文字母是能够正确输出的(源码是DLL)
    ```C++
    PLOG_NSTR("line@");

    // output
    line@
    ```
    - Shift-JIS格式的源码配合日文内容也是能够正确输出的(源码是DLL)
    ```C++
    PLOG_NSTR("line内容テスト@");

    // output
    line内容テスト@
    ```
    - 当在UEEditor中实验的时候，Shift-JIS的格式源码没有正确输出。

结论：源码的存储格式应该存储为Unicode格式。在字符编码想要支持Unicode和ANSI两种格式的时候应该添加类似以下的宏
```C++
#ifdef _UNICODE
#define __T(x)    L##x
#define _T(x)     __T(x)
#else
#define _T(x)     x
#endif
```
关于为什么不直接使用添加L的宏的原因是：防止数据变量声明和定义时的类型冲突。使用宏避免char与L的宽字符存储类型冲突。


在tchar.h文件中可以找到关于__T(x)宏的定义，貌似不包含这个文件的时候，自己定义也可以。
#### TCHAR
THAR是对上述两种字符存储类型的统一，参考以下：
```C++
#ifdef UNICODE
typedef wchar_t TCHAR;
#else
typedef char TCHAR;
#endif
```

当程序定义了UNICODE的时候TCHAR就是宽字符存储类型即wchar_t，当未定义的时候就是普通的char数据类型。
## UE4中的字符类型转换
当对字符的存储类型有了一些了解，就来看看他们之间的转换吧。先列出一些参考文章：
- [Charactor Encoding](https://docs.unrealengine.com/en-US/Programming/UnrealArchitecture/StringHandling/CharacterEncoding)


## C++宽字符
当我觉得我能理解并分别和使用C++中的宽字符的时候，现实告诉我还是太天真了。

先看看这篇文章：
- [彻底解密C++宽字符](https://blog.csdn.net/daliang126/article/details/53584395)

写了很多我看不懂的，就是很复杂。我有照着别的示例程序试着写了一下宽窄字符转换，但是英文还好，没看出来什么变化，但是把日语从`const char*` 变成`const wchar_t*`的时候乱码还是乱码。

我使用的是`mbstowcs`函数来实现的，微软的官网文档也有介绍。官方推荐的是使用`mbstowcs_s`这个函数。有机会的话可以再试试。

应该有更深层次的原因，只不过我放弃了。借用4的`FString`和`TEXT()`宏来解决了。








我在plog中看到的宽窄字符转换跟下面的很像说不定好用：
```C++
// Unicode字符集下可用
//--------------------------------------------------------宽字符串转换到窄字符串
char* pC = NULL;   
wchar_t wStr[20] = L"宽字符串";  
int iLen = WideCharToMultiByte( CP_ACP,0,wStr,-1,NULL,0,NULL,NULL); 
if( iLen > 0 )
{
    pC = ( char* )HeapAlloc( GetProcessHeap() ,0 ,iLen );
    if( !pC ) return;
    WideCharToMultiByte( CP_ACP ,0 ,wStr ,-1 ,pC ,iLen ,NULL ,NULL );  
    printf( "%s \n", pC );
    HeapFree( GetProcessHeap() ,0 ,pC );
}
```
```C++
//--------------------------------------------------------窄字符串转换到宽字符串
char cStr[20] = "这是窄字符串";
wchar_t* pWideString = NULL;
int iLenWide = MultiByteToWideChar( CP_ACP ,0 ,cStr ,-1 ,NULL ,0 ); 
if ( iLenWide > 0 )
{
    pWideString = ( wchar_t* )malloc( iLenWide * sizeof(wchar_t) );
    if( !pWideString ) return 0;
    MultiByteToWideChar( CP_ACP ,0 ,cStr ,-1 ,pWideString ,iLenWide ); 
    MessageBox( NULL, pWideString , 0 , 0 );  
    free( pWideString ); 
}
```


