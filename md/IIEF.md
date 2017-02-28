# 立即调用的函数表达式 Immediately-invoked Function Expression #

> 保护代码免受其他代码的干扰，并且通过封装的方式组织你的代码。

在 JavaScript 中，意外的声明一个全局变量是最危险也是最容易发生的事情是，而这并不是你的初衷。如果你忘记声明一个变量，JavaScript 将把它声明为一个全局变量！
```js
var foo = "foo";
bar = function() {
    foobar = foo + " foo";
    window.foobarbar = foobar + " bar!";
    return foobar + " foo bar!";
};

console.log( window.foo );       // I'm a global variable
console.log( window.bar() );     // I'm a global function
console.log( window.foobar );    // I'm a global variable too :(
console.log( window.foobarbar ); // I'm a global variable too :|
```


# 为什么全局对象是一个问题？ #

 - 与你的代码冲突
 
 - 与你的代码和第三方库冲突
 
 -  与你的代码和浏览器附加元件/扩展/插件冲突

包括一些chrome的插件，或者微信中的插件，有时候就会有一些意想不到的命名冲突


# 保护自己的多种方式 #

>对象字面量

把其他的全局变量，都放到一个对象中，对于全局，只暴露这一个对象
```js
var foo = {
    bar: "bar",
    foobar: function() {
        console.log( this.bar + " foobar!" );
    }
};

console.log( window.foo );          // Only 1 global object!
console.log( window.foo.bar );      // bar
console.log( window.foo.foobar() ); // bar foobar! 
```

>立即调用的函数表达式

解决全局问题的另一项技术是立即调用的函数表达式（IIFE）。这项技术允许开发人员向消费者公开公共和私有的属性和方法。在 JavaScript 中，**变量的作用域由函数作用域决定，而不是块级作用域**。
```js
// JavaScript can parse this just fine now, yay!
(function() {
    // All memory is contained within this scope
}()); // <-- Immediately Invoked
```



```js
// Revealing Module Pattern
var Customer = (function() {
    var Cust = function(id, name) {
        this.id = id;
        this.name = name;
    };

    Cust.prototype = {
		query : function(){
			//bala bala
		},
		clear : function(){
            //bala bala
        },
        getId : function(){
	        return this.id;
		}
	}
	

    return { // Only the items returned are public
        create : function(id, name) {
            return new Cust(id, name);
        }
    };
}());

var lori = Customer.create("00001", "lori");
console.log(Customer);     // Only 1 global object!
console.log(lori.name);    // Public property
console.log(lori.clear()); // Public method
console.log(lori.Cust);    // undefined, private variable can`t be accessed
```

## 现在比较流行的写法 ##
```js
;(function () {
	var elite = {};
    // Node.js
    if (typeof module !== 'undefined' && module.exports) {
        module.exports = elite;
    }
    // AMD / RequireJS
    else if (typeof define !== 'undefined' && define.amd) {
        define([], function () {
            return elite;
        });
    }
    // included directly via <script> tag
    else {
        root.elite = elite;
    }

}());

```