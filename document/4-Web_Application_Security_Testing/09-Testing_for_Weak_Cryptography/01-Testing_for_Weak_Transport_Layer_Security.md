# 测试弱传输层安全性

|ID          |
|------------|
|WSTG-CRYP-01|

## 概述

当信息在客户端和服务器之间发送时，必须对其进行加密和保护，以防止攻击者能够读取或修改它。这通常使用HTTPS完成，HTTPS使用[传输层安全（TLS）](https://en.wikipedia.org/wiki/Transport_Layer_Security)协议，这是较旧的安全套接层（SSL）协议的替代品。TLS还提供了一种方式，让服务器向客户端证明他们已连接到正确的服务器，方法是提供可信的数字证书。

多年来，在SSL和TLS协议以及它们使用的密码学算法中已经发现了大量密码学弱点。此外，这些协议的实现也存在严重漏洞。因此，测试站点不仅要实施TLS，而且要确保以安全的方式实施，这一点很重要。

## 测试目标

- 验证服务配置。
- 审查数字证书的密码学强度和有效性。
- 确保TLS安全性不可绕过，并且在整个应用程序中正确实施。

## 如何测试

传输层安全性相关问题可大致分为以下几个领域：

### 服务器配置

TLS支持大量协议版本、密码套件和扩展。其中许多被视为遗留版本，具有密码学弱点，如下所列。请注意，随着时间的推移，可能会发现新的弱点，因此此列表可能不完整。

- [SSLv2 (DROWN)](https://drownattack.com/)
- [SSLv3 (POODLE)](https://en.wikipedia.org/wiki/POODLE)
- [TLSv1.0 (BEAST)](https://www.acunetix.com/blog/web-security-zone/what-is-beast-attack/)
- [TLSv1.1 (RFC 8996已弃用)](https://tools.ietf.org/html/rfc8996)
- [EXPORT密码套件 (FREAK)](https://en.wikipedia.org/wiki/FREAK)
- NULL密码套件（[它们仅提供身份验证](https://tools.ietf.org/html/rfc4785)）。
- 匿名密码套件（这些可能在SMTP服务器上受支持，如[RFC 7672](https://tools.ietf.org/html/rfc7672#section-8.2)中所讨论的）
- [RC4密码套件 (NOMORE)](https://www.rc4nomore.com/)
- CBC模式密码套件 (BEAST, [Lucky 13](https://en.wikipedia.org/wiki/Lucky_Thirteen_attack))
- [TLS压缩 (CRIME)](https://en.wikipedia.org/wiki/CRIME)
- [弱DHE密钥 (LOGJAM)](https://weakdh.org/)

[Mozilla服务器端TLS指南](https://wiki.mozilla.org/Security/Server_Side_TLS)详细说明了当前推荐的协议和密码套件。
由于与后量子密码学的兼容性，TLS 1.3比TLS 1.2更应优先使用。

#### 可利用性

应强调的是，虽然其中许多攻击已在实验室环境中得到证明，但它们通常不被认为是可在现实世界中实际利用的，因为它们需要（通常是主动的）中间人（MitM）攻击和大量资源。因此，除了国家级攻击者外，它们不太可能被任何人利用。

### 数字证书

#### 密码学弱点

从密码学角度来看，数字证书需要审查两个主要领域：证书本身的签名和TLS握手期间使用的密钥交换算法。使用经典密码学时，应考虑以下几点：

- 所使用的密码强度应至少为128位安全或更高。
- 对于经典签名，您至少应使用RSA-3072或ECDSA与P-256结合SHA-256或更强的组合。
- 对于经典密钥交换，最少使用P-256或X25519椭圆曲线进行ECDHE。

此外，签名和密钥交换算法都应支持后量子密码学。应考虑以下几点：

- 对于后量子签名，您至少应使用ML-DSA-44（根据[FIPS-204](https://csrc.nist.gov/pubs/fips/204/final)）或更高版本。
- 对于后量子密钥交换，您至少应使用ML-KEM-512（根据[FIPS-203](https://csrc.nist.gov/pubs/fips/203/final)）或更高版本。
- 在过渡到后量子算法期间，您可以使用混合密码套件，如X25519+ML-KEM-768。

#### 有效性

除了密码学安全外，证书还必须被认为是有效（或可信）的。这意味着它必须：

- 在定义的有效期内。
    - 2020年9月1日之后颁发的任何证书的最大寿命不得超过[398天](https://blog.mozilla.org/security/2020/07/09/reducing-tls-certificate-lifespans-to-398-days/)。
    - 证书的有效期将逐渐缩短至[2029年3月的最长47天](https://github.com/cabforum/servercert/blob/main/docs/BR.md#11-overview)。
- 由可信证书颁发机构（CA）签名。
    - 对于外部面向的应用程序，这应该是可信的公共CA；对于内部应用程序，应该是内部CA。
    - 不要仅仅因为*您的*系统不信任CA就将内部应用程序标记为不受信任的证书。
- 具有与系统主机名匹配的主题备用名称（SAN）。
    - 通用名称（CN）字段被现代浏览器忽略，浏览器只查看SAN。
    - 确保使用正确的名称访问系统（例如，如果您通过IP访问主机，则任何证书都将显示为不受信任）。

某些证书可能是为通配符域颁发的（例如`*.example.org`），这意味着它们可以对多个子域有效。虽然方便，但围绕这一点有许多安全问题需要考虑。这些在[OWASP传输层保护速查表](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#carefully-consider-the-use-of-wildcard-certificates)中讨论。

证书还可能在颁发者和SAN字段中泄露有关内部系统或域名信息，这在尝试构建内部网络图片或进行社会工程活动时可能很有用。

### 实现漏洞

多年来，各种TLS实现中都存在漏洞。这里列出的太多了，但一些关键示例包括：

- [Debian OpenSSL可预测随机数生成器](https://www.debian.org/security/2008/dsa-1571) (CVE-2008-0166)
- [OpenSSL不安全重协商](https://www.openssl.org/news/secadv/20091111.txt) (CVE-2009-3555)
- [OpenSSL Heartbleed](https://heartbleed.com) (CVE-2014-0160)
- [F5 TLS POODLE](https://support.f5.com/csp/article/K15882) (CVE-2014-8730)
- [Microsoft Schannel拒绝服务](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2014/ms14-066) (MS14-066 / CVE-2014-6321)

### 应用程序漏洞

除了底层TLS配置需要安全配置外，应用程序还需要以安全方式使用它。本指南的其他地方讨论了其中一些要点：

- [不通过未加密通道发送敏感数据 (WSTG-CRYP-03)](03-Testing_for_Sensitive_Information_Sent_via_Unencrypted_Channels.md)
- [设置HTTP严格传输安全头 (WSTG-CONF-07)](../02-Configuration_and_Deployment_Management_Testing/07-Test_HTTP_Strict_Transport_Security.md)
- [在Cookie上设置安全标志 (WSTG-SESS-02)](../06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

#### 混合活动内容

混合活动内容是指活动资源（如脚本到CSS）通过未加密的HTTP加载并包含到安全（HTTPS）页面中。这是危险的，因为它允许攻击者修改这些文件（因为它们是未加密发送的），这可能允许他们在页面中执行任意代码（JavaScript或CSS）。通过不安全连接加载的被动内容（如图像）也可能泄露信息或允许攻击者篡改页面，尽管导致完全妥协的可能性较小。

> 注意：现代浏览器将阻止从不安全源加载活动内容到安全页面。

#### 从HTTP重定向到HTTPS

许多站点将接受通过未加密HTTP的连接，然后立即将用户重定向到站点的安全（HTTPS）版本，并带有`301永久移动`重定向。然后HTTPS版本的站点设置`Strict-Transport-Security`头，以指示浏览器将来始终使用HTTPS。

但是，如果攻击者能够拦截此初始请求，他们可以将用户重定向到恶意站点，或使用[sslstrip](https://github.com/moxie0/sslstrip)等工具拦截后续请求。

为了防御此类攻击，必须将站点添加到[预加载列表](https://hstspreload.org)。

## 自动测试

有大量扫描工具可用于识别服务的SSL/TLS配置中的弱点，包括专用工具和通用漏洞扫描器。一些更流行的包括：

- [Nmap](https://nmap.org)（各种脚本）
- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslscan](https://github.com/rbsec/sslscan)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [SSL Labs](https://www.ssllabs.com/ssltest/)
- [testssl.sh](https://github.com/drwetter/testssl.sh)

### 手动测试

也可以使用命令行工具手动执行大部分检查，例如`openssl s_client`或`gnutls-cli`来连接特定协议、密码套件或选项。

当像这样测试时，请注意，随大多数现代系统一起提供的OpenSSL或GnuTLS版本可能不支持某些过时和不安全的协议，如SSLv2或EXPORT密码套件。在使用它进行测试之前，请确保您的版本支持过时版本，否则您将获得假阴性。

也可以使用Web浏览器进行有限测试，因为现代浏览器将在其开发者工具中提供正在使用的协议和密码套件的详细信息。它们还提供了一种简单的方法来测试证书是否被认为是可信的，方法是浏览到服务并查看是否收到证书警告。

## 参考资料

- [OWASP传输层保护速查表](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
- [Mozilla服务器端TLS指南](https://wiki.mozilla.org/Security/Server_Side_TLS)
- [CWE-1428：依赖HTTP而非HTTPS](https://cwe.mitre.org/data/definitions/1428.html)
