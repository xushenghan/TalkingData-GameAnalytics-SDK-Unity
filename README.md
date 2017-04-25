## Unity平台集成指南

### 快速集成

#### 导入SDK
　　在 Unity 编译器中选择 Assets --> Import Package --> Custom Package 找到本地目录下的 Game_Analytics_SDK_Unity.unitypackage 文件，点击`Open`按钮即可导入成功。

#### 添加权限和依赖的框架
###### Android
　　Android平台中SDK需要获取适当的权限才可以正常工作，开发者需要在 `AndroidManifest.xml` 文件中添加下列所有权限申明。

* `INTERNET` 允许游戏联网和发送统计数据的权限。
* `ACCESS_NETWORK_STATE` 允许游戏检测网络连接状态，在网络异常状态下避免数据发送，节省流量和电量。
* `READ_PHONE_STATE` 允许游戏以只读的方式访问手机设备的信息, 通过获取的信息来定位唯一的玩家。
* `ACCESS_WIFI_STATE` 用来获取设备的 mac 地址。
* `WRITE_EXTERNAL_STORAGE` 用于保存设备信息，以及记录日志。
* `ACCESS_FINE_LOCATION(可选)` 用来获取该游戏被使用的精确位置信息。
* `ACCESS_COARSE_LOCATION(可选)` 用来获取该游戏被使用的粗略位置信息。

###### iOS
　　iOS平台需要添加适当的依赖框架才可以正常工作，开发者需要在编译生成的 `Xcode` 工程中添加下列所有依赖框架。

* `AdSupport.framework` 获取advertisingIdentifier
* `CFNetwork.framework` 发送统计数据
* `CoreTelephony.framework` 获取运营商标识
* `CoreMotion.framework` 支持摇一摇功能
* `Security.framework` 辅助存储设备标识
* `SystemConfiguration.framework` 检测网络状况
* `libz.tbd` 进行数据压缩

###### WindowsPhone
　　Windows Phone平台中，SDK需要获取适当的权限才可以正常工作，开发者需要在 `WMAppManifest.xml` 文件中添加下列所有权限申明。

* `ID_CAP_IDENTITY_DEVICE` 用于唯一标识一台设备。
* `ID_CAP_IDENTITY_USER` 用于唯一标识一台设备。
* `ID_CAP_LOCATION(可选)` 允许游戏访问位置，用于统计用户的区域来源。
* `ID_CAP_NETWORKING` 允许游戏联网和发送统计数据的权限。

#### 初始化集成
　　在游戏启动后(包含游戏刚开启或从后台恢复到前台)，调用 `TalkingDataGA.OnStart`，该接口完成统计模块的初始化和统计 session 的创建，所以越早调用越好。  
　　在游戏结束时(包含切出游戏和退到后台情况，如点击home、锁屏等按键)，调用 `TalkingDataGA.OnEnd`。  
　　可以选择性的在channelId中填入推广渠道的名称，数据报表中则可单独查询到他们的数据。每台设备仅记录首次安装激活的渠道，更替渠道包安装不会重复计量。不同的渠道ID包请重新编译打包。

### 高级功能
###### 统计玩家帐户
　　定义一个玩家单元，更新玩家最新的属性信息。

```
//设置唯一帐户
public static TDGAAccount SetAccount(string accountId)
//设置帐户类型
public void SetAccountType(AccountType accountType)
//设置帐户的显性名
public void SetAccountName(string accountName)
//设置级别
public void SetLevel(int level)
//设置性别
public void SetGender(Gender gender)
//设置年龄
public void SetAge(int age)
//设置区服
public static void SetGameServer(string gameServer)
```

###### 跟踪玩家充值
　　跟踪玩家充值现金而获得虚拟币的行为，充入的现金将反映至游戏收入中。

```
//充值请求
public static void OnChargeRequest(string orderId, string iapId, double currencyAmount, string currencyType, double virtualCurrencyAmount, string paymentType)
//充值成功
public static void OnChargeSuccess (string orderId)
```

###### 跟踪获赠的虚拟币
　　跟踪全部免费赠予虚拟币的数据。

```
//赠予虚拟币
public static void OnReward (double virtualCurrencyAmount, string reason)
```

###### 跟踪游戏消费点
　　跟踪游戏中全部使用到虚拟币的消费点

```
//记录付费点
public static void OnPurchase(string item, int itemNumber, double priceInVirtualCurrency)
//消耗物品或服务等
public static void OnUse(string item, int itemNumber)
```

###### 任务、关卡或副本
　　跟踪玩家任务/关卡/副本的情况。

```
//接受或进入
public static void OnBegin(string missionId)
//完成
public static void OnCompleted(string missionId)
//失败
public static void OnFailed(string missionId, string cause)
```

###### 自定义事件
　　用于统计任何您期望去跟踪的数据

```
public static void OnEvent(string eventId, final Dictionary<string, object="">)
```

### 推送营销
　　营销推送组件帮助您获得利用数据进行精准推送的能力，结合数据平台提供的各种人群，可以实时编辑发送内容，对任意人群完成推送，并支持实时查阅推送效果数据，不断对比效果，优化营销方法。  
　　营销推送更强大之处在于，您不仅可以选择使用TalkingData提供的推送通道，他还可以与个推、极光等推送平台组合使用，让以往的粗放推送都可达到实时精准化，并实时查阅效果数据。

#### Android
##### 集成TalkingData推送
　　增加以下的调用方法，来让您的应用可以接收推送通知；需注意在获得推送能力的同时，您的应用会在后台中长期运行。

###### 修改AndroidManifest.xml文件
1.添加推送功能所必要的权限。

```
<!-- 允许App开机启动，来接收推送 -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"></uses-permission>
<!--发送持久广播 -->
<uses-permission android:name="android.permission.BROADCAST_STICKY"></uses-permission>
<!-- 修改全局系统设置-->
<uses-permission android:name="android.permission.WRITE_SETTINGS"></uses-permission>
<!-- 允许振动，在接收推送时提示客户 -->
<uses-permission android:name="android.permission.VIBRATE"></uses-permission>
<!-- 侦测Wifi 变化，以针对不同 Wifi 控制最佳心跳频率，确保push的通道稳定 -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"></uses-permission>
<!-- 此权限用于在接到推送时，可唤醒屏幕，可选择性添加权限 -->
<uses-permission android:name="android.permission.WAKE_LOCK"></uses-permission>
```

2.添加TalkingDataGameAnalytics所必须的Service。

```
<service android:name="com.gametalkingdata.push.service.PushService" android:process=":push" android:exported="true"></service>
```

3.添加TalkingDataGameAnalytics所必须的BroadCastReceiver，以支持接收消息。

```
<receiver android:name="com.gametalkingdata.push.service.PushServiceReceiver" android:exported="true">
	<intent-filter>
		<action android:name="android.intent.action.CMD"></action>
		<action android:name="android.talkingdata.action.notification.SHOW"></action>
		<action android:name="android.talkingdata.action.media.MESSAGE"></action>
		<action android:name="android.intent.action.BOOT_COMPLETED"></action>
		<action android:name="android.net.conn.CONNECTIVITY_CHANGE"></action>
		<action android:name="android.intent.action.USER_PRESENT"></action>
	</intent-filter>
</receiver>
<receiver android:name="com.tendcloud.tenddata.TalkingDataMessageReceiver" android:enabled="true">
	<intent-filter>
		<action android:name="android.talkingdata.action.media.SILENT"></action>
		<action android:name="android.talkingdata.action.media.TD.TOKEN"></action>
	</intent-filter>
	<intent-filter>
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
</receiver>
```

##### 第三方推送平台与TalkingData组合使用
　　TalkingData支持从平台中划定精准用户群，组合第三方推送平台来直接发送推送和收集实时数据效果；目前支持两家第三方推送平台：个推、极光。如果您已经是三方推送的使用者，这种方式能让您更舒服的实现精准推送并验证效果。

###### 集成第三方推送平台
　　您需要在第三方推送平台中申请账户，并已完成三方推送的对接。 您需要在对接后确认测试通过，三方推送可以接收到推送消息。

###### 添加TalkingData Game Analytics所必须的BroadCastReceiver
1.个推推送

```
<receiver android:name="com.tendcloud.tenddata.TalkingDataMessageReceiver" android:enabled="true">
	<intent-filter>
		<!-- 必须添加 -->
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
	<intent-filter>
		<!--注：H0DPYSxUkR9NFoWnvff656要换成开发者自己的appid-->
		<action android:name="com.igexin.sdk.action.H0DPYSxUkR9NFoWnvff656"></action>
	</intent-filter>
</receiver>
```

2.极光推送

```
<receiver android:name="com.tendcloud.tenddata.TalkingDataMessageReceiver" android:enabled="true">
	<intent-filter>
		<!-- 必须添加 -->
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
	<intent-filter>
		<!-- 如果使用极光推送，必须添加 -->
		<action android:name="cn.jpush.android.intent.REGISTRATION"></action>
		<action android:name="cn.jpush.android.intent.MESSAGE_RECEIVED"></action>
		<category android:name="您的应用包名"></category>
	</intent-filter>
</receiver>
```

###### 配置推送key
　　完成以上集成后，您还需要在TalkingData平台中配置推送相关的Key；请进入平台“推送营销”-“推送配置”在android平台配置中添加好这些配置。

#### iOS
##### 在Start方法中注册通知服务
```
void Start () {
	TalkingDataGA.OnStart("your_app_id", "your_channel_id");
#if UNITY_IPHONE
#if UNITY_5
	UnityEngine.iOS.NotificationServices.RegisterForNotifications(
		UnityEngine.iOS.NotificationType.Alert |
		UnityEngine.iOS.NotificationType.Badge |
		UnityEngine.iOS.NotificationType.Sound);
#else
	NotificationServices.RegisterForRemoteNotificationTypes(
		RemoteNotificationType.Alert |
		RemoteNotificationType.Badge |
		RemoteNotificationType.Sound);
#endif
#endif
	// other code
}
```

##### 在Update方法中调用TalkingDataGA的方法
```
void Update () {
#if UNITY_IPHONE
	TalkingDataGA.SetDeviceToken();
	TalkingDataGA.HandleTDGAPushMessage();
#endif
	// other code
}
```
