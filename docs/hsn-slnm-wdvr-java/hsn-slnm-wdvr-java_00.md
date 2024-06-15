# 前言

Selenium 是一个开源的大型项目，它可以实现对 Web 浏览器的自动化。Selenium 项目的核心组件是 Selenium WebDriver，一个用于以编程方式控制浏览器（如 Chrome、Firefox、Edge、Safari 或 Opera）的库。Selenium WebDriver 提供了跨浏览器的应用程序编程接口（API），支持多种编程语言（官方支持 Java、JavaScript、Python、C＃或 Ruby）。

尽管我们可以使用 Selenium WebDriver 实现与浏览器自动化相关的多种目的，但它的主要用途是为 Web 应用程序验证实现端到端测试。全球数千家组织和测试人员现在都在使用 Selenium，它是端到端测试的主要解决方案之一，支持一个价值数百万美元的行业。

# 谁应该阅读本书

本书全面总结了 Selenium WebDriver 版本 4 的主要功能，使用 Java 作为语言绑定。它回顾了自动化 Web 导航、浏览器操作、Web 元素交互、用户模拟、自动化驱动程序管理、页面对象模型（POM）设计模式、使用远程和云基础设施、与 Docker 和第三方工具集成等主要方面。

本书的主要受众包括不同级别的 Java 程序员（从初学者到高级），比如开发人员、测试人员、质量保证工程师等。因此，您需要对 Java 语言和面向对象编程有基本的了解。最终目标是全面了解 Selenium WebDriver 的主要方面，以便使用您选择的不同测试框架（例如 JUnit 或 TestNG）在 Java 中创建端到端测试。

# 为什么写这本书

测试自动化是一种利用自动化工具控制测试执行的软件测试技术。它可以提高效率和效果，同时确保软件系统的整体质量。在这个领域，Selenium WebDriver 是开发面向 Web 应用程序的端到端测试的事实标准库。本书是迄今为止对 Selenium 4 的第一次完整评估。

本书采用了一种学以致用的方法。为此，我们通过可立即执行的测试示例来回顾 Selenium WebDriver 的主要功能。这些示例在 GitHub 的一个开源存储库中公开可用（[*https://github.com/bonigarcia/selenium-webdriver-java*](https://github.com/bonigarcia/selenium-webdriver-java)）。为了完整起见，此存储库包含每个测试示例在不同的嵌入式测试框架中的不同风味：JUnit 4、JUnit 5（单独或与 Selenium-Jupiter 结合使用）和 TestNG。

# 阅读本书

本书内容分为 3 部分和 10 章节：

## 第一部分，介绍

第一部分提供了关于 Selenium、测试自动化和项目设置的技术背景。这部分比较理论，由两章组成：

+   第一章，“Selenium 概述”，介绍了 Selenium 项目的核心组件（WebDriver、Grid 和 IDE）及其生态系统（即围绕 Selenium 的工具和技术）。此外，本章还回顾了与 Selenium 相关的端到端测试原则。

+   第二章，“测试准备”，解释了如何设置包含使用 Selenium WebDriver API 的端到端测试的 Java 项目（Maven 和 Gradle）。然后，您将学习如何使用不同的测试框架（JUnit 4、JUnit 5（单独或与 Selenium-Jupiter 结合）、TestNG）开发您的第一个 WebDriver 测试。

## 第二部分，Selenium WebDriver API

第 II 部分提供了对 Selenium WebDriver API 的实用洞察。本部分以示例库中可用的测试为指导，并包括以下章节：

+   第三章，“WebDriver 基础”，描述了用于自动化与 Web 应用程序交互的 Selenium WebDriver API 的主要方面。因此，本章还回顾了几种定位和等待 Web 元素的策略。此外，您将了解如何在浏览器中模拟用户操作（即使用键盘和鼠标进行的自动化交互）。

+   第四章，“与浏览器无关的功能”，回顾了 Selenium WebDriver API 在不同浏览器中可互操作的方面。因此，本章展示了如何执行 JavaScript、创建事件监听器、管理窗口、制作屏幕截图、处理 Shadow DOM、操作 Cookie、访问浏览器历史或 Web 存储，以及与窗口、标签和 iframe 等元素交互。

+   第五章，“特定于浏览器的操作”，解释了 Selenium WebDriver API 特定于特定浏览器的方面。这些特性组包括浏览器功能（选项、参数、偏好设置等）、Chrome 开发者工具协议（CDP）、地理位置功能、基本和 Web 身份验证、将页面打印为 PDF 或 WebDriver BiDi API。

+   第六章，“远程 WebDriver”，描述了如何使用 Selenium WebDriver API 来控制远程浏览器。然后，您将学习如何设置和使用 Selenium Grid 版本 4。最后，您将了解如何在云提供商（如 Sauce Labs、BrowserStack 或 CrossBrowserTesting 等）和 Docker 容器中使用高级基础设施进行 Selenium 测试。

## 第三部分，高级概念

第 III 部分专注于在不同领域和用例中利用 Selenium WebDriver API。本部分包括以下章节：

+   第七章，“页面对象模型（POM）”，介绍了 POM，这是与 Selenium WebDriver 一起使用的流行设计模式。该模式允许用户使用面向对象的类来建模网页，以便于测试维护和减少代码重复。

+   第八章，“测试框架的细节”，回顾了与 Selenium WebDriver 一起使用的单元测试框架的几个特定功能，这些功能允许改进整个测试过程的不同方面。为此，本章首先解释了如何进行跨浏览器测试（即，使用参数化测试和测试模板重用相同的测试逻辑以验证使用不同浏览器的 Web 应用程序）。

+   第九章，“第三方集成”，回顾了您可以使用的不同技术来增强您的 Selenium WebDriver 测试，例如报告工具、测试数据生成和其他框架（例如 Cucumber 或 Spring）。此外，本章描述了如何使用外部库与 Selenium 结合使用来实现特定用例，例如文件下载或非功能性测试（例如负载、安全性或可访问性）。

+   第十章，“超越 Selenium”，介绍了与 Selenium 相关的几个自动化框架：Appium（用于移动测试）和 REST Assured（用于测试 REST Web 服务）。最后，我们回顾了一些与 Selenium WebDriver 最相关的当前替代方案，例如 Cypress、WebDriverIO、TestCafe、Puppeteer 或 Playwright。

# 本书中使用的约定

本书使用了以下印刷约定：

*斜体*

指示新术语、网址、电子邮件地址、文件名和文件扩展名。

`等宽`

用于程序列表，以及在段落内引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

**`等宽粗体`**

显示用户应该逐字输入的命令或其他文本。

*`等宽斜体`*

显示应替换为用户提供的值或由上下文确定的值的文本。

###### 提示

这个元素表示提示或建议。

###### 注意

这个元素表示一般注释。

###### 警告

这个元素表示警告或注意。

# 使用代码示例

可以在 [*https://github.com/bonigarcia/selenium-webdriver-java*](https://github.com/bonigarcia/selenium-webdriver-java) 上下载代码示例。如果您有技术问题或使用代码示例时遇到问题，请发送电子邮件至 *bookquestions@oreilly.com*。

这本书旨在帮助您完成工作。一般情况下，如果本书提供示例代码，您可以在您的程序和文档中使用它。除非您复制了代码的大部分内容，否则无需联系我们以获得许可。例如，编写使用本书多个代码片段的程序不需要许可。销售或分发 O’Reilly 书籍中的示例代码需要许可。引用本书并引用示例代码回答问题无需许可。将本书的大量示例代码整合到您产品的文档中需要许可。

我们感激，但通常不需要，署名。署名通常包括标题、作者、出版商和 ISBN。例如：“*Hands-On Selenium WebDriver with Java* 作者 Boni García（O’Reilly）。版权所有 2022 Boni García，978-1-098-11000-0。”

如果您认为您对代码示例的使用超出了公平使用或上述授权范围，请随时通过*permissions@oreilly.com*联系我们。


# 致谢

首先，我要感谢 O’Reilly 团队使这本书成为现实。他们在这段旅程的每个阶段都提供了卓越的编辑支持。

我也想感谢这本书的技术审阅人员。他们宝贵的反馈和专业建议显著提高了书籍的质量：Diego Molina（Sauce Labs 的高级软件工程师，Selenium 项目的技术负责人），Filippo Ricca（热那亚大学计算机科学副教授），Andrea Stocco（瑞士意大利大学软件研究所的博士后研究员），Ivan Krutov（Aerokube 的软件开发人员）和 Daniel Hinojosa（独立顾问、程序员、讲师、演讲者和作者）——非常感谢你们。

最后，我要感谢 Simon Stewart 的贡献（WebDriver 的创造者，直到 2021 年担任 Selenium 项目负责人）。非常感谢你，Simon，为这本书写序言并提供关于内容的宝贵反馈。但更重要的是，我要感谢你在这些年中领导 Selenium 项目的工作。你对自动化测试社区的贡献已经成为软件历史的一部分。
