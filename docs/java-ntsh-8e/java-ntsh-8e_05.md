# 第四章 Java 类型系统

在本章中，我们超越了基本的面向对象编程与类，进入了有效使用 Java 类型系统所需的其他概念。

###### 注意

*静态类型*语言是指变量具有明确的类型，并且将不兼容类型的值分配给变量是编译时错误的语言。仅在运行时检查类型兼容性的语言称为*动态类型*语言。

Java 是一个典型的静态类型语言的例子。JavaScript 则是一个动态类型语言的例子，允许任何变量存储任何类型的值。

Java 类型系统不仅涉及类和基本类型，还包括与类的基本概念相关的其他种类的引用类型，但它们在某些方面有所不同，并且通常由`javac`或 JVM 特殊处理。

我们已经见过数组和类，Java 最广泛使用的两种引用类型之一。本章开始讨论另一种非常重要的引用类型——*接口*。然后我们进入讨论 Java 的*泛型*，它在 Java 类型系统中扮演重要角色。掌握了这些主题后，我们可以讨论 Java 中编译时和运行时类型的差异。

为了完整展示 Java 参考类型的全貌，我们看看特殊类型的类和接口——被称为*枚举*和*注解*。我们在这一章节结束时讨论*lambda 表达式*和*嵌套类型*，然后回顾增强类型推断如何使 Java 的*非显式类型*可供程序员使用。

让我们开始看一下接口——除了类之外 Java 最重要的参考类型之一，也是 Java 类型系统其余部分的关键构建块。

# 接口

在第三章中，我们介绍了继承的概念。我们也看到 Java 类只能继承自一个类。这对我们想要构建的面向对象程序类型是一个相当大的限制。Java 的设计者们知道这一点，但他们也希望确保 Java 的面向对象编程方法比如 C++更简单且不易出错。

他们选择的解决方案是引入接口的概念到 Java 中。像类一样，*接口*定义了一个新的引用类型。顾名思义，接口旨在表示 API——因此它提供了一个类型的描述以及实现该 API 的类必须提供的方法（及其签名）的描述。

通常情况下，Java 接口不提供描述的方法的任何实现代码。这些方法被认为是*强制性的*——希望实现接口的任何类必须提供这些方法的实现。

但是，接口可能希望标记一些 API 方法是可选的，如果选择不实现它们，则实现类不需要实现它们。这是通过`default`关键字完成的，并且接口必须提供这些可选方法的实现，这将被任何选择不实现它们的实现类使用。

###### 注意

在 Java 8 中引入了接口中可选方法的能力。在任何早期版本中都不可用。请参阅“记录和接口”以获取有关可选（也称为默认）方法如何工作的完整描述。

不可能直接实例化一个接口并创建一个接口类型的成员。相反，类必须*实现*接口以提供必要的方法体。

实现类的任何实例都*与*类定义的类型和接口定义的类型兼容。这意味着实例可以在需要类类型或接口类型的任何代码中替换。这扩展了 Liskov 原则，如在“引用类型转换”中所见。

另一种说法是，如果两个对象不共享相同的类或超类，它们仍然可以与相同接口类型兼容，如果两个对象都是实现接口的类的实例的话。

## 定义接口

接口定义有些类似于类定义，其中所有（必需的）方法都是抽象的，关键字`class`已被替换为`interface`。例如，以下代码显示了名为`Centered`的接口的定义（例如，一个`Shape`类，比如在第三章中定义的那些，如果想要允许其中心坐标被设置和查询，则可能实现该接口）：

```java
interface Centered {
  void setCenter(double x, double y);
  double getCenterX();
  double getCenterY();
}
```

对接口成员施加了一些限制：

+   所有接口的强制方法都是隐式`abstract`的，必须用分号代替方法体。`abstract`修饰符是允许的，但按照惯例通常省略。

+   接口定义了一个公共 API。按照惯例，接口成员隐式地是`public`的，并且通常省略不必要的`public`修饰符。

+   一个接口不能定义任何实例字段。字段是一个实现细节，而接口是一个规范，不是一个实现。在接口定义中唯一允许的字段是声明为`static`和`final`的常量。

+   一个接口不能被实例化，因此它不定义构造函数。

+   接口可以包含嵌套类型。任何此类类型都隐式地是`public`和`static`的。请参阅“嵌套类型”以获取嵌套类型的完整描述。

+   自 Java 8 起，接口可以包含静态方法。Java 的早期版本不允许这样做，这被广泛认为是 Java 语言设计上的一个缺陷。

+   从 Java 9 开始，接口可以包含`private`方法。这些方法的使用案例有限，但是随着接口结构的其他变化，禁止它们似乎是随意的。

+   在接口中尝试定义`protected`方法是编译时错误。

## 扩展接口

接口可以扩展其他接口，并且与类定义类似，接口定义通过包含一个`extends`子句来指示这一点。当一个接口扩展另一个接口时，它继承其超接口的所有方法和常量，并且可以定义新的方法和常量。然而，不同于类，接口定义的`extends`子句可以包含多个超接口。例如，以下是一些扩展其他接口的接口：

```java
interface Positionable extends Centered {
  void setUpperRightCorner(double x, double y);
  double getUpperRightX();
  double getUpperRightY();
}
interface Transformable extends Scalable, Translatable, Rotatable {}
interface SuperShape extends Positionable, Transformable {}
```

如果一个接口扩展了多个接口，则继承每个接口的所有方法和常量，并且可以定义自己的额外方法和常量。实现这种接口的类必须实现直接由接口定义的抽象方法，以及从所有超接口继承的所有抽象方法。

## 实现一个接口

就像类使用`extends`指定其超类一样，它可以使用`implements`来命名一个或多个它支持的接口。`implements`关键字可以出现在类声明中，在`extends`子句之后。它应该跟随一个逗号分隔的接口列表，该类实现这些接口。

当一个类在其`implements`子句中声明一个接口时，它表明它为该接口的每个强制方法提供了一个实现（即一个主体）。如果一个类实现了一个接口但没有为每个强制接口方法提供实现，它会从接口继承这些未实现的`abstract`方法，并且必须自己声明为`abstract`。如果一个类实现了多个接口，则必须实现每个接口的每个强制方法（或声明为`abstract`）。

以下代码显示了如何定义一个`CenteredRectangle`类，它扩展了第三章中的`Rectangle`类，并实现我们的`Centered`接口：

```java
public class CenteredRectangle extends Rectangle implements Centered {
  // New instance fields
  private double cx, cy;

  // A constructor
  public CenteredRectangle(double cx, double cy, double w, double h) {
    super(w, h);
    this.cx = cx;
    this.cy = cy;
  }

  // We inherit all the methods of Rectangle but must
  // provide implementations of all the Centered methods.
  public void setCenter(double x, double y) { cx = x; cy = y; }
  public double getCenterX() { return cx; }
  public double getCenterY() { return cy; }
}
```

假设我们实现了`CenteredCircle`和`CenteredSquare`，就像我们实现了这个`CenteredRectangle`类一样。每个类都扩展了`Shape`，所以类的实例可以被视为`Shape`类的实例，正如我们之前看到的那样。因为每个类实现了`Centered`接口，实例也可以被视为该类型的实例。以下代码演示了对象如何可以是类类型和接口类型的成员：

```java
Shape[] shapes = new Shape[3];      // Create an array to hold shapes

// Create some centered shapes, and store them in the Shape[]
// No cast necessary: these are all compatible assignments
shapes[0] = new CenteredCircle(1.0, 1.0, 1.0);
shapes[1] = new CenteredSquare(2.5, 2, 3);
shapes[2] = new CenteredRectangle(2.3, 4.5, 3, 4);

// Compute average area of the shapes and
// average distance from the origin
double totalArea = 0;
double totalDistance = 0;
for(int i = 0; i < shapes.length; i = i + 1) {
  totalArea += shapes[i].area();   // Compute the area of the shapes

  // Be careful, in general, the use of instanceof to determine the
  // runtime type of an object is quite often an indication of a
  // problem with the design
  if (shapes[i] instanceof Centered) { // The shape is a Centered shape
    // Note the required cast from Shape to Centered (no cast would
    // be required to go from CenteredSquare to Centered, however).
    Centered c = (Centered) shapes[i];

    double cx = c.getCenterX();    // Get coordinates of the center
    double cy = c.getCenterY();    // Compute distance from origin
    totalDistance += Math.sqrt(cx*cx + cy*cy);
  }
}
System.out.println("Average area: " + totalArea/shapes.length);
System.out.println("Average distance: " + totalDistance/shapes.length);
```

###### 注意

接口是 Java 中的数据类型，就像类一样。当一个类实现一个接口时，该类的实例可以赋值给接口类型的变量。

不要解释此示例为必须将`CenteredRectangle`对象分配给`Centered`变量，然后才能调用`setCenter()`方法或者分配给`Shape`变量然后再调用`area()`方法。相反，因为`CenteredRectangle`类定义了`setCenter()`方法并从其`Rectangle`超类继承了`area()`方法，所以你总是可以调用这些方法。

正如我们可以通过查看字节码（例如，使用`javap`工具我们将在第十三章遇到）所见，JVM 根据持有形状的局部变量类型是`CenteredRectangle`还是`Centered`而稍有不同地调用`setCenter()`方法，但这在大多数情况下在编写 Java 代码时并不重要。

## 记录和接口

记录（records）作为类的一种特例，可以像任何其他类一样实现接口。记录的主体必须包含接口所有强制方法的实现代码，并且可以包含接口任意默认方法的覆盖实现。

让我们看一个例子，应用于我们在上一章中遇到的`Point`记录。给定一个定义如下的接口：

```java
interface Translatable {
    Translatable deltaX(double dx);
    Translatable deltaY(double dy);
    Translatable delta(double dx, double dy);
}
```

那么我们可以像这样更新`Point`类型：

```java
public record Point(double x, double y) implements Translatable {
    public Translatable deltaX(double dx) {
        return delta(dx, 0.0);
    }

    public Translatable deltaY(double dy) {
        return delta(0.0, dy);
    }

    public Translatable delta(double dx, double dy) {
        return new Point(x + dx, y + dy);
    }
}
```

注意，因为记录是不可变的，所以不可能在原地修改实例，因此，如果我们需要一个修改过的对象，我们必须显式地创建一个并返回它。这意味着并非每个接口都适合由记录类型实现。

## 密封接口

我们在上一章中遇到了`sealed`关键字，应用于类。它也可以应用于接口，如下所示：

```java
sealed interface Rotate90 permits Circle, Rectangle {
    void clockwise();
    void antiClockwise();
}
```

这个密封接口表示一个形状可以旋转 90 度的能力。注意声明中还包含一个`permits`子句，指定允许实现这个接口的唯一类——在这种情况下，只有`Circle`和`Rectangle`类，以简化问题。`Circle`被修改如下：

```java
public final class Circle extends Shape implements Rotate90 {
    // ...

    @Override
    public void clockwise() {
        // No-op, circles are rotation-invariant
    }

    @Override
    public void antiClockwise() {
        // No-op, circles are rotation-invariant
    }

    // ...
}
```

而`Rectangle`已被修改如下：

```java
public final class Rectangle extends Shape implements Rotate90 {
    // ...

    @Override
    public void clockwise() {
        // Swap width and height
        double tmp = w;
        w = h;
        h = tmp;
    }

    @Override
    public void antiClockwise() {
        // Swap width and height
        double tmp = w;
        w = h;
        h = tmp;
    }

    // ...
}
```

目前为止，我们不希望处理其他形状具有旋转行为的复杂性，因此我们限制接口只能由两种最简单的情况实现：圆和矩形。

密封接口与记录之间还有一个有趣的互动，我们将在第五章讨论。

## 默认方法

从 Java 8 开始，可以在接口中声明包含实现的方法。在本节中，我们将讨论这些方法，应该将它们理解为接口所代表的 API 中的可选方法——通常称为*默认方法*。让我们首先看看为什么我们需要首先的默认机制。

### 向后兼容性

Java 平台一直非常关注向后兼容性。这意味着为早期版本的平台编写（甚至编译）的代码必须继续在后续版本的平台上运行。这一原则使得开发团队对其 JDK 或 Java 运行时环境（JRE）的升级具有高度信心，不会破坏当前正常运行的应用程序。

向后兼容性是 Java 平台的一大优势，但为了实现它，平台对其施加了一些限制。其中之一是接口不能在新版本中添加新的强制性方法。

例如，假设我们想要更新`Positionable`接口以添加底部左下角边界点的能力：

```java
public interface Positionable extends Centered {
  void setUpperRightCorner(double x, double y);
  double getUpperRightX();
  double getUpperRightY();
  void setLowerLeftCorner(double x, double y);
  double getLowerLeftX();
  double getLowerLeftY();
}
```

通过这个新的定义，如果我们试图将这个新接口与为旧接口开发的代码一起使用，那就不会起作用，因为现有代码缺少强制性方法`setLowerLeftCorner()`、`getLowerLeftX()`和`getLowerLeftY()`。

###### 注意

您可以很容易地在自己的代码中看到这种效果。编译一个依赖于接口的类文件。然后向接口添加一个新的强制性方法，并尝试使用新版本的接口与旧类文件一起运行程序。您应该会看到程序因为`NoClassDefError`而崩溃。

这个限制是 Java 8 设计者的一个关注点——因为他们的目标之一是能够升级核心 Java 集合库并引入使用 lambda 表达式的方法。

要解决这个问题，需要一个新的机制，基本上允许接口通过添加新方法来演变，而不会破坏向后兼容性。

### 实现默认方法

在不破坏向后兼容性的情况下向接口添加新方法，需要为接口的旧实现提供一些实现，以便它们可以继续工作。这个机制就是`default`方法，在 JDK 8 中首次添加到平台中。

###### 注意

可以向任何接口添加默认方法（有时称为可选方法）。这必须包括一个内联的实现，称为*默认实现*，它写在接口定义中。

默认方法的基本行为是：

+   实现类可以（但不需要）实现默认方法。

+   如果实现类实现了默认方法，则使用类中的实现。

+   如果找不到其他实现，则使用默认实现。

一个例子是`sort()`方法。它已经在 JDK 8 中被添加到接口`java.util.List`中，并且定义如下：

```java
// The <E> syntax is Java's way of writing a generic type - see
// the next section for full details. If you aren't familiar with
// generics, just ignore that syntax for now.
interface List<E> {
  // Other members omitted

  public default void sort(Comparator<? super E> c) {
    Collections.<E>sort(this, c);
  }
}
```

因此，从 Java 8 开始，任何实现`List`的对象都有一个`sort()`实例方法，可用于使用适当的`Comparator`对列表进行排序。由于返回类型是`void`，我们可能期望这是一种原地排序，事实也是如此。

默认方法的一个结果是，在实现多个接口时，可能有两个或更多接口包含具有完全相同名称和签名的默认方法。

例如：

```java
interface Vocal {
  default void call() {
    System.out.println("Hello!");
  }
}

interface Caller {
  default void call() {
    Switchboard.placeCall(this);
  }
}

public class Person implements Vocal, Caller {
  // ... which default is used?
}
```

这两个接口对`call()`的默认语义有很大的不同，并且可能导致潜在的实现冲突——*冲突的默认方法*。在 Java 8 之前的版本中，这种情况是不可能发生的，因为语言只允许单一实现继承。引入默认方法意味着 Java 现在允许一种有限的*多继承*形式（但仅限于方法实现）。Java 仍然不允许（也没有计划添加）对象状态的多重继承。

###### 提示

在一些其他语言中，特别是 C++，这个问题被称为*菱形继承*。

默认方法有一组简单的规则，以帮助解决任何潜在的歧义：

+   如果一个类以导致默认方法实现潜在冲突的方式实现了多个接口，则实现类必须重写冲突方法并提供所需的定义。

+   提供了语法，允许实现类简单地调用接口的默认方法之一，如果需要的话：

```java
public class Person implements Vocal, Caller {

    public void call() {
        // Can do our own thing
        // or delegate to either interface
        // e.g.,
        // Vocal.super.call();
        // or
        // Caller.super.call();
    }
}
```

由于默认方法的设计，存在一个轻微但无法避免的使用问题，可能在演化中的接口出现方法冲突时出现。考虑一个字节码版本为 51.0（Java 7）的类实现了两个接口`A`和`B`，它们的版本号分别为`a.0`和`b.0`。由于 Java 7 中没有默认方法，这个类将正常工作。然而，如果稍后其中一个或两个接口采用了冲突方法的默认实现，则可能会发生编译时断裂。

例如，如果版本`a.1`在`A`中引入了一个默认方法，那么当使用新版本的依赖运行时，实现类将采用这个实现。如果版本`b.1`现在也引入了相同的方法，就会造成冲突：

+   如果`B`将方法引入为强制性的（即抽象的）方法，则实现类将继续工作——无论是在编译时还是在运行时。

+   如果`B`将方法引入为默认方法，则这是不安全的，实现类将在编译时和运行时均失败。

这个小问题很大程度上是一个边界情况，在实践中支付的代价很小，以便在语言中拥有可用的默认方法。

在使用默认方法时，我们应该意识到我们可以在默认方法内部执行的操作集合有一定的限制：

+   调用接口公共 API 中的另一个方法（无论是强制性还是可选的）；此类方法的某些实现是可用的。

+   在接口上调用私有方法（Java 9 及以上）。

+   调用静态方法，无论是在接口上还是在其他地方定义的。

+   使用`this`引用（例如，作为方法调用的参数）。

这些限制的最大教训是，即使有了默认方法，Java 接口仍然缺乏有意义的状态；我们不能在接口内部修改或存储状态。

默认方法对 Java 实践者处理面向对象编程的方式产生了深远影响。与 lambda 表达式的兴起相结合，它们颠覆了许多以前的 Java 编码约定；我们将在下一章中详细讨论这一点。

## 标记接口

有时候定义一个完全空的接口是很有用的。一个类可以通过在其 `implements` 子句中简单地命名该接口来实现它，而无需实现任何方法。在这种情况下，该类的任何实例也将成为该接口的有效实例，并且可以将其强制类型转换为该类型。Java 代码可以使用 `instanceof` 运算符检查对象是否是接口的实例，因此这种技术是提供关于对象的附加信息的有用方式。它可以被看作是为类提供额外的辅助类型信息。

###### 提示

标记接口的使用远不如以前广泛。由于注解（我们将很快见到）在传递扩展类型信息时具有更大的灵活性，它们已经大多取代了标记接口。

接口 `java.util.RandomAccess` 就是一个标记接口的示例：`java.util.List` 实现使用这个接口来表明它们提供对列表元素的快速随机访问。例如，`ArrayList` 实现了 `RandomAccess`，而 `LinkedList` 则没有。关心随机访问操作性能的算法可以像这样测试 `RandomAccess`：

```java
// Before sorting the elements of a long arbitrary list, we may want
// to make sure that the list allows fast random access.  If not,
// it may be quicker to make a random-access copy of the list before
// sorting it. Note that this is not necessary when using
// java.util.Collections.sort().
List l = ...;  // Some arbitrary list we're given
if (l.size() > 2 && !(l instanceof RandomAccess)) {
    l = new ArrayList(l);
}
sortListInPlace(l);
```

正如我们稍后将看到的，Java 的类型系统与类型名称紧密耦合，这被称为 *命名类型* 的方法。标记接口就是一个很好的例子：除了名称外，它什么都没有。

# Java 泛型

Java 平台的一个显著优势是其提供的标准库。它提供了大量有用的功能，特别是常见数据结构的健壮实现。这些实现相对简单易用，并且有很好的文档支持。这些库被称为 Java 集合框架，我们将在 第八章 中详细讨论它们。如需更全面的信息，请参阅 Maurice Naftalin 和 Philip Wadler 的书 [*Java Generics and Collections*](http://shop.oreilly.com/product/9780596527754.do)（O’Reilly）。

尽管最早期的集合版本仍然非常有用，但它们存在一个相当重要的限制：数据结构（有时称为 *容器*）基本上会隐藏在其中存储的数据类型。

###### 注意

数据隐藏和封装是面向对象编程的重要原则，但在这种情况下，容器的不透明性给开发者带来了许多问题。

让我们通过展示问题并展示*泛型类型*的引入是如何解决它并使 Java 开发人员的生活变得更加轻松的。

## 泛型介绍

如果我们想要构建一个`Shape`实例的集合，我们可以使用`List`来持有它们，就像这样：

```java
List shapes = new ArrayList();   // Create a List to hold shapes

// Create some centered shapes, and store them in the list
shapes.add(new CenteredCircle(1.0, 1.0, 1.0));
// This is legal Java-but is a very bad design choice
shapes.add(new CenteredSquare(2.5, 2, 3));

// List::get() returns Object, so to get back a
// CenteredCircle we must cast
CenteredCircle c = (CenteredCircle)shapes.get(0);

// Next line causes a runtime failure
CenteredCircle c = (CenteredCircle)shapes.get(1);
```

这段代码的一个问题源于需要执行强制类型转换以获得可用形式的形状对象——`List`不知道它包含的对象类型。不仅如此，而且实际上可以将不同类型的对象放入同一个容器中，一切工作正常，直到使用非法强制转换并导致程序崩溃。

我们真正想要的是一种形式的`List`，它能理解它包含的类型。然后，`javac`可以在将非法参数传递给`List`的方法时检测到并导致编译错误，而不是推迟到运行时处理。

###### 注意

所有元素类型相同的集合称为*同类*，而可能包含不同类型元素的集合称为*异类*（有时称为“神秘肉集合”）。

Java 提供了一个简单的语法来适应同类集合。要指示一个类型是一个容器，它持有另一个引用类型的实例，我们将容器持有的*有效载荷*类型括在尖括号内：

```java
// Create a List-of-CenteredCircle
List<CenteredCircle> shapes = new ArrayList<CenteredCircle>();

// Create some centered shapes, and store them in the list
shapes.add(new CenteredCircle(1.0, 1.0, 1.0));

// Next line will cause a compilation error
shapes.add(new CenteredSquare(2.5, 2, 3));

// List<CenteredCircle>::get() returns a CenteredCircle, no cast needed
CenteredCircle c = shapes.get(0);
```

这种语法确保了编译器在运行时之前能够捕获大类不安全的代码。这当然是静态类型系统的整体目标——利用编译时的知识尽可能地帮助消除运行时问题。

结果类型结合了一个封装的容器类型和一个有效载荷类型，通常称为*泛型类型*，并且它们声明如下：

```java
interface Box<T> {
  void box(T t);
  T unbox();
}
```

这表明`Box`接口是一个通用的构造，可以容纳任何类型的有效载荷。它本身并不是一个完整的接口——它更像是一个整个接口家族的通用描述，每个接口都可以用`T`的类型替代。

## 泛型类型和类型参数

我们已经看到如何使用泛型类型通过利用编译时知识来提供增强的程序安全性，以防止简单的类型错误。在这一节中，让我们更深入地探讨泛型类型的属性。

`<T>`这种语法有一个特殊的名称，*类型参数*，另一个泛型类型的名称是*参数化类型*。这应该传达出容器类型（例如，`List`）由另一种类型（有效载荷类型）参数化的意义。当我们写一个类型像`Map<String, Integer>`时，我们正在为类型参数指定具体的值。

当我们定义具有参数的类型时，需要以不假设类型参数的方式进行。因此，`List` 类型以泛型方式声明为 `List<E>`，而类型参数 `E` 在整个过程中都作为占位符，用于当程序员使用 `List` 数据结构时使用的实际类型的载荷。

###### 提示

类型参数总是代表引用类型。不可能使用原始类型作为类型参数的值。

类型参数可以像真实类型一样在方法的签名和主体中使用，例如：

```java
interface List<E> extends Collection<E> {
  boolean add(E e);
  E get(int index);
  // other methods omitted
}
```

注意类型参数 `E` 如何用作返回类型和方法参数的参数。我们不假设载荷类型具有任何特定属性，只做一致性的基本假设——我们放入的类型是后来取出的相同类型。

这种增强实际上引入了一种新类型到 Java 的类型系统中。通过将容器类型与类型参数的值组合，我们正在创建新类型。

## Diamond 语法

当我们创建泛型类型的实例时，赋值语句的右侧重复了类型参数的值。通常情况下这是不必要的，因为编译器可以推断出类型参数的值。在现代版本的 Java 中，我们可以在所谓的 *diamond 语法* 中省略重复的类型值。

让我们通过重新编写我们早期的一个例子来看如何使用 diamond 语法：

```java
// Create a List-of-CenteredCircle using diamond syntax
List<CenteredCircle> shapes = new ArrayList<>();
```

这是赋值语句冗长性的小幅改进——我们设法节省了一些键入字符。在本章稍后讨论 Lambda 表达式时，我们将返回类型推断的话题。

## 类型擦除

在 “默认方法” 中，我们讨论了 Java 平台对向后兼容性的强烈偏好。Java 5 中引入泛型就是向新语言特性的向后兼容性的另一个例子。

中心问题是如何设计一个类型系统，允许旧的非泛型集合类与新的泛型集合类并存。设计决策是通过使用强制类型转换来实现这一点：

```java
List someThings = getSomeThings();
// Unsafe cast, but we know that the
// contents of someThings are really strings
List<String> myStrings = (List<String>)someThings;
```

这意味着 `List` 和 `List<String>` 作为类型是兼容的，至少在某种程度上是这样。Java 通过 *类型擦除* 实现了这种兼容性。这意味着泛型类型参数只在编译时可见——它们被 `javac` 剥离并不反映在字节码中。¹

###### 警告

非泛型类型 `List` 通常被称为 *原始类型*。对于现在是泛型的类型来说，使用原始形式仍然是完全合法的 Java。然而，这几乎总是质量较差代码的标志。

类型擦除机制导致 `javac` 和 JVM 看到的类型系统存在差异——我们将在“编译和运行时类型”中全面讨论这一点。

类型擦除还禁止了一些其他本来看起来合法的定义。在这段代码中，我们想要计算两种稍有不同的数据结构中表示的订单：

```java
// Won't compile
interface OrderCounter {
  // Name maps to list of order numbers
  int totalOrders(Map<String, List<String>> orders);

  // Name maps to total orders made so far
  int totalOrders(Map<String, Integer> orders);
}
```

这段代码看起来像是完全合法的 Java 代码，但它将无法编译。问题在于，尽管这两个方法看起来像是普通的重载方法，但在类型擦除后，两个方法的签名变成了相同的：

```java
  int totalOrders(Map);
```

类型擦除后，容器的原始类型仅剩下 `Map`。运行时无法通过签名区分这些方法，因此语言规范将此语法视为非法。

## 有界类型参数

考虑一个简单的泛型盒子：

```java
public class Box<T> {
    protected T value;

    public void box(T t) {
        value = t;
    }

    public T unbox() {
        T t = value;
        value = null;
        return t;
    }
}
```

这是一个有用的抽象，但假设我们想要一个只能容纳数字的限制形式的盒子。Java 允许我们通过对类型参数设置 *边界* 来实现这一点。这是限制可以用作类型参数值的类型的能力，例如：

```java
public class NumberBox<T extends Number> extends Box<T> {
    public int intValue() {
        return value.intValue();
    }
}
```

类型边界 `T extends Number` 确保 `T` 只能被兼容于 `Number` 类型的类型所替代。因此，编译器知道 `value` 必定有一个可用的 `intValue()` 方法。

###### 注意

请注意，由于 `value` 字段具有受保护的访问权限，在子类中可以直接访问它。

如果我们试图用类型参数的无效值实例化 `NumberBox`，结果将是编译错误：

```java
NumberBox<Integer> ni = new NumberBox<>(); // This compiles fine

NumberBox<Object> no = new NumberBox<>(); // Won't compile
```

初学者应尽量避免使用原始类型。即使是有经验的 Java 程序员在使用时也可能遇到问题。例如，在使用原始类型处理类型边界时，类型边界可能会被规避，但这样做会使代码容易受到运行时异常的影响：

```java
// Compiles
NumberBox n = new NumberBox();
// This is very dangerous
n.box(new Object());
// Runtime error
System.out.println(n.intValue());
```

调用 `intValue()` 失败，并抛出 `java.lang.ClassCastException` —— 因为在调用方法之前，`javac` 已经对 `value` 插入了一个无条件的强制类型转换到 `Number`。

通常情况下，类型边界可用于编写更好的泛型代码和库。通过实践，一些相当复杂的结构可以被构建，例如：

```java
public class ComparingBox<T extends Comparable<T>> extends Box<T>
                            implements Comparable<ComparingBox<T>> {
    @Override
    public int compareTo(ComparingBox<T> o) {
        if (value == null)
            return o.value == null ? 0 : -1;
        return value.compareTo(o.value);
    }
}
```

这个定义可能看起来令人生畏，但 `ComparingBox` 实际上只是包含一个 `Comparable` 值的 `Box`。该类型还通过比较两个盒子的内容，扩展了对 `ComparingBox` 类型本身的比较操作。

## 引入协变性

Java 泛型的设计包含了一个古老问题的解决方案。在 Java 的早期版本中，甚至在引入集合库之前，语言就不得不面对一个深层次的类型系统设计问题。

简单来说，问题是这样的：

> 字符串数组是否应与类型为对象数组的变量兼容？

换句话说，这段代码应该合法吗？

```java
String[] words = {"Hello World!"};
Object[] objects = words;
```

如果没有这一点，那么甚至像 `Arrays::sort` 这样的简单方法都将非常难以以预期的方式编写：

```java
Arrays.sort(Object[] a);
```

方法声明仅适用于类型为 `Object[]` 而不适用于任何其他数组类型。由于这些复杂性的结果，Java 语言标准的第一个版本确定了以下结论：

> 如果类型 `C` 的值可以分配给类型 `P` 的变量，则类型 `C[]` 的值可以分配给类型 `P[]` 的变量。

也就是说，数组的赋值语法 *随其所持有的基本类型变化*，或者说数组是 *协变的*。

这个设计决定相当不幸，因为它导致了立即的负面后果：

```java
String[] words = {"Hello", "World!"};
Object[] objects = words;

// Oh, dear, runtime error
objects[0] = new Integer(42);
```

对 `objects[0]` 的赋值企图将 `Integer` 存储到期望保存 `String` 的存储空间中。这显然是行不通的，并将抛出 `ArrayStoreException`。

###### 警告

协变数组的实用性导致它们在平台的早期阶段被视为一种必要之恶，尽管这种功能暴露了静态类型系统中的漏洞。

然而，对现代开源代码库的更多研究表明，数组协变极少被使用，且是语言的误功能。² 写新代码时应避免使用它。

在考虑 Java 平台上泛型行为时，可以提出一个非常相似的问题：“`List<String>` 是否是 `List<Object>` 的子类型？”也就是说，我们可以这样写：

```java
// Is this legal?
List<Object> objects = new ArrayList<String>();
```

乍一看，这似乎是完全合理的——`String` 是 `Object` 的子类，因此我们知道集合中的任何 `String` 元素也是有效的 `Object`。

然而，请考虑以下代码（只是将数组协变代码转换为使用 `List`）：

```java
// Is this legal?
List<Object> objects = new ArrayList<String>();

// What do we do about this?
objects.add(new Object());
```

由于 `objects` 的类型声明为 `List<Object>`，因此将 `Object` 实例添加到其中应该是合法的。然而，由于实际实例持有字符串，尝试添加 `Object` 将不兼容，因此在运行时会失败。

这将与数组的情况没有任何变化，因此解决方案是意识到虽然这是合法的：

```java
Object o = new String("X");
```

这并不意味着泛型容器类型的相应语句也是正确的，因此：

```java
// Won't compile
List<Object> objects = new ArrayList<String>();
```

另一种说法是，`List<String>` *不是* `List<Object>` 的子类型，或者泛型类型是 *不变的*，而不是 *协变的*。在讨论有界通配符时，我们将详细说明这一点。

## 通配符

例如 `ArrayList<T>` 这样的参数化类型是不 *可实例化* 的；我们无法创建它们的实例。这是因为 `<T>` 只是一个类型参数，仅仅是一个真实类型的占位符。只有当我们为类型参数提供一个具体值（例如 `ArrayList<String>`）时，类型才变得完全形成，我们才能创建该类型的对象。

如果我们希望在编译时不知道要使用的类型，则会出现问题。幸运的是，Java 类型系统能够容纳这一概念。它通过具有显式概念的*未知类型*来实现。这表示为`<?>`。这是 Java 的*通配符类型*的最简单示例。

我们可以编写涉及未知类型的表达式：

```java
ArrayList<?> mysteryList = unknownList();
Object o = mysteryList.get(0);
```

这是完全有效的 Java 代码：`ArrayList<?>`是一个变量可以拥有的完整类型，不像`ArrayList<T>`。我们不知道`mysteryList`的载荷类型的任何信息，但这对我们的代码可能并非问题。

例如，当我们从`mysteryList`中获取一个项时，它具有完全未知的类型。但是，我们可以确保该对象可以赋值给`Object`，因为泛型类型参数的所有有效值都是引用类型，而所有引用值都可以赋给类型为`Object`的变量。

另一方面，当我们使用未知类型时，它在用户代码中有一些使用限制。例如，以下代码将无法编译：

```java
// Won't compile
mysteryList.add(new Object());
```

这样做的原因很简单：我们不知道`mysteryList`的载荷类型是什么！例如，如果`mysteryList`实际上是`ArrayList<String>`的实例，那么我们不希望能够将`Object`放入其中。

我们知道我们始终可以将`null`插入到容器中，因为我们知道`null`是任何引用类型的可能值。这并不是很有用，因此，Java 语言规范还排除了使用未知类型作为载荷来实例化容器对象的可能性，例如：

```java
// Won't compile
List<?> unknowns = new ArrayList<?>();
```

未知类型可能看起来用处不大，但它的一个非常重要的用途是作为解决协变问题的起点。如果我们想要为容器使用子类型关系，我们可以使用未知类型，例如：

```java
// Perfectly legal
List<?> objects = new ArrayList<String>();
```

这意味着`List<String>`实际上是`List<?>`的子类型 — 虽然当我们使用像前面这样的赋值时，我们会丢失一些类型信息。例如，`objects.get()`的返回类型现在实际上是`Object`。

###### 注意

对于类型参数`T`的任何值，`List<?>`不是类型`List<T>`的子类型。

未知类型有时会使开发人员感到困惑，引发类似以下问题：“为什么不只使用`Object`而不是未知类型？”然而，正如我们所见，需要在泛型类型之间具有子类型关系，这实质上要求我们具有未知类型的概念。

### 有界通配符

实际上，Java 的通配符类型不仅限于未知类型，还有*有界通配符*的概念。

这些用于描述大部分未知类型的继承层次结构 —— 有效地使类似“我不知道这种类型的任何信息，但它必须实现`List`”的语句成立。

在类型参数中，这将被写成`? extends List`。这为程序员提供了一个有用的生命线。不再局限于完全未知的类型，他们知道至少类型边界的功能是可用的。

###### 警告

不管约束类型是类类型还是接口类型，都始终使用`extends`关键字。

这是一个被称为*类型变异*的概念的示例，它是关于容器类型之间继承如何与它们的载荷类型之间的继承关系相关的一般理论。

类型协变性

这意味着容器类型之间的关系与载荷类型的关系相同。这是用`extends`关键字来表达的。

类型逆变性

这意味着容器类型之间的关系与载荷类型的关系相反。这是用`super`关键字来表达的。

当讨论容器类型时，这些想法往往会出现。例如，如果`Cat`扩展`Pet`，那么`List<Cat>`是`List<? extends Pet>`的子类型，因此：

```java
List<Cat> cats = new ArrayList<Cat>();
List<? extends Pet> pets = cats;
```

然而，这与数组情况不同，因为类型安全性是以以下方式维护的：

```java
pets.add(new Cat()); // won't compile
pets.add(new Pet()); // won't compile
cats.add(new Cat());
```

编译器不能证明由`pets`指向的存储能够存储`Cat`，因此拒绝调用`add()`。然而，由于`cats`明确指向一个`Cat`对象列表，因此将新对象添加到列表中是可以接受的。

因此，非常普遍地看到这些类型的通用构造与作为载荷类型的生产者或消费者的类型一起使用。

例如，当`List`充当`Pet`对象的*生产者*时，适当的关键字是`extends`。

```java
Pet p = pets.get(0);
```

请注意，对于生产者情况，载荷类型出现为生产者方法的返回类型。

对于作为某种类型实例的*消费者*的容器类型，我们将使用`super`关键字，并且我们期望在方法参数的类型中看到载荷类型。

###### 请注意

这在由 Joshua Bloch 提出的*生产者扩展，消费者超级*（PECS）原则中得到了具体表述。

正如我们将在第八章中讨论的那样，协变性和逆变性都出现在 Java 集合中。它们主要存在是为了确保通用性“做正确的事情”，并且表现出不会让开发人员感到惊讶的方式。

## 通用方法

*通用方法*是能够接受任何引用类型实例的方法。

让我们看一个例子。在 Java 中，逗号用于允许在单行中进行多个声明（通常称为*复合声明*）。其他语言，如 Javascript 或 C，具有更一般的逗号运算符。JS 的逗号运算符 `(,)` 会评估其提供的两个表达式（从左到右），并返回最后一个表达式的值。其目的是创建一个复合表达式，在这个表达式中，多个表达式被评估，而复合表达式的值是其成员表达式的最右边的值。请注意，与短路逻辑运算符不同，逗号评估表达式的任何副作用总是会触发。

Java 的逗号比设计时更为严格。这是因为其他语言中的逗号可能导致一些非常难以理解的代码，并且可能是错误的一个极好的来源。然而，如果我们确实想要模仿其他语言中逗号运算符的行为，我们可以通过创建一个泛型方法来实现：

```java
// Note that this class is not generic
public class Utils {
  public static <T> T comma(T a, T b) {
    return b;
  }
}
```

调用 `Utils.comma()` 方法将导致计算表达式 `a` 和 `b` 的值，并在方法调用之前触发任何副作用，这是我们想要的行为。

然而，需要注意的是，即使在方法的定义中使用了类型参数，其定义所在的类（`Utils`）也不是泛型的。相反，我们看到使用了新的语法来指示可以自由使用该方法，并且返回类型与参数类型相同。

让我们再看一个例子，来自 Java 集合库。在 `ArrayList` 类中，我们可以找到一个方法，用于从 ArrayList 实例创建一个新的数组对象：

```java
@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
    if (a.length < size)
        // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

这个方法使用低级的 `arraycopy()` 方法来执行实际的工作。

###### 注意

如果我们查看 `ArrayList` 的类定义，我们可以看到它是一个泛型类，但类型参数是 `<E>`，而不是 `<T>`，而且类型参数 `<E>` 在 `toArray()` 的定义中根本不出现。

`toArray()` 方法提供了集合与 Java 原始数组之间桥接 API 的一半。API 的另一半——从数组到集合的转换——涉及一些额外的细微差别，我们将在第八章中讨论。

## 编译和运行时类型

考虑一个代码示例：

```java
List<String> l = new ArrayList<>();
System.out.println(l);
```

我们可以提出以下问题：`l` 的类型是什么？这个问题的答案取决于我们是在编译时（即 `javac` 看到的类型）还是在运行时（作为 JVM 看到的类型）考虑 `l`。

`javac` 将会将 `l` 的类型视为 `List-of-String`，并使用该类型信息来仔细检查语法错误，比如尝试对非法类型进行 `add()` 操作。

JVM 将会把 `l` 视为 `ArrayList` 类型的对象，正如我们可以从 `println()` 语句看到的那样。由于类型擦除，`l` 的运行时类型是原始类型。

因此，编译时和运行时类型略有不同。稍微奇怪的是，在某些方面，运行时类型既比编译时类型更具体，也比编译时类型更不具体。

运行时类型比编译时类型更不具体，因为有关有效载荷类型的类型信息已经消失 —— 它已经被擦除，并且产生的运行时类型只是一个原始类型。

编译时类型比运行时类型更不具体，因为我们不知道 `l` 将具体是什么类型；我们只知道它将是与 `List` 兼容的类型。

编译时和运行时类型的差异有时会让新手 Java 程序员感到困惑，但这种区别很快会被视为语言工作中的正常部分。

## 使用和设计泛型类型

在使用 Java 泛型时，按照两种不同的理解层次进行思考可能会有所帮助：

实践者

从实践者的角度来看，需要使用现有的通用库并构建一些相当简单的通用类。在这个层次上，开发人员还应该理解类型擦除的基础知识，因为几个 Java 语法特性如果没有对泛型运行时处理的意识，可能会感到困惑。

设计师

使用泛型的新库的设计者需要更多地了解泛型的能力。规范中还包括一些更难理解的部分，包括对通配符的完全理解，以及高级主题，例如“捕获”错误消息。

Java 泛型是语言规范中最复杂的部分之一，具有许多潜在的边界情况。并非每个开发人员在首次接触 Java 类型系统的时候都需要完全理解这部分内容。

# 枚举和注解

我们已经见过记录（records），但 Java 还有额外的专用类和接口形式，用于在类型系统中扮演特定角色。它们被称为*枚举类型*和*注解类型*，通常简称为*枚举*和*注解*。

## 枚举

枚举是类的一种变体，具有有限的功能和特定的语义意义，即该类型仅具有少量可能的允许值。

例如，假设我们想定义一个类型来表示红、绿和蓝的主要颜色，并且我们希望这些是该类型的唯一可能值。我们可以使用 `enum` 关键字来实现：

```java
public enum PrimaryColor {
  // The ; is not required at the end of the list of instances
  RED, GREEN, BLUE
}
```

然后，类型 `PrimaryColor` 的唯一可用实例可以作为静态字段进行引用：`PrimaryColor.RED`、`PrimaryColor.GREEN` 和 `PrimaryColor.BLUE`。

###### 注

在其他语言（如 C++）中，通过使用常量整数来实现枚举类型的角色，但 Java 的方法提供了更好的类型安全性和更大的灵活性。

由于枚举是专门的类，因此枚举可以具有成员字段和方法。如果它们具有主体（由字段或方法组成），则需要在实例列表的末尾使用分号，并且枚举常量列表必须在方法和字段之前。

例如，假设我们想要一个枚举来包含标准扑克牌的花色。我们可以通过使用一个带有参数值的枚举来实现这一点，像这样：

```java
public enum Suit {
    // ; at the end of list required for enums with parameters
    HEART('♥'),
    CLUB('♣'),
    DIAMOND('♦'),
    SPADE('♠');

    private char symbol;
    private char letter;

    public char getSymbol() {
        return symbol;
    }

    public char getLetter() {
        return letter;
    }

    private Suit(char symbol) {
        this.symbol = symbol;
        this.letter = switch (symbol) {
            case '♥' -> 'H';
            case '♣' -> 'C';
            case '♦' -> 'D';
            case '♠' -> 'S';
            default -> throw new RuntimeException("Illegal:" + symbol);
        };
    }
}
```

参数（在此示例中仅有一个）被传递给构造函数以创建单个枚举实例。由于枚举实例由 Java 运行时创建，并且不能从外部实例化，因此构造函数被声明为私有。

枚举具有一些特殊属性：

+   所有（隐式地）扩展`java.lang.Enum`

+   可能不是泛型的

+   可能实现接口

+   不能被扩展

+   如果所有枚举值提供实现主体，则可能只有抽象方法。

+   可能不能直接通过`new`实例化

## 注释

注释是一种特殊的接口，顾名思义，用于注释 Java 程序的某些部分。

例如，考虑`@Override`注释。您可能在一些早期的示例中的一些方法上看到了它，并且可能提出了以下问题：它是做什么的？

简短的，也许令人惊讶的答案是它根本不起作用。

简短的答案是，与所有注释一样，它没有直接影响，而是作为有关所注释的方法的附加信息；在这种情况下，它表示一个方法覆盖了超类方法。

这对编译器和集成开发环境（IDE）来说是一个有用的提示——如果开发人员拼错了一个意图作为超类方法的覆盖的方法的名称，那么在拼错的方法上存在`@Override`注释（它不覆盖任何内容）会提示编译器有些地方不对。

注释，如最初的构思，不应改变程序语义；相反，它们应该提供可选的元数据。在最严格的意义上，这意味着它们不应影响程序执行，而应该只为编译器和其他执行前阶段提供信息。

在实践中，现代 Java 应用程序广泛使用注释，现在包括许多使用情况，实际上使带注释的类在没有额外运行时支持的情况下无法使用。

例如，带有诸如`@Inject`、`@Test`或`@Autowired`之类的注释的类在适当的容器之外实际上不能使用。因此，很难说此类注释不违反“没有语义意义”规则。

平台在`java.lang`中定义了一小部分基本注释。最初的集合是`@Override`、`@Deprecated`和`@SuppressWarnings`，它们用于指示方法已被覆盖、已过时或生成了一些应该被抑制的编译器警告。

Java 7 中通过 `@SafeVarargs` 扩展了这些（为可变参数方法提供了扩展警告抑制），Java 8 中通过 `@FunctionalInterface` 进行了扩展。

这个最后的注解表明一个接口可以作为 lambda 表达式的目标使用 — 虽然不是强制的，我们会看到它是一个有用的标记注解。

注解与常规接口相比具有一些特殊的属性：

+   所有（隐式）扩展 `java.lang.annotation.Annotation`

+   不得是泛型的

+   不得扩展任何其他接口

+   只能定义零参数方法

+   不得定义抛出异常的方法

+   对方法的返回类型有限制

+   方法可以有默认返回值

在实践中，注解通常没有太多功能，而是一个相当简单的语言概念。

## 定义自定义注解

为了在自己的代码中使用定义的自定义注解类型并不困难。`@interface` 关键字允许开发人员定义新的注解类型，与使用 `class` 或 `interface` 类似。

###### 注意

自定义注解的关键在于使用“元注解”。这些特殊的注解出现在新（自定义）注解类型的定义中。

元注解定义在 `java.lang.annotation` 中，并允许开发人员指定新注解类型的使用策略，以及编译器和运行时的处理方式。

创建新注解类型时需要两个主要的元注解 — `@Target` 和 `@Retention`，它们都接受枚举表示的值。

`@Target` 元注解指示新的自定义注解可以在 Java 源代码中合法放置的位置。枚举 `ElementType` 包含可能的取值 `TYPE`, `FIELD`, `METHOD`, `PARAMETER`, `CONSTRUCTOR`, `LOCAL_VARIABLE`, `ANNOTATION_TYPE`, `PACKAGE`, `TYPE_PARAMETER`, 和 `TYPE_USE`，注解可以指示它们意图在一个或多个位置使用。

另一个元注解是 `@Retention`，它指示 `javac` 和 Java 运行时如何处理自定义注解类型。它可以有三个值，由枚举 `RetentionPolicy` 表示：

`SOURCE`

具有此保留策略的注解在编译时由 `javac` 丢弃。

`CLASS`

这意味着注解将出现在类文件中，但不一定可以通过 JVM 在运行时访问。这很少使用，但有时在对 JVM 字节码进行离线分析的工具中可见。

`RUNTIME`

这表明注解将可供用户代码在运行时访问（通过反射）。

让我们看一个例子，一个简单的注解称为 `@Nickname`，允许开发人员为方法定义一个昵称，然后可以在运行时通过反射找到该方法：

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Nickname {
    String[] value() default {};
}
```

定义注解所需的全部内容只是一个语法元素，用于注解可以出现的位置，保留策略和元素的名称。由于我们需要能够提供方法的昵称，我们还需要在注解上定义一个方法。尽管如此，定义新的自定义注解是一项非常紧凑的工作。

除了两个主要的元注解外，还有`@Inherited`和`@Documented`元注解。这两个在实践中遇到的频率要低得多，详细信息可以在平台文档中找到。

## 类型注解

随着 Java 8 的发布，`ElementType`新增了两个新的取值：`TYPE_PARAMETER`和`TYPE_USE`。这些新的取值允许在以前不合法的地方使用注解，比如任何类型使用的地方。这使得开发人员可以编写如下代码：

```java
@NotNull String safeString = getMyString();
```

`@NotNull`传递的额外类型信息可以被特殊类型检查器用于检测问题（例如可能的`NullPointerException`），并执行额外的静态分析。基本的 Java 8 发行版附带了一些基本的可插拔类型检查器，但它也提供了一个框架，允许开发人员和库作者创建他们自己的类型检查器。

在本节中，我们已经接触了 Java 的枚举和注解类型。让我们继续考虑 Java 类型系统的下一个重要部分：lambda 表达式。

# Lambda 表达式

Java 8 最令人期待的功能之一是引入了 Lambda 表达式（通常简称为 lambda）。

这次 Java 平台的重大升级由五个目标驱动，大致按优先级降序排列：

+   更有表现力的编程

+   更好的库

+   简洁的代码

+   提升的编程安全性

+   潜在的增加数据并行性

Lambda 具有三个关键方面，帮助定义该特性的基本特性：

+   它们允许在程序中将小段代码以字面量形式内联编写。

+   它们通过使用类型推断放宽了 Java 代码的严格语法。

+   它们促进了更加功能化的 Java 编程风格。

正如我们在第二章中看到的，lambda 表达式的语法是取参数列表（其类型通常是推断出来的），并将其附加到方法体，就像这样：

```java
(p, q) -> { /* method body */ }
```

这可以提供一种非常紧凑的方式来表示实际上是单一方法的内容。这也是与早期版本的 Java 的一个重大变化 —— 目前为止，我们总是需要一个类声明，然后是一个完整的方法声明，所有这些都增加了代码的冗长。

事实上，在 lambda 出现之前，模拟这种编码风格的唯一方法是使用*匿名类*，我们将在本章后面讨论。然而，自 Java 8 以来，lambda 表达式在 Java 程序员中非常受欢迎，并且现在大多数情况下已经取代了匿名类的角色。

###### 注意

尽管 Lambda 表达式与匿名类之间有相似之处，但 Lambda 并不仅仅是匿名类的语法糖。事实上，Lambda 使用方法句柄（我们将在 第十一章 中遇到）和一个名为 `invokedynamic` 的特殊 JVM 字节码实现。

Lambda 表达式代表创建特定类型的对象。创建的实例类型称为 Lambda 的 *目标类型*。

只有特定类型才能作为 Lambda 的目标。

目标类型也称为 *功能接口*，它们必须：

+   必须是接口

+   只能有一个非默认方法（但可以有其他默认方法）

一些开发人员也喜欢使用 *单一抽象方法*（或 SAM）类型来指代 Lambda 转换的接口类型。这突出了一个事实，即要能够使用 Lambda 表达式机制，接口必须只有一个非默认方法。

###### 注意

Lambda 表达式几乎具备方法的所有组成部分，唯一的异常是 Lambda 没有名称。事实上，许多开发人员喜欢将 Lambda 视为“匿名方法”。

因此，这意味着单行代码：

```java
Runnable r  = () -> System.out.println("Hello");
```

并不执行 `println()`，而是创建一个对象，该对象赋值给变量 `r`，类型为 `Runnable`。这个对象 `r` 将执行 `println()` 语句，但只有在调用 `r.run()` 时才执行，而不是立即执行。

## Lambda 表达式转换

当 `javac` 遇到 Lambda 表达式时，它将其解释为具有特定签名的方法体——但是哪个方法？

要解决这个问题，`javac` 查看周围的代码。为了合法的 Java 代码，Lambda 表达式必须满足以下属性：

+   Lambda 必须出现在期望接口类型的实例位置上。

+   预期的接口类型应该有且仅有一个强制方法。

+   预期的接口方法应该具有与 Lambda 表达式完全匹配的签名。

如果是这样，那么将创建一个实现预期接口并将 Lambda 体作为强制方法实现的类型的实例。

这种稍微复杂的转换方法源于希望保持 Java 的类型系统纯粹 *名义*（基于名称）。Lambda 表达式被称为 *转换* 成正确接口类型的实例。

从这个讨论中，我们可以看到，虽然 Java 8 添加了 Lambda 表达式，但它们被专门设计为适应 Java 现有的类型系统——这个系统非常强调名义类型（而不是其他一些编程语言中可能存在的类型）。

让我们考虑 lambda 转换的一个例子——`java.io.File`类的`list()`方法。此方法列出目录中的文件。在返回列表之前，它会将每个文件的名称传递给程序员必须提供的`FilenameFilter`对象。这个`FilenameFilter`对象接受或拒绝每个文件，是`java.io`包中定义的 SAM 类型之一。

```java
@FunctionalInterface
public interface FilenameFilter {
    boolean accept(File dir, String name);
}
```

类型`FilenameFilter`携带了`@FunctionalInterface`注解，以指示它是一个适合作为 lambda 目标类型的合适类型。然而，此注解并非必需，任何符合要求的类型（通过是接口且为 SAM 类型）都可以用作目标类型。

这是因为在 Java 8 发布之前，JDK 和现有的 Java 代码库已经拥有大量的 SAM 类型。要求潜在的目标类型携带注解会阻止将 lambda 适配到现有代码中，但并没有真正的好处。

###### 提示

在您编写的代码中，您应该始终尝试指示您的类型可用作目标类型，这可以通过为它们添加`@FunctionalInterface`来实现。这有助于提高可读性，并且可以帮助一些自动化工具。

下面是如何定义一个`FilenameFilter`类，以仅列出那些文件名以*.java*结尾的文件，使用 lambda：

```java
File dir = new File("/src");      // The directory to list

String[] filelist = dir.list((d, fName) -> fName.endsWith(".java"));
```

对于列表中的每个文件，将评估 lambda 表达式中的代码块。如果方法返回`true`（如果文件名以*.java*结尾），则该文件将包含在输出中，最终存储在数组`filelist`中。

这种模式，其中一个代码块用于测试容器中的元素是否满足条件，并且仅返回通过条件的元素，被称为*过滤习语*。这是函数式编程的标准技术之一，我们将很快更深入地讨论它。

## 方法引用

请回忆，我们可以将 lambda 表达式视为代表没有名称的方法的对象。现在，请考虑这个 lambda 表达式：

```java
// In real code this would probably be
// shorter because of type inference
(MyObject myObj) -> myObj.toString()
```

这将自动转换为实现`@FunctionalInterface`类型的实现，该类型具有一个非默认方法，接受一个`MyObject`并返回一个`String`，具体来说，是通过在`MyObject`实例上调用`toString()`获取的字符串。然而，这似乎是过度样板代码，因此 Java 8 提供了一种语法以使其更易于阅读和编写：

```java
MyObject::toString
```

这种简写称为*方法引用*，它使用现有方法作为 lambda 表达式。方法引用语法与作为 lambda 表达式表示的先前形式完全等效。可以将其视为使用现有方法但忽略方法名称，因此它可以用作 lambda，然后以通常的方式自动转换。Java 定义了四种方法引用类型，这等效于四种略有不同的 lambda 表达式形式（见 Table 4-1）。

表格 4-1\. 方法引用

| 名称 | 方法引用 | 等效的 lambda |
| --- | --- | --- |
| 未绑定 | `Trade::getPrice` | `trade -> trade.getPrice()` |
| 绑定 | `System.out::println` | `s -> System.out.println(s)` |
| 静态 | `System::getProperty` | `key -> System.getProperty(key)` |
| 构造函数 | `Trade::new` | `price -> new Trade(price)` |

我们最初引入的形式可以看作是一个 *未绑定的方法引用*。当我们使用未绑定的方法引用时，它等同于一个期望包含方法引用的类型实例的 lambda 表达式—​在 Table 4-1 中，这是一个 `Trade` 对象。

它被称为未绑定的方法引用，因为接收对象需要在使用方法引用时提供（作为 lambda 的第一个参数）。也就是说，我们将在某个 `Trade` 对象上调用 `getPrice()`，但方法引用的提供者尚未定义具体是哪一个。这由引用的使用者决定。

相比之下，*绑定的方法引用* 总是将接收者作为方法引用的实例化的一部分。在 Table 4-1 中，接收者是 `System.out`，因此在使用引用时，`println()` 方法将始终在 `System.out` 上调用，并且 lambda 的所有参数都将作为 `println()` 方法的参数使用。

我们将在下一章节更详细地讨论方法引用与 lambda 表达式的使用场景。

## 函数式编程

Java 从根本上来说是一种面向对象的语言。然而，随着 lambda 表达式的到来，编写接近函数式编程风格的代码变得更加容易。

###### 注意

没有一个确切的定义可以说明什么是 *函数式语言* —— 但至少有共识认为，它应该至少包含将函数表示为可以放入变量中的值的能力。

自从版本 1.1 以来，Java 一直能够通过内部类来表示函数（参见下一节），但语法复杂且缺乏清晰度。Lambda 表达式极大地简化了这种语法，因此很自然地，更多的开发人员将寻求在其 Java 代码中使用函数式编程的方面。

Java 开发人员可能会遇到的第一次函数式编程尝试是三种基本习语，这些习语非常实用：

`map()`

映射习语通常与列表和类列表容器一起使用。其思想是传入一个应用于集合中每个元素的函数，并创建一个由将该函数应用于每个元素的结果组成的新集合。这意味着映射习语可以将一个类型的集合转换为可能是不同类型的新集合。

`filter()`

当我们讨论如何用 lambda 替换`FilenameFilter`的匿名实现时，我们已经见过 filter 惯用语的一个例子。该 filter 惯用语用于基于某些选择条件生成集合的新子集。请注意，在函数式编程中，通常生成新集合而不是就地修改现有集合是正常的。

`reduce()`

reduce 惯用语有几种不同的形式。它是一个聚合操作，也可以称为*fold*、*accumulate*或*aggregate*，以及 reduce。其基本思想是使用初始值和聚合（或缩减）函数，逐个应用缩减函数于每个元素，通过一系列中间结果（类似于“运行总计”）构建整个集合的最终结果，当 reduce 操作遍历集合时。

Java 具有对这些关键函数惯用语（和其他几种）的全面支持。具体实现在第八章中有详细解释，我们在那里讨论了 Java 的数据结构和集合，特别是*stream*抽象，这使得所有这些都成为可能。

让我们在这个介绍中总结一些警告。值得注意的是，Java 最好被视为对“稍微函数式编程”的支持。它不是特别函数式的语言，也没有尝试成为一个。Java 的一些特定方面反对它成为函数式语言的任何主张，包括：

+   Java 没有结构类型，这意味着没有“真正”的函数类型。每个 lambda 都会自动转换为相应的目标类型。

+   类型擦除对函数式编程造成问题——对于高阶函数，类型安全可能会丢失。

+   Java 本质上是可变的（正如我们将在第六章中讨论的那样）—对于函数式语言来说，可变性通常被认为是非常不可取的。

+   Java 集合是命令式的，而不是函数式的。集合必须转换为流才能使用函数式风格。

尽管如此，易于访问函数式编程的基础知识——特别是 map、filter 和 reduce 等惯用语——对 Java 社区来说是一大步向前的。这些惯用语非常有用，以至于大多数 Java 开发人员永远不需要或错过具有更彻底函数式血统的语言提供的更高级功能。

事实上，许多这些技术使用嵌套类型是可能的（请参阅下一节的详细信息），通过诸如回调和处理程序之类的模式，但是语法总是相当繁琐的，特别是在你需要仅表达单行代码的回调时，你必须显式定义一个全新的类型。

## 词法作用域和局部变量

局部变量在定义其*作用域*的代码块内部定义，在该作用域之外，无法访问局部变量并且停止存在。只有在定义块边界的花括号内的代码可以使用该块中定义的局部变量。这种作用域被称为*词法作用域*，它只定义了可以使用变量的源代码部分。

程序员通常将这样的作用域视为*临时*，即将局部变量视为从 JVM 开始执行块到控制退出块的时间存在。这通常是一种合理的局部变量及其作用域的思考方式。然而，lambda 表达式（以及稍后将遇到的匿名和本地类）有能力弯曲或打破这种直觉。

这可能导致一些开发人员最初感到惊讶的效果。因为 lambda 可以使用局部变量，它们可以包含来自不存在的词法范围的值的副本。这可以在以下代码中看到：

```java
public interface IntHolder {
    public int getValue();
}

public class Weird {
    public static void main(String[] args) {
        IntHolder[] holders = new IntHolder[10];
        for (int i = 0; i < 10; i++) {
            final int fi = i;

            holders[i] = () -> {
                return fi;
            };
        }
  // The lambda is now out of scope, but we have 10 valid instances
  // of the class the lambda has been converted to in our array.
  // The local variable fi is not in our scope here, but is still
  // in scope for the getValue() method of each of those 10 objects.
  // So call getValue() for each object and print it out.
  // This prints the digits 0 to 9.
        for (int i = 0; i < 10; i++) {
            System.out.println(holders[i].getValue());
        }
    }
}
```

每个 lambda 实例都有一个自动创建的私有副本每个使用的最终局部变量，因此实际上它有其自己的私有副本在创建时存在的作用域。这有时被称为*captured*变量。

捕获变量这样的 lambda 称为*closures*，而这些变量被称为*closed over*。

###### 警告

其他编程语言对闭包的定义可能略有不同。事实上，一些理论家会质疑 Java 的机制是否算得上闭包，因为技术上来说，被捕获的是变量的内容（一个值），而不是变量本身。

实际上，前述闭包示例比实际需要的更冗长，有两种不同的方式：

+   Lambda 有一个明确的作用域`{}`和`return`语句。

+   变量`fi`明确声明为`final`。

编译器`javac`帮助处理这两种情况。

只返回单个表达式值的 lambda 不需要包括作用域或者`return`；相反，lambda 的主体只是表达式，不需要花括号。在我们的示例中，我们明确地包含了花括号和`return`语句，以阐明 lambda 正在定义其自身的作用域。

在 Java 早期版本中，关闭变量时有两个严格的要求：

+   在捕获后，被捕获的变量不能被修改（例如，在 lambda 之后）。

+   被捕获的变量必须声明为`final`。

然而，在最近的 Java 版本中，`javac`可以分析代码并检测程序员是否尝试在 lambda 的范围之后修改捕获的变量。如果没有，则可以省略对捕获变量的`final`修饰符（这样的变量被称为*effectively final*）。如果省略了`final`修饰符，则试图在 lambda 范围之后修改捕获变量将导致编译时错误。

这是因为 Java 通过将变量内容的位模式复制到闭包创建的范围来实现闭包。对于封闭变量内容的进一步更改不会反映在闭包范围中的副本中，因此设计决策是使这些更改非法并在编译时错误。

这些来自`javac`的辅助功能意味着我们可以将前面示例的内部循环重写为非常紧凑的形式：

```java
for (int i = 0; i < 10; i++) {
    int fi = i;
    holders[i] = () -> fi;
}
```

闭包在某些编程风格中非常有用，不同的编程语言以不同的方式定义和实现闭包。Java 将闭包实现为 lambda 表达式，但本地类和匿名类也可以捕获状态——实际上这是 Java 在 lambda 可用之前实现闭包的方式。

# 嵌套类型

在本书中到目前为止所见的类、接口和枚举类型都被定义为*顶级类型*。这意味着它们是包的直接成员，独立于其他类型之外定义。然而，类型定义也可以嵌套在其他类型定义之内。这些*嵌套类型*，通常被称为“内部类”，是 Java 语言的一个强大特性。

一般来说，嵌套类型用于两个不同的目的，都与封装相关。首先，类型可能被嵌套，因为它需要特别亲密地访问另一个类型的内部。作为嵌套类型，它以与成员变量和方法相同的方式访问。这意味着嵌套类型具有特权访问权限，可以被视为“略微违反封装规则”。

对于嵌套类型的此用例的另一种思考方式是，它们是与另一个类型紧密联系的类型。这意味着它们实际上并没有完全独立的实体存在，只是共存。

或者，类型可能仅仅是因为一个非常特定的原因在代码的一个非常小的部分中需要。这意味着它应该被紧密地局部化，因为它实际上是实现细节的一部分。

在较早版本的 Java 中，这样做的唯一方法是使用嵌套类型，例如接口的匿名实现。实际上，随着 Java 8 的出现，这种用例已经大大被 lambda 表达式取代。将匿名类型作为紧密局部化类型的使用在某些情况下仍然存在，但显著下降。

类型可以以四种不同的方式嵌套在另一个类型中：

静态成员类型

静态成员类型是定义为另一个类型的`static`成员的任何类型。嵌套接口、枚举和注解始终是静态的（即使您没有使用关键字）。

非静态成员类

“非静态成员类型”简单地是不声明为`static`的成员类型。只有类可以是非静态成员类型。

本地类

本地类是在 Java 代码块内定义并且仅在其中可见的类。接口、枚举和注解不能在本地定义。

匿名类

匿名类是一种没有对人类有意义的有意义名称的本地类；它仅仅是编译器分配的任意名称，程序员不应直接使用。接口、枚举和注解不能匿名定义。

“嵌套类型”这个术语虽然准确和精确，但并不被开发人员广泛使用。相反，大多数 Java 程序员使用更模糊的术语“内部类”。根据情况，这可能指非静态成员类、本地类或匿名类，但不是静态成员类型，没有真正的区分方法。

幸运的是，尽管描述嵌套类型的术语并不总是清晰，但与其一起工作的语法通常是明显的，通常可以从上下文中看出正在讨论哪种类型的嵌套类型。

###### 注意

直到 Java 11，嵌套类型是使用编译器技巧实现的，大部分是语法糖。有经验的 Java 程序员应注意，这个细节在 Java 11 中发生了变化，不再像过去那样实现。

让我们继续详细描述四种嵌套类型中的每一种。每个部分描述了嵌套类型的特点、其使用的限制以及与该类型一起使用的任何特殊 Java 语法。

## 静态成员类型

*静态成员类型*与常规顶层类型非常相似。然而，为了方便起见，它嵌套在另一个类型的命名空间内。静态成员类型具有以下基本属性：

+   静态成员类型与类的其他静态成员（如静态字段和静态方法）类似。

+   静态成员类型与包含类的任何实例无关（即没有`this`对象）。

+   静态成员类型可以（仅）访问包含它的类的`static`成员。

+   静态成员类型可以访问其包含类型的所有`static`成员（包括任何其他静态成员类型）。

+   嵌套接口、枚举和注解无论`static`关键字是否出现，都隐式为静态。

+   任何嵌套在接口或注解中的类型也隐式为`static`。

+   静态成员类型可以在顶层类型内定义，也可以在其他静态成员类型内的任何深度嵌套。

+   静态成员类型不能在任何其他类型的嵌套类型内定义。

让我们快速看一下静态成员类型的语法示例。示例 4-1 展示了一个帮助接口作为包含接口的静态成员定义的示例，本例中为 Java 的`Map`。

##### 示例 4-1\. 定义并使用静态成员接口

```java
public interface Map<K, V> {
    // ...

    Set<Map.Entry<K, V>> entrySet();

    // All nested interfaces are automatically static
    interface Entry<K, V> {
        K getKey();
        V getValue();
        V setValue(V value);

        // other members elided
    }

    // other members elided
}
```

当被外部类使用时，`Entry`将通过其层次名称`Map.Entry`来引用。

### 静态成员类型的特点

静态成员类型可以访问其包含类型的所有静态成员，包括`private`成员。反之亦然：包含类型的方法可以访问静态成员类型的所有成员，包括这些类型的`private`成员。静态成员类型甚至可以访问任何其他静态成员类型的所有成员，包括这些类型的`private`成员。静态成员类型可以使用任何其他静态成员，而无需使用包含类型的名称限定其名称。

顶层类型可以声明为`public`或包私有（如果它们没有使用`public`关键字声明）。但将顶层类型声明为`private`和`protected`并没有太大意义——`protected`只是意味着与包私有相同，而`private`顶层类无法被任何其他类型访问。

另一方面，静态成员类型是成员，因此可以使用与包含类型的其他成员相同的任何访问控制修饰符。对于静态成员类型，这些修饰符的含义与类型的其他成员相同。

在大多数情况下，类名使用`Outer.Inner`语法可以很好地提醒内部类与其包含类型的互联性。但是，Java 语言允许您使用`import`指令直接导入静态成员类型：

```java
import java.util.Map.Entry;
```

然后，您可以引用嵌套类型，而无需包含其封闭类型的名称（例如，只需像`Entry`这样）。

###### 注意

您还可以使用`import static`指令导入静态成员类型。有关`import`和`import static`的详细信息，请参见第二章中的“包和 Java 命名空间”。

然而，导入嵌套类型会掩盖该类型与其包含类型紧密关联的事实——这通常是重要信息，并因此不常见。

## 非静态成员类

*非静态成员类*是一种声明为包含类或枚举类型的成员的类，没有`static`关键字：

+   如果静态成员类型类比于类字段或类方法，那么非静态成员类类比于实例字段或实例方法。

+   只有类可以是非静态成员类型。

+   非静态成员类的实例始终与封闭类型的实例相关联。

+   非静态成员类的代码可以访问其封闭类型的所有字段和方法（包括`static`和非`static`）。

+   Java 语法具有几个特定功能，专门用于处理非静态成员类的封闭实例。

示例 4-2 展示了如何定义和使用成员类。这个例子展示了一个`LinkedStack`的例子：它定义了一个嵌套接口，描述了堆栈底层的链表节点，并且定义了一个嵌套类来允许对堆栈上的元素进行枚举。成员类定义了`java.util.Iterator`接口的一个实现。

##### 示例 4-2\. 作为成员类实现的迭代器

```java
import java.util.Iterator;

public class LinkedStack {

    // Our static member interface
    public interface Linkable {
        public Linkable getNext();
        public void setNext(Linkable node);
    }

    // The head of the list
    private Linkable head;

    // Method bodies omitted here
    public void push(Linkable node) { ... }
    public Linkable pop() { ... }

    // This method returns an Iterator object for this LinkedStack
    public Iterator<Linkable> iterator() { return new LinkedIterator(); }

    // Here is the implementation of the Iterator interface,
    // defined as a nonstatic member class.
    protected class LinkedIterator implements Iterator<Linkable> {
        Linkable current;

        // The constructor uses a private field of the containing class
        public LinkedIterator() { current = head; }

        // The following three methods are defined
        // by the Iterator interface
        public boolean hasNext() {  return current != null; }

        public Linkable next() {
            if (current == null)
              throw new java.util.NoSuchElementException();
            Linkable value = current;
            current = current.getNext();
            return value;
        }

        public void remove() { throw new UnsupportedOperationException(); }
    }
}
```

注意`LinkedIterator`类如何嵌套在`LinkedStack`类内部。`LinkedIterator`是一个仅在`LinkedStack`内部使用的辅助类，因此在包含类使用它的地方定义它可以产生清晰的设计。

### 成员类的特性

像实例字段和实例方法一样，每个非静态成员类的实例都与其定义的包含类的实例关联。这意味着成员类的代码可以访问包含实例的所有实例字段和实例方法（以及`static`成员），包括任何声明为`private`的成员。

这一关键特性已经在示例 4-2 中进行了演示。这里再次展示了`LinkedStack.LinkedIterator()`构造函数：

```java
public LinkedIterator() { current = head; }
```

这行代码将内部类的`current`字段设置为包含类的`head`字段的值。尽管在包含类中，`head`声明为`private`字段，代码如所示仍然可以工作。

非静态成员类，像类的任何成员一样，可以被分配标准访问控制修饰符之一。在示例 4-2 中，`LinkedIterator`类声明为`protected`，因此对使用`LinkedStack`类的代码（在不同包中）是不可访问的，但对任何子类化`LinkedStack`的类是可访问的。

成员类有两个重要的限制：

+   非静态成员类不能与任何包含类或包具有相同的名称。这是一个重要的规则，不同于字段和方法。

+   非静态成员类不能包含任何`static`字段、方法或类型，除了被声明为`static`和`final`的常量字段。

### 成员类的语法

成员类最重要的特性是它可以访问其包含对象的实例字段和方法。

如果我们想要使用显式引用，并使用`this`，那么我们必须使用一种特殊的语法来显式地引用`this`对象的包含实例。例如，如果我们在构造函数中想要显式地表示，我们可以使用以下语法：

```java
public LinkedIterator() { this.current = LinkedStack.this.head; }
```

通用语法是*`classname.this`*，其中*`classname`*是包含类的名称。请注意，成员类本身可以包含成员类，嵌套到任意深度。

然而，没有任何成员类可以与任何包含类具有相同的名称，因此，在`this`之前加上包含类名称是引用任何包含实例的一种完全通用的方式。换句话说，`EnclosingClass.this`的语法构造是引用包含实例的一种明确方式，称为*上级引用*。

## 本地类

*本地类*是在 Java 代码块内部声明的类，而不是类的成员。只有类可以在本地定义：接口、枚举类型和注释类型必须是顶级或静态成员类型。通常，本地类在方法内部定义，但也可以在类的静态初始化器或实例初始化器内定义。

正如所有 Java 代码块都出现在类定义内部一样，所有本地类都嵌套在包含块内部。因此，尽管本地类与成员类共享许多特性，通常更合适的是将它们视为一种完全不同的嵌套类型。

###### 注意

详见第 5 章，了解何时适合选择本地类而不是 lambda 表达式。

本地类的定义特点是它仅在代码块的范围内有效。示例 4-3 演示了如何修改`LinkedStack`类的`iterator()`方法，使其将`LinkedIterator`定义为本地类而不是成员类。

这样做可以将类的定义更接近其使用位置，从而进一步提高代码的清晰度。为简洁起见，示例 4-3 仅显示了`iterator()`方法，而不是包含它的整个`LinkedStack`类。

##### 示例 4-3\. 定义和使用本地类

```java
// This method returns an Iterator object for this LinkedStack
public Iterator<Linkable> iterator() {
    // Here's the definition of LinkedIterator as a local class
    class LinkedIterator implements Iterator<Linkable> {
        Linkable current;

        // The constructor uses a private field of the containing class
        public LinkedIterator() { current = head; }

        // The following three methods are defined
        // by the Iterator interface
        public boolean hasNext() {  return current != null; }

        public Linkable next() {
            if (current == null)
              throw new java.util.NoSuchElementException();
            Linkable value = current;
            current = current.getNext();
            return value;
        }

        public void remove() { throw new UnsupportedOperationException(); }
    }

    // Create and return an instance of the class we just defined
    return new LinkedIterator();
}
```

### 本地类的特性

本地类具有以下有趣的特性：

+   像成员类一样，本地类与包含实例关联，并且可以访问包含类的任何成员，包括`private`成员。

+   除了访问包含类定义的字段外，本地类还可以访问任何本地方法定义作用域内的局部变量、方法参数或异常参数，并且这些变量必须声明为`final`。

本地类受以下限制：

+   本地类的名称仅在定义它的块内部有效；它永远不能在该块外部使用。（注意，但是，在类的范围内创建的本地类的实例可以继续存在于该范围之外。本节稍后将详细描述这种情况。）

+   本地类不能声明为`public`、`protected`、`private`或`static`。

+   与成员类一样，由于同样的原因，局部类不能包含`static`字段、方法或类。唯一的例外是同时声明为`static`和`final`的常量。

+   接口、枚举类型和注解类型不能在局部定义。

+   与成员类一样，局部类也不能与其封闭类的任何名称相同。

+   正如前面提到的，局部类可以关闭作用域内的局部变量、方法参数甚至异常参数，但前提是这些变量或参数是有效地`final`。

### 局部类的作用域

在讨论非静态成员类时，我们看到成员类可以访问从超类继承的任何成员以及由其包含的类定义的任何成员。

对于局部类也是如此，但局部类还可以像 Lambda 一样访问有效的`final`局部变量和参数。示例 4-4 展示了局部类（或 Lambda）可以访问的不同类型的字段和变量。

##### 示例 4-4\. 局部类可访问的字段和变量

```java
class A { protected char a = 'a'; }
class B { protected char b = 'b'; }

public class C extends A {
  private char c = 'c';         // Private fields visible to local class
  public static char d = 'd';
  public void createLocalObject(final char e)
  {
    final char f = 'f';
    int i = 0;                  // i not final; not usable by local class
    class Local extends B
    {
      char g = 'g';
      public void printVars()
      {
        // All of these fields and variables are accessible to this class
        System.out.println(g);  // (this.g) g is a field of this class
        System.out.println(f);  // f is a final local variable
        System.out.println(e);  // e is a final local parameter
        System.out.println(d);  // (C.this.d) d field of containing class
        System.out.println(c);  // (C.this.c) c field of containing class
        System.out.println(b);  // b is inherited by this class
        System.out.println(a);  // a is inherited by the containing class
      }
    }
    Local l = new Local();      // Create an instance of the local class
    l.printVars();              // and call its printVars() method.
  }
}
```

因此，局部类具有相当复杂的作用域结构。要了解原因，请注意，局部类的实例的生命周期可以延伸到 JVM 退出定义局部类的块之后。

###### 注意

换句话说，如果您创建了局部类的实例，那么当 JVM 完成定义类的块的执行时，该实例不会自动消失。因此，即使类的定义是局部的，该类的实例也可以逃离其定义的位置。

因此，局部类在许多方面的行为类似于 Lambda，尽管局部类的用例比 Lambda 更通用。然而，在实践中，很少需要额外的通用性，并且尽可能使用 Lambda。

## 匿名类

*匿名类*是一种没有名称的局部类。它在一个表达式中使用`new`运算符进行定义和实例化。虽然局部类定义是 Java 代码块中的语句，但匿名类定义是一个表达式，这意味着它可以作为较大表达式的一部分，例如方法调用。

###### 注意

为了完整起见，我们在这里涵盖了匿名类，但对于大多数用例，Lambda 表达式（参见“Lambda 表达式”）已经取代了匿名类。

请参考示例 4-5，它展示了`LinkedIterator`类作为`LinkedStack`类的`iterator()`方法内的匿名类实现。与示例 4-4 进行比较，它展示了相同的类作为局部类实现。

##### 示例 4-5\. 使用匿名类实现的枚举

```java
public Iterator<Linkable> iterator() {
    // The anonymous class is defined as part of the return statement
    return new Iterator<Linkable>() {
        Linkable current;
        // Replace constructor with an instance initializer
        { current = head; }

        // The following three methods are defined
        // by the Iterator interface
        public boolean hasNext() {  return current != null; }
        public Linkable next() {
            if (current == null)
              throw new java.util.NoSuchElementException();
            Linkable value = current;
            current = current.getNext();
            return value;
        }
        public void remove() { throw new UnsupportedOperationException(); }
    };  // Note the required semicolon. It terminates the return statement
}
```

如您所见，定义匿名类并创建该类的实例的语法使用 `new` 关键字，后跟类型名称和用大括号括起的类体定义。如果 `new` 关键字后面的名称是类的名称，则匿名类是指定类的子类。如果 `new` 后面的名称指定了一个接口，就像前两个示例中一样，匿名类实现该接口并扩展 `Object`。

###### 注意

匿名类的语法特意不包括任何指定 `extends` 子句、`implements` 子句或类名的方式。

因为匿名类没有名称，所以不可能在类体中为其定义构造函数。这是匿名类的基本限制之一。在匿名类定义中紧随超类名称后的括号内指定的任何参数都会隐式传递给超类构造函数。匿名类通常用于子类化不需要任何构造函数参数的简单类，因此匿名类定义语法中的括号经常是空的。

因为匿名类只是一种局部类的类型，匿名类和局部类共享相同的限制。匿名类不能定义任何 `static` 字段、方法或类，除了 `static final` 常量。接口、枚举类型和注解类型不能匿名定义。此外，像局部类一样，匿名类不能是 `public`、`private`、`protected` 或 `static`。

定义匿名类的语法将定义与实例化结合在一起，类似于 lambda 表达式。如果每次执行包含块时需要创建多个类的实例，则不适合使用匿名类而应使用局部类。

# 描述 Java 类型系统

到目前为止，我们已经涵盖了 Java 类型系统的所有主要方面，因此我们可以对其进行描述和表征。

Java 类型系统的最重要和显而易见的特征是它是：

+   静态

+   不是单根

+   名义上

静态类型是三个方面中最广为人知的，意味着在 Java 中，每个数据存储（如变量、字段等）都有一个类型，并且该类型在首次引入存储时声明。尝试将不兼容的值放入不支持的存储中会导致编译时错误。

Java 的类型系统不是单根的也是立即显而易见的。Java 有原始类型和引用类型。Java 中的每个对象都属于一个类，除了 `Object` 外，每个类都有一个单一的父类。这意味着任何 Java 程序中的类集合形成一个以 `Object` 为根的树结构。

然而，任何原始类型和`Object`之间都没有继承关系。因此，Java 类的整体图形由大量的引用类型树和八个不相交的孤立点（对应于原始类型）组成。这导致需要使用包装类型，如`Integer`，在必要时将原始值表示为对象（例如在 Java 集合中）。

最后一个方面，则需要更详细的讨论。

## 名义类型

在 Java 中，每种类型都有一个名称。在 Java 编程的正常过程中，这将是一个简单的字母（有时是数字）串，具有反映类型用途的语义意义。这种方法被称为*名义类型*。

并非所有语言都具有纯粹的名义类型；例如，一些语言可以表达“此类型具有特定签名方法”的概念，而无需显式引用类型名称，有时被称为*结构类型*。

例如，在 Python 中，您可以对定义了`__len__()`方法的任何对象调用`len()`。当然，Python 是一种动态类型语言，如果无法进行`len()`调用，则会引发运行时异常。但是，在静态类型语言中也可以表达类似的概念，例如 Scala。

Java 另一方面，没有办法在不使用接口的情况下表达这个想法，这当然有一个名字。Java 也严格基于继承和实现来维护类型兼容性。让我们看一个例子：

```java
@FunctionalInterface
public interface MyRunnable {
    void run();
}
```

接口`MyRunnable`有一个单一方法，与`Runnable`完全匹配。然而，这两个接口彼此没有继承或其他关系，所以像这样的代码：

```java
MyRunnable myR = () -> System.out.println("Hello");
Runnable r = (Runnable)myR;
r.run();
```

将会编译成功，但在运行时会失败并抛出`ClassCastException`。事实上，即使两个接口上存在具有相同签名的`run()`方法，编译器也不会考虑，实际上程序根本没有执行到调用`run()`的地步：它失败在前一行，即尝试进行类型转换的地方。

另一个重要的点是 Java 的整个 lambda 表达式构建，特别是将目标类型定型为函数接口，是为了确保 lambda 能够适应名义类型方法。例如，考虑这样一个接口：

```java
@FunctionalInterface
public interface MyIntProvider {
    int run() throws InterruptedException;
}
```

然后，可以在多种不同的情况下使用一个产生常量的 lambda 表达式，例如`() -> 42`：

```java
MyIntProvider prov       = () -> 42;
Supplier<Integer> sup    = () -> 42;
Callable<Integer> callMe = () -> 42;
```

从这里我们可以看到，单独的表达式`() -> 42`是不完整的。Java lambda 表达式依赖于类型推断，因此我们需要看到表达式与其目标类型结合在一起才有意义。与目标类型组合时，lambda 的类类型是“一个在编译时未知的目标接口的实现”，程序员必须将接口类型作为 lambda 的类型使用。

除了 lambda 之外，在 Java 中还有一些名义类型的边缘情况。一个例子是匿名类，但即使在这里，类型仍然有名称。然而，匿名类型的类型名称是由编译器自动生成的，并且专门选择以便 JVM 可以使用但 Java 源代码编译器不接受。

还有另一种边缘情况需要考虑，它与近期 Java 版本引入的增强类型推断有关。

## 非标记类型和`var`

从 Java 11 开始（实际上是在 Java 10 非 LTS 版本中引入），Java 开发人员可以利用新的语言特性*局部变量类型推断*（LVTI），又称`var`。这是 Java 类型推断能力的增强，可能比一开始看起来更重要。在最简单的情况下，它允许如下代码：

```java
var ls = new ArrayList<String>();
```

将推断从值的类型移到变量的类型。

实现方法是将`var`作为保留的类型名称而不是关键字。这意味着代码仍然可以将`var`用作变量、方法或包名，而不受新语法的影响。然而，先前将`var`用作类型名称的代码将需要重新编译。

这个简单的案例旨在减少冗长，并使从其他语言（特别是 Scala、.NET 和 JavaScript）转到 Java 的程序员感觉更舒适。然而，过度使用可能会模糊编写代码的意图，因此应该谨慎使用。

除了简单的情况外，`var`实际上允许了以前不可能的编程构造。为了看到差异，让我们考虑`javac`一直允许的一种非常有限的类型推断：

```java
public class Test {
    public static void main(String[] args) {
        (new Object() {
            public void bar() {
                System.out.println("bar!");
            }
        }).bar();
    }
}
```

代码将编译并运行，打印出`bar!`。这种略显反直觉的结果发生是因为`javac`保留了关于匿名类的足够类型信息（即它有一个`bar()`方法），以至于编译器可以推断调用`bar()`是有效的。

实际上，这种边缘情况自 2009 年以来就在[Java 社区中已知](https://oreil.ly/RVqng)，早在 Java 7 到来之前。

这种类型推断的问题在于它没有真正的实际应用：“带有 bar 方法的对象”的类型存在于编译器中，但是这种类型无法表达为变量的类型——它不是一个*可标记的类型*。这意味着在 Java 10 之前，这种类型的存在仅限于单个表达式，不能在更大的范围内使用。

然而，随着 LVTI 的到来，变量的类型并不总是需要显式指定。相反，我们可以使用`var`来允许我们通过避免指定类型来保留静态类型信息。

这意味着现在我们可以修改我们的示例并编写：

```java
var o = new Object() {
    public void bar() {
        System.out.println("bar!");
    }
};

o.bar();
```

这使得我们能够在单个表达式之外保留`o`的真实类型。`o`的类型不能被指定，因此它不能作为方法参数或返回类型的类型出现。这意味着类型仍然仅限于单个方法，但仍然可以用于表达某些在其他情况下会很尴尬或不可能的结构。

将`var`用作“魔术类型”允许程序员为每个`var`的不同使用保留类型信息，这在某种程度上类似于 Java 泛型的有界通配符。

更高级的`var`用法与非注记类型[是可能的](https://oreil.ly/p0w-a)。虽然这个特性不能满足所有对 Java 类型系统的批评，但它确实代表了一个明确（尽管谨慎）的进步步骤。

# 概要

通过分析 Java 的类型系统，我们已经能够建立起 Java 平台对数据类型的世界观的清晰图景。Java 的类型系统可以被描述为：

静态的

所有的 Java 变量在编译时都有已知的类型。

名义上的

Java 类型的名称至关重要。Java 不允许结构类型，并且对于非注记类型的支持有限。

面向对象/命令式

Java 代码是面向对象的，所有的代码必须存在于方法中，方法必须存在于类中。然而，Java 的原始类型阻止了对“一切皆对象”的完全采纳。

稍微具有函数式特征

Java 提供对一些常见的函数式习语的支持，但更多作为程序员的便利而非其他。

类型推断

Java 优化了代码的可读性（即使是对初学者），并倾向于显式声明，但在不影响代码可读性的情况下使用类型推断来减少样板代码。

强大的向后兼容性

Java 主要是面向业务的语言，向后兼容性和保护现有代码库是非常高的优先事项。

类型擦除

Java 允许参数化类型，但这些信息在运行时不可用。

Java 的类型系统经过多年的演进（尽管缓慢而谨慎），现在与其他主流编程语言的类型系统处于同一水平。Lambda 表达式与默认方法一起，代表了自 Java 5 问世以来最大的转变，以及泛型、注解及相关创新的引入。

默认方法代表了 Java 面向对象编程方法的一个重大转变，也许是自语言问世以来最大的转变。从 Java 8 开始，接口可以包含实现代码。这从根本上改变了 Java 的性质。此前是单继承语言的 Java，现在在行为上可以多继承（但仅限于行为，状态仍然不支持多继承）。

尽管有所有这些创新，Java 的类型系统并没有（也不打算）配备类似于 Scala 或 Haskell 等语言的类型系统的强大能力。相反，Java 的类型系统在简洁性、可读性和新手学习曲线方面都倾向于简单。

过去 10 年，Java 也从其他语言中开发的类型方法中受益匪浅。Scala 作为一种静态类型语言的例子，通过使用类型推断实现了很多动态类型语言的感觉，为 Java 添加特性提供了很好的思路，即使这两种语言有着非常不同的设计理念。

仍然有一个问题是，Java 中 Lambda 表达式提供的对函数式习惯的适度支持是否足以满足大多数 Java 程序员的需求。

###### 注：

Java 的类型系统的长期发展方向正在研究项目中探索，例如 Valhalla，其中正在探索诸如数据类、模式匹配和密封类等概念。

尚待观察的是，普通 Java 程序员是否需要像 Scala 那样的高级（且远非名义上的）类型系统所带来的更大能力——以及随之而来的复杂性，还是 Java 8 中引入的“稍微函数式编程”（例如 *map*、*filter*、*reduce* 等）已经足够满足大多数开发者的需求。

¹ 一些泛型的小痕迹仍然存在，可以通过反射在运行时看到。

² Raoul-Gabriel Urma 和 Janina Voigt，“使用 OpenJDK 探究 Java 中的协变”，*Java Magazine*（2012 年 5 月/6 月）：44–47。
