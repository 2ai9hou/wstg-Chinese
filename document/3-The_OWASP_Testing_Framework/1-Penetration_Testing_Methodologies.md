# 渗透测试方法论

## 摘要

- [OWASP 测试指南](#owasp-testing-guides)
    - Web 安全测试指南（WSTG）
    - 移动安全测试指南（MSTG）
    - 固件安全测试方法论
- [渗透测试执行标准](#penetration-testing-execution-standard)
- [PCI 渗透测试指南](#pci-penetration-testing-guide)
    - [PCI DSS 渗透测试指南](#pci-dss-penetration-testing-guidance)
    - [PCI DSS 渗透测试要求](#pci-dss-penetration-testing-requirements)
- [渗透测试框架](#penetration-testing-framework)
- [信息安全测试和评估技术指南](#technical-guide-to-information-security-testing-and-assessment)
- [开源安全测试方法论手册](#open-source-security-testing-methodology-manual)
- [参考](#references)

## OWASP 测试指南

在技术安全测试执行方面，OWASP 测试指南是强烈推荐的。根据应用类型，测试指南分别列出用于 Web/云服务、移动应用（Android/iOS）或 IoT 固件。

- [OWASP Web 安全测试指南](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP 移动安全测试指南](https://owasp.org/www-project-mobile-security-testing-guide/)
- [OWASP 固件安全测试方法论](https://github.com/scriptingxss/owasp-fstm)

## 渗透测试执行标准

渗透测试执行标准（PTES）将渗透测试定义为 7 个阶段。特别地，PTES 技术指南给出了测试程序的手动建议，以及安全测试工具的建议。

- 预参与互动
- 情报收集
- 威胁建模
- 漏洞分析
- 利用
- 利用后
- 报告

[PTES 技术指南](http://www.pentest-standard.org/index.php/PTES_Technical_Guidelines)

## PCI 渗透测试指南

支付卡行业数据安全标准（PCI DSS）要求 11.3 定义了渗透测试。PCI 还定义了渗透测试指南。

### PCI DSS 渗透测试指南

PCI DSS 渗透测试指南提供以下方面的指导：

- 渗透测试组成部分
- 渗透测试人员的资质
- 渗透测试方法论
- 渗透测试报告指南

### PCI DSS 渗透测试要求

PCI DSS 要求参考支付卡行业数据安全标准（PCI DSS）要求 11.3

- 基于行业接受的方法
- 覆盖 CDE 和关键系统
- 包括外部和内部测试
- 测试验证范围减少
- 应用层测试
- 网络和操作系统的网络层测试

[PCI DSS 渗透测试指南](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)

## 渗透测试框架

渗透测试框架（PTF）提供了全面的手动渗透测试指南。它还列出了每个测试类别中安全测试工具的使用情况。渗透测试的主要领域包括：

- 网络足迹（侦察）
- 发现和探测
- 枚举
- 密码破解
- 漏洞评估
- AS/400 审计
- 蓝牙特定测试
- Cisco 特定测试
- Citrix 特定测试
- 网络骨干
- 服务器特定测试
- VoIP 安全
- 无线渗透
- 物理安全
- 最终报告 - 模板

[渗透测试框架](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)

## 信息安全测试和评估技术指南

NIST 发布的信息安全测试和评估技术指南（NIST 800-115），其中列出了以下一些评估技术。

- 审查技术
- 目标识别和分析技术
- 目标漏洞验证技术
- 安全评估规划
- 安全评估执行
- 测试后活动

更多详情见：[NIST 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)

## 开源安全测试方法论手册

开源安全测试方法论手册（OSSTMM）是一种测试物理位置、工作流程、人员安全测试、物理安全测试、无线安全测试、电信安全测试、数据网络安全测试和合规性的操作安全性的方法论。OSSTMM 可以作为 ISO 27001 的支持参考，而不是手动或技术应用渗透测试指南。

OSSTMM 包括以下关键部分：

- 安全分析
- 运营安全指标
- 信任分析
- 工作流程
- 人员安全测试
- 物理安全测试
- 无线安全测试
- 电信安全测试
- 数据网络安全测试
- 合规法规
- 使用 STAR（安全测试审计报告）报告

[开源安全测试方法论手册](https://www.isecom.org/OSSTMM.3.pdf)

## 参考

- [PCI 数据安全标准 - 渗透测试指南](https://www.pcisecuritystandards.org/documents/Penetration-Testing-Guidance-v1_1.pdf)
- [PTES 标准](http://www.pentest-standard.org/index.php/Main_Page)
- [开源安全测试方法论手册（OSSTMM）](https://www.isecom.org/research.html#content5-9d)
- [信息安全测试和评估技术指南 NIST SP 800-115](https://csrc.nist.gov/publications/detail/sp/800-115/final)
- [HIPAA 安全测试指南](https://www.hhs.gov/hipaa/for-professionals/security/guidance/cybersecurity/index.html)
- [渗透测试框架 0.59（存档）](https://web.archive.org/web/20130925044437/http://www.vulnerabilityassessment.co.uk/Penetration%20Test.html)
- [OWASP 移动安全测试指南](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Kali Linux](https://www.kali.org/)
- [信息补充：要求 11.3 渗透测试](https://www.pcisecuritystandards.org/pdfs/infosupp_11_3_penetration_testing.pdf)
