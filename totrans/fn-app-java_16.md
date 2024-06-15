# 第十四章。函数式设计模式

函数式编程对面向对象设计模式的回答通常是“只使用函数即可”。从技术上讲，这是正确的；在函数式编程中，这是一种无止境的递归。然而，从一个希望用函数原则增强你的代码的面向对象思维出发，需要更多实际的建议来以函数式方式利用已知模式。

本章将介绍 *四人组* 描述的一些常用面向对象设计模式，以及它们如何从函数式方法中受益。

# 什么是设计模式？

每次解决问题时，你都不必重新发明轮子。许多问题已经解决了，或者至少以设计模式的形式存在着一个通用的方法。作为一名 Java 开发者，你很可能已经使用过或遇到过一个或多个面向对象设计模式，即使当时你并不知道它们。

本质上，面向对象的设计模式是经过测试、验证、形式化和可重复使用的对常见问题的解决方案。

*四人组* 将他们描述的模式分为三组：

行为模式

如何处理对象的职责和通信之间的关系。

创建型模式

如何抽象对象创建/实例化过程，以帮助创建、组合和表示对象。

结构型模式

如何组合对象以形成更大或增强的对象。

设计模式是一般性的脚手架，可用于将知识与应用它们到特定问题的概念共享。这就是为什么并不是每种语言或方法都适用于每种模式。尤其是在函数式编程中，许多问题除了“只是函数”之外，并不需要特定的模式。

# （函数式）设计模式

让我们来看看四种常用的面向对象设计模式以及如何以函数式方式处理它们：

+   工厂模式（创建型）

+   装饰器模式（结构型）

+   策略模式（行为型）

+   建造者模式（创建型）

## 工厂模式

*工厂模式* 属于 *创建型模式* 群体之一。它的目的是创建一个对象的实例，而不是通过使用 *工厂* 来暴露 *如何创建* 这些对象的实现细节。

### 面向对象方法

实现工厂模式的多种方式。对于我的示例，所有对象都具有共享的接口，而一个 `enum` 负责标识所需的对象类型：

```java
public interface Shape {
  int corners();
  Color color();
  ShapeType type();
}

public enum ShapeType {
  CIRCLE,
  TRIANGLE,
  SQUARE,
  PENTAGON;
}
```

形状由记录表示，只需要一个 `Color`，因为它们可以直接推断出其他属性。一个简单的 `Circle` 记录可能是这样的：

```java
public record Circle(Color color) implements Shape {

  public int corners() {
    return 0;
  }

  public ShapeType type() {
    return ShapeType.CIRCLE;
  }
}
```

一个 `Shape` 工厂需要接受 `type` 和 `color` 来创建相应的 `Shape` 实例，如下所示：

```java
public class ShapeFactory {

  public static Shape newShape(ShapeType type,
                               Color color) {
    Objects.requireNonNull(color);

    return switch (type) {
      case CIRCLE -> new Circle(color);
      case TRIANGLE -> new Triangle(color);
      case SQUARE -> new Square(color);
      case PENTAGON -> new Pentagon(color);
      default -> throw new IllegalArgumentException("Unknown type: " + type);
    };
  }
}
```

迄今为止所涉及的所有代码，模式有四个明确的部分：

+   共享的 `interface Shape`

+   标识形状的 `enum ShapeType`

+   形状的具体实现（未显示）

+   `ShapeFactory`根据其`type`和`color`创建形状

这些部分彼此依赖是预期的。然而，工厂和`enum`之间的这种相互依赖使整个方法对变更非常脆弱。如果引入新的`ShapeType`，工厂必须考虑它，否则在`switch`的`default`情况下会抛出`IllegalArgumentException`，即使存在具体的实现类型。

###### 注意

并不一定需要`default`情况，因为所有情况都已声明。它用来说明`ShapeType`和`ShapeFactory`之间的依赖关系以及如何减轻它。

为了改进工厂，可以通过引入更功能化的方法来减少其脆弱性，并进行编译时验证。

### 更加功能化的方法

此示例创建了相当简单的记录，只需要一个参数：`Color`。这些相同的构造函数使您可以直接将“工厂”移入`enum`，因此任何新形状都自动需要相应的工厂函数。

即使 Java 的`enum`类型基于常量名称，您仍然可以为每个常量附加相应的值。在这种情况下，用于创建离散对象的工厂函数形式的`Function<Color, Shape>`值：

```java
public enum ShapeType {
  CIRCLE,
  TRIANGLE,
  SQUARE,
  PENTAGON;

  public final Function<Color, Shape> factory;

  ShapeType(Function<Color, Shape> factory) {
    this.factory = factory;
  }
}
```

由于常量声明现在需要额外的`Function<Color, Shape>`，所以代码不再编译。幸运的是，Shapes 的构造函数可用作方法引用，以创建相当简洁的工厂方法代码：

```java
public enum ShapeType {
  CIRCLE(Circle::new),
  TRIANGLE(Triangle::new),
  SQUARE(Square::new),
  PENTAGON(Pentagon::new);

  // ...
}
```

`enum`通过将离散的创建方法作为其每个常量的附加值来增强了其功能。这样，任何未来的增加，比如`HEXAGON`，都会强制您提供相应的工厂方法，无法遗漏，因为编译器会强制执行。

现在剩下的就是创建新实例的能力。您可以简单地直接使用`factory`字段及其 SAM`accept(Color color)`，但我更喜欢添加一个额外的方法来进行健全性检查：

```java
public enum ShapeType {

  // ...

  public Shape newInstance(Color color) {
    Objects.requireNonNull(color);
    return this.factory.apply(color);
  }
}
```

现在创建一个新的`Shape`实例非常容易：

```java
var redCircle = ShapeType.CIRCLE.newInstance(Color.RED);
```

由于现在有了一个用于实例创建的专用方法，所以公共字段`factory`可能看起来多余。这在某种程度上是正确的。但它仍然提供了与工厂进一步交互的功能方式，比如功能组合来记录形状的创建：

```java
Function<Shape, Shape> cornerPrint =
  shape -> {
    System.out.println("Shape created with " + shape.corners() + " corners.");
  };

ShapeType.CIRCLE.factory.andThen(cornerPrint)
                        .apply(Color.RED);
```

通过将工厂与`enum`融合，决策过程——调用哪个工厂方法——被直接绑定到`ShapeType`的对应方法上取代了。现在，Java 编译器强制您在对`enum`进行任何添加时实现工厂。

这种方法通过增加的编译时安全性减少了所需的样板代码，用于未来扩展。

## 装饰器模式

*装饰器模式*是允许在运行时修改对象行为的*结构型模式*。而不是子类化，对象被包装在一个“装饰器”中，其中包含所需的行为。

### 面向对象的方法

这种模式的面向对象实现要求装饰器与它们应装饰的类型共享接口。为了简化编写新装饰器，使用实现共享接口的抽象类作为任何装饰器的起点。

想象一个咖啡机，有一个方法来准备咖啡。共享接口和具体实现如下：

```java
public interface CoffeeMaker {
  List<String> getIngredients();
  Coffee prepare();
}

public class BlackCoffeeMaker implements CoffeeMaker {

  @Override
  public List<String> getIngredients() {
    return List.of("Robusta Beans", "Water");
  }

  @Override
  public Coffee prepare() {
    return new BlackCoffee();
  }
}
```

目标是装饰咖啡机以添加像牛奶或糖这样的功能。因此，装饰器必须接受咖啡机并装饰 `prepare` 方法。一个简单的共享 `abstract` 装饰器如 ??? 所示。

```java
public abstract class Decorator implements CoffeeMaker { ![1](img/1.png)

  private final CoffeeMaker target;

  public Decorator(CoffeeMaker target) { ![2](img/2.png)
    this.target = target;
  }

  @Override
  public List<String> getIngredients() { ![3](img/3.png)
    return this.target.getIngredients();
  }

  @Override
  public Coffee prepare() { ![3](img/3.png)
    return this.target.prepare();
  }
}
```

![1](img/#co_functional_design_patterns_CO1-1)

`Decorator` 实现 `CoffeeMaker`，因此它可以作为一个插入式替换使用。

![2](img/#co_functional_design_patterns_CO1-2)

构造函数接受原始的 `CoffeeMaker` 实例，该实例应该被装饰。

![3](img/#co_functional_design_patterns_CO1-3)

`getIngredients` 和 `prepare` 方法只需调用装饰的 `CoffeeMaker`，因此任何实际的装饰器都可以使用 `super` 调用来获取“原始”结果。

`abstract Decorator` 类型聚合了装饰一个 `CoffeeMaker` 所需的最小功能。借助它的帮助，将蒸牛奶添加到咖啡中变得简单。现在你所需的只是一个牛奶盒，如 示例 14-1 中所示。

##### 示例 14-1\. 使用装饰器添加牛奶

```java
public class AddMilkDecorator extends Decorator {

  private final MilkCarton milkCarton;

  public AddMilkDecorator(CoffeeMaker target,
                          MilkCarton milkCarton) { ![1](img/1.png)
    super(target);

    this.milkCarton = milkCarton;
  }

  @Override
  public List<String> getIngredients() { ![2](img/2.png)
    var newIngredients = new ArrayList<>(super.getIngredients());
    newIngredients.add("Milk");
    return newIngredients;
  }

  @Override
  public Coffee prepare() { ![3](img/3.png)
    var coffee = super.prepare();
    coffee = this.milkCarton.pourInto(coffee);
    return coffee;
  }
}
```

![1](img/#co_functional_design_patterns_CO2-1)

构造函数需要接受所有的要求，因此除了 `CoffeeMaker`，还需要一个 `MilkCarton`。

![2](img/#co_functional_design_patterns_CO2-2)

通过首先调用`super`，使结果可变，并将牛奶添加到之前使用的成分列表，装饰器钩子进入`getIngredients`调用。

![3](img/#co_functional_design_patterns_CO2-3)

`prepare` 调用同样要求 `super` 执行其预期目的，并用牛奶“装饰”结果的咖啡。

现在很容易制作“café con leche^(3)"：

```java
CoffeeMaker coffeeMaker = new BlackCoffeeMaker();

CoffeeMaker decoratedCoffeeMaker =
  new AddMilkDecorator(coffeeMaker,
                       new MilkCarton());

Coffee cafeConLeche = decoratedCoffeeMaker.prepare();
```

装饰器模式实现起来相当简单。尽管如此，为了在咖啡中加入牛奶，需要大量代码。如果你的咖啡里也要加糖，你需要创建另一个装饰器，其中有冗余的样板代码，并且需要再次包装装饰的 `CoffeeMaker`：

```java
CoffeeMaker coffeeMaker = new BlackCoffeeMaker();

CoffeeMaker firstDecoratedCoffeeMaker =
  new AddMilkDecorator(coffeeMaker,
                       new MilkCarton());

CoffeeMaker lastDecoratedCoffeeMaker =
  new AddSugarDecorator(firstDecoratedCoffeeMaker);

Coffee lastDecoratedCoffeeMaker = coffeeMaker.prepare();
```

必须有一种更简单的方法来改进装饰器的创建和使用多个装饰器的过程。

因此，让我们看看如何改用功能组合。

### 更功能化的方法

任何重构工作朝向更功能化方法的第一步是剖析实际发生的事情。装饰器模式由两部分组成，适合改进：

+   使用一个或多个装饰器装饰 `CoffeeMaker`

+   创建一个 `Decorator` 本身

“如何装饰”的第一部分归结为获取现有的 `CoffeeMaker` 并“某种方式”添加新行为并返回一个新的 `CoffeeMaker` 以供使用。因此，实质上，该过程看起来像是一个 `Function<CofeeMaker, CoffeeMaker>`。

与以往一样，逻辑被捆绑为方便类型中的 `static` 高阶方法。该方法接受一个 `CoffeeMaker` 和一个装饰器，并用功能组合将它们组合起来：

```java
public final class Barista {

  public static CoffeeMaker decorate(CoffeeMaker coffeeMaker,
                                     Function<CoffeeMaker, CoffeeMaker> decorator) {

    return decorator.apply(coffeeMaker);
  }

  private Barista() {
    // Suppress default constructor.
    // Ensures non-instantiability and non-extendability.
  }
}
```

`Barista` 类有一个带参数的 `decorate` 方法，通过接受一个 `Function<CofeeMaker, CoffeeMaker>` 来反转流程以实际执行装饰的过程。尽管装饰现在“感觉”更加功能化，但仍然只接受一个单一的 `Function`，对于多个装饰仍然显得繁琐：

```java
CoffeeMaker decoratedCoffeeMaker =
  Barista.decorate(new BlackCoffeeMaker(),
                   coffeeMaker -> new AddMilkDecorator(coffeeMaker,
                                                       new MilkCarton()));

CoffeeMaker finalCoffeeMaker =
  Barista.decorate(decoratedCoffeeMaker,
                   AddSugarDecorator::new);
```

幸运的是，在 Chapter 6 中我讨论了一种功能 API 来按顺序处理多个元素：Streams。

装饰过程有效地是一个 *减少*，原始 `CoffeMaker` 作为其初始值，并且 `Function<CoffeeMaker, CoffeeMaker>` 接受先前的值以创建新的 `CoffeeMaker`。因此，装饰过程看起来像是 Example 14-2 中描述的。

##### 示例 14-2\. 通过减少多个装饰

```java
public final class Barista {

 public static
 CoffeeMaker decorate(CoffeeMaker coffeeMaker, ![1](img/1.png)
                      Function<CoffeeMaker, CoffeeMaker>... decorators) {

    Function<CoffeeMaker, CoffeeMaker> reducedDecorations = ![2](img/2.png)
      Arrays.stream(decorators)
            .reduce(Function.identity(),
                    Function::andThen);

    return reducedDecorations.apply(coffeeMaker); ![3](img/3.png)
  }
}
```

![1](img/#co_functional_design_patterns_CO3-1)

`decorate` 方法仍然接受原始的 `CoffeeMaker` 进行装饰。但是，由于可变参数的存在，可以提供任意数量的装饰。

![2](img/#co_functional_design_patterns_CO3-2)

通过创建一个流并将所有元素减少为单一的 `Function<CoffeeMaker, CoffeeMaker>` 来将装饰组合在一起。

![3](img/#co_functional_design_patterns_CO3-3)

最后，单一的减少装饰与 `CoffeeMaker` 组合。

现在，通过组合多个功能性和功能类似的技术，制作 café con leche 变得更加简单：

```java
CoffeeMaker decoratedCoffeeMaker =
  Barista.decorate(new BlackCoffeeMaker(),
                   coffeeMaker -> new AddMilkDecorator(coffeeMaker,
                                                       new MilkCarton()),
                   AddSugarDecorator::new);
```

装饰过程通过将装饰器逐个嵌套改进为一个单一调用。然而，使用函数仍然可以改进装饰器的创建。

您可以使用另一个便利类型将装饰器组合在一起，而不必自己创建一个 `Function<CoffeeMaker, CoffeeMaker>` 形式的装饰器，通过使用 lambda 或方法引用。

`Decorations` 便利类型的实现与其静态工厂方法非常直接，如下代码所示：

```java
public final class Decorations {

  public static Function<CoffeeMaker, CoffeeMaker> addMilk(MilkCarton milkCarton) {
    return coffeeMaker -> new AddMilkDecorator(coffeeMaker, milkCarton);
  }

  public static Function<CoffeeMaker, CoffeeMaker> addSugar() {
    return AddSugarCoffeeMaker::new;
  }

  // ...
}
```

所有可能的成分都可以通过单一类型获得，无需任何调用者了解实际实现或其他要求，除了每种方法的参数。这样，您可以使用更简洁流畅的调用来装饰您的咖啡：

```java
CoffeeMaker maker = Barista.decorate(new BlackCoffeeMaker(),
                                     Decorations.addMilk(milkCarton),
                                     Decorations.addSugar());
var coffee = maker.prepare();
```

函数式方法的主要优点在于可能消除显式嵌套并暴露具体实现类型。与其用额外的类型和重复的样板填充您的包裹，不如利用 JDK 已有的函数接口来帮助您以更简洁的代码实现相同的结果。尽管应该将相关代码组合在一起，使相关功能位于一个单独的文件中，如果它能创建更好的层次结构，也可以拆分它，但并不需要。

## 策略模式

*策略模式*属于*行为模式*的一种。由于主导大多数面向对象设计的*开闭原则*⁠^(4)，不同的系统通常通过抽象耦合，如针对接口而不是具体实现进行编程。

这种*抽象耦合*提供了更多理论组件一起工作的有用虚构，以便稍后实现，而不是您的代码知道实际实现。策略使用这种解耦的代码风格来创建基于相同抽象的可互换的小逻辑单元。哪个被选择在运行时决定。

### 面向对象方法

想象一下，您正在一个销售实物商品的电子商务平台上工作。这些商品必须以某种方式运送给客户。有多种方法可以运送物品，如不同的运输公司或运输类型。

这些各种运输选项共享一个常见的抽象，然后在系统的另一部分中使用，如`ShippingService`类型，用于发货包裹：

```java
public interface ShippingStrategy {
  void ship(Parcel parcel);
}

public interface ShippingService {
  void ship(Parcel parcel,
            ShippingStrategy strategy);
}
```

然后，每个选项都实现为一个`ShipppingStrategy`。在这种情况下，让我们只看标准和加急运输：

```java
public class StandardShipping implements ShippingStrategy {
  // ...
}

public class ExpeditedShipping implements ShippingStrategy {

  public ExpeditedShipping(boolean signatureRequired) {
    //...
  }

  // ...
}
```

每种策略都需要自己的类型和具体实现。这种一般方法看起来与我在前一节讨论的装饰器非常相似。这就是为什么它几乎可以以相同的功能方式简化的原因。

### 更功能化的方法

*策略模式*背后的整体概念归结为*行为参数化*。这意味着`ShippingService`提供了一个通用的框架，允许包裹被发货。然而，如何实际发货需要使用从外部传递给它的`ShippingStrategy`填充。

策略应该是小型和上下文绑定的决策，并且通常可以由一个功能接口表示。在这种情况下，您有多种选项来创建和使用策略：

+   Lambda 表达式和方法引用

+   部分应用函数

+   具体实现

简单的策略没有任何额外要求最好是通过`class`分组，并通过方法引用来使用与签名兼容的方法：

```java
public final class ShippingStrategies {

  public static ShippingStrategy standardShipping() {
    return parcel -> ...;
  }
}

// HOW TO USE
shippingService.ship(parcel,
                     ShippingStrategies::standardShipping);
```

更复杂的策略可能需要额外的参数。这就是部分应用函数将代码累积到一个单一类型中以提供更简单创建方法的地方：

```java
public final class ShippingStrategies {

  public static ShippingStrategy expedited(boolean requiresSignature) {

    return parcel -> {
      if (requiresSignature) {
        // ...
      }
    };
  }
}

// HOW TO USE
shippingService.ship(parcel,
                     ShippingStrategies.expedited(true));
```

这两个功能选项用于创建和使用策略已经是一种更为简洁的处理策略的方法。它们还消除了需要额外的实现类型来表示策略的要求。

然而，如果由于更复杂的策略或其他要求而两种功能选项都不可行，您总是可以使用具体实现。如果从面向对象的策略过渡，它们将是具体实现，从一开始就开始。这就是为什么策略模式是引入逐步转换现有策略为功能代码或至少在新策略中使用它的首选方法。

## 构建者模式

*建造者模式*是另一种用于通过将构建与表示分离来创建更复杂数据结构的*创建型模式*。它解决了各种对象创建问题，例如多步创建、验证和改进的可选参数处理。因此，它是`Records`的良好伴侣，后者只能一次性创建。在第五章中，我已经讨论了如何为`Record`创建一个构建器。然而，本节将从功能角度来看待构建器。

### 面向对象方法

假设您有一个简单的`record User`，具有三个属性和一个组件验证：

```java
public record User(String email, String name, List<String> permissions) {

  public User {
    if (email == null || email.isBlank()) {
      throw new IllegalArgumentException("'email' must be set.");
    }

    if (permissions == null) {
      permissions = Collections.emptyList();
    }
  }
}
```

如果您需要分步创建`User`，比如稍后再添加`permissions`，如果没有额外的代码，您就无法成功。因此，让我们像???中展示的那样添加一个内部构建器。

```java
public record User(String email, String name, List<String> permissions) {

  // ... shorthand constructor omitted

  public static class Builder { ![1](img/1.png)

    private String email;
    private String name;
    private final List<String> permissions = new ArrayList<>();

    public Builder email(String email) { ![2](img/2.png)
      this.email = email;
      return this;
    }

    public Builder name(String name) { ![2](img/2.png)
      this.name = name;
      return this;
    }

    public Builder addPermission(String permission) { ![3](img/3.png)
      this.permissions.add(permission);
      return this;
    }

    public User build() { ![4](img/4.png)
      return new User(this.email, this.name, this.permissions);
    }
  }

  public static Builder builder() { ![5](img/5.png)
    return new Builder();
  }
}
```

![1](img/#co_functional_design_patterns_CO4-1)

构建器被实现为一个内部`static`类，模仿其父记录的所有组件。

![2](img/#co_functional_design_patterns_CO4-2)

每个组件都有其专用的设置方法，返回`Builder`实例以进行流畅的调用链。

![3](img/#co_functional_design_patterns_CO4-4)

针对基于集合的字段的附加方法允许您添加单个元素。

![4](img/#co_functional_design_patterns_CO4-5)

`build`方法只需调用适当的`User`构造函数。

![5](img/#co_functional_design_patterns_CO4-6)

添加了一个`static builder`方法，因此您不需要自己创建`Builder`实例。

这需要相当多的样板代码和重复，以允许像这样更加灵活和简单的创建流程：

```java
var builder = User.builder()
                  .email("ben@example.com")
                  .name("Ben Weidig");

// DO SOMETHING ELSE, PASS BUILDER ALONG

var user = builder.addPermission("create")
                  .addPermission("edit")
                  .build();
```

通常，通过添加更好的支持可选和非可选字段的*telescoping constructors*或附加验证代码，构建器会更加复杂。

老实说，目前设计中优化或更改生成器模式的方式并不多。你可以使用工具辅助的方法为你生成生成器，但这只会减少你需要编写的代码量，而不是生成器本身的必要性。

但这并不意味着生成器不能通过一些功能性的触摸来改进。

### 更多功能性方法

大多数情况下，生成器与它正在构建的类型强耦合，作为一个内部`class`，具有流畅的方法来提供参数和一个`build`方法来创建实际的对象实例。功能性方法可以通过多种方式改进此创建流程。

首先，它使昂贵值的惰性计算成为可能。与直接接受一个值不同，`Supplier<T>`为你提供了一个仅在`build`调用中解析的惰性包装器：

```java
public record User(String email, String name, List<String> permissions) {

  // ...

  private Supplier<String> emailSupplier;

  public Builder email(Supplier<String> emailSupplier) {
    this.emailSupplier = emailSupplier;
    return this;
  }

  // ...

  User build() {
    var email = this.emailSupplier.get();
    // ...
  }
}
```

你可以支持懒惰和非懒惰的变体。例如，你可以更改原始方法以设置`emailSupplier`，而不是要求`email`和`emailSupplier`字段：

```java
public record User(String email, String name, List<String> permissions) {

  // ...

  private Supplier<String> emailSupplier;

  public Builder email(String email) {
    this.emailSupplier = () -> email;
    return this;
  }

  // ...
}
```

第二，生成器可以模仿 Groovy 的`with`⁠^(5)，如下所示：

```java
var user = User.builder()
               .with(builder -> {
                 builder.email = "ben@example.com";
                 builder.name = "Ben Weidig";
               })
               .withPermissions(permissions -> {
                 permissions.add("create");
                 permissions.add("view");
               })
               .build();
```

要实现这一点，必须向生成器添加基于`Consumer`的高阶方法，如示例 14-3 所示。

##### 示例 14-3\. 向`User`生成器添加`with`方法

```java
public record User(String email, String name, List<String> permissions) {

  // ...

  public static class Builder {

    public String email; ![1](img/1.png)
    public String name;

    private List<String> permissions = new ArrayList<>(); ![2](img/2.png)

    public Builder with(Consumer<Builder> builderFn) { ![3](img/3.png)
      builderFn.accept(this);
      return this;
    }

    public Builder withPermissions(Consumer<List<String>> permissionsFn) { ![3](img/3.png)
      permissionsFn.accept(this.permissions);
      return this;
    }

    // ...
  }

  // ...
}
```

![1](img/#co_functional_design_patterns_CO5-1)

为了在`Consumer`中可变，生成器字段需要是`public`的。

![2](img/#co_functional_design_patterns_CO5-2)

但并非所有字段都应该是`public`的。例如，基于集合的类型最好由它们自己的`with`方法提供服务。

![3](img/#co_functional_design_patterns_CO5-3)

为`permissions`添加另一个`with`方法可以防止意外将其设置为`null`，并减少`Consumer`中所需的代码到实际所需的操作。

当然，生成器本来可以使用`public`字段。但是，那样就无法进行流式调用。通过向其添加基于`Consumer`的`with`方法，整体调用链仍然是流畅的，而且你可以在创建流程中使用 lambda 甚至方法引用。

即使像生成器模式这样的设计模式没有与之相等的功能性变体，也可以通过在其中添加一些功能性概念使其更加通用。

# 对功能性设计模式的最终思考

将其称为“功能性设计模式”通常感觉像是一个反讽，因为它们几乎与它们的面向对象的对应物相反。面向对象的设计模式通常是针对常见（面向对象的）问题的形式化和易于重复的解决方案。这种形式化通常伴随着许多严格的概念隐喻和样板代码，几乎没有偏离的余地。

解决由面向对象设计模式提出的问题的函数化方法利用了函数的*第一类公民权*。它用函数接口替换了以前明确形式化的模板和必需的类型结构。结果代码更为简洁明了，还可以以新的方式进行结构化，比如返回具体实现的`static`方法或者部分应用的函数，而不是复杂的自定义类型层次结构。

然而，首先删除样板代码是一件*好事*吗？更为简单和简洁的代码始终是一个值得追求的目标。然而，初始的样板代码不仅仅是面向对象方法的要求：它还用于创建一个更复杂的操作域。

将所有中间类型替换为已有的函数接口，会减少代码阅读者直接可见的信息量。因此，在替换更具表达力的基于领域的方法及其所有类型和结构与使用更为函数化的方法之间，需要找到一个折中点。

幸运的是，与本书中讨论的大多数技术一样，这并非是“非此即彼”。在经典面向对象模式中识别函数化可能性需要你更高层次地看待问题的解决方式。例如，*责任链*设计模式处理为多个对象提供在预定义操作链中处理元素的机会。这听起来与流（Stream）或可选（Optional）管道的工作方式非常相似，或者函数组合创建功能链的方式。

面向对象设计模式帮助你识别解决问题的一般方法。然而，部分或完全转向更为函数化的解决方案往往能提供更简单和更简洁的替代方案。

# 要点

+   面向对象设计模式是知识共享的一种被证明和形式化的方式。通常需要多种类型来表示特定领域的常见问题的解决方案。

+   函数化方法利用*第一类公民权*替换所有额外类型与已有的函数接口。

+   函数式原则允许删除许多通常由许多面向对象设计模式所需的样板代码。

+   模式的实现变得更为简洁，但使用类型的显式表达能力可能会受到影响。如有必要，使用领域特定的函数接口恢复表达能力。

+   即使对于没有函数等效的设计模式，添加某些函数技术也可以提高其灵活性和简洁性。

^(1) 所谓的“无底蜷缠”描述了*无限回归*的问题：由递归原则管理的无限系列实体。每个实体依赖于或由其前身产生，这与许多函数设计哲学相符。

^(2) Gamma, E., Helm, R., Johnson, R., & Vlissides, J. (1994). Design patterns: Elements of reusable object-oriented software. Boston, MA: Addison Wesley.

^(3) *Café con leche*是西班牙和拉丁美洲流行的一种咖啡变种。字面意思是“咖啡加牛奶”。我没有使用“flat white”作为我的例子，因为那样我就需要先蒸牛奶。

^(4) *开闭原则*是*SOLID 原则*的一部分。它指出，实体（如类、方法、函数等）应*对扩展开放*，但*对修改关闭*。详细信息请参见维基百科页面：[开闭原则](https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle)和[SOLID](https://en.wikipedia.org/wiki/SOLID)。

^(5) Groovy 提供了一个`with`方法，接受闭包以简化对相同变量的重复使用。更多信息请参阅[官方 Groovy 风格指南](https://groovy-lang.org/style-guide.xhtml#_using_code_with_code_and_code_tap_code_for_repeated_operations_on_the_same_bean)。
