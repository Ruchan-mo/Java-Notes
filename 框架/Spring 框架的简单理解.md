Spring 框架的简单理解

Spring 的容器是 Spring 框架中十分重要的概念。容器可以认为是一个管理着许多对象的集合，这些对象包括了 Java 类的实例。容器负责它们的创建、配置、销毁等。

最常用的容器有 BeanFactory 和 ApplicationContext。BeanFactory 提供了基本的依赖注入功能，ApplicationContext 是它的扩展，提供了更多的企业级功能。

可以把 BeanFactory 想象成一个维护实例对象的列表的容器，这些对象被称为 “bean”。这些 bean 可以提供 Spring 框架的各种功能。可以在 bean 中配置 AOP，实现横切关注点的功能，比如日志记录，性能监控等。可以在 bean 中配置事务管理、国际化等。

Spring 最重要的设计模式就是 “依赖注入”。简单来说，就是把以前自己在代码中 new 对象的过程：

```java
public class Car {
    private Engine engine;
    
    public Car() {
        this.engine = new Engine();
    }
}
```

转变为，使用 Spring 帮我们创建的对象：

```java
public class Car {
    private Engine engine;
    
    // 使用构造函数注入依赖
    public Car(Engine engine) {
        this.engine = engine;
        w
    }
}
```

第一种方式是硬编码依赖，依赖关系被直接硬编码在类的实现中。

在第二种方式中，Car 类所需要的 Engine 对象，由构造函数的参数传递进来，而这个具体传递进来的 Engine 对象，是由 Spring 容器负责创建的。

除了使用构造函数注入容器帮我们创建好的对象，我们还可以使用其他方式注入 bean：

1. Setter 方法注入：通过在类中定义 setter 方法，并在 setter 方法上添加注解或配置文件来注入对象。Spring 会在对象创建后调用这些 setter 方法来设置依赖对象。

   ```java
   public class Car {
       private Engine engine;
       public void setEngine(Engine engine) {
           this.engine = engine;
       }
   }
   ```

2. 字段注入：通过在字段上添加注解或配置文件来注入依赖对象。Spring 会在对象创建后直接设置字段的值来注入依赖对象。

   ```java
   public class Car {
       @Autowired
       private Engine engine;
   }
   ```

3. 接口注入：不再推荐使用，因为它增加了类之间的耦合。

4. 构造器注入：通过类的构造方法注入依赖。

   ```java
   public class Car {
       private Engine engine;
       @Autowired
       public Car(Engine engine) {
           this.engine = engine;
       }
   }
   ```

5. 方法注入：在方法上使用 `@Autowired` 注解，为方法提供依赖。

   ```java
   public class Car {
       private Engine engine;
       @Autowired
       public void equip(Engine engine) {
           this.engine = engine;
           // do something
       }
   }
   ```

6. 使用 `@Resource` 注解：按照名称或者类型注入依赖。

   ```java
   public class Car {
       @Resource(name = "engine")
       private Engine engine;
   }
   ```

7. 使用 `@Value` 注解：注入基本类型、String 类型的值。

   ```java
   public class Car {
       @Value("${some.property}") // 来自配置文件的值
       private String price;
   }
   ```

那么这些 bean 对象都是从哪里来的呢？Spring 是如何创建这些 bean 对象的？

依赖对象的创建取决于配置信息和注解的使用方式，常见的创建方式有：

1. 通过 XML 配置文件

   ```java
   <bean id="engine" class="com.myboot.Engine"/>
   ```

2. 在类上使用 `@Component`、`@Service`、`@Controller`、`@Repository` 注解，被注解的类将会被 Spring 创建为单例对象。

   ```java
   @Component
   public class Engine {
       
   }
   ```

   在 Spring Boot 应用中时，被注解的类必须在启动类的根目录或者子目录路径下，否则不会生效。

   或者在启动类中使用 `@ComponentScan` 注解标注扫描的路径。

   ```java
   @ComponentScan(value = {"com.myboot", "com.micro.config"})
   public class MyApplication {
       public static void main(String[] args) {
           SpringApplication.run(MyApplication.class, args);
       }
   }
   ```

3. 使用 `@Bean` 注解

   `@Configuration` 标识该类是一个 Spring Boot 配置类，Spring 将会扫描类中是否存在 `@Bean` 注解的方法，如果有，则创建该对象并放入容器中。

   ```java
   @Configuration
   public class CarConfig {
       @Bean
       public Engine engine() {
           return new Engine();
       }
   }
   ```

4. 使用 `@Import` 注解，会创建对象并注入容器中

   ```java
   @Import(Engine.class)
   public class MyApplication {
       public static void main(String[] args) {
           SpringApplication.run(MyApplication.class, args);
       }
   }
   ```



