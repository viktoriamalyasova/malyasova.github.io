---
layout: post
title:  "Augmented Red-Black Tree"
date:   2020-08-18 14:17:28 +0300
categories: data_structures
---
<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

In the [previous post]({% post_url 2020-08-16-righter-and-smaller %}), I used an augmented binary search tree to solve the following problem in $$O(n \log n)$$ average time:

`You are given an integer array nums and you have to return a new counts array. The counts array has the property where counts[i] is the number of smaller elements to the right of nums[i].`

In this post, I'll modify the algorithm to work for a **self-balancing binary search tree**. A self-balancing BST is a BST that always keeps its height small - of the order of $$O(\log n)$$. Insertions into a balanced BST always take $$O(\log n)$$ time, and thus my algorithm will have $$O(n \log n)$$ worst-case complexity. One kind of self-balancing BSTs are red-black trees.

A **red-black tree** is a binary search tree where each node stores stores an additional color attribute, which satisfies the following properties:

1. Each node can be either red or black.
2. The root is black.
3. All leaves are black and have NIL value.
4. All simple paths from the root to leaves have the same number of black nodes.
5. No two red nodes are adjacent.

Thanks to properties 4 and 5, a red-black tree with $$n$$ nodes has height at most $$2(\lg n + 1)$$, so insertion into a red-black tree has a worst-case $$ O (\log n) $$ complexity. 

After a new node is inserted, the red-black tree goes through a *fixup procedure* to ensure that the properties 1-5 are fulfilled. I am not going to describe it in detail; you can read about it in [Wikipedia](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree) or examine the code below. The important thing about the fixup procedure is that it only uses two basic operations: recoloring nodes and rotation.

The rotation operations on a binary search tree works as follows:
![Left and right rotation](/assets/rotation.png)

The operation **LeftRotate** transforms the configuration of the two nodes on the right into the configuration on the left. The inverse operation **RightRotate** transforms the configuration on the left into the configuration on the right. The letters $$\alpha$$, $$\beta$$ and $$\gamma$$ represent arbitrary subtrees.

In the previous post, we augmented a node with an "lc" field to store the number of nodes in the lest subtree:
{% highlight python %}
class Node:
    def __init__(self, key, lc=0, left=None, right=None, parent=None):
        self.key = key 
        self.lc = lc #how many nodes are in the left subtree of this node
        self.left = left
        self.right = right
        self.parent = parent
{%endhighlight%}

To augment a red-black tree the same way, we need to ensure that this field is correctly updated during the fixup operation. Recoloring doesn't change this number, and during rotation we only need to update the y node.

For the right rotation,
{% highlight python %}
y.lc = y.lc - x.lc - 1
{% endhighlight %}
For the left rotation,
{% highlight python %}
y.lc = y.lc + x.lc + 1
{% endhighlight %}

The full algorithm is below:
{% highlight python %}
RED = "RED"
BLACK = "BLACK"
class AugNode: #Augmented node of a Red-black tree
    def __init__(self, key, color, lc=0, left=None, right=None, parent=None):
        self.color = color
        self.key = key
        self.lc = lc
        self.left = left
        self.right = right
        self.parent = parent
        
class AugRBTree:
    def __init__(self):
        self.Nil = AugNode(None, BLACK, 0) 
        self.root = self.Nil
        
    def LeftRotate(self, node):
        y = node.right
        node.right = y.left
        y.left = node
        y.lc = node.lc + y.lc + 1
        if node.right != self.Nil:
            node.right.parent = node
        y.parent = node.parent
        if node.parent == self.Nil:
            self.root = y
        elif node == node.parent.left:
            node.parent.left = y
        else:
            node.parent.right = y
        node.parent = y
        
    def RightRotate(self, node):
        y = node.left
        node.left = y.right
        y.right = node
        node.lc = node.lc - y.lc - 1
        if node.left != self.Nil:
            node.left.parent = node
        y.parent = node.parent
        if node.parent == self.Nil:
            self.root = y
        elif node == node.parent.right:
            node.parent.right = y
        else:
            node.parent.left = y
        node.parent = y
        
    def Insert(self, node):
        node.lc = 0
        y = self.Nil
        x = self.root
        st = x.lc
        while x != self.Nil:
            y = x
            if x.key < node.key:
                if x.right != self.Nil:
                    st = st + x.right.lc + 1
                x = x.right
            else:
                if x.left != self.Nil:
                    st = st - x.lc + x.left.lc
                    x.lc += 1
                x = x.left
        node.parent = y
        if y == self.Nil:
            self.root = node
        elif y.key < node.key:
            y.right = node
            st += 1
        else:
            y.left = node  
            y.lc += 1
        
        node.color = RED
        node.left = self.Nil
        node.right = self.Nil
        self.InsertFixup(node)
        return st
        
    def InsertFixup(self, z):
        while z.parent.color == RED:
            if z.parent == z.parent.parent.left:
                uncle = z.parent.parent.right
                if uncle.color == RED:
                    #recolor
                    uncle.color = BLACK
                    z.parent.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    #if z is a right child turn it into a left child
                    if z == z.parent.right:
                        z = z.parent
                        self.LeftRotate(z)
                    z.parent.color = BLACK
                    z.parent.parent.color = RED
                    self.RightRotate(z.parent.parent)
            else:
                uncle = z.parent.parent.left
                if uncle.color == RED:
                    #recolor
                    uncle.color = BLACK
                    z.parent.color = BLACK
                    z.parent.parent.color = RED
                    z = z.parent.parent
                else:
                    if z == z.parent.left:
                        z = z.parent
                        self.RightRotate(z)
                    z.parent.color = BLACK
                    z.parent.parent.color = RED
                    self.LeftRotate(z.parent.parent)
        self.root.color = BLACK
        
def right_and_smaller2(nums):
    tree = AugRBTree()
    result = []
    for num in reversed(nums):
        newnode = AugNode(num, BLACK)
        result.append(tree.Insert(newnode))
    return reversed(result)
{%endhighlight %} 

After moving a little further in studing data structures, I realized my invention closely resembles an [Order Statistic Tree](https://en.wikipedia.org/wiki/Order_statistic_tree)

An **Order Statistic tree** is a self-balancing binary tree where each node stores the total number of nodes below it, including itself (the NIL nodes' size is defined as zero). An Order Statistic tree allows to 

- insert elements in $$O(\log n)$$ time
- find the rank of an element in $$O(\log n)$$ time
- find the kth smallest element in $$O(\log n)$$ time

So, an order statistic tree would serve just as well for solving our problem.

In the [next post]({% post_url 2020-08-30-fenwick-tree %}), I'll describe another way to efficiently solve this problem with a different data structure: Fenwick tree.
