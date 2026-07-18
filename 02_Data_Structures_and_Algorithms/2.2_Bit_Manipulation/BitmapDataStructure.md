# Lecture 6. Bitmap Data Structure

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **1000개의 true/false 값을 꼭 1000 bytes 이상으로 저장해야 하는가?**

아니다.

`bool flags[1000]`처럼 저장하면 보통 최소 1000 bytes가 필요하다.

하지만 각 flag는 사실상 정보가 하나뿐이다.

```txt
true  / false
1     / 0
```

즉 flag 하나는 **1 bit**면 충분하다.

그래서 1000개의 flag는 이론적으로:

```txt
1000 bits = 125 bytes
```

면 저장할 수 있다.

이런 식으로 여러 boolean 상태를 bit 단위로 압축해서 저장하는 자료구조가 **bitmap**이다.

---

# 2. 형식적 정의

## 2.1 Bitmap이란?

**Bitmap**은 각 bit를 하나의 boolean flag처럼 사용하는 자료구조다.

```txt
bit = 0 → false
bit = 1 → true
```

예를 들어 8-bit bitmap 하나가 있다고 하자.

```txt
bitmap = 00101101
```

bit index를 오른쪽부터 세면:

```txt
bit index: 7 6 5 4 3 2 1 0
value:     0 0 1 0 1 1 0 1
```

의미:

```txt
flag 0 = true
flag 1 = false
flag 2 = true
flag 3 = true
flag 4 = false
flag 5 = true
flag 6 = false
flag 7 = false
```

---

## 2.2 Word-based bitmap

실무에서는 보통 `uint8_t`보다 `uint32_t` 또는 `uint64_t` 단위로 bitmap을 만든다.

예:

```c
uint32_t bitmap[32];
```

`uint32_t` 하나는 32 bits다.

따라서:

```txt
uint32_t bitmap[32]
= 32 words
= 32 * 32 bits
= 1024 bits
```

즉 1000개의 flag를 저장할 수 있다.

---

## 2.3 Index mapping

어떤 flag index `i`가 있을 때:

```txt
i = 73
```

이 flag가 bitmap 배열의 어느 word, 어느 bit에 들어가는지 계산해야 한다.

`uint32_t` 기준:

```c
word_index = i / 32;
bit_index  = i % 32;
```

또는 bit operation으로:

```c
word_index = i >> 5;      // i / 32
bit_index  = i & 31u;     // i % 32
```

왜냐하면:

```txt
32 = 2^5
```

이므로:

```txt
i / 32  → i >> 5
i % 32  → i & 31
```

하지만 처음에는 `/`와 `%`로 쓰는 것이 더 읽기 쉽다.
컴파일러가 상수 32에 대한 나눗셈과 나머지를 최적화할 수 있다.

---

# 3. 직관적 설명

일반 boolean 배열은 이렇게 생겼다고 볼 수 있다.

```txt
bool flags[8]

[true][false][true][true][false][true][false][false]
```

보통 각 칸이 최소 1 byte를 차지한다.

```txt
8 flags ≈ 8 bytes
```

하지만 bitmap은 이렇게 저장한다.

```txt
1 byte 안에 8개의 flag 저장

bit index: 7 6 5 4 3 2 1 0
value:     0 0 1 0 1 1 0 1
```

즉:

```txt
8 flags = 1 byte
```

비유하면:

> `bool[]`은 좌석 하나에 사람 하나씩 앉히는 방식이고, bitmap은 한 줄의 좌석 번호표를 bit로 압축해서 들고 있는 방식이다.

---

# 4. 왜 필요한지

Bitmap은 다음 상황에서 유용하다.

| 상황               | Bitmap이 유용한 이유                      |
| ---------------- | ----------------------------------- |
| 많은 boolean flag  | 메모리 절약                              |
| Embedded system  | RAM이 작기 때문에 flag 압축 중요              |
| OS kernel        | page frame 사용 여부 관리                 |
| Memory allocator | block 사용 여부 표시                      |
| Scheduler        | runnable task set 관리                |
| Filesystem       | inode/block 사용 여부 관리                |
| Network protocol | option/support flag 저장              |
| Game/server      | entity state, visited set           |
| Algorithm        | set membership, sieve, bloom filter |

Embedded에서는 특히 중요하다.

예를 들어 RAM이 8KB인 MCU에서 4000개의 센서 상태 flag를 저장한다고 하자.

`bool` 배열:

```txt
4000 bytes
```

bitmap:

```txt
4000 bits = 500 bytes
```

차이:

```txt
약 3500 bytes 절약
```

8KB RAM 시스템에서 3.5KB는 매우 큰 차이다.

---

# 5. 내부 원리

## 5.1 1000개 flag를 저장하려면 몇 word가 필요한가?

`uint32_t` 하나는 32개의 flag를 저장한다.

1000개의 flag가 필요하다면:

```txt
ceil(1000 / 32) = 32
```

왜냐하면:

```txt
31 words = 31 * 32 = 992 bits
32 words = 32 * 32 = 1024 bits
```

따라서:

```c
#define FLAG_COUNT 1000u
#define WORD_BITS  32u
#define WORD_COUNT ((FLAG_COUNT + WORD_BITS - 1u) / WORD_BITS)

uint32_t bitmap[WORD_COUNT];
```

계산:

```txt
WORD_COUNT = (1000 + 32 - 1) / 32
           = 1031 / 32
           = 32
```

C의 정수 나눗셈은 소수 부분을 버리므로 이 방식으로 올림 나눗셈을 구현한다.

---

## 5.2 특정 index의 word와 bit 계산

예를 들어 `i = 73`.

```txt
word_index = 73 / 32 = 2
bit_index  = 73 % 32 = 9
```

즉:

```txt
bitmap[2]의 bit 9
```

mask:

```c
uint32_t mask = UINT32_C(1) << 9;
```

시각화:

```txt
bitmap[0] → flags 0..31
bitmap[1] → flags 32..63
bitmap[2] → flags 64..95
                       ^
                       flag 73는 여기
```

`73 - 64 = 9`이므로 `bitmap[2]`의 bit 9다.

---

## 5.3 Set

flag `i`를 true로 만들려면:

```c
bitmap[word_index] |= mask;
```

예:

```txt
before = 00000000 00000000 00000000 00000000
mask   = 00000000 00000000 00000010 00000000
------------------------------------------------
after  = 00000000 00000000 00000010 00000000
```

---

## 5.4 Clear

flag `i`를 false로 만들려면:

```c
bitmap[word_index] &= ~mask;
```

예:

```txt
before = 00000000 00000000 00000010 00000000
mask   = 00000000 00000000 00000010 00000000
~mask  = 11111111 11111111 11111101 11111111
------------------------------------------------
after  = 00000000 00000000 00000000 00000000
```

---

## 5.5 Test

flag `i`가 true인지 확인하려면:

```c
(bitmap[word_index] & mask) != 0u
```

결과가 0이 아니면 해당 bit가 set되어 있다는 뜻이다.

---

# 6. 단계별 예시

## 예시 1: 기본 bitmap API

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

#define BITMAP_WORD_BITS 32u

static bool bitmap_index(size_t bit_count,
                         size_t index,
                         size_t *word_index,
                         unsigned *bit_index) {
    if (index >= bit_count || word_index == NULL || bit_index == NULL) {
        return false;
    }

    *word_index = index / BITMAP_WORD_BITS;
    *bit_index = (unsigned)(index % BITMAP_WORD_BITS);

    return true;
}

bool bitmap_set(uint32_t bitmap[], size_t bit_count, size_t index) {
    size_t word;
    unsigned bit;

    if (bitmap == NULL) {
        return false;
    }

    if (!bitmap_index(bit_count, index, &word, &bit)) {
        return false;
    }

    bitmap[word] |= UINT32_C(1) << bit;
    return true;
}

bool bitmap_clear(uint32_t bitmap[], size_t bit_count, size_t index) {
    size_t word;
    unsigned bit;

    if (bitmap == NULL) {
        return false;
    }

    if (!bitmap_index(bit_count, index, &word, &bit)) {
        return false;
    }

    bitmap[word] &= ~(UINT32_C(1) << bit);
    return true;
}

bool bitmap_test(const uint32_t bitmap[], size_t bit_count, size_t index, bool *out) {
    size_t word;
    unsigned bit;

    if (bitmap == NULL || out == NULL) {
        return false;
    }

    if (!bitmap_index(bit_count, index, &word, &bit)) {
        return false;
    }

    *out = (bitmap[word] & (UINT32_C(1) << bit)) != 0u;
    return true;
}
```

핵심은 이 세 줄이다.

```c
word = index / 32;
bit  = index % 32;
mask = UINT32_C(1) << bit;
```

나머지는 안전성 검사다.

---

## 예시 2: 1000개 flag bitmap

```c
#include <stdint.h>
#include <stddef.h>

#define FLAG_COUNT 1000u
#define WORD_BITS  32u
#define WORD_COUNT ((FLAG_COUNT + WORD_BITS - 1u) / WORD_BITS)

uint32_t flags[WORD_COUNT];
```

계산:

```txt
WORD_COUNT = 32
flags 크기 = 32 * sizeof(uint32_t)
           = 32 * 4
           = 128 bytes
```

정확히 1000 bits만 보면 125 bytes지만, `uint32_t` 단위로 잡기 때문에 128 bytes를 사용한다.

비교:

| 방식                 |               크기 |
| ------------------ | ---------------: |
| `bool flags[1000]` | 보통 최소 1000 bytes |
| `uint8_t` bitmap   |        125 bytes |
| `uint32_t` bitmap  |        128 bytes |
| `uint64_t` bitmap  |        128 bytes |

`uint32_t` bitmap은 3 bytes 정도 더 쓰지만, word 단위 연산이 자연스럽고 빠를 수 있다.

---

## 예시 3: 직접 set/test 해보기

```c
#include <stdint.h>
#include <stdio.h>
#include <stdbool.h>

#define FLAG_COUNT 1000u
#define WORD_BITS  32u
#define WORD_COUNT ((FLAG_COUNT + WORD_BITS - 1u) / WORD_BITS)

static uint32_t flags[WORD_COUNT];

static void set_flag(size_t index) {
    size_t word = index / WORD_BITS;
    unsigned bit = (unsigned)(index % WORD_BITS);

    flags[word] |= UINT32_C(1) << bit;
}

static bool test_flag(size_t index) {
    size_t word = index / WORD_BITS;
    unsigned bit = (unsigned)(index % WORD_BITS);

    return (flags[word] & (UINT32_C(1) << bit)) != 0u;
}

int main(void) {
    set_flag(0);
    set_flag(73);
    set_flag(999);

    printf("flag 0   = %d\n", test_flag(0));
    printf("flag 73  = %d\n", test_flag(73));
    printf("flag 100 = %d\n", test_flag(100));
    printf("flag 999 = %d\n", test_flag(999));

    return 0;
}
```

출력 개념:

```txt
flag 0   = 1
flag 73  = 1
flag 100 = 0
flag 999 = 1
```

이 예시는 단순하지만 `index` 범위 검사를 하지 않는다.
실무 API에서는 `index >= FLAG_COUNT`를 반드시 검사하는 것이 좋다.

---

## 예시 4: set된 flag 개수 세기

bitmap 전체에서 true인 flag 개수를 세려면 각 word의 popcount를 합치면 된다.

```c
#include <stdint.h>
#include <stddef.h>

static unsigned popcount32(uint32_t x) {
    unsigned count = 0;

    while (x != 0u) {
        x &= x - 1u;
        ++count;
    }

    return count;
}

size_t bitmap_count_set_bits(const uint32_t bitmap[], size_t word_count) {
    size_t count = 0;

    if (bitmap == NULL) {
        return 0;
    }

    for (size_t i = 0; i < word_count; ++i) {
        count += popcount32(bitmap[i]);
    }

    return count;
}
```

컴파일러 builtin을 쓰는 버전:

```c
static unsigned popcount32_fast(uint32_t x) {
#if defined(__GNUC__) || defined(__clang__)
    return (unsigned)__builtin_popcount(x);
#else
    unsigned count = 0;

    while (x != 0u) {
        x &= x - 1u;
        ++count;
    }

    return count;
#endif
}
```

주의:

`__builtin_popcount`는 ISO C 표준이 아니라 GCC/Clang 확장이다.
portable code가 필요하면 wrapper를 두는 편이 좋다.

---

# 7. 실제 응용

## 7.1 Embedded sensor availability map

센서가 100개 있다고 하자.

각 센서가 현재 활성 상태인지 저장해야 한다.

나쁜 방식은 아니다. 하지만 메모리를 많이 쓴다.

```c
bool sensor_active[100];
```

bitmap 방식:

```c
#define SENSOR_COUNT 100u
#define SENSOR_WORD_COUNT ((SENSOR_COUNT + 31u) / 32u)

uint32_t sensor_active[SENSOR_WORD_COUNT];
```

센서 42번 활성화:

```c
sensor_active[42u / 32u] |= UINT32_C(1) << (42u % 32u);
```

센서 42번 비활성화:

```c
sensor_active[42u / 32u] &= ~(UINT32_C(1) << (42u % 32u));
```

센서 42번 확인:

```c
bool active =
    (sensor_active[42u / 32u] & (UINT32_C(1) << (42u % 32u))) != 0u;
```

실무에서는 이런 식을 직접 반복하지 말고 함수로 감싼다.

---

## 7.2 OS page frame bitmap

운영체제는 물리 메모리 page가 사용 중인지 확인해야 한다.

예:

```txt
page 0: used
page 1: free
page 2: free
page 3: used
...
```

이를 bitmap으로 저장할 수 있다.

```txt
1 = used
0 = free
```

page allocator는 free page를 찾기 위해 bitmap을 순회한다.

```txt
bitmap word 안에서 0인 bit 찾기
→ 해당 page 할당
→ bit를 1로 set
```

이 구조는 OS, filesystem, allocator에서 매우 흔하다.

---

## 7.3 Scheduler ready set

작은 RTOS에서 task가 최대 32개라면 `uint32_t` 하나로 ready task를 표현할 수 있다.

```c
uint32_t ready_set;
```

task 5를 ready 상태로 만들기:

```c
ready_set |= UINT32_C(1) << 5;
```

task 5를 not-ready 상태로 만들기:

```c
ready_set &= ~(UINT32_C(1) << 5);
```

ready task가 하나라도 있는지 확인:

```c
if (ready_set != 0u) {
    // at least one task is ready
}
```

가장 우선순위 높은 ready task를 찾는 작업도 bit operation으로 빠르게 할 수 있다.
예를 들어 lowest bit가 highest priority라면 `ctz` 계열 연산을 쓸 수 있다.

```c
unsigned task_id = __builtin_ctz(ready_set);
```

주의:

`__builtin_ctz(0)`은 undefined behavior다.
따라서 반드시 먼저 `ready_set != 0`을 확인해야 한다.

```c
if (ready_set != 0u) {
    unsigned task_id = (unsigned)__builtin_ctz(ready_set);
}
```

---

## 7.4 Set membership

정수 ID가 작은 범위에 있다면 bitmap은 set처럼 쓸 수 있다.

예:

```txt
ID 범위: 0..999
```

ID가 set에 포함되어 있으면 bit를 1로 둔다.

```c
bitmap_set(seen, 1000, id);
```

이미 본 ID인지 확인:

```c
bool exists;
bitmap_test(seen, 1000, id, &exists);
```

이 방식의 장점:

```txt
insert: O(1)
lookup: O(1)
memory: 매우 작음
```

단점:

```txt
ID 범위가 너무 크면 bitmap도 커짐
음수나 큰 sparse key에는 부적합
```

예를 들어 ID가 `0..4,000,000,000` 범위라면 bitmap은 너무 커질 수 있다.

---

## 7.5 Bloom filter 맛보기

Bloom filter는 bitmap을 기반으로 하는 probabilistic set 자료구조다.

아이디어는 간단하다.

```txt
1. 원소를 여러 hash function에 넣는다.
2. 나온 위치들의 bit를 1로 set한다.
3. 조회할 때 같은 위치들이 모두 1인지 확인한다.
```

특징:

| 항목             | 의미         |
| -------------- | ---------- |
| False negative | 없음         |
| False positive | 있을 수 있음    |
| 삭제             | 기본형에서는 어려움 |
| 메모리            | 매우 효율적     |

즉:

```txt
"없다"는 확실히 말할 수 있음
"있다"는 아마 있다고 말함
```

예를 들어 웹 크롤러가 URL을 이미 방문했는지 빠르게 대략 확인하거나, 데이터베이스가 어떤 key가 없다는 것을 빠르게 판단하는 데 사용할 수 있다.

Bloom filter는 bitmap의 고급 응용이다.

---

# 8. 성능과 메모리 관점

## 8.1 메모리 절약

1000개 flag 기준:

```txt
bool[1000]      ≈ 1000 bytes
uint32_t bitmap = 128 bytes
```

절약량:

```txt
872 bytes 절약
```

비율:

```txt
약 87.2% 절약
```

Embedded에서는 이 차이가 직접적으로 중요하다.

---

## 8.2 Cache locality

작은 데이터는 cache에 더 많이 들어간다.

예를 들어 cache line이 64 bytes라고 하면:

```txt
bool array:
64 flags per cache line

bitmap:
64 bytes = 512 bits
512 flags per cache line
```

즉 같은 cache line 안에 훨씬 많은 flag가 들어간다.

장점:

```txt
순차적으로 많은 flag를 검사할 때 cache miss 감소 가능
```

하지만 trade-off도 있다.

```txt
bool array:
flags[i] 접근이 단순함

bitmap:
word 계산 + shift + mask 필요
```

따라서 bitmap이 항상 빠르지는 않다.
메모리 절약과 cache locality가 이득인 상황에서 강하다.

---

## 8.3 Word 단위 연산의 장점

bitmap은 bit 하나씩 처리하지 않고 word 단위로 처리할 수 있다.

예:

```c
if (bitmap[word] == 0u) {
    // 이 word 안의 32개 flag는 모두 false
}
```

이렇게 하면 32개의 flag를 한 번에 건너뛸 수 있다.

모두 true인지 확인:

```c
if (bitmap[word] == UINT32_MAX) {
    // 이 word 안의 32개 flag는 모두 true
}
```

단, 마지막 word에는 실제 flag 범위를 넘어서는 padding bit가 있을 수 있다.

1000 flags라면 1024 bits 공간을 쓰므로 마지막 24 bits는 unused다.

```txt
1024 - 1000 = 24 unused bits
```

이 padding bit들을 처리할 때 주의해야 한다.

---

## 8.4 동시성 관점

Embedded에서 bitmap을 interrupt handler와 main loop가 동시에 접근할 수 있다.

예:

```c
main loop:        bitmap_clear(...)
interrupt:        bitmap_set(...)
```

이때 다음 연산은 atomic하지 않을 수 있다.

```c
bitmap[word] |= mask;
```

실제로는 read-modify-write다.

```txt
1. memory에서 word 읽기
2. OR 연산
3. memory에 word 쓰기
```

중간에 interrupt가 끼면 update가 손실될 수 있다.

해결 방법은 상황에 따라 다르다.

| 상황                | 해결                                          |
| ----------------- | ------------------------------------------- |
| single-core MCU   | critical section, interrupt disable         |
| RTOS              | mutex, spinlock, critical section           |
| multi-core        | atomic operation                            |
| hardware register | vendor manual의 atomic set/clear register 사용 |

중요한 점:

> `volatile`은 atomic을 보장하지 않는다.

`volatile`은 compiler 최적화 제어에 가깝다.
동시성 안전성을 보장하는 도구가 아니다.

---

# 9. 흔한 오해

## 오해 1: Bitmap은 항상 bool array보다 빠르다

아니다.

Bitmap은 메모리를 적게 쓰고 cache locality가 좋을 수 있다.
하지만 개별 bit 접근에는 계산이 필요하다.

```c
word = index / 32;
bit = index % 32;
mask = 1u << bit;
```

반면 `bool array`는 단순하다.

```c
flags[index]
```

따라서 flag 수가 적고 메모리가 충분하다면 `bool array`가 더 명확하고 충분히 빠를 수 있다.

---

## 오해 2: Bitmap의 bit order는 endianness와 같다

다르다.

Bitmap에서 우리가 정하는 bit index는 **논리적 numbering**이다.

```txt
bit 0, bit 1, bit 2 ...
```

Endianness는 multi-byte integer가 메모리에 어떤 byte 순서로 저장되는지의 문제다.

예를 들어:

```c
uint32_t word = 1u;
```

논리적으로 bit 0이 set되어 있다는 사실은 같다.

하지만 메모리 byte 순서는 little-endian과 big-endian에서 다를 수 있다.

```txt
논리 bit 연산: 동일
메모리 직렬화: endian 고려 필요
```

즉 bitmap을 파일이나 네트워크로 저장/전송할 때는 byte order와 bit numbering 규칙을 명확히 정해야 한다.

---

## 오해 3: `1 << bit`는 안전하다

반복해서 강조하지만 위험할 수 있다.

```c
uint32_t mask = 1 << bit;
```

`1`은 signed `int`다.

더 안전한 코드:

```c
uint32_t mask = UINT32_C(1) << bit;
```

그리고 `bit < 32`도 보장해야 한다.

```c
if (bit >= 32u) {
    return false;
}
```

---

## 오해 4: 마지막 word의 unused bit는 신경 쓰지 않아도 된다

경우에 따라 신경 써야 한다.

1000 flags를 `uint32_t[32]`에 저장하면 1024 bits가 있다.

마지막 24 bits는 실제 flag가 아니다.

문제 상황:

```c
if (bitmap[31] == UINT32_MAX) {
    // 마지막 32개 flag가 모두 true?
}
```

하지만 마지막 word에는 실제 flag가 8개뿐이다.

```txt
flags 992..999  → 실제 사용
flags 1000..1023 → unused
```

따라서 전체 bitmap에서 “모든 flag가 set되었는지” 검사할 때는 마지막 word를 특별히 처리해야 한다.

---

# 10. 반례 또는 실패 사례

## 실패 사례 1: index 범위 검사 누락

위험한 코드:

```c
void bitmap_set_bad(uint32_t bitmap[], size_t index) {
    size_t word = index / 32u;
    unsigned bit = (unsigned)(index % 32u);

    bitmap[word] |= UINT32_C(1) << bit;
}
```

문제:

```c
bitmap_set_bad(bitmap, 999999);
```

배열 범위를 넘어선 memory write가 발생한다.

해결:

```c
if (index >= bit_count) {
    return false;
}
```

---

## 실패 사례 2: shift count overflow

위험한 코드:

```c
uint32_t mask = UINT32_C(1) << bit;
```

`bit >= 32`이면 undefined behavior다.

bitmap에서는 보통:

```c
bit = index % 32;
```

로 계산하므로 `bit`는 0..31 범위다.

하지만 외부에서 bit 번호를 직접 받는 API라면 반드시 검사해야 한다.

---

## 실패 사례 3: `uint8_t`에서 promotion 착각

```c
uint8_t bitmap[16];
uint8_t mask = 1u << bit;
```

이 코드도 보통 동작하지만, C에서는 작은 정수형이 `int`로 promotion된다는 점을 알아야 한다.

더 명확한 스타일:

```c
uint8_t mask = (uint8_t)(UINT8_C(1) << bit);
```

단, `UINT8_C`는 실제 타입이 promotion될 수 있으므로, 실무에서는 word type을 `uint32_t`로 두는 것이 더 단순할 때가 많다.

---

## 실패 사례 4: `volatile`만 붙이면 안전하다고 생각

```c
volatile uint32_t flags;

void set_flag(uint32_t mask) {
    flags |= mask;
}
```

이 코드는 compiler가 access를 제거하지 못하게 할 수는 있다.

하지만 다음을 보장하지 않는다.

```txt
atomicity
race-free update
interrupt-safe update
multi-core synchronization
```

특히 `flags |= mask`는 read-modify-write다.

interrupt와 공유한다면 critical section이 필요할 수 있다.

---

## 실패 사례 5: signed type으로 bitmap 구현

나쁜 스타일:

```c
int bitmap[32];
bitmap[word] |= 1 << bit;
```

문제:

```txt
1. sign bit shift 위험
2. signed overflow/representation 오해
3. right shift와 결합 시 portability 문제
4. bit pattern 의도가 불명확
```

권장:

```c
uint32_t bitmap[32];
bitmap[word] |= UINT32_C(1) << bit;
```

---

# 11. Portability-safe coding style

## 11.1 기본 상수

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

#define BITMAP_WORD_BITS 32u
```

`uint32_t`를 사용할 때만 32로 둔다.

더 일반적으로는:

```c
#include <limits.h>

#define BITMAP_WORD_BITS (sizeof(uint32_t) * CHAR_BIT)
```

다만 `uint32_t`는 존재한다면 정확히 32 bits다.
그래도 `CHAR_BIT` 기반 표현은 의도를 명확히 보여준다.

---

## 11.2 안전한 bitmap 구조체

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

typedef struct {
    uint32_t *words;
    size_t bit_count;
    size_t word_count;
} Bitmap;
```

초기화는 사용자가 배열을 제공하는 방식으로 할 수 있다.

```c
bool bitmap_init(Bitmap *bm,
                 uint32_t storage[],
                 size_t bit_count,
                 size_t word_count) {
    if (bm == NULL || storage == NULL || bit_count == 0u) {
        return false;
    }

    size_t required_words = (bit_count + 31u) / 32u;

    if (word_count < required_words) {
        return false;
    }

    bm->words = storage;
    bm->bit_count = bit_count;
    bm->word_count = required_words;

    for (size_t i = 0; i < required_words; ++i) {
        bm->words[i] = 0u;
    }

    return true;
}
```

이런 구조로 만들면 `bit_count`와 `word_count`를 함께 관리할 수 있어서 범위 오류를 줄인다.

---

## 11.3 구조체 기반 set/test

```c
bool bitmap_set_bm(Bitmap *bm, size_t index) {
    if (bm == NULL || bm->words == NULL || index >= bm->bit_count) {
        return false;
    }

    size_t word = index / 32u;
    unsigned bit = (unsigned)(index % 32u);

    bm->words[word] |= UINT32_C(1) << bit;
    return true;
}

bool bitmap_test_bm(const Bitmap *bm, size_t index, bool *out) {
    if (bm == NULL || bm->words == NULL || out == NULL || index >= bm->bit_count) {
        return false;
    }

    size_t word = index / 32u;
    unsigned bit = (unsigned)(index % 32u);

    *out = (bm->words[word] & (UINT32_C(1) << bit)) != 0u;
    return true;
}
```

---

# 12. 확인 문제

## 문제 1

1000개의 boolean flag를 저장하려고 한다.

```txt
bool array 방식은 대략 몇 bytes?
bitmap은 이론적으로 몇 bytes?
uint32_t bitmap은 몇 bytes?
```

---

## 문제 2

`uint32_t bitmap[]`에서 flag index가 `73`일 때:

```txt
word_index = ?
bit_index = ?
mask = ?
```

---

## 문제 3

다음 코드는 어떤 일을 하는가?

```c
bitmap[index / 32u] |= UINT32_C(1) << (index % 32u);
```

---

## 문제 4

다음 코드의 문제점은 무엇인가?

```c
void set_bad(uint32_t bitmap[], size_t index) {
    bitmap[index / 32u] |= UINT32_C(1) << (index % 32u);
}
```

---

## 문제 5

`volatile uint32_t flags;`에 대해 다음 코드가 atomic하지 않을 수 있는 이유를 설명해라.

```c
flags |= mask;
```

---

## 문제 6

Bitmap에서 bit numbering과 endianness는 같은 개념인가?

정확히 구분해서 설명해라.

---

# 13. 실습 과제

## 실습 1: 1000개 flag bitmap 구현

다음 요구사항을 만족하는 코드를 작성해라.

```txt
1. 1000개의 flag 저장
2. set, clear, test 함수 제공
3. index가 범위를 벗어나면 false 반환
```

예상 구조:

```c
#include <stdint.h>
#include <stddef.h>
#include <stdbool.h>

#define FLAG_COUNT 1000u
#define WORD_BITS  32u
#define WORD_COUNT ((FLAG_COUNT + WORD_BITS - 1u) / WORD_BITS)

static uint32_t flags[WORD_COUNT];

bool flag_set(size_t index) {
    if (index >= FLAG_COUNT) {
        return false;
    }

    size_t word = index / WORD_BITS;
    unsigned bit = (unsigned)(index % WORD_BITS);

    flags[word] |= UINT32_C(1) << bit;
    return true;
}

bool flag_clear(size_t index) {
    if (index >= FLAG_COUNT) {
        return false;
    }

    size_t word = index / WORD_BITS;
    unsigned bit = (unsigned)(index % WORD_BITS);

    flags[word] &= ~(UINT32_C(1) << bit);
    return true;
}

bool flag_test(size_t index, bool *out) {
    if (out == NULL || index >= FLAG_COUNT) {
        return false;
    }

    size_t word = index / WORD_BITS;
    unsigned bit = (unsigned)(index % WORD_BITS);

    *out = (flags[word] & (UINT32_C(1) << bit)) != 0u;
    return true;
}
```

---

## 실습 2: bitmap count 함수 구현

현재 set된 flag 개수를 세는 함수를 작성해라.

```c
size_t flag_count_set(void);
```

예상 구현:

```c
static unsigned popcount32(uint32_t x) {
    unsigned count = 0;

    while (x != 0u) {
        x &= x - 1u;
        ++count;
    }

    return count;
}

size_t flag_count_set(void) {
    size_t count = 0;

    for (size_t i = 0; i < WORD_COUNT; ++i) {
        count += popcount32(flags[i]);
    }

    return count;
}
```

단, 이 구현은 마지막 word의 unused bit가 0으로 유지된다는 가정이 있다.
만약 unused bit가 set될 가능성이 있다면 마지막 word mask 처리가 필요하다.

---

## 실습 3: 마지막 word mask 처리

`FLAG_COUNT = 1000`일 때 마지막 word에서 실제 사용하는 bit 수는:

```txt
1000 % 32 = 8
```

따라서 마지막 word에서는 하위 8 bits만 실제 flag다.

mask:

```c
#define LAST_USED_BITS (FLAG_COUNT % WORD_BITS)

#if LAST_USED_BITS == 0
#define LAST_WORD_MASK UINT32_MAX
#else
#define LAST_WORD_MASK ((UINT32_C(1) << LAST_USED_BITS) - 1u)
#endif
```

마지막 word를 count할 때:

```c
size_t flag_count_set_strict(void) {
    size_t count = 0;

    if (WORD_COUNT == 0u) {
        return 0u;
    }

    for (size_t i = 0; i + 1u < WORD_COUNT; ++i) {
        count += popcount32(flags[i]);
    }

    count += popcount32(flags[WORD_COUNT - 1u] & LAST_WORD_MASK);

    return count;
}
```

이렇게 하면 unused bit가 실수로 set되어도 count에 포함되지 않는다.

---

## 실습 4: ID membership set

ID 범위가 `0..999`라고 하자.

다음 함수를 작성해라.

```c
bool id_add(size_t id);
bool id_remove(size_t id);
bool id_contains(size_t id, bool *out);
```

내부적으로는 실습 1의 bitmap 함수를 재사용하면 된다.

```c
bool id_add(size_t id) {
    return flag_set(id);
}

bool id_remove(size_t id) {
    return flag_clear(id);
}

bool id_contains(size_t id, bool *out) {
    return flag_test(id, out);
}
```

이 구조는 작은 정수 ID set을 구현할 때 매우 효율적이다.

---

# 14. 핵심 정리

이번 강의 핵심은 다음이다.

1. **Bitmap은 각 bit를 하나의 boolean flag로 사용하는 자료구조다.**
2. **1개의 `uint32_t` word는 32개의 flag를 저장할 수 있다.**
3. **index `i`의 위치는 `word = i / 32`, `bit = i % 32`로 계산한다.**
4. **set은 `bitmap[word] |= mask`다.**
5. **clear는 `bitmap[word] &= ~mask`다.**
6. **test는 `(bitmap[word] & mask) != 0`이다.**
7. **1000개 flag는 이론적으로 125 bytes, `uint32_t` bitmap으로는 128 bytes면 충분하다.**
8. **Bitmap은 메모리 절약과 cache locality에 강점이 있다.**
9. **개별 접근은 bool array보다 복잡할 수 있으므로 항상 더 빠른 것은 아니다.**
10. **마지막 word의 unused bit를 조심해야 한다.**
11. **`volatile`은 atomic을 보장하지 않는다.**
12. **Embedded에서 interrupt와 공유하는 bitmap은 critical section 또는 atomic 처리가 필요할 수 있다.**
13. **Bitmap의 논리적 bit numbering과 endianness는 다른 개념이다.**
14. **Bit manipulation은 unsigned fixed-width type으로 작성하는 것이 안전하다.**

---

# 15. 면접 대비 핵심 문장

면접에서는 이렇게 말하면 된다.

> A bitmap is a compact data structure that stores boolean flags using individual bits. For a `uint32_t` bitmap, the word index is `i / 32`, the bit index is `i % 32`, and the mask is `1u << bit_index`.

한국어로는:

> Bitmap은 boolean flag를 bit 단위로 압축해서 저장하는 자료구조입니다. `uint32_t` 기반 bitmap에서는 index `i`에 대해 `word_index = i / 32`, `bit_index = i % 32`, `mask = 1u << bit_index`로 위치를 계산합니다.

Embedded 관점에서는 이렇게 설명하면 좋다.

> Bitmap is useful in embedded systems because it can reduce memory usage significantly. For example, 1000 boolean flags can be stored in about 125 bytes instead of around 1000 bytes.

한국어로는:

> Embedded system에서는 RAM이 작기 때문에 bitmap이 유용합니다. 예를 들어 1000개의 boolean flag를 `bool` 배열로 저장하면 보통 1000 bytes 정도가 필요하지만, bitmap으로는 약 125 bytes, `uint32_t` 배열 기준으로는 128 bytes면 충분합니다.

주의점까지 포함하면:

> However, bitmap operations involve bit calculation and read-modify-write updates, so bounds checking, unsigned types, shift-count safety, and concurrency issues must be handled carefully.

한국어로는:

> 다만 bitmap은 index 계산, shift, mask, read-modify-write가 필요하므로 범위 검사, unsigned type 사용, shift count 안전성, interrupt나 multi-thread 환경에서의 동시성 문제를 주의해야 합니다.

---