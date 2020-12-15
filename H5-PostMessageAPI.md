# 通过HTML5的postMessage和onMessage实现跨页面互传消息

1. 动态页面中首先有个地方注册事件，比如某个常驻动态页面的加载命令里

   ```javascript
   window.addEventListener("message", function(event) {
   	// 收到消息，获取event中参数，event.data的值就是postMessage时候传递的值
       console.log(event.data)
   }, false);
   ```

2. 第三方页面发出消息

   如果第三方页面是ngs中的一个iframe，那么可以

   ```javascript
   window.parent.postMessage({
   	action: "search"
       param: {
       	p1: "p1",
       	p2: "p2"
   	}
   }, "*")
   ```

   