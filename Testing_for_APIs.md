# API 测试

Web API 已被广泛采用，因其允许第三方程序以更高效、更便捷的方式与站点交互。在本指南中，我们将讨论 API 的一些基本概念以及 API 安全测试的方法。

## 背景概念

REST（表述性状态转移）是一种在开发者设计 API 时实现的架构。
遵循 REST 风格的 Web 应用 API 被称为 REST API。
REST API 使用 URI（统一资源标识符）来访问资源。[RFC3986](https://tools.ietf.org/html/rfc3986) 中定义的通用 URI 语法如下：

> URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]

我们对 URI 的路径感兴趣，因为它是用户与资源之间的关系。
例如，`https://api.test.xyz/admin/testing/report`，这显示了测试报告，其中存在用户 admin 与其报告之间的关系。

任何 URI 的路径都将定义 REST API 资源模型，资源通过正斜杠分隔，基于自上而下设计。
例如：

- `https://api.test.xyz/admin/testing/report`
- `https://api.test.xyz/admin/testing/`
- `https://api.test.xyz/admin/`

REST API 请求遵循 [RFC7231](https://tools.ietf.org/html/rfc7231) 中定义的 [HTTP 请求方法](https://tools.ietf.org/html/rfc7231#section-4)

| 方法   | 描述                     |
|--------|------------------------|
| GET    | 获取资源状态的表述       |
| POST   | 创建新资源              |
| PUT    | 更新资源                |
| DELETE | 删除资源                |
| HEAD   | 获取与资源状态关联的元数据 |
| OPTIONS| 列出可用方法            |

REST API 使用 HTTP 响应消息的状态码来通知客户端其请求的结果。

| 响应码 | 响应消息              | 描述                                                           |
|--------|----------------------|---------------------------------------------------------------|
| 200   | OK                   | 处理客户端请求成功                                              |
| 201   | Created              | 新资源已创建                                                   |
| 301   | Moved Permanently    | 永久重定向                                                     |
| 304   | Not Modified         | 缓存相关响应，当客户端拥有的资源副本与服务器相同时返回                 |
| 307   | Temporary Redirect   | 临时重定向                                                     |
| 400   | Bad Request          | 客户端发送的格式错误的请求                                       |
| 401   | Unauthorized         | 客户端无权发出请求或访问特定资源                                   |
| 402   | Forbidden            | 客户端被禁止访问资源                                           |
| 404   | Not Found            | 资源不存在或根据请求不正确                                       |
| 405   | Method Not Allowed   | 使用了无效或未知方法                                            |
| 500   | Internal Server Error| 服务器因内部错误无法处理请求                                       |

HTTP 头用于请求和响应中。
在发起 API 请求时，使用 Content-Type 头并设置为 `application/json`，因为消息体包含 JSON 数据格式。

Web 认证类型基于：

- Bearer 令牌：由 `Authorization: Bearer <token>` 头标识。用户登录后，将获得一个 bearer 令牌，在每个请求中发送该令牌以认证用户并授权其访问 OAuth 2.0 保护的资源。
- HTTP Cookie：由 `Cookie: <name>=<unique value>` 头标识。用户登录成功时，服务器使用 `Set-Cookie` 头回复，指定其名称和唯一值。在每个请求中，浏览器会自动将其附加到发往该服务器的请求，遵循 [SOP](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。
- 基本 HTTP 认证：由 `Authorization: Basic <base64 value>` 头标识。用户尝试登录时，请求随附上述头发送，其中包含 base64 值，其内容为 `username:password`。这是最弱的认证形式之一，因为它在每个请求中以编码方式传输用户名和密码，容易被获取。

## 如何测试

### 通用测试方法

步骤 1：列出端点并使用不同的请求方法：使用用户配置文件登录，并使用爬虫工具列出此角色的端点。
要检查端点，您需要使用不同的请求方法并观察 API 的行为。

步骤 2：利用漏洞——在步骤 1 中了解了如何列出端点和使用 HTTP 方法检查端点后，我们将找到一些利用漏洞的方法。以下是一些测试策略：

- IDOR 测试
- 权限提升

### 特定测试——（基于令牌的）认证

基于令牌的身份验证通过在每个 HTTP 请求中发送签名令牌（由服务器验证）来实现。

最常用的令牌格式是 JSON Web Token（JWT），定义于 [RFC7519](https://tools.ietf.org/html/rfc7519)。[测试 JSON Web 令牌](/document/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md) 指南包含如何测试 JWT 的更多详细信息。

## 相关测试用例

- [IDOR](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)
- [权限提升](https://github.com/OWASP/wstg/blob/master/document/4-Web_Application_Security_Testing/05-Authorization_Testing/03-Testing_for_Privilege_Escalation.md)
- 所有[会话管理](https://github.com/OWASP/wstg/tree/master/document/4-Web_Application_Security_Testing/06-Session_Management_Testing)测试用例
- [测试 JSON Web 令牌](/document/4-Web_Application_Security_Testing/06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## 工具

- ZAP
- Burp suite

## 参考资料

- [REST HTTP 方法](https://restfulapi.net/http-methods/)
- [RFC3986 URI](https://tools.ietf.org/html/rfc3986)
- [JWT](https://jwt.io/)
- [破解 JWT](https://www.sjoerdlangkemper.nl/2016/09/28/attacking-jwt-authentication/)
