---
layout: post
title:  "Augmented Binary Search Tree"
date:   2020-08-16 14:17:28 +0300
categories: data_structures
---
<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

A **binary search tree (BST)** is a rooted binary tree satisfying the BST property: each node's value is greater or equal to all values in its left subtree, and smaller than the values in its right subtree.

![Binary search tree. Image source: Wikipedia](/assets/binary_search_tree.png)

An **augmented binary search tree** is a binary search tree that stores additional information in its nodes.

In this post I'll show how to use this data structure to solve the following [Leetcode problem][leetcode] :

`You are given an integer array nums and you have to return a new counts array. The counts array has the property where counts[i] is the number of smaller elements to the right of nums[i].`

With the help of an augmented BST, this problem can be solved in $$O(n \log n)$$ average time. Here's how.

In each node, I am going to store its key, its left child, right child and parent pointer, and an additional number lc: number of nodes in the left subtree of the node.
{% highlight python %}
class Node:
    def __init__(self, key, lc=0, left=None, right=None, parent=None):
        self.key = key 
        self.lc = lc #how many nodes are in the left subtree of this node
        self.left = left
        self.right = right
        self.parent = parent
{%endhighlight%}

As I insert a node into the binary search tree, I preserve this property, and use this information to compute how many nodes are to the left of the inserted one:

{%highlight python %}
class AugBTree: # Augmented binary search tree
    def __init__(self):
        self.root = None
    def Insert(self, node):
        y = None
        x = self.root
        st = 0 #how many nodes are smaller than current node
        if x:
            st = x.lc
        while x:
            y = x
            if x.key < node.key:
                if x.right:
                    st = st + x.right.lc + 1
                x = x.right
            else:
                if x.left:
                    st = st - x.lc + x.left.lc
                    x.lc += 1
                x = x.left
        node.parent = y
        if y is None:
            self.root = node
            st = 0
        elif y.key < node.key:
            y.right = node
            st += 1
        else:
            y.left = node  
            y.lc = 1
            
        return st
        
def right_and_smaller(nums):
    tree = AugBTree()
    result = []
    for num in reversed(nums):
        newnode = Node(num)
        result.append(tree.Insert(newnode))
    return reversed(result)
{%endhighlight%}

The average complexity of insertion into a binary tree is $$ O(\log n)$$, and the worst case is $$O(n)$$. Hence the average complexity of this algorithm is $$O(n \log n)$$, and the worst case is $$O(n^2)$$. However, we can do better with a balanced binary search tree, where the worst-case time of insertion is $$O(n)$$. This is the topic of the [next post]({% post_url 2020-08-23-augmented-red-black-tree %}).

[leetcode]: https://leetcode.com/problems/count-of-smaller-numbers-after-self/
