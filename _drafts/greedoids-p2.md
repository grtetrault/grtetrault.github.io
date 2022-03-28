---
layout: post
title:  "Greedoids, Part 2: Putting into practice"
date:   2022-03-24
author: Garrett Tetrault
categories: greedoid greedy algorithm
excerpt_separator: <!--more-->
katex: true
comments: true
publish: true
---

<!-- 
Outline

start with example copying Kruskal's algorithm

show why this is naive (only linear problems)

Give an example of a problem that uses nonlinear problems

Talk a bit more on what weight functions are allowable (can be hand-waving here)

Go into the scheduling problem example
 -->

In the last post, we explored what a greedoid was

An important distinction arises: greedoids identify the structure 

We can easily copy the structure of Kruskal's algorithm from the previous post to form the algorithm:

```python
def greedy(greedoid, weight_fn):
    solution = []
    for element in sorted(greedoid.search_space, key=weight_fn):
        candidate = solution + element
        if candidate in greedoid.feasible_sets:
            solution = candidate
    return solution
```

You may not have noticed but the weight function was hardly mentioned in the previous post.

But this can only account for *linear* weight functions - that is where each element has a weight and we want to minimize (or maximize) them sum.

This is the case for Kurskal's algorithm.
What if, instead, we wanted to find the MST that minimizes maximal [degree](https://en.wikipedia.org/wiki/Degree_(graph_theory))?
This is known as the minimum degree spanning tree problem, one that is known to be NP-complete.
So, while the MST can be viewed as a greedoid under both problems, the weight function is the prohibitive factor.

minimizing `sum(weight_fn(e) for e in greedoid.search_space)`

For what weight functions 