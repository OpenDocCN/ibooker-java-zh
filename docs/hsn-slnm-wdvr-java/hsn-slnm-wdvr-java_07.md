# 第五章\. 浏览器特定操作

正如你所见，Selenium WebDriver API 的许多特性在各种浏览器中是兼容的，即我们可以使用 Selenium WebDriver 以编程方式控制不同类型的浏览器。Selenium WebDriver API 的其他部分在浏览器之间是不可互操作的。换句话说，一些 WebDriver 特性对于某些浏览器（如 Chrome 或 Edge）是可用的，而对于其他浏览器（如 Firefox）则不可用（或不同）。本章回顾了这些特定于浏览器的特性。

# 浏览器 Capabilities

Selenium WebDriver 允许通过*capabilities*来指定特定于浏览器的方面。例如，无头模式、页面加载策略、使用网页扩展或推送通知管理等功能。正如图 5-1 所示，Selenium WebDriver API 提供了一组 Java 类来定义这些 capabilities。`Capabilities`接口位于这个层次结构的顶部。在内部，capabilities 接口使用键-值对处理数据，封装浏览器的特定方面。然后，不同的 Java 类实现此接口，为 Web 浏览器（如 Chrome、Edge、Firefox 等）指定 capabilities。表 5-1 总结了`Capabilities`层次结构的主要类及其对应的目标浏览器。

![hosw 0501](img/hosw_0501.png)

###### 图 5-1\. Capabilities 层次结构

表 5-1\. Capabilities 层次结构描述

| 包 | 类 | 浏览器 |
| --- | --- | --- |

|

```java
org.openqa.selenium
```

|

```java
MutableCapabilities
```

| 通用（跨浏览器） |
| --- |

|

```java
org.openqa.selenium.chrome
```

|

```java
ChromeOptions
```

| Chrome |
| --- |

|

```java
org.openqa.selenium.edge
```

|

```java
EdgeOptions
```

| Edge |
| --- |

|

```java
org.openqa.selenium.firefox
```

|

```java
FirefoxOptions
```

| Firefox |
| --- |

|

```java
org.openqa.selenium.safari
```

|

```java
SafariOptions
```

| Safari |
| --- |

|

```java
org.openqa.selenium.opera
```

|

```java
OperaOptions
```

| Opera |
| --- |

|

```java
org.openqa.selenium.ie
```

|

```java
InternetExplorerOptions
```

| Internet Explorer |
| --- |

|

```java
org.openqa.selenium.remote
```

|

```java
DesiredCapabilities
```

| 远程浏览器（参见第六章） |
| --- |

以下小节将回顾本书讨论的主要 Web 浏览器（如 Chrome、Edge 和 Firefox）的最重要 capabilities。由于 Chrome 和 Edge 都是基于 Chromium 的浏览器，因此两者可用的 capabilities 是等效的。这一事实反映在图 5-1 中，显示了`ChromeOptions`和`EdgeOptions`这两个 capability 类都继承自同一个父类（称为`ChromiumOptions`）。

## 无头浏览器

不需要图形界面与 Web 应用程序进行交互的浏览器称为 *无头* 浏览器。这些浏览器的主要用途之一是端到端测试，即自动化与 Web 应用程序的交互。目前的 Web 浏览器如 Chrome、Edge 或 Firefox 都可以作为无头浏览器运行。Selenium WebDriver API 允许使用能力（capabilities）在无头模式下启动这些浏览器。为此，首先需要创建一个浏览器能力（capabilities）的实例。在主要浏览器中，这些对象分别是 `ChromeOptions`、`EdgeOptions` 或 `FirefoxOptions` 的实例。然后，通过调用浏览器能力对象中的 `setHeadless(true)` 方法来启用无头模式。最后，在创建 `WebDriver` 对象时设置这些能力。

正如 “WebDriver 创建” 所解释的，我们有不同的方法来创建 `WebDriver` 对象。首先，我们可以使用 `WebDriver` 构造函数（例如 `new ChromeDriver()`）。此外，我们可以使用 Selenium WebDriver API 提供的构建器（即 `RemoteWebDriver.builder()`）。最后，我们可以使用 WebDriverManager 构建器解析驱动程序，并在一行代码中创建 `WebDriver` 实例。以下示例展示了这些替代方法，与浏览器能力结合使用，以启用无头浏览器模式，即：

+   示例 5-1 使用 Chrome 无头模式。此示例使用所需的构造函数（在本例中为 `ChromeDriver`）创建一个 `WebDriver` 实例。

+   示例 5-2 使用 Edge 无头模式。此示例使用 Selenium WebDriver API 中提供的构建器创建一个 `WebDriver` 实例。

+   示例 5-3 使用 Firefox 无头模式。此示例使用 WebDriverManager 创建一个 `WebDriver` 实例。请注意，在这种情况下，不需要设置方法，因为 WebDriverManager 在 WebDriver 实例化的同一行解析了驱动程序。

+   示例 5-4 通过 Selenium-Jupiter 使用 Chrome 无头模式。此示例使用 Selenium-Jupiter 提供的参数解析机制，因此我们只需在测试方法中声明一个 `ChromeDriver` 参数。然后，我们使用注解 `@Arguments` 对这个参数进行修饰，以指定该浏览器的无头模式。

##### 示例 5-1\. 使用 Chrome 无头模式进行测试

```java
class HeadlessChromeJupiterTest {

    WebDriver driver;

    @BeforeAll
    static void setupClass() {
        WebDriverManager.chromedriver().setup(); ![1](img/1.png)
    }

    @BeforeEach
    void setup() {
        ChromeOptions options = new ChromeOptions(); ![2](img/2.png)
        options.setHeadless(true); ![3](img/3.png)

        driver = new ChromeDriver(options); ![4](img/4.png)
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testHeadless() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_browser_specific_manipulation_CO1-1)

我们解析所需的驱动程序（在本例中为 chromedriver）。

![2](img/#co_browser_specific_manipulation_CO1-2)

我们使用 `ChromeOptions` 构造函数创建浏览器能力。

![3](img/#co_browser_specific_manipulation_CO1-3)

我们启用无头模式。这一行相当于 `options.add​Arguments("--headless");`。

![4](img/#co_browser_specific_manipulation_CO1-4)

我们通过在`ChromeDriver`构造函数中将选项作为参数来设置浏览器能力。

##### 示例 5-2\. 使用 Edge 浏览器的无头模式测试

```java
class HeadlessEdgeJupiterTest {

    WebDriver driver;

    @BeforeAll
    static void setupClass() {
        WebDriverManager.edgedriver().setup(); ![1](img/1.png)
    }

    @BeforeEach
    void setup() {
        EdgeOptions options = new EdgeOptions(); ![2](img/2.png)
        options.setHeadless(true); ![3](img/3.png)

        driver = RemoteWebDriver.builder().oneOf(options).build(); ![4](img/4.png)
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testHeadless() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_browser_specific_manipulation_CO2-1)

与往常一样，我们需要解析所需的驱动程序（本例中是 msedgedriver）。

![2](img/#co_browser_specific_manipulation_CO2-2)

由于我们打算使用 Edge，因此我们需要创建一个`EdgeOptions`实例来指定能力。

![3](img/#co_browser_specific_manipulation_CO2-3)

我们启用无头模式。同样，这行等同于`options.addArguments("--headless");`。

![4](img/#co_browser_specific_manipulation_CO2-4)

我们使用 WebDriver 构建器来创建`WebDriver`对象，并将选项作为参数传递。

##### 示例 5-3\. 使用 Firefox 无头模式测试

```java
class HeadlessFirefoxJupiterTest {

    WebDriver driver;

    @BeforeEach
    void setup() {
        FirefoxOptions options = new FirefoxOptions(); ![1](img/1.png)
        options.setHeadless(true); ![2](img/2.png)

        driver = WebDriverManager.firefoxdriver().capabilities(options)
                .create(); ![3](img/3.png)
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testHeadless() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_browser_specific_manipulation_CO3-1)

在这个测试中，我们使用 Firefox，因此我们创建一个`FirefoxOptions`对象来指定能力。

![2](img/#co_browser_specific_manipulation_CO3-2)

与前面的示例相同，我们启用了无头模式。

![3](img/#co_browser_specific_manipulation_CO3-3)

在此示例中，我们使用 WebDriverManager 来解析所需的驱动程序，并创建`WebDriver`对象，同时指定先前创建的浏览器能力。

###### 注意

在这些示例中，创建`WebDriver`对象的策略是可以互换的。换句话说，例如，我们也可以为每个浏览器的无头模式使用 WebDriverManager 构建器。

##### 示例 5-4\. 使用 Selenium-Jupiter 中的 Chrome 无头模式测试

```java
@ExtendWith(SeleniumJupiter.class)
class HeadlessChromeSelJupTest {

    @Test
    void testHeadless(@Arguments("--headless") ChromeDriver driver) { ![1](img/1.png)
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_browser_specific_manipulation_CO4-1)

我们使用注解`@Arguments`来指定浏览器（本例中为 Chrome）中的无头模式。

## 页面加载策略

Selenium WebDriver 允许配置不同的页面加载方式。为此，Selenium WebDriver API 提供了`PageLoadStrategy`枚举。表 5-2 描述了该枚举的可能值及其目的。Selenium WebDriver 内部使用 DOM API 的`document.readyState`属性来检查网页加载状态。

表 5-2\. PageLoadStrategy 值

| 加载策略 | 描述 | 准备状态 |
| --- | --- | --- |

|

```java
PageLoadStrategy.NORMAL
```

| 默认模式。Selenium WebDriver 等待整个页面加载完成（即 HTML 内容和子资源，如样式表、图像、JavaScript 文件等）。 |
| --- |

```java
"complete"
```

|

|

```java
PageLoadStrategy.EAGER
```

| Selenium WebDriver 等待 HTML 文档完成加载和解析，但子资源（脚本、图像、样式表等）仍在加载中。 |
| --- |

```java
"interactive"
```

|

|

```java
PageLoadStrategy.NONE
```

| Selenium WebDriver 仅等待 HTML 文档下载完成。 |
| --- |

```java
"loading"
```

|

我们需要调用浏览器能力（例如`ChromeOptions`，`FirefoxOptions`等）的`setPageLoadStrategy()`方法来设置这些策略（`NORMAL`，`EAGER`或`NONE`）。示例 5-5 展示了使用 Chrome 和`NORMAL`策略的测试。在示例库中，您可以找到使用其他策略（`EAGER`和`NONE`）的 Edge 和 Firefox 的等效示例。在这些示例中，除了在测试设置中指定加载策略外，测试逻辑还计算加载页面所需的时间，并在标准输出中显示此值。

##### 示例 5-5\. 在 Chrome 中使用正常页面加载策略的测试

```java
class PageLoadChromeJupiterTest {

    static final Logger log = getLogger(lookup().lookupClass());

    WebDriver driver;

    PageLoadStrategy pageLoadStrategy;

    @BeforeEach
    void setup() {
        ChromeOptions options = new ChromeOptions(); ![1](img/1.png)
        pageLoadStrategy = PageLoadStrategy.NORMAL;
        options.setPageLoadStrategy(pageLoadStrategy); ![2](img/2.png)

        driver = WebDriverManager.chromedriver().capabilities(options).create(); ![3](img/3.png)
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testPageLoad() {
        long initMillis = System.currentTimeMillis(); ![4](img/4.png)
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        Duration elapsed = Duration
                .ofMillis(System.currentTimeMillis() - initMillis); ![5](img/5.png)

        Capabilities capabilities = ((RemoteWebDriver) driver)
                .getCapabilities(); ![6](img/6.png)
        Object pageLoad = capabilities
                .getCapability(CapabilityType.PAGE_LOAD_STRATEGY); ![7](img/7.png)
        String browserName = capabilities.getBrowserName();
        log.debug(
                "The page took {} ms to be loaded using a '{}' strategy in {}",
                elapsed.toMillis(), pageLoad, browserName); ![8](img/8.png)

        assertThat(pageLoad).isEqualTo(pageLoadStrategy.toString()); ![9](img/9.png)
    }

}
```

![1](img/#co_browser_specific_manipulation_CO5-1)

由于我们在此测试中使用 Chrome，我们实例化`ChromeOptions`以指定能力。

![2](img/#co_browser_specific_manipulation_CO5-2)

我们设置页面加载策略为`NORMAL`。

![3](img/#co_browser_specific_manipulation_CO5-3)

我们使用 WebDriverManager 解析驱动程序，创建`WebDriver`实例并指定能力。

![4](img/#co_browser_specific_manipulation_CO5-4)

在加载页面之前获取系统时间戳。

![5](img/#co_browser_specific_manipulation_CO5-5)

在加载页面后获取系统时间戳。

![6](img/#co_browser_specific_manipulation_CO5-6)

我们读取`WebDriver`对象的能力。

![7](img/#co_browser_specific_manipulation_CO5-7)

我们读取了所用的页面加载策略。

![8](img/#co_browser_specific_manipulation_CO5-8)

我们追踪加载网页所需的时间。

![9](img/#co_browser_specific_manipulation_CO5-9)

我们验证加载策略是否如最初配置的那样。

## 设备仿真

主要的网页浏览器使用开发工具（即 Chromium 基浏览器中的 DevTools 和 Firefox 中的 Developer Tools）以以下方式模拟移动设备：

模拟移动视窗

使用给定移动设备的宽度和高度减少网页的用户可见区域

限制网络带宽

为了模拟移动网络（例如 3G），需要降低连接速度。

限制 CPU 速度

减慢处理性能

模拟地理位置

设置自定义全球定位系统（GPS）坐标

设置方向

旋转屏幕

图 5-2 展示了通过 DevTools 在 Chrome 中使用移动仿真的屏幕截图。

![hosw 0502](img/hosw_0502.png)

###### 图 5-2\. 使用 Chrome 开发工具进行移动仿真

在撰写本文时，这种移动设备仿真可以通过 Selenium WebDriver API 在基于 Chromium 的浏览器（Chrome 和 Edge）中自动化，但在 Firefox 中不能（因为 geckodriver 中未实现）。为此，我们需要在`ChromeOptions`或`EdgeOptions`中设置实验性选项`mobileEmulation`。

然后，有两种方法来指定要模拟的移动设备。首先，我们可以指定特定的移动设备（例如，Pixel 2、iPad Pro 或 Galaxy Fold 等）。由于此列表在每个 Chromium 发布版中更新，检查可能性的最佳方法是检查 DevTools 中可用设备（例如，图 5-2 中选择了 iPhone X）。示例 5-6 展示了一个测试设置，其中我们使用标签 `iPhone 6/7/8` 指定了特定的移动设备。

##### 示例 5-6\. 通过指定设备进行移动模拟的测试设置

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    Map<String, Object> mobileEmulation = new HashMap<>(); ![1](img/1.png)
    mobileEmulation.put("deviceName", "iPhone 6/7/8"); ![2](img/2.png)
    options.setExperimentalOption("mobileEmulation", mobileEmulation); ![3](img/3.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create(); ![4](img/4.png)
}
```

![1](img/#co_browser_specific_manipulation_CO6-1)

我们需要创建一个 `HashMap` 对象来指定移动模拟选项。

![2](img/#co_browser_specific_manipulation_CO6-2)

然后，我们只需要选择设备名称（在本例中为 iPhone 6/7/8）。

![3](img/#co_browser_specific_manipulation_CO6-3)

我们使用实验选项设置设备模拟。

![4](img/#co_browser_specific_manipulation_CO6-4)

通常情况下，我们创建一个指定这些选项的 `WebDriver` 对象。

设置移动模拟的第二种选择是指定模拟设备的各个属性。这些属性包括：

`width`

设备屏幕宽度（以像素为单位）

`height`

设备屏幕高度（以像素为单位）

`pixelRatio`

物理像素与逻辑像素之间的比率

`touch`

是否模拟触摸事件；默认值为 `true`

除了这些属性之外，我们还可以指定模拟设备的 *用户代理*。在 HTTP 中，用户代理是指定在请求标头中的字符串，唯一地标识 Web 浏览器的类型。它包含开发代号、版本、平台和其他信息。示例 5-7 展示了一个测试设置，说明了此功能的使用。

##### 示例 5-7\. 通过指定各个属性进行设备模拟的测试设置

```java
@BeforeEach
void setup() {
    EdgeOptions options = new EdgeOptions();
    Map<String, Object> mobileEmulation = new HashMap<>();
    Map<String, Object> deviceMetrics = new HashMap<>(); ![1](img/1.png)
    deviceMetrics.put("width", 360);
    deviceMetrics.put("height", 640);
    deviceMetrics.put("pixelRatio", 3.0);
    deviceMetrics.put("touch", true);
    mobileEmulation.put("deviceMetrics", deviceMetrics);  ![2](img/2.png)
    mobileEmulation.put("userAgent",
            "Mozilla/5.0 (Linux; Android 4.2.1; en-us; Nexus 5 Build/JOP40D) "
                    + "AppleWebKit/535.19 (KHTML, like Gecko) "
                    + "Chrome/18.0.1025.166 Mobile Safari/535.19");  ![3](img/3.png)
    options.setExperimentalOption("mobileEmulation", mobileEmulation);

    driver = WebDriverManager.edgedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO7-1)

我们创建了一个哈希映射来存储模拟移动设备的各个属性，即，`width`、`height`、`pixelRatio` 和 `touch`。

![2](img/#co_browser_specific_manipulation_CO7-2)

我们通过在移动模拟映射中设置标签 `deviceMetrics` 来设置这些属性。

![3](img/#co_browser_specific_manipulation_CO7-3)

我们为 Nexus 5 设备中的 Chrome Mobile 18 设置了自定义用户代理。

## 网页扩展

网页扩展（也称为 *插件* 或 *附加组件*）是可以修改或增强 Web 浏览器默认操作的程序。用户通常使用 Web 商店安装网页扩展。这些商店是由浏览器维护者支持的 Web 应用程序，用于托管公共 Web 扩展。表 5-3 总结了 Chrome、Edge 和 Firefox 的 Web 商店。

表 5-3\. 主要浏览器的网络商店

| 网络商店 | 浏览器 | 网址 |
| --- | --- | --- |
| Chrome 网上应用店 | Chrome | [*https://chrome.google.com/webstore/category/extensions*](https://chrome.google.com/webstore/category/extensions) |
| Edge 插件 | Edge | [*https://microsoftedge.microsoft.com/addons/Microsoft-Edge-Extensions-Home*](https://microsoftedge.microsoft.com/addons/Microsoft-Edge-Extensions-Home) |
| Firefox 浏览器插件 | Firefox | [*https://addons.mozilla.org/zh-CN/firefox*](https://addons.mozilla.org/zh-CN/firefox) |

我们可以在 WebDriver 会话中使用功能来安装网页扩展。在基于 Chromium 的浏览器中，如 Chrome 和 Edge，我们使用 `ChromeOptions` 或 `EdgeOptions` 对象的 `addExtensions()` 方法。示例 5-8 展示了在 Chrome 中安装本地扩展的测试设置。

##### 示例 5-8\. 在 Chrome 中安装网页扩展的测试设置

```java
@BeforeEach
void setup() throws URISyntaxException {
    Path extension = Paths
            .get(ClassLoader.getSystemResource("dark-bg.crx").toURI()); ![1](img/1.png)
    ChromeOptions options = new ChromeOptions();
    options.addExtensions(extension.toFile()); ![2](img/2.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO8-1)

我们安装一个打包为 Chrome 扩展（CRX）文件的网页扩展。这个文件是测试资源（位于 Java 项目的 `src\test\resources` 文件夹中）。该扩展会改变网站的外观和感觉，以在黑色背景上使用浅色文本。图 5-3 展示了使用 WebDriver 测试加载此扩展时的实践网站的截图。

![2](img/#co_browser_specific_manipulation_CO8-2)

我们在 Chrome 选项中添加扩展，将扩展作为 Java `File` 传递。

![hosw 0503](img/hosw_0503.png)

###### 图 5-3\. 使用 dark-bg.crx 扩展加载时的实践站点

当 Firefox 由 WebDriver 控制时，也允许加载网页扩展。然而，语法有所不同。示例 5-9 说明了这一点。

##### 示例 5-9\. 在 Firefox 中安装网页扩展的测试设置

```java
@BeforeEach
void setup() throws URISyntaxException {
    Path extension = Paths
            .get(ClassLoader.getSystemResource("dark-bg.xpi").toURI()); ![1](img/1.png)
    FirefoxOptions options = new FirefoxOptions();
    FirefoxProfile profile = new FirefoxProfile(); ![2](img/2.png)
    profile.addExtension(extension.toFile()); ![3](img/3.png)
    options.setProfile(profile); ![4](img/4.png)

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

![1](img/#co_browser_specific_manipulation_CO9-1)

我们在 Firefox 中使用与 Chrome/Edge 相同的扩展，但在这种情况下，打包是专门针对 Firefox 的。请注意文件是不同的。这次，它被打包为 *XPInstall* 文件，即包含网页扩展源代码、资源（例如图像）和元数据的压缩归档。

![2](img/#co_browser_specific_manipulation_CO9-2)

我们需要创建一个自定义的 Firefox 配置文件（即存储自定义设置的地方）。

![3](img/#co_browser_specific_manipulation_CO9-3)

我们将扩展作为 Java `File` 添加到 Firefox 配置文件中。

![4](img/#co_browser_specific_manipulation_CO9-4)

我们在 Firefox 选项中设置配置文件。

基于 Chromium 的浏览器（例如 Chrome、Edge）还允许从源代码加载扩展（即未打包为 CRX 文件）。这个功能在开发期间自动测试网页扩展非常方便。示例 5-10 展示了说明此功能的测试设置。

##### 示例 5-10\. 从源代码中安装 Edge 中的网页扩展的测试设置

```java
@BeforeEach
void setup() throws URISyntaxException {
    Path extension = Paths
            .get(ClassLoader.getSystemResource("web-extension").toURI()); ![1](img/1.png)
    EdgeOptions options = new EdgeOptions();
    options.addArguments(
            "--load-extension=" + extension.toAbsolutePath().toString()); ![2](img/2.png)

    driver = WebDriverManager.edgedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO10-1)

此示例中使用的扩展位于`web-extension`文件夹中；这是一个测试资源文件夹（存储在 Java 项目的`src\test\resources`中）。此扩展遵循[浏览器扩展 API](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions)。它使用 JavaScript 来更改一级标题（`h1` 标签）的内容为自定义消息。图 5-4 显示了使用此扩展时练习网站的屏幕截图。

![2](img/#co_browser_specific_manipulation_CO10-2)

我们使用 `--load-extension` 参数指定扩展路径。

###### 注意

Selenium WebDriver 每次执行时都会创建一个新的浏览器配置文件。因此，通过 Selenium WebDriver 安装的 Web 扩展在目标浏览器中不是永久性的。

![hosw 0504](img/hosw_0504.png)

###### 图 5-4\. 使用本地扩展加载练习网站时的情况

自 Selenium 4.1 起，Firefox 也允许从其源代码安装 Web 扩展。为此，`FirefoxDriver` 扩展了接口 `HasExtensions`，提供了方法 `installExtension`。示例 5-11 展示了使用此功能的测试设置。

##### 示例 5-11\. 在 Firefox 中安装 Web 扩展的测试设置

```java
@BeforeEach
void setup() throws URISyntaxException {
    Path extensionFolder = Paths
            .get(ClassLoader.getSystemResource("web-extension").toURI()); ![1](img/1.png)
    zippedExtension = zipFolder(extensionFolder); ![2](img/2.png)

    driver = WebDriverManager.firefoxdriver().create();
    ((FirefoxDriver) driver).installExtension(zippedExtension, true); ![3](img/3.png)
}
```

![1](img/#co_browser_specific_manipulation_CO11-1)

我们使用位于项目类路径中的 Web 扩展源代码。

![2](img/#co_browser_specific_manipulation_CO11-2)

方法 `installExtension` 要求从其源代码安装的扩展是压缩的。WebDriverManager 提供了名为 `zipFolder(Path)` 的静态辅助方法来简化此过程。

![3](img/#co_browser_specific_manipulation_CO11-3)

我们将压缩的扩展作为临时附加组件安装到 Firefox 中。

## 地理位置

[地理位置 API](https://www.w3.org/TR/geolocation) 是一个 W3C 规范，允许访问与 Web 浏览器的主机设备（如笔记本电脑或移动设备）关联的地理位置信息。通常的地理位置数据源包括 GPS 数据和从网络推断的位置，例如 IP 地址。地理位置 API 可通过调用 JavaScript 对象 `navigator.geolocation` 在 Web 浏览器中使用。出于隐私原因，使用此语句时，会弹出提示用户允许报告位置数据。

实践站点包含一个使用地理位置的网页。图 5-5 展示了此页面的屏幕截图。该图显示了用户点击“获取坐标”按钮时显示给用户的权限弹出窗口。为了使用 Selenium WebDriver API 处理此对话框，我们使用功能。与其他情况一样，Chrome/Edge 中授予访问地理位置数据所需的功能不同于 Firefox。以下代码片段展示了这种差异。首先，示例 5-12 展示了在 Chrome 中授予地理位置访问权限的测试设置。Edge 中将使用相同的实验性偏好（`profile.default_content_setting_values.geolocation`），你可以在示例存储库中找到完整的测试。接下来，示例 5-13 展示了等效的 Firefox 测试设置。

![hosw 0505](img/hosw_0505.png)

###### 图 5-5\. 显示地理位置权限弹出窗口的实践站点

##### 示例 5-12\. 在 Chrome 中允许地理位置的测试设置

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    Map<String, Object> prefs = new HashMap<>(); ![1](img/1.png)
    prefs.put("profile.default_content_setting_values.geolocation", 1); ![2](img/2.png)
    options.setExperimentalOption("prefs", prefs); ![3](img/3.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO12-1)

我们为实验选项创建一个哈希映射。

![2](img/#co_browser_specific_manipulation_CO12-2)

我们将实验选项 `profile.default_content_setting_​val⁠ues.geolocation` 设置为 `1`，以允许访问地理位置。其他可能的值包括：默认行为的 `0` 和阻止访问地理位置数据的 `2`。

![3](img/#co_browser_specific_manipulation_CO12-3)

我们使用 Chrome 选项中的 `prefs` 标签设置实验选项。

###### 注意

假设你需要在 macOS 设备的 Chrome 或 Edge 中访问地理位置坐标。那么，你还需要在 macOS 的偏好设置（系统偏好设置 → 安全性与隐私 → 位置服务）中启用这些浏览器的位置服务。图 5-6 展示了这个配置。

![hosw 0506](img/hosw_0506.png)

###### 图 5-6\. 在 macOS 中为 Chrome 和 Edge 启用位置服务

##### 示例 5-13\. 在 Firefox 中允许地理位置的测试设置

```java
@BeforeEach
void setup() {
    FirefoxOptions options = new FirefoxOptions();
    options.addPreference("geo.enabled", true); ![1](img/1.png)
    options.addPreference("geo.prompt.testing", true); ![2](img/2.png)
    options.addPreference("geo.provider.use_corelocation", true); ![3](img/3.png)

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

![1](img/#co_browser_specific_manipulation_CO13-1)

要启用地理位置 API

![2](img/#co_browser_specific_manipulation_CO13-2)

要允许访问地理位置数据（即点击访问弹出窗口中的 `allow`）

![3](img/#co_browser_specific_manipulation_CO13-3)

要使用设备中所有可用组件（如 GPS、WiFi 或蓝牙）收集数据

## 通知

[通知 API](https://notifications.spec.whatwg.org)是一个标准的 Web API，允许网站发送显示在操作系统桌面上的通知。该 API 通过 JavaScript 对象`Notification`提供。在网站能够发送通知之前，用户必须授予权限。此授权类似于地理位置数据，在对话框弹出中提示用户同意。练习站点包含一个使用通知 API 的网页。图 5-7 显示了此页面的通知权限弹出窗口的屏幕截图。图 5-8 显示了此网页在 Linux 主机上发送的消息。

![hosw 0507](img/hosw_0507.png)

###### 图 5-7\. 练习站点显示通知权限弹出窗口

![hosw 0508](img/hosw_0508.png)

###### 图 5-8\. 练习站点在 Linux 桌面上显示通知

Selenium WebDriver API 允许通过使用能力来授予通知。与其他功能一样，这些能力的语法在 Chrome/Edge 和 Firefox 中有所不同。示例 5-14 展示了启用 Chrome 选项中通知的测试设置。我们在 Edge 中使用相同的偏好设置（`profile.default_content_setting_values.notifications`）来允许通知。示例 5-15 展示了 Firefox 的等效测试设置。在这种情况下，偏好标签（`permissions.default.desktop-notification`）不同，尽管其值（`1`）用于允许通知。另一个可能的值是`2`，用于阻止通知（在 Chrome/Edge 和 Firefox 中均适用）。

##### 示例 5-14\. 在 Chrome 中允许通知的测试设置

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    Map<String, Object> prefs = new HashMap<>();
    prefs.put("profile.default_content_setting_values.notifications", 1);
    options.setExperimentalOption("prefs", prefs);

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

##### 示例 5-15\. 在 Firefox 中允许通知的测试设置

```java
@BeforeEach
void setup() {
    FirefoxOptions options = new FirefoxOptions();
    options.addPreference("permissions.default.desktop-notification", 1);

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

示例 5-16 展示了与之前设置一起使用的测试逻辑。通常情况下，您可以在示例存储库中找到完整的测试用例。这个测试是异步脚本执行的一个例子。此脚本覆盖了原始的`Notification` JavaScript 对象。此对象的新实现获取通知消息的标题，该标题在脚本回调中返回给 WebDriver 测试。

##### 示例 5-16\. 测试处理通知

```java
@Test
void testNotifications() {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/notifications.html");
    JavascriptExecutor js = (JavascriptExecutor) driver;

    String script = String.join("\n",
            "const callback = arguments[arguments.length - 1];", ![1](img/1.png)
            "const OldNotify = window.Notification;", ![2](img/2.png)
            "function newNotification(title, options) {", ![3](img/3.png)
            "    callback(title);", ![4](img/4.png)
            "    return new OldNotify(title, options);", ![5](img/5.png)
            "}",
            "newNotification.requestPermission = " +
                    "OldNotify.requestPermission.bind(OldNotify);",
            "Object.defineProperty(newNotification, 'permission', {",
            "    get: function() {",
            "        return OldNotify.permission;",
            "    }",
            "});",
            "window.Notification = newNotification;",
            "document.getElementById('notify-me').click();"); ![6](img/6.png)
    log.debug("Executing the following script asynchronously:\n{}", script);

    Object notificationTitle = js.executeAsyncScript(script); ![7](img/7.png)
    assertThat(notificationTitle).isEqualTo("This is a notification"); ![8](img/8.png)
}
```

![1](img/#co_browser_specific_manipulation_CO14-1)

如在异步脚本执行中通常的那样，最后一个参数是用于信号化脚本终止的回调函数。

![2](img/#co_browser_specific_manipulation_CO14-2)

我们存储原始`Notification`构造函数的副本。

![3](img/#co_browser_specific_manipulation_CO14-3)

我们为通知创建一个新的构造函数。

![4](img/#co_browser_specific_manipulation_CO14-4)

我们将消息标题作为回调的一个参数传递。因此，标题将返回给 WebDriver 调用（在本例中为 Java）。

![5](img/#co_browser_specific_manipulation_CO14-5)

我们使用旧构造函数创建一个原始的`Notification`对象。

![6](img/#co_browser_specific_manipulation_CO14-6)

我们点击触发网页上通知的按钮。

![7](img/#co_browser_specific_manipulation_CO14-7)

我们获取脚本执行后返回的对象。

![8](img/#co_browser_specific_manipulation_CO14-8)

我们验证通知标题是否符合预期。

## 浏览器二进制

Selenium WebDriver 可以自动检测控制的网络浏览器（例如 Chrome、Firefox 等）的路径。尽管如此，我们可以使用 capabilities 指定浏览器可执行文件的自定义路径。当浏览器安装路径不是标准路径时（例如 beta/development/canary 浏览器的情况），这一功能非常有用。

我们使用相同的 capabilities 语法来指定 Chrome、Edge 和 Firefox 的二进制路径。示例 5-17 展示了使用 Chrome beta 的测试设置。

##### 示例 5-17\. 设置 Chrome 自定义二进制路径的测试设置

```java
@BeforeEach
void setup() {
    Path browserBinary = Paths.get("/usr/bin/google-chrome-beta"); ![1](img/1.png)
    assumeThat(browserBinary).exists(); ![2](img/2.png)

    ChromeOptions options = new ChromeOptions();
    options.setBinary(browserBinary.toFile()); ![3](img/3.png)
    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO15-1)

我们使用 Java 的 `Path` 来获取浏览器二进制路径（例如 Linux 下的 Chrome beta）。

![2](img/#co_browser_specific_manipulation_CO15-2)

我们使用假设条件，在前述路径不存在时条件性地跳过这个测试（例如在 CI 服务器上）。

![3](img/#co_browser_specific_manipulation_CO15-3)

我们使用 Chrome options 的 `setBinary` 方法来设置二进制路径（作为 Java `File`）。

## 网页代理

在计算机网络中，*代理* 是充当客户端和服务器之间中介的服务器。网页代理是浏览器和 Web 服务器之间的代理，可以用于多种目的，例如：

访问特定区域的信息

代理通常位于不同于客户端的区域，因此服务器相应地响应该区域。

避免限制

代理可以帮助访问被中间防火墙阻断的网站。

捕获网络流量

代理可以收集 HTTP 请求和响应。

缓存

代理可以加速网站检索速度。

图 5-9 展示了 Selenium WebDriver 架构中网页代理的位置，与不使用网页代理的典型场景进行了比较。可以看到，网页代理位于浏览器和测试中的网页应用程序之间，并且在 HTTP 层面上起作用。这样，网页代理允许在 Selenium WebDriver 测试中实现前面提到的目的（例如捕获 HTTP 网络流量）。

![hosw 0509](img/hosw_0509.png)

###### 图 5-9\. Selenium WebDriver 架构，包括和不包括网页代理

Selenium WebDriver API 提供了一个 `Proxy` 类来配置网页代理。这个类通过 capabilities 配置到 `WebDriver` 对象中。示例 5-18 展示了具体用法。

##### 示例 5-18\. 配置网页代理的测试设置

```java
@BeforeEach
void setup() {
    Proxy proxy = new Proxy(); ![1](img/1.png)
    String proxyStr = "proxy:port"; ![2](img/2.png)
    proxy.setHttpProxy(proxyStr); ![3](img/3.png)
    proxy.setSslProxy(proxyStr); ![4](img/4.png)

    ChromeOptions options = new ChromeOptions();
    options.setAcceptInsecureCerts(true); ![5](img/5.png)
    options.setProxy(proxy); ![6](img/6.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO16-1)

我们创建了一个 `Proxy` 类的实例。

![2](img/#co_browser_specific_manipulation_CO16-2)

指定代理的语法是`host:port`。

![3](img/#co_browser_specific_manipulation_CO16-3)

我们指定代理用于 HTTP 连接。

![4](img/#co_browser_specific_manipulation_CO16-4)

我们还指定代理用于 HTTPS 连接。

![5](img/#co_browser_specific_manipulation_CO16-5)

虽然不是强制要求，但通常需要接受不安全的证书。

![6](img/#co_browser_specific_manipulation_CO16-6)

我们将代理设置为一个能力。这一行等同于`options.setCapability(CapabilityType.PROXY, proxy);`。

###### 提示

“捕获网络流量” 展示了如何使用第三方库在 Selenium WebDriver 测试中通过使用 Web 代理来捕获网络流量。

## 日志收集

Selenium WebDriver API 允许收集不同的日志来源。这一功能通过能力来启用，尽管目前仅在基于 Chromium 的浏览器中支持。示例 5-19 展示了一个测试设置，用于收集浏览器日志（即控制台消息）。这段代码还包含了测试逻辑，在其中我们需要调用`driver.manage().logs()`来收集日志列表。

##### 示例 5-19\. 使用 Chrome 测试收集浏览器日志

```java
@BeforeEach
void setup() {
    LoggingPreferences logs = new LoggingPreferences();
    logs.enable(LogType.BROWSER, Level.ALL); ![1](img/1.png)

    ChromeOptions options = new ChromeOptions();
    options.setCapability(CapabilityType.LOGGING_PREFS, logs); ![2](img/2.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}

@Test
void testBrowserLogs() {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/console-logs.html"); ![3](img/3.png)

    LogEntries browserLogs = driver.manage().logs().get(LogType.BROWSER); ![4](img/4.png)
    Assertions.assertThat(browserLogs.getAll()).isNotEmpty(); ![5](img/5.png)
    browserLogs.forEach(l -> log.debug("{}", l)); ![6](img/6.png)
}
```

![1](img/#co_browser_specific_manipulation_CO17-1)

我们启用收集所有级别的浏览器日志。

![2](img/#co_browser_specific_manipulation_CO17-2)

我们设置`loggingPrefs`能力。

![3](img/#co_browser_specific_manipulation_CO17-3)

我们打开一个练习页面，在浏览器控制台中记录几条迹象。

![4](img/#co_browser_specific_manipulation_CO17-4)

我们收集所有日志，并按浏览器（控制台追踪）进行过滤。

![5](img/#co_browser_specific_manipulation_CO17-5)

我们验证追踪数量不为零。

![6](img/#co_browser_specific_manipulation_CO17-6)

我们将每个日志显示在标准输出中。

###### 警告

在写作时，W3C WebDriver 规范中不支持日志收集。尽管如此，某些驱动程序已经实现了这一功能，如 chromedriver 或 msedgedriver（即 Chrome 和 Edge），但在其他驱动程序如 geckodriver（即 Firefox）中不可用。

## 获取用户媒体

[WebRTC](https://webrtc.org) 是一组标准技术，允许使用 Web 浏览器交换实时媒体。这项技术允许使用客户端 JavaScript API 创建音频和视频会议的 Web 应用程序。实践站点包含一个网页，使用*getUserMedia* JavaScript API 获取用户媒体（麦克风和摄像头）。与其他 API 类似，出于安全和隐私考虑，浏览器弹出窗口在访问用户媒体前请求权限。图 5-10 展示了提示此对话框时的示例网页。

![hosw 0510](img/hosw_0510.png)

###### 图 5-10\. 弹出用户媒体权限的实践网站

我们使用能力来在 Selenium WebDriver API 中授予用户媒体访问权限。这些能力在 Chrome 和 Edge 中的语法相同（参见示例 5-20），但在 Firefox 中不同（参见示例 5-21）。

##### 示例 5-20\. 在 Chrome 中设置合成用户媒体的测试

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    options.addArguments("--use-fake-ui-for-media-stream"); ![1](img/1.png)
    options.addArguments("--use-fake-device-for-media-stream"); ![2](img/2.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

![1](img/#co_browser_specific_manipulation_CO18-1)

允许访问用户媒体（音频和视频）的参数。

![2](img/#co_browser_specific_manipulation_CO18-2)

通过合成视频（绿色旋转器）和音频（每秒钟一声蜂鸣）来伪造用户媒体的参数。您可以在图 5-11 中看到这个视频。

![hosw 0511](img/hosw_0511.png)

###### 图 5-11\. 在 Chrome 中使用合成用户媒体的实践网站

##### 示例 5-21\. 在 Firefox 中设置合成用户媒体的测试

```java
@BeforeEach
void setup() {
    FirefoxOptions options = new FirefoxOptions();
    options.addPreference("media.navigator.permission.disabled", true); ![1](img/1.png)
    options.addPreference("media.navigator.streams.fake", true); ![2](img/2.png)

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

![1](img/#co_browser_specific_manipulation_CO19-1)

优先访问用户媒体。

![2](img/#co_browser_specific_manipulation_CO19-2)

偏好于使用合成视频（背景颜色变化）和音频（恒定蜂鸣声）来伪造用户媒体。您可以在图 5-12 中看到这个视频。

![hosw 0512](img/hosw_0512.png)

###### 图 5-12\. 在 Firefox 中使用合成用户媒体的实践网站

## 加载不安全页面

当 Web 浏览器尝试加载使用 HTTPS（安全超文本传输协议）的网页，但服务器端的证书无效时，浏览器会向用户发出警告。无效证书的示例包括自签名、吊销或密码学上不安全的证书。图 5-13 展示了 Chrome 中此警告的截图。

![hosw 0513](img/hosw_0513.png)

###### 图 5-13\. 使用不安全证书的网页

这个问题并不一定意味着安全问题。例如，在开发网站时使用自签名证书可能会发生这种情况。因此，Selenium WebDriver API 允许使用`acceptInsecureCerts`能力禁用证书检查。该能力在 Chrome、Edge 和 Firefox 中都相同。示例 5-22 展示了在 Chrome 中使用此能力的测试设置。此代码片段还包含一个打开不安全网站的测试。

##### 示例 5-22\. 测试使用不安全证书的 Web 应用程序

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    options.setAcceptInsecureCerts(true); ![1](img/1.png)

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}

@Test
void testInsecure() {
    driver.get("https://self-signed.badssl.com/"); ![2](img/2.png)

    String bgColor = driver.findElement(By.tagName("body"))
            .getCssValue("background-color");
    Color red = new Color(255, 0, 0, 1);
    assertThat(Color.fromString(bgColor)).isEqualTo(red); ![3](img/3.png)
}
```

![1](img/#co_browser_specific_manipulation_CO20-1)

我们启用了允许不安全证书的能力。

![2](img/#co_browser_specific_manipulation_CO20-2)

我们打开了一个使用不安全证书（本例中为自签名证书）的网站。

![3](img/#co_browser_specific_manipulation_CO20-3)

如果网站已加载，则页面背景应为红色。

## 本地化

在软件工程中，*本地化* 指的是将应用程序调整为满足其最终用户文化和语言（称为*语言环境*）的过程。本地化有时被写作 *l10n*（10 是英语单词本地化中 *l* 和 *n* 之间的字母数）。最常见的本地化活动是将应用程序 UI 中显示的文本翻译成不同的语言。此外，根据语言环境，还可以调整其他 UI 方面，如货币（欧元、美元等）、度量系统（例如，公制或英制）、或数字和日期格式。

本地化是 *国际化*（i18n）的一部分，国际化是设计和开发支持异构目标受众轻松进行本地化的应用程序的过程。启用 i18n 的常见实践包括使用 Unicode 进行文本编码或为垂直文本或非拉丁文字体添加 CSS 支持。

*本地化测试* 是一种非功能性测试形式，用于验证特定语言环境设置下的 SUT。Selenium WebDriver API 允许我们根据浏览器语言进行本地化测试，通过设置 `intl.accept_languages` 能力。此能力允许您指定语言环境标识符，如 *en_US* 表示美式英语或 *es_ES* 表示西班牙语（欧洲），等等。示例 5-23 展示了在 Chrome 中配置此能力的测试设置。在 Edge 中可以使用相同的语法，尽管我们在 Firefox 中将此能力作为首选项进行配置（参见 示例 5-24）。

##### 示例 5-23\. 使用 Chrome 首选语言的测试

```java
String lang;

@BeforeEach
void setup() {
    lang = "es-ES";
    ChromeOptions options = new ChromeOptions();
    Map<String, Object> prefs = new HashMap<>();
    prefs.put("intl.accept_languages", lang); ![1](img/1.png)
    options.setExperimentalOption("prefs", prefs);

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}

@Test
void testAcceptLang() {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/multilanguage.html"); ![2](img/2.png)

    ResourceBundle strings = ResourceBundle.getBundle("strings",
            Locale.forLanguageTag(lang)); ![3](img/3.png)
    String home = strings.getString("home");
    String content = strings.getString("content");
    String about = strings.getString("about");
    String contact = strings.getString("contact");

    String bodyText = driver.findElement(By.tagName("body")).getText();
    assertThat(bodyText).contains(home).contains(content).contains(about)
            .contains(contact); ![4](img/4.png)
}
```

![1](img/#co_browser_specific_manipulation_CO21-1)

我们在 Chrome 中指定西班牙语（欧洲）作为首选语言。

![2](img/#co_browser_specific_manipulation_CO21-2)

我们打开一个支持多语言（英语和西班牙语）的练习页面。

![3](img/#co_browser_specific_manipulation_CO21-3)

我们使用资源包读取文本翻译。您可以在项目文件夹 `src/test/resources` 中的 `strings_es.properties`（以及 `strings_en.properties`）文件中找到这些字符串。

![4](img/#co_browser_specific_manipulation_CO21-4)

我们断言文档正文包含所有预期的字符串。

##### 示例 5-24\. 指定 Firefox 的首选语言环境测试设置

```java
@BeforeEach
void setup() {
    lang = "es-ES";
    FirefoxOptions options = new FirefoxOptions();
    options.addPreference("intl.accept_languages", lang);

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

还有另一种使用 Selenium WebDriver 进行本地化测试的选择。我们可以更改浏览器的默认语言，而不是更改首选语言（确定 HTTP 头`accept-language`）。如果该 HTTP 头不存在，多语言应用程序将交替使用浏览器语言。Selenium WebDriver API 允许使用一个简单的参数`--lang`来更改浏览器语言，该参数被指定为浏览器能力。这个参数在 Chrome、Edge 和 Firefox 中是可互操作的。示例 5-25 展示了如何使用 WebDriver 能力将浏览器语言设置为美式英语。

##### 示例 5-25\. 在 Chrome 中更改浏览器语言的测试设置

```java
@BeforeEach
void setup() {
    lang = "en-US";
    ChromeOptions options = new ChromeOptions();
    options.addArguments("--lang=" + lang);

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

## 隐身

隐身模式确保浏览器以干净的状态运行。这种模式允许私密浏览，即与主会话和用户数据隔离运行。Selenium WebDriver API 通过能力启用了在隐身模式下执行浏览器的功能。对于 Chrome 和 Edge，这种模式是通过使用`--incognito`参数激活的（参见示例 5-26），而对于 Firefox，则使用`-private`偏好设置（参见示例 5-27）。

##### 示例 5-26\. 使用 Chrome 在隐身模式下的测试设置

```java
@BeforeEach
void setup() {
    ChromeOptions options = new ChromeOptions();
    options.addArguments("--incognito");

    driver = WebDriverManager.chromedriver().capabilities(options).create();
}
```

##### 示例 5-27\. 使用 Firefox 在隐身模式下的测试设置

```java
@BeforeEach
void setup() {
    FirefoxOptions options = new FirefoxOptions();
    options.addArguments("-private");

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

## IE 模式中的 Edge

Edge 为微软传统浏览器即 Internet Explorer（IE）提供了内置支持。因此，要创建一个使用 IE 模式中的 Edge 的 Selenium WebDriver 测试，我们需要先在 Edge 中启用 IE 模式。如图 5-14 所示，这个选项是在 Edge 设置→默认浏览器→允许在 Internet Explorer 模式下重新加载网站中启用的。然后，我们可以使用 Selenium WebDriver API，如示例 5-28 所示。

![hosw 0514](img/hosw_0514.png)

###### 图 5-14\. 启用 IE 模式中 Edge 的浏览器设置

##### 示例 5-28\. 使用 IE 模式中的 Edge 进行测试设置

```java
@BeforeAll
static void setupClass() {
    assumeThat(IS_OS_WINDOWS).isTrue(); ![1](img/1.png)
    WebDriverManager.iedriver().setup(); ![2](img/2.png)
}

@BeforeEach
void setup() {
    Optional<Path> browserPath = WebDriverManager.edgedriver()
            .getBrowserPath(); ![3](img/3.png)
    assumeThat(browserPath).isPresent();

    InternetExplorerOptions options = new InternetExplorerOptions();
    options.attachToEdgeChrome(); ![4](img/4.png)
    options.withEdgeExecutablePath(browserPath.get().toString()); ![5](img/5.png)

    driver = new InternetExplorerDriver(options); ![6](img/6.png)
}
```

![1](img/#co_browser_specific_manipulation_CO22-1)

我们假设测试在 Windows 中执行，因为 IE 模式不支持其他操作系统。

![2](img/#co_browser_specific_manipulation_CO22-2)

我们使用 WebDriverManager 来管理 IEDriver（Internet Explorer 所需的驱动程序）。

![3](img/#co_browser_specific_manipulation_CO22-3)

我们使用 WebDriverManager 来发现 Edge 的路径。

![4](img/#co_browser_specific_manipulation_CO22-4)

我们使用 IE 选项来指定我们使用 IE 模式中的 Edge。

![5](img/#co_browser_specific_manipulation_CO22-5)

我们在 IE 选项中设置了先前发现的 Edge 路径。

![6](img/#co_browser_specific_manipulation_CO22-6)

我们创建驱动程序实例来使用 Internet Explorer（实际上将是 IE 模式中的 Edge）。

# Chrome 开发者工具协议

[Chrome DevTools](https://developer.chrome.com/docs/devtools) 是一组针对基于 Chromium 的浏览器（如 Chrome 和 Edge）的 web 开发工具。这些工具允许检查、调试或分析这些浏览器，以及其他功能。[Chrome DevTools Protocol (CDP)](https://chromedevtools.github.io/devtools-protocol) 是一种通信协议，允许外部客户端操作 Chrome DevTools。Firefox 实现了 CDP 的子集，以支持像 Selenium WebDriver 这样的自动化工具。

在 Selenium WebDriver 中使用 CDP 有两种方式。从版本 4 开始，Selenium WebDriver 提供了 `HasDevTools` 接口，用于向浏览器发送 CDP 命令。这个接口由 `ChromiumDriver`（用于 Chrome 和 Edge）和 `FirefoxDriver`（用于 Firefox）实现。这种机制非常强大，因为它直接提供了与 Selenium WebDriver 结合使用 CDP 的访问权限。然而，它也有一个重要的限制，即与浏览器类型和版本紧密耦合。

因此，Selenium WebDriver API 提供了第二种使用 Chrome DevTools 协议（CDP）的方式，基于一组在浏览器上构建的包装类，用于高级浏览器操作。这些包装类允许执行不同的操作，如网络流量拦截或基本和摘要认证。下面的子节解释了这些包装类。之后，我会展示几个直接使用 CDP 命令的例子。

## CDP Selenium 包装类

Selenium WebDriver API 包含一组辅助类，包装了部分 CDP 命令。这些类旨在为 Selenium WebDriver 测试提供友好的 API，支持高级功能。

### 网络拦截器

第一个构建在 CDP 之上的包装类称为 `NetworkInterceptor`。这个类允许桩后端请求、拦截网络流量并返回预先准备好的响应。这个特性可能通过使用快速、直接的响应来模拟外部调用，简化复杂的端到端测试。要实例化 `NetworkInterceptor`，需要在其构造函数中指定参数（参见 示例 5-29）：

+   实现了 CDP 的 `WebDriver` 对象（例如 `ChromeDriver` 或 `EdgeDriver`）

+   `Route` 对象用于将网络请求映射到响应

##### 示例 5-29\. 使用 NetworkInterceptor 测试拦截网络流量

```java
@Test
void testNetworkInterceptor() throws Exception {
    Path img = Paths
            .get(ClassLoader.getSystemResource("tools.png").toURI()); ![1](img/1.png)
    byte[] bytes = Files.readAllBytes(img);

    try (NetworkInterceptor interceptor = new NetworkInterceptor(driver,
            Route.matching(req -> req.getUri().endsWith(".png"))
                    .to(() -> req -> new HttpResponse()
                            .setContent(Contents.bytes(bytes))))) { ![2](img/2.png)
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/"); ![3](img/3.png)

        int width = Integer.parseInt(driver.findElement(By.tagName("img"))
                .getAttribute("width"));
        assertThat(width).isGreaterThan(80); ![4](img/4.png)
    }
}
```

![1](img/#co_browser_specific_manipulation_CO23-1)

我们加载一个存储在 Java 项目中作为测试资源的本地图像。

![2](img/#co_browser_specific_manipulation_CO23-2)

我们创建一个网络拦截器实例，为所有以 `.png` 结尾的请求创建一个路由，并通过新的响应打桩这个请求，在这种情况下发送前一张图片的内容。

![3](img/#co_browser_specific_manipulation_CO23-3)

我们打开实践站点。

![4](img/#co_browser_specific_manipulation_CO23-4)

如果拦截正常工作，页面上的图片应该比原始的徽标宽。

###### 注意

如果使用与前述代码不同的浏览器（如 Firefox），将抛出`DevToolsException`异常。

### 基本认证和摘要认证

HTTP 提供了两种内置机制来识别用户身份，称为*基本*和*摘要*认证。这两种方法允许使用一对值（用户名和密码）来指定用户的凭据。它们之间的区别在于它们如何传递凭据。一方面，摘要认证方法通过将用户名和密码应用哈希函数发送加密的凭据。另一方面，基本认证使用 Base64 编码（而非加密）凭据。

Selenium WebDriver 提供了`HasAuthentication`接口以无缝实现基本和摘要认证。示例 5-30 展示了使用 Chrome 和基本认证的测试。您可以在 Edge 和摘要认证中使用相同的机制（详见[示例库](https://github.com/bonigarcia/selenium-webdriver-java)的完整测试）。

##### 示例 5-30\. 使用 Chrome 进行基本认证测试

```java
@Test
void testBasicAuth() {
    ((HasAuthentication) driver)
            .register(() -> new UsernameAndPassword("guest", "guest")); ![1](img/1.png)

    driver.get("https://jigsaw.w3.org/HTTP/Basic/"); ![2](img/2.png)

    WebElement body = driver.findElement(By.tagName("body"));
    assertThat(body.getText()).contains("Your browser made it!"); ![3](img/3.png)
}
```

![1](img/#co_browser_specific_manipulation_CO24-1)

我们将驱动对象转换为`HasAuthentication`并注册凭据（用户名和密码）。

![2](img/#co_browser_specific_manipulation_CO24-2)

我们打开了一个使用基本认证保护的网站。

![3](img/#co_browser_specific_manipulation_CO24-3)

我们验证页面内容是否可用。

当使用其他浏览器（如 Firefox）时，我们无法将驱动对象转换为`HasAuthentication`。尽管如此，可以使用 URL 中的语法`protocol://username:password@domain`来发送凭据。示例 5-31 展示了此用法。

##### 示例 5-31\. 使用基本认证和 Firefox 进行测试

```java
@Test
void testGenericAuth() {
    driver.get("https://guest:guest@jigsaw.w3.org/HTTP/Basic/");

    WebElement body = driver.findElement(By.tagName("body"));
    assertThat(body.getText()).contains("Your browser made it!");
}
```

## CDP 原始命令

自版本 4 起，Selenium WebDriver 提供了`HasDevTools`接口来直接使用 CDP。该接口由`ChromiumDriver`（用于 Chrome 和 Edge）和`FirefoxDriver`（用于 Firefox）实现。要使用此功能，我们首先需要使用`DevTools`实例的`createSession()`方法打开 CDP 会话（即客户端与浏览器之间的 WebSocket 连接）。示例 5-32 展示了在 Selenium WebDriver 测试中使用 CDP 的推荐结构。正如您所见，CDP 会话在测试设置中创建，并在拆卸时关闭。每个测试将使用类属性`devTools`与 Chrome DevTools 交互。

##### 示例 5-32\. 使用 Chrome DevTools 进行测试结构

```java
WebDriver driver;

DevTools devTools; ![1](img/1.png)

@BeforeEach
void setup() {
    driver = WebDriverManager.chromedriver().create();
    devTools = ((ChromeDriver) driver).getDevTools(); ![2](img/2.png)
    devTools.createSession(); ![3](img/3.png)
}

@AfterEach
void teardown() {
    devTools.close(); ![4](img/4.png)
    driver.quit();
}
```

![1](img/#co_browser_specific_manipulation_CO25-1)

我们声明了一个`DevTools`类属性。

![2](img/#co_browser_specific_manipulation_CO25-2)

我们从驱动对象中获取`DevTools`实例。在本示例（以及其他示例中），我使用`ChromeDriver`（尽管`EdgeDriver`实例也是有效的）。

![3](img/#co_browser_specific_manipulation_CO25-3)

我们创建一个 CDP 会话以在测试逻辑中与 Chrome DevTools 交互。

![4](img/#co_browser_specific_manipulation_CO25-4)

每次测试结束并退出 WebDriver 会话之前，我们终止 CDP 会话。

下面的小节展示了几个示例，说明了 WebDriver 测试中 DevTools 的潜力。在这些示例中，我们使用 `DevTools` 实例通过 `send()` 方法发送 CDP 命令。Selenium WebDriver API 提供了各种命令，允许进行不同的操作，如模拟网络条件、处理 HTTP 标头、阻止 URL 等。

###### 警告

使用原始 CDP 命令的 Selenium WebDriver 测试（如下一节中所述）与特定的浏览器版本相关联。您可以通过检查导入子句（例如 `import org.openqa.selenium.devtools.v96.*;`）来查看此版本，在示例存储库中提供了完整的测试。

### 模拟网络条件。

CDP 允许模拟不同网络（如移动 2G/3G/4G、WiFi 或蓝牙等）和条件（例如延迟或吞吐量）。这个功能对于测试特定连接参数下的 Web 应用行为非常有用。示例 5-33 展示了使用此功能的测试。如您所见，此测试发送了两个 CDP 命令：

`Network.enable()`

要激活网络跟踪。此命令有三个可选参数：

`Optional<Integer> maxTotalBufferSize`

网络有效载荷的最大缓冲区大小（以字节为单位）。

`Optional<Integer> maxResourceBufferSize`

单个资源的最大缓冲区大小（以字节为单位）。

`Optional<Integer> maxPostDataSize`

最长的请求正文大小（以字节为单位）。

`Network.emulateNetworkConditions()`

要激活网络仿真。使用以下参数指定仿真条件：

`Boolean offline`

要模拟无互联网连接。 `Number latency`：请求到响应的最小延迟（以毫秒为单位）。

`Number downloadThroughput`

最大下载吞吐量（以字节/秒为单位）。`-1` 禁用下载限制。

`Number uploadThroughput`

最大上传吞吐量（以字节/秒为单位）。`-1` 禁用上传限制。

`Optional<ConnectionType> connectionType`

模拟连接技术。枚举 `ConnectionType` 接受以下选项：`NONE`、`CELLULAR2G`、`CELLULAR3G`、`CELLULAR4G`、`BLUETOOTH`、`ETHERNET`、`WIFI`、`WIMAX` 和 `OTHER`。

##### 示例 5-33\. 测试模拟网络条件

```java
@Test
void testEmulateNetworkConditions() {
    devTools.send(Network.enable(Optional.empty(), Optional.empty(),
            Optional.empty())); ![1](img/1.png)
    devTools.send(Network.emulateNetworkConditions(false, 100, 50 * 1024,
            50 * 1024, Optional.of(ConnectionType.CELLULAR3G))); ![2](img/2.png)

    long initMillis = System.currentTimeMillis(); ![3](img/3.png)
    driver.get("https://bonigarcia.dev/selenium-webdriver-java/"); ![4](img/4.png)
    Duration elapsed = Duration
            .ofMillis(System.currentTimeMillis() - initMillis); ![5](img/5.png)
    log.debug("The page took {} ms to be loaded", elapsed.toMillis());

    assertThat(driver.getTitle()).contains("Selenium WebDriver");
}
```

![1](img/#co_browser_specific_manipulation_CO26-1)

我们激活网络跟踪（无需调整任何网络参数）。

![2](img/#co_browser_specific_manipulation_CO26-2)

我们使用 50 KBps 的移动 3G 网络模拟下载和上传带宽。

![3](img/#co_browser_specific_manipulation_CO26-3)

我们在加载网页之前获取系统时间戳。

![4](img/#co_browser_specific_manipulation_CO26-4)

我们加载实践站点的索引页面。

![5](img/#co_browser_specific_manipulation_CO26-5)

我们计算加载此页面所需的时间。

### 网络监控

我们还可以使用 CDP 在与网页交互时监控网络流量。示例 5-34 展示了使用此功能的测试。此测试使用 `DevTools` 对象的 `add​Lis⁠tener()` 方法来跟踪 HTTP 请求和响应。

##### 示例 5-34\. 监控 HTTP 请求和响应的测试

```java
@Test
void testNetworkMonitoring() {
    devTools.send(Network.enable(Optional.empty(), Optional.empty(),
            Optional.empty()));

    devTools.addListener(Network.requestWillBeSent(), request -> {
        log.debug("Request {}", request.getRequestId());
        log.debug("\t Method: {}", request.getRequest().getMethod());
        log.debug("\t URL: {}", request.getRequest().getUrl());
        logHeaders(request.getRequest().getHeaders());
    }); ![1](img/1.png)

    devTools.addListener(Network.responseReceived(), response -> {
        log.debug("Response {}", response.getRequestId());
        log.debug("\t URL: {}", response.getResponse().getUrl());
        log.debug("\t Status: {}", response.getResponse().getStatus());
        logHeaders(response.getResponse().getHeaders());
    }); ![2](img/2.png)

    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
    assertThat(driver.getTitle()).contains("Selenium WebDriver");
}

void logHeaders(Headers headers) {
    log.debug("\t Headers:");
    headers.toJson().forEach((k, v) -> log.debug("\t\t{}:{}", k, v));
}
```

![1](img/#co_browser_specific_manipulation_CO27-1)

我们创建一个 HTTP 请求监听器，并在控制台中记录捕获的数据。

![2](img/#co_browser_specific_manipulation_CO27-2)

我们创建一个 HTTP 响应监听器，并在控制台中记录捕获的数据。

### 整页截图

CDP 的另一个可能用途是制作整页截图（即捕获超出视口的内容页面）。示例 5-35 在 Chrome 中展示了此功能。

##### 示例 5-35\. 在 Chrome 中使用 CDP 制作整页截图的测试

```java
@Test
void testFullPageScreenshotChrome() throws IOException {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/long-page.html"); ![1](img/1.png)
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.presenceOfNestedElementsLocatedBy(
            By.className("container"), By.tagName("p"))); ![2](img/2.png)

    GetLayoutMetricsResponse metrics = devTools
            .send(Page.getLayoutMetrics());
    Rect contentSize = metrics.getContentSize(); ![3](img/3.png)
    String screenshotBase64 = devTools
            .send(Page.captureScreenshot(Optional.empty(), Optional.empty(),
                    Optional.of(new Viewport(0, 0, contentSize.getWidth(),
                            contentSize.getHeight(), 1)),
                    Optional.empty(), Optional.of(true))); ![4](img/4.png)
    Path destination = Paths.get("fullpage-screenshot-chrome.png");
    Files.write(destination, Base64.getDecoder().decode(screenshotBase64)); ![5](img/5.png)

    assertThat(destination).exists(); ![6](img/6.png)
}
```

![1](img/#co_browser_specific_manipulation_CO28-1)

我们加载包含长文本的练习页面（因此其内容超出标准视口）。

![2](img/#co_browser_specific_manipulation_CO28-2)

我们等待段落加载完成。

![3](img/#co_browser_specific_manipulation_CO28-3)

我们获取页面布局指标（以计算页面尺寸）。

![4](img/#co_browser_specific_manipulation_CO28-4)

我们发送 CDP 命令以截取超出页面视口的屏幕截图。结果，我们将截图以 Base64 字符串形式获取。

![5](img/#co_browser_specific_manipulation_CO28-5)

我们将 Base64 内容解码为 PNG 文件。

![6](img/#co_browser_specific_manipulation_CO28-6)

我们断言测试结束时 PNG 文件存在。

此功能在其他完全实现了 CDP 的浏览器（如 Chrome 或 Edge）中可用。然而，在 Firefox 等其他浏览器中可能不可用。幸运的是，Firefox 通过 `FirefoxDriver` 对象中可用的 `getFullPageScreenshotAs()` 方法支持相同的特性。示例 5-36 展示了使用此方法和 Firefox 进行的测试。

##### 示例 5-36\. 在 Firefox 中制作整页截图的测试

```java
@Test
void testFullPageScreenshotFirefox() throws IOException {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/long-page.html");
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    wait.until(ExpectedConditions.presenceOfNestedElementsLocatedBy(
            By.className("container"), By.tagName("p")));

    byte[] imageBytes = ((FirefoxDriver) driver)
            .getFullPageScreenshotAs(OutputType.BYTES); ![1](img/1.png)
    Path destination = Paths.get("fullpage-screenshot-firefox.png");
    Files.write(destination, imageBytes);

    assertThat(destination).exists();
}
```

![1](img/#co_browser_specific_manipulation_CO29-1)

我们制作整个页面的截图。与常规截图一样（见第四章中的表 4-2），输出类型可以是 `FILE`、`BASE64` 或 `BYTES`。我们使用后者将截图获取为字节数组。

### 性能指标

CDP 允许收集运行时性能指标，例如加载的文档数、DOM 节点数、加载 DOM 的时间以及脚本持续时间等，示例 5-37 展示了一个收集这些指标并在标准输出中显示的测试。

##### 例子 5-37\. 测试收集性能指标

```java
@Test
void testPerformanceMetrics() {
    devTools.send(Performance.enable(Optional.empty())); ![1](img/1.png)
    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");

    List<Metric> metrics = devTools.send(Performance.getMetrics()); ![2](img/2.png)
    assertThat(metrics).isNotEmpty();
    metrics.forEach(metric -> log.debug("{}: {}", metric.getName(),
            metric.getValue()));
}
```

![1](img/#co_browser_specific_manipulation_CO30-1)

我们启用收集指标。

![2](img/#co_browser_specific_manipulation_CO30-2)

我们收集所有指标。

### 额外的头部

CDP 允许在 HTTP 层面添加额外的头部。为此，我们需要在 CDP 会话中发送命令 `Network.setExtraHTTPHeaders()`。例子 5-38 展示了一个使用该命令来添加 HTTP 头部 `Authorization` 的测试，用于在需要基本认证登录的网页中发送凭据（用户名和密码）。

##### 例子 5-38\. 测试添加额外的 HTTP 头部

```java
@Test
void testExtraHeaders() {
    devTools.send(Network.enable(Optional.empty(), Optional.empty(),
            Optional.empty()));

    String userName = "guest";
    String password = "guest";
    Map<String, Object> headers = new HashMap<>();
    String basicAuth = "Basic " + new String(Base64.getEncoder()
            .encode(String.format("%s:%s", userName, password).getBytes()));
    headers.put("Authorization", basicAuth); ![1](img/1.png)
    devTools.send(Network.setExtraHTTPHeaders(new Headers(headers))); ![2](img/2.png)

    driver.get("https://jigsaw.w3.org/HTTP/Basic/"); ![3](img/3.png)
    String bodyText = driver.findElement(By.tagName("body")).getText();
    assertThat(bodyText).contains("Your browser made it!"); ![4](img/4.png)
}
```

![1](img/#co_browser_specific_manipulation_CO31-1)

我们将用户名和密码进行 Base64 编码。

![2](img/#co_browser_specific_manipulation_CO31-2)

我们创建授权头部。

![3](img/#co_browser_specific_manipulation_CO31-3)

我们打开一个使用基本认证保护的网页。

![4](img/#co_browser_specific_manipulation_CO31-4)

我们验证页面是否正确显示。

### 阻止 URL

CDP 提供了在会话中阻止给定 URL 的能力。例子 5-39 提供了一个测试，阻止练习网页的 logo URL。如果你运行这个测试并在执行期间检查浏览器，你会发现该 logo 不会显示在页面上。

##### 例子 5-39\. 测试阻止 URL

```java
@Test
void testBlockUrl() {
    devTools.send(Network.enable(Optional.empty(), Optional.empty(),
            Optional.empty()));

    String urlToBlock =
            "https://bonigarcia.dev/selenium-webdriver-java/img/hands-on-icon.png";
    devTools.send(Network.setBlockedURLs(ImmutableList.of(urlToBlock))); ![1](img/1.png)

    devTools.addListener(Network.loadingFailed(), loadingFailed -> {
        BlockedReason reason = loadingFailed.getBlockedReason().get();
        log.debug("Blocking reason: {}", reason);
        assertThat(reason).isEqualTo(BlockedReason.INSPECTOR);
    }); ![2](img/2.png)

    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
    assertThat(driver.getTitle()).contains("Selenium WebDriver");
}
```

![1](img/#co_browser_specific_manipulation_CO32-1)

我们阻止给定的 URL。

![2](img/#co_browser_specific_manipulation_CO32-2)

我们创建一个监听器以追踪失败的事件。

### 设备模拟

CDP 提供的另一个功能是模拟移动设备（例如智能手机、平板电脑）的能力。例子 5-40 说明了这个用法。该测试首先通过发送命令 `Network.setUserAgentOverride()` 来覆盖用户代理，然后通过发送命令 `Emulation.setDeviceMetricsOverride` 来模拟设备指标。

##### 例子 5-40\. 测试模拟移动设备

```java
@Test
void testDeviceEmulation() {
    // 1\. Override user agent (Apple iPhone 6)
    String userAgent = "Mozilla/5.0 (iPhone; CPU iPhone OS 8_0 like Mac OS X)"
            + "AppleWebKit/600.1.3 (KHTML, like Gecko)"
            + "Version/8.0 Mobile/12A4345d Safari/600.1.4";
    devTools.send(Network.setUserAgentOverride(userAgent, Optional.empty(),
            Optional.empty(), Optional.empty())); ![1](img/1.png)

    // 2\. Emulate device dimension
    Map<String, Object> deviceMetrics = new HashMap<>();
    deviceMetrics.put("width", 375);
    deviceMetrics.put("height", 667);
    deviceMetrics.put("mobile", true);
    deviceMetrics.put("deviceScaleFactor", 2);
    ((ChromeDriver) driver).executeCdpCommand(
            "Emulation.setDeviceMetricsOverride", deviceMetrics); ![2](img/2.png)

    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
    assertThat(driver.getTitle()).contains("Selenium WebDriver");
}
```

![1](img/#co_browser_specific_manipulation_CO33-1)

我们覆盖用户代理以模拟苹果 iPhone 6。

![2](img/#co_browser_specific_manipulation_CO33-2)

我们覆盖设备屏幕参数。

### 控制台监听器

CDP 允许您实现监听器以监视控制台事件，即网页 JavaScript 的日志和错误追踪。例子 5-41 展示了这个测试。该测试使用了一个在练习站点中意图追踪多个 JavaScript 消息（使用命令 `console.log()`、`console.error()` 等）并抛出 JavaScript 异常的网页。

##### 例子 5-41\. 测试监听控制台事件

```java
@Test
void testConsoleListener() throws Exception {
    CompletableFuture<ConsoleEvent> futureEvents = new CompletableFuture<>();
    devTools.getDomains().events()
            .addConsoleListener(futureEvents::complete); ![1](img/1.png)

    CompletableFuture<JavascriptException> futureJsExc = new CompletableFuture<>();
    devTools.getDomains().events()
            .addJavascriptExceptionListener(futureJsExc::complete); ![2](img/2.png)

    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/console-logs.html"); ![3](img/3.png)

    ConsoleEvent consoleEvent = futureEvents.get(5, TimeUnit.SECONDS); ![4](img/4.png)
    log.debug("ConsoleEvent: {} {} {}", consoleEvent.getTimestamp(),
            consoleEvent.getType(), consoleEvent.getMessages()); ![5](img/5.png)

    JavascriptException jsException = futureJsExc.get(5,
            TimeUnit.SECONDS); ![6](img/6.png)
    log.debug("JavascriptException: {} {}", jsException.getMessage(),
            jsException.getSystemInformation());
}
```

![1](img/#co_browser_specific_manipulation_CO34-1)

我们创建一个控制台事件监听器。

![2](img/#co_browser_specific_manipulation_CO34-2)

我们为 JavaScript 错误创建另一个监听器。

![3](img/#co_browser_specific_manipulation_CO34-3)

我们打开一个在浏览器控制台中写入消息的练习页面。

![4](img/#co_browser_specific_manipulation_CO34-4)

我们等待最多五秒，直到收到一个控制台事件。

![5](img/#co_browser_specific_manipulation_CO34-5)

我们在标准输出中写入接收到的控制台事件的信息。

![6](img/#co_browser_specific_manipulation_CO34-6)

我们为 JavaScript 异常重复相同的过程。

### 地理位置覆盖

CDP 提供的另一个功能是能够覆盖主机设备处理的地理位置坐标。示例 5-42 展示了如何做到这一点。此测试发送命令 `Emulation.setGeolocationOverride()`，接受三个可选参数：纬度、经度和准确度。

##### 示例 5-42。覆盖位置坐标的测试

```java
@Test
void testGeolocationOverride() {
    devTools.send(Emulation.setGeolocationOverride(Optional.of(48.8584),
            Optional.of(2.2945), Optional.of(100))); ![1](img/1.png)

    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/geolocation.html"); ![2](img/2.png)
    driver.findElement(By.id("get-coordinates")).click();

    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    WebElement coordinates = driver.findElement(By.id("coordinates"));
    wait.until(ExpectedConditions.visibilityOf(coordinates));
}
```

![1](img/#co_browser_specific_manipulation_CO35-1)

我们使用埃菲尔铁塔的坐标（法国巴黎）覆盖地理位置。

![2](img/#co_browser_specific_manipulation_CO35-2)

我们打开一个访问设备位置并向用户显示坐标的练习网页。

### 管理 cookies

CDP 还允许管理 Web cookies。示例 5-43 显示了读取管理一些 cookies 的练习页面的测试。

##### 示例 5-43。管理 cookies 的测试

```java
@Test
void testManageCookies() {
    devTools.send(Network.enable(Optional.empty(), Optional.empty(),
            Optional.empty()));
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/cookies.html");

    // Read cookies
    List<Cookie> cookies = devTools.send(Network.getAllCookies()); ![1](img/1.png)
    cookies.forEach(cookie -> log.debug("{}={}", cookie.getName(),
            cookie.getValue()));
    List<String> cookieName = cookies.stream()
            .map(cookie -> cookie.getName()).sorted()
            .collect(Collectors.toList());
    Set<org.openqa.selenium.Cookie> seleniumCookie = driver.manage()
            .getCookies();
    List<String> selCookieName = seleniumCookie.stream()
            .map(selCookie -> selCookie.getName()).sorted()
            .collect(Collectors.toList());
    assertThat(cookieName).isEqualTo(selCookieName); ![2](img/2.png)

    // Clear cookies
    devTools.send(Network.clearBrowserCookies()); ![3](img/3.png)
    List<Cookie> cookiesAfterClearing = devTools
            .send(Network.getAllCookies());
    assertThat(cookiesAfterClearing).isEmpty(); ![4](img/4.png)

    driver.findElement(By.id("refresh-cookies")).click();
}
```

![1](img/#co_browser_specific_manipulation_CO36-1)

我们读取网页的所有 cookies。

![2](img/#co_browser_specific_manipulation_CO36-2)

我们验证使用 CDP 命令读取的 cookies 和使用 Selenium WebDriver API（使用 `getCookies();`）读取的 cookies 是否相同。

![3](img/#co_browser_specific_manipulation_CO36-3)

我们移除所有 cookies。

![4](img/#co_browser_specific_manipulation_CO36-4)

我们验证此时没有任何 cookies。

### 加载不安全的页面

CDP 还允许您加载不安全的网页（即，使用 HTTPS 的网页，但其证书无效）。示例 5-44 说明了这个功能。

##### 示例 5-44。测试加载不安全的网页

```java
@Test
void testLoadInsecure() {
    devTools.send(Security.enable()); ![1](img/1.png)
    devTools.send(Security.setIgnoreCertificateErrors(true)); ![2](img/2.png)
    driver.get("https://expired.badssl.com/");

    String bgColor = driver.findElement(By.tagName("body"))
            .getCssValue("background-color");
    Color red = new Color(255, 0, 0, 1);
    assertThat(Color.fromString(bgColor)).isEqualTo(red); ![3](img/3.png)
}
```

![1](img/#co_browser_specific_manipulation_CO37-1)

我们启用跟踪安全性。

![2](img/#co_browser_specific_manipulation_CO37-2)

我们忽略证书错误。

![3](img/#co_browser_specific_manipulation_CO37-3)

我们验证页面是否正确加载。

# 位置上下文

Selenium WebDriver API 为模拟用户设备地理位置坐标提供了接口 `LocationContext`。此接口由 `ChromeDriver`、`EdgeDriver` 和 `OperaDriver` 实现。因此，这些驱动程序可以调用方法 `setLocation()` 来指定自定义坐标（纬度、经度和高度）。示例 5-45 展示了使用此功能的基本测试。

##### 示例 5-45。通过 LocationContext 设置自定义地理位置坐标的测试

```java
@Test
void testLocationContext() {
    LocationContext location = (LocationContext) driver; ![1](img/1.png)
    location.setLocation(new Location(27.5916, 86.5640, 8850)); ![2](img/2.png)

    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/geolocation.html"); ![3](img/3.png)
    driver.findElement(By.id("get-coordinates")).click();

    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(5));
    WebElement coordinates = driver.findElement(By.id("coordinates"));
    wait.until(ExpectedConditions.visibilityOf(coordinates)); ![4](img/4.png)
}
```

![1](img/#co_browser_specific_manipulation_CO38-1)

我们将驱动对象转换为`LocationContext`（仅适用于 Chrome、Edge 或 Opera）。

![2](img/#co_browser_specific_manipulation_CO38-2)

我们打开一个实践页面，显示地理位置坐标给最终用户。

![3](img/#co_browser_specific_manipulation_CO38-3)

我们设置自定义位置，即珠穆朗玛峰的坐标（位于尼泊尔与中国边境上）。

![4](img/#co_browser_specific_manipulation_CO38-4)

我们断言页面上可见坐标。

# Web 认证

Web 认证 API（也称为*WebAuthn*）是一项[W3C 规范](https://www.w3.org/TR/webauthn-2)，允许服务器使用公钥加密而不是密码来注册和认证用户。自 2019 年 1 月起，主要浏览器（Chrome、Firefox、Edge 和 Safari）已支持 WebAuthn。这些浏览器允许使用 U2F（Universal 2nd Factor）令牌进行凭据创建和断言，这些令牌是通用串行总线（USB）或近场通信（NFC）安全设备。

在传统的 Web 认证方法中，用户通过 Web 表单将其用户名和密码发送到服务器。在 WebAuthn 中，Web 服务器使用 Web 认证 API 提示用户创建私钥-公钥对（称为*凭据*）。私钥安全存储在用户设备上，而公钥则发送到服务器。然后，服务器可以使用该公钥验证用户身份。

从版本 4 开始，Selenium WebDriver 直接支持*WebAuthn*。为此，Selenium WebDriver API 提供了`HasVirtualAuthenticator`接口。这个接口允许我们使用虚拟认证器，而不是使用安全物理设备。虽然`RemoteWebDriver`类实现了这个接口，在撰写本文时，这种机制仅在基于 Chromium 的浏览器（例如 Chrome 和 Edge）中受支持。示例 5-46 展示了使用 Web 认证 API 的测试。

##### 示例 5-46\. 使用 WebAuthn 进行测试

```java
@Test
void testWebAuthn() {
    driver.get("https://webauthn.io/"); ![1](img/1.png)
    HasVirtualAuthenticator virtualAuth = (HasVirtualAuthenticator) driver; ![2](img/2.png)
    VirtualAuthenticator authenticator = virtualAuth
            .addVirtualAuthenticator(new VirtualAuthenticatorOptions()); ![3](img/3.png)

    String randomId = UUID.randomUUID().toString();
    driver.findElement(By.id("input-email")).sendKeys(randomId); ![4](img/4.png)
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(20));
    driver.findElement(By.id("register-button")).click(); ![5](img/5.png)
    wait.until(ExpectedConditions.textToBePresentInElementLocated(
            By.className("popover-body"), "Success! Now try logging in"));

    driver.findElement(By.id("login-button")).click(); ![6](img/6.png)
    wait.until(ExpectedConditions.textToBePresentInElementLocated(
            By.className("main-content"), "You're logged in!")); ![7](img/7.png)

    virtualAuth.removeVirtualAuthenticator(authenticator); ![8](img/8.png)
}
```

![1](img/#co_browser_specific_manipulation_CO39-1)

我们打开一个使用 Web 认证 API 保护的网站。

![2](img/#co_browser_specific_manipulation_CO39-2)

我们将驱动对象转换为`HasVirtualAuthenticator`。

![3](img/#co_browser_specific_manipulation_CO39-3)

我们创建并注册一个新的虚拟认证器。

![4](img/#co_browser_specific_manipulation_CO39-4)

我们在 Web 表单中发送一个随机标识符。

![5](img/#co_browser_specific_manipulation_CO39-5)

我们提交该标识符并等待接收。

![6](img/#co_browser_specific_manipulation_CO39-6)

我们点击按钮登录。

![7](img/#co_browser_specific_manipulation_CO39-7)

我们验证认证已正确执行。

![8](img/#co_browser_specific_manipulation_CO39-8)

我们移除虚拟认证器。

# 打印页面

Selenium WebDriver 允许将网页打印为 PDF 文档。为此，Selenium WebDriver API 提供了`PrintsPage`接口。这个接口由类`RemoteWebDriver`继承，因此，它适用于 Selenium WebDriver 支持的所有浏览器。然而，当使用不同浏览器时存在细微差异。例如，只有在 Chrome 和 Edge 中以无头模式启动浏览器时才能打印页面。对于 Firefox，则不需要此限制，我们可以像平常一样使用 Firefox。示例 5-47 展示了将网页打印为 PDF 的测试逻辑。您可以在[示例存储库](https://github.com/bonigarcia/selenium-webdriver-java)中找到 Firefox 和无头 Chrome/Edge 的完整测试。

##### 示例 5-47\. 测试将网页打印为 PDF

```java
@Test
void testPrint() throws IOException {
    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
    PrintsPage pg = (PrintsPage) driver; ![1](img/1.png)
    PrintOptions printOptions = new PrintOptions();
    Pdf pdf = pg.print(printOptions); ![2](img/2.png)

    String pdfBase64 = pdf.getContent(); ![3](img/3.png)
    assertThat(pdfBase64).contains("JVBER"); ![4](img/4.png)

    byte[] decodedImg = Base64.getDecoder()
            .decode(pdfBase64.getBytes(StandardCharsets.UTF_8)); ![5](img/5.png)
    Path destinationFile = Paths.get("my-pdf.pdf");
    Files.write(destinationFile, decodedImg); ![6](img/6.png)
}
```

![1](img/#co_browser_specific_manipulation_CO40-1)

我们将驱动对象转换为`PrintsPage`。

![2](img/#co_browser_specific_manipulation_CO40-2)

我们使用默认配置将当前网页打印为 PDF。

![3](img/#co_browser_specific_manipulation_CO40-3)

我们获取 PDF 的内容并转换为 Base64。

![4](img/#co_browser_specific_manipulation_CO40-4)

我们验证这个内容是否包含文件签名（“魔术词”`JVBER`）。

![5](img/#co_browser_specific_manipulation_CO40-5)

我们将 Base64 转换为原始字节数组。

![6](img/#co_browser_specific_manipulation_CO40-6)

我们将 PDF 内容（字节数组）写入本地文件。

# WebDriver BiDi

[WebDriver BiDi](https://w3c.github.io/webdriver-bidi) 是一个 W3C 草案，定义了双向 WebDriver 协议。BiDi 引入了一个 WebSocket 连接，使驱动程序和浏览器之间可以进行双向通信，而不是 WebDriver 协议的严格命令/响应格式。这样，WebDriver BiDi 将允许使用快速的双向传输执行不同的操作（即，无需轮询浏览器以获取响应）。

在 Selenium WebDriver 中，BiDi 的目标是长期替代目前由 CDP 支持的高级操作。例如，Selenium WebDriver API 支持通过`HasLog​E⁠vents`接口实现事件监听器。在撰写本文时，该接口在 CDP 之上运行。然而，未来的 Selenium WebDriver 版本将在内部使用 BiDi，提供更强大的跨浏览器兼容性。`HasLogEvents`允许实现以下事件的监听器：

`domMutation`

为了捕获 DOM 中的变化事件。示例 5-48 展示了一个实现此类事件监听器的测试。

`consoleEvent`

为了捕获浏览器控制台的变化事件，比如 JavaScript 的跟踪。示例 5-49 展示了第二个实现这种类型监听器的测试。

##### 示例 5-48\. 测试实现 DOM 变异事件的监听器

```java
@Test
void testDomMutation() throws InterruptedException {
    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");

    HasLogEvents logger = (HasLogEvents) driver; ![1](img/1.png)
    JavascriptExecutor js = (JavascriptExecutor) driver;

    AtomicReference<DomMutationEvent> seen = new AtomicReference<>();
    CountDownLatch latch = new CountDownLatch(1);
    logger.onLogEvent(CdpEventTypes.domMutation(mutation -> {
        seen.set(mutation);
        latch.countDown();
    })); ![2](img/2.png)

    WebElement img = driver.findElement(By.tagName("img"));
    String newSrc = "img/award.png";
    String script = String.format("arguments[0].src = '%s';", newSrc);
    js.executeScript(script, img); ![3](img/3.png)

    assertThat(latch.await(10, TimeUnit.SECONDS)).isTrue(); ![4](img/4.png)
    assertThat(seen.get().getElement().getAttribute("src"))
            .endsWith(newSrc); ![5](img/5.png)
}
```

![1](img/#co_browser_specific_manipulation_CO41-1)

我们将驱动对象转换为`HasLogEvents`。此转换仅适用于 Chrome 和 Edge 浏览器。

![2](img/#co_browser_specific_manipulation_CO41-2)

我们创建了一个监听器以捕获 DOM 变化事件。这个测试期望仅捕获一个事件，使用倒计时锁定来同步。

![3](img/#co_browser_specific_manipulation_CO41-3)

我们通过执行 JavaScript 强制 DOM 变化来改变图像来源。

![4](img/#co_browser_specific_manipulation_CO41-4)

我们验证事件最多在 10 秒内发生。

![5](img/#co_browser_specific_manipulation_CO41-5)

我们检查了图片来源已经改变。

##### 示例 5-49\. 实现一个监听器以捕获控制台事件

```java
@Test
void testConsoleEvents() throws InterruptedException {
    HasLogEvents logger = (HasLogEvents) driver;

    CountDownLatch latch = new CountDownLatch(4);
    logger.onLogEvent(CdpEventTypes.consoleEvent(consoleEvent -> {
        log.debug("{} {}: {}", consoleEvent.getTimestamp(),
                consoleEvent.getType(), consoleEvent.getMessages());
        latch.countDown();
    })); ![1](img/1.png)

    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/console-logs.html"); ![2](img/2.png)

    assertThat(latch.await(10, TimeUnit.SECONDS)).isTrue();
}
```

![1](img/#co_browser_specific_manipulation_CO42-1)

我们创建了一个监听器来捕获控制台事件。这个测试期望使用倒计时锁定来同步捕获四个事件。

![2](img/#co_browser_specific_manipulation_CO42-2)

我们打开了实践网页，该网页在 JavaScript 控制台中记录了几条消息。

# 摘要与展望

本章介绍了 Selenium WebDriver API 的实际概述，这些功能在各种浏览器之间不兼容。首先，您了解到如何使用能力在无头模式下运行浏览器，更改页面加载策略，使用 Web 扩展或管理浏览器弹出窗口（例如地理位置、通知或获取用户媒体等），以及其他能力。然后，您了解到 Selenium WebDriver 提供了使用 CDP 与 Web 浏览器交互的不同方式。这种机制允许在 Selenium WebDriver 测试中集成许多强大的功能，如模拟网络条件、基本和摘要身份验证、网络监控、处理 HTTP 头部或阻止 URL 等。然后，您了解到其他浏览器特定的功能，如位置上下文、Web 认证（WebAuthn）和将网页打印成 PDF 文档。最后，您了解到 WebDriver BiDi，这是一个草案标准化，用于定义与浏览器的双向通信，用于自动化目的。在撰写本文时，BiDi 处于早期阶段。目标是在未来版本中，Selenium WebDriver 将在 BiDi 之上支持不同的标准功能。

下一章总结了我们与 Selenium WebDriver API 的旅程。该章节解释了如何使用这个 API 来控制远程浏览器。这些浏览器可以托管在 Selenium Grid 上，云提供商（例如 Sauce Labs、BrowserStack 或 CrossBrowserTesting），或者在 Docker 容器中执行。
