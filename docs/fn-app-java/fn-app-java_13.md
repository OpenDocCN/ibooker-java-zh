# 第十一章\. 惰性评估

虽然*懒惰*在人们看来经常是性格缺陷，但在一些编程语言中可以被视为有利特性。在计算机科学术语中，*懒惰*是代码评估的对手，与*严格性* — 或*渴望性* — 相对立。

本章将向您展示如何通过懒惰提高性能。您将了解严格和惰性评估之间的区别及其对代码设计的影响。

# 懒惰与严格性

语言的严格性描述了代码评估的语义。

*严格*评估尽可能快地发生，例如在声明或设置变量或将表达式作为参数传递时。然而，*非严格*评估发生在实际需要表达式结果时。这样，即使一个或多个子表达式无法评估，表达式仍然可以具有值。

例如，*Haskell* 是一种函数式编程语言，默认具有*非严格*语义，从最外层到最内层评估表达式。这允许您创建控制结构或无限数据序列，因为表达式的*创建*和*消费*分离。

让我们来看一个简单方法的*严格* Java 代码，它接受两个参数，但只使用一个来进行逻辑处理：

```java
int add(int x, int y) {
  return x + x;
}
```

*非严格* Haskell 等效函数声明更像变量赋值：

```java
add x y = x + x
```

此函数也仅使用其第一个参数，并且根本不评估第二个参数`y`。这就是为什么以下 Haskell 代码仍然产生结果：

```java
add 5 (1/0)
=> 10
```

如果您使用相同的参数调用此函数的 Java 等效项，值为`1`和表达式`(1/0)`，它将抛出异常：

```java
var result = add(5, (1/0));
// => java.lang.ArithmeticException: Division by zero
```

即使`add`调用的第二个参数在任何情况下都未被使用，作为*严格*语言的 Java 立即评估表达式。方法参数是*按值传递*的，这意味着它们在传递给方法之前会被评估，这在这种情况下会抛出`ArithmeticException`。

###### 注意

Java 的方法参数始终是按值传递的。对于非原始类型，在 JVM 中，参数作为*对象句柄*传递，使用一种称为`引用`的特殊类型。这在技术上仍然是按值传递，这使得一般术语和语义相当混乱。

相反，惰性评估被定义为仅在需要其结果时评估表达式。这意味着表达式的声明不会立即触发其评估，这使得 Java lambda 表达式成为惰性评估的完美匹配，正如在示例 11-1 中所见。

##### 示例 11-1\. Java 和 Suppliers 实现的惰性评估

```java
int add (IntSupplier x, IntSupplier y) {

  var actualX = x.getAsInt();

  return actualX + actualX;
}

var result = add(() -> 5,
                 () -> 1 / 0);
// => 10
```

`IntSupplier` 实例的声明，或其内联等效项，是一个严格语句，并立即评估。然而，实际 lambda 体在没有显式调用`getAsInt`时不会评估，因此在这种情况下避免了`ArithmeticException`。

本质上，*严格性* 是关于“做事情”，但 *惰性* 是关于“考虑要做的事情”。

# Java 有多严格？

大多数编程语言既不是完全惰性也不是完全严格。Java 被认为是一种严格的语言，但在语言级别和 JDK 提供的类型中具有一些值得注意的惰性异常。

让我们来看看它们。

## 短路求值

Java 中提供了语言集成的惰性，采用逻辑运算符 `&&`（双与）和 `||`（双或）进行逻辑 *短路求值*。这些运算符从左到右评估其操作数，仅在必要时进行评估。如果逻辑表达式由运算符左侧的表达式满足，则根本不会评估右侧的操作数，如表 11-1 中所示。

表 11-1\. 逻辑短路运算符的评估

| 操作 | `leftExpr` 的值 | 是否评估 `rightExpr`？ |
| --- | --- | --- |
| `leftExpr && rightExpr` | `true` | 是 |
| `false` | 否 |
| `leftExpr &#124;&#124; rightExpr` | `true` | 否 |
| `false` | 是 |

# 位运算逻辑运算符

类似的位运算符 `&`（单与）和 `|`（单或）进行 *急切评估*，并且具有与其逻辑兄弟不同的目的。位运算符比较整数类型的单个位，导致整数结果。

尽管与控制结构类似，这些逻辑操作数不能单独存在。它们必须始终作为另一个语句的一部分，例如 `if`-block 的条件或变量赋值，如示例 11-2 所示。对于赋值的短路求值的另一个优势是它们创建（实际上）`final` 参考，使它们成为与 Java 的功能方法完美匹配的选择。

##### 示例 11-2\. 逻辑短路运算符的使用

```java
// WON'T COMPILE: unused result

left() || right();

// COMPILES: used as if condition

if (left() || right()) {
    // ...
}

// COMPILES: used as variable assignment

var result = left() || right();
```

如果表达式耗时或具有任何副作用，或者如果左侧无需评估则无需评估右侧表达式，则忽略右侧操作数的评估将非常有用。然而，它也可能是不评估所需表达式的来源，如果语句被短路，则需要的表达式在右侧。如果将它们作为决策的一部分，请务必仔细设计它们。

任何决策性代码都会因为纯函数而受益匪浅。预期行为简单直接，易于理解，没有任何潜在的副作用可能在重新设计或重构代码时被忽视，引入难以解决的微妙错误。您应该确保要么根本没有副作用，我认为这太绝对且通常是不现实的目标，要么命名您的方法以反映其后果。

## 控制结构

控制结构负责改变通过代码指令的路径。例如，`if`-`else`结构是一个条件分支，有一个（仅`if`）或多个（`if`-`else`）代码块。这些块根据其相应条件进行评估，这是一种惰性特性。在声明时严格评估`if`-`else`结构的任何部分将破坏其作为条件分支的目的。这种“怠惰例外于急切规则”的应用适用于所有分支和循环结构，如表 11-2 所列。

表 11-2\. Java 中的惰性结构

| 分支控制结构 | 循环结构 |
| --- | --- |

| `if`-`else` `? :`（三元运算符）

`switch`

`catch` | `for` `while`

`do-while` |

严格的非惰性控制结构语言很难想象，甚至可能不可能。

## JDK 中的惰性类型

到目前为止，我已经讲述了 Java 的惰性如何直接内建到语言中，以操作符和控制结构的形式。然而，JDK 还提供了多种内置类型和数据结构，在运行时也具有一定程度的惰性。

### 惰性映射

地图的常见任务是检查键是否已经有映射值，并在缺少时提供一个。相关的代码需要多次检查和非（有效地）`final`变量，如下所示：

```java
Map<String, User> users = ...;

var email = "john@doe.com";

var user = users.get(email);
if (user == null) {
  user = loadUser(email);
  users.put(email, user);
}
```

代码可能会因实际的`Map`实现而有所不同，但主要思想应该是清晰的。

一般来说，这已经是一种懒惰的方法，直到必要时才延迟加载用户。在 JDK 8 的多种类型中添加功能性增强的过程中，`Map`类型使用其`computeIf…​`方法获得了更简洁和功能性的替代方法。

根据键的映射值的存在性有两种可用的方法：

+   `V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction)`

+   `V computeIfPresent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)`

第一个方法是前一个示例代码的理想替代品，如下所示：

```java
Map<String, User> users = ...;

var email = "john@doe.com";

var user = users.computeIfAbsent(email,
                                 this::loadUser);
```

它需要所需的键作为其第一个参数，并且需要一个映射器`Function<K, V>`作为其第二个参数，如果键不存在，则为其提供新的映射值。`computeIfPresent`是仅在存在时重新映射值的对手。

还可以以`V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction)`方法的形式结合两种方法。它能够根据`remapping`函数的结果更新甚至删除映射值，如图 11-1 所示。

![Map#compute 进行延迟重映射](img/afaj_1101.png)

###### 图 11-1\. 使用 Map#compute 进行延迟重映射

函数式方法的一般主题在 Maps 的惰性添加中清晰可见。现在，不再需要编写冗长和重复的代码来 *处理* Map 及其映射值，而是可以集中精力于 *发生什么* 以及如何处理键和值。

### Streams

Java Streams 是惰性功能流水线的完美示例。您可以定义一个复杂的 Stream 框架，其中包含昂贵的功能操作，这些操作只有在调用终端操作后才开始评估。处理的元素数量完全取决于管道设计，通过将表达式的定义与其实际评估分开，在数据处理管道中最大限度地减少所需的工作。

第六章 详细解释了 Streams 及其对数据处理的惰性方法。

### Optionals

Optionals 是处理 `null` 值的一种非惰性方式。它们的一般方法类似于 Streams，但相对于 Streams，它们是严格评估的。有一些延迟操作可用，例如使用 `Supplier` 的 `T orElseGet(Supplier<? extends T> supplier)` 方法，将执行延迟到绝对必要时。

第九章详细介绍了 Optionals，以及如何使用它们的更多信息。

# Lambdas 和高阶函数

Lambdas 是在代码层面引入惰性的好方法。它们的声明是一个语句，因此是严格评估的。它们的主体——*单抽象方法*——封装了实际的逻辑，并在您的决定下进行评估。这使它们成为存储和传输表达式以供以后评估的简单方法。

让我们看看一些急切的代码，为方法提供参数，并展示如何利用 lambda 使其变为惰性。

## 一个急切的方法

在 示例 11-3 中，一个假想的 `User` 被一系列角色更新。这种更新并非总是发生，而是取决于 `update` 方法的内部逻辑。参数是 *急切* 提供的，需要通过 *DAO*⁠^(2) 进行昂贵的查找调用。

##### 示例 11-3\. 使用急切方法参数更新 `User`

```java
User updateUser(User user, List<Role> availableRoles) { ![1](img/1.png)
  // ...
}

// HOW TO USE

var user = loadUserById(23L);
var availableRoles = this.dao.loadAllAvailableRoles(); ![2](img/2.png)
var updatedUser = updateUser(user, availableRoles); ![3](img/3.png)
```

![1](img/#co_lazy_evaluation_CO1-1)

`updateUser` 方法需要 `user` 和所有可用角色的列表。更新本身依赖内部逻辑，可能实际上并不需要这些角色。

![2](img/#co_lazy_evaluation_CO1-2)

`loadAllAvailableRoles(user)` 被调用，无论 `updateUser` 方法是否需要角色。这导致了一个可能是不必要的昂贵的数据库访问。

![3](img/#co_lazy_evaluation_CO1-3)

所有参数在方法调用时已经评估完成。

即使在每种用例中角色并非必需，为 `updateUser` 提供角色会导致不必要的数据库调用和性能浪费。

那么如果这个调用并不总是必需的，你如何使它成为非强制性的呢？通过引入惰性。

## 更懒的方法

在像 Java 这样严格的语言中，所有方法参数都是提前提供的。即使一个参数实际上并不需要，方法也别无选择而必须接受它们。这在涉及执行昂贵的操作来预先创建这些参数（例如数据库调用）时尤为成问题，这可能会消耗您可用的资源和性能。

解决不必要的数据库调用的幼稚方法是将`updateUser`更改为直接接受 DAO，因此只有在必要时才能使用它：

```java
User updateUser(User user, DAO roleDAO) {
  // ...
}
```

现在`updateUser`方法具有加载可用角色所需的所有工具。从表面上看，非惰性数据访问的初始问题得到了解决，但这种“解决方案”却引发了一个新的问题：内聚性。

现在`updateUser`方法直接使用 DAO，并不再与角色获取方式隔离。这种方法会使方法变得不纯，因为访问数据库被视为副作用，并使得验证和测试变得更加困难。如果`updateUser`方法根本不知道 DAO 类型，可能的 API 边界会使情况变得更加复杂。因此，您需要创建另一个抽象来检索角色。与创建一个额外的抽象层来弥合 DAO 和`updateUser`方法之间的差距不同，您可以将`updateUser`变成高阶函数并接受一个 lambda 表达式。

## 一个功能性的方法

要为在示例 11-3 中检索所需用户角色的功能抽象化，首先必须将问题分解为更抽象的表示，找出实际需要作为参数的内容，而不是如何获得参数值。

`updateUser`方法需要访问可用角色，这反映在原始方法签名中。这正是在代码中引入惰性将为您提供最灵活的解决方案的地方。

`Supplier<T>`类型是封装某些逻辑以按您的意愿检索值的最低级别可能性。与直接向`updateUser`提供 DAO 不同，lambda 表达式是惰性中间构造，用于加载角色，正如示例 11-4 中所见。

##### 示例 11-4\. 使用 lambda 更新`User`

```java
void updateUser(User user, Supplier<List<Role>> availableRolesFn) { ![1](img/1.png)
  // ...

  var availableRoles = availableRolesFn.get();

  // ...
}

// HOW TO USE

var user = loadUserById(23L);

updateUser(user, this.dao::loadAllAvailableRoles); ![2](img/2.png)
```

![1](img/#co_lazy_evaluation_CO2-1)

`updateUser`方法签名必须更改为接受`Supplier<List<Role>>`，而不是已加载的`List<Role>`或 DAO 本身。

![2](img/#co_lazy_evaluation_CO2-2)

现在如何获取角色的逻辑已封装在一个方法引用中。

通过接受`Supplier`将`updateUser`变成高阶函数，可以创建一个表面上的新层，而无需额外的自定义类型来包装角色加载过程。

直接将`DAO`作为参数使用可以消除这些不利因素：

+   `DAO`和`updateUser`方法之间不再存在连接，从而可能产生一个纯粹的、无副作用的方法。

+   你不需要额外的类型来表示这个抽象。已经可用的`Supplier<T>`函数接口是可能的最简单和最兼容的抽象形式。

+   可以恢复测试性，而不需要可能复杂的 DAO 的模拟。

昂贵的操作，比如数据库查询，如果调用可以避免，可以极大地从延迟执行中受益。但这并不意味着，没有真正的需要，就将所有方法参数都延迟执行是正确的方法。还有其他解决方案，比如缓存昂贵调用的结果，可能比设计方法调用来接受延迟参数更简单。

# 使用**Thunk**进行延迟执行

Lambda 表达式是封装表达式以供以后评估的一种简单而低级的方式。不过，有一件事情是缺失的，那就是在评估后存储结果 — *记忆化* — 这样如果调用两次就不需要重新评估表达式了。有一种简单的方法可以弥补这个缺失：**Thunk**。

一个 **Thunk** 是一个对计算进行包装的延迟执行，直到需要结果为止。与`Supplier<T>`不同，后者也延迟执行计算，但 **Thunk** 仅在第一次评估后直接返回结果，并在后续调用中不再进行评估。

**Thunk** 属于*延迟加载/初始化*的一般类别，这是在面向对象的代码中经常发现的设计模式。延迟加载和延迟初始化两种技术都是实现相同目标的类似机制：非严格评估和缓存结果。`Supplier<T>`只是延迟评估，而 **Thunk** 也缓存其结果。

让我们创建一个简单的 **Thunk** ，遵循*虚拟代理*设计模式^(3)以成为`Supplier<T>`的可替换解决方案。

## 创建一个简单的 **Thunk**

最简单的方法是包装一个`Supplier<T>`实例，并在第一次评估后存储其结果。通过还实现`Supplier<T>`接口， **Thunk** 变成了一个可替换的解决方案，如示例 11-5 所示。

##### 示例 11-5\. 一个简单的`Thunk<T>`

```java
public class Thunk<T> implements Supplier<T> { ![1](img/1.png)

  private final Supplier<T> expression; ![2](img/2.png)

  private T result; ![3](img/3.png)

  private Thunk(Supplier<T> expression) {
    this.expression = expression;
  }

  @Override
  public T get() {
    if (this.result == null) { ![4](img/4.png)
      this.result = this.expression.get();
    }
    return this.result;
  }

  public static <T> Thunk<T> of(Supplier<T> expression) { ![5](img/5.png)
    if (expression instanceof Thunk<T>) { ![6](img/6.png)
      return (Thunk<T>) expression;
    }

    return new Thunk<T>(expression);
  }
}
```

![1](img/#co_lazy_evaluation_CO3-1)

`Thunk<T>`实现了`Supplier<T>`以充当一种可替换的解决方案。

![2](img/#co_lazy_evaluation_CO3-2)

实际的`Supplier<T>`需要存储以延迟评估。

![3](img/#co_lazy_evaluation_CO3-3)

必须在评估后存储结果。

![4](img/#co_lazy_evaluation_CO3-4)

如果尚未评估，则解析表达式，并存储其结果。

![5](img/#co_lazy_evaluation_CO3-5)

一个方便的工厂方法，用于创建一个`Thunk`，而不需要`new`或泛型类型信息，所以唯一的构造函数可以是`private`。

![6](img/#co_lazy_evaluation_CO3-6)

不需要为`Thunk<T>`创建一个`Thunk<T>`。

这种 Thunk 实现简单而强大。通过使用任何`Supplier<T>`调用工厂方法添加记忆功能，以创建一个可替换的组件。像前一节中更新`User`那样，需要将方法引用包装在`Thunk.of`方法中：

```java
updateUser(user, Thunk.of(this.dao::loadAllAvailableRoles));
```

对于`Thunk<T>`的功能性增强并不止于此。您可以轻松添加“粘合方法”，正如我在第二章中讨论的那样，以支持函数组合，如 Example 11-6 所示。

##### 示例 11-6。对`Thunk<T>`的功能性增强

```java
public class Thunk<T> implements Supplier<T> {

  // ...

  public static <T> Thunk<T> of(T value) { ![1](img/1.png)
    return new Thunk<T>(() -> value);
  }

  public <R> Thunk<R> map(Function<T, R> mapper) { ![2](img/2.png)
    return Thunk.of(() -> mapper.apply(get()));
  }

  public <R> Thunk<R> flatMap(Function<T, Thunk<R>> mapper) { ![3](img/3.png)
    return Thunk.of(() -> mapper.apply(get()).get());
  }

  public void accept(Consumer<T> consumer) { ![4](img/4.png)
    consumer.accept(get());
  }
}
```

![1](img/#co_lazy_evaluation_CO4-1)

创建一个`Thunk<T>`的工厂方法，用于创建单个值，而不是`Supplier<T>`。

![2](img/#co_lazy_evaluation_CO4-2)

创建一个包括`mapper`函数的新的`Thunk<R>`。

![3](img/#co_lazy_evaluation_CO4-3)

从返回 a+Thunk<T>+的函数创建一个新的`Thunk<R>`，而不需要无谓地将其包装在另一个`Thunk`中。

![4](img/#co_lazy_evaluation_CO4-4)

消耗一个 Thunk 的结果。

通过添加“粘合”方法，`Thunk<T>`类型变得更加多功能，用于创建单表达式的惰性管道。

然而，还存在一个一般性问题：*线程安全*。

## 线程安全的 Thunk

对于单线程环境，在前一节中讨论的`Thunk<T>`实现可以正常工作。但是，如果在表达式评估时从另一个线程访问它，可能会导致竞态条件重新评估。唯一可以防止这种情况发生的方法是在所有访问线程中同步它。

最直接的方法是将关键字`synchronized`添加到其`get`方法中。然而，这样做的明显缺点是*始终*需要`synchronized`访问以及相关的开销，即使评估已经完成。同步可能不像过去那样慢，但对于每次调用`get`方法来说仍然是一种开销，并且肯定会不必要地减慢代码速度。

那么，您如何改变实现以消除竞态条件，同时又尽可能不影响总体性能？您需要对*何时*和*何处*竞态条件可能发生进行风险分析。

评估相关的竞态条件风险仅存在于表达式评估之前。之后，不会发生双重评估，因为结果已返回。这使您只需同步评估本身，而不是每次调用`get`方法。

Example 11-7 展示了引入专门的和`synchronized`的`evaluate`方法。将会很快解释它的实际实现及如何访问其结果。

##### 示例 11-7。带有同步评估的`Thunk<T>`

```java
public class Thunk<T> implements Supplier<T> {

  private Thunk(Supplier<T> expression) {
    this.expression = () -> evaluate(expression);
  }

  private synchronized T evaluate(Supplier<T> expression) {
    // ...
  }

  // ...
}
```

先前版本的`Thunk`使用了额外的`value`字段来确定`expression`是否已经评估。然而，新的线程安全变体用专用的抽象替换了存储的`value`及其检查，其持有值，如下所示：

```java
private static class Holder<T> implements Supplier<T> {

  private final T value;

  Holder(T value) {
    this.value = value;
  }

  @Override
  public T get() {
    return this.value;
  }
}
```

`Holder<T>`执行两件事：

+   持有评估值

+   实现`Supplier<T>`

由于作为`expression`字段的插拔式替换，一种称为*比较与交换*（CAS）的技术得以实现。它用于设计并发算法，通过将变量的值与期望值进行比较，如果它们相等，则将该值交换为新值。该操作必须是*原子的*，这意味着访问底层数据要么完全成功，要么完全失败。这就是为什么`evaluate`方法必须是`synchronized`的原因。任何线程都可以在评估之前或之后看到数据，但从不会在评估过程中，从而消除竞态条件。

在示例 11-8 中，您可以看到`evaluate`的 CAS 实现。

现在，`private`字段`+expression`可以被新类型替换，如示例 11-7 所示。

##### 示例 11-8。使用`Holder<T>`而不是`Supplier<T>`

```java
public class Thunk<T> implements Supplier<T> {

  private static class Holder<T> implements Supplier<T> {
    // ...
  }

  private Supplier<T> holder; ![1](img/1.png)

  private Thunk(Supplier<T> expression) {
    this.holder = () -> evaluate(expression);
  }

  private synchronized T evaluate(Supplier<T> expression) {
    if (Holder.class.isInstance(this.holder) == false) { ![2](img/2.png)
      var evaluated = expression.get();
      this.holder = new Holder<>(evaluated); ![3](img/3.png)
    }
    return this.holder.get();
  }

  @Override
  public T get() {
    return this.holder.get(); ![4](img/4.png)
  }
}
```

![1](img/#co_lazy_evaluation_CO5-1)

该字段被重命名以更好地反映其用途，并且在表达式评估后也被设置为非`final`。

![2](img/#co_lazy_evaluation_CO5-2)

只有当`holder`字段当前不是`Holder`实例时，表达式才会被评估，而是在构造函数中创建的表达式。

![3](img/#co_lazy_evaluation_CO5-3)

在这一点上，`holder`字段持有原始的 lambda 表达式以评估初始表达式，并将其替换为带有评估结果的`Holder`实例。

![4](img/#co_lazy_evaluation_CO5-4)

非`synchronized`的`get`方法直接使用`holder`字段访问值，因为它总是引用一个`Supplier`。

改进后的`Thunk<T>`实现不再像以前那样简单，但通过将表达式的评估与访问解耦，消除了竞态条件。

首次访问时，`holder`字段将调用`synchronized`的`evaluate`方法，因此是线程安全的。在评估表达式时，任何额外的调用也将调用`evaluate`方法。而不是重新评估，`holder`字段的类型检查直接跳过，返回`this.holder.get()`的结果。在重新分配`holder`之后的任何访问都将完全跳过任何`synchronized`。

就这样，您现在拥有一个线程安全的、惰性评估的`Supplier<T>`插入，只评估一次。

我们的`Thunk`实现使用`synchronized`，但有多种实现*比较与交换*算法的方法。可以使用 JDK 中的`java.util.concurrent.atomic.Atomic…​`类型之一，或者甚至使用`ConcurrentHashMap#computeIfAbsent`来防止竞态条件。Brian Goetz 的书籍“Java Concurrency”^(4)为更好理解原子变量、非阻塞同步和 Java 的并发模型提供了一个良好的起点。

# 惰性的最终思考

本质上，惰性的核心思想是推迟所需的工作，直到它变得不可或缺的时刻。将*创建*和*消耗*表达式分离给了您的代码一个新的模块化轴线。如果一个操作是可选的，并且不是每种用例都需要，这种方法可以极大地提高性能。然而，惰性评估也意味着您必须放弃对评估确切时间的某种程度控制。

感知和实际*控制丧失*使得对代码所需性能和内存特性的推理变得更加困难。总体性能需求是所有评估部分的总和。急切评估允许相当线性和组合性能评估。惰性将实际的计算成本从表达式定义的地方转移到使用它们的时候，有可能根本不运行代码。这就是为什么习惯性的惰性性能更难评估，因为与急切评估相比，感知的性能可能会立即改善，特别是如果您的代码有许多昂贵但可能是可选的代码路径。总体性能需求可能会根据一般情况和实际评估的代码而变化。您必须分析您的惰性代码的“平均”使用模式，并在不同场景下估计所需的性能特性，使得直接的基准测试变得非常困难。

软件开发是一个持续的战斗，要*有效利用稀缺资源*以达到所需或必需的性能。像延迟评估或数据处理中的流这样的惰性技术，是改进代码性能的低成本^(5)，易于集成到现有代码库中的有效方法。它绝对会将所需工作降至最低，甚至可能为其他任务释放宝贵的性能。如果可以避免某些表达式或昂贵的计算，将其设为惰性绝对是一项值得长远努力的事业。

# 总结

+   严格评估意味着表达式和方法参数在声明时立即评估。

+   惰性评估通过推迟或甚至完全不评估表达式的结果，将*创建*和*消耗*表达式分离，从而实现。

+   *严格性*是关于“做事情”; *惰性*是关于“考虑要做的事情”。

+   Java 在表达式和方法参数方面是一种“严格”的语言，尽管存在某些 *惰性* 操作符和控制结构。

+   Lambas 封装表达式，使它们成为可在您自行决定时进行评估的惰性包装器。

+   JDK 中有多个懒加载运行时结构和辅助方法。例如，流（Streams）是惰性的功能管道，`Optional` 和 `Map` 在其一般接口中提供了 *惰性* 的扩展。

+   `Supplier<T>` 接口是创建延迟计算的最简单方式。

+   Memoization，以 `Thunk` 的形式，有助于避免重新评估，可以用作 `Supplier<T>` 的即插即用替代方案。

+   懒加载是性能优化的重要手段。最好的代码是根本不运行的代码。其次好的选择是仅在“按需”时运行它。

+   对于延迟代码的性能需求评估较为困难，如果在不符合“真实世界”使用情况的环境中进行测试，可能会掩盖性能问题。

^(1) 参见“有效地 final”以了解 *有效* `final` 变量的定义和要求。

^(2) *DAO*（数据访问对象）是一种模式，提供了与数据库等持久化层的抽象接口。它将应用程序调用转换为对底层持久化层特定操作的操作，而不会暴露其详细信息。

^(3) 维基百科关于[代理模式](https://zh.wikipedia.org/wiki/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F)提供了不同种类代理及其用途的概述。

^(4) Brian Goetz. 2006 年. “Java 并发编程实践.” Addison-Wesley. ISBN 978-0321349606.

^(5) “低 hanging fruit” 的概念描述了一个易于实现或利用的目标，相比于重新设计或重构整个代码库的替代方案。
