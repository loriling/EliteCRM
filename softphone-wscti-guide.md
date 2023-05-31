#基于WSCTI的Softphone开发帮助 

传统软电话都是需要和本地ocx或者dll控件去练本地的cti控件，从而实现软电话与cti的交互。
WSCTI，打破了这种客户端控件模式，彻底做到了客户端无控件，通过websocket与服务端的WSCTIProxy互联，实现软电话的所有功能。

1. [初始化](#init)
2. [软电话调用](#callSoftphone)
3. [接收软电话事件](#handleSoftphoneEvent)
4. [常量说明](#constants)

##<div id="init">初始化</div>
###一.加载相关的js，构造Softphone对象
页面中需要加载的js，这部分公共的js可以直接应用ngs服务下的，也可以复制到自己服务中去引
然后通过require加载softphone相关js，并构造Softphone对象，继承与SoftphonAPI对象
```
<script src="js/lib/jquery-3.1.1.min.js"></script>
<script src="js/lib/guid-1.0.min.js"></script>
<script src="js/lib/jquery.cookies.min.js"></script>
<script src="js/lib/require.min.js"></script>
<script src="js/lib/lib-0.1.3.min.js"></script>
<script src="js/lib/utils-1.0.2.min.js"></script>
<script src="js/lib/jsonsql-1.0.min.js"></script>
<script src="js/lib/md5.min.js"></script>
<script src="js/lib/sm3.min.js"></script>
<script src="js/function.js"></script>
<script src="js/bounddata.min.js"></script>
<script src="js/softphone/wscti/elite.softphone.min.js"></script>

<script type="text/javascript">
	$(function(){
    	var baseStartTime = 0, callTimeTimer;
		// 通过require加载softphone相关js
    	require(['js/softphone/wscti/softphone.api',
            "js/softphone/callstate",
            "js/softphone/agentstate", 
            "js/softphone/agentmode", 
            "js/softphone/ctistatus", 
            "js/softphone/timer", 
            "js/softphone/calltype"], function(SoftphoneAPI, CallState, AgentState, AgentMode, CTIStatus, Timer, CallType) {
    		
    		//定义自己的softphone对象，继承与SoftphoneAPI
    		var Softphone = function(){
    	    	SoftphoneAPI.apply(this, arguments);
    	    	this.currentCallState = 0;
    	        this.currentAgentState = 0;
    	        this.currentAgentMode = 0;
    	        this.currentBreakType = -1;
    	        this.agentConferenced = false;
    	        this.agentTransfered = false;
    	        this.startInitTransfer = false;
    	        this.startInitConference = false;
    	        this.agentLogined = false;
    	        this.notClearNumberWhenECTUNKNOW = false;
    	        this.transferOnDialingHold = false;
    	        this.currentAgentStateText = "";
    	        this.currentCTIStatus = 0;
    	        this.baseStartTime = 0;
    	        this.extension = "";
    	        this.agentId = "";
    	        this.currentCallNumber = "";

    	        this.totalIB = 0;
    	        this.totalOB = 0;
    	        this.totalIBAnswered = 0;
    	        this.totalOBReached = 0;
    	        this.totalIBDuration = 0;
    	        this.totalOBDuration = 0;
    	        this.totalIBDisplay = "";
    	        this.totalOBDisplay = "";
    		}
    		
    		//Softphone继承与SoftphoneAPI
    	    var F = function(){};
    	    F.prototype = SoftphoneAPI.prototype;
    	    Softphone.prototype = new F();
    	    Softphone.prototype.constructor = Softphone;

			......


	});
```
###二.登录ngs
因为软电话需要与数据库交互，因此需要先登录ngs服务，让相关数据接口可以被正常调用
其中参数包括：ngs服务地址，登录名，密码，数据服务名，登录项目，登录组。相关参数可以问项目相关人员。
```
//在合适的地方调用$E.init，进行登录操作
$E.init({
	ngsUrl : opts.ngsUrl || '/ngs',//ngs服务地址
	eliteuser : opts.eliteuser,//ngs登录名
    elitepwd : opts.elitepwd,//ngs登录密码
    jsds : opts.eliteds,//ngs登录数据服务名
    eliteprj : opts.eliteprj,//ngs登录项目id
    elitegrp : opts.elitegrp,//ngs登录组id
    success : function(d) {
    	//登录成功
    },
    fail : function(d) {
    	//登录失败
    }
});
```
###三.初始化软电话并登录
当ngs登录成功后，就可以调用之前构造的软电话对象初始化方法initSoftphone。
当initSoftphone成功后，就可以调用软电话的登录了
软电话的登录中，就会发起与服务端WSCTIProxy的websocket的连接，连接的地址在softphone.ini.js中配置
```
softphone.initSoftphone({}, function(d){
	if (!d) {
		$F.alert("软电话初始化失败");
	} else {
		softphone.login({
			agentId : "5005",//工号
			extension : "8005",//分级
			password : "1234"//密码
		}, function(d) {
			if (!d) {
				alert("登录失败");
			} else {
		      	//登录成功
		      	......
		    }
		});
	}
})
```

至此，软电话整个初始化过程完毕，过程大致分为如下

> 
 - 引入相关js
 - 调用$E.init登录ngs
 - 调用softphone.initSoftphone初始化软电话
 - 调用softphone.login登录软电话


##<div id="callSoftphone">软电话softphone对象方法说明</div>

```
/**
 * 初始化软电话，加载config
 */
initSoftphone : function(params, callback) {
	var a = this;
	softphone.initSoftphone(function(){
		a.initialized = true;
		console.log('初始化WSCTI软电话成功');
		callback && callback(true);
	}, function(){
		console.error('初始化WSCTI软电话失败');
		callback && callback(false);
	}, params);
},
/**
 * 软电话的commonCall
 * @param callName 名字
 * @param callParas 参数
 * @param callback 回调
 */
simpleCommonCall : function(callName, callParas, callback){
	if (!this.softphoneReady()) {
		return false;
	}
	return softphone.commonCall(callName, callParas, callback);
},
/**
 * 获取软电话的callInfo
 * @param callback
 */
getCallInfo : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	var allAttachDatas = softphone.CallInfo.AttachData("^JSON^");
	var callInfo = new CallInfo({
		agentid : softphone.agentID,
		extension : softphone.extension,
		agentstate : softphone.AgentState,
		evtdetail : softphone.EvtDetail,
		rawattachdata : softphone.CallInfo.RawAttachData,
		callstate : softphone.CallInfo.CallState,
		ani : softphone.CallInfo.ANI,
		dnis : softphone.CallInfo.DNIS,
		otherdn : softphone.CallInfo.OtherDN,
		starttime : softphone.CallInfo.starttime,
		establishtime : softphone.CallInfo.establishtime,
		endtime : softphone.CallInfo.endtime,
		calltype : softphone.CallInfo.CallType,
		attachdatadisplay : '',
		istranin : softphone.CallInfo.IsTranIn,
		istranout : softphone.CallInfo.IsTranOut,
		hanguptype : softphone.CallInfo.HangupType,
		callid2 : softphone.CallInfo.CallID2,
		callid1 : softphone.CallInfo.CallID1,
		callstate2 : softphone.CallInfo.CallState2,
		event_guid : softphone.CallInfo.Event_GUID,
		transferflag : softphone.CallInfo.transferflag,
		allattachdatas : allAttachDatas
	});
	if (callback) {
        callback(callInfo);
	}
	return callInfo;
},
/**
 * 获取callStat
 * @param callback
 */
getCallStat : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	var callStat = {};
	if (softphone && softphone.CallStat) {
		callStat = {
			TotalIB : softphone.CallStat.TotalIB,
			TotalOB : softphone.CallStat.TotalOB,
			TotalIBAnswered : softphone.CallStat.TotalIBAnswered,
			TotalOBReached : softphone.CallStat.TotalOBReached,
			TotalIBDuration : softphone.CallStat.TotalIBDuration,
			TotalOBDuration : softphone.CallStat.TotalOBDuration
		};
	}
	if (callback) {
		callback(callStat);
	}
	return callStat;
},
/**
 * 获取agentState和agentMode
 * @param callback
 */
getAgentStateAndMode : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	var agentStateAndMode = {
		agentState : softphone.agentState,
		agentMode : softphone.agentMode,
		reasonCode : softphone.reasonCode
	}
	if (callback) {
		callback(agentStateAndMode);
	}
	return agentStateAndMode;
},
/**
 * 获取transferOnDialingHold属性值
 * @param callback
 */
getTransferOnDialingHold : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	if (callback) {
		callback({
			TransferOnDialingHold : softphone.TransferOnDialingHold
		});
	}
	return softphone.TransferOnDialingHold;
},
/**
 * 获取事件详细值
 * @param callback
 */
getEvtDetail : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	if (callback) {
		callback({
			EvtDetail : softphone.EvtDetail
		});
	}
	return softphone.EvtDetail;
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
	if (!this.softphoneReady()) {
    	if (callback) {
    		callback({ret : "false"});
    	}
		return false;
	}
	softphone.QuitSoftphone();
	if (callback) {
		callback({ret : "success"});
	}
},

/**
 * 拨打电话
 * @param number 电话号码
 * @param sData 随路数据
 * @param callback 回调
 */
makeCall : function(number, sData, callback){
	if (!this.softphoneReady()) {
		return false;
	}
	var ret = softphone.makeCall(number, sData);
	if (callback) {
		callback(ret);
	}
},
/**
 * 挂断电话
 * @param callback
 */
releaseCall : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.releaseCall();
	if (callback) {
		callback();
	}
},
/**
 * 接起电话
 * @param callback
 */
answerCall : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.answerCall();
	if (callback) {
		callback();
	}
},
/**
 * 保持电话
 * @param callback
 */
holdCall : function(callback){
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.holdCall();
	if (callback) {
		callback();
	}
},
/**
 * 接回电话
 * @param callback
 */
retrieveCall : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.retrieveCall();
	if (callback) {
		callback();
	}
},
/**
 * 发起会议
 * @param number 会议号码
 * @param sData 随路数据
 * @param callback
 */
initConference : function(number, sData, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var ret = softphone.initConference(number, sData);
	if (callback) {
		callback(ret);
	}
},
/**
 * 完成会议
 * @param callback
 */
completeConference : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.completeConference();
	if (callback) {
		callback();
	}
},
/**
 * 发起转接
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
initTransfer : function(number, sData, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var ret = softphone.initTransfer(number, sData);
	if (callback) {
		callback(ret);
	}
},
/**
 * 完成转接
 * @param callback
 */
completeTransfer : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.completeTransfer();
	if (callback) {
		callback();
	}
},
/**
 * 直接转
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
muteTransfer : function(number, sData, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.muteTransfer(number, sData);
	if (callback) {
		callback();
	}
},
/**
 * 单步转
 * @param number 转接号码
 * @param sData 随路数据
 * @param callback
 */
singleStepTransfer : function(number, sData, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.singleStepTransfer(number, sData);
	if (callback) {
		callback();
	}
},
/**
 * 取消转接或者会议
 * @param callback
 */
reconnect : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.reconnect();
	if (callback) {
		callback();
	}
},
/**
 * 置闲
 * @param callback
 */
ready : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.agentReady();
},
/**
 * 置忙
 * @param callback
 */
notReady : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.agentNotReady();
},
/**
 * 小休
 * @param reasonCode, callback
 */
notReady2 : function(reasonCode, callback){
	softphone.agentNotReady2(3, reasonCode);
},
/**
 * 小休
 * @param callback
 */
breaks : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.agentNotReady2();
},
/**
 * 静音
 * @param callback
 */
mute : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.mute();
	if (callback) {
		callback();
	}
},
/**
 * 取消静音
 * @param callback
 */
unMute : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.unmute();
	if (callback) {
		callback();
	}
},
/**
 * 通知软电话工作开始
 * @param workStatus
 * @param callback
 */
notifyWorkStart : function(workStatus, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.notifyWorkStart(workStatus);
	if (callback) {
		callback();
	}
},
/**
 * 通知软电话工作结束
 * @param callback
 */
notifyWorkEnd : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.notifyWorkEnd();
	if (callback) {
		callback();
	}
},
/**
 * 获取所有的随路数据
 * @param callback
 */
allAttachDatas : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var allAttachDatas = softphone.CallInfo.AttachData("^JSON^");
	if (callback) {
		callback(allAttachDatas);
	}
	return allAttachDatas;
},
/**
 * 根据给定key获取随路数据
 * @param key
 * @param callback
 */
attachData : function(key, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var attachData = softphone.CallInfo.AttachData(key);
	if (callback) {
		callback(attachData);
	}
	return attachData;
},
/**
 * 获取工号
 * @param callback
 */
getAgentId : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var agentId = softphone.agentId;
	if (callback) {
		callback({
			agentId: agentId
		});
	}
	return agentId;
},
/**
 * 获取分机
 * @param callback
 */
getExtension : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var extension = softphone.dn;
	if (callback) {
		callback({
			extension: extension
		});
	}
	return extension;
},
/**
 * 获取录音guid
 * @param callback
 */
getRecordGuid : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var recordGuid = softphone.Recording_GUID;
	if (callback) {
		callback({
			recordGuid: recordGuid
		});
	}
	return recordGuid;
},

/**
 * 获取录音相关信息
 * @param callback
 */
getRecordInfo : function(callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	var recordGuid = softphone.Recording_GUID,
		recordFile = softphone.Recording_File,
		recordFlag = softphone.RecordingFlag;
	if (callback) {
		callback({
			recordGuid: recordGuid,
			recordFile: recordFile,
			recordFlag: recordFlag
		});
	}
	return recordGuid;
},
/**
 * 强退
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentLogout : function(dn, agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.forceAgentLogout(dn, agentId);
    if (callback) {
    	callback();
    }
},
/**
 * 强忙
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentNotReady : function(dn, agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.forceAgentNotReady(dn, agentId);
    if (callback) {
    	callback();
    }
},
/**
 * 强闲
 * @param dn
 * @param agentId
 * @param callback
 */
forceAgentReady : function(dn, agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.forceAgentReady(dn, agentId);
    if (callback) {
    	callback();
    }
},
/**
 * 监听
 * @param dn
 * @param callback
 */
listen : function (dn, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.listen(dn);
    if (callback) {
    	callback();
    }
},
/**
 * 强拆
 * @param agentId
 * @param callback
 */
intercept : function (agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.intercept(agentId);
    if (callback) {
    	callback();
    }
},
/**
 * 强断
 * @param dn
 * @param callback
 */
forceDisconnectAgentByDN : function (dn, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.forceDisconnectAgentByDN(dn);
    if (callback) {
    	callback();
    }
},
/**
 * 板卡挂断
 * @param callback
 */
forceHangUp : function (callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.forceHangUp();
    if (callback) {
    	callback();
    }
},
/**
 * 强插
 * @param dn
 * @param callback
 */
bargeIn : function (dn, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.bargeIn(dn);
    if (callback) {
    	callback();
    }
},
/**
 * 拦截
 * @param dn
 * @param agentId
 * @param callback
 */
takeOver : function (dn, agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.takeOver(dn, agentId);
    if (callback) {
    	callback();
    }
},
/**
 * 暗语
 * @param dn
 * @param agentId
 * @param callback
 */
coach : function (dn, agentId, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.coach(dn, agentId);
    if (callback) {
    	callback();
    }
},

/**
 * 设置软电话自动应答属性
 * @param autoAnswer
 */
setAutoAnswer : function(autoAnswer) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.autoAnswer = autoAnswer;
},

/**
 * 设置软电话自动添加号码头
 * @param autoPrefix
 */
setAutoPrefix : function(autoPrefix) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.autoPrefix = autoPrefix;
},

/**
 * 发送DTMF数据
 * @param number
 * @param callback
 */
sendDTMF : function(number, callback) {
	if (!this.softphoneReady()) {
		return false;
	}
	softphone.sendDTMF(number);
    if (callback) {
    	callback();
    }
},

/**
 * 给软电话注册事件
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
	if (callback) {
		softphone.addEvtListener('softphone.api', 'OnSCTIEvent', function(data){
			var callInfo = softphone.CallInfo;
			var sfInfo = {
				ani : callInfo.ANI,
				dnis : callInfo.DNIS,
				agentid : softphone.agentID,
				extension : softphone.extension,
				agentstate : softphone.AgentState,
				evtdetail : softphone.EvtDetail,
				calltype : callInfo.CallType,
				attachdatadisplay : ''
			};
			callback({
				type: 'SCTIEvent',
				msg: data,
				sfInfo: sfInfo
			});
		});
    	softphone.addEvtListener('softphone.api', 'OnCTIEvent', function(data){
			callback({
				type: 'CTIEvent',
				data: data
			});
		});
    }
},

/**
 * 软电话是否可用
 * @returns {Boolean}
 */
available : function() {
	if (softphone) {
		return true;
	}
	return false;
},

/**
 * 软电话登录
 * @param param 登录参数，是一个json对象
 *  {
         agentId : agentId, 		// 工号
         extension : extension, 	// 分机
         queue : queue,  		// 队列
         position : position, 	// 位置
         password : password, 	// 密码
         pds: {},                // 预测外呼相关参数
         loginType: loginType    // 签入类型
     }
 * @param callback 登录结果回调，参数result: {true|false}
 */
login : function(param, callback) {
 	softphone.login2(Object.assign(param, {
   		dn: param.extension,
		pds: param.pds || {},
    	callback: callback
    }));
},

/**
 * 软电话登出
 */
logout : function(callback) {
	softphone.agentLogout();
},

/**
 * 获取ini值
 */
getIniInfo : function(callback) {
	var iniInfo = {};
	iniInfo.NeedAgentID = softphone.commonCall("GetiniValue", "[CTI]NeedAgentID");
	iniInfo.NeedPassword = softphone.commonCall("GetiniValue", "[CTI]NeedPassword");
	iniInfo.NeedExtension = softphone.commonCall("GetiniValue", "[CTI]NeedExtension");
	iniInfo.NeedQueue = softphone.commonCall("GetiniValue", "[CTI]NeedQueue");
	iniInfo.NeedQueueExt = softphone.commonCall("GetiniValue", "[CTI]NeedQueueExt");
	iniInfo.QueueMultiSelect = softphone.commonCall("GetiniValue", "[CTI]QueueMultiSelect");
	iniInfo.NeedPosition = softphone.commonCall("GetiniValue", "[CTI]NeedPosition");
	iniInfo.AgentID = softphone.commonCall("GetiniValue", "[CTI]AgentID");
	iniInfo.Extension = softphone.commonCall("GetiniValue", "[CTI]Extension");
	if (callback) {
		callback(iniInfo);
	}
	return iniInfo;
},

/**
 * 判断软电话是否准备好
 */
softphoneReady : function() {
	if (this.available) {
		if (this.initialized) {
    		return true;
    	}
	}
	return false;
}
```

##<div id="handleSoftphoneEvent">接收软电话事件</div>

当我们可以调用软电话方法后，还需要接收软电话发出的事件,事件主要包括了SCTIEvent和CTIEvent
```
/**
 * 给软电话注册事件监听
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
softphone.addEventListener(function(data, ev){
	if(data.type === "SCTIEvent"){
		onSCTIEvent(data.msg);
	}
});
```
这里给出我们自己的例子，监听SCTIEvent后，做响应的界面显示，按钮状态变化等操作：
```
function onSCTIEvent(evt){
    console.info(new Date().format('HH:mm:ss ') + ' ' + 'onSCTIEvent(' + evt.toString() + ')');
    switch(evt.toLowerCase()){
        case 'onlogin':
            $('#agentstate').val('已登录');
            $('.login').attr('disabled', true);
       		$('.logout').removeAttr('disabled');
       		$('.makecall').removeAttr('disabled');
			break;
		case 'onlogout':
			$('#agentstate').val('已登出');
			$('.login').removeAttr('disabled');
            $('.logout').attr('disabled', true);
            $('.makecall').attr('disabled', true);
            $('.ready').attr('disabled', true);
            $('.notready').attr('disabled', true);
            $('.notready2').attr('disabled', true);
            
            $('.answercall').attr('disabled', true);
            $('.makecall').attr('disabled', true);
            $('.releasecall').attr('disabled', true);
            $('.holdcall').attr('disabled', true);
            $('.retrievecall').attr('disabled', true);
            $('.inittransfer').attr('disabled', true);
            $('.iniconference').attr('disabled', true);
            $('.mutetransfer').attr('disabled', true);
            $('.completetransfer').attr('disabled', true);
            $('.completeconference').attr('disabled', true);
            $('.reconnect').attr('disabled', true);
            break;
		case 'agentstatechange':
        	softphone.getAgentStateAndMode(function (agentInfo) {
            	softphone.currentAgentState = agentInfo.agentState;
            	softphone.currentAgentMode = agentInfo.agentMode;
            	softphone.currentReasonCode = agentInfo.reasonCode;
            	var s1 = '',
            	s2 = '';
               	console.info('  agentState = ' + softphone.currentAgentState + ', agentMode = ' + softphone.currentAgentMode + ', reasonCode = ' + softphone.currentReasonCode);
               	if (softphone.currentAgentState == 2) {
               		s1 = '就绪';
                    $('.ready').attr('disabled', true);
               		$('.notready').removeAttr('disabled');
               		$('.notready2').removeAttr('disabled');
               	} else if (softphone.currentAgentState == 3) {
               		$('.ready').removeAttr('disabled');
                    $('.notready').attr('disabled', true);
               		$('.notready2').removeAttr('disabled');
               		if (softphone.currentAgentState == 4) {
               			s1 = 'ACW';
               		} else if (softphone.currentAgentState == 3) {
               			if (softphone.currentReasonCode) {
               				$('.notready').removeAttr('disabled');
                   			s1 = '小休';
                   			s2 = softphone.currentReasonCode;
               			} else {
                   			s1 = '未就绪';
               			}
               		}
               	} else if (softphone.currentAgentState == 1) {
               		s1 = '已登出';
               	} else {
               		s1 = softphone.currentAgentState + '.' + softphone.currentReasonCode;
               	}
               		
            	$('#agentstate').val(s1);
            	$('#agentreasoncode').val(s2);
            });
              	
			break;
		case 'callstatechange':
			var scallstate = '';
            softphone.getCallInfo(function (callInfo) {
                softphone.currentCallState = callInfo.getValue("callState2");
                console.info('  CallState2 = ' + softphone.currentCallState);

               	switch (softphone.currentCallState) {
	                	case 1:
	                		scallstate = '来电振铃';
	                		break;
	                	case 2:
	                		scallstate = '通话中';
	                		startCalltimeTimer();
	                		break;
	                	case 3:
	                		scallstate = '保持';
	                		break;
	                	case 4:
	                		scallstate = '外呼振铃';
	                		break;
	                	case 32:
	                		scallstate = '转接通话';
	                		break;
	                	case 34:
	                		scallstate = '转接振铃';
	                		break;
	                	case 0:
	                		if (callTimeTimer) {
	                			clearInterval(callTimeTimer);
	                		}
	                		break;
	                }
				$('#callstate').val(scallstate);

               	if (softphone.currentCallState == 1) {
                	$('.answercall').removeAttr('disabled');
                } else {
                	 $('.answercall').attr('disabled', true);
                }
                if (softphone.currentCallState == 0) {
                	$('.makecall').removeAttr('disabled');
                } else {
                	$('.makecall').attr('disabled', true);
                }
                if (softphone.currentCallState == 2 || softphone.currentCallState == 4) {
                	$('.releasecall').removeAttr('disabled');
				} else {
                	$('.releasecall').attr('disabled', true);
                }
                if (softphone.currentCallState == 2) {
                	$('.holdcall').removeAttr('disabled');
				} else {
                	$('.holdcall').attr('disabled', true);
                }
                if (softphone.currentCallState == 3) {
                	$('.retrievecall').removeAttr('disabled');
                } else {
                	$('.retrievecall').attr('disabled', true);
                }
                if (softphone.currentCallState == 2) {
                	$('.inittransfer').removeAttr('disabled');
                    $('.mutetransfer').removeAttr('disabled');
                    $('.initconference').removeAttr('disabled');
                    $('.manyidu').removeAttr('disabled');
                    $('.manyidu0').removeAttr('disabled');
                } else {
                	$('.inittransfer').attr('disabled', true);
                    $('.mutetransfer').attr('disabled', true);
                    $('.initconference').attr('disabled', true);
                    $('.manyidu').attr('disabled', true);
                    $('.manyidu0').attr('disabled', true);
                }
                if (softphone.currentCallState == 32) {
                	var transferflag = callInfo.getValue("transferflag");
                    if (transferflag == 1) {
                    	$('.completetransfer').removeAttr('disabled');
                    } else if (transferflag == 2) {
      	            	$('.completeconference').removeAttr('disabled');
                    }
                } else {
                	$('.completetransfer').attr('disabled', true);
                    $('.completeconference').attr('disabled', true);
                }
                if (softphone.currentCallState == 32 || softphone.currentCallState == 34) {
                	$('.reconnect').removeAttr('disabled');
                } else {
                	$('.reconnect').attr('disabled', true);
				}
			});
            break;
		case 'oncallstart':
        	softphone.getCallInfo(function (callInfo) {
            	var callType = callInfo.getValue("callType");
                var ani = callInfo.getValue("ANI");
                var dnis = callInfo.getValue("DNIS");
                $('#ani').val(ani);
              	$('#dnis').val(dnis);
                if (callType == '2') {
                	//来电弹屏
                  	$('#calltype').val('呼入');
                } else {
                	$('#calltype').val('呼出');
                };
                  	
               	$('.ready').attr('disabled', true);
               	$('.notready').attr('disabled', true);
               	$('.notready2').attr('disabled', true);
               	
               	startCalltimeTimer();
           	});
          		
			break;
		case 'oncallestablish':
            //电话接通
            break;
        case 'oncallend':
        	//电话结束
            break;
	}
}
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


