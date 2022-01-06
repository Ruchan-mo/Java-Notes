### Java 基础类库

- ##### StringBuffer

  ```java
  public class StringBufferDemo {
  
      public static void main(String[] args) {
          String str = "hello";
          change(str);
          System.out.println(str);
      }
  
      private static void change(String tmp) {
          tmp += " world!";
      }
  }
  // 输出结果还是 hello
  ```

  在上面的例子中，change方法接收 "hello" 的引用地址，随后更改字符串内容时，由于字符串不可更改，Java 将 tmp 的引用指向了新的地址 "hello world!" 中，并且由于没有返回，所以 str 的地址也没有改变，所以最终输出的结果不变。

  StringBuffer 可以解决这个问题。它有如下几个方法：

  1. public StringBuffer()
  2. public StringBuffer(String str)
  3. public StringBuffer append(E e)
  4. public StringBuffer insert(int index, String str)
  5. public StringBuffer delete(int start, int end)
  6. public StringBuffer reverse()

  与 StringBuffer 类似的还有一个 StringBuilder，区别在于 StringBuffer 是线程安全的类。

- ##### CharSequence 接口

  String, StringBuilder, StringBuffer 都实现了 CharSequence 接口。该接口有几个方法：

  1. public char charAt(int index)
  2. public int length()
  3. public CharSequence subSequence(int start, int end)

  CharSequence 描述的就是一个字符串

- ##### AutoCloseable 接口

  用于资源开发的处理上，实现资源的自动关闭（释放资源），比如数据库的相关开发。

  ```java
  public class CloseDemo {
      public static void main(String[] args) {
          Message message = new Message();
          message.send();
          message.close(); // 需要手动关闭
      }
  }
  
  interface IMessage {
      void send();
  }
  
  class Message implements IMessage {
      @Override
      public void send() {
          if (this.open()) {
              System.out.println("发送消息");
          }
      }
      private boolean open() {
          System.out.println("打开连接通道");
          return true;
      }
      public void close() {
          System.out.println("关闭连接通道");
      }
  }
  ```

  既然每次都需要手动调用`close()`，那么能不能实现一个接口，让其自动执行呢？只要实现了 AutoCloseable 接口，并且在 try-with-resource 中进行相应的操作，就可以实现自动关闭的效果。

  ```java
  public class CloseDemo {
      public static void main(String[] args) {
          try(Message message = new Message()) {
          	message.send();
          } catch (Exception e) {
              
          }
          // message.close(); // close() 方法没有调用也会自动执行
      }
  }
  
  interface IMessage extends AutoCloseable {
      void send();
  }
  
  class Message implements IMessage {
      @Override
      public void send() {
          if (this.open()) {
              System.out.println("发送消息");
          }
      }
      private boolean open() {
          System.out.println("打开连接通道");
          return true;
      }
      public void close() {
          System.out.println("关闭连接通道");
      }
  }
  ```

- ##### Runtime 类

  几个重要方法：

  1. public native int availableProcessors()
  2. public native long freeMemory()
  3. public native long maxMemory()
  4. public native long totalMemory()
  5. public native void gc()      // 垃圾回收
  
- ##### 国际化实现

  1. ResourceBundle 类读取文件
  2. Locale 类切换地区
  3. MessageFormat 格式化内容

  ```java
  // message_zh_CN.properties
  info=欢迎用户{0},日期：{1}
  ```

  ```java
  // message_en_US.properties
  info=info=Welcome! {0}. date:{1}
  ```

  ```java
  // JavaAPIDemo.java
  public class JavaAPIDemo {
      public static void main(String[] args) {
          Locale loc = new Locale("zh", "CN");
          ResourceBundle message = ResourceBundle.getBundle("messsage", loc); // 传入资源文件名和地区
          String val = message.getString("info"); // 通过键取值
          String result = MessageFormat.format(val, "mldn", new SimpleDateFormat("yyyy-MM-dd").format(new Date()));
          System.out.println(result);
      }
  }
  ```

- ##### 定时任务

  1. TimerTask 类实现业务
  2. Timer 设置任务执行细节

  ```java
  public class JavaAPIDemo {
      public static void main(String[] args) {
          // 匿名类创建任务
          TimerTask tt = new TimerTask(){
              @Override
              public void run() {
                  System.out.println(Thread.currentThread().getName() + new SimpleDateFormat("yyyy-MM-dd HH:mm:Ss").format(new Date()));
              }
          };
          // Timer 设置执行计划
          Timer timer = new Timer();
          timer.scheduleAtFixedRate(tt, 2000, 1000); // 延迟两秒后执行，间隔一秒执行一次
      }
  }
  ```

  
