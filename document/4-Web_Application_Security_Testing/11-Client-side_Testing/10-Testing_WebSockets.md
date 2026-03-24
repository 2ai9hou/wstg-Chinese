# 测试WebSockets

|ID          |
|------------|
|WSTG-CLNT-10|

## 概述

传统上，HTTP协议每个TCP连接只允许一个请求/响应。异步JavaScript和XML（AJAX）允许客户端异步（在后台无需页面刷新）向服务器发送和接收数据，但是，AJAX要求客户端发起请求并等待服务器响应（半双工）。

[WebSockets](https://html.spec.whatwg.org/multipage/web-sockets.html#network)允许客户端或服务器创建"全双工"（双向）通信通道，允许客户端和服务器真正异步通信。WebSockets通过HTTP进行初始*升级*握手，之后所有通信通过TCP通道使用帧进行。有关详细信息，请参阅[WebSocket协议](https://tools.ietf.org/html/rfc6455)。

### Origin

服务器有责任验证初始HTTP WebSocket握手中的[`Origin`头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)。如果服务器不验证初始WebSocket握手中的origin头，WebSocket服务器可能接受来自任何源的连接。这可能允许攻击者与WebSocket服务器进行跨域通信，从而允许类似CSRF的问题。另请参阅[Top 10-2017 A5-访问控制损坏](https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control)。此弱点的利用称为跨站WebSocket劫持（CSWH或CSWSH）。

### 保密性和完整性

WebSockets可通过未加密的TCP或通过加密的TLS使用。要使用未加密的WebSockets，使用`ws://` URI方案（默认端口80），要使用加密（TLS）WebSockets，使用`wss://` URI方案（默认端口443）。另请参阅[Top 10-2017 A3-敏感数据暴露](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)。

### 输入清理

与来自不受信任来源的任何数据一样，数据应适当清理和编码。另请参阅[Top 10-2017 A1-注入](https://owasp.org/www-project-top-ten/2017/A1_2017-Injection)和[Top 10-2017 A7-跨站脚本（XSS）](https://owasp.org/www-project-top-ten/2017/A7_2017-Cross-Site_Scripting_(XSS))。

## 测试目标

- 识别WebSockets的使用。
- 通过使用与正常HTTP通道相同的测试评估其实现。

## 如何测试

### 黑盒测试

1. 识别应用程序是否使用WebSockets。
   - 检查客户端源代码中`ws://`或`wss://` URI方案。
   - 使用Google Chrome的开发者工具查看网络WebSocket通信。
   - 使用[ZAP的](https://www.zaproxy.org)WebSocket选项卡。
2. Origin。
   - 使用WebSocket客户端（可在下面的工具部分找到一个）尝试连接到远程WebSocket服务器。如果建立连接，服务器可能未检查origin头。
3. 保密性和完整性。
   - 检查WebSocket连接是否使用TLS传输敏感信息`wss://`。
   - 检查HTTPS实现的安全问题（有效证书、BEAST、CRIME、RC4等）。请参阅本指南的[测试弱传输层安全性](../09-Testing_for_Weak_Cryptography/01-Testing_for_Weak_Transport_Layer_Security.md)部分。
4. 认证。
   - WebSockets不处理身份验证，应进行正常的黑盒身份验证测试。请参阅本指南的[认证测试](../04-Authentication_Testing/README.md)部分。
5. 授权。
   - WebSockets不处理授权，应进行正常的黑盒授权测试。请参阅本指南的[授权测试](../05-Authorization_Testing/README.md)部分。
6. 输入清理。
   - 使用[ZAP的](https://www.zaproxy.org)WebSocket选项卡来重放和模糊WebSocket请求和响应。请参阅本指南的[测试数据验证](../07-Input_Validation_Testing/README.md)部分。

#### 示例1

一旦我们识别出应用程序正在使用WebSockets（如上所述），我们可以使用[Zed Attack Proxy (ZAP)](https://www.zaproxy.org)拦截WebSocket请求和响应。ZAP可以用于重放和模糊WebSocket请求/响应。

![ZAP WebSockets](images/OWASP_ZAP_WebSockets.png)\
*图4.11.10-1：ZAP WebSockets*

#### 示例2

使用WebSocket客户端（可在下面的工具部分找到一个）尝试连接到远程WebSocket服务器。如果允许连接，WebSocket服务器可能未检查WebSocket握手的origin头。尝试重放先前拦截的请求，以验证跨域WebSocket通信是否可能。

![WebSocket客户端](images/WebSocket_Client.png)\
*图4.11.10-2：WebSocket客户端*

### 灰盒测试

灰盒测试类似于黑盒测试。在灰盒测试中，渗透测试人员具有应用程序的部分知识。唯一的区别是，您可能有应用程序的API文档，其中包括预期的WebSocket请求和响应。

## 工具

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [WebSocket客户端](https://github.com/ethicalhack3r/scripts/blob/master/WebSockets.html)
- [Google Chrome简单WebSocket客户端](https://chrome.google.com/webstore/detail/simple-websocket-client/pfdhoblngboilpfeibdedpjgfnlcodoo?hl=en)

## 参考资料

- [HTML5 Rocks - 介绍WebSockets：将套接字带到Web](https://www.html5rocks.com/en/tutorials/websockets/basics/)
- [W3C - WebSocket API](https://html.spec.whatwg.org/multipage/web-sockets.html#network)
- [IETF - WebSocket协议](https://tools.ietf.org/html/rfc6455)
- [CWE-1385：WebSockets中缺少Origin验证](https://cwe.mitre.org/data/definitions/1385.html)
- [Christian Schneider - 跨站WebSocket劫持（CSWSH）](https://www.christian-schneider.net/blog/cross-site-websocket-hijacking/)
- [Robert Koch - 论渗透测试中的WebSockets](https://repositum.tuwien.at/retrieve/21955)
- [DigiNinja - ZAP和Web套接字](https://digi.ninja/blog/zap_web_sockets.php)
