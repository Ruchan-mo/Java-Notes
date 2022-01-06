**@Bean 和 @Component 的区别**

1. 两者的联系

   都是使用注解来定义 bean 的方式

2. 两者的区别

   **@Component** （包括 @Service、@Repository、@Controller）用于自动检测和使用类路径扫描自动配置 bean。注释类和 bean 之间存在隐式的一对一映射（即每个类一个 bean

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

   

   **@Bean** 用于显式声明单个 bean，而不是让 Spring 像上面那样自动执行它。它将 bean 的声明与类定义分离，并允许你精确地创建和配置 bean（也就是不用在类里面就写好 bean 的声明）

   ```java
   @Configuration
   public class WebSocketConfig{
       @Bean
       public Student student() {
           return new Student();
       }
   }
   ```

   

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

