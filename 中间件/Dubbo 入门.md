# Dubbo 入门

### Dubbo 介绍

Apache Dubbo 是一款 RPC 服务开发框架，用于解决微服务架构下的服务治理与通信问题。使用 Dubbo 开发的微服务原生具备相互之间的远程地址发现与通信能力， 利用 Dubbo 提供的丰富服务治理特性，可以实现诸如服务发现、负载均衡、流量调度等服务治理诉求。Dubbo 被设计为高度可扩展，用户可以方便的实现流量拦截、选址的各种定制逻辑。

### Dubbo 能做什么

- 微服务编程范式和工具
- 高性能的 RPC 通信
- 微服务监控与治理
- 部署在多种环境

### 快速上手

Dubbo 作为一款微服务框架，最重要的是向用户提供跨进程的 RPC 远程调用能力。

Dubbo 的服务消费者(Consumer) 通过一系列的工作将请求发送给服务提供者(Provider)。

为了实现这个目标，Dubbo 引入了注册中心（Registry）组件，通过注册中心，Consumer 可以感知到 Provider 的连接方式，从而将请求发送给正确的 Provider。

在 Dubbo 的实现里面，一共有三个重要的角色：

1. 定义双方通信的 Api 接口
2. 提供服务的 Provider，实现了上述的 Api 接口
3. 使用服务的 Consumer，调用了上述的 Api 接口

### 代码实现

`dubbo.demo.api.GreetingService`

```java
public interface GreetingService {
    String sayHi(String name);
}
```

`dubbo.demo.provider.GreetingServiceImpl`

```java
public class GreetingServiceImpl implements GreetingService {
    @Override
    public String sayHi(String name) {
        return "Hi, " + name;
    }
}
```

`dubbo.demo.provider.Application`

```java
public class Application {
    public static void main(String[] args) {
        // 定义具体的服务
        ServiceConfig<GreetingService> service = new ServiceConfig<>();
        service.setInterface(GreetingService.class);
        service.setRef(new GreetingServiceImpl());

        // 定义 dubbo
        DubboBootstrap.getInstance()
                .application("first-dubbo-provider")
                .registry(new RegistryConfig("zookeeper://localhost:2181"))
                .protocol(new ProtocolConfig("dubbo", -1))
                .service(service)
                .start()
                .await();
    }
}
```

`dubbo.demo.consumer.Application`

```java
public class Application {

    public static void main(String[] args) throws IOException {
        // 定义服务的代理
        ReferenceConfig<GreetingService> reference = new ReferenceConfig<>();
        reference.setInterface(GreetingService.class);

        DubboBootstrap.getInstance()
                .application("first-dubbo-consumer")
                .registry(new RegistryConfig("zookeeper://localhost:2181"))
                .reference(reference);

        // 获取服务、使用服务
        GreetingService service = reference.get();
        String message = service.sayHi("dubbo");
        System.out.println("Receive result ======> " + message);
        int n = service.calc(10, 20);
        System.out.println("Calculation result ======> " + n);
        System.in.read();
    }
}
```

另外还需要一个注册中心 zookeeper：

```java
public class EmbeddedZooKeeper {

    public static void main(String[] args) throws Exception {
        int port = 2181;
        if (args.length == 1) {
            port = Integer.parseInt(args[0]);
        }

        Properties properties = new Properties();
        File file = new File(System.getProperty("java.io.tmpdir")
                + File.separator + UUID.randomUUID());
        file.deleteOnExit();
        properties.setProperty("dataDir", file.getAbsolutePath());
        properties.setProperty("clientPort", String.valueOf(port));

        QuorumPeerConfig quorumPeerConfig = new QuorumPeerConfig();
        quorumPeerConfig.parseProperties(properties);

        ZooKeeperServerMain zkServer = new ZooKeeperServerMain();
        ServerConfig configuration = new ServerConfig();
        configuration.readFrom(quorumPeerConfig);

        System.setProperty("zookeeper.admin.enableServer", "false");

        try {
            zkServer.runFromConfig(configuration);
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
}
```

启动注册中心后，启动 provider，provider 即处于等待被消费的状态。启动 consumer，可以看到返回了消息。



### 使用 SpringBoot 形式

#### Api

api 接口定义不需要改变。

#### Provider

provider 的 api 实现加上 `@DubboService` 注解，注册为 Dubbo 服务：

```java
@DubboService
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String name) {
        return "Hello " + name;
    }
}

```

provider 应用使用注解的形式启动：

```java
@SpringBootApplication
@EnableDubbo
public class ProviderApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }
}
```

具体的配置都放到配置文件 `application.yml`

```yml
dubbo:
  application:
    name: dubbo-springboot-demo-provider
  protocol:
    name: dubbo
    port: -1
  registry:
    address: zookeeper://${zookeeper.address:127.0.0.1}:2181
```

#### Consumer

consumer 同样使用注解：

```java
@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

配置文件 `application.yml`

```yml
dubbo:
  application:
    name: dubbo-springboot-demo-consumer
  protocol:
    name: dubbo
    port: -1
  registry:
    address: zookeeper://${zookeeper.address:127.0.0.1}:2181
```

