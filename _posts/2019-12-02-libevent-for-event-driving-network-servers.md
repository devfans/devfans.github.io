---
layout: post
title:  "Libevent for Event Driven Network Servers"
author: devfans
categories: [ libevent, c++ ]
image: /cdn.stocksnap.io/img-thumbs/960w/O5NJKI9KGZ.jpg
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

// Stop listening
evconnlistener_disable(listener);

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

// Init openssl
SSL_load_error_strings();
SSL_library_init();
OpenSSL_add_all_algorithms();
init_ssl_locking();
if (!RAND_pool()) {
  std::cout << "SSL random seed error " << get_ssl_error_string();
}

SSL_CTX * ssl_ctx = SSL_CTX_new(TLS_server_method());

assert(SSL_CTX_use_certificate_chain_file(ssl_ctx, tls_cert_file.c_str()));
assert(SSL_CTX_use_PrivateKey_file(ssl_ctx, tls_key_file.c_str(), SSL_FILETYPE_PEM));

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

#### Http

Libevent provides `struct evhttp` for web http serving.

```
#include <event2/http.h>
#include <event2/keyvalq_struct.h>

struct evhttp * http = evhttp_new(base);


// Set allowed methods
evhttp_set_allowed_methods(http, EVHTTP_REQ_GET | EVHTTP_REQ_POST, EVHTTP_REQ_HEAD);

// Set endpoint callback
evhttp_set_cb(
  http,
  path,
  cb,
  data   // Could pass `this` here for member function callback
);

// Bind with handle
struct evhttp_bound_socket * handle = evhttp_bind_socket_with_handle(
  http,
  listen_address,
  listen_port
);
assert(handle);

// Implement a callback
void handle_request(
  req,  // `struct evhttp_requst *`
  data  // User supplied data
) {
  // Get request headers
  struct evkeyvalq * input_headers = evhttp_request_get_input_headers(req);
  const char * value = evhttp_find_header(input_headers, "key");

  // Set response headers
  struct evkeyvalq * output_headers = evhttp_request_get_output_headers(req);
  evhttp_add_header(output_headers, "key", "value");

  // Get method
  evhttp_cmd_type method = evhttp_request_get_command(req);
  if (method == EVHTTP_REQ_POST) {}

  // Parse url parameters and convert to evkeyvalq
  struct evhttp_uri * uri = evhttp_uri_parse(evhttp_request_get_url(req));
  char * query_dup = nullptr;
  const char * query = evhttp_uri_get_query(uri);
  query_dup = strdup(query);
  evhttp_uri_free(uri);
  struct evkeyvalq params;
  evhttp_parse_query_str(query_dup, &params); // Then can use evhttp_find_header to get key values
  free(query_dup);

  // Parse body
  char * body;
  struct evbuffer * buffer = evhttp_request_get_input_buffer(req);
  auto length = evbuffer_get_length(buffer);
  body = (char *)malloc(length + 1);
  evbuffer_copy_out(buffer, body, length);
  body[length] = '\0'  // event buffer does not include '\0'
  evbuffer_free(buffer);

  // Send string response
  struct evbuffer * reply = evbuffer_new();
  reply_add_printf(reply, "{ \"data\": \"%s\" }", data.c_str());
  evhttp_send_reply(req, HTTP_OK, "OK", reply);
  evbuffer_free(reply);
}

```

#### Event

Libevent provides general purpose event to schedule executions to happen at deired events.

```
// Allocate new events
struct event * new_event = event_new(
  base,
  fd,        // fd, or signal or -1
  events,    // 0 or EV_PERSIST, EV_READ, EV_WRITE, EV_CLOSED
  callable,
  data
);

// Schedule the event with a timeout
event_add(
  new_event,
  timeout     // `struct timeval *` if NULL, the event will only be activated when deired events appear
);

// Schedule tasks to be executed in event loop thread
event_add(new_event, NULL);
event_activate(new_event, 0 /*flags to pass in callback*/, 0);

```

#### Libevent in Mutlithreads

When using libevents in multithreads, needs to state to use pthread locks before event base is created

```
#include <event2/thread.h>

evthread_use_pthreads();
```


### Sum up

Libevent encapsues system I/O and sigal handling, providing convenient functionalilites for improving network server performance, however, it also has some drawbacks like some bad implementations for dns and http servers and the timers behave inexactly and cant cope with time jumps. There's another library called `libev` trying to solve this issue and mainly focus on POSIX event efficiency, while libevent is providing a full stack solution. 


##### Refs:

+ [Libevent Reference][libevent_refs]
+ [libevent vs libev][libevent_vs_libev]

[libevent_refs]: https://libevent.org/doc/index.html
[libevent_vs_libev]: https://stackoverflow.com/questions/9433864/whats-the-difference-between-libev-and-libevent


