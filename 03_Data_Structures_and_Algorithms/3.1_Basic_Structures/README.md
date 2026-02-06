# 3.1 Basic Data Structures (Arrays & Lists)

## Fundamental Considerations

### Data Structure Selection Criteria (3 Factors)
1. **Data Size**
   - How much data needs to be stored?
   - Static vs dynamic growth?
   
2. **Access Frequency (Read/Write)**
   - Read-heavy vs Write-heavy?
   - Random access vs sequential?
   
3. **Memory Constraints**
   - Available RAM (critical in embedded systems)
   - Memory fragmentation concerns

## Complexity Analysis

### Big-O Notation (Time Complexity)
- **Definition**
  - Growth rate of operations as input size N increases
  - Worst-case scenario analysis
  
- **Why Important?**
  - Predicts algorithm scalability
  - Compares algorithm efficiency
  - Identifies bottlenecks

### Complexity Comparison
- **O(1) - Constant Time**
  - Array index access
  - Hash table lookup (average)
  
- **O(log N) - Logarithmic**
  - Binary search
  - Balanced tree operations
  
- **O(N) - Linear**
  - Array traversal
  - Linear search
  
- **O(N log N) - Linearithmic**
  - Efficient sorting (merge, heap, quick)
  
- **O(N²) - Quadratic**
  - Nested loops
  - Bubble sort

### Space Complexity
- **Definition**
  - Extra memory required by algorithm
  - Auxiliary space (excluding input)
  
- **Embedded Systems**
  - Often more critical than time complexity
  - RAM is limited resource
  
- **Trade-offs**
  - Time vs space (e.g., memoization)

## Array vs Linked List

### Key Difference
| Feature | Array | Linked List |
|---------|-------|-------------|
| Memory | Contiguous | Non-contiguous |
| Access | O(1) by index | O(N) traversal |
| Insert/Delete | O(N) (shifting) | O(1) at position |
| Memory overhead | None | Pointer per node |
| Cache performance | Excellent | Poor |

### Cache Locality Advantage (Array)
- **Spatial Locality**
  - Adjacent elements in memory
  - CPU prefetch works efficiently
  - Cache line utilization
  
- **Performance Impact**
  - Array traversal: fast (cache hits)
  - Linked list: slow (cache misses)
  - Modern CPUs benefit greatly from arrays

## Dynamic Array (Vector)

### Resizing Mechanism
- **Growth Strategy**
  - When full: allocate 2× size
  - Copy all elements
  - Free old memory
  
- **Amortized O(1)**
  - Most insertions: O(1)
  - Occasional resize: O(N)
  - Average over many operations: O(1)
  
- **Trade-off**
  - Wasted space (capacity > size)
  - Faster than linked list in most cases

## Linked List Variants

### Doubly Linked List
- **Advantages**
  - Bidirectional traversal
  - Delete node with only its pointer (no prev needed)
  
- **Disadvantages**
  - 2× pointer memory overhead
  - More complex to maintain

### Circular Linked List
- **Characteristics**
  - Last node points to first
  - No NULL termination
  
- **Use Cases**
  - Round-robin scheduling
  - Ring buffer implementation
  - Circular queue

## Insertion & Deletion

### Array: Middle Element Deletion
- **Problem**
  - Must shift all following elements left
  - Complexity: O(N)
  
- **Example**
  ```
  [1, 2, 3, 4, 5] → delete 3
  [1, 2, _, 4, 5] → shift
  [1, 2, 4, 5]
  ```

### Linked List: Middle Element Deletion
- **Complexity**
  - Find position: O(N)
  - Delete: O(1) (just change pointer)
  
- **If position known**: O(1)
  ```
  prev->next = curr->next;
  free(curr);
  ```

## Sparse Matrix
- **Definition**
  - Matrix with mostly zeros
  - Inefficient to store as 2D array
  
- **Storage Methods**
  - Coordinate list (row, col, value)
  - Compressed sparse row (CSR)
  - Hash table
  
- **Memory Savings**
  - Only store non-zero elements

## Stack, Queue, Deque

### Stack (LIFO - Last In First Out)
- **Operations**
  - Push: add to top - O(1)
  - Pop: remove from top - O(1)
  - Peek: view top - O(1)
  
- **Use Cases**
  - Function call stack
  - Undo/Redo functionality
  - Expression evaluation
  - Backtracking algorithms

### Queue (FIFO - First In First Out)
- **Operations**
  - Enqueue: add to rear - O(1)
  - Dequeue: remove from front - O(1)
  
- **Use Cases**
  - Task scheduling
  - BFS traversal
  - Buffering (print queue, I/O)

### Deque (Double-Ended Queue)
- **Characteristics**
  - Insert/delete from both ends
  - Combines stack + queue functionality
  
- **Operations**
  - Push/pop front: O(1)
  - Push/pop back: O(1)

## Circular Queue (Ring Buffer)

### Full vs Empty Distinction
- **Problem**
  - Both states: `front == rear`
  - Need to differentiate
  
- **Solutions**
  1. **Waste one slot**
     - Empty: `front == rear`
     - Full: `(rear + 1) % SIZE == front`
  
  2. **Use counter**
     - Track number of elements
  
  3. **Use flag**
     - Boolean full/empty flag

### Ring Buffer in Embedded Systems
- **Why Essential?**
  - Continuous data stream (UART, ADC)
  - No memory reallocation
  - Constant memory footprint
  - Producer-consumer pattern
  
- **UART Example**
  - ISR writes to buffer (producer)
  - Main loop reads from buffer (consumer)
  - Infinite circular operation

## Classic Algorithms

### Reverse Linked List (Iterative)
```c
Node* reverse(Node* head) {
    Node *prev = NULL, *curr = head, *next;
    while (curr) {
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```
- **Three pointers**: prev, curr, next
- **Link reversal**: point to previous

### Binary Search Requirement
- **Prerequisite**
  - Array MUST be sorted
  - Cannot work on unsorted data
  
- **Complexity**
  - O(log N) time
  - O(1) space

## Hash Table

### Principle
- **Key-Value Mapping**
  - Hash function: key → index
  - Direct access by key
  
- **Complexity**
  - Average: O(1)
  - Worst case (many collisions): O(N)
  
- **Load Factor**
  - α = n / m (elements / buckets)
  - Keep α < 0.75 for performance

### Hash Collision (covered in section 3.4)

## Interview Practice
- [ ] Explain array vs linked list trade-offs
- [ ] Implement ring buffer in C
- [ ] Reverse linked list (iterative and recursive)
- [ ] Explain amortized analysis for dynamic array
- [ ] Design cache-friendly data structure
- [ ] Implement stack using two queues
- [ ] Detect cycle in linked list (Floyd's algorithm)
