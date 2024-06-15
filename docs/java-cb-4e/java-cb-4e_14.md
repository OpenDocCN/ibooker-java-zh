# 第十四章：处理 JSON 数据

# 14.0 简介

JSON，或 JavaScript 对象表示法，具有以下特点：

+   一个简单、轻量级的数据交换格式。

+   一个比 XML 更简单、更轻的替代方案。

+   使用`println()`或几个 API 之一轻松生成。

+   在所有网页浏览器中直接被 JavaScript 解析器识别。

+   支持各种常见语言的附加框架（Java、C/C++、Perl、Ruby、Python、Lua、Erlang、Haskell 等等）；在[主页](http://json.org)上列出了一个包括 Java 二十多个解析器的非常长的支持语言列表。

一个简单的 JSON 消息可能是这样的：

*json/src/main/resources/json/softwareinfo.json/*

```java
{
  "name": "robinparse",
  "version": "1.2.3",
  "description": "Another Parser for JSON",
  "className": "RobinParse",
  "contributors": [
        "Robin Smythe",
        "Jon Jenz",
        "Jan Ardann"
    ]
}
```

如您所见，语法简单、可嵌套且适合人类检查。

[JSON 主页](http://json.org)提供了 JSON 语法的简明总结。有两种结构：JSON 对象（映射）和 JSON 数组（列表）。JSON 对象是一组名称和值对，可以表示为`java.util.Map` *或* Java 对象的属性。例如，2019 年 4 月 1 日的`LocalDate`对象的字段可以表示为：

```java
{
	"year": 2019,
	"month": 4,
	"day" : 1
}
```

JSON 数组是有序列表，在 Java 中表示为数组或`java.util.List`。两个日期的列表可能如下所示：

```java
{
	[{
		"year": 2019,
		"month": 4,
		"day" : 1
	},{
		"year": 2019,
		"month": 5,
		"day" : 15
	}]
}
```

JSON 是自由格式的，因此前述内容也可以写成以下形式，虽然会失去人类可读性，但不会失去信息或功能性：

```java
{[{"year":2019,"month":4,"day":1},{"year":2019,"month":5,"day":15}]}
```

我相信已经为 JSON 编写了数百个解析器。在 Java 的世界中，一些我能想到的包括以下几个：

`stringtree.org`

非常小且轻量级

`json.org parser`

因其免费且具有良好的域名而广泛使用

`jackson.org parser`

因其非常强大而广泛使用，与 Spring Framework、JBoss RESTEasy 和 Wildfly 一起使用

`javax.json`

Oracle 官方但目前仅限 EE 标准

本章展示了使用前面列出的各种 API 处理 JSON 数据的几种方法。官方的`javax.json`API 只包含在 Java EE 中，而不包含在 Java SE 中，因此在客户端很少见到使用。这个 API 使用了与`org.json`API 一些相同的名称，但不足以被认为是兼容的。

因为这是一本面向客户端 Java 开发者的书籍，因此将不会讨论直接在服务器生成的基于浏览器的 JavaScript 中处理 JSON 的能力，尽管这在构建企业应用程序时非常有用。

# 14.1 直接生成 JSON

## 问题

你想生成 JSON 而不需要使用 API。

## 解决方案

获取您想要的数据，并根据需要使用`println()`或`String.format()`。

## 讨论

如果你细心的话，你可以自己生成 JSON 数据。对于极为简单的情况，你可以使用`PrintWriter.println()`或者`String.format()`。然而，对于大量数据，通常最好使用其中一个 API。

此代码打印了`LocalTime`对象的年份、月份和日期（请参阅 Recipe 6.1）。 一些 JSON 格式化被委托给 `toJson()` 方法：

```java
/**
 * Convert an object to JSON, not using any JSON API.
 * BAD IDEA - should use an API!
 */
public class LocalDateToJsonManually {

    private static final String OPEN = "{";
    private static final String CLOSE = "}";

    public static void main(String[] args) {
        LocalDate dNow = LocalDate.now();
        System.out.println(toJson(dNow));
    }

    public static String toJson(LocalDate dNow) {
        StringBuilder sb = new StringBuilder();
        sb.append(OPEN).append("\n");
        sb.append(jsonize("year", dNow.getYear()));
        sb.append(jsonize("month", dNow.getMonth()));
        sb.append(jsonize("day", dNow.getDayOfMonth()));
        sb.append(CLOSE).append("\n");
        return sb.toString();
    }

    public static String jsonize(String key, Object value) {
        return String.format("\"%s\": \"%s\",\n", key, value);
    }
}
```

当然，这是一个极端琐碎的例子。 对于任何更复杂的情况，或者对于必须解析 JSON 对象的常见情况，使用其中一个框架将更容易使您的神经放松。

# 14.2 使用 Jackson 解析和写入 JSON

## 问题

您希望使用完整功能的 JSON API 读取和/或写入 JSON。

## 解决方案

使用 Jackson，完整的 JSON API。

## 讨论

Jackson 提供了许多工作方式。 对于简单情况，可以将 POJO（Plain Old Java Objects）几乎自动转换为/从 JSON，如 Example 14-1 所示。

##### 示例 14-1\. json/src/main/java/json/ReadWriteJackson.java（使用 Jackson 读写 POJOs）

```java
public class ReadWriteJackson {

    public static void main(String[] args) throws IOException {
        ObjectMapper mapper = new ObjectMapper();                ![1](img/1.png)

        String jsonInput =                                       ![2](img/2.png)
                "{\"id\":0,\"firstName\":\"Robin\",\"lastName\":\"Wilson\"}";
        Person q = mapper.readValue(jsonInput, Person.class);
        System.out.println("Read and parsed Person from JSON: " + q);

        Person p = new Person("Roger", "Rabbit");                ![3](img/3.png)
        System.out.print("Person object " + p +" as JSON = ");
        mapper.writeValue(System.out, p);
    }
}
```

![1](img/#co_processing_json_data_CO1-1)

创建一个 Jackson `ObjectMapper`，可以将 POJOs 映射到/从 JSON。

![2](img/#co_processing_json_data_CO1-2)

将字符串 `jsonInput` 映射为 `Person` 对象，一次调用 `readValue()`。

![3](img/#co_processing_json_data_CO1-3)

使用一次 `writeValue()` 调用将 `Person` 对象 `p` 转换为 JSON。

运行此示例会产生以下输出：

```java
Read and parsed Person from JSON: Robin Wilson
Person object Roger Rabbit as JSON = {"id":0,"firstName":"Roger",
	"lastName":"Rabbit","name":"Roger Rabbit"}
```

作为另一个例子，这段代码读取了打开本章的示例文件（碰巧是一个 JSON 解析器的描述）。 请注意，贡献者数组的声明为 `List<String>`：

```java
public class SoftwareParseJackson {
    final static String FILE_NAME = "/json/softwareinfo.json";

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper(); ![1](img/1.png)

        InputStream jsonInput =
            SoftwareParseJackson.class.getResourceAsStream(FILE_NAME);
        if (jsonInput == null) {
            throw new NullPointerException("can't find " + FILE_NAME);
        }
        SoftwareInfo sware = mapper.readValue(jsonInput, SoftwareInfo.class);
        System.out.println(sware);
    }

}
```

![1](img/#co_processing_json_data_CO2-1)

`ObjectMapper` 执行 JSON 输入的实际解析。

运行此示例会产生以下输出：

```java
Software: robinparse (1.2.3) by [Robin Smythe, Jon Jenz, Jan Ardann]
```

当然，有些情况下映射会变得更加复杂； 为此，Jackson 提供了一组注解来控制映射。 但默认映射非常不错！

Jackson 还提供了一个流式 API；请参考网站了解详情。

# 14.3 使用 org.json 解析和写入 JSON

## 问题

您希望使用中等规模、广泛使用的 JSON API 读取/写入 JSON。

## 解决方案

考虑使用 org.json API，也称为 JSON-Java； 它被广泛使用，也用于 Android。

## 讨论

*org.json* 包不如 Jackson 高级，也不如高级；它让你以 JSON 抽象层次而不是 Java 代码层次思考和工作。 例如，这是从本章开头读取软件描述的 *org.json* 版本：

```java
public class SoftwareParseOrgJson {
    final static String FILE_NAME = "/json/softwareinfo.json";

    public static void main(String[] args) throws Exception {

        InputStream jsonInput =
            SoftwareParseOrgJson.class.getResourceAsStream(FILE_NAME);
        if (jsonInput == null) {
            throw new NullPointerException("can't find" + FILE_NAME);
        }
        JSONObject obj = new JSONObject(new JSONTokener(jsonInput));      ![1](img/1.png)
        System.out.println("Software Name: " + obj.getString("name"));    ![2](img/2.png)
        System.out.println("Version: " + obj.getString("version"));
        System.out.println("Description: " + obj.getString("description"));
        System.out.println("Class: " + obj.getString("className"));
        JSONArray contribs = obj.getJSONArray("contributors");            ![3](img/3.png)
        for (int i = 0; i < contribs.length(); i++) {                     ![4](img/4.png)
            System.out.println("Contributor Name: " + contribs.get(i));
        }
    }

}
```

![1](img/#co_processing_json_data_CO3-1)

从输入创建 `JSONObject`。

![2](img/#co_processing_json_data_CO3-2)

检索单个 `String` 字段。

![3](img/#co_processing_json_data_CO3-3)

检索贡献者名称的 `JSONArray`。

![4](img/#co_processing_json_data_CO3-4)

`org.json.JSONArray` 不实现 `Iterable`，因此无法使用 `forEach` 循环。

运行它会产生预期的输出：

```java
Software Name: robinparse
Version: 1.2.3
Description: Another Parser for JSON
Class: RobinParse
Contributor Name: Robin Smythe
Contributor Name: Jon Jenz
Contributor Name: Jan Ardann
```

`JSONObject` 和 `JSONArray` 使用它们的 `toString()` 方法生成（正确格式化的）JSON 字符串，如下所示：

```java
public class WriteOrgJson {
    public static void main(String[] args) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("Name", "robinParse").        ![1](img/1.png)
            put("Version", "1.2.3").
            put("Class", "RobinParse");
        String printable = jsonObject.toString();    ![2](img/2.png)
        System.out.println(printable);
    }
}
```

![1](img/#co_processing_json_data_CO4-1)

它提供了流畅 API，允许方法调用的链式操作。

![2](img/#co_processing_json_data_CO4-2)

`toString()` 将转换为文本 JSON 表示形式。

运行这个程序会产生以下结果：

```java
{"Name":"robinParse","Class":"RobinParse","Version":"1.2.3"}
```

## 参见

org.json 库的代码，包括其 javadoc 文档，都可以在 [*https://github.com/stleary/JSON-java*](https://github.com/stleary/JSON-java)（在名称上使用 JSON-java 以区分它与其他包）在线查看。

# 14.4 使用 JSON-B 解析和写入 JSON

## 问题

您希望使用中型、符合标准的 JSON API 进行读写 JSON。

## 解决方案

考虑使用 JSON-B，这是新的 Java 标准（JSR-367）。

## 讨论

JSON-B（JSON Binding）API 旨在简化读写 Java POJOs。这在代码示例 Example 14-2 中得到了清晰的展示。

##### 示例 14-2\. json/src/main/java/json/ReadWriteJsonB.java（使用 JSON-B 读写 JSON）

```java
public class ReadWriteJsonB {

    public static void main(String[] args) throws IOException {

        Jsonb jsonb = JsonbBuilder.create();            ![1](img/1.png)

        // Read
        String jsonInput =                              ![2](img/2.png)
                "{\"id\":0,\"firstName\":\"Robin\",\"lastName\":\"Williams\"}";
        Person rw = jsonb.fromJson(jsonInput, Person.class);
        System.out.println(rw);

        String result = jsonb.toJson(rw);                ![3](img/3.png)
        System.out.println(result);
    }
}
```

![1](img/#co_processing_json_data_CO5-1)

创建一个 `Jsonb` 对象，这是你使用 JSON-B 服务的入口。

![2](img/#co_processing_json_data_CO5-2)

获取一个 JSON 字符串，并使用 `jsonb.fromJson()` 将其转换为 Java 对象。

![3](img/#co_processing_json_data_CO5-3)

使用反向 `jsonb.toJson()` 将 `Person` 对象转换回 JSON 字符串。

注意方法命名合理，不需要在 Java 实体类上使用任何注解即可实现功能。然而，我们有一个 API 允许我们自定义。例如，`fullName` 属性只是一个便利，用于在名字和姓氏之间添加一个空格。因此，它完全是多余的，不需要在 JSON 网络流上传输。然而，运行程序会产生以下输出：

```java
{"firstName":"Robin","fullName":"Robin Williams","id":0,"lastName":"Williams"}
```

我们只需在 `Person` 类的 `getFullName()` 访问器中添加 `@JsonbTransient` 注解即可消除冗余；现在运行程序会产生较小的输出：

```java
{"firstName":"Robin","id":0,"lastName":"Williams"}
```

## 参见

与大多数其他 JSON API 一样，从简单注解到编写完整自定义序列化器/反序列化器助手，都有全面的支持。请参阅 [JSON-B 规范页面](https://javaee.github.io/jsonb-spec)，[JSON-B 主页](http://json-b.net) 和 [这个更长的在线教程](https://www.baeldung.com/java-json-binding-api)。

# 14.5 使用 JSON 指针查找 JSON 元素

## 问题

当你有一个 JSON 文档并且只想从中提取选定的值时。

## 解决方案

使用 `javax.json` 的 *JSON Pointer* 实现，这是从 JSON 中提取选定元素的标准 API。

## 讨论

[互联网标准 RFC 6901](https://tools.ietf.org/html/rfc6901) 详细说明了 JSON Pointer 的语法，这是一种独立于语言的语法，用于匹配 JSON 文档中的元素。显然受到 XML 语法 XPath 的启发，JSON Pointer 比 XPath 简单一些，因为 JSON 本身的简单性。基本上，JSON Pointer 是一个字符串，用于标识 JSON 文档中的元素（简单或数组）。`javax.json` 包提供了一种对象模型 API，与 Java 的 XML DOM API 类似，允许您创建不可变对象来表示对象（通过 `JsonObjectBuilder` 和 `JsonArrayBuilder`）或通过 `Reader` 或 `InputStream` 从 JSON 字符串格式中读取它们。

JSON 指针以 “/” 开始（继承自 XPath），后跟我们要查找的元素或子元素的名称。假设我们扩展了示例 14-2 中的 `Person` 示例，以添加喜剧演员扮演的角色数组，看起来像这样：

```java
{"firstName":"Robin","lastName":"Williams",
	"age": 63,"id":0,
	"roles":["Mork", "Mrs. Doubtfire", "Patch Adams"]}
```

接下来，以下 JSON 指针应生成给定的匹配：

```java
/firstName => Robin
/age => 63
/roles => ["Mork","Mrs. Doubtfire","Patch Adams"]
/roles/1 => "Mrs. Doubtfire"
```

示例 14-3 中的程序演示了这一点。

##### 示例 14-3\. json/src/main/java/json/JsonPointerDemo.java

```java
public class JsonPointerDemo {

    public static void main(String[] args) {
        String jsonPerson =
            "{\"firstName\":\"Robin\",\"lastName\":\"Williams\"," +
                "\"age\": 63," +
                "\"id\":0," +
                "\"roles\":[\"Mork\", \"Mrs. Doubtfire\", \"Patch Adams\"]}";

        System.out.println("Input: " + jsonPerson);

        JsonReader rdr =
                Json.createReader(new StringReader(jsonPerson));       ![1](img/1.png)
        JsonStructure jsonStr = rdr.read();
        rdr.close();

        JsonPointer jsonPointer;
        JsonString jsonString;

        jsonPointer = Json.createPointer("/firstName");                ![2](img/2.png)
        jsonString = (JsonString)jsonPointer.getValue(jsonStr);
        String firstName = jsonString.getString();
        System.out.println("/firstName => " + firstName);

        JsonNumber num =                                               ![3](img/3.png)
                (JsonNumber) Json.createPointer("/age").getValue(jsonStr);
        System.out.println("/age => " + num + "; a " + num.getClass().getName());

        jsonPointer = Json.createPointer("/roles");                    ![4](img/4.png)
        JsonArray roles = (JsonArray) jsonPointer.getValue(jsonStr);
        System.out.println("/roles => " + roles);
        System.out.println("JsonArray roles.get(1) => " + roles.get(1));

        jsonPointer = Json.createPointer("/roles/1");                  ![5](img/5.png)
        jsonString = (JsonString)jsonPointer.getValue(jsonStr);
        System.out.println("/roles/1 => " + jsonString);
    }
}
```

![1](img/#co_processing_json_data_CO6-1)

通过使用 `JsonReader` 从 `StringReader` 创建 `JsonStructure`，这是进入此 API 的入口。

![2](img/#co_processing_json_data_CO6-2)

创建 `firstName` 元素的 JSON Pointer，并从元素的值获取 `JsonString`。如果不确定元素是否会被找到，则使用 `jsonPointer.containsValue(jsonStr)` 先检查，因为 `getValue()` 如果未找到元素将抛出异常。

![3](img/#co_processing_json_data_CO6-3)

对于 `age` 也是一样，但使用更流畅的语法。如果打印匹配 `/age` 的类名称，它将报告一个特定于实现的实现类，例如 `org.glassfish.json.JsonNumberImpl$JsonIntNumber`。将 XML 中的年龄从 63 更改为 63.5，它将打印一个带有 `BigDecimal` 的类名。无论哪种方式，该对象的 `toString()` 将只返回数值。

![4](img/#co_processing_json_data_CO6-4)

在 JSON 文件中，`roles` 是一个数组。因此，使用 JSON 指针获取它应返回一个 `JsonArray` 对象，因此我们将其转换为该类型的引用。这类似于不可变的 `List` 实现，因此我们调用 `get()`。JSON 数组索引从零开始，与 Java 相同。

![5](img/#co_processing_json_data_CO6-5)

直接检索相同的数组元素，使用带有“/1”的模式表示数组中的编号元素。

有可能（但幸运的是不常见）JSON 元素名称包含特殊字符，如斜杠。大多数字符对于 JSON Pointer 不是特殊的，但要匹配包含斜杠（*/）的名称，斜杠必须输入为 *~1*，而因为这使得波浪号（*~*）变成特殊字符，所以波浪号必须输入为 *~0*。因此，如果 Person JSON 文件有一个像 `"ft/pt/~"` 的元素，你需要使用 `Json.createPointer("/ft~1pt~1~0")` 来查找它。

## 参见

JSON Pointer API 提供了额外的方法，让你可以修改数值和添加/移除元素。官方主页为 `javax.json`，其中包括 JSON Pointer，位于 [*jakarta.ee*](https://jakarta.ee/specifications/jsonp/1.1)。`javax.json` 的 javadoc 可从该页面链接获取。

## 概要

Java 有很多 API。Jackson 是最大最强大的；org.json、javax.json 和 JSON-B 处于中间地带，StringTree（我没有给出示例因为它没有 Maven Artifact）最小。要查看这些及其他 JSON API 的列表，请参阅 [*https://www.json.org/json-en.html*](https://www.json.org/json-en.html)，并滚动到语法总结之后。
