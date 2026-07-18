# Lecture 8. Classic Patterns

## Reverse Linked List, Binary Search, Floyd Cycle Detection

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **자료구조를 이해했다면, 그 위에서 반복적으로 등장하는 알고리즘 패턴을 어떻게 알아볼 것인가?**

자료구조 공부는 단순히 Array, List, Stack, Queue 이름을 외우는 것으로 끝나지 않는다.

실전에서는 이런 식으로 문제가 나온다.

```text
"이 linked list를 뒤집어라."
"정렬된 배열에서 빠르게 찾아라."
"linked list에 cycle이 있는지 판단해라."
"stack 두 개로 queue를 만들어라."
```

이번 강의에서는 자료에 나온 classic algorithm 항목 중 핵심 세 가지를 다룬다.

| 패턴                    | 핵심 자료구조      | 핵심 아이디어        |
| --------------------- | ------------ | -------------- |
| Reverse Linked List   | Linked List  | 포인터 방향 뒤집기     |
| Binary Search         | Sorted Array | 탐색 범위 절반씩 제거   |
| Floyd Cycle Detection | Linked List  | 빠른 포인터와 느린 포인터 |

자료에도 reverse linked list, binary search requirement, Floyd’s algorithm 등이 interview practice 항목으로 제시되어 있습니다. 

---

# 2. 형식적 정의

## 2.1 Reverse Linked List

**Formal definition**

Singly linked list가 다음과 같이 주어졌다고 하자.

```text
L = n₁ -> n₂ -> n₃ -> ... -> nₖ -> NULL
```

Reverse linked list는 이를 다음 형태로 바꾸는 연산이다.

```text
reverse(L) = nₖ -> nₖ₋₁ -> ... -> n₃ -> n₂ -> n₁ -> NULL
```

즉 모든 `next` 포인터의 방향을 반대로 바꾼다.

---

## 2.2 Binary Search

**Formal definition**

Binary search는 **정렬된 배열** `A[0...n-1]`에서 target `x`를 찾는 알고리즘이다.

전제 조건:

```text
A는 정렬되어 있어야 한다.
```

각 단계에서 가운데 원소 `mid`를 확인한다.

```text
A[mid] == x  → 찾음
A[mid] < x   → 오른쪽 절반 탐색
A[mid] > x   → 왼쪽 절반 탐색
```

시간 복잡도:

```text
O(log N)
```

자료에서도 binary search의 prerequisite은 **array must be sorted**이며, 시간 복잡도는 O(log N), 공간 복잡도는 O(1)이라고 정리되어 있습니다. 

---

## 2.3 Floyd Cycle Detection

**Formal definition**

Floyd’s cycle detection algorithm은 linked list에서 cycle이 존재하는지 판단하는 알고리즘이다.

두 포인터를 사용한다.

```text
slow: 한 번에 1칸 이동
fast: 한 번에 2칸 이동
```

cycle이 있다면 언젠가 두 포인터는 만난다.

cycle이 없다면 `fast`가 `NULL`에 도달한다.

---

# 3. 직관적 설명

## 3.1 Reverse Linked List 직관

현재 리스트:

```text
1 -> 2 -> 3 -> NULL
```

각 화살표를 반대로 뒤집고 싶다.

```text
1 <- 2 <- 3
```

그러면 새로운 head는 `3`이 된다.

최종:

```text
3 -> 2 -> 1 -> NULL
```

문제는 단순히 `curr->next = prev`를 하면 원래 다음 노드를 잃어버릴 수 있다는 점이다.

그래서 세 포인터가 필요하다.

```text
prev
curr
next
```

자료에도 reverse linked list는 `prev`, `curr`, `next` 세 포인터를 사용한다고 되어 있습니다. 

---

## 3.2 Binary Search 직관

정렬된 책장에서 특정 번호의 책을 찾는다고 하자.

```text
[1, 3, 5, 7, 9, 11, 13]
```

target이 `11`이면 가운데를 먼저 본다.

```text
mid = 7
```

`11`은 `7`보다 크다.

그러면 왼쪽 절반은 볼 필요가 없다.

```text
[9, 11, 13]
```

다시 가운데를 본다.

```text
11
```

찾았다.

핵심은:

```text
한 번 비교할 때마다 후보 영역이 절반으로 줄어든다.
```

---

## 3.3 Floyd Cycle Detection 직관

트랙을 도는 두 사람이 있다고 하자.

```text
slow: 천천히 감
fast: 두 배 빠르게 감
```

트랙이 원형이면 빠른 사람은 언젠가 느린 사람을 따라잡는다.

하지만 길이 직선이고 끝이 있다면, 빠른 사람은 끝에 먼저 도달한다.

Linked list cycle도 같다.

Cycle 있음:

```text
1 -> 2 -> 3 -> 4
          ^    |
          |____|
```

Cycle 없음:

```text
1 -> 2 -> 3 -> 4 -> NULL
```

---

# 4. 왜 필요한지

이 세 패턴은 면접용 문제가 아니다. 실제 시스템에도 나온다.

| 패턴                    | 실제 의미                              |
| --------------------- | ---------------------------------- |
| Reverse Linked List   | 포인터 조작, ownership 변경, list 재구성     |
| Binary Search         | 정렬된 데이터에서 빠른 탐색                    |
| Floyd Cycle Detection | 잘못 연결된 구조, cycle, infinite loop 방지 |

예를 들어 C/C++에서 linked list를 잘못 연결하면 순회가 끝나지 않을 수 있다.

```text
A -> B -> C -> B -> C -> B -> ...
```

이런 버그는 프로그램을 무한 루프에 빠뜨린다.

Cycle detection은 이런 구조적 오류를 찾는 데 중요하다.

---

# 5. 내부 원리

## 5.1 Reverse Linked List 내부 원리

현재 상태:

```text
prev = NULL
curr = head
```

리스트:

```text
NULL    1 -> 2 -> 3 -> NULL
prev    curr
```

반복마다 다음 세 단계를 수행한다.

```text
1. next = curr->next
2. curr->next = prev
3. prev = curr
4. curr = next
```

왜 `next`를 먼저 저장해야 하는가?

`curr->next = prev`를 하는 순간, 원래 다음 노드로 가는 길이 끊어진다.

따라서 먼저 저장한다.

```c
next = curr->next;
```

그다음 뒤집는다.

```c
curr->next = prev;
```

그리고 한 칸 전진한다.

```c
prev = curr;
curr = next;
```

---

## 5.2 Binary Search 내부 원리

Binary search는 다음 invariant를 유지한다.

```text
target이 존재한다면 항상 [left, right] 범위 안에 있다.
```

초기 상태:

```text
left = 0
right = n - 1
```

반복:

```text
mid = left + (right - left) / 2
```

`A[mid] < target`이면:

```text
left = mid + 1
```

왜?

정렬되어 있으므로 `mid` 이하 원소는 모두 target보다 작거나 같다.

`A[mid] > target`이면:

```text
right = mid - 1
```

왜?

정렬되어 있으므로 `mid` 이상 원소는 target보다 크거나 같다.

이렇게 탐색 범위가 매번 줄어든다.

---

## 5.3 Floyd Cycle Detection 내부 원리

반복마다:

```text
slow = slow->next
fast = fast->next->next
```

만약 cycle이 있다면, cycle 내부에서 fast는 slow보다 매번 1칸씩 가까워진다.

왜냐하면:

```text
fast는 2칸
slow는 1칸
상대 속도는 1칸
```

원형 공간에서 상대 속도가 1이면 언젠가 같은 위치가 된다.

cycle이 없다면 `fast` 또는 `fast->next`가 `NULL`이 된다.

---

# 6. 단계별 예시

## 예시 1. Reverse Linked List

초기 리스트:

```text
1 -> 2 -> 3 -> NULL
```

초기 포인터:

```text
prev = NULL
curr = 1
```

---

### Step 1

```text
next = 2
curr->next = prev
```

결과:

```text
NULL <- 1    2 -> 3 -> NULL
prev   curr  next
```

전진:

```text
prev = 1
curr = 2
```

---

### Step 2

```text
next = 3
curr->next = prev
```

결과:

```text
NULL <- 1 <- 2    3 -> NULL
            curr  next
```

전진:

```text
prev = 2
curr = 3
```

---

### Step 3

```text
next = NULL
curr->next = prev
```

결과:

```text
NULL <- 1 <- 2 <- 3
```

전진:

```text
prev = 3
curr = NULL
```

반복 종료.

새 head는 `prev`.

```text
3 -> 2 -> 1 -> NULL
```

### C 코드

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;

Node *reverse(Node *head) {
    Node *prev = NULL;
    Node *curr = head;
    Node *next = NULL;

    while (curr != NULL) {
        next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }

    return prev;
}
```

복잡도:

```text
Time: O(N)
Space: O(1)
```

자료의 iterative reverse 코드도 이와 같은 세 포인터 구조입니다. 

---

## 예시 2. Binary Search

배열:

```text
index:  0   1   2   3   4   5   6
value: [2] [5] [8] [12][16][23][38]
```

target:

```text
23
```

초기:

```text
left = 0
right = 6
```

### Step 1

```text
mid = 0 + (6 - 0) / 2 = 3
A[3] = 12
```

`12 < 23`

오른쪽 탐색:

```text
left = mid + 1 = 4
right = 6
```

---

### Step 2

```text
mid = 4 + (6 - 4) / 2 = 5
A[5] = 23
```

찾았다.

### C 코드

```c
int binary_search(int arr[], int n, int target) {
    int left = 0;
    int right = n - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }

    return -1;
}
```

복잡도:

```text
Time: O(log N)
Space: O(1)
```

---

## 예시 3. Floyd Cycle Detection

리스트:

```text
1 -> 2 -> 3 -> 4 -> 5
          ^         |
          |_________|
```

즉 `5->next = 3`.

초기:

```text
slow = 1
fast = 1
```

| 단계 | slow | fast |
| -: | ---: | ---: |
|  0 |    1 |    1 |
|  1 |    2 |    3 |
|  2 |    3 |    5 |
|  3 |    4 |    4 |

slow와 fast가 만났다.

따라서 cycle이 있다.

### C 코드

```c
int has_cycle(Node *head) {
    Node *slow = head;
    Node *fast = head;

    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;

        if (slow == fast) {
            return 1;
        }
    }

    return 0;
}
```

복잡도:

```text
Time: O(N)
Space: O(1)
```

---

# 7. 실제 응용

## 7.1 Reverse Linked List 응용

Reverse linked list는 다음 상황에서 쓰인다.

```text
- list 기반 stack 재구성
- undo history 방향 변경
- persistent list 처리
- low-level memory block list 조작
- interview pointer manipulation 훈련
```

하지만 더 중요한 것은 “리스트 뒤집기” 자체보다, 다음 능력이다.

```text
포인터를 잃지 않고 구조를 바꿀 수 있는가?
```

C/C++ 시스템 프로그래밍에서 이 감각이 중요하다.

---

## 7.2 Binary Search 응용

Binary search는 단순히 배열에서 값을 찾는 데만 쓰지 않는다.

실제로는 다음 패턴으로 확장된다.

```text
- lower_bound
- upper_bound
- answer space binary search
- parameter search
- sorted log lookup
- time-series index lookup
```

예:

```text
"가능한 최소 용량은 얼마인가?"
"조건을 만족하는 가장 작은 x는?"
"처음으로 실패하는 버전은?"
```

이런 문제는 값 자체를 배열에서 찾는 것이 아니라, **정답 범위**를 이분 탐색한다.

---

## 7.3 Floyd Cycle Detection 응용

Cycle detection은 다음에 쓰인다.

```text
- linked list cycle bug 탐지
- graph-like pointer structure 검사
- memory allocator free list 검증
- 반복 함수 f(x)의 주기 탐지
- infinite loop 가능성 확인
```

예를 들어 free list가 cycle을 가지면 allocator가 무한 루프에 빠질 수 있다.

```text
free block A -> free block B -> free block C -> free block B ...
```

이런 구조는 시스템 레벨에서 치명적이다.

---

# 8. 흔한 오해

## 오해 1. “Reverse linked list는 값을 바꾸는 것이다”

아니다.

값을 바꾸는 게 아니라 **포인터 방향을 바꾸는 것**이다.

잘못된 접근:

```text
1, 2, 3의 value만 swap한다.
```

이건 일부 경우에는 결과가 같아 보일 수 있지만, list 구조를 reverse한 것이 아니다.

정확한 reverse는:

```text
node들의 next 관계를 반대로 만든다.
```

---

## 오해 2. “Binary search는 아무 배열에나 쓸 수 있다”

아니다.

전제 조건은 정렬이다.

```text
[10, 3, 7, 1, 9]
```

이런 배열에서는 binary search를 쓰면 안 된다.

자료에서도 binary search의 prerequisite은 배열이 반드시 sorted여야 한다고 명시합니다. 

---

## 오해 3. “Binary search는 mid = (left + right) / 2면 충분하다”

작은 입력에서는 괜찮다.

하지만 큰 정수에서는 overflow 가능성이 있다.

위험한 코드:

```c
int mid = (left + right) / 2;
```

더 안전한 코드:

```c
int mid = left + (right - left) / 2;
```

---

## 오해 4. “Cycle detection은 visited set이 있어야만 가능하다”

visited set을 쓰는 방법도 있다.

```text
방문한 node 주소를 hash set에 저장
```

하지만 공간이 O(N)이다.

Floyd algorithm은 추가 공간 O(1)로 cycle을 찾는다.

---

# 9. 반례 또는 실패 사례

## 실패 사례 1. Reverse에서 next를 저장하지 않음

잘못된 코드:

```c
while (curr != NULL) {
    curr->next = prev;
    prev = curr;
    curr = curr->next;
}
```

문제:

```text
curr->next = prev를 하는 순간 원래 다음 노드 주소를 잃어버린다.
```

예:

```text
1 -> 2 -> 3
```

`1->next = NULL`로 바꾸면, `2`로 갈 방법이 사라진다.

그래서 반드시 먼저 저장해야 한다.

```c
next = curr->next;
```

---

## 실패 사례 2. Binary search에서 무한 루프

잘못된 코드:

```c
while (left < right) {
    int mid = (left + right) / 2;

    if (arr[mid] < target) {
        left = mid;
    } else {
        right = mid;
    }
}
```

문제:

```text
left와 mid가 같아지는 순간 left가 증가하지 않는다.
```

예:

```text
left = 3, right = 4
mid = 3
left = mid = 3
```

무한 루프 가능.

일반 search에서는 다음처럼 확실히 범위를 줄인다.

```c
left = mid + 1;
right = mid - 1;
```

단, lower_bound 스타일에서는 loop invariant를 다르게 설계해야 한다.

---

## 실패 사례 3. Floyd에서 fast->next 체크 없이 접근

잘못된 코드:

```c
while (fast != NULL) {
    slow = slow->next;
    fast = fast->next->next;
}
```

문제:

```text
fast->next가 NULL일 수 있다.
```

그러면 `fast->next->next`에서 null pointer dereference가 발생한다.

올바른 조건:

```c
while (fast != NULL && fast->next != NULL)
```

---

# 10. 확인 문제

## 문제 1

다음 linked list를 reverse하면?

```text
10 -> 20 -> 30 -> NULL
```

정답:

```text
30 -> 20 -> 10 -> NULL
```

---

## 문제 2

Reverse linked list에서 세 포인터 `prev`, `curr`, `next` 중 `next`가 필요한 이유는?

정답:

```text
curr->next를 바꾸기 전에 원래 다음 노드의 주소를 보존하기 위해서다.
```

---

## 문제 3

Binary search를 사용할 수 없는 배열은?

```text
A = [1, 3, 5, 7, 9]
B = [9, 1, 5, 3, 7]
```

정답:

```text
B
```

이유:

```text
정렬되어 있지 않기 때문이다.
```

---

## 문제 4

Binary search의 시간 복잡도가 O(log N)인 이유는?

정답:

```text
매 단계마다 탐색 범위를 절반으로 줄이기 때문이다.
```

---

## 문제 5

Floyd cycle detection에서 fast pointer는 몇 칸씩 이동하는가?

정답:

```text
2칸
```

slow pointer는 1칸씩 이동한다.

---

# 11. 실습 과제

## 과제 1. Reverse Linked List 직접 구현

다음 구조체를 사용해라.

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;
```

구현:

```c
Node *reverse(Node *head);
```

테스트 케이스:

```text
empty list
single node
1 -> 2
1 -> 2 -> 3 -> 4
```

---

## 과제 2. Binary Search 직접 구현

구현:

```c
int binary_search(int arr[], int n, int target);
```

테스트 케이스:

```text
[]에서 찾기
[5]에서 5 찾기
[5]에서 3 찾기
[1,2,3,4,5]에서 1 찾기
[1,2,3,4,5]에서 5 찾기
[1,2,3,4,5]에서 6 찾기
```

---

## 과제 3. Floyd Cycle Detection 구현

구현:

```c
int has_cycle(Node *head);
```

테스트 케이스:

```text
empty list
single node no cycle
single node points to itself
1 -> 2 -> 3 -> NULL
1 -> 2 -> 3 -> 2 ...
```

---

## 과제 4. Binary Search Invariant 설명하기

다음 문장을 완성해라.

```text
반복문이 시작될 때마다 target이 존재한다면,
target은 항상 ______ 범위 안에 있다.
```

정답:

```text
[left, right]
```

이 invariant가 깨지면 binary search는 잘못된 결과를 낸다.

---

# 12. 핵심 정리

```text
Classic pattern은 “외운 코드”가 아니라,
자료구조의 제약을 이용하는 반복 가능한 사고방식이다.
```

| 알고리즘                  | 핵심 아이디어                     | 복잡도            |
| --------------------- | --------------------------- | -------------- |
| Reverse Linked List   | next 포인터 방향 뒤집기             | O(N), O(1)     |
| Binary Search         | 정렬된 배열에서 절반씩 제거             | O(log N), O(1) |
| Floyd Cycle Detection | slow/fast pointer로 cycle 탐지 | O(N), O(1)     |

## 반드시 기억할 문장

> Reverse는 포인터를 잃지 않는 훈련이고, Binary Search는 invariant를 유지하는 훈련이며, Floyd는 공간 없이 구조적 오류를 찾는 훈련이다.