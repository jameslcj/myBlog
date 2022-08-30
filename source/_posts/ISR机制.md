---
title: ISR机制
date: 2022-03-01 21:14:47
tags: kafka
---
# 一、引言
前面我们把**LeaderAndIsrRequest**更新逻辑稍微讲了一下，看这名字叫**LeaderAndIsr**，但我们似乎只讲了**Leader**的情况， 一直没讲**ISR**的信息， 导致有些逻辑暂时无法理解， 所以接下来我们就一起了解一下**ISR的机制**。
# 二、 什么是ISR
要搞懂ISR更新机制， 我们需要先明白什么是ISR？
**ISR是insyncReplicas**的缩写， 也就是**同步中的副本**。那什么叫同步中？
要理解这个概念前， 我们先要明白Kafka定义的**LEO和HW**这2个概念；
**LEO(LogEndOffset)**：就是**每个分区副本消息数据的最大offset**；
**HW(HighWaterMark)**：是**用户读取到的分区最大位点**;
为什么需要这2个概念呢？ 由于follower副本是**定期向leader副本进行同步数据**， 所以同一时间内**， 各个副本同步到的offset不一定是一样的**， 也可以说**follower副本的offset<=leader副本的offset**。
拿之前的topic testA举个例子， 假设0号分区的leader副本在101号broker节点上，它的最大offset已经7了， 102的follower副本同步到的offset是6， 103的follower副本同步到的offset是5。
你觉得用户拉取到的最大offset是多少呢？如果能拉取到offset=7， 那么101宕机了，副本的leader被failover到其他节点上， **就会导致丢消息的现象**。
为了避免这类问题， kafka引入了LEO和HW的概念， 上面的例子， 分区的最大LEO=7，HW=5，用户最多能拉取到HW的offset=5的位点。

![image.png](https://img.alicdn.com/imgextra/i4/O1CN01KnvLup1qNinmEs3jM_!!6000000005484-2-tps-1480-1704.png)
**ISR就是来决定HW是否增长的，也就是这个ISR集合里的副本的最小LEO就是分区的HW。**

我们之前讲过**AR(Assigned Replicas)**，就是分区所有的副本，它又和ISR有什么区别的？

还是上面的例子，默认情况下， AR等于ISR，因此103副本的LEO就是分区的HW，假设103由于网络等原因， 短时间内不能向Leader拉取同步数据了，这种情况下， 101和102的LEO都会一直增加， 但分区的HW就不会增长， 这就会导致同步发送的请求最终都会超时失败。这种情况肯定不是我们想看到的， 为了解决这样的问题， Kafka引入了ISR的概念， 当一个副本节点在规定的时间下（[v2.5以前的版本默认是10s，之后的版本是30s](https://cwiki.apache.org/confluence/display/KAFKA/KIP-537%3A+Increase+default+zookeeper+session+timeout)）， 没有赶上leader的LEO的位点，就会被踢出ISR集合。
所以上面的例子， 最终会导致ISR集合中只有101和102，同时也从这两者之间的最小LEO来生成HW位点。


# 三、 更新副本状态
由于各follower副本都会定期向leader副本进行数据同步， leader会记录各个follower副本同步到的offset位置， 这样leader是知道各个follower副本的LEO，那么由leader副本来维护HW是最容易的事情了， **leader副本会记录follower每次来拉取的位点和当前分区的ISR， 来判断是否需要增加HW。**

我们来看下follower向leader拉取数据时， leader进行操作的具体代码实现：
![image.png](https://img.alicdn.com/imgextra/i4/O1CN0150dSbY25akIhuiLfa_!!6000000007543-2-tps-1118-156.png)
![image.png](https://img.alicdn.com/imgextra/i4/O1CN01Hckoy51KcJiUoeyjH_!!6000000001184-2-tps-1680-1248.png)

1. 记录副本之前的offset，用于后面判断位点是否增加
1. 更新副本的最新offset, 和拉取时间，这块我们等下单独讲
1. 如果不再isr集合里， 判断是否可以加入isr集合，这个我们等下单独讲
1. 判断是否可以触发分区HW增长
1. 如果HW增长， 判断是否完成发送和消费的异步任务

## **更新follower副本同步状态（updateFetchState）**
我们先来看下followerReplica.updateFetchState的逻辑
![image.png](https://img.alicdn.com/imgextra/i2/O1CN01ZoXrfO1pIOZud9f24_!!6000000005337-2-tps-2000-804.png)

1. 判断拉取的offset是否与当前leader的offset一样， 如果一样，说明leader没有更新新数据, 已经赶上leader的LEO，则更新_lastCaughtUpTimeMs；
1. 如果拉取的offset大于上次来拉取的leader的LEO，也说明赶上了leader的LEO， 因为在有数据发送的情况下， leader的LEO是一直在增加的， 所以按上次的结果来判断；
1. 记录来拉取的信息；
1.  记录本次拉取时， leader的LEO值，以便下轮判断；
1. 记录本次拉取时间，以便下轮判断；

可以看出来， 这块逻辑的核心，就是**维护_lastCaughtUpTimeMs的更新**；那么_lastCaughtUpTimeMs又是做什么的呢？他主要是来**判断follower副本是不是可以进入或者踢出ISR集合的**，我们先来看下加入isr的逻辑。


## **扩容ISR集合（maybeExpandIsr）	**
当发现当前来拉取的follower副本不在ISR集合中， 就会调用方法，它的具体逻辑如下：
![image.png](https://img.alicdn.com/imgextra/i1/O1CN01aJz91g1EfcsxF458p_!!6000000000379-2-tps-1204-332.png)
![image.png](https://img.alicdn.com/imgextra/i3/O1CN014xnvEL1c8hBcBeiMT_!!6000000003556-2-tps-1872-278.png)
![image.png](https://img.alicdn.com/imgextra/i4/O1CN01x9Oncq22vPJvSTaug_!!6000000007182-2-tps-1586-182.png)

1. 首先会判断是否符合isr集合的要求， 它的要求就是follower副本的LEO是否大于等于目前分区的HW；
1. 如果满足要求， 那么这个follower副本就会被加入到isr集合里；
1. 然后通过修改zookeeper节点， 通知controller，这块逻辑我们等下再讲；

看了上面的逻辑， 似乎加入isr的条件只和分区HW有关， 那之前说跟_lastCaughtUpTimeMs也有关系， 这是为什么呢？我们继续看HW增长逻辑maybeIncrementLeaderHW方法就会明白。

## **更新分区位点（maybeIncrementLeaderHW）**
![image.png](https://img.alicdn.com/imgextra/i2/O1CN019ggFF61QXAy1ayo3R_!!6000000001985-2-tps-1982-692.png)

1. 将leader的LEO作为默认的最小的LEO，也就是HW
1. 如果副本的LEO小于目前最小的LEO
1. 同时根据_**lastCaughtUpTimeMs**，判断在默认时间内（[v2.5以前的版本默认是10s，之后的版本是30s](https://cwiki.apache.org/confluence/display/KAFKA/KIP-537%3A+Increase+default+zookeeper+session+timeout)）赶上过leader的LEO, 或者在isr里的副本；
1. 满足上述条件， 则将当前的副本的LEO作为最小的LEO
1. 将找到的最小LEO更新为HW
1. HW变动就可以唤起发送和消费的异步线程了， 这块逻辑这里就不细讲了

我们之前讲过HW是由ISR集合副本的最小LEO决定的， 但发现实际代码会多一个根据_lastCaughtUpTimeMs判断规定时间内是否赶上过leaderLEO的逻辑， 这是为什么？
假设ISR集合里只有一个副本了， 也就是leader副本自身， 那么在这种情况下， leader的LEO就是分区的HW，如果这个分区一直有新进来的消息，HW一直在增加， 如果没有判断规定时间内赶上leaderLEO的逻辑， 那么其他follower副本永远赶不上HW，那么也加入不了ISR集合里了。

## **缩容ISR集合（maybeShrinkIsr）**
上面讲了加入ISR集合的逻辑，那么什么时候会被踢出ISR集合呢？

![image.png](https://img.alicdn.com/imgextra/i2/O1CN01cQCOC01gHdNgNVkTV_!!6000000004117-2-tps-1874-304.png)
![image.png](https://img.alicdn.com/imgextra/i2/O1CN01I3nF3n1RsX3IT69od_!!6000000002167-2-tps-1134-300.png)
![image.png](https://img.alicdn.com/imgextra/i3/O1CN01DGuDvy1KyIjPTfOpA_!!6000000001232-2-tps-1150-408.png)
ReplicaManager的调度器会定期（v2.5以前的版本默认是5s，之后的版本是15s）根据检测各个follower副本的**_lastCaughtUpTimeMs**， 如果超过规定时间（v2.5以前的版本默认是10s，之后的版本是30s）没有赶上过leaderLEO， 就会被踢出ISR集合，然后通知KafkaController。
接下来我们看下是如何通知KafkaController的。

我们先来看下更新isr的逻辑
![image.png](https://img.alicdn.com/imgextra/i3/O1CN018HCmb71xWbnURNyxF_!!6000000006451-2-tps-1908-404.png)
![image.png](https://img.alicdn.com/imgextra/i2/O1CN016knUvY1ieC7dm9H6Z_!!6000000004437-2-tps-1642-312.png)

1. ReplicaManager会直接将最新的isr集合更新到zookeeper的LeaderAndIsr节点上， 也就是讲得/brokers/topics/testA/partitions/0/state 这个节点上；
1. 如果更新成功，就把发生过isr变更的分区和时间记录下来，具体记录下来的信息如下；

![image.png](https://img.alicdn.com/imgextra/i4/O1CN01AEAnWQ1OQe5SKxHem_!!6000000001700-2-tps-1022-274.png)
看了上面的代码肯定会有很多疑问。

1. ReplicaManager居然直接修改LeaderAndIsr节点， 上面我们讲过这个节点，KafkaController也会进行修改， 那么必然会有**并发问题**，这问题咋解的呢？
   1. 细心的同学会发现**LeaderAndIsr**对象里有个**zkVersion**的属性， 它是zookeeper节点的**dataVersion**，这个值，是**单调递增**的， 每次更新节点就会自动递增1，**因此Kafka就将这个值作为乐观锁**，对LeaderAndIsr节点进行修改， 如果值符合传递进来的值就会修改成功并自动递增1，否则就会修改失败。
2. 即使能修改成功， 那么KafkaController又是怎么知道节点信息的变化的呢？这就和刚记录下来的信息有关了， 我们继续看下它是如何传递通知KafkaController。

## **通知KafkaController（maybePropagateIsrChanges）**
ReplicaManager的调度器每隔2500ms会调用如下方法，来通知KafkaController LeaderAndIsr节点有变化：
![image.png](https://img.alicdn.com/imgextra/i2/O1CN01P18CRi1mzxP5qVMqW_!!6000000005026-2-tps-1496-510.png)
为了避免在网络不稳定的时候， follower副本不停的加入isr和退出isr，导致频繁通知KafkaController，Kafka这里做了一个**延迟通知机制**，具体实现如下：

1. 判断上次isr发生变更时间是否超过5s，如果超过了才发送通知；
1. 既然有了上面的逻辑， 为什么还有下面的逻辑呢？假设isr一直在变化， 那么**lastIsrChangeMs**就会一直在变化，那么永远不会与当前时间相差超过5s然后发送通知， 为了避免这种“**饿死现象**”，kafka做了一个**兜底机制**，如果isr发生了，要保证60s至少要发送一次通知。


看了上面isr通知逻辑， 大家是不是觉得这块设计有一些缺陷？是的，这块设计确实存在很多问题：

1. 起初是isr频繁更新，导致KafkaController频繁变更状态， 后来加了这个传递延迟逻辑， 但就有可能**导致信息不一致问题**；
1. 因此在v2.5以后， 为了减少isr频繁变化的问题，[将isr赶上leaderLEO的默认时间从10s改到30s](https://cwiki.apache.org/confluence/display/KAFKA/KIP-537%3A+Increase+default+zookeeper+session+timeout)，虽然有所缓解问题， 但没彻底解决问题。

个人认为，ISR更新机制是Kafka状态机设计中问题较多的地方，毕竟核心问题还是多个对象操作同一个点， 必然会导致信息结果不一致的问题。这个问题在V2.2以前的版本尤其明显， 一旦zookeeper抖动， 就有可能导致信息不一致问题。

[所以在V2.7版本以后， leader副本不再直接修改LeaderAndIsr节点了， 而是发rpc通知KafkaController进行修改LeaderAndIsr](https://cwiki.apache.org/confluence/display/KAFKA/KIP-497%3A+Add+inter-broker+API+to+alter+ISR)，虽然解决了信息不一致的问题， 但也有可能遇到网络抖动的问题，请求未能及时发送出去导致isr不能及时更新的问题，但相对现在的问题， 我觉得还算是一个比较优的解了。


回到问题， 那么现在这版本KafkaController是如何知道ISR发生变更的呢？
![image.png](https://img.alicdn.com/imgextra/i1/O1CN01zAVOiR1NUwBZkvbj2_!!6000000001574-2-tps-1864-234.png)
![image.png](https://img.alicdn.com/imgextra/i1/O1CN01Q4tWE01jFICYutWJM_!!6000000004518-2-tps-898-240.png)
我们发现ReplicaManager会把发生变化的分区写入到**/isr_change_notification**节点的子节点上，记忆好的同学肯定还记得在**KafkaFailover**的时候， 曾经注册监听过这个handler，让我们看下它的具体逻辑吧。

## **KafkaController处理ISR变动（processIsrChangeNotification）**
![image.png](https://img.alicdn.com/imgextra/i3/O1CN011mzM4h1euRi9XhXOS_!!6000000003931-2-tps-1642-932.png)

1. 获取所有isr变化过的分区；
1. 将最新的LeaderAndIsr更新到**controllerContext.partitionLeadershipInfo**对象上；
1. 发送最新的**UpdateMetadata更新， 让所有节点更新metadata**；
1. 删除对应的isr通知节点

上面做的事情，**核心就是发送UpdateMetadataRequest请求， 让各节点的metadata信息变得一致**。

# 四、 总结
##  时序图
![](https://img.alicdn.com/imgextra/i3/O1CN014NqISu1Wh9HqGN01V_!!6000000002819-2-tps-2322-627.png)
## 流程图
![](https://img.alicdn.com/imgextra/i2/O1CN01wao1ya1ajfvwamiPB_!!6000000003366-2-tps-1880-1168.png)