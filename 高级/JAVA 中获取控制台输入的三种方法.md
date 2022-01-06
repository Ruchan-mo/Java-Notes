### JAVA 中获取控制台输入的三种方法（以及输出）

1. 单个字符的获取，使用 `InputStream` 

   ```java
   public class JavaAPI {
       public static void main(String[] args) {
           InputStream input = System.in;
           char c = (char) in.read();
           System.out.println(c);
       }
   }
   ```

2. 多个字符的获取，使用 `BufferedReader` ，这种方法可以获取一整行字符串

   ```java
   public class JavaAPI {
       public static void main(String[] args) {
           BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
           String line = reader.readLine();
           System.out.println(line);
       }
   }
   ```

3. 多个字符的获取，使用 `Scanner，`可以获取各种数据类型

   ```java
   public class JavaAPI {
       public static void main(String[] args) {
           Scanner scanner = new Scanner(System.in);
           int num = scanner.nextInt(); // 获取整数
           long l = scanner.nextLong(); // 获取long
           double d = scanner.nextDouble(); // 获取double
           float f = scanner.nextFloat(); // 获取float
           byte b = scanner.nextByte(); // 获取byte
           String str = scanner.next(); // 获取字符串
       }
   }
   ```


PS. 对于输出到控制台我们已经了解，可以使用 `System.out.println()` 方法。如果需要把内容输出到其他地方，优先考虑`PrintWriter`类。它的构造方法：

`public PrintWriter(OutputStream output)` 接收一个 `OutputStream` 对象

`public PrintWriter(Writer writer)` 接收一个 `Writer` 对象

我们可以根据需要，传入具体的对象来使用它。比如，输出到磁盘上的文件：

```java
public class JavaDemo {
    public static void main(String[] args) {
        PrintWriter pw = new PrintWriter(new FileOutputStream(new File("D:" + File.separator + "test.txt")));
        pw.write("hello world\r\n");
        pw.flush();
        pw.close();
    }
}
```

我们也可以将内容输出到控制台，只需要在实例化的时候传入 `System.out`：

```java
public class JavaDemo {
    public static void main(String[] args) {
        PrintWriter pw = new PrintWriter(System.out);
        pw.println("hello world");
        pw.flush();
        pw.close();
    }
}
```

**实例**：利用`Scanner` 以及 `PrintWriter`  实现文本内容拷贝：

```java
public class JavaDemo {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(new File("D:" + File.separator + "test.txt")); // 建立输入流
        PrintWriter pw = new PrintWriter(new File("D:" + File.separator + "test_copy.txt")); // 建立输出流
        scanner.useDelimiter("\n"); // 设置回车为换行符，作为next的判断标准
        while (scanner.hasNext()) {
            pw.print(scanner.next());
            pw.flush();
        }
        pw.close();
        scanner.close();w
    }
}
```

