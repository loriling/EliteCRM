# WebChat第三方客户端接入说明（WebSocket版本） #
11/16/2016 5:33:49 PM 
### 为了满足更高的消息时效性，websocket版本的接口应运而生 ###
登录接口与一些特殊接口（入文件上传）使用http协议，其余消息接口都是基于WebSocket协议，数据均采用json格式传输

## 一. 客户端登录并连接上websocket ##

- 客户端登录，登录采用http接口，url为： http://xxxx/webchat/tpi，参数是如下json
```
请求示例：
//如果是登录
var params = {
    type: 1,//登录
    loginName: 'lori',//登录名
    password: '',//密码
	ip: '192.168.0.1',//客户访问ip（可选）
    epid: ''//企业号(可选)
};
//也可以是登录加注册，如果有这个用户名的用户就直接使用，如果没有就直接注册一个新的
var params = {
    type: 3,//登录或者注册
    loginName: 'lori',//登录名
    name: '罗瑞',//姓名（可选）
	portraitUri： '',//头像地址（可选）
	ip: '192.168.0.1',//客户访问ip（可选）
    epid: ''//企业号（可选）
};
//如果是匿名访客登录
var params = {
    type: 4,//访客登录
    visitorId: 'xxxxxxxx',//访客唯一标示guid
    ip: '192.168.0.1',//访客请求过来的ip地址（可选）
    epid: ''//企业号（可选）
};
//发出请求
$.ajax({
    url: "http://127.0.0.1:8980/webchat/tpi",
    method: "post",
    data : JSON.stringify(params)
});
//响应参数示例：
{
	result: 1, // 1成功 0失败
	message: '', // 失败消息
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168'//成功后，会返回token作为之后的接口凭据
	config: {
	    //相关配置信息，暂无，待扩展
	}
}
```

- 当登录成功后，就可以去连接websocket了：
```
//这里需要传递登录成功后获取的token参数，不然没法正确握手
ws = new WebSocket("ws://127.0.0.1:8980/webchat/cws?token=" + data.token);
```

## 二. 连上websocket后，就可以开始发送消息了 ##

- 客户端登出： 
```
{
	messageId: 1,//消息id，唯一标示此消息，消息从服务端返回时候，也会带上此id，这样客户端就可以知道返回的消息是对应到客户端发出的哪个消息了
	type: 2,//登出
	time: 1400913140127,//时间戳
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168'//登录成功后获取到的凭据
}
```

- 客户端发出聊天排队请求： 
```
{
	messageId: 2,//消息id，唯一标示此消息
	type: 101,//人工聊天请求
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//时间戳
	queueId: 1, //排队的队列号
	from: "APP", //（可选）请求来源 分为:PC MOBILE WECHAT APP 不同来源会让坐席端看到不同客户的默认头像
	brand: "",//（可选）请求来源2，作为第二个来源字段
	toUserId: "SELITE", //（可选）直接指明排队给某个坐席的坐席id，不传递则由系统自动分配
	tracks: "" //（可选）客户浏览轨迹 json字符串，具体格式查看相关文档
}
```

- 客户端发出取消聊天排队请求： 
```
{
	messageId: 3,//消息id，唯一标示此消息
	type: 102,//取消请求
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//时间戳
	requestId: 724//排队成功后，获取到的排队号
}
```

- 客户端发出结束聊天请求： 
```
{
	messageId: 4,//消息id，唯一标示此消息
	type: 103,//结束聊天
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//时间戳
	sessionId: 68//排到队后，聊天开始时，获取到的会话id
}
```

- 客户端发出满意度评价请求： 
```
{
	messageId: 5,//消息id，唯一标示此消息
	type: 104,//满意度发送
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//时间戳
	sessionId: 68，//排到队后，聊天开始时，获取到的会话id
	rating: {
	    ratingId: 1,//满意度评分
	    ratingComments: ''//（可选）备注
	}
}
```

- 客户端发出聊天消息： 
```
{
	messageId: 6,//消息id，唯一标示此消息
	type: 110,//发送消息
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//发出请求的时间戳
	sessionId: 120,//聊天会话号，排队成功后返回
	msg: {
	    type: 1,//消息类型：1:文本, 2:图片, 3:文件, 4:位置, 5:语音 目前有这五种类型
	    content: {}//具体消息内容
	}
}
文本的content:
{
    text: '你好',//文本消息内容
    extra: ''//（可选）额外消息
}
图片的content:
{
    name: 'xxx.png',//图片名称
    imageUri: 'http://xxxx/xxx.png',//图片url地址
    thumbData: 'xxxxx', //（可选）缩率图的base64
    extra: ''//（可选）额外消息
}
文件的content:
{
    name: 'xxx.doc',
    url: 'http://xxxxx/xxx.doc',
    size: 1024, //（可选）文件大小
    type: 'zip', //（可选）文件类型
    extra: ''//（可选）额外消息
}
位置的content:
{
    poi: '',//（可选）地址
    longitude: 180.345,//经度
    latitude: 45.234,//纬度
    map: 'baidu', //（可选）地图类型
    thumbData: 'xxxxx', //（可选）缩略图的base64
    extra: ''//（可选）额外信息
}
语音的content:
{
    voiceLength: 28,//语音时长，单位秒
    voiceData: 'xxxxx'//语音的base64
    extra: ''//（可选）额外信息
}
```
- 当收到坐席发送消息后，需要发送消息回执(服务端如果配置了开启回执功能的话)，通知服务器消息已经收到
```
{
	messageId: 7,//消息id，唯一标示此消息
	type: 120,//接受消息回执
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//发出请求的时间戳
	rsId: '597eb7b6-b7b5-4238-8dc8-86b739cf5788'//用来消息发送消息回执时候传递的id
}
```

- 上传文件接口（http）url为： http://xxxx/webchat/tpiu?token=xxxxxxxx
```
请求时候需要带上token参数，请求体就是需要上传附件的byte
响应参数示例：
{
	result: 1, // 1成功 0失败
	message: '', // 失败消息
	url: '/webchat/xxxxx.jpg', //文件url地址
	fileName: 'xxxxx.jpg' //文件名字
}
```


- 客户端发出预聊天消息（排队中，客户可以先发送消息，排上队后，之前发送的预消息会一次性发送给排到的坐席）： 
```
{
	messageId: 7,//消息id，唯一标示此消息
	type: 111,//发送预消息
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	time: 1400913140127,//时间戳
	requestId: 724,//排队成功后，获取到的排队号
	content: 'xxxxxxxx'//发送的消息内容
}
```

- 客户端发出预览消息
```
//客户输入一些内容，但是这时候还没有按发送的时候，可以发送预览消息出来，让坐席预先知道客户可能会发送什么信息
//（此类消息可以在客户输入内容改变时候，发送出来当前已经输入的内容，但需要注意发送频率，比如可以加个时间5秒内发送一次）
{
	messageId: 8,//消息id，唯一标示此消息
	type: 112,//发送预消息
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	sessionId: 120,//聊天会话号，排队成功后返回
	content: "xxxxx"//预览的消息内容
}
```

- 客户端发出正在输入提示
```
//（此类消息建议在用户焦点进入输入框时候发送，需要注意发送频率，比如可以加个时间5秒内发送一次）
{
	messageId: 9,//消息id，唯一标示此消息
	type: 113,//发送预消息
	token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
	sessionId: 120//聊天会话号，排队成功后返回
}
```

- 心跳请求： 
```
{
	messageId: 10,//消息id，唯一标示此消息
	type: 10//心跳检测的消息类型编号
}
```

## 三. 客户端接收消息接口说明 ##

**一. 每个客户端发出的请求都会有响应的消息返回，表示发送成功还是失败。**

- 客户端登出结果： 
```
{
	messageId: 1,//消息id，这里的id就是之前调用此接口时候传递过来的id，客户端可以通过此id来知道是哪个请求的返回内容
	type: 2,//登出
	result: 1, // 1成功 0失败
	message: '' // 失败消息
}
```

- 发出聊天排队请求结果： 
```
{
	messageId: 2,//消息id，唯一标示此消息
	type: 101,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 取消请求结果： 
```
{
	messageId: 3,//消息id，唯一标示此消息
	type: 102,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 结束会话结果： 
```
{
	messageId: 4,//消息id，唯一标示此消息
	type: 103,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 满意度评价结果： 
```
{
	messageId: 5,//消息id，唯一标示此消息
	type: 104,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 客户端发出聊天消息结果： 
```
{
	messageId: 6,//消息id，唯一标示此消息
	type: 110,//发送消息
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 客户端发出预聊天消息结果： 
```
{
	messageId: 7,//消息id，唯一标示此消息
	type: 111,//发送消息
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 客户端发出预览消息结果： 
```
{
	messageId: 8,//消息id，唯一标示此消息
	type: 112,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 客户端发出正在输入结果： 
```
{
	messageId: 9,//消息id，唯一标示此消息
	type: 113,
	result: 1, //结果代码
	message: ''//结果描述
}
```

- 心跳请求返回结果： 
```
{
	messageId: 10,//消息id，唯一标示此消息
	type: 10,
	result: 1//结果代码
}
```

----------

**二. 客户端直接收到的相关消息**
<span style="color:#e74c3c">如果配置了消息重发机制，每个客户端收到的消息还会多一个**rsId**的属性，客户端收到消息后，需要立即发送消息回执，并传递这个rsId回去，表示这条消息已经收到，不然时间到了没有收到回执看会触发重发机制。</span>

- 客户端连上websocket之后，告知客户端相关配置信息： 
```
{
	type: 200,//初始化连上ws后返回的配置信息
	ratings: [],//满意度评价相关信息，用来告诉客户端目前所有的可选的满意度选项，这个仅用于参考，客户端可以不考虑这个配置信息
	fileAcceptExtensionsArr: "zip,rar,jpg,jpeg,png,gif,bmp,doc",//所有允许的上传的附件的扩展名,字符串,多个用逗号分隔
	hisSessions: [1288,1290,1310],//当登录客户的历史会话数组
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 通知客户端聊天排队状态更新： 
```
{
	type: 201,//排队状态更新
	requestId: 724, //请求id号，如果发起请求成功，就会返回这个id
	sessionId: 10, //（可能存在）如果是机器人转人工的请求，就会传递当前会话的sessionId
	requestStatus: 0,//当前请求状态，具体看：聊天请求状态码
	queueLength: 3, //当前排到第几位
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 通知客户端排队最终结果： 
```
{
	type: 202,//通知排队结果
	sessionId: 68, //会话id号，排到队后会返回这个id
	continueLastSession: false，//是否是继续的上一个会话（当用户异常断开连接后马上再次排队，这时候服务端会继续上一次会话）
	users: [//这个会话中所有人员的信息，坐席的话会有comments，客户没有
	    {
	        id: 'SELITE',//坐席id，通常是6位
	        name: 'Elite',//名字
	        icon: 'http://xxxx/head.png',//头像地址
	        comments: ''//备注信息
	    },
	    {
	        id: 'A9BA8B8A-A888-9888-8BBA-BABB9A88BB98',//客户guid，通常36位
	        name: '客户1',//名字
	        icon: 'http://xxxx/head.png'//头像地址
	    }
	],
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 坐席推送了满意度给客户： 
```
{
	type: 203,//推送满意度
	sessionId: 68, //会话id号，排到队后会返回这个id
	agentId: 'SELITE',//发出消息的坐席id
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 坐席人员信息变更，可能是修改了头像，可能是修改了名字，可能是换了个坐席，也可能是增加了一个坐席变成了会议方式聊天： 
```
{
	type: 204,//坐席人员信息变更
	sessionId: 68, //会话id号，排到队后会返回这个id
	agents: [
	    {
	        id: 'SELITE',
	        name: 'Elite',//名字
	        icon: 'http://xxxx/head.png',//头像地址
	        comments: ''//备注信息
	    },
	    {
	        id: 'A00001',
	        name: 'Lori',//名字
	        icon: 'http://xxxx/lori.png',//头像地址
	        comments: '小组长'//备注信息
	    }
	],
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 坐席结束了会话： 
```
{
	type: 205,//会话被结束
	sessionId: 68, //会话id号，排到队后会返回这个id
	agentId: 'SELITE',//发出消息的坐席id
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
```

- 收到坐席聊天消息： 
```
消息类型一共有：1:文本, 2:图片, 3:文件, 4:位置, 5:语音, 99:系统通知 
目前坐席可能发出的正常聊天消息type可能是1:文本，2:图片
{
	type: 210,//收到坐席聊天消息
	sessionId: 68, //会话id号，排到队后会返回这个id
	agentId: 'SELITE'//发出消息的坐席id
	msg: {
	    type: 1,//消息类型，正常消息是1或者2
	    content: ''//具体消息内容：图片和语音都是对应的base64码，位置是对应的json，文件需要调用额外的上传接口，上传成功后这里的内容为文件相关信息的json
	},
	rsId: "xxxxxxx"//如果配置重发机制就会此属性,用来消息发送消息回执时候传递的id
}
坐席也可能发出type是99的通知类消息，通知类消息还有个noticeType表示通知类型
TYPING = 5;//用户正在输入提示
INVITE_NOTICE = 10;//会议后提示
TRANSFER_NOTICE = 11;//转接后提示
USER_LEFT_NOTICE = 12;//用户从会话中离开提示（通常在会议模式下，有坐席离开时候）
CUSTOM = 99;//自定义提示
{
	type: 210,//收到坐席聊天消息
	sessionId: 68, //会话id号，排到队后会返回这个id
	agentId: 'SELITE'//发出消息的坐席id
	msg: {
	    type: 99,//消息类型
	    noticeType: 5,//通知类型, 5表示坐席正在输入内容
		content: ""//通知内容
	}
}
```

## 四. 机器人聊天接口 ##

机器人接口采用http方式，调用URL： http://xxxx/webchat/robotSearch

- 机器人查询接口
```
	请求示例：
	用表单提交格式，POST方法，参数如下：
	"epid": "E00001", //企业id
	"action": "search", //查询答案
	"question": "测试"， //查询的问题
	"sign": "xxxxxxxxxxxxxxxxx" // 参数的签名
	
	签名规则：
	所有入参的值（除去sign）按参数名的字母排序，再加上PUBLIC_KEY，做MD5加密后的结果，用utf-8编码。具体PUBLIC_KEY询问相关人员。
	拿上面举例，假设PUBLIC_KEY=123456，排序后（action, epid, question）得到： searchE00001测试123456
	然后根据utf-8编码获取byte数组，再做MD5，得到sign
	
	返回数据json示例
	{
		"searchResult" : {
			"question" : "测试",
			"srs" : [{ //返回的查询结果数组
					"preview" : "<span style=\"color:#FFFFF0;background-color:#1E90FF;font-weight:bold;\" id=\"kmhighlighter\">测试<\/span>1.1 真的，1居然也会高亮", //预览信息
					"question" : "",
					"ra" : { //具体文章信息
						"articleAId" : "3baf0f47-f232-1737-7610-e65f77b2d6d0",
						"path" : "/oldkm/ArticleStore/E9D9C25E_1741_48C9_915E_2B6EE7FA6385/3baf0f47-f232-1737-7610-e65f77b2d6d0.htm",//文章路径
						"title" : "测试标题"//文章标题
					},
					"title" : "测试标题"
				}
			]
		},
		"result" : 1,
		"message" : ""
	}
```


## 五. 常量说明 ##

1. websocket消息类型说明
```
	//发送和接收通用消息
	int LOGON = 1;//登录
	int LOGOUT = 2;//登出
	int LOGON_OR_REGISTER_REQUEST = 3;//登录或者注册
	int VISITOR_LOGON = 4;//匿名访客登录
	
	//客户发送
	int SEND_CHAT_REQUEST = 101;//发出聊天请求
	int CANCEL_CHAT_REQUEST = 102;//取消聊天请求
	int CLOSE_SESSION = 103;//结束聊天
	int RATE_SESSION = 104;//满意度评价
	int SEND_CHAT_MESSAGE = 110;//发送聊天消息
	int SEND_PRE_CHAT_MESSAGE = 111;//发送预消息（还没排完队时候的消息）
	int PREVIEW_CHAT_MESSAGE = 112;//发出预览信息，客户输入的内容，这时候还没有真正发送信息，就可以先发送这个接口过来给坐席"预览"
	int SEND_TYPING_NOTICE = 113;//发出正在输入提示
	int MESSAGE_RECEIPT = 120;//消息回执，客户端通知服务坐席发出的消息已经收到
	int SEND_CUSTOM_MESSAGE = 199;//发送自定义消息
	
	//客户接受
	int CHAT_INIT_CONFIG = 200;//初始化连上ws后返回的配置信息
	int CHAT_REQUEST_STATUS_UPDATE = 201;//聊天排队状态更新
	int CHAT_STARTED = 202;//通知客户端可以开始聊天
	int AGENT_PUSH_RATING = 203;//坐席推送了满意度
	int AGENT_UPDATED = 204;//坐席人员变更
	int AGENT_CLOSE_SESSION = 205;//坐席关闭
	int RECEIVE_MESSAGE = 210;//收到聊天消息

	int ROBOT_MESSAGE = 301;//机器人相关消息
	int ROBOT_TRANSFER_MESSAGE = 302;//机器人转人工消息
```

2. 聊天消息类型说明
```
	int TEXT = 1;
	int IMG = 2;
	int FILE = 3;
	int LOCATION = 4;
	int VOICE = 5;
	int VIDEO = 6;
	int SYSTEM_NOTICE = 99;
```

3. 聊天请求状态码
```
	WAITING = 0;//等待中
	ACCEPTED = 1;//坐席接受
	REFUSED = 2;//坐席拒绝
	TIMEOUT = 3;//排队超时
	DROPPED = 4;//异常丢失
	NO_AGENT_ONLINE = 5;//无坐席在线
	OFF_HOUR = 6;//不在工作时间
	CANCELED_BY_CLIENT = 7;//被客户取消
	ENTERPRISE_WECHAT_ACCEPTED = 11;//坐席企业号接收
```

4. 异常编码
```
	SUCCESS = 1;
	REQUEST_ALREADY_IN_ROULTING = -1;
	ALREADY_IN_CHATTING = -2;
	NOT_IN_WORKTIME = -3;
	INVAILD_SKILLGROUP = -4;
	NO_AGENT_ONLINE = -5;
	INVAILD_CLIENT_ID = -6;
	INVAILD_QUEUE_ID = -7;
	REQUEST_ERROR = -8;
	INVAILD_TO_USER_ID = -9;
	INVAILD_CHAT_REQUEST_ID = -10;
	INVAILD_CHAT_SESSION_ID = -11;
	INVAILD_MESSAGE_TYPE = -12;
	UPLOAD_FILE_FAILED = -13;
	INVAILD_PARAMETER = -14;
	INVAILD_TOKEN = -15;
	INVAILD_FILE_EXTENSION = -16;
	EMPTY_MESSAGE = -17;
	INVAILD_LOGINNAME_OR_PASSWORD = -20;
	INVAILD_FILE_SIZE = -21;
	UPLOAD_IMAGE_FAILED = -22;
	UPLOAD_VOICE_FAILED = -23;
	INVAILD_SIGN = -30
	INTERNAL_ERROR = -100;
```
