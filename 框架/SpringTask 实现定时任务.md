### SpringTask 实现定时任务

#### 技术栈

- SpringTask

  | SpringTask 是Spring 自主研发的i轻量级定时任务工具，相比于 Quartz 更加简单方便，且不需要引入其他依赖即可使用。

- Cron 表达式

  | Cron 表达式是一个字符串，包括6~7个时间元素，在 SpringTask 中可以用于指定任务的执行时间。

#### Cron 介绍

##### Cron 的语法格式

```
Seconds Minutes Hours DayofMonth Month DayofWeek
```

##### Cron 格式中每个时间元素的说明

| 时间元素   | 可出现的字符  | 有效数值范围   |
| ---------- | ------------- | -------------- |
| Seconds    | , - * /       | 0-59           |
| Minutes    | , - * /       | 0-59           |
| Hours      | , - * /       | 0-23           |
| DayofMonth | , - * / ? L W | 0-31           |
| Month      | , - * /       | 1-12           |
| DayofWeek  | , - * / ? L # | 1-7 或 SUN-SAT |

##### Cron 格式中特殊字符说明

| 字符 | 作用                                          | 举例                                                         |
| ---- | --------------------------------------------- | ------------------------------------------------------------ |
| ,    | 列出枚举值                                    | 在 Minutes 域使用 5,10，表示在5分和10分各触发一次            |
| -    | 表示触发范围                                  | 在 Minutes 域使用 5-10，表示从5分到10分钟每分钟触发一次      |
| *    | 匹配任意值                                    | 在 Minutes 域中使用 *，表示每分钟都会触发一次                |
| /    | 起始时间开始触发，每隔固定时间触发一次        | 在 MInutes 域中使用 5/10，表示5分时触发一次，每10分钟再触发一次 |
| ?    | 在 DayofMonth 和 DayofWeek 中，用于匹配任意值 | 在 DayofMonth 域使 ?，表示每天都触发一次                     |
| #    | 在 DayofMonth 中，确定第几个星期几            | 1#3 表示第三个星期天                                         |
| L    | 表示最后                                      | 在 DayofWeek 中使用 5L，表示在最后一个星期四触发             |
| W    | 表示有效工作日（周一到周五）                  | 在 DayofMonth 中使用 5W，如果5号是星期六，则将在最近的工作日 4号触发一次 |



#### 在 Spring 中实现定时任务

| 开启一个定时任务，每隔5分钟检查，超时未付款的订单则取消

无需添加依赖， SpringTask 已经存在 Spring 框架中。

##### 添加配置，开启定时任务

```java
@Configuration
@EnableScheduling
public class SpringTaskConfig {
}
```

##### 通过 @Scheduled 注解设置定时任务

```java
@Component
public class OrderTimeOutCancelTask {
    private Logger LOGGER = LoggerFactory.getLogger(OrderTimeOutCancelTask.class);
    
    @Scheduled(cron = "0 0/10 * ? * ?")
    private void cancelTimeOutOrder() {
        // TODO: 调用取消订单的方法
        LOGGER.info("取消订单，并根据 SKU 编号释放锁定库存");
    }
}
```

