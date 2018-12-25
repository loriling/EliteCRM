# WebChat第三方客户端接入说明 #
10/24/2016 10:30:30 AM 
### EliteWebChat提供整套从客服端到客户端的界面与展示解决方案，同时也可以支持与第三方客户端的对接。其中成功案例就有与小米的米聊对接，与livebot的微信服务对接等。 ###

<div style="display: block;position: fixed;left: 0px;top: 10px;padding-left: 20px;max-width: 200px;">
目录

一. <a href="#">接口说明</a>
<div style="padding-left: 20px">**客户发出请求**
1. <a href="#login">客户发出登录</a>
2. <a href="#loginOrRegister">客户发出登录或者注册</a>
3. <a href="#visitorLogin">访客进入</a>
4. <a href="#chatRequest">客户发出聊天请求</a>
5. <a href="#checkRequestStatus">查看聊天请求当前状态</a>
6. <a href="#sendMessageWhileQueuing">排队期间发送消息</a>
7. <a href="#cancelRequest">取消排队请求</a>
8. <a href="#sendTextMsg">发送文本消息</a>
9. <a href="#sendImgMsg">发送图片消息</a>
10. <a href="#sendVoiceMsg">发送语音消息</a>
11. <a href="#sendLocationMsg">发送位置消息</a>
12. <a href="#sendFileMsg">发送文件消息</a>
13. <a href="#uploadFile">上传文件</a>
14. <a href="#previewMessage">发送预览消息</a>
15. <a href="#typingNotice">发送正在输入通知</a>
16. <a href="#robotTransfer">发送转人工请求</a>
17. <a href="#closeSession">发送结束聊天会话</a>
18. <a href="#rating">发送满意度评价</a>
19. <a href="#changeCustomer">发送客户改变消息</a>
20. <a href="#queryHistoryMessages">客户查询聊天历史</a>

**坐席发送请求**
1. <a href="#agentSendStartSession">坐席发送开始聊天通知</a>
2. <a href="#agentSendMsg">坐席发送消息</a>
3. <a href="#agentSendTypingNotice">坐席发送正在输入通知</a>
4. <a href="#agentSendAgentsUpdate">坐席发送会话中坐席更新</a>
5. <a href="#agentSendUpdateRequestStatus">坐席发送请求状态更新</a>
6. <a href="#agentSendPushRating">坐席推送满意度评价</a>
7. <a href="#agentSendCloseSession">坐席发起结束聊天通知</a>
</div>
二. <a href="#requestAndResponseCode">请求代码与相应结果代码</a>
<div style="display: inline-block;padding-left: 20px">1. <a href="#requestCode">请求代码</a>
2. <a href="#responseCode">响应代码</a>
</div>

三. <a href="#requestResultCode">聊天请求状态码</a>
四. <a href="#emoji">消息体中表情的定义</a>

</div>

## 一. http对接接口说明 ##
接口才用http协议，包括了webchat端提供的接口与第三方提供的接口，数据传输使用json格式。
下面从模拟整个聊天过程的方式来介绍这些接口：

- <div id="login">客户发出登录请求： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 1,//登录请求
	loginName: "lori",//客户登录名
	password: "xxxxx",//客户密码
	ip: "192.168.0.1"//客户端ip（可选）
}
返回参数：
{
	result: 1, //登录成功（1）或登录验证失败（-20）
	clientId: "xxxxxxxxxx", //用户的唯一标示，与后面开始会话接口中的clientId对应
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B" //登录成功后返回的token，用来之后请求时候传递
}
```

- <div id="loginOrRegister">客户发出登录或者注册请求，如果不需要登录验证，并且希望新客户自动创建的话，就调用这个接口： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 3,//登录或注册
	loginName: "lori",//客户登录名
	name: "罗瑞",//客户姓名（昵称）
	portraitUri: "http://139.196.108.236/ngs/fs/get?file=headimg/979DEECA-80C4-F7D5-D9F2-A8D4C071BEA6.png",//头像地址
	ip: "192.168.0.1"//客户端ip（可选）
}
返回参数：
{
	result: 1, //登录成功
	clientId: "xxxxxxxxxx", //用户的唯一标示，与后面开始会话接口中的clientId对应
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B" //登录成功后返回的token，用来之后请求时候传递
}
```

- <div id="visitorLogin">匿名访客想要聊天，可以发出访客进入的接口： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 4,//访客登录
	visitorId: "xxxxxxxxxxxx",//访客的唯一标示ID，通常是个36位guid
	ip: "",//访客请求过来的ip地址（可选）
}
返回参数：
{
	result: 1, //登录成功
	clientId: "xxxxxxxxxx", //用户的唯一标示，与后面开始会话接口中的clientId对应
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B" //登录成功后返回的token，用来之后请求时候传递
}
```

- <div id="chatRequest">客户从网页或者微信端发出聊天请求： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 1000,//发出聊天的请求
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
	time: 1400913140127,//发出请求的时间戳
	queueId: 1,//请求的队列号
	from: "APP",//请求来源，分为:PC MOBILE WECHAT APP 不同来源会让坐席端看到不同客户的默认头像
	brand: "",//请求来源2，作为第二个来源字段（可选）
	serverAddr: "http://www.xxxx.com", //请求方服务地址（可选）如果传递，则后面的回调都会按这个地址来
	tracks: "" //（可选）客户浏览轨迹 json字符串，具体格式查看相关文档
}
返回参数：
{
	result: 1, //请求发送成功（1）或失败（0）
	message: "success", //成功或失败的消息
	requestId: 103, //请求id号，如果发起请求成功，就会返回这个id
	queueLength: 3, //队列长度
	continueLastSession: false,//是不是继续上一次的会话，（可选）如果有且为true，表示这个请求是继续之前一个会话的，可能是之前那个会话没有正确的关闭掉造成的
	sessionId: 10023,//如果是机器人队列，不需要经过排队过程，直接创建出会话并返回出会话id，拿到这个会话id后即可直接聊天
	agentId: "BOT002",//如果是机器人队列，机器人坐席id
	robotType: 2,//如果是机器人队列，机器人坐席类型
	agentName: "小i机器人"//如果是机器人队列，机器人坐席的名字
}
```

- <div id="checkRequestStatus">更新聊天排队情况： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 1001,//查看聊天请求当前状态的请求
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
	requestId: 103//需要查看状态的请求id（发起聊天请求成功后会返回这个id）
}
返回参数：
{
	result: 1,//请求发送成功（1）或失败（0）
	message: "success",//成功或失败的消息
	requestStatus: 1,//请求状态，具体状态码看下方说明（三. 聊天请求状态码）
	queueLength: 2//当前排队长度
}
```
	
- <div id="sendMessageWhileQueuing">客户在排队中，就想发起消息，如果排队排上，之前的消息就会一起一次性发送给对应坐席： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 2020,//发送预消息
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
	requestId: 103,//请求id（发起聊天请求成功后会返回这个id）
	text: 'xxxxxx'
}
返回参数：
{
	result: 1,//请求发送成功（1）或失败（0）
	message: "success"//成功或失败的消息
}
```

- <div id="cancelRequest">客户排队时间过长，想取消排队： </div>
```
http://xxxxx/EliteWebChat/tpi
发送参数：
{
	type: 1002,//取消聊天排队请求
	token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
	requestId: 103//取消取消的请求id（发起聊天请求成功后会返回这个id）
}
返回参数：
{
	result: 1,//请求发送成功（1）或失败（0）
	message: "success"//成功或失败的消息
}
```

- <div id="agentSendStartSession" style="color:#0086b3">webchat端路由请求到某个客服后，创建出聊天会话，通知第三方可以开始聊天了</div>
```
http://xxxxx/ThirdPartService/create
发送参数：
{
    result：1,
    message:"success",
    agentId:"SELITE",
    agentName:"Elite", //坐席昵称(webchat_alias)
    agent: {
        id: "SELITE",
        name: "Elite", //坐席昵称(webchat_alias)
        alias: "坐席A", //坐席别名(staffname)
        icon: "http://xxxx/xxx/xx.png", //坐席头像url
        commonts: "备注信息" //备注信息
    },
    clientId:"xxxxxxxxx",
    sessionId: 1299,
	continueLastSession: false,//是否继续上一个会话(如果客户端异常断开后短时间内再次发出聊天请求，可能会继续上一次聊天会话)
    time: 1400913830109
}
返回参数:
{
    result:1,
    message:"success"
}
```

- <div id="sendTextMsg">客户发送文本消息给客服</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2001,//发送文本消息请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109,
    text: "你好，请问xxxxx",
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="sendImgMsg">客户发送图片消息给客服</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2002,//发送图片消息请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109,
    imageUri: "http://xxxxxx/xxx.png", //具体图片文件的url
    thumbData: "iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="//缩略图的base64编码
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1, // 也可能是-22 表示图片存储失败
    message: ""
}
```

- <div id="sendVoiceMsg">客户发送语音消息给客服</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2003,//发送语音消息请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109,
    voiceLength: 33, //语音长度，单位是秒
    voiceData: "iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="//语音文件的base64编码，语音文件只支持接收amr格式的文件
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1, // 也可能是-23 表示语音存储失败
    message: ""
}
```

- <div id="sendLocationMsg">客户发送地图消息给客服</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2004,//发送地图消息请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109,
    latitude: 34.12, //纬度值
    longitude: 131.24, //经度值
    poi: "上海市中山西路2025号", //地址描述（可选）
    map: "baidu", //地图类型，默认是高德地图，也可以传递baidu，表示百度地图（可选）
    thumbData: "iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="//地图的缩率图的base64编码（可选）
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="sendFileMsg">客户发送文件消息给客服</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2005,//发送地图消息请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109,
    name: "xxx.docx", //文件名
    fileUrl: "http://xxxxx/xxx.docx" //文件url地址
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="uploadFile">客户上传文件给chat服务: http://xxxxx/EliteWebChat/tpiu</div>
这个接口用来给客户上传图片，附件等相关信息，配合了发送图片接口和发送附件接口，通常先调用此接口上传需要发的文件，然后获取到上传成功后文件的url地址，再调用发送相关的消息接口，把对应文件url传递过来
```
//用java代码为例，主要需要两部分参数，一部分就是文件本身，一部分就是之前登录成功后的token值
//这里只是一个示例，具体开发过程中可能是其他语言，按规则传递参数就好了
HttpClient httpclient = getHttpClient();
httpclient.getParams().setParameter(CoreProtocolPNames.PROTOCOL_VERSION, HttpVersion.HTTP_1_1);
HttpPost httppost = new HttpPost(serverUrl);
File file = new File(fileToUpload);
MultipartEntity mpEntity = new MultipartEntity();
ContentBody cbFile = new FileBody(file);
mpEntity.addPart("fileToUpload", cbFile); // 对应的文件
mpEntity.addPart("token", new StringBody(token)); // 获取到的token
httppost.setEntity(mpEntity);
HttpResponse response = httpclient.execute(httppost);
```
而这个请求的response：
```
返回参数：
{
    result: 1, // 1表示上传成功，其他失败码看附录，当失败的时候会有message属性来描述失败原因
    fileName: "xxx.docx", //文件名
    url: "/xxx/xxx.docx" //上传成功后会返回文件相对路径,之后发送附件消息或者图片消息时候，需要拿着这个返回的url，拼接上前面的webchat服务地址，再传递过来
}
```	

- <div id="previewMessage">客户输入一些内容，但是这时候还没有按发送的时候，可以发送预览消息出来，让坐席预先知道客户可能会发送什么信息（此类消息可以在客户输入内容改变时候，发送出来当前已经输入的内容，但需要注意发送频率，比如可以加个时间5秒内发送一次）</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2021,//发送预览消息
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    text: "xxxxx",//客户输入到一半的内容
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="typingNotice">客户发出正在输入提示（此类消息建议在用户焦点进入输入框时候发送，需要注意发送频率，比如可以加个时间5秒内发送一次）</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2022,//正在输入提示
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    extra: ""//附加信息（可选）
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="robotTransfer">客户转接人工（如果是机器人会话中，客户可以发起转接人工的请求）</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 2023,//正在输入提示
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
	from: ""//转人工请求来源(可选)
    transferInfo: ""//转人工相关信息(可选)，如果转人工时候需要指定队列，就需要传递这个参数，不然转的就是开始时候排队的队列，具体格式是个json字符串，比如：{"queueId":2}
}
返回参数：
{
    result: 1,
    message: ""
}
```

- <div id="agentSendMsg" style="color:#0086b3">客服发送消息给客户</div>
```
http://xxxxx/ThirdPartService/msg
{
    sessionId: 1299,
    agentId: "SELITE",
    agentName: "Elite", //坐席昵称
    agentHeadimg: "http://xxxx/xxx/xx.png", //坐席头像
    agentComment: "备注信息", //坐席备注
    time: 1400913830109,
    content: "xxxxxxxxxx，xxxxx",
    clientId: "xxxxxxxxxx",//（备用，可能不传）
    type: "text"//消息类型，文本中可能是一段html片段，会有a标签和img标签，需要自行解析（type只有text和notice，notice表示系统通知类）
}
返回参数：
{
    result: 1,
    message: "success"
}
```

- <div id="agentSendTypingNotice" style="color:#0086b3">客服发送正在输入提示给客户</div>
```
http://xxxxx/ThirdPartService/msg
发送参数:
{
    sessionId: 1299,
    content: "",
    clientId: "xxxxxxxxxx",//（备用，可能不传）
    type: "notice",//通知类消息
	noticeType: 5//通知消息类型，5表示正在输入的消息
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="agentSendAgentsUpdate" style="color:#0086b3">会话发生转接或者会议后，坐席信息更新</div>
```
http://xxxxx/ThirdPartService/agentsUpdate
发送参数:
{
	clientId: "xxxxxxxxxx",
	sessionId: 1299,//会话id
	agents: [//当前会话中所有坐席信息，如果是会议就存在多个坐席
		{
			id: 'SELITE',
			name: 'Elite',
			icon: 'http://xxxxx/xxxx.png',
			comments: '备注信息'
		},
		{
			id: 'AE1EA',
			name: 'Lori',
			icon: 'http://xxxxx/xxxx.png',
			comments: '备注信息'
		}
	]
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="agentSendUpdateRequestStatus" style="color:#0086b3">排队后，请求状态改变的通知</div>
```
http://xxxxx/ThirdPartService/updateRequestStatus
发送参数:
{
	clientId: "xxxxxxxxxx",
	requestId: 103,//请求id
	requestStatus: 1,//请求状态 0等待 1接受 2拒绝 3超时 具体可以下面的请求状态码
	queueLength: 0// 当前队列长度
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="agentSendPushRating" style="color:#0086b3">客服推送满意度给客户</div>
```
http://xxxxx/ThirdPartService/pushRating
发送参数:
{
    sessionId: 1299,
    agentId: 'SELITE',
    clientId: 'xxxxxxxxxx'//（备用，可能不传）
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="closeSession">客户结束聊天</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 4001,
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    time: 1400913830109
}
返回参数：
{
    result:1,
    message:""
}
```

- <div id="agentSendCloseSession" style="color:#0086b3">客服关闭聊天会话</div>
```
http://xxxxx/ThirdPartService/close
发送参数:
{
    sessionId: 1299,
    agentId: 'SELITE',
    clientId: 'xxxxxxxxxx',//（备用，可能不传）
    time: 1400913830109
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="rating">客户发出满意度评价请求</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 5001,//满意度评价请求
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,
    ratingId: 1,//具体id值对应的含义需要询问具体项目人员
    ratingComments: "服务非常满意",
    ratingExtends: {}//扩展信息，具体按相关项目自己定义传递
}
返回参数:
{
    result: 1,
    message: "success"
}
```

- <div id="changeCustomer">客户改变请求</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 5002,//客户改变的消息类型
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    sessionId: 1299,//会话id
    clientId: "xxxxxxxxxx"//改变后客户的id
}
返回参数:
{
    result: 1,
    message: "success",
    clientId: "xxxxxxxxxx",
    sessionId: 1299
}
```

- <div id="queryHistoryMessages">查询当前客户聊天历史</div>
```
http://xxxxx/EliteWebChat/tpi
发送参数:
{
    type: 5003,//查询当前客户聊天历史消息
    token: "21B92585-4FDF-EF9D-1C8E-19D34D06F34B",//登录成功后获取的token值
    fromSessionId: 1299,//(可选)从哪个会话id开始往前查询，如果不传递或者传递0，则表示从最新会话开始往前搜索
    fromMessageId: 100302,//(可选)从那条消息开始往前查询，如果不传递或者传递0，则表示从最新消息开始往前搜索
	maxCount: 10,//(可选)最大查询消息条数，默认10条
	permFlag: 0,//(可选)0表示普通会话，1表示持久会话，默认0
}
返回参数:
{
    result: 1,
    message: "success",
    hisMessages: [
		{
            "_id": "",
            "contentLength": 0,
            "createTime": {
                "date": 25,
                "day": 2,
                "hours": 13,
                "minutes": 56,
                "month": 11,
                "seconds": 20,
                "time": 1545717380705,
                "timezoneOffset": -480,
                "year": 118
            },
            "extra": "",
            "firstReply": false,
            "id": 152814,
            "message": "聊天小助手，很高兴为您服务！请问有什么可以帮您？",
            "messageType": 2,
            "postTime": 1545703462782,
            "replyDelay": 0,
            "revokeFlag": 0,
            "sessionId": 0,
            "toUserId": "",
            "userId": "SELITE",
            "userType": 0
        },
		...
	]
}
```


## <div id="requestAndResponseCode">二. 请求代码与相应结果代码</div> ##
- <div id="requestCode">request中的type</div>
```
LOGON_REQUEST = 1;//登录
LOGOUT_REQUEST = 2;//登出
LOGON_OR_REGISTER_REQUEST = 3;//登录或注册
CHAT_REQUEST = 1000;//发出聊天排队请求
CHAT_REQUEST_STATE_UPDATE = 1001;//更新聊天排队请求状态
CANCEL_CHAT_REQUEST = 1002;//取消聊天排队请求
MSG_REQUEST = 2001;//发送文本消息
MSG_IMG_REQUEST = 2002;//发送图片消息
MSG_VOICE_REQUEST = 2003;//发送语音消息
MSG_LOCATION_REQUEST = 2004;//发送坐标地理消息
MSG_FILE_REQUEST = 2005;//发送文件消息
PRE_MSG_REQUEST = 2020;//发送预消息，当聊天会话还没创建时候，可以通过聊天请求id来发送预消息
PREVIEW_MSG_REQUEST = 2021;//发送预览消息，客户打字还没发送时候，就可以预先把打字内容作为预览消息发送给客服
TYPING_NOTICE = 2022;//正在输入提示
ROBOT_TRANSFER = 2023;//机器人转人工
CLOSE_REQUEST = 4001;//结束聊天
RATING_REQUEST = 5001;//满意度评价
CHANGE_CUSTOMER = 5002;//客户改变
QUERY_HISTORY_SESSION = 5003;//查询历史会话消息
```

- <div id="responseCode">response中的result</div>
```
SUCCESS = 1;
REQUEST_ALREADY_IN_ROULTING = -1;//请求已经在路由中了，意思是不用重复再次发出聊天请求
ALREADY_IN_CHATTING = -2;//已经在聊天中，意思不用重复发起聊天请求
NOT_IN_WORKTIME = -3;//排队返回，意思当前不是工作时间
INVAILD_SKILLGROUP = -4;//排队返回，意思是发出的queue值不对，不是预设的某个队列的值（微信端使用）
NO_AGENT_ONLINE = -5;//排队返回，当前没有坐席在线
INVAILD_CLIENT_ID = -6;//非法的客户id，发出的请求中客户id不正确
INVAILD_QUEUE_ID = -7;//非法的队列id，表示发出排队请求时候队列id传递的不对
REQUEST_ERROR = -8;//请求错误，解析发送来的请求json内容时候出错
INVAILD_TO_USER_ID = -9;//非法的to_user_id，发出的排队请求时候，如果带上了要排队给谁，这个谁如果没有这个人或者不在线，就会返回这个错
INVAILD_CHAT_REQUEST_ID = -10;//发送的请求id错误，没有找到这个id的请求
INVAILD_CHAT_SESSION_ID = -11;//发送的会话id错误，没有找到这个id的会话
UPLOAD_FILE_FAILED = -13;//上传文件失败
INVAILD_MESSAGE_TYPE = -12;//消息类型错误，收到的消息类型不是预设的消息类型
INVAILD_TOKEN = -15;//非法的token，查看调用接口时候传递的token，可能需要重新获取token
INVAILD_FILE_EXTENSION = -16;//非法的文件扩展类型，上传了某个不允许的文件类型
EMPTY_MESSAGE = -17;//发送或收到的消息是空内容时候报错
INVAILD_SESSION_TYPE = -18;//非法的会话类型，排队后，如果找到了一个已有的会话，但是这个会话不是需要的会话类型时候报错
INVAILD_REQUEST_TYPE = -19;//非法的请求类型，收到的消息请求类型不是预设的类型
INVAILD_LOGINNAME_OR_PASSWORD = -20;//登录异常，用户名或密码错误
INVAILD_FILE_SIZE = -21;//非法的文件大小，上传的文件大小超过了配置的阈值
UPLOAD_IMAGE_FAILED = -22;//上传图片文件失败
UPLOAD_VOICE_FAILED = -23;//上传语音文件失败
INVAILD_SIGN = -30;//非法的接口签名，查看签名逻辑是否正确
INTERNAL_ERROR = -100;//内部错误，请联系管理员
```

## <div id="requestResultCode">三. 聊天请求状态码</div> ##
```
WAITING = 0;
ACCEPTED = 1;
REFUSED = 2;
TIMEOUT = 3;
DROPPED = 4;
NO_AGENT_ONLINE = 5;
OFF_HOUR = 6;
CANCELED_BY_CLIENT = 7;
ENTERPRISE_WECHAT_ACCEPTED = 11;
```

## <div id="emoji">四. 消息体中表情的定义</div> ##

> 聊天消息中支持发送表情，比如，发送/::)，就表示微笑，一共支持如下105个表情，包括发送和接受都应该按这个规则来处理表情。
> 如果需要支持自定义表情，则只能通过&lt;img src='自定义表情.gif'&gt;方式来发送。

```
0 : "/::)", // 微笑
1 : "/::~", // 撇嘴
2 : "/::B", // 色
3 : "/::|", // 发呆
4 : "/:8-)", // 得意
5 : "/::<", // 流泪
6 : "/::$", // 害羞
7 : "/::X", // 闭嘴
8 : "/::Z", // 睡
9 : "/::'(", // 大哭
10 : "/::-|", // 尴尬
11 : "/::@", // 发怒
12 :"/::P", // 调皮
13 : "/::D", // 呲牙
14 : "/::O", // 惊讶
15 : "/::(", // 难过
16 : "/::+", // 酷
17 : "/:–-b", // 冷汗
18 : "/::Q", // 抓狂
19 : "/::T", // 吐
20 : "/:,@P", // 偷笑
21 : "/:,@-D", // 愉快
22 : "/::d", // 白眼
23 : "/:,@o", // 傲慢
24 : "/::g", // 饥饿
25 : "/:|-)", // 困
26 : "/::!", // 惊恐
27 : "/::L", // 流汗
28 : "/::>", // 憨笑
29 : "/::,@", // 悠闲
30 : "/:,@f", // 奋斗
31 : "/::-S", // 咒骂
32 : "/:?", // 疑问
33 : "/:,@x", // 嘘
34 : "/:,@@", // 晕
35 : "/::8", // 疯了
36 : "/:,@!", // 衰
37 : "/:!!!", // 骷髅
38 : "/:xx", // 敲打
39 : "/:bye", // 再见
40 : "/:wipe", // 擦汗
41 : "/:dig", // 抠鼻
42 : "/:handclap", // 鼓掌
43 : "/:&-(", // 糗大了
44 : "/:B-)", // 坏笑
45 : "/:<@", // 左哼哼
46 : "/:@>", // 右哼哼
47 : "/::-O", // 哈欠
48 : "/:>-|", // 鄙视
49 : "/:P-(", // 委屈
50 : "/::'|", // 快哭了
51 : "/:X-)", // 阴险
52 : "/::*", // 亲亲
53 : "/:@x", // 吓
54 : "/:8*", // 可怜
55 : "/:pd", // 菜刀
56 : "/:<W>", // 西瓜
57 : "/:beer", // 啤酒
58 : "/:basketb", // 篮球
59 : "/:oo", // 乒乓
60 : "/:coffee", // 咖啡
61 : "/:eat", // 饭
62 : "/:pig", // 猪头
63 : "/:rose", // 玫瑰
64 : "/:fade", // 凋谢
65 : "/:showlove", // 嘴唇
66 : "/:heart", // 爱心
67 : "/:break", // 心碎
68 : "/:cake", // 蛋糕
69 : "/:li", // 闪电
70 : "/:bome", // 炸弹
71 : "/:kn", // 刀
72 : "/:footb", // 足球
73 : "/:ladybug", // 瓢虫
74 : "/:shit", // 便便
75 : "/:moon", // 月亮
76 : "/:sun", // 太阳
77 : "/:gift", // 礼物
78 : "/:hug", // 拥抱
79 : "/:strong", // 强
80 : "/:weak", // 弱
81 : "/:share", // 握手
82 : "/:v", // 胜利
83 : "/:@)", // 抱拳
84 : "/:jj", // 勾引
85 : "/:@@", // 拳头
86 : "/:bad", // 差劲
87 : "/:lvu", // 爱你
88 : "/:no", // NO
89 : "/:ok", // OK
90 : "/:love", // 爱情
91 : "/:<L>", // 飞吻
92 : "/:jump", // 跳跳
93 : "/:shake", // 发抖
94 : "/:<O>", // 怄火
95 : "/:circle", // 转圈
96 : "/:kotow", // 磕头
97 : "/:turn", // 回头
98 : "/:skip", // 跳绳
99 : "/:oY", // 投降
100 : "/:#-0", // 激动
101 : "/:hiphot", // 乱舞
102 : "/:kiss", // 献吻
103 : "/:<&", // 左太极
104 : "/:&>" // 右太极
```
