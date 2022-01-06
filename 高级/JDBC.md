# JDBC

##### 连接对象 Connection

先将相应的驱动导入项目，在程序中加载驱动。加载后即可获取连接对象。

```java
public class JDBCDemo {
    private static final String DATABASE_DRIVER = "oracle.jdbc.driver.OracleDriver";
    private static final String DATABASE_URL = "jdbc:oracle:thin:@localhost:1521:mldn";
    private static final String DATABASE_USER = "scott";
    private static final String DATABASE_PASSWORD = "tiger";
    
    public static void main(String[] args) {
        Connection conn = null;
        Class.forName(DATABASE_DRIVER);
        conn = DriverManager.getConnection(DATABASE_URL, DATABASE_USER, DATABASE_PASSWORD);
        System.out.println(conn);
    }
}
```

DriverManager 是工厂设计模式里的 Factory，Connection 接口是一套 JDBC 的标准，具体生成的子类，由加载的驱动程序决定。

每一个 Connection 描述的是每一个用户的连接。

##### 数据库操作 Statement

Statement 描述的是每一个 SQL 操作。

```java
Statement statement = conn.createStatement();
```

- 更新处理（INSERT, UPDATE, DELETE）: public int executeUpdate(String sql) throws SQLException;
- 查询处理（SELECT, 统计查询、复杂查询）：public ResultSet executeQuery(String sql) throws SQLException;

数据的更新操作，返回受影响行数的 int 就可以了。而查询操作，需要返回具体查询到的内容，所以在 Java 里使用了 ResultSet 接口，来描述查询结果。

ResultSet 描述的其实是一个行列组成的表格。行代表查询到的结果数量，列代表数据库中的字段。

`next()`方法返回一个 boolean 值，如果还有数据，则返回 true。

```java
ResultSet rs = statement.executeQuery(sql);
while(rs.next()) {
    int id = rs.getInt(1);
    String title = rs.getString(2);
    // 参数中的数字，表示字段在表格中的位置，从1开始。
}
```

Statement 接口存在的问题：

- 字符串拼接，太乱
- 不能很好的描述日期的形式
- 对于敏感的字符数据无法进行合理拼凑
- 可能被 sql 注入攻击

##### PreparedStatement

为了解决 Statement 字符串拼接的问题，Java 提供了一个 PreparedStatement，可以进行参数的设置。

```java
PreparedStatement ps = conn.prepareStatement(sql);
```

使用 PreparedStatement 时，sql 中要设置的数据，使用 "?" 作为占位符。

```java
String sql = "insert into user(id, name, age) values (?, ?, ?)";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setInt(1, 5);
ps.setString(2, "jack");
ps.setDouble(3, 23);
ResultSet rs = ps.executeQuery(); // 获取查询结果
```

##### java.util.Date

Date 类下有三个子类，都是 java.sql 包下的。

java.sql.Date 描述日期数据，java.sql.Time 描述时间数据，java.sql.Timestamp 描述日期时间。

每一个子类都可以通过 getTime()  方法，转换成 long 数据。再利用 java.util.Date 的构造方法，转换为 java.util.Date

