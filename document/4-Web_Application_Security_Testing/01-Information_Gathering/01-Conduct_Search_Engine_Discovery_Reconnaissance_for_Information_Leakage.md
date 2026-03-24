# 进行搜索引擎侦察以发现信息泄露

|ID          |
|------------|
|WSTG-INFO-01|

## 摘要

为了让搜索引擎工作，计算机程序（或"机器人"）定期从互联网上数十亿的网页获取数据（称为[爬取](https://en.wikipedia.org/wiki/Web_crawler)）。这些程序通过从其他页面跟随链接，或通过查看站点地图来查找 Web 内容和功能。如果站点使用名为 `robots.txt` 的特殊文件来列出不希望搜索引擎获取的页面，则其中列出的页面将被忽略。这是一个基本概述——Google 提供了关于[搜索引擎如何工作](https://support.google.com/webmasters/answer/70897?hl=en)的更深入解释。

测试人员可以使用搜索引擎对站点和 Web 应用进行侦察。搜索引擎发现和侦察有直接和间接要素：直接方法涉及搜索索引以及来自缓存的相关内容，而间接方法涉及通过搜索论坛、新闻组和招标站点来了解敏感设计和配置信息。

一旦搜索引擎机器人完成爬取，它会根据标签和相关属性（如 `<TITLE>`）开始索引 Web 内容，以便返回相关搜索结果。如果 `robots.txt` 文件在站点生命周期内未更新，并且用于指示机器人不要索引内容的内联 HTML meta 标签未使用，那么索引可能包含非所有者打算包含的 Web 内容。站点所有者可以使用前面提到的 `robots.txt`、HTML meta 标签、认证和搜索引擎提供的工具来删除此类内容。

## 测试目标

- 识别应用中、系统或组织暴露的敏感设计和配置信息是直接暴露（在组织站点上）还是间接暴露（通过第三方服务）。

## 如何测试

使用搜索引擎搜索潜在的敏感信息。这可能包括：

- 网络图表和配置；
- 管理员或其他关键员工的存档帖子和电子邮件；
- 登录程序和用户名格式；
- 用户名、密码和私钥；
- 第三方或云服务配置文件；
- 揭示错误消息内容；和
- 非公开应用（开发、测试、用户验收测试（UAT）和预发布版本的站点）。

### 搜索引擎

不要将测试限制在仅仅一个搜索引擎提供商上，因为不同的搜索引擎可能产生不同的结果。搜索引擎结果可能在几个方面有所不同，这取决于引擎上次爬取内容的时间，以及引擎用于确定相关页面的算法。考虑使用以下（按字母顺序排列的）搜索引擎：

- [百度](https://www.baidu.com/)，中国[最流行](https://en.wikipedia.org/wiki/Web_search_engine#Market_share)的搜索引擎。
- [Bing](https://www.bing.com/)，由微软拥有和运营的搜索引擎，是全球第二大[最流行](https://en.wikipedia.org/wiki/Web_search_engine#Market_share)的搜索引擎。支持[高级搜索关键词](https://help.bing.microsoft.com/#apex/18/en-US/10001/-1)。
- [binsearch.info](https://binsearch.info/)，一个用于二进制 Usenet 新闻组的搜索引擎。
- [Common Crawl](https://commoncrawl.org/)，"一个任何人可以访问和分析的开放 Web 爬取数据存储库"。
- [DuckDuckGo](https://duckduckgo.com/)，一个隐私focused的搜索引擎，从许多不同的[来源](https://help.duckduckgo.com/results/sources/)编译结果。支持[搜索语法](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/)。
- [Google](https://www.google.com/)，提供世界上[最流行](https://en.wikipedia.org/wiki/Web_search_engine#Market_share)的搜索引擎，并使用排名系统尝试返回最相关的结果。支持[搜索运算符](https://support.google.com/websearch/answer/2466433)。
- [Internet Archive Wayback Machine](https://archive.org/web/)，"以数字形式构建互联网站点和其他文化 artifacts 的数字图书馆"。
- [Censys](https://censys.io) 是一个安全focused的搜索引擎，索引互联网连接的基础设施，包括服务器、证书和开放服务。它为有限每月积分的免费社区层级提供付费计划供企业使用。
- [Shodan](https://www.shodan.io/)，一种用于搜索互联网连接设备和服务。 使用选项包括有限的免费计划以及付费订阅计划。

### 搜索运算符

搜索运算符是扩展常规搜索查询能力的特殊关键词或语法，可以帮助获得更具体的结果。它们通常采用 `operator:query` 的形式。以下是一些常用支持的搜索运算符：

- `site:` 将把搜索限制在提供的域名。
- `inurl:` 将只返回 URL 中包含关键词的结果。
- `intitle:` 将只返回页面标题中包含关键词的结果。
- `intext:` 或 `inbody:` 将只搜索页面正文中 的关键词。
- `filetype:` 将只匹配特定的文件类型，即 `.png` 或 `.php`。

例如，要找到典型搜索引擎索引的 owasp.org 的 Web 内容，所需的语法是：

```text
site:owasp.org
```

![Google Site 运算符搜索结果示例](images/Google_site_Operator_Search_Results_Example_20200406.png)\
*图 4.1.1-1：Google Site 运算符搜索结果示例*

#### Internet Archive Wayback Machine

[Internet Archive Wayback Machine](https://archive.org/web/) 是查看网页历史快照最全面的工具。它维护着可追溯到 1996 年的网页的广泛存档。

要查看站点的存档版本，请访问 `https://web.archive.org/web/*/` 后跟目标 URL：

```text
https://web.archive.org/web/*/owasp.org
```

这将显示一个日历视图，显示站点随时间的所有可用快照。

#### 其他缓存内容服务

用于查看缓存或存档网页的其他服务包括：

- [archive.ph](https://archive.ph)（也称为 archive.md）- 按需存档服务，创建永久快照
- [CachedView](https://cachedview.com/) - 聚合来自多个来源的缓存页面，包括 Google Cache 历史数据、Wayback Machine 等

### Google 黑客或 Dorking

当与测试人员的创造力结合时，使用运算符搜索可以成为一种非常有效的发现技术。运算符可以链接以有效地发现特定类型的敏感文件和信息。这种技术称为 [Google 黑客](https://en.wikipedia.org/wiki/Google_hacking) 或 Dorking，也可以使用其他搜索引擎，只要支持搜索运算符。

dorks 数据库，如 [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)，是一个有用的资源，可以帮助发现特定信息。AI 辅助查询生成器如 [DorkGPT](https://www.dorkgpt.com) 可以将自然语言提示转换为 Google dork 语法，减少构造复杂搜索运算符的手动工作。此数据库中可用的一些 dorks 类别包括：

- 立足点
- 包含用户名的文件
- 敏感目录
- Web 服务器检测
- 漏洞文件
- 漏洞服务器
- 错误消息
- 包含 juicy 信息的文件
- 包含密码的文件
- 敏感的在线购物信息

## OSINT 关联工具

除了个别搜索引擎，测试人员可以使用专门的 OSINT 框架来关联和可视化发现实体之间的关系：

[Maltego](https://maltego.com) 是一个行业标准的 OSINT 和链接分析平台，通过自动化数据转换映射域名、IP 地址、电子邮件地址和组织之间的关系。测试人员使用它来通过从单个实体 pivot 来可视化组织的攻击面，以发现相关基础设施和相关数据点。免费社区版可用于非商业用途。

## 修复

在在线发布之前，仔细考虑设计和配置信息的敏感性。

定期审查在线发布的设计和配置信息的敏感性。
