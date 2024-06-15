# 第十章。文件处理和 I/O

Java 自第一个版本以来就支持输入/输出（I/O）功能。然而，由于 Java 强烈追求平台独立性，早期版本的 I/O 功能强调可移植性而不是功能性。因此，它们并不总是易于使用。

我们将在本章后面看到原始 API 是如何被补充的——它们现在非常丰富，功能完备，并且非常易于开发。让我们从查看 Java I/O 的原始“经典”方法开始，而更现代的方法则在其之上增加了层次。

# 经典的 Java I/O

`File`类是 Java 原始文件 I/O 方式的基石。这种抽象化可以同时表示文件和目录，但在处理过程中有时会显得有些累赘，导致这样的代码：

```java
// Get a file object to represent the user's home directory
var homedir = new File(System.getProperty("user.home"));

// Create an object to represent a config file (should
// already be present in the home directory)
var f = new File(homedir, "app.conf");

// Check the file exists, really is a file, and is readable
if (f.exists() && f.isFile() && f.canRead()) {

  // Create a file object for a new configuration directory
  var configdir = new File(homedir, ".configdir");
  // And create it
  configdir.mkdir();

  // Finally, move the config file to its new home
  f.renameTo(new File(configdir, ".config"));
}
```

这展示了`File`类可能具有的一些灵活性，但也展示了抽象化带来的一些问题。它非常通用，因此需要大量的方法来查询`File`对象，以确定它实际代表什么以及其功能。

## 文件

`File`类上有大量的方法，但某些基本功能（特别是直接提供读取文件实际内容的方式）则没有，并且从未直接提供过。以下是`File`方法的快速总结：

```java
// Permissions management
boolean canX = f.canExecute();
boolean canR = f.canRead();
boolean canW = f.canWrite();

boolean ok;
ok = f.setReadOnly();
ok = f.setExecutable(true);
ok = f.setReadable(true);
ok = f.setWritable(false);

// Different views of the file's name
File absF = f.getAbsoluteFile();
File canF = f.getCanonicalFile();
String absName = f.getAbsolutePath();
String canName = f.getCanonicalPath();
String name = f.getName();
String pName = f.getParent();
URI fileURI = f.toURI(); // Create URI for File path

// File metadata
boolean exists = f.exists();
boolean isAbs = f.isAbsolute();
boolean isDir = f.isDirectory();
boolean isFile = f.isFile();
boolean isHidden = f.isHidden();
long modTime = f.lastModified(); // milliseconds since epoch
boolean updateOK = f.setLastModified(updateTime); // milliseconds
long fileLen = f.length();

// File management operations
boolean renamed = f.renameTo(destFile);
boolean deleted = f.delete();

// Create won't overwrite existing file
boolean createdOK = f.createNewFile();

// Temporary file handling
var tmp = File.createTempFile("my-tmp", ".tmp");
tmp.deleteOnExit();

// Directory handling
boolean createdDir = dir.mkdir(); // Non-recursive create only
String[] fileNames = dir.list();
File[] files = dir.listFiles();
```

`File`类还有一些方法并不完全适合于该抽象化。它们主要涉及对文件所在的文件系统进行询问（例如，询问可用的空闲空间）：

```java
long free = f.getFreeSpace();
long total = f.getTotalSpace();
long usable = f.getUsableSpace();

File[] roots = File.listRoots(); // all available Filesystem roots
```

## I/O 流

I/O 流抽象化（不要与处理 Java 8 集合 API 时使用的流混淆）已存在于 Java 1.0 中，作为处理来自磁盘或其他源的顺序字节流的一种方式。

这个 API 的核心是一对抽象类，`InputStream`和`OutputStream`。它们被广泛使用，事实上，“标准”输入和输出流，即称为`System.in`和`System.out`的流，属于这种类型。它们是`System`类的公共静态字段，甚至在最简单的程序中经常被使用：

```java
System.out.println("Hello World!");
```

流的特定子类，包括`FileInputStream`和`FileOutputStream`，可以用于操作文件中的单个字节，例如，通过计算 ASCII 97（小写字母*a*）在文件中出现的次数：

```java
try (var is = new FileInputStream("/Users/ben/cluster.txt")) {
  byte[] buf = new byte[4096];
  int len, count = 0;
  while ((len = is.read(buf)) > 0) {
    for (int i = 0; i < len; i = i + 1) {
      if (buf[i] == 97) {
        count = count + 1;
      }
    }
  }
  System.out.println("'a's seen: "+ count);
} catch (IOException e) {
  e.printStackTrace();
}
```

处理磁盘数据的这种方法可能缺乏一些灵活性——大多数开发人员都是以字符为单位思考，而不是以字节为单位。为了解决这个问题，这些流通常与更高级别的`Reader`和`Writer`类结合使用，提供字符流级别的交互，而不是由`InputStream`和`OutputStream`及其子类提供的低级字节流。

## 读者和写作者

通过转向以字符为基础的抽象层，而不是字节，开发人员面对的是一个更加熟悉的 API，它隐藏了许多字符编码、Unicode 等问题。

`Reader`和`Writer`类旨在覆盖字节流类，并消除对 I/O 流的低级处理的需要。它们有几个子类经常用于彼此叠加，例如：

+   `FileReader`

+   `BufferedReader`

+   `StringReader`

+   `InputStreamReader`

+   `FileWriter`

+   `PrintWriter`

+   `BufferedWriter`

要从文件中读取所有行并将它们打印出来，我们使用一个在`FileReader`之上叠加的`BufferedReader`，如下所示：

```java
try (var in = new BufferedReader(new FileReader(filename))) {
  String line;

  while((line = in.readLine()) != null) {
    System.out.println(line);
  }
} catch (IOException e) {
  // Handle FileNotFoundException, etc. here
}
```

如果我们需要从控制台而不是文件中读取行，则通常会使用`InputStreamReader`应用于`System.in`。让我们看一个例子，我们想从控制台读取输入行，但要将以特殊字符开头的输入行视为特殊命令（“元命令”），而不是普通文本。这是许多聊天程序（包括 IRC）的常见特性。我们将使用来自第九章的正则表达式来帮助我们：

```java
// Meta example: "#info username"
var SHELL_META_START = Pattern.compile("^#(\\w+)\\s*(\\w+)?");

try (var console =
      new BufferedReader(new InputStreamReader(System.in))) {
  String line;

  while((line = console.readLine()) != null) {
    // Check for special commands ("metas")
    Matcher m = SHELL_META_START.matcher(line);
    if (m.find()) {
      String metaName = m.group(1);
      String arg = m.group(2);
      doMeta(metaName, arg);
    } else {
      System.out.println(line);
    }
  }
} catch (IOException e) {
  // Handle FileNotFoundException, etc. here
}
```

要将文本输出到文件，我们可以使用以下代码：

```java
var f = new File(System.getProperty("user.home")
 + File.separator + ".bashrc");
try (var out =
      new PrintWriter(new BufferedWriter(new FileWriter(f)))) {
  out.println("## Automatically generated config file. DO NOT EDIT");
  // ...
} catch (IOException iox) {
  // Handle exceptions
}
```

这种较旧的 Java I/O 风格还有许多其他偶尔有用的功能。例如，要处理文本文件，`FilterInputStream`类经常很有用。或者对于希望以类似于经典“管道”I/O 方法进行通信的线程，提供了`PipedInputStream`，`PipedReader`及其写入对应物。

到目前为止，我们在本章中一直在使用“try-with-resources”（TWR）这种语言特性。这种语法在“try-with-resources 语句”中简要介绍过，但是结合像 I/O 这样的操作时，它才发挥出最大的潜力，并且它给旧的 I/O 风格带来了新生。

## 再探讨 try-with-resources

要充分利用 Java 的 I/O 功能，理解何时以及如何使用 TWR 至关重要。只要可能，代码就应该使用 TWR，这一点非常容易理解。

在 TWR 之前，资源必须手动关闭；资源之间复杂的交互导致有漏洞的泄漏资源的错误代码。

实际上，Oracle 的工程师们估计，初始 JDK 6 版本中 60%的资源处理代码是错误的。因此，即使是平台的作者也无法可靠地处理手动资源处理，那么所有新代码肯定都应该使用 TWR。

TWR 的关键在于一个新接口—`AutoCloseable`。这个接口是`Closeable`的直接超接口。它标志着一个必须自动关闭的资源，并且编译器将插入特殊的异常处理代码。

在 TWR 资源子句中，只能声明实现了`AutoCloseable`接口的对象，但开发人员可以声明所需数量的对象：

```java
try (var in = new BufferedReader(
                           new FileReader("profile"));
     var out = new PrintWriter(
                         new BufferedWriter(
                           new FileWriter("profile.bak")))) {
  String line;
  while((line = in.readLine()) != null) {
    out.println(line);
  }
} catch (IOException e) {
  // Handle FileNotFoundException, etc. here
}
```

这意味着资源自动限定于`try`块。这些资源（无论是可读还是可写的）会按照打开的相反顺序自动关闭，并且编译器会插入异常处理以考虑资源之间的依赖关系。

TWR 与其他语言和环境中的类似概念相关，例如 C++中的 RAII（资源获取即初始化）。然而，如终结部分所讨论的，TWR 仅限于块范围。这种轻微的限制是因为该功能由 Java 源代码编译器实现 —— 它在退出作用域时自动插入调用资源的`close()`方法的字节码（无论通过何种方式退出）。

因此，TWR 的整体效果更类似于 C#的`using`关键字，而不是 C++版本的 RAII。对于 Java 开发人员，理解 TWR 的最佳方式是“正确执行的终结”。正如在“Finalization”中所述，新代码不应直接使用终结机制，而应始终使用 TWR。较旧的代码应尽快重构为使用 TWR，因为它为资源处理代码提供了真正的实际好处。

## 经典 I/O 存在问题

即使引入了`try`-with-resources，`File`类及其相关类在执行标准 I/O 操作时仍存在一些问题。例如：

+   缺少常见操作的方法。

+   平台之间并不一致地处理文件名。

+   没有统一的文件属性模型（例如，建模读写访问）。

+   难以遍历未知的目录结构

+   没有平台或操作系统特定的功能。

+   不支持文件系统的非阻塞操作。

要解决这些缺点，Java 的 I/O 在几个重要版本中逐步演变。随着 Java 7 的发布，这种支持变得真正简单和高效。

# 现代 Java I/O

Java 7 引入了全新的 I/O API —— 通常称为 NIO.2 —— 几乎完全取代了原始的`File`方法进行 I/O 操作。

新的类位于`java.nio.file`包中，对于许多用例来说更加简单。API 由两个主要部分组成。第一个是称为`Path`的新抽象（可以视为表示文件位置，实际上可能存在也可能不存在）。第二部分是许多处理文件和文件系统的新便利和实用方法。这些方法作为`Files`类的静态方法提供。

## 文件

例如，当您使用新的`Files`功能时，基本的复制操作现在就像这样简单：

```java
var inputFile = new File("input.txt");
try (var in = new FileInputStream(inputFile)) {
  Files.copy(in, Path.of("output.txt"));
} catch(IOException ex) {
  ex.printStackTrace();
}
```

让我们快速浏览一些`Files`中的主要方法——它们的大多数操作都是很明显的。在许多情况下，这些方法有返回类型。我们已经省略了处理这些内容，因为它们除了人为示例和复制等效的 C 代码行为外，很少有用：

```java
Path source, target;
Attributes attr;
Charset cs = StandardCharsets.UTF_8;

// Creating files
//
// Example of path --> /home/ben/.profile
// Example of attributes --> rw-rw-rw-
Files.createFile(target, attr);

// Deleting files
Files.delete(target);
boolean deleted = Files.deleteIfExists(target);

// Copying/moving files
Files.copy(source, target);
Files.move(source, target);

// Utility methods to retrieve information
long size = Files.size(target);

FileTime fTime = Files.getLastModifiedTime(target);
System.out.println(fTime.to(TimeUnit.SECONDS));

Map<String, ?> attrs = Files.readAttributes(target, "*");
System.out.println(attrs);

// Methods to deal with file types
boolean isDir = Files.isDirectory(target);
boolean isSym = Files.isSymbolicLink(target);

// Methods to deal with reading and writing
List<String> lines = Files.readAllLines(target, cs);
byte[] b = Files.readAllBytes(target);

var br = Files.newBufferedReader(target, cs);
var bwr = Files.newBufferedWriter(target, cs);

var is = Files.newInputStream(target);
var os = Files.newOutputStream(target);
```

`Files`上的一些方法提供了传递可选参数的机会，以提供额外的（可能是实现特定的）操作行为。

这里的一些 API 选择会产生偶尔令人讨厌的行为。例如，默认情况下，复制操作不会覆盖现有文件，因此我们需要指定此行为作为复制选项：

```java
Files.copy(Path.of("input.txt"), Path.of("output.txt"),
           StandardCopyOption.REPLACE_EXISTING);
```

`StandardCopyOption`是一个实现`CopyOption`接口的枚举。这也被`LinkOption`实现。因此，`Files.copy()`可以接受任意数量的`LinkOption`或`StandardCopyOption`参数。`LinkOption`用于指定如何处理符号链接（当然，前提是底层操作系统支持符号链接）。

## Path

`Path`是用于在文件系统中定位文件的类型。它表示一个路径，其特点是：

+   系统相关

+   分层的

+   由一系列路径元素组成

+   假设性的（可能尚不存在，或已被删除）

因此，它与`File`基本不同。特别是，系统依赖通过`Path`作为接口而不是类来体现，这使得不同的文件系统提供者可以各自实现`Path`接口，并提供系统特定的功能，同时保留总体抽象。

`Path`的元素包括可选的根组件，它标识了此实例所属的文件系统层次结构。请注意，例如，相对`Path`实例可能没有根组件。除了根之外，所有的`Path`实例都有零个或多个目录名称和一个名称元素。

名称元素是距离目录层次结构根部最远的元素，并表示文件或目录的名称。`Path`可以被认为是由特殊分隔符或分隔符连接的路径元素组成。

`Path`是一个抽象概念；它不一定与任何物理文件路径绑定。这使得我们可以轻松地讨论尚不存在的文件的位置。`Path`接口提供了用于创建`Path`实例的静态工厂方法。

###### 注意

当 NIO.2 在 Java 7 中引入时，接口上不支持静态方法，因此引入了`Paths`类来保存工厂方法。到了 Java 17，推荐使用`Path`接口的方法，而`Paths`类可能会在未来被弃用。

`Path` 提供了两个 `of()` 方法用于创建 `Path` 对象。通常版本使用一个或多个 `String` 实例，并使用默认的文件系统提供者。`URI` 版本利用了 NIO.2 的能力，可以插入额外提供定制文件系统的提供者。这是一个高级用法，有兴趣的开发人员应该查阅主要文档。让我们看一些如何使用 `Path` 的简单示例：

```java
var p = Path.of("/Users/ben/cluster.txt");
var p2 = Path.of(new URI("file:///Users/ben/cluster.txt"));
System.out.println(p2.equals(p));

File f = p.toFile();
System.out.println(f.isDirectory());

Path p3 = f.toPath();
System.out.println(p3.equals(p));
```

该示例还展示了 `Path` 和 `File` 对象之间的简单互操作性。`Path` 增加了 `toFile()` 方法，而 `File` 增加了 `toPath()` 方法，允许开发人员在两个 API 之间轻松切换，并允许通过简单的方法重构基于 `File` 的代码内部，改为使用 `Path`。

我们还可以使用 `Files` 类提供的一些有用的“桥接”方法。例如，通过提供方便的方法来打开 `Writer` 对象到指定的 `Path` 位置：

```java
var logFile = Path.of("/tmp/app.log");
try (var writer =
       Files.newBufferedWriter(logFile, StandardCharsets.UTF_8,
                               StandardOpenOption.WRITE,
                               StandardOpenOption.CREATE)) {
  writer.write("Hello World!");
  // ...
} catch (IOException e) {
  // ...
}
```

我们使用了 `StandardOpenOption` 枚举，它提供了类似于复制选项的能力，但用于打开新文件的情况。我们同时提供了 `WRITE` 和 `CREATE`，所以如果文件不存在，它将被创建；否则，我们只是打开它以进行额外的写入操作。

在这个示例用例中，我们已经使用了 `Path` API 来：

+   创建对应于新文件的 `Path`

+   使用 `Files` 类来创建这个新文件

+   打开一个 `Writer` 到该文件

+   向该文件写入

+   在完成时自动关闭它

在我们的下一个示例中，我们将在此基础上操作 JAR 文件，将其作为一个独立的 `FileSystem` 来修改，直接向 JAR 中添加文件。请记住，JAR 文件实际上只是 ZIP 文件，因此这种技术也适用于 *.zip* 归档文件：

```java
var tempJar = Path.of("sample.jar");
try (var workingFS =
      FileSystems.newFileSystem(tempJar)) {

  Path pathForFile = workingFS.getPath("/hello.txt");
  Files.write(pathForFile,
              List.of("Hello World!"),
              Charset.defaultCharset(),
              StandardOpenOption.WRITE, StandardOpenOption.CREATE);
}
```

这显示了我们如何创建一个 `FileSystem` 对象，以便创建引用 jar 内文件的 `Path` 对象，通过 `getPath()` 方法。这使得开发人员基本上可以将 `FileSystem` 对象视为黑匣子：它们通过服务提供者接口（SPI）机制自动创建。

要查看您的计算机上可用的文件系统，您可以运行类似于以下的代码：

```java
for (FileSystemProvider f : FileSystemProvider.installedProviders()) {
    System.out.println(f.toString());
}
```

`Files` 类还提供了处理临时文件和目录的方法，这是一个令人惊讶地常见的用例（也可能是安全漏洞的源头）。例如，让我们看看如何从类路径中加载资源文件，将其复制到新创建的临时目录，然后安全地清理临时文件（使用在线书籍资源中提供的 `Reaper` 类）：

```java
Path tmpdir = Files.createTempDirectory(Path.of("/tmp"), "tmp-test");
try (InputStream in =
      FilesExample.class.getResourceAsStream("/res.txt")) {
    Path copied = tmpdir.resolve("copied-resource.txt");
    Files.copy(in, copied, StandardCopyOption.REPLACE_EXISTING);
    // ... work with the copy
}
// Clean up when done...
Files.walkFileTree(tmpdir, new Reaper());
```

Java 原始 I/O API 的一个批评是缺乏对本地和高性能 I/O 的支持。在 Java 1.4 中首次添加了解决方案，即 Java 新 I/O (NIO) API，并在后续 Java 版本中进行了改进。

# NIO 通道和缓冲区

NIO 缓冲区是高性能 I/O 的低级抽象。它们提供了一个特定原始类型的线性序列元素的容器。我们将在示例中使用`ByteBuffer`（最常见的情况）。

## ByteBuffer

这是一系列字节，可以概念上看作是与`byte[]`工作的性能关键替代品。为了获得最佳性能，`ByteBuffer`提供了支持直接处理 JVM 运行平台的本地能力。

这种方法称为*直接缓冲区*情况，尽可能绕过 Java 堆。直接缓冲区在本机内存中分配，而不是在标准的 Java 堆上，并且不会像常规的在堆 Java 对象那样受垃圾收集的影响。

要获取直接的`ByteBuffer`，调用`allocateDirect()`工厂方法。也提供了一个在堆上的版本`allocate()`，但在实践中这不常用。

获取字节缓冲区的第三种方法是使用`wrap()`一个现有的`byte[]` —— 这将提供一个在堆上的缓冲区，用于提供对底层字节的更面向对象的视图：

```java
var b = ByteBuffer.allocateDirect(65536);
var b2 = ByteBuffer.allocate(4096);

byte[] data = {1, 2, 3};
ByteBuffer b3 = ByteBuffer.wrap(data);
```

字节缓冲区都是关于对字节的低级访问。这意味着开发人员必须手动处理细节 —— 包括处理字节的字节顺序和 Java 整数原始类型的有符号性：

```java
b.order(ByteOrder.BIG_ENDIAN);

int capacity = b.capacity();
int position = b.position();
int limit = b.limit();
int remaining = b.remaining();
boolean more = b.hasRemaining();
```

要将数据输入或输出到缓冲区，我们有两种操作类型 —— 单值类型，读取或写入单个值，以及批量操作，接受一个`byte[]`或`ByteBuffer`并作为单个操作操作（可能大量的）值。我们期望从批量操作中获得性能提升：

```java
b.put((byte)42);
b.putChar('x');
b.putInt(0xc001c0de);

b.put(data);
b.put(b2);

double d = b.getDouble();
b.get(data, 0, data.length);
```

单值形式也支持用于在缓冲区内进行绝对定位的形式：

```java
b.put(0, (byte)9);
```

缓冲区是内存中的抽象。要影响外部世界（例如文件或网络），我们需要使用`java.nio.channels`包中的`Channel`。通道代表可以支持读取或写入操作的实体连接。文件和套接字是通道的常见示例，但我们可以考虑用于低延迟数据处理的自定义实现。

通道在创建时处于打开状态，可以随后关闭。一旦关闭，就不能重新打开。通常通道要么可读要么可写，但不能同时。理解通道的关键在于：

+   从通道读取将字节放入缓冲区

+   向通道写入从缓冲区中获取的字节

例如，假设我们有一个大文件，想要以 16M 块进行校验和计算：

```java
FileInputStream fis = getSomeStream();
boolean fileOK = true;

try (FileChannel fchan = fis.getChannel()) {
  var buffy = ByteBuffer.allocateDirect(16 * 1024 * 1024);
  while(fchan.read(buffy) != -1 || buffy.position() > 0 || fileOK) {
    fileOK = computeChecksum(buffy);
    buffy.compact();
  }
} catch (IOException e) {
  System.out.println("Exception in I/O");
}
```

这将尽可能使用本机 I/O，并避免在 Java 堆上复制大量字节。如果`computeChecksum()`方法已经实现良好，那么这可能是一个非常高效的实现。

## 映射的字节缓冲区

这些是一种直接字节缓冲区，包含一个内存映射文件（或其一部分）。它们是从 `FileChannel` 对象创建的，但请注意，与 `MappedByteBuffer` 对应的 `File` 对象在内存映射操作后不能再使用，否则会抛出异常。为了减少这种情况，我们再次使用 `try`-with-resources，以紧密地限定对象的作用域：

```java
try (var raf =
  new RandomAccessFile(new File("input.txt"), "rw");
     FileChannel fc = raf.getChannel();) {

  MappedByteBuffer mbf =
    fc.map(FileChannel.MapMode.READ_WRITE, 0, fc.size());
  var b = new byte[(int)fc.size()];
  mbf.get(b, 0, b.length);
  for (int i = 0; i < fc.size(); i = i + 1) {
    b[i] = 0; // Won't be written back to the file, we're a copy
  }
  mbf.position(0);
  mbf.put(b); // Zeros the file
}
```

即使使用缓冲区，Java 对于大型 I/O 操作（例如在文件系统之间传输 10G）也存在一些限制，这些操作在单个线程上同步执行。在 Java 7 之前，这类操作通常通过编写自定义的多线程代码来完成，并管理单独的线程执行后台复制。让我们继续看看 JDK 7 中新增的异步 I/O 特性。

# 异步 I/O

异步功能的关键在于 `Channel` 的新子类，它们可以处理需要交给后台线程的 I/O 操作。相同的功能可以应用于大型、长时间运行的操作以及其他几种用例。

在本节中，我们将专注于文件 I/O 的 `AsynchronousFileChannel`，但还有几个其他异步通道需要了解。我们将在本章末尾查看异步套接字。我们将看到：

+   用于文件 I/O 的 `AsynchronousFileChannel`

+   用于客户端套接字 I/O 的 `AsynchronousSocketChannel`

+   用于异步套接字接受传入连接的 `AsynchronousServerSocketChannel`

与异步通道交互的有两种不同的方式 — `Future` 风格和回调风格。

## 基于 Future 的风格

对 `Future` 接口的全面讨论将使我们深入了解 Java 并发的细节。然而，对于本章的目的，它可以被视为一个可能已经完成或尚未完成的任务。它有两个关键方法：

`isDone()`

返回一个布尔值，指示任务是否已完成。

`get()`

返回结果。如果完成，则立即返回。如果未完成，则阻塞直到完成。

让我们看一个示例程序，异步读取一个大文件（可能达到 100 Mb）：

```java
try (var channel =
         AsynchronousFileChannel.open(Path.of("input.txt"))) {
  var buffer = ByteBuffer.allocateDirect(1024 * 1024 * 100);
  Future<Integer> result = channel.read(buffer, 0);

  while(!result.isDone()) {
    // Do some other useful work....
  }

  System.out.println("Bytes read: " + result.get());
}
```

## 基于回调的风格

异步 I/O 的回调风格基于 `CompletionHandler`，它定义了两个方法 `completed()` 和 `failed()`，在操作成功或失败时将回调调用。

如果您希望在异步 I/O 中立即收到事件通知，则此风格非常有用 — 例如，如果有大量的 I/O 操作正在进行中，但任何单个操作的失败并不一定是致命的：

```java
byte[] data = {2, 3, 5, 7, 11, 13, 17, 19, 23};
ByteBuffer buffy = ByteBuffer.wrap(data);

CompletionHandler<Integer,Object> h =
  new CompletionHandler<>() {
    public void completed(Integer written, Object o) {
      System.out.println("Bytes written: " + written);
    }

    public void failed(Throwable x, Object o) {
      System.out.println("Asynch write failed: "+ x.getMessage());
    }
  };

try (var channel =
       AsynchronousFileChannel.open(Path.of("primes.txt"),
          StandardOpenOption.CREATE, StandardOpenOption.WRITE)) {

  channel.write(buffy, 0, null, h);

  // Give the CompletionHandler time to run before foreground exit
  Thread.sleep(1000);
}
```

`AsynchronousFileChannel` 对象与后台线程池相关联，因此 I/O 操作可以继续进行，而原始线程可以继续进行其他任务。

###### 注意

`CompletionHandler` 接口有两个抽象方法，而不是一个，所以它不可以是 lambda 表达式的目标类型，遗憾地。

默认情况下，这使用由运行时提供的托管线程池。如果需要，可以创建一个由应用程序管理的线程池（通过`AsynchronousFileChannel.open()`的重载形式），但这很少是必要的。

最后，为了完整起见，让我们简要介绍一下 NIO 对多路复用 I/O 的支持。这使得单个线程能够管理多个通道，并检查这些通道以查看哪些已准备好进行读取或写入。支持这些功能的类位于`java.nio.channels`包中，包括`SelectableChannel`和`Selector`。

当编写需要高可伸缩性的高级应用程序时，这些非阻塞的多路复用技术可以非常有用，但是在本书的讨论范围之外。通常情况下，非阻塞 API 应仅用于高级用例，当高性能或其他非功能性要求要求时。

## 监控服务和目录搜索

我们将考虑的最后一类异步服务是监视目录或访问目录（或树）。观察服务通过观察目录中发生的一切来操作，例如文件的创建或修改：

```java
try {
  var watcher = FileSystems.getDefault().newWatchService();

  var dir = FileSystems.getDefault().getPath("/home/ben");
  dir.register(watcher,
                StandardWatchEventKinds.ENTRY_CREATE,
                StandardWatchEventKinds.ENTRY_MODIFY,
                StandardWatchEventKinds.ENTRY_DELETE);

  while(!shutdown) {
    WatchKey key = watcher.take();
    for (WatchEvent<?> event: key.pollEvents()) {
      Object o = event.context();
      if (o instanceof Path) {
        System.out.println("Path altered: "+ o);
      }
    }
    key.reset();
  }
}
```

相比之下，目录流提供了对单个目录中当前所有文件的视图。例如，要列出所有 Java 源文件及其大小（以字节为单位），我们可以使用如下代码：

```java
try(DirectoryStream<Path> stream =
    Files.newDirectoryStream(Path.of("/opt/projects"), "*.java")) {
  for (Path p : stream) {
    System.out.println(p +": "+ Files.size(p));
  }
}
```

此 API 的一个缺点是，它仅根据通配符语法返回匹配的元素，有时不够灵活。我们可以通过使用`Files.find()`和`Files.walk()`方法进一步处理通过目录的递归遍历获取的每个元素：

```java
var homeDir = Path.of("/Users/ben/projects/");
Files.find(homeDir, 255,
  (p, attrs) -> p.toString().endsWith(".java"))
     .forEach(q -> {System.out.println(q.normalize());});
```

还可以进一步构建基于`java.nio.file`中的`FileVisitor`接口的高级解决方案，但这要求开发人员实现接口的所有四个方法，而不仅仅是像这里所做的单个 lambda 表达式。

在本章的最后一部分，我们将讨论 Java 的网络支持和使其成为可能的核心 JDK 类。

# 网络

Java 平台提供了对大量标准网络协议的访问，这使得编写简单的网络应用程序非常容易。Java 网络支持的核心位于`java.net`包中，通过`javax.net`（特别是`javax.net.ssl`）提供了额外的可扩展性，所有这些都在模块`java.base`中。

构建应用程序最简单的协议之一是超文本传输协议（[HTTP](https://zh.wikipedia.org/wiki/超文本传输协议)），这是 Web 的基本通信协议。

## HTTP

HTTP 是 Java 支持的最常见和流行的高级网络协议。它是一个非常简单的协议，在标准 TCP/IP 协议栈的基础上实现。它可以运行在任何网络端口上，但通常在加密的 TLS（称为 HTTPS）上使用端口 443 或者在未加密时使用端口 80。如今，应尽可能在所有地方默认使用 HTTPS。

Java 有两个单独的处理 HTTP 的 API —— 其中一个可以追溯到平台早期的日子，另一个是全面支持的 Java 11 新 API。

为了完整起见，让我们快速看一下旧 API。在这个 API 中，`URL` 是关键类 —— 它支持 `http://`、`ftp://`、`file://` 和 `https://` 形式的 URL。它非常易于使用，Java HTTP 支持的最简单示例是下载特定 URL：

```java
var url = new URL("http://www.google.com/");
try (InputStream in = url.openStream()) {
  Files.copy(in, Path.of("output.txt"));
} catch(IOException ex) {
  ex.printStackTrace();
}
```

对于更低级别的控制，包括请求和响应的元数据，我们可以使用 `URLConnection` 来实现类似以下的操作：

```java
try {
  URLConnection conn = url.openConnection();

  String type = conn.getContentType();
  String encoding = conn.getContentEncoding();
  Date lastModified = new Date(conn.getLastModified());
  int len = conn.getContentLength();
  InputStream in = conn.getInputStream();
} catch (IOException e) {
  // Handle exception
}
```

HTTP 定义了“请求方法”，即客户端可以在远程资源上执行的操作。这些方法包括 GET、POST、HEAD、PUT、DELETE、OPTIONS 和 TRACE。

每个方法有稍微不同的用法，例如：

+   GET 应仅用于检索文档，并且*永远*不应执行任何副作用。

+   HEAD 与 GET 类似，但不返回主体 —— 如果程序想通过标头快速检查 URL 是否已更改，则此方法很有用。

+   当我们需要将数据发送到服务器进行处理时使用 POST。

默认情况下，Java 使用 GET 方法，但它提供了一种使用其他方法构建更复杂应用程序的方式；然而，这样做比较复杂。在下一个示例中，我们使用 Postman 提供的 echo 函数来返回我们发布的数据的视图：

```java
var url = new URL("https://postman-echo.com/post");
var encodedData = URLEncoder.encode("q=java", "ASCII");
var contentType = "application/x-www-form-urlencoded";

var conn = (HttpURLConnection) url.openConnection();
conn.setInstanceFollowRedirects(false);
conn.setRequestMethod("POST");
conn.setRequestProperty("Content-Type", contentType );
conn.setRequestProperty("Content-Length",
  String.valueOf(encodedData.length()));

conn.setDoOutput(true);
OutputStream os = conn.getOutputStream();
os.write( encodedData.getBytes() );

int response = conn.getResponseCode();
if (response == HttpURLConnection.HTTP_MOVED_PERM
    || response == HttpURLConnection.HTTP_MOVED_TEMP) {
  System.out.println("Moved to: "+ conn.getHeaderField("Location"));
} else {
  try (InputStream in = conn.getInputStream()) {
    Files.copy(in, Path.of("postman.txt"),
                StandardCopyOption.REPLACE_EXISTING);
  }
}
```

注意，我们需要将查询参数发送到请求体中并在发送前进行编码。我们还需要禁用 HTTP 重定向的跟随，并手动处理来自服务器的任何重定向。这是由于 `HttpURLConnection` 类在处理 POST 请求的重定向时存在限制。

旧 API 明显显露其年龄，事实上仅实现了 HTTP 标准的 1.0 版本，非常低效且被认为是过时的。作为替代，现代 Java 程序可以使用新 API，这是为了支持新的 HTTP/2 协议而添加的结果。自 Java 11 开始，它已经作为 `java.net.http` 模块的完全支持部分提供。让我们看一个使用新 API 的简单示例：

```java
        var client = HttpClient.newBuilder().build();
        var uri = new URI("https://www.oreilly.com");
        var request = HttpRequest.newBuilder(uri).build();

        var response = client.send(request,
                ofString(Charset.defaultCharset()));
        var body = response.body();
        System.out.println(body);
```

请注意，该 API 设计为可扩展，例如通过 `HttpResponse.BodySubscriber` 接口来实现自定义处理。该接口还可以无缝隐藏 HTTP/2 和旧版 HTTP/1.1 协议之间的差异，这意味着 Java 应用程序能够在 Web 服务器采用新版本时进行优雅的迁移。

让我们继续看网络堆栈中更底层的下一层，传输控制协议（TCP）。

## TCP

TCP 是互联网上可靠网络传输的基础。它确保网页和其他互联网流量以完整和可理解的状态传输。从网络理论的角度来看，使 TCP 能够作为互联网流量的“可靠性层”的协议特性包括：

基于连接

数据属于单个逻辑流（即连接）。

保证有序交付

直到数据包到达为止，数据包将被重新发送。

经过错误检查

网络传输引起的损坏将被自动检测和修复。

TCP 是双向（或双向）通信通道，并使用特殊的编号方案（TCP 序列号）来确保通信流的两侧保持同步。为了支持同一网络主机上的多个不同服务，TCP 使用端口号来标识服务，并确保发送到一个端口的流量不会进入另一个端口。

在 Java 中，TCP 由`Socket`和`ServerSocket`类表示。它们用于提供能力，分别作为连接的客户端和服务器端—这意味着 Java 既可用于连接到网络服务，也可作为实现新服务的语言。

###### 注意

Java 的原始套接字支持在 Java 13 中重新实现，没有 API 更改。经典套接字 API 现在与更现代的 NIO 基础结构共享代码，并且将继续有效地运行到未来。

举例来说，让我们考虑重新实现 HTTP 1.1。这是一个相对简单的基于文本的协议。我们需要实现连接的双方，所以让我们从基于 TCP 套接字的 HTTP 客户端开始。为了完成这个任务，我们实际上需要实现 HTTP 协议的细节，但我们有一个优势，那就是我们可以完全控制 TCP 套接字。

我们将需要从客户端套接字读取和写入，并且我们将按照 HTTP 标准（称为 RFC 2616，并使用显式的行结束语法）构造实际的请求行。生成的客户端代码将类似于以下内容：

```java
var hostname = "www.example.com";
int port = 80;
var filename = "/index.xhtml";

try (var sock = new Socket(hostname, port);
  var from = new BufferedReader(
      new InputStreamReader(sock.getInputStream()));
  var to = new PrintWriter(
      new OutputStreamWriter(sock.getOutputStream())); ) {

  // The HTTP protocol
  to.print("GET " + filename +
    " HTTP/1.1\r\nHost: "+ hostname +"\r\n\r\n");
  to.flush();

  for (String l = null; (l = from.readLine()) != null; )
    System.out.println(l);
}
```

在服务器端，我们需要接收可能的多个传入连接。为了处理这一点，我们将启动一个主服务器循环，然后使用`accept()`从操作系统获取新连接。然后快速将新连接传递给一个单独的处理程序类，以便主服务器循环可以继续监听新连接。这段代码比客户端情况复杂一些：

```java
// Handler class
private static class HttpHandler implements Runnable {
  private final Socket sock;
  HttpHandler(Socket client) { this.sock = client; }

  public void run() {
    try (var in =
           new BufferedReader(
             new InputStreamReader(sock.getInputStream()));
         var out =
           new PrintWriter(
             new OutputStreamWriter(sock.getOutputStream())); ) {
      out.print("HTTP/1.0 200\r\nContent-Type: text/plain\r\n\r\n");
      String line;
      while((line = in.readLine()) != null) {
        if (line.length() == 0) break;
        out.println(line);
      }
    } catch(Exception e) {
      // Handle exception
    }
  }
}

// Main server loop
public static void main(String[] args) {
  try {
    var port = Integer.parseInt(args[0]);

    ServerSocket ss = new ServerSocket(port);
    while (true) {
      Socket client = ss.accept();
      var handler = new HTTPHandler(client);
      new Thread(handler).start();
    }
  } catch (Exception e) {
    // Handle exception
  }
}
```

当设计一个应用程序通过 TCP 进行通信的协议时，有一个简单而深刻的网络架构原则，即著名的波斯特尔定律（以互联网之父之一乔恩·波斯特尔命名），你应该始终牢记。它有时被表述为：“在发送时要严格，接收时要宽松。”这个简单的原则意味着即使在网络系统中存在相当不完美的实现，通信仍然可以广泛可能。

波斯特尔定律与协议应尽可能简单的一般原则结合起来（有时被称为 KISS 原则），将使得开发者实现基于 TCP 的通信任务比起其他方式要容易得多。

TCP 下面是互联网的通用运输协议——互联网协议（IP）本身。

## IP

IP，作为“最低公共分母”传输，为实际从点 A 到点 B 传输字节的物理网络技术提供了一个有用的抽象。

不同于 TCP，IP 数据包的传递并不保证，可能会被沿途任何过载系统丢弃。IP 数据包确实有一个目的地，但通常没有路由数据——这是沿途（可能是多个不同的）物理传输的责任来实际交付数据。

在 Java 中可以创建基于单个 IP 数据包（或带有 UDP 头部而不是 TCP 的数据包）的“数据报”服务，但这通常只有在需要极低延迟的应用程序时才需要。Java 使用类`DatagramSocket`来实现此功能，尽管少数开发人员可能永远不需要深入到这个网络栈的层次。

最后，值得注意的是，当前在互联网上使用的寻址方案正在飞行中发生一些变化。目前主导的 IP 版本是 IPv4，其拥有 32 位的可能网络地址空间。现在这个空间已经非常紧张，已经部署了各种缓解技术来处理这种枯竭。

下一个 IP 版本（IPv6）正在逐步推出，但尚未完全被接受，也尚未取代 IPv4，尽管向它成为标准的稳步进展正在持续。截至目前，IPv6 流量约占互联网流量的 35%，并且稳步增长。在未来的 10 年内，IPv6 有可能在流量量上超过 IPv4，并且低级网络将需要适应这个根本性的新版本。

然而，对于 Java 程序员来说，好消息是该语言和平台多年来一直在为 IPv6 和其引入的变化提供良好支持。与其他许多语言相比，IPv4 到 IPv6 的过渡可能对 Java 应用程序来说会更加平稳和少问题。

# 摘要

在这一章中，我们介绍了 Java SDK 提供的文件处理、I/O 和网络功能。然而，并不是所有这些功能都被同等频繁地使用。核心的文件处理类（特别是`Path`和 NIO.2 的其他部分）被 Java 开发者经常使用，而更高级的功能则较少遇到。

网络库的情况有所不同。了解这些功能很好，但它们相当基础。在实践中，通常会使用第三方提供的更高级别的库（例如 Netty）。唯一的例外是，Java 开发者可以相对经常遇到的低级别 JDK 网络库是`java.net.http`中的新 HTTP 库。

现在让我们来了解一些 Java 的关键动态特性——类加载和反射——这些强大的技术允许代码在运行时以编译时未知的方式被发现、加载和执行。
