---
layout: post
title:  "Message Queue Introduction - Kafka"
author: devfans
categories: [ MessageQueue, Kafka ]
image: /static.livefeed.cn/static/blog/mq-app.png
tags: [sticky]
---
### 消息队列简介
消息队列的应用场景非常多，Kafka作为一款成熟的解决方案，在很多公司的业务中应用十分广泛。

#### 消息队列的主要应用场景

- 应用解耦
- 异步处理
- 流量削峰

说白了就是把需要处理的逻辑可以用消息队列把面向前台的业务解耦，从而隔断前端流量，添加缓冲区使业务得到异步延迟处理。


#### 常见消息中间件

![mq-vs](https://static.livefeed.cn/static/blog/mq-vs.png)

##### Redis List
Redis作为常用的内存服务器，支持多种数据类型，其中List可以用作常用简单的消息队列。由于Redis的数据操作具有原子性，可以同时可以有多个生产者和消费者操作同一个List。消费者消息处理失败后可以再次把消息放入queue中等待下次处理。

The max length of a list is 232 - 1 elements (4294967295, more than 4 billion of elements per list).

##### Redis Pub/Sub
基于发布订阅的redis pubsub功能提供了基于channel和pattern的事件订阅，可以作为即时消息发布或即时事件订阅，不适合作为队列。

##### ActiveMQ
ActiveMQ是一款老牌的消息队列，支持多种协议，由于ActiveMQ不适合超高吞吐量的业务，使用任务在逐渐减少

##### RabbitMQ
LinkedIn公司开发的RabbitMQ基于AMQP协议实现的消息队列也曾风靡一时，支持多种Exchange模式及延迟及死信功能，很强大。

##### Apache Kafka
消息队列中的骨干，Apache顶级项目，特别是在高吞吐量的业务中，支持分片，消息事务及流处理，在日志收集领域的独占鳌头。

##### RocketMQ
阿里的产品，很强大，基于Java开发，对常用消息队列功能做了很多优化。只是在其他公司不那么流行。

##### Apache Pulsar
Yahoo开发的消息队列，较Kafka有很多改善。很有前途。

#### 消息中间件常见问题

##### 消息有序性
先进先出，业务处理可能有优先级

##### 消息幂等性
无论同一消息处理重复几次，最终效果都一样

##### 消息事务性
消息流转可以做成事务，保持数据一致性

##### 吞吐量及延迟
吞吐量对业务很关键

##### 消息持久性（落盘）
重要消息要落盘，不能丢

### Kafka简介
2011 开源，Apache顶级项目，由Scala及Java开发。 主要概念有Broker，Topic, Partition, Replica, Producer, Consumer, ConsumerGroup, Offset, Coordinator, ISR等。

#### Kafka分区Leader逻辑
Kafka Broker中会选举一个作为Controller, 由Controller指定各个分区的Leader, 一般通过i % N（broker数量）确认Leader分区副本。每个分区的Follower副本会从Leader副本同步数据，同步较好的副本子集即为ISR。当Leader宕机时会从ISR选出一个作为新的Leader。副本中同步延迟较多的会被剔除。

#### Kafka 客户端

- 官方客户端Java
- librdkafka
- sarama

##### 消息生产入Queue
消息生产可以指定分区将消息入Queue，也可以采用一致的分区选择策略判定分区，一般是通过对传入的key作hash计算。同时生产者可以指定需要返回的acks, 比如 0 ：不需要确认， -1： Leader确认即可， 1: 所有ISR确认。同时如果idempotent开启，producer会对消息分配递增的seq no，用于确保消息幂等性，防止消息重复入queue，也可监测是否有消息丢失。另外，消息入Queue可以批量异步处理。

##### 消息消费
- 简单消息消费者可以指定topic分区和offset进行消费，offset由客户端维护。

- 消费者分组可以分配多个消费者接收同一topic的消息。客户端向组coordinator发送joinGroup请求时会触发group rebalance，coordinator指定的group leader会根据选定的balance策略进行分区分配。消费者组成员数不能多于分区数量。常用策略有 range和round robin。

- 触发group rebalance
  + joinGroup
  + leaveGroup
  + Coordinator宕机
  + Broker宕机
  + 分区改变
  
sarama 客户端消费者组样例：
```
	xlog.Infof("Starting consumer loop for %s", c.Name)
	c.Wg.Add(1)
	defer c.Wg.Done()

	g := cogroup.Start(sess.Context(), uint(c.Procs), uint(c.Batch), false)
	var last *sarama.ConsumerMessage

	done := false
	for {
		select {
		case msg, ok := <-ch:
			if ok {
				last = msg
				if g.Insert(func(ctx context.Context) error { return c.Handler(ctx, msg) }) {
					sess.MarkOffset(msg.Topic, msg.Partition, msg.Offset-int64(g.Size()), "")
				}
			} else {
				done = true
			}
		case <-c.sess.Done():
			xlog.Info("Consumer exits for session refresh")
			done = true
		}
		if done {
			break
		}
	}

	size := g.Wait()
	if last != nil {
		offset := last.Offset - int64(size)
		sess.MarkOffset(last.Topic, last.Partition, offset, "")
	}
	return nil


```
消费loop中不断从broker获取消息，之后放入task队列知道task队列满暂停获取新的消息。当队列有空间时继续获取。任务入queue后会进行一次任务执行进展检查，并前移offset标志位。
如果发生context取消，则放弃循环，等待正在执行的任务结束之后，进行最后的offset提交。
  
##### Kafka 消息事务性 
Kafka支持事务消息， 通过
  - initTransaction
  - beginTransaction
  - sendOffSetToTransaction
  - commitTransaction
  - send
  - abortTransaction
  可以执行事务性操作

#### Kafka主要缺憾

##### 需要预先规划分区，扩容时需要耗时复制分区数据
扩容手段就是加分区，加单机资源，单个消费者加多线程，或者采用临时Topic进行短时高流量扩容

##### Topic消费者数量不能多于分区数量
没办法

##### 消费者组分区Rebalance时会中断消费
设想50个分区消费者组入组的过程

##### 功能单一，无延迟队列等
延迟队列很常见

##### 不支持消息推送
不支持推，落后啊

