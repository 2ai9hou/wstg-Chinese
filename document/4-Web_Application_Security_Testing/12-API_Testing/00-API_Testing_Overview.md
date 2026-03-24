# API测试概述

## Web API简介

Web应用程序编程接口（API）在网络或互联网上促进不同软件系统之间的通信和数据交换。Web API使不同应用程序能够以标准化和高效的方式相互交互，使它们能够利用彼此的功能和数据。

云计算、微服务架构和单页应用程序等不同技术的采用都促进了API作为架构运动的采用。

与任何新概念的引入一样，可能存在需要测试的缺陷和漏洞。否则，安全保护不当的API可能提供通往敏感数据的无限制直接路径。

本章试图指导安全研究人员了解测试API所需的必要概念。本节特别调查了不同的API技术及其历史。

## 使用哪种API技术？

在我们对正在测试的API类型做出假设之前，了解安全研究人员可能遇到的问题范围的全部内容可能会有所帮助。这些包括：

1. 表述性状态转移（REST）API
2. 简单对象访问协议（SOAP）API
3. GraphQL API
4. gRPC远程过程调用（gRPC）
5. WebSockets API

## REST（表述性状态转移）API

### 什么是REST？

REST是一组用于与Web资源交互的规则和约定。URI、HTTP方法、头和状态码的关键组件支持REST原则。

### 历史

由于其简单性、可扩展性和与现有Web基础设施的兼容性，基于REST的API在撰写本文时已成为互联网上最常见的API架构。REST API不是立即出现的，而是有从研究到采用的漫长道路。

1994年，HTTP规范的主要作者之一Roy Fielding开始了他在加州大学欧文分校攻读博士学位期间REST的工作。到2000年，他发表了论文[架构风格和网络软件架构设计](https://ics.uci.edu/~fielding/pubs/dissertation/top.htm)，其中他引入并定义了REST作为一种架构风格。REST被设计利用HTTP的现有特性，强调可扩展性、无状态交互和统一接口。

在2010年代，REST因其简单性和与Web底层架构的兼容性成为Web API的实际标准。REST API的广泛采用是由移动应用程序、云计算和微服务架构的增长推动的。Swagger/OpenAPI、RAML和API Blueprint等工具和框架的开发促进了REST API的设计、文档和测试。

到2020年代，现代发展为REST带来了GraphQL等技术。此外，OpenAPI/Swagger规范成为描述REST API的广泛采用标准，实现了更好的集成和自动化。

### 统一资源标识符

REST API使用统一资源标识符（URI）访问资源。URI是URI架构的关键元素。URI是唯一标识特定资源的字符序列。URI在互联网上广泛用于定位和交互与资源，如网页、文件和服务。

URI由几个组件组成，每个组件都有特定用途。[RFC3986](https://tools.ietf.org/html/rfc3986)中定义的通用URI语法如下：

> `URI = scheme "://" authority "/" path [ "?" query ] [ "#" fragment ]`

对于REST，**方案**通常是`HTTP`或`HTTPS`，但一般表示用于访问资源的协议或方法。其他常见方案包括`ftp`、`mailto`和`file`。

**权限**指定资源所在服务器的域名或IP地址，可能包括端口号。它还可以包括userinfo作为子组件。

**路径**指定资源在服务器上的特定位置。我们对URI的路径感兴趣，因为这是用户和资源之间的关系。例如，`https://api.example.com/admin/testing/report`可能显示测试报告。这是用户admin与其报告之间的关系。

任何URI的路径都将定义REST API资源模型。资源由正斜杠分隔，基于自顶向下设计。

例如：

- `https://api.example.com/admin/testing/report`
- `https://api.example.com/admin/testing/`
- `https://api.example.com/admin/`

**查询**为资源提供其他参数。它以`?`开头，由`&`分隔的键值对组成。

**片段**表示资源的特定部分，如网页内的部分。它以`#`开头。值得注意的是，片段标识符仅在客户端处理，不发送到服务器。

### HTTP方法

REST API使用标准HTTP方法对资源执行操作，遵循[RFC7231](https://tools.ietf.org/html/rfc7231)中定义的[HTTP请求方法](https://tools.ietf.org/html/rfc7231#section-4)。这些方法映射到CRUD，这是计算机科学中持久存储的四个基本功能。CRUD代表创建、读取、更新和删除，这是可以对数据执行的四个操作。

HTTP请求方法为：

| 方法 | 描述                                   |
| ---------| ---------------------------------------- |
| GET     | 获取资源状态的表示    |
| POST    | 创建新资源                         |
| PUT     | 更新资源                             |
| DELETE  | 删除资源                             |
| HEAD    | 获取与资源状态关联的元数据 |
| OPTIONS | 列出可用方法                        |

#### 头

REST依赖头来支持在请求或响应中传达附加信息。这些包括：

- `Content-Type`：指示资源的媒体类型（例如`application/json`）。
- `Authorization`：包含认证凭据（例如令牌）。
- `Accept`：指定响应可接受的媒体类型。

#### 状态码

符合REST原则的应用程序API使用HTTP响应消息的状态码通知客户端其请求的结果。

| 响应代码 | 响应消息      | 描述   |
|---------------|-----------------------|--------------------------------------------------------------------------------------------------------|
| 200           | OK                    | 成功处理客户端请求                                                                      |
| 201           | Created               | 新资源创建                                                                                   |
| 301           | Moved Permanently     | 永久重定向                                                                                  |
| 304           | Not Modified          | 缓存相关响应，当客户端具有与服务器相同的资源副本时返回 |
| 307           | Temporary Redirect    | 临时资源重定向                                                                      |
| 400           | Bad Request           | 客户端格式错误的请求                                                                |
| 401           | Unauthorized          | 客户端不允许发出请求或访问特定资源                                 |
| 403           | Forbidden             | 客户端被禁止访问资源                                                             |
| 404           | Not Found             | 资源不存在或基于请求不正确                                               |
| 405           | Method Not Allowed    | 使用了无效或未知方法                                                                  |
| 500           | Internal Server Error | 服务器由于内部错误无法处理请求                                          |

## 参考资料

1. [OWASP REST安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html)
2. [OWASP REST评估速查表](https://cheatsheetseries.owasp.org/cheatsheets/REST_Assessment_Cheat_Sheet.html)
3. [OWASP API安全项目](https://owasp.org/www-project-api-security/)
4. [OWASP API安全工具](https://owasp.org/www-community/api_security_tools)
