# Lecture 5. Bit Masking

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **어떻게 정수 하나 안에서 특정 bit만 켜고, 끄고, 뒤집고, 검사할 수 있는가?**

Bit masking은 embedded C에서 가장 많이 쓰는 bit manipulation 패턴이다.

대표 연산은 네 가지다.

```c
x |= mask;     // set bit
x &= ~mask;    // clear bit
x ^= mask;     // toggle bit
x & mask       // check bit
```

예를 들어 8-bit 값이 있다고 하자.

```txt
x = 00000000
```

3번 bit만 켜고 싶으면:

```txt
mask = 00001000
```

그리고 OR 연산을 한다.

```txt
  00000000
| 00001000
----------
  00001000
```

---

# 2. 형식적 정의

## 2.1 Mask란 무엇인가?

**Mask**는 특정 bit 위치를 선택하기 위해 사용하는 bit pattern이다.

예:

```txt
mask = 00001000
```

이 mask는 bit 3을 선택한다.

```txt
bit index: 7 6 5 4 3 2 1 0
mask:      0 0 0 0 1 0 0 0
```

C에서는 보통 이렇게 만든다.

```c
uint32_t mask = UINT32_C(1) << n;
```

예:

```c
uint32_t mask = UINT32_C(1) << 3;
```

결과:

```txt
00000000 00000000 00000000 00001000
```

---

## 2.2 Set bit

특정 bit를 1로 만든다.

```c
x = x | mask;
```

또는 축약해서:

```c
x |= mask;
```

성질:

```txt
0 | 1 = 1
1 | 1 = 1
```

즉 mask가 1인 위치는 무조건 1이 된다.

---

## 2.3 Clear bit

특정 bit를 0으로 만든다.

```c
x = x & ~mask;
```

또는:

```c
x &= ~mask;
```

성질:

```txt
0 & 0 = 0
1 & 0 = 0
```

`~mask`는 mask의 반전이다.

예:

```txt
mask  = 00001000
~mask = 11110111
```

AND를 하면 mask 위치만 0이 된다.

---

## 2.4 Toggle bit

특정 bit를 반전한다.

```c
x = x ^ mask;
```

또는:

```c
x ^= mask;
```

성질:

```txt
0 ^ 1 = 1
1 ^ 1 = 0
```

즉 mask가 1인 위치만 뒤집힌다.

---

## 2.5 Check bit

특정 bit가 1인지 확인한다.

```c
if ((x & mask) != 0u) {
    // bit is set
}
```

또는 bit 값을 0/1로 얻고 싶다면:

```c
bit = (x >> n) & 1u;
```

---

# 3. 직관적 설명

Bit mask는 “투명 필름”처럼 생각하면 된다.

값이 있다.

```txt
x = 10110110
```

우리가 bit 2만 보고 싶다고 하자.

```txt
mask = 00000100
```

AND를 한다.

```txt
  10110110
& 00000100
----------
  00000100
```

결과가 0이 아니므로 bit 2는 켜져 있다.

다른 bit들은 mask 때문에 전부 가려진다.

```txt
mask에서 1인 곳만 본다.
mask에서 0인 곳은 무시한다.
```

---

# 4. 왜 필요한지

Bit masking은 다음 상황에서 필수다.

| 상황                        | 예시                       |
| ------------------------- | ------------------------ |
| Embedded register control | UART enable bit 설정       |
| GPIO 제어                   | 특정 pin만 HIGH/LOW         |
| Status flag 관리            | READY, ERROR, BUSY       |
| Protocol parsing          | header field 추출          |
| OS permission             | read/write/execute flag  |
| Graphics                  | RGB channel 추출           |
| Compression               | bit 단위 데이터 저장            |
| Bitmap                    | 여러 boolean 값을 bit 단위로 저장 |

Embedded에서는 하드웨어 register의 각 bit가 의미를 가진다.

예를 들어 어떤 register가 있다고 하자.

```txt
CTRL_REG

bit 0: ENABLE
bit 1: RESET
bit 2: INTERRUPT_ENABLE
bit 3: DMA_ENABLE
```

이때 device를 enable하려면 전체 register를 바꾸는 것이 아니라 **bit 0만** 바꿔야 한다.

```c
CTRL_REG |= ENABLE_MASK;
```

다른 bit는 건드리면 안 된다.

---

# 5. 내부 원리

## 5.1 OR로 set하는 원리

```txt
x    = 10110000
mask = 00000100
```

OR:

```txt
  10110000
| 00000100
----------
  10110100
```

mask가 1인 bit 2만 1이 되었다.

다른 bit는 유지된다.

| 원래 bit | mask bit | 결과 |
| -----: | -------: | -: |
|      0 |        0 |  0 |
|      1 |        0 |  1 |
|      0 |        1 |  1 |
|      1 |        1 |  1 |

핵심:

```txt
x | 0 = x
x | 1 = 1
```

---

## 5.2 AND + NOT으로 clear하는 원리

```txt
x    = 10110100
mask = 00000100
```

먼저 mask를 반전한다.

```txt
~mask = 11111011
```

AND:

```txt
  10110100
& 11111011
----------
  10110000
```

bit 2만 0이 되었다.

핵심:

```txt
x & 1 = x
x & 0 = 0
```

---

## 5.3 XOR로 toggle하는 원리

```txt
x    = 10110000
mask = 00000100
```

XOR:

```txt
  10110000
^ 00000100
----------
  10110100
```

한 번 더 toggle하면:

```txt
  10110100
^ 00000100
----------
  10110000
```

핵심:

```txt
x ^ 0 = x
x ^ 1 = flipped x
```

---

## 5.4 AND로 check하는 원리

```txt
x    = 10110100
mask = 00000100
```

AND:

```txt
  10110100
& 00000100
----------
  00000100
```

결과가 0이 아니므로 bit 2는 set되어 있다.

만약 bit 1을 검사하면:

```txt
x    = 10110100
mask = 00000010
```

```txt
  10110100
& 00000010
----------
  00000000
```

결과가 0이므로 bit 1은 clear 상태다.

---

# 6. 단계별 예시

## 예시 1: 기본 set / clear / toggle / check

```c
#include <stdint.h>
#include <stdbool.h>

#define BIT3 (UINT32_C(1) << 3)

bool is_bit3_set(uint32_t x) {
    return (x & BIT3) != 0u;
}

int main(void) {
    uint32_t x = 0;

    x |= BIT3;       // set bit 3
    x &= ~BIT3;      // clear bit 3
    x ^= BIT3;       // toggle bit 3

    if (is_bit3_set(x)) {
        // bit 3 is set
    }

    return 0;
}
```

흐름:

```txt
초기:
00000000

set bit 3:
00001000

clear bit 3:
00000000

toggle bit 3:
00001000
```

---

## 예시 2: 여러 flag 관리

```c
#include <stdint.h>
#include <stdbool.h>

#define FLAG_READY  (UINT32_C(1) << 0)
#define FLAG_ERROR  (UINT32_C(1) << 1)
#define FLAG_BUSY   (UINT32_C(1) << 2)
#define FLAG_SLEEP  (UINT32_C(1) << 3)

void set_flags(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags |= mask;
}

void clear_flags(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags &= ~mask;
}

void toggle_flags(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags ^= mask;
}

bool are_flags_set(uint32_t flags, uint32_t mask) {
    return (flags & mask) == mask;
}

bool is_any_flag_set(uint32_t flags, uint32_t mask) {
    return (flags & mask) != 0u;
}
```

차이점이 중요하다.

```c
(flags & mask) == mask
```

은 mask에 해당하는 bit들이 **전부 set**인지 확인한다.

```c
(flags & mask) != 0u
```

은 mask에 해당하는 bit 중 **하나라도 set**인지 확인한다.

예:

```txt
flags = 00000101
mask  = 00000111
```

```txt
flags & mask = 00000101
```

따라서:

```txt
(flags & mask) == mask  → false
(flags & mask) != 0     → true
```

---

## 예시 3: lower byte extraction

32-bit 값에서 하위 8-bit만 꺼내고 싶다고 하자.

```c
#include <stdint.h>

uint8_t lower_byte(uint32_t value) {
    return (uint8_t)(value & 0xFFu);
}
```

예:

```txt
value = 0x12345678
mask  = 0x000000FF
-----------------
&     = 0x00000078
```

결과:

```txt
lower byte = 0x78
```

상위 byte를 꺼내고 싶다면 shift를 함께 쓴다.

```c
uint8_t byte2(uint32_t value) {
    return (uint8_t)((value >> 16) & 0xFFu);
}
```

예:

```txt
value = 0x12345678

value >> 16 = 0x00001234
& 0xFF      = 0x00000034
```

결과:

```txt
0x34
```

---

## 예시 4: bit field 삽입

어떤 32-bit register에서 bit 8..15에 `value`를 넣고 싶다고 하자.

```txt
bits 8..15: FIELD
```

코드:

```c
#include <stdint.h>

#define FIELD_SHIFT 8u
#define FIELD_WIDTH 8u
#define FIELD_MASK  (UINT32_C(0xFF) << FIELD_SHIFT)

uint32_t set_field(uint32_t reg, uint32_t value) {
    value &= 0xFFu;              // 8-bit로 제한
    reg &= ~FIELD_MASK;          // 기존 field 제거
    reg |= value << FIELD_SHIFT; // 새 field 삽입
    return reg;
}
```

예:

```txt
reg   = 0x12345678
value = 0xAB
```

기존 field 제거:

```txt
reg         = 0x12345678
~FIELD_MASK = 0xFFFF00FF
------------------------
&           = 0x12340078
```

새 field 삽입:

```txt
value << 8 = 0x0000AB00
```

OR:

```txt
0x12340078
0x0000AB00
----------
0x1234AB78
```

---

# 7. 실제 응용

## 7.1 Embedded GPIO 제어

가상의 GPIO output register가 있다고 하자.

```c
#include <stdint.h>

volatile uint32_t GPIO_OUT;

#define GPIO_PIN5 (UINT32_C(1) << 5)

void gpio5_high(void) {
    GPIO_OUT |= GPIO_PIN5;
}

void gpio5_low(void) {
    GPIO_OUT &= ~GPIO_PIN5;
}

void gpio5_toggle(void) {
    GPIO_OUT ^= GPIO_PIN5;
}

int gpio5_is_high(void) {
    return (GPIO_OUT & GPIO_PIN5) != 0u;
}
```

의미:

| 함수              | 동작           |
| --------------- | ------------ |
| `gpio5_high`    | 5번 pin HIGH  |
| `gpio5_low`     | 5번 pin LOW   |
| `gpio5_toggle`  | 5번 pin 반전    |
| `gpio5_is_high` | 5번 pin 상태 확인 |

여기서 `volatile`은 중요하다.

```c
volatile uint32_t GPIO_OUT;
```

하드웨어 register는 프로그램 외부의 하드웨어가 값을 바꿀 수 있다.
따라서 compiler가 읽기/쓰기를 마음대로 제거하거나 재배치하면 안 된다.

---

## 7.2 UART control register

예:

```txt
UART_CTRL

bit 0: UART_ENABLE
bit 1: TX_ENABLE
bit 2: RX_ENABLE
bit 3: PARITY_ENABLE
```

C 코드:

```c
#include <stdint.h>

volatile uint32_t UART_CTRL;

#define UART_ENABLE        (UINT32_C(1) << 0)
#define UART_TX_ENABLE     (UINT32_C(1) << 1)
#define UART_RX_ENABLE     (UINT32_C(1) << 2)
#define UART_PARITY_ENABLE (UINT32_C(1) << 3)

void uart_enable(void) {
    UART_CTRL |= UART_ENABLE;
}

void uart_disable_parity(void) {
    UART_CTRL &= ~UART_PARITY_ENABLE;
}

void uart_enable_tx_rx(void) {
    UART_CTRL |= UART_TX_ENABLE | UART_RX_ENABLE;
}
```

여러 bit를 동시에 set할 수 있다.

```c
UART_CTRL |= UART_TX_ENABLE | UART_RX_ENABLE;
```

mask:

```txt
UART_TX_ENABLE = 00000010
UART_RX_ENABLE = 00000100
OR result      = 00000110
```

---

## 7.3 Status register 확인

```txt
STATUS_REG

bit 0: READY
bit 1: ERROR
bit 2: RX_DONE
bit 3: TX_DONE
```

코드:

```c
#include <stdint.h>
#include <stdbool.h>

volatile uint32_t STATUS_REG;

#define STATUS_READY   (UINT32_C(1) << 0)
#define STATUS_ERROR   (UINT32_C(1) << 1)
#define STATUS_RX_DONE (UINT32_C(1) << 2)
#define STATUS_TX_DONE (UINT32_C(1) << 3)

bool is_ready(void) {
    return (STATUS_REG & STATUS_READY) != 0u;
}

bool has_error(void) {
    return (STATUS_REG & STATUS_ERROR) != 0u;
}
```

Embedded에서 이런 코드는 매우 흔하다.

```c
while ((STATUS_REG & STATUS_READY) == 0u) {
    // wait
}
```

주의:

이런 busy-wait loop는 간단하지만 CPU를 계속 점유한다.
실제 시스템에서는 interrupt, timeout, sleep mode 등을 고려해야 한다.

---

# 8. 성능과 메모리 관점

## 8.1 Masking은 일반적으로 빠르다

다음 연산들은 CPU instruction 하나 또는 매우 적은 instruction으로 처리되는 경우가 많다.

```c
x & mask
x | mask
x ^ mask
x << n
x >> n
```

하지만 중요한 점은 이것이다.

> 빠르다는 이유만으로 가독성을 희생하면 안 된다.

좋지 않은 코드:

```c
reg = (reg & ~(0x7u << 13)) | ((v & 0x7u) << 13);
```

동작은 맞지만 의미가 잘 안 보인다.

더 좋은 코드:

```c
#define MODE_SHIFT 13u
#define MODE_MASK  (UINT32_C(0x7) << MODE_SHIFT)

reg = (reg & ~MODE_MASK) | ((v & 0x7u) << MODE_SHIFT);
```

더 좋게는 함수로 감싼다.

```c
static inline uint32_t set_mode_field(uint32_t reg, uint32_t mode) {
    mode &= 0x7u;
    return (reg & ~MODE_MASK) | (mode << MODE_SHIFT);
}
```

---

## 8.2 Memory saving

여러 boolean flag를 따로 저장하면 메모리를 더 많이 쓴다.

```c
bool ready;
bool error;
bool busy;
bool sleep;
```

대부분의 C 구현에서 `bool`은 최소 1 byte다.

4개의 flag면 대략 4 byte다.

하지만 bit mask를 쓰면 1개의 `uint8_t`에도 담을 수 있다.

```c
uint8_t flags;
```

```txt
bit 0: READY
bit 1: ERROR
bit 2: BUSY
bit 3: SLEEP
```

메모리 관점:

| 방식               |               대략적 크기 |
| ---------------- | -------------------: |
| `bool flags[8]`  |              8 bytes |
| `uint8_t flags`  |               1 byte |
| `uint32_t flags` | 4 bytes, 32 flags 가능 |

Embedded에서 RAM이 작은 경우 이 차이는 크다.

---

## 8.3 Cache locality

많은 flag를 compact하게 저장하면 cache locality가 좋아질 수 있다.

```txt
bool array:
[1 byte][1 byte][1 byte][1 byte] ...

bitmap/flags:
[bit][bit][bit][bit][bit][bit][bit][bit]
```

하지만 trade-off가 있다.

| 장점                          | 단점            |
| --------------------------- | ------------- |
| 메모리 절약                      | 코드 복잡도 증가     |
| cache locality 개선 가능        | bit 추출 연산 필요  |
| register/protocol 표현에 자연스러움 | 디버깅이 어려울 수 있음 |

일반 application code에서 flag가 몇 개 안 된다면 `bool` 여러 개가 더 읽기 쉬울 수도 있다.

Embedded register, protocol, OS, bitmap에서는 bit mask가 자연스럽다.

---

# 9. 흔한 오해

## 오해 1: bit 번호는 왼쪽부터 센다

보통 bit index는 오른쪽, 즉 LSB부터 센다.

```txt
bit index: 7 6 5 4 3 2 1 0
value:     1 0 1 1 0 1 0 0
```

따라서:

```c
1u << 0
```

은 가장 오른쪽 bit다.

```txt
00000001
```

```c
1u << 7
```

은 8-bit 기준 가장 왼쪽 bit다.

```txt
10000000
```

---

## 오해 2: `~mask`는 항상 원하는 width 안에서만 반전된다

C에서 `~mask`는 integer promotion과 type width에 영향을 받는다.

예:

```c
uint8_t mask = 0x0F;
uint8_t x = 0xFF;

x &= ~mask;
```

겉으로는 괜찮아 보이지만, `mask`는 연산 전에 `int`로 promotion될 수 있다.

개념적으로:

```c
x = (uint8_t)((int)x & ~(int)mask);
```

결과는 보통 기대대로 나오지만, 작은 정수 type에서는 promotion을 이해해야 한다.

더 명확한 스타일:

```c
x = (uint8_t)(x & (uint8_t)~mask);
```

또는 working type을 `uint32_t`로 둔다.

```c
uint32_t flags = 0xFFu;
flags &= ~0x0Fu;
```

Embedded register는 보통 32-bit이므로 `uint32_t` mask를 쓰는 편이 명확하다.

---

## 오해 3: `1 << n`은 항상 안전하다

아니다.

```c
int mask = 1 << 31;
```

위험하다.

`1`은 signed `int`다.
32-bit int에서 sign bit로 shift하면 undefined behavior가 될 수 있다.

권장:

```c
uint32_t mask = UINT32_C(1) << 31;
```

또한 `n >= 32`이면 이것도 undefined behavior다.

```c
uint32_t mask = UINT32_C(1) << n;  // n >= 32면 위험
```

외부 입력이면 검사해야 한다.

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask_u32(unsigned n, uint32_t *out) {
    if (out == NULL || n >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << n;
    return true;
}
```

---

## 오해 4: check bit는 `(x & mask) == 1`로 하면 된다

틀릴 수 있다.

예:

```c
#define BIT3 (1u << 3)

if ((x & BIT3) == 1u) {
    // wrong
}
```

bit 3이 set되어 있다면 결과는 `1`이 아니라 `8`이다.

```txt
x & 00001000 = 00001000 = 8
```

따라서 check는 이렇게 해야 한다.

```c
if ((x & BIT3) != 0u) {
    // correct
}
```

또는 정확히 mask와 같은지 본다.

```c
if ((x & BIT3) == BIT3) {
    // also correct
}
```

---

# 10. 반례 또는 실패 사례

## 실패 사례 1: 다른 bit를 지워버리는 register write

잘못된 코드:

```c
CTRL_REG = UART_ENABLE;
```

이 코드는 UART_ENABLE bit만 켜는 것이 아니라, register 전체를 그 값으로 덮어쓴다.

즉 기존에 켜져 있던 다른 bit가 사라질 수 있다.

예:

```txt
CTRL_REG before = 00001110
UART_ENABLE     = 00000001

CTRL_REG after  = 00000001
```

bit 1, 2, 3이 전부 꺼져버렸다.

원하는 것이 bit 0만 켜기라면:

```c
CTRL_REG |= UART_ENABLE;
```

결과:

```txt
CTRL_REG before = 00001110
UART_ENABLE     = 00000001
OR result       = 00001111
```

---

## 실패 사례 2: clear하려고 OR 사용

잘못된 코드:

```c
flags |= ~FLAG_ERROR;
```

이건 ERROR bit를 clear하지 않는다.
오히려 거의 모든 bit를 1로 만들어버릴 수 있다.

clear는 반드시 AND + NOT이다.

```c
flags &= ~FLAG_ERROR;
```

비교:

```txt
flags      = 00001111
FLAG_ERROR = 00000010
~FLAG_ERR  = 11111101

flags | ~FLAG_ERROR
= 11111111   // 대형 사고

flags & ~FLAG_ERROR
= 00001101   // 의도한 결과
```

---

## 실패 사례 3: multi-bit field를 OR만으로 덮어쓰기

잘못된 코드:

```c
reg |= mode << MODE_SHIFT;
```

이 코드는 기존 mode field를 지우지 않고 OR만 한다.

예:

```txt
기존 mode = 111
새 mode   = 001
```

OR만 하면:

```txt
111 | 001 = 111
```

값이 바뀌지 않는다.

정확한 순서:

```c
reg &= ~MODE_MASK;                 // 기존 field 제거
reg |= (mode << MODE_SHIFT);       // 새 field 삽입
```

또는 한 줄로:

```c
reg = (reg & ~MODE_MASK) | ((mode & 0x7u) << MODE_SHIFT);
```

---

## 실패 사례 4: mask width 불일치

```c
uint64_t x = UINT64_C(1) << 40;
uint64_t mask = 1u << 40;  // wrong
```

`1u`는 보통 32-bit unsigned int다.
`1u << 40`은 shift count가 width 이상이므로 undefined behavior다.

올바른 코드:

```c
uint64_t mask = UINT64_C(1) << 40;
```

---

# 11. Portability-safe coding style

## 11.1 bit macro

```c
#include <stdint.h>

#define BIT_U32(n) (UINT32_C(1) << (n))
#define BIT_U64(n) (UINT64_C(1) << (n))
```

주의:

```txt
BIT_U32(n): n < 32일 때만 안전
BIT_U64(n): n < 64일 때만 안전
```

외부 입력은 함수로 처리한다.

```c
#include <stdint.h>
#include <stdbool.h>

bool bit_u32(unsigned n, uint32_t *out) {
    if (out == NULL || n >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << n;
    return true;
}
```

---

## 11.2 flag 조작 함수

```c
#include <stdint.h>
#include <stdbool.h>

static inline void flags_set(uint32_t *flags, uint32_t mask) {
    if (flags != NULL) {
        *flags |= mask;
    }
}

static inline void flags_clear(uint32_t *flags, uint32_t mask) {
    if (flags != NULL) {
        *flags &= ~mask;
    }
}

static inline void flags_toggle(uint32_t *flags, uint32_t mask) {
    if (flags != NULL) {
        *flags ^= mask;
    }
}

static inline bool flags_any(uint32_t flags, uint32_t mask) {
    return (flags & mask) != 0u;
}

static inline bool flags_all(uint32_t flags, uint32_t mask) {
    return (flags & mask) == mask;
}
```

---

## 11.3 register field helper

```c
#include <stdint.h>

#define FIELD_PREP(mask, shift, value) \
    (((uint32_t)(value) << (shift)) & (mask))

#define FIELD_GET(mask, shift, reg) \
    (((uint32_t)(reg) & (mask)) >> (shift))
```

사용 예:

```c
#define MODE_SHIFT 4u
#define MODE_MASK  (UINT32_C(0x7) << MODE_SHIFT)

uint32_t mode = FIELD_GET(MODE_MASK, MODE_SHIFT, reg);
reg = (reg & ~MODE_MASK) | FIELD_PREP(MODE_MASK, MODE_SHIFT, mode);
```

이런 helper는 Linux kernel 스타일의 `FIELD_PREP`, `FIELD_GET`와 비슷한 사고방식이다.

---

# 12. 확인 문제

## 문제 1

다음 결과를 계산해라.

```txt
x    = 10110000
mask = 00000100

x | mask  = ?
x & ~mask = ?
x ^ mask  = ?
x & mask  = ?
```

---

## 문제 2

다음 코드는 어떤 일을 하는가?

```c
flags |= FLAG_READY;
```

---

## 문제 3

다음 코드는 어떤 일을 하는가?

```c
flags &= ~FLAG_ERROR;
```

---

## 문제 4

다음 코드는 왜 틀릴 수 있는가?

```c
if ((flags & FLAG_BUSY) == 1u) {
    // busy
}
```

---

## 문제 5

다음 코드의 문제는 무엇인가?

```c
reg |= mode << MODE_SHIFT;
```

단, `mode`는 multi-bit field이고 기존 field 값이 이미 있을 수 있다.

---

## 문제 6

다음 코드는 왜 위험한가?

```c
uint64_t mask = 1u << 40;
```

---

# 13. 실습 과제

## 실습 1: flag 조작 함수 구현

다음 flag를 정의해라.

```c
#define FLAG_READY  (UINT32_C(1) << 0)
#define FLAG_ERROR  (UINT32_C(1) << 1)
#define FLAG_BUSY   (UINT32_C(1) << 2)
#define FLAG_SLEEP  (UINT32_C(1) << 3)
```

그리고 다음 함수를 구현해라.

```c
void set_flag(uint32_t *flags, uint32_t mask);
void clear_flag(uint32_t *flags, uint32_t mask);
void toggle_flag(uint32_t *flags, uint32_t mask);
bool is_flag_set(uint32_t flags, uint32_t mask);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>

void set_flag(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags |= mask;
}

void clear_flag(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags &= ~mask;
}

void toggle_flag(uint32_t *flags, uint32_t mask) {
    if (flags == NULL) {
        return;
    }

    *flags ^= mask;
}

bool is_flag_set(uint32_t flags, uint32_t mask) {
    return (flags & mask) != 0u;
}
```

---

## 실습 2: lower byte / upper byte 추출

다음 함수를 구현해라.

```c
#include <stdint.h>

uint8_t get_byte0(uint32_t value);
uint8_t get_byte1(uint32_t value);
uint8_t get_byte2(uint32_t value);
uint8_t get_byte3(uint32_t value);
```

예상 구현:

```c
#include <stdint.h>

uint8_t get_byte0(uint32_t value) {
    return (uint8_t)(value & 0xFFu);
}

uint8_t get_byte1(uint32_t value) {
    return (uint8_t)((value >> 8) & 0xFFu);
}

uint8_t get_byte2(uint32_t value) {
    return (uint8_t)((value >> 16) & 0xFFu);
}

uint8_t get_byte3(uint32_t value) {
    return (uint8_t)((value >> 24) & 0xFFu);
}
```

테스트:

```txt
value = 0x12345678

byte0 = 0x78
byte1 = 0x56
byte2 = 0x34
byte3 = 0x12
```

주의:

이건 **정수 값에서 byte 값을 추출**하는 것이다.
메모리에 어떤 순서로 저장되는지는 endianness 문제이고 Lecture 8에서 다룬다.

---

## 실습 3: register field 설정

다음 register layout이 있다고 하자.

```txt
bits 0..3   : STATUS
bits 4..6   : MODE
bit  7      : ENABLE
bits 8..15  : VALUE
```

다음 함수를 작성해라.

```c
uint32_t set_mode(uint32_t reg, uint32_t mode);
uint32_t enable_device(uint32_t reg);
uint32_t disable_device(uint32_t reg);
uint32_t set_value(uint32_t reg, uint32_t value);
```

예상 구현:

```c
#include <stdint.h>

#define MODE_SHIFT   4u
#define MODE_MASK    (UINT32_C(0x7) << MODE_SHIFT)

#define ENABLE_MASK  (UINT32_C(1) << 7)

#define VALUE_SHIFT  8u
#define VALUE_MASK   (UINT32_C(0xFF) << VALUE_SHIFT)

uint32_t set_mode(uint32_t reg, uint32_t mode) {
    mode &= 0x7u;
    reg &= ~MODE_MASK;
    reg |= mode << MODE_SHIFT;
    return reg;
}

uint32_t enable_device(uint32_t reg) {
    return reg | ENABLE_MASK;
}

uint32_t disable_device(uint32_t reg) {
    return reg & ~ENABLE_MASK;
}

uint32_t set_value(uint32_t reg, uint32_t value) {
    value &= 0xFFu;
    reg &= ~VALUE_MASK;
    reg |= value << VALUE_SHIFT;
    return reg;
}
```

---

## 실습 4: 안전한 mask 생성

다음 함수를 구현해라.

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask_u32(unsigned bit, uint32_t *out);
```

조건:

```txt
bit >= 32이면 false
out == NULL이면 false
성공하면 *out에 mask 저장 후 true
```

구현:

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask_u32(unsigned bit, uint32_t *out) {
    if (out == NULL || bit >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << bit;
    return true;
}
```

---

# 14. 핵심 정리

이번 강의 핵심은 다음이다.

1. **Mask는 특정 bit 위치를 선택하기 위한 bit pattern이다.**
2. **`x | mask`는 mask에 해당하는 bit를 set한다.**
3. **`x & ~mask`는 mask에 해당하는 bit를 clear한다.**
4. **`x ^ mask`는 mask에 해당하는 bit를 toggle한다.**
5. **`x & mask`는 mask에 해당하는 bit가 set되어 있는지 확인한다.**
6. **`(x & mask) != 0`은 하나라도 set인지 확인한다.**
7. **`(x & mask) == mask`는 mask의 모든 bit가 set인지 확인한다.**
8. **multi-bit field를 설정할 때는 기존 field를 먼저 clear한 뒤 새 값을 OR해야 한다.**
9. **`1 << n`보다 `UINT32_C(1) << n` 같은 unsigned fixed-width 표현이 안전하다.**
10. **외부 입력으로 shift count가 들어오면 반드시 width를 검사해야 한다.**
11. **embedded register에서는 다른 bit를 건드리지 않기 위해 masking이 필수다.**
12. **bit trick은 의미 있는 이름의 macro, inline function, helper로 감싸는 것이 좋다.**

---

# 15. 면접 대비 핵심 문장

면접에서는 이렇게 말하면 된다.

> A bit mask is a bit pattern used to select or modify specific bits. To set bits, we use OR; to clear bits, we use AND with the inverted mask; to toggle bits, we use XOR; and to check bits, we use AND.

한국어로는:

> Bit mask는 특정 bit를 선택하거나 수정하기 위한 bit pattern입니다. 특정 bit를 켤 때는 OR, 끌 때는 `AND`와 `~mask`, 반전할 때는 XOR, 확인할 때는 AND를 사용합니다.

Embedded 관점에서는 이렇게 설명하면 좋다.

> In embedded systems, hardware registers often assign a different meaning to each bit. Bit masking allows us to modify one control bit without changing the other bits in the same register.

한국어로는:

> Embedded system에서는 하드웨어 register의 각 bit가 enable, error, interrupt 같은 의미를 가집니다. Bit masking을 사용하면 같은 register 안의 다른 bit를 건드리지 않고 원하는 bit만 제어할 수 있습니다.

안전성까지 포함하면:

> For portable C code, masks should usually be built with unsigned fixed-width constants such as `UINT32_C(1) << n`, and the shift count must be checked when it comes from external input.

한국어로는:

> Portable한 C 코드를 위해서는 mask를 만들 때 `UINT32_C(1) << n`처럼 unsigned fixed-width 상수를 사용하는 것이 좋고, `n`이 외부 입력이라면 type width보다 작은지 반드시 검사해야 합니다.

---