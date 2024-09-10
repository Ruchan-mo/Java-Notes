## Stream 流

Stream 提供了一种高效且易于使用的处理数据的方式。（类似 erlang 的 lists 库）

Stream 不存储元素，而是数据渠道，用到操作数据源（集合、数组）所生成的元素序列。集合负责存储数据，Stream 负责处理数据。

通过一个数据源可以创建 Stream（如集合、数组等），获取一个流。

中间操作：每次处理都会返回一个持有结果的新 Stream，因为中间操作可以是个操作链，可以对数据源的数据进行 n 次处理（在终结操作前，不会真正执行）。

终止操作：产生结果，并结束 Stream。返回值类型不再是 Stream，因此不可以再执行新的中间操作。

#### 创建方式

##### 通过集合

Collection 接口提供了获取流的方法

```java
default Stream<E> stream();
```

##### 通过数组

Arrays 的静态方法 `stream()` 可以获取数据流

```java
static <T> Stream<T> stream(T[] array);
```

##### 通过 Stream 的 `of()` 方法

```java
public static <T> Stream<T> of(T... values);
```

#### 中间操作

`filter(Predicate p)`

过滤符合指定条件的元素

`distinct()`

去重，根据元素的 `equals()` 方法判断

`limit(long maxSize)` 

保留指定个数

`skip(long n )`

跳过元素，返回一个舍弃掉前 n 个元素的流

`sorted()`

产生一个按照自然顺序排序的流

`sorted(Comparator<T> comparator)` 

产生一个排序的流，规则来自传入的 comparator

`map(Function f)`

接受一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素

`flatMap(Funtion f)` 

接受一个函数作为参数，讲流中的每个值映射成一个新的流，然后把所有流连接成一个流

#### 终止操作

`boolean allMatch(Predicate p)`

全部匹配

`boolean anyMatch(Predicate p)` 

任意匹配

`boolean noneMatch(Predicate p)`

全部不匹配

`Optional<T> findFirst()`

返回第一个元素

`long count()`

计数

`Optional<T> max()`

返回最大值

`optional<T> min()`

返回最小值

`void forEach(Consumer c)`

迭代

#### 示例

```java
import java.util.*;
import java.util.stream.IntStream;
import java.util.stream.Stream;

import org.junit.Test;

public class TestStream {

    static List<Student> list = new ArrayList<Student>();
    static {
        Student stu1 = new Student("赵四", 22, "深圳", '男', 88);
        Student stu2 = new Student("大拿", 21, "广州", '女', 50);
        Student stu3 = new Student("赵五", 22, "东莞", '男', 98);
        Student stu4 = new Student("小宝", 20, "深圳", '男', 88);
        Student stu5 = new Student("刘能", 34, "深圳", '男', 66);
        Student stu6 = new Student("刘大能", 75, "深圳", '男', 85);
        Student stu7 = new Student("刘小能", 60, "东莞", '男', 12);
        Student stu8 = new Student("广坤", 70, "广州", '男', 32);
        Student stu9 = new Student("广坤", 70, "广州", '女', 32);
        Student stu10 = new Student("广坤", 70, "广州", '男', 32);
        Student stu11 = new Student("广坤", 70, "广州", '男', 32);

        list.add(stu1);
        list.add(stu2);
        list.add(stu3);
        list.add(stu4);
        list.add(stu5);
        list.add(stu6);
        list.add(stu7);
        list.add(stu8);
        list.add(stu9);
        list.add(stu10);
        list.add(stu11);
    }

    public void createStream() {
        Stream<Student> stream = list.stream();

        IntStream stream1 = Arrays.stream(new int[]{1, 2, 3, 4, 5});

        Stream<String> a = Stream.of("a", "b", "c");

        Stream.generate(Math::random).limit(5).forEach(System.out::println);


    }

    @Test
    public void testFilter() {
        list.stream().filter(stu -> stu.getSex() == '女').forEach(System.out::println);

        System.out.println("-----------------------------------");

        list.stream().filter(stu -> stu.getSex() == '女').filter(stu -> stu.getAge() >= 25).forEach(System.out::println);

    }

    @Test
    public void testDistinct() {
        list.stream().distinct().forEach(System.out::println);
    }

    @Test
    public void testLimit() {
        list.stream().filter(stu -> stu.getSex() == '男').limit(3).forEach(System.out::println);
    }

    @Test
    public void testSkip() {
        list.stream().filter(stu -> stu.getSex() == '男').skip(3).forEach(System.out::println);
    }

    @Test
    public void testSorted() {
        list.stream().sorted().forEach(System.out::println);
    }

    @Test
    public void testSortedComparator() {
        list.stream().sorted((stu1, stu2) -> (int) (stu1.getScore() - stu2.getScore())).forEach(System.out::println);
    }

    @Test
    public void testMap() {
        Stream<String> a = Stream.of("a", "b", "c");
        a.map(String::toUpperCase).forEach(System.out::println);
    }

    @Test
    public void testFlatMap() {
        Stream.of("abc", "hello", "world").flatMap(TestStream::buildStringAsStream).forEach(System.out::println);
    }

    public static Stream<Character> buildStringAsStream(String str) {
        char [] chars = str.toCharArray();
        List<Character> list = new ArrayList<>();
        for (char ch : chars) {
            list.add(ch);
        }
        return list.stream();
    }

    @Test
    public void testAllMatch() {
        boolean flag = list.stream().allMatch(stu -> stu.getAge() >= 18);
        System.out.println("flag = " + flag);
    }

    @Test
    public void testAnyMatch() {
        boolean flag = list.stream().anyMatch(stu -> stu.getAge() < 18);
        System.out.println("flag = " + flag);
    }

    @Test
    public void testNoneMatch() {
        boolean flag = list.stream().noneMatch(stu -> stu.getScore() < 60);
        System.out.println("flag = " + flag);
    }

    @Test
    public void testFindFirst() {
        Optional<Student> first = list.stream().findFirst();
        System.out.println("first = " + first);
    }

    @Test
    public void testCount() {
        long count = list.stream()
                .filter(stu -> stu.getAddress().equals("深圳"))
                .filter(stu -> stu.getSex() == '男')
                .filter(stu -> stu.getAge() >= 20)

                .count();
        System.out.println("count = " + count);
    }

    @Test
    public void testMax() {
        Optional<Student> max = list.stream().max(Comparator.comparingInt(Student::getAge));
        System.out.println("max = " + max);
    }

    @Test
    public void testMin() {
        Optional<Student> min = list.stream().min(Comparator.comparingInt(Student::getAge));
        System.out.println("min = " + min);

    }
}
```

