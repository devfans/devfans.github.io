---
layout: post
title:  "Locks in C++ Multithreading"
author: devfans
categories: [ locks, c++, multithreading ]
image: /cdn.stocksnap.io/img-thumbs/960w/Z1TKDI29FZ.jpg
tags: [featured]
---

Locks are normaly used for sychronized resouce access. In C++, there're some common locks available, here is to explain a bit.

### Pthread Locks

`pthread.h` provides `mutex`, `rwlock` and `conditional var`.

```
#include <pthread.h>

// Initialize mutex
pthread_mutex_t mutex_lock = PTHREAD_MUTEX_INITIALIZER;
// Or
pthread_mutex_t mutex_lock;
pthread_mutex_init(&mutex_lock, NULL);

// Lock
pthread_mutex_lock(&mutex_lock);

// Unlock
pthread_mutex_unlock(&mutex_lock);

// Destroy
pthread_mutex_destroy(&mutex_lock);

// Add a cond var
pthread_cond_t cond_lock = PTHREAD_COND_INITIALIZER;

// Block
pthread_mutex_lock(&mutex_lock);
pthread_cond_wait(&cond_lock, &mutex_lock); // Wait for signal to relock the mutex
pthread_mutex_unlock(&mutex_lock);

// Signal
pthread_mutex_lock(&mutex_lock);
pthread_cond_signal(&cond_lock);
pthread_mutex_unlock(&mutex_lock);


// Initialize rwlock 
pthread_rwlock_t rw_lock;
pthread_rwlock_init(rw_lock, NULL);

// Lock
pthread_rwlock_rdlock(&rw_lock);
pthread_rwlock_wrlock(&rw_lock);

// Unlock
pthread_rwlock_unlock(&rw_lock);

// Destroy
pthread_rwlock_destroy(&rw_lock);

```

### Std Locks

```
// Mutex
std::mutex mutex_lock;

Do  kjk


```


## Refs

+ [pthread locks][pthread_locks]


[pthread_locks]: https://pubs.opengroup.org/onlinepubs/009695399/basedefs/pthread.h.html




