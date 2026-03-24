# 测试易受攻击的记住密码功能

|ID          |
|------------|
|WSTG-ATHN-05|

## 概述

凭据是使用最广泛的认证技术。由于用户名-密码对的使用如此广泛，用户已无法在众多应用程序中正确处理他们的凭据。

为了帮助用户管理凭据，出现了多种技术：

- 应用程序提供"记住我"功能，允许用户在很长时间内保持认证状态，而无需再次询问用户凭据。
- 密码管理器 - 包括浏览器密码管理器 - 允许用户以安全方式存储凭据，随后在用户表单中注入，而无需任何用户干预。

## 测试目标

- 验证生成的会话是否被安全管理，不会危及用户的凭据。

## 如何测试

由于这些方法提供了更好的用户体验并使用户无需记住所有凭据，它们增加了攻击面。一些应用程序：

- 以编码形式将凭据存储在浏览器的存储机制中，可通过遵循 [Web 存储测试场景](../11-Client-side_Testing/12-Testing_Browser_Storage.md)并通过[会话分析](../06-Session_Management_Testing/01-Testing_for_Session_Management_Schema.md#session-analysis)场景进行验证。凭据不应以任何方式存储在客户端应用程序中，应替换为服务器端生成的令牌。
- 自动注入用户凭据可能被滥用：
    - [点击劫持](../11-Client-side_Testing/09-Testing_for_Clickjacking.md)攻击。
    - [CSRF](../06-Session_Management_Testing/05-Testing_for_Cross_Site_Request_Forgery.md)攻击。
- 应分析令牌的令牌生命周期，其中一些令牌永不过期，如果这些令牌被盗，会使用户处于危险之中。确保遵循[会话超时](../06-Session_Management_Testing/07-Testing_Session_Timeout.md)测试场景。

## 修复

- 遵循[会话管理](https://cheatseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)最佳实践。
- 确保没有以明文形式存储凭据，也没有以易于检索的编码或加密形式存储在浏览器存储机制中；它们应存储在服务器端并遵循良好的[密码存储](https://cheatseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)实践。
