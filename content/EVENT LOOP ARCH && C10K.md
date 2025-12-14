# OBJ:

Looking up a way to recreate nginx/cloudfare concurrent request processing & integrate it into my own http server (side project) , idea is to look up libuv ,boost.asio ,litespeed, nginx , c10k problem & come up with a simpler version of the whole process to understand internals of libuv , concurrency , nginx  etc.L

# LEARNING:

Clock cycles & instruction cycle , the clock is a periodic pulse which helps in making sure the instructions are executed or worked upon in sync manner ie. in-case processing of a previous ins ,required for current work etc, it helps making sure everything is in sync & nothing wait for no reason , instruction cycle is the various steps involved in a ins (fetch , decode , exe ... etc), clock cycle represent a unit work on this above steps here , you could take maybe 1 cyc , 2 cyc or more (processor is bad) to complete a single step , so faster the clock cyc faster the processing speed , and more amount of work done in same time. "CPU cycles, or clock cycles, are the fundamental, rhythmic pulses that govern a computer's CPU's operations , 4GHz clock speed does 4B ops/sec " (source: reddit & intel doc) 

Some numbers why we are talking about this , disk look up requires 41000 cycles  , network related tasks 240 000 cycles .

Originally we had apache httpd , which used to follow the one process per connection , essentially a pop-up thread per connection , while we have nginx which uses a async event driven arch , it uses a event loop to listen to requests by connections , which are queued & then we have a set of worker threads which handle the underlying io (file,disk data fetch etc ... use auxillary threads for longer processes or which require dedicated threads for it  while the worker thread can help in connection of new clients after giving this work to aux threads) , this more memory safe & less race conditions to worry about & less amount of memory usage as a single thread is used instead of spawning new processes .

NodeJs also another example uses this arch , it use the Libuv project in-order to do this ,*need to write a in depth doc about libuv itself* for checking how it manages the event loop & polling & callbacks

Epoll , kqueues etc are the event multiplexers according to various unix systems . Short note on epoll , epoll is like event registry waiting to create events when occur and then put them on a event queue , we have edge trigger & level trigger , this system is supposed to work on blocking stuff like network calls , regular file calls & disk calls are not async impl using epoll rather we use multi threading to acheieve that entirely .

Some issues with event loops , every task has to be done fast or else a single task could starve the upcoming tasks , which could perform worse than multi threading , so inorder to mitigate this cloudfare (like nginx , libuv  before it ..) adopted a hybrid thread pool for intensive taks + event loop approach , used in filesystem reads . Cloudfare talks about their firewall application (which is a cpu intensive task) ,

*Need to complete the cloudfare blog metrics & read about nginx blog , epoll/libuv/boost.asio impl a bit , will need to focus on impl after this ..* 

# SOURCES:
cloudfare blog
Nginx master blog
Libuv doc
Hoff._world yt
