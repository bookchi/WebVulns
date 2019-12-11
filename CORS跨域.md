# CORS跨域漏洞总结

这篇写的很垃圾，没有细致深入的分析什么点。就只是配置不当。

## CORS相关基本概念

- SOP：同源策略，要同协议+同域名+同端口，不再赘述

- JSONP：利用`<script>`标签的`src`属性没有跨域限制的特性，来跨域获取json数据；

  ​				不过，一定要服务器支持；

  > 显然，服务器如果不支持jsonp，根本不会有什么卵用

- CORS：(Cross-Origin Resource Sharing）跨源资源共享，是HTML5的一个新特性；其思想是使用自定义的HTTP头部让浏览器与服务器进行沟通，它允许浏览器向跨域服务器发出XHR请求，从而克服AJAX只能同源使用的限制。

  > 自定义的HTTP头部是？？？？

  基本原理：第三方网站服务器生成访问控制策略，指导用户浏览器放宽 SOP 的限制，实现与指定的目标网站共享数据。

  > 感觉就是，放宽了访问控制

  相比之下，CORS较JSONP更为复杂，JSONP只能用于获取资源（即只读，类似于GET请求），而CORS支持所有类型的HTTP请求，功能完善。

  CORS跨域访问资源示意图：

  ![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/1.png)

  CORS的具体工作流程可分为三步，如图所示：

  ![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/2.png)

  1. 请求方从browser发起跨域请求。浏览器会自动在每个跨域请求头中添加Origin头，用于声明请求方的源。
  2. server根据请求中的Origin头返回访问控制策略(也就是Access-Control-Allow-Origin header)，并在其中声明读取相应内容的origin
  3. browser检查server在`Access-Control-Allow-Origin`中声明的源，看是否去请求方的相符合；if so，就允许请求方脚本读取响应内容，否则就不允许。

  > 从这些来看，是browser去查看响应头与自己的origin是否符合，从而决定是否显示响应内容
  >
  > - 那我是不是就可以修改响应头，然后绕过策略呢？

### 基本用法

当b.com服务器想要与a.com共享资源内容时，它只需要在HTTP响应中添加如下响应头。这个响应头告诉浏览器放宽SOP限制，允许a.com脚本读取响应内容：

> 那么，浏览器是如何判断是否同源呢？看那三个东西是否一致？---那这又是如何判断？
>
> origin又是如何生成的？浏览器自己会带上？

```http
Access-Control-Allow-Origin: http://a.com
// 这个头的作用？
Access-Control-Allow-Credentials: true
```

a.com则可以通过下面的JS脚本，跨域读取b.com的内容

```javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function(){
  if (xhr.readyState == XMLHttpRequest.DONE) { 
        alert(xhr.responseText); 
	}
}
xhr.open(“GET“, ”http://b.com/api“, true);
xhr.withCredentials = true;
xhr.send();
```

#### 几个关键的HTTP头

CORS中几个关键的HTTP头字段如下：

```
Access-Control-Allow-Origin：指定哪些外域可以访问本域资源；
Access-Control-Allow-Credentials：指定浏览器是否会一起发送cookie。当为true时，才会发送cookie
Access-Control-Allow-Methods：指定可以使用哪些HTTP请求方法
Access-Control-Allow-Headers：指定可以在请求报文中添加的HTTP头字段
Access-Control-Max-Age：指定超时时间；
```

> 那如果，我只想让a.com访问b.com的某部分内容，其他的不允许呢？可以更细粒度吗？

### 请求分类

browser将CORS请求分成两类：简单请求和非简单请求。

- 简单请求

  满足以下条件：

  - 使用下列方法之一：GET、HEAD、POST
  - HTTP的头信息不超出以下几种字段:`Accept、Accept-Language、Content-Language、Content-Type`（其值仅限于：`application/x-www-form-urlencoded、multipart/form-data、text/plain`）

  简单请求如图所示，browser和server之间请求只进行了一次：

  ![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/3.png)

- 非简单请求

  简单请求以外的请求。

  非简单请求要先进行预检请求，即使用OPTIONS方法发起一个req到server，目的是询问server我所在网页的域名是否时访问控制允许的and哪些HTTP方法和header是被允许的。当得到server肯定的响应时，browser才会发送正式的XHR请求，否则报错。

  关于预检请求，需要注意一下两点：

  - 预检请求对JS来说是透明的，即JS获取不到预检请求的任何信息；
  - 预检请求并不是每次请求都发生，服务端设置的Access-Control-Max-Age头部指定了预检请求的有效期，在有效期内的非简单请求不需要再次发送预检请求

  ![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/4.png)

### 跨域场景

- 后段开发完一部分业务后，提供API给前段。这种前后端分离的模式下，前后端的域名不一致的话，会发生跨域访问

- 程序员在本地做开发，本地的文件夹并不是在一个域下面，当一个文件需要发送ajax请求，请求另外一个页面的内容的时候，就会跨域。

  > 说实话，这句没太看懂

- 电商网站想通过用户browser加载第三方快递网站的物流信息

- 子站域名希望调用主站域名的用户资料接口，并将数据显示出来。

应用demo

编写两个不同域下的文件cors.html和cors.php; cors.html放在本域下，目标是跨域获取cors.php中的内容。

Cors.html

```html
<html>
<head>
<meta charset="UTF-8" />
<title>CORS Test</title>
</head>
<body>
<div id='userInfo'></div>
</body>
<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
<script type="text/javascript">
	var url = "http://a.com/cors.php";
	$.get(url, {a:"getUserInfo"}, function(data) {
		$("#userInfo").text("Id:" + data.uid + " Name:" + data.name);
	}, "json");
</script>
</html>
```

Cors.php，限制cors跨域相关的字段

```php
<?php
//header('Access-Control-Allow-Origin: *');
//header('Access-Control-Allow-Credentials: true');
$a = !empty($_GET['a']) ? trim($_GET['a']) : '';
if($a == 'getUserInfo') {
        echo json_encode(array(
                'uid' => 1,
                'name' => 'mi1k7ea',
        ));
} else {
        echo '';
}
?>
```

然后访问cors.html，发现不能获取到cors.php的内容，在控制台会报错显示没有`Access-Control-Allow-Ogigin`字段。

![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/5.png)

然后，把那两句注释掉，再次访问，就可以访问到啦！

![img](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/6.png)

## CORS跨域漏洞

### 漏洞点

CORS跨域漏洞的本质，是Server配置不当。即`Access-Control-Allow-Origin`设置为`*`或是直接取自请求头`Origin`字段，Access-Control-Allow-Credentials设置为true。

感觉没什么特别的，就是没有控制策略没配置好，允许了恶意脚本。



### 检测

特点：`Access-Control-Allow-Origin: *`

或者代码里，直接从origin取，只要试着改一下origin看会不会跟着变即可。

> 还有别的吗？

## Reference
- [cors跨域漏洞总结](https://www.mi1k7ea.com/2019/08/18/CORS%E8%B7%A8%E5%9F%9F%E6%BC%8F%E6%B4%9E%E6%80%BB%E7%BB%93/)
