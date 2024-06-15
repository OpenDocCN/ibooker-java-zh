# 第十五章： 使用响应式编程模型

我们都熟悉主导企业软件开发数十年的客户端-服务器架构。然而，最近我们的架构风格发生了变化。除了标准的客户端-服务器方法外，我们还有消息驱动的应用程序，微服务，响应式应用程序甚至无服务器！使用 Quarkus 可以创建所有这些类型的应用程序。在接下来的示例中，您将了解有关响应式编程模型，消息总线和流式传输的信息。

###### 注意

Quarkus（以及本书！）使用 SmallRye Mutiny 作为其响应式库。您可以在 [SmallRye Mutiny](https://oreil.ly/nBP7H) 上阅读更多信息。还支持 RxJava 和 Reactor，但它们不是首选选择。要使用它们中的任何一个，您需要使用从 Mutiny 转换器。

# 15.1 创建异步 HTTP 端点

## 问题

您希望创建异步 HTTP 端点。

## 解决方案

Quarkus 与 Java Streams，Eclipse MicroProfile 响应式规范和 SmallRye Mutiny 集成。这些集成使得支持异步 HTTP 端点变得非常容易。您首先需要确定希望使用哪些库。如果您希望使用原生 Streams 或 MicroProfile 响应式规范，您需要添加 `quarkus-smallrye-reactive-streams-operators` 扩展。如果您想使用 SmallRye Mutiny，则需要向项目添加 `quarkus-resteasy-mutiny` 扩展。

###### 注意

从现在开始，Mutiny 将是 Quarkus 内所有响应式相关内容的首选库。

一旦扩展位于适当位置，您需要做的就是在您的 HTTP 端点中返回一个响应式类：

```java
@GET
@PATH("/reactive")
@Produces(MediaType.TEXT_PLAIN) ![1](img/1.png)
public CompletionStage<String> reactiveHello() { ![2](img/2.png)
    return ReactiveStreams.of("h", "e", "l", "l", "o")
        .map(String::toUpperCase)
        .toList()
        .run()
        .thenApply(list -> list.toString());
}
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO1-1)

自然地，任何有效的 `MediaType` 都是有效的；为简单起见，我们使用了纯文本

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO1-2)

`CompletionStage` 来自 `java.util.concurrent` 包

对于 Mutiny，这个示例变成了以下内容：

```java
@GET
@PATH("/reactive")
@Produces(MediaType.TEXT_PLAIN)
public Multi<String> helloMutiny() {
    return Multi.createFrom().items("h", "e", "l", "l", "o");
}
```

## 参见

欲了解更多信息，请访问以下网站：

+   [SmallRye Mutiny](https://oreil.ly/nBP7H)

+   [SmallRye 响应式流操作符](https://oreil.ly/Ab8eo)

+   [响应式流](https://oreil.ly/pgyMk)

# 15.2 异步流式处理数据

## 问题

您希望以异步方式流式传输数据。

## 解决方案

与创建异步 HTTP 端点非常相似，Quarkus 允许您使用服务器发送事件（SSE）或服务器端事件从应用程序中流式传输事件。在这种情况下，您需要返回一个 `Publisher` 并告诉 JAX-RS 您的端点生成 `MediaType.SERVER_SENT_EVENTS`。以下是每 500 毫秒流式传输 `long` 的示例：

```java
@GET
@Path("/integers")
@Produces(MediaType.SERVER_SENT_EVENTS) ![1](img/1.png)
public Publisher<Long> longPublisher() { ![2](img/2.png)
    return Multi.createFrom()
            .ticks().every(Duration.ofMillis(500));
}
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO2-1)

确保告诉 JAX-RS 你在使用 SSEs

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO2-2)

方法的返回类型必须是来自 Reactive Streams 库的 `org.reactivestream.Publisher`

使用 Mutiny，一个 `Multi` 是一个 `Publisher`，这样做更加简单，只需返回一个 `Multi`。

## 另请参见

更多信息，请访问以下网站：

+   [Wikiwand：服务器发送事件](https://oreil.ly/m0cAC)

+   [MDN Web Docs：使用服务器发送事件](https://oreil.ly/ZIX4X)

# 15.3 使用消息传递来解耦组件

## 问题

你想使用消息传递来解耦组件。

## 解决方案

Quarkus 使用的一个底层/捆绑框架是 Vert.x。Vert.x 是一个用于构建异步、事件驱动、响应式应用的框架，类似于 Quarkus！Quarkus 利用 Vert.x 事件总线发送和接收事件/消息，类之间是解耦的。

要使用 Vert.x，就像许多 Quarkus 的特性一样，你需要将适当的扩展添加到你的应用中。Vert.x 扩展的名称是 `vertx`。

### 处理事件/消息

我们将首先看一下监听或消费事件。 在 Quarkus 中消费事件的最简单方式是使用 `io.quarkus.vertx.ConsumeEvent` 注解。`@ConsumeEvent` 有一些属性，我们稍后会讲到，但让我们先看看它的实际应用：

```java
package com.acme.vertx;

import javax.enterprise.context.ApplicationScoped;

import io.quarkus.vertx.ConsumeEvent;

@ApplicationScoped
public class GreetingService {
    @ConsumeEvent   ![1](img/1.png)
    public String consumeNormal(String name) { ![2](img/2.png)
        return name.toUpperCase();
    }
}
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO3-1)

如果没有设置值，事件的地址是 bean 的完全限定名；在这种情况下，应该是 `com.acme.vertx.GreetingService`

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO3-2)

消费者的参数是消息体；如果方法返回任何内容，它会被封装为消息响应。

### 发送事件/消息

要发送事件，你将与 Vert.x 事件总线进行交互。你可以通过注入获取实例：`@Inject io.vertx.axle.core.eventbus.EventBus bus`。你将主要使用事件总线上两个方法：

`send`

发送消息并可选地期待回复

`publish`

向每个监听器发布消息

```java
bus.send("address", "hello"); ![1](img/1.png)
bus.publish("address", "hello"); ![2](img/2.png)
bus.send("address", "hello, how are you?") ![3](img/3.png)
    .thenAccept(msg -> {
    // do something with the message });
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO4-1)

向特定地址发送消息，一个消费者接收后，便忽略响应。

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO4-2)

向特定地址发布消息，所有消费者都接收该消息。

![3](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO4-3)

向特定地址发送消息，并期待回复。

你应该已经有足够的信息来使用 Vert.x 事件机制创建你自己的问候服务版本！

## 讨论

您还可以返回 `CompletionStage` 以异步方式处理事件。最后，如果希望使用 `io.vertx.axle.core.eventbus.Message` 作为方法参数，可以这样做，并在事件处理程序中访问消息的其余部分。

火而忘交互方式同样简单——只需从您的方法返回 `void` 即可。

###### 警告

Vert.x 事件循环上调用消耗事件的方法。Vert.x 的第一个原则是永不阻塞事件循环。您的代码 *必须* 是非阻塞的。 *如果* 需要方法阻塞，将 `@ConsumeEvent` 上的 `blocking` 属性设置为 `true`。

要配置事件处理程序的名称或地址，请使用 `value` 参数：

```java
    @ConsumeEvent(value = "greeting")
    public String consumeNamed(String name) {
        return name.toUpperCase();
    }
```

## 参见

欲了解更多信息，请访问以下网站：

+   [Vert.x](https://www.vertx.io)

+   [Vert.x：事件总线](https://oreil.ly/TAnxk)

# 15.4 响应 Apache Kafka 消息

## 问题

您希望响应 Apache Kafka 消息。

## 解决方案

Quarkus 利用 Eclipse MicroProfile Reactive Messaging 与 Apache Kafka 进行交互。

反应式消息规范是建立在三个主要概念之上的：

1.  `Message`

1.  `@Incoming`

1.  `@Outgoing`

本配方专注于 `Message` 和 `@Incoming`；如果您需要进行反向操作，请参阅配方 15.5。

### 消息

简而言之，`Message` 是围绕有效载荷的封套。封套还携带可选的元数据，尽管更多时候，您只关心有效载荷。

### @Incoming 注解

此注解 (`org.eclipse.microprofile.reactive.messaging.Incoming`) 表示该方法消耗消息流。唯一的属性是流或主题的名称。方法以此方式注解是为了处理链的末端，也称为 *sink*。以下是在 Quarkus 中的几个用法：

```java
package org.acme.kafka;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.CompletionStage;

import javax.enterprise.context.ApplicationScoped;

import org.eclipse.microprofile.reactive.messaging.Incoming;
import org.eclipse.microprofile.reactive.messaging.Message;

@ApplicationScoped
public class CharacterReceiver {
  @Incoming("ascii-char")
  public CompletionStage<Void> processKafkaChar(Message<String> character) {
    return CompletableFuture.runAsync(() -> {
      System.out.println("Received a message from Kafka "
          + "using CompletableFuture: '" + character.getPayload() + "'");
    });
  }

  @Incoming("ascii-char")
  public void processCharacter(String character) {
    System.out.println("Received a String from kafka: '" + character + "'");
  }
}
```

您可以看到两种方法都可以运行；但是，在 `processKafkaCharacter` 的情况下，它接收一个 `Message` 并返回一个 `CompletionStage`。如果您的方法接收 `Message` 作为参数，则 *必须* 返回 `CompletionStage`。

如果您仅对有效载荷感兴趣，则无需担心其中任何内容，可以简单接受有效载荷的类型并返回 void，就像前一个代码中的 `processCharacter` 演示的那样。

### 配置

预期地，您需要配置应用程序以与 Apache Kafka 进行通信：

```java
mp.messaging.incoming.ascii-char.connector=smallrye-kafka
mp.messaging.incoming.ascii-char.value.deserializer=org.apache.kafka.common\
 .serialization\
 .StringDeserializer
mp.messaging.incoming.ascii-char.broadcast=true
```

在前述代码中，我们有多个订阅者，因此需要使用 `broadcast=true`。`broadcast` 属性让 MicroProfile Reactive Messaging（SmallRye 是一个实现）知道接收到的消息可以分发给多个订阅者。

配置的语法如下：

```java
mp.messaging.[outgoing|incoming].{channel-name}.property=value
```

`channel-name` 段中的值必须与 `@Incoming` 中设置的值匹配（以及在下一个配方中涵盖的 @`Outgoing`）。

您可以在 [SmallRye Reactive Messaging: Apache Kafka](https://oreil.ly/L5WHK) 中看到一些合理的默认值。

## 讨论

要快速在开发环境中使用 Apache Kafka，您可以访问 “另请参阅” 中列出的网站，或者使用以下 *docker-compose.yml* 文件以及 `docker-compose`：

```java
version: '2'

services:

  zookeeper:
    image: strimzi/kafka:0.11.3-kafka-2.1.0
    command: [
      "sh", "-c",
      "bin/zookeeper-server-start.sh config/zookeeper.properties"
    ]
    ports:
      - "2181:2181"
    environment:
      LOG_DIR: /tmp/logs

  kafka:
    image: strimzi/kafka:0.11.3-kafka-2.1.0
    command: [
      "sh", "-c",
      "bin/kafka-server-start.sh config/server.properties
      --override listeners=$${KAFKA_LISTENERS}
      --override advertised.listeners=$${KAFKA_ADVERTISED_LISTENERS}
      --override zookeeper.connect=$${KAFKA_ZOOKEEPER_CONNECT}"
    ]
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      LOG_DIR: "/tmp/logs"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
```

## 另请参阅

欲了解更多信息，请访问以下网址：

+   [GitHub：MicroProfile 的响应式消息传递](https://oreil.ly/DSu7u)

+   [SmallRye 响应式消息传递](https://oreil.ly/QlGHK)

+   [Apache Kafka](https://oreil.ly/6xsAP)

+   [Apache Kafka：消费者配置](https://oreil.ly/iE8aU)

+   [Vert.x Kafka 客户端](https://oreil.ly/zFD5J)

# 15.5 向 Apache Kafka 发送消息

## 问题

您想要发送消息到 Apache Kafka。

## 解决方案

首先，您需要将 `quarkus-smallrye-reactive-messaging-kafka` 扩展添加到您的项目中。在本示例中，我们还使用了 SmallRye Mutiny，因此还需添加 `io.smallrye.reactive:mutiny` 依赖项。

要将消息发送到 Apache Kafka，请使用来自 Eclipse MicroProfile Reactive Messaging 的 `@Outgoing` 注解。

当您生成要发送到 Apache Kafka 的数据时，您将使用 `org.eclipse.microprofile.reactive.messaging.Outgoing` 注解您的方法。您可以发送一系列消息或单个消息。如果您希望发布一系列消息，则必须返回 `org.reactivestreams.Publisher` 或 `org.eclipse.microprofile.reactive.streams.operators.PublisherBuilder`。如果您希望发布单个消息，请返回 `org.eclipse.microprofile.reactive.messaging.Message`、`java.util.concurrent.CompletionStage` 或您消息负载的相应类型。

以下是一个基本示例，每秒创建一个新的 ASCII 字符并将其发送到 “letter-out” 通道：

```java
package org.acme.kafka;

import java.time.Duration;
import java.util.concurrent.ThreadLocalRandom;

import io.smallrye.mutiny.Multi;
import org.eclipse.microprofile.reactive.messaging.Outgoing;
import org.reactivestreams.Publisher;

public class CharacterGenerator {
    @Outgoing("letter-out")
    public Publisher<String> generate() {
        return Multi.createFrom()
                .ticks().every(Duration.ofSeconds(1))
                .map(tick -> {
                    final int i = ThreadLocalRandom.current().nextInt(95);
                    return String.valueOf((char) (i + 32));
                });
    }
}
```

`@Outgoing` 的 value 属性是必需的，是出站通道的名称。在本示例中，我们使用了 SmallRye Mutiny，但您也可以使用任何返回 `org.reactivestreams.Publisher` 实例的内容；例如，来自 RXJava2 的 `Flowable` 也可以很好地工作。

还需要以下配置：

```java
mp.messaging.outgoing.letter-out.connector=smallrye-kafka
mp.messaging.outgoing.letter-out.topic=ascii-char
mp.messaging.outgoing.letter-out.value.serializer=org.apache.kafka.common\
 .serialization\
 .StringSerializer
```

## 讨论

如果您发现自己需要以命令的方式发送消息，您可以在应用程序中注入一个 `org.eclipse.microprofile.reactive.messaging.Emitter`：

```java
@Inject @Channel("price-create") Emitter<Double> priceEmitter;

@POST
@Consumes(MediaType.TEXT_PLAIN)
public void addPrice(Double price) {
    priceEmitter.send(price);
}
```

## 另请参阅

欲了解更多信息，请查看以下内容：

+   配方 15.4

+   [Apache Kafka：生产者配置](https://oreil.ly/hZ9Bm)

+   [SmallRye 响应式消息传递](https://oreil.ly/QlGHK)

# 15.6 将 POJOs 编组到/从 Kafka

## 问题

您希望将 POJOs 序列化/反序列化到 Kafka。

## 解决方案

Quarkus 具有处理 JSON Kafka 消息的功能；您需要选择 JSONB 或 Jackson 作为实现。所需的扩展名为 `quarkus-resteasy-jsonb` 或 `quarkus-resteasy-jackson`，具体取决于您的偏好。

然后，您需要创建一个反序列化器。最简单的方法是扩展 JSONB 的 `JsonDeserializer` 或 Jackson 的 `ObjectMapperDeserializer`。这是 `Book` 类及其反序列化器：

```java
package org.acme.kafka;

public class Book {
    public String title;
    public String author;
    public Long isbn;

    public Book() {
    }

    public Book(String title, String author, Long isbn) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
    }
}
```

对于 JSONB，反序列化器如下所示：

```java
package org.acme.kafka;

import io.quarkus.kafka.client.serialization.JsonbDeserializer;

public class BookDeserializer extends JsonbDeserializer<Book> {
    public BookDeserializer() {
        super(Book.class);
    }
}
```

对于杰克逊来说，这同样容易：

```java
package com.acme.kafka;

import io.quarkus.kafka.client.serialization.ObjectMapperDeserializer;

public class BookDeserializer extends ObjectMapperDeserializer<Book> {
    public BookDeserializer() {
        super(Book.class);
    }
}
```

您需要做的最后一件事是将您的反序列化器和默认序列化器添加到 Quarkus 配置中：

```java
# Configure the Kafka source (we read from it)
mp.messaging.incoming.book-in.connector=smallrye-kafka
mp.messaging.incoming.book-in.topic=book-in
mp.messaging.incoming.book-in.value.deserializer=com.acme\
 .kafka.BookDeserializer

# Configure the Kafka sink (we write to it)
mp.messaging.outgoing.book-out.connector=smallrye-kafka
mp.messaging.outgoing.book-out.topic=book-out
mp.messaging.outgoing.book-out.value.serializer=io.quarkus.kafka\
 .client.serialization\
 .JsonbSerializer
```

或者，对于 Jackson：

```java
# Configure the Kafka source (we read from it)
mp.messaging.incoming.book-in.connector=smallrye-kafka
mp.messaging.incoming.book-in.topic=book-in
mp.messaging.incoming.book-in.value.deserializer=com.acme\
 .kafka.BookDeserializer

# Configure the Kafka sink (we write to it)
mp.messaging.outgoing.book-out.connector=smallrye-kafka
mp.messaging.outgoing.book-out.topic=book-out
mp.messaging.outgoing.book-out.value.serializer=io.quarkus.kafka.client\
 .serialization\
 .ObjectMapperSerializer
```

## 讨论

如果您使用 JSONB 并且不希望为每个通过电线发送的 POJO 创建反序列化器，您可以使用通用的`io.vertx.kafka.client.serialization.JsonObjectDeserializer`。返回的对象将是一个`javax.json.JsonObject`。在这里，我们选择使用默认的序列化器。

如果您需要比基本功能更多的东西，您还可以创建自己的序列化器。

# 15.7 使用 Kafka Streams API

## 问题

您希望使用 Kafka Streams API 来查询数据。

## 解决方案

Quarkus 中的 Apache Kafka 扩展(`quarkus-smallrye-reactive-messaging-kafka`)与 Apache Kafka Streams API 集成。这个例子有点深奥，需要一些额外的移动部件。当然，您需要一个运行中的 Apache Kafka 实例。如果您还没有可用的实例，我们建议您使用 Kubernetes 设置一个 Apache Kafka 实例。如果您只是需要开发用的东西，您可以使用以下*docker-compose.yml*文件：

```java
version: '3.5'

services:
  zookeeper:
    image: strimzi/kafka:0.11.3-kafka-2.1.0
    command: [
      "sh", "-c",
      "bin/zookeeper-server-start.sh config/zookeeper.properties"
    ]
    ports:
      - "2181:2181"
    environment:
      LOG_DIR: /tmp/logs
    networks:
      - kafkastreams-network
  kafka:
    image: strimzi/kafka:0.11.3-kafka-2.1.0
    command: [
      "sh", "-c",
      "bin/kafka-server-start.sh config/server.properties
      --override listeners=$${KAFKA_LISTENERS}
      --override advertised.listeners=$${KAFKA_ADVERTISED_LISTENERS}
      --override zookeeper.connect=$${KAFKA_ZOOKEEPER_CONNECT}
      --override num.partitions=$${KAFKA_NUM_PARTITIONS}"
    ]
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      LOG_DIR: "/tmp/logs"
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_NUM_PARTITIONS: 3
    networks:
      - kafkastreams-network
```

此解决方案的下一部分是创建一个生成值并将这些生成的值发送到 Kafka 主题的生产者。我们将使用一个点唱机的概念。我们的点唱机将包含多首歌曲及其艺术家，以及歌曲播放次数。每个都将发送到不同的主题，然后由另一个服务聚合在一起：

```java
package org.acme.kafka.jukebox;

import java.time.Duration;
import java.time.Instant;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.ThreadLocalRandom;

import javax.enterprise.context.ApplicationScoped;

import io.smallrye.mutiny.Multi;
import io.smallrye.reactive.messaging.kafka.KafkaRecord;
import org.eclipse.microprofile.reactive.messaging.Outgoing;
import org.jboss.logging.Logger;

@ApplicationScoped
public class Jukebox {
    private static final Logger LOG = Logger.getLogger(Jukebox.class);

    private ThreadLocalRandom random = ThreadLocalRandom.current();

    private List<Song> songs = Collections.unmodifiableList(
            Arrays.asList(
                new Song(1, "Confessions", "Usher"),
                new Song(2, "How Do I Live", "LeAnn Rimes"),
                new Song(3, "Physical", "Olivia Newton-John"),
                new Song(4, "You Light Up My Life", "Debby Boone"),
                new Song(5, "The Twist", "Chubby Checker"),
                new Song(6, "Mack the Knife", "Bobby Darin"),
                new Song(7, "Night Fever", "Bee Gees"),
                new Song(8, "Bette Davis Eyes", "Kim Carnes"),
                new Song(9, "Macarena (Bayside Boys Mix)", "Los Del Rio"),
                new Song(10, "Yeah!", "Usher")
            )
    );

    @Outgoing("song-values")
    public Multi<KafkaRecord<Integer, String>> generate() {
        return Multi.createFrom().ticks().every(Duration.ofMillis(500))
                .onOverflow().drop()
                .map(tick -> {
                   Song s = songs.get(random.nextInt(songs.size()));
                   int timesPlayed = random.nextInt(1, 100);

                   LOG.infov("song {0}, times played: {1,number}",
                           s.title, timesPlayed);
                   return KafkaRecord.of(s.id, Instant.now()
                                               + ";" + timesPlayed);
                });
    }

    @Outgoing("songs")
    public Multi<KafkaRecord<Integer, String>> songs() {
        return Multi.createFrom().iterable(songs)
                .map(s -> KafkaRecord.of(s.id,
                        "{\n" +
                        "\t\"id\":\""+ s.id + "\",\n" +
                        "\t\"title\":\"" + s.title + "\",\n" +
                        "\t\"artist\":\"" + s.artist + "\"\n" +
                        "}"
                        ));
    }

    private static class Song {
        int id;
        String title;
        String artist;

        public Song(int id, String title, String artist) {
            this.id = id;
            this.title = title;
            this.artist = artist;
        }
    }
}
```

每 500 毫秒，一个包含歌曲及其播放次数以及时间戳的新消息将发送到`songs`主题。我们将跳过额外的配置——您可以在食谱 15.5 中查看详细步骤。

接下来，我们需要构建管道。第一步是创建一些值持有者：

```java
package org.acme.kafka.jukebox;

public class Song {
    public int id;
    public String title;
    public String artist;
}
```

现在我们需要一个播放计数器：

```java
package org.acme.kafka.jukebox;

import java.time.Instant;

public class PlayedCount {
    public int count;
    public String title;
    public String artist;
    public int id;
    public Instant timestamp;

    public PlayedCount(int id, String title, String artist,
                       int count, Instant timestamp) {
        this.count = count;
        this.title = title;
        this.artist = artist;
        this.id = id;
        this.timestamp = timestamp;
    }
}
```

最后，对于值持有者，这是一个对象，用于在管道中处理消息时跟踪值的聚合：

```java
package org.acme.kafka.jukebox;

import java.math.BigDecimal;
import java.math.RoundingMode;

public class Aggregation {
    public int songId;
    public String songTitle;
    public String songArtist;
    public int count;
    public int sum;
    public int min;
    public int max;
    public double avg;

    public Aggregation updateFrom(PlayedCount playedCount) {
        songId = playedCount.id;
        songTitle = playedCount.title;
        songArtist = playedCount.artist;

        count++;
        sum += playedCount.count;
        avg = BigDecimal.valueOf(sum / count)
                .setScale(1, RoundingMode.HALF_UP).doubleValue();
        min = Math.min(min, playedCount.count);
        max = Math.max(max, playedCount.count);

        return this;
    }
}
```

现在，进入魔法部分！拼图的最后一部分是流式查询实现。我们只需要定义一个作为 CDI 生产者的方法，返回一个 Apache Kafka Stream `Topology`。Quarkus 将负责配置，生命周期将处理 Kafka Streams 引擎：

```java
package org.acme.kafka.jukebox;

import java.time.Instant;

import javax.enterprise.context.ApplicationScoped;
import javax.enterprise.inject.Produces;

import io.quarkus.kafka.client.serialization.JsonbSerde;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.Consumed;
import org.apache.kafka.streams.kstream.GlobalKTable;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.state.KeyValueBytesStoreSupplier;
import org.apache.kafka.streams.state.Stores;

@ApplicationScoped
public class TopologyProducer {
    static final String SONG_STORE = "song-store";

    private static final String SONG_TOPIC = "songs";
    private static final String SONG_VALUES_TOPIC = "song-values";
    private static final String SONG_AGG_TOPIC = "song-aggregated";

    @Produces
    public Topology buildTopology() {
        StreamsBuilder builder = new StreamsBuilder();

        JsonbSerde<Song> songSerde = new JsonbSerde<>(Song.class);
        JsonbSerde<Aggregation> aggregationSerde =
                new JsonbSerde<>(Aggregation.class);

        KeyValueBytesStoreSupplier storeSupplier =
                Stores.persistentKeyValueStore(SONG_STORE);

        GlobalKTable<Integer, Song> songs = builder.globalTable(SONG_TOPIC,
                Consumed.with(Serdes.Integer(), songSerde));

        builder.stream(SONG_VALUES_TOPIC, Consumed.with(Serdes.Integer(),
                Serdes.String()))
                .join(
                        songs,
                        (songId, timestampAndValue) -> songId,
                        (timestampAndValue, song) -> {
                            String[] parts = timestampAndValue.split(";");
                            return new PlayedCount(song.id, song.title,
                                    song.artist,
                                    Integer.parseInt(parts[1]),
                                    Instant.parse(parts[0]));
                        }
                )
                .groupByKey()
                .aggregate(
                        Aggregation::new,
                        (songId, value, aggregation) ->
                                aggregation.updateFrom(value),
                        Materialized.<Integer, Aggregation> as(storeSupplier)
                            .withKeySerde(Serdes.Integer())
                            .withValueSerde(aggregationSerde)
                )
                .toStream()
                .to(
                        SONG_AGG_TOPIC,
                        Produced.with(Serdes.Integer(), aggregationSerde)
                );
        return builder.build();
    }
}
```

解释所有发生的事情超出了本教程的范围，但是 Kafka Streams 站点在“另请参阅”中链接了完整的教程和视频专门讨论此主题。简而言之，这将连接到先前的`songs`和`song-values`主题，然后根据歌曲 ID 合并值。然后对播放计数值执行一些聚合操作，并将输出发送回 Apache Kafka 到一个新主题。

## 讨论

我们建议使用`kafkacat`工具来查看发送到主题的消息。

###### 重要

在这两个 Apache Kafka 示例中，我们仅连接了单个客户端和机器。这不是 Quarkus 的限制，而是我们为了简化示例而做的。

## 另请参阅

欲知详情，请访问以下网站：

+   [Apache Kafka：Kafka Streams](https://oreil.ly/gNAof)

+   [Confluent：kafkacat 实用工具](https://oreil.ly/bgIT_)

# 15.8 使用 AMQP 与 Quarkus

## 问题

您希望使用 AMQP（高级消息队列协议）作为消息系统。

## 解决方案

使用`quarkus-smallrye-reactive-messaging-amqp`扩展。

就像 Kafka 集成一样，Quarkus 使用 Eclipse MicroProfile Reactive Messaging 作为所有消息交互的外观。通过向您的项目添加`quarkus-smallrye-reactive-messaging-amqp`扩展，您将获得 SmallRye AMQP 连接器及其相关依赖项。这将使得`@Outbound`、`@Inbound`、`@Broadcast`和其他 Eclipse MicroProfile Reactive Messaging 注解和概念可以与 AMQP 一起工作。

###### 警告

这些注解适用于 AMQP 1.0，不适用于 0.9.x。

你还需要在*application.properties*文件中将通道连接器设置为`smallrye-amqp`。请记住，这些配置的语法如下：

```java
mp.messaging.[outgoing|incoming].[channel-name].property=value
```

您还可以通过以下方式全局设置 AMQP 连接的用户名和密码：

```java
amqp-username=[my-username]
amqp-password=[my-secret-password]
```

或者，如果您需要与具有自己凭据的不同实例通信，可以在每个通道上设置这些凭据。详细属性请参阅 SmallRye 文档。

从 Recipe 15.5 中的代码，无论是用 AMQP 还是 Kafka，只要通道名称相同且 AMQP 设置和连接信息正确，都能正常工作。

## 另请参阅

欲知详情，请访问以下网站：

+   [SmallRye 响应式消息传递：AMQP 1.0](https://oreil.ly/ViPyo)

# 15.9 使用 MQTT

## 问题

您希望使用 MQTT（MQ Telemetry Transport）作为消息系统。

## 解决方案

使用`quarkus-smallrye-reactive-messaging-mqtt`扩展。

就像 Kafka 和 AMQP 集成一样，Quarkus 使用 Eclipse MicroProfile Reactive Messaging 作为所有消息交互的外观。通过向您的项目添加`quarkus-smallrye-reactive-messaging-mqtt`扩展，您将获得 SmallRye MQTT 连接器及其相关依赖项。这将使得`@Outbound`、`@Inbound`、`@Broadcast`和其他 Eclipse MicroProfile Reactive Messaging 注解和概念可以与 MQTT 一起工作。

你还需要在*application.properties*文件中将通道连接器设置为`smallrye-mqtt`。请记住，这些配置的语法如下：

```java
mp.messaging.[outgoing|incoming].[channel-name].property=value
```

连接和凭据可以按通道设置。详细属性请参阅 SmallRye 文档。

配方 15.4 的代码与 MQTT 一样适用于 Kafka，假设通道名称相同，并且其余的 MQTT 设置及连接信息正确。

## 讨论

还支持作为 MQTT 服务器使用；但是，它不是一个完整功能的 MQTT 服务器。例如，它只处理发布请求及其确认；不处理订阅请求。

## 另请参阅

欲了解更多信息，请访问以下网站：

+   [SmallRye 响应式消息传递：MQTT](https://oreil.ly/QmkVY)

# 15.10 使用响应式 SQL 进行查询

## 问题

您希望使用 PostgreSQL 响应式客户端查询数据。

## 解决方案

Quarkus 集成了 Vert.x 响应式 SQL 客户端，可与 MySQL/MariaDB 和 PostgreSQL 配合使用。在这个示例中，我们将演示与 PostgreSQL 的集成；在下一个示例中，我们将使用 MariaDB。

自然地，您需要添加扩展以利用响应式 SQL 客户端。目前有两个扩展：`quarkus-reactive-pg-client`和`quarkus-reactive-mysql-client`，分别适用于这两个数据库。如果您正在使用 JAX-RS，则还需要确保项目中包含以下扩展：

+   `quarkus-resteasy`

+   `quarkus-resteasy-jsonb`或`quarkus-resteasy-jackson`

+   `quarkus-resteasy-mutiny`

就像使用任何数据存储一样，您需要配置访问：

```java
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=quarkus_test
quarkus.datasource.password=quarkus_test
quarkus.datasource.reactive.url=postgresql://localhost:5432/quarkus_test
```

现在您可以使用客户端：

```java
package org.acme.pg;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import io.vertx.mutiny.pgclient.PgPool;
import io.vertx.mutiny.sqlclient.Row;
import io.vertx.mutiny.sqlclient.Tuple;

public class Book {
    public Long id;
    public String title;
    public String isbn;

    public Book() {
    }

    public Book(String title, String isbn) {
        this.title = title;
        this.isbn = isbn;
    }

    public Book(Long id, String title, String isbn) {
        this.id = id;
        this.title = title;
        this.isbn = isbn;
    }

    public static Book from(Row row) {
        return new Book(row.getLong("id"),
                        row.getString("title"),
                        row.getString("isbn"));
    }

    public static Multi<Book> findAll(PgPool client) {
        return client.query("SELECT id, title, isbn " +
                            "FROM books ORDER BY title ASC") ![1](img/1.png)
                .onItem().produceMulti(Multi.createFrom()::iterable) ![2](img/2.png)
                .map(Book::from); ![3](img/3.png)
    }
}
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO5-1)

查询数据库，返回`Uni<RowSet<Row>>`

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO5-2)

一旦查询返回，创建`Multi<Row>`

![3](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO5-3)

将每一行映射为`Book`实例

要完成练习，您可以使用 RESTful 端点：

```java
package org.acme.pg;

import javax.annotation.PostConstruct;
import javax.inject.Inject;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import io.smallrye.mutiny.Uni;
import io.vertx.mutiny.pgclient.PgPool;
import org.eclipse.microprofile.config.inject.ConfigProperty;

@Path("/books")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class BookResource {

    @Inject
    PgPool client;
    @GET
    public Uni<Response> get() {
        return Book.findAll(client)
                .collectItems().asList()
                .map(Response::ok)
                .map(Response.ResponseBuilder::build);
    }
}
```

## 讨论

您还可以通过使用`preparedQuery`方法和`Tuple`类来使用预准备查询：

```java
    public static Uni<Boolean> delete(PgPool client, Long id) {
        return client.preparedQuery("DELETE FROM books " +
                                    "WHERE id = $1", Tuple.of(id))
                .map(rowSet -> rowSet.rowCount() == 1); ![1](img/1.png)
    }
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO6-1)

使用元数据从`RowSet`实例返回以验证已删除的行

## 另请参阅

底层实现可以在[Vert.x: 响应式 PostgreSQL 客户端](https://oreil.ly/0nuDM)找到。

# 15.11 使用响应式 SQL 客户端进行插入

## 问题

您希望使用 MySQL 响应式客户端插入数据。

## 解决方案

与使用 PostgreSQL 的上一个示例类似，可以使用响应式 MySQL 客户端进行数据插入。需要使用相同的扩展来将`quarkus-reactive-pg-client`更改为`quarkus-reactive-mysql-client`：

+   `quarkus-resteasy`

+   `quarkus-resteasy-jsonb`或`quarkus-resteasy-jackson`

+   `quarkus-resteasy-mutiny`

当然，您需要设置数据源：

```java
quarkus.datasource.db-kind=mysql
quarkus.datasource.username=quarkus_test
quarkus.datasource.password=quarkus_test
quarkus.datasource.reactive.url=mysql://localhost:3306/quarkus_test
```

您将在`Book.save`方法中看到与上一个示例中相似的主题：

```java
    public Uni<Long> save(MySQLPool client) {
        String query = "INSERT INTO books (title,isbn) VALUES (?,?)";
        return client.preparedQuery(query, Tuple.of(title, isbn))
                .map(rowSet -> rowSet
                        .property(MySQLClient.LAST_INSERTED_ID)); ![1](img/1.png)
    }
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO7-1)

使用`RowSet`的属性获取插入的 ID。

到目前为止，您应该能够为`BookResource`端点编写适当的 POST 方法，调用从用户接收到的新`Book`实例的`save`方法。

## 参见

欲了解更多信息，请访问以下网站：

+   [Vert.x：Reactive MySQL 客户端](https://oreil.ly/UHxfh)

# 15.12 使用反应式 MongoDB 客户端

## 问题

您想要使用反应式 MongoDB 客户端。

## 解决方案

MongoDB Quarkus 扩展还包括反应式 MongoDB 客户端。如配方 7.21 所示，您需要添加`quarkus-mongodb-client`。您还需要向项目添加以下扩展：

`quarkus-resteasy-mutiny`

为了返回和与 Mutiny 交互以进行端点返回。

`quarkus-smallrye-context-propagation`

这允许像注入和事务这样的事情与异步代码一起工作。

集成的其余部分非常简单。这里是从先前的 MongoDB 配方中的服务和资源类的版本，但以反应式方式编写：

```java
package org.acme.mongodb;

import java.util.List;
import java.util.Objects;

import javax.enterprise.context.ApplicationScoped;
import javax.inject.Inject;

import com.mongodb.client.model.Filters;
import io.quarkus.mongodb.reactive.ReactiveMongoClient;
import io.quarkus.mongodb.reactive.ReactiveMongoCollection;
import io.smallrye.mutiny.Uni;
import org.bson.Document;

@ApplicationScoped
public class ReactiveBookService {
    @Inject
    ReactiveMongoClient mongoClient;

    public Uni<List<Book>> list() {
        return getCollection().find()
                .map(Book::from).collectItems().asList();
    }

    public Uni<Void> add(Book b) {
        Document doc = new Document()
                .append("isbn", b.isbn)
                .append("title", b.title)
                .append("authors", b.authors);

        return getCollection().insertOne(doc);
    }

    public Uni<Book> findSingle(String isbn) {
        return Objects.requireNonNull(getCollection()
                .find(Filters.eq("isbn", isbn))
                .map(Book::from))
                .toUni();
    }

    private ReactiveMongoCollection<Document> getCollection() {
        return mongoClient.getDatabase("book")
                .getCollection("book");
    }
}
```

除了导入和从命令式转变为反应式使用 Mutiny，没有什么改变。同样适用于 REST 终点：

```java
package org.acme.mongodb;

import java.util.List;

import javax.inject.Inject;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import io.smallrye.mutiny.Uni;

@Path("/reactive_books")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ReactiveBookResource {
    @Inject
    ReactiveBookService service;

    @GET
    public Uni<List<Book>> getAll() {
        return service.list();
    }

    @GET
    @Path("{isbn}")
    public Uni<Book> getSingle(@PathParam("isbn") String isbn) {
        return service.findSingle(isbn);
    }

    @POST
    public Uni<Response> add(Book b) {
        return service.add(b).onItem().ignore()
                .andSwitchTo(this::getAll)
                .map(books -> Response.status(Response.Status.CREATED)
                                      .entity(books).build());
    }
}
```

## 参见

欲了解更多信息，请访问以下网站：

+   [Quarkus：Quarkus 中的上下文传播](https://oreil.ly/bd9hA)

# 15.13 使用反应式 Neo4j 客户端

## 问题

您想要使用反应式 Neo4j 客户端。

## 解决方案

Neo4j Quarkus 扩展支持反应式驱动程序。

您需要使用 4 版或更高版本的 Neo4j 完全反应式。您还需要向项目添加`quarkus-resteasy-mutiny`扩展。从配方 7.23 继续，除了使用驱动程序的`RxSession`和使用 Mutiny 外，没有太多改变：

```java
package org.acme.neo4j;

import java.util.stream.Collectors;

import javax.inject.Inject;
import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;

import io.smallrye.mutiny.Multi;
import io.smallrye.mutiny.Uni;
import org.neo4j.driver.Driver;
import org.neo4j.driver.Record;
import org.neo4j.driver.Value;
import org.neo4j.driver.Values;
import org.neo4j.driver.reactive.RxResult;
import org.reactivestreams.Publisher;

@Path("/reactivebooks")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ReactiveBookResource {
    @Inject
    Driver driver;

    @GET
    @Produces(MediaType.SERVER_SENT_EVENTS) ![3](img/3.png)
    public Publisher<Response> getAll() {
        return Multi.createFrom().resource( ![2](img/2.png)
                driver::rxSession,
                rxSession -> rxSession.readTransaction(tx -> { ![1](img/1.png)
                    RxResult result = tx.run("MATCH (b:Book) RETURN " +
                                             "b ORDER BY b.title");
                    return Multi.createFrom().publisher(result.records())
                            .map(Record::values)
                            .map(values -> values.stream().map(Value::asNode)
                                                          .map(Book::from)
                                                          .map(Book::toJson))
                            .map(bookStream ->
                                    Response.ok(bookStream
                                            .collect(Collectors.toList()))
                                    .build());
                }))
                .withFinalizer(rxSession -> { ![4](img/4.png)
                    return Uni.createFrom().publisher(rxSession.close());
                });
    }

    @POST
    public Publisher<Response> create(Book b) {
        return Multi.createFrom().resource(
                driver::rxSession,
                rxSession -> rxSession.writeTransaction(tx -> {
                    String query = "CREATE " +
                                   "(b:Book {title: $title, isbn: $isbn," +
                                   " authors: $authors}) " +
                                   "RETURN b";
                    RxResult result = tx.run(query,
                            Values.parameters("title", b.title,
                                    "isbn", b.isbn, "authors", b.authors));
                    return Multi.createFrom().publisher(result.records())
                            .map(record -> Response.ok(record
                                    .asMap()).build());
                })
        ).withFinalizer(rxSession -> {
            return Uni.createFrom().publisher(rxSession.close());
        });
    }
}
```

![1](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO8-3)

从驱动程序获取`RxSession`

![2](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO8-2)

使用 Mutiny 与 ReactiveStreams `Publisher` 进行交互

![3](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO8-1)

将结果流式传输回用户

![4](img/#co_working_with_a_reactive___span_class__keep_together__programming_model__span__CO8-4)

在最后关闭会话
