### 1. JS中的Array.splice()和Array.slice()方法有什么区别

slice和splice虽然都是对于数组对象进行截取,但是二者还是存在明显区别,函数参数上slice和splice第一个参数都是截取开始位置,slice第二个参数是截取的结束位置(不包含),而splice第二个参数(表示这个从开始位置截取的长度),slice不会对原数组产生变化,而splice会直接剔除原数组中的截取数据!



### 2. 理解this

this就是指针， 指向我们调用函数的对象；  this是JavaScript中的一个关键字，它是函数运行时，在函数体内自动生成的一个对象，只能在函数体内部使用。

1.   全局环境中this指向全局变量（window）；

2.   函数中的this，由调用函数的方式来决定，

   （1）如果函数是独立调用的，在严格模式下（use strict）是指向undefined的，在非严格模式下，指向window；

   （2）<u>**如果这个函数是被某一个对象调用，那么this指向被调用的对象**</u>；

3.   构造函数与原型里面的this

     构造函数里的this以及原型里的this对象指的都是生成的实例；（由new决定的）

通过new操作符可以初始化一个constructor的指向，new的作用就是创建一个对象的实例，constructor也就指向了一个新的执行环境“在这个对象之中”；

4.  箭头函数按词法作用域来绑定它的上下文，所以this 实际上会引用到原来的上下文。

（箭头函数会保持它当前执行上下文的词法作用域不变，而普通函数则不会，箭头函数从包含它的词法作用域中继承了this的值）


```js
var name = 'AAA';
var obj = {
    name: 'BBB',
    prop: {
        getName: function(){
            return this.name
        }
    }
}
console.log(obj.prop.getName());
var a = obj.prop.getName;
console.log(a());
```



### 3. **什么是跨域问题，如何解决跨域问题？**

因为浏览器有同源策略，如果协议（http或者https）不同、端口不同、主机不同，三者满足其一即产生跨域，跨域的解决方案最优的是cors

header("Access-Control-Allow-Origin", "*");



### 4. promise有几种状态？

pending, fulfilled, rejected

pending -> fulfilled;  pending -> rejected

pending到rejected时候，会进入catch

```js
const promise = new Promise((resolve, reject) => {
	console.log(1);
    resolve();
    console.log(2);
});
promise.then(() => { // then内部代码在档次时间魂环的结尾立刻执行
    console.log(3);
})
console.log(4);
// 1,2,4,3
```

手写Promise

```js
class Promise {
  constructor(executor) {
    this.status = 'pending';
    this.value = undefined;
    this.reason = undefined;
    let resolveFn = value => {
      if (this.status === 'pending') {
        this.status = 'fulfilled';
        this.value - value;
      }
    }
    let rejectFn = reason => {
      if (this.status === 'pending') {
        this.status = 'rejected';
        this.reason = reason;
      }
    }
    try {
      executor(resolveFn, rejectFn);
    } catch(e) {
      rejectFn(e);
    }
  }
  
  then(onFulfilled, onRejected) {
    if (this.status === 'fulfilled') {
      onFulfilled(this.value);
    }
    if (this.status === 'rejected') {
      onRejected(this.reason);
    }
  }
}
```



### 5. 有什么优雅的方式处理异步的嵌套

1. 回调函数
2. promise  promise.all
3. async await
4. async.js



### 6. **说一下VUE双向绑定的原理**

**1. Vue2.0通过Object.definePropety来劫持对象属性的getter和setter操作，当数据发生变化时通知**

**2. Vue3.0通过Proxy来劫持数据，当数据发生变化时发出通知**

proxy相较于object.defineProperty的优势

1. defineProperty只能监听某个属性，不能对全对象监听
2. Proxy可以监听数组的变化，不用再去单独的对数组做特异性操作
3. 可以检测到数组内部数据的变化



### 7. **Vue的父组件和子组件生命周期钩子执行顺序是什么**

vue2 中 执行顺序 `beforeCreate`=>`created`=>`beforeMount` =>`mounted`=>`beforeUpdate` =>`updated`=>`beforeDestroy`=>`destroyed`

1.首次加载过程

父beforeCreate ->父created ->父beforeMount ->子beforeCreate ->子created ->子beforeMount ->子mounted ->父mounted

2.父组件更新过程

父beforeUpdate->父updated

3.子组件更新过程

父beforeUpdate ->子beforeUpdate ->子updated ->父updated

4.销毁过程

父beforeDestroy->子beforeDestroy ->子destroyed ->父destroyed



vue3 中 执行顺序 `setup`=>`onBeforeMount`=>`onMounted`=>`onBeforeUpdate`=>`onUpdated`=>`onBeforeUnmount`=>`onUnmounted`