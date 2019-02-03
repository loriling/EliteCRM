#Codetable对象 

###全局缓存操作类
Codetable表中记录了一些常用的代码表信息，系统加载时候会把这些信息一起加载到页面缓存中，通过Codetable对象的一些方法，可以快速方便的获取到这些缓存信息。其中有些信息获取可能在第一次时候会涉及到访问服务器，因此返回的是Promise对象，使用时候需要注意

1. <span id="Codetable.query">**Codetable.query**</span>: 对一个codetable表进行query查询
```
/**
 * 对一个codetable表进行query查询
 * @method query
 * @param tableName {string} 全局SQL名
 * @param expression {string} JSONSQL查询条件
 * @return {array}
 */
query : function(tableName, expression)
```

1. <span id="Codetable.getByKey">**Codetable.getByKey**</span>: 根据key值获取某个codetable表中的记录
```
/**
 * 快速检索返回codetable表的某条记录
 * @method getByKey
 * @param tableName {string} 全局SQL名
 * @param key {string} 主键值
 * @return {object}
 */
getByKey : function(tableName, key)
```

1. <span id="Codetable.sysParam">**Codetable.sysParam**</span>: 获取系统参数值
```
/**
 * 获取系统参数值
 * @method sysParam
 * @param key {string} 系统参数ID
 * @return {string} 系统参数值
 */
sysParam : function(key)
```

1. <span id="Codetable.sysAddins">**Codetable.sysAddins**</span>: 获取系统组件对象
```
/**
 * 获取系统组件对象
 * @method sysParam
 * @param addinId {string} 系统组件ID
 * @return {object} 系统组件JSON对象
 */
sysAddins : function(addinId)
```

1. <span id="Codetable.sysObjectiveType">**Codetable.sysObjectiveType**</span>: 获取系统市场活动对象
```
/**
 * 获取系统市场活动对象
 * @method sysObjectiveType
 * @param otId {string} 系统市场活动ID
 * @return {object} 市场活动JSON对象
 */
sysObjectiveType : function(otId)
```

1. <span id="Codetable.curStaff">**Codetable.curStaff**</span>: 获取当前登录STAFF账号的记录信息
```
/**
 * 获取当前登录STAFF账号的记录信息
 * @method curStaff
 * @return {Promise}
 */
curStaff : function()
```

1. <span id="Codetable.curRolegroup">**Codetable.curRolegroup**</span>: 获取当前登录STAFF账号登录的组信息
```
/**
 * 获取当前登录STAFF账号登录的组信息
 * @method curRolegroup
 * @return {Promise}
 */
curRolegroup : function()
```