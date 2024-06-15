# 附录. 调优标志摘要

本附录涵盖了常用标志并指导何时使用它们。*常用* 在这里包括了在早期 Java 版本中常用且不再推荐的标志；旧版本 Java 的文档和提示可能会推荐这些标志，因此在这里进行了提及。

表 A-1. 调整即时编译器的标志

| 标志 | 功能 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| `-server` | 此标志不再起作用，会被默默忽略。 | 不适用 | “分层编译” |
| `-client` | 此标志不再起作用，会被默默忽略。 | 不适用 | “分层编译” |
| `-XX:+TieredCompilation` | 使用分层编译。 | 除非内存严重受限，否则始终使用。 | “分层编译” 和 “分层编译的权衡” |
| `-XX:ReservedCodeCacheSize=`*`<MB>`* | 为 JIT 编译器编译的代码保留空间。 | 运行大型程序时，如果看到代码缓存不足的警告。 | “调整代码缓存” |
| `-XX:InitialCodeCacheSize=`*`<MB>`* | 为 JIT 编译器编译的代码分配初始空间。 | 如果需要预先分配代码缓存的内存（这种情况很少见）。 | “调整代码缓存” |
| `-XX:CompileThreshold=`*`<N>`* | 设置方法或循环执行多少次后进行编译。 | 此标志已不再推荐使用。 | “编译阈值” |
| `-XX:+PrintCompilation` | 提供 JIT 编译器操作的日志。 | 当怀疑某个重要方法未被编译，或者对编译器的操作感到好奇时。 | “检查编译过程” |
| `-XX:CICompilerCount=`*`<N>`* | 设置 JIT 编译器使用的线程数。 | 当启动了过多的编译器线程时。主要影响运行多个 JVM 的大型机器。 | “编译线程” |
| `-XX:+DoEscapeAnalysis` | 启用编译器的激进优化。 | 在罕见情况下可能引发崩溃，因此有时建议禁用。除非确定它引起了问题，否则不要禁用。 | “逃逸分析” |
| `-XX:UseAVX=`*`<N>`* | 设置在 Intel 处理器上使用的指令集。 | 在 Java 11 早期版本中应将此设置为 2；在后续版本中，默认为 2。 | “特定于 CPU 的代码” |
| `-XX:AOTLibrary=`*`<path>`* | 使用指定库进行预编译。 | 在某些有限情况下，可能加速初始程序执行。仅在 Java 11 中为实验特性。 | “预编译” |

表 A-2. 选择 GC 算法的标志

| Flag | 它的作用 | 何时使用它 | 另请参阅 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| `-XX:+UseSerialGC` | 使用简单的单线程 GC 算法。 | 适用于单核虚拟机和容器，或者小（100 MB）堆。 | “串行垃圾收集器” |
| `-XX:+UseParallelGC` | 使用多线程收集年轻代和老年代，同时应用程序线程停止。 | 用于通过吞吐量调优而不是响应性；Java 8 的默认选项。 | “吞吐量收集器” |
| `-XX:+UseG1GC` | 使用多线程收集年轻代，同时应用程序线程停止，以及后台线程从老年代中删除垃圾，最小化暂停时间。 | 当您有可用的 CPU 用于后台线程，并且不希望出现长时间的 GC 暂停时使用。Java 11 的默认选项。 | “G1 GC 收集器” |
| `-XX:+UseConcMarkSweepGC` | 使用后台线程从老年代中删除垃圾，最小化暂停时间。 | 不再推荐使用；请改用 G1 GC。 | “CMS 收集器” |
| `-XX:+UseParNewGC` | 与 CMS 一起，使用多线程收集年轻代，同时应用程序线程停止。 | 不再推荐使用；请改用 G1 GC。 | “CMS 收集器” |
| `-XX:+UseZGC` | 使用实验性的 Z 垃圾收集器（仅适用于 Java 12）。 | 为了减少年轻代 GC 的暂停时间，可以同时收集。 | “并发压缩：ZGC 和 Shenandoah” |
| `-XX:+UseShenandoahGC` | 使用实验性的 Shenandoah 垃圾收集器（仅适用于 Java 12 OpenJDK）。 | 为了减少年轻代 GC 的暂停时间，可以同时收集。 | “并发压缩：ZGC 和 Shenandoah” |
| `-XX:+UseEpsilonGC` | 使用实验性的 Epsilon 垃圾收集器（仅适用于 Java 12）。 | 如果您的应用程序从不需要执行 GC。 | “无收集：Epsilon GC” |

表 A-3\. 所有 GC 算法共同的标志

| Flag | 它的作用 | 何时使用它 | 另请参阅 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| `-Xms` | 设置堆的初始大小。 | 当默认的初始大小对您的应用程序来说太小时。 | “调整堆大小” |
| `-Xmx` | 设置堆的最大大小。 | 当默认的最大大小对您的应用程序来说太小（或可能太大）时。 | “调整堆大小” |
| `-XX:NewRatio` | 设置年轻代与老年代的比例。 | 增加此比例以减少分配给年轻代的堆空间比例；降低此比例以增加分配给年轻代的堆空间比例。这只是一个初始设置；除非关闭自适应大小调整，否则比例将会变化。随着年轻代大小的减少，您将看到更频繁的年轻代 GC 和较少的完全 GC（反之亦然）。 | “调整代大小” |
| `-XX:NewSize` | 设置年轻代的初始大小。 | 当您已经精确调整了应用程序的需求时。 | “代际大小调整” |
| `-XX:MaxNewSize` | 设置年轻代的最大大小。 | 当您已经精确调整了应用程序的需求时。 | “代际大小调整” |
| `-Xmn` | 设置年轻代的初始和最大大小。 | 当您已经精确调整了应用程序的需求时。 | “代际大小调整” |
| `-XX:MetaspaceSize=*`N`*` | 设置元空间的初始大小。 | 对于使用大量类的应用程序，可以增加此值以超过默认值。 | “大小调整元空间” |
| `-XX:MaxMetaspaceSize=*`N`*` | 设置元空间的最大大小。 | 将此数字降低以限制类元数据使用的本机空间量。 | “大小调整元空间” |
| `-XX:ParallelGCThreads=*`N`*` | 设置垃圾收集器用于前台活动（例如收集年轻代和对吞吐量 GC 来说，收集老年代）的线程数。 | 在运行多个 JVM 或者在 Java 8 更新 192 之前的 Docker 容器中，可以将此值降低。考虑在非常大的系统上增加这个值以支持非常大的堆的 JVM。 | “控制并行度” |
| `-XX:+UseAdaptiveSizePolicy` | 设置后，JVM 将调整各种堆大小以尝试满足 GC 目标。 | 如果堆大小已经精确调整，请关闭此选项。 | “自适应大小调整” |
| `-XX:+PrintAdaptiveSizePolicy` | 向 GC 日志添加有关代的调整大小信息。 | 使用此标志可以了解 JVM 的操作方式。使用 G1 时，检查此输出以查看是否通过巨大对象分配触发了完整 GC。 | “自适应大小调整” |
| `-XX:+PrintTenuringDistribution` | 将续期信息添加到 GC 日志中。 | 使用续期信息确定是否以及如何调整续期选项。 | “续期和幸存者空间” |
| `-XX:InitialSurvivorRatio=*`N`*` | 设置年轻代中专门用于幸存者空间的空间量。 | 如果短生命周期对象频繁晋升到老年代，可以增加此值。 | “续期和幸存者空间” |
| `-XX:MinSurvivorRatio=*`N`*` | 设置年轻代中专用于幸存者空间的自适应空间量。 | 减少此值会减少幸存者空间的最大大小（反之亦然）。 | “续期和幸存者空间” |
| `-XX:TargetSurvivorRatio=*`N`*` | JVM 尝试保持在幸存者空间中的空闲空间量。 | 增加此值会减少幸存者空间的大小（反之亦然）。 | “续期和幸存者空间” |
| `-XX:InitialTenuringThreshold=*`N`*` | JVM 尝试保持对象在 survivor 空间中的初始 GC 周期数。 | 增加此数字以使对象在 survivor 空间中保持更长时间，尽管要注意 JVM 会对其进行调整。 | “Tenuring 和 Survivor Spaces” |
| `-XX:MaxTenuringThreshold=*`N`*` | JVM 尝试保持对象在 survivor 空间中的最大 GC 周期数。 | 增加此数字以使对象在 survivor 空间中保持更长时间；JVM 将在此值和初始阈值之间调整实际阈值。 | “Tenuring 和 Survivor Spaces” |
| `-XX:+DisableExplicitGC>` | 阻止对`System.gc()`的调用产生任何效果。 | 用于防止糟糕的应用程序显式执行 GC。 | “Causing 和 Disabling Explicit Garbage Collection” |
| `-XX:-AggressiveHeap` | 启用了一组对具有大量内存的机器以及运行单个具有大堆的 JVM 进行了“优化”的调整标志。 | 最好不要使用此标志，而是根据需要使用特定的标志。 | “AggressiveHeap” |

表 A-4。控制 GC 日志记录的标志

| 标志 | 作用 | 何时使用 | 另请参阅 |
| --- | --- | --- | --- |
| `-Xlog:gc*` | 控制 Java 11 中的 GC 日志记录。 | 应始终启用 GC 日志记录，即使在生产中也是如此。 与 Java 8 的以下一组标志不同，此标志控制 Java 11 GC 日志记录的所有选项； 有关将此选项映射到 Java 8 标志的文本，请参阅文本。 | “GC 工具” |
| `-verbose:gc` | 在 Java 8 中启用基本的 GC 日志记录。 | 应始终启用 GC 日志记录，但通常最好使用其他更详细的日志记录。 | “GC 工具” |
| `-Xloggc:`*`<path>`* | 在 Java 8 中，将 GC 日志定向到特殊文件而不是标准输出。 | 始终如此，以更好地保存日志中的信息。 | “GC 工具” |
| `-XX:+PrintGC` | 在 Java 8 中启用基本的 GC 日志记录。 | 应始终启用 GC 日志记录，但通常更详细的日志记录更好。 | “GC 工具” |
| `-XX:+PrintGCDetails` | 在 Java 8 中启用详细的 GC 日志记录。 | 始终如此，即使在生产中（日志记录开销很小）。 | “GC 工具” |
| `-XX:+PrintGCTimeStamps` | 在 Java 8 中，为 GC 日志中的每个条目打印相对时间戳。 | 始终如此，除非启用了日期时间戳。 | “GC 工具” |
| `-XX:+PrintGCDateStamps` | 在 Java 8 中为 GC 日志中的每个条目打印时间戳。 | 比时间戳的开销略大，但可能更容易处理。 | “GC 工具” |
| `-XX:+PrintReferenceGC` | 在 Java 8 中，在 GC 期间打印关于软引用和弱引用处理的信息。 | 如果程序大量使用这些引用，请添加此标志以确定它们对 GC 开销的影响。 | “软引用、弱引用及其他引用” |
| `-XX:+UseGCLogFileRotation` | 启用 GC 日志的轮转以节省文件空间在 Java 8 中。 | 在生产系统中，运行时间长达数周时，GC 日志可能会占用大量空间。 | “GC 工具” |
| `-XX:NumberOfGCLogFiles=*`N`*` | 当在 Java 8 中启用日志文件轮转时，指示要保留的日志文件数。 | 在生产系统中，运行时间长达数周时，GC 日志可能会占用大量空间。 | “GC 工具” |
| `-XX:GCLogFileSize=*`N`*` | 当在 Java 8 中启用日志文件轮转时，指示每个日志文件在轮转之前的大小。 | 在生产系统中，运行时间长达数周时，GC 日志可能会占用大量空间。 | “GC 工具” |

表 A-5\. 吞吐量收集器的标志

| 标志 | 功能 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| `-XX:MaxGCPauseMillis=*`N`*` | 给吞吐量收集器一个提示，暂停时间应该是多长；堆的大小动态调整以尝试达到该目标。 | 如果默认计算出的堆大小不符合应用程序目标，作为调优吞吐量收集器的第一步。 | “自适应和静态堆大小调优” |
| `-XX:GCTimeRatio=*`N`*` | 给吞吐量收集器一个提示，你愿意在 GC 中花费多少时间；堆的大小动态调整以尝试达到该目标。 | 如果默认计算出的堆大小不符合应用程序目标，作为调优吞吐量收集器的第一步。 | “自适应和静态堆大小调优” |

表 A-6\. G1 收集器的标志

| 标志 | 功能 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| `-XX:MaxGCPauseMillis=*`N`*` | 给 G1 收集器一个提示，暂停时间应该是多长；G1 算法会调整以尝试达到该目标。 | 作为调优 G1 收集器的第一步；增加此值以尝试防止 Full GC。 | “调优 G1 GC” |
| `-XX:ConcGCThreads=*`N`*` | 设置用于 G1 后台扫描的线程数。 | 当有大量 CPU 可用并且 G1 正在经历并发模式失败时。 | “调优 G1 GC” |
| `-XX:InitiatingHeapOccupancyPercent=*`N`*` | 设置 G1 后台扫描开始的阈值。 | 如果 G1 正在经历并发模式失败，请降低此值。 | “调优 G1 GC” |
| `-XX:G1MixedGCCountTarget=*`N`*` | 设置混合 GC 的次数，G1 尝试释放已标记为主要包含垃圾的区域。 | 如果 G1 经历并发模式失败，请降低此值；如果混合 GC 周期过长，请增加此值。 | “调优 G1 GC” |
| `-XX:G1MixedGCCountTarget=*`N`*` | 设置混合 GC 的次数，G1 尝试释放已标记为主要包含垃圾的区域。 | 如果 G1 经历并发模式失败，请降低此值；如果混合 GC 周期过长，请增加此值。 | “调优 G1 GC” |
| `-XX:G1HeapRegionSize=*`N`*` | 设置 G1 区域的大小。 | 对于非常大的堆或应用程序分配非常大的对象，请增加此值。 | “G1 GC 区域大小” |
| `-XX:+UseStringDeduplication` | 允许 G1 消除重复字符串。 | 适用于有大量重复字符串且国际化不可行的程序。 | “重复字符串和字符串国际化” |

表 A-7\. CMS 收集器标志

| 标志 | 功能 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| `-XX:CMSInitiating​OccupancyFraction``=*N*` | 确定 CMS 应在老年代后台扫描开始时刻。 | 当 CMS 经历并发模式失败时，降低此值。 | “理解 CMS 收集器” |
| `-XX:+UseCMSInitiating​OccupancyOnly` | 导致 CMS 仅使用 `CMSInitiatingOccupancyFraction` 来确定何时启动 CMS 后台扫描。 | 每当指定 `CMSInitiatingOccupancyFraction` 时。 | “理解 CMS 收集器” |
| `-XX:ConcGCThreads=*`N`*` | 设置用于 CMS 后台扫描的线程数。 | 当大量 CPU 可用且 CMS 经历并发模式失败时。 | “理解 CMS 收集器” |
| `-XX:+CMSIncrementalMode` | 以增量模式运行 CMS。 | 不再支持。 | N/A |

表 A-8\. 内存管理标志

| 标志 | 功能 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| `-XX:+HeapDumpOnOutOfMemoryError` | 在 JVM 抛出内存溢出错误时生成堆转储。 | 如果应用程序因堆空间或永久代导致内存溢出错误，请启用此标志，以便分析堆中的内存泄漏。 | “内存溢出错误” |
| `-XX:HeapDumpPath=<path>` | 指定自动生成堆转储时应写入的文件名。 | 若要指定除了 *java_pid<pid>.hprof* 之外的路径用于在内存溢出错误或 GC 事件时生成的堆转储，请使用此选项。 | “内存溢出错误” |
| `-XX:GCTimeLimit=<N>` | 指定 JVM 在执行太多 GC 周期时不抛出`OutOfMemoryException`的时间。 | 降低此值，以便在程序执行太多 GC 周期时，JVM 更早地抛出 OOM 异常。 | “内存不足错误” |
| `-XX:HeapFreeLimit=<N>` | 指定 JVM 必须释放的内存量，以防止抛出`OutOfMemoryException`。 | 降低此值，以便在程序执行太多 GC 周期时，JVM 更早地抛出 OOM 异常。 | “内存不足错误” |
| `-XX:SoftRefLRUPolicyMSPerMB=*`N`*` | 控制软引用在被使用后存活的时间。 | 在低内存条件下，缩短此值以更快地清理软引用。 | “软引用、弱引用和其他引用” |
| `-XX:MaxDirectMemorySize=*`N`*` | 控制通过`ByteBuffer`类的`allocateDirect()`方法分配多少本机内存。 | 如果要限制程序可以分配的直接内存量，考虑设置此标志。不再需要设置此标志来分配超过 64 MB 的直接内存。 | “本机 NIO 缓冲区” |
| `-XX:+UseLargePages` | 指示 JVM 从操作系统的大页系统中分配页面（如果适用）。 | 如果操作系统支持，此选项通常会提高性能。 | “大页” |
| `-XX:+StringTableSize=*`N`*` | 设置 JVM 用于保存国际化字符串的哈希表的大小。 | 如果应用程序执行大量的字符串国际化，则增加此值。 | “重复字符串和字符串国际化” |
| `-XX:+UseCompressedOops` | 模拟对象引用的 35 位指针。 | 对于小于 32 GB 的堆，默认是这个值；禁用它永远没有好处。 | “压缩 Oops” |
| `-XX:+PrintTLAB` | 在 GC 日志中打印关于 TLAB 的摘要信息。 | 在使用不支持 JFR 的 JVM 时，请确保 TLAB 分配工作效率。 | “线程本地分配缓冲区” |
| `-XX:TLABSize=*`N`*` | 设置 TLAB 的大小。 | 当应用程序在 TLAB 外执行大量分配时，使用此值来增加 TLAB 的大小。 | “线程本地分配缓冲区” |
| `-XX:-ResizeTLAB` | 禁用 TLAB 的调整大小功能。 | 每当指定`TLABSize`时，请确保禁用此标志。 | “线程本地分配缓冲区” |

表 A-9。本机内存跟踪的标志

| 标志 | 作用 | 使用时机 | 参见 |
| --- | --- | --- | --- |
| `-XX:NativeMemoryTracking=*X*` | 启用本机内存跟踪。 | 当需要查看 JVM 在堆外使用的内存时。 | “本机内存跟踪” |
| `-XX:+PrintNMTStatistics` | 在程序终止时打印本地内存跟踪统计信息。 | 当需要查看 JVM 在堆外使用的内存时使用。 | “本地内存跟踪” |

Table A-10\. 线程处理标志

| Flag | What it does | When to use it | See also |
| --- | --- | --- | --- |
| `-Xss<N>` | 设置线程的本机堆栈大小。 | 减小此大小以为 JVM 的其他部分提供更多内存。 | “调整线程堆栈大小” |
| `-XX:-BiasedLocking` | 禁用 JVM 的偏向锁定算法。 | 可以改善基于线程池的应用程序的性能。 | “偏向锁定” |

Table A-11\. JVM 杂项标志

| Flag | What it does | When to use it | See also |
| --- | --- | --- | --- |
| `-XX:+CompactStrings` | 在可能的情况下使用 8 位字符串表示（仅适用于 Java 11）。 | 默认；始终使用。 | “紧凑字符串” |
| `-XX:-StackTraceInThrowable` | 阻止每次抛出异常时收集堆栈跟踪。 | 在系统具有非常深的堆栈并且频繁抛出异常的情况下使用（且修复代码以减少异常抛出不可行时）。 | “异常” |
| `-Xshare` | 控制类数据共享。 | 使用此标志为应用程序代码创建新的 CDS 存档。 | “类数据共享” |

Table A-12\. Java Flight Recorder 标志

| Flag | What it does | When to use it | See also |
| --- | --- | --- | --- |
| `-XX:+FlightRecorder` | 启用 Java Flight Recorder。 | 始终建议启用 Flight Recorder，因为除非实际进行记录（在这种情况下，根据使用的功能，开销将有所不同，但仍然相对较小）。 | “Java Flight Recorder” |
| `-XX:+FlightRecorderOptions` | 设置通过命令行进行默认录制的选项（仅适用于 Java 8）。 | 控制如何为 JVM 进行默认录制。 | “Java Flight Recorder” |
| `-XX:+StartFlightRecorder` | 使用给定的 Flight Recorder 选项启动 JVM。 | 控制如何为 JVM 进行默认录制。 | “Java Flight Recorder” |
| `-XX:+UnlockCommercialFeatures` | 允许 JVM 使用商业（非开源）功能。 | 如果具有适当的许可证，则必须设置此标志才能在 Java 8 中启用 Java Flight Recorder。 | “Java Flight Recorder” |
