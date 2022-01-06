# Spring Boot 实战

#### 一、项目构建

一个完整可运行的 Spring Boot 项目包含几个部分：

1. 启动类

   ```java
   @SpringBootApplication // 开启组件扫描和自动配置
   public class ReadingListApplication {
       public static void main(String[] args) {
           SpringApplication.run(ReadingListApplication.class, args); // 负责启动引导应用程序
       }
   }
   ```

   @SpringBootApplication 将三个有用的注解组合在了一起。

   - Spring 的 @Configuration：标明该类使用 Spring 基于 Java 的配置。（相对 XML）
   - Spring 的 @ComponentScan：启用组件扫描，我们写的 Web 控制器类和其他组件才能被自动发现并注册为 Spring 应用上下文里的 Bean。
   - Spring Boot 的 @EnableAutoConfiguration：开启自动配置。

   可以在启动类中编写简单的配置，不过一般会把配置写在新的类中，并加上 @Configuration 注解。

2. 测试类

   ```java
   @RunWith(SpringJUnit4ClassRunner.class)
   @SpringApplicationConfiguration(
   	classes = ReadingListApplication.class)  // 通过 Spring Boot 加载上下文
   @WebAppConfiguration
   public class ReadingListApplicationTests {
       @Test
       public void contextLoads() {	// 测试加载的上下文
       }
   }
   ```

3. 配置应用程序属性

   application.properties 在最开始是个空文件，这个文件时可选的，删了它不会影响程序正常运行。这个文件只要存在就会被 Spring Boot 加载。

4. 依赖文件（ maven 构建）

   如果使用 maven 构建项目，会生成一个 pom.xml 文件，  可以为我们提供项目运行所需要的依赖。其中的构建插件还可以帮助我们将项目打包成可运行的 jar 文件。

   在 Maven 里，可以用 <exclusions> 元素来排除不需要的依赖。

   ```xml
   <dependency>
   	<groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
       <exclusions>
       	<exclusion>
           	<groupId>com.fasterxml.jackson.core</groupId>
           </exclusion>
       </exclusions>
   </dependency>
   ```

   Maven 总是会使用最新的依赖。如果你在项目的 pom 文件中增加了一个更新版本的依赖，那它就会覆盖旧版本的依赖。

#### 二、自定义配置

Spring Boot 提供了非常强大的自动配置，但在某些情况下，自动配置不能满足我们的需求。比如，向项目中添加 Spring Security时，自动配置提供的单一账户，以及每次运行生成的随机密码，并不符合我们的实际需求。我们可以使用 Java 形式的配置，来覆盖自动配置。在这个例子中，我们需要一个扩展了 WebSecurityConfigurerAdapter 的配置类。

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private ReaderRepository readerRepository;	// reader DAO 接口
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.
            authorizeRequests()
            	.antMatchers("/readingList").access("hasRole('READER')")	// readingList 路径下，要求登录者有 READER 角色
            	.antMatchers("/**").permitAll()	// 其他所有请求路径向所有用户开放了访问权限
            
            .and()
            
            .formLogin()
            	.loginPage("/login")	// 设置登录表单路径
            	.failureUrl("/login?error=true");	// 设置登录失败页路径
    }
    
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .userDetailsService(new UserDetailsServices() {
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                return readerRepository.findOne(username);	// 定义自定义 UserDetailsService
            }
        });
    }
}
```

注：也可以自定义一个 CustomUserService 类实现 UserDetailsService，重写 `loadUserByUsername ()` 方法，来自定义验证的规则。

Spring Security 为身份认证提供了众多选项，后端可以是 JDBC（Java Database Connectivity）、LDAP和内存用户存储。

`loadUserByUsername()`方法返回的是 UserDetails 对象，所以需要我们自定义的User类（在这里是Reader）实现UserDetails 接口，这样我们的类就可以代表Spring Security 里的用户了。

```java
@Entity
public class Reader implements UserDetails {
    ...
    @Override
    public boolean isAccountNotExpired() {	// 账户不过期
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {	// 账户不会锁定
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {	// 认证不会过期
        return true;
    }
    @Override
    public boolean isEnabled() { 	// 账户不会被禁用
        return true;
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return Arrays.asList(new SimpleGrantedAuthority("READER")); 	// 授予 READER 权限
    }
}
```

##### 条件化配置

Spring Boot 的自动配置自带了很多配置类，它们会在需要的时候开启或者关闭，这是因为它们使用了 Spring 4.0 的条件话配置。

```java
@Bean
@ConditionalOnMissingBean(JdbcOperations.class) 	// 条件话配置的关键，只有缺失相应的 bean 才生效
public JdbcTemplate jdbcTemplate() {
    return new JdbcTemplate(this.dataSource);
}
```

也就是说，当我们手动配置了数据库，已经存在了一个 JdbcTemplate Bean，那么就会忽略自动配置的 JdbcTemplate Bean。

关于 Spring Security

```java
@Configuration
@EnableConfigurationProperties
@ConditionalOnClass({ EnableWebSecurity.class }) 	// 必须有 @EnableWebSecurity 注解才生效
@ConditionalOnMissingBean(WebSecurityConfiguration.class) 	// 当下没有 WebSecurityConfiguration 类型的 Bean 才生效
@ConditionalOnWebApplication 	// 必须是 Web 应用才生效
public class SpringBootWebSecurityConfiguration {
    ...
}
```

注：通过@EnableWebSecurity 注解，我们实际上间接创建了一个 WebSecurityConfiguration Bean。

#### 三、属性文件配置

我们用自定义的类可以取代 Spring Boot 的自动配置，而通过配置文件，我们可以微调一些细节，比如改改端口号和日志级别。

我们可以通过**环境变量**、**Java系统属性**、**JNDI**、**命令行参数**或者**属性文件**进行配置。

通过命令行参数：

```bash
$ java -jar readingList-0.0.1-SNAPSHOT.jar --spring.main.show-banner=false
```



另一种方式是创建一个名为 application.properties 的文件，包含以下内容：

```properties
spring.main.show-banner=false
```

也可以创建名为 application.yml 的文件：

```yaml
spring:
	main:
		show-banner: false
```

还可以将属性设置为环境变量：

```bash
$ export spring_main_show_banner=false
```

application.properties 和 application.yml 文件能放在四个位置：

(1) 外置，在相对于应用程序运行目录的 /config 子目录里

(2) 外置，在应用程序运行的目录里

(3) 内置，在 config 包内

(4) 内置，在 Classpath 根目录

这个列表按照优先级排序，也就是说，/config 子目录里的 application.properties 会覆盖应用程序 Classpath 里的 application.properties 中的相同属性。相同优先级位置中的 yml 文件会覆盖 properties 文件。

##### 在类里收集属性

我们可以将一系列自定义的属性创建成一个单独的 Bean，为它加上 @ConfigurationProperties 注解，让这个 Bean 收集所有配置属性。

```java
@Component
@ConfigurationProperties("amazon")
public class AmazonProperties{
    private String associateId;
    
    public void setAssociateId(String associateId) {
        this.associateId = associateId;
    }
    
    public String getAssociateId() {
        return associateId;
    }
}
```

此时，我们在 application.properties 配置文件中设置该属性：

```properties
amazon.associateId=1874yin
```

便可以在 Java 代码中获取到该属性。

##### 针对不同运行环境的配置文件

假若我们有开发环境和生产环境，那么我们可以创建 application-production.properties 和 application-development.properties 文件，在两个文件中编写针对不同环境的属性配置。那些并不特定于哪个环境或者保持默认值的属性，可以继续放在 application.properties 文件中。

```properties
spring.profiles.active=development
```

通过上面的方法即可激活特定环境的配置文件。

也可以在 Java 类中针对不同环境选择开启或关闭配置：

```java
@Profile("production") 	 // 只有在 production 环境中该配置类才会生效
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

