#Utils对象 

###其他工具操作类

1. <span id="Utils.getIp">**Utils.getIp**</span>: 获取ip
```
/**
 * 获取当前客户端访问到服务器的ip地址
 * @return {Promise}
 */
getIp()
```

1. <span id="Utils.ip2long">**Utils.ip2long**</span>: ip字符串转换成long数值
```
/**
 * ip字符串转换成long数值
 * @param ip {string} ip字符串
 * @return {number}
 */
ip2long: function (ip)
```

1. <span id="Utils.long2ip">**Utils.long2ip**</span>: long数值转换为ip字符串
```
/**
 * long数值转换为ip字符串
 * @param number {number} ip数值
 * @return {string}
 */
long2ip: function (number)
```

1. <span id="Utils.getAgeFromIdCard">**Utils.getAgeFromIdCard**</span>: 从有效身份证号中获取年龄
```
/**
 * 从有效身份证号中获取年龄
 * @param id {string} 身份证号码字符串
 * @return {int} 年龄
 */
getAgeFromIdCard : function(id)
```

1. <span id="Utils.getBirthdayFromIdCard">**Utils.getBirthdayFromIdCard**</span>: 从有效身份证号中获取生日
```
/**
 * 从有效身份证号中获取生日
 * @param id {string} 身份证号码字符串
 * @return {string} 生日日期字符串
 */
getBirthdayFromIdCard : function(id)
```

1. <span id="Utils.getGenderFromIdCard">**Utils.getGenderFromIdCard**</span>: 从有效身份证号中获取性别
```
/**
 * 从有效身份证号中获取性别
 * @param id {string} 身份证号码字符串
 * @return {int} 性别，0=女，1=男
 */
getGenderFromIdCard : function(id)
```

1. <span id="Utils.isValidIdCard">**Utils.isValidIdCard**</span>: 判断是否有效身份证号码
```
/**
 * 判断是否有效身份证号码
 * @param id {string} 身份证号码字符串
 * @return {int} 不等于1时为否
 */
isValidIdCard : function(id) 
```

1. <span id="Utils.stringEncode">**Utils.stringEncode**</span>: 将UTF-8编码的字符串转换为指定编码的字符串
```
/**
 * 将UTF-8编码的字符串转换为指定编码的字符串
 * @param content {string} UTF-8编码的字符串
 * @param [charset] {string} 指定的编码名称
 * @return {Promise}
 */
stringEncode : function(content, charset)
```

1. <span id="Utils.stringDecode">**Utils.stringDecode**</span>: 将指定编码编码的字符串转换为UTF-8的字符串
```
/**
 * 将指定编码编码的字符串转换为UTF-8的字符串
 * @param content {string} 指定编码的字符串
 * @param [charset] {string} 指定的编码名称
 * @return {Promise} UTF-8编码的字符串
 */
stringDecode : function(content, charset)
```

1. <span id="Utils.anyCall">**Utils.anyCall**</span>: 调用第三方提供的jar包接口
```
/**
 * 调用第三方提供的jar包接口
 * @param beanName {string} 在cust-context.xml中定义的bean名称
 * @param methodName {string} bean对象提供的方法名称
 * @param args {array} 入參队列
 * @param [callback] {cmdId | function} 执行完成后的回调命令或方法
 * @return {Promise}
 * @example
 * 		// 第三方jar包提供的方法原型
 * 		// public String AES.encrypt(String content, String key);
 * 		Utils.anyCall('AES', 'encrypt', [{
	 * 				type:'string',			// 参数1的类型
	 * 				value: R.var.string		// 参数1的值
	 * 			}, {
	 * 				type: 'string',			// 参数2的类型
	 * 				value: R.var.key		// 参数2的值
	 * 			}], 'after.callback')
 *
 * 		// 参数类型说明 (java类型 -- anyCall类型)
 * 		// String 		-- string
 * 		// CharSequence -- chars
 * 		// char			-- char
 * 		// byte[]		-- bytes
 * 		// byte			-- byte
 * 		// Integer		-- integer
 * 		// Long			-- long
 * 		// Double		-- double
 * 		// Float		-- float
 * 		// Date			-- date
 * 		// Map			-- map
 * 		// List			-- list
 * 		// HttpRequest	-- request
 */
anyCall(beanName, methodName, args, callback)
```

1. <span id="Utils.serviceCall">**Utils.serviceCall**</span>: 调取数据服务器提供的类库方法
```
/**
 * 调取数据服务器提供的类库方法
 * @param className {string} JSService提供的类名
 * @param dbPool {string} 数据连接池
 * @param params {array} 入参数组，依次为入参值：['参数1', '参数2' ...]
 * @param [cmdId] {string} 执行完毕的回调命令，执行结果为R.RESULT。配置了cmdId之后，该方法为异步执行，否则为同步执行
 * @return {Promise}
 */
serviceCall: function(className, dbPool, params, cmdId)
```

1. <span id="Utils.stringEncode">**Utils.stringEncode**</span>: 将UTF-8编码的字符串转换为指定编码的字符串
```
/**
 * 将UTF-8编码的字符串转换为指定编码的字符串
 * @param content {string} 字符串
 * @param [charset] {string} 指定的编码名称
 * @return {Promise} 指定编码的字符串
 */
stringEncode(content, charset)
```

1. <span id="Utils.qrencode">**Utils.qrencode**</span>: 获取二维码图片
```
/**
 * 获取二维码图片
 * @param options {object} 入參
 * @param options.content {string} 二维码的内容
 * @param [options.size] {integer} 二维码的尺寸（单位像素，默认值为400）
 * @param [options.callback] {string | function} 回调方法或者命令id
 * @return {Promise} 回调入參为R.RESULT
 */
qrencode(options)
```

1. <span id="Utils.qrdecode">**Utils.qrdecode**</span>: 解码二维码图片
```
/**
 * 解码二维码图片
 * @param options {object} 入參
 * @param options.content {string} 二维码的图片路径，可是使用http绝对路径的url，也可以使用fs服务相对路径
 * @param [options.callback] {string | function} 回调方法或者命令id
 * @return {Promise} 回调入參为R.RESULT
 */
qrdecode(options) 
```