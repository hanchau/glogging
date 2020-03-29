+++
author = "hanchau"
title = "CAP theorem!"
date = "2020-03-22"
description = "Pain of the distributed state."
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

This article offers a basic understanding of the CAP theorem.
<!--more-->

#### What is CAP theorem?

In the last post I discussed why we need distributed systems and how to setup gearman on a cluster. In this post I'll try to convince you that CAP theorem is indeed a *"theorem"*. Anyways, We like distributed systems!! (because they provide us the [features](https://medium.com/system-design-blog/key-characteristics-of-distributed-systems-781c4d92cce3) that we really want) but..., Are they trivial to implement?.

My colleague once said
> More code => More Complexity => More Monster Bugs => More Pain

Let's start with that, Initially we used to have a beautiful piece of machine having 8 cores, 16 gigs of memory, few hundread gigs of storage. So if you wanted to get a *state* (An abstraction over the data stored in the memory or in disk), you can do
```
A.getState()
```

and if you want to set the state you can similarly do
```
A.setState(10)
````

So these are the fundamental operations that you can do to a *state*.

You have a stream of operations, you can always maintain a queue and do the following-
```
1. while not_empty(Q):
2. op = dequeue(Q)
3. if validate(op):
4.     response = exec(op)
5.     send_back(response)
6. else:
7.     response = op_not_found_error()
8.     send_back(response)
```

So we can clearly see that the machine is Consistent, it eithers gives you the *recent write* (because all the operations are sequential) or the error, We have a single source of truth here, i.e our beautiful machine's state. The Availability of the machine depends on the network and the machine itself. Now, we want to imporve the Availability of the machine. If we add one more machine to the system and want to implement the same *queue* Q that we did for a single machine we can do the following.

```

1. def exec_on_machine(op, machine_name):
3.    if validate(op):
4.      response = exec_mac(op, machine_id)
5.      send_back(response)
6.    else:
7.      response = op_not_found_error()
8.      send_back(response)
9.
10. while not_empty(Q):
11. op = dequeue(Q)
12. if check_machine_A_online():
13.     exec_on_machine(op, A)
14.     sync_states(A,B)
15. else:
16.     exec_on_machine(op, B)
17.     sync_states(B,A)
```

Hurray!! We implemented a more Availabile system now... NO we haven't. Consider the following scenario-

```
Given queue of operation requests- Q = object.setState(10) | object.setState(15) | object.getState()

```
coming soon..

Thanks, Cheers & Happy Glogging.
