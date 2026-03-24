# 审查网页内容以发现信息泄露

|ID          |
|------------|
|WSTG-INFO-05|

## 摘要

对于程序员来说，在源代码中包含详细注释和元数据是非常常见的，甚至可以说是推荐的。然而，HTML 代码中包含的注释和元数据可能泄露不应被潜在攻击者获取的内部信息。应进行注释和元数据审查以确定是否有信息被泄露。此外，一些应用可能在重定向响应的正文中泄露信息。

对于现代 Web 应用，使用客户端 JavaScript 构建前端越来越流行。流行的前端构建技术使用客户端 JavaScript，如 React、Angular 或 Vue。与 HTML 代码中的注释和元数据类似，许多程序员也在前端将敏感信息硬编码到 JavaScript 变量中。敏感信息可能包括（但不限于）：私有 API 密钥（如不受限制的 Google Map API 密钥）、内部 IP 地址、敏感路由（如通往隐藏管理页面或功能的路由），甚至是凭据。这些敏感信息可能从前端 JavaScript 代码中泄露。应进行审查以确定是否有可被攻击者滥用的敏感信息泄露。

对于大型 Web 应用，性能问题是程序员关注的重点。程序员使用不同方法来优化前端性能，包括语法增强样式表（Sass）、Sassy CSS（SCSS）、webpack 等。使用这些技术，前端代码有时会变得难以理解和调试，正因为如此，程序员经常部署 sourcemap 文件用于调试。"sourcemap" 是一种特殊文件，它将资源的压缩/丑化版本（CSS 或 JavaScript）连接到原始编写版本。程序员仍在争论是否将 sourcemap 文件带到生产环境。然而，不可否认的是，如果 sourcemap 文件或调试文件发布到生产环境，将使其源代码更易读。它可能使攻击者更容易从端发现漏洞或从中收集敏感信息。应进行 JavaScript 代码审查以确定是否有任何调试文件从前端暴露。根据项目的上下文和敏感性，安全专家应决定文件是否应存在于生产环境中。

## 测试目标

- 审查网页注释、元数据和重定向正文以发现任何信息泄露。
- 收集 JavaScript 文件并审查 JS 代码以更好地理解应用并发现任何信息泄露。
- 识别是否存在 sourcemap 文件或其他前端调试文件。

## 如何测试

### 审查网页注释和元数据

HTML 注释经常被开发人员用于包含关于应用的调试信息。有时，他们忘记注释并将它们留在生产环境中。测试人员应查找以 `<!--` 开头的 HTML 注释。

检查 HTML 源代码中是否包含有助于攻击者获取更多关于应用的信息的敏感信息的注释。它可能是 SQL 代码、用户名和密码、内部 IP 地址或调试信息。

```html
...
<div class="table2">
  <div class="col1">1</div><div class="col2">Mary</div>
  <div class="col1">2</div><div class="col2">Peter</div>
  <div class="col1">3</div><div class="col2">Joe</div>

<!-- Query: SELECT id, name FROM app.users WHERE active='1' -->

</div>
...
```

测试人员甚至可能发现如下内容：

```html
<!-- Use the DB administrator password for testing:  f@keP@a$$w0rD -->
```

检查 HTML 版本信息以获取有效版本号和文档类型定义（DTD）URL

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "https://www.w3.org/TR/html4/strict.dtd">
```

- `strict.dtd` -- 默认严格 DTD
- `loose.dtd` -- 松散 DTD
- `frameset.dtd` -- 框架文档的 DTD

一些 `META` 标签不提供主动攻击向量，而是允许攻击者对应用进行画像：

```html
<META name="Author" content="Andrew Muller">
```

一个常见的（但不符合 [WCAG](https://www.w3.org/WAI/standards-guidelines/wcag/)）`META` 标签是 [Refresh](https://en.wikipedia.org/wiki/Meta_refresh)。

```html
<META http-equiv="Refresh" content="15;URL=https://www.owasp.org/index.html">
```

`META` 标签的常见用途是指定搜索引擎可用于提高搜索结果质量的关键字。

```html
<META name="keywords" lang="en-us" content="OWASP, security, sunshine, lollipops">
```

尽管大多数 Web 服务器通过 `robots.txt` 文件管理搜索引擎索引，但也可以通过 `META` 标签管理。以下标签将告知机器人不要索引和不跟随包含该标签的 HTML 页面上的链接。

```html
<META name="robots" content="none">
```

[互联网内容选择平台（PICS）](https://www.w3.org/PICS/)和 [Web 描述资源协议（POWDER）](https://www.w3.org/2007/powder/)提供了将元数据与互联网内容关联的基础设施。

### 识别 JavaScript 代码和收集 JavaScript 文件

程序员经常在前端将敏感信息硬编码到 JavaScript 变量中。测试人员应检查 HTML 源代码并查找 `<script>` 和 `</script>` 标签之间的 JavaScript 代码。测试人员还应识别外部 JavaScript 文件以审查代码（JavaScript 文件具有文件扩展名 `.js`，JavaScript 文件的名称通常放在 `<script>` 标签的 `src`（源）属性中）。

检查 JavaScript 代码中是否有攻击者可用于进一步滥用或操纵系统的敏感信息泄露。查找以下值：API 密钥、内部 IP 地址、敏感路由或凭据。例如：

```javascript
const myS3Credentials = {
  accessKeyId: config('AWSS3AccessKeyID'),
  secretAccessKey: config('AWSS3SecretAccessKey'),
};
```

测试人员甚至可能发现如下内容：

```javascript
var conString = "tcp://postgres:1234@localhost/postgres";
```

当发现 API 密钥时，测试人员可以检查 API 密钥限制是否按服务或 IP、HTTP referrer、应用、SDK 等设置。

例如，如果测试人员发现 Google Map API 密钥，他们可以检查此 API 密钥是否按 IP 限制或仅按 Google Map API 限制。如果 Google API 密钥仅按 Google Map API 限制，攻击者仍然可以使用该 API 密钥查询不受限制的 Google Map API，而应用程序所有者必须为此付费。

```html

<script type="application/json">
...
{"GOOGLE_MAP_API_KEY":"AIzaSyDUEBnKgwiqMNpDplT6ozE4Z0XxuAbqDi4", "RECAPTCHA_KEY":"6LcPscEUiAAAAHOwwM3fGvIx9rsPYUq62uRhGjJ0"}
...
</script>
```

在某些情况下，测试人员可能从 JavaScript 代码中发现敏感路由，如内部或隐藏管理页面的链接。

```html
<script type="application/json">
...
"runtimeConfig":{"BASE_URL_VOUCHER_API":"https://sticker-voucher.victim.net/api", "BASE_BACKOFFICE_API":"https://10.10.10.2/api", "ADMIN_PAGE":"/hidden_administrator"}
...
</script>
```

### 识别 Sourcemap 文件

当 DevTools 打开时，sourcemap 文件通常会被加载。测试人员也可以通过在每个外部 JavaScript 文件的扩展名后添加 ".map" 扩展名来查找 sourcemap 文件。例如，如果测试人员看到 `/static/js/main.chunk.js` 文件，他们可以通过访问 `/static/js/main.chunk.js.map` 来检查其 sourcemap 文件。

检查 sourcemap 文件中是否有可帮助攻击者获取更多关于应用信息的敏感信息。例如：

```json
{
  "version": 3,
  "file": "static/js/main.chunk.js",
  "sources": [
    "/home/sysadmin/cashsystem/src/actions/index.js",
    "/home/sysadmin/cashsystem/src/actions/reportAction.js",
    "/home/sysadmin/cashsystem/src/actions/cashoutAction.js",
    "/home/sysadmin/cashsystem/src/actions/userAction.js",
    "..."
  ],
  "..."
}
```

当站点加载 sourcemap 文件时，前端源代码将变得可读且更易于调试。

### 识别泄露信息的重定向响应

虽然重定向响应通常不被期望包含任何重要的 Web 内容，但不能保证它们不能包含内容。因此，虽然 300 系列（重定向）响应通常包含"重定向到 `https://example.com/`"类型内容，但它们也可能泄露内容。

考虑重定向响应是认证或授权检查结果的情况，如果该检查失败，服务器可能响应将用户重定向回"安全"或"默认"页面，但重定向响应本身可能仍包含不在浏览器中显示但确实传输给客户端的内容。这可以通过利用浏览器开发者工具或通过个人代理（如 ZAP、Burp、Fiddler 或 Charles）看到。

### 审查生成的文件以发现元数据泄露

Web 应用可能生成可下载文件，如 PDF、电子表格、发票或报告。这些文件可能包含描述用于创建它们的软件或库的嵌入式元数据。

*Producer*、*Creator*、*Application* 或 *Library Version* 等元数据字段可能泄露后端技术和特定版本号。此信息可帮助攻击者对应用进行指纹识别并识别公开已知的漏洞（CVE）。

例如，生成的 PDF 可能暴露用于创建它的库：

`Producer: iText 2.1.7`

攻击者可以搜索影响该版本的已知漏洞。

#### 如何测试

从应用下载生成的文件并检查其元数据。

对于 PDF 文件：

```bash
exiftool file.pdf
pdfinfo file.pdf
strings file.pdf | grep -i producer
```

示例输出：

`Producer: iText 2.1.7`

对于 Office 文档（DOCX、XLSX）：

```bash
exiftool file.docx
```

检查以下元数据字段：

- Producer
- Creator
- Application
- Creation Tool
- Library Version

## 工具

- [Wget](https://www.gnu.org/software/wget/wget.html)
- 浏览器"查看源代码"功能
- 肉眼检查
- [Curl](https://curl.haxx.se/)
- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
- [Google Maps API Scanner](https://github.com/ozguralp/gmapsapiscanner/)
- [exiftool](https://exiftool.org/)

## 参考资料

- [KeyHacks](https://github.com/streaak/keyhacks)
- [RingZer0 Online CTF](https://ringzer0ctf.com/challenges/104) - 挑战 104 "Admin Panel"。

### 白皮书

- [HTML 版本 4.01](https://www.w3.org/TR/1999/REC-html401-19991224)
- [XHTML](https://www.w3.org/TR/2010/REC-xhtml-basic-20101123/)
- [HTML 版本 5](https://www.w3.org/TR/html5/)
