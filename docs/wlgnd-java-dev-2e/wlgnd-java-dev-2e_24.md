# 附录 A. 选择您的 Java

随着 Oracle JDK 分发和支持的变化，关于使用 Oracle JDK 与 Oracle OpenJDK 构建或其他提供商的 OpenJDK 构建的权利存在相当大的不确定性。存在多种方式可以获得免费更新（包括安全更新）和来自多个供应商提供的（新和现有）付费支持模型。关于这个主题的完整说明，请参阅 Java Champs 社区的开创性指南：“Java Is Still Free”，网址为[`mng.bz/Qvdw`](http://mng.bz/Qvdw)，该社区是 Java 行业中的独立 Java 领导者团体。[`dev.java.community/jcs/`](https://dev.java.community/jcs/)

## A.1 Java 仍然免费

您仍然可以通过多个提供商（包括 Oracle）在 GPLv2+CE 许可的完全自由下获取 OpenJDK 构建（请参阅[`openjdk.java.net/legal/gplv2+ce.html`](https://openjdk.java.net/legal/gplv2+ce.html)）。在某些情况下，Oracle JDK 仍然免费（无成本）。本节其余部分将详细介绍这一点的确切细微差别。

### A.1.1 Java SE/OpenJDK/Oracle OpenJDK 构建/Oracle JDK

OpenJDK 社区根据 Java 社区过程（JCP）和通过每个功能发布的一个总括 Java 规范请求（JSR）定义，创建并维护 Java SE 规范的（GPLv2+CE）开源参考实现（RI）。Java SE 的实现（主要是基于 OpenJDK 的）可以从各种提供商处获得，例如阿里巴巴、Amazon、Azul、BellSoft、Eclipse Adoptium（AdoptOpenJDK 的继任者）、IBM、Microsoft、Red Hat、Oracle、SAP 等。

Oracle JDK 8 已经完成了“公共更新结束”的过程，这意味着从 2019 年 4 月开始的更新需要生产使用支持合同。如前所述，您可以从其他提供商完全免费地获取 OpenJDK 8、11 和 17 构建。Oracle 还提供了 Oracle JDK 17 的零成本二进制文件。

您有多种获取 JDK 的选择；本文件重点介绍 Java SE 8、11 和 17。

## A.2 保持使用 Java SE 8

由于各种原因，有些人希望继续使用 Java SE 8：

1.  自 2019 年 4 月更新以来，Oracle JDK 8 已经受到商业使用限制。要获取更新的 Java SE 8 二进制文件，用户可以获取 Oracle JDK 8 的付费支持计划或使用来自其他提供商的 Java SE 8/OpenJDK 8 二进制文件。

1.  如果您不使用 Oracle JDK 8，您的当前 Java SE 8/OpenJDK 8 提供商可能提供更新或付费支持计划。

### A.2.1 $免费（就像啤酒一样）和免费（就像使用一样）Java SE 8

如果您想免费更新 Java SE 8（包括安全更新），请使用通过 TCK 测试的 OpenJDK 发行版，例如 Amazon、Azul、BellSoft、Eclipse Adoptium、IBM、Microsoft、Red Hat、SAP 等。

## A.3 获取 Java SE 11

您可以从以下选项中选择。请仔细阅读它们，特别是 Oracle JDK 如何管理 Java SE 11 的发布和更新：

1.  对于 Java SE 11，Oracle 通过以下方式提供他们的（基于 OpenJDK 的）JDK：

    1.  *Oracle OpenJDK 11 构建*——在 GPLv2+CE 许可下

    1.  *Oracle JDK*—在付费的商业许可下（但对于个人使用、开发、测试、原型制作和演示以及某些类型的应用程序是免费的），对于那些不想使用 GPLv2+CE 或正在使用与 Oracle 产品或服务一起的 Oracle JDK 的人。

1.  您还可以从各种其他提供商那里获取 Java SE / OpenJDK 的二进制分发。这些提供商为您提供了不同时间段的更新（包括安全更新），但对于 LTS 版本通常是更长时间。

### A.3.1 $免费（就像啤酒一样）和免费（就像使用一样）的 Java SE 11

如果您想要 Java SE 11 的免费更新（包括安全更新），请使用通过 TCK 的 OpenJDK 分发，例如，Amazon、Azul、BellSoft、Eclipse Adoptium、IBM、Microsoft、Red Hat、SAP 等。

## A.4 获取 Java SE 17（LTS）

您可以从以下选项中选择。请仔细阅读它们，特别是 Oracle JDK 如何管理 Java SE 17 的发布和更新：

1.  从 Java SE 17 开始，Oracle 通过以下方式提供他们的（基于 OpenJDK 的）JDK：

    1.  *Oracle OpenJDK 构建*—在 GPLv2+CE 许可下

    1.  *Oracle JDK*—在三年内的无费条款和条件（NFTC）许可之后，然后是常规的商业许可

1.  您还可以从各种其他提供商那里获取 Java SE/OpenJDK 的二进制分发。这些提供商为您提供了不同时间段的更新（包括安全更新），但对于 LTS 版本通常是更长时间。

注意：NFTC 许可对 Oracle JDK 17 的免费再分发有一些限制。请确保您仔细阅读许可的详细信息。

### A.4.1 $免费（就像啤酒一样）和免费（就像使用一样）的 Java SE 17

如果您想要 Java SE 17 的免费更新（包括安全更新），请使用通过 TCK 的 OpenJDK 分发，例如 Amazon、Azul、BellSoft、Eclipse Adoptium、IBM、Microsoft、Red Hat、SAP 等。

## A.5 付费支持

对于来自 Azul、BellSoft、IBM、Oracle、Red Hat 等公司的 Java SE/OpenJDK 8、11 和 17 的二进制文件，存在广泛的付费支持选项。Azul 还提供中期支持版本。
