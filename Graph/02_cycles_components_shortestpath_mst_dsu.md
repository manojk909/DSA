# GRAPH KNOWLEDGE BASE — FILE 2
## Parts 6–11: Cycle Detection, Connected Components, Topological Sort, Shortest Paths, MST, DSU

---

# PART 6 — CYCLE DETECTION

## Undirected Graphs

### Method 1: DFS with parent tracking
A cycle exists if you reach an already-visited vertex that is **not** your immediate parent.
```cpp
bool dfs(int u, int parent, vector<int> adj[], vector<bool>& vis) {
    vis[u] = true;
    for (int v : adj[u]) {
        if (!vis[v]) { if (dfs(v, u, adj, vis)) return true; }
        else if (v != parent) return true; // back edge to non-parent = cycle
    }
    return false;
}
```
**Why it works**: in an undirected DFS tree, any edge to an already-visited node that isn't the parent is a "back edge," which closes a cycle.

### Method 2: BFS with parent tracking
Same idea — if you dequeue a neighbor that's visited and isn't the parent that pushed it, cycle found.

### Method 3: DSU (Union-Find)
Process edges one by one; if `find(u) == find(v)` **before** union, adding this edge creates a cycle.
```cpp
for (auto [u,v] : edges) {
    if (find(u) == find(v)) return true; // cycle
    unite(u, v);
}
```
**When to use DSU over DFS**: when edges arrive incrementally (online), or you need cycle detection alongside connectivity counting (Redundant Connection problem).

## Directed Graphs

### Method 1: DFS recursion stack (3-color / gray-set)
```cpp
// 0 = unvisited, 1 = in current recursion stack (GRAY), 2 = fully done (BLACK)
vector<int> state(N, 0);
bool dfs(int u) {
    state[u] = 1;
    for (int v : adj[u]) {
        if (state[v] == 1) return true;       // back edge -> cycle
        if (state[v] == 0 && dfs(v)) return true;
    }
    state[u] = 2;
    return false;
}
```
**Why it works**: a cycle in a directed graph = an edge pointing back into the **current DFS call stack**, i.e., to a GRAY node. An edge to a BLACK node is a "cross/forward edge," which is safe (already fully explored, not an ancestor).

### Method 2: Color method (same as above, just named differently — White/Gray/Black is the classic CLRS terminology)

### Method 3: Topological Sort (Kahn's Algorithm) as a cycle detector
Run Kahn's BFS-based topological sort. If the number of nodes processed < V, a cycle exists (remaining nodes are stuck in a cycle since their in-degree never drops to 0).
```cpp
// after Kahn's algorithm:
if (topoOrder.size() != V) cycleExists = true;
```
**When to use this over DFS-coloring**: when you need topological order anyway (Course Schedule II) — get cycle detection "for free."

## When Each Method Wins

| Scenario | Best Method |
|---|---|
| Static undirected graph, one-shot check | DFS parent-tracking |
| Edges added incrementally / need connectivity too | DSU |
| Directed graph, need actual cycle detection | DFS 3-color |
| Directed graph, need topo order **and** cycle check | Kahn's BFS |

---

# PART 7 — CONNECTED COMPONENTS

## Connected Components (undirected)
Run DFS/BFS from every unvisited vertex; each run = one component.
```cpp
int countComponents(int n, vector<int> adj[]) {
    vector<bool> vis(n, false);
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (!vis[i]) { count++; dfs(i, adj, vis); }
    }
    return count;
}
```

## Weakly Connected (directed)
Ignore edge direction, treat as undirected, count components normally.

## Strongly Connected (directed)
A component where **every** vertex can reach **every other** vertex following edge directions. Requires specialized algorithms:

### Kosaraju's Algorithm (2-pass DFS)
1. DFS the original graph, push nodes onto a stack by **finish time** (like topological order).
2. Reverse (transpose) the graph.
3. Pop nodes from the stack; for each unvisited node, DFS on the **reversed** graph — each DFS tree = one SCC.
```cpp
// Step 1: fill finish-time stack
void dfs1(int u, vector<int> adj[], vector<bool>& vis, stack<int>& st){
    vis[u]=true;
    for(int v: adj[u]) if(!vis[v]) dfs1(v, adj, vis, st);
    st.push(u);
}
// Step 2: DFS on transpose, popping from stack
void dfs2(int u, vector<int> radj[], vector<bool>& vis, vector<int>& comp){
    vis[u]=true; comp.push_back(u);
    for(int v: radj[u]) if(!vis[v]) dfs2(v, radj, vis, comp);
}
```
**Time**: O(V+E). Simple, 2 DFS passes + graph transpose.

### Tarjan's Algorithm (single-pass, low-link)
Uses `tin[]`/`low[]` (see Part 13) + an explicit stack. When `low[u] == tin[u]`, `u` is the root of an SCC — pop the stack until you pop `u`.
```cpp
int timer=0; vector<int> tin(N,-1), low(N,-1);
stack<int> st; vector<bool> onStack(N,false);
void tarjan(int u){
    tin[u]=low[u]=timer++;
    st.push(u); onStack[u]=true;
    for(int v: adj[u]){
        if(tin[v]==-1){ tarjan(v); low[u]=min(low[u],low[v]); }
        else if(onStack[v]) low[u]=min(low[u],tin[v]);
    }
    if(low[u]==tin[u]){
        while(true){
            int v=st.top(); st.pop(); onStack[v]=false;
            // v belongs to this SCC
            if(v==u) break;
        }
    }
}
```
**Time**: O(V+E), single pass — preferred in competitive programming for speed, though Kosaraju is easier to reason about and code correctly under pressure.

## Condensation Graph
Collapse every SCC into a single super-node. The condensation graph is **always a DAG**. This is a powerful technique: reduces a cyclic directed graph to a DAG so you can apply topological-sort-based DP on it (e.g., "longest path visiting most SCCs").

## Applications
- Detecting deadlock cycles.
- 2-SAT solving (variables/negations as nodes, implications as edges, SCC membership determines satisfiability).
- Simplifying a graph for DP (condensation).
- Social network "clique/community" approximation via SCC on directed follow-graphs.

---

# PART 8 — TOPOLOGICAL SORT

Only valid on a **DAG**. A topological order is a linear ordering of vertices such that for every directed edge `u→v`, `u` comes before `v`.

## DFS-based Topological Sort
```cpp
void dfs(int u, vector<int> adj[], vector<bool>& vis, stack<int>& st){
    vis[u]=true;
    for(int v: adj[u]) if(!vis[v]) dfs(v, adj, vis, st);
    st.push(u); // push on finish (post-order)
}
// topo order = pop stack until empty
```
**Why it works**: a node is pushed only after all its descendants are fully processed, so popping gives dependencies-first order automatically.

## Kahn's Algorithm (BFS-based, in-degree)
```cpp
vector<int> kahn(int n, vector<int> adj[]) {
    vector<int> indeg(n, 0);
    for (int u = 0; u < n; u++) for (int v : adj[u]) indeg[v]++;
    queue<int> q;
    for (int i = 0; i < n; i++) if (indeg[i]==0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u]) if (--indeg[v]==0) q.push(v);
    }
    return order; // if order.size() < n -> cycle exists
}
```
**Advantage over DFS version**: naturally detects cycles, gives BFS-level "layers" (useful for parallel scheduling — all nodes in the same layer can run simultaneously).

## Classic Problems & Their Tricks
| Problem | Trick |
|---|---|
| Course Schedule | Cycle detection = "can all courses be finished" |
| Course Schedule II | Return the actual topo order (Kahn's) |
| Alien Dictionary | Build edges from **adjacent word comparisons** (first differing character = edge); careful with the "prefix is longer" invalid case |
| Build Order | Standard topo sort, dependencies = edges |
| Dependency Resolution (task scheduler) | Topo sort + sometimes combined with a priority queue for lexicographic/priority tie-breaking |

**Common mistake in Alien Dictionary**: only comparing adjacent word pairs (not all pairs) — this is correct and sufficient, but people over-engineer it. Also must handle the case where a shorter word is NOT a prefix-consistent predecessor of a longer one that starts identically (invalid ordering, return "").

---

# PART 9 — SHORTEST PATH ALGORITHMS

## BFS
Unweighted graphs. O(V+E). Covered in Part 3/5.

## 0-1 BFS
Weights ∈ {0,1}. O(V+E). Covered in Part 3.

## Dijkstra's Algorithm
Non-negative weights only. Greedy: always finalize the closest unvisited node.
```cpp
vector<int> dijkstra(int src, int n, vector<pair<int,int>> adj[]) {
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src]=0; pq.push({0,src});
    while(!pq.empty()){
        auto [d,u]=pq.top(); pq.pop();
        if (d>dist[u]) continue; // stale entry, skip
        for (auto [v,w] : adj[u]) {
            if (dist[u]+w < dist[v]) {
                dist[v]=dist[u]+w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```
**Time**: O((V+E) log V) with binary heap. **Space**: O(V+E).
**Why it fails with negative edges**: once a node is "finalized" (popped from PQ), Dijkstra assumes its distance can never improve — a negative edge later could violate this.

## Bellman-Ford
Handles negative edges; detects negative cycles. Relax all edges `V-1` times.
```cpp
bool bellmanFord(int src, int n, vector<array<int,3>>& edges, vector<long long>& dist) {
    dist.assign(n, LLONG_MAX); dist[src]=0;
    for (int i=0; i<n-1; i++)
        for (auto& [u,v,w] : edges)
            if (dist[u]!=LLONG_MAX && dist[u]+w < dist[v]) dist[v]=dist[u]+w;
    // one more pass: if any edge still relaxes -> negative cycle
    for (auto& [u,v,w] : edges)
        if (dist[u]!=LLONG_MAX && dist[u]+w < dist[v]) return false; // negative cycle
    return true;
}
```
**Time**: O(V·E). **Space**: O(V).
**Relaxation proof intuition**: shortest path has at most `V-1` edges (no point repeating a vertex), so `V-1` full relaxation rounds guarantee convergence — round `i` guarantees correctness for paths of ≤ `i` edges.

## SPFA (Shortest Path Faster Algorithm)
Bellman-Ford optimized with a queue (only re-relax from nodes whose distance just improved). Average case much faster than plain Bellman-Ford, but worst case still O(V·E) — and adversarial inputs can break it. Mostly seen on Codeforces; many judges have anti-SPFA test cases, so prefer Dijkstra/Bellman-Ford in interviews.

## Floyd-Warshall
All-pairs shortest path via DP over "allowed intermediate nodes."
```cpp
for (int k=0; k<n; k++)
    for (int i=0; i<n; i++)
        for (int j=0; j<n; j++)
            if (dist[i][k]!=INF && dist[k][j]!=INF)
                dist[i][j] = min(dist[i][j], dist[i][k]+dist[k][j]);
```
**Time**: O(V³). **Space**: O(V²). Works with negative edges (not negative cycles — check `dist[i][i] < 0` after running to detect one). Best when `V` is small (≤ ~500) and you need **all** pairs, not just one source.

## Johnson's Algorithm
All-pairs shortest path for **sparse** graphs with negative edges (better than Floyd-Warshall's O(V³) when E << V²).
1. Add a virtual source node connected to all vertices with weight 0.
2. Run Bellman-Ford from it to get potentials `h[v]`.
3. Reweight every edge: `w'(u,v) = w(u,v) + h[u] - h[v]` (now all non-negative).
4. Run Dijkstra from every vertex on reweighted graph.
**Time**: O(V·E log V) — much better than O(V³) when sparse.

## A* Search
Dijkstra + a **heuristic** `h(v)` estimating distance to target, guiding search toward the goal faster. Priority = `g(v) + h(v)` (cost so far + estimated remaining cost). Must use an **admissible** heuristic (never overestimates) for correctness — e.g., Manhattan/Euclidean distance in grid pathfinding.

## Comparison Table

| Algorithm | Time | Negative Edges? | Negative Cycle Detection | All-Pairs? | Best Use Case |
|---|---|---|---|---|---|
| BFS | O(V+E) | N/A (unweighted) | N/A | No | Unweighted shortest path |
| 0-1 BFS | O(V+E) | No | N/A | No | Weights ∈ {0,1} |
| Dijkstra | O((V+E)logV) | No | No | No | Non-negative weighted, single source |
| Bellman-Ford | O(V·E) | Yes | Yes | No | Negative edges, single source |
| SPFA | O(V·E) worst, faster avg | Yes | Yes | No | CP only, avoid on adversarial judges |
| Floyd-Warshall | O(V³) | Yes | Yes (diag<0) | Yes | Small dense graph, all pairs |
| Johnson | O(V·E logV) | Yes | Yes (via BF step) | Yes | Sparse graph, all pairs, negative edges |
| A* | ~O((V+E)logV) w/ pruning | No | N/A | No | Pathfinding with a good heuristic |

---

# PART 10 — MINIMUM SPANNING TREE (MST)

A spanning tree connects all `V` vertices with exactly `V-1` edges, no cycles. MST = the one with minimum total edge weight.

## Kruskal's Algorithm
Sort edges by weight; greedily add if it doesn't form a cycle (checked via DSU).
```cpp
sort(edges.begin(), edges.end()); // by weight
int mstWeight = 0, edgesUsed = 0;
for (auto& [w,u,v] : edges) {
    if (find(u) != find(v)) {
        unite(u, v);
        mstWeight += w;
        edgesUsed++;
    }
}
```
**Time**: O(E log E) for sort + O(E·α(V)) for DSU ≈ O(E log E). Best for **sparse** graphs / when edges are given as a list.

## Prim's Algorithm
Grow the MST one vertex at a time from a start node, always picking the cheapest edge crossing the current tree's boundary (via a min-heap).
```cpp
priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // {weight, node}
vector<bool> inMST(n, false);
pq.push({0, 0});
int mstWeight = 0;
while (!pq.empty()) {
    auto [w,u] = pq.top(); pq.pop();
    if (inMST[u]) continue;
    inMST[u] = true; mstWeight += w;
    for (auto [v,wt] : adj[u]) if (!inMST[v]) pq.push({wt, v});
}
```
**Time**: O(E log V) with binary heap. Best for **dense** graphs / adjacency-list-given graphs, or when you conceptually "grow from a source."

## Boruvka's Algorithm
Each component simultaneously picks its cheapest outgoing edge, merge all at once (repeat until 1 component). O(E log V) — used mainly when parallelism matters, or as a building block for advanced MST-on-implicit-graph techniques (e.g., Euclidean MST).

## Greedy Proof (Cut Property)
For any cut (partition of vertices into 2 sets), the minimum-weight edge crossing the cut **must** be in some MST. This justifies both Kruskal (each edge considered is the min crossing some cut at that point) and Prim (always picks the min crossing edge of the growing set).

## Applications
- Network design (minimum cost to connect all cities/computers).
- Approximation algorithms for TSP (MST gives a 2-approximation).
- Clustering (remove the k-1 largest MST edges to get k clusters — "single-linkage clustering").
- Image segmentation.

## Kruskal vs Prim

| | Kruskal | Prim |
|---|---|---|
| Best for | Sparse graphs, edge list given | Dense graphs, adjacency list given |
| Data structure | DSU | Min-heap |
| Complexity | O(E log E) | O(E log V) |
| Style | Global greedy (sort all edges) | Local greedy (grow from a node) |

---

# PART 11 — DISJOINT SET UNION (DSU / Union-Find)

## Core Structure
```cpp
vector<int> parent, rank_;
void init(int n) { parent.resize(n); iota(parent.begin(), parent.end(), 0); rank_.assign(n,0); }
int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]); // path compression
    return parent[x];
}
void unite(int x, int y) {
    int rx = find(x), ry = find(y);
    if (rx == ry) return;
    if (rank_[rx] < rank_[ry]) swap(rx, ry);
    parent[ry] = rx;
    if (rank_[rx] == rank_[ry]) rank_[rx]++;
}
```

## Path Compression
While finding the root, re-point every visited node directly to the root — flattens the tree for future queries.

## Union by Rank / Union by Size
Always attach the smaller/shallower tree under the larger/deeper one's root — prevents the DSU forest from becoming a long chain.

**Together**, path compression + union by rank give amortized **O(α(V))** per operation (α = inverse Ackermann, effectively constant ≤ 4 for any realistic input).

## Applications
| Problem | How DSU is used |
|---|---|
| Connected Components | Union all edges, count distinct roots |
| Cycle Detection (undirected) | `find(u)==find(v)` before union = cycle |
| Redundant Connection | First edge that connects an already-connected pair = the answer |
| Accounts Merge | Union accounts sharing an email, then group by root |
| Dynamic Connectivity | Handle "connect(u,v)" queries online; for **disconnect** queries, need offline processing (process queries in reverse) since DSU has no native "un-union" |
| Kruskal's MST | Core cycle-avoidance mechanism |
| Number of Islands II (online) | Union newly added land cells with neighbors |

**Key limitation to remember**: DSU supports union and find efficiently but **not deletion/disconnection** natively. Problems requiring edge removal are typically solved by reversing the query order (process deletions as "the state before they happened," working backward with unions only).

---
**End of File 2.** Continue to File 3 for Bipartite Graphs, Advanced Graphs (SCC/Bridges/Articulation/Euler), Tree-as-Graph, Grid Graph patterns, Graph DP, Greedy, and Flow.
