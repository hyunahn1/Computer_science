# 3.4 Sorting & Searching Algorithms

## Simple Sorting Algorithms

### Bubble Sort
- **Algorithm**
  - Compare adjacent elements
  - Swap if out of order
  - Repeat until no swaps
  
- **Complexity**
  - Time: O(N²)
  - Space: O(1)
  
- **Why Not Used?**
  - Too many swap operations
  - Slow even for small data
  - No advantages over insertion sort

### Selection Sort
- **Algorithm**
  - Find minimum element
  - Swap with first unsorted position
  - Repeat for remaining elements
  
- **Complexity**
  - Time: O(N²) always
  - Space: O(1)
  
- **Characteristics**
  - Few swaps: exactly N-1
  - Not stable
  - Not adaptive (same speed even if sorted)

### Insertion Sort
- **Algorithm**
  - Build sorted portion one element at a time
  - Insert new element in correct position
  
- **Complexity**
  - Best: O(N) (nearly sorted)
  - Average: O(N²)
  - Worst: O(N²)
  - Space: O(1)
  
- **Advantages**
  - Adaptive (fast on nearly sorted)
  - Stable
  - Online (can sort as data arrives)
  - Good for small arrays (<10 elements)

## Efficient Sorting Algorithms

### Quick Sort

#### Algorithm
1. Choose pivot element
2. Partition: elements < pivot on left, > pivot on right
3. Recursively sort left and right

#### Complexity
- **Best/Average**: O(N log N)
- **Worst**: O(N²) (already sorted, bad pivot)
- **Space**: O(log N) stack space
- **Not stable**

#### Pivot Selection Strategies
- **First/Last element**: Worst for sorted data
- **Random**: Good average case
- **Median-of-3**: First, middle, last → take median
- **Better performance on partially sorted**

#### Why Fastest in Practice?
- Cache friendly (locality)
- In-place (no extra memory)
- Small constant factors
- Good average case

### Merge Sort

#### Algorithm
1. Divide array into halves
2. Recursively sort each half
3. Merge two sorted halves

#### Complexity
- **Always**: O(N log N)
- **Space**: O(N) for temporary array
- **Stable**: maintains relative order

#### Advantages
- Predictable performance
- Stable sorting
- Good for linked lists (no random access needed)

#### Disadvantages
- Extra memory required
- Slower than quick sort on average (cache)

### Heap Sort

#### Algorithm
1. Build max heap: O(N)
2. Swap root with last element
3. Reduce heap size, heapify
4. Repeat

#### Complexity
- **Always**: O(N log N)
- **Space**: O(1) in-place
- **Not stable**

#### Comparison with Quick Sort
| Feature | Heap Sort | Quick Sort |
|---------|-----------|------------|
| Worst case | O(N log N) | O(N²) |
| Average | O(N log N) | O(N log N) faster |
| Cache | Poor (random jumps) | Good (locality) |
| Stable | No | No |
| In-place | Yes | Yes |

## Non-Comparison Sorting

### Counting Sort
- **Algorithm**
  - Count frequency of each value
  - Calculate cumulative counts
  - Place elements in output array
  
- **Complexity**
  - Time: O(N + K) where K = range
  - Space: O(K)
  
- **Requirements**
  - Integer keys
  - Known range
  
- **Use When**
  - Small range of values
  - Plenty of memory

### Radix Sort
- **Algorithm**
  - Sort by each digit (LSD or MSD)
  - Use stable sort for each digit (counting sort)
  
- **Complexity**
  - Time: O(d × N) where d = number of digits
  - Space: O(N + K)
  
- **Use Cases**
  - Fixed-length strings
  - Fixed-width integers
  - When N >> log N

## Stable vs Unstable Sort

### Stable Sort
- **Definition**
  - Equal elements maintain original order
  
- **Examples**
  - Merge Sort
  - Insertion Sort
  - Bubble Sort
  - Counting Sort
  - Radix Sort
  
- **Use Case**
  - Sort by multiple keys (e.g., sort by name, then by age)

### Unstable Sort
- **Examples**
  - Quick Sort
  - Heap Sort
  - Selection Sort
  
- **Can Make Stable**
  - Store original index with each element
  - Compare indices when values equal

## Binary Search

### Algorithm
```c
int binary_search(int arr[], int n, int target) {
    int low = 0, high = n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;  // Avoid overflow
        if (arr[mid] == target)
            return mid;
        else if (arr[mid] < target)
            low = mid + 1;
        else
            high = mid - 1;
    }
    return -1;  // Not found
}
```

### Mid Calculation Bug
- **Wrong**: `(low + high) / 2`
  - Overflow if low + high > INT_MAX
  
- **Correct**: `low + (high - low) / 2`
  - No overflow risk

### Prerequisite
- **Array MUST be sorted**
- Cannot work on unsorted data

### Complexity
- Time: O(log N)
- Space: O(1) iterative, O(log N) recursive

## Binary Search Variants

### Lower Bound
- **Find**: First position ≥ target
- **Use**: Insert position to maintain sorted order

```c
int lower_bound(int arr[], int n, int target) {
    int low = 0, high = n;
    while (low < high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] < target)
            low = mid + 1;
        else
            high = mid;
    }
    return low;
}
```

### Upper Bound
- **Find**: First position > target
- **Use**: Range queries

## Hash Table Collision Resolution

### Chaining
- **Method**
  - Each bucket is a linked list
  - Colliding elements added to list
  
- **Advantages**
  - Simple implementation
  - Never "full"
  - Good for unknown dataset size
  
- **Disadvantages**
  - Extra memory for pointers
  - Cache unfriendly

### Open Addressing
- **Method**
  - All elements in main table
  - Find next empty slot on collision
  
- **Linear Probing**
  - Try `(hash + i) % size`
  - Clustering problem
  
- **Quadratic Probing**
  - Try `(hash + i²) % size`
  - Less clustering
  
- **Double Hashing**
  - Try `(hash1 + i × hash2) % size`
  - Best distribution

## Hybrid & Optimized Sorts

### Tim Sort
- **Used In**
  - Python `sorted()`
  - Java `Arrays.sort()` for objects
  
- **Algorithm**
  - Identify runs (sorted subsequences)
  - Use insertion sort for small runs
  - Merge runs using merge sort
  
- **Advantages**
  - O(N) on nearly sorted data
  - O(N log N) worst case
  - Stable
  - Adaptive

### External Sorting
- **Problem**
  - Data too large for memory
  
- **Solution**
  - Divide into chunks that fit in memory
  - Sort each chunk (internal sort)
  - Merge chunks (k-way merge)
  
- **Based on**: Merge sort
- **Use**: Large database files

## Advanced Search Algorithms

### Interpolation Search
- **Idea**
  - Estimate position based on value distribution
  - Like looking up word in dictionary
  
- **Formula**
  ```
  pos = low + ((target - arr[low]) × (high - low)) / (arr[high] - arr[low])
  ```
  
- **Complexity**
  - Average: O(log log N) for uniform distribution
  - Worst: O(N)

### Quick Select (K-th Element)
- **Problem**
  - Find k-th smallest element
  
- **Algorithm**
  - Like quick sort, but only recurse on one partition
  
- **Complexity**
  - Average: O(N)
  - Worst: O(N²)
  
- **Use Cases**
  - Find median: O(N) average
  - Top K elements

## Sorting Lower Bound

### Comparison-Based Sorting
- **Theorem**
  - No comparison-based sort can be faster than O(N log N)
  
- **Proof**
  - Decision tree has N! leaves
  - Tree height ≥ log₂(N!) ≈ N log N
  
- **Conclusion**
  - Merge sort, heap sort are asymptotically optimal
  - Can only improve constants, not Big-O

## Practical Considerations

### Linked List Sorting
- **Best Algorithm**: Merge Sort
  - No random access needed
  - Only pointer manipulation
  - O(1) space (in-place link changes)

### Top K Elements from N
- **Problem**: Find top 10 from 1 million
- **Solution**: Min Heap of size K
  1. Build heap with first K elements
  2. For remaining elements:
     - If larger than root, replace root and heapify
  3. Heap contains top K elements
- **Complexity**: O(N log K)
- **Better than**: Sorting entire array O(N log N)

## Interview Practice
- [ ] Implement quick sort with median-of-3
- [ ] Implement merge sort for linked list
- [ ] Write binary search (iterative and recursive)
- [ ] Find first and last position of element (lower/upper bound)
- [ ] Implement counting sort
- [ ] Find kth largest element (quick select)
- [ ] Merge K sorted lists
- [ ] Sort colors (Dutch flag problem)
- [ ] Top K frequent elements
- [ ] Explain why quick sort is fastest in practice
