## 附录 A. 其他语言更新

在本附录中，我们讨论了 Java 8 中的三个其他语言更新：重复注解、类型注解和泛型目标类型推断。附录 B 讨论了 Java 8 中的库更新。我们不讨论 JDK 8 更新，如 Nashorn 和 Compact Profiles，因为它们是新的 JVM 功能。本书重点介绍 *库* 和 *语言* 更新。如果您对 Nashorn 和 Compact Profiles 感兴趣，请阅读以下链接：[`openjdk.java.net/projects/nashorn/`](http://openjdk.java.net/projects/nashorn/) 和 [`openjdk.java.net/jeps/161`](http://openjdk.java.net/jeps/161)。

### A.1. 注解

Java 8 中的注解机制在两个方面得到了增强：

+   你可以重复注解。

+   你可以对任何类型使用注解。

在解释这些更新之前，快速回顾一下在 Java 8 之前你可以用注解做什么是有价值的。

Java 中的 *注解* 是一种机制，允许你用附加信息装饰程序元素（注意，在 Java 8 之前，只有声明可以被注解）。换句话说，它是一种 *语法元数据*。例如，注解在 JUnit 框架中很受欢迎。在以下代码中，方法 `setUp` 被注解为 `@Before`，而方法 `testAlgorithm` 被注解为 `@Test`：

```
@Before
public void setUp(){
    this.list = new ArrayList<>();
}

@Test
public void testAlgorithm(){
     ...
    assertEquals(5, list.size());
}
```

注解适用于多种用例：

+   在 JUnit 的上下文中，注解可以区分应该作为单元测试运行的方法和用于设置工作的方法。

+   注解可以用于文档。例如，`@Deprecated` 注解用于指示一个方法不应再使用。

+   Java 编译器也可以处理注解，以检测错误、抑制警告或生成代码。

+   注解在 Java EE 中很受欢迎，它们用于配置企业应用程序。

#### A.1.1. 重复注解

以前的 Java 版本禁止在声明上指定给定注解类型的多个注解。因此，以下代码是无效的：

```
@interface Author { String name(); }

@Author(name="Raoul") @Author(name="Mario")  @Author(name="Alan")      *1*
class Book{ }
```

+   ***1* 错误：重复注解**

Java EE 程序员经常使用一种惯用语来规避这个限制。你声明一个新的注解，其中包含一个要重复的注解数组。它看起来像这样：

```
@interface Author { String name(); }
@interface Authors {
    Author[] value();
}

@Authors(
  { @Author(name="Raoul"), @Author(name="Mario") , @Author(name="Alan")}
)
class Book{}
```

`Book` 类的嵌套注解看起来相当丑陋。这就是为什么 Java 8 基本上移除了这个限制，从而使事情变得整洁一些。现在，你可以在声明上指定多个相同类型的注解，前提是它们规定该注解是可重复的。这不是默认行为；你必须明确要求注解可重复。

##### 使注解可重复

如果一个注解被设计为可重复的，你只需使用它即可。但是，如果你为用户提供注解，则需要设置来指定注解可以重复。这里有两个步骤：

1.  将注解标记为`@Repeatable`。

1.  提供一个容器注解。

这样你可以使`@Author`注解可重复：

```
@Repeatable(Authors.class)
@interface Author { String name(); }
@interface Authors {
    Author[] value();
}
```

因此，`Book`类可以用多个`@Author`注解进行注解：

```
@Author(name="Raoul") @Author(name="Mario")  @Author(name="Alan")
class Book{ }
```

在编译时，`Book`类被认为是通过`@Authors({ @Author(name= "Raoul"), @Author(name="Mario"), @Author(name="Alan")})`注解的，因此你可以将这种新机制视为 Java 程序员之前使用的惯用语的语法糖。注解仍然被封装在一个容器中，以确保与旧版反射方法的兼容性。Java API 中的`getAnnotation(Class<T> annotationClass)`方法返回注解元素上的类型为`T`的注解。如果有多个类型为`T`的注解，这个方法应该返回哪个注解？

不深入细节，类`Class`支持一个新的`get-AnnotationsByType`方法，它简化了可重复注解的处理。例如，你可以如下使用它来打印`Book`类上的所有`Author`注解：

```
public static void main(String[] args) {
    Author[] authors = Book.class.getAnnotationsByType(Author.class);      *1*
    Arrays.asList(authors).forEach(a -> { System.out.println(a.name()); });
}
```

+   ***1* 获取由可重复的`Author`注解组成的数组**

为了使这可行，可重复注解及其容器都必须有`RUNTIME`保留策略。有关与旧版反射方法兼容性的更多信息，请参阅此处：[`cr.openjdk.java.net/~abuckley/8misc.pdf`](http://cr.openjdk.java.net/~abuckley/8misc.pdf)。

#### A.1.2\. 类型注解

截至 Java 8，注解也可以应用于任何类型的使用。这包括`new`运算符、类型转换、`instanceof`检查、泛型类型参数以及`implements`和`throws`子句。在这里，我们使用`@NonNull`注解表明`String`类型的变量`name`不能为`null`：

```
@NonNull String name = person.getName();
```

同样，你可以注解列表中元素的类型：

```
List<@NonNull Car> cars = new ArrayList<>();
```

这有什么有趣的地方？类型上的注解可以用于程序分析。在这两个例子中，一个工具可以确保`getName`不会返回`null`，以及列表中的汽车元素始终不是`null`。这可以帮助减少你代码中的意外错误。

Java 8 不提供官方注解或使用它们的工具。它只提供了在类型上使用注解的能力。幸运的是，存在一个名为 Checker 框架的工具，它定义了几个类型注解，并允许你使用它们来增强类型检查。如果你感兴趣，我们邀请你查看它的教程：[`www.checkerframework.org`](http://www.checkerframework.org)。有关在你的代码中可以使用注解的位置的更多信息，请参阅此处：[`docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.7.4`](http://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.7.4)。

### A.2\. 泛化目标类型推断

Java 8 增强了泛型参数的推断。你已经在 Java 8 之前熟悉了使用上下文信息进行类型推断。例如，Java 中`empty-List`方法的定义如下：

```
static <T> List<T> emptyList();
```

方法`emptyList`使用类型参数`T`进行参数化。你可以如下调用它以向类型参数提供显式类型：

```
List<Car> cars = Collections.<Car>emptyList();
```

但 Java 能够推断泛型参数。以下等价：

```
List<Car> cars = Collections.emptyList();
```

在 Java 8 之前，这种基于上下文（即目标类型）的推断机制是有限的。例如，以下是不可能的：

```
static void cleanCars(List<Car> cars) {

}
cleanCars(Collections.emptyList());
```

你会得到以下错误：

```
cleanCars (java.util.List<Car>)cannot be applied to
     (java.util.List<java.lang.Object>)
```

为了修复它，你必须提供像我们之前展示的那样显式的类型参数。

在 Java 8 中，目标类型包括方法参数，因此你不需要提供显式的泛型参数：

```
List<Car> cleanCars = dirtyCars.stream()
                               .filter(Car::isClean)
                               .collect(Collectors.toList());
```

在此代码中，正是这种增强让你能够编写`Collectors.toList()`而不是`Collectors.<Car>toList()`。
