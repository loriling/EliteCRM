#Event对象 

###Event对象可以让配置人员方便实现基于事件驱动的编程，集合了事件的注册，触发等等api。

1. <span id="Event.register">**Event.register**</span>: 注册一个事件监听器
```
/**
 * 注册一个事件监听器
 * @method register
 * @param eventKey {string} 事件名称
 * @param callback {string|integer|function} 命令号或者回调方法
 * @param [instanceScope] {boolean} 将该消息事务注册在当前实例内
 * @param [runtimeScope] {boolean} 将该消息事务注册在runtime内
 */
register(eventKey, callback, instanceScope, runtimeScope)
```

1. <span id="Event.fire">**Event.fire**</span>: 触发一个事件
```
/**
 * 触发一个事件
 * @method fire
 * @param eventKey {string} 事件名称
 * @param [parameters] {any} 事件传递的数据对象，JSON格式
 * @param [instanceScope] {boolean} 仅触发当前实例内注册的消息事务
 */
fire(eventKey, parameters, instanceScope)
```

1. <span id="Event.remove">**Event.remove**</span>: 移除事件监听
```
/**
 * 移除事件监听
 * @method remove
 * @param [eventKey] {string} 事件名称
 * @param [instanceScope] {boolean} 移除当前实例内的事件监听
 * @param [runtimeScope] {boolean} 移除当前runtime内的事件监听
 */
remove(eventKey, instanceScope, runtimeScope) 
```

1. <span id="Event.fireCtl">**Event.fireCtl**</span>: 触发一个控件的事件
```
/**
 * 触发一个控件的事件
 *
 * @param ctl {string | object} 控件ID或对象
 * @param e {string} 事件名称
 */
fireCtl(ctl, e)
```

1. <span id="Event.fireComponentKey">**Event.fireComponentKey**</span>: 触发组件内注册的消息别名
```
/**
 * 触发组件内注册的消息别名
 * @method fireComponentKey
 * @param eventKey {string} 注册在组件内的消息别名
 * @param [parameters] {any} 事件传递的数据对象，JSON格式
 * @param [instanceScope] {boolean} 仅触发当前实例内注册的消息事务
 * @param [quitely] {boolean} 是否以安静模式运行（即时别名不存在也不会抛出异常），默认为false
 */
fireComponentKey(eventKey, parameters, instanceScope, quitely)
```

1. <span id="Event.fireComponentCmd">**Event.fireComponentCmd**</span>: 调用组件的某个命令
```
/**
 * 调用组件的某个命令
 * @method fireComponentCmd
 * @param cmdId {string} 命令号
 * @param [path] {string} 组件在页面中的path，为空时代表执行该实例下所有组件的cmdId命令
 * @param [quietly] {boolean} 是否以安静模式运行（即时组件未加载或不存在也不会抛出异常），默认为false
 */
fireComponentCmd(cmdId, path, quietly)
```

1. <span id="Event.fireCpCmd">**Event.fireCpCmd**</span>: 调用组件的某个命令
```
/**
 * 调用组件的某个命令
 * @param cmdId {string} 命令号
 * @param [path] {object} 组件在页面中的path，为空时代表执行该实例下所有组件的cmdId命令
 * @param [parameters] {object} 入參给组件命令的对象
 * @param [quietly] {boolean} 是否以安静模式运行（即时组件未加载或不存在也不会抛出异常），默认为false
 */
fireCpCmd(cmdId, path, parameters, quietly)
```