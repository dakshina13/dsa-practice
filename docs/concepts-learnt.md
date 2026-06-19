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

## Problems Solved

### LC 200 - Number of Islands
**Pattern:** Disconnected Graphs / DFS
**Key Takeaway:** Outer loop counts islands (components). Each DFS sinks the entire island so it's never recounted.

### LC 695 - Max Area of Island
**Pattern:** Disconnected Graphs / DFS
**Key Takeaway:** Same structure as LC 200. Make `dfs` return `int` and accumulate area with `return 1 + dfs(...) + dfs(...) + ...`
