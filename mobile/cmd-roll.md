#Roll对象 

###轮询数组对象操作

1. <span id="Roll.build">**Roll.build**</span>: 创建一个轮询对象
```
/**
 * 创建一个轮询对象
 * @method build
 * @param name {string} 轮询对象名称
 * @param data {array | string} 用于轮询的数组，或变量名称
 */
build: function (name, data)
```

1. <span id="Roll.get">**Roll.get**</span>: 获取一个build成功的轮询对象，该对象只能被get一次
```
/**
 * 获取一个build成功的轮询对象，该对象只能被get一次
 * @param name {string} 轮询对象名称
 * @return {object} 轮询对象
 */
get: function (name)
```

1. <span id="Roll.pick">**Roll.pick**</span>: 从Project缓存中取出一个轮询对象到instance中
```
/**
 * 从Project缓存中取出一个轮询对象，并置入当前实例中。该对象被取出后，将从Project中被清除。
 * @param name
 */
pick: function (name)
```
 

1. <span id="Roll.remove">**Roll.remove**</span>: 从轮询对象中删除数据
```
/**
 * 从轮询对象中删除数据
 * @param target {string | object} 轮询对象或者名称
 * @param index {integer} 删除数据时对比的列号，下标为0
 * @param value {any} 删除数据时对比的值，在制定列号上相同值得记录将从循环队列中移除
 * @return {integer} 实际移除的行数
 */
remove: function (target, index, value) 
```

1. <span id="Roll.hasNext">**Roll.hasNext**</span>: 判断是否还有下一条记录
```
/**
 * 判断是否还有下一条记录
 * @param target {string | object} 轮询对象或者名称
 * @return {boolean}
 */
hasNext: function (target)
```

1. <span id="Roll.next">**Roll.next**</span>: 获取下一条数据
```
 /**
 * 获取下一条数据
 * @param target {string | object} 轮询对象或者名称
 * @return {any} 轮询对象的下一条数据
 */
next: function (target)
```

1. <span id="Roll.hasPrev">**Roll.hasPrev**</span>: 判断是否有上一条记录
```
/**
 * 判断是否有上一条记录
 * @param target {string | object} 轮询对象或者名称
 * @return {boolean}
 */
hasPrev: function (target)
```

1. <span id="Roll.prev">**Roll.prev**</span>: 获取上一条数据
```
/**
 * 获取上一条数据
 * @param target {string | object} 轮询对象或者名称
 * @return {any} 轮询对象的上一条数据
 */
prev: function (target) 
```

1. <span id="Roll.reset">**Roll.reset**</span>: 重设轮询对象指针
```
/**
 * 重设轮询对象指针
 * @method reset
 * @param target {string | object} 轮询对象或者名称
 */
reset: function (target)
```
