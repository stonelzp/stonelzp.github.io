---
title: UE4-反射机制
permalink: ue4-reflection
date: 2020-11-18 12:07:20
category:
- UnrealEngine4
tags:
- UE4
- UE4.25
---
反射在其它的比如说C#，JAVA语言中比较常见。反射概况的来说是描述类在运行时的状态，反射中包含的信息有类名，类数据成员，数据成员类型，还有每个成员位于对象内存映像的偏移(offset)，类所有成员函数的信息。

<!--more-->

C++本身是不支持反射的，4在C++的基础上搭建了自己的一套反射机制。


# UE4的反射机制原理


- [Unreal Property System (Reflection)](https://www.unrealengine.com/en-US/blog/unreal-property-system-reflection?sessionInvalidated=true&lang=en-US)

Property system的层级
```c++
UField
        UStruct
                UClass (C++ class)
                UScriptStruct (C++ struct)
                UFunction (C++ function)

        UEnum (C++ enumeration)

        UProperty (C++ member variable or function parameter)

                (Many subclasses for different types)
```

上面的文章虽然是很久之前文章，但是还是很厉害。



## 利用反射对属性进行读写
最近在搞使用ImGui对ini文件进行读写的功能，同时又希望像UE4的Editor那样，对于添加的UCLASS的类中的UPROPERTY属性自动的显示各种类型的UI。于是就研究了一下反射，虽然原理什么的我还是不是很清楚，希望能多读读上面的文章而不是只是单纯的记录下来，终归动还是能动的。

直接贴上代码：
```c++
    // Debug settings UI
    bool show_about_app = true;
    ImGui::Begin("DebugSettings", &show_about_app, ImGuiWindowFlags_AlwaysAutoResize);

    // Creaete DebugUI for UObject
    bool AnyPropertiesEdited = false;
    UClass* obj = DebugSettings->GetClass();
    for(TFieldIterator<FProperty> PropertyIterator(obj); PropertyIterator; ++PropertyIterator)
    {
            FProperty* Property = *PropertyIterator;
            if(FBoolProperty* BoolProperty = CastField<FBoolProperty>(Property))
            {
                    // UE_LOG(LogTemp, Log, TEXT("cast bool property successed."));
                    static bool sampleBool = BoolProperty->GetPropertyValue_InContainer(DebugSettings);
                    ImGui::Checkbox(TCHAR_TO_ANSI(*BoolProperty->GetName()), &sampleBool);
                    if(ImGui::IsItemEdited())
                    {
                            // Apply changes
                            BoolProperty->SetPropertyValue(BoolProperty->ContainerPtrToValuePtr<bool>(DebugSettings), sampleBool);
                            AnyPropertiesEdited = true;
                    }
            }
            else if(FIntProperty* IntProperty = CastField<FIntProperty>(Property))
            {
                    // UE_LOG(LogTemp, Log, TEXT("cast int property successed."));
                    static int sampleInt = IntProperty->GetPropertyValue_InContainer(DebugSettings);
                    ImGui::InputInt(TCHAR_TO_ANSI(*IntProperty->GetName()), &sampleInt);
                    if(ImGui::IsItemEdited())
                    {
                            // Apply changes
                            IntProperty->SetPropertyValue(IntProperty->ContainerPtrToValuePtr<int>(DebugSettings), sampleInt);
                            AnyPropertiesEdited = true;
                    }
            }
    }
    if(AnyPropertiesEdited)
    {
            DebugSettings->SaveConfig();
    }

    ImGui::End();
```
没有对应所有的类型，但是知道了方法就行了。

这其中有一篇文章很重要：
- [UProperty -> FPropertyへの変化(UE4 4.25)](https://www.ayumax.net/entry/2020/03/22/144226)

UE4.25版本更改了`UProperty->FProperty`，跟之前写法会有些出入

还参考了其他人的提问的文章
- [How to list UProperties of an UObject at runtime? Read/write UProperty?](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/108323-how-to-list-uproperties-of-an-uobject-at-runtime-read-write-uproperty)

##  利用反射对函数进行操作
有一天突发奇想，UE4的反射机制可不可以遍历`UObject`中的`UFUNCTION`？然后根据`UUFUNCTION`的的类型，比如说根据Meta修饰符来切换ImmGui的UI类型，Button啥的。

遍历`UFUNCTION`貌似是可行的:
- [How I Get All Functions of one Object??? - Unreal Engine Forums](https://forums.unrealengine.com/development-discussion/c-gameplay-programming/1461760-how-i-get-all-functions-of-one-object)

```c++
void UMyClass::ListAllObjectUFunctions(const UObject* Object) {

    for ( TFieldIterator<UFunction> FIT ( Object->GetClass(), EFieldIteratorFlags::IncludeSuper ); FIT; ++FIT) {

        UFunction* Function = *FIT;
        UE_LOG ( LogTemp, Log, TEXT( "Function Found:  %s();" ), *Function->GetName() );

    }
}
```

至于自定义的Meta修饰符，我还没开始调查。啧。

## 参考资料
- [UE4反射原理的探究](https://blog.csdn.net/mohuak/article/details/81913532?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.add_param_isCf&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromBaidu-1.add_param_isCf)
- [虚幻4属性系统（反射）翻译](https://www.cnblogs.com/ghl_carmack/p/5698438.html)
- [UE4反射机制](https://zhuanlan.zhihu.com/p/60622181)
- [《InsideUE4》UObject（四）类型系统代码生成](https://zhuanlan.zhihu.com/p/25098685)

感觉上面的文章都很重要。尤其是最后一篇。
 
