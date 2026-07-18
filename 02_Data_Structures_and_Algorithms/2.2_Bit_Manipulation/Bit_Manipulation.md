## Part 2.2 전체 커리큘럼

| Lecture | 주제                                       | 핵심 목표                                    |
| ------: | ---------------------------------------- | ---------------------------------------- |
|       1 | Bit Manipulation을 왜 배우는가                 | CPU, 메모리, C integer를 bit pattern으로 이해    |
|       2 | XOR Operations                           | XOR의 성질과 “차이”라는 직관                       |
|       3 | Power-of-Two Operations                  | `x & (x - 1)`, popcount, power-of-two 판별 |
|       4 | Odd/Even, Shift, Arithmetic Optimization | shift와 산술 최적화의 실제 의미                     |
|       5 | Bit Masking                              | set/clear/toggle/check bit와 register 제어  |
|       6 | Bitmap Data Structure                    | boolean flag를 1 bit로 압축 저장               |
|       7 | Two’s Complement                         | 음수 표현과 signed overflow 위험                |
|       8 | Endianness                               | byte order, network/file/embedded 의미     |
|       9 | Shift Operations in Depth                | logical/arithmetic shift, portability    |
|      10 | Advanced Bit Tricks                      | branchless, MSB, Gray code, builtin      |
|      11 | Bit Fields in Struct                     | C bit-field와 portability 문제              |
|      12 | 64-bit Operations in 32-bit Environment  | 32-bit MCU에서 64-bit 연산 비용                |
|      13 | Interview Patterns                       | 대표 면접 문제 구현                              |
|      14 | Final Review                             | 언제 bit trick을 쓰고, 언제 피할지 판단              |

---

# Lecture 1. Bit Manipulation을 왜 배우는가

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **C에서 정수는 단순한 숫자인가, 아니면 메모리 위의 bit pattern인가?**

답은 둘 다다.

C 코드에서는 `int x = 5;`처럼 숫자로 보인다.
하지만 CPU와 메모리 입장에서는 결국 다음과 같은 bit pattern이다.

```txt
5  ==  00000101
```

Bit manipulation은 이 bit pattern을 직접 다루는 기술이다.

---

## 2. 형식적 정의

### Bit

**bit**는 컴퓨터가 표현할 수 있는 가장 작은 정보 단위다.

```txt
0 또는 1
```

### Byte

일반적으로 1 byte는 8 bit다.

```txt
1 byte = 8 bits
```

예:

```txt
01010110
```

### Integer as bit pattern

C에서 정수형 객체는 메모리에 일정한 크기의 bit sequence로 저장된다.

예를 들어 `uint8_t x = 5;`라면 보통 다음과 같이 생각할 수 있다.

```txt
decimal: 5
binary : 00000101
hex    : 0x05
```

### Unsigned integer

`unsigned` 정수는 bit pattern을 **0 이상의 정수**로 해석한다.

8-bit unsigned라면:

```txt
00000000 = 0
00000001 = 1
01111111 = 127
10000000 = 128
11111111 = 255
```

즉 범위는:

```txt
0 ~ 2^N - 1
```

`N`은 bit 수다.

### Signed integer

`signed` 정수는 양수와 음수를 모두 표현한다.

현대 시스템에서는 거의 항상 **two’s complement**를 사용한다.
예를 들어 8-bit signed라면 보통:

```txt
00000101 = 5
11111011 = -5
```

다만 중요한 점이 있다.

C에서 **unsigned 연산은 modulo 방식으로 wrap-around가 정의되어 있지만**, **signed integer overflow는 undefined behavior**로 취급된다. WG14 문서에서도 unsigned 연산은 표현 범위를 넘으면 modulo/wrap-around 성격을 갖는다고 설명하고, signed overflow는 undefined behavior라는 전제를 유지한다. ([Open Standards][1])

---

## 3. 직관적 설명

정수를 “숫자”로만 보면 이렇게 보인다.

```c
int x = 5;
```

하지만 시스템 프로그래밍에서는 이렇게 봐야 한다.

```txt
x = 00000000 00000000 00000000 00000101
```

즉, 정수는 실제로는 **bit들의 묶음**이다.

비트 연산은 이 묶음에서 특정 bit를 켜거나, 끄거나, 뒤집거나, 검사하는 기술이다.

예를 들어 embedded system에서 센서 상태 register가 다음과 같다고 하자.

```txt
status register = 00001010
```

각 bit의 의미가 다음과 같을 수 있다.

```txt
bit 0: sensor ready
bit 1: error occurred
bit 2: data available
bit 3: overheat warning
```

그러면 `00001010`은 다음 뜻이다.

```txt
bit 1 = 1 → error occurred
bit 3 = 1 → overheat warning
```

즉, 이 값은 단순히 decimal `10`이 아니라:

> “에러가 발생했고, 과열 경고가 켜져 있다.”

라는 상태 정보다.

---

## 4. 왜 필요한지

Bit manipulation은 다음 분야에서 거의 필수다.

| 분야                      | 왜 필요한가                                       |
| ----------------------- | -------------------------------------------- |
| Embedded system         | 하드웨어 register 제어                             |
| OS                      | permission, flag, page table, interrupt mask |
| Networking              | packet header parsing, endian conversion     |
| Graphics                | color channel, pixel packing                 |
| Compression             | 데이터를 bit 단위로 압축                              |
| Cryptography            | XOR, rotation, masking                       |
| Data structure          | bitmap, bloom filter                         |
| Performance engineering | 메모리 절약, cache locality 개선                    |

특히 embedded에서는 bit manipulation을 모르면 하드웨어를 제대로 제어하기 어렵다.

예를 들어 MCU의 GPIO register에서 특정 pin만 켜고 싶다면:

```c
GPIO_OUT |= (1u << 5);
```

이 코드는 “5번 bit를 1로 만들어라”라는 뜻이다.

즉:

```txt
before: 00000000
mask  : 00100000
after : 00100000
```

---

## 5. 내부 원리

## 5.1 CPU는 값을 register에 올려서 연산한다

C 코드:

```c
uint32_t x = 5;
uint32_t y = x & 3;
```

CPU 관점:

```txt
x: 00000000 00000000 00000000 00000101
3: 00000000 00000000 00000000 00000011
---------------------------------------
&: 00000000 00000000 00000000 00000001
```

결과:

```txt
y = 1
```

왜냐하면 AND는 둘 다 1인 bit만 1로 남기기 때문이다.

---

## 5.2 메모리는 byte 단위로 주소를 가진다

대부분의 시스템에서 메모리는 byte 단위로 주소가 붙는다.

```txt
address 0x1000: 00000101
address 0x1001: 00000000
address 0x1002: 00000000
address 0x1003: 00000000
```

`uint32_t x = 5;`가 little-endian 시스템에 저장되면 보통 이렇게 저장된다.

```txt
x = 0x00000005

low address → high address

0x1000: 05
0x1001: 00
0x1002: 00
0x1003: 00
```

Endianness는 Lecture 8에서 자세히 다룬다.

지금은 이것만 기억하면 된다.

> **정수는 메모리에서 byte들의 배열이고, 각 byte는 bit들의 배열이다.**

---

## 5.3 C type은 “몇 bit를 어떻게 해석할지”를 정한다

같은 bit pattern도 type에 따라 다르게 해석된다.

예를 들어 8-bit pattern이 있다고 하자.

```txt
11111111
```

이를 `uint8_t`로 보면:

```txt
255
```

이를 `int8_t`로 보면, two’s complement 시스템에서는:

```txt
-1
```

즉, bit pattern 자체는 같다.

```txt
11111111
```

하지만 해석이 다르다.

| Type      | Interpretation |
| --------- | -------------: |
| `uint8_t` |            255 |
| `int8_t`  |             -1 |
| raw bits  |     `11111111` |

이것이 bit-level thinking의 핵심이다.

---

## 6. 단계별 예시

## 예시 1: 숫자를 bit로 보기

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint8_t x = 5;

    printf("x = %u\n", x);
    printf("x & 1 = %u\n", x & 1u);

    return 0;
}
```

`x = 5`는 binary로:

```txt
00000101
```

`x & 1`은:

```txt
  00000101
& 00000001
----------
  00000001
```

결과는 `1`.

즉:

```c
x & 1
```

은 최하위 bit를 검사한다.

최하위 bit가 1이면 홀수, 0이면 짝수다.

---

## 예시 2: flag 저장하기

여러 상태를 각각 `bool` 변수로 저장할 수도 있다.

```c
#include <stdbool.h>

bool ready = true;
bool error = false;
bool data_available = true;
bool overheat = false;
```

하지만 이를 1개의 byte에 담을 수도 있다.

```c
#include <stdint.h>

#define FLAG_READY          (1u << 0)
#define FLAG_ERROR          (1u << 1)
#define FLAG_DATA_AVAILABLE (1u << 2)
#define FLAG_OVERHEAT       (1u << 3)

int main(void) {
    uint8_t status = 0;

    status |= FLAG_READY;
    status |= FLAG_DATA_AVAILABLE;

    if (status & FLAG_READY) {
        // ready flag is set
    }

    return 0;
}
```

상태 변화:

```txt
초기 상태:
00000000

READY 설정:
00000001

DATA_AVAILABLE 설정:
00000101
```

즉, `status = 00000101`은:

```txt
READY = 1
ERROR = 0
DATA_AVAILABLE = 1
OVERHEAT = 0
```

---

## 예시 3: 잘못된 코드와 안전한 코드

위험한 코드:

```c
int mask = 1 << 31;
```

왜 위험한가?

`1`은 기본적으로 signed `int`다.
32-bit `int` 환경에서 `1 << 31`은 sign bit 위치로 shift한다. 이 결과가 `int`로 표현 불가능하면 undefined behavior가 될 수 있다.

더 안전한 코드:

```c
#include <stdint.h>

uint32_t mask = 1u << 31;
```

더 명확한 코드:

```c
uint32_t mask = UINT32_C(1) << 31;
```

핵심은 이것이다.

> Bit manipulation에서는 기본적으로 `unsigned` type을 사용하라.

---

## 7. 실제 응용

## 7.1 Embedded register control

가상의 하드웨어 register가 있다고 하자.

```c
#define UART_CTRL_ENABLE   (1u << 0)
#define UART_CTRL_TX_INT   (1u << 1)
#define UART_CTRL_RX_INT   (1u << 2)

volatile uint32_t UART_CTRL;
```

UART를 enable하려면:

```c
UART_CTRL |= UART_CTRL_ENABLE;
```

TX interrupt를 끄려면:

```c
UART_CTRL &= ~UART_CTRL_TX_INT;
```

RX interrupt가 켜져 있는지 확인하려면:

```c
if (UART_CTRL & UART_CTRL_RX_INT) {
    // RX interrupt enabled
}
```

여기서 `volatile`은 중요하다.

하드웨어 register 값은 프로그램 바깥, 즉 하드웨어에 의해 바뀔 수 있다.
그래서 compiler에게 “이 값은 마음대로 최적화해서 제거하지 말라”고 알려줘야 한다.

---

## 7.2 Memory saving

1000개의 boolean flag를 저장한다고 하자.

일반 배열:

```c
bool flags[1000];
```

대부분의 환경에서 `bool`은 최소 1 byte를 사용한다.

```txt
1000 flags ≈ 1000 bytes
```

Bitmap을 쓰면:

```txt
1000 bits = 125 bytes
```

메모리를 약 1/8로 줄일 수 있다.

Embedded system에서는 RAM이 2KB, 8KB, 32KB처럼 매우 작을 수 있다.
이때 875 bytes 차이는 매우 크다.

---

## 7.3 Cache locality

메모리를 적게 쓰면 cache에 더 많은 데이터를 넣을 수 있다.

```txt
bool array:
[1 byte][1 byte][1 byte][1 byte] ...

bitmap:
[bit][bit][bit][bit][bit][bit][bit][bit] ...
```

작은 데이터는 cache miss를 줄일 수 있다.

다만 주의해야 한다.

Bitmap은 메모리는 아끼지만, bit 추출 연산이 필요하다.

```c
word = bitmap[i / 32];
bit  = (word >> (i % 32)) & 1u;
```

따라서 항상 빠른 것은 아니다.

> 메모리 절약, cache locality, 코드 복잡도 사이의 trade-off를 봐야 한다.

---

## 8. 성능과 메모리 관점

Bit operation은 보통 CPU에서 매우 빠른 primitive instruction으로 매핑된다.

예:

```c
x & mask
x | mask
x ^ mask
x << n
```

하지만 이것을 무조건 “직접 쓰면 빠르다”고 생각하면 안 된다.

현대 compiler는 이미 다음과 같은 최적화를 잘 한다.

```c
x * 8
```

를 내부적으로:

```c
x << 3
```

와 유사한 instruction으로 바꿀 수 있다.

그래서 다음 코드는 대부분 직접 쓸 필요가 없다.

```c
x = x << 3;  // 무조건 좋은 스타일 아님
```

대신 의도가 산술이면 이렇게 쓰는 편이 좋다.

```c
x = x * 8;
```

의도가 bit field 조작이면 shift를 쓴다.

```c
mask = 1u << bit_index;
```

정리하면:

| 상황                      | 추천                |                            |
| ----------------------- | ----------------- | -------------------------- |
| 산술 의미                   | `*`, `/`, `%` 사용  |                            |
| bit field 의미            | `&`, `            | `, `^`, `~`, `<<`, `>>` 사용 |
| compiler가 쉽게 최적화 가능     | readability 우선    |                            |
| 하드웨어 register 제어        | manual masking 필요 |                            |
| protocol/header parsing | manual masking 필요 |                            |

---

## 9. 흔한 오해

## 오해 1: `^`는 제곱이다

C에서 `^`는 제곱이 아니다.

```c
2 ^ 3
```

이것은 `2`의 `3`제곱이 아니다.
XOR 연산이다.

```txt
2 = 0010
3 = 0011
---------
^ = 0001
```

결과는 `1`.

---

## 오해 2: `&`와 `&&`는 비슷하다

전혀 다르다.

| 연산자  | 의미          |
| ---- | ----------- |
| `&`  | bitwise AND |
| `&&` | logical AND |

예:

```c
int a = 2;  // 0010
int b = 1;  // 0001

printf("%d\n", a & b);   // 0
printf("%d\n", a && b);  // 1
```

`a & b`:

```txt
0010
0001
----
0000
```

`a && b`:

```txt
a is non-zero, b is non-zero → true
```

---

## 오해 3: signed integer도 overflow하면 자연스럽게 wrap-around된다

많은 실제 CPU에서는 signed overflow가 two’s complement wrap처럼 보일 수 있다.

하지만 C 언어 관점에서는 signed overflow가 undefined behavior다.

위험한 코드:

```c
int x = 2147483647;
x = x + 1;  // undefined behavior
```

안전하게 wrap-around를 원한다면:

```c
#include <stdint.h>

uint32_t x = UINT32_MAX;
x = x + 1u;  // defined: 0
```

---

## 오해 4: right shift는 항상 2로 나누기다

`unsigned`에서는 보통 맞다.

```c
uint32_t x = 8;
x >> 1;  // 4
```

하지만 signed 음수에서는 조심해야 한다.

```c
int x = -8;
x >> 1;
```

음수 signed right shift는 구현에 따라 동작이 달라질 수 있는 영역이다.
일반적으로 arithmetic shift가 쓰이지만, portable C 코드에서는 이 가정에 의존하지 않는 편이 좋다.

---

## 10. 반례 또는 실패 사례

## 실패 사례 1: shift count가 type width 이상

```c
uint32_t x = 1u << 32;
```

이 코드는 위험하다.

`uint32_t`는 32-bit다.
shift count가 32 이상이면 undefined behavior다.

안전하게 하려면:

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask(uint32_t *out, unsigned bit) {
    if (bit >= 32) {
        return false;
    }

    *out = UINT32_C(1) << bit;
    return true;
}
```

---

## 실패 사례 2: signed와 unsigned 비교

```c
int i = -1;
unsigned int u = 1;

if (i < u) {
    printf("i is smaller\n");
} else {
    printf("i is not smaller\n");
}
```

직관적으로는 `-1 < 1`이므로 true일 것 같다.

하지만 C에서는 usual arithmetic conversions 때문에 `i`가 unsigned로 변환될 수 있다.
그러면 `-1`이 매우 큰 unsigned 값으로 바뀐다.

따라서 결과가 직관과 다를 수 있다.

안전한 스타일:

```c
int i = -1;
int u_as_int = 1;

if (i < u_as_int) {
    printf("i is smaller\n");
}
```

또는 애초에 비교 대상의 signedness를 맞춘다.

---

## 실패 사례 3: 하드웨어 register에서 `volatile` 누락

위험한 코드:

```c
uint32_t *reg = (uint32_t *)0x40000000;

while ((*reg & 1u) == 0) {
    // wait
}
```

compiler는 `*reg` 값이 루프 안에서 바뀌지 않는다고 판단할 수 있다.

embedded register라면 하드웨어가 값을 바꿀 수 있으므로 다음처럼 써야 한다.

```c
volatile uint32_t *reg = (volatile uint32_t *)0x40000000;

while ((*reg & 1u) == 0) {
    // wait until hardware sets bit 0
}
```

---

## 11. 확인 문제

아래 문제는 바로 풀 수 있어야 한다.

### 문제 1

다음 코드의 출력은?

```c
#include <stdio.h>
#include <stdint.h>

int main(void) {
    uint8_t x = 0b00000101;

    printf("%u\n", x & 1u);
    printf("%u\n", x & 4u);
    printf("%u\n", x & 2u);

    return 0;
}
```

---

### 문제 2

다음 중 더 안전한 bit mask 코드는?

```c
int a = 1 << 31;
uint32_t b = 1u << 31;
uint32_t c = UINT32_C(1) << 31;
```

각각 왜 그런지 설명해라.

---

### 문제 3

다음 코드의 문제점은?

```c
uint32_t mask = 1u << bit;
```

단, `bit`는 외부 입력으로 들어온다.

---

### 문제 4

`uint8_t x = 255;`에서 `x + 1`의 결과는 어떻게 되는가?

단, C의 integer promotion까지 고려해라.

힌트:

```c
uint8_t x = 255;
printf("%d\n", x + 1);
```

이 문제는 생각보다 중요하다.

---

## 12. 실습 과제

## 실습 1: bit 출력 함수 만들기

`uint8_t` 값을 binary로 출력하는 함수를 작성해라.

예:

```txt
input : 5
output: 00000101
```

시작 코드:

```c
#include <stdint.h>
#include <stdio.h>

void print_bits8(uint8_t x) {
    for (int i = 7; i >= 0; --i) {
        putchar(((x >> i) & 1u) ? '1' : '0');
    }
}

int main(void) {
    print_bits8(5);
    putchar('\n');
    return 0;
}
```

---

## 실습 2: flag system 만들기

다음 flag를 정의해라.

```txt
bit 0: READY
bit 1: ERROR
bit 2: BUSY
bit 3: DATA_AVAILABLE
```

그리고 다음 함수를 구현해라.

```c
void set_flag(uint8_t *status, uint8_t mask);
void clear_flag(uint8_t *status, uint8_t mask);
void toggle_flag(uint8_t *status, uint8_t mask);
int  is_flag_set(uint8_t status, uint8_t mask);
```

주의:

```c
uint8_t *status
```

가 `NULL`이면 어떻게 처리할지도 생각해라.

---

## 실습 3: embedded-style register simulation

가상의 register를 하나 만들고:

```c
volatile uint32_t REG_STATUS;
```

다음 bit를 정의해라.

```txt
bit 0: READY
bit 1: ERROR
bit 2: RX_DONE
bit 3: TX_DONE
```

그리고 다음 동작을 코드로 표현해라.

1. READY bit 켜기
2. ERROR bit 끄기
3. RX_DONE bit 확인하기
4. TX_DONE bit toggle하기

---

## 13. 핵심 정리

Lecture 1에서 가져가야 할 핵심은 이것이다.

1. **정수는 숫자이면서 동시에 bit pattern이다.**
2. **C type은 bit pattern을 어떻게 해석할지 결정한다.**
3. **Unsigned integer는 modulo arithmetic이 정의되어 있다.**
4. **Signed integer overflow는 undefined behavior다.**
5. **Bit manipulation은 embedded register, flag, bitmap, protocol, OS, graphics에서 필수다.**
6. **Bit trick을 성능 때문에 무조건 쓰면 안 된다.**
7. **현대 compiler는 산술 최적화를 잘하므로 readability와 portability를 우선해야 한다.**
8. **하드웨어 register를 다룰 때는 `volatile`, mask, unsigned type이 중요하다.**

---

## 14. 면접 대비용 핵심 문장

면접에서는 이렇게 설명하면 좋다.

> Bit manipulation은 정수를 단순한 numeric value가 아니라 fixed-width bit pattern으로 보고, 특정 bit를 set, clear, toggle, check하는 기술입니다. Embedded systems에서는 hardware register의 각 bit가 장치 상태나 제어 신호를 의미하기 때문에 필수적입니다.

또 이렇게 말할 수 있어야 한다.

> In C, unsigned integer arithmetic is defined modulo 2^N, but signed integer overflow is undefined behavior. Therefore, low-level bit manipulation should usually be done with unsigned fixed-width types such as `uint32_t`.

한국어로는:

> C에서 비트 연산을 안전하게 하려면 `int`보다 `uint32_t`, `uint8_t` 같은 unsigned fixed-width type을 쓰는 것이 좋습니다. signed overflow, signed shift, shift count overflow는 portability 문제를 만들 수 있기 때문입니다.

---

[1]: https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2817.pdf?utm_source=chatgpt.com "Can Signed Integers Overflow?"
