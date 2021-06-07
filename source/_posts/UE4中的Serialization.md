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
