---
layout: posts
title: "dijkstra algorithm"
date: 2017-07-18
---

## Problem

In a directed weighted graph where all edge weights are nonnegative, find the shortest path from a single vertex to all vertices. For example, find the distance from San Francisco to all other cities in the US. 

## Idea

We maintain a set S (visited) of vertices whose final shortest path (min weights) from the source vertex s have already been determined. Then it repeatedly select the vertex u from (V -S) with the minimum shortest path, adds u to set S and relax all edges leaving u. Each time we add a u to S, the distance from s to u is shortest of all. For detailed proof, see [CLRS](https://mitpress.mit.edu/books/introduction-algorithms). 

The priority in the queue is the distance from the source vertex as of current state. We initialize all vertices with distance infinity from source code, and distance is 0 for the source vertex s, as the distance from s to s is 0. 

Then we expand the vertex from the priority queue with the least distance from source vertex. We iterate all its neighbors and update(relax) them if the distance is smaller than before. If there is a tie, we pick one randomly. Keep doing it until the queue is empty, meaning all vertice have been expanded.

Note that, one vertex can be expanded once and only once. After expanded, the shortest path of the vertex is determined. However, before expanded, a vertex's distance from source s can be updated multiple times. Also, if we use Dijkstra's Algorithm to find the shortest path from a source vertex to another vertex, then after the destination vertex is expanded, the algorithm should be terminated.

## Python implementation

```python
import heapq

def dijkstra(adj, costs, s, t):
    '''
    :param adj: adjacent list representation of a undirected weighted graph
    :param costs: weight of edges
    :param s: source node
    :param t: dest node
    :return: Return predecessors and min distance if there exists a shortest path
        from s to t; Otherwise, return None
    '''
    pq = []  # pq of mutable entry
    entry_finder = {}  # vertex -> [d[v], v]
    dist = {s: 0}  # vertex -> minimal distance
    pre = {}  # predecessor
    visited = set()
    REMOVED = '<removed-vertex>' # placeholder for update heapq

    for v in adj.keys():
        dist[v] = float('inf')
        entry = [dist[v], v]
        entry_finder[v] = entry
        pre[v] = None
        pq.append(entry)

    heapq.heapify(pq)
    dist[s] = 0
    entry_finder[s][0] = 0
    # the correct way to update a heap in python is:
    # use a placeholder for old one, just leave it to pop after updating
    # incorrect way of using private methods: 
    # heapq._siftdown(pq, 0, pq.index(entry_finder[s]))
    while pq:
        # expand the min dist node
        # cost, parent, u = heapq.heappop(pq)
        _, u = heapq.heappop(pq)
        if u is not REMOVED and u not in visited:
            # pre[u] = parent
            visited.add(u)
            if u == t:
                return pre, dist[u]
            # generate all neighbors
            for v in adj.get(u, []):
                if dist[v] > dist[u] + costs[u, v]:
                    dist[v] = dist[u] + costs[u, v]
                    # update mutable entry in entry_finder and heap
                    old_entry = entry_finder.pop(v)
                    old_entry[-1] = REMOVED  # update dist(pq key)
                    new_entry = [dist[v], v]
                    entry_finder[v] = new_entry
                    heapq.heappush(pq, new_entry)
                    pre[v] = u  # update predessor
    return None


def make_undirected(cost):
    ucost = {}
    for k, w in cost.items():
        ucost[k] = w
        ucost[(k[1],k[0])] = w
    return ucost

if __name__=='__main__':

    # adjacent list
    adj = { 1: [2,3,6],
            2: [1,3,4],
            3: [1,2,4,6],
            4: [2,3,5,7],
            5: [4,6,7],
            6: [1,3,5,7],
            7: [4,5,6]}

    # edge costs
    cost = { (1,2):7,
            (1,3):9,
            (1,6):14,
            (2,3):10,
            (2,4):15,
            (3,4):11,
            (3,6):2,
            (4,5):6,
            (5,6):9,
            (4,7):2,
            (5,7):1,
            (6,7):12}

    cost = make_undirected(cost)

    s, t = 1, 7
    predecessors, min_cost = dijkstra(adj, cost, s, t)
    c = t
    path = [c]
    print('min cost:', min_cost)
    while predecessors.get(c):
        c = predecessors[c]
        path.append(c)
    path.reverse()
    print('shortest path:', path)

'''
class implementation
http://www.bogotobogo.com/python/python_Dijkstras_Shortest_Path_Algorithm.php
'''
```
