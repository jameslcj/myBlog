---
title: FastJSON Key Hash冲突问题
date: 2021-10-20 22:41:55
tags: JAVA
---
# 一、背景介绍
背景需求是这样的， 需要将下面的json字符串转换成TopicOutputItem对象，对象的字段在字符串里都是有的， 但是解析后， 发现其他字段都可以成功获取， 唯独topic字段是空的。
```
{"__topic__":"","outputReqs":"0.0","__tag__:__receive_time__":"1644412388","__tag__:__user_defined_id__":"xxx","outputBytes":"0.0","userId":"xxxx","outputConvs":"0.0","instanceId":"xxx","__tag__:__hostname__":"xxx","name":"topicOutputItem","outputFailedReqs":"0.0","topic":"test","time":"1644412385458","logtime":1644412385,"__tag__:__path__":"","__tag__:__pack_id__":""}
```
```
class TopicOutputItem {
    private String topic;
    private double outputBytes;
    private double outputReqs;
    private String name;
    private String instanceId;
    private String userId;
    private Long time;
}
```
# 二、问题分析
这个字符串数据是从sls上获取的， 原本json字符串只有我对象的字段， 但从sls获取时， 被加上了很多sls内部字段， 都是以__开头， 比如__topic__。刚好此值是空的， 因此我把这个值手动赋值后， 居然能成功映射到对象的topic上， 因此断定fastJson解析逻辑出了问题， 因此顺着fastJson源码找了一下原因。

## 解析流程
首先了解一下fastJson解析的大致流程，fastJson的代码比较冗长， 但主要分为3个步骤
1. 通过ASM获取我们的对象的字节码
2.  然后按顺序匹配json字符串
![image.png](https://img.alicdn.com/imgextra/i3/O1CN01c87GSQ1XgykoBSkk2_!!6000000002954-2-tps-1500-495.png)
![image.png](https://img.alicdn.com/imgextra/i3/O1CN01rXk4hy1CBjnmBKGIk_!!6000000000043-2-tps-1500-437.png)
3. 将从字符串上获取的key和value，赋值到对象上
![image.png](https://img.alicdn.com/imgextra/i3/O1CN01KMixU71o9PBZndNfK_!!6000000005182-2-tps-976-670.png)
所以问题应该出在匹配json字符串上。

## 匹配逻辑：
1. fastJson会先获取json字符串的第一个key， 就是__topic__，然后与asm字节码对象匹配是否有对应的key
2. 发现所有对象都不匹配，fastJson默认会使用smartMatch方式再进行匹配key
3. smartMatch方式会使用key的hash进行匹配，理论上topic和__topic__不会发生hash冲突， 但是这个hash获取方式比较奇怪， 会过滤掉"_"， 因此导致这两者的hash会一致。
4. 一旦获取到值后， 即使后面再匹配上topic，也会被跳过，因此最终导致数据错了。

# 三、解决方法
## 方案一
通过传递 Feature.DisableFieldSmartMatch 关闭smartMatch功能。
这种方案， 不敢全局配置， 只能每次调用解析时进行关闭，只能暂时解决我遇到的问题。


## 方案二
提前过滤掉多余字符字段， 或是将顺序调整至后面。
咨询了一下sls的同学， 这些字段无法自动去除， 只能自己手动去除。
