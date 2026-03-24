# 审查 Web 服务器元文件以发现信息泄露

|ID          |
|------------|
|WSTG-INFO-03|

## 摘要

本节描述如何测试各种元数据文件以发现 Web 应用路径或功能的信息泄露。此外，要被 Spiders、Robots 或 Crawlers 避免的目录列表也可以创建为[通过应用映射执行路径](07-Map_Execution_Paths_Through_Application.md)的依赖项。其他信息也可能被收集以识别攻击面、技术细节或用于社交工程参与。

## 测试目标

- 通过分析元数据文件识别隐藏或混淆的路径和功能。
- 提取和映射其他可能导致对手更好地理解手头系统的信息。

## 如何测试

> 下面使用 `wget` 执行的任何操作也可以使用 `curl` 完成。许多动态应用安全测试（DAST）工具，如 ZAP 和 Burp Suite，包括检查或解析这些资源作为其 spider/crawler 功能的一部分。它们也可以使用各种 [Google Dorks](https://en.wikipedia.org/wiki/Google_hacking) 或利用 `inurl:` 等高级搜索功能来识别。

### Robots

Web Spiders、Robots 或 Crawlers 检索网页，然后递归遍历超链接以检索进一步的 Web 内容。它们的可接受行为由 Web 根目录中 [robots.txt](https://www.robotstxt.org/) 文件的 [Robots Exclusion Protocol](https://www.robotstxt.org) 指定。

例如，下面引用了 2020 年 5 月 5 日从 [Google](https://www.google.com/robots.txt) 采样的 `robots.txt` 文件的开头：

```text
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
Disallow: /sdch
...
```

[User-Agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) 指令指特定的 Web spider/robot/crawler。例如，`User-Agent: Googlebot` 指来自 Google 的 spider，而 `User-Agent: bingbot` 指来自 Microsoft 的 crawler。上面示例中的 `User-Agent: *` 适用于所有 [Web spiders/robots/crawlers](https://support.google.com/webmasters/answer/6062608?visit_id=637173940975499736-3548411022&rd=1)。

`Disallow` 指令指定哪些资源被 spiders/robots/crawlers 禁止。在上面的示例中，以下是被禁止的：

```text
...
Disallow: /search
...
Disallow: /sdch
...
```

Web spiders/robots/crawlers 可以[故意忽略](https://blog.isc2.org/isc2_blog/2008/07/the-attack-of-t.html) `robots.txt` 文件中指定的 `Disallow` 指令。因此，`robots.txt` 不应被视为强制执行第三方如何访问、存储或重新发布 Web 内容限制的机制。

`robots.txt` 文件从 Web 服务器的根目录检索。例如，使用 `wget` 或 `curl` 从 `www.google.com` 检索 `robots.txt`：

```bash
$ curl -O -Ss https://www.google.com/robots.txt && head -n5 robots.txt
User-agent: *
Disallow: /search
Allow: /search/about
Allow: /search/static
Allow: /search/howsearchworks
...
```

#### 使用 Google Webmaster Tools 分析 robots.txt

站点所有者可以使用 Google "Analyze robots.txt" 功能作为其 [Google Webmaster Tools](https://www.google.com/webmasters/tools) 的一部分来分析站点。此工具可协助测试，程序如下：

1. 使用 Google 账户登录 Google Webmaster Tools。
2. 在仪表板上，输入要分析的站点的 URL。
3. 在可用方法之间进行选择，然后按照屏幕上的指示进行操作。

### META 标签

`<META>` 标签位于每个 HTML 文档的 `<HEAD>` 部分中，并且应在整个站点中保持一致，以防 robot/spider/crawler 起始点不是从 webroot 即 [deep link](https://en.wikipedia.org/wiki/Deep_linking) 开始的文档链接。Robots 指令也可以使用特定的 [META 标签](https://www.robotstxt.org/meta.html) 指定。

#### Robots META 标签

如果没有 `<META NAME="ROBOTS" ... >` 条目，则"Robots Exclusion Protocol"默认值分别为 `INDEX,FOLLOW`。因此，"Robots Exclusion Protocol"定义的另外两个有效条目以 `NO...` 为前缀，即 `NOINDEX` 和 `NOFOLLOW`。

基于 webroot 中 `robots.txt` 文件中列出的 Disallow 指令，在每个网页中执行 `<META NAME="ROBOTS"` 的正则表达式搜索。然后将结果与 webroot 中的 robots.txt 文件进行比较。

#### 其他 META 信息标签

组织经常在 Web 内容中嵌入信息性 META 标签以支持各种技术，如屏幕阅读器、社交网络预览、搜索引擎索引等。此类 meta 信息对于测试人员识别使用的技术以及要探索和测试的其他路径/功能可能很有价值。以下 meta 信息是通过 2020 年 5 月 05 日的查看页面源代码从 `www.whitehouse.gov` 检索的：

```html
...
<meta property="og:locale" content="en_US" />
<meta property="og:type" content="website" />
<meta property="og:title" content="The White House" />
<meta property="og:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta property="og:url" content="https://www.whitehouse.gov/" />
<meta property="og:site_name" content="The White House" />
<meta property="fb:app_id" content="1790466490985150" />
<meta property="og:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta property="og:image:secure_url" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:description" content="We, the citizens of America, are now joined in a great national effort to rebuild our country and to restore its promise for all. – President Donald Trump." />
<meta name="twitter:title" content="The White House" />
<meta name="twitter:site" content="@whitehouse" />
<meta name="twitter:image" content="https://www.whitehouse.gov/wp-content/uploads/2017/12/wh.gov-share-img_03-1024x538.png" />
<meta name="twitter:creator" content="@whitehouse" />
...
<meta name="apple-mobile-web-app-title" content="The White House">
<meta name="application-name" content="The White House">
<meta name="msapplication-TileColor" content="#0c2644">
<meta name="theme-color" content="#f5f5f5">
...
```

### 站点地图

站点地图是开发者或组织提供关于站点或应用提供的页面、视频和其他文件的信息以及它们之间关系的文件。搜索引擎可以使用此文件更高效地导航站点。同样，测试人员可以利用 'sitemap.xml' 文件来更深入地了解正在调查的站点或应用。

以下是 2020 年 5 月 05 日检索的 Google 主站点地图的摘要。

```bash
$ wget --no-verbose https://www.google.com/sitemap.xml && head -n8 sitemap.xml
2020-05-05 12:23:30 URL:https://www.google.com/sitemap.xml [2049] -> "sitemap.xml" [1]

<?xml version="1.0" encoding="UTF-8"?>
<sitemapindex xmlns="https://www.google.com/schemas/sitemap/0.84">
  <sitemap>
    <loc>https://www.google.com/gmail/sitemap.xml</loc>
  </sitemap>
  <sitemap>
    <loc>https://www.google.com/forms/sitemaps.xml</loc>
  </sitemap>
...
```

从那里探索，测试人员可能希望检索 gmail 站点地图 `https://www.google.com/gmail/sitemap.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="https://www.sitemaps.org/schemas/sitemap/0.9" xmlns:xhtml="https://www.w3.org/1999/xhtml">
  <url>
    <loc>https://www.google.com/intl/am/gmail/about/</loc>
    <xhtml:link href="https://www.google.com/gmail/about/" hreflang="x-default" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/el/gmail/about/" hreflang="el" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/it/gmail/about/" hreflang="it" rel="alternate"/>
    <xhtml:link href="https://www.google.com/intl/ar/gmail/about/" hreflang="ar" rel="alternate"/>
...
```

### 安全 TXT

[security.txt](https://securitytxt.org) 由 IETF 批准为 [RFC 9116 - 用于安全漏洞披露的文件格式](https://www.rfc-editor.org/rfc/rfc9116.html)，允许站点定义安全策略和联系详情。在测试场景中，这可能引起兴趣的原因有多种，包括但不限于：

- 识别进一步包含在发现/分析中的路径或资源。
- 开源情报收集。
- 查找 Bug Bounties 等信息。
- 社交工程。

该文件可能存在于 Web 服务器的根目录或 `.well-known/` 目录中，例如：

- `https://example.com/security.txt`
- `https://example.com/.well-known/security.txt`

以下是 2020 年 5 月 05 日从 LinkedIn 检索的真实世界示例：

```bash
$ wget --no-verbose https://www.linkedin.com/.well-known/security.txt && cat security.txt
2020-05-07 12:56:51 URL:https://www.linkedin.com/.well-known/security.txt [333/333] -> "security.txt" [1]
# Conforms to IETF `draft-foudil-securitytxt-07`
Contact: mailto:security@linkedin.com
Contact: https://www.linkedin.com/help/linkedin/answer/62924
Encryption: https://www.linkedin.com/help/linkedin/answer/79676
Canonical: https://www.linkedin.com/.well-known/security.txt
Policy: https://www.linkedin.com/help/linkedin/answer/62924
```

OpenPGP 公钥包含一些可提供关于密钥本身信息的元数据。以下是可以从 OpenPGP 公钥提取的一些常见元数据元素：

- **密钥 ID**：密钥 ID 是从公钥材料派生的短标识符。它有助于识别密钥，通常显示为八个字符的十六进制值。
- **密钥指纹**：密钥指纹是从密钥材料派生的更长且更唯一的标识符。它通常显示为 40 个字符的十六进制值。密钥指纹通常用于验证公钥的完整性和真实性。
- **密钥算法**：密钥算法表示公钥使用的加密算法。OpenPGP 支持各种算法，如 RSA、DSA 和 ECC（椭圆曲线加密）。
- **密钥大小**：密钥大小是指加密密钥的位数长度。它表示密钥的强度并决定密钥提供的安全级别。
- **密钥创建日期**：密钥创建日期表示密钥生成或创建的时间。
- **密钥过期日期**：OpenPGP 公钥可以设置过期日期，之后它们被视为无效。密钥过期日期指定密钥不再有效的时间。
- **用户 ID**：公钥可以有一个或多个关联的用户 ID，用于识别与密钥关联的所有者或实体。用户 ID 通常包括密钥所有者的姓名、电子邮件地址和可选注释等信息。

### Humans TXT

`humans.txt` 是了解站点背后人们的倡议。它采用文本文件的形式，包含关于参与构建站点的不同人员的信息。此文件通常（但不总是）包含与职业或工作站点/路径相关的信息。

以下示例于 2020 年 5 月 05 日从 Google 检索：

```bash
$ wget --no-verbose  https://www.google.com/humans.txt && cat humans.txt
2020-05-07 12:57:52 URL:https://www.google.com/humans.txt [286/286] -> "humans.txt" [1]
Google is built by a large team of engineers, designers, researchers, robots, and others in many different sites across the globe. It is updated continuously, and built with more tools and technologies than we can shake a stick at. If you'd like to help us out, see careers.google.com.
```

### 其他 .well-known 信息来源

还有其他 RFC 和互联网草案建议 `.well-known/` 目录中文件的标准化用法。这些列表可以在 [WikiPedia 上这里](https://en.wikipedia.org/wiki/List_of_/.well-known/_services_offered_by_webservers) 或 [通过 IANA 这里](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml) 找到。

对于测试人员来说，审查 RFC/草案并创建列表提供给 crawler 或 fuzzer 以验证此类文件的存在或内容是相当简单的。

## 工具

- 浏览器（查看源代码或 Dev Tools 功能）
- cURL
- wget
- Burp Suite
- ZAP
