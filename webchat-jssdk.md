
## ELiteWebChat jssdk 文档说明

网聊的 [WebSocket版本](http://loriling.github.io/EliteCRM/webchat-websocket-guide.html) 

#### 接口方法说明：
###### 引入 SDK
    <script src="http://www.elitecrm.com:9997/webchat/js/sdk/EliteIMLib.min.js"></script>

1、设置监听
```javascript
    EliteIMClient.setOnReceiveMessageListener({
        onSysNoticeReceived: function(data) {
            console.log("收到坐席的系统信息" + JSON.stringify(data));
             switch(data.noticeType){
                case EliteIMClient.CONSTANTS.NOTICETYPE.NORMAL:
                    // data.content => 系统提示信息 坐席正在输入提示内容
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.TRACK_CHANGE:
                    // data.content => 系统提示信息 访客轨迹改变提示
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.INPUTING:
                    // data.content => 系统提示信息 坐席正在输入提示内容
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.INVITE_NOTICE:
                    // data.content => 系统提示信息 会议后提示
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.TRANSFER_NOTICE:
                    // data.content => 系统提示信息 转接后提示
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.USER_LEFT_NOTICE:
                    // data.content => 系统提示信息 用户从会话中离开提示（通常在会议模式下，有坐席离开时候）
                    //TODO
                    break;
                case EliteIMClient.CONSTANTS.NOTICETYPE.CUSTOM:
                    // data.content => 系统提示信息 自定义提示
                    //TODO
                    break;
                default:
                    /**
                    *  其他的提示  通过NGS命令
                    *  var noticeType = 100; //这里也可以自己修改这个数字，100以上都可以作为自定义的类型值
                    * $project.events.notify(window.$CONST.ChatEvent.SEND_MESSAGE, {
                      		sessionId: [temp.SessionID],//会话id
                      		content: '请填写表格',//消息文本内容
                      		noticeType: noticeType, //（可选）
                      		receiverId: '[temp.CHAT_CLIENTID]',
                                  success: function(){ //发送成功后的回调函数
                      			console.log('发送成功');
                      		}
                      	});
                    */
                    //do something……
                    break;
        },
        onReceived: function (data) {
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
            //TODO something 更新排队信息
            switch (data.requestStatus) {
                case EliteIMClient.CONSTANTS.SENDREQUEST.REFUSED :
                    console.log('坐席拒绝');
                    break;
                case EliteIMClient.CONSTANTS.SENDREQUEST.TIMEOUT :
                    console.log('请求超时');
                    break;
                case EliteIMClient.CONSTANTS.SENDREQUEST.DROPPED :
                    console.log('异常丢失');
                    break;
                case EliteIMClient.CONSTANTS.SENDREQUEST.ENTERPRISE_WECHAT_ACCEPTED :
                    console.log('企业号接受信息');
                    break;
            }
        },
        onSuccessQueue: function (sessionId) {
            //TODO something 排队成功
        },
        onEndChat: function () {
            //TODO something 结束聊天进行的操作
    
        }
    });
```

2、获取token
```javascript
    //普通登陆
    var loginName = 'lori';//登录名
    var password = '';//密码
    var epid = '';//企业号(可选)
    var params = EliteIMClient.LoginParam(loginName, password, epid); // 登陆客户
    
    //注册客户并且登陆
    var loginName = 'lori';//登录名
    var name = '罗瑞';//姓名（可选）
    var portraitUri = '';//头像地址（可选）
    var epid = '';//企业号（可选）
    var params = EliteIMClient.RegistParam(loginName, name, portraitUri, epid); // 注册客户
    
    //访客模式登陆
    var visitorId = 'xxxxxxxx';//访客唯一标示guid
    var ipAddr = '';//访客请求过来的ip地址（可选）
    var epid = '';//企业号（可选）
    var params = EliteIMClient.VisitParam(visitorId, ipAddr, epid); // 访客客户
   
   var url = ""; //网聊的
   EliteIMClient.getToken({
        url: '',
        params: params, // 创建出来的params 方法返回的结果
        statusChange: function (data) {
            switch (data.result) {
                case EliteIMClient.CONSTANTS.CONNECT.SUCCESS:
                    console.log('链接成功');
                    self.token = data.token;
                    break;
                case EliteIMClient.CONSTANTS.CONNECT.INVAILD_LOGINNAME_OR_PASSWORD:
                    console.log('用户名或者是密码错误');
                    break;
                case EliteIMClient.CONSTANTS.AJAX.UNINITIALIZED:
                    console.log('初始化状态');
                    break;
                case EliteIMClient.CONSTANTS.AJAX.OPEN:
                    console.log('正在创建连接服务');
                    break;
                case EliteIMClient.CONSTANTS.AJAX.SENT:
                    console.log('请求已发送到 Web 服务器');
                    break;
                case EliteIMClient.CONSTANTS.AJAX.RECEIVING:
                    console.log('所有响应头部都已经接收到。响应体开始接收但未完成。');
                    break;
                case EliteIMClient.CONSTANTS.AJAX.LOADED:
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
    token可以通过两种方式来获取分为两种方式 [跨域请求] 和 [不跨域请求]:
        1.不跨域请求方式可以直接通过jssdk中的 [获取token]的方法来获取到token
        2.跨域的时候需要通过后台程序去调用： http://xxxx/webchat/tpi，参数是如下json 参考 网聊的 [WebSocket版本](http://loriling.github.io/EliteCRM/webchat-websocket-guide.html) 文档内部的接口说明来获取
    注: token 有效的时间是24小时， 建议每次获取之后保存下来，当发送信息token失效之后再次重复获取   
    
    var token = "";
    var wsUrl = ""; // 网聊websocket地址假设网聊的地址是： http://192.168.2.99/webchat, websocket地址为 ws://192.168.2.99/webchat
    EliteIMClient.connect({
                    "token": token,
                    "url": wsUrl,
                    onclose: function(e) {
                        //TODO close socket do something
                        console.log("console close by html " + e);
                    },
                    onerror: function() {
                        //TODO error socket do something
                        console.log("console error by html " + e);
                    }
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
    注： 在跨域的情况下 发送附件的地址：上传文件接口（http）url为： http://xxxx/webchat/tpiu?token=xxxxxxxx
        请求时候需要带上token参数，请求体就是需要上传附件的byte
        
        响应参数示例：
        {
            result: 1, // 1成功 0失败
            message: '', // 失败消息
            url: '/webchat/xxxxx.jpg', //文件url地址
            fileName: 'xxxxx.jpg' //文件名字
        }
```

5、发送请求
```javascript

    var queue = 1;
    var token = "";  //前面获取到的token的值
    var urlfrom = "MOBILE"; //（可选）请求来源 分为:PC MOBILE WECHAT APP 不同来源会让坐席端看到不同客户的默认头像
    EliteIMClient.sendRequest({
        token: token,
        queue: queue,
        urlfrom: urlfrom,
        onSuccess: function (data) {
            console.log("[ 排队中 ]" + JSON.stringify(data)); 
        },
        onOffhour: function (data) {
            console.log("[ 不在工作时间 ]" + JSON.stringify(data));  
            //TODO  something         
        },
        onUnline: function (data) {
            console.log("[ 坐席不在线 ]" + JSON.stringify(data));    
            //TODO  something          
        },
        onFail: function(data) {
            console.log("[ 发送聊天请求失败 ]" + JSON.stringify(data));
            //TODO  something   
        }
    });
```

6、发送信息
```javascript
    //发送TEXT 信息
    var text = '*****'; //文本内容
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.TextMessage(text, extra);
    var token = "";  //前面获取到的token的值
    EliteIMClient.sendMessage({
        token: token,
        msg: msg,
        onSuccess: function (data) {
                console.log("[ 发送成功 ]");
        },
        onFail: function(data) {
             console.log("[ 发送失败 ]");
        }
    });
    
    //发送IMG信息
    var name = '***'; // 图片名称
    var thumbData = '***'; // 图片base64编码
    var imageUri = '***'; // 图片地址
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.ImgMessage(name, thumbData, imageUri, extra);
    var token = "";  //前面获取到的token的值
    EliteIMClient.sendMessage({
        token: token,
        msg: msg,
        onSuccess: function (data) {
                console.log("[ 发送成功 ]");
        },
        onFail: function(data) {
             console.log("[ 发送失败 ]");
        }
    });
    //发送FILE信息
    var name = '***'; // 附件名称
    var url = '***'; // 附件地址
    var size = '***'; // 附件大小
    var type = '***'; // 附件类型
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.FileMessage(name, url, size, type, extra);
    var token = "";  //前面获取到的token的值
    EliteIMClient.sendMessage({
        token: token,
        msg: msg,
        onSuccess: function (data) {
                console.log("[ 发送成功 ]");
        },
        onFail: function(data) {
             console.log("[ 发送失败 ]");
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
    var token = "";  //前面获取到的token的值
    EliteIMClient.sendMessage({
        token: token,
        msg: msg,
        onSuccess: function (data) {
                console.log("[ 发送成功 ]");
        },
        onFail: function(data) {
             console.log("[ 发送失败 ]");
        }
    });

    //发送VOICE信息
    var voiceLength = '***'; // 语音长度
    var voiceData = '***'; // 语音编码
    var extra = '***'; // 扩展信息
    var msg = new EliteIMClient.VoiceMessage(voiceLength, voiceData, extra) ;
    var token = "";  //前面获取到的token的值
    EliteIMClient.sendMessage({
        token: token,
        msg: msg,
        onSuccess: function (data) {
                console.log("[ 发送成功 ]");
        },
        onFail: function(data) {
             console.log("[ 发送失败 ]");
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
    var token = "";  //前面获取到的token的值
    var requestId = 360;
    EliteIMClient.cancelRequest({
            token: token,
            requestId: requestId
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```

9、推送满意度
   ```javascript
       var token = "";  //前面获取到的token的值
       var sessionId = '***';  //聊天的sessionId
       var rateId = '***'; //满意度id
       var rateConments = '***'; //满意度内容
       EliteIMClient.sendRating({
            "token": token,
            "sessionId": sessionId,
            "rateId": rateId, 
            "rateConments": rateConments,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
       });
   ```
   

10、关闭聊天
```javascript
    var token = "";  //前面获取到的token的值
    var sessionId = '***';  //聊天的sessionId
    EliteIMClient.closeChat({
            "token": token,
            "sessionId": sessionId,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```


11、发送预发信息： 在排队的过程中发送的信息
```javascript
    var token = "";  //前面获取到的token的值
    var requestId = 0;  //聊天的requestId
    var content = '***';  //发送的预发信息内容
    EliteIMClient.advanceMessage({
            "token": token,
            "requestId": requestId,
            "content": content,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```

12、发送预览信息： 坐席端可以看到客户敲打的内容
```javascript
    var token = "";  //前面获取到的token的值
    var sessionId = 0;  //聊天的sessionId
    var content = '***';  //发送的预发信息内容
    EliteIMClient.previewMessage({
            "token": token,
            "sessionId": sessionId,
            "content": content,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```

13、发送正在输入的提示信息
```javascript
    var token = "";  //前面获取到的token的值
    var sessionId = 0;  //当前聊天的sessionid
    EliteIMClient.sendTypingMessage({
            "token": token,
            "sessionId": sessionId,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```

14、登出
```javascript
    var token = "";  //前面获取到的token的值
    EliteIMClient.loginOut({
            "token": token,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
    
```
15、聊天历史
```javascript
    var token = "";  //前面获取到的token的值
    var fromMessageId = 0;  //前面获取到的token的值
    var count = 5;  //前面获取到的token的值
    var sessionId = '***';  //聊天的sessionId
    EliteIMClient.chatHistory({
            "token": token,
            "sessionId": sessionId,
            "fromMessageId": fromMessageId,
            "count": count,
            onSuccess: function (data) {
                    console.log("[ 发送成功 ]");
            },
            onFail: function(data) {
                 console.log("[ 发送失败 ]");
            }
    });
```

## 聊天插件配置：
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
