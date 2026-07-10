# GRAPH KNOWLEDGE BASE — FILE 6
## Part 23 (Java Edition) — Complete Graph Template Library

All templates below use `ArrayList<Integer>[]` / `ArrayList<int[]>[]` adjacency lists (fast, primitive-friendly), and follow the same logic as File 4's C++ versions, adapted to Java idioms (no raw arrays of generics, `Deque` instead of `stack`, `PriorityQueue` with comparators, etc.)

```java
import java.util.*;

public class GraphTemplates {

    // ===================== Build adjacency list =====================
    @SuppressWarnings("unchecked")
    static List<Integer>[] buildAdj(int n, int[][] edges, boolean directed) {
        List<Integer>[] adj = new List[n];
        for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
        for (int[] e : edges) {
            adj[e[0]].add(e[1]);
            if (!directed) adj[e[1]].add(e[0]);
        }
        return adj;
    }

    // Weighted version: adj[u] = list of {v, weight}
    @SuppressWarnings("unchecked")
    static List<int[]>[] buildWeightedAdj(int n, int[][] edges, boolean directed) {
        List<int[]>[] adj = new List[n];
        for (int i = 0; i < n; i++) adj[i] = new ArrayList<>();
        for (int[] e : edges) { // e = {u, v, w}
            adj[e[0]].add(new int[]{e[1], e[2]});
            if (!directed) adj[e[1]].add(new int[]{e[0], e[2]});
        }
        return adj;
    }

    // ===================== DFS (Recursive) =====================
    static void dfs(int u, List<Integer>[] adj, boolean[] visited) {
        visited[u] = true;
        // process u here
        for (int v : adj[u]) if (!visited[v]) dfs(v, adj, visited);
    }
    // NOTE (Java-specific): for graphs with V > ~10,000-50,000, run this on a
    // dedicated thread with a bigger stack to avoid StackOverflowError:
    //   new Thread(null, () -> dfs(0, adj, visited), "dfs", 1 << 26).start();

    // ===================== DFS (Iterative) =====================
    static void dfsIterative(int src, List<Integer>[] adj, boolean[] visited) {
        Deque<Integer> stack = new ArrayDeque<>();
        stack.push(src);
        while (!stack.isEmpty()) {
            int u = stack.pop();
            if (visited[u]) continue;
            visited[u] = true;
            // process u here
            for (int v : adj[u]) if (!visited[v]) stack.push(v);
        }
    }

    // ===================== BFS =====================
    static int[] bfs(int src, int n, List<Integer>[] adj) {
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        Deque<Integer> queue = new ArrayDeque<>();
        dist[src] = 0;
        queue.add(src);
        while (!queue.isEmpty()) {
            int u = queue.poll();
            for (int v : adj[u]) {
                if (dist[v] == -1) {
                    dist[v] = dist[u] + 1;
                    queue.add(v);
                }
            }
        }
        return dist;
    }

    // ===================== Multi-source BFS =====================
    static int[] multiSourceBFS(List<Integer> sources, int n, List<Integer>[] adj) {
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        Deque<Integer> queue = new ArrayDeque<>();
        for (int s : sources) { dist[s] = 0; queue.add(s); }
        while (!queue.isEmpty()) {
            int u = queue.poll();
            for (int v : adj[u]) {
                if (dist[v] == -1) { dist[v] = dist[u] + 1; queue.add(v); }
            }
        }
        return dist;
    }

    // ===================== Grid BFS =====================
    static final int[] DX = {-1, 1, 0, 0};
    static final int[] DY = {0, 0, -1, 1};

    static int gridBFS(int[][] grid, int sr, int sc, int tr, int tc) {
        int R = grid.length, C = grid[0].length;
        int[][] dist = new int[R][C];
        for (int[] row : dist) Arrays.fill(row, -1);
        Deque<int[]> queue = new ArrayDeque<>();
        dist[sr][sc] = 0;
        queue.add(new int[]{sr, sc});
        while (!queue.isEmpty()) {
            int[] cur = queue.poll();
            int r = cur[0], c = cur[1];
            for (int d = 0; d < 4; d++) {
                int nr = r + DX[d], nc = c + DY[d];
                if (nr < 0 || nr >= R || nc < 0 || nc >= C) continue;
                if (grid[nr][nc] == 1) continue; // wall
                if (dist[nr][nc] != -1) continue;
                dist[nr][nc] = dist[r][c] + 1;
                queue.add(new int[]{nr, nc});
            }
        }
        return dist[tr][tc];
    }

    // ===================== Grid DFS (flood fill) =====================
    static void gridDFS(int[][] grid, int r, int c, boolean[][] visited) {
        int R = grid.length, C = grid[0].length;
        if (r < 0 || r >= R || c < 0 || c >= C) return;
        if (visited[r][c] || grid[r][c] == 0) return;
        visited[r][c] = true;
        for (int d = 0; d < 4; d++) gridDFS(grid, r + DX[d], c + DY[d], visited);
    }

    // ===================== 0-1 BFS =====================
    static int[] zeroOneBFS(int src, int n, List<int[]>[] adj /* {v, w in {0,1}} */) {
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        Deque<Integer> deque = new ArrayDeque<>();
        dist[src] = 0;
        deque.add(src);
        while (!deque.isEmpty()) {
            int u = deque.poll();
            for (int[] edge : adj[u]) {
                int v = edge[0], w = edge[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    if (w == 0) deque.addFirst(v);
                    else deque.addLast(v);
                }
            }
        }
        return dist;
    }

    // ===================== Dijkstra =====================
    static long[] dijkstra(int src, int n, List<int[]>[] adj /* {v, w} */) {
        long[] dist = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[src] = 0;
        // PQ entries: {distance, node} packed as long[] to avoid autoboxing overhead
        PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        pq.add(new long[]{0, src});
        while (!pq.isEmpty()) {
            long[] top = pq.poll();
            long d = top[0];
            int u = (int) top[1];
            if (d > dist[u]) continue; // stale entry
            for (int[] edge : adj[u]) {
                int v = edge[0], w = edge[1];
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.add(new long[]{dist[v], v});
                }
            }
        }
        return dist;
    }

    // ===================== Bellman-Ford =====================
    // edges[i] = {u, v, w}. Returns null if a negative cycle is detected.
    static long[] bellmanFord(int src, int n, int[][] edges) {
        long[] dist = new long[n];
        Arrays.fill(dist, Long.MAX_VALUE);
        dist[src] = 0;
        for (int i = 0; i < n - 1; i++) {
            for (int[] e : edges) {
                int u = e[0], v = e[1], w = e[2];
                if (dist[u] != Long.MAX_VALUE && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }
        for (int[] e : edges) {
            int u = e[0], v = e[1], w = e[2];
            if (dist[u] != Long.MAX_VALUE && dist[u] + w < dist[v]) {
                return null; // negative cycle
            }
        }
        return dist;
    }

    // ===================== Floyd-Warshall =====================
    static void floydWarshall(long[][] dist, int n) {
        // dist[i][j] pre-filled with edge weights, Long.MAX_VALUE/2 for "no edge" (avoid overflow on add)
        for (int k = 0; k < n; k++)
            for (int i = 0; i < n; i++)
                for (int j = 0; j < n; j++)
                    if (dist[i][k] + dist[k][j] < dist[i][j])
                        dist[i][j] = dist[i][k] + dist[k][j];
        // negative cycle check: any dist[i][i] < 0 after running
    }

    // ===================== DSU (Union-Find) =====================
    static class DSU {
        int[] parent, rank_;
        DSU(int n) {
            parent = new int[n];
            rank_ = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]); // path compression
            return parent[x];
        }
        boolean union(int x, int y) {
            int rx = find(x), ry = find(y);
            if (rx == ry) return false; // already connected -> would form a cycle
            if (rank_[rx] < rank_[ry]) { int tmp = rx; rx = ry; ry = tmp; }
            parent[ry] = rx;
            if (rank_[rx] == rank_[ry]) rank_[rx]++;
            return true;
        }
    }

    // ===================== Topological Sort (Kahn's) =====================
    static List<Integer> kahnTopoSort(int n, List<Integer>[] adj) {
        int[] indegree = new int[n];
        for (int u = 0; u < n; u++) for (int v : adj[u]) indegree[v]++;
        Deque<Integer> queue = new ArrayDeque<>();
        for (int i = 0; i < n; i++) if (indegree[i] == 0) queue.add(i);
        List<Integer> order = new ArrayList<>();
        while (!queue.isEmpty()) {
            int u = queue.poll();
            order.add(u);
            for (int v : adj[u]) if (--indegree[v] == 0) queue.add(v);
        }
        return order; // order.size() < n -> cycle exists
    }

    // ===================== Directed Cycle Detection (3-color DFS) =====================
    static boolean hasCycleDirected(int n, List<Integer>[] adj) {
        int[] state = new int[n]; // 0=white, 1=gray, 2=black
        for (int i = 0; i < n; i++) {
            if (state[i] == 0 && dfsCycleCheck(i, adj, state)) return true;
        }
        return false;
    }
    static boolean dfsCycleCheck(int u, List<Integer>[] adj, int[] state) {
        state[u] = 1;
        for (int v : adj[u]) {
            if (state[v] == 1) return true;
            if (state[v] == 0 && dfsCycleCheck(v, adj, state)) return true;
        }
        state[u] = 2;
        return false;
    }

    // ===================== Kosaraju's SCC =====================
    static List<List<Integer>> kosarajuSCC(int n, List<Integer>[] adj) {
        boolean[] visited = new boolean[n];
        Deque<Integer> finishStack = new ArrayDeque<>();
        for (int i = 0; i < n; i++) if (!visited[i]) fillOrder(i, adj, visited, finishStack);

        List<Integer>[] transpose = new List[n];
        for (int i = 0; i < n; i++) transpose[i] = new ArrayList<>();
        for (int u = 0; u < n; u++) for (int v : adj[u]) transpose[v].add(u);

        Arrays.fill(visited, false);
        List<List<Integer>> sccs = new ArrayList<>();
        while (!finishStack.isEmpty()) {
            int u = finishStack.pop();
            if (!visited[u]) {
                List<Integer> comp = new ArrayList<>();
                dfsCollect(u, transpose, visited, comp);
                sccs.add(comp);
            }
        }
        return sccs;
    }
    static void fillOrder(int u, List<Integer>[] adj, boolean[] visited, Deque<Integer> stack) {
        visited[u] = true;
        for (int v : adj[u]) if (!visited[v]) fillOrder(v, adj, visited, stack);
        stack.push(u);
    }
    static void dfsCollect(int u, List<Integer>[] adj, boolean[] visited, List<Integer> comp) {
        visited[u] = true;
        comp.add(u);
        for (int v : adj[u]) if (!visited[v]) dfsCollect(v, adj, visited, comp);
    }

    // ===================== Bridges (Tarjan low-link) =====================
    static int timer = 0;
    static void findBridges(int u, int parent, List<Integer>[] adj, int[] tin, int[] low,
                             boolean[] visited, List<int[]> bridges) {
        visited[u] = true;
        tin[u] = low[u] = timer++;
        for (int v : adj[u]) {
            if (v == parent) continue; // NOTE: for multigraphs, skip by edge-id instead
            if (visited[v]) {
                low[u] = Math.min(low[u], tin[v]);
            } else {
                findBridges(v, u, adj, tin, low, visited, bridges);
                low[u] = Math.min(low[u], low[v]);
                if (low[v] > tin[u]) bridges.add(new int[]{u, v});
            }
        }
    }

    // ===================== Articulation Points =====================
    static void findArticulationPoints(int u, int parent, List<Integer>[] adj, int[] tin, int[] low,
                                        boolean[] visited, boolean[] isAP) {
        visited[u] = true;
        tin[u] = low[u] = timer++;
        int children = 0;
        for (int v : adj[u]) {
            if (v == parent) continue;
            if (visited[v]) {
                low[u] = Math.min(low[u], tin[v]);
            } else {
                children++;
                findArticulationPoints(v, u, adj, tin, low, visited, isAP);
                low[u] = Math.min(low[u], low[v]);
                if (low[v] >= tin[u] && parent != -1) isAP[u] = true;
            }
        }
        if (parent == -1 && children > 1) isAP[u] = true;
    }

    // ===================== Kruskal's MST =====================
    static long kruskalMST(int n, int[][] edges /* {u, v, w} */) {
        Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));
        DSU dsu = new DSU(n);
        long total = 0;
        for (int[] e : edges) {
            if (dsu.union(e[0], e[1])) total += e[2];
        }
        return total;
    }

    // ===================== Prim's MST =====================
    static long primMST(int n, List<int[]>[] adj /* {v, w} */) {
        boolean[] inMST = new boolean[n];
        PriorityQueue<long[]> pq = new PriorityQueue<>((a, b) -> Long.compare(a[0], b[0]));
        pq.add(new long[]{0, 0}); // {weight, node}
        long total = 0;
        while (!pq.isEmpty()) {
            long[] top = pq.poll();
            int u = (int) top[1];
            if (inMST[u]) continue;
            inMST[u] = true;
            total += top[0];
            for (int[] edge : adj[u]) {
                int v = edge[0], w = edge[1];
                if (!inMST[v]) pq.add(new long[]{w, v});
            }
        }
        return total;
    }

    // ===================== LCA (Binary Lifting) =====================
    static class LCA {
        int LOG;
        int[][] up;
        int[] depth;
        List<Integer>[] adj;

        LCA(int n, List<Integer>[] adj, int root) {
            this.adj = adj;
            LOG = Math.max(1, (int) (Math.log(n) / Math.log(2)) + 1);
            up = new int[n][LOG];
            depth = new int[n];
            dfs(root, root);
        }

        void dfs(int u, int parent) {
            up[u][0] = parent;
            for (int k = 1; k < LOG; k++) up[u][k] = up[up[u][k - 1]][k - 1];
            for (int v : adj[u]) {
                if (v != parent) {
                    depth[v] = depth[u] + 1;
                    dfs(v, u);
                }
            }
        }

        int query(int u, int v) {
            if (depth[u] < depth[v]) { int tmp = u; u = v; v = tmp; }
            int diff = depth[u] - depth[v];
            for (int k = 0; k < LOG; k++) if (((diff >> k) & 1) == 1) u = up[u][k];
            if (u == v) return u;
            for (int k = LOG - 1; k >= 0; k--) {
                if (up[u][k] != up[v][k]) { u = up[u][k]; v = up[v][k]; }
            }
            return up[u][0];
        }
    }

    // ===================== Euler Tour (tin/tout) =====================
    static int eulerTimer = 0;
    static void eulerTour(int u, int parent, List<Integer>[] adj, int[] tin, int[] tout) {
        tin[u] = eulerTimer++;
        for (int v : adj[u]) if (v != parent) eulerTour(v, u, adj, tin, tout);
        tout[u] = eulerTimer++;
    }
    // isAncestor(u, v): tin[u] <= tin[v] && tout[v] <= tout[u]

    // ===================== Generic Tree DP =====================
    static int[] treeDp;
    static void computeTreeDp(int u, int parent, List<Integer>[] adj) {
        treeDp[u] = 0; // base case
        for (int v : adj[u]) {
            if (v != parent) {
                computeTreeDp(v, u, adj);
                treeDp[u] = Math.max(treeDp[u], treeDp[v] + 1);
            }
        }
    }

    // ===================== Bipartite Check (BFS) =====================
    static boolean isBipartite(int n, List<Integer>[] adj) {
        int[] color = new int[n];
        Arrays.fill(color, -1);
        for (int start = 0; start < n; start++) {
            if (color[start] != -1) continue;
            color[start] = 0;
            Deque<Integer> queue = new ArrayDeque<>();
            queue.add(start);
            while (!queue.isEmpty()) {
                int u = queue.poll();
                for (int v : adj[u]) {
                    if (color[v] == -1) {
                        color[v] = 1 - color[u];
                        queue.add(v);
                    } else if (color[v] == color[u]) {
                        return false;
                    }
                }
            }
        }
        return true;
    }
}
```

## Java-Specific Performance Notes (apply across all templates above)
- Prefer `ArrayDeque<Integer>` over `LinkedList<Integer>` for queue/stack use — it's faster (no node allocation per element) and avoids the `Stack` class's legacy synchronization overhead.
- Prefer `PriorityQueue<long[]>` (packing distance+node into a primitive array) over `PriorityQueue<int[]>` with boxed comparisons, or over storing custom objects, to reduce GC churn on large graphs (V, E ~ 10^5-10^6).
- For adjacency lists at competitive-programming scale, consider CSR (Compressed Sparse Row) format using raw `int[]` arrays instead of `ArrayList<Integer>[]` if you hit TLE — Java's ArrayList/boxing overhead is real at 10^6+ edges.
- For deep recursion (DFS on trees/graphs with >10^4-10^5 depth), either convert to iterative, or explicitly increase thread stack size as shown above — this is the single most common Java-specific WA/RE (`StackOverflowError`) on competitive judges.
- `Arrays.fill()` is fast, but if you're resetting large arrays across many test cases, consider a "visited version/timestamp" array instead of `Arrays.fill` every time, to avoid O(V) reset cost per query when only O(k) nodes were actually touched.
