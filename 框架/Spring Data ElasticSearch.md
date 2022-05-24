### Spring Data ElasticSearch

##### 常用注解

###### @Document

```java
// 标示映射到 Elasticsearch 文档上的领域对象
public @interface Document {
    // 索引库名次，mysql中数据库的概念
    String indexName();
    // 文档类型，Mysql 中表的概念
    String type() default "";
    // 默认分片数
    short shards() default 5;
    // 默认副本数量
    short replicas() default 1;
}
```

###### @Id

```java
// 表示文档的id，文档可以认为是 Mysql 中表行的概念
public @interface Id {
    
}
```

@Field

```java
public @interface Field {
    // 文档中字段的类型
    FieldType type() default FieldType.AUTO;
    // 是否建立倒排索引
    boolean index() default true;
    // 是否进行存储
    boolean store() default false;
    // 分词器名次
    String analyzer() default "";
}
```

```java
// 为文档指定元数据类型
public enum FieldType {
    Text, // 会进行分词并建立了索引的字符类型
    Integer,
    Long,
    Date,
    Float,
    Double,
    Boolean,
    Object,
    Auto, // 自动判断字段类型
    Nested, // 嵌套对象类型
    Ip,
    Attachment,
    Keyword // 不会进行分词建立索引的类型
}
```

##### Spring Data 方式的数据操作

继承 ElasticsearchRepository 接口，可以获得常用的数据操作方法

![img](http://www.macrozheng.com/assets/arch_screen_31.17948c33.png)

可以使用衍生查询

| 在接口中直接指定**查询方法名称便可查询**，无需进行实现，如商品表中有商品名称、标题和关键字，直接定义以下查询，就可以对这三个字段进行全文搜索。

```java
/**
 * 搜索查询
 * @Param name			商品名称
 * @Param subTitle 		商品标题
 * @Param keywords 		商品关键字
 * @Param page			分页信息
 */
Page<EsProduct> findByNameOrSubTitleOrKeywords(String name, String subTitle, String keywords, Pageable page);
```

使用 @Query 注解，可以自定义 Elasticsearch 的 DSL 语句进行查询

```java
@Query("{\"bool\" : {\"must\" : {\"field\" : {\"name\" : \"?0\"}}}}")
Page<EsProduct> findByName(String name,Pageable pageable);
```

##### 整合 Elasticsearch

###### 依赖

```xml
<!--Elasticsearch相关依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch<artifactId>
</dependency>
```

###### 修改 SpringBoot 配置文件

```yml
spring:
  data:
    elasticsearch:
      repository:
        enabled: true
      cluster-nodes: localhost:9300 # es的连接地址和端口号
      cluster-name: elasticsearch # es集群的名称
```

###### 添加商品文档对象 EsProduct

| 不需要中文分词的字段设置成 @Field(type = FieldType.Keyword) 类型，需要中文分词的设置成 @Filed(type = FieldType.Text, analyzer = "ik_max_word") 类型

```java
@Document(indexName = "pms", type = "product", shards = 1, replicas = 0)
public class EsProduct implements Serializable {
    private static final long serialVersionUID = -1L;
    @Id
    private Long id;
    @Field(type = FieldType.Keyword)
    private String productSn;
    private Long brandId;
    @Field(type = FieldType.Keyword)
    private String brandName;
    private Long productCategoryId;
    @Field(type = FieldType.Keyword)
    private String productCategoryName;
    private String pic;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String name;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String subTitle;
    @Field(analyzer = "ik_max_word", type = FieldType.Text)
    private String keywords;
    private BigDecimal price;
    private Integer sale;
    private Integer newStatus;
    private Integer recommandStatus;
    private Integer stock;
    private Integer promotionType;
    private Integer sort;
    @Field(type = FieldType.Nested)
    private List<EsProductAttributeValue> attrValueList;
    
    // 省略 getter 和 setter 方法
}
```

###### 添加 EsProductRepository 接口用于操作 Elasticsearch

| 继承 ElasticsearchRepository 接口，这样就拥有了一些基本的 Elasticsearch 数据操作方法，同时定义了一个衍生查询方法。

```java
public interface EsproductRepository extends ElasticsearchRepository<EsProduct, Long> {
    
    Page<EsProduct> findByNameOrSubTitleOrKeywords(String name, String subTitle, String keywords, Pageable page);
}
```

###### 商品搜索管理 Service 实现类

```java
@Service
public class EsProductServiceImpl implements EsProductService {
    private static final Logger LOGGER = LoggerFactory.getLogger(EsProductServiceImpl.class);
    @Autowired
    private EsProductDao productDao;
    @Autowired
    private EsProductRepository productRepository;
    
    @Override
    public int importAll() {
        List<EsProduct> esProductList = productDao.getAllEsProductList(null);
        Iterable<EsProduct> esProductIterable = productRepository.saveAll(esProductList);
        Iterable<EsProduct> iterator = esProductIterable.iterator();
        int result = 0;
        while (iterator.hasNext()) {
            result++;
            iterator.next();
        }
        return result;
    }
    
     @Override
    public void delete(Long id) {
        productRepository.deleteById(id);
    }

    @Override
    public EsProduct create(Long id) {
        EsProduct result = null;
        List<EsProduct> esProductList = productDao.getAllEsProductList(id);
        if (esProductList.size() > 0) {
            EsProduct esProduct = esProductList.get(0);
            result = productRepository.save(esProduct);
        }
        return result;
    }

    @Override
    public void delete(List<Long> ids) {
        if (!CollectionUtils.isEmpty(ids)) {
            List<EsProduct> esProductList = new ArrayList<>();
            for (Long id : ids) {
                EsProduct esProduct = new EsProduct();
                esProduct.setId(id);
                esProductList.add(esProduct);
            }
            productRepository.deleteAll(esProductList);
        }
    }

    @Override
    public Page<EsProduct> search(String keyword, Integer pageNum, Integer pageSize) {
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        return productRepository.findByNameOrSubTitleOrKeywords(keyword, keyword, keyword, pageable);
    }
}
```

使用的时候要先将数据库中数据导入到 Elasticsearch 中。