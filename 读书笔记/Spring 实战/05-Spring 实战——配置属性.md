### 05-Spring 实战——配置属性

#### 自动配置

Spring 中的配置分为两种：

- bean装配：在 Spring 应用上下文中创建哪些应用组件以及它们之间如何互相注入。
- 属性注入：设置 bean 的值。

在 Spring 的 XML 方式中和基于 Java 的配置中，这两种配置通常一起声明：

```xml
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
           <property name="driverClass" value="${jdbc.driver}"/>
           <property name="jdbcUrl" value="${jdbc.url}"/>
</bean>
```

```java
@Bean
public DataSource dataSource() {
    return new EmbeddedDataSourceBuilder()
        .setType(H2)
        .addScript("taco_schema.sql")
        .addScripts("user_data.sql", "ingredient_data.sql")
        .build();
}
```

依赖于 Spring Boot 的自动配置，只要运行时能在类路径下找到 H2 依赖，那么对应的 DataSource bean 会被自动创建，这个 bean 会运行名为 schema.sql 和 data.sql 的脚本。

但是如果我们想给 SQL 脚本使用其他的名称，或者想要指定两个以上的 SQL 脚本，那么配置属性就能够发挥它的作用了。

Spring 环境会拉取多个属性源，需要这些属性的 bean 就可以从 Spring 本身中获取了。

- JVM 系统属性；
- 操作系统环境变量
- 命令行参数
- 应用属性配置文件

![Spring 环境拉取属性](https://github.com/1874yin/Java-Notes/blob/main/Pics/Spring%E7%8E%AF%E5%A2%83%E4%BB%8E%E5%90%84%E4%B8%AA%E5%B1%9E%E6%80%A7%E6%BA%90%E6%8B%89%E5%8F%96%E5%B1%9E%E6%80%A7.png)

图 Spring 环境从各个属性源拉取属性，并让 Spring 应用上下文的 bean 可以使用它们

Spring Boot 自动配置的 bean 都可以通过 Spring 环境拉取的属性进行配置。比如说，我们希望底层的 Servlet 容器使用另外一个端口监听请求，而不是 8080。

- 在 “src/main/resources/application.properties” 中设置：

  ```properties
  server.port=9090
  ```

- 在 “src/main/resources/application.yml” 中设置：

  ```yaml
  server:
    port: 9090
  ```

- 通过操作系统的环境变量，可以设置应用始终在一个特定的端口启动：(Spring 能够将 SERVER_PORT 解析为 server.port)

  ```bash
  $ export SERVER_PORT=9090
  ```

- 命令行参数：

  ```bash
  $ java -jar tacocloud-0.0.5-SNAPSHOT.jar --server.port=9090
  ```

常见的配置：

- 配置数据源，添加 mysql 的配置属性：

  ```yml
  spring:
    datasource:
      url: jdbc:mysql://localhost/tacocloud
      username: tacodb
      password: tacopassword
      driver-class-name: com.mysql.jdbc.Driver
  ```

  在前面我们建议要有一种方式声明应用启动的时候要执行的数据库初始化脚本。在这里也可以用配置文件声明：

  ```yaml
  spring:
    datasource:
      schema:
        - order-schema.sql
        - ingredient-schema.sql
        - taco-shema.sql
        - user-schema.sql
      data:
        - ingredients.sql
  ```

- 配置嵌入式服务器（Tomcat）

  将 server.port 设置为 0，服务器不会真的在端口 0 上启动，它会任选一个可用的端口，这样能够保证并发运行的测试不会和硬编码的端口号冲突。

  对底层容器常见的一个设置是让它处理 HTTPS 请求，首先我们要用 JDK 的 keytool 命令行工具生成 keystore：

  ```bash
  $ keytool -keystore mykeys.jks -genkey -alias tomcat keyalg RSA
  ```

  接着在配置文件中声明：

  ```yaml
  server:
    port: 8443
    ssl:
      key-store: file:///path/to/mykeys.jks
      key-store-password: letmein
      key-password: letmein
  ```

  server.ssl.key-store 属性应该设置为我们所创建的 keystore 文件的路径，这里使用了文件路径 file://URL，如果是在类路径下则应该使用 "classpath:" URL。

- 使用特定属性值

  spring 的属性值还可以借助其他属性值派生，如：

  ```yaml
  greeting:
    welcome: ${spring.application.name}
  ```

  甚至还可以嵌入到其他文本中：

  ```yaml
  greeting:
    welcome: You are using ${spring.application.name}.
  ```

#### 创建自己的配置属性

配置属性只不过是 bean 的属性，它们可以从 Spring 环境中接受配置。

Spring Boot 提供了 @ConfigurationProperties 注解，将它放到 bean 上，它就会为该 bean 的属性注入 Spring 环境中有的值。

当我们在配置文件中声明了某个属性，如：

```properties
taco.orders.page=15
```

在 Java 中我们就可以借助 @ConfigurationProperties 获取到该属性值

```java
@Controller
@RequestMapping("/orders")
@ConfigurationProperties(prefix="taco.orders") // 这里就是指定注入属性的地方
public class OrderController {
    private int page = 20; //
    
    public OrderController(int page) {
        this.page = page; // 属性文件中的值被注入
    }
    
    ...
}
```

我们可以在命令行中快速修改这个值，这样就不用重新构建和部署应用了：

```bash
$ export TACO_ORDERS_PAGE=10
```

**定义 Property Holder**

我们可以将以下这样的一系列属性，抽取到一个类中：

```yaml
taco:
  orders:
    page: 5
    size: 20
    sort: createdAt
```

```java
@Component
@ConfigurationProperties(prefix="taco.orders")
@Data
@Validated
public class OrderProps {
    
    @Min(value=5, message="must be between 5 and 25")
    @Max(value=25, message="must be between 5 and 25")
    private int page = 10;
    private int size = 10;
    private String sort = "name";
}
```

@Component 将类创建为 bean，以便我们在其他类中使用到它时可以将它注入。

@Validated, @Min, @Max 对字段进行了限制。

```java
@Controller
@RequestMapping("/orders")
public class OrderController {
    private OrderProps props;
    
    public OrderController(OrderRepository orderRepo, OrderProps props) {
        this.orderRepo = orderRepo;
        this.props = props;
    }
    
    ...

    int page = props.getPage();
    int size = props.getSize();
    ...
}
```



#### 使用 profile

当在开发环境和生产环境中需要配置不同的属性值时，我们可以选择使用环境变量设置属性，但这样做的太过麻烦，也无法在出现错误时回滚。我们可以使用 Spring profile。

在运行时，可以根据哪些 profile 处于激活状态，使用或忽略不同的 **bean**、**配置类**和**配置属性**。

**定义特定 profile 的属性**

我们可以按照 application-{profile}.properties 或 application-{profile}.yml 来对配置文件命名。例如，我们创建一个 application-prod.yml 以及一个 application-dev.yml 文件，在application.yml 文件上，激活其中一个配置文件：

```yaml
spring:
  profiles:
    active: prod
```

或者 application.properties 写法：

```properties
spring.profiles.active=prod
```

除此之外，yml 文件还支持另一种写法，可以在同一个文件中划分不同的 profiles，只需要用 "---" 分隔：

```yaml
logging:
  level:
    tacos: DEBUG
spring:
  profiles:
    active: prod

# 上面的配置是所有 profile 通用的
---
spring:
  profiles: prod

  datasource:
    url: jdbc:mysql://localhost/tacocloud
    username: tacouser
    password: tacopassword

# profiles 激活了 prod 配置，所以通用的 logging 的配置被覆盖掉
logging:
  level:
    tacos: WARN

---
spring:
  profiles: dev

  datasource:
  ...
```

**激活 profile**

在 yml 文件中可以这样激活 profile：

```yaml
spring:  
  profiles:
    active: 
      - prod
      - audit
      - ha
```

我们也可以使用环境变量来激活 profile：

```bash
$ export SPRING_PROFILES_ACTIVE=prod,audit,ha
```

注意 profiles 是复数形式的，多个profile只需要用逗号隔开。

如果是可执行 JAR 文件，还可以用命令行参数设置：

```bash
$ java -jar taco-cloud.jar --spring.profiles.active=prod,audit
```

**基于 profile 条件化创建 bean**

如果我们需要某个 bean 在生产环境下才被创建，可以使用 @Profile 注解：

```java
@Bean
@Profile("prod")
public CommandLineRunner dataLoader(IngredientRepository repo) {
    ...
}
```

也可以对文件进行取反 @Profile("!prod") ，这表示只有在 prod 配置文件没有被激活时，才创建 bean。