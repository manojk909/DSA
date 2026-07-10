# GRAPH KNOWLEDGE BASE — FILE 3
## Parts 12–18: Bipartite Graphs, Advanced Graphs, Tree-as-Graph, Grid Graph, Graph DP, Greedy, Flow

---

# PART 12 — BIPARTITE GRAPH

## Coloring Check (2-coloring)
A graph is bipartite iff it can be colored with 2 colors such that no edge connects same-colored vertices.

### DFS version
```cpp
vector<int> color; // -1 = uncolored
bool dfs(int u, int c, vector<int> adj[]) {
    color[u] = c;
    for (int v : adj[u]) {
        if (color[v] == -1) { if (!dfs(v, 1-c, adj)) return false; }
        else if (color[v] == c) return false; // same color adjacent -> not bipartite
    }
    return true;
}
```

### BFS version
```cpp
bool bfsCheck(int src, vector<int> adj[], vector<int>& color) {
    queue<int> q; q.push(src); color[src]=0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) {
            if (color[v]==-1) { color[v]=1-color[u]; q.push(v); }
            else if (color[v]==color[u]) return false;
        }
    }
    return true;
}
```
Remember to run this for **every** unvisited component (a disconnected graph can be bipartite in one part and not another).

## Odd Cycle Theorem
**A graph is bipartite if and only if it contains no odd-length cycle.** This is why 2-coloring fails exactly when an odd cycle exists — you'd need a 3rd color to break the parity mismatch.

## Applications
- **Matching**: Bipartite Matching (Hungarian Algorithm / Hopcroft-Karp) — assign jobs to workers, students to schools.
- **Scheduling**: conflict graphs where 2 "colors" = 2 time slots/shifts.
- **Possible Bipartition** (LeetCode): direct 2-coloring application on a "dislike" graph.
- Checking if a graph can be split into 2 independent sets (Max Independent Set is NP-hard in general, but trivial via coloring when the graph is known bipartite).

---

# PART 13 — ADVANCED GRAPHS

## Strongly Connected Components
Covered in depth in File 2, Part 7 (Kosaraju, Tarjan, Condensation Graph).

## Bridges (Cut Edges)
An edge whose removal **increases** the number of connected components.

```cpp
int timer=0; vector<int> tin(N,-1), low(N,-1);
vector<pair<int,int>> bridges;
void dfs(int u, int parent) {
    tin[u]=low[u]=timer++;
    for (int v : adj[u]) {
        if (v == parent) continue; // skip immediate parent edge (handles simple case; for multi-edges pass edge-id instead)
        if (tin[v] != -1) low[u] = min(low[u], tin[v]); // back edge
        else {
            dfs(v, u);
            low[u] = min(low[u], low[v]);
            if (low[v] > tin[u]) bridges.push_back({u, v}); // bridge condition
        }
    }
}
```
**Bridge condition**: `low[v] > tin[u]` means subtree of `v` has **no back edge** reaching `u` or above — so edge `u-v` is the only connection, making it a bridge.

## Articulation Points (Cut Vertices)
A vertex whose removal increases the number of connected components.
```cpp
void dfs(int u, int parent, bool& isRoot) {
    tin[u]=low[u]=timer++;
    int children=0;
    for (int v : adj[u]) {
        if (v == parent) continue;
        if (tin[v] != -1) low[u]=min(low[u], tin[v]);
        else {
            dfs(v, u, isRoot);
            low[u]=min(low[u], low[v]);
            if (low[v] >= tin[u] && parent != -1) isArticulation[u] = true;
            children++;
        }
    }
    if (parent == -1 && children > 1) isArticulation[u] = true; // root special case
}
```
**Root special case**: the root of the DFS tree is an articulation point iff it has **2+ children** in the DFS tree (no back edges connect them, meaning removing root disconnects them).

## Bridge Tree / Block-Cut Tree
- **Bridge Tree**: contract each 2-edge-connected component (component with no bridges) into a single node; connect them via the original bridges. Result is always a **tree**. Useful for answering "does removing this edge disconnect u and v" type queries in O(1) after O(V+E) preprocessing.
- **Block-Cut Tree**: similar idea but for articulation points — each "block" (maximal 2-connected subgraph) becomes a node, articulation points become nodes too, connected based on membership.

## Low Link Values
`low[u]` = smallest `tin` reachable from `u`'s subtree using **at most one back edge**. This single concept powers bridges, articulation points, AND Tarjan's SCC — worth deeply internalizing.

## Euler Path & Euler Circuit
- **Euler Path**: visits **every edge** exactly once (vertices can repeat).
- **Euler Circuit**: Euler path that starts and ends at the same vertex.

**Existence conditions (undirected)**:
- Euler Circuit exists iff graph is connected (ignoring isolated vertices) and **every vertex has even degree**.
- Euler Path (not circuit) exists iff exactly **0 or 2** vertices have odd degree (if 2, path must start/end at those).

**Existence conditions (directed)**:
- Euler Circuit exists iff graph is strongly connected and **in-degree == out-degree** for every vertex.
- Euler Path exists iff at most one vertex has `out-in = 1` (start), at most one has `in-out = 1` (end), and all others balanced.

**Hierholzer's Algorithm** (finds the actual Euler path/circuit in O(E)):
```cpp
void hierholzer(int u, vector<vector<int>>& adj, vector<int>& circuit) {
    while (!adj[u].empty()) {
        int v = adj[u].back(); adj[u].pop_back();
        hierholzer(v, adj, circuit);
    }
    circuit.push_back(u);
}
// resulting circuit vector, reversed, is the Euler path/circuit
```

## Hamiltonian Path & Hamiltonian Cycle
Visits **every vertex** exactly once (edges may not all be used). Unlike Euler paths, **NP-complete** in general — no known polynomial algorithm. Solved via:
- Bitmask DP for small `V` (≤ ~20): `dp[mask][v]` = can we visit exactly the vertex set `mask`, ending at `v`. O(2^V · V²).
- Backtracking with pruning for slightly larger but still small `V`.

## Chinese Postman Problem (Route Inspection)
Find the shortest closed walk that traverses **every edge at least once**. If the graph already has an Euler circuit, that's the answer. Otherwise: find all odd-degree vertices, compute minimum-weight perfect matching between them (shortest paths), duplicate those matched paths' edges to make all degrees even, then run Hierholzer's. Uses Blossom Algorithm or min-cost matching for the pairing step — advanced, rarely required outside specialized CP problems.

---

# PART 14 — TREE AS GRAPH

## Diameter
Longest path between any two nodes in a tree.
**Two-BFS/DFS trick**: BFS from any node → find farthest node `A`. BFS from `A` → farthest node `B`. Distance `A-B` = diameter. Works because trees have no cycles, so this greedy 2-pass approach is provably correct.
Alternative: single DFS computing `(deepest, second-deepest)` child depth per node, diameter = max over all nodes of `deepest+second-deepest`.

## Center
The node(s) minimizing the maximum distance to all other nodes = the middle of the diameter path. Repeatedly remove leaves layer by layer ("peeling" / topological-like BFS from leaves inward) — the last 1 or 2 remaining nodes are the center(s).

## Centroid
The node whose removal minimizes the size of the largest resulting subtree (splits the tree as evenly as possible; at most `n/2` nodes in any piece).
```cpp
int subtreeSize[N];
int findCentroid(int u, int p, int n) {
    for (int v : adj[u]) {
        if (v == p) continue;
        if (subtreeSize[v] > n/2) return findCentroid(v, u, n);
    }
    return u;
}
```
**Centroid Decomposition**: recursively find centroid, remove it, recurse on resulting pieces — builds a decomposition tree of depth O(log n), used for advanced path-query problems (count paths with property X in a tree) in O(n log n).

## LCA (Lowest Common Ancestor)
### Binary Lifting (most common in interviews/CP)
Precompute `up[u][k]` = 2^k-th ancestor of `u`.
```cpp
int LOG = 20;
int up[N][20], depth[N];
void dfs(int u, int p) {
    up[u][0] = p;
    for (int k=1; k<LOG; k++) up[u][k] = up[up[u][k-1]][k-1];
    for (int v : adj[u]) if (v != p) { depth[v]=depth[u]+1; dfs(v,u); }
}
int lca(int u, int v) {
    if (depth[u] < depth[v]) swap(u, v);
    int diff = depth[u]-depth[v];
    for (int k=0; k<LOG; k++) if (diff & (1<<k)) u = up[u][k];
    if (u == v) return u;
    for (int k=LOG-1; k>=0; k--)
        if (up[u][k] != up[v][k]) { u = up[u][k]; v = up[v][k]; }
    return up[u][0];
}
```
**Time**: O(log V) per query after O(V log V) preprocessing.

### Euler Tour + Sparse Table (RMQ-based LCA)
Flatten tree via Euler tour, LCA(u,v) = the minimum-depth node between first occurrences of u and v in the tour → reduces to Range Minimum Query, answerable in O(1) with Sparse Table after O(V log V) preprocessing.

## Heavy-Light Decomposition (HLD)
Decomposes tree into chains such that any root-to-leaf path crosses O(log V) chains. Enables path queries/updates (sum, max, etc. along a tree path) in O(log² V) using a Segment Tree over the flattened chains. Core idea: at each node, the "heavy child" (largest subtree) continues the current chain; other children start new chains.

## Tree DP
Standard pattern: compute a DP value at each node by combining children's DP values (post-order DFS).
```cpp
int dp[N];
void dfs(int u, int p) {
    dp[u] = /* base value */;
    for (int v : adj[u]) if (v != p) {
        dfs(v, u);
        dp[u] = combine(dp[u], dp[v]); // e.g. max, sum, etc.
    }
}
```
Examples: max independent set on tree, diameter via DP, house robber III.

## Rerooting DP
Computes the Tree DP answer **for every node as root** in O(V) total (instead of O(V²) via naive re-run). Two passes:
1. Standard post-order DP (down-up) rooted arbitrarily.
2. Pre-order pass (up-down) propagating "answer if parent were excluded" info from parent to child, combining with siblings' contributions.

Used for: "sum of distances in tree" (LeetCode 834), "for each node, count/size assuming it's root."

---

# PART 15 — GRID GRAPH PATTERNS

Grid = implicit graph, cell = node, adjacency = 4-directional (or 8-directional) neighbors.

| Pattern | Core Technique |
|---|---|
| Flood Fill | DFS/BFS from a seed cell, propagate a value to connected same-valued cells |
| Number of Islands | DFS/BFS from every unvisited land cell, count components |
| Max Area of Island | DFS/BFS returning subtree/component size |
| Number of Distinct Islands | DFS recording **relative shape** (offsets from start cell) as a signature, dedupe via set |
| Number of Enclaves | DFS/BFS from **border** land cells first (mark as "escaped"), remaining land = enclaved |
| Closed Island | Same reverse-border trick but for water/0-surrounded land |
| Shortest Path in Binary Matrix | BFS (8-directional), classic min-moves |
| Rotting Oranges | Multi-source BFS, level = minutes |
| Walls and Gates / 01 Matrix | Multi-source BFS from all "0" cells |
| Pacific Atlantic Water Flow | Reverse BFS/DFS **from both oceans inward**, intersect reachable sets |
| Maze (shortest path with obstacles) | BFS, obstacles = skip in neighbor generation |
| Monster/Fire Escape ("Escape the Spreading Fire") | BFS fire arrival times first, then BFS/binary-search person's path comparing arrival times |

**Universal grid BFS/DFS mistake list**:
- Forgetting bounds check before array access (`nr<0 || nr>=R || nc<0 || nc>=C`).
- Marking visited on **pop** instead of **push** for BFS → duplicate queue entries, subtle bugs.
- Using `vector<vector<bool>>` visited but sharing it across multiple independent flood-fill calls without resetting appropriately.

---

# PART 16 — GRAPH DP

## DP on DAG
Since DAGs have no cycles, DP is well-defined: process nodes in topological order, `dp[v] = f(dp[u] for all u with edge u→v)`.

## Longest Path in a DAG
```cpp
vector<int> topo = kahnTopoSort(...);
vector<int> dp(n, 0); // dp[v] = longest path ending at v
for (int u : topo)
    for (auto [v, w] : adj[u])
        dp[v] = max(dp[v], dp[u] + w);
```
**Note**: Longest path is NP-hard in **general** graphs but polynomial (O(V+E)) on DAGs — always check for cycles first.

## Count Paths (number of distinct paths from A to B in a DAG)
```cpp
vector<long long> ways(n, 0); ways[src] = 1;
for (int u : topo) for (int v : adj[u]) ways[v] += ways[u];
```

## State Compression / Bitmask DP
When a "state" includes a subset of visited elements (≤ ~20 items): `dp[mask][v]` = best value having visited exactly the set `mask`, currently at `v`. Classic: Traveling Salesman Problem — `dp[mask][i]` = min cost to visit all cities in `mask` ending at `i`, O(2^n · n²).

## Tree DP / Rerooting
Covered in Part 14 — Tree DP is Graph DP specialized to acyclic, connected, single-parent structure.

---

# PART 17 — GRAPH GREEDY

## Minimum Edge Removal to make graph acyclic / disconnect
Often reduces to: find a spanning tree/forest, extra edges beyond `V - components` are removable. Related to MST cut-property reasoning.

## Minimum Vertex Cover
Smallest set of vertices touching every edge. **NP-hard in general graphs**, but has a clean polynomial solution on **bipartite graphs** via König's Theorem: `Min Vertex Cover = Max Matching` (computed via Hopcroft-Karp or augmenting-path bipartite matching).

## Maximum Independent Set
Largest set of vertices with no edge between any two. **NP-hard in general**, but on trees solvable in O(V) via Tree DP (`dp[u][0/1]` = max independent set in subtree of u, excluding/including u). On bipartite graphs: `Max Independent Set = V - Max Matching` (complement of min vertex cover).

## Interval Graph Coloring / Activity Selection
Model intervals as a graph where overlapping intervals share an edge; minimum colors needed = minimum "rooms"/resources = graph's chromatic number, which for **interval graphs specifically** equals the maximum clique size and is solvable greedily (sort by start time, sweep with a min-heap of end times) — no need for general graph coloring (NP-hard) because interval graphs are a special "perfect graph" class.

---

# PART 18 — FLOW ALGORITHMS

## Ford-Fulkerson Method
Repeatedly find an augmenting path (any path with residual capacity > 0) from source to sink via DFS, push flow equal to the bottleneck capacity along it, update residual graph (subtract forward, add backward residual edges).
**Time**: O(E · maxFlow) — can be slow if capacities are large and paths are chosen poorly (integer capacities required for the naive bound to hold).

## Edmonds-Karp
Ford-Fulkerson specifically using **BFS** to find augmenting paths (shortest path by edge count each time).
**Time**: O(V·E²) — polynomial regardless of capacity values, a strict improvement in worst-case guarantee over naive Ford-Fulkerson.

## Dinic's Algorithm
Builds a **level graph** via BFS, then finds a **blocking flow** (all augmenting paths within that level graph) via DFS, repeats.
**Time**: O(V²·E) general graphs, O(E·√V) for unit-capacity bipartite matching graphs — the standard choice for competitive programming max-flow problems.
```cpp
// Skeleton
while (bfsBuildLevelGraph()) {
    while (int pushed = dfsBlockingFlow(source, INF)) totalFlow += pushed;
}
```

## Push-Relabel (Preflow-Push)
Different paradigm: allows temporary "excess" flow at nodes (preflow), pushes it based on a "height" labeling, more excess flow gets pushed downhill until stable. **Time**: O(V²√E) with best implementations (e.g., highest-label variant) — asymptotically among the fastest general max-flow algorithms, but more complex to implement correctly.

## Max Flow = Min Cut (Duality Theorem)
The maximum flow from source to sink equals the minimum total capacity of edges that, if removed, disconnect source from sink. After running any max-flow algorithm, the min cut = edges from BFS-reachable (in residual graph) nodes to unreachable nodes.

## Applications
- **Bipartite Matching**: model as flow with source→left nodes (cap 1), left→right (cap 1, per original edge), right→sink (cap 1); max flow = max matching.
- **Min-Cost Max-Flow**: extends flow with edge costs — used for assignment problems with weighted preferences.
- **Image Segmentation**: min-cut partitions pixels into foreground/background.
- **Circulation problems / project selection**: model profit/cost tradeoffs as flow networks.

## Flow Algorithm Comparison

| Algorithm | Time | Notes |
|---|---|---|
| Ford-Fulkerson (DFS) | O(E · maxFlow) | Simple but can be slow, needs integer caps |
| Edmonds-Karp (BFS) | O(V·E²) | Polynomial guarantee |
| Dinic's | O(V²·E), O(E√V) bipartite | CP standard |
| Push-Relabel | O(V²√E) | Fastest asymptotically, complex |

---
**End of File 3.** Continue to File 4 for Pattern Recognition, Interview Tricks, Variations, LeetCode Pattern Grouping, and the full C++ Template Library.
