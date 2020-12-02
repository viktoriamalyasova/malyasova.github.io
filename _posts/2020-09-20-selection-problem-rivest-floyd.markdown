---
layout: post
title:  "Selection Problem: Rivest-Floyd algorithm"
date:   2020-09-20 14:17:28 +0300
categories: algorithms
---
The idea of the Rivest-Floyd selection algorithm is that if we select a subset of an array and find, say, its median, it will be usually pretty close to the original array's median, and, therefore, a good choice for a pivot. For big arrays, Rivest-Floyd algorithm runs twice as fast as quickselect [1].

Here is my implementation of Rivest-Floyd in Python. I need a slightly different partition function this time:

{%highlight python%}
def partition2(A, lo, hi):
    """rearrange A[lo:hi+1] and return j such that
    A[lo:j] <= pivot
    A[j] == pivot
    A[j+1:hi+1] >= pivot
    """
    pivot = A[lo]
    if A[hi] > pivot:
        A[lo], A[hi] = A[hi], A[lo]
    #now A[hi] <= A[lo], and A[hi] and A[lo] need to be exchanged
    i = lo
    j = hi
    while i < j:
        A[i], A[j] = A[j], A[i]
        i += 1
        while A[i] < pivot:
            i += 1
        j -= 1
        while A[j] > pivot:
            j -= 1
    #now put pivot in the j-th place
    if A[lo] == pivot:
        A[lo], A[j] = A[j], A[lo]
    else:
        #then A[right] == pivot
        j += 1
        A[j], A[hi] = A[hi], A[j]
    return j

def rivest_floyd(A, left, right, k):
    assert k < len(A)
    while right > left:
        if right - left > C1:
            #select a random sample from A
            N = right - left + 1
            I = k - left + 1
            Z = math.log(N)
            S = C2 * math.exp(2/3 * Z) #sample size
            SD = C3 * math.sqrt(Z * S * (N - S) / N) * math.copysign(1, I - N/2)
            #select subsample such that kth element lies between newleft and newright most of the time
            newleft = max(left, k - int(I * S / N + SD))
            newright = min(right, k + int((N - I) * S / N + SD))
            rivest_floyd(A, newleft, newright, k)
        A[left], A[k] = A[k], A[left]
        j = partition2(A, left, right)
        if j <= k:
            left = j+1
        if k <= j:
            right = j-1
    return A[k]
{%endhighlight%}

[1] Floyd, Robert W., and Ronald L. Rivest. "Algorithm 489: the algorithm SELECT—for finding the i th smallest of n elements [M1]." Communications of the ACM 18.3 (1975): 173.
