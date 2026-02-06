# 3.5 Graph Algorithms & Advanced Topics

## Graph Representation

### Adjacency Matrix
- **Structure**
  - 2D array: `matrix[i][j] = 1` if edge exists
  
- **Advantages**
  - O(1) edge lookup
  - Simple implementation
  
- **Disadvantages**
  - O(V²) space (V = vertices)
  - Wasteful for sparse graphs
  
- **Best For**
  - Dense graphs
  - Frequent edge queries

### Adjacency List
- **Structure**
  - Array of lists: `adj[u]` contains neighbors of u
  
- **Advantages**
  - O(V + E) space (E = edges)
  - Efficient for sparse graphs
  - Fast neighbor iteration
  
- **Disadvantages**
  - O(degree) edge lookup
  
- **Best For**
  - Sparse graphs (most real-world graphs)
  - BFS/DFS traversal

## Graph Traversal

### DFS (Depth-First Search)

#### Implementation
```c
void dfs(int v, bool visited[], Graph* g) {
    visited[v] = true;
    visit(v);
    
    for (int u : g->adj[v]) {
        if (!visited[u])
            dfs(u, visited, g);
    }
}
```

#### Iterative (Stack)
```c
void dfs_iterative(int start, Graph* g) {
    Stack s;
    bool visited[MAX_V] = {false};
    
    push(&s, start);
    while (!isEmpty(&s)) {
        int v = pop(&s);
        if (visited[v]) continue;
        visited[v] = true;
        visit(v);
        
        for (int u : g->adj[v])
            if (!visited[u])
                push(&s, u);
    }
}
```

#### Use Cases
- Maze solving
- Topological sorting
- Cycle detection
- Connected components
- Path finding

### BFS (Breadth-First Search)

#### Implementation
```c
void bfs(int start, Graph* g) {
    Queue q;
    bool visited[MAX_V] = {false};
    
    enqueue(&q, start);
    visited[start] = true;
    
    while (!isEmpty(&q)) {
        int v = dequeue(&q);
        visit(v);
        
        for (int u : g->adj[v]) {
            if (!visited[u]) {
                visited[u] = true;
                enqueue(&q, u);
            }
        }
    }
}
```

#### Use Cases
- Shortest path in unweighted graph
- Level-order traversal
- Connected components
- Bipartite checking

#### BFS for Shortest Path
- **Unweighted Graph**
  - BFS finds shortest path
  - Distance = number of edges
  
- **Weighted Graph**
  - BFS not optimal
  - Use Dijkstra or Bellman-Ford

## Shortest Path Algorithms

### Dijkstra's Algorithm

#### Principle
- **Greedy approach**
  - Always select unvisited vertex with minimum distance
  - Update neighbors' distances
  - Mark as visited
  
#### Implementation
```c
void dijkstra(Graph* g, int src, int dist[]) {
    PriorityQueue pq;  // Min heap
    bool visited[MAX_V] = {false};
    
    for (int i = 0; i < g->V; i++)
        dist[i] = INF;
    dist[src] = 0;
    
    push(&pq, src, 0);
    
    while (!isEmpty(&pq)) {
        int u = extractMin(&pq);
        if (visited[u]) continue;
        visited[u] = true;
        
        for (Edge e : g->adj[u]) {
            int v = e.to, weight = e.weight;
            if (dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
                push(&pq, v, dist[v]);
            }
        }
    }
}
```

#### Complexity
- **With Binary Heap**: O((V + E) log V)
- **With Fibonacci Heap**: O(E + V log V)

#### Limitation
- **Cannot handle negative edges**
- Greedy assumption breaks

### Bellman-Ford Algorithm

#### Principle
- **Dynamic programming**
  - Relax all edges V-1 times
  - Detect negative cycles
  
#### Algorithm
```c
bool bellman_ford(Graph* g, int src, int dist[]) {
    for (int i = 0; i < g->V; i++)
        dist[i] = INF;
    dist[src] = 0;
    
    // Relax all edges V-1 times
    for (int i = 0; i < g->V - 1; i++) {
        for (Edge e : g->edges) {
            if (dist[e.from] != INF && 
                dist[e.from] + e.weight < dist[e.to]) {
                dist[e.to] = dist[e.from] + e.weight;
            }
        }
    }
    
    // Check for negative cycle
    for (Edge e : g->edges) {
        if (dist[e.from] != INF && 
            dist[e.from] + e.weight < dist[e.to]) {
            return false;  // Negative cycle exists
        }
    }
    return true;
}
```

#### Complexity
- **Time**: O(VE)
- **Slower than Dijkstra**

#### Advantages
- Handles negative edges
- Detects negative cycles

### Floyd-Warshall Algorithm

#### Purpose
- **All-pairs shortest paths**
- Find shortest path between every pair of vertices

#### Algorithm
```c
void floyd_warshall(int graph[V][V], int dist[V][V]) {
    // Initialize
    for (int i = 0; i < V; i++)
        for (int j = 0; j < V; j++)
            dist[i][j] = graph[i][j];
    
    // Try all intermediate vertices
    for (int k = 0; k < V; k++)
        for (int i = 0; i < V; i++)
            for (int j = 0; j < V; j++)
                if (dist[i][k] + dist[k][j] < dist[i][j])
                    dist[i][j] = dist[i][k] + dist[k][j];
}
```

#### Complexity
- **Time**: O(V³)
- **Space**: O(V²)

#### Use When
- Need all-pairs distances
- Dense graph
- Small graph

## Minimum Spanning Tree (MST)

### Definition
- **Spanning Tree**
  - Connects all vertices
  - No cycles
  - V vertices → V-1 edges
  
- **Minimum**
  - Sum of edge weights is minimum

### Kruskal's Algorithm

#### Approach
- **Edge-centric, greedy**
  1. Sort edges by weight
  2. Add edge if doesn't create cycle
  3. Use Union-Find for cycle detection

#### Algorithm
```c
void kruskal(Graph* g, Edge mst[]) {
    sort_edges_by_weight(g->edges);
    UnionFind uf;
    init(&uf, g->V);
    
    int count = 0;
    for (Edge e : g->edges) {
        if (find(&uf, e.from) != find(&uf, e.to)) {
            mst[count++] = e;
            union(&uf, e.from, e.to);
            if (count == g->V - 1) break;
        }
    }
}
```

#### Complexity
- **Time**: O(E log E) = O(E log V)
- Dominated by sorting

#### Best For
- Sparse graphs

### Prim's Algorithm

#### Approach
- **Vertex-centric, greedy**
  1. Start from any vertex
  2. Add minimum edge connecting tree to non-tree vertex
  3. Repeat until all vertices included

#### Complexity
- **With Binary Heap**: O(E log V)
- **With Fibonacci Heap**: O(E + V log V)

#### Best For
- Dense graphs

## Topological Sort

### Definition
- **Linear ordering of vertices**
- For directed edge u → v, u comes before v
- Only possible for DAG (Directed Acyclic Graph)

### Applications
- Task scheduling with dependencies
- Build systems (Makefile)
- Course prerequisites

### Algorithm (Kahn's Algorithm)
```c
void topological_sort(Graph* g, int result[]) {
    int indegree[MAX_V] = {0};
    Queue q;
    
    // Calculate in-degrees
    for (int u = 0; u < g->V; u++)
        for (int v : g->adj[u])
            indegree[v]++;
    
    // Enqueue vertices with 0 in-degree
    for (int i = 0; i < g->V; i++)
        if (indegree[i] == 0)
            enqueue(&q, i);
    
    int index = 0;
    while (!isEmpty(&q)) {
        int u = dequeue(&q);
        result[index++] = u;
        
        for (int v : g->adj[u]) {
            indegree[v]--;
            if (indegree[v] == 0)
                enqueue(&q, v);
        }
    }
    
    // If index != V, graph has cycle
}
```

### DFS-based Approach
- Post-order DFS
- Reverse the order

## Union-Find (Disjoint Set)

### Operations
- **Find**: Which set does element belong to?
- **Union**: Merge two sets

### Implementation with Optimizations
```c
typedef struct {
    int parent[MAX_V];
    int rank[MAX_V];
} UnionFind;

void init(UnionFind* uf, int n) {
    for (int i = 0; i < n; i++) {
        uf->parent[i] = i;
        uf->rank[i] = 0;
    }
}

// Path compression
int find(UnionFind* uf, int x) {
    if (uf->parent[x] != x)
        uf->parent[x] = find(uf, uf->parent[x]);
    return uf->parent[x];
}

// Union by rank
void union_sets(UnionFind* uf, int x, int y) {
    int xroot = find(uf, x);
    int yroot = find(uf, y);
    
    if (xroot == yroot) return;
    
    if (uf->rank[xroot] < uf->rank[yroot])
        uf->parent[xroot] = yroot;
    else if (uf->rank[xroot] > uf->rank[yroot])
        uf->parent[yroot] = xroot;
    else {
        uf->parent[yroot] = xroot;
        uf->rank[xroot]++;
    }
}
```

### Complexity
- **Nearly O(1)** with both optimizations
- Amortized O(α(n)) where α = inverse Ackermann

### Use Cases
- Cycle detection in undirected graph
- Kruskal's MST algorithm
- Connected components

## Dynamic Programming

### Key Conditions
1. **Overlapping Subproblems**
   - Same subproblem solved multiple times
   - Can cache results
   
2. **Optimal Substructure**
   - Optimal solution contains optimal solutions to subproblems

### Memoization (Top-Down)
- **Approach**
  - Recursion + caching
  - Compute on demand
  
```c
int fib_memo(int n, int memo[]) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo);
    return memo[n];
}
```

### Tabulation (Bottom-Up)
- **Approach**
  - Iteration + table
  - Compute all subproblems
  
```c
int fib_tab(int n) {
    int dp[n+1];
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i-1] + dp[i-2];
    return dp[n];
}
```

## Greedy Algorithms

### Principle
- **Local optimal choice**
- Hope for global optimum
- Not always correct

### When Greedy Works
- Activity selection
- Huffman coding
- Dijkstra's algorithm
- Prim's/Kruskal's MST

### Knapsack Problem

#### 0/1 Knapsack
- Items cannot be divided
- **Solution**: Dynamic Programming
- O(nW) time

#### Fractional Knapsack
- Items can be divided
- **Solution**: Greedy (sort by value/weight ratio)
- O(n log n) time

## Additional Algorithms

### LCS (Longest Common Subsequence)
- **Problem**: Find longest subsequence in both strings
- **Solution**: DP
- `dp[i][j]` = LCS length of first i chars of s1, first j chars of s2

### Two Pointers
- **Technique**
  - Use two indices to traverse array
  - Often O(N) solution
  
- **Examples**
  - Two sum (sorted array)
  - Container with most water
  - Remove duplicates

### Sliding Window
- **Technique**
  - Fixed or variable window size
  - Move window across array
  
- **Examples**
  - Maximum sum subarray of size k
  - Longest substring without repeating chars
  - Network packet analysis

### Backtracking
- **Technique**
  - Try all possibilities
  - Prune branches that can't lead to solution
  
- **DFS with constraints**
  - N-Queens problem
  - Sudoku solver
  - Permutations/Combinations

### A* Search Algorithm
- **Principle**
  - Dijkstra + Heuristic
  - `f(n) = g(n) + h(n)`
    - g(n) = cost from start to n
    - h(n) = estimated cost from n to goal
  
- **Requirements**
  - Admissible heuristic (never overestimate)
  
- **Use Cases**
  - Game pathfinding
  - Route planning
  - AI navigation

## Recursion Considerations

### Stack Overflow Risk
- **Problem**
  - Each recursive call uses stack
  - Deep recursion → stack overflow
  
- **Embedded Systems**
  - Limited stack size (KB not MB)
  - Critical issue

### Tail Recursion
- **Definition**
  - Recursive call is last operation
  - Can be optimized to iteration by compiler
  
```c
// Tail recursive
int factorial_tail(int n, int acc) {
    if (n == 0) return acc;
    return factorial_tail(n-1, n*acc);
}
```

### Recursion to Iteration
- Always possible to convert
- Use explicit stack if needed

## Interview Practice
- [ ] Implement DFS and BFS (recursive and iterative)
- [ ] Dijkstra's algorithm from scratch
- [ ] Detect cycle in directed/undirected graph
- [ ] Topological sort (two methods)
- [ ] Kruskal's MST with Union-Find
- [ ] LCS problem with DP
- [ ] Backtracking: N-Queens
- [ ] Two pointers: container with most water
- [ ] Sliding window: longest substring
- [ ] Explain when to use greedy vs DP
