## 附录 B. 其他库更新

本附录回顾了 Java 8 库的主要新增内容。

### B.1\. 集合

集合 API 的最大更新是引入了流，我们已在第四章、第五章和第六章中讨论过。第九章中也讨论了其他更新，本附录中还总结了额外的少量新增内容。

#### B.1.1\. 其他方法

Java API 设计者充分利用了默认方法，并为集合接口和类添加了几个新方法。新方法列在表 B.1 中。

##### 表 B.1\. 集合类和接口新增的方法

| 类/接口 | 新方法 |
| --- | --- |
| Map | getOrDefault, forEach, compute, computeIfAbsent, computeIfPresent, merge, putIfAbsent, remove(key, value), replace, replaceAll, of, ofEntries |
| Iterable | forEach, spliterator |
| Iterator | forEachRemaining |
| 集合 | removeIf, stream, parallelStream |
| List | replaceAll, sort, of |
| BitSet | stream |
| Set | of |

##### 地图

`Map`接口是最新的接口，支持几个新的便捷方法。例如，可以使用`getOrDefault`方法来替换之前检查`Map`是否包含给定键映射的现有惯用语。如果没有，您可以提供一个默认值以返回。之前您会这样做：

```
Map<String, Integer> carInventory = new HashMap<>();
Integer count = 0;
if(map.containsKey("Aston Martin")){
  count = map.get("Aston Martin");
}
```

您现在可以更简单地执行以下操作：

```
Integer count = map.getOrDefault("Aston Martin", 0);
```

注意，这仅在没有任何映射的情况下才有效。例如，如果键显式映射到值`null`，则不会返回任何默认值。

另一个特别有用的方法是`computeIfAbsent`，我们在第十九章（[kindle_split_034.xhtml#ch19]）中简要提到，当时解释了记忆化。它允许您方便地使用缓存模式。假设您需要从不同的网站获取和处理数据。在这种情况下，缓存数据很有用，这样您就不必多次执行（昂贵的）获取操作：

```
public String getData(String url){
    String data = cache.get(url);
    if(data == null){                 *1*
        data = getData(url);
        cache.put(url, data);         *2*
    }
    return data;
}
```

+   ***1* 检查数据是否已缓存。**

+   ***2* 如果不是，请获取数据，然后将其缓存到 Map 中以供将来使用。**

您现在可以通过使用`computeIfAbsent`来更简洁地编写以下代码：

```
public String getData(String url){
    return cache.computeIfAbsent(url, this::getData);
}
```

所有其他方法的描述可以在官方 Java API 文档中找到（[`docs.oracle.com/javase/8/docs/api/java/util/Map.html`](http://docs.oracle.com/javase/8/docs/api/java/util/Map.html)）。请注意，`ConcurrentHashMap`也更新了额外的功能。我们将在第 B.2 节中讨论它们。

##### 集合

可以使用`removeIf`方法来删除集合中所有匹配谓词的元素。请注意，这与`Streams` API 中包含的`filter`方法不同。`Streams` API 中的`filter`方法会产生一个新的流；它不会修改当前的流或源。

##### 列表

`replaceAll` 方法将 `List` 中的每个元素替换为应用给定运算符后的结果。它与流中的 `map` 方法类似，但它会修改 `List` 的元素。相比之下，`map` 方法会产生新的元素。

例如，以下代码将打印 `[2, 4, 6, 8, 10]`，因为 `List` 在原地被修改：

```
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.replaceAll(x -> x * 2);
System.out.println(numbers);          *1*
```

+   ***1* 打印 [2, 4, 6, 8, 10]**

#### B.1.2\. 集合类

`Collections` 类已经存在很长时间，用于操作或返回集合。现在它包括额外的返回不可修改、同步、检查和空的 `NavigableMap` 和 `NavigableSet` 的方法。此外，它还包括 `checkedQueue` 方法，该方法返回一个扩展了动态类型检查的 `Queue` 视图。

#### B.1.3\. 比较器

`Comparator` 接口现在包括默认和静态方法。你曾在第三章中使用 `Comparator.comparing` 静态方法来返回一个 `Comparator` 对象，该对象由提取排序键的函数给出。

新增实例方法包括以下内容：

+   `reversed`——返回一个具有当前 `Comparator` 反向排序的 `Comparator`。

+   `thenComparing`——返回一个在两个对象相等时使用另一个 `Comparator` 的 `Comparator`。

+   `thenComparingInt`、`thenComparingDouble`、`thenComparingLong`——与 `thenComparing` 方法类似，但接受针对原始类型的专用函数（分别是 `ToIntFunction`、`ToDoubleFunction` 和 `ToLongFunction`）。

新增静态方法包括以下内容：

+   `comparingInt`、`comparingDouble`、`comparingLong`——与 `comparing` 方法类似，但接受针对原始类型的专用函数（分别是 `ToIntFunction`、`ToDoubleFunction` 和 `ToLongFunction`）。

+   `naturalOrder`——返回一个对 `Comparable` 对象施加自然排序的 `Comparator` 对象。

+   `nullsFirst`、`nullsLast`——返回一个将 `null` 视为小于非 `null` 或大于非 `null` 的 `Comparator` 对象。

+   `reverseOrder—` 等同于 `naturalOrder().reverse()`。

### B.2\. 并发

Java 8 带来了与并发相关的几个更新。首先是当然的，并行流的引入，我们将在第七章中探讨。还有 `CompletableFuture` 类的引入，你可以在第十六章中了解它。

有其他值得注意的更新。例如，`Arrays` 类现在支持并行操作。我们将在章节 B.3 中讨论这些操作。

在本节中，我们查看 `java.util.concurrent.atomic` 包的更新，该包处理原子变量。此外，我们讨论 `ConcurrentHashMap` 类的更新，它支持几个新方法。

#### B.2.1\. 原子操作

`java.util.concurrent.atomic` 包提供了几个数字类，例如 `AtomicInteger` 和 `AtomicLong`，它们支持对单个变量的原子操作。它们已更新以支持新的方法：

+   `getAndUpdate`—原子地使用给定的函数更新当前值，返回更新前的值。

+   `updateAndGet`—原子地使用给定的函数更新当前值，返回更新后的值。

+   `getAndAccumulate`—原子地使用给定的函数更新当前值，该函数应用于当前值和给定值，返回更新前的值。

+   `accumulateAndGet`—原子地使用给定的函数更新当前值，该函数应用于当前值和给定值，返回更新后的值。

这里是如何原子性地设置一个观察到的值为 10 和现有原子整数之间的最小值：

```
int min = atomicInteger.accumulateAndGet(10, Integer::min);
```

##### 加法器和累加器

Java API 建议在多个线程频繁更新但读取较少的情况下（例如，在统计学的上下文中）使用新的类`LongAdder`、`LongAccumulator`、`DoubleAdder`和`DoubleAccumulator`，而不是使用等效的`Atomic`类。这些类设计为动态增长以减少线程竞争。

类`LongAdder`和`DoubleAdder`支持加法操作，而`LongAccumulator`和`DoubleAccumulator`提供了一个组合值的函数。例如，要计算几个值的总和，可以使用以下方式`LongAdder`。

##### 列表 B.1\. 使用`LongAdder`计算值的总和

```
LongAdder adder = new LongAdder();       *1*
adder.add(10);                           *2*
// ...
long sum = adder.sum();                  *3*
```

+   ***1* 使用默认构造函数将初始总和值设置为 0。**

+   ***2* 在几个不同的线程中进行一些加法操作。**

+   ***3* 在某个点上获取总和。**

或者你可以按照以下方式使用`LongAccumulator`。

##### 列表 B.2\. 使用`LongAccumulator`计算值的总和

```
LongAccumulator acc = new LongAccumulator(Long::sum, 0);
acc.accumulate(10);                                       *1*
// ...
long result = acc.get();                                  *2*
```

+   ***1* 在几个不同的线程中累积值。**

+   ***2* 在某个点上获取结果。**

#### B.2.2\. ConcurrentHashMap

`ConcurrentHashMap`类被引入以提供更现代的`HashMap`，它是线程安全的。`ConcurrentHashMap`允许并发添加和更新，但只锁定内部数据结构的一部分。因此，与同步的`Hashtable`替代品相比，读写操作的性能得到了提高。

##### 性能

`ConcurrentHashMap`的内部结构已更新以提高性能。映射的条目通常存储在通过键的生成哈希码访问的桶中。但如果许多键返回相同的哈希码，性能将下降，因为桶实现为具有 O(*n*)检索的`List`。在 Java 8 中，当桶变得太大时，它们会动态地替换为具有 O(log(*n*))检索的有序树。请注意，这仅在键是`Comparable`（例如，`String`或`Number`类）时才可能。

##### 类似于流的操作

`ConcurrentHashMap`支持三种新的操作，这些操作让人联想到你在流中看到的内容：

+   `forEach`—对每个(key, value)执行给定的操作

+   `reduce`—使用一个归约函数将所有给定的(key, value)组合成一个结果

+   `search`—对每个（键，值）应用一个函数，直到函数产生一个非 `null` 结果

每种操作支持四种形式，接受具有键、值、`Map.Entry` 和（键，值）参数的函数：

+   与键和值操作（`forEach`、`reduce`、`search`）

+   与键操作（`forEachKey`、`reduceKey`、`searchKey`）

+   与值操作（`forEachValue`、`reduceValue`、`searchValues`）

+   与 `Map.Entry` 对象操作（`forEachEntry`、`reduceEntries`、`searchEntries`）

注意，这些操作不会锁定 `ConcurrentHashMap` 的状态。它们在操作过程中对元素进行操作。提供给这些操作的功能不应依赖于任何排序或任何可能在计算过程中改变的其他对象或值。

此外，您还需要为所有这些操作指定一个并行度阈值。如果当前 `map` 的大小估计小于给定的阈值，则操作将按顺序执行。使用 `1` 的值启用最大并行性，使用公共线程池。使用 `Long.MAX_VALUE` 的值在单个线程上运行操作。

在此示例中，我们使用 `reduceValues` 方法在 `map` 中找到最大值：

```
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
Optional<Integer> maxValue =
    Optional.of(map.reduceValues(1, Integer::max));
```

注意，对于每个 `reduce` 操作，都有 `int`、`long` 和 `double` 的原始特化（例如，`reduceValuesToInt`、`reduceKeysToLong` 等）。

##### 计数

`ConcurrentHashMap` 类提供了一个名为 `mappingCount` 的新方法，它以 `long` 返回 `map` 中的映射数量。它应该用于替代返回 `int` 的方法 `size`。这是因为映射的数量可能不适合 `int`。

##### 集合视图

`ConcurrentHashMap` 类提供了一个名为 `keySet` 的新方法，它返回 `ConcurrentHashMap` 作为 `Set` 的视图（`map` 的更改反映在 `Set` 中，反之亦然）。您还可以使用新的静态方法 `newKeySet` 创建由 `ConcurrentHashMap` 支持的 `Set`。

### B.3\. Arrays

`Arrays` 类提供了各种静态方法来操作数组。现在它包括四个新的方法（它们有原始特化的重载变体）。

#### B.3.1\. 使用 parallelSort

`parallelSort` 方法以并行方式对指定的数组进行排序，使用自然顺序，或者使用额外的 `Comparator` 对对象数组进行排序。

#### B.3.2\. 使用 setAll 和 parallelSetAll

`setAll` 和 `parallelSetAll` 方法分别按顺序或并行地设置指定数组的所有元素，使用提供的函数计算每个元素。该函数接收元素索引并返回该索引的值。因为 `parallelSetAll` 是并行执行的，所以该函数必须是无副作用的，如第七章和 18 所述。

例如，您可以使用 `setAll` 方法生成包含值 0、2、4、6、... 的数组：

```
int[] evenNumbers = new int[10];
Arrays.setAll(evenNumbers, i -> i * 2);
```

#### B.3.3\. 使用 parallelPrefix

`parallelPrefix`方法并行地累积给定数组中的每个元素，使用提供的二元运算符。在下一个列表中，您将生成 1、2、3、4、5、6、7、...的值。

##### 列表 B.3. `parallelPrefix`并行累积数组元素

```
int[] ones = new int[10];
Arrays.fill(ones, 1);
Arrays.parallelPrefix(ones, (a, b) -> a + b);      *1*
```

+   ***1* ones is now [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]**

### B.4. 数字和数学

Java 8 API 通过新方法增强了`Number`和`Math`类。

#### B.4.1. 数字

`Number`类的新方法如下：

+   `Short`、`Integer`、`Long`、`Float`和`Double`类包含了`sum`、`min`和`max`静态方法。您在第五章中与`reduce`操作一起看到了这些方法。

+   `Integer`和`Long`类包括`compareUnsigned`、`divide-Unsigned`、`remainderUnsigned`和`toUnsignedString`方法，用于处理无符号值。

+   `Integer`和`Long`类分别包括静态方法`parseUnsignedInt`和`parseUnsignedLong`，用于将字符串解析为无符号的`int`或`long`。

+   `Byte`和`Short`类包括`toUnsignedInt`和`toUnsigned-Long`方法，通过无符号转换将参数转换为`int`或`long`。同样，`Integer`类现在包括静态方法`toUnsignedLong`。

+   `Double`和`Float`类包括静态方法`isFinite`，用于检查参数是否为有限的浮点值。

+   `Boolean`类现在包括静态方法`logicalAnd`、`logicalOr`和`logicalXor`，用于在两个布尔值之间应用`and`、`or`和`xor`操作。

+   `BigInteger`类包括`byteValueExact`、`shortValueExact`、`intValueExact`和`longValueExact`方法，将这些`BigInteger`转换为相应的原始类型。但如果转换过程中信息丢失，它将抛出一个算术异常。

#### B.4.2. 数学

`Math`类包括在操作结果溢出时抛出算术异常的新方法。这些方法包括`addExact`、`subtractExact`、`multiplyExact`、`incrementExact`、`decrementExact`和`negateExact`，它们具有`int`和`long`参数。此外，还有一个静态的`toIntExact`方法，用于将`long`值转换为`int`。其他增加包括静态方法`floorMod`、`floorDiv`和`nextDown`。

### B.5. 文件

对`Files`类的明显增加让您可以从文件生成流。我们在第五章中提到了新的静态方法`Files.lines`；它允许您以流的形式惰性读取文件。其他返回流的实用静态方法包括以下内容：

+   `Files.list`——生成一个包含给定目录条目的`Stream<Path>`。列表不是递归的。因为流是惰性消费的，所以这是一个处理可能非常大的目录的有用方法。

+   `Files.walk`—就像`Files.list`一样，它生成一个包含给定目录条目的`Stream<Path>`。但是列表是递归的，并且可以配置深度级别。请注意，遍历是按深度优先进行的。

+   `Files.find`—通过递归遍历目录以找到匹配给定谓词的条目，从而生成一个`Stream<Path>`。

### B.6\. 反射

我们在附录 A 中讨论了 Java 8 中注释机制的几个变化。附录 A。反射 API 已更新以支持这些变化。

反射 API 的另一个新增功能是，现在可以通过新的`java.lang.reflect.Parameter`类来访问方法参数的信息，如名称和修饰符，该类在新的`java.lang.reflect.Executable`类中被引用，该类作为`Method`和`Constructor`共同功能的一个共享超类。

### B.7\. 字符串

`String`类现在包含一个方便的静态方法`join`，正如你可能猜到的，它使用分隔符连接字符串！你可以如下使用它：

```
String authors = String.join(", ", "Raoul", "Mario", "Alan");
System.out.println(authors);                                  *1*
```

+   ***1* 拉乌尔，马里奥，艾伦**
