# 漏洞命名方案

随着需要管理的 IT 资产数量不断增长，安全从业人员需要新的更强大的工具来执行自动化和大规模分析。多亏了软件，可以将注意力集中在更具创造性和智力挑战性的问题上。不幸的是，让漏洞评估工具、防病毒软件和入侵检测系统进行通信并不是一件容易的事。它导致了一些技术复杂性，需要一种标准化方式来识别每个已识别的软件缺陷、漏洞或配置问题。缺乏这种互操作能力可能导致安全评估期间的不一致性、报告混乱，以及额外的关联工作等问题，这些问题将造成资源和时间的重大浪费。

命名方案是一种系统化的方法论，用于识别每个漏洞，以促进清晰的识别和信息共享。这一目标通过为每个漏洞定义一个唯一的、结构化的、软件高效的名称来实现。有多种方案可用于促进这一工作，最常见的是：

- 通用平台枚举（`CPE`）
- 软件标识标签（`SWID`）
- 包 URL（`PURL`）

## 软件标识标签

软件标识标签（`SWID`）是国际标准化组织定义的标准，定义于 ISO/IEC 19770-2:2015。`SWID` 标签用于作为综合软件资产管理生命周期的一部分清晰标识每个软件。美国国家标准与技术研究院（NIST）推荐将此信息模式作为任何已开发或已安装软件的主要标识使用。从 `SWID` 可以生成其他模式，例如国家漏洞数据库（NVD）使用的 `CPE`，但反向过程是不可能的。

每个 `SWID` 标签以标准化的 XML 格式表示。`SWID` 标签由三组元素组成。第一个块由 7 个预定义元素组成，这是成为有效标签的必要条件。随后是一个可选块，提供 30 个可能的预定义元素，标签创建者可用其提供可靠和详细信息。最后，`Extended` 元素组为标签创建者提供了定义任何非预定义元素的机会，以准确描述所描述的软件。`SWID` 提供的高粒度不仅能够描述给定软件产品，还能描述其在软件生命周期中的特定状态。

### 示例

- _ACME Roadrunner Service Pack 1_ 补丁，由 ACME Corporation 为已安装的产品创建，产品由 `@tagId`：_com.acme.rms-ce-v4-1-5-0_ 标识：

```xml
<SoftwareIdentity
                  xmlns="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  name="ACME Roadrunner Service Pack 1"
                  tagId="com.acme.rms-ce-sp1-v1-0-0"
                  patch="true"
                  version="1.0.0">
  <Entity
          name="The ACME Corporation"
          regid="acme.com"
          role="tagCreator softwareCreator"/>
  <Link
        rel="patches"
        href="swid:com.acme.rms-ce-v4-1-5-0">
    ...
</SoftwareIdentity>
```

- 适用于 x86-64 架构的 Red Hat Enterprise Linux 8：

```xml
<SoftwareIdentity
                  xmlns="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="https://standards.iso.org/iso/19770/-2/2015/schema.xsd"
                  xml:lang="en-US"
                  name="Red Hat Enterprise Linux"
                  tagId="com.redhat.RHEL-8-x86_64"
                  tagVersion="1"
                  version="8"
                  versionScheme="multipartnumeric"
                  media="(OS:linux)">
```

## 通用平台枚举

通用平台枚举（`CPE`）是一种结构化命名方案，用于信息技术系统、软件和包，由 `NVD` 维护。通常与通用漏洞和暴露标识符（例如 `CVE-2017-0147`）结合使用。尽管被认为是已被 `SWID` 取代的已弃用方案，但 `CPE` 仍被多种安全解决方案广泛使用。

定义为由 `NVD` 提供的注册值字典。每个 `CPE` 代码可以定义为格式良好的名称或 URL。每个值必须遵循以下结构：

- _cpe-name_ = "cpe:" component-list
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition ":" lang
- _component-list_ = part ":" vendor ":" product ":" version ":" update ":" edition
- _component-list_ = part ":" vendor ":" product ":" version ":" update
- _component-list_ = part ":" vendor ":" product ":" version
- _component-list_ = part ":" vendor ":" product
- _component-list_ = part ":" vendor
- _component-list_ = part
- _component-list_ = empty
- _part_ = "h" / "o" / "a" = string
- _vendor_ = string
- _product_ = string
- _version_ = string
- _update_ = string
- _edition_ = string
- _lang_ LANGTAG / empty
- _string_ = *( unreserved / pct-encoded )
- _empty_ = ""
- _unreserved_ = ALPHA / DIGIT / "-" / "." / "_" / " ̃"
- _pct-encoded_ = "%" HEXDIG HEXDIG
- _ALPHA_ = %x41-5a / %x61-7a ; A-Z or a-z
- _DIGIT_ = %x30-39 ; 0-9
- _HEXDIG_ = DIGIT / "a" / "b" / "c" / "d" / "e" / "f"
- _LANGTAG_ = cf. [RFC5646]

### 示例

- Microsoft Internet Explorer 8.0.6001 Beta（任何版本）：`wfn:[part="a",vendor="microsoft",product="internet_explorer", version="8\.0\.6001",update="beta",edition=ANY]`，绑定到以下 URL：`cpe:/a:microsoft:internet_explorer:8.0.6001:beta`。
- 适用于 iPod Touch 80GB 的 Foo\Bar Big$Money Manager 2010 特别版：`wfn:[part="a",vendor="foo\\bar",product="big\$money_manager_2010", sw_edition="special",target_sw="ipod_touch",target_hw="80gb"]`，绑定到以下 URL：`cpe:/a:foo%5cbar:big%24money_manager_2010:::~~special~ipod_touch~80gb~`。

## 包 URL

包 URL 标准化了软件包元数据的表示方式，以便无论包属于哪个供应商、项目或生态系统，都可以普遍定位包。

PURL 是一个有效的 `RFC3986` ASCII 字符串，定义为由七个元素组成的 URL。每个元素由定义字符分隔，以便软件轻松操作。

`scheme:type/namespace/name@version?qualifiers#subpath`

每个组件的定义如下：

- _scheme_：URL 方案合规常量值"pkg"。（**必需**）
- _type_：包类型或包协议，如 maven、npm、nuget、gem、pypi 等。（**必需**）
- _namespace_：包前缀的类型特定值，例如其所有者名称、groupid 等。（可选）
- _name_：包的名称。（**必需**）
- _version_：包版本。（可选）
- _qualifiers_：包的额外限定数据，如操作系统、架构、发行版等。（可选）
- _subpath_：包内相对于包根的额外子路径。（可选）

### 示例

- Curl 软件，打包为适用于 Debian Jessie 的 `.deb` 包，面向 i386 架构：`pkg:deb/debian/curl@7.50.3-1?arch=i386&distro=jessie`
- Apache Cassandra 的 Docker 镜像，使用 SHA256 哈希 244fd47e07d1004f0aed9c 签名：`pkg:docker/cassandra@sha256:244fd47e07d1004f0aed9c`

## 建议使用

| 用途 | 建议 |
|---|---|
| 客户端或服务器应用 | CPE 或 SWID |
| 容器 | PURL 或 SWID |
| 固件 | CPE 或 SWID* |
| 库或框架（包） | PURL |
| 库或框架（非包） | SWID |
| 操作系统 | CPE 或 SWID |
| 操作系统包 | PURL 或 SWID |

> 注意：由于 `CPE` 已弃用，行业建议似乎是新项目在需要在这两种方法之间做出决定时实施 `SWID`。尽管众所周知 `CPE` 是当前活跃项目和解决方案中广泛使用的命名模式。

## 参考资料

- [NISTIR 8060 - 创建互操作软件标识（SWID）标签的指南（PDF）](https://nvlpubs.nist.gov/nistpubs/ir/2016/NIST.IR.8060.pdf)
- [NISTIR 8085 - 从软件标识（SWID）标签形成通用平台枚举（CPE）名称](https://csrc.nist.gov/CSRC/media/Publications/nistir/8085/draft/documents/nistir_8085_draft.pdf)
- [ISO/IEC 19770-2:2015 - 信息技术—软件资产管理—第2部分：软件标识标签](https://www.iso.org/standard/65666.html)
- [官方通用平台枚举（CPE）词典](https://nvd.nist.gov/products/cpe)
- [通用平台枚举：词典规范版本 2.3](https://csrc.nist.gov/publications/detail/nistir/7697/final)
- [PURL 规范](https://github.com/package-url/purl-spec)

### 已知实现

- [packageurl-go](https://github.com/package-url/packageurl-go)
- [packageurl-dotnet](https://github.com/package-url/packageurl-dotnet)
- [packageurl-java](https://github.com/package-url/packageurl-java), [package-url-java](https://github.com/sonatype/package-url-java)
- [packageurl-python](https://github.com/package-url/packageurl-python)
- [packageurl-rust](https://github.com/package-url/packageurl.rs)
- [packageurl-js](https://github.com/package-url/packageurl-js)
