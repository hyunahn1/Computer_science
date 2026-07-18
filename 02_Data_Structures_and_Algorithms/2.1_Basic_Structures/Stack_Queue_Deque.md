# Lecture 5. Stack, Queue, Deque

## 추상 자료형과 구현의 분리

## 1. 핵심 질문

**Stack, Queue, Deque는 자료구조인가, 아니면 규칙인가?**

핵심 질문은 이것이다.

> “데이터를 어떻게 저장하느냐보다, 어떤 순서로 꺼낼 수 있느냐가 더 중요한 경우가 있다.”

Array, linked list, vector는 주로 **메모리 배치 방식**에 관한 이야기였다.

반면 Stack, Queue, Deque는 주로 **접근 규칙**에 관한 이야기다.

자료에서도 Stack은 LIFO, Queue는 FIFO, Deque는 양쪽 끝 삽입/삭제가 가능한 구조로 정리되어 있습니다. 

---

# 2. 형식적 정의

## 2.1 Stack

**Formal definition**

Stack은 다음 연산을 지원하는 추상 자료형이다.

```text
push(x): 원소 x를 top에 추가
pop(): top 원소를 제거
top() 또는 peek(): top 원소를 확인
is_empty(): 비어 있는지 확인
```

Stack의 규칙은:

```text
LIFO = Last In, First Out
```

즉:

> 마지막에 들어온 것이 가장 먼저 나간다.

예:

```text
push(10)
push(20)
push(30)

pop() → 30
pop() → 20
pop() → 10
```

---

## 2.2 Queue

**Formal definition**

Queue는 다음 연산을 지원하는 추상 자료형이다.

```text
enqueue(x): rear에 원소 x 추가
dequeue(): front 원소 제거
front(): front 원소 확인
is_empty(): 비어 있는지 확인
```

Queue의 규칙은:

```text
FIFO = First In, First Out
```

즉:

> 먼저 들어온 것이 먼저 나간다.

예:

```text
enqueue(10)
enqueue(20)
enqueue(30)

dequeue() → 10
dequeue() → 20
dequeue() → 30
```

---

## 2.3 Deque

**Formal definition**

Deque는 **Double-Ended Queue**다.

양쪽 끝에서 삽입과 삭제가 가능하다.

```text
push_front(x)
push_back(x)
pop_front()
pop_back()
front()
back()
```

즉 Stack처럼도, Queue처럼도 쓸 수 있다.

| 사용 방식      | Deque 연산                                           |
| ---------- | -------------------------------------------------- |
| Stack처럼 사용 | `push_back`, `pop_back`                            |
| Queue처럼 사용 | `push_back`, `pop_front`                           |
| 양방향 버퍼     | `push_front`, `push_back`, `pop_front`, `pop_back` |

자료에서도 Deque는 양쪽 끝에서 insert/delete가 가능하고, stack과 queue 기능을 결합한다고 설명합니다. 

---

# 3. 직관적 설명

## Stack 직관

Stack은 접시 더미다.

```text
맨 위
[30] ← 마지막에 올림, 먼저 꺼냄
[20]
[10]
바닥
```

접시를 중간에서 빼지 않는다.

항상 위에서 넣고, 위에서 뺀다.

```text
push = 위에 올리기
pop = 위에서 꺼내기
```

---

## Queue 직관

Queue는 줄 서기다.

```text
front                         rear
[10] -> [20] -> [30]
```

10이 먼저 왔으므로 먼저 나간다.

```text
enqueue = 뒤에 줄 서기
dequeue = 앞에서 나가기
```

---

## Deque 직관

Deque는 양쪽 문이 있는 줄이다.

```text
front                         back
[10] <-> [20] <-> [30]
```

앞에서도 넣고 뺄 수 있고, 뒤에서도 넣고 뺄 수 있다.

```text
push_front(5)
push_back(40)
pop_front()
pop_back()
```

---

# 4. 왜 필요한지

Stack, Queue, Deque가 중요한 이유는 단순하다.

많은 알고리즘은 **순서 제어**가 핵심이기 때문이다.

| 문제              | 필요한 순서            | 적합한 구조 |
| --------------- | ----------------- | ------ |
| 함수 호출 관리        | 나중에 호출된 함수가 먼저 끝남 | Stack  |
| Undo / Redo     | 마지막 작업부터 되돌림      | Stack  |
| 괄호 검사           | 최근 열린 괄호부터 닫혀야 함  | Stack  |
| BFS             | 먼저 발견한 노드부터 탐색    | Queue  |
| 작업 스케줄링         | 먼저 들어온 작업부터 처리    | Queue  |
| Sliding window  | 양쪽 끝 관리           | Deque  |
| Monotonic queue | 앞뒤 원소 제거          | Deque  |

자료에서도 Stack은 함수 호출 스택, Undo/Redo, expression evaluation, backtracking에 쓰이고, Queue는 task scheduling, BFS, buffering에 사용된다고 되어 있습니다. 

---

# 5. 내부 원리

중요한 개념이 있다.

```text
ADT와 Implementation은 다르다.
```

ADT는 **무엇을 할 수 있는가**를 정의한다.

Implementation은 **어떻게 구현하는가**를 정의한다.

---

## 5.1 Stack은 여러 방식으로 구현 가능하다

Stack이라는 규칙:

```text
push
pop
top
```

구현 방식:

| 구현                     | 특징                    |
| ---------------------- | --------------------- |
| Array                  | 고정 크기, 빠름             |
| Dynamic Array / Vector | 자동 확장, cache-friendly |
| Linked List            | 동적 노드 할당, 크기 제한 적음    |

즉 Stack은 반드시 특정한 메모리 구조를 의미하지 않는다.

Stack은 **연산 규칙**이다.

---

## 5.2 Queue도 여러 방식으로 구현 가능하다

Queue 규칙:

```text
enqueue at rear
dequeue from front
```

구현 방식:

| 구현               | 문제                       |
| ---------------- | ------------------------ |
| Array + shifting | dequeue마다 O(N), 비효율      |
| Linked List      | front/rear pointer로 O(1) |
| Circular array   | 고정 메모리, O(1)             |
| Deque            | 양쪽 끝 O(1)                |

가장 초보적인 실수는 array에서 dequeue할 때 매번 당기는 것이다.

```text
[10][20][30][40]

dequeue 10

[20][30][40][_]
```

이렇게 하면 매번 O(N)이다.

그래서 queue는 보통 **front index**를 둔다.

---

# 6. 단계별 예시

## 예시 1. Stack으로 괄호 검사

문제:

```text
"({[]})" 는 올바른 괄호인가?
```

규칙:

```text
여는 괄호는 stack에 push
닫는 괄호는 stack top과 매칭되는지 확인
```

### Step-by-step

입력:

```text
( { [ ] } )
```

| 문자  | 동작                  | Stack   |
| --- | ------------------- | ------- |
| `(` | push                | `(`     |
| `{` | push                | `( {`   |
| `[` | push                | `( { [` |
| `]` | top이 `[`인지 확인 후 pop | `( {`   |
| `}` | top이 `{`인지 확인 후 pop | `(`     |
| `)` | top이 `(`인지 확인 후 pop | empty   |

마지막에 stack이 비어 있으면 valid.

### C++ 예시

```cpp
#include <stack>
#include <string>

bool is_valid_parentheses(const std::string& s) {
    std::stack<char> st;

    for (char c : s) {
        if (c == '(' || c == '{' || c == '[') {
            st.push(c);
        } else {
            if (st.empty()) return false;

            char t = st.top();
            st.pop();

            if (c == ')' && t != '(') return false;
            if (c == '}' && t != '{') return false;
            if (c == ']' && t != '[') return false;
        }
    }

    return st.empty();
}
```

복잡도:

```text
Time: O(N)
Space: O(N)
```

왜 space가 O(N)인가?

최악의 경우 입력이 전부 여는 괄호일 수 있다.

```text
(((((((((
```

이 경우 stack에 N개가 쌓인다.

---

## 예시 2. Queue로 BFS

그래프:

```text
0 -- 1 -- 3
|
2 -- 4
```

인접 리스트:

```text
0: 1, 2
1: 0, 3
2: 0, 4
3: 1
4: 2
```

BFS는 가까운 노드부터 탐색한다.

```text
start = 0
```

### Step-by-step

|   단계 | Queue    | 방문        |
| ---: | -------- | --------- |
|   시작 | `[0]`    | 0         |
| 0 처리 | `[1, 2]` | 0,1,2     |
| 1 처리 | `[2, 3]` | 0,1,2,3   |
| 2 처리 | `[3, 4]` | 0,1,2,3,4 |
| 3 처리 | `[4]`    | 그대로       |
| 4 처리 | `[]`     | 완료        |

### C++ 예시

```cpp
#include <queue>
#include <vector>

void bfs(int start, const std::vector<std::vector<int>>& graph) {
    std::vector<bool> visited(graph.size(), false);
    std::queue<int> q;

    visited[start] = true;
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
}
```

복잡도:

```text
Time: O(V + E)
Space: O(V)
```

V는 vertex 개수, E는 edge 개수다.

---

## 예시 3. Deque로 Sliding Window Maximum

문제:

```text
배열 [1, 3, -1, -3, 5, 3, 6, 7]
window size = 3

각 구간의 최댓값을 구하라.
```

구간:

```text
[1, 3, -1] → 3
[3, -1, -3] → 3
[-1, -3, 5] → 5
[-3, 5, 3] → 5
[5, 3, 6] → 6
[3, 6, 7] → 7
```

정답:

```text
[3, 3, 5, 5, 6, 7]
```

단순 방식은 각 window마다 3개를 검사한다.

일반화하면:

```text
O(NK)
```

Deque를 쓰면:

```text
O(N)
```

핵심 아이디어:

```text
Deque에는 “최댓값 후보의 index”만 유지한다.
앞쪽에는 항상 현재 window의 최댓값 후보가 온다.
```

C++ 예시:

```cpp
#include <deque>
#include <vector>

std::vector<int> sliding_window_max(const std::vector<int>& nums, int k) {
    std::deque<int> dq;
    std::vector<int> result;

    for (int i = 0; i < (int)nums.size(); i++) {
        // 1. window 밖 index 제거
        while (!dq.empty() && dq.front() <= i - k) {
            dq.pop_front();
        }

        // 2. 현재 값보다 작거나 같은 값들은 후보 탈락
        while (!dq.empty() && nums[dq.back()] <= nums[i]) {
            dq.pop_back();
        }

        // 3. 현재 index 추가
        dq.push_back(i);

        // 4. 첫 window가 완성된 뒤부터 정답 기록
        if (i >= k - 1) {
            result.push_back(nums[dq.front()]);
        }
    }

    return result;
}
```

이건 Deque가 단순히 queue의 확장이 아니라, 알고리즘 패턴으로도 중요하다는 예다.

---

# 7. 실제 응용

## 7.1 Stack — Function Call Stack

함수 호출은 stack 구조로 관리된다.

```c
void c() {}
void b() { c(); }
void a() { b(); }

int main() {
    a();
}
```

호출 흐름:

```text
main calls a
a calls b
b calls c
```

Call stack:

```text
top
[c]
[b]
[a]
[main]
bottom
```

`c()`가 끝나면 먼저 제거된다.

```text
pop c
pop b
pop a
pop main
```

그래서 함수 호출은 LIFO다.

---

## 7.2 Stack — DFS / Backtracking

미로 탐색, 조합 생성, 재귀 문제는 stack과 관련이 깊다.

재귀도 내부적으로 call stack을 사용한다.

```cpp
void dfs(int cur) {
    visited[cur] = true;

    for (int next : graph[cur]) {
        if (!visited[next]) {
            dfs(next);
        }
    }
}
```

재귀 호출이 깊어지면 stack이 쌓인다.

너무 깊으면:

```text
stack overflow
```

가 발생할 수 있다.

---

## 7.3 Queue — Producer/Consumer

Queue는 producer-consumer 구조에서 자주 사용된다.

```text
Producer → Queue → Consumer
```

예:

```text
네트워크 스레드가 패킷을 queue에 넣음
처리 스레드가 queue에서 패킷을 꺼냄
```

이 구조는 시스템 설계에서 매우 흔하다.

---

## 7.4 Queue — OS Scheduling

운영체제의 ready queue는 실행 준비가 된 프로세스/스레드를 관리한다.

```text
ready queue:
[P1][P2][P3][P4]
```

Round-robin scheduling에서는 앞에서 하나 꺼내 실행하고, 아직 끝나지 않았으면 다시 뒤에 넣는다.

```text
dequeue P1
run P1
enqueue P1
```

이건 다음 Lecture에서 볼 circular queue와도 연결된다.

---

# 8. 흔한 오해

## 오해 1. “Stack은 무조건 배열로 구현한다”

아니다.

Stack은 ADT다.

구현은 여러 가지다.

```text
Array-based stack
Vector-based stack
Linked-list-based stack
```

다만 실무에서는 vector 기반 stack이 cache locality 때문에 유리한 경우가 많다.

---

## 오해 2. “Queue는 array로 쉽게 구현하면 된다”

조심해야 한다.

잘못된 queue 구현:

```cpp
std::vector<int> q;

q.push_back(10);
q.push_back(20);

q.erase(q.begin()); // dequeue
```

`erase(q.begin())`은 O(N)이다.

뒤 원소를 전부 앞으로 당기기 때문이다.

queue를 배열로 구현하려면 보통 `front` index를 따로 둬야 한다.

---

## 오해 3. “Deque는 그냥 queue랑 비슷하다”

Deque는 queue보다 더 강력하다.

Queue:

```text
push back
pop front
```

Deque:

```text
push front
push back
pop front
pop back
```

특히 sliding window, monotonic queue 같은 문제에서 매우 중요하다.

---

## 오해 4. “재귀는 stack과 상관없다”

재귀 호출은 call stack을 사용한다.

예:

```cpp
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);
}
```

`factorial(5)`는 내부적으로 이런 stack을 만든다.

```text
factorial(5)
factorial(4)
factorial(3)
factorial(2)
factorial(1)
factorial(0)
```

그래서 재귀 깊이가 너무 크면 stack overflow가 생긴다.

---

# 9. 반례 또는 실패 사례

## 실패 사례 1. Queue를 vector erase로 구현

```cpp
std::vector<int> q;

void enqueue(int x) {
    q.push_back(x);
}

int dequeue() {
    int x = q.front();
    q.erase(q.begin());
    return x;
}
```

문제:

```text
dequeue가 O(N)
```

작업이 N번이면 전체 O(N²)이 될 수 있다.

개선:

```cpp
#include <queue>

std::queue<int> q;
```

또는 circular buffer 구현.

---

## 실패 사례 2. Stack empty 체크 없이 pop

```cpp
std::stack<int> st;

int x = st.top();
st.pop();
```

비어 있는 stack에서 `top()` 또는 `pop()`을 호출하면 문제가 된다.

안전한 방식:

```cpp
if (!st.empty()) {
    int x = st.top();
    st.pop();
}
```

---

## 실패 사례 3. BFS에서 visited 체크를 늦게 함

잘못된 방식:

```cpp
while (!q.empty()) {
    int cur = q.front();
    q.pop();

    if (visited[cur]) continue;
    visited[cur] = true;

    for (int next : graph[cur]) {
        q.push(next);
    }
}
```

이 방식도 동작할 수는 있지만, 같은 노드가 queue에 여러 번 들어갈 수 있다.

더 좋은 방식:

```cpp
visited[start] = true;
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

노드를 queue에 넣는 순간 visited 처리하면 중복 삽입을 줄일 수 있다.

---

# 10. 확인 문제

## 문제 1

다음 출력은?

```cpp
std::stack<int> st;

st.push(1);
st.push(2);
st.push(3);

std::cout << st.top() << "\n";
```

정답:

```text
3
```

이유:

```text
Stack은 LIFO라서 마지막에 들어간 3이 top이다.
```

---

## 문제 2

다음 출력 순서는?

```cpp
std::queue<int> q;

q.push(1);
q.push(2);
q.push(3);

while (!q.empty()) {
    std::cout << q.front() << " ";
    q.pop();
}
```

정답:

```text
1 2 3
```

이유:

```text
Queue는 FIFO다.
```

---

## 문제 3

BFS에 queue를 쓰는 이유는?

정답:

```text
먼저 발견한 노드를 먼저 처리해야 거리 순서가 보장되기 때문이다.
```

---

## 문제 4

DFS에 stack 또는 재귀가 자연스러운 이유는?

정답:

```text
한 경로를 깊게 들어갔다가 되돌아오는 구조가 LIFO와 맞기 때문이다.
```

---

## 문제 5

Deque가 필요한 대표 알고리즘 패턴 하나는?

정답 예시:

```text
Sliding window maximum
Monotonic queue
Palindrome check
0-1 BFS
```

---

# 11. 실습 과제

## 과제 1. Array 기반 Stack 구현

C로 구현해라.

```c
#define CAPACITY 100

typedef struct {
    int data[CAPACITY];
    int top;
} Stack;
```

필요 함수:

```c
void init(Stack *s);
int push(Stack *s, int value);
int pop(Stack *s, int *out);
int peek(Stack *s, int *out);
int is_empty(Stack *s);
```

예시 구현:

```c
#define CAPACITY 100

typedef struct {
    int data[CAPACITY];
    int top;
} Stack;

void init(Stack *s) {
    s->top = 0;
}

int is_empty(Stack *s) {
    return s->top == 0;
}

int push(Stack *s, int value) {
    if (s->top == CAPACITY) {
        return 0;
    }

    s->data[s->top] = value;
    s->top++;
    return 1;
}

int pop(Stack *s, int *out) {
    if (is_empty(s)) {
        return 0;
    }

    s->top--;
    *out = s->data[s->top];
    return 1;
}

int peek(Stack *s, int *out) {
    if (is_empty(s)) {
        return 0;
    }

    *out = s->data[s->top - 1];
    return 1;
}
```

---

## 과제 2. Queue를 잘못 구현한 코드 분석

다음 코드의 문제점을 설명해라.

```cpp
std::vector<int> q;

void enqueue(int x) {
    q.push_back(x);
}

int dequeue() {
    int x = q[0];
    q.erase(q.begin());
    return x;
}
```

분석해야 할 것:

```text
1. dequeue의 시간 복잡도
2. 많은 데이터를 처리할 때 왜 느려지는가
3. 어떤 구조로 바꿀 수 있는가
```

---

## 과제 3. 괄호 검사 직접 구현

다음 입력에 대해 valid/invalid를 판별하는 함수를 작성해라.

```text
"()[]{}"      → valid
"([{}])"      → valid
"(]"          → invalid
"([)]"        → invalid
"((("         → invalid
```

조건:

```text
- stack 사용
- O(N)
- empty stack에서 top 접근 금지
```

---

## 과제 4. BFS 구현

그래프가 주어졌을 때 BFS 방문 순서를 출력해라.

```cpp
std::vector<std::vector<int>> graph = {
    {1, 2},
    {0, 3},
    {0, 4},
    {1},
    {2}
};
```

시작 정점:

```text
0
```

예상 방문 순서:

```text
0 1 2 3 4
```

단, 인접 리스트 순서에 따라 방문 순서는 달라질 수 있다.

---

# 12. 핵심 정리

```text
Stack, Queue, Deque의 본질은 “메모리 배치”가 아니라 “접근 순서의 규칙”이다.
```

| 구조    | 규칙      | 주요 연산                   | 대표 응용                                    |
| ----- | ------- | ----------------------- | ---------------------------------------- |
| Stack | LIFO    | push, pop, top          | 함수 호출, 괄호 검사, DFS, undo                  |
| Queue | FIFO    | enqueue, dequeue, front | BFS, scheduling, buffering               |
| Deque | 양쪽 끝 접근 | push/pop front/back     | sliding window, monotonic queue, 0-1 BFS |

## 반드시 기억할 문장

> Stack과 Queue는 “어떻게 저장하는가”보다 **어떤 순서로 꺼내는가**를 정의하는 추상 자료형이다.