# String类特点

#### 初始化方式

1. 直接赋值引号

   ```java
   String str = "Hello World!";
   ```

   这个操作会开辟出一块栈内存空间和一块堆内存空间，此种方式初始化的字符串，会加入字符串常量池，实现字符串对象的共享。

   ```java
   String str1 = "hello";
   String str2 = "hello";
   System.out.println(str1 == str2); // == 判断两边是否同一个对象
   // 输出结果: true，证明两个引用共享了同一个对象
   ```

2. new 关键字

   ```java
   String str = new String("Hello World");
   ```

   将以上过程拆解：

   ```java
   String str;		// 在内存中开辟一块栈空间
   "Hello World";	// 字符串是匿名类对象，所以在堆内存中开辟一块空间
   new String();	// 构造方法在堆内存中开辟一块空间
   ```

   由此可见，使用 String 类构造方法实例化字符串对象时，会在堆内存中开辟两块空间，且其中一块会沦为垃圾空间。

#### 什么是字符串常量池？

在堆内存中提供一个专门的数组，用来存放字符串。假如新建的字符串是数组中存在的，那么直接将引用指向该字符串对象，节省空间。如果数组（池）中没有需要的字符串对象，则新建对象，然后将该对象放入字符串常量池中，以便下次使用。

```java
String str1 = "hello"; // 第一次创建该字符串，放入字符串常量池
String str2 = "hello"; // 已经存在该字符串，所以不用新建，直接引用
```

注：new 关键字方式新建的字符串对象不会放入字符串常量池，所以建议使用引号赋值的方式。可以使用 `intern` 方法让 new 创建的字符串入池。

另：

Java中的常量池还分为 **静态常量池** 和 **运行时常量池** ：

- 静态常量池：程序（*.class）文件中的常量池，包含字符串（数字）字面量，以及类、方法的信息。
- 运行时常量池：在 JVM 虚拟机完成类装载后，将 class 文件中的常量池载入到内存中，并保存在方法区。运行时常量池的一个重要特性就是 **具备动态性**，运行时也可能将新的常量放入池中，比如 String 类的 `intern()` 方法。

常量池的**好处**是可以避免频繁的创建和销毁对象而影响系统的性能，通过实现对象共享达到这个目的。

```java
String s1 = "hello";
String s2 = "hello";
String s3 = "hel" + "lo";
String s4 = "hel" + new String("lo");
String s5 = new String("hello");
String s6 = s5.intern();
String s7 = "h";
String s8 = "ello";
String s9 = s7 + s8;
// 判断
System.out.println(s1 == s2); // true. s1、s2在赋值时，均使用了字符串字面量，直接把字符串写死，在编译期间，这种字面量会被放入 class 文件的常量池中，从而实现复用，s1、s2指向的是同一个内存地址。
System.out.println(s1 == s3); // true. 虽然s3是拼接出来的，但是由于参与拼接的部分都是已知的字面量，在编译期间，编译器会将这种拼接优化，直接帮你拼好，所以 String s3 = "hel" + "lo" 会被优化成 String s3 = "hello";
System.out.println(s1 == s4); // false. s4虽然也是拼接出来的，但是 new String("lo") 不是已知字面量，是一个不可预料的部分，编译器不会优化，必须等到运行时才可以确定结果。
System.out.println(s1 == s9); // false. s7、s8在赋值的时候使用了字符串字面量，但是拼接成s9的时候，s7、s8作为两个变量，都是不可预料的，编译器不会优化。只能等到运行时，在堆中创建s7、s8拼接成的新字符串，在堆中地址不确定，不可能与常量池中的字符串地址相同。。
```

#### String 类常用方法

1. 与字符

   `public String(char[] chars)` 将传入的字符数组变为字符串

   `public String(char[] chars, int offset, int length)` 将传入的字符数组指定长度变为字符串

   `public char charAt()` 返回某个位置的字符

   `public char[] toCharArray()` 将字符串转换为字符数组

2. 与字节（为了二进制数据传输，或者为了编码转换）

   `public String(byte[] bytes)` 将传入的字节数组变为字符串

   `public String(byte[] bytes, int offset, int length)` 将传入的字节数组指定长度变为字符串

   `public byte[] getBytes()` 将字符串转换为字节数组

3. 查找

   `public boolean contains(String s)` 判断子字符串是否存在

   `public int indexOf(String str)` 从头查找指定字符串的位置，返回-1代表不存在该子串

   `public int indexOf(String str, int fromIndex)` 从指定位置查找指定字符串

   `public int lastIndexOf(String str)` 从后向前查找指定字符串的位置

   `public int lastIndexOf(String str, int fromIndex)` 从指定位置从后向前查找指定字符串位置

   `public boolean startsWith(String prefix)` 判断是否以指定的字符串开头

   `public boolean startsWith(String prefix, int toffset)` 

   `public boolean endsWith(String suffix)` 判断是否以指定的字符串结尾

4. 替换

   `public String replaceAll(String regex, String replacement)` 替换全部符合条件的文本

   `public String replaceFirst(String regex, String replacement)` 替换首个符合条件的文本

5. 拆分

   `public String[] split(String regex)` 按照指定的字符串全部拆分

   `public String[] split(String regex, int limit)` 按照指定的字符串拆分为指定个数的数组

6. 截取

   `public String substring(int beginIndex)` 从指定索引截取到结尾

   `public String susstring(int beginIndex, int endIndex)` 截取指定索引范围中的字符串

7. 格式化字符串

   `public static String format(String format, Object args)` 根据指定的结构格式化文本

   ```java
   String name = "张三";
   int age = 10;
   double score = 24.892394;
   String str = String.format("姓名：%s、年龄：%d、成绩：%.2f。", name, age, score);
   System.out.println(str);
   // 输出：姓名：张三、年龄：10、成绩：24.89。
   ```

8. 其他方法

   `public String concat(String str)` 连接字符串

   `public Stirng intern()` 字符串入池

   `public boolean isEmpty()` 判断是否为空字符串（不是null）

   `public int length()` 返回字符串长度

   `public String trim()` 消除左右的空格

   `public String toUpperCase()` 转换为大写

   `public String toLowerCase()` 转换为小写