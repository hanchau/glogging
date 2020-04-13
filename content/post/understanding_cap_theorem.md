+++
author = "hanchau"
title = "CAP theorem!"
date = "2020-03-29"
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

If you have a stream of operations, you can always maintain a queue and do the following-
```
1. while !empty(Q):
2. op = dequeue(Q)
3. if validate(op):
4.     response = exec(op)
5.     send_back(response)
6. else:
7.     response = op_not_found_error()
8.     send_back(response)
```

So we can see that the machine is Consistent, it either gives you the *recent write* (because all the operations are sequential) or the error. We have a single source of truth here (our beautiful machine's state). The Availability of the machine depends on the network and the machine itself. Now, we want to imporve the **Availability** of the machine. If we add one more machine to the system and want to implement the same *queue* Q that we did for a single machine we can do the following.

```

1. def exec_on_machine(op, machine_name):
2.    if validate(op):
3.      response = exec_mac(op, machine_id)
4.      send_back(response)
5.    else:
6.      response = op_not_found_error()
7.      send_back(response)
8.
9.
10. while !empty(Q):
11.     op = dequeue(Q)
12.     if check_machine_A_online():
13.         exec_on_machine(op, A)
14.         sync_states(A,B)
15.     else:
16.         exec_on_machine(op, B)
17.         sync_states(B,A)
```

Hurray!! We implemented a more availabile system, but.... Consider the following scenario:

```
Given a queue of operation requests - Q = object1.setState(10) | object1.setState(15) | object1.getState()

```
Op. No. | Online | Offline | A's state | B's State
--------|--------|---------|-----------|----------
 1      | A, B   | -       | 10        | 10
 2      | A      | B       | 15        | 10
 3      | B      | A       | 15        | 10

Above table shows the state of both machines. We can see that the recent write is 15. The state of B is still 10 which is a not a recent write. Due to the down time of B during the second operation it never synced with the machine A.  The third operation will not get the old state not the recent write. Hence our system becomes **Availabile** but **Inconsistent**.

Now let's say we don't want any inconsistency in the system and for that we make a policy that if all the machines are not online then just don't execute any operation, which can be implemented as follows:


```
1. def exec_on_machine(op, machine_name):
2.    if validate(op):
3.      response = exec_mac(op, machine_id)
4.      send_back(response)
5.    else:
6.      response = op_not_found_error()
7.      send_back(response)
8.
9.
10. while !empty(Q):
11.     op = dequeue(Q)
12.     if check_machine_A_online() and check_machine_B_online():
13.                           # added a get_location condition to show
14.                           # the significance of 2 machines.
15.         if get_location()=="Pune":
16.             exec_on_machine(op, A)
17.             sync_states(A,B)
18.         else:              # machine in Mumbai
19.             exec_on_machine(op, B)
20.             sync_states(B,A)
21.     else:                  # enqueue the dequeued op request
22.         enqueue(Q,op)
```

Now consider a scenario:

```
Given a queue of operation requests - Q = object1.setState(10) | object1.setState(15) | object1.getState()

```
Op. No. | Online | Offline | A's state | B's State
--------|--------|---------|-----------|----------
 1      | A, B   | -       | 10        | 10
 2      | A      | B       | 10        | 10
 3      | B      | A       | 10        | 10

We can see that second and third request will not be executed because one of the machine is offline in both the cases.
That means our systems is **Consistent** but it is not as **Availabile** as it was previously.

I think you got the point I'm trying to make. So let's copy the definition from the [wikipedia](https://en.wikipedia.org/wiki/CAP_theorem) which says
```
It is impossible for a distributed data store to simultaneously provide more than two out of the following three guarantees:

Consistency: Every read receives the most recent write or an error

Availability: Every request receives a (non-error) response, without the guarantee that it contains the most recent write

Partition tolerance: The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes
```

When we headed towards a distributed system from our beautiful machine we realized that we can either promise Availability of the system or Consistency in the system given a Partition occurs.

Ending the post with few words of wisdom.

**Your Power is Your Weakness**. *at least for the distributed systems*.


Thanks, Cheers & Happy Glogging.
