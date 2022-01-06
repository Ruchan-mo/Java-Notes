
# Java 实现动态代理的两种方式
### 静态代理

先看静态代理的场景：

租房的时候，委托房东。房东除了做租房这件事，还会帮你找房（租房前）、签合同（租房后）。

```java
interface UserDao {
	void save();
}

class UserDaoImpl implements UserDao {
	@Override
	public void save() {
		System.out.println("正在保存用户...");
	}
}

class TransactionHandler implements UserDao {
	private UserDao userDao;
	
	public TransactionHandler(UserDao userDao) {
		this.userDao = userDao;
	}
	
	@Override
	public void save() {
		System.out.println("开启事务控制...");
		userDao.save();
		System.out.println("关闭事务控制...");
	}
}

public class Main {
	public static void main(String[] args) {
		UserDaoImpl target = new UserDaoImpl();
		UserDao userDao = new TransactionHandler(target);
		userDao.save();
	}
}
```

在上面的例子中，没有直接使用`UserDaoImpl` 的对象调用 `save()` 方法，而是将其传入代理类 `TransactionHanlder` ，在代理类中调用 `save()`。这样，除了执行主要业务，还可以在业务前后插入其他事务的操作。

### 动态代理

#### JDK 的 Proxy 类实现动态代理

使用 JAVA 的 Proxy类，通过源码可以看到实现过程：

```java
InvocationHandler handler = new MyInvocationHandler(...);
Class<?> proxyClass = Proxy.getProxyClass(Foo.class.getClassLoader(), Foo.class);
Foo f = (Foo) proxyClass.getConstructor(InvocationHandler.class).newInstance(handler);
```

**步骤如下**：

1. 创建 `InvocationHanlder` 实现代理逻辑
2. 根据被代理类的类加载器和接口创建获取代理类
3. 通过包含 `InvocationHandler` 构造函数创建代理类实例

Proxy类将这三步进行了包装，`newProxyInstance()` 方法就是做这件事情的：

```java
Foo f = (Foo) Proxy.newProxyInstance(Foo.class.getClassLoader(),
                                    	new Class<?>[] { Foo.class },
                                    	handler);
```

`InvocationHandler` 类的注解

```java
/**
 * Processes a method invocation on a proxy instance and returns
 * the result.  This method will be invoked on an invocation handler
 * when a method is invoked on a proxy instance that it is
 * associated with.
 */ 
//执行一个委托给代理实例的方法并返回结果。当绑定的proxy实例调用方法时，将会调用InvocationHandler的这个方法
public Object invoke(Object proxy, Method method, Object[] args)
    throws Throwable;
```

**示例**

首先创建一个接口 Bird

```java
public interface Bird {
    void fly();
}
```

其次建立一个实现类 Pigeon，我们实现一个代理，这个代理是对任意 Bird 的子类，都能增加”唱歌”的功能：

```java
public class Pigeon implements Bird {
    @Override
    public void fly() {
        System.out.println("信鸽飞行");
    }
    
    public static void main(String[] args) {
        Bird bird = new Pigeon();
        Bird proxy = (Bird) Proxy.newProxyInstance(bird.getClass().getClassLoader(), 
                                                   bird.getClass().getInterfaces(), 
                                                   new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println(proxy.getClass().getName());
                System.out.println("唱歌"); // 增加的功能
                method.invode(bird, args);
                return null;00
            }
        });
        System.out.println("------");
        proxy.fly();
        System.out.println("------");
    }
}
```

结果如下：

```java
------
com.sun.proxy.$Proxy0
唱歌
信鸽飞行
------
```

**总结**

JDK 的 Proxy 类近乎完美地实现了动态代理，唯一限制是，被代理类需要实现某个接口，因为生成代理类时，需要通过反射，根据接口的方法来生成代理类中的方法。

#### CGLIB 实现动态代理

对于没有定义接口的业务方法的类，可以使用 CGlib 进行动态代理。CGlib 采用底层的字节码技术，可以为一个类创建子类，在子类中采用方法拦截的技术拦截所有父类方法的调用，并织入横切逻辑。

**代理过程**

```java
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("横切逻辑开始执行");
        Object obj = methodProxy.invokeSuper(o, objects);
        System.out.println("横切逻辑执行结束");
        return obj;
    } 
}
```

CGlib 定义的`intercept()` 接口方法，拦截所有目标类方法的调用，其中 o 代表目标类的实例，method 为目标类方法的反射，args为方法的动态入参，proxy为代理类实例

```java
@Test
public void test() {
    // 使用 Enhancer 创建代理类
    Enhancer enhancer = new Enhancer();
    // 继承被代理类
    enhancer.setSuperClass(UserDemo.class);
    enhancer.setCallback(new CglibProxy()); // 设置回调
    // 生成代理对象
    UserDemo ud = (UserDemo) enhancer.create();
    
    ud.show(); // 在调用代理类方法时，会被方法拦截器拦截
}
```

**注意**

如果委托类被 final 修饰，那么它不可被代理，如果委托类中存在被 final 或者 private 修饰的方法，那么该方法不可被代理。

