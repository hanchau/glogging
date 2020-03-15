+++
author = "hanchau"
title = "Parallel processing!"
date = "2020-03-08"
description = "Why parallel processing comes into the picture?"
tags = [
    "parallel systems",
    "parallel processing",
    "gearman",
]
categories = [
    "posts",
    "concepts",
]
+++

This article offers a basic understanding of the parallel systems.
<!--more-->

#### Why Parallel Processing?.

[Gearman Job Server](http://gearman.org/) is the first parallel/distributed processing framework that I've learned/worked on which is indeed a very old framework out there.
One of the things I like about Gearman is that it's very generic and you can play around a lot.
In the last post I wrote how to setup the gearman job server and in this post I'm going to brief you why we need parallel processing with some examples:

**Example 1**

Let's say we have 10000 colored balls having red, green and blue colors and 3 buckets of each color. The task is to segregate the balls to the respective colored buckets.

1 Approach: You pick a ball, look at the color, put it inside the same colored bucket and repeat.
Time Taken: Assuming T to be the time taken to segregate one ball, total time would be **(10000*T)**.


2 Approach: Call K number of friends if possibles, Get K number of beers, Divide the balls among y'all so that each one of you will process 10000/K number of balls only, Everyone follow the I Approach.
Time Taken: Assuming T to be the time taken to segregate one ball, total time would be **(10000*T/K)**.



**Example 2**

Again, Let's say we have 10000 balls and 3 buckets A, B, C with A having clean water, B having soap water, C having clean water. The task is to clean all the balls pun intended :D.

1 Approach: You pick a ball, put it inside A, then B and then C and repeat for all the balls.
```
NOTE: when you are using one bucket, others are not in use or idle.
```
Time Taken: Assuming T/3 to be the time taken to process a ball through one bucket, total time would be ((T/3 + T/3 + T/3)10000) ~ **(10000*T)** .


2 Approach: Call your two best friends Alex and Bob, Assign each task of using bucket A, bucket B, bucket C to y'all.
Now what you will do is just pick a ball, dip it in bucket A and give it to Alex and repeat.
Alex will dip it in bucket B and will pass the ball to the Bob and repeat.
Bob will dip it in bucket C and Voilla, One ball is cleaned.
Time Taken: Assuming T/3 to be the time taken to process a ball through one bucket, but now there's a **catch**
```
NOTE: when you are using one bucket, others are being used as well.
After everty T/3 time, one ball is getting cleaned which makes the effective time taken to clean a ball is T/3 +- some error;
```
total time would be ~ **(10000*T/3)**.


So, I think I've convinced you that parallel processing is gooood.
```
NOTE: Parallel processing can only be done with some specific types of tasks. Some tasks are inherently sequential. A fine example would be making a cup of tea, you have to follow the steps. One way to identify if a task can be parallelized or not is that if there is a some sort of repetition or lack of system resource utilization (like in example 2) then you can think of making the task into sub tasks and processing them parallelly.
```

#### How Gearman helps you achieve that?.

In the above examples we saw how we can do tasks in parallel ways. Let's say we have a task which can be divided into K subtasks. To complete the task what we can do in gearman is create a worker which will process one subtask at a time. Now just instantiate P number of workers and submit all the K tasks to the gearman job server which will keep them in a queue. What you will see is that K number of subtasks are being processed parallelly i.e. every worker is processing one subtask. Hurrray!! We just processed our task parallelly!!
