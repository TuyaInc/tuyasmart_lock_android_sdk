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
|dpCode|The identifier of the device function point. Each function point in the device has a name and number. Please refer to [Door Lock Function Points](#door-lock-function-points)|
|hijack|Door lock hijacking refers to setting a specific password (fingerprint, password, etc.) as the hijacking password. <br/> When the user enters this password to open the door, the door lock considers the user to open the door involuntarily, and sends the alarm information.|
|door lock member|Door lock members are divided into family members and non-family members. <br/> Family members are members who are added to the user's family. The door lock can be used to manage family members and set the unlock mode. <br/> Non-family members are members created in door locks and can be managed through door lock related interfaces.|

## Door Lock Function Points

Dp means data point, it can also be called a function point.

| dp name              | dp code                        |
| -------------------- | ------------------------------ |
|unlock by bluetooth|bluetooth_unlock|
| unlock by fingerprint   | unlock\_fingerprint            |
| unlock by password | unlock\_password               |
| unlock by temporary password | unlock\_temporary              |
| unlock by dynamic password | unlock\_dynamic                |
| unlock by card | unlock\_card                   |
| unlock by face | unlock\_face                   |
| unlock by key | unlock\_key                    |
| alarm record     | alarm\_lock                    |
| apply remote unlock | unlock\_request                |
| reply remote unlock | reply\_unlock\_request         |
| battery status | battery\_state                 |
| residual electricity | residual\_electricity          |
| lock from inside | reverse\_lock                  |
| child lock status | child\_lock                    |
| unlock by app | unlock\_app                    |
| hijack alarm record | hijack                         |
| open the door from inside | open\_inside                   |
| door opening and closing status | closed\_opened                 |
| doorbell alarm record | doorbell                       |
| SMS notifacation             | message                        |
| lock from outside | anti\_lock\_outside            |
| unlock by eye | unlock\_eye                    |
| unlock by hand | unlock\_hand                   |
| unlock by finger vein | unlock\_finger\_vein           |
| update all finger record | update\_all\_finger            |
| update all password record | update\_all\_password          |
| update all card record | update\_all\_card              |
| update all face record | update\_all\_face              |
| update all eye record | update\_all\_eye               |
| update all hand record | update\_all\_hand              |
| update all finger vein record | update\_all\_fin\_vein         |
| offline password unlock report | unlock\_offline\_pd            |
| offline password clear report | unlock\_offline\_clear         |
| single offline password clear report | unlock\_offline\_clear\_single |