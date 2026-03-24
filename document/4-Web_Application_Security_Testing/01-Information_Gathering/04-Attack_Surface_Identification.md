# 攻击面识别

|ID          |
|------------|
|WSTG-INFO-04|

## 摘要

识别 Web 应用的攻击面涉及发现与目标基础设施相关的所有应用、域名、虚拟主机和外部暴露的服务。此过程超出识别托管应用的范畴，包括 DNS 枚举、子域名发现、虚拟主机分析、非标准端口以及数字证书和证书透明度日志的审查。

随着虚拟托管和共享基础设施的普及，IP 地址与 Web 服务器之间的传统一对一关系已在很大程度上消失。单个 IP 地址可能托管跨不同域名、环境或管理接口的多个应用。未能识别这些资产可能导致评估不完整和遗漏漏洞。

安全专业人员有时会获得一组 IP 地址作为测试目标。可以说，这种场景更类似于渗透测试类型的参与，但无论如何，预期是这样的任务将测试通过此目标可访问的所有 Web 应用。问题是给定的 IP 地址在端口 80 上托管 HTTP 服务，但如果测试人员通过指定 IP 地址访问它（这是他们所知道的全部），它会报告"此地址未配置 Web 服务器"或类似消息。但该系统可能"隐藏"多个与不相关符号（DNS）名称关联的 Web 应用。显然，测试人员是测试所有应用还是只测试他们知道的应用，会深度影响分析的范围。

有时，目标规范更丰富。测试人员可能获得 IP 地址及其对应符号名称的列表。然而，此列表可能传达部分信息，即它可能省略一些符号名称，而客户甚至可能不知道这一点（这更可能发生在大型组织中）。

影响评估范围的其他问题包括在非显而易见 URL（如 `https://www.example.com/some-strange-URL`）上发布的 Web 应用，这些 URL 在其他地方没有被引用。这可能由于错误（由于配置错误）或故意（例如，未公开的管理接口）而发生。

为解决这些问题，必须执行全面的攻击面识别过程。

## 测试目标

- 枚举范围内的所有 Web 应用。
- 识别与目标关联的 DNS 名称、域名和虚拟主机。
- 使用被动和主动 DNS 技术发现其他域名和子域名。
- 分析数字证书和证书透明度日志以获取其他主机名。

## 如何测试

Web 应用发现是旨在识别给定基础设施上的 Web 应用的过程。后者通常指定为一组 IP 地址（可能是网段），但可能包括一组 DNS 符号名称或两者的混合。此信息在评估执行之前提供，无论是经典式渗透测试还是应用重点评估。在这两种情况下，除非参与规则另有规定（例如，仅测试位于 URL `https://www.example.com/` 的应用），否则评估应努力成为范围最全面的，即应识别通过给定目标可访问的所有应用。以下示例检查了一些可用于实现此目标的技术。

> 以下一些技术适用于面向互联网的 Web 服务器，即 DNS 和基于反向 IP 的 Web 搜索服务以及搜索引擎的使用。示例使用私有 IP 地址（如 `192.168.1.100`），除非另有说明，这些地址代表 *通用* IP 地址，仅用于匿名目的。

有三个因素影响多少应用与给定 DNS 名称（或 IP 地址）相关：

1. **不同的基础 URL**

    Web 应用最明显的入口点是 `www.example.com`，即我们用这种简写表示起源于 `https://www.example.com/` 的 Web 应用（HTTPS 同样适用）。然而，尽管这是最常见的情况，但没有什么能强制应用从 `/` 开始。

    例如，同一符号名称可能与三个 Web 应用关联：`https://www.example.com/app1` `https://www.example.com/app2` `https://www.example.com/app3`

    在这种情况下，URL `https://www.example.com/` 不会与有意义的页面关联。三个应用将保持 **隐藏**，除非测试人员明确知道如何访问它们，即测试人员知道 *app1*、*app2* 或 *app3*。通常没有必要以这种方式发布 Web 应用，除非所有者不希望它们以标准方式访问，并准备通知用户其确切位置。这并不意味着这些应用是秘密的，只是它们的存在和位置没有明确广告。

2. **非标准端口**

    虽然 Web 应用通常位于端口 80（HTTP）和 443（HTTPS），但这些端口号没有什么是固定或强制的。事实上，Web 应用可能关联任意 TCP 端口，并且可以通过指定端口号来引用：`http[s]://www.example.com:port/`。例如，`https://www.example.com:20000/`。

3. **虚拟主机**

    DNS 允许单个 IP 地址关联一个或多个符号名称。例如，IP 地址 `192.168.1.100` 可能关联 DNS 名称 `www.example.com`、`helpdesk.example.com`、`webmail.example.com`。所有名称不必属于同一 DNS 域。这种一对多关系可以通过使用所谓的虚拟主机来反映服务不同内容。指定我们正在引用的虚拟主机的信息嵌入在 HTTP 1.1 [Host 头](https://datatracker.ietf.org/doc/html/rfc7230#section-5.4)中。

    除了明显的 `www.example.com` 之外，人们不会怀疑其他 Web 应用的存在，除非他们知道 `helpdesk.example.com` 和 `webmail.example.com`。

### 解决问题 1 的方法 - 非标准 URL

无法完全确定非标准命名 Web 应用的存在。由于非标准，没有管理命名约定的固定标准，但是有多种技术测试人员可以使用来获得一些额外的见解。

首先，如果 Web 服务器配置错误并允许目录浏览，则可能发现这些应用。漏洞扫描器可能在这方面提供帮助。

其次，这些应用可能由其他网页引用，并且有可能已被网络搜索引擎爬取和索引。如果测试人员怀疑在 `www.example.com` 上存在这些**隐藏**应用，他们可以使用 *site* 运算符搜索，并检查 `site: www.example.com` 查询的结果。在返回的 URL 中，可能有一个指向此类非显而易见应用的 URL。

另一个选项是探测可能是未发布应用候选的 URL。例如，Web 邮件前端可能通过 URL 如 `https://www.example.com/webmail`、`https://webmail.example.com/` 或 `https://mail.example.com/` 访问。管理接口同样如此，它们可能发布在隐藏 URL（如 Tomcat 管理接口），但在任何地方都没有引用。因此，进行一些字典风格搜索（或"智能猜测"）可能产生一些结果。漏洞扫描器可能在这方面提供帮助。

### 解决问题 2 的方法 - 非标准端口

检查非标准端口上 Web 应用的存在很容易。端口扫描器（如 Nmap）能够通过 `-sV` 选项执行服务识别，并将识别任意端口上的 http[s] 服务。需要的是对整个 64k TCP 端口地址空间进行全面扫描。

例如，以下命令将使用 TCP 连接扫描查找 IP `192.168.1.100` 上的所有开放端口，并尝试确定绑定到它们的服务（仅显示 *基本* 选项——Nmap 具有广泛的选项集，其讨论超出范围）：

`nmap –Pn –sT –sV –p0-65535 192.168.1.100`

检查输出并查找 HTTP 或 TLS 封装服务的指示就足够了（应探查以确认它们是 HTTPS）。例如，前一个命令的输出可能如下所示：

```bash
Interesting ports on 192.168.1.100:
(The 65527 ports scanned but not shown below are in state: closed)
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 3.5p1 (protocol 1.99)
80/tcp    open  http        Apache httpd 2.0.40 ((Red Hat Linux))
443/tcp   open  ssl         OpenSSL
901/tcp   open  http        Samba SWAT administration server
1241/tcp  open  ssl         Nessus security scanner
3690/tcp  open  unknown
8000/tcp  open  http-alt?
8080/tcp  open  http        Apache Tomcat/Coyote JSP engine 1.1
```

从这个示例可以看到：

- 端口 80 上运行着 Apache HTTP 服务器。
- 端口 443 上看起来有 HTTPS 服务器（但需要确认，例如，通过浏览器访问 `https://192.168.1.100`）。
- 端口 901 上有 Samba SWAT Web 接口。
- 端口 1241 上的服务不是 HTTPS，而是 TLS 封装的 Nessus 守护进程。
- 端口 3690 具有未指定的服务（Nmap 给出其*指纹*——此处为清晰起见省略——以及将其提交到 Nmap 指纹数据库的说明，前提是您知道它代表什么服务）。
- 端口 8000 上有另一个未指定的服务；这可能是 HTTP，因为在 此端口上找到 HTTP 服务器并不罕见。让我们检查这个问题：

```bash
$ telnet 192.168.10.100 8000
Trying 192.168.1.100...
Connected to 192.168.1.100.
Escape character is '^]'.
GET / HTTP/1.0

HTTP/1.0 200 OK
pragma: no-cache
Content-Type: text/html
Server: MX4J-HTTPD/1.0
expires: now
Cache-Control: no-cache

<html>
...
```

这确认了它实际上是一个 HTTP 服务器。或者，测试人员可以使用 Web 浏览器访问 URL；或使用 GET 或 HEAD Perl 命令，这些命令模拟上述 HTTP 交互（但是 HEAD 请求可能不被所有服务器尊重）。

- 端口 8080 上运行着 Apache Tomcat。

同样的任务可以由漏洞扫描器执行，但首先检查所选扫描器能够识别非标准端口上运行的 HTTP[S] 服务。例如，Nessus 能够识别任意端口上的服务（前提是它被指示扫描所有端口），并且将与 Nmap 相比提供大量针对已知 Web 服务器漏洞的测试，以及 HTTPS 服务的 TLS/SSL 配置。如前所述，Nessus 还能够发现可能被忽视的流行应用或 Web 接口（例如，Tomcat 管理接口）。

### 解决问题 3 的方法 - 虚拟主机

有多种技术可用于识别与给定 IP 地址 `x.y.z.t` 关联的 DNS 名称。

#### DNS 枚举

DNS 枚举旨在识别与目标组织关联的域名、子域名和相关 DNS 记录，以扩大评估范围。DNS 枚举在识别映射到同一 IP 地址的其他虚拟主机方面发挥关键作用。这可能揭示开发系统、预发布环境、旧服务或管理接口。

可以使用被动和主动技术。

#### 被动 DNS 枚举

被动技术不直接与目标基础设施交互，而是依赖公开可用的数据源。示例包括：

- 公开 DNS 记录（A、AAAA、MX、TXT、NS）
- 反向 DNS 查找（PTR 记录）
- 搜索引擎
- 被动 DNS 数据库
- 证书透明度日志

在早期侦察阶段首选被动技术以避免检测。

#### 主动 DNS 枚举

主动技术直接查询目标的 DNS 基础设施，可能会在目标系统上生成日志。这些包括：

- 子域名暴力破解
- DNS 区域传输尝试
- 使用工具进行 DNS 记录枚举

用于 DNS 枚举的常用工具包括：

- `amass`
- `subfinder`
- `dnsrecon`
- `fierce`
- `dig`
- `nslookup`

使用 `dig` 的示例：`dig example.com ANY`

#### DNS 区域传输

考虑到区域传输在很大程度上不被 DNS 服务器尊重的事实，这种技术现今用途有限。然而，它可能仍然值得尝试。首先，测试人员必须确定为 `x.y.z.t` 提供服务的名称服务器。如果 `x.y.z.t` 已知符号名称（让它成为 `www.example.com`），可以通过 `nslookup`、`host` 或 `dig` 等工具请求 DNS NS 记录来确定其名称服务器。

如果 `x.y.z.t` 不知道符号名称，但目标定义至少包含一个符号名称，测试人员可以尝试应用相同的过程并查询该名称的名称服务器（希望 `x.y.z.t` 也由该名称服务器提供服务）。例如，如果目标由 IP 地址 `x.y.z.t` 和名称 `mail.example.com` 组成，则确定域 `example.com` 的名称服务器。

以下示例显示如何使用 `host` 命令识别 `www.owasp.org` 的名称服务器：

```bash
$ host -t ns www.owasp.org
www.owasp.org is an alias for owasp.org.
owasp.org name server ns1.secure.net.
owasp.org name server ns2.secure.net.
```

现在可以向域 `example.com` 的名称服务器请求区域传输。如果测试人员幸运，他们可能会收到此域的 DNS 条目列表作为响应。这将包括明显的 `www.example.com` 和不那么明显的 `helpdesk.example.com` 和 `webmail.example.com`（可能还有其他）。检查区域传输返回的所有名称，并考虑与所评估目标相关的所有名称。

尝试从其名称服务器之一请求 `owasp.org` 的区域传输：

```bash
$ host -l www.owasp.org ns1.secure.net
Using domain server:
Name: ns1.secure.net
Address: 192.220.124.10#53
Aliases:

Host www.owasp.org not found: 5(REFUSED)
; Transfer failed.
```

#### DNS 反向查询

此过程与前一个类似，但依赖反向（PTR）DNS 记录。不请求区域传输，而是将记录类型设置为 PTR 并对给定 IP 地址发出查询。如果测试人员幸运，他们可能会收到 DNS 名称条目作为响应。此技术依赖于 IP 到符号名称映射的存在，但这不能保证。

#### 基于 Web 的 DNS 搜索

这种搜索类似于 DNS 区域传输，但依赖基于 Web 的服务来执行 DNS 的基于名称的搜索。其中一种服务是 [Netcraft Search DNS](https://searchdns.netcraft.com/?host) 服务。测试人员可以查询属于您所选域的名称列表，如 `example.com`。然后他们将检查获得的名称是否与所检查的目标相关。

#### 反向 IP 服务

反向 IP 服务类似于 DNS 反向查询，不同之处在于测试人员查询基于 Web 的应用而不是名称服务器。有多种此类服务可用。由于它们往往返回部分（且经常不同）结果，因此最好使用多种服务以获得更全面的分析。

- [MxToolbox Reverse IP](https://mxtoolbox.com/ReverseLookup.aspx)
- [DNSstuff](https://www.dnsstuff.com/)（多种服务可用）
- [Net Square](https://web.archive.org/web/20190515092354/https://www.net-square.com/mspawn.html)（对域和 IP 地址的多种查询，需要安装）

#### Google 搜索

继从前述技术收集信息之后，测试人员可以依靠搜索引擎来可能细化和增加他们的分析。这可能产生属于目标的额外符号名称的证据，或通过非显而易见 URL 访问的应用。

例如，考虑到前面关于 `www.owasp.org` 的示例，测试人员可以查询 Google 和其他搜索引擎，查找与新发现的域 `webgoat.org`、`webscarab.com` 和 `webscarab.net` 相关的信息（因此是 DNS 名称）。

Google 搜索技术在 [测试：蜘蛛、机器人和爬虫](01-Conduct_Search_Engine_Discovery_Reconnaissance_for_Information_Leakage.md) 中有解释。

#### 数字证书

如果服务器通过 HTTPS 接受连接，则证书上的公用名（CN）和主题备用名（SAN）可能包含一个或多个主机名。但是，如果 Web 服务器没有可信证书，或使用通配符，这可能不会返回任何有效信息。

可以通过手动检查证书获取 CN 和 SAN，或通过 OpenSSL 等其他工具：

```sh
openssl s_client -connect 93.184.216.34:443 </dev/null 2>/dev/null | openssl x509 -noout -text | grep -E 'DNS:|Subject:'

Subject: C = US, ST = California, L = Los Angeles, O = Internet Corporation for Assigned Names and Numbers, CN = www.example.org
DNS:www.example.org, DNS:example.com, DNS:example.edu, DNS:example.net, DNS:example.org, DNS:www.example.com, DNS:www.example.edu, DNS:www.example.net
```

#### 证书透明度日志

证书透明度（CT）日志是公开可访问的 TLS 证书记录。这些日志可以搜索以识别与目标组织关联的主机名和子域名，包括预发布系统、管理接口、旧系统或其他外部可访问服务。

审查 CT 日志可能揭示不能单独通过 DNS 区域传输、反向查找或搜索引擎查询直接发现的主机名。测试人员应提取发现的主机名并通过 DNS 解析验证它们，以确定它们是否活跃且在评估定义的范围内。

审查 CT 日志数据时，考虑：

- 指示开发、预发布或测试环境的主机名。
- 管理或管理接口。
- 可能仍可访问的已弃用或旧系统。
- 可能暗示额外未发现子域名的通配符证书。

在进一步测试之前，应验证从 CT 日志收集的信息以确认所有权和相关性。

必须注意尊重参与中定义的范围限制。

在进一步测试活动之前，应验证并记录发现的资产。

查询 CT 日志的一种常见方法是使用聚合证书数据的公开可用搜索门户。例如，测试人员可以搜索颁发给 `example.com` 的证书并审查列出的子域名。

例如：`https://crt.sh/?q=%25.example.com`

![CT 日志搜索示例](images/Figure-4.1.4-CT-logs-example.png)

*图 4.1.4-1：证书透明度日志搜索结果示例。*

结果可能列出如 `dev.example.com`、`staging.example.com` 或其他未从主站点直接引用的主机名。应在进一步测试之前通过 DNS 解析验证发现的主机名。

## 工具

- DNS 查找工具如 `nslookup`、`dig` 和 `host`
- 子域名枚举工具如 `amass`、`subfinder`、`dnsrecon` 和 `fierce`
- 搜索引擎（Google、Bing 和其他主要搜索引擎）
- 反向 IP 查找服务
- [Nmap](https://nmap.org/)
- [Nessus 漏洞扫描器](https://www.tenable.com/products/nessus)
- [Nikto](https://github.com/sullo/nikto)
