# OWASP Web 安全测试指南

[![欢迎贡献](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat)](https://github.com/OWASP/wstg/issues)
[![OWASP 旗舰项目](https://img.shields.io/badge/owasp-flagship-brightgreen.svg)](https://owasp.org/projects/)
[![Twitter 关注](https://img.shields.io/twitter/follow/owasp_wstg?style=social)](https://x.com/owasp_wstg)

[![知识共享许可](https://licensebuttons.net/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/ "CC BY-SA 4.0")

欢迎来到开放全球应用安全项目®（OWASP®）Web 安全测试指南（WSTG）的官方仓库。WSTG 是一份全面的 Web 应用和 Web 服务安全测试指南。由安全专业人员和志愿者的协作努力创建，WSTG 提供了一套被全球渗透测试人员和组织使用的最佳实践框架。

我们目前正在进行 5.0 版本的开发。您可以[在 GitHub 上阅读当前文档](https://github.com/OWASP/wstg/tree/master/document)。

如需查看最新稳定版本，请[查看 4.2 版本](https://github.com/OWASP/wstg/releases/tag/v4.2)。也可以[在线访问](https://owasp.org/www-project-web-security-testing-guide/v42/)。

- [如何引用 WSTG 场景](#如何引用-wstg-场景)
    - [链接](#链接)
- [贡献、功能请求和反馈](#贡献功能请求和反馈)
- [与我们交流](#与我们交流)
- [项目负责人](#项目负责人)
- [核心团队](#核心团队)
- [翻译](#翻译)

## 如何引用 WSTG 场景

每个场景都有一个格式为 `WSTG-<category>-<number>` 的标识符，其中：'category' 是一个 4 字符大写字符串，用于标识测试或弱点的类型，'number' 是一个从 01 到 99 的零填充数值。例如：`WSTG-INFO-02` 是第二个信息收集测试。

标识符可能在不同版本之间发生变化。因此，其他文档、报告或工具最好使用格式：`WSTG-<version>-<category>-<number>`，其中：'version' 是去掉标点符号的版本标签。例如：`WSTG-v42-INFO-02` 将被理解为特指 4.2 版中第二个信息收集测试。

如果使用的标识符不包括 `<version>` 元素，则应假定它们指的是最新版的 Web 安全测试指南内容。显然，随着指南的发展和变化，这可能会变得有问题，这就是为什么作者或开发人员应该包括版本元素。

### 链接

链接到 Web 安全测试指南场景应使用版本化链接，而不是 `stable` 或 `latest`，后者肯定会随时间变化。但是，项目团队的意图是版本化链接不会改变。例如：`https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/01-Information_Gathering/02-Fingerprint_Web_Server.html`。注意：`v42` 元素指的是 4.2 版。

## 贡献、功能请求和反馈

我们正在积极邀请新的贡献者！首先，请阅读[贡献指南](CONTRIBUTING.md)。

第一次来这里？以下是 [GitHub 为首次贡献者提供的建议](https://github.com/OWASP/wstg/contribute)。

这个项目离不开许多志愿者的辛勤工作。鼓励每个人以各种方式提供帮助。以下是您可以提供帮助的一些方式：

- 阅读当前内容，帮助我们修复任何拼写错误或语法错误。
- 帮助[翻译](CONTRIBUTING.md#translation)工作。
- 选择一个现有议题，提交 Pull Request 来修复。
- 提交一个新的议题来报告改进机会。

要了解如何成功贡献，请阅读[贡献指南](CONTRIBUTING.md)。

成功贡献者的名单会显示在[项目的作者、审校者或编辑列表](document/1-Frontispiece/README.md)中。

## 与我们交流

您可以在 Slack 上轻松找到我们：

1. 使用此[邀请链接](https://owasp.org/slack/invite)加入 OWASP Group Slack。
2. 加入本项目的[频道，#testing-guide](https://app.slack.com/client/T04T40NHX/CJ2QDHLRJ)。

欢迎提问、提出想法或分享您的最佳实践。

您可以在 𝕏（Twitter）上 @ 我们 [@owasp_wstg](https://x.com/owasp_wstg)。

您也可以加入我们的 [Google Group](https://groups.google.com/a/owasp.org/forum/#!forum/testing-guide-project)。

## 项目负责人

- [Rick Mitchell](https://github.com/kingthorin)
- [Elie Saad](https://github.com/ThunderSon)

## 核心团队

- [Rejah Rehim](https://github.com/rejahrehim)
- [Victoria Drake](https://github.com/victoriadrake)

## 翻译

- [葡萄牙语-BR](https://github.com/doverh/wstg-translations-pt)
- [俄语](https://github.com/andrettv/WSTG/tree/master/WSTG-ru)
- [波斯语（法尔斯语）](https://github.com/whoismh11/owasp-wstg-fa)
- [土耳其语](https://github.com/enoskom/Owasp-wstg)
- [西班牙语](https://github.com/frangelbarrera/wstg)

---

开放全球应用安全项目和 OWASP 是 OWASP 基金会的注册商标。
