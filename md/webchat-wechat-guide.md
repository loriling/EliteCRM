#WebChat与微信公众号对接接口说明

webchat已经可以和我们自己的livebot(微信公众号托管服务)对接，实现在微信公众号界面直接与客服聊天，如果其他公众号也想实现此功能，就需要按如下接口来实现。

接口都是使用http协议，post方式，参数直接通过body中传递json格式字符串（content-type: application/json）
主要分两部分：
> 一部分是webchat提供的接口，让微信托管服务来调用，这部分我们已经提供
> 另一部分是微信托管服务提供，让webchat来调用，这部分需要对方实现

具体接口定义如下：

##一.webchat端提供的接口（从微信托管服务发送请求到webchat)

- 从微信端发出聊天开始请求，通常发出后返回一个当前的排队位置

```
http://xxxxx/webchat/createcon.do
接收参数:
{
	openId: "xxxxx",//当前微信用户的openId
	skillGroup: 1,//chat排队的队列号
	nickname: "昵称",//（可选）
	headImgUrl: "头像地址",//（可选）
	requestTime: 1512026448289,//请求发出的时间戳
	epid: "服务号的id",//（可选）
	serviceAccount: "服务号名称",//（可选）
	agentId: "SELITE", //（可选）要排给的坐席id，只有当要固定排给某个人时候才需要传递
	tracks: "" //（可选）客户浏览轨迹 json字符串，具体格式查看相关文档
}

返回参数：
{
	result: 1,//1表示成功，详细看请求结果代码
	message: "ok",//结果的备注说明
	queueLength: 1,//当前队列长度
	sessionId: 123,//当持久队列中的持久会话，则直接返回此会话id（非持久会话时候是不会返回此参数）
	agentId: "SELITE",//专属坐席id（非持久会话时候是不会返回此参数）
	agentName: "伊力特"//专属坐席名称（非持久会话时候是不会返回此参数）
}

```

- 从微信端发消息给坐席，有的多媒体消息会有一个媒体id，webchat之后会通过这个媒体id去微信服务器上获取真正的媒体文件

```
http://xxxxx/webchat/getmsg.do
接收参数：
{
	sessionId: 123,//会话id
	epid: "服务号id",//(可选)
	agentId: "SELITE",//坐席id
	openId: "xxxxx",//
	msgType: "text",//消息类型，包括了（text，image，voice，video，link，location）
	content: ""//消息内容，不同消息类型，消息的内容格式也不同，除了text直接是字符串，其他都是一个json对象
}

image的content:
{
	mediaId: ""//图片对应的微信媒体id
}
voice的content:
{
	recognition: "",//语音识别出来的文字
	format: "",//语音文件格式
	mediaId: ""//语音对应的微信媒体id
}
video的content:
{
	thumbMediaId: "",//缩略图的微信媒体id
	mediaId: ""//视频对应的微信媒体id
}
link的content:
{
	url: "",//链接url
	title: "",//链接标题
	description: ""//描述信息
}
location的content:
{
	x: 121.23,//经度
	y: 34.213,//纬度
	scale: 1,//坐标地图大小
	label: ""//坐标标签
}

返回参数：
{
	result: 1,
	message: "ok"
}

```

- 从微信端结束聊天

```
http://xxxx/webchat/closecon.do
接收参数
{
	sessionId: 123,//会话id
	epid: "服务号id"//(可选)
}

返回参数：
{
	result: 1,
	message: "ok"
}

```

- 从微信端提交满意度

```
http://xxxx/webchat/ratesession.do
接收参数
{
	chatSessionId: 123,//会话id
	ratingId: 1,//满意度评分，具体值由具体项目业务决定
	ratingComment: "",//满意度备注信息
	epid: "服务号id",//(可选)
}

返回参数：
{
	result: 1,
	message: "ok"
}

```

##二. 微信托管服务提供的接口（从webchat端发消息给微信托管服务）

- 通知聊天开始。当客户发出过聊天请求后，请求经过排队，分配给了某个坐席，坐席接受了请求，这时候聊天会话就建立了，webchat也就会调用这个接口，通知微信托管服务。

```
http://xxxx/ari/create
接收参数：
{
	result: 1,//排队结果，1表示成功
	message: "ok",//备注消息
	agentId: "",//坐席id
	agentName: "",//坐席名
	sessionId: 123,//会话id
	responseTime: 1512026448289,//发送时间
	openId: "",//客户openId
	epid: "服务号id",
}

返回参数：
{
	result: 1,
	message: "ok"
}
```

- 发送消息给微信

```
http://xxxx/ari/msg
接受参数:
{
	sessionId: 123,//会话id
	agentId: "SELITE",//坐席id
	openId: "",//客户openId
	epid: "",//服务号id
	msgTime: 12341234324,//发送时间戳
	content: ""//消息内容，内容是带有html片段的富文本，需要微信端自己解析其中的表情或者图片
}

返回参数：
{
	result: 1,
	message: "ok"
}
```

- 坐席端通知微信聊天结束

```
http://xxxx/ari/close
接受参数:
{
	closeTime: 12341234324,//结束时间戳
	agentId: "",//坐席id
	sessionId: 123,//会话id
	openId: "",//客户openId
	epid: ""//服务号id
}

返回参数：
{
	result: 1,
	message: "ok"
}
```

- 获取媒体文件

```
http://xxxx/ari/getMedia
接受参数:
{
	mediaId: "xxx",//媒体id
	agentId: "",//坐席id
	sessionId: 123,//会话id
	openId: "",//客户openId
	epid: ""//服务号id
}

返回具体媒体文件的文件流
```

##常量说明

请求结果代码
```
public static final int SUCCESS = 1;
public static final int REQUEST_ALREADY_IN_ROULTING = -1;
public static final int ALREADY_IN_CHATTING = -2;
public static final int NOT_IN_WORKTIME = -3;
public static final int INVAILD_SKILLGROUP = -4;
public static final int NO_AGENT_ONLINE = -5;
public static final int INVAILD_TO_USER_ID = -9;
public static final int NO_SERVED_AGENT_FOUND = -10;
public static final int INTERNAL_ERROR = -100;
```