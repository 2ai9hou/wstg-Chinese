# 测试路径混淆

|ID          |
|------------|
|WSTG-CONF-13|

## 概述

应用程序路径的正確配置很重要，因为如果路径配置不正确，攻击者可以利用此配置错误在后续阶段利用其他漏洞。

例如，如果路由配置不正确且目标还使用了 CDN，攻击者可以利用此配置错误执行 Web 缓存欺骗攻击。

因此，为了防止其他攻击，测试人员应评估此配置。

## 测试目标

- 确保应用程序路径配置正确。

## 如何测试

### 黑盒测试

在黑盒测试场景中，测试人员应将所有现有路径替换为不存在的路径，然后检查目标的行为和状态码。

例如，应用程序中有一个路径是仪表板，显示用户账户余额（金钱、游戏积分等）。

假设路径是 `https://example.com/user/dashboard`，测试人员应测试开发者可能考虑过的此路径的不同模式。对于 Web 缓存欺骗漏洞，分析师应考虑 `https:// example.com/user/dashboard/non.js` 这样的路径，如果仪表板信息可见且目标使用 CDN（或其他 Web 缓存），则 Web 缓存欺骗攻击可能适用。

### 白盒测试

检查应用程序路由配置，大多数时候，开发人员在应用程序路由中使用正则表达式。

在这个例子中，在 Django 框架应用程序的 `urls.py` 文件中，我们看到了路径混淆的一个示例。开发者没有使用正确的正则表达式，导致了漏洞：

```python
    from django.urls import re_path
    from . import views

    urlpatterns = [

        re_path(r'.*^dashboard', views.path_confusion ,name = 'index'),

    ]
```

如果路径 `https://example.com/dashboard/none.js` 也在浏览器中被用户打开，用户仪表板信息可以被显示，如果目标使用 CDN 或 Web 缓存，则可以实施 Web 缓存欺骗攻击。

## 工具

- [Zed Attack Proxy](https://www.zaproxy.org)
- [Burp Suite](https://portswigger.net/burp)

## 修复

- 不要根据文件扩展名或路径对缓存进行分类/处理（利用 content-type）。
- 确保缓存机制遵守应用程序指定的 cache-control 头。
- 实施符合 RFC 的文件未找到处理和重定向。

## 参考资料

- [绕过 Web 缓存中毒对策](https://portswigger.net/research/bypassing-web-cache-poisoning-countermeasures)
- [路径混淆：Web 缓存欺骗威胁用户在线信息](https://portswigger.net/daily-swig/path-confusion-web-cache-deception-threatens-user-information-online)
- [Web 缓存欺骗攻击](https://omergil.blogspot.com/2017/02/web-cache-deception-attack.html)
