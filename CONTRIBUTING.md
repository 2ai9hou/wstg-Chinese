# 为测试指南做贡献

感谢您考虑为 Web 安全测试指南（WSTG）做出贡献！

以下是您可以进行有益贡献的一些方式。[开源指南：为何以及如何做贡献](https://opensource.guide/how-to-contribute/) 也是一个很好的资源。您需要一个 [GitHub 账户](https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account) 才能提供帮助。

- [成为作者](#成为作者)
- [成为审校者或编辑](#成为审校者或编辑)
    - [技术审校](#技术审校)
    - [编辑审校](#编辑审校)
- [如何创建议题](#如何创建议题)
- [如何提交 Pull Request](#如何提交-pull-request)
- [如何设置贡献者环境](#如何设置贡献者环境)
- [使用 GitHub 开发环境进行贡献](#使用-github-开发环境进行贡献)

## 成为作者

没有安全社区作家的贡献，这个项目就不可能存在！我们的作者帮助保持 WSTG 对每个人都相关且有用。

无论您是提交新部分还是在现有内容中添加信息，请遵循[模板示例](template/999-Foo_Testing/1-Testing_for_a_Cat_in_a_Box.md)。[模板章节说明在此](template/999-Foo_Testing/2-Template_Explanation.md)。

提交[拉取请求](#如何提交-pull-request)时，作者应将贡献链接到议题：

1. 打开一个[添加新内容议题](https://github.com/OWASP/wstg/issues/new?assignees=&labels=New&template=new-content.md&title=)，或选择一个[未分配的新内容议题](https://github.com/OWASP/wstg/issues?q=is%3Aopen+is%3Aissue+label%3ANew+no%3Aassignee) 并请求分配给您。
2. 创建并切换到名为 `new-<issue number>` 的新本地分支。例如，`git checkout -b new-164`。

## 成为审校者或编辑

保持项目最新且看起来漂亮是团队努力！WSTG 是一份不断更新的文档，从您的技术或编辑审校中受益。

提交[拉取请求](#如何提交-pull-request)时，审校者和编辑应将贡献链接到议题：

1. 选择一个[带有 `help wanted` 标签的开放议题](https://github.com/OWASP/wstg/labels/help%20wanted) 来处理，或自己[打开一个议题](https://github.com/OWASP/wstg/issues/new/choose)。在议题中发表评论并请求分配给您。
2. 创建并切换到名为 `fix-<issue number>` 的新本地分支。例如，`git checkout -b fix-88`。

### 技术审校

如果您在 WSTG 涵盖的任何主题方面有专业知识，我们鼓励您进行技术审校。请确保文章：

- 遵循[文章模板材料](template)
- 遵循[撰写规范](style_guide.md)
- 准确描述漏洞和测试
- 有适当和最新的资源内联链接
- 提供适合具有基本技术专业知识读者的完整相关信息

### 编辑审校

语法爱好者集合！WSTG 欢迎您在语法、格式、措辞和简洁方面的改进。所有更改应遵循[撰写规范](style_guide.md)。

请不要犹豫进行您认为合适的尽可能多的更改，特别是如果您注意到现有内容与[文章模板材料](template)不符。

### 翻译

由于同步图片和删除内容的挑战，WSTG 不再直接处理入站翻译工作。

目前我们建议您启动另一个仓库来处理特定语言的翻译。一旦您为给定版本的指南制作了 PDF，我们将很乐意将其附加到相应的发布中。只需在此[打开一个议题](https://github.com/OWASP/wstg/issues/new)要求我们这样做。

我们还愿意列出您的翻译仓库，只需[告诉我们](https://github.com/OWASP/wstg/issues/new)它在哪里。

## 如何创建议题

使用适当的模板[创建议题](https://github.com/OWASP/wstg/issues/new/choose)。

选择一个简短、描述性的标题。简要解释您认为需要更改的内容。除其他外，您的建议可能包括语法或拼写错误，或解决不充分或过时的内容。

## 如何提交 Pull Request

以下是创建和提交我们可以快速审阅和合并的拉取请求（PR）的步骤。

1. [设置您的环境](#如何设置贡献者环境) 以派生项目并安装 Markdown 检查器。
2. 将您的贡献与[议题](https://github.com/OWASP/wstg/issues)关联。要更改现有内容，请阅读[成为审校者或编辑](#成为审校者或编辑)。要添加内容，请阅读[成为作者](#成为作者)。
3. 进行您的修改。请务必遵循我们的[撰写规范](style_guide.md)。
4. 当您准备好提交工作时，将您的更改推送到您的派生。确保您的派生与 `master`[同步](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/syncing-a-fork)。
5. 您可以提交[草稿 PR](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests#draft-pull-requests) 或[常规 PR](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork)。如果您的工作尚未准备好审阅和合并，请选择草稿 PR。当您的更改准备好被审阅时，您可以转换为常规 PR。参见[如何更改 PR 的阶段](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/changing-the-stage-of-a-pull-request)了解更多。

您可能需要[允许维护者编辑](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/allowing-changes-to-a-pull-request-branch-created-from-a-fork)，以便我们可以帮助进行小的更改，如修复拼写错误。

一旦您提交了准备好审阅的 PR，我们将对其进行审阅。我们可能会发表评论要求澄清或更改，所以请在接下来几天内回来查看。

为增加您的 PR 被合并的机会，请确保：

1. 您已遵循上述将您的工作与议题关联的指南。
2. 您的作品已通过 Markdown 检查。
3. 您的写作遵循[文章模板材料](template)和[撰写规范](style_guide.md)。
4. 您的代码片段正确、经过良好测试，并在必要时注释以帮助理解。

一旦 PR 完成，我们将合并它！此时，您可能希望将自己添加到[项目的作者、审校者或编辑列表](document/1-Frontispiece/README.md)中。

## 如何设置贡献者环境

我们已经使开始变得容易！仓库包括 Visual Studio Code 和其他编辑器的配置文件，以帮助您保持与项目撰写规范的一致性。

1. [在 GitHub 上创建账户](https://help.github.com/en/github/getting-started-with-github/signing-up-for-a-new-github-account)。
2. 派生并克隆您自己的仓库副本。以下是[派生和与 GitHub 同步](https://help.github.com/en/github/getting-started-with-github/fork-a-repo)的完整说明。
3. 选择您的开发环境：

### 使用 Visual Studio Code

Visual Studio Code 被推荐以获得最佳体验。仓库在 `.vscode` 目录中包含预配置的设置。

1. 安装 [Visual Studio Code](https://code.visualstudio.com/)。
2. 在 Visual Studio Code 中打开克隆的仓库。
3. 出现提示时，从 `.vscode/extensions.json` 安装推荐的扩展。这些包括：
    - **markdownlint**：确保 Markdown 文件遵循项目的撰写规范
    - **Markdown All in One**：提供有用的 Markdown 编辑功能
    - **Code Spell Checker**：捕获拼写错误
    - **Prettier**：代码格式化程序
    - **GitHub Pull Request**：直接从 Visual Studio Code 管理 PR

4. `.vscode/settings.json` 中的工作区设置将自动配置 markdownlint 使用位于 `.github/configs/.markdownlint.json` 的项目配置文件。

### 使用其他编辑器

如果您使用不同的编辑器，`.editorconfig` 文件将有助于在不同编辑器之间保持一致的格式。大多数现代编辑器原生支持 EditorConfig 或通过插件支持：

- **Vim/Neovim**：安装 [editorconfig-vim](https://github.com/editorconfig/editorconfig-vim)
- **Sublime Text**：安装 [EditorConfig](https://packagecontrol.io/packages/EditorConfig)
- **Atom**：安装 [EditorConfig](https://atom.io/packages/editorconfig)
- **IntelliJ/WebStorm**：内置支持

### 本地运行检查器

为确保您的更改遵循项目的 Markdown 撰写规范，您可以在本地运行检查器：

1. 安装依赖项（需要 [Node.js](https://nodejs.org/)）：

    ```bash
    npm install
    ```

2. 运行检查器：

    ```bash
    npm run lint
    ```

检查器将检查所有 Markdown 文件并报告在提交拉取请求之前需要修复的任何样式问题。

## 使用 GitHub 开发环境进行贡献

您可以使用 GitHub 基于云的开发环境（Codespaces 和 github.dev）为这个仓库做贡献，而无需设置本地环境！

### 使用 github.dev

对于快速编辑，您可以使用 github.dev 基于 Web 的编辑器：

1. 导航到 GitHub 上的仓库。
2. 在键盘上按 `.`（句点）打开 github.dev 编辑器。
3. 进行更改并直接从浏览器提交。

注意：github.dev 编辑器对运行命令的支持有限，因此最适合简单的文本编辑。对于测试检查器和其他脚本，请使用 Codespaces 或本地环境。

### 使用 GitHub Codespaces

GitHub Codespaces 在云中提供完整的 Visual Studio Code 环境，预装了所有推荐的扩展。

1. 了解更多关于 [GitHub Codespaces](https://docs.github.com/en/codespaces/overview)。
2. 通过为此仓库[创建 codespace](https://docs.github.com/en/codespaces/developing-in-codespaces/creating-a-codespace) 开始。

我们的 `.vscode` 配置将自动使用正确的检查和格式化设置设置工作区。
