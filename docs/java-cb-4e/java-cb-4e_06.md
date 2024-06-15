# 第六章。日期和时间

# 6.0 引言

开发人员在 Java 1.0 的 `Date` 类和其替代品 Java 1.1 的 `Calendar` 类的不一致性和模糊性下遭受了长达十五年的苦难。出现了几种替代 `Date` 的包，包括简单而明智的 [Date4J](http://date4j.net) 和更全面的 [Joda-Time 包](http://www.joda.org/joda-time)。Java 8 引入了一个新的、一致且经过深思熟虑的日期和时间处理包，由 Java 社区流程 JSR-310 主持，由开发人员 Stephen Colebourne 提出，基于他早期的 Joda-Time 包，但进行了几个重要的设计更改。^(1) 该包偏向于 ISO 8601 日期；例如，默认格式为 2015-10-23T10:22:45。当然，它也可以与其他日历方案一起使用。

新 API 的一个关键优势是它提供了像添加/减去日期/时间等 *有用的操作*。开发人员曾多次浪费时间重复实现这些有用的操作。有了新的 API，可以使用内置功能。尽管如此，数百万行代码基于旧的 API，因此我们将简要回顾它们，并在本章的最后一节（Recipe 6.9）考虑将新 API 与遗留代码进行接口化。

新 API 的另一个优点是几乎所有对象都是不可变的，因此是线程安全的。随着我们迅速进入大规模并行的时代，这可能带来相当大的好处。

由于没有 `set` 方法，因此 getter 方法范式并不总是有意义，API 提供了一系列新方法来替换这些方法，列在 表 6-1 中。

表 6-1。新日期/时间 API：常见方法

| Name | Description |
| --- | --- |
| `at` | 与另一个对象结合 |
| `format` | 使用提供的格式化器生成格式化字符串 |
| `from` | 工厂：将输入参数转换为目标实例 |
| `get` | 从实例中检索一个字段 |
| `is` | 检查给定对象的状态 |
| `minus` | 返回减去给定量的副本 |
| `now` | BuilderFactory：获取当前时间、日期等 |
| `of` | 工厂：通过解析输入创建新的方法 |
| `parse` | 工厂：解析单个输入字符串以生成目标实例 |
| `plus` | 返回添加了给定量的副本 |
| `to` | 将此对象转换为另一种类型 |
| `with` | 返回更改了给定字段的副本；替换 `set` 方法 |

JSR 310 API 指定了大约十几个主要类。表示时间的那些类要么是连续时间，要么是人类时间。*连续时间* 基于 Unix 时间，这是计算机时间的黎明时期的更深层真相，表示为单个单调递增的数字。Unix 中的时间值 0 代表 1970 年 1 月 1 日 UTC 的第一秒，大约是 Unix 发明的时候。每个增量单位代表一秒钟的时间。自从 Y2K 事件后，大多数 Unix 系统已经在 2038 年前通过将时间值从 32 位转换为 64 位来静静地避免了可能的 Y2038 事件。Java 也使用了这种时间基准，但使用了 64 位，并且以毫秒为单位存储其时间，因为自 1970 年以来的 64 位毫秒时间不会在未来几年内溢出（请在您的日历中保留此日期——292,278,994 年 8 月 17 日）。下面是我得到该日期的计算：

```java
        Date endOfTime = new Date(Long.MAX_VALUE);
        System.out.println("Java time overflows on " + endOfTime);
```

新 API 包含五个包，如表 6-2 所示；通常，顶层包含最常用的部分。

表 6-2\. 新的日期/时间 API：包

| 名称 | 描述 |
| --- | --- |
| `java.time` | 日期、时间、时刻和持续时间的通用类 |
| `java.time.chrono` | 非 ISO 日历系统的 API |
| `java.time.format` | 格式化类（参见 Recipe 6.2） |
| `java.time.temporal` | 使用字段、单位和调整器访问日期和时间 |
| `java.time.zone` | 支持时区及其规则 |

基础的`java.time`包包含大约十几个类，以及几个枚举和一个通用异常（在表格 6-3、6-4 和 6-5 中显示）。

表 6-3\. 新的日期/时间 API：基础

| 类 | 描述 |
| --- | --- |
| `Clock` | 可替换的工厂，用于获取当前时间 |
| `Instant` | 表示自 1970 年 1 月 1 日以来的某一时刻，以纳秒表示 |
| `Duration` | 表示一段时间，以纳秒表示 |

人类时间将时间和日期表示为我们日常生活中使用的方式。这些类在表 6-4 中列出。

表 6-4\. 新的日期/时间 API：人类时间

| 类 | 描述 |
| --- | --- |
| `Calendrical` | 连接到低级 API |
| `DateTimeFields` | 存储字段-值对的映射，这些对不需要保持一致 |
| `DayOfWeek` | 一周的某一天（例如星期二） |
| `LocalDate` | 仅表示日期（日、月、年），没有任何调整 |
| `LocalTime` | 仅表示时间（小时、分钟、秒），没有任何调整 |
| `LocalDateTime` | 上述所有内容的组合 |
| `MonthDay` | 月份和日期 |
| `OffsetTime` | 带有时区偏移（例如 -04:00）的某一天的时间，没有日期或时区 |
| `OffsetDateTime` | 带有时间偏移的日期和时间，如 –04:00，但没有时区 |
| `Period` | 描述性的时间量，例如“2 个月 3 天” |
| `ZonedDateTime` | 带有时区和偏移量的日期和时间 |
| `Year` | 一个年份 |
| `YearMonth` | 年和月份 |

几乎所有顶级类直接扩展 `java.lang.Object` 并通过各种接口保持一致性，这些接口在子包中声明。日期和时间类大多实现了 `Comparable`，这是有意义的。

表 6-5 展示了与 `ZonedDateTime`, `OffsetDateTime`, 和 `OffsetTime` 一起使用的两个特定于时区的类。

表 6-5\. 新的日期/时间 API：支持

| 类 | 描述 |
| --- | --- |
| `ZoneOffset` | 从 UTC 偏移的时间偏移量（小时，分钟，秒） |
| `ZoneId` | 定义诸如 *Canada/Eastern* 等时区及其转换规则 |

新 API 是*流畅 API*，大多数操作返回它们所操作的对象，因此您可以在不需要繁琐和烦人的临时变量的情况下链式调用多个调用：

```java
LocalTime time = LocalTime.now().minusHours(5); // the time 5 hours ago
```

在我看来，这样会产生更自然和方便的编码风格。如果您愿意，总是可以编写带有大量临时变量的代码；最后需要读这些代码的是您自己。

# 6.1 寻找今天的日期

## 问题

您希望找到今天的日期和/或时间。

## 解决方案

调用适当的构建器以获取 `LocalDate`, `LocalTime`, 或 `LocalDateTime` 对象，并调用其 `toString()` 方法。

## 讨论

这些类没有公共构造函数，因此您需要调用其中一个工厂方法来获取实例。它们都提供一个 `now()` 方法，其功能如其名称所示。`CurrentDateTime` 演示程序展示了所有三个类的简单用法：

```java
public class CurrentDateTime {
    public static void main(String[] args) {
        LocalDate dNow = LocalDate.now();
        System.out.println(dNow);
        LocalTime tNow = LocalTime.now();
        System.out.println(tNow);
        LocalDateTime now = LocalDateTime.now();
        System.out.println(now);
    }
}
```

运行它会产生以下输出：

```java
2013-10-28
22:23:55.641
2013-10-28T22:23:55.642
```

格式化并不引人注目，但足够使用。我们将在食谱 6.2 中处理更复杂的格式化。

尽管这样可以工作，在大型应用程序中，建议将一个 `Clock` 实例传递给所有的 `now()` 方法。`Clock` 是一个工厂对象，用于在内部查找当前时间。在测试中，通常希望使用已知的日期或时间进行比较以获取已知的输出。`Clock` 类使这变得容易。示例 6-1 使用了一个 `Clock` 并允许通过调用 setter 来替换默认的 `Clock`。或者，您可以使用像 CDI 或 Spring 这样的依赖注入框架来提供正确版本的 `Clock` 类。

##### 示例 6-1\. main/src/main/java/datetime/TestableDateTime

```java
package datetime;

import java.time.Clock;
import java.time.LocalDateTime;

/**
 * TestableDateTime allows test code to plug in a Fixed clock
 */
public class TestableDateTime {
    private static Clock clock = Clock.systemDefaultZone();
    public static void main(String[] args) {
        System.out.println("It is now " + LocalDateTime.now(clock));
    }
    public static void setClock(Clock clock) {
        TestableDateTime.clock = clock;
    }
}
```

在正常操作中，会获取当前日期和时间。在测试中，您可以使用 `setClock()` 方法和从静态方法 `Clock.fixed(Instant fixedInstant, ZoneId zone)` 获得的 `Clock` 实例，传入您的测试代码期望的时间。固定时钟不会滴答，所以在将时钟设置为固定时和调用测试之间的毫秒数不必担心。

# 6.2 格式化日期和时间

## 问题

您希望为日期和时间对象提供更好的格式化。

## 解决方案

使用 `java.time.format.DateTimeFormatter`。

## 讨论

`DateTimeFormatter` 类提供了大量可能的格式化样式。如果您不想使用提供的约 20 种预定义格式之一，可以使用 `DateTimeFormatter.ofPattern(String pattern)` 来定义自己的格式。`pattern` 字符串可以包含任何字符，但除了明显的 *Y*, *M*, *D*, *h*, *m*, 和 *s* 外，几乎每个字母都有其特定含义。此外，引号字符和方括号也有定义，井号 (*#*) 和花括号则保留供将来使用。

如日期格式化语言通常所示，模式中字母的重复次数提示了其意图的详细程度。因此，例如，“MMM” 表示 “Jan”，而 “MMMM” 表示 “January”。

表格 6-6 是从 JSR-310 的 javadoc 改编的格式字符完整列表的尝试。

表格 6-6\. DateFormatter 格式字符

| 符号 | 含义 | 显示方式 | 示例 |
| --- | --- | --- | --- |
| `G` | 纪元 | 文本 | AD; Anno Domini |
| `y` | 年份 | 年份 | 2004; 04 |
| `u` | 年份 | 年份 | 见备注。 |
| `D` | 年份中的日期 | 数字 | 189 |
| `M/L` | 月份 | 数字/文本 | 7; 07; Jul; July; J |
| `d` | 月份中的日期 | 数字 | 10 |
| `Q/q` | 年份的季度 | 数字/文本 | 3; 03; Q3, 3rd quarter |
| `Y` | 基于周的年份 | 年份 | 1996; 96 |
| `w` | 基于年的周数 | 数字 | 27 |
| `W` | 月中的周数 | 数字 | 4 |
| `e/c` | 本地化的星期几 | 数字/文本 | 2; 02; Tue; Tuesday; T |
| `E` | 星期几 | 文本 | Tue; Tuesday; T |
| `F` | 月中的周数 | 数字 | 3 |
| `a` | 上午/下午 | 文本 | PM |
| `h` | 上午/下午的小时数 (1-12) | 数字 | 12 |
| `K` | 上午/下午的小时数 (0-11) | 数字 | 0 |
| `k` | 上午/下午的小时数 (1-24) | 数字 | 0 |
| `H` | 一天中的小时数 (0-23) | 数字 | 0 |
| `m` | 小时中的分钟数 | 数字 | 30 |
| `s` | 分钟中的秒数 | 数字 | 55 |
| `S` | 秒的分数 | 分数 | 978 |
| `A` | 一天中的毫秒数 | 数字 | 1234 |
| `n` | 秒中的纳秒数 | 数字 | 987654321 |
| `N` | 一天中的纳秒数 | 数字 | 1234000000 |
| `V` | 时区 ID | 时区 ID | America/Los_Angeles; Z; –08:30 |
| `z` | 时区名称 | 时区名称 | Pacific Standard Time; PST |
| `X` | 零时区偏移量 *Z* | 偏移量-X | Z; –08; –0830; –08:30; –083015; –08:30:15; |
| `x` | 时区偏移 | 偏移-x | +0000; –08; –0830; –08:30; –083015; –08:30:15; |
| `Z` | 时区偏移 | 偏移-Z | +0000; –0800; –08:00; |
| `O` | 本地化时区偏移 | 偏移-O | GMT+8; GMT+08:00; UTC–08:00; |
| `p` | 下一个填充 | 填充修饰符 | 1 |

###### 注意

*y* 和 *u* 对于公元年份是相同的；但是，对于公元前 3 年，*y* 模式返回 3，而 *u* 模式返回 -2（也称为预测年）。

示例 6-2 包含一些在字符串和日期之间进行双向转换的示例。

##### 示例 6-2\. main/src/main/java/datetime/DateFormatter.java（示例日期格式化和解析）

```java
public class DateFormatter {
    public static void main(String[] args) {

        // Format a date ISO8601-like but with slashes instead of dashes
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy/LL/dd");
        System.out.println(df.format(LocalDate.now()));

        // Parse a String to a date using the same formatter
        System.out.println(LocalDate.parse("2014/04/01", df));

        // Format a Date and Time without timezone information
        DateTimeFormatter nTZ =
            DateTimeFormatter.ofPattern("d MMMM, yyyy h:mm a");
        System.out.println(ZonedDateTime.now().format(nTZ));
    }
}
```

# 6.3 在日期/时间、YMDHMS 和时代秒之间进行转换

## 问题

你需要在日期/时间、YMDHMS、时代秒或其他一些数字值之间进行转换。

## 解决方案

使用适当的日期/时间工厂或检索方法。

## 讨论

时代是现代操作系统开始的时间。Unix 时间和一些 Windows 时间版本从时代开始不可避免地计算秒数。1970 年，肯·汤普森和丹尼斯·里奇提出这种格式时，秒数看起来像是一个不错的度量单位，32 位的秒数似乎近乎无限。然而，在将时代存储为 32 位整数的操作系统上，时间正在流逝。大多数操作系统的旧版本将其存储为 32 位有符号整数，不幸的是，这将在 2038 年溢出。

Java 刚发布时，它有一个叫做`System.currentTimeMillis()`的方法，以毫秒精度表示时代秒。新的 Java API 使用的是纳秒时代，仍然处于相同的时间基准上，可以通过调用`System.nanoTime()`获得。

这些与时代相关的任何数字都可以转换为本地日期/时间，也可以从中获取。还可以使用其他数字，如整数年份、月份和天数。通常情况下，有一些工厂方法可以在请求更改时创建新对象。下面是一个演示这些转换的程序：

*main/src/main/java/datetime/DateConversions.java*

```java
        // Convert a number of seconds since the epoch to a local date/time
        Instant epochSec = Instant.ofEpochSecond(1000000000L);
        ZoneId zId = ZoneId.systemDefault();
        ZonedDateTime then = ZonedDateTime.ofInstant(epochSec, zId);
        System.out.println("The epoch was a billion seconds old on " + then);

        // Convert a date/time to epoch seconds
        long epochSecond = ZonedDateTime.now().toInstant().getEpochSecond();
        System.out.println("Current epoch seconds = " + epochSecond);

        LocalDateTime now = LocalDateTime.now();
        ZonedDateTime there = now.atZone(ZoneId.of("Canada/Pacific"));
        System.out.printf("When it's %s here, it's %s in Vancouver%n",
            now, there);
```

# 6.4 将字符串解析为日期

## 问题

你需要将用户输入转换为`java.time`对象。

## 解决方案

使用`parse()`方法。

## 讨论

许多日期/时间类都有一个`parse()`工厂方法，试图将字符串解析为该类的对象。例如，`LocalDate.parse(String)`返回一个给定日期的`LocalDate`对象：

```java
public class DateParse {
    public static void main(String[] args) {

        String armisticeDate = "1914-11-11";
        LocalDate aLD = LocalDate.parse(armisticeDate);
        System.out.println("Date: " + aLD);

        String armisticeDateTime = "1914-11-11T11:11";
        LocalDateTime aLDT = LocalDateTime.parse(armisticeDateTime);
        System.out.println("Date/Time: " + aLDT);
```

正如你现在可能预期的那样，默认格式是 ISO8601 日期格式。但是，我们经常需要处理其他格式的日期。为此，`DateTimeFormatter`允许您指定特定的模式。例如，“dd MMM uuuu”表示月份中的日期（两位数字）、月份名称的三个字母（Jan、Feb、Mar，…）和四位数字年份：

```java
        DateTimeFormatter df = DateTimeFormatter.ofPattern("dd MMM uuuu");
        String anotherDate = "27 Jan 2011";
        LocalDate random = LocalDate.parse(anotherDate, df);
        System.out.println(anotherDate + " parses as " + random);
```

`DateTimeFormatter`对象是双向的；它既可以解析输入，也可以格式化输出。我们可以将这一行添加到`DateParse`示例中：

```java
System.out.println(aLD + " formats as " + df.format(aLD));
```

当我们运行程序时，输出如下所示：

```java
Date: 1914-11-11
Date/Time: 1914-11-11T11:11
27 Jan 2011 parses as 2011-01-27
1914-11-11 formats as 11 Nov 1914
```

`DateTimeFormatter`也是本地化的（参见配方 3.12），可以在调用`ofPattern()`后调用`withLocale()`进行配置。

# 6.5 两个日期之间的差异

## 问题

你需要计算两个日期之间的差异。

## 解决方案

使用静态方法`Period.between()`来找到两个`LocalDate`之间的差异。

## 讨论

给定两个`LocalDate`对象，你可以使用静态方法`Period.between()`简单地找到它们之间的差异作为一个`Period`。你可以`toString()`这个`Period`，或者如果默认格式不够好，可以自己格式化结果：

```java
import java.time.LocalDate;
import java.time.Period;

/**
 * Tutorial/Example of LocalDate date difference subtraction
 */
public class DateDiff {

    public static void main(String[] args) {
        /** The date at the end of the last century */
        LocalDate endof20thCentury = LocalDate.of(2000, 12, 31);
        LocalDate now = LocalDate.now();
        if (now.getYear() > 2100) {
            System.out.println("The 21st century is over!");
            return;
        }

        Period diff = Period.between(endof20thCentury, now);

        System.out.printf("The 21st century (up to %s) is %s old%n", now, diff);
        System.out.printf(
                "The 21st century is %d years, %d months and %d days old",
                diff.getYears(), diff.getMonths(), diff.getDays());
    }
}
```

我在 2013 年 10 月底写下了这个配方；公元 20 世纪末在 2000 年末结束，所以值应约为 12¹⁰/[12]年，即：

```java
$ java datetime.DateDiff
The 21st century (up to 2013-10-28) is P12Y9M28D old
The 21st century is 12 years, 9 months and 28 days old
```

由于 API 的规律性，你可以在`LocalTime`或`LocalDateTime`中使用相同的技术。

`ChronoUnit`也存在，包含许多范围值，比如`DAYS`、`HOURS`、`MINUTES`等（实际范围从`NANOS`到`MILLENIA`、`ERAS`甚至`FOREVER`）。如果你想要在某个单位获取差异信息：

```java
jshell> import java.time.temporal.*;

jshell> ChronoUnit.DAYS.between(LocalDate.now(), LocalDate.parse("2022-02-22"))
$6 ==> 786

jshell> ChronoUnit.DECADES.between(LocalDate.of(1970,01,01),
  LocalDate.of(2020,01,01));
$7 ==> 5
```

Unix 已经进入了它的第五个十年！

## 参见

讨论配方 6.2 讨论了格式化日期/时间值的高级方法。

# 6.6 增加或减少日期

## 问题

你需要对日期加上或减去一个固定的周期。

## 解决方案

使用如`Local⁠Date.plus​(Period.ofDays(N));`这样的表达式可以创建过去或未来的日期。

## 讨论

`java.time`提供了`Period`类来表示一段时间，比如几天、几小时和几分钟。`LocalDate`等类提供了`plus()`和`minus()`方法来增加或减少`Period`或其他与时间相关的对象。`Period`还提供了像`ofDays()`这样的工厂方法。以下代码计算从现在起 700 天后的日期：

```java
import java.time.LocalDate;
import java.time.Period;

/** DateAdd -- compute the difference between two dates
 * (e.g., today and 700 days from now).
 */
public class DateAdd {
    public static void main(String[] av) {
        /** Today's date */
        LocalDate now =  LocalDate.now();

        Period p = Period.ofDays(700);
        LocalDate then = now.plus(p);

        System.out.printf("Seven hundred days from %s is %s%n", now, then);
    }
}
```

运行这个程序会报告当前的日期和时间，以及从现在起 700 天后的日期和时间：

```java
Seven hundred days from 2013-11-09 is 2015-10-10
```

# 6.7 处理重复事件

## 问题

你需要处理重复发生的日期，例如每个月的第三个星期三。

## 解决方案

使用`TemporalAdjusters`类。

## 讨论

`TemporalAdjuster`接口和`TemporalAdjusters`工厂类提供了大部分处理重复事件所需的功能。有许多有趣而强大的调整器可用，如[表 6-7](https://wiki.example.org/javacook-dates-recurring-temporalAdjusters)中所示，当然你也可以自己开发。

表 6-7\. 新日期/时间 API：TemporalAdjusters 工厂方法

| 方法签名 |
| --- |
| public static TemporalAdjuster firstDayOfMonth(); |
| public static TemporalAdjuster lastDayOfMonth(); |
| public static TemporalAdjuster firstDayOfNextMonth(); |
| public static TemporalAdjuster firstDayOfYear(); |
| public static TemporalAdjuster lastDayOfYear(); |
| public static TemporalAdjuster firstDayOfNextYear(); |
| public static TemporalAdjuster firstInMonth(java.time.DayOfWeek); |
| public static TemporalAdjuster lastInMonth(java.time.DayOfWeek); |
| public static TemporalAdjuster dayOfWeekInMonth(int, java.time.DayOfWeek); |
| public static TemporalAdjuster next(java.time.DayOfWeek); |
| public static TemporalAdjuster nextOrSame(java.time.DayOfWeek); |
| public static TemporalAdjuster previous(java.time.DayOfWeek); |
| public static TemporalAdjuster previousOrSame(java.time.DayOfWeek); |
| public static TemporalAdjuster ofDateAdjuster( java.util.function.UnaryOperator<java.time.LocalDate>); |

大多数名称直接告诉您它们的功能。在阅读有关如 `UnaryOperator` 的函数接口的第九章 Chapter 9 后，最后一个将变得清晰明了。

这些与日期/时间对象的 `with()` 方法一起使用。例如，GTABUG 小组 ([*http://gtabug.org*](http://gtabug.org)) 每月第三个星期三举行会议。在 `darwinsys-api` 库中有一个 `RecurringEventDatePicker` 类；其核心部分始于方法 `getMeetingDateInMonth(LocalDate dateContainingMonth)`，在我们的情况下选择给定月份的第三个星期三（在构造函数中设置了 *`dayOfWeek`* 和 *`weekOfMonth`*）。我们获取月份 (`dateContainingMonth`)，使用 `firstInMonth()` 工厂方法调整为该月的第一个星期三，然后添加周数以获取正确星期的星期三：

```java
// Variant versions from older version of RecurringDatePicker.java
// First version, not for production use!
private LocalDate getMeetingForMonth(LocalDate dateContainingMonth) {
    return
        dateContainingMonth.with(TemporalAdjusters.firstInMonth(dayOfWeek))
            .plusWeeks(Math.max(0, weekOfMonth - 1));
}
```

第二个版本简化了它以更好地使用现有的 API：

```java
private LocalDate getMeetingForMonth(LocalDate dateContainingMonth) {
    return dateWithMonth.with(
        TemporalAdjusters.dayOfWeekInMonth(weekOfMonth,dayOfWeek)
}
```

由于此版本仅包含一个语句，且仅使用两次，我们将其内联到`getNextMeeting(int howManyMonthsAway)`方法中，该方法返回给定月份中正确日期的`LocalDate`。其唯一的复杂性在于，对于当前月份，会议可能在今天的日期之前或之后，因此我们进行相应调整：

```java
public LocalDate getEventLocalDate(int meetingsAway) {
    LocalDate thisMeeting = now.with(
        TemporalAdjusters.dayOfWeekInMonth(weekOfMonth,dayOfWeek));
    // Has the meeting already happened this month?
    if (thisMeeting.isBefore(now)) {
        // start from next month
        meetingsAway++;
    }
    if (meetingsAway > 0) {
        thisMeeting = thisMeeting.plusMonths(meetingsAway).
            with(TemporalAdjusters.dayOfWeekInMonth(weekOfMonth,dayOfWeek));
    }
    return thisMeeting;
}
```

然后在 JavaServer Page (JSP) web 视图中调用此方法（略有简化；真实代码具有 JavaScript 中的 Add To Calendar API 的复杂性）。如果您尚未使用 JSP，请直接输出纯 HTML 代码，`<% %>` 标签的内容将被执行，`<%= %>` 标签的内容将被评估并打印到 HTML 页面中，例如：

```java
Upcoming Meetings:
<ul>
    <%
    RecurringEventDatePicker mp =
      new RecurringEventDatePicker(3, DayOfWeek.WEDNESDAY);
    DateTimeFormatter dfm = DateTimeFormatter.ofPattern("MMMM dd, yyyy");
    for (int i = 0; i <= 2; i++) {
        LocalDateTime dt = mp.getEventLocalDateTime(i);
    %>
    <li>
        <%= dt.format(dfm) %>
    </li>
    <%
    }
    %>
</ul>
```

当在 2015 年的六月或七月访问此网站时，您可能会看到类似以下内容：

```java
Upcoming Meetings:

* July 15, 2015
* August 19, 2015
* September 16, 2015
```

# 6.8 计算涉及时区的日期

## 问题

想象一个问题：“您的孩子正在从多伦多飞往伦敦的跨大西洋航班，从实际起飞时间（YYZ）起需 5 小时 10 分钟。您的岳父母需要一个小时到达 LHR 并找到停车位。您应该在什么时间打电话告诉他们去机场？”

## 解决方案

解决方案需要考虑时区差异。可以使用 `ZonedDateTime` 类及其 `plus()` 和 `minus()` 方法来解决。

## 讨论

基本步骤如示例 Example 6-3 中所示。

##### 示例 6-3\. main/src/main/java/datetime/FlightArrivalTimeCalc.java

```java
public class FlightArrivalTimeCalc {

    static Duration driveTime = Duration.ofHours(1);

    public static void main(String[] args) {
        LocalDateTime when = null;
        if (args.length == 0) {
            when = LocalDateTime.now();                                        ![1](img/1.png)
        } else {
            String time = args[0];
            LocalTime localTime = LocalTime.parse(time);
            when = LocalDateTime.of(LocalDate.now(), localTime);               ![1](img/1.png)
        }
        calulateArrivalTime(when);
    }

    public static ZonedDateTime calulateArrivalTime(LocalDateTime takeOffTime) {
        ZoneId torontoZone = ZoneId.of("America/Toronto"),
                londonZone = ZoneId.of("Europe/London");
        ZonedDateTime takeOffTimeZoned =
            ZonedDateTime.of(takeOffTime, torontoZone);                        ![2](img/2.png)
        Duration flightTime =
            Duration.ofHours(5).plus(10, ChronoUnit.MINUTES);                  ![3](img/3.png)
        ZonedDateTime arrivalTimeUnZoned = takeOffTimeZoned.plus(flightTime);  ![4](img/4.png)
        ZonedDateTime arrivalTimeZoned =
            arrivalTimeUnZoned.toInstant().atZone(londonZone);                 ![5](img/5.png)
        ZonedDateTime phoneTimeHere = arrivalTimeUnZoned.minus(driveTime);     ![6](img/6.png)

        System.out.println("Flight departure time " + takeOffTimeZoned);
        System.out.println("Flight expected length: " + flightTime);
        System.out.println(
            "Flight arrives there at " + arrivalTimeZoned + " London time.");
        System.out.println("You should phone at " + phoneTimeHere + " Toronto time");
        return arrivalTimeZoned;
    }
}
```

![1](img/#co_dates_and_times_CO1-1)

获取出发时间作为 `LocalDateTime`（如果没有传入参数到 `main()`，则默认为 `now()`，假设我们在飞机起飞时运行应用程序）。

![2](img/#co_dates_and_times_CO1-3)

将出发时间转换为 `ZonedDateTime`。

![3](img/#co_dates_and_times_CO1-4)

将航班时间转换为 `Duration`。

![4](img/#co_dates_and_times_CO1-5)

通过将出发时间加上飞行持续时间来获取到达时间。

![5](img/#co_dates_and_times_CO1-6)

使用 `atZone()` 将到达时间转换为伦敦时间。

![6](img/#co_dates_and_times_CO1-7)

由于全家需要一个小时才能到达机场，从到达时间减去这段时间。这会得出你应该打电话的时间。

# 6.9 与旧版日期和日历类的接口

## 问题

您需要处理旧的 `Date` 和 `Calendar` 类。

## 解决方案

假设您的代码使用原始的 `java.util.Date` 和 `java.util.Calendar`，您可以根据需要使用转换方法转换值。

## 讨论

新 API 中的所有类和接口都被选择为避免与传统 API 冲突。因此，在相同的代码中导入这两个包可能是可能的，并且在一段时间内将是常见的。

为了保持新的 API 的整洁，大部分必要的转换例程被添加到*旧 API*中。Table 6-8 总结了这些转换例程；请注意，如果显示的是以大写类名调用的静态方法，则这些方法是静态的，否则它们是实例方法。

Table 6-8\. 传统日期/时间互操作性

| 传统类 | 转换为传统 | 转换为现代 |
| --- | --- | --- |
| `java.util.Date` | `date.from(Instant)` | `Date.toInstant()` |
| `java.util.Calendar` | `calendar.toInstant()` | - |
| `java.util.GregorianCalendar` | `GregorianCalendar.from(ZonedDateTime)` | `calendar.toZonedDateTime()` |
| `java.util.TimeZone` | - | `timeZone.toZoneId()` |
| `java.time.DateTimeFormatter` | - | `dateTimeFormatter.toFormat()` |

Example 6-4 展示了其中一些 API 的运行情况。

##### Example 6-4\. main/src/main/java/datetime/LegacyDates.java

```java
public class LegacyDates {
    public static void main(String[] args) {

        // There and back again, via Date
        Date legacyDate = new Date();
        System.out.println(legacyDate);

        LocalDateTime newDate =
            LocalDateTime.ofInstant(legacyDate.toInstant(),
            ZoneId.systemDefault());
        System.out.println(newDate);

        Date backAgain =
            Date.from(newDate.atZone(ZoneId.systemDefault()).toInstant());
        System.out.println("Converted back as " + backAgain);

        // And via Calendar
        Calendar c = Calendar.getInstance();
        System.out.println(c);
        LocalDateTime newCal =
            LocalDateTime.ofInstant(c.toInstant(),
            ZoneId.systemDefault());
        System.out.println(newCal);
    }
}
```

当然，您不必使用这些旧的转换器；您可以自由地编写自己的转换器。如果您希望追求这种选择，*javasrc* 仓库中的 *LegacyDatesDIY.java* 文件探讨了这个选项。

鉴于在 Java 8 之前编写的大量代码，直到 Java 的终结，传统的 `Date` 和 `Calendar` 可能会存在。

新的日期/时间 API 有许多我们尚未探索的功能。事实上，这几乎足以编写一本关于该主题的小书。同时，您可以在 [Oracle](https://docs.oracle.com/en/java/javase/13/docs/api/java.base/java/time/package-summary.html) 学习 API 的详细信息。

^(1) 对于那些对历史奥秘感兴趣的人来说，这些差异在他的[博客](http://blog.joda.org/2009/11/why-jsr-310-isn-joda-time_4941.html)上有详细记录。
