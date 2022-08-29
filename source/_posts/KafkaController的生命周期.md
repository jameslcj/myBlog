---
title: KafkaController的生命周期
date: 2022-02-20 23:11:52
tags: kafka
---
# 一、 背景介绍
对Kafka了解的同学都知道，KafkaController与状态机犹如Kafka的“大脑”，时刻需要协调着整个Kafka集群的运转。因此它的整体设计是比较复杂的。之前对这块源码了解的也不是非常深入， 刚好前段时间有幸参与研发Kafka的Zookeeper迁移项目，借此机会对KafkaController与状态机的源码有了更深入的了解。 同时把这次的收获记录下来，结合线上遇到的真实案例与大家一起学习分享。
此次源码解析系列主要分成两部分。 第一部分主要依照KafkaV2.6版本的核心源码，简单介绍KafkaController与状态机的逻辑； 第二部分对照现有的KafkaController与状态机逻辑，简单介绍一下Kafka最新版3.2版本的KRaft状态机机制。
整体内容比较多， 会不定期更新。

# 二、 KafkaController
## **2.1 基础功能**
KafkaController它的主要功能是做什么的呢？顾名思义， 我们大概能猜出**KafkaController是Kafka集群的协调者**，他的主要工作是根据各broker节点的存活状态来**协调管理集群中的分区和副本的状态**。
举个例子，假设一个kafka集群， 拥有3个Broker节点， 同时创建一个topic叫topicA，它有1个分区， 每个分区存在3个副本。 在Controller的分配协调下，最终3个副本会被分配给各个Broker节点， 同时101上的副本被选为副本leader， 然后其他副本会启动额外线程定期从leader副本上拉取复制数据。
![](https://img.alicdn.com/imgextra/i3/O1CN01V8iRwS1h3ixv8xMzk_!!6000000004222-2-tps-2924-1344.png)
上面了解了一下Controller的一项基本工作，此时候你是不是会有很多疑问？3个副本是是怎么分配的？为什么101的副本作为leader？ 101宕机了怎么办？ 等等问题。。。回答这些问题前， 我们先弄清楚谁是Controller。
## **2.2 组件架构图**
首先我们通过一张组件架构图， 简单的了解一下KafkaController的组件架构全貌， 后续我们也会围绕这些组件和大家介绍KafkaController的相关源码。
![](https://img.alicdn.com/imgextra/i3/O1CN01KVhOzU20ZIyWUvTL1_!!6000000006863-2-tps-1500-1011.png)

## **2.3 竞选 Controller**
继续上面的问题， Kafka的Controller是谁呢？ 可以告诉大家的是， **Kafka的Controller是由自身的计算节点来充当的**， 也就是说上面例子中， 101-103都是潜在的Controller。 
那大家都有机会成为Controller， 那又是如何确定谁是Controller的呢？
这时候就需要提到Kafka额外依赖的组件 Zookeeper（[KRaft架构可不再依赖](https://cwiki.apache.org/confluence/display/KAFKA/KIP-500%3A+Replace+ZooKeeper+with+a+Self-Managed+Metadata+Quorum)）。
各个节点连接Zookeeper集群后，向**/controller**创建一个**EPHEMERAL类型的ZK节点**，一旦某个节点创建成功，那么这个节点就会被选为集群的Controller，后续来创建的节点都会失败， 但他们会监听这个节点的生命周期变化， 一旦发现这个节点被删除， 各个节点又会开始尝试创建， 以此来保证时刻有Controller存在。

EPHEMERAL类型的意思是： 这个节点的生命周期与这次的zookeeper连接的生命周期息息相关， 当此次的session连接断开时，被Zookeeper服务端检测到， 就会将这个节点删除

### **2.3.1 ControllerEventManager**
接下里我们看下源码是如何实现的。
![](https://img.alicdn.com/imgextra/i4/O1CN01ydLbsV25GasDXxa9h_!!6000000007499-2-tps-1500-666.png)
为了方便演示，部分示例中的代码会删除一些非核心逻辑，后续不再提示。

首先， 我们看到当KafkaServer启动后， 初始化kafkaController对象并调用其startup方法， startup方法里主要做的事情如下， 首先在zkClient上注册一个stateChagnehandler了， 这个我们稍微再讲， 我们先讲**EventManager**， 我们看到它向eventManager队列里插入了一个Startup类， 同时调用start启动eventManager。
顾名思义， EventManager是一个事件管理中心, 后续KafkaController相关事件处理， 都会被放到这里进行处理，他的实现也比较简单，就一个线程不停的从阻塞队列里拉取新进来的事件， 然后进行处理。 
**他的主要作用如下：**

1. 所有事件统一管控， 当发生controller发生切换后，** 可立即清除未处理完的消息， 避免产生脏数据**；
1. 所有事件异步化处理， 同时所有事件按**顺序执行，不存在并发问题**；

![](https://img.alicdn.com/imgextra/i1/O1CN01w2IQiV1cBtxSzqTpW_!!6000000003563-2-tps-1500-890.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01cKibQn20zrwcnZ2ke_!!6000000006921-2-tps-1500-1188.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01UiNdfe1ycqoIrT1WX_!!6000000006600-2-tps-760-208.png)
当eventManager 执行到**processStartup**对应的事件时, 方法里面的内容也很简单， 就做了两件事， **会触发elect进行Controller选举**（**registerZNodeChangeHandlerAndCheckExistence**这个我们稍微再讲，大家先记住这个方法）， 具体逻辑我们进一步看下。
### **2.3.2 选举核心逻辑**
![](https://img.alicdn.com/imgextra/i4/O1CN01BBZT4C1NwPwklgeJf_!!6000000001634-2-tps-1500-600.png)
![](https://img.alicdn.com/imgextra/i1/O1CN01hV4Qtp27dc0m8Q8Oo_!!6000000007820-2-tps-1500-445.png)
选举逻辑也非常清晰

1. 先从zk节点上查询**/controller**节点的值 ；
1. 如果存在，说明有其他节点已经提前成为Controller了， 则直接退出；
1. 如果不存在， 则先创建/controller类型为EPHEMERAL, 如果成功，再将/controller_epoch 递增+1；
1. 如果创建成功， 则会将**controller_epoch和epochZkVersion 记录下来， 这两个值比较重要， 大家先记住**， 后面会说明它的作用；
1. 最后执行**onControllerFailover**，开始成为Controller的一些初始化操作， 这块比较复杂， 我们后面单独来讲；

controller_epoch 是用来表示controller更新到哪一代了， 也就是一旦controller变化一次， controler_epoch就会递增+1， 这个值在后面用途非常重要， 大家重视一下；

epochZkVersion 是controller_epoch在zk节点上的dataVersion， 也就是说/controller_epoch节点每更新一次也会递增一次， 一般情况下此值和controller_epoch的一样的， 这个值在后面用途非常重要， 大家重视一下；

### **2.3.3 选举问题**
整体逻辑看起来不是非常复杂， 但大家有没有如下疑问：
假设controller节点创建成功， 但由于某些原因（比如网络抖动），导致修改controller_epoch失败了，会怎么样？

我们看上面代码， kafka使用了**Zookeeper的multi的api**， 这个api有什么特性呢？
当任何一步操作失败了，都会导致之前的操作回滚，zookeeper的事务功能就是使用这个来实现的。

**因此当controller_epoch修改失败， 也会导致此次竞选controller失败。**
一旦elect竞选成功， KafkaController**就开始触发failover进行初始化**。failover的逻辑比较多，我们稍后单独来讲， 我们先把Controller完整生命周期讲完。接下来我们讲下卸载Controller相关的逻辑。

## **2.4 卸任 Controller**
卸任Controller的方式，主要分为两种：

1. 一种是**主动触发：删除zookeeper的/controller节点触发；**
1. 一种是**被动触发： zookeeper的session过期触发；**

当KafkaController卸任时， 就会触发新一轮Controller竞选。 那这里就有几个问题：

1. 当前的KafkaController是怎么知道自己要进行“退休并办理交接手续”的呢？
1. 其他节点是怎么知道Controller已经退位， 要开始新一轮竞选Controller的呢？
### **2.4.1 主动触发**
带着上面两个问题， 我们回到源码找答案。
![](https://img.alicdn.com/imgextra/i2/O1CN01SgQXRg1ebfTNIveDz_!!6000000003890-2-tps-1322-202.png)
![](https://img.alicdn.com/imgextra/i1/O1CN01prPy4o1GVD6sR4wzf_!!6000000000627-2-tps-1500-304.png)
刚刚我们研究**processStartup**方法时， 我们忽略了一下方法 **registerZNodeChangeHandlerAndCheckExistence**，它的**作用就是在zookeeper上注册监听/controller的生命周期**，同时传递进去**ControllerChangeHandler回调对象**， 当/controller的生命周期一旦发生变化， 就会回调ControllerChangeHandler对象里对应的方法，然后把需要做的事情放到我们上面讲过的Eventmanger对象里异步执行。
其实**ControllerChangeHandler**对象里做的事情也比较简单:

1. 当/controller节点被主动删除时， 就触发elect竞选逻辑， 就是我们上面讲得竞选逻辑。
1. 当/controller节点被创建或者数据发生改变时， 就判断自己还是不是KafkaController， 如果不是了就调用onControllerResignation卸任Controller。 onControllerResignation的逻辑我们后面再讲。

![](https://img.alicdn.com/imgextra/i3/O1CN01pM1E901NrNajNO3uR_!!6000000001623-2-tps-880-132.png)
![](https://img.alicdn.com/imgextra/i2/O1CN017sdDrk1rrmPSryAdY_!!6000000005685-2-tps-1324-352.png)
#### **Zookeeper监听机制**
注意一个细节， 在上面的**maybeResign**方法里， 我们发现程序又调用了一次**registerZNodeChangeHandlerAndCheckExistence**，在zookeeper再次注册监听，那么问题来了， 当zookeeper发生相应事件时， 会不会被**多次调用**呢？这肯定不是我们预期的， 那么KafkaController是如何避免这类问题呢？
![](https://img.alicdn.com/imgextra/i3/O1CN01zzKJYC1wPuO8yNVBq_!!6000000006301-2-tps-638-280.png)
![](https://img.alicdn.com/imgextra/i1/O1CN01OCO21c1O8KFDAQ90b_!!6000000001660-2-tps-1338-146.png)
我们看到kafka不是直接在zookeeper上注册对应事件的， 而是内部维护了一个叫**zNodeChangeHandlers**的map对象， 他的key为ZNodeChangeHandler接口的path， value为具体的ZNodeChangeHandler对象。在回到上面看ControllerChangeHandler对象， 它是实现了ZNodeChangeHandler接口， 因此它的key就是ControllerZNode.path， 也就是/controller，**因此多次注册， 也会以最后一次调用为准。**
那它又是如何与zookeeper关联起来的呢？
![](https://img.alicdn.com/imgextra/i1/O1CN01XoCxuQ27ardMlbSyz_!!6000000007814-2-tps-1500-877.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01f5Bq7C1aGoyjZ3uP1_!!6000000003303-2-tps-1500-79.png)
![](https://img.alicdn.com/imgextra/i2/O1CN01rtul6Y1rHb7XCMkzu_!!6000000005606-2-tps-1500-472.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01PxbvjF1YYZ5t4sABm_!!6000000003071-2-tps-1500-135.png)
Kafka这块实现还是比较精巧， 它在**初始化zookeeper对象时**，传递进去一个叫**ZooKeeperClientWatcher**对象， **后续所有监听事件都由这个对象统一管理，这样减小了代码的复杂度和额外的内存开销。**
然后在调用exists、getData、getChildren时，判断**zNodeChangeHandlers或zNodeChildChangeHandlers**这2个map对象里是否有对应的key， 来决定是否监听对应zookeeper节点的生命周期变化。
当zookeeper发生监听的事件时，在从zNodeChangeHandlers或zNodeChildChangeHandlers的map对象上判断这个key是否存在， 存在就调用相应方法。
以上这些设计， 就大大降低了监听事件的整体复杂度。

zNodeChildChangeHandlers 是用来监听zookeeper节点的子节点数量变化情况

#### **整体调用链路**
至此Controller的整体调用链路我们已经非常清楚了， 就是ZookeeperWatch触发调用各种监听Handlers，然后放入ControllerEventManager的队列里， 按进队顺序执行，最后操作zk或者改变状态机。
![](https://img.alicdn.com/imgextra/i3/O1CN01qRsiOS1fqc0oEghPu_!!6000000004058-2-tps-1500-1377.png)
### **2.4.2 被动触发**
上面讲了当/controller节点被主动删除而引起的卸任。那么被动触发是怎么引起的？
一般情况下，是由于**网络抖动**， zookeeper的客户端在规定的时间内（[V2.5以前的版本默认为6s，V2.5后的版本18s](https://cwiki.apache.org/confluence/display/KAFKA/KIP-537%3A+Increase+default+zookeeper+session+timeout)）未向服务端成功发送**心跳**，当网络恢复后，客户端再次向服务端发送心跳时，服务端会判断此客户端**session已过期**， 会向客户端发送**session expired 事件**。最终会调用**reinitialize**方法。
![](https://img.alicdn.com/imgextra/i3/O1CN01N6kCdF1FbKn7AHmd7_!!6000000000505-2-tps-1150-610.png)
![](https://img.alicdn.com/imgextra/i1/O1CN01OKN9WQ1GiwzStFaPL_!!6000000000657-2-tps-1188-252.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01hG8P3a1brkWzvGTik_!!6000000003519-2-tps-1500-959.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01g9jx3j1oTYcL6ClhQ_!!6000000005226-2-tps-1500-564.png)
查看**reinitialize**方法， 我们发现它会回调**stateChangeHandlers**的方法， 这是什么东西呢？
我们把代码回到前面我们讲过得**KafkaController.startup**的地方，KafkaController类刚启动的时候会注册这个**stateChangeHandler**。
![](https://img.alicdn.com/imgextra/i1/O1CN01VL3esj1Qhi4nBEu8F_!!6000000002008-2-tps-1500-663.png)
因此reinitialize方法， 会调调用这个state的**beforeInitializingSession**方法， 也就是先把未执行的任务清楚掉，再把**Expire**类放入eventManager里进行调用， 并且阻塞线程， 直到Expire执行完毕。
Expire逻辑也非常简单， 就是调用**onControllerResignation**, 进行卸任Controller。
![](https://img.alicdn.com/imgextra/i2/O1CN01eJ8JFt1REgapVtVDM_!!6000000002080-2-tps-948-182.png)
卸任完毕后， 会初始一个新的Zookeeper对象，然后回调**afterInitializingSession**方法。
**afterInitializingSession**里面方法逻辑也非常简单:

1. 因为之前的zookeeper已经session过期， 所以需要**重新创建/brokers/ids/[101,102,103], 告知Controler自己的存在**；
1. **processReelect**操作，就是**挂载监听/controller节点变化**，同时尝试竞选KafkaController， 和之前的逻辑基本一样， 这里不再复述；

/brokers/ids/xxx 这个节点和/controller节点一样，都是EPHEMERAL类型节点， 它的生命周期和broker节点的生命周期一样， 也就是说broker节点宕机这个节点就会被删除; 节点上线， 这个节点就会被创建，KafkaController也通过这个zookeeper节点来判断对应的broker节点是否存活
#### **Session 过期问题**
**注册/brokers/ids** 这里会有个**session过期问题**需要说下。就是当创建/brokers/ids/[101,102,103] 这个节点时，发现节点还存在，kafka会这么做？
我们先来看下**0.10.2版本**的逻辑。
![](https://img.alicdn.com/imgextra/i2/O1CN0155Wlsf1UGSyqo36IG_!!6000000002490-2-tps-1500-598.png)
0.10.2版本， 发现节点存在， **就会直接抛出异常**， 不再做其他事情了。大家先想想这样做是否合理？
看到这里， 大家会有疑问，刚上面说了这个节点是EPHEMERAL节点，也就说，刚刚zookeeper客户端重新初始化了， 之前的sessionId应该已经过期了， 那么对应的节点应该被删除了， 为何还存在呢？
一般情况下，是不会出现上面的问题的， 但当一个大zookeeper集群的网络出现问题时， 就有可能出现上面这问题， **当网络出现问题时， 整个集群的客户端都因为心跳不可达而过期，但是由于集群过期的节点非常多， zookeeper服务端还没来得及删除所有的过期节点时， 网络恢复了， 然后此时去创建这个节点， 就会发现这个节点存在**。过了一会， 这个节点就会被zookeeper删除， 但由于**sessionId不一样， broker是无法感知到的，那么就会有可能导致各节点的metadata信息不一致的问题。**

我们之前就遇到过ZK网络抖动导致的这类问题，所有还在用0.10.2版本的同学尽快升级了。

我们再来看下**2.x的版本**是如何实现的。

![](https://img.alicdn.com/imgextra/i2/O1CN01unc1BF1CAp0xRof1T_!!6000000000041-2-tps-1500-580.png)
![](https://img.alicdn.com/imgextra/i4/O1CN015Wi9OY1dWLFcIRdre_!!6000000003743-2-tps-1500-439.png)
2.x的版本这块逻辑会比0.10.2版本稍微复杂有点， 当创建节点时， 发现节点存在, 会先进行判断这个节点的**ephemeralOwnerId**是否与当前sessionId一样， 如果不一样， 就需要重新创建。
Zookeeper 的EPHEMERAL节点的ephemeralOwnerId 等于创建他时连接的sessionId，用这个来关联这个zookeeper节点的生命周期， 也就是说， 对应的sesionId的连接过期了， 这个节点就会被自动删除了。
#### **脑裂问题**
我看到有些文章里说， 自从KafkaController有了ControllerEventManager后， 依靠单向队列的方式，可以防止Controller“脑裂事件”发生；

Controller脑裂是指集群中同时存在2个及以上controller；

他们依据是当发生session过期时，**Controller 执行卸任前**，会先将事件队列里的所有事件清空不再执行， 防止与新Controller同时执行发生脑裂。

这个优化确实比老版本好很多， 同时防止了很多脑裂事件，但是在某些情况下，它还是无法防止脑裂问题。

大家先想想，会在什么情况下，它是无法防止脑裂问题的？
![](https://img.alicdn.com/imgextra/i1/O1CN01A7y9th1Z2Kq6aHRab_!!6000000003136-2-tps-1500-678.png)
以下两种情况是会导致脑裂问题的：

1. 已经在执行的时间， ControllerEventManager是没法让其停止的；
1. 网络抖动导致的并发问题：我们前面说过ZK上的** /controller节点和/brokers/ids/[101,102,103]**这2个节点都是跟session有关的，一旦**网络抖动导致session过期**， 就有可能触发断开事件。在网络不佳的状态下， 这2个事件触发顺序是不一定的， 如果/brokers/ids/xxx断开的事件先触发， 那么KafkaController会执行**Broker下线的逻辑**（这个逻辑我们后续再讲），如果在**topic分区比较多的情况**，这个下线逻辑是非常耗时的， 比如一个实例他有接近上万个分区， 每次broker下线逻辑要走2分钟之久。 同时Event队列每次只能执行一个事件， 因此在这个Broker下线事件执行完之前， 是无法执行Controller卸任逻辑的。但是别的Broker节点也许网络是正常的， 因此是可以感知/controller节点被删除，就会触发成为Controller，执行Controller相关逻辑， 在这个阶段整个集群就会存在两个Controller。

在这个阶段，由于集群同时存在2个Controller，会导致各个节点的metadata信息不一致， 有可能会影响整个集群的isr扩缩容。

**那针对这种脑裂问题， Kafka是如何避免的呢？**
我们先来想下， 为什么会有2个controller，核心就是老的controller不知道已经有新的controller诞生了，因此只需让他知道有新的controller诞生了，自己不要做controller相关的事情就ok了。
那老的controller怎么知道有新的controller诞生了呢？这就回到我们前面说过的**/controller_epoch ZK节点**，它表示controller的代数，每次有新的controller诞生， 它就会**递增**。
**同时ZooKeeper是Kafka持久化状态机的对象， 每次状态机变动前，都会将最新的状态机持久化到Zookeeper上。**
因此利用Zookeeper事务multiApi，在所有对Zookeeper操作前绑定一个**检测/controller_epoch的zkVersion，如果zkVersion与自己的不一样就会抛出异常，进行卸任Controller。**
这个问题是在V2.2.0版本修复的： [https://issues.apache.org/jira/browse/KAFKA-7235](https://issues.apache.org/jira/browse/KAFKA-7235)。还是那句老话**还在用0.x 的用户赶紧升级吧**。
![](https://img.alicdn.com/imgextra/i1/O1CN01DlgF4t1tdF4JhPKhM_!!6000000005924-2-tps-1500-384.png)
![](https://img.alicdn.com/imgextra/i3/O1CN01jZB1yk1Gn4YekSH7G_!!6000000000666-2-tps-1426-634.png)
![](https://img.alicdn.com/imgextra/i2/O1CN01NUt6ah1K3y0rVOAxI_!!6000000001109-2-tps-1384-508.png)


## **2.5 ControllerFailover**
上面详细介绍了KafkaController的竞选和卸任基本流程。现在再来简单讲下节点成为Controller后， 做的**onControllerFailover**操作。
### **2.5.1 监听Zookeeper状态变化**
![](https://img.alicdn.com/imgextra/i4/O1CN01Q9VbbD1CBHPcKSCsh_!!6000000000042-2-tps-1500-304.png)
failover第一步操作就是**监听各个zookeeper节点的变化**， 下面我先简单说下他们的作用：

- 子节点变化：
- brokerChangeHandler： 当各个broker节点发生上下线时， 会触发调用这个方法； 
- topicChagneHandler: 当有新建的topic时， 会触发调用这个方法
- topicDeletionHandler: 当需要删除某个topic时， 会调用这个这个方法
- logDirEventNotificationHandler: 当某块磁盘损耗， 会调用这个方法
- isrChangeNotificationHandler: 某个分区的isr发生变化时， 会调用这个方法
- 数据变化：
- preferredReplicaElectionHandler: 当需要优化副本leader时，会调用这个方法
- partitionReassignmentHandler: 当需要迁移副本时， 会调用这个方法

大家先大概记住这些方法， 我们后续讲状态机的时候， 会再结合这些方法精讲细节逻辑。
### **2.5.2 初始化上下文**
```scala
protected def initializeControllerContext(): Unit = {
  val curBrokerAndEpochs = zkClient.getAllBrokerAndEpochsInCluster
  // 1. 先从zookeeper上获取所有broker节点的接入点信息,  存在controllerContext.liveBrokers对象上， 用来表示所有的broker节点的存活状态
  controllerContext.setLiveBrokers(curBrokerAndEpochs)
  // 2. 再从zookeeper上获取所有topic信息, 存入controllerContext.alltopics对象上，后续可以根据这个信息， 判断topic是新增还是删除；  
  controllerContext.setAllTopics(zkClient.getAllTopicsInCluster(true))
  zkClient.getFullReplicaAssignmentForTopics(controllerContext.allTopics.toSet).foreach {
      case (topicPartition, replicaAssignment) =>
      // 3. 获取所有Topic的分区AR信息，存在controllerContext.partitionAssignments对象上， 也就是说topic每个分区的副本分布情况
        controllerContext.updatePartitionFullReplicaAssignment(topicPartition, replicaAssignment)
        if (replicaAssignment.isBeingReassigned)
      // 4. 获取正在迁移副本的分区信息，存入controllerContext.partitionsBeingReassigned上，后续迁移副本都会依据这个对象
          controllerContext.partitionsBeingReassigned.add(topicPartition)
  }
  // 5. 监听这些topic的zookeeper接上的数据变化情况， 主要作用是， 当topic新增分区时，controller能够知道
  registerPartitionModificationsHandlers(controllerContext.allTopics.toSeq)
  // 6. 获取所有topic的当前副本的状态机分配情况，存在controllerContext.partitionLeadershipInfo对象里
  updateLeaderAndIsrCache()
  // 7. 启动channel通信线程， 它的主要作用是KafkaController与其他节点进行通信
  controllerChannelManager.startup()
  }

private def updateLeaderAndIsrCache(partitions: Seq[TopicPartition] = controllerContext.allPartitions.toSeq): Unit = {
    val leaderIsrAndControllerEpochs = zkClient.getTopicPartitionStates(partitions)
    leaderIsrAndControllerEpochs.foreach { case (partition, leaderIsrAndControllerEpoch) =>
      controllerContext.partitionLeadershipInfo.put(partition, leaderIsrAndControllerEpoch)
    }
}
```
这部操作是**初始化controllerContext**， 看名字就知道这个对象非常重要，你可以理解controller的所有操作结果的上下文都存在这里， 我们后续操作状态机信息， 强依赖这个对象。
我将代码精简了一下， 它的主要操作如下：

1. 先从zookeeper上获取所有broker节点的接入点信息, 存在**controllerContext.liveBrokers对象上**， 用来表**示所有的broker节点的存活状态；**
1. 再从zookeeper**上获取所有topic信息, 存入controllerContext.alltopics对象上**，后续可以根据这个信息， 判断topic是新增还是删除；
1. 获取所有**Topic的分区AR(Assigned Replicas)信息，存在controllerContext.partitionAssignments对象上**， 也就是说topic每个分区的副本分布情况；
1. 获取正在**迁移副本的分区信息**，存到**controllerContext.partitionsBeingReassigned**上，后续迁移副本都会依据这个对象；
1. 监听这些topic的zookeeper节点的数据变化情况， 主要作用是， **当topic新增分区时**，controller能够知道，并做相应处理；
1. 获取所有topic的**当前副本的状态机分配情况，存在controllerContext.partitionLeadershipInfo对象里**；
1. 启动**channel通信线程**， 它的主要作用是**KafkaController与其他节点进行通信**；

大家先简单了解一下，不要完全了解细节， 主要是对这几个对象要印象深刻，后续状态机部分会和这些对象息息相关：

- **controllerContext.liveBrokers**
- **controllerContext.alltopics**
- **controllerContext.partitionAssignments**
- **controllerContext.partitionsBeingReassigned**
- **controllerContext.partitionLeadershipInfo**
```scala
val (topicsToBeDeleted, topicsIneligibleForDeletion) = fetchTopicDeletionsInProgress()
    topicDeletionManager.init(topicsToBeDeleted, topicsIneligibleForDeletion)

    info("Sending update metadata request")
    sendUpdateMetadataRequest(controllerContext.liveOrShuttingDownBrokerIds.toSeq, Set.empty)

    replicaStateMachine.startup()
    partitionStateMachine.startup()

    info(s"Ready to serve as the new controller with epoch $epoch")

    initializePartitionReassignments()
    topicDeletionManager.tryTopicDeletion()
    val pendingPreferredReplicaElections = fetchPendingPreferredReplicaElections()
    onReplicaElection(pendingPreferredReplicaElections, ElectionType.PREFERRED, ZkTriggered)
    info("Starting the controller scheduler")
    kafkaScheduler.startup()
    if (config.autoLeaderRebalanceEnable) {
      scheduleAutoLeaderRebalanceTask(delay = 5, unit = TimeUnit.SECONDS)
    }
```
### **2.5.3 剩余流程**
![](https://img.alicdn.com/imgextra/i3/O1CN01gYkGcN1cFZ7x2GD6i_!!6000000003571-2-tps-1494-980.png)
剩余的流程， 主要都是初始化各个管理器：

1. **初始化删除topic管理器**， 后续的topic流程， 都由它管控；
1. 启动**副本状态机管理器**，这个对象非常重要， 所有副本的状态都由它来控制；
1. 启动**分区状态机管理器**，这个对象非常重要， 所有副本的leader都由它控制；
1. 初始副本迁移流程， 之前没有迁移完毕的流程， 从这恢复开始；
1. 重新触发未删除的topic的流程，**因此我们经常删除topic失败， 重选controller就能删除成功**， 其实关键步骤就在这里， 这里会重新开始删除；
1. 根据分区的AR分配情况， **优化各节点leader分配情况**， 这个后面再讲；
1. 启动调度器， 后续KafkaController的调度任务都由它来完成；
1. 定时调度优化各节点的leader分区情况；

上面这些对象都是非常重要的对象， 后面状态机部分，我们再细讲他们的功能， 先有个印象吧。
## **2.6 ControllerResignation**
```scala
private def onControllerResignation(): Unit = {
    debug("Resigning")
    // de-register listeners
    zkClient.unregisterZNodeChildChangeHandler(isrChangeNotificationHandler.path)
    zkClient.unregisterZNodeChangeHandler(partitionReassignmentHandler.path)
    zkClient.unregisterZNodeChangeHandler(preferredReplicaElectionHandler.path)
    zkClient.unregisterZNodeChildChangeHandler(logDirEventNotificationHandler.path)
    unregisterBrokerModificationsHandler(brokerModificationsHandlers.keySet)

    // shutdown leader rebalance scheduler
    kafkaScheduler.shutdown()
    offlinePartitionCount = 0
    preferredReplicaImbalanceCount = 0
    globalTopicCount = 0
    globalPartitionCount = 0
    topicsToDeleteCount = 0
    replicasToDeleteCount = 0
    ineligibleTopicsToDeleteCount = 0
    ineligibleReplicasToDeleteCount = 0

    // stop token expiry check scheduler
    if (tokenCleanScheduler.isStarted)
      tokenCleanScheduler.shutdown()

    // de-register partition ISR listener for on-going partition reassignment task
    unregisterPartitionReassignmentIsrChangeHandlers()
    // shutdown partition state machine
    partitionStateMachine.shutdown()
    zkClient.unregisterZNodeChildChangeHandler(topicChangeHandler.path)
    unregisterPartitionModificationsHandlers(partitionModificationsHandlers.keys.toSeq)
    zkClient.unregisterZNodeChildChangeHandler(topicDeletionHandler.path)
    // shutdown replica state machine
    replicaStateMachine.shutdown()
    zkClient.unregisterZNodeChildChangeHandler(brokerChangeHandler.path)

    controllerChannelManager.shutdown()
    controllerContext.resetContext()

    info("Resigned")
  }
```
**卸任Controller**具体逻辑我就不细讲了， 其实看代码就知道， 它和**ControllerFailover**是对应的， 就是将之前初始的内容摘除，防止与新Controller之间存在**“脑裂情况”和内存泄露**等。

# 三、总结
上面主要细讲了Controller的竞选和卸任的逻辑， failover和resignation只是稍微提了一下， 主要是让大家对一些对象混个眼熟， 具体细节我们在状态机部分细讲， 这样大家理论起来更加方便。

大家可以看下下面的时序图和流程图， 来回顾一下上面讲得内容。

## 3.1 时序图
![](https://img.alicdn.com/imgextra/i1/O1CN016rnPKj1go9UrZwksi_!!6000000004188-2-tps-1782-611.png)
## 3.2 流程图
![image.png](https://img.alicdn.com/imgextra/i3/O1CN01bkl04P1OzSBu38lvO_!!6000000001776-2-tps-1500-959.png)