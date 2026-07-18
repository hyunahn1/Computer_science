# Lecture 2. XOR Operations

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **XOR는 왜 “같으면 0, 다르면 1”이라는 단순한 연산인데, bit manipulation에서 그렇게 자주 쓰이는가?**

XOR는 단순한 연산이지만 다음 성질 때문에 매우 강력하다.

```txt
A ^ A = 0
A ^ 0 = A
A ^ B = B ^ A
(A ^ B) ^ C = A ^ (B ^ C)
```

이 성질 덕분에 XOR는 다음 문제에 자주 쓰인다.

* 같은 값 제거하기
* 차이점 찾기
* bit toggle
* parity 계산
* 간단한 암호화
* missing/single number 문제
* checksum류 로직
* embedded flag 조작

---

## 2. 형식적 정의

## 2.1 XOR truth table

XOR는 **exclusive OR**이다.

두 bit가 다르면 1, 같으면 0이다.

|  A |  B | A ^ B |
| -: | -: | ----: |
|  0 |  0 |     0 |
|  0 |  1 |     1 |
|  1 |  0 |     1 |
|  1 |  1 |     0 |

즉:

```txt
same      → 0
different → 1
```

---

## 2.2 Boolean algebra 관점

XOR는 다음처럼 정의할 수 있다.

```txt
A ^ B = (A OR B) AND NOT (A AND B)
```

즉, 둘 중 하나만 1일 때 결과가 1이다.

```txt
A ^ B = 1 iff A != B
```

---

## 2.3 C에서의 XOR

C에서 XOR 연산자는 `^`이다.

```c
uint32_t c = a ^ b;
```

주의:

```c
2 ^ 3
```

은 `2`의 `3`제곱이 아니다.
C에서는 XOR다.

제곱은 `pow(2, 3)` 같은 함수를 써야 한다.

---

## 3. 직관적 설명

XOR의 가장 중요한 직관은 이것이다.

> **XOR는 두 bit가 다른 위치를 표시한다.**

예를 들어:

```txt
A = 10110010
B = 10100011
```

XOR를 하면:

```txt
    10110010
^   10100011
------------
    00010001
```

결과 `00010001`은 이런 뜻이다.

```txt
A와 B는 bit 4, bit 0에서 다르다.
```

즉 XOR는 단순 계산이 아니라:

> **두 bit pattern의 차이 지도**

처럼 볼 수 있다.

---

## 4. 왜 필요한지

XOR는 다음 상황에서 필요하다.

| 상황                | XOR가 유용한 이유                      |
| ----------------- | -------------------------------- |
| 값 비교              | 어느 bit가 다른지 바로 알 수 있음            |
| 중복 제거             | `A ^ A = 0` 성질 사용                |
| flag toggle       | 특정 bit를 뒤집을 수 있음                 |
| parity            | 1의 개수가 홀수인지 짝수인지 계산              |
| 간단한 암호화           | 같은 key로 암호화/복호화 가능               |
| embedded register | 특정 control bit toggle            |
| graphics          | pixel difference, mask operation |
| 알고리즘 면접           | single number, missing number 문제 |

특히 embedded에서는 register의 특정 bit를 반전시키는 일이 있다.

```c
GPIO_OUT ^= (1u << 5);
```

이 코드는 GPIO output register의 5번 bit를 toggle한다.

```txt
0이면 1로
1이면 0으로
```

---

## 5. 내부 원리

## 5.1 XOR는 bit별로 독립적으로 수행된다

예를 들어 8-bit 값 두 개가 있다고 하자.

```txt
A = 11001010
B = 01101100
```

XOR는 각 자리별로 독립적으로 계산된다.

```txt
bit index: 7 6 5 4 3 2 1 0

A:         1 1 0 0 1 0 1 0
B:         0 1 1 0 1 1 0 0
--------------------------------
A ^ B:     1 0 1 0 0 1 1 0
```

결과:

```txt
10100110
```

각 bit는 서로 영향을 주지 않는다.
carry도 없다. borrow도 없다.

그래서 XOR는 덧셈보다 단순한 논리 연산이다.

---

## 5.2 XOR와 덧셈의 차이

XOR와 덧셈은 겉으로 조금 비슷해 보일 때가 있다.

1-bit에서 보면:

|  A |  B | A + B의 sum bit | A ^ B |
| -: | -: | -------------: | ----: |
|  0 |  0 |              0 |     0 |
|  0 |  1 |              1 |     1 |
|  1 |  0 |              1 |     1 |
|  1 |  1 |     0, carry 1 |     0 |

즉 XOR는 binary addition에서 **carry를 무시한 sum bit**와 같다.

```txt
A ^ B = addition without carry
```

그래서 나중에 Lecture 10에서 `+` 없이 덧셈을 구현할 때 XOR가 등장한다.

```c
sum_without_carry = a ^ b;
carry             = (a & b) << 1;
```

지금은 깊게 들어가지 않아도 된다.
핵심은 이것이다.

> XOR는 carry 없는 덧셈처럼 동작한다.

---

## 6. XOR의 핵심 성질

## 6.1 `A ^ A = 0`

같은 bit는 XOR하면 0이다.

```txt
A = 10110100

    10110100
^   10110100
------------
    00000000
```

C 코드:

```c
uint32_t a = 0xB4;
uint32_t result = a ^ a;  // 0
```

이 성질 때문에 XOR는 “같은 값 제거”에 강하다.

---

## 6.2 `A ^ 0 = A`

0과 XOR하면 원래 값이 유지된다.

```txt
A = 10110100

    10110100
^   00000000
------------
    10110100
```

C 코드:

```c
uint32_t a = 0xB4;
uint32_t result = a ^ 0u;  // a
```

---

## 6.3 Commutative property

순서를 바꿔도 결과가 같다.

```txt
A ^ B = B ^ A
```

예:

```txt
5 ^ 3 = 3 ^ 5
```

binary:

```txt
5 = 0101
3 = 0011

0101 ^ 0011 = 0110
0011 ^ 0101 = 0110
```

---

## 6.4 Associative property

묶는 순서를 바꿔도 결과가 같다.

```txt
(A ^ B) ^ C = A ^ (B ^ C)
```

이 성질 때문에 배열 전체를 순서 상관없이 XOR할 수 있다.

```c
uint32_t result = 0;

for (size_t i = 0; i < n; ++i) {
    result ^= arr[i];
}
```

이런 코드가 가능한 이유가 associative property다.

---

## 7. 단계별 예시

## 예시 1: 두 값의 다른 bit 찾기

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint8_t a = 0b10110010;
    uint8_t b = 0b10100011;

    uint8_t diff = a ^ b;

    printf("diff = 0x%02X\n", diff);

    return 0;
}
```

계산:

```txt
a = 10110010
b = 10100011
------------
^ = 00010001
```

`diff`에서 1인 bit는 두 값이 다른 위치다.

---

## 예시 2: 특정 bit toggle

```c
#include <stdint.h>

#define LED_BIT (1u << 3)

int main(void) {
    uint8_t port = 0b00000000;

    port ^= LED_BIT;  // bit 3 toggle: 0 -> 1
    port ^= LED_BIT;  // bit 3 toggle: 1 -> 0

    return 0;
}
```

흐름:

```txt
초기:
00000000

LED_BIT:
00001000

1회 toggle:
00001000

2회 toggle:
00000000
```

XOR의 핵심:

```txt
x ^ 1 = flip
x ^ 0 = keep
```

즉 mask에서 1인 위치만 뒤집힌다.

```txt
value = 10110010
mask  = 00001111
----------------
^     = 10111101
```

아래 4bit만 뒤집힌다.

---

## 예시 3: swap without temp

고전적인 XOR swap이다.

```c
#include <stdio.h>

int main(void) {
    unsigned int a = 10;
    unsigned int b = 20;

    a = a ^ b;
    b = a ^ b;
    a = a ^ b;

    printf("a = %u, b = %u\n", a, b);

    return 0;
}
```

단계별로 보자.

처음:

```txt
a = A
b = B
```

1단계:

```txt
a = A ^ B
b = B
```

2단계:

```txt
b = a ^ b
  = (A ^ B) ^ B
  = A ^ (B ^ B)
  = A ^ 0
  = A
```

현재:

```txt
a = A ^ B
b = A
```

3단계:

```txt
a = a ^ b
  = (A ^ B) ^ A
  = (A ^ A) ^ B
  = 0 ^ B
  = B
```

결과:

```txt
a = B
b = A
```

### 하지만 실무에서는 거의 쓰지 않는다

요즘은 이렇게 쓰는 게 낫다.

```c
unsigned int temp = a;
a = b;
b = temp;
```

이유:

* XOR swap은 가독성이 나쁘다.
* compiler가 일반 swap을 충분히 최적화한다.
* 같은 메모리 위치를 swap하면 문제가 생긴다.
* 임베디드에서도 명확성이 더 중요할 때가 많다.

특히 포인터를 통해 swap할 때 위험하다.

```c
void xor_swap(unsigned int *a, unsigned int *b) {
    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}
```

만약 `a == b`이면?

```c
xor_swap(&x, &x);
```

그러면 값이 0이 될 수 있다.

따라서 실무에서는 이렇게 한다.

```c
void safe_swap(unsigned int *a, unsigned int *b) {
    if (a == b) {
        return;
    }

    unsigned int temp = *a;
    *a = *b;
    *b = temp;
}
```

면접에서는 XOR swap의 원리는 설명할 수 있어야 하지만, 실무에서 추천하지 않는다는 점까지 말하면 좋다.

---

## 예시 4: find single number

문제:

> 배열에서 모든 숫자는 두 번 나오고, 하나의 숫자만 한 번 나온다. 그 숫자를 찾아라.

예:

```txt
[4, 1, 2, 1, 2]
```

답:

```txt
4
```

XOR를 사용하면 된다.

```c
#include <stddef.h>
#include <stdint.h>
#include <stdio.h>

uint32_t find_single(const uint32_t arr[], size_t n) {
    uint32_t result = 0;

    for (size_t i = 0; i < n; ++i) {
        result ^= arr[i];
    }

    return result;
}

int main(void) {
    uint32_t arr[] = {4, 1, 2, 1, 2};
    size_t n = sizeof(arr) / sizeof(arr[0]);

    printf("%u\n", find_single(arr, n));

    return 0;
}
```

단계:

```txt
result = 0

0 ^ 4 = 4
4 ^ 1 = 5
5 ^ 2 = 7
7 ^ 1 = 6
6 ^ 2 = 4
```

수학적으로 보면:

```txt
4 ^ 1 ^ 2 ^ 1 ^ 2
= 4 ^ (1 ^ 1) ^ (2 ^ 2)
= 4 ^ 0 ^ 0
= 4
```

왜 순서를 마음대로 바꿀 수 있는가?

```txt
XOR는 commutative + associative이기 때문
```

---

## 예시 5: simple XOR cipher

XOR는 같은 key로 암호화와 복호화를 모두 할 수 있다.

```txt
encrypted = data ^ key
decrypted = encrypted ^ key
```

왜?

```txt
decrypted = (data ^ key) ^ key
          = data ^ (key ^ key)
          = data ^ 0
          = data
```

C 코드:

```c
#include <stdint.h>
#include <stdio.h>

void xor_cipher(uint8_t data[], size_t n, uint8_t key) {
    for (size_t i = 0; i < n; ++i) {
        data[i] ^= key;
    }
}

int main(void) {
    uint8_t msg[] = {'H', 'e', 'l', 'l', 'o'};
    size_t n = sizeof(msg) / sizeof(msg[0]);

    xor_cipher(msg, n, 0x5A);  // encrypt
    xor_cipher(msg, n, 0x5A);  // decrypt

    for (size_t i = 0; i < n; ++i) {
        putchar(msg[i]);
    }

    putchar('\n');
    return 0;
}
```

출력:

```txt
Hello
```

### 보안 주의

이건 암호학적으로 안전한 암호가 아니다.

특히 같은 key를 반복 사용하면 쉽게 깨진다.
실무 보안 시스템에서는 이런 단순 XOR cipher를 쓰면 안 된다.

다만 XOR는 실제 암호 알고리즘 내부에서도 많이 사용된다.
차이는 key stream 생성, random성, mode, 인증 구조가 훨씬 정교하다는 점이다.

---

## 8. 실제 응용

## 8.1 Embedded GPIO toggle

LED pin을 toggle하는 코드:

```c
#include <stdint.h>

#define LED_PIN 5
#define LED_MASK (UINT32_C(1) << LED_PIN)

volatile uint32_t GPIO_OUT;

void toggle_led(void) {
    GPIO_OUT ^= LED_MASK;
}
```

의미:

```txt
GPIO_OUT의 5번 bit만 반전
다른 bit는 그대로 유지
```

예:

```txt
GPIO_OUT = 00100000
LED_MASK = 00100000
-------------------
^        = 00000000
```

다시 toggle하면:

```txt
GPIO_OUT = 00000000
LED_MASK = 00100000
-------------------
^        = 00100000
```

---

## 8.2 상태 변화 감지

이전 상태와 현재 상태가 있을 때, 어떤 bit가 바뀌었는지 알고 싶다면 XOR를 쓴다.

```c
#include <stdint.h>

uint32_t changed_bits(uint32_t old_status, uint32_t new_status) {
    return old_status ^ new_status;
}
```

예:

```txt
old = 10110000
new = 10111001
--------------
^   = 00001001
```

결과 `00001001`은 bit 3과 bit 0이 바뀌었다는 뜻이다.

embedded에서 버튼 입력, 센서 상태, interrupt flag 변화를 감지할 때 유용하다.

---

## 8.3 특정 bit가 변했는지 확인

```c
#include <stdint.h>
#include <stdbool.h>

#define ERROR_BIT (1u << 2)

bool error_bit_changed(uint32_t old_status, uint32_t new_status) {
    return ((old_status ^ new_status) & ERROR_BIT) != 0u;
}
```

해석:

```txt
1. old_status ^ new_status
   → 바뀐 bit 위치를 1로 표시

2. & ERROR_BIT
   → 그중 ERROR_BIT만 확인

3. != 0
   → ERROR_BIT가 바뀌었는지 true/false
```

---

## 8.4 Parity 계산

Parity는 1의 개수가 홀수인지 짝수인지 나타낸다.

예:

```txt
1011 → 1이 3개 → odd parity
1001 → 1이 2개 → even parity
```

간단한 parity 계산:

```c
#include <stdint.h>

unsigned parity32(uint32_t x) {
    unsigned p = 0;

    while (x != 0u) {
        p ^= (x & 1u);
        x >>= 1;
    }

    return p;
}
```

의미:

```txt
1의 개수가 홀수면 1
1의 개수가 짝수면 0
```

더 빠른 방법이나 builtin은 나중에 popcount 강의에서 다룬다.

---

## 9. 성능과 메모리 관점

XOR는 일반적으로 CPU에서 매우 빠른 instruction이다.

대부분의 ISA에는 XOR instruction이 직접 있다.

예시 개념:

```asm
xor r0, r1
```

그러나 중요한 점이 있다.

> 빠르다고 해서 무조건 XOR trick을 써야 하는 것은 아니다.

예를 들어 swap은 이론적으로 XOR만으로 가능하지만:

```c
a ^= b;
b ^= a;
a ^= b;
```

실무에서는 다음이 더 좋다.

```c
tmp = a;
a = b;
b = tmp;
```

이유:

* compiler가 최적화 가능
* CPU register allocation이 좋으면 실제 메모리 비용이 거의 없음
* aliasing 문제를 피하기 쉬움
* 가독성이 좋음
* 유지보수가 쉬움

정리:

| 목적                       | XOR 사용 적합성 |
| ------------------------ | ---------- |
| bit 차이 확인                | 매우 적합      |
| bit toggle               | 매우 적합      |
| 중복 제거 알고리즘               | 적합         |
| 단순 swap 최적화              | 보통 부적합     |
| 암호화 학습 예제                | 가능         |
| 실제 보안 암호                 | 부적합        |
| embedded register toggle | 적합         |

---

## 10. signed / unsigned 관점

XOR는 bitwise operation이다.
그래서 가능하면 unsigned type으로 수행하는 것이 좋다.

권장:

```c
uint32_t x = 0xF0000000u;
uint32_t y = 0x0F000000u;
uint32_t z = x ^ y;
```

덜 권장:

```c
int x = 0xF0000000;
int y = 0x0F000000;
int z = x ^ y;
```

이유:

1. `int`의 크기는 환경에 따라 다를 수 있다.
2. sign bit가 얽히면 읽는 사람이 의미를 오해하기 쉽다.
3. 이후 shift 연산과 결합되면 signed right shift 문제가 생길 수 있다.
4. fixed-width type이 bit-level 의도를 더 명확히 표현한다.

bit manipulation에서는 보통 다음 스타일이 안전하다.

```c
#include <stdint.h>

uint32_t flags = 0;
uint32_t mask = UINT32_C(1) << 7;

flags ^= mask;
```

---

## 11. 흔한 오해

## 오해 1: XOR는 OR의 변형일 뿐이다

OR와 XOR는 다르다.

|  A |  B | A | B | A ^ B |
| -: | -: | ----: | ----: |
|  0 |  0 |     0 |     0 |
|  0 |  1 |     1 |     1 |
|  1 |  0 |     1 |     1 |
|  1 |  1 |     1 |     0 |

핵심 차이:

```txt
OR  : 하나라도 1이면 1
XOR : 정확히 하나만 1이면 1
```

---

## 오해 2: XOR swap은 temp swap보다 항상 빠르다

아니다.

옛날에는 임시 변수 없이 swap한다는 점에서 흥미로웠다.
하지만 현대 compiler와 CPU에서는 일반 swap이 더 좋은 경우가 많다.

```c
unsigned int tmp = a;
a = b;
b = tmp;
```

이 코드는 충분히 최적화될 수 있고, 사람이 읽기도 쉽다.

XOR swap은 면접에서 원리를 설명하는 용도로는 좋지만, 실무 코드에 무조건 쓰는 것은 좋지 않다.

---

## 오해 3: XOR cipher는 안전한 암호다

아니다.

```txt
cipher = plain ^ key
```

이 구조 자체는 유용하지만, 단순히 한 key를 반복해서 XOR하는 방식은 안전하지 않다.

특히 공격자가 plaintext 일부를 알면 key를 역추적할 수 있다.

```txt
cipher ^ plain = key
```

XOR는 암호학의 재료로는 중요하지만, 단독으로 안전한 암호가 아니다.

---

## 오해 4: XOR 결과가 작으면 두 값이 비슷하다

항상 그렇지 않다.

```txt
a = 01111111
b = 10000000
```

XOR:

```txt
a ^ b = 11111111
```

숫자로는 `127`과 `128`이라 차이가 1이지만, bit pattern은 모든 bit가 다르다.

반대로:

```txt
a = 00000000
b = 10000000
```

숫자 차이는 크지만, XOR 결과는 한 bit만 다르다.

```txt
a ^ b = 10000000
```

즉 XOR는 numeric distance가 아니라 **bitwise difference**를 나타낸다.

---

## 12. 반례 또는 실패 사례

## 실패 사례 1: XOR swap에서 같은 주소 전달

```c
void xor_swap(unsigned int *a, unsigned int *b) {
    *a ^= *b;
    *b ^= *a;
    *a ^= *b;
}
```

문제:

```c
unsigned int x = 10;
xor_swap(&x, &x);
```

`a`와 `b`가 같은 주소면 같은 값을 자기 자신과 XOR하게 된다.

```txt
x = x ^ x = 0
```

결과적으로 값이 망가진다.

해결:

```c
void swap(unsigned int *a, unsigned int *b) {
    if (a == b) {
        return;
    }

    unsigned int tmp = *a;
    *a = *b;
    *b = tmp;
}
```

---

## 실패 사례 2: single number 문제의 조건을 착각

XOR로 single number를 찾는 코드는 다음 조건에서만 제대로 동작한다.

```txt
모든 숫자는 정확히 두 번 나오고,
하나의 숫자만 한 번 나온다.
```

예를 들어:

```txt
[4, 1, 2, 1, 2]
```

는 가능하다.

하지만:

```txt
[4, 1, 2, 1, 2, 4, 7, 8]
```

여기서는 한 번만 나오는 값이 `7`, `8` 두 개다.

전체 XOR 결과는:

```txt
7 ^ 8
```

이지, `7`이나 `8` 하나가 아니다.

즉 문제 조건을 확인해야 한다.

---

## 실패 사례 3: signed 값을 bit pattern으로 착각

```c
int a = -1;
int b = 0xFF;

printf("%d\n", a ^ b);
```

이 코드는 환경에 따라 `int` 크기와 representation을 고려해야 해서 초보자에게 혼란을 준다.

대부분의 현대 시스템에서 `int`가 32-bit two’s complement라면:

```txt
a = -1    = 0xFFFFFFFF
b = 0xFF  = 0x000000FF
a ^ b     = 0xFFFFFF00
```

이를 signed int로 출력하면 음수로 보일 수 있다.

더 명확한 코드:

```c
#include <stdint.h>
#include <stdio.h>
#include <inttypes.h>

int main(void) {
    uint32_t a = UINT32_C(0xFFFFFFFF);
    uint32_t b = UINT32_C(0x000000FF);

    uint32_t result = a ^ b;

    printf("0x%08" PRIX32 "\n", result);
    return 0;
}
```

출력:

```txt
0xFFFFFF00
```

bit manipulation에서는 unsigned fixed-width type을 쓰면 해석이 명확해진다.

---

## 13. 확인 문제

## 문제 1

다음 결과는?

```c
#include <stdio.h>

int main(void) {
    printf("%d\n", 5 ^ 3);
    return 0;
}
```

힌트:

```txt
5 = 0101
3 = 0011
```

---

## 문제 2

다음 XOR 결과를 binary로 계산해라.

```txt
10101010
^ 11001100
----------
?
```

---

## 문제 3

다음 코드의 출력은?

```c
#include <stdio.h>
#include <stdint.h>

int main(void) {
    uint32_t x = 42;

    printf("%u\n", x ^ x);
    printf("%u\n", x ^ 0u);

    return 0;
}
```

---

## 문제 4

다음 배열에서 한 번만 등장하는 숫자는?

```txt
[9, 3, 7, 3, 9, 11, 11]
```

XOR 과정으로 설명해라.

---

## 문제 5

다음 함수는 어떤 일을 하는가?

```c
#include <stdint.h>

uint32_t f(uint32_t old_value, uint32_t new_value) {
    return old_value ^ new_value;
}
```

면접식으로 설명해라.

---

## 14. 실습 과제

## 실습 1: XOR difference 출력

두 `uint8_t` 값을 받아서 서로 다른 bit 위치를 출력하는 함수를 만들어라.

예:

```txt
a = 10110010
b = 10100011

different bits: 4, 0
```

시작 코드:

```c
#include <stdint.h>
#include <stdio.h>

void print_different_bits(uint8_t a, uint8_t b) {
    uint8_t diff = a ^ b;

    for (int i = 7; i >= 0; --i) {
        if (((diff >> i) & 1u) != 0u) {
            printf("bit %d is different\n", i);
        }
    }
}

int main(void) {
    print_different_bits(0b10110010, 0b10100011);
    return 0;
}
```

---

## 실습 2: toggle flag 함수 구현

다음 flag를 정의하고 toggle 함수를 만들어라.

```c
#include <stdint.h>

#define FLAG_READY  (UINT32_C(1) << 0)
#define FLAG_ERROR  (UINT32_C(1) << 1)
#define FLAG_BUSY   (UINT32_C(1) << 2)
#define FLAG_SLEEP  (UINT32_C(1) << 3)
```

함수:

```c
void toggle_flag(uint32_t *flags, uint32_t mask);
```

주의:

```c
flags == NULL
```

인 경우를 처리해라.

예상 구현:

```c
#include <stdint.h>

void toggle_flag(uint32_t *flags, uint32_t mask) {
    if (flags == 0) {
        return;
    }

    *flags ^= mask;
}
```

---

## 실습 3: single number 구현

다음 함수를 구현해라.

```c
#include <stddef.h>
#include <stdint.h>

uint32_t find_single(const uint32_t arr[], size_t n);
```

조건:

```txt
배열의 모든 값은 두 번 등장하고,
오직 하나의 값만 한 번 등장한다.
```

추가로 생각할 점:

```txt
arr == NULL이면 어떻게 할 것인가?
n == 0이면 어떻게 할 것인가?
```

현실적인 구현 예:

```c
#include <stddef.h>
#include <stdint.h>
#include <stdbool.h>

bool find_single_safe(const uint32_t arr[], size_t n, uint32_t *out) {
    if (arr == NULL || out == NULL || n == 0u) {
        return false;
    }

    uint32_t result = 0;

    for (size_t i = 0; i < n; ++i) {
        result ^= arr[i];
    }

    *out = result;
    return true;
}
```

다만 이 함수는 “입력 조건이 올바른지”까지 검증하지는 않는다.
즉, 실제로 모든 값이 두 번 등장하는지는 확인하지 않는다.

---

## 실습 4: XOR cipher 구현

문자열 buffer를 key 하나로 XOR하는 함수를 작성해라.

```c
#include <stddef.h>
#include <stdint.h>

void xor_cipher(uint8_t data[], size_t n, uint8_t key);
```

조건:

```txt
1. data == NULL이면 아무것도 하지 말 것
2. 같은 함수를 두 번 적용하면 원본으로 돌아와야 함
```

예상 구현:

```c
#include <stddef.h>
#include <stdint.h>

void xor_cipher(uint8_t data[], size_t n, uint8_t key) {
    if (data == NULL) {
        return;
    }

    for (size_t i = 0; i < n; ++i) {
        data[i] ^= key;
    }
}
```

테스트:

```c
#include <stdio.h>

int main(void) {
    uint8_t msg[] = {'T', 'e', 's', 't'};
    size_t n = sizeof(msg) / sizeof(msg[0]);

    xor_cipher(msg, n, 0x5A);  // encrypt
    xor_cipher(msg, n, 0x5A);  // decrypt

    for (size_t i = 0; i < n; ++i) {
        putchar(msg[i]);
    }

    putchar('\n');
    return 0;
}
```

출력:

```txt
Test
```

---

# 15. 핵심 정리

이번 강의의 핵심은 다음이다.

1. **XOR는 두 bit가 다르면 1, 같으면 0이다.**
2. **XOR는 bitwise difference를 나타낸다.**
3. **`A ^ A = 0`이므로 같은 값은 XOR로 제거된다.**
4. **`A ^ 0 = A`이므로 0은 XOR에서 항등원이다.**
5. **XOR는 commutative, associative이므로 순서와 묶음에 영향을 받지 않는다.**
6. **특정 bit toggle에는 XOR가 자연스럽다.**
7. **single number 문제는 XOR의 대표적인 면접 패턴이다.**
8. **XOR swap은 원리는 중요하지만 실무에서는 일반 swap이 보통 더 낫다.**
9. **단순 XOR cipher는 학습용으로는 좋지만 보안적으로 안전하지 않다.**
10. **bit manipulation에서는 signed보다 unsigned fixed-width type을 쓰는 것이 안전하다.**

---

# 16. 면접 대비 핵심 문장

면접에서는 이렇게 설명하면 된다.

> XOR returns 1 when two bits are different and 0 when they are the same. Because `A ^ A = 0` and `A ^ 0 = A`, XOR is useful for canceling duplicate values, detecting changed bits, and toggling specific bits.

한국어로는:

> XOR는 두 비트가 다르면 1, 같으면 0을 반환하는 연산입니다. `A ^ A = 0`, `A ^ 0 = A` 성질 때문에 중복 값을 제거하거나, 변경된 bit를 찾거나, 특정 bit를 toggle할 때 자주 사용됩니다.

single number 문제는 이렇게 설명할 수 있다.

> 배열에서 모든 값이 두 번 나오고 하나만 한 번 나온다면, 전체 값을 XOR하면 중복된 값들은 `A ^ A = 0`으로 사라지고, 한 번만 나온 값만 남습니다.

XOR swap은 이렇게 말하면 좋다.

> XOR swap은 임시 변수 없이 두 값을 바꿀 수 있지만, 가독성이 낮고 aliasing 문제가 있으며 현대 compiler가 일반 swap을 잘 최적화하므로 실무에서는 보통 temp variable을 사용하는 방식이 더 적절합니다.

---
