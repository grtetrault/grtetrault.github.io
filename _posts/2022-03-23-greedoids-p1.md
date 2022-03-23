---
layout: post
title:  "Greedoids, Part 1: What is a greedoid?"
date:   2022-03-23
author: Garrett Tetrault
categories: greedoid greedy algorithm
excerpt_separator: <!--more-->
katex: true
comments: true
---

Greedy algorithms build up a solution by making the locally optimal, or "greedy", choice at each step.
For certain problems, this family of algorithms is highly effective, producing optimal solutions.
Other times, using a greedy algorithm can return woefully incorrect results.
So, when can a greedy algorithm be used to find the optimal solution?

Greedoids, a topic in combinatorics, shed light on this question.
In fact, it's right there in the name!
*Greed-* as in "greedy algorithm" and *-oid* as in "resembling" or "like".
Formulated 1981 in by Bernhard Korte and László Lovász, greedoids provide a useful framework for thinking about optimization.
They make explicit common properties that allow problems to be optimized by the simple "greedy choice" procedure.

In this post, the concept of a greedoid will be introduced and the intuition behind it will be developed.
To that end, we will work backwards from a greedy algorithm (GA) and build up to an accessible definition of a greedoid.

## Kruskal's algorithm
Kruskal's algorithm, which solves the minimum spanning tree (MST) problem, makes for a good case study.
In this problem, each edge in a connected graph is given a weight and we are tasked with finding the set of edges that form an MST.
A good introduction to MSTs from can be found [here](https://courses.grainger.illinois.edu/cs225/fa2021/resources/mst/).

At a high level, Kruskal's algorithm starts with an empty list and repeatedly appends the cheapest edge that doesn't create a cycle.
This is translated into an (incomplete and rather slow) python implementation below:

```python
def kruskal(graph, weight_fn):
    solution = []
    for edge in sorted(graph.edges, key=weight_fn):
        candidate = solution + edge
        if not contains_cycle(candidate):
            solution = candidate
    return solution
```

What can be drawn from this?
One observation is that the edges of `graph` make up the search space of our problem; this establishes a domain for our optimization.

A second is that, despite its name, the variable `solution` only makes up an MST (a proper solution for this problem) when it is being returned.
At every other point along the way, it is only a "partial solution".
Further remarks about these partial solutions can be made:

* The empty list is a partial solution.
* Every partial solution is a [forest](https://en.wikipedia.org/wiki/Tree_(graph_theory)#Forest), guaranteed by the `not contains_cycle` condition.

So, the solution is an MST while the partial solutions are the collection of all sub-forests of `graph`.
In a more general sense, a solution is what the algorithm is optimizing for, while the partial solutions are the snapshots at each iteration.

## The building blocks
These ideas of search space, solutions, and partial solutions are the building blocks of a greedoid.

In greedoid theory, the search space is referred to as the *ground set*,
solutions are called *bases* (as in basis),
and the collection of both the solutions and partial solutions are known as the *feasible sets*.

Of note is that, for the above observations, there was no particular solution in mind.
This means that the notion of partial solutions extends to all possible solutions for the problem.
Indeed, the feasible sets of a greedoid capture all such solutions and partial solutions.

For Kruskal's algorithm and the MST problem, these sets would have the following form:

* The ground set consists of the edges of the given graph.
* The bases are the collection of all spanning trees.
* The feasible sets are the collection of all (not necessarily spanning) forests.

Notice how the feasible sets naturally contain all bases.

<figure>
<img src="/assets/images/greedoids/mst-feasibility.png" style="width:75%"/>
<figcaption>
    <div class="centered-figcaption">
        Examples of feasible sets (green) and non-feasible sets (red) in the MST problem. 
        Feasible sets must be contain no cycles.
        Note the rightmost is a basis.
    </div>
</figcaption>
</figure>

Having a ground set and feasible sets alone does not make a problem a greedoid. 
Indeed, most problems can be be put into the framework of ground set (domain) and feasible sets (constraints).
But, these building blocks provide a language to describe the properties needed to ensure the optimality of a GA.

## The defining properties
Uncovering these requisite properties is no small feat.
One has to:

* Identify properties that capture the features a GA exploits.
* Determine if these properties generalize well.
* Ensure that these properties can be formalized (in a manageable way), so proofs and assurances can be made.

This is where the insights Korte and Lovász come in.
They noticed two properties, *accessibility* and *exchangeability*, that effectively satisfy these conditions.

### Accessible set systems
Consider the following procedure in the context of the MST problem:
<!-- More here -->

1. Hand pick an MST within `graph`.
2. Record the edges of this MST into the set `hand_picked`.
3. Set `hp_weight_fn` to be zero if an edge is in `hand_picked` and one if not.
4. Run Kruskal's algorithm with inputs `graph` and `hp_weight_fn`.

Under these circumstances, Kruskal's algorithm would return back exactly the set `hand_picked`.
Because `hand_picked` is arbitrary, any MST could be chosen to construct this set.
As such, *every basis* must be reachable from our starting point, the empty set.

The above procedure can be repeated with "forest" in place of "MST" in step one.
In this case, if the hand-picked forest has $$n$$ edges,
then the first $$n$$ elements of the output of Kruskal's algorithm would be `hand_picked`.
This implies that the hand-picked forest was visited during some iteration.

This widens the previous statement to the following: *every feasible set* must be reachable from the empty set.
This is the essence of accessibility.

<figure>
<img src="/assets/images/greedoids/mst-accessibility.png" style="width:80%"/>
<figcaption>
    <div class="centered-figcaption">
        Examples of a feasible set chain in the MST problem.
        Each feasible set contains a feasible set one element smaller, 
        until the empty set is reached.
    </div>
</figcaption>
</figure>

To illustrate how this is formalized, consider a feasible set `target`.

For a GA to arrive at `target`, it must add an edge to a feasible set one element shorter.
The same applies to this smaller set.
Repeating this, a chain of smaller and smaller feasible sets is created.
Eventually, the empty set will be met.

This chain can be traversed in reverse order such that, from the empty set, we arrive at `target`.
This is what is meant by "reaching" a feasible set.

Formalizing the existence of this chain yields the following definition:
a collection of feasible sets is an *accessible set system* if every feasible set contains a feasible set one element smaller.

### The exchange property
This brings us to the *exchange property*.
Here the concern is another requirement of a GA: a basis must always be returned.
That is, the greedy choice must never lead to a dead-end.

A dead-end occurs when no elements in the search space can be added to a feasible set.
So, the opposite of a dead-end is the ability to append to feasible sets.
Because bases are maximal feasible sets, growing feasible sets always ends up at a basis. 

This reasoning allows us to reduce the requirement "a basis must always be returned" to "we must always be able to append to a feasible set".

This reduced statement is still too hazy to be formal.
To resolve this, Korte and Lovász shifted perspective:
instead of appending to a single feasible set, 
consider the interplay of a feasible set with all others.

<figure>
<img src="/assets/images/greedoids/mst-exchange.png" style="width:50%"/>
<figcaption>
    <div class="centered-figcaption">
        A feasible set (right) in the MST problem
        receiving an edge (light green) from a larger feasible set (left) that preserves feasibility. 
    </div>
</figcaption>
</figure>

To that end, the MST problem will again be probed.
Let `smaller` and `larger` be forests with the latter containing more edges.
The goal is to find an edge in `larger` that can be appended to `smaller`.

Comparing the nodes in each set, there are two cases:
* The set of nodes in `larger` has a node not in `smaller`.
* All nodes of `larger` are also in `smaller`.

In the former case, any edge containing a node not in `smaller` can be chosen.
This would create a branch off of `smaller` into new territory (the new node).

In the latter case, consider the [components](https://en.wikipedia.org/wiki/Component_(graph_theory)) of each forest.
Because `larger` has more edges, it must has fewer components.
If not, then one component would contain a cycle.

By the [pigeonhole principle](https://www.cantorsparadise.com/the-pigeonhole-principle-e4c637940619),
there must be two or more nodes that are in different components of `smaller`,
but are in the same component of `larger`.
Along the path between these nodes, there is an edge that spans components of `smaller`.
Because it spans components, this edge can be added without creating a cycle. 

This shows that for any two forests, one smaller and one larger, there is an edge of the larger that can be appended to the smaller without creating a cycle.

Generalizing to greedoids gives the following:
for any two feasible sets, one smaller and one larger, there is an element of the larger that can be appended to the smaller that preserves feasibility.
This is exactly the definition of the exchange property.

Reversing the reduction above, this property gives the guarantee that a basis is always returned.
And, though the example here relies on graph specific features, the properties is remarkably ubiquitous.

## Conclusion
Summarizing the results, the definition of a greedoid can now be made.
A greedoid over a ground set is a collection of feasible sets that satisfy the following:

1. Every (non-empty) feasible set contains a feasible set one element smaller.
2. For any two feasible sets, one smaller and one larger, there is an element of the larger that can be appended to the smaller that preserves feasibility.

In the context of a GA, what makes a greedoid interesting is that these properties ensure (respectively) that:

1. Every solution can be reached.
2. A solution is always returned.

And these make a greedy algorithm work.
Moreover, the properties of a greedoid are incredibly general.
A considerable number of problems solved by GAs, from the MST, to Dijkstra's algorithm, to even Gaussian elimination, can be put into the context of greedoids.

However, there still are problems solved by greedy algorithms that do not have a greedoid representation.
One conspicuous omission is Huffman coding.

In the next post, the capabilities and limitations of greedoids will be explored further.
To that end, the GA for greedoids will be given and used to develop an algorithm to solve the scheduling problem.
The aim is to give a picture of when one can use greedoids and why one would want to.

Thanks for reading!

## References

* [Björner, Anders; Ziegler, Günter M. (1992), "Introduction to greedoids"](https://www.mi.fu-berlin.de/math/groups/discgeom/ziegler/Preprintfiles/006PREPRINT.pdf?1397057423)
* [Boyd, Andrew E.; Faigle, Ulrich (1990), "An algorithmic characterization of antimatroids"](https://www.sciencedirect.com/science/article/pii/0166218X9090002T)
* [Cook, William R.; Nedunuri, Srinivas; Smith Douglas R. Cook, William R. (2010), "A Class of Greedy Algorithms and Its Relation to Greedoids"](https://link.springer.com/chapter/10.1007/978-3-642-14808-8_24)
* [Darío G, "Proof of Graphic Matroids (Using Rank in Context of Graph Theory)", StackExchange](https://math.stackexchange.com/questions/2780714/proof-of-graphic-matroids-using-rank-in-context-of-graph-theory)