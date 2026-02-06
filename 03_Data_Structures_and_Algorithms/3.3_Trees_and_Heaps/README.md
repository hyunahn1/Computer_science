# 3.3 Trees & Heaps

## Tree Fundamentals

### Tree vs Graph
- **Tree**
  - Acyclic connected graph
  - Hierarchical structure
  - N nodes → N-1 edges
  - One path between any two nodes
  
- **Graph**
  - Can have cycles
  - May be disconnected
  - Multiple paths possible

## Binary Tree Types

### Classifications
1. **Full Binary Tree**
   - Every node has 0 or 2 children
   - No node has only 1 child
   
2. **Complete Binary Tree**
   - All levels filled except possibly last
   - Last level filled from left to right
   - Used in heap implementation
   
3. **Perfect Binary Tree**
   - All internal nodes have 2 children
   - All leaves at same level
   - Total nodes = 2^(h+1) - 1

### Depth vs Height
- **Depth**
  - Distance from root to node
  - Root depth = 0
  
- **Height**
  - Longest path from node to leaf
  - Leaf height = 0
  - Tree height = root height

## Binary Search Tree (BST)

### Properties
- **Invariant**
  - Left subtree < Node < Right subtree
  - Applies recursively to all subtrees
  
- **Complexity**
  - Search: O(log N) average, O(N) worst (skewed)
  - Insert: O(log N) average, O(N) worst
  - Delete: O(log N) average, O(N) worst

### Skewed Tree Problem
- **Issue**
  - Inserting sorted data creates linked list
  - Performance degrades to O(N)
  
- **Solution**
  - Use balanced BST (AVL, Red-Black)

## Tree Traversal

### Depth-First Search (DFS)

#### Pre-order (Root → Left → Right)
```c
void preorder(Node* node) {
    if (node == NULL) return;
    visit(node);
    preorder(node->left);
    preorder(node->right);
}
```
- **Use**: Copy tree, prefix expression

#### In-order (Left → Root → Right)
```c
void inorder(Node* node) {
    if (node == NULL) return;
    inorder(node->left);
    visit(node);
    inorder(node->right);
}
```
- **BST Property**: Visits nodes in sorted order
- **Use**: Get sorted elements from BST

#### Post-order (Left → Right → Root)
```c
void postorder(Node* node) {
    if (node == NULL) return;
    postorder(node->left);
    postorder(node->right);
    visit(node);
}
```
- **Use**: Delete tree, postfix expression

### Breadth-First Search (BFS)
- **Level-order traversal**
  - Visit level by level
  - Use queue
  
```c
void levelorder(Node* root) {
    Queue q;
    enqueue(&q, root);
    while (!isEmpty(&q)) {
        Node* node = dequeue(&q);
        visit(node);
        if (node->left) enqueue(&q, node->left);
        if (node->right) enqueue(&q, node->right);
    }
}
```

## Heap Data Structure

### Heap Properties

#### Min Heap
- **Property**
  - Parent ≤ Children
  - Root is minimum element
  
#### Max Heap
- **Property**
  - Parent ≥ Children
  - Root is maximum element

### Heap as Complete Binary Tree
- **Shape**
  - Complete binary tree
  - Fills level by level, left to right
  
- **Not a BST**
  - No ordering between left/right children
  - Only parent-child relationship matters

### Array Representation
- **Index Formula (1-based)**
  - Parent: `i / 2`
  - Left child: `2 * i`
  - Right child: `2 * i + 1`
  
- **Index Formula (0-based)**
  - Parent: `(i - 1) / 2`
  - Left child: `2 * i + 1`
  - Right child: `2 * i + 2`

### Heap Operations

#### Insert (Bubble Up)
1. Add element at end
2. Compare with parent
3. Swap if violates heap property
4. Repeat until heap property satisfied
- **Complexity**: O(log N)

#### Extract Min/Max (Bubble Down)
1. Remove root
2. Replace root with last element
3. Compare with children
4. Swap with smaller/larger child
5. Repeat until heap property satisfied
- **Complexity**: O(log N)

## Priority Queue

### Implementation with Heap
- **Why Heap?**
  - Insert: O(log N)
  - Extract min/max: O(log N)
  - Peek min/max: O(1)
  
- **Array Alternative**
  - Unsorted: Insert O(1), Extract O(N)
  - Sorted: Insert O(N), Extract O(1)
  
- **Heap is Balanced**
  - Best of both worlds

### Use Cases
- Task scheduling (OS)
- Dijkstra's algorithm
- Huffman coding
- Median maintenance

### Find Max in Min Heap
- **Complexity**: O(N)
- **Reason**
  - Max is in leaf nodes
  - Must check all leaves
  - Leaves ≈ N/2

## Balanced BST

### AVL Tree
- **Balance Factor**
  - Height(left) - Height(right)
  - Must be -1, 0, or 1
  
- **Rotations**
  - Single (LL, RR)
  - Double (LR, RL)
  
- **Characteristics**
  - Strictly balanced
  - Faster search than Red-Black
  - Slower insert/delete (more rotations)

### Red-Black Tree
- **Properties**
  - Every node is red or black
  - Root is black
  - Red node → black children
  - All paths have same black height
  
- **Characteristics**
  - Loosely balanced (height ≤ 2 log N)
  - Faster insert/delete than AVL
  - Used in Linux kernel, C++ std::map

## Advanced Trees

### B-Tree & B+Tree
- **Purpose**
  - Optimized for disk I/O
  - Used in databases and file systems
  
- **Characteristics**
  - Multiple keys per node
  - High branching factor
  - Minimize disk accesses
  
- **B+Tree**
  - All data in leaves
  - Leaves linked (range queries)
  - Interior nodes only for routing

### Trie (Prefix Tree)
- **Structure**
  - Tree of characters
  - Each path represents word
  
- **Complexity**
  - Insert/Search: O(L) where L = word length
  - Independent of number of words
  
- **Advantages**
  - Fast prefix matching
  - Autocomplete
  
- **Disadvantages**
  - High memory usage (many pointers)

### Segment Tree
- **Purpose**
  - Range queries (sum, min, max)
  - Point updates
  
- **Complexity**
  - Build: O(N)
  - Query: O(log N)
  - Update: O(log N)
  
- **Use Cases**
  - Range sum queries
  - Range minimum queries

## Classic Tree Algorithms

### Invert Binary Tree
```c
Node* invert(Node* root) {
    if (root == NULL) return NULL;
    Node* temp = root->left;
    root->left = invert(root->right);
    root->right = invert(temp);
    return root;
}
```

### LCA (Lowest Common Ancestor)
- **Binary Tree**
  - Recursive: check if nodes in left/right
  - O(N) time
  
- **BST**
  - Compare with root value
  - O(h) time where h = height

### Perfect Binary Tree Properties
- **Height h**
  - Total nodes: 2^(h+1) - 1
  - Leaf nodes: 2^h
  - Internal nodes: 2^h - 1

## Huffman Coding

### Algorithm
1. Count character frequencies
2. Build min heap of nodes
3. Repeatedly merge two smallest nodes
4. Build tree bottom-up
5. Assign codes: left=0, right=1

### Properties
- Variable-length prefix-free codes
- More frequent → shorter code
- Used in compression (ZIP, JPEG)

## Heap Sort

### Algorithm
1. Build max heap: O(N)
2. Swap root with last element
3. Reduce heap size
4. Heapify root
5. Repeat until heap size = 1

### Characteristics
- **Time**: O(N log N) always
- **Space**: O(1) (in-place)
- **Not stable**: equal elements may swap
- **Cache inefficient**: random access pattern

## Interview Practice
- [ ] Implement BST insert/delete/search
- [ ] Validate if tree is BST
- [ ] Implement min heap from scratch
- [ ] Find kth largest element using heap
- [ ] Serialize and deserialize binary tree
- [ ] Find LCA in binary tree and BST
- [ ] Implement level-order traversal
- [ ] Invert binary tree
- [ ] Check if tree is balanced
- [ ] Build tree from inorder + preorder
