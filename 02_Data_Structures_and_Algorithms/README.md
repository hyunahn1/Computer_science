# Data Structures & Algorithms

## Overview
Comprehensive study materials covering fundamental data structures and algorithms. Focus on trade-offs, time/space complexity, and practical applications. In embedded systems, **space complexity is as important as time complexity** due to memory constraints.

## Course Structure

### [2.1 Basic Data Structures](./2.1_Basic_Structures/)
- Data structure selection criteria (3 factors)
- Big-O notation and complexity analysis (formal complexity classes: **[4.2](../04_Theory_of_Computer_Science/4.2_Computability_and_Complexity/)**)
- Array vs Linked List trade-offs
- Cache locality advantages
- Dynamic array resizing (amortized O(1))
- Doubly and circular linked lists
- Stack, Queue, Deque
- Ring buffer (critical for embedded systems)
- Sparse matrix storage
- Binary search requirements

### [2.2 Bit Manipulation & Optimization](./2.2_Bit_Manipulation/)
- XOR operations and applications
- Power-of-two operations (`x & (x-1)`)
- Odd/even check, multiply/divide by 2
- Bit counting (Brian Kernighan's algorithm)
- Bit masking techniques
- Two's complement representation
- Endianness and byte swapping
- Shift operations (logical vs arithmetic)
- Bitmap data structure
- Gray code and MSB finding

### [2.3 Trees & Heaps](./2.3_Trees_and_Heaps/)
- Tree vs Graph differences
- Binary tree types (Full, Complete, Perfect)
- Binary Search Tree (BST) operations
- Tree traversals (Pre/In/Post-order, Level-order)
- Heap properties and operations
- Priority queue implementation
- Balanced BST (AVL vs Red-Black)
- B-Tree and B+Tree (databases)
- Trie (prefix tree)
- Segment tree for range queries
- Huffman coding

### [2.4 Sorting & Searching](./2.4_Sorting_and_Searching/)
- Simple sorts (Bubble, Selection, Insertion)
- Efficient sorts (Quick, Merge, Heap)
- Pivot selection strategies
- Non-comparison sorts (Counting, Radix)
- Stable vs Unstable sorting
- Binary search and variants (Lower/Upper bound)
- Hash collision resolution (Chaining, Open addressing)
- **String pattern matching**: KMP, Rabin-Karp (and rolling hash intuition)
- Tim Sort hybrid approach
- External sorting for large data
- Quick Select for k-th element
- Sorting lower bound proof

### [2.5 Graph Algorithms & Advanced Topics](./2.5_Graph_and_Advanced/)
- **Cache / eviction design**: LRU, LFU — map to data structures (hash map + doubly linked list, heaps, frequency lists)
- Graph representations (Matrix vs List)
- DFS and BFS traversals
- Shortest path (Dijkstra, Bellman-Ford, Floyd-Warshall)
- Minimum Spanning Tree (Kruskal, Prim)
- Topological sort for DAGs
- Union-Find (Disjoint Set)
- Dynamic Programming (Memoization vs Tabulation)
- Greedy algorithms
- Knapsack problems (0/1 vs Fractional)
- LCS (Longest Common Subsequence)
- Two pointers and sliding window
- Backtracking techniques
- A* search algorithm
- Recursion considerations (stack overflow, tail recursion)

## Study Philosophy

### Trade-off Thinking
- Don't just memorize algorithms
- Understand **why** to choose each data structure
- Compare time vs space complexity
- Consider cache performance (embedded systems)

### Embedded Systems Focus
- Memory is limited (RAM in KB, not GB)
- Space complexity often more critical than time
- Cache locality matters greatly
- Stack size is constrained (recursion risks)

### Interview Preparation
- Explain algorithms, don't just code
- Discuss complexity analysis
- Consider edge cases
- Optimize step by step

## Complexity Reference

| Data Structure | Access | Search | Insert | Delete | Space |
|----------------|--------|--------|--------|--------|-------|
| Array | O(1) | O(N) | O(N) | O(N) | O(N) |
| Linked List | O(N) | O(N) | O(1)* | O(1)* | O(N) |
| Hash Table | - | O(1) avg | O(1) avg | O(1) avg | O(N) |
| BST | O(log N) avg | O(log N) avg | O(log N) avg | O(log N) avg | O(N) |
| Heap | - | - | O(log N) | O(log N) | O(N) |

*At known position

| Sort Algorithm | Best | Average | Worst | Space | Stable |
|----------------|------|---------|-------|-------|--------|
| Quick Sort | O(N log N) | O(N log N) | O(N²) | O(log N) | No |
| Merge Sort | O(N log N) | O(N log N) | O(N log N) | O(N) | Yes |
| Heap Sort | O(N log N) | O(N log N) | O(N log N) | O(1) | No |
| Counting Sort | O(N+K) | O(N+K) | O(N+K) | O(K) | Yes |

## Advanced Topics to Add

- Proof techniques: loop invariants, exchange arguments, cut properties, amortized analysis, lower bounds.
- Advanced structures: balanced trees, B-trees, skip lists, bloom filters, persistent structures, lock-free queues.
- Graph algorithms: SCC, max flow/min cut, bipartite matching, shortest paths with potentials, A* admissibility.
- Algorithm design: divide and conquer, DP optimization, randomized algorithms, approximation algorithms.
- Practical performance: cache locality, allocation patterns, branch prediction, recursion limits, adversarial inputs.

## Expert Depth Checklist
- [ ] Prove correctness using an invariant, induction, exchange argument, or cut property where appropriate.
- [ ] Derive time and space complexity, including worst case, amortized case, and hidden constants when relevant.
- [ ] Implement the core algorithm or data structure without relying on library shortcuts.
- [ ] Test edge cases: empty input, duplicate keys, adversarial ordering, overflow, recursion depth, and memory pressure.
- [ ] Compare competing approaches and state when each one wins in practice.
- [ ] Explain cache locality, allocation behavior, and stack usage when they affect real performance.
- [ ] Solve at least three problems that require recognizing the pattern rather than copying the algorithm.
