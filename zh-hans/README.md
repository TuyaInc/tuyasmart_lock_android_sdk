# 涂鸦智能门锁 Android SDK 文档

**当前 SDK 处于开发者预览阶段，仍在持续改善中，如有任何使用上的问题或建议，请联系我们。**

涂鸦智能门锁 Android SDK 提供了与智能门锁设备的功能封装，加速和简化门锁应用功能开发过程，主要包括了以下功能：

* 门锁用户体系 （包括门锁用户管理、关联密码等功能）
* 门锁密码解锁（包括动态密码获取、临时密码管理相关的功能）
* 门锁使用记录（包括门锁开锁记录、门铃记录、报警记录等功能）

本 SDK 支持[蓝牙门锁](resource/BLELockDoc.md)和[ Wi-Fi 门锁](resource/WiFiLockDoc.md)两类产品。

## 准备工作

该 SDK 依赖于涂鸦全屋智能 SDK，基于此基础上进行拓展开发，准备工作请查阅涂鸦全屋智能  SDK 接入文档](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/zh-hans/resource/Preparation.html)

## 快速集成

### 添加 maven 地址

在根目录 `build.gradle` 中添加涂鸦 maven 地址：

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

### 添加依赖

在 module 层的 `build.gradle `的 `dependencies` 中添加门锁 SDK 依赖

```groovy
dependencies {
    ...
   implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.4-SNAPSHOT'
}
```

### 权限说明

为了扫描和连接蓝牙设备，你需要添加下列权限到AndroidManifest.xml中，非蓝牙设备可忽略。

```xml
<!-- Required. Allows applications to connect to paired bluetooth devices.  -->
<uses-permission android:name="android.permission.BLUETOOTH" />
<!-- Required. Allows applications to discover and pair bluetooth devices.  -->
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
<!-- Required.  Allows an app to scan bluetooth device.  -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!-- Required.  Allows an app to scan bluetooth device.  -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!--  Allows an app to use bluetooth low energy feature  -->
<uses-feature
android:name="android.hardware.bluetooth_le"
android:required="false" />
```

动态权限获取可以参考文档，[请求运行时权限](https://developer.android.com/training/permissions/requesting?hl=zh-cn#make-the-request)。

