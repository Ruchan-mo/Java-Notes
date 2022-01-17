### 02-Spring 实战——开发 Web 应用

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

