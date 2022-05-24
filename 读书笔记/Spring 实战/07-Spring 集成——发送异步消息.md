### 07-Spring 集成——发送异步消息

原生的发送异步消息方式（activemq为例）：

- 创建连接
- 创建session
- 创建目的地
- 创建生产者 / 创建消费者
- 发送消息 / 设置消息监听器
- 提交（生产者）

```java
// Producer
public class JMSProducer {
    private static final String USERNAME = ActiveMQConnection.DEFAULT_USER;

    private static final String PASSWORD = ActiveMQConnection.DEFAULT_PASSWORD;

    private static final String BROKERURL = ActiveMQConnection.DEFAULT_BROKER_URL;

    private static final int SENDNUM = 10;
    
    public static void main(String[] args) {

        // 连接工厂
        ConnectionFactory connectionFactory;
        Connection connection = null;

        Session session = null;

        Destination destination;

        MessageProducer messageProducer = null;

        connectionFactory = new ActiveMQConnectionFactory(JMSProducer.USERNAME, JMSProducer.PASSWORD, JMSProducer.BROKERURL);

        try {
            connection = connectionFactory.createConnection();

            connection.start();

            session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);

            destination = session.createQueue("topic_style");

            messageProducer = session.createProducer(destination);

            sendMessage(session, messageProducer);

            session.commit();
        } catch (JMSException e) {
            e.printStackTrace();
        } finally {
            if (messageProducer != null) {
                try {
                    messageProducer.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if (session != null) {
                try {
                    session.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
            if (connection != null) {
                try {
                    connection.close();
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private static void sendMessage(Session session, MessageProducer messageProducer) throws JMSException {
        for (int i = 0; i < JMSProducer.SENDNUM; i++) {

            TextMessage message = session.createTextMessage("ActiveMQ 发送消息" + i);
            System.out.println("发送消息：Activemq 发送消息：" + i);
            messageProducer.send(message);
        }
    }
}
```



