### 创建 REST 服务

@RestController 注解有两个作用：

- 类似于 @Controller、@Service的构造型注解，能够让类被组件扫描功能发现
- 控制器中所有处理器方法的返回值都要直接写入响应体，而不是放入模型中传递给一个视图进行渲染（ModelAndView）

假如没有使用 @RestController，那么我们就要为每个处理器方法加上 @ResponseBody 注解作为替代。或者返回 ResponseEntity 对象。

在 @RequestMapping 设置 **produces** 属性，例如

```java
@RequestMapping(path="/design", produces="application/json", "text/xml")
```

可以指明类中所有的处理器方法只会处理 Accept 头信息包含“application/json“的请求。它不仅会限制 API 只生成 JSON 结果，还会允许其他 Controller 处理相同路径的请求，只要这些请求不要求 JSON 格式输出即可。

@CrossOrigin 注解允许来自任何域的客户端消费该 API，突破 **CORS（跨域资源共享）**的限制。

```java
@GetMapping("/{id}")
public Taco tacoById(@PathVariable("id") Long id) {
    Optional<Taco> optTaco = tacoRepo.findById(id);
    if (optTaco.isPresent()) {
        return optTaco.get();
    }
    return null;
}
```

@PathVariable 让请求路径中的占位符 {id} 和方法参数 id 关联了起来。

findById() 返回了一个 Optional<Taco>，如果能够匹配到一个 Taco，可以 get() 方法返回实际的 Taco。

如果该 ID 无法匹配任何 taco，那么将会返回 null。这样处理并不完美，浏览器会接收到一个空的响应体以及200（OK）的 HTTP 状态码。我们可以使用 ResponseEntity<Taco> 将返回值和状态码包装。

```java
@GetMapping("/{id}")
public ResponseEntity<Taco> tacoById(@PathVariable("id") Long id) {
    Optional<Taco> optTaco = tacoRepo.findById(id);
    if (OptTaco.isPresent()) {
        return new ResponseEntity<>(optTaco.get(), HttpStatus.OK);
    }
    return new ResponseEntity<>(null, HttpStatus.NOT_FOUND);
}
```

在处理 POST 方法的时候：

```java
@PostMapping(consumes="application/json")
@ResponseStatus(HttpStatus.CREATED)
public Taco postTaco(@RequestBody Taco taco) {
    return tacoRepo.save(taco);
}
```

@PostMapping 设置了 **consumes** 属性，指定了请求输入，而 **produces** 指定了请求输出。consumes 属性表明该方法只会处理 Content-type 与 application/json 相匹配的请求。

@RequestBody 注解表明请求应该被转换为一个 TAco 对象并绑定到该参数上。由于请求中包含请求参数（查询参数，表单参数）和请求体，有了 @RequestBody 注解，Spring MVC 能够确保请求体中的 JSON 会被绑定到 Taco 对象上。

@ResponseStatus(HttpStatus.CREATED) 设置了方法默认返回 201（CREATED）的 HTTP 状态码。

PUT 经常被用来更新资源，但它的语义其实是 GET 的对立面。GET 是从服务器往客户端传输数据，PUT 则是从客户端往服务器传输数据。

所以，PUT 的真正目的时执行大规模替换操作，比如替换整个 Order。而 PATCH 的目的才是对资源数据打补丁或局部更新，比如修改 Order 的某个属性值。

但是请**注意**：PUT 和 PATCH 只能用来指定某个方法能够处理什么类型的请求，并没有规定该如何处理请求。尽管 PATCH 在语义上表示局部更新，但是在处理器方法中实际编写代码执行更新的还是我们自己。

对于 DELETE 方法：

我们通常使用 @ResponseStatus(HttpStatus.NOT_FOUND)。对于已经不存在的资源，我们没有必要返回任何的资源数据给客户端，所有 DELETE 请求通常并没有响应体，方法的返回值为 void。