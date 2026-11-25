# Linear Optimization: LO_4 & LO_5


---

## LO_4: Hungarian Algorithm (Assignment Problem)

**Problem:** Given a bipartite graph with left nodes `L` and right nodes `R`, where each edge has a cost, find a one-to-one matching with minimum total cost.

### Input format (`Testcase4.csv`)
A CSV with no header and 3 rows. Each column is one edge:

| Row | Meaning |
|-----|---------|
| 1 | Left node (`u`) |
| 2 | Right node (`v`) |
| 3 | Edge weight (cost) |

### How it works
1. `load()` reads the edges, builds the sets `L` and `R`, and fills a square cost matrix. Missing edges are set to `inf`.
2. `run()` maintains dual variables `u` (left nodes) and `v` (right nodes):
   - `v` is initialized to the minimum of each column.
   - Left nodes are processed one at a time.
   - For each one, the algorithm looks for an augmenting path using slack values (`c[i][j] - u[i] - v[j]`).
   - If no tight edge is available, the duals are adjusted to create one.
   - The matching is then updated along the path found.
3. `dual()` prints the current dual variables.
4. `mfinal()` prints the final duals, the matching, and the total cost.

### Sample output
- Left nodes: `{0, 1, 2}`, right nodes: `{3, 4, 5}`
- Matching: `0-3 (1)`, `1-4 (2)`, `2-5 (3)`
- **Total cost = 6**

### Usage
Update the filename in `demo()` to run a different test case:
```python
algo = Hungarian()
if algo.load('Testcase4.csv'):
    mL, mR, u, v = algo.run()
```

---

## LO_5: Min Cost Max Flow (Primal-Dual)

**Problem:** In a flow network with edge capacities and per-unit costs, push the maximum possible flow from a source to a sink at minimum total cost.

### Input format (`Testcase9.csv`)
A CSV with no header and exactly 4 rows. Each column is one edge:

| Row | Meaning |
|-----|---------|
| 1 | From vertex |
| 2 | To vertex |
| 3 | Capacity |
| 4 | Cost per unit flow |

The **source is vertex 1** and the **sink is vertex 2** (fixed in `main()`).

### Class and function overview
| Name | Description |
|------|-------------|
| `MCMF` | Class that computes min cost max flow |
| `__init__` | Initializes the adjacency list, graph, and edge list |
| `AE(u, v, capacity, cost)` | Adds an edge along with its reverse residual edge (negated cost) |
| `DWP(src, sk, ptl)` | Dijkstra's shortest path using reduced costs (`cost + ptl[u] - ptl[v]`); returns distance, parent, and edge used |
| `mcmf(src, sk)` | Main loop; returns `(max_flow, min_cost)` |

### How it works
1. Start with all vertex potentials (dual variables) set to 0.
2. Run Dijkstra on the residual graph with reduced costs to find the cheapest augmenting path.
3. Update the potentials using the computed distances.
4. Push the bottleneck flow along the path and update the residual edges.
5. Repeat until the sink is unreachable.

### Sample output
| Iteration | Path | Flow | Path cost |
|-----------|------|------|-----------|
| 1 | 1 → 3 → 4 → 2 | 2 | 4 |
| 2 | 1 → 3 → 2 | 1 | 4 |
| 3 | 1 → 4 → 2 | 1 | 6 |
| 4 | 1 → 4 → 3 → 2 | 1 | 8 |

**Maximum flow = 5, minimum cost = 22**

---

## Notes
- The Hungarian algorithm is a special case of min cost flow, so LO_4 solves matching while LO_5 solves the general network problem.
- LO_5 assumes non-negative edge costs. Negative costs would require a Bellman-Ford initialization of the potentials.
- The CSV test files must be placed in the same directory as the notebooks (or uploaded in Colab).

## Requirements
- Python 3
- `numpy`, `pandas` (LO_4)
- Standard library only: `csv`, `heapq`, `collections` (LO_5)
