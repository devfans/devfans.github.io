---
layout: post
title:  "Common Ways of Multithreading in C++"
author: devfans
categories: [ multithreading, c++ ]
image: /cdn.stocksnap.io/img-thumbs/960w/7JBL5TC3S7.jpg
tags: [featured]
---

## Multithreading for parallelism
Multithreading is a basic way to improve software through parallelism, and it provides more efficient ways for communications and resource sharing than multiple processes. Before c++11, we can use POSIX library, and then the std::thread was introduced in c++. Besides that, there are some other libraries to use like boost provide similar interfaces too.


#### POSIX thread
The standard implementation only support Unix-like POSIX-conformant platforms, may need other third party packages if to use on other platforms like Windows. 


```
#include <pthread.h>

int result_code;
// Thread ID
thread_t tid = 0;
result_code = pthread_create(&tid, NULL, callable_fn, args);
assert(!result_code);
result_code = pthread_join(tid, NULL);
assert(!result_code);

```

#### std thread
When a `std::thread` object is created, a new thread would be launched and start to execute the callable specified.

```
#include <thread>

// params is optional
std::thread new_thread(callable, params);
new_thread.join();
```

When using `std::future` object, `async` provider could implicitly launch threads for you too.
```
#include <future>

std::future<bool> fut = std::async(callable, params);
bool ret = fut.get();

```
You can specify the thread launching policy as the first arg(like `std::async(std::launch::async, callable, params)`), which has values:
+ `std::launch::async` Function is executed by new thread asynchronous.
+ `std::launch::deferred` Function is executed when shared resources are accessed.
+ `std::lauch::async | std::launch::deferred` Determined automatically.

#### boost thread

```
#include <boost/thread.hpp>

boost::thread new_thread(callable, params);
new_thread.join(); // Join is not harmful even if the thread is already completed.
new_thread.detach(); // Detach would also been called during destruction, which would continue the execution.
boost::thread another_thread(boost::bind(member_func, this)); // bind is unnnecessary though for thread constructor would bind it internally.

```

### So which one shall we use?

For historical reasons, POSIX threads may look more mature on common platforms than std thread, but you could use boost thread too if it's one of your concerns while allowing introducing extra dependencies. The new std thread is at a higher abstraction layer which provides better interfaces and functionalities.


## Thread pool

When you get lots of tasks to execute with thread, you may want to avoid the overhead of thread objects management and get a bit more control of the resources to be used, which means you probably want a thread pool.

#### boost asio thread pool

Using boost thread group.

```
#include <boost/asio/io_service.hpp>
#include <boost/bind.hpp>
#include <boost/thread/thread.hpp>

// Initialize
boost::asio::io_service task_io_service;
boost::thread_group task_thread_group;

// Start the processing loop
boost::asio::io_service::work task_work(task_io_service);

// Add one thread, add more if you want
task_thread_group.create_thread(boost::bind(&boost::asio::io_service::run, &task_io_service));

// Submit tasks
task_io_service.post(callable, params);

// Stop the execution loop
task_io_service.stop();


// Join all threads
task_thread_group.join_all();

```

Using boost thread pool starting from 1.66.0

```
#include <boost/asio/thread_pool.hpp>
#include <boost/asio/post.hpp>

boost::asio::thread_pool task_thread_pool(4);  // size
boost::asio::post(task_thread_pool, callable, params);
task_thread_pool.join();

```


#### Create a std thread pool manually

To manage a thread pool, basicly the solution is the maintains a list of threads, and each of them is executing a loop which is blocked when no more task is available, and fetch one and execute it once it's notified a new task is inserted. If the result of tasks should be provided after it completed, it can be provided via `std::future` object which could wait for the result to be ready in caller thread.

Here's showing a piece of code for managing producer/consumer event using conditional variables:

```
std::mutex shared_mutex;
std::condition_variable task_queue_not_full;
std::condition_variable task_queue_not_empty;

// Add new tasks
{
  std::unique_lock<std::mutex> l { shared_mutex };
  task_queue_not_full.wait(l);
  // insert task
}

// Fetch task from the queue
{
  std::unique_lock<std::mutex> l { shared_mutex };
  task_queue_not_empty.wait(l)
  // pop task from queue
}

```

Since thread pool is a common practice for computation tasks, some implementation samples are shared online and available for trying out: [this one][thread_pool] and [ctpl]


[thread_pool]: https://github.com/Tyler-Hardin/thread_pool
[ctpl]: https://github.com/vit-vit/CTPL

