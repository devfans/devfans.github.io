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

#### Redis Pub/Sub
基于发布订阅的redis pubsub功能提供了基于channel和pattern的事件订阅，可以作为即时消息发布，不适合作为队列。

#### ActiveMQ
