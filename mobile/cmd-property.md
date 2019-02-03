#Property对象 

###变量操作类

1. <span id="Property.create">**Property.create**</span>: 创建一个动态变量
```
/**
 * 创建一个动态变量
 * @method create
 * @param propertyName {string} 变量名
 * @param type {integer} 变量类型
 * 1: Float
 * 2: Date
 * 3: String
 * 4：Array
 * 5：Int
 * 6：Object
 * @param [defaultValue] {any} 可选参数，变量默认值
 * @return {boolean} 创建成功返回true，否则为false
 */
create: function (propertyName, type, defaultValue)
```

1. <span id="Property.copy">**Property.copy**</span>: 从源变量复制一个副本到目标变量
```
/**
 * 从源变量复制一个副本到目标变量，如果目标变量存在，则会被覆盖
 * @method copy
 * @param sourcePropertyName {string} 源变量
 * @param targetPropertyName {string} 目标变量
 * @param includeValue {boolean} 需要同步复制值为true，否则为false
 * @return {boolean} 复制成功返回true，否则为false
 */
copy: function (sourcePropertyName, targetPropertyName, includeValue)
```

1. <span id="Property.bind">**Property.bind**</span>: 绑定变量到控件对象
```
/**
 * 绑定变量到控件对象
 * @method bind
 * @param ctrl {string | object} 控件对象或者控件ID
 * @param propertyName {string} 变量名称
 * @return {boolean}
 */
bind: function (ctrl, propertyName) 
```