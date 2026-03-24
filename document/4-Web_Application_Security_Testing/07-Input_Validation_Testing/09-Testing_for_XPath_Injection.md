# 测试XPath注入

|ID          |
|------------|
|WSTG-INPV-09|

## 概述

XPath是一种语言，主要设计用于寻址XML文档的部分。在XPath注入测试中，我们测试是否可以将XPath语法注入到由应用程序解释的请求中，从而允许攻击者执行用户控制的XPath查询。成功利用此漏洞可能允许攻击者绕过身份验证机制或未经授权访问信息。

Web应用程序严重依赖数据库来存储和访问其操作所需的数据。从历史上看，关系数据库一直是数据存储最常见的技术，但是近年来，我们看到使用XML语言组织数据的数据库越来越受欢迎。正如关系数据库通过SQL语言访问一样，XML数据库使用XPath作为其标准查询语言。

从概念上讲，XPath与SQL在其目的和应用方面非常相似，一个有趣的结果是XPath注入攻击遵循与[SQL注入](https://owasp.org/www-community/attacks/SQL_Injection)攻击相同的逻辑。在某些方面，XPath甚至比标准SQL更强大，因为其全部功能已经存在于其规范中，而SQL注入攻击中可以使用的许多技术取决于目标数据库使用的SQL方言的特性。这意味着XPath注入攻击可以更加适应性强和普遍存在。XPath注入攻击的另一个优点是，与SQL不同，没有ACL强制执行，因为我们的查询可以访问XML文档的每个部分。

## 测试目标

- 识别XPATH注入点。

## 如何测试

[XPath攻击模式首次由Amit Klein发布](https://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf)，与常规SQL注入非常相似。为了首先掌握问题，让我们想象一个管理应用程序身份验证的登录页面，用户必须在其中输入用户名和密码。让我们假设我们的数据库由以下XML文件表示：

```xml
<?xml version="1.0" encoding="ISO-8859-1"?>
<users>
    <user>
        <username>gandalf</username>
        <password>!c3</password>
        <account>admin</account>
    </user>
    <user>
        <username>Stefan0</username>
        <password>w1s3c</password>
        <account>guest</account>
    </user>
    <user>
        <username>tony</username>
        <password>Un6R34kb!e</password>
        <account>guest</account>
    </user>
</users>
```

返回用户名`gandalf`且密码为`!c3`的账户的XPath查询如下：

`string(//user[username/text()='gandalf' and password/text()='!c3']/account/text())`

如果应用程序不正确清理用户输入，测试人员将能够注入XPath代码并干扰查询结果。例如，测试人员可以输入以下值：

```text
Username: ' or '1' = '1
Password: ' or '1' = '1
```

看起来很熟悉，不是吗？使用这些参数，查询变为：

`string(//user[username/text()='' or '1' = '1' and password/text()='' or '1' = '1']/account/text())`

与常见SQL注入攻击一样，我们创建了一个始终计算为真的查询，这意味着即使用户未提供用户名或密码，应用程序也将对用户进行身份验证。与常见SQL注入攻击一样，使用XPath注入，第一步是在要测试的字段中插入单引号（`'`），在查询中引入语法错误，并检查应用程序是否返回错误消息。

如果没有关于XML数据内部细节的知识，并且应用程序没有提供有助于重建其内部逻辑的有用错误消息，则可以执行[盲XPath注入](https://owasp.org/www-community/attacks/Blind_XPath_Injection)攻击，其目标是重建整个数据结构。该技术类似于基于推理的SQL注入，因为方法是注入代码以创建返回一位信息的查询。[盲XPath注入](https://owasp.org/www-community/attacks/Blind_XPath_Injection)在Amit Klein的参考论文中有更详细的解释。

## 参考资料

### 白皮书

- [Amit Klein："盲XPath注入"](https://dl.packetstormsecurity.net/papers/bypass/Blind_XPath_Injection_20040518.pdf)
- [XPath 1.0规范](https://www.w3.org/TR/1999/REC-xpath-19991116/)
