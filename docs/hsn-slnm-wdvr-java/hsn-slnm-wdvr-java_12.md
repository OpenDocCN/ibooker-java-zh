# 第九章：第三方集成

本章介绍了不同的第三方技术（如库或框架），我们可以与 Selenium WebDriver 结合使用。当 Selenium WebDriver API 不足以执行特定任务时，我们需要使用这些技术，比如文件下载，我们需要使用第三方工具来等待文件正确下载，或者使用 HTTP 客户端来控制下载。我们还可以使用第三方代理来捕获 HTTP 流量。

另一个场景是我们需要使用 Selenium WebDriver 与外部工具结合来实现非功能性测试，比如性能、安全性、可访问性或 A/B 测试。我们还可以使用第三方库开发 Selenium WebDriver 测试，使用流畅的 API，生成虚拟测试数据，或者改进测试报告。最后，我们可以集成相关框架如 Cucumber 用于行为驱动开发（BDD），或者 Spring Framework（用于开发 Web 应用）。本章节将详细介绍所有这些用途。

###### 小贴士

要使用本章介绍的第三方工具，你必须首先在项目中包含所需的依赖项。你可以在 附录 C 中找到使用 Maven 和 Gradle 解决每个依赖项的详细信息。

# 文件下载

Selenium WebDriver 对文件下载的支持有限，因为其 API 不公开下载进度。换句话说，我们可以使用 Selenium WebDriver 下载来自 Web 应用的文件，但无法控制将这些文件复制到本地文件系统所需的时间。因此，我们可以使用第三方库来增强使用 Selenium WebDriver 进行 Web 下载的体验。有不同的替代方案来实现这个目标。下面的小节将详细解释如何做到这一点。

## 使用浏览器特定的能力

我们可以使用特定于浏览器的能力（就像我们在 第五章 中所做的那样）来配置文件下载的各种参数，例如目标文件夹。这种方法很方便，因为这些功能在 Selenium WebDriver API 中是开箱即用的，但它也有几个缺点。首先，它与不同的浏览器类型（如 Chrome、Firefox 等）不兼容。换句话说，每个浏览器的所需能力是不同的。其次，更重要的是，我们无法控制跟踪下载进度。为了解决这个问题，我们需要使用第三方库。在本书中，我建议使用开源库 [Awaitility](http://www.awaitility.org)。

Awaitility 是一个流行的库，提供处理异步操作的功能。它通过提供流畅的 API 来管理线程、超时和并发问题。在使用 Selenium WebDriver 下载文件的情况下，我们使用 Awaitility API 等待直到文件存储在文件系统中。示例 9-1 展示了使用 Chrome 和 Awaitility 的示例。示例 9-2 展示了使用 Firefox 时等效的测试设置。

##### Example 9-1\. 使用 Chrome 和 Awaitility 测试下载文件

```java
class DownloadChromeJupiterTest {

    WebDriver driver;

    File targetFolder;

    @BeforeEach
    void setup() {
        targetFolder = new File(System.getProperty("user.home"), "Downloads"); ![1](img/1.png)
        Map<String, Object> prefs = new HashMap<>();
        prefs.put("download.default_directory", targetFolder.toString()); ![2](img/2.png)
        ChromeOptions options = new ChromeOptions();
        options.setExperimentalOption("prefs", prefs);

        driver = WebDriverManager.chromedriver().capabilities(options).create();
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testDownloadChrome() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/download.html"); ![3](img/3.png)

        driver.findElement(By.xpath("(//a)[2]")).click(); ![4](img/4.png)
        driver.findElement(By.xpath("(//a)[3]")).click();

        ConditionFactory await = Awaitility.await()
                .atMost(Duration.ofSeconds(5)); ![5](img/5.png)
        File wdmLogo = new File(targetFolder, "webdrivermanager.png");
        await.until(() -> wdmLogo.exists()); ![6](img/6.png)

        File wdmDoc = new File(targetFolder, "webdrivermanager.pdf");
        await.until(() -> wdmDoc.exists()); ![7](img/7.png)
    }

}
```

![1](img/#co_third_party_integrations_CO1-1)

我们指定一个文件夹保存下载的文件。但是需要注意，Chrome 只允许特定的目录用于下载。例如，它允许下载目录（及其子文件夹），但禁止使用其他路径，如桌面文件夹或主目录。

![2](img/#co_third_party_integrations_CO1-2)

我们使用 Chrome 首选项指定目标文件夹。

![3](img/#co_third_party_integrations_CO1-3)

我们使用练习网站上提供的网页通过点击按钮下载不同的文件（见图 9-1）。

![4](img/#co_third_party_integrations_CO1-4)

我们点击页面上的两个按钮。结果，浏览器开始下载两个文件：一个 PNG 图片和一个 PDF 文档。

![5](img/#co_third_party_integrations_CO1-5)

我们使用 Awaitility 配置了五秒的等待超时。

![6](img/#co_third_party_integrations_CO1-6)

我们等待直到第一个文件在文件系统中。

![7](img/#co_third_party_integrations_CO1-7)

我们还等待第二个文件下载完成。

##### 示例 9-2\. 使用 Firefox 下载文件的测试设置

```java
@BeforeEach
void setup() {
    FirefoxOptions options = new FirefoxOptions();
    targetFolder = new File("."); ![1](img/1.png)
    options.addPreference("browser.download.dir",
            targetFolder.getAbsolutePath()); ![2](img/2.png)
    options.addPreference("browser.download.folderList", 2); ![3](img/3.png)
    options.addPreference("browser.helperApps.neverAsk.saveToDisk",
            "image/png, application/pdf"); ![4](img/4.png)
    options.addPreference("pdfjs.disabled", true); ![5](img/5.png)

    driver = WebDriverManager.firefoxdriver().capabilities(options)
            .create();
}
```

![1](img/#co_third_party_integrations_CO2-1)

Firefox 允许指定任何文件夹来下载文件。在这种情况下，我们使用本地项目文件夹。

![2](img/#co_third_party_integrations_CO2-2)

我们使用 Firefox 首选项指定自定义下载目录。

![3](img/#co_third_party_integrations_CO2-3)

我们需要将首选项 `browser.download.folderList` 设置为 `2` 以选择自定义下载文件夹。其他可能的值为 `0`（将文件下载到用户桌面）和 `1`（使用下载文件夹，即默认值）。

![4](img/#co_third_party_integrations_CO2-4)

我们指定 Firefox 不会要求保存在本地文件系统中的内容类型。

![5](img/#co_third_party_integrations_CO2-5)

我们禁用 PDF 文件的预览。

![hosw 0901](img/hosw_0901.png)

###### 图 9-1\. 用于下载文件的练习网页

## 使用 HTTP 客户端

使用 Selenium WebDriver 下载文件的另一种机制是使用 HTTP 客户端库。我建议使用[Apache HttpClient](https://hc.apache.org/httpcomponents-client-5.1.x)，因为 WebDriverManager 内部使用了这个库，因此你可以在项目中作为传递依赖使用它。示例 9-3 展示了使用 Apache HttpClient 从实践站点下载多个文件的完整测试用例。注意，在这种情况下，不需要显式等待文件下载完成，因为 Apache HttpClient 同步处理 HTTP 响应。

##### 示例 9-3\. 使用 HTTP 客户端下载文件的测试

```java
class DownloadHttpClientJupiterTest {

    WebDriver driver;

    @BeforeEach
    void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @Test
    void testDownloadHttpClient() throws IOException {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/download.html"); ![1](img/1.png)

        WebElement pngLink = driver.findElement(By.xpath("(//a)[2]")); ![2](img/2.png)
        File pngFile = new File(".", "webdrivermanager.png");
        download(pngLink.getAttribute("href"), pngFile); ![3](img/3.png)
        assertThat(pngFile).exists();

        WebElement pdfLink = driver.findElement(By.xpath("(//a)[3]")); ![4](img/4.png)
        File pdfFile = new File(".", "webdrivermanager.pdf");
        download(pdfLink.getAttribute("href"), pdfFile);
        assertThat(pdfFile).exists();
    }

    void download(String link, File destination) throws IOException {
        try (CloseableHttpClient client = HttpClientBuilder.create().build()) { ![5](img/5.png)
            HttpUriRequestBase request = new HttpGet(link);
            try (CloseableHttpResponse response = client.execute(request)) { ![6](img/6.png)
                FileUtils.copyInputStreamToFile(
                        response.getEntity().getContent(), destination); ![7](img/7.png)
            }
        }
    }

}
```

![1](img/#co_third_party_integrations_CO3-1)

我们再次使用实践网页下载文件。

![2](img/#co_third_party_integrations_CO3-2)

我们点击一个按钮来下载一个文件。

![3](img/#co_third_party_integrations_CO3-3)

我们重构了类方法`download`中下载文件的通用逻辑。

![4](img/#co_third_party_integrations_CO3-4)

我们为第二个要下载的文件重复操作。

![5](img/#co_third_party_integrations_CO3-5)

我们在 try-with-resources 中创建了一个 Apache HTTPClient 实例。这个客户端在语句作用域结束时会自动关闭。

![6](img/#co_third_party_integrations_CO3-6)

我们使用另一个 try-with-resources 语句向提供的 URL 发送 HTTP 请求，并获得 HTTP 响应。

![7](img/#co_third_party_integrations_CO3-7)

我们将生成的文件复制到本地文件系统中。

# 捕获网络流量

“网络监控” 和 “网络拦截器” 解释了如何使用特定于浏览器的功能来捕获 Selenium WebDriver 和测试中的 Web 应用程序之间的 HTTP 流量。这种机制的缺点是只有支持 CDP 的浏览器才能使用。然而，我们可以为其他浏览器使用第三方代理。在本书中，我建议您为此目的使用[BrowserMob](https://bmp.lightbody.net)代理。

BrowserMob 是一个开源代理，允许使用 Java 库操纵 HTTP 流量。示例 9-4 展示了在 Selenium WebDriver 测试中使用此代理的完整测试。在此示例中，我们使用 BrowserMob 代理拦截测试和目标网站之间的 HTTP 流量，并跟踪这些流量（请求-响应）作为日志跟踪。

##### 示例 9-4\. 通过 BrowserMob 代理捕获网络流量的测试

```java
class CaptureNetworkTrafficFirefoxJupiterTest {

    static final Logger log = getLogger(lookup().lookupClass());

    WebDriver driver;

    BrowserMobProxy proxy;

    @BeforeEach
    void setup() {
        proxy = new BrowserMobProxyServer(); ![1](img/1.png)
        proxy.start(); ![2](img/2.png)
        proxy.newHar(); ![3](img/3.png)
        proxy.enableHarCaptureTypes(CaptureType.REQUEST_CONTENT,
                CaptureType.RESPONSE_CONTENT); ![4](img/4.png)

        Proxy seleniumProxy = ClientUtil.createSeleniumProxy(proxy); ![5](img/5.png)
        FirefoxOptions options = new FirefoxOptions();
        options.setProxy(seleniumProxy); ![6](img/6.png)
        options.setAcceptInsecureCerts(true); ![7](img/7.png)

        driver = WebDriverManager.firefoxdriver().capabilities(options)
                .create();
    }

    @AfterEach
    void teardown() {
        proxy.stop(); ![8](img/8.png)
        driver.quit();
    }

    @Test
    void testCaptureNetworkTrafficFirefox() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");

        List<HarEntry> logEntries = proxy.getHar().getLog().getEntries();
        logEntries.forEach(logEntry -> { ![9](img/9.png)
            log.debug("Request: {} - Response: {}",
                    logEntry.getRequest().getUrl(),
                    logEntry.getResponse().getStatus());
        });
    }

}
```

![1](img/#co_third_party_integrations_CO4-1)

我们创建一个 BrowserMob 的实例。

![2](img/#co_third_party_integrations_CO4-2)

我们启动这个代理。

![3](img/#co_third_party_integrations_CO4-3)

我们使用 HAR（HTTP 存档）来捕获 HTTP 流量，这是一种基于 JSON 的文件格式，用于捕获和导出这些流量。

![4](img/#co_third_party_integrations_CO4-4)

我们启用捕获交换的 HTTP 请求和响应。

![5](img/#co_third_party_integrations_CO4-5)

我们将 BrowserMob 服务器转换为 Selenium WebDriver 代理。

![6](img/#co_third_party_integrations_CO4-6)

我们将这个代理设置为浏览器选项（在这种情况下，用于 Firefox）。

![7](img/#co_third_party_integrations_CO4-7)

我们需要允许不安全的证书，因为与代理的通信是使用 HTTP（而不是 HTTPS）完成的。

![8](img/#co_third_party_integrations_CO4-8)

我们在测试后停止代理。

![9](img/#co_third_party_integrations_CO4-9)

我们使用代理实例来收集 HTTP 流量（请求和响应）。在这个基本示例中，我们使用记录器将这些信息写入标准输出。

# 非功能测试

如第一章所述，Selenium WebDriver 主要用于评估 Web 应用程序的功能需求。换句话说，测试人员使用 Selenium WebDriver API 来验证测试中的 Web 应用程序是否按预期行为。然而，我们可以利用这个 API 来测试非功能需求，即性能、安全性、可访问性等质量属性。实现这一目标的常见策略是与特定的第三方实用程序集成。以下各小节解释了用于非功能测试的 Selenium WebDriver 的不同集成。

## 性能

性能测试评估了特定工作负载下系统在响应速度和稳定性方面的表现。与 Selenium WebDriver 不同，测试人员通常采用专门的工具，如[Apache JMeter](https://jmeter.apache.org)进行性能测试。 Apache JMeter 是一个开源工具，允许向给定的 URL 端点发送多个 HTTP 请求，同时测量响应时间和其他指标。尽管 Selenium WebDriver 与 Apache JMeter 之间的直接集成并不容易，但我们可以将现有的 Selenium WebDriver 测试用作 JMeter 测试计划（即 JMeter 执行的一系列步骤）。这种方法的好处在于，生成的 JMeter 测试计划将模仿 Selenium WebDriver 测试中使用的相同用户工作流程，重用浏览器进行的相同 HTTP 流量（例如，用于 JavaScript 库、CSS 等）。为此，我提出以下程序：

1.  使用 BrowserMob 代理（在前一节中介绍）将 Selenium WebDriver 中交换的网络流量捕获为 HAR 文件。

1.  将生成的 HAR 文件转换为 JMeter 测试计划。 JMeter 中的测试计划存储为扩展名为 JMX 的基于 XML 的文件。

1.  在 JMeter 中加载 JMX 测试计划并进行调整以模拟并发用户并包括结果监听器

1.  运行测试计划并评估结果。

示例 9-5 展示了实现第一步的完整测试案例。正如您所见，需要登录才能开始并创建 HAR 文件的步骤在每次测试前后都已完成。您可以使用此方法将现有的功能测试（即 `@Test` 方法中的逻辑）作为性能测试来执行（在 JMeter 中执行）。

##### 示例 9-5\. 创建 HAR 文件的测试。

```java
class HarCreatorJupiterTest {

    WebDriver driver;

    BrowserMobProxy proxy;

    @BeforeEach
    void setup() {
        proxy = new BrowserMobProxyServer(); ![1](img/1.png)
        proxy.start();
        proxy.newHar();
        proxy.enableHarCaptureTypes(CaptureType.REQUEST_CONTENT,
                CaptureType.RESPONSE_CONTENT);

        Proxy seleniumProxy = ClientUtil.createSeleniumProxy(proxy);
        ChromeOptions options = new ChromeOptions();
        options.setProxy(seleniumProxy);
        options.setAcceptInsecureCerts(true);

        driver = WebDriverManager.chromedriver().capabilities(options).create();
    }

    @AfterEach
    void teardown() throws IOException {
        Har har = proxy.getHar(); ![2](img/2.png)
        File harFile = new File("login.har");
        har.writeTo(harFile);

        proxy.stop();
        driver.quit();
    }

    @Test
    void testHarCreator() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/login-form.html");

        driver.findElement(By.id("username")).sendKeys("user");
        driver.findElement(By.id("password")).sendKeys("user");
        driver.findElement(By.cssSelector("button")).click();
        String bodyText = driver.findElement(By.tagName("body")).getText();
        assertThat(bodyText).contains("Login successful");
    }

}
```

![1](img/#co_third_party_integrations_CO5-1)

我们在测试前启动 BrowserMob 并在 `WebDriver` 会话中配置它。

![2](img/#co_third_party_integrations_CO5-2)

在测试后，我们获取 HAR 文件并将其写入本地文件。

运行前面的测试后，我们获得一个名为 `login.har` 的 HAR 文件。现在，我们需要将其转换为 JMeter 测试计划。有多种替代方案可供选择。您可以找到多个程序（例如 Ruby 或 Java）在网上免费提供此服务。此外，您还可以使用在线转换服务，如 [BlazeMeter JMX Converter](https://converter.blazemeter.com)。在本示例中，我使用了这个在线服务，并在 JMeter 中打开生成的 JMX 测试计划。此时，您可以根据需要调整 JMeter 配置（您可以在官方的 [用户手册](https://jmeter.apache.org/usermanual/index.html) 中找到有关 JMeter 的更多信息）。例如，图 9-2 显示了加载生成的 JMX 测试计划后 JMeter GUI 的更改情况：

+   将并发用户数增加到一百（在“Thread Group”选项卡中）

+   包括一些结果监听器，例如“聚合图”和“图形结果”。

![hosw 0902](img/hosw_0902.png)

###### 图 9-2\. JMeter GUI 加载生成的测试计划。

现在，我们可以使用 JMeter 运行测试计划（例如，在 JMeter GUI 中单击绿色三角形按钮）。结果是，根据最初开发的互动，生成了一百个并发用户的负载，作为 Selenium WebDriver 测试的一部分（示例 9-5）。图 9-3 显示了之前添加的监听器的结果。

![hosw 0903](img/hosw_0903.png)

###### 图 9-3\. JMeter 结果。

### 使用浏览器生成负载

对于许多 Web 应用性能测试场景，使用像 JMeter 这样的工具非常方便。然而，在需要实际浏览器重现完整用户工作流程（例如视频会议 Web 应用程序）时，此方法不适合。在这种情况下，一种可能的解决方案是与 Docker 一起使用 WebDriverManager。示例 9-6 展示了这种用法。正如您在本测试中所见，WebDriverManager 允许通过在 `create()` 方法中指定大小参数来简单地创建 `WebDriver` 实例列表。然后，例如，我们可以使用标准的 Java 使用线程池并行执行测试的 Web 应用程序。

##### 示例 9-6\. 使用 WebDriverManager 和 Docker 进行负载测试。

```java
class LoadJupiterTest {

    static final int NUM_BROWSERS = 5;

    final Logger log = getLogger(lookup().lookupClass());

    List<WebDriver> driverList;

    WebDriverManager wdm = WebDriverManager.chromedriver().browserInDocker(); ![1](img/1.png)

    @BeforeEach
    void setupTest() {
        assumeThat(isDockerAvailable()).isTrue(); ![2](img/2.png)
        driverList = wdm.create(NUM_BROWSERS); ![3](img/3.png)
    }

    @AfterEach
    void teardown() {
        wdm.quit();
    }

    @Test
    void testLoad() throws InterruptedException {
        ExecutorService executorService = newFixedThreadPool(NUM_BROWSERS); ![4](img/4.png)
        CountDownLatch latch = new CountDownLatch(NUM_BROWSERS);

        driverList.forEach((driver) -> { ![5](img/5.png)
            executorService.submit(() -> {
                try {
                    checkHomePage(driver);
                } finally {
                    latch.countDown();
                }
            });
        });

        latch.await(60, SECONDS); ![6](img/6.png)
        executorService.shutdown();
    }

    void checkHomePage(WebDriver driver) {
        log.debug("Session id {}", ((RemoteWebDriver) driver).getSessionId());
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_third_party_integrations_CO6-1)

我们创建一个 Chrome 管理器实例，使用 Docker 将浏览器作为容器执行。

![2](img/#co_third_party_integrations_CO6-2)

我们假设运行此测试的机器上已安装 Docker。否则，将跳过此测试。

![3](img/#co_third_party_integrations_CO6-3)

我们创建一个 `WebDriver` 列表（本示例中包含五个实例）。

![4](img/#co_third_party_integrations_CO6-4)

我们使用与 `WebDriver` 列表相同大小的线程池。

![5](img/#co_third_party_integrations_CO6-5)

我们使用线程池来并行执行 SUT 评估。

![6](img/#co_third_party_integrations_CO6-6)

我们等待每个并行评估完成。我们使用基于计数器门闩的同步方法来实现。

## 安全性

软件安全领域的一个相关组织是 [OWASP](https://owasp.org)（开放网络应用安全项目），这是一个促进开放解决方案以提高软件安全性的非营利基金会。其中最流行的 OWASP 项目之一是 Zed Attack Proxy（ZAP）。[OWASP ZAP](https://www.zaproxy.org) 是一个开源的 Web 应用安全性扫描工具，用于实施*漏洞评估*（即寻找安全问题）或*渗透测试*（即模拟的网络攻击）以发现可利用的 Web 应用漏洞。

我们可以将 OWASP ZAP 作为独立的桌面应用程序使用。Figure 9-4 显示了其 GUI 的截图。

![hosw 0904](img/hosw_0904.png)

###### 图 9-4\. OWASP ZAP GUI

此 GUI 提供不同的功能以自动化扫描，以检测 Web 应用可能面临的安全威胁，如 SQL 注入、跨站点脚本（XSS）或跨站点请求伪造（CSRF）等。

除了独立应用程序外，我们还可以将 Selenium WebDriver 测试与 ZAP 集成。示例 9-7 提供了一个说明此集成的测试用例。执行此测试所需的步骤包括：

1.  在本地主机上启动 OWASP ZAP。默认情况下，OWASP 启动一个代理，监听端口 8080。您可以使用 OWASP GUI 中的菜单选项 Tools → Options → Local Proxies 更改此端口。

1.  禁用 API 密钥（或在 Selenium WebDriver 测试中复制其值）。您可以在菜单选项 Tools → Options → API 中更改此值。

1.  实现一个使用 OWASP ZAP 作为代理的 Selenium WebDriver 测试（类似于 示例 9-7）。

1.  执行 Selenium WebDriver 测试。此时，您应该在 ZAP GUI 中看到生成的漏洞报告。

##### 示例 9-7\. 使用 OWASP ZAP 作为安全扫描器的测试

```java
class SecurityJupiterTest {

    static final Logger log = getLogger(lookup().lookupClass());

    static final String ZAP_PROXY_ADDRESS = "localhost"; ![1](img/1.png)
    static final int ZAP_PROXY_PORT = 8080;
    static final String ZAP_API_KEY = "<put-api-key-here-or-disable-it>"; ![2](img/2.png)

    WebDriver driver;

    ClientApi api;

    @BeforeEach
    void setup() {
        String proxyStr = ZAP_PROXY_ADDRESS + ":" + ZAP_PROXY_PORT;
        assumeThat(isOnline("http://" + proxyStr)).isTrue();

        Proxy proxy = new Proxy(); ![3](img/3.png)
        proxy.setHttpProxy(proxyStr);
        proxy.setSslProxy(proxyStr);

        ChromeOptions options = new ChromeOptions();
        options.setAcceptInsecureCerts(true);
        options.setProxy(proxy);

        driver = WebDriverManager.chromedriver().capabilities(options).create();

        api = new ClientApi(ZAP_PROXY_ADDRESS, ZAP_PROXY_PORT, ZAP_API_KEY); ![4](img/4.png)
    }

    @AfterEach
    void teardown() throws ClientApiException {
        if (api != null) {
            String title = "My ZAP report";
            String template = "traditional-html";
            String description = "This is a sample report";
            String reportfilename = "zap-report.html";
            String targetFolder = new File("").getAbsolutePath();
            ApiResponse response = api.reports.generate(title, template, null,
                    description, null, null, null, null, null, reportfilename,
                    null, targetFolder, null); ![5](img/5.png)
            log.debug("ZAP report generated at {}", response.toString());
        }
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    void testSecurity() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/web-form.html");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_third_party_integrations_CO7-1)

我们配置 ZAP 本地代理监听的地址和端口。

![2](img/#co_third_party_integrations_CO7-2)

如果未禁用 ZAP API 密钥，则需要在这里设置其值。

![3](img/#co_third_party_integrations_CO7-3)

我们将 ZAP 配置为 Selenium WebDriver 代理。

![4](img/#co_third_party_integrations_CO7-4)

我们使用其 API 与 ZAP 进行交互。

![5](img/#co_third_party_integrations_CO7-5)

在测试结束后，我们生成一个 HTML 报告，报告中包含在执行 Selenium WebDriver 测试期间发现的漏洞。图 9-5 显示了此报告的屏幕截图。

###### 注意

我们还可以作为独立的 GUI 使用 OWASP ZAP，如前所介绍。与 Selenium WebDriver 集成的潜在好处可能是重用现有的功能测试以评估安全性或自动化安全评估（例如，由 CI 服务器执行的回归测试套件）。

![hosw 0905](img/hosw_0905.png)

###### 图 9-5\. ZAP 在执行 Selenium WebDriver 测试后生成的报告

## 可访问性

数字可访问性指的是残障用户有效使用软件系统（如网站、移动应用等）的能力。在这一领域的一个重要参考是 [Web 内容可访问性指南](https://www.w3.org/WAI/standards-guidelines/wcag)（WCAG），这是由 W3C [Web 可访问性倡议](https://www.w3.org/WAI)（WAI）制定的一套标准建议，解释如何使网页内容对残障人士更易访问。

有几种方法可以测试 Web 应用的可访问性。最常见的方法是检查 WCAG 建议。为此，我们可以使用自动化可访问性扫描器，如 [Axe](https://www.deque.com/axe)，这是一个开源引擎，用于按照 WCAG 规则自动测试 Web 应用的可访问性。Axe 通过 Selenium WebDriver 的 Java 绑定与一个 [辅助库](https://github.com/dequelabs/axe-core-maven-html) 实现无缝集成。示例 9-8 展示了使用此库进行测试。

##### 示例 9-8\. 使用 Axe 生成可访问性报告的测试

```java
@Test
void testAccessibility() {
    driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
    assertThat(driver.getTitle()).contains("Selenium WebDriver");

    Results result = new AxeBuilder().analyze(driver); ![1](img/1.png)
    List<Rule> violations = result.getViolations(); ![2](img/2.png)
    violations.forEach(rule -> {
        log.debug("{}", rule.toString()); ![3](img/3.png)
    });
    AxeReporter.writeResultsToJsonFile("testAccessibility", result); ![4](img/4.png)
}
```

![1](img/#co_third_party_integrations_CO8-1)

我们使用 Axe 分析当前的 `WebDriver` 会话。这样，浏览器中加载的所有页面都将由 Axe 扫描。

![2](img/#co_third_party_integrations_CO8-2)

我们获得了可访问性违规的报告。

![3](img/#co_third_party_integrations_CO8-3)

我们将每个违规记录在标准输出中。在此示例中，发现的问题包括：

色彩对比

元素必须具有足够的颜色对比度。

标题顺序

标题级别应仅增加一级。

图片替代文本

图像必须具有替代文本。

链接名称

链接必须具有可识别的文本。

![4](img/#co_third_party_integrations_CO8-4)

我们将结果写入本地的 JSON 文件中。

## A/B 测试

A/B 测试是一种评估可用性的形式，它比较同一应用程序的不同变体，以发现哪一个对最终用户更有效。不同的商业产品为 Selenium WebDriver 测试提供了 A/B 测试的高级功能。例如，[Applitools Eyes](https://applitools.com/products-eyes) 提供了多个网页变体的自动视觉比较。另一个选择是 [Optimizely](https://www.optimizely.com)，这是一家提供定制和实验 A/B 测试工具的公司。

另一种进行 A/B 测试的方法是使用原始的 Selenium WebDriver API 和自定义条件来处理网页的不同变体。示例 9-9 展示了一个基于手动方法实现多变体网页的基本测试。请注意，这个测试展示了一种基于评估不同页面变体的 A/B 测试的简单方法。

##### 示例 9-9\. 使用 Selenium WebDriver 的基本 A/B 测试

```java
@Test
void testABTesting() {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/ab-testing.html"); ![1](img/1.png)
    WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
    WebElement header = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.tagName("h6")));

    if (header.getText().contains("variation A")) { ![2](img/2.png)
        assertBodyContains(driver, "Lorem ipsum");
    } else if (header.getText().contains("variation B")) {
        assertBodyContains(driver, "Nibh netus");
    } else {
        fail("Unknown variation");
    }
}

void assertBodyContains(WebDriver driver, String text) {
    String bodyText = driver.findElement(By.tagName("body")).getText();
    assertThat(bodyText).contains(text);
}
```

![1](img/#co_third_party_integrations_CO9-1)

我们打开一个多变体练习网页。该页面的内容每次随机加载的概率为 50%。

![2](img/#co_third_party_integrations_CO9-2)

我们检查页面的变体是否符合预期。

# 流畅 API

如在 第一章 中介绍的，Selenium 是其他框架和库的基础技术。例如，我们可以找到几个封装了 Selenium WebDriver 的库，以便为 Web 应用程序创建端到端测试的流畅 API。这类库的一个例子是 [Selenide](https://selenide.org)，这是一个开源（MIT 许可证）的库，它在 Selenium WebDriver 之上定义了一个简洁的流畅 API。Selenide 提供多种好处，例如自动等待 Web 元素或支持 AJAX 应用程序。

与 Selenium WebDriver 相比，Selenide 的一个显著区别在于它在内部处理 `WebDriver` 对象。为此，它使用 WebDriverManager 来解析所需的驱动程序（例如 chromedriver、geckodriver 等），将 `WebDriver` 实例保持在一个单独的线程中，在测试结束时关闭。因此，不需要创建和终止 `WebDriver` 对象所需的测试样板代码。示例 9-10 通过展示一个基本的 Selenide 测试演示了这个特性。

##### 示例 9-10\. 使用 Selenide 进行测试

```java
class SelenideJupiterTest {

    @Test
    void testSelenide() {
        open("https://bonigarcia.dev/selenium-webdriver-java/login-form.html"); ![1](img/1.png)

        $(By.id("username")).val("user"); ![2](img/2.png)
        $(By.id("password")).val("user");
        $("button").pressEnter(); ![3](img/3.png)
        $(By.id("success")).shouldBe(visible)
                .shouldHave(text("Login successful")); ![4](img/4.png)
    }

}
```

![1](img/#co_third_party_integrations_CO10-1)

我们使用 Selenide 提供的 `open` 静态方法来访问给定的 URL。默认情况下，Selenide 使用本地的 Chrome 浏览器，尽管可以通过配置类（例如 `Configuration.browser = "firefox";`）或使用 Java 系统属性（例如 `-Dselenide.browser=firefox`）来更改浏览器。

![2](img/#co_third_party_integrations_CO10-2)

Selenide 的 `$` 方法允许您通过 CSS 选择器或使用 Selenium WebDriver 的 `By` 定位器定位 Web 元素。此行代码使用后者向输入字段输入文本。

![3](img/#co_third_party_integrations_CO10-3)

我们通过 CSS 选择器定位另一个网页元素，并单击它。

![4](img/#co_third_party_integrations_CO10-4)

我们验证登录成功的网页元素是否存在，并包含预期的文本。

# 测试数据

任何测试用例的相关部分都是测试数据，即用于执行 SUT 的输入数据。选择适当的测试数据对于实施有效的测试至关重要。经典测试理论中用于测试数据选择的不同技术包括：

等价分区

通过将所有可能的输入测试数据分成我们假定以相同方式处理的值集的过程来进行测试。

边界测试

在输入值的极端端点或分区之间进行测试的过程。这种方法的基本思想是在输入域中选择代表性极限值（例如，最小值以下、最小值、略高于最小值、标称值、略低于最大值、最大值和超过最大值的值）。

这些方法在端到端测试中可能并不实用，因为执行这些策略所需的测试数目（以及随之而来的执行时间）可能是巨大的。相反，我们通常手动选择一些代表性测试数据来验证*快乐路径*（即*正向*测试），并可选择一些用于意外情况的测试数据（*负向*测试）。

选择测试数据的另一种选择是使用*假*数据，即不同域的随机数据，如个人姓名、姓氏、地址、国家、电子邮件、电话号码等。实现这一目标的一个简单替代方法是使用[Java Faker](https://dius.github.io/java-faker)，这是流行的[Ruby faker](https://github.com/faker-ruby/faker)库的 Java 移植版。示例 9-11 展示了使用该库进行测试的示例。此测试使用假数据提交了一个练习站点上可用的 Web 表单。图 9-6 展示了在使用该假数据提交表单后的网页。

##### 示例 9-11\. 使用 Java Faker 生成不同类型的假数据进行测试

```java
@Test
void testFakeData() {
    driver.get(
            "https://bonigarcia.dev/selenium-webdriver-java/data-types.html");

    Faker faker = new Faker(); ![1](img/1.png)

    driver.findElement(By.name("first-name")) ![2](img/2.png)
            .sendKeys(faker.name().firstName());
    driver.findElement(By.name("last-name"))
            .sendKeys(faker.name().lastName());
    driver.findElement(By.name("address"))
            .sendKeys(faker.address().fullAddress());
    driver.findElement(By.name("zip-code"))
            .sendKeys(faker.address().zipCode());
    driver.findElement(By.name("city")).sendKeys(faker.address().city());
    driver.findElement(By.name("country"))
            .sendKeys(faker.address().country());
    driver.findElement(By.name("e-mail"))
            .sendKeys(faker.internet().emailAddress());
    driver.findElement(By.name("phone"))
            .sendKeys(faker.phoneNumber().phoneNumber());
    driver.findElement(By.name("job-position"))
            .sendKeys(faker.job().position());
    driver.findElement(By.name("company")).sendKeys(faker.company().name());

    driver.findElement(By.tagName("form")).submit();

    List<WebElement> successElement = driver
            .findElements(By.className("alert-success"));
    assertThat(successElement).hasSize(10); ![3](img/3.png)

    List<WebElement> errorElement = driver
            .findElements(By.className("alert-danger"));
    assertThat(errorElement).isEmpty(); ![4](img/4.png)
}
```

![1](img/#co_third_party_integrations_CO11-1)

我们创建一个 Java Faker 实例。

![2](img/#co_third_party_integrations_CO11-2)

我们发送不同类型的随机数据（姓名、地址、国家等）。

![3](img/#co_third_party_integrations_CO11-3)

我们验证数据是否被正确提交。

![4](img/#co_third_party_integrations_CO11-4)

我们检查页面上是否没有错误。

![hosw 0906](img/hosw_0906.png)

###### 图 9-6\. 使用假数据的实践页面

# 报告

*测试报告*是在执行测试套件后总结结果的文档。这份文档通常包括执行的测试数量及其判定结果（通过、失败、跳过）和执行时间。在我们的 Java 项目中，有不同的方法可以获得测试报告。例如，在使用 Maven 时，我们可以使用以下命令创建基本的测试报告：

```java
mvn test ![1](img/1.png)
mvn surefire-report:report-only ![2](img/2.png)
mvn site -DgenerateReports=false ![3](img/3.png)
```

![1](img/#co_third_party_integrations_CO12-1)

我们使用 Maven 执行测试。因此，Maven Surefire 插件在`target`文件夹中生成了一组 XML 文件。这些文件包含了测试执行的结果。

![2](img/#co_third_party_integrations_CO12-2)

我们将 XML 报告转换为 HTML 报告。您可以在项目文件夹`target/site`中找到此 HTML 报告。

![3](img/#co_third_party_integrations_CO12-3)

我们强制复制 CSS 和 HTML 报告中所需的图像。图 9-7 显示了这份报告的屏幕截图。

![hosw 0907](img/hosw_0907.png)

###### 图 9-7。使用 Maven 生成的测试报告

我们还可以使用 Gradle 生成等效的报告。在使用此构建工具执行测试套件后，Gradle 会自动在文件夹`build/reports`中生成一个 HTML 报告。图 9-8 展示了执行一组测试（使用 shell 命令`gradle test --tests Hello*`）时生成的测试报告示例。

![hosw 0908](img/hosw_0908.png)

###### 图 9-8。使用 Gradle 生成的测试报告

除了 Maven 和 Gradle，我们还可以使用现有的报告库来创建更丰富的报告。一个可能的替代方案是[Extent Reports](https://www.extentreports.com)，一个用于创建交互式测试报告的库。Extent Reports 提供专业（商业）和社区（开源）版本。示例 9-12 展示了使用后者的测试。

##### 示例 9-12。使用 Extent Reports 生成 HTML 报告的测试

```java
class ReportingJupiterTest {

    WebDriver driver;

    static ExtentReports reports;

    @BeforeAll
    static void setupClass() {
        reports = new ExtentReports(); ![1](img/1.png)
        ExtentSparkReporter htmlReporter = new ExtentSparkReporter(
                "extentReport.html");
        reports.attachReporter(htmlReporter); ![2](img/2.png)
    }

    @BeforeEach
    void setup(TestInfo testInfo) {
        reports.createTest(testInfo.getDisplayName()); ![3](img/3.png)
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @AfterAll
    static void teardownClass() {
        reports.flush();
    }

    // Test methods ![4](img/4.png)

}
```

![1](img/#co_third_party_integrations_CO13-1)

我们创建了一个测试报告生成器的实例。

![2](img/#co_third_party_integrations_CO13-2)

我们对其进行配置，以生成一个 HTML 报告。

![3](img/#co_third_party_integrations_CO13-3)

在每次测试后，我们使用测试名称作为标识符在测试报告中创建一个条目。在 JUnit 5 中，我们使用`TestInfo`，这是一个内置的参数解析器，允许检索关于当前测试的信息。

![4](img/#co_third_party_integrations_CO13-4)

和往常一样，您可以在[示例仓库](https://github.com/bonigarcia/selenium-webdriver-java)中找到完整的源代码。特别是，这个类有两个测试方法。图 9-9 展示了当执行这个测试类时生成的测试报告的结果。

![hosw 0909](img/hosw_0909.png)

###### 图 9-9。使用 Extent Reports 生成的测试报告

Extent Reports 的一个不便之处是我们需要显式地将每个测试添加到报告器中。解决此问题的一个可能方法是使用自定义测试监听器（如“测试监听器”中所述）来组织报告的公共逻辑。

另一个生成丰富测试报告的可能库是[Allure](http://allure.qatools.ru)，一个开源报告框架，用于生成 Java、Python 和 JavaScript 等不同编程语言的测试报告。Allure 和 Extent Reports 之间一个显著的区别在于 Allure 使用了在构建工具 Maven 或 Gradle 中配置的测试监听器（详见附录 C 了解有关此配置的详细信息）。因此，我们无需更改测试套件即可生成 Allure 报告。表格 9-1 总结了使用 Maven 和 Gradle 创建 Allure 报告所需的命令。

表格 9-1\. 使用 Allure 生成测试报告的 Maven 和 Gradle 命令

| Maven | Gradle | 描述 |
| --- | --- | --- |

|

```java
mvn test
mvn allure:report
mvn allure:serve
```

|

```java
gradle test
gradle allureReport
gradle allureServe
```

| 运行测试用例，生成目标文件夹中的报告

使用本地 Web 服务器打开 HTML 报告（如图 9-10 所示） |

![hosw 0910](img/hosw_0910.png)

###### 图 9-10\. 使用 Allure 生成并本地展示的测试报告

# 行为驱动开发

如第一章介绍的，行为驱动开发（BDD）是一种软件方法论，促进使用高级用户场景开发和测试软件系统。不同的工具实现了 BDD 方法论。其中最流行的之一是[Cucumber](https://cucumber.io)。Cucumber 根据用*Gherkin*语言编写的用户故事执行测试，这种语言是一种基于自然语言（如英语和其他语言）的人类可读标记。Gherkin 旨在供非程序员（例如客户或最终用户）使用，其主要关键字如下（详见[Gherkin 用户手册](https://cucumber.io/docs/gherkin)获取更多信息）：

特性

被测试的软件功能的高级描述。

场景

描述业务规则的具体测试。场景描述了 Gherkin 行话中称为*步骤*的不同信息，例如：

给定

前提条件和初始状态

当

用户动作

并且

附加用户操作

然后

预期结果

示例 9-13 展示了一个包含两个场景（登录成功和失败）的 Gherkin 特性。

##### 示例 9-13\. 用于登录实践站点的 Gherkin 场景

```java
Feature: Login in practice site ![1](img/1.png)

  Scenario: Successful login ![2](img/2.png)
    Given I use "Chrome" ![3](img/3.png)
    When I navigate to
        "https://bonigarcia.dev/selenium-webdriver-java/login-form.html" ![4](img/4.png)
    And I log in with the username "user" and password "user" ![5](img/5.png)
    And I click Submit
    Then I should see the message "Login successful" ![6](img/6.png)

  Scenario: Failure login ![7](img/7.png)
    Given I use "Chrome"
    When I navigate to
        "https://bonigarcia.dev/selenium-webdriver-java/login-form.html"
    And I log in with the username "bad-user" and password "bad-password"
    And I click Submit
    Then I should see the message "Invalid credentials"
```

![1](img/#co_third_party_integrations_CO14-1)

功能描述

![2](img/#co_third_party_integrations_CO14-2)

第一个场景（登录成功）

![3](img/#co_third_party_integrations_CO14-3)

要使用的浏览器

![4](img/#co_third_party_integrations_CO14-4)

网页 URL

![5](img/#co_third_party_integrations_CO14-5)

一组动作（输入凭据并点击提交按钮）

![6](img/#co_third_party_integrations_CO14-6)

预期的消息

![7](img/#co_third_party_integrations_CO14-7)

第二个场景（登录失败）

要将 Gherkin 场景作为测试用例运行，我们首先必须创建相应的*步骤定义*。步骤定义是使用在场景中指定信息的*glue code*来执行 SUT。在 Java 中，我们使用注解（如`@Given`、`@Then`、`@When`或`@And`）来装饰实现每个步骤的方法。这些注解包含一个字符串值，用于映射每个步骤定义和参数。Example 9-14 展示了用于 Gherkin 场景的步骤定义，该场景在 Example 9-13 中定义。我们使用 Selenium WebDriver API 来执行导航、网页元素交互等所需的操作。

##### Example 9-14\. 使用 Cucumber 登录练习站点的步骤

```java
public class LoginSteps {

    private WebDriver driver;

    @Given("I use {string}") ![1](img/1.png)
    public void iUse(String browser) {
        driver = WebDriverManager.getInstance(browser).create();
    }

    @When("I navigate to {string}") ![2](img/2.png)
    public void iNavigateTo(String url) {
        driver.get(url);
    }

    @And("I log in with the username {string} and password {string}") ![3](img/3.png)
    public void iLogin(String username, String password) {
        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);

    }

    @And("I click Submit") ![4](img/4.png)
    public void iPressEnter() {
        driver.findElement(By.cssSelector("button")).click();
    }

    @Then("I should see the message {string}") ![5](img/5.png)
    public void iShouldSee(String result) {
        try {
            String bodyText = driver.findElement(By.tagName("body")).getText();
            assertThat(bodyText).contains(result);
        } finally {
            driver.quit();
        }
    }

}
```

![1](img/#co_third_party_integrations_CO15-1)

我们使用第一步创建一个`WebDriver`实例。

![2](img/#co_third_party_integrations_CO15-2)

我们打开 URL。

![3](img/#co_third_party_integrations_CO15-3)

我们输入凭据。

![4](img/#co_third_party_integrations_CO15-4)

我们点击提交按钮。

![5](img/#co_third_party_integrations_CO15-5)

我们验证页面上是否有预期的消息。

最后，我们需要将步骤定义作为测试用例运行。通常情况下，我们使用单元测试框架创建该测试。这个测试在与 Cucumber 集成时依赖于单元测试框架，换句话说，不同的是在 JUnit 4 中（见 Example 9-15），JUnit 5 中（见 Example 9-16）和 TestNG 中（见 Example 9-17）。生成的测试以通常的方式执行（即使用 Shell 或 IDE）。

###### 注意

Selenium-Jupiter 在与 Cucumber 集成时并未提供任何额外功能，因此在示例存储库中的 Selenium-Jupiter 项目中，使用的是默认的 JUnit 5 过程。

##### Example 9-15\. 使用 JUnit 4 进行 Cucumber 测试

```java
@RunWith(Cucumber.class) ![1](img/1.png)
@CucumberOptions(features = "classpath:io/github/bonigarcia", glue = {
        "io.github.bonigarcia" }) ![2](img/2.png)
public class CucumberTest {

}
```

![1](img/#co_third_party_integrations_CO16-1)

我们使用 Cucumber 运行器来执行步骤定义作为测试用例。

![2](img/#co_third_party_integrations_CO16-2)

我们指定了 Gherkin 场景的位置。在这种情况下，特性位于项目类路径中的文件夹`io/github/bonigarcia`（具体而言，在`src/test/resources`文件夹中）。此注解还指定了要搜索 glue code（即步骤定义）的初始包。

##### Example 9-16\. 使用 JUnit 5 进行 Cucumber 测试

```java
@Suite ![1](img/1.png)
@IncludeEngines("cucumber") ![2](img/2.png)
@SelectClasspathResource("io/github/bonigarcia") ![3](img/3.png)
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "io.github.bonigarcia") ![4](img/4.png)
public class CucumberTest {

}
```

![1](img/#co_third_party_integrations_CO17-1)

我们需要使用 JUnit 5 套件模块来运行 Cucumber 测试。

![2](img/#co_third_party_integrations_CO17-2)

我们在 JUnit 平台中包含了 Cucumber 引擎。

![3](img/#co_third_party_integrations_CO17-3)

我们指定了项目类路径中特性文件的路径。

![4](img/#co_third_party_integrations_CO17-4)

我们将初始包设定为搜索粘合代码。

##### 例 9-17。使用 TestNG 进行 Cucumber 测试

```java
@CucumberOptions(features = "classpath:io/github/bonigarcia", glue = {
        "io.github.bonigarcia" }) ![1](img/1.png)
public class CucumberTest extends AbstractTestNGCucumberTests { ![2](img/2.png)

}
```

![1](img/#co_third_party_integrations_CO18-1)

我们指定 Gherkin 场景的位置和搜索粘合代码的包的位置。

![2](img/#co_third_party_integrations_CO18-2)

我们扩展 TestNG 父类以运行 Cucumber 测试。

# Web 框架

Web 框架是旨在支持 Web 应用程序和服务开发的软件框架。Java 语言中最流行的框架之一是 [Spring Framework](https://spring.io)。Spring 是一个用于构建 Java 应用程序（包括企业 Web 应用程序和服务）的开源框架。Spring 的核心技术被称为*控制反转*（IoC），它是一种在使用这些对象的类之外创建实例的过程。Spring 术语中称这些对象为*bean*或*component*，它们稍后按需作为依赖项由 Spring IoC 容器注入。

以下示例展示了用 [Spring-Boot](https://spring.io/projects/spring-boot) 创建的本地 Web 应用程序的基本测试，Spring-Boot 是 Spring 系列的一个子项目，通过惯例优于配置和自动发现功能简化了基于 Spring 的应用程序的开发。此外，Spring-Boot 提供了一个嵌入式 Web 服务器，以便轻松开发 Web 应用程序。在这种项目中与 Selenium WebDriver 的集成有助于通过在每个测试案例中自动部署在嵌入式 Web 服务器中来简化 Spring 基础 Web 应用程序的测试过程。

用于集成 Spring-Boot 与 Selenium WebDriver 和本书中使用的单元测试框架的代码是不同的。示例 9-18 显示了一个集成 Spring-Boot 和 JUnit 4 的测试。示例 9-19 展示了使用 TestNG 时的差异，示例 9-20 说明了如何使用 JUnit 5，最后，示例 9-21 展示了一个基于 JUnit 5 和 Selenium-Jupiter 的 Spring 测试。

##### 例 9-18。使用 Spring-Boot 和 JUnit 4 进行测试

```java
@RunWith(SpringRunner.class) ![1](img/1.png)
@SpringBootTest(classes = SpringBootDemoApp.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) ![2](img/2.png)
public class SpringBootJUnit4Test {

    private WebDriver driver;

    @LocalServerPort ![3](img/3.png)
    protected int serverPort;

    @Before
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @After
    public void teardown() {
        driver.quit();
    }

    @Test
    public void testSpringBoot() {
        driver.get("http://localhost:" + serverPort);
        String bodyText = driver.findElement(By.tagName("body")).getText();
        assertThat(bodyText)
                .contains("This is a local site served by Spring-Boot");
    }

}
```

![1](img/#co_third_party_integrations_CO19-1)

我们在 JUnit 4 中使用 Spring 运行器。

![2](img/#co_third_party_integrations_CO19-2)

我们使用 Spring-Boot 的测试注解来定义 Spring-Boot 类名。此外，我们指定 Web 应用程序使用随机可用端口部署。

![3](img/#co_third_party_integrations_CO19-3)

我们将 Web 应用程序端口注入为类属性。

##### 例 9-19。使用 Spring-Boot 和 TestNG 进行测试

```java
@SpringBootTest(classes = SpringBootDemoApp.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) ![1](img/1.png)
public class SpringBootNGTest extends AbstractTestNGSpringContextTests { ![2](img/2.png)

    // Same logic as the previous test 
}
```

![1](img/#co_third_party_integrations_CO20-1)

我们在 JUnit 4 中也同样使用 `@SpringBootTest` 注解。

![2](img/#co_third_party_integrations_CO20-2)

我们扩展 TestNG 父类以使用 Spring 上下文运行此测试。

##### 例 9-20。使用 Spring-Boot 和 JUnit 5 进行测试

```java
@ExtendWith(SpringExtension.class) ![1](img/1.png)
@SpringBootTest(classes = SpringBootDemoApp.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) ![2](img/2.png)
class SpringBootJupiterTest {

    // Same logic as the previous test 
}
```

![1](img/#co_third_party_integrations_CO21-1)

我们使用 JUnit 5 的 Spring 扩展将 Spring 上下文集成到 Jupiter 测试中。

![2](img/#co_third_party_integrations_CO21-2)

我们使用 Spring-Boot 来启动我们的 Spring 应用程序上下文，就像之前的例子中一样。

##### 示例 9-21\. 使用 Spring-Boot 和 JUnit 5 加上 Selenium-Jupiter 进行测试

```java
@ExtendWith({ SeleniumJupiter.class, SpringExtension.class }) ![1](img/1.png)
@SpringBootTest(classes = SpringBootDemoApp.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class SpringBootSelJupTest {

    @LocalServerPort
    protected int serverPort;

    @Test
    void testSpringBoot(ChromeDriver driver) { ![2](img/2.png)
        driver.get("http://localhost:" + serverPort);
        String bodyText = driver.findElement(By.tagName("body")).getText();
        assertThat(bodyText)
                .contains("This is a local site served by Spring-Boot");
    }

}
```

![1](img/#co_third_party_integrations_CO22-1)

在 Selenium-Jupiter 的情况下，我们使用了两个 JUnit 5 扩展（用于 Spring 和 Selenium WebDriver）。

![2](img/#co_third_party_integrations_CO22-2)

如同在 Selenium-Jupiter 中一贯的做法，我们使用了 JUnit 5 的参数解析机制来声明在本测试中使用的 `WebDriver` 实例的类型。

# 总结与展望

本章为使用 Selenium WebDriver 进行端到端测试的不同技术（如工具、库和框架）集成提供了实际概述。首先，我们使用 Awaitility（处理异步操作的库）等待 Selenium WebDriver 下载文件。执行相同用例（即下载文件）的另一种库是 Apache HttpClient。然后，我们使用 BrowserMob 代理拦截 Selenium WebDriver 测试交换的 HTTP 流量。接下来的技术组集中于使用 Selenium WebDriver 进行非功能测试：BrowserMob（为性能测试创建 JMeter 测试计划）、OWASP ZAP（安全测试）和 Axe（可访问性测试）。然后，我们使用 Selenide 提供的流畅 API 和 Java Faker 为 Selenium WebDriver 测试创建虚拟测试数据，并使用 Extent Reports 和 Allure 生成丰富的测试报告。最后，我们了解了如何将 Cucumber（BDD 框架）和 Spring（Java 和 Web 框架）与 Selenium WebDriver 集成。

下一章通过介绍 Selenium WebDriver 的补充框架来总结本书，即用于测试 REST 服务的 REST Assured 和用于测试移动应用程序的 Appium。最后，本章介绍了几个流行的 Selenium WebDriver 浏览器自动化空间的替代品：Cypress、WebDriverIO、TestCafe、Puppeteer 和 Playwright。
