# 测试LDAP注入

|ID          |
|------------|
|WSTG-INPV-06|

## 概述

轻量级目录访问协议（LDAP）用于存储有关用户、主机和许多其他对象的信息。[LDAP注入](https://wiki.owasp.org/index.php/LDAP_injection)是一种服务器端攻击，可允许泄露在LDAP结构中表示的用户和主机的敏感信息、修改或插入信息。这是通过操纵输入参数实现的，这些参数随后被传递到内部搜索、添加和修改函数。

Web应用程序可以使用LDAP让用户对目录结构中的其他用户信息进行身份验证或搜索。LDAP注入攻击的目标是注入LDAP搜索过滤器元字符，这些字符将被应用程序执行。

[Rfc2254](https://www.ietf.org/rfc/rfc2254.txt)定义了如何在LDAPv3上构建搜索过滤器的语法，并扩展了[Rfc1960](https://www.ietf.org/rfc/rfc1960.txt)（LDAPv2）。

LDAP搜索过滤器以波兰表示法构建，也称为[波兰表示法](https://en.wikipedia.org/wiki/Polish_notation)。

这意味着搜索过滤器上的伪代码条件如：

`find("cn=John & userPassword=mypass")`

将表示为：

`find("(&(cn=John)(userPassword=mypass))")`

可以使用以下元字符在LDAP搜索过滤器上应用布尔条件和组聚合：

| 元字符 | 含义              |
|----------|-----------------------|
| &        | 布尔AND          |
| \|       | 布尔OR           |
| !        | 布尔NOT          |
| =        | 等于               |
| ~=       | 近似               |
| >=       | 大于               |
| <=       | 小于               |
| *        | 任意字符        |
| ()       | 分组括号 |

有关构建搜索过滤器的更完整示例，请参阅相关RFC。

成功利用LDAP注入漏洞可使测试人员：

- 访问未授权内容
- 逃避应用程序限制
- 收集未授权信息
- 在LDAP树结构中添加或修改对象

## 测试目标

- 识别LDAP注入点。
- 评估注入的严重程度。

## 如何测试

### 示例1：搜索过滤器

假设我们有一个使用如下搜索过滤器的Web应用程序：

`searchfilter="(cn="+user+")"`

通过如下HTTP请求实例化：

`https://www.example.com/ldapsearch?user=John`

如果值`John`被`*`替换，通过发送请求：

`https://www.example.com/ldapsearch?user=*`

过滤器将如下：

`searchfilter="(cn=*)"`

这将匹配其'cn'属性等于任何内容的每个对象。

如果应用程序容易受到LDAP注入，它将显示部分或全部用户属性，这取决于应用程序的执行流程和LDAP连接用户的权限。

测试人员可以使用试错方法，在参数中插入`(`, `|`, `&`, `*`和其他字符，以检查应用程序的错误。

### 示例2：登录

如果Web应用程序使用LDAP在登录过程中检查用户凭据，并且容易受到LDAP注入，则可以通过注入始终为真的LDAP查询来绕过身份验证检查（类似于SQL和XPATH注入的方式）。

假设Web应用程序使用过滤器匹配LDAP用户/密码对。

`searchlogin= "(&(uid="+user+")(userPassword={MD5}"+base64(pack("H*",md5(pass)))+"))";`

通过使用以下值：

```txt
user=*)(uid=*))(|(uid=*
pass=password
```

搜索过滤器的结果为：

`searchlogin="(&(uid=*)(uid=*))(|(uid=*)(userPassword={MD5}X03MO1qnZdYdgyfeuILPmQ==))";`

这是正确且始终为真的。这样，测试人员将作为LDAP树中的第一个用户获得登录状态。

## 工具

- [Softerra LDAP浏览器](https://www.ldapadministrator.com)

## 参考资料

- [LDAP注入防范速查表](https://cheatsheetseries.owasp.org/cheatsheets/LDAP_Injection_Prevention_Cheat_Sheet.html)

### 白皮书

- [Sacha Faust：LDAP注入：您的应用程序容易受到攻击吗？](httphttps://www.networkdls.com/articles/ldapinjection.pdf)
- [IBM论文：理解LDAP](https://www.redbooks.ibm.com/redbooks/pdfs/sg244986.pdf)
- [RFC 1960：LDAP搜索过滤器的字符串表示](https://www.ietf.org/rfc/rfc1960.txt)
- [LDAP注入](https://www.blackhat.com/presentations/bh-europe-08/Alonso-Parada/Whitepaper/bh-eu-08-alonso-parada-WP.pdf)
