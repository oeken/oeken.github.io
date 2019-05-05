---
layout: post
title:  "Not Darkest Dungeon"
date:   2019-04-27
description: Finding the way out for your flawed heroes.
comments: true
permalink: not-darkest-dungeon
fig-size: 600
---

<!-- intro -->
Recently I have been playing this game called _Darkest Dungeon._ It is a rogue-like turn-based RPG game. The player gathers a gang of fighters with various characteristics and skills and goes onto exploration of dungeons with various properties. Every dungeon campaign has a goal, and the player suffers a big penalty when returned back from the dungeon without achieving the goal. I really enjoy playing this game as in it's essence, it pushes the player to develop efficient combat, exploration and team formation strategies under numerous constraints introduced by game dynamics.

<!-- motivation -->
### Motivation
One of the common goals in dungeons is, namely, __Explore 90% of the rooms__. Dungeon maps are designed as a network of __rooms__. Every room is connected to one or more other rooms with __corridors__. The connections, and therefore the structure of the dungeon map, could be quite arbitrary. See an example dungeon map below. Visiting rooms or corridors is always a costly decision in the game by design. Fighters suffer stress penalty, traps deal HP damage, and resources such as light deplete with each step team takes. Such events in rooms and corridors are marked on the dungeon maps.
For this reason, in this kind of dungeon campaigns, it is almost always a good idea to explore only 90% of the rooms and not more than that. In addition, for the same reason player should rather avoid redundant room visits as they do not contribute to the progress towards the goal.

<center>  
    <div style="margin-top:40px;margin-bottom:40px;">
        <figure style="display:inline-block;">
          <img width="{{ page.fig-size }}px" src="/assets/not-darkest-dungeon/dd-map-1.jpg">
        </figure>

        <p><i>An example map with various event annotations. The large boxes are <b>rooms</b> whereas the sequences of small boxes represent the <b>corridors</b>.</i></p>
    </div>
</center>

<!-- formulation -->
### Problem
This kind of dungeon goal and map posed as an attributed graph, motivated me to come up with an algorithm that calculates the optimal path for the team. To facilitate algorithm’s presentation let’s now make some assumptions, simplifications, and definitions.

Denote the set of all rooms with \\(V\\) and set of all corridors with \\(E\\). Furthermore, denote the __cost__ of visiting a room \\(v \in V \\) with \\(C(v)\\). Similarly, let the __cost__ of visiting a corridor \\([u,v] \in E \\) be \\(C([u,v])\\). If we start our dungeon campaign in room \\(s\\), then we want to come up with a sequence of rooms \\( P = [s, v_1, v_2, ...] \\) where the total cost, \\( \sum_{i = 2}^{\|P\|} C(p_i) + C([p_{i-1},p_{i}]) \\), is minimal while every room \\(v \in V \\) is present in \\( P \\).

For the sake of simplicity, the algorithm presented addresses the constrained version of this problem, although it is easily generalizable. Specifically, let's assume:
- We have to visit not only 90% of the rooms but __all__. 
- Cost of visiting any __room__ is __1__.
- Cost of visiting any __corridor__ is __0__.

With this set of assumptions, our algorithm will attempt to find a path that contains all rooms but with as few repetitions as possible.

<!-- algorithm -->
### Algorithm
Following a recursive formulation, one can notice that two factors are important to be considered: 
- Current room, \\(r\\)
- Map exploration state, \\(S\\)

Here \\(S\\) is a bitset that maps rooms to \\(\\{0,1\\}\\). In other words,  it stores the visited status of the rooms.

Let \\(f(r, S)\\) be the function computing the cost of the optimal path on a given map starting from room \\(r\\) with the given exploration state \\(S\\). We can define it recursively as:

$$
    f(r,S) = 
    \begin{cases} 
        0 & S = 1 \\
        \min_{k \in \mathcal{N}(r)} \{ f(k, S_k) \} & otherwise
    \end{cases}
$$

And notation clarifications...
> - \\(S = 1\\) indicates a set \\(S\\) where all rooms are mapped to 1s. 
> - \\(\mathcal{N}(r)\\) denotes the neighbours of room \\(r\\).
> - \\(S_k\\) denotes the transformed version of set \\(S\\) where room \\(k\\) is mapped to 1.

Note that, in contrast to the graph traversal algorithms such as DFS, we may want to visit rooms that are already visited. See the example map below where we start exploring from room 1. 

<center>
  <div style="margin-top:40px;margin-bottom:40px;">
        <figure style="display:inline-block;" style="margin-top:40px;margin-bottom:40px;">
          <img display="block" width="{{ page.fig-size  | divided_by:1.5 }}px" src="/assets/not-darkest-dungeon/graph1.svg">
          <figcaption style="margin-top:10px"><i>Example map with starting room #1</i></figcaption>
        </figure>

        <figure style="display:inline-block;" style="margin-top:40px;margin-bottom:40px;">
          <img display="block" width="{{ page.fig-size  | divided_by:1.5 }}px" src="/assets/not-darkest-dungeon/graph1-1.svg">    
          <figcaption style="margin-top:10px"><i>Optimal solution #1</i></figcaption>
        </figure>

        <figure style="display:inline-block;" style="margin-top:40px;margin-bottom:40px;">
          <img display="block" width="{{ page.fig-size  | divided_by:1.5 }}px" src="/assets/not-darkest-dungeon/graph1-2.svg">
          <figcaption style="margin-top:10px"><i>Optimal solution #2</i></figcaption>
        </figure>
    </div>
</center>

With the given recursive formulation, we do not prevent revisiting of rooms but introduce another problem: __infinite recursion__. Let's unfold the recursion tree for this toy case.

<center>
  <div style="margin-top:40px;margin-bottom:40px;">
        <figure style="display:inline-block;" style="margin-top:40px;margin-bottom:40px;">
          <img display="block" width="{{ page.fig-size  | divided_by:1 }}px" src="/assets/not-darkest-dungeon/recursion-tree1.svg">    
          <figcaption style="margin-top:10px"><i>Recursion tree for the toy example. Nodes that cause an infinite loop are marked in red.</i></figcaption>
        </figure>
    </div>
</center>

<!-- implementation tricks -->
<!-- complexity -->
Following edges that lead to __red__ nodes would always add redundant costs to the solutions (assuming positive costs) to be explored in the recursion tree. Therefore with an implementation trick, we can detect the loop inducing nodes and never visit those. We have a total of \\(N * 2^N\\) nodes, therefore worst-case time complexity of this approach would be \\(O(N * 2^N)\\), where \\(N\\) represents the total number of rooms in the map. Furthermore, the function we defined calculates the optimal cost but not the path itself. To reconstruct the optimal path, a tailored implementation is needed. See below the critical parts from the full implementation to get an overview of the algorithm.

<!-- code -->
```cpp
struct Graph {
    int n;      // number of nodes
    int e;      // number of edges
    vector<vector<int>> adjList;    // adj list
    vector<vector<int>> adjMat;    // adj matrix
};

struct DDPath {
    vector<int> seq;
    int cost;
};
```

```cpp
DDPath searchDDPath(Graph& G, int node){
    // initialize
    DDPath path;
    uint v = 0;
    v = setBit(v, node);
    vector<unordered_set<uint>> inQuery(G.n);
    vector<unordered_map<uint, uint>> dpCost(G.n);
    vector<unordered_map<uint, int>> dpNext(G.n);

    // run recursive algorithm
    searchDDPathUtil(node, v, G, dpCost, dpNext, inQuery);

    // save and return
    path.cost = dpCost[node][v];
    path.seq = reconstructSequence(node, v, dpNext);
    return path;
}
```

```cpp
void searchDDPathUtil(int i, uint v, Graph& G, vector<unordered_map<uint, uint>>& dpCost, vector<unordered_map<uint, int>>& dpNext, vector<unordered_set<uint>>& inQuery){
    // base case
    if(isFull(v, G.n)){ // search completed
        dpCost[i].insert(make_pair(v,0)); // cost of the path
        dpNext[i].insert(make_pair(v,-1)); // end of the path
        return;
    }
    if(inQuery[i].find(v) != inQuery[i].end()){ // loop detected
        return;
    }
    if(dpCost[i].find(v) != dpCost[i].end()){ // already calculated
        return;
    }

    // recurse
    inQuery[i].insert(v); // mark this node as in query

    int bestTrg = -1;
    int bestTrgCost = INT_MAX;
    for(int trg: G.adjList[i]){
        int trgV = setBit(v, trg);
        searchDDPathUtil(trg, trgV, G, dpCost, dpNext, inQuery);
        if(dpCost[trg].find(trgV) != dpCost[trg].end() && dpCost[trg][trgV] < bestTrgCost){
            bestTrgCost = dpCost[trg][trgV];
            bestTrg = trg;
        }
    }

    int cost = bestTrgCost + G.adjMat[i][bestTrg];
    dpCost[i].insert(make_pair(v, cost));
    dpNext[i].insert(make_pair(v, bestTrg));

    inQuery[i].erase(v); // unmark this node as in query
}
```

<!-- extensions -->
### Extensions
In order to generalize this approach to the problem definition given earlier, one can simply change the recursion termination condition from __all rooms visited__ to __X% of the rooms visited__. To deal with varying room and corridor costs, one can simply add those values at each recursion step followed.

