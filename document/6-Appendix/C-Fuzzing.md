# 模糊测试

## 引言

模糊测试是向目标站点在一定时间间隔内发送大量请求的过程或技术。换句话说，它也类似于暴力破解。模糊测试是可以使用Wfuzz、ffuf等工具实现的过程。作为测试人员，您需要向工具提供目标URL、参数、端点等以及某种输入。然后模糊测试工具会制作请求并将其发送到目标。模糊测试完成后，需要分析响应、时间、状态码和其他特征以发现潜在漏洞。

## 为什么需要模糊测试？

通过手动逐个输入来测试漏洞可能很混乱。在当今这个人们时间少、耐心低的时代，手动输入来发现漏洞/安全问题的想法可能令人难以承受。为了减少这种感觉并节省时间，模糊测试可能是一个很大的优势。模糊测试是一个自动化过程，大部分艰苦工作由模糊测试工具处理。分析师所要做的就是在过程完成后分析各种特征。考虑一个有许多输入字段需要测试XSS的站点。在手动方法中，我们所做的是将XSS载荷逐个输入到输入字段，这将非常繁忙。相比之下，对于自动化方法，您只需要将XSS载荷列表提供给模糊测试工具，所有请求都由模糊测试工具处理。

## 用于模糊测试的工具

业界有数百种工具可用于进行模糊测试。但下面列出了一些评价最高、最流行的模糊测试工具。

### Wfuzz

[Wfuzz](https://github.com/xmendez/wfuzz) 通过将字典值替换到占位符 `FUZZ` 所在的位置来工作。为了更清楚地理解这一点，让我们看一个例子：

```bash
wfuzz -w userIDs.txt https://example.com/view_photo?userId=FUZZ
```

在上述命令中，`userIds.txt` 是一个包含数字ID值的字典文件。这里，我们告诉 wfuzz 对示例 URL 的请求进行模糊测试。请注意 URL 中的 `FUZZ` 单词，它将作为占位符，供 wfuzz 用字典中的值替换。`userIDs.txt` 文件中的所有数字 ID 值都将被插入替换 `FUZZ` 关键字。

### Ffuf

[Ffuf](https://github.com/ffuf/ffuf) 是一个用 Go 语言编写的 Web 模糊测试工具，它非常快速且本质上是递归的。它的工作方式与 Wfuzz 类似，但它是递归的。Ffuf 也是通过用字典值替换占位符 `FUZZ` 来工作的。例如：

```bash
ffuf -w userIDs.txt -u https://example.com/view_photo?userId=FUZZ
```

这里的 `-w` 是字典的标志，`-u` 是目标 URL 的标志。其余的工作机制与 Wfuzz 相同。它用 `userIDs.txt` 中的值替换 `FUZZ` 占位符。

### GoBuster

[GoBuster](https://github.com/OJ/gobuster) 是另一个用 Go 语言编写的模糊测试工具，最常用于模糊测试 URI、目录/路径、DNS 子域、AWS S3 桶、vhost 名称，并支持并发。例如：

```bash
gobuster dir -w endpoints.txt -u https://example.com
```

在上述命令中，`dir` 指定我们正在模糊测试目录，`-u` 是 URL 的标志，`-w` 是字典的标志，其中 `endpoints.txt` 是将从中获取载荷的字典文件。该命令向端点发送并发请求以发现可用目录。

### ZAP

[ZAP](https://www.zaproxy.org) 是一个 Web 应用安全扫描器，可用于发现 Web 应用中的漏洞和弱点。它还包括一个[模糊测试器](https://www.zaproxy.org/docs/desktop/addons/fuzzer/)。

ZAP 的关键特性之一是它能够执行被动扫描和主动扫描。被动扫描涉及观察用户与 Web 应用之间的流量，而主动扫描涉及向 Web 应用发送测试载荷以识别漏洞。

### 字典和参考资料

在上面的例子中，我们看到了为什么需要字典。只是字典是不够的，字典必须非常适合您的模糊测试场景。如果您找不到匹配必要场景的字典，请考虑生成您自己的字典。以下是一些流行的字典和参考资料。

- [跨站脚本（XSS）备忘单](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)
- [AwesomeXSS](https://github.com/s0md3v/AwesomeXSS)
- [Payloads All The Things](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [Big List of Naughty Strings](https://github.com/minimaxir/big-list-of-naughty-strings)
- [Bo0oM Fuzz List](https://github.com/Bo0oM/fuzz.txt)
- [FuzzDB](https://github.com/fuzzdb-project/fuzzdb)
- [bl4de Dictionaries](https://github.com/bl4de/dictionaries)
- [Open Redirect Payloads](https://github.com/cujanovic/Open-Redirect-Payloads)
- [EdOverflow Bug Bounty Cheat Sheet](https://github.com/EdOverflow/bugbounty-cheatsheet)
- [Daniel Miessler - SecLists](https://github.com/danielmiessler/SecLists)
- [XssPayloads Twitter Feed](https://twitter.com/XssPayloads)
- [XssPayloads List](https://github.com/payloadbox/xss-payload-list)
