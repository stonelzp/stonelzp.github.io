---
title: UE4中的Serialization
date: 2021-06-07 22:53:31
permalink: ue4-serialization

categories:
- UnrealEngine4

tags:
- UE4
---
这一次要好好的理解UE4中的Serialization这个概念。
<!--more-->

# Serialization
序列化的概念和UE4的反射概念息息相关，而且我在另一篇*UE4-反射机制*这篇文章稍微展开了一点内容，这一篇主要是对UE4.25的新功能**UnversionedPropertySerialization**进行深入了解的同时，希望对序列化的概念如何在UE4中发挥作用这个问题有更深入的了解。

## 何为Serialization（序列化）
在其他的文章应该也有提到过就是序列化这个概念并不是UE4独有的，而是整个程序领域中存在的概念。

资料中举了一个例子：`ABananaCharacter`:
![Serialization_Sample01](SerializationSample01.png)

 可以看到UE4中分为**Property**和普通的**C++变量**两个部分。区别就是普通的C++变量只能在RunTime的时候进行读写，而Property则可以通过UE4的Editor蓝图等进行修改等操作。

 持有Property属性的变量，会被保存在`UClass`这个容器中，这个UClass就像是这些个属性的说明书一样，由于UClass会将所有的属性必要的信息保存起来，我们可以很容易的通过它来追踪到各个属性在内存里的状态，保存的值。而将这些必要的信息进行收集方便我们处理，以便于之后追踪各个属性变量在内存中展开的状态的过程，就是序列化。可以进行序列化的对象非常的多，包括但不限于普通的变量，容器，复杂的结构体(UStruct等等)。当这个过程结束之后，便可以得到一串加工过的连续的数据。
![Serialization_Sample02](SerializationSample02.png)

再附上日语的解说：
![Serialization_Sample03](SerializationSample03.png)

反序列化就是将上面序列化过的数据重新复原到内存上的过程。
![Deserialization_Sample04](SerializationSample04.png)

了解了序列化的工作之后，UE4又是如何利用这个特性的呢？或者说UE4用这个特性来做什么呢？
图片中的内容不是全部：
![Deserialization_Sample05](SerializationSample05.png)

## UE4中的Serializer
目前在UE4中完成序列化处理的Serializer有两种，应该说是UE4.25开始导入了第二种Serializer
![Deserialization_Sample06](SerializationSample06.png)

### TaggedPropertySerializer(TPS)
在4.25之前，UE4使用的都是TPS这个Serializer，相比于UPS来说，操作的代价比较昂贵。

TaggedPropertySerializer:
- Editor的Asset保存和读取等操作
- Cook时Asset的保存
- Cook済みビルドでアセットを読み込む(具体我还是不知道是个什么阶段)

UPS导入之后，后两个操作则可以由UPS来处理了。Cook完后的Asset操作使用UPS的话，效率就会大大提升，相比于原来的Serializer来说。

#### 序列化过程(TPS)
简单说明一下TPS的序列化过程：
![Deserialization_Sample07](SerializationSample07.png)

结合上面的图片，TPS首先找到`UClass`中的持有`FProperty`属性的变量，这个FProperty属性保存着这个变量的**名字**，**类型**，**类中的位置**，**meta修饰符数据** 等等的数据情报。根据变量的FProperty属性，TPS会为其创建一个`FPropertyTag`的数据。随后Serializer将创建好的`FPropertyTag`数据和经过特定处理的已经直列化(日语直列化的说法，就是数据被整齐的排成没有多余空间的连续的数据)的数据针对每一个变量都一起保存到Asset文件(uasset)中。

这个`FPropertyTag`是这样的东西
![Deserialization_Sample08](SerializationSample08.png)

`FPropertyTag`里面包含了很多的信息，或者说，`FPropertyTag`包含了GUID数据的结构等数据，使得在我们对其数据结构进行修改，升级UE4版本或者对已经Release的游戏版本的数据结构进行修改的时候，我们的数据也不会丢失。比如说游戏内的各种设置设定，都不会消失，这也是UE4的引擎在背后默默实现的重要功能之一。

#### 反序列化过程(TPS)
反序列化的过程就是跟上面的相反

首先是从Asset文件(uasset)中取出直列化的数据和`FPropertyTag`数据：
![Deserialization_Sample09](SerializationSample09.png)

然后根据`FPropertyTag`中保存的数据(GUID和名字等数据)在UClass中进行检索，找到了相应的位置数据之后，按照顺序将每一个变量的直列化数据展开到内存中数据对象的适当位置。
![Deserialization_Sample10](SerializationSample10.png)
![Deserialization_Sample11](SerializationSample11.png)

#### 总结
TPS实现的内容：
- PropertyTag的存在使得数据等到了更好的**互换性**
- 需要对每一个变量添加一个PropertyTag
- 将数据再展开到内存的时候会有检索开销

![Deserialization_Sample12](SerializationSample12.png)

### UnversionedPropertySerializer(UPS)
来到UPS的方式，则主要是为了应对以下的问题：
![Deserialization_Sample13](SerializationSample13.png)

#### 序列化过程(UPS)
首先是按照顺序收集UClass中的FProperty情报，然后将数据直列化之后，按照顺序直接保存到Asset(uasset)文件中去。(这里没有考虑无效数据的情况，直列化的数据都是有效数据的情况，如果存在无效数据的时候是什么样的视频里面没有说)
![Deserialization_Sample14](SerializationSample14.png)

当所有的FProperty的直列化数据都保存之后，为这些直列化完毕的数据生成**Header**数据情报。
![Deserialization_Sample15](SerializationSample15.png)

#### 反序列化过程(UPS)
反序列化的过程就是上面的相反的操作，根据生成的**Header**的情报将直列化的数据按照顺序展开到对象的内存数据中去。

视频中倒是没有说明这个关键的**Header**中具体保存了什么样的数据。

#### 总结
从序列化的过程就可以看出来UPS是一种超级高速的简单的实装方式。但是简单就意味着使用过程中如果不注意一些东西就会出现错误。

料想到的容易发生的case就是Editor编译的对象与Cook完毕之后的Asset文件对象数据结构发生改变的情形：
![Deserialization_Sample16](SerializationSample16.png)

由于UPS的序列化过程和反序列化过程并没有提供数据结构发生修改的互换性，所以当反序列化的时候将数据展开到内存的时候就会发生严重的错误。
![Deserialization_Sample17](SerializationSample17.png)

很明显TPS方式应该就可以对应这种情况，当我们使用了UPS的方式来对Asset进行Cook的时候发生了上面那种事情的话，首先Debug的第一个就是先看看有没有会发生上面那种操作的代码。

#### UPS的使用方式
这种序列化的方式由上面的内容可以知道这是针对Cook素材的内容，同时想要使用的时候也需要手动开启。

开启的方式有两种：
![Deserialization_Sample18](SerializationSample18.png)

关于`-unversion`的拓展知识：
当我们创建新的Asset的时候，UE4的版本内容也会被包含在Asset的信息里面，当我们进行引擎的升级等的操作之后，版本情报也会改变，会导致当我们在进行Biniary的差分Patch的时候，明明没有对Asset进行额外的数据修改却使得Patch的体积变得很大。
这个`-unversion`option会把表示引擎版本的数值变得无效，代表现在的版本一直都是最新的版本。

不光是使用UPS的情况，用到需要进行差分Patch的场合或者一些我不知道的场合的时候，都是推荐使用的。

视频的剩下内容是对UPS方式的效率进行了验证，我就不赘述了。

最后是视频的总结：
![Deserialization_Sample18](SerializationSample18.png)

参考资料：
- [【UE4.25 新機能】新しいシリアライゼーション機能「Unversioned Property Serialization」について](https://www.youtube.com/watch?v=V5tUNlfiJ5s)
- [【UE4.25 新機能】新しいシリアライゼーション機能「Unversioned Property Serialization」について](https://www2.slideshare.net/EpicGamesJapan/new-loadingsystem-unversionedpropertyserializationv2/)

# NetWorking Serialization
关于Networking的序列化操作是一个很久之前就困扰我的谜题，有那么一段代码我经常会看见，知道是为了什么而存在的但是不知道需要怎么用，这次就让我彻底的了解这些个代码。

## Custom Struct Serialization
关于UE4中的自定义的`USTRUCT`类型的网络序列化问题，为了缩减RPC调用的带宽(bandwidth)，UE4提供了很强大的序列化功能。但是这不是自动的，需要我们做一些设置。

很早以前就一直发现但是一直没有仔细看和整理的文章
- [Custom Struct Serialization for Networking in Unreal Engine](http://www.aclockworkberry.com/custom-struct-serialization-for-networking-in-unreal-engine/)

当我们使用UE4提供的**USTRUCT**自定义了一个结构体之后，我们可以为其添加一个`NetSerialize`函数，来为UE4的Networking中**属性复制(Properties Replication)**和**RPC**提供序列化(Serialization)和反序列化(Deserialization)方法。

这是基于UE4所提供的**struct trait system**之上的。

关于如何使用这个函数其实在UE4的源码中也有大量的使用案例和Mannual，最集中的就是源码了。
- **Runtime/Engine/Classes/Engine/NetSerialization.h**

关于这个方法的定义：
```c++
/**
 * @param Ar FArchive to read or write from.
 * @param Map PackageMap used to resolve references to UObject*
 * @param bOutSuccess return value to signify if the serialization was succesfull (if false, an error will be logged by the calling function)
 *
 * @return return true if the serialization was fully mapped. If false, the property will be considered 'dirty' and will replicate again on the next update.
 * This is needed for UActor* properties. If an actor's Actorchannel is not fully mapped, properties referencing it must stay dirty.
 * Note that UPackageMap::SerializeObject returns false if an object is unmapped. Generally, you will want to return false from your ::NetSerialize
 * if you make any calls to ::SerializeObject that return false.
 *
 */
 bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess);
```

结构体中的使用：
```c++
USTRUCT()
struct FMyCustomNetSerializableStruct
{
	UPROPERTY()
	float SomeProperty;

	bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess);
}

template<>
struct TStructOpsTypeTraits<FMyCustomNetSerializableStruct> : public TStructOpsTypeTraitsBase2<FMyCustomNetSerializableStruct>
{
	enum
	{
		WithNetSerializer = true
	};
};
```
上面的模板类型匹配是UE4会自动调用我们自定义结构体中`NetSerialize`函数的关键，如果我们不提供这样的类型匹配不将`WithNetSerializer`设为true，那这个函数不会被调用，当然自己在别的函数中手动调用的话除外。

除了`WithNetSerializer`这个之外，UE4还提供了许多其他的特征(type traits):
```c++
// Runtime/CoreUObject/Public/UObject/Class.h

/** type traits to cover the custom aspects of a script struct **/
struct TStructOpsTypeTraitsBase
{
	enum
	{
		WithZeroConstructor            = false, // struct can be constructed as a valid object by filling its memory footprint with zeroes.
		WithNoInitConstructor          = false, // struct has a constructor which takes an EForceInit parameter which will force the constructor to perform initialization, where the default constructor performs 'uninitialization'.
		WithNoDestructor               = false, // struct will not have its destructor called when it is destroyed.
		WithCopy                       = false, // struct can be copied via its copy assignment operator.
		WithIdenticalViaEquality       = false, // struct can be compared via its operator==.  This should be mutually exclusive with WithIdentical.
		WithIdentical                  = false, // struct can be compared via an Identical(const T* Other, uint32 PortFlags) function.  This should be mutually exclusive with WithIdenticalViaEquality.
		WithExportTextItem             = false, // struct has an ExportTextItem function used to serialize its state into a string.
		WithImportTextItem             = false, // struct has an ImportTextItem function used to deserialize a string into an object of that class.
		WithAddStructReferencedObjects = false, // struct has an AddStructReferencedObjects function which allows it to add references to the garbage collector.
		WithSerializer                 = false, // struct has a Serialize function for serializing its state to an FArchive.
		WithPostSerialize              = false, // struct has a PostSerialize function which is called after it is serialized
		WithNetSerializer              = false, // struct has a NetSerialize function for serializing its state to an FArchive used for network replication.
		WithNetDeltaSerializer         = false, // struct has a NetDeltaSerialize function for serializing differences in state from a previous NetSerialize operation.
		WithSerializeFromMismatchedTag = false, // struct has a SerializeFromMismatchedTag function for converting from other property tags.
	};
};
```

关于原文中有这样一段：
> The `FArchive` is a class which implements a common pattern for data serialization, allowing the writing of two-way functions. Basically, when it comes to serialization, you have to make sure that the way you serialize your data is exactly the same you use for deserialization. The best way to ensure this behavior is to write one single context-sensitive function that does both. The black magic of the `FArchive` lays in its overloaded `<<` operator. This operator is at the base of the creation of two-way functions. Its behavior is context-sensitive: when the `FArchive` is in write mode, it copies data from right to left, when the `FArchive` is in read mode, it copies data from left to right.

`<<`看似是单向的其实是双向的，真的神奇。

作者用这里的代码作为参考：
```c++
// Runtime/Core/Private/Math/UnrealMath.cpp

void FRotator::SerializeCompressed( FArchive& Ar )
{
	uint8 BytePitch = FRotator::CompressAxisToByte(Pitch);
	uint8 ByteYaw = FRotator::CompressAxisToByte(Yaw);
	uint8 ByteRoll = FRotator::CompressAxisToByte(Roll);

	uint8 B = (BytePitch!=0);
	Ar.SerializeBits( &B, 1 );
	if( B )
	{
		Ar << BytePitch;
	}
	else
	{
		BytePitch = 0;
	}

	B = (ByteYaw!=0);
	Ar.SerializeBits( &B, 1 );
	if( B )
	{
		Ar << ByteYaw;
	}
	else
	{
		ByteYaw = 0;
	}

	B = (ByteRoll!=0);
	Ar.SerializeBits( &B, 1 );
	if( B )
	{
		Ar << ByteRoll;
	}
	else
	{
		ByteRoll = 0;
	}

  // 这里就是context-sensitive：只有在archive处于read mode的时候数据会被还原到结构体的属性中
	if( Ar.IsLoading() )
	{
		Pitch = FRotator::DecompressAxisFromByte(BytePitch);
		Yaw	= FRotator::DecompressAxisFromByte(ByteYaw);
		Roll = FRotator::DecompressAxisFromByte(ByteRoll);
	}
}
```

这在我们MultiplayerNetworking中是一个相当有用的特性。比如说当我们想要复制玩家的control的时候，我们需要将这些数据同步到其他客户端上，我们可以像上面的例子一样，将每一个control的数据压缩到一个字节中，而且仅当这个数据不为0，由于我们的控制也许大部分时间都是处于0的状态，这样就可以节省大量的带宽。下面是作者给出的例子：
```c++
bool NetSerialize(FArchive& Ar, class UPackageMap* Map, bool& bOutSuccess)
{
    uint8 ByteAcceleration  = FMath::Quantize8UnsignedByte(Acceleration);
    uint8 ByteBrake         = FMath::Quantize8UnsignedByte(Brake);
    uint8 ByteTurn          = FMath::Quantize8SignedByte(Turn);
    uint8 BytePitch         = FMath::Quantize8SignedByte(Pitch);
    uint8 ByteRoll          = FMath::Quantize8SignedByte(Roll);

    uint8 B = (ByteAcceleration != 0);
    Ar.SerializeBits(&B, 1);
    if (B)  Ar << ByteAcceleration; else ByteAcceleration = 0;

    B = (ByteBrake != 0);
    Ar.SerializeBits(&B, 1);
    if (B) Ar << ByteBrake; else ByteBrake = 0;

    B = (ByteTurn != 0);
    Ar.SerializeBits(&B, 1);
    if (B) Ar << ByteTurn; else ByteTurn = 0;

    B = (BytePitch != 0);
    Ar.SerializeBits(&B, 1);
    if (B) Ar << BytePitch; else BytePitch = 0;

    B = (ByteRoll != 0);
    Ar.SerializeBits(&B, 1);
    if (B) Ar << ByteRoll; else ByteRoll = 0;

    if (Ar.IsLoading())
    {
        Acceleration    = Decompress8UnsignedByte(ByteAcceleration);
        Brake           = Decompress8UnsignedByte(ByteBrake);
        Turn            = Decompress8SignedByte(ByteTurn);
        Pitch           = Decompress8SignedByte(BytePitch);
        Roll            = Decompress8SignedByte(ByteRoll);
    }

    return true;
}
```

关于UE4中提供的可以快速序列化和反序列化的类型：
> Unreal Engine implements a generic data serialization for atomic properties of an Actor like ints, floats, objects* and a generic delta serialization for dynamic properties like TArrays. Delta serialization is performed by comparing a previous base state with the current state and generating a diff state and a full state to be used as a base state for the next delta serialization.

可以知道的有`ints`,`floats`,`object*`,还有`TArray`。数组对应的应该也是那些基础类型。

数组的实现有些特殊，**DeltaSerialization for Dynamic properties**，DeltaSerialization是通过之前的**base state**和现在的**current state**进行对比生成一个**diff state**和一个**full state**,这个**full state**则会作为下次的**DeltaSerialization**的**base state**。

而在**USTRUCT**中也是可以对上面的**DeltaSerialization**实现自定义的。通过定义一个`NetDeltaSerialize`函数。
```c++
/**
 * @param DeltaParms	Generic struct of input parameters for delta serialization
 *
 * @return return true if the serialization was fully mapped. If false, the property will be considered 'dirty' and will replicate again on the next update.
 *	This is needed for UActor* properties. If an actor's Actorchannel is not fully mapped, properties referencing it must stay dirty.
 *	Note that UPackageMap::SerializeObject returns false if an object is unmapped. Generally, you will want to return false from your ::NetSerialize
 *  if you make any calls to ::SerializeObject that return false.
 *
*/
bool NetDeltaSerialize(FNetDeltaSerializeInfo & DeltaParms);
```
上面提到的UE4的Serialization源码中有很多实现的教程。

**Custom net delta serialization**主要是跟**Fast TArray Replication(FTR)**结合使用的。
基本上如果我们想要有效的对**TArray**进行复制(replicated)，又或者想要在客户端检测到add和remove的事件，那么就非常推荐我们在struct中使用FTR。

下面是关于FTR的代码源码的注释说明：
> Fast TArray Replication is a custom implementation of NetDeltaSerialize that is suitable for TArrays of UStructs. It offers performance improvements for large data sets, it serializes removals from anywhere in the array optimally, and allows events to be called on clients for adds and removals. The downside is that you will need to have game code mark items in the array as dirty, and well as the order of the list is not guaranteed to be identical between client and server in all cases.

这是关于如何在自定义的struct中使用FTR的例子，来自于UE4的源码：
```c++
/** Step 1: Make your struct inherit from FFastArraySerializerItem */
USTRUCT()
struct FExampleItemEntry : public FFastArraySerializerItem
{
	GENERATED_USTRUCT_BODY()

	// Your data:
	UPROPERTY()
	int32		ExampleIntProperty;

	UPROPERTY()
	float		ExampleFloatProperty;


	/**
	 * Optional functions you can implement for client side notification of changes to items;
	 * Parameter type can match the type passed as the 2nd template parameter in associated call to FastArrayDeltaSerialize
	 *
	 * NOTE: It is not safe to modify the contents of the array serializer within these functions, nor to rely on the contents of the array
	 * being entirely up-to-date as these functions are called on items individually as they are updated, and so may be called in the middle of a mass update.
	 */
	void PreReplicatedRemove(const struct FExampleArray& InArraySerializer);
	void PostReplicatedAdd(const struct FExampleArray& InArraySerializer);
	void PostReplicatedChange(const struct FExampleArray& InArraySerializer);
};

/** Step 2: You MUST wrap your TArray in another struct that inherits from FFastArraySerializer */
USTRUCT()
struct FExampleArray: public FFastArraySerializer
{
	GENERATED_USTRUCT_BODY()

	UPROPERTY()
	TArray<FExampleItemEntry>	Items;	/** Step 3: You MUST have a TArray named Items of the struct you made in step 1. */

	/** Step 4: Copy this, replace example with your names */
	bool NetDeltaSerialize(FNetDeltaSerializeInfo & DeltaParms)
	{
	   return FFastArraySerializer::FastArrayDeltaSerialize<FExampleItemEntry, FExampleArray>( Items, DeltaParms, *this );
	}
};

/** Step 5: Copy and paste this struct trait, replacing FExampleArray with your Step 2 struct. */
template<>
struct TStructOpsTypeTraits< FExampleArray > : public TStructOpsTypeTraitsBase
{
       enum
       {
			WithNetDeltaSerializer = true,
       };
};

#endif

/** Step 6 and beyond:
 *		-Declare a UPROPERTY of your FExampleArray (step 2) type.
 *		-You MUST call MarkItemDirty on the FExampleArray when you change an item in the array. You pass in a reference to the item you dirtied.
 *			See FFastArraySerializer::MarkItemDirty.
 *		-You MUST call MarkArrayDirty on the FExampleArray if you remove something from the array.
 *		-In your classes GetLifetimeReplicatedProps, use DOREPLIFETIME(YourClass, YourArrayStructPropertyName);
 *
 *		You can override the following virtual functions in your structure (step 1) to get notifies before add/deletes/removes:
 *			-void PreReplicatedRemove(const FFastArraySerializer& Serializer)
 *			-void PostReplicatedAdd(const FFastArraySerializer& Serializer)
 *			-void PostReplicatedChange(const FFastArraySerializer& Serializer)
 *
 *		Thats it!
 */
```

关于上面的第六步及以后，作者给了示例：
```c++
// adding a FExampleArray property to an Actor
UCLASS()
class MyActor : public AActor
{
	GENERATED_UCLASS_BODY()

public:
	UPROPERTY(Replicated)
	FExampleArray DeltaTest;
}

// Adding DOREPLIFETIME to the GetLifetimeReplicatedProps method
void MyActor::GetLifetimeReplicatedProps(TArray< FLifetimeProperty > & OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME(MyActor, DeltaTest);
}

// Adding an element to the array
void MyActor::AddItem() {
	FExampleItemEntry a;
	a.ExampleFloatProperty = 3.14;
	a.ExampleIntProperty = 1234;		
	DeltaTest.MarkItemDirty(DeltaTest.Items.Add_GetRef(a));
}

// Modifying an element
void MyActor::ChangeItem(int32 ItemID) {
	if (DeltaTest.Items.Num() > ItemID) {
		DeltaTest.Items[ItemID].ExampleFloatProperty = 6.28;
		DeltaTest.Items[ItemID].ExampleIntProperty = 5678;
		DeltaTest.MarkItemDirty(DeltaTest.Items[ItemID]);
	}
}

// Removing an element
void MyActor::RemoveLastItem() {
	if (DeltaTest.Items.Num() > 0) {
		DeltaTest.Items.RemoveAt(DeltaTest.Items.Num()-1, 1);
		DeltaTest.MarkArrayDirty();
	}
}
```

作者将一个Item标志为Dirty位，当新添加一个Item或者修改了一个Item的时候。
作者将整个Array标志为Dirty位，当移除了某个Item的时候。

这里我也不清楚这篇文章的作者是有意为之还是说移除某个对象的时候，其整体Array都需要标志为Dirty位是必须操作。不过我感觉这个是必须操作。
