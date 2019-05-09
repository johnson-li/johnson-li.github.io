---
title: "A Generic Radix Sorting Algorithm"
date: 2019-05-10
categories:
  - Datatable
---

As we know, a comparasion-based sorting algorithm can never run faster than `o(n * log(n)) time` in the average case. 
But a non-comparative sorting algorithm, E.g, radix sort, is not limited by such constraint. 
The basic idea is that we can put candidates into a number of bins. 
The bins are sorted by nature and the candidates that fall into the same bin are regarded of the same value. 
So we just need to collect the condidates in the order of the bins, which can be completed within `o(n) time`.  
Such algorithm is called **bucket sort**.

But considering that candidates may be of a wide range and we do not want to manuplate lots of bins. 
We can use a small number of bins to sort the candidates according to the most significant bits at first. 
The result is that the candidates are partially sorted. Then we perform the same sort algorithm inside each bins to sort them totally.
Such algorithm is called **radix sort**, or **MSB radix sort** more precisely.

Radix sort is fast and simple, but the most challenge part is how to deal with different data types.
We need to find out suitable 'bins' for different types, I.e., integer, float and string. 
The algorithm presented below is a simplified version of [datatable](https://github.com/h2oai/datatable).

// TO BE CONTINUED
