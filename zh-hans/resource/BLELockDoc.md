# 蓝牙门锁功能说明

## 专有名词解释

|                       | 介绍                                                         |
| --------------------- | ------------------------------------------------------------ |
|dpCode|设备功能点的标识符。设备中的每个功能点都有名称和编号，可参考[蓝牙门锁功能点列表](#蓝牙门锁功能点列表)|
| 劫持                  | 门锁劫持是指将录入特定的密码（指纹、密码等），设置为劫持密码，当受到劫持使用该密码进行开锁时，被迫开门，门锁将防劫持特殊开门报警信息发送至家人手机或物业管理系统。 |
| 门锁成员              | 门锁成员分为家庭成员与非家庭成员。<br />家庭成员即为涂鸦全屋智能的家庭成员概念，门锁内可将对应的门锁密码编号与该账号关联起来；<br />非家庭成员即为门锁内的成员，跟随设备关联，可以创建并分配，门锁内可将对应的门锁密码编号与该成员关联起来。 |
| lockUserId  和 userId | lockUserId 是创建门锁成员时，云端为设备分配的固件成员 id，代表着固件内记录的用户 id 号<br />userId 是创建门锁成员时，云端分配的数据库记录 id，代表着用户的唯一 id |


## 使用说明

| 类名               | 说明                                 |
| ------------------ | ------------------------------------ |
| `TuyaOptimusSdk`   | 初始化SDK入口，用来获取门锁管理类    |
| `ITuyaLockManager` | 门锁管理类，可以获取不同类型的门锁类 |
| `ITuyaBleLock` | 蓝牙门锁类，所有蓝牙门锁相关方法都在其中 |

**示例代码**

通过设备id创建蓝牙门锁类。

```java
// 初始化SDK，仅需要调用一次
TuyaOptimusSdk.init(getApplicationContext());
// 获取ITuyaLockManager
ITuyaLockManager tuyaLockManager = TuyaOptimusSdk.getManager(ITuyaLockManager.class);
// 创建 ITuyaBleLock
ITuyaBleLock tuyaLockDevice = tuyaLockManager.getBleLock(your_device_id);
```

## 门锁成员管理
门锁内可以分为家庭成员和门锁成员，家庭成员为全屋智能中的概念，具体可以查阅[家庭成员管理](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/zh-hans/resource/HomeMember.html)

### 获取门锁成员列表
**接口说明**

```java
/**
 * get lock users
 */
public void getLockUsers(final ITuyaResultCallback<List<BLELockUser>> callback)
```

**参数说明**

**`BLELockUser ` 数据模型**

|字段|类型|描述|
|---|---|---|
|userId|String|用户id|
|lockUserId|int|门锁中的用户id|
|userContact|String|联系方式|
|nickName|String|昵称|
|avatarUrl|String|头像url|
|userType|int|用户类型，10为管理员，20为普通家庭用户，30为门锁用户|
|supportUnlockTypes|List<String>|支持的解锁类型，可以查看TuyaUnlockType中的解锁方式|
|effectiveTimestamp|long|用户生效时间戳，单位 ms|
|invalidTimestamp| long |用户失效时间戳，单位 ms|


**示例代码**

```java
tuyaLockDevice.getLockUsers(new ITuyaResultCallback<List<BLELockUser>>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock users failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(List<BLELockUser> user) {
        Log.i(TAG, "get lock users success: lockUserBean = " + user);
    }
});
```

### 创建门锁成员

SDK 支持往门锁里添加成员，后续可以单独为该成员进行解锁方式绑定

```sequence
Title: 创建门锁成员流程

participant app
participant 云端

note over app: 用户填写成员信息
app->云端: 调用接口，创建用户
云端-->app: 生成门锁用户 id、硬件成员id，返回创建结果
note over app: 处理、显示结果
```

> 注意，这里创建的非家庭成员，家庭成员的管理需要参考文档[家庭成员管理](https://tuyainc.github.io/tuyasmart_home_android_sdk_doc/zh-hans/resource/HomeMember.html)

**接口说明**

```java
/**
 * add lock user
 *
 * @param userName           userName
 * @param allowedUnlock      Whether allowed unlock with bluetooth
 * @param permanent          Whether the user is permanent
 * @param effectiveTimestamp User effective time
 * @param invalidTimestamp   User invalid time
 * @param avatarFile         avatar
 * @param callback           callback
 */
void addLockUser(final String userName, boolean allowedUnlock, boolean permanent, long effectiveTimestamp, long invalidTimestamp, File avatarFile, final ITuyaResultCallback<Boolean> callback);
```

**参数说明**

|参数|说明|
|---|---|
| userName |用户名称|
| allowedUnlock |允许用户使用蓝牙开锁|
| unlockType |解锁方式，可查看TuyaUnlockType类|
| permanent |是否是永久用户|
| effectiveTimestamp |用户生效时间戳，单位ms, 如果permanent 值为true，则该值可忽略|
| invalidTimestamp |用户失效时间戳，单位ms, 如果permanent 值为true，则该值可忽略|
| avatarFile |用户头像文件|

**示例代码**

```java
tuyaLockDevice.addLockUser("your_user_name", true, true, 0, 0, null, new ITuyaResultCallback<Boolean>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "add lock user failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Boolean result) {
        Log.i(TAG, "add lock user success");
    }
});
```

### 更新门锁成员

SDK 提供了修改门锁成员的功能，修改门锁成员会和硬件进行交互，需要设备保持蓝牙连接

```sequence
Title: 更新门锁成员流程

participant 云端
participant app
participant 门锁

note over app: 用户填写新的成员信息
app->门锁: 建立蓝牙连接
app->门锁: 发送更新用户信息指令
门锁-->app: 回复更新结果
app->云端: 调用接口，更新用户
云端-->app: 返回更新结果
note over app: 处理、显示结果
```

**接口说明**

```java
/**
 * update lock user
 *
 * @param userId             userId
 * @param userName           userName
 * @param allowedUnlock      Whether allowed unlock with bluetooth
 * @param permanent          Whether the user is permanent
 * @param effectiveTimestamp User effective time
 * @param invalidTimestamp   User invalid time
 * @param avatarFile         avatar
 * @param callback           callback
 */
void updateLockUser(final String userId, boolean allowedUnlock, final String userName, final boolean permanent, long effectiveTimestamp, long invalidTimestamp, File avatarFile, final ITuyaResultCallback<Boolean> callback);

```

**参数说明**

|参数|说明|
|---|---|
| userId |用户id|
| allowedUnlock |允许用户使用蓝牙开锁|
| userName |用户名称|
| permanent |是否是永久用户|
| effectiveTimestamp |用户生效时间戳，单位ms, 如果permanent 值为true，则该值可忽略|
| invalidTimestamp |用户失效时间戳，单位ms, 如果permanent 值为true，则该值可忽略|
| avatarFile |用户头像文件|

**示例代码**

```java
tuyaLockDevice.updateLockUser("your_user_id", true, "your_user_name", true, 0, 0, null, new ITuyaResultCallback<Boolean>() {
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

### 删除门锁成员

SDK 提供了删除门锁成员的功能，**删除门锁成员会和硬件进行交互，会删除该用户下所有的开锁方式、密码等，操作需要设备保持蓝牙连接**

```sequence
Title: 删除门锁成员流程

participant 云端
participant app
participant 门锁

note over app: 用户确认删除成员
app->门锁: 建立蓝牙连接
app->门锁: 发送删除用户信息指令
门锁-->app: 回复删除结果
app->云端: 调用接口，删除用户
云端-->app: 返回删除结果
note over app: 处理、显示结果
```

**接口说明**

```java
/**
 * delete lock user
 * @param user user bean
 * @param callback  callback
 */
public void deleteLockUser(BLELockUser user, final ITuyaResultCallback<Boolean> callback)
```

## 设备蓝牙连接状态
蓝牙门锁需要 App 开启蓝牙后，部分功能才能正常使用。

### 判断门锁蓝牙是否连接

**接口说明**

判断蓝牙门锁是否和手机连接。

控制门锁相关的操作基本都需要调用这个接口判断，必须门锁是在线状态才能操作。

```java
/**
 *  @return if lock online, return true
 */
public boolean isBLEConnected() 
```

**示例代码**

```java
boolean online = tuyaLockDevice.isBLEConnected();
```

### 连接蓝牙门锁

**接口说明**

如果判断门锁未连接，调用此接口连接到门锁

```java
/**
 * connect to lock
 *
 * @param connectListener callback BLE lock connect status
 */
public void connect(ConnectListener connectListener)
```

**参数说明**

ConnectListener为设备连接状态回调，其中的onStatusChanged将返回在线状态。

**示例代码**

```java
tuyaLockDevice.connect(new ConnectListener() {
    @Override
    public void onStatusChanged(boolean online) {
        Log.i(TAG, "onStatusChanged  online: " + online);
    }
});
```


## 动态密码

使用 SDK 获取动态密码并在门锁上进行输入后即可开锁，动态密码有效时间为 5 分钟

### 获取动态密码
**接口说明**

```java
public void getDynamicPassword(final ITuyaResultCallback<String> callback)
```
**参数说明**

| 参数    | 说明                                       |
| ------- | ------------------------------------------ |
| callback | 接口回调 |

**示例代码**

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


## 蓝牙解锁 & 落锁

### 蓝牙解锁

```sequence
Title: 蓝牙解锁开门流程

participant 用户
participant app
participant 门锁

note over app: 蓝牙开启，连接上门锁
用户->app: 点击开锁
app->门锁: 发送蓝牙开锁指令
note over 门锁: 收到蓝牙开锁指令，开锁
门锁-->app: 返回开锁结果
note over app: 处理、显示结果
```


**接口说明**

```java
/**
 * unlock the door
 */
public void unlock(String lockUserId)
```

**参数说明**

|参数|说明|
|---|---|
| lockUserId |门锁设备中的用户id|

所有用户在门锁中都会有一个对应的id，id从1开始，每添加一个新用户数字加一。

**示例代码**

```java
// "1"是当前用户在门锁设备中的id
tuyaLockDevice.unlock("1");
```

###  蓝牙落锁
```sequence
Title: 蓝牙落锁流程

participant 用户
participant app
participant 门锁

note over app: 蓝牙开启，连接上门锁
用户->app: 点击落锁
app->门锁: 发送蓝牙落锁指令
note over 门锁: 收到蓝牙落锁指令，开锁
门锁-->app: 返回落锁结果
note over app: 处理、显示结果
```

**接口说明**

门锁与app连接后，可调用此接口上锁。

```java
/**
 * lock the door
 */
public void lock()
```

**示例代码**

```java
tuyaLockDevice.lock();
```
## 门锁记录

### 获取告警记录

**接口说明**

```java
/**
 * get alarm records
 * @param offset page number
 * @param limit item count
 * @param callback callback
 */
void getAlarmRecords(int offset, int limit, final ITuyaResultCallback<Record> callback);
```


**参数说明**

|参数|说明|
|---|---|
| offset |记录页码|
| limit |返回的记录条目数量|

**`Record ` 数据模型**

`Record`字段说明

|字段|类型|描述|
|---|---|---|
|totalCount|int|总条目|
|hasNext|boolean|是否有下一页|
|datas|List<DataBean>|记录数据内容|

其中`DataBean`字段说明如下：

| 字段     | 类型                    | 描述                           |
| -------- | ----------------------- | ------------------------------ |
| userId   | String                | 成员 id                        |
| userName | String                | 用户昵称                       |
| unlockType     | String          | 解锁类型         |
| devId    | String                | 设备 id                        |
|createTime|long|该记录的时间戳|
| tags     | int               | 标位，0表示其他，1表示劫持报警 |
|unlockRelation|UnlockRelation|解锁类型和解锁密码编号的关系实例，如不是开锁记录，可为空|

**示例代码**

```java
tuyaLockDevice. getAlarmRecords(0, 10, new ITuyaResultCallback<Record>() {
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

### 获取解锁记录

**接口说明**

```java
/**
 * get unlock records
 * @param unlockTypes unlock type list 
 * @param offset page number
 * @param limit item count
 * @param callback callback
 */
void getUnlockRecords(int offset, int limit, final ITuyaResultCallback<Record> callback);
```

**参数说明**

|参数|说明|
|---|---|
| offset |记录页码|
| limit |返回的记录条目数量|


**示例代码**

```java
tuyaLockDevice.getUnlockRecords(0, 10, new ITuyaResultCallback<Record>() {
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


## 解锁方式管理

本节提供添加、修改、删除解锁方式的接口。

下图为添加解锁方式的交互过程：

```mermaid
sequenceDiagram
SDK->>Lock: Connect
Note left of Lock: Connect via Bluetooth
Lock-->>SDK: Connect success
SDK->>Lock: Add unloack mode
Lock-->>SDK: Success
SDK->>Server: Notify to server
loop save info
	Server->>Server: 
end
Server-->>SDK: Success
```

### 获取解锁方式列表

**接口说明**

```java
/**
 * get unlock mode by unlockType
 *
 * @param unlockType unlock type {@link com.tuya.smart.optimus.lock.api.TuyaUnlockType}
 * @param callback callback
 */
void getUnlockModeList(String unlockType, final ITuyaResultCallback<ArrayList<UnlockMode>> callback);
```


**参数说明**

**`UnlockMode` 数据模型**

|字段|类型|描述|
|---|---|---|
|userId|String|用户id|
|lockUserId|int|门锁中的用户id|
|userName|String|用户昵称|
|unlockAttr|int|解锁方式属性，0是普通密码，1是劫持密码|
|userType|int|用户类型，10为管理员，20是普通用户，30是门锁用户|
|unlockModeId|String|当前解锁方式在服务端的id|
|unlockId|String|当前解锁方式在门锁中的id|
|unlockName|String|当前解锁方式的名称|
|unlockType|String|解锁方式类型，参见TuyaUnlockType|

**示例代码**

```java
tuyaLockDevice.getUnlockModeList(TuyaUnlockType.PASSWORD, new ITuyaResultCallback<ArrayList<UnlockMode>>() {
    @Override
    public void onSuccess(ArrayList<UnlockMode> result) {
        Log.i(TAG, "getUnlockModeList  onSuccess: " + result);
    }

    @Override
    public void onError(String errorCode, String errorMessage) {
        Log.e(TAG, "getUnlockModeList failed: code = " + errorCode + "  message = " + errorMessage);
    }
});
```


### 注册开门方式监听


添加、修改、删除开锁方式的接口均为异步调用，都是从这个接口返回。

**接口说明**

注册开门方式监听

```java
void setUnlockModeListener(UnlockModeListener unlockModeListener);
```

其中`UnlockModeListener`中代码如下：


```java
public interface UnlockModeListener {

    /**
     * 解锁方式参数丢失
     */
    int FAILED_STATUS_ILLEGAL_ARGUMENT = -1;
    /**
     * 当前操作不支持此开锁方式
     */
    int FAILED_STATUS_NOT_SUPPORT_UNLOCK_TYPE = -2;
    /**
     * 控制指令发送失败
     */
    int FAILED_STATUS_SEND_ERROR = -3;
    /**
     * 请求服务端失败
     */
    int FAILED_STATUS_REQUEST_SERVER_ERROR = -4;
    /**
     * 指纹不完整
     */
    int FAILED_STATUS_FINGERPRINT_INCOMPLETE = -5;
    /**
     * 服务端响应失败
     */
    int FAILED_STATUS_SERVER_RESPONSE_FAILED = -6;
    /**
     * 门锁响应失败
     */
    int FAILED_STATUS_LOCK_RESPONSE_FAILED = -7;
    /*-----以下状态为门锁中定义，创建开锁方式失败的返回码-------*/
    int FAILED_STATUS_TIMEOUT = 0x00;
    int FAILED_STATUS_FAILED = 0x01;
    int FAILED_STATUS_REPEAT = 0x02;
    int FAILED_STATUS_LOCK_ID_EXHAUSTED = 0x03;
    int FAILED_STATUS_PASSWORD_NOT_NUMBER = 0x04;
    int FAILED_STATUS_PASSWORD_WRONG_LENGTH = 0x05;
    int FAILED_STATUS_NOT_SUPPORT = 0x06;
    int FAILED_STATUS_ALREADY_ENTERED = 0x07;
    int FAILED_STATUS_ALREADY_BOUND_CARD = 0x08;
    int FAILED_STATUS_ALREADY_BOUND_FACE = 0x09;
    int FAILED_STATUS_PASSWORD_TOO_SIMPLE = 0x0A;
    int FAILED_STATUS_WRONG_LOCK_ID = 0xFE;

    /**
     * 开门方式添加、删除、修改的回调方法
     *
     * @param devId              设备id
     * @param userId             用户id
     * @param unlockModeResponse 响应数据
     */
    void onResult(String devId, String userId, UnlockModeResponse unlockModeResponse);
}
```

**参数说明**


|字段|类型|描述|
|---|---|---|
|unlockMethod|String|当前操作门锁的方法类型，其值有以下三个：<br>/** 添加开锁方式  \*/ <br/>public static final String UNLOCK\_METHOD\_CREATE = "unlock\_method\_create"; <br/>/** 修改开锁方式  \*/ <br/>public static final String UNLOCK\_METHOD\_MODIFY = "unlock\_method\_modify";<br/> /** 删除开锁方式  \*/ <br/>public static final String UNLOCK\_METHOD\_DELETE = "unlock\_method\_delete";|
|unlockType|String|解锁方式类型，参见TuyaUnlockType|
|stage|int|当前操作门锁所处的阶段，其值定义在`BleLockConstant`中，详情如下：<br>int STAGE_AFTER = -2; // 操作门锁完成之后，和服务端同步数据<br/>int STAGE_BEFORE = -1;// 与门锁交互之前，如发送命令、和服务端交互<br/> int STAGE_START = 0x00; // 开始录入解锁方式<br/>int STAGE_CANCEL = 0xFE;// 解锁方式录入取消<br/> int STAGE_FAILED = 0xFD;// 解锁方式录入失败 <br/>int STAGE_ENTERING = 0xFC;// 解锁方式录入中<br/> int STAGE_SUCCESS = 0xFF;// 解锁方式录入成功|
|lockUserId|int|在门锁中的用户id|
|unlockId|int|当前解锁方式在门锁中的id|
|unlockModeId|String|当前解锁方式在服务端的id|
|admin|boolean|是否是管理员|
|times|int|0表示永久有效，1~254表示实际有效次数，其他数字无效|
|status|int|失败的状态码，详情查看UnlockModeListener|
|failedStage|int|失败发生的阶段。stage=STAGE_FAILED时才有意义|

**示例代码**

```java
tuyaLockDevice.setUnlockModeListener(new UnlockModeListener() {
  @Override
  public void onResult(String devId, String userId, UnlockModeResponse unlockModeResponse) {
    Log.i(TAG, "UnlockModeListener devId: " + devId);
    Log.i(TAG, "UnlockModeListener userId: " + userId);
    Log.i(TAG, "UnlockModeListener unlockType: " + unlockModeResponse.unlockType);
    Log.d(TAG, "UnlockModeListener: " + unlockModeResponse);
    if (unlockModeResponse.success) {
      if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_CREATE)) {
        Log.i(TAG, "Create unlock mode success");
      } else if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_MODIFY)) {
        Log.i(TAG, "Modify unlock mode success");
      } else if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_DELETE)) {
        Log.i(TAG, "Delete unlock mode success");
      }
    } else if (unlockModeResponse.stage == BleLockConstant.STAGE_FAILED) {
      if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_CREATE)) {
        Log.w(TAG, "Create unlock mode failed.");
        Log.w(TAG, "Create unlock mode failed reason: " + unlockModeResponse.status);
        Log.w(TAG, "Create unlock mode failed stage: " + unlockModeResponse.failedStage);
      } else if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_MODIFY)) {
        Log.w(TAG, "Modify unlock mode failed.");
        Log.w(TAG, "Modify unlock mode failed reason: " + unlockModeResponse.status);
        Log.w(TAG, "Modify unlock mode failed stage: " + unlockModeResponse.failedStage);
      } else if (TextUtils.equals(unlockModeResponse.unlockMethod, UnlockModeResponse.UNLOCK_METHOD_DELETE)) {
        Log.w(TAG, "Delete unlock mode failed.");
        Log.w(TAG, "Delete unlock mode failed reason: " + unlockModeResponse.status);
        Log.w(TAG, "Delete unlock mode failed stage: " + unlockModeResponse.failedStage);
      }
    }
  }
});
```

### 添加开锁方式


**接口说明**

按解锁类型给用户添加解锁方式。**需要设备保持蓝牙连接**

```java
/**
 * Add unlock method.
 *
 * @param unlockType TuyaUnlockType {@link com.tuya.smart.optimus.lock.api.TuyaUnlockType}
 * @param user       BLELockUser {@link com.tuya.smart.sdk.optimus.lock.bean.ble.BLELockUser}
 * @param name       Unlock mode name.
 * @param password   Unlock mode password. If it is not the password unlock method, this field can be null
 * @param times      Number of times the unlock mode can be used. The value range is 0 to 254, 0 means unlimited times, and 1 ~ 254 is the actual number of times.
 * @param isHijack   Hijack flag. If it is true, a hijacking alarm will be triggered when unlocking with this unlock mode.
 */
void addUnlockMode(final String unlockType, final BLELockUser user, String name, String password, int times, boolean isHijack);
```

**参数说明**

|字段|描述|
|---|---|
|unlockType|解锁方式类型，参见TuyaUnlockType|
|user|BLELockUser，用户的数据模型|
|name|解锁方式名称|
|password|解锁密码，解锁方式为密码类型必填，其他解锁方式可为空|
|times|密码有效次数。0表示永久有效，1~254表示实际有效次数，其他数字无效|
|isHijack|劫持标记，设置为true时使用该密码解锁将触发劫持告警|

**示例代码**

给家庭用户添加密码

```java
tuyaLockDevice.getHomeUsers(new ITuyaResultCallback<List<BLELockUser>>() {
    @Override
    public void onSuccess(List<BLELockUser> result) {
        Log.i(TAG, "getHomeUsers  onSuccess: " + result);
        // add password unlock mode
        tuyaLockDevice.addUnlockMode(TuyaUnlockType.PASSWORD, result.get(0), "test_unlock_mode1", "431232", 0, false);
    }

    @Override
    public void onError(String errorCode, String errorMessage) {
        Log.e(TAG, "getHomeUsers failed: code = " + errorCode + "  message = " + errorMessage);
    }
});
```

### 更新开锁方式信息

**接口说明**

更新开锁方式的名称，修改劫持标记。**不需要设备保持蓝牙连接**

此接口不与门锁交互，仅和服务端通信。

```java
/**
 * Update name and hijack flag of the unlocking method. Only update server information, not communicate with door lock device
 *
 * @param unlockMode UnlockMode bean {@link com.tuya.smart.sdk.optimus.lock.bean.ble.UnlockMode}
 * @param name       Unlock mode name
 * @param isHijack   Hijack flag. If it is true, a hijacking alarm will be triggered when unlocking with this unlock mode.
 */
void updateUnlockModeServerInfo(UnlockMode unlockMode, String name, boolean isHijack);
```

**参数说明**

| 字段       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| unlockMode | 解锁方式数据模型                            |
| name       | 解锁方式名称                                                 |
| isHijack   | 劫持标记，设置为true时使用该密码解锁将触发劫持告警           |

**示例代码**

```java
tuyaLockDevice.getUnlockModeList(TuyaUnlockType.FINGERPRINT, new ITuyaResultCallback<ArrayList<UnlockMode>>() {
    @Override
    public void onSuccess(ArrayList<UnlockMode> result) {
        Log.i(TAG, "getUnlockModeList  onSuccess: " + result);
        for (UnlockMode unlockMode : result) {
            if (TextUtils.equals(unlockMode.unlockName, "test_unlock_mode1")) {
                tuyaLockDevice.updateUnlockModeServerInfo(unlockMode, "test_unlock2", false);// rename unlock mode
            }
        }
    }

    @Override
    public void onError(String errorCode, String errorMessage) {
        Log.e(TAG, "getUnlockModeList failed: code = " + errorCode + "  message = " + errorMessage);
    }
});
```

### 删除开锁方式

**接口说明**

删除指定的开锁方式。**需要设备保持蓝牙连接**

```java
/**
 * Delete unlockMode.
 *
 * @param unlockMode unlockMode
 */
void deleteUnlockMode(UnlockMode unlockMode);
```

**示例代码**

```java
tuyaLockDevice.deleteUnlockMode(unlockMode);
```

### 指纹录入取消

**接口说明**

指纹解锁方式一般需要录入4到5次指纹，如果录入过程中需要取消，可以调用此接口。**需要设备保持蓝牙连接**

```java
/**
 * Cancel fingerprint entry.
 * <p>
 * The fingerprint entry process will be repeated multiple times and can be cancelled during the entry process.
 *
 * @param user BLELockUser {@link com.tuya.smart.sdk.optimus.lock.bean.ble.BLELockUser}
 */
void cancelFingerprintUnlockMode(final BLELockUser user);
```
### 密码类型更新密码

**接口说明**

密码类型的解锁方式设置之后可以更新密码，可以调用此接口修改密码名称、密码、解锁次数和劫持标记。**需要设备保持蓝牙连接**

注意：仅密码类型的支持调用此接口。

```java
/**
 * Update the name, password, validity period and other information of the unlocking method
 *
 * @param unlockMode UnlockMode bean {@link com.tuya.smart.sdk.optimus.lock.bean.ble.UnlockMode}
 * @param name       Unlock mode name
 * @param password   Unlock mode password. If it is not the password unlock method, this field can be null
 * @param times      Number of times the unlock mode can be used. The value range is 0 to 254, 0 means unlimited times, and 1 ~ 254 is the actual number of times.
 * @param isHijack   Hijack flag. If it is true, a hijacking alarm will be triggered when unlocking with this unlock mode.
 */
void updatePasswordUnlockMode(UnlockMode unlockMode, String name, String password, int times, boolean isHijack);
```

**参数说明**

| 字段       | 描述                                               |
| ---------- | -------------------------------------------------- |
| unlockMode | 解锁方式数据模型                                   |
| name       | 解锁方式名称                                       |
|password|解锁密码，解锁方式为密码类型必填，其他解锁方式可为空|
|times|密码有效次数。0表示永久有效，1~254表示实际有效次数，其他数字无效|
| isHijack   | 劫持标记，设置为true时使用该密码解锁将触发劫持告警 |

**示例代码**

```java
tuyaLockDevice.getUnlockModeList(TuyaUnlockType.PASSWORD, new ITuyaResultCallback<ArrayList<UnlockMode>>() {
    @Override
    public void onSuccess(ArrayList<UnlockMode> result) {
        Log.i(TAG, "getUnlockModeList  onSuccess: " + result);
        for (UnlockMode unlockMode : result) {
            if (TextUtils.equals(unlockMode.unlockName, "test_password")) {
                tuyaLockDevice.updatePasswordUnlockMode(unlockMode, "test_password", "131232", 0, false);// modify password
            }
        }
    }

    @Override
    public void onError(String errorCode, String errorMessage) {
        Log.e(TAG, "getUnlockModeList failed: code = " + errorCode + "  message = " + errorMessage);
    }
});
```

## 蓝牙门锁功能点列表

| dp name                  | dp code                     |
| ------------------------ | --------------------------- |
| 添加开门方式             | unlock_method_create        |
| 删除开门方式             | unlock_method_delete        |
| 修改开门方式             | unlock_method_modify        |
| 冻结开门方式             | unlock_method_freeze        |
| 解冻开门方式             | unlock_method_enable        |
| 蓝牙解锁                 | bluetooth_unlock            |
| 蓝牙解锁反馈             | bluetooth_unlock_fb         |
| 剩余电量                 | residual_electricity        |
| 电量状态                 | battery_state               |
| 童锁状态                 | child_lock                  |
| 上提反锁                 | anti_lock_outside           |
| 指纹解锁                 | unlock_fingerprint          |
| 普通密码解锁             | unlock_password             |
| 动态密码解锁             | unlock_dynamic              |
| 卡片解锁                 | unlock_card                 |
| 钥匙解锁                 | unlock_key                  |
| 开关门事件               | open_close                  |
| 从门内侧打开门锁         | open_inside                 |
| 蓝牙解锁记录             | unlock_ble                  |
| 门被打开                 | door_opened                 |
| 告警                     | alarm_lock                  |
| 劫持报警                 | hijack                      |
| 门铃呼叫                 | doorbell                    |
| 短信通知                 | message                     |
| 门铃选择                 | doorbell_song               |
| 门铃音量                 | doorbell_volume             |
| 门锁语言切换             | language                    |
| 显示屏欢迎词管理         | welcome_words               |
| 按键音量                 | key_tone                    |
| 门锁本地导航音量         | beep_volume                 |
| 反锁状态                 | reverse_lock                |
| 自动落锁开关             | automatic_lock              |
| 单一解锁与组合解锁切换   | unlock_switch               |
| 同步成员开门方式         | synch_member                |
| 自动落锁延时设置         | auto_lock_time              |
| 定时自动落锁             | auto_lock_timer             |
| 指纹录入次数             | finger_input_times          |
| 人脸识别解锁             | unlock_face                 |
| 开合状态                 | closed_opened               |
| 虹膜解锁                 | unlock_eye                  |
| 掌纹解锁                 | unlock_hand                 |
| 指静脉解锁               | unlock_finger_vein          |
| 硬件时钟RTC              | rtc_lock                    |
| 自动落锁倒计时上报       | auto_lock_countdown         |
| 手动落锁                 | manual_lock                 |
| 落锁状态                 | lock_motor_state            |
| 锁帖电机转动方向     | lock_motor_direction        |
| 冻结用户                 | unlock_user_freeze          |
| 解冻用户                 | unlock_user_enable          |
| 蓝牙锁临时密码添加       | temporary password_creat    |
| 蓝牙锁临时密码删除       | temporary password_delete   |
| 蓝牙锁临时密码修改       | temporary password_modify   |
| 同步开门方式（数据量大） | synch_method                |
| 临时密码解锁             | unlock_temporary            |
| 电机扭力                 | motor_torque                |
| 组合开锁记录             | unlock_double               |
| 离家布防开关             | arming_mode                 |
| 配置新免密远程解锁       | remote_no_pd_setkey         |
| 新免密远程开门-带密钥    | remote_no_dp_key            |
| 远程手机解锁上报         | unlock_phone_remote         |
| 远程语音解锁上报         | unlock_voice_remote         |
| 离线密码T0时间下发       | password_offline_time       |
| 单条离线密码清空上报     | unlock_offline_clear_single |
| 离线密码清空上报         | unlock_offline_clear        |
| 离线密码解锁上报         | unlock_offline_pd           |
