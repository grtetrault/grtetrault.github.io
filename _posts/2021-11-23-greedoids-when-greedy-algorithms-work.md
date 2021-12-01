---
layout: post
title:  "[Draft] Greedoids: When Greedy Algorithms Work"
date:   2021-11-23 00:00:00 -0400
author: Garrett Tetrault
categories: greedoid greedy algorithm
excerpt_separator: <!--more-->
katex: true
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

[^solution-basis]: In greedoid theory, "solutions" are referred to as "bases", however this language is not being used here to avoid a a digression into where this terminology comes from. For interested readers, this is borrowed from matroid theory. <!-- REF NEEDED -->
[^feasible-prim-kruskal]: For the MST problem, $$F$$ can be either the collection of all trees or all forests. The former corresponds to Prim's algorithm, while the latter to Kruskal's.

<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/spanning-tree-feasibility.png"/>
<figcaption>
    <div class="centered-figcaption">
        Examples of feasible sets (green) and non-feasible sets (red) in the MST problem. 
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
Below, we have a partial `python` implementation of this algorithm to serve as pseudo-code:
```python
# Here, Greedoid is some previously defined class.
def greedy_algorithm(greedoid: Greedoid, cost: Callable):
    result = []
    while True:
        is_feasible = lambda X: X in greedoid.feasible_sets
        candidates = [x for x in greedoid.search_space if is_feasible(result + [x])]

        if len(candidates) > 0:
            min_candidate = min(candidates, key=lambda x: cost(result + [x]))
            result.append(min_candidate)
        else:
            break
    
    return result
```
You may think this algorithm looks pretty inefficient, and you'd be right!
Most greedy algorithms you've seen are optimizations on this simple approach, 
either by reducing the search space, using more appropriate data structures, or by another technique.
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
Recalling our algorithm, we initiate `result = []` -- the empty set is our starting point!
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
Solutions will always be the bigger set until we reach another solution.
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

In summary, these properties tell us that greedy algorithms work when you can, 
one, reach every solution from the empty set and,
two, always grow a partial solution into a complete solution.
Written like this, one can get the gist of why greedoids are optimally solved by greedy algorithms.
The properties that define this structure seem to be the minimum to guarantee we can always reach the minimal solution.

## An Example: Scheduling

Until now, I've only mentioned how greedoids look for the MST problem. 
Spanning trees lend themselves well to visualizations and many problems resemble their structure.
To show the scope of greedoids, I'd like to walk through a problem that feels very different: scheduling [^type-of-scheduling].

[^type-of-scheduling]: More specifically, the scheduling problem here is single-machine scheduling with precedents. In the three field notation for job scheduling, the problem is $$\textbf{1} \vert \textbf{prec} \vert h_{\max}$$.

The problem is stated as follows.
We are given a set of $$n$$ jobs $$J = \{j_1, j_2, \ldots, j_n\}$$ where each job $$j_i \in J$$ has,
* A runtime $$p_i$$.
* A (possibly empty) set of precedent jobs $$J_i \subseteq J$$.
    Note that we can't have any circular dependencies [^catch-22].
* And a cost function $$h_j$$ which is a function of completion time (scheduled time + runtime).

[^catch-22]: For example "to turn the lights on, you need to find the switch", and "to find the switch, the lights need to be on" would be a circular dependency. The order is actually constrained to the more precise definition of a partially ordered set. <!-- REF NEEDED -->

The goal is to find a schedule of jobs that respects the precedent constraints and minimizes the maximum cost over all jobs.
It is important to note we are concerned with *minimizing the maximum cost*.
A typically variant of this problem minimizes maximum lateness.
This is done by giving each $$j_i \in J$$ a deadline $$d_i$$ and setting $$h_i(t) = t - d_i$$ where $$t$$ is the completion time.
For objectives that minimize aggregate statistics, like total number of late jobs or profit, greedy algorithms do not necessarily produce the optimal solution.

<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/scheduling-precedence-structure.png" style="width:12%"/>
<figcaption>
    <div class="centered-figcaption">
        An example precedence structure for jobs \(J = \{j_1,j_2,j_3,j_4\}\). 
        Arrows indicate an "is dependent on" relationship.
        For example, if \(j_2 \to j_1\), this means that \(j_1\) is a precedent of \(j_2\). 
    </div>
</figcaption>
</figure>

### Fitting the Greedoid
To define a greedoid we need to specify the search space and feasible sets.
A great place to start is identifying the solutions; what are we looking for?
Greedoids impose a lot of restrictions on solutions that reduce the amount of viable representations. 
This allows you to iterate faster to find ones that work.
For example, the fact that all solutions must be unordered sets of the same size narrows how you can define them.

Here, we are looking for an optimal schedule or, equivalently, a sequence of jobs.
As all jobs must be scheduled, solutions will be represented by the set $$J$$.
This may seem like an odd choice because $$J$$ is an unordered set; all solutions would have the same representation.
The reason we can use this is that greedy algorithms are order aware.
Indeed, if you look back to the pseudo-code, `result` is an ordered set and `cost` acts on augmentations of this ordered set.
So, while feasible sets and solutions are unordered, **the weight function and output of the algorithm can be order dependent**.

The search space comes from breaking down the solutions into their building blocks.
A good rule of thumb I've found is to keep the search space as generalizable as possible.
The structure should be imposed by the feasible sets, not the the search space.
If we look at the solutions, all members are made up of the same, complete set of jobs.
A natural choice for the search space, then, would also be $$J$$.
That is, $$S = J$$.

Turning to the feasible sets, this is where we need to enforce the precedence structure.
Further, we need to make sure that both solutions and empty sets are feasible.
These two provide an easy check to see if you're on the right track for problems in the wild.
Because feasible sets are unordered, enforcing precedence amounts to making sure that a job never appears in a set without its precedents.
The empty set would satisfy this vacuously, while the solutions contain all jobs and hence all precedents.
The formal construction we'll use here is,

$$
F = \{X \mid X \in 2^S \;\textrm{where}\; J_i \subseteq X \;\forall\; j_i \in X\}
$$

As the precedent constraints (and the subsequent feasible sets) impose an ordering on jobs,
we can actually visualize what these feasible sets look like.

<!-- Diagram of feasible sets for some job -->
<figure>
<img src="/assets/images/greedoids-when-greedy-algorithms-work/scheduling-feasible-sets.png" style="width:25%"/>
<figcaption>
    <div class="centered-figcaption">
        The feasible sets from the precedence structure above.
        The lines show superset and subset relationships between feasible sets.
    </div>
</figcaption>
</figure>

The proof that $$S$$ and $$F$$ make up a greedoid is fairly straightforward, so I'll leave it up to the interested reader.
For a hint, it relies heavily on the fact that there are no circular dependencies.


## Conclusion

Before I wrap up this post, there are two things I have to come clean about.

First is that **greedoids can't represent all problems solved by greedy algorithms**.
There exist problems that either can't or haven't been characterized as greedoids.
For example, a greedoid representation for Huffman codes, one of the canonical examples I mentioned, hasn't been found.
Before you think this wasn't fruitful, though, greedoids *do* characterize a huge swath of problems.
The minimum path problem solved by Dijkstra's algorithm, solving systems of linear equations via Gaussian elimination, and much more have corresponding greedoids.
I find this more of a comment on myriad complexities that the seemingly simple greedy algorithm exploits, rather than the lack of utility in greedoids.

Second, these optimization problems are very dependent on their cost function.
There are some pretty different forms that appear in the MST and scheduling problems (linear and min-max respectively), and many more forms are possible.
What hasn't been discussed here is that there are restrictions on what these can be.
And... there isn't a quick answer to what these restrictions are.
I've tried in this post to focus on the *structure* of the problems at the cost of discussing cost functions (a topic deserving of its own post).
This is a topic that has been explored in depth which the references give good introductions to.

Greedy algorithms provide an elegant solution to a problem when they work. 
They are often simple, expressive, and easy to implement.
The class of problems solved optimally by greedy algorithms is both rich and diverse.
The task of characterizing these problems and identifying threads of similarity between them is no easy feat.
Though not complete, greedoids give valuable insight into what these problems look like, with a simplicity that makes them feasible for use in the real world.

---

### References

* [Björner, Anders; Ziegler, Günter M. (1992), "Introduction to greedoids"](https://www.mi.fu-berlin.de/math/groups/discgeom/ziegler/Preprintfiles/006PREPRINT.pdf?1397057423)
* [Boyd, E. Andrew; Faigle, Ulrich (1990), "An algorithmic characterization of antimatroids"](https://www.sciencedirect.com/science/article/pii/0166218X9090002T)

### Footnotes