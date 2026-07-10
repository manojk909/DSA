# GRAPH KNOWLEDGE BASE — FILE 4
## Parts 19–23: Pattern Recognition, Interview Tricks, Variations, LeetCode Patterns, C++ Templates

---

# PART 19 — GRAPH PATTERN RECOGNITION (Decision Tree)

The single highest-leverage skill: converting **English clues in the problem statement** into **algorithm choice**.

## Master Decision Tree

```
Is it explicitly about a grid / matrix?
├── YES → is it about counting regions / flood fill? → DFS/BFS components
│         is it about shortest steps? → BFS (multi-source if multiple starts)
│         is it about reachability from 2 sets? → reverse BFS from both, intersect
│
└── NO → Is the graph WEIGHTED?
    ├── NO (unweighted) → "minimum moves/steps/operations/levels"? → BFS
    │                     "all paths / connectivity / backtracking"? → DFS
    │                     "groups / clusters / merge"? → DSU or Components
    │                     "order / dependency / prerequisite"? → Topological Sort
    │                     "cycle exists?" → DFS (undirected: parent-track; directed: 3-color)
    │
    └── YES (weighted) → any NEGATIVE edges?
        ├── YES → need ALL PAIRS? → Floyd-Warshall (dense) / Johnson (sparse)
        │         need SINGLE SOURCE? → Bellman-Ford (+ negative cycle check)
        │
        └── NO → weights only 0/1? → 0-1 BFS
                 need ALL PAIRS, small V? → Floyd-Warshall
                 need SINGLE SOURCE? → Dijkstra
                 need MINIMUM SPANNING (connect all, min total cost)? → Kruskal/Prim
                 need MAX FLOW / bottleneck / matching? → Ford-Fulkerson family
```

## Keyword → Algorithm Cheat Table

| Question says... | → Use |
|---|---|
| "minimum moves / minimum steps / fewest operations" | BFS |
| "weighted shortest path", "minimum cost path" | Dijkstra |
| "negative edges" | Bellman-Ford |
| "negative cycle" | Bellman-Ford (extra pass) |
| "all pairs shortest path" | Floyd-Warshall / Johnson |
| "dependency / prerequisite / build order / course schedule" | Topological Sort |
| "groups / provinces / clusters / merge accounts" | DSU or Connected Components |
| "connect all with minimum cost" | MST (Kruskal/Prim) |
| "islands / regions in grid" | Grid DFS/BFS |
| "tree" explicitly mentioned + "subtree / rooted computation" | DFS + Tree DP |
| "diameter / farthest node" | 2×BFS or Tree DP |
| "ancestor query, kth ancestor" | Binary Lifting / LCA |
| "bipartition / can be split into 2 groups / no conflicts" | 2-coloring / Bipartite check |
| "matching / assignment / pairing" | Bipartite Matching (flow-based) |
| "max flow / bottleneck / capacity" | Ford-Fulkerson / Dinic's |
| "critical connections / cannot be removed" | Bridges (Low-link) |
| "single point of failure" | Articulation Points |
| "cycle detection, redundant connection" | DSU |
| "visit every edge once" | Euler Path/Circuit |
| "visit every vertex once" | Hamiltonian Path (bitmask DP, small n) |
| "0/1 weighted", "some free moves" | 0-1 BFS |
| "state includes extra info (keys, fuel, time)" | State-space BFS/DFS |
| "strongly connected / mutual reachability" | Kosaraju / Tarjan |

**How to use this table under pressure**: underline every quantitative/qualitative clue word in the problem statement first, map each to a row, then cross-check with graph size constraints (`V, E ≤ ?`) which often rules out one candidate (e.g., `V ≤ 20` screams bitmask DP; `V ≤ 10^5` rules out Floyd-Warshall's O(V³)).

---

# PART 20 — COMPLETE GRAPH TRICKS (Interview-Level)

| Trick | What it does | Example use |
|---|---|---|
| Visited array timing | Mark visited on **push** (BFS) not pop, to avoid duplicate queue entries | Any BFS |
| Parent tracking | Skip the edge back to immediate parent in undirected DFS/BFS | Cycle detection, path reconstruction |
| Color/state array (0/1/2) | Distinguish unvisited / in-progress / done for directed cycle detection | Directed cycle detection |
| Timestamp (tin/tout) | Encode subtree ranges, ancestor checks in O(1) | LCA, subtree queries, Euler tour |
| Low-link values | Track earliest reachable ancestor via back edges | Bridges, articulation points, Tarjan SCC |
| Discovery time | Order of first visit, used alongside low-link | Same as above |
| Lazy deletion | Skip stale priority-queue entries instead of removing them | Dijkstra with decrease-key simulation |
| Priority Queue optimization | Replace O(V²) Dijkstra/Prim with O(E log V) using a heap | Dijkstra, Prim |
| Path compression | Flatten DSU trees during find() | DSU |
| Coordinate compression | Map large/sparse coordinate values to dense indices | Grid/graph problems with huge coordinate ranges |
| Virtual/dummy nodes | Add an artificial node to simplify multi-source/multi-sink logic | Super source/sink in flow, multi-source BFS via a "meta source" |
| State graph expansion | Encode extra dimensions (keys, direction, parity) into the node identity | Shortest Path to Get All Keys |
| Bitmask state | Represent a subset (visited set, key set) as an integer for O(1) transitions | TSP, Hamiltonian path, keys problems |
| Direction array | `dx[], dy[]` for compact neighbor generation | Grid BFS/DFS |
| Reverse/Transpose graph | Build the graph with all edges flipped | Kosaraju's SCC, "who can reach me" queries |
| Edge relaxation | Update `dist[v]` if a shorter path is found via edge `(u,v)` | Dijkstra, Bellman-Ford |
| Reverse BFS | BFS backward from target(s) instead of forward from every source | Pacific Atlantic, Walls and Gates |
| Multi-source initialization | Push all sources into the queue at distance 0 before starting BFS | Rotting Oranges, 01 Matrix |
| Meet-in-the-middle BFS | Run BFS from both ends, stop when frontiers intersect | Word Ladder II, huge branching factor |
| Pruning | Cut off search branches that can't improve the answer | Backtracking-heavy graph search |
| Memoization | Cache repeated subproblem results (works because state space is finite) | DAG DP, state-space search |
| Graph compression / Condensation | Collapse SCCs into single nodes to get a DAG | Post-SCC DP |
| Super source / Super sink | Single artificial node connected to all sources/sinks with 0/∞ capacity | Multi-source shortest path, flow problems |

---

# PART 21 — COMMON VARIATIONS

| Variation | Defining trait | Special handling |
|---|---|---|
| Grid Graph | Cells as nodes, 4/8-directional edges | No explicit adjacency list — generate neighbors via direction arrays |
| Hexagonal Graph | 6-directional adjacency (used in hex-based games/maps) | Custom neighbor offset set (axial or offset coordinates) |
| Weighted | Edges carry cost | Requires Dijkstra/Bellman-Ford/Floyd instead of plain BFS |
| Directed | Edges one-way | In-degree/out-degree distinct; reverse graph often needed |
| Undirected | Edges two-way | Always add both directions; watch parent-skip logic |
| Functional Graph | Exactly 1 outgoing edge per node | Always has a cycle; use Floyd's tortoise-hare or iterative pointer jumping |
| Tree | Connected + acyclic | No visited array strictly needed if parent-tracked |
| Forest | Disjoint trees | Loop DFS/BFS over all unvisited roots |
| DAG | Directed, no cycle | Enables topological-order DP |
| Complete Graph | All pairs connected | Adjacency matrix often more natural than list |
| Sparse Graph | E ≈ O(V) | Adjacency list, avoid O(V²) algorithms |
| Dense Graph | E ≈ O(V²) | Adjacency matrix OK, O(V²) algorithms tolerable |
| State Space Graph | Nodes = abstract states, not physical entities | Expand state explicitly; visited becomes multi-dimensional |
| Implicit Graph | No explicit adjacency; neighbors generated by rule | Generate on the fly, be mindful of branching factor for BFS/DFS cost |
| Dynamic Graph | Graph changes over time (edges added/removed) | DSU for additions; offline reverse-processing for deletions; or Link-Cut Trees for advanced online dynamic connectivity |
| Online Graph | Queries interleaved with updates, must answer immediately | Requires online-capable structures (DSU for union-only; segment tree on time for full dynamic connectivity) |

---

# PART 22 — LEETCODE PATTERN GROUPING

| Pattern | Core Concept | Algorithms | Difficulty Range | Must-Know Problems |
|---|---|---|---|---|
| Grid Traversal | Flood fill / region counting | DFS, BFS | Easy–Medium | Number of Islands, Max Area of Island, Flood Fill, Rotting Oranges, 01 Matrix, Pacific Atlantic Water Flow, Surrounded Regions |
| Shortest Path (Unweighted) | Level-order expansion | BFS, Multi-source BFS | Medium | Word Ladder, Open the Lock, Shortest Path in Binary Matrix, Nearest Exit from Maze |
| Shortest Path (Weighted) | Greedy relaxation | Dijkstra, Bellman-Ford | Medium–Hard | Network Delay Time, Cheapest Flights Within K Stops, Path With Maximum Probability |
| Cycle Detection | Back-edge / recursion stack | DFS, Kahn's | Medium | Course Schedule, Find Eventual Safe States, Redundant Connection |
| Topological Sort | Dependency ordering | Kahn's, DFS post-order | Medium–Hard | Course Schedule II, Alien Dictionary, Sequence Reconstruction |
| Union-Find | Dynamic connectivity | DSU | Easy–Hard | Number of Provinces, Accounts Merge, Redundant Connection, Satisfiability of Equality Equations |
| Connectivity / Components | Component counting/labeling | DFS/BFS/DSU | Easy–Medium | Number of Connected Components, Friend Circles |
| Bipartite | 2-coloring | DFS/BFS coloring | Medium | Is Graph Bipartite, Possible Bipartition |
| MST | Greedy edge selection | Kruskal, Prim | Medium–Hard | Min Cost to Connect All Points, Connecting Cities With Minimum Cost |
| Advanced (Bridges/AP/SCC) | Low-link values | Tarjan, Kosaraju | Hard | Critical Connections in a Network |
| Tree-specific | Rooted computation | DFS + Tree DP | Medium–Hard | Diameter of Binary Tree, Sum of Distances in Tree, Binary Tree Cameras |
| LCA / Ancestor | Binary lifting / Euler tour | LCA algorithms | Medium–Hard | Lowest Common Ancestor variants, Kth Ancestor of a Tree Node |
| State-Space BFS | Multi-dimensional states | State BFS | Hard | Shortest Path to Get All Keys, Minimum Genetic Mutation, Sliding Puzzle |
| Graph DP | DAG-based DP | Topo-sort DP, bitmask DP | Hard | Longest Increasing Path in a Matrix, Course Schedule III |
| Matching/Flow | Bipartite matching, max flow | Flow algorithms | Hard | Maximum Bipartite Matching variants (rare on LeetCode, common on CF) |

## Curated Lists Cross-Reference
- **Blind 75**: Number of Islands, Clone Graph, Course Schedule, Pacific Atlantic Water Flow, Graph Valid Tree (Premium).
- **Neetcode 150 (Graphs section)**: adds Word Ladder, Alien Dictionary, Redundant Connection, Reconstruct Itinerary, Min Cost to Connect All Points, Swim in Rising Water, Network Delay Time.
- **Top Interview 150**: emphasizes Number of Islands, Course Schedule, Evaluate Division (weighted graph + DFS), Snake and Ladders.
- **FAANG-frequent**: Number of Islands, Course Schedule (I/II), Clone Graph, Word Ladder, Redundant Connection, Network Delay Time, Critical Connections — these repeatedly appear across Google/Meta/Amazon loops.

---

# PART 23 — COMPLETE GRAPH TEMPLATE LIBRARY (C++)

```cpp
// ===================== DFS (Recursive) =====================
vector<int> adj[N];
bool vis[N];
void dfs(int u) {
    vis[u] = true;
    for (int v : adj[u]) if (!vis[v]) dfs(v);
}

// ===================== DFS (Iterative) =====================
void dfsIter(int src) {
    stack<int> st; st.push(src);
    while (!st.empty()) {
        int u = st.top(); st.pop();
        if (vis[u]) continue;
        vis[u] = true;
        for (int v : adj[u]) if (!vis[v]) st.push(v);
    }
}

// ===================== BFS =====================
vector<int> bfs(int src, int n) {
    vector<int> dist(n, -1);
    queue<int> q; q.push(src); dist[src] = 0;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) if (dist[v] == -1) { dist[v] = dist[u]+1; q.push(v); }
    }
    return dist;
}

// ===================== Multi-source BFS =====================
vector<int> multiSourceBFS(vector<int>& sources, int n) {
    vector<int> dist(n, -1);
    queue<int> q;
    for (int s : sources) { dist[s] = 0; q.push(s); }
    while (!q.empty()) {
        int u = q.front(); q.pop();
        for (int v : adj[u]) if (dist[v] == -1) { dist[v] = dist[u]+1; q.push(v); }
    }
    return dist;
}

// ===================== Grid BFS =====================
int dx[] = {-1,1,0,0}, dy[] = {0,0,-1,1};
// see File 1, Part 5 for full template

// ===================== Grid DFS =====================
void gridDfs(vector<vector<int>>& g, int r, int c, vector<vector<bool>>& vis) {
    int R=g.size(), C=g[0].size();
    if (r<0||r>=R||c<0||c>=C||vis[r][c]||g[r][c]==0) return;
    vis[r][c] = true;
    for (int d=0; d<4; d++) gridDfs(g, r+dx[d], c+dy[d], vis);
}

// ===================== Dijkstra =====================
vector<long long> dijkstra(int src, int n, vector<pair<int,int>> adj[]) {
    vector<long long> dist(n, LLONG_MAX);
    priority_queue<pair<long long,int>, vector<pair<long long,int>>, greater<>> pq;
    dist[src]=0; pq.push({0,src});
    while(!pq.empty()){
        auto [d,u]=pq.top(); pq.pop();
        if (d>dist[u]) continue;
        for (auto [v,w] : adj[u])
            if (dist[u]+w < dist[v]) { dist[v]=dist[u]+w; pq.push({dist[v],v}); }
    }
    return dist;
}

// ===================== Bellman-Ford =====================
// see File 2, Part 9 for full template with negative-cycle detection

// ===================== Floyd-Warshall =====================
// see File 2, Part 9

// ===================== DSU =====================
struct DSU {
    vector<int> parent, rnk;
    DSU(int n) : parent(n), rnk(n,0) { iota(parent.begin(), parent.end(), 0); }
    int find(int x) { return parent[x]==x ? x : parent[x]=find(parent[x]); }
    bool unite(int x, int y) {
        int rx=find(x), ry=find(y);
        if (rx==ry) return false;
        if (rnk[rx]<rnk[ry]) swap(rx,ry);
        parent[ry]=rx;
        if (rnk[rx]==rnk[ry]) rnk[rx]++;
        return true;
    }
};

// ===================== Topological Sort (Kahn's) =====================
vector<int> kahnTopo(int n, vector<int> adj[]) {
    vector<int> indeg(n,0);
    for (int u=0;u<n;u++) for (int v: adj[u]) indeg[v]++;
    queue<int> q;
    for (int i=0;i<n;i++) if (!indeg[i]) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u=q.front(); q.pop(); order.push_back(u);
        for (int v: adj[u]) if (--indeg[v]==0) q.push(v);
    }
    return order; // order.size()<n -> cycle
}

// ===================== Kosaraju's SCC =====================
// see File 2, Part 7

// ===================== Tarjan's SCC / Bridges / Articulation =====================
// see File 2 Part 7 and File 3 Part 13 (shared low-link machinery)

// ===================== Prim's MST =====================
// see File 2, Part 10

// ===================== Kruskal's MST =====================
long long kruskal(int n, vector<array<int,3>>& edges /* {w,u,v} */) {
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    long long total = 0;
    for (auto& [w,u,v] : edges) if (dsu.unite(u,v)) total += w;
    return total;
}

// ===================== LCA (Binary Lifting) =====================
// see File 3, Part 14

// ===================== Euler Tour (tin/tout) =====================
// see File 1, Part 4

// ===================== Tree DP (generic) =====================
int dp[N];
void treeDp(int u, int p) {
    dp[u] = 0; // base case
    for (int v : adj[u]) if (v != p) { treeDp(v, u); dp[u] = max(dp[u], dp[v] + 1); }
}

// ===================== 0-1 BFS =====================
// see File 1, Part 3
```

---
**End of File 4.** Continue to File 5 for Common Mistakes, the Interview Thinking Process, the Practice Roadmap, and the full Cheat Sheet / Complexity Table / Visualization set.
