# 测试角色定义

|ID          |
|------------|
|WSTG-IDNT-01|

## 概述

应用程序具有多种类型的功能和服务，这些功能和服务的访问权限基于用户的需求来确定。用户可以是：

- 管理员（administrator），负责管理应用程序的功能。
- 审计员（auditor），负责审查应用程序的事务并提供详细报告。
- 支持工程师（support engineer），负责帮助客户调试和修复其账户上的问题。
- 客户（customer），与应用程序交互并从中获得服务。

为了处理这些用途以及该应用程序的任何其他用例，需要设置角色定义（更常见的称呼是 [RBAC](https://en.wikipedia.org/wiki/Role-based_access_control)，基于角色的访问控制）。基于这些角色，用户能够完成所需的任务。

## 测试目标

- 识别并记录应用程序使用的角色。
- 尝试切换、更改或访问其他角色。
- 审查角色的粒度以及授予权限背后的需求。

## 如何测试

### 角色识别

测试人员应通过以下任何一种方法开始识别正在测试的应用程序角色：

- 应用程序文档。
- 应用程序开发人员或管理员的指导。
- 应用程序注释。
- 模糊测试可能的角色：
    - Cookie 变量（如 `role=admin`，`isAdmin=True`）
    - 账户变量（如 `Role: manager`）
    - 隐藏目录或文件（如 `/admin`，`/mod`，`/backups`）
    - 切换到已知用户（如 `admin`，`backups` 等）

### 切换到可用角色

在识别出可能的攻击向量后，测试人员需要测试并验证他们可以访问可用的角色。

> 一些应用程序在创建用户时通过严格的检查和策略来定义用户角色，或通过后端创建签名来确保用户角色得到适当保护。发现角色存在并不意味着它们存在漏洞。

### 审查角色权限

在获得对系统上角色的访问权限后，测试人员必须了解授予每个角色的权限。

支持工程师不应能够执行管理功能、管理备份或代表用户进行任何交易。

管理员不应拥有系统上的全部权限。敏感的管理功能应采用制衡原则（maker-checker principle），或使用 MFA 来确保管理员正在执行交易。2020年的 [Twitter 事件](https://blog.twitter.com/en_us/topics/company/2020/an-update-on-our-security-incident.html) 就是一个很好的例子。

## 工具

上述测试可以在不使用任何工具的情况下进行，除了用于访问系统的那一个。

为了使测试更容易且有据可查，可以使用：

- [Burp 的 Autorize 扩展](https://github.com/Quitten/Autorize)
- [ZAP 的访问控制测试附加组件](https://www.zaproxy.org/docs/desktop/addons/access-control-testing/)

## 参考资料

- [Role Engineering for Enterprise Security Management, E Coyne & J Davis, 2007](https://www.bookdepository.co.uk/Role-Engineering-for-Enterprise-Security-Management-Edward-Coyne/9781596932180)
- [角色工程和 RBAC 标准](https://csrc.nist.gov/projects/role-based-access-control#rbac-standard)
