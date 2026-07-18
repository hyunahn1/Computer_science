맞습니다. Tree 구조는 이미 봤으니 **Heap부터 바로 시작하겠습니다.**

# Lecture 8. Heap Fundamentals

## 1. 핵심 질문

이번 강의의 목표는 다음을 이해하는 것이다.

1. Heap은 정확히 어떤 자료구조인가?
2. Min Heap과 Max Heap은 무엇이 다른가?
3. Heap이 왜 Complete Binary Tree여야 하는가?
4. Heap과 BST는 어떻게 다른가?
5. Root가 왜 최소값 또는 최대값임을 보장할 수 있는가?
6. Heap은 실제 메모리에서 왜 배열로 구현하는가?
7. Heap의 invariant가 깨지면 어떻게 복구하는가?

---

# 2. Heap의 형식적 정의

Heap은 보통 다음 두 조건을 만족하는 binary tree다.

## 2.1 Shape invariant

Heap은 **Complete Binary Tree**다.

즉:

* 마지막 level을 제외한 모든 level이 채워져 있다.
* 마지막 level은 왼쪽부터 연속적으로 채워진다.

```text
        4
      /   \
     7     6
    / \   /
   9  10 8
```

위 tree는 complete하다.

---

## 2.2 Order invariant

Heap은 parent와 child 사이에 특정 대소 관계를 유지한다.

### Min Heap

모든 parent가 자신의 child보다 작거나 같다.

[
parent \le child
]

```text
        2
      /   \
     5     4
    / \   /
   9   7 8
```

모든 간선을 확인하면:

```text
2 ≤ 5
2 ≤ 4
5 ≤ 9
5 ≤ 7
4 ≤ 8
```

따라서 Min Heap이다.

### Max Heap

모든 parent가 자신의 child보다 크거나 같다.

[
parent \ge child
]

```text
        20
       /  \
     15    18
    / \    /
   8  10  12
```

---

## 핵심 정의

[
\boxed{
Heap = Complete\ Binary\ Tree + Heap\ Order
}
]

Heap은 단순히 값의 순서만 맞으면 되는 것이 아니다.

다음 두 invariant가 동시에 필요하다.

```text
Shape invariant: Complete Binary Tree
Order invariant: Parent-child ordering
```

---

# 3. 직관적 설명

Min Heap을 “작은 값이 위로 떠오르는 구조”라고 생각할 수 있다.

```text
작은 값
   ↑
   │
큰 값
```

Max Heap은 반대다.

```text
큰 값
   ↑
   │
작은 값
```

하지만 Heap 전체가 완전히 정렬되어 있는 것은 아니다.

Min Heap은 다음만 보장한다.

> 부모는 자기 자식보다 작거나 같다.

형제 사이에는 아무 조건도 없다.

```text
        2
       / \
    100   3
```

이것도 valid Min Heap이다.

* `2 ≤ 100`
* `2 ≤ 3`

왼쪽 자식 100이 오른쪽 자식 3보다 커도 문제없다.

---

# 4. 왜 Heap이 필요한가

다음과 같은 상황을 생각해보자.

```text
현재 대기 중인 작업 중
가장 우선순위가 높은 작업을 계속 꺼내야 한다.
```

예:

* 운영체제 작업 스케줄링
* 가장 가까운 정점 선택
* 가장 작은 비용의 이벤트 처리
* 가장 빈도가 낮은 두 문자의 선택
* Top K 원소 찾기

단순 배열을 사용하면 trade-off가 생긴다.

## 정렬되지 않은 배열

```text
[8, 3, 10, 2, 7]
```

삽입은 배열 끝에 넣으면 된다.

* 삽입: `O(1)`
* 최솟값 찾기: 모든 원소 확인 → `O(N)`
* 최솟값 제거: `O(N)`

## 정렬된 배열

```text
[2, 3, 7, 8, 10]
```

최솟값은 바로 찾을 수 있다.

* 최솟값 확인: `O(1)`
* 삽입 위치 찾고 이동: `O(N)`

## Heap

* 삽입: `O(log N)`
* 최소·최대 확인: `O(1)`
* 최소·최대 제거: `O(log N)`

Heap은 모든 원소를 완전히 정렬하지 않는다.

대신 다음에 필요한 원소, 즉 최소값이나 최대값만 빠르게 관리한다.

---

# 5. Min Heap의 내부 원리

다음 Min Heap을 보자.

```text
        2
      /   \
     5     4
    / \   / \
   9   7 8   6
```

## 왜 Root가 최소값인가?

Heap invariant에 의해:

```text
root ≤ root의 자식
```

그리고 각 자식 역시 자신의 자식보다 작거나 같다.

예를 들어 root에서 어떤 leaf까지 내려가면:

[
root \le v_1 \le v_2 \le \cdots \le leaf
]

따라서 root는 모든 descendant보다 작거나 같다.

Tree의 모든 노드는 root의 descendant이므로:

[
\boxed{root = minimum}
]

Max Heap에서도 같은 논리로 root가 최대값이다.

---

# 6. Heap은 BST가 아니다

Heap과 BST는 둘 다 binary tree지만 invariant가 전혀 다르다.

## Binary Search Tree

```text
왼쪽 subtree의 모든 값 < node
오른쪽 subtree의 모든 값 > node
```

예:

```text
        8
       / \
      3   12
     / \  / \
    1  6 10 15
```

BST는 좌우 방향에 의미가 있다.

---

## Min Heap

```text
parent ≤ children
```

예:

```text
        1
       / \
      8   3
     / \
    10  9
```

이것은 Min Heap이지만 BST는 아니다.

Root의 왼쪽 subtree에 `8, 10, 9`가 있고 오른쪽에는 `3`이 있다.

BST라면 왼쪽 값들이 root보다 작아야 하지만 그렇지 않다.

---

## 비교

| 관점           | BST                 | Heap                             |
| ------------ | ------------------- | -------------------------------- |
| 핵심 invariant | Left < Node < Right | Parent ≤ Child 또는 Parent ≥ Child |
| 좌우 child 관계  | 중요                  | 서로 관계 없음                         |
| 최소값          | 가장 왼쪽 node          | Min Heap의 root                   |
| 최대값          | 가장 오른쪽 node         | Max Heap의 root                   |
| 임의 값 검색      | 평균 `O(log N)` 가능    | 일반적으로 `O(N)`                     |
| 최소·최대 접근     | 높이만큼 이동             | `O(1)`                           |
| 형태           | 반드시 complete일 필요 없음 | 반드시 complete                     |
| 대표 용도        | 검색·정렬된 순회           | Priority Queue                   |

---

# 7. Heap에서 임의 값 검색이 느린 이유

Min Heap:

```text
        2
       / \
      5   4
     / \ / \
    9  7 8  6
```

`7`을 찾는다고 하자.

Root가 2라는 사실은 7이 왼쪽에 있는지 오른쪽에 있는지 알려주지 않는다.

```text
왼쪽 subtree: 5, 9, 7
오른쪽 subtree: 4, 8, 6
```

두 subtree 모두 7이 있을 가능성이 있다.

따라서 최악에는 모든 node를 확인해야 한다.

[
O(N)
]

Heap은 검색용 자료구조가 아니다.

Heap은 다음 질문에 특화되어 있다.

> 현재 가장 작거나 가장 큰 원소는 무엇인가?

---

# 8. 왜 Complete Binary Tree를 사용하는가

## 이유 1: 높이가 작게 유지된다

Complete Binary Tree는 level을 빈틈없이 채운다.

```text
Level 0:      1개
Level 1:      2개
Level 2:      4개
Level 3:      8개
```

높이가 하나 증가할 때 저장 가능한 node 수가 약 두 배가 된다.

따라서 node가 (N)개라면 높이는 대략:

[
\log_2 N
]

예:

|      Node 수 | Heap 높이 |
| ----------: | ------: |
|           1 |       0 |
|         2–3 |       1 |
|         4–7 |       2 |
|        8–15 |       3 |
|       16–31 |       4 |
|     약 1,000 |     약 9 |
| 약 1,000,000 |    약 19 |

삽입이나 삭제 시 root와 leaf 사이를 이동하므로 약 `O(log N)`이다.

---

## 이유 2: 배열에 빈틈없이 저장할 수 있다

다음 complete tree를 보자.

```text
        2
      /   \
     5     4
    / \   /
   9   7 8
```

0-based 배열에 level 순서로 저장하면:

```text
Index:  0  1  2  3  4  5
Value: [2, 5, 4, 9, 7, 8]
```

Pointer 없이 parent와 child의 위치를 계산할 수 있다.

```text
          data[0]
          /     \
     data[1]   data[2]
      /   \      /
 data[3] data[4] data[5]
```

---

# 9. 배열 Index 공식

## 0-based indexing

현재 node의 index가 `i`일 때:

```c
parent = (i - 1) / 2
left   = 2 * i + 1
right  = 2 * i + 2
```

단, root `i == 0`에는 parent가 없다.

---

## 예시

배열:

```text
Index:  0  1  2  3  4  5
Value: [2, 5, 4, 9, 7, 8]
```

Index 1의 값은 5다.

```text
left  = 2 * 1 + 1 = 3 → 9
right = 2 * 1 + 2 = 4 → 7
parent = (1 - 1) / 2 = 0 → 2
```

Index 5의 값은 8이다.

```text
parent = (5 - 1) / 2 = 2 → 4
```

---

## 왜 공식이 성립하는가?

Level-order로 배열에 저장하기 때문이다.

```text
        0
      /   \
     1     2
    / \   / \
   3   4 5   6
```

각 index `i`의 child가 항상 다음 위치에 배치된다.

```text
left  = 2i + 1
right = 2i + 2
```

Complete Tree이므로 중간 빈자리가 없어서 이 관계가 유지된다.

---

# 10. Min Heap의 예와 반례

## 올바른 Min Heap

```text
        1
       / \
      3   2
     / \ / \
    8  5 7  4
```

Parent-child 관계:

```text
1 ≤ 3, 2
3 ≤ 8, 5
2 ≤ 7, 4
```

모두 성립한다.

---

## 잘못된 Min Heap

```text
        1
       / \
      6   2
     / \
    4   8
```

`6 ≤ 4`가 거짓이다.

따라서 heap property가 깨졌다.

---

## 값의 순서만 맞지만 Heap이 아닌 구조

```text
        1
       / \
      2   3
           \
            4
```

모든 parent가 child보다 작다.

하지만 Complete Binary Tree가 아니다.

오른쪽 아래에 node가 있는데 왼쪽 자리가 비어 있다.

따라서 Min Heap이 아니다.

---

# 11. Heap invariant

Min Heap의 핵심 invariant는 두 가지다.

## Shape invariant

배열에서 유효한 원소는 다음처럼 연속되어야 한다.

```text
data[0], data[1], ..., data[size - 1]
```

중간에 빈 index가 없어야 한다.

## Order invariant

모든 유효한 child index `i`에 대해:

[
data[parent(i)] \le data[i]
]

Max Heap은 반대다.

[
data[parent(i)] \ge data[i]
]

Heap 연산은 항상 이 두 invariant를 유지해야 한다.

---

# 12. Heap의 C 자료구조

```c
#include <stddef.h>

typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} MinHeap;
```

각 필드의 의미:

```text
data      : 실제 원소를 저장하는 동적 배열
size      : 현재 저장된 원소 수
capacity  : 배열에 저장 가능한 최대 원소 수
```

Invariant:

```text
0 <= size <= capacity
data != NULL, if capacity > 0
유효 원소는 data[0 ... size-1]
```

---

# 13. Heap 초기화

```c
#include <stddef.h>

void min_heap_init(MinHeap *heap)
{
    if (heap == NULL) {
        return;
    }

    heap->data = NULL;
    heap->size = 0;
    heap->capacity = 0;
}
```

초기 상태:

```text
data = NULL
size = 0
capacity = 0
```

Empty Heap이다.

---

# 14. Heap 메모리 해제

Heap은 pointer-based tree가 아니라 배열이므로 아래 child부터 하나씩 `free`할 필요가 없다.

```c
#include <stdlib.h>

void min_heap_destroy(MinHeap *heap)
{
    if (heap == NULL) {
        return;
    }

    free(heap->data);

    heap->data = NULL;
    heap->size = 0;
    heap->capacity = 0;
}
```

Allocation이 배열 하나뿐이라면 `free(heap->data)` 한 번이면 된다.

이것이 pointer-based binary tree와 heap의 중요한 차이다.

## Pointer-based tree

```text
node마다 개별 malloc
→ child부터 재귀적으로 free
```

## Array-based heap

```text
하나의 배열 allocation
→ 배열 한 번 free
```

단, heap의 원소가 별도 동적 메모리의 포인터라면 ownership 정책에 따라 각 원소도 먼저 해제해야 한다.

예:

```c
typedef struct {
    Task **data;
    size_t size;
    size_t capacity;
} TaskHeap;
```

Heap이 `Task` 객체까지 소유한다면:

```text
각 Task free
→ data 배열 free
```

Heap이 pointer만 빌려서 저장한다면:

```text
data 배열만 free
```

Ownership을 명확히 해야 한다.

---

# 15. Peek 연산

Min Heap의 최솟값은 root, 즉 배열의 index 0에 있다.

```c
int min_heap_peek(const MinHeap *heap, int *out_value)
{
    if (heap == NULL || out_value == NULL) {
        return 0;
    }

    if (heap->size == 0) {
        return 0;
    }

    *out_value = heap->data[0];
    return 1;
}
```

## 시간 복잡도

배열의 첫 번째 원소 하나만 읽는다.

[
O(1)
]

Heap 크기가 10개든 100만 개든 작업 수가 거의 같다.

---

# 16. 시간 복잡도 쉽게 이해하기

## Heap 높이: `O(log N)`

Complete Binary Tree에서는 한 level마다 저장 가능한 node 수가 두 배가 된다.

```text
높이 0 → 1개
높이 1 → 최대 3개
높이 2 → 최대 7개
높이 3 → 최대 15개
높이 4 → 최대 31개
```

따라서 node 수가 많이 증가해도 높이는 천천히 증가한다.

```text
N = 8       → 높이 약 3
N = 1,024   → 높이 약 10
N = 1,000,000 → 높이 약 20
```

삽입과 삭제는 한 level씩 위나 아래로 움직이기 때문에 `O(log N)`이다.

---

## Peek: `O(1)`

Root 하나만 확인한다.

```text
data[0]
```

---

## 임의 값 검색: `O(N)`

좌우 subtree의 정렬 관계가 없으므로 최악에는 모든 값을 확인해야 한다.

---

## 전체 Heap 검사: `O(N)`

모든 parent-child 관계를 검사해야 한다.

```text
각 node를 한 번 확인
```

---

# 17. 실제 성능과 메모리 구조

Heap은 배열 기반이어서 pointer-based tree보다 cache locality가 좋다.

```text
[data0][data1][data2][data3][data4]
```

인접 node가 메모리에서도 인접하게 배치된다.

장점:

* Pointer chasing 없음
* Node별 `malloc` 없음
* Allocation metadata 감소
* Cache line 활용 개선
* Index 연산만으로 parent/child 접근

Heap 삽입·삭제 과정은 배열 안에서 다소 멀리 떨어진 index를 오가지만, 개별 node가 무작위 주소에 흩어진 pointer tree보다는 일반적으로 메모리 배치가 조밀하다.

---

# 18. Index Overflow 주의

다음 계산은 overflow될 수 있다.

```c
size_t left = 2 * i + 1;
size_t right = 2 * i + 2;
```

`i`가 매우 크면 `size_t` 범위를 초과할 수 있다.

안전하게 검사하려면:

```c
#include <stdint.h>

int heap_left_child_index(size_t i, size_t *out_index)
{
    if (out_index == NULL) {
        return 0;
    }

    if (i > (SIZE_MAX - 1) / 2) {
        return 0;
    }

    *out_index = 2 * i + 1;
    return 1;
}
```

실제 heap은 메모리 한계 때문에 보통 이 크기에 도달하기 전에 allocation이 실패하지만, 시스템 코드에서는 산술 overflow 자체를 고려해야 한다.

---

# 19. 흔한 오해

## 오해 1: Heap은 완전히 정렬되어 있다

아니다.

Heap은 parent-child 순서만 보장한다.

```text
        1
       / \
    100   2
```

Valid Min Heap이다.

---

## 오해 2: Min Heap의 왼쪽 child가 오른쪽 child보다 작다

그런 조건은 없다.

```text
        1
       / \
      8   3
```

Valid Min Heap이다.

---

## 오해 3: Heap에서 모든 검색이 `O(log N)`이다

아니다.

* 최솟값·최댓값 접근: `O(1)`
* 삽입·root 제거: `O(log N)`
* 임의 값 검색: `O(N)`

---

## 오해 4: Heap은 BST다

아니다.

BST는 좌우 subtree 전체의 대소 관계를 유지한다.

Heap은 parent-child 관계만 유지한다.

---

## 오해 5: Heap은 pointer로 연결된 tree다

개념적으로는 binary tree지만 실제 구현은 대부분 배열이다.

---

## 오해 6: Complete와 Perfect는 같다

아니다.

Heap은 마지막 level이 완전히 차지 않아도 된다.

왼쪽부터 채워져 있으면 된다.

```text
        1
       / \
      3   2
     / \
    8   5
```

Perfect는 아니지만 Complete이고, valid Min Heap이 될 수 있다.

---

# 20. 반례와 실패 사례

## Order는 맞지만 shape가 잘못됨

```text
        1
       / \
      3   2
           \
            4
```

Parent-child 순서는 맞지만 complete가 아니다.

Heap이 아니다.

---

## Shape는 맞지만 order가 잘못됨

```text
        1
       / \
      8   2
     / \
    4   9
```

Complete이지만 `8 > 4`이므로 Min Heap property가 깨졌다.

Heap이 아니다.

---

## Root만 확인하고 Heap이라고 판단

```text
        1
       / \
      3   2
     / \
    0   8
```

Root는 전체에서 최소값처럼 보일 수 있지만 `3 ≤ 0`이 거짓이다.

Heap invariant는 모든 parent-child 관계에서 검사해야 한다.

---

# 21. 확인 문제

## 문제 1

다음은 Min Heap인가?

```text
        2
       / \
      4   3
     / \
    8   5
```

정답: 맞다.

* Complete Binary Tree
* 모든 parent가 child보다 작거나 같음

---

## 문제 2

다음은 Min Heap인가?

```text
        2
       / \
      5   3
     /
    4
```

정답: 아니다.

Shape는 complete지만:

```text
5 ≤ 4
```

가 거짓이다.

---

## 문제 3

다음은 Min Heap인가?

```text
        1
       / \
      2   3
           \
            4
```

정답: 아니다.

Order property는 만족하지만 complete가 아니다.

---

## 문제 4

Min Heap에서 두 번째로 작은 값은 반드시 root의 child 중 하나인가?

정답: 그렇다.

Root 다음으로 작은 값이 더 아래에 있다고 가정하면, 그 node의 parent는 그 값보다 작거나 같아야 한다. 그러면 그 parent가 두 번째로 작은 후보가 된다. 따라서 두 번째로 작은 값은 root의 child 중 하나에 존재한다.

단, 중복값 정책에 따라 “두 번째로 작은 서로 다른 값”은 별도로 정의해야 한다.

---

## 문제 5

Min Heap에서 최대값은 어디에 있는가?

정답: leaf 중 하나에 있다.

Internal node는 적어도 하나의 child를 가지며 parent는 child보다 작거나 같으므로, 최대값 후보는 아래쪽 leaf들에 존재한다.

하지만 어느 leaf인지는 알 수 없어서 모든 leaf를 확인해야 한다.

Leaf는 약 (N/2)개이므로:

[
O(N)
]

---

# 22. 실습 과제

## 실습 1: 배열을 Tree로 변환

다음 배열을 tree로 그려라.

```text
[2, 5, 4, 9, 7, 8, 6]
```

그리고 다음을 확인하라.

* 각 index의 parent
* 각 index의 left child
* 각 index의 right child
* Min Heap 여부

---

## 실습 2: Heap 검증 함수

다음 함수를 작성하라.

```c
int min_heap_is_valid(const MinHeap *heap);
```

검사 조건:

* `heap != NULL`
* `size <= capacity`
* `size > 0`이면 `data != NULL`
* 모든 child에 대해 `parent <= child`
* Index overflow 고려

기본 아이디어:

```c
for (size_t child = 1; child < heap->size; child++) {
    size_t parent = (child - 1) / 2;

    if (heap->data[parent] > heap->data[child]) {
        return 0;
    }
}
```

각 child를 한 번씩 확인하므로 시간은 `O(N)`이다.

---

## 실습 3: BST와 Heap 구분

다음 tree가 각각 무엇인지 판단하라.

```text
        4
       / \
      2   7
     / \ / \
    1  3 6  8
```

* Binary Tree: 맞음
* BST: 맞음
* Min Heap: 아님
* Max Heap: 아님

---

# 23. 핵심 정리

## Heap의 정의

[
\boxed{
Heap = Complete\ Binary\ Tree + Parent\text{-}Child\ Order
}
]

## Min Heap

[
parent \le children
]

Root가 최소값이다.

## Max Heap

[
parent \ge children
]

Root가 최대값이다.

## Heap과 BST

* BST: 좌우 subtree 전체에 ordering
* Heap: parent와 child 사이에만 ordering
* Heap에서 임의 검색은 `O(N)`

## 배열 표현

Heap은 Complete Binary Tree라서 배열에 빈틈없이 저장할 수 있다.

0-based index:

```text
parent = (i - 1) / 2
left   = 2i + 1
right  = 2i + 2
```

## 복잡도

| 연산           |         시간 |
| ------------ | ---------: |
| Peek         |     `O(1)` |
| Insert       | `O(log N)` |
| Extract root | `O(log N)` |
| 임의 값 검색      |     `O(N)` |
| Heap 검증      |     `O(N)` |

---

# 24. 면접 대비 핵심 문장

> Heap은 Complete Binary Tree의 shape invariant와 parent-child 간의 heap-order invariant를 동시에 유지하는 자료구조입니다.

> Min Heap에서는 모든 parent가 child보다 작거나 같기 때문에 root가 전체 최소값입니다. 하지만 형제나 서로 다른 subtree 사이에는 정렬 관계가 없습니다.

> Heap은 Complete Binary Tree이므로 높이가 `O(log N)`으로 유지되고, 배열에 빈틈없이 저장할 수 있습니다.

> Heap은 BST가 아닙니다. BST는 왼쪽과 오른쪽 subtree 전체의 순서를 보장하지만, Heap은 parent와 child 사이의 순서만 보장합니다.

> Heap은 최소값이나 최대값을 반복적으로 꺼내는 Priority Queue에 적합하지만, 임의 값 검색에는 적합하지 않아 최악 `O(N)`이 필요합니다.
