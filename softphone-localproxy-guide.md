#基于LocalProxy的Softphone开发帮助 
11/23/2017 2:33:14 PM 

1. [连接websocket server](#connectWS)
2. [软电话调用](#callSoftphone)
3. [接收软电话事件](#handleSoftphoneEvent)
4. [常量说明](#constants)
5. [Demo](#demo)

LocalProxy服务，其实就是一个本地vb写的一个应用程序，这个应用程序做了两件事：
1.自己开启一个websocket server。
2.开启一个ie浏览器，加载一个html，这个HTML中加载了各种ocx本地控件。

这时候，chrome或者其他支持ws的浏览器就可以去连接这个本地的websocket server，从而相互通讯。
注意，因为是通过ws来通讯的，因此所有的方法调用，都是异步的。

##<div id="connectWS">连接websocket server</div>
假设这时候，你已经把LocalProxy安装完毕并且正常启动起来了，可以开始编写自己的客户端代码了。
```
//连接本地8888端口的ws，localproxy默认起的端口是8888，如果需要修改端口，可以从localproxy目录下的LocalProxy.html中修改
var ws = new WebSocket('127.0.0.1:8888');
//收到消息后，做相应的处理
ws.onmessage = function(e) {
	try {
		var data = JSON.parse(e.data);
		console.log('Elite - WebSocket', e.data);
		$E.wsAsyncCallback(data);//这里我们是把消息加到一个数组里，然后
	} catch(ex) {
		console.err('Elite - Websocket', ex);
	}
};
ws.onopen = function(e) {
	$E.ws = ws;
	console.log('Elite - WebSocket', 'WebSocket连接成功');					
};
//处理websocket关闭事件
ws.onclose = function() {
	console.log('Elite - WebSocket', 'WebSocket连接断开');
}
//处理websocket异常事件
ws.onerror = function() {
	console.log('Elite - WebSocket', 'WebSocket连接断开');
}
```
这时候，我们发现需要封装一下websocket相关的发消息，收消息，回调等相关的方法，下面的代码使我们自己的实现，仅供参考，不一定需要这样写，我这里展示出来方便后面的理解：
```
$E = {
	/**
     * Websocket请求回调队列
     * @property wsAsyncQueue
     * @type {array}
     * @private
     */
    wsAsyncQueue : [],
    
    /*
     * websocket回调方法
     */
    wsAsyncCallback : function(e) {
    	var a = this;
    	$.each(a.wsAsyncQueue, function(i, listener) {
    		if (listener.token == e.refid) {
        		listener.callback(e.data, listener.ev);
        		if (! listener.persistence)
        			a.wsAsyncQueue.splice(i, 1);
        		return false;
    		}
    	})
    },
	
    /**
     * 通过websocket发送消息
     * @method wsSend
     * @param module {string} 服务名称
     * @param data {any} 发送的消息对象
     * @param [callback] {callback} 需要请求回调的方法
     * @param [ev] {any} 回调的环境参数
     */
    wsSend : function(module, data, callback, ev) {
		var token = Guid.raw();
    	if ($.isFunction(callback)) {
    		this.wsAsyncQueue.push({
    			token : token,
    			callback : callback,
    			ev: ev
    		});
        	data['module'] = module;
        	data['refid'] = token;
    	}
    	try {
        	if (! this.ws) {
            	if ($.isFunction(callback)) {
            		$E.wsAsyncCallback({
            			refid: token,
            			data: {
            				code: -1,
            				message: 'WebSocket未准备就绪'
            			}
            		})
            	}       		
        	} else {
                console.log('WebSocket - Send', JSON.stringify(data));
                this.ws.send(JSON.stringify(data));
            }
    	} catch (e) {
    		console.err('WebSocket - Send', e);
        	if ($.isFunction(callback)) {
        		$E.wsAsyncCallback({
        			refid: token,
        			data: {
        				code: -1,
        				message: e
        			}
        		})
        	}       		
    	}
    }
}

```

##<div id="callSoftphone">软电话调用</div>
既然chrome和localproxy已经通过websocket相关打通了，那后面就可以在js（chrome中）里调用软电话（localproxy中）了：

```
//先封装两个基础方法simpleActionCall和simpleCommonCall，之后所有的软电话调用，都是基于这两个方法的

/**
 * 基本的动作调用，之后很多具体操作都会调用此基础方法
 * @param action 动作名字
 * @param param 相关参数
 * @param callback 回调函数
 */
simpleActionCall : function(action, param, callback){
    var data = {
        action : action,
        param : param
    };
    $E.wsSend(moduleName, data, function(d){
        if($.isFunction(callback)){
            callback(d);
        }
    }, {});
},
/**
 * 软电话的commonCall
 * @param callName 名字
 * @param callParas 参数
 * @param callback 回调
 */
simpleCommonCall : function(callName, callParas, callback){
    var data = {
        action : "CommonCall",
        callName : callName,
        callParas : callParas
    };
    $E.wsSend(moduleName, data, function(d){
        if($.isFunction(callback)){
            callback(d);
        }
    }, {});
},
```
然后我们可以通过这两个方法去调用其他的软电话相关方法了（具体代码可以从提供的 ***softphone.api.js*** 中获取）：
```
/**
 * 获取软电话的callInfo
 * @param callback
 */
getCallInfo : function(callback){
	this.simpleActionCall('CallInfo', {}, function(d){
		if($.isFunction(callback)){
			var callInfo = new CallInfo(d);
            callback(callInfo);
		}
	});
},
/**
 * 获取callStat
 * @param callback
 */
getCallStat : function(callback){
    this.simpleActionCall("CallStat", {}, callback);
},
/**
 * 获取agentState和agentMode
 * @param callback
 */
getAgentStateAndMode : function(callback){
    this.simpleActionCall("getAgentStateAndMode", {}, callback);
},
/**
 * 获取transferOnDialingHold属性值
 * @param callback
 */
getTransferOnDialingHold : function(callback){
    this.simpleActionCall("getTransferOnDialingHold", {}, callback);
},
/**
 * 获取事件详细值
 * @param callback
 */
getEvtDetail : function(callback){
    this.simpleActionCall("getEvtDetail", {}, callback);
},
/**
 * 获取事件
 * @param callback
 */
getEvent : function(callback){
    this.simpleCommonCall("getevent", "", callback);
},
/**
 * 退出
 * @param callback
 */
quit : function(callback){
    this.simpleActionCall("quit", {}, callback);
},

/**
 * 拨打电话
 * @param number 电话号码
 * @param sData 随路数据
 * @param callback 回调
 */
makeCall : function(number, sData, callback){
    this.simpleActionCall("MakeCall", {
        number : number,
        sDataS : sData || ''
    }, callback);
},
/**
 * 挂断电话
 * @param callback
 */
releaseCall : function(callback){
    this.simpleActionCall("ReleaseCall", {}, callback);
},
/**
 * 接起电话
 * @param callback
 */
answerCall : function(callback){
    this.simpleActionCall("AnswerCall", {}, callback);
},
/**
 * 保持电话
 * @param callback
 */
holdCall : function(callback){
    this.simpleActionCall("HoldCall", {}, callback);
},
/**
 * 接回电话
 * @param callback
 */
retrieveCall : function(callback){
    this.simpleActionCall("RetrieveCall", {}, callback);
},
/**
 * 发起会议
 * @param number 会议号码
 * @param sData 随路数据
 * @param callback
 */
initConference : function(number, sData, callback){
    this.simpleActionCall("InitConference", {
        number : number,
        sDataS : sData?sData:this.sDataS
    }, callback);
},
/**
 * 完成会议
 * @param callback
 */
completeConference : function(callback){
    this.simpleActionCall("CompleteConference", {}, callback);
},
/**
 * 发起转接
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
initTransfer : function(number, sData, callback){
    this.simpleActionCall("InitTransfer", {
        number : number,
        sDataS : sData?sData:this.sDataS
    }, callback);
},
/**
 * 完成转接
 * @param callback
 */
completeTransfer : function(callback){
    this.simpleActionCall("CompleteTransfer", {}, callback);
},
/**
 * 直接转
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
muteTransfer : function(number, sData, callback){
    this.simpleActionCall("MuteTransfer", {
        number : number,
        sDataS : sData?sData:this.sDataS
    }, callback);
},
/**
 * 单步转
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
singleStepTransfer : function(number, sData, callback) {
	softphone.singleStepTransfer(number, sData);
	if (callback) {
		callback();
	}
},
/**
 * 取消转接或者会议
 * @param callback
 */
reconnect : function(callback){
    this.simpleActionCall("Reconnect", {}, callback);
},
/**
 * 置闲
 * @param callback
 */
ready : function(callback){
    this.simpleCommonCall("button.click", "ready", callback);
},
/**
 * 置忙
 * @param callback
 */
notReady : function(callback){
    this.simpleCommonCall("button.click", "notready", callback);
},
/**
 * 小休
 * @param callback
 */
breaks : function(callback){
    this.simpleCommonCall("button.click", "break", callback);
},
/**
 * 静音
 * @param callback
 */
mute : function(callback){
    this.simpleActionCall("Mute", {}, callback);
},
/**
 * 取消静音
 * @param callback
 */
unMute : function(callback){
    this.simpleActionCall("UnMute", {}, callback);
},
/**
 * 通知软电话工作开始
 * @param workStatus
 * @param callback
 */
notifyWorkStart : function(workStatus, callback){
    this.simpleActionCall("NotifyWorkStart", {
        workStatus : workStatus
    }, callback);
},
/**
 * 通知软电话工作结束
 * @param callback
 */
notifyWorkEnd : function(callback){
    this.simpleActionCall("NotifyWorkEnd", {}, callback);
},
/**
 * 获取所有的随路数据
 * @param callback
 */
allAttachDatas : function(callback){
    this.simpleActionCall("AllAttachDatas", {}, callback);
},
/**
 * 根据给定key获取随路数据
 * @param key
 * @param callback
 */
attachData : function(key, callback){
    this.simpleActionCall("attachData", {
        key : key
    }, callback);
},
/**
 * 获取工号
 * @param callback
 */
getAgentId : function(callback){
    this.simpleActionCall("getAgentId", {}, callback);
},
/**
 * 获取分机
 * @param callback
 */
getExtension : function(callback){
    this.simpleActionCall("getExtension", {}, callback);
},
/**
 * 获取录音guid
 * @param callback
 */
getRecordGuid : function(callback){
    this.simpleActionCall("getRecordGuid", {}, callback);
},
/**
 * 强退
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentLogout : function(dn, agentId, callback){
    this.simpleActionCall("ForceAgentLogout", {
        dn : dn,
        agentId : agentId
    }, callback);
},
/**
 * 强忙
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentNotReady : function(dn, agentId, callback){
    this.simpleActionCall("ForceAgentNotReady", {
        dn : dn,
        agentId : agentId
    }, callback);
},
/**
 * 强闲
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentReady : function(dn, agentId, callback){
    this.simpleActionCall("ForceAgentReady", {
        dn : dn,
        agentId : agentId
    }, callback);
},
/**
 * 监听
 * @param dn
 * @param callback
 */
listen : function (dn, callback) {
    this.simpleActionCall("Listen", {
        dn : dn
    }, callback);
},
/**
 * 强拆
 * @param agentId
 * @param callback
 */
intercept : function (agentId, callback){
    this.simpleActionCall("Intercept", {
        agentId : agentId
    }, callback);
},
/**
 * 强断
 * @param dn
 * @param callback
 */
forceDisconnectAgentByDN : function (dn, callback){
    this.simpleActionCall("ForceDisconnectAgentByDN", {
        dn : dn
    }, callback);
},
/**
 * 板卡挂断
 * @param callback
 */
forceHangUp : function (callback) {
    this.simpleActionCall("ForceHangup", {
        dn : dn
    }, callback);
},
/**
 * 强插
 * @param dn
 * @param callback
 */
bargeIn : function (dn, callback){
    this.simpleActionCall("BargeIn", {
        dn : dn
    }, callback);
},
/**
 * 拦截
 * @param dn
 * @param agentId
 * @param callback
 */
takeOver : function (dn, agentId, callback){
    this.simpleActionCall("TakeOver", {
        dn : dn,
        agentId : agentId
    }, callback);
},
/**
 * 暗语
 * @param dn
 * @param agentId
 * @param callback
 */
coach : function (dn, agentId, callback){
    this.simpleActionCall("Coach", {
        dn : dn,
        agentId : agentId
    }, callback);
},
/**
 * 抓屏
 * @param ip
 * @param port
 * @param password
 * @param format
 * @param callback
 */
flexMonitorScreen : function (ip, port, password, format, callback) {
    this.simpleActionCall("flexMonitorScreen", {
        ip : ip,
        port : port,
        password : password,
        format : format
    }, callback);
},

/**
 * 设置软电话自动应答属性
 * @param autoAnswer
 */
setAutoAnswer : function(autoAnswer){
    this.simpleActionCall('setAutoAnswer', {
        autoAnswer : autoAnswer
    });
},

/**
 * 设置软电话自动添加号码头
 * @param autoPrefix
 */
setAutoPrefix : function(autoPrefix){
    this.simpleActionCall('setAutoPrefix', {
    	autoPrefix : autoPrefix
    });
},

/**
 * 发送DTMF数据
 * @param number
 * @param callback
 */
sendDTMF : function(number, callback){
    this.simpleActionCall('SendDTMF', {
        number : number
    }, callback);
},

/**
 * 软电话是否可用
 * @returns {Boolean}
 */
available : function() {
	if ($E.ws) {
		return true;
	}
	return false;
},

/**
 * 软电话登录
 * @param param 登录参数，是一个json对象
 *  {
        agentId : agentId, 		//工号
        extension : extension, 	//分机
        queue : queue,  		//队列
        position : position, 	//位置
        password : password  	//密码
    }
 * @param callback 登录结果回调，参数result: {true|false}
 */
login : function(param, callback) {
	var data = {
        action : "Login",
        param : param
    };
    $E.wsSend(moduleName, data, function(result){
        if (callback) {
        	callback(result);
        }
    }, {});
},

/**
 * 软电话登出
 * @param callback
 */
logout : function(callback) {
	this.simpleActionCall('Logout');
},

/**
 * 获取初始化ini配置信息
 * @param callback
 */
getIniInfo : function(callback) {
	this.simpleActionCall('getIniInfo', {}, callback);
},

/**
 * 发送消息给IVR
 * @param message
 * @param callback
 */
sendIVRMessage : function(message, callback) {
	this.simpleActionCall('SendIVRMessage', {
		message: message
	}, callback);
},

/**
 * 获取小修原因
 * @param callback
 */
getReasonCode : function(callback) {
	this.simpleActionCall('getReasonCode', {}, callback);
}

```
除了这些封装好的方法之外，还有一些通过commonCall来调用的方法，包括了：
```
/**
 * 调用小修，带小修原因，后面的this.value就是具体小修原因代码
 */
softphone.simpleCommonCall("button.click", "notreadycode_" + this.value);
```
其实所有的这些调用软电话的方法，都是通过发送一个websocket消息到localproxy中，localproxy会去处理消息，调用softphone.ocx的方法，然后再把返回的消息发送回来，具体的逻辑可以查看localproxy目录下的 *main.js*

##<div id="handleSoftphoneEvent">接收软电话事件</div>

当我们可以调用软电话方法后，还需要接收软电话发出的事件,事件主要包括了SCTIEvent和CTIEvent
```
/**
 * 给软电话注册vblocalproxy事件
 * @param callback 回调函数 接收参数data
 * 主要分为：
 * 1.data.type === "SCTIEvent" SCTIEvent事件
 *      包括了：data.msg == 'callstatechange' 电话状态变化
 *              data.msg == 'agentstatechange' 坐席状态变化
 *              data.msg == 'OnCallStart' 电话开始
 *              data.msg == 'OnCallEstablish' 电话接起
 *              data.msg.startWith('OnCallAction') 一些电话行为，包括了：MakeCall，InitTransfer，InitConference，MuteTransfer，SingleStepTransfer，PhoneNo
 *              data.msg == 'OnCallEnd' 电话结束
 *              data.msg == '73' notready
 *              data.msg == '74' EVT_DATAUPDATED
 *              data.msg == '23' mute
 *              data.msg == '24' unmute
 *
 * 2.data.type === "CTIEvent" CTIEvent事件
 *      CTIEvent相较于SCTIEvent来说，更底层，更全面，其中的参数也需要自己解析，解析过程可以自己尝试打印出内容查看后自行解析。主要用到了：
 *          var parsedEvent = parseEventDetail(data.EvtDetail);//解析事件详细内容
            var evtObj = parsedEvent.evtObj;//事件内容
            var attachObj = parsedEvent.attachObj;//随路数据
 *          evtObj["event"] == "9"  partydeleted
 *          evtObj["event"] == "22" partychanged
 *
 * 3.data.type === 'inited'
 *      电话初始化成功后事件
 */
addEventListener : function(callback) {
	var softphone = this;
	//注册softphone事件 callback
    $E.wsAsyncQueue.push({
        token : 'WS.SOFTPHONE',
        persistence : true,	// 持久化回调方法，不需要删除
        callback : function(data, ev){
            if (callback) {
            	callback(data, ev);
            }
        }
    });
},
```
这里给出我们自己的例子，监听SCTIEvent后，做响应的界面显示，按钮状态变化等操作：
```
/**
 * 软电话本身处理SCTI事件
 * @param type
 */
Softphone.prototype.handleSCTIEvents = function (type) {
    var softphone = this;
    try {
        if (type == "callstatechange" || type == "agentstatechange") {
            if (type == "callstatechange") {
                softphone.getCallInfo(function (callInfo) {
                    softphone.currentCallState = callInfo.getValue("callState2");
                    $F.log("Softphone.callstatechange", "Start to handle callState " + softphone.currentCallState);
                    if (softphone.currentCallState == CallState.ECT_TALKING) {
                        if (!softphone.callTimeTimer.running) {
                            softphone.baseStartTime = new Date().getTime();
                            softphone.callTimeTimer.start();
                        }
                        if (softphone.agentConferenced) {
                            softphone.agentConferenced = false;
                        }
                        if (softphone.agentTransfered) {
                            softphone.agentTransfered = false;
                        }
                    } else if (softphone.currentCallState == CallState.ECT_UNKNOWN) {
                        try {
                            softphone.callTimeTimer.stop();
                            softphone.agentConferenced = false;
                            softphone.agentTransfered = false;
                            if (!softphone.notClearNumberWhenECTUNKNOW) {
                                $(BUTTONS.ANI).val('');
                            }
                        } catch (e) {
                            $F.err("Softphone - callstatechange", "Handle callState.ECT_UNKNOWN error:" + e);
                        }
                    } else if (softphone.currentCallState == CallState.ECT_HOLD) {

                    } else if (softphone.currentCallState == CallState.ECT_TALKINGHOLD) {
                        if (softphone.startInitTransfer) {
                            softphone.agentTransfered = true;
                        } else if (softphone.startInitConference) {
                            softphone.agentConferenced = true;
                        }
                    }
                    if (softphone.currentCallState == CallState.ECT_RINGING ||
                        softphone.currentCallState == CallState.ECT_DIALING ||
                        softphone.currentCallState == CallState.ECT_DIALINGHOLD) {
                        $("#c_softphone .sp_info .calltime").html("<span style='color: #FBF230'>00:00</span>");
                        if (!softphone.hideCallNumber) {
                            $("#c_softphone .sp_callnum").html("<span style='color: #FFFFCC;font-weight: bold'>" + softphone.currentCallNumber + "</span>");
                            softphone.callNumBlinkTimer.start();
                        }
                    } else {
                        if (!softphone.hideCallNumber) {
                            softphone.callNumBlinkTimer.stop();
                            $("#c_softphone .sp_callnum").html("<span style='color: #FFFFCC;font-weight: bold'>" + softphone.currentCallNumber + "</span>");
                        }
                    }
                    if (softphone.currentCallState == CallState.ECT_RINGING) {
                        softphone.answerBtnBlinkTimer.start();
                    } else {
                        softphone.answerBtnBlinkTimer.stop();
                    }
                    softphone.setAgentStateText();
                    softphone.setBtnStatus();
                    $F.log("Softphone.callstatechange", "Handle callState " + softphone.currentCallState + " done.");
                });
            } else if (type == "agentstatechange") {
                softphone.getAgentStateAndMode(function (agentInfo) {
                    softphone.currentAgentState = agentInfo.agentState;
                    softphone.currentAgentMode = agentInfo.agentMode;
                    $F.log("Softphone.agentstatechange", "Start to handle agentState " + softphone.currentAgentState + " and agentMode " + softphone.currentAgentMode);
                    if (softphone.currentAgentState != AgentState.EAS_LOGOUT) {
                        softphone.setAgentLogined(true);
                        var showExtension = softphone.extension;
                        if (softphone.getCurrentProject().getParam("EXTSHD") == "1") {
                            var extensionSheld = "";
                            for (var i = 0; i < softphone.extension.length; i++) {
                                extensionSheld += "*";
                            }
                            showExtension = extensionSheld;
                        }
                        $("#c_softphone .sp_info .extension").html(showExtension);
                        $("#c_softphone .sp_info .agent_id").html(softphone.agentId);
                    } else {
                        softphone.setAgentLogined(false);
                        $("#c_softphone .sp_info .extension").text("");
                        $("#c_softphone .sp_info .agent_id").text("");
                    }
                    softphone.setAgentStateText();
                    softphone.setBtnStatus();
                    $F.log("Softphone.agentstatechange", "Handle agentState " + softphone.currentAgentState + " and agentMode " + softphone.currentAgentMode + " done.");
                });
            }

        } else if (type == "OnCallStart") {
            softphone.getCallInfo(function (callInfo) {
                var proj = softphone.getCurrentProject();
                var callType = callInfo.getValue("callType");
                if (callType == CallType.INBOUND) {
                    var ani = callInfo.getValue("ANI");
                    if (!$F.isNull(ani)) {
                        softphone.currentCallNumber = $F.shieldNumber(ani,
                            proj.getParam("TELWLD"), proj.getParam("TELSHD"), proj.getParam("TELWLF"), proj.getParam("TELSHR"));
                        $F.log("Softphone.OnCallStart", "set currentCallNumber by ani:" + softphone.currentCallNumber);
                    }
                } else if (callType == CallType.OUTBOUND || callType == CallType.PDS_OUTBOUND) {
                    var dnis = callInfo.getValue("DNIS");
                    if (!$F.isNull(dnis)) {
                        softphone.currentCallNumber = $F.shieldNumber(dnis,
                            proj.getParam("TELWLD"), proj.getParam("TELSHD"), proj.getParam("TELWLF"), proj.getParam("TELSHR"));
                        $F.log("Softphone.OnCallStart", "set currentCallNumber by dnis:" + softphone.currentCallNumber);
                    }
                }
            });
            softphone.callTimeTimer.start();
            softphone.baseStartTime = new Date().getTime();
        } else if (type == "OnCallEstablish") {
            softphone.callTimeTimer.reset();
        } else if (type.startWith("OnCallAction")) {
            var arr = type.split(",");
            var actionType = arr[1];
            var dest = arr[2];
            if (actionType == "MakeCall" || actionType == "InitTransfer" || actionType == "InitConference" ||
                actionType == "MuteTransfer" || actionType == "SingleStepTransfer" || actionType == "PhoneNo") {
                softphone.startInitTransfer = false;
                softphone.startInitConference = false;
                if (actionType == "InitTransfer") {
                    softphone.startInitTransfer = true;
                } else if (actionType == "InitConference") {
                    softphone.startInitConference = true;
                }
                if (actionType == 'PhoneNo') {
                    $(BUTTONS.ANI).val(dest);
                }
            }
        } else if (type == "OnCallEnd") {
            softphone.getCallStat(function (callStat) {
                softphone.refreshCallStat(callStat);
            });
            //电话结束时候，清空静音状态
            if (softphone.isMute) {
                softphone.isMute = false;
                hideBtn($(BUTTONS.UNMUTE));
                $(BUTTONS.MUTE).show();
            }
        } else if (type == "73") {//notready
            //if(softphone.currentAgentMode == AgentMode.EAM_AUX){
            softphone.getEvtDetail(function (data) {
                $F.log("Softphone.handleSCTIEvents", "Evt detail: " + data.EvtDetail);
                if (!$F.isNull(data.EvtDetail)) {
                    var details = data.EvtDetail.split("\|");
                    if ($.isArray(details) && details.length > 0) {
                        details.forEach(function (detail) {
                        	if (detail.toLowerCase().indexOf("reasoncode") > -1) {
                                var reasonCode = detail.substr(detail.indexOf("=") + 1);
                                var auxDesc = softphone.auxMap[reasonCode];
                                $F.log("Softphone.handleSCTIEvents", "Find aux : " + auxDesc + " with reason code " + reasonCode);
                                softphone.setBreakTypeLabelText($F.parseEmpty(auxDesc));
                            }
                        });
                    }
                }
            });
            //}
        } else if (type == "74") {//EVT_DATAUPDATED
            softphone.getEvtDetail(function (data) {
                $F.log("Softphone.EVT_DATAUPDATED", "SCTIEventHandler 74 event detail: " + data.EvtDetail);
                var parsedEvent = parseEventDetail(data.EvtDetail);
                var evtObj = parsedEvent.evtObj;
                var attachObj = parsedEvent.attachObj;
                if (evtObj.hasOwnProperty("event")) {
                    if (evtObj["event"] == "74") {
                        softphone.getCallInfo(function (callInfo) {
                            var callType = callInfo.getValue("callType");
                            if (callType == CallType.INBOUND) {
                                var newANI = attachObj["ani"];
                                if (!$F.isEmpty(newANI)) {
                                    if (!softphone.hideCallNumber) {
                                        $("#c_softphone .sp_callnum").html("<span style='color: #FFFFCC;font-weight: bold'>" + newANI + "</span>");
                                    }
                                }
                            }
                        });
                    }
                }
            });
        } else if (type == "23") {//mute
            softphone.isMute = true;
            hideBtn($(BUTTONS.MUTE));
            $(BUTTONS.UNMUTE).show();
        } else if (type == "24") {//unmute
            softphone.isMute = false;
            hideBtn($(BUTTONS.UNMUTE))
            $(BUTTONS.MUTE).show();
        }
    } catch (e) {
        $F.err("Softphone - handleSCTIEvents", e.message);
    }
};
```


##<div id="constants">常量说明</div>
agentMode坐席模式

```
EAM_UNKNOWN: 0,
EAM_AUTOIN: 1,
EAM_MANUALIN: 2,
EAM_AUX: 3,
EAM_ACW: 4,
EAM_NOCALLDISCONNECT: 5,
EAM_MAX: 6

```
agentState坐席状态
```
EAS_UNKNOWN: 0,
EAS_LOGOUT: 1,
EAS_READY: 2,
EAS_NOTREADY: 3,
EAS_BUSY: 4,
EAS_MAX: 5
```
callState电话状态
```
ECT_UNKNOWN: 0,
ECT_RINGING: 1,
ECT_TALKING: 2,
ECT_HOLD: 3,
ECT_DIALING: 4,
ECT_TALKINGHOLD: 32,
ECT_DIALINGHOLD: 34
```
calltype电话类型
```
UNKNOW : 0,
INBOUND : 2,
OUTBOUND : 3,
SIMULAT_INBOUND : 4,
PDS_OUTBOUND : 6
```
ctiStatus交换机状态
```
LOGIN : 1,
READY: 2,
AUX: 3,
ACW: 4,
BUSY: 5,
LOGOUT: 6,
RINGING: 20,
HOLD: 22,
DIALING: 23,
CONSULTING: 25,
OFFHOOK: 24
```
工作类型
```
WT_OB : "WT-OB",//外呼工作
WT_IB : "WT-IB",//呼入工作
WT_PDS_OB : "WT_PDS_OB"//预览外呼工作
```

##<div id="demo">Demo</div>
[示例下载](attachments/softphone-localproxy-demo.zip "示例下载")  
附件解压后：
> 1.开启localproxy，保证软电话加载正常
> 2.可能要修改localproxy-api-demo.html中的localproxy的websocket地址：localWsAddr: 'ws://127.0.0.1:8888',
> 3.本地打开localproxy-api-demo.html，可以看到console中提示正常连接到localproxy的websocket
> 4.在界面上可以对软电话做相关操作了
