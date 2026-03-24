# REST 评估备忘单

## 关于 RESTful Web 服务

Web 服务是用于机器对机器通信的 Web 技术实现。因此，它们用于应用间通信、Web 2.0 和混搭，以及桌面和移动应用程序调用服务器。

RESTful Web 服务（通常简称为 REST）是基于 RESTful 设计模式的 Web 服务的轻量级变体。在实践中，RESTful Web 服务利用与其他 Web 服务技术（如 SOAP 利用复杂协议）形成对比的类似于常规 HTTP 调用的 HTTP 请求。

## RESTful Web 服务的关键相关属性

- 使用 HTTP 方法（`GET`、`POST`、`PUT` 和 `DELETE`）作为请求操作的主要动词。
- 非标准参数规范：
    - 作为 URL 的一部分。
    - 在头中。
- 使用 JSON 或 XML 的结构化参数和响应，用于参数值、请求体或响应体中的通信。这些是传达机器有用信息所必需的。
- 自定义认证和会话管理，通常利用自定义安全令牌：这是必需的，因为机器对机器通信不允许登录序列。
- 缺乏正式文档。Sun Microsystems 提交了一个名为 WADL 的[描述 RESTful Web 服务的提议标准](https://www.w3.org/Submission/wadl/)，但从未被正式采用。

## RESTful Web 服务安全测试的挑战

- 检查应用程序不会显示攻击面，即 RESTful Web 服务使用的 URL 和参数结构。原因如下：
    - 没有应用程序利用服务暴露的所有可用功能和参数。
    - 那些被使用的通常由客户端代码动态激活，而不是页面中的链接。
    - 客户端应用程序通常不是 Web 应用程序，不允许检查激活链接甚至相关代码。
- 参数非标准使得很难确定什么是 URL 或常量头的组成部分，什么是值得[模糊测试](https://owasp.org/www-community/Fuzzing)的参数。
- 作为机器接口，使用的参数数量可能非常大，例如 JSON 结构可能包含数十个参数。[模糊测试](https://owasp.org/www-community/Fuzzing)每一个都会显著延长测试所需的时间。
- 自定义认证机制需要逆向工程，使流行工具无用，因为它们无法跟踪登录会话。

## 如何对 RESTful Web 服务进行渗透测试

通过文档确定攻击面——如果允许某种级别的白盒测试并且您可以获得有关服务的信息，RESTful 渗透测试可能会更好。

此信息将确保更全面地覆盖攻击面。需要查找的信息：

- 正式服务描述——虽然对于其他类型的 Web 服务（如 SOAP）通常有正式描述（通常是 WSDL），但这种情况在 REST 中很少见。也就是说，WSDL 2.0 或 WADL 可以描述 REST，有时也会被使用。
- 使用服务的开发人员指南可能不那么详细，但通常会找到，甚至可能被认为 *黑盒*。
- 应用程序源代码或配置——在许多框架中，包括 dotNet，REST 服务定义可能很容易从配置文件而不是从代码中获得。

使用[代理](https://www.zaproxy.org/)收集完整请求——虽然这始终是渗透测试的重要步骤，但对于基于 REST 的应用程序更为重要，因为应用程序 UI 可能不会给出实际攻击面的线索。

请注意，代理必须能够收集完整请求，而不仅仅是 URL，因为 REST 服务不仅仅利用 GET 参数。

分析收集的请求以确定攻击面：

- 寻找非标准参数：
    - 寻找异常 HTTP 头——这些通常是头基于的参数。
    - 确定 URL 段是否在 URL 中有重复模式。这种模式可以包括日期、数字或类似 ID 的字符串，并指示 URL 段是 URL 嵌入参数。
        - 例如：`https://server/srv/2013-10-21/use.php`
    - 寻找结构化参数值——这些可能是 JSON、XML 或非标准结构。
    - 如果 URL 的最后一个元素没有扩展名，它可能是参数。这尤其正确（如果应用程序技术通常使用扩展名）或之前的段确实有扩展名。
        - 例如：`https://server/svc/Grid.asmx/GetRelatedListItems`
    - 寻找高度变化的 URL 段——具有多个值的单个 URL 段可能是参数而不是物理目录。
        - 例如，如果 URL `https://server/src/XXXX/page` 对 `XXXX` 的值重复数百次，则 `XXXX` 很可能是参数。

验证非标准参数：在某些情况下（但不是全部），将怀疑为参数的 URL 段的值设置为预期无效的值有助于确定它是路径元素还是参数。如果是路径元素，Web 服务器将返回 *404* 消息，而对于参数的无效值，答案将是应用级别消息，因为该值在 Web 服务器级别是合法的。

分析收集的请求以优化[模糊测试](https://owasp.org/www-community/Fuzzing)——在识别要模糊测试的潜在参数后，分析每个参数的收集值以确定：

- 有效与无效值，以便[模糊测试](https://owasp.org/www-community/Fuzzing)可以专注于边际无效值。
    - 例如，为发现总是正整数的值发送 *0*。
- 允许模糊超出可能分配给当前用户的范围的序列。

最后，当[模糊测试](https://owasp.org/www-community/Fuzzing)时，不要忘记模拟所使用的认证机制。

## 相关资源

- [REST 安全备忘单](https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html) - 这个备忘单的另一面
- [RESTful 服务，Web 安全盲点](https://www.youtube.com/watch?v=pWq4qGLAZHI) - 一个视频演示，阐述了这个备忘单的大部分主题
