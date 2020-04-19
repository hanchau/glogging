+++
author = "hanchau"
title = "HyperLogLog!"
date = "2020-04-19"
description = "Elegance of LogLog/HyperLogLog, A Cardinality Estimation Algorithm"
tags = [
    "LogLog",
    "HyperLogLog",
    "Cardinality Estimation Algorithm",
]
categories = [
    "posts",
    "concepts",
]
+++

This article offers a basic understanding of a Cardinality Estimation Algorithm.
<!--more-->

#### Cardinality

Quoting [wikipedia](https://en.wikipedia.org/wiki/Cardinality), the Cardinality is nothing but "the number of elements in a given set".

#### Cardinality Estimation Algorithm

In last post we discussed CAP theorem and saw why it becomes hard to make systems Availabile and Consistent when there are potential partitions in the systems. I was thinking about my next post back then and finialized Consistent Hashing, But then I came across an idea to first write about HyperLogLog beacause of its Elegance and Power.

Let's jump into it. I'm just gonna throw a problem at you.
```
We have a list of elements and we want to count the number of distinct elements in that list. How can we solve it?
```

> NOTE: Before proceeding please give it some thoughts. Few hints: Can we do it in one parse. Think of bloom filters. Think of approximating it. Think of upper bounding it, cuz every finite set must have finite distinct elements, right?

All of this initially started with counting, The list example is just to simplify it. The real use case would be to estimate big number like-
```
I have a webpage and I want to count the distinct IPs that have landed on the page.
```
Let's sart solving it.

##### Famework
We'll take these points in consideration when we'll solve the problem with multiple approaches -
- Space Complexity
- Time Complexity
- Estimation error

Some fundamental operations are -
- Load(x) : Store x in the memory.
- Lookup(x,map): lookup x in a map.


##### Approach 1: Maintain a Hash Map

```
1. def distinct(L):
2.     _hMap = {}                         # A Hash Map
3.     for i in L:
4.         if Lookup(i,_hMap):
5              _hMap[i] += 1
6.         else:
7.             _hMap[i] = 1
8.     return len(_hMap)
9.
10. L = Load(element)                     # Space Complexity = O(n)
11. crdnlty = distinct(L)                 # Time Complexity = O(n)
```
Approach 1 simply maintains a hasmap in the memory. So whenever a new element comes, it looks up the element in the map and if it exists then it updates the count of that element and if the element doesnt exist then it updates the hasmap with that element.


Space Complexity | Time Complexity | Estimation Error %
-----------------|-----------------|-------------------
    O(n)         | O(n)            | 0


Thanks, Cheers & Happy Glogging.
