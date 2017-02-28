#EliteWebChat Android sdk 使用说明

1. 导入融云相关的sdk模块，这里使用到了IMLib，IMKit，LocationLib，PushLib。并按文档说明配置相关参数。注意IMLib中的AndroidManifest.xml里，需要配置融云相关的APP_KEY
		
		<meta-data
            android:name="RONG_CLOUD_APP_KEY"
            android:value="xxxxxxxxxxx" />

2. 在融云的网站上，登录账户，进入对应的app，配置服务端实时消息路由的路由地址。具体地址，由过河兵相关人员提供。
3. 配置AndroidManifest.xml。主要是配置一个android:name=".App", 相关权限，一个会话界面的activity，一些元数据，相关的provider，receiver，service。具体看demo代码。
4. 创建一个App类，其中在onCreate回调中初始化RongIM，具体看demo示例代码。
	
	    //初始化融云
	    RongIM.init(this);
	    //注册接收消息监听器
	    RongIM.setOnReceiveMessageListener(new EliteReceiveMessageListener());
	    //注册自定义消息
	    RongIM.registerMessageType(EliteMessage.class);
    
5. 在主Activity中，初始化并启动EliteChat

		/**
	     * 初始化EliteChat， 并且启动聊天
	     * @param serverAddr EliteWebChat服务地址
	     * @param userId 用户登录id
	     * @param name 用户名
	     * @param portraitUri 用户头像uri
	     * @param context 当前上下文
	     * @param queueId 排队队列号
	     */
        EliteChat.initAndStart("http://192.168.2.80:8980/webchat/rcs", "000001", "test1", "", MainActivity.this,  1);
	这里的EliteWebChat服务地址，需要找过河兵相关人员提供，用户登录id可以是你们系统中的用户名，不重复即可，这里会自动查询如果不存在与过河兵系统中，会自动创建相关客户。排队队列号也是找过河兵相关人员提供即可。

相关代码说明都可以从demo示例代码中找到


