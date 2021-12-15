# NGS 工单3相关操作解析说明

传统的工单操作，主要是基于动态页面中，特殊临时变量的改动，绑定到个个操作上，各个命令组，在相应的时间被调用，通过特殊的临时变量来获取到相关工单的信息，加以修改后，保存等等。
在本文中，会着重介绍，除了传统动态页面操作外，还有的一些直接的js的命令，其中有一些是原有方式的具体实现，有一些是内部使用，有一些是工具方法。了解了这些js，可以帮助你更好的掌握工单的内部操作逻辑，一些工具方法，也可以帮助你更高效的处理你的业务：



## workorder 工单对象 `workorder3.js`

> 工单对象，包含了所有工单相关操作，在动态页面中，可以通过：**$instance.workorder**  获取到当前工单对象。注意：当前动态页面，需要是个工单，而不是普通的动态页面。

- **objective** 当前工单objective的boundData对象，关联的就是objective表。
- **taskBoundData** 当前工单task的boundData对象，关联到elitetask表。
``` javascript
//可以直接从workorder上获取到objective和taskBoundData，并可以通过boundData对象的getValue和setValue方法去赋值或者去取值
//赋值过的字段，最后动态页面保存时候，都会自动记录到数据库中去
$instance.workorder.objective.getValue('objective_guid');
$instance.workorder.objective.setValue('curstep_id_map', 'xxxxx');
$instance.workorder.taskBoundData.getValue('map_id');
$instance.workorder.taskBoundData.setValue('comments', 'xxxxx');
```

- **loadByObjectiveGuid 根据objective_guid加载工单**  此方法主要是在工单内部调用，通常我们动态页面中不需要直接调用此方法，这里主要用来说明，帮助理解
``` javascript
/**
 * 1.load by objective_guid
 * 根据oId打开工单
 * @param params
 *  objectiveGuid 市场活动guid
 *  taskId task的guid
 *  addinId 工单模块id
 *  tabName tab的名字
 *  parameters 参数
 *  notifyId 提醒的guid
 *  container 打开的容器
 *  readonly 是否是只读
 *  continuable 是否可持续的在一个instance中打开
 *  lastInstance 之前的instance
 * @returns {number} -1表示失败，如果成功会返回响应创建出来的instance实例
 */
loadByObjectiveGuid : function (params)
```
动态页面中是通过事件的方式，发出工单3打开的事件，来实现调用上面的方法，上面的方法中，需要先有一个workorder对象，才能去打开工单，而事件的方式，**可以不需要有workorder对象**，直接调用即可：
``` javascript
// 打开工单命令，其实是发出一个工单3打开的事件到project
// 根据objective_guid和taskId打开工单3
$project.event.notify(window.$CONST.EVENT.OPEN_WO3, {
    addinId : 'AddinWO3',
    tabName : 'xxxx',
    oId: '',//需要打开的市场活动guid
    taskId: '', //需要打开的工单任务guid
    notifyId: '', //相关提醒的guid，非必传
    parameter: {}, //带入参数
    dbPool: '', //数据源，非必传
    continuable: true, // 操作结束时候，是否不自动关闭工单
    lastInstance: lastInstance // 上个工单的instance对象（用来实现不关闭工单tab直接加载新工单，仅当上个实例里的动态页面Id和新打开的动态页面Id一致时候）
});
```

- **loadByOtId 根据mapId打开工单** 此方法主要是在工单内部调用，通常我们动态页面中不需要直接调用此方法，这里主要用来说明，帮助理解
``` javascript
/**
 * 2.load by map_id
 * 根据otId打开工单，可能是新建工单(其实是根据mapId打开)
 * 
 * @param params
 * 	mapId 工单的mapId
 *  dupFlag 是否允许重复的标志 1表示允许重复，当前客户已经有此objective时候还是会新建一个，0表示不允许
 *  addinId 模块id
 *  tabName tab名字
 *  parameters 参数
 *  container 工单容器
 *  customerGuid 可以指定某个客户
 *  stepId 指定某个步骤
 *  readonly 只读模式
 *  rolegroupId 子流程id(机构id)
 * @returns {number} -1表示失败，如果成功会返回响应创建出来的instance实例
 */
loadByOtId : function (params)
```
动态页面中是通过另外一个事件：
``` javascript
//根据mapId打开工单命令，可能是新建工单
$project.event.notify(window.$CONST.EVENT.OPEN_WO3, {
    addinId : 'AddinWO3',
    tabName : 'xxxx',
    otId: '', //需要打开的市场活动类型id
    customerGuid: '', //可以直接传递某个客户guid，不传递则按原来逻辑获取客户guid
    stepId: 'xxx', //步骤id，可以从指定步骤新建工单
    parameter: {}, //带入参数
    dbPool: '', //数据源，非必传
    alwaysNew: true, //是否每次都
});
```

- **getCurrentMpaExtends 获取当前工单主结构上的扩展字段** 根据名字获取扩展字段值
``` javascript
/**
 * 获取当前流程的自定义扩展属性值
 * @param name
 */
this.getCurrentMapExtends = function(name)
```

- **getCurrentNodeExtends 获取工单当前结点上的扩展字段** 根据名字获取扩展字段值
``` javascript
/**
 * 获取当前节点的自定义扩展属性值
 * @param name
 */
this.getCurrentNodeExtends = function(name)
```

- **getNodeExtends 获取当前工单指定节点上的扩展字段** 根据名字获取扩展字段值
``` javascript
/**
 * 获取指定节点的自定义扩展属性值
 * @param nodeMapId 
 * @param name
 */
 this.getNodeExtends = function(nodeMapId, name)
```

- **calculateExpectEndTime 根据给定时间来计算期望结束时间** 
``` javascript
/**
 * 根据给定时间来计算期望结束时间
 * @param baseTime 基准时间
 * @param stepId （可选）步骤id，如果不传递默认为当前步骤
 */
this.calculateExpectEndTime = function(baseTime, stepId)
```

- **getNextNode 获取下一个节点** 根据当前工单信息计算出下送的下一个节点
``` javascript
/**
 * 获取下一个节点
 */
 this.getNextNode = function()
```

- **getPrevNode 获取上一个节点id** 根据当前工单path3表信息，获取上一个步骤的步骤id
``` javascript
/**
 * 获取上一个节点id
 */
this.getPrevNode = function()
```

- **setOBFlagByNodeMapId 设置obflag的值** 根据给定的节点id获取obflag扩展字段的值，并赋值给taskBoundData
``` javascript
/**
 * 根据给定的节点id获取obflag扩展字段的值，并赋值task表的obflag字段
 */
this.setOBFlagByNodeMapId = function(nodeMapId)
```


---

##struct 工单结构对象 `struct.js`
> **struct工单结构**：包含了工单主map上的配置信息；所有node，也就是所有步骤上的配置信息；所有链路上的信息；主结构扩展属性信息
> 可以通过工单对象的struct属性获取到结构对象：**$instance.workorder.struct**

- **getStruct 获取当前工单的主结构** 
``` javascript
getStruct : function()

//获取到的struct，就是一个标准的json对象，长成类似如下的样子：
{
	"MAP_ID" : "S6LZUU",
	"MAPNAME" : "项目管理",
	"FOLDER_ID" : "66A30AB0-55FE-B487-3956-CD85BF0E82FF",
	"WORKORDERFRAME_ID" : "8DTGAK",
	"CATEGORY" : "WORDER",
	"PRIORITY_ID" : "NORMAL",
	"EXPIREDPERIODTYPE_ID" : "",
	"EXPIREDPERIOD" : 0,
	"FORECASTUNIT" : "",
	"FORECASTLEN" : 0,
	"CUSTOMERFLAG" : 2,
	"CUSTOMER_GUID" : "1",
	"GROUP_IDA" : "",
	"SHOWTRACEFLAG" : 0,
	"GROUPIDAMOBILE" : "",
	"CANGENERATEOB" : 0,
	"GENERATERATE" : 0,
	"GROUP_IDA2" : "",
	"CREATEDBY" : "B2QOBF",
	"CREATEDDATE" : "2016-07-31 01:00:39",
	"MODIFIEDBY" : "SPDU74",
	"MODIFIEDDATE" : "2016-10-20 14:58:10",
	"OBJECTIVESTATUS" : "",
	"HISTORYFLAG" : 0,
	"DUPLICATE" : 1,
	"DEFAULTHANDLEGROUP_ID" : "",
	"DEFAULTHANDLEBY_ID" : "",
	"DELAYDATE" : 0,
	"EXPECTQUOTA" : 0,
	"CURRENTQUOTA" : 0,
	"PDSFLAG" : 0,
	"STRINGFIELDKST" : "",
	"AVAYAPDSPATH" : "",
	"MAXTRYTIMES" : 0,
	"SCRIPTID" : "",
	"PDSSQL" : "",
	"OBTYPE" : 0,
	"ADDIN_ID" : "",
	"ADDINCOMMENT" : "",
	"ADDINCOMMENT_ID" : "",
	"CHANNELSEQUENCE" : "",
	"TEL_1STARTTIME" : "",
	"TEL_1ENDTIME" : "",
	"TEL_2STARTTIME" : "",
	"TEL_2ENDTIME" : "",
	"MOBILESTARTTIME" : "",
	"MOBILEENDTIME" : "",
	"DOCGROUP_ID" : "",
	"DOCTEMPLATE_ID" : "",
	"OBJECTIVEFOLD_GUID" : "",
	"QUESTIONAREFLAG" : 0,
	"OBJECTIVETYPESTATUS" : 0,
	"CUSTOMERCOUNT" : 0,
	"OBORDERBY" : "",
	"ADDGETPERSONALDATASQL" : "",
	"ADDOBHEADFORMAT" : "",
	"GETDATASQL" : "",
	"OBSHOWCOLS" : 0,
	"ONCERECORD" : 0,
	"MAXCACHEREC" : 0,
	"MINCACHEREC" : 0,
	"PERSONGETFLAG" : 0,
	"PERSONSHOWROWS" : 0,
	"PERSONMAXCACHE" : 0,
	"PERSONMINCACHE" : 0,
	"PERSONGETDATASQL" : "",
	"PEFLAG" : 0,
	"IMPDEFAULTFLAG" : 0,
	"IMPFILTER" : "",
	"MAXFAILURETRYTIMES" : 0,
	"ATTEMPTFLAG" : 0,
	"CALLBYROUND" : "",
	"MPEFLAG" : 0,
	"REASSIGNTARGET" : 0,
	"ADDIN_ID_SAAS" : "",
	"ADDINCOMMENT_ID_SAAS" : "",
	"ADDINCOMMENT_SAAS" : "",
	"EXPIREDDATE" : "",
	"REAL_GROUP_IDA" : "ZIVA22",
	"REAL_GROUP_IDA2" : "",
	"REAL_GROUPIDAMOBILE" : "",
	"nodes" : [{
			"NODEMAP_ID" : "F50ZGG",
			"NODEMAPNAME" : "项目管理",
			"MAP_ID" : "S6LZUU",
			"WORKORDERFRAME_ID" : "8DTGAK",
			"WORKORDERNODE_ID" : "F50ZGG",
			"PRIORITY_ID" : "NORMAL",
			"KEEPSTAYTIME" : 0,
			"NEXTBEGINTIME" : 0,
			"EXPIREDPERIODTYPE_ID" : "M",
			"EXPIREDPERIOD" : 0,
			"FORECASTUNIT" : "",
			"FORECASTLEN" : 0,
			"CANSENDBACK" : 0,
			"CANBACKTOBY" : 0,
			"CANCLOSE" : 1,
			"CANSKIP" : 0,
			"CANMODIFY" : 0,
			"TICKOFFFLAG" : 0,
			"MODULETYPE" : "DYN",
			"DOCTEMPLATE" : "",
			"OPFLAG" : 0,
			"GROUP_IDA" : "",
			"GROUPIDAMOBILE" : "",
			"RETURNOLDSTEP" : "",
			"RETURNGROUPFLAG" : 0,
			"HANDLEGROUP_ID" : "SYSTEM",
			"HANDLEBY_ID" : "",
			"EXPIREDHANDLEGROUP_ID" : "",
			"EXPIREDHANDLEBY_ID" : "",
			"EMAILNOTIFY" : 0,
			"SMSNOTIFY" : 0,
			"NODETOP" : 100,
			"NODELEFT" : 240,
			"FOLLOWROLE" : 0,
			"MULTITASK" : 0,
			"TASKFLAG" : 0,
			"ADVANCEFLAG" : 0,
			"REAL_GROUP_IDA" : "ZIVA22",
			"REAL_MODULETYPE" : "DYN",
			"REAL_GROUPIDAMOBILE" : "",
			"nodeMapExtends" : {}
		},
		...
	],
	"links" : [{
			"LINKMAP_ID" : "XJGT9T",
			"LINKMAPNAME" : "链路",
			"MAP_ID" : "S6LZUU",
			"WORKORDERFRAME_ID" : "8DTGAK",
			"TARGET_ID" : "F50ZGG",
			"SOURCE_ID" : "BEGIN",
			"ALWAYSOPEN" : 1,
			"ADVANCEFLAG" : 0,
			"ADDINVARIANT" : "",
			"COMPARETYPE" : "",
			"VARIANTVALUE" : ""
		},
		...
	],
	"mapExtends" : {}
}
```


- **getOneNode 根据步骤id获取当前Node对象** 
``` javascript
getOneNode : function(stepId)

//获取到的node对象，也是一个标准json，长成类似如下的样子：
{
	"NODEMAP_ID" : "F50ZGG",
	"NODEMAPNAME" : "项目管理",
	"MAP_ID" : "S6LZUU",
	"WORKORDERFRAME_ID" : "8DTGAK",
	"WORKORDERNODE_ID" : "F50ZGG",
	"PRIORITY_ID" : "NORMAL",
	"KEEPSTAYTIME" : 0,
	"NEXTBEGINTIME" : 0,
	"EXPIREDPERIODTYPE_ID" : "M",
	"EXPIREDPERIOD" : 0,
	"FORECASTUNIT" : "",
	"FORECASTLEN" : 0,
	"CANSENDBACK" : 0,
	"CANBACKTOBY" : 0,
	"CANCLOSE" : 1,
	"CANSKIP" : 0,
	"CANMODIFY" : 0,
	"TICKOFFFLAG" : 0,
	"MODULETYPE" : "DYN",
	"DOCTEMPLATE" : "",
	"OPFLAG" : 0,
	"GROUP_IDA" : "",
	"GROUPIDAMOBILE" : "",
	"RETURNOLDSTEP" : "",
	"RETURNGROUPFLAG" : 0,
	"HANDLEGROUP_ID" : "SYSTEM",
	"HANDLEBY_ID" : "",
	"EXPIREDHANDLEGROUP_ID" : "",
	"EXPIREDHANDLEBY_ID" : "",
	"EMAILNOTIFY" : 0,
	"SMSNOTIFY" : 0,
	"NODETOP" : 100,
	"NODELEFT" : 240,
	"FOLLOWROLE" : 0,
	"MULTITASK" : 0,
	"TASKFLAG" : 0,
	"ADVANCEFLAG" : 0,
	"REAL_GROUP_IDA" : "ZIVA22",
	"REAL_MODULETYPE" : "DYN",
	"REAL_GROUPIDAMOBILE" : "",
	"nodeMapExtends" : {}
}
```
- **getAllLinks 获取当前工单的所有Link对象** 就是获取工单主结构上的links属性，是由多个link对象组成的一个数组
``` javascript
getAllLinks : function()

//单个link对象长成如下：
{
	"LINKMAP_ID" : "XJGT9T",
	"LINKMAPNAME" : "链路",
	"MAP_ID" : "S6LZUU",
	"WORKORDERFRAME_ID" : "8DTGAK",
	"TARGET_ID" : "F50ZGG",
	"SOURCE_ID" : "BEGIN",
	"ALWAYSOPEN" : 1,
	"ADVANCEFLAG" : 0,
	"ADDINVARIANT" : "",
	"COMPARETYPE" : "",
	"VARIANTVALUE" : ""
}
```
- **getAllNodes 获取当前工单的所有Node对象**  就是获取工单主结构上的nodes属性，是由多个node对象组成的一个数组
``` javascript
getAllNodes : function()
```

- **getFirstNode 获取当前工单的起始节点对象** 也就是从所有的link中找到 **link.SOURCE_ID** 是`BEGIN`的，然后取这个 **link.TARGET_ID**
``` javascript
getFirstNode : function()
```


---





## advlib中的workorder3类 `advlib.js`

> 在advlib中返回出来的对象，都是可以在动态页面中直接调用的，而这里要说的就是workorder3这个类，动态页面中直接使用 **`WO3`** 即可获取到这个类的对象，从而这个类中的所有方法都就可以被调用了。



- **woload 加载工单** 在使用WO3对象时候，通常需要先加载，也就是给缓存中的objective和elitetask的bounddata装载数据，成功加载后，才可以做后面保存，下送等等操作
``` javascript
/**
 * 加载工单
 * @param oId objective的guid
 * @param taskId elitetask的guid
 * @returns {number}
 */
woload : function(oId, taskId)
```
- **wonew 新建工单** 当然，除了加载，也可以通过新建工单方式，给objective和elitetask的bounddata对象初始化数据
``` javascript
/**
 * 新建工单
 * @param mapId {string} 工单mapid
 * @param customerGuid {string} 客户guid
 * @param [rolegroupId] {string} 机构id
 * @returns {*}
 */
wonew : function(mapId, customerGuid, rolegroupId)
```
- **genSave 生成工单保存bounddata对象 **
```js
/**
* 生成工单保存bounddata对象
* @param comments 备注信息
* @return {BoundData}
*/
genSave: function(comments)
```

- **wosave 工单保存** 加载完工单或者新建完工单，就可以开始调用工单的那些个方法了
``` javascript
/**
 * 工单保存
 * @param comments 备注信息
 * @param dbpool
 * @returns {number}
 */
wosave : function(comments, dbpool)
```
- **genClose 生成结案bounddata对象** 
```js
/**
* 生成结案bounddata对象
* @param comments 备注信息
* @return {BoundData}
*/
genClose: function(comments)
```

- **woclose 工单结案**
``` javascript
/**
 * 工单结案
 * @param comments 备注信息
 * @param dbpool
 * @returns {number}
 */
woclose : function(comments, dbpool)
```
- **genSend 生成下送bounddata对象** 
```js
/**
* 生成下送bounddata对象
* @param params {object} 参数
*        comments 备注信息
*        [toStep] 下送的步骤
*        [toGrp] 下送的组
*        [toBy] 下送的人
*        [operateTaskGuid] 需要操作的taskId
*        [dbPool] 数据源
*        [toRole] 下送的角色
*        [substatus] 子状态
*        [rolegroupId] 机构id
* @return {BoundData}
*/
genSend: function(params)
```

- **wosend 工单下送** 可以加载一次，下送多次，这样可以实现给多个人同时下送工单，相同的objective，不同的elitetask
``` javascript
/**
 * 工单下送
 * @param params {object} 参数
 * 	   comments 备注信息
 * 	   [toStep] 下送的步骤
 * 	   [toGrp] 下送的组
 * 	   [toBy] 下送的人
 * 	   [operateTaskGuid] 需要操作的taskId
 * 	   [dbPool] 数据源
 * 	   [toRole] 下送的角色
 * 	   [substatus] 子状态
 * 	   [rolegroupId] 机构id
 * @returns {number}
 */
wosend : function(params)
```

- **genCd 生成催单bounddata对象**
```js
/**
* 生成催单bounddata对象
* @param comments 备注信息
* @return {BoundData}
*/
genCd : function(comments)
```

- **wocd 工单催单** 
``` javascript
/**
 * 工单催单
 * @param comments 备注信息
 * @param dbpool
 * @returns {number} 1 成功 , -1 失败
 */
wocd : function(comments, dbpool)
```

- **genFailedClose 失败结案bounddata生成** 
```js
/**
* 失败结案bounddata生成
* @param comments 备注信息
* @param objectiveStatus
* @return {BoundData}
*/
genFailedClose: function(comments, objectiveStatus) 
```

- **wofailedclose 工单失败结案**
```js
/**
* 工单失败结案
* @param comments {string} 备注信息
* @param objectiveStatus {string} 工单状态，默认是FAILED，也可以自己传递
* @param dbPool {string} 数据库连接池
*/
wofailedclose : function(comments, objectiveStatus, dbPool)
```

- **genRevoke 撤单bounddata生成** 
```js
/**
* 撤单bounddata生成
* @param params {object}  参数
*    revokeToNodeId 撤单到的节点id
*    comments 备注信息 		
* @return {BoundData}
*/
genRevoke: function(params) 
```

- **worevoke 工单撤单**
```js
/**
* 工单撤单
* @param params {object}  参数
*    revokeToNodeId 撤单到的节点id
*    comments 备注信息 		
* @param dbPool {string} 数据库连接池
* @returns {number} 1 成功 , -1 失败
*/
worevoke : function(params, dbPool)
```


- **woclear 清空工单缓存** 包括了objective和elitetask的bounddata对象，和工单结构
``` javascript
WO3.woclear(mapId, customerGuid);
```

- **wosetobj 给objective的bounddata对象属性赋值** 加载完工单后，需要给对应的objective表某些字段赋值，就调用此方法，然后可以wosave保存到数据库 
``` javascript
/**
 * 给objective表赋值
 * @param field 字段名
 * @param value 字段值
 * @returns {number}
 */
wosetobj : function(field, value){
```

- **wosettask 给elitetask的bounddata对象属性赋值** 除了objective表，当然还有elitetask表 
``` javascript
/**
 * 本来是setinfo，3里面变成settask了
 * 给task表赋值
 * @param field 字段名
 * @param value 字段值
 * @returns {number}
 */
wosettask : function(field, value)
```

- **wosettabc 给tabc表相关的bounddata对象属性赋值** 随便几个tabc表都可以这里赋值
``` javascript
/**
 * 给tabc表赋值
 * @param tbName 表名
 * @param field 字段名
 * @param value 字段值
 * @returns {number}
 */
wosettabc : function(tbName, field, value)
```

- **getStruct 获取当前工单的工单结构对象** 当加载过或者新建过工单后，就可以通过这个方法获取到工单结构对象
``` javascript
/**
 * 获取当前工单的工单结构对象
 */
getStruct : function()
```

- **getNextNode 获取下送到的下个步骤的id** 根据工单结构，当前步骤id，当前substatus获取下一步下送到的步骤id
``` javascript
/**
 * 获取下一步节点的stepId
 * @param struct 工单结构
 * @param currentStepId 当前步骤id
 * @param substatus 子状态id
 * @returns {*}
 */
getNextNode : function(struct, currentStepId, substatus)
```

- **getNodeExtends 获取指定步骤的扩展属性值** 扩展属性是根据node结构，nodeMap结构上综合后得到的值，优先nodeMap的配置，当nodeMap上没有时候才获取node上的配置。这里的mapId不是必传的，当工单加载过或者新建过后，缓存中已经有了当前工单的结构信息，就可以不传递此参数。
``` javascript
/**
 * 根据工单mapid，节点mapId和扩展属性名，来获取节点上的对应扩展属性的值
 * @param nodeMapId 步骤id
 * @param name 扩展属性名
 * @param mapId 工单mapId （非必传）
 * @returns {*}
 */
getNodeExtends : function(nodeMapId, name, mapId)
```

- **setOBFlagByNodeMapId 给指定的步骤id赋值obflag这个扩展属性** 其实就是利用了getNdeExtends方法获取到相关配置的值后，set到当前工单的taskBoundData上去
``` javascript
/**
 * 根据节点mapid设置obflag
 * @param nodeMapId 步骤id
 */
setOBFlagByNodeMapId : function(nodeMapId)
```