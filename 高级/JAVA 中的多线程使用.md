## JAVA 中的多线程使用

#### 多线程的创建

- 第一种：继承 Thread 类，重写 `run()` 方法，**调用 `start()` 方法**

  ```java
  class MyThread extends Thread {
      private String title;
      public MyThread(String title) {
          this.title = title;
      }
      @Override
      public void run() {
          for (int i = 0; i < 10; i++) {
              System.out.println("线程" + title + "执行第" + i + "次");
          }
      }
  }
  
  public class ThreadDemo {
      public static void main(String[] args) {
          MyThread t1 = new MyTread("线程A");
          MyThread t2 = new MyTread("线程B");
          t1.start();
          t2.start();
      }
  }
  ```

  ps：线程的启动，必须使用 Thread 类的`start()` 方法，其实现如下：

  ```java
  public synchronized void start() {
  
          if (threadStatus != 0)
              throw new IllegalThreadStateException();
      
          group.add(this);
  
          boolean started = false;
          try {
              start0(); // 这是一个 native 方法，根据操作系统的不同，会执行不同的操作
              started = true; // 启动成功后，started 设置为 true
          } finally {
              try {
                  if (!started) {
                      group.threadStartFailed(this);
                  }
              } catch (Throwable ignore) {
                  /* do nothing. If start0 threw a Throwable then
                    it will be passed up the call stack */
              }
          }
      }
  ```

- 第二种：实现 Runnable 接口，重写 `run()` 方法，将类作为参数传入 Thread 构造方法中，调用 `start()` 方法

  ```java
  class MyThread implements Runnable {
      private String title;
  
      public MyThread(String title) {
          this.title = title;
      }
      /* 这一步与继承 Thread 类相同，
      	都是重写 run() 方法 */
      @Override
      public void run() {
          for (int x = 0; x < 10; x++) {
              System.out.println(this.title + "运行，x = " + x);
          }
      }
  }
  
  public class ThreadDemo {
      public static void main(String[] args) {
         // 在实例化 Thread 对象的时候传入实现了 Runnable 接口的类对象
         Thread t1 = new Thread(new MyThread("线程A"));
         Thread t2 = new Thread(new MyThread("线程B"));
         t1.start();
         t2.start();
  
      }
  }
  ```

  实现 Runnable 接口的好处是，自定义类不会收到单继承特性的限制。

  注意到由于 Runnable 是一个函数式接口，那么我们可以利用 lambda 表达式简化操作。

  ```java
  public class ThreadDemo {
      public static void main(String[] args) {
          new Thread(() -> {
              for (int i = 0; i < 10; i++) {
                  System.out.println("线程A，第" + i + "次运行。");
              }
          }).start();
          new Thread(() -> {
              for (int i = 0; i < 10; i++) {
                  System.out.println("线程B，第" + i + "次运行。");
              }
          }).start();
      }
  }
  ```

  **Thread 类和 Runnable 接口的关系：**

  - Thread 类也实现了 Runnable 接口，所以实际上，我们自定义的 MyThread 类和 Thread类之间是一种代理和主体的关系。MyThread 类负责项目核心功能，线程相关资源的调度都交给 Thread 类来处理。

    ![image-20210930192019771](C:\Users\lonel\Desktop\Java学习笔记\Pics\多线程Thread和Runnable.png)

  - 在 Thread 中 `run()` 方法的实现如下：

    ```java
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
    ```

    这里的 target 类型是 Runnable，即代理类和主体类共同实现的接口。Thread 类执行 `start()` 方法的时候，会执行自身的 `run()` 方法，而这个 `run()` 方法所做的就是，执行实现了 Runnable 的子类重写过的 `run()` 方法。

  - 多线程开发中，Thread 描述线程，Runnable 描述资源。

    ![image-20210930191657644](C:\Users\lonel\Desktop\Java学习笔记\Pics\多线程开发.png)

  在开发中多线程的实现最好方式是 Runnable 接口，但是这种方法有一个缺陷，即无法获取一个返回值。我们可以使用 Callable 接口来替代。

- 第三种：实现 Callable 接口。

  ```java
  import java.util.concurrent.Callable;
  import java.util.concurrent.ExecutionException;
  import java.util.concurrent.FutureTask;
  
  class MyThread implements Callable<String> {
      
      @Override
      public String call() throws Exception {
          for (int i = 0; i < 10; i++) {
              System.out.println("线程执行");
          }
          return "运行完毕";
      }
  }
  
  public class ThreadDemo {
      public static void main(Stirng[] args) {
          FutureTask<String> task = new FutureTask<>(new MyThread()); // 传入我们自定义的线程类
          new Thread(task).start(); // 匿名类启动线程
          System.out.println(task.get());
          
      }
  }
  ```

  

#### 多线程的状态

调用 `start()` 方法后，进入**就绪**状态 >> 调度，进入**运行**状态(`run()` 方法) >>  让出资源，进入**阻塞**状态 >> 重新进入**就绪**状态。

线程的 `run()` 方法执行完毕后，该线程的任务也就完成了，此时线程进入停止状态。

#### 卖票问题引出线程同步问题

假如有这样一个程序，三个票贩子（线程）一起卖票（资源），加上延迟后，产生了同步的问题。

```java
public class ThreadDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread mt = new MyThread();
        Thread t1 = new Thread(mt, "票贩子A");
        Thread t2 = new Thread(mt, "票贩子B");
        Thread t3 = new Thread(mt, "票贩子C");
        t1.start();
        t2.start();
        t3.start();
    }
}

class MyThread implements Runnable {
    private int ticket = 10;
    @Override
    public void run() {
        while (true) {
            if (ticket > 0) {
                try {
                    Thread.sleep(100); // 在这里延迟后，对于 ticket 数量的判断就不再是实时的了，多个线程在争夺同一资源时就会出现同步问题
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "卖票" + ", ticket = " + ticket--);
            } else {
                System.out.println("票已经卖完了");
                break;
            }
        }
    }
}
```

解决同步问题的关键就是：**锁**。

既然多个线程争夺同一资源会出现同步问题，那么我们对该资源加上锁，限制同一时间内，只能有一个线程访问该资源。

- 同步代码块。对资源相关代码块加锁。锁对象可以使用 this

  ```java
  synchronized(this) {
      while (true) {
              if (ticket > 0) {
                  try {
                      Thread.sleep(100); // 在这里延迟后，对于 ticket 数量的判断就不再是实时的了，多个线程在争夺同一资源时就会出现同步问题
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
                  System.out.println(Thread.currentThread().getName() + "卖票" + ", ticket = " + ticket--);
              } else {
                  System.out.println("票已经卖完了");
                  break;
              }
          }
  }
  ```

- 同步方法。将资源相关代码封装成方法。

  ```java
  private synchronized void sale() {
      while (true) {
          if (ticket > 0) {
              try {
                  Thread.sleep(100); // 在这里延迟后，对于 ticket 数量的判断就不再是实时的了，多个线程在争夺同一资源时就会出现同步问题
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
              System.out.println(Thread.currentThread().getName() + "卖票" + ", ticket = " + ticket--);
          } else {
              System.out.println("票已经卖完了");
              break;
          }
      }
  }
  ```


#### 线程间的通信

synchronized 可以解决多个线程对同一个资源的争夺问题，而线程的调度，则要依赖于线程间的通信。

假设有这样一个场景：Producer（生产者）生产 Message（消息），消费者（Consumer）消费消息。在生产者生产出消息之前，消费者是不可以消费消息的。同理，在消费者消费完消息之前，生产者也不可以生产消息。因此，多个线程间就需要通过通信，来确定另一个线程的状态，以便知道自己是否可以继续下一步操作。

在下面的例子中，`wait()` 方法让线程进入等待状态，`notify()`方法唤醒第一个进入等待的线程。

```java
public class ThreadDemo {
    public static void main(String[] args) {
        Message msg = new Message();
        new Thread(new Producer(msg)).start();
        new Thread(new Consumer(msg)).start();
    }
}

class Message {
    private String title;
    private String content;
    private boolean flag = true; // true: 生产  false: 消费
    public synchronized void set(String title, String content) {
        if (!flag) {
            try {
                super.wait(); // 无法生产，进入等待状态
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        } 
        this.title = title;
        this.content = content;
        this.flag = false; // 生产完，设置成可消费状态
        super.notify(); // 唤醒刚刚因为无法消费而等待的线程
    }
    public synchronized String get() {
        if (flag) {
            try {
                super.wait(); // 无法消费，进入等待状态
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        try {
            return this.title + " - " + this.content;
        } finally {
            // 无论如何都会执行
            this.flag = true;
            super.notify();
        }
    }
}

class Producer implements Runnable {
    private Message msg;
    public Producer(Message msg) {
        this.msg = msg;
    }
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            if (i % 2 == 0) {
                this.msg.set("张三", "大美女");
            } else {
                this.msg.set("李四", "大帅哥");
            }
        }
    }
}

class Consumer implements Runnable {
    private Message msg;
    public Consumer(Message msg) {
        this.msg = msg;
    }
    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(this.msg.get());
        }
    }
}
```