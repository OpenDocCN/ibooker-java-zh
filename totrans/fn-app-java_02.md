# 第一章。函数式编程简介

要更好地理解如何在 Java 中加入更多函数式编程风格，首先需要理解语言被视为函数式的含义以及其基本概念。

本章将探讨需要将更多函数式编程风格融入工作流程的函数式编程根源。

# 何谓函数式语言？

编程范式 — 如面向对象、函数式或过程化 — 是对语言进行分类并提供在特定风格中结构化程序以及使用不同方法解决问题的综合概念。像大多数范式一样，函数式编程没有一个统一的定义，人们对是否语言*实际上*是函数式进行了许多争论。与其给出我自己的定义，我将讨论构成语言函数式的不同方面。

当一个语言可以通过创建和组合抽象函数来表达计算时，它被认为是函数式的。这个概念源于 20 世纪 30 年代逻辑学家阿隆佐·丘奇发明的形式化数学系统*Lambda Calculus*。^(1) 它是一个用抽象函数表达计算及如何将变量应用于它们的系统。名称“lambda calculus”选自希腊字母“lambda”，因其符号而选择：<math alttext="lamda"><mi>λ</mi></math> 。

作为面向对象开发者，你习惯于*命令式*编程：通过定义一系列语句，告诉计算机*如何*通过一系列*语句*完成特定任务。

编程语言要被认为是函数式的，需要能够以*声明式*风格表达计算逻辑，而不描述其实际的控制流。在这样的声明式编程风格中，你描述的是结果以及*程序*应该如何工作，而不是*语句*应该做什么。

在 Java 中，*表达式*是由操作符、操作数和方法调用组成的序列，定义了一个计算并且求值为单一值：

```java
x * x
2 * Math.PI * radius
value == null ? true : false
```

*语句*，另一方面，是代码执行的动作，形成一个完整的执行单元，包括没有返回值的方法调用。每当你分配或更改变量的值，调用一个`void`方法，或使用像`if`/`else`这样的控制流构造时，你正在使用语句。通常它们与表达式混合使用：

```java
int totalTreasure = 0; ![1](img/1.png)

int newTreasuresFound = findTreasure(6); ![2](img/2.png)

totalTreasure = totalTreasure + newTreasuresFound; ![3](img/3.png)

if (treasureCounter > 10) { ![4](img/4.png)
  System.out.println("You have a lot of treasure!"); ![5](img/5.png)
} else {
  System.out.println("You should look for more treasure!"); ![5](img/5.png)
}
```

![1](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO1-1)

将初始值分配给变量，引入状态到程序中。

![2](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO1-2)

函数调用`findTreasure(6)`是一个函数式表达式，但`newTreasuresFound`的赋值则是一个语句。

![3](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO1-3)

`totalTreasure` 的重新分配是使用右侧表达式的结果作为语句。

![4](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO1-4)

控制流语句`if`/`else`根据表达式`(treasureCounter > 10)`的结果传达应采取的操作。

![5](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO1-5)

将内容打印到`System.out`是一种语句，因为调用没有返回结果。

表达式和语句的主要区别在于是否返回值。在像 Java 这样的通用多范式语言中，它们之间的界限经常是有争议的，并且很快就会变得模糊。

# 函数式编程概念

由于函数式编程主要基于抽象函数，其构成范式的许多概念可以以声明式风格集中于“如何解决”的方法，与命令式的“如何解决”方法形成对比。

我们将深入探讨函数式编程在其基础上使用的最常见和重要的方面。尽管这些并不专属于函数范式。但这些背后的许多思想也适用于其他编程范式。

## 纯函数和引用透明性

函数式编程将函数分为两类：*纯*和*不纯*。

*纯函数*具有两个基本保证：

*相同*的输入将*总是*生成相同的输出。

*纯*函数的返回值必须完全依赖于其输入参数。

它们是自包含的，没有任何副作用

代码不能影响全局状态，比如更改参数值或使用任何 I/O。

这两个保证允许*纯函数*在任何环境中都能安全使用，甚至可以并行执行。以下代码展示了一个方法作为*纯函数*，接受参数但不会影响其上下文之外的任何东西：

```java
public String toLowercase(String str) {
  return str;
}
```

违反这两个保证的函数被认为是*不纯*的。以下代码是一个*不纯函数*的例子，因为它使用当前时间来执行其逻辑：

```java
public String buildGreeting(String name) {
  var now = LocalTime.now();
  if (now.getHour() < 12) {
    return "Good morning " + name;
  } else {
    return "Hello " + name;
  }
}
```

符号“纯”和“不纯”可能会因其可能激起的内涵而不太合适。*不纯函数*在一般情况下并不比*纯函数*差。它们只是根据您想要遵循的编码风格和范式而采用不同的方式使用。

*表达式*或*纯函数*无副作用的另一个方面是它们的确定性特性，这使它们*引用透明*。这意味着您可以将它们替换为其评估结果，而不会改变程序的行为。

抽象函数：

<math alttext="StartLayout 1st Row 1st Column f left-parenthesis x right-parenthesis 2nd Column equals x asterisk x EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>f</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo></mrow></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>x</mi> <mo>*</mo> <mi>x</mi></mrow></mtd></mtr></mtable></math>

替换评估表达式：

<math alttext="StartLayout 1st Row 1st Column r e s u l t 2nd Column equals f left-parenthesis 5 right-parenthesis plus f left-parenthesis 5 right-parenthesis 2nd Row 1st Column Blank 2nd Column equals 25 plus f left-parenthesis 5 right-parenthesis 3rd Row 1st Column Blank 2nd Column equals f left-parenthesis 5 right-parenthesis plus f left-parenthesis 5 right-parenthesis 4th Row 1st Column Blank 2nd Column equals 25 plus 25 EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mrow><mi>r</mi> <mi>e</mi> <mi>s</mi> <mi>u</mi> <mi>l</mi> <mi>t</mi></mrow></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>f</mi> <mo>(</mo> <mn>5</mn> <mo>)</mo> <mo>+</mo> <mi>f</mi> <mo>(</mo> <mn>5</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mn>25</mn> <mo>+</mo> <mi>f</mi> <mo>(</mo> <mn>5</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mi>f</mi> <mo>(</mo> <mn>5</mn> <mo>)</mo> <mo>+</mo> <mi>f</mi> <mo>(</mo> <mn>5</mn> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="left"><mrow><mo>=</mo> <mn>25</mn> <mo>+</mo> <mn>25</mn></mrow></mtd></mtr></mtable></math>

所有这些变体是相等的，不会改变你的程序。纯度和引用透明度是相辅相成的，给你提供了一个强大的工具，因为理解和推理你的代码更容易。

## 不可变性

面向对象的代码通常基于可变的程序状态。对象可以并且通常会在创建后发生变化，使用 *setters*。但是，改变数据结构可能会产生意外的副作用。然而，可变性不仅限于数据结构和面向对象编程。方法中的局部变量也可能是可变的，并且可能会导致与对象的变化字段一样的问题。

使用 *不可变性*，数据结构在初始化后就无法再更改。通过永远不变化，它们始终一致，无副作用，可预测，并且更容易推理。与 *纯函数* 一样，在并发和并行环境中使用它们是安全的，而不会出现通常的未同步访问或超出范围状态更改的问题。

如果数据结构在初始化后从不改变，那么程序就不会很有用。这就是为什么你需要创建一个新的更新版本，包含了变异状态，而不是直接改变数据结构。

为每个更改创建新的数据结构可能会很麻烦，而且由于每次复制数据而变得相当低效。许多编程语言采用“结构共享”来提供高效的复制机制，以最小化要求每次更改都需要新数据结构的低效性。这样，不同实例的数据结构之间共享不可变数据。第 4 章将更详细地解释为什么具有无副作用的数据结构的优点胜过可能需要的额外工作。

## 递归

*递归* 是一种解决问题的技术，通过部分解决相同形式的问题，并将部分结果组合起来最终解决原始问题。简而言之，递归函数会调用自身，但其输入参数略有变化，直到达到结束条件并返回实际值。第 12 章将详细讨论递归的细节。

一个简单的例子是计算阶乘，即小于或等于输入参数的所有正整数的乘积。函数不是使用中间状态计算值，而是使用递减的输入变量调用自身，如图 1-1 所示。

![使用递归计算阶乘](img/afaj_0101.png)

###### 图 1-1。使用递归计算阶乘

纯函数式编程通常更喜欢使用递归而不是循环或迭代器。其中一些，如[Haskell](https://www.haskell.org)，更进一步，根本没有`for`或`while`循环。

反复调用函数可能效率低下，甚至有可能由于堆栈溢出的风险而变得危险。这就是为什么许多函数式语言利用像“展开”递归成循环或*尾递归优化*之类的优化技术来减少所需的堆栈帧。Java 不支持这些优化技术中的任何一种，我将在第十二章中详细讨论。

## 头等和高阶函数

先前讨论的许多概念不必作为深度集成的语言特性出现，以支持在代码中更加函数式的编程风格。然而，头等和高阶函数的概念是绝对必不可少的。

要使函数成为所谓的“头等公民”，它们必须遵守语言的其他实体固有的所有属性。它们需要能够分配给变量，并在其他函数和表达式中作为参数和返回值使用。

*高阶*函数使用这种*头等公民*身份来接受函数作为参数或将函数作为它们的结果返回，或者两者兼而有之。这是下一个概念*函数组合*的重要特性。

## 函数组合

*纯函数*可以组合以创建更复杂的表达式。在数学术语中，这意味着两个函数 <math alttext="f left-parenthesis x right-parenthesis"><mrow><mi>f</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo></mrow></math> 和 <math alttext="g left-parenthesis x right-parenthesis"><mrow><mi>g</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo></mrow></math> 可以组合成一个函数 <math alttext="h left-parenthesis x right-parenthesis equals g left-parenthesis f left-parenthesis x right-parenthesis right-parenthesis"><mrow><mi>h</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo> <mo>=</mo> <mi>g</mi> <mo>(</mo> <mi>f</mi> <mo>(</mo> <mi>x</mi> <mo>)</mo> <mo>)</mo></mrow></math> ，如图 1-2 所示。

![将函数 f 和 g 组合成新函数 h](img/afaj_0102.png)

###### 图 1-2\. 组合函数

这样，函数可以尽可能地小而精确，因此更易于重用。为了创建更复杂和完整的任务，此类函数可以根据需要快速组合。

## 柯里化

函数*柯里化*意味着将一个接受多个参数的函数转换为一系列每个只接受单个参数的函数。

###### 注意

柯里化技术借用了数学家和逻辑学家 Haskell Brook Curry（1900-1982）的名字。他不仅是被称为*柯里化*的函数技术的名字来源，还有三种不同的编程语言以他的名字命名：[Haskell](https://www.haskell.org/)、[Brook](http://graphics.stanford.edu/projects/brookgpu/) 和 [Curry](http://curry-lang.org/)。

想象一个接受三个参数的函数。可以如下柯里化它：

初始函数：

<math alttext="StartLayout 1st Row 1st Column x 2nd Column equals f left-parenthesis a comma b comma c right-parenthesis EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mi>x</mi></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>f</mi> <mo>(</mo> <mi>a</mi> <mo>,</mo> <mi>b</mi> <mo>,</mo> <mi>c</mi> <mo>)</mo></mrow></mtd></mtr></mtable></math>

柯里化函数：

<math alttext="StartLayout 1st Row 1st Column h 2nd Column equals g left-parenthesis a right-parenthesis 2nd Row 1st Column i 2nd Column equals h left-parenthesis b right-parenthesis 3rd Row 1st Column x 2nd Column equals i left-parenthesis c right-parenthesis EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mi>h</mi></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>g</mi> <mo>(</mo> <mi>a</mi> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mi>i</mi></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>h</mi> <mo>(</mo> <mi>b</mi> <mo>)</mo></mrow></mtd></mtr> <mtr><mtd columnalign="right"><mi>x</mi></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>i</mi> <mo>(</mo> <mi>c</mi> <mo>)</mo></mrow></mtd></mtr></mtable></math>

一系列柯里化函数：

<math alttext="StartLayout 1st Row 1st Column x 2nd Column equals g left-parenthesis a right-parenthesis left-parenthesis b right-parenthesis left-parenthesis c right-parenthesis EndLayout" display="block"><mtable displaystyle="true"><mtr><mtd columnalign="right"><mi>x</mi></mtd> <mtd columnalign="left"><mrow><mo>=</mo> <mi>g</mi> <mo>(</mo> <mi>a</mi> <mo>)</mo> <mo>(</mo> <mi>b</mi> <mo>)</mo> <mo>(</mo> <mi>c</mi> <mo>)</mo></mrow></mtd></mtr></mtable></math>

一些功能性编程语言在其类型定义中反映了*柯里化*的一般概念，如 Haskell 如下所示。

```java
add :: Integer -> Integer -> Integer ![1](img/1.png)
add x y =  x + y ![2](img/2.png)
```

![1](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO2-1)

函数`add`被声明为接受一个`Integer`并返回另一个接受另一个`Integer`的函数，其本身返回一个`Integer`。

![2](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO2-2)

实际的定义反映了声明：两个输入参数和主体的结果作为返回值。

乍一看，这个概念可能对面向对象或命令式开发者感觉奇怪和陌生，就像许多基于数学的原则一样。但它完美地传达了一个多参数函数如何可以表示为函数的函数，这是支持下一个概念的重要认识。

## 部分函数应用

*部分函数应用*是通过为现有函数提供不完整的参数来创建新函数的过程。它经常与*柯里化*混淆，但对部分应用的函数的调用返回一个结果，而不是柯里化链中的另一个函数。

前一节的柯里化示例可以部分应用以创建更具体的函数：

```java
add :: Integer -> Integer -> Integer ![1](img/1.png)
add x y =  x + y

add3 = add 3 ![2](img/2.png)

add3 5 ![3](img/3.png)
```

![1](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO3-1)

函数`add`与之前一样声明，接受两个参数。

![2](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO3-2)

调用函数`add`仅为第一个参数`x`的值返回类型为`Integer → Integer`的部分应用函数，其绑定到名称`add3`。

![3](img/#co_an_introduction_to__span_class__keep_together__functional_programming__span__CO3-3)

调用`add3 5`等同于`add 3 5`。

通过部分应用，您可以即时创建新的更简洁的函数或从更通用的池中创建专门的函数以匹配代码当前的上下文和要求。

## 惰性求值

*惰性求值*是一种评估策略，它延迟对表达式的评估，直到其结果确实被需要，通过将创建表达式的方式与实际使用它的时间或方式分开。这也是另一个不仅根植于功能性编程，而且是使用其他功能性概念和技术的必备概念。

许多非函数式语言，包括 Java，在本质上是*严格*或*急切*评估的，意味着表达式会立即评估。这些语言仍然具有一些惰性构造，如控制流语句（如`if`-`else`语句或循环）或逻辑短路运算符。立即评估`if`-`else`结构的两个分支或所有可能的循环迭代都没有多大意义，是吧？因此，运行时只评估绝对需要的分支和迭代。

惰性使得某些否则不可能的构造成为可能，例如无限数据结构或某些算法的更有效实现。它还与*引用透明性*非常配合。如果表达式和其结果没有区别，你可以延迟评估而不影响结果。延迟评估可能仍会影响程序的性能，因为你可能不知道评估的确切时间。

在第十一章中，我将讨论如何使用你手头的工具在 Java 中实现惰性评估，并且如何创建你自己的方法。

# 函数式编程的优点

经过对函数式编程最常见和基本概念的深入了解后，你可以看到它们如何体现在更加函数化方法所提供的优势中：

简单性

没有可变状态和副作用，你的函数往往更小，只做“它们应该做的事情”。

一致性

不可变数据结构是可靠且一致的。不再担心意外或意外的程序状态。

（数学）正确性

使用一致的数据结构编写更简单的代码将自动导致“更正确”的代码，同时 bug 的表面更小。你的代码越“纯粹”，就越容易理解，从而更容易进行简化的调试和测试。

更安全的并发性

并发是“传统”Java 中最具挑战性的任务之一。函数式概念使你能够消除许多烦恼，并几乎免费地获得更安全的并行处理。

模块化

小而独立的函数使得更简单的可重用性和模块化成为可能。结合函数组合和部分应用，你可以轻松地从这些更小的部分构建更复杂的任务。

可测试性

许多函数式概念，如纯函数、引用透明性、不可变性和关注点分离，使得测试和验证变得更加容易。

# 函数式编程的缺点

虽然函数式编程有许多优点，但了解其可能的陷阱也很重要。

学习曲线

函数式编程所基于的高级数学术语和概念可能会让人望而生畏。然而，要增强你的 Java 代码，你绝对不需要知道“一个单子只是一个自函子类别中的幺半群^(2)”然而，你会面对新的、通常是陌生的术语和概念。

更高的抽象水平

OOP 使用对象来建模其抽象，而 FP 使用更高级别的抽象来表示其数据结构，使它们变得相当优雅，但通常更难以识别。

处理状态

处理状态并非易事，无论选择何种范式。即使 FP 的不可变方法消除了许多可能的错误表面，但如果实际上需要更改数据结构，则更难以变异，特别是如果你习惯于在你的 OO 代码中有 setter。

性能影响

在并发环境中，函数式编程更易于使用且更安全。然而，这并不意味着与其他范式相比它本质上更快，特别是在单线程环境中。尽管有许多好处，但许多函数式技术，如不可变性或递归，可能会因所需的开销而受到影响。这就是为什么许多函数式编程语言利用大量优化来减轻负担，比如最小化复制的专用数据结构，或者用于递归等技术的编译器优化^(3)。

最佳问题背景

并非所有问题背景都适合采用函数式方法。高性能计算（HPC）、I/O 密集问题或低级系统和嵌入式控制器等领域，需要对诸如数据局部性和显式内存管理等事物进行精细控制，与函数式编程不太相容。

作为程序员，我们必须在任何范式和编程方法的优势和劣势之间找到平衡。这就是为什么这本书向你展示如何选择 Java 函数式进化的最佳部分，并利用它们来增强你的面向对象的 Java 代码。

# 收获

+   函数式编程是建立在 lambda 演算的数学原理之上的。

+   基于表达式而不是语句的声明式编码风格对函数式编程至关重要。

+   许多编程概念在本质上都感觉像是函数式的，但这并不是使语言或代码“函数式”的绝对要求。即使非函数式代码也从它们的基本思想和整体思维中受益。

+   纯度、一致性和简单性是将这些属性应用到你的代码中以获得函数式方法最大优势的关键。

+   函数式概念和它们在现实世界中的应用之间可能需要权衡。它们的优势通常会超过劣势，或者至少可以以某种形式加以减轻。

^(1) Church, Alonzo. 1936\. “初等数论中的一个不可解问题。”《美国数学杂志》, Vol. 58, 345-363.](https://doi.org/10.2307/2268571)

^(2) James Iry 在他幽默的博客文章[“编程语言简史：简洁、不完整且大部分错误”](http://james-iry.blogspot.com/2009/05/brief-incomplete-and-mostly-wrong.xhtml)中使用这个短语，来说明 Haskell 的复杂性。这也是一个很好的例子，说明你不需要了解编程技术背后的所有数学细节就能享受其带来的好处。但如果你真的想知道它的含义，可以参考 Saunders Mac Lane 的书籍，《工作数学家的范畴》（Springer, 1998），这本书最初使用了这个短语。

^(3) 《Java Magazine》的文章[“花括号 #6：递归与尾调用优化”](https://blogs.oracle.com/javamagazine/post/curly-braces-java-recursion-tail-call-optimization)详细介绍了递归代码中尾调用优化的重要性。
