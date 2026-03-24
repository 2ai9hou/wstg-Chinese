# 映射应用架构

|ID          |
|------------|
|WSTG-INFO-10|

## 摘要

为了有效地测试应用，并能够提供关于如何处理已发现问题的有意义的建议，了解实际测试的内容非常重要。此外，确定特定组件是否应被视为测试范围之外可能是有帮助的。

现代 Web 应用的复杂性可能差异很大，从运行在单个服务器上的简单脚本到分布在数十个不同系统、语言和组件上的高度复杂的应用。还可能有额外的网络级组件，如防火墙或入侵防御系统，可能对测试产生重大影响。

## 测试目标

- 理解应用的架构以及使用的技术。

## 如何测试

当从黑盒角度进行测试时，尝试构建应用如何工作的清晰图片以及哪些技术和组件到位是很重要的。在某些情况下，可以测试特定组件，如 Web 应用防火墙，而其他组件可以通过检查应用的行为来识别。

以下部分提供了常见架构组件的高级概述以及如何识别它们的详细信息。

### 应用组件

#### Web 服务器

简单应用可能在单个服务器上运行，可以使用指南的[识别 Web 服务器指纹](02-Fingerprint_Web_Server.md)部分中讨论的步骤来识别。

#### 平台即服务（PaaS）

在平台即服务（PaaS）模型中，Web 服务器和底层基础设施由服务提供商管理，客户仅负责部署在其上的应用。从测试的角度来看，有两个主要区别：

- 应用所有者无法访问底层基础设施，这意味着他们将无法直接修复任何问题
- 基础设施测试可能超出任何参与的范围

在某些情况下，可以识别 PaaS 的使用，因为应用可能使用特定域名（例如，部署在 Azure App Services 上的应用将具有 `*.azurewebsites.net` 域名——尽管它们也可以使用自定义域名）。在其他情况下，很难确定是否使用了 PaaS。

#### 无服务器

在无服务器模型中，开发人员提供直接在各平台上作为单个函数运行的代码，而不是运行部署在 webroot 中的传统较大 Web 应用。这使其非常适合基于微服务的架构。与 PaaS 环境一样，基础设施测试可能超出范围。

在某些情况下，可以通过特定 HTTP 头的存在来指示无服务器代码的使用。例如，AWS Lambda 函数通常返回以下头：

```http
X-Amz-Invocation-Type
X-Amz-Log-Type
X-Amz-Client-Context
```

Azure Functions 不太明显。它们通常返回 `Server: Kestrel` 头——但仅凭它本身不足以确定它是 Azure App 函数，因为它可能是 Kestrel 上运行的其他代码。

#### 微服务

在基于微服务的架构中，应用 API 由多个独立服务组成，而不是作为单体应用运行。这些服务本身通常在容器内运行（通常使用 Kubernetes），可以使用各种不同的操作系统和语言。虽然它们通常位于单个 API 网关和域之后，但多种语言的使用（通常在详细错误消息中指示）可能表明正在使用微服务。

#### 静态存储

许多应用将静态内容存储在专用存储平台上，而不是直接托管在主 Web 服务器上。两个最常见的平台是 Amazon 的 S3 Buckets 和 Azure 的 Storage Accounts，可以通过域名轻松识别：

- `BUCKET.s3.amazonaws.com` 或 `s3.REGION.amazonaws.com/BUCKET` 用于 Amazon S3 Buckets
- `ACCOUNT.blob.core.windows.net` 用于 Azure Storage Accounts

如[测试云存储指南](../02-Configuration_and_Deployment_Management_Testing/11-Test_Cloud_Storage.md)部分所述，这些存储帐户通常可以暴露敏感文件。

#### 数据库

大多数非平凡的 Web 应用使用某种数据库来存储动态内容。在某些情况下，可以确定数据库。这通常可以通过以下方式完成：

- 对服务器进行端口扫描并查找与特定数据库关联的任何开放端口
- 触发 SQL（或 NoSQL）相关错误消息（或从[搜索引擎](../01-Information_Gathering/01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md)查找现有错误）

当无法确定性地确定数据库时，测试人员通常可以基于应用的其他方面进行有根据的猜测：

- Windows、IIS 和 ASP.NET 通常使用 Microsoft SQL server
- 嵌入式系统通常使用 SQLite
- PHP 通常使用 MySQL 或 PostgreSQL
- APEX 通常使用 Oracle

这些不是硬性规则，但如果无法获得更好的信息，它们当然可以提供合理的起点。

#### 认证

大多数应用都有用户认证。有多种认证后端可以使用：

- Web 服务器配置（包括 `.htaccess` 文件）或在脚本中硬编码密码
    - 通常显示为 HTTP Basic 认证，由浏览器中的弹出窗口和 `WWW-Authenticate: Basic` HTTP 头指示
- 数据库中的本地用户帐户
    - 通常集成在应用中的表单或 API 端点中
- 现有中央认证源，如 Active Directory 或 LDAP 服务器
    - 可能使用 NTLM 认证，由 `WWW-Authenticate: NTLM` HTTP 头指示
    - 可以表单形式集成到 Web 应用中
    - 可能要求以"DOMAIN\username"格式输入用户名，或者可能提供可用域的下拉菜单
- 使用内部或外部提供商的单点登录（SSO）
    - 通常使用 OAuth、OpenID Connect 或 SAML

应用可能为用户提供多种认证选项（如注册本地帐户或使用他们现有的 Facebook 帐户），并可能对普通用户和管理员使用不同机制。

#### 第三方服务和 API

几乎所有 Web 应用都包含加载或客户端与之交互的第三方资源。这些可以包括：

- [主动内容](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_active_content)（如脚本、样式表、字体和 iframe）
- [被动内容](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content#mixed_passivedisplay_content)（如图片和视频）
- 外部 API
- 社交媒体按钮
- 广告网络
- 支付网关

这些资源由用户的浏览器直接请求，使其更容易使用开发者工具或拦截代理来识别。虽然识别它们很重要（因为它们可能影响应用的安全性），但请记住，*它们通常不在测试范围内*，因为它们属于第三方。

### 网络组件

#### 反向代理

反向代理位于一个或多个后端服务器前面，并将请求重定向到适当的目的地。它们可用于实现各种功能，例如：

- 作为[负载平衡器](#load-balancer)或 [Web 应用防火墙](#web-application-firewall-waf)
- 允许多个应用托管在单个 IP 地址或域上（子文件夹中）
- 实施 IP 过滤或其他限制
- 缓存来自后端的内容以提高性能

并不总是能够检测到反向代理（特别是如果后面只有一个应用），但您有时可以通过以下方式识别它：

- 前端服务器和后端应用之间的不匹配（例如，`Server: nginx` 头与 ASP.NET 应用）
    - 这有时可能导致[请求走私漏洞](https://portswigger.net/web-security/request-smuggling)
- 重复头（特别是 `Server` 头）
- 托管在同一 IP 地址或域上的多个应用（特别是如果它们使用不同语言）

#### 负载平衡器

负载平衡器位于多个后端服务器前面，并在它们之间分配请求，以提供更大的冗余和处理能力。

负载平衡器可能难以检测，但有时可以通过发出多个请求并检查响应的差异来识别，例如：

- 不一致的系统时间
- 详细错误消息中不同的内部 IP 地址或主机名
- 从[服务端请求伪造（SSRF）](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md)返回的不同地址

它们也可以通过特定 Cookie 的存在来指示（例如，F5 BIG-IP 负载平衡器将创建一个名为 `BIGipServer` 的 Cookie）。

#### 内容分发网络（CDN）

内容分发网络（CDN）是一组地理分布的缓存代理服务器，旨在提高站点性能。

它通常通过将面向公众的域指向 CDN 的服务器来配置，然后配置 CDN 连接到正确的后端服务器（有时称为"源"）。

检测 CDN 的最简单方法是对域解析到的 IP 地址执行 WHOIS 查找。如果它们属于 CDN 公司（如 Akamai、Cloudflare 或 Fastly——请参阅[维基百科](https://en.wikipedia.org/wiki/Content_delivery_network#Notable_content_delivery_service_providers)获取更完整的列表），则可能在使用 CDN。

当在 CDN 后面测试站点时，应牢记以下要点：

- IP 地址和服务器属于 CDN 提供商，可能超出基础设施测试的范围
- 许多 CDN 还包括诸如机器人检测、速率限制和 Web 应用防火墙等功能
- CDN 通常缓存内容。因此，后端的更改可能不会立即显示在站点上。

如果站点在 CDN 后面，识别后端服务器可能很有用。如果没有正确实施访问控制，测试人员可能能够通过直接访问后端服务器来绕过 CDN（及其提供的任何保护）。有多种不同方法可以识别后端系统：

- 应用发送的电子邮件可能直接来自后端服务器，这可能泄露其 IP 地址
- 域的 DNS grinding、区域传输或证书透明度列表可能在子域上显示它
- 扫描公司已知的 IP 范围可能有助于识别后端服务器
- 利用[服务端请求伪造（SSRF）](../07-Input_Validation_Testing/19-Testing_for_Server-Side_Request_Forgery.md)可能泄露 IP 地址
- 应用的详细错误消息可能暴露 IP 地址或主机名

### 安全组件

#### 网络防火墙

大多数 Web 服务器将受到数据包过滤或状态检测防火墙的保护，阻止任何不需要的网络流量。要检测此问题，请对服务器执行端口扫描并检查结果。

如果大多数端口显示为"关闭"（即，它们对初始 `SYN` 数据包返回 `RST` 数据包），则表明服务器可能未受到防火墙保护。如果端口显示为"过滤"（即，发送到未使用端口的 `SYN` 数据包时没有收到响应），则很可能已设置防火墙。

此外，如果不当的服务暴露给外部（如 SMTP、IMAP、MySQL 等），则表明要么没有防火墙，要么防火墙配置错误。

#### 网络入侵检测和防御系统

网络入侵检测系统（IDS）旨在检测可疑或恶意网络级活动（如端口或漏洞扫描）并发出警报。入侵防御系统（IPS）类似，但也会采取行动防止活动，通常通过阻止源 IP 地址。

IPS 通常可以通过对目标运行自动化扫描工具（如端口扫描器）并查看源 IP 是否被阻止来检测。但是，许多应用级工具可能不会被 IPS 检测到（特别是如果它不解密 TLS）。

#### Web 应用防火墙（WAF）

Web 应用防火墙（WAF）检查 HTTP 请求的内容并阻止那些看起来可疑或恶意的请求。它们也可以用于动态应用其他控制（如 CAPTCHA 或速率限制）。它们通常利用一组已知恶意签名和正则表达式，如 [OWASP 核心规则集](https://owasp.org/www-project-modsecurity-core-rule-set/)，以识别恶意流量。WAF 可以有效保护免受某些类型的攻击（如 SQL 注入或跨站脚本），但对其他类型（如访问控制或业务逻辑相关问题）效果较差。

WAF 可以部署在多个位置，包括：

- 在 Web 服务器本身上
- 在单独的虚拟机或硬件设备上
- 在云中，位于后端服务器前面

因为 WAF 阻止恶意请求，所以可以通过添加常见攻击字符串到参数并观察它们是否被阻止来检测。例如，尝试添加名为 `foo` 的参数，值为 `' UNION SELECT 1` 或 `><script>alert(1)</script>`。如果这些请求被阻止，则可能存在 WAF。此外，阻止页面的内容可能提供关于正在使用的特定技术的信息。最后，一些 WAF 可能在响应中添加 Cookie 或 HTTP 头以揭示它们的存在。

如果使用基于云的 WAF，则可能通过使用与[内容分发网络](#content-delivery-network-cdn)部分讨论的相同方法直接访问后端服务器来绕过它。
