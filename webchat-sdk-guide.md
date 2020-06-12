# WebChat App SDK 对接说明 #

### 在第三方的app中，可以与我们WebChat服务进行对接，其中对接方式主要有2种：
> 1. 直接嵌入mobile的h5页面，这种方式集成方便，几乎不需要什么开发量，缺点也很明显，由于是基于h5，界面友好度不够，且只有在页面打开时候可以查看到消息，当页面关闭后，消息是没法收到的。
> 2. 使用我们提供的App SDK，做深度的集成。这种方式会对第三方app有一定的开发要求，针对ios和android，还需要有不同的开发集成。而这种方式的好处在于，用户体验很高，可以发送语音视频等多媒体消息，即使app没有打开，也可以收到消息等。

这里主要讲一下App SDK的集成方式


##基于融云的app sdk 集成

我们的移动App SDK，集成了融云的移动通讯能力。借助融云的App消息上下行的能力，实现了坐席PC与客户手机app聊天。
具体说明可以查看融云官网：[http://www.rongcloud.cn/](http://www.rongcloud.cn/ "http://www.rongcloud.cn/")

看这篇文档时候，假设你已经是个android开发者（或者ios），并且对融云已经有大致了解。在此基础上，我们来对具体集成步骤，加以说明：

###一. 注册融云账号

1. 在融云官网注册一个自己的账户，并创建一个app。[https://developer.rongcloud.cn/app](https://developer.rongcloud.cn/app "https://developer.rongcloud.cn/app")

2. 配置服务端实时消息路由的路由地址。具体地址，由过河兵相关人员提供。
![设置服务端消息路由地址](images/webchat-sdk-guide/2.png)

###二. APP 集成

这里以Android系统为例：

1. 导入融云相关的sdk模块，这里使用到了IMLib，IMKit，LocationLib，PushLib。并按文档说明配置相关参数。注意IMLib中的AndroidManifest.xml里，需要配置融云相关的APP_KEY，具体可以查看[http://www.rongcloud.cn/docs/android.html](http://www.rongcloud.cn/docs/android.html "http://www.rongcloud.cn/docs/android.html")
```xml
	<meta-data
		android:name="RONG_CLOUD_APP_KEY"
		android:value="xxxxxxxxxxx" />
```	

2. 配置AndroidManifest.xml。主要是配置一个android:name=".App", 相关权限，一个会话界面的activity，一些元数据，相关的provider，receiver，service。具体看demo代码。

3. 创建一个App类，其中在onCreate回调中初始化RongIM，具体看demo示例代码。
```java
	//初始化chat
    Chat.init(this);
    // 集成推送，具体可以看融云文档 https://www.rongcloud.cn/docs/android_push.html
    PushConfig config = new PushConfig.Builder()
            .enableHWPush(true)
            .enableMiPush("小米 appId", "小米 appKey")
            .enableMeiZuPush("魅族 appId", "魅族 appKey")
            .enableFCM(true)
            .build();
    RongPushClient.setPushConfig(config);
	//初始化融云
    RongIM.init(this);
    //注册接收消息监听器
    RongIM.setOnReceiveMessageListener(new EliteReceiveMessageListener());
    //注册自定义消息
    RongIM.registerMessageType(EliteMessage.class);
    //注册机器人消息
    RongIM.registerMessageType(RobotMessage.class);
    RongIM.registerMessageTemplate(new RobotMessageItemProvider());
    //注册卡片消息
    RongIM.registerMessageType(CardMessage.class);
    RongIM.registerMessageTemplate(new CardMessageItemProvider());
    //注册自定义用户信息提供者
    RongIM.setUserInfoProvider(new EliteUserInfoProvider(), true);
```	

4. 导入过河兵SDK相关类，这里以可以以jar包方式提供，也可以直接提供源码。导入com.elitecrm.rcclient包下的所有类，其中App.java和MainActivity.java不需要，使用自己app中的即可。
![导入过河兵SDK相关类](images/webchat-sdk-guide/3.png)

5. 在主Activity中，初始化并启动EliteChat
```java
	/**
     *      ** 聊天的入口 **
     *
     * @param serverAddr WebChat服务地址
     * @param userId 用户登录id
     * @param name 用户名
     * @param portraitUri 用户头像uri
     * @param targetId 窗口id
     * @param context 当前上下文
     * @param queueId 排队队列号
     * @param ngsAddr ngs服务地址
     * @param from 请求来源
     * @param tracks 轨迹信息（可选）
     * @param title 聊天窗口标题 （可选）
     */
    public static void startChat(String serverAddr, String userId, String name, String portraitUri, String targetId, Context context, int queueId, String ngsAddr, String from, String tracks, String title)
```
这里的EliteWebChat服务地址，需要找过河兵相关人员提供，用户登录id可以是你们系统中的用户名，不重复即可，这里会自动查询如果不存在与过河兵系统中，会自动创建相关客户。排队队列号也是找过河兵相关人员提供即可。

###三. SDK中更多高级方法说明
经过上述的步骤，App SDK已经可以对接起来了。客户可以发出聊天请求，排队聊天，如果有坐席接起这个聊天请求，客户就可以和坐席正常聊天了。下面再说一些高级功能：

1. 在排队之前，就发出一些预发消息
```java
	// 发送文字消息，在调用EliteChat.initAndStart之前，就可以调用此方法，之后一旦聊天建立起来后，这个预发消息会自动发出。
	MessageUtils.sendTextMessage("firstMsg", target);
	// 发送自定义消息，这个消息内容随便自己定义，坐席端可以收到相关消息自行做对应处理。比如这里发送一个商品信息的json字符串。坐席端可以收到后显示出对应的商品信息。
	MessageUtils.sendCustomMessage("{\"name\":\"xxx\"}", target);
	// 发送图片消息
	String imgPath = Environment.getExternalStorageDirectory().getAbsolutePath() + "/DCIM/chat.png";
    MessageUtils.sendImgMessage(Uri.fromFile(new File(imgPath)), Uri.fromFile(new File(imgPath)), target);
```

2. 如果客户已经进入过聊天，返回到app其他页面后，再次想打开聊天，这时候可以直接启动页面，而不需要再次发起排队了
```java
	//先判断，当前会话是否还在活动中，如果活动中则可以不发起排队，直接显示页面
	if (Chat.getInstance().isSessionAvailable()) {
        if(Chat.getInstance().isTokenValid()) {
			//显示出聊天界面
            RongIM.getInstance().startConversation(v.getContext(), Conversation.ConversationType.PRIVATE, Chat.getInstance().getClient().getTargetId(), "在线客服");
            return;
        }
    }
	//如果会话已经结束了，或者token已经失效，则重新发起聊天请求
	EliteChat.initAndStart(getResources().getString(R.string.app_server_addr), userId, name, portraitUri, target, v.getContext(), queueId, null);
```

3. App没有开启或者不在前台时候，手机收到消息会有提示，如果点击了提示，可以调方法直接打开聊天界面。
```java
	/**
	 * EliteChat类方法
     * 当收到提醒时候，onNotificationMessageClicked 方法触发，然后可以通过调用此方法来打开chat
     * @param context
     */
    public static void switchToChat(Context context, String target)
```

4. 如果需要使用地图发送地图消息
SDK支持高德地图和百度地图两种选择，这里以百度地图为例：
```java
	//1. 在AndroidManifest.xml中配置百度地图API_KEY
	<meta-data
            android:name="com.baidu.lbsapi.API_KEY"
            android:value="xxxxx" />

	//2. 在onCreate方法里初始化百度地图
	SDKInitializer.initialize(getApplicationContext());

	//3. 在onCreate方法里，注册自定义扩展模块，先去除默认扩展，再注册自定义扩展
	//注册自定义扩展模块，先去除默认扩展，再注册自定义扩展
    List<IExtensionModule> extensionModules = RongExtensionManager.getInstance().getExtensionModules();
    if (extensionModules != null) {
        for (IExtensionModule extensionModule : extensionModules) {
            RongExtensionManager.getInstance().unregisterExtensionModule(extensionModule);
        }
    }
    EliteExtensionModule extensionModule = EliteExtensionModule.getInstance()
            .enableImage(true) // 加号中开启发送图片
            .enableMap("baidu") // 加号中开启百度地图位置发送
            .enableFile(true) // 加号中开启发送文件
            .enableSight(true)// 加号按钮中，开启小视频功能（注意小视频消息是需要额外收费的（融云收费功能））
            .enableCloseSession(true);// 加号按钮中，开启客户主动结束聊天功能
    RongExtensionManager.getInstance().registerExtensionModule(extensionModule);
```


**ios同理，android中提到的方法，ios中都有相同的对应的方法**

###**完整的相关代码说明都可以从demo示例代码中找到**

ios的GitHub代码仓库地址 ：[https://github.com/jinguoxi/RyWebChat](https://github.com/jinguoxi/RyWebChat "https://github.com/jinguoxi/RyWebChat")
android的GitHub代码仓库地址 ：[https://github.com/loriling/RCClient](https://github.com/loriling/RCClient "https://github.com/loriling/RCClient")



