# 测试会话劫持

|ID          |
|------------|
|WSTG-SESS-09|

## 概述

攻击者获取用户会话Cookie后，可以通过显示这些Cookie来冒充他们。这种攻击称为会话劫持。在考虑网络攻击者（即控制受害者使用的网络的攻击者）时，会话Cookie可能通过HTTP被不当暴露给攻击者。为防止这种情况，应使用`Secure`属性标记会话Cookie，以便它们仅通过HTTPS通信。

请注意，当Web应用完全部署在HTTPS上时，也应使用`Secure`属性，否则可能发生以下Cookie盗窃攻击。假设`example.com`完全部署在HTTPS上，但未将其会话Cookie标记为`Secure`。可能发生以下攻击步骤：

1. 受害者向`https://another-site.com`发送请求。
2. 攻击者破坏相应响应，使其触发对`https://example.com`的请求。
3. 浏览器现在尝试访问`https://example.com`。
4. 虽然请求失败，但会话Cookie通过HTTP明文泄露。

或者，可以通过使用[HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security)禁止使用HTTP来防止会话劫持。请注意，这里有一个与Cookie作用域相关的微妙之处。特别是，当会话Cookie使用设置了`Domain`属性时，需要完整的HSTS采用。

完整的HSTS采用在一篇名为*Testing for Integrity Flaws in Web Sessions*的论文中描述，作者是Stefano Calzavara、Alvise Rabitti、Alessio Ragazzo和Michele Bugliesi。完整的HSTS采用发生在主机为自己及其所有子域激活HSTS时。部分HSTS采用是主机仅为自己激活HSTS。

使用设置了`Domain`属性，会话Cookie可以在子域之间共享。应避免与子域一起使用HTTP，以防止通过HTTP发送的未加密Cookie泄露。为了说明这个安全缺陷，假设站点`example.com`激活HSTS但没有`includeSubDomains`选项。该站点发出将`Domain`属性设置为`example.com`的会话Cookie。可能发生以下攻击：

1. 受害者向`https://another-site.com`发送请求。
2. 攻击者破坏相应响应，使其触发对`https://fake.example.com`的请求。
3. 浏览器现在尝试访问`https://fake.example.com`，这是HSTS配置允许的。
4. 由于请求被发送到具有设置`Domain`属性的`example.com`子域，它包含会话Cookie，这些Cookie通过HTTP明文泄露。

应在apex域上激活完整HSTS以防止此攻击。

## 测试目标

- 识别有漏洞的会话Cookie。
- 劫持有漏洞的Cookie并评估风险级别。

## 如何测试

测试策略针对网络攻击者，因此只需应用于没有完整HSTS采用的站点（具有完整HSTS采用的站点是安全的，因为它们的Cookie不会通过HTTP通信）。我们假设在测试站点上有两个测试账户，一个充当受害者，一个充当攻击者。我们模拟一个场景，攻击者窃取所有未通过HTTP泄露保护的Cookie，并将它们呈现给站点以访问受害者的账户。如果这些Cookie足以代表受害者行事，会话劫持是可能的。

以下是执行此测试的步骤：

1. 以受害者身份登录并到达任何需要认证的安全功能页面。
2. 从Cookie存储中删除满足以下任何条件的Cookie：
    - 如果没有HSTS采用：设置了`Secure`属性。
    - 如果有部分HSTS采用：设置了`Secure`属性或未设置`Domain`属性。
3. 保存Cookie存储的快照。
4. 触发步骤1中识别的安全功能。
5. 观察步骤4中的操作是否成功执行。如果成功，攻击成功。
6. 清除Cookie存储，以攻击者身份登录并到达步骤1中的页面。
7. 逐个将步骤3中保存的Cookie写入Cookie存储。
8. 再次触发步骤1中识别的安全功能。
9. 清除Cookie存储，再次以受害者身份登录。
10. 观察步骤8中的操作是否在受害者的账户中成功执行。如果成功，攻击成功；否则，站点可以防御会话劫持。

我们建议为受害者和攻击者使用两台不同的机器或浏览器。这允许您在Web应用进行指纹识别以验证从给定Cookie启用的访问时减少误报。测试策略的一个更短但精度较低的变体只需要一个测试账户。它遵循相同的模式，但在步骤5停止（请注意，这使得步骤3无用）。

## 工具

- [ZAP](https://www.zaproxy.org)
- [JHijack - 数字会话劫持工具](https://sourceforge.net/projects/jhijack/)
