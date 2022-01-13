# Spring 实战

#### 一、起步

##### 什么是 Spring

任何实际的应用程序都是由很多组件完成的，每个组件负责整个应用功能的一部分，这些组件需要与其他的应用元素进行协调以完成自己的任务。当应用程序运行时，需要以某种方式创建并引入这些组件。

Spring 的核心是 **提供了一个容器，通常称为 Spring 应用上下文（Spring  application context）**，它们会创建和管理应用组件。

这些组件也可以称为 bean，会在 Spring 应用上下文中装配在一起。

bean 的装配是通过一种基于*依赖注入*（dependency injection）的模式实现的。组件不会自己创建它所依赖的组件并管理它们的生命周期，而是依赖于容器来创建和维护所有的组件，并将其注入到需要它们的 bean 中。这种过程也叫控制反转。通常这是通过构造器参数（Constructor）和属性访问方法（Setter）来实现的。

在以前使用 XML 文件配置 bean 的装配，如下的 XML 描述了两个 bean，InventoryService bean 和 ProductService bean，并且通过构造器参数将 InventoryService 装配到了 ProductService中：

```xml
<bean id="inventoryService" 
      class="com.example.InventoryService" />
<bean id="productService"
      class="com.example.ProductService">
	<constructor-arg ref="inventoryService"/>
</bean>
```

在最近的 Spring 版本中，基于 Java 的配置更为常见，如下的 Java 代码等同于上面的 XML 配置：

```java
@Configuration
public class ServiceConfiguration {
    @Bean
    public InventoryService inventoryService() {
        return new InventoryService();
    }
    
    @Bean
    public ProductService productService() {
        return new ProductService(inventoryService());
    }
}
```

@Configuration 注解会告知 Spring 这是一个配置类，会为 Spring 应用上下文提供 bean。

@Bean 注解表明这些方法所返回的对象会以 bean 的形式添加到 Spring 的应用上下文。默认情况下，bean ID 与方法名相同。

借助组件扫描（component scanning）技术，Spring 能够自动发现应用类路径下的组件，并将它们创建成 Spring 应用上下文中的 bean。借助自动装配（autowiring）技术，Spring 能够自动为组件注入它们所依赖的其他 bean。

##### 初始化 Spring 应用

我们可以在以下途径使用 Spring Initializr 初始化 Spring 应用：

- 在 https://start.spring.io/ 的 Web 应用
- 在命令行中使用 curl 命令
- 在命令行中使用 Spring Boot 命令行接口
- 在 Spring Tool Suite 中创建
- 在 IntelliJ IDEA 中创建
- 在 NetBeans 中创建

使用以上任一种途径初始化 Spring，会得到一个最基础的 Spring 应用，它是一个典型的 Maven 或 Gradle 项目。

以 taco-cloud 项目为例。

**pom.xml 文件：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent> <!-- 父 POM -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.2</version>  <!-- Spring Boot 的版本 -->
        <relativePath/>
    </parent>

    <groupId>sia</groupId>
    <artifactId>taco-cloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>	<!-- 打包为 JAR -->

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>htmlunit-driver</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

<packaging> 指定了应用的打包方式为可执行的 JAR 文件，而不是 WAR 文件。这是基于云思维做出的选择，所有的 Java 云平台都能够运行可执行的 JAR 文件。

<parent> 表明了我我们要以  spring-boot-starter-parent 作为其父 POM，version 表明要使用的 Spring Boot 版本。

<dependencies> 下有三个依赖中 artifact ID 上都有 starter 这个单词。Spring Boot starter 依赖的特别之处在于它们本身并不包含库代码，而是传递性地拉取其他的库。这种 starter 有3个好处：

- pom 文件显著减小并易于管理，因为不必为每个所需的依赖库都声明依赖。
- 如果开发 web 应用，只需添加 web starter 就可以了。
- 不必担心版本兼容问题，给定 Spring Boot 的版本，传递引入的库的版本就是兼容的。

<build> 中的<plugins>包含了一个 Spring Boot 的插件，提供了一些功能：

- 提供一个 Maven goal，允许我们使用 Maven 来运行应用
- 确保依赖的所有库都会包含在 JAR 文件中，并且在运行时类路径下可用
- 在 JAR 中生成一个 manifest 文件，将引导类声明为可执行 JAR 的主类。

**引导类**
因为我们构建的是一个 JAR 可执行程序，所以我们需要一个主类，它会在 JAR 文件执行的时候被运行。我们还需要一个最基本的 Spring 配置。这就是 TacoCloudApplication 类的作用。

```java
package tacos;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TacoCloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(TacoCloudApplication.class, args);
    }

}
```

@SpringBootApplication 注解表明这是一个 Spring Boot 应用。这是一个组合注解，组合了其他三个注解：

- @SpringBootConfiguration：将该类声明为配置类。实际上是 @Configuration 注解的特殊形式。
- @EnableAutoConfiguration：启用 Spring Boot 的自动配置。
- @ComponentScan：启用组件扫描。这样我们能够通过像 @Component、@Controller、@Service 这样的注解声明其他类，Spring会自动发现它们并将它们注册为 Spring 应用上下文中的组件。

main() 方法是 JAR 文件执行时要运行的方法。main() 方法调用 SpringApplication 中静态的 run() 方法，后者会真正执行应用的引导过程，也就是创建 Spring 的应用上下文。run() 方法中的两个参数，一个是配置类（默认引导类），一个是命令行参数。

一般来说，不在引导类中配置组件，而是为需要配置的功能创建一个单独的配置类。



#### 二、开发 Web 应用

##### 展现信息

在 Spring Web 应用中，获取和处理数据是控制器的任务，而将数据渲染到 HTML 中并在浏览器中展现则是试图的任务。-

Spring Web 中各个组件之间的关系：

![SpringWeb各个组件之间的关系](https://github.com/1874yin/Java-Notes/blob/main/Pics/SpringWeb%E5%90%84%E4%B8%AA%E7%BB%84%E4%BB%B6%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB.jpg?raw=true)

控制器的主要职责是处理 HTTP 请求，要么将请求传递给试图以便于渲染 HTML （浏览器展现），要么直接将数据写入响应体（RESTful）。

**Spring MVC 的请求映射注解**

| 注释            | 描述                  |
| --------------- | --------------------- |
| @RequestMapping | 通用的请求处理        |
| @GetMapping     | 处理 HTTP GET 请求    |
| @PostMapping    | 处理 HTTP POST 请求   |
| @PutMapping     | 处理 HTTP PUT 请求    |
| @DeleteMapping  | 处理 HTTP DELETE 请求 |
| @PatchMapping   | 处理 HTTP PATCH 请求  |

像 Thymeleaf 这样的视图在设计时是与特定的 web 框架解耦。也就是说，它们无法与控制器放到 Model 中的数据协同工作。但是，它们可以利用 Servlet 中的 request 属性。在Spring 将请求转移到视图之前，会把模型数据复制到 request 属性中，Thymeleaf 就能访问到它们了（类似JSP）。

Thymeleaf 就是增加一些额外元素的 HTML，这些属性可以指导模板如何渲染 request 数据。假如有一个请求属性的  key 为 "message"，我们想要使用 Thymeleaf 将其渲染到 HTML 的要给 <p> 标签中，可以这么写：

```html
<p th:text="${message}">placeholder message</p>
```

当模板渲染成 HTML 时，<p> 元素会被替换为 Servlet Request 中 key 为 "message" 的属性值。

如果想渲染 "wrap" 配料的列表，我们可以使用 "th:each" ：

```html
<h3>Designate your wrap:</h3>
<div th:each="ingredient : ${wrap}">
    <input name="ingredients" type="checkbox" th:value="${ingredient.id}" />
    <span th:text="${ingredient.name}">INGREDIENT</span><br/>
</div>
```

渲染完成后，其中一个 <div> 迭代的渲染结果可能会如下所示：

```html
<div>
    <input name="ingredients" type="checkbox" value="FLTO" />
    <span>Flour Tortilla</span><br/>
</div>
```

**检验表单输入**

在视图中接收数据后，我们想要检验数据的合理性。可以在 controller 的方法中添加大量 if/then 代码块，逐个检查，确保每个输入域都满足对应的规则。但是，这样会非常烦琐，并且难以阅读和调试。

Sping 支持 Java 的 Bean 检验 API（Bean Validation API）。我们可以更容易地声明检验规则，而不用在应用程序代码中显式编写声明逻辑。要在 Spring MVC 中应用校验，我们需要：

- 在要被校验的类上声明校验规则 （Bean）
- 在控制器方法中声明要进行校验（Controller 中的方法）
- 修改表单视图以展现校验错误

validation 需要的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**定义规则**

```java
import lombok.Data;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import java.util.List;

@Data
public class Taco {
    @NotNull
    @Size(min = 5, message = "Name must be at least 5 characters long")
    private String name;
    @Size(min = 1, message = "You must choose at least 1 ingredient")
    private List<String> ingredients;

}
```

@NotNull 要求 name 属性不为 null，@Size 要求它的值在长度上至少要有5个字符。

```java
import lombok.Data;
import org.hibernate.validator.constraints.CreditCardNumber;
import javax.validation.constraints.Digits;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.Pattern;

@Data
public class Order {

    @NotBlank(message = "Name is required")
    private String name;
    
    @NotBlank(message = "Street is requried")
    private String street;
    
    @NotBlank(message = "City is required")
    private String city;
    
    @NotBlank(message = "State is required")
    private String state;
    
    @NotBlank(message = "Zip code is required")
    private String zip;
    
    @CreditCardNumber(message = "Not a valid credit card number")
    private String ccNumber;
    
    @Pattern(regexp = "^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$", message = "Must be formatted MM/YY")
    private String ccExpiration;
    
    @Digits(integer=3, fraction = 0, message = "Invalid CVV")
    private String ccCVV;
}
```

对于地址类的属性，我们要确保用户没有提交空白字段。

@NotBlank 注解要求输入的内容不为空。

@CreditCardNumber 注解要求该属性的值必须是合法的信用卡号。（美国信用卡）

@Pattern 注解提供了一个正则表达式，确保属性值符合预期格式。

@Digits 注解确保它的值包含3个数字。

所有的注解都包含了一个 message 属性，当输入的信息不满足声明的校验规则时会将该属性展现给用户。

**开启校验**

在 Controller 的方法中使用 @Valid 开启校验

```java
@PostMapping
public String processDesign(@Valid @ModelAttribute("design") Taco design, Errors errors) {
    if (errors.hasErrors()) {
        return "design";
    }
    // Save the taco design...
    log.info("Processing design: " + design);
    return "redirect:/orders/current";
}
```

@Valid 注解会告诉 Spring MVC 要对提交的 Taco 对象进行校验，校验时机是在它绑定完表单数据之后、调用 processDesign() 之前。如果存在错误，那么这些错误的细节会被捕获到一个 Errors 对象中。

一般的控制器类有几个特点：

- 它们都使用了 @Controller 注解，表明他们是控制器类，并且应该被 Spring 的组件扫描功能发现并初始化为 Spring 应用上下文的 bean；
- 在类级别使用了 @RequestMapping注解，据此定义该控制器所处理的基本请求模式；
- 有一个或多个带 @GetMapping 或 @PostMapping 注解的方法，指明了该由哪个方法来处理某种类型的请求。

如果一个控制器非常简单，不需要填充模型或处理输入（比如 HomeController），那么可以使用另一种方式定义控制器。

```java
@Controller
@RequestMapping("/")
public String homeController() {
    return "home";
}
```

上面的控制器可以替代为：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

#### 三、持久化

在处理关系型数据的时候，Spring 支持 JDBC 和 JPA 两种形式。

##### 从 JDBC 到 JPA

Spring 对 JDBC 的支持主要是 JdbcTemplate。

**不使用 JdbcTemplate ：**

```java
@Override
public Ingredient findOne(String id) {
    Connection connection = null;
    PreparedStatement statement = null;
    ResultSet resultSet = null;
    try {
        connection = dataSource.getConnection();
        statement = connection.prepareStatement(
        "select id, name, type from Ingredient where id=?");
        statement.setString(1, id);
        resultSet = statement.executeQuery();
        Ingredient ingredient = null;
       	if (resultSet.next()) {
            ingredient = new Ingredient(
            	resultSet.getString("id");
                resultSet.getString("name");
                Ingredient.Type.valueOf(resultSet.getString("type")));
        }
        return ingredient;
    } catch (SQLException e) {
        // Do something...
    } finally {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {}
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {}
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {}
        }
    }
    return null;
}
```

大量重复的创建连接、创建语句以及关闭连接、语句和结果集使得代码看起来非常繁琐复杂。

**使用 JdbcTemplate**

查询一个对象：

```java
private JdbcTemplate jdbc;

@Override
public Ingredient findOne(String id){ 
	return jdbc.queryForObject(
    	"select id, name, type from Ingredient where id=?",
        this::mapRowToIngredient, id);
    );
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
    return new Ingredient(
    	rs.getString("id"),
        rs.getString("name"),
        Ingredinet.Type.valueOf(rs.getString("type")));
}
```

使用 JdbcTemplate 可以让我们仅仅关注执行查询 `queryForObject()` 和映射对象 `mapRowToIngredient()` 上。

查询多个对象的集合：

```java
@Override
public Iterable<Ingredient> findAll() {
    return jdbc.query("select id, name, type from Ingredient",
                     this::mapRowToIngredient);
}
```

插入一行数据：

```java
@Override
public Ingredient save(Ingredient ingredient) {
    jdbc.update(
    	"insert into Ingredient (id, name, type) values (?, ?, ?)",
        ingredient.getId(),
        ingredient.getName(),
        ingredient.getType.toString());
    return ingredient;
}
```

复杂插入：

- 使用 `update()` 方法

  ```java
  private long saveTacoInfo(Taco taco) {
      taco.setCreatedAt(new Date());
      PreparedStatementCreator psc = 
          new PreparedStatementCreatorFactory(	// 工厂类，传入 sql 语句和每个参数的类型
      		"insert into Taco (name, createdAt) values (?, ?)",
          	Types.VARCHAR, Types.TIMESTAMP
      	).newPreparedStatementCreator(
      		Arrays.asList(
              	taco.getName(),
                  new Timestamp(taco.getCreatedAt().getTime())));
      KeyHolder keyHolder = new GeneratedKeyHolder();
      jdbc.update(psc, keyHolder);	// 传入 KeyHolder，查询后便可以获取到taco的ID
   
      return keyHolder.getKey().longValue();
  }
  ```

  

- 使用 SimpleJdbcInsert 包装器类

  先创建 SimpleJdbcInsert，插入指定的表中。使用 ObjectMapper 将 Order转换为 Map

  ```java
  private SimpleJdbcInsert orderInserter;
  private SimpleJdbcInsert orderTacoInserter;
  private ObjectMapper objectMapper;
  
  @Autowired
  public JdbcOrderRepository(JdbcTemplate jdbc) {
      this.orderInserter = new SimpleJdbcInsert(jdbc)
          .withTableName("Taco_Order")
          .usingGeneratedKeyColumns("id");
      this.orderTacoInserter = new SimpleJdbcInsert(jdbc)
          .withTableName("Taco_Order_Tacos");
      this.objectMapper = new ObjectMapper();
  }
  ```

  使用：

  ```java
  private long saveOrderDetails(Order order) {
      Map<String, Object> values = objectMapper.convertValue(order, Map.class);
      values.put("placedAt", order.getPlacedAt()); 	// Date 类型会被 ObjectMapper 转换为 long
      long orderId = 
          orderInserter
          	.executeAndReturnKey(values)	// 返回 Number 类型的 ID
          	.longValue();
      return orderId;
  }
  
  private void saveTacoToOrder(Taco taco, long orderId) {
      Map<String, Object> values = new HashMap<>();
      values.put("tacoOrder", orderId);
      values.put("taco", taco.getId());
      orderTacoInserter.execute(values);
  }
  ```

**使用 Spring Data JPA**

Spring Data 提供了一项特性，就是基于 repository 规范接口自动生成 repository 的功能。

准备：

为了使用 Spring Data JPA，我们需要先将 domain 对象标注为实体。

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@Entity
public class Ingredient {
    @Id
    private final String id;
    private final String name;
    private final Type type;
    ...
}
```

@Entity 注解可以将 Ingredient 声明为 JPA 实体，另外它的 id 属性还需要添加 @Id 注解。

@NoArgsConstructor JPA 需要实体有一个无参构造器，我们不想注解使用它，所以 access 设置为 PRIVATE，因为类中有 final 的属性，我们将 force 设置为 true，这样 Lombok 生成的构造器就会将它们设置为 null。

```java
@Data
@Entity
@Table(name="taco")
public class Taco {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    @ManyToMany(targetEntity = Ingredient.class)
    private List<Ingredient> ingredients;
    
    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```

@GeneratedValue 指定了自动生成 ID 值。

@ManyToMany 注解声明了 Taco 与其关联的 Ingredient 列表之间的关系：多对多。

@PrePersist 注解的方法会在 Taco 持久化之前执行，将 createdAt 设置为当前的日期和时间。

@Table 注解表明了实体应该持久化到数据库中的表名。hibernate 对表名大小写敏感，而数据库中表名是不区分大小写的，类似 “Taco Order” 的表名会被自动转为 “taco_order”，@Table 注解中应该使用小写表名，否则会导致报错。

声明：

将 repository 继承 CrudRepository 接口即可

```java
public interface IngredientRepository extends CrudRepository<Ingredient, String> {
    
}
```

CrudRepository 第一个参数是要持久化的实体类，第二个参数是实体 ID 属性的类型。它定义了很多用于 CRUD 操作的方法。

使用：

```java
@Autowired
private IngredientRepository ingredientRepo;

...
	ingredientRepo.findAll().forEach(i -> ingredients.add(i));
```

自定义 JPA repository

```java
List<Order> findByDeliveryZip(String deliveryZip); 	// 获取投递到指定 Zip 的订单
List<Order> readOrdersByDeliveryZipAndPlacedAtBetween(
	String deliveryZip, Date startDate, Date endDate); 	// 查找投递到指定 Zip 且在一定范围内的订单，read, get, find 都有一样的作用
```

较为复杂的查询可以使用 @Query 注解，明确指明方法调用时要执行的查询

```java
@Query("from Order o where o.deliveryCity='Seattle'")
List<Order> readOrdersDeliveredInSeattle();
```

Spring Boot  会在启动的时候执行根类路径下名为 schema.sql 和 data.sql 的文件，我们可以把定义表结构的 schema.sql 和 插入数据的 data.sql 文件放在 "src/main/resources" 文件夹下。

schema.sql

```sql
create table if not exists Taco_Ingredients (
  taco bigint not null,
  ingredient varchar(4) not null
);
```

data.sql

```sql
delete from Taco_Ingredients;
insert into Ingredient (id, name, type) values ('FLTO', 'Flour Tortilla', 'WRAP');
```



#### 四、保护 Spring

##### 启动 Spring Security

在项目中增加 Spring Security 的依赖即可启用 Spring Security。

Spring 会默认启用基本的安全配置，访问网站需要提供身份验证，默认用户名为 user ，随机的密码出现在应用的日志文件（控制台）中，类似：

```bash
Using default security password: 087cfc6a-027d-44bc-95d7-cbb3a798alea
```

基本的安全配置特性：

- 所有的 HTTP 请求都需要认证
- 不需要角色和权限
- 没有登录页面
- 认证过程是通过 HTTP basic 认证对话框实现的
- 系统只有一个用户，用户名为 user

要想确保应用的安全性，我们至少需要以下功能：

- 自定义登录页面
- 提供多用户，以及注册功能
- 不同请求路径执行不同的安全规则

为此，我们可以编写显式的配置覆盖掉自动配置提供的功能。从用户存储开始，这样我们就可以有多个用户了。

##### 配置 Spring Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        ...
    }
}
```

Spring Security 提供了多种配置用户存储的可选方案，我们只要覆盖 WebSecurityConfigurerAdapter 基础类中定义的  `configure() ` 方法来进行配置。

**基于内存**

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .inMemoryAuthentication()
        	.withUser("root")
                .password("{noop}admin")
                .authorities("ROLE_USER")
        	.and()
        	.withUser("user")
        		.password("{noop}123456")
        		.authorities("ROLE_USER");
}
```

**基于 JDBC**

```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .jdbcAuthentication()
        	.dataSource(dataSource)
        	.usersByUsernameQuery( 	// 自定义查询 sql 语句
    			"select username, password, enabled from Users where username = ?")
        	.authoritiesByUsernameQuery(
    			"select username, authority from UserAuthorities where username = ?")
        	.passwordEncoder(new StandardPasswordEncoder("53cr3t"));
}
```

在 `configure()`  中调用了 AuthenticationManagerBuilder 的 `jdbcAuthentication()`方法，并且设置了一个 DataSource，这个 DataSource 是自动装配得到的。

我们还可以用自定义的 sql 语句替换默认的的查询。

如果存储在数据库的密码使用明文存储，就很容易受到黑客的攻击窃取。我们可以对密码进行转码处理，保护密码。

为了解决这个问题，我们需要借助 `passwordEncoder()` 方法指定一个转码器（encoder），它可以接受 Spring Security 中 PasswordEncoder 接口的任意实现：

- BCryptPasswordEncoder： 使用 bcrypt 强哈希加密。
- NoOpPasswordEncoder： 不进行任何转码。
- Pbkdf2PasswordEncoder：使用 PBKDF2 加密。
- SCryptPasswordEncoder：使用 scrypt 哈希加密。
- StandardPasswordEncoder：使用 SHA-256 哈希加密。

你也可以自己实现 PasswordEncoder：

```java
public interface PasswordEncoder {
    String encode(CharSequence rawPassword);
    boolean matches(CharSequence rawPassword, String encodedPassword);
}
```

数据库中的密码是永远不会解码的。用户在登录时输入的密码会按照相同的算法进行转码，然后和数据库中已经转码过的密码进行对比。这个对比是在 `matches()`方法中进行的。

**自定义用户认证**

我们可以使用基于 JDBC 的认证，但更好的办法是使用 Spring Data repository 来存储用户。在此之前我们要先创建 Field 对象。

```java
@Entity
@Data
@NoArgsConstructor(access=AccessLevel.PRIVATE, force=true)
@RequiredArgsConstructor
public class User implements UserDetails {
    private static final long serialVersionUID = 1L;
    
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    
    private final String username;
    private final String password;
    
    ...
        
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
    
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

通过实现 UserDetails 接口，我们能够提供更多信息给框架，比如用户都被授予了哪些权限以及用户的账号是否可用。

`getAuthorities()`方法返回用户被授予权限的一个集合。在这里所有的用户都被授予了 ROLE_USER 权限。

各种 `is...Expired()`方法返回一个 boolean 值，表明用户的账号是否可用或过期。

**创建 UserDetailsService**
Spring Security 的 UserDetailsService 是一个相当简单直接的接口：

```java
public interface UserDetailsService {
    UserDetails loadUserbyUsername(String username) throws UsernameNotFoundException;
}
```

这个接口的实现会接收一个用户名，然后返回一个 UserDetails 对象，或者在根据用户名无法得到任何结果的情况下抛出 UsernameNotFoundException。正好 User 类实现了 UserDetails，可以在这里使用。

```java
@Service
public class UserRepositoryDetailsService implements UserDetailsService {
    private UserRepository userRepo;
    
    @Autowired
    public UserRepositoryUserDetailsService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username);
        if (user != null) {
            return user;
        }
        throw new UsernameNotFoundException("User '" + username + "' not found");
    }
}
```

通过注入进来的 UserRepository，我们使用用户名来查找User。

UserDetailsService 添加了 @Service，表明这个类要包含到 Spring 的组件扫描中，Spring 会自动发现它并初始化为一个 bean。

现在我们要将自定义的用户详情和 Spring Security 配置在一起，因此我们再次回到 `configure()`方法：

```java
@Autowired
private UserDetailsService userDetailsService;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .userDetailsService(userDetailsService);
}
```

像基于 JDBC 的认证一样，我们应该配置一个密码转码器，保证在数据库中的密码是转码过的。

```java
@Bean
public PasswordEncoder encoder() {
    return new Pbkdf2PasswordEncoder("bs23c3");
}

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
        .userDetailsService(userDetailsService)
        .passwordEncoder(encoder());
}
```

用 @Bean 将 `encoder()`方法注册，交给容器管理，以后用到的地方就可以自动装配。

我们可以将接收表单数据的类和 User 类分开

```java
@Data
public class RegistrationForm {
	
    private String username;
    private String password;
    ...
    
    public User toUser(PasswordEncoder passwordEncoder) {
        return new User(
        	username, passwordEncoder.encode(password),
            fullname, street, city, state, zip, phone);
    }
}
```

这样，我们便可以用 RegistrationForm 对象绑定请求的数据，并且在密码存进数据库之前把它转码：

```java
private UserRepository userRepo;
private PasswordEncoder passwordEncoder;

public RegistrationController(UserRepository userRepo, PasswordEncoder passwordEncoder) {
    this.userRepo = userRepo;
    this.passwordEncoder = passwordEncoder;
}

@PostMapping
public String processRegistration(RegistrationForm form) {
    userRepo.save(form.toUser(passwordEncoder));
    return "redirect:/login";
}
```

##### 保护 Web 请求

一般的网页请求必须经过认证，但是，主页、登录页和注册页应该对未认证的用户开放。我们需要用到 WebSecurityConfigurerAdapter 的 `configure()`方法。

我们可以使用 HttpSecurity 配置的功能包括：

- 在为某个请求提供服务之前，需要预先满足特定的条件
- 配置自定义的登录页
- 支持用户退出应用
- 预防跨站请求伪造

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        	.antMatchers("/design", "/orders")
        		.hasRole("USER")
        	.antMatchers("/", "/**").permitAll();
}
```

对 `authorizeRequests()` 的调用会返回一个对象（ExpressionInterceptUrlRegistry），基于它我们可以指定 URL 路径和这些路几个的安全需求。在上面的例子中，我们指定了两条规则：

- 具备 ROLE_USER 权限的用户才能访问 "/design" 和 "/orders"；
- 其他的请求允许所有用户访问

注意声明在前面的规则比后面的规则有更高的优先级。

在声明请求路径的安全需求时，`hasRole()` 和  `permitAll()`只是众多方法中的两个。

**用来定义如何保护路径的配置方法**

| 方法                       | 能够做什么                                                   |
| -------------------------- | ------------------------------------------------------------ |
| access(String)             | 如果给定的 SpEL 表达式计算结果为 true，就允许访问            |
| anonymous()                | 允许匿名用户访问                                             |
| authenticated()            | 允许认证过的用户访问                                         |
| denyAll()                  | 无条件拒绝所有访问                                           |
| fullAuthenticated()        | 如果用户是完整认证的（不是通过 Remember me功能认证的），就允许访问 |
| hasAnyAuthority(String...) | 如果用户具备给定权限中的一个，就允许访问                     |
| hasAnyRole(String...)      | 如果用户具备给定角色中的一个，就允许访问                     |
| hasAuthority(String)       | 如果用户具备给定权限，就允许访问                             |
| hasIpAddress(String)       | 如果请求来自给定 IP 地址，就允许访问                         |
| hasRole(String)            | 如果用户具备给定角色，就允许访问                             |
| not()                      | 对其他访问方法的结果求反                                     |
| permitAll()                | 无条件允许访问                                               |
| rememberMe()               | 如果用户是通过 Remember-me 功能认证的，就允许访问            |

`access()`方法可以让定义更加灵活。比如假设我们只允许具备 ROLE_USER 权限的用户在星期二访问，那我们可以重写表达式：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        	.antMatchers("/design", "/orders")
        		.access("hasRole('ROLE_USER') && T(java.util.Calendar).getInstance().get(T(java.util.Calendar).DAY_OF_WEEK)"
                       + " == T(java.util.Calendar).TUESDAY")
        	.antMatchers("/", "/**").access("permitAll");
}
```

**自定义登录页和退出**

我们首先需要告诉 Spring Security 自定义登录页的路径。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .authorizeRequests()
        	.antMatchers("/design", "/orders")
        		.hasRole("USER")
        	.antMatchers("/", "/**").permitAll()
        
        .and()
        	.formLogin()
        		.loginPage("/login")
        		.defaultSuccessUrl("/design")
        .and()
        	.logout()
        		.logoutSuccessUrl("/")
        ;
}
```

我们调用 `formLogin()`开始配置自定义的登录表单，`loginPage()`的调用声明了我们提供的自定义登录页面路径。只需要在视图控制器里将该路径映射到对应视图即可。用户成功登录之后将会被定向到 “/design” 页面。

调用`logout()`方法可以启用退出功能，这样做会搭建一个安全过滤器，该过滤器会拦截对 “/logout” 的请求。只要用户访问 "/logout" 路径，他们的 session 就会被清理。在这里我们设置了用户退出后返回主页。

**防止跨站请求伪造**

跨站请求伪造（Cross-Site Request Forgery, CSRF）是一种常见的安全攻击。简单来说就是让用户在一个伪造的页面填写敏感信息，然后将数据 POST 到例如用户银行 Web 站点的 URL 上。

为了防止这种攻击，应用可以在展现表单的时候生成一个 CSRF token，放到隐藏域，将其临时存储起来。在提交表单的时候，token 将和其他的表单数据一起发送至服务器端。请求会被服务器拦截，并对比 token，如果一致，则请求将会允许处理。（如果不一致，会导致 403 错误）

Spring Security 默认开启 CSRF 保护，我们只需要在每个表单中增加一个名为“_csrf” 的字段，持有 CSRF token。在 Thymeleaf 模板中，我们可以简化这个步骤：

 ```html
<input type="hidden" name="_csrf" th:value="${_csrf.token}" />
 ```

甚至我们只需要确保 <form> 的某个属性带有 Themeleaf 属性前缀（th）即可。比如：

```html
<form method="POST" th:action="@{/login}" id="loginForm">
```

除此之外，还可以禁用 Spring Security 对 CSRF 的支持，可以调用 `disable()` 来实现：

```java
.and()
    .csrf()
    	.disable()
```

不要禁用 CSRF 防护，特别是在生产环境中。

**获取登录用户**

常见的方法：

- 注入 Principal 对象到控制器方法中
- 注入 Authentication 对象到控制器方法中
- 使用 SecurityContextHolder 来获取安全上下文
- 使用 @AuthenticationPrincipal 注解来标注方法

使用 Principal：

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, Principal principal) {
    String username = principal.getName();
    User user = userRepo.findByUsername(username);
    ...
}
```

使用 Authentication

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, Authentication authentication) {
    User user = (User) authentication.getPrincipal();
    ...
}
```

直接接受一个 User 对象，但是需要加上 @AuthenticationPrincipal 注解，才会变成认证的 principal：

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors, SessionStatus sessionStatus, @AuthenticationPrincipal User user) {
    ...
}
```

从安全上下文中获取一个 Authentication 对象，它可以在程序的任何地方使用，不仅仅在控制器中：

```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
User user = (User) authentication.getPrincipal();
```

