# WebChat第三方客户端接入说明 #
10/24/2016 10:30:30 AM 
### EliteWebChat提供整套从客服端到客户端的界面与展示解决方案，同时也可以支持与第三方客户端的对接。其中成功案例就有与小米的米聊对接，与livebot的微信服务对接等。 ###

## 一. http对接接口说明 ##
接口才用http协议，包括了webchat端提供的接口与第三方提供的接口，数据传输使用json格式。
下面从模拟整个聊天过程的方式来介绍这些接口：


- 客户从网页或者微信端发出聊天请求： 
> 	http://xxxxx/EliteWebChat/tpi
> 	发送参数：
> 	{
> 		type: 1000,//发出聊天的请求
> 		clientId: "xxxxxxxxx",//客户的唯一标示
> 		nickname: "lori",//客户昵称（微信昵称）
> 		time: 1400913140127,//发出请求的时间戳
> 		queueId: 1,//请求的队列号
> 		from: "APP",//请求来源，分为:PC MOBILE WECHAT APP 不同来源会让坐席端看到不同客户的默认头像
> 		serverAddr: "http://www.xxxx.com" //请求方服务地址（可选）如果传递，则后面的回调都会按这个地址来
> 	}
> 	返回参数：
> 	{
> 		result: 1, //请求发送成功（1）或失败（0）
> 		message: "success", //成功或失败的消息
> 		requestId: 103, //请求id号，如果发起请求成功，就会返回这个id
> 		queueLength: 3, //队列长度
> 		continueLastSession: false//是不是继续上一次的会话，（可选）如果有且为true，表示这个请求是继续之前一个会话的，可能是之前那个会话没有正确的关闭掉造成的
> 	}

- 更新聊天排队情况： 
> 	http://xxxxx/EliteWebChat/tpi
> 	发送参数：
> 	{
> 		type: 1001,//查看聊天请求当前状态的请求
> 		requestId: 103//需要查看状态的请求id（发起聊天请求成功后会返回这个id）
> 	}
> 	返回参数：
> 	{
> 		result: 1,//请求发送成功（1）或失败（0）
> 		message: "success",//成功或失败的消息
> 		requestStatus: 1,//请求状态，具体状态码看下方说明（三. 聊天请求状态码）
> 		queueLength: 2//当前排队长度
> 	}
	
- 客户在排队中，就想发起消息，如果排队排上，之前的消息就会一起一次性发送给对应坐席： 
> 	http://xxxxx/EliteWebChat/tpi
> 	发送参数：
> 	{
> 		type: 2005,//发送预消息
> 		requestId: 103,//请求id（发起聊天请求成功后会返回这个id）
> 		text: 'xxxxxx'
> 	}
> 	返回参数：
> 	{
> 		result: 1,//请求发送成功（1）或失败（0）
> 		message: "success"//成功或失败的消息
> 	}

- 客户排队时间过长，想取消排队： 
> 	http://xxxxx/EliteWebChat/tpi
> 	发送参数：
> 	{
> 		type: 1002,//取消聊天排队请求
> 		requestId: 103//取消取消的请求id（发起聊天请求成功后会返回这个id）
> 	}
> 	返回参数：
> 	{
> 		result: 1,//请求发送成功（1）或失败（0）
> 		message: "success"//成功或失败的消息
> 	}

- <div style="color:#0086b3">webchat端路由请求到某个客服后，创建出聊天会话，通知第三方可以开始聊天了</div>
>     http://xxxxx/ThirdPartService/create
>     发送参数：
>     {
>         result：1,
>         message:"success",
>         agentId:"SELITE",
>         agentName:"Elite", //坐席昵称(webchat_alias)
>         agent: {
              id: "SELITE",
              name: "Elite", //坐席昵称(webchat_alias)
              alias: "坐席A", //坐席别名(staffname)
              icon: "http://xxxx/xxx/xx.png", //坐席头像url
              commonts: "备注信息" //备注信息
>         },
>         clientId:"xxxxxxxxx",
>         sessionId: 1299,
>         time: 1400913830109
>     }
>     返回参数:
>     {
>         result:1,
>         message:"success"
>     }

- 客户发送文本消息给客服
>     http://xxxxx/EliteWebChat/tpi
>     发送参数:
>     {
>         type: 2001,//发送文本消息请求
>         sessionId: 1299,
>         clientId: "xxxxxxxxxx",
>         time: 1400913830109,
>         text: "你好，请问xxxxx"
>     }
>     返回参数：
>     {
>         result: 1,
>         message: ""
>     }

- 客户发送图片消息给客服
>     http://xxxxx/EliteWebChat/tpi
>     发送参数:
>     {
>         type: 2002,//发送图片消息请求
>         sessionId: 1299,
>         clientId: "xxxxxxxxxx",
>         time: 1400913830109,
>         imgName: "IMG_001",
>         imgData: "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="//图片dataURI，标准的html5中dataURI的格式
>     }
>     返回参数：
>     {
>         result: 1,
>         message: ""
>     }

- <div style="color:#0086b3">客服发送消息给客户</div>
>     http://xxxxx/ThirdPartService/msg
>     {
>         sessionId: 1299,
>         agentId: "SELITE",
>         agentName: "Elite", //坐席昵称
>         agentHeadimg: "http://xxxx/xxx/xx.png", //坐席头像
>         agentComment: "备注信息", //坐席备注
>         time: 1400913830109,
>         content: "xxxxxxxxxx，xxxxx",
>         clientId: "xxxxxxxxxx",//（备用，可能不传）
>         type: "text"//（备用，可能不传）客服发送送出的微信消息目前只支持文字信息
>     }
>     返回参数：
>     {
>         result: 1,
>         message: "success"
>     }

- <div style="color:#0086b3">客服推送满意度给客户</div>
>     http://xxxxx/ThirdPartService/pushRating
>     发送参数:
>     {
>         sessionId: 1299,
>         agentId: 'SELITE',
>         clientId: 'xxxxxxxxxx'//（备用，可能不传）
>     }
>     返回参数:
>     {
>         result: 1,
>         message: "success"
>     }

- 客户结束聊天
>     http://xxxxx/EliteWebChat/tpi
>     发送参数:
>     {
>         type: 4001,
>         sessionId: 1299,
>         time: 1400913830109
>     }
>     返回参数：
>     {
>         result:1,
>         message:""
>     }

- <div style="color:#0086b3">客服关闭聊天会话</div>
>     http://xxxxx/ThirdPartService/close
>     发送参数:
>     {
>         sessionId: 1299,
>         agentId: 'SELITE',
>         clientId: 'xxxxxxxxxx',//（备用，可能不传）
>         time: 1400913830109
>     }
>     返回参数:
>     {
>         result: 1,
>         message: "success"
>     }

- 客户发出满意度评价请求
>     http://xxxxx/EliteWebChat/tpi
>     发送参数:
>     {
>         type: 5001,//满意度评价请求
>         sessionId: 1299,
>         ratingId: 1,//具体id值对应的含义需要询问具体项目人员
>         ratingComments: "服务非常满意"
>     }
>     返回参数:
>     {
>         result: 1,
>         message: "success"
>     }

- 客户改变请求
>     http://xxxxx/EliteWebChat/tpi
>     发送参数:
>     {
>         type: 5002,//客户改变的消息类型
>         sessionId: 1299,//会话id
>         clientId: "xxxxxxxxxx"//改变后客户的id
>     }
>     返回参数:
>     {
>         result: 1,
>         message: "success",
>         clientId: "xxxxxxxxxx",
>         sessionId: 1299
>     }

## 二. 请求代码与相应结果代码 ##
- request中的type
>      CHAT_REQUEST = 1000;
>      CHAT_REQUEST_STATE_UPDATE = 1001;
>      CANCEL_CHAT_REQUEST = 1002;
>      MSG_REQUEST = 2001;
>      MSG_IMG_REQUEST = 2002;
>      PRE_MSG_REQUEST = 2005
>      CLOSE_REQUEST = 4001;
>      RATING_REQUEST = 5001;

- response中的result
>      SUCCESS = 1;
>      REQUEST_ALREADY_IN_ROULTING = -1;
>      ALREADY_IN_CHATTING = -2;
>      NOT_IN_WORKTIME = -3;
>      INVAILD_SKILLGROUP = -4;
>      NO_AGENT_ONLINE = -5;
>      INVAILD_CLIENT_ID = -6;
>      INVAILD_QUEUE_ID = -7;
>      REQUEST_ERROR = -8;
>      INVAILD_TO_USER_ID = -9;
>      INVAILD_CHAT_REQUEST_ID = -10;
>      INVAILD_CHAT_SESSION_ID = -11;
>      INTERNAL_ERROR = -100;

## 三. 聊天请求状态码 ##
>      WAITING = 0;
>      ACCEPTED = 1;
>      REFUSED = 2;
>      TIMEOUT = 3;
>      DROPPED = 4;
>      NO_AGENT_ONLINE = 5;
>      OFF_HOUR = 6;
>      CANCELED_BY_CLIENT = 7;
>      ENTERPRISE_WECHAT_ACCEPTED = 11;

## 四. 消息体中表情的定义 ##
> 
> 聊天消息中支持发送表情，比如，发送/::)，就表示微笑，一共支持如下105个表情，包括发送和接受都应该按这个规则来处理表情。
> 如果需要支持自定义表情，则只能通过&lt;img src='自定义表情.gif'&gt;方式来发送。

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

