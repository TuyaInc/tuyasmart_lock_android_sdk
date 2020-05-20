[Chinese Document](README.md)

# Tuya Smart Lock Android SDK

** The SDK is currently in the developer preview stage and is still being continuously improved. If you have any questions or suggestions, please contact us. **

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

1. Add tuya maven address in root `build.gradle` 

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
2. Add the door lock SDK dependency in the dependencies of `build.gradle` in the module layer

    ```groovy
    dependencies {
        ...
       implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.3-SNAPSHOT'
    }
    ```

## Term Explanation

|Term|Explanation|
|---|---|
|dpCode|The identifier of the device function point. Each function point in the device has a name and number. This document is for Wi-Fi door lock devices, please refer to [Wi-Fi Door Lock Function Points](#wi-fi-door-lock-function-points)|
|hijack|Door lock hijacking refers to setting a specific password (fingerprint, password, etc.) as the hijacking password. <br/> When the user enters this password to open the door, the door lock considers the user to open the door involuntarily, and sends the alarm information to the family's mobile phone or property management system.|
|door lock member|Door lock members are divided into family members and non-family members. <br/> Family members are members who are added to the user's family. The door lock can be used to manage family members and set the unlock mode. <br/> Non-family members are members created in door locks and can be managed through door lock related interfaces.|

## WiFi Door Lock

[WIFI Door Lock Document](en/resource/WiFiLockDoc.md)

## BLE Door Lock

[ WiFi Door Lock Document](en/resource/BLELockDoc.md)