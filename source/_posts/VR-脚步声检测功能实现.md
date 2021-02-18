---
title: VR-脚步声检测功能实现
permalink: vr-footstep-detection
date: 2020-01-18 03:27:04
categories:
- VR
tags:
- C#
- Unity
- 非公开文章
---
也算是业务相关吧，业务要求要只用HMD(Head Mounted Display, 这里就是VR头戴设备)检测脚步声，这里我根据公司前辈的工作内容进行了一些改良，主要是利用VR设备移动的时候的前后上下的加速度和速度来判定脚步。准确率正常情况下有90%以上吧。

<!--more-->

对源码进行改良，我主要的灵感来源是这篇文章
- [CSS3人行走动作图解和动画实现](https://techbrood.com/zh/news/css3/css3%E5%8A%A8%E7%94%BB%EF%BC%9A%E4%BA%BA%E8%A1%8C%E8%B5%B0%E5%8A%A8%E4%BD%9C%E5%9B%BE%E8%A7%A3_2.html)

根据这种分解的例子找到判定的条件。

~~代码实现的话有时间贴上，并说明。~~

# 检测实现
算是我在公司制作的骚东西吧，就成品的结果来看还是可以的，但是总觉得有那么一丝丝不足。
- 首先是当玩家快速移动的时候,检测精度就大幅度下降，有多大幅度呢？就是接近0，但是这个快速移动吧，就是跑的飞快的那种，倒也还好。
- 在里面还加入了左右检测的功能，在头朝前移动左右的判定倒也准确，就是扭头状态下移动的时候多少会有些不准确。
- 再就是检测的延迟感，虽说体感上不明显，但是意识到的我自己总觉得检测的时机有些"粘稠感"。

下面先贴上代码实现。
```csharp
using UnityEngine;

namespace MyProject
{
    public class HMDMoveFootsteps : MonoBehaviour
    {
        static ILogger logger = new Logger();

        [SerializeField]
        private int precisionIndex = 0;

        public float YMoveThreshold = -0.00075f;
        public float ZXMoveThreshold = 0.00425f;
        int SENum = 5;
        int lastSEIndex = 0;
        const int RecordNum = 5;
        const int AveRecordNum = 5;

        bool onceEnter = true;
        int onceEnterCounter = 0;
        bool prevFlag = false;
        bool playOrder = false;
        // Detect footstep one time, and reset before next time
        private bool oneStep = false;
        private bool stepCD = false;
        private int stepEnter = 0;

        // For FootCheckBeta
        [SerializeField]
        private bool Mute = false;
        const int sampleDataCount = 5;

        private float accelerateThreshold
        {
            get { return highAccelerateThreshold / (precisionIndex + 1); }

        }
        private float speedYThreshold
        {
            get { return highSpeedYThreshold / (precisionIndex + 1); }
        }
        private float speedXZThreshold
        {
            get { return highSpeedXZThreshold / (precisionIndex + 1); }
        }
        const float highAccelerateThreshold = 0.2f;
        const float highSpeedYThreshold = 0.02f;
        const float highSpeedXZThreshold = 0.01f;
        
        private Unity.Collections.NativeArray<float> preFramesDataY;
        private Unity.Collections.NativeArray<float> preFramesDataZ;
        private Unity.Collections.NativeArray<float> preFramesDataX;
        private Unity.Collections.NativeArray<float> preFramesDeltaTime;
        private Unity.Collections.NativeArray<float> preFramesYSpeed;
        private Unity.Collections.NativeArray<float> preFramesZSpeed;
        private Unity.Collections.NativeArray<float> preFramesXSpeed;

        private EventFootstepTrigger _LeftEventFootstepTrigger;
        private EventFootstepTrigger LeftEventFootstepTrigger
        {
            get
            {
                return _LeftEventFootstepTrigger ?? (_LeftEventFootstepTrigger = AvatarManager.OwnerAvatar.LeftEventFootstepTrigger);
            }
        }
        private EventFootstepTrigger _RightEventFootstepTrigger;
        private EventFootstepTrigger RightEventFootstepTrigger
        {
            get
            {
                return _RightEventFootstepTrigger ?? (_RightEventFootstepTrigger = AvatarManager.OwnerAvatar.RightEventFootstepTrigger);
            }
        }

        // Use this for initialization
        void Start()
        {
            preFramesDataY = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesDataZ = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesDataX = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesDeltaTime = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesYSpeed = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesZSpeed = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            preFramesXSpeed = new Unity.Collections.NativeArray<float>(sampleDataCount, Unity.Collections.Allocator.Persistent);
            // Initialization?
        }

        private void OnDestroy()
        {
            preFramesDataY.Dispose();
            preFramesDataZ.Dispose();
            preFramesDataX.Dispose();
            preFramesDeltaTime.Dispose();
            preFramesYSpeed.Dispose();
            preFramesZSpeed.Dispose();
            preFramesXSpeed.Dispose();
        }

        // Update is called once per frame
        void Update()
        {
            if (Mute) return;
            FootCheckVerBeta();
        }

        void FootCheckVerBeta()
        {
            for (var i = sampleDataCount - 1; i > 0; i--)
            {
                preFramesDataY[i] = preFramesDataY[i - 1];
                preFramesDataZ[i] = preFramesDataZ[i - 1];
                preFramesDataX[i] = preFramesDataX[i - 1];
                preFramesDeltaTime[i] = preFramesDeltaTime[i - 1];
                preFramesYSpeed[i] = preFramesYSpeed[i - 1];
                preFramesZSpeed[i] = preFramesZSpeed[i - 1];
                preFramesXSpeed[i] = preFramesXSpeed[i - 1];
            }

            var pos = StageController.PlayerCamera.transform.position;
            preFramesDataY[0] = pos.y;
            preFramesDataZ[0] = pos.z;
            preFramesDataX[0] = pos.x;
            preFramesDeltaTime[0] = Time.deltaTime;

            float sumY = 0f;
            float sumZ = 0f;
            float sumX = 0f;
            float sumTime = 0f;
            for (var j = 0; j < sampleDataCount -2; j++)
            {
                sumY += (preFramesDataY[j] - preFramesDataY[j + 1]);
                sumZ += (preFramesDataZ[j] - preFramesDataZ[j + 1]);
                sumX += (preFramesDataX[j] - preFramesDataX[j + 1]);
            }
            for(var m = 0; m < sampleDataCount-1; m++)
            {
                sumTime += preFramesDeltaTime[m];
            }

            // Y axis movement speed
            preFramesYSpeed[0] = sumY / sumTime;

            preFramesZSpeed[0] = sumZ / sumTime;
            preFramesXSpeed[0] = sumX / sumTime;

            float sumSpeed = 0f;
            for(var k = 0; k < sampleDataCount-2; k++)
            {
                sumSpeed += (preFramesYSpeed[k] - preFramesYSpeed[k + 1]);
            }

            float acceleration = sumSpeed / sumTime;

            float preFramesSum = 0f;
            for(var n = 0; n < sampleDataCount -1; n++)
            {
                preFramesSum += preFramesYSpeed[n];
            }
            // float averageSpeed = preFramesSum / sampleDataCount;

            float speedXZ = preFramesZSpeed[0] * preFramesZSpeed[0] + preFramesXSpeed[0] * preFramesXSpeed[0];

            if(!stepCD && acceleration > accelerateThreshold && System.Math.Abs(preFramesYSpeed[0]) > speedYThreshold && speedXZ >= speedXZThreshold)
            {
                oneStep = true;
                stepCD = true;
            }

            if (oneStep)
            {
                // logger.Log($"DebugUI: Acceleration is {acceleration}");
                // logger.Log($"DebugUI: YSpeed is {preFramesYSpeed[0]}");
                oneStep = false;
                stepEnter = 30;

                var moveDir = StageController.PlayerCamera.transform.forward;
                var shakeDir = new Vector3(preFramesXSpeed[0], 0f, preFramesZSpeed[0]);
                var dirResult = Vector3.Cross(moveDir, shakeDir);
                // logger.Log($"DebugUI: direction : {dirResult.y}");
                if(dirResult.y >= 0)
                {
                    logger.Log("DebugUI: FootStep----Right!");
                }
                else
                {
                    logger.Log("DebugUI: FootStep----Left!");
                }
            }

            if (stepCD)
            {
                stepEnter--;
                if(stepEnter <= 0)
                {
                    stepCD = false;
                }
            }

            /*
            if(System.Math.Abs(preFramesSpeed[0]) > speedThreshold)
                logger.Log($"DebugUI: YSpeed is {preFramesSpeed[0]}");
            
            if(System.Math.Abs(averageSpeed)> 0.02f)
            logger.Log($"DebugUI: {averageSpeed}");
            */

            // Debug
            // float speedXZ = preFramesZSpeed[0] * preFramesZSpeed[0] + preFramesXSpeed[0] * preFramesXSpeed[0];
            // logger.Log($"DebugUI: speed_XZ = {speedXZ}");
            // logger.Log($"DebugUI: player forward ( {StageController.PlayerCamera.transform.forward})");
        }
    }
}

```

上面就是检测的全部实现了。

实现中对检测的精度设置了三个档位，于是我想让这三个档位能实现更好的切换，就对Inspector进行了扩展。

# 检测脚本的Editor扩展


```csharp
using UnityEditor;
using UnityEngine;

[CustomEditor(typeof(HMDMoveFootsteps))]
public class HMDFootStepsInspector : Editor
{
    protected SerializedProperty precisionIndex;
    protected SerializedProperty mute;

    private void OnEnable()
    {
        mute = serializedObject.FindProperty("Mute");
        precisionIndex = serializedObject.FindProperty("precisionIndex");
    }

    public override void OnInspectorGUI()
    {
        serializedObject.Update();


        EditorGUILayout.PropertyField(mute, new GUIContent("Mute"));
        var index = GUILayout.Toolbar(precisionIndex.intValue, new string[] { "High Precision", "Middle Precision", "Low Precision" });
        precisionIndex.intValue = index;

        serializedObject.ApplyModifiedProperties();
    }
}

```
达成的效果就像下面这样：

![Inspector视图，2019.3.5f1版本](HMDFootStep_Inspector.png)

编辑器扩展的内容本来不想放在这个部分，因为有专门放在扩展的文章，但是分开又觉得不好。另外就是扩展的部分的内容并没有配截图，那个时候还没有为这个博客添加图片。
想想那一部分应该都配上展示图片为好。

# 总结
对于这个脚步检测的结果我还是比较满意的，但是在实际的体验中总是有些不尽人意，所以肯定还有改进的空间。另外这篇文章内容暂时还是不公开为好，毕竟我直接从项目里面Copy的代码。感觉该删的都删掉了但是还是感觉不舒服。

