# 识别 Web 应用框架指纹

|ID          |
|------------|
|WSTG-INFO-08|

## 摘要

可以说，几乎所有可想象的 Web 应用创意都已经进入开发阶段。随着大量活跃开发和部署的免费开源软件项目，非常可能在应用安全测试中遇到完全或部分依赖这些知名应用或框架的目标（如 WordPress、phpBB、Mediawiki 等）。了解被测试的 Web 应用组件显著有助于测试过程，也会大大减少测试期间所需的工作量。这些知名的 Web 应用在这些位置具有特定的 HTML 头、Cookie 和目录结构，可以被枚举以识别应用。大多数 Web 框架在这些位置都有几个标记，可以帮助攻击者或测试人员识别它们。这基本上是所有自动工具做什么，它们从预定义位置查找标记，然后与已知签名的数据库进行比较。为了更高的准确性，通常使用几个标记。

## 测试目标

- 对 Web 应用使用的组件进行指纹识别。

## 如何测试

### 黑盒测试

有几个需要考虑的常见位置以识别框架或组件：

- HTTP 头
- Cookie
- HTML 源代码
- 特定文件和文件夹
- 文件扩展名
- 错误消息

#### HTTP 头

识别 Web 框架的最基本形式是查看 HTTP 响应头中的 `X-Powered-By` 字段。许多工具可用于对目标进行指纹识别，最简单的是 netcat。

考虑以下 HTTP 请求-响应：

```html
$ nc 127.0.0.1 80
HEAD / HTTP/1.0

HTTP/1.1 200 OK
Server: nginx/1.0.14
[...]
X-Powered-By: Mono
```

从 `X-Powered-By` 字段，我们了解 Web 应用框架可能是 `Mono`。然而，虽然这种方法简单快速，但并非在所有情况下都有效。可以通过正确配置轻松禁用 `X-Powered-By` 头。还有几种允许站点混淆 HTTP 头的技术（参见[修复](#修复)部分中的示例）。在上面的示例中，我们还可以注意到特定版本的 `nginx` 正用于服务内容。

在同一示例中，测试人员可能错过 `X-Powered-By` 头或获得如下响应：

```html
HTTP/1.1 200 OK
Server: nginx/1.0.14
Date: Sat, 07 Sep 2013 08:19:15 GMT
Content-Type: text/html;charset=ISO-8859-1
Connection: close
Vary: Accept-Encoding
X-Powered-By: Blood, sweat and tears
```

有时有更多 HTTP 头指向某个框架。在以下示例中，根据 HTTP 请求的信息，可以看到 `X-Powered-By` 头包含 PHP 版本。然而，`X-Generator` 头指出实际使用的框架是 `Swiftlet`，这有助于渗透测试人员扩展攻击向量。执行指纹识别时，仔细检查每个 HTTP 头以发现此类泄露。

```html
HTTP/1.1 200 OK
Server: nginx/1.4.1
Date: Sat, 07 Sep 2013 09:22:52 GMT
Content-Type: text/html
Connection: keep-alive
Vary: Accept-Encoding
X-Powered-By: PHP/5.4.16-1~dotdeb.1
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
X-Generator: Swiftlet
```

#### Cookie

另一种类似且更可靠地确定当前 Web 框架的方法是使用特定框架的 Cookie。

考虑以下 HTTP 请求：

![Cakephp HTTP 请求](images/Cakephp_cookie.png)\
*图 4.1.8-7：Cakephp HTTP 请求*

Cookie `CAKEPHP` 已自动设置，这提供了正在使用的框架信息。[Cookie](#cookies-1) 部分提供常见 Cookie 名称列表。在此识别机制上仍然存在限制——可以更改 Cookie 的名称。例如，对于选定的 `CakePHP` 框架，这可以通过以下配置完成（摘自 `core.php`）：

```php
/**
 * The name of CakePHP's session cookie.
 *
 * Note the guidelines for Session names states: "The session name references
 * the session id in cookies and URLs. It should contain only alphanumeric
 * characters."
 * @link https://php.net/session_name
 */
Configure::write('Session.cookie', 'CAKEPHP');
```

然而，与更改 `X-Powered-By` 头相比，这些更改不太可能被制作，使这种识别方法更可靠。

#### HTML 源代码

这种技术基于在 HTML 页面源代码中查找某些模式。通常可以发现大量信息，帮助测试人员识别特定组件。常见的标记之一是直接导致框架披露的 HTML 注释。更常见的是，可以找到特定框架的路径，即指向框架特定 CSS 或 JS 文件夹的链接。最后，特定脚本变量也可能指向某个框架。

从下面的屏幕截图中，人们可以容易地通过提到的标记确定正在使用的框架及其版本。注释、特定路径和脚本变量都可以帮助攻击者快速确定 ZK 框架实例。

![ZK 框架示例](images/Zk_html_source.png)\
*图 4.1.8-2：ZK 框架 HTML 源代码示例*

通常此类信息位于 HTTP 响应的 `<head>` 部分、`<meta>` 标签中，或在页面末尾。然而，应该分析整个响应，因为它可用于其他目的，如检查其他有用的注释和隐藏字段。然而，Web 开发人员可能没有充分隐藏关于所使用的框架或组件的信息。仍然可能偶然发现类似在页面底部的内容：

![Banshee 底部页面](images/Banshee_bottom_page.png)\
*图 4.1.8-3：Banshee 底部页面*

### 特定文件和文件夹

还有另一种方法极大地帮助攻击者或测试人员高精度识别应用或组件。每个 Web 应用组件在服务器上都有其特定的文件和文件夹结构。已注意到可以从 HTML 页面源代码看到特定路径，但有时它们没有明确显示，仍可能驻留在服务器上。

为了揭露它们，使用了一种称为强制浏览或"dirbusting"的技术。Dirbusting 使用已知文件夹和文件名对目标进行暴力破解，并监控 HTTP 响应以枚举服务器内容。此信息可用于查找默认文件并攻击它们，以及对 Web 应用进行指纹识别。Dirbusting 可以通过多种方式进行，以下示例在 Burp Suite 的定义列表和 intruder 功能的帮助下显示了对 WordPress 支持的目标的成功 dirbusting 攻击。

![使用 Burp 进行 Dirbusting](images/Wordpress_dirbusting.png)\
*图 4.1.8-4：使用 Burp 进行 Dirbusting*

我们可以看到，对于某些 WordPress 特定文件夹（例如 `/wp-includes/`、`/wp-admin/` 和 `/wp-content/`），HTTP 响应分别是 403（禁止）、302（找到，重定向到 `wp-login.php`）和 200（确定）。这是目标由 WordPress 支持的一个很好的指示器。同样，可以对不同的应用插件文件夹及其版本进行 dirbusting。在下面的屏幕截图中，我们可以看到典型 Drupal 插件的 CHANGELOG 文件，它提供了关于正在使用的应用的信息，并披露了易受攻击的插件版本。

![Drupal Botcha 披露](images/Drupal_botcha_disclosure.png)\
*图 4.1.8-5：Drupal Botcha 披露*

提示：在开始 dirbusting 之前，首先检查 `robots.txt` 文件。有时应用特定文件夹和其他敏感信息也可以在那里找到。这样的 `robots.txt` 文件的示例在下面的屏幕截图中显示。

![Robots 信息披露](images/Robots-info-disclosure.png)\
*图 4.1.8-6：Robots 信息披露*

特定文件和文件夹每个特定应用都不同。如果识别的应用或组件是开源的，在渗透测试期间设置临时安装可能很有价值，以更好地理解呈现的基础设施或功能，以及可能留在服务器上的文件。然而，已经存在几个有用的文件列表；一个值得注意的示例是 [FuzzDB 可预测文件/文件夹词表](https://github.com/fuzzdb-project/fuzzdb)。

#### 文件扩展名

URL 可能包含文件扩展名，这些扩展名也可以帮助识别 Web 平台或技术。

例如，OWASP wiki 使用 PHP：

```text
https://wiki.owasp.org/index.php?title=Fingerprint_Web_Application_Framework&action=edit&section=4
```

以下是一些常见的 Web 文件扩展名及相关技术：

- `.php` -- PHP
- `.aspx` -- Microsoft ASP.NET
- `.jsp` -- Java Server Pages

#### 错误消息

如下图所示，列出的文件系统路径指向使用 WordPress（`wp-content`）。此外，测试人员应该知道 WordPress 是基于 PHP 的（`functions.php`）。

![WordPress 解析错误](images/wp-syntaxerror.png)\
*图 4.1.8-7：WordPress 解析错误*

## 常见标识符

### Cookie

| 框架 | Cookie 名称 |
|--------------|-----------------------------------|
| Zope | zope3 |
| CakePHP | cakephp |
| Kohana | kohanasession |
| Laravel | laravel_session |
| phpBB | phpbb3_ |
| WordPress | wp-settings |
| 1C-Bitrix | BITRIX_ |
| AMPcms | AMP |
| Django CMS | django |
| DotNetNuke | DotNetNukeAnonymous |
| e107 | e107_tz |
| EPiServer | EPiTrace, EPiServer |
| Graffiti CMS | graffitibot |
| Hotaru CMS | hotaru_mobile |
| ImpressCMS | ICMSession |
| Indico | MAKACSESSION |
| InstantCMS | InstantCMS[logdate] |
| Kentico CMS | CMSPreferredCulture |
| MODx | SN4[12symb] |
| TYPO3 | fe_typo_user |
| Dynamicweb | Dynamicweb |
| LEPTON | lep[some_numeric_value]+sessionid |
| Wix | Domain=.wix.com |
| VIVVO | VivvoSessionId |
| Tiny File Manager | filemanager |
| Zenphoto | zenphoto_auth |

### HTML 源代码

| 应用 | 关键字 |
|-------------|--------------------------------------------------------------------------------|
| WordPress | `<meta name="generator" content="WordPress 3.9.2" />` |
| phpBB | `<body id="phpbb"` |
| Mediawiki | `<meta name="generator" content="MediaWiki 1.21.9" />` |
| Joomla | `<meta name="generator" content="Joomla! - Open Source Content Management" />` |
| Drupal | `<meta name="Generator" content="Drupal 7 (https://drupal.org)" />` |
| DotNetNuke | `DNN Platform - [https://www.dnnsoftware.com](https://www.dnnsoftware.com)` |

#### 通用标记

- `%framework_name%`
- `powered by`
- `built upon`
- `running`

#### 特定标记

| 框架 | 关键字 |
|-------------------|--------------------------------|
| Adobe ColdFusion | `<!-- START headerTags.cfm` |
| Microsoft ASP.NET | `__VIEWSTATE` |
| ZK | `<!-- ZK` |
| Business Catalyst | `<!-- BC_OBNW -->` |
| Indexhibit | `ndxz-studio` |

## 修复

虽然可以尝试使用不同的 Cookie 名称（通过更改配置）、隐藏或更改文件/目录路径（通过重写或源代码更改）、删除已知头等，但这些努力归结为"通过模糊实现安全"。系统所有者/管理员应认识到，这些努力只会减缓最原始的对手。时间和精力最好用于提高相关方意识和维护解决方案。

## 工具

下面提供了一个通用和知名工具的列表。还有许多其他实用程序以及基于框架的指纹识别工具。

### WhatWeb

网站：[https://github.com/urbanadventurer/WhatWeb](https://github.com/urbanadventurer/WhatWeb)

WhatWeb 是目前市场上最好的开源指纹识别工具之一，包含在默认的 [Kali Linux](https://www.kali.org/) 构建中。语言：Ruby 指纹识别匹配通过以下方式进行：

- 文本字符串（区分大小写）
- 正则表达式
- Google Hack 数据库查询（有限的关键字集）
- MD5 哈希
- URL 识别
- HTML 标签模式
- 用于被动和主动操作的自定义 ruby 代码

下面呈现示例输出屏幕截图：

![Whatweb 输出示例](images/Whatweb-sample.png)\
*图 4.1.8-8：Whatweb 输出示例*

### Wappalyzer

网站：[https://www.wappalyzer.com/](https://www.wappalyzer.com/)

Wappalyzer 有多种使用模式，其中最流行的可能是 Firefox/Chrome 扩展。它们主要通过正则表达式匹配工作，不需要浏览器中加载的页面以外的任何东西。它完全在浏览器级别工作，以图标形式给出结果。虽然有时有误报，但这对于在浏览页面后立即了解构建目标站点的技术非常方便。

下面呈现插件的示例输出屏幕截图。

![Wappalyzer 对 OWASP 网站的输出](images/Owasp-wappalyzer.png)\
*图 4.1.8-9：Wappalyzer 对 OWASP 网站的输出*

## 参考资料

### 白皮书

- [Saumil Shah："HTTP 指纹识别入门"](https://web.archive.org/web/20190526182734/https://net-square.com/httprint_paper.html)
- [Anant Shrivastava："Web 应用指纹识别"](https://anantshri.info/articles/web_app_finger_printing.html)
