# 测试工具资源

## 引言

本附录旨在提供 Web 应用测试常用工具的列表。它的目的不是成为完整的工具参考，在此处包含工具不应被视为 OWASP 对该工具的特定认可。

列表仅包含可免费下载和使用的工具（尽管它们可能有限制商业使用许可的许可证）。

## 通用 Web 测试

### Web 代理

- [ZAP](https://www.zaproxy.org)
    - Zed Attack Proxy（ZAP）是一款易于使用的集成渗透测试工具，用于发现 Web 应用中的漏洞。它专为具有广泛安全经验的人群设计，因此非常适合渗透测试新手开发人员和功能测试人员。
        - ZAP 提供自动扫描器以及一组工具，允许您手动发现安全漏洞。
- [Burp Suite 社区版](https://portswigger.net/burp/communitydownload)
    - Burp Suite 是用于安全测试的拦截代理。它允许拦截和修改双向通过的所有 HTTP(S) 流量，它可以处理自定义 TLS 证书和非代理感知客户端。
- [Telerik Fiddler](https://www.telerik.com/fiddler)
    - Fiddler 是一个拦截式 Web 代理，主要面向开发人员而非渗透测试人员，但仍提供有用的功能。它还直接挂钩到 Windows HTTP API，允许它拦截某些不允许设置自定义代理的软件的流量。

### Firefox 扩展

- [Firefox HTTP Header Live](https://addons.mozilla.org/en-US/firefox/addon/http-header-live)
    - 随时浏览时查看页面的 HTTP 头。
- [Firefox Multi-Account Containers](https://addons.mozilla.org/en-GB/firefox/addon/multi-account-containers/)
    - 创建多个容器，每个容器都有自己隔离的 cookie 和会话。用于测试不同用户之间的访问控制。
- [Firefox Tamper Data](https://addons.mozilla.org/en-US/firefox/addon/tamper-data-for-ff-quantum/)
    - 使用 Tamper Data 查看和修改 HTTP/HTTPS 头和 post 参数
- [Firefox Web Developer](https://addons.mozilla.org/en-US/firefox/addon/web-developer/)
    - Web Developer 扩展向浏览器添加各种 Web 开发人员工具。

### Chrome 扩展

- [Chrome Web Developer](https://chrome.google.com/webstore/detail/bfbameneiokkgbdmiekhjnmfkcnldhhm)
    - Web Developer 扩展向浏览器添加工具栏按钮，其中包含各种 Web 开发人员工具。这是 Chrome 版 Web Developer 扩展的官方移植版本。
- [Cookie Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
    - 一个强大且易于使用的浏览器扩展，允许您快速创建、编辑和删除当前标签页的 cookie。用于开发、测试或手动管理 cookie。

### 特定漏洞测试

#### 测试 SQL 注入

- [sqlmap](https://sqlmap.org)

#### 测试 TLS

- [OWASP O-Saft](https://owasp.org/www-project-o-saft/)
- [sslyze](https://github.com/nabla-c0d3/sslyze)
- [testssl.sh](https://github.com/drwetter/testssl.sh)
- [SSLScan](https://github.com/rbsec/sslscan)
- [SSLLabs](https://www.ssllabs.com/ssltest/)

#### 测试暴力攻击

##### 哈希破解器

- [John the Ripper](https://github.com/openwall/john)
- [hashcat](https://hashcat.net/hashcat/)

##### 远程暴力破解

- [ZAP](https://www.zaproxy.org)
- [Patator](https://github.com/lanjelot/patator)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Burp Suite 社区版（Intruder）](https://portswigger.net/burp/communitydownload)

#### 模糊测试器

- [Ffuf](https://github.com/ffuf/ffuf)
- [Wfuzz](https://github.com/xmendez/wfuzz)
- [Jdam](https://gitlab.com/michenriksen/jdam)

#### Google 黑客

- [Google Hacking 数据库](https://www.exploit-db.com/google-hacking-database/)

#### 慢速 HTTP

- [Slowloris](https://github.com/gkbrk/slowloris)
- [slowhttptest](https://github.com/shekyan/slowhttptest)

### 网站镜像

- [wget](https://www.gnu.org/software/wget/)
- [wget for windows](https://gnuwin32.sourceforge.net/packages/wget.htm)
- [cURL](https://curl.haxx.se)

### 内容发现

- [Gobuster](https://github.com/OJ/gobuster)
- [Waybackurls](https://github.com/tomnomnom/waybackurls)
    - Waybackurls 获取 Wayback Machine 中已知的给定域名的所有 URL，用于侦察。
    - **用法：**

```bash
waybackurls example.com
```

- [GAU（获取所有 URL）](https://github.com/lc/gau)
    - GAU 从多个公共档案馆收集 URL，包括 Wayback Machine 和 Common Crawl。
    - **用法：**

```bash
gau example.com
```

- [Unfurl](https://github.com/tomnomnom/unfurl)
    - Unfurl 从 URL 中提取子域、路径和参数，以便进行更深入的分析。
    - **用法：**

```bash
unfurl "https://example.com/page?query=123"
```

### 端口和服务发现

- [Nmap](https://nmap.org/)

## 漏洞扫描器

- [ZAP](https://www.zaproxy.org)
- [Nikto](https://cirt.net/Nikto2)
- [Nuclei](https://nuclei.projectdiscovery.io/)
- [SecOps Solution](https://secopsolution.com)

## 利用框架

- [Metasploit](https://github.com/rapid7/metasploit-framework)
- [BeEF](https://github.com/beefproject/beef/)

## Linux 发行版

- [Kali](https://www.kali.org)
- [Parrot](https://www.parrotsec.org)
- [Samurai](https://github.com/SamuraiWTF/samuraiwtf)
- [Santoku](https://sourceforge.net/projects/santoku/)
- [BlackArch](https://blackarch.org/downloads.html)

## 源代码分析器

- [Spotbugs](https://spotbugs.github.io)
- [Find Security Bugs](https://find-sec-bugs.github.io)
- [phpcs-security-audit](https://github.com/squizlabs/PHP_CodeSniffer)
- [PMD](https://pmd.github.io)
- [Microsoft 的 .NET 分析器](https://docs.microsoft.com/en-us/visualstudio/code-quality/install-net-analyzers)
- [SonarQube 社区版](https://www.sonarqube.org)

## 浏览器自动化工具

浏览器自动化工具用于验证 Web 应用的功能。一些采用脚本方法，通常利用单元测试框架来构建测试套件和测试用例。如果不是全部，大多数可以改编为执行安全特定测试以及功能测试。

### 开源工具

- [HtmlUnit](https://htmlunit.sourceforge.io) - HtmlUnit 是 Java 程序的无 GUI 浏览器。它对 HTML 文档建模，并提供 API 来调用页面、填写表单、点击链接以及与 JavaScript 和复杂 AJAX 库交互。它可以根据配置模拟 Chrome、Firefox 或 Edge，通常用于自动化测试或 Web 抓取。HtmlUnit 也可以通过 [htmlunit-driver](https://github.com/SeleniumHQ/htmlunit-driver) 用作 Selenium 兼容浏览器。最新的稳定版本是 4.21.0（`org.htmlunit:htmlunit:4.21.0`）。
    - GitHub 仓库：[HtmlUnit/htmlunit](https://github.com/HtmlUnit/htmlunit)
- [Selenium](https://www.selenium.dev)
    - 基于 JavaScript 的测试框架，跨平台并提供用于创建测试的 GUI。

## 安全测试工具比较在线资源

除了本附录中列出的各个工具外，从业人员通常需要帮助跨类别（如 SAST、DAST、SCA 和 API 安全）比较和评估安全测试工具的资源。

以下免费资源提供精心策划的比较和评估指导。

### AppSec Santa 工具比较

- [AppSec Santa](https://appsecsanta.com)
    - 精心策划的比较，涵盖 160 多种应用安全工具，类别包括 SAST、DAST、SCA、API 安全、容器安全等。

### 安全头分析器

- [SecurityHeaders](https://securityheaders.com)
    - 一个免费在线工具，分析 HTTP 响应头并突出显示缺失或配置错误的安全保护，如内容安全策略、HSTS 和 X-Frame-Options。

### ZAP（Zed Attack Proxy）文档

ZAP（Zed Attack Proxy）动态应用安全测试工具的官方文档和学习资源。

- [ZAP 文档](https://www.zaproxy.org/docs/)

### Nuclei 模板项目

一个大型开源漏洞检测模板仓库，用于自动化安全扫描。

- [Nuclei 模板](https://github.com/projectdiscovery/nuclei-templates)
