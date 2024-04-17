# Dubbo 入门

### Dubbo 介绍

Apache Dubbo 是一款 RPC 服务开发框架，用于解决微服务架构下的服务治理与通信问题。使用 Dubbo 开发的微服务原生具备相互之间的远程地址发现与通信能力， 利用 Dubbo 提供的丰富服务治理特性，可以实现诸如服务发现、负载均衡、流量调度等服务治理诉求。Dubbo 被设计为高度可扩展，用户可以方便的实现流量拦截、选址的各种定制逻辑。

### Dubbo 能做什么

- 面向接口代理的高性能 RPC 调用
- 智能容错和负载均衡
- 服务自动注册和发现
- 高度可扩展能力
- 运行期流量调度
- 可视化的服务治理与运维

简单来说，Dubbo 不光可以帮助我们调用远程服务，还提供了一些其他开箱即用的功能比如智能负载均衡。

### 为什么要用 Dubbo

单一应用架构、垂直应用架构无法满足越来越大的网站规模，越来越多的用户数量，这时候就需要用到分布式服务架构。

分布式服务架构下，系统被拆分为不同的服务，如短信服务、安全服务、订单服务，每个服务独立提供系统的某个核心服务。

可以用 Java RMI（Java Remote Method Invocation）、Hessian 这种支持远程调用的框架来简单地暴露和引用远程服务。但是当服务越来越多，服务调用关系越来越复杂。应用访问压力越来越大后，负载均衡以及服务监控的需求也迫在眉睫。

Dubbo 的出现让上述问题得到了解决。

![Dubbo 能力概览](https://oss.javaguide.cn/github/javaguide/system-design/distributed-system/rpc/dubbo-features-overview.jpg)



### 分布式基础

#### 什么是分布式

分布式就是把整个系统拆分成不同的服务，然后将这些服务放在不同的服务器上，减轻单体服务的压力，提高并发量和性能。

比如电商系统：简单地拆分成订单系统、商品系统、登录系统等等。拆分之后的每个服务可以部署在不同的机器上，如果某一个服务的访问量比较大，也可以将这个服务同时部署在多台机器上。

#### 为什么要分布式

单体应用：所有代码集中在一起。

分布式应用：系统的代码根据业务被拆分，每个团队可以负责一个服务的开发，提升开发效率。根据业务拆分之后的代码，更加便于维护和扩展。

拆分后的业务，能提高整个系统的性能。

### Dubbo 架构

#### Dubbo 架构中的核心角色

![dubbo-relation](https://oss.javaguide.cn/%E6%BA%90%E7%A0%81/dubbo/dubbo-relation.jpg)

- Container：服务运行容器，负责加载、运行 Provider，必须。
- Provider：暴露服务的服务提供方，会向注册中心注册自己提供的服务，必须。
- Consumer：调用远程服务的服务消费方，会向注册中心订阅自己所需的服务，必须。
- Registry：服务注册与发现的注册中心，注册中心会返回服务提供者地址列表给消费者，非必须。
- Monitor：统计服务调用次数和调用时间的监控中心，服务消费者和提供者会定时发送统计数据到监控中心。非必须。

#### Invoker

`Invoker` 是 Dubbo 对远程调用的抽象。

![dubbo_rpc_invoke.jpg](https://oss.javaguide.cn/java-guide-blog/dubbo_rpc_invoke.jpg)

分为：

- 服务提供 `Invoker`
- 服务消费 `Invoker`

调用一个远程方法，需要动态代理来屏蔽远程调用的细节，屏蔽掉的这些细节就依赖对应的 `Invoker` 实现。

#### 工作原理

Dubbo 的整体设计从下至上分为10层，各层单向依赖。

- config **配置层**

  Dubbo 相关的配置。支持代码配置，同时也支持基于 Spring 来做配置，以 `ServerConfig`, `ReferenceConfig` 为中心

- proxy **服务代理层**

  调用远程方法像调用本地方法一样简单的一个关键，真实调用过程依赖代理类，以 `ServiceProxy` 为中心

- registry **注册中心层**

  封装服务地址的注册与发现

- cluster **路由层**

  封装多个提供者的路由及负载均衡，并桥接注册中心，以`Invoker` 为中心

- monitor **监控层**

  RPC 调用次数和调用时间监控，以 `Statistic` 为中心

- protocol **远程调用层**

  封装 RPC 调用，以 `Invocation` ，`Result` 为中心

- exchange **信息交换层**

  封装请求响应模式，同步转异步，以 `Request` ，`Response`  为中心

- transport **网络传输层**

  抽象 mina 和 netty 为统一接口，以 `Message` 为中心

- serialize **数据序列化层**

  对需要在网络传输的数据进行序列化

#### 一些问题：

##### 注册中心的作用

注册中心负责服务地址的注册与查找，相当于目录服务，Provider 和 Consumer 只在启动时与注册中心交互。

##### Provider 宕机后，注册中心会做什么

注册中心会立即推送事件通知消费者。

##### 监控中心的作用

监控中心负责统计各服务调用次数，调用时间等。

##### 注册中心和监控中心都宕机的话，服务会都挂掉吗

不会。两者都宕机也不影响已经运行的 Provider 和 Consumer，Consumer 在本地缓存了 Provider 的列表。注册中心和监控中心都是可选的，Consumer 可以直连 Provider。

#### Dubbo 提供的负载均衡策略

##### RandomLoadBalance 

根据权重随机选择（加权随机算法）。这是 Dubbo 默认采用的一种负载均衡策略。

实现原理：

假如有两个提供相同服务的服务器 S1，S2，S1 的权重为 7，S2 的权重为3。

把这些权重值分布在坐标区间就会得到：S1 -> [0, 7)，S2 -> [7, 10) 。我们生成 [0, 10) 之间的随机数，随机数落到对应的区间，我们就选择对应的服务器来处理请求。

源码：

```java
public class RandomLoadBalance extends AbstractLoadBalance {
    public static final String NAME = "random";
    
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        boolean sameWeight = true;
        int[] weights = new int[length];
        int totalWeight = 0;
        // 下面这个 for 循环的主要作用是计算所有 Provider 的权重之和 totalWeight
        // 除此之外，还会检测每个 Provider 的权重是否相同
        for (int i = 0; i < length; i++) {
            int weight = getWeight(invokers.get(i), invocation);
            totalWeight += weight;
            weights[i] = totalWeight;
            if (sameWeight && totalWeight != weight * (i + 1)) {
                sameWeight = false;
            }
        }
        if (totalWeight > 0 && !sameWeight) {
            // 随机生成一个 [0, totalWeight) 区间内的数字
            int offset = ThreadLocalRandom.current().nextInt(totalWeight);
            // 判断落在哪个 Provider 的区间
            for (int i = 0; i < length; i++) {
                if (offset < weight[i]) {
                    return invokers.get(i);
                }
            }
         
      	return invokers.get(ThreadLocalRandom.current().nextInt(length));
        }
    }
}
```

##### LeastActiveLoadBalance

`leastActiveLoadBalance` 直译就是最小活跃数负载均衡。

初始状态下所有 provider 的活跃数均为 0，每收到一个请求，对应的 provider 活跃数 +1，当这个请求处理完之后，活跃数 -1。

因此可以看出，活跃数越大的，provider 正在处理的任务就越多；活跃数越小，这个 provider 就越空闲，性能也越好。

如果有多个 provide 的活跃数相等，则再走一遍 `RandomLoadBalance` 。

源码：

```java
public class LeastActiveLoadBalance extends AbstractLoadBalance {
    public static final String NAME = "leastactive";
    
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, URL url, Invocation invocation) {
        int length = invokers.size();
        int leastActive = -1;
        int leastCount = 0;
        int[] leastIndexes = new int[length];
        int[] weights = new int[length];
        int totalWeight = 0;
        int firstWeight = 0;
        boolean sameWeight = true;
        // 这个 for 循环的主要作用是遍历 invokers 列表，找出活跃数最小的 Invoker
        // 如果有多个 Invoker 具有相同的最小活跃数，还会记录下这些 Invoker 在 invokers 集合中的下标，并累加它们的权重，比较它们的权重是否相等
        for (int i = 0; i < length; i++) {
            Invoker<T> invoker = invokers.get(i);
            int active = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName()).getActive();
            int afterWarmup = getWeight(invoker, invocation);
            weights[i] = afterWarmup;
            if (leastActive == -1 || active < leastActive) {
                leastActive = active;
                leastCount = 1;
                leastIndexes[0] = i;
                totalWeight = afterWarmup;
                firstWeight = afterWarmup;
                sameWeight = true;
            } else if (active == leastActive) {
                leastIndexes[leastCount++] = i;
                totalWeight += afterWarmup;
                if (sameWeight && afterWarmup != firstWeight) {
                    sameWeight = false;
                }
            }
        }
        // 如果只有一个 Invoker 具有最小的活跃数，此时直接返回该 Invoker 即可
        if (leastCount == 1) {
            return invokers.get(leastIndexes[0]);
        }
        // 如果有多个 Invoker 具有相同的最小活跃数，但它们之间的权重不同
        // 这里的处理方式和 RandomLoadBalance 一致了
        if (!sameWeight && totalWeight > 0) {
            int offsetWeight = ThreadLocalRandom.current().nextInt(totalWeight);
            for (int i = 0; i < leastCount; i++) {
                int leastIndex = leastIndexes[i];
                offsetWeight -= weights[leastIndex];
                if (offsetWeight < 0) {
                    return invokers.get(leasetIndex);
                }
            }
        }
        return invokers.get(leastIndexes[ThreadLocalRandom.current().nextInt(leastCount)]);
    }
}
```

活跃数是通过 `RpcStatus` 中的一个 `ConcurrentMap` 保存的，根据 URL 以及 provider 被调用的方法的名称，我们可以获取到对应的活跃数。也就是说 provider 中的每一个方法的活跃数都是互相独立的。

```java
public class RpcStatus {
    private static final ConcurrentMap<String, ConscurrentMap<String, RpcStatus>> METHOD_STATISTICS = new ConcurrentHashMap<String, ConcurrentMap<String, RpcStatus>>();
    public static RpcStatus getStatus(URL url, String methodName) {
        String uri = url.toIdentityString();
        ConcurrentMap<String, RpcStatus> map = METHOD_STATISTICS.computeIfAbsent(uri, k -> new ConcurrentHashMap<>());
        return map.computeIfAbsent(methodName, k -> new RpcStatus());
    }
    public int getActive() {
        return active.get();
    }
}
```

##### ConsistentHashLoadBalance

`ConsistentHashLoadBalance` 是**一致性 Hash 负载均衡策略**，在分库分表、各种集群中经常使用。`ConsistentHashLoadBalance` 没有权重的概念，具体由哪个 provider 处理请求，是请求的参数决定的，也就是说相同的请求参数总是会发到同一个 provider。

为了避免数据倾斜问题（大量请求落到同一节点），引入了虚拟节点的概念。通过虚拟节点让节点更加分散，有效均衡各个节点的请求量。

##### RoundRobinLoadBalance

**加权轮询负载均衡**。轮询就是把请求依次分配到每个 provider，加权轮询就是在轮询的基础上，让更多的请求落到权重更高的 provider 上。假如有两个服务器，S1权重为7，S2权重为3，那么10次请求中，就会有7次落到S1，3次落到S2。

对比 `RandomLoadBalance` 很可能存在10次请求有9次都被 S1 处理的情况。

#### 序列化

Dubbo 支持多种序列化方式：JDK自带、hessian2、JSON、Kryo、FST、Protostuff、ProtoBuf 等。

默认使用的序列化方式是 hessian2。

一般不直接使用 JDK 自带的序列化方式，不支持跨语言调用，性能较差。

JSON 序列化由于性能问题，一般也不使用。

Protostuff、Protobuf、hessian2 都是跨语言的序列化方式，可以使用。

Kryo 和 FST 性能非常好，不过都是针对 Java 语言的。



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

