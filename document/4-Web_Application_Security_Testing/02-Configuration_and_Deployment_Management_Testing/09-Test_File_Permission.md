# 测试文件权限

|ID          |
|------------|
|WSTG-CONF-09|

## 概述

当资源的权限设置向比所需更广泛的参与者提供访问权限时，可能导致敏感信息泄露或资源被意外方修改。当资源与程序配置、执行或敏感用户数据相关时，这尤其危险。

一个明显的例子是可由未经授权用户运行的可执行文件。另一个例子是用于访问 API 的账户信息或令牌值。这些在现代 Web 服务和微服务中越来越常见，可能存储在默认情况下安装时具有全局可读权限的配置文件中。这类敏感数据可能通过恶意内部参与者或远程攻击者泄露。远程攻击者可能通过其他漏洞危害服务，但仅获得普通用户权限。

## 测试目标

- 审查并识别任何有问题的文件权限。

## 如何测试

在 Linux 中，使用 `ls` 命令检查文件权限。或者，也可以使用 `namei` 递归列出文件权限。

`$ namei -l /PathToCheck/`

需要测试文件权限的文件和目录包括但不限于：

- Web 文件/目录
- 配置文件/目录
- 敏感文件（加密数据、密码、密钥）/目录
- 日志文件（安全日志、操作日志、管理日志）/目录
- 可执行文件（脚本、EXE、JAR、class、PHP、ASP）/目录
- 数据库文件/目录
- 临时文件/目录
- 上传文件/目录

## 修复

正确设置文件和目录的权限，以防止未经授权的用户访问关键资源。

## 工具

- [Windows AccessEnum](https://technet.microsoft.com/en-us/sysinternals/accessenum)
- [Windows AccessChk](https://technet.microsoft.com/en-us/sysinternals/accesschk)
- [Linux namei](https://linux.die.net/man/1/namei)

## 参考资料

- [CWE-732：关键资源的权限分配错误](https://cwe.mitre.org/data/definitions/732.html)
