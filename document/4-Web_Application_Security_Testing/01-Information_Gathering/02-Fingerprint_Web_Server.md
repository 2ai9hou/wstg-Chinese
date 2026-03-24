# 识别 Web 服务器指纹

|ID          |
|------------|
|WSTG-INFO-02|

## 摘要

Web 服务器指纹识别是识别目标运行的网络服务器类型和版本的任务。虽然 Web 服务器指纹识别通常包含在自动化测试工具中，但对于研究人员来说，理解这些工具尝试识别软件的基础知识以及为什么这有用是很重要的。

准确发现应用运行的 Web 服务器类型可以使安全测试人员能够确定应用是否容易受到攻击。特别地，运行没有最新安全补丁的软件旧版本的服务器可能容易受到特定版本已知漏洞的影响。

## 测试目标

- 确定运行中的 Web 服务器的版本和类型，以启用进一步发现任何已知漏洞。

## 如何测试

用于 Web 服务器指纹识别的技术包括 [banner grabbing](https://en.wikipedia.org/wiki/Banner_grabbing)、对错误请求的响应以及使用自动化工具执行使用多种策略的更健壮扫描。所有这些技术运行的基本前提是相同的。它们都努力从 Web 服务器引发一些响应，然后可以与已知响应和行为的数据库进行比较，从而匹配到已知服务器类型。

### Banner Grabbing

Banner grabbing 通过向 Web 服务器发送 HTTP 请求并检查其[响应头](https://developer.mozilla.org/en-US/docs/Glossary/Response_header)来执行。这可以使用各种工具完成，包括用于 HTTP 请求的 `telnet`，或用于 TLS/SSL 请求的 `openssl`。

例如，这是发送到 Apache 服务器的请求的响应。

```http
HTTP/1.1 200 OK
Date: Thu, 05 Sep 2019 17:42:39 GMT
Server: Apache/2.4.41 (Unix)
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
ETag: "75-591d1d21b6167"
Accept-Ranges: bytes
Content-Length: 117
Connection: close
Content-Type: text/html
...
```

这是另一个响应，这次是由 nginx 发送的。

```http
HTTP/1.1 200 OK
Server: nginx/1.17.3
Date: Thu, 05 Sep 2019 17:50:24 GMT
Content-Type: text/html
Content-Length: 117
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Connection: close
ETag: "5d71489a-75"
Accept-Ranges: bytes
...
```

这是 lighttpd 发送的响应。

```sh
HTTP/1.0 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4192788355"
Last-Modified: Thu, 05 Sep 2019 17:40:42 GMT
Content-Length: 117
Connection: close
Date: Thu, 05 Sep 2019 17:57:57 GMT
Server: lighttpd/1.4.54
```

在这些示例中，服务器类型和版本是明确暴露的。然而，注重安全的应用可能通过修改头部来混淆服务器信息。例如，以下是对具有修改头部的站点的请求的响应摘要：

```sh
HTTP/1.1 200 OK
Server: Website.com
Date: Thu, 05 Sep 2019 17:57:06 GMT
Content-Type: text/html; charset=utf-8
Status: 200 OK
...
```

在服务器信息被混淆的情况下，测试人员可能根据头部字段的顺序猜测服务器类型。请注意，在上面的 Apache 示例中，字段按以下顺序：

- Date
- Server
- Last-Modified
- ETag
- Accept-Ranges
- Content-Length
- Connection
- Content-Type

然而，在 nginx 和被混淆的服务器示例中，公共字段按以下顺序：

- Server
- Date
- Content-Type

测试人员可以使用此信息猜测被混淆的服务器是 nginx。然而，考虑到许多不同的 Web 服务器可能共享相同的字段顺序，并且字段可以被修改或删除，此方法不是确定性的。

### 发送错误请求

Web 服务器可以通过检查其错误响应来识别，在这些错误响应未被定制的情况下，还包括它们的默认错误页面。强制服务器显示这些的一种方法是发送有意错误或格式错误的请求。

例如，以下是对不存在的 HTTP 版本 `SANTA CLAUS` 的请求从 Apache 服务器的响应。

```sh
GET / SANTA CLAUS/1.1


HTTP/1.1 400 Bad Request
Date: Fri, 06 Sep 2019 19:21:01 GMT
Server: Apache/2.4.41 (Unix)
Content-Length: 226
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
</body></html>
```

这是从 nginx 对相同请求的响应。

```sh
GET / SANTA CLAUS/1.1


<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.17.3</center>
</body>
</html>
```

这是从 lighttpd 对相同请求的响应。

```sh
GET / SANTA CLAUS/1.1


HTTP/1.0 400 Bad Request
Content-Type: text/html
Content-Length: 345
Connection: close
Date: Sun, 08 Sep 2019 21:56:17 GMT
Server: lighttpd/1.4.54

<?xml version="1.0" encoding="iso-8859-1"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
         "https://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="https://www.w3.org/1999/xhtml/" xml:lang="en" lang="en">
 <head>
  <title>400 Bad Request</title>
 </head>
 <body>
  <h1>400 Bad Request</h1>
 </body>
</html>
```

由于默认错误页面提供了 Web 服务器类型之间的许多差异化因素，即使服务器头字段被混淆，它们的检查也可以是一种有效的指纹识别方法。此外，此[资源](https://0xdf.gitlab.io/cheatsheets/404)可以派上用场，特别是当你遇到不披露 Web 服务器类型的默认错误页面时。

### 使用自动化扫描工具

如前所述，Web 服务器指纹识别通常作为自动化扫描工具的功能包含在内。这些工具能够发出类似于上面演示的请求，以及发送其他更具服务器特定性的探测。自动化工具可以比手动测试更快地比较 Web 服务器的响应，并利用已知响应的大型数据库来尝试服务器识别。由于这些原因，自动化工具更有可能产生准确的结果。

以下是一些常用扫描工具的列表，这些工具包含 Web 服务器指纹识别功能：

- [Netcraft](https://toolbar.netcraft.com/site_report)，一个在线工具，扫描站点的信息，包括 Web 服务器详情。
- [Nikto](https://github.com/sullo/nikto)，一个开源命令行扫描工具。
- [Nmap](https://nmap.org/)，一个也有 GUI [Zenmap](https://nmap.org/zenmap/) 的开源命令行工具。

## 修复

虽然暴露的服务器信息本身不一定是漏洞，但它是可能帮助攻击者利用其他可能存在的漏洞的信息。暴露的服务器信息也可能导致攻击者发现特定版本的服务器漏洞，这些漏洞可用于利用未修补的服务器。因此，建议采取一些预防措施。这些行动包括：

- 在头部中混淆 Web 服务器信息，例如使用 Apache 的 [mod_headers 模块](https://httpd.apache.org/docs/current/mod/mod_headers.html)。
- 使用强化的[反向代理服务器](https://en.wikipedia.org/wiki/Proxy_server#Reverse_proxies)在 Web 服务器和互联网之间创建额外的安全层。
- 确保 Web 服务器使用最新软件和安全补丁保持更新。
