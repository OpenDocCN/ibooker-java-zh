# 第八章：测试框架具体细节

在本书中提供的示例中，我推荐在使用不同单元测试框架（JUnit 4、JUnit 5（单独或与 Selenium-Jupiter 扩展一起使用）或 TestNG）时，将对 Selenium WebDriver API 的调用嵌入到使用注解`@Test`装饰的 Java 方法中。在执行常规测试时，使用一个或另一个测试框架的差异是微小的。然而，每个测试框架都有特定的特性用于不同的用例。本章总结了一些用于实现 Selenium WebDriver 测试的这些特性。像往常一样，你可以在本书示例存储库中找到本章的源代码。你可以使用这些示例来比较和选择最适合你特定需求的单元测试框架。

# 参数化测试

单元测试框架通常支持的一个广泛特性是创建*参数化测试*。这个特性使得可以使用不同的参数多次执行测试。虽然我们可以在 JUnit（4 和 5）和 TestNG 中都实现参数化测试，但每种实现之间存在显著差异。

## JUnit 4

我们需要在 JUnit 4 中使用一个名为`Parameterized`的*测试运行器*来实现参数化测试。在 JUnit 4 中，测试运行器是负责运行测试的 Java 类。我们使用 JUnit 4 注解`@RunWith`来装饰一个 Java 类来指定测试运行器。然后，我们需要使用 JUnit 4 注解`@Parameters`来装饰提供测试参数的方法。有两种方法可以将这些参数注入到测试类中：在测试类构造函数中或作为使用注解`@Parameter`装饰的类属性。示例 8-1 展示了一个测试用例，其中使用第二种技术注入测试参数。这个示例使用不同的凭据（用户名和密码）执行相同的登录测试。因此，网页提供的消息是不同的（登录成功或无效凭据）。

##### 示例 8-1。使用 JUnit 4 进行参数化测试

```java
@RunWith(Parameterized.class) ![1](img/1.png)
public class ParameterizedJUnit4Test {

    WebDriver driver;

    @Parameter(0) ![2](img/2.png)
    public String username;

    @Parameter(1)
    public String password;

    @Parameter(2)
    public String expectedText;

    @Before
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @After
    public void teardown() {
        driver.quit();
    }

    @Parameters(name = "{index}: username={0} password={1} expectedText={2}") ![3](img/3.png)
    public static Collection<Object[]> data() {
        return Arrays
                .asList(new Object[][] { { "user", "user", "Login successful" },
                        { "bad-user", "bad-passwd", "Invalid credentials" } }); ![4](img/4.png)
    }

    @Test
    public void testParameterized() { ![5](img/5.png)
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/login-form.html");

        driver.findElement(By.id("username")).sendKeys(username);
        driver.findElement(By.id("password")).sendKeys(password);
        driver.findElement(By.cssSelector("button")).click();

        String bodyText = driver.findElement(By.tagName("body")).getText();
        assertThat(bodyText).contains(expectedText); ![6](img/6.png)
    }

}
```

![1](img/#co_testing_framework_specifics_CO1-1)

我们为这个 Java 类指定了`Parameterized`测试运行器。

![2](img/#co_testing_framework_specifics_CO1-2)

我们将三个测试参数注入为类属性：用户名（索引`0`）、密码（索引`1`）和预期文本（索引`2`）。

![3](img/#co_testing_framework_specifics_CO1-3)

我们在一个返回泛型参数集合（`Collection<Object[]>`）的方法中指定测试参数。

![4](img/#co_testing_framework_specifics_CO1-4)

我们返回一个包含三个`String`集合的集合，用作测试参数。每个条目的值将使用之前声明的三个参数（用户名、密码和预期文本）注入。

![5](img/#co_testing_framework_specifics_CO1-5)

|

|

在参数化测试中，JUnit 4 和 TestNG 之间的一个显著差异是，在 TestNG 中，参数（例如本示例中的用户名、密码和预期测试）作为测试方法参数注入。

###### |

|

## ![2](img/#co_testing_framework_specifics_CO2-2)

|

##### 示例 8-2\. 使用 TestNG 进行参数化测试

```java
public class ParameterizedNGTest {

    WebDriver driver;

    @BeforeMethod
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterMethod
    public void teardown() {
        driver.quit();
    }

    @DataProvider(name = "loginData") ![1](img/1.png)
    public static Object[][] data() {
        return new Object[][] { { "user", "user", "Login successful" },
                { "bad-user", "bad-passwd", "Invalid credentials" } };
    }

    @Test(dataProvider = "loginData") ![2](img/2.png)
    public void testParameterized(String username, String password,
            String expectedText) { ![3](img/3.png)
        // Same test logic than the example before
    }

}
```

![1](img/#co_testing_framework_specifics_CO2-1)

|

|

我们指定此测试将使用之前称为 `loginData` 的数据提供器。

![6](img/#co_testing_framework_specifics_CO1-6)

我们可以使用注解 `@DataProvider` 来装饰提供 TestNG 参数化测试参数的方法。正如您在 示例 8-2 中所看到的，此方法返回一个通用 Java 对象的双数组。注解 `@Data​Pro⁠vider` 应该提供一个名称作为属性。稍后此名称将在 `@Test` 方法中用于指定数据提供器。最后，参数被注入到测试方法中。

## 我们断言预期数据（根据提供的凭据不同而异）在页面正文中是可用的。

|

+   |

+   |

|

| Java 枚举的常量 |
| --- |
| 实现 `ArgumentsProvider` 接口的类 |

TestNG

```java
@ValueSource
```

| 字面值数组 |
| --- |

```java
@ParameterizedTest
@ValueSource(strings = { "Hi", "Bye" })
void test(String argument) {
    log.debug("arg: {}", argument);
}
```

|

```java
arg: Hi
arg: Bye
```

|

|

```java
@EnumSource
```

| Jupiter（JUnit 5 的编程和扩展模型）为创建参数化测试提供了强大的机制。简言之，要在 JUnit 5 中实现这些测试，我们需要两个元素： |
| --- |

```java
@ParameterizedTest
@EnumSource(TimeUnit.class)
void test(TimeUnit argument) {
    log.debug("{}", argument);
}
```

| 提供一个值流的类的静态方法 |

```java
NANOSECONDS
MICROSECONDS
MILLISECONDS
SECONDS
MINUTES
HOURS
DAYS
```

在测试逻辑中（将根据每个数据输入执行两次），我们尝试使用作为参数提供的用户名和密码登录练习站点。

| 单个 `null` 参数 |

```java
@MethodSource
```

| 注解 `@ParameterizedTest`（而不是通常的 `@Test` 注解），用于装饰注入参数的测试方法。 |
| --- |

```java
static IntStream intProvider() {
    return IntStream.of(0, 1);
}

@ParameterizedTest
@MethodSource("intProvider")
void test(int argument) {
    log.debug("arg: {}", argument);
    assertNotNull(argument);
}
```

|

```java
arg: 0
arg: 1
```

|

|

```java
@CsvSource
```

| 注解内的逗号分隔值（CSV） |
| --- |

```java
@ParameterizedTest
@CsvSource({ "hello, 1", "world, 2"})
void test(String first, int second) {
    log.debug("{} and {} ", first,
            second);
}
```

表 8-1\. JUnit 5 中的参数提供器

```java
hello and 1
world and 2
```

| 单个空参数 |

|

```java
@CsvFileSource
```

| |
| --- |

```java
@ParameterizedTest
@CsvFileSource(resources =
            "/input.csv")
void test(String first, int second) {
    log.debug("{} and {} ", first,
            second);
}
```

|

```java
hi and 3
there and 4
```

警告

![3](img/#co_testing_framework_specifics_CO2-3)

```java
@ArgumentsSource
```

| |
| --- |

```java
@ParameterizedTest
@ArgumentsSource(MyArgs.class)
void test(String first, int second) {
    log.debug("{} and {} ", first,
            second);
}

public class MyArgs implements
            ArgumentsProvider {
  @Override
  public Stream<? extends
        Arguments> provideArguments(
        ExtensionContext context) {
     return Stream.of(Arguments.
        of("hi", 5), Arguments.
        of("there", 6));
  }
}
```

|

```java
hi and 5
there and 6
```

我们创建一个作为数据提供器的方法。

参数提供器，用于参数化测试的数据源。表 8-1 提供了这些参数提供器的综合概述。

```java
@NullSource
```

| 一个 `null` 加一个空参数 |
| --- |

```java
@ParameterizedTest
@ValueSource(strings = { "one",
            "two" })
@NullSource
void test(String argument) {
    log.debug("arg: {}", argument);
}
```

| 位于类路径中的文件中以 CSV 格式的值 |

```java
arg: one
arg: two
arg: null
```

JUnit 4 的最显著限制之一是每个 Java 类只能使用一个测试运行器。换句话说，JUnit 4 中的测试运行器不可组合。为了克服这一限制（以及其他限制），JUnit 团队于 2017 年发布了 JUnit 5。

JUnit 5

```java
@EmptySource
```

| |
| --- |

```java
@ParameterizedTest
@ValueSource(strings = { "three",
            "four" })
@EmptySource
void test(String argument) {
    log.debug("arg: {}", argument);
}
```

|

```java
arg: three
arg: four
arg:
```

| --- | --- | --- | --- |

|

```java
@NullAndEmptySource
```

| 注解 | 描述 | 示例 | 示例输出 |
| --- | --- | --- | --- |

```java
@ParameterizedTest
@ValueSource(strings = { "five",
            "six" })
@NullAndEmptySource
void test(String arg) {
    log.debug("arg: {}", arg);
}
```

|

```java
arg: five
arg: six
arg: null
arg:
```

|

Example 8-3 展示了与前面示例相同的参数化测试的 Jupiter 版本。我们可以使用不同的参数提供者来实现此参数化测试。在这种情况下，我们使用`@MethodSource`返回参数流。适合这种测试的另一种选择是使用`@CsvSource`将输入数据和预期结果嵌入为 CSV 格式。

##### Example 8-3\. 使用 JUnit 5 的参数化测试

```java
class ParameterizedJupiterTest {

    WebDriver driver;

    @BeforeEach
    void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterEach
    void teardown() {
        driver.quit();
    }

    static Stream<Arguments> loginData() { ![1](img/1.png)
        return Stream.of(Arguments.of("user", "user", "Login successful"),
                Arguments.of("bad-user", "bad-passwd", "Invalid credentials"));
    }

    @ParameterizedTest ![2](img/2.png)
    @MethodSource("loginData") ![3](img/3.png)
    void testParameterized(String username, String password,
            String expectedText) { ![4](img/4.png)
        // Same test logic than the examples before
    }

}
```

![1](img/#co_testing_framework_specifics_CO3-1)

我们定义一个静态方法作为`@MethodSource`中的参数提供者。

![2](img/#co_testing_framework_specifics_CO3-2)

而不是常规的`@Test`，我们实现了一个参数化测试。

![3](img/#co_testing_framework_specifics_CO3-3)

参数提供者与`loginData`方法提供的数据相关联。

![4](img/#co_testing_framework_specifics_CO3-4)

参数被注入到测试方法中。

## Selenium-Jupiter

当使用 Selenium-Jupiter 时，你可以使用相同的方法来实现 JUnit 5 的参数化测试。唯一的区别在于，你通过 Selenium-Jupiter 委托创建和销毁`WebDriver`对象。Example 8-4 演示了如何实现前面章节中解释的相同测试（即参数化登录），但使用 Selenium-Jupiter。

##### Example 8-4\. 使用 JUnit 5 和 Selenium-Jupiter 的参数化测试

```java
@ExtendWith(SeleniumJupiter.class)
class ParameterizedSelJupTest {

    static Stream<Arguments> loginData() {
        return Stream.of(Arguments.of("user", "user", "Login successful"),
                Arguments.of("bad-user", "bad-passwd", "Invalid credentials"));
    }

    @ParameterizedTest
    @MethodSource("loginData")
    void testParameterized(String username, String password,
            String expectedText, ChromeDriver driver) { ![1](img/1.png)
        // Same test logic than the examples before
    }

}
```

![1](img/#co_testing_framework_specifics_CO4-1)

当在 Jupiter 测试中使用不同的参数解析器时，按照惯例，我们必须首先声明由`@ParameterizedTest`注入的参数，然后是由扩展（在本例中为 Selenium-Jupiter，用于`WebDriver`对象）注入的参数。

## 跨浏览器测试

跨浏览器测试是一种功能测试，通过使用不同类型的 Web 浏览器验证 Web 应用程序是否按预期工作。通过使用浏览器类型（例如 Chrome、Firefox、Edge 等）作为测试参数，可以实现参数化测试的可能方法。接下来的章节描述了如何使用单元测试框架的能力来进行适用于跨浏览器测试的参数化测试。在这些示例中，我们将使用本地浏览器（Chrome、Firefox 和 Edge）。进行跨浏览器测试的另一种方法是使用远程浏览器（从 Selenium 服务器、云提供商或 Docker 中），详见第六章。

## JUnit 4

示例 8-5 展示了使用 JUnit 4 实现的跨浏览器测试。 我们使用 WebDriverManager 来简化参数化。 正如“通用管理器”中所解释的那样，WebDriverManager 可以根据参数的值使用一个或另一个管理器。 此参数可以是 `WebDriver` 类、枚举或浏览器名称。 在以下示例中我们使用了后者（虽然您可以在[示例库](https://github.com/bonigarcia/selenium-webdriver-java)中找到替代方法）。

##### 示例 8-5\. 使用 JUnit 4 进行跨浏览器测试

```java
@RunWith(Parameterized.class)
public class CrossBrowserJUnit4Test {

    WebDriver driver;

    @Parameter(0)
    public String browserName;

    @Parameters(name = "{index}: browser={0}")
    public static Collection<Object[]> data() {
        return Arrays.asList(
                new Object[][] { { "chrome" }, { "edge" }, { "firefox" } }); ![1](img/1.png)
    }

    @Before
    public void setup() {
        driver = WebDriverManager.getInstance(browserName).create(); ![2](img/2.png)
    }

    @After
    public void teardown() {
        driver.quit();
    }

    @Test
    public void testCrossBrowser() { ![3](img/3.png)
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_testing_framework_specifics_CO5-1)

我们使用它们的名称指定了三个浏览器。

![2](img/#co_testing_framework_specifics_CO5-2)

我们使用 WebDriverManager 通用管理器，使用这些浏览器名称作为参数。 选择一种或另一种浏览器的替代方法是使用通用管理器而不带参数（即使用方法 `.getInstance()`，如“通用管理器”中所述），然后使用 Java 系统属性 `wdm.defaultBrowser` 来为测试（或测试套件）参数化（例如，在使用 Maven 或 Gradle 运行时）。

![3](img/#co_testing_framework_specifics_CO5-3)

此测试执行三次，每次使用不同的浏览器（Chrome、Edge 和 Firefox）。

## TestNG

示例 8-6 展示了相同的跨浏览器测试，这次使用 TestNG。 在这种情况下，测试参数（浏览器名称）被注入到测试方法中。

##### 示例 8-6\. 使用 TestNG 进行跨浏览器测试

```java
public class CrossBrowserNGTest {

    WebDriver driver;

    @DataProvider(name = "browsers")
    public static Object[][] data() {
        return new Object[][] { { "chrome" }, { "edge" }, { "firefox" } };
    }

    @AfterMethod
    public void teardown() {
        driver.quit();
    }

    @Test(dataProvider = "browsers")
    public void testCrossBrowser(String browserName) {
        driver = WebDriverManager.getInstance(browserName).create(); ![1](img/1.png)

        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_testing_framework_specifics_CO6-1)

我们需要在测试逻辑中创建 `WebDriver` 实例，因为在使用 TestNG 时测试参数被注入到测试方法中。

## JUnit 5

示例 8-7 展示了遵循 Jupiter 模型的同一跨浏览器测试。 再次使用 WebDriverManager 创建 `WebDriver` 实例，使用浏览器名称作为参数。 由于这些参数是字符串，我们使用 `@ValueSource` 作为参数提供程序。

##### 示例 8-7\. 使用 JUnit 5 进行跨浏览器测试

```java
class CrossBrowserJupiterTest {

    WebDriver driver;

    @AfterEach
    void teardown() {
        driver.quit();
    }

    @ParameterizedTest
    @ValueSource(strings = { "chrome", "edge", "firefox" })
    void testCrossBrowser(String browserName) {
        driver = WebDriverManager.getInstance(browserName).create();  ![1](img/1.png)

        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_testing_framework_specifics_CO7-1)

在 Jupiter 中，参数化测试中的参数被注入到测试方法中。 因此，我们需要在测试逻辑中创建驱动程序实例。

## Selenium-Jupiter

Selenium-Jupiter 提供了一个补充功能，用于创建跨浏览器测试，称为*测试模板*。 测试模板是 Jupiter 支持的一种特殊的参数化测试，在这种测试中，扩展程序收集参数。 Selenium-Jupiter 使用这一特性以一种名为*浏览器场景*的自定义 JSON 符号的全面方式来指定不同的浏览器方面（如类型、版本、参数和能力）。 您可以在[Selenium-Jupiter 文档](https://bonigarcia.dev/selenium-jupiter/#template-tests)中找到更多关于此功能的详细信息。

示例 8-8 展示了一个样本浏览器场景。这个 JSON 存储在名为`browsers.json`的文件中，这是模板测试使用的默认名称。示例 8-9 展示了使用这个浏览器场景的模板测试。

##### 示例 8-8\. Selenium-Jupiter 中用于测试模板的浏览器场景

```java
{
   "browsers": 
      [
         {
            "type": "chrome" ![1
         }
      ],
      
         {
            "type": "edge", ![2
             "arguments" : [
                "--headless"
             ]
         }
      ],
      
         {
            "type": "firefox-in-docker", ![3
            "version": "93"
         }
      ]
   ]
}
```

![1](img/#co_testing_framework_specifics_CO8-1)

这个浏览器场景包含三个浏览器。第一个是本地的 Chrome 浏览器。

![2](img/#co_testing_framework_specifics_CO8-2)

第二个浏览器是本地的无头 Edge 浏览器。

![3](img/#co_testing_framework_specifics_CO8-3)

第三个浏览器是在 Docker 容器中执行的 Firefox 93 版本。

##### 示例 8-9\. 使用 Selenium-Jupiter 在 JUnit 5 中进行跨浏览器测试模板

```java
@EnabledIfDockerAvailable ![1](img/1.png)
@ExtendWith(SeleniumJupiter.class)
class CrossBrowserJsonSelJupTest {

    @TestTemplate ![2](img/2.png)
    void testCrossBrowser(WebDriver driver) { ![3](img/3.png)
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getTitle()).contains("Selenium WebDriver");
    }

}
```

![1](img/#co_testing_framework_specifics_CO9-1)

当 Docker 不可用时（因为场景中的一个浏览器使用 Docker），我们使用 Selenium-Jupiter 注解来跳过这个测试。

![2](img/#co_testing_framework_specifics_CO9-2)

我们需要使用`@TestTemplate`修饰测试方法，而不是通常的`@Test`注解。

![3](img/#co_testing_framework_specifics_CO9-3)

我们使用通用的`WebDriver`来注入驱动实例。另外，`RemoteWebDriver`在测试模板中也是有效的。

# 测试分类和过滤

基于 Selenium WebDriver 构建测试套件时（特别是测试数量很多时），常见的需求是只执行一组测试。有多种方法可以实现单个或组测试的执行。在使用 IDE 运行测试时，可以选择要执行的具体测试。在使用命令行时，可以使用其他机制来选择这些测试。

乍一看，我们可以使用构建工具提供的过滤机制。例如，Maven 和 Gradle 允许基于测试类和方法名包含或排除特定测试。这些命令的基本语法在附录 C 中介绍。表 8-2 展示了使用这些命令的几个常见示例。请注意，通配符`*`在这些示例中用于匹配测试类名中的任意字符。

表 8-2\. Maven 和 Gradle 命令示例，包括和排除测试

| 描述 | Maven | Gradle |
| --- | --- | --- |
| 运行以 *Hello* 开头的测试 |

```java
mvn -B test
    -Dtest=Hello*
```

|

```java
gradle test
    --tests Hello*
```

|

| 运行包含 *Basic* 或 *Timeout* 的测试 |
| --- |

```java
mvn test
    -Dtest=*Basic*,*Timeout*
```

|

```java
gradle test
    --tests *Basic* --tests *Timeout*
```

|

| 运行除了以 *Firefox* 开头的测试 |
| --- |

```java
mvn test
    -Dtest=!*Firefox*
```

|

```java
gradle test
    -PexcludeTests=**/*Firefox*
```

|

| 运行除了以 *Docker* 开头或包含 *Remote* 的测试 |
| --- |

```java
mvn test
    -Dtest=!Docker*,!*Remote*
```

|

```java
gradle test
    -PexcludeTests=**/Docker*,**/*Remote*
```

|

除了构建工具，我们还可以利用单元测试框架提供的内置功能，对测试进行分类（也称为分组或标记）并基于这些分类进行过滤。下面的子节将详细解释如何操作。

## JUnit 4

JUnit 4 提供了`@Category`注解来对测试进行分组。我们需要在此注解中指定一个或多个 Java 类作为属性。然后，我们可以使用这些类来选择和执行属于一个或多个类别的测试。示例 8-10 展示了使用此功能的基本类。

##### 示例 8-10\. 使用类别和 JUnit 4 进行测试

```java
public class CategoriesJUnit4Test {

    WebDriver driver;

    @Before
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @After
    public void teardown() {
        driver.quit();
    }

    @Test
    @Category(WebForm.class) ![1](img/1.png)
    public void testCategoriesWebForm() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/web-form.html");
        assertThat(driver.getCurrentUrl()).contains("web-form");
    }

    @Test
    @Category(HomePage.class) ![2](img/2.png)
    public void tesCategoriestHomePage() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getCurrentUrl()).doesNotContain("web-form");
    }

}
```

![1](img/#co_testing_framework_specifics_CO10-1)

`WebForm`是示例库中的一个空接口。

![2](img/#co_testing_framework_specifics_CO10-2)

`HomePage`是示例库中的另一个空接口。

然后我们可以根据其组执行测试。例如，以下命令展示了运行属于`HomePage`类别的测试的 Maven 和 Gradle 命令。

```java
mvn test -Dgroups=
    io.github.bonigarcia.webdriver.junit4.ch08.categories.HomePage
gradle test -Pgroups=
    io.github.bonigarcia.webdriver.junit4.ch08.categories.HomePage
```

我们可以将此过滤与 Maven 和 Gradle 支持结合使用，根据类名选择测试。例如，以下命令执行属于`HomePage`类别的测试，但仅在测试类`CategoriesJUnit4Test`中。

```java
mvn test -Dtest=CategoriesJUnit4Test -DexcludedGroups=
    io.github.bonigarcia.webdriver.junit4.ch08.categories.HomePage
gradle test --tests CategoriesJUnit4Test -PexcludedGroups=
    io.github.bonigarcia.webdriver.junit4.ch08.categories.HomePage
```

## TestNG

TestNG 也允许对测试进行分组。示例 8-11 演示了此功能的基本用法。总之，`@Test`注解允许为这些组指定字符串标签。

##### 示例 8-11\. 使用组和 TestNG 进行测试

```java
public class CategoriesNGTest {

    WebDriver driver;

    @BeforeMethod(alwaysRun = true) ![1](img/1.png)
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterMethod(alwaysRun = true)
    public void teardown() {
        driver.quit();
    }

    @Test(groups = { "WebForm" }) ![2](img/2.png)
    public void testCategoriesWebForm() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/web-form.html");
        assertThat(driver.getCurrentUrl()).contains("web-form");
    }

    @Test(groups = { "HomePage" }) ![3](img/3.png)
    public void tesCategoriestHomePage() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getCurrentUrl()).doesNotContain("web-form");
    }

}
```

![1](img/#co_testing_framework_specifics_CO11-1)

我们将属性`alwaysRun`设置为`true`，以指示在测试执行期间不过滤设置和拆卸方法。

![2](img/#co_testing_framework_specifics_CO11-2)

我们将组名`WebForm`分配给该类的第一个测试。

![3](img/#co_testing_framework_specifics_CO11-3)

我们将组名`HomePage`设置为第二个测试。

然后我们可以根据这些分类使用命令行过滤测试执行。以下片段首先展示了如何执行属于`HomePage`组的测试。第二个示例说明了如何将这种分组与基于类名的 Maven 和 Gradle 过滤机制结合使用。

```java
mvn test -Dgroups=HomePage
gradle test -Pgroups=HomePage

mvn test -Dtest=CategoriesNGTest -DexcludedGroups=HomePage
gradle test --tests CategoriesNGTest -PexcludedGroups=HomePage
```

## JUnit 5

Jupiter 编程模型提供了一种基于自定义标签（称为*标签*）对测试进行分组的方式。我们使用`@Tag`注解来实现这一目的。示例 8-12 说明了此功能。

##### 示例 8-12\. 使用标签和 JUnit 5 进行测试

```java
class CategoriesJupiterTest {

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
    @Tag("WebForm") ![1](img/1.png)
    void testCategoriesWebForm() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/web-form.html");
        assertThat(driver.getCurrentUrl()).contains("web-form");
    }

    @Test
    @Tag("HomePage") ![2](img/2.png)
    void testCategoriesHomePage() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        assertThat(driver.getCurrentUrl()).doesNotContain("web-form");
    }

}
```

![1](img/#co_testing_framework_specifics_CO12-1)

我们使用标签`WebForm`标记第一个测试。

![2](img/#co_testing_framework_specifics_CO12-2)

我们使用`HomePage`标签对第二个测试进行分类。

我们可以使用这些标签在命令行执行测试时包含或排除测试。以下命令展示了 Maven 和 Gradle 的几个示例：

```java
mvn test -Dgroups=HomePage
gradle test -Pgroups=HomePage

mvn test -Dtest=CategoriesNGTest -DexcludedGroups=HomePage
gradle test --tests CategoriesNGTest -PexcludedGroups=HomePage
```

# 测试排序

在本书中使用的单元测试框架中，测试执行顺序事先是未知的。尽管如此，仍然有机制可以选择给定的执行顺序。在 Selenium WebDriver 领域中使用此功能的一个可能用例是通过不同的测试以给定顺序（即使用相同的 `WebDriver` 实例）重用同一浏览器会话与 SUT 进行交互。以下示例演示了对 JUnit 4、TestNG、JUnit 5 和 JUnit 5 加 Selenium-Jupiter 使用此功能的情况。

## JUnit 4

JUnit 4 提供了注解 `@FixMethodOrder` 来确定测试执行顺序。此注解接受一个名为 `MethodSorters` 的枚举，由以下值组成：

`NAME_ASCENDING`

按方法名按字典顺序对测试方法进行排序

`JVM`

将测试方法按 JVM 返回的顺序排序。

`DEFAULT`

将测试方法以确定性但不可预测的顺序排序

Example 8-13 显示了一个完整的测试用例，其中测试使用方法名执行。

##### 示例 8-13\. 使用 JUnit 4 对测试进行排序

```java
@FixMethodOrder(MethodSorters.NAME_ASCENDING) ![1](img/1.png)
public class OrderJUnit4Test {

    static WebDriver driver;

    @BeforeClass ![2](img/2.png)
    public static void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterClass ![3](img/3.png)
    public static void teardown() {
        driver.quit();
    }

    @Test ![4](img/4.png)
    public void testA() {
        driver.get(
                "https://bonigarcia.dev/selenium-webdriver-java/navigation1.html");
        assertBodyContains("Lorem ipsum");
    }

    @Test
    public void testB() {
        driver.findElement(By.linkText("2")).click();
        assertBodyContains("Ut enim");
    }

    @Test
    public void testC() {
        driver.findElement(By.linkText("3")).click();
        assertBodyContains("Excepteur sint");
    }

    void assertBodyContains(String text) {
        String bodyText = driver.findElement(By.tagName("body")).getText();
        assertThat(bodyText).contains(text);
    }

}
```

![1](img/#co_testing_framework_specifics_CO13-1)

我们在类级别使用注解 `@FixMethodOrder` 来确定此类中可用测试的顺序。

![2](img/#co_testing_framework_specifics_CO13-2)

我们在所有测试之前创建驱动程序实例（因为我们希望在所有测试中使用 `WebDriver` 会话）。

![3](img/#co_testing_framework_specifics_CO13-3)

我们在所有测试完成后退出驱动程序实例。因此，在该类的最后一个测试之后结束会话。

![4](img/#co_testing_framework_specifics_CO13-4)

由于测试名称按字典顺序排序（`testA`、`testB` 和 `testC`），测试执行将遵循此顺序。

## TestNG

在 TestNG 中按照每个测试使用增量优先级的简单方式对测试进行排序。Example 8-14 通过在 `@Test` 注解中使用 `priority` 属性来演示此功能。

##### 示例 8-14\. 使用 TestNG 对测试进行排序

```java
public class OrderNGTest {

    static WebDriver driver;

    @BeforeClass
    public static void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterClass
    public static void teardown() {
        driver.quit();
    }

    @Test(priority = 1)
    public void testA() {
        // Test logic
    }

    @Test(priority = 2)
    public void testB() {
        // Test logic
    }

    @Test(priority = 3)
    public void testC() {
        // Test logic
    }

}
```

## JUnit 5

Jupiter 提供了注解 `@TestMethodOrder` 来对测试进行排序。此注解可以使用以下排序实现进行配置：

`DisplayName`

根据显示名称按字母数字顺序排序测试方法。

`MethodName`

根据名称按字母数字顺序对测试方法进行排序。

`OrderAnnotation`

基于使用 `@Order` 注解指定的数字值对测试方法进行排序。Example 8-15 展示了使用此方法的测试。  

`Random`

以伪随机方式对测试方法进行排序。

##### 示例 8-15\. 使用 JUnit 5 对测试进行排序

```java
@TestMethodOrder(OrderAnnotation.class)
class OrderJupiterTest {

    static WebDriver driver;

    @BeforeAll
    static void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterAll
    static void teardown() {
        driver.quit();
    }

    @Test
    @Order(1)
    void testA() {
        // Test logic
    }

    @Test
    @Order(2)
    void testB() {
        // Test logic
    }

    @Test
    @Order(3)
    void testC() {
        // Test logic
    }

}
```

## Selenium-Jupiter

和往常一样，使用 Selenium-Jupiter 的测试也使用 Jupiter 编程模型；因此，这些特性（如测试排序）对于 Selenium-Jupiter 测试同样有效。示例 8-16 展示了之前相同的测试，使用 Selenium-Jupiter 进行驱动程序实例化。默认情况下，驱动程序对象在每次测试之前创建，并在每次测试后终止。Selenium-Jupiter 提供了`@SingleSession`注解来改变这种行为，创建所有测试之前的驱动程序实例，并在所有测试之后关闭会话。

##### 示例 8-16\. 使用 JUnit 5 和 Selenium-Jupiter 排序测试

```java
@ExtendWith(SeleniumJupiter.class)
@TestMethodOrder(OrderAnnotation.class)
@SingleSession
class OrderSelJupTest {

    WebDriver driver;

    OrderSelJupTest(ChromeDriver driver) {
        this.driver = driver;
    }

    @Test
    @Order(1)
    void testA() {
        // Test logic
    }

    @Test
    @Order(2)
    void testB() {
        // Test logic
    }

    @Test
    @Order(3)
    void testC() {
        // Test logic
    }

}
```

# 故障分析

*故障分析*（也称为*故障排除*）是收集和分析数据以发现故障原因的过程。对于 Selenium WebDriver 测试来说，这个过程可能具有挑战性，因为整个系统被测试，导致测试失败的根本原因可能有多个。例如，端到端测试失败的原因可能是客户端（前端）逻辑、服务器端（后端）逻辑，甚至是与其他组件的集成（例如数据库或外部服务）。

我们可以使用不同的技术来帮助开发人员和测试人员进行故障分析过程。这样做的典型方式是检测测试失败，并在终止驱动程序会话之前收集一些数据以发现原因。以下资产可以帮助此过程：

屏幕截图

测试失败后，Web 应用程序 UI 的图片可能有助于确定失败原因。“屏幕截图”解释了如何使用 Selenium WebDriver API 进行截图。

浏览器日志

当出现错误时，JavaScript 控制台可以是另一个潜在的信息来源。“日志收集”解释了如何进行这种日志收集。

会话记录

在使用 Docker 容器中的浏览器时，我们可以轻松记录浏览器会话。“Docker 容器中的浏览器”解释了如何使用 WebDriverManager 和 Selenium-Jupiter 实现这一点。

以下各小节提供了有关如何在测试失败时制作浏览器截图的基本示例。为此，我们需要依赖单元测试的特定功能来检测失败的测试。

## JUnit 4

JUnit 允许通过使用*规则*来调整测试的默认行为。测试类通过使用`@Rule`注解修饰类属性来定义规则。表 8-3 总结了 JUnit 4 提供的默认规则。

表 8-3\. JUnit 4 中的规则

| 规则 | 描述 | 示例 |
| --- | --- | --- |

|

```java
ErrorCollector
```

| 允许在发生异常时继续执行测试（同时收集这些异常） |
| --- |

```java
@Rule
public ErrorCollector collector =
        new ErrorCollector();

@Ignore
@Test
public void test() {
    collector.checkThat("a", equalTo("b"));
    collector.checkThat(1, equalTo(2));
}
```

|

|

```java
ExternalResource
```

| 提供一个基类，在每次测试之前设置和拆除外部资源 |
| --- |

```java
private Resource resource;

@Rule
public ExternalResource rule =
        new ExternalResource() {
    @Override
    protected void before() throws Throwable {
        resource = new Resource();
        resource.open();
    }

    @Override
    protected void after() {
        resource.close();
    }
};
```

|

|

```java
TestName
```

| 使当前测试方法可用于测试名称 |
| --- |

```java
@Rule
public TestName name = new TestName();

@Test
public void testA() {
    assertThat("testA")
        .isEqualTo(name.getMethodName());
}
```

|

|

```java
TemporaryFolder
```

| 允许创建临时文件和文件夹 |
| --- |

```java
@Rule
public TemporaryFolder folder =
        new TemporaryFolder();

@Test
public void test() throws IOException {
    File file = folder.newFile("myfile.txt");
}
```

|

|

```java
Timeout
```

| 在类中对所有测试方法应用超时 |
| --- |

```java
@Rule
public Timeout timeout =
        new Timeout(10, SECONDS);

@Test
public void test() {
    while (true);
}
```

|

|

```java
TestWatcher
```

| 允许捕获测试的多个执行阶段：`starting`、`succeeded`、`failed`、`skipped` 和 `finished`。 |
| --- |

```java
@Rule
public TestWatcher watcher =
        new TestWatcher() {
    @Override
    protected void succeeded(Description d) {
        log.debug("Test succeeded: {}",
            d.getMethodName());
    }

    @Override
    protected void failed(Throwable e,
            Description d) {
        log.debug("Test failed: {}",
            d.getMethodName());
    }
};
```

|

我们可以使用 `TestWatcher` 规则来收集 JUnit 4 失败分析的数据。Example 8-17 展示了一个在测试失败时捕获截图的测试。Example 8-18 包含了此规则的实现。正如前面提到的，我们制作浏览器截图的逻辑在 Example 8-19 中。

##### Example 8-17\. 使用 JUnit 4 分析失败的测试

```java
public class FailureJUnit4Test {

    static WebDriver driver;

    @Rule
    public TestRule testWatcher = new FailureWatcher(driver); ![1](img/1.png)

    @BeforeClass
    public static void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterClass
    public static void teardown() {
        driver.quit();
    }

    @Test
    public void testFailure() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        fail("Forced error"); ![2](img/2.png)
    }

}
```

![1](img/#co_testing_framework_specifics_CO14-1)

我们在类级别定义规则，并将驱动程序实例作为参数传递。

![2](img/#co_testing_framework_specifics_CO14-2)

我们强制此测试失败，以便使用规则来截取浏览器的截图。

##### Example 8-18\. 使用 JUnit 4 分析失败的测试

```java
public class FailureWatcher extends TestWatcher {

    FailureManager failureManager;

    public FailureWatcher(WebDriver driver) {
        failureManager = new FailureManager(driver); ![1](img/1.png)
    }

    @Override
    public void failed(Throwable throwable, Description description) { ![2](img/2.png)
        failureManager.takePngScreenshot(description.getDisplayName());
    }

}
```

![1](img/#co_testing_framework_specifics_CO15-1)

我们将失败分析的逻辑封装在一个单独的类中。

![2](img/#co_testing_framework_specifics_CO15-2)

我们重写了当测试失败时触发的方法。在这种情况下，我们简单地使用失败管理器实例来截取一张截图。

##### Example 8-19\. 使用 JUnit 4 分析失败的测试

```java
public class FailureManager {

    static final Logger log = getLogger(lookup().lookupClass());

    WebDriver driver;

    public FailureManager(WebDriver driver) {
        this.driver = driver;
    }

    public void takePngScreenshot(String filename) { ![1](img/1.png)
        TakesScreenshot ts = (TakesScreenshot) driver;
        File screenshot = ts.getScreenshotAs(OutputType.FILE);
        Path destination = Paths.get(filename + ".png");

        try {
            Files.move(screenshot.toPath(), destination);
        } catch (IOException e) {
            log.error("Exception moving screenshot from {} to {}", screenshot,
                    destination, e);
        }
    }

}
```

![1](img/#co_testing_framework_specifics_CO16-1)

我们将截图保存为一个 PNG 文件，并以作为参数传递的文件名命名。

## TestNG

TestNG 提供了几个默认的 *监听器*。这些监听器是捕获测试生命周期中不同事件的类。例如，`ITestResult` 监听器允许您监控测试的状态和结果。如 Example 8-20 所示，我们可以轻松地在 Selenium WebDriver 测试中使用此监听器来实现失败分析。

##### Example 8-20\. 使用 TestNG 分析失败的测试

```java
public class FailureNGTest {

    WebDriver driver;

    @BeforeMethod
    public void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterMethod
    public void teardown(ITestResult result) { ![1](img/1.png)
        if (result.getStatus() == ITestResult.FAILURE) { ![2](img/2.png)
            FailureManager failureManager = new FailureManager(driver); ![3](img/3.png)
            failureManager.takePngScreenshot(result.getName());
        }

        driver.quit();
    }

    @Test
    public void testFailure() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        fail("Forced error");
    }

}
```

![1](img/#co_testing_framework_specifics_CO17-1)

我们在测试结束时的方法中声明了一个 `ITestResult` 参数。

![2](img/#co_testing_framework_specifics_CO17-2)

我们读取测试的状态。

![3](img/#co_testing_framework_specifics_CO17-3)

在失败的情况下，我们创建一个失败管理器的实例（我们使用与 Example 8-19 描述的相同逻辑）以创建一个截图。

## JUnit 5

在 JUnit 5 中，Jupiter 扩展模型取代并改进了基于规则的 JUnit 4 测试生命周期管理。正如 Chapter 2 中介绍的，Jupiter 提供的扩展模型允许在 Jupiter 编程模型的基础上添加新功能。这样，Jupiter 扩展是实现一个或多个 *扩展点* 的 Java 类，这些接口允许在 Jupiter 编程模型中执行不同类型的操作。Table 8-4 总结了 Jupiter 提供的扩展点。

Table 8-4\. Jupiter 扩展点

| 类别 | 描述 | 扩展点(s) |
| --- | --- | --- |
| 测试生命周期回调 | 在测试生命周期中包含自定义逻辑 |

```java
BeforeAllCallback
BeforeEachCallback
BeforeTestExecutionCallback
AfterTestExecutionCallback
AfterEachCallback
AfterAllCallback
```

|

| 参数解析 | 在测试方法或构造函数中注入参数 |
| --- | --- |

```java
ParameterResolver
```

|

| 测试模板 | 使用`@TestTemplate`实现测试 |
| --- | --- |

```java
TestTemplateInvocationContextProvider
```

|

| 条件测试执行 | 根据自定义条件启用或禁用测试 |
| --- | --- |

```java
ExecutionCondition
```

|

| 异常处理 | 处理测试及其生命周期中的异常 |
| --- | --- |

```java
TestExecutionExceptionHandler
LifecycleMethodExecutionExceptionHandler
```

|

| 测试实例 | 创建和处理测试类实例 |
| --- | --- |

```java
TestInstanceFactory
TestInstancePostProcessor
TestInstancePreDestroyCallback
```

|

| 拦截调用 | 拦截对测试代码的调用（并决定这些调用是否继续） |
| --- | --- |

```java
InvocationInterceptor
```

|

实施失败分析的一个便捷扩展点是`AfterTestExecutionCallback`，因为它允许在单个测试执行后立即包含自定义逻辑。示例 8-21 提供了使用自定义注解的 Jupiter 测试（参见示例 8-22）来实现此扩展点。

##### 示例 8-21\. 使用 JUnit 5 分析失败的测试

```java
class FailureJupiterTest {

    static WebDriver driver;

    @RegisterExtension
    FailureWatcher failureWatcher = new FailureWatcher(driver); ![1](img/1.png)

    @BeforeAll
    static void setup() {
        driver = WebDriverManager.chromedriver().create();
    }

    @AfterAll
    static void teardown() {
        driver.quit();
    }

    @Test
    void testFailure() {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        fail("Forced error"); ![2](img/2.png)
    }

}
```

![1](img/#co_testing_framework_specifics_CO18-1)

我们使用此类中提供的`FailureWatcher`扩展来进行测试。我们将驱动程序实例作为参数传递。

![2](img/#co_testing_framework_specifics_CO18-2)

我们强制失败以使扩展获取浏览器截图。

##### 示例 8-22\. 使用 JUnit 5 分析失败的测试

```java
public class FailureWatcher implements AfterTestExecutionCallback { ![1](img/1.png)

    FailureManager failureManager;

    public FailureWatcher(WebDriver driver) {
        failureManager = new FailureManager(driver);
    }

    @Override
    public void afterTestExecution(ExtensionContext context) throws Exception { ![2](img/2.png)
        if (context.getExecutionException().isPresent()) { ![3](img/3.png)
            failureManager.takePngScreenshot(context.getDisplayName()); ![4](img/4.png)
        }
    }

}
```

![1](img/#co_testing_framework_specifics_CO19-1)

此扩展实现了一个单一的扩展点：`AfterTestExecution​Call⁠back`。

![2](img/#co_testing_framework_specifics_CO19-2)

此扩展点必须重写此方法，该方法在每个测试后立即执行。

![3](img/#co_testing_framework_specifics_CO19-3)

我们检查执行异常是否存在。

![4](img/#co_testing_framework_specifics_CO19-4)

如果是这样，我们使用`WebDriver`实例来截图。

## Selenium-Jupiter

Selenium-Jupiter 是一个 Jupiter 扩展，除其他功能外，还允许轻松进行浏览器截图。示例 8-23 展示了这一特性。

##### 示例 8-23\. 使用 Selenium-Jupiter 分析 JUnit 5 的失败测试

```java
class FailureSelJupTest {

    @RegisterExtension
    static SeleniumJupiter seleniumJupiter = new SeleniumJupiter();

    @BeforeAll
    static void setup() {
        seleniumJupiter.getConfig().enableScreenshotWhenFailure(); ![1](img/1.png)
    }

    @Test
    void testFailure(ChromeDriver driver) {
        driver.get("https://bonigarcia.dev/selenium-webdriver-java/");
        fail("Forced error");
    }

}
```

![1](img/#co_testing_framework_specifics_CO20-1)

Selenium-Jupiter 通过使用此配置能力，在测试失败时获取浏览器截图。

# 重试测试

如第七章中所述，端到端测试中存在测试*不稳定性*（即可靠性不足）是一个众所周知的问题。作为测试人员，有时我们需要识别*不稳定*的测试（即在相同条件下通过或失败的测试），为此，我们重试给定的测试以检查其结果是否一致。因此，我们可能需要一种机制，在测试失败时重试测试。本节解释了如何使用不同的单元测试框架执行此过程。

## JUnit 4

我们需要使用自定义的 JUnit 4 规则来重试失败的测试。示例 8-24 展示了使用这种规则的测试示例，而示例 8-25 包含了该规则的源代码。

##### 示例 8-24\. 使用 JUnit 4 重试测试

```java
public class RandomCalculatorJUnit4Test {

    static WebDriver driver;

    @Rule
    public RetryRule retryRule = new RetryRule(5); ![1](img/1.png)

    @BeforeClass
    public static void setup() {
        driver = WebDriverManager.chromedriver().create(); ![2](img/2.png)
    }

    @AfterClass
    public static void teardown() {
        driver.quit();
    }

    @Test
    public void testRandomCalculator() {
        driver.get(
         "https://bonigarcia.dev/selenium-webdriver-java/random-calculator.html"); ![3](img/3.png)
        // 1 + 3
        driver.findElement(By.xpath("//span[text()='1']")).click(); ![4](img/4.png)
        driver.findElement(By.xpath("//span[text()='+']")).click();
        driver.findElement(By.xpath("//span[text()='3']")).click();
        driver.findElement(By.xpath("//span[text()='=']")).click();

        // ... should be 4
        String result = driver.findElement(By.className("screen")).getText();
        assertThat(result).isEqualTo("4"); ![5](img/5.png)
    }

}
```

![1](img/#co_testing_framework_specifics_CO21-1)

我们将重试规则声明为测试属性。

![2](img/#co_testing_framework_specifics_CO21-2)

我们所有重复使用相同的浏览器。

![3](img/#co_testing_framework_specifics_CO21-3)

我们打开一个名为*随机计算器*的实践网页。该页面被设计为在一定百分比的时间内（默认为 50%）生成错误结果。然后，计算器在配置的次数后（默认为五次）正常工作。

![4](img/#co_testing_framework_specifics_CO21-4)

我们使用计算器 GUI 进行重要的算术操作。

![5](img/#co_testing_framework_specifics_CO21-5)

我们验证结果。前五次尝试有 50% 的概率得到错误结果。

##### 示例 8-25\. 使用 JUnit 4 规则重试失败的测试

```java
public class RetryRule implements TestRule { ![1](img/1.png)

    static final Logger log = getLogger(lookup().lookupClass());

    int maxRetries;

    public RetryRule(int maxRetries) {
        this.maxRetries = maxRetries; ![2](img/2.png)
    }

    @Override
    public Statement apply(Statement base, Description description) { ![3](img/3.png)
        return new Statement() {
            @Override
            public void evaluate() throws Throwable {
                Throwable throwable = null;
                for (int i = 0; i < maxRetries; i++) { ![4](img/4.png)
                    try {
                        base.evaluate();
                        return;
                    } catch (Throwable t) { ![5](img/5.png)
                        throwable = t;
                        log.debug("{}: run {} failed",
                                description.getDisplayName(), i + 1);
                    }
                }
                log.debug("{}: giving up after {} failures",
                        description.getDisplayName(), maxRetries);
                throw throwable; ![6](img/6.png)
            }
        };
    }
}
```

![1](img/#co_testing_framework_specifics_CO22-1)

我们为 JUnit 4 规则实现了通用接口，即`TestRule`。

![2](img/#co_testing_framework_specifics_CO22-2)

此规则在其构造函数中接受一个整数值，用于确定最大重试次数。

![3](img/#co_testing_framework_specifics_CO22-3)

我们需要重写`apply`方法，该方法允许操作测试的生命周期。

![4](img/#co_testing_framework_specifics_CO22-4)

我们在循环中重复执行测试，重复次数最多等于重试次数。

![5](img/#co_testing_framework_specifics_CO22-5)

在测试执行期间发生错误时，我们获取异常对象并重复执行测试。

![6](img/#co_testing_framework_specifics_CO22-6)

如果达到此行，表示测试已重试最大次数。

## TestNG

TestNG 提供了一个自定义功能来实现测试重试。如示例 8-26 所示，我们使用`@Test`注解的`retryAnalyzer`属性来启用此功能。示例 8-27 展示了重试分析器的实现。

##### 示例 8-26\. 使用 TestNG 重试测试

```java
@Test(retryAnalyzer = RetryAnalyzer.class)
public void testRandomCalculator() {
    // Same logic than the example before
}
```

##### 示例 8-27\. TestNG 的测试分析器

```java
public class RetryAnalyzer implements IRetryAnalyzer { ![1](img/1.png)

    static final int MAX_RETRIES = 5; ![2](img/2.png)

    int retryCount = 0;

    @Override
    public boolean retry(ITestResult result) { ![3](img/3.png)
        if (retryCount <= MAX_RETRIES) { ![4](img/4.png)
            retryCount++;
            return true;
        }
        return false;
    }
}
```

![1](img/#co_testing_framework_specifics_CO23-1)

我们需要实现一个称为`IRetryAnalyzer`的 TestNG 监听器以实现重试分析器。

![2](img/#co_testing_framework_specifics_CO23-2)

我们无法为此类参数化；因此，我们在类内声明最大重试次数（在本例中作为常量）。

![3](img/#co_testing_framework_specifics_CO23-3)

我们需要重写方法`retry`。该方法返回一个布尔值，用于确定在失败时是否重试测试。

![4](img/#co_testing_framework_specifics_CO23-4)

确定此值的逻辑是一个累加器，检查是否达到重试阈值。

## JUnit 5

我们需要使用先前解释的扩展模型（参见表 8-4）来重试失败的测试。我们可以利用现有的开源 Jupiter 扩展而不是重复造轮子。为了重试测试，正如在第二章中介绍的那样，有多种选择：[JUnit Pioneer](https://junit-pioneer.org)或[rerunner-jupiter](https://github.com/artsok/rerunner-jupiter)。示例 8-28 展示了使用后者的测试。

##### 示例 8-28\. 使用 JUnit 5 重试测试

```java
@RepeatedIfExceptionsTest(repeats = 5) ![1](img/1.png)
void testRandomCalculator() {
    // Same logic as the example before }
```

![1](img/#co_testing_framework_specifics_CO24-1)

通过简单地使用此注解装饰测试，在失败的情况下最多重复测试五次。

## Selenium-Jupiter

使用 Selenium-Jupiter 的测试也可以使用其他扩展。示例 8-29 展示了如何在 Selenium-Jupiter 测试中使用 rerunner-jupiter。

##### 示例 8-29\. 使用 JUnit 5 和 Selenium-Jupiter 重试测试

```java
@SingleSession ![1](img/1.png)
@ExtendWith(SeleniumJupiter.class)
class RandomCalculatorSelJupTest {

    @RepeatedIfExceptionsTest(repeats = 5)
    void testRandomCalculator(ChromeDriver driver) {
        // Same logic than the example before
    }

}
```

![1](img/#co_testing_framework_specifics_CO25-1)

我们重复所有可能的重试中都使用同一个浏览器。

# 并行测试执行

执行 Selenium WebDriver 测试套件所需的时间可能相当长（特别是如果测试数量很多）。这种缓慢的原因在于，常规的 Selenium WebDriver 测试每次启动一个新的浏览器，导致整体执行时间增加。解决此问题的一种可能方案是并行执行测试。有多种方法可以实现此并行化。首先，我们可以使用构建工具（如 Maven 或 Gradle）提供的内置并行执行功能。其次，我们可以利用单元测试框架（JUnit 4 或 5 以及 TestNG）提供的功能来实现。以下各小节详细解释了所有这些选项。

## Maven

Maven 提供了不同的机制来进行并行执行。首先，Maven 允许并行构建多模块项目的模块。为此，我们需要从命令行使用选项`-T`调用 Maven 命令。此选项接受两种类型的参数进行并行化：使用固定数量的线程或使用系统中可用 CPU 核心数的乘数因子。以下代码段展示了每种类型的示例：

```java
mvn test -T 4 ![1](img/1.png)
mvn test -T 1C ![2](img/2.png)
```

![1](img/#co_testing_framework_specifics_CO26-1)

它使用四个线程并行执行多模块项目（例如示例仓库）的测试。

![2](img/#co_testing_framework_specifics_CO26-2)

它使用与 CPU 核心数量相同的线程数（例如，在四核系统中使用四个线程）并行执行多模块项目的测试。

此外，用于在 Maven 中执行单元测试的插件（称为 *Surefire*）提供了两种并行运行测试的方法。第一种是在单个 JVM 进程内进行多线程操作。要启用此模式，我们需要指定不同的配置参数，例如：

`parallel`

为并行执行配置并行度级别。该参数的可能值包括`methods`（在单独的线程中执行测试方法）、`classes`（测试类）、`suites`（测试套件）、`suitesAndClasses`（测试套件和类）、`suitesAndMethods`（测试套件和方法）以及`all`（在单独的线程中执行每个测试）。

`threadCount`

定义并行性的最大线程数。

`useUnlimitedThreads`

允许无限线程。

有两种方法可以指定这些配置参数。首先，我们可以直接在 Maven 配置文件（即`pom.xml`文件）中配置它们。示例 8-30 演示了如何进行配置。此外，我们还可以在使用命令行时将这些参数作为系统属性指定，例如：

```java
mvn test -Dparallel=classesAndMethods -DthreadCount=4
```

##### 示例 8-30\. Maven Surefire 配置示例，用于并行执行

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>${maven-surefire-plugin.version}</version>
            <configuration>
                <parallel>classesAndMethods</parallel>
                <threadCount>4</threadCount>
            </configuration>
        </plugin>
    </plugins>
</build>
```

使用 Maven Surefire 实现并行的第二种方式是 *forking*，即创建多个 JVM 进程。如果需要防止线程级并发问题，这种选择很有帮助，因为不同的进程不共享内存空间，这与多线程不同。作为缺点，forking 消耗更多内存且性能较低。要启用 forking，我们需要将 `forkCount` 配置属性（再次在`pom.xml`或作为系统属性中）设置为大于一的值（即要创建的 JVM 进程数）。例如，以下命令使用四个 JVM 进程执行 Maven 项目的测试：

```java
mvn test -DforkCount=4
```

## Gradle

Gradle 也提供了几种执行测试的并行方式。首先，它允许在多模块项目中并行执行任务。有两种启用此模式的方法。首先，在配置文件`gradle.properties`中设置属性`org.gradle.parallel=true`。其次，使用命令中的选项`--parallel`，例如：

```java
gradle test --parallel
```

此外，我们可以在 Gradle 配置文件中使用配置属性 `maxParallelForks` 来指定要并行启动的最大测试进程数。默认情况下，Gradle 一次只执行一个测试类。通过为此参数设置高于一的值，我们可以更改此默认行为。除了固定值外，我们还可以指定系统中可用 CPU 核心的数量：

```java
maxParallelForks = Runtime.runtime.availableProcessors()
```

在示例存储库中，此属性通过名为`parallel`的配置文件条件性启用（见附录 C）。因此，我们可以使用以下命令行来使用此配置文件：

```java
gradle test -Pparallel
```

## JUnit 4

JUnit 通过类 `Parallel​Com⁠puter` 提供了一种基本的方式来并行执行测试。此类在其构造函数中接受两个布尔参数，以分别启用类和方法的并行测试执行。示例 8-31 展示了使用此类进行测试的示例。

##### 示例 8-31\. 使用 JUnit 4 进行并行测试执行

```java
public class ParallelJUnit4Suite {

    @Test
    public void runInParallel() {
        Class<?>[] classes = { Parallel1JUnit4Test.class,
                Parallel2JUnit4Test.class }; ![1](img/1.png)
        JUnitCore.runClasses(new ParallelComputer(true, true), classes); ![2](img/2.png)
    }

}
```

![1](img/#co_testing_framework_specifics_CO27-1)

我们指定要并行执行的测试类。

![2](img/#co_testing_framework_specifics_CO27-2)

我们为测试类和方法启用并行测试执行。

## TestNG

在 TestNG 中指定测试并行执行的常见方式是通过配置文件 `testng.xml`。在 TestNG 中启用此模式最相关的属性包括：

`parallel`

指定并行运行测试的模式。替代方案为 `methods`、`tests` 和 `classes`。

`threadcount`

设置并行运行测试的默认最大线程数。

示例 8-32 显示了用于测试并行性的 `testng.xml` 的基本配置。

##### 示例 8-32\. TestNG 的并行测试配置

```java
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="parallel-suite" parallel="classes" thread-count="2">
    <test name="parallel-tests">
        <classes>
            <class name=
              "io.github.bonigarcia.webdriver.testng.ch08.parallel.Parallel1NGTest"/>
            <class name=
              "io.github.bonigarcia.webdriver.testng.ch08.parallel.Parallel2NGTest"/>
        </classes>
    </test>
</suite>
```

我们可以在命令行中使用 Maven 或 Gradle 运行之前的并行测试套件：

```java
mvn test -Dsurefire.suiteXmlFiles=src/test/resources/testng.xml
gradle test -Psuite=src/test/resources/testng.xml
```

## JUnit 5

JUnit 5 允许以不同的方式并行执行测试。以下列表总结了此目的的最相关配置参数：

`junit.jupiter.execution.parallel.enabled`

启用测试并行性的布尔标志（默认为 `false`）。

`junit.jupiter.execution.parallel.mode.classes.default`

为了并行运行测试类。可能的值为 `same_thread` 表示单线程执行（默认），`concurrent` 表示并行执行。

`junit.jupiter.execution.parallel.mode.default`

为了并行运行测试方法。可能的值与之前的测试类相同。

有两种指定这些参数的方式。首先，在配置文件 `junit-platform.properties` 中（应该位于项目类路径中可用）。示例 8-33 显示了此文件的示例内容。其次，通过使用系统属性和命令行。以下命令（Maven/Gradle）展示了如何操作：

```java
mvn test -Djunit.jupiter.execution.parallel.enabled=true
gradle test -Djunit.jupiter.execution.parallel.enabled=true
```

##### 示例 8-33\. 使用 JUnit 5 进行并行测试执行

```java
junit.jupiter.execution.parallel.enabled = true
junit.jupiter.execution.parallel.mode.default = same_thread
junit.jupiter.execution.parallel.mode.classes.default = same_thread
```

此外，Jupiter 编程模型提供了注解 `@Execution`，用于更改测试类或方法的并行模式。该注解可以在类级别或方法级别使用，接受两个值：`ExecutionMode.CONCURRENT`（用于并行执行）和 `ExecutionMode.SAME_THREAD`（用于单线程执行）。示例 8-34 展示了示例仓库中包含的测试类的结构。假设启用了并行测试（如示例 8-33），此类将与其他允许并行化的测试一起并行执行。

##### 示例 8-34\. 使用 JUnit 5 进行并行测试执行

```java
@Execution(ExecutionMode.CONCURRENT)
class Parallel1JupiterTest {

    // Test logic

}
```

# 测试监听器

在测试过程中通常需要跟踪不同阶段的测试执行。单元测试框架因此提供了称为*测试监听器*的功能。测试监听器可以被视为通过在测试执行周期的多个阶段执行自定义操作来修改默认测试行为的实用程序。通常情况下，每个单元测试框架都为这些测试监听器提供了自己的实现。

## JUnit 4

在 JUnit 4 中，测试监听器包括在测试开始、通过、完成、失败、跳过或忽略时执行的自定义操作。实现 JUnit 4 监听器的第一步是创建一个扩展`RunListener`类的 Java 类。在这个类中，您可以重写几个方法（例如`testRunStarted`、`testIgnored`、`testFailure`等），以在测试生命周期的不同步骤中包含额外的逻辑。示例 8-35 展示了一个基本的 JUnit 4 测试监听器实现。此监听器简单地在标准输出中显示有关测试阶段的消息。

##### 示例 8-35\. 使用 JUnit 4 测试监听器

```java
public class MyTestListener extends RunListener {

    static final Logger log = getLogger(lookup().lookupClass());

    @Override
    public void testStarted(Description description) throws Exception {
        super.testStarted(description);
        log.debug("testStarted {}", description.getDisplayName());
    }

    @Override
    public void testFailure(Failure failure) throws Exception {
        super.testFailure(failure);
        log.debug("testFailure {} {}", failure.getException(),
                failure.getMessage());
    }

    // Other listeners

}
```

在 JUnit 4 中注册测试监听器的常见方法是创建一个自定义运行器，并在测试类中使用该运行器。示例 8-36 展示了注册前述监听器的自定义测试运行器。示例 8-37 展示了使用此运行器的测试框架。

##### 示例 8-36\. 使用 JUnit 4 测试监听器

```java
public class MyTestRunner extends BlockJUnit4ClassRunner { ![1](img/1.png)

    public MyTestRunner(Class<?> clazz) throws InitializationError {
        super(clazz);
    }

    @Override
    public void run(RunNotifier notifier) {
        notifier.addListener(new MyTestListener()); ![2](img/2.png)
        super.run(notifier); ![3](img/3.png)
    }
}
```

![1](img/#co_testing_framework_specifics_CO28-1)

我们扩展`Blockjunit4classrunner`，这是 JUnit 4 中的默认测试运行器。

![2](img/#co_testing_framework_specifics_CO28-2)

我们注册我们的自定义测试监听器。

![3](img/#co_testing_framework_specifics_CO28-3)

我们使用默认的测试运行程序继续调用父级。

##### 示例 8-37\. 使用 JUnit 4 测试监听器

```java
@RunWith(MyTestRunner.class) ![1](img/1.png)
public class ListenersJUnit4Test {

    // Test logic 
}
```

![1](img/#co_testing_framework_specifics_CO29-1)

我们使用 JUnit 4 注解`@RunWith`和我们的自定义运行器装饰测试类。

## TestNG

TestNG 提供了接口`ITestListener`来实现测试监听器。实现此接口的类可以重写各个 TestNG 生命周期阶段的方法，如`onTestSuccess`、`onTestFailure`或`onTestSkipped`等。示例 8-38 展示了实现此接口的示例类。在这个示例中，监听器方法在标准输出中记录一条消息。示例 8-39 展示了使用此监听器的测试。

##### 示例 8-38\. 使用 TestNG 测试监听器

```java
public class MyTestListener implements ITestListener {

    static final Logger log = getLogger(lookup().lookupClass());

    @Override
    public void onTestStart(ITestResult result) {
        ITestListener.super.onTestStart(result);
        log.debug("onTestStart {}", result.getName());
    }

    @Override
    public void onTestFailure(ITestResult result) {
        ITestListener.super.onTestFailure(result);
        log.debug("onTestFailure {}", result.getThrowable());
    }

    // Other listeners

}
```

##### 示例 8-39\. 使用 TestNG 测试监听器

```java
@Listeners(MyTestListener.class) ![1](img/1.png)
public class ListenersNGTest {

    // Test logic 
}
```

![1](img/#co_testing_framework_specifics_CO30-1)

我们使用 TestNG 注解`@Listeners`来指定此类中的所有测试使用我们的自定义测试监听器。

## JUnit 5

正如前面讨论的（见 表 8-4），Jupiter 提供了各种扩展点，用于在 JUnit 5 测试生命周期中包含自定义逻辑。除了这个扩展模型，JUnit 5 还允许实现测试监听器来跟踪几个测试执行阶段，如测试启动、跳过或完成。此功能通过 JUnit Launcher API 提供，该 API 用于发现、过滤和执行 JUnit 平台中的测试（见 图 2-4）。

要在 JUnit 5 中创建一个测试监听器，我们需要实现 `TestExecutionListener` 接口。实现这个接口的类可以重写不同的方法，以便在测试执行期间被通知发生的事件。示例 8-40 包含一个实现此接口的基本类。这类监听器通过标准的 Java 服务加载器机制在 JUnit 5 中注册。为此，我们需要在项目类路径中创建一个名为 `/META-INF/services/org.junit.platform.launcher.TestExecutionListener` 的文件，并写入要注册的测试监听器的完全限定名称（例如 `io.github.bonigarcia.webdriver.jupiter.ch08.listeners.MyTestListener` 对应 示例 8-40）。请注意，这个文件没有包含在示例存储库中，以避免侵入整个测试套件。

##### 示例 8-40\. 使用 JUnit 5 测试监听器

```java
public class MyTestListener implements TestExecutionListener {

    static final Logger log = getLogger(lookup().lookupClass());

    @Override
    public void executionStarted(TestIdentifier testIdentifier) {
        TestExecutionListener.super.executionStarted(testIdentifier);
        log.debug("Test execution started {}", testIdentifier.getDisplayName());
    }

    @Override
    public void executionSkipped(TestIdentifier testIdentifier, String reason) {
        TestExecutionListener.super.executionSkipped(testIdentifier, reason);
        log.debug("Test execution skipped: {}", reason);
    }

    @Override
    public void executionFinished(TestIdentifier testIdentifier,
            TestExecutionResult testExecutionResult) {
        TestExecutionListener.super.executionFinished(testIdentifier,
                testExecutionResult);
        log.debug("Test execution finished {}",
                testExecutionResult.getStatus());
    }

}
```

###### 注意

接口 `TestExecutionListener` 属于 JUnit 平台启动器 API；因此，要使用它，我们需要在项目中额外包含这个 API 作为依赖项。附录 C 解释了为此设置所需的 Maven 和 Gradle 配置。

# 禁用的测试

单元测试框架允许以编程方式禁用（即在测试执行中跳过）整个测试类或单个测试方法。下面的小节解释了 JUnit 4、TestNG、JUnit 5 和 Selenium-Jupiter 之间的区别。

## JUnit 4

JUnit 4 提供了注解 `@Ignore` 来禁用测试。这个注解可以用在类级别或方法级别。可选地，我们可以在注解中包含消息，以说明禁用的原因。示例 8-41 包含一个被禁用的测试。

##### 示例 8-41\. 使用 JUnit 4 禁用测试

```java
@Ignore("Optional reason for disabling")
@Test
public void testDisabled() {
    // Test logic
}
```

## TestNG

TestNG 允许以两种方式禁用测试。首先，我们可以对测试类或方法使用注解 `@Ignore`。其次，我们可以使用 `@Test` 注解的 `enabled` 属性。示例 8-42 说明了这两种方法。

##### 示例 8-42\. 使用 TestNG 禁用测试

```java
@Ignore("Optional reason for disabling")
@Test
public void testDisabled1() {
    // Test logic
}

@Test(enabled = false)
public void testDisabled2() {
    // Test logic
}
```

## JUnit 5

Jupiter 编程模型提供了各种注解，用于根据不同条件禁用测试。表 8-5 总结了这些注解，而 示例 8-43 提供了一个使用其中一些注解的基本示例。

表 8-5\. 用于禁用测试的 Jupiter 注解

| 注解 | 描述 |
| --- | --- |

|

```java
@Disabled
```

| 禁用测试类或方法 |
| --- |

|

```java
@DisabledOnJre
@EnabledOnJre
```

| 根据 Java 版本来禁用/启用测试 |
| --- |

|

```java
@DisabledOnJreRange
@EnabledOnJreRange
```

| 根据 Java 版本范围来禁用/启用测试 |
| --- |

|

```java
@DisabledOnOs
@EnabledOnOs
```

| 根据操作系统（例如，Windows、Linux、macOS 等）来禁用/启用测试 |
| --- |

|

```java
@DisabledIfSystemProperty
@DisabledIfSystemProperties
@EnabledIfSystemProperty
@EnabledIfSystemProperties
```

| 根据系统属性的值来禁用/启用测试 |
| --- |

|

```java
@DisabledIfEnvironmentVariable
@DisabledIfEnvironmentVariables
@EnabledIfEnvironmentVariable
@EnabledIfEnvironmentVariables
```

| 根据环境变量的值来禁用/启用测试 |
| --- |

|

```java
@DisabledIf
@EnabledIf
```

| 根据自定义方法的布尔返回值来禁用/启用测试 |
| --- |

##### 示例 8-43\. 使用 JUnit 5 禁用的测试

```java
@Disabled("Optional reason for disabling") ![1](img/1.png)
@Test
public void testDisabled1() {
    // Test logic }

@DisabledOnJre(JAVA_8) ![2](img/2.png)
@Test
public void testDisabled2() {
    // Test logic }

@EnabledOnOs(MAC) ![3](img/3.png)
@Test
public void testDisabled3() {
    // Test logic }
```

![1](img/#co_testing_framework_specifics_CO31-1)

总是跳过此测试。

![2](img/#co_testing_framework_specifics_CO31-2)

在使用 Java 8 时跳过此测试。

![3](img/#co_testing_framework_specifics_CO31-3)

除 macOS 外的任何操作系统均跳过此测试。

## Selenium-Jupiter

Selenium-Jupiter 提供了额外的注解，可以根据 Selenium WebDriver 测试的特定条件有条件地禁用测试。这些条件包括浏览器可用性、Docker 可用性和 URL 在线性（即使用 `GET` HTTP 方法请求 URL 时返回 200 响应代码）。示例 8-44 展示了使用这些注解的几个测试。

##### 示例 8-44\. 使用 JUnit 5 和 Selenium-Jupiter 禁用的测试

```java
@EnabledIfBrowserAvailable(SAFARI) ![1](img/1.png)
@Test
void testDisabled1(SafariDriver driver) {
    // Test logic }

@EnabledIfDockerAvailable ![2](img/2.png)
@Test
void testDisabled2(@DockerBrowser(type = CHROME) WebDriver driver) {
    // Test logic }

@EnabledIfDriverUrlOnline("http://localhost:4444/") ![3](img/3.png)
@Test
void testDisabled3(
        @DriverCapabilities("browserName=chrome") WebDriver driver) {
    // Test logic }
```

![1](img/#co_testing_framework_specifics_CO32-1)

如果系统中没有 Safari，则跳过此测试。

![2](img/#co_testing_framework_specifics_CO32-2)

如果系统中没有安装 Docker，则跳过此测试。

![3](img/#co_testing_framework_specifics_CO32-3)

如果 Selenium Server URL 不在线，则跳过此测试。如果在线，则执行该测试，并使用前述 URL 创建 `RemoteWebDriver` 的实例。在此测试中，我们使用 `@DriverCapabilities` 注解指定所需的能力（如 第六章 中所解释的）。

# 总结与展望

本章介绍了本书中使用的测试框架（即 JUnit 4、TestNG、JUnit 5 和 Selenium-Jupiter）的一些最相关的特定功能，用于开发 Selenium WebDriver 测试。首先，您学习了如何实现参数化测试。此功能对于跨浏览器测试（即在不同浏览器上进行网页测试）非常方便。然后，您学习了如何对测试进行分类，并使用这些分类来包含或排除它们以进行测试执行。您还学习了测试失败分析的机制（例如，测试失败时制作浏览器截图）、重试测试或并行执行测试。最后，您了解了如何实现测试监听器以及禁用测试的不同机制。

在下一章中，您将学习如何将 Selenium WebDriver 与不同的第三方工具集成，以实现高级端到端测试。您将了解如何从 Web 应用程序下载文件，在不使用 CDP 的情况下捕获流量（例如在 Firefox 中），测试非功能需求（如性能、安全性或可访问性），处理不同的输入数据，改进测试报告，并与现有框架（如 Spring 或 Cucumber）集成。
