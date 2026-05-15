# OBJ:

Looking up a way to recreate nginx/cloudfare concurrent request processing & integrate it into my own http server (side project) , idea is to look up libuv ,boost.asio ,h20, oxy,openrusty, nginx , c10k problem & come up with a simpler version of the whole process to understand internals of libuv , concurrency , nginx  etc .

# LEARNING:

Clock cycles & instruction cycle , the clock is a periodic pulse which helps in making sure the instructions are executed or worked upon in sync manner ie. in-case processing of a previous ins ,required for current work etc, it helps making sure everything is in sync & nothing wait for no reason , instruction cycle is the various steps involved in a ins (fetch , decode , exe ... etc), clock cycle represent a unit work on this above steps here , you could take maybe 1 cyc , 2 cyc or more (processor is bad) to complete a single step , so faster the clock cyc faster the processing speed , and more amount of work done in same time. "CPU cycles, or clock cycles, are the fundamental, rhythmic pulses that govern a computer's CPU's operations , 4GHz clock speed does 4B ops/sec " (source: reddit & intel doc) 

Some numbers why we are talking about this , disk look up requires 41000 cycles  , network related tasks 240 000 cycles .

Originally we had apache httpd , which used to follow the one process per connection , essentially a pop-up thread per connection , while we have nginx which uses a async event driven arch , it uses a event loop to listen to requests by connections , which are queued & then we have a set of worker threads which handle the underlying io (file,disk data fetch etc ... use auxillary threads for longer processes or which require dedicated threads for it  while the worker thread can help in connection of new clients after giving this work to aux threads) , this more memory safe & less race conditions to worry about & less amount of memory usage as a single thread is used instead of spawning new processes .

NodeJs also another example uses this arch , it use the Libuv project in-order to do this ,*need to write a in depth doc about libuv itself* for checking how it manages the event loop & polling & callbacks

Epoll , kqueues etc are the event multiplexers according to various unix systems . Short note on epoll , epoll is like event registry waiting to create events when occur and then put them on a event queue , we have edge trigger & level trigger , this system is supposed to work on blocking stuff like network calls , regular file calls & disk calls are not async impl using epoll rather we use multi threading to acheieve that entirely .

Some issues with event loops , every task has to be done fast or else a single task could starve the upcoming tasks , which could perform worse than multi threading , so inorder to mitigate this cloudfare (like nginx , libuv  before it ..) adopted a hybrid thread pool for intensive taks + event loop approach , used in filesystem reads . Cloudfare talks about their firewall application (which is a cpu intensive task) ,

From the above , nginx was based on a event driven approach with workers and a master process to manage the corresponding process in the run loop , worker (single threaded process) , it comes with a modular structure to work come on various request , phase handlers , protocols , filters , upstream and load balancers etc.

https://aosabook.org/static/nginx/architecture.png

**Ngnix** is a web server with some extra jazz features like caching , load balancing , bandwidth control . The need for better web servers was the due to the growing number of connections & users (due to mobiles , modern browsers using parallel connections .. ) , and older systems worked on the principle of allocating more memory to newer connections , which obviously puts a clear upper limit due to the hardware we had , Nginx was a solution to C10k (Daniel Kegel) problem

```
nginx is very well suited for this, as it provides the key features necessary to conveniently offload concurrency, latency processing, SSL (secure sockets layer), static content, compression and caching, connections and requests throttling, and even HTTP media streaming from the application layer to a much more efficient edge web server layer. It also allows integrating directly with memcached/Redis or other "NoSQL" solutions, to boost performance when serving a large number of concurrent users.
```

nginx uses multiplexing and event notifications heavily, and dedicates specific tasks to separate processes. Connections are processed in a highly efficient run-loop in a limited number of single-threaded processes called `worker`s. Within each `worker` nginx can handle many thousands of concurrent connections and requests per second.

The core of the web server is the event loop as discussed and the Modules make up the second part which are responsible lots of application/presentation functionality , outbound filtering , upstream comm & transforming content .

The entire build is based on the performance & conservation of cpu (cpu cycles due avoiding creation-destroy pattern) , and also uses pool & slab memory allocators to work on this , with corresponding syscalls for a small memory footprint , this make having mult-core scale well without any thread-thrashing & lock-ups occuring.

Older problems was scripting langs blocking a worker and stalling the whole setup , & Disk IO performance slowing a worker 

Coming to Processes which are present a Master Process , bunch of Worker Process , Cache loader & Cache manager . All of the above share memory mechanisms for ipc .

```
The master process is responsible for the following tasks:

reading and validating configuration
creating, binding and closing sockets
starting, terminating and maintaining the configured number of worker processes
reconfiguring without service interruption
controlling non-stop binary upgrades (starting new binary and rolling back if necessary)
re-opening log files
compiling embedded Perl scripts
The worker processes accept, handle and process connections from clients, provide reverse proxying and filtering functionality and do almost everything else that nginx is capable of. In regards to monitoring the behavior of an nginx instance, a system administrator should keep an eye on workers as they are the processes reflecting the actual day-to-day operations of a web server.

The cache loader process is responsible for checking the on-disk cache items and populating nginx's in-memory database with cache metadata. Essentially, the cache loader prepares nginx instances to work with files already stored on disk in a specially allocated directory structure. It traverses the directories, checks cache content metadata, updates the relevant entries in shared memory and then exits when everything is clean and ready for use.

The cache manager is mostly responsible for cache expiration and invalidation. It stays in memory during normal nginx operation and it is restarted by the master process in the case of failure.

```

Caching is done through the shared mem & os's page cache itself , and heiharical fs directories (too spread out the content acoss multiple directortries) , 

Over here , we look up the internals of nginx a bit more in detail , mainly abou the core module and the remaining supporting module  , remember the entire arch is just a collection of plug and play modules attached in a way to make the processing of different requests better and efficient , which includes utility functions such as upstream , compression etc .

The functional modules can be divided into event modules, phase handlers, output filters, variable handlers, protocols, upstreams and load balancers.

```

REQUEST FLOW  IN NGINX;
REQUEST -> PHASE HANDLERS -> UPSTREAM -> FILTERS -> RESPONSES

```

Phase handlers typically do four things: get the location configuration, generate an appropriate response, send the header, and send the body. A handler has one argument: a specific structure describing the request. A request structure has a lot of useful information about the client request, such as the request method, URI, and header.

Filters are also attached to locations, and there can be several filters configured for a location. Filters do the task of manipulating the output produced by a handler. The order of filter execution is determined at compile time. For the out-of-the-box filters it's predefined, and for a third-party filter it can be configured at the build stage. In the existing nginx implementation, filters can only do outbound changes and there is currently no mechanism to write and attach filters to do input content transformation. Input filtering will appear in future versions of nginx.

Powerful nature of subrequests in ngnix which allows the user/nginx to setup a internal redirect option for the incoming requests and also chain subrequests to creating some powerful operations (authentication biggest example , obtain a token from  once use it to create another auth object etc or SSI  SERVER SIDE INCLUDES )

Upstreams and Load balancing are some other crucial modules present which work in conjecture to proxy-pass module .

Memory alloaction is done through pointers and copy is obv not preferred and For each connection, the necessary memory buffers are dynamically allocated, linked, used for storing and manipulating the header and body of the request and the response, and then freed upon connection release.

BTW Modules responses are kept buffers and then linked to a chain which is maintained even for filters , it can get complicated and hard to follow and woeks different based on the filters that we use .

The task of managing memory allocation is done by the nginx pool allocator. Shared memory areas are used to accept mutex, cache metadata, the SSL session cache and the information associated with bandwidth policing and management (limits). There is a slab allocator implemented in nginx to manage shared memory allocation. To allow simultaneous safe use of shared memory, a number of locking mechanisms are available (mutexes and semaphores). In order to organize complex data structures, nginx also provides a red-black tree implementation. Red-black trees are used to keep cache metadata in shared memory, track non-regex location definitions and for a couple of other tasks.

Coming to some new stuff Pingora , a rust based proxy by cloudfare built to handle a trillon requests a day , it is succesor to nginx implementation used to have , mainly this is concerned about how we proxy our requests to the servers in question , their was a few blogs about how to enhance the connection management of connection to the Cloudfare Global network with a few ideas being http2 and quic upgrades etc , which is not exactly what we are looking at here .

Coming to issues with nginx was that the unbalanced load of work on workers in the system , in another blog we mentioned how the we have two methods of running nginx here combined queue model and SO_REUSEPORT port (per worker queue allocation does not go through a combined queue when processing), here the issue is in the first model we see that load balancing is bad (a issue with how epoll based solution for accepting connections always lead to the busiest worker getting the job again ) but latency is predictable , and in the second idea we distribute it across many queues , so load balancing is excellent but latency is all over the place (not predictable) , their have been some patchs to fix this issue the epoll impl

Now coming back their is a another problem with nginx that incase of CPU heavy tasks it blocks all incoming tasks , making it a burden incase of constant CPU intensive jobs . A another identified issue is reuse of tcp connections from proxy service to origin servers , the re costruction of connections (TSL handshake) instead of reusing existing is something we would not want .The complexity of nginx C based codebase was issue here , and remember we use lua with is less performant , along with all these standing issues the community has development work being behind closed doors .

Moving Pingora was build to make become to work a lot with non RFC compliant http requests which cloudfare handle at large , so normal rust based lisb like hyper were not a great choice because they do not support this exactly (they did patch a request from cloudflare but ..) , the wanted to use multithreading and work stealing which is bundled in tokio async runtime really well .

Pingora surpassed nginx in almost every way , and some key points are that multithreading model lets better share of resources compared to existing nginx ;s shared memory which is tied down with a mutex and you can store integers and strings only , better reuse ratio due to work stealing obviously .





# SOURCES:
- cloudfare blog
- Nginx master blog
- Libuv doc
- Hoff._world yt
- https://aosabook.org/en/v2/nginx.html
- https://blog.cloudflare.com/how-we-built-pingora-the-proxy-that-connects-cloudflare-to-the-internet/
- 

