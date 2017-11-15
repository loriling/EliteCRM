
##ELiteWebChat jssdk 文档说明

网聊的 [WebSocket版本](http://loriling.github.io/EliteCRM/webchat-websocket-guide.html) 

####接口方法说明：
######引入 SDK
	<script src="http://www.elitecrm.com:9997/webchat/js/sdk/EliteIMLib.min.js"></script>

1、设置监听
```javascript
    EliteIMClient.setOnReceiveMessageListener({
        onSysNoticeReceived: function(data) {
            console.log("收到坐席的系统信息" + JSON.stringify(data));
             switch(data.noticeType){
                case EliteIMClient.$E.CONSTANTS.NOTICETYPE.INPUTING:
                    // data.content => 系统提示信息 坐席正在输入提示内容
                    //TODO
                    break;
                default:
                    //do something……
                    break;
        },
        onReceived: function(data) {
            console.log("收到坐席的信息" + JSON.stringify(data));
             switch(data.type){
                case EliteIMClient.CONSTANTS.MESSAGECODE.TEXT:
                    // data.content => 消息内容
                    //do something……
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.MESSAGECODE.IMG:
                    // data.content.imgUrl => 图片地址
                    // data.content.imgName => 图片名字
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.MESSAGECODE.FILE:
                    // data.content.fileUrl => 附件地址
                    // data.content.fileName => 附件名称
                    //do something……
                    break;
                case EliteIMClient.CONSTANTS.MESSAGECODE.LOCATION:
                    // data.content.longitude => 位置经度
                    // data.content.latitude => 位置纬度
                    // data.content.locationAddress => 位置名称
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.MESSAGECODE.VIDEO:
                    // data.content => 消息内容
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.MESSAGECODE.VOICE:
                    // data.content.voiceData => 语音内容
                    //data.content.voiceLength => 语音长度
                    //TODO
                    break;
                default:
                //do something……
        },
        onUpdateQueueStatus: function (data) {
            //TODO something
        },
        onSuccessQueue: function (sessionId) {
            //TODO something
        },
        onEndChat: function () {
            //TODO something
    
        }
    });
```

2、获取token
```javascript
   EliteIMClient.getToken({
        url: '/webchat',
        type: EliteIMClient.CONSTANTS.CHATTYPE.CUST_SERVICE,  //聊天类型  目前只有 1
        loginName: '111',
        password: '',
        urlFrom: 'WEB',
        epid: 'MAIN',
        statusChange: function(data) {
            console.log(data);
            switch (data.result) {
                case EliteIMClient.CONSTANTS.CONNECT.SUCCESS:
                    console.log('链接成功');
                    // data.token => token内容
                    break;
                case EliteIMClient.CONSTANTS.CONNECT.INVAILD_LOGINNAME_OR_PASSWORD:
                  console.log('用户名或者是密码错误');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.Uninitialized:
                  console.log('初始化状态');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.Open:
                  console.log('正在创建连接服务');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.Sent:
                  console.log('请求已发送到 Web 服务器');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.Receiving:
                  console.log('所有响应头部都已经接收到。响应体开始接收但未完成。');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.Loaded:
                  console.log('HTTP 响应已经完全接收。');
                  break;
                case EliteIMClient.CONSTANTS.ERRORCODE.INTERNAL_ERROR:
                  console.log('异常错误');
                  break;
                case EliteIMClient.CONSTANTS.AJAX.INVAILD_URL:
                    console.log('地址不正确');
                  break;
            }
        }
    }).done(function(data){
        console.log(data);
    })
```

3、 连接服务器
```javascript

	var token = "";
    EliteIMClient.connect({
    				"token": token
                	"url": ""
                    }, function() {
                    	//TODO open websocket 之后执行的内容
                    });
```

4、获取到用户信息
```javascript

    EliteIMClient.config.getFileAcceptExtensionArr(); //获取到上传附件所允许的格式
    EliteIMClient.config.getClient(); //获取到客户信息
    EliteIMClient.config.getAgent(); //获取坐席的信息
    EliteIMClient.config.getRatings(); //获取到配置的满意度的内容信息
    EliteIMClient.config.getUploadUrl(); // 获取到文件上传的服务地址
```

5、发送请求
```javascript

	var token = "******";
	var queue = 1;
    EliteIMClient.sendRequest({
        token: token,
        queue: queue,
        onSuccess: function (data) {
            console.log(data);
            if (data.result == 1) {
                console.log("[ 发送聊天请求成功 ]");
                isLiving = true;
                self.uiData.notice.show = true;
                self.uiData.notice.text = data.message;
            } else {
                console.log("[ 发送聊天请求失败 ]");
            }
        }
    });
```

6、发送信息
```javascript

    //发送TEXT 信息
    var text = '*****'; //文本内容
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.TextMessage(text, extra);
    EliteIMClient.sendMessage({
        token: this.token,
        msg: msg,
        onSuccess: function (data) {
            if (data.result == 1) {
                console.log("[ 发送成功 ]");
            } else {
                console.log("[ 发送失败 ]");
            }
        }
    });
    
    //发送IMG信息
    var name = '***'; // 图片名称
    var thumbData = '***'; // 图片base64编码
    var imageUri = '***'; // 图片地址
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.ImgMessage(name, thumbData, imageUri, extra);
    EliteIMClient.sendMessage({
        token: this.token,
        msg: msg,
        onSuccess: function (data) {
            if (data.result == 1) {
                console.log("[ 发送成功 ]");
            } else {
                console.log("[ 发送失败 ]");
            }
        }
    });
    //发送FILE信息
    var name = '***'; // 附件名称
    var url = '***'; // 附件地址
    var size = '***'; // 附件大小
    var type = '***'; // 附件类型
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.FileMessage(name, url, size, type, extra);
    EliteIMClient.sendMessage({
        token: this.token,
        msg: msg,
        onSuccess: function (data) {
            if (data.result == 1) {
                console.log("[ 发送成功 ]");
            } else {
                console.log("[ 发送失败 ]");
            }
        }
    });
    
    //发送LOCATION信息
    var poi = '***'; // 地址信息
    var longitude = '***'; // 经度
    var latitude = '***'; // 纬度
    var map = '***'; // 地图类型
    var thumbData = '***'; // 编码
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.LocationMessage(poi, longitude, latitude, map, thumbData, extra) ;
    EliteIMClient.sendMessage({
        token: this.token,
        msg: msg,
        onSuccess: function (data) {
            if (data.result == 1) {
                console.log("[ 发送成功 ]");
            } else {
                console.log("[ 发送失败 ]");
            }
        }
    });

    //发送VOICE信息
    var voiceLength = '***'; // 语音长度
    var voiceData = '***'; // 语音编码
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.VoiceMessage(voiceLength, voiceData, extra) ;
    EliteIMClient.sendMessage({
        token: this.token,
        msg: msg,
        onSuccess: function (data) {
            if (data.result == 1) {
                console.log("[ 发送成功 ]");
            } else {
                console.log("[ 发送失败 ]");
            }
        }
    });
```

7、表情转换
```javascript

	// 需要引入的css http://host:port/webchat/css/emotions.css
	var content = '[跳跳]';
	msgHtml = EliteIMClient.filterEmojis(content);
	//结果为："<img class="qqemoji qqemoji92" src="../../jsp/common/images/spacer.gif" height="1" width="1" />"
```

8、取消聊天
```javascript

    EliteIMClient.cancelRequest({
            onSuccess: function (data) {
                if (data.result == 1) {
                    console.log("[ 发送成功 ]");
                } else {
                    console.log("[ 发送失败 ]");
                }
            }
    });
```

9、推送满意度
   ```javascript
   
       var token = '***';
       var sessionId = '***';  //聊天的sessionId
       var rateId = '***'; //满意度id
       var rateConments = '***'; //满意度内容
       EliteIMClient.sendRating({
               "token": token,
               "sessionId": sessionId,
                "rateId": rateId, 
                "rateConments": rateConments,
               onSuccess: function (data) {
                   if (data.result == 1) {
                       console.log("[ 发送成功 ]");
                   } else {
                       console.log("[ 发送失败 ]");
                   }
               }
       });
   ```
   
10、推送结束聊天
```javascript

    var token = '***';
    var sessionId = '***';  //聊天的sessionId
    EliteIMClient.sendRating({
            "token": token,
            "sessionId": sessionId,
            onSuccess: function (data) {
                if (data.result == 1) {
                    console.log("[ 发送成功 ]");
                } else {
                    console.log("[ 发送失败 ]");
                }
            }
    });
```

11、关闭聊天
```javascript

    var token = '***';
    var sessionId = '***';  //聊天的sessionId
    EliteIMClient.closeChat({
            "token": token,
            "sessionId": sessionId,
            onSuccess: function (data) {
                if (data.result == 1) {
                    console.log("[ 发送成功 ]");
                } else {
                    console.log("[ 发送失败 ]");
                }
            }
    });
```

12、登出
```javascript

    var token = '***';
    EliteIMClient.loginOut({
            "token": token,
            onSuccess: function (data) {
                if (data.result == 1) {
                    console.log("[ 发送成功 ]");
                } else {
                    console.log("[ 发送失败 ]");
                }
            }
    });
```


##聊天插件配置：
chat插件是通过vue开发出来的，如果需要使用插件需要通过vue来引入插件并且需要引入几个必须的css 和 js：

```

    <link rel="stylesheet" type="text/css" href="../../css/elite-webchat-sdk.css">  //插件显示css
    <link rel="stylesheet" type="text/css" href="../../css/emotions.css"> //表情显示的css
    <script src="../../js/jquery/jquery-1.10.2.min.js"></script> //jQuery
    <script src="../../js/sdk/EliteIMLib.min.js"></script> // jssdk
    <script src="../../js/require.min.js" data-main="wsVue.js?123123"></script> //require
    <script src="../../js/jquery/wisdge.upload-1.0.js"></script> //文件上传
    <script type="text/javascript" src="https://api.map.baidu.com/api?v=2.0&ak=b3arF64BGGimKwGsp3KkN5st"></script> //百度地图jssdk
```
html代码：
```

	<div id="chatDemo">
		<chat queue='68' username="111" password="" epid="MAIN" urlFrom="WEBDEMO"></chat>
	</div>
```
创建vue对象：
```

	new Vue({
	    el: '#chatDemo'
	});
```
