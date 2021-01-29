---
layout: post
title:  "Create A Golang Coroutine Group 手撸Golang协程并发器"
author: devfans
categories: [ errgroup, golang, coroutine ]
image: /static.livefeed.cn/static/blog/mq-app.png
tags: [sticky]
---

业务中经常要处理一批任务需要多线程处理，这时如何有效的控制任务执行细节和线程并发就变的很重要。因为大家都造了很多轮子，来，我也造一个！

### 并发控制器主要思考点

#### 任务是否需要有序执行
#### 控制并发量
#### 任务执行结果反馈
#### 任务取消控制
#### 任务完成等待
