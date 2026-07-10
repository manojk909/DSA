# GRAPH KNOWLEDGE BASE — FILE 1
## Parts 1–5: Fundamentals, Representation, Traversals

---

# PART 1 — GRAPH FUNDAMENTALS

## What is a Graph?
A graph `G = (V, E)` is a set of **vertices (V)** connected by **edges (E)**. It models *relationships*, not sequences (arrays) or hierarchies-only (trees — which are actually a restricted graph).

## Why Graphs Exist
Almost every real-world "network" is a graph: road networks, social networks, dependency chains, state machines, circuit boards, web pages + hyperlinks. Any problem with entities + relationships between entities is a graph problem in disguise. Recognizing the disguise is 50% of graph mastery (see Part 19).

## Core Terminology

| Term | Meaning | Intuition |
|---|---|---|
| Vertex (Node) | An entity | A city |
| Edge | A connection between 2 vertices | A road |
| Path | Sequence of vertices, no repeated vertex | A route with no revisits |
| Walk | Sequence of vertices, **repeats allowed** | Wandering, can revisit |
| Trail | Walk with no repeated **edge** (vertices may repeat) | Reuse a city, not a road |
| Cycle | Path that starts and ends at same vertex | A loop route |
| Connected Component | Maximal set of vertices reachable from each other | An "island" of the graph |
| Degree | # edges touching a vertex | How many roads leave a city |
| In-degree | # incoming edges (directed graphs) | Roads entering |
| Out-degree | # outgoing edges (directed graphs) | Roads leaving |
| Self loop | Edge from a vertex to itself | A road that loops back to same city |
| Parallel edge | 2+ edges between same pair of vertices | Two roads connecting same 2 cities |
| Weight/Cost | Value attached to an edge | Distance, toll, time |
| Tree | Connected, acyclic graph, `V` vertices & `V-1` edges | A graph with zero redundancy |
| Forest | Disjoint union of trees | Multiple unconnected trees |
| DAG | Directed graph, no directed cycle | Dependency graphs, build systems |
| Complete graph | Every pair of vertices connected | Everyone knows everyone |
| Bipartite graph | Vertices split into 2 sets; edges only between sets, never within | Boys ↔ Girls dance pairing, Jobs ↔ Workers |
| Sparse graph | `E ≈ O(V)` | Road networks |
| Dense graph | `E ≈ O(V²)` | Social networks of a small tight group |

## ASCII Intuition
```
Undirected graph:        Directed graph (digraph):      Tree:
   1---2                     1 --> 2                        1
   |   |                     ^     |                       / \
   3---4                     |     v                      2   3
                              4 <-- 3                     /     \
                                                          4       5
```

## Key distinctions to internalize
- **Path vs Walk vs Trail**: Path (no repeat vertex) ⊂ Trail (no repeat edge) ⊂ Walk (anything goes). Almost all interview problems mean "path" when they say "path."
- **Tree = connected + acyclic**. If a graph is connected and has exactly `V-1` edges, it IS a tree — no need to separately check acyclicity.
- **DAG ≠ Tree**. A DAG can have multiple parents per node (diamond shapes); a tree cannot.

---

# PART 2 — GRAPH REPRESENTATION

## 1. Edge List
`vector<tuple<int,int,int>> edges; // (u, v, weight)`

- **Pros**: Simplest to build, best for algorithms that just need to iterate all edges (Kruskal, Bellman-Ford).
- **Cons**: No fast adjacency lookup (can't quickly ask "who are u's neighbors").
- **Time**: Build O(E). Query neighbors O(E).
- **Space**: O(E).
- **When expected**: Kruskal's MST, Bellman-Ford, when edges themselves are the "objects" being sorted/processed.

## 2. Adjacency Matrix
`int adj[N][N];`

- **Pros**: O(1) edge existence check. Simple for dense graphs, works well with Floyd-Warshall, useful when `V` is small (≤ ~2000).
- **Cons**: O(V²) space even if graph is sparse. Iterating neighbors is O(V) instead of O(degree).
- **Time**: Build O(V²) or O(E). Edge check O(1). Iterate neighbors O(V).
- **Space**: O(V²).
- **When expected**: Floyd-Warshall (all pairs), small dense graphs, when you need frequent "is there an edge u-v" queries.

## 3. Adjacency List
`vector<int> adj[N];` or `vector<vector<int>>`

- **Pros**: O(E) space (sparse-friendly), O(degree) neighbor iteration — the default for almost everything.
- **Cons**: O(degree) edge existence check (unless you also keep a set/hashmap).
- **Time**: Build O(V+E). Iterate neighbors O(degree).
- **Space**: O(V+E).
- **When expected**: **Default choice for 90% of problems** — BFS, DFS, Dijkstra, topological sort.

## 4. Weighted Graph Representation
`vector<pair<int,int>> adj[N]; // {neighbor, weight}`
Or `vector<vector<pair<int,int>>>` — pair = (to, weight). For Dijkstra with a priority_queue, store `{dist, node}` in the PQ itself, not in adjacency.

## 5. Directed vs 6. Undirected
- Directed: add edge only `adj[u].push_back(v)`.
- Undirected: add both directions `adj[u].push_back(v); adj[v].push_back(u);` — this single line is the #1 source of bugs (forgetting the reverse edge, or adding it when the graph is actually directed).

## 7. Implicit Graph
No adjacency list is built at all — neighbors are **generated on the fly** via a rule.
Example: Word Ladder (neighbors = words differing by 1 letter), Knight moves (neighbors = 8 offsets), Sudoku solvers. Recognize this: **if you'd need to build a combinatorially huge explicit graph, keep it implicit and generate neighbors lazily.**

## 8. Grid Graph
2D matrix where each cell is a node, edges = adjacency to up/down/left/right (or 8-directional). Represented directly as `grid[i][j]`, neighbors generated via direction arrays:
```cpp
int dx[] = {-1, 1, 0, 0};
int dy[] = {0, 0, -1, 1};
```
No separate adjacency list needed — the grid itself IS the graph.

## 9. State Graph
Nodes aren't "physical" entities but **states**: `(position, keys_collected)`, `(row, col, direction)`, `(remaining_fuel, city)`. Used in BFS/DFS over expanded state spaces (Shortest Path to Get All Keys, Minimum Cost to Reach Destination in Time). Recognize: **if a single "position" isn't enough to determine future moves, expand your state.**

## 10. Functional Graph
Every node has **exactly one outgoing edge** (`next[i]` arrays). Always contains cycles (since finite nodes, one out-edge each → must eventually repeat). Used in: Cycle detection with Floyd's Tortoise-Hare, "Next Greater Node," teleportation problems. Structure looks like a "rho" (ρ): a tail leading into a cycle.

## Representation Comparison Table

| Representation | Space | Neighbor Iter | Edge Check | Best For |
|---|---|---|---|---|
| Edge List | O(E) | O(E) | O(E) | Kruskal, Bellman-Ford |
| Adjacency Matrix | O(V²) | O(V) | O(1) | Floyd-Warshall, small dense |
| Adjacency List | O(V+E) | O(deg) | O(deg) | Default: BFS/DFS/Dijkstra |
| Implicit | O(1) | O(branching factor) | N/A | Word ladder, puzzles |
| Grid | O(V) | O(1) (4 or 8) | O(1) | Islands, flood fill |
| State Graph | O(states) | O(transitions) | N/A | Multi-dimensional BFS |
| Functional | O(V) | O(1) | O(1) | Cycle detection, teleport |

---

# PART 3 — GRAPH TRAVERSALS

## DFS (Depth-First Search)
**Why it works**: explores as deep as possible before backtracking, using a stack (implicit via recursion, or explicit).

### Recursive DFS
```cpp
vector<int> adj[N];
bool visited[N];
void dfs(int u) {
    visited[u] = true;
    // process u
    for (int v : adj[u]) {
        if (!visited[v]) dfs(v);
    }
}
```
**Dry run** on `1-2, 1-3, 2-4`: start dfs(1) → mark 1 → visit 2 → mark 2 → visit 4 → mark 4 → backtrack to 2 → no more neighbors → backtrack to 1 → visit 3 → mark 3 → done. Order: 1,2,4,3.

### Iterative DFS
```cpp
void dfsIter(int src) {
    stack<int> st; st.push(src);
    while (!st.empty()) {
        int u = st.top(); st.pop();
        if (visited[u]) continue;
        visited[u] = true;
        // process u
        for (int v : adj[u]) if (!visited[v]) st.push(v);
    }
}
```
**Note**: iterative DFS order can differ slightly from recursive (reverse push order fixes this if exact order matters).

- **Time**: O(V+E). **Space**: O(V) (recursion stack / explicit stack).
- **Common mistakes**: marking visited *after* popping instead of *before pushing* (causes duplicate stack entries, not wrong but wasteful); stack overflow on deep recursion (V > ~10⁵) — switch to iterative.
- **Pattern recognition**: "explore all," "connected components," "does a path exist," "backtracking," "subtree computations."

## BFS (Breadth-First Search)
**Why it works**: explores level by level using a queue — guarantees shortest path in **unweighted** graphs because it visits nodes in increasing order of distance.

```cpp
void bfs(int src) {
    queue<int> q; q.push(src); visited[src] = true;
    vector<int> dist(N, -1); dist[src] = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (!visited[v]) {
                visited[v] = true;
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }
}
```
**Dry run** same graph `1-2,1-3,2-4`: queue=[1] → pop 1, push 2,3 → queue=[2,3] → pop 2, push 4 → queue=[3,4] → pop 3 → queue=[4] → pop 4. Distances: d1=0, d2=1, d3=1, d4=2.

- **Time**: O(V+E). **Space**: O(V).
- **Common mistake**: marking visited when **popped** instead of when **pushed** → causes duplicates in queue → can blow up complexity or reprocess.

## Multi-source BFS
Push **all** sources into the queue at once with dist=0 before starting. Used when several starting points spread simultaneously (Rotting Oranges, distance to nearest 0 in a matrix).

## 0-1 BFS
For graphs where edge weights are **only 0 or 1**. Use a `deque`: push to front if weight 0, push to back if weight 1. Achieves Dijkstra-like correctness in O(V+E) without a heap.
```cpp
deque<int> dq; dq.push_back(src); dist[src]=0;
while(!dq.empty()){
  int u=dq.front(); dq.pop_front();
  for(auto [v,w]: adj[u]){
    if(dist[u]+w < dist[v]){
      dist[v]=dist[u]+w;
      if(w==0) dq.push_front(v); else dq.push_back(v);
    }
  }
}
```

## Bidirectional BFS
Run BFS simultaneously from source and target, meet in the middle. Cuts search space from `O(b^d)` to `O(2*b^(d/2))`. Used in Word Ladder II style shortest transformation problems where branching factor is high.

## Layer BFS
Standard BFS but explicitly process **one full level at a time** (using `size = q.size()` snapshot each iteration) — needed when you must know level boundaries (Binary Tree Level Order, Rotting Oranges' "minutes" count).

## State BFS
BFS where the "node" is a composite state, e.g. `(row, col, keysBitmask)`. Visited array becomes multi-dimensional. Used in "Shortest Path to Get All Keys," "Minimum Genetic Mutation."

### Traversal cheat table

| Traversal | Guarantees | Structure | Use When |
|---|---|---|---|
| DFS | Full exploration, not shortest path | Stack | Components, cycles, backtracking, topo sort |
| BFS | Shortest path (unweighted) | Queue | Shortest path/moves, level order |
| Multi-source BFS | Shortest from nearest of many sources | Queue | Rotting oranges, nearest 0 |
| 0-1 BFS | Shortest path, weights∈{0,1} | Deque | Weighted-but-binary edges |
| Bidirectional BFS | Faster shortest path | 2 Queues | Huge branching factor |
| State BFS | Shortest path in expanded state space | Queue + multi-dim visited | Extra constraints (keys, fuel, direction) |

---

# PART 4 — DFS COMPLETE

## Tree DFS vs Graph DFS
Tree DFS never revisits a node (no need for a visited array if you track `parent` to avoid going backward). Graph DFS **requires** a visited array since cycles exist.

```cpp
void treeDfs(int u, int parent) {
    for (int v : adj[u]) {
        if (v == parent) continue; // skip the edge back to parent
        treeDfs(v, u);
    }
}
```

## Parent Tracking
Essential to avoid infinitely bouncing between a node and its parent in undirected trees. Also used to reconstruct paths (`parent[v] = u` lets you walk back from destination to source).

## Depth / Subtree
`depth[u] = depth[parent] + 1`. Subtree size computed post-order:
```cpp
int subtreeSize(int u, int p) {
    int sz = 1;
    for (int v : adj[u]) if (v != p) sz += subtreeSize(v, u);
    return sz;
}
```

## Backtracking & Recursive Stack
DFS naturally backtracks — when a recursive call returns, you're back at the parent's context. This is exploited for combinatorial search (permutations, N-Queens) layered on graph structure.

## Entry Time / Exit Time (Euler Tour concept)
```cpp
int timer = 0, tin[N], tout[N];
void dfs(int u, int p) {
    tin[u] = timer++;
    for (int v : adj[u]) if (v != p) dfs(v, u);
    tout[u] = timer++;
}
```
**Property**: `v` is in subtree of `u` iff `tin[u] <= tin[v] <= tout[u]`. This is the backbone of Euler Tour flattening (subtree → range) used for LCA, subtree sum queries with Segment Trees/BIT, Heavy-Light Decomposition.

## Applications of DFS
| Application | How DFS helps |
|---|---|
| Connected Components | Run DFS from every unvisited node; each call = 1 component |
| Cycle Detection | Recursion stack (directed) or parent-check (undirected) — see Part 6 |
| Topological Sort | Post-order DFS, reverse the finish order |
| Bridges | Low-link values compared against `tin` (see Part 13) |
| Articulation Points | Low-link values + child count of root (see Part 13) |
| Tree DP | Combine children's DFS return values at each node |
| Backtracking | DFS over decision tree (not graph per se, but same recursive skeleton) |

---

# PART 5 — BFS COMPLETE

## Shortest Path / Minimum Moves
BFS is optimal for **unweighted shortest path** because it expands the frontier uniformly. Any "minimum number of steps/moves/operations" phrasing is a strong BFS signal.

## Grid BFS Pattern (template used across many problems)
```cpp
int dx[]={-1,1,0,0}, dy[]={0,0,-1,1};
int bfsGrid(vector<vector<int>>& grid, int sr, int sc) {
    int R=grid.size(), C=grid[0].size();
    vector<vector<int>> dist(R, vector<int>(C, -1));
    queue<pair<int,int>> q;
    dist[sr][sc]=0; q.push({sr,sc});
    while(!q.empty()){
        auto [r,c]=q.front(); q.pop();
        for(int d=0; d<4; d++){
            int nr=r+dx[d], nc=c+dy[d];
            if(nr<0||nr>=R||nc<0||nc>=C) continue;
            if(grid[nr][nc]==1) continue; // wall/blocked
            if(dist[nr][nc]!=-1) continue;
            dist[nr][nc]=dist[r][c]+1;
            q.push({nr,nc});
        }
    }
    return dist[R-1][C-1];
}
```

## Named Problem Patterns (all = BFS variants)
- **Knight moves**: implicit graph, 8 offset moves, BFS for min moves.
- **Rotting Oranges**: multi-source BFS, track minutes as BFS levels.
- **Nearest Exit from Maze**: BFS from start, first boundary cell reached = answer.
- **Flood Fill**: BFS/DFS both work, propagate a value/color to connected region.
- **Distance Transform / 01 Matrix**: multi-source BFS from all 0-cells simultaneously.
- **Walls and Gates**: multi-source BFS from all gates.
- **Fire Spread / Virus Spread (e.g. "Escape the Fire")**: run BFS for the spreading entity first to get its arrival times, then BFS/simulate the escaping entity comparing against those times.
- **Level graph** (used in Dinic's max-flow): BFS assigns levels to build a layered DAG before DFS-based blocking flow.
- **Pacific Atlantic**: reverse thinking — BFS/DFS **outward** from ocean borders instead of from every cell (avoids O(V²)).

## Key Insight: "Reverse BFS" trick
For problems like Pacific Atlantic Water Flow, Walls and Gates, or Fire spread — instead of asking "can this cell reach the target" (expensive, from every cell), flip it: BFS **from the target(s) backward**. This single trick converts many O(V·(V+E)) brute forces into O(V+E).

---
**End of File 1.** Continue to File 2 for Cycle Detection, Connected Components, Topological Sort, Shortest Path Algorithms, MST, and DSU.
