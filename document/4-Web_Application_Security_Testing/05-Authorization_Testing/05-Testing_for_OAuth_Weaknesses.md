# 测试 OAuth 弱点

|ID          |
|------------|
|WSTG-ATHZ-05|

## 概述

[OAuth 2.0](https://oauth.net/2/)（以下简称 OAuth）是一种授权框架，允许客户端代表其用户访问资源。

为了实现这一目标，OAuth 大量依赖令牌在不同实体之间进行通信，每个实体具有不同的[角色](https://datatracker.ietf.org/doc/html/rfc6749#section-1.1)：

- **资源所有者：** 授予资源访问权限的实体，通常是用户本人
- **客户端：** 代表资源所有者请求访问资源的应用程序。这些客户端有两种[类型](https://oauth.net/2/client-types/)：
    - **公共客户端：** 无法保护密钥的客户端（如前端应用程序，如 SPA、移动应用程序等）
    - **机密客户端：** 能够通过保护其注册密钥安全地与授权服务器进行身份验证的客户端（如后端服务）
- **授权服务器：** 保存授权信息并授予访问权限的服务器
- **资源服务器：** 客户端访问内容所在的应用程序

由于 OAuth 的职责是将所有者对客户端的访问权限进行委托，这对攻击者来说是一个非常有吸引力的目标，而糟糕的实现会导致对用户资源和信息的未授权访问。

为了向客户端应用程序提供访问权限，OAuth 依赖几种[授权授予类型](https://oauth.net/2/grant-types/)来生成访问令牌：

- [授权码](https://oauth.net/2/grant-types/authorization-code/)：由机密和公共客户端使用，将授权码交换为访问令牌，但仅推荐给机密客户端
- [代码交换密钥证明（PKCE）](https://oauth.net/2/pkce/)：PKCE 在授权码授予之上构建，为公共客户端提供更强的安全性，并提高机密客户端的安全状态
- [客户端凭据](https://oauth.net/2/grant-types/client-credentials/)：用于机器对机器通信，其中"用户"是请求访问资源服务器自有资源的机器
- [设备码](https://oauth.net/2/grant-types/device-code/)：用于输入能力有限的设备。
- [刷新令牌](https://oauth.net/2/grant-types/refresh-token/)：由授权服务器提供，以允许客户端在用户访问令牌变得无效或过期时刷新用户的访问令牌。此授予类型与其他授予类型结合使用。

两种流程将在 [OAuth 2.1](https://oauth.net/2.1/) 发布时被弃用，不推荐使用：

- [隐式流](https://oauth.net/2/grant-types/implicit/)：PKCE 的安全实现使此流程变得过时。在 PKCE 之前，隐式流被用于客户端应用程序，如[单页应用程序](https://en.wikipedia.org/wiki/Single-page_application)，因为 [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) 放宽了站点间通信的[同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。有关为何不推荐隐式授予的更多信息，请查看此[部分](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.1.2)。
- [资源所有者密码凭据](https://oauth.net/2/grant-types/password/)：用于直接将用户凭据交换给客户端，然后客户端将凭据发送给授权服务器以交换访问令牌。有关此流程不推荐的原因的信息，请查看此[部分](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics#section-2.4)。

*：OAuth 中的隐式流仅被弃用，但在 Open ID Connect (OIDC) 中仍然是检索 `id_tokens` 的可行解决方案。请注意理解隐式流是如何被使用的，如果仅使用 `/authorization` 端点来获取访问令牌，而不依赖 `/token` 端点，则可以识别隐式流的使用方式。这方面的示例可以在[这里](https://auth0.com/docs/get-started/authentication-and-authorization-flow/implicit-flow-with-form-post)找到。

*请注意，OAuth 流程是一个复杂的主题，以上仅包括关键领域的摘要。内联引用包含有关特定流程的更多信息。*

## 测试目标

- 确定 OAuth 2.0 实现是否存在漏洞或使用已弃用或自定义实现。

## 如何测试

### 测试已弃用的授予类型

已弃用的授予类型因安全和功能原因而被淘汰。识别它们是否正在使用，使我们能够快速审查它们是否容易受到与其使用相关的任何威胁的威胁。某些内容可能超出攻击者的范围，例如客户端可能使用用户凭据的方式。这应被记录并提交给内部工程团队。

对于公共客户端，通常可以在对 `/token` 端点的请求中识别授予类型。它在令牌交换中由 `grant_type` 参数指示。

以下示例显示带 PKCE 的授权码授予。

```http
POST /oauth/token HTTP/1.1
Host: as.example.com
[...]

{
  "client_id":"example-client",
  "code_verifier":"example",
  "grant_type":"authorization_code",
  "code":"example",
  "redirect_uri":"https://client.example.com"
}
```

`grant_type` 参数的值及其指示的授予类型：

- `password`：表示 ROPC 授予。
- `client_credentials`：表示客户端凭据授予。
- `authorization_code`：表示授权码授予。

隐式流类型不由 `grant_type` 参数指示，因为令牌在 `/authorization` 端点请求的响应中呈现，而是可以通过 `response_type` 识别。以下是一个示例。

```http
GET /authorize
  ?client_id=<some_client_id>
  &response_type=token 
  &redirect_uri=https%3A%2F%2Fclient.example.com%2F
  &scope=openid%20profile%20email
  &state=<random_state>
```

以下 URL 参数指示正在使用的 OAuth 流程：

- `response_type=token`：表示隐式流，因为客户端直接请求授权服务器返回令牌。
- `response_type=code`：表示授权码流程，因为客户端请求授权服务器返回一个代码，之后将与令牌交换。
- `code_challenge=sha256(xyz)`：表示 PKCE 扩展，因为没有其他流程使用此参数。

以下是带 PKCE 的授权码流程的授权请求示例：

```http
GET /authorize
    ?redirect_uri=https%3A%2F%2Fclient.example.com%2F
    &client_id=<some_client_id>
    &scope=openid%20profile%20email
    &response_type=code
    &response_mode=query
    &state=<random_state>
    &nonce=<random_nonce>
    &code_challenge=<random_code_challenge>
    &code_challenge_method=S256 HTTP/1.1
Host: as.example.com
[...]
```

#### 公共客户端

带 PKCE 扩展的授权码授予被推荐用于公共客户端。带 PKCE 的授权码流程的授权请求应包含 `response_type=code` 和 `code_challenge=sha256(xyz)`。

令牌交换应包含授予类型 `authorization_code` 和 `code_verifier`。

公共客户端的不当授予类型：

- 不带 PKCE 扩展的授权码授予
- 客户端凭据
- 隐式流
- ROPC

#### 机密客户端

带 PKCE 扩展的授权码授予被推荐用于机密客户端。PKCE 扩展也可以使用。

机密客户端的不当授予类型：

- 客户端凭据（机器对机器除外 - 见下文）
- 隐式流
- ROPC

##### 机器对机器

在没有用户交互且客户端仅为机密客户端的情况下，可以使用客户端凭据授予。

如果您知道 `client_id` 和 `client_secret`，可以通过传递 `client_credentials` 授予类型来获取令牌。

```bash
$ curl --request POST \
  --url https://as.example.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<some_client_id>","client_secret":"<some_client_secret>","grant_type":"client_credentials"}' --proxy https://localhost:8080/ -k
```

### 凭据泄露

根据流程，OAuth 通过 URL 参数传输多种类型的凭据。

以下令牌可被视为泄露的凭据：

- 访问令牌
- 刷新令牌
- 授权码
- PKCE 代码挑战/代码验证码

由于 OAuth 的工作方式，授权 `code` 以及 `code_challenge` 和 `code_verifier` 可能成为 URL 的一部分。如果 `response_mode` 未设置为 [`form_post`](https://openid.net/specs/oauth-v2-form-post-response-mode-1_0.html)，隐式流会通过 URL 的一部分传输授权令牌。这可能导致在 Referrer 头、日志文件和代理中泄露请求的令牌或代码，因为这些参数是通过查询或片段传递的。

隐式流泄露令牌的风险远高于泄露 `code` 或任何其他 `code_*` 参数的风险，因为它们绑定到特定客户端，泄露后更难被滥用。

为了测试此场景，请使用 ZAP 等 HTTP 拦截代理并拦截 OAuth 流量。

- 逐步完成授权过程，识别 URL 中存在的任何凭据。
- 如果 OAuth 流程中涉及的页面包含外部资源，请分析向它们发出的请求。凭据可能通过 Referrer 头泄露。

在逐步完成 OAuth 流程并使用应用程序后，HTTP 拦截代理的请求历史中会捕获一些请求。在请求历史中搜索包含授权服务器和客户端 URL 的 HTTP Referrer 头（例如 `Referer: https://idp.example.com/`）。

检查 HTML meta 标签（尽管此标签并非[在所有浏览器中都受支持](https://caniuse.com/mdn-html_elements_meta_name_referrer)），或 [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) 可以帮助评估是否正在通过 Referrer 头发生任何凭据泄露。

## 相关测试用例

- [测试 JSON Web Token](../06-Session_Management_Testing/10-Testing_JSON_Web_Tokens.md)

## 修复方案

- 在实施 OAuth 时，始终考虑所使用的技术以及应用程序是能够避免泄露密钥的服务器端应用程序，还是无法做到这一点的客户端应用程序。
- 在几乎所有情况下，请使用带 PKCE 的授权码流程。一个例外可能是机器对机器流程。
- 使用 POST 参数或头值来传输密钥。
- 当没有其他可能性时（例如，无法迁移的遗留应用程序），实施额外的安全头，如 `Referrer-Policy`。

## 工具

- [BurpSuite](https://portswigger.net/burp/releases)
- [EsPReSSO](https://github.com/portswigger/espresso)
- [ZAP](https://www.zaproxy.org/)

## 参考资料

- [使用 OAuth 2.0 进行用户身份验证](https://oauth.net/articles/authentication/)
- [OAuth 2.0 授权框架](https://datatracker.ietf.org/doc/html/rfc6749)
- [OAuth 2.0 授权框架：Bearer 令牌使用](https://datatracker.ietf.org/doc/html/rfc6750)
- [OAuth 2.0 威胁模型和安全注意事项](https://datatracker.ietf.org/doc/html/rfc6819)
- [OAuth 2.0 安全最佳当前实践](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics-16)
- [带 PKCE 的授权码流程](https://auth0.com/docs/authorization/flows/authorization-code-flow-with-proof-key-for-code-exchange-pkce)
