# GRAPH KNOWLEDGE BASE — FILE 5
## Parts 24–30: Mistakes, Interview Thinking Process, Roadmap, Cheat Sheets, Complexity Table, Visualizations

---

# PART 24 — COMMON MISTAKES

| Mistake | Why it happens | Fix |
|---|---|---|
| Wrong visited timing | Marking visited on **pop** instead of **push** in BFS | Mark visited the moment you enqueue, not when you dequeue |
| Infinite recursion | Forgetting a base case, or not marking visited before recursing in DFS | Mark visited **before** the recursive call, not after |
| Queue duplicates | Same node pushed multiple times before being marked visited | Check-and-mark atomically at push time |
| Stack overflow (Java-specific) | Deep recursive DFS on graphs with V > ~10,000–50,000 — Java's default thread stack (~512KB–1MB) is smaller than you'd expect and overflows faster than C++ | Convert to iterative DFS with an explicit `Deque<Integer>` as a stack, OR run the recursive DFS on a new `Thread` with a larger stack size: `new Thread(null, this::dfs, "dfs", 1 << 26).start();` |
| Disconnected graphs | Only running DFS/BFS once, missing components not reachable from node 0 | Always loop `for (int i = 0; i < n; i++) if (!visited[i]) dfs(i);` |
| Indexing | Mixing 0-indexed and 1-indexed nodes (common when problems give 1-indexed input) | Decide one convention up front, convert immediately at input parsing |
| Directed vs Undirected confusion | Adding only one direction when the graph is meant to be undirected (or vice versa) | Re-read the problem statement's edge semantics before writing `addEdge` |
| Weighted vs Unweighted confusion | Running BFS on a weighted graph (wrong shortest path) or Dijkstra on unweighted (unnecessary overhead, but not wrong — just wasteful) | Match the algorithm to whether weights matter |
| Integer overflow | Summing many edge weights or large `dist` values into an `int` | Use `long` for distances/costs in Java once values can exceed ~2×10⁹ |
| Memory limit (Java-specific) | `boolean[V][V]` adjacency matrices or `HashMap`-based adjacency lists for large sparse graphs waste huge memory in Java vs primitive arrays | Prefer `ArrayList<Integer>[] adj` (array of ArrayLists) or CSR-style int arrays over `HashMap<Integer, List<Integer>>` for performance-critical graphs |
| Autoboxing overhead (Java-specific) | Using `PriorityQueue<Integer>`/`PriorityQueue<int[]>` heavily in Dijkstra causes GC pressure on large inputs | Use `PriorityQueue<long[]>` packing `{dist, node}`, or encode both into a single `long` (`dist * 100000L + node`) to avoid object overhead in tight loops |
| Forgetting `Arrays.fill` / re-initializing shared state | Reusing `visited[]`/`dist[]` arrays across multiple test cases or multiple BFS calls without resetting | Reset explicitly before each independent traversal, or use a "visited timestamp" trick instead of `Arrays.fill` every time for performance |

---

# PART 25 — INTERVIEW THINKING PROCESS

For **every** graph problem, walk through these 7 questions out loud (this is literally how FAANG interviewers grade you):

## 1. What clues identify the algorithm?
Scan the problem for: graph size constraints (`V, E ≤ ?`), whether weights exist, whether weights can be negative, whether it's a grid, whether "shortest/minimum" appears, whether dependencies/ordering are implied. Map to Part 19's decision tree.

## 2. Why not another algorithm?
Explicitly rule out alternatives: *"I won't use BFS here because edges are weighted — BFS only guarantees shortest path when all edges cost the same."* This demonstrates depth, not just pattern matching.

## 3. Complexity analysis
State both time and space **before** coding: *"With V up to 10^5 and weighted edges, Dijkstra with a binary heap gives O((V+E) log V), which fits comfortably."*

## 4. Dry run
Walk a small example by hand (3-5 nodes) before or after coding — catches off-by-one errors and wrong assumptions about traversal order.

## 5. Optimization
Ask: can this be done with less memory (e.g., adjacency list vs matrix)? Can early termination help (stop Dijkstra once target is popped)? Can the state space be reduced (dedupe equivalent states)?

## 6. Edge cases
- Empty graph / single node.
- Disconnected graph.
- Self-loops and parallel edges.
- Negative weights when not expected.
- Cycles when a DAG was assumed.
- Source == target.
- Grid with all walls / all open.

## 7. Follow-up questions interviewers may ask
- "What if the graph were directed instead?"
- "What if edge weights could be negative?"
- "Can you do this with O(V) extra space instead of O(V+E)?"
- "What if you had to answer this query for every possible source node?" (→ signals all-pairs algorithms or rerooting DP)
- "What if the graph updates dynamically (edges added/removed) between queries?" (→ signals DSU limitations, offline processing, or advanced dynamic connectivity structures)

---

# PART 26 — PRACTICE ROADMAP

## Beginner (Weeks 1–3)
- Master representations: adjacency list/matrix, directed/undirected/weighted.
- DFS & BFS from scratch, iterative and recursive, in Java.
- Connected components, simple grid flood fill.
- **Problems**: Number of Islands, Flood Fill, Find if Path Exists in Graph, Clone Graph.

## Intermediate (Weeks 4–7)
- Cycle detection (all 4 methods), Topological Sort (both versions), DSU with path compression + union by rank.
- Dijkstra, Bellman-Ford, Floyd-Warshall — know when to use each.
- Bipartite check.
- **Problems**: Course Schedule I/II, Network Delay Time, Redundant Connection, Is Graph Bipartite, Rotting Oranges, 01 Matrix.

## Advanced (Weeks 8–11)
- MST (Kruskal + Prim), Bridges & Articulation Points, Kosaraju/Tarjan SCC.
- LCA (binary lifting), Tree DP, diameter/centroid.
- State-space BFS, 0-1 BFS.
- **Problems**: Critical Connections in a Network, Min Cost to Connect All Points, Word Ladder, Shortest Path to Get All Keys, Swim in Rising Water.

## Expert (Weeks 12–15)
- Max Flow (Edmonds-Karp → Dinic's), bipartite matching.
- Bitmask DP on graphs (TSP, Hamiltonian path), Euler paths/circuits.
- Heavy-Light Decomposition, Centroid Decomposition (CP-focused, less common in interviews).
- **Problems**: CSES Graph Series (full section), Codeforces Div 2 D/E graph problems, Reconstruct Itinerary (Euler path).

## Master (Ongoing)
- Timed CP contests (Codeforces, AtCoder) focusing on graph rounds.
- Re-derive every algorithm from scratch without references, under time pressure.
- Teach the concepts to someone else (or write your own notes, which you're already doing — the 5-file structure you asked for is exactly this).

## Resource Map
| Resource | Use For |
|---|---|
| Blind 75 | Core interview coverage, minimum viable prep |
| Neetcode 150 | Broader interview coverage with pattern grouping |
| Top Interview 150 (LeetCode curated) | FAANG-weighted practice |
| CSES Problem Set — Graph Algorithms section | The best structured graph-only practice set, ~30 problems covering nearly every algorithm in this knowledge base |
| Codeforces (search by tag: `graphs`, `dsu`, `shortest-paths`, `dfs-and-similar`) | Timed, competitive-level graph problems |
| AtCoder (Educational DP Contest has graph-adjacent DP; ABC/ARC graph tags) | Clean, well-tested problem statements |
| USACO Guide (Silver → Gold → Platinum, Graph sections) | Excellent explanations, especially for MST/SCC/flow |
| Books: *Competitive Programmer's Handbook* (Antti Laaksonen, free PDF), *CLRS* Ch 22-26 (rigorous proofs) | Theory depth and proofs |

---

# PART 27 — CHEAT SHEET (One Page)

**Traversal**: DFS = go deep (stack/recursion), explore everything. BFS = go wide (queue), shortest unweighted path.

**Weighted shortest path**: no negatives → Dijkstra. negatives, single source → Bellman-Ford. all pairs → Floyd-Warshall (dense/small V) or Johnson (sparse).

**Connectivity**: components → DFS/BFS or DSU. Cycle (undirected) → DSU or parent-tracked DFS. Cycle (directed) → 3-color DFS or Kahn's.

**Ordering**: dependencies → Topological Sort (Kahn's for cycle-safety, DFS for simplicity).

**Minimum connect-all-cost**: MST → Kruskal (sparse/edge-list) or Prim (dense/adjacency-list).

**Critical structure**: single edge failure → Bridges. single vertex failure → Articulation Points. mutual reachability groups → SCC (Kosaraju/Tarjan).

**Tree-specific**: rooted aggregate → Tree DP. all-roots aggregate → Rerooting DP. ancestor queries → Binary Lifting LCA. longest path → 2×BFS/DFS diameter trick.

**Grid**: implicit graph, use `dx[]/dy[]`, generate neighbors inline, no explicit adjacency list needed.

**Extra state needed** (keys, fuel, parity, direction): expand into a state graph, multi-dimensional visited array.

**Matching/Flow**: assignment problems → Bipartite Matching (flow-based). general max flow → Dinic's.

---

# PART 28 — COMPLEXITY TABLE (Master Reference)

| Algorithm | Time | Space | Directed? | Weighted? | Neg. Edge? | Neg. Cycle Detect? | Works on Tree? | Works on Grid? | Works on DAG? |
|---|---|---|---|---|---|---|---|---|---|
| DFS | O(V+E) | O(V) | Both | N/A | N/A | N/A | Yes | Yes | Yes |
| BFS | O(V+E) | O(V) | Both | N/A | N/A | N/A | Yes | Yes | Yes |
| 0-1 BFS | O(V+E) | O(V) | Both | 0/1 only | No | N/A | Yes | Yes | Yes |
| Dijkstra | O((V+E)logV) | O(V+E) | Both | Yes | No | No | Yes | Yes | Yes |
| Bellman-Ford | O(V·E) | O(V) | Both | Yes | Yes | Yes | Yes | Yes | Yes |
| Floyd-Warshall | O(V³) | O(V²) | Both | Yes | Yes | Yes (diag) | Yes (overkill) | Yes (overkill) | Yes |
| Johnson | O(V·E logV) | O(V²) output | Both | Yes | Yes | Yes | N/A | N/A | Yes |
| Kruskal (MST) | O(E logE) | O(V+E) | Undirected | Yes | Yes | N/A | Trivial (already tree) | Yes | N/A |
| Prim (MST) | O(E logV) | O(V+E) | Undirected | Yes | Yes | N/A | Trivial | Yes | N/A |
| DSU (per op) | O(α(V)) amortized | O(V) | Undirected | N/A | N/A | N/A | Yes | Yes | N/A |
| Kahn's Topo Sort | O(V+E) | O(V) | Directed | N/A | N/A | Detects cycle | N/A (trees have trivial order) | N/A | Yes |
| Kosaraju SCC | O(V+E) | O(V+E) | Directed | N/A | N/A | N/A | N/A | N/A | Collapses to itself |
| Tarjan SCC/Bridges/AP | O(V+E) | O(V) | Both | N/A | N/A | N/A | Trivial (no bridges in undirected sense typically all edges are bridges) | Yes | Yes |
| Binary Lifting LCA | O(V logV) build, O(logV) query | O(V logV) | N/A (tree) | N/A | N/A | N/A | Yes (tree-only) | No | N/A |
| Ford-Fulkerson | O(E · maxFlow) | O(V+E) | Directed | Yes (capacity) | N/A | N/A | N/A | Rare | N/A |
| Edmonds-Karp | O(V·E²) | O(V+E) | Directed | Yes | N/A | N/A | N/A | Rare | N/A |
| Dinic's | O(V²E), O(E√V) bipartite | O(V+E) | Directed | Yes | N/A | N/A | N/A | Rare | N/A |
| Bitmask DP (TSP/Hamiltonian) | O(2^V · V²) | O(2^V · V) | Both | Yes | N/A | N/A | Rare (overkill) | Rare | N/A |

---

# PART 29 — VISUALIZATIONS

## BFS Queue Evolution (graph: 1-2, 1-3, 2-4, 3-4)
```
Start: queue=[1], dist=[0,-,-,-]
Pop 1 → push 2,3 → queue=[2,3], dist=[0,1,1,-]
Pop 2 → push 4    → queue=[3,4], dist=[0,1,1,2]
Pop 3 → 4 already visited → queue=[4]
Pop 4 → queue=[]
Final dist: node1=0, node2=1, node3=1, node4=2
```

## DFS Recursive Stack Evolution (same graph, start at 1)
```
call dfs(1)              stack: [1]
 └─ call dfs(2)           stack: [1,2]
     └─ call dfs(4)       stack: [1,2,4]
         └─ (4's other neighbor 3, not visited) call dfs(3)  stack:[1,2,4,3]
             └─ no unvisited neighbors, return    stack:[1,2,4]
         └─ return                                stack:[1,2]
     └─ return                                    stack:[1]
 └─ (1's neighbor 3 already visited) skip
 └─ return                                         stack:[]
Order visited: 1, 2, 4, 3
```

## tin/tout (Euler Tour) Example
```
Tree:        1
            / \
           2   3
           |
           4

DFS from 1:
tin[1]=0 → tin[2]=1 → tin[4]=2 → tout[4]=3 → tout[2]=4 → tin[3]=5 → tout[3]=6 → tout[1]=7

Check "is 4 in subtree of 2?": tin[2]=1 <= tin[4]=2 <= tout[2]=4 → YES
Check "is 3 in subtree of 2?": tin[2]=1 <= tin[3]=5 <= tout[2]=4 → 5 > 4 → NO
```

## Dijkstra Distance Array Evolution
```
Graph: 1-2 (w=4), 1-3 (w=1), 3-2 (w=1), 2-4 (w=1)
Start: dist=[0,∞,∞,∞], pq=[(0,1)]
Pop(0,1): relax 2→4, 3→1. dist=[0,4,1,∞], pq=[(1,3),(4,2)]
Pop(1,3): relax 2 via 3: 1+1=2 < 4. dist=[0,2,1,∞], pq=[(2,2),(4,2)]
Pop(2,2): relax 4: 2+1=3. dist=[0,2,1,3], pq=[(3,4),(4,2)]
Pop(3,4): done.
Pop(4,2): stale (dist[2]=2 already < 4) → skip.
Final: dist = [0,2,1,3]
```

## Visited Array in Grid Flood Fill (3x3 grid, land=1, start at (0,0))
```
Grid:            Visited after DFS from (0,0):
1 1 0             T T F
1 0 0             T F F
0 0 1             F F F   <- (2,2) is a separate island, untouched
```

---
# PART 30 — FINAL GOAL / HOW TO USE THIS KNOWLEDGE BASE

You now have 5 files covering all 30 parts:
1. **File 1**: Fundamentals, Representation, Traversals (Parts 1–5)
2. **File 2**: Cycle Detection, Components, Topo Sort, Shortest Paths, MST, DSU (Parts 6–11)
3. **File 3**: Bipartite, Advanced Graphs, Tree-as-Graph, Grid, Graph DP, Greedy, Flow (Parts 12–18)
4. **File 4**: Pattern Recognition, Tricks, Variations, LeetCode Patterns, Templates (Parts 19–23)
5. **File 5** (this file): Mistakes, Interview Process, Roadmap, Cheat Sheets, Complexity Table, Visualizations (Parts 24–30)

**Suggested study loop for each algorithm**: read intuition (File 1-3) → dry run by hand (Part 29 style) → implement from the Java template library → solve 3-5 LeetCode problems from Part 22's grouping → re-derive the algorithm from memory a week later without looking.

By working through this systematically — intuition before code, dry run before optimization — you'll be equipped to independently solve the large majority of graph problems across LeetCode, Codeforces, AtCoder, HackerRank OAs, and FAANG interview loops.
