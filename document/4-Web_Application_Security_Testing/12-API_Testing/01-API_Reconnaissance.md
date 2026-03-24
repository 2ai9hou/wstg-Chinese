# API侦察

|ID          |
|------------|
|WSTG-APIT-01|

## 概述

侦察在任何渗透测试参与中都是重要步骤。这包括API渗透测试。侦察通过收集有关API的信息并开发对目标的理解，显著提高测试过程的有效性。此阶段不仅增加了发现关键安全问题的可能性，还确保了对API安全状况的全面评估。

本指南有[信息收集](../01-Information_Gathering/README.md)部分，适用于审计API时。但是，有一些区别。作为安全研究人员，我们经常关注特定区域，搜索本指南中适用的部分可能非常耗时。为确保研究人员有一个专注于API的单一位置，本节集中于适用于API的项目，并提供本指南其他地方的参考。

### API类型

API可以是公共的或私有的。

#### 公共API

公共API通常在Swagger/OpenAPI文档中发布其详细信息。获取此文档以了解攻击面非常重要。同样重要的是找到可能显示已弃用但仍有效代码的此文档的旧版本，这些代码可能具有安全漏洞。

但是，请记住，无论多好意图，此文档可能不准确，也可能不会披露完整API。

公共API也可能在共享库或API目录中记录。

#### 私有API

私有API的可见性取决于预期消费者是谁。API可以是私有的，但仅可供订阅客户端（也称为"合作伙伴"）访问，或者仅供内部客户端访问，例如同公司内的其他部门。使用侦察技术发现私有API也很重要。可以使用多种技术发现这些API，我们将在下面讨论。

## 测试目标

- 找到后端服务器代码支持的所有API端点，有文档或无文档。
- 找到后端服务器支持的每个端点的所有参数，有文档或无文档。
- 在发送给客户端的HTML和JavaScript中发现与API相关的有趣数据。

## 如何测试

### 找到文档

在公共和私有情况下，API文档将基于其质量和准确性有用。公共API文档通常与所有人共享，而私有API文档仅与预期客户端共享。但是，在这两种情况下，找到文档、意外泄露或其他方式都将帮助您的调查。

无论API的可见性如何，搜索API文档可以找到尚未发布的旧版本或意外泄露的API文档。此文档将非常有助于理解API暴露的攻击面。

### API目录

API文档的其他来源包括API目录，例如：

- GitHub总体
- [GitHub公共API仓库](https://github.com/public-apis/public-apis)
- [APIs.guru](https://apis.guru)
- [RapidAPI](https://rapidapi.com/)
- [PublicAPIs](https://publicapis.dev/)和[PublicAPIs](https://publicapis.io/)
- [Postman API网络](https://www.postman.com/explore)

### 在知名位置查找

如果文档不明显，您可以基于一些明显名称或路径主动搜索目标上的文档。这些包括：

- /api-docs
- /doc
- /swagger
- /swagger.json
- /openapi.json
- /.well-known/schema-discovery

### Robots.txt

`robots.txt`是站点所有者创建的文本文件，用于指示Web爬虫（如搜索引擎机器人）如何爬网和索引其站点。它是robots排除协议（REP）的一部分，该协议规范爬虫如何与站点交互。

此文件可能提供路径结构或API端点的额外线索。

[信息收集](../01-Information_Gathering/README.md)部分在多种情况下引用robots.txt，包括WSTG-INFO-01、WSTG-INFO-03、WSTG-INFO-05和WSTG-INFO-08。

### GitDorking

如果应用程序使用GitHub、GitLab或其他面向公众的Git仓库，我们也可以搜索任何线索或敏感内容（也称为"GitDorking"）。此信息可能包括密码、API密钥、配置文件以及开发人员可能意外或无意提交到仓库的其他机密数据。组织可能意外共享敏感代码、示例或测试代码，这些可能提供实施细节的线索。目标的员工的个人GitHub账户也可能意外发布可能提供线索的信息。

### 浏览和爬网应用程序

即使您有API文档，浏览应用程序也是一个好主意。文档可能已过时、不准确或不完整。

使用ZAP或Burp Suite等拦截代理浏览应用程序会记录端点供以后检查。此外，使用其内置爬网功能，拦截代理可以帮助生成全面的端点列表。从爬网的URL中，寻找具有明显API URL命名方案的链接。这些包括：

- `https://example.com/api/v1`（或v2等）
- `https://example.com/graphql`

或应用程序可能消费或依赖的子域：

- `https://api.example.com/api/v1`

重要的是，渗透测试人员应尝试在应用程序中行使尽可能多的功能。这不仅是为了生成全面的端点列表，还为了避免延迟加载和代码分割的问题。此外，您的渗透测试参与应包括不同权限级别的样本账户，以便您的浏览器和爬网可以访问并暴露尽可能多的功能端点。

完成后，从浏览和爬网应用程序获得的端点信息可以帮助渗透测试人员使用Postman等其他工具编写目标API文档。

### Google Dorking

使用被动侦察技术（如Google Dorking）及`site`和`inurl`指令，允许我们定制搜索Google索引者可能找到的常见API关键字。以下是一些特定API示例：

`site:"mytargetsite.com" inurl:"/api"`

`inurl:apikey filetype:env`

其他关键字可以包括`"v1"`、`"api"`、`"graphql"`。

我们可以扩展Google Dorking以包括目标的子域。

词表在此处有帮助，可以获取全面的常用单词列表。

### 回溯，追溯

一般来说，API会随着时间变化。但弃用或旧版本可能仍可操作，无论是故意还是由于错误配置。这些也应该测试，因为它们很可能包含已在新版本中修复的漏洞。此外，API的变化显示了新功能，这些功能可能更健壮，因此是测试的良好候选。

为了发现旧版本，我们可以使用`Wayback machine`帮助找到旧端点。一个名为TomNomNom的[WayBackUrls](https://github.com/tomnomnom/waybackurls)的有用工具获取Wayback Machine知道的所有URL用于域。

- [WayBackUrls](https://github.com/tomnomnom/waybackurls)。获取Wayback Machine知道的所有URL用于域。
- [waymore](https://github.com/xnl-h4ck3r/waymore)。从Wayback Machine、Common Crawl、Alien Vault OTX、URLScan和VirusTotal找到更多。
- [gau](https://github.com/lc/gau)。从AlienVault的开放威胁交换、Wayback Machine和Common Crawl获取已知URL。

### 客户端应用程序

服务器发送到客户端的HTML和JavaScript是API和其他信息的极佳来源。有时，客户端应用程序泄露敏感信息，包括API和秘密。[审查网页内容以查找信息泄露](../01-Information_Gathering/05-Review_Web_Page_Content_for_Information_Leakage.md)部分有用于审查网页内容泄露的一般信息。在这里，我们将扩展重点放在JavaScript内容中API相关秘密的审查。

有多种工具可帮助我们从发送到浏览器的JavaScript中提取敏感信息。这些工具通常基于两种方法之一：正则表达式或抽象语法树（AST）。然后有通用工具可帮助我们组织或管理JS文件以进行AST和正则表达式工具调查。

正则表达式更简单，通过搜索JS或HTML内容中已知模式。但是，此方法可能无法识别正则表达式中未明确标识的内容。给定某些JS的结构，此方法可能会遗漏很多。AST另一方面是表示代码语法的树状结构。树中的每个节点对应代码的一部分。对于JavaScript，AST将代码分解为基本组件，允许工具和编译器轻松理解和修改代码。

#### 通用工具

1. [Uproot](https://github.com/0xDexter0us/uproot-JS)。一个BurpSuite插件，将遇到的任何JS文件保存到磁盘。这有助于提取文件以供任何命令行工具分析。
2. [OpenAPI支持](https://www.zaproxy.org/docs/desktop/addons/openapi-support/)。此ZAP附加组件允许您爬网和导入OpenAPI（Swagger）定义，版本1.2、2.0和3.0。
3. [OpenAPI解析器](https://github.com/aress31/openapi-parser)。一个BurpSuite插件，将OpenAPI文档解析为Burp Suite，用于自动化基于OpenAPI的API安全评估。

#### 正则表达式工具

1. [JSParser](https://github.com/nahamsec/JSParser)。一个使用Tornado和JSBeautifier的python 2.7脚本，用于从JavaScript文件解析相对URL。
2. [JSMiner](https://github.com/PortSwigger/js-miner)。一个BurpSuite插件，尝试在静态文件中找到有趣的内容；主要是JavaScript和JSON文件。此工具在爬网应用程序时"被动"扫描。
3. [JSpector](https://github.com/hisxo/JSpector)。一个BurpSuite插件，被动爬网JavaScript文件，自动创建在JS文件上找到的URL、端点和危险方法的问题。
4. [Link Finder](https://github.com/GerbenJavado/LinkFinder)。一个python脚本，用于在JavaScript文件中查找端点。

#### AST工具

1. [JSLuice](https://github.com/BishopFox/jsluice)。一个命令行工具，从JavaScript源代码中提取URL、路径、秘密和其他有趣数据。

### 其他侦察工具

1. [攻击面检测器](https://github.com/secdec/attack-surface-detector-burp)。一个BurpSuite插件，使用静态代码分析通过解析路由和识别参数来识别Web应用端点。
2. [Param Miner](https://github.com/portswigger/param-miner)。一个BurpSuite插件，识别隐藏的、未链接的参数。
3. [xnLinkFinder](https://github.com/xnl-h4ck3r/xnLinkFinder)。一个python工具，用于发现给定目标的端点、潜在参数和目标特定词表。
4. [GAP](https://github.com/xnl-h4ck3r/GAP-Burp-Extension)。Burp扩展，用于发现潜在端点、参数，并生成自定义目标词表。

### 主动模糊测试

主动模糊测试涉及使用具有词表的工具和过滤请求结果以进行暴力端点发现。

#### Kiterunner

[KiteRunner](https://github.com/assetnote/kiterunner)是一种在现代应用程序和API中执行传统内容发现和暴力路由/端点的工具。

```console
kr [scan|brute] <input> [flags]
```

要使用词表扫描目标的API：

```console
kr scan https://example.com/api -w /usr/share/wordlists/apis/routes-large.kite --fail-status-codes 404,403
```

#### FFUF/DirBuster/GoBuster

这三个工具都旨在通过暴力技术发现Web服务器上的隐藏路径和文件。三个都使用可自定义词表生成对目标Web服务器的请求，尝试识别有效目录和文件。三个都支持多线程或高效处理以加速暴力过程。

API的一些常见词表文件包括：[SecLists](https://github.com/danielmiessler/SecLists)在Discovery/Web-Content/api部分、[GraphQL词表](https://github.com/Escape-Technologies/graphql-wordlist)和[Assetnote](https://wordlists.assetnote.io/)。

GoBuster示例：

`gobuster dir -u <target url> -w <wordlist file>`

## 参考资料

### OWASP资源

- [REST评估速查表](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)

### 书籍

- Corey J. Ball - "Hacking APIs : breaking web application programming interfaces"，No Starch，2022 - ISBN-13: 978-1-7185-0244-4
- Confidence Staveley - "API Security for White Hat Hackers"，Packt，2024 - ISBN 978-1-80056-080-2
