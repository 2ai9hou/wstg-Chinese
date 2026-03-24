# 枚举基础设施和应用程序管理接口

|ID          |
|------------|
|WSTG-CONF-05|

## 概述

管理员接口可能存在于应用程序或应用程序服务器上，以允许某些用户在网站上执行特权活动。应该进行测试以揭示未经授权或标准用户是否可以访问此特权功能，以及如何访问。

应用程序可能需要管理员接口，以允许特权用户访问可能更改站点功能的更改。这些更改可能包括：

- 用户账户配置
- 站点设计和布局
- 数据操作
- 配置更改

在许多情况下，这些接口没有足够的控制来保护它们免受未经授权的访问。测试旨在发现这些管理员接口并访问为特权用户提供的功能。

## 测试目标

- 识别隐藏的管理员接口和功能。

## 如何测试

### 黑盒测试

以下部分描述了可用于测试管理员接口存在性的向量。这些技术也可用于测试相关问题，包括权限提升，在本指南的其他地方（例如[测试绕过授权模式](../05-Authorization_Testing/02-Testing_for_Bypassing_Authorization_Schema.md)和[测试不安全的直接对象引用](../05-Authorization_Testing/04-Testing_for_Insecure_Direct_Object_References.md)）有更详细的描述。

- 目录和文件枚举：管理员接口可能存在但对测试人员不可见。管理员接口的路径可以通过简单请求如 /admin 或 /administrator 来猜测。在某些情况下，可以使用高级 Google 搜索技术（在几秒钟内）揭示这些路径 - [Google dorks](https://www.exploit-db.com/google-hacking-database)。有许多工具可用于对服务器内容进行暴力破解，请参阅下面的工具部分以获取更多信息。测试人员可能还需要识别管理页面的文件名。强行浏览到已识别页面可能会提供对接口的访问。
- 源代码中的注释和链接：许多站点使用对所有站点用户加载的通用代码。通过检查发送到客户端的所有源代码，可以发现管理员功能的链接，并应进行调查。
- 审查服务器和应用程序文档：如果应用程序服务器或应用程序以其默认配置部署，则可能可以使用配置或帮助文档中描述的信息访问管理接口。如果找到管理接口且需要凭据，应查阅默认密码列表。
- 公开可用的信息：许多应用程序（如 WordPress）默认情况下有可用的管理接口。
- 替代服务器端口：管理接口可能在与主应用程序不同的主机端口上显示。例如，Apache Tomcat 的管理接口通常可以在端口 8080 上看到。
- 参数篡改：GET 或 POST 参数，或 cookie 可能需要启用管理员功能。这方面的线索包括隐藏字段的存在，例如：

```html
<input type="hidden" name="admin" value="no">
```

或在 cookie 中：

`Cookie: session_cookie; useradmin=0`

一旦发现管理接口，可以使用上述技术的组合来尝试绕过身份验证。如果失败，测试人员可能希望尝试暴力攻击。在这种情况下，测试人员应该意识到如果存在此功能，管理员账户被锁定的可能性。

### 灰盒测试

应该对服务器和应用程序组件进行更详细的检查，以确保强化（即管理员页面不向所有人访问，通过 IP 过滤或其他控制），并在适用的情况下验证所有组件不使用默认凭据或配置。
应审查源代码以确保授权和身份验证模型确保普通用户和站点管理员之间的清晰职责分离。应审查在普通用户和管理员用户之间共享的用户界面功能，以确保此类组件之间的清晰分离以及此类共享功能的信息泄露。

每个 Web 框架可能有自己的默认管理页面或路径，如以下示例：

PHP：

```html
/phpinfo
/phpmyadmin/
phpMyAdmin/
/mysqladmin/
/MySQLadmin
/MySQLAdmin
/login.php
/logon.php
/xmlrpc.php
/dbadmin
```

WordPress：

```html
wp-admin/
wp-admin/about.php
wp-admin/admin-ajax.php
wp-admin/admin-db.php
wp-admin/admin-footer.php
wp-admin/admin-functions.php
wp-admin/admin-header.php
```

Joomla：

```html
/administrator/index.php
/administrator/index.php?option=com_login
/administrator/index.php?option=com_content
/administrator/index.php?option=com_users
/administrator/index.php?option=com_menus
/administrator/index.php?option=com_installer
/administrator/index.php?option=com_config
```

Tomcat：

```html
/manager/html
/host-manager/html
/manager/text
/tomcat-users.xml
```

Apache：

```html
/index.html
/httpd.conf
/apache2.conf
/server-status
```

Nginx：

```html
/index.html
/index.htm
/index.php
/nginx_status
/index.php
/nginx.conf
/html/error
```

## 工具

Several tools can assist in identifying hidden administrator interfaces and functionality, including:

- [ZAP - Forced Browse](https://www.zaproxy.org/docs/desktop/addons/forced-browse/) 是 OWASP 以前的 DirBuster 项目的当前维护版本。
- [THC-HYDRA](https://github.com/vanhauser-thc/thc-hydra) 是一种允许对许多接口（包括基于表单的 HTTP 身份验证）进行暴力破解的工具。
- 当使用好的字典时，暴力破解器会更有效，例如 [Netsparker](https://www.netsparker.com/blog/web-security/svn-digger-better-lists-for-forced-browsing/) 字典。

## 参考资料

- [Cirt: Default Password list](https://cirt.net/passwords)
- [FuzzDB 可用于对管理员登录路径进行暴力浏览](https://github.com/fuzzdb-project/fuzzdb/blob/master/discovery/predictable-filepaths/login-file-locations/Logins.txt)
- [常见的调试参数](https://github.com/fuzzdb-project/fuzzdb/blob/master/attack/business-logic/CommonDebugParamNames.txt)
