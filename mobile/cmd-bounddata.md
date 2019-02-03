#BoundData对象与Bound命令集 

###BoundData对象和Bound命令集可以帮助我们快速方便的对数据库对象进行增删改操作，特别是遇到一些循环命令时候，使用BoundData来构建对象缓存，并且使用Bound命令集一次提交批量操作，提高交互效率。

示例：
```
R.temp.customerList.forEach(function(c) {
	var cbd = new BoundData('CUSTOMER');//构造一个customer表的BoundData对象
	cbd.setKey("CUSTOMER_GUID");//设置CUSTOMER_GUID为为主键
    cbd.setCreateInfo("CREATEDDATE", "CREATEDBY", $E.staff.id);//设置创建时间创建人
    cbd.setModifyInfo("MODIFIEDDATE", "MODIFIEDBY", $E.staff.id);//设置修改时间修改人
	cbd.setValue('CUSTOMER_GUID', c[0]);//给CUSTOMER_GUID赋值
	cbd.setValue('CUSTOMERNAME', c[1]);
	... //从表格中每一项中数据给customer各个字段赋值
	...
	Bound.push(cbd);//把这一个BoundData对象添加到Bound缓存中
});
//提交缓存，数据保存到数据库
Bound.submit().then(function(ret) {//手机端与服务器交互操作都是异步的，这里的submit也是返回的Promise对象
	if (ret === 1) {
		$F.notify('保存成功')
	} else {
		$F.alert('保存失败')
	}
}); 
```

###下面我们来分别具体介绍BoundData和Bound各提供了哪些方法：

##BoundData对象

1. <span id="BoundData">**BoundData构造函数**</span>: 构造BoundData对象
在面向对象的编程中，世间万物皆对象。而我们需要使用某个对象，调用这个对象的方法，就需要先构造这个对象，这里会运用到关键字<span class="imp">new</span>
```
/**
 * BoundData构造函数
 * @param tableName {string} 表名
 */
constructor (tableName)
例如：
var cbd = new BoundData('CUSTOMER');//构造一个customer表的BoundData对象
```

1. <span id="BoundData.setKey">**BoundData.setKey（BoundData.KeyInfoEdit）**</span>: 设置主键
```
/**
 * 设置当前表的主键字段
 * @method setKey
 * @param fieldName {string} 主键字段
 * @return {BoundData}
 */
setKey(fieldName)
```

1. <span id="BoundData.setCreateInfo">**BoundData.setCreateInfo**</span>: 设置创建人创建时间字段
```
/**
 * 设置创建人创建时间字段
 * @param vDField 创建时间字段
 * @param vBField 创建人字段
 * @param vBy 创建人
 * @return {BoundData}
 */
setCreateInfo(vDField, vBField, vBy)
```

1. <span id="BoundData.setModifyInfo">**BoundData.setModifyInfo**</span>: 设置修改人修改时间字段
```
/**
 * 设置修改人修改时间字段
 * @param vDField 修改时间字段
 * @param vBField 修改人字段
 * @param vBy 修改人
 * @return {BoundData}
 */
setModifyInfo(vDField, vBField, vBy)
```

1. <span id="BoundData.setValue">**BoundData.setValue**</span>: 对字段赋值
```
/**
 * 对字段赋值
 * @method setValue
 * @param fieldName {string} 字段名
 * @param value {any} 字段值
 * @param fieldType {number} 字段类型
 * @param flag {number} 设置标志
 * @param methodType {boolean} 字段类型。无值或false时对新建字段赋值，否则对已有字段操作
 * @return {BoundData}
 */
setValue (fieldName, value, fieldType, flag, methodType)
```

1. <span id="BoundData.getValue">**BoundData.getValue**</span>: 获取某个字段的值
```
/**
 * 获取某个字段的值
 * @param vFN {string} 字段名
 * @return {string}
 */
getValue(vFN)
```

1. <span id="BoundData.deleteFlag">**BoundData.deleteFlag**</span>: 删除的标记
```
如果需要删除这条记录，只需要对BoundData对象的deleteFlag赋值1即可，默认时候这个标记为0。
例如：
var cbd = new BoundData('CUSTOMER');
cdb.deleteFlag = 1;
```

1. <span id="BoundData.notFindDupli">**BoundData.notFindDupli**</span>: 不除重标记
```
如果不设置此属性为1，则系统默认按设置的主键值来判断是新增还是更新记录。如果赋值为1后，系统则不考虑是否存在相同主键记录，直接做插入操作0。
例如：
var cbd = new BoundData('CUSTOMER');
cdb.notFindDupli = 1;
```

1. <span id="BoundData.copy">**BoundData.copy**</span>: 拷贝一个BoundData对象
```
/**
 * 拷贝一个BoundData对象
 * @param tableName 表名
 * @param resetKey 是否重置主键字段
 * @return {BoundData}
 */
copy(tableName, resetKey)
```

##Bound类

1. <span id="Bound.save">**Bound.save**</span>: 直接向数据库提交保存一个BoundData对象
```
/**
 * 直接向数据库提交保存一个BoundData对象
 * @param boundData {object} BoundData对象
 * @param [dbpool] {string} 提交数据的数据库连接池
 * @return {Promise}
 */
save(boundData, dbpool)
```

1. <span id="Bound.push">**Bound.push**</span>: 向缓存内置入一个BoundData对象
```
/**
 * 向缓存内置入一个BoundData对象
 * @param boundData {object} BoundData对象
 */
push(boundData)
```

1. <span id="Bound.submit">**Bound.submit**</span>: 提交缓存内的所有BoundData对象，并清空缓存
```
/**
 * 提交缓存内的所有BoundData对象，并清空缓存
 * @param [dbpool] {string} 提交数据的数据库连接池
 * @return {Promise} 1:正确提交，-1：提交异常
 */
submit(dbpool)
```

1. <span id="Bound.clear">**Bound.clear**</span>: 清空缓存内的BoundData对象
```
/**
 * 清空缓存内的BoundData对象
 */
clear()
```

1. <span id="Bound.log">**Bound.log**</span>: 输出当前Bound中BoundData信息
```
/**
 * 输出当前Bound中BoundData信息
 */
log()
```

1. <span id="Bound.type">**Bound.type**</span>: 返回某个字段的类型
```
/**
 * 返回某个字段的类型
 * @param propName {string} 字段名
 * @return {number}
 */
type(propName)
类型包含了：
BoundType : {
    F_String : 0,
    F_Float : 1,
    F_Date : 2,
    F_Boolean : 3,
    F_Clob : 4,
    F_Long : 5,
    F_Int : 6
}
```