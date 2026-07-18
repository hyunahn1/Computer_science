# Lecture 4. Odd/Even, Shift, and Arithmetic Optimization

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **`x << 1`은 정말 항상 `x * 2`와 같고, `x >> 1`은 정말 항상 `x / 2`와 같은가?**

답은:

> **unsigned 정수에서는 비교적 예측 가능하지만, signed 정수에서는 조심해야 한다.**

특히 C에서는 다음을 반드시 구분해야 한다.

```txt
unsigned shift  → 비교적 명확하고 portable
signed shift    → overflow, sign bit, implementation-defined 문제 존재
```

이번 강의에서는 다음을 다룬다.

```txt
x & 1
x << 1
x >> 1
x * 2^n
x / 2^n
x % 2^n
i & (2^n - 1)
```

그리고 핵심 판단 기준은 이것이다.

> **산술을 표현하려는가, bit pattern을 조작하려는가?**

---

## 2. 형식적 정의

## 2.1 Odd / Even 판별

정수 `x`가 홀수인지 짝수인지는 최하위 bit, 즉 LSB로 알 수 있다.

```txt
LSB = Least Significant Bit
```

이진수에서 가장 오른쪽 bit다.

```txt
00000101
       ^
       LSB
```

규칙:

```txt
LSB = 0 → even
LSB = 1 → odd
```

C 코드:

```c
if ((x & 1u) != 0u) {
    // odd
} else {
    // even
}
```

---

## 2.2 Left shift

```c
x << n
```

은 bit pattern을 왼쪽으로 `n`칸 이동시킨다.

예:

```txt
x       = 00000101  // 5
x << 1  = 00001010  // 10
x << 2  = 00010100  // 20
```

`unsigned` 정수에서 overflow 없이 해석하면 대략:

```txt
x << n ≈ x * 2^n
```

하지만 정확히는 `unsigned`에서 다음과 같이 동작한다.

```txt
(x << n) mod 2^width
```

단, `n`은 type width보다 작아야 한다.

---

## 2.3 Right shift

```c
x >> n
```

은 bit pattern을 오른쪽으로 `n`칸 이동시킨다.

`unsigned`에서는 왼쪽이 0으로 채워진다.

```txt
x       = 00001000  // 8
x >> 1  = 00000100  // 4
x >> 2  = 00000010  // 2
```

즉 `unsigned`에서는 대략:

```txt
x >> n = floor(x / 2^n)
```

---

## 2.4 Modulo power of two

`N = 2^k`일 때:

```c
x % N
```

은 다음과 동일하게 표현할 수 있다.

```c
x & (N - 1)
```

예:

```c
x % 8
```

은:

```c
x & 7
```

과 같다.

왜냐하면:

```txt
8     = 00001000
8 - 1 = 00000111
```

`& 7`은 하위 3 bit만 남긴다.

---

## 3. 직관적 설명

## 3.1 왜 `x & 1`로 홀짝을 알 수 있는가

10진수에서 어떤 수가 홀수인지 짝수인지는 마지막 자리를 보면 된다.

```txt
1234 → 마지막 자리 4 → 짝수
1235 → 마지막 자리 5 → 홀수
```

2진수에서도 비슷하다.

2진수에서 각 자리의 값은 다음과 같다.

```txt
... 16 8 4 2 1
```

가장 오른쪽 bit만 `1`의 자리다.

```txt
101101
     ^
     1의 자리
```

따라서 이 bit가 1이면 전체 숫자는 홀수다.

```txt
101100 = 44 → 짝수
101101 = 45 → 홀수
```

---

## 3.2 왜 left shift는 곱하기 2처럼 보이는가

10진수에서 왼쪽으로 한 자리 밀면 10배가 된다.

```txt
123 → 1230
```

2진수에서는 왼쪽으로 한 자리 밀면 2배가 된다.

```txt
101 = 5
1010 = 10
```

그래서:

```txt
x << 1 ≈ x * 2
x << 2 ≈ x * 4
x << 3 ≈ x * 8
```

단, 이건 bit가 밀려나지 않고, signed overflow가 없다는 조건에서의 직관이다.

---

## 3.3 왜 right shift는 나누기 2처럼 보이는가

2진수에서 오른쪽으로 한 자리 밀면 2로 나눈 몫처럼 된다.

```txt
1000 = 8
0100 = 4
0010 = 2
0001 = 1
```

또 다른 예:

```txt
1011 = 11
0101 = 5
```

`11 / 2 = 5.5`인데, 정수에서는 소수 부분이 버려져서 5다.

즉 unsigned 기준으로:

```txt
x >> 1 = floor(x / 2)
```

---

## 4. 왜 필요한지

Shift와 bit-based arithmetic은 다음 곳에서 많이 등장한다.

| 상황                         | 예시                            |
| -------------------------- | ----------------------------- |
| Embedded register field 구성 | 특정 bit 위치에 값을 넣기              |
| Protocol parsing           | header field 추출               |
| Ring buffer                | `% size` 대신 `& (size - 1)`    |
| Graphics                   | RGB channel packing           |
| Memory alignment           | 주소가 2의 거듭제곱 경계에 맞는지 확인        |
| Compression                | bit stream 처리                 |
| OS                         | page size, flags, permissions |
| DSP / signal processing    | fixed-point 연산                |
| Interview                  | 홀짝, shift, modulo 최적화 문제      |

예를 들어 RGB 값을 하나의 32-bit 값으로 packing할 때:

```c
uint32_t color =
    ((uint32_t)r << 16) |
    ((uint32_t)g << 8)  |
    ((uint32_t)b);
```

이 코드는 다음 구조를 만든다.

```txt
00000000 RRRRRRRR GGGGGGGG BBBBBBBB
```

---

## 5. 내부 원리

## 5.1 `x & 1`의 내부 원리

예를 들어 `x = 45`.

```txt
45 = 00101101
1  = 00000001
```

AND:

```txt
  00101101
& 00000001
----------
  00000001
```

결과가 1이므로 홀수다.

`x = 44`라면:

```txt
44 = 00101100
1  = 00000001
```

AND:

```txt
  00101100
& 00000001
----------
  00000000
```

결과가 0이므로 짝수다.

---

## 5.2 Left shift 내부 원리

예:

```txt
x = 00000101
```

`x << 1`:

```txt
00000101
→ 00001010
```

왼쪽으로 밀리고 오른쪽에는 0이 들어온다.

```txt
before: 00000101
after : 00001010
```

`x << 2`:

```txt
before: 00000101
after : 00010100
```

---

## 5.3 Right shift 내부 원리

`unsigned` 기준:

```txt
x = 10000000
```

`x >> 1`:

```txt
01000000
```

왼쪽에는 0이 들어온다.

```txt
before: 10000000
after : 01000000
```

이것을 **logical right shift**라고 부른다.

---

## 5.4 Signed right shift 문제

`signed int`가 음수일 때는 문제가 생긴다.

예를 들어 8-bit two’s complement로 `-8`을 표현하면:

```txt
-8 = 11111000
```

이 값을 오른쪽으로 1칸 shift하면 두 가지 가능성이 있다.

### 가능성 1: logical shift

왼쪽에 0을 채운다.

```txt
11111000 >> 1
= 01111100
```

이러면 양수가 되어버린다.

### 가능성 2: arithmetic shift

왼쪽에 sign bit인 1을 채운다.

```txt
11111000 >> 1
= 11111100
```

이 값은 `-4`다.

대부분의 현대 CPU와 compiler는 signed 음수 right shift에 대해 arithmetic shift처럼 동작한다.
하지만 C 관점에서는 이 동작이 portable하다고 가정하면 안 된다.

정리:

```txt
unsigned right shift → 왼쪽에 0 채움
signed negative right shift → implementation-defined 가능성
```

---

## 6. 단계별 예시

## 예시 1: 홀짝 판별

```c
#include <stdint.h>
#include <stdbool.h>

bool is_odd_u32(uint32_t x) {
    return (x & 1u) != 0u;
}

bool is_even_u32(uint32_t x) {
    return (x & 1u) == 0u;
}
```

테스트:

```c
#include <stdio.h>
#include <stdint.h>
#include <stdbool.h>

bool is_odd_u32(uint32_t x) {
    return (x & 1u) != 0u;
}

int main(void) {
    for (uint32_t i = 0; i < 8u; ++i) {
        printf("%u: %s\n", i, is_odd_u32(i) ? "odd" : "even");
    }

    return 0;
}
```

출력 개념:

```txt
0: even
1: odd
2: even
3: odd
4: even
5: odd
6: even
7: odd
```

---

## 예시 2: `x << 1`

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t x = 5;

    printf("%u\n", x << 1);  // 10
    printf("%u\n", x << 2);  // 20
    printf("%u\n", x << 3);  // 40

    return 0;
}
```

binary:

```txt
5      = 00000101
5 << 1 = 00001010 = 10
5 << 2 = 00010100 = 20
5 << 3 = 00101000 = 40
```

---

## 예시 3: `x >> 1`

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t x = 40;

    printf("%u\n", x >> 1);  // 20
    printf("%u\n", x >> 2);  // 10
    printf("%u\n", x >> 3);  // 5

    return 0;
}
```

binary:

```txt
40      = 00101000
40 >> 1 = 00010100 = 20
40 >> 2 = 00001010 = 10
40 >> 3 = 00000101 = 5
```

---

## 예시 4: modulo power of two

`x % 8`은 `x & 7`과 같다.

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    for (uint32_t x = 0; x < 20u; ++x) {
        printf("%2u: x %% 8 = %u, x & 7 = %u\n",
               x, x % 8u, x & 7u);
    }

    return 0;
}
```

일부 결과:

```txt
0: 0, 0
1: 1, 1
2: 2, 2
3: 3, 3
4: 4, 4
5: 5, 5
6: 6, 6
7: 7, 7
8: 0, 0
9: 1, 1
```

왜냐하면 `& 7`은 하위 3 bit만 남기기 때문이다.

```txt
7 = 00000111
```

예:

```txt
13 = 00001101
7  = 00000111
--------------
&  = 00000101 = 5

13 % 8 = 5
```

---

## 7. 실제 응용

## 7.1 Embedded register field 설정

어떤 register의 bit 4~6에 mode 값을 넣어야 한다고 하자.

```txt
bit 6 5 4: MODE
```

예를 들어 `mode = 5`.

```txt
mode = 101
```

이를 bit 4 위치로 옮기려면:

```c
mode << 4
```

코드:

```c
#include <stdint.h>

#define MODE_SHIFT 4u
#define MODE_MASK  (UINT32_C(0x7) << MODE_SHIFT)

volatile uint32_t CTRL_REG;

void set_mode(uint32_t mode) {
    mode &= 0x7u;  // 3-bit 값만 허용

    CTRL_REG = (CTRL_REG & ~MODE_MASK) | (mode << MODE_SHIFT);
}
```

해석:

```txt
1. MODE_MASK로 기존 mode field 위치 정의
2. CTRL_REG & ~MODE_MASK로 기존 mode field 제거
3. mode << MODE_SHIFT로 새 mode를 field 위치에 배치
4. OR로 합침
```

이 패턴은 embedded register 제어에서 매우 중요하다.

---

## 7.2 RGB color packing

각 색상 channel이 8-bit라고 하자.

```txt
r = 0x12
g = 0x34
b = 0x56
```

하나의 32-bit 값으로 합치기:

```c
#include <stdint.h>

uint32_t pack_rgb(uint8_t r, uint8_t g, uint8_t b) {
    return ((uint32_t)r << 16) |
           ((uint32_t)g << 8)  |
           ((uint32_t)b);
}
```

결과:

```txt
0x00123456
```

주의할 점은 cast다.

```c
(uint32_t)r << 16
```

이렇게 먼저 넓은 type으로 변환한 뒤 shift하는 것이 안전하다.

---

## 7.3 Packet header parsing

네트워크나 파일 포맷에서는 한 byte 안에 여러 field가 들어갈 수 있다.

예:

```txt
header byte = 10110110

bits 7..5: version
bits 4..0: type
```

추출:

```c
#include <stdint.h>

uint8_t get_version(uint8_t header) {
    return (uint8_t)((header >> 5) & 0x07u);
}

uint8_t get_type(uint8_t header) {
    return (uint8_t)(header & 0x1Fu);
}
```

`header = 10110110`이라면:

```txt
version = 101 = 5
type    = 10110 = 22
```

---

## 7.4 Ring buffer index

크기가 256인 ring buffer:

```c
#define BUF_SIZE 256u
```

다음 index 계산:

```c
idx = (idx + 1u) & (BUF_SIZE - 1u);
```

`BUF_SIZE - 1`:

```txt
255 = 0xFF = 11111111
```

이 방식은 `idx + 1`의 하위 8 bit만 남긴다.

```txt
255 + 1 = 256 = 1 00000000
& 255          0 11111111
-------------------------
0
```

따라서 index가 자동으로 0으로 돌아간다.

단, 다시 강조한다.

> `& (size - 1)` 방식은 size가 2의 거듭제곱일 때만 맞다.

---

## 8. 성능과 메모리 관점

## 8.1 직접 shift를 쓰는 것이 항상 더 빠른가?

아니다.

예를 들어:

```c
x = x * 8;
```

현대 compiler는 최적화 옵션이 켜져 있으면 이를 shift나 더 효율적인 instruction으로 바꿀 수 있다.

따라서 산술 의미라면:

```c
x * 8
```

을 쓰는 것이 더 낫다.

반대로 bit field 배치가 목적이라면:

```c
x << 3
```

이 더 명확하다.

비교:

| 의도                            | 좋은 표현    |
| ----------------------------- | -------- |
| 숫자를 8배로 만들기                   | `x * 8`  |
| bit field를 3칸 왼쪽으로 배치         | `x << 3` |
| 8로 나눈 나머지                     | `x % 8`  |
| power-of-two ring buffer wrap | `x & 7`  |

---

## 8.2 `%` 대신 `&`를 쓰면 항상 좋은가?

아니다.

```c
x % 8
```

는 compiler가 충분히:

```c
x & 7
```

과 유사하게 최적화할 수 있다.

그리고 다음 코드는 의미가 더 명확하다.

```c
remainder = x % 8;
```

반면 ring buffer처럼 크기가 2의 거듭제곱이라는 invariant를 의도적으로 사용하는 경우에는:

```c
idx = (idx + 1u) & (BUF_SIZE - 1u);
```

가 적절하다.

실무 판단:

```txt
산술 나머지 → %
bit mask / ring buffer invariant → &
```

---

## 8.3 Embedded에서는 왜 그래도 중요할까?

임베디드에서는 다음 이유로 shift와 mask를 직접 이해해야 한다.

1. **하드웨어 register는 bit 단위 의미를 가진다.**
2. **MCU에 따라 나눗셈 instruction이 느리거나 없을 수 있다.**
3. **메모리가 작아 flag packing이 중요하다.**
4. **protocol field를 직접 조립/분해해야 한다.**
5. **compiler 최적화를 이해해야 코드 리뷰가 가능하다.**

하지만 그렇다고 무조건 bit trick을 남발하면 안 된다.

좋은 embedded C 코드는 보통 이렇게 쓴다.

```c
#define UART_BAUD_SHIFT  8u
#define UART_BAUD_MASK   (UINT32_C(0xFF) << UART_BAUD_SHIFT)

static inline uint32_t uart_baud_field(uint32_t baud_code) {
    return (baud_code & 0xFFu) << UART_BAUD_SHIFT;
}
```

즉:

```txt
비트 연산은 쓰되, 의미 있는 이름으로 감싼다.
```

---

## 9. 흔한 오해

## 오해 1: `x << 1`은 항상 `x * 2`와 같다

`unsigned`에서 modulo 관점으로는 예측 가능하다.

```c
uint32_t x = UINT32_MAX;
uint32_t y = x << 1;
```

이 경우 상위 bit가 버려진다.

하지만 `signed int`에서는 위험하다.

```c
int x = 1073741824;  // 2^30
int y = x << 1;      // 2^31, signed int 범위 문제
```

이건 undefined behavior가 될 수 있다.

안전한 bit-level 코드:

```c
uint32_t x = UINT32_C(1) << 30;
uint32_t y = x << 1;
```

---

## 오해 2: `x >> 1`은 항상 `x / 2`와 같다

`unsigned`에서는:

```txt
x >> 1 = floor(x / 2)
```

하지만 signed 음수에서는 조심해야 한다.

예:

```c
int x = -3;
```

C에서:

```c
x / 2
```

는 0 방향으로 truncate된다.

```txt
-3 / 2 = -1
```

하지만 arithmetic right shift라면:

```txt
-3 >> 1 = -2
```

가 될 수 있다.

왜냐하면 음수의 arithmetic shift는 보통 floor division처럼 동작하기 때문이다.

즉:

```txt
-3 / 2   = -1   // C integer division truncates toward zero
-3 >> 1  = -2   // arithmetic shift라면
```

따라서 signed 음수에 대해 `>>`를 `/ 2` 대체로 쓰면 위험하다.

---

## 오해 3: `%`는 항상 느리니 `&`로 바꿔야 한다

아니다.

상수가 2의 거듭제곱이면 compiler가 최적화할 수 있다.

```c
x % 8
```

은 최적화 후 내부적으로 `x & 7`에 가까워질 수 있다.

그리고 signed 값에서는 `%`와 `&`가 의미가 다르다.

예:

```c
int x = -1;
```

```txt
x % 8 = -1
```

하지만 two’s complement bit pattern 기준으로:

```txt
x & 7 = 7
```

따라서 signed 값에서는 절대 무턱대고 바꾸면 안 된다.

---

## 오해 4: shift count는 아무 값이나 가능하다

아니다.

다음은 위험하다.

```c
uint32_t x = 1u << 32;
```

`uint32_t`의 width가 32라면 shift count 32는 undefined behavior다.

안전한 코드:

```c
#include <stdint.h>
#include <stdbool.h>

bool make_bit_mask_u32(unsigned bit, uint32_t *out) {
    if (out == 0 || bit >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << bit;
    return true;
}
```

---

## 10. 반례 또는 실패 사례

## 실패 사례 1: signed left shift overflow

```c
int x = 1 << 31;
```

많은 사람이 “sign bit를 켜는 코드”라고 생각한다.

하지만 `1`은 signed `int`다.
32-bit int 환경에서 `1 << 31`은 signed int로 표현 가능한 양수 범위를 넘을 수 있다.

더 안전한 코드:

```c
#include <stdint.h>

uint32_t x = UINT32_C(1) << 31;
```

이 경우 의도가 명확하다.

```txt
32-bit unsigned mask의 31번 bit를 켠다.
```

---

## 실패 사례 2: signed right shift를 division으로 사용

```c
int div2_bad(int x) {
    return x >> 1;
}
```

양수에 대해서는 대체로 맞다.

```txt
8 >> 1 = 4
```

하지만 음수에서 `/ 2`와 다를 수 있다.

```c
int div2_good(int x) {
    return x / 2;
}
```

산술적 나눗셈 의도라면 `/`를 써라.

bit pattern 이동 의도라면 unsigned로 변환해서 `>>`를 써라.

---

## 실패 사례 3: `&`로 modulo를 잘못 구현

```c
uint32_t mod100_bad(uint32_t x) {
    return x & 99u;
}
```

이건 `x % 100`이 아니다.

`& (N - 1)`은 `N`이 2의 거듭제곱일 때만 `x % N`과 같다.

올바른 코드:

```c
uint32_t mod100_good(uint32_t x) {
    return x % 100u;
}
```

2의 거듭제곱일 때만:

```c
uint32_t mod128_fast(uint32_t x) {
    return x & 127u;
}
```

---

## 실패 사례 4: small integer type promotion 착각

다음 코드를 보자.

```c
#include <stdint.h>

uint8_t x = 0x80;
uint8_t y = x << 1;
```

많은 사람이 `uint8_t`에서 바로 shift된다고 생각한다.

하지만 C에서는 `uint8_t`가 보통 `int`로 integer promotion된 후 shift된다.

즉 개념적으로:

```c
uint8_t y = (uint8_t)((int)x << 1);
```

결과는 마지막에 다시 `uint8_t`로 잘린다.

`0x80 << 1 = 0x100`이고, 이를 `uint8_t`에 넣으면 `0x00`이 된다.

더 명확하게 쓰려면:

```c
uint8_t y = (uint8_t)((uint32_t)x << 1);
```

또는 애초에 working type을 넓게 둔다.

```c
uint32_t temp = x;
temp <<= 1;
```

---

## 11. Portability-safe coding style

## 11.1 bit mask 생성

좋은 스타일:

```c
#include <stdint.h>

#define BIT_U32(n) (UINT32_C(1) << (n))
```

하지만 이 macro도 `n >= 32`면 위험하다.

외부 입력이면 함수로 검사한다.

```c
#include <stdint.h>
#include <stdbool.h>

bool bit_u32(unsigned n, uint32_t *out) {
    if (out == 0 || n >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << n;
    return true;
}
```

---

## 11.2 field 추출

```c
#include <stdint.h>

#define FIELD_SHIFT 8u
#define FIELD_MASK  UINT32_C(0xFF)

uint32_t extract_field(uint32_t reg) {
    return (reg >> FIELD_SHIFT) & FIELD_MASK;
}
```

---

## 11.3 field 삽입

```c
#include <stdint.h>

#define FIELD_SHIFT 8u
#define FIELD_MASK  (UINT32_C(0xFF) << FIELD_SHIFT)

uint32_t insert_field(uint32_t reg, uint32_t value) {
    value &= 0xFFu;

    reg &= ~FIELD_MASK;
    reg |= value << FIELD_SHIFT;

    return reg;
}
```

---

## 11.4 signed 산술은 산술 연산자를 써라

좋음:

```c
int half(int x) {
    return x / 2;
}
```

덜 좋음:

```c
int half_bad(int x) {
    return x >> 1;
}
```

---

## 11.5 bit manipulation은 unsigned로 하라

좋음:

```c
uint32_t mask = UINT32_C(1) << 31;
```

나쁨:

```c
int mask = 1 << 31;
```

---

## 12. 확인 문제

## 문제 1

다음 값의 결과를 binary로 계산해라.

```txt
x = 00010110

x & 1 = ?
x << 1 = ?
x >> 1 = ?
```

---

## 문제 2

다음 코드는 무엇을 검사하는가?

```c
if ((x & 1u) == 0u) {
    // ...
}
```

---

## 문제 3

다음 두 코드는 항상 같은가?

```c
uint32_t a = x % 16u;
uint32_t b = x & 15u;
```

`x`가 `uint32_t`일 때와 `int32_t`일 때를 나누어 설명해라.

---

## 문제 4

다음 코드는 왜 위험한가?

```c
int mask = 1 << 31;
```

---

## 문제 5

다음 코드는 언제 올바른가?

```c
idx = (idx + 1u) & (size - 1u);
```

조건을 정확히 말해라.

---

## 문제 6

다음 코드의 결과가 `/ 2`와 다를 수 있는 이유를 설명해라.

```c
int half(int x) {
    return x >> 1;
}
```

특히 `x = -3`을 생각해라.

---

## 13. 실습 과제

## 실습 1: 홀짝 판별 함수

```c
#include <stdint.h>
#include <stdbool.h>

bool is_odd_u32(uint32_t x);
bool is_even_u32(uint32_t x);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>

bool is_odd_u32(uint32_t x) {
    return (x & 1u) != 0u;
}

bool is_even_u32(uint32_t x) {
    return (x & 1u) == 0u;
}
```

---

## 실습 2: power-of-two modulo 함수

`size`가 2의 거듭제곱일 때만 `x % size`를 mask로 계산하는 안전한 함수를 작성해라.

```c
#include <stdint.h>
#include <stdbool.h>

bool mod_power_of_two_u32(uint32_t x, uint32_t size, uint32_t *out);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>

static bool is_power_of_two_u32(uint32_t x) {
    return x != 0u && (x & (x - 1u)) == 0u;
}

bool mod_power_of_two_u32(uint32_t x, uint32_t size, uint32_t *out) {
    if (out == 0 || !is_power_of_two_u32(size)) {
        return false;
    }

    *out = x & (size - 1u);
    return true;
}
```

---

## 실습 3: register field 추출

다음 register layout이 있다고 하자.

```txt
bits  0..3  : STATUS
bits  4..7  : MODE
bits  8..15 : VALUE
```

다음 함수를 작성해라.

```c
#include <stdint.h>

uint32_t get_status(uint32_t reg);
uint32_t get_mode(uint32_t reg);
uint32_t get_value(uint32_t reg);
```

예상 구현:

```c
#include <stdint.h>

uint32_t get_status(uint32_t reg) {
    return reg & 0xFu;
}

uint32_t get_mode(uint32_t reg) {
    return (reg >> 4) & 0xFu;
}

uint32_t get_value(uint32_t reg) {
    return (reg >> 8) & 0xFFu;
}
```

---

## 실습 4: register field 삽입

`MODE` field, 즉 bit 4..7에 값을 넣는 함수를 작성해라.

```c
#include <stdint.h>

uint32_t set_mode(uint32_t reg, uint32_t mode);
```

예상 구현:

```c
#include <stdint.h>

#define MODE_SHIFT 4u
#define MODE_MASK  (UINT32_C(0xF) << MODE_SHIFT)

uint32_t set_mode(uint32_t reg, uint32_t mode) {
    mode &= 0xFu;

    reg &= ~MODE_MASK;
    reg |= mode << MODE_SHIFT;

    return reg;
}
```

---

## 실습 5: 안전한 bit mask 생성

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask_u32(unsigned bit, uint32_t *out);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>

bool make_mask_u32(unsigned bit, uint32_t *out) {
    if (out == 0 || bit >= 32u) {
        return false;
    }

    *out = UINT32_C(1) << bit;
    return true;
}
```

---

## 14. 핵심 정리

이번 강의 핵심은 다음이다.

1. **`x & 1`은 최하위 bit를 검사한다.**
2. **최하위 bit가 1이면 홀수, 0이면 짝수다.**
3. **`x << n`은 bit pattern을 왼쪽으로 n칸 이동한다.**
4. **unsigned에서 `x << n`은 대략 `x * 2^n`처럼 보이지만, overflow 시 modulo 효과가 난다.**
5. **`x >> n`은 unsigned 기준으로 `floor(x / 2^n)`와 같다.**
6. **signed 음수 right shift는 portability 문제가 있으므로 `/ 2` 대체로 쓰면 안 된다.**
7. **`x % 2^n`은 unsigned에서 `x & (2^n - 1)`로 표현할 수 있다.**
8. **단, `& (size - 1)` 방식은 size가 2의 거듭제곱일 때만 맞다.**
9. **산술 의도면 `*`, `/`, `%`를 쓰고, bit field 의도면 shift와 mask를 써라.**
10. **bit manipulation에서는 unsigned fixed-width type을 우선 사용하라.**

---

## 15. 면접 대비 핵심 문장

면접에서는 이렇게 설명하면 된다.

> `x & 1` checks the least significant bit. If the least significant bit is 1, the number is odd; otherwise, it is even.

한국어로는:

> `x & 1`은 최하위 bit를 검사합니다. 이 bit는 1의 자리이므로, 값이 1이면 홀수이고 0이면 짝수입니다.

Shift에 대해서는 이렇게 말하면 된다.

> For unsigned integers, left shift by `n` is equivalent to multiplication by `2^n` modulo the integer width, and right shift by `n` is equivalent to floor division by `2^n`.

한국어로는:

> unsigned 정수에서 `x << n`은 bit를 왼쪽으로 n칸 이동시키며, overflow를 제외하고는 `x * 2^n`처럼 볼 수 있습니다. `x >> n`은 왼쪽에 0을 채우는 logical shift이며, `floor(x / 2^n)`와 같습니다.

실무 판단은 이렇게 정리하면 좋다.

> If the intent is arithmetic, use arithmetic operators like `*`, `/`, and `%`. If the intent is bit-field manipulation, register control, or masking, use shift and bitwise operators. Modern compilers already optimize many arithmetic operations, so readability and portability matter.

한국어로는:

> 산술 계산이 목적이면 `*`, `/`, `%`를 쓰는 것이 좋고, register field 조작이나 bit mask가 목적이면 shift와 bitwise operator를 쓰는 것이 좋습니다. 현대 컴파일러는 많은 산술 최적화를 자동으로 수행하므로, 단순히 빠르다는 이유만으로 bit trick을 남발하면 안 됩니다.

---