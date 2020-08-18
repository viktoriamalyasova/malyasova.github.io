---
layout: post
title:  "Welcome to Jekyll!"
date:   2020-08-17 14:17:28 +0300
categories: jekyll update
---
In this series of posts, I am going to demonstrate how to use several advanced data structures to solve the following [Leetcode problem][leetcode]:

You are given an integer array nums and you have to return a new counts array. The counts array has the property where counts[i] is the number of smaller elements to the right of nums[i].

It is easy to write an $\mathbb O(n^2)$ solution:

{%highlight python %}
def right_and_smaller(nums):
    result = []
    for i in range(len(nums)):
        rs = 0
        for j in range(i+1, len(nums)):
            if nums[j] < nums[i]:
                 rs += 1
        result.append(rs)
    return result
{% endhighlight %}

Now I am going to use an augmented binary search tree to solve it in $\mathbb O(n \log n)$.

[leetcode]: https://leetcode.com/problems/count-of-smaller-numbers-after-self/
