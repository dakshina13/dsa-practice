# Problem Backlog

## Disconnected Graphs

| # | Problem | Hint | Status |
|---|---------|------|--------|
| LC 130 | [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | DFS from borders inward | Done |
| LC 417 | [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | DFS from two borders | Done |

---

## Cycle Detection

| # | Problem | Hint | Status |
|---|---------|------|--------|

---

## Solved

| # | Problem | Pattern | Notes |
|---|---------|---------|-------|
| LC 200 | Number of Islands | Disconnected Graphs / DFS | Outer loop counts components, DFS sinks visited cells |
| LC 695 | Max Area of Island | Disconnected Graphs / DFS | Same as LC 200, dfs returns area instead of void |
| LC 684 | Redundant Connection | Cycle Detection / Union-Find | Process edges one by one; return edge where both endpoints share a root |
| LC 207 | Course Schedule | Cycle Detection / Kahn's | Build adj list + inDegree; cycle exists if processed count != numCourses |
| LC 210 | Course Schedule II | Topological Sort / Kahn's | Same as LC 207 but collect nodes into result list as they're dequeued; return empty array if cycle |
