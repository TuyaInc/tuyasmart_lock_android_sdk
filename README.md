# 涂鸦智能门锁 Android SDK 文档



[English Document](README_en.md)



涂鸦智能门锁 Android SDK 提供了与智能门锁设备的功能封装，加速和简化门锁应用功能开发过程，主要包括了以下功能：

* 门锁用户体系 （包括门锁用户管理、关联密码等功能）
* 门锁密码解锁（包括动态密码获取、临时密码管理相关的功能）
* 门锁使用记录（包括门锁开锁记录、门铃记录、报警记录等功能）

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
       implementation 'com.tuya.smart:tuyasmart-lock-sdk:1.0.0'
    }
    ```

## 专有名词解释

|名词|解释|
|---|---|
|dpCode|设备功能点的标识符。设备中的每个功能点都有名称和编号，本文档针对Wi-Fi门锁设备，可参考[Wi-Fi 门锁功能点列表](#Wi-Fi-门锁功能点列表)|
|劫持|门锁劫持是指将录入特定的密码（指纹、密码等），设置为劫持密码。<br>当用户输入此密码开门时，门锁认为用户非自愿开门，会将告警信息发送至家人手机或物业管理系统。|
|门锁成员|门锁成员分为家庭成员与非家庭成员。<br>家庭成员即添加到用户家庭中的成员，门锁内可对管理家庭成员，设置开锁方式。<br>非家庭成员为在门锁内创建的成员，可通过门锁相关接口进行管理。|

## API 列表

下方的门锁方法均封装在 TuyaLockDevice 类中, 通过传入设备 id 创建 TuyaLockDevice 对象后使用：

```java
TuyaLockDevice tuyaLockDevice = new TuyaLockDevice(deviceId);
```

### 门锁成员
门锁内可以分为家庭成员和非家庭成员，家庭成员为全屋智能中的概念，具体可以查阅家庭成员管理

#### 获取门锁成员列表

**接口说明**

```java
public void getUsers(final ITuyaResultCallback<List<LockUserBean>> callback)
```

**参数说明**

**`LockUserBean` 字段说明**

|字段|类型|描述|
|---|---|---|
|userId|String|成员 id|
|userName| String |用户昵称|
|avatarUrl| String |头像地址|
|contact| String |联系方式|
|unlockRelations|List<UnlockRelationBean>|开锁方式及密码编号|
|devId| String |门锁设备 id|
|ownerId| String |所属家庭 id|
|userType|int|门锁成员类型，1: 家庭成员 2: 非家庭成员|


**示例代码**

```java
tuyaLockDevice.getUsers(new ITuyaResultCallback<List<LockUser>>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock users failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(List<LockUser> lockUserBean) {
        Log.i(TAG, "get lock users success: lockUserBean = " + lockUserBean);
    }
});
```

#### 创建门锁成员
**接口说明**

使用 SDK 创建非家庭成员。以供后续开锁记录关联操作

```java
public void addUser(final String userName, File avatarFile, final List<UnlockRelation> unlockRelations, final ITuyaResultCallback<String> callback)
```

**参数说明**

|参数|能否为空|说明|
|----|----|----|
|userName|否|成员名称|
| avatarFile |是|图片文件，不传则为默认头像|
| unlockRelationBeans |否|成员解锁方式与密码编号的关联关系|

**`UnlockRelationBean` 数据 bean 字段说明**

|字段|类型|描述|
|----|----|----|
|TuyaUnlockType|枚举|解锁方式|
|passwordNumber|int|密码编号, 范围 0 - 999|


**示例代码**

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

**提示**

`UnlockRelation`可以从[获取门锁记录](#获取门锁记录)一节中获取。

当门锁使用密码或其他解锁方式后，可以获取解锁记录，然后可以将解锁记录中的解锁方式分配给某个用户。


#### 更新门锁成员信息
**接口说明**

使用 SDK 更新门锁成员信息，包括用户名、头像、解锁密码对应关系等

```java
public void updateUser(final String userId, final String userName, File avatarFile, final List<UnlockRelation> unlockRelations, final ITuyaResultCallback<Boolean> callback)
```

**参数说明**

|参数|能否为空|说明|
|----|----|----|
|userId|否|成员用户 id，必填|
|userName|是|成员用户名称，可选，不传则不修改|
|avatarFile|是|成员用户头像，可选，不传则不修改|
|unlockRelations|否|成员解锁方式与密码序列号 的关联关系，必填，不修改则填原本的值|

**示例代码**

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

#### 删除门锁成员

**接口说明**

使用 SDK 删除门锁成员信息，删除成员并不会删除已有的密码

```java
public void deleteUser(String userId, final ITuyaResultCallback<Boolean> callback)
```

**参数说明**

|参数|说明|
|---|---|
|userId|门锁成员用户 id|

**示例代码**

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

### 临时密码

使用 SDK 创建临时密码并在门锁上进行输入后即可开锁

#### 创建临时密码
**接口说明**

临时密码可以自定义密码的有效期间，当创建完成后，需要在门锁设备上进行同步

```java
public void createTempPassword(TempPassword tempPassword, final ITuyaResultCallback<Boolean> callback)
```

**参数说明**

`TempPassword`通过TempPassword.Builder创建，具体参考下面的代码示例。

下面为该类的字段说明：

|参数|能否为空|说明|
|---|---|---|
|name|否|密码名称|
|password|否|临时密码，纯数字，7 位|
|effectiveDate|否|密码生效时间戳，单位 ms|
|invalidDate|否|密码失效时间，单位 ms|
|countryCode|是|国家码，例如 86|
|phone|是|手机号码，当创建成功时，会通知给该手机用户|

手机号和国家码可以不传，如果传了需要购买短信服务才会生效。

**示例代码**

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

#### 获取临时密码列表

**接口说明**

使用 SDK 获取临时密码列表，可以查看临时密码的使用状态情况

	public void getTempPasswords(final ITuyaResultCallback<List<TempPassword>> callback)

**参数说明**


**`TuyaSmartLockTempPwdModel` 数据模型**

|字段|类型|描述|
|---|---|---|
|phone| String |手机号|
|name| String |临时密码名称|
|countryCode| String |国家码|
|invalidTime|long|失效时间戳，单位 ms|
|effectiveTime| long |生效时间戳，单位 ms|
|createTime| long |创建时间戳，单位 ms|
|id|int|密码唯一 id|
|sequenceNumber| int |密码编号，关联账号使用|
|status|int|密码状态|

其中，密码状态包含下面几种类型：

```java
int REMOVED = 0; // 已删除
int INVALID = 1; // 失效
int TO_BE_PUBILSH = 2; // 待下发
int WORKING = 3; // 使用中
int TO_BE_DELETED = 4; // 待删除
int EXPIRED = 5; // 已过期
```

**示例代码**

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

#### 删除临时密码

**接口说明**

使用 SDK 删除临时密码，删除后需要门锁设备进行更新

```java
public void deleteTempPassword(int passwordId, final ITuyaResultCallback<Boolean> callback)
```

**参数说明**

|参数|说明|
|---|---|
|passwordId|门锁临时密码唯一 id|

**示例代码**

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

### 动态密码

使用 SDK 获取动态密码并在门锁上进行输入后即可开锁，动态密码有效时间为 5 分钟

#### 获取动态密码
**接口说明**

```java
public void getDynamicPassword(final ITuyaResultCallback<String> callback)
```

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


### 门锁记录

使用 SDK 获取门锁记录，包括开锁记录、门铃记录、报警记录等

#### 获取门锁记录

获取门锁记录接口可根据传入的功能点获取对应的开锁记录。

**接口说明**

```java
public void getRecords(ArrayList<String> dpCodes, int offset, int limit, final ITuyaResultCallback<RecordBean> callback)
```

**参数说明**

|参数|说明|
|---|---|
|dpCodes|需要查询劫持的记录的解锁方式 dp code，具体可以参考[Wi-Fi 门锁功能点列表](#Wi-Fi-门锁功能点列表)|
|offset|页数|
|limit|条数|

**callback返回的参数说明**

`RecordBean`字段说明

|字段|类型|描述|
|---|---|---|
|totalCount|int|总条目|
|hasNext|boolean|是否有下一页|
|datas|List<DataBean>|记录数据内容|

其中`DataBean`字段说明如下：

|字段|类型|描述|
|---|---|---|
|userId|String|成员 id|
|avatarUrl|String|头像 url|
|userName| String |用户昵称|
|createTime|long|该条记录的时间戳，单位 ms|
|devId| String |设备 id|
|dpCodesMap|HashMap<String, Object>|该记录的 dpCode和value 数据|
|unlockRelation|UnlockRelation|解锁类型和解锁密码编号的关系实例，如不是开锁记录，可为空|
|tags|int|标位，0 表示其他，1 表示劫持报警|

**提示**

你可以通过查询开锁记录，获得`unlockRelation`，然后分配给创建的用户。

比如你在门锁上创建了一个密码，然后使用了该密码开锁，这样就产生了一条开锁记录。app端查询到这条开锁记录，就可以将密码分配给你想分配的用户。

**代码示例**

获取开门记录：可传入开门相关功能点，即可获取开门记录

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

获取门锁告警记录：传入告警功能点，获取告警记录

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

#### 获取门锁劫持记录

使用 SDK 获取门锁劫持开门记录

**接口说明**

```java
public void getLockHijackRecords(int offset, int limit, final ITuyaResultCallback<RecordBean> callback)
```

**参数说明**

|参数|说明|
|---|---|
|offset|页数|
|limit|条数|

**代码示例**

```java
tuyaLockDevice.getHijackRecords(0, 10, new ITuyaResultCallback<Record>() {
    @Override
    public void onError(String code, String message) {
        Log.e(TAG, "get lock hijack records failed: code = " + code + "  message = " + message);
    }

    @Override
    public void onSuccess(Record hijackingRecordBean) {
        Log.i(TAG, "get lock hijack records success: hijackingRecordBean = " + hijackingRecordBean);
    }
});
```

### 远程开门

在门锁上触发远程开门请求后，使用 SDK 可以进行远程开门

远程开门需要注册远程开门监听，收到门锁的远程开门请求后，调用远程开门接口即可。

#### 注册远程开门监听

**接口说明**

```java
public void setRemoteUnlockListener(RemoteUnlockListener remoteUnlockListener)
```

**参数说明**

RemoteUnlockListener 接口中有一个方法：

```java
void onReceive(String devId, int second);
```

方法参数如下

|字段|类型|描述|
|---|---|---|
|devId |String|设备 id|
|second |int|需要在多少秒内处理|

#### 请求远程开锁

**接口说明**

```java
public void replyRemoteUnlock(boolean allow, final ITuyaResultCallback<Boolean> callback)
```

**参数说明**

|字段|类型|描述|
|---|---|---|
| allow |boolean|是否允许开锁|

#### 远程开锁调用示例

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_lock_device);
	
	....

    // 注册远程开锁监听
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
 * 创建远程开锁确认弹框
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
 * 请求远程开锁接口
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

## Wi-Fi 门锁功能点列表

| dp name              | dp code                        |
| -------------------- | ------------------------------ |
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