# 第七章\. 集合与泛型

随着我们利用日益增长的对象知识来处理更多有趣的问题，一个经常出现的问题是如何存储我们在解决这些问题过程中操作的数据？我们肯定会使用各种不同类型的变量，但我们还需要更大更复杂的存储选项。我们在“数组”章节中讨论过的数组是一个开始，但是数组有一些限制。在本章中，我们将看到如何使用 Java 集合的概念来高效、灵活地访问大量数据。我们还将看到如何处理我们想要存储在这些大容器中的各种类型的数据，就像我们处理变量中的单个值一样。这就是泛型的用武之地。我们将在“类型限制”中深入讨论它们。

# 集合

*集合*是所有类型编程中基础的数据结构。每当我们需要引用一组对象时，就会涉及某种类型的集合。在核心语言级别上，Java 通过数组支持集合。但是数组是静态的，由于长度固定，对于应用程序生命周期内增长和缩小的对象组来说显得笨拙。数组也不擅长表示对象之间的抽象关系。在早期，Java 平台仅有两个基本类来满足这些需求：`java.util.Vector`类代表动态对象列表，`java.util.Hashtable`类保存键/值对映射。如今，Java 有了更全面的方法，称为*集合框架*。该框架标准化了处理各种集合的方式。旧的类仍然存在，但已经被整合到框架中（带有一些古怪之处），通常不再使用。

虽然在概念上简单，集合是任何编程语言中最强大的部分之一。它们实现了管理复杂问题核心的数据结构。基础计算机科学致力于描述如何以最高效的方式实现某些类型的算法来操作集合。 （如何在大型集合中快速找到某物？如何对集合中的项目进行排序？如何高效地添加或删除项目？）掌握这些工具并理解如何使用它们可以使您的代码变得更小更快。它还可以避免重复造轮子。

原始的集合框架有两个主要缺陷。第一个是集合由于需要是无类型的，只能使用未区分的`Object`而不能使用特定类型如`Date`和`String`。这意味着每次从集合中取出对象时都必须进行类型转换。这与 Java 的编译时类型安全相悖。但实际上，这不是问题，只是非常繁琐和乏味。第二个问题是，出于实际原因，集合只能处理对象而不能处理原始类型。这意味着每当你想将数字或其他原始类型放入集合时，你必须首先将其存储在包装类中，然后在检索时解包。这些因素的结合使得使用集合的代码更加难以阅读和更加危险。

泛型类型（稍后在“类型限制”中详述）使得真正类型安全的集合可以由程序员控制。除了泛型，原始类型的自动装箱和拆箱意味着在涉及集合时，你通常可以将对象和原始类型视为相等。这些新特性的结合增加了一些安全性，并且可以显著减少你编写的代码量。正如我们将看到的，现在所有的集合类都利用了这些特性。

集合框架围绕`java.util`包中的少数接口展开。这些接口分为两个层次结构。第一个层次结构从`Collection`接口派生。这个接口（及其子类）代表一个容器，用来保存其他对象。第二个独立的层次结构基于`Map`接口，另一个容器，表示一组键值对，其中键可以用来以高效的方式检索值。

## **集合接口**

所有集合的鼻祖是一个名为`Collection`的接口。它作为容器来保存其他对象，即它的*元素*。它并不明确指定对象的组织方式；例如，它并不说明是否允许重复对象或对象是否以某种方式有序。这些细节留给子接口或实现类处理。尽管如此，`Collection`接口定义了一些对所有集合通用的基本操作：

`public boolean add(` *`element`* `)`

将提供的对象添加到此集合。如果操作成功，则此方法返回`true`。如果对象已经存在于此集合中并且集合不允许重复，则返回`false`。此外，某些集合是只读的。如果调用此方法，则这些集合会抛出`UnsupportedOperationException`。

`public boolean remove(` *`element`* `)`

从此集合中移除指定对象。类似于`add()`方法，如果从集合中移除对象，则此方法返回`true`。如果对象在此集合中不存在，则返回`false`。只读集合在调用此方法时会抛出`UnsupportedOperationException`。

`public boolean contains(` *`element`* `)`

如果集合包含指定对象，则返回`true`。

`public int size()`

返回此集合中的元素数。

`public boolean isEmpty()`

如果此集合没有元素，则返回`true`。

`public Iterator iterator()`

检查此集合中的所有元素。此方法返回一个`Iterator`，这是一个可以用来遍历集合元素的对象。我们将在下一节详细讨论迭代器。

另外，方法`addAll()`、`removeAll()`和`containsAll()`接受另一个`Collection`，并添加、移除或测试供应集合的所有元素。

## 集合类型

`Collection`接口有三个子接口。`Set`表示不允许重复元素的集合。`List`是其元素具有特定顺序的集合。`Queue`接口是具有“头”元素概念的对象缓冲区，该元素是下一个要处理的元素。

### 集合

`Set`除了从`Collection`继承的方法外，没有其他方法。它只是强制执行不允许重复的规则。如果尝试添加已存在于`Set`中的元素，则`add()`方法简单地返回`false`。`SortedSet`按照规定的顺序维护元素；类似于无法包含重复项的排序列表。您可以使用`subSet()`、`headSet()`和`tailSet()`方法检索子集（这些子集也是排序的）。这些方法接受一个或两个标记边界的元素。`first()`和`last()`调用提供对第一个和最后一个元素的访问。`comparator()`方法返回用于比较元素的对象（关于此方法的更多信息请参见“深入了解：sort() 方法”）。

`NavigableSet`扩展了`SortedSet`并添加了一些方法，用于在`Set`的排序顺序内找到大于或小于目标值的最接近匹配。您可以使用跳跃表等技术有效地实现此接口，使得查找有序元素变得更快。

### 列表

`Collection`的下一个子接口是`List`。`List`是一个有序集合，类似于数组，但具有用于操作列表中元素位置的方法：

`public boolean add(E` *`element`* `)`

将指定元素从列表末尾移除。

`public void add(int` *`index`* `, E` *`element`* `)`

在列表中指定位置插入给定对象。如果位置小于零或大于列表长度，则抛出`IndexOutOfBoundsException`。原来在指定位置的元素和其后的所有元素都将向上移动一个索引位置。

`public void remove(int` *`index`* `)`

移除指定位置的元素。所有后续元素向下移动一个索引位置。

`public E get(int` *`index`* `)`

返回给定位置的元素，但不更改列表。

`public Object set(int` *`index`* `, E` *`element`* `)`

将给定位置的元素更改为指定对象。必须已经有一个对象在索引处，否则会抛出 `IndexOutOfBoundsException`。不会影响列表的其他元素。

这些方法中的类型 `E` 是 `List` 类的参数化元素类型。`Collection`、`Set` 和 `List` 都是接口类型。这是我们在本章开头提到的泛型特性的一个示例，我们将很快看到这些类型的具体实现。

### 队列

`Queue` 是一个行为类似缓冲区的集合。队列维护放入其中的项目的插入顺序，并且有“头”项目的概念。队列可以是先进先出（FIFO 或“按顺序”）或后进先出（LIFO，有时是“最近”或“逆序”），这取决于实现：

`public boolean offer(E element)`, `public boolean add(E element)`

`offer()` 方法尝试将元素放入队列，如果成功则返回 `true`。不同的 `Queue` 类型可能对元素类型（包括容量）有不同的限制或限制。该方法与从 `Collection` 继承的 `add()` 方法不同，它返回一个布尔值而不是抛出异常以指示集合无法接受元素。

`public E` `poll()`, `public E remove()`

`poll()` 方法移除队列头部的元素并返回它。该方法与 `Collection` 的 `remove()` 方法不同，如果队列为空，则返回 `null` 而不是抛出异常。

`public E` `peek()`

返回头部元素，但不从队列中删除它。如果队列为空，则返回 `null`。

## 映射接口

集合框架还包括 `java.util.Map`，它是一组键值对的集合。映射的其他名称包括“字典”或“关联数组”。映射存储和检索具有键值的元素；它们对于像缓存和最小数据库这样的东西非常有用。将值存储在映射中时，您将一个键对象与该值关联起来。当您需要查找值时，映射使用键检索它。

使用泛型（再次出现的 `E` 类型），`Map` 类型是使用两种类型进行参数化的：一种用于键，一种用于值。以下片段使用 `HashMap`，这是一种高效但无序的映射实现类型，我们稍后会讨论它：

```java
    Map<String, Date> dateMap = new HashMap<String, Date>();
    dateMap.put("today", new Date());
    Date today = dateMap.get("today");
```

在旧代码中，映射简单地将 `Object` 类型映射到 `Object` 类型，并需要适当的类型转换来检索值。

`Map` 的基本操作很简单。在以下方法中，类型 `K` 是指键参数类型，类型 `V` 是指值参数类型：

`public V put(K` *`key`* `, V` *`value`* `)`

将指定的键/值对添加到地图中。如果地图已经包含指定键的值，则旧值将被替换并作为结果返回。

`public V get(K` *`key`* `)`

从地图中检索与`key`对应的值。

`public V remove(K` *`key`* `)`

从地图中删除与`key`对应的值。返回已删除的值。

`public int size()`

返回此地图中键/值对的数量。

使用以下方法可以检索地图中的所有键或值：

`public Set keySet()`

此方法返回一个`Set`，其中包含此地图中的所有键。

`public Collection values()`

使用此方法检索此地图中的所有值。返回的`Collection`可以包含重复的元素。

`public Set entrySet()`

此方法返回一个`Set`，该集合包含此地图中的所有键/值对（作为`Map.Entry`对象）。

`Map`有一个子接口，`SortedMap`。`SortedMap`根据键的特定顺序维护其键/值对排序。它提供了`subMap()`、`headMap()`和`tailMap()`方法来检索排序地图的子集。与`SortedSet`一样，它还提供了一个`comparator()`方法，该方法返回一个对象，该对象确定地图键的排序方式。我们将在“更近距离看：sort()方法”中详细讨论这一点。Java 7 添加了一个`NavigableMap`，其功能与`NavigableSet`类似；也就是说，它添加了搜索排序元素的方法，以查找大于或小于目标值的元素。

最后，我们应该明确指出，虽然它们相关，但`Map`不是`Collection`的一种类型（`Map`不扩展`Collection`接口）。你可能会想为什么。`Collection`接口的所有方法似乎都适用于`Map`，除了`iterator()`。再次强调，`Map`有两组对象：键和值，并且分别有迭代器。这就是为什么`Map`不实现`Collection`的原因。如果您确实想要`Map`的类似`Collection`的视图，包含键和值，您可以使用`entrySet()`方法。

关于地图的另一个说明：某些地图实现（包括 Java 的标准`HashMap`）允许使用`null`作为键或值，但其他地图则不允许。

# 类型限制

泛型是关于抽象的。泛型允许您创建在不同类型的对象上以相同方式工作的类和方法。术语*generic*源于我们希望能够编写通用算法，这些算法可以广泛地重用于许多类型的对象，而不是必须使我们的代码适应每种情况。这个概念并不新鲜；这正是面向对象编程背后的推动力。Java 泛型并不是向语言添加新功能，而是使可重用的 Java 代码更易于编写和阅读。

泛型将重用推向了一个新的水平，通过使我们处理的对象的 *类型* 成为泛型代码的显式参数。因此，泛型也被称为 *参数化类型*。对于泛型类来说，开发者在使用泛型类型时指定一个类型作为参数（一个参数），代码则根据提供的类型进行自适应。

在其他语言中，泛型有时被称为 *模板*，这更多是一种实现术语。模板就像是中间类，等待其类型参数以便使用。Java 走了一条不同的路线，这既有利也有弊，我们将在本章节详细描述。

Java 泛型有很多值得探讨的地方。一些细节起初可能显得有点晦涩，但不要灰心。你将会大量使用泛型，例如使用现有的类如 `List` 和 `Set`，这些都是简单直观的。设计和创建你自己的泛型需要更谨慎的理解，以及一点耐心和试验。

我们从直觉的角度开始讨论泛型的最引人注目的案例：刚才提到的容器类和集合。接下来，我们退一步，看看 Java 泛型的好坏与丑陋。最后，我们将看几个 Java 中真实世界的泛型类。

## 容器：打造更好的捕鼠器

请记住，在像 Java 这样的面向对象编程语言中，*多态性* 意味着对象总是在某种程度上可互换的。任何类型对象的子类都可以替代其父类型，最终，每个对象都是 `java.lang.Object` 的子类：可以说是面向对象的“夏娃”。

Java 中最一般类型的容器通常与类型 `Object` 一起工作，因此它们可以容纳几乎任何内容。通过 *容器*，我们指的是以某种方式持有其他类实例的类。我们在前一节中看到的 Java 集合框架就是容器的最佳例子。`List`，简而言之，持有一个类型为 `Object` 的有序元素集合。而 `Map` 则持有键值对的关联，其键和值也是最一般的类型 `Object`。通过原始类型的包装器的帮助，这种安排已经为我们服务良好。但是（不要太深奥），“任何类型的集合”也是“没有类型的集合”，而且使用 `Object` 带来了开发者很大的责任。

这有点像对象的化装派对，每个人都戴着同样的面具，消失在集合的人群中。一旦对象穿上`Object`类型的服装，编译器就再也看不到真正的类型并且无法跟踪它们。用户需要稍后使用类型转换来穿透对象的匿名性。就像试图拔掉派对参与者的假胡须一样，您最好确保类型转换是正确的，否则会得到一个不受欢迎的惊喜：

```java
    Date date = new Date();
    List list = new ArrayList();
    list.add(date);
    // other code that might add or remove elements ...
    Date firstElement = (Date)list.get(0); // Is the cast correct? Maybe.
```

`List`接口有一个接受任何类型`Object`的`add()`方法。在这里，我们分配了一个`ArrayList`的实例，它只是`List`接口的一个实现，并添加了一个`Date`对象。这个例子中的转换是否正确？这取决于省略的“其他代码”段内发生了什么。

Java 编译器知道这种类型的活动是危险的，并且在您向简单的`ArrayList`添加元素时发出警告，就像上面的例子一样。我们可以通过一个小小的*jshell*迂回看到这一点。在从`java.util`和`javax.swing`包导入后，尝试创建一个`ArrayList`并添加一些不同的元素：

```java
jshell> import java.util.ArrayList;

jshell> import javax.swing.JLabel;

jshell> ArrayList things = new ArrayList();
things ==> []

jshell> things.add("Hi there");
|  Warning:
|  unchecked call to add(E) as a member of the raw type java.util.ArrayList
|  things.add("Hi there");
|  ^--------------------^
$3 ==> true

jshell> things.add(new JLabel("Hi there"));
|  Warning:
|  unchecked call to add(E) as a member of the raw type java.util.ArrayList
|  things.add(new JLabel("Hi there"));
|  ^--------------------------------^
$5 ==> true

jshell> things
things ==> [Hi there, javax.swing.JLabel[...,text=Hi there,...]]
```

无论您添加的是什么类型的对象，您都可以看到警告是相同的。在最后一步，当我们显示`things`的内容时，普通的`String`对象和`JLabel`对象都在列表中。编译器并不担心使用不同的类型；它友好地警告您，它不知道像上面的`(Date)`转换在运行时是否会起作用。

## 可以修复容器吗？

自然而然地会问，是否有办法改善这种情况。如果我们知道我们只会将`Date`放入我们的列表中，我们不能只创建一个只接受`Date`对象的列表，消除转换，再次让编译器帮助我们吗？也许令人惊讶的答案是，不行。至少，不是以一种令人满意的方式。

我们的第一反应可能是尝试在子类中“重写”`ArrayList`的方法。但当然，重写`add()`方法在子类中实际上并没有覆盖任何东西；它会添加一个新的*重载*方法：

```java
    public void add(Object o) { ... } // still here
    public void add(Date d) { ... }   // overloaded method
```

结果对象仍然接受任何类型的对象——它只是调用不同的方法来实现这一点。

继续前进，我们可能会承担更大的任务。例如，我们可以编写自己的`DateList`类，该类不是扩展`ArrayList`，而是将其方法的实质部分委托给`ArrayList`的实现。通过相当多的单调工作，我们可以得到一个对象，它可以做所有`List`做的事情，但以一种编译器和运行时环境都能理解和强制执行的方式处理`Date`。然而，我们现在给自己挖了个大坑，因为我们的容器不再是`List`的一个实现。这意味着我们不能与所有处理集合的实用程序（如`Collections`.`sort()`）互操作，也不能使用`Collection addAll()`方法将其添加到另一个集合中。

总结一下，问题在于我们并不想细化对象的行为，我们真正想做的是改变它们与用户的契约。我们希望调整它们的方法签名以适应更具体的类型，而多态性无法做到这一点。所以我们是否为我们的集合困于`Object`？这就是泛型的用武之地。

# 进入泛型

如前一节介绍类型限制时所指出的，泛型增强了允许我们为特定类型或一组类型定制类的语法。泛型类在引用类类型时需要一个或多个*类型参数*。它们用于自定义自身。

例如，如果你查看`List`类的源代码或 Javadoc，你会看到它定义了类似这样的内容：

```java
public class List< E > {
  // ...
  public void add(E element) { ... }
  public E get(int i) { ... }
}
```

角括号（`<>`）之间的标识符`E`是*类型参数*。^(1) 它指示`List`类是泛型的，并需要一个 Java 类型作为参数以使其完整。名称`E`是任意的，但随着我们继续，会看到一些惯例。在这种情况下，类型参数`E`代表我们希望存储在列表中的元素类型。`List`类在其体和方法中引用类型参数，就好像它是一个真实的类型，稍后会被替换。类型参数可以用于声明实例变量、方法参数和方法的返回类型。在这种情况下，`E`用作我们将通过`add()`方法添加的元素的类型，以及`get()`方法的返回类型。让我们看看如何使用它。

当我们想使用`List`类型时，同样的角括号语法提供了类型参数：

```java
    List<String> listOfStrings;
```

在这个片段中，我们使用了泛型类型`List`声明了一个名为`listOfStrings`的变量，其类型参数为`String`。`String`指的是`String`类，但我们也可以有一个以任何 Java 类类型为类型参数的专门化`List`。例如：

```java
    List<Date> dates;
    List<java.math.BigDecimal> decimals;
    List<HelloJava> greetings;
```

通过提供其类型参数来完成类型称为*实例化该类型*。有时也称为*调用该类型*，类比于调用方法并提供其参数。与普通的 Java 类型不同，我们简单地通过名称引用类型，像`List<>`这样的泛型类型必须在使用时用参数实例化。^(2) 具体而言，这意味着我们必须在可以出现类型的任何地方实例化类型：作为变量的声明类型（如本代码片段所示），作为方法参数的类型，作为方法的返回类型，或者在使用`new`关键字的对象分配表达式中。

回到我们的`listOfStrings`，现在实际上是一个`List`，其中`String`类型已经替换了类体中的类型变量`E`：

```java
public class List< String > {
  // ...
  public void add(String element) { ... }
  public String get(int i) { ... }
}
```

我们已将`List`类专门化为仅与`String`类型的元素一起使用。此方法签名不再能接受任意的`Object`类型。

`List` 只是一个接口。要使用该变量，我们需要创建一些实际的 `List` 实现的实例。正如我们在介绍中所做的那样，我们将使用 `ArrayList`。与以前一样，`ArrayList` 是实现 `List` 接口的类，但在这种情况下，`List` 和 `ArrayList` 都是泛型类。因此，在使用它们的地方需要类型参数来实例化它们。当然，我们将创建我们的 `ArrayList` 来保存 `String` 元素以匹配我们的 `List` 的 `String`：

```java
    List<String> listOfStrings = new ArrayList<String>();
    // Or shorthand in Java 7.0 and later
    List<String> listOfStrings = new ArrayList<>();
```

如往常一样，`new` 关键字接受一个 Java 类型和可能包含类构造函数参数的括号。在这种情况下，类型是 `ArrayList<String>`——泛型 `ArrayList` 类型实例化为 `String` 类型。

声明变量（如上例中第一行所示）有点麻烦，因为它要求我们在变量类型的左侧和初始化表达式的右侧各提供一次泛型参数类型。在复杂情况下，泛型类型可以变得非常冗长且相互嵌套。

编译器足够智能，可以从您分配给变量的表达式的类型中推断出初始化表达式的类型。这称为*泛型类型推断*，其本质在于您可以通过在变量声明的右侧省略 `<>` 符号的内容来使用简写，如示例的第二个版本所示。

现在我们可以使用我们专门的字符串 `List`。编译器甚至阻止我们尝试将除 `String` 对象（如果有的话还有子类型）之外的任何东西放入列表中。它还允许我们使用 `get()` 方法获取 `String` 对象，而无需进行任何强制转换：

```java
jshell> ArrayList<String> listOfStrings = new ArrayList<>();
listOfStrings ==> []

jshell> listOfStrings.add("Hey!");
$8 ==> true

jshell> listOfStrings.add(new JLabel("Hey there"));
|  Error:
|  incompatible types: javax.swing.JLabel cannot be converted to java.lang.String
|  listOfStrings.add(new JLabel("Hey there"));
|                    ^---------------------^

jshell> String s = strings.get(0);
s ==> "Hey!"
```

让我们从 Collections API 中再举一个例子。`Map` 接口提供了类似字典的映射，将键对象与值对象关联起来。键和值不必是相同类型。泛型 `Map` 接口需要两个类型参数：一个是键的类型，另一个是值的类型。Javadoc 如下所示：

```java
public class Map< K, V > {
  // ...
  public V put(K key, V value) { ... } // returns any old value
  public V get(K key) { ... }
}
```

我们可以创建一个 `Map`，用于按 `Integer` 类型的“员工 ID”号存储 `Employee` 对象，如下所示：

```java
    Map< Integer, Employee > employees = new HashMap<Integer, Employee>();
    Integer bobsId = 314; // hooray for autoboxing!
    Employee bob = new Employee("Bob", ...);

    employees.put(bobsId, bob);
    Employee employee = employees.get(bobsId);
```

在这里，我们使用了 `HashMap`，它是实现 `Map` 接口的泛型类。我们用类型参数 `Integer` 和 `Employee` 实例化了两种类型。现在，`Map` 只能使用类型为 `Integer` 的键，并保存类型为 `Employee` 的值。

我们在这里使用 `Integer` 来保存我们的数字的原因是，泛型类的类型参数必须是类类型。我们不能使用原始类型（例如 `int` 或 `boolean`）参数化泛型类。幸运的是，在 Java 中，原始类型的自动装箱（参见“原始类型的包装类”）几乎使其看起来像是我们可以通过允许我们像使用包装类型一样使用原始类型。

超过 Collections 的许多其他 API 都使用泛型来使您能够将它们适应特定类型。我们将在本书的各个部分讨论它们。

## 谈论类型

在我们转向更重要的事情之前，我们应该对我们如何描述泛型类的特定参数化方式说几句话。因为最常见和最引人注目的泛型案例是用于类似容器的对象，所以通常会以泛型类型“持有”参数类型的方式来思考。在我们的示例中，我们称我们的 `List<String>` 为“字符串列表”，因为确实是这样的。类似地，我们可能会称我们的员工映射为“员工 ID 到员工对象的映射”。然而，这些描述更专注于类的*行为*而不是类型本身。

取而代之的是，考虑一个名为 `Trap<E>` 的单个对象容器，可以实例化为 `Mouse` 类型或 `Bear` 类型的对象；也就是说，`Trap<Mouse>` 或 `Trap<Bear>`。我们本能地称新类型为“捕鼠器”或“熊夹”。我们也可以将我们的字符串列表看作是一个新类型。我们可以讨论“字符串列表”，或将我们的员工映射描述为新的“整数员工对象映射”类型。您可以使用您喜欢的任何措辞，但后一种描述更专注于将泛型视为*类型*的概念，并且在讨论泛型类型在类型系统中如何相关时，可能会帮助您保持术语的清晰性。在那里，我们将看到容器术语实际上有点反直觉。

在接下来的部分中，我们将从不同的角度讨论 Java 中的泛型类型。我们已经看到它们能做些什么；现在我们需要讨论它们如何做到这一点。

# “没有勺子”

在电影 *The Matrix* 中，主人公尼奥被提出了一个选择：服用蓝色药丸并留在幻想世界中，或者服用红色药丸并看到事物的真实面目。在处理 Java 中的泛型时，我们面临着类似的本体论困境。在讨论泛型时，我们只能走得那么远，然后不得不面对它们如何实现的现实。我们的幻想世界是编译器为了让我们编写代码更容易接受而创造的一个地方。我们的现实（虽然不像电影中的反乌托邦噩梦那样严峻）是一个更加艰难的地方，充满了看不见的危险和问题。为什么强制类型转换和测试在泛型中不能正常工作？为什么我不能在一个类中实现看似两个不同的泛型接口？为什么我可以声明一个泛型类型的数组，即使在 Java 中无法创建这样的数组？！

我们将在本章的其余部分回答这些问题，您甚至无需等待续集。您将很快能够弯曲勺子（好吧，类型）。让我们开始吧。

## 擦除

Java 通用类型的设计目标是雄心勃勃的：在语言中添加一个全新的语法，安全地引入参数化类型，并且不影响性能，并且，哦，顺便兼容所有现有的 Java 代码，并且不以任何严重的方式改变编译后的类。令人惊讶的是，他们实际上满足了这些条件，也不奇怪这需要一些时间。但是一如既往，一些必要的妥协导致了一些头痛。

为了实现这一功能，Java 采用了一种称为*擦除*的技术。擦除与这样一个想法有关：由于我们与通用类型的大多数操作都是在编译时静态应用的，通用信息不需要在编译后的类中保留。编译器强制执行的类的通用特性可以在二进制类中被“擦除”，以保持与非通用代码的兼容性。

虽然 Java 在编译形式中保留了关于类的通用特性的信息，但这些信息主要由编译器使用。Java 运行时根本不知道通用类型（generics），也不会浪费任何资源在其上。

我们可以使用*jshell*来确认参数化的`List<E>`在运行时仍然是一个`List`：

```java
jshell> import java.util.*;

jshell> List<Date> dateList = new ArrayList<Date>();
dateList ==> []

jshell> dateList instanceof List
$3 ==> true
```

但是我们的通用`dateList`显然没有实现刚刚讨论的`List`方法：

```java
jshell> dateList.add(new Object())
|  Error:
|  incompatible types: java.lang.Object cannot be converted to java.util.Date
|  dateList.add(new Object())
|               ^----------^
```

这说明了 Java 通用类型的有些古怪的性质。编译器相信它们，但运行时却说它们是幻觉。如果我们尝试一些更简单的事情并检查我们的`dateList`是否是一个`List<Date>`：

```java
jshell> dateList instanceof List<Date>;
|  Error:
|  illegal generic type for instanceof
|  dateList instanceof List<Date>;
|                      ^--------^
```

这次编译器直截了当地说：“不行。” 你不能在`instanceof`操作中测试一个通用类型。由于在运行时没有可辨别不同参数化的`List`的类（每个`List`仍然是一个`List`），`instanceof`运算符无法区分一个`List`的不同实例。所有的通用安全检查都是在编译时完成的，因此在运行时我们只是处理一个单一的实际`List`类型。

事实上是这样的：编译器抹去了所有的尖括号语法，并在我们的`List`类中用一个在运行时可以与任何允许的类型一起工作的类型替换了类型参数：在这种情况下，是`Object`。我们似乎回到了起点，只是编译器仍然具有在编译时强制我们使用通用类型的知识，并且因此可以为我们处理类型转换。如果你反编译一个使用了`List<Date>`（使用*javap*命令和*-c*选项显示字节码，如果你敢的话），你会看到编译后的代码实际上包含了到`Date`的转换，尽管我们自己并没有写。

现在我们可以回答本节开始时提出的一个问题：“为什么我不能在一个类中实现看起来是两个不同的泛型接口？”我们不能有一个类同时实现两个不同的泛型`List`实例化，因为它们在运行时实际上是相同类型，没有办法区分它们：

```java
public abstract class DualList implements List<String>, List<Date> { }
// Error: java.util.List cannot be inherited with different arguments:
//    <java.lang.String> and <java.util.Date>
```

幸运的是，总有办法解决。例如，在这种情况下，您可以使用一个共同的超类或创建多个类。虽然这些替代方法可能不那么优雅，但您几乎总能找到一个干净的答案，即使有点冗长。

## 原始类型

尽管编译器在编译时将泛型类型的不同参数化视为不同的类型（具有不同的 API），但我们已经看到在运行时只存在一个真正的类型。例如，`List<Date>`和`List<String>`共享旧式的 Java 类`List`。`List`被称为泛型类的*原始类型*。每个泛型都有一个原始类型。它是“普通”的 Java 形式，所有泛型类型信息已被移除，类型变量被一般的 Java 类型如`Object`替换。

在 Java 中可以使用原始类型。然而，Java 编译器在以“不安全”方式使用它们时生成警告。在*jshell*之外，编译器仍然会注意到这些问题：

```java
    // nongeneric Java code using the raw type
    List list = new ArrayList(); // assignment ok
    list.add("foo"); // Compiler warning on usage of raw type
```

此代码片段像 Java 5 之前的老式 Java 代码一样使用了原始的`List`类型。不同之处在于现在 Java 编译器在我们尝试向列表中插入对象时会发出*未经检查的警告*：

```java
% javac RawType.java
Note: RawType.java uses unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.
```

编译器指导我们使用`-Xlint:unchecked`选项，以获取有关不安全操作位置的更详细信息：

```java
% javac -Xlint:unchecked MyClass.java
RawType.java:6: warning: [unchecked] unchecked call to add(E)
as a member of the raw type List
    list.add("foo");
            ^
  where E is a type-variable:
    E extends Object declared in interface List
```

请注意，创建和分配原始的`ArrayList`并不会生成警告。只有当我们尝试使用“不安全”的方法（引用类型变量的方法）时才会收到警告。这意味着仍然可以使用与原始类型相关的旧式非泛型 Java API。只有在我们自己的代码中做一些不安全的操作时才会收到警告。

在我们继续之前，还有关于擦除的一件事。在前面的示例中，类型变量被`Object`类型替换，它可以表示适用于类型变量`E`的任何类型。后面我们会看到，并非总是这样。我们可以对参数类型设置限制或*边界*，当我们这样做时，编译器对类型的擦除可以更加严格，例如：

```java
class Bounded< E extends Date > {
  public void addElement(E element) { ... }
}
```

此参数类型声明表示元素类型`E`必须是`Date`类型的子类型。在这种情况下，`addElement()`方法的擦除因此比`Object`更加严格，编译器使用`Date`：

```java
  public void addElement(Date element) { ... }
```

`Date`被称为此类型的*上界*，这意味着它是对象层次结构的顶部。您只能在`Date`或“较低”（更派生或子类化）类型上实例化参数化类型。

现在我们对泛型类型的真实含义有了一些了解，我们可以更详细地探讨它们的行为。

# 参数化类型关系

我们现在知道参数化类型共享一个普通的原始类型。这就是为什么我们的参数化 `List<Date>` 在运行时只是一个 `List`。事实上，如果需要，我们可以将 `List` 的任何实例分配给原始类型：

```java
    List list = new ArrayList<Date>();
```

我们甚至可以反过来，将原始类型分配给泛型类型的特定实例：

```java
    List<Date> dates = new ArrayList(); // unchecked warning
```

此语句在分配时生成了未检查的警告，但之后，编译器相信该列表在分配之前只包含 `Date`。您可以尝试将 `new ArrayList()` 强制转换为 `List<Date>`，但这不会解决警告。我们将在“类型转换”中讨论向泛型类型的转换。

无论运行时类型如何，编译器都掌控着一切。它不允许我们分配明显不兼容的事物：

```java
    List<Date> dates = new ArrayList<String>(); // Compile-time Error!
```

当然，`ArrayList<String>` 没有实现编译器需要的 `List<Date>` 方法，所以这些类型是不兼容的。

但更有趣的类型关系呢？例如，`List` 接口是更一般的 `Collection` 接口的子类型。你能将泛型 `List` 的特定实例分配给某个泛型 `Collection` 实例吗？这是否取决于类型参数及其关系？显然，`List<Date>` 不是 `Collection<String>`。但 `List<Date>` 是否是 `Collection<Date>`？`List<Date>` 可以是 `Collection<Object>` 吗？

首先，我们先快速说出答案，然后详细讲解。到目前为止，我们讨论的简单泛型实例的规则是*继承仅适用于“基本”泛型类型，而不适用于参数类型*。此外，仅当两个泛型类型在*完全相同的参数类型*上实例化时，可赋值性才适用。换句话说，仍然存在一维继承，遵循基本泛型类类型，但有一个附加限制，即参数类型必须完全相同。

例如，由于 `List` 是 `Collection` 的一种类型，当类型参数完全相同时，我们可以将 `List` 的实例分配给 `Collection` 的实例：

```java
    Collection<Date> cd;
    List<Date> ld = new ArrayList<Date>();
    cd = ld; // Ok!
```

这段代码片段表明 `List<Date>` 是 `Collection<Date>` —— 非常直观。但在参数类型变化的情况下尝试相同的逻辑则失败：

```java
    List<Object> lo;
    List<Date> ld = new ArrayList<Date>();
    lo = ld; // Compile-time Error!  Incompatible types.
```

虽然我们的直觉告诉我们，`List` 中的 `Date` 可以作为 `Object` 幸福地存活，但分配是一个错误。我们将在下一节中详细解释原因，但现在只需注意类型参数并非完全相同，并且在泛型中，参数类型之间没有继承关系。

这是一个有助于以类型而不是以实例化对象所做的事情的角度来思考的情况。这些实际上并不是“日期列表”和“对象列表”——更像是一个`DateList`和一个`ObjectList`，它们之间的关系并不显而易见。

试着挑出下面示例中哪些是可以的，哪些是不可以的：

```java
    Collection<Number> cn;
    List<Integer> li = new ArrayList<Integer>();
    cn = li;
```

一个`List`的实例化可以是一个`Collection`的实例化，但前提是参数类型完全相同。继承不遵循参数类型，所以这个示例中的最后一个赋值失败了。

之前我们提到过，这个规则适用于本章我们讨论过的实例化的简单类型。还有哪些类型呢？嗯，到目前为止我们所见过的实例化类型，我们将一个实际的 Java 类型作为参数插入的那种类型被称为**具体类型实例化**。稍后，我们将讨论*通配符实例化*，它们类似于类型的数学集合操作（比如并集和交集）。还可能有更多的泛型实例化，其中类型关系实际上是二维的，取决于基类型和参数化。但不用担心：这种情况并不经常发生，也没有听起来那么可怕。

## 为什么`List<Date>`不是`List<Object>`？

这是一个合理的问题。为什么我们不能将我们的`List<Date>`分配给`List<Object>`并将`Date`元素作为`Object`类型来使用？

原因在于泛型的理论基础：改变编程契约。在最简单的情况下，假设一个`DateList`类型扩展了一个`ObjectList`类型，那么`DateList`将拥有所有`ObjectList`的方法，我们可以向其中插入`Object`。现在，你可能会反对说泛型让我们改变了方法签名，所以这不再适用。这是正确的，但有一个更大的问题。如果我们能够将`DateList`分配给`ObjectList`变量，我们就可以使用`Object`方法将类型不是`Date`的元素插入其中。

我们可以将`DateList`（提供替代性、更广泛的类型）**别名**为`ObjectList`。使用别名对象，我们可以试图欺骗它接受其他类型：

```java
    DateList dateList = new DateList();
    ObjectList objectList = dateList; // Can't really do this
    objectList.add(new Foo()); // should be runtime error!
```

当实际的`DateList`实现被呈现错误类型的对象时，我们预期会得到一个运行时错误。

这就是问题所在。Java 泛型没有运行时表示。即使这个功能很有用，Java 也无法在运行时知道该做什么。这个特性非常危险——它允许在运行时发生无法在编译时捕获的错误。通常，我们希望在编译时捕获类型错误。

如果您认为 Java 可以通过禁止这些赋值来在编译时不生成未检查的警告来保证代码的类型安全性。不幸的是它不能，但这种限制与泛型无关；它与数组有关。（如果这些对你听起来很熟悉，那是因为我们在第四章中提到了这个问题，与 Java 数组有关。）数组类型具有一种继承关系，允许这种别名发生：

```java
    Date [] dates = new Date[10];
    Object [] objects = dates;
    objects[0] = "not a date"; // Runtime ArrayStoreException!
```

数组在运行时具有不同的类表示。它们在运行时进行自检，在这种情况下会抛出`ArrayStoreException`。如果你以这种方式使用数组，Java 编译器无法保证你的代码的类型安全性。

# 强制类型转换

现在我们已经谈论了泛型类型之间甚至泛型类型与原始类型之间的关系。但我们还没有真正探讨在泛型世界中的强制类型转换的概念。

当我们用泛型与它们的原始类型交换时，是不需要强制类型转换的。但我们会触发编译器的未检查警告：

```java
    List list = new ArrayList<Date>();
    List<Date> dl = list;  // unchecked warning
```

通常情况下，我们在 Java 中使用强制类型转换来处理可能可赋值的两种类型。例如，我们可以尝试将一个`Object`转换为`Date`，因为`Object`可能是一个`Date`值。然后强制类型转换会在运行时进行检查，以查看我们是否正确。

在不相关的类型之间进行强制类型转换是一个编译时错误。例如，我们甚至无法尝试将一个`Integer`转换为`String`。这些类型之间没有继承关系。那么在兼容的泛型类型之间进行转换呢？

```java
    Collection<Date> cd = new ArrayList<Date>();
    List<Date> ld = (List<Date>)cd; // Ok!
```

这段代码片段展示了从更一般的`Collection<Date>`到`List<Date>`的有效转换。这里的转换是合理的，因为一个`Collection<Date>`可以赋值并且实际上可能是一个`List<Date>`。

类似地，下面的强制类型转换捕获了我们的错误：我们将`TreeSet<Date>`别名为`Collection<Date>`，然后尝试将其转换为`List<Date>`：

```java
    Collection<Date> cd = new TreeSet<Date>();
    List<Date> ld = (List<Date>)cd; // Runtime ClassCastException!
    ld.add(new Date());
```

但是有一种情况下，泛型的强制类型转换是无效的，那就是在尝试根据它们的参数类型来区分类型时：

```java
    Object o = new ArrayList<String>();
    List<Date> ld = (List<Date>)o; // unchecked warning, ineffective
    Date d = ld.get(0); // unsafe at runtime, implicit cast may fail
```

在这里，我们将一个`ArrayList<String>`别名为一个普通的`Object`。接下来，我们将`o`强制类型转换为`List<Date>`。不幸的是，Java 在运行时无法区分`List<String>`和`List<Date>`之间的区别，所以这种强制类型转换是无效的。编译器通过在强制类型转换位置生成未检查的警告来提醒我们。当我们尝试使用强制类型转换的对象`ld`时，我们可能会发现它是不正确的。由于擦除和缺乏类型信息，泛型类型上的强制类型转换在运行时是无效的。

## 在集合和数组之间进行转换

尽管它们没有直接的继承关系或共享接口，但在集合和数组之间进行转换仍然很简单。为了方便起见，您可以使用以下方法将集合的元素作为数组检索出来：

```java
    public Object[] toArray()
    public <E> E[] toArray(E[] a)
```

第一个方法返回一个普通的 `Object` 数组。通过第二种形式，我们可以更具体地返回正确元素类型的数组。如果我们提供了足够大小的数组，它将用值填充。但如果数组长度太短（例如，长度为零），Java 将创建一个所需长度的相同类型的新数组，并返回它。因此，您可以像这样传递一个空数组来获取正确类型的数组：

```java
    Collection<String> myCollection = ...;
    String [] myStrings = myCollection.toArray(new String[0]);
```

这个技巧有点笨拙。如果 Java 允许我们使用 `Class` 引用显式指定类型，会更好，但出于某种原因，它并没有这样做。

另一种方法是，您可以使用 `java.util.Arrays` 辅助类的静态 `asList()` 方法将对象数组转换为 `List` 集合：

```java
    String [] myStrings = { "a", "b", "c" };
    List list = Arrays.asList(myStrings);
```

编译器还足够智能，能够识别对 `List<String>` 变量的有效赋值。

## 迭代器

*迭代器* 是一种允许您逐步浏览一系列值的对象。这种操作非常常见，因此有一个标准接口：`java.util.Iterator`。`Iterator` 接口有三个有趣的方法：

`public E next()`

此方法返回关联集合的下一个元素（泛型类型 E 的元素）。

`public boolean hasNext()`

如果尚未遍历完 `Collection` 的所有元素，则此方法返回 `true`。换句话说，如果可以调用 `next()` 获取下一个元素，则返回 `true`。

`public void remove()`

此方法从关联的 `Collection` 中移除从 `next()` 返回的最近对象。

以下示例显示了如何使用 `Iterator` 打印集合的每个元素：

```java
  public void printElements(Collection c, PrintStream out) {
    Iterator iterator = c.iterator();
    while (iterator.hasNext()) {
      out.println(iterator.next());
    }
  }
```

使用 `next()` 获取下一个元素后，有时可以使用 `remove()` 将其移除。例如，通过待办事项清单的方式进行处理时，可能会遵循以下模式：“获取一个项目，处理该项目，移除该项目”。但是，迭代器的移除功能并不总是合适，也不是所有迭代器都实现了 `remove()`。例如，无法从只读集合中移除元素是没有意义的。

如果不允许删除元素，则从此方法中抛出 `UnsupportedOperationException`。如果在首次调用 `next()` 之前调用 `remove()`，或者连续两次调用 `remove()`，则会抛出 `IllegalStateException`。

### 遍历集合

在 “for 循环” 中描述的一种形式的 `for` 循环可以操作所有 `Iterable` 类型，这意味着它可以迭代所有 `Collection` 对象类型，因为该接口扩展了 `Iterable`。例如，它现在可以遍历类型化的 `Date` 对象集合的所有元素，如下所示：

```java
    Collection<Date> col = ...
    for (Date date : col) {
      System.out.println(date);
    }
```

Java 内置 `for` 循环的这个特性称为“增强” `for` 循环（与预泛型、仅数字 `for` 循环相对）。增强 `for` 循环仅适用于 `Collection` 类型的集合，而不适用于 `Map`。但在某些情况下，遍历映射可能很有用。您可以使用 `Map` 方法 `keySet()` 或 `values()`（甚至 `entrySet()` 如果您希望每个键/值对作为单个实体）从您的映射中获取一个可以使用这个增强 `for` 循环的集合：

```java
    Map<Integer, Employee> employees = new HashMap<>();
    // ...
    for (Integer id : employees.keySet()) {
      System.out.print("Employee " + id);
      System.out.println(" => " + employees.get(id));
    }
```

键的集合是一个简单的无序集合。上面的增强 `for` 循环将显示所有您的员工，但它们的打印顺序可能看起来有些随机。如果您希望按其 ID 或者也许是它们的名称列出它们，您需要首先对键或值进行排序。幸运的是，排序是一个非常常见的任务——集合框架可以帮助。

# 详细解析：`sort()` 方法

在 `java.util.Collections` 类中查找，我们找到了各种用于处理集合的静态实用方法。其中之一是这个好东西——静态泛型方法 `sort()`：

```java
<T extends Comparable<? super T>> void sort(List<T> list) { ... }
```

另一个我们要解决的难题。让我们专注于边界的最后部分：

```java
Comparable<? super T>
```

这是我们在 “参数化类型关系” 中提到的通配符实例化。在这种情况下，它是一个接口，因此我们可以将 `sort()` 方法返回类型中的 `extends` 理解为 `implements`。

`Comparable` 包含一个 `compareTo()` 方法，用于某个参数类型。`Comparable<String>` 意味着 `compareTo()` 方法接受类型 `String`。因此，`Comparable<? super T>` 是在 `T` 及其所有超类上的 `Comparable` 实例化的集合。`Comparable<T>` 足够，并且在另一端，`Comparable<Object>` 也是如此。

这在英语中意味着元素必须与它们自己的类型可比较，或者与它们自己的类型的某个超类型可比较，以便 `sort()` 方法可以使用它们。这确保了所有元素可以相互比较，但并不像说它们都必须自己实现 `compareTo()` 方法那样具有限制性。一些元素可以从一个知道如何仅与 `T` 的某个超类型比较的父类继承 `Comparable` 接口，这正是允许的。

# 应用：田野上的树木

本章中有很多理论。不要害怕理论——它可以帮助您预测新场景中的行为，并激发解决新问题的解决方案。但实践同样重要，所以让我们回顾一下我们在 “类” 中开始的游戏。特别是，现在是存储每种类型多个对象的时候了。

在 第十三章 中，我们将介绍网络和创建需要存储多个物理学家的两人游戏设置。现在，我们仍然只有一个物理学家可以一次扔一个苹果。但我们可以在我们的场地上种植几棵树作为靶子练习。

让我们添加六棵树。我们将使用一对循环，这样您可以轻松增加树木数量（如果愿意）。我们的`Field`当前仅存储一个树实例。我们可以将该存储升级为一个类型化列表（我们称之为`trees`）。从那里，我们可以以多种方式添加和移除树木：

+   我们可以为`Field`创建一些处理列表的方法，也许还可以实施其他一些游戏规则（例如管理最大数量的树木）。

+   我们可以直接使用列表，因为`List`类已经有了大多数我们想做的事情的好方法。

+   我们可以结合这些方法的一些组合：适合我们的游戏的特殊方法，以及其他所有地方直接操作。

由于我们确实有一些特定于`Field`的游戏规则，因此我们将在此处采取第一种方法。（但是请查看示例，并考虑如何修改以直接使用树木列表。）我们将从一个`addTree()`方法开始。采用这种方法的一个好处是，我们还可以将树实例的创建重定位到我们的方法中，而不是单独创建和操作树。以下是在场地上添加树木的一种方法：

```java
  List<Tree> trees = new ArrayList<>();
  // other field state

  public void addTree(int x, int y) {
    Tree tree = new Tree();
    tree.setPosition(x,y);
    trees.add(tree);
  }
```

有了这种方法，我们可以很快地添加一些树木：

```java
    Field field = new Field();
    // other setup code
    field.addTree(100,100);
    field.addTree(200,100);
```

这两行代码并排添加了一对树木。让我们继续编写我们需要创建六棵树的循环：

```java
    Field field = new Field();
    // other setup code
    for (int row = 1; row <= 2; row++) {
      for (int col = 1; col <=3; col++) {
        field.addTree(col * 100, row * 100);
      }
    }
```

现在你能看到，如果要添加八、九或一百棵树是多么容易了吗？计算机在重复方面做得很好。

为了创建我们的苹果目标森林万岁！尽管我们遗漏了一些关键细节。最重要的是，我们需要在屏幕上显示我们的新森林。我们还需要更新`Field`类的绘图方法，以便正确理解和使用我们的树木列表。随着我们为游戏添加更多功能，我们也将对物理学家和苹果执行相同的操作。此外，我们还需要一种方法来移除不再活动的元素。但首先，让我们看看我们的森林！

```java
// File: Field.java
  protected void paintComponent(Graphics g) {
    g.setColor(fieldColor);
    g.fillRect(0,0, getWidth(), getHeight());
    for (Tree t : trees) {
      t.draw(g);
    }
    physicist.draw(g);
    apple.draw(g);
  }
```

由于我们已经在存储我们的树木的`Field`类中，没有必要编写一个单独的函数来提取单个树并对其进行绘制。我们可以使用巧妙的增强型`for`循环结构，快速将所有树木放在场地上，如图 7-1 所示。

![ljv6 0701](img/ljv6_0701.png)

###### 图 7-1。在我们的`List`中渲染所有树木

# 有用的特性

Java 集合和泛型是语言中非常强大和有用的附加功能。尽管本章后半部分深入探讨的一些细节可能看起来令人生畏，但通常使用却非常简单和引人注目：泛型使集合更好。随着您对泛型的使用增加，您会发现您的代码变得更加可读和可维护。集合允许优雅、高效的存储。泛型使您之前根据使用推断出来的内容显式化。

## 复习问题

1.  如果你想存储一个包含姓名和电话号码的联系人列表，哪种类型的集合最适合？

1.  你使用什么方法来获取`Set`中项的迭代器？

1.  如何将`List`转换为数组？

1.  如何将数组转换为`List`？

1.  你应该实现哪个接口以使用`Collections.sort()`方法对列表进行排序？

## 代码练习

1.  *ch07exercises*文件夹中的`EmployeeList`类包含了一些员工，加载到一个员工 ID 和`Employee`对象的映射中，与前面几个示例中使用的类似。我们提到要按 ID 号对这些员工进行排序，但没有展示任何代码。尝试按照他们的 ID 号排序员工。你可能需要使用`keySet()`方法，然后从该集合创建一个临时但可排序的列表。

1.  在高级练习 5.1 中，你创建了一个新的障碍类`Hedge`。更新游戏，使得你可以拥有多个树篱，类似于多个树木。确保当你运行程序时所有的树木和树篱都能正确绘制。

## 高级练习

1.  在上述代码练习 1 中，你可能对映射的键进行了排序，然后使用正确排序的键来获取相应的员工。为了更具挑战性，尝试在你的`Employee`类中实现`Comparable`接口。你可以决定如何组织员工：按 ID、姓氏、全名或者可能是这些属性的某种组合。而不是对`keySet()`集合进行排序，尝试直接通过从你的映射的`values()`中构建临时列表来对你新建的可比较员工进行排序。

^(1) 你可能还会看到术语*类型变量*。Java 语言规范大多使用*参数*，所以我们尽量坚持这个术语，但你可能会看到两个名称都在使用。

^(2) 也就是说，除非你想以非泛型方式使用泛型类型。我们稍后会讨论“原始”类型。

^(3) 如果你们中有些人想要了解本节标题的背景，这里是它的来源。我们的英雄 Neo 正在学习他的超能力。

男孩：不要试图弯曲勺子。那是不可能的。相反，只需试图意识到真相。

Neo：什么真相？

男孩：没有勺子。

Neo：没有勺子？

男孩：那么你会看到不是勺子弯曲，只有你自己在弯曲。

—瓦卓斯基姐弟。*黑客帝国*。136 分钟。华纳兄弟，1999 年。
