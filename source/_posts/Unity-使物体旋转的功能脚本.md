---
title: Unity-使物体旋转的功能脚本
permalink: unity-make-object-rotating
date: 2020-01-06 14:51:52
categories:
- Unity
tags:
- 笔记
---

记一个使物体旋转的功能脚本，感觉这个功能时不时的会遇到，每次写的时候都挺绕的，不如记下来。

<!--more-->

## 实现
**功能实现：**
```C#
using UnityEngine;
using System;

namespace Clown.Utility
{
    [Serializable, AddComponentMenu("Clown/Utility/Make Object Rotating")]
    public class MakeObjectRotating : MonoBehaviour
    {
        [SerializeField]
        private bool clockWise = false;
        [SerializeField]
        private bool rotateAxisY = true;
        [SerializeField]
        private bool rotateAxisX = false;
        [SerializeField]
        private bool rotateAxisZ = false;
        [SerializeField]
        private float rotateSpeed = 0.0f;
        [SerializeField]
        private bool hasEnterTime = false;
        [SerializeField]
        private float enterTime = 4.0f;
        [SerializeField]
        private bool hasExitTime = false;
        [SerializeField]
        private float exitTime = 2.0f;

        bool isRotating = false;
        float speed = 0.0f;

        bool rotatingSwitch = false;

        public void OnEnable()
        {
            rotatingSwitch = true;
        }

        private void Update()
        {
            var angles = transform.rotation.eulerAngles;
            var anglesY = angles.y;
            var anglesX = angles.x;
            var anglesZ = angles.z;

            if (isRotating)
            {
                if (!rotatingSwitch)
                {
                    if (hasExitTime)
                    {
                        speed -= (rotateSpeed / exitTime * Time.deltaTime);
                        if (speed < 0.0f)
                        {
                            speed = 0.0f;
                            isRotating = false;

                            return;
                        }
                        anglesY -= rotateAxisY ? (speed * Time.deltaTime) : 0f;
                        anglesX -= rotateAxisX ? (speed * Time.deltaTime) : 0f;
                        anglesZ -= rotateAxisZ ? (speed * Time.deltaTime) : 0f;
                    }
                    else
                    {
                        isRotating = false;
                        speed = 0.0f;

                        return;
                    }
                }
                else
                {
                    anglesY -= rotateAxisY ? (rotateSpeed * Time.deltaTime) : 0f;
                    anglesX -= rotateAxisX ? (rotateSpeed * Time.deltaTime) : 0f;
                    anglesZ -= rotateAxisZ ? (rotateSpeed * Time.deltaTime) : 0f;
                }

            }
            else
            {
                if(!rotatingSwitch)
                {
                    enabled = false;

                    return;
                }

                if (hasEnterTime)
                {
                    speed += (rotateSpeed / enterTime * Time.deltaTime);
                    if (speed > rotateSpeed)
                    {
                        speed = rotateSpeed;
                        isRotating = true;
                    }

                    anglesY -= rotateAxisY ? (speed * Time.deltaTime) : 0f;
                    anglesX -= rotateAxisX ? (speed * Time.deltaTime) : 0f;
                    anglesZ -= rotateAxisZ ? (speed * Time.deltaTime) : 0f;
                }
                else
                {
                    isRotating = true;
                    anglesY -= rotateAxisY ? (rotateSpeed * Time.deltaTime) : 0f;
                    anglesX -= rotateAxisX ? (rotateSpeed * Time.deltaTime) : 0f;
                    anglesZ -= rotateAxisZ ? (rotateSpeed * Time.deltaTime) : 0f;
                }
            }

            angles.y = anglesY;
            angles.x = anglesX;
            angles.z = anglesZ;
            transform.rotation = Quaternion.Euler(angles);

        }

        public void SetValueToRotate(bool val)
        {
            rotatingSwitch = val;
        }
    }
}

```
使用的话，就在合适的时间调用`SetValueToRotate`或者`OnEnable`函数就好了，当然使用条件不一样注意一下，我就不写了。

这里我想着应该可以再优化一下代码结构的，但是想想值不值得还是两说，暂时也没有什么非常棒的想法。

这里还有就是我本来想把全部都写在项目文件里的，之前我只是实现了Y轴的旋转，因为项目面基本上不会出现其他轴的旋转了我就没有把全部实现都提交，也许会减轻一些处理吧，心理上的。

这里再贴一下**Editor扩展的实现：**
```C#
using UnityEditor;
using Project.Utility;

[CustomEditor(typeof(MakeObjectRotating))]
public class MakeObjectRotationInspector : Editor
{
    protected SerializedProperty clockWise;
    protected SerializedProperty rotateAxisY;
    protected SerializedProperty rotateAxisX;
    protected SerializedProperty rotateAxisZ;
    protected SerializedProperty rotateSpeed;
    protected SerializedProperty hasEnterTime;
    protected SerializedProperty enterTime;
    protected SerializedProperty hasExitTime;
    protected SerializedProperty exitTime;

    private void OnEnable()
    {
        clockWise = serializedObject.FindProperty("clockWise");
        rotateAxisY = serializedObject.FindProperty("rotateAxisY");
        rotateAxisX = serializedObject.FindProperty("rotateAxisX");
        rotateAxisZ = serializedObject.FindProperty("rotateAxisZ");
        rotateSpeed = serializedObject.FindProperty("rotateSpeed");
        hasEnterTime = serializedObject.FindProperty("hasEnterTime");
        enterTime = serializedObject.FindProperty("enterTime");
        hasExitTime = serializedObject.FindProperty("hasExitTime");
        exitTime = serializedObject.FindProperty("exitTime");
    }

    public override void OnInspectorGUI()
    {
        // base.OnInspectorGUI();

        serializedObject.Update();

        EditorGUILayout.PropertyField(clockWise);
        EditorGUILayout.PropertyField(rotateAxisY);
        EditorGUILayout.PropertyField(rotateAxisX);
        EditorGUILayout.PropertyField(rotateAxisZ);
        EditorGUILayout.PropertyField(rotateSpeed);

        EditorGUILayout.PropertyField(hasEnterTime);
        if (hasEnterTime.boolValue)
        {
            EditorGUI.indentLevel++;
            EditorGUILayout.PropertyField(enterTime);
            EditorGUI.indentLevel--;
        }
        EditorGUILayout.PropertyField(hasExitTime);
        if (hasExitTime.boolValue)
        {
            EditorGUI.indentLevel++;
            EditorGUILayout.PropertyField(exitTime);
            EditorGUI.indentLevel--;
        }

        serializedObject.ApplyModifiedProperties();
        
    }
}

```
