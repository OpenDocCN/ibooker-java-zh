# 附录 B. 管理 API 的生命周期

API 很少是静态的。随着你的产品发展，你需要通过 API 公开新的功能和特性，这意味着你需要创建新的端点或更改你的模式以引入新的实体或字段。通常，API 变更是不向后兼容的，这意味着未意识到新变更的客户端将收到失败的响应。管理 API 的一部分是确保你做出的任何变更都不会破坏与其他应用程序已经存在的集成，而 API 版本控制就是为了这个目的。在本附录中，我们研究 API 版本控制策略来管理 API 变更。

除了发展和变化之外，API 有时也会结束。也许你正在将 REST API 迁移到 GraphQL，或者你正在完全停止一个产品。如果你计划弃用 API，你必须让你的客户知道何时以及如何发生，在本附录的第二部分，你将学习如何向用户广播此信息。

## B.1 API 版本控制策略

让我们看看我们如何使用版本控制来管理 API 变更。我们为 API 使用两种主要的版本控制系统：

+   *语义版本控制*（SemVer，[`semver.org/`](https://semver.org/)）——这是最常见的版本控制类型，它被广泛用于管理软件发布。其格式如下：MAJOR.MINOR.PATCH，例如，1.1.0。第一个数字表示发布的主版本号，第二个数字表示次版本号，第三个数字表示补丁版本号。

    每当你对 API 进行重大变更时，主版本号就会发生变化，例如，当请求负载中需要新的字段时。次版本号代表 API 的非破坏性变更，例如引入新的可选查询参数。API 消费者期望能够在不同的次版本上以相同的方式调用你的端点，并继续获得响应。补丁版本表示错误修复。

    在 API 的上下文中，我们通常只使用主版本号，所以我们可能有 API 的 v1 和 v2 版本。一般而言，改进 API 的次级变更和补丁可以安全地推出，而不会破坏现有的集成。

+   日历版本控制（CalVer，[`calver.org/`](https://calver.org/)）——日历版本控制使用日历来为发布版本。当你的 API 变更非常频繁，或者你的发布对时间敏感时，这个系统非常有用。越来越多的软件产品使用日历版本控制，包括 Ubuntu ([`ubuntu.com/`](https://ubuntu.com/))。AWS 也在其一些产品中使用了日历版本控制，例如 CloudFormation ([`mng.bz/epQZ`](http://mng.bz/epQZ)) 和 S3 API ([`mng.bz/p6B0`](http://mng.bz/p6B0))。

    CalVer 没有提供关于如何格式化版本的完整规范；它只强调使用日期。一些项目使用 YYYY.MM.DD 的格式，而另一些则使用 YY.MM。如果您每天发布多个版本，您可以使用额外的计数器来跟踪每个版本，例如，2022.12.01.3，这意味着这是在 2022 年 12 月 12 日发布的第三个版本。（有关日历版本化的更多详细信息，请参阅[`mng.bz/O6MO`](http://mng.bz/O6MO)。)

哪种版本控制系统更好？这取决于您的具体需求和整体 API 管理策略。SemVer 因其直观性而被更广泛地使用。然而，如果您的产品发布对时间敏感，CalVer 可能更适合。您选择的版本控制系统也将受到您的版本管理策略的影响，因此让我们看看我们用来表示 API 版本的不同方法：

+   *使用 URL 进行版本控制*—您可以将 API 版本嵌入到 URL 中，例如，[`coffeemesh.com/api/v1/coffee`](https://coffeemesh.com/api/v1/coffee)。这非常方便，因为您的 API 消费者知道他们始终能够调用相同的端点并获得相同的结果。如果您发布 API 的新版本，该版本将进入不同的 URL 路径（/api/v2），因此不会与之前的发布冲突。这也使得您的 API 更容易探索，因为要发现和测试其不同版本，API 消费者只需更改 URL 中的版本字段。然而，在处理 REST API 时，使用 URL 来管理版本被认为违反了 REST 原则，因为每个资源应该只由一个且仅一个 URI 表示。

+   *使用`Accept`头部字段进行版本控制*—API 消费者使用`Accept` HTTP 请求头部字段来声明他们可以解析的内容类型。在 API 的上下文中，`Accept`头部字段的典型值是`application/json`，这意味着客户端只接受 JSON 格式的数据。由于 API 版本也影响我们从服务器接收的内容类型，我们可以使用`Header`字段来声明我们想要使用的 API 版本。一个指定内容类型和 API 版本的`Header`字段示例是`Accept: application/json;v1`。

    这种方法与 REST 原则更为和谐，因为它不修改资源端点，但需要仔细解析。在头部值中引入额外的字符，如以下片段所示，可能导致运行时错误：

    ```
    Accept: application/json; v1  # note the additional space after the 
    ➥ semicolon
    ```

    由于我们使用`Accept`头部，对于 API 版本声明中的任何错误，或者当客户端请求不可用的 API 版本时，我们以 415（不支持的媒体类型）响应。

+   *使用自定义* `Request` `Header` *字段*进行版本控制——在这种方法中，你使用一个自定义的`Request` `Header`字段，例如`Accept-version`，来指定你想要使用的 API 版本。这种方法是最不受欢迎的，因为某些框架可能不接受非标准的`Header`字段，这可能导致与客户的集成问题。

每种版本控制策略都有其自身的优点和挑战。URL 版本控制是最广泛采用的策略，因为它直观且易于使用。然而，在 URL 中指示 API 版本也意味着我们的资源 URI 会根据 API 的版本而变化，这可能会让一些客户感到困惑。

使用`Accept`头信息是另一个流行的选项，但它将处理我们的媒体类型的逻辑与处理我们的 API 版本的逻辑耦合在一起。此外，使用相同的错误状态码来处理媒体类型和 API 版本可能会让我们的 API 客户端感到困惑。最好的策略是仔细考虑我们应用程序的需求，并与你的 API 客户端就最理想的解决方案达成一致。

## B.2 管理你的 API 的生命周期

在本节中，我们研究如何优雅地弃用我们的 API。API 不会永远存在；随着你通过 API 提供的产品和服务的发展和变化，一些 API 将变得过时，你最终将淘汰它们。然而，你可能有一些外部消费者，他们的系统依赖于你的 API，因此你不能随意关闭它们，以免给客户造成干扰。你必须协调你的 API 弃用过程，正如你所看到的，我们使用特定的 HTTP 头来通知 API 的弃用。让我们看看它是如何工作的！

在你退役一个 API 之前，你应该首先弃用它。一个已弃用的 API 仍然在服务中，但它缺乏维护、增强和修复。一旦你弃用了你的 API，你的用户就不会期望它们有进一步的变更。弃用为用户提供了一个宽限期，以便他们有时间将系统迁移到新的 API，而不会干扰他们的操作。

一旦你决定弃用你的 API，你应该通过标准通信渠道，例如通过电子邮件或在通讯稿中，通知你的 API 消费者。同时，你应该在你的响应中设置`Deprecation`头信息。¹ 如果 API 将在未来被弃用，我们将`Deprecation`头信息设置为 API 将被弃用的日期：

```
Deprecation: Friday, 22nd March 2025 23:59:59 GMT
```

一旦 API 被弃用，我们将`Deprecation`头信息设置为`true`：

```
Deprecation: true
```

你还可以使用`Link`头信息来提供有关你的 API 弃用过程的额外信息。例如，你可以提供一个链接到你的弃用策略：

```
Link: <https://coffeemesh.com/deprecation>; rel=”deprecation”; 
➥ type=”text/html”
```

在这种情况下，我们正在告诉用户，他们可以点击[`coffeemesh.com/deprecation`](https://coffeemesh.com/deprecation)链接来找到有关 API 弃用的更多信息。

如果您正在弃用 API 的旧版本，您可以使用 `Link` 标头提供替换或取代当前 API 版本的 URL：

```
Link: <https://coffeemesh.com/v2.0.0/coffee>; rel=”successor-version”
```

除了广播您的 API 弃用信息外，您还应该宣布 API 将何时被弃用。我们使用 `Sunset` 标头来表示 API 将何时被弃用：²。

```
Sunset: Friday, 22nd June 2025 23:59:59 GMT
```

`Sunset` 标头的日期必须晚于或与 `Deprecation` 标头中给出的日期相同。一旦您弃用了一个 API，您必须让您的 API 客户知道旧端点不再可用。当用户调用旧 API 时，您可以使用任何 3xx 和 4xx 状态码的组合。一个好的选择是 410（已消失）状态码。我们使用 410 状态码来表示请求的资源因已知原因不再存在。在某些情况下，301（永久移动）可能是有用的。我们使用 301 状态码来表示请求的资源已被分配了新的 URI，因此当您将 API 迁移到新的端点时，这可能是有用的。

正确管理 API 变更和弃用是确保提供高质量和可靠 API 集成的一个关键但常常被忽视的要素。通过应用本附录中的建议，您将能够有信心地演进您的 API，而不会破坏与客户的集成。

* * *

¹ Sanjay Dalal 和 Erik Wilde，“The Deprecation HTTP Header Field，”[`datatracker.ietf.org/doc/html/draft-ietf-httpapi-deprecation-header-02`](https://datatracker.ietf.org/doc/html/draft-ietf-httpapi-deprecation-header-02)。

² Erik Wilde，“The Sunset HTTP Header Field，”RFC 8594，[`tools.ietf.org/html/rfc8594`](https://tools.ietf.org/html/rfc8594)。
