# Concepts Learnt

## Graph DFS

**Key Takeaway:** DFS explores as deep as possible down one path before backtracking. Always maintain a `visited` tracker to avoid infinite loops in cyclic graphs.

### Core Template (Java)
```java
void dfs(int[][] grid, int row, int col) {
    if (row < 0 || row >= grid.length || col < 0 || col >= grid[0].length || grid[row][col] == 0)
        return;

    grid[row][col] = 0; // mark visited by sinking

    dfs(grid, row - 1, col);
    dfs(grid, row + 1, col);
    dfs(grid, row, col - 1);
    dfs(grid, row, col + 1);
}
```

### Visited Tracking Options
| Approach | When to use |
|----------|-------------|
| Sink the cell (`grid[r][c] = 0`) | Grid problems — O(1) space, preferred in interviews |
| `Set<String> visited` with `"row,col"` key | When you can't modify the input |
| Avoid `int[]` as HashSet key | Java compares arrays by reference, not value |

### Disconnected Graphs
- A single DFS from one node only explores its connected component.
- To cover all components, wrap DFS in an outer loop over all nodes.
- Each fresh DFS start = one new disconnected component.

```java
for (int node : allNodes) {
    if (!visited.contains(node)) {
        dfs(graph, node, visited); // one call per component
    }
}
```

### Grid = Graph
- Grid cells are nodes; horizontal/vertical adjacency defines edges.
- **Diagonal movement is NOT allowed** unless the problem explicitly says so.
- No need to convert the grid to an adjacency list — traverse it directly.

### Complexity
| | |
|---|---|
| Time | O(V + E) → O(m × n) for grids |
| Space | O(V) → O(m × n) for recursion stack in worst case |

---

---

## Graph Representations

| Representation | Storage | Get neighbors | Edge lookup | Best for |
|---|---|---|---|---|
| Edge list | O(E) | O(E) scan | O(E) | Union-Find, Kruskal's |
| Adjacency list | O(V+E) | O(degree) | O(degree) | DFS, BFS, most traversals |
| Adjacency matrix | O(V²) | O(V) | O(1) | Dense graphs, frequent edge lookups |

**Key insight:** Adjacency matrix row `i` is the same information as `adj.get(i)` — just encoded as 0s and 1s across all V columns instead of a compact neighbor list. The tradeoff is O(V²) space regardless of edge count.

---

## Cycle Detection — Undirected Graphs

### Approach 1: DFS with Parent Tracking
A cycle exists if DFS visits a node that is **already visited AND is not the parent** of the current node.

```java
private static boolean dfs(List<List<Integer>> adj, boolean[] visited, int node, int parent) {
    visited[node] = true;
    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor]) {
            if (dfs(adj, visited, neighbor, node)) return true;
        } else if (neighbor != parent) {
            return true; // visited + not parent = back edge = cycle
        }
    }
    return false;
}
```

**Gotcha — multigraphs:** Two edges between the same pair of nodes triggers a false positive with parent node tracking. Fix: track the **parent edge ID** instead of the parent node, and skip only the exact edge you arrived on.

### Approach 2: Union-Find (DSU)
Process edges one by one. If both endpoints already share a root → adding this edge creates a cycle.

```java
boolean union(int x, int y) {
    int px = find(x), py = find(y);
    if (px == py) return false; // same component → cycle
    if (rank[px] < rank[py]) { int t = px; px = py; py = t; }
    parent[py] = px;
    if (rank[px] == rank[py]) rank[px]++;
    return true;
}
```

**Rank rule:** Only increment rank when merging two trees of equal rank — that's the only case where the resulting tree grows taller. Rank comparisons must use roots (`px`, `py`), never the original input nodes.

**When to use Union-Find over DFS:** When processing edges incrementally (online) or given an edge list.

---

## Cycle Detection — Directed Graphs

### Approach 1: DFS 3-Color
Track node state with three colors. A back edge (hitting a GRAY node) means a cycle.

| Color | Meaning |
|-------|---------|
| WHITE (0) | Not yet visited |
| GRAY (1) | Currently on the DFS call stack |
| BLACK (2) | Fully processed — entire subtree is cycle-free |

```java
private static boolean dfs(List<List<Integer>> adj, int[] color, int node) {
    color[node] = GRAY;
    for (int neighbor : adj.get(node)) {
        if (color[neighbor] == GRAY) return true;   // back edge → cycle
        if (color[neighbor] == WHITE) {
            if (dfs(adj, color, neighbor)) return true;
        }
        // BLACK → already fully explored, safe to skip
    }
    color[node] = BLACK;
    return false;
}
```

### Approach 2: Kahn's Algorithm (BFS / Topological Sort)
A directed graph has a cycle if and only if it has no valid topological ordering.

```java
// 1. Build adj list and inDegree array
// 2. Enqueue all nodes with inDegree == 0
// 3. Process: decrement neighbors' inDegree, enqueue if it hits 0
// 4. If processed != numNodes → cycle exists
return processed == numCourses;
```

**When to use Kahn's:** When you need the topological order anyway. DFS 3-color is more direct for pure cycle detection.

### Which approach for directed graphs?
| Scenario | Best approach |
|---|---|
| Pure cycle detection | DFS 3-color |
| Need topological order too | Kahn's |

---

## Topological Sort

**Key Takeaway:** An ordering of nodes in a DAG such that for every edge `u → v`, `u` appears before `v`. Not a comparison sort — it orders by *dependency*, not value. Only valid on DAGs (no cycles).

### DFS-based (post-order + stack)

Push a node onto the stack only **after all its descendants are fully explored**. Reading the stack top-to-bottom gives the topological order.

```java
private static void dfs(int node, List<List<Integer>> adj,
                         boolean[] visited, boolean[] inStack, Deque<Integer> stack) {
    visited[node] = true;
    inStack[node] = true;

    for (int neighbor : adj.get(node)) {
        if (!visited[neighbor])
            dfs(neighbor, adj, visited, inStack, stack);
        else if (inStack[neighbor])
            hasCycle = true; // back edge = cycle
    }

    inStack[node] = false;
    stack.push(node); // post-order: push after all descendants done
}
```

### `visited` vs `inStack` — critical distinction

Both are boolean arrays but they answer different questions:

| Array | Question it answers |
|-------|---------------------|
| `visited` | Has this node ever been seen? (prevents re-exploring) |
| `inStack` | Is this node currently on the active DFS call stack? (detects cycles) |

You can reach an already-visited node in two situations:
- **Cross/forward edge** — visited via a different path, already fully explored → `inStack[node] = false` → safe, no cycle
- **Back edge** — visited AND still open on the current path → `inStack[node] = true` → cycle

`inStack` is set to `true` on entry and back to `false` on exit from the recursive call. Only nodes in the *current recursion frame* have `inStack = true`.

### Kahn's Algorithm (BFS / in-degree)

```java
// 1. Build adj list and inDegree array
// 2. Enqueue all nodes with inDegree == 0
// 3. Poll node → add to result list → decrement neighbors' inDegree → enqueue if hits 0
// 4. If result.size() != numNodes → cycle exists, return []
```

**Why it works for topo order:** Nodes with in-degree 0 have no unresolved dependencies — they're safe to take first. Removing them may unlock new zero-in-degree nodes, and so on.

### When to use which

| Scenario | Approach |
|---|---|
| Need topological order | Either; Kahn's is iterative and easier to reason about |
| Pure cycle detection | DFS 3-color (no stack needed) |
| Order + cycle detection together | Kahn's (size check is the cycle signal) |

---

## Graph BFS

**Key Takeaway:** BFS explores level by level using a queue. Preferred over DFS when you need shortest path or minimum steps.

### Core Template (Java)
```java
Deque<int[]> queue = new ArrayDeque<>();
queue.offer(new int[]{startRow, startCol});
grid[startRow][startCol] = VISITED; // mark at enqueue, not dequeue

while (!queue.isEmpty()) {
    int[] curr = queue.poll();
    int row = curr[0], col = curr[1];

    // check 4 neighbors
    for each valid neighbor where grid[neighbor] == target:
        grid[neighbor] = VISITED; // mark before enqueuing
        queue.offer(neighbor);
}
```

### Mark at Enqueue, Not Dequeue
Always mark a cell as visited **when you enqueue it**, not when you dequeue it. If you mark at dequeue time, the same cell can be enqueued multiple times by different neighbors before it's ever processed.

### Level Order Traversal (counting steps/time)
To count BFS levels (e.g., minutes elapsed), snapshot the queue size before each level:

```java
int time = 0;
while (!queue.isEmpty()) {
    int size = queue.size(); // all nodes at the current level
    for (int i = 0; i < size; i++) {
        int[] curr = queue.poll();
        // process and enqueue neighbors
    }
    time++;
}
```
After the loop, `time` is over-counted by 1 — use `time - 1`, or `Math.max(0, time - 1)` to handle the case where the queue started empty.

### Multi-Source BFS
When rot/infection/spread starts from **multiple sources simultaneously**, seed all sources into the queue at time 0 before the loop begins. They all belong to level 0 and spread in unison.

```java
// enqueue ALL sources first
for (each source cell):
    queue.offer(source);

// then BFS normally — all sources are treated as level 0
```

### The `oc == color` Edge Case (Flood Fill style)
If you're filling with the same color that's already there, your "already visited" check (`cell != originalColor`) never fires — every cell looks unvisited forever. Always guard:

```java
if (image[sr][sc] == color) return image;
```

---

## Word Ladder / BFS on Word Graph

**Key Takeaway:** Treat each word as a node; two words are connected if they differ by exactly one letter. Shortest path → BFS. Never DFS for shortest path — DFS finds *a* path, not the *shortest* one.

### Candidate Generation — O(26 × L) not O(N × L)
Instead of comparing current word against every word in the list, generate all one-letter variants and check each against a `HashSet`:

```java
for (int i = 0; i < arr.length; i++) {
    char original = arr[i];
    for (char c = 'a'; c <= 'z'; c++) {
        if (c == original) continue;
        arr[i] = c;
        String candidate = new String(arr);
        if (wordSet.contains(candidate)) { /* process */ }
    }
    arr[i] = original; // restore
}
```

### Visited Tracking
Remove a word from the HashSet once it's added to the queue — this prevents revisiting and avoids cycles without a separate visited set.

### Level Tracking and Count Initialization
Use `queue.size()` snapshot for level-order BFS. Initialize `count = 1` (not 0) because `beginWord` is itself step 1 in the sequence. Return `count + 1` when `endWord` is found (current level hasn't been counted yet).

```java
int count = 1;
while (!queue.isEmpty()) {
    int size = queue.size();
    for (int j = 0; j < size; j++) {
        // process word
        if (neighbor.equals(endWord)) return count + 1;
    }
    count++;
}
return 0;
```

### Word Ladder II — All Shortest Paths

**Two-phase approach:**
1. **BFS phase** — build a `Map<String, List<String>> backtrace` mapping each word to all its shortest-path parents
2. **DFS phase** — backtrack from `endWord` to `beginWord` using the parent map, reversing each completed path

**Level-safe removal:** Within a BFS level, multiple words can reach the same neighbor. Use a `Set<String> levelVisited` to ensure each neighbor is enqueued only once per level (preventing duplicate backtrace entries), while still recording all parents. Remove `levelVisited` from the main word set only *after* the full level is processed.

```java
Set<String> levelVisited = new HashSet<>();
if (wl.contains(candidate)) {
    if (!levelVisited.contains(candidate)) {
        levelVisited.add(candidate);
        queue.offer(candidate);       // enqueue once
    }
    backtrace.computeIfAbsent(candidate, k -> new ArrayList<>()).add(parentStr); // all parents
}
// after inner loops:
wl.removeAll(levelVisited);
```

**DFS backtracking with path mutation:** Add `word` to path, recurse on its parents, then remove it. Crucially, the base case (`word.equals(beginWord)`) must also remove the word before returning — otherwise sibling branches inherit a corrupted path.

```java
void buildList(String word, String beginWord, List<String> path,
               List<List<String>> result, Map<String, List<String>> backtrace) {
    path.add(word);
    if (word.equals(beginWord)) {
        List<String> copy = new ArrayList<>(path);
        Collections.reverse(copy);
        result.add(copy);
        path.remove(path.size() - 1); // must remove in base case too
        return;
    }
    for (String parent : backtrace.getOrDefault(word, new ArrayList<>())) {
        buildList(parent, beginWord, path, result, backtrace);
    }
    path.remove(path.size() - 1); // use index, not value
}
```

---

## Problems Solved

### LC 684 - Redundant Connection
**Pattern:** Cycle Detection / Union-Find
**Key Takeaway:** Process edges one by one with Union-Find. The first edge where both endpoints share a root is the redundant edge. Use `edges.length + 1` for UnionFind size since nodes are 1-indexed.

### LC 207 - Course Schedule
**Pattern:** Cycle Detection / Kahn's Algorithm
**Key Takeaway:** Build adjacency list with `b → a` for each `[a,b]` prerequisite, track inDegree per node. If `processed != numCourses` after Kahn's, a cycle exists. Pre-initialize adjacency list as `List<List<Integer>>` to avoid null checks on nodes with no outgoing edges.

### LC 210 - Course Schedule II
**Pattern:** Topological Sort / Kahn's Algorithm
**Key Takeaway:** Same as LC 207 but collect each dequeued node into a result list — that list is the topological order. Return empty array if `result.size() != numCourses` (cycle). The `processed` counter is redundant; use `result.size()` directly.

### LC 200 - Number of Islands
**Pattern:** Disconnected Graphs / DFS
**Key Takeaway:** Outer loop counts islands (components). Each DFS sinks the entire island so it's never recounted.

### LC 695 - Max Area of Island
**Pattern:** Disconnected Graphs / DFS
**Key Takeaway:** Same structure as LC 200. Make `dfs` return `int` and accumulate area with `return 1 + dfs(...) + dfs(...) + ...`

### LC 130 - Surrounded Regions
**Pattern:** Disconnected Graphs / Border-first DFS
**Key Takeaway:** Reverse the search — don't find islands to flip, find what's safe first. DFS from every border `'O'`, mark connected cells `'A'`. Final sweep: `'O'` → `'X'` (captured), `'A'` → `'O'` (restore safe). Next time you see "ignore regions touching the boundary", think border-first DFS + temp marker.

### LC 417 - Pacific Atlantic Water Flow
**Pattern:** Disconnected Graphs / Border-first DFS (two oceans)
**Key Takeaway:** Reverse water flow direction — DFS uphill from each ocean's border (`neighbor height >= current height`). Run once from Pacific borders (top row + left col), once from Atlantic borders (bottom row + right col). Answer is the intersection of both visited sets. Use `boolean[][]` or `int[][]` for visited — never `HashSet<int[]>` in Java (array reference equality breaks it).

### LC 994 - Rotting Oranges
**Pattern:** Multi-Source BFS / Level Order Traversal
**Key Takeaway:** Seed all initially rotten cells into the queue at time 0. Use level-order BFS (snapshot `queue.size()` before each level) to count elapsed minutes. Mark cells rotten **at enqueue time** to prevent duplicate enqueues. Return `Math.max(0, time - 1)` — not `time - 1` — to handle the edge case where no fresh oranges exist and the queue starts empty (would return `-1` otherwise).

### LC 733 - Flood Fill
**Pattern:** BFS / Single-Source Spread
**Key Takeaway:** Standard BFS from the starting cell, spreading only to neighbors matching the original color. Two critical guards: (1) mark cells with the new color **at enqueue time** to prevent re-enqueuing; (2) early return `if (originalColor == newColor)` — without it, the "already visited" check never fires and you get an infinite loop.

### LC 785 - Is Graph Bipartite?
**Pattern:** Graph 2-Coloring / BFS
**Key Takeaway:** A graph is bipartite if and only if it can be 2-colored (no two adjacent nodes share a color). BFS from each unvisited node, assigning the opposite color to every neighbor. If a neighbor is already colored with the same color as the current node → not bipartite.

**Critical gotchas:**
- **Disconnected graphs:** Must loop over all nodes and start a fresh BFS from each unvisited one — a single BFS from node 0 misses separate components.
- **`&&` vs `||` in unvisited check:** `colorMap[i] != 'W' || colorMap[i] != 'B'` is **always true** (a char can't be both at once). The correct "not yet colored" check is `colorMap[i] != 'W' && colorMap[i] != 'B'`.
- **Array index vs neighbor node:** When iterating `for (int j = 0; j < adj.length; j++)`, use `adj[j]` (the actual neighbor node), not `j` (the index into adj).

```java
public boolean isBipartite(int[][] graph) {
    char[] colorMap = new char[graph.length];
    Deque<Integer> queue = new ArrayDeque<>();

    for (int i = 0; i < graph.length; i++) {
        if (colorMap[i] != 'W' && colorMap[i] != 'B') {  // unvisited
            colorMap[i] = 'W';
            queue.offer(i);
        }
        while (!queue.isEmpty()) {
            int n = queue.poll();
            char opp = colorMap[n] == 'W' ? 'B' : 'W';
            for (int a : graph[n]) {
                if (colorMap[a] != 'W' && colorMap[a] != 'B') {
                    colorMap[a] = opp;
                    queue.offer(a);
                } else if (colorMap[a] == colorMap[n]) {
                    return false;
                }
            }
        }
    }
    return true;
}
```

**Complexity:** Time O(V + E), Space O(V) for the color array and queue.
