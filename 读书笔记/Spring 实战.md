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

![SpringWeb各个组件之间的关系](C:\Users\lonel\Desktop\Java学习笔记\Pics\SpringWeb各个组件之间的关系.jpg)

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





