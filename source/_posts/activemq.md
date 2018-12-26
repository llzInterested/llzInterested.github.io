---
title: ActiveMQ学习记录
date: 2018-12-24 16:40:05
categories: 
    - java
tags: 
    - 消息队列
    - ActiveMQ  
---
# JMS消息模型
## 名词解释
### Destination
- 目的地，JMS Provider(消息中间件)负责维护，用于对Message进行管理的对象
- MessageProducer需要指定Destination才能发送消息
- MessageConsumer需要指定Destination才能接收消息

### Producer
消息生产者(客户端，生成消息)，负责发送Message到Destination，应用接口为MessageProducer

### Consumer(Receiver)
消息消费者,负责从目的地中消费【处理|监听|订阅】Message，应用接口为MessageConsumer

### Message
消息，消息封装一次通信的内容，常见类型：StreamMessage,BytesMessage,TextMessage,ObjectMessage,MapMessage,传输时得实现Serializable接口

### ConnectionFactory
连接工厂，用于创建连接的工厂类型

### Connection
连接，用于建立访问ActiveMQ连接的类型，由连接工厂创建

### Session
会话，一次持久有效有状态的访问，由连接创建

### Queue & Topic
- Queue是队列目的地,队列中的消息，默认只能由唯一的一个消费者处理，一旦处理后消息删除
- Topic是主题目的地，主题中的消息，会发送给所有的消费者同时处理，只有在消息可以重复处理的业务场景中可使用
- Queue和Topic都是Destination的子接口

## JMS消息确认机制
在session接口中定义几个常量：
- AUTO_ACKNOWLEDGE = 1  自动确认，常用，商业开发不推荐
- CLIENT_ACKNOWLEDGE = 2    客户端手动确认
- DUPS_OK_ACKNOWLEDGE = 3   一个消息可以多次处理，可以降低session消耗，在可以容忍重复消息时使用，不推荐使用
- SESSION_TRANSACTED = 0    事务提交并确认

## activemqAPI
### activemq作用
1. 同步转异步
2. 分流

### ProducerAPI
#### 发送消息
> MessageProducer.send(Message message);  

发送消息到默认目的地(创建Producer时指定的目的地)


>    send(Destination destination,Message message);     

发送消息到指定目的地，Producer不建议绑定目的地，即创建Producer时使用如下写法：`session.createProducer(null)`

>    send(Message message,int deliveryMode,int priority,long timeToLive);    

发送消息到默认目的地，且设置相关参数
- deliveryMode：持久化方式，取值
    - DeliveryMode.PERSISTENT(持久化) ，消息会持久化到数据库(kahadb,JDBC等)
    - DeliveryMode.NON_PERSISTENT(不持久化)，消息只保存到内存中
- priority：优先级，0-9，取值越大优先级越高，不保证绝对顺序
- timeToLive：消息有效期，单位ms，过期后会将失效消息保存到死信队列(ActiveMQ_DLQ),不持久化的数据超时后直接丢弃，不保存到死信队列，死信队列中的消息不能恢复

>    send(Destinnation destination,Message message,int deliveryMode,int priority,long timeToLive);

发送消息到指定目的地，且设置相关参数

### ConsumerAPI
#### 消息确认
- Consumer拉取消息后，如果没有确认`acknowledge()`，此消息不会从mq中删除，如果有其他新的consumer访问mq，会拉取到重复的消息
- 消息如果被拉去到consumer后，未确认，那么消息被锁定，如果consumer关闭的时候仍旧没有确认消息，则释放消息锁定信息，消息将发送给其他consumer处理
- 消息一旦处理，必须确认，类似数据库中的事务管理机制


## P2P点对点模型
### 概述
- P2P是基于Queue实现的消息处理方式
- 每个消息都被发送到Queue中，消费者从Queue中获取消息并消费
- 消息被消费后，Queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息
- Queue支持存在多个消费者，但对一个消息而言，只会有一个消费者可以消费，其他消费者不能消费此消息了
- 当消费者不存在时，消息会一直保存，直到有消费者来消费

==如果希望发送的每个消息都会被成功处理，需要使用P2P模式==

### 特点
- 每个消息只有一个消费者(Consumer)，即一旦被消费，消息就不再在消息队列中
- 发送者和接收者之间在时间上没有依赖性，即当发送者发送消息后，不管接收者有没有在运行，它不会影响消息被发送到队列
- 接收者在成功接收消息后需向队列应答成功

### demo
```java
//消息的发送方-------生产者
@Test
public void test1() throws JMSException {
    /**创建连接工厂对象
     * 创建工厂构造方法有3个参数，分别为用户名，密码，连接地址
     * 无参构造：有默认的连接地址，本地连接
     * 一个参数(连接地址)的构造：无验证模式的，没有用户认证
     * 三个参数的构造：有认证+指定地址
     */
    ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
    //从工厂中获取一个连接对象
    Connection connection = connectionFactory.createConnection();
    /**建议启动连接，生产者不是必须启动连接，但消费者必须启动连接
     * 生产者在发送消息的时候，会检查是否启动了连接，如果未启动，会自动启动
     * 如果有特殊的配置，建议配置完毕后再启动连接
     */
    connection.start();
    /**获得session对象
     * 第一个参数transacted：是否支持事务
     *      true：支持事务，第二个参数对生产者默认无效，建议设为Session.SESSION_TRANSACTED
     *      false：不支持事务，第二个参数必须传递且有效
     * 第二个参数acknowledgeMode：如何确认消息的处理
     */
    Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
    //创建目的地，参数是目的地名称，是目的地的唯一标记
    Destination destination = session.createQueue("dest");
    //通过session对象创建消息的发送者
    MessageProducer messageProducer = session.createProducer(destination);
    //通过sesion对象创建消息对象
    TextMessage textMessage = session.createTextMessage("测试的文本消息");
    //通过发送者发送消息
    messageProducer.send(textMessage);
    //关闭资源
    messageProducer.close();
    session.close();
    connection.close();
}

//消息的接收方-----消费者
@Test
public void test2() throws JMSException {
    //创建连接工厂对象
    ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
    //从工厂中获取一个连接对象
    Connection connection = connectionFactory.createConnection();
    //连接MQ服务
    connection.start();
    //获得session对象
    Session session = connection.createSession(false,Session.CLIENT_ACKNOWLEDGE);
    //创建目的地
    Destination destination = session.createQueue("dest");
    //创建消费者对象
    MessageConsumer consumer = session.createConsumer(destination);
    //注册监听器,注册成功后，队列中的消息变化会主动触发监听器代码
    consumer.setMessageListener(new MessageListener() {
        @Override
        public void onMessage(Message message) {
            try {
                //acknowledge()方法确认，代表consumer已接收到消息，之后mq删除对应的消息
                message.acknowledge();
                TextMessage textMessage = (TextMessage)message;
                System.out.println("读取的消息：" + textMessage);
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    });
    //阻塞当前代码，保证listener代码未结束，如果代码结束，监听器自动关闭
    while(true){}
}
```

## Publish/Subscribe(Pub/Sub)发布订阅模式
### 概述
- Pub/Sub模式基于Topic实现的消息处理方式
- Pub/Sub模式包含3个角色：主题(Topic),发布者(Publisher),订阅者(Subscriber)
- 多个发布者将消息发送到Topic，系统将这些消息传递给多个订阅者

==如果希望发送的消息可以被多个消费者处理的话，那么可以采用Pub/Sub模型==
### 特点
- 每个消息可以有多个消费者
- 发布者和订阅者之间有时间上的依赖性，针对某个主题(Topic)的订阅者，它必须创建一个订阅后才能消费发布者的消息
- 当生产者发布消息，不管有没有消费者，都不会保存消息
- 为了消费消息，订阅者必须保持运行状态
- 为了缓和这样严格的时间相关性，JMS运行订阅者创建一个可持久化的订阅，这样即使订阅者没有被激活(运行)，它也能接收到发布者的消息

### Topic消息失败重发

 
##### demo
- 消息消费端在创建session对象时需指定应答模式为客户端手动应答
- 当消费者获取到消息并成功处理后需要调用 ==message.acknowledge()== 方法进行应答，通知Broker消费成功
- 如果处理过程出现异常，需调用 ==session.recover()== 通知Broker重发消息，重发有次数限制


```java
public class ActiveMQTest {
    //消息的发送方-------生产者
    @Test
    public void test1() throws JMSException {
        /**创建连接工厂对象
         * 创建工厂构造方法有3个参数，分别为用户名，密码，连接地址
         * 无参构造：有默认的连接地址，本地连接
         * 一个参数(连接地址)的构造：无验证模式的，没有用户认证
         * 三个参数的构造：有认证+指定地址，默认端口61616，可以在conf/activemq.xml文件中查看
        */
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
        //从工厂中获取一个连接对象
        Connection connection = connectionFactory.createConnection();
        /**建议启动连接，生产者不是必须启动连接，但消费者必须启动连接
         * 生产者在发送消息的时候，会检查是否启动了连接，如果未启动，会自动启动
         * 如果有特殊的配置，建议配置完毕后再启动连接
         */
        connection.start();
        //获得session对象
        Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
        //通过session对象创建topic
        Topic topic = session.createTopic("testTopic");
        //通过session对象创建消息的发送者
        MessageProducer messageProducer = session.createProducer(topic);
        //通过sesion对象创建消息对象
        TextMessage textMessage = session.createTextMessage("测试的文本消息");
        //通过发送者发送消息
        messageProducer.send(textMessage);
        //关闭资源
        messageProducer.close();
        session.close();
        connection.close();
    }

    //消息的接收方-----消费者
    @Test
    public void test2() throws JMSException {
        //创建连接工厂对象
        ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
        //从工厂中获取一个连接对象
        Connection connection = connectionFactory.createConnection();
        //连接MQ服务
        connection.start();
        //获得session对象
        final Session session = connection.createSession(false,Session.CLIENT_ACKNOWLEDGE);
        //通过session对象创建topic
        Topic topic = session.createTopic("testTopic");
        //通过session对象创建消息的接收者
        MessageConsumer messageConsumer = session.createConsumer(topic);
        //指定消息的监听器
        messageConsumer.setMessageListener(new MessageListener() {
            //当监听的topic中存在消息，这个方法自动执行
            @Override
            public void onMessage(Message message) {
                TextMessage textMessage = (TextMessage) message;
                try {
                    if(textMessage.getText().equals("ping")){
                        System.out.println("消费者接收到的消息：" + textMessage.getText());
                        //客户端手动应答
                        message.acknowledge();
                    }else{
                        //模拟消息处理失败
                        System.out.println("消息处理失败");
                        //通知mq进行消息重发
                        session.recover();
                        int i = 1 / 0;

                    }
                } catch (JMSException e) {
                    e.printStackTrace();
                }
            }
        });
        while(true){}
    }
}

```

## P2P和SUB对比

|| Topic | Queue
---|---|---
有无状态 | topic数据默认不落地，是无状态的,消息发送后mq中不存储 | Queue数据默认会在mq服务器上以文件形式保存，比如ActiveMQ一般保存在$AMQ_HOME/data/kahadb下，也可以配置成DB存储
完整性保障 | 不保证发布者发布的每条数据，订阅者都能收到 | Queue保证每条数据都能被接收到，前提是消息未超时
消息是否会丢失 | 发布者发布消息到某个topic时，只有正在监听该topic地址的订阅者能接收到消息，若没有订阅者在监听，该topic就丢失了 | 发送者发送消息到目标Queue，接收者可以异步接收这个Queue上的消息，Queue上的消息如果暂时没有接收者接收，也不会丢失，前提是消息未超时
消息发布接收策略 | 一对多的消息发布接收策略，监听同一个topic地址的多个订阅者都能收到发布者发送的消息，订阅者接收完通知mq服务器 | 一对一的消息发布接收策略，一个发送者发送的消息，只能有一个接收者接收，通知mq服务器已接受，mq服务器对queue中的消息采取删除或其他操作

# ActiveMQ安全认证
即用户名密码登录规则，如果需要使用安全认证
1. 必须在activemq核心配置文件(conf/activemq.xml)中开启安全配置,在broker标签下添加如下内容：
```xml
<plugins>
    <!--  use JAAS to authenticate using the login.config file on the classpath to configure JAAS -->
　　　　　　　<!--  添加jaas认证插件activemq在login.config里面定义,详细见login.config-->

    <jaasAuthenticationPlugin configuration="activemq" />
    <!--  lets configure a destination based authorization mechanism -->
    <authorizationPlugin>
        <map>
            <authorizationMap>
                <authorizationEntries>
                    <authorizationEntry topic=">" read="admins" write="admins" admin="admins" />
                    <authorizationEntry queue=">" read="admins" write="admins" admin="admins" />
                    <!--authorizationEntry topic="FirstTopic" read="smeall,smeadmin" write="smeadmin" admin="smeall,smeadmin" /-->
                    <authorizationEntry topic="ActiveMQ.Advisory.>" read="admins" write="admins" admin="admins"/>
                    <authorizationEntry queue="ActiveMQ.Advisory.>" read="admins" write="admins" admin="admins"/>
                </authorizationEntries>
            </authorizationMap>
        </map>
    </authorizationPlugin>
</plugins>
```
2. 开启认证后，认证使用的用户信息由其他配置文件提供conf/login.config
```xml
activemq {
    org.apache.activemq.jaas.PropertiesLoginModule required
        org.apache.activemq.jaas.properties.user="users.properties"
        org.apache.activemq.jaas.properties.group="groups.properties";
};
```
conf/users.properties中"用户名=密码"    
conf/groups.properties中"用户组名=用户名，用户名"

# ActiveMQ持久化
- ActiveMQ中持久化是指对消息数据的持久化
- 在ActiveMQ中，默认的消息保存在内存中
- 当内存容量不足的时候，或ActiveMQ正常关闭的时候，会将内存中未处理的消息持久化到磁盘中，具体的持久化策略由配置文件中具体配置决定
- 默认存储策略是kahadb
- 如果使用jdbc作为持久化策略，则会将所有的需要持久化的消息保存到数据库中
- 所有持久化配置都在conf/activemq.xml中，配置信息在broker标签中定义

## kahadb方式(默认)
```xml
<persistenceAdapter>
    <kahaDB directory="${activemq.data}/kahadb"/>
</persistenceAdapter>
```
### 概念
- kahadb是一个文件型数据库，使用内存+文件保证数据持久化
- kahadb可以限制每个数据文件的大小，不代表总计数据容量

### 特点
- 日志形式存储消息
- 消息索引以B-Tree结构存储，可以快速更新
- 完全支持JMS事务
- 支持多种恢复机制

 
## JDBC持久化方式
### 前期工作
- ActiveMQ将数据持久化到数据库中，不指定具体数据库，这里以mysql为例
- 其中jdbcPersistenceAdapter标签中的createTableOnStartup属性默认为true，表示每次启动时都去创建数据表，一般是第一次启动时设置为true，之后改为false
- 将mysql的数据库驱动jar包复制到activeMQ的lib目录下
- 创建相应的数据库(demo数据库名为activemq)，之后启动activemq会在对应的数据库创建3张表

---
activemq_msgs表存储消息，Queue和Topic都存储在这个表中
字段 | 描述
---|---
ID | 自增主键
container | 消息的destination
msgid_prod | 消息发送者客户端的主键
msg_seq | 发送消息的顺序，msgid_prop+msg_seq可以组成JMS的MessageID
expiration | 消息的过期时间，存储的是从1970-01-01到现在的毫秒数
msg | 消息本体的java序列化对象的二进制数据
priority | 优先级，从0-9，数值越大优先级越高

---

activemq_acks表存储订阅关系，如果是持久化Topic，订阅者和服务器的订阅关系保存在这
字段 | 描述
---|---
container | 消息的destination
sub_dest | 如果是使用static集群，这个字段会有集群其他系统的信息
client_id | 每个订阅者都必须有一个唯一的客户端id
sub_name | 订阅者名称
selector | 选择器，可以选择只消费满足条件的消息，条件可以用自定义属性实现，可支持多属性AND和OR操作
last_acked_id | 记录消费过的消息的ID

---

activemq_lock表在集群环境中才有用，只有一个Broker可以获得消息，称为Master Broker，其他的只能作为备份等待Master Broker不可用，才可能成为下一个Master Broker，这个表用来记录哪个Broker是当前的Master Broker

```xml
<bean id="mysql-ds" class="org.apache.commons.dbcp.BasicDataSource" destory-method="close">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value=""jdbc:mysql://localhost:3306/activemq?relaxAutoCommit=true/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    <property name="poolPreparedStatements" value="true"/> 
</bean>

<broker xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}">
    <persistenceAdapter>
        <jdbcPersistenceAdapter dataDirectory="${activemq.base}/data" dataSource="#mysql-ds" createTableOnStartup="true"/>
    </persistenceAdapter>
</broker>
```

### demo
1. 消息发送方发送时send()方法改变
```java
//创建连接工厂对象
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
//从工厂中获取一个连接对象
Connection connection = connectionFactory.createConnection();
//连接MQ服务
connection.start();
//获得session对象
Session session = connection.createSession(false,Session.AUTO_ACKNOWLEDGE);
//通过session对象创建topic
Topic topic = session.createTopic("testTopic");
//通过session对象创建消息的发送者
MessageProducer messageProducer = session.createProducer(topic);
/**
*message：发送的消息
*deliveryMode：取值为DeliveryMode.PERSISTENT 表示持久化
*priority：优先级
*timeToLive：超时时间，单位ms
*/
messageProducer.send(message,deliveryMode,priority,timeToLive)
```
2. 消息接收方设置clientID以及创建消息接收者方法改变
```java
//创建连接工厂对象
ConnectionFactory connectionFactory = new ActiveMQConnectionFactory("tcp://localhost:61616");
//从工厂中获取一个连接对象
Connection connection = connectionFactory.createConnection();
//连接MQ服务
connection.start();
connection.setClientID("client-1");
//获得session对象
Session session = connection.createSession(false,Session.CLIENT_ACKNOWLEDGE);
//通过session对象创建topic
Topic topic = session.createTopic("testTopic");
//通过session对象创建消息接收者
TopicSubscriber consumer = session.createDurableSubscriber(topic,"client1-sub");
```