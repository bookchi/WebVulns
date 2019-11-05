## API鉴权方式总结

前后端分离开发模式越来越流行，往往是后端暴露出一组API供前端调用，后端根据不同的API请求进行不同的操作。不过，后端不应该对所有请求都进行操作，因为有的请求可能不合法。如何保证请求API用户的合法性？——鉴权，防止他人随意调用。下面是几种鉴权方式：

### 1 cookie-session认证

客户端首先发送用户名密码之类的给服务器，服务器认证用户身份，创建相应的session。然后返回sessionID给前端，浏览器的cookie中加入sessionid。

进行API调用时，浏览器携带sessionID发起请求，server端收到后，根据sessionID判断，用户是否有权限。

### 2 用户token认证

流程如下：

1. 客户端使用user/passwd请求登陆
2. 服务端收到请求，验证用户名和密码。陈工后，签发一个Token，然后发送给客户端。
3. 客户端收到token后，存储起来。比如放在Cookie中。客户端每次请求资源时，都要带上签发的Token
4. 服务器收到请求后，验证客户端里带的Token。



看起来，cookie-session和token鉴权很像。但其实是有不同的。

- session只是一个唯一标识的字符串，更像是一个桥梁。server通过sessionID找到对应的session，其中保存着用户的状态；

  而token本身就是一种登陆凭证，她是在登陆成功后，按照某种规则生成的信息凭证，本身就保存着用户的登录状态。server只需验证token是否合法即可。

- cookie-session机制需要cookie配合，也就是需要浏览器，他回去解析响应中的set-cookie。但是对于原生app，这就不起作用了。

  但Token不同，它是在登陆成功后，返回的相应信息。客户端收到相应时，可以把他存在本地的cookie、storage或内存中。然后下次请求的是欧，带上就可以了。

  简而言之，cookie-session限制了客户端类型，而token丰富了客户端类型。

- 时效性。sessionID在登陆时生成，并且在登出时一直不变。而token则在一段时间内可动态改变。相比之下，安全性更高。

- 可扩展性。token验证本身是比较灵活的，一是token的解决方案有许多，常用的是JWT,二来我们可以基于token验证机制，专门做一个鉴权服务，用它向多个服务的请求进行统一鉴权。

例如JSON Web TOKEN（JWT）

JWT是由Auth0提出的，通过对JSON进行加密签名来实现授权验证的方案。就是，登陆成功后将相关信息组成json对象，然后对这个对象进行某中方式的加密生成token，返回给客户端。

JWT对象通常由三部分构成：

 1. Headers：类别，加密算法

    ```json
    {
    			"alg": "HS256",
          "typ": "JWT"
    }
    ```

    2. Claims：包含要传递的用户信息

    ```json
    {
          "sub": "1234567890",
          "name": "John Doe",
          "admin": true
    }
    ```

    3. Signature：根据alg算法和私钥进行加密，得到的签名字串， 这一段是最重要的敏感信息，只能在服务端解密；

    ```
    HMACSHA256(  
        base64UrlEncode(Headers) + "." +
        base64UrlEncode(Claims),
        SECREATE_KEY
    )
    ```

    编码之后的JWT看起来是这样的一串字符：

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

> 简单来说，就是对用户的一些登陆状态进行加密，然后解密验证合法性。

### 3 请求参数认证

### 4 OAuth2.0鉴权

OAuth（开放授权）是一个开放标准，允许用户授权第三方网站，让它去访问用户存储在其他服务提供者上的信息，而不需要将用户名and密码提供给第三方网站，or分享用户数据的所有内容。

为保证用户数据的安全，第三方网站访问用户数据前，都需要显示的向用户征求授权。称阿金的提供OAuth认证服务的厂商有支付宝、QQ、微信等。

OAuth协议又有1.0和2.0两个版本。相比较1.0版，2.0版整个授权验证流程更简单更安全，也是目前最主要的用户身份验证和授权方式。 2.0大致流程如下：

![image-20190812221622789](/Users/furuijuan/Library/Application Support/typora-user-images/image-20190812221622789.png)

步骤大致如下：（CSDN为第三方）

1. 首先，第三方网站向用户请求授权。

   用户名和密码，是提供给了QQ本身。

```http
# 请求授权页面的地址
https://graph.qq.com/oauth2.0/show?which=Login&display=pc&response_type=code&client_id=100270989&redirect_uri=https%3A%2F%2Fpassport.csdn.net%2Faccount%2Flogin%3Foauth_provider%3DQQProvider&state=test
# decode URL
https://graph.qq.com/oauth2.0/show?which=Login&display=pc&response_type=code&client_id=100270989&redirect_uri=https://passport.csdn.net/account/login?oauth_provider=QQProvider&state=test
```

看到OAuth2.0常用的几个参数：

```
response_type，
client_id，第三方应用id,由授权服务器（qq）在第三方应用提交时颁发给第三方应用。
redirect_url，登陆成功重定向页面 
oath_provider，授权提供方
state，由第三方应用给出的随机码 
```

2. 返回用户凭证（code）

   当用户点击授权并登陆后，授权服务器将生成一个用户凭证（code）。这个用户凭证会附加在重定向的地址redirect_uri的后面。

   ```
   https://passport.csdn.net/account/login?code=9e3efa6cea739f9aaab2&state=XXX
   ```

3. 请求授权服务器授权

   第二步获取Code后，之后的工作就可以交给后台去处理，和用户的交互就结束了。

   第三方应用后台通过第二步的code向授权server请求Access Token，需要以下几个信息：

   ```
   client_id 标识第三方应用的id，由授权服务器（Github）在第三方应用提交时颁发给第三方应用
   client_secret 第三方应用和授权服务器之间的安全凭证，由授权服务器（Github）在第三方应用提交时颁发给第三方应用
   code 第一步中返回的用户凭证redirect_uri 第一步生成用户凭证后跳转到第二步时的地址
   state 由第三方应用给出的随机码
   ```

4. 授权server同意后，返回一个资源访问凭证Access Token

5. 第三方应用通过Access Token向资源server请求相关资源。

> 大致流程就是，首先征得用户同意，然后根据一些信息，征得授权server同意，最后证得资源server同意。



关键在于实现，原理我是懂了。但是这和API鉴权有关系吗？



## OAuth 2.0

### 应用场景

云冲印网站，可以打印用户存储在google伤的照片。但是，必须要获得用户的授权，Google才会同意云冲印去读取这些照片。怎么办呢？

传统的方法是，用户把自己的Google用户名密码告诉云冲印，他就可以读取用户照片了。但是，这有几个严重缺点。

```
1. 云冲印为了后续服务，会保存用户密码，这很不安全
2. Google不得不部署密码登陆，然而单纯的密码登陆并不安全
3. 云冲印拥有了用户在Google的所有资料的权利，而且用户没办法限制授权范围和有效期
4. 只有改密码，用户才能收回赋予的权限。但是，这样会使其他的第三方应用全部失效
5. 只要有一个第三方应用被破解，就会导致用户密码泄漏，被密码保护的数据泄漏
```

OAuth就是为了解决上面这些问题而诞生的。大致流程如下：

![image-20190813114952694](/Users/furuijuan/Library/Application Support/typora-user-images/image-20190813114952694.png)

不难看出，B是关键。即，用户怎样才能给于客户端授权。

客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0定义了四种授权方式。

- 授权码模式（authorization code）
- 简化模式（implicit）
- 密码模式（resource owner password credentials）
- 客户端模式（client credentials）

### 授权码模式

授权码模式（authorization code）是功能最完整、流程最严密的授权模式。特点就是通过客户端的后台服务器，与“服务提供商”的认证服务器进行互动。（第三方app与google认证server互动）

![image-20190813115215615](/Users/furuijuan/Library/Application Support/typora-user-images/image-20190813115215615.png)

步骤如下：

```
A 用户访问客户端，它将用户导向到认证server
B 用户选择是否给予客户端授权
C if用户同意授权，认证server将用户导向到客户端事先指定的URI-redirect uri，同时附上授权码
D 客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
E 认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。
```

下面是各个步骤需要的参数：

A步骤，客户端申请认证的URI，包含以下参数：

- response_type：表示授权类型，必选项，此处的值固定为"code"
- client_id：表示客户端的ID，必选项
- redirect_uri：表示重定向URI，可选项
- scope：表示申请的权限范围，可选项
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

例如，

```http
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

C步骤，服务器回应客户端的URI，包含以下参数

- code：授权码，必选项。该码的有效期应该很短，通常设为10min，客户端只能使用该码一次，再用会被授权server拒绝。该码与客户端ID和重定向URI，是一一对应关系。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

例如，

```http
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
          &state=xyz
```

D步骤，客户端向认证serber申请令牌的请求，需包含以下参数：

- grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
- code：表示上一步获得的授权码，必选项。
- redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
- client_id：表示客户端ID，必选项。

例如，

```http
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

E步骤中，认证服务器发送的HTTP回复，包含以下参数：

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

例如，

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
	"access_token":"2YotnFZFEjr1zCsicMWpAA",
	"token_type":"example",
	"expires_in":3600,
	"refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
	"example_parameter":"example_value"
}
```

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。此外，HTTP头信息中明确指定不得缓存。
