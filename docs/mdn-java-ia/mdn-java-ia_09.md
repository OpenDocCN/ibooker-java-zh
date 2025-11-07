## 附录 D. Lambda 和 JVM 字节码

您可能会想知道 Java 编译器如何实现 lambda 表达式，以及 Java 虚拟机 (JVM) 如何处理它。如果您认为 lambda 表达式可以简单地翻译成匿名类，请继续阅读。本附录简要讨论了 lambda 表达式是如何编译的，通过检查生成的类文件。

### D.1. 匿名类

我们在 第二章 中展示了匿名类可以同时声明和实例化一个类。因此，就像 lambda 表达式一样，它们可以用来为函数式接口提供实现。

由于 lambda 表达式为函数式接口的抽象方法提供了实现，因此似乎在编译过程中要求 Java 编译器将 lambda 表达式翻译成匿名类是直截了当的。但是匿名类有一些不希望的特性，这些特性会影响应用程序的性能：

+   *编译器为每个匿名类生成一个新的类文件。* 文件名通常看起来像 `ClassName$1`，其中 `ClassName` 是匿名类出现的类的名称，后面跟着一个美元符号和一个数字。生成许多类文件是不希望的，因为每个类文件在使用之前都需要被加载和验证，这会影响应用程序的启动性能。如果 lambda 被翻译成匿名类，那么每个 lambda 就会有一个新的类文件。

+   *每个新的匿名类都会为类或接口引入一个新的子类型。* 如果您有表达 `Comparator` 的 100 个不同的 lambda，那么这意味着 100 个不同的 `Comparator` 子类型。在某些情况下，这可能会使 JVM 提高运行时性能变得更加困难。

### D.2. 字节码生成

一个 Java 源文件由 Java 编译器编译成 Java 字节码。然后 JVM 可以执行生成的字节码并运行应用程序。匿名类和 lambda 表达式在编译时使用不同的字节码指令。您可以使用以下命令检查任何类文件的字节码和常量池：

```
javap -c -v ClassName
```

让我们尝试使用旧的 Java 7 语法实现 `Function` 接口的实例，作为一个匿名内部类，如下所示。

##### 列表 D.1. 作为匿名内部类实现的 `Function`

```
import java.util.function.Function;
public class InnerClass {
    Function<Object, String> f = new Function<Object, String>() {
        @Override
        public String apply(Object obj) {
            return obj.toString();
        }
    };
}
```

这样做，作为匿名内部类创建的 `Function` 对应生成的字节码将大致如下：

```
 0: aload_0
 1: invokespecial #1        // Method java/lang/Object."<init>":()V
 4: aload_0
 5: new           #2        // class InnerClass$1
 8: dup
 9: aload_0
10: invokespecial #3        // Method InnerClass$1."<init>":(LInnerClass;)V
13: putfield      #4        // Field f:Ljava/util/function/Function;
16: return
```

这段代码展示了以下内容：

+   使用字节码操作 `new` 实例化类型为 `InnerClass$1` 的对象。同时将新创建的对象引用压入栈中。

+   操作 `dup` 复制栈上的那个引用。

+   此值随后被指令 `invokespecial` 消耗，该指令初始化对象。

+   栈顶现在仍然包含对对象的引用，该对象使用 `putfield` 指令存储在 `LambdaBytecode` 类的 `f1` 字段中。

`InnerClass$1` 是编译器为匿名类生成的名称。如果你想确认这一点，你也可以检查 `InnerClass$1` 类文件，你将找到实现 `Function` 接口的代码：

```
class InnerClass$1 implements
          java.util.function.Function<java.lang.Object, java.lang.String> {
  final InnerClass this$0;
  public java.lang.String apply(java.lang.Object);
    Code:
       0: aload_1
       1: invokevirtual #3 //Method
                             java/lang/Object.toString:()Ljava/lang/String;
       4: areturn
}
```

### D.3\. Invokedynamic to the rescue

现在，让我们尝试使用新的 Java 8 语法，即 lambda 表达式，来做同样的事情。检查以下列表中代码生成的类文件。

##### 列表 D.2\. 使用 lambda 表达式实现的 `Function`

```
import java.util.function.Function;
public class Lambda {
    Function<Object, String> f = obj -> obj.toString();
}
```

你将找到以下字节码指令：

```
 0: aload_0
 1: invokespecial #1     // Method java/lang/Object."<init>":()V
 4: aload_0
 5: invokedynamic #2,  0 // InvokeDynamic
                            #0:apply:()Ljava/util/function/Function;
10: putfield      #3     // Field f:Ljava/util/function/Function;
13: return
```

我们解释了在匿名内部类中翻译 lambda 表达式的缺点，确实你可以看到结果非常不同。额外类的创建已被 `invokedynamic` 指令所取代。

|  |
| --- |

**invokedynamic 指令**

`invokedynamic` 字节码指令是在 JDK7 中引入的，以支持 JVM 上的动态类型语言。`invokedynamic` 在调用方法时添加了另一层间接性，以便让一些依赖于特定动态语言的逻辑确定调用目标。此指令的典型用途如下：

```
    def add(a, b) { a + b }
```

在这里，`a` 和 `b` 的类型在编译时是未知的，并且可能会不时地改变。因此，当 JVM 首次执行 `invokedynamic` 时，它会咨询一个启动方法，该启动方法实现了确定要调用实际方法的与语言相关的逻辑。启动方法返回一个链接的调用点。如果 `add` 方法用两个 `int` 调用，那么随后的调用也很可能也是用两个 `int` 调用。因此，在每次调用时都不需要重新发现要调用的方法。调用点本身可以包含定义在什么条件下需要重新链接的逻辑。

|  |
| --- |

在列表 D.2 中，`invokedynamic` 指令被用于一个与最初引入时的不同目的。实际上，这里它被用来延迟在字节码中翻译 lambda 表达式的策略，直到运行时。换句话说，以这种方式使用 `invokedynamic` 允许将 lambda 表达式的实现代码生成推迟到运行时。这种设计选择有积极的影响：

+   将 lambda 表达式体翻译成字节码的策略变成了一个纯粹的实施细节。它也可以在动态中更改，或者在未来的 JVM 实现中进行优化和修改，以保持字节码的向后兼容性。

+   如果 lambda 从未使用，则没有开销，例如额外的字段或静态初始化器。

+   对于无状态（非捕获）lambda，可以创建一个 lambda 对象的实例，将其缓存，并始终返回相同的实例。这是一个常见的用例，在 Java 8 之前，人们习惯于显式地这样做；例如，在静态最终变量中声明特定的 `Comparator` 实例。

+   由于这个翻译必须在 lambda 首次被调用时执行，并且其结果被链接，所以没有额外的性能开销。所有后续调用都可以跳过这个慢路径，并调用之前链接的实现。

### D.4. 代码生成策略

lambda 表达式通过将其主体放入在运行时创建的静态方法之一中，被翻译成字节码。无状态 lambda，即不捕获其封闭作用域中的任何状态的 lambda，如我们在列表 D.2 中定义的那样，是最简单的 lambda 类型，需要被翻译。在这种情况下，编译器可以生成一个与 lambda 表达式具有相同签名的方 法，因此这个翻译过程的最终结果可以逻辑上看作如下：

```
public class Lambda {
    Function<Object, String> f = [dynamic invocation of lambda$1]

    static String lambda$1(Object obj) {
        return obj.toString();
    }
}
```

在以下示例中，lambda 表达式捕获最终（或实际上是最终）局部变量或字段的情况，要复杂一些：

```
public class Lambda {
    String header = "This is a ";
    Function<Object, String> f = obj -> header + obj.toString();
}
```

在这种情况下，生成的方法的签名不能与 lambda 表达式相同，因为需要添加额外的参数来携带封闭上下文的额外状态。实现这个目标的最简单方法是，在 lambda 表达式的参数前面添加一个额外的参数，用于每个捕获的变量，因此用于实现前一个 lambda 表达式的生成方法将类似于以下这样：

```
public class Lambda {
    String header = "This is a ";
    Function<Object, String> f = [dynamic invocation of lambda$1]

    static String lambda$1(String header, Object obj) {
        return obj -> header + obj.toString();
    }
}
```

关于 lambda 表达式翻译过程的更多信息，可以在这里找到：[`cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html`](http://cr.openjdk.java.net/~briangoetz/lambda/lambda-translation.html)。
