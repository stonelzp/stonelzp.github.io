---
title: Unity-Serialization理解
date: 2019-09-20 18:44:14
permalink: unity-serialization

categories:
- Unity

tags:
- serialization
---
这个概念对于Unity来说非常重要。了解这个概念应该会对开发的效率有很大的提升。

<!--more-->
在最开始应该理解一下什么是序列化。

## Serialization 序列化
> 序列化又称串行化，是.NET运行时环境用来支持用户定义类型的流化的机制。其目的是以某种存储形式使自定义对象持久化，或者将这种对象从一个地方传输到另一个地方。

Unity中非常多非常多的场景中使用了序列化，参考一下这篇文章[Serialization in Unity](https://blogs.unity3d.com/2014/06/24/serialization-in-unity/)，深入理解一下Unity中的序列化。

下面我就对这篇文章的内容进行整理，主要是做笔记。

### Serialization in Unity
先是比较重要的内容摘抄一下。

- **Storing data stroed in your scripts.**
    - 保存你脚本中保存的数据。是大多数人最熟知的功能。
- **Inspector window.**
    - Inspector窗口不是经过C#的API来了解内部的数据结构或者属性，它要求对象序列化数据然后显示序列化之后的数据内容。
- **Prefab.**
    - prefab是一个或多个对象和组件的被序列化的数据流(serialized data stream)。一个prefab实例(instance)实际上是一组应该被应用到这个被序列化数据实例的修改内容(A prefab instance is a list of modifications that should be made on the serialized data for this instance.)。Prefab这个概念实际上只存在于Editor阶段，当Unity开始Build工程的时候就会把这些修改应用到这些被序列化的数据流上，并且当这些修改被实例化，被实例化的数据对象并不会知道它们曾经是Editor编辑器中的Prefab。
- **Instantiation.**
    - 当你调用`Instantiate()`函数，对prefab，scene中的gameobject，或者是其它的能被序列化的数据(everything that derives from UnityEngine.Object can be serialized)的操作结果，我们将会序列化对象，然后创建一个新的对象，然后反序列化对象数据到新的对象数据中。
        > (We then run the same serialization code again in a different variant, where we use it to report which other UnityEngine.Object’s are being referenced. We then check for all referenced UnityEngine.Object’s if they are part of the data being Instantiated(). If the reference is pointing to something “external” (like a texture) we keep that reference as it is, if it is pointing to something “internal” (like a child gameobject), we patch the reference to the corresponding copy).
        
        这里我直接贴了英文原文，感觉翻译不出人家的意思。要注意的是：
        - Variant: 在unity编辑器中偶尔会看见这个名字，在我prefab中嵌套另一个prefab的时候印象最深。按照上面的说法，这个是用来通知被引用的UnityEngine.Object's，然后同样对这些对象执行上述的序列化代码。
        - 如果引用的内容是外部数据("external")将会保留原有的引用。
        - 如果引用的内容是内部数据("internal")将会将引用替换为原有数据的副本。
        - 在这里，在这篇文章的评论部分有提及，作者举了一个例子来说明这个过程：
        > The scenario you mean is when you call Instantiate() on something. Let’s take this example. There are three objects.
        >
        > O1: GameObject components=O2, O3
        > O2: RigidBody
        > O3: BoxCollider
        >
        > when you invoke Instatiate(gameObject1), we duplicate all three objects.
        >
        > O4: GameObject components=O2, O3
        > O5: RigidBody
        > O6: BoxCollider
        > 
        > Notice how the cloned object O4, actually points to O2 and O3 in its component list. this is obviously not what you intended. In the second phase of Instantiate, we fix this up, by running the serializer in a special mode on O1,O2&O3. we ask it “please report your object references”, and then we check if any of the objects referenced were included in the list of objects that were cloned. For each reference that referenced an object that was cloned (both entries in the componentlist in our case), we update that reference to the cloned version instead of the original. after the fix it looks like this:
        > 
        > O4: GameObject components=O5, O6
        > O5: RigidBody
        > O6: BoxCollider

- **Saving.**
    - 如果设置了"force text serialization"，并且用文本编辑器打开`.unity`的scene文件，**we run the serializer with a ymal backend.**
- **Loading.**
    - 向后兼容加载（backwards compatible loading）也是基于系列化机制(serialization)的系统。In-Editor的yaml loading利用了序列化机制，运行时加载scenes和assets也利用了这个。Assetbundles也利用了序列化系统(serialization system)。
- **Hot reloading of editor code.**
    - 当你改变了editor脚本数据，我们会序列化所有的editor窗口（它们都继承自UnityEngine.Object!），然后我们销毁所有的窗口，unload掉所有的旧的C#代码，加载新的C#代码，重新创建窗口，并在最后反序列化数据流中的数据到新的窗口。
- **Resource.GarabageCollectSharedAssets().**
    - Unity中使用的GC(native garabage collector)，不同于C#所使用的GC。我们使用这个系统，当加载一个scene，之前的scene未引用的内容就会被unload掉。这种GC会在某种模式下运行序列器(serializer)，在这个模式下我们用它来让对象通知所有的引用到外部的UnityEngine.Objects。这就是为什么我们在scene1中使用的textures会在scene2中被unload掉。
    > This is our native garbage collector and is different to the C# garbage collector. It is the thing that we run after you load a scene to figure out which things from the previous scene are no longer referenced, so we can unload them. The native garbage collector runs the serializer in a mode where we use it to have objects report all references to external UnityEngine.Objects. This is what makes textures that were used by scene1, get unloaded when you load scene2.

> The serialization system is written in C++, we use it for all our internal object types (Textures, AnimationClip, Camera, etc). Serialization happens at the UnityEngine.Object level, each UnityEngine.Object is always serialized as a whole. They can contain references to other UnityEngine.Objects and those references get serialized properly.

由于一些执行效率的需要，serializer的行为不完全是如你所愿，比如说MonoBehaviour component的序列化是由你写的脚本所支持的，所以了解serializer的运行细节能够让你更好的使用它。

**为了能让我们写的脚本中的序列化区域(a fieldof my script)能够被序列化需要什么条件？**
- Be public,or have [SerializeField] attribute
- Not be static
- Not be const
- Not be readonly
- The fieldtype needs to be a type that we can serialize.

**什么样的类型(fieldtype)能被序列化呢？**
- Custom non abstrace classes with [Serializable] attribute.
- Custom structs with [Serializable] attribute. (new inUnity4.5)
- References to objects that derive from UnityEngine.Object
- Primitive data dypes(int,float,double,bool,string,etc)
- Array of a fieldtype we can serialize
- List<T> of a fieldtype we can serialize

那么什么情况是，serializer的动作会是与我们的期望有所不同呢？
```csharp
[Serializable]
class Animal
{
    public string name;
}

class MyScript : MonoBehaviour
{
    public Animal[] animals;
}
```

*当我们试着在数组`animals`加入三个相同的`Animal object`对象引用的时候，即三个引用都指向相同的对象。在已序列化的数据流中你会发现三个对象，当你进行反序列化会发现那里有三个不同的对象。*

当你需要需要序列化一个比较复杂的引用的对象列表，这个时候就不能依靠Unity的serializer。你需要做些操作为了能让这些数据正常序列化。

但是需要注意的是，这个情况只适应于自定义类(custonm classes)，因为他们是被内联的序列化（serialized "inline"）,因为他们的数据成为了既存的MonoBehaviour中的完整的序列化数据的一部分。当你的变量区域有一个UnityEngine.Object的派生类引用，那么数据并不是内联序列化，那么其实际的引用数据会被成功的序列化。
> Note that this is only true for custom classes, as they are serialized “inline” because their data becomes part of the complete serializationdata for the MonoBehaviour they are used in. When you have fields that have a reference to something that is a UnityEngine.Object derived class, like a “public Camera myCamera”, the data from that camera are not serialized inline, and an actual reference to the camera UnityEngine.Object is serialized.

(我担心自己的理解有偏差还是把英文原文贴了出来。)

**No support for null for custom classes**

> The serializer does not support null. If it serializes an object and a field is null, we just instantiate a new object of that type and serialize that. Obviously this could lead to infinite cycles, so we have a relatively magical depth limit of 7 levels. At that point we just stop serializing fields that have types of custom classes/structs and lists and arrays. [1]

Serializer不支持null类型的序列化，如果执行这样的操作了就会导致死循环，unity在这种死循环中加入了最大的生成层次。在Unity4.5之后的版本加入了警告的信息。

**No support for polymorphism**

不支持多态。
```
public Animal[] animals;
```
在里面加入一个`dog`,`cat`,`giraffe`三个派生实例，在序列化之后我们得到的是三个`Animal`类型的实例。

这会发生在自定义类("custom classes")的内联序列化(get serialized inline)过程中。当对象实例是UnityEngine.Object's派生的实例多态的特性还是有效的。

> You’d make a ScriptableObject derived class or another MonoBehaviour derived class, and reference that. The downside of doing this, is that you need to store that monobehaviour or scriptable object somewhere and cannot serialize it inline nicely.

*你可以利用ScriptableObject派生类或者另一个MonoBehavior派生类，并引用这个生成的实例。这种方式的缺点是，你必须在某个地方存储monobehaviour或者scriptable对象，而不能很好的内联序列化(serailized inline)。*

产生这样的限制的原因是，序列化系统的一个核心功能之一的实现，了解一个对象的数据结构依靠的是类的类型，而不是运行时被存储在这个类区域的数据类型。
> The reason for these limitations is that one of the core foundations of the serialization system is that the layout of the datastream for an object is known ahead of time, and depends on the types of the fields of the class, instead of what happens to be stored inside the fields.

**那么如何序列化一些Uinty不支持的数据类型？**

大多情况下可以使用**serialization callbacks**。它们会在serializer开始读取你的自定义数据区域之前通知你，并在serializer结束之后写入数据。（应该就是上面说的序列化之前存储monobehaviour或者scriptable对象，并在序列化完成之后将数据写入。）
> In many cases the best approach is to use serialization callbacks. They allow you to be notified before the serializer reads data from your fields and after it is done writing to them. You can use this to have a different representation of your hard-to-serialize data at runtime than when you actually serialize. You’d use these to transform your data into something Unity understands right before Unity wants to serialize it, you also use it to transform the serialized form back into the form you’d like to have your data in at runtime, right after Unity has written the data to your fields.

让我们创建一个树状的数据的结构(tree datastructure)，如果你直接让Unity去序列化这样的数据，"no support for null"的限制会让你的数据流变得非常大，导致一些程序效率的问题。
> Let’s say you want to have a tree datastructure. If you let Unity directly serialize the data structure, the “no support for null” limitation would cause your datastream to become very big, leading to performance degradations in many systems:

```csharp
using UnityEngine;
using System.Collections.Generic;
using System;
 
public class VerySlowBehaviourDoNotDoThis : MonoBehaviour
{
    [Serializable]
    public class Node
    {
        public string interestingValue = "value";
        
       //The field below is what makes the serialization data become huge because
       //it introduces a 'class cycle'.
       public List&lt;Node&gt; children = new List&lt;Node&gt;();
    }
 
    //this gets serialized
    public Node root = new Node();
 
    void OnGUI()
    {
        Display (root);
    }
 
    void Display(Node node)
    {
        GUILayout.Label ("Value: ");
        node.interestingValue = GUILayout.TextField(node.interestingValue, GUILayout.Width(200));
 
        GUILayout.BeginHorizontal ();
        GUILayout.Space (20);
        GUILayout.BeginVertical ();
 
        foreach (var child in node.children)
            Display (child);
 
        if (GUILayout.Button ("Add child"))
            node.children.Add (new Node ());
 
        GUILayout.EndVertical ();
        GUILayout.EndHorizontal ();
    }
}
```

> Instead, you tell Unity not to serialize the tree directly, and you make a seperate field to store the tree in a serialized format, suited for Unity’s serializer:

为serializer添加自定义数据的序列化操作

```csharp
using UnityEngine;
using System.Collections.Generic;
using System;

public class BehaviourWithTree : MonoBehaviour, ISerializationCallbackReceiver
{
    //node class that is used at runtime
    public class Node
    {
        public string interestingValue = "value";
        public List&lt;Node&gt; children = new List&lt;Node&gt;();
    }

    //node class that we will use for serialization
    [Serializable]
    public struct SerializableNode
    {
        public string interestingValue;
        public int childCount;
        public int indexOfFirstChild;
    }

    //the root of what we use at runtime. not serialized.
    Node root = new Node();

    //the field we give unity to serialize.
    public List&lt;SerializableNode&gt; serializedNodes;

    public void OnBeforeSerialize()
    {
        //unity is about to read the serializedNodes field's contents. lets make sure
        //we write out the correct data into that field "just in time".
        serializedNodes.Clear();
        AddNodeToSerializedNodes(root);
    }

    void AddNodeToSerializedNodes(Node n)
    {
        var serializedNode = new SerializableNode () {
            interestingValue = n.interestingValue,
            childCount = n.children.Count,
            indexOfFirstChild = serializedNodes.Count+1
        };

        serializedNodes.Add (serializedNode);
        foreach (var child in n.children)
            AddNodeToSerializedNodes (child);
    }

    public void OnAfterDeserialize()
    {
        //Unity has just written new data into the serializedNodes field.
        //let's populate our actual runtime data with those new values.

        if (serializedNodes.Count &gt; 0)
            root = ReadNodeFromSerializedNodes (0);
        else
            root = new Node ();
    }

    Node ReadNodeFromSerializedNodes(int index)
    {
        var serializedNode = serializedNodes [index];
        var children = new List&lt;Node&gt; ();
        for(int i=0; i!= serializedNode.childCount; i++)
            children.Add(ReadNodeFromSerializedNodes(serializedNode.indexOfFirstChild + i));

        return new Node() {
            interestingValue = serializedNode.interestingValue,
            children = children
        };
    }

    void OnGUI()
    {
        Display (root);
    }

    void Display(Node node)
    {
        GUILayout.Label ("Value: ");
        node.interestingValue = GUILayout.TextField(node.interestingValue, GUILayout.Width(200));

        GUILayout.BeginHorizontal ();
        GUILayout.Space (20);
        GUILayout.BeginVertical ();

        foreach (var child in node.children)
            Display (child);

        if (GUILayout.Button ("Add child"))
            node.children.Add (new Node ());

        GUILayout.EndVertical ();
        GUILayout.EndHorizontal ();
    }
}
```

> Beware that the serializer, including these callbacks coming from the serializer, usually do not run on the main thread, so you are very limited in what you can do in terms of invoking Unity API. (Serialization happening as part of loading a scene happens on a loading thread. Serialization happening as part of you invoking Instantiate() from script happens on the main thread). You can however do the necessary data transformations do get your data from a non-unity-serializer-friendly format to a unity-serializer-friendly-format.

需要注意的是，这些回调通常并不会运行在主线程中，所以当调用UnityAPI的时候就需要注意加一些限制。（当是scene加载过程中的序列化是loading thread中发生的，你调用`Instantiate()`函数过程中的序列化是在主线程中发生的。）

通过上面的手段就可以实现一些Unity不支持的数据类型的序列化。

这个时候我就会问**List&lt;Node&gt;**是个什么，查了一下，就是`<`,`>`的表示。[HTML特殊文字コード表](http://www.shurey.com/js/labo/character.html)。

额...

看了一下时间，是2014年的文章，现在已经2020年了...不过自定义序列化数据之前的东西还是没变的感觉，仍是非常有用，对我来说。


- [【Unity】【エディタ拡張】SerializedObjectの勘所をまとめてみる](http://light11.hatenadiary.com/entry/2018/03/15/225709)

