### SpringBoot + JWT + Spring Security 实现认证和授权

#### 技术栈

- Spring Security

- JWT

  | JWT 是 JSON WEB TOKEN 的缩写，它是基于 RFC 7519 标准定义的一种可以安全传输的 JSON 对象，由于使用了数字签名，所以是可信任和安全的。

#### JWT 的组成

- JWT 的格式： `header.payload.signature`

  ```
  eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJhZG1pbiIsImNyZWF0ZWQiOjE1NTY3NzkxMjUzMDksImV4cCI6MTU1NzM4MzkyNX0.d-iki0193X0bBOETf2UN3r3PotNIEAV7mzIxxeI5IxFyzzkOZxS0PGfF_SK6wxCv2K8S0cZjMkv6b5bCqc0VBw
  ```

- header 中用于存放签名的生成算法

  ```json
  {"alg": "HS512"}
  ```

  payload 中用于存放用户名 sub、token的生成时间 created 和过期时间 exp

  ```json
  {
      "sub": "admin",
      "created": "1489079980393",  // 毫秒
      "exp": 1489684781	// 秒
  }
  ```

  signature 为以 header 和 payload 生成的签名，一旦 header 和 payload 被篡改，验证将失败。token 的生成依赖密钥secret，不同的secret生成的 token 是不同的，没有密钥，客户端无法伪造 token。

  ```json
  // secret 为加密算法的密钥
  String signature = HMACSHA512(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
  ```

#### 项目搭建

###### 依赖

```xml
<!--SpringSecurity依赖配置-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!--Hutool Java工具包-->
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>4.5.7</version>
</dependency>
<!--JWT(Json Web Token)登录支持-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.0</version>
</dependency>
```

###### JWT 工具类

| 用于生成和解析 JWT 的工具类。简单来说，jwt的生成和解析，依赖于 Jwts 这个库的 builder() 和 parser() 方法。

- `generateToken(UserDetails userDetails)`：用于根据登录用户信息生成 token
- `getUserNameFromToken(String token)`： 从 token 中获取登录用户的信息
- `validateToken(String token, UserDetails userDetails)`：判断 token 是否还有效

```java
@Component
public class JwtTokenUtil {
    private static final Logger LOGGER = LoggerFactory.getLogger(JwtTokenUtil.class);
    private static final String CLAIM_KEY_USERNAME = "sub";
    private static final String CLAIM_KEY_CREATED = "created";
    @Value("${jwt.secret}")
    private String secret;	// secret 就是 token 生成过程使用的密钥，保证 token 不会被伪造
    @Value("${jwt.expiration}")
    private Long expiration;	// expiration 是 token 的有效时间，用系统当前时间加上这个值，生成token的过期时间
    
    /**
     * 根据 payload 生成 JWT 的 Token
     */
    private String generateToken(Map<String, Object> claims) {
        return Jwts.builder()
            .setClaims(claims)							// 设置 payload
            .setExpiration(generateExpirationDate())	// 生成过期时间
            .signWith(SignatureAlgorithm.HS512, secret)	// 根据 secret 生成签名
            .compact();
    }
    
    /**
     * 根据 token 获取 JWT 中的 payload
     */
    private Claims getClaimsFromToken(String token) {
        Claims claims = null;
        try {
            claims = Jwts.parser()
                		.setSigningKey(secret)
                		.parseClaimsJws(token)
                		.getBody();
        } catch (Exception e) {
            LOGGER.info("JWT 格式验证失败:{}", token);
        }
        return claims;
    }
    
    /**
     * 生成 token 的过期时间，当前系统时间+有效时长=过期时间
     */
    public Date generateExpirationDate() {
        return new Date(System.currentTimeMillis() + expiration * 1000);
    }
    
    /**
     * 从 token 中获取登录用户名
     */
    public String getUserNameFromToken(String token) {
        String username;
        try {
            Claims claims = getClaimsFromToken(token);
            username = claims.getSubject();
        } catch (Exception e) {
            username = null;
        }
        return username;
    }
    
    /**
     * 验证 token 是否有效，验证用户名是否正确，以及 token 是否过期
     */
    public boolean validateToken(String token, UserDetails userDetails) {
        String username = getUserNameFromToken(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    /**
     * 判断 token 是否过期
     */
    private boolean isTokenExpired(String token) {
    	Date expiredDate = getExpiredDateFromToken(token);
        retrun expiredDate.before(new Date());	// 如果过期时间在当前时间之前，则说明 token 已经失效
    }
    
    /**
     * 从 token 中获取过期时间
     */
    private getExpiredDateFromToken(String token) {
        Claims claims = getClaimsFromToken(token);
        return claims.getExpiration();	// 返回 Date 类型
    }
    
    /**
     * 根据用户信息生成 token
     */
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put(CLAIM_KEY_USERNAME, userDetails.getUsername());	// 设置 sub 值
        claims.put(CLAIM_KEY_CREATED, new Date());	// 设置当前时间
        return generateToken(claims);
    }
    
    /**
     * 判断 token 是否可以刷新
     */
    public boolean canRefresh(String token) {
        return !isTokenExpired(token);	// 过期 token 不可以刷新
    }
    
    /**
     * 刷新 token
     */
    public String refreshToken(String token) {
        Claims claims = getClaimsFromToken(token);
        claims.put(CLAIM_KEY_CREATED, new Date()); // 更新创建时间
        return generateToken(claims);	// 更新创建时间后重新生成 token
    }                       
}
```

###### SpringSecurity 配置类

| 该类主要作用：

- `configure(HttpSecurity httpSecurity)`：用于配置需要拦截的url路径、jwt过滤器及出异常后的处理器；
- `configure(AuthenticationManagerBuilder auth)`：用于配置UserDetailsService及PasswordEncoder；
- `RestfulAccessDeniedHandler`：当用户没有访问权限时的处理器，用于返回JSON格式的处理结果；
- `RestAuthenticationEntryPoint`：当未登录或token失效时，返回JSON格式的结果；
- `UserDetailsService`：SpringSecurity 定义的核心接口，用于根据用户名获取用户信息，需要自行实现；
- `UserDetails`：SpringSecurity定义用于封装用户信息的类（主要是用户信息和权限），需要自行实现；
- `PasswordEncoder`：SpringSecurity定义的用于对密码进行编码及比对的接口，目前使用的是BCryptPasswordEncoder；
- `JwtAuthenticationTokenFilter`：在用户名和密码校验前添加的过滤器，如果有jwt的token，会自行根据token信息进行登录。

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled=true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Autowired
    private UmsAdminService adminService; 	// 查找用户，获取用户权限列表
    @Autowired
    private RestfulAccessDeniedHandler restfulAccessDeniedHandler;	// 自定义接口没有访问权限时的返回结果
    @Autowired
    private RestAuthenticationEntryPoint restAuthenticationEntryPoint;	// 自定义未登录或token失效时候的返回结果

    
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        httpSecurity.csrf()	// 由于使用 JWT，不需要 csrf
            	.disable()
            .sessionManagement() // 基于 token，所以不需要 session
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeRequests()
            .antMatchers(HttpMethod.GET,	// 放行静态资源，运行无授权访问
                        "/",
                        "/*.html",
                        "/favicon.ico",
                        "/**/*.html",
                        "/**/*.css",
                        "/**/*.js",
                        "/swagger-resources/**",
                        "/v2/api-docs/**"
             )
            .permitAll()
            .antMatchers("/admin/login", "/admin/register")	// 对登录注册允许匿名访问
            .permitAll()
            .antMatchers(HttpMethod.OPTIONS)	// 跨域请球会先进行一次 options 请球
            .permitAll()
            // .antMathchers("/**") // 测试时允许全部访问
            // .permitAll()
            .anyRequest()	// 除了上面外的所有请求，都需要鉴权认证
            .authenticated();
        // 禁用缓存
        httpSecurity.headers().cacheControl();
        // 添加 JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        // 添加自定义未授权和未登录结果返回
        httpSecurity.exceptionHandling()
            .accessDeniedHandler(restfulAccessDeniedHandler)
            .authenticationEntryPoint(restAuthenticationEntryPoint);
    }
    
    /**
     * 使用自定义用户和自定义的编码解码
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService())
            .passwordEncoder(passwordEncoder());
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public UserDetailsService userDetailsService() {
        // 获取登录用户信息
        return username -> {
            UmsAdmin admin = adminService.getAdminByUsername(username);
            if (admin != null) {
                List<UmsPermission> permissionList = adminService.getPermissionList(admin.getId());
                return new AdminUserDetails(admin, permissionList);
            }
            throw new UsernameNotFoundException("用户名或密码错误");
        };
    }
    
    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }
    
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
}
```

######  RestfulAccessDeniedHandler

| 以 JSON 形式返回没有权限的请求结果。 JSONUtil 是 Hutool 包下的一个工具类，可以将 Object 对象转为 JSON 对象。

```java
@Component
public class RestfulAccessDeniedHandler implements AccessDeniedHandler{
    @Override
    public void handle(HttpServletRequest request,
                       HttpServletResponse response,
                       AccessDeniedException e) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println(JSONUtil.parse(CommonResult.forbidden(e.getMessage())));
        response.getWriter().flush();
    }
}
```

###### RestAuthenticationEntryPoint

| 实现了 AuthenticationEntryPoint 接口，会处理未授权的请求

```java
@Component
public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println(JSONUtil.parse(CommonResult.unauthorized(authException.getMessage())));
        response.getWriter().flush();
    }
}
```

###### AdminUserDetails 类

| Spring Security 需要的用户详情，包含 User类，Permission 列表

```java
public class AdminUserDetails implements UserDetails {
    private UmsAdmin umsAdmin;
    private List<UmsPermission> permissionList;
    public AdminUserDetails(UmsAdmin umsAdmin, List<UmsPermission> permissionList) {
        this.umsAdmin = umsAdmin;
        this.permissionList = permissionList;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // 返回当前用户的权限
        return permissionList.stream()
            	.filter(permission -> permission.getValue() != null)	// 返回有value值的permission
            	.map(permission -> new SimpleGrantedAuthority(permission.getValue()))	// 将每个permission的value 用 SimpleGrantedAuthority 包装
            	.collect(Collectors.toList());
    }
    
    @Override
    public String getPassword() {
        return umsAdmin.getPassword();
    }

    @Override
    public String getUsername() {
        return umsAdmin.getUsername();
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
        return umsAdmin.getStatus().equals(1);
    }
    
}
```

###### JwtAuthenticationTokenFilter 类

| 登录授权过滤器，继承了 OncePerRequestFilter，在 Security 配置类中，调用 httpSecurity.addFilter()，实现如果请求中带有 token，可以取出 token 中的用户名，调用 Spring Security 的 API 进行操作。

可以看到，使用 token 进行操作时，会在请求中带上 key 为 Authorization ，value 为 Bearer 开头 + token 的 header。

所以在 filter 中，拦截请求，获取 key 为 Authrozation 的 header，去掉 tokenHead的值 “bearer”， 剩下的就是 token 的内容。

利用 token 的值，获取 username，通过 username 得到 UserDetails 类，其中包含 admin 和 permissionList，验证 token 有效后，将其加入 SecurityContextHolder 中。

![image-20220515223448238](C:\Users\lonel\AppData\Roaming\Typora\typora-user-images\image-20220515223448238.png)

```java
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {

    private static final Logger LOGGER = LoggerFactory.getLogger(JwtAuthenticationTokenFilter.class);
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    @Value("${jwt.tokenHeader}")
    private String tokenHeader;
    @Value("${jwt.tokenHead}")
    private String tokenHead;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws ServletException, IOException {
        String authHeader = request.getHeader(this.tokenHeader);
        if (authHeader != null && authHeader.startsWith(this.tokenHead)) {
            String authToken = authHeader.substring(this.tokenHead.length());
            String username = jwtTokenUtil.getUserNameFromToken(authToken);
            LOGGER.info("checking username:{}", username);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = this.userDetailsService.loadUserByUsername(username);
                if (jwtTokenUtil.validateToken(authToken, userDetails)) {
                    UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    LOGGER.info("authenticated user:{}", username);
                    SecurityContextHolder.getContext().setAuthentication(authentication);
                }
            }
        }
        chain.doFilter(request, response);
    }
}
```

###### UmsAdminController 类

| 处理返回结果，具体的业务逻辑还是放到 Service 类中

```java
@Api(tags = "后台用户管理")
@RequestMapping("/admin")
@RestController
public class UmsAdminController {

    @Autowired
    private UmsAdminService umsAdminService;
    @Value("${jwt.tokenHead}")
    private String tokenHead;
    
    @ApiOperation(value = "用户注册")
    @PostMapping("/register")
    public CommonResult<UmsAdmin> register(@RequestBody UmsAdmin umsAdminParam) {
        UmsAdmin umsAdmin = umsAdminService.register(umsAdminParam);
        if (null == umsAdmin) {
            return CommonResult.failed();
        }
        return CommonResult.success(umsAdmin);
    }
    
    @ApiOperation(value = "登陆以后返回Token")
    @PostMapping("/login")
    public CommonResult login(@RequestBody UmsAdminLoginParam umsAdminLoginParam) {
        String token = umsAdminService.login(umsAdminLoginParam.getUsername(), umsAdminLoginParam.getPassword());
        if (null == token) {
            return CommonResult.validateFailed("用户名或密码错误");
        }
        Map<String, String> tokenMap = new HashMap<>();
        tokenMap.put("token", token);
        tokenMap.put("tokenHead", tokenHead);	// 在 application.yml 的自定义配置
        return CommonResult.success(tokenMap);
    }
    
    @ApiOperation("获取用户所有权限（包括+-权限")
    @GetMapping("/permission/{adminId}")
    public CommonResult<List<UmsPermission>> getPermissionList(@PathVariable Long adminId) {
        List<UmsPermission> permissionList = umsAdminService.getPermissionList(adminId);
        return CommonResult.success(permissionList);
    }

}
```

###### UmsAdminServiceImpl 类

| 具体的业务在这个类中处理

```java
@Service
public class UmsAdminServiceImpl implements UmsAdminService {
    
    // 是 org.slf4j.Logger 类
    private static final Logger LOGGER = LoggerFactory.getLogger(UmsAdminServiceImpl.class);
    @Autowired
    private UserDetailsService userDetailsService;
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    @Autowired
    private PasswordEncoder passwordEncoder;
    @Autowired
    private UmsAdminMapper umsAdminMapper;
    @Autowired
    private UmsAdminRoleRelationDao adminRoleRelationDao;
    
    @Override
    public UmsAdmin getAdminByUsername(String username) {
        UmsAdminExample example = new UmsAdminExample();
        example.createCriteria().andUsernameEqualTo(username);
        List<UmsAdmin> adminList = adminMapper.selectByExample(example);
        if (adminList != null && adminList.size() > 0) {
            return adminList.get(0);
        }
        return null;
    }
    
    @Override
    public UmsAdmin register(UmsAdmin umsAdminParam) {
        UmsAdmin umsAdmin = new UmsAdmin();
        BeanUtils.copyProperties(umsAdminParam, umsAdmin);
        umsAdmin.setCreateTime(new Date());
        umsAdmin.setStatus(1);
        // 查询是否有相同用户名的用户
        UmsAdminExample example = new UmsAdminExample();
        example.createCriteria().andUsernameEqualTo(umsAdmin.getUsername());
        Lis<UmsAdmin> umsAdminList = adminMapper.selectByExample(example);
        if (umsAdminList.size() > 0) {
            return null;
        }
        // 对密码进行加密操作
        String encodedPassword = passwordEncoder.encode(umsAdmin.getPassword());
        umsAdmin.setPassword(encodedPassword);
        adminMapper.insert(umsAdmin);
        return umsAdmin;
    }
    
    @Override
    public String login(String username, String password) {
        String token = null;	// 返回结果
        try {
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            if (!passwordEncoder.matches(password, userDetails.getPassword())) {
                throw new BadCredentialsException("密码不正确");
            }
            UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authentication);
            token = jwtTokenUtil.generateToken(userDetails);
        } catch (AuthenticationException e) {
            LOGGER.warn("登录异常:{}", e.getMessage());
        }
        return token;
    }
    
    @Override
    public List<UmsPermission> getPermissionList(Long adminId) {
        return adminRoleRelationDao.getPermissionList(adminId);
    }
}
```

###### 给 PmsBrandController 接口中的方法添加访问权限

- 给查询接口添加pms:brand:read权限
- 给修改接口添加pms:brand:update权限
- 给删除接口添加pms:brand:delete权限
- 给添加接口添加pms:brand:create权限

如：

```java
@PreAuthorize("hasAuthority('pms:brand:read')")
public CommonResult<List<PmsBrand>> getBrandList() {
    return CommonResult.success(brandService.ListAllBrand());
}
```



**Note：@RequestBody 注解的使用**

在 UmsAdminController 中使用了 @RequestBody 注解，

```java
public CommonResult login(@RequestBody UmsAdminLoginParam umsAdminLoginParam) {
```

第一次将其去除后，Swagger-UI 中不会提供 JSON 格式的参数建议。

```json
{
    "username": "string",
    "password": "string"
}
```

提交请求后，Spring 也不会自动将 username 和 password 参数绑定到 UmsAdminLoginParam 类中。

@RequestBody 注解常用来处理 content-type 不是默认的 application/x-www-form-urlencoded 编码的内容，比如：application/json 或者是 application/xml 等。

也就是说，@RequestBody 注解可以将 JSON 格式的参数，绑定到对应的 Bean 中去。

也可以将其绑定到对应的字符串上。例如：

```js
$.ajax({
    url: "/login",
    type: "POST",
    data: '{
    			"username": "admin",
    			"pwd": "admin123"
			}',
    content-type: "application/json charset=utf-8",
    success: function(data) {
    	alert("request success!");
	}
});
```

在后台代码中可以这么接收参数：

```java
@PostMapping("/login")
public void login(@RequestBody String username, @RequestBody String pwd) {
    // TODO
}
```

| 注：一个请求中，RequestBody 只能有一个，RequestParam 可以有多个。