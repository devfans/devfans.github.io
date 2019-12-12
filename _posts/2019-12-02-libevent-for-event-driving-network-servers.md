---
layout: post
title:  "Libevent for Event Driven Network Servers"
author: devfans
categories: [ libevent, c++ ]
image: /cdn.stocksnap.io/img-thumbs/960w/7JBL5TC3S7.jpg
tags: [featured]
---

When dealing with network IO events, we normally donot want the events block the execution of threads, instead other on-blocking tasks should switch in and take of the resources. Libevent is a well known event driven frameworks commonly used for network servers.
According to the official introduction:


"Libevent is meant to replace the event loop found in event driven network servers. An application just needs to call event_base_dispatch() and then add or remove events dynamically without having to change the event loop. Currently, Libevent supports /dev/poll, kqueue(2), select(2), poll(2), epoll(4), and evports. The internal event mechanism is completely independent of the exposed event API, and a simple update of Libevent can provide new functionality without having to redesign the applications. As a result, Libevent allows for portable application development and provides the most scalable event notification mechanism available on an operating system. Libevent can also be used for multithreaded programs. Libevent should compile on Linux, *BSD, Mac OS X, Solaris and, Windows."


#### Event Base

Libevent is using `struct event_base` to watch the state of events. To create an `struct event_base` object:

```
#include <event2/event.h>

struct event_base * base = event_base_new();
assert(base);

// To start the event loop: 0 if successful, -1 if an error occurred, or 1 if we exited because no events were pending or active.
auto dipatch_ret = event_base_dispatch(base);

// Exit the event loop: 0 if successful, or -1 if an error occurred
// The second arg is `const struct timeval *` to specify the timeout duration, after which loop will exit.
auto exit_ret = event_base_loopexit(base, NULL);


// To release it when after stopped the event loop
if (base != nullptr)
  event_base_free(base);

```

#### Connection Listener

Most network servers need to listen for connections.

```
#include <event2/event.h>
#include <event2/listener.h>

// Define a callable to be executed when new connection comes in
void callable(
  listener,
  fd,                  // evutil_socket_t
  socket_address,
  socket_address_size,
  data                 // as below
);

// Create the listener and bind to a socket address
struct evconnlistener * listener = evconnlistener_new_bind(
  base,             // event base
  callable,         // could be a member function or other callable
  data,              // user supplied pointer, a usage is to pass in `(void *)this`
  flags,            // for example: LEV_OPT_REUSEABLE | LEV_OPT_CLOSE_ON_FREE | LEV_OPT_REUSEABLE_PORT
  backlog,          // normal set as -1 for a reasonable default
  socket_address,   // `struct sockaddr *`
  socket_address_size
);
assert(listener);

// To release it
if (listener != nullptr)
  evconnlistener_free(listener);

```

#### Buffer Event

When dealing with IO events with socket fd, `struct bufferevent` is commonly used.

```
#include <event2/bufferevent.h>

struct bufferevent * bev = bufferevent_socket_new(
  base,
  fd,
  flag  // For example: BEV_OPT_CLOSE_ON_FREE | BEV_OPT_THREADSAFE
);

// Bind callbacks
bufferevent_setcb(
  bev,
  read_cb,
  write_cb,
  event_cb,
  data
);

// Set Read/Write timeouts
bufferevent_set_timeouts(
  bev,
  read_timeout,   // read timeout `struct timeval *`
  write_timeout   // write timeout `struct timeval *`
); 

// Enable events
bufferevent_enable(bev, EV_READ | EV_WRITE);

// Define a event callback
void event_cb(
 bev,
 events  // For example: BEV_EVENT_CONNECTED, BEV_EVENT_EOF, BEV_EVENT_ERROR, BEV_EVENT_TIMEOUT
);

// Define a read callback and read the data
void read_cb(
  bev,
  data
) {
  auto data = bufferevent_get_input(bev);  // event buffer see below
}

// Write data
bufferevent_write(bev, data, data_size);

// Release
bufferevent_free(bev);

```

When using openssl to encrypt the traffic

```
#include <openssl/ssl.h>
#include <event2/bufferevent_ssl.h>

SSL_CTX * ssl_ctx = get_server_ssl_ctx(
  tls_cert_file,
  tls_key_file
);

SSL * ssl = SSL_new(ssl_ctx);

struct bufferevent * bev = bufferevent_openssl_socket_new(
  base,
  fd,
  ssl,
  ssl_stats,  // BUFFEREVENT_SSL_ACCEPTING
  flags   
);
```

#### Event Buffer

`struct evbuffer` is used for processing buffered bytes.

```
// Create an event buffer to concating received data
struct evbuffer * buffer = evbuffer_new();

// Concate data
evbuffer_add_buffer(
  buffer,
  data      // Received data: `struct evbuffer *`
);

// Get length
auto length = evbuffer_get_length(buffer);

// Copy out
evbuffer_copyout(buffer, data_out, size);

// Read out
evbuffer_remove(buffer, data_out, size);

// Example for read a line
{
  struct evbuffer_ptr eol = evbuffer_search_eol(
    buffer,
    nullptr,          // Start point
    nullptr,          // EOL size to write
    EVBUFFER_EOF_LF   // EOL style
  );
  if (eol.pos >= 0) {
    std::string line;
    line.resize(eol.pos + 1);
    evbuffer_remove(buffer, &line.front(), &line.size());
  }
}

// Release
evbuffer_free(buffer);

```






