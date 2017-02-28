模块化 & 依赖管理 & 性能优化
===============

随着前端开发在近几年的飞速发展，JavaScript被越来越广泛的使用，单个页面上的js也越来越多，越来越复杂，这时候问题就来了。

```html
<script src="js/jquery/jquery.min.js"></script>
<script src="js/jquery.cookie.js"></script>
<script src="js/jquery.tmpl.min.js"></script>
<script src="js/bootstrap/js/bootstrap.js"></script>
<script src="js/spin.min.js"></script>
<script src="js/jquery.spin.js"></script>
<script src="js/main.js"></script>
<script src="js/area.js"></script>
```

平时我们写网页时候，都会在head里引入一个个js，这样的写法有很大的缺点。

 - 首先，加载的时候，浏览器会停止网页渲染，加载文件越多，网页失去响应的时间就会越长；
 - 其次，由于js文件之间存在依赖关系，因此必须严格保证加载顺序（比如上例的jquery.min.js要在其他jquery插件的前面），依赖性最大的模块一定要放到最后加载，当依赖关系很复杂的时候，代码的编写和维护都会变得困难。




CommonJS 社区带来了AMD规范
==============

**AMD 规范**
> define(id?, dependencies?, factory);

该方法用来定义一个 JavaScript 模块，开发人员可以用这个方法来将部分功能模块封装在这个 define 方法体内。


而RequireJS就是 AMD 规范最好的实现者之一
===========================

**RequireJS主要解决了两个问题：**

（1）实现js文件的异步加载，避免网页失去响应；
（2）管理模块之间的依赖性，便于代码的编写和维护。



繁琐复杂的js加载变成了一行
--------------

```html
<script src="js/require.js" data-main="js/main"></script>
```

main.js中，通过require.config来定义了各模块的路径，通过shim来包装一些本身非AMD模式的模块，并且可以配置相关依赖， **而正在要做的事情，就从require之后开始~**

```js
//main.js
require.config({
　　baseUrl: "js/lib",
　　paths: {
　　　　"jquery": "jquery.min",
　　　　"underscore": "underscore.min",
　　　　"backbone": "backbone.min"
　　}
　　shim: {
　　　　'jquery.scroll': {
　　　　　　deps: ['jquery'],
　　　　　　exports: 'jQuery.fn.scroll'
　　　　}
　　}
});

require(['jquery', 'underscore', 'backbone'], function ($, _, Backbone){
　　// some code here
});
```

定义自己的AMD模块
----------

```js
/**
* CallSummary module 
*/
define(["js/softphone/calltype", "js/softphone/workinfo", "js/softphone/pdstype", "js/objective" + $E.jsv], 
		function(CallType, WorkInfo, PDSType, Objective) {

	var csInst;
	var CallSummary = function(proj){
		this.proj = proj;
	}
	return {
        getInstance : function(proj){
            if(!csInst){
                csInst = new CallSummary(proj);
            }
            return csInst;
        }
    };
});
```

使用RequireJS提供的r.js来对项目做优化
-----------------------
r.js是RequireJS的一部分(optimizer)。它依赖于UglifyJS，而UglifyJS基于nodejs。r.js多数时候配合模块化(AMD)写法进行合并，压缩。如果你的代码不采用AMD方式，也可以用它来压缩。

首先，编写**app.build.js**

```js
/**
 * Created by Lori on 2015/12/16.
 * cd D:\Apache Software Foundation\Apache2.2\htdocs\test
 * node r.js -o app.build.js
 */
({
    baseUrl: "amd",               // The main root URL
    dir: "h:/temp/dist",            // Directory that we build to
    mainConfigFile: "amd/js/main.js", // Location of main.js
    paths : {
        "jquery": "js/jquery-2.1.4.min",
        "jquery.elite": "js/jquery.elite",
        "require": "js/require"
    }
})
```

通过**node r.js -o build.js**来对项目做build

```
D:\Apache Software Foundation\Apache2.2\htdocs\test>node r.js -o app.build.js
Uglifying file: h:/temp/dist/js/jquery-2.1.4.min.js
Uglifying file: h:/temp/dist/js/jquery.elite.js
Uglifying file: h:/temp/dist/js/jquery.eliteamd.js
Uglifying file: h:/temp/dist/js/main.js
Uglifying file: h:/temp/dist/js/require.js
```