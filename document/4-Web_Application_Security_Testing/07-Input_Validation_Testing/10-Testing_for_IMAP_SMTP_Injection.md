# 测试IMAP SMTP注入

|ID          |
|------------|
|WSTG-INPV-10|

## 概述

此威胁影响所有与邮件服务器（IMAP/SMTP）通信的应用程序，通常是Webmail应用程序。此测试的目的是验证由于输入数据未正确清理而向邮件服务器注入任意IMAP/SMTP命令的能力。

IMAP/SMTP注入技术在邮件服务器不能直接从Internet访问时更有效。在可以与后端邮件服务器进行完整通信的情况下，建议进行直接测试。

IMAP/SMTP注入可以访问否则不能从Internet直接访问的邮件服务器。在某些情况下，这些内部系统没有与前端Web服务器相同的基础设施安全和强化级别。因此，邮件服务器结果可能更容易受到最终用户的攻击（请参阅图1中呈现的方案）。

![IMAP SMTP注入](images/Imap-smtp-injection.png)\
*图4.7.10-1：使用IMAP/SMTP注入技术与邮件服务器通信*

图1描述了使用Webmail技术时通常看到的流量。步骤1和2是用户与Webmail客户端交互，而步骤2是测试人员绕过Webmail客户端直接与后端邮件服务器交互。

此技术允许各种操作和攻击。可能性取决于注入的类型和范围以及正在测试的邮件服务器技术。

一些使用IMAP/SMTP注入技术的攻击示例：

- 利用IMAP/SMTP协议中的漏洞
- 规避应用程序限制
- 反自动化流程规避
- 信息泄露
- 中继/垃圾邮件

## 测试目标

- 识别IMAP/SMTP注入点。
- 了解系统的数据流和部署结构。
- 评估注入影响。

## 如何测试

### 识别易受攻击的参数

为了检测易受攻击的参数，测试人员必须分析应用程序处理输入的能力。输入验证测试要求测试人员向服务器发送虚假的或恶意的请求并分析响应。在安全应用程序中，响应应该是错误以及一些告诉客户端出现问题已发生的相应操作。在易受攻击的应用程序中，恶意请求可能被后端应用程序处理，后端应用程序将回复`HTTP 200 OK`响应消息。

重要的是，请求是所测试的技术匹配的。发送针对Microsoft SQL Server的SQL注入字符串，而使用MySQL服务器时，将导致误报响应。在这种情况下，发送恶意IMAP命令是 modus operandi，因为IMAP是正在测试的底层协议。

IMAP特殊参数应使用：

| 在IMAP服务器上     | 在SMTP服务器上 |
|------------------------|--------------------|
| 身份验证         | 发件人电子邮件     |
| 邮箱操作（列出、读取、创建、删除、重命名） | 目标电子邮件 |
| 消息操作（读取、复制、移动、删除） | 主题   |
| 断开连接          | 消息正文       |
|                        | 附件     |

在此示例中，通过操作参数的所有请求测试"邮箱"参数：

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46106&startMessage=1`

以下示例可以使用。

- 为参数分配空值：

`https://<webmail>/src/read_body.php?mailbox=&passed_id=46106&startMessage=1`

- 用随机值替换值：

`https://<webmail>/src/read_body.php?mailbox=NOTEXIST&passed_id=46106&startMessage=1`

- 向参数添加其他值：

`https://<webmail>/src/read_body.php?mailbox=INBOX PARAMETER2&passed_id=46106&startMessage=1`

- 添加非标准特殊字符（即：`\`、'`、`"`、`@`、`#`、`!`、`|`）：

`https://<webmail>/src/read_body.php?mailbox=INBOX"&passed_id=46106&startMessage=1`

- 消除参数：

`https://<webmail>/src/read_body.php?passed_id=46106&startMessage=1`

上述测试的最终结果为测试人员提供了三种可能的情况：
S1 - 应用程序返回错误代码/消息
S2 - 应用程序不返回错误代码/消息，但未实现请求的操作
S3 - 应用程序不返回错误代码/消息并正常实现请求的操作

情况S1和S2表示成功的IMAP/SMTP注入。

攻击者的目标是接收S1响应，因为它是应用程序容易受到注入和进一步操纵的指标。

让我们假设用户使用以下HTTP请求检索电子邮件头：

`https://<webmail>/src/view_header.php?mailbox=INBOX&passed_id=46105&passed_ent_id=0`

攻击者可以通过注入字符`"`（使用URL编码为`%22`）修改参数INBOX的值：

`https://<webmail>/src/view_header.php?mailbox=INBOX%22&passed_id=46105&passed_ent_id=0`

在这种情况下，应用程序响应可能为：

```txt
ERROR: Bad or malformed request.
Query: SELECT "INBOX""
Server responded: Unexpected extra arguments to Select
```

情况S2更难以成功测试。测试人员需要使用盲命令注入以确定服务器是否容易受到攻击。

另一方面，最后一种情况（S3）与本段无关。

> 易受攻击的参数列表
>
> - 受影响的函数
> - 可能注入的类型（IMAP/SMTP）

### 了解客户端的数据流和部署结构

在识别所有易受攻击的参数（例如`passed_id`）后，测试人员需要确定可以注入的程度，然后设计测试计划以进一步利用应用程序。

在此测试用例中，我们检测到应用程序的`passed_id`参数容易受到攻击，并用于以下请求：

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=46225&startMessage=1`

使用以下测试用例（当需要数值时提供字母值）：

`https://<webmail>/src/read_body.php?mailbox=INBOX&passed_id=test&startMessage=1`

将生成以下错误消息：

```txt
ERROR : Bad or malformed request.
Query: FETCH test:test BODY[HEADER]
Server responded: Error in IMAP command received by server.
```

在此示例中，返回的错误消息显示了所执行命令的名称和相应参数。

在其他情况下，错误消息（应用程序的`not controlled`）包含执行的命令的名称，但阅读适当的[RFC](#参考资料)允许测试人员了解与上述功能相关的其他可能的命令。

如果应用程序不返回描述性错误消息，测试人员需要分析受影响的功能以推断所有与上述功能相关的可能命令（和参数）。例如，如果在创建邮箱功能中检测到易受攻击的参数，则可以合理地假设受影响的IMAP命令是`CREATE`。根据RFC，`CREATE`命令接受一个参数，该参数指定要创建的邮箱的名称。

> 受影响的IMAP/SMTP命令列表
>
> - 受影响的IMAP/SMTP命令预期的类型、值和参数数量

### IMAP/SMTP命令注入

一旦测试人员识别出易受攻击的参数并分析了执行它们的上下文，下一阶段就是利用功能。

此阶段有两种可能的结果：

1. 可以在未认证状态下进行注入：受影响的功能不需要用户进行身份验证。可用的注入（IMAP）命令限于：`CAPABILITY`、`NOOP`、`AUTHENTICATE`、`LOGIN`和`LOGOUT`。
2. 只能在认证状态下进行注入：成功利用需要用户在测试继续之前完全认证。

在任何情况下，IMAP/SMTP注入的典型结构如下：

- 头：预期命令的结尾；
- 正文：新命令的注入；
- 页脚：预期命令的开始。

重要的是要记住，为了执行IMAP/SMTP命令，必须使用CRLF（`%0d%0a`）序列终止上一个命令。

让我们假设在[识别易受攻击的参数](#identifying-vulnerable-parameters)阶段，攻击者检测到以下请求中的参数`message_id`容易受到攻击：

`https://<webmail>/read_email.php?message_id=4791`

让我们还假设在阶段2（"了解客户端的数据流和部署结构"）执行的分析将命令和与此参数关联的参数识别为：

`FETCH 4791 BODY[HEADER]`

在这种情况下，IMAP注入结构为：

`https://<webmail>/read_email.php?message_id=4791 BODY[HEADER]%0d%0aV100 CAPABILITY%0d%0aV101 FETCH 4791`

这将生成以下命令：

```sql
???? FETCH 4791 BODY[HEADER]
V100 CAPABILITY
V101 FETCH 4791 BODY[HEADER]
```

其中：

```sql
Header = 4791 BODY[HEADER]
Body   = %0d%0aV100 CAPABILITY%0d%0a
Footer = V101 FETCH 4791
```

> 受影响的IMAP/SMTP命令列表
>
> - 任意IMAP/SMTP命令注入

## 参考资料

### 白皮书

- [RFC 0821"简单邮件传输协议"](https://tools.ietf.org/html/rfc821)
- [RFC 3501"Internet消息访问协议 - 版本4rev1"](https://tools.ietf.org/html/rfc3501)
