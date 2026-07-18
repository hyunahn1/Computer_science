# Lecture 8. Endianness

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **`uint32_t x = 0x12345678;`은 메모리에 어떤 순서로 저장되는가?**

정수 값 자체는 다음과 같다.

```txt
x = 0x12345678
```

하지만 메모리는 byte 단위로 주소가 붙는다.

`0x12345678`은 4 bytes로 나눌 수 있다.

```txt
0x12  0x34  0x56  0x78
```

문제는 이 byte들이 메모리에 어떤 순서로 들어가느냐이다.

```txt
낮은 주소 → 높은 주소

Big-endian:
0x12 0x34 0x56 0x78

Little-endian:
0x78 0x56 0x34 0x12
```

이 byte 순서 문제가 **endianness**다.

---

# 2. 형식적 정의

## 2.1 Endianness

**Endianness**는 multi-byte data를 메모리에 저장할 때, byte들을 어떤 순서로 배치하는지를 의미한다.

중요한 점:

> Endianness는 **byte order** 문제다.
> bit order 문제가 아니다.

예를 들어 32-bit 값:

```txt
0x12345678
```

은 byte 단위로 보면:

```txt
0x12 0x34 0x56 0x78
```

이 4개의 byte가 메모리에 어떤 순서로 놓이는지가 endian 문제다.

---

## 2.2 Big-endian

Big-endian은 **가장 중요한 byte**, 즉 MSB가 낮은 주소에 먼저 저장된다.

```txt
value = 0x12345678

address 낮음 → 높음

0x1000: 0x12
0x1001: 0x34
0x1002: 0x56
0x1003: 0x78
```

즉 사람이 쓰는 hexadecimal 순서와 메모리 순서가 같다.

```txt
0x12 0x34 0x56 0x78
```

---

## 2.3 Little-endian

Little-endian은 **가장 덜 중요한 byte**, 즉 LSB가 낮은 주소에 먼저 저장된다.

```txt
value = 0x12345678

address 낮음 → 높음

0x1000: 0x78
0x1001: 0x56
0x1002: 0x34
0x1003: 0x12
```

즉 hexadecimal로 보는 순서와 메모리 byte 순서가 반대처럼 보인다.

```txt
0x78 0x56 0x34 0x12
```

---

## 2.4 Network byte order

네트워크 프로토콜에서는 보통 **network byte order**를 사용한다.

```txt
network byte order = big-endian
```

따라서 little-endian CPU에서 네트워크 packet을 만들거나 읽을 때 endian conversion이 필요할 수 있다.

대표 함수:

```c
htonl(); // host to network long
htons(); // host to network short
ntohl(); // network to host long
ntohs(); // network to host short
```

단, 이 함수들은 ISO C 표준이 아니라 POSIX/BSD socket 계열 API다. Linux/macOS에서는 `<arpa/inet.h>`, Windows에서는 Winsock 계열을 사용한다.

---

# 3. 직관적 설명

## 3.1 숫자 값과 메모리 표현은 다르다

우리가 보는 값:

```txt
0x12345678
```

이 값 자체는 변하지 않는다.

하지만 메모리에는 byte 단위로 저장된다.

```txt
0x12 | 0x34 | 0x56 | 0x78
```

Big-endian은 사람이 읽는 순서대로 저장한다.

```txt
낮은 주소 → 높은 주소
12 34 56 78
```

Little-endian은 낮은 자리 byte부터 저장한다.

```txt
낮은 주소 → 높은 주소
78 56 34 12
```

핵심 직관:

> Big-endian은 “큰 자리부터 저장”한다.
> Little-endian은 “작은 자리부터 저장”한다.

---

## 3.2 Decimal 숫자로 비유

숫자 `1234`를 생각해보자.

자리값은 다음과 같다.

```txt
1 thousands
2 hundreds
3 tens
4 ones
```

Big-endian식 사고:

```txt
1 2 3 4
```

큰 자리부터 저장.

Little-endian식 사고:

```txt
4 3 2 1
```

작은 자리부터 저장.

물론 실제 컴퓨터는 decimal digit이 아니라 byte 단위로 저장한다.

```txt
0x12345678

big-endian    : 12 34 56 78
little-endian : 78 56 34 12
```

---

# 4. 왜 필요한지

Endianness를 모르면 다음 상황에서 버그가 난다.

| 상황                       | Endianness가 중요한 이유                                   |
| ------------------------ | ---------------------------------------------------- |
| 네트워크 프로토콜                | packet field는 정해진 byte order를 따라야 함                  |
| 파일 포맷                    | PNG, WAV, ELF 등 포맷마다 byte order 규칙이 있음               |
| Embedded register        | peripheral register byte order와 CPU access width가 중요 |
| Sensor data parsing      | high byte / low byte 조립 필요                           |
| Binary serialization     | 다른 CPU에서 같은 데이터로 해석해야 함                              |
| Debugging                | memory dump를 값으로 해석해야 함                              |
| Cross-platform C         | x86, ARM, MCU 간 데이터 교환                               |
| Cryptography/compression | byte order가 algorithm spec에 포함될 수 있음                 |

예를 들어 센서가 다음 두 byte를 보낸다고 하자.

```txt
high byte = 0x12
low byte  = 0x34
```

문서에 big-endian 16-bit value라고 되어 있으면 값은:

```txt
0x1234
```

하지만 실수로 little-endian처럼 조립하면:

```txt
0x3412
```

완전히 다른 값이 된다.

---

# 5. 내부 원리

## 5.1 `uint32_t x = 0x12345678`

정수 값:

```txt
x = 0x12345678
```

byte 분해:

```txt
MSB                         LSB
0x12     0x34     0x56     0x78
```

MSB는 Most Significant Byte, 가장 큰 자리 byte다.
LSB는 Least Significant Byte, 가장 작은 자리 byte다.

---

## 5.2 Big-endian memory

```txt
address   value
0x1000    0x12
0x1001    0x34
0x1002    0x56
0x1003    0x78
```

그림:

```txt
낮은 주소                              높은 주소
+--------+--------+--------+--------+
|  0x12  |  0x34  |  0x56  |  0x78  |
+--------+--------+--------+--------+
```

---

## 5.3 Little-endian memory

```txt
address   value
0x1000    0x78
0x1001    0x56
0x1002    0x34
0x1003    0x12
```

그림:

```txt
낮은 주소                              높은 주소
+--------+--------+--------+--------+
|  0x78  |  0x56  |  0x34  |  0x12  |
+--------+--------+--------+--------+
```

---

## 5.4 중요한 구분: value operation vs memory representation

다음 코드는 endian과 무관하게 같은 결과를 낸다.

```c
uint32_t x = 0x12345678;

uint8_t b0 = (uint8_t)(x & 0xFFu);
uint8_t b1 = (uint8_t)((x >> 8) & 0xFFu);
uint8_t b2 = (uint8_t)((x >> 16) & 0xFFu);
uint8_t b3 = (uint8_t)((x >> 24) & 0xFFu);
```

결과:

```txt
b0 = 0x78
b1 = 0x56
b2 = 0x34
b3 = 0x12
```

왜냐하면 이건 메모리를 읽는 것이 아니라 **정수 값에 대한 shift/mask 연산**이기 때문이다.

반면 다음은 실제 메모리 byte를 읽는다.

```c
uint32_t x = 0x12345678;
unsigned char *p = (unsigned char *)&x;
```

`p[0]`은 시스템 endian에 따라 다르다.

```txt
big-endian    : p[0] = 0x12
little-endian : p[0] = 0x78
```

이 차이를 반드시 구분해야 한다.

---

# 6. 단계별 예시

## 예시 1: 메모리 byte 확인하기

C에서는 객체의 byte representation을 `unsigned char *`로 읽는 것이 허용된다.

```c
#include <stdint.h>
#include <stdio.h>

int main(void) {
    uint32_t x = UINT32_C(0x12345678);
    unsigned char *p = (unsigned char *)&x;

    for (size_t i = 0; i < sizeof x; ++i) {
        printf("p[%zu] = 0x%02X\n", i, p[i]);
    }

    return 0;
}
```

Little-endian 시스템이면 보통:

```txt
p[0] = 0x78
p[1] = 0x56
p[2] = 0x34
p[3] = 0x12
```

Big-endian 시스템이면:

```txt
p[0] = 0x12
p[1] = 0x34
p[2] = 0x56
p[3] = 0x78
```

주의:

이 코드는 endian 확인용이다.
파일이나 네트워크 포맷을 처리할 때 이런 식으로 메모리를 그대로 덤프하는 방식에 의존하면 portability가 떨어질 수 있다.

---

## 예시 2: runtime endian check

```c
#include <stdint.h>
#include <stdbool.h>

bool is_little_endian(void) {
    uint32_t x = 1;
    unsigned char *p = (unsigned char *)&x;

    return p[0] == 1;
}
```

설명:

```txt
x = 0x00000001
```

Little-endian memory:

```txt
01 00 00 00
```

Big-endian memory:

```txt
00 00 00 01
```

따라서 `p[0] == 1`이면 little-endian이다.

---

## 예시 3: big-endian byte array를 `uint32_t`로 조립

네트워크나 파일에서 다음 byte가 들어왔다고 하자.

```txt
buf[0] = 0x12
buf[1] = 0x34
buf[2] = 0x56
buf[3] = 0x78
```

이것이 big-endian 32-bit integer라면 값은:

```txt
0x12345678
```

portable하게 조립:

```c
#include <stdint.h>

uint32_t read_u32_be(const uint8_t buf[4]) {
    return ((uint32_t)buf[0] << 24) |
           ((uint32_t)buf[1] << 16) |
           ((uint32_t)buf[2] << 8)  |
           ((uint32_t)buf[3]);
}
```

핵심:

```txt
buf[0]은 가장 큰 자리 byte
buf[3]은 가장 작은 자리 byte
```

이 코드는 CPU가 little-endian이든 big-endian이든 같은 값을 만든다.

---

## 예시 4: little-endian byte array를 `uint32_t`로 조립

```txt
buf[0] = 0x78
buf[1] = 0x56
buf[2] = 0x34
buf[3] = 0x12
```

이것이 little-endian 32-bit integer라면 값은:

```txt
0x12345678
```

portable하게 조립:

```c
#include <stdint.h>

uint32_t read_u32_le(const uint8_t buf[4]) {
    return ((uint32_t)buf[0])       |
           ((uint32_t)buf[1] << 8)  |
           ((uint32_t)buf[2] << 16) |
           ((uint32_t)buf[3] << 24);
}
```

핵심:

```txt
buf[0]은 가장 작은 자리 byte
buf[3]은 가장 큰 자리 byte
```

---

## 예시 5: `uint32_t`를 big-endian byte array로 쓰기

```c
#include <stdint.h>

void write_u32_be(uint8_t buf[4], uint32_t x) {
    buf[0] = (uint8_t)((x >> 24) & 0xFFu);
    buf[1] = (uint8_t)((x >> 16) & 0xFFu);
    buf[2] = (uint8_t)((x >> 8) & 0xFFu);
    buf[3] = (uint8_t)(x & 0xFFu);
}
```

예:

```txt
x = 0x12345678

buf[0] = 0x12
buf[1] = 0x34
buf[2] = 0x56
buf[3] = 0x78
```

---

## 예시 6: `uint32_t`를 little-endian byte array로 쓰기

```c
#include <stdint.h>

void write_u32_le(uint8_t buf[4], uint32_t x) {
    buf[0] = (uint8_t)(x & 0xFFu);
    buf[1] = (uint8_t)((x >> 8) & 0xFFu);
    buf[2] = (uint8_t)((x >> 16) & 0xFFu);
    buf[3] = (uint8_t)((x >> 24) & 0xFFu);
}
```

예:

```txt
x = 0x12345678

buf[0] = 0x78
buf[1] = 0x56
buf[2] = 0x34
buf[3] = 0x12
```

---

# 7. Byte swap

## 7.1 Byte swap이란?

Byte swap은 byte 순서를 뒤집는 연산이다.

```txt
0x12345678
→ 0x78563412
```

이 연산은 endian conversion에서 자주 등장한다.

---

## 7.2 직접 구현

```c
#include <stdint.h>

uint32_t bswap32_manual(uint32_t x) {
    return ((x & UINT32_C(0x000000FF)) << 24) |
           ((x & UINT32_C(0x0000FF00)) << 8)  |
           ((x & UINT32_C(0x00FF0000)) >> 8)  |
           ((x & UINT32_C(0xFF000000)) >> 24);
}
```

분해:

```txt
x = 0x12345678

0x00000078 << 24 = 0x78000000
0x00005600 <<  8 = 0x00560000
0x00340000 >>  8 = 0x00003400
0x12000000 >> 24 = 0x00000012

OR result = 0x78563412
```

---

## 7.3 `__builtin_bswap32`

GCC/Clang에서는 다음 builtin을 제공한다.

```c
uint32_t y = __builtin_bswap32(x);
```

예:

```c
#include <stdint.h>
#include <stdio.h>
#include <inttypes.h>

int main(void) {
    uint32_t x = UINT32_C(0x12345678);
    uint32_t y = __builtin_bswap32(x);

    printf("0x%08" PRIX32 "\n", y);

    return 0;
}
```

출력:

```txt
0x78563412
```

주의:

* `__builtin_bswap32`는 ISO C 표준 함수가 아니다.
* GCC/Clang extension이다.
* MSVC는 `_byteswap_ulong` 같은 intrinsic을 사용한다.
* portable code에서는 wrapper를 두는 것이 좋다.

예:

```c
#include <stdint.h>

static inline uint32_t bswap32(uint32_t x) {
#if defined(__GNUC__) || defined(__clang__)
    return __builtin_bswap32(x);
#else
    return ((x & UINT32_C(0x000000FF)) << 24) |
           ((x & UINT32_C(0x0000FF00)) << 8)  |
           ((x & UINT32_C(0x00FF0000)) >> 8)  |
           ((x & UINT32_C(0xFF000000)) >> 24);
#endif
}
```

---

# 8. 실제 응용

## 8.1 네트워크 packet parsing

예를 들어 network packet에 32-bit length field가 big-endian으로 들어 있다고 하자.

```txt
buf[0] buf[1] buf[2] buf[3]
00     00     04     00
```

이 값은:

```txt
0x00000400 = 1024
```

portable parsing:

```c
#include <stdint.h>

uint32_t parse_length_be(const uint8_t *buf) {
    return ((uint32_t)buf[0] << 24) |
           ((uint32_t)buf[1] << 16) |
           ((uint32_t)buf[2] << 8)  |
           ((uint32_t)buf[3]);
}
```

잘못된 방식:

```c
uint32_t length = *(uint32_t *)buf;
```

이 방식은 문제가 많다.

```txt
1. CPU endian에 의존
2. alignment 문제 가능
3. strict aliasing 문제 가능
4. buffer size 검증 누락 가능
```

네트워크나 파일 포맷에서는 byte 단위로 명시적으로 조립하는 방식이 안전하다.

---

## 8.2 파일 포맷 parsing

파일 포맷은 endian 규칙을 명시한다.

예를 들어 어떤 custom binary file이 다음 규칙을 가진다고 하자.

```txt
magic: 4 bytes ASCII
version: uint16 little-endian
payload_size: uint32 little-endian
```

그러면 version과 payload_size는 little-endian으로 읽어야 한다.

```c
#include <stdint.h>

uint16_t read_u16_le(const uint8_t buf[2]) {
    return ((uint16_t)buf[0]) |
           ((uint16_t)buf[1] << 8);
}

uint32_t read_u32_le(const uint8_t buf[4]) {
    return ((uint32_t)buf[0])       |
           ((uint32_t)buf[1] << 8)  |
           ((uint32_t)buf[2] << 16) |
           ((uint32_t)buf[3] << 24);
}
```

중요한 원칙:

> 파일 포맷을 읽을 때는 host endian을 믿지 말고, 포맷이 정의한 endian으로 조립해야 한다.

---

## 8.3 Embedded sensor data

I2C/SPI 센서는 high byte, low byte를 따로 주는 경우가 많다.

예:

```txt
TEMP_H = 0xFF
TEMP_L = 0xEC
```

센서 문서에 “16-bit signed big-endian two’s complement”라고 되어 있으면:

```c
#include <stdint.h>

int32_t parse_temp(uint8_t high, uint8_t low) {
    uint16_t raw = ((uint16_t)high << 8) | (uint16_t)low;

    if (raw < 0x8000u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x10000;
}
```

결과:

```txt
0xFFEC = -20
```

여기서는 두 가지 개념이 같이 나온다.

```txt
1. Endianness: high byte가 먼저인가, low byte가 먼저인가?
2. Two's complement: raw 16-bit를 signed로 어떻게 해석하는가?
```

---

## 8.4 Embedded memory-mapped register

MCU peripheral register는 보통 다음처럼 접근한다.

```c
#define REG_ADDR ((volatile uint32_t *)0x40000000u)

uint32_t value = *REG_ADDR;
```

여기서 endian은 CPU와 bus, peripheral 설계에 따라 영향을 받을 수 있다.

일반적으로 32-bit register를 32-bit access로 읽는다면 프로그래머는 보통 register value를 정상적인 정수로 본다.

```c
uint32_t value = *REG_ADDR;
```

그리고 bit mask로 처리한다.

```c
if ((value & READY_MASK) != 0u) {
    // ready
}
```

하지만 byte 단위로 register를 접근하거나, DMA buffer를 다른 장치와 공유하거나, 외부 bus protocol과 연결되면 byte order를 명확히 확인해야 한다.

Embedded에서 특히 확인할 것:

```txt
1. CPU endian
2. peripheral register access width
3. bus endian conversion 여부
4. DMA buffer byte order
5. sensor/protocol 문서의 byte order
```

---

## 8.5 Debugging memory dump

디버거에서 메모리가 이렇게 보인다고 하자.

```txt
address:
0x1000: 78
0x1001: 56
0x1002: 34
0x1003: 12
```

Little-endian 시스템에서 이 4 bytes를 `uint32_t`로 읽으면:

```txt
0x12345678
```

Big-endian 시스템에서 읽으면:

```txt
0x78563412
```

따라서 memory dump를 볼 때는 반드시 target endian을 알아야 한다.

---

# 9. 성능과 메모리 관점

## 9.1 Endian conversion 비용

Endian conversion은 보통 byte swap instruction이나 shift/mask로 처리된다.

예:

```c
x = bswap32(x);
```

현대 CPU에서는 byte swap 전용 instruction이 있는 경우가 많다.

```txt
x86: bswap
ARM: rev 계열 instruction
```

컴파일러 builtin을 쓰면 이런 instruction으로 최적화될 가능성이 높다.

```c
__builtin_bswap32(x)
```

하지만 embedded MCU에서는 상황이 다를 수 있다.

| 환경                | 고려점                                 |
| ----------------- | ----------------------------------- |
| 고성능 CPU           | byte swap instruction 존재 가능         |
| 작은 MCU            | 여러 shift/mask instruction으로 풀릴 수 있음 |
| DMA/network stack | buffer 전체 conversion 비용 주의          |
| protocol parsing  | 필요한 field만 변환하는 편이 나을 수 있음          |

---

## 9.2 구조체를 그대로 전송하면 위험한 이유

다음 구조체가 있다고 하자.

```c
#include <stdint.h>

typedef struct {
    uint16_t type;
    uint32_t length;
} Header;
```

이걸 그대로 파일에 쓰거나 네트워크로 보내는 것은 위험하다.

```c
Header h = {1, 1024};
write(fd, &h, sizeof h);
```

문제:

```txt
1. endian이 host에 따라 달라짐
2. padding byte가 들어갈 수 있음
3. alignment가 platform마다 다를 수 있음
4. struct layout이 compiler/ABI에 의존할 수 있음
```

더 안전한 방식:

```c
uint8_t buf[6];

write_u16_be(&buf[0], type);
write_u32_be(&buf[2], length);
```

즉, wire format은 byte array로 명확히 작성해야 한다.

---

## 9.3 Readability vs optimization

다음 코드는 빠를 수 있지만 의도가 불명확하다.

```c
x = ((x & 0xFF) << 24) | ...;
```

실무에서는 wrapper 함수로 의도를 드러내는 것이 좋다.

```c
uint32_t network_length = read_u32_be(buf);
```

또는:

```c
uint32_t swapped = bswap32(x);
```

핵심:

> endian conversion은 성능보다 정확성과 명확성이 먼저다.
> 필요한 경우에만 benchmark를 보고 최적화한다.

---

# 10. 흔한 오해

## 오해 1: Little-endian이면 bit 순서도 반대다

아니다.

Endianness는 byte order다.

```txt
0x12345678의 byte 순서:
12 34 56 78 또는 78 56 34 12
```

각 byte 안의 bit numbering을 뒤집는 문제가 아니다.

예를 들어 byte `0x12`는 항상 값으로:

```txt
00010010
```

이다.

물론 통신 프로토콜에서는 bit transmission order라는 별도 문제가 있을 수 있다.
하지만 CPU endian과는 다른 개념이다.

---

## 오해 2: `x & 0xFF` 결과는 endian에 따라 달라진다

아니다.

```c
uint32_t x = 0x12345678;
uint32_t low = x & 0xFFu;
```

결과는 항상:

```txt
0x78
```

Endian과 무관하다.

왜냐하면 이건 메모리 접근이 아니라 정수 값에 대한 bit operation이기 때문이다.

Endian이 영향을 주는 것은 다음처럼 메모리 byte를 직접 볼 때다.

```c
unsigned char *p = (unsigned char *)&x;
p[0]
```

---

## 오해 3: `memcpy`로 읽으면 endian 문제가 사라진다

아니다.

`memcpy`는 alignment와 aliasing 문제를 피하는 데 도움이 된다.
하지만 byte order 문제는 해결하지 않는다.

예:

```c
uint32_t x;
memcpy(&x, buf, sizeof x);
```

이 코드는 unaligned access나 strict aliasing 문제는 줄일 수 있다.
하지만 `buf`의 byte order가 host endian과 다르면 값은 잘못 해석된다.

따라서 binary format parsing에서는 보통:

```c
x = read_u32_be(buf);
```

처럼 명시적으로 조립한다.

---

## 오해 4: x86만 생각하면 endian은 몰라도 된다

개인 PC 프로그램만 작성한다면 endian 문제를 자주 만나지 않을 수 있다.
하지만 다음 영역에서는 반드시 등장한다.

```txt
network
file format
embedded sensor
firmware
cross-platform serialization
binary protocol
debugging memory dump
```

Embedded와 systems programming에서는 endian을 모르면 raw data parsing을 신뢰할 수 없다.

---

## 오해 5: ARM은 항상 little-endian이다

대부분의 현대 일반 환경에서 ARM은 little-endian으로 사용되는 경우가 많다.
하지만 ARM 아키텍처 자체는 endian configuration과 관련된 다양한 모드를 지원해온 역사가 있다.

실무적으로는 이렇게 접근해야 한다.

```txt
x86/x86-64: 일반적으로 little-endian
ARM Cortex-M: 일반적으로 little-endian
ARM application processors: 일반적으로 little-endian 환경이 많음
Network byte order: big-endian
```

하지만 최종 판단은 target architecture, ABI, SoC manual, compiler configuration, protocol specification을 확인해야 한다.

---

# 11. 반례 또는 실패 사례

## 실패 사례 1: network packet을 pointer cast로 읽기

위험한 코드:

```c
uint32_t length = *(uint32_t *)&buf[0];
```

문제:

```txt
1. little-endian host에서는 big-endian packet을 잘못 읽음
2. buf가 4-byte aligned가 아닐 수 있음
3. strict aliasing 문제가 생길 수 있음
4. buffer 길이 검증이 빠져 있을 수 있음
```

안전한 코드:

```c
uint32_t length = read_u32_be(buf);
```

---

## 실패 사례 2: struct를 그대로 저장

위험한 코드:

```c
typedef struct {
    uint16_t id;
    uint32_t size;
} Record;

fwrite(&record, sizeof record, 1, fp);
```

문제:

```txt
1. endian이 저장됨
2. padding이 저장될 수 있음
3. 다른 compiler에서 layout이 다를 수 있음
4. 버전 관리가 어려움
```

안전한 방식:

```c
uint8_t buf[6];

write_u16_le(&buf[0], record.id);
write_u32_le(&buf[2], record.size);

fwrite(buf, sizeof buf, 1, fp);
```

---

## 실패 사례 3: 센서 byte 순서 반대로 조립

센서가 big-endian으로 보낸다.

```txt
high = 0x01
low  = 0x00
```

올바른 값:

```txt
0x0100 = 256
```

잘못된 코드:

```c
uint16_t value = ((uint16_t)low << 8) | high;
```

결과:

```txt
0x0001 = 1
```

high/low byte 순서를 착각하면 값이 완전히 달라진다.

---

## 실패 사례 4: byte swap을 두 번 함

데이터가 이미 host endian으로 변환되었는데 또 swap하는 경우가 있다.

```c
uint32_t x = ntohl(network_value);
x = bswap32(x); // 잘못된 중복 변환일 수 있음
```

Endian conversion은 데이터의 현재 상태를 정확히 알아야 한다.

```txt
wire endian → host endian
host endian → wire endian
```

이 흐름을 명확히 이름으로 표현하는 것이 좋다.

```c
uint32_t host_len = read_u32_be(buf);
write_u32_be(out, host_len);
```

---

# 12. Portability-safe coding style

## 12.1 명시적 read/write 함수 사용

```c
#include <stdint.h>

uint16_t read_u16_be(const uint8_t buf[2]) {
    return ((uint16_t)buf[0] << 8) |
           ((uint16_t)buf[1]);
}

uint16_t read_u16_le(const uint8_t buf[2]) {
    return ((uint16_t)buf[0]) |
           ((uint16_t)buf[1] << 8);
}

void write_u16_be(uint8_t buf[2], uint16_t x) {
    buf[0] = (uint8_t)((x >> 8) & 0xFFu);
    buf[1] = (uint8_t)(x & 0xFFu);
}

void write_u16_le(uint8_t buf[2], uint16_t x) {
    buf[0] = (uint8_t)(x & 0xFFu);
    buf[1] = (uint8_t)((x >> 8) & 0xFFu);
}
```

32-bit:

```c
#include <stdint.h>

uint32_t read_u32_be(const uint8_t buf[4]) {
    return ((uint32_t)buf[0] << 24) |
           ((uint32_t)buf[1] << 16) |
           ((uint32_t)buf[2] << 8)  |
           ((uint32_t)buf[3]);
}

uint32_t read_u32_le(const uint8_t buf[4]) {
    return ((uint32_t)buf[0])       |
           ((uint32_t)buf[1] << 8)  |
           ((uint32_t)buf[2] << 16) |
           ((uint32_t)buf[3] << 24);
}
```

---

## 12.2 pointer cast 대신 byte 조립

피해야 할 스타일:

```c
uint32_t x = *(uint32_t *)buf;
```

권장:

```c
uint32_t x = read_u32_be(buf);
```

또는 포맷이 little-endian이면:

```c
uint32_t x = read_u32_le(buf);
```

이 방식은 endian이 코드에 명시되어 있다.

---

## 12.3 host endian을 코드 곳곳에 흩뿌리지 말기

좋은 함수 이름:

```c
read_u32_be
read_u32_le
write_u32_be
write_u32_le
host_to_be32
be32_to_host
```

나쁜 코드:

```c
if (is_little_endian()) {
    x = bswap32(x);
}
```

이런 코드가 여기저기 흩어지면 실수하기 쉽다.

차라리 wrapper를 둔다.

```c
static inline uint32_t be32_to_host(uint32_t x) {
#if defined(MY_TARGET_LITTLE_ENDIAN)
    return bswap32(x);
#else
    return x;
#endif
}
```

단, 실제 프로젝트에서는 compiler predefined macro나 build system에서 target endian을 정해주는 경우가 많다.

---

## 12.4 signed value는 unsigned로 조립 후 decode

센서나 파일에서 signed 16-bit big-endian 값을 읽는 경우:

```c
#include <stdint.h>

int32_t read_i16_be_as_i32(const uint8_t buf[2]) {
    uint16_t raw = read_u16_be(buf);

    if (raw < 0x8000u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x10000;
}
```

이 방식은 다음을 분리한다.

```txt
1. endian 처리: bytes → uint16_t raw
2. signed 해석: uint16_t raw → int32_t value
```

이렇게 분리하면 디버깅이 쉬워진다.

---

# 13. 확인 문제

## 문제 1

`uint32_t x = 0x12345678;`

메모리 주소가 낮은 곳부터 높은 곳으로 갈 때, 다음을 써라.

```txt
Big-endian    : ?
Little-endian : ?
```

---

## 문제 2

다음 memory dump가 있다.

```txt
0x1000: 78
0x1001: 56
0x1002: 34
0x1003: 12
```

이 4 bytes를 little-endian `uint32_t`로 읽으면 값은?

Big-endian `uint32_t`로 읽으면 값은?

---

## 문제 3

다음 두 코드는 endian 영향을 받는가?

```c
uint32_t x = 0x12345678;
uint8_t a = (uint8_t)(x & 0xFFu);
```

```c
uint32_t x = 0x12345678;
unsigned char *p = (unsigned char *)&x;
uint8_t b = p[0];
```

각각 설명해라.

---

## 문제 4

다음 코드는 왜 위험한가?

```c
uint32_t length = *(uint32_t *)buf;
```

---

## 문제 5

`buf = {0x12, 0x34, 0x56, 0x78}`일 때, big-endian으로 읽은 값과 little-endian으로 읽은 값을 각각 구해라.

---

## 문제 6

Network byte order는 일반적으로 big-endian인가 little-endian인가?

---

## 문제 7

Endianness와 bit numbering은 같은 개념인가?

정확히 구분해서 설명해라.

---

# 14. 실습 과제

## 실습 1: endian 확인 함수 작성

다음 함수를 구현해라.

```c
#include <stdbool.h>

bool is_little_endian(void);
```

예상 구현:

```c
#include <stdint.h>
#include <stdbool.h>

bool is_little_endian(void) {
    uint32_t x = 1;
    unsigned char *p = (unsigned char *)&x;

    return p[0] == 1;
}
```

---

## 실습 2: `read_u32_be`, `read_u32_le` 구현

```c
#include <stdint.h>

uint32_t read_u32_be(const uint8_t buf[4]);
uint32_t read_u32_le(const uint8_t buf[4]);
```

예상 구현:

```c
#include <stdint.h>

uint32_t read_u32_be(const uint8_t buf[4]) {
    return ((uint32_t)buf[0] << 24) |
           ((uint32_t)buf[1] << 16) |
           ((uint32_t)buf[2] << 8)  |
           ((uint32_t)buf[3]);
}

uint32_t read_u32_le(const uint8_t buf[4]) {
    return ((uint32_t)buf[0])       |
           ((uint32_t)buf[1] << 8)  |
           ((uint32_t)buf[2] << 16) |
           ((uint32_t)buf[3] << 24);
}
```

테스트:

```txt
buf = {0x12, 0x34, 0x56, 0x78}

read_u32_be(buf) = 0x12345678
read_u32_le(buf) = 0x78563412
```

---

## 실습 3: `write_u32_be`, `write_u32_le` 구현

```c
#include <stdint.h>

void write_u32_be(uint8_t buf[4], uint32_t x);
void write_u32_le(uint8_t buf[4], uint32_t x);
```

예상 구현:

```c
#include <stdint.h>

void write_u32_be(uint8_t buf[4], uint32_t x) {
    buf[0] = (uint8_t)((x >> 24) & 0xFFu);
    buf[1] = (uint8_t)((x >> 16) & 0xFFu);
    buf[2] = (uint8_t)((x >> 8) & 0xFFu);
    buf[3] = (uint8_t)(x & 0xFFu);
}

void write_u32_le(uint8_t buf[4], uint32_t x) {
    buf[0] = (uint8_t)(x & 0xFFu);
    buf[1] = (uint8_t)((x >> 8) & 0xFFu);
    buf[2] = (uint8_t)((x >> 16) & 0xFFu);
    buf[3] = (uint8_t)((x >> 24) & 0xFFu);
}
```

테스트:

```txt
x = 0x12345678

write_u32_be → 12 34 56 78
write_u32_le → 78 56 34 12
```

---

## 실습 4: `bswap32` 구현

```c
#include <stdint.h>

uint32_t bswap32_manual(uint32_t x);
```

예상 구현:

```c
#include <stdint.h>

uint32_t bswap32_manual(uint32_t x) {
    return ((x & UINT32_C(0x000000FF)) << 24) |
           ((x & UINT32_C(0x0000FF00)) << 8)  |
           ((x & UINT32_C(0x00FF0000)) >> 8)  |
           ((x & UINT32_C(0xFF000000)) >> 24);
}
```

테스트:

```txt
0x12345678 → 0x78563412
0xAABBCCDD → 0xDDCCBBAA
```

---

## 실습 5: signed 16-bit big-endian sensor 값 읽기

센서 byte 2개를 받아 signed 16-bit two’s complement 값으로 해석해라.

```c
#include <stdint.h>

int32_t read_i16_be_sensor(const uint8_t buf[2]);
```

예상 구현:

```c
#include <stdint.h>

uint16_t read_u16_be(const uint8_t buf[2]) {
    return ((uint16_t)buf[0] << 8) |
           ((uint16_t)buf[1]);
}

int32_t read_i16_be_sensor(const uint8_t buf[2]) {
    uint16_t raw = read_u16_be(buf);

    if (raw < 0x8000u) {
        return (int32_t)raw;
    }

    return (int32_t)raw - 0x10000;
}
```

테스트:

```txt
{0x00, 0x14} → 20
{0xFF, 0xEC} → -20
{0x7F, 0xFF} → 32767
{0x80, 0x00} → -32768
```

---

# 15. 핵심 정리

이번 강의 핵심은 다음이다.

1. **Endianness는 multi-byte data의 byte order 문제다.**
2. **Big-endian은 MSB를 낮은 주소에 저장한다.**
3. **Little-endian은 LSB를 낮은 주소에 저장한다.**
4. **`0x12345678`은 big-endian memory에서 `12 34 56 78`, little-endian memory에서 `78 56 34 12`로 보인다.**
5. **Endian은 byte order 문제이지 bit order 문제가 아니다.**
6. **정수 값에 대한 shift/mask 연산은 endian과 무관하다.**
7. **메모리 byte를 직접 읽으면 endian의 영향을 받는다.**
8. **Network byte order는 일반적으로 big-endian이다.**
9. **파일, 네트워크, 센서, binary protocol에서는 포맷이 정의한 endian으로 읽고 써야 한다.**
10. **`*(uint32_t *)buf` 같은 pointer cast parsing은 endian, alignment, aliasing 문제 때문에 위험하다.**
11. **Portable한 코드는 `read_u32_be`, `read_u32_le`처럼 byte 단위로 명시적으로 조립한다.**
12. **`__builtin_bswap32`는 유용하지만 compiler-specific이므로 wrapper를 두는 것이 좋다.**
13. **Embedded에서는 CPU endian, peripheral register access width, DMA buffer, sensor protocol byte order를 구분해야 한다.**
14. **Signed sensor 값은 먼저 endian에 맞게 unsigned raw value로 조립한 뒤 two’s complement로 해석하는 것이 안전하다.**

---

# 16. 면접 대비 핵심 문장

면접에서는 이렇게 말하면 된다.

> Endianness describes the byte order used to store multi-byte values in memory. In big-endian, the most significant byte is stored at the lowest address. In little-endian, the least significant byte is stored at the lowest address.

한국어로는:

> Endianness는 multi-byte 값을 메모리에 저장할 때 byte를 어떤 순서로 배치하는지를 의미합니다. Big-endian은 가장 큰 자리 byte를 낮은 주소에 저장하고, little-endian은 가장 작은 자리 byte를 낮은 주소에 저장합니다.

`0x12345678` 예시는 이렇게 설명하면 된다.

> For `0x12345678`, big-endian memory order is `12 34 56 78`, while little-endian memory order is `78 56 34 12`.

한국어로는:

> `0x12345678`은 big-endian 메모리에서는 `12 34 56 78` 순서로 저장되고, little-endian 메모리에서는 `78 56 34 12` 순서로 저장됩니다.

실무 안전성까지 포함하면:

> When parsing files or network packets, do not cast a byte buffer directly to an integer pointer. Instead, explicitly assemble the integer using shifts and masks according to the format’s byte order.

한국어로는:

> 파일이나 네트워크 packet을 parsing할 때 byte buffer를 `uint32_t *`로 cast해서 읽으면 endian, alignment, aliasing 문제가 생길 수 있습니다. 포맷이 정의한 byte order에 따라 shift와 mask로 명시적으로 조립하는 방식이 안전합니다.

---