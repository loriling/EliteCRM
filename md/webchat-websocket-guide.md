# WebChat第三方客户端接入说明（WebSocket版本） #
11/16/2016 5:33:49 PM 
### 为了满足更高的消息时效性，websocket版本的接口应运而生 ###
登录接口与一些特殊接口（入文件上传）使用http协议，其余消息接口都是基于WebSocket协议，数据均采用json格式传输

## 一. 客户端登录并连接上websocket ##

- 客户端登录，登录采用http接口，url为： http://xxxx/webchat/tpi，参数是如下json

> 	请求示例：
> 	var params = {
        type: 1,
        loginName: 'lori',//登录名
        password: '',//密码
        epid: ''//企业号
    };
    $.ajax({
        url: "http://127.0.0.1:8980/webchat/tpi",
        method: "post",
        data : JSON.stringify(params)
    })；
> 	响应参数示例：
> 	{
> 		result: 1, // 1成功 0失败
> 		message: '', // 失败消息
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168'//成功后，会返回token作为之后的接口凭据
> 		config: {
> 		    //相关配置信息，暂无，待扩展
> 		}
> 	}

- 当登录成功后，就可以去连接websocket了：

> 	//这里需要传递登录成功后获取的token参数，不然没法正确握手
> 	ws = new WebSocket("ws://127.0.0.1:8980/webchat/cws?token=" + data.token);
> 	

## 二. 连上websocket后，就可以开始发送消息了 ##

- 客户端登出： 
> 	{
> 		messageId: 1,//消息id，唯一标示此消息，消息从服务端返回时候，也会带上此id，这样客户端就可以知道返回的消息是对应到客户端发出的哪个消息了
> 		type: 2,//登录
> 		time: 1400913140127,//时间戳
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168'//登录成功后获取到的凭据
> 	}

- 客户端发出聊天排队请求： 
> 	{
> 		messageId: 2,//消息id，唯一标示此消息
> 		type: 101,//人工聊天请求
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//时间戳
> 		queueId: 1, //排队的队列号
> 		from: "APP" //请求来源 分为:PC MOBILE WECHAT APP 不同来源会让坐席端看到不同客户的默认头像
> 	}

- 客户端发出取消聊天排队请求： 
> 	{
> 		messageId: 3,//消息id，唯一标示此消息
> 		type: 102,//取消请求
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//时间戳
> 		requestId: 724//排队成功后，获取到的排队号
> 	}

- 客户端发出结束聊天请求： 
> 	{
> 		messageId: 4,//消息id，唯一标示此消息
> 		type: 103,//结束聊天
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//时间戳
> 		sessionId: 68//排到队后，聊天开始时，获取到的会话id
> 	}

- 客户端发出满意度评价请求： 
> 	{
> 		messageId: 5,//消息id，唯一标示此消息
> 		type: 104,//满意度发送
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//时间戳
> 		sessionId: 68，//排到队后，聊天开始时，获取到的会话id
> 		rating: {
> 		    id: 1,//满意度评分
> 		    comments: ''//备注
> 		}
> 	}

- 客户端发出聊天消息： 
> 	{
> 		messageId: 6,//消息id，唯一标示此消息
> 		type: 110,//发送消息
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//发出请求的时间戳
> 		sessionId: 120,//聊天会话号，排队成功后返回
> 		msg: {
> 		    type: 1,//消息类型：1:文本, 2:图片, 3:文件, 4:位置, 5:语音 目前有这五种类型
> 		    content: {}//具体消息内容
> 		}
> 	}
> 	文本的content:
> 	{
> 	    content: '你好',//文本消息内容
> 	    extra: ''//额外消息
> 	}
> 	图片的content:
> 	{
> 	    name: 'xxx.png',//图片名称
> 	    imageUri: 'http://xxxx/xxx.png',//图片url地址
> 	    thumbData: 'xxxxx', //缩率图的base64
> 	    extra: ''//额外消息
> 	}
> 	文件的content:
> 	{
> 	    name: 'xxx.doc',
> 	    url: 'http://xxxxx/xxx.doc',
> 	    size: 1024, //文件大小
> 	    type: 'zip', //文件类型
> 	    extra: ''//额外消息
> 	}
> 	位置的content:
> 	{
> 	    poi: '',//地址
> 	    longitude: 180.345,//经度
> 	    latitude: 45.234,//纬度
> 	    map: 'baidu', //地图类型
> 	    thumbData: 'xxxxx', //缩略图的base64
> 	    extra: ''//额外信息
> 	}
> 	语音的content:
> 	{
> 	    voiceLength: 28,//语音时长，单位秒
> 	    voiceData: 'xxxxx'//语音的base64
> 	    extra: ''//额外信息
> 	}

- 上传文件接口（http）url为： http://xxxx/webchat/tpiu?token=xxxxxxxx
> 	
> 	请求时候需要带上token参数，请求体就是需要上传附件的byte
> 	
> 	响应参数示例：
> 	{
> 		result: 1, // 1成功 0失败
> 		message: '', // 失败消息
> 		url: '/webchat/xxxxx.jpg', //文件url地址
> 		fileName: 'xxxxx.jpg' //文件名字
> 	}


- 客户端发出预聊天消息（排队中，客户可以先发送消息，排上队后，之前发送的预消息会一次性发送给排到的坐席）： 
> 	{
> 		messageId: 7,//消息id，唯一标示此消息
> 		type: 111,//发送预消息
> 		token: '6CF7BBEB-8D6B-A4DF-D80C-D04FF1016168', //登录成功后获取到的凭据
> 		time: 1400913140127,//时间戳
> 		requestId: 724,//排队成功后，获取到的排队号
> 		content: 'xxxxxxxx'//发送的消息内容
> 	}

## 三. 客户端接收消息接口说明 ##

**一. 每个客户端发出的请求都会有响应的消息返回，表示发送成功还是失败**

- 客户端登出结果： 
> 	{
> 		messageId: 1,//消息id，这里的id就是之前调用此接口时候传递过来的id，客户端可以通过此id来知道是哪个请求的返回内容
> 		type: 2,//登出
> 		result: 1, // 1成功 0失败
> 		message: '', // 失败消息
> 	}

- 发出聊天排队请求结果： 
> 	{
> 		messageId: 2,//消息id，唯一标示此消息
> 		type: 101,
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}
> 	
- 取消请求结果： 
> 	{
> 		messageId: 3,//消息id，唯一标示此消息
> 		type: 102,
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}

- 结束会话结果： 
> 	{
> 		messageId: 4,//消息id，唯一标示此消息
> 		type: 103,
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}

- 满意度评价结果： 
> 	{
> 		messageId: 5,//消息id，唯一标示此消息
> 		type: 104,
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}

- 客户端发出聊天消息结果： 
> 	{
> 		messageId: 6,//消息id，唯一标示此消息
> 		type: 110,//发送消息
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}

- 客户端发出预聊天消息结果： 
> 	{
> 		messageId: 7,//消息id，唯一标示此消息
> 		type: 111,//发送消息
> 		result: 1, //结果代码
> 		message: ''//结果描述
> 	}

----------

**二. 客户端直接收到的相关消息**

- 通知客户端聊天排队状态更新： 
> 	
> 	{
> 		type: 201,//排队状态更新
> 		requestId: 724, //请求id号，如果发起请求成功，就会返回这个id
> 		requestStatus: 0,//当前请求状态
> 		queueLength: 3 //当前排到第几位
> 	}

- 通知客户端排队最终结果： 
> 	
> 	{
> 		type: 202,//通知排队结果
> 		sessionId: 68, //会话id号，排到队后会返回这个id
> 		agents: [
> 		    {
> 		        id: 'SELITE',
> 		        name: 'Elite',//名字
> 		        icon: 'http://xxxx/head.png',//头像地址
> 		        comments: ''//备注信息
> 		    }
> 		]
> 	}

- 坐席推送了满意度给客户： 
> 	
> 	{
> 		type: 203,//推送满意度
> 		sessionId: 68, //会话id号，排到队后会返回这个id
> 		agentId: 'SELITE'//发出消息的坐席id
> 	}

- 坐席人员信息变更，可能是修改了头像，可能是修改了名字，可能是换了个坐席，也可能是增加了一个坐席变成了会议方式聊天： 
> 	
> 	{
> 		type: 204,//坐席人员信息变更
> 		sessionId: 68, //会话id号，排到队后会返回这个id
> 		agents: [
> 		    {
> 		        id: 'SELITE',
> 		        name: 'Elite',//名字
> 		        icon: 'http://xxxx/head.png',//头像地址
> 		        comments: ''//备注信息
> 		    },
> 		    {
> 		        id: 'A00001',
> 		        name: 'Lori',//名字
> 		        icon: 'http://xxxx/lori.png',//头像地址
> 		        comments: '小组长'//备注信息
> 		    }
> 		]
> 	}

- 坐席结束了会话： 
> 	
> 	{
> 		type: 205,//会话被结束
> 		sessionId: 68, //会话id号，排到队后会返回这个id
> 		agentId: 'SELITE'//发出消息的坐席id
> 	}

- 收到坐席聊天消息： 
> 	
> 	{
> 		type: 210,//收到坐席聊天消息
> 		sessionId: 68, //会话id号，排到队后会返回这个id
> 		agentId: 'SELITE'//发出消息的坐席id
> 		msg: {
> 		    type: 1,//消息类型：1:文本, 2:图片, 3:文件, 4:位置, 5:语音, 99:系统通知 
> 		    content: ''//具体消息内容：图片和语音都是对应的base64码，位置是对应的json，文件需要调用额外的上传接口，上传成功后这里的内容为文件相关信息的json
> 		}
> 	}

## 四. 机器人聊天接口 ##

机器人接口采用http方式，调用URL： http://xxxx/webchat/robotSearch

- 机器人查询接口

> 	请求示例：
> 	用表单提交格式，POST方法，参数如下：
> 	"epid": "E00001", //企业id
> 	"action": "search", //查询答案
> 	"question": "测试"， //查询的问题
> 	"sign": "xxxxxxxxxxxxxxxxx" // 参数的签名

> 	签名规则：
> 	所有入参的值（除去sign）按参数名的字母排序，再加上PUBLIC_KEY，做MD5加密后的结果，用utf-8编码。具体PUBLIC_KEY询问相关人员。
> 	拿上面举例，假设PUBLIC_KEY=123456，排序后（action, epid, question）得到： searchE00001测试123456
> 	然后根据utf-8编码获取byte数组，再做MD5，得到sign


> 	返回数据json示例
> 	{
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
> 	}


## 五. 常量说明 ##

1. websocket消息类型说明

		//发送和接收通用消息
		int LOGON = 1;
		int LOGOUT = 2;
		int REGISTER = 3;
		
		//客户发送
		int SEND_CHAT_REQUEST = 101;//发出聊天请求
		int CANCEL_CHAT_REQUEST = 102;//取消聊天请求
		int CLOSE_SESSION = 103;//结束聊天
		int RATE_SESSION = 104;//满意度评价
		int SEND_CHAT_MESSAGE = 110;//发送聊天消息
		int SEND_PRE_CHAT_MESSAGE = 111;//发送预消息（还没排完队时候的消息）
		
		//客户接受
		int CHAT_REQUEST_STATUS_UPDATE = 201;//聊天排队状态更新
		int CHAT_STARTED = 202;//通知客户端可以开始聊天
		int AGENT_PUSH_RATING = 203;//坐席推送了满意度
		int AGENT_UPDATED = 204;//坐席人员变更
		int AGENT_CLOSE_SESSION = 205;//坐席关闭
		int RECEIVE_MESSAGE = 210;//收到聊天消息

2. 聊天消息类型说明

		int TEXT = 1;
		int IMG = 2;
		int FILE = 3;
		int LOCATION = 4;
		int VOICE = 5;
		int VIDEO = 6;
		int SYSTEM_NOTICE = 99;

3. 聊天请求状态码

		WAITING = 0;//等待中
		ACCEPTED = 1;//坐席接受
		REFUSED = 2;//坐席拒绝
		TIMEOUT = 3;//排队超时
		DROPPED = 4;//异常丢失
		NO_AGENT_ONLINE = 5;
		OFF_HOUR = 6;//不在工作时间
		CANCELED_BY_CLIENT = 7;//被客户取消
		ENTERPRISE_WECHAT_ACCEPTED = 11;//坐席企业号接收

4. 异常编码

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
		INVAILD_SIGN = -30
		INTERNAL_ERROR = -100;
