# 附录 E. 有用资源

## E.1 有用的 Fluentd 资源

这些是与 Fluentd 直接相关并由 Fluentd 社区支持的资源。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| Fluentd 官方文档 | [`docs.fluentd.org/`](https://docs.fluentd.org/) | Fluentd 的官方 GitHub 文档 |
| Slack | [`slack.fluentd.org`](https://slack.fluentd.org/) | Fluentd 社区在 Slack 上 |
| Fluentd 的 Stack Overflow | [`stackoverflow.com/questions/tagged/fluentd`](https://stackoverflow.com/questions/tagged/fluentd) | 与 Fluentd 相关的 Stack Overflow 内容 |

## E.2 有用的 Fluentd 第三方工具

Fluentd 是用 Ruby 编写的，如果您正在编写 Fluentd 插件，这些资源将非常有帮助。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| Fluentular | [`fluentular.herokuapp.com/`](https://fluentular.herokuapp.com/) | 用于帮助验证 Fluentd 配置中的正则表达式的实用工具。 |
| Grok 解析器 | [`mng.bz/nYJa`](https://mng.bz/nYJa) | 这使用基于 Grok 的方法从日志条目中提取详细信息。它包括多行支持。 |
| 微软的 Visual Studio Code | [`code.visualstudio.com/`](https://code.visualstudio.com/) | 通过插件框架支持大量语言和语法的免费 IDE。 |
| 多格式解析器 | [`mng.bz/vo17`](https://mng.bz/vo17) | 尝试按照定义的顺序使用不同的格式模式以获得匹配。 |
| VS Code—Fluentd 插件 | [`github.com/msysyamamoto/vscode-fluentd`](https://github.com/msysyamamoto/vscode-fluentd) | 在 Visual Studio Code 中搜索 msysyamamoto.vscode-fluentd。 |
| VS Code 插件的网站版本 | [`regexper.com/`](https://regexper.com/) | 提供正则表达式的可视化表示。有助于解决正确的分组等问题。 |

## E.3 有用的日志记录实践资源

有效使用日志的关键是良好的日志记录。我们提供了许多关于推荐实践的见解。但如果您想了解他人的看法，那么这些资源可能会有所帮助。

| 描述 | 网址 |
| --- | --- |
| Logz.io 的日志最佳实践。 | [`logz.io/blog/logging-best-practices/`](https://logz.io/blog/logging-best-practices/) |
| Loggly 日志指南—涵盖多种语言视角。 | [`www.loggly.com/ultimate-guide/`](https://www.loggly.com/ultimate-guide/) |
| Loggly 的《实用日志手册》。 | [`mng.bz/4j1w`](https://mng.bz/4j1w) |
| 国家标准与技术研究院（NIST）计算机安全日志管理指南 | [`mng.bz/QW8G`](https://mng.bz/QW8G) |
| 这定义了 Syslog 的标准；除了帮助您更好地理解 Syslog 之外，它还包含了一些关于日志记录实践的好想法。 | [`tools.ietf.org/html/rfc5424`](https://tools.ietf.org/html/rfc5424) |

## E.4 常见日志格式及描述

行业已经为日志文件结构开发了几个事实上的或正式化的行业标准。以下是对这些行业规范的参考。

| 日志格式 | 定义参考 |
| --- | --- |
| Apache HTTP 日志 | [`httpd.apache.org/docs/2.4/logs.html`](https://httpd.apache.org/docs/2.4/logs.html) |
| 通用事件格式 (CEF) | [`mng.bz/XW5v`](https://mng.bz/XW5v) |
| Graylog 扩展日志格式 (GELF) | [`mng.bz/y4QB`](https://mng.bz/y4QB) |
| Nginx 日志 | [`mng.bz/M2BW`](https://mng.bz/M2BW) |
| Syslog | [`tools.ietf.org/html/rfc5424`](https://tools.ietf.org/html/rfc5424) |
| Systemd 日志 | [`systemd.io/JOURNAL_FILE_FORMAT/`](https://systemd.io/JOURNAL_FILE_FORMAT/) |
| W3C 扩展日志文件格式 (ELF) | [www.w3.org/TR/WD-logfile.html](https://www.w3.org/TR/WD-logfile.html) |
| WinLoG（Windows 本地日志） | [`mng.bz/aD17`](https://mng.bz/aD17) |

## E.5 有用的 Ruby 资源

Fluentd 的主体是用 Ruby 构建的，如果你正在编写 Fluentd 插件，你可能需要这些资源。

| 名称 | URL | 描述 |
| --- | --- | --- |
| 全局解释器锁 (GIL) 的解释 | [`thoughtbot.com/blog/untangling-ruby-threads`](https://thoughtbot.com/blog/untangling-ruby-threads) | Ruby 使用全局解释器锁，这影响了线程的管理方式以及如何调整 Fluentd。 |
| Gem 规范 | [`mng.bz/g4BV`](https://mng.bz/g4BV) | 定义了打包自定义开发的插件所需的 gemspec 文件细节。 |
| Gemspec 与 Gemfile 的比较 | [`mng.bz/5Kwa`](https://mng.bz/5Kwa) | 解释了 Gemspec 和 Gemfile 之间的区别。 |
| Minitest——Ruby 单元测试框架 | [`github.com/seattlerb/minitest`](https://github.com/seattlerb/minitest) | 另一个流行的单元测试框架。此资源包括帮助区分它与其他常用框架的信息。 |
| Rake | [`github.com/ruby/rake`](https://github.com/ruby/rake) | 提供了一个可以集成到 CI/CD 管道中的构建过程。 |
| rbenv | [`github.com/rbenv/rbenv`](https://github.com/rbenv/rbenv) | 这个开源工具使得确保不同的应用程序可以使用不同的 Ruby 版本进行工作变得更加容易。 |
| RDoc | [`github.com/ruby/rdoc`](https://github.com/ruby/rdoc) | 标准的 Ruby-Doc 生成工具。如果你需要生成文档但不想使用 Yard 的扩展，那么 RDoc 是标准的。 |
| RSpec Ruby 单元测试框架 | [`rspec.info/`](https://rspec.info/) | RSpec 是 Ruby 单元测试的替代品，它支持测试驱动开发 (TDD) 的开发方法。 |
| RuboCop | [`docs.rubocop.org/rubocop/ installation.html`](https://docs.rubocop.org/rubocop/installation.html) | Ruby 的 Lint 工具。如果你在开发自定义插件，那么它是有价值的。 |
| Ruby API | [`rubyapi.org/`](https://rubyapi.org/) | Ruby 文档工具。 |
| RubyGems | [`rubygems.org/`](https://rubygems.org/) | RubyGems 目录以及 RubyGems 管理器。 |
| RubyGuides | [www.rubyguides.com](https://www.rubyguides.com) | 一套使用 Ruby 实现特定功能的在线指南。 |
| Ruby in Twenty Minutes | [www.ruby-lang.org/en/documentation/quickstart/](https://www.ruby-lang.org/en/documentation/quickstart/) | 使用 Hello World 介绍 Ruby 的优秀入门指南。 |
| *Ruby in Practice* | [www.manning.com/books/ruby-in-practice](https://www.manning.com/books/ruby-in-practice) | *Ruby in Practice* 是来自 Manning Publications 的第二本书，采用不同的方法教授 Ruby 开发。 |
| RubyInstaller | [`rubyinstaller.org/`](https://rubyinstaller.org/) | Ruby 的 Windows 安装程序。 |
| Ruby 语言规范和文档 | [www.ruby-lang.org/en/documentation/](https://www.ruby-lang.org/en/documentation/) | 包括 Ruby 与其他语言（如 Java）不同的摘要页面。 |
| ruby-lint | [`rubygems.org/gems/ruby-lint/`](https://rubygems.org/gems/ruby-lint/) | 在开发自定义插件时，lint 工具将帮助保持代码整洁，并且重要的是，帮助发现任何潜在的错误。 |
| Ruby 线程 | [`thoughtbot.com/blog/untangling-ruby -threads`](https://thoughtbot.com/blog/untangling-ruby-threads) | 对 Ruby 线程的更详细探讨。 |
| Ruby 单元测试框架 | [`test-unit.github.io/`](https://test-unit.github.io/) | Fluentd 提供了额外的支持资源，使插件能够使用 test-unit 工具轻松进行测试。 |
| test-unit | [`github.com/test-unit/test-unit`](https://github.com/test-unit/test-unit) | 基于 xUnit 的 Ruby 单元测试解决方案。 |
| *The Well-Grounded Rubyist, third edition* | [www.manning.com/books/the-well-grounded -rubyist-third-edition](https://www.manning.com/books/the-well-grounded-rubyist-third-edition) | Manning 关于 Ruby 开发的圣经。 |
| VSCode Ruby 插件 | [`marketplace.visualstudio.com/items?itemName=wingrunr21.vscode-ruby`](https://marketplace.visualstudio.com/items?itemName=wingrunr21.vscode-ruby) | 在 Microsoft 的 Visual Studio Code 中为 Ruby 提供语法感知高亮显示。 |
| YARD | [`yardoc.org/`](https://yardoc.org/) | 符合 RubyDoc 规范的文档生成器，同时也支持标签符号，有助于提供更全面的元数据。 |

## E.6 Docker 和 Kubernetes

Docker 是最常用的容器化技术，通常与 Kubernetes 一起使用。Kubernetes 和 CNCF 的采用对 Fluentd 的采用产生了重大影响。如果您想了解更多关于 Kubernetes 的信息，以下资源将有所帮助。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| Docker | [www.docker.com](https://www.docker.com) | Docker 生态系统的家园。 |
| Docker Hub | [`hub.docker.com/`](https://hub.docker.com/) | Docker 镜像仓库，包括 Fluentd 镜像和本书使用的其他镜像。 |
| Visual Studio Code 的 Docker 插件 | [`mng.bz/6ZDA`](https://mng.bz/6ZDA) | 帮助理解 Docker 文件和文件语法的插件。 |
| Kubernetes 的家园 | [`kubernetes.io`](https://kubernetes.io) | 官方的 Kubernetes 网站。 |
| Kubernetes 日志（klog） | [`github.com/kubernetes/klog`](https://github.com/kubernetes/klog) | Kubernetes klog 组件的实现和文档。Klog 正在被 Kubernetes 采用作为默认的本地记录器。 |
| logrotate | [`github.com/logrotate/logrotate`](https://github.com/logrotate/logrotate) | Logrotate 是一个开源工具，用于处理日志轮转。根据 Kubernetes 的配置方式，它被部署来管理 Kubernetes 的日志轮转。 |
| *Kubernetes in Action* 第二版 | [`mng.bz/oa1p`](https://mng.bz/oa1p) | Manning 的 Kubernetes 官方指南。 |
| Kubernetes 密钥 | [`mng.bz/nYW2`](https://mng.bz/nYW2) | Kubernetes 与 pods 和容器凭证共享的方法。 |
| 开放容器 | [`opencontainers.org/`](https://opencontainers.org/) | Docker 和其他几个容器开发组织支持的标准，用于标准化 Kubernetes 与容器交互的方式。 |
| Docker 的替代容器 | [`containerd.io/`](https://containerd.io/) [`cri-o.io/`](https://cri-o.io/) | 几个替代的容器化开发倡议，包括来自英特尔、IBM、红帽和其他 Linux 供应商（如 SUSE）等组织的贡献。 |
| 部署工具 | [`rancher.com`](https://rancher.com) [`helm.sh`](https://helm.sh) | 将 pods 和配置部署到 Kubernetes 可能很复杂。已经开发出几种解决方案，如 Helm 和 Rancher，代表了这一领域的领先解决方案。 |

## E.7 Elasticsearch

Elasticsearch 是日志聚合的常见目标之一。Elasticsearch 插件不是 Fluentd 开源部署的标准插件集的一部分（尽管它是预构建 Treasure Data 代理版本的一部分）。对于纯 Fluentd，需要安装代理。

| 名称 | URL | 描述 |
| --- | --- | --- |
| Elasticsearch 堆栈 | [www.elastic.co/elastic-stack](https://www.elastic.co/elastic-stack) | ELK 堆栈（Elasticsearch、Logstash、Kibana）的详细信息。虽然 Logstash 可以被视为 Fluentd 的竞争对手，但 Fluentd 通常与 Elasticsearch 和 Kibana 一起使用。 |
| Elasticvue | [`elasticvue.com`](https://elasticvue.com) | 用于查看 Elasticsearch 内容的 UI 工具。 |
| Fluentd Elasticsearch 插件 | [`docs.fluentd.org/output/elasticsearch`](https://docs.fluentd.org/output/elasticsearch) | 允许您集成 Elasticsearch 的插件。 |
| Elasticsearch 插件源码 | [`mng.bz/von4`](https://mng.bz/von4) | Fluentd Elasticsearch 插件的仓库。 |
| *Manning, Elasticsearch in Action* | [www.manning.com/books/ elasticsearch-in-action](https://www.manning.com/books/elasticsearch-in-action) | 虽然 *Elasticsearch in Action* 超出了我们所需了解的缓存知识，但这份 Redis 指南很有帮助。 |

## E.8 Redis

Redis 可以被 Fluentd 用来提供内存缓存解决方案。以下资源将提供更多详细信息、下载等。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| Redis 的家园 | [`redis.io/`](https://redis.io/) | 提供 Redis 的基本文档和下载。除了商业产品外，我们还将使用 Redis 进行自定义插件开发。 |
| Manning, *Redis in Action* | [www.manning.com/books/redis-in-action](https://www.manning.com/books/redis-in-action) | 虽然 *Redis in Action* 超出了我们所需了解的缓存知识，但这份 Redis 指南很有帮助。 |
| Redis RubyGem | [`github.com/redis/redis-rb`](https://github.com/redis/redis-rb) | 这是连接和使用 Redis 操作的 Ruby 库。我们将在构建自定义插件时使用它。 |

## E.9 SSL/TLS 和安全

在分布式操作中使用 SSL/TLS 的能力是必不可少的。以下是一些关于使用 SSL/TLS 的有用链接。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| 证书颁发机构 | [`jamielinux.com/docs/ openssl-certificate-authority/`](https://jamielinux.com/docs/openssl-certificate-authority/) | 了解如何建立自己的证书颁发机构。 |
| TLS 简介 | [www.internetsociety.org/ deploy360/tls/basics/](https://www.internetsociety.org/deploy360/tls/basics/) | TLS 简介。 |
| Let’s Encrypt | [`letsencrypt.org/`](https://letsencrypt.org/) | Let’s Encrypt 是由 Linux 基金会开发的一项服务，提供有效期短的免费证书。该服务包括一些自动化功能，用于自动化证书续签。 |
| OWASP TLS 技巧表 | [`cheatsheetseries.owasp.org/ cheatsheets/ Transport_Layer_Protection _Cheat_Sheet.html`](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html) | 提供了来自开放网络应用安全项目的关于 TLS 及其工作原理的实用、有帮助的信息。 |
| 自签名证书 | [`dzone.com/articles/ creating-self-signed-certificate`](https://dzone.com/articles/creating-self-signed-certificate) | 解释了如何创建自签名证书。 |
| SSL 和 TLS | [www.hostingadvice.com/how-to/tls-vs-ssl/](https://www.hostingadvice.com/how-to/tls-vs-ssl/) | 该网站解释了 SSL 和 TLS 之间的区别。 |
| 保险库 | [www.vaultproject.io](https://www.vaultproject.io) | Vault 是一个用于管理秘密的工具，包括用户名和密码等详细信息。它包括从主节点安全分发信息的方法。 |
| OpenSSL | [www.openssl.org](https://www.openssl.org) | 开源的全功能工具包，用于传输层安全（TLS）和安全的套接字层（SSL）协议。被许多产品采用，包括 Fluentd。 |

## E.10 环境设置

以下资源是设置 Fluentd 环境的有用额外信息来源。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| Browserling 工具 | [www.browserling.com/tools](https://www.browserling.com/tools) | 一套在线工具，帮助进行格式转换等。 |
| Chocolatey | [`chocolatey.org/`](https://chocolatey.org/) | Chocolatey 是 Windows 的包管理器。它提供了一个类似于 Linux 的包管理器用户体验，如 rpm、yum 等。 |
| Clang | [`clang.llvm.org/`](https://clang.llvm.org/) | 支持编译平台原生元素的库。 |
| GCC | [`gcc.gnu.org/`](https://gcc.gnu.org/) | GCC 编译工具，如果你需要为源代码构建操作系统原生功能，则需要这些工具。 |
| GraalVM | [www.graalvm.org/](https://www.graalvm.org/) | 支持 Java 和其他语言的下一代多语言虚拟机。它能够创建平台原生二进制文件。 |
| Linux 包管理器摘要 | [`mng.bz/4jDj`](https://mng.bz/4jDj) | 提供有关不同包管理器和根据您的包管理器使用相应命令的信息。 |
| NTP（网络时间协议） | [`doc.ntp.org/`](https://doc.ntp.org/) | NTP 定义的官方网站；包括有关该主题的软件和白皮书。 |
| Quarkus | [`quarkus.io/`](https://quarkus.io/) | 使用 OpenJDK 的基于 Java 的堆栈，能够创建 Kubernetes 原生解决方案。 |
| 语义化版本控制 | [`semver.org/`](https://semver.org/) | 艺术品的版本标准形式。 |

## E.11 日志框架

以下不是所有日志框架的详尽总结，因为维护此类列表本身就是一项全职工作。此列表确实提供了涵盖，包括链接到语言原生日志框架和我们认为活跃的开源框架。

| 名称 | 语言 | 网址 | 描述 |
| --- | --- | --- | --- |
| N/A | PHP | [`www.php-fig.org/psr/psr-3/`](https://www.php-fig.org/psr/psr-3/) | PHP 社区已经开发了各种标准（PSR），包括日志框架（PSR-3）。PSR 定义了日志的接口标准，因此任何支持此标准的框架都将兼容。从 Zend 到 Drupal，广泛的 PHP 框架支持此规范。例如，Magneto 这样的框架提供了符合 PSR 的日志，支持与 Fluentd 的通信。 |
| Django | Python | [`docs.djangoproject.com/en/3.1/`](https://docs.djangoproject.com/en/3.1/) | Django 是一个利用原生日志并提供一些附加功能的 Web 应用程序框架。这意味着 Fluentd 扩展可以被集成。 |
| 语言原生 | Java | [`mng.bz/QWPv`](https://mng.bz/QWPv) | 这是 Java 语言的一部分；然而，它并不是最终的日志解决方案。许多人仍然更喜欢使用其他开源解决方案。 |
| 语言原生 | C#—语言原生库 | [`mng.bz/XWNa`](https://mng.bz/XWNa) | 这是微软提供的 C# 原生框架。它支持 JSON 配置文件控制和从配置中驱动的日志记录器注入。 |
| 语言原生 | VB.Net—语言原生库 | [`mng.bz/y4Qd`](https://mng.bz/y4Qd) | 这涵盖了 Microsoft 为 Visual Basic 在 .Net 中提供的日志记录功能。 |
| 语言原生 | Ruby—语言原生库 | [`docs.ruby-lang.org/en/2.4.0/Logger.html`](https://docs.ruby-lang.org/en/2.4.0/Logger.html) | Ruby 提供了一个具有日志级别的原生日志类。输出选项有限。语言基础可以扩展以支持更多目的地。 |
| 语言原生 | Python—语言原生库 | [`docs.python.org/3/library/logging.html`](https://docs.python.org/3/library/logging.html) | Python 的原生语言特性是几个扩展功能的日志框架的基础，包括一些 Python 框架，它们通过添加追加器增加了价值。这包括 Fluentd Python 库。 |
| 语言原生 | Go—原生日志 | [`golang.org/pkg/log/`](https://golang.org/pkg/log/) | 这是 Go 的原生日志包。与许多其他日志机制相比，这相当简单，最好将其视为一组辅助方法，用于以更结构化的方式将日志事件发送到 stderr。 |
| lgr | R | [`mng.bz/M2BB`](https://mng.bz/M2BB) | 这是一个为 R 语言设计的日志框架，其设计原则来源于 Log4J。提供的追加器数量较少，专注于数据库（考虑到这是一个以数据分析为重点的语言，这是可以理解的）。 |
| Log4cplus | C++ | [`sourceforge.net/p/log4cplus/wiki/Home/`](https://sourceforge.net/p/log4cplus/wiki/Home/) | 基于 Log4J 框架。 |
| Log4Cxx | C++ | [`logging.apache.org/log4cxx/latest_stable/`](https://logging.apache.org/log4cxx/latest_stable/) | 作为 Apache Log4J 家族的一部分，Log4Cxx 是 Log4J 的端口。 |
| Log4J2 | Java | [`logging.apache.org/log4j/2.x/index.html`](https://logging.apache.org/log4j/2.x/index.html) | Apache 还为 Kotlin 和 Scala 提供了几个 JVM 相关的语言封装。 |
| Log4Net | C# & VB 和其他在 .Net 框架上支持的语言 | [`logging.apache.org/log4net/`](https://logging.apache.org/log4net/) | Log4Net 是 Apache 基金会移植的 Log4J 框架，该基金会开发了原始框架。 |
| Logrus | Go | [`github.com/Sirupsen/logrus`](https://github.com/Sirupsen/logrus) | Logrus 目前处于仅维护状态，因为开发者认为它已经达到了可扩展性的极限，同时没有破坏兼容性。然而，它因其常用性而被引用。 |
| Monolog | PHP | [`github.com/Seldaek/monolog`](https://github.com/Seldaek/monolog) | Monolog 只是一个支持 PSR-3 规范的日志框架。它包括一个用于 Fluentd 的格式化器，可以与用于套接字级别通信的处理程序结合使用。 |
| NLog | C# & VB 和其他在 .Net 框架上支持的语言 | [`nlog-project.org/`](https://nlog-project.org/) | 框架扩展中包含的 HTTP 追加器。 |
| Pino | Node JS | [`getpino.io/`](https://getpino.io/) | Pino 适合集成到各种 Node.JS 框架中，例如 Express。它使用传输的概念将日志发送到其他系统，包括一些本机云服务提供商的解决方案。它支持套接字和 HTTP 端点，因此可以使用这些来与 Fluentd 进行通信。 |
| Serilog | C# & VB 和其他在 .Net 框架上支持的语言 | [`serilog.net/`](https://serilog.net/) | Serilog 是一个开源框架，它促进了更强的结构化日志记录。它提供了广泛的输出目的地，包括 Fluentd。 |
| SLF4J | Java | [`www.slf4j.org/`](https://www.slf4j.org/) | Simple Logging Facade for Java (SLF4J) 不是一个框架，而是一个抽象层，使得不同的日志框架可以使用相同的底层基础；例如，Logback 和 Log4J2。 |
| Twisted | Python | [`twistedmatrix.com/trac/`](https://twistedmatrix.com/trac/) | 这又是一个具有事件驱动模型的 Python 框架。它提供了一种集成到更广泛生态系统的日志机制。从那里，Twisted 可以用来将日志事件发送到一组本机发布者，或者通过 Python 的原生日志功能发送日志事件。 |
| Winston JS | Node JS | [`github.com/winstonjs/winston`](https://github.com/winstonjs/winston) | Winston 具有许多日志框架的特征，具有日志级别和追加器（在 Winston 中称为传输）等功能。与一些其他框架相比，传输的范围有限，但提供了一个构建自己框架的框架。 |

## E.12 立法信息门户

一旦我们记录敏感信息，例如可识别的个人、信用卡以及许多其他事物，我们的日志和日志存储就可能受到一系列法律法规的约束。以下是我们过去查看的一些资源，以帮助我们进一步理解。

| 常见名称 | URL | 描述 |
| --- | --- | --- |
| DLA Piper 数据保护 | [www.dlapiperdataprotection.com](https://www.dlapiperdataprotection.com) | DLA Piper 是一家全球律师事务所，它开发和维护了一个网站，提供了关于各国在数据保护方面定位的良好见解。 |
| GDPR |

+   [`gdpr-info.eu/`](https://gdpr-info.eu/)

+   [`ico.org.uk/for-organisations/guide-to-data-protection/`](https://ico.org.uk/for-organisations/guide-to-data-protection/)

+   [`oag.ca.gov/privacy/ccpa`](https://oag.ca.gov/privacy/ccpa)

| 欧洲联盟（EU）制定的通用数据保护条例（GDPR）旨在加强个人数据保护。所有欧盟国家都已批准了这项立法，以及一些基于其构建的国家立法。许多国家和美国各州都制定了他们自己的衍生立法；例如，加利福尼亚州的《加州消费者隐私法案》（CCPA）。 |
| --- |
| 健康保险可携带性和问责制法案（HIPAA） | [www.hhs.gov/hipaa/index.html](https://www.hhs.gov/hipaa/index.html) | 健康保险可携带性和问责制法案（HIPAA）涵盖了与医疗保健相关的信息细节。 |
| ISO/IEC 27001 | [www.iso.org/isoiec-27001-information-security.html](https://www.iso.org/isoiec-27001-information-security.html) | 许多组织寻求符合 ISO/IEC 27001；虽然不是由立法驱动的规则集，但它是一套最佳实践标准。 |
| PCI DSS（支付卡行业数据安全标准） | [www.pcisecuritystandards.org/](https://www.pcisecuritystandards.org/) | 所有支付卡运营商采用的标准，定义了从基础设施到开发实践的一系列要求。 |
| 萨班斯-奥克斯利法案（SOX） | [www.govinfo.gov/content/pkg/STATUTE-116/pdf/STATUTE-116-Pg745.pdf](https://www.govinfo.gov/content/pkg/STATUTE-116/pdf/STATUTE-116-Pg745.pdf) | 萨班斯-奥克斯利法案（SOX）是在美国制定的，旨在解决公司法律报告问题。但作为副产品，它确立了包括数据处理在内的安全实践。还有其他国家的变体，如 J-SOX（日本）、C-SOX（加拿大）和 TC-SOX（土耳其）。 |
| 联合国贸易和发展会议（UNCTAD） | [`mng.bz/aD1m`](https://mng.bz/aD1m) | UNCTAD 提供与 DLA Piper 类似资源，但专注于电子商务方面的考虑。 |

## E.13 其他有用的信息来源

| 名称 | URL | 描述 |
| --- | --- | --- |
| 转换不同的时间表示 | [www.epochconverter.com/](https://www.epochconverter.com/) | 将时间戳转换为 Linux/Unix 系统和 Java 等语言使用的秒或毫秒纪元表示形式。 |
| 记录错误代码的示例 |

+   HTTP 状态码：[`datatracker.ietf.org/ doc/html/rfc7231#section-6.1`](https://datatracker.ietf.org/doc/html/rfc7231#section-6.1)

+   电子邮件状态码：[`www.rfc-editor.org/ rfc/rfc5248.html`](https://www.rfc-editor.org/rfc/rfc5248.html)

+   WebLogic 服务器：[`docs.oracle.com/ cd/E24329_01/doc.1211/ e26117/chapter_bea_messages.htm#sthref7`](https://docs.oracle.com/cd/E24329_01/doc.1211/e26117/chapter_bea_messages.htm#sthref7)

| 优秀的错误代码文档示例。涵盖 IETF 和 WebLogic 应用程序服务器的 HTTP 和电子邮件。 |
| --- |
| ISO 8601 日期时间标准 | [www.w3.org/TR/NOTE-datetime](https://www.w3.org/TR/NOTE-datetime) | 这描述了定义日期和时间的不同行业标准方式。 |
| ITIL（信息技术基础设施库） | [www.axelos.com/ best-practice-solutions/itil](https://www.axelos.com/best-practice-solutions/itil) | 一套行业标准推荐实践和流程，如 ITSM（IT 服务管理）。 |
| 支付卡行业（PCI）安全标准委员会（SSC） | [`www.pcisecuritystandards.org/`](https://www.pcisecuritystandards.org/) | 定义 PCI DSS（数据安全标准）的组织——处理支付卡数据的网络安全标准。 |
| 正则表达式开发 | [www.regular-expressions.info/](https://www.regular-expressions.info/) | 该网站详细介绍了正则表达式的使用。 |
| N 层架构 | [`stackify.com/ n-tier-architecture/`](https://stackify.com/n-tier-architecture/)[`livebook.manning.com/book/the-cloud-at-your-service/chapter-6/point-12033-14-14-1`](https://livebook.manning.com/book/the-cloud-at-your-service/chapter-6/point-12033-14-14-1) | 提供了 N 层架构及其价值主张的解释。 |
| TCP 和 UDP 网络协议 | [www.vpnmentor.com/blog/tcp-vs-udp/](https://www.vpnmentor.com/blog/tcp-vs-udp/)[www.cs.dartmouth.edu/ ~campbell/cs60/socketprogramming.html](https://www.cs.dartmouth.edu/~campbell/cs60/socketprogramming.html) | TCP 和 UDP 及其差异的解释。 |

## E.14 支持 Fluentd 资源

以下表格提供了 Fluentd 社区提供的与将日志事件发送到 Fluentd 相关的额外资源的链接。

| 名称 | URL | 描述 |
| --- | --- | --- |
| Fluentd 提供的日志库 | [`github.com/fluent/fluent-logger-java`](https://github.com/fluent/fluent-logger-java) | Fluentd 为 Java 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-ruby`](https://github.com/fluent/fluent-logger-ruby) | Fluentd 为 Ruby 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-python`](https://github.com/fluent/fluent-logger-python) | Fluentd 为 Python 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-perl`](https://github.com/fluent/fluent-logger-perl) | Fluentd 为 Perl 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-php`](https://github.com/fluent/fluent-logger-php) | Fluentd 为 PHP 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-node`](https://github.com/fluent/fluent-logger-node) | Fluentd 为 NodeJS 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-scala`](https://github.com/fluent/fluent-logger-scala) | Fluentd 为 Scala 直接日志记录提供了一个库。 |
|  | [`github.com/fluent/fluent-logger-golang`](https://github.com/fluent/fluent-logger-golang) | Fluentd 提供了一个从 Go 直接进行日志记录的库。 |
|  | [`github.com/fluent/fluent-logger-erlang`](https://github.com/fluent/fluent-logger-erlang) | Fluentd 提供了一个从 Erlang 直接进行日志记录的库。 |
|  | [`github.com/fluent/fluent-logger-ocaml`](https://github.com/fluent/fluent-logger-ocaml) | Fluentd 提供了一个从 OCaml 直接进行日志记录的库。 |
| msgpack | [`msgpack.org/`](https://msgpack.org/) | 这是一个在多种语言中实现的压缩库，Fluentd 可以利用它。在转发插件发送或接收事件时使用。 |

## E.15 相关阅读

Fluentd，正如你所观察到的，在其潜在应用中跨越了许多边界。下表反映了这一点。列出的书籍可能有助于你扩展和利用那些相关服务。

| 名称 | 网址 | 描述 |
| --- | --- | --- |
| *Core Kubernetes* | [`www.manning.com/books/core-kubernetes`](https://www.manning.com/books/core-kubernetes) | Fluentd 通常在 Kubernetes 的上下文中使用，并且只是 Kubernetes 设置的一个方面。这本书和*Kubernetes in Action*将涵盖你需要的大部分内容。 |
| *Design Patterns* | [`refactoring.guru/design-patterns/catalog`](https://refactoring.guru/design-patterns/catalog) | 这是首先由四人帮（GoF）描述的核心模式及其书籍《设计模式：可重用面向对象软件元素》的详细信息。四人帮是 Erich Gamma、John Vlissides、Richard Helm 和 Ralph Johnson。我们为每个模式提供了一个简要指南的链接。 |
| *Docker in Action* | [`mng.bz/g4Bv`](https://mng.bz/g4Bv) | Docker 是实现容器的典型技术。当我们不需要 Kubernetes 的复杂性时，我们会更直接地使用 Docker。这本书涵盖了构建和运行容器的核心。 |
| Effective Unit Testing | [www.manning.com/books/effective-unit-testing](https://www.manning.com/books/effective-unit-testing) | 在第九章中，当我们实现自定义插件时，我们探讨了单元测试。理想情况下，为生产使用构建的单元测试应该是广泛的。虽然这本书专注于 Java，但它将提供最佳实践的见解。 |
| *Elasticsearch in Action* | [www.manning.com/books/elasticsearch-in-action](https://www.manning.com/books/elasticsearch-in-action) | 当与 Elasticsearch 和 Fluentd 一起工作时，这本书可能会很有用。 |
| *Groovy in Action*, 第二版 | [`mng.bz/en1V`](https://mng.bz/en1V) | 我们使用的日志模拟器是用 Groovy 构建的，这使得它非常容易添加增强功能，以有效地模拟不同的来源。如果你对深入了解 Groovy 感兴趣，这本书将有所帮助。 |
| *Kubernetes in Action* | [`mng.bz/p2PK`](https://mng.bz/p2PK) | 本书涵盖了 Kubernetes 及其工作方式的大量细节，为容器编排提供了更多见解。 |
| *MongoDB in Action*, Second Edition | [`mng.bz/OGxw`](https://mng.bz/OGxw) | 我们使用 MongoDB 作为输出目标。这将提供有关 MongoDB 可能需要的所有信息。 |
| *Operations Anti-Patterns, DevOps Solutions* | [`mng.bz/Yg1z`](https://mng.bz/Yg1z) | 获得您的操作流程和日志是实现 DevOps 工作方式的一个关键部分。本书探讨了您可能最终面临的潜在陷阱（或反模式）。 |
| *Redis in Action* | [www.manning.com/books/redis-in-action](https://www.manning.com/books/redis-in-action) | 我们在构建自定义插件的示例中使用了 Redis。这本书将更深入地介绍 Redis。 |
| *Ruby in Practice* | [www.manning.com/books/ruby-in-practice](https://www.manning.com/books/ruby-in-practice) | 提供更多关于使用 Ruby 的实际指导。 |
| *Software Telemetry* | [www.manning.com/books/software-telemetry](https://www.manning.com/books/software-telemetry) | *Software Telemetry* 讲述了获取指标、日志和某些类型的业务应用状态数据，并使用这些数据为您提供健康视角的想法。Fluentd 可以是遥测解决方案的关键部分。 |
| *The Well-Grounded Rubyist*, Third Edition | [`mng.bz/GGyD`](https://mng.bz/GGyD) | 如果您正在考虑深入了解 Fluentd 或开发自己的自定义插件，那么这是一本很好的读物。 |
| Securing DevOps | [www.manning.com/books/securing-devops](https://www.manning.com/books/securing-devops) | 探讨了保护云环境的需求和技术。 |
| YAML | [`yaml.org/`](https://yaml.org/) | YAML 的官方站点包含了其语法的详细信息。 |
