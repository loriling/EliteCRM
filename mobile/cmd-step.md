#Step对象 

###动态页面子页面操作类

1. <span id="Step.next">**Step.next**</span>: 跳转到下一页
```
/**
 * 跳转到下一页
 * @method next
 * @return {boolean}
 */
next: function () 
```

1. <span id="Step.prev">**Step.prev**</span>: 跳转到上一页
```
/**
 * 跳转到上一页
 * @method prev
 * @return {boolean}
 */
prev: function ()
```

1. <span id="Step.jump">**Step.jump**</span>: 跳转到某个子页面
```
/**
 * 跳转到某个子页面
 * @method jump
 * @param subPageId {string} 子页面ID
 * @return {boolean}
 */
jump: function (subPageId)
```