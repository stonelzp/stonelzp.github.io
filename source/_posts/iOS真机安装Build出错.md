---
title: iOS真机安装Build出错
date: 2018-06-08 22:27:08
categories: 备忘
tags: 
- Bugs
- iOS
---
为了面试，需要用尽一切手段了，把自己的毕业设计的时候做的2D游戏也拿出来了（捂脸）。找到了自己之前保存好的已经编译好的安装包，希望能够平安无事的安装到自己的系统高版本的iPhone7上。但是再次编译的时候果不其然还是要报错。
```swift
 '__declspec' attributes are not enabled; use '-fdeclspec' or '-fms-extensions' to enable support for __declspec attributes
```

<!--more-->

但是无论是Xcode还是Unity都已经升级的面目全非了，打开以前保存的Unity工程果不其然各种报错。想要解决出现的问题，第一个是NGUI的版本问题，代码中出现的一部分说是已经不支持Unity5.4+了...上网搜了一下，结果没有找到任何解答。

第一下就碰壁，只能再回去找已经编译好的文件的Build错误了。

## 问题解决
继续搜关键词，看到了Unity社区中有一个这个提问：[Error "unknown type name __declspec" after Xcode 7.3 upgrade](https://forum.unity.com/threads/error-unknown-type-name-__declspec-after-xcode-7-3-upgrade.393128/)

出问题的代码部分跟我的是一样的:
```swift
NORETURN static void il2cpp_codegen_raise_exception (Il2CppCodeGenException *ex)
{
    il2cpp::vm::Exception::Raise ((Il2CppException*)ex);
#if __has_builtin(__builtin_unreachable)
    __builtin_unreachable();
#endif
}
```
当然我是根本不知道这几行代码到底是干什么的。但是就是出错了...继续往下看，好像看到了一个解决方案:

> It has helped me:
>
> 1) Remove 'NORETURN'
> 2) Clean build
> 3) Build it
>
> Like a hack:)
>
> XCode7.3, Unity5.2.2
>
> UPD: XCode7.3, Unity5.3.4f1 - no issue

跟我的情况太像了，抱着试一试的态度，然后...Build通过了，游戏也成功的安装到了我的手机上了。

で？这个`NORETURN`到底是个什么？

查了一下发现这个属性不光是`Swift`中的，`C++`中也有。所以说我已经厌倦了什么都学，什么都学对自己来说就是什么都学不会。自己之前一段时间看了`Swift`然后又去干别的，妥妥的全部忘光。

## noreturn in Swift
`noreturn`是一种属性，被这个属性修饰的函数表示没有任何返回值，函数可以被重写，但是重写之后也必须没有返回值。

具有代表性的函数有：`exit(),abort()`等等。

这里有一篇介绍Swift的Attributes的文章[Swift - Attributes(@attribute) について 【編集中】](https://qiita.com/hayatan/items/fb875b24084e19cf484b)介绍了Swift中的修饰属性。

## noreturn in C++
相比之下我更在意`noreturn`在`C++`中的作用。试着查了一下[What is the point of noreturn?](/questions/10538291/what-is-the-point-of-no)

字面意思看起来是:不返回函数的结果，没有返回值。但实际上跟`void funtion`还是有很大的区别。
- `void function`运行会返回调用函数，只不过没有返回值。
- 被`noreturn`修饰的函数在运行结束之后并不会返回调用函数。


```c++
[[ noreturn ]] void f() {
    throw "error";
    // OK
}

void g() {
   f();
   // unreachable:  在调用了`f()`之后，下面的代码永远不会执行
   std::cout << "No! That's impossible" << std::endl;
}
```

别的事情不说，每次发现这种有关语言的语法的地方不明白的时候，就觉得特别打脸。
