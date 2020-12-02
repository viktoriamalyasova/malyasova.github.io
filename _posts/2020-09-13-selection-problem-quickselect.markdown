---
layout: post
title:  "Selection Problem: Quickselect"
date:   2020-09-13 14:17:28 +0300
categories: algorithms
---
<!-- Mathjax Support -->
<script type="text/javascript" async
  src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

Selecting kth biggest element in an array can be done in $$O(n)$$ worst-case time. The algorithm is called median-of-medians and described in [1]. The constant before n is pretty big for this algorithm, so in practice people use faster algorithms like quickselect or introselect.

Quickselect has an average complexity of $$O(n)$$ and a worst-case of $$O(n^2)$$.
It works like this:
>1. Select a random pivot
>2. Partition the array around the pivot
>3. If the pivot ended up in kth place, return it
>4. Otherwise, call quickselect on the subarray which must contain the kth element

In educational courses, step 2 is often implemented as [Lomuto partition scheme](https://en.wikipedia.org/wiki/Quicksort#Lomuto_partition_scheme). In the code below, I implement the more efficient [Hoare partition scheme](https://en.wikipedia.org/wiki/Quicksort#Hoare_partition_scheme).

{%highlight python%}
def partition(A, lo, hi):
    #rearrange A and return j such that
    #A[:j+1] <= pivot
    #A[j+1:] >= pivot
    pivot = A[(lo + hi)//2]
    i = lo - 1
    j = hi + 1
    while True:
        i += 1
        while A[i] < pivot:
            i += 1
        j -= 1
        while A[j] > pivot:
            j -= 1
        if i >= j:
            return j
        A[i], A[j] = A[j], A[i]
        
def quickselect(A, left, right, k):
    if left >= right:
        return A[left]
    pivotIndex = partition(A, left, right)
    if k <= pivotIndex:
        return quickselect(A, left, pivotIndex, k)
    else:
        return quickselect(A, pivotIndex + 1, right, k) 
{%endhighlight%}

Introselect[2] is an algorithm that combines the speed of quickselect and worst-case linear time of median-of-medians. The gist is that we start running quickselect, but if we notice we've been through too many iterations, we switch to median-of-medians.

In the [next post]({% post_url 2020-09-20-selection-problem-rivest-floyd %}), I'll describe another way to speed up quickselect.

[1] Cormen, Thomas H.; Leiserson, Charles E.; Rivest, Ronald L.; Stein, Clifford (2009) [1990]. Introduction to Algorithms (3rd ed.). MIT Press and McGraw-Hill. ISBN 0-262-03384-4., pp.220-223

[2] Musser, David R. "Introspective sorting and selection algorithms." Software: Practice and Experience 27.8 (1997): 983-993.
