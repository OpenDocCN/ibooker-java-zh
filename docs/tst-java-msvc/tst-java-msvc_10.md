## 附录。使用 Arquillian Chameleon 隐藏多个容器

对多个容器（如 WildFly、Glassfish、Tomcat 等）进行测试，或者在不同模式（管理、远程或嵌入式）之间切换，不可避免地会导致 pom.xml 文件膨胀。您会遇到以下问题：

+   您需要为每个容器和您想要测试的模式注册特定的依赖项，这也需要几个配置文件。

+   在管理容器的情况下，您需要下载、安装，并且可能还需要配置应用程序服务器。

*Chameleon 容器* ([`github.com/arquillian/arquillian-container-chameleon`](https://github.com/arquillian/arquillian-container-chameleon)) 可以快速适应您的需求，而无需进行额外的依赖项配置。要使用这种方法，就像在 Arquillian 中通常所做的那样，但将 Chameleon 容器添加到 pom.xml 而不是特定应用程序服务器的组件中：

```
<dependency>
  <groupId>org.jboss.arquillian.junit</groupId>
  <artifactId>arquillian-junit-container</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.arquillian.container</groupId>
  <artifactId>arquillian-container-chameleon</artifactId>
  <version>1.0.0.Beta2</version>
  <scope>test</scope>
</dependency>
```

然后，将此配置添加到 arquillian.xml 文件中：

```
<container qualifier="chameleon" default="true">
  <configuration>
    <property name="chameleonTarget">
    wildfly:9.0.0.Final:managed</property>                          *1*
    <property name="serverConfig">standalone-full.xml</property>    *2*
  </configuration>
</container>
```

+   ***1* 选择容器、版本和模式**

+   ***2* 适配器的特定属性**

此示例告诉 Arquillian 使用管理模式的 WildFly 9.0.0。当运行测试时，Chameleon 会检查配置的容器是否已安装；如果没有安装，Chameleon 会下载并安装它。然后，Chameleon 在类路径上注册所需的底层适配器。之后，容器启动，并正常执行测试。

注意，任何设置的属性都会传递给底层的适配器。这意味着使用 Chameleon，您可以使用在适配器中有效的任何属性。

您可以在 [`github.com/arquillian/arquillian-container-chameleon`](https://github.com/arquillian/arquillian-container-chameleon) 上了解更多关于 Arquillian Chameleon 以及使用它的优势。
