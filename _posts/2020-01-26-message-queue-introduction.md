---
layout: post
title:  "Message Queue Introduction - Kafka"
author: devfans
categories: [ MessageQueue, Kafka ]
image: /static.livefeed.cn/static/blog/communication-connection-data-162622-min.jpg
tags: [featured]
---
消息队列的应用场景非常多，Kafka作为一款成熟的解决方案，在很多公司的业务中应用十分广泛。

#### 消息队列的主要应用场景

- 应用解耦
- 异步处理
- 流量削峰

说白了就是把需要处理的逻辑可以用消息队列把面向前台的业务解耦，从而隔断前端流量，添加缓冲区使业务得到异步延迟处理。

/图 

#### 常见消息中间件

##### Redis List
Redis作为常用的内存服务器，支持多种数据类型，其中List可以用作常用简单的消息队列。由于Redis的数据操作具有原子性，可以同时可以有多个生产者和消费者操作同一个List。消费者消息处理失败后可以再次把消息放入queue中等待下次处理。

The max length of a list is 232 - 1 elements (4294967295, more than 4 billion of elements per list).

##### Redis Pub/Sub
基于发布订阅的redis pubsub功能提供了基于channel和pattern的事件订阅，可以作为即时消息发布，不适合作为队列。

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

##### 消息幂等性

##### 消息事务性

##### 吞吐量及延迟

##### 消息持久性（落盘）

