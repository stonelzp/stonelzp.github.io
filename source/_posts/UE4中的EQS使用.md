---
title: UE4中的EQS使用
date: 2021-09-16 21:59:42
permalink: ue4-how-to-use-eqs
categories:
- UnrealEngine4
tags:
- UE4
---

EQS(Environment Query System)的简称，主要应用场景就是制作游戏角色的AI了。
<!--more-->

使用EQS的启动条件还需要AIController和BehaviourTree的知识，这部分就在其它文章中展开了。

# 在UE4中使用EQS

## EQS是什么
- [Unreal Engine 4 でEQSを使えるようになる](https://qiita.com/manuo/items/cc332b71aedb70ad6bfc)

上面的文章介绍了EQS中的基础概念，和两个组成部分**Generator**和**Test**。

## 自定义使用
标准的功能肯定是不能满足项目需求的，如何自定义EQS的功能也是必修课。

### 自定义Test
新建一个`EnvQueryTest`类，从UE4Editor创建就可以了。

参考文章：
- [【UE4】AIを作るのに必要不可欠なEQSのテストをC++で作成できるようになろう！](https://qiita.com/4_mio_11/items/419a614c81bc05381f2c)
- [EQS-テスト自作](http://com04.sakura.ne.jp/unreal/wiki/index.php?EQS-%A5%C6%A5%B9%A5%C8%BC%AB%BA%EE)

基本上需要定义的函数：
```
#pragma once

#include "CoreMinimal.h"
#include "EnvironmentQuery/EnvQueryTest.h"
#include "MyEnvQueryTest.generated.h"

UCLASS()
class NPC_CREATE_API UMyEnvQueryTest : public UEnvQueryTest
{
    GENERATED_UCLASS_BODY()

    virtual void RunTest(FEnvQueryInstance& QueryInstance) const override;

    virtual FText GetDescriptionTitle() const override;
    virtual FText GetDescriptionDetails() const override;
};
```

实现：
```
#include "EnvironmentQuery/Items/EnvQueryItemType_VectorBase.h"

UMyEnvQueryTest::UMyEnvQueryTest(const FObjectInitializer& ObjectInitializer) : Super(ObjectInitializer)
{
	//
	Cost = EEnvTestCost::Low;

	//
	ValidItemType = UEnvQueryItemType_VectorBase::StaticClass();
}

void UMyEnvQueryTest::RunTest(FEnvQueryInstance& QueryInstance) const
{
	// オーナーが居ない
	UObject* QueryOwner = QueryInstance.Owner.Get();
	if (QueryOwner == nullptr)
	{
		return;
	}

	FloatValueMin.BindData(QueryOwner, QueryInstance.QueryID);
	float MinThresholdValue = FloatValueMin.GetValue();

	FloatValueMax.BindData(QueryOwner, QueryInstance.QueryID);
	float MaxThresholdValue = FloatValueMax.GetValue();

	// アイテムの数だけブン回す
	float score = 0.0f;  // いい感じにする
	for (FEnvQueryInstance::ItemIterator It(this, QueryInstance); It; ++It)
	{
		score += 0.05f;  // いい感じにする
		It.SetScore(TestPurpose, FilterType, score, MinThresholdValue, MaxThresholdValue);
	}
}

FText UMyEnvQueryTest::GetDescriptionTitle() const
{
	return Super::GetDescriptionTitle();
}

FText UMyEnvQueryTest::GetDescriptionDetails() const
{
	return DescribeFloatTestParams();
}
```

关于构造函数里的初始化操作

#### Cost
这个设定是用来设置Test的优先执行顺序的，有`Low`,`Medium`,`High`,优先级越低就会被优先执行，默认情况下时Low。这个某种程度上可以提升一点执行的效率，将执行效率低的处理设置为高优先级尽可能的减少处理次数。

#### ValidItemType
这个暂时还不是很清楚。


### 自定义Generator


# AI相关的Debug工具

- [AI Debugging - Describes the different ways in which you can debug your AI with the AI Debugging Tools.](https://docs.unrealengine.com/4.27/en-US/InteractiveExperiences/ArtificialIntelligence/AIDebugging/)
