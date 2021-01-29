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
一般任务可以并列执行，如果需要部分有序执行，那只能分成多个执行队列。如果需要全部有序，那估计没得玩了，逐个干吧。

#### 控制并发量
机器资源有限，如果不控制并发量任意开新的执行线程，反而会使任务执行效率变低。

#### 任务执行结果反馈
如果任务执行结果需要收集，可以考虑用channel或着写数据库。如果执行结果出错，则判断是否需要反馈给线程池。一般任务的执行结果可以直接丢掉，如果执行出错，可能也会直接略过，或着回Queue等待下次执行。同时可以加error recover防止影响程序继续执行。

#### 任务取消控制
如果上游Context触发取消，那么线程池中的任务就需要有序取消，特别是任务中的非事务性数据操作。

#### 任务完成等待
做任务计数，如果没有发生中途取消，则等待全部完成。


### 明确实现需求

Golang中的errgroup包可以做一些任务控制，但并未做并发控制，并且其中的任务取消控制存在瑕疵。通过梳理，目标并发器业务场景需要满足以下几点：

- 任务可以无序执行
- 任务执行结果可以忽略，但需要捕获panic，防止任务中止
- 任务中操作最好保持数据一致性，不要中途无序中止
- 上游Context触发取消时，任务获取可以中止，尚在队列中的任务可以放弃，但执行中的任务需要继续执行完成
- 如果没有取消，则需等待任务全部执行完成


### 上代码：

```

type CoGroup struct {
	context.Context
	wg   sync.WaitGroup
	ch   chan func(context.Context) error // Task chan
	sink bool                             // Use group context or not
	open bool                             // Open signal
	jobs int
	done chan bool // Close chan for draining
	sync.Mutex
}


```
context又上游控制，wg用来等待全部线程退出，任务放在channel里，open作为队列是否仍开放的标志。jobs进行任务计数，done channel用来做任务执行反馈计数。

```
func (g *CoGroup) start(n int) {
	for i := 0; i < n; i++ {
		g.wg.Add(1)
		go g.process()
	}
}

func (g *CoGroup) process() {
	defer g.wg.Done()
	for {
		select {
		case f, ok := <-g.ch:
			if !ok {
				return
			}
			g.run(f)
		case <-g.Done():
			return
		}
	}
}

```

任务开始开始
