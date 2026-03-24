# 测试格式字符串注入

|ID          |
|------------|
|WSTG-INPV-13|

## 概述

格式字符串是一个空终止字符序列，还包含在运行时解释或转换的转换说明符。如果服务器端代码[将用户输入与格式字符串连接](https://www.netsparker.com/blog/web-security/string-concatenation-format-string-vulnerabilities/)，攻击者可以附加其他转换说明符以导致运行时错误、信息泄露或缓冲区溢出。

格式字符串漏洞最坏的情况发生在不检查参数且包含`%n`说明符（写入内存）的语言中。这些函数，如果被攻击者修改格式字符串利用，可能导致[信息泄露和代码执行](https://www.veracode.com/security/format-string)：

- C和C++ [printf](https://en.cppreference.com/w/c/io/fprintf)及类似方法fprintf、sprintf、snprintf
- Perl [printf](https://perldoc.perl.org/functions/printf.html)和sprintf

这些格式字符串函数不能写入内存，但攻击者仍可通过更改格式字符串来输出开发人员不打算发送的值，从而导致信息泄露：

- Python 2.6和2.7 [str.format](https://docs.python.org/2/library/string.html)和Python 3 unicode [str.format](https://docs.python.org/3/library/stdtypes.html#str.format)可以通过注入字符串来修改，这些字符串可以指向[内存中的其他变量](https://lucumr.pocoo.org/2016/12/29/careful-with-str-format/)

以下格式字符串函数如果攻击者添加转换说明符，可能导致运行时错误：

- Java [String.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/String.html#format%28java.util.Locale%2Cjava.lang.String%2Cjava.lang.Object...%29)和[PrintStream.format](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/io/PrintStream.html#format%2528java.util.Locale%252Cjava.lang.String%252Cjava.lang.Object...%2529)
- PHP [printf](https://www.php.net/manual/es/function.printf.php)

导致格式字符串漏洞的代码模式是包含未清理用户输入的字符串格式函数的调用。以下示例显示调试`printf`如何使程序容易受到攻击：

C中的示例：

```c
char *userName = /* input from user controlled field */;

printf("DEBUG Current user: ");
// Vulnerable debugging code
printf(userName);
```

Java中的示例：

```java
final String userName = /* input from user controlled field */;

System.out.printf("DEBUG Current user: ");
// Vulnerable code:
System.out.printf(userName);
```

在这个特殊示例中，如果攻击者将其`userName`设置为一个或多个转换说明符，将会产生不良行为。如果`userName`包含`%p%p%p%p%p`，C示例将[打印内存内容](https://www.defcon.org/images/defcon-18/dc-18-presentations/Haas/DEFCON-18-Haas-Adv-Format-String-Attacks.pdf)，如果字符串中有`%n`，则可能破坏内存内容。在Java示例中，包含任何需要输入的说明符（包括`%x`或`%s`）的`username`将导致程序崩溃，并显示`IllegalFormatException`。尽管示例仍可能受其他问题影响，但可以通过printf参数`printf("DEBUG Current user: %s", userName)`修复漏洞。

## 测试目标

- 评估将格式字符串转换说明符注入用户控制字段是否会导致应用程序产生不良行为。

## 如何测试

测试包括代码分析和向被测应用程序注入转换说明符作为用户输入。

### 静态分析

静态分析工具可以在代码或二进制文件中找到格式字符串漏洞。工具示例包括：

- C和C++：[Flawfinder](https://dwheeler.com/flawfinder/)
- Java：[FindSecurityBugs规则](https://find-sec-bugs.github.io/bugs.htm#FORMAT_STRING_MANIPULATION) FORMAT_STRING_MANIPULATION
- PHP：[phpsa](https://github.com/ovr/phpsa/blob/master/docs/05_Analyzers.md#function_string_formater)中的字符串格式化器分析器

### 手动代码审查

静态分析可能遗漏更复杂代码生成的格式字符串。要在代码库中手动查找漏洞，测试人员可以查找接受格式字符串并追溯以确保不受信任的输入无法更改格式字符串的所有调用。

### 转换说明符注入

测试人员可以在单元测试或完整系统测试级别通过在任何字符串输入中发送转换说明符来进行检查。使用系统中所有语言的转换说明符[模糊测试](https://owasp.org/www-community/Fuzzing)程序。请参阅[OWASP格式字符串攻击](https://owasp.org/www-community/attacks/Format_string_attack)页面以获取可能使用的输入。如果测试失败，程序将崩溃或显示意外输出。如果测试通过，发送转换说明符的尝试应被阻止，或者字符串应与任何其他有效输入一样通过系统。

以下小节中的示例具有以下形式的URL：

`https://vulnerable_host/userinfo?username=x`

- 用户控制的值是`x`（对于`username`参数）。

#### 手动注入

测试人员可以使用Web浏览器或其他Web API调试工具执行手动测试。浏览Web应用程序或网站，使查询包含转换说明符。请注意，如果通过URL发送，由于包含`%`和`{`等特殊字符，大多数转换说明符需要[编码](https://tools.ietf.org/html/rfc3986#section-2.1)。可以通过使用以下URL浏览来引入字符串`%s%s%s%n`：

`https://vulnerable_host/userinfo?username=%25s%25s%25s%25n`

如果网站容易受到攻击，浏览器或工具应收到错误，可能包括超时或HTTP返回代码500。

Java代码返回错误

`java.util.MissingFormatArgumentException: Format specifier '%s'`

根据C实现，进程可能完全崩溃，显示`Segmentation Fault`。

#### 工具辅助模糊测试

包括[wfuzz](https://github.com/xmendez/wfuzz)在内的模糊测试工具可以自动化注入测试。对于wfuzz，首先使用以下内容（此示例中为fuzz.txt）创建一个文本文件，每行一个输入：

fuzz.txt:

```text
alice
%s%s%s%n
%p%p%p%p%p
{event.__init__.__globals__[CONFIG][SECRET_KEY]}
```

`fuzz.txt`文件包含：

- 有效输入`alice`，以验证应用程序可以处理正常输入
- 两个带C类转换说明符的字符串
- 一个Python转换说明符，尝试读取全局变量

要将模糊测试输入文件发送到被测Web应用程序，使用以下命令：

`wfuzz -c -z file,fuzz.txt,urlencode https://vulnerable_host/userinfo?username=FUZZ`

在上述调用中，`urlencode`参数启用对字符串的适当转义，`FUZZ`（大写）告诉工具在哪里引入输入。

输出示例如下

```text
ID           Response   Lines    Word     Chars       Payload
==================================================================

000000002:   500        0 L      5 W      142 Ch      "%25s%25s%25s%25n"
000000003:   500        0 L      5 W      137 Ch      "%25p%25p%25p%25p%25p"
000000004:   200        0 L      1 W      48 Ch       "%7Bevent.__init__.__globals__%5BCONFIG%5D%5BSECRET_KEY%5D%7D"
000000001:   200        0 L      1 W      5 Ch        "alice"
```

上述结果验证了应用程序容易注入C类转换说明符`%s`和`%p`。
