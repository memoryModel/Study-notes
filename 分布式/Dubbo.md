## Dubbo

### 简介

Dubbo是alibaba的一款开源软件，它是基于java的RPC调用框架

Dubbo主要提供了三种功能：

- 提供了基于接口的远程调用接口
- 容错性和负载均衡
- 服务自动注册及发现

### Dubbo架构

![dubbo架构图](http://i2.51cto.com/images/blog/201802/09/4b475373661f5c1372b21dea2c990127.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

| 节点      | 说明                                   |
| --------- | -------------------------------------- |
| Provider  | 暴露服务的服务提供方                   |
| Consumer  | 调用远程服务的服务消费方               |
| Registry  | 服务注册与发现的注册中心               |
| Monitor   | 统计服务的调用次数和调用时间的监控中心 |
| Container | 服务运行容器                           |

**调用关系说明**

- 服务容器负责启动，加载，运行服务提供者
- 服务提供者在启动时，向注册中心注册自己提供的服务
- 服务消费者在启动时，向注册中心订阅自己所需的服务

### 配置覆盖关系

以 timeout 为例，显示了配置的查找顺序，其它 retries, loadbalance, actives 等类似：

- 方法级优先，接口级次之，全局配置再次之
- 如果级别一样，则消费方优先，提供方次之
- 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
- 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

### 启动服务检查

**< dubbo:reference>**中的check属性，默认值为true，可设置为false

false定义为：对依赖服务不进行检查

true定义为：启动时会对所依赖的服务进行检查提供方是否正常提供服务

什么情况下check属性必须要设置为false？

> 循环依赖情况下，check属性必须设置为false，比如：订单和pay两个服务相互依赖，启动时肯定会有个加载顺序 

**< dubbo:consumer check="false" />** 没有服务提供者会报错

**< dubbo:registry check="false" /> **注册订阅失败报错

### 多协议支持

**dubbo://**

> Dubbo缺省协议采用单一长连接和NIO异步通讯，适合于小数据量大并发的服务调用，以及服务消费者机器数远大于服务提供者机器数的情况
>
> Dubbo缺省协议不适合传送大数据量的服务，比如传文件，传视频等，除非请求量很低
>
> Dubbo协议缺省每服务每提供者每消费者使用单一长连接，如果数据量较大，可以使用多个连接
>
> 连接个数：单连接
>
> 连接方式：长连接
>
> 传输协议：TCP
>
> 传输方式：NIO异步传输
>
> 序列化：Hessian二进制序列化
>
> 适用范围：传入传出参数数据包较小（建议小于100K），消费者比提供者个数多，单一消费者无法压满提供者，尽量不要用dubbo协议传输大文件或超大字符串
>
> 适用场景：常规远程服务方法调用

**rmi://**

**hessian://**

**http://**

**webservice://**

**thrift://**

**memcached://**

**redis://**

**为什么要使用多协议？**

> dubbo是基于netty协议的一种长连接，根据不同的应用场景，进行适配；数据较小或者并发较大时，使用长连接；数据较大，使用短连接

### 多注册中心支持

可以配置多个注册中心

在不同的发布服务上，引用不同的注册中心

### 多版本支持

当一个接口实现，出现不兼容升级时，可以用版本号过渡，版本号不同的服务相互间不引用

- **在低压力时间段，先升级一半提供者为新版本**
- **再将所有消费者升级为新版本**
- **然后将剩下的一半提供者升级为新版本**

```xml
<dubbo:service interface="com.foo.BarService" version="1.0.0" />
<dubbo:service interface="com.foo.BarService" version="2.0.0" />
<dubbo:reference id="barService" interface="com.foo.BarService" version="1.0.0" />
<dubbo:reference id="barService" interface="com.foo.BarService" version="2.0.0" />
```

### 异步调用

在消费方< dubbo:reference />里加入async=“true”

不支持hessian协议

### 主机绑定

> **1》第一步：**从配置文件中获取host，检验host是否合理，如果合理，则直接返回。反之，进行下一步的获取
>
> ```java
> String host = protocolConfig.getHost();
> NetUtils.isInvalidLocalHost(host);
> ```
>
> **2》第二步：**获取本地ip地址，检验是否合理，如果合理，则直接返回。反之，进行下一步的获取
>
> ```java
> host = InetAddress.getLocalHost().getHostAddress();
> NetUtils.isInvalidLocalHost(host);
> ```
>
> **3》第三步：**利用注册中心去获取，首先判断注册中心地址是不是不为空，如果不为空，循环每个注册中心（dubbo支持多注册中心），通过socket去连接对应的注册中心地址，连接成功后，再通过socket获取本地ip地址。检验是否合理，如果合理，则直接返回。反之，进行下一步的获取
>
> ```java
> Socket socket = new Socket();
> try {
>     SocketAddress addr = new InetSocketAddress(registryURL.getHost(), registryURL.getPort());
>     socket.connect(addr, 1000);
>     host = socket.getLocalAddress().getHostAddress();
>     break;
> } finally {
>     try {
>         socket.close();
>     } catch (Throwable e) {}
> }
> ```
>
> **4》第四步（最后一步）：**遍历本地网卡，返回第一个合理的ip地址
>
> ```java
> host = NetUtils.getLocalHost();
> public static String getLocalHost(){
>     InetAddress address = getLocalAddress();
>     return address == null ? LOCALHOST : address.getHostAddress();
> }
> /**
> * 遍历本地网卡，返回第一个合理的IP。
> * 
> * @return 本地网卡IP
> */
> public static InetAddress getLocalAddress() {
>     if (LOCAL_ADDRESS != null)
>         return LOCAL_ADDRESS;
>     InetAddress localAddress = getLocalAddress0();
>     LOCAL_ADDRESS = localAddress;
>     return localAddress;
> }
> ```
>
>

### dubbo服务只订阅

只对服务进行订阅，不对自身服务发布

```xml
<dubbo:registry protocol="zookeeper" 					     address="172.16.93.101:2181,172.16.93.102:2181,172.16.93.103:2181"
register="false"/>
```

### dubbo服务只注册

使用场景：多注册中心情况下，如果想只对其中一个注册中心提供服务，不对服务进行订阅，就这么搞

```xml
<dubbo:registry subscribe="false" />
```

### 负载均衡



### 问题

#### spi是什么？

#### dubbo的扩展机制

#### 为什么dubbo相关的配置文件要放在META-INF/spring包下？

> 在Main.main(args)执行时时，读取args中的参数，会有一个spring的key，来源于dubbojar包的internal包下的`com.alibaba.dubbo.container.Container`包，dubbo会从配置文件中读取spring的值，就是`com.alibaba.dubbo.container.spring.SpringContainer`，启动`SpringContainer.class`的`start`方法默认从这个包下读取配置文件，为硬性读取

```java
SpringContainer.class
public void start() {
        String configPath = ConfigUtils.getProperty("dubbo.spring.config");
        if (configPath == null || configPath.length() == 0) {
            configPath = "classpath*:META-INF/spring/*.xml";
        }

        context = new ClassPathXmlApplicationContext(configPath.split("[,\\s]+"));
        context.start();
    }
```

#### 日志是如何集成的？

> dubbo加载时，会查找配置文件中`<dubbo:application name="order-provider" owner="mac" logger="log4j"/>`这段代码，如果配置了`logger`属性，那么会根据所配置的类型加载类型日志，如果没有配置默认`log4j`   

```java
// 代码来源com.alibaba.dubbo.common.logger.LoggerFactory
static {
        String logger = System.getProperty("dubbo.application.logger");
        if ("slf4j".equals(logger)) {
            setLoggerAdapter((LoggerAdapter)(new Slf4jLoggerAdapter()));
        } else if ("jcl".equals(logger)) {
            setLoggerAdapter((LoggerAdapter)(new JclLoggerAdapter()));
        } else if ("log4j".equals(logger)) {
            setLoggerAdapter((LoggerAdapter)(new Log4jLoggerAdapter()));
        } else if ("jdk".equals(logger)) {
            setLoggerAdapter((LoggerAdapter)(new JdkLoggerAdapter()));
        } else {
            try {
                setLoggerAdapter((LoggerAdapter)(new Log4jLoggerAdapter()));
            } catch (Throwable var6) {
                try {
                    setLoggerAdapter((LoggerAdapter)(new Slf4jLoggerAdapter()));
                } catch (Throwable var5) {
                    try {
                        setLoggerAdapter((LoggerAdapter)(new JclLoggerAdapter()));
                    } catch (Throwable var4) {
                        setLoggerAdapter((LoggerAdapter)(new JdkLoggerAdapter()));
                    }
                }
            }
        }

    }
```

#### admin控制台如何安装

>  控制台用作服务治理  比如控制服务的权重、路由等等
>
> 1. 下载dubbo源码
> 2. 找到dubbo-admin项目
> 3. 进入webapp/WEB-INF/dubbo.properties文件
> 4. 修改dubbo.registry.address中zookpper集群路径
> 5. 启动项目

#### simple 监控中心

> 监控服务的调用次数、调用关系、响应事件等等

