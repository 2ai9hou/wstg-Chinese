# 测试应用平台配置

|ID          |
|------------|
|WSTG-CONF-02|

## 摘要

正确配置组成应用架构的各个元素对于防止可能危害整个架构的错误非常重要。

审查和测试配置是创建和维护架构的关键任务。这是因为各种系统通常包含通用配置，可能与它们应该在安装的特定站点上执行的任务不对齐。

虽然典型的 Web 和应用服务器安装将包含许多功能（如应用示例、文档、测试页面），但在部署之前应删除非必要内容，以避免安装后利用。

## 测试目标

- 确保已删除默认和已知文件。
- 验证生产环境中没有留下调试代码或扩展。
- 审查为应用设置的日志机制。

## 如何测试

### 黑盒测试

#### 示例和已知文件与目录

在默认安装中，许多 Web 服务器和应用服务器为开发人员提供示例应用和文件，以在安装后正确测试服务器是否正常工作。然而，众所周知，许多默认 Web 服务器应用后来已被发现存在漏洞。例如，CVE-1999-0449（安装 Exair 示例站点时 IIS 中的拒绝服务）、CAN-2002-1744（Microsoft IIS 5.0 中 CodeBrws.asp 中的目录遍历漏洞）、CAN-2002-1630（Oracle 9iAS 中使用 sendmail.jsp）或 CAN-2003-1172（Apache 的 Cocoon 中 view-source 示例中的目录遍历）。

包含不同 Web 或应用服务器提供的已知文件和目录样本详细列表的 CGI 扫描器可能是快速确定这些文件是否存在的方法。但是，真正确保的唯一方法是完整审查 Web 服务器或应用服务器的内容，并确定它们是否与应用本身相关。

#### 注释审查

程序员在开发大型基于 Web 的应用时添加注释是非常常见的。但是，HTML 代码中包含的内联注释可能泄露不应被攻击者获取的内部信息。有时，当功能不再需要时，源代码的一部分被注释掉了，但这个注释无意中泄露给了返回给用户的 HTML 页面。

应进行注释审查以确定是否有信息通过注释泄露。只有通过分析 Web 服务器的静态和动态内容以及文件搜索才能彻底完成此审查。自动或引导方式浏览站点并存储所有检索内容可能很有用。然后可以搜索此检索内容以分析代码中可用的任何 HTML 注释。

#### 系统配置

各种工具、文档或检查清单可用于为 IT 和安全专业人员提供关于目标系统符合各种配置基线或基准的详细评估。此类工具包括但不限于：

- [CIS-CAT Lite](https://www.cisecurity.org/blog/introducing-cis-cat-lite/)
- [Microsoft 的攻击面分析器](https://github.com/microsoft/AttackSurfaceAnalyzer)
- [NIST 的国家检查清单程序](https://nvd.nist.gov/ncp/repository)

### 灰盒测试

#### 配置审查

Web 服务器或应用服务器配置在保护站点内容方面起着重要作用，必须仔细审查以发现常见配置错误。显然，推荐的配置因站点策略和应由服务器软件提供的功能而异。但是，在大多数情况下，应遵循配置指南（由软件供应商或外部方提供）以确定服务器是否已正确保护。

不可能泛泛地说服务器应如何配置，但是，应考虑一些常见指南：

- 仅启用应用所需的服务器模块（IIS 情况下为 ISAPI 扩展）。这减少了攻击面，因为随着软件模块被禁用，服务器的大小和复杂性减小。如果漏洞仅存在于已禁用的模块中，它还能防止供应商软件中出现的漏洞影响站点。
- 使用定制的页面而不是默认的 Web 服务器页面处理服务器错误（40x 或 50x）。特别确保任何应用错误都不会返回给最终用户，并且不会通过这些错误泄露任何代码，因为它将帮助攻击者。实际上，由于开发人员在预生产环境中需要此信息，因此很容易忘记这一点。
- 确保服务器软件在操作系统中以最小特权运行。这防止了服务器软件中的错误直接危害整个系统，尽管一旦作为 Web 服务器运行代码，攻击者可以提升权限。
- 确保服务器软件正确记录合法访问和错误。
- 确保服务器配置正确处理过载并防止拒绝服务攻击。确保服务器已正确调整性能。
- 切勿授予非管理标识（`NT SERVICE\WMSvc` 除外）对 applicationHost.config、redirection.config 和 administration.config 的访问权限（读取或写入）。这包括 `Network Service`、`IIS_IUSRS`、`IUSR` 或 IIS 应用程序池使用的任何自定义标识。IIS 工作进程不旨在直接访问任何这些文件。
- 切勿在网络上共享 applicationHost.config、redirection.config 和 administration.config。使用共享配置时， prefer to export applicationHost.config 到另一个位置（参见"为共享配置设置权限"部分）。
- 请记住，默认情况下所有用户都可以读取 .NET Framework `machine.config` 和根 `web.config` 文件。如果敏感信息仅供管理员查看，请不要将这些文件存储在其中。
- 加密应仅由 IIS 工作进程读取而非计算机上的其他用户读取的敏感信息。
- 不要授予 Web 服务器用于访问共享 `applicationHost.config` 的标识写入访问权限。此标识应仅具有读取访问权限。
- 使用单独标识将 applicationHost.config 发布到共享。不要使用此标识在 Web 服务器上配置对共享配置的访问。
- 导出与共享配置一起使用的加密密钥时使用强密码。
- 维护对包含共享配置和加密密钥的共享的受限访问。如果此共享被泄露，攻击者将能够读取和写入 Web 服务器的任何 IIS 配置，将流量从站点重定向到恶意来源，在某些情况下通过将任意代码加载到 IIS 工作进程中来控制所有 Web 服务器。
- 考虑用防火墙规则和 IPsec 策略保护此共享，仅允许成员 Web 服务器连接。

#### 日志记录

日志是应用架构安全的重要资产，因为它可用于检测应用缺陷（如用户持续尝试检索实际不存在的文件）以及来自恶意用户的持续攻击。日志通常由 Web 和其他服务器软件正确生成。很少发现有应用正确将其操作记录到日志中，而且，当它们这样做时，应用日志的主要目的是产生可用于程序员分析特定错误的调试输出。

在两种情况下（服务器和应用日志），应根据日志内容测试和分析几个问题：

1. 日志是否包含敏感信息？
2. 日志是否存储在专用服务器上？
3. 日志使用是否会产生拒绝服务条件？
4. 它们如何轮换？日志是否保留了足够的时间？
5. 如何审查日志？管理员能否使用这些审查来检测有针对性的攻击？
6. 日志备份如何保存？
7. 在记录之前，是否对正在记录的数据进行验证（最小/最大长度、字符等）？

##### 日志中的敏感信息

一些应用可能使用 GET 请求转发可以在服务器日志中看到的表单数据。这意味着服务器日志可能包含敏感信息（如用户名和密码或银行账户详细信息）。如果攻击者获得日志（例如通过管理界面或已知的 Web 服务器漏洞或错误配置，如 Apache HTTP 服务器中众所周知的 `server-status` 错误配置），此敏感信息可能被滥用。

事件日志通常包含对攻击者有用的数据（信息泄露）或可直接用于漏洞利用的数据：

- 调试信息
- 堆栈跟踪
- 用户名
- 系统组件名称
- 内部 IP 地址
- 较不敏感的个人数据（例如，与指定个人关联的电子邮件地址、邮政地址和电话号码）
- 业务数据

此外，在某些管辖区，将某些敏感信息（如个人数据）存储在日志文件中可能使企业有义务对日志文件应用他们将应用于后端数据库的数据保护法律。如果不知道这一点，即使是无心的，也可能根据适用的数据保护法律受到处罚。

更广泛的敏感信息列表：

- 应用源代码
- 会话标识值
- 访问令牌
- 敏感个人数据和某些形式的个人身份信息（PII）
- 认证密码
- 数据库连接字符串
- 加密密钥
- 银行账户或持卡人数据
- 高于日志系统允许存储的安全分类的数据
- 商业敏感信息
- 在相关管辖区非法收集的信息
- 用户选择不收集或不同意收集的信息，例如使用不跟踪，或同意收集已过期

#### 日志位置

通常，服务器将生成其操作和错误的本地日志，消耗服务器运行所在系统的磁盘。但是，如果服务器被泄露，其日志可以被入侵者清除以清理其攻击和方法的所有痕迹。如果发生这种情况，系统管理员将无法知道攻击如何发生或攻击源在哪里。实际上，大多数攻击者工具包都包含一个"日志清除器"，能够清除包含给定信息（如攻击者的 IP 地址）的任何日志，并通常在攻击者的系统级 root kit 中使用。

因此，明智的做法是将日志保存在单独的位置，而不是 Web 服务器本身。这也使得聚合来自不同来源（指同一应用，如 Web 服务器场）的日志更容易，并且使得日志分析（可能消耗大量 CPU）更容易执行，而不影响服务器本身。

#### 日志存储

日志存储不当可能引入拒绝服务条件。如果没有任何特别阻止，具有足够资源的任何攻击者可能产生大量请求来填充分配给日志文件的空间。但是，如果服务器配置不正确，日志文件将与用于操作系统软件或应用本身的磁盘存储在同一分区中。这意味着如果磁盘满，操作系统或应用可能因无法写入磁盘而失败。

在 UNIX 系统中，日志通常位于 /var（尽管某些服务器安装可能位于 /opt 或 /usr/local），重要的是确保存储日志的目录位于单独的分区。在某些情况下，为了防止系统日志受到影响，服务器软件本身的日志目录（如 Apache Web 服务器中的 /var/log/apache）应存储在专用分区中。

这并不是说应该允许日志增长以填满它们所在的文件系统。应监控服务器日志的增长以检测此状况，因为这可能表明正在发生攻击。

在生产环境中测试此状况可能有风险，可以通过发出足够且持续的请求数量来进行，看看这些请求是否被记录，以及是否有可能通过这些请求填满日志分区。在某些环境中，无论是否通过 GET 或 POST 请求产生，QUERY_STRING 参数都会被记录，因此可以模拟大型查询，因为通常单个请求只会记录少量数据，如日期和时间、源 IP 地址、URI 请求和服务器结果。

#### 日志轮换

大多数服务器（但很少有自定义应用）将轮换日志以防止它们填满所在的文件系统。在日志轮换期间的假设是，其中的信息仅在有限时间内必要。

应测试此功能以确保：

- 日志按照安全策略定义的时间保留，不多不少。
- 日志在轮换后被压缩（这是方便的，因为这意味着相同的可用磁盘空间将存储更多日志）。
- 轮换日志文件的文件系统权限应与日志文件本身的权限相同（或更严格）。例如，Web 服务器需要写入它们使用的日志，但它们实际上不需要写入轮换日志，这意味着文件权限可以在轮换时更改，以防止 Web 服务器进程修改这些文件。

某些服务器可能在日志达到给定大小时轮换日志。如果发生这种情况，必须确保攻击者不能强制日志轮换以隐藏其踪迹。

#### 日志访问控制

事件日志信息不应向最终用户可见。即使 Web 管理员也不应访问此类日志，因为它违反了职责分离控制。确保用于保护对原始日志访问的任何访问控制模式，以及用于查看或搜索日志的任何应用功能，不与其他应用用户角色的访问控制模式相关联。也不应向未经身份验证的用户显示任何日志数据。

#### 日志审查

审查日志不仅可用于提取 Web 服务器中文件的使用统计（这是大多数基于日志的应用关注的），还可用于确定是否正在发生对 Web 服务器的攻击。

为了分析 Web 服务器攻击，需要分析服务器的误差日志文件。审查应集中于：

- 40x（未找到）错误消息。来自同一来源的大量这些消息可能表明正在使用 CGI 扫描器工具对 Web 服务器进行扫描。
- 50x（服务器错误）消息。这些可能是攻击者滥用应用中意外失败部分的指示。例如，SQL 注入攻击的第一阶段将在 SQL 查询未正确构造且其执行在后端数据库上失败时产生这些错误消息。

不应在与生成日志的同一服务器上生成或存储日志统计或分析。否则，攻击者可能通过 Web 服务器漏洞或错误配置访问它们，并检索与日志文件本身可能泄露的相似信息。

## 参考资料

- Apache
    - Apache Security，Ivan Ristic，O'reilly，2005年3月。
    - Apache Security Secrets: Revealed (Again)，Mark Cox，2003年11月
    - Apache Security Secrets: Revealed，ApacheCon 2002，拉斯维加斯，Mark J Cox，2002年10月
    - [Performance Tuning](https://httpd.apache.org/docs/current/misc/perf-tuning.html)
- Lotus Domino
    - Lotus Security Handbook，William Tworek 等，2004年4月，IBM Redbooks 收藏中提供
    - Lotus Domino Security，X-force 白皮书，Internet Security Systems，2002年12月
    - Hackproofing Lotus Domino Web Server，David Litchfield，2001年10月
- Microsoft IIS
    - [Security Best Practices for IIS 8](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/jj635855(v=ws.11))
    - [CIS Microsoft IIS Benchmarks](https://www.cisecurity.org/benchmark/microsoft_iis/)
    - Securing Your Web Server (Patterns and Practices)，Microsoft Corporation，2004年1月
    - IIS Security and Programming Countermeasures，Jason Coombs
    - From Blueprint to Fortress: A Guide to Securing IIS 5.0，John Davis，Microsoft Corporation，2001年6月
    - Secure IIS 5 Checklist，Michael Howard，Microsoft Corporation，2000年6月
- Red Hat's（formerly Netscape's）iPlanet
    - Guide to the Secure Configuration and Administration of iPlanet Web Server, Enterprise Edition 4.1，James M Hayes，Systems and Network Attack Center (SNAC) 的网络应用团队，NSA，2001年1月
- WebSphere
    - IBM WebSphere V5.0 Security，WebSphere Handbook Series，Peter Kovari 等，IBM，2002年12月。
    - IBM WebSphere V4.0 Advanced Edition Security，Peter Kovari 等，IBM，2002年3月。
- 通用
    - [Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)，OWASP
    - [SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final) Guide to Computer Security Log Management，NIST
    - [PCI DSS v3.2.1](https://www.pcisecuritystandards.org/document_library) Requirement 10 and PA-DSS v3.2 Requirement 4，PCI Security Standards Council

- 通用：
    - [CERT Security Improvement Modules: Securing Public Web Servers](https://resources.sei.cmu.edu/asset_files/SecurityImprovementModule/2000_006_001_13637.pdf)
