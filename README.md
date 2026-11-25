# Linear Optimization

# Simplex Algorithm: LO_1, LO_2 & LO_3

These three notebooks implement the **simplex method in its geometric (vertex-walking) form** for the linear program:

```
maximize    cᵀx
subject to  Ax ≤ b
```

At each vertex, the algorithm takes the `n` tight constraints `A_B`, computes the edge directions `V = -A_B⁻¹`, and moves along a direction that improves the objective until it hits a new constraint. The three notebooks differ in **how much information is given in the input**:

| Notebook | Input given | What the code must figure out |
|----------|-------------|-------------------------------|
| LO_1 | Initial feasible point, `c`, `A`, `b` | Move the point to a vertex, then run simplex |
| LO_2 | Same CSV as LO_1, but the point is ignored | Find a feasible start (Phase I), then run simplex |
| LO_3 | Only `c`, `A`, `b` (general case) | Find a feasible start (Phase I), then run simplex |

---

## LO_1: Simplex from a Given Feasible Point

**Use case:** An initial feasible point `z` is supplied, but it may be in the interior of the feasible region rather than at a vertex.

### Input format (`testcase_1.csv`)
CSV with no header. The last column is a separator/RHS column.

| Row | Meaning |
|-----|---------|
| 1 | Initial feasible point `z` |
| 2 | Cost vector `c` |
| 3 onwards | Constraint rows: coefficients of `A`, followed by the entry of `b` in the last column |

### How it works
1. **`findvertex(A, b, z)`** moves `z` to a vertex:
   - Finds the constraints that are already tight at `z`.
   - While fewer than `n` independent constraints are tight, it picks the nearest non-tight constraint, moves toward it, and records the newly tight constraint.
2. **Simplex iterations:**
   - Build `A0` from the tight rows and compute `V = -A0⁻¹`.
   - For each direction `v` with `c·v > 0`, run a ratio test to find the step length and the constraint that becomes tight.
   - Move to the best improving vertex and swap the leaving row in `A0` for the entering constraint.
   - Stop when no improving direction exists.
3. The notebook prints the sequence of vertices visited, the optimal vertex, and the optimal cost.

### Sample output
- `z = (0, 0)`, `c = (5, 2)`
- `findvertex` moves to the vertex **(1, 1)** with tight rows `[0, 1]`
- No improving direction exists, so it is already optimal
- **Optimal vertex = (1, 1), optimal cost = 7**

---

## LO_2: Simplex with Phase I (Initial Point Not Used)

**Use case:** The CSV still contains the initial point row, but the code passes `x0=None`, so the algorithm must find its own feasible start.

### Input format (`testcase_4.csv`)
Same as LO_1 (row 1 is `z`, which is read but not used).

### Structure
| Function | Description |
|----------|-------------|
| `simplexp1(A, b)` | **Phase I:** finds a feasible starting point `x0` |
| `simplexp2(A, b, c, x0)` | **Phase II:** runs the vertex-walking simplex and returns `(x, objective)` |
| `simplex(A, b, c, x0=None)` | Wrapper: runs Phase I if `x0` is missing, then Phase II |

### How it works
1. **Phase I (`simplexp1`):**
   - Flips any constraint with `b[i] < 0`.
   - Takes the least-squares solution of `Ax = b` and clips it to be non-negative.
   - If it is still infeasible, applies up to 50 corrective steps.
   - Reports failure if no feasible point is found.
2. **Phase II (`simplexp2`):**
   - Identifies the tight rows at `x0` and fills up to `n` rows using the smallest slacks.
   - Uses the pseudo-inverse if the basis matrix is singular (degeneracy).
   - Tracks visited bases to stop cycling.
   - Returns `inf` if an improving direction has no blocking constraint (unbounded problem).

### Sample output
- `c = (3, 9)`, with 4 constraints
- Phase I finds `x0 = (0, 1.7778)`
- Moves to vertex `(0.8889, 2.2222)` with objective `22.67`
- The next iteration finds an unbounded improving direction
- **Result: problem is unbounded (objective = ∞)**

---

## LO_3: Simplex, General Case

**Use case:** The general LP. No initial point is provided, only the cost vector and the constraints.

### Input format (`testcase_7.csv`)
CSV with no header.

| Row | Meaning |
|-----|---------|
| 1 | Cost vector `c` (last column is a placeholder) |
| 2 onwards | Constraint rows: coefficients of `A`, followed by the entry of `b` in the last column |

### How it works
The code is the same as LO_2 (`simplexp1`, `simplexp2`, `simplex`). The only difference is the input reader, which has no initial-point row:

```python
c = data[0, :-1]
A = data[1:, :-1]
b = data[1:, -1]
xopt, val = simplex(A, b, c, x0=None)
```

### Sample output
- `c = (5, 2)`, constraints: `-y ≤ 1`, `-x ≤ 1`, `-x - y ≤ 1`
- Phase I finds `x0 = (0, 0)`
- The first iteration finds an unbounded improving direction
- **Result: problem is unbounded (objective = ∞)**

---

## Notes
- All three assume the form **maximize `cᵀx` subject to `Ax ≤ b`**.
- Phase I in LO_2 and LO_3 is a heuristic (least squares plus corrective steps) rather than a full two-phase simplex. It assumes a non-negative feasible point and may report "infeasible" on some feasible problems.
- `simplexp1` modifies `A` and `b` in place when flipping negative-`b` constraints.
- The `Unbounded` case returns the last point visited together with `inf`, so the printed "Optimal solution" is not a true optimum in that case.
- CSV test files must be placed next to the notebooks (or uploaded in Colab).

## Requirements
- Python 3
- `numpy`, `pandas`
---

## LO_4 and LO-5 : classic network optimization algorithms using the primal-dual approach

## LO_4: Hungarian Algorithm 

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
