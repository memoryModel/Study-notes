# ActiveMQ

## 1、简介

ActiveMQ就是JMS规范的具体实现；它是Apache下的一个项目，采用Java语言开发；是一款非常流行的开源消息服务器，ActiveMQ 同时也是一个 MOM，具体来说是一个实现了 JMS 规范的系统间远程通信的消息代理

## 2、应用场景

消息队列在实际应用中常用的使用场景为异步处理请求、应用解耦、流量削峰、消息通信四个场景

## 3、安装步骤

1. http://activemq.apache.org/activemq-5140-release.html 下载

2. tar -zxvf apache-activemq-5.14.0-bin.tar.gz

3. cd apache-activemq-5.14.0/bin

4. ./activemq start

5. 关闭防火墙

   1. 如果使用了云服务器需要先开启8161(web管理页面端口）、61616（activemq服务监控端口） 两个端口

   2. 打开linux防火墙端口

      >/sbin/iptables -I INPUT -p tcp --dport 8161 -j ACCEPT&&/etc/init.d/iptables save&&service iptables restart&&/etc/init.d/iptables status
      >
      >/sbin/iptables -I INPUT -p tcp --dport 61616 -j ACCEPT&&/etc/init.d/iptables save&&service iptables restart&&/etc/init.d/iptables status

6. 浏览器访问 http://ip:8161/admin

7. 点击Manage ActiveMQ broker  默认用户名和密码admin

## 4、JMS

### 4.1、 概念

java消息服务（Java Message Service）是java平台中关于面向消息中间件的API，用于在两个应用程序之间，或者分布式系统中发送消息，进行异步通信

JMS是一个与具体平台无关的API，绝大多数的MOM（Message Oriented Middleware）（面向消息中间件）提供商都对JMS提供了支持

#### 4.1.1、什么是MOM

面向消息的中间件，使用消息传送提供者来协调消息传输操作。MOM需要提供API和管理工具，客户端调用API，把消息发送到消息传送提供者指定的目的地

在消息发送之后，客户端会继续执行其它的工作。并且在接受方收到消息之前，提供者会一直保留改消息

### 4.2、JMS API

ConnectionFactory	连接工厂

Connection			封装客户端与JMS provider之间的一个虚拟的连接

Session		生产与消费消息的一个单线程上下文；用于创建producer、consumer、message、queue...\

Destination			消息发送或者消息接受的目的地

MessageProducer  	消息生产者

MessageConsumer	消息消费者

### 4.3、JMS的可靠性机制

JMS消息被确认之后，才会认为是成功消费消息。消息的消费包含三个阶段：客户端接受消息、客户端处理消息、消息被确认

#### 4.3.1、事务性会话

```java
// 设置为true时，消息会在session.commit以后自动签收
Session session = connection.createSession(Boolean.TRUE, Session.AUTO_ACKNOWLEDGE);
```

#### 4.3.2、非事务性会话

在该模式下，消息何时被确认取决于创建会话时的应答模式

如果使用持久化发送策略，那么该发送方式为同步模型

```java
// 非事务性会话要将第一个参数设置为FLALSE，不然第二个参数不生效
Session session = connection.createSession(Boolean.FALSE, Session.AUTO_ACKNOWLEDGE);
```

**Session.AUTO_ACKNOWLEDGE**

当客户端成功从recive方法返回后，或者【MessageListener.onMessage】方法成功返回以后，会话会自动确认该消息

**Session.CLIENT_ACKNOWLEDGE**

客户端通过调用消息的`message.acknowledge(); `确认消息

在这种模式中，如果一个消息消费者一共消费了十个消息，目前消费了四个消息，然后在第五个消息通过`message.acknowledge(); `那么之前所有消息都会被消费

**Session.DUPS_OK_ACKNOWLEDGE（延迟确认）**

#### 4.3.3、本地事务

> - 在一个JMS客户端，可以使用本地事务来组合消息的发送和接收。JMS Session 接口提供了commit和rollback
>
> - 方法JMS Provider会缓存每个生产者当前生产的所有消息，直到commit或者rollback，commit操作将会导致事务中所有的消息被持久存储；rollback意味着JMS Provider将会清除此事务下所有的消息记录。在事务未提交之前，消息是不会被持久化存储的，也不会被消费者消费;
>
> - 事务提交意味着生产的所有消息都被发送，消费的所有消息都被确认
> - 事务回滚意味着生产的所有消息被销毁，消费的所有消息被恢复，也就是下次仍然能够接收到发送端的消息，除非消息已经过期了

```java
Destination destination = session.createQueue("first-queue");
Destination destination1 = session.createQueue("first-queue1");
session.rollback();
```

#### 4.3.4、JMS（p2p）模型

- 如果session关闭时，有一些消息已经收到，但还没有被签收，那么当消费者下次连接到相同的队列时，消息还会被签收
- 如果用户在receive方法中设定了消息选择条件，那么不符合条件的消息会留在队列中不会被接收
- 队列可以长久保存消息直到消息被消费者签收。消费者不需要担心因为消息丢失而时刻与jms provider保持连接状态

#### 4.3.5、JMS（pub/sub）模型

- 订阅可以分为非持久订阅和持久订阅
- 当所有的消息必须接收的时候，则需要用到持久订阅。反之，则用非持久订阅

### 4.4 Broker

轻量级，ActiveMQ运行在Broker之中

自定义Broker持久化数据所在地址？

> ActiveMQ默认持久化策略KahaDB定义的地址，其地址会在控制台输出

## 5、消息传递域

### 5.1、点对点（p2p）

- 每个消息只能有一个消费者
- 消息的生产者和消费者之间没有时间上的相关性。无论消费者在生产者发送消息的时候是否处于运行状态，都可以提取消息

### 5.2、发布订阅（pub/sub）

- 每个消息可以有多个消费者
- 消息的生产者和消费者之间存在时间上的相关性，消费者只能消费开启订阅之后接收到的消息；**JMS规范允许提供客户端创建持久订阅**

## 6、消息的组成

### 6.1、消息头

包含消息的识别信息和路由信息

### 6.2、消息体

TextMessage

MapMessage

BytesMessage

StramMessage

ObjectMessage 

### 6.3、属性

## 7、ActiveMQ持久化存储

### 7.1、kahaDB （缺省）

```xml
<persistenceAdapter>
     <kahaDB directory="${activemq.data}/kahadb"/>
</persistenceAdapter>
```

### 7.2、AMQ

基于文件的存储方式

写入速度快，容易恢复 

文件大小默认32M（会有清理历史数据的策略）

```xml
<persistenceAdapter>
     <amqPersistenceAdapter directory="" maxFileLength="" />
</persistenceAdapter>
```

### 7.3、JDBC

#### 7.3.1、JDBC基于数据库的存储

```xml
<!-- 需要引入指定的jar包到ActiveMQ的lab包下 -->
<persistenceAdapter>
    <!-- createTablesOnStartup:初始是否生成数据库表（默认false） -->
    <!-- #号代表引用 -->
    <jdbcPersistenceAdapter dataSource="#dataSource" createTablesOnStartup="true" />
</persistenceAdapter>

<!-- 需要在broker标签外引入该Bean -->
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="${jdbc.driverClassName}" />
    <property name="url" value="${jdbc.url}" />
    <property name="username" value="${jdbc.username}" />
    <property name="password" value="${jdbc.password}" />
</bean>
```

ACTIVEMQ_ACKS ： 存储持久订阅的信息

ACTIVEMQ_LOCK ： 锁表（用来做集群的时候，实现master选举的表）

ACTIVEMQ_MSGS ： 消息表

#### 7.3.2、JBDBC Message store with activeMQ journal

- 引入了快速缓存机制，缓存到Log文件中
- 性能比jdbc store要好
- 不能应用于master/slave模式

### 7.4、Memory 基于内存存储

### 7.5、LevelDB

5.8之后引入的持久化策略，通常用于集群配置；如果使用master、slave模式，则需要和zookeeper进行集成使用

## 8、ActiveMQ的网络连接

activeMQ如果要实现扩展性和高可用性要求的话，就需要用到网络连接模式

### 8.1、NetworkConnector

主要用来配置Broker于Broker之间的通信连接

![](http://47.94.158.71/2018/12/19/图片 1.png)

#### 8.1.1、静态网络连接

分为`单向连接`、`双向连接`

如果使用`双向连接`，配置`duplex=true`则不需要在两端服务器进行配置

修改`activemq.xml`，增加如下内容：

```xml
<networkConnectors>
    <!-- url地址为配置的另一台机器的ip加端口 -->
	<networkConnector uri="static://(tcp://172.16.93.101:61616)"  />
</networkConnectors>
```

![dd](http://47.94.158.71/2018/12/19/12345665432.png)

> 实现效果：
>
> ​	两个Brokers通过一个static的协议来进行网络连接；一个Consumer连接到BrokerB的一个地址上，当Producer在BrokerA上以相同的地址发送消息时，此时消息会被转移到BrolerB上，也就是说BrokerA会转发到BrokerB上

##### 8.1.1.1、消息丢失

**丢失产生**

> 一些consumer连接到broker1、消费broker2上的消息。消息先被broker1从broker2消费掉，然后转发给这些consumers。假设，转发消息的时候broker1重启了，这些consumers发现brokers1连接失败，通过failover连接到broker2.但是因为有一部分没有消费的消息被broker2已经分发到broker1上去了，这些消息就好像消失了。除非有消费者重新连接到broker1上来消费

**解决方式**

> 从5.6版本开始，在destinationPolicy上新增了一个选项replayWhenNoConsumers属性，这个属性可以用来解决当broker1上有需要转发的消息但是没有消费者时，把消息回流到它原始的broker。同时把enableAudit设置为false，为了防止消息回流后被当作重复消息而不被分发
>
> 通过如下配置，在activeMQ.xml中。 分别在两台服务器都配置。即可完成消息回流处理

```xml
<policyEntry queue=">" enableAudit="false">
    <networkBridgeFilterFactory>
        <conditionalNetworkBridgeFilterFactory replayWhenNoConsumers="true" />
    </networkBridgeFilterFactory>
</policyEntry>
```

#### 8.1.2、动态网络连接



## 9、消息的发送策略

### 9.1、持久化消息

#### 9.1.1、异步消息方式

默认情况下，生产者发送的消息是持久化；消息发送到broker以后，producer会等待broker对这条消息处理情况的反馈（同步模型）

```java
// 发送端 ： 持久化消息（默认）
textMessage.setJMSDeliveryMode(DeliveryMode.PERSISTENT);
```

可以设置消息发送端发送持久化消息的<font color="#FF0000">**异步方式**</font>

```java
ActiveMQConnectionFactory connectionFactory =
                new ActiveMQConnectionFactory("tcp://172.16.93.101:61616");
connectionFactory.setUseAsyncSend(true);
```

#### 9.1.2、回执窗口大小设置

消息发送方设置回执窗口大小，如果发送消息大小达到指定值，需要服务器消息的回执，size会做一个递减

```java
connectionFactory.setProducerWindowSize();
```

### 9.2、非持久化消息

非持久化消息不会把消息数据持久化到硬盘里面，而是会存放在内存之中

非持久化消息默认就是异步发送过程

```java
// 发送端 ： 非持久化消息
textMessage.setJMSDeliveryMode(DeliveryMode.NON_PERSISTENT);
```

非持久化消息模式下，默认就是异步发送的过程，如果需要对非持久化消息的每次发送的消息都获得broker的回执的话

```java
connectionFactory.setAlwaysSyncSend();	
```

## 10、消息确认模型

ACK_TYPE  消费端和broker交换ack指令，还需要告知broker  ACK_TYPE

ACK_TYPE表示确认指令的类型，broker可以根据不同的ACK_TYPE去针对当前消息做不同的应对策略

ACK_TYPE中值：

REDELIVERED_ACK_TYPE (broker会重新发送该消息)  重发侧策略

DELIVERED_ACK_TYPE  消息已经接收，但是尚未处理结束

STANDARD_ACK_TYPE  表示消息处理成功

...

## 11、ActiveMQ支持的传输协议

ActiveMQ支持的client-broler通讯协议有：TCP（缺省）、NIO、UDP、SSL、Http(s)、VM

TCP协议默认监听端口L：61616

每一种传输协议都有其特点，应根据实际应用场景来取决使用哪种协议



## 问题答疑

#### 1、consumer获取消息是pull还是（broker的主动push）？

>- 默认情况下，mq服务器（broker）采用异步的方式主动向客户端推送消息（push）
>
>- broker在向某个消费者会话推送消息之后，不会等待消费者响应消息，直到消费者处理完消息以后，主动向broker返回处理结果
>- broker端一旦有消息，就主动按照默认设置的规则推送给当前活动的消费者；每次推送都有一定的数量限制，而这个数量就是<font color="#FF0000">prefetchSize（预取消息数量）</font>
>- Queue（prefetchSize）
>  - 持久化消息		prefetchSize=1000
>  - 非持久化消息         prefetchSize=1000
>- Topic（prefetchSize）
>  - 持久化消息		prefetchSize=100
>  - 非持久化消息         prefetchSize=32766
>- 假如prefetchSize=0，对于consumer来说，就是一个pull模式
>- 如何设置prefetchSize大小？
>  - 在Destination创建时，在参数中加入"?customer.prefetchSize=100"

#### 2、为什么acknowledge可以消费之前没有被broker确认的消息？

> 消费端消费，但是在broker上没有被确认的消息列表，会存在一个队列中，在执行acknowledge方法时，会将这个列表清空
>
> deliveredMessages  一个LinkedList集合