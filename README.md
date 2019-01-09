# ALIS_RobotSwarm
A study on how robots would navigate withing the ALIS system

This repository contains a theoretical study of a pathfinding problem, which also has to avoid collisions. See problem description below.

The Algorithm will be capable of finding adequately good paths for each bot in the swarm, avoiding collisions. 

This case study will be done in Python.

## Problem Description

We have a collection of robots in a 3D-grid, each with a current position and a destination. We now wish to compute the shortest path to destination, taking into account that paths might currently be occupied by another bot.

## Problem Representation

To be able to use Dijkstra's algorithm for shortest-path evaluation we must translate the 3D grid to a graph using vertices and paths between these vertices.

There are two possible approaches to this problem:
 
- We put a vertex at every coordinate in the grid, along with edges to all neighbors. 

- We predetermine shortest paths between two locations, and  make only this path availiable. Every vertex is then a hub for different paths: the robots can change direction here.

In this analyisis, we will use the second representation: it is far more scaleable in size of the grid. Subpaths can also be precalculated between frequently visited nodes, utilising a maximum capacity of these paths.

 If computational power is of no consequence, however, the first representation might be preferred.

# Solving this problem for one bot

Having translated the grid to a graph, this has become a well-studied problem. Many algorithms have been devised for this purpose, but in light of the overarching problem (solving this for many bots, without collisions) a simple solution is preferred, so that it can easily be modified.

One such candidate would be Dijkstra's algorithm. While there are many optimalisations possible for this algorithm, its simplicity makes it a good fit for this problem.

This algorithm works, at its base, using a greedy approach: at every step in the algorithm, it will add the currently cheapest path from a previously reached node to a new node to its 'active' paths. This will always result in the shortest possible path to the new node.

The key that we will exploit in this algorithm to prevent collisions from happening is the cost function of the paths.

# Solving this problem for many bots

This problem, but for a series of bots, cannot be solved by simply running a series of dijkstra algorithms over the base graph: two robots cannot use the the same path concurrently. One of the robots will need to allow the other to go first.

As such, we will need to change our underlying graph after every run to incorporate the paths that are already in use at a current time.

To have the dijkstra algorithm take into account the used paths, we can determine the cost of a path in function of the starttime you want to use the path on, and the previous uses of this path by other bots. Given those two parameters, a suitable cost can be decided for this path: in the example given above, the second bot could have cost + 1 for this path and in doing so we can model the one-turn-wait. 
The same works when a bot is travelling on the path in the opposite direction: we would first have to wait the remaining time the path is in use, and only then would we be able to use the path ourselves, incurring double the cost.

We must be careful to always pick the lowest possible cost in using this specific path: should a shorter path exist by going in a loop first, and then using this path (which now has, for some reason, a decreased cost), the algorithm will not find this path: this node has been visited, and will not be checked again. Such paths can be modelled as waiting points, instead.

Waiting points can be found by backtracking on the previous path: some nodes can be specified at waiting points, where two robots can pass each other.

If a path has been decided on by the algorithm, the entity responsible for the cost calculation must then be updated on this, as it needs this info for the next bot. This entity is always aware of the next paths for all the robots.

# A note on multiple stops

Using this translation of the world to a graph, we can also model a robot having multiple stops in the building, and needing to find the shortest path to reach all his destinations.

This problem is known as the travelling salesman problem, and is an np-complete problem. It is therefore computationally infeasible to solve precisely. Heuristics would most likely need to be applied.
