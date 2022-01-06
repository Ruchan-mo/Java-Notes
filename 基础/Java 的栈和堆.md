# Java 的栈和堆

#### 栈

栈用来存放调用的方法，以及局部变量

![栈 (1)](D:\Download\栈 (1).jpg)

当你调用一个方法时，该方法的堆栈块会被放在调用栈的栈顶，它带有**方法的状态**，包括执行到哪一行程序以及所有的局部变量的值。

栈顶上的方法是目前正在执行的方法。方法会一直待在这里直到执行完毕。

现在有这样三个方法：

```java
public void doStuff() {
    boolean b = true;
    go(4);
}

public void go(int x) {
    int z = x + 24;
    crazy();
}

public void crazy() {
    char c = 'a';
}
```

第一个方法在执行过程中会调用第二个方法，第二个会调用第三个。每个方法都在内容中声明一个局部变量（go() 方法还声明了一个参数）

![堆栈](D:\Download\堆栈.jpg)

1. 某段代码调用了方法 `doStuff()` ，于是它被放在栈顶
2.  `doStuff()` 调用 `go()` ，`go()` 被放在栈顶
3. `go()` 又调用 `crazy()` ，`crazy()` 被放在栈顶
4. `crazy()` 执行完成后，它的堆栈块就被释放掉了。执行回到了 `go()` 



**要点：**

· 实例变量是声明在类中，方法之外的地方。

```java
public class User {
    private String name; // 实例变量
    private int age; // 实例变量
}
```

· 局部变量声明在方法或方法的参数上。

```java
pblic class User {
    
    public void checkUser(int age) { // 局部变量
        int boolean = true; // 局部变量
        if (age < 0) {
            System.out.println("Not a valid age.");
        }
    }
}
```

· 所有局部变量都存在于栈上相对应的堆栈块中。

· 对象引用变量与 primitive 主数据类型变量都是放在栈上

```java
User user;	// 在栈内存里面开辟了空间给引用变量user，这个时候user=null
user = new User();
/**
* 1. new User() 在堆内存中开辟了空间给 User 类的对象，这个对象还没有名字
* 2. User() 随即调用了 User 类的构造函数
* 3. "=" 符号把对象的地址在堆内存的地址给引用变量 user
*/
```

· 不管是实例变量或局部变量，对象本身，都是在堆上的。



#### 堆

所有对象都存活在可垃圾回收的堆上。



![未命名文件](D:\Download\未命名文件.png)

实例变量生存在栈还是堆上？

当我们新建一个 `User()` 时，Java 会在堆上开辟一个空间给 User 对象，开辟空间的大小，是这个 User 对象所有实例变量的空间。

也就是说对于一个如下的 User 类：

```java
public class User {
    private int age;
    private long id;
}
```

Java 会在堆中根据两个 primitive 主数据类型大小分配空间，int 需要32位，long 需要64位。

如果 User 里有一个实例变量是对象呢？如：

```java
public class User {
    private Student student;
}
```

Java开辟空间时，是否还要分配新建 Student 类需要的空间？并不是。Java 只会给 Student 对象的引用变量分配空间。

那 Student 对象会在堆上取得自己的空间吗？这要看 Student 对象是在何时创建的，如果是：

 ```java
private Student student = new Student();
 ```

则会在堆上为 Student 开辟空间。

![实例变量在堆上](D:\Download\实例变量在堆上.jpg)