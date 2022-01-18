### 03-Spring 实战——持久化

在处理关系型数据的时候，Spring 支持 JDBC 和 JPA 两种形式。

##### 从 JDBC 到 JPA

Spring 对 JDBC 的支持主要是 JdbcTemplate。

**不使用 JdbcTemplate ：**

```java
@Override
public Ingredient findOne(String id) {
    Connection connection = null;
    PreparedStatement statement = null;
    ResultSet resultSet = null;
    try {
        connection = dataSource.getConnection();
        statement = connection.prepareStatement(
        "select id, name, type from Ingredient where id=?");
        statement.setString(1, id);
        resultSet = statement.executeQuery();
        Ingredient ingredient = null;
       	if (resultSet.next()) {
            ingredient = new Ingredient(
            	resultSet.getString("id");
                resultSet.getString("name");
                Ingredient.Type.valueOf(resultSet.getString("type")));
        }
        return ingredient;
    } catch (SQLException e) {
        // Do something...
    } finally {
        if (resultSet != null) {
            try {
                resultSet.close();
            } catch (SQLException e) {}
        }
        if (statement != null) {
            try {
                statement.close();
            } catch (SQLException e) {}
        }
        if (connection != null) {
            try {
                connection.close();
            } catch (SQLException e) {}
        }
    }
    return null;
}
```

大量重复的创建连接、创建语句以及关闭连接、语句和结果集使得代码看起来非常繁琐复杂。

**使用 JdbcTemplate**

查询一个对象：

```java
private JdbcTemplate jdbc;

@Override
public Ingredient findOne(String id){ 
	return jdbc.queryForObject(
    	"select id, name, type from Ingredient where id=?",
        this::mapRowToIngredient, id);
    );
}

private Ingredient mapRowToIngredient(ResultSet rs, int rowNum) throws SQLException {
    return new Ingredient(
    	rs.getString("id"),
        rs.getString("name"),
        Ingredinet.Type.valueOf(rs.getString("type")));
}
```

使用 JdbcTemplate 可以让我们仅仅关注执行查询 `queryForObject()` 和映射对象 `mapRowToIngredient()` 上。

查询多个对象的集合：

```java
@Override
public Iterable<Ingredient> findAll() {
    return jdbc.query("select id, name, type from Ingredient",
                     this::mapRowToIngredient);
}
```

插入一行数据：

```java
@Override
public Ingredient save(Ingredient ingredient) {
    jdbc.update(
    	"insert into Ingredient (id, name, type) values (?, ?, ?)",
        ingredient.getId(),
        ingredient.getName(),
        ingredient.getType.toString());
    return ingredient;
}
```

复杂插入：

- 使用 `update()` 方法

  ```java
  private long saveTacoInfo(Taco taco) {
      taco.setCreatedAt(new Date());
      PreparedStatementCreator psc = 
          new PreparedStatementCreatorFactory(	// 工厂类，传入 sql 语句和每个参数的类型
      		"insert into Taco (name, createdAt) values (?, ?)",
          	Types.VARCHAR, Types.TIMESTAMP
      	).newPreparedStatementCreator(
      		Arrays.asList(
              	taco.getName(),
                  new Timestamp(taco.getCreatedAt().getTime())));
      KeyHolder keyHolder = new GeneratedKeyHolder();
      jdbc.update(psc, keyHolder);	// 传入 KeyHolder，查询后便可以获取到taco的ID
   
      return keyHolder.getKey().longValue();
  }
  ```

  

- 使用 SimpleJdbcInsert 包装器类

  先创建 SimpleJdbcInsert，插入指定的表中。使用 ObjectMapper 将 Order转换为 Map

  ```java
  private SimpleJdbcInsert orderInserter;
  private SimpleJdbcInsert orderTacoInserter;
  private ObjectMapper objectMapper;
  
  @Autowired
  public JdbcOrderRepository(JdbcTemplate jdbc) {
      this.orderInserter = new SimpleJdbcInsert(jdbc)
          .withTableName("Taco_Order")
          .usingGeneratedKeyColumns("id");
      this.orderTacoInserter = new SimpleJdbcInsert(jdbc)
          .withTableName("Taco_Order_Tacos");
      this.objectMapper = new ObjectMapper();
  }
  ```

  使用：

  ```java
  private long saveOrderDetails(Order order) {
      Map<String, Object> values = objectMapper.convertValue(order, Map.class);
      values.put("placedAt", order.getPlacedAt()); 	// Date 类型会被 ObjectMapper 转换为 long
      long orderId = 
          orderInserter
          	.executeAndReturnKey(values)	// 返回 Number 类型的 ID
          	.longValue();
      return orderId;
  }
  
  private void saveTacoToOrder(Taco taco, long orderId) {
      Map<String, Object> values = new HashMap<>();
      values.put("tacoOrder", orderId);
      values.put("taco", taco.getId());
      orderTacoInserter.execute(values);
  }
  ```

**使用 Spring Data JPA**

Spring Data 提供了一项特性，就是基于 repository 规范接口自动生成 repository 的功能。

准备：

为了使用 Spring Data JPA，我们需要先将 domain 对象标注为实体。

```java
@Data
@RequiredArgsConstructor
@NoArgsConstructor(access = AccessLevel.PRIVATE, force = true)
@Entity
public class Ingredient {
    @Id
    private final String id;
    private final String name;
    private final Type type;
    ...
}
```

@Entity 注解可以将 Ingredient 声明为 JPA 实体，另外它的 id 属性还需要添加 @Id 注解。

@NoArgsConstructor JPA 需要实体有一个无参构造器，我们不想注解使用它，所以 access 设置为 PRIVATE，因为类中有 final 的属性，我们将 force 设置为 true，这样 Lombok 生成的构造器就会将它们设置为 null。

```java
@Data
@Entity
@Table(name="taco")
public class Taco {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    
    @ManyToMany(targetEntity = Ingredient.class)
    private List<Ingredient> ingredients;
    
    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}
```

@GeneratedValue 指定了自动生成 ID 值。

@ManyToMany 注解声明了 Taco 与其关联的 Ingredient 列表之间的关系：多对多。

@PrePersist 注解的方法会在 Taco 持久化之前执行，将 createdAt 设置为当前的日期和时间。

@Table 注解表明了实体应该持久化到数据库中的表名。hibernate 对表名大小写敏感，而数据库中表名是不区分大小写的，类似 “Taco Order” 的表名会被自动转为 “taco_order”，@Table 注解中应该使用小写表名，否则会导致报错。

声明：

将 repository 继承 CrudRepository 接口即可

```java
public interface IngredientRepository extends CrudRepository<Ingredient, String> {
    
}
```

CrudRepository 第一个参数是要持久化的实体类，第二个参数是实体 ID 属性的类型。它定义了很多用于 CRUD 操作的方法。

使用：

```java
@Autowired
private IngredientRepository ingredientRepo;

...
	ingredientRepo.findAll().forEach(i -> ingredients.add(i));
```

自定义 JPA repository

```java
List<Order> findByDeliveryZip(String deliveryZip); 	// 获取投递到指定 Zip 的订单
List<Order> readOrdersByDeliveryZipAndPlacedAtBetween(
	String deliveryZip, Date startDate, Date endDate); 	// 查找投递到指定 Zip 且在一定范围内的订单，read, get, find 都有一样的作用
```

较为复杂的查询可以使用 @Query 注解，明确指明方法调用时要执行的查询

```java
@Query("from Order o where o.deliveryCity='Seattle'")
List<Order> readOrdersDeliveredInSeattle();
```

Spring Boot  会在启动的时候执行根类路径下名为 schema.sql 和 data.sql 的文件，我们可以把定义表结构的 schema.sql 和 插入数据的 data.sql 文件放在 "src/main/resources" 文件夹下。

schema.sql

```sql
create table if not exists Taco_Ingredients (
  taco bigint not null,
  ingredient varchar(4) not null
);
```

data.sql

```sql
delete from Taco_Ingredients;
insert into Ingredient (id, name, type) values ('FLTO', 'Flour Tortilla', 'WRAP');
```

