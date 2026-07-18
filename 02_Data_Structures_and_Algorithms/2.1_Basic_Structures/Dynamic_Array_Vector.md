# Lecture 4. Dynamic Array / Vector

## size, capacity, resizing, amortized O(1)

## 1. 핵심 질문

**배열은 크기가 고정인데, `std::vector`는 어떻게 계속 커질 수 있는가?**

그리고 더 중요한 질문:

> `push_back`은 가끔 O(N)인데, 왜 평균적으로 O(1)이라고 말하는가?

이 Lecture의 핵심은 세 단어다.

```text
size
capacity
amortized analysis
```

자료에서도 Dynamic Array는 꽉 찼을 때 보통 **2배 크기로 새 메모리를 할당하고, 기존 원소를 복사한 뒤, old memory를 해제**한다고 설명합니다. 이 과정 때문에 대부분의 삽입은 O(1)이지만, 가끔 resize는 O(N)이고, 전체 평균은 amortized O(1)입니다. 

---

## 2. 형식적 정의

### Dynamic Array

**Formal definition**

Dynamic array는 다음 조건을 만족하는 배열 기반 자료구조다.

> 내부적으로 연속된 메모리 블록을 사용하지만, 필요할 때 더 큰 메모리 블록을 새로 할당하여 논리적 크기를 확장할 수 있는 자료구조.

보통 두 개의 값을 가진다.

| 개념         | 의미                              |
| ---------- | ------------------------------- |
| `size`     | 현재 실제로 들어 있는 원소 개수              |
| `capacity` | 현재 할당된 메모리 공간이 담을 수 있는 최대 원소 개수 |

예:

```cpp
std::vector<int> v;
```

내부적으로는 대략 이런 상태를 가진다.

```text
size = 0
capacity = 0
data = nullptr
```

원소를 넣으면:

```cpp
v.push_back(10);
v.push_back(20);
v.push_back(30);
```

상태는 예를 들어 이렇게 될 수 있다.

```text
size = 3
capacity = 4

data:
[10][20][30][_]
```

여기서 `_`는 아직 사용하지 않는 공간이다.

---

## 3. 직관적 설명

Dynamic array는 **확장 가능한 책장**이 아니다.

더 정확히는:

> 책장이 꽉 차면, 더 큰 책장을 새로 사고, 기존 책을 전부 옮긴 다음, 낡은 책장을 버리는 구조다.

예를 들어 현재 capacity가 4라고 하자.

```text
size = 4
capacity = 4

[10][20][30][40]
```

여기에 `50`을 넣고 싶다.

하지만 공간이 없다.

그러면 vector는 보통 이런 일을 한다.

```text
1. 더 큰 공간을 새로 할당한다. 보통 2배.
2. 기존 원소를 새 공간으로 복사 또는 이동한다.
3. 새 원소 50을 넣는다.
4. 기존 공간을 해제한다.
```

결과:

```text
size = 5
capacity = 8

[10][20][30][40][50][_][_][_]
```

즉 vector는 기존 배열을 직접 늘리는 것이 아니다.

**새 배열로 이사하는 것**이다.

---

## 4. 왜 필요한지

고정 배열만 있으면 불편하다.

```c
int arr[100];
```

이 배열은 최대 100개만 담을 수 있다.

그런데 실제 프로그램에서는 데이터 개수를 미리 모르는 경우가 많다.

예:

```text
- 사용자 입력 개수
- 네트워크 패킷 개수
- 로그 라인 개수
- 그래프의 인접 정점 개수
- 파일에서 읽어온 토큰 개수
```

이때 매번 linked list를 쓰면?

```text
- index access 느림
- cache locality 나쁨
- pointer memory overhead 있음
```

그래서 실무에서는 대부분 다음 선택을 한다.

```text
일단 vector를 쓴다.
문제가 생기면 그때 다른 자료구조를 검토한다.
```

이유는 단순하다.

`std::vector`는:

```text
- 배열처럼 빠른 index access
- 배열처럼 좋은 cache locality
- 필요하면 자동 resizing
```

을 제공한다.

---

## 5. 내부 원리

Dynamic array의 내부 원리는 다음 네 단계로 보면 된다.

```text
1. 현재 size와 capacity를 확인한다.
2. capacity가 남아 있으면 바로 삽입한다.
3. capacity가 꽉 찼으면 더 큰 메모리를 할당한다.
4. 기존 원소를 옮기고 새 원소를 삽입한다.
```

---

### 5.1 Capacity가 남아 있는 경우

현재 상태:

```text
size = 3
capacity = 4

[10][20][30][_]
```

`push_back(40)` 실행:

```text
[10][20][30][40]
```

작업:

```text
data[size] = 40
size++
```

복잡도:

```text
O(1)
```

---

### 5.2 Capacity가 꽉 찬 경우

현재 상태:

```text
size = 4
capacity = 4

[10][20][30][40]
```

`push_back(50)` 실행.

공간이 없다.

그러면:

```text
new_capacity = 8
```

새 메모리:

```text
[_][_][_][_][_][_][_][_]
```

기존 원소 복사:

```text
[10][20][30][40][_][_][_][_]
```

새 원소 삽입:

```text
[10][20][30][40][50][_][_][_]
```

기존 메모리 해제.

복잡도:

```text
O(N)
```

왜냐하면 기존 N개의 원소를 옮겨야 하기 때문이다.

---

## 6. 단계별 예시

`capacity`가 1에서 시작하고, 꽉 찰 때마다 2배로 증가한다고 하자.

우리가 다음 원소들을 넣는다.

```text
10, 20, 30, 40, 50, 60, 70, 80
```

### Step 1. push 10

```text
size = 1
capacity = 1

[10]
```

비용:

```text
1 insertion
```

---

### Step 2. push 20

꽉 찼다.

capacity 1 → 2.

```text
copy 10
insert 20

[10][20]
```

비용:

```text
1 copy + 1 insert = 2
```

---

### Step 3. push 30

꽉 찼다.

capacity 2 → 4.

```text
copy 10, 20
insert 30

[10][20][30][_]
```

비용:

```text
2 copy + 1 insert = 3
```

---

### Step 4. push 40

공간 있음.

```text
[10][20][30][40]
```

비용:

```text
1
```

---

### Step 5. push 50

꽉 찼다.

capacity 4 → 8.

```text
copy 10, 20, 30, 40
insert 50

[10][20][30][40][50][_][_][_]
```

비용:

```text
4 copy + 1 insert = 5
```

---

### 전체 비용 보기

| 삽입 | 작업              | 비용 |
| -: | --------------- | -: |
| 10 | insert          |  1 |
| 20 | copy 1 + insert |  2 |
| 30 | copy 2 + insert |  3 |
| 40 | insert          |  1 |
| 50 | copy 4 + insert |  5 |
| 60 | insert          |  1 |
| 70 | insert          |  1 |
| 80 | insert          |  1 |

총 비용:

```text
1 + 2 + 3 + 1 + 5 + 1 + 1 + 1 = 15
```

8번 삽입에 총 비용 15.

평균:

```text
15 / 8 ≈ 1.875
```

즉 삽입 하나당 평균적으로 상수 시간에 가깝다.

그래서:

```text
push_back = amortized O(1)
```

---

## 7. Amortized Analysis

여기가 이 Lecture의 핵심이다.

### Formal explanation

**Amortized time complexity**는 다음을 의미한다.

> 하나의 연산이 최악의 경우 비싸더라도, 긴 연산 sequence 전체에서 평균 비용이 낮으면 그 평균 비용으로 분석하는 방식.

`vector.push_back()`은 한 번만 보면 최악의 경우 O(N)이다.

하지만 N번 연속으로 보면 전체 비용은 O(N)이다.

그래서 한 번당 평균 비용은:

```text
O(N) / N = O(1)
```

---

### 왜 전체 비용이 O(N)인가?

capacity가 1, 2, 4, 8, 16, ... 으로 증가한다고 하자.

N개를 삽입할 때 copy 비용은 대략:

```text
1 + 2 + 4 + 8 + ... + N/2
```

이 합은:

```text
< N
```

삽입 자체 비용 N개까지 포함하면:

```text
N insert + less than N copy = less than 2N
```

즉 전체 비용은:

```text
O(N)
```

그래서 push_back 하나의 amortized cost는:

```text
O(1)
```

---

## 8. 실제 응용

## 8.1 C++ `std::vector`

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;

    for (int i = 0; i < 10; i++) {
        v.push_back(i);

        std::cout << "size = " << v.size()
                  << ", capacity = " << v.capacity()
                  << "\n";
    }
}
```

출력은 구현마다 다르지만 대략 이런 패턴이 나온다.

```text
size = 1, capacity = 1
size = 2, capacity = 2
size = 3, capacity = 4
size = 4, capacity = 4
size = 5, capacity = 8
...
```

정확한 growth factor는 C++ 표준이 고정하지 않는다.

하지만 보통 1.5배 또는 2배에 가까운 방식으로 증가한다.

---

## 8.2 `reserve`

이미 몇 개가 들어올지 안다면 `reserve`를 쓰는 것이 좋다.

```cpp
std::vector<int> v;
v.reserve(1000000);

for (int i = 0; i < 1000000; i++) {
    v.push_back(i);
}
```

이렇게 하면 중간 resize를 크게 줄일 수 있다.

### 언제 중요한가?

```text
- 대량 데이터 로딩
- 실시간 처리
- embedded system
- latency-sensitive code
- pointer/reference 안정성이 필요한 코드
```

---

## 8.3 pointer/reference invalidation

이 부분은 실무에서 매우 중요하다.

`vector`가 resize되면 내부 메모리 주소가 바뀔 수 있다.

예:

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;
    v.push_back(10);

    int *p = &v[0];

    v.push_back(20);
    v.push_back(30);
    v.push_back(40);
    v.push_back(50);

    std::cout << *p << "\n"; // 위험할 수 있음
}
```

왜 위험한가?

`push_back` 도중 resize가 발생하면 기존 메모리가 해제되고, 새 메모리로 이동한다.

그러면 `p`는 더 이상 유효하지 않은 주소를 가리킬 수 있다.

이런 포인터를 dangling pointer라고 한다.

---

## 8.4 Embedded system에서의 주의점

embedded system에서는 dynamic array를 조심해야 한다.

이유:

```text
- resize 시점이 예측 불가능할 수 있음
- malloc/new 실패 가능성
- memory fragmentation
- worst-case latency 발생
```

실시간 시스템에서는 평균 성능보다 최악 시간이 중요하다.

예를 들어 자동차 제어 코드에서:

```text
평균적으로 빠르다
```

는 충분하지 않다.

중요한 것은:

```text
최악의 경우에도 deadline 안에 끝나는가?
```

그래서 embedded system에서는 다음 방식이 자주 사용된다.

```text
- 고정 크기 배열
- ring buffer
- static allocation
- memory pool
- reserve로 사전 할당
```

자료에서도 embedded systems에서는 RAM이 제한적이고, memory fragmentation이 중요한 고려사항이라고 언급합니다. 

---

## 9. 흔한 오해

## 오해 1. “vector는 배열이 아니다”

`std::vector`는 내부적으로 dynamic array다.

즉 본질은 배열이다.

```text
contiguous memory
index access O(1)
cache locality good
```

다만 고정 배열과 달리 필요하면 새 메모리로 이사한다.

---

## 오해 2. “push_back은 항상 O(1)이다”

정확히는:

```text
push_back은 amortized O(1)
```

가끔은 resize 때문에 O(N)이다.

면접이나 CS 설명에서는 이 차이를 정확히 말해야 한다.

---

## 오해 3. “capacity는 size와 같다”

아니다.

```cpp
std::vector<int> v;
v.reserve(10);
```

이때:

```text
size = 0
capacity >= 10
```

capacity는 공간만 확보한 것이다.

실제 원소가 10개 들어간 것이 아니다.

---

## 오해 4. “clear하면 메모리도 바로 줄어든다”

보통 아니다.

```cpp
std::vector<int> v;
v.reserve(1000);

v.push_back(1);
v.push_back(2);

v.clear();
```

이후:

```text
size = 0
capacity = 1000일 수 있음
```

`clear()`는 원소를 제거하지만, capacity를 반드시 줄이지는 않는다.

메모리를 줄이고 싶으면 보통:

```cpp
v.shrink_to_fit();
```

또는 swap trick을 쓴다.

```cpp
std::vector<int>().swap(v);
```

---

## 10. 반례 또는 실패 사례

## 실패 사례 1. 실시간 루프 안에서 push_back

```cpp
void control_loop() {
    std::vector<int> samples;

    while (true) {
        int x = read_sensor();
        samples.push_back(x);

        run_control_algorithm(samples);
    }
}
```

문제:

```text
push_back 도중 resize가 발생할 수 있다.
```

일반 프로그램에서는 괜찮을 수 있다.

하지만 실시간 제어에서는 특정 순간에 갑자기 latency가 튈 수 있다.

개선:

```cpp
std::vector<int> samples;
samples.reserve(MAX_SAMPLES);
```

또는 고정 배열 / ring buffer를 쓴다.

---

## 실패 사례 2. vector 원소 주소 저장 후 push_back

```cpp
std::vector<int> v;
v.push_back(10);

int *p = &v[0];

for (int i = 0; i < 1000; i++) {
    v.push_back(i);
}

std::cout << *p << "\n"; // 위험
```

`v`가 resize되면 `p`는 invalid해질 수 있다.

수정:

```cpp
std::vector<int> v;
v.reserve(1001);

v.push_back(10);

int *p = &v[0];

for (int i = 0; i < 1000; i++) {
    v.push_back(i);
}

std::cout << *p << "\n";
```

단, `reserve` 이후에도 capacity를 넘기면 다시 invalidation이 발생할 수 있다.

더 안전한 방법은 pointer 대신 index를 저장하는 것이다.

```cpp
size_t idx = 0;
std::cout << v[idx] << "\n";
```

---

## 실패 사례 3. 앞에서 계속 지우기

```cpp
std::vector<int> v;

// repeatedly
v.erase(v.begin());
```

`vector`의 앞 원소를 지우면 뒤의 모든 원소를 앞으로 당겨야 한다.

```text
[10][20][30][40]
erase 10
[20][30][40]
```

이 작업은 O(N).

반복하면 전체가 O(N²)이 될 수 있다.

이런 상황에서는 `deque`나 queue 구조가 더 적합할 수 있다.

---

## 11. 확인 문제

### 문제 1

다음 코드에서 `size`와 `capacity`는 같을까?

```cpp
std::vector<int> v;
v.reserve(100);
```

정답:

```text
size = 0
capacity >= 100
```

`reserve`는 공간만 확보한다.

---

### 문제 2

`vector.push_back()`의 시간 복잡도는?

정확한 답:

```text
Amortized O(1), but worst-case O(N)
```

---

### 문제 3

왜 resize가 O(N)인가?

정답:

```text
기존 원소 N개를 새 메모리로 복사 또는 이동해야 하기 때문이다.
```

---

### 문제 4

다음 코드의 문제는?

```cpp
std::vector<int> v;
v.push_back(1);

int *p = &v[0];

v.push_back(2);
v.push_back(3);
v.push_back(4);

std::cout << *p << "\n";
```

정답:

```text
push_back 중 resize가 발생하면 p가 dangling pointer가 될 수 있다.
```

---

### 문제 5

Embedded system에서 vector 사용 시 주의해야 할 점은?

정답:

```text
동적 할당, resize latency, memory fragmentation, allocation failure를 고려해야 한다.
```

---

## 12. 실습 과제

## 과제 1. size/capacity 관찰하기

다음 코드를 실행해라.

```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> v;

    for (int i = 0; i < 32; i++) {
        v.push_back(i);

        std::cout << "i = " << i
                  << ", size = " << v.size()
                  << ", capacity = " << v.capacity()
                  << "\n";
    }

    return 0;
}
```

관찰할 것:

```text
- capacity가 언제 증가하는가?
- 몇 배씩 증가하는가?
- 매 push_back마다 증가하지 않는 이유는 무엇인가?
```

---

## 과제 2. 간단한 Dynamic Array 직접 구현

C로 아주 단순한 vector를 만들어라.

```c
typedef struct {
    int *data;
    int size;
    int capacity;
} IntVector;
```

구현할 함수:

```c
void init_vector(IntVector *v);
int push_back(IntVector *v, int value);
void free_vector(IntVector *v);
```

예시:

```c
#include <stdlib.h>

typedef struct {
    int *data;
    int size;
    int capacity;
} IntVector;

void init_vector(IntVector *v) {
    v->data = NULL;
    v->size = 0;
    v->capacity = 0;
}

int push_back(IntVector *v, int value) {
    if (v->size == v->capacity) {
        int new_capacity = (v->capacity == 0) ? 1 : v->capacity * 2;

        int *new_data = malloc(sizeof(int) * new_capacity);
        if (new_data == NULL) {
            return 0;
        }

        for (int i = 0; i < v->size; i++) {
            new_data[i] = v->data[i];
        }

        free(v->data);

        v->data = new_data;
        v->capacity = new_capacity;
    }

    v->data[v->size] = value;
    v->size++;

    return 1;
}

void free_vector(IntVector *v) {
    free(v->data);
    v->data = NULL;
    v->size = 0;
    v->capacity = 0;
}
```

분석:

```text
push_back:
- 일반 case: O(1)
- resize case: O(N)
- amortized: O(1)

space:
- O(N)
- capacity가 size보다 클 수 있으므로 약간의 낭비가 있음
```

---

## 과제 3. `reserve` 효과 비교하기

다음 두 코드를 비교해라.

### Version A

```cpp
std::vector<int> v;

for (int i = 0; i < 1000000; i++) {
    v.push_back(i);
}
```

### Version B

```cpp
std::vector<int> v;
v.reserve(1000000);

for (int i = 0; i < 1000000; i++) {
    v.push_back(i);
}
```

분석할 것:

```text
- resize 횟수
- 실행 시간
- pointer invalidation 가능성
```

---

## 13. 핵심 정리

```text
Dynamic Array의 본질은 “연속 메모리를 유지하면서, 필요하면 더 큰 배열로 이사하는 구조”다.
```

| 개념                      | 의미                                           |
| ----------------------- | -------------------------------------------- |
| `size`                  | 현재 원소 개수                                     |
| `capacity`              | 현재 확보된 공간                                    |
| `push_back` 일반 case     | O(1)                                         |
| `push_back` resize case | O(N)                                         |
| `push_back` 전체 평균       | amortized O(1)                               |
| 장점                      | 빠른 index access, cache locality, 사용 편의성      |
| 단점                      | resize 비용, pointer invalidation, capacity 낭비 |
| 실무 팁                    | 예상 크기를 알면 `reserve` 사용                       |

## 반드시 기억할 문장

> `std::vector`는 “커지는 배열”이 아니라, **꽉 차면 더 큰 배열로 이사하는 자료구조**다.