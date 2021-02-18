---
title: Unity-Editor扩展
date: 2019-08-13 19:58:46
permalink: unity-editor-extension

categories:
- Unity
tags:
- C#
- Unity
---

编辑器扩展是很重要的一个部分，多使用可以提高开发效率。当然也是需要学习的。(未整理)


<!--more-->

由于是Editor的扩展功能在实际的项目中要注意不要把Editor的功能打包在项目的功能里面。

## Unity Editor扩展
Unity编辑器扩展好多内容啊，只能记下我遇见的了。有时间要找找一些感觉用的上的。

参考公司大佬的写法，我做了一个当我准备关掉Editor的时候会弹出来一个MessageBox的功能。没啥实际的功能，就是为了提醒。

### 实现需求
[EditorApplication.wantsToQuit](https://docs.unity3d.com/ScriptReference/EditorApplication-wantsToQuit.html)就是实现这个功能的主角。

实现这个功能的代码:
```csharp
[InitializeOnLoad]
public class EditorReminder
{
    [InitializeOnLoadMethod]
    public static void ValidateReminder()
    {

        EditorApplication.wantsToQuit += EditorWantToQuitReminder;

    }

    static bool EditorWantToQuitReminder()
    {

        bool.TryParse(EditorUserSettings.GetConfigValue("EnableReminder"), out var enable);
        if(!enable)
        {
            return true;
        }

        return EditorUtility.DisplayDialog("Reminder", "Have you released all your locks on GitHub?", "Yes, quit!", "No, wait a second.");
    }
    
}
```
几行就能实现这个功能，真的好简单啊（站着说话不腰疼）。

### InitializeOnLoad
> **[Running Editor Script Code on Launch](https://docs.unity3d.com/Manual/RunningEditorCodeOnLaunch.html)**
>
> Sometimes, it is useful to be able to run some editor script code in a project as soon as Unity launches without requiring action from the user. You can do this by applying the InitializeOnLoad attribute to a class which has a static constructor. A static constructor is a function with the same name as the class, declared static and without a return type or parameters (see [here](http://docs.go-mono.com/index.aspx?link=ecmaspec%3a17.11) for more information):-

官方文档上说对于有一个静态构造函数的类前可以加上**InitializeOnLoad**这个attribute，但是上面的那个例子我并没有显式的去声明一个静态的构造函数。而且执行下来也没什么问题，难道说有一个静态构造函数会好一些吗。

> A static constructor is always guaranteed to be called before any static function or instance of the class is used, but the InitializeOnLoad attribute ensures that it is called as the editor launches.

就是这样使用的。

### InitializeOnLoadMethod
[InitializeOnLoadMethodAttribute](https://docs.unity3d.com/ScriptReference/InitializeOnLoadMethodAttribute.html)

> Allow an editor class method to be initialized when Unity loads without action from the user.

项目被加载到Unity中的时候带有这个属性的静态函数就会被自动调用。

### RuntimeInitializeOnLoadMethod
- [RuntimeInitializeOnLoadMethodAttribute](https://docs.unity3d.com/ScriptReference/RuntimeInitializeOnLoadMethodAttribute.html)

添加了这个属性的函数会在游戏开始运行的时候被自动调用而不需要人为调用。
> Allow a runtime class method to be initialized when a game is loaded at runtime without action from the user.

`[RuntimeInitializeOnLoadMethod]`的默认行为会在`Awake`之后被调用，还有一种说法是在`OnEnable`之后调用。

> The execution order of methods marked [RuntimeInitializeOnLoadMethod] is not guaranteed.
这句话应该是说都有`[RuntimeInitializeOnLoadMethod]`的函数调用顺序无法被保证。

#### RuntimeInitializeLoadType
- [RuntimeInitializeLoadType](https://docs.unity3d.com/ScriptReference/RuntimeInitializeLoadType.html)

指定参数的话就可以指定不同时间段调用。
```csharp
using UnityEngine;

public class InitTest  : MonoBehaviour
{
    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
    static void OnBeforeSceneLoadRuntimeMethod ()
    {
        Debug.Log("Before scene loaded");
    }

    void Awake()
    {
        Debug.Log("Awake");
    }
    void OnEnable()
    {
        Debug.Log("OnEnable");
    }

    [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.AfterSceneLoad)]
    static void OnAfterSceneLoadRuntimeMethod()
    {
        Debug.Log("After scene loaded");
    }

    [RuntimeInitializeOnLoadMethod]
    static void OnRuntimeMethodLoad()
    {
        Debug.Log("RuntimeMethodLoad: After scene loaded");
    }

    void Start()
    {
        Debug.Log("Start");
    }
}
```

执行顺序为:`BeforeSceneLoad`->`Awake`->`OnEnable`->`AfterSceneLoad`->`no paramater`->`Start`



### EditorApplication
[EditorApplication](https://docs.unity3d.com/ScriptReference/EditorApplication.html)

#### wantsToQuit
就是我用的那个啦，连用法都是抄的官方文档的。

> Unity raises this event when the editor application wants to quit.

```csharp
using UnityEngine;
using UnityEditor;

// Ensure class initializer is called whenever scripts recompile
[InitializeOnLoad]
public class EditorWantsToQuitExample
{
    static bool WantsToQuit()
    {
        Debug.Log("Editor prevented from quitting.");
        return false;
    }

    static EditorWantsToQuitExample()
    {
        EditorApplication.wantsToQuit += WantsToQuit;
    }
}
```
想要在Editor退出前做点什么？那么用这个吧。

#### quitting
想要在Editor退出的时候做点什么那么用这个。


### EditorUtility
**[EditorUtility](https://docs.unity3d.com/ScriptReference/EditorUtility.html)**

一些通用的函数库，看名字就知道应该会很多
#### EditorUtility.DisplayDialog
**[EditorUtility.DisplayDialog](https://docs.unity3d.com/ScriptReference/EditorUtility.DisplayDialog.html)**

弹出一个窗口？
#### EditorUtility.DisplayDialogComplex
弹出一个更复杂的窗口？（貌似是支持三个按钮）

### MenuItem
**[MenuItem](https://docs.unity3d.com/ScriptReference/MenuItem.html)**

这个属性可以让在Editor上的菜单栏添加项目，可以用来做什么呢？那可太多了，比如说需要方便切换的项目设定，亦或者想要方便调用一个函数（啊这个好像可以直接在想要调用的GameObject上右键调用，使用另外一个Attribute）。总而言之，就是方便。

需要注意的是**只有静态函数才能使用这个MenuItem属性**。

这个光看官方文档不靠谱，讲的太少了，看我总结的？我总结的也没什么好看的，看这个
- [メニューを追加するための属性「MenuItem」は意外と多機能【Unity】【エディタ拡張】【属性】](http://kan-kikuchi.hatenablog.com/entry/MenuItem)

还能设定快捷键，厉害了。

```csharp
[MenuItem(menuRoot + "EnableReminder")]
private static void EnableReminder()
{
    var path = menuRoot + "EnableReminder";
    var enable = Menu.GetChecked(path);

    Menu.SetChecked(path, !enable);
    EditorUserSettings.SetConfigValue("EnableReminder", (!enable).ToString());
}
```
用的话就是这么用的。我靠**EditorUserSettings**又是什么？

---
- 2020/01/16更新

#### MenuItem参数说明
在菜单栏里添加自定义项的话当然需要对其参数有所了解，直接进入源码查看即可，这里我顺便贴一下注释:
```csharp
//
// Summary:
//     Creates a menu item and invokes the static function following it, when the menu
//     item is selected.
//
// Parameters:
//   itemName:
//     The itemName is the menu item represented like a pathname. For example the menu
//     item could be "GameObject/Do Something".
//
//   isValidateFunction:
//     If isValidateFunction is true, this is a validation function and will be called
//     before invoking the menu function with the same itemName.
//
//   priority:
//     The order by which the menu items are displayed.
public MenuItem(string itemName, bool isValidateFunction, int priority);
```
这里关于`isValidateFunxtion`的说明就是增加一个验证函数，比如说我只想在游戏运行的时候执行某个操作，在Editor未运行的时候调用这个函数没有意义那就加上一个验证函数。

-----
- 2020/02/10 更新内容

过了好久终于有机会亲自尝试一下这个功能，同时对一些疑问进行了验证。

1.首先是这个MenuItem的`isValidateFunction`为True的情况下的执行情况：

按照源码中的注释所说，标注了这个属性的函数(以下称为*验证函数*)会在具有相同`itemName`的函数之前执行，在我实际添加了执行log的结果来看，这个函数确实会在前面执行，但是执行的时机却很迷惑。

单纯的情况下，在执行无`isValidateFunction`同名`itemName`函数之前，这个验证函数被执行了两次。哪怕是点开MenuItem执行其他的功能函数，甚至什么都不执行点击其他位置关掉点开MenuItem弹出来的菜单，这个验证函数都会被执行一次。

看来这个验证函数还跟Unity的Editor的运行有关，比如说关掉MenuItem的窗口会刷新一下状态，执行一遍所有的验证函数（我猜的）什么的。

2.验证函数的返回值的影响

细心的我发现，与被调用的函数不同，验证函数会有一个布尔类型的返回值，对于这个返回值我测试得到的结果是：

当返回值是true的时候，就是平常一样正常的调用。

返回值是false的时候，被调用的函数(具有相同的`itemName`)的MenuItem就变灰色了，不能点击调用。

-----

关于`priority`的含义就是可以自定义菜单选项的顺序，这里要注意的是想要在弹出来的菜单中看到分割线的话，优先级的数值是10起跳的。摘一段官网的文字:
> Note: The understanding of ten or greater is considered to create a divider in the menu. However, as per the example above, the difference between script function need to have the priority separated by 11 or more. This is why the example before has a value of 100 and one of 111. Changing 111 to 110 does not have a divider.

### ScriptTemplates
简单创建一个想要的脚本模板。Unity提供了两种方式。

一种是直接在Unity原有的ScriptTemplates的地方添加类似的新的模板。

另一种是在工程里添加新的模板。

当然是推荐第二种啦。但是第一种也应该知道，毕竟要看人家是怎么做的啊。贴一下Unity引擎内置的脚本模板的位置:

Windows:
```
C:\Program Files\Unity\Editor\Data\Resources\ScriptTemplates\
```
Mac:
```
/Applications/Unity/Unity.app/Contents/Resources/ScriptTemplates/
```

主要是第二种使用方法，需要做的是以下的事情：
- 创建Assets/ScriptTemplates/文件路径，模板文件放置在里面
- 模板文件命名规则：{priority}-{メニュー名}-{ファイル名}.{拡張子}.txt
- 重启Editor 

不需要MenuItem，只要起对了名字，放对了位置，那么重启一下就能用。厉害了。

知道了之后也不是很难嘛，然后我看到了一篇关于**ScriptableObject**式的脚本模板文章：
- [ScriptTemplatesでScriptableObjectのための環境構築](https://qiita.com/keidroid/items/40db91531c32a13c257f)

虽然我暂时并没有找到任何非要使用ScriptableObject的理由，是我还不了解这个概念，什么情况下该使用这个东西不知道。

### EditorUserSettings
- [プロジェクト内でデータを保存【Unity】【エディタ拡張】](http://kan-kikuchi.hatenablog.com/entry/EditorUserSettings)

挺神奇的一个功能，用来保存本地设定相当理想的功能，但是貌似只有Get和Set两个方法写入的内容没法删除，别瞎写好么，我第一印象。


##先记下，后整理
- [【Unity】エディタ拡張で使用できるコールバックを40個まとめて紹介](http://baba-s.hatenablog.com/entry/2017/12/04/090000#Unity-%E3%82%A8%E3%83%87%E3%82%A3%E3%82%BF%E3%82%92%E7%B5%82%E4%BA%86%E3%81%97%E3%81%9F%E6%99%82)




## Editor扩展实例


### Sample-Template (Editor扩展的模板写法)
模板写法包含了一些关键部分，就是个参照。

```csharp
using UnityEditor;
using UnityEngine;

namespace EditorExtension
{
    [CustomEditor(typeof(ExtensionClass), true)]
    public class ExtensionClassInspector ： Editor
    {
        protected ExtensionClass extensionClass = null;
        protected SerializedProperty attributeA;
        protected SerializedProoerty attributeB;
        ......

        private void OnEnable()
        {
            extensionClass = target as ExtensionClass;

            attributeA = serializedObject.FindObject("ExtensionClass_AttributeAName");
            attributeB = serializedObject.FindObject("ExtensionClass_AttributeBName");
            ......
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();

            EditorGUILayout.PropertyField(attributeA);
            EditorGUILayout.PropertyField(attributeB);
            ......

            serializedObject.ApplyModifiedProperties();
        }
    }
}
```

最普通的格式就是上面那样，这是对Inspector格式的扩展，内容都是最基础的。需要注意的是在对`OnInspectorGUI`函数进行重写的时候，最开始要**Update()**，最后要**ApplyModifiedProperties()**。

#### indentLevel (缩进)
```csharp
// 在上面的例子中的话
EditorGUILayout.PropertyField(attributeA);

EditorGUI.indentLevel++;
EditorGUILayout.PropertyField(attributeB);
EditorGUI.indentLevel--;

```
#### Content-Name (Inspector变量名设定)
在Inspector不指定变量名字的话就会使用默认的变量名字，但是也可以自己设置。

```csharp
EditorGUILayout.PropertyField(attributeA, new GUIContent("NameA"));
```

这里的话就不自觉的产生了**EditorGUILayout**是什么的想法，还有其它的作用吧

1. EditorGUILayout
2. GUIContent


#### Type-judgement (类型判断)
在`OnInspectorGUI`函数中也想要使用条件判断，这个时候不知道变量的类型不好办，上面的代码中表示的，变量类型都是**SerializedProperty**，条件语句对这个变量类型并不知晓。

```csharp
if(attributeA.boolValue) return;

switch(attributeB.intValue) {}
```

等等类型，基本数据类型应该都会有覆盖到，不太清楚。

#### Disable-Area (Inspector在运行时添加不可交互内容)
```csharp
if(EditorAppliation.isPlaying)
{
    EditorGUILayout.BeginVertical(GUI.skin.box);
    EditorGUILayout.LabelField("Running status");

    //
    EditorGUI.BeginDisabledGroup(true);
    EditorGUILayout.PropertyField(flag1);
    EditorGUILayout.PropertyField(flag2);
    EditorGUI.EndDisabledGroup();

    EditorGUILayout.BeginHorizontal();
    GUILayout.FlexibleSpace();

    if (GUILayout.Button("buttonA", GUILayout.Width(100)))
    {
        (target as ExtensionClass)?.SampleFunctionA();
    }
    EditorGUILayout.EndHorizontal();

    EditorGUILayout.EndVertical();

}
```
我只是拿来用而已.....出现了好多我不会用的控件。还需要学习啊。


### Inspector中显示List元素
- [【Unity】【エディタ拡張】配列やリストのInspector表示を改良する](http://light11.hatenadiary.com/entry/2019/04/13/232549)

因为比我想象的要麻烦，我就直接用了人家的代码......原本颜色什么的都无所谓的。

真的超级麻烦，如果希望上面的List中的元素是List又该怎么办呢？

答案是没人这么写啊...只好自己写个类，类里面的属性加上List好了。

1. error解决

```
Unsupport type ..
ApplyModifiedProperties()
```

- ['Unsupported type' error in custom editor script](https://answers.unity.com/questions/533541/unsupported-type-error-in-custom-editor-script.html)

删掉重新再add component之后就修复了。

### Inspector中显示多元List
- [多次元のListをInspectorに表示する【Unity】](http://kan-kikuchi.hatenablog.com/entry/ValueListList)

方法很简单，但是这里遇见了一个问题，因为使用了Inspector扩展，Inspector上不显示。
```
// EditorGUILayout.PropertyField(pathAnchors);
EditorGUILayout.PropertyField(pathAnchors, true);
```
加上一个true的参数就好了，原因我也不知道，之后在调查吧。

```csharp
[Serializable]
public class AreaPathAnchor
{
    public List<GameObject> List = new List<GameObject>();

    public AreaPathAnchor(List<GameObject> list)
    {
        List = list;
    }
}

[SerializeField]
private List<AreaPathAnchor> pathAnchors = new List<AreaPathAnchor>();
```

Inspector扩展中对应的代码:
```chsarp
public override void OnInspectorGUI()
{
    EditorGUILayout.PropertyField(pathAnchors, true);
}
```

### Inspector中加入HDRColor选择(未整理)
其实这个属性也不需要刻意的去扩展，只要在属性前加上限定Unity就会自动处理了。
```csharp
[ColorUsage(false,true)] private Color color1;
```

像这样子。

关于这个属性的更详细的说明之后再整理。

- [ColorUsageAttribute Constructor](https://docs.unity3d.com/ScriptReference/ColorUsageAttribute-ctor.html)
- [マテリアルのEmissionを操作](https://nopitech.com/2019/01/29/post-962/)



### MinMaxSlider实现 （可以拖拽的SliderBar)
目的是实现一个有最大值和最小值的可拖拽的SlideBar，而且最大值和最小值可以指定，但是不能超过固定的范围。

用的是自己做的项目中的一些实现，所以直接粘贴了。

首先是要扩展的变量声明：
```chsarp
using System;
using UnityEngine;

namespace CubeFantasy.EditorExtension
{
    /// <summary>
    /// Sample Editor Extension basic class.
    /// </summary>
    ///
    [Serializable, AddComponentMenu("CubeFantasy/SampleEditorExtension")]
    public class SampleEditorExtension : MonoBehaviour
    {
        // sample for MinMaxSlider
        [SerializeField, Range(0, 30), HideInInspector]
        public int timeMin = 0;
        [SerializeField, Range(0, 30), HideInInspector]
        public int timeMax = 0;
    }
}
```
然后是真正的扩展实现：
```csharp
using CubeFantasy.EditorExtension;
using UnityEditor;
using UnityEngine;

namespace CubeFantasy
{
    [CustomEditor(typeof(SampleEditorExtension),true)]
    public class SampleEditorExtensionInspector : Editor
    {
        SerializedProperty TimeMin;
        SerializedProperty TimeMax;

        private void OnEnable()
        {
            TimeMin = serializedObject.FindProperty("timeMin");
            TimeMax = serializedObject.FindProperty("timeMax");
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();

            // base.OnInspectorGUI();

            EditorGUILayout.BeginHorizontal();

            EditorGUILayout.PrefixLabel("MinMaxSlider");

            var min = (float)TimeMin.intValue;
            var max = (float)TimeMax.intValue;

            min = Mathf.Clamp((int)EditorGUILayout.FloatField(min), 0, max);
            EditorGUILayout.MinMaxSlider(ref min, ref max, 0, 30);
            max = Mathf.Clamp((int)EditorGUILayout.FloatField(max), min, 30);

            TimeMin.intValue = (int)min;
            TimeMax.intValue = (int)max;

            EditorGUILayout.EndHorizontal();

            serializedObject.ApplyModifiedProperties();
        }
    }
}
```
最后实现的效果：

![MinMaxSlider实现效果](MinMaxSlider.png)

### Foldout and window Popup 实现
这是一个小的功能，和一个大的功能，我以为很简单但实际上是非常的浮复杂，涉及的知识也不少。我将其分为几部实现。

主要是参考了公司大佬的Wwise的Event的扩展，方便在Editor中添加删除WwiseEvent和选择WwiseEvent。我想这个功能会很常见就想着自己保存下来实践一下，发现要实现的内容不是那么简单。

#### Foldout 实现
折叠效果的实现很简单，借助Unity的`EditorGUILayout.Foldout(bool folding, "ContentName")`直接使用就可以了。重要的是需要折叠的内容，这次我要举的列子是ArrayList。

这是我们要进行扩展的对象：
```csharp
// 省略using部分
namespace CubeFantasy.EditorExtension
{
    /// <summary>
    /// Sample Editor Extension basic class.
    /// </summary>
    ///
    [Serializable, AddComponentMenu("CubeFantasy/SampleEditorExtension")]
    public class SampleEditorExtension : MonoBehaviour
    {
        // sample for foldout and pop up list
        [SerializeField]
        public List<string> WwiseEventNameList = new List<string>();
    }
}
```
是的，实际上我们想要扩展的只是这个文字列的数组而已，但是我希望这个文字列的数组，在Inspector上的表现是
- 可以折叠
- 可以动态添加元素
- 添加的元素是我们指定添加的(Popup Window)
- 甚至可以在PopWindow中进行查找

下面是Inspector的扩展实现，首先是Flodout实现：
```csharp
// 省略using部分的内容
namespace CubeFantasy
{
    [CustomEditor(typeof(SampleEditorExtension), true)]
    public class SampleEditorExtensionInspector : Editor
    {
        SampleEditorExtension SampleObjectScript;

        SerializedProperty WwiseEventName;
        SerializedProperty WwiseEventNameList;

        bool floding = true;

        private void OnEnable()
        {
            SampleObjectScript = target as SampleEditorExtension;

            WwiseEventNameList = serializedObject.FindProperty("WwiseEventNameList");
        }

        public override void OnInspectorGUI()
        {
            serializedObject.Update();
            // base.OnInspectorGUI();

            // Sample for Flodout
            if (floding = EditorGUILayout.Foldout(floding, "SampleFoldout"))
            {
                EditorGUI.indentLevel++;
                var length = WwiseEventNameList.arraySize;
                var newLength = EditorGUILayout.IntField("Size", length);
                if (newLength > length)
                {
                    var currentSize = WwiseEventNameList.arraySize;
                    for (var i = 0; i < newLength - currentSize; i++)
                    {
                        WwiseEventNameList.InsertArrayElementAtIndex(WwiseEventNameList.arraySize);
                    }
                }
                if (newLength < length)
                {
                    var currentSize = WwiseEventNameList.arraySize;
                    for (var i = 0; i < currentSize - newLength; i++)
                    {
                        WwiseEventNameList.DeleteArrayElementAtIndex(WwiseEventNameList.arraySize - 1);
                    }
                }
                for (var i = 0; i < WwiseEventNameList.arraySize; i++)
                {
                    WwiseEventNameField(WwiseEventNameList.GetArrayElementAtIndex(i));
                }

                EditorGUI.indentLevel--;
            }

            serializedObject.ApplyModifiedProperties();
        }

        void WwiseEventNameField(SerializedProperty p)
        {
            var w = WwiseEvents.None;
            if (!string.IsNullOrEmpty(p.stringValue))
            {
                try
                {
                    w = (WwiseEvents)Enum.Parse(typeof(WwiseEvents), p.stringValue, true);
                }
                catch (ArgumentException)
                {
                    UnityEngine.Debug.LogError($"{SampleObjectScript.name} の {p.stringValue} は存在しません。");
                }
            }

            var eventNamePopupRect = GUILayoutUtility.GetRect(0, 0);
            eventNamePopupRect.y += EditorGUIUtility.singleLineHeight + 2;

            EditorGUILayout.BeginHorizontal();
            EditorGUILayout.PrefixLabel("WwiseEventName");

            eventNamePopupRect.x += EditorGUIUtility.labelWidth;

            if (EditorGUILayout.DropdownButton(new GUIContent(w.GetDescription()), FocusType.Passive))
            {
                new WwiseEventSearchWindow().Popup(eventNamePopupRect, Enum.GetNames(typeof(WwiseEvents)), w.GetDescription(),
                    x =>
                    {
                        p.stringValue = ((WwiseEvents)x).GetDescription();

                        serializedObject.ApplyModifiedProperties();
                    });
            }
            EditorGUILayout.EndHorizontal();
        }
    }
}

```
上面的就是实现的一部分代码，为了能让其运行还缺三个部分：
- `WwiseEvent`的`enum`类型
- `Enum`类型的扩展，好方便Wwise的类型获得其Attribute的Description内容
- Popup窗口的扩展

#### WwiseEvent的enum创建
主要为了用`enum`类型来管理Wwise的事件，什么场合下适合这么用呢，比如说想要把事件的名称传递给第三方库（这里就会Wwise），而这种事件又可能会产生很多，这种情况下，使用enum类型便能方便的管理。

这里的内容只要放到相应位置，比如说放置一些固定数据的文件内容中，然后把命名空间公开，方便扩展那一部分好使用就可以了。
```csharp
using System;

namespace CubeFantasy
{
    public static class CubeFantasyConst
    {
        public enum WwiseEvents : int
        {
            [System.ComponentModel.Description("None")]
            None,
            [System.ComponentModel.Description("Wwise_Event_Name_Sample_1")]
            Wwise_Event_Name_Sample_1,
            [System.ComponentModel.Description("Wwise_Event_Name_Sample_2")]
            Wwise_Event_Name_Sample_2,
            [System.ComponentModel.Description("Wwise_Event_Name_Sample_2")]
            Wwise_Event_Name_Sample_3,
            [System.ComponentModel.Description("Wwise_Event_Name_Sample_2")]
            Wwise_Event_Name_Sample_4,
            [System.ComponentModel.Description("Wwise_Event_Name_Sample_2")]
            Wwise_Event_Name_Sample_5
    }
    }
}
```

#### 对Enum进行扩展
可以看到上面我们为WwiseEvent添加了Description，我也不知道中文怎么翻译，反正就是那个意思，为这个枚举的元素添加了属性描述，要取得这个描述就需要我们为Enum进行扩展。

那么Enum是什么呢？我也不清楚...

```csharp
using System;
using System.ComponentModel;

namespace CubeFantasy.Extensions
{
    public static class EnumExtensions 
    {
        public static string GetDescription(this Enum value)
        {
            var field = value.GetType().GetField(value.ToString());
            var attribute = Attribute.GetCustomAttribute(field, typeof(DescriptionAttribute)) as DescriptionAttribute;
            if(attribute != null)
            {
                return attribute.Description;
            }

            return value.ToString();
        }
    }
}
```
这种写法我在学习UniRx的时候经常看到，貌似是C#的写法，可以为一个类添加在类外的静态函数？参数要使用本身`this`，当进行调用的时候this是缺省的。在有多个参数的时候，第一个一定是this，然后调用的时候是缺省的。

感觉C#方面的技术要提升了。

#### 对PopUpWindow进行扩展实现
即上面的代码中出现的`WwiseEventSearchWindow`这个类的实现，内容我没有搞懂，直接Copy过来就使用了。

```csharp
using UnityEngine;
using UnityEditor;
using System.Text;
using System;
using System.Linq;

public class WwiseEventSearchWindow : PopupWindowContent
{
    class FilterItem
    {
        public string Name;
        public int Index;
    }

    private float WindowWidth = 400;
    private float WindowHeight
    {
        get
        {
            var height = 0.0f;
            var singleLineHeight = EditorGUIUtility.singleLineHeight + EditorGUIUtility.standardVerticalSpacing;

            height += (Math.Min(filterdItems.Length, 10) + 1) * singleLineHeight;

            return height;
        }
    }

    Vector2 scroll;
    int selectedIndex = 0;
    FilterItem[] list;
    FilterItem[] filterdItems;
    Action<int> onCloseAction;
    bool init;
    string filter;

    GUIStyle toolbarSearchField;
    GUIStyle toolbarSearchFieldCancelButton;
    GUIStyle toolbarSearchFieldCancelButtonEmpty;

    /// <summary>
    /// サイズを取得する
    /// </summary>
    public override Vector2 GetWindowSize() => new Vector2(WindowWidth, WindowHeight);

    public void Popup(Rect rect, string[] list_, string selectedVal, Action<int> onClose)
    {
        for (var i = 0; i < list_.Length; i++)
        {
            if (list_[i] == selectedVal)
            {
                selectedIndex = i;
            }
        }

        init = true;
        list = list_.Select((item, i) => new FilterItem() { Index = i, Name = item }).ToArray();
        filterdItems = list;
        onCloseAction = onClose;

        UnityEditor.PopupWindow.Show(rect, this);
    }

    /// <summary>
    /// GUI描画
    /// </summary>
    public override void OnGUI(Rect rect)
    {
        var newFilter = SearchFilterField(filter);

        if (init || newFilter != filter)
        {
            init = false;
            filter = newFilter;
            filterdItems = list
                .Where(x => x.Index == selectedIndex
                    || string.IsNullOrEmpty(filter)
                    || x.Name.ToLower().Contains(filter.ToLower()))
                .ToArray();
        }

        scroll = EditorGUILayout.BeginScrollView(scroll);

        var style = GUI.skin.GetStyle("ExposablePopupItem");
        style.richText = true;

        for (var i = 0; i < filterdItems.Length; i++)
        {
            var item = filterdItems[i];

            var flag = GUILayout.Toggle(selectedIndex == item.Index, FormatString(item.Name, filter), style);
            if (flag != (selectedIndex == item.Index))
            {
                selectedIndex = item.Index;
                onCloseAction(selectedIndex);
                editorWindow.Close();
                break;
            }
        }

        EditorGUILayout.EndScrollView();
    }

    private static int searchFieldHash = "SearchBoxTestWindow_SearchField".GetHashCode();

    string SearchFilterField(string filter)
    {
        var style = GUI.skin.GetStyle("ExposablePopupItem");

        int maxWidth = 0;
        for (var i = 0; i < filterdItems.Length; i++)
        {
            var w = style.CalcSize(new GUIContent(filterdItems[i].Name));
            maxWidth = Math.Max(maxWidth, (int)w.x);
        }

        WindowWidth = maxWidth + 24;

        var rect = GUILayoutUtility.GetRect(16f, 24f,
            EditorGUIUtility.singleLineHeight + EditorGUIUtility.standardVerticalSpacing,
            EditorGUIUtility.singleLineHeight + EditorGUIUtility.standardVerticalSpacing,
            new GUILayoutOption[] { GUILayout.Width(maxWidth) });
        rect.x += 4f;
        rect.y += 4f;

        int controlID = GUIUtility.GetControlID(searchFieldHash, FocusType.Passive, rect);

        return ToolbarSearchField(controlID, rect, filter, false);
    }

    string ToolbarSearchField(int id, Rect position, string text, bool showWithPopupArrow)
    {
        var position2 = position;
        position2.width -= 14f;
        var position3 = position;
        position3.x += position.width;
        position3.width = 14f;

        GUI.SetNextControlName("filterField");
        text = EditorGUI.TextField(position, text, toolbarSearchField);
        GUI.FocusControl("filterField");

        if (text == "")
        {
            GUI.Button(position3, GUIContent.none, toolbarSearchFieldCancelButtonEmpty);
        }
        else
        {
            if (GUI.Button(position3, GUIContent.none, toolbarSearchFieldCancelButton))
            {
                text = "";
                GUIUtility.keyboardControl = 0;
            }
        }
        return text;
    }

    static private GUIStyle GetStyle(string styleName)
    {
        var style = GUI.skin.FindStyle(styleName);
        if (style == null)
        {
            style = EditorGUIUtility.GetBuiltinSkin(EditorSkin.Inspector).FindStyle(styleName);
        }
        if (style == null)
        {
            style = new GUIStyle();
        }
        return style;
    }

    string FormatString(string text, string filter)
    {
        if (string.IsNullOrEmpty(filter))
        {
            return text;
        }

        var b = new StringBuilder();

        while (true)
        {
            var index = text.IndexOf(filter, StringComparison.CurrentCultureIgnoreCase);
            if (index == -1)
            {
                if (text.Length > 0)
                {
                    b.Append(text);
                }
                break;
            }

            b.Append(text.Substring(0, index));
            b.Append("<b><color=cyan>");
            b.Append(text.Substring(index, filter.Length));
            b.Append("</color></b>");
            text = text.Substring(index + filter.Length);
        }
        return b.ToString();
    }

    /// <summary>
    /// 開いたときの処理
    /// </summary>
    public override void OnOpen()
    {
        toolbarSearchField = GetStyle("ToolbarSeachTextField");
        toolbarSearchFieldCancelButton = GetStyle("ToolbarSeachCancelButton");
        toolbarSearchFieldCancelButtonEmpty = GetStyle("ToolbarSeachCancelButtonEmpty");
    }

    /// <summary>
    /// 閉じたときの処理
    /// </summary>
    public override void OnClose()
    {
    }
}
```

当然我不搞懂也行，当到了一定时间我就可以直接复制使用，但是万一出现了一些更新，里面的内容可能就会变得一团糟。

**所以我还是很有必要把这一部分的内容完全搞懂的。**

#### 实现的效果
写了那么多，实现的内容一张图片就足够总结了，这一部分说来简单也很复杂，算是各种知识的拓展了。

![实现效果](FoldoutAndPopupView.png)

#### Foldout效果的官网实现
- [EditorGUILayout.Foldout](https://docs.unity3d.com/ja/current/ScriptReference/EditorGUILayout.Foldout.html)

嘛，重要的不是他的实现，而是里面的东西，一些没见过的东西。

```csharp
// Create a foldable menu that hides/shows the selected transform position.
// If no Transform is selected, the Foldout item will be folded until
// a transform is selected.

using UnityEditor;
using UnityEngine;

public class FoldoutUsage : EditorWindow
{
    bool showPosition = true;
    string status = "Select a GameObject";

    [MenuItem("Examples/Foldout Usage")]
    static void Init()
    {
        FoldoutUsage window = (FoldoutUsage)GetWindow(typeof(FoldoutUsage));
        window.Show();
    }

    public void OnGUI()
    {
        showPosition = EditorGUILayout.Foldout(showPosition, status);
        if (showPosition)
            if (Selection.activeTransform)
            {
                Selection.activeTransform.position =
                    EditorGUILayout.Vector3Field("Position", Selection.activeTransform.position);
                status = Selection.activeTransform.name;
            }

        if (!Selection.activeTransform)
        {
            status = "Select a GameObject";
            showPosition = false;
        }
    }

    public void OnInspectorUpdate()
    {
        this.Repaint();
    }
}
```
第一次见到的弹出新窗口的使用`(FoldoutUsage)GetWindow(typeof(FoldoutUsage))`，和`Selection.activeTransform`的使用。

至于这段代码的效果就是，调用这个方法后弹出一个窗口，里面包含着你选定物体的Transform，然后可以折叠，但是只能选一个，多选没有用。

都是比较重要的知识。


### UnityEditor Inspector script field
当我选择注释掉`base.OnInspectorGUI()`这一行的时候，那个显示脚本名称的ScriptField就消失了，虽然看起来更清楚了但是有的时候还是会有混淆，这里讲如何恢复这个*ScriptField*。

首先是参考了下面两篇文章：
- [How to add a "Script" field in custom inspector?](https://answers.unity.com/questions/550829/how-to-add-a-script-field-in-custom-inspector.html)
- [UnityのEditor拡張で「Script」フィールドを設置する方法](http://sokuhatiku.hateblo.jp/entry/2016/12/13/174513)

```chasrp
[CustomEditor(typeof(SampleClass))]
public class SampleClassInspector : Editor
{
    MonoScript script = null;

    private void OnEnable()
    {
        script = MonoScript.FromMonoBehaviour((SampleClass)target);
    }
    public override void OnInspectorGUI()
    {
        serializedObject.Update();
        // base.OnInspectorGUI();
        EditorGUI.BeginDisabledGroup(true);
        EditorGUILayout.ObjectField("Script", script, typeof(MonoScript), false);
        EditorGUI.EndDisabledGroup();
    }
}
```

代码实现就是那样，至于参数，等遇到之后再追加说明吧。



## SerializedObject
- [【Unity】【エディタ拡張】SerializedObjectの勘所をまとめてみる](http://light11.hatenadiary.com/entry/2018/03/15/225709)





















