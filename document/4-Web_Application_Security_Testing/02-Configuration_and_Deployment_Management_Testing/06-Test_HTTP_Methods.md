# 测试 HTTP 方法

|ID          |
|------------|
|WSTG-CONF-06|

## 概述

HTTP 提供了许多可用于在 Web 服务器上执行操作的方法（或动词）。虽然 GET 和 POST 是用于访问 Web 服务器提供的信息的最常见方法，但还有其他各种方法也可能被支持，有时会被攻击者利用。

[RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231) 定义了主要的有效 HTTP 请求方法（或动词），尽管其他 RFC 中添加了额外的方法，如 [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789)。在 [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) 应用程序中，这些动词中有几个被重新用于不同目的，下表列出了这些用途。

| 方法 | 原始用途 | RESTful 用途 |
|--------|------------------|-----------------|
| [`GET`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.1) | 请求文件。 | 请求对象。|
| [`HEAD`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2) | 请求文件，但只返回 HTTP 头。 | |
| [`POST`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.3) | 提交数据。 | |
| [`PUT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.4) | 上传文件。 | 创建对象。 |
| [`DELETE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.5) | 删除文件 | 删除对象。 |
| [`CONNECT`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.6) | 建立到另一个系统的连接。 | |
| [`OPTIONS`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.7) | 列出支持的 HTTP 方法。 | 执行 [CORS 预检](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request) 请求。 |
| [`TRACE`](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.8) | 回显 HTTP 请求以进行调试。 | |
| [`PATCH`](https://datatracker.ietf.org/doc/html/rfc5789#section-2) |  | 修改对象。 |

## 测试目标

- 枚举支持的 HTTP 方法。
- 测试访问控制绕过。
- 测试 HTTP 方法覆盖技术。

## 如何测试

### 发现支持的方法

要执行此测试，测试人员需要一种方法来识别正在检查的 Web 服务器支持哪些 HTTP 方法。最简单的方法是向服务器发出 `OPTIONS` 请求：

```http
OPTIONS / HTTP/1.1
Host: example.org
```

服务器应该响应支持的方法列表：

```http
HTTP/1.1 200 OK
Allow: OPTIONS, GET, HEAD, POST
```

但是，并非所有服务器都响应 OPTIONS 请求，有些甚至可能返回不准确的信息。还值得注意的是，服务器可能对不同路径支持不同的方法。这意味着即使某种方法不支持根目录 /，也不一定表示该方法在其他地方不被支持。

更可靠的测试支持方法的方式是使用该方法类型发出简单请求，并检查服务器响应。如果不允许该方法，服务器应返回 `405 Method Not Allowed` 状态。

请注意，某些服务器将未知方法视为等同于 `GET`，因此它们可能响应任意方法，如下面所示的请求。这有时可用于绕过 Web 应用程序防火墙或任何阻止特定方法的其他过滤。

```http
FOO / HTTP/1.1
Host: example.org
```

也可以使用 curl 的 `-X` 选项发出带有任意方法的请求：

```bash
curl -X FOO https://example.org
```

也有各种自动化工具可以尝试确定支持的方法，例如 [`http-methods`](https://nmap.org/nsedoc/scripts/http-methods.html) Nmap 脚本。但是，这些工具可能不会测试危险方法（即可能引起更改的方法，如 `PUT` 或 `DELETE`），或者如果这些方法被支持，可能会无意中导致对 Web 服务器的更改。因此，应谨慎使用这些工具。

### PUT 和 DELETE

`PUT` 和 `DELETE` 方法可能有不同的效果，这取决于它们是被 Web 服务器还是运行在其上的应用程序解释。

#### 传统 Web 服务器

一些传统 Web 服务器允许使用 `PUT` 方法在服务器上创建文件。例如，如果服务器配置为允许此操作，以下请求将在服务器上创建一个名为 `test.html` 的文件，内容为 `<script>alert(1)</script>`。

```http
PUT /test.html HTTP/1.1
Host: example.org
Content-Length: 25

<script>alert(1)</script>
```

类似的请求也可以使用 cURL 完成：

```bash
curl https://example.org --upload-file test.html
```

这允许攻击者将任意文件上传到 Web 服务器，如果允许上传可执行代码（如 PHP 文件），这可能会导致完全系统受损。但是，这种配置非常罕见，不太可能在现代系统上看到。

同样，`DELETE` 方法可用于从 Web 服务器删除文件。请注意，这是一个**破坏性操作**；因此，测试此方法时应格外小心。

```http
DELETE /test.html HTTP/1.1
Host: example.org
```

或使用 cURL：

```bash
curl https://example.org/test.html -X DELETE
```

#### RESTful API

相比之下，`PUT` 和 `DELETE` 方法通常由现代 RESTful 应用程序用于创建和删除对象。例如，以下 API 请求可用于创建名为 "foo" 且角色为 "user" 的用户：

```http
PUT /api/users/foo HTTP/1.1
Host: example.org
Content-Length: 34

{
    "role": "user"
}
```

带有 DELETE 方法的类似请求可用于删除对象。

```http
DELETE /api/users/foo HTTP/1.1
Host: example.org
```

尽管自动化扫描工具可能会报告，但在 RESTful API 上存在这些方法**不是安全问题**。但是，此功能可能有其他漏洞（如弱访问控制），应进行彻底测试。

### TRACE

`TRACE` 方法（或 Microsoft 的等效 `TRACK` 方法）导致服务器回显请求的内容。这导致了一个名为跨站追踪（XST）的漏洞在 [2003](https://www.cgisecurity.com/whitehat-mirror/WH-WhitePaper_XST_ebook.pdf)（PDF）中被公开，该漏洞可用于访问设置了 `HttpOnly` 标志的 cookie。`TRACE` 方法已在所有浏览器和插件中被阻止多年；因此，此问题不再可利用。但是，自动化扫描工具可能仍会标记此问题，并且在 Web 服务器上启用 `TRACE` 方法表明它未被正确强化。

### CONNECT

`CONNECT` 方法导致 Web 服务器打开到另一个系统的 TCP 连接，然后将流量从客户端传递到该系统。这可能允许攻击者通过服务器代理流量，以隐藏其源地址、访问内部系统或访问绑定到 localhost 的服务。`CONNECT` 请求的示例如下：

```http
CONNECT 192.168.0.1:443 HTTP/1.1
Host: example.org
```

### PATCH

`PATCH` 方法在 [RFC 5789](https://datatracker.ietf.org/doc/html/rfc5789) 中定义，用于提供关于应如何修改对象的指令。RFC 本身没有定义这些指令应该是什么格式，但在其他标准中定义了各种方法，例如 [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://datatracker.ietf.org/doc/html/rfc6902)。

例如，如果我们有一个名为 "foo" 的用户，具有以下属性：

```json
{
    "role": "user",
    "email": "foo@example.org"
}
```

以下 JSON PATCH 请求可用于将此用户的角色更改为 "admin"，而不修改电子邮件地址：

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{ "op": "replace", "path": "/role", "value": "admin" }
```

尽管 RFC 说明它应该包含关于应如何修改对象的指令，但 `PATCH` 方法通常（被滥用）包含更改的内容，如下所示。与之前的请求类似，这将把 "role" 值更改为 "admin"，而不修改对象的其他部分。这与 `PUT` 方法形成对比，`PUT` 方法会覆盖整个对象，因此结果将是一个没有 "email" 属性的对象。

```http
PATCH /api/users/foo HTTP/1.1
Host: example.org

{
    "role": "admin"
}
```

与 `PUT` 方法一样，此功能可能有访问控制弱点或其他漏洞。此外，应用程序在修改对象时可能不会执行与创建对象时相同的输入验证。这可能潜在地允许注入恶意值（如存储型跨站脚本攻击中所述），或允许可能导致业务逻辑相关问题的损坏或无效对象。

### 测试访问控制绕过

如果应用程序页面在用户尝试直接访问时使用 302 代码重定向到登录页面，则可以通过使用不同的 HTTP 方法（如 `HEAD`、`POST` 或甚至是虚构的方法如 `FOO`）发出请求来绕过此限制。如果 Web 应用程序响应 `HTTP/1.1 200 OK` 而不是预期的 `HTTP/1.1 302 Found`，则可能绕过身份验证或授权。以下示例显示了 `HEAD` 请求如何导致页面设置管理 cookie，而不是将用户重定向到登录页面：

```http
HEAD /admin/ HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 200 OK
[...]
Set-Cookie: adminSessionCookie=[...];
```

或者，可能可以直接请求导致操作的页面，例如：

```http
HEAD /admin/createUser.php?username=foo&password=bar&role=admin HTTP/1.1
Host: example.org
```

或：

```http
FOO /admin/createUser.php
Host: example.org
Content-Length: 36

username=foo&password=bar&role=admin
```

### 测试 HTTP 方法覆盖

一些 Web 框架提供了一种覆盖请求中实际 HTTP 方法的方法。它们通过模拟缺失的 HTTP 动词并在请求中传递一些自定义头来实现。其主要目的是绕过阻止特定方法的中间件应用程序（如代理或 Web 应用程序防火墙）。以下替代 HTTP 头可能可用于此目的：

- `X-HTTP-Method`
- `X-HTTP-Method-Override`
- `X-Method-Override`

要测试此问题，请考虑受限动词（如 `PUT` 或 `DELETE`）返回 `405 Method not allowed` 的场景。在这种情况下，重放相同的请求，但添加用于 HTTP 方法覆盖的替代头。然后，观察系统的响应。在支持方法覆盖的情况下，应用程序应响应不同的状态码（如 `200 OK`）。

以下示例中的 Web 服务器不允许 `DELETE` 方法并阻止它：

```http
DELETE /resource.html HTTP/1.1
Host: example.org
```

```http
HTTP/1.1 405 Method Not Allowed
[...]
```

添加 `X-HTTP-Method` 头后，服务器响应 200：

```http
GET /resource.html HTTP/1.1
Host: example.org
X-HTTP-Method: DELETE
```

```http
HTTP/1.1 200 OK
[...]
```

## 修复

- 确保只允许需要的方法，并且这些方法被正确配置。
- 确保没有实现绕过用户代理、框架或 Web 服务器实施的安全措施的解决方法。

## 工具

- [Ncat](https://nmap.org/ncat/)
- [cURL](https://curl.haxx.se/)
- [Nmap http-methods NSE 脚本](https://nmap.org/nsedoc/scripts/http-methods.html)

## 参考资料

- [RFC 7231 - 超文本传输协议 (HTTP/1.1)](https://datatracker.ietf.org/doc/html/rfc7231)
- [RFC 5789 - HTTP 的 PATCH 方法](https://datatracker.ietf.org/doc/html/rfc5789)
- [HTACCESS: BILBAO Method Exposed](https://web.archive.org/web/20160616172703/https://www.kernelpanik.org/docs/kernelpanik/bme.eng.pdf)
- [Fortify - 误用 HTTP 方法覆盖](https://vulncat.fortify.com/en/detail?id=desc.dynamic.xtended_preview.often_misused_http_method_override)
- [Mozilla Developer Network - 安全 HTTP 方法](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)
