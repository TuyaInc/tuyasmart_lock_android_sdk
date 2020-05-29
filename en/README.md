# Tuya Smart Lock Android SDK

**The SDK is currently in the developer preview stage and is still being continuously improved. If you have any questions or suggestions, please contact us. **

Tuya smart lock Android The SDK provides function packaging with smart door lock devices to speed up and simplify the development process of door lock application functions, including the following functions:

* Door lock user system (including lock user management, associated password, etc.)
* Door lock password unlocking (including dynamic password, temporary password management, etc.)
* Door lock usage records (including door lock unlock records, doorbell records, alarm records, etc.)

## Preparation

Tuya lock SDK is based on [Tuya Smart Home SDK](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/en/).

Before integrating Tuya Lock SDK, you need to do the following:

* Integrate TuyaHomeSdk (including application for tuya App ID and App Secret, security image configuration related environment)，please read [Tuya Smart Home SDK Document](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/en/resource/Preparation.html)

* Activation of the door lock device

## Fast Integration

### Add Maven Url

Add tuya maven address in root `build.gradle` 

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

### Add Dependency

Add the door lock SDK dependency in the dependencies of `build.gradle` in the module layer

```groovy
dependencies {
    ...
   implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.4-SNAPSHOT'
}
```

### Permissions

In order to scan and connect Bluetooth devices, you need to add the following permissions to AndroidManifest.xml.

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

You can refer to the documentation for dynamic permission acquisition，[Request App Permissions](https://developer.android.com/training/permissions/requesting?hl=en#make-the-request)。

