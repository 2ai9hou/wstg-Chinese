# 识别应用入口点

|ID          |
|------------|
|WSTG-INFO-06|

## 摘要

枚举应用及其攻击面是进行任何彻底测试之前的关键前提，因为它允许测试人员识别可能的弱点区域。本节旨在帮助识别和描绘应用中在完成枚举和映射后需要调查的区域。

## 测试目标

- 通过请求和响应分析识别可能的入口和注入点。

## 如何测试

在任何测试开始之前，测试人员应始终对应用以及用户和浏览器如何与其通信有一个很好的理解。当测试人员浏览应用时，他们应注意所有 HTTP 请求以及传递给应用的每个参数和表单字段。他们应特别注意何时使用 GET 请求以及何时使用 POST 请求将参数传递给应用。此外，他们还需要注意何时使用 RESTful 服务的其他方法。

请注意，为了查看在 POST 请求正文中发送的参数，测试人员可能需要使用拦截代理等工具（参见[工具](#工具)）。在 POST 请求中，测试人员还应特别注意任何隐藏的表单字段，因为这些通常包含敏感信息，如状态信息、物项数量、物项价格等，而开发人员从未打算让任何人看到或更改这些信息。

在此测试阶段，使用拦截代理和笔记应用（如电子表格软件）在测试人员中很流行。代理将跟踪测试人员和应用之间的每个请求和响应。此外，在这一点上，测试人员通常会捕获每个请求和响应，以便他们能看到传递给应用的每个头、参数等，以及返回的内容。这有时可能相当乏味，特别是在大型交互式站点上（想象一个银行应用）。然而，经验会显示要寻找什么，这一阶段可以显著优化。

当测试人员浏览应用时，他们应注意 URL 中任何有趣的参数、自定义头或请求/响应的正文，并将它们保存在电子表格中。电子表格应包括请求的页面（添加来自代理的请求编号以供将来参考可能也很好）、有趣的参数、请求类型（GET、POST 等）、是否经过认证/未认证、是否使用 TLS、是否是多重步骤的一部分、是否使用 WebSocket 以及任何其他相关注释。一旦他们绘制出应用的所有区域，他们就可以遍历应用并测试他们已识别的每个区域，并记录哪些有效哪些无效。本指南的其余部分将识别如何测试这些区域，但必须在开始任何实际测试之前进行本节所述的工作。

以下是对所有请求和响应的一些兴趣点。在请求部分，专注于 GET 和 POST 方法，因为它们构成大部分请求。请注意，其他方法（如 PUT 和 DELETE）也可能找到。通常，如果允许此类较不常见的请求，它们可能暴露漏洞。本指南有一个专门用于测试这些 HTTP 方法的特别部分。

### 请求

- 识别在哪里使用 GET 和在哪里使用 POST。
- 识别 POST 请求中使用的所有参数（这些在请求正文中）。
- 在 POST 请求中，特别注意任何隐藏参数。当发送 POST 时，所有表单字段（包括隐藏参数）将在 HTTP 消息的正文中发送到应用。这些通常看不到，除非使用代理或查看 HTML 源代码。更改这些隐藏参数可能会更改随后加载的页面、它们包含的数据以及授予的访问程度。
- 识别 GET 请求中使用的所有参数（即，在 URL 中），特别是查询字符串（通常出现在 ? 标记之后）。
- 识别查询字符串的所有参数。这些通常采用 `foo=bar` 的成对格式。还要注意，许多参数可以由一个 `&`、`\~`、`:` 或任何其他特殊字符或编码在一个查询字符串中分隔。
- 请注意，当识别一个字符串或 POST 请求中的多个参数时，部分或全部参数将需要执行攻击。测试人员需要识别所有参数（即使它们被编码或加密）并识别哪些由应用处理。指南的后面部分将介绍如何测试这些参数。在这一点上，重要的是确保识别每个参数。
- 还要注意任何额外或自定义的非典型头（如 `debug: false`）。

### 响应

- 识别在哪里设置新 Cookie（`Set-Cookie` 头）、修改或添加。
- 识别任何重定向（3xx HTTP 状态码）、400 状态码（特别是 403 禁止）和正常响应（即未修改请求）期间的 500 内部服务器错误。
- 还要注意任何有趣的头使用。例如，`Server: BIG-IP` 表示站点是负载平衡的。因此，如果站点是负载平衡的且一个服务器配置错误，则测试人员可能需要发出多个请求以访问易受攻击的服务器，具体取决于所使用的负载平衡类型。

### OWASP 攻击面检测器

攻击面检测器（ASD）工具调查源代码并揭示 Web 应用的端点、这些端点接受的参数以及这些参数的数据类型。这包括蜘蛛无法找到的非链接端点，以及在客户端代码中完全未使用的可选参数。它还具有计算两个版本应用之间攻击面变化的能力。

攻击面检测器可作为 ZAP 和 Burp Suite 的插件使用，也可作为命令行工具使用。命令行工具将攻击面导出为 JSON 输出，然后可由 ZAP 和 Burp Suite 插件使用。这对于不直接向渗透测试人员提供源代码的情况很有帮助。例如，渗透测试人员可以从不希望提供源代码本身的客户处获取 json 输出文件。

#### 如何使用

CLI jar 文件可从 [https://github.com/secdec/attack-surface-detector-cli/releases](https://github.com/secdec/attack-surface-detector-cli/releases) 下载。

您可以运行以下命令让 ASD 从目标 Web 应用的源代码中识别端点。

`java -jar attack-surface-detector-cli-1.3.5.jar <source-code-path> [flags]`

以下是针对 [OWASP RailsGoat](https://github.com/OWASP/railsgoat) 运行命令的示例。

```text
$ java -jar attack-surface-detector-cli-1.3.5.jar railsgoat/
Beginning endpoint detection for '<...>/railsgoat' with 1 framework types
Using framework=RAILS
[0] GET: /login (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[1] GET: /logout (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[2] POST: /forgot_password (0 variants): PARAMETERS={email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '29'-'38')
[3] GET: /password_resets (0 variants): PARAMETERS={token=name=token, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '19'-'27')
[4] POST: /password_resets (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user=name=user, paramType=QUERY_STRING, dataType=STRING, confirm_password=name=confirm_password, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/password_resets_controller.rb (lines '5'-'17')
[5] GET: /sessions/new (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
[6] POST: /sessions (0 variants): PARAMETERS={password=name=password, paramType=QUERY_STRING, dataType=STRING, user_id=name=user_id, paramType=SESSION, dataType=STRING, remember_me=name=remember_me, paramType=QUERY_STRING, dataType=STRING, url=name=url, paramType=QUERY_STRING, dataType=STRING, email=name=email, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '11'-'31')
[7] DELETE: /sessions/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/sessions_controller.rb (lines '33'-'37')
[8] GET: /users (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '9'-'11')
[9] GET: /users/{id} (0 variants): PARAMETERS={}; FILE=/app/controllers/api/v1/users_controller.rb (lines '13'-'15')
... snipped ...
[38] GET: /api/v1/mobile/{id} (0 variants): PARAMETERS={id=name=id, paramType=QUERY_STRING, dataType=STRING, class=name=class, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/api/v1/mobile_controller.rb (lines '8'-'13')
[39] GET: / (0 variants): PARAMETERS={url=name=url, paramType=QUERY_STRING, dataType=STRING}; FILE=/app/controllers/sessions_controller.rb (lines '6'-'9')
Generated 40 distinct endpoints with 0 variants for a total of 40 endpoints
Successfully validated serialization for these endpoints
0 endpoints were missing code start line
0 endpoints were missing code end line
0 endpoints had the same code start and end line
Generated 36 distinct parameters
Generated 36 total parameters
- 36/36 have their data type
- 0/36 have a list of accepted values
- 36/36 have their parameter type
--- QUERY_STRING: 35
--- SESSION: 1
Finished endpoint detection for '<...>/railsgoat'
----------
-- DONE --
0 projects had duplicate endpoints
Generated 40 distinct endpoints
Generated 40 total endpoints
Generated 36 distinct parameters
Generated 36 total parameters
1/1 projects had endpoints generated
To enable logging include the -debug argument
```

您还可以使用 `-json` 标志生成 JSON 输出文件，可由 ZAP 和 Burp Suite 的插件使用。有关更多详细信息，请参阅以下链接。

- [ZAP 插件 ASD 主页](https://github.com/secdec/attack-surface-detector-zap/wiki)
- [PortSwigger Burp 的 ASD 插件主页](https://github.com/secdec/attack-surface-detector-burp/wiki)

### 测试应用入口点

以下是如何检查应用入口点的两个示例。

#### 示例 1

此示例显示将购买网上购物应用中物品的 GET 请求。

```http
GET /shoppingApp/buyme.asp?CUSTOMERID=100&ITEM=z101a&PRICE=62.50&IP=x.x.x.x HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=Z29vZCBqb2IgcGFkYXdhIG15IHVzZXJuYW1lIGlzIGZvbyBhbmQgcGFzc3dvcmQgaXMgYmFy
```

> 请求的所有参数（如 CUSTOMERID、ITEM、PRICE、IP 和 Cookie，这些可能只是编码参数或用于会话状态的参数）。

#### 示例 2

此示例显示将让您登录应用的 POST 请求。

```http
POST /example/authenticate.asp?service=login HTTP/1.1
Host: x.x.x.x
Cookie: SESSIONID=dGhpcyBpcyBhIGJhZCBhcHAgdGhhdCBzZXRzIHByZWRpY3RhYmxlIGNvb2tpZXMgYW5kIG1pbmUgaXMgMTIzNA==;CustomCookie=00my00trusted00ip00is00x.x.x.x00

user=admin&pass=pass123&debug=true&fromtrustIP=true
```

可以注意到参数在几个位置发送：

1. 在查询字符串中：`service`
2. 在 Cookie 头中：`SESSIONID`、`CustomCookie`
3. 在请求正文中：`user`、`pass`、`debug`、`fromtrustIP`

多种注入位置为攻击者提供了链接可能性，可以提高在处理代码中发现漏洞的机会。

## 工具

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
- [Burp Suite](https://www.portswigger.net/burp/)
- [Fiddler](https://www.telerik.com/fiddler)

## 参考资料
