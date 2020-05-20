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
      maven {
        url "https://maven-other.tuya.com/repository/maven-snapshots/"
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
|dpCode|设备功能点的标识符。设备中的每个功能点都有名称和编号，可参考[门锁功能点列表](#门锁功能点列表)|
|劫持|门锁劫持是指将录入特定的密码（指纹、密码等），设置为劫持密码。<br>当用户输入此密码开门时，门锁认为用户非自愿开门，将发送告警通知。|
|门锁成员|门锁成员分为家庭成员与非家庭成员。<br>家庭成员即添加到用户家庭中的成员，门锁内可对管理家庭成员，设置开锁方式。<br>非家庭成员为在门锁内创建的成员，可通过门锁相关接口进行管理。|

## WIFI 门锁文档

[WIFI 门锁文档](zh-hans/resource/WiFiLockDoc.md)

## 蓝牙门锁文档

[蓝牙门锁文档](zh-hans/resource/BLELockDoc.md)

### 门锁功能点列表

| dp name              | dp code                        |
| -------------------- | ------------------------------ |
|蓝牙解锁|bluetooth_unlock|
| 指纹解锁             | unlock\_fingerprint            |
| 普通密码解锁         | unlock\_password               |
| 临时密码解锁         | unlock\_temporary              |
| 动态密码解锁         | unlock\_dynamic                |
| 卡片解锁             | unlock\_card                   |
| 人脸识别解锁         | unlock\_face                   |
| 钥匙解锁             | unlock\_key                    |
| 告警                 | alarm\_lock                    |
| 远程开门请求倒计时   | unlock\_request                |
| 远程开门请求回复     | reply\_unlock\_request         |
| 电池电量状态         | battery\_state                 |
| 剩余电量             | residual\_electricity          |
| 反锁状态             | reverse\_lock                  |
| 童锁状态             | child\_lock                    |
| App远程解锁wifi门锁  | unlock\_app                    |
| 劫持告警             | hijack                         |
| 从门内侧打开门锁     | open\_inside                   |
| 开合状态             | closed\_opened                 |
| 门铃呼叫             | doorbell                       |
| 短信通知             | message                        |
| 上提反锁             | anti\_lock\_outside            |
| 虹膜解锁             | unlock\_eye                    |
| 掌纹解锁             | unlock\_hand                   |
| 指静脉解锁           | unlock\_finger\_vein           |
| 同步所有指纹编号     | update\_all\_finger            |
| 同步所有密码编号     | update\_all\_password          |
| 同步所有卡编号       | update\_all\_card              |
| 同步所有人脸编号     | update\_all\_face              |
| 同步所有虹膜编号     | update\_all\_eye               |
| 同步所有掌纹编号     | update\_all\_hand              |
| 同步所有指静脉编号   | update\_all\_fin\_vein         |
| 离线密码解锁上报     | unlock\_offline\_pd            |
| 离线密码清空上报     | unlock\_offline\_clear         |
| 单条离线密码清空上报 | unlock\_offline\_clear\_single |
