# 测试HTTP响应拆分

|ID          |
|------------|
|WSTG-INPV-15|

## 概述

HTTP响应拆分漏洞发生在应用程序将未清理的用户输入合并到HTTP响应头中时，允许攻击者注入回车符（CR）和换行符（LF）字符。结果，单个HTTP响应可以被客户端或中间系统解释为多个不同的响应。

成功利用HTTP响应拆分可能导致多种影响，包括Web缓存中毒、跨站脚本（XSS）、内容欺骗、会话固定或其他客户端攻击，这取决于如何处理注入的响应。

本节专门关注在应用程序层识别和测试HTTP响应拆分漏洞。HTTP请求走私（依赖于多个HTTP代理之间的解析差异）在另一章节中介绍。

## 测试目标

- 识别反映到HTTP响应头中的用户控制输入。
- 评估是否可以注入CR（`\r`）和LF（`\n`）字符到响应头。
- 确定成功HTTP响应拆分攻击的潜在影响，如缓存中毒或客户端利用。

## 如何测试

### 黑盒测试

某些Web应用程序使用用户提供的输入来生成某些HTTP响应头的值。一个常见的例子是重定向逻辑，其中目标URL从请求参数派生。

例如，假设要求用户选择标准或高级界面。选择的选项作为参数传递并反映在重定向响应头中。

如果参数`interface`的值为`advanced`，应用程序可能响应：

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
```

当浏览器收到此响应时，它会遵循`Location`头中指定的URL。但是，如果应用程序不正确验证或清理用户输入，攻击者可能会注入表示用于分隔HTTP头行的CRLF字符的序列`%0d%0a`。

通过注入CRLF序列，测试人员可能导致响应被下游客户端或中间系统解释为两个不同的HTTP响应。这种行为可用于毒化缓存或向用户交付恶意内容。

例如，测试人员为`interface`参数提供以下值：

`advanced%0d%0aContent-Length:%200%0d%0a%0d%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2035%0d%0a%0d%0a<html>Sorry,%20System%20Down</html>`

易受攻击的应用程序的结果响应可能是：

```http
HTTP/1.1 302 Moved Temporarily
Date: Sun, 03 Dec 2005 16:22:19 GMT
Location: https://victim.com/main.jsp?interface=advanced
Content-Length: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 35

<html>Sorry,%20System%20Down</html>
```

处理此响应的Web缓存可能将其解释为两个不同的响应。如果攻击者立即发起对`/index.html`的请求，缓存可能将该请求与其第二个响应相关联并存储它。结果，所有随后通过该缓存访问`victim.com/index.html`的用户可能收到攻击者控制的内容。

或者，攻击者可以注入JavaScript payload，对缓存提供服务的用户执行跨站脚本攻击。虽然漏洞存在于应用程序中，但主要目标是其用户。

要识别此问题，测试人员应找到所有影响HTTP响应头的用户控制输入，并验证是否可以注入CRLF序列。

与HTTP响应拆分最常见的响应头包括：

- `Location`
- `Set-Cookie`

在实际场景中成功利用可能需要仔细考虑其他因素：

- 测试人员可能需要制作适合缓存的响应头（例如，将`Last-Modified`设置为未来的日期），并可能使用诸如`Pragma: no-cache`之类的头使现有缓存条目无效。
- 应用程序可能过滤CRLF字符，但允许替代编码或字符表示，有时可用于绕过输入验证。
- 某些平台URL编码响应头的一部分（如`Location`头中的路径），同时保留查询字符串未编码，允许通过URL的特定组件进行注入。

有关此类攻击和其他利用场景的深入讨论，请参阅参考资料部分列出的白皮书。

### 灰盒测试

在灰盒测试场景中，应用程序架构和服务器行为的知识提高了利用可靠性。

不同的服务器或中间件可能以不同方式确定消息边界（例如，使用固定大小的缓冲区），需要精确的偏移量或填充。当易受攻击的参数通过GET传输时，URL长度限制可能会截断有效载荷。测试人员应识别替代注入点或请求方法（如POST），以更好地控制有效载荷的长度和定位。

## 修复

确保用户提供的输入从不放置到HTTP头中，而无需严格验证和清理。

- **输入验证：** 在将包含回车（`\r`、`%0d`）或换行（`\n`、`%0a`）字符的输入用于HTTP头之前，拒绝或剥离它们。
- **URL编码：** 如果输入是URL的一部分（如`Location`头中），确保其被正确URL编码，以防止控制字符被解释为分隔符。
- **使用安全框架：** 使用框架内置的设置头函数（如`setHeader()`、`addHeader()`）而不是手动构造原始HTTP响应字符串。现代环境通常默认阻止头注入。

## 工具

- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net/burp)
- [CRLFuzz](https://github.com/dwisiswant0/crlfuzz) - 专门设计用于扫描CRLF漏洞的工具。
- [Nuclei](https://github.com/projectdiscovery/nuclei) - 可用于使用特定模板检测CRLF注入模式。

## 参考资料

- [Amit Klein，"分而治之：HTTP响应拆分、Web缓存中毒攻击及相关主题"](https://packetstormsecurity.com/files/32815/Divide-and-Conquer-HTTP-Response-Splitting-Whitepaper.html)
- [Amit Klein："HTTP消息拆分、走私及其他动物"](https://www.slideserve.com/alicia/http-message-splitting-smuggling-and-other-animals-powerpoint-ppt-presentation)
- [Amit Klein："HTTP请求走私 - ERRATA（IIS 48K缓冲区现象）"](https://web.archive.org/web/20210614052317/https://www.securityfocus.com/archive/1/411418)
- [Amit Klein："HTTP响应走私"](https://web.archive.org/web/20210126213458/https://www.securityfocus.com/archive/1/425593)
- [Chaim Linhart, Amit Klein, Ronen Heled, Steve Orrin:"HTTP请求走私"](https://web.archive.org/web/20210816212852/https://www.cgisecurity.com/lib/http-request-smuggling.pdf)
