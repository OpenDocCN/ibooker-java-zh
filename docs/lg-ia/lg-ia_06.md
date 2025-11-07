# 4 使用 Fluentd 输出日志事件

本章涵盖

+   使用文件、MongoDB 和 Slack 的输出插件

+   使用 Fluentd 应用不同的缓冲选项

+   查看缓冲的优势

+   处理缓冲过载和其他缓冲风险

+   添加格式化器以结构化日志事件

第三章演示了如何捕获日志事件以及辅助插件（如解析器）如何发挥作用。但是，如果我们不能对数据进行有意义的操作，例如将事件交付到格式化后的端点以便使用——例如，将事件存储在日志分析引擎中或将消息发送到操作（Ops）团队进行调查，那么捕获数据就只有价值。本章将展示 Fluentd 如何使我们能够做到这一点。我们将探讨 Fluentd 输出插件如何从文件中使用，以及 Fluentd 如何与 MongoDB 和 Slack 的协作/社交工具配合进行快速通知。 

本章将继续使用 LogSimulator，我们还将使用一些其他工具，例如 MongoDB 和 Slack。与之前一样，完整的配置可在 Manning 的下载包中找到，或通过 GitHub 仓库获取，这样我们就可以专注于相关插件（s）的配置。MongoDB 和 Slack 的安装步骤在附录 A 中介绍。

## 4.1 文件输出插件

与 *tail*（文件输入）插件相比，我们不太可能使用文件输出插件，因为我们通常希望输出到允许我们查询、分析和可视化事件的工具。当然，对于生产环境中确实需要文件输出的情况，这将是最佳选择之一。然而，它是一个很好的起点，因为我们可以轻松地看到各种插件（如解析器和过滤器）的结果和影响。将重要事件记录到文件也便于在必要时轻松存档日志事件以供将来参考（例如，审计日志事件以支持法律要求）。因此，在继续探讨更复杂的输出之前，我们将查看文件输出。

在文件输出（以及由此扩展的任何涉及直接或间接写入物理存储的输出）中，我们需要考虑几个因素：

+   我们可以在文件系统中写入哪里，这取决于存储容量和权限？

+   那个位置是否有足够的容量（分配的容量和物理容量）？

+   物理硬件可以提供多少 I/O 吞吐量？

+   数据访问是否存在延迟（NAS 和 SAN 设备通过网络访问）？

虽然基础设施性能不太可能影响开发工作，但在预生产（例如，性能测试环境）和生产环境中却极为重要。值得注意的是，设备性能对于文件插件至关重要。其他输出插件可能正在使用包括优化 I/O（例如，数据库缓存、分配文件空间的优化）逻辑的服务。使用输出插件，我们可能已经整合了多个日志事件的来源。因此，我们可能会得到一个配置，其中 Fluentd 将所有输入写入一个文件或位置。可以通过缓冲区（正如我们很快将看到的）和缓存来减轻物理性能方面的考虑。

### 4.1.1 基本文件输出

让我们从 Fluentd 的一个相对基本的配置开始。在前几章的示例中，我们只看到了写入控制台的内容。现在，而不是控制台，我们应该简单地将所有内容推送到一个文件。为此，我们需要在配置中添加一个新的 `match` 指令，但我们将继续使用文件源来处理日志事件。

为了说明输出插件可以在配置中处理多个输入，我们除了包含前一章中展示的自监控源配置外，还包含了一个日志文件源。为了控制 Fluentd 的 `self_monitor` 生成的日志事件的频率，我们可以定义另一个属性，称为 `emit_interval`，它接受一个持续时间值——例如，10s（10 秒）。`emit_interval` 提供的值是 Fluentd 生成日志事件之间的时间。自监控可以包括诸如已处理的事件数量、管理的工人数等详细信息。

至少，文件输出插件只需要定义 `type` 属性，并使用 `path` 属性指定输出位置。在以下列表中，我们可以看到我们的 `Chapter4/Fluentd/rotating-file-read-file-out.conf` 文件的相关部分。这个配置的结果可能会让你感到惊讶，但让我们看看会发生什么。

列表 4.1 Chapter4/Fluentd/rotating-file-read-file-out.conf—match extract

```
<source>
  @type monitor_agent
  bind 0.0.0.0
  port 24220
  @id in_monitor_agent
  include_config true 
  tag self
  emit_interval 10s
</source>

<match *>
    @type file                             ❶
    path ./Chapter4/fluentd-file-output    ❷
</match>
```

❶ 将插件类型更改为文件

❷ 要写入的文件位置

使用此新的 `match` 指令的结果，如果使用以下命令运行 LogSimulator 和 Fluentd，就可以看到：

+   `fluentd -c ./Chapter4/Fluentd/rotating-file-read-file-out.conf`

+   `groovy LogSimulator.groovy ./Chapter4/SimulatorConfig/jul-log-output2.properties./TestData/medium-source.txt`

容易的假设是所有内容都写入一个名为`fluentd-file-output`的文件。然而，实际上发生的是使用路径的最后部分作为其名称创建了一个文件夹（即，`fluentd-file-output`），您将在该文件夹中看到两个文件。文件将以半随机名称出现（以区分不同的缓冲区文件），并且有一个具有相同基本名称的元数据文件。Fluentd 所做的是隐式地使用缓冲机制。在输出插件中使用默认选项的缓冲区并不罕见；有些插件放弃了缓冲区的使用——例如，*stdout*插件。

### 4.1.2 缓冲区的基本原理

缓冲区，如您可能从第一章回忆起来，是 Fluentd 的一个辅助插件。输出插件需要意识到它们可能对 I/O 性能产生的影响。因此，大多数输出插件都使用一个可以**同步**或**异步**行为的缓冲区插件。

同步方法意味着当日志事件收集到*块*中时，一旦块满了，它就被写入存储。异步方法利用一个额外的队列阶段。与输出通道交互的队列阶段在单独的线程中执行，因此块填充不应受到任何 I/O 性能因素的影响。

之前的例子没有明确定义缓冲区；我们看到输出插件应用了默认的文件缓冲区。当你意识到文件输出插件支持使用 gzip 压缩输出文件的能力时，这更有意义，压缩的内容越多，效果越好。

在图 4.1 中，我们已对步骤进行了编号。正如箭头的不同路径所示，生命周期中的步骤可以被绕过。所有日志事件都从步骤 1 开始，但如果未使用缓冲区，则过程立即移动到步骤 5，在那里发生物理 I/O 操作，然后我们继续到步骤 6。如果发生错误，步骤 6 可以将逻辑发送回前面的步骤再次尝试。这非常依赖于插件实现，但这是数据库插件等插件中常见的做法。如果重试失败或插件不支持该概念，一些插件支持指定二级插件的想法。

![图 4.1](img/CH04_F01_Wilkins.png)

图 4.1 日志事件通过输出生命周期（例如，斜体步骤仅在出现问题时使用）

二级插件是另一个可以被调用的输出插件（步骤 7）。通常，二级插件会尽可能简单，以最小化出现问题的可能性，以便稍后可以恢复日志事件。例如，假设输出插件从 Fluentd 节点调用远程服务（例如，在不同的网络、单独的服务器集群，甚至数据中心）。在这种情况下，二级插件可以是一个简单的文件输出到本地存储设备。

注意：我们始终建议实现一个具有最少软件和基础设施依赖的二级输出。需要降级到二级插件强烈表明存在更广泛的问题。因此，它越简单，对其他因素的依赖性越少，输出就越不可能被中断。文件很好地支持这种方法。

如果已配置缓冲，则步骤 1 到 3 将会被执行。但接下来的操作将取决于缓冲是否为异步。如果是同步的，则过程将跳转到步骤 5，我们将遵循相同的步骤。对于异步缓冲，日志块将进入一个单独的过程，该过程管理一个待写入的块队列。步骤 4 代表缓冲异步操作。随着块的填充，它们被放入队列结构中，等待输出机制将每个块输出内容。这意味着要处理的下一个日志事件不会被步骤 5 及以后的 I/O 操作所阻塞。

理解 gzip 压缩

Gzip 是 GNU 对 ZLIB 压缩格式的实现，该格式由 IETF RFC 的 1950、1951 和 6713 定义。Zip 文件使用一种称为 Lempel-Ziv 编码（LZ77）的算法来压缩内容。简单来说，该算法通过寻找字符的重复模式来工作；当找到重复时，该字符串会被替换为对前一次出现的引用。因此，识别为重复的字符串越大，引用就越有效，从而提供更多的压缩——文件越大，找到重复的可能性就越高。

默认情况下，Fluentd 提供以下缓冲类型：

+   *内存*

+   *文件*

如您所意识到的那样，路径被用作文件夹位置来存储其缓冲内容，并包含内容和元数据文件。使用文件 I/O 作为缓冲在存储设备方面不会带来很大的性能提升，除非您建立了一个 *RAM 磁盘*（也称为 *RAM 驱动器*）。基于文件的缓冲仍然提供一些好处；文件的使用方式被优化（保持文件打开等）。它还充当一个临时区域，在应用压缩之前积累内容（如前所述，参与 zip 压缩的数据越多，可能的压缩就越大）。此外，由于某种形式的进程或硬件故障，记录的内容不会丢失，并且当服务器和/或 Fluentd 重新启动时，缓冲可以重新恢复。

注意：RAM 驱动通过为存储分配一块内存，然后告诉操作系统的文件系统它是一个额外的存储设备。使用此存储的应用程序认为它们正在写入一个像磁盘这样的物理设备，但实际上内容是写入内存的。更多信息可以在 [www.techopedia.com/definition/2801/ram-disk](http://www.techopedia.com/definition/2801/ram-disk) 找到。

由于许多插件默认或明确地涉及缓冲区，我们应该看看如何开始配置缓冲区行为。我们知道当事件从缓冲区移动到输出目标时，此类动作发生的频率，以及这些配置如何影响性能。图 4.1 中所示的生命周期图提供了配置可能性的线索。

### 4.1.3 块和缓冲区控制

如图 4.1 所示，缓冲区的核心结构是块的概念。我们配置块的方式，除了同步和异步之外，还会影响性能。块可以通过分配存储空间（允许使用连续的内存或磁盘的一部分）或通过时间段来控制。例如，一个时间段内的所有事件都进入一个块，或者一个块将继续填充事件，直到达到特定的日志事件数量或块达到一定的大小。如果需要为 I/O 提供连接重试，这种分离 I/O 和块填充是有益的，例如在共享服务或网络远程服务（如数据库）的情况下。

在两种方法中，通过配置可以设置属性，使得日志事件不会因为阈值从未完全满足而滞留在缓冲区中。采用哪种方法将受你的日志事件源的行为以及性能与可用资源（例如，内存）之间的权衡以及可接受的延迟量（在日志事件向下移动时）的影响。

个人而言，我倾向于使用大小约束，这提供了可预测的系统行为；这可能反映了我 Java 背景和偏好，不希望过度调整虚拟机。

表 4.1 显示了缓冲区上大多数可能的控制。当控制可以有一些微妙的行为时，我们包括了更详细的说明。

表 4.1 缓冲区配置控制

| 属性 | 描述 |
| --- | --- |
| `timekey` | 这是指每个块将负责保持的秒数（默认为 1 天）。`log_event` 的 `time` 属性然后确定将事件添加到哪个块。例如，如果我们的 timekey 设置为 300（秒）且块从小时开始，那么当 10:00:01 标记的事件到达，并且每 30 秒到达更多事件时，第一个块将保留额外的 9 个事件。下一个块将保留 10:05:00 后开始到达的事件，因此下一个事件将是 10:05:01。 |
| `timekey` | 如果在 10:05 之前有额外的乱序事件到达（例如，带有时间戳 10:03:15 和 10:03:55），但它们直到 10:04:31 才到达，那么它们仍然会被添加到第一个块中。这种行为可以通过 `timekey_wait` 属性进一步修改。 |
| `timekey_wait` | 这是块存储周期结束后，块写入之前需要等待的秒数。默认为 60 秒。扩展我们的 `timekey` 示例，如果此值设置为 60s（60 秒），则该块将在 10:06 之前保留在内存中，然后才会刷新。如果收到的时间戳为 10:04:49 的另一个事件在 10:05:21，这将进入我们的第一个块，而不是覆盖接收时间的块。 |
| `chunk_limit_size` | 这定义了块的最大大小，默认为内存中的 8 MB 和文件缓冲区中的 256 MB。提高此阈值的可能性很小，但您可以考虑将其降低以限制容器的最大占用空间或物联网设备的限制。请记住，您可以操作多个块。 |
| `chunk_limit_records` | 这定义了单个块中日志事件的最多数量。如果日志事件的大小波动很大，则需要考虑这一点。大量的大型日志可能会创建一个非常大的块，从而产生内存耗尽和块写入持续时间变化的风险。 |
| `total_limit_size` | 这是所有块允许的存储限制，在新事件接收之前，将丢弃错误事件。默认为内存中的 512 MB 和文件中的 64 GB。 |
| `chunk_full_`阈值` | 一旦缓冲区容量的百分比超过此值，块就被视为已满，并移动到 I/O 阶段。默认为 0.95。如果日志事件相对于分配的内存非常大，您可以考虑降低此阈值以确保更可预测的性能，尤其是如果您限制了队列大小。 |
| `queued_chunks_limit_size` | 这定义了队列中等待按需持久化的块的数量。理想情况下，这个值不应大于 `flush_thread_count`。默认值是 1。 |
| `compress` | 仅接受 `text`（默认）或 `gzip` 的值。当设置为 `gzip` 时，则应用压缩。如果引入了其他压缩机制，则可用的选项将扩展。 |
| `flush_at_shutdown` | 这告诉缓冲区是否应在允许 Fluentd 优雅地关闭之前将所有内容写入输出。对于内存缓冲区，默认为 true，但对于文件，默认为 false，因为内容可以在启动时恢复。鉴于您可能不知道 Fluentd 将何时重新启动并处理缓存的事件（如果可以的话），我们建议在大多数情况下将其设置为 true。 |
| `flush_interval` | 这是一个持续时间，定义了缓冲内容应该写入输出存储机制的频率。这意味着我们可以配置基于体积或时间间隔的行为。默认为 60 秒（60s）。 |

| `flush_mode` | 接受的值是

+   `default`—如果定义了块键，则使用 `lazy`，否则使用 `interval`

+   `lazy`—每次 `timekey` 时刷新/写入块一次。

+   `interval`—通过 `flush_interval` 指定的时间间隔刷新/写入块。

+   `immediate`—在事件被追加到块中后立即刷新/写入块。

|

| `flush_thread_count` | 用于写入块的线程数量。大于 1 的数字将创建并行线程——这取决于输出类型，可能不是所希望的。例如，如果数据库连接池可以处理多个连接，那么超过 1 是值得考虑的。但在文件上超过 1 可能会造成竞争或写入冲突。默认为 1。 |
| --- | --- |
| `flush_thread_interval` | 清洗线程在检查是否需要刷新之前应该休眠的时间长度。以浮点数格式表示秒数，默认为 1。 |
| `delayed_commit_timeout` | 当使用异步 I/O 时，我们需要设置一个最大时间，允许线程运行，在我们认为它必须遇到错误之前。如果这个时间超过了（默认 60 秒），则停止线程。这需要根据目标系统的响应性进行调整。例如，将大块数据写入远程数据库将比将小块数据写入本地文件系统花费更长的时间。 |

| `overflow_action` | 如果缓冲区的输入速度超过我们能够从缓冲区中写入内容的速度，我们将遇到溢出条件。此配置允许我们定义如何解决这个问题。选项有

+   `throw_exception`—抛出一个异常，该异常将作为`BufferOverflowError`出现在 Fluentd 日志中；这是默认设置。

+   `block`—阻止输入处理以允许写入事件。

+   `interval`—通过`flush_interval`按指定时间刷新/写入块。

+   `drop_oldest_chunk`—丢弃最旧的块数据以释放一个块供使用。

+   抛出异常在永远不会预期溢出场景时可能是可以接受的，并且潜在的日志事件丢失是一个值得承担的风险。但在更关键的区域，我们建议有意识地选择一个替代方案，例如`interval`。

|

我们可以通过理解缓冲区行为（或使用准备好的配置）来修改配置。由于我们建议在关闭时刷新，我们应该将其设置为 true（`flush_at_shutdown true`）。由于我们希望快速看到更改的影响，让我们将最大记录数设置为 10（`chunk_limit_records 10`）并将刷新块的最大时间设置为 30 秒（`flush_interval 30`）。否则，如果缓冲区中有 1 到 9 个日志事件，如果源停止创建日志事件，它们将永远不会被刷新。最后，我们在配置中增加了一层额外的保护，通过强制执行缓冲区写入过程的时间超时。我们可以在以下列表中看到这一点。

列表 4.2 第四章/Fluentd/rotating-file-read-file-out2.conf—匹配提取

```
<match *>
    @type file                             
    path ./Chapter4/fluentd-file-output    
    <buffer>
     flush_at_shutdown true       ❶

      delayed_commit_timeout 10        
     chunk_limit_records 10       ❷

     flush_interval 30            ❸

    </buffer>
</match>
```

❶ 默认情况下，文件缓冲区在关闭时不会刷新，因为停止 Fluentd 实例不会导致事件丢失。然而，在关闭时看到所有事件完成是可取的。存在这样的风险，即配置更改意味着文件缓冲区在重启时不会被选中，导致日志事件实际上处于悬而未决的状态。

❷ 由于我们理解日志内容并希望事情发生得非常快，我们将使用多个日志而不是控制块大小的能力，这间接影响事件从缓冲区移动到输出目标的速度。

❸ 由于自我监控事件的累积速度将比我们的文件源慢得多，因此强制按时间刷新也将确保我们可以以合理的频率看到这些事件通过输出。

要运行这个场景，让我们重置（删除`structured-rolling-log.*`和`rotating-file-read.pos_file`文件以及`fluentd-file-output`文件夹）并再次运行，在每个单独的 shell 中使用这些命令：

+   `fluentd -c Chapter4/Fluentd/rotating-file-read-file-out2.conf`

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/jul-log-file2.properties ./TestData/medium-source.txt`

一旦日志模拟器完成，不要关闭 Fluentd。我们会看到文件夹`fluentd-file-output`仍然被创建，并且像之前一样包含缓冲文件。但与此同时，我们还会看到以`fluentd-file-output.<date>_<incrementing number>.log`（例如，`fluentd-file-output.20200505_12.log`）命名的文件。打开这些文件中的任何一个，你会看到 10 行日志数据。你会注意到日志数据是以日期时间戳、标签名称，然后是有效负载体格式化的，反映了日志事件的标准化组成。如果你浏览这些文件，你会找到标签不是 simpleFile 而是 self 的事件。这反映了我们保留了源报告在自我监控上，并且与我们的 simpleFile 相匹配，simpleFile 正在跟踪旋转日志文件。

最后，优雅地关闭 Fluentd。在 Windows 中，在 shell 中这样做最简单：按一次`CTRL-c`（并且只按一次），然后对关闭提示响应 yes（在 Linux 中，可以使用中断事件）。一旦我们可以在控制台中看到 Fluentd 已关闭，查找最后一个日志文件并检查它。这涉及到一些时间因素，但如果检查最后一个文件，它很可能包含少于 10 条记录，因为缓冲区在关闭时将刷新到输出文件中任何日志事件。

缓冲区大小错误

如果你设置缓冲区小于单个日志事件，那么处理该日志事件将失败，出现如下错误

```
emit transaction failed: error_class=Fluent::Plugin::Buffer
::BufferChunkOverflowError error="a 250bytes record is larger than
buffer chunk limit size" location="C:/Ruby26-x64/lib/ruby/gems/2.6.0/
gems/fluentd-1.9.3-x64-mingw32/lib/fluent/plugin/buffer.rb:711:in `block
in write_step_by_step'"
```

### 4.1.4 重试和退避

缓冲区的使用还允许 Fluentd 提供重试机制。在出现如短暂网络中断等问题的情况下，我们可以告诉缓冲区在识别到问题时进行重试，而不是仅仅丢失日志事件。为了使重试工作而不产生新的问题，我们需要定义控制措施，告诉缓冲区在放弃数据之前重试多长时间或多少次。此外，我们还可以定义在重试之前等待多长时间。我们可以规定无限期重试（将`retry_forever`属性设置为 true），但我们建议非常谨慎地使用此类选项。

使用`retry_type`属性有两种方式来实现重试：通过固定间隔（`periodic`）或通过指数回退（`exponential_backoff`）。指数回退是默认模型，每次重试失败都会导致重试延迟加倍。例如，如果重试间隔是 1 秒，第二次重试将是 2 秒，第三次重试是 4 秒，依此类推。我们可以通过定义`retry_wait`（使用表示秒数的数值）来控制重试之间的初始或重复等待周期。例如，1 表示 1 秒，60 表示 1 分钟。

除非我们希望无限期地重试，否则我们需要提供一种方法来确定是否继续重试。对于周期性重试模型，我们可以通过数量或时间来控制。这是通过设置重试写入每个块的最大周期（`retry_timeout`）或最大重试尝试次数（`retry_max_attempts`）来实现的。

对于回退方法，我们可以规定回退次数（`retry_exponential_backoff_base`）或回退可以持续的最大时长，在停止之前（`retry_max_interval`）。

假设我们想要配置缓冲重试为从 3 秒开始的指数回退。在最多 10 次尝试的情况下，我们可能会达到近 26 分钟的重试间隔峰值。我们需要配置的配置属性包括

```
retry_exponential_backoff_base 3
retry_max_times 10
```

指数回退的重要之处在于确保你了解可能的总时间。一旦指数曲线开始，时间会迅速延长。在这个例子中，前 5 次重试将在一分钟内发生，但之后间隔会真正开始拉长。

### 4.1.5 将配置缓冲大小设置付诸实践

你被要求帮助团队更好地理解缓冲机制。有一个共识，即应该修改现有的配置以帮助实现这一点。复制配置文件 `/Chapter4/Fluentd/rotating-file-read-file-out2.conf` 并修改它，以便缓冲块配置基于 500 字节的尺寸（参见附录 B 了解如何表示存储大小）。

运行修改后的配置以展示对输出文件的影响。

在讨论过程中，出现的一个问题是如果输出插件受到间歇性网络问题的影响，我们有哪些选项可以防止任何日志信息的丢失？

答案

以下列表显示了将包含在结果中的缓冲区配置。

列表 4.3 `Chapter4/ExerciseResults/rotating-file-read-file-out2-Answer.conf`

```
<match *>
    @type file
    @id bufferedFileOut
    path ./Chapter4/fluentd-file-output
    <buffer>                             ❶

      delayed_commit_timeout 10
      flush_at_shutdown true
      chunk_limit_size 500               ❷
      flush_interval 30
    </buffer>
</match>
```

❶ 注意，我们保留了延迟和刷新时间，所以如果缓冲区停止填充，它仍然会被强制输出。

❷ 基于大小的约束而不是基于时间的

完整的配置文件在`Chapter4/ExerciseResults/rotating -file-read-file-out2-Answer.conf`中提供。日志文件的变化率可能看起来不同，但内容将是相同的。为了解决减轻日志丢失风险的问题，可以应用几种选项：

+   在缓冲区上配置重试和退避参数，以便在丢失信息之前重试存储事件。

+   使用定义一个*二级*日志机制的能力，例如本地文件，这样事件就不会丢失。提供一种方法，以便在稍后日期将日志注入到 Kafka 流中。这甚至可以是一个额外的来源。

## 4.2 输出格式化选项

我们如何结构化日志事件的输出，与我们如何对输入应用结构一样重要。不出所料，格式化插件可以包含在输出插件中。有了格式化插件，合理地期望有几个预构建的格式化器。让我们看看通常遇到的“开箱即用”的格式化器；完整的格式化器集在附录 C 中详细说明。

### 4.2.1 输出文件

这可能是最简单的格式化器，并且已经被隐式使用。格式化器通过在`time`、`tag`和`record`值之间使用分隔符来工作。默认情况下，`*分隔符*`是一个制表符字符。这只能通过将属性分隔符的值更改为`comma`或`space`来更改，例如，`delimiter comma`。也可以通过基于布尔值的属性来控制输出哪些字段：

+   `output_tag`—这个属性接受真或假值来决定是否在行中包含标签（默认情况下，行中的第二个值）。

+   `output_time`—这个属性接受真或假来定义是否包含时间（默认情况下，行中的第一个值）。如果你已经在核心事件记录中包含了时间，你可能希望省略时间。

+   `time_format`—这个属性可以用来定义日期和时间在路径中使用的方式。如果未定义且设置了 timekey，则 timekey 将告知格式的结构。具体来说：

    +   0...60 秒则使用`'%Y%m%d%H%M%S'`

    +   60...3600 秒则使用`'%Y%m%d%H%M'`

    +   3600...86400 秒则使用`'%Y%m%d%H'`

注意：如果格式化器无法识别设置的属性值，它将忽略提供的值并使用默认值。

### 4.2.2 JSON

这个格式化器将日志事件记录视为单行上的 JSON 有效负载。与事件关联的时间戳和标签被丢弃。

### 4.2.3 LTSV

与`out_file`格式化程序和解析器一样，可以使用`*delimiter*`（每个标签值）和`label_delimiter`来更改限制器，以分隔值和标签。例如，如果将分隔符设置为`delimiter ;`并且设置了`label_delimiter =`，那么如果记录表示为`{"my1stValue":" blah", "secondValue": "more blah", "thirdValue": "you guessed – blah"`，输出将变为`my1st Value=blah; secondValue= more blah; thirdValue=you guessed – blah.`。

由于值是标签值，因此减少了对按行分隔记录的需求，因此可以通过将`add_newline false`（默认值为 true）关闭来停止使用新行来分隔每个记录。

### 4.2.4 csv

就像 ltsv 和`out_file`一样，可以通过设置属性`delimiter`来定义分隔符。该属性有一个默认值，即逗号。此外，csv 输出允许我们使用`fields`属性定义可以包含在输出中的值。如果我用记录再次说明，如果我的事件是`{"my1stValue":" blah"`，`"secondValue":" more blah"`，`"thirdValue":"you guessed – blah"`，并且我将字段属性设置为`secondValue, thirdValue fields`，那么输出将是`"more blah"`，`"you guessed – blah"`。如果需要，可以通过`force_quotes`的布尔值禁用每个值的引号。

### 4.2.5 msgpack

与`msgpack`解析器一样，格式化程序与 MessagePack 框架一起工作，该框架使用核心日志事件记录，并使用 MessagePack 库来压缩内容。通常，我们只期望在期望接收 MessagePack 内容的 HTTP 转发输出插件中使用它。为了获得类似的压缩性能提升，我们可以使用 gzip 进行文件和块存储。

注意：你可能已经注意到，大多数格式化程序（`out_file`除外）省略了从日志事件中添加时间和键。因此，如果你想保留这些信息，将需要确保它们被包含在日志事件记录中，或者输出插件明确地使用这些值。可以使用`*inject*`插件将附加数据添加到日志事件有效负载中，该插件可以在`*match*`或`*filter*`指令中使用。我们将在第六章讨论过滤时介绍注入功能。

### 4.2.6 应用格式化程序

我们可以将现有的配置扩展为从当前的隐式配置转换为显式配置。让我们从使用默认的`out_file`格式化程序开始，使用逗号作为分隔符（`delimiter comma`）并排除日志事件标签（`output_tag false`）。我们将继续使用之前相同的源来演示格式化程序的效果。以下列表显示了此`out_file`格式化程序配置。

列表 4.4 第四章/Fluentd/rotating-file-read-file-out3.conf—格式化程序配置

```
<match *>
    @type file
    @id bufferedFileOut
    path ./Chapter4/fluentd-file-output
    <buffer>
      delayed_commit_timeout 10
      flush_at_shutdown true
      chunk_limit_records 50        ❶

      flush_interval 30
      flush_mode interval
    </buffer>
    <format>
      @type out_file                ❷

      delimiter comma               ❸
      output_tag false              ❹
    </format>
</match>
```

❶ 为了减少生成的文件数量，我们已将每个文件中的记录数设置得很大。

❷ 我们明确地将输出格式器定义为 out_file，因为我们想覆盖默认的格式化行为。

❸ 将制表符分隔符替换为逗号。

❹ 从输出中排除标签信息。

假设现有的日志文件和输出已经被移除，我们可以使用以下命令开始示例：

+   `fluentd -c Chapter4/Fluentd/rotating-file-read-file-out3.conf`

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/jul-log-file2.properties ./TestData/medium-source.txt`

### 4.2.7 将 JSON 格式化配置投入实际应用

您的组织已决定，作为标准做法，所有输出都应使用 JSON 结构完成。这种方法确保了日志事件中任何现有的或应用的结构意义不会丢失。为了支持这一目标，需要修改配置文件 `/Chapter4/Fluentd/rotating-file-read-file-out3.conf`。

答案

配置文件中的格式声明应减少到类似于以下列表中的片段。

列表 4.5 Chapter4/ExerciseResults/rotating-file-file-out3-Answer.conf

```
<format>
@type json
</format>
```

本答案的完整示例配置可以在 `/Chapter4/ ExerciseResults/rotating-file-read-file-out3-Answer.conf` 中找到。

## 4.3 将日志事件发送到 MongoDB

虽然将日志事件输出到某种形式的基于文件的存储是一种简单且易于存储日志事件的方法，但它并不适合执行任何分析或数据处理。为了使日志分析成为实际可能，Fluentd 需要与能够执行分析的系统进行交互，例如 SQL 或 NoSQL 数据库引擎、搜索工具如 Elasticsearch，甚至是 SaaS 服务如 Splunk 和 Datadog。

第一章强调了 Fluentd 对任何特定的日志分析引擎或供应商都没有忠诚度，这使得 Fluentd 与许多其他工具区分开来。因此，许多供应商发现 Fluentd 是将日志事件输入其产品或服务的有吸引力的解决方案。为了使采用变得非常容易，供应商已经开发了他们的适配器。

我们选择使用 MongoDB 来帮助说明将事件输入到能够进行日志分析的工具的方法。虽然 MongoDB 不像 Elasticsearch 那样专注于文本搜索，但其功能非常适合具有良好 JSON 结构的日志事件。MongoDB 在入门时非常灵活且要求不高，所以如果你没有使用过 MongoDB，请不要担心。MongoDB 的安装指南可以在附录 A 中找到。

MongoDB 概述

我们不希望过多地偏离 MongoDB 的机制，但总结一些基本概念是值得的。大多数读者都熟悉关系型数据库及其概念。数据库结构在 MongoDB 和关系型数据库中的作用是相似的。在数据库模式中有一组表。在 MongoDB 中，与之最接近的是“集合”。与关系型数据库不同，集合可以包含几乎所有内容，这就是为什么它有时被描述为使用“文档模型”。可以说，集合中的每个条目大致相当于一个包含 DB 分配的 ID 和 BLOB（大型二进制对象）或文本数据类型的表中的记录。通常，MongoDB 的行或文档是一个结构化文本对象，通常是 JSON 格式。MongoDB 可以然后搜索和索引这些结构的一部分，从而提供一种灵活的解决方案。对于 Fluentd 来说，这意味着我们可以存储可能具有不同记录结构的日志事件。

更新版的 MongoDB 引擎提供了验证进入集合的内容结构的手段。这为内容提供了一定的可预测性。如果使用此功能，那么我们可以利用 Fluentd 来结构化必要的有效负载。

除了控制每个文档的内容必须严格遵循结构外，Mongo 还引入了“固定大小”的概念。此功能允许我们限制集合使用的存储量，并且集合作为 FIFO（先进先出）列表运行。

当您想要清空关系型数据库中的表时，通常简单地删除并重新创建表会更简单。MongoDB 的等效操作是删除集合；然而，如果这是数据库中唯一的集合，MongoDB 将会删除整个数据库。有两种选择：仅删除集合的内容并保留集合，或者创建一个空的第二个集合，以保持数据库不为空。

您可以通过阅读 Kyle Banker 等人所著的《MongoDB in Action》一书来发现更多内容（[www.manning.com/books/mongodb-in-action-second-edition](https://www.manning.com/books/mongodb-in-action-second-edition)）。

### 4.3.1 部署 MongoDB Fluentd 插件

MongoDB 输入插件包含在 Fluentd 的 Treasure Data Agent 构建中，但不包含在标准部署中。如果我们想让 Fluentd 与 MongoDB 一起工作，我们需要安装 RubyGem，如果它尚未安装的话。

为了确定是否需要安装以及执行 gems 的安装，我们可以使用一个利用 RubyGems 工具的包装工具，称为 *fluent-gem*。要查看 gem 是否已经安装，请在命令行中运行 `fluent-gem list` 命令。该命令将显示本地安装的 gems，其中包含 Fluentd 及其插件。在此阶段，不应有任何关于 *fluent-plugin-mongo* gem 的指示。因此，我们可以使用命令 `fluent -gem install fluent-plugin-mongo` 来执行安装。这将检索并安装该 gem 的最新稳定版本，包括文档和依赖项，如 MongoDB 驱动程序。

### 4.3.2 配置 Fluentd 的 Mongo 输出插件

在一个匹配规则中，我们需要引用 `mongo` 插件并设置相关属性。就像连接到任何数据库一样，我们需要提供一个地址（可以通过 *host* [名称] 和 *port* [网络上的端口号] 实现）以及用户名和密码，或者通过 *connection* *string*（例如，`mongodb://127.0.0.1:27017/Fluentd`）。在我们的示例配置中，我们采用了前一种方法并避免了凭证问题。需要数据库（模式）和集合（类似于关系型数据库中的表）来确定日志事件放置的位置。

在 MongoDB 输出中，不需要使用格式化工具，因为 MongoDB 插件假设所有内容已经结构化。正如我们所见，如果没有配置，将采用默认的缓冲区。像以前一样，我们将保持缓冲区设置以支持低流量，这样我们可以快速看到变化。我们可以在以下列表中看到结果。

列表 4.6 Chapter4/Fluentd/rotating-file-read-mongo-out.conf—匹配配置

```
<match *>
    @type mongo                ❶
    @id mongo-output   
    host localhost             ❷

    port 27017                 ❸

    database Fluentd           ❹

    collection Fluentd         ❺

    <buffer>                   ❻

      delayed_commit_timeout 10
      flush_at_shutdown true
      chunk_limit_records 50
      flush_interval 30
      flush_mode interval
    </buffer>
</match>
```

❶ Mongo 是插件名称。

❷ 识别 MongoDB 主机服务器。在我们的开发设置中，这仅仅是本地机器；这也可以通过使用连接属性来定义。

❸ 与目标服务器通信的端口。我们也可以通过将主机名、端口和数据库组合成一个字符串来表示 URI。

❹ 由于 MongoDB 安装可以支持多个数据库，我们需要命名数据库。

❺ 我们想要添加日志事件的数据库中的集合——类似于基于 SQL 的表

❻ 缓冲区配置以确保日志事件能够快速集成到 MongoDB 中

Mongo 与 Mongo 副本集插件

如果你已经查看过 Fluentd 在线文档，你可能已经注意到两个输出插件，*out_mongo*和*out_mongo_replset*。关键区别在于*replset*（副本集）可以支持 MongoDB 的扩展方法，其中可以定义 Mongo 的额外实例作为主副本。当这种情况发生时，理想的模式是直接将活动写入主节点，但从副本节点读取。在配置差异方面，需要一个以逗号分隔的节点列表，而不是指定单个主机名。每个节点代表副本组中的一个节点（例如，`nodes 192.168.0.10:27017, 192.168.0.20:27017, 192.168.0.30:27017`）。还需要副本集名称（例如，`replica_set myFluentReps`）。有关 Mongo 副本集机制的更多信息，请参阅[`docs.mongodb.com/manual/replication/`](https://docs.mongodb.com/manual/replication/)。

确保 MongoDB 实例正在运行（这可以通过在 shell 窗口中运行命令`mongod --version`或通过使用 Compass UI 尝试连接到服务器来完成）。如果 MongoDB 服务器没有运行，那么它需要被启动。最简单的方法是运行命令`mongod`。

当 Mongo 运行时，我们可以在每个 shell 中运行我们的模拟日志和 Fluentd 配置命令：

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/jul-log-file2-exercise.properties ./TestData/medium-source.txt`

+   `fluentd -c Chapter4/fluentd/rotating-file-read-mongo-out.conf`

Mongo 插件启动警告

当 Fluentd 与 MongoDB 插件一起启动时，它将记录以下警告：`[mongo-output]`。从 v0.8 版本开始，无效记录检测将被移除，因为 mongo 驱动程序 v2.x 和 API 规范不提供它。你可能会丢失无效记录，因此你不应该将此类记录发送到 Mongo 插件。

这实际上意味着正在使用的 MongoDB 驱动程序不对有效负载（集合可能已配置为需要）进行任何结构检查。因此，如果 MongoDB 引擎正在应用强检查，它可能会丢弃更新，但信息不会通过驱动程序返回，因此 Fluentd 将一无所知，导致数据丢失。对于 Fluentd 更常见的应用，最好不对有效负载施加严格的检查。如果这不是一个选项，那么一个过滤器指令可以识别将失败 MongoDB 检查的日志事件。

![](img/CH04_F02_Wilkins.png)

图 4.2 通过 Compass UI 工具查看 MongoDB，其中包含日志内容。

在 MongoDB 中查看日志事件

要查看 MongoDB 在查询 JSON 日志事件方面的有效性，如果您将表达式`{"msg" : {$regex : ".*software.*"}}`添加到 FILTER 字段并点击 FIND，我们将得到一些结果。这些结果将显示包含单词 software 的`msg`的日志事件。查询表达式告诉 MongoDB 在文档中查找，如果它们有一个名为`msg`的顶级元素，则使用正则表达式评估此值。

如果您检查 MongoDB 中的内容，您将看到存储的内容仅仅是核心日志事件记录，而不是相关的时间或标签。

要在命令行中清空集合以便进一步执行场景，请运行以下命令：

```
mongo Fluentd –-eval "db.Fluentd.remove({})".
```

mongo 插件还有一些其他技巧。当配置中包含`tag_mapped`属性时，标签名称用作集合名称，如果不存在，MongoDB 将创建该集合。这使得将日志事件分离到不同的集合变得极其简单。如果标签名称已经按层次结构使用，则可以删除标签前缀以简化`tag_mapped`功能。这可以通过`remove_tag_prefix`属性定义，它接受要删除的前缀名称。

由于集合可以动态建立，可以在配置中定义特性；例如，集合是否应该限制大小。

在此配置中，我们没有正式定义任何用户名或密码。这是因为我们在 MongoDB 配置中没有强制凭证限制。在 Fluentd 配置文件中包含凭证也不是最佳实践。第七章中讨论了在 Fluentd 配置中安全处理凭证的技术，不仅适用于 MongoDB，也适用于需要身份验证的其他系统。

### 4.3.3 将 MongoDB 连接配置字符串投入实际应用

修改配置以通过连接属性定义连接，而不是使用主机、端口和数据库名。这最好从复制配置文件`Chapter4/fluentd/rotating-file-read-mongo-out.conf`开始。调整运行命令以使用新的配置。命令看起来可能如下所示：

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/jul-log-file2-exercise.properties ./TestData/medium-source.txt`

+   `fluentd -c Chapter4/fluentd/my-rotating-file-read-mongo-out.conf`

答案

MongoDB 连接的配置应类似于以下列表中所示的配置。

列表 4.7 Chapter4/ExerciseResults/rotating-file-read-mongo-out-Answer.conf

```
<match *>
    @type mongo
    @id mongo-output
    connection_string mongodb://localhost:27017/Fluentd     ❶

    collection Fluentd
    <buffer>
      delayed_commit_timeout 10
      flush_at_shutdown true
      chunk_limit_records 50
      flush_interval 30
      flush_mode interval
    </buffer>
</match>
```

❶ 注意，为了这个目的，省略了主机、端口和数据库名属性。您可以使用您主机的特定 IP 地址或 127.0.0.1 而不是 localhost。

示例配置可以在`Chapter4/ExerciseResults/rotating-file-read-mongo-out-Answer.conf`中查看。

在我们离开 MongoDB 之前，我们关注的是 MongoDB 的输出用途，以便它可以用于查询日志事件。但 MongoDB 也可以作为 Fluentd 的输入，允许新添加到 MongoDB 的记录作为日志事件检索。

## 4.4 可采取行动的日志事件

在第一章中，我们介绍了使日志事件可采取行动的想法。到目前为止，我们已经看到了统一日志事件的方法。为了使日志事件可采取行动，我们需要几个元素：

+   能够将事件发送到外部系统，该系统能够触发动作，例如 Ops 人员可以看到并对此做出反应的协作/通知平台，或者调用脚本或工具执行动作

+   将需要采取行动的日志事件与仅提供信息但不需要特定行动的事件分开

在本书的后面部分将讨论如何分离或过滤出需要采取行动的事件。但在这里，我们可以看看如何在所有日志都到达分析平台并运行其分析过程之前，如何使事件可采取行动。

### 4.4.1 通过服务调用实现可采取行动的日志事件

使日志事件可采取行动的一种方法是通过调用一个可以执行必要修复的应用程序，作为接收 API 调用的结果。这可能就像调用 *Ansible Tower* REST API ([`mng.bz/Nxm1`](http://mng.bz/Nxm1)) 来启动一个执行一些日常维护工作（例如，将日志移动到存档存储或告知 Kubernetes 有关 Pod 的内部状态）的模板作业一样聪明。我们需要控制动作执行的频率；例如，*flow_counter* 和 *notifier* 这样的插件可以帮助。为了调用通用网络服务，我们可以使用 Fluentd 核心部分之一的 HTTP 输出插件。为了给出可能的技艺感，以下是此插件支持的一般功能的总结：

+   支持 HTTP *post* 和 *put* 操作

+   允许在路由中使用代理

+   配置内容类型，自动设置格式化器（格式化器也可以显式定义）

+   定义头信息，以便可以定义所需的额外头值（例如，API 密钥）

+   配置连接以使用 SSL 和 TLS 证书，包括定义要使用的证书的位置、版本、要使用的加密套件等。

+   当收到非成功的 HTTP 代码响应时，创建日志错误

+   支持基本身份验证（截至编写时，不支持 OAuth）

+   设置超时

+   使用缓冲区插件

### 4.4.2 通过用户交互工具实现可采取行动

以这种方式自动化问题解决使我们朝着自愈系统迈进，但并非许多组织都为这种高级状态做好了准备，或者必然想要达到这种程度。他们更愿意信任快速的人工干预来确定因果关系，而不是依赖于自动化诊断，在自动化诊断中，高度的确定性可能很难实现。拥有足够知识和适当信息的人可以迅速确定并解决此类问题。因此，Fluentd 有一套丰富的插件，用于社交协作机制。以下只是几个例子：

+   IRC (互联网中继聊天) ([`tools.ietf.org/html/rfc2813`](https://tools.ietf.org/html/rfc2813))

+   Twilio (支持许多不同的通讯渠道) ([www.twilio.com](https://www.twilio.com))

+   Jabber ([`xmpp.org`](https://xmpp.org))

+   Redmine ([www.redmine.org](https://www.redmine.org))

+   Typetalk ([www.typetalk.com](https://www.typetalk.com))

+   PagerDuty ([www.pagerduty.com](https://www.pagerduty.com))

+   Yammer ([www.microsoft.com/en-gb/microsoft-365/yammer](https://www.microsoft.com/en-gb/microsoft-365/yammer))

+   Slack ([`slack.com/`](https://slack.com/))

显然，我们需要在社交渠道通信中包含相关信息。为了帮助做到这一点，可以采取一系列措施，从清晰的日志事件，这些事件可以链接到解决方案指南，到将 Fluentd 配置为从日志事件中提取相关信息以共享。

## 4.5 Slack 示例以展示社交输出

Slack 已经成为一个领先的团队消息协作工具，它拥有强大的 API 层和一个免费版本，该版本仅受对话存档大小的限制。作为一个云服务，它是展示 Fluentd 与通过社交平台可操作日志事件交集的绝佳工具。虽然以下步骤是针对 Slack 的，但涉及的原则对 Microsoft Teams、Jabber 以及许多其他协作服务都是相同的。

如果你已经在使用 Slack，可能会倾向于使用现有的群组来运行示例。为了避免在你用测试日志事件填满频道时，其他 Slack 工作空间用户收到通知和消息而感到烦恼，我们建议设置你自己的测试工作空间。如果你没有使用 Slack，那也不是问题；在附录 A 中，我们解释了如何获取 Slack 账户并将其配置为准备使用。确保你在 Slack 配置过程中记下 API 令牌，该令牌以`xoxb`为前缀。

Slack 提供了一组丰富的配置选项以供交互使用。与 MongoDB 和许多 IaaS- 和 PaaS-级别的插件不同，由于 Slack 是一个 SaaS 服务，因此 Slack 实例的解析既简化了，又通过使用单个 `token`（无需服务器地址等）隐藏于我们面前。`username` 并非关于凭证，而是关于如何表示 Fluentd 插件所扮演的机器人；因此，使用一个有意义的名称是值得的。`channel` 与消息将被显示的 Slack 频道相关。通用频道默认存在，但如果你想在 Slack 中创建一个自定义频道并限制对该频道的访问，以便控制谁可以看到消息，你可能希望这样做。毕竟，你希望企业 Slack 设置中的每个人都看到每个操作消息吗？

`message` 和 `message_keys` 属性与消息一起使用，使用 `%s` 来指示已识别的有效负载元素的值插入的位置。`message` 中的引用与在 `message_keys` 中按顺序列出的 JSON 有效负载元素相关。

`title` 和 `title_keys` 与 `message` 和 `message_keys` 的工作方式类似，但用于 Slack UI 中显示的消息的标题。在我们的案例中，我们只是将使用 `tag`。最后一部分是 `flush` 属性；这告诉插件如何快速将 Slack 消息推送给用户。如果周期太长，可以将多个消息分组。为了保持快速流动，让我们每秒刷新一次。

编辑提供的现有配置（`Chapter4/Fluentd/rotating-file -read-slack-out.conf`），以在 Slack 设置中包含捕获的详细信息。以下列表展示了这一点。

列表 4.8 第四章/Fluentd/rotating-file-read-slack-out.conf—匹配配置

```
<match *>
   @type slack
   token xoxb-9999999999999-999999999999-XXXXXXXXXXXXXXXXXXXXXXXX    ❶

   username UnifiedFluent                                            ❷
   icon_emoji :ghost:                                                ❸
   channel general                                                   ❹

   message Tell me if you've heard this before - %s                  ❺

   message_keys msg                                                  ❻

   title %s                                                          ❼

   title_keys tag                                                    ❽
   flush_interval 1s                                                 ❾

</match>
```

❶ 这就是从 Slack 获取的令牌被放置以正确识别工作空间并合法化连接的地方。

❷ 将在 Slack 对话中显示的用户名

❸ 定义与机器人关联的单独的 emoji（图标）

❹ 根据名称确定将消息放置在哪个频道。在我们的演示中，我们只是使用默认频道。

❺ 要显示的消息，按照配置中提供的顺序引用值。消息属性与消息 _keys 属性协同工作。

❻ 为要包含在日志事件中的日志事件的 JSON 元素命名。然后，该命名元素由消息标签获取并插入到消息文本中。

❼ 消息的标题采用 title_keys 的值。使用顺序在配置值之间进行映射。

❽ 消息标题的定义

❾ 定义 Fluentd 获取 Slack 发布日志事件的频率

在运行解决方案之前，我们还需要使用命令 `fluent-gem install fluent-plugin-slack` 安装 Slack 插件。一旦安装完成，我们就可以使用以下命令启动日志模拟器和 Fluentd：

+   `fluentd -c Chapter4/Fluentd/rotating-file-read-slack-out.conf`

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/social-logs.properties ./TestData/small-source.txt`

一旦启动，如果你在 Web 客户端或应用程序中打开*#general*频道，你将看到 Fluentd 流过的消息。

Slack 插件的全部细节可以从[`github.com/sowawa/fluent-plugin-slack`](https://github.com/sowawa/fluent-plugin-slack)获得。我们关于 Slack 使用的说明相对简单（图 4.3）。通过使用几个插件，我们可以快速从源标签路由 Slack 消息到最相关的个人或直接。或者，我们可以为每个应用程序设置不同的频道，并将消息直接发送到这些频道。

![图片](img/CH04_F03_Wilkins.png)

图 4.3 在 Slack 中显示的我们的 Fluentd 日志事件

### 4.5.1 更小心地处理令牌和凭证

很长时间以来，良好的安全实践告诉我们不应该将凭证硬编码到代码和配置文件中，因为任何人都可以查看文件并获取敏感凭证。实际上，如果你将代码提交到 GitHub，它将标记任何包含 GitHub 所知的服务的安全令牌字符串的配置文件或代码。当 Slack 被告知并决定这是一个有效的令牌时，请放心，它将吊销该令牌。

那么，我们如何解决这个问题呢？根据你的情况，有一系列策略；以下是一些选项：

+   使用具有有限会话作用域的环境变量设置敏感凭证。可以通过多种方式配置环境变量，例如使用 Chef 和 Puppet 等工具从密钥库中设置值。

+   在应用程序或配置中嵌入访问密钥库或秘密管理解决方案（如 HashiCorp 的 Vault）的方法。

配置文件可能看起来不像我们在这本书中看到的那样可以基于凭证进行安全保护。但是，我们可以通过在 Fluentd 配置文件中嵌入 Ruby 片段来实现这两种方法来安全地管理凭证，因为 Fluentd 允许我们这样做。这并不意味着我们立即需要学习 Ruby。对于这些方法中的第一种，我们只需要理解几个基本模式。嵌入对 Vault 的调用方法更具挑战性，但可以完成。

```
   token “#{ENV[’slack-token']}”
```

### 4.5.2 在实际操作中外部化 Slack 配置属性

挑战在于设置你的环境，以便你有一个名为*SlackToken*的环境变量，该变量设置为保存你之前获得的令牌。然后自定义`Chapter4/Fluentd/rotating-file-read-slack-out.conf`以使用环境变量，并使用以下命令重新运行示例设置

+   `fluentd -c Chapter4/Fluentd/rotating-file-read-slack-out.conf`

+   `groovy LogSimulator.groovy Chapter4/SimulatorConfig/social-logs.properties ./TestData/small-source.txt`

确认日志事件是否已到达 Slack。

答案

通过设置环境变量，您将创建一个看起来像以下之一的命令

```
set slack-token= xoxb-9999999999999-999999999999-XXXXXXXXXXXXXXXXXXXXXXXX  
```

或者为 Windows 或 Linux

```
Export slack-token= xoxb-9999999999999-999999999999-XXXXXXXXXXXXXXXXXXXXXXXX  
```

配置现在将改变，看起来像以下列表中的示例。

列表 4.9 第四章/rotating-file-read-slack-out-Answer.conf—匹配配置

```
<match *>
   @type slack
   token "{ENV["slack-token"]}"
   username UnifiedFluent  
   icon_emoji :ghost:  
   channel general  
   message Tell me if you've heard this before - %s
   message_keys msg  
   title %s 
   title_keys tag 
   flush_interval 1s 
</match>
```

## 4.6 适合工作的正确工具

在第一章中，我们强调了由于各种原因（如下所述）不同的人想要不同的工具的问题：

+   为了执行日志分析，不同的工具具有不同的优势和劣势

+   多云，因此专业团队（以及网络流量的成本考虑）意味着使用不同的云服务提供商工具

+   为了做出影响个人偏好和政策的决策（先前经验等）

正如我们所展示的，Fluentd 可以支持许多社交平台和协议。当然，这不会是日志事件放置的唯一地方。核心目的地类型之一是日志分析工具或平台。Fluentd 拥有大量插件来为日志分析平台提供数据；除了我们之前提到的两个之外，其他可以轻松插入的主要解决方案包括

+   Azure Monitor

+   Graphite

+   Elasticsearch

+   CloudWatch

+   Google Stackdriver

+   Sumo Logic

+   Logz.io

+   Oracle 日志分析

然后，当然，我们可以将日志发送到各种数据存储解决方案以备后用或进行数据分析；例如：

+   Postgres、InfluxDB、MySQL、Couchbase、DynamoDB、Aerospike、SQL Server、Cassandra

+   Kafka，AWS Kinesis（时间序列存储/事件流）

+   存储区域，如 AWS S3、Google Cloud Storage、Google BigQuery、WebHDFS

因此，问题变成了，我的需求是什么，哪些工具最适合？如果我们的需求随时间变化，那么我们就根据需要添加或删除我们的目标。改变技术可能会提出更多关于如何处理我们当前的日志事件的问题，而不是如何将数据放入解决方案。

## 摘要

+   Fluentd 拥有广泛的支持文件、其他 Fluentd 节点、关系型数据库和文档数据库（如 MongoDB、Elasticsearch 等）的输出插件。

+   插件支持不仅限于分析和存储解决方案，还包括协作和通知工具，如 Slack。这使得 Fluentd 能够对重要的日志事件做出更快的反应。

+   Fluentd 提供了一些强大的辅助插件，包括格式化和缓冲，使得日志事件输出配置非常高效且易于使用。

+   日志事件可以通过分析工具和可视化工具等工具变得易于消费。Fluentd 提供了使用格式化插件（如 out_file 和 json）格式化日志事件的方法。

+   缓冲辅助插件可以根据需要支持不同的生命周期，从简单的同步缓存到完全异步。有了这个，缓冲存储可以根据大小或日志事件的数量进行组织。

+   缓冲区可以被配置为不仅在关闭时刷新其内容，还可以在其他条件下刷新，例如缓冲一段时间的新事件。
