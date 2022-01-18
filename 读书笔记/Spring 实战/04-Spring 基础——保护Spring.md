### 04-Spring 实战——保护Spring

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

