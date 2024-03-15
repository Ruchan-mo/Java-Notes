## @Bean 和 @Component 的区别

### 联系

跟 xml 配置方式一样，都是定义 bean 的方式

### 区别

#### @Component 

（包括 @Service、@Repository、@Controller、@Configuration）用于自动检测和使用类路径扫描自动配置 bean。注释类和 bean 之间存在隐式的一对一映射（即每个类一个 bean）。实际上就是把整个类作为 bean 注入到 Spring / Spring MVC 容器中。

```java
@Component
public class Student{
    
    private String name = "lkm";
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
}
```

我们在写完一个 Controller 后，会在类上加上注解 @Controller  / @RestController（@Controller + @ResponseBody），当 SpringMVC 容器扫描到该类，就会将该类注入到容器中，方便使用。

#### @Bean 

用于显式声明单个 bean，而不是让 Spring 像上面那样自动执行它。它将 bean 的声明与类定义分离，并允许你精确地创建和配置 bean（也就是不用在类里面就写好 bean 的声明）

```java
@Configuration
public class MyAppConfig {

    @Bean
    public Student student() {
        return new Student();
    }
    
    @Bean
    public Dog dog1() {
        return new Dog("小黄", 4);
    }
    
    /**
     * MySQL 配置
     */
    @Bean
    public MysqlProperties mysqlProperties() {
        MysqlProperties mp = new MysqlPropertiesBuilder()
            .url("jdbc:mysql://localhost:3306/emp?characterEncoding=UTF-8")
            .username("root")
            .password("admin")
            .driver("com.mysql.jdbc.Driver")
            .build();
    }
}
```

我们可以使用 @Bean 在一个配置类 (加上 @Configuration 注解的类）中配置很多个 Bean，使用了 @Configuration 注解的类也会被创建为 bean 注入到容器中。使用 @Bean 注解配置的 bean，名称为方法名，如 student、dog1。

**都可以使用 @Autowired 和 @Resource 注解注入**

```java
@Autowired
Student student;

// 或者 

@Resource
Student student;
```



**为什么有了 @Component，还需要 @Bean**

如果想将第三方的类变成组件，而你没有源代码，也就没办法在类上使用 @Component 的方式进行自动配置，这个时候 @Bean 就派上用场了。当然，你也可以使用 xml 的方式来定义。

另外，@Bean 注解的方法返回值是对象，可以在方法中为对象设置属性

#### XML 方式配置的 bean

```xml
<bean id = "mysqlProperties" class="com.demo.MysqlProperties">
    <property name="url" value="jdbc:mysql://localhost:3306/emp?characterEncoding=UTF-8"></property>
    <property name="username" value="root"></property>
    <property name="password" value="admin"></property>
    <property name="driver" value="com.mysql.jdbc.Driver"></property>
</bean>
```



#### Q&A

我们在 Service 层能不能注入 Controller 的 bean？
