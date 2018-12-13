## Zookeeper

### 简介

zookeeper是一个开源的分布式协调服务，是由雅虎创建的，基于google chubby。

能够用作于数据的发布/订阅（配置中心:disconf）  、 负载均衡（dubbo利用了zookeeper机制实现负载均衡） 、命名服务、master选举(kafka、hadoop、hbase)、分布式队列、分布式锁

<font color="#FF0000">**zookeeper并不是用来存储数据的，通过监控数据状态的变化，达到基于数据的集群管理**</font>

### 特性

**顺序一致性**

从同一个客户端发起的事务请求，最终会严格按照顺序被应用到zookeeper中

**原子性**

所有的事务请求的处理结果在整个集群中的所有机器上的应用情况是一致的，也就是说，要么整个集群中的所有机器都成功应用了某一事务、要么全都不应用

**可靠性**

一旦服务器成功应用了某一个事务数据，并且对客户端做了响应，那么这个数据在整个集群中一定是同步并且保留下来的

**实时性**

一旦一个事务被成功应用，客户端就能够立即从服务器端读取到事务变更后的最新数据状态；（zookeeper仅仅保证在一定时间内，近实时）

### 安装流程

1. 下载zookeeper的.tar.gz安装包，解压命令`tar -zxvf ${zookeeper.name}`
2. `cd conf` 
3. `cp zoo_sample.cfg zoo.cfg`

### 启动流程

1. `cd bin`  进入zookeeper的bin目录
2. `sh zkServer.sh start`  启动zookeeper服务
3. `sh zkCli.sh`  进入zookeeper的客户端
4. `sh zkServer.sh`  查看zookeeper的信息

### 集群角色

**Leader** 

​	 leader接收所有客户端的写请求，然后转发给所有的follower节点

​	 **有且仅有一个Leader节点**

​	 **客户端的写请求一定会发送到leader**

 	 **客户端的读请求会根据算法发送到不同节点**	

​	 **所有follower节点的数据都只会从leader上去同步，followert节点负责投票**

​	 **<font color="#FF0000">事务请求</font>的唯一调度者和处理者，保证集群事务处理的顺序性**

​	 **<font color="#FF0000)">集群内部的各个服务器（Follower、Observer）的调度者</font>**

**Follower**

​	处理客户端非事务请求，以及转发事务请求给Leader服务器

​	参与事务请求提议的投票（客户端的一个事务请求，需要半数以上的Follower服务器投票通过才能通知Leader 进行Commit；Leader会发起一个提案，要求Follower进行投票）

​	参与Leader的选举投票 

**Observer**

​	直接为客户端服务但并不参与提案的投票，同时与leader进行数据交换（同步）

​	不影响写性能的情况下去水平扩展zookeeper

> 如果大量客户端访问zookeeper，需要横向扩展zookeeper集群数量，从而集群的性能。但是会导致写性能下降，因为随着zookeeper集群机器的增加，如果数据发生变动，需要半数以上的服务器投票同意才能实施。造成一定网络消耗增加投票成本。
>
> 1. observer不参与投票，只接受投票结果
> 2. 不属于zookeeper的部位  如果down掉后不会对集群产生影响
> 3. 对于客户端来提高读的性能，对于zookeeper集群来提高写性能

### 集群组成

zookeeper集群一般由2n+1台机器组成，尽量保持在一个奇数

### 会话

客户端连接到zookeeper的一个过程信息

NOT_CONNECTED（未连接） - > CONNECTING（连接中） ->CONNECTED（连接成功）->ClOSE（关闭）

> 客户端在响应接受的最大时间（在连接时配置）如果收不到服务端的响应从 CONNECTED 状态变为 CONNECTING

### Leader选举  *

<font color="#FF0000">leader选举默认算法：基于TCP的**FastLeaderElection** </font>

选举过程详见下文问题总结第三问！

#### 选举算法分析

**1. 第一次投票**

> 无论哪种导致进行Leader选举，集群的所有机器都处于试图选举出一个Leader的状态，即LOOKING状态，LOOKING机器会向所有其他机器发送消息，该消息称为投票。投票中包含了SID（服务器的唯一标识）和ZXID（事务ID），(SID, ZXID)形式来标识一次投票信息。假定Zookeeper由5台机器组成，SID分别为1、2、3、4、5，ZXID分别为9、9、9、8、8，并且此时SID为2的机器是Leader机器，某一时刻，1、2所在机器出现故障，因此集群开始进行Leader选举。在第一次投票时，每台机器都会将自己作为投票对象，于是SID为3、4、5的机器投票情况分别为(3, 9)，(4, 8)， (5, 8)。

**2. 变更投票**

> 每台机器发出投票后，也会收到其他机器的投票，每台机器会根据一定规则来处理收到的其他机器的投票，并以此来决定是否需要变更自己的投票，这个规则也是整个Leader选举算法的核心所在
>
> > - **vote_sid**：接收到的投票中所推举Leader服务器的SID
> > - **vote_zxid**：接收到的投票中所推举Leader服务器的ZXID
> > - **self_sid**：当前服务器自己的SID
> > - **self_zxid**：当前服务器自己的ZXID
>
> 每次对收到的投票的处理，都是对(vote_sid, vote_zxid)和(self_sid, self_zxid)对比的过程
>
> > 1. 如果vote_zxid大于self_zxid，就认可当前收到的投票，并再次将该投票发送出去
> > 2. 如果vote_zxid小于self_zxid，那么坚持自己的投票，不做任何变更
> > 3. 如果vote_zxid等于self_zxid，那么就对比两者的SID，如果vote_sid大于self_sid，那么就认可当前收到的投票，并再次将该投票发送出去
> > 4. 如果vote_zxid等于self_zxid，并且vote_sid小于self_sid，那么坚持自己的投票，不做任何变更

**3. 确定Leader**

> 经过第二轮投票后，集群中的每台机器都会再次接收到其他机器的投票，然后开始统计投票，如果一台机器收到了超过半数的相同投票，那么这个投票对应的SID机器即为Leader。此时Server3将成为Leader

> 由上面规则可知，通常那台服务器上的数据越新（ZXID会越大），其成为Leader的可能性越大，也就越能够保证数据的恢复。如果ZXID相同，则SID越大机会越大

####  Leader选举实现细节

##### 服务器状态

服务器具有四种状态，分别是LOOKING、FOLLOWING、LEADING、OBSERVING

**LOOKING**：寻找Leader状态。当服务器处于该状态时，它会认为当前集群中没有Leader，因此需要进入Leader选举状态

**FOLLOWING**：跟随者状态。表明当前服务器角色是Follower

**LEADING**：领导者状态。表明当前服务器角色是Leader

**OBSERVING**：观察者状态。表明当前服务器角色是Observer

##### 投票数据结构

每个投票中包含了两个最基本的信息，所推举服务器的SID和ZXID，投票（Vote）在Zookeeper中包含字段如下

**id**：被推举的Leader的SID

**zxid**：被推举的Leader事务ID

**electionEpoch**：逻辑时钟，用来判断多个投票是否在同一轮选举周期中，该值在服务端是一个自增序列，每次进入新一轮的投票后，都会对该值进行加1操作

**peerEpoch**：被推举的Leader的epoch

**state**：当前服务器的状态

##### QuorumCnxManager

每台服务器在启动的过程中，会启动一个QuorumPeerManager，负责各台服务器之间的底层Leader选举过程中的网络通信

1. **消息队列**

   > QuorumCnxManager内部维护了一系列的队列，用来保存接收到的、待发送的消息以及消息的发送器，除接收队列以外，其他队列都按照SID分组形成队列集合，如一个集群中除了自身还有3台机器，那么就会为这3台机器分别创建一个发送队列，互不干扰
   >
   > - **recvQueue**：消息接收队列，用于存放那些从其他服务器接收到的消息
   > - **queueSendMap**：消息发送队列，用于保存那些待发送的消息，按照SID进行分组
   > - **senderWorkerMap**：发送器集合，每个SenderWorker消息发送器，都对应一台远程Zookeeper服务器，负责消息的发送，也按照SID进行分组
   > - **lastMessageSent**：最近发送过的消息，为每个SID保留最近发送过的一个消息

2. **建立连接**

   > 为了能够相互投票，Zookeeper集群中的所有机器都需要两两建立起网络连接。QuorumCnxManager在启动时会创建一个ServerSocket来监听Leader选举的通信端口(默认为3888)。开启监听后，Zookeeper能够不断地接收到来自其他服务器的创建连接请求，在接收到其他服务器的TCP连接请求时，会进行处理。<font color="#FF0000">为了避免两台机器之间重复地创建TCP连接，Zookeeper只允许SID大的服务器主动和其他机器建立连接，否则断开连接。在接收到创建连接请求后，服务器通过对比自己和远程服务器的SID值来判断是否接收连接请求，如果当前服务器发现自己的SID更大，那么会断开当前连接，然后自己主动和远程服务器建立连接。一旦连接建立，就会根据远程服务器的SID来创建相应的消息发送器SendWorker和消息接收器RecvWorker，并启动</font>

3. **消息接受与发送**

   1. 消息接受

      > 由消息接收器RecvWorker负责，<font color="#FF0000">由于Zookeeper为每个远程服务器都分配一个单独的RecvWorker，</font>因此，每个RecvWorker只需要不断地从这个TCP连接中读取消息，并将其保存到recvQueue队列中

   2. 消息发送

      > 由于Zookeeper为每个远程服务器都分配一个单独的SendWorker，因此，每个SendWorker只需要不断地从对应的消息发送队列中获取出一个消息发送即可，同时将这个消息放入lastMessageSent中。<font color="#FF0000">在SendWorker中，一旦Zookeeper发现针对当前服务器的消息发送队列为空，那么此时需要从lastMessageSent中取出一个最近发送过的消息来进行再次发送，这是为了解决接收方在消息接收前或者接收到消息后服务器挂了，导致消息尚未被正确处理。同时，Zookeeper能够保证接收方在处理消息时，会对重复消息进行正确的处理</font>

##### FastLeaderElection：选举算法核心

- **外部投票**：特指其他服务器发来的投票
- **内部投票**：服务器自身当前的投票
- **选举轮次**：Zookeeper服务器Leader选举的轮次，即logicalclock

1. 选票管理

   - **sendqueue**：选票发送队列，用于保存待发送的选票
   -  **recvqueue**：选票接收队列，用于保存接收到的外部投票
   - **WorkerReceiver**：选票接收器。其会不断地从QuorumCnxManager中获取其他服务器发来的选举消息，并将其转换成一个选票，然后保存到recvqueue中，在选票接收过程中，<font color="#FF0000">如果发现该外部选票的选举轮次小于当前服务器的，那么忽略该外部投票，同时立即发送自己的内部投票</font>
   -  **WorkerSender**：选票发送器，不断地从sendqueue中获取待发送的选票，并将其传递到底层QuorumCnxManager中

2. 算法核心

   > 1. **自增选举轮次**。Zookeeper规定所有有效的投票都必须在同一轮次中，在开始新一轮投票时，会首先对logicalclock进行自增操作
   > 2. **初始化选票**。在开始进行新一轮投票之前，每个服务器都会初始化自身的选票，并且在初始化阶段，每台服务器都会将自己推举为Leader
   > 3. **发送初始化选票**。完成选票的初始化后，服务器就会发起第一次投票。Zookeeper会将刚刚初始化好的选票放入sendqueue中，由发送器WorkerSender负责发送出去
   > 4. **接收外部投票**。每台服务器会不断地从recvqueue队列中获取外部选票。如果服务器发现无法获取到任何外部投票，那么就会立即确认自己是否和集群中其他服务器保持着有效的连接，如果没有连接，则马上建立连接，如果已经建立了连接，则再次发送自己当前的内部投票
   > 5. **判断选举轮次**。在发送完初始化选票之后，接着开始处理外部投票。在处理外部投票时，会根据选举轮次来进行不同的处理。
   >    - **<font color="#FF0000">外部投票的选举轮次大于内部投票</font>**。若服务器自身的选举轮次落后于该外部投票对应服务器的选举轮次，那么就会立即更新自己的选举轮次(logicalclock)，并且清空所有已经收到的投票，然后使用初始化的投票来进行PK以确定是否变更内部投票。最终再将内部投票发送出去
   >    - **<font color="#FF0000">外部投票的选举轮次小于内部投票</font>**。若服务器接收的外选票的选举轮次落后于自身的选举轮次，那么Zookeeper就会直接忽略该外部投票，不做任何处理，并返回步骤4
   >    - **<font color="#FF0000">外部投票的选举轮次等于内部投票</font>**。此时可以开始进行选票PK
   > 6. **选票PK**。在进行选票PK时，符合任意一个条件就需要变更投票
   >    - 若外部投票中推举的Leader服务器的选举轮次大于内部投票，那么需要变更投票
   >    - 若选举轮次一致，那么就对比两者的ZXID，若外部投票的ZXID大，那么需要变更投票
   >    - 若两者的ZXID一致，那么就对比两者的SID，若外部投票的SID大，那么就需要变更投票
   > 7. **变更投票**。经过PK后，若确定了外部投票优于内部投票，<font color="#FF0000">那么就变更投票，即使用外部投票的选票信息来覆盖内部投票，变更完成后，再次将这个变更后的内部投票发送出去</font>
   > 8. **选票归档**。无论是否变更了投票，都会将刚刚收到的那份外部投票放入选票集合recvset中进行归档。recvset用于记录当前服务器在本轮次的Leader选举中收到的所有外部投票（按照服务队的SID区别，如{(1, vote1), (2, vote2)...}）
   > 9. **统计投票**。完成选票归档后，就可以开始统计投票，统计投票是为了统计集群中是否已经有过半的服务器认可了当前的内部投票，如果确定已经有过半服务器认可了该投票，则终止投票。否则返回步骤4
   > 10. **更新服务器状态**。若已经确定可以终止投票，那么就开始更新服务器状态，服务器首选判断当前被过半服务器认可的投票所对应的Leader服务器是否是自己，若是自己，则将自己的服务器状态更新为LEADING，若不是，则根据具体情况来确定自己是FOLLOWING或是OBSERVING

### ZAB协议

拜占庭问题

paxos协议主要就是如何保证在分布式环网络环境下，各个服务器如何达成一致最终保证数据的一致性问题

ZAB协议是基于paxos协议的一个改进

zab协议为分布式协调服务zookeeper专门设计的一种支持崩溃恢复的原子广播协议

zookeeper并没有完全采用paxos算法， 而是采用zab Zookeeper atomic broadcast

#### zab协议原理

1. 在zookeeper 的主备模式下，通过zab协议来保证集群中各个副本数据的一致性

2. zookeeper使用的是单一的主进程来接收并处理所有的事务请求，并采用zab协议，把数据的状态变更以事务请求的形式广播到其他的节点

3. zab协议在主备模型架构中，保证了同一时刻只能有一个主进程来广播服务器的状态变更

4.  所有的事务请求必须由全局唯一的服务器来协调处理，这个的服务器叫leader，其他的叫follower

   leader节点主要负责把客户端的事务请求转化成一个事务提议（proposal），并分发给集群中的所有follower节点，再等待所有follower节点的反馈。一旦超过半数服务器进行了正确的反馈，那么leader就会commit这条消息

崩溃恢复

原子广播

### ZAB协议的工作原理

1. 什么情况下zab协议会进入崩溃恢复模式
   - 当服务器启动时
   - 当leader服务器出现网络中断、崩溃或者重启的情况
   - 集群中已经不存在过半的服务器与该leader保持正常通信
2. zab协议进入崩溃恢复模式会做什么
   - 当leader出现问题，zab协议进入崩溃恢复模式，并且选举出新的leader。当新的leader选举出来以后，如果集群中已经有过半机器完成了leader服务器的状态同（数据同步），退出崩溃恢复，进入消息广播模式
   - 当新的机器加入到集群中的时候，如果已经存在leader服务器，那么新加入的服务器就会自觉进入数据恢复模式，找到leader进行数据同步

### Watch原理

#### TODO

### 集群配置步骤

**1. 修改配置文件zoo.cfg**

```java
// 组成集群环境的每台机器都要感知其余机器的状态
// 格式：server.id=host:port:port
//	   id:机器的唯一表示，不可重复；取值范围：0～255；作用：用id来标志该机器在集群中的唯一机器序号
//	   2188:zookeeper的默认端口号；作用：leader和follower节点进行数据交换的端口
//     3181:表示leader选举的端口
// 如果指定机器为observer  需要在3181后面指定:observer  以及在zoo.cfg中加入 peerType=observer
server.1=172.16.93.101:2188:3181
server.2=172.16.93.102:2188:3181
server.3=172.16.93.103:2188:3181
```

**2. 创建myid文件**

在每一台服务器zookeeper文件下的zoo.cfg配置文件的dataDir属性对应的目录下创建一个myid的文件，内容为为台机器server对应的id

**3. 启动zookeeper**

### Zoo.cfg 配置文件

**tickTime=200**

​	zookeeper中最小的时间单位（ms）

**initLimit=10**

​	follwer节点与leader节点进行数据同步的最大时间（initLimit*tickTime）

**syncLimit=5**

​	leader节点与follower节点进行心跳检测的最大延迟时间  如果大于syncLimit*tickTime这个时间还没有回应，则表示这个follower节点不归leader节点监控了

**dataDir=/tmp/zookeeper**

​	表示zookeeper服务器存储快照文件的目录

**dataLogDir**

​	表示zookeeper存储事务日志的目录，默认是是在`dataDir`

**clientPort=2181**

​	表示客户端与服务端建立连接的端口号，默认。

### 数据模型

#### 简介

zookeeper的每一个数据节点成为一个`znode`，是zookeeper中最小的存储单元。每一个znode上都可以存放数据以及挂载节点，从而组成了一个层次化比较好的树形结构，最大长度Integer的Max单位，默认最大容量是3M

#### 节点类型

**持久化节点**

​	节点创建后会一直保存在zookeeper上，知道节点被删除，所有数据节点类型名称都是唯一的

**持久化有序节点**

​	每个持久化有序节点都会为它的一级子节点维护一个顺序

**临时节点**

​	临时节点的生命周期和客户端的会话保持一致，客户端会话失效，临时节点被清理

​	<font color="#FF0000">会话失效之后，zookeeper不会马上删除临时节点，而是有一个缓冲时间来检查</font>

**临时有序节点**

​	在临时节点上多了一个有序性

### zookeeper客户端基本命令

**create -s -e path data acl**

​	 -s 表示节点是否有序  `默认为无序节点`

​	-e  表示节点是否是临时节点  `默认持久化节点`

​	<font color="#FF0000">创建节点需要从根节点进行创建</font>

**get path [watch]**

​	获得path指定路径上的节点信息

**set path data [version]**

​	修改节点对应的data

​	[version] 

​		乐观锁概念

​		数据库里面会存有一个version字段去控制数据行的版本号

**delete path [version]**

​	删除置顶path上的节点

​	<font color="#FF0000">删除一个节点必须从最底部节点删除，不能删除目录下具有子级节点的节点路径</font>

#### Watcher

**zookeeper提供了分布式数据发布/订阅，zookeeper允许客户端向服务器端注册一个watcher监听。当服务器节点触发了指定事件的时候，服务器端会向客户端发送通知**

<font color="#FF0000">watcher通知是一次性的，一旦触发一次之后，该watcher信息失效，如果需要永久监听，则需要反复注册</font>

**事件**

>EventyType
>
>None 客户端与服务器端成功建立会话
>
>NodeCreated  节点创建
>
>NodeDeleted  节点删除
>
>NodeDataChanged 数据变更：数据内容
>
>NodeChildrenChanged 子节点发生变更： 子节点删除、新增的时候，才会触发

#### ACL

zookeeper提供了节点访问权限功能，用于有效的保证zookeeper节点中数据的安全性。避免误操作而产生的重大事故。

CREATE/READ/WRITE/DELETE/ADMIN  五种节点访问权限

#### Stat信息

```java
cversion = 0	// 子节点的版本号
dataVersion = 0 // 当前节点的数据版本号
aclVersion = 0	// acl的版本号，修改节点权限
    
cZxid = 0x300000004 // 节点被创建时的事务id
mZxid = 0x300000004 // 节点最后一次被更新的事务id
pZxid = 0x300000004 // 当前节点下的子节点最后一次被修改时的事务ID
    
ctime = Sat Dec 08 23:47:06 CST 2018 // 节点创建时间
mtime = Sat Dec 08 23:47:06 CST 2018 // 节点修改时间

// 创建临时节点的时候会有一个sessionid  该值存储的就是sessionid  其作用时客户端会话关闭时确认清除信息
ephemeralOwner = 0x0 // 临时节点示例  ephemeralOwner = 0x1678e4cca080001
dataLength = 3 // 当前节点存储数据长度
numChildren = 0 // 拥有的子级节点数量
```

### JAVA中API使用

**原生ZookeeperApi**

连接状态

`KeeperStat.Expired`  在一定时间内客户端没有收到服务器的通知， 则认为当前的会话已经过期了。

`KeeperStat.Disconnected`  断开连接的状态

`KeeperStat.SyncConnected`  客户端和服务器端在某一个节点上建立连接，并且完成一次version、zxid同步

`KeeperStat.authFailed`  授权失败

事件类型

`NodeCreated`  当节点被创建的时候，触发

`NodeChildrenChanged`  表示子节点被创建、被删除、子节点数据发生变化

`NodeDataChanged`    节点数据发生变化

`NodeDeleted`        节点被删除

`None`   客户端和服务器端连接状态发生变化的时候，事件类型就是None

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.8</version>
</dependency>
```

**zkClient**

```xml
<dependency>
    <groupId>com.101tec</groupId>
    <artifactId>zkclient</artifactId>
    <version>0.10</version>
</dependency>
```

**curator**

curator本身是Netflix公司开源的zookeeper客户端；

curator提供了各种应用场景的实现封装

curator-framework  提供了fluent（链式）风格api

curator-replice     提供了实现封装

<font color="#FF0000">事务控制，curator独有</font>

**curator连接的重试策略**

ExponentialBackoffRetry(间隔时间，重试次数)  衰减重试 

RetryNTimes 指定最大重试次数

RetryOneTime 仅重试一次

RetryUnitilElapsed 一直重试直到规定的时间

```xml
<dependency>
	<groupId>org.apache.curator</groupId>
	<artifactId>curator-framework</artifactId>
	<version>2.11.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>2.11.0</version>
</dependency>
```

### 数据存储

**内存数据**

1. 存储在内存中，使用ConcurrentHashMap<String, DataNode>为容器进行存储
2. zookeeper会定时把数据存储在磁盘上

**磁盘数据**

1. 查看事务日志的命令

   > java -cp :/mic/data/program/zookeeper-3.4.10/lib/slf4j-api-1.6.1.jar:/mic/data/program/zookeeper-3.4.10/zookeeper-3.4.10.jar org.apache.zookeeper.server.LogFormatter log.200000001

**Zookeeper有三种日志**

1. zookeeper.out -- 运行日志
2. 快照   存储某一时刻的全辆数据
3. 事务日志  事务操作的日志记录
   - 事务存储日志格式 -- log.zxid

### 应用场景

**数据发布订阅/配置中心**

> zookeeper采用的是推拉相结合的方式。 客户端向服务器端注册自己需要关注的节点。一旦节点数据发生变化，那么服务器端就会向客户端
>
> 发送watcher事件通知。客户端收到通知后，主动到服务器端获取更新后的数据
>
> 1. 数据量比较小
>
> 2. 数据内容在运行时会发生动态变更
>
> 3. 集群中的各个机器共享配置

​	watcher机制

​	统一配置管理（disconf）

**分布式锁**

> 不同进程之间，访问一个共享数据，会出现一种竞争的关系，分布式锁就是为了实现多进程间对于顺序的一种访问，防止多个进程同时对一个资源造成修改，导致数据读取不正确的一种方式。

​	**redis**

​		setNx：不存在返回0，存在返回1

​	**数据库**

​		1. 通过索引唯一的方式

​			创建一个表，增加一条记录，insert一条记录，根据索引唯一方式来进行独占，使用之后进行delete

​		2. 使用InnoDb的方式 for update使用行锁来进行独占

​		<font color="#FF0000">两种方式都不够友好，第一种需要考虑异常情况，第二种效率较低，不过实现方式简单</font>

​	**zookeeper**

​		排他锁（写锁）

​			假如想为多个客户端加入排他锁，一起创建一个节点，根据节点命名唯一特性，创建成功拥有锁

​		共享锁（读锁）

**负载均衡**

​	请求/数据根据规则分配到不同的数据单元上

**ID生成器**

**分布式队列**

**统一命名服务**

**master/leader选举**

### 问题总结

1. (Zookeeper)为什么zookeeper的节点数保持在一个奇数比较好？

   > 集群中只要有过半的机器是正常工作的，那么整个集群对外就是可用的。也就是说如果有2个zookeeper，那么只要有1个死了zookeeper就不能用了，因为1没有过半，所以2个zookeeper的死亡容忍度为0；同理，要是有3个zookeeper，一个死了，还剩下2个正常的，过半了，所以3个zookeeper的容忍度为1；同理你多列举几个：2->0;3->1;4->1;5->2;6->2会发现一个规律，2n和2n-1的容忍度是一样的，都是n-1，所以为了更加高效，何必增加那一个不必要的zookeeper呢
   >
   > [来源网址](https://www.cnblogs.com/zongyl/p/8058660.html)

2. (ZAB协议)假设一个事务在leader服务器被提交了，并且已经有过半的follower返回了ack。 在leader节点把commit消息发送给folower机器之前，leader服务器挂了怎么办？

   > 1. 选举新的leader（zxid的最大值，如果相等，会比较sid（myid）。详情请查看上文**Leader选举部分**）
   > 2. 同步数据给其他的folower以及observer
   >
   > 会将该事务丢弃，zab协议需要保证，在崩溃恢复过程中跳过那些已经被丢弃的事务
   >
   > zab协议，一定需要保证已经被leader执行同步后的事务也能够被所有follower提交

3. （Zookeeper）zookeeper集群什么情况下会触发leader选举（简略版）？

   > 1. 服务器初始化启动
   >
   >    > - (1)  **每个Server发出一个投票**。由于是初始情况，Server1和Server2都会将自己作为Leader服务器来进行投票，每次投票会包含所推举的服务器的myid和ZXID，使用(myid, ZXID)来表示，此时Server1的投票为(1, 0)，Server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。
   >    >
   >    > - (2)  **接受来自各个服务器的投票**。集群的每个服务器收到投票后，首先判断该投票的有效性，如检查是否是本轮投票、是否来自LOOKING状态的服务器。
   >    >
   >    > - (3)  **处理投票**。针对每一个投票，服务器都需要将别人的投票和自己的投票进行PK，PK规则如下
   >    >
   >    >   　　　　**· 优先检查ZXID**。ZXID比较大的服务器优先作为Leader。
   >    >
   >    >   　　　　**· 如果ZXID相同，那么就比较myid**。myid较大的服务器作为Leader服务器。
   >    >
   >    > - 对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的ZXID，均为0，再比较myid，此时Server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于Server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。
   >    >
   >    > - (4)  **统计投票**。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于Server1、Server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了Leader。
   >    >
   >    > - (5)  **改变服务器状态**。一旦确定了Leader，每个服务器就会更新自己的状态，如果是Follower，那么就变更为FOLLOWING，如果是Leader，就变更为LEADING。
   >
   > 2. 服务运行期间无法和leader保持连接
   >
   >    > - (1) **变更状态**。Leader挂后，余下的非Observer服务器都会讲自己的服务器状态变更为LOOKING，然后开始进入Leader选举过程。
   >    > - (2) **每个Server会发出一个投票**。在运行期间，每个服务器上的ZXID可能不同，此时假定Server1的ZXID为123，Server3的ZXID为122；在第一轮投票中，Server1和Server3都会投自己，产生投票(1, 123)，(3, 122)，然后各自将投票发送给集群中所有机器。
   >    > - (3) **接收来自各个服务器的投票**。与启动时过程相同。
   >    > - (4) **处理投票**。与启动时过程相同，此时，Server1将会成为Leader。
   >    > - (5) **统计投票**。与启动时过程相同。
   >    > - (6) **改变服务器的状态**。与启动时过程相同。

4. （逻辑时钟：logicalclock）什么情况下zookeeper集群各个节点的逻辑时钟会出现值不一致?

   > 举例，初始情况下5台机器，sid分别为1、2、3、4、5，逻辑时钟都是0。依次启动后，开始选举，所有的机器逻辑时钟自增为1。经过多次投票，假设第三台机器为leader，其他4台机器为follower，此时5台机器的逻辑时钟都为1
   >
   >  一般情况下，逻辑时钟应该都是相同的。但是，由于一些机器崩溃的问题，是可能出现逻辑时钟不一致的情况的。例如，上例中，sid=3的机器为leader。之后某一刻，sid为1、3的机器崩溃，zookeeper仍然可以正常对外提供服务。但需要重新选主，剩下的2、4、5重新投票选主，假设sid=5成为新的leader，逻辑时钟自增，由1变成2。之后某一刻，sid为5的机器奔溃，sid为1的机器复活，仍然有3台机器运行，zookeeper可以对外提供服务，但需要重新选主。重新选主，逻辑时钟自增，这时sid为2、4的机器的逻辑时钟是由2自增为3，而sid为1的机器的逻辑时钟是由1自增为2。这种情况下，就出现了逻辑时钟不一致的情况。这时，需要清楚sid为1的机器内部的投票数据，因为这些投票数据都是过时的数据

## 参考网址如下：

1. https://blog.csdn.net/gaoshan12345678910/article/details/67638657
2. https://blog.csdn.net/zh15732621679/article/details/80723358
3. 沽泡学院MIC老师讲解