# 13 Docker 化微服务 API

本章涵盖了

+   如何 Docker 化一个应用程序

+   如何运行 Docker 容器

+   如何使用 Docker Compose 运行应用程序

+   将 Docker 镜像发布到 AWS 弹性容器注册库

Docker 是一种虚拟化技术，它允许我们通过简单地拥有 Docker 执行运行时在任何地方运行我们的应用程序。Docker 去除了调整和配置环境以运行代码所需的所有痛苦和努力。它还使部署更加可预测，因为它产生可复制的工件（容器镜像），我们可以在本地以及云中运行。

在本章中，你将学习如何将 Python 应用程序 Docker 化。Docker 化是将应用程序打包成 Docker 镜像的过程。你可以将 Docker 镜像视为一个构建或工件，它已准备好部署和执行。要执行镜像，Docker 会创建镜像的运行实例，称为 *容器*。为了部署 Docker 镜像，我们通常使用容器编排器，如 Kubernetes，它负责管理容器的生命周期。在下一章中，你将学习如何使用 Kubernetes 部署 Docker 构建。我们将演示如何使用 CoffeeMesh 平台的订单服务来 Docker 化应用程序。你还将学习如何通过将镜像上传到 AWS 的弹性容器注册库（ECR）来发布你的 Docker 构建。

所有代码示例都可在本书 GitHub 仓库的 ch13 文件夹中找到。我们将从设置环境开始，以便在 13.1 节中处理本章内容。

## 13.1 设置本章环境

在本节中，我们设置环境，以便你可以跟随本章其余部分的示例。我们继续实现订单服务，其中我们在第十一章中停止，那时我们添加了身份验证和授权层。首先，将第十一章的代码复制到一个名为 ch13 的新文件夹中：

```
$ cp -r ch11 ch13
```

在 ch13 中 `cd`，并运行以下命令安装依赖项并激活虚拟环境：

```
$ cd ch13 && pipenv install --dev && pipenv shell
```

当我们部署应用程序时，我们使用 PostgreSQL 引擎，这是在生产环境中运行应用程序最受欢迎的 SQL 引擎之一。为了与数据库通信，我们使用 `psycopg2`，这是 Python 最受欢迎的 PostgreSQL 驱动之一：

```
$ pipenv install psycopg2
```

安装 PSYCOPG2 如果你在安装和编译 `psycopg2` 时遇到问题，尝试通过运行 `pipenv install psycopg2-binary` 安装编译好的包，或者从本书的 GitHub 仓库中拉取 ch13/Pipfile 和 ch13/Pipfile.lock，然后运行 `pipenv install --dev`。还有两个强大的 PostgreSQL 驱动程序是 `asyncpg` ([`github.com/MagicStack/asyncpg`](https://github.com/MagicStack/asyncpg)) 和 `pscycopg3` ([`github.com/psycopg/psycopg`](https://github.com/psycopg/psycopg))，它们都支持异步操作。我鼓励你检查它们！

要构建和运行 Docker 容器，您需要在您的机器上安装 Docker 运行时。安装说明因平台而异，请参阅官方文档了解如何在您的系统上安装 Docker ([`docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/))。

由于我们打算将 Docker 镜像发布到 AWS 的 ECR，我们需要安装 AWS CLI：

```
$ pipenv install --dev awscli
```

接下来，访问 [`aws.amazon.com/`](https://aws.amazon.com/)。创建一个 AWS 账户并获取一个访问密钥，以便能够以编程方式访问 AWS 服务。您用于创建 AWS 账户的用户配置文件是账户的根用户。出于安全考虑，建议您不要使用根用户生成您的访问密钥。相反，创建一个 IAM 用户并为该用户生成一个访问密钥。IAM 是 AWS 的身份访问管理服务，它允许您创建用户、角色和细粒度的策略，以授予对您账户中其他服务的访问权限。按照 AWS 文档了解如何创建 IAM 用户 ([`mng.bz/neP8`](http://mng.bz/neP8)) 以及如何生成您的访问密钥和配置 AWS CLI ([`mng.bz/vXxq`](http://mng.bz/vXxq))。

现在我们已经准备好了环境，是时候将我们的应用程序 Docker 化了！

## 13.2 Docker 化微服务

将应用程序 Docker 化意味着什么？*Docker 化*是将应用程序打包成 Docker 镜像的过程。您可以将 Docker 镜像想象为一个可以在 Docker 运行时部署和执行的构建或工件。所有系统依赖项都已安装在 Docker 镜像中，要运行镜像，我们只需要一个 Docker 运行时。要执行镜像，Docker 运行时会创建一个容器，这是镜像的运行实例。如图 13.1 所示，使用 Docker 非常方便，因为它允许我们在隔离进程中运行我们的应用程序。根据您的平台，安装 Docker 运行时有不同的选项，请参阅官方文档以确定哪个选项最适合您 ([`docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/))。

![](img/13-01.png)

图 13.1 Docker 容器在主机操作系统之上运行在隔离进程中。

在本节中，我们创建了一个优化后的订单服务的 Docker 镜像。在这个过程中，您将学习如何编写 Dockerfile，这是一个包含构建 Docker 镜像所需所有指令的文档。您还将学习如何运行 Docker 容器，以及如何将容器端口映射到主机操作系统，以便您能够与容器内运行的应用程序交互。最后，您还将学习如何使用 Docker CLI 管理容器。

Docker 基础知识 如果你想了解更多关于 Docker 如何工作以及它与宿主操作系统的交互方式，请查看 Prabath Siriwardena 和 Nuwan Dias 在他们所著的《Microservices Security in Action》（Manning，2020）一书中优秀的“Docker 基础知识”部分。[`mng.bz/49Ag`](http://mng.bz/49Ag)。

在构建镜像之前，我们需要对我们的应用程序代码进行两项小的修改，以便为部署做好准备。到目前为止，订单服务一直在使用硬编码的数据库 URL，但为了在不同的环境中运行服务，我们需要使这个设置可配置。以下代码展示了需要修改`orders/repository/unit_of_work.py`文件以从环境中获取数据库 URL 的更改，其中新添加的代码以粗体字符显示。我们使用断言语句，如果未提供数据库 URL，则立即退出应用程序。

列表 13.1 从环境中获取数据库 URL

```
# file: orders/repository/unit_of_work.py

import os

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

DB_URL = os.getenv('DB_URL')                                             ①

assert DB_URL is not None, 'DB_URL environment variable needed.'         ②

class UnitOfWork:
    def __init__(self):
        self.session_maker = sessionmaker(bind=create_engine(DB_URL))    ③

    def __enter__(self):
        self.session = self.session_maker()
        return self

    ...
```

① 从 DB_URL 环境变量中获取数据库 URL。

② 如果未设置 DB_URL，则退出应用程序。

③ 使用 DB_URL 的值连接到数据库。

我们还需要更新我们的 Alembic 文件，以便从环境中获取数据库 URL。以下代码展示了需要修改`migrations/env.py`以实现此目的的更改，其中新添加的代码以粗体显示。我们使用省略号省略了代码的非相关部分，以便更容易观察更改。

列表 13.2 从环境中获取`alembic`的数据库 URL

```
# file: migrations/env.py

import os
from logging.config import fileConfig

from sqlalchemy import create_engine
from sqlalchemy import pool

from alembic import context

...
def run_migrations_online():
    """...
    """

    url = os.getenv('DB_URL')                                        ①

    assert url is not None, 'DB_URL environment variable needed.' ②

    connectable = create_engine(url)

    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    ...
```

① 从 DB_URL 环境变量中获取数据库 URL。

② 如果未设置 DB_URL，则退出应用程序。

现在代码已经准备好了，是时候将其 Docker 化了！要构建一个 Docker 镜像，我们需要编写一个 Dockerfile。创建一个名为 Dockerfile 的文件。表 13.3 显示了该文件的包含内容。我们使用官方 Python 3.9 Docker 镜像的精简版作为我们的基础镜像。精简镜像只包含运行我们的应用程序所需的依赖项，这导致了更轻的镜像。要使用基础镜像，我们使用 Docker 的`FROM`指令。然后我们创建一个名为/orders/orders 的应用程序代码文件夹。要运行 bash 命令，例如本例中的`mkdir`，我们使用 Docker 的`RUN`指令。我们还使用 Docker 的`WORKDIR`指令将/orders/orders 设置为工作目录。工作目录是应用程序运行时的目录。

接下来，我们安装 `pipenv`，复制我们的 Pipenv 文件，并安装依赖项。我们使用 Docker 的 `COPY` 指令将文件从我们的文件系统复制到 Docker 镜像中。由于我们在 Docker 中运行，我们不需要虚拟环境，所以我们使用 `pipenv` 的 `--system` 标志来安装依赖项。我们还使用 `pipenv` 的 `--deploy` 标志来检查我们的 Pipenv 文件是否是最新的。最后，我们复制我们的源代码并指定需要执行的命令以启动订单服务。Docker 必须使用 Docker 的 `CMD` 指令来执行我们的应用程序。我们还使用 Docker 的 `EXPOSE` 指令确保正在运行的容器监听端口 8000，这是我们的 API 运行的端口。如果我们不暴露端口，我们就无法与 API 交互。

我们在 Dockerfile 中的语句顺序很重要，因为 Docker 会缓存构建的每个步骤。如果上一个步骤发生了变化，例如，如果我们安装了新的依赖项，或者我们的某个文件发生了变化，Docker 才会再次执行一个步骤。由于我们的应用程序代码可能比我们的依赖项变化更频繁，所以我们将在构建的最后复制代码。这样，Docker 将只安装一次依赖项并缓存该步骤，直到它们发生变化。

列表 13.3 订单服务的 Dockerfile

```
# file: Dockerfile

FROM python:3.9-slim                                         ①

RUN mkdir -p /orders/orders                                  ②

WORKDIR /orders                                              ③

RUN pip install -U pip && pip install pipenv

COPY Pipfile Pipfile.lock /orders/                           ④

RUN pipenv install --system --deploy                         ⑤

COPY orders/orders_service /orders/orders/orders_service/    ⑥
COPY orders/repository /orders/orders/repository/
COPY orders/web /orders/orders/web/
COPY oas.yaml /orders/
COPY public_key.pem /orders/public_key.pem
COPY private.pem /orders/private.pem

EXPOSE 8000                                                  ⑦

CMD ["uvicorn", "orders.web.app:app", "--host", "0.0.0.0"]   ⑧
```

① 基础镜像

② 我们应用程序的基本文件夹结构

③ 我们将运行代码的工作目录

④ 我们复制我们的 pipenv 文件。

⑤ 我们安装依赖项。

⑥ 我们复制应用程序的其他文件。

⑦ 我们将应用程序的端口暴露给主机机器。

⑧ API 服务器的启动命令

要从列表 13.3 构建 Docker 镜像，你需要从 ch13 目录运行以下命令：

```
$ docker build -t orders:1.0 .
```

`-t` 标志代表 *标签*。Docker 标签有两个部分：冒号左侧的镜像名称和冒号右侧的标签名称。标签名称通常是构建的版本。在这种情况下，我们命名镜像为 orders 并将其标记为 `1.0`。确保你不要漏掉构建语句末尾的点：它代表构建的源代码路径（在 Docker 术语中称为 *上下文*）。一个点表示当前目录。

一旦镜像构建完成，你可以使用以下命令来执行它：

```
$ docker run --env DB_URL=sqlite:///orders.db \
-v $(pwd)/orders.db:/orders/orders.db -p 8000:8000 -it orders:1.0
```

如你在图 13.2 中所见，`--env` 标志允许我们在容器中设置环境变量，我们用它来设置数据库的 URL。为了使应用程序对主机机器可访问，我们使用 `-p` 标志，它允许我们将容器内应用程序运行的端口绑定到主机机器上的端口。我们还使用 `-v` 标志将卷挂载到 SQLite 数据库文件上。Docker 卷允许容器从主机机器的文件系统中访问文件。

![图片](img/13-02.png)

图 13.2 当我们运行容器时，我们可以包含各种配置来在容器内设置环境变量，或者允许它访问宿主操作系统的文件。

您现在可以通过以下 URL 访问应用程序：http://127.0.0.1:8000/docs/orders。之前的命令执行了连接到您当前终端会话的容器，这使得您可以在与应用程序交互时看到日志展开。在这种情况下，您可以通过按 Ctrl-C 组合键来停止容器，就像停止任何其他进程一样。

您还可以以分离模式运行容器，这意味着进程没有连接到您的终端会话，因此当您关闭终端时，进程将继续运行。如果您只想运行容器与之交互，而不需要查看日志，这很方便。我们通常以分离模式运行容器化的数据库。要分离模式运行容器，您使用 `-d` 标志：

```
$ docker run -d –-env DB_URL=sqlite:///orders.db \
-v $(pwd)/orders.db:/orders/orders.db -p 8000:8000 orders:1.0
```

在这种情况下，您需要使用 `docker` 的 `stop` 命令停止容器。首先，您需要使用以下命令找出运行中容器的 ID：

```
$ docker ps
```

此命令将列出您机器上当前运行的所有容器。输出看起来像这样（输出已使用省略号截断）：

```
CONTAINER ID   IMAGE       COMMAND       CREATED         STATUS...    
83e6189a02ee   orders:1.0  "uvicorn..."  7 seconds ago   Up 6 seconds
```

拿到容器 ID（在本例中为 `83e6189a02ee`），然后使用以下命令停止进程：

```
$ docker stop 83e6189a02ee
```

构建和运行 Docker 容器只需这些步骤！Docker 的功能远不止我们在本节中看到的，如果您想了解更多关于这项技术的信息，我推荐您阅读 Ian Miell 和 Aidan Hobson Sayers 所著的 *Docker in Practice*（Manning, 2019）以及 Jeff Nickoloff 和 Stephen Kuenzli 所著的 *Docker in Action*（Manning, 2019）。

## 13.3 使用 Docker Compose 运行应用程序

在上一节中，我们通过将其挂载到我们的本地 SQLite 数据库上来运行了订单服务的容器。这对于快速测试来说是可以的，但它并不能真正告诉我们我们的应用程序是否能够像预期的那样与 PostgreSQL 数据库一起工作。将我们的容器化应用程序连接到数据库的常见策略是使用 Docker Compose，它允许我们在共享网络中运行多个容器，这样它们就可以相互通信。在本节中，您将学习如何使用 `docker-compose` 运行带有 PostgreSQL 数据库的订单服务。

要使用 Docker Compose，首先我们需要安装它。它是一个 Python 包，所以我们使用 `pip` 来安装它：

```
$ pip install docker-compose
```

接下来，让我们编写我们的 Docker Compose 文件——声明我们运行应用程序所需的资源。列表 13.4 显示了订单服务的 `docker-compose` 文件。我们使用 Docker Compose 的最新规范格式，版本 3.9，并声明了两个服务：`database` 和 `api`。`database` 运行 PostgreSQL 的官方 Docker 镜像，而 `api` 运行订单服务。我们使用 `build` 关键字指向 Docker 构建上下文，并给它一个点值（`.`）。通过使用点，我们指示 Docker Compose 在当前目录中查找 Dockerfile 并构建镜像。通过 `environment` 关键字，我们配置运行我们的应用程序所需的环境变量。我们暴露 `database` 的 5432 端口，以便我们可以从我们的主机机器连接到数据库，以及 `api` 的 8000 端口，以便我们可以访问 API。最后，我们使用一个名为 `database-data` 的卷，`docker-compose` 将使用它来持久化我们的数据。这意味着如果你重启 `docker-compose`，你不会丢失你的数据。

列表 13.4 为订单服务的 `docker-compose` 文件

```
# file: docker-compose.yaml

version: "3.9"                               ①

services:                                    ②

  database:                                  ③
    image: postgres:14.2                     ④
    ports:                                   ⑤
      - 5432:5432
    environment:                             ⑥
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
    volumes:                                 ⑦
      - database-data:/var/lib/postgresql/data

  api:                                       ⑧
    build: .                                 ⑨
    ports:                                   ⑩
      - 8000:8000
    depends_on:                              ⑪
      - database
    environment:                             ⑫
      DB_URL: postgresql://postgres:postgres@database:5432/postgres

volumes:                                     ⑬
  database-data:
```

① 此文件的 docker-compose 格式版本。

② 我们声明我们的服务。

③ 数据库服务

④ 数据库服务的 Docker 镜像

⑤ 我们将数据库端口暴露给主机机器。

⑥ 数据库环境配置

⑦ 我们将数据库的数据文件夹挂载到本地卷上。

⑧ API 服务

⑨ API 的构建上下文

⑩ 我们将 API 的端口暴露给主机机器。

⑪ API 依赖于数据库。

⑫ API 的环境配置

⑬ 数据库的卷

执行以下命令以运行我们的 Docker Compose 文件：

```
$ docker-compose up --build
```

`--build` 标志指示 Docker Compose 在你的文件更改时重新构建你的镜像。一旦 Web API 启动并运行，你可以在 http://localhost:8000/docs/orders 上访问它。如果你尝试任何端点，你的表不存在。那是因为我们没有在我们的新 PostgreSQL 数据库上运行迁移！要运行迁移，打开一个新的终端窗口，`cd` 进入 ch13 文件夹，激活你的 `pipenv` 环境，并运行以下命令：

```
$ PYTHONPATH=`pwd` \
DB_URL=postgresql://postgres:postgres@localhost:5432/postgres alembic \
upgrade heads
```

一旦应用了迁移，你就可以再次点击 API 端点，一切应该都会正常工作。要停止 `docker-compose`，请在另一个终端窗口中，在 ch13 文件夹内运行以下命令：

```
$ docker-compose down
```

这就是运行 Docker Compose 所需的全部！你已经学会了使用最强大的自动化工具之一。Docker Compose 经常用于运行集成测试，并为在客户端应用程序（如 SPAs）上工作的开发者提供一个简单的方式来运行后端。

当我们的 Docker 栈准备就绪且我们的镜像经过测试后，是时候学习如何将镜像推送到容器注册库了。继续阅读下一节，了解如何操作！

## 13.4 将 Docker 构建发布到容器注册库

要部署我们的 Docker 构建，我们需要首先将它们发布到一个 Docker 容器注册库。容器注册库是 Docker 镜像的存储库。在下一章中，我们将部署我们的应用程序到 AWS 的 Elastic Kubernetes 服务，因此我们将我们的构建发布到 AWS 的 ECR。将我们的 Docker 镜像保留在 AWS 中将使它们部署到 EKS 更容易。

首先，让我们使用以下命令为我们的镜像创建一个 ECR 仓库：

```
$ aws ecr create-repository --repository-name coffeemesh-orders
{
    "repository": {
        "repositoryArn": 
➥ "arn:aws:ecr:<aws_region>:<aws_account_id>:repository/coffeemesh-orders",
        "registryId": "876701361933",
        "repositoryName": "coffeemesh-orders",
        "repositoryUri": 
➥ "<aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders",
        "createdAt": "2021-11-16T10:08:42+00:00",
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        },
        "encryptionConfiguration": {
            "encryptionType": "AES256"
        }
    }
}
```

在此命令中，我们创建了一个名为 `coffeemesh-orders` 的 ECR 仓库。命令的输出是一个描述我们刚刚创建的仓库的有效负载。当你运行此命令时，输出有效负载中的 `<aws_account_id>` 占位符将包含你的 AWS 账户 ID，而 `<aws_region>` 将包含你的默认 AWS 区域。要将我们的 Docker 构建发布到 ECR，我们需要用 ECR 仓库的名称标记我们的构建。获取上一条命令输出中的 `repository.repositoryArn` 属性（加粗），并使用它来标记我们在 13.2 节中创建的 Docker 构建，如下所示：

```
$ docker tag orders:1.0 \
<aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders:1.0
```

要将我们的镜像发布到 ECR，我们需要使用以下命令获取登录凭证：

```
$ aws ecr get-login-password --region <aws_region> | docker login \
--username AWS --password-stdin \ 
<aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

确保将此命令中的 `<aws_region>` 替换为你创建 Docker 仓库的 AWS 区域，例如欧洲（爱尔兰）的 eu-west-1 或美国东部（俄亥俄州）的 us-east-2。同时将 `<aws_account_id>` 替换为你的 AWS 账户 ID。查看 AWS 文档了解如何找到你的 AWS 账户 ID ([`mng.bz/Qnye`](http://mng.bz/Qnye))。

AWS 区域 当你在 AWS 上部署服务时，你将它们部署到特定的区域。每个区域都有一个标识符，例如爱尔兰的 eu-west-1 或俄亥俄州的 eu-east-2。有关 AWS 中可用区域的最新列表，请参阅 [`mng.bz/XaPM`](http://mng.bz/XaPM)。

`aws ecr get-login-password` 命令生成一个 Docker 可以使用的指令来登录到 ECR。我们现在已经准备好发布我们的构建了！运行以下命令将镜像推送到 ECR：

```
$ docker push \
<aws_account_id>.dkr.ecr.<aws_region>.amazonaws.com/coffeemesh-orders:1.0
```

哇！我们的 Docker 构建现在已经在 ECR 中了。在下一章中，你将学习如何将此构建部署到 AWS 的 Kubernetes 集群。

## 概述

+   Docker 是一种虚拟化技术，它允许我们通过简单地拥有一个 Docker 执行运行时，在任何地方运行我们的应用程序。Docker 构建被称为镜像，它是在称为 Docker 容器的进程中执行的。

+   Docker Compose 是一个容器编排框架，它允许你同时运行多个容器，例如数据库和 API。使用 Docker Compose 是运行整个后端的一种简单而有效的方式，无需安装和配置额外的依赖项。

+   要部署 Docker 镜像，我们需要将它们发布到一个容器注册库，例如 AWS 的 ECR——一个强大且安全的容器注册库，它使得将我们的容器部署到 AWS 服务变得容易。
