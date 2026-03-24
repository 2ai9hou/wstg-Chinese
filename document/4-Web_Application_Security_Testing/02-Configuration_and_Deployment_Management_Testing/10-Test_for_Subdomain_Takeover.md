# 测试子域名接管

|ID          |
|------------|
|WSTG-CONF-10|

## 概述

成功利用此类漏洞允许攻击者声明并控制受害者的子域名。此攻击依赖于以下条件：

1. 受害者的外部 DNS 服务器子域名记录配置为指向不存在的或非活动的资源/外部服务/端点。XaaS（一切即服务）产品和公共云服务的普及提供了许多可能的目标需要考虑。
2. 托管资源/外部服务/端点的服务提供商没有正确处理子域名所有权验证。

如果子域名接管成功，可能发生各种攻击（提供恶意内容、网络钓鱼、窃取用户会话 cookie、凭据等）。此漏洞可能针对各种 DNS 资源记录进行利用，包括：`A`、`CNAME`、`MX`、`NS`、`TXT` 等。就攻击严重性而言，`NS` 子域名接管（尽管可能性较小）影响最大，因为成功攻击可能导致对整个 DNS 区域和受害者域名的完全控制。

### GitHub

1. 受害者（victim.com）使用 GitHub 进行开发，并配置了 DNS 记录（`coderepo.victim.com`）来访问它。
2. 受害者决定将代码仓库从 GitHub 迁移到商业平台，但没有从 DNS 服务器中删除 `coderepo.victim.com`。
3. 攻击者发现 `coderepo.victim.com` 托管在 GitHub 上，并使用自己的 GitHub 账户通过 GitHub Pages 声明它。

### 过期域名

1. 受害者（victim.com）拥有另一个域名（victimotherdomain.com），并使用 CNAME 记录（www）引用另一个域名（`www.victim.com` --> `victimotherdomain.com`）
2. 在某个时候，victimotherdomain.com 过期，任何人都可以注册。由于 CNAME 记录没有从 victim.com DNS 区域中删除，任何注册 `victimotherdomain.com` 的人都可以完全控制 `www.victim.com`，直到 DNS 记录被删除或更新。

## 测试目标

- 枚举所有可能的域名（以前的和当前的）。
- 识别任何被遗忘或配置错误的域名。

## 如何测试

### 黑盒测试

第一步是枚举受害者 DNS 服务器和资源记录。有多种方法可以完成此任务；例如，使用常见子域名字典的 DNS 枚举、DNS 暴力破解或使用 Web 搜索引擎和其他 OSINT 数据源。

使用 dig 命令，测试人员查找以下值得进一步调查的 DNS 服务器响应消息：

- `NXDOMAIN`
- `SERVFAIL`
- `REFUSED`
- `no servers could be reached.`

#### 测试 DNS A、CNAME 记录子域名接管

使用 `dnsrecon` 对受害者域（`victim.com`）执行基本 DNS 枚举：

```bash
$ ./dnsrecon.py -d victim.com
[*] Performing General Enumeration of Domain: victim.com
...
[-] DNSSEC is not configured for victim.com
[*]      A subdomain.victim.com 192.30.252.153
[*]      CNAME subdomain1.victim.com fictioussubdomain.victim.com
...
```

识别哪些 DNS 资源记录已失效并指向不活跃/未使用的服务。使用 dig 命令测试 `CNAME` 记录：

```bash
$ dig CNAME fictioussubdomain.victim.com
; <<>> DiG 9.10.3-P4-Ubuntu <<>> ns victim.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 42950
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
```

以下 DNS 响应值得进一步调查：`NXDOMAIN`。

要测试 `A` 记录，测试人员执行 whois 数据库查询并识别 GitHub 作为服务提供商：

```bash
$ whois 192.30.252.153 | grep "OrgName"
OrgName: GitHub, Inc.
```

测试人员访问 `subdomain.victim.com` 或发出 HTTP GET 请求，该请求返回 "404 - File not found" 响应，这是漏洞的明确指示。

![GitHub 404 File Not Found response](images/subdomain_takeover_ex1.jpeg)\
*图 4.2.10-1：GitHub 404 File Not Found 响应*

测试人员使用 GitHub Pages 声明该域名：

![GitHub claim domain](images/subdomain_takeover_ex2.jpeg)\
*图 4.2.10-2：GitHub 声明域名*

#### 测试 NS 记录子域名接管

识别范围内域的所有 nameserver：

```bash
$ dig ns victim.com +short
ns1.victim.com
nameserver.expireddomain.com
```

在这个虚构的示例中，测试人员检查域 `expireddomain.com` 是否在域名注册商处处于活动状态。如果该域可以购买，则子域名存在漏洞。

以下 DNS 响应值得进一步调查：`SERVFAIL` 或 `REFUSED`。

### 灰盒测试

测试人员可以获得 DNS 区域文件，这意味着不需要 DNS 枚举。测试方法相同。

## 修复

为了减轻子域名接管的风险，应从 DNS 区域中删除有漏洞的 DNS 资源记录。建议持续监控和定期检查作为最佳实践。

## 工具

- [dig - man page](https://linux.die.net/man/1/dig)
- [recon-ng - Web 侦察框架](https://github.com/lanmaster53/recon-ng)
- [theHarvester - OSINT 情报收集工具](https://github.com/laramies/theHarvester)
- [Sublist3r - OSINT 子域名枚举工具](https://github.com/aboul3la/Sublist3r)
- [dnsrecon - DNS 枚举脚本](https://github.com/darkoperator/dnsrecon)
- [OWASP Amass DNS 枚举](https://github.com/OWASP/Amass)
- [OWASP Domain Protect](https://owasp.org/www-project-domain-protect)

## 参考资料

- [HackerOne - 子域名接管指南](https://www.hackerone.com/blog/Guide-Subdomain-Takeovers)
- [子域名接管：基础知识](https://0xpatrik.com/subdomain-takeover-basics/)
- [子域名接管：超越 CNAME](https://0xpatrik.com/subdomain-takeover-ns/)
- [can-i-take-over-xyz - 漏洞服务列表](https://github.com/EdOverflow/can-i-take-over-xyz/)
- [OWASP AppSec Europe 2017 - Frans Rosén：使用云提供商进行 DNS 劫持 – 无需验证](https://2017.appsec.eu/presos/Developer/DNS%20hijacking%20using%20cloud%20providers%20%E2%80%93%20no%20verification%20needed%20-%20Frans%20Rosen%20-%20OWASP_AppSec-Eu_2017.pdf)
