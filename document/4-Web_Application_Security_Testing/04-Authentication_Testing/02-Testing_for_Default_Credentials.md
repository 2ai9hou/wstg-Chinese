# 测试默认凭据

|ID          |
|------------|
|WSTG-ATHN-02|

## 概述

许多 Web 应用程序和硬件设备为内置管理员账户设置了默认密码。尽管在某些情况下这些密码是随机生成的，但它们通常是静态的，这意味着攻击者可以很容易地猜测或获取这些密码。

此外，当在应用程序中创建新用户时，这些用户可能具有预定义的密码。这些密码可能是由应用程序自动生成的，也可能是由工作人员手动创建的。在这两种情况下，如果这些密码不是以安全的方式生成的，攻击者就有可能猜到这些密码。

## 测试目标

- 确定应用程序是否存在使用默认密码的用户账户。
- 审查新用户账户是否使用弱密码或可预测的密码创建。

## 如何测试

### 测试供应商默认凭据

识别默认密码的第一步是确定正在使用的软件。这在指南的[信息收集](../01-Information_Gathering/README.md)部分有详细说明。

一旦识别了软件，尝试查找它是否使用默认密码，如果有，密码是什么。这应包括：

- 搜索"【软件名称】默认密码"。
- 查阅手册或供应商文档。
- 检查常见默认密码数据库，如 [CIRT.net](https://cirt.net/passwords)、[SecLists 默认密码](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials) 或 [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)。
- 检查应用程序源代码（如果可用）。
- 在虚拟机上安装应用程序并进行检查。
- 检查物理设备上的标签（通常存在于网络设备上）。

如果找不到默认密码，尝试常见选项，例如：

- "admin"、"password"、"12345"或其他[常见默认密码](https://github.com/nixawk/fuzzdb/blob/master/bruteforce/passwds/default_devices_users%2Bpasswords.txt)。
- 空密码或空白密码。
- 设备的序列号或 MAC 地址。

如果用户名未知，有多种枚举用户的方法，详见[测试账户枚举](../03-Identity_Management_Testing/04-Testing_for_Account_Enumeration_and_Guessable_User_Account.md)指南。或者，尝试常见选项，如 "admin"、"root" 或 "system"。

### 测试组织默认密码

当组织内的工作人员为新账户手动创建密码时，他们可能以可预测的方式创建密码。这通常包括：

- 一个共同的简单密码，如 "Password1"。
- 组织特定的信息，如组织名称或地址。
- 遵循简单模式的密码，如 "Monday123"（如果账户是在星期一创建的）。

这类密码从黑盒测试的角度很难识别，除非能够成功猜测或暴力破解。但是，在进行灰盒测试或白盒测试时很容易发现。

### 测试应用程序生成的默认密码

如果应用程序自动为新用户账户生成密码，这些密码可能也是可预测的。为了测试这些，创建多个具有相似信息的账户在同一时间，比较分配给它们的密码。

密码可能基于：

- 账户间共享的单个静态字符串。
- 账户详情的哈希或混淆部分，如 `md5($username)`。
- 基于时间的算法。
- 弱伪随机数生成器（PRNG）。

这类问题从黑盒测试的角度通常很难识别。

## 工具

- [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder)
- [THC Hydra](https://github.com/vanhauser-thc/thc-hydra)
- [Nikto 2](https://www.cirt.net/nikto2)
- [Nuclei](https://github.com/projectdiscovery/nuclei)
    - [默认登录 - Nuclei 模板](https://github.com/projectdiscovery/nuclei-templates/tree/6b26c63d8f63b2a812a478f14c4c098b485d54b4/http/default-logins)

## 参考资料

- [CIRT](https://cirt.net/passwords)
- [SecLists 默认密码](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials)
- [DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet/blob/main/DefaultCreds-Cheat-Sheet.csv)
