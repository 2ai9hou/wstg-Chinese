# 业务逻辑简介

在多功能动态Web应用程序中测试业务逻辑缺陷需要以非常规方法思考。如果应用程序的认证机制被设计为按特定顺序执行步骤1、2、3来认证用户。如果用户直接从步骤1转到步骤3会怎样？在这个简单的例子中，应用程序是通过开放访问来提供访问；拒绝访问，还是只是用500消息出错？

可以举出很多例子，但不变的一课是"跳出常规思维"。这种漏洞无法被漏洞扫描器检测到，需要渗透测试人员的技能和创造力。此外，这种漏洞通常是最难检测的，通常是特定于应用程序的，但同时，如果被利用，它通常是对应用程序最具破坏性的。

业务逻辑缺陷的分类一直没有得到充分研究；尽管在现实世界的系统中经常发生业务缺陷的利用，许多应用安全研究人员也在研究它们。重点主要在Web应用程序上。社区内就这些问题是否代表特别新的概念，或者它们是否是众所周知的原理的变体存在争论。

业务逻辑缺陷的测试类似于功能测试人员使用的测试类型，专注于逻辑或有限状态测试。这些类型的测试要求安全专业人员以不同的方式思考，开发滥用和误用案例，并使用功能测试人员采用的许多测试技术。业务逻辑滥用案例的自动化是不可能的，仍然是一种依赖于测试人员及其对完整业务流程和规则的知识的手工艺术。

## 业务限制和限制

考虑应用程序提供的业务功能的规则。人们的行 为是否有任何限制或限制？然后考虑应用程序是否强制执行这些规则。如果您熟悉业务，验证应用程序的测试和分析用例通常很容易。如果您是第三方测试人员，那么您将不得不运用常识，或者询问企业是否应允许不同的操作。

有时，在非常复杂的应用程序中，测试人员最初不会完全理解应用程序的每个方面。在这些情况下，最好让客户引导测试人员浏览应用程序，以便在实际测试开始之前更好地了解应用程序的限制和预期功能。此外，在测试期间与开发人员保持直接联系（如可能）将极大地帮助回答有关应用程序功能的任何问题。

## 逻辑测试的挑战

自动化工具很难理解上下文，因此由人来执行这些测试。以下两个示例将说明理解应用程序的功能、开发人员的意图和一些创造性的"跳出框框"思维如何能够破坏应用程序的逻辑。第一个示例从简单的参数操作开始，而第二个是导致完全颠覆应用程序的多步骤过程的真实示例。

**示例1**：

假设一个电子商务网站允许用户选择要购买的商品，查看摘要页面，然后付款。如果攻击者能够返回摘要页面，保持其相同的有效会话，并注入较低的商品价格以完成交易，然后结账，会发生什么？

**示例2**：

在线购买时持有/锁定资源并阻止他人购买这些物品可能导致攻击者以较低价格购买商品。对此问题的对策是实施超时和确保只能收取正确价格的机制。

**示例3**：

如果用户能够启动与其俱乐部/会员账户关联的交易，然后在积分添加到账户后取消交易，会发生什么？积分/信用是否仍会应用到他们的账户？

## 工具

虽然有工具用于测试和验证业务流程在有效情况下正常运作，但这些工具无法检测逻辑漏洞。例如，工具无法检测用户是否能够通过编辑参数、预测资源名称或升级权限来访问受限资源来绕过业务流程流，也没有机制帮助人工测试人员怀疑这种状态。

以下是一些可有助于识别业务逻辑问题的常见工具类型。

安装附加组件时，您应始终考虑它们请求的权限以及您的浏览器使用习惯。

### 拦截代理

观察HTTP流量请求和响应块

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
- [Burp Proxy](https://portswigger.net/burp)

### Web浏览器插件

查看和修改HTTP/HTTPS头、发布参数，并观察浏览器DOM

- [Tamper Data for FF Quantum](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum)
- [Tamper Chrome (for Google Chrome)](https://chrome.google.com/webstore/detail/tamper-chrome-extension/hifhgpdkfodlpnlmlnmhchnkepplebkb)

## 其他测试工具

- [Web Developer toolbar](https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - Web Developer扩展向浏览器添加工具栏按钮，包含各种Web开发人员工具。这是Firefox的Web Developer扩展的官方移植版本。
- [Chrome的HTTP Request Maker](https://chrome.google.com/webstore/detail/kajfghlhfkcocafkcjlajldicbikpgnp)
- [Firefox的HTTP Request Maker](https://addons.mozilla.org/en-US/firefox/addon/http-request-maker)
    - Request Maker是一个渗透测试工具。借助它，您可以轻松捕获网页发出的请求，篡改URL、头和POST数据，当然，也可以发出新请求
- [Chrome的Cookie Editor](https://chrome.google.com/webstore/detail/fngmhnnpilhplaeedifhccceomclgfbg)
- [Firefox的Cookie Editor](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor)
    - Cookie Editor是一个Cookie管理器。您可以添加、删除、编辑、搜索、保护和阻止Cookie

## 参考资料

### 白皮书

- [通用误用评分系统(CMSS)：软件功能误用漏洞的度量 - NISTIR 7864](https://csrc.nist.gov/publications/detail/nistir/7864/final)
- [图形用户界面的有限状态测试，Fevzi Belli](https://pdfs.semanticscholar.org/b57c/6c8022abfd2cb17ec785d3622027b3edfaaf.pdf)
- [测试有限状态机的原理和方法 - 综述，David Lee, Mihalis Yannakakis](https://ieeexplore.ieee.org/document/533956)
- [在线游戏中的安全问题，Jianxin Jeff Yan and Hyun-Jin Choi](https://www.researchgate.net/publication/220677013_Security_issues_in_online_games)
- [保护虚拟世界免受真实攻击，Dr. Igor Muttik, McAfee](https://www.info-point-security.com/open_downloads/2008/McAfee_wp_online_gaming_0808.pdf)
- [七个使您的网站处于风险中的业务逻辑缺陷 – Jeremiah Grossman创始人兼首席技术官，WhiteHat Security](https://www.slideshare.net/jeremiahgrossman/seven-business-logic-flaws-that-put-your-website-at-risk-harvard-07062008)
- [面向Web应用程序逻辑漏洞自动检测 - Viktoria Felmetsger Ludovico Cavedon Christopher Kruegel Giovanni Vigna](https://www.usenix.org/legacy/event/sec10/tech/full_papers/Felmetsger.pdf)

### OWASP相关内容

- [如何防止Web应用程序中的业务缺陷漏洞，Marco Morana](https://www.slideshare.net/slideshow/issa-louisville-2010morana/5391600)

### 有用的网站

- [业务逻辑](https://en.wikipedia.org/wiki/Business_logic)
- [业务逻辑缺陷和Yahoo游戏](https://blog.jeremiahgrossman.com/2006/12/business-logic-flaws.html)
- [CWE-840：业务逻辑错误](https://cwe.mitre.org/data/definitions/840.html)
- [挑战逻辑：测试应用程序逻辑的复杂系统的理论、设计和实现](https://pdfs.semanticscholar.org/d14a/18f08f6488f903f2f691a1d159e95d8ee04f.pdf)

### 书籍

- [决策模型：链接业务和技术的业务逻辑框架，Barbara Von Halle, Larry Goldberg，CRC Press出版，ISBN1420082817 (2010)](https://isbnsearch.org/isbn/1420082817)
