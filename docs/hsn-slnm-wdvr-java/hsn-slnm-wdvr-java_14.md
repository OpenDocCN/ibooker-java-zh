# 附录 A. Selenium 4 新功能简介

本附录总结了 Selenium 4 中可用的新功能。内容的目的有两个：首先，它列举了 Selenium 套件核心组件（即 WebDriver、Driver 和 IDE）中的新功能，并提供了解释每个方面的书籍章节的链接。此外，本附录还描述了 Selenium 4 中随 Selenium 3 到 4 迁移时变化的其他方面，例如文档和治理。第二个目标是识别从 Selenium 3 到 4 迁移时废弃部分及相应的新功能。

# Selenium WebDriver

Selenium WebDriver 4.0.0 的第一个稳定版本于 2021 年 10 月 13 日发布。表 A-1 总结了此版本与上一个稳定版本（即 Selenium WebDriver 3.141.59）相比最重要的新功能。

表 A-1\. Selenium WebDriver 4 中的新功能

| 功能 | 描述 | 章节 | 部分 |
| --- | --- | --- | --- |
| 完全采纳 W3C WebDriver | Selenium WebDriver API 和驱动程序之间的标准通信协议 | 第一章 | “Selenium WebDriver” |
| 相对定位器 | 基于其他网页元素的接近程度的定位策略 | 第三章 | “相对定位器” |
| 固定脚本 | 将一段 JavaScript 附加到 WebDriver 会话 | 第四章 | “固定脚本” |
| 元素截图 | 捕获网页元素的截图（而不是整个页面） | 第四章 | “WebElement 截图” |
| 影子 DOM | 无缝访问影子树 | 第四章 | “影子 DOM” |
| 打开新窗口和标签页 | 改进的导航到不同窗口和标签页的方式 | 第四章 | “标签页和窗口” |
| 装饰器 | 用于实现事件监听器的 `WebDriver` 对象包装器 | 第四章 | “事件监听器” |
| Chrome DevTools 协议 | 在基于 Chromium 的浏览器中（如 Chrome 和 Edge）原生访问 DevTools | 第五章 | “Chrome DevTools 协议” |
| 网络拦截 | 针对后端请求进行存根处理并拦截网络流量 | 第五章 | “网络拦截器” |
| 基本身份验证 | 简化的基本和摘要身份验证 API | 第五章 | “基本和摘要身份验证” |
| 全页面截图 | 捕获网页内容的完整截图 | 第五章 | “全页面截图” |
| 位置上下文 | 模拟地理位置坐标 | 第五章 | “位置上下文” |
| 将网页打印为 PDF | 将网页保存为 PDF 文档 | 第五章 | “打印页面” |
| WebDriver BiDi | 驱动程序和浏览器之间的双向通信 | 第五章 | “WebDriver BiDi” |

## 迁移指南

本节总结了将使用 Selenium WebDriver 3 的现有代码库迁移到版本 4 所需的更改。

### 定位器

用于查找元素的实用方法（`FindsBy` 接口）在 Selenium WebDriver 4 中已移除。表 A-2 比较了 Selenium WebDriver 中查找 Web 元素的旧 API 和新 API。有关此功能的更多详细信息，请参见 “定位 Web 元素”。

表 A-2\. Selenium WebDriver 4 中网页元素位置的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
driver.findElementByTagName("tagName");
driver.findElementByLinkText("link");
driver
    .findElementByPartialLinkText("partLink");
driver.findElementByName("name");
driver.findElementById("id");
driver.findElementByClassName("class");
driver.findElementByCssSelector("css");
driver.findElementByXPath("xPath");
```

|

```java
driver.findElement(By.tagName("tagName"));
driver.findElement(By.linkText("link"));
driver.findElement(By
    .partialLinkText("partLink"));
driver.findElement(By.name("name"));
driver.findElement(By.id("id"));
driver.findElement(By.className("class"));
driver.findElement(By.cssSelector("css"));
driver.findElement(By.xpath("xPath"));
```

|

### 用户手势

类 `Actions` 允许模拟复杂的用户手势（如拖放、悬停、鼠标移动等）。正如表 A-3 中展示的，这个类暴露的 API 在 Selenium WebDriver 4 中已经简化。关于这个类的更多信息和示例，请参见 “用户手势”。

表 A-3\. Selenium WebDriver 4 中 Actions 类的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
actions.moveToElement(webElement).click();
actions.moveToElement(webElement).doubleClick();
actions.moveToElement(webElement).contextClick();
actions.moveToElement(webElement).clickAndHold();
actions.moveToElement(webElement).release();
```

|

```java
actions.click(webElement);
actions.clickAndHold(webElement);
actions.contextClick(webElement);
actions.doubleClick(webElement);
actions.release(webElement);
```

|

### 等待和超时

指定超时的参数从 `TimeUnit` 切换到 `Duration`。表 A-4 中描述了这一变化。您可以在 “等待策略” 和 “超时” 中找到更多细节。

表 A-4\. Selenium WebDriver 4 中等待和超时的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
new WebDriverWait(driver, 3).
    until(ExpectedConditions.
    elementToBeClickable(By.id("id")));

Wait<WebDriver> wait =
    new FluentWait<WebDriver>(driver)
    .withTimeout(30, TimeUnit.SECONDS)
    .pollingEvery(5, TimeUnit.SECONDS)
    .ignoring(NoSuchElementException.class);

driver.manage().timeouts()
    .implicitlyWait(10, TimeUnit.SECONDS);
driver.manage().timeouts()
    .setScriptTimeout(3, TimeUnit.MINUTES);
driver.manage().timeouts()
    .pageLoadTimeout(30, TimeUnit.SECONDS);
```

|

```java
new WebDriverWait(driver,
    Duration.ofSeconds(3)).until(
    ExpectedConditions.
    elementToBeClickable(By.id("id")));

Wait<WebDriver> wait =
     new FluentWait<WebDriver>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofSeconds(5))
    .ignoring(NoSuchElementException.class);

driver.manage().timeouts()
    .implicitlyWait(Duration.ofSeconds(10));
driver.manage().timeouts()
    .scriptTimeout(Duration.ofMinutes(3));
driver.manage().timeouts()
    .pageLoadTimeout(Duration.ofSeconds(30));
```

|

### 事件监听器

在 Selenium WebDriver 3 中，我们使用 `EventFiringWebDriver` 类来创建事件监听器。在 Selenium WebDriver 4 中，这个类已被弃用，建议改用 `EventFiringDecorator` 类。表 A-5 概述了这一变化。关于此功能的完整示例，请参见 “事件监听器”。

表 A-5\. Selenium WebDriver 4 中事件监听器的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
EventFiringWebDriver newDriver =
    new EventFiringWebDriver(originalDriver);
wrapper.register(myListener);
```

|

```java
WebDriver newDriver =
    new EventFiringDecorator(myListener)
    .decorate(originalDriver);
```

|

### 功能

选择不同浏览器类型的静态方法在 Selenium WebDriver 4 中已移除。相反，我们应该使用特定于浏览器的选项。表 A-6 概述了这一变化。关于这个特性的更多细节，请参见 “浏览器能力”。

表 A-6\. Selenium WebDriver 4 中期望能力的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
DesiredCapabilities caps =
    DesiredCapabilities.chrome();
DesiredCapabilities caps =
    DesiredCapabilities.edge();
DesiredCapabilities caps =
    DesiredCapabilities.firefox();
DesiredCapabilities caps =
    DesiredCapabilities.internetExplorer();
DesiredCapabilities caps =
    DesiredCapabilities.safari();
DesiredCapabilities caps =
    DesiredCapabilities.chrome();
```

|

```java
ChromeOptions options =
    new ChromeOptions();
EdgeOptions options =
    new EdgeOptions();
FirefoxOptions options =
    new FirefoxOptions();
InternetExplorerOptions options =
    new InternetExplorerOptions();
SafariOptions options =
    new SafariOptions();
ChromeOptions options =
    new ChromeOptions();
```

|

然后，基于能力的 `WebDriver` 构造函数已弃用，而非特定于浏览器的选项。表 A-7 展示了此过程的示例。您可以在 “浏览器能力” 中找到使用此构造函数的示例。

表 A-7\. Selenium WebDriver 4 中使用能力进行 WebDriver 实例化的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
WebDriver driver = new ChromeDriver(caps);
```

|

```java
WebDriver driver = new ChromeDriver(options);
```

|

此外，Selenium WebDriver 4 中的能力合并方式发生了变化。在 Selenium WebDriver 3 中，可以通过改变调用对象来合并能力。而在 Selenium WebDriver 4 中，需要分配合并操作。表 A-8 提供了这一变化的示例。

表 A-8\. Selenium WebDriver 4 中合并能力的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
MutableCapabilities caps =
    new MutableCapabilities();
caps.setCapability("platformName",
    "Linux");
ChromeOptions options =
    new ChromeOptions();
options.setHeadless(true);
options.merge(caps);
```

|

```java
MutableCapabilities caps =
    new MutableCapabilities();
caps.setCapability("platformName",
    "Linux");
ChromeOptions options =
    new ChromeOptions();
options.setHeadless(true);
options = options.merge(caps);
```

|

最后，`BrowserType` 接口已弃用，转而采用新的 `Browser` 接口。表 A-9 显示了指定能力时这些接口之间的区别。有关此方面的更多细节，请参阅 “创建 RemoteWebDriver 对象”。

表 A-9\. Selenium WebDriver 4 中能力的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
caps.setCapability("browserName",
    BrowserType.CHROME);
```

|

```java
caps.setCapability("browserName",
    Browser.CHROME);
```

|

# Selenium Grid

Selenium Grid 4 已完全重写。新代码库基于 Selenium Grid 3 的所有经验教训，提高了其源代码的可维护性。Selenium Grid 4 支持经典模式，即独立和中心节点架构，同时提供了完全分布式架构以改善其整体性能和可扩展性。最后，Selenium Grid 4 使用 Docker、Kubernetes 或使用 OpenTelemetry 进行分布式跟踪等现代基础设施技术。

“Selenium Grid” 介绍了 Selenium Grid。然后，“Selenium Grid” 解释了 Selenium Grid 不同模式（即独立、中心节点和完全分布式）的详细信息，以及如何在 Selenium WebDriver 测试中使用它。

### 迁移指南

当作为 Java 依赖项使用 Selenium Grid 并升级到版本 4 时，除了升级版本外，还需知道 Selenium Grid 4 中项目坐标的变化。以前的 `artifactId` 是 `selenium-server`，在 Selenium Grid 4 中变为 `selenium-grid`。表 A-10 包含了 Maven 和 Gradle 中 Selenium Grid 4 的新坐标。

表 A-10\. 将 Selenium Grid 4 作为 Maven 和 Gradle 依赖项的迁移

| Selenium WebDriver 3 | Selenium WebDriver 4 |
| --- | --- |

|

```java
<dependency>
 <groupId>org.seleniumhq.selenium</groupId>
 <artifactId>selenium-server</artifactId>
 <version>3.141.59</version>
</dependency>
```

|

```java
<dependency>
 <groupId>org.seleniumhq.selenium</groupId>
 <artifactId>selenium-grid</artifactId>
 <version>4.0.0</version>
</dependency>
```

|

|

```java
testImplementation("org.seleniumhq.selenium:
 selenium-server:3.141.59")
```

|

```java
testImplementation("org.seleniumhq.selenium:
 selenium-grid:4.0.0")
```

|

# Selenium IDE

Selenium IDE 在 “Selenium IDE” 中引入。截至 Selenium 4，一些新功能包括：

备份元素选择器

Selenium IDE 为每个元素记录多个定位器（例如按 id、XPath 或 CSS 选择器），因此，如果测试执行未能使用第一个定位器找到元素，它会依次尝试后续的定位器直到找到为止。

控制流

Selenium IDE 通过条件语句（例如`if`、`else`、`else if`和`end`）和循环（例如`do`、`while`、`times`和`forEach`）增强了脚本执行。

代码导出

Selenium IDE 允许将录制结果导出为多种 Selenium WebDriver 绑定语言（即 C#、Java、JavaScript、Python 和 Ruby）和单元测试框架（即 NUnit、xUnit、JUnit、Mocha、pytest 和 RSpec）。

插件

我们可以通过自定义插件扩展 Selenium IDE（例如引入新的命令或第三方集成）。

# 其他新功能

Selenium 4 大大改进了官方的 Selenium 文档。新网站位于[*https://www.selenium.dev*](https://www.selenium.dev)，内容涵盖了 Selenium 的子项目（WebDriver、IDE 和 Grid）、用户指南、博客、支持和其他项目信息。

最后，自 2009 年以来一直担任 Selenium 项目联合创始人和项目负责人的 Simon Stewart，于 2021 年 10 月 27 日辞去了 Selenium 项目的领导职务。你可以在[Selenium 网站](https://www.selenium.dev/project)上找到当前的项目结构（包括项目领导委员会、技术领导委员会和 Selenium 贡献者及触发器）和治理（即模型、哲学、项目角色、决策过程等）。
