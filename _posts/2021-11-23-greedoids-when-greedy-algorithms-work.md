---
layout: post
title:  "[Draft] Greedoids: When Greedy Algorithms Work"
date:   2021-11-23 00:00:00 -0400
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
For example, in finding a minimum spanning tree (MST), $$S$$ is the set of edges of the graph.

In our search space, there are some subsets that can contribute to a solution and some that cannot.
These are called feasible sets and the collection of all feasible sets is denoted by $$F$$.
In this context, solutions are maximal feasible sets and are typically what you'd like to find [^solution-basis].
Returning to MSTs, $$F$$ would consist of all (not necessarily spanning) trees and
the set of solutions would be the collection of all spanning trees [^feasible-prim-kruskal].

[^solution-basis]: In greedoid theory, "solutions" are referred to as "bases", however this language is not being used here to avoid a a digression into where this terminology comes from. For interested readers, this is borrowed from matroid theory, which you can learn more about here. <!-- REF NEEDED -->
[^feasible-prim-kruskal]: For the MST problem, $$F$$ can be either the collection of all trees or all forests. The former corresponds to Prim's algorithm, while the latter to Kruskal's.

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
# Here, Greedoid is some previously defined class.
def greedy_algorithm(greedoid: Greedoid, weight_fn: Callable):
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
<!-- TODO FLOW -->

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
<img src="/assets/images/greedoids-when-greedy-algorithms-work/spanning-tree-exchange.png" style="width:55%"/>
<figcaption>
    <div class="centered-figcaption">
        A non-maximal feasible set \(Y\) (right) in the MST problem receiving an edge \(x\) (light green) from a solution \(X\) (left) that maintains feasibility. 
    </div>
</figcaption>
</figure>

<!-- TODO FLOW -->

## An Example: Scheduling

Until now, I've only mentioned how greedoids look for the MST problem. 
Spanning trees lend themselves well to visualizations and many problems resemble their structure.
To show the scope of greedoids, I'd like to walk through a problem that feels very different: scheduling [^type-of-scheduling].

[^type-of-scheduling]: More specifically, the scheduling problem here is single-machine scheduling with precedents. In the three field notation for job scheduling, the problem is $$\textbf{1} \vert \textbf{prec} \vert h_{\max}$$.

The problem is stated as follows.
We are given a set of jobs $$J = \{j_1, j_2, \ldots, j_n\}$$,
where each job $$j_i \in J$$ has,
* A runtime $$p_i$$,
* A (possibly empty) set of precedent jobs $$J_i \subseteq J$$,
* And a cost function $$h_j$$ which is a function of completion time (scheduled time + runtime).

The goal is to find a schedule of jobs that respects the precedent constraints and minimizes the maximum cost over all jobs.
There are some constraints on what these precedent jobs can be.
The one I'll mention is that we can't have any Catch-22s; there can't be any circular dependencies.
For example "to turn the lights on, you need to find the switch", and "to find the switch, the lights need to be on" would be a circular dependency.

A typically variant of this problem minimizes maximum lateness, where $$h_i(t) = t - d_i$$ where $$t$$ is the completion time and $$d_i$$ is the deadline for job $$j_i$$.

An example...

$$S = \{X \subseteq J \mid J_i \subseteq X \; \forall \; j_i \in X\}$$

<!-- For example, we could be looking at a kitchen, where some jobs are prep work and others are customer orders -->

<!-- Until now, I've only mentioned how greedoids look for the MST problem. 
ƒSpanning trees lend themselves well to visualizations and many problems resemble their structure.
To show the scope of greedoids, I'd like to walk through a problem that feels very different: huffman codes.

The problem is stated as follows.
We are given a set of symbols $$A = \{a_1, a_2, \ldots, a_n\}$$ with and weight function $$w: A \to \mathbf{R}^+$$ with positive real outputs.
The goal is to find a binary prefix code $$C = \{c_1, c_2, \ldots, c_n\}$$, 
where $$c_i$$ is the code word for $$a_i$$ and $$i \in \{1, 2, \ldots, n\}$$,
that minimizes the following cost function,

$$\mathcal{L}_{(A,w)}(C) = \sum_{i=1}^n w(a_i) \times \mbox{length}(c_i).$$

To be a binary prefix code, no code word in $$C$$ can be the prefix of any other.
For example, if `010` $$\in C$$, then neither `0` nor `01` can be in $$C$$.
A great way to represent a binary prefix code is through a full binary tree [^code-as-tree].
In this representation, the leaves are the set of symbols in $$A$$. 
Every symbol has a path of rights and lefts (or zeros and ones) from the root that determines its code word.
To get a different prefix code, these paths would need to be changed.

[^code-as-tree]: For a more in depth look at prefix codes and why you can represent them with binary trees, see here. <!-- REF NEEDED -->

<!-- Quick diagram showing prefix code to binary tree. -->
<!-- Maybe show a couple different ones (same set up as catalan number wiki) -->

<!-- But, these paths aren't actually that unique.
In the cost function, the only attribute of a code word we are concerned with is its length.
As length corresponds directly with depth in the tree representation,
we can swap the left and right children of any node and the value of the cost doesn't change. 
This equates to flipping a bit for all descendants.

### Search Space

To define a greedoid we need to specify the search space and feasible sets.
A great place to start is identifying the solutions; what are we looking for?
Here, we are looking for an optimal prefix code or, equivalently, an optimal binary tree with $$A$$ as the set of leaves.
These trees will be our solutions.

The search space comes from breaking down the solutions into their building blocks.
For problems in the wild, there is definitely some exploration required here.
Binary trees, for example, have a ton of representations and choosing one requires some trial and error.
A good rule of thumb is to keep the members of the search space as basic and generalizable as possible.
This comes from thinking forward to the when we'll have to satisfy the exchange property.
The structure should be imposed by the feasible sets, not the the search space [^another-tree-representation].

[^another-tree-representation]: Another representation would be to have each node a tuple of its left a right children $$(l, r)$$. Each node, then, would admit a tree e.g. $$(((a, b), (c, d)), e)$$. This additional structure makes it difficult (if even possible) to satisfy the exchange property.

Here, we will define a node by its descendant leaves; that is, a subset of $$A$$.
In this representation, the root node of a solution would be $$A$$ itself.
Using this, the search space is defined by $$S = 2^A$$, the powerset of $$A$$.
Of note is that the search space includes both the leaf nodes (whose descendent leaves are themselves), 
and the empty set (with no descendants).
This is for convenience in formalization, but for a sketch of the problem, they are not necessary.

### Feasible Sets

With the search space outlined, we turn to the feasible sets.
This is where we want to enforce the tree structure.
As above, there are a plethora of ways the feasible sets can be constructed.
The one used here was the result of a lot of fiddling with the search space and seeing what feels natural and satisfies the greedoid properties.
I know "natural" can be ambiguous, so I've provided some reasoning to (hopefully) communicate what I mean.

To be a full binary trees, there must be some notion of children.
A node must have exactly one left child and one right child.
For this particular problem though, which child is left and which is right doesn't matter much.
In the cost function, the only attribute of a code word we are concerned with is its length.
As length corresponds directly with depth in the tree,
swapping the left and right children of a node doesn't change the cost.
This helps because order no longer matters.

Further, a symbol cannot be a descendent of a node without it being a descendent of its children.

the 

descendent leaves of the children must account for all descendants of the parent. 

Putting these together, a set $$X \subseteq S$$ is feasible if, for all $$n \in X$$, there exists unique $$l, r \in X$$ such that $$l \cup r = n$$. -->

---

### References

### Footnotes