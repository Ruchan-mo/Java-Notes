### 创建 REST 服务

#### 编写 RESTful 控制器

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

#### 启用超媒体

超媒体（HATEOAS）是一种创建子描述 API 的方式。简单来说，想要对某个资源进行操作，传统的方法是将其 id 属性字符串拼接到固定的 URL 上，这样会导致后期的很多问题。HATEOAS 使得 API 在返回的资源中会包含相关资源的链接。这种在 JSON 响应中嵌入超链接的简单格式被称为 HAL。

```json
{
    "_embedded": {
        "tacolist": [
            {
                "name":"Veg-Out",
                "createdAt": "2018-01-31T20:15:53.219+0000",
                "ingredients": [
                    {
                        "name": "Flour Tortilla",
                    	"type": "WRAP",
                    	"_links": {
                    		"self": {
                    			"href": "http://localhost:8080/ingredients/FLTO"
                    		}
                    	}
                    }
                ],
        		"_links": {
                    "self": {
                    	"href": "http://localhost:8080/design/4"
                    }
                }
            }
        ]
    },
    "_links": {
        "recents": {
            "href": "http://localhost:8080/design/recent"
        }
    }
}
```

上面的例子中，tacolist、design、ingredient 都有属于本资源的 self 链接（tacolist 命名为 recents），这是借助 Resource 和Resources 类完成的。

Resource 代表一个资源，如 Taco、Ingredient。

Resources 代表资源的集合，如 List<Taco>，Resources 类实现了 Iterable。

**对于 Resources** ，首先把 List<Taco> 类型包装转为 Resources 类型，然后用 `add()` 方法将链接添加进去：

```java
Resources<Resource<Taco>> recentResources = Resources.wrap(tacos);
recentResources.add(
	new Link("http://localhost:8080/design/recent", "recents"));
```

 使用 ControllerLinkBuilder 解决链接的硬编码问题：

```java
recentResources.add(
	ControllerLinkBuilder.linkTo(DesignTacoController.class)
    						.slash("recent")
    						.withRel("recents"));
```

`slash()` 方法会为 URL 添加斜线（/）和给定的值，上面的例子中在 URL 后面拼接了“/recent”。我们还可以使用 ControllerLinkBuilder 的另一个方法消除拼接。

```java
recentResources.add(
	ControllerLinkBuilder.linkTo(
        ControllerLinkBuilder.methodOn(DesignTacoController.class).recentTacos())
    						.withRel("recents"));
```

**对于 Resource**，我们可以遍历 List<Taco> ，然后一个个添加 link，但这种方式太过麻烦。

我们可以将用 TacoResource 替代 Taco，它和 Taco 类基本相同，但是扩展了 ResourceSupport，继承了一个 Link 对象的列表和管理链接列表的方法。

```java
public class TacoResource extends ResourceSupport {
    @Getter
    private final String name;
    
    @Getter
    private final Date createdAt;
    
    @Getter
    private final List<Ingredient> ingredients;
    
    public TacoResource(Taco taco) {
        this.name = taco.getName();
        this.createdAt = taco.getCreatedAt();
        this.ingredients = taco.getIngredients();
    }
}
```

除此之外，这个 TacoResource 并没有包含 Taco 的 id 属性，因为我们没必要在 API 中暴露 ID 属性，如果需要操作资源，可以使用 self 链接。

现在，我们只需要一个 resource assembler，就可以将 Taco 集合转换为 Resources<TacoResource>。

```java
public class RacoResourceAssembler extends ResourceAssemblerSupport<Taco, TacoResource> {
    
    // 告诉超类在创建 TacoResource 中的链接时会使用 DesignTacoController 来确定所有 URL 的基础路径
    public TacoResourceAssembler() {
        super(DesignTacoController.class, TacoResource.class);
    }
    
    // 基于给定的 Taco 实例化 TacoResource
    @Override
    protected TacoResource instantiateResource(Taco taco) {
        return new TacoResource(taco);
    }
    
    // 通过 Taco 创建 TacoResource，并且要设置一个 self 链接，这个链接的 URL 是根据 Taco 对象的 id 属性衍生出来的
    // 在其内部会调用 instantiateResource()
    @Override
    public TacoResource toResource(Taco taco) {
        return createResourceWithId(taco.getId(), taco);
    }
}
```

调整后的 `recentTacos()` 方法：

```java
List<TacoResource> tacoResources = new TacoResourceAssembler().toResources(tacos);
Resources<TacoResource> recentResources = new Resources<TacoResource>(tacoResources);
...
```

对于 Taco 中的 Ingredients 集合，也是用相同的方法新建 IngredientResouce 和 IngredientResourceAssembler 类，再将 TacoResource 修改以下，让它携带 IngredientResource 对象

```java
@Relation(collectionRelation = "tacos", value = "taco")
public class TacoResource extends ResourceSupport {

    private static final IngredientResourceAssembler
            ingredientAssembler = new IngredientResourceAssembler();

    @Getter
    private final String name;

    @Getter
    private final Date createdAt;

    @Getter
    private final List<IngredientResource> ingredients;

    public TacoResource(Taco taco) {
        this.name = taco.getName();
        this.createdAt = taco.getCreatedAt();
        this.ingredients = ingredientAssembler.toResources(taco.getIngredients());

    }
}
```

@Reletion 注解能够消除 JSON 字段名和 Java 代码中定义的资源类名之间的耦合。通过 @Relation 注解，我们能指定 JSON 中的字段名：

```json
{
    "_embedded": {
        "tacos": [
            ...
        ]
    }
}
```

#### Spring Data Rest 自动创建 API

只要使用 Spring Data 实现了 repository，就可以使用 Spring Data Rest。将它的依赖添加到构建文件，就能得到一套 API。

开启 Spring Data Rest 后，将 repository 直接暴露为 API，通过 API 可以直接操作数据库中的内容。

例如对 “localhost:8080/ingredients” 进行 get，post，put，delete可以操作 ingredient 资源。

可以在 属性文件中设置 API 的基础路径：

```yaml
spring:
  data:
    rest:
      base-path: /api
```

向 API 的基础路径发送 GET 请求就能得到所有端点的链接：

```bash
$ curl localhost:8080/api
{
	"_links": {
		"orders" : {
			"href" : "http://localhost:8080/api/orders"
		},
		"ingredients" : {
			"href" : "http://localhost:8080/api/ingredients"
		},
		"tacoes" : {
			"href" : "http://localhost:8080/api/tacoes{?page,size,sort}",
			"templated" : true
		},
		...
	}
}
```

通过为 Taco 添加一个简单的注解可以修改 JSON 输出结果：

```java
@Data
@Entity
@RestResource(rel="tacos", path="tacos")
public class Taco {
    ...
}
```

现在，tacos 有了正确的复数形式：

```json
"tacos" : {
    "href" : "http://localhost:8080/api/tacos{?page,size,sort}",
    "templated" : true
}
```

##### 添加自定义端点

Spring Data Rest 可以为 CRUD 操作方便地创建端点，但是有时候我们需要创建复杂的端点。

使用 @RestController 实现有一些缺点：

- 控制器路径没有映射到 Spring Data Rest 的基础路径下，如果后期基础路径发生变化，就又要再次修改。
- 返回资源不会包含超链接

我们可以使用 @RepositoryRestController 注解解决基础路径的问题，用 TacoResource 解决超链接问题。

```java
@RepositoryRestController
public class RecentTacosController {

    private TacoRepository tacoRepo;

    public RecentTacosController(TacoRepository tacoRepo) {
        this.tacoRepo = tacoRepo;
    }

    @GetMapping(path="/tacos/recent", produces="application/hal+json")
    public ResponseEntity<Resources<TacoResource>> recentTacos() {
        PageRequest page = PageRequest.of(
                0, 12, Sort.by("createdAt").descending());
        List<Taco> tacos = tacoRepo.findAll(page).getContent();

        List<TacoResource> tacoResources =
                new TacoResourceAssembler().toResources(tacos);
        Resources<TacoResource> recentResources =
                new Resources<TacoResource>(tacoResources);

        recentResources.add(
                linkTo(methodOn(RecentTacosController.class).recentTacos())
                        .withRel("recents"));
        return new ResponseEntity<>(recentResources, HttpStatus.OK);
    }

}
```

##### 为 Spring Data 端点添加自定义的超链接

”api/tacos“ 返回的超链接中没有 recent tacos 端点，我们可以使用配置添加。

”api/tacos“ 端点返回的类型应该是 PagedResources<Resource<Taco>> 类型的资源，通过声明资源处理器（resource processor）bean，我们可以为其添加一个 recents 链接。

```java
@Configuration
public class SpringDataRestConfiguration {

    @Bean
    public ResourceProcessor<PagedResources<Resource<Taco>>> tacoProcessor(EntityLinks links) {

        return new ResourceProcessor<PagedResources<Resource<Taco>>>() {
            @Override
            public PagedResources<Resource<Taco>> process(PagedResources<Resource<Taco>> resource) {
                resource.add(
                        links.linkFor(Taco.class)
                                .slash("recent")
                                .withRel("recents")
                );
                return resource;
            }
        };
    }
}
```

