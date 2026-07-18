# Lecture 6. Circular Queue / Ring Buffer

## 고정 메모리와 실시간 시스템

## 1. 핵심 질문

**Queue를 배열로 구현할 때, 앞에서 꺼낸 공간을 어떻게 재사용할 것인가?**

일반 queue는 FIFO다.

```text
enqueue: 뒤에 넣기
dequeue: 앞에서 빼기
```

그런데 배열로 queue를 단순하게 구현하면 문제가 생긴다.

```text
[10][20][30][40][_]
 front          rear
```

`dequeue()`를 두 번 하면:

```text
[_][_][30][40][_]
        front  rear
```

앞쪽에 빈 공간이 생긴다.

이 공간을 버릴 것인가?
아니면 다시 사용할 것인가?

**Circular Queue / Ring Buffer의 핵심은 이 빈 공간을 다시 사용하는 것**이다.

자료에서도 circular queue의 핵심 문제는 `front == rear`일 때 이것이 empty인지 full인지 구분하기 어렵다는 점이고, 이를 해결하기 위한 방법으로 **one slot waste, counter, flag**를 제시합니다. 

---

# 2. 형식적 정의

## 2.1 Circular Queue

**Formal definition**

Circular queue는 고정 크기 배열 `A[0...capacity-1]`과 두 인덱스 `front`, `rear`를 사용하여 FIFO queue를 구현하는 자료구조다.

핵심 연산은 다음과 같다.

```text
enqueue(x): rear 위치에 x를 넣고 rear를 한 칸 이동
dequeue(): front 위치의 값을 꺼내고 front를 한 칸 이동
```

단, 인덱스 이동은 일반 증가가 아니라 **modulo 연산**을 사용한다.

```text
next_index = (index + 1) % capacity
```

예를 들어 capacity가 5라면:

```text
0 -> 1 -> 2 -> 3 -> 4 -> 0 -> 1 -> ...
```

즉 배열의 끝에 도달하면 다시 처음으로 돌아온다.

---

## 2.2 Ring Buffer

Ring buffer는 circular queue의 실무적 이름에 가깝다.

특히 embedded system, OS, networking, audio processing에서 자주 쓰인다.

```text
Producer writes data
Consumer reads data
Fixed-size circular memory
```

자료에서도 ring buffer는 UART, ADC 같은 continuous data stream에서 중요하며, **memory reallocation이 없고 constant memory footprint를 가진다**고 설명합니다. 

---

# 3. 직관적 설명

Queue를 일렬 줄이라고 생각하면 앞에서 빠진 공간이 낭비된다.

```text
[ ][ ][30][40][ ]
```

Circular queue는 배열을 원형 트랙처럼 생각한다.

```text
        0
    4       1
      3   2
```

인덱스가 끝까지 가면 다시 0으로 돌아온다.

```text
(index + 1) % capacity
```

예:

```text
capacity = 5

0 다음은 1
1 다음은 2
2 다음은 3
3 다음은 4
4 다음은 0
```

이 구조 때문에 앞에서 빠진 공간을 재사용할 수 있다.

---

# 4. 왜 필요한지

Circular queue가 필요한 이유는 명확하다.

## 4.1 배열 queue의 shifting 문제 제거

나쁜 queue 구현:

```cpp
std::vector<int> q;

void dequeue() {
    q.erase(q.begin());
}
```

이 방식은 앞 원소를 지울 때마다 뒤 원소를 전부 앞으로 당긴다.

```text
[10][20][30][40]

dequeue 10

[20][30][40][_]
```

복잡도:

```text
O(N)
```

Circular queue는 shifting하지 않는다.

```text
front만 한 칸 이동한다.
```

복잡도:

```text
O(1)
```

---

## 4.2 실시간 시스템에서 예측 가능성

실시간 시스템에서는 평균적으로 빠른 것보다 더 중요한 것이 있다.

```text
최악의 경우에도 일정 시간 안에 끝나는가?
```

Dynamic array/vector는 resize가 발생할 수 있다.

```text
평소 push_back: O(1)
resize 순간: O(N)
```

반면 ring buffer는 capacity가 고정되어 있다.

```text
enqueue: O(1)
dequeue: O(1)
memory allocation 없음
```

그래서 embedded system에서 강하다.

예:

```text
UART receive buffer
ADC sample buffer
audio stream buffer
network packet queue
sensor data queue
```

---

# 5. 내부 원리

Circular queue는 보통 세 가지 요소를 가진다.

```c
#define CAPACITY 8

typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
} CircularQueue;
```

의미:

| 필드      | 의미              |
| ------- | --------------- |
| `data`  | 고정 크기 배열        |
| `front` | 다음에 dequeue할 위치 |
| `rear`  | 다음에 enqueue할 위치 |

초기 상태:

```text
front = 0
rear  = 0
```

비어 있음:

```text
front == rear
```

하지만 여기서 문제가 생긴다.

---

## 5.1 Empty와 Full이 같은 모양이 되는 문제

단순하게 구현하면:

```text
empty: front == rear
```

그런데 계속 enqueue해서 꽉 차면 `rear`가 한 바퀴 돌아 `front`와 같아질 수 있다.

```text
full: front == rear
```

즉:

```text
front == rear
```

가 empty인지 full인지 구분이 안 된다.

자료에서도 circular queue에서 `front == rear`가 empty와 full 모두를 나타낼 수 있다는 점을 핵심 문제로 제시합니다. 

---

# 6. 해결 방법 3가지

## 방법 1. 한 칸 비워두기

가장 흔한 방식 중 하나다.

규칙:

```text
empty: front == rear
full:  (rear + 1) % capacity == front
```

즉 실제 capacity가 8이라면 최대 7개만 저장한다.

```text
usable capacity = capacity - 1
```

장점:

```text
구현이 단순하다.
```

단점:

```text
한 칸을 사용하지 못한다.
```

---

## 방법 2. count 사용

현재 원소 개수를 따로 저장한다.

```c
typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
    int count;
} CircularQueue;
```

규칙:

```text
empty: count == 0
full:  count == capacity
```

장점:

```text
배열 전체 capacity를 사용할 수 있다.
```

단점:

```text
count 업데이트를 정확히 해야 한다.
동시성 환경에서는 count race condition에 주의해야 한다.
```

---

## 방법 3. full flag 사용

별도의 boolean flag를 둔다.

```c
typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
    int full;
} CircularQueue;
```

규칙:

```text
empty: front == rear && !full
full:  front == rear && full
```

장점:

```text
전체 capacity 사용 가능
```

단점:

```text
상태 관리가 조금 복잡하다.
```

---

# 7. 단계별 예시

이번에는 **한 칸 비워두기 방식**으로 설명한다.

capacity = 5라고 하자.
그러면 실제로 저장 가능한 원소 수는 4개다.

```text
data:  [ _ ][ _ ][ _ ][ _ ][ _ ]
index:   0    1    2    3    4

front = 0
rear  = 0
```

## Step 1. enqueue(10)

rear 위치에 넣고 rear 이동.

```text
data:  [10][ _ ][ _ ][ _ ][ _ ]
index:   0    1    2    3    4

front = 0
rear  = 1
```

---

## Step 2. enqueue(20)

```text
data:  [10][20][ _ ][ _ ][ _ ]

front = 0
rear  = 2
```

---

## Step 3. enqueue(30)

```text
data:  [10][20][30][ _ ][ _ ]

front = 0
rear  = 3
```

---

## Step 4. dequeue()

front 위치의 10을 꺼내고 front 이동.

```text
return 10

data:  [10][20][30][ _ ][ _ ]
          ^
        old value, ignored

front = 1
rear  = 3
```

주의:

```text
data[0]을 반드시 지울 필요는 없다.
front가 1이 되었기 때문에 data[0]은 논리적으로 queue 밖이다.
```

---

## Step 5. enqueue(40)

```text
data:  [10][20][30][40][ _ ]

front = 1
rear  = 4
```

---

## Step 6. enqueue(50)

rear가 4이므로 data[4]에 넣는다.

그다음 rear는:

```text
(rear + 1) % capacity = (4 + 1) % 5 = 0
```

결과:

```text
data:  [10][20][30][40][50]

front = 1
rear  = 0
```

논리적 queue는 다음 순서다.

```text
20 -> 30 -> 40 -> 50
```

배열상으로는 끊겨 보이지만, 논리적으로는 이어져 있다.

```text
index 1 -> 2 -> 3 -> 4 -> 0
```

---

# 8. C 구현: 한 칸 비워두기 방식

```c
#include <stdio.h>

#define CAPACITY 5

typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
} CircularQueue;

void init_queue(CircularQueue *q) {
    q->front = 0;
    q->rear = 0;
}

int is_empty(CircularQueue *q) {
    return q->front == q->rear;
}

int is_full(CircularQueue *q) {
    return (q->rear + 1) % CAPACITY == q->front;
}

int enqueue(CircularQueue *q, int value) {
    if (is_full(q)) {
        return 0;
    }

    q->data[q->rear] = value;
    q->rear = (q->rear + 1) % CAPACITY;

    return 1;
}

int dequeue(CircularQueue *q, int *out) {
    if (is_empty(q)) {
        return 0;
    }

    *out = q->data[q->front];
    q->front = (q->front + 1) % CAPACITY;

    return 1;
}
```

## 복잡도

| 연산         | 시간 복잡도 |
| ---------- | -----: |
| `enqueue`  |   O(1) |
| `dequeue`  |   O(1) |
| `is_empty` |   O(1) |
| `is_full`  |   O(1) |

공간 복잡도:

```text
O(CAPACITY)
```

단, capacity가 고정되어 있으면 실시간 시스템 관점에서는 constant memory다.

---

# 9. 실제 응용

## 9.1 UART Receive Buffer

Embedded system에서 UART 데이터를 받는다고 하자.

상황:

```text
외부 장치가 바이트를 계속 보냄
MCU는 interrupt로 바이트를 받음
main loop는 나중에 처리함
```

구조:

```text
ISR producer  ->  Ring Buffer  ->  Main loop consumer
```

자료에서도 UART 예시를 들어 ISR이 buffer에 쓰고, main loop가 읽는 producer-consumer pattern을 설명합니다. 

간단한 구조:

```c
#define UART_BUF_SIZE 128

volatile unsigned char buffer[UART_BUF_SIZE];
volatile int front = 0;
volatile int rear = 0;
```

ISR:

```c
void UART_RX_ISR(void) {
    unsigned char byte = UART_READ();

    int next = (rear + 1) % UART_BUF_SIZE;

    if (next != front) {
        buffer[rear] = byte;
        rear = next;
    } else {
        // buffer full: drop byte or set overflow flag
    }
}
```

Main loop:

```c
int uart_read_byte(unsigned char *out) {
    if (front == rear) {
        return 0;
    }

    *out = buffer[front];
    front = (front + 1) % UART_BUF_SIZE;
    return 1;
}
```

핵심:

```text
ISR은 최대한 짧게 끝나야 한다.
동적 할당하면 안 된다.
고정된 시간 안에 처리되어야 한다.
```

Ring buffer가 적합하다.

---

## 9.2 Audio Buffer

Audio processing에서도 ring buffer가 많이 쓰인다.

```text
Audio input thread  -> ring buffer -> audio processing thread
```

이유:

```text
- 오디오 데이터는 계속 흐른다.
- 버퍼가 끊기면 noise/dropout 발생.
- 일정한 latency가 중요하다.
```

---

## 9.3 Network Packet Queue

네트워크 드라이버나 OS 내부에서도 packet buffer를 queue로 관리한다.

```text
NIC receives packet
driver writes packet descriptor into ring
kernel consumes descriptor
```

고성능 네트워크에서는 ring 구조가 매우 일반적이다.

---

## 9.4 Logging Buffer

실시간 시스템에서 로그를 바로 파일이나 네트워크로 쓰면 느릴 수 있다.

대신:

```text
log producer -> ring buffer -> background flush
```

이렇게 분리한다.

---

# 10. 흔한 오해

## 오해 1. “Circular queue는 배열이 실제로 원형이다”

아니다.

메모리는 여전히 일렬 배열이다.

```text
[0][1][2][3][4]
```

원형이라는 말은 **인덱스 계산이 원형처럼 돌아간다**는 뜻이다.

```text
next = (i + 1) % capacity
```

---

## 오해 2. “dequeue하면 값을 지워야 한다”

필수는 아니다.

```text
dequeue는 front를 이동시키는 것만으로 충분하다.
```

값을 0으로 지우는 것은 디버깅에는 도움이 될 수 있지만, 논리적으로는 필요 없다.

---

## 오해 3. “front == rear이면 항상 empty다”

한 칸 비워두기 방식에서는 맞다.

하지만 count 방식이나 flag 방식에서는 다르게 해석될 수 있다.

```text
front == rear && count == 0        -> empty
front == rear && count == capacity -> full
```

따라서 구현 방식에 따라 의미가 달라진다.

---

## 오해 4. “Ring buffer는 overflow를 자동으로 해결한다”

아니다.

Buffer가 꽉 찼을 때 정책을 정해야 한다.

대표 정책:

| 정책              | 의미                 |
| --------------- | ------------------ |
| Drop new        | 새 데이터를 버림          |
| Drop old        | 오래된 데이터를 버림        |
| Block producer  | 공간 생길 때까지 기다림      |
| Overwrite       | 새 데이터가 오래된 데이터를 덮음 |
| Signal overflow | overflow flag 설정   |

Embedded UART에서는 보통 overflow flag를 세우거나 새 데이터를 버린다.

Audio에서는 오래된 데이터를 버리거나 dropout 처리를 할 수 있다.

---

# 11. 반례 또는 실패 사례

## 실패 사례 1. Full condition을 잘못 계산

잘못된 코드:

```c
int is_full(CircularQueue *q) {
    return q->rear == q->front;
}
```

문제:

```text
empty와 full을 구분할 수 없다.
```

한 칸 비워두기 방식에서는 이렇게 해야 한다.

```c
int is_full(CircularQueue *q) {
    return (q->rear + 1) % CAPACITY == q->front;
}
```

---

## 실패 사례 2. Modulo를 빼먹음

잘못된 코드:

```c
q->rear = q->rear + 1;
```

문제:

```text
rear가 CAPACITY를 넘어 배열 범위를 벗어난다.
```

올바른 코드:

```c
q->rear = (q->rear + 1) % CAPACITY;
```

---

## 실패 사례 3. Producer/Consumer 동시성 문제

ISR과 main loop가 같은 `front`, `rear`를 동시에 만진다고 하자.

```text
ISR: rear 변경
main: front 변경
```

단일 producer, 단일 consumer에서는 비교적 단순하게 만들 수 있지만, 그래도 다음을 고려해야 한다.

```text
- volatile 필요성
- atomicity
- interrupt disable 구간
- memory ordering
```

특히 multi-thread 환경에서는 lock-free ring buffer를 잘못 구현하면 data race가 생긴다.

---

## 실패 사례 4. 크기를 너무 작게 잡음

UART buffer를 8바이트로 잡았다고 하자.

```text
baud rate가 빠름
main loop 처리가 느림
buffer가 금방 참
데이터 손실
```

Ring buffer는 overflow를 없애는 마법이 아니다.

**생산 속도와 소비 속도의 차이**를 흡수하는 장치다.

평균적으로 producer가 consumer보다 계속 빠르면 언젠가는 반드시 overflow가 난다.

---

# 12. 확인 문제

## 문제 1

capacity = 5이고, 한 칸 비워두기 방식을 쓴다.
최대 몇 개의 원소를 저장할 수 있는가?

정답:

```text
4개
```

이유:

```text
empty와 full을 구분하기 위해 한 칸을 비워둔다.
```

---

## 문제 2

다음 식의 의미는?

```c
rear = (rear + 1) % CAPACITY;
```

정답:

```text
rear를 한 칸 이동시키되, 배열 끝에 도달하면 다시 0으로 돌아가게 한다.
```

---

## 문제 3

한 칸 비워두기 방식에서 empty 조건은?

정답:

```c
front == rear
```

---

## 문제 4

한 칸 비워두기 방식에서 full 조건은?

정답:

```c
(rear + 1) % CAPACITY == front
```

---

## 문제 5

Ring buffer가 embedded system에 적합한 이유는?

정답:

```text
고정 메모리를 사용하고, enqueue/dequeue가 O(1)이며, 동적 메모리 할당이 없기 때문이다.
```

---

# 13. 실습 과제

## 과제 1. Circular Queue 직접 구현

다음 구조체를 사용해라.

```c
#define CAPACITY 8

typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
} CircularQueue;
```

구현할 함수:

```c
void init_queue(CircularQueue *q);
int is_empty(CircularQueue *q);
int is_full(CircularQueue *q);
int enqueue(CircularQueue *q, int value);
int dequeue(CircularQueue *q, int *out);
```

조건:

```text
- 한 칸 비워두기 방식 사용
- enqueue 성공 시 1, 실패 시 0
- dequeue 성공 시 1, 실패 시 0
```

---

## 과제 2. 상태 추적하기

capacity = 5, 한 칸 비워두기 방식.

다음 연산 후 `front`, `rear`, 논리적 queue 상태를 적어라.

```text
enqueue(10)
enqueue(20)
enqueue(30)
dequeue()
enqueue(40)
enqueue(50)
dequeue()
enqueue(60)
```

힌트:

```text
index는 0,1,2,3,4 다음 다시 0으로 돌아간다.
```

---

## 과제 3. Overflow 정책 설계하기

UART receive buffer를 만든다고 가정해라.

Buffer가 꽉 찼을 때 어떤 정책을 쓸 것인가?

선택지:

```text
1. 새 바이트 버리기
2. 오래된 바이트 덮어쓰기
3. overflow flag 설정
4. interrupt disable 후 처리
```

질문:

```text
어떤 상황에서 어떤 정책이 더 안전한가?
```

---

## 과제 4. Count 방식으로 다시 구현하기

이번에는 한 칸을 비우지 말고 `count`를 사용해라.

```c
typedef struct {
    int data[CAPACITY];
    int front;
    int rear;
    int count;
} CircularQueue;
```

조건:

```text
empty: count == 0
full: count == CAPACITY
```

구현할 함수는 과제 1과 같다.

---

# 14. 핵심 정리

```text
Circular Queue의 본질은 “고정 배열을 원형처럼 사용해서 FIFO를 O(1)에 구현하는 것”이다.
```

| 개념       | 핵심                           |
| -------- | ---------------------------- |
| `front`  | 다음에 읽을 위치                    |
| `rear`   | 다음에 쓸 위치                     |
| modulo   | 끝에서 다시 처음으로 돌아가기             |
| empty 문제 | `front == rear`              |
| full 구분  | one slot waste, count, flag  |
| enqueue  | O(1)                         |
| dequeue  | O(1)                         |
| 장점       | 고정 메모리, 예측 가능한 성능            |
| 단점       | capacity 초과 시 overflow 정책 필요 |

## 반드시 기억할 문장

> Ring buffer는 “빠른 queue”가 아니라, **메모리 재할당 없이 일정한 시간에 데이터를 흘려보내는 실시간 버퍼 구조**다.