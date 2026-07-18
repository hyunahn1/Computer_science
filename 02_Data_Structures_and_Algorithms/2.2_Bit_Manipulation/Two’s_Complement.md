# Lecture 7. Two’s Complement

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **컴퓨터는 음수를 어떻게 bit pattern으로 표현하는가?**

예를 들어 `+5`는 8-bit로 쉽게 표현할 수 있다.

```txt
+5 = 00000101
```

그런데 `-5`는 어떻게 표현할까?

현대 컴퓨터 시스템에서는 거의 항상 **two’s complement**, 즉 **2의 보수** 방식을 사용한다.

```txt
-5 = 11111011   // 8-bit two's complement
```

이번 강의의 핵심은 이것이다.

> **Two’s complement에서는 음수를 “양수의 bit를 뒤집고 1을 더한 값”으로 표현한다.**

---

# 2. 형식적 정의

## 2.1 N-bit unsigned range

N-bit unsigned integer는 다음 범위를 가진다.

```txt
0 ~ 2^N - 1
```

예를 들어 8-bit unsigned는:

```txt
0 ~ 255
```

```txt
00000000 = 0
00000001 = 1
01111111 = 127
10000000 = 128
11111111 = 255
```

---

## 2.2 N-bit signed two’s complement range

N-bit signed two’s complement integer는 다음 범위를 가진다.

```txt
-2^(N-1) ~ 2^(N-1) - 1
```

예를 들어 8-bit signed는:

```txt
-128 ~ 127
```

| Bit pattern | Signed value |
| ----------- | -----------: |
| `00000000`  |            0 |
| `00000001`  |            1 |
| `01111111`  |          127 |
| `10000000`  |         -128 |
| `10000001`  |         -127 |
| `11111111`  |           -1 |

중요한 점:

```txt
MSB = 0 → non-negative
MSB = 1 → negative
```

MSB는 Most Significant Bit, 즉 가장 왼쪽 bit다.

---

## 2.3 Two’s complement의 수학적 정의

N-bit pattern을 unsigned 값 `U`로 해석했을 때, signed 값은 다음과 같다.

```txt
if U < 2^(N-1):
    value = U
else:
    value = U - 2^N
```

예: 8-bit에서 `11111011`

```txt
11111011 = unsigned 251
```

MSB가 1이므로 signed value는:

```txt
251 - 256 = -5
```

따라서:

```txt
11111011 = -5
```

---

# 3. 직관적 설명

Two’s complement를 직관적으로 이해하려면 숫자 원을 생각하면 된다.

8-bit unsigned는 `0~255`까지 표현하고, 255 다음은 다시 0으로 돌아간다.

```txt
0 → 1 → 2 → ... → 254 → 255 → 0
```

이것을 modulo 256 세계라고 볼 수 있다.

그런데 signed two’s complement에서는 이 원의 절반을 음수로 해석한다.

```txt
00000000 ~ 01111111  → 0 ~ 127
10000000 ~ 11111111  → -128 ~ -1
```

즉 같은 bit pattern을 unsigned로 보면 큰 양수이고, signed로 보면 음수일 수 있다.

```txt
11111111

unsigned 해석: 255
signed 해석  : -1
```

핵심:

> **bit pattern은 그대로이고, type이 그 bit pattern을 어떻게 해석할지 결정한다.**

---

# 4. 왜 필요한지

Two’s complement는 단순히 음수 표현 방식이 아니다.
시스템 프로그래밍에서 매우 중요하다.

| 상황                   | Two’s complement가 중요한 이유                  |
| -------------------- | ----------------------------------------- |
| C signed/unsigned 이해 | 같은 bit pattern이 다르게 해석됨                   |
| Overflow 이해          | unsigned wrap-around와 signed overflow 차이  |
| Assembly/CPU 이해      | 덧셈과 뺄셈 회로가 단순해짐                           |
| Embedded register    | register 값이 signed field일 수 있음            |
| Sensor data          | ADC, IMU, 온도 센서가 signed raw value를 줄 수 있음 |
| Network/file parsing | raw bytes를 signed integer로 해석해야 함         |
| Debugging            | `0xFFFFFFFF`가 `-1`인지 `4294967295`인지 구분    |
| Bit tricks           | `~x + 1`, sign extension, mask 처리         |

예를 들어 embedded sensor가 16-bit raw temperature 값을 준다고 하자.

```txt
0xFFEC
```

이걸 unsigned로 보면:

```txt
65516
```

하지만 signed 16-bit two’s complement로 보면:

```txt
-20
```

센서 데이터 파싱에서 이런 해석 차이를 모르면 값이 완전히 틀어진다.

---

# 5. 내부 원리

## 5.1 양수 5의 8-bit 표현

```txt
+5 = 00000101
```

---

## 5.2 `-5` 만들기: one’s complement + 1

Two’s complement에서 `-x`를 만드는 절차는 다음과 같다.

```txt
1. x의 bit를 모두 뒤집는다.    // one's complement
2. 1을 더한다.                // +1
```

`+5`:

```txt
00000101
```

bit를 모두 뒤집는다.

```txt
11111010
```

1을 더한다.

```txt
11111010
+       1
---------
11111011
```

따라서:

```txt
-5 = 11111011
```

---

## 5.3 왜 이것이 -5인가?

8-bit 세계에서는 modulo 256으로 생각할 수 있다.

`+5`와 `-5`를 더하면 0이 되어야 한다.

```txt
  00000101   // +5
+ 11111011   // -5
-----------
1 00000000
```

결과는 9-bit로 보면 `1 00000000`이다.
하지만 8-bit에서는 아래 8 bits만 남는다.

```txt
00000000
```

즉:

```txt
5 + (-5) = 0
```

이 성질 때문에 `11111011`을 `-5`로 해석할 수 있다.

---

## 5.4 `-1`의 표현

`+1`:

```txt
00000001
```

bit 반전:

```txt
11111110
```

1 더하기:

```txt
11111111
```

따라서:

```txt
-1 = 11111111
```

이것은 매우 중요하다.

C에서 unsigned로 보면:

```c
uint8_t x = 0xFF;  // 255
```

signed 8-bit로 보면:

```c
int8_t y = -1;     // 보통 0xFF
```

같은 bit pattern이다.

```txt
11111111
```

해석만 다르다.

---

# 6. 단계별 예시

## 예시 1: `-5`의 8-bit 표현

```txt
+5:
00000101

one's complement:
11111010

+1:
11111011

따라서:
-5 = 11111011
```

검산:

```txt
  00000101
+ 11111011
-----------
1 00000000

8-bit 결과:
00000000
```

---

## 예시 2: `-128`의 8-bit 표현

8-bit signed two’s complement의 최소값은 `-128`이다.

```txt
10000000 = -128
```

왜냐하면 unsigned로 보면 `128`이고:

```txt
128 - 256 = -128
```

이다.

특이한 점:

```txt
-128의 양수 counterpart인 +128은 8-bit signed에 존재하지 않는다.
```

8-bit signed의 범위는:

```txt
-128 ~ 127
```

따라서 `+128`은 표현할 수 없다.

이것이 나중에 `INT_MIN` 문제가 되는 이유다.

---

## 예시 3: `~x + 1`

unsigned에서 two’s complement negation을 직접 계산할 수 있다.

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint8_t x = 5;
    uint8_t neg = (uint8_t)(~x + 1u);

    printf("x   = %u\n", x);
    printf("neg = 0x%02X\n", neg);

    return 0;
}
```

개념적 결과:

```txt
x   = 00000101
~x  = 11111010
+1  = 11111011
```

즉:

```txt
neg = 0xFB
```

이 bit pattern을 `int8_t`로 해석하면 `-5`다.

주의:

`uint8_t`는 C에서 연산 시 `int`로 promotion될 수 있으므로, 실제 코드에서는 cast와 type을 명확히 해야 한다.

---

## 예시 4: 같은 bit pattern, 다른 해석

```c
#include <stdint.h>
#include <stdio.h>
#include <inttypes.h>

int main(void) {
    uint8_t u = 0xFF;
    int8_t s = (int8_t)u;

    printf("unsigned: %" PRIu8 "\n", u);
    printf("signed  : %" PRId8 "\n", s);

    return 0;
}
```

대부분의 현대 시스템에서 출력은:

```txt
unsigned: 255
signed  : -1
```

단, `uint8_t`에서 `int8_t`로 범위를 벗어나는 값을 변환할 때의 세부 동작은 C 표준 관점에서 구현 정의 영역이 있을 수 있다.
실무에서는 대부분 two’s complement 시스템이라 위처럼 동작하지만, portable한 코드에서는 이런 변환을 신중하게 다뤄야 한다.

---

# 7. 덧셈과 뺄셈이 같은 회로로 동작하는 이유

Two’s complement의 가장 큰 장점은 이것이다.

> **뺄셈을 덧셈으로 바꿀 수 있다.**

```txt
a - b = a + (-b)
```

그리고 `-b`는:

```txt
~b + 1
```

으로 만들 수 있다.

예를 들어:

```txt
7 - 5
```

는:

```txt
7 + (-5)
```

로 바뀐다.

8-bit로 계산:

```txt
  00000111   // +7
+ 11111011   // -5
-----------
1 00000010
```

아래 8-bit만 보면:

```txt
00000010 = 2
```

즉:

```txt
7 - 5 = 2
```

CPU 입장에서는 별도의 복잡한 뺄셈 시스템 없이:

```txt
1. b를 invert
2. carry-in으로 1을 넣음
3. adder로 더함
```

이렇게 뺄셈을 처리할 수 있다.

---

# 8. Zero가 하나만 있는 이유

옛날 음수 표현 방식 중에는 **sign-magnitude**와 **one’s complement**가 있었다.

## 8.1 Sign-magnitude

가장 왼쪽 bit를 부호로 사용한다.

```txt
00000101 = +5
10000101 = -5
```

문제는 zero가 두 개 생긴다.

```txt
00000000 = +0
10000000 = -0
```

---

## 8.2 One’s complement

음수는 bit를 뒤집어서 만든다.

```txt
+5 = 00000101
-5 = 11111010
```

이 방식도 zero가 두 개 생긴다.

```txt
+0 = 00000000
-0 = 11111111
```

---

## 8.3 Two’s complement

음수는 bit를 뒤집고 1을 더한다.

```txt
+0 = 00000000
~0 = 11111111
+1 = 00000000   // 8-bit overflow로 다시 0
```

즉:

```txt
-0 = 0
```

따라서 zero가 하나만 있다.

이것이 two’s complement의 큰 장점이다.

---

# 9. 실제 응용

## 9.1 Embedded sensor raw value 해석

센서에서 16-bit raw 값이 들어온다고 하자.

```txt
high byte = 0xFF
low byte  = 0xEC
```

합치면:

```txt
0xFFEC
```

이 값은 unsigned로는:

```txt
65516
```

하지만 signed 16-bit로는:

```txt
-20
```

C 코드:

```c
#include <stdint.h>

int16_t parse_i16_be(uint8_t high, uint8_t low) {
    uint16_t u = ((uint16_t)high << 8) | (uint16_t)low;
    return (int16_t)u;
}
```

주의:

이 코드는 대부분의 현대 two’s complement 환경에서는 기대대로 동작한다.
엄격한 portability가 필요한 경우에는 signed 변환의 구현 정의성을 피하기 위해 직접 계산할 수 있다.

```c
#include <stdint.h>

int32_t parse_i16_be_portable(uint8_t high, uint8_t low) {
    uint16_t u = ((uint16_t)high << 8) | (uint16_t)low;

    if (u < 0x8000u) {
        return (int32_t)u;
    }

    return (int32_t)u - 0x10000;
}
```

예:

```txt
u = 0xFFEC = 65516
65516 - 65536 = -20
```

이 방식은 two’s complement 해석을 수학적으로 직접 구현한다.

---

## 9.2 `-1`을 mask로 사용하는 경우

Two’s complement에서 `-1`은 모든 bit가 1이다.

```txt
-1 = 11111111 11111111 11111111 11111111
```

그래서 이런 코드를 볼 수 있다.

```c
uint32_t all_ones = UINT32_MAX;
```

혹은:

```c
uint32_t all_ones = ~UINT32_C(0);
```

권장하는 표현은 보통 더 명확한 것이다.

```c
uint32_t all_ones = UINT32_MAX;
```

`-1`을 unsigned에 넣는 것도 동작은 정의되어 있지만, 의도가 덜 명확할 수 있다.

```c
uint32_t all_ones = (uint32_t)-1;
```

실무에서는 `UINT32_MAX`가 더 읽기 좋다.

---

## 9.3 Error code에서 `-1`

C API에서 실패를 `-1`로 반환하는 경우가 많다.

```c
int result = read_sensor();

if (result == -1) {
    // error
}
```

`-1`이 내부적으로 모든 bit가 1이라는 사실 때문에, low-level API나 assembly debugging에서 자주 보인다.

```txt
-1 = 0xFFFFFFFF   // 32-bit two's complement
```

하지만 C 코드에서 error code를 bit mask처럼 무조건 해석하면 안 된다.
의도는 API 문서 기준으로 판단해야 한다.

---

## 9.4 Sign extension

작은 signed 값을 큰 signed 값으로 확장할 때는 부호 bit가 유지되어야 한다.

예를 들어 8-bit `-5`:

```txt
11111011
```

이를 16-bit로 확장하면:

```txt
11111111 11111011
```

왼쪽이 1로 채워진다.

이것을 **sign extension**이라고 한다.

반대로 unsigned 확장은 0으로 채운다.

```txt
uint8_t  11111011
uint16_t 00000000 11111011
```

C 예시:

```c
#include <stdint.h>
#include <stdio.h>
#include <inttypes.h>

int main(void) {
    int8_t s = -5;
    int32_t extended = s;

    uint8_t u = 0xFB;
    uint32_t zero_extended = u;

    printf("%" PRId32 "\n", extended);       // -5
    printf("%" PRIu32 "\n", zero_extended);  // 251

    return 0;
}
```

같은 bit pattern `0xFB`라도 signed로 확장하느냐 unsigned로 확장하느냐에 따라 결과가 다르다.

---

# 10. 성능과 메모리 관점

## 10.1 Two’s complement의 하드웨어 장점

Two’s complement가 널리 쓰이는 이유는 하드웨어가 단순해지기 때문이다.

| 연산             | 장점                                |
| -------------- | --------------------------------- |
| 덧셈             | signed/unsigned 모두 같은 adder 사용 가능 |
| 뺄셈             | `a + (~b + 1)`로 처리 가능             |
| zero           | zero가 하나만 존재                      |
| 비교             | 하드웨어 설계가 단순해짐                     |
| sign extension | MSB 복사로 구현 가능                     |

중요한 점:

> **CPU의 bit-level 덧셈 회로는 signed와 unsigned를 구분하지 않고 같은 bit pattern을 더한다.**

구분은 해석 단계에서 발생한다.

예:

```txt
11111111 + 00000001 = 00000000  // 8-bit 결과
```

이걸 unsigned로 보면:

```txt
255 + 1 = 0  // modulo 256
```

signed로 보면:

```txt
-1 + 1 = 0
```

같은 회로, 다른 해석이다.

---

## 10.2 C에서 성능보다 중요한 것

Two’s complement 시스템이라고 해서 signed overflow를 마음대로 사용하면 안 된다.

위험한 코드:

```c
int x = 2147483647;
x = x + 1;  // signed overflow: undefined behavior
```

대부분의 CPU에서는 bit pattern이 `0x80000000`이 될 수 있다.
하지만 C 언어 관점에서는 signed overflow가 undefined behavior다.

즉 compiler는 이런 상황이 발생하지 않는다고 가정하고 최적화할 수 있다.

반면 unsigned overflow는 정의되어 있다.

```c
#include <stdint.h>

uint32_t x = UINT32_MAX;
x = x + 1u;  // defined: 0
```

따라서 wrap-around가 필요한 low-level code에서는 unsigned를 사용해야 한다.

---

# 11. 흔한 오해

## 오해 1: MSB가 1이면 항상 음수다

type에 따라 다르다.

```txt
11111111
```

`uint8_t`로 보면:

```txt
255
```

`int8_t`로 보면:

```txt
-1
```

즉 MSB가 1이면 음수라는 말은 **signed two’s complement로 해석할 때만** 맞다.

---

## 오해 2: signed overflow도 unsigned처럼 wrap-around된다

C에서는 아니다.

위험한 코드:

```c
int x = INT_MAX;
x++;
```

이건 signed overflow다.
C에서는 undefined behavior다.

안전한 wrap-around:

```c
uint32_t x = UINT32_MAX;
x++;
```

결과는 0으로 정의되어 있다.

---

## 오해 3: `abs(INT_MIN)`은 안전하다

안전하지 않다.

예를 들어 32-bit int에서:

```txt
INT_MIN = -2147483648
INT_MAX =  2147483647
```

`INT_MIN`의 양수 counterpart인 `2147483648`은 `int`로 표현할 수 없다.

따라서:

```c
#include <stdlib.h>
#include <limits.h>

int x = abs(INT_MIN);
```

은 문제가 된다.

직접 구현해도 마찬가지다.

```c
int my_abs(int x) {
    return x < 0 ? -x : x;
}
```

`x == INT_MIN`이면 `-x`가 표현 불가능하다.

---

## 오해 4: `-x == ~x + 1`은 C에서 항상 안전하다

수학적 two’s complement bit pattern으로는 맞다.

하지만 C signed integer에서 다음은 위험할 수 있다.

```c
int x = INT_MIN;
int y = -x;  // undefined behavior
```

`~x + 1`도 signed overflow 문제가 생길 수 있다.

bit pattern 조작이 목적이라면 unsigned로 해야 한다.

```c
#include <stdint.h>

uint32_t negate_pattern_u32(uint32_t x) {
    return ~x + 1u;
}
```

이 함수는 modulo `2^32`에서의 two’s complement negation을 계산한다.

---

## 오해 5: `char`는 signed라고 생각해도 된다

아니다.

C에서 plain `char`가 signed인지 unsigned인지는 구현에 따라 다를 수 있다.

```c
char c = 0xFF;
```

이 값이 `-1`처럼 동작할지 `255`처럼 동작할지는 환경에 따라 다를 수 있다.

low-level code에서는 명확하게 써라.

```c
int8_t s;
uint8_t u;
```

단, `int8_t`와 `uint8_t`는 해당 크기를 지원하는 환경에서만 제공된다. 대부분의 현대 시스템에서는 제공된다.

---

# 12. 반례 또는 실패 사례

## 실패 사례 1: signed overflow에 의존

```c
#include <limits.h>

int wrap_bad(int x) {
    return x + 1;
}
```

`x == INT_MAX`이면 signed overflow다.

```c
wrap_bad(INT_MAX);
```

C에서는 undefined behavior다.

wrap-around가 의도라면:

```c
#include <stdint.h>

uint32_t wrap_good(uint32_t x) {
    return x + 1u;
}
```

---

## 실패 사례 2: `INT_MIN` negation

```c
#include <limits.h>

int negate_bad(int x) {
    return -x;
}
```

문제:

```c
negate_bad(INT_MIN);
```

`INT_MIN`의 양수값은 `int` 범위에 없다.

안전한 방식은 목적에 따라 다르다.

### 방법 1: 더 넓은 타입 사용

```c
#include <stdint.h>

int64_t negate_i32_to_i64(int32_t x) {
    return -(int64_t)x;
}
```

### 방법 2: unsigned bit pattern으로 처리

```c
#include <stdint.h>

uint32_t twos_complement_negate_u32(uint32_t x) {
    return ~x + 1u;
}
```

이 함수는 signed 숫자 계산이 아니라 bit pattern 계산이다.

---

## 실패 사례 3: 센서 값을 unsigned로만 해석

센서 raw value:

```txt
0xFFEC
```

잘못된 코드:

```c
uint16_t raw = 0xFFEC;
printf("%u\n", raw);  // 65516
```

하지만 센서 문서가 signed 16-bit two’s complement라고 했다면 실제 값은:

```txt
-20
```

portable하게 해석:

```c
#include <stdint.h>

int32_t decode_i16(uint16_t raw) {
    if (raw < 0x8000u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x10000;
}
```

---

## 실패 사례 4: sign extension 실수

```c
#include <stdint.h>

uint8_t raw = 0xFB;
uint32_t value = raw;
```

결과:

```txt
value = 251
```

하지만 `raw`가 signed 8-bit `-5`를 의미한다면:

```c
int8_t s = (int8_t)raw;
int32_t value = s;
```

대부분의 현대 시스템에서는:

```txt
value = -5
```

더 portable하게 직접 해석할 수도 있다.

```c
#include <stdint.h>

int32_t decode_i8(uint8_t raw) {
    if (raw < 0x80u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x100;
}
```

---

# 13. Portability-safe coding style

## 13.1 bit pattern 조작은 unsigned로

좋음:

```c
uint32_t x = UINT32_C(0xFFFFFFFF);
uint32_t y = ~x + 1u;
```

나쁨:

```c
int x = -1;
int y = ~x + 1;
```

signed 값에 bit trick을 적용하면 representation, overflow, shift 문제 때문에 위험해질 수 있다.

---

## 13.2 signed 범위 체크

```c
#include <stdint.h>
#include <stdbool.h>
#include <limits.h>

bool add_i32_checked(int32_t a, int32_t b, int32_t *out) {
    if (out == NULL) {
        return false;
    }

    if ((b > 0 && a > INT32_MAX - b) ||
        (b < 0 && a < INT32_MIN - b)) {
        return false;
    }

    *out = a + b;
    return true;
}
```

이 코드는 signed overflow가 발생하기 전에 미리 검사한다.

---

## 13.3 raw bytes를 signed value로 해석

엄격한 해석이 필요하면 unsigned로 조립한 뒤 수학적으로 변환한다.

```c
#include <stdint.h>

int32_t decode_twos_complement(uint32_t raw, unsigned bits) {
    uint32_t sign_bit;
    uint32_t modulus;

    if (bits == 0u || bits >= 32u) {
        return 0;  // 실제 API라면 error 처리
    }

    sign_bit = UINT32_C(1) << (bits - 1u);
    modulus = UINT32_C(1) << bits;

    raw &= modulus - 1u;

    if ((raw & sign_bit) == 0u) {
        return (int32_t)raw;
    }

    return (int32_t)(raw - modulus);
}
```

예:

```txt
decode_twos_complement(0xFB, 8) = -5
decode_twos_complement(0xFFEC, 16) = -20
```

단, 위 함수는 `bits < 32`로 제한했다.
32-bit 전체를 다루려면 `uint64_t`를 사용해서 overflow 없이 계산하는 편이 안전하다.

---

# 14. 확인 문제

## 문제 1

`+5`를 8-bit two’s complement에서 `-5`로 바꾸는 과정을 써라.

```txt
+5       = ?
invert   = ?
+1       = ?
-5       = ?
```

---

## 문제 2

다음 bit pattern을 8-bit signed two’s complement로 해석해라.

```txt
11111111
10000000
11111011
01111111
```

---

## 문제 3

8-bit signed two’s complement의 범위는?

```txt
최소값 = ?
최대값 = ?
```

왜 양수 쪽이 하나 적은가?

---

## 문제 4

다음 코드는 왜 위험한가?

```c
int x = INT_MAX;
x = x + 1;
```

---

## 문제 5

다음 코드는 왜 위험한가?

```c
int x = INT_MIN;
int y = -x;
```

---

## 문제 6

다음 두 값은 같은 bit pattern일 수 있다. 각각 어떻게 해석되는가?

```c
uint8_t u = 0xFF;
int8_t s = -1;
```

---

## 문제 7

센서에서 `0xFFEC`가 들어왔다.
센서 문서에 “signed 16-bit two’s complement”라고 되어 있다면 이 값은 몇인가?

---

# 15. 실습 과제

## 실습 1: 8-bit two’s complement 직접 출력

`uint8_t x`를 받아서 `-x`에 해당하는 two’s complement bit pattern을 계산해라.

```c
#include <stdint.h>

uint8_t twos_neg_u8(uint8_t x) {
    return (uint8_t)(~x + 1u);
}
```

테스트:

```c
#include <stdio.h>
#include <stdint.h>

uint8_t twos_neg_u8(uint8_t x) {
    return (uint8_t)(~x + 1u);
}

int main(void) {
    uint8_t x = 5;
    uint8_t neg = twos_neg_u8(x);

    printf("0x%02X\n", neg);  // 0xFB

    return 0;
}
```

---

## 실습 2: signed 8-bit decode

`uint8_t raw`를 signed 8-bit two’s complement 값으로 직접 해석하는 함수를 만들어라.

```c
#include <stdint.h>

int32_t decode_i8(uint8_t raw) {
    if (raw < 0x80u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x100;
}
```

테스트:

```txt
0x00 → 0
0x01 → 1
0x7F → 127
0x80 → -128
0xFB → -5
0xFF → -1
```

---

## 실습 3: signed 16-bit big-endian sensor decode

센서가 high byte, low byte를 준다고 하자.

```c
#include <stdint.h>

int32_t decode_i16_be(uint8_t high, uint8_t low) {
    uint16_t raw = ((uint16_t)high << 8) | (uint16_t)low;

    if (raw < 0x8000u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x10000;
}
```

테스트:

```txt
0x00 0x14 → 20
0xFF 0xEC → -20
0x7F 0xFF → 32767
0x80 0x00 → -32768
```

---

## 실습 4: signed overflow 없는 덧셈

`int32_t` 두 개를 더하되 overflow가 발생하면 false를 반환하는 함수를 작성해라.

```c
#include <stdint.h>
#include <stdbool.h>

bool add_i32_checked(int32_t a, int32_t b, int32_t *out);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>
#include <limits.h>

bool add_i32_checked(int32_t a, int32_t b, int32_t *out) {
    if (out == NULL) {
        return false;
    }

    if ((b > 0 && a > INT32_MAX - b) ||
        (b < 0 && a < INT32_MIN - b)) {
        return false;
    }

    *out = a + b;
    return true;
}
```

---

# 16. 핵심 정리

이번 강의 핵심은 다음이다.

1. **Two’s complement는 현대 시스템에서 일반적으로 쓰이는 signed integer 표현 방식이다.**
2. **N-bit signed two’s complement 범위는 `-2^(N-1)`부터 `2^(N-1)-1`까지다.**
3. **음수 `-x`의 bit pattern은 `~x + 1`로 만들 수 있다.**
4. **8-bit에서 `-5`는 `11111011`이다.**
5. **같은 bit pattern도 unsigned와 signed에서 다르게 해석된다.**
6. **Two’s complement는 zero가 하나만 있다.**
7. **Two’s complement 덕분에 뺄셈을 `a + (~b + 1)` 형태로 처리할 수 있다.**
8. **CPU는 같은 adder로 signed/unsigned 덧셈을 처리할 수 있다.**
9. **C에서 unsigned overflow는 정의되어 있지만 signed overflow는 undefined behavior다.**
10. **`INT_MIN`은 양수로 바꿀 수 없으므로 `-INT_MIN`, `abs(INT_MIN)`은 위험하다.**
11. **Bit pattern 조작은 signed보다 unsigned fixed-width type으로 하는 것이 안전하다.**
12. **Embedded sensor raw data는 signed two’s complement인지 unsigned인지 반드시 문서를 확인해야 한다.**
13. **Sign extension과 zero extension을 구분해야 한다.**

---

# 17. 면접 대비 핵심 문장

면접에서는 이렇게 설명하면 된다.

> Two’s complement represents negative numbers by inverting all bits of the positive value and adding one. For example, in 8 bits, `+5` is `00000101`, so `-5` is `11111011`.

한국어로는:

> Two’s complement는 양수의 모든 bit를 반전한 뒤 1을 더해서 음수를 표현하는 방식입니다. 예를 들어 8-bit에서 `+5`는 `00000101`이고, 이를 반전한 뒤 1을 더하면 `11111011`, 즉 `-5`가 됩니다.

덧셈/뺄셈 관점에서는 이렇게 말하면 된다.

> The advantage of two’s complement is that addition and subtraction can use the same binary adder. Subtraction `a - b` can be implemented as `a + (~b + 1)`.

한국어로는:

> Two’s complement의 장점은 덧셈과 뺄셈을 같은 adder 회로로 처리할 수 있다는 점입니다. `a - b`는 `a + (~b + 1)`로 바꿀 수 있습니다.

C 안전성까지 포함하면:

> In C, unsigned arithmetic wraps around modulo `2^N`, but signed integer overflow is undefined behavior. Therefore, low-level bit manipulation should usually be done with unsigned fixed-width types.

한국어로는:

> C에서 unsigned 정수 연산은 `2^N` modulo로 wrap-around가 정의되어 있지만, signed integer overflow는 undefined behavior입니다. 그래서 low-level bit manipulation은 보통 `uint32_t` 같은 unsigned fixed-width type으로 작성하는 것이 안전합니다.

---