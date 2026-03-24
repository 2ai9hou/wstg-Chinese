# 映射应用执行路径

|ID          |
|------------|
|WSTG-INFO-07|

## 摘要

在开始安全测试之前，了解应用的结构至关重要。没有对应用布局的彻底理解，就不可能进行全面的测试。

## 测试目标

- 绘制目标应用并理解主要工作流程。

## 如何测试

在黑盒测试中，测试整个代码库是非常困难的。这不仅是因为测试人员看不到通过应用的代码路径，而且测试所有代码路径也非常耗时。协调这一点的一种方法是记录已发现和测试的代码路径。

有几种方法可用于测试和测量代码覆盖率：

- **路径** - 测试通过应用的每个路径，包括每个决策路径的组合和边界值分析测试。虽然这种方法提供了彻底的覆盖，但每个决策分支的可测试路径数量呈指数增长。
- **数据流（或污点分析）** - 通过外部交互（通常为用户）测试变量赋值。专注于映射整个应用中的数据流、转换和使用。
- **竞态** - 测试操作相同数据的多个并发应用实例。

方法的选择以及每种方法的使用程度应与应用所有者协商。此外，可以采用更简单的方法。例如，测试人员可以询问应用所有者关于他们特别关注的特定功能或代码段，并讨论如何访问这些代码段。

为了向应用所有者展示代码覆盖率，测试人员可以首先在电子表格中记录从爬取应用中发现的所有链接（手动或自动）。测试人员可以更仔细地查看应用中的决策点，并研究发现了多少重要代码路径。然后应在电子表格中记录这些内容，包括 URL、路径的文字和屏幕截图描述。

### 自动爬虫

自动爬虫是一种用于自动发现特定站点上新资源（URL）的工具。它从要访问的 URL 列表（称为种子）开始，这取决于爬虫的启动方式。虽然有很多爬虫工具，但以下示例使用 [Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy)：

![Zed Attack Proxy 屏幕](images/OWASPZAPSP.png)\
*图 4.1.7-1：Zed Attack Proxy 屏幕*

[ZAP](https://github.com/zaproxy/zaproxy) 提供各种自动爬虫选项，可根据测试人员的需要加以利用：

- [Spider](https://www.zaproxy.org/docs/desktop/start/features/spider/)
- [Ajax Spider](https://www.zaproxy.org/docs/desktop/addons/ajax-spider/)
- [OpenAPI Support](https://www.zaproxy.org/docs/desktop/addons/openapi-support/)

## 工具

- [Zed Attack Proxy (ZAP)](https://github.com/zaproxy/zaproxy)
- [电子表格软件列表](https://en.wikipedia.org/wiki/List_of_spreadsheet_software)
- [图表软件](https://en.wikipedia.org/wiki/List_of_concept-_and_mind-mapping_software)

## 参考资料

- [代码覆盖率](https://en.wikipedia.org/wiki/Code_coverage)
