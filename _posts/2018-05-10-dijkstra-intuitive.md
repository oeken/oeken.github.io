---
layout: post
title:  "Intuition Behind Dijkstra's SPA"
date:   2018-05-09
description: Examining a recursive formulation to solve single-source shortest path problems.
comments: true
permalink: dijkstra-intuition
fig-size: 480
---

**Prerequisite:** Familiarity with graph data structure and recursion/dynamic programming.

# Motivation

[**Dijkstra's Shortest Path Algorithm**](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm){:target="_blank"} is one of the fundamental algorithms on graph data structures. It has many application areas thus used very frequently. There are numerous resources (textbooks, lectures etc.) from which one can learn how it works. The problem with those resources is that the following classic recipe is used to explain this algorithm:

1. List the instructions in pseudocode
2. Elaborate on the idea of each instruction

In general, this is a nice strategy to teach algorithms but in the case of Dijkstra's algorithm, it does not work well. When I was an undergrad student I learned it this way as well, but the algorithm faded in my mind in a relatively short time when I did not go over it. I think the main reason is it was not possible to state the idea of Dijkstra's algorithm in simple terms.

It turns out that there is a more intuitive and simpler explanation for this algorithm. It is unraveled when the solution to the problem is formulated recursively. The Dijkstra's algorithm essentially:
- Solves the recursive formulation through tabulation
- Includes some implementation tricks to reduce the runtime

For the better understanding of what exactly Dijkstra's SPA does, in this post, we will elaborate on the recursive formulation to solve this problem.

# Idea

**Problem:** Given a source node \\(S\\), for each node \\(N_i\\) in the graph we would like to find a path (a sequence of nodes) from \\(S\\) to \\(N_i\\) whose cost is minimal. The cost of a path is the sum of the weights of the edges observed in the path. This problem is known as *single-source shortest path problem* (SSSP). Let us only consider the graphs with non-negative edge weights. 

To show the recursive relation, as always, we assume that we already have the solution to a smaller version of the problem. In this case:
- Let \\(K\\) denote the set of nodes in which we know the shortest paths to and the costs associated with them. 
	- Includes \\(S\\) too.
	- None of the paths follow a node that is not in \\(K\\).
- Let \\(T\\) be the set of nodes that have an immediate connection to \\(K\\). 
	- In other words, the nodes that have an edge to at least one of the nodes in \\(K\\). 
- Furthermore, let \\(L\\) be the set of nodes that are not in \\(K\\) or \\(T\\).

Let's demonstrate these 3 sets with a figure. Consider only the nodes in the graph in Figure 1 where:
- **<span style="color:#9673A6">Purple</span> \\(S\\) node:** the source node
- **Nodes in <span style="color:#FF6666">red area</span>:** the set \\(K\\)
- **<span style="color:#D79B00">Orange</span> nodes:** the set \\(T\\)
- **<span style="color:#6C8EBF">Pale blue</span> nodes:** the set \\(L\\)

<center><img width="{{ page.fig-size }}px" src="/assets/dijsktra-intuitive/dijkstra.svg" style="margin-top:20px;margin-bottom:30px;"></center>
<center><b>Figure 1:</b> <i>An example graph with node sets K, T, L and the candidate paths for node C</i></center> 
<br>

To concretize better we have put 3 nodes in \\(T\\), namely \\(A\\), \\(B\\), and \\(C\\). Let's say you consider all possible paths going from \\(S\\) to \\(C\\). There are 3 different *groups of paths*:
- **1st group:** the paths that go to \\(C\\) without visiting \\(A\\) or B
- **2nd group:** the paths that go to \\(C\\) by visiting \\(A\\)
- **3rd group:** the paths that go to \\(C\\) by visiting \\(B\\)

Although 2nd and 3rd group are not disjoint, this is not a problem for us. Keep in mind that union of these 3 groups cover all paths going from \\(S\\) to \\(C\\). 

>We will use the following notation in the remaining parts of this post:
>- \\(w(U,V)\\) denoting the weight of the edge between the nodes \\(U\\) and \\(V\\).
>- \\(sp(U,V)\\) denoting the cost of the shortest path from the node \\(U\\) to the node \\(V\\).

Furthermore, let \\(M_C\\) be the node in \\(K\\) that \\(C\\) is connected to and the following sum  is minimal: \\(sp(S,M_C) + w(M_C,C)\\)  
Let us call this sum **d**irect **s**hortest **p**ath to \\(C\\), denoted with \\(dsp(S,C)\\).

Note that \\(dsp(S,.)\\) is not only defined for \\(C\\) but also for the other nodes in \\(T\\). 

For the node \\(C\\), We see that **the shortest path in each group** (let's call it candidate path) would have the following costs:

- 1st candidate cost: <span style="color:#009900">\\(dsp(S,C)\\)</span>
- 2nd candidate cost: <span style="color:#004C99">\\(dsp(S,A)\\)</span> \\(+\\) <span style="color:#007FFF">\\(sp(A,C)\\)</span>
- 3rd candidate cost: <span style="color:#CC0000">\\(dsp(S,B)\\)</span> \\(+\\) <span style="color:#FF0000;">\\(sp(B,C)\\)</span>

Here the \\(sp\\) terms are unknown to us, we only know that they are non-negative. Other terms are available. Each term is color-coded and their correspondent paths are shown in Figure 1 with arrows.

Just like the node \\(C\\), we can define the same groups for \\(A\\) and \\(B\\) as well. And the costs of the candidate paths in those groups would be as following.

For \\(A\\):
- 1st candidate cost: \\(dsp(S,A)\\)
- 2nd candidate cost: \\(dsp(S,B) + sp(B,A)\\)
- 3rd candidate cost: \\(dsp(S,C) + sp(C,A)\\)

For \\(B\\):
- 1st candidate cost: \\(dsp(S,B)\\)
- 2nd candidate cost: \\(dsp(S,A) + sp(A,B)\\)
- 3rd candidate cost: \\(dsp(S,C) + sp(C,B)\\)

#### Key Observation

So, for each of \\(A\\), \\(B\\), \\(C\\) there are 3 candidate shortest paths. Without loss of generality, lets say \\(dsp(S,C) \leq dsp(S,A) \leq dsp(S,B)\\). This comparison enables us to give a guarantee.

As soon as we know this inequality **we can guarantee that the 1st candidate path of \\(C\\) has smaller (or equal) cost than other two candidate paths of \\(C\\)!** This works only given the fact that no term (e.g. \\(w\\), \\(dsp\\), \\(sp\\)) is negative. Which essentially means no edges with negative weights. For other two nodes \\(A\\) and \\(B\\) we cannot give guarantees. That is not a problem though, we have just discovered a new shortest path from \\(S\\) which goes to \\(C\\). We now include \\(C\\) in the set \\(K\\), repeat the same procedure with the enlarged set \\(K\\). As for the base case, we would have only one node in \\(K\\) that is the source node \\(S\\).

In plain English, the guarantee we found says that:

> Among the paths that go from \\(S\\) to the nodes in the immediate neighbourhood of set \\(K\\) using only the nodes in \\(K\\), the one with the smallest cost is the shortest path from \\(S\\) to the node where the path ends.

Notice that we did not mention if the edges are directed or undirected. That is because it does not matter, undirected edges can be represented with directed edges. The recurrence relation takes only 2 things into consideration: given a node what other nodes are reachable are and at what cost is. Directed weighted edges contain this information.


# Procedure and Optimization

This recursive relation is easiest implemented by tabulation. Starting from the base case we grow the set \\(K\\) one node at a time. The following is the inefficient pseudocode describing the procedure. 

```
Input:  N, number of nodes
	E, set of edges with weights
	s, source node
Output: cost, shortest path cost to each node

procedure dijkstraSSSP
  // base case
  cost[s] = 0
  K = {s}

  // tabulate N-1 times
  for i = 2 ... N
    T = getImmediateNodes(K)    
    shortestNode = -1
    shortestCost = Inf
    for each node in T
      for each (source,node,weight) in E
        if cost[source] + weight < shortestCost
          shortestCost = cost[source] + weight
          shortestNode = node
        end
      end
    end

    cost[shortestNode] = shortestCost
    K.add(shortestNode)
  end
end
```

**Optimization Idea:** Essentially we have a set of costs (of paths that go to nodes in \\(T\\) from \\(S\\)) in which sometimes we add and remove elements (caused by moving node from set \\(T\\) to \\(K\\)). We are interested in the minimum element in this set. This is the perfect scenario for [heaps](https://en.wikipedia.org/wiki/Heap_(data_structure)).

# Remarks

Once one notices the above-mentioned guarantee, the idea is very simple to implement. What makes the regular explanation of Dijkstra's algorithm hard is:
- No explanation for the guarantee/observation
- Optimized code with implementation details

