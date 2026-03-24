# 测试批量赋值

|ID          |
|------------|
|WSTG-INPV-20|

## 概述

现代Web应用程序非常基于框架。许多这些Web应用程序框架允许自动绑定用户输入（以HTTP请求参数的形式）到内部对象。这通常称为自动绑定。
此功能有时可能被利用来访问从未打算从外部修改的字段，导致权限提升、数据篡改、绕过安全机制等。
在这种情况下，存在批量赋值漏洞。

敏感属性的示例：

- **与权限相关的属性**：应仅由特权用户设置（如`is_admin`、`role`、`approved`）。
- **流程相关属性**：应仅在流程完成后在内部设置（如`balance`、`status`、`email_verified`）
- **内部属性**：应仅由应用程序在内部设置（如`created_at`、`updated_at`）

## 测试目标

- 识别修改对象的请求
- 评估是否可能修改从未打算从外部修改的字段

## 如何测试

以下是一个可以帮助说明此问题的经典示例。

假设一个具有类似于以下的`User`对象的Java Web应用程序：

```java
public class User {
   private String username;
   private String password;
   private String email;
   private boolean isAdmin;

   //Getters & Setters
}
```

为了创建新`User`，Web应用程序实现以下视图：

```html
<form action="/createUser" method="POST">
     <input name="username" type="text">
     <input name="password" type="text">
     <input name="email" text="text">
     <input type="submit" value="Create">
</form>
```

处理创建请求的控制器（Spring提供带有`User`模型的自动绑定）：

```java
@RequestMapping(value = "/createUser", method = RequestMethod.POST)
public String createUser(User user) {
   userService.add(user);
   return "successPage";
}
```

当提交表单时，浏览器生成以下请求：

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test
```

但是，由于自动绑定，攻击者可以向请求添加`isAdmin`参数，控制器将自动绑定到模型。

```http
POST /createUser
[...]
username=bob&password=supersecretpassword&email=bob@domain.test&isAdmin=true
```

然后，用户创建时将`isAdmin`属性设置为`true`，授予他们对应用程序的管理权限。

### 黑盒测试

#### 检测处理程序

为了确定应用程序的哪个部分容易受到批量赋值，枚举应用程序中所有接受用户内容并可能映射到模型的部分。这包括所有HTTP请求（可能是GET、POST和PUT），这些请求似乎允许对后端执行创建或更新操作。
潜在批量赋值的最简单的指标之一是输入参数名称中括号语法的存在，例如：

```html
<input name="user[name]" type="text">
```

当遇到此类模式时，尝试添加与不存在的属性相关的输入（例如`user[nonexistingattribute]`）并分析响应/行为。
如果应用程序未实施任何控制（如允许字段列表），则可能因应用程序找不到与对象关联的属性而响应错误（例如500）。更有趣的是，这些错误有时有助于发现属性名称和值数据类型所需的属性名称，从而在没有源代码访问权限的情况下利用此问题。

#### 识别敏感字段

因为在黑盒测试中测试人员无法看到源代码，有必要使用其他方式收集与对象关联的属性信息。
分析从后端收到的响应，特别注意：

- HTML页面源代码
- 自定义JavaScript代码
- API响应

例如，通常可以利用返回对象详情的处理程序获取关联字段的线索。
例如，假设一个返回用户配置文件的处理程序（例如`GET /profile`），这可能包括与用户相关的更多属性（在此示例中，`isAdmin`属性看起来特别有趣）。

```json
{"_id":12345,"username":"bob","age":38,"email":"bob@domain.test","isAdmin":false}
```

然后，尝试利用允许修改或创建用户的处理程序，添加`isAdmin`属性并设置为`true`。

另一种方法是使用词汇表来尝试枚举所有潜在属性。然后可以自动执行枚举（例如通过wfuzz、Burp Intruder、ZAP模糊器等）。sqlmap工具包括一个[common-columns.txt](https://github.com/sqlmapproject/sqlmap/blob/master/data/txt/common-columns.txt)词汇表，可用于识别潜在的敏感属性。
常见有趣属性名称的小示例如下：

- `is_admin`
- `is_administrator`
- `isAdmin`
- `isAdministrator`
- `admin`
- `administrator`
- `role`

当有多个角色可用时，尝试比较不同用户级别发出的请求（特别注意特权角色）。例如，如果在由管理用户发出的请求中包含额外参数，作为低特权/匿名用户尝试这些参数。

#### 检查影响

批量赋值的影响可能因上下文而异，因此对于上一阶段尝试的每个测试输入，分析结果并确定它是否代表了 对Web应用程序安全有实际影响的漏洞。
例如，修改对象的`id`可能导致应用程序拒绝服务或权限提升。另一个示例与修改用户角色/状态的能力有关（如`role`或`isAdmin`），导致垂直权限提升。

### 灰盒测试

当使用灰盒测试方法进行分析时，可以遵循相同的方法来验证此问题。但是，关于应用程序的更多知识允许更容易地识别容易受到批量赋值漏洞的框架和处理程序。
特别是，当源代码可用时，可以更轻松准确地搜索输入向量。在源代码审查期间，使用简单工具（如grep命令）搜索应用程序代码中的一个或多个常见模式。
访问DB模式或源代码也允许轻松识别敏感字段。

#### Java

Spring MVC允许自动将用户输入绑定到对象。识别处理状态更改请求的控制器（例如找到`@RequestMapping`的出现），然后验证是否存在控制（在控制器上或涉及的模型上）。批量赋值利用的限制示例为：

- 通过`DataBinder`类的`setAllowedFields`方法列出可绑定字段（例如`binder.setAllowedFields(["username","password","email"])`）
- 通过`DataBinder`类的`setDisallowedFields`方法列出不可绑定字段（例如`binder.setDisallowedFields(["isAdmin"])`）

还要注意`@ModelAttribute`注解的使用，它允许指定不同的名称/键。

#### PHP

Laravel Eloquent ORM提供`create`方法，允许自动分配属性。但是，Eloquent ORM的最新版本在默认情况下提供针对批量赋值漏洞的保护，需要明确指定可以自动分配的允许属性，通过`$fillable`数组，或需要保护的属性（非可绑定），通过`$guarded`数组。因此，通过分析模型（扩展`Model`类的类），可以识别允许或拒绝的属性，从而指出潜在的漏洞。

#### .NET

ASP.NET中的模型绑定自动将用户输入绑定到对象属性。这也适用于复杂类型，如果属性名称与输入匹配，它将自动转换输入数据到属性。
识别控制器，然后验证是否存在控制（在控制器内部或涉及的模型上）。批量赋值利用的限制示例为：

- 声明为`ReadOnly`的字段
- 通过`Bind`属性可绑定字段列表（例如`[Bind(Include = "FirstName, LastName")] Student std`），通过`includeProperties`（例如`includeProperties: new[] { "FirstName, LastName" }`）或通过`TryUpdateModel`
- 通过`Bind`属性不可绑定字段列表（例如`[Bind(Exclude = "Status")] Student std`）或通过`excludeProperties`（例如`excludeProperties: new[] { "Status" }`）

## 修复

使用框架提供的内置功能来定义可绑定和不可绑定字段。基于允许字段（可绑定）的方法（其中仅明确指定应由用户更新的属性）是首选。
防止此问题的架构方法是使用*数据传输对象*（DTO）模式来避免直接绑定。DTO应仅包括打算由用户编辑的字段。

## 参考资料

- [OWASP：API安全](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa6-mass-assignment.md)
- [OWASP：速查表系列](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
- [CWE-915：动态确定的对象属性的不当控制修改](https://cwe.mitre.org/data/definitions/915.html)
