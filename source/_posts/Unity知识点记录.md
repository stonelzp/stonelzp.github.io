---
title: Unity知识点记录
date: 2019-05-23 17:13:44
permalink: unity-learning-note

categories:
- Unity

tags:
- Unity2019.1.3
- C#
- plugins
- 笔记

description: 记录学习Unity过程中的比较碎的知识点，只记录一些零碎的知识。
---

# 更新内容
- (2019/05/29) 更新MenuItem的详细使用
- (2020/03/21) 更新CustomEditor的内容
# Unity的使用
这次终于不是自己单枪匹马的干，而是真的项目中使用Unity引擎来做开发了。

先附上官方文档：
- [Unity User Manual (2019.1)](https://docs.unity3d.com/Manual/index.html)

## 1.Unity C# 中的一些比较重要的使用

### 1.MenuItem
```C#
#if UNITY_EDITOR
        [UnityEditor.MenuItem("Project/Operation")
        public static void OperationRun(){}
#endif
```

或者
```C#
using UnityEditor;
using UnityEngine;

    [MenuItem("Project/Operation")]
    public static void OperationRun(){}

```

上面的用法，貌似也可以被叫做**Debug Menu** ,感觉用处相当大。

便利的调用一些比较麻烦的函数，触发条件和时机需要自己掌控等等，总而言之，这个很重要。

- [第8章　MenuItem](https://anchan828.github.io/editor-manual/web/part1-menuitem.html)

但是要注意添加的方法都是static的方法，调用非静态函数的方法，我知道的有两个:
- 1.向静态函数中传递对象引用。
- 2.在静态函数遍历所有对象找到那个想要的对象并调用对象函数，这限于Unity中debug场景，毕竟可以遍历场景中的所有对象。使用`FindObjectofType`之类的函数。

其他要注意的是，这个功能算是对UnityEditor的扩展功能，使用的时候不要忘记：
```
#if UNITY_EDITOR
using UnityEditor
#endif

加上编译选项才安全。

```
### 2.Unity C# 中的序列化(Serialization)
- [Unity与C#的序列化与反序列化](https://zhuanlan.zhihu.com/p/27990334)
- [Unity 遊戲存檔機制淺談，從序列化 (Serialization) 到儲存裝置 (Storage)](https://dev.twsiyuan.com/2018/06/how-to-save-and-load-gamesaves-in-unity.html)


#### SerializeField
- [Unityの[SerializeField]について色々な疑問に答えてみる](https://qiita.com/makopo/items/8ef280b00f1cc18aec91)


#### HideInInspector
[【Unity】HideInInspectorとSerializeFieldの興味深い関係](https://ekulabo.com/hide-in-inspector)


#### Serializable
- [[Unity] 自前のクラスをインスペクタから編集できるようにする](http://ftvoid.com/blog/post/732)


### 3.MessagePack
- [黒騎士と白の魔王におけるMessagePack-CSharpのUnionの活用事例](http://engineering.grani.jp/entry/2017/05/24/160006)

### 4.FindObjectOfType

> （シーン上にある該当する物の中からUnityが適当に選んだ１つが返ります。１つも存在していなかったらnullが返ります）

> ゲームオブジェクトを参照して、スクリプトにアクセスするのがGetCompornetで、
> 
> スクリプトを参照して、スクリプトにアクセスするのがFindObjectOfTypeです。

就像上面说的那样，下面附上使用：
- [Object.FindObjectOfType](https://docs.unity3d.com/ja/current/ScriptReference/Object.FindObjectOfType.html)
- [Object.FindObjectsOfType](https://docs.unity3d.com/ja/current/ScriptReference/Object.FindObjectsOfType.html)

```C#
public static Object FindObjectOfType (Type type);

public static Object[] FindObjectsOfType (Type type);
```

这个方法非常低效，不适合在每一帧都调用，可以利用SingletonPattern。

[【Unity】GameObject.Find 系関数の処理速度の検証結果](http://baba-s.hatenablog.com/entry/2014/07/09/093240)

### 5. Attribute

#### 1. AddComponentMenu


#### 2. RequireComponent

### 6. CustomEditor
> Defines which object type the custom editor class can edit.

个人理解就是为自定义的类进行编辑器扩展。
```csharp
// 无参数
[CustomEditor(typeof(CustomClass))]
// 有参数
[CustomEditor(typeof(CustomeClass), true)]
```

关于参数的解释，我贴一下源码：

```csharp
using System;

namespace UnityEditor
{
    //
    // Summary:
    //     Tells an Editor class which run-time type it's an editor for.
    public class CustomEditor : Attribute
    {
        //
        // Summary:
        //     Defines which object type the custom editor class can edit.
        //
        // Parameters:
        //   inspectedType:
        //     Type that this editor can edit.
        //
        //   editorForChildClasses:
        //     If true, child classes of inspectedType will also show this editor. Defaults
        //     to false.
        public CustomEditor(Type inspectedType);
        //
        // Summary:
        //     Defines which object type the custom editor class can edit.
        //
        // Parameters:
        //   inspectedType:
        //     Type that this editor can edit.
        //
        //   editorForChildClasses:
        //     If true, child classes of inspectedType will also show this editor. Defaults
        //     to false.
        public CustomEditor(Type inspectedType, bool editorForChildClasses);

        //
        // Summary:
        //     If true, match this editor only if all non-fallback editors do not match. Defaults
        //     to false.
        public bool isFallback { get; set; }
    }
}

```
有参数版本的，设置为true的理由就是，inspedtedType子类也会显示这个部分的Editor内容。（说实话不太理解）

除此之外，`isFallback`变量的作用我也不是很理解。

### 7. ContextMenu
可以在Inspector视图里直接调用函数？



## 2.C# 语法

### ？类型声明
- [Why is there a questionmark on the private variable definition?](https://stackoverflow.com/questions/2326158/why-is-there-a-questionmark-on-the-private-variable-definition)

### ?? operator
- [What do two question marks together mean in C#?](https://stackoverflow.com/questions/446835/what-do-two-question-marks-together-mean-in-c)

### $ operator
- [$ - 字符串内插（C# 参照）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/tokens/interpolated)

### => operator
看见的第一眼以为是lambda表达式，但是后面接着出现的是是变量而不是表达式。

- [C# の => プロパティ](http://var.blog.jp/archives/67192510.html)

相当于`{get;}`但是又不是简单的get。

```
◆ private A a => new A();
◆ ＝ private A a { get { return new A(); } } 
◆ ≠ private A a { get; } = new A();
```

### where关键字
- [where (generic type constraint) (C# Reference)](https://docs.microsoft.com/ja-jp/dotnet/csharp/language-reference/keywords/where-generic-type-constraint)

一般类型约束？

> For example, you can declare a generic class, `MyGenericClass`, such that the type parameter T implements the `IComparable<T>` interface:
```C#
public class AGenericClass<T> where T : IComparable<T> { }
```

使用了模板但是约束了类型，是这样的么。

我发誓这个关键字在我以前用C#的时候肯定知道，但是我忘了。。

### @
- [What does the @ symbol before a variable name mean in C#? [duplicate]](https://stackoverflow.com/questions/429529/what-does-the-symbol-before-a-variable-name-mean-in-c)


### await/async

- [C# 今更ですが、await / async](https://qiita.com/rawr/items/5d49960a4e4d3823722f)
- [Taskを極めろ！async/await完全攻略](https://qiita.com/acple@github/items/8f63aacb13de9954c5da)

### IEnumerator
- [[C#]IEnumeratorとIEnumerableを調べた](https://qiita.com/vc_kusuha/items/2048391d821cb94fa489)



## 3.Unity语法

### 1.Instantiate
根据Prefab生成实例。





## 3.LINQ (Linq)
- [はじめての LINQ - Qiita](https://qiita.com/nskydiving/items/c9c47c1e48ea365f8995)

当我想深入了解LINQ的时候，我发现我需要先理解Lambda表达式。


## 4.Reactive Extensions (Rx)
- [こわくないReactive Extensions超入門 - Qiita](https://qiita.com/acple@github/items/6cfee916f09632037a6e)

## 5.UniRx

- [又见Rx——Rx via UniRx](https://zhuanlan.zhihu.com/p/35189325)

这是我在知乎上找到的一篇文章，感觉能学到很多东西。


[Reactive Extensions for Unity](https://github.com/neuecc/UniRx)

- [UniRxを導入するメリット ～こういう時にUniRxは使えるよ～ - Qiita](https://qiita.com/Marimoiro/items/2ad8a1b422a3fe98f938)


## 6.Lambda
直接搜`C# lambda`最先出来的三个链接。

- [Lambda expressions (C# Programming Guide)](https://docs.microsoft.com/ja-jp/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)
- [=> operator (C# Reference)](https://docs.microsoft.com/ja-jp/dotnet/csharp/language-reference/operators/lambda-operator)





# Unity中的插件

## SteamVR Unity
要记录的是
SteamVR Unity Plugin v2这个Unity的VR开发插件。开始调查这个插件的契机是不知道`SteamVR_Behaviour_Pose`这个类的作用，而明白这个是插件的内容。

- [Unity＋HTC Vive開発メモ](https://framesynthesis.jp/tech/unity/htcvive/)

- [Unity標準のVR機能（UnityEngine.XR）メモ](https://framesynthesis.jp/tech/unity/xr/)

上面的这个人的文章我看很厉害就直接先粘链接了。

### SteamVR_Behaviour_Pose

## Amplify Shader Editor
- [【Amplify Shader Editor】ノードベースでシェーダ作りのAmplifyを触ってみました。エディタの操作性、学習コスト、サンプルデモを大量に紹介！](http://www.asset-sale.net/entry/Amplify_Shader_Editor)


