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

In the last post we discussed CAP theorem and saw why it becomes hard to make systems Available and Consistent when there are potential partitions in the systems. I was thinking about my next post back then and finalized Consistent Hashing, But then I came across an idea to first write about HyperLogLog because of its Elegance and Power.

Let's jump into it. I'm just gonna throw a problem at you.
```
We have a stream of elements and we want to maintain the count of distinct elements. How can we solve it?
```

> NOTE: Before proceeding please give it some thoughts. Few hints: Can we do it in Run time. Think of bloom filters. Think of approximating it. Think of upper bounding it, cuz every finite set must have finite distinct elements, right?

Initially, All of this started with counting, The above example is just an abstraction. The real use case would be like -
```
I have a webpage and I want to maintain a counter of the distinct IPs that have landed on the page.
(Also the page is distributed over multiple servers)
```
Let's start solving it.

#### Famework
We'll take these points into consideration when we'll solve the problem with multiple approaches -
- Space Complexity
- Time Complexity
- Estimation error

Some fundamental operations are -
- Load(x) : Store `x` in the memory.
- Lookup(x, ds) : lookup `x` in the `ds` data structure (memory).
- get_new_elem() : get a new element.
- len(L) : return the length of a `L` which can either be a list of a Map.
- dump(x, d) : dump the `x` data into the `d` data store (disk).
- hash(x) : hash `x` into a 6 bit binary hash.
  e.g. hash("A") = 001011, hash("B") = 001001
- f_k_bits(x) : get the `first k` bits of binary `x`.
  e.g. f_3_bits (00101101) = 001, f_5_bits (00101001) = 00101
- l_k_bits(x) : get the `last k` bits of binary `x`.
  e.g. l_3_bits (00101101) = 101, l_5_bits (00101011) = 01011
- rho(x) : return the position of `first 1 bit from left` of binary x.
  e.g. rho(001101) = 3, rho(010101) = 2
- Max_bits(x,y): return the maximum between 2 binary strings.
- Sum(L) : return the sum of all elements of the list `L`.

Some Assumptions are -
- We're getting a stream of elements.


#### Approach 1: Store the elements in the disk.

Whenever a new element comes, dump it into the big data store, and rerun the [count distinct alogrithm](https://www.geeksforgeeks.org/count-distinct-elements-in-an-array/) -
```
1. def distinct(L):
2.    counter = 1
      # loop 1
3.    for i in range(1,len(L)):
          # loop 2
4.        for j in range(0,i):
5.            if L[i]==L[j]:
6.                break
7.        if i==j+1:
8.            counter+=1
9.    return counter
10.
11. def main():
12.     while True:
13.         new_elem = get_new_elem()
14.         dump(new_elem, D)
            # Space Complexity = O(n)
15.         L = Load(D)
            # Time Complexity = O(n^2)
16.         crdnlty = distinct(L)
17.
18. main()
```
Approach 1: It is a trivial approach. It stores the elements into the disk and it has to load the elements into the memory every time it needs to calculate the Cardinality.

Space Complexity | Time Complexity | Estimation Error %
-----------------|-----------------|-------------------
    O(n)         | O(n^2)          | 0

#### Approach 2: Maintain a Hash Map

```
1. def distinct(new_elem, _hMap):
      # Lookup in Hashmap = O(1)
2.    if Lookup(new_elem,_hMap):
3.        _hMap[new_elem] += 1
4.    else:
5.        _hMap[new_elem] = 1
6.    return len(_hMap)
7.
8. def main():
      # A Hash Map, Space C - O(n)
9.    _hMap = {}
10.   while True:
11.       new_elem = get_new_elem()
12.       distinct(new_elem, _hMap)
          # Time Complexity = O(1)
13.       crdnlty = distinct(new_elem,_hmap)
14.
15. main()
```

Approach 2: Maintain a hashmap in memory. So whenever a new element comes, it looks up the element in the map. If the element exists then it updates the count of the element. If the element doesn't exist then it adds the element in the map and initializes its count with 1. The length of the map is the Cardinality of the set which we are looking for.


Space Complexity | Time Complexity | Estimation Error %
-----------------|-----------------|-------------------
    O(n)         | O(1)            | 0


#### Approach 3: LogLog Estimation
```
1. def addToLogLog(new_elem, LL):
2.    hash_elem = hash(new_elem)
3.    frst_2_bits = f_2_bits(hash_elem)
4.    last_4_bits = l_2_bits(hash_elem)
5.    old_max = LL.get(frst_2_bits)
6.    rho_elem = rho(last_4_bits)
7.    new_max = Max(old_max, rho_elem)
8.    LL.update( {first_2_bits: new_max})
9.
10. def getLLEstimate(LL):
11.    sumLL = Sum(LL.values())
12.    lenLL = len(LL)
13.    arith_mean = sumLL/lenLL
14.    LLEstimate = 2**arith_mean
15.    return LLEstimate
16.
17. def main():
      # Hash Map of Space Cmplxty - O(m)
      # `m` is the number of buckets
18.   LL = { '00':0, '01':0,
19.          '10':0, '11':0 }
20.
21.   while True:
22.       new_elem = get_new_elem()
23.       addToLogLog(new_elem, LL)
          # Time Complexity = O(m)
24.       crdnlty = getLLEstimate(LL)
25.
26. main()
```

Approach 3: A lot is going on in the third approach. Let's get the intuition first, and then we can walk through an example to get the Cardinality of a stream using this method.

##### Intuition:

The idea behind LogLog Estimation is to
- Map the arbitrary stream to a uniformly distributed hash function,
- Observe the bit pattern of the hash seen so far,
- Make an Estimation attempt assuming the uniform distribution.

*In other words*
- Let's say We got an element - "A",
- We hashed it i.e. hash("A") - '001010' (assumption),
- The rho(x) in the algorithm is the bit pattern,
- rho('001010') = 3 : the position of `first 1 bit from left`,
- Estimation attempt: Cardinality = 2^3 i.e. 8
- The Estimation is **wrong** as we have only seen "A" so far.
- Continue Reading for an explanation :).

Park the above Estimation aside for a while, I know you must have some questions like - from where this exponent is coming into the picture, why did we hash our elements, etc.

Quoting from the LogLog Paper -
> For a string x ∈ {0,1}∞, Let ρ(x) denotes the position of its first 1-bit. Thus ρ(1...) = 1, ρ(001...) = 3, etc. Clearly we expect about n/2ᵏ amongst the distinct elements of the set to have a ρ value equal to k.

```
Let's look at it this way :
Part A:
Consider 2 binary strings - A = '1 _ _ _ _ _' and B = '0 _ _ _ _ _'.
We can see that the first bit divides the set of all possible strings of length 6 into two halves. Similarly the first 2 bits divide the set of strings into 4 equal subsets.
That's why we use an exponent 2.

Part B:
The large value of the ρ indicates a large number of distinct elements. Because of the uniform distribution of the hash function, we can say that the large ρ implies the high probability of having a large number of distinct elements.

Part C:
Our Estimation was wrong because the paper clearly states that the algorithm gives asymptotic good results i.e. as the number of elements in the stream reaches ∞, the estimation error reaches 1.30/√m where m is the number of buckets.

Part D.
In our algorithm we've used 6-bit hash function, and bucket size of 4, but the usual numbers are 32-bit or 64-bit hash functions and 1024 or 2^10 buckets. The paper talks about many design decisions for optimizations.
```

##### Example Walkthrough:
Stream = ['a', 'b', 'c', 'd', 'e', 'f', 'b', 'c', 'e'].

Hash of stream = [101110, 100010, 010001, 110010, 010010, 001001, 100010, 010001, 010010]

HLL buckets | elem | ρ
------------------------|-----------|--
00:0, 01:0, 10:0, 11:0  | 10 11 10  | 1
00:0, 01:0, 10:1, 11:0  | 10 00 10  | 3
00:0, 01:0, 10:3, 11:0  | 01 00 01  | 4
00:0, 01:4, 10:3, 11:0  | 11 00 10  | 3
00:0, 01:4, 10:3, 11:3  | 01 00 10  | 3
00:0, 01:4, 10:3, 11:3  | 00 10 01  | 1
00:1, 01:4, 10:3, 11:3  | 10 00 10  | 3
00:1, 01:4, 10:3, 11:3  | 01 00 01  | 4
00:1, 01:4, 10:3, 11:3  | 01 00 10  | 3
00:1, 01:4, 10:3, 11:3  | -  | -

Now the Estimate is ~ `2^(11/4) i.e. 2^2.8 ~ 7` which is pretty a good estimate as we have 6 distinct elements in our stream.

Space Complexity | Time Complexity | Estimation Error %
-----------------|-----------------|-------------------
O(m) few KBs     | O(m)  constant  | 1.30/√m


##### Cardinality Estimation over a distributed system.
We can see that Approach 1 and Approach 2 are non-scalable solutions. If we increase the number of elements in the stream to 2^32, then (assuming each element takes k bytes) we'll need a memory of size 2^32*k bytes which are roughly K GBs. On the contrary, the LogLog uses few KiloBytes of Memory.
In a distributed system, the Total Cardinality of the streams seen by all the components of the system can be calculated as -
Assuming there are D machines in a distributed system.
Each machine maintains its LogLog Data structure as given below.

    HLL_1 = {x1: a11, x2:a12, x3:a13, .. , xk:a1k}

    HLL_2 = {x1: a21, x2:a22, x3:a23, .. , xk:a2k}
    .
    .
    .

    HLL_D = {x1: aD1, x2:aD2, x3:aD3, .. , xk:aDk}

    We can easily merge them into HLL_Total by taking a component-wise maximum of all elements i.e.

    HLL_Total = {x1: max(a11, a21, ..., aD1),

                 x2: max(a12, a22, ..., aD2), ...

                 xD: max(a1k, a2k, ..., aDk)}
Thus we can get the Cardinality Estimation of a distributed system with similar Time and Space Complexities.


##### Why named LogLog??
I think by now you must've got the idea why it is named LogLog.
Assuming we are counting till 2^32 ( log2^32(base 2) = 32). Assuming 2^10 i.e. 1024 number of buckets. We are left with 22 digits (after 10 digits) in a 32-bit binary string.
To store the ρ of 22-bit string we only need log(22) bits because in a 22-bit string the maximum possible value of ρ can be 22 and this number can be stored in the memory using log(22) bits i.e. ~ 5. This is the reason why the Space Complexity of this algorithm is constant (a few KBs).

##### Some Notes on SuperLogLog and HyperLogLog.
**SuperLogLog**: The LogLog paper talks about a few engineering optimizations like Truncation Rule and Restriction Rule which when combined with the LogLog algorithm yields a new algorithm, SuperLogLog
1. Truncation Rule: This rule simply states that when we are calculating the arithmetic mean of the HLL map, just drop the top k % values. The optimal value of k is found to be 30% for the best estimation.

If HLL_1 = {x1: a1, x2:a2, x3:a3, .. , x100:a100} (sorted on a), then drop 0.3k elements from the top i.e. HLL_1 = {x31: a31, x32:a32, x33:a33, .. , x100:a100}

2. Restriction Rule: This rule bounds the number of buckets to be used.

**HyperLogLog**: The HyperLogLog uses a Harmonic mean instead of Geometric Mean. If HLL_1 = {x1: a1, x2:a2, x3:a3, .. , x100:a100} then the estimation becomes ~ ( 2^-a1 + 2^-a2 + 2^-a3 + ... + 2^-a100) / m^-1.


##### Conclusion
Having LogLog Cardinality Estimation in our bag of algorithms now we can estimate cardinalities without worrying about space. Our webpage now can be hosted on a machine having few gigs of memory with maintaining the count of distinct users that have landed on the page. With the emergence of IoT the number of IPs will grow like crazy, from IPv4 to IPv6. The HyperLogLog will save the day.



Thanks, Cheers & Happy Glogging.

#### References
- [The Original LogLog Paper](https://www.ic.unicamp.br/~celio/peer2peer/math/bitmap-algorithms/durand03loglog.pdf)
- [The HyperLogLog Paper: An improvement on LogLog](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf)
- [Google's Paper on furthur improvements](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/40671.pdf)

#### Cool Videos to watch.
---

{{< youtube y3fTaxA8PkU >}}

<br>

---

{{< youtube jD2d7jr7z1Q >}}

<br>
