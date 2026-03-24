# 测试会话固定

|ID          |
|------------|
|WSTG-SESS-03|

## 概述

会话固定源于在认证前后保持会话Cookie相同值的不安全做法。这通常发生在会话Cookie用于在登录前存储状态信息时，例如在认证支付之前将商品添加到购物车。

在会话固定漏洞的通用利用中，攻击者可以在不首先认证的情况下从目标站点获取一组会话Cookie。攻击者可以使用不同技术将这些Cookie强制注入受害者的浏览器。如果受害者在目标站点稍后进行认证，并且Cookie在登录时未刷新，则受害者将被攻击者选择的会话Cookie识别。攻击者随后能够使用这些已知Cookie冒充受害者。

可以通过在认证过程后刷新会话Cookie来修复此问题。或者，可以通过确保会话Cookie的完整性来防止攻击。在考虑网络攻击者（即控制受害者使用的网络的攻击者）时，使用完整的[HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)或向Cookie名称添加`__Host-` / `__Secure-`前缀。

完整的HSTS采用发生在主机为自己及其所有子域激活HSTS时。这在一篇名为*Testing for Integrity Flaws in Web Sessions*的论文中描述，作者是Stefano Calzavara、Alvise Rabitti、Alessio Ragazzo和Michele Bugliesi。

## 测试目标

- 分析认证机制及其流程。
- 强制使用Cookie并评估影响。

## 如何测试

在本节中，我们将解释将在下一节中显示的测试策略。

第一步是向要测试的站点发送请求（例如`www.example.com`）。如果测试人员请求以下内容：

```http
GET / HTTP/1.1
Host: www.example.com
```

他们将获得以下响应：

```html
HTTP/1.1 200 OK
Date: Wed, 14 Aug 2008 08:45:11 GMT
Server: IBM_HTTP_Server
Set-Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1; Path=/; secure
Cache-Control: no-cache="set-cookie,set-cookie2"
Expires: Thu, 01 Dec 1994 16:00:00 GMT
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html;charset=Cp1254
Content-Language: en-US
```

应用为客户端设置了一个新的会话标识符`JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1`。

接下来，如果测试人员通过以下POST成功认证到应用：`https://www.example.com/authentication.php`：

```http
POST /authentication.php HTTP/1.1
Host: www.example.com
[...]
Referer: https://www.example.com
Cookie: JSESSIONID=0000d8eyYq3L0z2fgq10m4v-rt4:-1
Content-Type: application/x-www-form-urlencoded
Content-length: 57

Name=Meucci&wpPassword=secret!&wpLoginattempt=Log+in
```

测试人员观察到服务器的以下响应：

```http
HTTP/1.1 200 OK
Date: Thu, 14 Aug 2008 14:52:58 GMT
Server: Apache/2.2.2 (Fedora)
X-Powered-By: PHP/5.1.6
Content-language: en
Cache-Control: private, must-revalidate, max-age=0
X-Content-Encoding: gzip
Content-length: 4090
Connection: close
Content-Type: text/html; charset=UTF-8
...
HTML data
...
```

由于在成功认证后没有颁发新的Cookie，测试人员知道除非确保会话Cookie的完整性，否则可以进行会话劫持。

测试人员可以将有效的会话标识符发送给用户（可能使用社会工程技巧），等待他们认证，然后验证权限是否已分配给此Cookie。

### 使用强制Cookie进行测试

此测试策略针对网络攻击者，因此只需应用于没有完整HSTS采用的站点（具有完整HSTS采用的站点是安全的，因为它们的所有Cookie都具有完整性）。我们假设在测试站点上有两个测试账户，一个充当受害者，一个充当攻击者。我们模拟一个场景，攻击者在受害者的浏览器中强制所有在登录后未新鲜颁发且没有完整性的Cookie。在受害者登录后，攻击者向站点呈现强制Cookie以访问受害者的账户：如果这些Cookie足以代表受害者行事，则会话固定是可能的。

以下是执行此测试的步骤：

1. 到达站点的登录页面。
2. 在登录前保存Cookie存储的快照，不包括名称中包含`__Host-`或`__Secure-`前缀的Cookie。
3. 以受害者身份登录并到达任何需要认证的安全功能页面。
4. 将Cookie存储设置为步骤2中拍摄的快照。
5. 触发步骤3中识别的安全功能。
6. 观察步骤5中的操作是否成功执行。如果成功，攻击成功。
7. 清除Cookie存储，以攻击者身份登录并到达步骤3中的页面。
8. 逐个将步骤2中保存的Cookie写入Cookie存储。
9. 再次触发步骤3中识别的安全功能。
10. 清除Cookie存储，再次以受害者身份登录。
11. 观察步骤9中的操作是否在受害者的账户中成功执行。如果成功，攻击成功；否则，站点可以防御会话固定。

我们建议为受害者和攻击者使用两台不同的机器或浏览器。这允许您在Web应用进行指纹识别以验证从给定Cookie启用的访问时减少误报。测试策略的一个更短但精度较低的变体只需要一个测试账户。它遵循相同的步骤，但在步骤6停止。

## 修复方案

在用户成功认证后实施会话令牌更新。

应用应首先使现有会话ID失效，然后在对用户进行认证，如果认证成功，则提供另一个会话ID。

## 工具

- [ZAP](https://www.zaproxy.org)

## 参考资料

- [Session Fixation](https://owasp.org/www-community/attacks/Session_fixation)
- [ACROS Security](https://www.acrossecurity.com/papers/session_fixation.pdf)
- [Chris Shiflett](https://shiflett.org/articles/session-fixation)
