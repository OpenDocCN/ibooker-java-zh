# 附录 C. 插件总结

我们在书中探讨了多种插件类型，但并未涵盖所有开箱即用的插件。本附录将对此进行说明。我们提供了一份所有核心插件和一些值得关注的开源插件的总结。

## C.1 格式化器插件

| 插件名称 | 概述 | Fluentd 核心 |
| --- | --- | --- |
| `csv` | 在第四章中有介绍。基本的逗号分隔值，尽管可以通过配置属性更改分隔符。 | 是 |
| `hash` | 这将日志事件记录转换为 Ruby 可以处理的哈希格式表示。可以在某些插件配置中嵌入 Ruby 代码片段，因此它被包含在内。 | 是 |
| `json` | 在第四章中有介绍。允许我们将日志事件内容以 JSON 格式输出。有关 JSON 格式的更多信息，请参阅 [www.json.org](https://www.json.org)。 | 是 |
| `ltsv` | 在第四章中有介绍。标签制表符（字符）分隔值——而不是依赖于列表中的位置来获取正确的值含义，我们可以包含一个标签。更多信息请参阅 [`ltsv.org/`](http://ltsv.org/)。 | 是 |
| `msgpack` | 在第四章中有介绍。一个理想的格式化器，用于压缩日志事件以在 Fluentd 和 Fluent Bit 之间通信。有关 msgpack 的更多信息，请参阅 [`msgpack.org/`](https://msgpack.org/)。 | 是 |
| `out_file` | 在第四章中有介绍。这个格式化器会打印出命名的日志事件属性，可以使用分隔符进行列表。 | 是 |
| `single_value` | `single_value` 格式化器有点类似于 CSV 格式化器，因为它可以从日志事件记录中选择内容并输出。然而，在这种情况下，只能通过使用 `message_key` 属性来识别日志事件记录的一部分。 | 是 |

## C.2 提取和注入插件支持

开箱即用的来源和匹配插件支持提取和注入如下：

| 来源 | 注入 | 提取 |
| --- | --- | --- |
| `dummy` | 否 | 否 |
| `exec` | 否 | 是 |
| `forward` | 否 | 否 |
| `http` | 否 | 否 |
| `monitor_agent` | 否 | 否 |
| `syslog` | 否 | 否 |
| `tail` | 否 | 否 |
| `tcp` | 否 | 是 |
| `udp` | 否 | 是 |
| `unix` | 否 | 否 |
| `windows_eventlog` | 否 | 否 |
| 匹配 | 注入 | 提取 |
| `copy` | 是 | 否 |
| `exec` | 否 | 否 |
| `exec_filter` | 是 | 是 |
| `file` | 是 | 否 |
| `forward` | 否 | 否 |
| `http` | 否 | 否 |
| `null` | 否 | 否 |
| `relabel` | 否 | 否 |
| `round_robin` | 否 | 否 |
| `secondary_file` | 否 | 否 |
| `stdout` | 是 | 否 |

## C.3 过滤器插件

| 插件名称 | 概述 | Fluentd 核心 |
| --- | --- | --- |
| `add` | 很有趣，因为它提供了一种无需努力即可添加 UUID（通用唯一标识符）和其他附加名称值对的方法。唯一标识符在日志事件通过多个 Fluentd 节点时非常有用。更多信息可以在 [`github.com/yu-yamada/fluent-plugin-add`](https://github.com/yu-yamada/fluent-plugin-add) 找到。 | 否 |
| `anonymizer` | 匿名化器可以配置为使用所选算法对日志事件中元素的 内容进行单向哈希。这对于屏蔽敏感数据非常理想。更多信息可以在 [`github.com/y-ken/fluent-plugin-anonymizer`](https://github.com/y-ken/fluent-plugin-anonymizer) 找到。 | N |
| `autotype` | 根据分析有效载荷对日志事件属性应用类型。对于处理数值，因为它消除了手动类型转换的需要，所以非常适合。 | N |
| `filter_parser` | 结合了解析插件的功能和过滤器。 | Y |
| `fluent-plugin-fields-autotype` | 此插件非常适合解析日志事件结构并选择正确的数据类型来评估属性。这也是 `fluent-plugin-auto-typ` 的一个变体。更多信息可以在 [`mng.bz/AxlK.`](http://mng.bz/AxlK) 找到。 | N |

| `geoip` | 此插件利用了互联网服务提供商发布它们分配的互联网协议（IP）地址以及这些 IP 连接到的物理位置的事实。这些信息随后由几个组织汇总。通过知道请求的 IP，可以查找位置。这很有用，因为它允许数据更有效地路由和过滤。例如，您可能有一个全球的 Fluentd 服务器网络。使用 GeoIP 将使我们能够将日志定向到最近的 Fluentd 服务器以聚合日志事件。这在处理分布式用例中的大量日志时非常有帮助，例如

+   物联网（IoT）部署

+   全球多区域和多云解决方案

更多信息可以在 [`github.com/y-ken/fluent-plugin-geoip`](https://github.com/y-ken/fluent-plugin-geoip) 找到。 | N |

| `grep` | 提供了定义关于日志事件属性规则的手段，以将它们从事件流中过滤出来。可以指定多个表达式以创建累积规则。 | Y |
| --- | --- | --- |
| `record_modifier` | `record_transformer` 的一个变体已被修改以提高插件效率：[`mng.bz/ZzoO`](http://mng.bz/ZzoO)。 | N |
| `record_transformer` | 最复杂的内置过滤器，提供了一组用于操作日志事件的选项。 | Y |
| `stdout` | 将所有事件发送到 `stdout`，而不从流中删除事件。 | Y |

## C.4 标签操作插件

| 插件名称 | 概述 | Fluentd 核心库 |
| --- | --- | --- |
| `rewrite`[`mng.bz/2jad`](http://mng.bz/2jad) | 这使得可以使用一个或多个规则修改标签，例如如果日志事件记录的属性与正则表达式匹配。因此，根据日志事件执行特定任务变得非常容易。 | N |
| `rewrite-tag-filter`[`mng.bz/1jMV`](http://mng.bz/1jMV) | 匹配指令中有一个或多个规则时，将对日志事件应用正则表达式。然后，根据结果，将标签更改为指定的值。规则可以设置为选择是否将重写应用于正则表达式的真或假结果。如果成功，则使用新标签重新发出日志事件以继续匹配事件之后。 | N |
| `route`[`mng.bz/PWx9`](http://mng.bz/PWx9) | 路由插件允许标签将日志事件定向到一个或多个操作，例如操作日志事件并复制它以通过另一个指令拦截。 | N |

## C.5 防止警报风暴

如果生成连续的错误流，一些资源可以帮助控制或防止可能的警报风暴。

| 插件名称 | 摘要 |
| --- | --- |
| *日志抑制过滤器*[`mng.bz/J1l0`](http://mng.bz/J1l0) | 如果在特定时间段内重复出现超过定义次数，Fluentd 插件将基于命名的日志事件属性保留之前日志条目的列表。 |
| *Log4J2 插件*[`mng.bz/wnPq`](http://mng.bz/wnPq) | 在 Log4J2 框架内抑制日志事件，以防止发出过多匹配一组规则的日志事件。 |
| *Log4Net*[`mng.bz/7W19`](http://mng.bz/7W19) | Log4J2 解决方案的 .Net 变体。 |

## C.6 分析和指标插件

一些插件可以帮助在 Fluentd 中创建分析或指标值，或使用工具帮助生成指标。

| 插件名称 | 摘要 |
| --- | --- |

| *Fluent-plugin-prometheus*[`mng.bz/mxJr`](http://mng.bz/mxJr) | 将六个插件捆绑在一起，包括以下内容：

+   `prometheus` 输入插件提供了一个指标 HTTP 端点，由 Prometheus 服务器在 24231/tcp（默认）上抓取。

+   `prometheus_input` 插件收集 Fluentd 中的内部指标，有点像监控代理。

+   `prometheus_output_monitor` 插件收集 Fluentd 中输出插件的内部指标。

+   `prometheus_tail_monitor` 插件收集 Fluentd 中尾插件的内部指标。这使我们能够确保尾插件按预期运行。

+   输出和过滤器插件使用额外的指标来装备日志事件，例如基于配置属性的值出现的次数。

|

| *Fluentd Elasticsearch 插件*[`mng.bz/5K1B`](http://mng.bz/5K1B) | 将日志事件发送到 Elasticsearch 的插件。该插件提供了一套非常丰富的配置属性。 |
| --- | --- |

| *Fluentd 数据计数器*[`mng.bz/6Z1o`](http://mng.bz/6Z1o) | 计算在指定属性中匹配任何指定正则表达式模式的日志事件：

+   每分钟/每小时/每天的计数

+   每秒计数（平均每分钟/每小时/每天）

+   每种模式在消息总数中的百分比

`DataCounterOutput` 发出包含结果数据的消息，因此您可以将这些消息（默认情况下带有 `datacount` 标签）输出到您想要的任何输出。|

| *Fluentd 数字计数器*[`mng.bz/oaJd`](http://mng.bz/oaJd) | 插件用于计算匹配数字范围模式的日志事件，并输出其结果（类似于`fluent-plugin-datacounter`）：

+   每分钟/每小时/每天的计数

+   每秒计数（平均每分钟/每小时/每天）

+   每个数字模式在日志事件总数中的百分比

|

## C.7 插件接口

以下表格总结了在开发自己的插件时可以或应该实现的功能。您可以通过查看 Fluentd 代码以及 https://github.com/fluent/fluentd/blob/master/lib/fluent/plugin 来查看详细信息，但我们希望这些表格将使生活更加容易。

输入

| 函数原型 | 描述 |
| --- | --- |
| `def emit` | 这为我们的需求重载了 Fluent::Input 类的方法。 |
| `def run` | 设置循环线程，如果没有要输出的内容，则进入睡眠状态。 |
| `def configure(conf)` | 此方法用于处理配置值，以便它们可以被验证，特别是如果值可能冲突。 |
| `def start` | 一个生命周期事件，用于开始创建连接。启动计时器以推动逻辑查看是否需要任何 I/O。 |
| `def shutdown` | 开始释放资产的过程，例如释放到资源的连接。 |

输出

| 函数原型 | 描述 |
| --- | --- |
| `commit_write(chunk_id)` | 一旦一个块可以写入，此方法告诉我们的实现哪个块 ID 要写入。返回到 Fluentd 后，该块将被清除。 |
| `def configure(conf)` | 此方法用于处理配置值，以便它们可以被验证，特别是如果值可能冲突。 |
| `def format(tag, time, record)` | 将代码序列化以存储在块中。`tag`、`time`和`record`代表日志事件的三个部分，时间是从纪元以来的秒数。此阶段的记录是一个哈希表，允许 JSON 操作。如果没有重载，将抛出`NotImplemented`错误。 |
| `def formatted_to_msgpack_binary?` | 用于指示自定义格式方法（#format），返回 msgpack 到二进制或否。如果#format 返回 msgpack 二进制，则覆盖此方法以返回`true`。默认情况下，它返回`false`。 |
| `def multi_workers_ready?` | False |

| `def prefer_buffered_processing` | 仅当以下所有条件都为真时，覆盖此方法以返回`false`：

+   该插件提供了缓冲和非缓冲方法的实现。

+   如果没有指定`<buffer>`部分为`true`，则插件预期作为非缓冲插件工作。

|

| `def prefer_delayed_commit` | 覆盖此方法以决定使用`write`还是`try_write`，如果两者都实现了。 |
| --- | --- |
| `def shutdown` | 关闭操作是启动事件的相反。在这个阶段，我们应该释放资源，例如网络连接。 |
| `def start` | 这是重要的生命周期事件之一；我们应该在收到此事件时开始消费或允许发送事件。 |
| `def try_write(chunk)` | 此方法用于实现异步 I/O。如果未重载，基类将抛出 `NotImplemented` 错误，但预期需要实现。 |
| `def write(chunk)` | 此方法接收一个块并将其同步写入。如果未重载，基类将抛出 `NotImplemented` 错误，但预期可以工作。 |
| `extract_placeholders(str, chunk)` | 用于使用块信息从 `str` 中提取一个字符串值。返回一个字符串。 |
| `process(tag, es)` | 此方法用于同步输出。如果没有重载，将抛出 `NotImplemented` 错误。`tag` 是应用于 `es` 结构中所有事件的标签。`es` 参数表示一个或多个要写入的日志事件。 |
| `rollback_write(chunk_id)` | 插件可以使用此方法控制块写入的回滚和重试。 |
