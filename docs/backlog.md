# Problem Backlog

## Disconnected Graphs

| # | Problem | Hint | Status |
|---|---------|------|--------|
| LC 130 | [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | DFS from borders inward | Done |
| LC 417 | [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | DFS from two borders | Done |

---

## Graph Coloring / Bipartite

| # | Problem | Hint | Status |
|---|---------|------|--------|
| LC 886 | [Possible Bipartition](https://leetcode.com/problems/possible-bipartition/) | Same 2-coloring BFS — build graph from dislikes list first | Backlog |
| LC 1042 | [Flower Planting With No Adjacent](https://leetcode.com/problems/flower-planting-with-no-adjacent/) | Graph coloring generalized to 4 colors | Backlog |

---

## Cycle Detection

| # | Problem | Hint | Status |
|---|---------|------|--------|

---

## Word Ladder / BFS on Word Graph

| # | Problem | Hint | Status |
|---|---------|------|--------|
| LC 127 | [Word Ladder](https://leetcode.com/problems/word-ladder/) | BFS level-order; generate 26×L candidates; count starts at 1 | Done |
| LC 126 | [Word Ladder II](https://leetcode.com/problems/word-ladder-ii/) | BFS to build parent map + DFS backtrack; level-safe removal | Done |

---

## Solved

| # | Problem | Pattern | Notes |
|---|---------|---------|-------|
| LC 200 | Number of Islands | Disconnected Graphs / DFS | Outer loop counts components, DFS sinks visited cells |
| LC 695 | Max Area of Island | Disconnected Graphs / DFS | Same as LC 200, dfs returns area instead of void |
| LC 684 | Redundant Connection | Cycle Detection / Union-Find | Process edges one by one; return edge where both endpoints share a root |
| LC 207 | Course Schedule | Cycle Detection / Kahn's | Build adj list + inDegree; cycle exists if processed count != numCourses |
| LC 210 | Course Schedule II | Topological Sort / Kahn's | Same as LC 207 but collect nodes into result list as they're dequeued; return empty array if cycle |
| LC 785 | Is Graph Bipartite? | Graph Coloring / BFS | 2-color BFS; loop all nodes for disconnected components; use `&&` not `||` in unvisited check |
| LC 127 | Word Ladder | BFS on Word Graph | Generate 26×L candidates; remove from HashSet to mark visited; count starts at 1; return count+1 on match |
| LC 126 | Word Ladder II | BFS + DFS Backtrack | BFS builds parent map; level-safe removal with levelVisited set; DFS reconstructs all paths in reverse |
