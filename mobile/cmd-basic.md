#基础命令集

####这些命令都是直接绑定在动态页面沙箱内的，在动态页面脚本环境中，都可以直接调用运行。

1. <span id="log">**log**</span>: 日志输出到控制台
```
/**
 * 日志输出
 * 输出内容到控制台，会拼接上日志头：'Cmd - log'
 * 当在expo模式或者开启了debug的时候，才会输出
 * 
 * @param e {string} 日志内容
 * @param [obj] {any} 日志内容对象
 */
log(e, obj)
```

2. <span id="out">**out**</span>： 日志输出到控制台，有红色提示
```
/**
 * 在控制台打印一条日志记录, 只有在expo模式下才回输出，当开启了debug模式时候，输出信息同时还会写入到本地文件中
 * 日志会由一个红色的[DEBUG]作为日志头
 * 
 * @param e {string} 日志消息
 * @param [obj] {any} 在控制台上进行的对象输出
 */
out(e, obj)
```

3. <span id="msg">**msg**</span>： 弹出消息提示
```
/**
 * 弹出一个消息框，使用"提示"作为消息框的默认title
 * 
 * @param e {string} 消息内容
 * @param [callback] {string|function} 回调命令组ID或者回调方法
 * @param [state] integer 类型, 不同state决定了消息的不同title, 0=消息, 1=信息, 2=警告, 3=提示，
 */
msg(e, callback, state)
```

4. <span id="info">**info**</span>： 对msg的进一步封装，弹出信息提示
```
/**
 * 弹出一个提示框，使用"信息"作为消息框的title
 * 
 * @param e {string} 消息内容
 * @param [callback] {string|function} 回调命令组ID或者回调方法
 */
info(e, callback)
```

5. <span id="warn">**warn**</span>： 对msg的进一步封装，弹出警告提示
```
/**
 * 弹出一个警示框，使用"警告"作为消息框的title
 * 
 * @param e {string} 消息内容
 * @param [callback] {string|function} 回调命令组ID或者回调方法
 */
warn(e, callback) 
```

6. <span id="alert">**alert**</span>： 对msg的进一步封装，弹出提示提示
```
/**
 * 弹出一个错误框，使用"提示"作为消息框的title
 * 
 * @param e {string} 消息内容
 * @param [callback] {string|function} 回调命令组ID或者回调方法
 */
alert(e, callback)
```

7. <span id="confirm">**confirm**</span>： 弹出确认对话框
```
/**
 * 弹出一个询问框
 * 
 * @param e {string} 询问内容
 * @param [submit] {callback | string} 确定回调方法或者命令号
 * @param [cancel] {callback | string} 取消回调方法或者命令号
 * @param [options] {Object} 选项
 * 	options = {
 * 		okBtn: '确定按钮文字',
 * 		cancelBtn: '取消按钮文字'
 * 	}
 */
confirm(e, submit, cancel, options)
```

8. <span id="notify">**notify**</span>： 轻提示，类似android中的Toast
```
/**
 * 轻提示
 * @method notify
 * @param message {string} 消息内容
 * @param [state] {number} 消息的类型，默认普通消息  0:Message, 1:Success, 2:Fail, 3:Smile, 4:Sad, 5:Info, 6:Stop
 * @param [duration] {number} 持续时长，单位毫秒 默认 2000ms
 * @param [position] {string} 显示位置，默认是center   'top', 'bottom', 'center'
 */
notify(message, state, duration, position)
```

9. **guid（newguid）**: 生成一个guid
```
/**
 * 创建一个36位的GUID
 * @return {string}
 */
guid()
```

10. <span id="id6">**id6**</span>: 生成一个6位伪id
```
/**
 * 创建一个6位的伪ID
 * @param [prefix] {string} 伪ID前缀（占位合并，不得多于2位）
 * @return {string}
 */
id6(prefix) 
```

11. <span id="now">**now**</span>: 获取数据库时间
区别于pc端命令，此命令不在直接返回时间的值字符串，而是返回Promise对象，通过Promise.then方法传递回调，从回调中获取值
```
/**
 * 获取当前数据库时间 yyyy-MM-dd HH:mm:ss
 * @return {Promise}
 */
now()
例如：
now().then(function(ret){
    out(ret);//这里的ret才是获取到的日期字符串， 2019-02-02 14:37:22
});
```

12. <span id="getYear">**getYear**</span>： 获取当前数据库时间的年份
```
/**
 * 获取当前年
 * @return {Promise}
 */
getYear() 
例如：
getYear().then(function(year){
    out(year);//这里的year才是获取到的年， 2019
});
```

13. <span id="getMonth">**getMonth**</span>： 获取当前数据库时间的月份
```
/**
 * 获取当前月
 * @return {Promise}
 */
getMonth() 
例如：
getMonth().then(function(month){
    out(month);//这里的month才是获取到的月， 2
});
```

14. <span id="getDate">**getDate**</span>： 获取当前数据库时间的日
```
/**
 * 获取当前日
 * @return {Promise}
 */
getDate() 
例如：
getDate().then(function(date){
    out(date);//这里的date才是获取到的日， 2
});
```

15. <span id="getDay">**getDay**</span>： 获取当前数据库时间的星期
```
/**
 * 获取当前星期几
 * @return {Promise}
 */
getDay() 
例如：
getDay().then(function(day){
    out(day);//这里的day才是获取到的星期几， 6
});
```

16. <span id="getHours">**getHours**</span>： 获取当前数据库时间的小时
```
/**
 * 获取当前小时
 * @return {Promise}
 */
getHours() 
例如：
getHours().then(function(hours){
    out(hours);//这里的hours才是获取到的小时
});
```

17. <span id="getMinutes">**getMinutes**</span>： 获取当前数据库时间的分钟
```
/**
 * 获取当前分钟
 * @return {Promise}
 */
getMinutes() 
例如：
getMinutes().then(function(minutes){
    out(minutes);//这里的minutes才是获取到的分钟
});
```

18. <span id="getSeconds">**getSeconds**</span>： 获取当前数据库时间的秒钟
```
/**
 * 获取当前秒钟
 * @return {Promise}
 */
getSeconds() 
例如：
getSeconds().then(function(seconds){
    out(seconds);//这里的seconds才是获取到的秒
});
```

19. <span id="getMilliseconds">**getMilliseconds**</span>： 获取当前数据库时间的分钟，感觉不会有人用这方法
```
/**
 * 获取当前毫秒
 * @return {Promise}
 */
getMilliseconds() 
例如：
getMilliseconds().then(function(ms){
    out(ms);//这里的ms才是获取到的毫秒
});
```

20. <span id="getTime">**getTime**</span>： 获取当前数据库时间的时间戳（1970年1月1日0点0分0秒到现在的毫秒数）
```
/**
 * 获取当前毫秒
 * @return {Promise}
 */
getTime() 
例如：
getTime().then(function(time){
    out(time);//这里的time才是获取到的时间戳
});
```

21. **dateDiff（datediff）**: 两个时间比较
```
/**
 * 两个日期比较
 * @param d1 时间1
 * @param d2 时间2
 * @return {number} 返回的是两个时间差几秒的秒数
 */
dateDiff(d1, d2)
```

22. **dataAdd（dataadd）**: 
```
/**
 * 在日期时间上增加值
 * @param unit {string}	增加值的类型，包括： year=年份；month=月份；day=月历日；hour=小时；minute=分钟；second=秒
 * @param d {string} 日期时间的字符串
 * @param len {number} 增加值
 * @return {string} 增加值后的日期时间
 */
dateAdd(unit, d, len)
```

23. <span id="dbtype">**dbtype**</span>: 获取指定数据库类型
区别于pc端命令，此命令不在直接返回数据库类型字符串，而是返回Promise对象，通过Promise.then方法传递回调，从回调中获取值
```
/**
 * 获取当前数据库类型
 * @param dbPool
 * @return {Promise}
 */
dbtype(dbPool)
例如：
dbtype('').then(function(t){
    out(t);//这里的t才是获取到的数据库类型
});
```

24. **cmd（doCommand）**: 执行一个命令组
区别于pc端命令，手机端的cmd命令有个第三个参数callback，命令返回的结果只有在callback中才能获取
```
/**
 * 执行命令组
 * @param cmdId {string | number} 命令号
 * @param ev {object} 运行环境
 * @param callback {function} 回调函数
 */
cmd(cmdId, ev, callback) 
例如：
cmd('P00001', {}, function(ret){
	out(ret);//这里可以获取到P00001命令的返回值ret
});
```

25. <span id="pcmd">**pcmd**</span>: 执行全局命令
区别于pc端命令，手机端的执行全局命令多了一个callback入参，用来接收结果的回调
```
/**
 * 执行全局命令
 * @param cmdId 全局命令id
 * @param callback 结果回调
 */
pcmd(cmdId, callback)
例如：
pcmd('G00001', {}, function(ret){
	out(ret);//这里可以获取到G00001全局命令的返回值ret
});
```

26. **ctl（getCtl）**： 获取某个ngs控件
```
/**
 * 获取当前运行环境下的某个NGS控件
 * @return {object} NGS控件对象
 */
ctl(ctlId)
```

1. <span id="val">**val**</span>: 给变量赋值或者取值
```
/**
 * 为变量赋值或者从变量取值
 * @param propertyName {string} 变量名
 * @param [value] {any} 赋值。可选，未定义时代表取值
 * @return {any}
 */
val(propertyName, value) 
```

1. <span id="setValue">**setValue**</span>： 为变量赋值
```
/**
 * 为变量赋值
 * @param propertyName {string} 变量名
 * @param value 值
 */
setValue(propertyName, value)
```

1. <span id="getValue">**getValue**</span>: 从变量取值
```
/**
 * 从变量取值
 * @param propertyName {string} 变量名
 * @return {any}
 */
getValue(propertyName) 
```

1. <span id="iif">**iif**</span>: 判断是否满足表达式，类似三元表达式
```
/**
 * 判断是否满足表达式
 * @param e 表达式
 * @param t 满足时候的值
 * @param f 不满足时候的值
 * @return {*}
 */
iif(e, t, f) 
```

1. <span id="refresh">**refresh**</span>： 通知系统即可刷新变量到UI
```
/**
 * 通知系统即可刷新变量到UI
 * @param [propertyName] {string} 变量名。可选，为空时刷新全部已经修改的变量
 * @param [data] {*} 刷新成的值
 */
refresh(propertyName, data)
```
 
1. <span id="clearDsl">**clearDsl**</span>: 清除DSL片段
```
/**
 * 清除DSL片段
 * @param [dslKey] {string} 被删除的DSL片段KEY，默认为空，删除全部DSL片段
 * @param [dslNum] {number} 被删除的DSL片段KEY的序列，默认为空，删除指定DSL片段的所有序列
 */
clearDsl(dslKey, dslNum)
```

1. <span id="showDsl">**showDsl**</span>: 展示DSL片段内容到控制台
```
/**
 * 展示DSL片段内容
 * @param dslKey {string} 要展示的DSL片段KEY
 */
showDsl(dslKey)
```

1. <span id="transArray">**transArray**</span>：对数组变量进行交叉
```
/**
 * 对数组变量进行交叉
 * @param propertyName {string} 变量名
 * @param fixCols {array} 固定列号，下标为1。例如[1,2,3]
 * @param transCol {integer} 交叉列号， 下标为1
 * @param dataCol {integer} 数据列号，下标为1
 * @return {array} 交叉完成后的数组
 */
transArray(propertyName, fixCols, transCol, dataCol) 
```

1. <span id="storeMethod">**storeMethod**</span>: 在当前实例或者runtime中创建一个内置方法
```
/**
 * 在当前实例或者runtime中创建一个内置方法（实例内有效）
 * @param name {string} 方法名
 * @param callback {function} 方法实体
 * @param runtimeScope {boolean} 是否创建与runtime环境内，默认创建与实例中
 */
storeMethod(name, callback, runtimeScope) 
```

1. <span id="doMethod">**doMethod**</span>: 执行某个在实例已创建的方法
```
/**
 * 执行某个在实例已创建的方法
 * @param name {string} 方法名
 * @param arguments {...} 入参
 * @return {any} 执行结果
 */
doMethod(name)
```

1. <span id="doMethodRT">**doMethodRT**</span>: 执行某个在runtime环境中已创建的方法
```
/**
 * 执行某个在runtime环境中已创建的方法
 * @param namespace 命名空间
 * @param name {string} 方法名
 * @param arguments {...} 入参
 * @return {any} 执行结果
 */
doMethodRT(namespace, name) 
```

1. <span id="setInstanceItem">**setInstanceItem**</span>： 向实例环境写入一个数据
```
/**
 * 向实例环境写入一个数据，用于同页面内数据交互
 * @method setInstanceItem
 * @param name {string} 数据名
 * @param item {any} 数据对象
 */
setInstanceItem(name, item)
```

1. <span id="getInstanceItem">**getInstanceItem**</span>： 从实例环境读取一个数据
```
/**
 * 从实例环境读取一个数据
 * @method getInstanceItem
 * @param name {string} 数据名
 * @return {any} 数据对象
 */
getInstanceItem(name)
```

1. <span id="removeInstanceItem">**removeInstanceItem**</span>： 从实例环境删除一个数据
```
/**
 * 从实例环境删除一个数据
 * @method removeInstanceItem
 * @param name {string} 数据名
 */
removeInstanceItem (name)
```

1. <span id="setProjectItem">**setProjectItem**</span>: 向项目环境写入一个数据
```
/**
 * 向项目环境写入一个数据，用于跨页面数据交互
 * @method setProjectItem
 * @param name {string} 数据名
 * @param item {any} 数据对象
 */
setProjectItem(name, item)
```

1. <span id="getProjectItem">**getProjectItem**</span>: 从项目环境读取一个数据
```
/**
 * 从项目环境读取一个数据
 * @method getProjectItem
 * @param name {string} 数据名
 * @return {any} 数据对象
 */
getProjectItem(name)
```

1. <span id="removeProjectItem">**removeProjectItem**</span>: 从项目环境删除一个数据
```
/**
 * 从项目环境删除一个数据
 * @method removeProjectItem
 * @param name {string} 数据名
 */
removeProjectItem(name)
```

1. <span id="setGlobalItem">**setGlobalItem**</span>: 向NGS全局环境写入一个数据
```
/**
 * 向NGS全局环境写入一个数据，用于框架核心数据交互
 * @method setGlobalItem
 * @param name {string} 数据名
 * @param item {any} 数据对象
 */
setGlobalItem(name, item)
```

1. <span id="setGlobalItem">**setGlobalItem**</span>: 从NGS全局环境读取一个数据
```
/**
 * 从NGS全局环境读取一个数据
 * @method getGlobalItem
 * @param name {string} 数据名
 * @return {any} 数据对象
 */
getGlobalItem(name)
```

1. <span id="removeGlobalItem">**removeGlobalItem**</span>: 从NGS全局环境删除一个数据
```
/**
 * 从NGS全局环境删除一个数据
 * @method removeGlobalItem
 * @param name {string} 数据名
 */
removeGlobalItem(name)
```

1. <span id="putTask">**putTask**</span>: 置入一个定时任务
```
/**
 * 置入一个定时任务
 * 
 * @method putTask
 * @param name {string} 定时器名称
 * @param delay {integer} 延时执行秒
 * @param immediate {boolean} 是否立即执行
 * @param cmdId {string} 任务回调的命令组ID
 * @param [ev] {any} 任务执行入参环境, JSON格式对象
 * @return {boolean}
 */
putTask(name, delay, immediate, cmdId, ev)
```

1. <span id="clearTask">**clearTask**</span>: 清除一个定时器任务
```
/**
 * 清除一个定时器任务
 * 
 * @method clearTask
 * @param name {string} 定时器名称
 */
clearTask(name)
```

1. **openDyn（openAddinDyn）**: 打开一个动态页面
```
/**
 * 打开一个动态页面
 * @param dynId {string} 动态页面ID
 * @param parameter {any} 打开动态页面时的入参json对象
 * @param replace {boolean}是否直接替换当前页面
 */
openDyn(dynId, parameter, replace)
```

1. <span id="closePage">**closePage**</span>: 关闭当前页面
与pc不同，pc的的模组，在手机端也有类似方式，就是通过这个closePage的传参，在关闭当前页时候，把信息传递给前一个页面
```
/**
 * 关闭当前页面
 * @param parameters 传递的参数，如果上一个页面也是动态页面，可以把这个parameters对象赋值到上一个动态页面的instance上
 */
closePage(parameters)
```

1. <span id="savePage">**savePage**</span>: 保存当前动态页面
与pc不同，返回的promise对象，从promise的then中才可以获取到返回
```
/**
 * 保存当前动态页面
 * @method savePage
 * @param flag {boolean} 保存成功后是否关闭当前动态页面
 * @return {Promise}
 */
savePage(flag)
```

1. <span id="setDynOwner">**setDynOwner**</span>: 设置当前动态页面的归属人
```
/**
 * 设置当前动态页面的归属人
 * @param owner {string} 归属人
 * @param inclusive {boolean} 独占性
 */
setDynOwner(owner, inclusive)
```

1. <span id="getCustomerInfo">**getCustomerInfo**</span>: 获取当前实例客户的一个属性值
```
/**
 * 获取当前实例客户的一个属性值
 * @param field {string} 客户资料字段名
 */
getCustomerInfo(field)
```

1. <span id="setCustomerInfo">**setCustomerInfo**</span>: 给当前实例客户设置一个属性值
```
/**
 * 给当前实例客户设置一个属性值
 * @param field {string} 客户资料字段名
 * @param value {any} 客户资料字段值
 */
setCustomerInfo(field, value)
```

1. <span id="isProjectCustomer">**isProjectCustomer**</span>: 判断当前实例客户是否相同与框架客户
```
/**
 * 判断当前实例客户是否相同与框架客户
 * @return {boolean}
 */
isProjectCustomer()
```

1. <span id="saveCustomer">**saveCustomer**</span>: 保存当前框架中的客户资料
```
/**
 * 保存当前框架中的客户资料
 * 注意，本方法并非针对当前实例中的客户进行保存，而是针对框架客户进行保存。
 * 当且仅当实例客户与框架客户相同时，该方法即等效于保存实例客户资料
 */
saveCustomer()
```

1. <span id="setCurrentCustomer">**setCurrentCustomer**</span>: 设置当前框架客户
```
/**
 * 设置当前框架客户
 * @param customerGuid {string} 客户GUID
 * @returns {Promise}
 */
setCurrentCustomer(customerGuid)
```

1. <span id="openAddinWO3ByOId">**openAddinWO3ByOId**</span>: 根据objective_guid和taskId打开工单3
```
/**
 * 根据objective_guid和taskId打开工单3
 * @param tabName {string} TAB标签
 * @param oId {string} 市场活动guid
 * @param taskId {string} 工单三任务guid
 * @returns {*}
 */
openAddinWO3ByOId(tabName, oId, taskId)
```

1. <span id="openAddinWO3ByOtId">**openAddinWO3ByOtId**</span>: 根据objectivetype_id打开工单3，可能是新建工单
```
/**
 * 根据objectivetype_id打开工单3，可能是新建工单
 * @param tabName {string} TAB标签
 * @param otId {string} 市场活动类型id
 * @returns {*}
 */
openAddinWO3ByOtId(tabName, otId)
```

1. <span id="openPlugin">**openPlugin**</span>: 打开一个插件
```
/**
 * 打开一个插件
 * @method openPlugin
 * @param plugin {string} 标签名称
 * @param params {*} 入参
 */
openPlugin(plugin, params)
```

1. <span id="stop">**stop**</span>: 停止当前的命令组运行
```
/**
 * 停止当前的命令组运行
 * @param e {string}
 */
stop(e)
```

1. <span id="smsCodeSend">**smsCodeSend**</span>: 发送短信验证码
```
/**
 * 发送短信验证码
 * @method smsCodeSend
 * @param type {number} 0=向当前客户发送短信（仅适用于NL模式），1=向当前staff发送短信，2=向指定手机发送短信
 * @param mobile {string} 接收短信的手机号码（仅当type=2时有效）
 * @return {Promise}
 */
smsCodeSend(type, mobile)
```

1. <span id="format">**format**</span>: 对内容进行格式化
```
/**
 * 对内容进行格式化
 * @param content {string} 需要格式化的内容
 * @param expression {string} 格式化表达式
 * @return {string}
 */
format(content, expression)
```
