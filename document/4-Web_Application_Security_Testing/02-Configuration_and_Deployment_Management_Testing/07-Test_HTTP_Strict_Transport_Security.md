# 测试 HTTP 严格传输安全

|ID          |
|------------|
|WSTG-CONF-07|

## 概述

HTTP 严格传输安全（HSTS）功能使 Web 服务器能够通过特殊的响应头通知用户的浏览器，它应该永远不会与指定域服务器建立未加密的 HTTP 连接。相反，它应该自动通过 HTTPS 建立所有连接请求来访问站点。这还可以防止用户覆盖证书错误。

考虑到此安全措施的重要性，验证站点是否使用了此 HTTP 头是很重要的，以确保所有数据在 Web 浏览器和服务器之间加密传输。

HTTP 严格传输安全头使用三个特定指令：

- `max-age`：指示浏览器应自动将所有 HTTP 请求转换为 HTTPS 的秒数。
- `includeSubDomains`：指示所有相关子域必须使用 HTTPS。
- `preload` 非官方：指示域在预加载列表中，浏览器不应在没有 HTTPS 的情况下连接。
    - 虽然所有主要浏览器都支持这一点，但它不是规范的一部分。（有关更多信息，请参阅 [hstspreload.org](https://hstspreload.org/)。）

以下是 HSTS 头实现的示例：

`Strict-Transport-Security: max-age=31536000; includeSubDomains`

必须检查此头的存在，因为缺少此头可能导致安全问题，例如：

- 攻击者拦截和访问通过未加密网络通道传输的信息。
- 攻击者通过利用接受不受信任证书的用户进行中间人（MITM）攻击。
- 用户错误地在浏览器中输入使用 HTTP 而不是 HTTPS 的地址，或者 Web 应用程序中的链接错误地使用 HTTP 协议。

## 测试目标

- 审查 HSTS 头及其有效性。

## 如何测试

- 通过拦截代理检查服务器的响应，确认 HSTS 头的存在。
- 按如下方式使用 curl：

```bash
$ curl -s -D- https://owasp.org | grep -i strict-transport-security:
Strict-Transport-Security: max-age=31536000
```

## 参考资料

- [OWASP HTTP 严格传输安全速查表](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html)
- [OWASP Appsec 教程系列 - 第 4 集：严格传输安全](https://www.youtube.com/watch?v=zEV3HOuM_Vw)
- [HSTS 规范](https://tools.ietf.org/html/rfc6797)
- [在 Apache 中启用 HTTP 严格传输安全](https://https.cio.gov/hsts/)
- [在 Nginx 中启用 HTTP 严格传输安全](https://www.nginx.com/blog/http-strict-transport-security-hsts-and-nginx/)
