#Perz对象 

###个性化数据操作类

1. <span id="Perz.setPublic">**Perz.setPublic**</span>: 写入一个全局私有数据
```
/**
 * 写入一个全局私有数据
 * @method setPublic
 * @param key {string} 私有数据的key
 * @param value {any} 私有数据的值
 */
setPublic: function (key, value)
```

1. <span id="Perz.getPublic">**Perz.getPublic**</span>: 读取一个全局私有数据
```
/**
 * 读取一个全局私有数据，全局私有数据在NGS项目中有效
 * @method getPublic
 * @param key {string} 私有数据的key
 * @param defaultValue {any} 当该私有数据不存在时，返回defaultValue
 * @return {any} 私有数据的值
 */
getPublic: function (key, defaultValue)
```

1. <span id="Perz.set">**Perz.set**</span>: 写入一个当前页面的私有数据
```
/**
 * 写入一个当前页面的私有数据
 * @method set
 * @param key {string} 私有数据的key
 * @param value {any} 私有数据的值
 */
set: function (key, value)
```

1. <span id="Perz.get">**Perz.get**</span>: 读取一个私有数据
```
/**
 * 读取一个私有数据。当前页面中如果不存在该私有数据，则自动在全局中记载该私有数据
 * @method get
 * @param key {string} 私有数据的key
 * @param defaultValue {any} 当该私有数据不存在时，返回defaultValue
 * @return {any} 私有数据的值
 */
get: function (key, defaultValue)
```

1. <span id="Perz.getLocal">**Perz.getLocal**</span>: 读取一个当前页面中的私有数据
```
/**
 * 读取一个当前页面中的私有数据
 * @method getLocal
 * @param key {string} 私有数据的key
 * @param defaultValue {any} 当该私有数据不存在时，返回defaultValue
 * @return {any} 私有数据的值
 */
getLocal: function (key, defaultValue)
```