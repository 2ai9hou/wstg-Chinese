# 测试绕过授权架构

|ID          |
|------------|
|WSTG-ATHZ-02|

## 概述

本测试用例重点识别授权弱点，在这些弱点中，已认证的用户能够访问超出其分配权限的资源或执行超出其权限的操作，包括水平权限提升和垂直权限提升。

虽然某些检查可能包括直接访问受保护资源而无需活动会话或注销后的场景，但本测试的主要目的是验证授权控制是否针对已认证用户和角色正确执行。

详细的未认证和认证后场景在"如何测试"部分中介绍。

## 测试目标

- 评估是否可能进行未认证访问、水平访问或垂直访问。

## 如何测试

- 在未登录的情况下访问资源并执行操作。- 直接页面请求（[强制浏览](https://owasp.org/www-community/attacks/Forced_browsing)）
- 水平访问资源并执行操作。
- 垂直访问资源并执行操作。

### 测试基本未认证访问

#### 手动使用浏览器

当 Web 应用程序未正确执行访问控制机制时，敏感资源会暴露，允许未认证用户查看。例如，如果用户通过强制浏览直接请求不同的页面，该页面可能在授予访问权限之前不检查匿名用户的授权。尝试通过浏览器地址栏直接访问受保护页面来使用此方法进行测试。

![直接请求受保护页面](images/Basm-directreq.jpg)\
*图 4.5.2-1：直接请求受保护页面*

#### 使用自动化工具

如果您有所有端点的列表，可以使用 ffuf、gobuster、ZAP 和 Burp Suite Intruder 等工具实现此过程自动化。

对于 ZAP，使用 [Access Control Testing](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/) 插件允许测试人员确定应用程序的哪些部分可供匿名用户访问，并识别潜在的访问控制问题。

对于 Burp Suite，Intruder 等内置工具以及许多插件（包括 Autorize）可帮助测试人员自动化授权测试。

### 测试水平绕过授权架构

对于应用程序执行的每个功能、特定角色或请求，需要验证：

- 是否可以访问应该可供持有不同身份但具有相同角色或权限的用户访问的资源？
- 是否可以操作应该可供持有不同身份的用户访问的资源上的功能？

对于每个角色：

1. 注册或生成两个具有相同权限的用户。
2. 保持两个不同的会话处于活动状态（每个用户一个）。
3. 对于每个请求，更改相关参数和从令牌一到令牌二的会话标识符，并诊断每个令牌的响应。
4. 如果响应相同、包含相同的私人数据或表明在其他用户的资源或数据上成功操作，则该应用程序被认为是存在漏洞的。

例如，假设 `viewSettings` 功能是应用程序每个账户菜单的一部分，具有相同的角色，并且可以通过请求以下 URL 访问：`https://www.example.com/account/viewSettings`。然后，在调用 `viewSettings` 功能时生成以下 HTTP 请求：

```http
POST /account/viewSettings HTTP/1.1
Host: www.example.com
[其他 HTTP 头]
Cookie: SessionID=USER_SESSION

username=example_user
```

有效且合法的响应：

```html
HTTP1.1 200 OK
[其他 HTTP 头]

{
  "username": "example_user",
  "email": "example@email.com",
  "address": "Example Address"
}
```

攻击者可能尝试使用相同的 `username` 参数执行该请求：

```html
POST /account/viewCCpincode HTTP/1.1
Host: www.example.com
[其他 HTTP 头]
Cookie: SessionID=ATTACKER_SESSION

username=example_user
```

如果攻击者的响应包含 `example_user` 的数据，则该应用程序容易受到横向移动攻击，用户可以在其中读取或写入其他用户的数据。

### 测试对管理功能的访问

例如，假设 `addUser` 功能是应用程序管理菜单的一部分，并且可以通过请求以下 URL 访问 `https://www.example.com/admin/addUser`。

然后，在调用 `addUser` 功能时生成以下 HTTP 请求：

```http
POST /admin/addUser HTTP/1.1
Host: www.example.com
[...]

userID=fakeuser&role=3&group=grp001
```

进一步的疑问或考虑将沿以下方向：

- 如果非管理员用户尝试执行该请求，会发生什么？
- 用户会被创建吗？
- 如果是这样，新用户可以使用他们的权限吗？

### 测试对分配给不同角色的资源的访问

各种应用程序根据用户角色设置资源控制。让我们以简历（CV）上传到 S3 存储桶为例。

作为普通用户，尝试访问这些文件的位置。如果您能够检索、修改或删除它们，则该应用程序存在漏洞。

### 测试特殊请求头处理

一些应用程序支持非标准头，如 `X-Original-URL` 或 `X-Rewrite-URL`，以便允许使用头值中指定的 URL 覆盖请求中的目标 URL。

当应用程序位于根据请求 URL 应用访问控制限制的组件后面时，可以利用这种行为。

基于请求 URL 的访问控制限制的示例可能是阻止从互联网访问在 `/console` 或 `/admin` 上暴露的管理控制台。

要检测对 `X-Original-URL` 或 `X-Rewrite-URL` 头的支持，可以应用以下步骤。

#### 1. 发送没有任何 X-Original-Url 或 X-Rewrite-Url 头的正常请求

```http
GET / HTTP/1.1
Host: www.example.com
[...]
```

#### 2. 发送带有指向不存在资源的 X-Original-Url 头的请求

```html
GET / HTTP/1.1
Host: www.example.com
X-Original-URL: /donotexist1
[...]
```

#### 3. 发送带有指向不存在资源的 X-Rewrite-Url 头的请求

```html
GET / HTTP/1.1
Host: www.example.com
X-Rewrite-URL: /donotexist2
[...]
```

如果任一请求的响应包含资源未找到的标记，则表明应用程序支持特殊请求头。这些标记可能包括 HTTP 响应状态代码 404，或响应体中的"资源未找到"消息。

一旦验证了对 `X-Original-URL` 或 `X-Rewrite-URL` 头的支持，就可以通过向应用程序发送预期请求（但将前端组件"允许"的 URL 指定为主请求 URL，并根据支持的头将真实目标 URL 指定在 `X-Original-URL` 或 `X-Rewrite-URL` 头中）来利用绕过访问控制限制的尝试。如果两者都支持，则逐一尝试以验证哪个头的绕过是有效的。

#### 4. 需要考虑的其他头

通常，管理面板或与管理相关的功能仅供本地网络上的客户端访问，因此可能滥用各种代理或转发相关的 HTTP头来获得访问权限。一些要测试的头和值包括：

- 头：
    - `X-Forwarded-For`
    - `X-Forward-For`
    - `X-Remote-IP`
    - `X-Originating-IP`
    - `X-Remote-Addr`
    - `X-Client-IP`
- 值
    - `127.0.0.1`（或 `127.0.0.0/8` 或 `::1/128` 地址空间中的任何地址）
    - `localhost`
    - 任何 [RFC1918](https://tools.ietf.org/html/rfc1918) 地址：
        - `10.0.0.0/8`
        - `172.16.0.0/12`
        - `192.168.0.0/16`
    - 链路本地地址：`169.254.0.0/16`

注意：除了地址或主机名外，还包括端口元素也可能有助于绕过边缘保护（如 Web 应用防火墙等）。
例如：`127.0.0.4:80`、`127.0.0.4:443`、`127.0.0.4:43982`

## 修复方案

对用户、角色和资源采用最小权限原则，确保不发生未授权访问。

## 工具

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org/)
    - [ZAP 插件：Access Control Testing](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)
- [Port Swigger Burp Suite](https://portswigger.net/burp)
    - [Burp 扩展：AuthMatrix](https://github.com/SecurityInnovation/AuthMatrix/)
    - [Burp 扩展：Autorize](https://github.com/Quitten/Autorize)

## 参考资料

[OWASP 应用安全验证标准 4.0.1](https://github.com/OWASP/ASVS/tree/master/4.0)，v4.0.1-1、v4.0.1-4、v4.0.1-9、v4.0.1-16
