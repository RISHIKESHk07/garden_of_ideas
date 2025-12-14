# OBJ:

Making a *HTTP streaming server* for web clients (VLC , HTML 5 video player ...), for streaming of video files , will extend the for video conferencing as well , Write a optimised http & rest server for working on the above , could add p2p as well later on for scaling it , explore websockets & other real-time solutions for dedicated connections , look up *H20/Drogon* for the the server it self , and will be using *ffmpeg & DASH*
# LEARNING

## High performance browser networking 
### Primer on latency & bandwidth
#### Introduction
Latency is about how long it takes to communicate between the client & server , while bandwidth talks about how much throughput can achieved on said channel , a bit about **hibernia network** 
#### Components of latency
Propagation delay , processing delay , transmission delay ( delay in order for the packet to be entirely sent) , queue delay (queue consumption time) , these comprise the delays faced by a packet over the network and contribute to the latency , **bufferbloat** is a documented problem of queue delay , routers have been made to avoid drop of packets as much as possible , this breaks tcp congestion control , which will studies later on , **newCol queue management algo** solves this 

#### Propagation latency
Optical fibre cables and how we have achieved ms speed of data transfer and a Round trip time (RTT) in ms , having a delay in 100 ms is lag , 300 ms is sluggish , we have CDNs which help in distributing content all over the world in order to deliver content from the nearest CDN to the user .
#### Last Mile latency
It talks about how intial transfer of data from your local router to the isp network as a good amount of node hops which represent a good chunk of latency (10  - 65 ms) , we use traceroute to find the path of nodes we traverse , it does through increasing ICPM hop limit packets being sent on the path to the destination till we reach it.
#### Bandwidth
We have a fixed bandwidth of the link offered to us , but the performance depend on the network itself the amount of congestion , hardware failures of intermediate nodes , hacking /DDos etc . Now we have reached point where we cannot depend on solely on hardware upgrades to optical fibres and require different strategies , protocols , caching etc to speed our apps


## TCP
### Introduction
RFC 791 — Internet Protocol
RFC 793 — Transmission Control Protocol
TCP is more about realiability of the link it abstracts , in order delivery , retransmission of lot packets , congestion control & avoidance , data integrity , and HTTP can use either TCP or UDP as means of transport protocol for communication 
### TCP FASTOPEN & THS
in threeway hand shake we move the required meta data & intialization connection data before actual data is transmitted , its like a negotiation for set of rules & numbers to follow , its expensive due to this to start up a tcp connection and optimization of every tcp connection is of atmost importance , fast open makes use of syn packeta few amount of data , it has limitation as well such as only few types of http requests can be sent & etc . Can have upto 40% increase in data being visible in high latency networks . 
### CONGESTION
Congestion collapse is that when the RTT > Retransmission interval , we observe that the host has sent duplicate packets on to the network of the same data , the intermediate nodes get overloaded and result in dropping all the packets , which leads to the network working in maximum RTT and after this the require data reaches the destination . TCP solves this using congestion control , flow control 
### FLOW CONTROL & SLOW START & CONGESTION CONTROL & CONGESTION AVOIDANCE
**flow control** is done through communication of window size of client & server , its a way for knowing how much amount is safe to send as the server/ client can be occupied with high overloads and it will require a way to know whats a good window size to communicate (rwand) , its done through **ack packet** is used , window scaling was also used as a fixed maximum window was deemed not optimal . **Slow start** are the means the network uses to solve the issue " we still don't know a optimal window size of intermediate network in the above case even though found a optimal size for client & server for flow control " , we ues a cwnd (congestion window) which starts out of a initial value and slowly increases over the time we get more acknowledgements from client (the congestion window & rwnd dictates how much we send over the network , here we use say that we can only place a min(rwnd,cwnd) data ie. not ack data .. on to the network before receiving an ack ) , we do this until the cwnd reachs the value of rwnd and we use the full possible capacity of the channel . **Slow-start restart** does the above but resets when we reach maximum rwnd limit when we do not use the connection for sometime (no work to do) , this generally gets in the way of web server serving content . **Congestion avoidance** just dictates how do we handle the flow of data after we hit the rwnd limit and we lose a packet because of it , it uses a specific algo to determine the next possible window size for transfer , algos: TCP-CUBIC(windows) , TCP BIC etc . AIMD algo : reduce the window size by half when packet loss happens , PRR algo. Bandwidth delay product dictates that maximum unACK data on the fly depends upon target data rate & RTT , which in turn require proper window sizes to work upon , here when RTT high we are limited to a low transfer rate anyway but even on a high speed LAN if we do not have a good windoe size we cannot utilise it to the max , by default we have a cap at 64kb and can be solved by **window scaling** above the cap accordingly . 

### HEAD OF LINE BLOCKING
TCP is great for reliability but the issue still exists that we want in order delivery by TCP , which makes to wait for packets lost to be retransmitted and maintain order , which results in starving / delaying the subsequent packets .This causes alot of issue in application level where we in order delivery is not always the first priority (we use UDP).The entire flow of tcp packet transmission depends on the feedback loop of packet loss and adjusting of window sizes for congestion & flow control.

### OPTIMISATION
With the latest kernel in place, it is good practice to ensure that your server is configured to use the following best practices:

[Increasing TCP’s Initial Congestion Window](https://hpbn.co/building-blocks-of-tcp/#increasing-tcps-initial-congestion-window)
A larger starting congestion window allows TCP to transfer more data in the first roundtrip and significantly accelerates the window growth.
[Slow-Start Restart](https://hpbn.co/building-blocks-of-tcp/#slow-start-restart)
Disabling slow-start after idle will improve performance of long-lived TCP connections that transfer data in periodic bursts.
[Window Scaling (RFC 1323)](https://hpbn.co/building-blocks-of-tcp/#window-scaling-rfc-1323)
Enabling window scaling increases the maximum receive window size and allows high-latency connections to achieve better throughput.
[TCP Fast Open](https://hpbn.co/building-blocks-of-tcp/#tcp-fast-open)
Allows application data to be sent in the initial SYN packet in certain situations. TFO is a new optimization, which requires support both on client and server; investigate if your application can make use of it.




## UDP

# SOURCES

- High performance browser networking by ilya 
- H20 Optimise web server
- Simple Web server Github
- Dorgon Github
- FFMPEG
- DASH
- HTML 5 webserver
- VLC