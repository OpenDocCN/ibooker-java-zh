# 前言

掌握软件开发需要学习一系列不同的概念。如果你是初级软件开发者，或者即使你已经有经验了，这看起来也像是一个无法逾越的障碍。你是否应该花时间学习面向对象世界中已被广泛接受的主题，如 SOLID 原则、设计模式或测试驱动开发？你是否应该尝试一些越来越流行的东西，如函数式编程？

即使你已经选择了一些要学习的主题，确定它们是如何相互联系的仍然很难。当你应该在项目中应用函数式编程思想的路线时？什么时候该关注测试？你怎么知道在什么时间引入或改进这些技术？你需要读每个主题的书，然后再看另一系列博客或视频来解释如何将它们结合在一起吗？你甚至不知道从哪里开始？

不用担心，这本书会帮助你。你将通过一种集成、以项目为驱动的学习方式得到帮助。你将学习成为高效开发者所需的核心主题。不仅如此，我们还展示了这些内容是如何融入更大项目中的。

# 为什么我们写这本书

多年来，我们积累了丰富的经验，帮助开发者学习编程。我们都写过关于 Java 8 及以后的书籍，并开设过专业软件开发的培训课程。在这个过程中，我们被认可为 Java 社区的杰出贡献者和国际会议的演讲者。

多年来，我们发现许多开发者需要对一些核心主题进行入门或复习。设计模式、函数式编程、SOLID 原则和测试等实践经常得到了良好的覆盖，但很少有人展示它们是如何良好地协同工作和相互结合的。有时人们甚至因为选择学习内容的犹豫而放弃提升自己的技能。我们不仅希望教会人们核心技能，还希望用一种易于接近和有趣的方式进行教学。

# 开发者导向的方法

这本书还为你提供了一种开发者导向的学习机会。它包含了大量的代码示例，每当我们介绍一个主题时，我们总是提供具体的代码示例。你可以得到书中项目的所有代码，所以如果你想跟着做，你甚至可以在*集成开发环境*（IDE）中逐步运行代码，或者运行程序来试验它们。

技术书籍的另一个常见问题是，它们通常采用正式的讲述风格。这不是普通人之间交流的方式！在这本书中，你将得到一种对话式的风格，能帮助你更好地投入到内容中，而不是让你感到被指责。

# 书中内容

每一章都围绕一个软件项目进行结构化。在一章的结尾，如果您一直在跟进，您应该能够编写那个项目。这些项目从简单的命令行批处理程序开始，逐渐发展到完整的应用程序。

通过项目驱动的结构，您将从多个方面受益。首先，您可以看到不同的编程技术如何在集成的环境中协同工作。当我们在书的末尾讨论函数式编程时，这不仅仅是抽象的集合处理操作——它们被呈现出来是为了计算实际项目中使用的结果。这解决了教育材料展示好的想法或方法，但开发人员经常不适当或上下文不当使用它们的问题。

其次，项目驱动的方法有助于确保每个阶段您都能看到现实的例子。教育材料通常充满了名为`Foo`的示例类和名为`bar`的方法。我们的例子与相关的项目有关，并展示如何将这些想法应用到真正的问题中，这些问题与您在职业生涯中可能遇到的类似。

最后，通过这种方式学习更有趣且更引人入胜。每一章都是一个全新的项目和学习新事物的机会。我们希望您能一直读到最后，真正享受阅读过程。每章都以一个挑战开始，并解决该挑战，然后结束评估您学到了什么以及如何解决这个挑战。我们特别在每章的开头和结尾明确指出挑战，以确保您理解其目标。

# 谁应该阅读这本书？

我们确信，来自各种背景的开发人员会在这本书中找到有用和有趣的内容。话虽如此，有些人会从这本书中获得最大的价值。

初级软件开发人员，通常刚从大学毕业或者从事编程职业几年，是我们认为这本书的核心读者群体。您将学习到我们预计在整个软件开发职业生涯中都具有相关性的基础主题。您并不需要有大学学位，但需要了解编程的基础知识，以便最好地利用这本书。例如，我们不会解释什么是 if 语句或循环。

您不需要对面向对象或函数式编程有很多了解就可以开始学习。在第二章，我们不做任何假设，只要求您知道什么是类，并且能够使用泛型的集合（例如，`List<String>`）。我们从基础知识开始。

另一组特别感兴趣的读者是从其他编程语言（如 C＃、C ++或 Python）学习 Java 的开发人员。这本书帮助你快速掌握语言结构，以及编写优秀 Java 代码所需的原则、实践和习惯。

如果您是一位经验丰富的 Java 开发者，可能希望跳过第二章，以避免重复已掌握的基础内容，但从第三章开始，将充满对许多开发人员有益的概念和方法。

我们发现学习可以是软件开发中最有趣的部分之一，希望您在阅读本书时也能有同样的感受。祝您在这段旅程中玩得开心。

# 本书使用的约定

本书采用以下排版约定：

*Italic*

表示新术语、URL、电子邮件地址、文件名和文件扩展名。

`Constant width`

用于程序清单，以及段落内引用程序元素（如变量或函数名称、数据库、数据类型、环境变量、语句和关键字）。

**`Constant width bold`**

显示用户应按字面意义输入的命令或其他文本。

*`Constant width italic`*

显示应由用户提供值或由上下文确定值的文本。

###### 注意

此元素表示一般说明。

# 使用代码示例

可以下载补充材料（代码示例、练习等）[*https://github.com/Iteratr-Learning/Real-World-Software-Development*](https://github.com/Iteratr-Learning/Real-World-Software-Development)。

如果您有技术问题或在使用代码示例时遇到问题，请发送电子邮件至*bookquestions@oreilly.com*。

这本书旨在帮助您完成工作。一般而言，如果本书提供了示例代码，您可以在自己的程序和文档中使用它。除非您复制了大量代码，否则无需征得我们的许可。例如，编写一个使用本书中多个代码片段的程序不需要许可。出售或分发 O’Reilly 书籍的示例代码需要许可。引用本书并引用示例代码来回答问题无需许可。将本书中大量示例代码合并到产品文档中需要许可。

我们感谢但通常不要求署名。署名通常包括标题、作者、出版社和 ISBN。例如：“《*Real-World Software Development*》由 Raoul-Gabriel Urma 和 Richard Warburton（O’Reilly）著，版权所有 2020 年由 Functor Ltd.和 Monotonic Ltd.出版，ISBN 978-1-491-96717-1。”

如果您认为您使用的代码示例超出了合理使用范围或上述许可，请随时联系我们：*permissions@oreilly.com*。
