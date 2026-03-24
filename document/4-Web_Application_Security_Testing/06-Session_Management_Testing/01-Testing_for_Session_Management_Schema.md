# 测试会话管理架构

|ID          |
|------------|
|WSTG-SESS-01|

## 概述

任何基于Web应用的核心组件之一是控制和维护用户与应用交互状态的机制。为了避免在网站的每个页面或服务上进行持续认证，Web应用实现了各种机制来存储和验证凭证，有效期为预设的时间段。这些机制被称为会话管理（Session Management）。

在本测试中，测试人员需要检查Cookie和其他会话令牌是否以安全且不可预测的方式创建。攻击者如果能够预测并伪造弱Cookie，就可以轻松劫持合法用户的会话。

Cookie用于实现会话管理，在RFC 2965中有详细描述。简而言之，当用户访问需要跨多个请求跟踪用户操作和身份的应用时，服务器会生成一个或多个Cookie并发送给客户端。客户端将在后续所有连接中将Cookie发送回服务器，直到Cookie过期或被销毁。Cookie中存储的数据可以为服务器提供关于用户身份、已执行的操作、偏好设置等多方面的信息，从而为像HTTP这样的无状态协议提供状态管理。

一个典型的例子是在线购物车。在用户的整个会话过程中，应用必须跟踪其身份、资料、已选择的商品、数量、单价、折扣等。Cookie是存储和传递这些信息的高效方式（其他方法包括URL参数和隐藏字段）。

由于Cookie存储的数据的重要性，它们对应用的整体安全性至关重要。篡改Cookie可能导致劫持合法用户的会话、在活动会话中获得更高权限，以及以未授权方式影响应用的运行。

在本测试中，测试人员需要检查向客户端颁发的Cookie是否能抵抗旨在干扰合法用户会话和应用本身的各种攻击。总体目标是能够伪造一个被应用视为有效的Cookie，并提供某种未授权访问（会话劫持、权限提升等）。

通常，攻击模式的主要步骤如下：

- **Cookie收集**：收集足够数量的Cookie样本；
- **Cookie逆向工程**：分析Cookie生成算法；
- **Cookie操纵**：伪造有效Cookie以执行攻击。最后一步可能需要大量尝试，具体取决于Cookie的创建方式（Cookie暴力破解攻击）。

另一种攻击模式是溢出Cookie。从严格意义上讲，这种攻击具有不同的性质，因为测试人员并不是试图重新创建一个完全有效的Cookie。相反，目标是溢出内存区域，从而干扰应用的正确行为，并可能注入（和远程执行）恶意代码。

## 测试目标

- 为同一用户和不同用户收集会话令牌（如果可能）。
- 分析并确保存在足够的随机性以阻止会话伪造攻击。
- 修改未签名且包含可被操纵信息的Cookie。

## 如何测试

### 黑盒测试和示例

客户端与应用之间的所有交互应至少根据以下标准进行测试：

- 所有`Set-Cookie`指令是否标记为`Secure`？
- 是否有任何Cookie操作通过未加密传输进行？
- Cookie是否可以被强制通过未加密传输？
- 如果可以，应用如何维持安全性？
- 是否有任何持久性Cookie？
- 持久性Cookie使用什么`Expires`时间，它们是否合理？
- 预期为临时性的Cookie是否配置为如此？
- 使用了什么HTTP/1.1 `Cache-Control`设置来保护Cookie？
- 使用了什么HTTP/1.0 `Cache-Control`设置来保护Cookie？

#### Cookie收集

操纵Cookie的第一步是了解应用如何创建和管理Cookie。对于这项任务，测试人员必须尝试回答以下问题：

- 应用使用了多少个Cookie？

  浏览应用。记录Cookie创建的时间。列出收到的Cookie、设置它们的页面（带有set-cookie指令）、它们有效的域名、它们的值及其特性。

- 应用的哪些部分生成或修改Cookie？

  浏览应用，找出哪些Cookie保持不变，哪些被修改。什么事件会修改Cookie？

- 应用的哪些部分需要此Cookie才能被访问和使用？

  找出应用的哪些部分需要Cookie。访问一个页面，然后尝试在没有Cookie或使用修改后的值的情况下再次访问。尝试映射哪些Cookie在哪里使用。

  一个将每个Cookie映射到相应应用部分及相关信息的电子表格可以是本阶段的有价值输出。

#### 会话分析

会话令牌（Cookie、SessionID或隐藏字段）本身应从安全角度进行审查。应根据其随机性、唯一性、对统计和密码分析的抵抗能力以及信息泄露等方面进行测试。

- 令牌结构与信息泄露

  第一阶段是检查应用提供的Session ID的结构和内容。一个常见错误是在令牌中包含特定数据，而不是发布一个通用值并在服务器端引用真实数据。

  如果Session ID是明文的，结构和相关数据可能立即显而易见，例如`192.168.100.1:owaspuser:password:15:58`。

  如果令牌的部分或全部被编码或哈希，应与各种技术进行比较以检查是否有明显的混淆。例如，字符串`192.168.100.1:owaspuser:password:15:58`以十六进制、base64和MD5哈希表示：

  - 十六进制：`3139322E3136382E3130302E313A6F77617370757365723A70617373776F72643A31353A3538`
  - Base64：`MTkyLjE2OC4xMDAuMTpvd2FzcHVzZXI6cGFzc3dvcmQ6MTU6NTg=`
  - MD5：`01c2fc4f0a817afd8366689bd29dd40a`

  一旦识别出混淆类型，可能可以解码回原始数据。然而，在大多数情况下，这是不太可能的。即便如此，从消息格式中枚举编码可能是有用的。此外，如果可以推断出格式和混淆技术，可以设计自动暴力破解攻击。

  混合令牌可能包含IP地址或用户ID等信息以及编码部分，例如`owaspuser:192.168.100.1:a7656fafe94dae72b1e1487670148412`。

  分析了单个会话令牌后，应检查代表性样本。简单的分析应立即揭示任何明显的模式。例如，32位令牌可能包括16位静态数据和16位可变数据。这可能表明前16位代表用户的固定属性——例如用户名或IP地址。如果第二个16位块以规律速率递增，可能表示令牌生成中包含顺序甚至是基于时间的元素。参见示例。

  如果识别出令牌的静态元素，应收集更多样本，一次改变一个潜在输入元素。例如，通过不同用户账户或不同IP地址进行的登录尝试可能会在会话令牌的先前静态部分产生差异。

  在单个和多个Session ID结构测试期间应解决以下方面：

  - Session ID的哪些部分是静态的？
  - Session ID中存储了什么明文机密信息？例如用户名/UID、IP地址
  - 存储了什么易于解码的机密信息？
  - 从Session ID的结构中可以推断出什么信息？
  - 在相同登录条件下，Session ID的哪些部分是静态的？
  - Session ID整体或单独部分中存在什么明显的模式？

#### Session ID可预测性和随机性

应对Session ID的可变区域（如果有）进行分析，以确定是否存在任何可识别或可预测的模式。这些分析可以手动进行，也可以使用定制的或市售的统计或密码分析工具来推断Session ID内容中的任何模式。手动检查应包括对相同登录条件颁发的Session ID的比较——例如，相同的用户名、密码和IP地址。

时间也是一个重要因素，必须加以控制。应建立高数量的同时连接，以便在同一时间窗口内收集样本并保持该变量不变。即使50毫秒或更小的量化也可能太粗略，以这种方式采集的样本可能揭示否则会被遗漏的基于时间的组件。

应随着时间分析可变元素，以确定它们是否是递增的。如果是递增的，应调查与绝对时间或已用时间的模式。许多系统使用时间作为伪随机元素的种子。当模式看似随机时，应考虑时间或其他环境变化的单向哈希。通常，密码哈希的结果是十进制或十六进制数字，因此应该是可识别的。

在分析Session ID序列、模式或循环时，静态元素和客户端依赖项都应被视为对应用结构和功能的可能贡献元素。

- Session ID在本质上是否可证明是随机的？产生的值是否可以重现？
- 相同的输入条件在后续运行时是否产生相同的ID？
- Session ID是否可证明对统计或密码分析具有抵抗能力？
- Session ID的哪些元素与时间相关？
- Session ID的哪些部分是可预测的？
- 给定生成算法和先前ID的完整知识，是否可以推断出下一个ID？

#### Cookie逆向工程

现在测试人员已经枚举了Cookie并对其用途有了总体概念，是时候更深入地查看那些看起来有趣的Cookie了。测试人员对哪些Cookie感兴趣？Cookie为了提供安全的会话管理方法，必须结合多个特性，每个特性都旨在保护Cookie免受不同类别的攻击。

这些特性总结如下：

1. 不可预测性：Cookie必须包含一些难以猜测的数据。伪造有效Cookie越难，闯入合法用户会话就越难。如果攻击者能够猜到合法用户活动会话中使用的Cookie，他们就能够完全冒充该用户（会话劫持）。为了使Cookie不可预测，可以使用随机值或加密。
2. 防篡改性：Cookie必须抵抗恶意的修改尝试。如果测试人员收到类似`IsAdmin=No`的Cookie，修改它以获得管理权限是很容易的，除非应用执行双重检查（例如，向Cookie附加其值的加密哈希）
3. 过期时间：关键Cookie只在适当的时间内有效，之后必须从磁盘或内存中删除，以避免被重放的风险。这不适用于存储跨会话需要记住的非关键数据的Cookie（例如，网站外观）。
4. `Secure`标志：其值对会话完整性至关重要的Cookie应启用此标志，以便仅在加密通道上传输，以阻止窃听。

这里的方法是收集足够数量的Cookie实例并开始查找其值中的模式。"足够"的确切含义可能有所不同——如果Cookie生成方法非常容易破解，可能只需要少数样本；如果测试人员需要进行一些数学分析（例如，卡方检验、吸引子），则可能需要数千个样本。

特别注意应用的工作流程很重要，因为会话的状态会对收集的Cookie产生重大影响。在认证前收集的Cookie可能与认证后获得的Cookie非常不同。

另一个需要考虑的方面是时间。始终记录获取Cookie的确切时间，当时间可能在Cookie值中起作用时（服务器可能使用时间戳作为Cookie值的一部分）。记录的时间可以是本地时间或HTTP响应中包含的服务器时间戳（或两者）。

在分析收集的值时，测试人员应尝试找出所有可能影响Cookie值的变量，并一次一个地尝试改变它们。将相同Cookie的修改版本传递给服务器对于理解应用如何读取和处理Cookie非常有帮助。

本阶段应执行的检查示例包括：

- Cookie使用什么字符集？Cookie是否有数值？字母数字？十六进制？如果测试人员在Cookie中插入不属于预期字符集的字符，会发生什么？
- Cookie是否由携带不同信息的不同子部分组成？不同部分是如何分隔的？用什么分隔符？Cookie的某些部分可能具有较高的方差，其他部分可能是常量，其他部分可能只假设有限的值集。将Cookie分解为其基本组件是第一步也是基本步骤。

一个容易发现的结构化Cookie示例如下：

```text
ID=5a0acfc7ffeb919:CR=1:TM=1120514521:LM=1120514521:S=j3am5KzC4v01ba3q
```

此示例显示5个不同字段，携带不同类型的数据：

- ID – 十六进制
- CR – 小整数
- TM和LM – 大整数（有趣的是它们持有相同的值。值得看看修改其中一个会发生什么）
- S – 字母数字

即使没有使用分隔符，有足够的样本也有助于理解结构。

#### 暴力破解攻击

暴力破解攻击不可避免地来自与可预测性和随机性相关的问题。必须结合应用会话持续时间和超时来考虑Session ID内的方差。如果Session ID内的变化相对较小，且Session ID有效性较长，则成功暴力破解攻击的可能性会更高。

 长的Session ID（或具有较大方差的Session ID）和较短的 validity 时间会使暴力破解攻击更难成功。

- 对所有可能的Session ID进行暴力破解攻击需要多长时间？
- Session ID空间是否足够大以防止暴力破解？例如，与有效寿命相比，密钥长度是否足够？
- 使用不同Session ID进行连接尝试之间的延迟是否减轻了此攻击的风险？

### 灰盒测试和示例

如果测试人员可以访问会话管理架构实现，他们可以检查以下内容：

- 随机会话令牌

  颁发给客户端的Session ID或Cookie不应易于预测（不要使用基于可预测变量（如客户端IP地址）的线性算法）。鼓励使用256位密钥长度的加密算法（如AES）。

- 令牌长度

  Session ID至少为50个字符长度。

- 会话超时

  会话令牌应有定义的超时（取决于应用管理数据的关键性）

- Cookie配置：
    - 非持久性：仅RAM内存
    - 安全（仅在HTTPS通道上设置）：`Set-Cookie: cookie=data; path=/; domain=.aaa.it; secure`
    - [HttpOnly](https://owasp.org/www-community/HttpOnly)（脚本不可读）：`Set-Cookie: cookie=data; path=/; domain=.aaa.it; HttpOnly`

更多信息请参阅：[测试Cookie属性](02-Testing_for_Cookies_Attributes.md)

## 工具

- [Zed Attack Proxy Project (ZAP)](https://www.zaproxy.org) - 具有会话令牌分析机制。
- [Burp Sequencer](https://portswigger.net/burp/documentation/desktop/tools/sequencer)
- [YEHG's JHijack](https://github.com/yehgdotnet/JHijack)

## 参考资料

### 白皮书

- [RFC 2965 "HTTP State Management Mechanism"](https://tools.ietf.org/html/rfc2965)
- [RFC 1750 "Randomness Recommendations for Security"](https://www.ietf.org/rfc/rfc1750.txt)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis" (2001)](https://lcamtuf.coredump.cx/oldtcp/tcpseq.html)
- [Michal Zalewski: "Strange Attractors and TCP/IP Sequence Number Analysis - One Year Later" (2002)](https://lcamtuf.coredump.cx/newtcp/)
- [Correlation Coefficient](https://mathworld.wolfram.com/CorrelationCoefficient.html)
- [ENT](https://fourmilab.ch/random/)
- [DMA 2005-0614a - Global Hauri ViRobot Server cookie overflow](https://seclists.org/lists/fulldisclosure/2005/Jun/0188.html)
- [OWASP Code Review Guide](https://wiki.owasp.org/index.php/Category:OWASP_Code_Review_Project)
