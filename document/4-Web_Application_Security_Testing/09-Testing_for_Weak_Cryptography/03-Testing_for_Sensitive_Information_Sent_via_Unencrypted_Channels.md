# 测试通过未加密通道发送的敏感信息

|ID          |
|------------|
|WSTG-CRYP-03|

## 概述

敏感数据在通过网络传输时必须受到保护。如果数据通过HTTPS或以另一种方式加密，保护机制不得有局限性或漏洞，如更广泛的文章[测试弱传输层安全性](01-Testing_for_Weak_Transport_Layer_Security.md)和其他OWASP文档中所解释：

- [OWASP Top 10 2017 A3-敏感数据暴露](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)。
- [OWASP ASVS - 验证V9](https://github.com/OWASP/ASVS/blob/master/4.0/en/0x17-V9-Communications.md)。
- [传输层保护速查表](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)。

根据经验，如果数据在存储时必须受到保护，则数据在传输期间也必须受到保护。敏感数据的一些示例包括：

- 用于认证的信息（例如，凭据、PIN、会话标识符、令牌、Cookie……）
- 受法律、法规或特定组织策略保护的信息（例如，信用卡、客户数据）

如果应用程序通过未加密的通道（例如HTTP）传输敏感信息，则视为安全风险。攻击者可以通过[嗅探网络流量](https://owasp.org/www-community/attacks/Manipulator-in-the-middle_attack)来接管账户。一些示例是通过HTTP以明文形式发送认证凭据的基本认证、通过HTTP发送的表单认证凭据，或由于法规、法律、组织政策或应用程序业务逻辑而被视为敏感的任何其他信息的明文传输。

个人身份信息（PII）的示例包括：

- 社会安全号码
- 银行账号
- 护照信息
- 与医疗相关的信息
- 医疗保险信息
- 学生信息
- 信用卡和借记卡号码
- 驾驶执照和州身份证信息

## 测试目标

- 识别通过各种通道传输的敏感信息。
- 评估所用通道的隐私和安全性。

## 如何测试

应用程序可能以明文形式传输的各种类型的信息必须受到保护。要检查这些信息是通过HTTP而不是HTTPS传输的，请捕获客户端与需要凭据的Web应用服务器之间的流量。对于任何包含敏感数据的消息，验证交换是使用HTTPS进行的。有关凭据的不安全传输的更多信息，请参阅[OWASP Top 10 2017 A3-敏感数据暴露](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)或[传输层保护速查表](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)。

### 示例1：通过HTTP的基本认证

一个典型的例子是通过HTTP使用基本认证。使用基本认证时，用户凭据被编码而不是加密，并作为HTTP头发送。在下面的示例中，测试人员使用[curl](https://curl.haxx.se/)测试此问题。请注意应用程序如何使用基本认证，以及使用HTTP而不是HTTPS。

```bash
$ curl -kis http://example.com/restricted/
HTTP/1.1 401 Authorization Required
Date: Fri, 01 Aug 2013 00:00:00 GMT
WWW-Authenticate: Basic realm="Restricted Area"
Accept-Ranges: bytes Vary:
Accept-Encoding Content-Length: 162
Content-Type: text/html

<html><head><title>401 Authorization Required</title></head>
<body bgcolor=white> <h1>401 Authorization Required</h1>  Invalid login credentials!  </body></html>
```

### 示例2：通过HTTP执行的基于表单的认证

另一个典型示例是通过HTTP传输用户认证凭据的认证表单。在下面的示例中，可以看到在表单的`action`属性中使用了HTTP。也可以通过使用拦截代理检查HTTP流量来查看此问题。

```html
<form action="http://example.com/login">
    <label for="username">User:</label> <input type="text" id="username" name="username" value=""/><br />
    <label for="password">Password:</label> <input type="password" id="password" name="password" value=""/>
    <input type="submit" value="Login"/>
</form>
```

### 示例3：通过HTTP发送的包含会话ID的Cookie

会话ID Cookie必须通过受保护的通道传输。如果Cookie没有设置[安全标志](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)，则允许应用程序未加密传输它。请注意，在没有安全标志的情况下设置cookie，整个登录过程在HTTP而不是HTTPS中执行。

```http
https://secure.example.com/login

POST /login HTTP/1.1
Host: secure.example.com
[...]
Referer: https://secure.example.com/
Content-Type: application/x-www-form-urlencoded
Content-Length: 188

HTTP/1.1 302 Found
Date: Tue, 03 Dec 2013 21:18:55 GMT
Server: Apache
Set-Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA; expires=Fri, 01-Jan-2014 00:00:00 GMT; path=/; domain=example.com; httponly
Location: private/
Content-Length: 0
Content-Type: text/html
```

```http
http://example.com/private

GET /private HTTP/1.1
Host: example.com
[...]
Referer: https://secure.example.com/login
Cookie: JSESSIONID=BD99F321233AF69593EDF52B123B5BDA;

HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 730
Date: Tue, 25 Dec 2013 00:00:00 GMT
```

### 示例4：通过HTTP的密码重置、更改密码或其他账户操作

如果Web应用程序具有允许用户更改账户或使用凭据调用其他服务的功能，请验证所有这些交互都使用HTTPS。要测试的交互包括：

- 允许用户处理忘记密码或其他凭据的表单
- 允许用户编辑凭据的表单
- 要求用户使用其他提供者（例如支付处理）进行认证的表单

### 示例5：在源代码或日志中测试密码敏感信息

使用以下技术之一搜索敏感信息。

检查密码或加密密钥是否硬编码在源代码或配置文件中。

`grep -r –E "Pass | password | pwd |user | guest| admin | encry | key | decrypt | sharekey " ./PathToSearch/`

检查日志或源代码是否可能包含电话号码、电子邮件地址、ID或任何其他PII。根据PII的格式更改正则表达式。

`grep -r " {2\}[0-9]\{6\} "  ./PathToSearch/`

## 修复

为整个网站使用HTTPS，并将任何HTTP请求重定向到HTTPS。

## 工具

- [curl](https://curl.haxx.se/)
- [grep](https://man7.org/linux/man-pages/man1/egrep.1.html)
- [Wireshark](https://www.wireshark.org/)
- [TCPDUMP](https://www.tcpdump.org/)

## 参考资料

- [OWASP 不安全传输](https://owasp.org/www-community/vulnerabilities/Insecure_Transport)
- [OWASP HTTP严格传输安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [让我们加密](https://letsencrypt.org)
