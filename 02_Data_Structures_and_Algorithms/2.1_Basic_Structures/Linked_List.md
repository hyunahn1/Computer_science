# Lecture 3. Linked List — 포인터 기반 구조

## 1. 핵심 질문

**Linked List는 왜 배열과 다르게 동작하는가?**

배열은 메모리에 원소들이 붙어 있다.

```text
[10][20][30][40][50]
```

Linked List는 원소들이 메모리 여기저기에 흩어져 있고, 각 원소가 **다음 원소의 주소**를 들고 있다.

```text
[10 | next] -> [20 | next] -> [30 | next] -> NULL
```

핵심 질문은 이것이다.

> “데이터를 연속된 메모리에 둘 것인가, 아니면 포인터로 연결할 것인가?”

자료에서도 array는 **contiguous memory**, linked list는 **non-contiguous memory**라고 구분합니다. 또한 array는 index access가 O(1), linked list는 traversal이 필요해 O(N)이라고 정리되어 있습니다. 

---

## 2. 형식적 정의

### Singly Linked List

**Formal definition**

Singly linked list는 다음과 같은 노드들의 유한한 sequence다.

```text
L = node₁ -> node₂ -> ... -> nodeₙ -> NULL
```

각 node는 보통 두 부분으로 구성된다.

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;
```

즉 하나의 노드는 다음 정보를 가진다.

| 필드      | 의미        |
| ------- | --------- |
| `value` | 저장할 데이터   |
| `next`  | 다음 노드의 주소 |

리스트 전체는 보통 `head` 포인터로 시작한다.

```c
Node *head;
```

`head`는 첫 번째 노드를 가리킨다.

```text
head
 |
 v
[10 | next] -> [20 | next] -> [30 | NULL]
```

빈 리스트는 다음과 같이 표현된다.

```c
head = NULL;
```

---

## 3. 직관적 설명

배열은 “번호가 붙은 방”이다.

```text
Room 0 | Room 1 | Room 2 | Room 3
```

그래서 `arr[3]`으로 바로 갈 수 있다.

Linked list는 “보물찾기 지도”에 가깝다.

```text
첫 번째 종이: 값은 10, 다음 종이는 주소 A
두 번째 종이: 값은 20, 다음 종이는 주소 B
세 번째 종이: 값은 30, 다음은 없음
```

즉 3번째 원소를 찾으려면:

```text
head -> next -> next
```

이렇게 따라가야 한다.

그래서 linked list는 index 접근이 느리다.

```text
get(index) = O(N)
```

반면 이미 어떤 노드의 위치를 알고 있다면, 그 뒤에 새 노드를 끼우는 것은 포인터 몇 개만 바꾸면 된다.

```text
A -> B -> D
```

B 뒤에 C 삽입:

```text
A -> B -> C -> D
```

---

## 4. 왜 필요한지

Linked list는 배열보다 일반적으로 느린 경우가 많다. 그런데도 배우는 이유가 있다.

### 이유 1. 포인터와 메모리 구조를 이해하는 핵심 자료구조

C/C++에서 linked list는 다음 개념들을 강제로 이해하게 만든다.

```text
- pointer
- dynamic allocation
- ownership
- memory leak
- dangling pointer
- NULL handling
- edge case
```

즉 linked list는 단순한 자료구조가 아니라, **메모리 사고 훈련 도구**다.

---

### 이유 2. 특정 상황에서는 삽입/삭제가 유리하다

배열에서 중간 삽입은 뒤 원소를 밀어야 한다.

```text
[10][20][30][40]
     ↑ 여기에 99 삽입

[10][99][20][30][40]
```

O(N).

Linked list는 삽입 위치를 이미 알고 있다면 포인터만 바꾸면 된다.

```text
prev -> curr
```

새 노드 삽입:

```text
prev -> new -> curr
```

O(1).

자료에서도 linked list는 position을 알고 있을 경우 삭제가 O(1)에 가능하다고 정리합니다. 다만 position을 찾는 과정은 O(N)입니다. 

---

### 이유 3. 다른 자료구조의 기반이 된다

Linked list 개념은 다음 구조를 이해할 때 필요하다.

| 구조                  | linked list 개념이 쓰이는 방식        |
| ------------------- | ----------------------------- |
| adjacency list      | graph 표현                      |
| hash table chaining | collision 처리                  |
| LRU cache           | doubly linked list + hash map |
| memory allocator    | free list                     |
| OS scheduler        | ready queue                   |
| undo/redo system    | stack-like linked structure   |

---

## 5. 내부 원리

Linked list의 본질은 **노드 단위 동적 할당 + 포인터 연결**이다.

### 5.1 노드 생성

```c
Node *node = malloc(sizeof(Node));
node->value = 10;
node->next = NULL;
```

메모리 어딘가에 노드 하나가 생긴다.

```text
주소 0x1000:
[ value: 10 | next: NULL ]
```

---

### 5.2 노드 연결

```c
Node *a = malloc(sizeof(Node));
Node *b = malloc(sizeof(Node));

a->value = 10;
b->value = 20;

a->next = b;
b->next = NULL;
```

구조:

```text
a
|
v
[10 | next] ---> [20 | NULL]
```

여기서 중요한 점:

```text
a와 b가 메모리상 바로 옆에 있다는 보장은 없다.
```

예를 들어 실제 주소는 이럴 수 있다.

```text
a: 0x1000
b: 0xA3F0
```

그래서 CPU cache 관점에서는 비효율적일 수 있다.

자료에서도 linked list는 pointer per node 때문에 memory overhead가 있고, cache performance가 poor하다고 비교합니다. 

---

### 5.3 traversal

Linked list는 반드시 `head`부터 따라간다.

```c
void print_list(Node *head) {
    Node *curr = head;

    while (curr != NULL) {
        printf("%d\n", curr->value);
        curr = curr->next;
    }
}
```

흐름:

```text
curr = head
print curr->value
curr = curr->next
repeat until NULL
```

복잡도:

```text
Time: O(N)
Space: O(1)
```

---

## 6. 단계별 예시

## 예시 1. 맨 앞에 삽입하기

현재 리스트:

```text
head
 |
 v
[20] -> [30] -> NULL
```

여기에 `10`을 앞에 넣고 싶다.

### Step 1. 새 노드 생성

```c
Node *new_node = malloc(sizeof(Node));
new_node->value = 10;
```

```text
[10 | ?]
```

### Step 2. 새 노드가 기존 head를 가리키게 한다

```c
new_node->next = head;
```

```text
[10] -> [20] -> [30] -> NULL
```

### Step 3. head를 새 노드로 바꾼다

```c
head = new_node;
```

최종:

```text
head
 |
 v
[10] -> [20] -> [30] -> NULL
```

전체 코드:

```c
Node *push_front(Node *head, int value) {
    Node *new_node = malloc(sizeof(Node));
    if (new_node == NULL) {
        return head;
    }

    new_node->value = value;
    new_node->next = head;

    return new_node;
}
```

복잡도:

```text
Time: O(1)
Space: O(1)
```

단, 새 노드 하나의 메모리는 추가로 사용한다.

---

## 예시 2. 특정 값 찾기

```c
Node *find(Node *head, int target) {
    Node *curr = head;

    while (curr != NULL) {
        if (curr->value == target) {
            return curr;
        }
        curr = curr->next;
    }

    return NULL;
}
```

리스트:

```text
[10] -> [20] -> [30] -> [40] -> NULL
```

`30`을 찾는 과정:

```text
1. 10 확인
2. 20 확인
3. 30 확인
4. 찾음
```

복잡도:

```text
Best case: O(1)
Worst case: O(N)
Space: O(1)
```

---

## 예시 3. 중간 노드 삭제

리스트:

```text
prev       curr
 |          |
 v          v
[10] ---> [20] ---> [30] ---> NULL
```

`curr`, 즉 20을 삭제하고 싶다.

핵심은 이 한 줄이다.

```c
prev->next = curr->next;
```

그러면:

```text
[10] ----------> [30] ---> NULL
          [20] disconnected
```

그다음 메모리 해제:

```c
free(curr);
```

전체:

```c
void delete_after(Node *prev) {
    if (prev == NULL || prev->next == NULL) {
        return;
    }

    Node *target = prev->next;
    prev->next = target->next;
    free(target);
}
```

복잡도:

```text
Time: O(1)
```

하지만 주의:

```text
prev를 이미 알고 있을 때만 O(1)
```

만약 삭제할 값을 먼저 찾아야 한다면 전체는 O(N)이다.

---

## 7. 실제 응용

## 7.1 Hash Table Collision Chaining

해시 테이블에서 서로 다른 key가 같은 bucket으로 들어갈 수 있다.

```text
hash("A") = 3
hash("B") = 3
```

이때 bucket 3에 linked list를 둘 수 있다.

```text
bucket[3] -> ["A"] -> ["B"] -> NULL
```

이를 separate chaining이라고 한다.

구조:

```c
typedef struct Entry {
    char *key;
    int value;
    struct Entry *next;
} Entry;

Entry *table[BUCKET_SIZE];
```

---

## 7.2 Graph Adjacency List

그래프에서 각 정점의 이웃을 저장할 때 linked list 개념을 사용할 수 있다.

```text
0: 1 -> 2 -> 5
1: 0 -> 3
2: 0 -> 4
```

C++에서는 보통 `vector<vector<int>>`를 더 많이 쓰지만, 개념적으로는 adjacency list다.

```cpp
std::vector<std::vector<int>> graph;
```

---

## 7.3 LRU Cache

LRU cache는 다음 두 구조를 결합한다.

```text
Hash map + Doubly linked list
```

Hash map:

```text
key -> node pointer
```

Doubly linked list:

```text
most recent <-> ... <-> least recent
```

왜 doubly linked list가 필요한가?

```text
- 중간 노드를 O(1)에 제거
- 맨 앞으로 O(1)에 이동
```

Hash map만 있으면 빠르게 찾을 수 있지만, 사용 순서를 관리하기 어렵다.

Linked list만 있으면 순서는 관리되지만, key 검색이 O(N)이다.

둘을 합치면:

```text
get(key): O(1)
put(key, value): O(1)
evict least recent: O(1)
```

---

## 7.4 Memory Allocator의 Free List

메모리 할당기는 사용 가능한 메모리 블록들을 linked list처럼 관리할 수 있다.

```text
free block -> free block -> free block -> NULL
```

이런 구조를 free list라고 한다.

실무적으로 linked list는 애플리케이션 로직보다 **시스템 내부 구현**에서 더 자주 등장한다.

---

## 8. 흔한 오해

## 오해 1. “Linked list는 중간 삽입/삭제가 항상 O(1)이다”

정확히는:

```text
삽입/삭제할 위치를 이미 알고 있으면 O(1)
위치를 찾아야 하면 O(N)
```

예:

```text
delete value 500
```

이 경우는 500이 어디 있는지 찾아야 한다.

```text
head부터 탐색 -> O(N)
삭제 자체 -> O(1)
전체 -> O(N)
```

---

## 오해 2. “Linked list는 배열보다 메모리를 아낀다”

아니다.

Linked list는 각 노드마다 포인터를 추가로 저장한다.

예를 들어 64-bit 시스템에서:

```c
typedef struct Node {
    int value;          // 4 bytes
    struct Node *next;  // 8 bytes
} Node;
```

단순히 보면 12바이트지만, alignment 때문에 실제로는 16바이트가 될 수 있다.

즉 `int` 하나 저장하려고 16바이트를 쓸 수 있다.

배열은:

```c
int arr[1000];
```

원소 하나당 4바이트만 쓴다.

---

## 오해 3. “Linked list는 동적이라 항상 좋다”

동적이라는 말은 장점이면서 단점이다.

장점:

```text
필요할 때 노드를 하나씩 만들 수 있다.
```

단점:

```text
malloc/free 비용
memory fragmentation
cache locality 악화
pointer bug 가능성
```

Embedded system에서는 특히 문제가 된다.

자료에서도 embedded systems에서는 RAM이 제한적이고 memory fragmentation concerns가 중요하다고 언급합니다. 

---

## 9. 반례 또는 실패 사례

## 실패 사례 1. head를 갱신하지 않음

잘못된 코드:

```c
void push_front_wrong(Node *head, int value) {
    Node *new_node = malloc(sizeof(Node));
    new_node->value = value;
    new_node->next = head;
    head = new_node;
}
```

문제:

```text
head는 함수 안의 지역 변수다.
```

함수 밖의 head는 바뀌지 않는다.

수정 방법 1: 새 head 반환

```c
Node *push_front(Node *head, int value) {
    Node *new_node = malloc(sizeof(Node));
    if (new_node == NULL) {
        return head;
    }

    new_node->value = value;
    new_node->next = head;

    return new_node;
}
```

사용:

```c
head = push_front(head, 10);
```

수정 방법 2: 이중 포인터 사용

```c
void push_front(Node **head, int value) {
    Node *new_node = malloc(sizeof(Node));
    if (new_node == NULL) {
        return;
    }

    new_node->value = value;
    new_node->next = *head;
    *head = new_node;
}
```

사용:

```c
push_front(&head, 10);
```

---

## 실패 사례 2. free 후 접근

```c
Node *target = head;
head = head->next;
free(target);

printf("%d\n", target->value); // 잘못됨
```

`free(target)` 이후 `target`이 가리키던 메모리는 더 이상 유효하지 않다.

이후 접근은 use-after-free다.

이런 버그는 C/C++에서 매우 위험하다.

---

## 실패 사례 3. next를 잃어버림

잘못된 삭제 코드:

```c
free(curr);
prev->next = curr->next;
```

문제:

```text
curr를 free한 뒤 curr->next에 접근한다.
```

올바른 순서:

```c
prev->next = curr->next;
free(curr);
```

또는 더 명확하게:

```c
Node *next = curr->next;
prev->next = next;
free(curr);
```

---

## 10. 확인 문제

### 문제 1

Linked list에서 `head`부터 100번째 노드에 접근하는 시간 복잡도는?

정답:

```text
O(N)
```

이유:

```text
주소 계산으로 바로 갈 수 없고, next pointer를 따라가야 한다.
```

---

### 문제 2

다음 상황에서 삭제의 전체 시간 복잡도는?

```text
값이 42인 노드를 삭제한다.
단, 그 노드의 위치는 모른다.
```

정답:

```text
O(N)
```

이유:

```text
삭제 자체는 O(1)이지만, 42를 찾는 데 O(N)이 걸린다.
```

---

### 문제 3

다음 상황에서 삭제의 시간 복잡도는?

```text
삭제할 노드의 이전 노드 prev를 이미 알고 있다.
```

정답:

```text
O(1)
```

이유:

```text
prev->next만 바꾸면 된다.
```

---

### 문제 4

Linked list가 array보다 실제로 느릴 수 있는 중요한 하드웨어적 이유는?

정답:

```text
cache locality가 나쁘기 때문이다.
```

---

## 11. 실습 과제

## 과제 1. Linked list 순회 구현

```c
void print_list(Node *head);
```

예시:

```c
void print_list(Node *head) {
    Node *curr = head;

    while (curr != NULL) {
        printf("%d\n", curr->value);
        curr = curr->next;
    }
}
```

---

## 과제 2. 리스트 길이 구하기

```c
int length(Node *head);
```

예시:

```c
int length(Node *head) {
    int count = 0;
    Node *curr = head;

    while (curr != NULL) {
        count++;
        curr = curr->next;
    }

    return count;
}
```

분석:

```text
Time: O(N)
Space: O(1)
```

---

## 과제 3. 값으로 노드 찾기

```c
Node *find(Node *head, int target);
```

예시:

```c
Node *find(Node *head, int target) {
    Node *curr = head;

    while (curr != NULL) {
        if (curr->value == target) {
            return curr;
        }
        curr = curr->next;
    }

    return NULL;
}
```

---

## 과제 4. 전체 리스트 해제하기

```c
void free_list(Node *head);
```

중요한 점:

```text
free하기 전에 next를 저장해야 한다.
```

예시:

```c
void free_list(Node *head) {
    Node *curr = head;

    while (curr != NULL) {
        Node *next = curr->next;
        free(curr);
        curr = next;
    }
}
```

잘못된 코드:

```c
while (curr != NULL) {
    free(curr);
    curr = curr->next; // use-after-free
}
```

---

## 12. 핵심 정리

```text
Linked List의 본질은 “연속 메모리”가 아니라 “포인터 연결”이다.
```

| 항목                | Linked List                                   |
| ----------------- | --------------------------------------------- |
| 메모리 배치            | 비연속적                                          |
| index access      | O(N)                                          |
| search            | O(N)                                          |
| head 삽입           | O(1)                                          |
| 위치를 알고 있을 때 삽입/삭제 | O(1)                                          |
| 위치를 모를 때 삽입/삭제    | O(N)                                          |
| 메모리 오버헤드          | node마다 pointer 필요                             |
| cache locality    | 나쁨                                            |
| 주요 위험             | memory leak, use-after-free, dangling pointer |

## 이번 강의에서 반드시 잡아야 하는 문장

> Linked list는 삽입/삭제가 빠른 자료구조가 아니라, **이미 위치를 알고 있을 때 포인터 변경이 빠른 자료구조**다.
