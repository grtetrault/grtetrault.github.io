---
layout: post
title:  "Greedoids: When Greedy Algorithms Work"
date:   2021-10-12 18:15:58 -0400
author: Garrett Tetrault
categories: greedoid greedy algorithm
excerpt_separator: <!--more-->
mathjax: true
comments: true
---

It can sometimes feel nebulous what problems are solved with a greedy algorithm. 
A greedy algorithm builds up a solution by making the locally optimal choice, but when can this be done to find the globally optimal solution?
<!--more-->

This family of algorithms is often introduced by way of a zoo. 
We have Kruskal's algorithm, then Dijkstra's, followed by Huffman codes and other canonical (and sometimes counter) examples. <!-- REF NEEDED -->
While the similarities in processes is made plain, the job of finding the similarities in the *problems* is often left to the reader.
After all, why should finding the shortest path and determining the most efficient prefix code be solved by the same type of algorithm?
What properties do these problems share?

Greedoids, a topic in combinatorics, can shed some light on these similarities.
In fact, it's right there in the name!
*Greed-* as in "greedy algorithm" and *-oid* as in "resembling" or "like".
Greedoids provide a useful framework for thinking about optimization.
They make explicit common properties that allow problems to be optimized by greedy algorithms.

In this article we'll take a look at these properties and then walk through an example of how you'd use greedoids to find a greedy algorithm.
Before we get to that, though, we have to answer the question...


## What is a Greedoid?

At its core, a greedoid consists of a set $$S$$ which serves as the search space for a problem.
For example, in finding a minimum spanning tree, $$S$$ is the set of edges of the graph.

In our search space, there are some subsets that can contribute to a solution and some that cannot.
These are called feasible sets and the collection of all feasible sets is denoted by $$F$$.
In this context, solutions are maximal feasible sets and are typically what you'd like to find [^solution-basis].
Returning to minimum spanning trees, $$F$$ would consist of all (not necessarily spanning) trees and
the set of solutions would be the collection of all spanning trees [^feasible-prim-kruskal].

[^solution-basis]: In greedoid theory, "solutions" are referred to as "bases", however this language is not being used here to avoid a a digression into where this terminology comes from. For interested readers, this is borrowed from matroid theory, which you can learn more about here. <!-- REF NEEDED -->
[^feasible-prim-kruskal]: For the minimum spanning tree problem, $$F$$ can be either the collection of all trees or all forests. The former corresponds to Prim's algorithm, while the latter to Kruskal's.

<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/spanning-tree-feasibility.png"/>
<figcaption>
    <div class="centered-figcaption">
        Examples of feasible sets (green) and non-feasible sets (red) in the spanning tree problem. 
        Feasible sets must be connected and contain no cycles.
        Note the leftmost is a solution.
    </div>
</figcaption>
</figure>

For these two sets to make a greedoid, they must satisfy the following two properties:
1. For every $$X \in F$$, there must be some $$x \in X$$ such that $$X \setminus \{x\} \in F$$,
2. For every $$X, Y \in F$$ such that $$\lvert X \rvert > \lvert Y \rvert$$, there is some $$x \in X$$ such that $$Y \cup \{x\} \in F$$.

There is a rich theory derived from these statements, with some powerful results. 
Some are particularly useful for identifying greedoids.
For example, an implication of the former property is that the empty set is feasible;
an implication of the latter is that all solutions have the same number of members.
<!-- Not loving this transition. -->

I'll digress here from combinatorics to try to make greedoids more concrete.
Despite being interesting in their own right, the objective here is to see how greedoids can help to understand greedy algorithms.
To that end, let's take a look at what a greedy algorithm over a greedoid looks like.
Fortunately, greedoids have a prototypical implementation that is quite expressive. <!-- REF NEEDED -->
Below, we have a partial `python` implementation of this algorithm to serve as pseudo-code:
```python
# Here, T is some generic type (e.g. edges in a graph).
def greedy_algorithm(greedoid: Greedoid[T], weight_fn: Callable[[T], int]):
    solution = []
    for s in sorted(greedoid.search_space, key=weight_fn):
        if solution + [s] in greedoid.feasible_sets:
            solution.append(s)
    
    return solution
```
You may think this algorithm looks pretty inefficient, and you'd be right!
Most greedy algorithms you've seen are optimizations on this simple approach, 
either by reducing the search space, using more appropriate data structures, or by using a function to determine feasibility instead of precomputing $$F$$ (which can quickly get big).
Still, this algorithm is optimal for *any* greedoid.
We won't go into a proof of this optimality here, <!-- REF NEEDED -->
but what is important to take away is this: 
**if a problem satisfies these properties, an optimal solution can be found using a greedy algorithm.**

Still though, it is not immediate what these properties look like.
There's a lot of meaning packed into the set notation above, but that meaning isn't readily apparent.
Moreover, finding the greedoid (or determining that there isn't one) in a problem isn't always an easy task.
Having a firm understanding of the criterion make this process much easier.
So, to develop a better understanding, let's break these properties down one by one.
<!-- TODO flow into next section... -->

### Accessible Set Systems

Any set that satisfies the first property is known as an *accessible set system*.
So in our case, $$F$$ would be an accessible set system.
And it's aptly named;
the word accessible can be defined as "possible to get", which is exactly what the property insures. <!-- REF NEEDED -->
To expand on this, we'll first restate the property:

1. For every $$X \in F$$, there must be some $$x \in X$$ such that $$X \setminus \{x\} \in F$$,

In a literal translation, it is saying that "all feasible sets contain at least one, smaller feasible set".
This implies that all solutions contain solutions to smaller problems, which contain solutions to even smaller problems, and so on. 
For each solution, this process will eventually lead to the empty set.
Recalling our algorithm, we initiate `solution = []` -- the empty set is our starting point!
This means that we can reach every solution in $$F$$ starting from the empty set and iteratively expanding, just as we do in a greedy algorithm.
As the minimal solution can be any solution depending on the weights, this ensures to us that the minimal solution is always *accessible*.

<!-- TODO image of rooted spanning tree getting smaller. -->
<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/spanning-tree-accessibility.png"/>
<figcaption>
    <div class="centered-figcaption">
        Examples of feasible sets in the spanning tree problem;
        edges are being incrementally taken away while maintaining feasibility (from left to right). 
        This process eventually leads to the empty set.
    </div>
</figcaption>
</figure>

This, however, is not a property unique to greedoids.
Many problems have this property that aren't greedoids and aren't optimally solved by greedy algorithms.
In fact, accessible set systems are also a requirement for dynamic programming (another large class of algorithms). <!-- REF NEEDED -->
So, more restrictions are needed.

### The Exchange Property

This brings us to the second property, called the *exchange property*.
In the way it functions, though, it resembles more of a donation; the set $$X$$ is giving up a member, and the set $$Y$$ is gaining one.
The property is as follows: <!-- REF NEEDED -->

<ol start="2">
<li><p>For every \(X, Y \in F\) such that \(\lvert X \rvert > \lvert Y \rvert\), there is some \(x \in X\) such that \(Y \cup \{x\} \in F\).</p></li>
</ol>

The exchange property states that, if we have two feasible sets, we can always find a member in the larger to adjoin to the smaller that preserves feasibility.
As solutions are maximal, this implies that any feasible set can keep growing until it becomes a solution.
Moreover, as the property only takes into account size, the most recent addition to a feasible set doesn't matter.
At any iteration of our greedy algorithm, we can make any choice (remembering that our choices must preserve feasibility) and still be guaranteed to reach a solution. 
Now that's a pretty powerful result!

<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/spanning-tree-exchange.png" style="width:60%"/>
<figcaption>
    <div class="centered-figcaption">
        A non-maximal feasible set \(Y\) (right) in the spanning tree problem receiving an edge \(x\) (light green) from a solution \(X\) (left) that maintains feasibility. 
    </div>
</figcaption>
</figure>

The problem now becomes: how can you tell if something is a greedoid?
We'll walk through this process in the following section to illustrate.


<!-- In greedoids, there is no chance to get stuck in a dead end.

Recalling that the empty set is feasible, 

Let's start by making some observations about greedy algorithms.
In a greedy algorithm, our choices are critical.
As there is no going back to fix bad decisions, we must be confident that our chosen starting point can yield a solution.
Otherwise, we could get stuck in  a dead end. -->

<!-- TODO image showing getting stuck in dead end. -->

<!-- On the flip side, we always start with the lowest weight member of the search space.
As we can assign weights however we want, any member can be our starting point. 
For example, let $$s \in S$$ be our desired starting point.
Then, for some weight function $$w:S \to \mathbf{Z}$$, we need only set $$w(s)=0$$ and $$w(t)=1$$ for all $$t \in S, \; t \neq s$$ for $$s$$ to be chosen first. -->

<!-- TODO image showing spanning trees generated from different starting edges. -->

<!-- Putting these two together, we can start anywhere in the search space and build up to a solution.
The exchange property encompasses this, but takes it one step further.
At any iteration of our greedy algorithm, any choice we make (remembering that our choices must preserve feasibility) will lead to a solution. -->

## An Example: Huffman Codes

Until now, I've only mentioned how greedoids look for the minimum spanning tree problem. 
Spanning trees lend themselves well to visualizing and many problems resemble their structure.
To show more of the power of greedoids, I'd like to walk through an example of a problem that feels very different: huffman codes.

The problem is stated as follows.
We are given a set of symbols $$A = (a_1, a_2, \ldots, a_n)$$ with and weight function $$w: A \to \mathbf{R}^+$$ with positive real outputs.
The goal is to find a binary prefix code $$C: A \to \{\texttt{0},\texttt{1}\}^*$$ [^automata-theory-notation], 
where $$c(a)$$ is the code word for $$a \in A$$,
that minimizes the following objective function,

$$\mathcal{L}_{(A,w)}(C) = \sum_{i=1}^n w(a_i) \times \text{length}(C(a_i)).$$

For the time being, let's forget about weights, an objective function, and even greedoids.
The focus will only be on the structure of the problem.

To be a binary prefix code, no whole code word in $$C(A)$$ is a prefix of any other.
A
Equivalently, for all $$c_i \in C(A)$$, there does not exists no $$c_j \in C(A)$$ such that $$c_i$$ starts with $$c_j$$. <!-- REF NEEDED -->
For a more in depth look at prefix codes, see here. <!-- REF NEEDED -->
A great way to represent binary prefix codes is through a binary tree.

<!-- Image of tree as prefix code. -->
 
...

From this, we can see that the leaf nodes and the root node stay the same, while the internal nodes are what is changing.

[^automata-theory-notation]: The set $$\{\texttt{0},\texttt{1}\}^*$$ refers to the set of all binary strings e.g. "$$\texttt{01101}$$". This notation is borrowed from automata theory.

---

### References

### Footnotes