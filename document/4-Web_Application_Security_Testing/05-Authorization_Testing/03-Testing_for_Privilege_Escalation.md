# 测试权限提升

|ID          |
|------------|
|WSTG-ATHZ-03|

## 概述

本节描述了将权限从一级别提升到另一级别的问题。在此阶段，测试人员应验证用户不可能以可能导致权限提升攻击的方式修改应用程序中的权限或角色。

当用户获得超出其正常允许范围的更多资源或功能访问权限时，就会发生权限提升，而这种提升或变更本应被应用程序阻止。这通常是由应用程序的缺陷引起的。结果是应用程序以比开发人员或系统管理员预期更多权限执行操作。

权限提升的程度取决于攻击者被授权拥有的权限，以及在成功利用中可以获得的权限。例如，允许用户在成功认证后获得额外权限的编程错误限制了提权的程度，因为用户已经被授权持有某些权限。同样，无需任何身份验证即可获得超级用户权限的远程攻击者表现出更大程度的权限提升。

通常，当能够访问授予更高权限账户的资源时（如获得应用程序的管理权限），人们称之为*垂直提权*；当能够访问授予配置相似的账户的资源时（如在在线银行应用程序中访问不同用户的信息），称之为*水平提权*。

## 测试目标

- 识别与权限操作相关的注入点。
- 模糊测试或以其他方式尝试绕过安全措施。

## 如何测试

### 测试角色/权限操作

在应用程序中用户可以创建信息（如进行支付、添加联系人或发送消息）、接收信息（如账户对账单、订单详情）或删除信息（如删除用户、消息）的每个部分，有必要记录该功能。测试人员应尝试作为另一个用户访问这些功能，以验证是否可能访问用户角色/权限不允许（但可能作为另一个用户允许）的功能。

#### 用户组操作

例如：
以下 HTTP POST 允许属于 `grp001` 的用户访问订单 #0001：

```http
POST /user/viewOrder.jsp HTTP/1.1
Host: www.example.com
...

groupID=grp001&orderID=0001
```

验证不属于 `grp001` 的用户是否可以修改 `groupID` 和 `orderID` 参数的值以获得对该特权数据的访问。

#### 用户配置文件操作

例如：
以下服务器响应显示在成功认证后返回给用户的 HTML 中的隐藏字段：

```html
HTTP/1.1 200 OK
Server: Netscape-Enterprise/6.0
Date: Wed, 1 Apr 2006 13:51:20 GMT
Set-Cookie: USER=aW78ryrGrTWs4MnOd32Fs51yDqp; path=/; domain=www.example.com
Set-Cookie: SESSION=k+KmKeHXTgDi1J5fT7Zz; path=/; domain= www.example.com
Cache-Control: no-cache
Pragma: No-cache
Content-length: 247
Content-Type: text/html
Expires: Thu, 01 Jan 1970 00:00:00 GMT
Connection: close

<form  name="autoriz" method="POST" action = "visual.jsp">
<input type="hidden" name="profile" value="SysAdmin">\

<body onload="document.forms.autoriz.submit()">
</td>
</tr>
```

如果测试人员将变量 `profile` 的值修改为 `SysAdmin` 会怎样？是否可能成为**管理员**？

#### 条件值操作

例如：
在服务器将错误消息作为一组响应代码中特定参数的值发送的环境中，如下所示：

```text
@0`1`3`3``0`UC`1`Status`OK`SEC`5`1`0`ResultSet`0`PVValid`-1`0`0` Notifications`0`0`3`Command  Manager`0`0`0` StateToolsBar`0`0`0`
StateExecToolBar`0`0`0`FlagsToolBar`0
```

服务器给予用户隐式信任。它相信用户将用上述消息关闭会话。

在这种情况下，验证通过修改参数值来提升权限是否不可能。在这个特定示例中，通过将 `PVValid` 值从 `-1` 修改为 `0`（无错误条件），可能可以作为管理员对服务器进行身份验证。

#### IP 地址操作

一些站点根据 IP 地址限制访问或计算失败登录尝试次数。

例如：

```text
X-Forwarded-For: 8.1.1.1
```

在这种情况下，如果站点使用 `X-forwarded-For` 的值作为客户端 IP 地址，测试人员可能需要更改 `X-forwarded-For` HTTP 头的 IP 值以绕过 IP 源识别。

### 测试垂直绕过授权架构

垂直授权绕过专门针对攻击者获得比自己更高角色的情况。对这种绕过的测试集中在验证垂直授权架构如何为每个角色实现的。对于应用程序执行的每个功能、页面、特定角色或请求，有必要验证是否可能：

- 访问仅应供更高角色用户访问的资源。
- 操作仅应供持有更高或特定角色身份的用户执行的功能。

对于每个角色：

1. 注册一个用户。
2. 基于两个不同的角色建立并保持两个不同的会话。
3. 对于每个请求，将会话标识符从原始会话更改为另一角色的会话标识符，并评估每个的响应。
4. 如果权限较弱的会话包含相同的数据，或表明在更高权限功能上成功操作，则该应用程序被认为是存在漏洞的。

#### 银行站点角色场景

下表说明了一个银行站点的系统角色。每个角色与事件菜单功能的特定权限绑定：

|      角色     |     权限    | 附加权限 |
|---------------|-------------|----------|
| 管理员        | 完全控制    | 删除     |
| 经理          | 修改、添加、读取 | 添加  |
| 员工          | 读取、修改  | 修改     |
| 客户          | 只读        |          |

在以下情况下，应用程序将被视为存在漏洞：

1. 客户可以操作管理员、经理或员工功能；
2. 员工用户可以操作经理或管理员功能；
3. 经理可以操作管理员功能。

假设 `deleteEvent` 功能是应用程序管理员账户菜单的一部分，并且可以通过请求以下 URL 访问：`https://www.example.com/account/deleteEvent`。然后，在调用 `deleteEvent` 功能时生成以下 HTTP 请求：

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[其他 HTTP 头]
Cookie: SessionID=ADMINISTRATOR_USER_SESSION

EventID=1000001
```

有效响应：

```http
HTTP/1.1 200 OK
[其他 HTTP 头]

{"message": "Event was deleted"}
```

攻击者可能尝试执行相同的请求：

```http
POST /account/deleteEvent HTTP/1.1
Host: www.example.com
[其他 HTTP 头]
Cookie: SessionID=CUSTOMER_USER_SESSION

EventID=1000002
```

如果攻击者请求的响应包含相同的数据 `{"message": "Event was deleted"}`，则该应用程序存在漏洞。

#### 管理员页面访问

假设管理员菜单是管理员账户的一部分。

如果除管理员以外的任何角色可以访问管理员菜单，则该应用程序将被视为存在漏洞。有时，开发人员仅在 GUI 级别执行授权验证，而不对功能进行授权验证，因此可能导致漏洞。

### URL 遍历

尝试遍历站点并检查是否有某些页面可能缺少授权检查。

例如：

```text
/../.././userInfo.html
```

### 白盒测试

如果 URL 授权检查仅通过部分 URL 匹配完成，那么测试人员或黑客可能通过 URL 编码技术绕过授权。

例如：

```text
startswith(), endswith(), contains(), indexOf()
```

## 参考资料

### 白皮书

- [Wikipedia - 权限提升](https://en.wikipedia.org/wiki/Privilege_escalation)

## 工具

- [Zed Attack Proxy (ZAP)](https://www.zaproxy.org)
