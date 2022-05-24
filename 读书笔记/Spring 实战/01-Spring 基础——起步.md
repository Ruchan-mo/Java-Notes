### 01-Spring 基础 —— 起步

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