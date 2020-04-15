[Chinese Document](README.md)

# Tuya Smart Lock Android SDK

** The SDK is currently in the developer preview stage and is still being continuously improved. If you have any questions or suggestions, please contact us. **

[TOC]

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
      google()
        jcenter()
      }
    }
    ```
2. Add the door lock SDK dependency in the dependencies of `build.gradle` in the module layer

    ```groovy
    dependencies {
        ...
       implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.1-beta1-SNAPSHOT'
    }
    ```

## Term Explanation

|Term|Explanation|
|---|---|
|dpCode|The identifier of the device function point. Each function point in the device has a name and number. This document is for Wi-Fi door lock devices, please refer to [Wi-Fi Door Lock Function Points](#wi-fi-door-lock-function-points)|
|hijack|Door lock hijacking refers to setting a specific password (fingerprint, password, etc.) as the hijacking password. <br/> When the user enters this password to open the door, the door lock considers the user to open the door involuntarily, and sends the alarm information to the family's mobile phone or property management system.|
|door lock member|Door lock members are divided into family members and non-family members. <br/> Family members are members who are added to the user's family. The door lock can be used to manage family members and set the unlock mode. <br/> Non-family members are members created in door locks and can be managed through door lock related interfaces.|

## Tuya WiFi Lock APIs

The following door lock methods are encapsulated in the TuyaWifiLockDevice class, which is used after creating a TuyaWifiLockDevice object by passing in the device id:

```java
TuyaWifiLockDevice tuyaLockDevice = new TuyaWifiLockDevice(deviceId);
```

### Door Lock Member Management
The door lock can be divided into family members and non-family members. Family members are Tuya home family members. For details, please refer to [Family Management](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/en/resource/HomeManager.html).

The following describes non-family member management operations in door locks

#### Get Door Lock Members

**Description**

```java
public void getUsers(final ITuyaResultCallback<List<LockUserBean>> callback)
```

**Parameters**

**`LockUserBean` Description**

|Field|Type|Description|
|---|---|---|
|userId|String|user id|
|userName| String |user name|
|avatarUrl| String |avatar url|
|contact| String |user's contact, email or phone number|
|unlockRelations|List<UnlockRelationBean>|unlock type and password sequenceNumber|
|devId| String |door lock device id|
|ownerId| String |user's home id|
|userType|int|door lock member type, 1: Family member 2: Non-family member|


**Example**

```java
tuyaLockDevice.getUsers(new ITuyaResultCallback<List<LockUser>>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock users failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(List<LockUser> wifiLockUserBean) {
        Log.i(TAG, "get lock users success: wifiLockUserBean = " + wifiLockUserBean);
    }
});
```

#### Create a Door Lock Member
**Description**

Create non-family members. 

```java
public void addUser(final String userName, File avatarFile, final List<UnlockRelation> unlockRelations, final ITuyaResultCallback<String> callback)
```

**Parameters**

|Parameter|Nullable|Description|
|----|----|----|
|userName|No|user name|
| avatarFile |Yes|file of avatar|
| unlockRelationBeans |No|list of UnlockRelationBean|

**`UnlockRelationBean` Description**

|Field|Type|Description|
|----|----|----|
|TuyaUnlockType|enum|unlock type|
|passwordNumber|int|password sequence number, 0 - 999|


**Example**

```java
ArrayList<UnlockRelation> unlockRelations = new ArrayList<>();
UnlockRelation unlockRelation = new UnlockRelation();
unlockRelation.unlockType = TuyaUnlockType.PASSWORD;
unlockRelation.passwordNumber = 1;
unlockRelations.add(unlockRelation);
File avatarFile = new File(getFilesDir(), "1.png");
tuyaLockDevice.addUser("Pan", avatarFile , unlockRelations, new ITuyaResultCallback<String>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "add lock user failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(String userId) {
        Log.i(TAG, "add lock user success: " + userId);
    }
});
```
**Tips**

`UnlockRelation` can be obtained from the [Get Door Lock Records](#get-door-lock-records) section.

When the door lock uses a password or other unlocking method, the unlocking record can be obtained, and then the unlocking method in the unlocking record can be assigned to a user.

#### Update a Door Lock Member Information
**Description**

Use SDK to update door lock member information, including username, avatar, unlock password correspondence, etc.

```java
public void updateUser(final String userId, final String userName, File avatarFile, final List<UnlockRelation> unlockRelations, final ITuyaResultCallback<Boolean> callback)
```

**Parameters**

|Parameter|Nullable|Description|
|----|----|----|
|userId|No|user id|
|userName|Yes|Member user name, optional, not modified if null|
|avatarImage|Yes|Member user avatar, optional, not modified if null|
|unlockRelations|No|Association between member unlocking method and password sequence number. If not modified, fill in the original value|

**Example**

```java
ArrayList<UnlockRelation> unlockRelations = new ArrayList<>();
UnlockRelation unlockRelation = new UnlockRelation();
unlockRelation.unlockType = TuyaUnlockType.PASSWORD;
unlockRelation.passwordNumber = 1;
unlockRelations.add(unlockRelation);
tuyaLockDevice.updateUser("0000005f1g", "pan", null, unlockRelations, new ITuyaResultCallback<Boolean>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "update lock user failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Boolean aBoolean) {
        Log.i(TAG, "update lock user success");
    }
});
```

#### Delete a Door Lock Member

**Description**

Use the SDK to delete door lock member information. Deleting members does not delete existing passwords

```java
public void deleteUser(String userId, final ITuyaResultCallback<Boolean> callback)
```

**Parameters**

|Parameter|Description|
|---|---|
|userId|user id|

**Example**

```java
tuyaLockDevice.deleteUser("0000004pnk", new ITuyaResultCallback<Boolean>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "delete lock user failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Boolean result) {
        Log.i(TAG, "delete lock user failed success");
    }
});
```

### Temporary Password

Use the SDK to create a temporary password and enter it on the door lock to unlock it.

#### Create Temporary Password
**Description**

The temporary password can customize the validity period of the password. When the password is created, it needs to be synchronized on the door lock device.

```java
public void createTempPassword(TempPassword tempPassword, final ITuyaResultCallback<Boolean> callback)
```

**Parameters**

`TempPassword` created by `TempPassword.Builder`, for details, refer to the following code example.

The following is a description of the fields of this class:

|Parameter|Nullable|Description|
|---|---|---|
|name|No|password name|
|password|No|temporary password|
|effectiveDate|No|Password effective timestamp, unit ms|
|invalidDate|No|Password expiration timestamp, unit ms|
|countryCode|Yes| country code，for example 86|
|phone|Yes|phone number, when the creation is successful, the mobile phone user will be notified|

The mobile phone number and country code can be omitted, and the SMS service must be purchased to take effect.

**Example**

```java
TempPasswordBuilder tempPasswordBuilder = new TempPasswordBuilder()
        .name("Liam's password")
        .password("1231231")
        .effectiveTime(System.currentTimeMillis())
        .invalidTime(System.currentTimeMillis() + 24 * 60 * 60 * 1000);
tuyaLockDevice.createTempPassword(tempPasswordBuilder, new ITuyaResultCallback<Boolean>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "create lock temp password: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Boolean result) {
        Log.i(TAG, "add lock user success");
    }
});
```

#### Get Temporary Passwords

**Description**

Use the SDK to get a list of temporary passwords, you can view the status of the use of temporary passwords

	public void getTempPasswords(final ITuyaResultCallback<List<TempPassword>> callback)

**Parameters**


**`TuyaSmartLockTempPwdModel` Data Model**

|Field|Type|Description|
|---|---|---|
|phone| String |phone number|
|name| String |password name|
|countryCode| String |country code|
|effectiveDate|long|Password effective timestamp, unit ms|
|invalidDate|long|Password expiration timestamp, unit ms|
|createTime| long |create timestamp，unit ms|
|id|int|password id|
|sequenceNumber| int |password sequence number|
|status|int|password status|

The password status includes the following types:

```java
int REMOVED = 0;
int INVALID = 1;
int TO_BE_PUBILSH = 2;
int WORKING = 3;
int TO_BE_DELETED = 4;
int EXPIRED = 5;
```

**Example**

```java
tuyaLockDevice.getTempPasswords(new ITuyaResultCallback<List<TempPassword>>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock temp passwords failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(List<TempPassword> tempPasswords) {
        Log.i(TAG, "get lock temp passwords success: tempPasswords" + tempPasswords); 
    }
});
```

#### Delete Temporary Password

**Description**

Use the SDK to delete the temporary password. After the deletion, the door lock device needs to be updated.

```java
public void deleteTempPassword(int passwordId, final ITuyaResultCallback<Boolean> callback)
```

**Parameters**

|Parameter|Description|
|---|---|
|passwordId|Door lock temporary password unique id|

**Example**

```java
tuyaLockDevice.deleteTempPassword(1111, new ITuyaResultCallback<Boolean>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "delete lock temp password failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Boolean result) {
        Log.i(TAG, "delete lock temp password success");
    }
});
```

### Dynamic Password

Use the SDK to get a dynamic password and enter it on the door lock to unlock. The dynamic password is valid for 5 minutes.

#### Get Dynamic Password
**Description**

```java
public void getDynamicPassword(final ITuyaResultCallback<String> callback)
```

**Example**

```java
tuyaLockDevice.getDynamicPassword(new ITuyaResultCallback<String>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock dynamic password failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(String dynamicPassword) {
        Log.i(TAG, "get lock dynamic password success: dynamicPassword = " + dynamicPassword);
    }
});
```


### Lock Record

Use SDK to obtain door lock records, including unlock records, doorbell records, alarm records, etc.

#### Get Door Lock Records

**Description**

```java
public void getRecords(ArrayList<String> dpIds, int offset, int limit, final ITuyaResultCallback<RecordBean> callback)
```

**Parameters**

|Parameter|Description|
|---|---|
|dpIds|Need to query the unlocking method dp id of the hijacked record. For details, refer to the function definition of the door lock of the corresponding device.|
|offset|page offset|
|limit|item count in one page|

**Parameters in callback**

`RecordBean`  Description

|Field|Type|Description|
|---|---|---|
|totalCount|int|total count of records|
|hasNext|boolean|has next page|
|datas|List<DataBean>|the list of records|

`DataBean` Description

|Field|Type|Description|
|---|---|---|
|userId|String|user id|
|avatarUrl|String|user avatar url|
|userName| String |user name|
|createTime|long|create timestamp, unit ms|
|devId| String |device id|
|dpCodesMap|HashMap<String, Object>|dp data of this record|
|unlockRelation|UnlockRelation| The relationship between the unlock type and the unlock password number, if it is not the unlock record, it can be empty|
|tags|int|record tag，0 means other, 1 means hijack alarm|

**Tips**

You can query the unlocking records, get the `unlockRelation`, and assign it to the created user.

For example, if you create a password on the door lock, and then use the password to unlock, a lock unlocking record is generated. The app can query this unlocking record and assign the password to the user you want to assign.

**Example**

Obtain the door opening record: you can enter the door opening related function points to obtain the door opening record.

```java
ArrayList<String> dpCodes = new ArrayList<>();
dpCodes.add("unlock_fingerprint");
dpCodes.add("unlock_password");
dpCodes.add("unlock_temporary");
dpCodes.add("unlock_dynamic");
dpCodes.add("unlock_card");
dpCodes.add("unlock_face");
dpCodes.add("unlock_key");
dpCodes.add("unlock_app");
dpCodes.add("unlock_eye");
dpCodes.add("unlock_hand");
dpCodes.add("unlock_finger_vein");
tuyaLockDevice.getRecords(dpCodes, 0, 10, new ITuyaResultCallback<Record>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get unlock records failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Record recordBean) {
        Log.i(TAG, "get unlock records success: recordBean = " + recordBean);
    }
});
```


Obtain door lock alarm records: 

```java
ArrayList<String> dpCodes = new ArrayList<>();
dpCodes.add("alarm_lock");
dpCodes.add("hijack");
dpCodes.add("doorbell");
tuyaLockDevice.getRecords(dpCodes, 0, 10, new ITuyaResultCallback<Record>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock records failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Record recordBean) {
        Log.i(TAG, "get lock records success: recordBean = " + recordBean);
    }
});
```


#### Get Door Lock Hijacking Record

Use the SDK to obtain the door lock hijacking record. You can query it based on the unlocking function definition point.

**Description**

```java
public void getLockHijackRecords(ArrayList<String> dpIds, int offset, int limit, final ITuyaResultCallback<RecordBean> callback)
```

### Open the Door Remotely

After triggering a remote door open request on the door lock, you can use the SDK to remotely open the door

To open a door remotely, you need to register for remote door opening monitoring. After receiving the remote door opening request for the door lock, you can call the remote door opening interface.

#### Register Remote Unlock Listener

**Description**

```java
public void setRemoteUnlockListener(RemoteUnlockListener remoteUnlockListener)
```

**Parameters**

RemoteUnlockListener has a method in the interface：

```java
void onReceive(String devId, int second);
```

The method parameters are as follows：

|Field|Type|Description|
|---|---|---|
|devId |String|device id|
|second |int|How many seconds do you need to process|

#### Request Remote Unlock

**Description**

```java
public void replyRemoteUnlock(boolean allow, final ITuyaResultCallback<Boolean> callback)
```

**Parameters**

|Field|Type|Description|
|---|---|---|
| allow |boolean|Whether to allow unlocking|

#### Remote Unlock Example

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_lock_device);
	
	....

    // Register for remote unlock listener
    tuyaLockDevice.setRemoteUnlockListener(new RemoteUnlockListener() {
        @Override
        public void onReceive(String devId, int second) {
            if (second != 0 && !dialogShowing) {
                dialogShowing = true;
                Log.i(TAG, "remote unlock request onReceive");
                onCreateDialog();
            }
        }
    });
}

/**
 * Create remote unlock confirmation dialog
 */
public void onCreateDialog() {
    // Use the Builder class for convenient dialog construction
    AlertDialog.Builder builder = new AlertDialog.Builder(this);
    builder.setMessage("Whether to allow remote unlocking?")
            .setPositiveButton("YES", new DialogInterface.OnClickListener() {
                public void onClick(DialogInterface dialog, int id) {
                    replyRemoteUnlockRequest(true);
                    Log.i(TAG, "remote unlock request access");
                    dialog.dismiss();
                    dialogShowing = false;
                }
            })
            .setNegativeButton("NO", new DialogInterface.OnClickListener() {
                public void onClick(DialogInterface dialog, int id) {
                    replyRemoteUnlockRequest(false);
                    dialog.dismiss();
                    Log.i(TAG, "remote unlock request deny");
                    dialogShowing = false;
                }
            }).setCancelable(false);
    AlertDialog alertDialog = builder.create();
    alertDialog.setCanceledOnTouchOutside(false);
    alertDialog.show();
}

/**
 * Request remote unlock interface
 *
 * @param allow
 */
private void replyRemoteUnlockRequest(boolean allow) {
    tuyaLockDevice.replyRemoteUnlock(allow, new ITuyaResultCallback<Boolean>() {
        @Override
        public void onError(String code, String message) {
            Log.e(TAG, "reply remote unlock failed: code = " + code + "  message = " + message);
        }

        @Override
        public void onSuccess(Boolean result) {
            Log.i(TAG, "reply remote unlock success");
        }
    });
}
```
### Wi-Fi Door Lock Function Points

Dp means data point, it can also be called a function point.

| dp name              | dp code                        |
| -------------------- | ------------------------------ |
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

## Bluetooth door lock APIs

Before accessing the Bluetooth door lock, please configure the device.

The following describes the functional interface of the Bluetooth door lock


```java
TuyaBleLockDevice tuyaLockDevice = new TuyaBleLockDevice(deviceId);
```

### the door lock isOnline

```java
/**
 *
 *  @return if lock online, return true
 */
public boolean isOnline() 
```

example

    boolean online = tuyaLockDevice.isOnline();
### connect the door

```java
tuyaLockDevice.connect(new ConnectListener() {
    @Override
    public void onStatusChanged(boolean online) {
        Log.i(TAG, "onStatusChanged  online: " + online);
    }
});
```

### set lock open or close status callback

    /**
     * callback lock open or close status
     * @param lockStatusListener lock open or close status callback
     */
    public void setLockStatusListener(LockStatusListener lockStatusListener)
    
### unlock the door

    /**
     * unlock the door
     */
    public void unlock(String lockUserId)

### lock the door

    /**
     * lock the door
     */
    public void lock()
    
### get home users

    /**
     * get home users
     */
    public void getHomeUsers(final ITuyaResultCallback<List<User>> callback)
    
### get lock users

    /**
     * get lock users
     */
    public void getLockUsers(final ITuyaResultCallback<List<User>> callback) 
    
### get user info by userId
    /**
     * get user info by userId
     * @param userId userId
     * @param callback callback
     */
    public void getUser(String userId, final ITuyaResultCallback<User> callback)
    
### get current user info

    /**
     * get current user info
     * @param userId userId
     * @param callback callback
     */
    public void getCurrentUser(String userId, final ITuyaResultCallback<User> callback)
    
### add lock user

    /**
     * add lock user
     * @param userName userName
     * @param unlockType unlockType {@link com.tuya.smart.optimus.lock.api.TuyaUnlockType}
     * @param permanent Whether the user is permanent
     * @param startTimestamp User effective time
     * @param endTimestamp User expiration time
     * @param avatarFile avatar
     * @param callback  callback
     */
    public void addUser(final String userName, final String unlockType, final boolean permanent, long startTimestamp, long endTimestamp, File avatarFile, final ITuyaResultCallback<Boolean> callback)

### update lock user

    /**
     * update lock user
     * @param userId userId
     * @param userName userName
     * @param unlockType unlockType {@link com.tuya.smart.optimus.lock.api.TuyaUnlockType}
     * @param permanent Whether the user is permanent
     * @param startTimestamp User effective time
     * @param endTimestamp User expiration time
     * @param avatarFile avatar
     * @param callback  callback
     */
    public void updateUser(final String userId, final String userName, final String unlockType, final boolean permanent, long startTimestamp, long endTimestamp, File avatarFile, final ITuyaResultCallback<Boolean> callback)
    

### delete lock user
	    
	/**
	 * delete lock user
	 * @param userId userId
	 * @param callback  callback
	 */
	public void deleteUser(String userId, final ITuyaResultCallback<Boolean> callback)
	 