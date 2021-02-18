---
title: C++中得到函数名字
date: 2019-01-28 15:22:04
categories:
- C++
tags:
- UE4
- C++
---

应某个要求，需要在运行过程中打印出运行的信息，即所谓的LOG收集，此时希望一起打印出来的内容包含调用的函数信息。

<!--more-->

在Python中就有这种库，直接能得到函数的调信息名字全部打印出来。但是在C++中就没有现成的库。

自己创建一个库？对我来说还是太早了。

## 在C++中得到函数名字
- `__func__`可以得到函数名字。在函数里面调用输出就好。
```C++
FString fun_name = FString(UTF8_TO_CHAR(__func__));

UE_LOG(LogTemp, Log, TEXT("Function name is : %s), *fun_name);
```

- `__FUNCTION__`可以得到类名加函数名。
```C++
FString func_name = FString(UTF8_TO_CHAR(__FUNCTION__));

UE_LOG(LogTemp, Log, TEXT("Function name is : %s), *func_name);
```

以上是在UE4引擎中输出的结果，也有试过`__PRETTY_FUNCTION__`来输出但是出现了编译错误。


参考文章：
- [6.49 Function Name as Strings](https://gcc.gnu.org/onlinedocs/gcc/Function-Names.html)
- [How to get function name in C++](https://codeyarns.com/2018/08/22/how-to-get-function-name-in-c/)

