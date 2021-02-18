---
title: UE4-正确使用LOG
date: 2020-11-29 17:08:28
permalink: how-to-use-log
categories:
- UnrealEngine4
tags:
- UE4
---

关于UE4中的LOG的使用我一直都是经常使用，但是也只是只用了最常见的`UE_LOG`，这篇文章注意主要想集中记录UE4中的LOG的使用和日志文件的问题。

<!--more-->

# UE4中的LOG
在想要在游戏中使用LOG的时候，首先需要知道的是UE4的LOG可以做到哪一步。

首先需要记录的UE4中的LOG。

## 自定义LOG的Category
参考文章：
- [Unreal Engine 4 - Custom Log Categories](https://blog.jamie.holdings/2020/04/21/unreal-engine-4-custom-log-categories/)

自定义LOG的步骤有三步：声明-定义-使用。就这么简单。

头文件中声明：
```c++
DECLARE_LOG_CATEGORY_EXTERN(LogMyAwesomeGame, Log, All);
```

CPP文件中定义：
```c++
DEFINE_LOG_CATEGORY(LogMyAwesomeGame);
```

使用：
```c++
UE_LOG(LogMyAwesomeGame, Log, TEXT("Test Log Message"));
```

然后是对声明中出现的参数进行说明。

第一个参数当然我们新定义的LOG类别的名称。
### 第二个参数：Log

第二个参数则是为这个种类的LOG定义了输出等级，UE4中包含了以下的几个LOG输出等级：
- Fatal
- Error
- Warning
- Display
- Log
- Verbose
- VeryVerbose 

像上面，如果你定义了Log类型，则Log以上包含Log的输出等级的Log都会被输出。

额外的，如果你想提升LOG的输出等级，只是为了Debug，我们可以在不修改代码改为修改设定文件的方式对该类型的默认LOG输出层级进行重写：
```
[Core.Log]
LogMyAwesomeGame=VeryVerbose
```

### 第三个参数：All
第三个参数则是决定了编译阶段会被编译进去的LOG层级，这意味着在Runtime对Log的输出层级进行修改变为不可能，当然是指修改为未被编译进去的LOG层级。

为什么会需要这个参数？原因就是对于我们不想输出的层级仍然需要进行比是否要输出该层级，索性直接将其从编译中移除更彻底。

按照文章的内容所说一个大量的日志输出，哪怕并不会输出到日志上，但是决定这些是否要输出日志这件事本身就很慢了。

**All**相当于**VeryVerbose**的缩写，效果是相同的。

## UE4的LOG输出时间
我一直以为UE4的Log没办法输出时间，实际上只是我没有调查而已...

`Preferences -> General ->  Apppearance -> Log Timestamp`选择类型就可以输出时间了。

## UE4的LOG输出日志文件
由于我没有调查过，UE4的日志输出文件应该也有功能，有时间一定要调查一下，或者说有需求着手之前一定要调查。

# 使用UE4之外的手段输出LOG
即我之前傻傻的未调查UE4的LOG系统可以做到什么程度，直接使用[plog](https://github.com/SergiusTheBest/plog)来进行Log日志的保存。但是我觉得效果还是一样的。

至少不用每次都写UE4那个看起来就长的输出用的LOG宏`UE_LOG(xxx)`。

等我有兴致再整理吧。

# 使用UE_LOG输出各种数据类型
在UE4的使用中真的不可避免的使用LOG输出，使用VS的Deebug模式也是很方便而且更准确，但是根据Debug模式的不同，有些地方编译器会进行优化设置断点也未必能取到有效的数据。

这篇算是总结和记录`UE_LOG`宏的各种数据类型的输出样例，方便我以后查看。

## FString
```c++
UE_LOG(LogTemp, Log, TEXT("output message %s"), *(FDateTime::Now().ToString()));
```
这个我在其它的文章里也有一点记录，算是重复了。

## FName
```c++
FName MyName;
 UE_LOG(LogTemp, Warning, TEXT("My Name: %s"), *MyName.ToString());
```