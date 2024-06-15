# 第九章\. 使用 Optionals 处理 null

作为 Java 开发者，你很可能遇到过大量的`NullPointerExceptions`，甚至更多。许多人称`null`引用为*亿美元错误*。事实上，`null`本身的发明者最初就是这样称呼它的：

> 我称之为我的亿美元错误。
> 
> 这是在 1965 年发明的`null`引用。当时，我正在设计第一个综合型对象导向语言(ALGOL W)的引用类型系统。我的目标是确保所有引用的使用都是绝对安全的，由编译器自动执行检查。但我无法抵挡诱惑，只是因为它太容易实现，就加入了一个`null`引用。
> 
> 这导致了无数的错误、漏洞和系统崩溃，在过去的四十年里，这可能造成了数十亿美元的痛苦和损失。
> 
> 查尔斯·安东尼·理查德·霍尔爵士，QCon 伦敦 2009

尽管如何处理这个“错误”还没有绝对的共识，但许多编程语言都有处理`null`引用的正确和惯用方法，通常直接集成到语言本身中。

本章将向您展示 Java 如何处理`null`引用，以及如何使用`Optional<T>`类型及其功能 API 在代码中改进它，并学习何时以及何时不使用 Optionals。

# 关于 null 引用的问题

Java 对于值的缺失的处理取决于类型。所有原始类型都有默认值，例如，数值类型的零等价物和`boolean`类型的`false`。非原始类型，如类、接口和数组，如果未分配，则使用`null`作为它们的默认值，这意味着变量不引用任何对象。

###### 注意

引用类型的概念可能看起来与 C/C++指针相似，但 Java 引用是 JVM 内的一个专门类型，称为`reference`。JVM 严格控制它们以确保类型安全和内存访问的安全性。

一个`null`引用不只是“空”，它是一个*特殊状态*，因为`null`是一个广义类型，可以用于任何对象引用，无论实际类型如何。如果尝试访问这样的`null`引用，JVM 将抛出`NullPointerException`，如果不适当处理，当前线程将崩溃。通常通过防御性编程方法来缓解这个问题，在运行时*随处*需要`null`检查，如示例 9-1 中所示。

##### 示例 9-1\. 可能存在空指针的地雷区

```java
record User(long id, String firstname, String lastname) {

  String fullname() {
    return String.format("%s %s", ![1](img/1.png)
                         firstname(),
                         lastname());
  }

  String initials() {
    return String.format("%s%s",
                         firstname().substring(0, 1), ![2](img/2.png)
                         lastname().substring(0, 1)); ![2](img/2.png)
  }
}

var user = new User(42L, "Ben", null);

var fullname = user.fullname();
// => Ben null ![1](img/1.png)

var initials = user.initials();
// => NullPointerException ![2](img/2.png)
```

![1](img/#co_handling_null_with_optionals_CO1-1)

`String.format`接受`null`值，只要它不是格式字符串之后的唯一值¹。它会翻译为字符串`null`，无论所选的格式说明符如何，甚至对于数值类型也是如此。

![2](img/#co_handling_null_with_optionals_CO1-2)

在方法调用中使用 `null` 作为参数可能不会导致当前线程崩溃。但是，在 `null` 引用上调用方法肯定会崩溃。

前面的例子突显了处理 `null` 中的两个主要问题。

首先，`null` 引用是变量、参数和返回值的有效值。这并不意味着 `null` 是每个情况下的预期、正确或甚至可接受的值，并且可能在处理过程中未被正确处理。

例如，在前面的例子中，对 `user` 调用 `getFullname` 函数时，使用 `null` 引用作为 `lastname` 是可以正常工作的，但输出“Ben null”很可能不是预期的结果。因此，即使您的代码和数据结构可以表面上处理 `null` 值，您仍然可能需要检查它们以确保正确的结果。

`null` 引用的第二个问题是它们的一个主要特性：类型模糊性。它们可以表示任何类型，而不实际成为该特定类型。这种独特的属性是必要的，因此可以在整个代码中使用单个关键字表示“缺少值”的泛化概念，而不需要为不同的对象类型使用不同的类型或关键字。尽管 `null` 引用可以像它表示的类型一样使用，但它仍然*不是*该类型本身，正如在示例 9-2 中所见。

##### 示例 9-2\. null 类型模糊性

```java
// "TYPE-LESS" NULL AS AN ARGUMENT

methodAcceptingString(null); ![1](img/1.png)

// ACCESSING A "TYPED" NULL

String name = null;

var lowerCaseName = name.toLowerCase(); ![2](img/2.png)
// => NullPointerException

// TEST TYPE OF NULL

var notString = name instanceof String; ![3](img/3.png)
// => false

var stillNotString = ((String) name) instanceof String; ![3](img/3.png)
// => false
```

![1](img/#co_handling_null_with_optionals_CO2-1)

`null` 可以表示任何对象类型，因此对于任何非原始类型的参数，它都是一个有效的值。

![2](img/#co_handling_null_with_optionals_CO2-2)

引用 `null` 的变量就像该类型的任何其他变量一样。但任何对它的调用都将导致 `NullPointerException`。

![3](img/#co_handling_null_with_optionals_CO2-3)

使用 `instanceof` 测试变量将始终评估为 `false`，而不管类型如何。即使它被显式地强制转换为所需的类型，`instanceof` 运算符也会测试基础值本身。因此，它针对无类型值 `null` 进行测试。

这些是处理 `null` 的最明显的痛点。不要担心，有方法可以减轻这种痛苦。

# 如何处理 Java 中的空值（在使用 Optional 之前）

在 Java 中处理 `null` 是每个开发人员工作中必不可少的重要部分，尽管它可能很麻烦。遇到意外且未处理的 `NullPointerException` 是许多问题的根源，必须相应地处理。

其他语言，如[Swift](https://www.swift.org)，提供了专用运算符和习语，如安全导航²或 `null` 合并运算符³，以便更轻松地处理 `null`。然而，Java 并没有提供这样的内置工具来处理 `null` 引用。

在使用 Optional 之前，有三种不同的方法来处理 `null` 引用：

+   最佳实践

+   工具辅助的 `null` 检查

+   Optional 等专门类型

正如您稍后将看到的那样，处理`null`引用不应仅依赖于 Optional。它们是前述技术的重要补充，通过在 JDK 中提供标准化和便捷的专用类型。然而，它们并不是如何在整个代码中管理`null`的最终想法，了解所有可用技术将是您技能工具包中的宝贵补充。

## 处理`null`的最佳实践

如果语言不提供集成的`null`处理，您必须依靠*最佳实践*和*非正式规则*来使您的代码免于`null`。这就是为什么许多公司、团队和项目开发自己的编码风格或适应现有的风格以提供编写一致和更安全代码的指南，不仅限于`null`。通过遵循这些自我规定的实践和规则，他们能够始终编写更可预测和不易出错的代码。

您不必开发或适应全面的样式指南来定义您的 Java 代码的每个方面。相反，遵循这四条规则是处理`null`引用的良好起点：

### 不要将变量初始化为 null

变量应始终具有非`null`值。如果值取决于决策块（如`if-else`语句），您应考虑将其重构为方法，或者如果它是一个简单的决策，则使用三元运算符。

```java
// DON'T

String value = null;

if (condition) {
  value = "Condition is true";
} else {
  value = "Fallback if false";
}

// DO

String asTernary = condition ? "Condition is true"
                             : "Fallback if false";

String asRefactored = refactoredMethod(condition);
```

其额外好处是如果稍后不再重新分配变量，则使变量实际上成为`final`，因此您可以将它们用作 lambda 表达式中的局部变量。

### 不要传递、接受或返回`null`

变量不应为`null`，因此任何参数和返回值也应避免为`null`。通过方法或构造函数的重载可以避免非必需参数为`null`的情况：

```java
public record User(long id, String firstname, String lastname) {

  // DO: Additional constructor with default values to avoid null values
  public User(long id) {
    this(id, "n/a", "n/a");
  }

  // ...
}
```

如果由于相同的参数类型而导致方法签名冲突，您始终可以改用更明确名称的`static`方法。

提供了针对可选值的特定方法和构造函数后，如果适当的话，原始方法中就不应接受`null`。最简单的方法是使用`java.util.Objects`上可用的`static` `requireNonNull`方法：

```java
public record User(long id, String firstname, String lastname) {

  // DO: Validate arguments against null
  public User {
    Objects.requireNonNull(firstname);
    Objects.requireNonNull(lastname);
  }

  // ...
}
```

`requireNonNull`调用会为您执行`null`检查，并在适当时抛出`NullPointerException`。自 Java 14 以来，任何`NullPointerException`都包含变量的名称，感谢[JEP 358](https://openjdk.org/jeps/358)。如果要包含特定消息或针对较早的 Java 版本进行定位，可以将`String`作为调用的第二个参数添加。

### 检查一切不在您控制之外的事情

即使遵循自己的规则，也不能依赖他人也这样做。始终假设使用非熟悉代码（特别是如果文档中未明确说明）可能为`null`，并且需要进行检查。

### `null`作为实现细节是可以接受的

避免`null`对于代码的`public`接口至关重要，但作为实现细节仍然是明智的。在内部，一个方法可以根据需要使用`null`，只要不将其返回给调用者即可。

### 何时遵循规则，何时不遵循规则

这些规则旨在尽可能减少`null`的一般使用，特别是在代码交集处，如 API 表面，因为更少的暴露会导致更少的必需`null`检查和可能的`NullPointerExceptions`。但这并不意味着你应该完全避免`null`。例如，在隔离的上下文中，如局部变量或非`public`的 API 中，使用`null`并不那么问题，甚至可能简化您的代码，只要谨慎使用即可。

你不能期望每个人都遵循与你相同的规则或者一样细心，因此你需要在代码中保持防御性，尤其是在你控制范围之外的情况下。这更加理由让你始终坚持最佳实践，并鼓励其他人也这样做。它们将提高你的整体代码质量，不论`null`如何。但这不是银弹，需要团队的纪律才能获得最大的好处。手动处理`null`并添加一些`null`检查优于因为假设某些东西“永远”不可能为`null`而导致意外的`NullPointerException`。即使是 JIT 编译器⁴也会执行“`null`检查消除”，从优化的汇编代码中删除许多显式的`null`检查，这要归功于其在运行时的更多知识。

## 工具辅助的空值检查

最佳实践和非正式规则方法的逻辑扩展是使用第三方工具自动强制执行它们。对于 Java 中的`null`引用，一个既定的最佳实践是使用注解将变量、参数和方法返回类型标记为`@Nullable`或`@NonNull`。

在此类注解之前，唯一可以记录空值的地方是 JavaDoc。通过这些注解，静态代码分析工具可以在编译时发现可能的`null`问题。更好的是，将这些注解添加到您的代码中，可以更明确地表达方法签名和类型定义的意图，如在示例 9-3 中所见。

##### 示例 9-3\. 带注解的空值处理

```java
interface Example {

  @NonNull List<@Nullable String> getListOfNullableStrings(); ![1](img/1.png)

  @Nullable List<@NonNull String> getNullableListOfNonNullStrings(); ![2](img/2.png)

  void doWork(@Nullable String identifier); ![3](img/3.png)
}
```

![1](img/#co_handling_null_with_optionals_CO3-1)

返回一个可能为`null`的`List`，其中包含非`null`的`String`对象。

![2](img/#co_handling_null_with_optionals_CO3-2)

返回可能为`null`的`List`，其中包含非`null`的`String`对象。

![3](img/#co_handling_null_with_optionals_CO3-3)

方法参数`identifier`允许为`null`。

JDK 不包括这些注解，对应的 [JSR 305](https://jcp.org/en/jsr/detail?id=305) 自 2012 年以来一直处于“休眠”状态。尽管如此，它仍然是事实上的社区标准，并被许多库、框架和 IDE 广泛采用。几个库⁵提供了缺失的注解，大多数工具支持多种变体。

###### 警告

尽管 `@NonNull` 和 `@Nullable` 的行为在表面上似乎显而易见，但实际实现可能因工具不同而有所不同，特别是在边缘情况下⁶。

工具辅助方法的一般问题在于对工具本身的依赖。如果它过于侵入式，你可能最终会得到无法运行的代码，特别是如果工具在“幕后”生成代码。然而，在涉及 `null` 相关注解的情况下，你不必过于担心。你的代码仍然可以在没有工具解释注解的情况下运行，并且你的变量和方法签名仍然能够清晰地传达它们的要求给任何使用它们的人，即使没有强制执行。

## 像 Optional 这样的专门类型

工具辅助方法给你编译时的 `null` 检查，而专门类型则在运行时提供更安全的 `null` 处理。在 Java 引入自己的 `Optional` 类型之前，不同的库填补了这个缺失功能，比如自 2011 年起由 [Google Guava 框架](https://github.com/google/guava/wiki/Release10) 提供的基本的 `Optional` 类型。

尽管 JDK 现在提供了一个集成的解决方案，Guava 在可预见的未来不打算弃用这个类⁷。然而，他们温和建议你在可能的情况下优先选择新的、标准的 Java `Optional<T>`。

# Optional 来拯救

Java 8 新的 `Optional<T>` 不仅仅是一个专门处理 `null` 的类型，它还是一个功能类似管道，从 JDK 中所有可用的功能补充中受益。

## 什么是 Optional？

想象 `Optional<T>` 类型最简单的方法是把它看作一个可能包含 `null` 的值的盒子。你可以使用这个盒子来替代传递可能为 `null` 的引用，如在 图 9-1 中所见。

![可变 vs 可选<T>](img/afaj_0901.png)

###### 图 9-1\. 可变 vs 可选<T>

这个盒子提供了一个安全的包装器围绕它的内部值。但是 Optional 不仅仅包装一个值。从这个盒子开始，你可以构建复杂的调用链，这些调用链依赖于值的存在或缺失。它们可以管理可能值的整个生命周期，直到盒子被解封，包括在这样一个调用链中没有值时的回退。

然而，使用包装器的缺点在于如果想要使用其内部值，必须实际查看和获取箱子。与流类似，额外的包装器还会在方法调用及其额外的栈帧方面产生无法避免的开销。另一方面，该箱提供了额外的功能，用于处理可能为`null`值的常见工作流的更简洁和直接的代码。

例如，让我们看一下通过标识符加载内容的工作流程。图 9-2 中的数字对应于示例 9-5 中即将出现的代码。

![加载内容中](img/afaj_0902.png)

###### 图 9-2\. 加载内容的工作流程

该工作流程简化了，并且未处理所有边缘情况，但它是将多步工作流转换为可选调用链的简单示例。在示例 9-4 中，您可以先看到未使用可选的工作流程实现。

##### 示例 9-4\. 使用非可选加载内容

```java
public Content loadFromDB(String contentId) {
  // ...
}

public Content get(String contentId) {

  if (contentId == null) {
    return null;
  }

  if (contentId.isBlank()) {
    return null;
  }

  var cacheKey = contentId.toLowerCase();

  var content = this.cache.get(cacheKey);
  if (content == null) {
    content = loadFromDB(contentId);
  }

  if (content == null) {
    return null;
  }

  if (!content.isPublished()) {
    return null;
  }

  return content;
}
```

例如是夸张的，但仍大多反映了防御性`null`处理的典型方法。

这里有三个明确的`null`检查，再加上关于当前值和两个临时变量的两个决策。尽管代码量不多，但总体流程由于其中许多`if`块和早期返回而不易理解。

让我们将代码转换为单个可选调用链，如示例 9-5 所示。别担心！接下来的章节将详细解释不同类型的操作。

##### 示例 9-5\. 使用可选调用链加载内容

```java
public Optional<Content> loadFromDB(String contentId) {
  // ...
}

public Optional<Content> get(String contentId) {

  return Optional.ofNullable(contentId) ![1](img/1.png)
                 .filter(Predicate.not(String::isBlank)) ![2](img/2.png)
                 .map(String::toLowerCase) ![3](img/3.png)
                 .map(this.cache::get); ![4](img/4.png)
                 .or(() -> loadFromDB(contentId)) ![5](img/5.png)
                 .filter(Content::isPublished); ![6](img/6.png)
}
```

![1](img/#co_handling_null_with_optionals_CO4-1)

第一个可能的`null`检查是使用`ofNullable`创建方法完成的。

![2](img/#co_handling_null_with_optionals_CO4-2)

下一个`if`块被`filter`操作替换。

![3](img/#co_handling_null_with_optionals_CO4-3)

而不是使用临时变量，`map`操作将值转换为匹配下一个调用的形式。

![4](img/#co_handling_null_with_optionals_CO4-4)

该内容也可以通过`map`操作来检索。

![5](img/#co_handling_null_with_optionals_CO4-5)

如果管道中不存在值，则从数据库加载内容。此调用将返回另一个可选项，以便调用链可以继续。

![6](img/#co_handling_null_with_optionals_CO4-6)

确保只有发布的内容可用。

可选调用链将整体代码压缩到每行一个操作，使整体流程易于理解。它完美地突显了使用可选调用链和“传统”方式检查`null`之间的差异。

让我们看看创建和使用可选流水线的不同步骤。

## 构建可选流水线

截至 Java 17，`Optional<T>` 提供了三个 `static` 和 15 个属于四个组的实例方法，表示 Optional 管道的不同部分：

+   创建一个新的 `Optional<T>` 实例

+   检查值或对值的存在或不存在做出反应

+   过滤和转换值

+   获取值或制定备用计划

这些操作可以构建一个流畅的管道，类似于 Streams。与 Streams 不同的是，它们 *不* 被惰性连接，直到向管道添加了类似 *terminal* 的操作，正如我在 “作为函数式数据流的流” 中讨论的那样。每个操作一旦添加到流畅的调用中，就会立即解析。Optional 之所以看起来懒散，是因为它们可能返回一个空的 Optional 或一个备用值，并跳过转换或过滤步骤。但是，这并不会使调用链本身变得懒散。然而，如果遇到 `null` 值，无论操作数量如何，执行的工作都是尽可能少的。

您可以将 Optional 调用链想象成两条火车轨道，如 图 9-3 所示。

![Optional 火车轨道](img/afaj_0903.png)

###### 图 9-3\. Optional 火车轨道

在这个类比中，我们有两条火车轨道：Optional 调用链轨道，通向返回具有内部值的 `Optional<T>`，和“空快车轨道”，通向空的 `Optional<T>`。火车始终在 `Optional<T>` 调用火车轨道上启动。当它遇到轨道切换（Optional 操作）时，它会寻找一个 `null` 值，这种情况下，火车将切换到空快车轨道。一旦进入快车轨道，就没有返回到 Optional 调用链轨道的机会，至少在 Java 9 之前是这样的，您将在 “获取（备用）值” 中看到。

从技术上讲，它仍然会在切换到空快车轨道后在 Optional 调用链上调用每个方法，但它只会验证参数并继续前进。如果火车在到达路线终点之前没有遇到 `null` 值，它将返回一个非空的 `Optional<T>`。如果它在路线上的任何时候遇到 `null` 值，它将返回一个空的 `Optional<T>`。

为了让火车动起来，让我们创建一些 Optionals。

### 创建一个 Optional

在 `Optional<T>` 类型上没有`public`构造函数可用。相反，它为您提供了三个 `static` 工厂方法来创建新实例。使用哪一个取决于您的用例和对内部值的先验知识：

如果值可能是 `null`，则使用 `Optional.ofNullable(T value)`

如果您知道一个值可能是 `null`，或者不在乎它可能为空，可以使用方法 `Optional.ofNullable(…​)` 创建一个可能具有内部 `null` 值的新实例。这是创建 `Optional<T>` 的最简单、最可靠的形式。

```java
String hasValue = "Optionals are awesome!";
Optional<String> maybeValue = Optional.ofNullable(hasValue);

String nullRef = null;
Optional<String> emptyOptional = Optional.ofNullable(nullRef);
```

如果值必须为非`null`，则使用 `Optional.of(T value)`

即使 Optionals 是处理 `null` 并防止 `NullPointerException` 的好方法，但如果你确保有一个值怎么办？例如，你已经处理了代码中的所有边缘情况，返回了空的 Optionals，并且现在你肯定有一个值。`Optional.of(…​)` 方法确保该值非`null`，否则会抛出 `NullPointerException`。这样，异常表明了代码中的真实问题。也许你忽略了一个边缘情况，或者某个外部方法调用现在返回 `null`。在这种情况下使用 `Optional.of(…​)` 可以使你的代码更加具有未来可扩展性，对行为不期而遇的变化更加鲁棒。

```java
var value = "Optionals are awesome!";
Optional<String> mustHaveValue = Optional.of(value);

value = null;
Optional<String> emptyOptional = Optional.of(value);
// => throws NullPointerException
```

如果没有值，则为 `Optional.empty()`。

如果你已经知道根本没有值，可以使用静态方法 `Optional.empty()`。调用 `Optional.ofNullable(null)` 是不必要的，因为在调用 `empty()` 本身之前会进行不必要的 `null` 检查。

```java
Optional<String> noValue = Optional.empty();
```

###### 警告

JDK 文档明确提到，由 `static` `Optional.empty` 方法返回的值不保证是单例对象。因此，你不应该使用 `==` （双等号）比较空的 Optionals，而是应该使用 `equals(Object obj)` 或比较 `isEmpty` 方法的结果。

使用 `Optional.ofNullable(T value)` 可能是最容忍 `null` 的创建方法，但你应该努力使用最适合表示你用例和上下文知识的方法。随着时间的推移，代码可能会被重构或重写，最好让你的代码为突然丢失的必需值抛出 `NullPointerException`，即使 API 本身正在使用 Optionals。

### 检查和响应值

Optionals 用于包装一个值并表示其存在或不存在。它们作为 Java 类型实现，并且因此在运行时级别具有不可避免的对象创建开销。为了补偿这一点，检查值应尽可能简单直接。

有四种方法可用于检查和响应值或其缺失。它们以“`is`”为前缀用于检查和以“`if`”为前缀用于响应的高阶函数：

+   `boolean isPresent()`

+   `boolean isEmpty()`（Java 11+）

仅仅检查值有其用途，但当你使用“`is`”方法时，检查、检索和使用值需要三个单独的步骤。

这就是为什么高阶“`if`”方法直接消耗一个值：

+   `void ifPresent(Consumer<? super T> action)`

+   `void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`

两种方法只有在存在值时才执行给定的 `action`。第二种方法在没有值时运行 `emptyAction`。不允许使用 `null` 作为动作，并会抛出 `NullPointerException`。没有 `ifEmpty…​` 的等效方法可用。

让我们看看如何在 示例 9-6 中使用这些方法。

##### 示例 9-6\. 检查 Optional 值的示例

```java
Optional<String> maybeValue = ...;

// VERBOSE VERSION

if (maybeValue.isPresent()) {
  var value = maybeValue.orElseThrow();
  System.out.println(value);
} else {
  System.out.println("No value found!");
}

// CONCISE VERSION

maybeValue.ifPresentOrElse(System.out::println,
                           () -> System.out.println("No value found!"));
```

由于缺少返回类型，两个`ifPresent`方法仅执行副作用代码。尽管在功能性方法中通常更可取，但 Optionals 处于接受功能性代码和完全适合命令式代码之间。

### 过滤和映射

安全处理可能的`null`值已经从任何开发者身上卸下了相当大的负担，但 Optionals 不仅仅是用来检查值的存在或缺失。

类似于流，你可以使用中间操作构建一个管道。这里有三个操作用于过滤和映射 Optionals：

+   `Optional<T> filter(Predicate<? super T> predicate)`

+   `<U> Optional<U> map(Function<? super T, ? extends U> mapper)`

+   `<U> Optional<U> flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)`

`filter`操作在存在值并且符合给定条件的情况下返回`this`。如果不存在值或者条件不匹配，则返回一个空的 Optional。

`map`操作使用提供的映射函数转换现有值，返回一个包含映射值的新的可空 Optional。如果不存在值，则操作返回一个空的`Optional<U>`。

如果映射函数返回的是`Optional<U>`而不是类型为`U`的具体值，则使用`flatMap`。在这种情况下如果使用`map`，返回值将是`Optional<Optional<U>>`。这就是为什么`flatMap`直接返回映射值而不是将其再次包装为另一个 Optional 的原因。

示例 9-7 展示了一个 Optional 调用链及其在假设的权限容器及其子类型中的非 Optional 等效形式。代码调用均附有对应的操作，但其描述适用于 Optional 版本。

##### 示例 9-7。寻找活跃管理员的中间操作

```java
public record Permissions(List<String> permissions, Group group) {
  public boolean isEmpty() {
    return permissions.isEmpty();
  }
}

public record Group(Optional<User> admin) {
  // NO BODY
}

public record User(boolean isActive) {
    // NO BODY
}

Permissions permissions = ...;

boolean isActiveAdmin =
  Optional.ofNullable(permissions) ![1](img/1.png)
          .filter(Predicate.not(Permissions::isEmpty)) ![2](img/2.png)
          .map(Permissions::group) ![3](img/3.png)
          .flatMap(Group::admin) ![4](img/4.png)
          .map(User::isActive) ![5](img/5.png)
          .orElse(Boolean.FALSE); ![6](img/6.png)
```

![1](img/#co_handling_null_with_optionals_CO5-1)

初始的`null`检查通过创建一个`Optional<Permissions>`来处理。

![2](img/#co_handling_null_with_optionals_CO5-2)

过滤非空权限。借助`static`的`Predicate.not`方法，lambda 表达式`permissions → !permissions.isEmpty()`被替换为一个更易读的包装方法引用。

![3](img/#co_handling_null_with_optionals_CO5-3)

获取权限对象的组。如果`Permissions::group`返回`null`，Optional 调用链将会在需要时跳过其取值操作。实际上，空的 Optional 会在流畅的调用中传递。

![4](img/#co_handling_null_with_optionals_CO5-4)

组可能没有管理员。这就是为什么它返回一个`Optional<User>`。如果简单地使用`map(Group::admin)`，在下一步中会得到一个`Optional<Optional<User>>`。而借助于`flatMap(Group::admin)`，不会创建不必要的嵌套 Optional。

![5](img/#co_handling_null_with_optionals_CO5-5)

使用 `User` 对象，可以过滤掉非活动用户。

![6](img/#co_handling_null_with_optionals_CO5-6)

如果调用链的任何方法返回空的 Optional，例如，组是 `null`，则最后一个操作返回回退值 `Boolean.FALSE`。下一节将解释不同类型的值检索操作。

解决底层问题的每个步骤都清晰、分离且直接连接。任何验证和决策，如 `null` 或空检查，都包含在基于方法引用的专用操作中。解决问题的意图和流程清晰可见，易于理解。

如果没有 Optional，做同样的事情会导致嵌套的代码混乱，如 示例 9-8 所示。

##### 示例 9-8\. 在没有 Optional 的情况下查找活动管理员

```java
boolean isActiveAdmin = false;

if (permissions != null && !permissions.isEmpty()) {

  if (permissions.group() != null) {
    var group = permissions.group();
    var maybeAdmin = group.admin();

    if (maybeAdmin.isPresent()) {
      var admin = maybeAdmin.orElseThrow();
      isActiveAdmin = admin.isActive();
    }
  }
}
```

两个版本之间的差异非常明显。

非 Optional 版本无法委托任何条件或检查，并依赖显式的 `if` 语句。这会创建深度嵌套的流程结构，增加代码的 *Cyclomatic Complexity*。很难理解代码块的整体意图，并且不如使用 Optional 调用链那样简洁。

###### 注意

*Cyclomatic Complexity⁸* 是用于确定代码复杂性的度量标准。它基于代码中的分支路径数或决策。总体思想是，直接的、非嵌套的语句和表达式更容易跟踪，比深度嵌套的决策分支，如嵌套的 `if` 语句，更不容易出错。

### 获取（回退）值

Optional 可能为可能的 `null` 值提供了一个安全的包装器，但是在某些情况下，你可能需要一个实际的值。有多种方法可以检索 Optional 的内部值，从“蛮力”到提供回退值。

第一个方法不关心任何安全检查：

+   `T get()`

强制解包 Optional，并且如果没有值，会抛出 `NoSuchElementException` 异常，因此请确保在此之前检查值是否存在。

接下来的两种方法在没有值时提供一个回退值：

+   `T orElse(T other)`

+   `T orElseGet(Supplier<? extends T> supplier)`

基于 `Supplier` 的变体允许延迟获取回退值，如果创建它需要大量资源，则非常有用。

有两种方法可以抛出异常：

+   `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)`

+   `T orElseThrow()`（Java 10+）

尽管 Optional 的主要优点之一是防止 `NullPointerException`，但有时如果没有值，你仍然需要一个特定的域异常。通过 `orElseThrow` 操作，你可以对缺失值的处理和要抛出的异常进行精细控制。第二个方法 `orElseThrow` 被添加为语义上正确和首选的 `get` 操作的替代方法。尽管调用不那么简洁，但它更符合整体命名方案，并表明可能会抛出异常。

Java 9 为提供另一个 `Optional<T>` 或 `Stream<T>` 作为回退添加了两个额外的方法。这些方法允许比以前更复杂的调用链：

第一个方法，`Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)`，在没有值时懒惰地返回另一个 Optional。这样，即使在调用 `or` 之前没有值，你也可以继续进行 Optional 调用链。回到“火车轨道”的比喻，`or` 操作是一种通过在 Optional 调用链轨道上创建一个新的起点来提供从空快速轨道返回的轨道切换的方法。

另一个方法，`Stream<T> stream()`，返回一个包含值作为其唯一元素的流，或者如果没有值，则返回一个空流。通常在中间流操作 `flatMap` 中使用，作为方法引用。Optional `stream` 操作在与我在 第七章 讨论的 Stream API 的互操作性中发挥更广泛的作用。

# 选项和流

如前几章所述，流是过滤和转换元素以获得所需结果的管道。Optional 作为可能的 `null` 引用的函数包装器正好适用，但在作为元素使用时必须遵循流管道的规则，并将其状态传递给管道。

## 选项作为流元素

使用流，通过使用过滤操作排除元素以进行进一步处理。基本上，Optional 本身表示一种过滤操作，尽管不直接与流期望元素的行为兼容。

如果一个流元素被 `filter` 操作排除，它将不会继续遍历流。这可以通过使用 `Optional::isPresent` 作为 `filter` 操作的参数来实现。然而，在内部值的情况下，结果流 `Stream<Optional<User>>` 并不是你想要的。

要恢复“正常”的 Stream 语义，需要将 Stream 从 `Stream<Optional<User>>` 映射到 `Stream<User>`，如 示例 9-9 所示。

##### 示例 9-9\. 选项作为流元素

```java
List<Permissions> permissions = ...;

List<User> activeUsers =
  permissions.stream()
             .filter(Predicate.not(Permissions::isEmpty))
             .map(Permissions::group)
             .map(Group::admin) ![1](img/1.png)
             .filter(Optional::isPresent) ![2](img/2.png)
             .map(Optional::orElseThrow) ![2](img/2.png)
             .filter(User::isActive)
             .toList();
```

![1](img/#co_handling_null_with_optionals_CO6-1)

`Group::admin` 方法引用返回一个 `Optional<User>`。此时，Stream 变为 `Stream<Optional<User>>`。

![2](img/#co_handling_null_with_optionals_CO6-2)

Stream 管道需要多个操作来检查值并安全地从其 `Optional` 中解包它。

在 Streams 中，对 `Optional<T>` 进行过滤和映射是 Optionals 的标准用例，以至于 Java 9 添加了 `stream` 方法到 `Optional<T>` 类型中。如果存在，它返回包含内部值的 `Stream<T>` 作为其唯一元素，否则返回空的 `Stream<T>`。这使得通过使用 Stream 的 `flatMap` 操作而不是专门的 `filter` 和 `map` 操作来结合 Optionals 和 Streams 的能力成为最简洁的方法，如在 Example 9-10 中所见。

##### Example 9-10\. 可选项作为流元素与 flatMap

```java
List<Permissions> permissions = ...;

List<User> activeUsers =
  permissions.stream()
             .filter(Predicate.not(Permissions::isEmpty))
             .map(Permissions::group)
             .map(Group::admin)
             .flatMap(Optional::stream)
             .filter(User::isActive)
             .toList();
```

单一的 `flatMap` 调用取代了之前的 `filter` 和 `map` 操作。即使你只节省了一个方法调用 —— 一个 `flatMap` 替代 `filter` 加 `map` 操作 —— 结果代码更易于理解，并更好地说明了期望的工作流程。`flatMap` 操作传达了理解 Stream 管道所需的所有必要信息，而不需要通过需要额外步骤来增加任何复杂性。处理 Optionals 是必要的，应尽可能简洁地完成，以便整体的 Stream 管道尽可能易于理解和直接。

没有理由在设计 API 时不使用 Optionals，只是为了避免在 Streams 中使用 `flatMap` 操作。如果 `Group::getAdmin` 返回 `null`，你仍然必须在另一个 `filter` 操作中添加 `null-check`。用 `filter` 操作替换 `flatMap` 操作并没有任何好处，除非 `admin` 调用现在需要在其签名中明确处理 `null`，即使不再显而易见。

## 终端 Stream 操作

在 Streams 中使用 Optionals 不仅限于中间操作。流 API 的五个终端操作返回 `Optional<T>`，以提供其返回值的改进表示。它们都试图找到一个元素或减少 Stream。在空 Stream 的情况下，这些操作需要一个合理的缺席值的表示。Optionals 典型地展示了这一概念，因此使用它们而不是返回 `null` 是合乎逻辑的选择。

### 寻找元素

在 Stream API 中，前缀 "`find`" 表示，“找到”一个基于其存在的元素。根据流是否并行，有两种 "`find`" 操作可用，具有不同的语义：

`Optional<T> findFirst()`

返回流的第一个元素的 Optional，如果流为空则返回空的 Optional。并行和串行流之间没有区别。如果流缺乏相遇顺序，则可能返回任何元素。

`Optional<T> findAny()`

返回流的任何元素的 Optional，如果流为空则返回空的 Optional。返回的元素是非确定性的，以最大化并行流的性能。在大多数情况下返回第一个元素，但无法保证此行为！因此，对于一致返回元素，请改用`findFirst`。

"`find`"操作仅适用于存在的概念，因此您需要相应地对流元素进行过滤。如果您只想知道特定元素是否存在，并且不需要元素本身，则可以使用相应的"`match`"方法之一：

+   `boolean anyMatch(Predicate<? super T> predicate)`

+   `boolean noneMatch(Predicate<? super T> predicate)`

这些终端操作包括过滤操作，并避免创建不必要的`Optional<T>`实例。

### 减少到单一值

将流通过组合或累积其元素减少到新数据结构是流的主要目的之一。并且与`find`操作一样，减少操作符必须处理空流。

这就是为什么流有三个终端的`reduce`操作，其中一个返回一个 Optional：`Optional<T> reduce(BinaryOperator<T> accumulator)`

使用提供的`accumulator`运算符对流的元素进行规约。返回值是规约的结果，如果流为空，则返回一个空的 Optional。

详见示例 9-11 以获取来自官方文档的等效伪代码示例⁹。

##### Example 9-11\. Pseudo-code equivalent to the `reduce` operation

```java
Optional<T> pseudoReduce(BinaryOperator<T> accumulator) {
  boolean foundAny = false;
  T result = null;

  for (T element : elements]) {
    if (!foundAny) {
      foundAny = true;
      result = element;
    } else {
      result = accumulator.apply(result, element);
    }
  }

  return foundAny ? Optional.of(result)
                  : Optional.empty();
}
```

另外两个`reduce`方法需要一个初始值来与流元素结合，以便返回一个具体的值，而不是一个 Optional。详见“减少元素”以获取更详细的解释和使用示例。

除了通用的`reduce`方法，还有两种常见的减少用例作为方法可用：

+   `Optional<T> min​(Comparator<? super T> comparator)`

+   `Optional<T> max(Comparator<? super T> comparator)`

这些方法基于提供的比较器返回“最小”或“最大”元素，如果流为空则返回空的 Optional。

`Optional<T>`是唯一适合由`min`/`max`返回的类型。无论如何，您都必须检查操作是否有结果。添加带有回退值作为参数的额外`min`/`max`方法将混乱流接口。由于返回的`Optional`，您可以轻松检查是否存在结果，或者改用回退值或异常。

# 原始可选项

您可能会问自己为什么需要原始的 Optional，因为原始变量永远不会是`null`。如果未初始化，任何原始变量都具有与其相应类型的零值。

尽管在技术上是正确的，但 Optional 并不只是防止值为`null`。它们还表示一种“无存在值”的实际状态，而这是原始类型所缺乏的。

在许多情况下，原始类型的默认值是足够的，比如表示网络端口：零是一个无效的端口号，因此你必须处理它。但如果零是一个有效的值，表达它的实际缺失变得更加困难。

直接使用`Optional<T>`类型与原始类型是行不通的，因为原始类型不能是泛型类型。然而，就像使用流一样，处理可选的原始值有两种方式：自动装箱或专用类型。

“原始类型”突显了使用对象包装类及其引入的开销问题。另一方面，自动装箱也不是免费的。

通常的原始类型都有专门的 Optional 变体：

+   `java.util.OptionalInt`

+   `java.util.OptionalLong`

+   `java.util.OptionalDouble`

它们的语义几乎与它们的通用对应物相同，但它们既不继承自`Optional<T>`也没有共同的接口。它们的特性也不相同，如缺少`filter`、`map`或`flatMap`等多个操作。

原始的 Optional 类型可以消除不必要的自动装箱，这可以提高性能，但缺乏`Optional<T>`提供的全部功能。此外，与我在“原始流”中讨论的原始流变体不同，没有一种简单的方法可以在原始 Optional 变体和其对应的`Optional<T>`等效型之间进行转换。

尽管创建自己的包装类型以改进 Optional 值的处理（特别是对于原始类型）很容易，但在大多数情况下我不建议这样做。对于内部或`private`实现，可以使用任何所需的包装类型。但是代码的`public`接口应始终坚持使用最受期待和可用的类型。通常情况下，这意味着 JDK 中已经包含的内容。

# 注意事项

Optional 可以通过提供一个多功能的“盒子”来极大地改善 JDK 中的`null`处理，用于保存可能的`null`值，并提供（部分）功能性 API 来构建处理该值存在与否的管道。尽管优势显而易见，但它也伴随着一些显著的缺点，你需要了解这些缺点，以正确地使用它们，并避免任何意外的惊喜。

## Optional 是普通类型

`Optional<T>`及其原始变体最明显的缺点之一是它们是普通类型。没有更深入地融入 Java 语法，比如新的 lambda 表达式语法，它们会遇到与 JDK 中任何其他类型相同的`null`引用问题。

因此，你仍然必须遵循最佳实践和非正式规则，以免抵消使用 Optionals 的好处。如果你设计一个 API 并决定使用 Optionals 作为返回类型，在任何情况下都*不能*返回 `null`！返回 Optional 是一个明确的信号，表明使用 API 的任何人至少会收到一个“盒子”，*可能*包含一个值，而不是可能的 `null` 值。如果没有可能的值，始终使用空的 Optional 或者等效的基本类型。

尽管如此，这个基本的设计要求必须通过约定来执行。没有额外的工具，如 [SonarSource](https://www.sonarsource.com/)¹⁰，编译器不会帮助你。

## 敏感于身份的方法

尽管 Optionals *是* 普通类型，但是与你预期的可能不同的是，一些敏感于身份的方法。这包括引用相等性运算符 `==`（双等号），使用 `hashCode` 方法，或者使用实例进行线程同步。

###### 注意

对象标识告诉你两个不同对象是否共享相同的内存地址，因此是同一个对象。这可以通过引用相等性运算符 `==`（双等号）来测试。两个对象的相等性，通过它们的 `equals` 方法来测试，意味着它们包含相同的状态。

两个相同的对象也是相等的，但反过来不一定成立。仅因为两个对象包含相同的状态并不意味着它们也共享相同的内存地址。

行为上的差异在于 Optional 是*基于值*的类型，意味着其内部值是其主要关注点。像 `equals`、`hashCode` 和 `toString` 这样的方法仅基于内部值，并忽略实际对象标识。因此，你应该将 Optional 实例视为可互换的，并且不适合用于像同步并发代码这样的身份相关操作，正如官方文档所述¹¹。

## 性能开销

使用 Optionals 时另一个需要考虑的点是性能影响，特别是在它们作为返回类型之外的主要设计目标时。

对于简单的 `null` 检查来说，Optionals 容易被误用，并提供一个回退值，如果内部值不存在：

```java
// DON'T DO THIS

String value = Optional.ofNullable(maybeNull)
                       .orElse(fallbackValue);

// DON'T DO THIS

if (Optional.ofNullable(maybeNull).isPresent()) {
  // ...
}
```

这样简单的 Optional 流水线需要一个新的 `Optional` 实例，每个方法调用都会创建一个新的堆栈帧，这使得 JVM 不能像简单的 `null` 检查那样轻松优化你的代码。创建 Optional 除了检查存在性或提供回退外，并没有太多意义。

使用三元运算符或直接的 `null` 检查应该是你的首选解决方案：

```java
// DO THIS INSTEAD

String value = maybeNull != null ? maybeNull
                                 : fallbackValue;

// DO THIS INSTEAD

if (maybeNull != null) {
  // ...
}
```

使用 Optional 而不是三元运算符可能看起来更美观，避免重复使用 `maybeNull`。减少实例创建和方法调用通常是可取的。

如果你仍然想要一个更美观的替代三元运算符的方法，Java 9 在 `java.util.Objects` 上引入了两个 `static` 帮助方法，用于检查`null`并提供替代值：

+   `T requireNonNullElse(T obj, T defaultObj)`

+   `T requireNonNullElseGet(T obj, Supplier<? extends T> supplier)`

回退值，或者在第二种方法中，`Supplier` 的结果，也必须是非`null`的。

为了避免由于意外的`NullPointerException`导致的崩溃，节省几个 CPU 循环毫无意义。就像流一样，性能和更安全、更简单的代码之间存在权衡。你需要根据你的需求找到这些之间的平衡。

## 集合的特殊考虑

`null` 是值缺失的技术表示。Optionals 为你提供了一种工具，可以安全地表示这种缺失，并使用实际对象进行进一步的转换、过滤等操作。然而，基于集合的类型已经可以表示其内部值的缺失。

集合类型已经是一个处理值的“盒子”，所以将其包装在`Optional<T>`中将创建另一个必须处理的层。空集合已经表示内部值的缺失，因此使用空集合作为`null`的替代品消除了可能的`NullPointerException` *以及* 使用 Optional 时的额外层次。

当然，你仍然必须处理集合本身的缺失，意味着一个`null`引用。如果可能的话，你不应该对集合使用`null`，无论是作为参数还是返回值。设计你的代码始终使用空集合而不是`null`将具有与 Optional 相同的效果。如果你仍然需要区分`null`和空集合，或者相关的代码不在你的控制范围内或无法更改，那么使用`null`检查可能仍然优于引入另一层来处理。

## Optionals 和序列化

`Optional<T>` 类型和原始变体不实现 `java.io.Serializable`，因此它们不适用于可序列化类型中的私有字段。这个决定是设计小组故意做出的，因为 Optionals 应该提供可选返回值的可能性，而不是成为空引用的通用解决方案。使`Optional<T>`可序列化将鼓励超出其预期设计目标的用例。

为了仍然在你的对象中获得 Optionals 的好处并保持可序列化性，你可以将它们用于你的`public` API，但在实现细节中使用非 Optional 字段，就像示例 9-12 中所示的那样

##### 例子 9-12\. 在可序列化类型中使用 Optionals

```java
public class User implements Serializable {

  private UUID id;
  private String username;
  private LocalDateTime lastLogin;

  // ... usual getter/setter for id and username

  public Optional<LocalDateTime> getLastLogin() {
    return Optional.ofNullable(this.lastLogin);
  }

  public void setLastLogin(LocalDateTime lastLogin) {
    this.lastLogin = lastLogin;
  }
}
```

通过仅依赖于获取器对`lastLogin`的 Optional，在类型保持可序列化的同时仍提供 Optional API。

# 对空引用的最终思考

尽管被称为“价值十亿美元的错误”，`null` 本身并不邪恶。`null` 的发明者，查尔斯·安东尼·理查德·霍尔爵士（Sir Charles Antony Richard Hoare）认为，编程语言设计者应对使用其语言编写的程序中的错误负责¹²。

一种语言应该提供一个坚实的基础，具有很多创造性和控制性。允许存在 `null` 引用只是 Java 的许多设计选择之一，不过也不仅仅是这样。正如在第十章中解释的“捕获或声明要求”和 `try`-`catch` 块为你提供了应对明显错误的工具一样。但是由于 `null` 是任何类型的有效值，每个引用都可能会导致崩溃。即使你认为某些东西永远不会是 `null`，经验告诉我们，某个时刻可能是可能的。

存在 `null` 引用并不能使一种语言被认为设计不良。`null` 有其存在的理由，但这要求你对自己的代码更加注意。这并不意味着你应该将代码中的每个变量和参数都用 Optionals 替换。

Optionals 的初衷是提供一个有限的机制来处理可选的返回值，所以不要因为方便而过度使用或误用它们。在你控制的代码中，你可以对引用的可能为空性做更多的假设和保证，并相应地处理它，即使没有 Optionals 也可以。如果你遵循本书中强调的其他原则，比如小型、自包含、无副作用的纯函数，那么确保你的代码不会意外返回 `null` 引用就更容易了。

# 总结

+   Java 中没有语言级别或特殊语法可用于处理 `null`。

+   `null` 是一个特殊情况，可以代表“不存在”和“未定义”两种状态，而你无法区分它们。

+   `Optional<T>` 类型允许使用操作链和后备来专门处理这些状态。

+   还有针对基本类型的专门类型，尽管它们不提供功能平等性。

+   其他处理 `null` 的方法也存在，比如注解或最佳实践。

+   并非所有情况都适合使用 Optional。如果数据结构已经有了空值的概念，比如集合，再添加一层反而会适得其反。除非需要表示“未定义”状态，否则不应该将其包装成 Optional。

+   Optionals 和 Streams 之间的互操作性很好，几乎没有摩擦。

+   Optionals 不可序列化，所以如果需要序列化类型，则不要将它们用作私有字段。相反，将 Optionals 用作 getter 的返回值。

+   存在替代实现，比如[Google Guava 框架](https://github.com/google/guava)，尽管 Google 自己建议使用 Java 的 Optional。

+   `null` 本身并不邪恶。不要没有充分理由就用 Optional 替换每个变量。

¹ 变长参数不接受单独的`null`作为参数，因为它是不精确的参数类型，可能代表`Object`或`Object[]`。要向变长参数传递单个`null`，需要将其包装在数组中：`new Object[]{ null }`。

² 许多编程语言都有专门的运算符，用于在可能为`null`的引用上安全调用字段或方法。[安全导航运算符维基百科文章](https://en.wikipedia.org/wiki/Safe_navigation_operator)中详细说明并提供了多种语言的示例。

³ `null`合并运算符类似于缩短的三元运算符。表达式`x != null ? x : y`缩短为`x ?: y`，其中`?:`（问号冒号）是运算符。不过，并非所有语言使用相同的运算符。[维基百科文章](https://en.wikipedia.org/wiki/Null_coalescing_operator)概述了支持哪种运算符形式的不同编程语言。

⁴ Java 的 JIT（即时）编译器执行多种优化以改善执行代码。必要时，当有更多关于其执行方式的信息时，它会重新编译代码。有关可能的优化概述可在[Open JDK Wiki](https://wiki.openjdk.org/display/HotSpot/PerformanceTechniques)上找到。

⁵ 提供标记注释最常见的库是[FindBugz](http://findbugs.sourceforge.net/)（适用于 Java 8 及以前），以及其精神继承者[SpotBugz](https://spotbugs.github.io/)。JetBrains，IntelliJ IDE 和 JVM 语言*Kotlin*的创建者，也[提供了一个包含注解的包](https://github.com/JetBrains/java-annotations)。

⁶ [Checker Framework](https://checkerframework.org)提供了这种在不同工具之间的“非标准”行为的[示例](https://checkerframework.org/manual/#findbugs-nullable)。

⁷ Guava 的[Optional<T>文档](https://guava.dev/releases/snapshot-jre/api/docs/com/google/common/base/Optional.xhtml)明确提到应优先使用 JDK 的变体。

⁸ McCabe, TJ. 1976\. “A Complexity Measure” [IEEE Transactions on Software Engineering, December 1976, Vol. SE-2 No. 4, 308–320](https://doi.org/10.1109/TSE.1976.233837)。

⁹ [Optional<T> reduce​(BinaryOperator<T> accumulator)文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/stream/Stream.xhtml#reduce(java.util.function.BinaryOperator))。

¹⁰ [SonarSource](https://www.sonarsource.com/)规则[RSPEC-2789](https://rules.sonarsource.com/java/RSPEC-2789)检查 Optional 是否为`null`。

¹¹ [官方文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.xhtml) 明确提到了不可预测的标识方法行为作为“API 注释”。

¹² 查尔斯·安东尼·理查德·霍尔爵士在 2009 年的 [“空引用：十亿美元的错误”](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/) 演讲中表达了这一观点。
