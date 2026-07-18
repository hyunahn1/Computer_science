# Part 2.1 — Basic Data Structures: Arrays & Lists

## 전체 커리큘럼

### Lecture 1. 자료구조를 왜 배우는가: Selection Criteria & Complexity

* 자료구조 선택의 3요소
* Big-O의 의미
* 시간 복잡도와 공간 복잡도
* “빠른 코드”와 “확장 가능한 코드”의 차이

### Lecture 2. Array: 연속 메모리와 Random Access

* 배열의 형식적 정의
* 인덱스 접근 O(1)
* 삽입/삭제 O(N)
* Cache locality

### Lecture 3. Linked List: 포인터 기반 구조

* Node, pointer, head
* 순차 접근 O(N)
* 삽입/삭제의 진짜 의미
* 메모리 오버헤드와 cache miss

### Lecture 4. Dynamic Array / Vector

* size vs capacity
* resizing
* amortized O(1)
* 왜 대부분의 경우 linked list보다 vector가 빠른가

### Lecture 5. Stack, Queue, Deque

* LIFO / FIFO
* 추상 자료형과 구현 자료구조의 차이
* 함수 호출 스택, BFS, task scheduling

### Lecture 6. Circular Queue / Ring Buffer

* front/rear 문제
* full vs empty 구분
* embedded system, UART, producer-consumer

### Lecture 7. Sparse Matrix & Hash Table Intro

* 희소 행렬 저장 방식
* key-value mapping
* hash collision과 load factor

### Lecture 8. Classic Patterns

* Reverse linked list
* Binary search
* Floyd cycle detection
* stack using queues

### 최종 목표

1. 자료구조를 “외워서” 고르는 것이 아니라, **access pattern / memory / complexity** 기준으로 선택한다.
2. Array, linked list, vector, stack, queue, deque, ring buffer의 **내부 동작 원리**를 설명할 수 있다.
3. C/C++에서 직접 구현하고, edge case와 성능 문제까지 분석할 수 있다.

---

# Lecture 1. 자료구조를 왜 배우는가: Selection Criteria & Complexity

## 1. 핵심 질문

**자료구조는 왜 중요한가?**

단순히 데이터를 저장하기 위해서가 아니다.

진짜 질문은 이것이다.

> “내 프로그램이 어떤 방식으로 데이터를 읽고, 쓰고, 삭제하고, 순회하며, 메모리를 사용하는가?”

자료구조 선택은 결국 다음 세 가지를 결정한다.

| 질문              | 의미                                |
| --------------- | --------------------------------- |
| 데이터가 얼마나 많은가?   | N이 커졌을 때 버틸 수 있는가                 |
| 데이터를 어떻게 접근하는가? | 랜덤 접근인가, 순차 접근인가                  |
| 메모리 제약이 있는가?    | RAM, cache, fragmentation 문제가 있는가 |

올린 자료에서도 자료구조 선택 기준을 **data size, access frequency, memory constraints** 세 가지로 정리하고 있습니다. 

---

## 2. 형식적 정의

### Data Structure

**Formal definition**

자료구조란:

> 데이터를 저장하는 방식과, 그 데이터에 대해 허용되는 연산들의 집합이다.

즉 자료구조는 단순한 “컨테이너”가 아니다.

예를 들어 Stack은 다음 연산을 가진다.

```text
push(x)
pop()
top()
is_empty()
```

Queue는 다음 연산을 가진다.

```text
enqueue(x)
dequeue()
front()
is_empty()
```

Array는 다음 연산을 가진다.

```text
get(i)
set(i, x)
iterate()
insert(i, x)
delete(i)
```

자료구조를 볼 때는 항상 이 질문을 해야 한다.

> “이 자료구조에서 어떤 연산이 싸고, 어떤 연산이 비싼가?”

---

## 3. 직관적 설명

자료구조는 **물건을 보관하는 방식**과 비슷하다.

### Array

배열은 책이 번호순으로 꽂힌 책장이다.

```text
index:  0   1   2   3   4
value: [A] [B] [C] [D] [E]
```

3번 책을 찾으려면 바로 간다.

```c
arr[3]
```

빠르다. O(1).

하지만 중간에 새 책을 넣으려면?

```text
[A] [B] [C] [D] [E]
        ↑ 여기에 X 삽입
```

뒤의 책들을 전부 밀어야 한다.

```text
[A] [B] [X] [C] [D] [E]
```

비싸다. O(N).

---

### Linked List

연결 리스트는 각 책이 “다음 책의 위치”를 적은 쪽지를 들고 있는 구조다.

```text
[A | next] -> [B | next] -> [C | next] -> NULL
```

3번째 원소를 찾으려면 처음부터 따라가야 한다.

```text
A -> B -> C
```

느리다. O(N).

하지만 어떤 노드를 이미 알고 있다면, 그 뒤에 새 노드를 끼우는 것은 쉽다.

```text
A -> B -> D
```

여기에 C를 넣으면:

```text
A -> B -> C -> D
```

포인터만 바꾸면 된다. O(1).

---

## 4. 왜 필요한지

자료구조를 잘못 고르면 알고리즘이 망가진다.

예를 들어 채팅 서버를 만든다고 하자.

사용자 목록을 저장해야 한다.

### 상황 A: 사용자를 ID로 자주 찾는다

```text
find_user_by_id(42)
```

Array나 linked list로 매번 처음부터 찾으면 O(N).

사용자가 10명일 때는 괜찮다.

사용자가 100,000명이면 문제가 된다.

이때는 hash table이 더 적절하다.

```text
user_id -> User*
```

평균 O(1).

---

### 상황 B: 모든 사용자를 자주 순회한다

```text
broadcast(message)
```

모든 유저에게 메시지를 보내야 한다면 결국 O(N) 순회가 필요하다.

이때는 cache locality가 좋은 array/vector가 유리할 수 있다.

---

### 상황 C: embedded system에서 UART 데이터를 받는다

실시간으로 들어오는 데이터를 계속 저장해야 한다.

동적 할당을 자주 하면 위험하다.

이때는 fixed-size ring buffer가 적합하다.

자료에도 ring buffer는 UART, ADC 같은 continuous data stream에서 중요하다고 되어 있습니다. 

---

## 5. 내부 원리

자료구조의 성능은 단순히 Big-O만으로 결정되지 않는다.

크게 세 층이 있다.

```text
Level 1: Abstract operation
        push, pop, insert, delete, search

Level 2: Algorithmic cost
        O(1), O(log N), O(N), O(N log N)

Level 3: Hardware reality
        cache locality, allocation cost, pointer chasing, memory fragmentation
```

초보자는 보통 Level 1만 본다.

```text
"insert가 빠르다"
"search가 느리다"
```

중급자는 Level 2를 본다.

```text
array insert: O(N)
linked list insert: O(1) if node known
```

전문가는 Level 3까지 본다.

```text
linked list는 이론상 삽입이 좋아도 cache miss 때문에 실제로 느릴 수 있다.
vector는 resize 비용이 있지만 대부분의 순회 성능이 매우 좋다.
```

자료에서도 array는 contiguous memory를 사용하고, linked list는 non-contiguous memory를 사용한다고 비교합니다. 또한 array는 cache performance가 excellent, linked list는 poor라고 정리되어 있습니다. 

---

## 6. 단계별 예시

예제를 보자.

우리는 정수 5개를 저장한다.

```c
int arr[5] = {10, 20, 30, 40, 50};
```

### Operation 1: index access

```c
arr[3]
```

메모리상으로 배열은 연속되어 있다.

```text
base address = 1000
int size     = 4 bytes

arr[0] = address 1000
arr[1] = address 1004
arr[2] = address 1008
arr[3] = address 1012
```

컴퓨터는 이렇게 계산한다.

```text
address(arr[i]) = base + i * sizeof(int)
```

그래서 `arr[3]`은 바로 접근 가능하다.

복잡도:

```text
O(1)
```

---

### Operation 2: linear search

값 40을 찾는다고 하자.

```c
int find(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```

최악의 경우:

```text
target이 마지막에 있거나 아예 없음
```

모든 원소를 본다.

복잡도:

```text
O(N)
```

---

### Operation 3: middle deletion

```text
[10, 20, 30, 40, 50]
```

30을 삭제한다.

```text
[10, 20, _, 40, 50]
```

빈칸을 없애려면 뒤 원소를 왼쪽으로 당긴다.

```text
[10, 20, 40, 50]
```

원소 이동이 필요하다.

복잡도:

```text
O(N)
```

자료에서도 array의 middle deletion은 뒤 원소들을 shift해야 해서 O(N)이라고 설명합니다. 

---

## 7. 실제 응용

### 1. C++ `std::vector`

`std::vector`는 내부적으로 dynamic array다.

```cpp
std::vector<int> v;

v.push_back(10);
v.push_back(20);
v.push_back(30);
```

대부분의 `push_back`은 O(1)이다.

하지만 capacity가 꽉 차면:

```text
1. 더 큰 메모리 블록 할당
2. 기존 원소 복사 또는 이동
3. 기존 메모리 해제
4. 새 원소 삽입
```

이 순간은 O(N).

하지만 전체적으로 평균을 내면 `push_back`은 amortized O(1)이다.

자료에서도 dynamic array는 꽉 차면 보통 2배 크기로 재할당하고, 대부분 삽입은 O(1), 가끔 resize가 O(N), 평균적으로 amortized O(1)이라고 정리합니다. 

---

### 2. 서버의 user table

IRC 서버를 만든다고 하면 다음 두 가지 구조가 가능하다.

#### Option A: vector

```cpp
std::vector<User> users;
```

장점:

```text
- 순회 빠름
- cache locality 좋음
- broadcast에 유리
```

단점:

```text
- 중간 삭제 비용
- user id로 직접 찾기 어려움
```

#### Option B: map/hash table

```cpp
std::unordered_map<int, User> users;
```

장점:

```text
- user id로 빠르게 찾음
- 평균 O(1)
```

단점:

```text
- 순회 cache locality는 vector보다 나쁠 수 있음
- hash collision 고려 필요
```

전문적인 판단은 이렇게 해야 한다.

```text
"유저를 자주 순회하는가?"
"ID 기반 검색이 많은가?"
"삭제가 많은가?"
"메모리 사용량이 중요한가?"
```

---

### 3. BFS에서 Queue

그래프 탐색 BFS는 queue 없이는 자연스럽게 구현하기 어렵다.

```cpp
std::queue<int> q;

q.push(start);

while (!q.empty()) {
    int cur = q.front();
    q.pop();

    for (int next : graph[cur]) {
        if (!visited[next]) {
            visited[next] = true;
            q.push(next);
        }
    }
}
```

Queue는 FIFO다.

```text
먼저 들어온 노드를 먼저 처리한다.
```

그래서 BFS는 가까운 거리부터 탐색한다.

자료에서도 queue는 FIFO 구조이며 task scheduling, BFS, buffering에 사용된다고 정리되어 있습니다. 

---

## 8. 흔한 오해

### 오해 1. “Linked list는 삽입/삭제가 무조건 빠르다”

정확히는 아니다.

Linked list에서 중간 원소를 삭제하려면 먼저 그 위치를 찾아야 한다.

```text
find position: O(N)
delete node: O(1)
```

따라서 전체는 보통 O(N)이다.

이미 삭제할 노드와 이전 노드를 알고 있을 때만 O(1)에 가깝다.

자료도 linked list의 middle deletion은 위치 찾기 O(N), 삭제 자체 O(1)이라고 구분합니다. 

---

### 오해 2. “Big-O가 같으면 성능도 같다”

아니다.

예를 들어 array traversal과 linked list traversal은 둘 다 O(N)이다.

하지만 실제 성능은 array가 훨씬 빠를 수 있다.

왜냐하면 array는 메모리가 연속되어 있어 CPU cache를 잘 활용한다.

```text
Array:
[1][2][3][4][5]  ← contiguous

Linked list:
[1] -> [2] -> [3] -> [4]  ← scattered memory
```

Linked list는 다음 노드가 메모리 어디에 있을지 모른다.

그래서 pointer chasing과 cache miss가 발생한다.

---

### 오해 3. “O(1)은 항상 빠르다”

O(1)은 입력 크기 N에 비례해 증가하지 않는다는 뜻이다.

하지만 실제 시간이 작다는 뜻은 아니다.

예:

```text
작은 O(N) 연산: 10개 원소 순회
큰 O(1) 연산: 복잡한 hash 계산 + lock + memory allocation
```

Big-O는 성장률이지 절대 시간이 아니다.

---

## 9. 반례 또는 실패 사례

### 실패 사례: linked list를 성능 최적화라고 착각한 경우

어떤 학생이 다음처럼 생각했다고 하자.

```text
"중간 삽입/삭제가 많으니까 linked list를 써야지."
```

그런데 실제 프로그램은 다음 패턴이었다.

```text
1. 전체 원소를 자주 순회함
2. 삭제할 원소를 찾기 위해 매번 처음부터 탐색함
3. 원소 개수는 수만 개
```

Linked list는 삭제 자체는 O(1)이지만, 삭제할 원소를 찾는 과정이 O(N)이다.

게다가 노드들이 메모리에 흩어져 있어서 cache miss가 많다.

결과적으로 vector보다 느려질 수 있다.

### 핵심

```text
Linked list가 유리한 경우:
- 이미 노드 위치를 알고 있음
- 중간 삽입/삭제가 매우 많음
- iterator 안정성이 중요함

Vector가 유리한 경우:
- 순회가 많음
- random access가 필요함
- cache locality가 중요함
- 대부분의 실제 일반 상황
```

---

## 10. 확인 문제

### 문제 1

다음 작업이 많다면 어떤 자료구조가 적절한가?

```text
- 100만 개의 정수 저장
- 인덱스로 자주 접근
- 중간 삽입/삭제는 거의 없음
```

답을 생각해보자.

적절한 선택:

```text
Array 또는 Dynamic Array(vector)
```

이유:

```text
- index access O(1)
- contiguous memory
- cache locality 좋음
```

---

### 문제 2

다음 작업의 시간 복잡도는?

```cpp
for (int i = 0; i < n; i++) {
    std::cout << arr[i] << "\n";
}
```

정답:

```text
O(N)
```

이유:

```text
원소 n개를 각각 한 번씩 방문한다.
```

---

### 문제 3

다음 작업의 시간 복잡도는?

```cpp
int x = arr[500];
```

정답:

```text
O(1)
```

이유:

```text
배열은 base address + index * element size로 바로 주소 계산이 가능하다.
```

---

### 문제 4

Linked list에서 10,000번째 원소를 찾는 작업은?

정답:

```text
O(N)
```

이유:

```text
head부터 next pointer를 따라가야 한다.
```

---

## 11. 실습 과제

### 과제 1. Array 검색 함수 구현

C로 다음 함수를 구현해라.

```c
int find_index(int arr[], int n, int target);
```

조건:

```text
- target을 찾으면 index 반환
- 없으면 -1 반환
- empty array도 처리
```

예상 코드:

```c
int find_index(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target)
            return i;
    }
    return -1;
}
```

분석:

```text
Best case: O(1)
Worst case: O(N)
Space: O(1)
```

---

### 과제 2. Array deletion 구현

다음 함수를 구현해라.

```c
int delete_at(int arr[], int *n, int index);
```

조건:

```text
- index가 유효하지 않으면 0 반환
- 삭제 성공 시 1 반환
- 삭제 후 n 감소
```

예상 코드:

```c
int delete_at(int arr[], int *n, int index) {
    if (index < 0 || index >= *n)
        return 0;

    for (int i = index; i < *n - 1; i++) {
        arr[i] = arr[i + 1];
    }

    (*n)--;
    return 1;
}
```

복잡도:

```text
Worst case: O(N)
Space: O(1)
```

---

### 과제 3. 자료구조 선택 설명하기

다음 상황마다 자료구조를 골라라.

| 상황                | 추천 자료구조      | 이유                 |
| ----------------- | ------------ | ------------------ |
| 인덱스 접근이 많다        | array/vector | O(1) random access |
| 앞에서 자주 pop한다      | queue/deque  | front removal 효율적  |
| 마지막에 push/pop만 한다 | stack/vector | O(1)               |
| 실시간 고정 크기 버퍼      | ring buffer  | 재할당 없음             |
| key로 빠르게 찾는다      | hash table   | 평균 O(1)            |

---

## 12. 핵심 정리

이번 강의의 핵심은 이것이다.

```text
자료구조는 “무엇을 저장하느냐”보다
“어떻게 접근하고 변경하느냐”가 더 중요하다.
```

### 반드시 기억할 것

| 개념               | 핵심                                               |
| ---------------- | ------------------------------------------------ |
| Array            | 인덱스 접근 O(1), 중간 삽입/삭제 O(N), cache locality 좋음    |
| Linked List      | 순차 접근 O(N), 위치를 알면 삽입/삭제 O(1), cache locality 나쁨 |
| Big-O            | 실행 시간의 성장률                                       |
| Space Complexity | 추가 메모리 사용량                                       |
| Dynamic Array    | resize는 가끔 O(N), 평균 push_back은 amortized O(1)    |
| Ring Buffer      | 고정 메모리, 실시간 시스템에 적합                              |
