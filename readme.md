# Spring Testing --- version 5.0.6.release

Spring团队提倡采用测试驱动开发（TDD）方法来开发软件，因此Spring所有代码都涵盖了对集成测试的支持（以及单元测试的最佳实践）。 Spring团队发现，正确使用IoC确实可以使单元测试和集成测试变得更容易（因为setter方法和相应的类构造函数的存在使得它们更容易在测试中连接在一起，而无需设置服务定位器注册表 等等）......专门用于测试的章节也希望能够让你相信这一点。

## 1.Spring测试简介

测试是企业软件开发不可或缺的一部分。 本章重点讨论IoC原理对单元测试的增值以及Spring Framework对集成测试支持的益处。 （企业对测试的彻底处理超出了本参考手册的范围。）

## 2.单元测试

依赖注入应该使你的代码对容器的依赖程度低于传统的Java EE开发。 构成应用程序的POJO应该可以在JUnit或TestNG测试中测试，对象只需使用新运算符实例化，不需要Spring或任何其他容器。 您可以使用模拟对象（连同其他有价值的测试技术）来独立测试您的代码。 如果您遵循Spring的体系结构建议，您的代码库所产生的清晰分层和组件化将有助于更轻松地进行单元测试。 例如，您可以通过存根或模拟DAO或存储库接口来测试服务层对象，而无需在运行单元测试时访问持久数据。

真正的单元测试通常运行速度非常快，因为没有运行时基础架构要设置。 将真正的单元测试强调为开发方法的一部分将会提高您的生产力。 您可能不需要测试章节的这一部分来帮助您为基于IoC的应用程序编写有效的单元测试。 但是，对于某些单元测试场景，Spring Framework提供了以下模拟对象和测试支持类。

### 2.1 模拟对象

- [2.1.1 环境](#2.1.1 环境)
- [2.1.2 JNDI](#2.1.2 JNDI)
- [2.1.3 Servlet API](#2.1.3 Servlet API)
- [2.1.4 响应式 Web](#2.1.4 响应式 Web)

#### 2.1.1 环境

org.springframework.mock.env包中包含Environment和PropertySource抽象的模拟实现（请参阅Bean定义配置文件和PropertySource抽象）。 MockEnvironment和MockPropertySource对于开发依赖于环境特定属性的代码的容器外测试很有用。

#### 2.1.2 JNDI

org.springframework.mock.jndi包中包含JNDI SPI的实现，您可以使用该实现为测试套件或独立应用程序设置简单的JNDI环境。 例如，如果JDBC DataSources在测试代码中与Java EE容器中的JNDI名称绑定到相同的JNDI名称，则可以在测试场景中重用应用程序代码和配置，而无需进行修改。

#### 2.1.3 Servlet API

org.springframework.mock.web包中包含一组全面的Servlet API模拟对象，这些对象可用于测试Web上下文，控制器和过滤器。 这些模拟对象的目标是使用Spring的Web MVC框架，并且通常比动态模拟对象（如EasyMock）或替代Servlet API模拟对象（如MockObjects）更方便使用。

> 自Spring Framework 5.0之后，org.springframework.mock.web中的模拟对象基于Servlet 4.0 API。

Spring MVC测试框架基于模拟的Servlet API对象，为Spring MVC提供了一个集成测试框架。 请参阅Spring MVC测试。

#### 2.1.4 响应式 Web

包org.springframework.mock.http.server.reactive包含用于WebFlux应用程序的ServerHttpRequest和ServerHttpResponse的模拟实现。包org.springframework.mock.web.server包含一个模拟的ServerWebExchange，它依赖于那些模拟的请求和响应对象。

MockServerHttpRequest和MockServerHttpResponse都从相同的抽象基类扩展到特定于服务器的实现，并与他们共享行为。例如，模拟请求在创建后是不可变的，但您可以使用ServerHttpRequest中的mutate（）方法创建修改后的实例。

为了使模拟响应正确执行写入合同并返回写入完成句柄（即Mono <Void>），默认情况下，它使用带有缓存（）。然后（）的Flux，缓存数据并使其可用于测试中的断言。应用程序可以设置自定义写入功能，例如测试无限流。

WebTestClient建立在模拟请求和响应之上，为在没有HTTP服务器的情况下测试WebFlux应用程序提供支持。客户端也可以用于运行服务器的端到端测试。

### 2.2 单元测试支持类

- [2.2.1 一般测试工具](#2.2.1 一般测试工具)
- [2.2.2 Spring MVC](#2.2.2 Spring MVC)

#### 2.2.1 一般测试工具

org.springframework.test.util包中包含几个用于单元和集成测试的通用实用程序。

ReflectionTestUtils是基于反射的实用程序方法的集合。开发人员在测试需要更改常量值，设置非公用字段，调用非公共配置方法或在测试涉及使用的应用程序代码时调用非公共配置或生命周期回调方法的场景中使用这些方法以下情况。

- 诸如JPA和Hibernate之类的ORM框架允许私有或受保护的字段访问，而不是域实体中属性的公共setter方法。

- Spring对@Autowired，@Inject和@Resource等注释提供支持，它为私有或受保护字段，setter方法和配置方法提供依赖注入。

- 用于生命周期回调方法的批注（如@PostConstruct和@PreDestroy）。

AopTestUtils是AOP相关实用程序方法的集合。这些方法可用于获取对隐藏在一个或多个Spring代理后面的基础目标对象的引用。例如，如果您已经使用EasyMock或Mockito等库将bean配置为动态模拟，并且将模拟包装在Spring代理中，则可能需要直接访问基础模拟以配置对它的期望并执行验证。对于Spring的核心AOP实用程序，请参阅AopUtils和AopProxyUtils。

#### 2.2.2 Spring MVC

org.springframework.test.web包中包含ModelAndViewAssert，您可以将其与JUnit，TestNG或任何其他测试框架结合使用，以用于处理Spring MVC ModelAndView对象的单元测试。

>单元测试Spring MVC控制器
要将Spring MVC控制器作为POJO进行单元测试，请将ModelAndViewAssert与Spring的Servlet API mock中的MockHttpServletRequest，MockHttpSession等结合使用。 对于Spring MVC和REST控制器的彻底集成测试，以及Spring MVC的WebApplicationContext配置，请改用Spring MVC测试框架。

## 3.集成测试

- [3.1 概述](#3.1概述)
- [3.2 集成测试的目的](#3.2 集成测试的目的)
- [3.3 JDBC测试支持]()
- [3.4 注解]()
- [3.5 Spring测试上下文框架]()
- [3.6 Spring MVC测试框架]()
- [3.7 Web测试客户端]()
- [3.8 宠物诊所的例子]()

### 3.1概述

能够执行一些集成测试而无需部署到应用程序服务器或连接到其他企业基础架构非常重要。这将使您能够测试以下内容：

- Spring IoC容器上下文的正确接线。
- 使用JDBC或ORM工具访问数据。这将包括诸如SQL语句的正确性，Hibernate查询，JPA实体映射等等。

Spring框架为Spring测试模块中的集成测试提供了一流的支持。实际的JAR文件的名称可能包含发布版本，也可能位于 org.springframework.test 长表单中，具体取决于您从何处获得它（请参阅“依赖管理”一节以获取解释）。该库包含org.springframework.test包，其中包含用于与Spring容器进行集成测试的有价值的类。此测试不依赖于应用程序服务器或其他部署环境。这样的测试运行速度比单元测试要慢，但要比等效的Selenium测试或依赖部署到应用程序服务器的远程测试快得多。

在Spring 2.5和更高版本中，单元和集成测试支持以注解驱动的Spring测试环境框架的形式提供。 测试环境框架不受所使用的实际测试框架的影响，因此允许在各种环境中进行测试，包括JUnit，TestNG等等。

### 3.2 集成测试的目的

Spring的集成测试支持具有以下主要目标：
- 在测试执行之间管理Spring IoC容器缓存。
- 提供测试实例的依赖注入。
- 提供适合集成测试的事务管理。
- 提供特定于Spring的基类，以帮助开发人员编写集成测试。
接下来的几节将介绍每个目标并提供实施和配置详细信息的链接。

#### 3.2.1 上下文管理和缓存

Spring测试上下文框架提供了Spring应用上下文和Web应用上下文的一致加载以及这些上下文的缓存。支持缓存加载的上下文很重要，因为启动时间可能成为一个问题 - 不是因为Spring本身的开销，而是因为Spring容器实例化的对象需要时间来实例化。例如，一个包含50到100个Hibernate映射文件的项目可能需要10到20秒才能加载映射文件，并且在每个测试夹具中运行每个测试之前产生的成本会导致整体测试运行速度降低，从而降低开发人员的生产力。

测试类通常为XML或Groovy配置元数据声明一个资源位置的数组（通常位于类路径中），或者用于配置应用程序的一组注释类。这些位置或类与用于生产部署的web.xml或其他配置文件中指定的位置或类相同或相似。

默认情况下，一旦加载，为每个测试重新使用配置的应用程序上下文。因此，每个测试套件只需支付一次设置成本，并且后续的测试执行速度要快得多。在这种情况下，术语测试套件意味着所有测试都运行在相同的JVM中 - 例如，对于给定的项目或模块，所有测试都是从Ant，Maven或Gradle构建运行的。在测试破坏应用程序上下文并需要重新加载的情况下（例如，通过修改bean定义或应用程序对象的状态），测试上下文框架可以配置为重新加载配置并重建应用程序上下文，然后再执行下一个测试。

使用测试上下文框架查看上下文管理和上下文缓存。

#### 3.2.2 测试的依赖注入

当测试上下文框架加载您的应用程序上下文时，它可以通过依赖注入选择性地配置测试类的实例。这为从应用程序上下文中使用预先配置的bean设置测试装置提供了一种便捷的机制。这里的强大优势在于，您可以在各种测试场景中重复使用应用程序上下文（例如配置Spring管理的对象图，事务代理，数据源等），从而避免需要针对单个测试案例复制复杂的测试夹具设置。

作为一个例子，考虑一下我们有一个类HibernateTitleRepository的场景，它实现了Title域实体的数据访问逻辑。我们希望编写测试以下领域的集成测试：

- Spring配置：基本上，与HibernateTitleRepository bean的配置相关的所有内容现在是不是仍然正确？
- Hibernate映射文件配置：是否正确映射了所有内容，并且是正确的延迟加载设置？
- HibernateTitleRepository的逻辑：这个类的配置实例是否按预期执行？

使用测试上下文框架查看测试的依赖注入。

#### 3.2.3 事务管理

访问真实数据库的测试中的一个常见问题是它们对持久性存储的状态的影响。即使在使用开发数据库时，对状态的更改也可能影响未来的测试。此外，许多操作（例如插入或修改持久数据）不能在事务外执行（或验证）。

测试上下文框架解决了这个问题。默认情况下，框架将为每个测试创建并回滚事务。您只需编写可以假定交易存在的代码。如果您在测试中调用事务代理对象，则它们将根据其配置的事务语义正确行为。另外，如果测试方法在为测试管理的事务内运行时删除所选表的内容，那么事务将默认回滚，并且数据库将在执行测试之前返回到其状态。事务支持通过在测试的应用程序上下文中定义的PlatformTransactionManager bean提供给测试。

如果你想要一个事务提交 -虽然这很不寻常，但是当你想要一个特定的测试来填充或修改数据库时偶尔有用 - 可以指示测试上下文框架使事务提交而不是通过@Commit注释回滚。

使用测试上下文框架查看事务管理。

#### 3.2.4 支持集成测试的类

Spring框架测试上下文提供了几个抽象支持类，可以简化集成测试的编写。 这些基础测试类为测试框架提供了定义良好的钩子以及方便的实例变量和方法，使您能够访问：

- 应用程序上下文用于执行显式bean查找或测试整个上下文的状态。
- 一个JdbcTemplate，用于执行SQL语句来查询数据库。 此类查询可用于在执行与数据库相关的应用程序代码之前和之后确认数据库状态，并且Spring确保此类查询在与应用程序代码相同的事务范围内运行。 当与ORM工具一起使用时，一定要避免误报。

另外，您可能希望创建自己的自定义应用程序范围的超类，其中包含特定于项目的实例变量和方法。

请参阅测试上下文框架的支持类。

### 3.3 JDBC测试支持

org.springframework.test.jdbc包中包含JdbcTestUtils，它是JDBC相关实用程序函数的集合，旨在简化标准数据库测试场景。具体而言，JdbcTestUtils提供以下静态实用程序方法。

- countRowsInTable（..）：计算给定表中的行数
- countRowsInTableWhere（..）：使用提供的WHERE子句对给定表中的行数进行计数
- deleteFromTables（..）：删除指定表中的所有行
- deleteFromTableWhere（..）：使用提供的WHERE子句从给定表中删除行
- dropTables（..）：删除指定的表

请注意，AbstractTransactionalJUnit4SpringContextTests和AbstractTransactionalTestNGSpringContextTests提供了方便的方法，它们委派给JdbcTestUtils中的上述方法。

spring-jdbc模块为配置和启动嵌入式数据库提供支持，该嵌入式数据库可用于与数据库交互的集成测试。有关详细信息，请参阅嵌入式数据库支持和使用嵌入式数据库测试数据访问逻

### 3.4注解

- [3.4.1 Spring测试注解](#3.4.1 Spring测试注解)
- [3.4.2 标准注解支持]()
- [3.4.3 Spring JUnit 4测试注解]()
- [3.4.4 Spring JUnit Jupiter测试注解]()
- [3.4.5 对测试的元注解支持]()

#### 3.4.1 Spring测试注解

Spring框架提供了以下一组Spring特定的注释，您可以在单元测试和集成测试中使用这些注释，并与Test Context框架结合使用。 有关更多信息，请参阅相应的java文档，其中包括默认属性值，属性别名等。

- `@BootstrapWith`

@BootstrapWith是一个用于配置Spring测试环境框架如何启动的类级注释。 具体而言，@ BootstrapWith用于指定自定义测试上下文引导程序。 有关更多详细信息，请参阅引导测试环境框架部分。
- `@ContextConfiguration`

@ContextConfiguration定义了用于确定如何为集成测试加载和配置应用程序上下文的类级别元数据。 具体而言，@ContextConfiguration声明应用程序上下文资源位置或将用于加载上下文的注释类。

资源位置通常是位于类路径中的XML配置文件或Groovy脚本; 而注释类通常是@Configuration类。 但是，资源位置也可以引用文件系统中的文件和脚本，并且带注释的类可以是组件类等。

```Java
@ContextConfiguration("/test-config.xml")
public class XmlApplicationContextTests {
    // class body...
}
```

```Java
@ContextConfiguration(classes = TestConfig.class)
public class ConfigClassApplicationContextTests {
    // class body...
}
```

作为声明资源位置或注释类的替代或补充，@ContextConfiguration可用于声明ApplicationContextInitializer类。

```Java
@ContextConfiguration(initializers = CustomContextIntializer.class)
public class ContextInitializerTests {
    // class body...
}
```

@ContextConfiguration也可以选择用来声明ContextLoader策略。 但是，请注意，通常不需要显式配置加载器，因为默认加载器支持资源位置或注释类以及初始化器。

```Java
@ContextConfiguration(locations = "/test-context.xml", loader = CustomContextLoader.class)
public class CustomLoaderXmlApplicationContextTests {
    // class body...
}
```

> @ContextConfiguration默认支持继承资源位置或配置类以及由超类声明的上下文初始化器。

有关更多详细信息，请参阅上下文管理和@ContextConfiguration javadocs。

- `@WebAppConfiguration`

@WebAppConfiguration是一个类级别的注释，用于声明为集成测试加载的应用程序上下文应该是Web应用程序上下文。 仅在测试类中存在@WebAppConfiguration可确保为测试加载Web应用程序上下文，并使用默认值“file：src / main / webapp”作为Web应用程序根目录的路径（即， 资源库路径）。 资源库路径在幕后用于创建一个模拟Servlet上下文，作为测试的Web应用程序上下文的Servlet上下文。

```Java
@ContextConfiguration
@WebAppConfiguration
public class WebAppTests {
    // class body...
}
```

要覆盖默认值，请通过隐式值属性指定不同的基础资源路径。 支持classpath：和file：资源前缀。 如果没有提供资源前缀，则路径被假定为文件系统资源。

```Java
@ContextConfiguration
@WebAppConfiguration("classpath:test-web-resources")
public class WebAppTests {
    // class body...
}
```
