# NGS JS 解读 - Softphone.js

softphone.js中主要封装了软电话相关操作的方法，软电话状态逻辑处理等。
与原来的flex中调用基本相似，但最大的差别在于，所有的方法调用，都是异步的，意味着，如果你需要得到方法的返回值，通常是需要在回调方法中获取，而不是方法的直接返回。

###具体方法说明（直接用动态页面中的写法来举例）
动态页面中，可以通过**$softphone**来获取到当前NGS框架的软电话对象。

- **simpleCommonCall** -  原软电话的commonCall
commonCall是软电话的一个非常重要且通用的方法，通常为了不去修改软电话接口方法，新加的一些功能都会通过用这个commonCall来实现，比如转ivr，pds的一些操作，设置event值等等
```javascript
//通过commonCall来调用setevent，给当前event赋值
var param = "customer_guid=" + R.temp.cust_customer_guid + "|event_guid=" + R.temp.event_guid;
$softphone.simpleCommonCall('setevent', 'param');

//通过commonCall来通知软电话项目改变
$softphone.simpleCommonCall('changeproject', 'ProjectId=gzl111');

//通过commonCall发起转ivr, 从项目上抄来，仅供参考，具体写法可能根据CTI不同或者其他什么不同，要咨询jeff
$softphone.simpleCommonCall('ConferenceIVR', 'Dest=7006|nodeCode=1001|idNo=[Temp.CUST_CUSTOMERID]');

//通过commonCall重置作息技能队列。
//从项目上抄来，仅供参考，具体可以咨询jeff或max
$softphone.simpleCommonCall('CTICommonCall', 'CTICallName=RESETAGENTSKILL|AgentID=[Temp.agent_id]|AgentGroup=[Temp.skill_end]|Permanent=0');
```

- **getCallInfo** -  获取callInfo
```javascript
$softphone.getCallInfo(function(callInfo) {
	//这个callInfo是我封装的一个js对象，其中包含了当前电话的很多信息，可以通过getValue方法获取具体信息的值，所有信息如下：
	//var callInfo = {
	//	agentid : softphone.AgentID,
	//	extension : softphone.Extension,
	//	agentstate : softphone.AgentState,
	//	evtdetail : softphone.EvtDetail,
	//	rawattachdata : softphone.CallInfo.RawAttachData,
	//	callstate : softphone.CallInfo.CallState,
	//	ani : softphone.CallInfo.ANI,
	//	dnis : softphone.CallInfo.DNIS,
	//	otherdn : softphone.CallInfo.OtherDN,
	//	starttime : softphone.CallInfo.starttime,
	//	establishtime : softphone.CallInfo.establishtime,
	//	endtime : softphone.CallInfo.endtime,
	//	calltype : softphone.CallInfo.CallType,
	//	attachdatadisplay : softphone.CallInfo.AttachDataDisplay,
	//	istranin : softphone.CallInfo.IsTranIn,
	//	istranout : softphone.CallInfo.IsTranOut,
	//	hanguptype : softphone.CallInfo.HangupType,
	//	callid2 : softphone.CallInfo.CallID2,
	//	callid1 : softphone.CallInfo.CallID1,
	//	callstate2 : softphone.CallInfo.CallState2,
	//	event_guid : event_guid,
	//	allattachdatas : allAttachDatas
	//};
	var eventGuid = callInfo.getValue('event_guid');//获取当前电话的event_guid
	var callType = callInfo.getValue('calltype');//获取当前电话的calltype
	log('eventGuid: ' + eventGuid + '    callType: ' + callType);//日志输出获取到的信息
	R.var.callInfo = callInfo;//把callInfo对象赋值给当前runtime的var.callInfo变量上
	sandbox.docommand('12345');//调用当前组件或者当前动态页面的12345命令组
});
```

- **getAgentStateAndMode** -  获取agentState和agentMode
**agentState包括：**
	//    EAS_UNKNOWN: 0,
    //    EAS_LOGOUT: 1,
    //    EAS_READY: 2,
    //    EAS_NOTREADY: 3,
    //    EAS_BUSY: 4,
    //    EAS_MAX: 5
**agentMode包括：**
	//    EAM_UNKNOWN: 0,
    //    EAM_AUTOIN: 1,
    //    EAM_MANUALIN: 2,
    //    EAM_AUX: 3,
    //    EAM_ACW: 4,
    //    EAM_NOCALLDISCONNECT: 5,
    //    EAM_MAX: 6
```javascript
$softphone.getAgentStateAndMode(function(stateAndMode) {
	//stateAndMode是个json对象，包含了：
	//{
	//    agentState : 1,
	//    agentMode: 4
	//}
	R.var.stateAndMode= stateAndMode;//把stateAndMode对象赋值给当前runtime的var.stateAndMode变量上
	sandbox.docommand('12345');//调用当前组件或者当前动态页面的12345命令组
});
```

- **makeCall** -  拨打电话
```javascript
/**
 * 拨打电话
 * @param number 电话号码
 * @param sData 随路数据, key=value多个用竖线分割
 * @param [callback] 结果回调
 */
$softphone.makeCall('13916312345', 'a=1|b=2', callback);
```

- **releaseCall** - 挂断电话
```javascript
/**
 * 挂断电话
 * @param [callback] 结果回调
 */
$softphone.releaseCall(callback);
```

- **answerCall** - 接起电话
```javascript
/**
 * 接起电话
 * @param [callback] 结果回调
 */
$softphone.answerCall(callback);
```

- **holdCall** - 保持电话
```javascript
/**
 * 保持电话
 * @param [callback] 结果回调
 */
$softphone.holdCall(callback);
```

- **retrieveCall** - 接回电话
```javascript
/**
 * 接回电话
 * @param [callback] 结果回调
 */
$softphone.retrieveCall(callback);
```

- **initConference** - 发出会议
```javascript
/**
 * 发起会议
 * @param number 会议号码
 * @param sData 随路数据，key=value多个用竖线分割
 * @param [callback] 结果回调
 */
$softphone.initConference('13916312345', 'a=1|b=2', callback);
```

- **completeConference** - 完成会议
```javascript
/**
 * 完成会议
 * @param [callback] 结果回调
 */
$softphone.completeConference(callback);
```

- **initTransfer** - 发出转接
```javascript
/**
 * 发起转接
 * @param number 转接号码
 * @param sData 随路数据，key=value多个用竖线分割
 * @param [callback] 结果回调
 */
$softphone.initTransfer('13916312345', 'a=1|b=2', callback);
```

- **completeTransfer** - 完成转接
```javascript
/**
 * 完成转接
 * @param [callback] 结果回调
 */
$softphone.completeConference(callback);
```

- **muteTransfer** - 直接转
```javascript
/**
 * 直接转
 * @param number 转接号码
 * @param sData 随路数据，key=value多个用竖线分割
 * @param [callback] 结果回调
 */
$softphone.muteTransfer('13916312345', 'a=1|b=2', callback);
```

- **singleStepTransfer** - 单步转
```javascript
/**
 * 单步转
 * @param number 转接号码
 * @param sData 随路数据，key=value多个用竖线分割
 * @param [callback] 结果回调
 */
$softphone.singleStepTransfer('13916312345', 'a=1|b=2', callback);
```

- **reconnect** - 取消转接或会议
```javascript
/**
 * 取消转接或会议
 * @param [callback] 结果回调
 */
$softphone.reconnect(callback);
```

- **ready** - 准备好
```javascript
/**
 * 置闲
 * @param [callback] 结果回调
 */
$softphone.ready(callback);
```

- **notReady** - 未准备好
```javascript
/**
 * 置忙
 * @param [callback] 结果回调
 */
$softphone.notReady(callback);
```

- **breaks** - 小修
```javascript
/**
 * 小修
 * @param [callback] 结果回调
 */
$softphone.breaks(callback);
```

- **mute** - 静音
```javascript
/**
 * 静音
 * @param [callback] 结果回调
 */
$softphone.mute(callback);
```

- **unMute** - 取消静音
```javascript
/**
 * 取消静音
 * @param [callback] 结果回调
 */
$softphone.unMute(callback);
```

- **notifyWorkStart** - 通知工作开始
	   "WT-OB", // 外呼
       "WT-IB", // 呼入
       "WT_PDS_OB" // PDS外呼
```javascript
/**
 * 通知软电话工作开始
 * @param workStatus
 * @param [callback] 结果回调
 */
$softphone.notifyWorkStart('WT-OB', callback);
```

- **notifyWorkEnd** - 通知工作结束
```javascript
/**
 * 通知软电话工作结束
 * @param [callback] 结果回调
 */
$softphone.notifyWorkEnd(callback);
```

- **allAttachDatas** - 获取所有随路数据
```javascript
$softphone.allAttachDatas(function(attachDatas) {
	log(attachDatas);//获取返回的随路数据json字符串，日志打印出来看看
	R.var.attachDatas= attachDatas;//把attachDatas对象赋值给当前runtime的var.attachDatas变量上
	sandbox.docommand('12345');//调用当前组件或者当前动态页面的12345命令组
});
```

- **attachData** - 获取指定key的随路数据
```javascript
$softphone.attachData('key', function(attachData) {
	log(attachData);//获取给定key的随路数据值，日志打印出来看看
	R.var.key = attachData;//把attachData对象赋值给当前runtime的var.key变量上
	sandbox.docommand('12345');//调用当前组件或者当前动态页面的12345命令组
});
```

- **getAgentId** - 获取工号
```javascript
$softphone.getAgentId(function(agentId) {
	log(agentId);//获取工号，日志打印出来看看
	R.var.agentId= agentId;//把agentId对象赋值给当前runtime的var.agentId变量上
});
```

- **getExtension** - 获取分机号
```javascript
$softphone.getExtension(function(extension) {
	log(extension);//获取分机号，日志打印出来看看
	R.var.extension= extension;//把extension对象赋值给当前runtime的var.extension变量上
});
```

- **getRecordGuid** - 获取录音guid
```javascript
$softphone.getRecordGuid(function(recordGuid) {
	log(recordGuid);//获取录音guid，日志打印出来看看
	R.var.recordGuid= recordGuid;//把recordGuid对象赋值给当前runtime的var.recordGuid变量上
});
```

- **getRecordInfo** - 获取录音信息
```javascript
$softphone.getRecordInfo(function(recordInfo) {
	log(recordInfo);//包括了3个属性： recordGuid, recordFile, recordFlag
});
```

- **forceAgentLogout** - 强退
```javascript
/**
 * 强退
 * @param dn 分机
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.forceAgentLogout(dn, agentId, callback);
```

- **forceAgentNotReady** - 强忙
```javascript
/**
 * 强忙
 * @param dn 分机
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.forceAgentNotReady(dn, agentId, callback);
```

- **forceAgentReady** - 强闲
```javascript
/**
 * 强闲
 * @param dn 分机
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.forceAgentReady(dn, agentId, callback);
```

- **listen** - 监听
```javascript
/**
 * 监听
 * @param dn 分机
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.listen(dn, agentId, callback);
```

- **intercept** - 强拆
```javascript
/**
 * 强拆
 * @param dn 分机
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.intercept(dn, agentId, callback);
```

- **forceDisconnectAgentByDN** - 强断
```javascript
 /**
 * 强断
 * @param dn 分机号
 * @param [callback] 结果回调
 */
$softphone.forceDisconnectAgentByDN(dn, callback);
```

- **forceHangUp** - 板卡挂断
```javascript
 /**
 * 板卡挂断
 * @param dn 分机号
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.forceHangUp(dn, agentId, callback);
```

- **bargeIn** - 强插
```javascript
/**
 * 强插
 * @param dn 分机号
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.bargeIn(dn, agentId, callback);
```

- **takeOver** - 拦截
```javascript
/**
 * 拦截
 * @param dn 分机号
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.takeOver(dn, agentId, callback);
```

- **coach** - 暗语
```javascript
/**
 * 暗语
 * @param dn 分机号
 * @param agentId 工号
 * @param [callback] 结果回调
 */
$softphone.coach(dn, agentId, callback);
```

- **sendDTMF** - 发送DTMF数据
```javascript
/**
 * 发送DTMF数据
 * @param number 号码
 * @param [callback] 结果回调
 */
$softphone.sendDTMF(number, callback);
```

- **addCustomButton** - 添加自定义软电话按钮
```javascript
/**
 * 添加自定义软电话按钮
 * @param id  按钮id 不可重复
 * @param title 按钮上的文字描述
 * @param handler 点击按钮后的行为
 * @param options (可选) 相关选项: 
 * 		{
 * 			needLogin : true //是否需要登录后才可点击
 * 		}
 */
$softphone.addCustomButton(id, title, function(){
    //执行某些命令
}, options);
```

- **setAutoAnswer** - 设置软电话自动应答属性
```javascript
$softphone.setAutoAnswer(true);
```

- **setAutoPrefix** - 设置软电话自动添加号码头
```javascript
$softphone.setAutoPrefix('021');
```


