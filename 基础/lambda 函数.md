# lambda 表达式

### lambda的使用

实际上是对接口中独特的抽象方法的一种简化。

例如，有如下接口，其中有唯一一个抽象方法

```java
interface IMath {
    int add(int x, int y);
}

public class JavaDemo {
    public static void main(String[] args) {
        IMath math = (t1, t2) -> t1 + t2;
        System.out.println(math.add(12, 23));
    }
}
```

假如 IMath 接口中还有其他的抽象方法，则不能使用 lambda 这样简化。如果是非抽象方法则可以。

```java
interface IMath {
    int add(int x, int y);
    public default void print() {
        System.out.println("I'm not a abstract method.");
    }
}
```

### 方法的引用

在 JAVA 中除了对象可以被引用外，方法也可以被引用。

例如：

- 引用类的静态方法

  ```java
  interface IFunction<P, R> {
      R change(P p);
  }
  
  public class JavaDemo {
      public static void main(String[] args) {
          IFunction<Integer, String> fun = String :: valueOf;
          String str = fun.change(100);
          System.out.println(str.length());
      }
  }
  ```

  在这里没有手动去实现`IFunction` 接口里的 `change` 方法，而是把方法的实现指向 `String` 类的 `valueOf` 方法去。

- 引用类的成员方法（需要实例化对象）

  ```java
  interface IFunction<P> {
      R upper();
  }
  
  public class JavaDemo {
      public static void main(String[] args) {
          IFunction<String> fun = "helloworld" :: toUpperCase; // 这里使用了字符串实例化对象
          System.out.println(fun.upper());
      }
  }
  ```

- 引用类的成员方法（不使用实例化对象）

  ```java
  interface IFunction<P> {
      int compare(P p1, P p2);
  }
  
  public class JavaDemo {
      public static void main(String[] args) {
          IFunction<String> fun = String :: compareTo ;
          System.out.println(fun.compare("a", "A"));
      }
  }
  ```

- 引用类的构造方法

  ```java
  class Person {
      private String name;
      private int age;
      public Person(String name, int age) {
          this.name = name;
          this.age = age;
      }
      public String toString() {
          return "姓名：" + this.name + "、年龄：" + this.age;
      }
  }
  
  interface IFunction<P> {
      P create(String s, int a);
  }
  
  public class JavaDemo {
  	public static void main(String[] args) {
  		IFunction<Person> fun = Person :: new;
  		System.out.println(fun.create("张三", 20));
  	}
  }
  ```

### 内建函数式接口

内建函数式接口可以为我们剩下每次自定义接口的麻烦。

- 功能型：接收一个参数然后返回一个结果的 `Function` 

  ```java
  import java.util.function.*;
  
  public class JavaDemo {
      public static void main(String[] args) {
          Function<String, Boolean> fun = "**hello" :: startsWith;
          System.out.println(fun.apply("**"));
      }
  }
  ```

  在上面的例子中，使用了 String 类的成员方法 `startsWith` 。

- 消费型：只进行数据处理操作，不返回结果的 `Consumer`

  JAVA 里，`System.out.println() ` 这个方法就是消费型的方法。

  ```java
  import java.util.function.*;
  
  public class JavaDemo {
      public static void main(String[] args) {
          Consumer<String> con = System.out :: println;
          con.accept("helloworld");
      }
  }
  ```

- 供给型：不需要接收参数，有返回结果的 `Supplier`

  ```java
  import java.util.function.*;
  
  public class JavaDemo {
      public static void main(String[] args) {
          Supplier<String> sup = "www.mldn.cn" :: toUpperCase;
          System.out.println(sup.get());
      }
  }
  ```

- 断言型：进行判断处理的 `Predicate`

  ```java
  import java.util.function.*;
  
  public class JavaDemo {
      public static void main(String[] args) {
          Predicate<String> pre = "MLDN" :: equalsIgnoreCase;
          System.out.println(pre.test("mldn"));
      }
  }
  ```

  