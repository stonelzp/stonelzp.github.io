---
title: UE4中的智能指针
date: 2018-07-31 19:59:19
categories:
- UnrealEngine4
tags:
- UE4
- C++
---
对于内存使用的了解，就不得不了解指针。UE4拥有跟C++类似的智能指针，在这里对虚幻4的智能指针库的内容进行一些总结跟提炼，同时也需要对C++的智能指针进行深入的了解。

<!--more-->

# UE4的智能指针
智能指针并不能使用`UPROPERTY()`，`TSharedRef`,`TSharedPtr`,`TWeakPtr`等等。

上一次更新这篇文章是两年前，都两年了我还没深入了解UE4的智能指针，我也不知道我都干了些什么。

我应该对UE4的智能指针进行分类探讨，根据自己遇到过的情况。

## Unreal Smart Pointer Library
- [Unreal Smart Pointer Library](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/)

参考了UE的官方文档内容。
UE4的智能指针库(smart pointer library)是C++11智能指针的自定义实现。实现包含了标准的**SharedPointers**,**WeakPointers**,**UniquePointers**。 **SharedReferences**则表现为非空的SharedPointers。

这些指针都不能作用于UE的`UObject`系统对象中，因为UE的UObject使用的是单独的内存追踪系统。(更适合游戏代码的系统，官网上如是说)

### SharedPointers
共享指针(`TSharedPtr`)拥有其指向的对象，如果指向该对象的所有共享指针和共享引用都不存在了的情况下负责该对象的销毁，否则该对象将一直被视为无法销毁的对象。
一个非空的共享指针可以为其对象生成共享引用。

具体内容可以参照官方文档：
- [Shared Pointers](https://docs.unrealengine.com/4.27/en-US/ProgrammingAndScripting/ProgrammingWithCPP/UnrealArchitecture/SmartPointerLibrary/SharedPointer/)

### SharedReferences
共享引用指针(`TSharedRef`)使用跟共享指针类似，但是指向对象不能为空。
共享引用可以转换为共享指针，而且该共享指针指向的对象一定不会为空。

具体内容可以参照官方文档：
-[]()

### WeakPointers
弱指针(`TWeakPtr`)不拥有其指向的对象，也就是说其指向的对象随时都有可能变为空。

具体内容可以参照官方文档：
-[]()
### UniquePointers
唯一指针(`TUniquePtr`)
> A Unique Pointer solely and explicitly owns the object it references. Since there can only be one Unique Pointer to a given resource, Unique Pointers can transfer ownership, but cannot share it. Any attempts to copy a Unique Pointer will result in a compile error. When a Unique Pointer is goes out of scope, it will automatically delete the object it references.

### 实现细节

#### 侵入式的访问器(Intrusive Accessors)
共享指针是非侵入式的，什么意思呢，就是对象本身并不知道自己是否被某个共享指针所拥有。
但是在程序设计的时候，将对象作为共享指针或共享引用来进行访问的时候会很方便，于是便有了这种侵入式的访问器的实现。

`TSharedFromThis`类提供了两个函数,这里需要注意的是，这是一个类，不是结构体（我有一次就想在结构体继承这个类=-=）。
- `AsShared()`
- `SharedThis()`

使用上面的两种方式将对象转换为指针引用，然后根据需要转换为共享指针。

官网上的示例代码：
```
class FRegistryObject;
class FMyBaseClass: public TSharedFromThis<FMyBaseClass>
{
    virtual void RegisterAsBaseClass(FRegistryObject* RegistryObject)
    {
        // 访问对"this"的共享引用。
        // 直接继承自< TSharedFromThis >，因此AsShared()和SharedThis(this)会返回相同的类型。
        TSharedRef<FMyBaseClass> ThisAsSharedRef = AsShared();
        // RegistryObject需要 TSharedRef<FMyBaseClass>，或TSharedPtr<FMyBaseClass>。TSharedRef可被隐式转换为TSharedPtr.
        RegistryObject->Register(ThisAsSharedRef);
    }
};
class FMyDerivedClass : public FMyBaseClass
{
    virtual void Register(FRegistryObject* RegistryObject) override
    {
        // 并非直接继承自TSharedFromThis<>，因此AsShared()和SharedThis(this)不会返回相同类型。
        // 在本例中，AsShared()会返回在TSharedFromThis<> - TSharedRef<FMyBaseClass>中初始指定的类型。
        // 在本例中，SharedThis(this)会返回具备"this"类型的TSharedRef - TSharedRef<FMyDerivedClass>。
        // SharedThis()函数仅在与 'this'指针相同的范围内可用。
        TSharedRef<FMyDerivedClass> AsSharedRef = SharedThis(this);
        // FMyDerivedClass是FMyBaseClass的一种类型，因此RegistryObject将接受TSharedRef<FMyDerivedClass>。
        RegistryObject->Register(ThisAsSharedRef);
    }
};
class FRegistryObject
{
    // 此函数将接受到FMyBaseClass或其子类的TSharedRef或TSharedPtr。
    void Register(TSharedRef<FMyBaseClass>);
};
```

#### Casting
> You can cast Shared Pointers (and Shared References) through several support functions included in the Unreal Smart Pointer Library. Up-casting is implicit, as with C++ pointers. You can const cast with the ConstCastSharedPtr function, and static cast (often to downcast to derived class pointers) with StaticCastSharedPtr. Dynamic casting is not supported, as there is no run-type type information (RTTI); static casting should be used instead, as in the following code:
```
// This assumes we validated that the FDragDropOperation is actually an FAssetDragDropOp through other means.
TSharedPtr<FDragDropOperation> Operation = DragDropEvent.GetOperation();
// We can now cast with StaticCastSharedPtr.
TSharedPtr<FAssetDragDropOp> DragDropOp = StaticCastSharedPtr<FAssetDragDropOp>(Operation);
```

#### 限制
- Avoid passing data to functions as TSharedRef or TSharedPtr parameters where possible, as these will incur overhead by dereferencing and reference-counting. Instead, pass the referenced object, preferably as a `const &`.

- You can forward-declare Shared Pointers to incomplete types.

- Shared Pointers are not compatible with Unreal objects (UObject and its derived classes). The Engine has a separate memory management system (see Object Handling documentation) for UObject management, and the two systems have no overlap with each other.

第一条限制对于刚开始准备利用共享指针的我无疑是一个很好的警示。

### 源码文档
去`Source\Runtime\Core\Public\Templates\SharedPointer.h`文件里有关于UE智能指针的一些文档，我参考的是UE5的源码。

源码中的文档内容：
```
/**
 *	SharedPointer - Unreal smart pointer library
 *
 *	This is a smart pointer library consisting of shared references (TSharedRef), shared pointers (TSharedPtr),
 *	weak pointers (TWeakPtr) as well as related helper functions and classes.  This implementation is modeled
 *	after the C++0x standard library's shared_ptr as well as Boost smart pointers.
 *
 *	Benefits of using shared references and pointers:
 *
 *		Clean syntax.  You can copy, dereference and compare shared pointers just like regular C++ pointers.
 *		Prevents memory leaks.  Resources are destroyed automatically when there are no more shared references.
 *		Weak referencing.  Weak pointers allow you to safely check when an object has been destroyed.
 *		Thread safety.  Includes "thread safe" version that can be safely accessed from multiple threads.
 *		Ubiquitous.  You can create shared pointers to virtually *any* type of object.
 *		Runtime safety.  Shared references are never null and can always be dereferenced.
 *		No reference cycles.  Use weak pointers to break reference cycles.
 *		Confers intent.  You can easily tell an object *owner* from an *observer*.
 *		Performance.  Shared pointers have minimal overhead.  All operations are constant-time.
 *		Robust features.  Supports 'const', forward declarations to incomplete types, type-casting, etc.
 *		Memory.  Only twice the size of a C++ pointer in 64-bit (plus a shared 16-byte reference controller.)
 *
 *
 *	This library contains the following smart pointers:
 *
 *		TSharedRef - Non-nullable, reference counted non-intrusive authoritative smart pointer
 *		TSharedPtr - Reference counted non-intrusive authoritative smart pointer
 *		TWeakPtr - Reference counted non-intrusive weak pointer reference
 *
 *
 *	Additionally, the following helper classes and functions are defined:
 *
 *		MakeShareable() - Used to initialize shared pointers from C++ pointers (enables implicit conversion)
 *		TSharedFromThis - You can derive your own class from this to acquire a TSharedRef from "this"
 *		StaticCastSharedRef() - Static cast utility function, typically used to downcast to a derived type.
 *		ConstCastSharedRef() - Converts a 'const' reference to 'mutable' smart reference
 *		StaticCastSharedPtr() - Dynamic cast utility function, typically used to downcast to a derived type.
 *		ConstCastSharedPtr() - Converts a 'const' smart pointer to 'mutable' smart pointer
 *
 *
 *	Examples:
 *		- Please see 'SharedPointerTesting.inl' for various examples of shared pointers and references!
 *
 *
 *	Tips:
 *		- Use TSharedRef instead of TSharedPtr whenever possible -- it can never be nullptr!
 *		- You can call TSharedPtr::Reset() to release a reference to your object (and potentially deallocate)
 *		- Use the MakeShareable() helper function to implicitly convert to TSharedRefs or TSharedPtrs
 *		- You can never reset a TSharedRef or assign it to nullptr, but you can assign it a new object
 *		- Shared pointers assume ownership of objects -- no need to call delete yourself!
 *		- Usually you should "operator new" when passing a C++ pointer to a new shared pointer
 *		- Use TSharedRef or TSharedPtr when passing smart pointers as function parameters, not TWeakPtr
 *		- The "thread-safe" versions of smart pointers are a bit slower -- only use them when needed
 *		- You can forward declare shared pointers to incomplete types, just how you'd expect to!
 *		- Shared pointers of compatible types will be converted implicitly (e.g. upcasting)
 *		- You can create a typedef to TSharedRef< MyClass > to make it easier to type
 *		- For best performance, minimize calls to TWeakPtr::Pin (or conversions to TSharedRef/TSharedPtr)
 *		- Your class can return itself as a shared reference if you derive from TSharedFromThis
 *		- To downcast a pointer to a derived object class, to the StaticCastSharedPtr function
 *		- 'const' objects are fully supported with shared pointers!
 *		- You can make a 'const' shared pointer mutable using the ConstCastSharedPtr function
 *		
 *
 *	Limitations:
 *
 *		- Shared pointers are not compatible with Unreal objects (UObject classes)!
 *		- Currently only types with that have regular destructors (no custom deleters)
 *		- Dynamically-allocated arrays are not supported yet (e.g. MakeSharable( new int32[20] ))
 *		- Implicit conversion of TSharedPtr/TSharedRef to bool is not supported yet
 *
 *
 *	Differences from other implementations (e.g. boost:shared_ptr, std::shared_ptr):
 *
 *		- Type names and method names are more consistent with Unreal's codebase
 *		- You must use Pin() to convert weak pointers to shared pointers (no explicit constructor)
 *		- Thread-safety features are optional instead of forced
 *		- TSharedFromThis returns a shared *reference*, not a shared *pointer*
 *		- Some features were omitted (e.g. use_count(), unique(), etc.)
 *		- No exceptions are allowed (all related features have been omitted)
 *		- Custom allocators and custom delete functions are not supported yet
 *		- Our implementation supports non-nullable smart pointers (TSharedRef)
 *		- Several other new features added, such as MakeShareable and nullptr assignment
 *
 *
 *	Why did we write our own Unreal shared pointer instead of using available alternatives?
 *
 *		- std::shared_ptr (and even tr1::shared_ptr) is not yet available on all platforms
 *		- Allows for a more consistent implementation on all compilers and platforms
 *		- Can work seamlessly with other Unreal containers and types
 *		- Better control over platform specifics, including threading and optimizations
 *		- We want thread-safety features to be optional (for performance)
 *		- We've added our own improvements (MakeShareable, assign to nullptr, etc.)
 *		- Exceptions were not needed nor desired in our implementation
 *		- We wanted more control over performance (inlining, memory, use of virtuals, etc.)
 *		- Potentially easier to debug (liberal code comments, etc.)
 *		- Prefer not to introduce new third party dependencies when not needed
 *
 */
```

文档中的内总结的相当好了，我就不继续画蛇添足了。

这里提到了`SharedPointerTesting.inl`文件，里面有很多实现的样例代码，实现的时候好好参照这个文件。


## TWeakPtr


## TWeakObjectPtr
关于`TWeakPtr`和`TweakObjectPtr`之间的区别：

**TWeakObjectPtr**是针对`UObjects`的弱指针引用，**TWeakPtr**则是针对除了`UObjects`的弱指针引用。
> TWeakObjectPtr is for weak pointers to UObjects, TWeakPtr for pointers to everything else.
>
> Since UObjects are garbage collected and shared pointers are reference counted, we cannot have the same weak pointer type for all, unfortunately.

- [Difference between TWeakPtr and TWeakObjectPtr?](https://answers.unrealengine.com/questions/298868/difference-between-tweakptr-and-tweakobjectptr.html)


参考链接：
- [Unreal Smart Pointer Library](https://api.unrealengine.com/INT/Programming/UnrealArchitecture/SmartPointerLibrary/index.html)

# C++的智能指针
指针的使用伴随着内存泄漏(memory leak)的问题，可能会发生内存泄漏的情况有：
- new或者malloc出来的内存因为程序员的疏忽忘记释放
- 程序运行发生错误(throw)，未能执行内存释放程序

所以不是说只要程序员足够谨慎就能够避免指针造成的内存泄漏的问题。

## C++11中的智能指针
主要在用的智能指针有：unique_ptr, shared_ptr, weak_ptr。

这3种指针组件就是采用了boost里的智能指针方案。很多有用过boost智能指针的朋友，很容易地就能发现它们之间的关系：

| std | boost | 功能说明 |
| ---- | ---- | ---- |
| unique_ptr | scoped_ptr | 独占指针对象，并保证指针所指对象生命周期与其一致 |
| shared_ptr |shared_ptr |可共享指针对象，可以赋值给shared_ptr或weak_ptr。<br>指针所指对象在所有的相关联的shared_ptr生命周期结束时结束，是强引用。 |
| weak_ptr | weak_ptr | 它不能决定所指对象的生命周期，引用所指对象时，需要lock()成shared_ptr才能使用。 |

参考链接：
- [C++11中的智能指针](https://my.oschina.net/hevakelcj/blog/465978)

## 三种智能指针的特性用法

参考链接：
- [C++11及C++14标准的智能指针](https://blog.csdn.net/haolexiao/article/details/56773039)

### weak_ptr
`std::weak_ptr`是一个很好的解决**悬空指针**问题的方式。使用原生指针（raw pointers）的话不知道现在所引用的资源是否已经被释放。而使
用`std::shared_ptr`来管理的话，`std::weak_ptr`只管使用，而不关心资源的使用情况，反正也不管理指向的资源。

因为本身`std::weak_ptr`并不能直接引用到对象，不会影响对象的自动释放，不会影响对象的引用计数，需要使用`lock()`来升级到`std::shared_ptr`来进行操作。

参考资料：
- [C++ weak pointer](https://my.oschina.net/u/2368952/blog/416606)
- [When is std::weak_ptr useful?](https://stackoverflow.com/questions/12030650/when-is-stdweak-ptr-useful)

## 比起直接使用new优先使用std::make_unique和std::make_shared

参考链接：
- [Item 21: 比起直接使用new优先使用std::make_unique和std::make_shared](https://www.cnblogs.com/boydfd/p/5146432.html)
