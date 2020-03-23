+++
author = "hanchau"
title = "Distributed systems!"
date = "2020-03-15"
description = "Why do we need distributed systems?"
tags = [
    "distributed systems",
    "gearman job server",
    "gearman",
]
categories = [
    "posts",
    "concepts",
]
+++

This article offers a basic understanding of the distributed systems.
<!--more-->

#### What are Distributed Systems?

In the last post I gave a brief about the concept of parallel processing through some examples. Now we know how to make our systems more parallel in nature, I think it's time learn the concept of distributed processing, how it's different from parallel processing and how to integrate it in our existing projects. There are lots of definitions of Distributed System but I'll give you the one I like the most which is obviously mine (HAHA).

> A system having a collection of multiple computer systems located at different geographical locations.

Some examples of distributed systems are:

1. [The Internet](https://arsalankhan.com/2015/07/02/is-internet-a-distributed-system/)
2. [Google](http://highscalability.com/google-architecture)
3. [Facebook](https://prezi.com/phbdww0euz94/facebook-as-a-distributed-system/)

There are some useful primitive features to exploit out from a distributed system:
```
Distributed Storage.
Distributed Computing.
Distributed Memory.
```
Now from these primitive features we can construct many high level features having varied quantities of primitives features.
1. No Single point of Failure.
2. High Scalability.
3. Low Latency.
4. Reliability.


#### Single Point of Failure.
Now we know how to process parallelly, let's say you wrote an application which uses all these cool parallel processing features and deployed it on the server. Days passes and one day a thunderstorm comes and electricity system of the city goes down. Nobody can use ur application. So... what to do?

I think you got the point, there should not be a single thing in our system on which the whole system depends i.e. if that thing goes down system goes down.

One solution is, why don't we replicate our system in another server who has a different electricity supplier. Well it is possible.

We can go with this solution and to achieve no single point of failure with gearman we just have to create a new replica of the original server.

The Gearman solution:
```
Given initial server A.
Get an new server B in a seperate geographical location.
Clone your application.
Start a new gearman job server.
  $ gearman -d <port>
Add all your workers/clients to both the gearman job servers A and B.
Daemonize your workers on B.
```
Now, if any one of them goes down there will be another one to process the tasks.
Hurray!! Now we're safe from any kind of electricity isssue.
```
NOTE: By getting servers of different electricity supplier we're just trying to minimize the probability of getting a downtime, because as we keep on increasing the number of backup servers from different electricity suppliers the probability of having an electricity shutdown for all the supplier at the same time would be very less.
```


#### High Scalability
I think of scaling as an increase in the number of tasks to perform by the system. These tasks can be CPU Intensive, I/O Intensive or can have both CPU and IO bursts. So, let's say you have a live video streaming application which was initially doing 10^1 taks/min with a machine limit of 10^2 tasks/min, but due to sudden increase in the user base because a live stream suddenly got viral, your system now receives 10^3 tasks/min. So... What will you do? Well you might say that this is not practical, but trust me if it happens, given you just started up as a live video streaming site.

The Gearman solution:
```
Given initial server A.
Get an extra server B.
Clone your application.
Start a new gearman job server.
  $ gearman -d <port>
Add all your workers/clients to both the gearman job servers A and B.
Daemonize your workers on B.
Repeat this for 8 new machines.
```

Maximum system limit becomes **10 * 10^2 = 10^3** tasks/min.

Hurray!!, Now we can sleep like a baby. Both of the other features are similar but with their own little flavor like, if you want your application response to be geography agnostic i.e. A person in India gets response in 100 ms and A person in US also gets response in the same time, Then you have to deploy it on multiple continents with all of its dependencies like DBs, Metadatas and Caches.

Thanks, Cheers & Happy Glogging.

> A Note on COVID-19 pandemic: Keep Calm, Eat Well & Stay @ Home.


#### A cool video to briefly get the downsides of distributed systems.
---

{{< youtube Y6Ev8GIlbxc >}}

<br>
