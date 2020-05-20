# 涂鸦智能门锁 Android SDK 文档

[English Document](README_en.md)

**当前 SDK 处于开发者预览阶段，仍在持续改善中，如有任何使用上的问题或建议，请联系我们。**

涂鸦智能门锁 Android SDK 提供了与智能门锁设备的功能封装，加速和简化门锁应用功能开发过程，主要包括了以下功能：

* 门锁用户体系 （包括门锁用户管理、关联密码等功能）
* 门锁密码解锁（包括动态密码获取、临时密码管理相关的功能）
* 门锁使用记录（包括门锁开锁记录、门铃记录、报警记录等功能）

本SDK支持蓝牙门锁和WIFI门锁两类产品。

## 集成前提

涂鸦门锁 SDK 是基于 [涂鸦全屋智能SDK](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/zh-hans/) 开发

集成涂鸦门锁 SDK 之前，需要做以下工作：

* 集成 TuyaHomeSdk（包括申请 tuya App ID 和 App Secret、安全图片配置相关环境），请查阅[涂鸦全屋智能  SDK 接入文档](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/zh-hans/resource/Preparation.html)

* 对门锁设备配网完成
  

## 快速集成

1. 在根目录 `build.gradle` 中添加涂鸦 maven 地址：

    ```groovy
    allprojects {
      repositories {
      // tuya maven url
      maven {
        url "https://maven-other.tuya.com/repository/maven-releases/"
      }
      google()
        jcenter()
      }
    }
    ```

2. 在 module 层的 `build.gradle `的 `dependencies` 中添加门锁 SDK 依赖

    ```groovy
    dependencies {
        ...
       implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.3-SNAPSHOT'
    }
    ```

## 专有名词解释

|名词|解释|
|---|---|
|dpCode|设备功能点的标识符。设备中的每个功能点都有名称和编号，本文档针对Wi-Fi门锁设备，可参考[Wi-Fi 门锁功能点列表](#Wi-Fi-门锁功能点列表)|
|劫持|门锁劫持是指将录入特定的密码（指纹、密码等），设置为劫持密码。<br>当用户输入此密码开门时，门锁认为用户非自愿开门，会将告警信息发送至家人手机或物业管理系统。|
|门锁成员|门锁成员分为家庭成员与非家庭成员。<br>家庭成员即添加到用户家庭中的成员，门锁内可对管理家庭成员，设置开锁方式。<br>非家庭成员为在门锁内创建的成员，可通过门锁相关接口进行管理。|

## WIFI 门锁文档

[WIFI 门锁文档](zh-hans/resource/WiFiLockDoc.md)

## 蓝牙门锁文档

[蓝牙门锁文档](zh-hans/resource/BLELockDoc.md)