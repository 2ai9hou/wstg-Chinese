# 利用开发工具

本附录概述了使用浏览器开发者工具功能的各种细节，以帮助安全测试活动。

显然，浏览器内置功能不能替代：DAST（动态应用安全测试）工具、SAST（静态应用安全测试）工具或测试人员的经验，但是，它可以用于某些测试活动和报告生产相关任务。

## 访问开发工具

打开开发工具可以通过多种方式实现。

1. 通过键盘快捷键 `F12`。
2. 在 Windows 上通过键盘快捷键 `ctrl` + `shift` + `i`。
3. 在 Mac 上通过键盘快捷键 `cmd` + `option` + `i`。
4. 通过网页右键上下文菜单，然后在 Google Chrome 中选择 `检查`。
5. 通过网页右键上下文菜单，然后在 Mozilla Firefox 中选择 `检查元素`。
6. 通过 Google Chrome 中的三点"肉丸"菜单，然后选择 `更多工具` 然后选择 `开发者工具`。
7. 通过 Mozilla Firefox 中的三线"汉堡"（或"煎饼"）菜单，然后选择 `Web 开发者` 然后选择 `切换工具`。

> 注意：以下大部分说明假设开发工具已经打开或处于活动状态。

## 功能

| 功能 | Chrome* | Firefox | Safari |
|-----------------------|:-------:|:-------:|:------:|
| User-Agent 切换 | Y | Y | Y |
| 编辑/重发请求 | Y | Y | N |
| Cookie 编辑 | Y | Y | N |
| 本地存储编辑 | Y | Y | N |
| 禁用 CSS | Y | Y | Y |
| 禁用 JavaScript | Y | Y | Y |
| 查看 HTTP 头 | Y | Y | Y |
| 截图 | Y | Y | N |
| 离线模式 | Y | Y | N |
| 编码和解码 | Y | Y | Y |
| 响应式设计模式 | Y | Y | Y |

`*` 适用于 Google Chrome 的任何内容都应适用于所有基于 Chromium 的应用程序。（包括微软在 2019/2020 年左右将 Edge 更名。）

## User-Agent 切换

### 相关测试

- [测试浏览器缓存弱点](../4-Web_Application_Security_Testing/04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)

### Google Chrome

1. 点击开发者工具窗格右侧的三点"肉丸"菜单，选择 `更多工具` 然后选择 `网络条件`。
2. 取消选中"自动选择"复选框。
3. 从下拉菜单中选择 user agent 或输入自定义 user agent

![Google Chrome 中 User-Agent 选择下拉菜单](images/f_chrome_devtools_ua_switch.png)\
*图 6.F-1：Google Chrome 开发者工具 User-Agent 切换功能*

### Mozilla Firefox

1. 导航到 Firefox 的 `about:config` 页面并点击 `我接受风险！`。
2. 在搜索字段中输入 `general.useragent.override`。
3. 查找 `general.useragent.override`，如果您看不到此首选项，请查找显示单选按钮组 `Boolean、Number、String` 的选项，选择 `String`，然后点击 `about:config` 页面上的加号 `添加` 按钮。
4. 将 `general.useragent.override` 的值设置为您所需的任何 [User-Agent](https://developers.whatismybrowser.com/useragents/explore/)。

![Mozilla Firefox 中 User-Agent 配置首选项](images/f_firefox_ua_switch.png)\
*图 6.F-2：Mozilla Firefox User-Agent 切换功能*

之后，点击 `general.useragent.override` 首选项右侧的垃圾箱 `删除` 按钮以删除覆盖并切换回默认 user agent。

## 编辑/重发请求

### 相关测试

- [认证测试](../4-Web_Application_Security_Testing/04-Authentication_Testing/README.md)
- [授权测试](../4-Web_Application_Security_Testing/05-Authorization_Testing/README.md)
- [会话管理测试](../4-Web_Application_Security_Testing/06-Session_Management_Testing/README.md)
- [输入验证测试](../4-Web_Application_Security_Testing/07-Input_Validation_Testing/README.md)
- [业务逻辑测试](../4-Web_Application_Security_Testing/10-Business_Logic_Testing/README.md)

### Mozilla Firefox

1. 选择 `网络` 选项卡。
2. 在 Web 应用中执行任何操作。
3. 右键单击列表中的 HTTP 请求并选择 `编辑和重发`。
4. 进行所需的修改并点击 `发送` 按钮。
5. 右键单击修改后的请求并选择 `在新标签页中打开`。

### Google Chrome

1. 选择 `网络` 选项卡。
2. 在 Web 应用中执行任何操作。
3. 右键单击列表中的 HTTP 请求并选择 `复制 > 复制为 fetch`。
4. 将提供的 JavaScript 代码粘贴到 `控制台` 选项卡中。
5. 进行任何需要的修改，然后按回车发送请求。

## Cookie 编辑

### 相关测试

- [认证测试](../4-Web_Application_Security_Testing/04-Authentication_Testing/README.md)
- [授权测试](../4-Web_Application_Security_Testing/05-Authorization_Testing/README.md)
- [会话管理测试](../4-Web_Application_Security_Testing/06-Session_Management_Testing/README.md)
- [测试 Cookie 属性](../4-Web_Application_Security_Testing/06-Session_Management_Testing/02-Testing_for_Cookies_Attributes.md)

### Google Chrome

1. 点击 `应用` 选项卡。
2. 在 `存储` 标题下展开 `Cookies`。
3. 选择相关域名。
4. 双击 `值` 列以编辑任何 cookie 值。

> 注意：选择后，可以按 `delete` 键或从右键上下文菜单中删除 cookie。

### Mozilla Firefox

1. 点击 `存储` 选项卡。
2. 展开 `Cookies` 部分。
3. 选择相关域名。
4. 双击 `值` 列以编辑任何 cookie 值。

> 注意：选择后，可以按 `delete` 键或从右键上下文菜单中的各种选项中删除 cookie。

![Mozilla Firefox 中 Cookie 编辑功能](images/f_firefox_cookie_edit.png)\
*图 6.F-3：Mozilla Firefox Cookie 编辑功能*

## 本地存储编辑

### 相关测试

- [测试浏览器存储](../4-Web_Application_Security_Testing/11-Client-side_Testing/12-Testing_Browser_Storage.md)

### Google Chrome

1. 点击 `应用` 选项卡。
2. 在 `存储` 标题下展开 `本地存储`。
3. 选择相关域名。
4. 双击 `值` 列以编辑任何 cookie 值。
5. 双击适用的单元格以编辑 `键` 或 `值`。

> 注意：编辑 `会话存储` 或 `索引 DB` 遵循基本相同的步骤。
>
> 注意：可以通过右键上下文菜单添加或删除项目。

### Mozilla Firefox

1. 点击 `存储` 选项卡。
2. 展开 `本地存储` 部分。
3. 选择相关域名。
4. 双击适用的单元格以编辑 `键` 或 `值`。

> 注意：编辑 `会话存储` 或 `索引 DB` 遵循基本相同的步骤。
>
> 注意：可以通过右键上下文菜单添加或删除项目。

## 禁用 CSS

### 相关测试

- [测试客户端资源操作](../4-Web_Application_Security_Testing/11-Client-side_Testing/06-Testing_for_Client-side_Resource_Manipulation.md)

### 通用

所有主要浏览器都支持利用开发者工具控制台和 JavaScript 功能操作 CSS：

- 移除所有外部样式表：`$('style,link[rel="stylesheet"]').remove();`
- 移除所有内部样式表：`$('style').remove();`
- 移除所有内联样式：`Array.prototype.forEach.call(document.querySelectorAll('*'),function(el){el.removeAttribute('style');});`
- 从 head 标签中移除所有内容：`$('head').remove();`

## 禁用 JavaScript

### Google Chrome

1. 点击 Web 开发者工具栏右侧的三点"肉丸"菜单，然后点击 `设置`。
2. 在 `偏好设置` 选项卡上，在 `调试器` 部分下，选中 `禁用 JavaScript` 复选框。

### Mozilla Firefox

1. 在开发者工具 `调试器` 选项卡上，点击开发者工具栏右上角的设置齿轮按钮。
2. 从下拉菜单中选择 `禁用 JavaScript`（这是一个启用/禁用菜单项；当 JavaScript 被禁用时，菜单项会有一个复选标记）。

## 查看 HTTP 头

### 相关测试

- [信息收集](../4-Web_Application_Security_Testing/01-Information_Gathering/README.md)

### Google Chrome

1. 在开发者工具的 `网络` 选项卡上选择任何 URL 或请求。
2. 在右下窗格中选择 `头` 选项卡。

![Google Chrome 中的头视图](images/f_chrome_devtools_headers.png)\
*图 6.F-4：Google Chrome 头视图*

### Mozilla Firefox

1. 在开发者工具的 `网络` 选项卡上选择任何 URL 或请求。
2. 在右下窗格中选择 `头` 选项卡。

![Mozilla Firefox 中的头视图](images/f_firefox_devtools_headers.png)\
*图 6.F-5：Mozilla Firefox 头视图*

## 截图

### 相关测试

- [报告](../5-Reporting/README.md)

### Google Chrome

1. 点击 `切换设备工具栏` 按钮或按 `ctrl` + `shift` + `m`。
2. 点击设备工具栏中的三点"肉丸"菜单。
3. 选择 `截取截图` 或 `截取整页截图`。

### Mozilla Firefox

1. 点击地址栏中的三点 `省略号` 按钮。
2. 选择 `截取屏幕截图`。
3. 选择 `保存整页` 或 `保存可见内容` 选项。

## 离线模式

### Google Chrome

1. 导航到 `网络` 选项卡。
2. 在 `节流` 下拉菜单中选择 `离线`。

![Google Chrome 中的离线选项](images/f_chrome_devtools_offline.png)\
*图 6.F-6：Google Chrome 离线选项*

### Mozilla Firefox

1. 从三线"汉堡"（或"煎饼"）菜单中选择 `Web 开发者` 然后选择 `离线工作`。

![Mozilla Firefox 中的离线选项](images/f_firefox_devtools_offline.png)\
*图 6.F-7：Mozilla Firefox 离线选项*

## 编码和解码

### 相关测试

- 许多（也许是大多数）类型的 [Web 应用安全测试](../4-Web_Application_Security_Testing/README.md) 可以从各种类型的编码中受益。

### 通用

所有主要浏览器都支持利用开发者工具控制台和 JavaScript 功能以各种方式对字符串进行编码和解码：

- base64 编码：`btoa("string-to-encode")` 和 base64 解码：`atob("string-to-decode")` - 用于将字符串编码为 base64 和从 base64 解码字符串的内置 JavaScript 函数。
- URL 编码：`encodeURIComponent("string-to-encode")` 和 URL 解码：`decodeURIComponent("string-to-decode")` - 对将用作 URL 一部分的用户提供的输入进行编码和解码，它对 URL 中具有特殊含义的所有字符进行编码，包括保留字符。
- URI 编码：`encodeURI()` 和 URI 解码：`decodeURI()` 是用于对完整 URI（如查询参数、路径段或片段）进行编码和解码的函数，包括特殊字符，但不包括保留字符，如 `：/?#[]@!$'()*+,;=` 它们在 URL 中具有特殊含义。

## 响应式设计模式

### 相关测试

- [测试浏览器缓存弱点](../4-Web_Application_Security_Testing/04-Authentication_Testing/06-Testing_for_Browser_Cache_Weaknesses.md)
- [测试替代通道中较弱的认证](../4-Web_Application_Security_Testing/04-Authentication_Testing/10-Testing_for_Weaker_Authentication_in_Alternative_Channel.md)
- [测试点击劫持](../4-Web_Application_Security_Testing/11-Client-side_Testing/09-Testing_for_Clickjacking.md)

### Google Chrome

1. 点击 `切换设备工具栏` 按钮或按 `ctrl` + `shift` + `m`。

![Google Chrome 中的响应式设计模式](images/f_chrome_responsive_design_mode.png)\
*图 6.F-8：Google Chrome 响应式设计模式*

### Mozilla Firefox

1. 点击 `响应式设计模式` 按钮或按 `ctrl` + `shift` + `m`。

![Mozilla Firefox 中的响应式设计模式](images/f_firefox_responsive_design_mode.png)\
*图 6.F-9：Mozilla Firefox 响应式设计模式*

## 参考资料

- [使用浏览器进行 Web 应用安全测试](https://getmantra.com/web-app-security-testing-with-browsers/)
- [Black Hills Information Security - 网络广播：免费工具！如何在 Web 应用渗透测试中使用开发者工具和 JavaScript](https://www.blackhillsinfosec.com/webcast-free-tools-how-to-use-developer-tools-and-javascript-in-webapp-pentests/)
- [Greg Malcolm - Chrome 开发者工具：突袭军械库](https://github.com/gregmalcolm/wacky-wandas-wicked-weapons-frontend/blob/fix-it/README.md)
- [User-Agent 字符串列表](https://techblog.willshouse.com/2012/01/03/most-common-user-agents/)
