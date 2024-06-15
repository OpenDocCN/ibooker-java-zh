# 第五章。业务规则引擎

# 挑战

您的业务现在非常顺利。事实上，您现在已经扩展到一个拥有成千上万名员工的组织。这意味着您已经雇佣了许多人来从事不同的业务职能：市场营销、销售、运营、管理、会计等等。您意识到所有业务职能都需要创建根据某些条件触发操作的规则；例如，“如果潜在客户的职位是'CEO'，则通知销售团队”。您可以要求技术团队为每个新需求实现定制软件，但是您的开发人员正在忙于其他产品。为了鼓励业务团队和技术团队之间的协作，您决定开发一个业务规则引擎，这将使开发人员和业务团队能够共同编写代码。这将使您能够提高生产力并减少实施新规则所需的时间，因为您的业务团队将能够直接做出贡献。

# 目标

在本章中，您将首先学习如何使用测试驱动开发方法来解决新设计问题。您将了解到一个称为模拟的技术概述，这将有助于指定单元测试。然后，您将学习一些 Java 中的现代特性：局部变量类型推断和 switch 表达式。最后，您将学习如何使用建造者模式和接口隔离原则开发友好的 API。

###### 注意

如果您想随时查看本章的源代码，可以查看书籍代码库中的`com.iteratrlearning.shu_book.chapter_05`包。

# 业务规则引擎需求

在开始之前，让我们考虑一下您想要实现的目标。您希望使非程序员能够在其自己的工作流程中添加或更改业务逻辑。例如，市场营销执行人员可能希望在潜在客户询问您的产品并符合某些条件时提供特别折扣。会计主管可能希望在支出异常高时创建警报。这些都是业务规则引擎可以实现的例子。它实质上是执行一个或多个业务规则的软件，通常使用简单的定制语言声明这些规则。业务规则引擎可以支持多个不同的组件：

事实

规则可以访问的可用信息

行动

您要执行的操作

条件

这些指定何时触发操作

规则

这些指定您希望执行的业务逻辑，实质上是将事实、条件和操作分组在一起

业务规则引擎的主要生产力优势在于它使规则能够在一个地方进行维护、执行和测试，而无需与主应用程序集成。

###### 注意

有许多成熟的 Java 业务规则引擎，比如[Drools](https://www.drools.org)。通常这样的引擎符合诸如*决策建模和标记*（DMN）的标准，并配备一个集中的规则库，一个使用*图形用户界面*（GUI）的编辑器和可视化工具，以帮助维护复杂的规则。在本章中，您将开发一个业务规则引擎的最小可行产品，并对其进行迭代，以改进其功能和可访问性。

# 测试驱动开发

从哪里开始？需求并不是一成不变的，预计会不断演变，因此您开始时只需列出用户需要完成的基本功能即可：

+   添加一个动作

+   运行动作

+   基本报告

这在示例 5-1 中的基本 API 中有所体现。每个方法抛出`UnsupportedOperationException`，表示它尚未实现。

##### 示例 5-1\. 业务规则引擎的基本 API

```java
public class BusinessRuleEngine {

    public void addAction(final Action action) {
        throw new UnsupportedOperationException();
    }

    public int count() {
        throw new UnsupportedOperationException();
    }

    public void run() {
        throw new UnsupportedOperationException();
    }

}
```

动作简单地是将要执行的代码片段。我们可以使用`Runnable`接口，但引入一个单独的接口`Action`更能代表手头的领域。`Action`接口将允许业务规则引擎与具体动作解耦。由于`Action`接口只声明了一个抽象方法，我们可以将其注释为功能接口，如示例 5-2 所示。

##### 示例 5-2\. 动作接口

```java
@FunctionalInterface
public interface Action {
   void execute();
}
```

接下来怎么办？现在是时候真正写些代码了——实现在哪里？你将使用一种称为*测试驱动开发*（TDD）的方法。TDD 的哲学是首先编写一些测试，这些测试将指导你编写代码的实现。换句话说，你先写测试，再写实现。这有点像迄今为止你所做的相反：你先为一个需求写了完整的代码，然后测试它。现在你会更多地关注测试。

## 为什么使用 TDD？

为什么要采用这种方法？有几个好处：

+   逐个编写测试将帮助您专注并完善需求，通过逐个正确实现一件事情来实现。

+   这是确保代码有关联组织的一种方式。例如，通过先写测试，你需要仔细考虑代码的公共接口。

+   随着你按需求迭代，构建全面的测试套件，这既增加了你符合需求的信心，也减少了 bug 的范围。

+   你不会写不需要的代码（过度工程），因为你只是写通过测试的代码。

## TDD 循环

TDD 方法大致包括以下循环步骤，如图 5-1 所示：

1.  写一个失败的测试

1.  运行所有测试

1.  使实现生效

1.  运行所有测试

![TDD 循环](img/rwsd_0501.png)

###### 图 5-1\. TDD 循环

在实践中，作为这个过程的一部分，你必须持续*重构*你的代码，否则它将变得难以维护。此时，当你引入变更时，你知道你有一套可以依赖的测试套件。图 5-2 展示了这一改进的 TDD 过程。

![TDD 循环与重构](img/rwsd_0502.png)

###### 图 5-2\. TDD 与重构

在 TDD 的精神下，让我们首先编写我们的第一个测试来验证`addActions`和`count`的行为是否正确，如示例 5-3 所示。

##### 示例 5-3\. 业务规则引擎的基本测试

```java
@Test
void shouldHaveNoRulesInitially() {
    final BusinessRuleEngine businessRuleEngine = new BusinessRuleEngine();

    assertEquals(0, businessRuleEngine.count());
}

@Test
void shouldAddTwoActions() {
    final BusinessRuleEngine businessRuleEngine = new BusinessRuleEngine();

    businessRuleEngine.addAction(() -> {});
    businessRuleEngine.addAction(() -> {});

    assertEquals(2, businessRuleEngine.count());
}
```

在运行测试时，你会看到它们失败，并显示`UnsupportedOperationException`，如图 5-3 所示。

![失败的测试](img/rwsd_0503.png)

###### 图 5-3\. 失败的测试

所有测试都失败了，但没关系。这给了我们一个可重现的测试套件，将指导代码的实现。现在可以添加一些实现代码，如示例 5-4 所示。

##### 示例 5-4\. 业务规则引擎的基本实现

```java
public class BusinessRuleEngine {

    private final List<Action> actions;

    public BusinessRuleEngine() {
        this.actions = new ArrayList<>();
    }

    public void addAction(final Action action) {
        this.actions.add(action);
    }

    public int count() {
        return this.actions.size();
    }

    public void run(){
        throw new UnsupportedOperationException();
    }
}
```

现在你可以重新运行测试，它们通过了！但是，还有一个关键操作缺失。我们如何为`run`方法编写测试？不幸的是，`run()`不返回任何结果。我们将需要一种称为*模拟*的新技术，以验证`run()`方法的正确操作。

# 模拟

模拟是一种技术，它允许你验证当执行`run()`方法时，业务规则引擎中添加的每个动作是否确实被执行。目前很难做到这一点，因为`BusinessRuleEngine`中的`run()`方法和`Action`中的`perform()`方法都返回`void`。我们无法编写断言！模拟在第六章中有详细介绍，但现在你将会得到一个简要概述，这样你就能继续编写测试了。你将使用 Mockito，这是一个流行的 Java 模拟库。在其最简单的形式下，你可以做两件事情：

1.  创建一个模拟对象。

1.  验证方法是否被调用。

那么，如何开始呢？你需要首先导入这个库：

```java
import static org.mockito.Mockito.*;
```

这个导入允许你使用`mock()`和`verify()`方法。静态方法`mock()`允许你创建一个模拟对象，然后你可以验证某些行为是否发生。方法`verify()`允许你设置断言，即特定方法是否被调用。示例 5-5 展示了一个例子。

##### 示例 5-5\. 模拟并验证与`Action`对象的交互

```java
@Test
void shouldExecuteOneAction() {
        final BusinessRuleEngine businessRuleEngine = new BusinessRuleEngine();
        final Action mockAction = mock(Action.class);

        businessRuleEngine.addAction(mockAction);
        businessRuleEngine.run();

        verify(mockAction).perform();
}
```

单元测试为`Action`创建了一个模拟对象。这通过将类作为参数传递给`mock`方法来实现。接下来是测试的*when*部分，在这里你调用行为。这里我们添加了动作并执行了`run()`方法。最后，单元测试的*then*部分设置了断言。在这种情况下，我们验证了`Action`对象上的`perform()`方法是否被调用。

如果你运行这个测试，正如预期的那样会失败，并显示 `UnsupportedOperationException`。如果 `run()` 方法体为空会发生什么？你将收到新的异常跟踪：

```java
Wanted but not invoked:
action.perform();
-> at BusinessRuleEngineTest.shouldExecuteOneAction(BusinessRuleEngineTest.java:35)
Actually, there were zero interactions with this mock.
```

这个错误来自 Mockito，并告诉你 `perform()` 方法从未被调用。现在是时候为 `run()` 方法编写正确的实现了，如 示例 5-6 所示。

##### 示例 5-6\. `run()` 方法的实现

```java
public void run() {
    this.actions.forEach(Action::perform);
}
```

重新运行测试，你会看到测试通过了。Mockito 能够验证当业务规则引擎运行时，`Action` 对象的 `perform()` 方法是否被调用。Mockito 允许你指定复杂的验证逻辑，比如方法应该被调用多少次，带有特定参数等。你将在 第六章 中了解更多相关信息。

# 添加条件

你必须承认，到目前为止，业务规则引擎的功能相当有限。你只能声明简单的动作。然而，在实践中，业务规则引擎的使用者需要根据某些条件执行动作。这些条件将依赖于一些事实。例如，仅当潜在客户的职位是 CEO 时，通知销售团队。

## 建模状态

你可以先编写代码，添加一个动作，并使用匿名类引用本地变量，如 示例 5-7 所示，或者使用 lambda 表达式，如 示例 5-8 所示。

##### 示例 5-7\. 使用匿名类添加一个动作

```java
// this object could be created from a form
final Customer customer = new Customer("Mark", "CEO");

businessRuleEngine.addAction(new Action() {

    @Override
    public void perform() {
        if ("CEO".equals(customer.getJobTitle())) {
            Mailer.sendEmail("sales@company.com", "Relevant customer: " + customer);
        }
    }
});
```

##### 示例 5-8\. 使用 lambda 表达式添加一个动作

```java
// this object could be created from a form
final Customer customer = new Customer("Mark", "CEO");

businessRuleEngine.addAction(() -> {
    if ("CEO".equals(customer.getJobTitle())) {
        Mailer.sendEmail("sales@company.com", "Relevant customer: " + customer);
    }
});
```

然而，出于几个原因，这种方法并不方便：

1.  如何测试这个动作？它不是一个独立的功能模块；它对 `customer` 对象有硬编码的依赖。

1.  `customer` 对象没有与动作分组。它是一种外部状态，被共享使用，导致责任混淆。

那么我们需要什么？我们需要封装状态，使其可供业务规则引擎中的动作使用。让我们通过引入一个名为 `Facts` 的新类来建模这些需求，`Facts` 将代表业务规则引擎中可用的状态，并且更新 `Action` 接口，使其能够操作 `Facts`。一个更新后的单元测试显示在 示例 5-9 中。该单元测试检查当业务规则引擎运行时，指定的动作是否确实被调用，并且传递了 `Facts` 对象作为参数。

##### 示例 5-9\. 使用事实测试一个动作

```java
@Test
public void shouldPerformAnActionWithFacts() {
    final Action mockAction = mock(Action.class);
    final Facts mockFacts = mock(Facts.class);
    final BusinessRuleEngine businessRuleEngine = new BusinessRuleEngine(mockedFacts);

    businessRuleEngine.addAction(mockAction);
    businessRuleEngine.run();

    verify(mockAction).perform(mockFacts);
}
```

为了遵循 TDD 哲学，此测试最初将失败。您始终需要运行测试以确保它们失败，否则可能会意外通过一个测试。要使测试通过，您需要更新 API 和实现代码。首先，您将引入`Facts`类，它允许您存储以键和值表示的事实。引入一个单独的`Facts`类来建模状态的好处是，您可以通过提供公共 API 控制用户可用的操作，并对类的行为进行单元测试。目前，`Facts`类仅支持`String`键和`String`值。`Facts`类的代码显示在示例 5-10 中。我们选择名称`getFact`和`addFact`，因为它们更好地表示手头的领域（处理事实），而不是`getValue`和`setValue`。

##### 示例 5-10\. Facts 类

```java
public class Facts {

    private final Map<String, String> facts = new HashMap<>();

    public String getFact(final String name) {
        return this.facts.get(name);
    }

    public void addFact(final String name, final String value) {
        this.facts.put(name, value);
    }
}
```

现在，您需要重构`Action`接口，以便`perform()`方法可以使用作为参数传递的`Facts`对象。这样一来，清楚地表明了在单个`Action`的上下文中可用的事实（示例 5-11）。

##### 示例 5-11\. 接受事实的行动接口

```java
@FunctionalInterface
public interface Action {
    void perform(Facts facts);
}
```

最后，您现在可以更新`BusinessRuleEngine`类，以利用事实和更新的`Action`的`perform()`方法，如示例 5-12 所示。

##### 示例 5-12\. 带有事实的 BusinessRuleEngine

```java
public class BusinessRuleEngine {

    private final List<Action> actions;
    private final Facts facts;

    public BusinessRuleEngine(final Facts facts) {
        this.facts = facts;
        this.actions = new ArrayList<>();
    }

    public void addAction(final Action action) {
        this.actions.add(action);
    }

    public int count() {
        return this.actions.size();
    }

    public void run() {
        this.actions.forEach(action -> action.perform(facts));
    }
}
```

现在`Facts`对象可用于行动，您可以在代码中指定查找`Facts`对象的任意逻辑，如示例 5-13 所示。

##### 示例 5-13\. 利用事实的行动

```java
businessRuleEngine.addAction(facts -> {
    final String jobTitle = facts.getFact("jobTitle");
    if ("CEO".equals(jobTitle)) {
        final String name = facts.getFact("name");
        Mailer.sendEmail("sales@company.com", "Relevant customer: " + name);
    }
});
```

让我们看一些更多的示例。这也是介绍 Java 中两个最近功能的好机会，我们按顺序探索：

+   局部变量类型推断

+   Switch 表达式

## 局部变量类型推断

Java 10 引入了局部变量类型推断。类型推断是编译器可以为您确定静态类型，因此您无需输入它们的想法。在示例 5-10 中，您在之前看到了类型推断的示例，当您编写时

```java
Map<String, String> facts = new HashMap<>();
```

而不是

```java
Map<String, String> facts = new HashMap<String, String>();
```

这是 Java 7 中引入的一个特性，称为 *菱形操作符*。基本上，当其上下文确定类型参数（在本例中为 `String, String`）时，您可以省略泛型的类型参数。在前面的代码中，赋值的左侧指示`Map`的键和值应为 `String`。

自 Java 10 起，类型推断已扩展到局部变量上。例如，示例 5-14 中的代码可以使用`var`关键字和局部变量类型推断进行重写，如示例 5-15 所示。

##### 示例 5-14\. 显式类型声明的局部变量声明

```java
Facts env = new Facts();
BusinessRuleEngine businessRuleEngine = new BusinessRuleEngine(env);
```

##### 示例 5-15\. 局部变量类型推断

```java
var env = new Facts();
var businessRuleEngine = new BusinessRuleEngine(env);
```

通过在 Example 5-15 中显示的代码中使用`var`关键字，变量`env`仍具有静态类型`Facts`，变量`businessRuleEngine`仍具有静态类型`BusinessRuleEngine`。

###### 注意

使用`var`关键字声明的变量不是`final`。例如，以下代码：

```java
final Facts env = new Facts();
```

不严格等同于：

```java
var env = new Facts();
```

在使用`var`声明后，仍然可以为变量`env`分配另一个值。您必须在变量`env`前显式添加`final`关键字，如下所示：

```java
final var env = new Facts()
```

在其余章节中，出于简洁性考虑，我们简单使用`var`关键字，不使用`final`。当我们显式声明变量类型时，我们使用`final`关键字。

类型推断有助于减少编写 Java 代码所需的时间。然而，您应该始终使用这个特性吗？值得记住的是，开发人员花费的时间更多是在阅读代码而不是编写代码。换句话说，您应该考虑优化阅读的便利性而不是编写的便利性。`var`改善这一点的程度总是主观的。您应该始终专注于帮助您的团队成员阅读您的代码，因此，如果他们乐意使用`var`阅读代码，那么您应该使用它，否则不要使用。例如，我们可以重构 Example 5-13 中的代码，使用本地变量类型推断来整理代码，如 Example 5-16 所示。

##### Example 5-16\. 使用事实和本地变量类型推断的操作

```java
businessRuleEngine.addAction(facts -> {
    var jobTitle = facts.getFact("jobTitle");
    if ("CEO".equals(jobTitle)) {
        var name = facts.getFact("name");
        Mailer.sendEmail("sales@company.com", "Relevant customer: " + name);
    }
});
```

## Switch 表达式

到目前为止，您只设置了处理一个条件的操作。这相当受限制。例如，假设您与销售团队合作。他们可能在他们的*客户关系管理*（CRM）系统中记录具有不同金额和不同阶段的不同交易。交易阶段可以表示为枚举`Stage`，其值包括`LEAD`、`INTERESTED`、`EVALUATING`、`CLOSED`，如 Example 5-17 所示。

##### Example 5-17\. 枚举表示不同的交易阶段

```java
public enum Stage {
    LEAD, INTERESTED, EVALUATING, CLOSED
}
```

根据交易阶段，您可以分配一个规则，给出您赢得交易的概率。因此，您可以帮助销售团队生成预测。例如，对于特定团队，`LEAD` 阶段的转化概率为 20%，那么一个金额为 1000 美元的`LEAD`阶段的交易将有一个预测金额为 200 美元。让我们创建一个操作来建模这些规则，并返回一个特定交易的预测金额，如 Example 5-18 所示。

##### Example 5-18\. 计算特定交易预测金额的规则

```java
businessRuleEngine.addAction(facts -> {
    var forecastedAmount = 0.0;
    var dealStage = Stage.valueOf(facts.getFact("stage"));
    var amount = Double.parseDouble(facts.getFact("amount"));
    if(dealStage == Stage.LEAD){
        forecastedAmount = amount * 0.2;
    } else if (dealStage == Stage.EVALUATING) {
        forecastedAmount = amount * 0.5;
    } else if(dealStage == Stage.INTERESTED) {
        forecastedAmount = amount * 0.8;
    } else if(dealStage == Stage.CLOSED) {
        forecastedAmount = amount;
    }
    facts.addFact("forecastedAmount", String.valueOf(forecastedAmount));
});
```

Example 5-18 中显示的代码基本上为每个枚举值提供一个值。更优选的语言构造是`switch`语句，因为它更简洁。这在 Example 5-19 中展示。

##### 示例 5-19\. 使用`switch`语句计算特定交易预测金额的规则

```java
switch (dealStage) {
    case LEAD:
        forecastedAmount = amount * 0.2;
        break;
    case EVALUATING:
        forecastedAmount = amount * 0.5;
        break;
    case INTERESTED:
        forecastedAmount = amount * 0.8;
        break;
    case CLOSED:
        forecastedAmount = amount;
        break;
}
```

注意示例 5-19 中代码中的所有`break`语句。`break`语句确保不执行`switch`语句中的下一个块。如果您不小心忘记了`break`，则代码仍然会编译，并且会出现所谓的*穿透*行为。换句话说，将执行下一个块，这可能导致微妙的错误。自 Java 12 起（使用语言功能预览模式），您可以通过使用不同的语法来重写此以避免穿透行为和多个`break`，来使用`switch`作为表达式，如示例 5-20 所示。

##### 示例 5-20\. 没有穿透行为的`switch`表达式

```java
var forecastedAmount = amount * switch (dealStage) {
    case LEAD -> 0.2;
    case EVALUATING -> 0.5;
    case INTERESTED -> 0.8;
    case CLOSED -> 1;
}
```

除了增加的可读性外，这种增强的`switch`形式还有一个好处是*穷尽性*。这意味着当您使用`switch`与枚举时，Java 编译器会检查所有枚举值是否有对应的`switch`标签。例如，如果您忘记处理`CLOSED`情况，Java 编译器将产生以下错误：

```java
error: the switch expression does not cover all possible input values.
```

可以像在示例 5-21 中展示的那样，使用`switch`表达式重新编写整体操作。

##### 示例 5-21\. 用于计算特定交易预测金额的规则

```java
businessRuleEngine.addAction(facts -> {
    var dealStage = Stage.valueOf(facts.getFact("stage"));
    var amount = Double.parseDouble(facts.getFact("amount"));
    var forecastedAmount = amount * switch (dealStage) {
        case LEAD -> 0.2;
        case EVALUATING -> 0.5;
        case INTERESTED -> 0.8;
        case CLOSED -> 1;
    }
    facts.addFact("forecastedAmount", String.valueOf(forecastedAmount));
});
```

## 接口隔离原则

现在我们想开发一个*检查工具*，允许业务规则引擎的用户检查可能的动作和条件的状态。例如，我们希望评估每个动作和相关条件，以便记录它们而不实际执行动作。我们该如何做呢？当前的`Action`接口不够用，因为它没有区分执行的代码与触发该代码的条件。目前没有办法将条件与动作代码分开。为了弥补这一点，我们可以引入一个增强的`Action`接口，其中内置了评估条件的功能。例如，我们可以创建一个名为`ConditionalAction`的接口，其中包括一个新方法`evaluate()`，如示例 5-22 所示。

##### 示例 5-22\. ConditionalAction 接口

```java
public interface ConditionalAction {
    boolean evaluate(Facts facts);
    void perform(Facts facts);
}
```

现在我们可以实现一个基本的`Inspector`类，它接受一组`ConditionalAction`对象并根据某些事实对它们进行评估，如示例 5-23 所示。`Inspector`返回一个报告列表，其中包含事实、条件动作和结果。`Report`类的实现如示例 5-24 所示。

##### 示例 5-23\. 条件检查器

```java
public class Inspector {

    private final List<ConditionalAction> conditionalActionList;

    public Inspector(final ConditionalAction...conditionalActions) {
        this.conditionalActionList = Arrays.asList(conditionalActions);
    }

    public List<Report> inspect(final Facts facts) {
        final List<Report> reportList = new ArrayList<>();
        for (ConditionalAction conditionalAction : conditionalActionList) {
            final boolean conditionResult = conditionalAction.evaluate(facts);
            reportList.add(new Report(facts, conditionalAction, conditionResult));
        }
        return reportList;
    }
}
```

##### 示例 5-24\. 报告类

```java
public class Report {

    private final ConditionalAction conditionalAction;
    private final Facts facts;
    private final boolean isPositive;

    public Report(final Facts facts,
                     final ConditionalAction conditionalAction,
                     final boolean isPositive) {
        this.facts = facts;
        this.conditionalAction = conditionalAction;
        this.isPositive = isPositive;
    }

    public ConditionalAction getConditionalAction() {
        return conditionalAction;
    }

    public Facts getFacts() {
        return facts;
    }

    public boolean isPositive() {
        return isPositive;
    }

    @Override
    public String toString() {
        return "Report{" +
                "conditionalAction=" + conditionalAction +
                ", facts=" + facts +
                ", result=" + isPositive +
                '}';
    }
}
```

我们如何测试`Inspector`？您可以通过编写一个简单的单元测试开始，如示例 5-25 所示。这个测试突显了我们当前设计的一个基本问题。事实上，`ConditionalAction`接口违反了*接口隔离原则*（ISP）。

##### 示例 5-25\. 强调 ISP 违规

```java
public class InspectorTest {

    @Test
    public void inspectOneConditionEvaluatesTrue() {

        final Facts facts = new Facts();
        facts.setFact("jobTitle", "CEO");
        final ConditionalAction conditionalAction = new JobTitleCondition();
        final Inspector inspector = new Inspector(conditionalAction);

        final List<Report> reportList = inspector.inspect(facts);

        assertEquals(1, reportList.size());
        assertEquals(true, reportList.get(0).isPositive());
    }

    private static class JobTitleCondition implements ConditionalAction {

        @Override
        public void perform(Facts facts) {
            throw new UnsupportedOperationException();
        }

        @Override
        public boolean evaluate(Facts facts) {
            return "CEO".equals(facts.getFact("jobTitle"));
        }
    }
}
```

什么是接口隔离原则？您可能注意到`perform`方法的实现是空的。事实上，它抛出了一个`UnsupportedOperationException`异常。这是一个情况，您依赖于一个接口（`ConditionalAction`），它提供了比您实际需要的更多内容。在这种情况下，我们只是想要建模一个条件——一个求值为真或假的东西。尽管如此，我们还是被迫依赖于`perform()`方法，因为它是接口的一部分。

这个通用想法是接口隔离原则的基础。它主张，任何类都不应该被迫依赖它不使用的方法，因为这会引入不必要的耦合。在第二章中，您学习了另一个原则，即*单一责任原则*（SRP），它促进了高内聚性。SRP 是一个通用的设计指导原则，一个类应该负责一个功能，并且只有一个改变的原因。尽管 ISP 听起来可能像是相同的想法，但它采取了不同的视角。ISP 关注的是接口的使用者而不是其设计。换句话说，如果一个接口最终非常庞大，可能是因为接口的使用者看到了一些它不关心的行为，这会导致不必要的耦合。

为了符合接口隔离原则，我们鼓励将概念分离到更小的接口中，这些接口可以独立演化。这个想法实质上促进了更高的内聚性。分离接口还为引入更接近所需领域的命名提供了机会，比如`Condition`和`Action`，我们将在下一节中探讨这些内容。

# 设计流畅的 API

到目前为止，我们为用户提供了一种添加具有复杂条件的操作的方式。这些条件是使用增强的开关语句创建的。然而，对于业务用户来说，语法并不像他们希望的那样友好，以指定简单条件。我们希望允许他们以符合其领域并且更简单的方式添加规则（条件和动作）。在本节中，您将了解建造者模式以及如何开发自己的流畅 API 来解决这个问题。

## 什么是流畅 API？

流畅 API 是专门为特定领域量身定制的 API，以便您可以更直观地解决特定问题。它还支持链式方法调用的概念，用于指定更复杂的操作。您可能已经熟悉几个知名的流畅 API：

+   [Java Streams API](https://oreil.ly/549wN) 允许您以更符合解决问题需求的方式指定数据处理查询。

+   [Spring Integration](https://oreil.ly/rMIMD) 提供了一个 Java API，用于使用与企业集成模式领域接近的词汇指定企业集成模式。

+   [jOOQ](https://www.jooq.org/) 提供了一个库，使用直观的 API 与不同的数据库进行交互。

## 领域建模

那么我们希望为我们的业务用户简化什么？我们希望帮助他们指定一个简单的“当某个条件成立时”，“然后执行某事”的组合作为规则。在此领域中有三个概念：

条件

应用于某些事实的条件，将评估为真或假。

动作

一组特定的操作或要执行的代码。

规则

这是一个条件和一个动作在一起。只有在条件为真时动作才会执行。

现在我们已经定义了领域中的概念，我们将其转换为 Java！让我们首先定义`Condition`接口，并重用我们现有的`Action`接口，如示例 5-26 所示。请注意，我们也可以使用自 Java 8 起可用的`java.util.function.Predicate`接口，但是`Condition`名称更能代表我们的领域。

###### 注意

在编程中名称非常重要，因为良好的名称有助于理解代码解决的问题。在许多情况下，名称比接口的“形状”（其参数和返回类型）更重要，因为名称向阅读代码的人传达上下文信息。

##### 示例 5-26\. Condition 接口

```java
@FunctionalInterface
public interface Condition {
    boolean evaluate(Facts facts);
}
```

现在剩下的问题是如何建模规则的概念？我们可以定义一个带有`perform()`操作的接口`Rule`。这将允许您提供`Rule`的不同实现。这个接口的一个合适的默认实现是一个名为`DefaultRule`的类，它将与执行规则相关的适当逻辑一起持有`Condition`和`Action`对象，如示例 5-27 所示。

##### 示例 5-27\. 建模规则的概念

```java
@FunctionalInterface
interface Rule {
    void perform(Facts facts);
}

public class DefaultRule implements Rule {

    private final Condition condition;
    private final Action action;

    public Rule(final Condition condition, final Action action) {
        this.condition = condition;
        this.action = action;
    }

    public void perform(final Facts facts) {
        if(condition.evaluate(facts)){
            action.execute(facts);
        }
    }
}
```

我们如何使用所有这些不同的元素创建新规则？您可以在示例 5-28 中看到一个示例。

##### 示例 5-28\. 构建一个规则

```java
final Condition condition = (Facts facts) -> "CEO".equals(facts.getFact("jobTitle"));
final Action action = (Facts facts) -> {
      var name = facts.getFact("name");
      Mailer.sendEmail("sales@company.com", "Relevant customer!!!: " + name);
};

final Rule rule = new DefaultRule(condition, action);
```

## 构建器模式

然而，即使代码使用了接近我们域的名称（`Condition`、`Action`、`Rule`），这段代码仍然相当手动。用户必须实例化单独的对象并将它们组合在一起。让我们引入所谓的*构建器模式*来改进使用适当条件和操作创建 `Rule` 对象的过程。这种模式的目的是以更简单的方式创建对象。构建器模式基本上会解构构造函数的参数，并提供方法来提供每个参数。这种方法的好处在于它允许您声明与手头域相适应的方法名称。例如，在我们的案例中，我们想要使用 `when` 和 `then` 的词汇。示例 5-29 中的代码展示了如何设置构建器模式以构建 `DefaultRule` 对象。我们引入了一个方法 `when()`，它提供了条件。方法 `when()` 返回 `this`（即当前实例），这将允许我们进一步链接其他方法。我们还引入了一个方法 `then()`，它提供了动作。方法 `then()` 也返回 `this`，这允许我们进一步链接方法。最后，方法 `createRule()` 负责创建 `DefaultRule` 对象。

##### 示例 5-29\. 用于 Rule 的构建器模式

```java
public class RuleBuilder {
    private Condition condition;
    private Action action;

    public RuleBuilder when(final Condition condition) {
        this.condition = condition;
        return this;
    }

    public RuleBuilder then(final Action action) {
        this.action = action;
        return this;
    }

    public Rule createRule() {
        return new DefaultRule(condition, action);
    }
}
```

使用这个新类，您可以创建 `RuleBuilder` 并使用 `when()`、`then()` 和 `createRule()` 方法配置 `Rule`，如 示例 5-30 所示。方法链的概念是设计流畅 API 的关键方面之一。

##### 示例 5-30\. 使用 RuleBuilder

```java
Rule rule = new RuleBuilder()
        .when(facts -> "CEO".equals(facts.getFact("jobTitle")))
        .then(facts -> {
            var name = facts.getFact("name");
            Mailer.sendEmail("sales@company.com", "Relevant customer: " + name);
        })
        .createRule();
```

此代码看起来更像一个查询，并利用了所涉领域：规则的概念、`when()` 和 `then()` 作为内置结构。但它并不完全令人满意，因为用户还是会遇到两个笨拙的构造体。

+   实例化一个“空的”`RuleBuilder`

+   调用方法`createRule()`

我们可以通过提出稍微改进的 API 来改进这一点。有三种可能的改进方法：

+   我们将构造函数设置为私有，以防止用户显式调用。这意味着我们需要为我们的 API 设计一个不同的入口点。

+   我们可以将方法`when()`改为静态方法，这样就可以直接调用并且实际上会将调用转发到旧构造函数。此外，静态工厂方法提高了发现正确方法以设置`Rule`对象的可读性。

+   方法`then()`将负责最终创建我们的`DefaultRule`对象。

示例 5-31 展示了改进的`RuleBuilder`。

##### 示例 5-31\. 改进的 RuleBuilder

```java
public class RuleBuilder {
    private final Condition condition;

    private RuleBuilder(final Condition condition) {
        this.condition = condition;
    }

    public static RuleBuilder when(final Condition condition) {
        return new RuleBuilder(condition);
    }

    public Rule then(final Action action) {
        return new DefaultRule(condition, action);
    }
}
```

现在，您可以通过从 `RuleBuilder.when()` 方法开始，然后使用 `then()` 方法简单地创建规则，如 示例 5-32 所示。

##### 示例 5-32\. 使用改进的 RuleBuilder

```java
final Rule ruleSendEmailToSalesWhenCEO = RuleBuilder
        .when(facts -> "CEO".equals(facts.getFact("jobTitle")))
        .then(facts -> {
            var name = facts.getFact("name");
            Mailer.sendEmail("sales@company.com", "Relevant customer!!!: " + name);
        });
```

现在我们已经重构了`RuleBuilder`，我们可以重构业务规则引擎以支持规则而不仅仅是动作，如示例 5-33 所示。

##### 示例 5-33\. 更新的业务规则引擎

```java
public class BusinessRuleEngine {

    private final List<Rule> rules;
    private final Facts facts;

    public BusinessRuleEngine(final Facts facts) {
        this.facts = facts;
        this.rules = new ArrayList<>();
    }

    public void addRule(final Rule rule) {
        this.rules.add(rule);
    }

    public void run() {
        this.rules.forEach(rule -> rule.perform(facts));
    }

}
```

# 收获

+   测试驱动开发（Test-Driven Development，TDD）的哲学是从编写一些测试开始，这些测试将指导您实现代码。

+   模拟允许您编写单元测试，以确保触发某些行为。

+   Java 支持局部变量类型推断和 switch 表达式。

+   建造者模式有助于为实例化复杂对象设计用户友好的 API。

+   接口隔离原则通过减少对不必要方法的依赖来促进高内聚。通过将大型接口分解为更小的内聚接口，使用户只看到他们需要的内容，从而实现这一目标。

# 在你身上循环

如果您想扩展并巩固本章的知识，您可以尝试以下活动之一：

+   增强`Rule`和`RuleBuilder`以支持名称和描述。

+   增强`Facts`类，以便可以从 JSON 文件加载事实。

+   增强业务规则引擎以支持具有多个条件的规则。

+   增强业务规则引擎以支持具有不同优先级的规则。

# 完成挑战

您的业务蒸蒸日上，您的公司已将业务规则引擎作为工作流程的一部分采纳！现在您正在寻找下一个创意，并希望将您的软件开发技能应用到能够帮助世界而不仅仅是公司的新事物上。是时候迈向下一章——Twootr 了！
