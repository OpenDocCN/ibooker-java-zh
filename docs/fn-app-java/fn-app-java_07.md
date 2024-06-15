# 第五章：使用 Records

Java 14 引入了一种新的数据结构类型作为预览¹ 功能，两个版本后最终定型：*Records*。它们不仅仅是另一种典型的 Java 类型或技术，您可以使用。相反，Records 是一个全新的语言特性，为您提供了一个简单但功能丰富的数据聚合器，具有最少的样板代码。

# 数据聚合类型

从一般角度来看，*数据聚合* 是从多个来源收集数据并以更适合预期目的和更可取的用法组装成的格式的过程。也许最知名的数据聚合类型是 *元组*。

## 元组

从数学角度来看，一个元组是一个“有限元素的有序序列”。在编程语言方面，一个元组是一个聚合多个值或对象的数据结构。

有两种元组。*结构化* 元组只依赖于包含元素的顺序，因此只能通过它们的索引访问，如下面的 Python 代码所示：

```java
apple = ("apple", "green")
banana = ("banana", "yellow")
cherry = ("cherry", "red")

fruits = [apple, banana, cherry]

for fruit in fruits:
  print "The", fruit[0], "is", fruit[1]
```

*名义* 元组不使用索引来访问它们的数据，而是使用组件名称，如下面的 Swift 代码所示：

```java
typealias Fruit = (name: String, color: String)

let fruits: [Fruit] = [
  (name: "apple", color: "green"),
  (name: "banana", color: "yellow"),
  (name: "cherry", color: "red")]

for fruit in fruits {
  println("The \(fruit.name) is \(fruit.color)")
}
```

为了展示 Records 提供了什么，你首先将看看如何从经典的 POJO 转变为一个不可变的 POJO，然后我将展示如何用 Record 来复制相同的功能。

## 一个简单的 POJO

首先，让我们看看 Java 中数据聚合的“Record 之前”的状态，以更好地把握 Records 提供了什么。举个例子，我们创建一个简单的“用户”类型作为“经典” POJO，演变为一个“不可变” POJO，最后变为一个 Record。它将是一个简单的类型，有一个用户名，一个活动状态，一个上次登录时间戳，以及在典型 Java 代码中常见的“典型”样板，如 示例 5-1 中所示。

##### 示例 5-1。简单的用户 POJO

```java
public final class User {

  private String        username;
  private boolean       active;
  private LocalDateTime lastLogin;

  public User() { } ![1](img/1.png)

  public User(String username,
              boolean active,
              LocalDateTime lastLogin) { ![1](img/1.png)
    this.username = username;
    this.active = active;
    this.lastLogin = lastLogin;
  }

  public String getUsername() { ![2](img/2.png)
    return this.username;
  }

  public void setUsername(String username) { ![3](img/3.png)
    this.username = username;
  }

  public boolean isActive() { ![2](img/2.png)
    return this.active;
  }

  public void setActive(boolean active) { ![3](img/3.png)
    this.active = active;
  }

  public LocalDateTime getLastLogin() { ![2](img/2.png)
    return this.lastLogin;
  }

  public void setLastLogin(LocalDateTime lastLogin) { ![3](img/3.png)
    this.lastLogin = lastLogin;
  }

  @Override
  public int hashCode() { ![4](img/4.png)
    return Objects.hash(this.username,
                        this.active,
                        this.lastLogin);
  }

  @Override
  public boolean equals(Object obj) { ![5](img/5.png)
    if (this == obj) {
      return true;
    }

    if (obj == null || getClass() != obj.getClass()) {
      return false;
    }

    User other = (User) obj;
    return Objects.equals(this.username, other.username)
           && this.active == other.active
           && Objects.equals(this.lastLogin, other.lastLogin);
  }

  @Override
  public String toString() { ![5](img/5.png)
    return new StringBuilder().append("User [username=")
                              .append(this.username)
                              .append(", active=")
                              .append(this.active)
                              .append(", lastLogin=")
                              .append(this.lastLogin)
                              .append("]")
                              .toString();
  }
}
```

![1](img/#co_working_with_records_CO1-1)

构造函数并非绝对必要，但出于便利起见会添加。如果存在带参数的任何构造函数，则还应添加一个显式的“空”构造函数。

![2](img/#co_working_with_records_CO1-3)

POJOs 通常具有 getter 而不是公共字段。

![3](img/#co_working_with_records_CO1-4)

第一种 `User` 类型的变体仍然是可变的，因为它具有 setter 方法。

![4](img/#co_working_with_records_CO1-9)

`hashCode` 和 `equals` 的第一种变体需要根据类型的实际结构进行专门实现。对类型的任何更改都需要调整这两种方法。

![5](img/#co_working_with_records_CO1-10)

`toString` 方法是另一个不是明确必需的便利添加。就像之前的方法一样，每当类型更改时都必须更新它。

包括空行和大括号，仅仅用于保存三个数据字段就有大约 75 行。难怪 Java 最常见的抱怨之一是其冗长性，以及完成*标准*任务时“过于繁琐”的感觉！

现在，让我们将其转换为一个不可变的 POJO。

## 从 POJO 到不可变性

使`User` POJO 不可变会稍微减少所需的样板代码，因为您不再需要任何 setter 方法，正如示例 5-2 中所示。

##### 示例 5-2\. 简单的不可变用户类型

```java
public final class User {

  private final String username; ![1](img/1.png)
  private final boolean active;
  private final LocalDateTime lastLogin;

  public User(String username,
              boolean active,
              LocalDateTime lastLogin) { ![2](img/2.png)
    this.username = username;
    this.active = active;
    this.lastLogin = lastLogin;
  }

  public String getUsername() { ![3](img/3.png)
    return this.username;
  }

  public boolean isActive() { ![3](img/3.png)
    return this.active;
  }

  public LocalDateTime getLastLogin() { ![3](img/3.png)
    return this.lastLogin;
  }

  @Override
  public int hashCode() { ![4](img/4.png)
    // UNCHANGED
  }

  @Override
  public boolean equals(Object obj) { ![4](img/4.png)
    // UNCHANGED
  }

  @Override
  public String toString() { ![4](img/4.png)
    // UNCHANGED
  }
}
```

![1](img/#co_working_with_records_CO2-1)

没有“setters”，字段可以声明为`final`。

![2](img/#co_working_with_records_CO2-2)

仅可能存在一个完全的“传递”构造函数，因为字段必须在对象创建时设置。

![3](img/#co_working_with_records_CO2-3)

“getters”与可变变体保持不变。

![4](img/#co_working_with_records_CO2-6)

与先前的实现相比，支持方法也未更改。

通过使类型本身不可变，只需删除 setter 和空构造函数的代码；其他所有内容仍然存在。对于仅包含三个`public final`字段和一个构造函数的类，这仍然是相当多的代码。当然，根据您的需求，这可能是“恰到好处”的选择。然而，额外的功能，如相等比较、正确的`hashCode`以便在`Set`或`HashMap`中使用，或合理的`toString`输出，都是值得拥有的功能。

## 从 POJO 到 Record

最后，让我们看一下使用 Record 的更通用、不那么繁琐但功能丰富的解决方案：

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {
  // NO BODY
}
```

那就是这样。

`User` Record 具有与不可变 POJO 相同的功能。如何用如此少的代码实现如此多的功能将在接下来的章节中详细解释。

# 记录来解救

Records 是定义纯粹*数据聚合器类型*的一种方式，它们通过名称访问其数据组件，类似于*命名元组*。与命名元组一样，Records 聚合有序序列的值，并通过名称而不是索引提供访问。它们的数据是浅不可变的，并且透明地可访问。通过生成访问器和数据驱动方法如`equals`和`hashCode`显著减少了其他数据类的典型样板。尽管[JEP 395 的最终版本](https://openjdk.java.net/jeps/395)明确指出“反对样板代码”不是目标，但许多开发人员仍将其视为一个令人满意的巧合。

作为“plain”数据聚合器类型，与其他选项相比确实缺少一些功能。本章将覆盖每个缺失的功能以及如何缓解它们，将 Records 转变为更灵活的解决方案，以满足您的数据聚合需求。

如前一节所示，Records 使用一个新关键字 — `record` — 来将它们与其他类和枚举分隔开来。数据组件直接在 Record 名称后的一对括号中声明，类似于构造函数或方法参数：

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {
  // NO BODY
}
```

记录的一般语法分为两部分：一个*标头*定义了与其他类型相同的属性，以及其组件和一个可选的*主体*，以支持额外的构造函数和方法。

```java
// HEADER
[visibility] record [Name]<optional generic types> {
  // BODY
}
```

标头类似于`class`或`interface`的标头，由多个部分组成：

可见性

与`class`、`enum`或`interface`定义类似，Record 支持 Java 的可见性关键字（`public`、`private`、`protected`）。

记录关键字

关键字`record`将标头与其他类型声明`class`、`enum`和`interface`区分开来。

名称

命名规则与《Java 语言规范》中定义的任何其他标识符相同，如*Java 语言规范*⁠²所述。

[泛型类型](https://example.org/generic_types)

泛型类型与 Java 中的其他类型声明一样受支持。

数据组件

名称后跟着一对括号，其中包含 Record 的组件。每个组件在幕后转换为一个`private final`字段和一个`public`访问器方法。组件列表还表示 Record 的构造函数。

主体

典型的 Java 主体，与任何其他`class`或`interface`类似。

编译器将一行代码有效地转换为与上一节中的示例 5-2 类似的类。它明确扩展了`java.lang.Record`而不是隐式地扩展`java.lang.Object`，就像枚举与`java.lang.Enum`一样。

## 背后的幕后

任何 Record 后面生成的类都会为您提供相当多的功能，而无需编写任何额外的代码。现在是时候深入了解实际发生的事情了。

JDK 包括命令`javap`，用于反汇编`.class`文件，并允许您查看字节码的 Java 对应 Java 代码。通过这种方式，很容易比较“数据聚合类型”中的`User`类型的 POJO 和 Record 版本之间的实际差异。两个变体的组合和清理后的输出显示在示例 5-3 中。

##### 示例 5-3\. Disassembled User.class POJO versus Record

```java
// IMMUTABLE POJO

public final class User {
  public User(java.lang.String, boolean, java.time.LocalDateTime);
  public java.lang.String getUsername();
  public boolean isActive();
  public java.time.LocalDateTime getLastLogin();

  public int hashCode();
  public boolean equals(java.lang.Object);
  public java.lang.String toString();
}

// RECORD

public final class User extends java.lang.Record {
  public User(java.lang.String, boolean, java.time.LocalDateTime);
  public java.lang.String username();
  public boolean active();
  public java.time.LocalDateTime lastLogin();

  public final int hashCode();
  public final boolean equals(java.lang.Object);
  public final java.lang.String toString();
}
```

如您所见，生成的类在功能上是相同的，只是访问器方法的命名不同。那么所有这些方法都来自哪里呢？嗯，这就是 Records 的“魔法”，让您获得了一个完整的数据聚合类型，而无需写更多的绝对必要的代码。

## 记录功能

Records 是透明的数据聚合器，具有特定的保证属性和明确定义的行为，通过自动³提供功能，无需重复编写以下琐碎的样板实现：

+   [组件访问器](https://example.org/component_accessors)

+   三种类型的构造函数

+   对象标识和描述方法

除了记录声明之外，这需要大量的功能而不需要任何额外的代码。任何缺失的部分都可以通过根据需要增加或重写这些功能来完成。

让我们来看看 Record 的自动特性以及其他典型的 Java 特性，比如泛型、注解和反射是如何适配的。

### 组件访问器

所有记录组件都存储在`private`字段中。在记录内部，其字段可以直接访问。从“外部”访问它们需要通过生成的`public`访问器方法。访问器方法的名称与其组件名称对应，不带典型的“getter”前缀`get`，就像以下代码示例中所示：

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {
  // NO BODY
}

var user = new User("ben", true, LocalDateTime.now());

var username = user.username();
```

访问器方法按原样返回对应字段的值。虽然可以重写它们，就像下面的代码所示，但我不建议这样做。

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {

  @Override
  public String username() {
    if (this.username == null) {
      return "n/a";
    }

    return this.username;
  }
}

var user = new User(null, true, LocalDateTime.now());

var username = user.username();
// => n/a
```

记录应该是*不可变*的数据持有者，因此在访问其数据时做出决策可能被视为代码异味。记录的创建定义了其数据，这就是任何验证或其他逻辑应该影响数据的地方，正如您将在下一节中了解的那样。

### 规范、紧凑和自定义构造函数

与记录组件定义相同的构造函数会自动可用，称为*规范*构造函数。记录的组件被直接分配给相应的字段“原样”。与组件访问器一样，规范构造函数可重写以验证输入，如`null`检查，甚至在必要时操作数据：

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {

  public User(String username,
              boolean active,
              LocalDateTime lastLogin) {

    Objects.requireNonNull(username);
    Objects.requireNonNull(lastLogin);

    this.username = username;
    this.active = active;
    this.lastLogin = lastLogin;
  }
}
```

为了进行两个实际的`null`检查，包括重新声明构造函数签名并将组件分配给不可见字段，这是相当多的额外代码行。

幸运的是，还提供了一种专门的*紧凑*形式，如下面的代码示例所示，并且如果不需要，它不会强制您重复任何样板代码。

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {

  public User { ![1](img/1.png)

    Objects.requireNonNull(username);
    Objects.requireNonNull(lastLogin);

    username = username.toLowerCase(); ![2](img/2.png)

    ![3](img/3.png)
  }
}
```

![1](img/#co_working_with_records_CO3-1)

构造函数省略了所有参数，包括括号。

![2](img/#co_working_with_records_CO3-2)

在紧凑的规范构造函数中不允许字段分配，但可以在分配之前自定义或标准化数据。

![3](img/#co_working_with_records_CO3-3)

组件将自动分配给其各自的字段。

虽然语法可能看起来不寻常，因为它省略了所有参数，包括括号。不过，这样做可以清楚地区分它与无参数构造函数。

紧凑的构造函数是放置任何验证的理想位置，正如我将在“记录验证和数据清理”中展示的那样。

与类一样，你可以声明额外的构造函数，但任何自定义构造函数必须以显式调用规范构造函数作为其第一条语句。相比于类，这是一个相当严格的要求。然而，这一要求对我即将讨论的一个重要特性至关重要，详情请见“组件默认值和便捷构造函数”。

### 对象标识和描述

记录提供了基于数据相等性的对象标识方法`int hashCode()`和`boolean equals(Object)`的“标准”实现。如果没有显式实现这两个对象标识方法，当记录的组件发生变化时，您无需担心更新代码。如果两个 Record 类型的实例的组件数据相等，则它们被视为相等。

对象描述方法`String toString()`也是从组件自动生成的，为您提供了一个合理的默认输出，例如：

```java
User[username=ben, active=true, lastLogin=2023-01-11T13:32:16.727249646]
```

对象标识和描述方法也是可重写的，就像组件访问器和构造函数一样。

### 通用类型

记录也支持通用类型，遵循“通常”的规则：

```java
public record Container<T>(T content,
                           String identifier) {
  // NO BODY
}

Container<String> stringContainer = new Container<>("hello, String!",
                                                    "a String container");

String content = stringContainer.content();
```

就个人而言，我建议避免滥用通用 Records。使用更具体的 Records 更接近于它们所代表的领域模型，可以提供更多的表现力，并减少意外误用的可能性。

### 注解

如果用于记录的组件上，注解的行为与您预期的有些不同：

```java
public record User(@NonNull String username,
                   boolean active,
                   LocalDateTime lastLogin) {
  // NO BODY
}
```

乍一看，`username`看起来像是一个参数，因此一个明智的结论是只有具有`ElementType.PARAMETER`目标的注解才可能存在⁴。但是对于 Records 及其自动生成的字段和组件访问器，必须考虑一些特殊情况。为了支持对这些特性进行注解，如果应用于组件，则任何具有`FIELD`、`PARAMETER`或`METHOD`目标的注解将传播到相应的位置。

除了现有的目标外，还引入了新的目标`ElementType.RECORD_COMPONENT`，用于在记录中实现更精细的注解控制。

### 反射

为了补充 Java 的反射能力，Java 16 添加了`getRecordComponents`方法到`java.lang.Class`中。对于基于 Record 的类型，该调用将返回一个数组，其中包含`java.lang.reflect.RecordComponent`对象，对于其他类型的`Class`则返回`null`。这些组件按照在记录头中声明的顺序返回，允许您通过在 Record 类上调用`getDeclaredConstructor()`来查找规范构造函数。

您可以在书籍的[代码存储库](https://github.com/benweidig/a-functional-approach-to-java)中找到一些基于反射的示例。

## 缺失的功能

Records 正是它们应该是的：*简单的、透明的、浅不可变的数据聚合器*。它们提供了大量功能，而无需编写除了它们的定义之外的任何代码。与其他可用的数据聚合器相比，它们缺少一些你可能习惯的功能，比如：

+   附加状态

+   继承

+   (简单的) 默认值

+   逐步创建

本节将向你展示哪些功能是“失踪”的，以及如何在可能的情况下加以缓解。

### 附加状态

允许任何附加的不透明状态是 Records 显而易见的遗漏。它们应该是*数据聚合器*，代表着一个透明的状态。这就是为什么向其主体添加任何附加字段都会导致编译器错误。

###### 提示

如果你需要的字段超过了 Record 组件本身所能提供的可能性，那么 Records 可能不是你要找的数据结构。

对于至少一些场景，你可以添加基于现有组件的*派生*状态，通过向 Records 添加方法：

```java
public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {

  public boolean hasLoggedInAtLeastOnce() {
    return this.lastLogin != null;
  }
}
```

可以添加方法，因为它们不会引入像字段一样的附加状态。它们可以访问 `private` 字段，即使组件访问器被重写，也能保证逐字数据访问。选择哪种 — 字段还是访问器 — 取决于你如何设计你的 Record 以及你个人的偏好。

### 继承

Records 是 `final` 类型，它们在幕后已经扩展了 `java.lang.Record`，就像之前在 示例 5-3 中看到的那样。因为 Java 不允许继承多个类型，所以 Records 不能使用继承。但这并不意味着它们不能实现任何接口。通过接口，你可以定义 Record 模板，并使用 `default` 方法共享公共功能。

示例 5-4 展示了如何为具有公共概念的多个形状创建 Records，即原点和表面积。

##### 示例 5-4\. 使用接口作为 Records 模板

```java
public interface Origin {

  int x(); ![1](img/1.png)
  int y(); ![1](img/1.png)

  default String origin() { ![2](img/2.png)
    return String.format("(%d/%d)", x(), y());
  }
}

public interface Area {

  float area(); ![3](img/3.png)
}

// DIFFERENT RECORDS IMPLEMENTING INTERFACES

public record Point(int x, int y) implements Origin {
  // NO BODY
}

public record Rectangle(int x, int y, int width, int height)
  implements Origin, Area {

  public float area() { ![3](img/3.png)
    return (float) (width() * height());
  }
}

public record Circle(int x, int y, int radius)
  implements Origin, Area {

  public float area() { ![3](img/3.png)
    return (float) Math.PI * radius() * radius();
  }
}
```

![1](img/#co_working_with_records_CO4-1)

该接口将实现记录的组件定义为具有正确名称的简单方法

![2](img/#co_working_with_records_CO4-3)

共享功能是通过 `default` 方法添加的。

![3](img/#co_working_with_records_CO4-4)

接口中的方法签名不得与任何实现记录类型相冲突。

与接口和`default`方法共享行为是一种简单的方法，只要所有实现者共享接口合同。接口可以为缺失继承的几个部分提供补充，可能会引人创建复杂的记录层次结构和互相依赖。但是以这种方式构造记录类型将会产生它们之间的内聚性，并不符合记录作为简单数据聚合器定义的原始精神。此示例是为了更好地说明多个接口可能性而过度工程化的。在现实世界中，您很可能也会将`Origin`作为记录，并使用组合和额外的构造函数来实现相同的功能。

### 组件默认值和便捷构造函数

不同于许多其他语言，Java 不支持任何构造函数或方法参数的默认值。记录只提供其带有所有组件的标准构造函数，这可能会变得笨重，特别是在组合数据结构的情况下：

```java
public record Origin(int x, int y) {
  // NO BODY
}

public record Rectangle(Origin origin, int width, int height) {
  // NO BODY
}

var rectangle = new Rectangle(new Origin(23, 42), 300, 400);
```

附加构造函数提供了一个简便的方法来获取合理的默认值：

```java
public record Origin(int x, int y) {

  public Origin() {
    this(0, 0);
  }
}

public record Rectangle(Origin origin, int width, int height) {

  public Rectangle(int x, int y, int width, int height) { ![1](img/1.png)
    this(new Origin(x, y), width, height);

  }

  public Rectangle(int width, int height) { ![2](img/2.png)
    this(new Origin(), width, height);
  }

  // ...
}

var rectangle = new Rectangle(23, 42, 300, 400);
// => Rectangle[origin=Origin[x=23, y=42], width=300, height=400]
```

![1](img/#co_working_with_records_CO5-1)

第一个附加构造函数模仿`Origin`的组件，以提供创建`Rectangle`的更便捷方式。

![2](img/#co_working_with_records_CO5-2)

第二个便捷构造函数通过删除提供`Origin`的必要性来实现。

由于 Java 的命名语义，不是所有默认值组合都可能存在，比如`Rectangle(int x, float width, float height)`与`Rectangle(int y, float width, float height)`具有相同的签名。在这种情况下，使用`static`工厂方法允许您创建所需的任何组合：

```java
public record Rectangle(Origin origin, int width, int height) {

  public static Rectangle atX(int x, int width, int height) {
    return new Rectangle(x, 0, width, height);
  }

  public static Rectangle atY(int y, int width, int height) {
    return new Rectangle(0, y, width, height);
  }

  // ...
}

var xOnlyRectangle = Rectangle.atX(23, 300, 400);
// => Rectangle[origin=Origin[x=23, y=0], width=300, height=400]
```

使用`static`工厂方法是自定义构造函数的更具表现力的替代方案，并且是重叠签名的唯一选择。

对于无参数构造函数，使用常量更有意义：

```java
public record Origin(int x, int y) {

    public static Origin ZERO = new Origin(0, 0);
}
```

首先，您的代码通过常量的有意义名称更具表现力。其次，只创建一个单一实例，因为底层数据结构是不可变的，所以在任何地方都是常量。

### 逐步创建

不可变数据结构的一个优点是缺乏“半初始化”的对象。然而，并非每个数据结构都能一次初始化。在这种情况下，您可以使用*构建器模式*来获取一个可变的中间变量，用于创建最终不可变的结果。尽管构建器模式起源于解决面向对象编程中重复对象创建问题的解决方案，但在更功能化的 Java 环境中，它对于创建不可变数据结构也是非常有益的。

通过将数据结构的构建与其表示分离，数据结构本身可以尽可能简单，使模式与 Records 完美匹配。任何必需的逻辑或验证都封装到一个（多步骤的）构建器中。

之前使用的 `User` 记录可以通过一个简单的构建器进行补充，如 Example 5-5 所示。

##### 示例 5-5\. 用户构建器

```java
public final class UserBuilder {

  private final String username;

  private boolean       active;
  private LocalDateTime lastLogin;

  public UserBuilder(String username) {
    this.username = username;
    this.active = true; ![1](img/1.png)
  }

  public UserBuilder active(boolean isActive) { ![2](img/2.png)
    if (this.active == false) { ![3](img/3.png)
      throw new IllegalArgumentException("...");
    }

    this.active = isActive;
    return this; ![4](img/4.png)
  }

  public UserBuilder lastLogin(LocalDateTime lastLogin) { ![5](img/5.png)
    this.lastLogin = lastLogin;
    return this;
  }

  public User build() { ![6](img/6.png)
    return new User(this.username, this.active, this.lastLogin);
  }
}

var builder = new UserBuilder("ben").active(false) ![7](img/7.png)
                                    .lastLogin(LocalDateTime.now());

// ...

var user = builder.build(); ![8](img/8.png)
```

![1](img/#co_working_with_records_CO6-1)

显式默认值是可能的，减少了创建所需的代码。

![2](img/#co_working_with_records_CO6-2)

在构建期间可以更改的字段需要类似 setter 的方法。

![3](img/#co_working_with_records_CO6-3)

验证逻辑绑定到特定的 setter 类型方法，而不是在任何构造函数中累积。

![4](img/#co_working_with_records_CO6-4)

返回 `this` 创建了一个流畅的构建器 API。

![5](img/#co_working_with_records_CO6-5)

`Optional` 字段可以使用它们的显式类型，并且仅在 `build()` 期间更改为 `Optional`。

![6](img/#co_working_with_records_CO6-6)

如果构建完成，调用 `build()` 将创建实际的不可变 `User` 记录。通常，构建器应根据需要验证其状态。

![7](img/#co_working_with_records_CO6-7)

构建过程流畅，您可以像处理任何其他变量一样传递构建器。

![8](img/#co_working_with_records_CO6-8)

最后，通过调用 `build()` 创建不可变对象。

将构建器类直接放置在相应类型中作为 `static` 嵌套类，可以增强类型与其构建器之间的黏合度，如在 Example 5-6 中所示。

##### 示例 5-6\. 嵌套构建器

```java
public record User(long id,
                   String username,
                   boolean active,
                   Optional<LocalDateTime> lastLogin) {

  public static final class Builder {
    // ...
  }
}

var builder = new User.Builder("ben");
```

使用记录来实现简单性和不可变性似乎是没有意义的，但仍然引入了构建器的复杂性。为什么不使用一个完整的 bean 呢？因为即使有构建器的复杂性，创建和使用数据的关注点是分开的。即使没有构建器，记录仍然可用，但构建器提供了创建记录实例的额外灵活方式。

# 用例和常见实践

Records 节省了大量样板代码，只需稍作增加，即可将它们升级为更加灵活和多功能的工具。

## 记录验证和数据清理

如在 “规范、紧凑和自定义构造函数” 中所示，Records 支持与 *普通* 构造函数行为不同的 *紧凑构造函数*。您可以访问规范构造函数的所有组件，但它没有任何参数。它为初始化过程中需要放置的任何 *附加* 代码提供了一个位置，而无需自行分配组件。这使得它成为放置任何验证和数据清理逻辑的理想位置：

```java
public record NeedsValidation(int x, int y) {

  public NeedsValidation {
    if (x < y) {
      throw new IllegalArgumentException("x must be equal or greater than y");
    }
  }
}
```

抛出异常是一种选择。另一种选项是*清洗*数据，并使用合理的替代方案调整组件值，以形成有效的记录：

```java
public record Time(int minutes, int seconds) {

  public Time {
    if (seconds >= 60) {
      int additionalMinutes = seconds / 60;
      minutes += additionalMinutes;
      seconds -= additionalMinutes * 60;
    }
  }
}

var time = new Time(12, 67);
// => Time[minutes=13, seconds=7]
```

将某些逻辑（例如越界值的归一化）直接移入记录中可以提供更一致的数据表示，不受初始数据的影响。另一种方法是要求事先进行此类数据清洗，并通过抛出适当的异常来限制记录仅进行*严格*验证。

## 增加不可变性

在“不可变集合”中，您了解到集合中浅不可变性的问题。浅不可变数据结构具有不可变引用，但其引用的数据仍可变。必须考虑到与非固有不可变记录组件相同的意外更改问题。通过尝试通过复制或重新包装来增加不可变性的级别是减少记录组件中任何更改的简单方法。

您可以使用规范构造函数创建组件的不可变副本：

```java
public record IncreaseImmutability(List<String> values) {

  public IncreaseImmutability {
    values = Collections.unmodifiableList(values);
  }
}
```

调用`Collections.unmodifiableList`创建了原始`List`的内存节省但不可修改的视图。这可以防止对记录组件的更改，但无法通过原始引用控制对底层`List`的更改。通过使用 Java 10+的方法`List.copy(Collection<? extends E> coll)`可以实现更高级别的不可变性，创建与原始引用独立的深层副本。

## 创建修改副本

即使记录的声明尽可能简化，创建稍作修改的副本仍然需要您自己动手，而无需依赖 JDK 的任何帮助。

如果您不想完全手动执行，创建修改副本有多种方法：

+   Wither 方法

+   构建器模式

+   工具辅助

+   反射

### Wither 方法

*Wither 方法* 遵循命名方案 `withcomponentName`。它们类似于设置器，但返回一个新实例而不是修改当前实例：

```java
public record Point(int x, int y) {

  public Point withX(int newX) {
    return new Point(newX, y());
  }

  public Point withY(int newY) {
    return new Point(x(), newY);
  }
}

var point = new Point(23, 42);
// => Point[x=23, y=42]

var newPoint = point.withX(5);
// => Point[x=5, y=42]
```

嵌套记录是将修改逻辑与实际记录分离的便捷方式：

```java
public record Point(int x, int y) {

  public With with() {
    return new With(this);
  }

  public record With(Point source) {

    public Point x(int x) {
      return new Point(x, source.y());
    }

    public Point y(int y) {
      return new Point(source.x(), y);
    }
  }
}

var sourcePoint = new Point(23, 42);

var modifiedPoint = sourcePoint.with().x(5);
```

原始记录只有一个额外的方法，并且所有的变异器/复制方法都封装在`With`类型中。

像“组件默认值和便捷构造函数”中的默认值一样，wither 方法的最明显缺点是需要为每个组件编写一个方法。将代码限制在最常见的场景中是明智的，只在需要时添加新方法。

### 构建器模式

构建器模式，如在“逐步创建”中介绍的，还允许更容易地管理更改。这样的构造函数允许您使用现有记录初始化构建器，进行适当的更改，并创建一个新的记录，如下所示：

```java
public record Point(int x, int y) {

  public static final class Builder {

    private int x;
    private int y;

    public Builder(Point point) {
      this.x = point.x();
      this.y = point.y();
    }

    public Builder x(int x) {
      this.x = x;
      return this;
    }

    public Builder y(int y) {
      this.y = y;
      return this;
    }

    public Point build() {
      return new Point(this.x, this.y);
    }
  }
}

var original = new Point(23, 42);

var updated = new Point.Builder(original)
                       .x(5)
                       .build();
```

这种方法与“wither”方法存在相同的问题：组件之间和创建记录副本所需的代码之间存在强烈的凝聚性，使得重构变得更加困难。为了缓解这一问题，你可以使用辅助工具方法。

### 工具辅助生成器

而不是每次记录更改时更新记录生成器类，你可以使用注解处理器来为你完成这项工作。像[RecordBuilder](https://github.com/randgalt/record-builder)这样的工具会为任何记录生成灵活的生成器，你只需添加一个单一的注解：

```java
@RecordBuilder
public record Point(int x, int y) {
  // NO BODY
}

// GENERAL BUILDER
var original = PointBuilder.builder()
                           .x(5)
                           .y(23)
                           .build();

// COPY BUILDER
var modified = PointBuilder.builder(original)
                           .x(12)
                           .build();
```

对记录组件的任何更改将自动在生成的生成器中生效。还可以使用基于“wither”的方法，但需要你的记录实现一个额外生成的接口：

```java
@RecordBuilder
public record Point(int x, int y) implements PointBuilder.With {
  // NO BODY
}

var original = new Point(5, 23);

// SINGLE CHANGE
var modified1 = original.withX(12);

// MULTI-CHANGE VIA BUILDER
var modified2 = original.with()
                        .x(12)
                        .y(21)
                        .build()

// MULTI-CHANGE VIA CONSUMER (doesn't require calling build())
var modified3 = original.with(builder -> builder.x(12)
                                                .y(21));
```

尽管使用外部工具来补充你的记录或任何代码可以节省大量输入，但也存在一些缺点。依赖于一个必不可少的项目部分的工具，会在它们之间形成难以打破的凝聚性。任何错误、安全问题或破坏性变更都可能以无法预料的方式影响你的代码，通常没有可能自行修复。注解处理器集成到你的构建工具中，使它们现在也相互关联。因此，请确保在将它们添加到你的项目之前对这些依赖进行彻底评估⁷。

## 记录作为本地名义元组

许多函数式编程语言中普遍存在一种构造类型在 Java 中缺失：*动态元组*。编程语言通常使用这些作为动态数据聚合器，而无需显式定义类型。Java 记录是简单的数据聚合器，并且在某种意义上可以被视为*名义元组*。与大多数元组实现最显著的不同之处在于，由于 Java 类型系统，它们所包含的数据由一个整体类型组合在一起。记录不像其他语言的元组实现那样灵活或可互换。但由于 Java 15 中对记录的一个补充，你可以将其用作本地*即时*数据聚合器：*本地记录*。

在上下文中本地化记录简化和规范化数据处理，并打包功能。想象一下，你有一个 90 年代的音乐专辑标题列表，按年份分组为`Map<Integer, List<String>>`，如下所示：

```java
Map<Integer, List<String>> albumns =
  Map.of(1990, List.of("Bossanova", " Listen Without Prejudice"),
         1991, List.of("Nevermind", "Ten", "Blue lines"),
         1992, List.of("The Chronic", "Rage Against the Machine"),
         1993, List.of("Enter the Wu-Tang (36 Chambers)"),
         ...
         1999, List.of("The Slim Shady LP", "Californication", "Play"));
```

处理这样的嵌套和不具体的数据结构相当麻烦。迭代 Maps 需要使用`entrySet()`方法，在这种情况下返回`Map.Entry<Integer, List<String>>`实例。处理这些条目可能会让你访问所有数据，但表达方式不够直观。

以下代码使用流管道为音乐专辑标题创建了一个过滤方法。即使没有阅读第六章，该章将详细介绍流，大部分代码也应该很容易理解，但我会指导你完成。

```java
public List<String> filterAlbums(Map<Integer, List<String>> albums,
                                 int minimumYear) {

  return albums.entrySet()
               .stream()
               .filter(entry -> entry.getKey() >= minimumYear) ![1](img/1.png)
               .sorted(Comparator.comparing(Map.Entry::getKey)) ![2](img/2.png)
               .map(Map.Entry::getValue) ![3](img/3.png)
               .flatMap(List::stream) ![4](img/4.png)
               .toList(); ![5](img/5.png)
}
```

![1](img/#co_working_with_records_CO7-1)

过滤至少是最小年份的专辑条目。

![2](img/#co_working_with_records_CO7-2)

按其相应年份对标题列表进行排序。

![3](img/#co_working_with_records_CO7-3)

将条目转换为其实际值。

![4](img/#co_working_with_records_CO7-4)

`flatMap`调用帮助“展平”包含年份'S 标题的`List<String>`元素到管道中的单一元素。

![5](img/#co_working_with_records_CO7-5)

将元素收集到`List<String>`中

每个流操作都必须处理`getKey()`或`getValue()`，而不是在其上下文中表示实际数据的具有表达性的名称。这就是为什么引入本地 Record 作为中间类型允许您在复杂的数据处理任务（如流管道）中恢复表达能力的原因，但任何数据处理都可以从更多的表达性中受益。您甚至可以将部分逻辑移到 Record 中，以便为每个操作使用方法引用或单个调用。

考虑你*拥有*数据的形式，以及它*应该*如何表示，然后相应地设计你的 Record。接下来，你应该将复杂的数据处理任务重构为 Record 方法。可能的候选者包括：

+   从`Map.Entry`实例创建 Record。

+   按年份筛选

+   按年份排序。

下面的 Record 代码展示了这些任务的实现：

```java
public record AlbumsPerYear(int year, List<String> titles) { ![1](img/1.png)

  public AlbumsPerYear(Map.Entry<Integer, List<String>> entry) { ![2](img/2.png)
    this(entry.getKey(), entry.getValue());
  }

  public static Predicate<AlbumsPerYear> minimumYear(int year) { ![3](img/3.png)
    return albumsPerYear -> albumsPerYear.year() >= year;
  }

  public static Comparator<AlbumsPerYear> sortByYear() { ![4](img/4.png)
    return Comparator.comparing(AlbumsPerYear::year);
  }
}
```

![1](img/#co_working_with_records_CO8-1)

Record 组件反映了你希望如何使用更具表达性的名称访问数据。

![2](img/#co_working_with_records_CO8-2)

另外的构造方法允许使用方法引用来创建新实例。

![3](img/#co_working_with_records_CO8-3)

如果任务依赖于超出范围的变量，则应定义为`static`辅助程序。

![4](img/#co_working_with_records_CO8-4)

排序应通过创建返回 `Comparator` 的`static`辅助方法来完成，或者如果只需要支持单一排序，则你的 Record 可以实现`Comparable`接口。

Record `AlbumsPerYear`专门为`filterAlbums`方法的流管道设计，并且只应在其作用域内可用。本地上下文限制了 Record，阻止它通过周围类中的状态泄漏。所有嵌套的 Record 都是隐式`static`的，以防止状态通过周围类泄漏到其中。示例 5-7 展示了 Record 如何存在于方法中，以及 Record 如何改进整体代码。

##### 示例 5-7\. 本地化 Record 的流管道

```java
public List<String> filterAlbums(Map<Integer, List<String>> albums,
                                 int minimumYear) {

  record AlbumsPerYear(int year, List<String> titles) { ![1](img/1.png)
    // ...
  }

  return albums.entrySet()
               .stream()
               .map(AlbumsPerYear::new) ![2](img/2.png)
               .filter(AlbumsPerYear.minimumYear(minimumYear)) ![3](img/3.png)
               .sorted(AlbumsPerYear.sortByYear()) ![3](img/3.png)
               .map(AlbumsPerYear::titles) ![3](img/3.png)
               .flatMap(List::stream) ![4](img/4.png)
               .toList();
}
```

![1](img/#co_working_with_records_CO9-1)

本地化 Record 直接在方法中声明，限制了其作用域。出于可读性考虑，我没有重复实际的实现。

![2](img/#co_working_with_records_CO9-2)

流管道的第一个操作是将`Map.Entry`实例转换为本地 Record 类型。

![3](img/#co_working_with_records_CO9-3)

每个后续操作使用本地记录的表达式方法，可以直接使用，也可以作为方法引用使用，而不是显式的 lambda 表达式。

![4](img/#co_working_with_records_CO9-6)

有些操作很难重构，比如 `flatMap`，因为流的整体处理逻辑决定了它们的使用。

如你所见，使用本地记录是改善声明性流水线的人机工程学和表现力的极好方法，而不会暴露类型超出其明显范围。

## 更好的可选数据处理

处理可选数据和可能的 `null` 值是每个 Java 开发人员的苦恼。一种选择是使用 Bean 验证 API，如 “记录验证和数据清洗” 中所示，并用 `@NonNull` 和 `@Nullable` 注解每个组件，尽管这种方法需要依赖。如果你想留在 JDK 内部，Java 8 通过引入 `Optional<T>` 类型来简化了 `null` 处理的痛苦，你将在 第九章 中详细了解它。现在，你只需要知道它是一种可能包含 `null` 值的容器类型，因此即使值是 `null`，你仍然可以与容器交互，而不会引发 `NullPointerException`。

`Optional` 类型清楚地表示一个组件是可选的，但它需要比仅仅更改类型更多的代码才能成为一个有效的工具。让我们在本章早些时候的 `User` 类型示例中添加一个可选组：

```java
public record User(String username,
                   boolean active,
                   Optional<String> group,
                   LocalDateTime lastLogin) {
  // NO BODY
}
```

即使使用 `Optional<String>` 存储用户的组，你仍然必须处理可能为整个容器接收到 `null` 的可能性。更好的选择是接受值本身为 `null`，但仍然具有 `Optional<String>` 组件。随着记录反映它们的定义及其访问器 1:1 的定义，为了更安全和更方便地使用具有可选组件的记录，还需要进行两个额外的步骤。

### 确保非空容器

要使记录更安全和更方便使用可选组件的第一步是确保 `Optional<String>` 不会为 `null`，从而破坏它的设计理念。最简单的方法是使用紧凑的构造函数验证它：

```java
public record User(String username,
                   boolean active,
                   Optional<String> group,
                   LocalDateTime lastLogin) {

  public User {
    Objects.requireNonNull(group, "Optional<String> group must not be null");
  }
}
```

最明显的问题是通过将可能的 `NullPointerException` 从组件访问器移到创建记录本身的时刻来避免，从而使其更安全地使用。

### 添加便利构造函数

使记录更安全和更方便使用的第二步是提供带有非可选参数的额外构造函数，并自己创建容器类型：

```java
public record User(String username,
                   boolean active,
                   Optional<String> group,
                   LocalDateTime lastLogin) {

  public User(String username,
              boolean active,
              String group,
              LocalDateTime lastLogin) {
    this(username,
         active,
         Optional.ofNullable(group),
         lastLogin);
  }

  // ...
}
```

代码完成将显示两个构造函数，指示 `group` 组件的可选性。

在记录创建时进行验证，并提供便利的构造函数，既给记录的创建者带来了灵活性，也让任何消费者能够更安全地使用它。

## 序列化不断发展的记录

如果未显式提及，Records 与类一样，如果实现了空标记接口`java.io.Serializable`，则会自动可序列化。与类相比，Records 的序列化过程遵循更灵活和更安全的策略，无需任何额外的代码。

###### 注意

完整的序列化过程包括*序列化*（将对象转换为字节流）和*反序列化*（从字节流中读取对象）。如果没有明确提到，序列化描述的是整个过程，而不仅仅是第一个方面。

对于普通的非 Record 对象，序列化依赖于昂贵的⁸ 反射来访问它们的私有状态。这个过程可以通过在类型中实现`private`方法`readObject`和`writeObject`进行定制。这两个方法没有由任何接口提供，但仍然属于[Java 对象序列化规范](https://docs.oracle.com/en/java/javase/17/docs/specs/serialization/serial-arch.xhtml)的一部分。它们难以正确实现，并且过去已经导致了许多漏洞⁹。

Record 仅由其不可变状态定义，由其组件表示。在创建后没有任何代码能够影响状态的情况下，序列化过程非常简单：

+   序列化仅基于 Record 的组件。

+   反序列化只需要使用规范构造函数，而不需要反射。

一旦 JVM 推导出 Record 的序列化形式，匹配的实例化程序就可以被缓存。无法定制这个过程，这实际上通过让 JVM 重新控制 Record 的序列化表示来实现更安全的序列化过程。这允许任何 Record 类型通过添加新组件继续演变，并且仍然可以成功地从之前序列化的数据中反序列化。在反序列化过程中遇到的任何未知组件，如果没有提供值，将自动使用其*默认值*（例如，对象类型为`null`，`boolean`类型为`false`等）。

###### 警告

请注意，在使用 JShell 时，序列化的代码示例不会按预期工作。替换 Record 定义后，内部类名称将不会相同，因此类型将无法匹配。

假设你有一个二维的`record Point(float x, float y)`需要进行序列化。以下代码不会带来任何意外：

```java
public record Point(int x, int y) implements Serializable {
  // NO BODY
}

var point = new Point(23, 42);
// => Point[x=23, y=42]

try (var out = new ObjectOutputStream(new FileOutputStream("point.data"))) {
  out.writeObject(point);
}
```

随着需求变化，你需要将 Record 添加第三个维度 `z`，如下面的代码所示。

```java
public record Point(int x, int y, int z) implements Serializable {
  // NO BODY
}
```

如果尝试将`point.data`文件反序列化为已更改的 Record，会发生什么？我们来看看吧！

```java
var in = new ObjectInputStream(new FileInputStream("point.data"));

var point = in.readObject();
// => Point[x=23, y=42, z=0]
```

它只是起作用。

新组件，在`points.data`的序列化表示中缺失，因此无法为 Record 的规范构造函数提供值，将会使用其类型的相应默认值进行初始化，在这种情况下，为`int`类型，初始化为`0`（零）。

如 “Records” 所述，Records 实际上是名义上的元组，仅基于其组件的名称和类型，而不是其确切的顺序。这就是为什么即使改变组件的顺序也不会破坏其反序列化能力。

```java
public record Point(int z, int y, int x) implements Serializable {
  // NO BODY
}

var in = new ObjectInputStream(new FileInputStream("point.data"));

var point = in.readObject();
// => Point[z=0, y=42, x=23]
```

移除组件也是可能的，因为在反序列化过程中会忽略任何缺失的组件。

然而，还存在一个普遍的警告。

从单个 Record 的视角来看，它们仅由它们的组件定义。然而，对于 Java 序列化过程，被序列化的类型也很重要。这就是为什么即使两个 Records 有相同的组件，它们也不能互换。如果尝试将其反序列化为具有相同组件的其他类型，则会遇到 `ClassCastException`：

```java
public record Point(int x, int y) implements Serializable {
  // NO BODY
}

try (var out = new ObjectOutputStream(new FileOutputStream("point.data"))) {
  out.writeObject(new Point(23, 42));
}

public record IdenticalPoint(int x, int y) implements Serializable {
  // NO BODY
}

var in = new ObjectInputStream(new FileInputStream("point.data"));
IdenticalPoint point = in.readObject();
// Error:
// incompatible types: java.lang.Object cannot be converted to IdenticalPoint
```

不同类型的序列化不兼容性是 Records 使用的 “更简单但更安全” 序列化过程的副作用。由于不能像传统 Java 对象那样手动影响序列化过程，您可能需要迁移已经序列化的数据。最直接的方法是将旧数据反序列化为旧类型，将其转换为新类型，并将其序列化为新类型。

## 记录模式匹配 (Java 19+)

即使本书针对的是 Java 11，并试图帮助理解一些新添加的功能，我还是想告诉你一个在写作时仍在开发中的即将推出的功能：*基于记录的模式匹配* ([JEP 405](https://openjdk.java.net/jeps/405))。

###### 注意

JDK 预览功能是 Java 语言、JVM 或 Java API 的新功能，完全指定、实现但是暂时的。一般的想法是收集实际使用的反馈，以便该功能可能在将来的版本中成为永久性的。

Java 16 引入了 `instanceof` 操作符的模式匹配¹⁰，在使用操作符后不再需要强制转换：

```java
// PREVIOUSLY

if (obj instanceof String) {
  String str = (String) obj;
  // ...
}

// JAVA 16+

if (obj instanceof String str) {
    // ...
}
```

Java 17 和 18 扩展了这个想法，通过启用 `switch` 表达式的模式匹配作为预览功能¹¹：

```java
// WITHOUT SWITCH PATTERN MACTHING

String formatted = "unknown";
if (obj instanceof Integer i) {
  formatted = String.format("int %d", i);
} else if (obj instanceof Long l) {
  formatted = String.format("long %d", l);
} else if (obj instanceof String str) {
  formatted = String.format("String %s", str);
}

// WITH SWITCH PATTERN MATCHING

String formatted = switch (obj) {
  case Integer i -> String.format("int %d", i);
  case Long l    -> String.format("long %d", l);
  case String s  -> String.format("String %s", s);
  default        -> "unknown";
};
```

Java 19+ 也包括了这些 Records 的功能，包括解构¹²，这意味着 Record 的组件可以直接作为变量在作用域中使用：

```java
record Point(int x, int y) {
  // NO BODY
};

var point = new Point(23, 42);

if (point instanceof Point(int x, int y)) {
  System.out.println(x + y);
  // => 65
}

int result = switch (anyObject) {
  case Point(var x, var y) -> x + y;
  case Point3D(var x, var y, var z) -> x + y + z;
  default -> 0.0;
};
```

正如你所看到的，Records 仍在演变，具有像模式匹配这样的新功能，这些功能增强了其功能集，使其成为一种更多才多艺、灵活的数据聚合类型，简化了你的代码。

# 关于 Records 的最终想法

Java 的新数据聚合类型 Records 通过尽可能少的代码提供了极大的简化。这是通过遵循特定的规则和限制来实现的，这些规则和限制最初可能看起来是武断和约束的，但它确实提供了更安全和一致的使用方式。Records 并不打算成为“一刀切”解决方案，完全替代所有 POJOs 或其他现有的数据聚合类型。它们仅仅提供了一个适合更功能化和不可变方法的新选项。

可用的功能集是有意选择的，以创建一种新类型的状态表示，*仅限于*状态。定义新记录的简单性不鼓励重用抽象类型，仅仅因为它可能比创建新的更合适的类型更方便。

记录可能不像 POJOs 或自定义类型那样灵活。但灵活性通常意味着更多复杂性，这往往会增加错误的可能性。处理复杂性的最佳方式是尽量减少其表面，并且如果记录的组件发生变化，它们不容易被破坏。

# 要点

+   记录（Records）是由它们的组件定义的透明数据聚合类型。

+   大多数类特有的功能，如实现接口、泛型或注解，也可以在 Records 中使用。

+   任何记录类型都可以获得规范构造函数、组件访问器、对象标识和对象描述的典型样板代码，而无需额外的代码。如有必要，您可以覆盖每一个。

+   记录具有某些限制，以确保其安全和简单的使用。与 POJOs 或 JavaBeans 等更灵活的解决方案相比，许多缺失的功能可以通过仅限于 JDK 的代码或注解处理工具来补充。

+   遵循诸如验证和系统化修改副本的常见做法，可以创建一致的用户体验。

+   记录提供了比基于类的兄弟更安全和更灵活的序列化解决方案。

¹ JDK 预览功能是设计、规范和实现完备，但不是永久性的功能。它旨在从社区中收集反馈以进一步发展。这样的功能可能在未来的版本中以不同的形式存在或完全不存在。

² 参见 Java 语言规范 [章节 3.8](https://docs.oracle.com/javase/specs/jls/se16/html/jls-3.xhtml#jls-3.8)，了解有效 Java 标识符的定义。

³ “自动魔法”一词描述了一种自动过程，对用户隐藏，因此类似于魔法。Records 提供它们的自动功能，无需额外的工具如注解处理器或额外的编译器插件。

⁴ 要了解更多关于注解的一般信息以及如何使用它们，你应该查看我的文章[Java 注解解析](https://belief-driven-design.com/4f54e6e6c3f/)。

⁵ Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). Design patterns: Elements of reusable object-oriented software. Addison Wesley.

⁶ *单一责任原则* 是面向对象编程中 *SOLID* 原则的第一个。其五个原则旨在使 OO 设计更灵活、可维护和简单。

⁷ 我在我的[个人博客](https://belief-driven-design.com/e3e769e891b/)上写了一篇关于如何评估依赖关系的文章。

⁸ 关于反射的“成本”一词与产生的性能开销和暴露于安全问题有关。反射使用动态解析的类型信息，这导致 JVM 无法利用其所有可能的优化。因此，反射比它们的非反射对应物具有更慢的性能。

⁹ 方法`readObject`可以执行任意代码，而不仅仅是读取对象。一些相关的 CVE 包括：[CVE-2019-6503](https://nvd.nist.gov/vuln/detail/CVE-2019-6503)，[CVE-2019-12630](https://nvd.nist.gov/vuln/detail/CVE-2019-12630)，[CVE-2018-1851](https://nvd.nist.gov/vuln/detail/CVE-2018-1851)。

¹⁰ 将`instanceof`运算符扩展为支持 *模式匹配* 的概述在 [JEP 394](https://openjdk.java.net/jeps/394) 中。

¹¹ `switch` 的模式匹配总结在 [JEP 406](https://openjdk.java.net/jeps/406) 和 [JEP 430](https://openjdk.java.net/jeps/420) 中。

¹² 记录的模式匹配总结在 [JEP 405](https://openjdk.java.net/jeps/405) 中。
