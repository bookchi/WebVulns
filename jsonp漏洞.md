# jsonp跨域漏洞

[toc]

jsonp劫持，就是攻击者在自己的server上写一个恶意的网页，受害者点击之后，攻击者就会获取到victime的敏感信息，并发送到attakcer的服务器。

## JSONP

jsonp是实现跨域的一种技术，应用于Web站点需要跨域获取数据的场景。

简单的场景如下：

`a.com`存在如下`data.json`文件：

```json
{ username: "mi1k7ea", password: "secret" }
```

用下面的`1.html`文件发起Ajax请求获取`data.json的`内容并记录日志：

```html
<script src='./jquery.js'></script>
<script >
 $.ajax({
 url: 'http://a.com/data.json',
 type:"get",     
 dataType: "json",
 success: function (data) {
  console.log(data);}
 })
</script>
```

若`1.html`与`data.json`同处于`a.com`下(也就是同源)，那么访问这个文件就可以获取到`data.json`文件的内容；

若`1.html`在`b.com`下，它就与`data.json`不同域，访问`1.html`时浏览器会报错，这是因为ajax不能发起跨域请求。

> 是ajax不能发起跨域请求？还是发起了请求但是不会被正确响应？

但是，开发为了方便程序间数据的调用，搞了几种跨域的方法，其中就包括了jsonp。

简单来讲，原理就是：`script`标签的`src`属性能够发起跨域请求。

将`1.html文件`改为：

```html
<body>
  <script src='./jquery.js'></script>
  <script>
    var s = document.createElement('script')
    s.src = "http://a.com/data.json"
    document.body.appendChild(s);
  </script>
</body>
```

此时，再访问`1.html`就可以发起跨域请求了；但是，会看到浏览器报错，这是因为`data.json`中的内容并不符合JavaScript代码规范。

重新定义`data.json`让它符合`JSONP`规范：

```json
callback({ username: "mi1k7ea", password: "secret" })
```

然后在`1.html`中添加callback的函数定义即可

```html
<body>
 <script src='./jquery.js'></script>
 <script type="text/javascript">
 function callback(json) {
  console.log(json);
 }
 var s = document.createElement('script');
 s.src = 'http://a.com/data.json';
 document.body.appendChild(s);
 </script>
</body>
```

此时，基本的JSONP功能就实现了，我们Web站点的HTML文件能够正常地跨域获取目标外域JSON数据了。

这就很清楚了，jsonp是跨域技术的一种，用来方便web站点突破SOP限制获取数据。

### 基本原理

JSON（JavaScript Object Notation），即JavaScript对象表示法。

JSONP（JSON with Padding）即填充式的JSON，通过填充额外的内容把JSON数据包装起来，变成一段有效的可以独立运行的JavaScript语句。它是基于JSON 格式的为解决跨域请求资源而产生的解决方案，基本原理是利用HTML里script元素标签，远程调用JSON文件来实现数据传递。

> 把json填充成js语句

JSONP的基本语法为：callback({“name”:”alan”, “msg”:”success”})

常见的例子包括函数调用`callback({“a”:”b”})`或变量赋值`var a={JSON data}`

### 实现流程

jsonp.php---a.com

```php
<?php
  header('Content-type: application/json');
	// 获取回调函数名
  $jsoncallback = htmlspecialchars($_REQUEST['callback']);
	// json数据
	$json_data = '["mi1k7ea","https://www.mi1k7ea.com"]';
	// 输出jsonp格式数据
	echo $jsoncallback . "(" . $json_data . ")";
  ?>
```

jsonp.html---localhost

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>JSONP Test</title>
</head>
<body>
<div id="here"></div>
  <script type="text/javascript">
    function callbackFunction(result, methodName){
      var html = "<ul>"
      for (var i = 0; i<result.length; i++)
        html += '<li>'+result[i]+'</li>'
      html += '</ul>'
      document.getElementById('here').innerHTML = html
    }
  </script>
  <script type="text/javascript" src="http://a.com/jsonp.php?callback=callbackFunction"></script>
</body>
</html>
```

直接访问本地的`jsonp.html`，看到其跨域访问JSONP端点并解析显示jsonp数据到页面中。

。。。

## JSONP跨域漏洞

主要是callback用户自定义导致的XSS和JSONP劫持。

### 导致的XSS

传入函数参数名，然后JSONO端点会根据我们的传参动态生成JSONP数据响应。

如果JSONP端点对传入的函数参数名callback处理不当，就会导致XSS；

例如未正确设置响应的`Content-Type`，or未对用户输入参数进行有效的过滤or转义。

#### 未设置Content-Type且未过滤

这是最常见的，网上大多数JSONP引起的XSS也都是这种场景的。

data.php

```php
<?php
  if(isset($_GET['callback'])){
		$callback = $_GET['callback'];
		print $callback.'({"username" : "mi1k7ea", "password" : "thisispassword"});';
	} else {
		echo 'No callback param.';
	}
  ?>
```

正常访问的话，会在页面返回jsonp数据。

```json
// http://xxx.com/data.php?callback=hello
hello({"username" : "mi1k7ea", "password" : "thisispassword"});
```

但是我输入XSS payload的话，会弹框，看到Content-Type的值为`text/html`

```json
// http://xxx.com/data.php?callback=hello<script>alert(0)</script>
hello	// 弹框
```

那么Content-Type应该如何设置才好呢？

- application/json

  查阅资料可以发现，json文本的MIME类型是`application/json`，默认编码为UTF-8；

  同时也建议设置为这个，从而防御XSS。

  Data.php

  ```php
  <?php
    // 设置content-type
    header('Content-type: application/json');
  if(isset($_GET['callback'])){
  	$callback = $_GET['callback'];
  	print $callback.'({"username" : "mi1k7ea", "password" : "thisispassword"});';
  } else {
  	echo 'No callback param.';
  }
    ?>
  ```

  ~~这时，输入xss payload或正常访问，页面都会报错，不显示正常内容。~~

  ~~但在浏览器查看数据，发现响应中有JSONP数据返回，只是没有在页面中解析而已。~~

  > ~~显然，不需要在页面显示的话，这样完全莫得问题~~

- text/json

  这个是application/json正式注册之前，JSON的实验版MIME类型。

  将`data.php`的`content-type`改为`text/json`，然后再访问页面。

  同上面的一样，响应中有jsonp数据，但是页面不会显示，同样也不会弹框。

- application/javascript与text/javascript

  其实，JSONP数据是JS数据，返回的内容就是：传入参数的JS函数的调用。

  `application/javascript`是JavaScript的正式注册的MIME媒体类型。

  因此，有些人会将类型设置为这个。在chrome和firefox下不会弹窗，IE这个搅屎棍就。。。

  > `Text/javascript`是`application/javascript`的测试版

- X-Content-Type-Options

  如果response中`X-Content-Type-Options`被设置为`nosniff`，那么`content-type`必须设置为`javascript`才行。

  这是因为在response中包含回调，产生的问题。这时它不在解析json而是js。

  > ？？？

### JSONP劫持

> jsonp函数的参数，是要获取的信息；如果信息是敏感信息，问题就来了。

JSONP劫持和CSRF攻击类似，只不过CSRF提交表单请求，但是JSONP是将获取到的数据发送到攻击者服务器，实现获取JSONP敏感信息。

JSONP劫持的前提和CSRF是一样的，当服务端没有校验请求来源，如未严格校验Referer或未存在token机制等，都会导致JSONP劫持的产生。

---未完
