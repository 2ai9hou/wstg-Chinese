# 测试Cookie属性

|ID          |
|------------|
|WSTG-SESS-02|

## 概述

Web Cookie（以下简称Cookie）通常是恶意用户（通常针对其他用户）的关键攻击向量，应用应始终采取应有的谨慎来保护Cookie。

HTTP是一种无状态协议，意味着它不保存对同一用户发送的请求的任何引用。为了解决这一问题，会话被创建并附加到HTTP请求中。正如在[测试浏览器存储](../11-Client-side_Testing/12-Testing_Browser_Storage.md)中讨论的，浏览器包含多种存储机制。在该部分指南中，对每种机制都进行了详细讨论。

浏览器中最常用的会话存储机制是Cookie存储。Cookie可以由服务器设置，方法是在HTTP响应中包含[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)头，或通过JavaScript设置。Cookie可用于多种原因，例如：

- 会话管理
- 个性化
- 跟踪

为了保护Cookie数据，行业已开发出帮助锁定这些Cookie并限制其攻击面的方法。随着时间的推移，Cookie已成为Web应用的首选存储机制，因为它们在使用和保护方面具有很大的灵活性。

保护Cookie的方法包括：

- [Cookie属性](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Creating_cookies)
- [Cookie前缀](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)

## 测试目标

- 确保Cookie设置了适当的安全配置。

## 如何测试

下面，将讨论每个属性和前缀的描述。测试人员应验证应用是否正确使用它们。Cookie可以通过使用[拦截代理](#intercepting-proxy)或通过查看浏览器的Cookie存储来审查。

### Cookie属性

#### Secure属性

[`Secure`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Secure)属性告诉浏览器仅在通过安全通道（如`HTTPS`）发送请求时才发送Cookie。这有助于保护Cookie不会被传递到未加密的请求中。如果应用可以通过`HTTP`和`HTTPS`访问，攻击者可能能够将用户重定向到在非保护请求中发送其Cookie。

#### HttpOnly属性

[`HttpOnly`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#HttpOnly)属性用于帮助防止会话泄露等攻击，因为它不允许通过客户端脚本（如JavaScript）访问Cookie。

> 这并不会限制XSS攻击的全部攻击面，因为攻击者仍然可以代替用户发送请求，但会极大地限制XSS攻击向量的影响范围。

#### Domain属性

[`Domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies)属性用于将Cookie的域与发送HTTP请求的服务器的域进行比较。如果域匹配或者是子域，则接下来会检查[`path`](#path-attribute)属性。

请注意，只有属于指定域的主机才能为该域设置Cookie。此外，`domain`属性不能是顶级域（如`.gov`或`.com`），以防止服务器为另一个域设置任意Cookie（例如，为`owasp.org`设置Cookie）。如果未设置domain属性，则使用生成Cookie的服务器的主机名作为`domain`的默认值。

例如，如果一个应用在`app.mydomain.com`设置了一个没有domain属性的Cookie，那么该Cookie将被重新提交给`app.mydomain.com`的所有后续请求，但不会提交给其子域（如`hacker.app.mydomain.com`），也不会提交给`otherapp.mydomain.com`。（然而，旧版本的Edge/IE行为不同，_确实_会将这些Cookie发送到子域。）如果开发者想放宽此限制，可以将`domain`属性设置为`mydomain.com`。在这种情况下，Cookie将被发送到`app.mydomain.com`和`mydomain.com`子域的所有请求，如`hacker.app.mydomain.com`，甚至`bank.mydomain.com`。如果某个子域上有一个易受攻击的服务器（例如`otherapp.mydomain.com`），并且`domain`属性设置得太宽松（例如`mydomain.com`），那么该易受攻击的服务器可能用于获取`mydomain.com`整个范围内的Cookie（如会话令牌）。

#### Path属性

[`Path`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Scope_of_cookies)属性在与[`domain`](#domain-attribute)结合设置Cookie范围方面起着重要作用。除了域之外，还可以指定Cookie有效的URL路径。如果域和路径都匹配，则Cookie将在请求中被发送。就像domain属性一样，如果path属性设置得太宽松，可能会使应用容易受到同一服务器上其他应用的攻击。例如，如果path属性设置为Web服务器根目录`/`，则应用Cookie将被发送到同一域内每个应用（如果有多个应用驻留在同一服务器上）。同一服务器下多个应用的一些示例：

- `path=/bank`
- `path=/private`
- `path=/docs`
- `path=/docs/admin`

#### Expires属性

[`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Permanent_cookies)属性用于：

- 设置持久性Cookie
- 如果会话持续时间过长，限制其生命周期
- 通过设置为过去的日期来强制删除Cookie

与[会话Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Session_cookies)不同，持久性Cookie将被浏览器使用，直到Cookie过期。一旦超过过期时间，浏览器将删除Cookie。

#### SameSite属性

[`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies)属性可用于断言Cookie是否应随跨站请求一起发送。此功能允许服务器减轻跨域信息泄露的风险。在某些情况下，它还被用作降低风险（或深度防御）策略，以防止[跨站请求伪造](05-Testing_for_Cross_Site_Request_Forgery.md)攻击。此属性可以配置为三种不同模式：

- `Strict`
- `Lax`
- `None`

##### Strict值

`Strict`值是`SameSite`最严格的用法，允许浏览器仅在第一方上下文中发送Cookie，不包括顶级导航。换句话说，与Cookie关联的数据仅在匹配浏览器URL栏中显示的当前站点的请求上发送。Cookie不会从第三方站点生成的请求中发送。此值特别推荐用于在同一域上执行的操作。但是，它可能会对一些会话管理系统产生负面影响，影响用户导航体验。由于浏览器不会从第三方域或电子邮件生成的任何请求发送Cookie，即促使用户可能已经拥有已认证会话，用户也需要再次登录。

##### Lax值

`Lax`值限制性比`Strict`小。当URL等于Cookie的域（第一方）时，即使链接来自第三方域，Cookie也会被发送。大多数浏览器将此值视为默认行为，因为它比`Strict`值提供了更好的用户体验。它不会为资产（如图像）触发，因为可能不需要Cookie来访问它们。

##### None值

`None`值指定浏览器在所有上下文中发送Cookie，包括跨站请求（实施`SameSite`之前的正常行为）。如果设置了`Samesite=None`，则必须设置Secure属性，否则现代浏览器将忽略SameSite属性，_例如_ `SameSite=None; Secure`。

### Cookie前缀

通过设计，Cookie没有保证其中存储的信息的完整性和机密性的能力。这些限制使得服务器不可能对给定Cookie的属性是如何设置的有信心。为了以向后兼容的方式向服务器提供此类功能，行业引入了[`Cookie名称前缀`](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-prefixes-00)的概念，以促进将此类详细信息作为Cookie名称的一部分嵌入。

#### Host前缀

[`__Host-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)前缀要求Cookie满足以下条件：

  1. Cookie必须设置[`Secure`属性](#secure-attribute)。
  2. Cookie必须从用户代理认为安全的URI设置。
  3. 仅发送到设置Cookie的主机，不得包含任何[`Domain`属性](#domain-attribute)。
  4. Cookie必须设置[`Path`属性](#path-attribute)，值为`/`，以便随主机的每个请求一起发送。

因此，Cookie `Set-Cookie: __Host-SID=12345; Secure; Path=/`将被接受，而以下任何Cookie将始终被拒绝：
`Set-Cookie: __Host-SID=12345`
`Set-Cookie: __Host-SID=12345; Secure`
`Set-Cookie: __Host-SID=12345; Domain=site.example`
`Set-Cookie: __Host-SID=12345; Domain=site.example; Path=/`
`Set-Cookie: __Host-SID=12345; Secure; Domain=site.example; Path=/`

#### Secure前缀

[`__Secure-`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#Cookie_prefixes)前缀限制较少，可以通过在Cookie名称前添加区分大小写的字符串`__Secure-`来引入。任何匹配`__Secure-`前缀的Cookie应满足以下条件：

  1. Cookie必须设置`Secure`属性。
  2. Cookie必须从用户代理认为安全的URI设置。

### 最佳实践

根据应用需求以及Cookie应该如何运作，必须应用属性和前缀。Cookie锁定得越多越好。

综合以上内容，我们可以将最安全的Cookie属性配置定义为：`Set-Cookie: __Host-SID=<session token>; path=/; Secure; HttpOnly; SameSite=Strict`。

## 工具

### 拦截代理

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Web Proxy Burp Suite](https://portswigger.net)

### 浏览器插件

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
- ["FireSheep" for FireFox](https://github.com/codebutler/firesheep)
- ["EditThisCookie" for Chrome](https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=en)
- ["Cookiebro - Cookie Manager" for FireFox](https://addons.mozilla.org/en-US/firefox/addon/cookiebro/)

## 参考资料

- [RFC 2965 - HTTP State Management Mechanism](https://tools.ietf.org/html/rfc2965)
- [RFC 2616 – Hypertext Transfer Protocol – HTTP 1.1](https://tools.ietf.org/html/rfc2616)
- [Same-Site Cookies - draft-ietf-httpbis-cookie-same-site-00](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)
- [The important "expires" attribute of Set-Cookie](https://seckb.yehg.net/2012/02/important-expires-attribute-of-set.html)
- [HttpOnly Session ID in URL and Page Body](https://seckb.yehg.net/2012/06/httponly-session-id-in-url-and-page.html)
- [Internet Explorer Cookie Internals (FAQ)](https://learn.microsoft.com/en-gb/archive/blogs/ieinternals/internet-explorer-cookie-internals-faq)
