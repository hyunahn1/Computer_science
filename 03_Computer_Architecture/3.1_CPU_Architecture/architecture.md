# Lecture 2. Von Neumann Architecture와 Harvard Architecture

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> CPU가 명령어와 데이터를 모두 메모리에서 읽는다면, 두 종류의 정보는 같은 경로를 사용해야 하는가?

그리고 다음 질문도 함께 다룹니다.

1. 명령어와 데이터는 본질적으로 다른 비트인가?
2. Von Neumann Architecture에서는 왜 명령어와 데이터가 충돌할 수 있는가?
3. Harvard Architecture는 이 문제를 어떻게 해결하는가?
4. 현대 x86·ARM 프로세서는 Von Neumann인가, Harvard인가?
5. Cortex-M microcontroller에서 Flash와 SRAM은 CPU에 어떻게 연결되는가?
6. 프로그램 코드가 데이터를 포함하거나, 데이터를 코드처럼 실행할 수 있는 이유는 무엇인가?

Lecture 1에서는 CPU가 Program Counter가 가리키는 주소에서 명령어를 가져오고, ISA 규칙에 따라 실행한다는 것을 배웠습니다. 이번 Lecture에서는 **명령어와 데이터가 CPU까지 어떤 경로로 이동하는가**를 살펴봅니다. 

---

## 2. 이전 Lecture와의 연결

Lecture 1에서 가장 단순한 CPU 모델을 다음과 같이 표현했습니다.

```text
PC
 |
 v
Memory ----> Instruction Decoder
                  |
                  v
             Registers
                  |
                  v
                 ALU
```

하지만 이 그림에는 중요한 문제가 숨겨져 있습니다.

CPU는 명령어만 메모리에서 읽는 것이 아닙니다.

다음 C 코드를 생각해 보겠습니다.

```c
int result = array[index] + 10;
```

CPU는 최소한 다음 두 가지를 읽어야 합니다.

```text
1. 실행할 machine instruction
2. array[index]에 저장된 data
```

따라서 질문이 생깁니다.

```text
Instruction fetch와 data load가 같은 memory interface를 사용하는가?

아니면 서로 독립된 memory interface를 사용하는가?
```

이 차이가 Von Neumann Architecture와 Harvard Architecture를 구분하는 출발점입니다.

---

## 3. 직관적 설명

### 3.1 하나의 도로를 공유하는 구조

도시와 공장 사이에 도로가 하나뿐이라고 생각해 봅시다.

공장은 두 종류의 차량을 받아야 합니다.

```text
작업 지시서를 운반하는 차량 → Instruction fetch
원재료를 운반하는 차량       → Data load/store
```

도로가 하나라면 두 차량은 같은 길을 사용합니다.

```text
                하나의 도로
공장 <----------------------------> 창고
      작업 지시서와 원재료가 공유
```

작업 지시서 차량이 도로를 사용하고 있다면 원재료 차량은 기다려야 할 수 있습니다.

이것이 전통적인 Von Neumann 구조의 직관입니다.

---

### 3.2 두 개의 도로를 사용하는 구조

이번에는 작업 지시서와 원재료를 위한 도로가 따로 있다고 가정하겠습니다.

```text
작업 지시서 도로
공장 <----------------------------> 명령어 창고

원재료 도로
공장 <----------------------------> 데이터 창고
```

이제 두 작업을 동시에 수행할 수 있습니다.

```text
Instruction fetch와 data load가 같은 cycle에 진행 가능
```

이것이 Harvard Architecture의 핵심 직관입니다.

---

### 3.3 비유와 실제 하드웨어의 대응

| 비유      | 실제 하드웨어                                   |
| ------- | ----------------------------------------- |
| 공장      | CPU core                                  |
| 작업 지시서  | Machine instruction                       |
| 원재료     | Program data                              |
| 창고      | Memory                                    |
| 도로      | Bus 또는 memory access path                 |
| 차량 충돌   | Structural hazard 또는 bandwidth contention |
| 두 개의 도로 | Separate instruction/data buses           |

---

### 3.4 비유의 한계

현대 프로세서는 단순히 “도로가 하나냐 둘이냐”로 구분하기 어렵습니다.

예를 들어 현대 CPU는 흔히 다음과 같은 구조를 사용합니다.

```text
                 +----------------+
                 |    CPU Core    |
                 +----------------+
                   |            |
             Instruction      Data
                 path          path
                   |            |
              +--------+    +--------+
              | L1 I$  |    | L1 D$  |
              +--------+    +--------+
                   \            /
                    \          /
                     +--------+
                     |  L2    |
                     +--------+
                         |
                     Main Memory
```

CPU 가까이에서는 instruction cache와 data cache가 분리되어 있지만, 아래 단계에서는 하나의 memory system을 공유합니다.

이런 구조를 흔히 **Modified Harvard Architecture**라고 부릅니다.

---

## 4. 형식적 정의

### 4.1 Von Neumann Architecture

**용어:** Von Neumann Architecture

**정의:**
명령어와 데이터를 동일한 주소 공간과 동일한 메모리 시스템에 저장하는 컴퓨터 구조입니다.

**필요한 이유:**
하나의 메모리 시스템으로 프로그램 코드와 데이터를 모두 저장하면 구조가 단순하고 유연해집니다.

**하드웨어에서의 역할:**
CPU는 동일한 memory path 또는 공유된 memory hierarchy를 통해 instruction과 data에 접근합니다.

**관련된 다른 개념:**

```text
Stored-program computer
Unified memory
Von Neumann bottleneck
Self-modifying code
Unified address space
```

가장 단순한 구조는 다음과 같습니다.

```text
                       Address bus
              +----------------------------+
              |                            |
              v                            |
        +-----------+               +-------------+
        |    CPU    |<------------->|   Memory    |
        +-----------+    Data bus   +-------------+
                                  Instructions
                                  + Data
```

---

### 4.2 Harvard Architecture

**용어:** Harvard Architecture

**정의:**
명령어와 데이터를 별도의 메모리, 주소 공간 또는 접근 경로에 저장하는 구조입니다.

**필요한 이유:**
Instruction fetch와 data access를 독립적으로 수행하여 처리량을 높이고, 서로 다른 memory technology나 width를 사용할 수 있게 합니다.

**하드웨어에서의 역할:**
CPU는 instruction memory와 data memory에 각각 별도의 bus 또는 port로 접근합니다.

**관련된 다른 개념:**

```text
Separate address spaces
Instruction memory
Data memory
Program memory
DSP
Microcontroller
Modified Harvard Architecture
```

```text
             +-------------+
             |     CPU     |
             +-------------+
               |         |
       Instruction bus   Data bus
               |         |
               v         v
        +----------+  +----------+
        | Program  |  |   Data   |
        | Memory   |  | Memory   |
        +----------+  +----------+
```

---

### 4.3 Stored-Program Concept

**용어:** 저장 프로그램 개념(Stored-Program Concept)

**정의:**
프로그램의 명령어를 고정된 배선이 아니라 메모리에 저장하고, CPU가 메모리에서 읽어 실행하는 방식입니다.

**필요한 이유:**
프로그램을 변경할 때 하드웨어 회로를 다시 구성할 필요가 없도록 하기 위해서입니다.

**하드웨어에서의 역할:**
Program Counter가 메모리 주소를 제공하고, 해당 위치의 instruction을 fetch합니다.

**관련된 다른 개념:**

```text
Executable
Loader
Bootloader
Firmware
Code segment
```

Von Neumann Architecture에서 특히 강조되지만, Harvard Architecture도 프로그램을 program memory에 저장한다는 점에서는 stored-program computer가 될 수 있습니다.

따라서 다음 두 개념은 완전히 같은 뜻이 아닙니다.

```text
Stored-program computer
≠
반드시 Von Neumann Architecture
```

---

### 4.4 Address Space

**용어:** 주소 공간(Address Space)

**정의:**
프로세서 또는 프로그램이 주소를 사용하여 접근할 수 있는 위치들의 집합입니다.

**필요한 이유:**
명령어와 데이터가 저장된 위치를 식별하기 위해 필요합니다.

**하드웨어에서의 역할:**
주소가 memory decoder, cache, MMU, bus interconnect에 전달되어 실제 대상 장치를 선택합니다.

**관련된 다른 개념:**

```text
Virtual address
Physical address
Memory map
MMIO
Code address
Data address
```

Von Neumann 구조에서는 명령어와 데이터가 일반적으로 하나의 주소 공간을 공유합니다.

```text
0x0000 ─┐
        │ Code
0x3FFF ─┤
        │ Data
0x7FFF ─┤
        │ Stack
0xFFFF ─┘
```

Harvard 구조에서는 같은 숫자 주소가 program memory와 data memory에서 각각 존재할 수 있습니다.

```text
Program address 0x1000
Data address    0x1000
```

두 주소는 숫자가 같더라도 서로 다른 물리적 위치를 의미할 수 있습니다.

---

### 4.5 Von Neumann Bottleneck

**용어:** Von Neumann Bottleneck

**정의:**
명령어와 데이터가 제한된 memory path 또는 bandwidth를 공유하여 CPU 처리 성능이 memory transfer 능력에 제한되는 현상입니다.

**필요한 이유:**
CPU 연산 속도와 memory access 속도의 차이를 이해하기 위한 개념입니다.

**하드웨어에서의 역할:**
Instruction fetch, load, store가 공유 자원에서 경쟁하면 stall이나 bandwidth 제한이 발생할 수 있습니다.

**관련된 다른 개념:**

```text
Memory wall
Bus contention
Cache
Prefetching
Pipeline stall
Structural hazard
```

단, 현대의 Von Neumann bottleneck은 단순히 하나의 물리적 bus 문제만을 뜻하지 않습니다.

더 넓게는 다음 문제를 포함합니다.

```text
CPU가 처리할 수 있는 속도
>
Memory system이 명령어와 데이터를 공급할 수 있는 속도
```

---

### 4.6 Modified Harvard Architecture

**용어:** Modified Harvard Architecture

**정의:**
상위 수준에서는 명령어와 데이터가 하나의 주소 공간을 공유하지만, CPU 가까운 곳에서는 instruction path와 data path를 분리한 구조입니다.

**필요한 이유:**
Von Neumann 구조의 유연성과 Harvard 구조의 병렬 접근 이점을 함께 얻기 위해 사용됩니다.

**하드웨어에서의 역할:**
일반적으로 별도의 L1 instruction cache와 L1 data cache를 사용하고, 하위 cache나 main memory는 공유합니다.

**관련된 다른 개념:**

```text
Split L1 cache
Unified L2 cache
Cache coherence
Instruction cache invalidation
Self-modifying code
```

현대의 많은 x86-64 및 Cortex-A CPU가 이 범주에 가깝습니다.

---

## 5. 내부 구조와 동작 원리

### 5.1 순수 Von Neumann 구조

가장 단순한 single-bus 구조를 생각해 보겠습니다.

```text
                            +----------------+
                            |     Memory     |
                            | Code + Data    |
                            +----------------+
                                   ^
                                   |
                              Shared bus
                                   |
             +---------------------+---------------------+
             |                                           |
             v                                           v
     Instruction fetch                              Load / Store
             |                                           |
             +------------------+------------------------+
                                |
                         +-------------+
                         |     CPU     |
                         +-------------+
```

CPU가 다음 명령어를 실행한다고 가정합니다.

```asm
ldr w0, [x1]
```

이 명령어는 두 번의 메모리 접근을 필요로 할 수 있습니다.

```text
1. ldr 명령어 자체를 memory에서 fetch
2. x1이 가리키는 data를 memory에서 load
```

공유 bus가 한 번에 하나의 요청만 처리할 수 있다면:

```text
Cycle 1: Instruction fetch
Cycle 2: Data load
```

두 접근을 같은 순간에 수행할 수 없습니다.

---

### 5.2 Harvard 구조

별도의 instruction bus와 data bus가 있다면:

```text
                         +------------------+
                         |       CPU        |
                         +------------------+
                            |            |
                Instruction bus       Data bus
                            |            |
                            v            v
                   +-------------+  +-------------+
                   | Instruction |  |    Data     |
                   |   Memory    |  |   Memory    |
                   +-------------+  +-------------+
```

같은 상황에서:

```text
Cycle N:
- 다음 instruction fetch
- 현재 load instruction의 data access
```

를 동시에 수행할 수 있습니다.

이 구조는 pipeline에서 특히 유용합니다.

---

### 5.3 5-stage pipeline에서의 문제

향후 배울 기본 pipeline은 다음 단계로 구성됩니다.

```text
IF → ID → EX → MEM → WB
```

여기서:

```text
IF  = Instruction Fetch
MEM = Data Memory Access
```

다음 명령어들이 있다고 합시다.

```asm
ldr w0, [x1]
add w2, w3, w4
```

시간표를 그리면:

```text
Cycle      1    2    3    4    5    6
ldr       IF   ID   EX   MEM  WB
add            IF   ID   EX   MEM  WB
```

Cycle 4에 주목해야 합니다.

```text
ldr instruction:
MEM stage에서 data memory 접근

후속 instruction:
IF stage에서 instruction memory 접근
```

메모리 port가 하나라면 두 요청이 충돌할 수 있습니다.

```text
Cycle 4:
- ldr의 data load 요청
- 새로운 instruction fetch 요청
```

이것은 **구조적 해저드(Structural Hazard)**입니다.

Instruction memory와 data memory가 분리되어 있으면 이 충돌을 피할 수 있습니다.

---

### 5.4 현대 CPU의 분리된 L1 cache

현대 CPU에서는 보통 다음과 같이 구성됩니다.

```text
                        +---------------------+
                        |      CPU Core       |
                        |                     |
                        | Front End  Back End |
                        +---------------------+
                           |             |
                    instruction       load/store
                       fetch              |
                           |             |
                       +------+      +------+
                       | L1 I |      | L1 D |
                       +------+      +------+
                           \             /
                            \           /
                             +---------+
                             |   L2    |
                             +---------+
                                  |
                             +---------+
                             |   L3    |
                             +---------+
                                  |
                                DRAM
```

### L1 Instruction Cache

입력:

```text
PC 또는 fetch address
```

출력:

```text
Instruction bytes
```

주요 특징:

```text
CPU가 쓰는 일반적인 data store의 직접 대상이 아님
실행 흐름에 맞춰 instruction을 빠르게 공급
branch predictor 및 instruction prefetcher와 연동
```

### L1 Data Cache

입력:

```text
Load/store address
```

출력 또는 처리:

```text
Load data 반환
Store data 임시 저장 또는 cache line 수정
```

주요 특징:

```text
Read와 write 모두 처리
Load/store unit과 연결
Store buffer와 연동 가능
```

---

### 5.5 하위 memory에서 다시 합쳐지는 이유

L1 cache는 매우 빠르게 동작해야 합니다.

Instruction과 data를 분리하면 다음 장점이 있습니다.

```text
동시에 instruction fetch와 data access 가능
각 cache를 목적에 맞게 설계 가능
port 경쟁 감소
front end와 load/store unit의 독립성 향상
```

그러나 L2, L3, DRAM까지 완전히 분리하면 다음 비용이 발생합니다.

```text
메모리 용량 활용의 비효율
회로 면적 증가
복잡한 coherence 관리
어떤 영역이 code이고 data인지 미리 분할해야 함
```

따라서 하위 단계에서는 통합 cache를 사용하는 경우가 많습니다.

---

### 5.6 명령어와 데이터는 어떻게 구분되는가?

중요한 사실은 다음과 같습니다.

> 메모리에 저장된 비트 자체에는 일반적으로 “instruction” 또는 “data”라는 본질적인 구분이 없습니다.

예를 들어 메모리에 다음 4바이트가 있다고 합시다.

```text
Address 0x1000:
00 00 80 D2
```

AArch64 instruction fetch 경로로 읽으면 유효한 명령어로 해석될 수 있습니다.

Data load 경로로 읽으면 단순한 정수 값으로 취급할 수 있습니다.

```text
Instruction fetch:
bit pattern → opcode와 operand로 decode

Data load:
bit pattern → integer, pointer, character 등의 값
```

차이를 결정하는 것은 다음과 같습니다.

```text
CPU가 해당 주소를 어떤 목적으로 접근했는가?
```

* PC가 가리키며 fetch했다면 instruction
* Load instruction이 읽었다면 data

운영체제의 page permission도 실행 가능성을 제한합니다.

```text
R: 읽기
W: 쓰기
X: 실행
```

---

## 6. 단계별 실행 추적

다음 C 코드를 사용하겠습니다.

```c
int load_and_add(const int *ptr)
{
    return *ptr + 10;
}
```

AArch64에서 개념적으로 다음과 같이 컴파일될 수 있습니다.

```asm
load_and_add:
    ldr w0, [x0]
    add w0, w0, #10
    ret
```

호출 전에:

```text
x0 = 0x4000
Memory[0x4000] = 32
```

명령어 주소:

```text
0x1000: ldr w0, [x0]
0x1004: add w0, w0, #10
0x1008: ret
```

---

### 6.1 `ldr w0, [x0]`의 Instruction Fetch

초기 상태:

```text
PC = 0x1000
x0 = 0x4000
```

Instruction path:

```text
PC 0x1000
   ↓
L1 Instruction Cache 조회
   ↓
ldr 명령어 bits 반환
   ↓
Instruction decoder
```

여기서 접근한 주소는 `0x1000`입니다.

---

### 6.2 Decode

Decoder가 다음 정보를 추출합니다.

```text
Operation        = Load 32-bit
Address register = x0
Destination      = w0
Offset           = 0
```

---

### 6.3 주소 계산

Load/store unit 또는 ALU가 effective address를 계산합니다.

```text
Effective address
= x0 + offset
= 0x4000 + 0
= 0x4000
```

---

### 6.4 Data Load

Data path:

```text
Address 0x4000
   ↓
L1 Data Cache 조회
   ↓
value 32 반환
   ↓
w0에 기록
```

이때 instruction fetch는 다음 명령어를 가져오기 위해 별도로 진행될 수 있습니다.

```text
Instruction path:
0x1004의 add 명령어 fetch

Data path:
0x4000의 값 32 load
```

Modified Harvard 구조에서는 두 접근이 서로 다른 L1 cache를 사용할 수 있습니다.

---

### 6.5 `add w0, w0, #10`

초기 operand:

```text
w0 = 32
Immediate = 10
```

ALU:

```text
32 + 10 = 42
```

결과:

```text
w0 = 42
```

---

### 6.6 `ret`

반환 주소를 PC에 적용합니다.

```text
PC ← x30
```

호출자는 `w0`에서 반환값 42를 받습니다.

---

### 6.7 전체 데이터 흐름

```text
Code address 0x1000
       |
       v
+----------------+
| L1 Instruction |
|     Cache       |
+----------------+
       |
       v
ldr w0, [x0]
       |
       v
Address calculation: 0x4000
       |
       v
+----------------+
| L1 Data Cache  |
+----------------+
       |
       v
     value 32
       |
       v
      w0 = 32
       |
       v
  add immediate 10
       |
       v
      w0 = 42
```

---

## 7. C/C++ 및 Assembly 예제

### 7.1 실행 가능한 코드

파일 이름: `memory_paths.c`

```c
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

static int load_and_add(const int *ptr)
{
    return *ptr + 10;
}

int main(int argc, char **argv)
{
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <integer>\n", argv[0]);
        return EXIT_FAILURE;
    }

    char *end = NULL;
    const long parsed = strtol(argv[1], &end, 10);

    if (argv[1][0] == '\0' || *end != '\0') {
        fprintf(stderr, "Invalid integer: %s\n", argv[1]);
        return EXIT_FAILURE;
    }

    /*
     * load_and_add() 안에서 signed overflow가 발생하지 않도록
     * INT_MAX - 10 이하로 제한합니다.
     */
    if (parsed < INT_MIN || parsed > INT_MAX - 10L) {
        fprintf(
            stderr,
            "Input must be between %d and %d.\n",
            INT_MIN,
            INT_MAX - 10
        );
        return EXIT_FAILURE;
    }

    const int value = (int)parsed;
    const int result = load_and_add(&value);

    if (printf("value = %d, result = %d\n", value, result) < 0) {
        fprintf(stderr, "Failed to write output.\n");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

---

### 7.2 컴파일

```bash
cc -O0 -Wall -Wextra -Wpedantic memory_paths.c -o memory_paths_O0
cc -O2 -Wall -Wextra -Wpedantic memory_paths.c -o memory_paths_O2
```

실행:

```bash
./memory_paths_O0 32
./memory_paths_O2 32
```

예상 출력:

```text
value = 32, result = 42
```

---

### 7.3 Assembly 생성

macOS 또는 Linux:

```bash
cc -O0 -S memory_paths.c -o memory_paths_O0.s
cc -O2 -S memory_paths.c -o memory_paths_O2.s
```

함수 찾기:

```bash
grep -n -A15 "load_and_add" memory_paths_O0.s
grep -n -A15 "load_and_add" memory_paths_O2.s
```

Apple Silicon에서는 개념적으로 다음과 비슷한 명령어가 나타날 수 있습니다.

```asm
_load_and_add:
    ldr w8, [x0]
    add w0, w8, #10
    ret
```

x86-64 System V에서는 다음과 비슷할 수 있습니다.

```asm
load_and_add:
    mov eax, DWORD PTR [rdi]
    add eax, 10
    ret
```

정확한 결과는 compiler, optimization level, ABI에 따라 달라집니다.

---

### 7.4 코드와 하드웨어의 대응

```c
return *ptr + 10;
```

대응 과정:

```text
ptr 값
 ↓
주소 계산
 ↓
Data cache에서 *ptr load
 ↓
Register에 값 저장
 ↓
Immediate 10과 ALU 연산
 ↓
반환 register에 결과 저장
```

여기서 함수의 machine instruction 자체는 instruction cache를 통해 공급됩니다.

```text
Instruction bytes → L1 I-cache
*ptr의 값         → L1 D-cache
```

---

### 7.5 코드 상수가 어디에 저장되는가?

다음 코드를 생각해 봅시다.

```c
static const int table[4] = { 10, 20, 30, 40 };
```

`const`라고 해서 반드시 instruction memory에 저장되는 것은 아닙니다.

운영체제가 있는 일반적인 시스템에서는 executable의 read-only data section에 배치될 가능성이 있습니다.

```text
.text    → executable instructions
.rodata  → read-only constants
.data    → initialized writable data
.bss     → zero-initialized data
```

하지만 실제 배치는 다음에 따라 달라집니다.

```text
Compiler
Linker script
Object format
Target architecture
Embedded memory map
Optimization
```

Microcontroller에서는 `const` table이 Flash에 저장되고 CPU가 별도의 경로로 접근할 수도 있습니다.

---

## 8. 성능과 메모리 관점

### 8.1 공유 memory path에서의 처리량 제한

다음과 같은 단순 CPU를 가정해 봅시다.

```text
공유 memory port는 cycle당 요청 1개 처리
모든 instruction은 한 번 fetch되어야 함
Load instruction은 추가 data access 1회 필요
```

명령어 100개 중 30개가 load라고 하겠습니다.

필요한 memory 요청 수:

```text
Instruction fetch = 100
Data load         = 30
Total             = 130 requests
```

Memory port가 cycle당 하나의 요청만 처리한다면, 최소한 130 cycle의 memory service 시간이 필요합니다.

명령어와 데이터 경로가 분리되어 있다면:

```text
Instruction side = 100 requests
Data side        = 30 requests
```

두 경로가 병렬로 처리될 수 있으므로 이상적인 최소 시간은 더 줄어들 수 있습니다.

단, 실제 성능은 cache miss, dependency, pipeline 구조 등에 영향을 받습니다.

---

### 8.2 Structural Hazard

다음 pipeline을 봅시다.

```text
Cycle       1    2    3    4    5
Load       IF   ID   EX   MEM  WB
Inst 2          IF   ID   EX   MEM
Inst 3               IF   ID   EX
Inst 4                    IF
```

Cycle 4에서:

```text
Load  → MEM 단계에서 data memory 사용
Inst4 → IF 단계에서 instruction memory 사용
```

하나의 memory port만 있다면 충돌합니다.

해결 방법:

```text
1. 한쪽 요청을 stall한다.
2. Dual-ported memory를 사용한다.
3. Instruction memory와 data memory를 분리한다.
4. 별도의 I-cache와 D-cache를 사용한다.
```

각 방법에는 비용이 있습니다.

| 해결책              | 장점         | 대가                    |
| ---------------- | ---------- | --------------------- |
| Stall            | 회로가 단순함    | 성능 저하                 |
| Dual-port memory | 동시 접근 가능   | 면적·전력·설계 비용           |
| Separate memory  | 높은 병렬성     | 공간 분할과 복잡성            |
| Split cache      | 현대 CPU에 적합 | cache coherence 관리 필요 |

---

### 8.3 Instruction Cache Miss와 Data Cache Miss

두 cache가 분리되어 있으면 miss 원인도 구분할 수 있습니다.

```text
I-cache miss:
실행할 instruction bytes를 찾지 못함

D-cache miss:
load/store 대상 data를 찾지 못함
```

I-cache miss가 발생하면 front end가 새로운 instruction을 공급하지 못합니다.

```text
I-cache miss
   ↓
Decode할 instruction 부족
   ↓
Execution engine이 점차 비어감
   ↓
IPC 감소
```

D-cache miss가 발생하면 해당 data에 의존하는 instruction이 기다립니다.

```text
D-cache miss
   ↓
Load 결과 대기
   ↓
Dependency chain 정지
   ↓
Memory stall
```

둘 다 성능을 낮추지만 영향 경로가 다릅니다.

---

### 8.4 Code Size와 I-cache

큰 프로그램이나 복잡한 함수는 더 많은 instruction bytes를 필요로 합니다.

```text
Code size 증가
   ↓
I-cache에 담기 어려움
   ↓
I-cache miss 증가 가능
   ↓
Front-end stall
```

따라서 compiler optimization에서 항상 “명령어 수가 적다”가 최선은 아닙니다.

예를 들어 loop unrolling은 branch overhead를 줄이지만 code size를 늘립니다.

```text
Loop unrolling 장점:
branch 수 감소
ILP 증가 가능

대가:
code size 증가
I-cache pressure 증가
```

---

### 8.5 Unified Cache의 장점

L2나 L3가 unified cache라면 code와 data가 같은 용량을 공유합니다.

장점:

```text
프로그램 특성에 따라 공간을 유연하게 사용
code가 작고 data가 크면 data가 더 많은 공간 사용 가능
data가 작고 code가 크면 instruction이 더 많은 공간 사용 가능
```

단점:

```text
Code와 data가 cache capacity를 두고 경쟁
한쪽 접근이 다른 쪽 cache line을 밀어낼 수 있음
```

---

### 8.6 성능 계산 예제

다음 조건을 가정하겠습니다.

```text
Base CPI              = 1.0
Instruction cache miss rate = 2%
I-cache miss penalty       = 20 cycles
Data load/store frequency  = 30%
D-cache miss rate          = 5%
D-cache miss penalty       = 30 cycles
```

#### Instruction miss로 인한 추가 CPI

모든 instruction은 fetch됩니다.

```text
추가 CPI
= I-cache miss rate × miss penalty
= 0.02 × 20
= 0.4
```

#### Data miss로 인한 추가 CPI

전체 instruction 중 30%만 data access를 수행한다고 가정합니다.

```text
추가 CPI
= Data access frequency
  × D-cache miss rate
  × miss penalty

= 0.30 × 0.05 × 30
= 0.45
```

#### 전체 평균 CPI

```text
Average CPI
= Base CPI
  + I-cache miss CPI
  + D-cache miss CPI

= 1.0 + 0.4 + 0.45
= 1.85
```

이 계산은 miss가 겹치지 않고 모든 penalty가 그대로 노출된다는 단순화입니다.

실제 out-of-order CPU에서는 여러 miss를 겹쳐 처리하거나, 다른 독립 명령어를 실행하여 일부 latency를 숨길 수 있습니다.

---

## 9. 실제 시스템 및 Embedded 응용

### 9.1 ARM Cortex-M

Cortex-M 기반 microcontroller는 Harvard 특성이 비교적 분명합니다.

일반적인 시스템을 단순화하면 다음과 같습니다.

```text
                  +----------------+
                  |   Cortex-M     |
                  +----------------+
                    |      |      |
                 I-Code  D-Code  System
                    |      |      |
                    v      v      v
                 Flash   Flash   SRAM/MMIO
```

구체적인 bus 구조는 Cortex-M 세대와 microcontroller 제조사에 따라 다르지만, 개념적으로:

```text
I-Code bus:
Flash에서 instruction fetch

D-Code bus:
Flash에 저장된 literal 또는 constant data 접근

System bus:
SRAM, peripheral, external memory 접근
```

따라서 Cortex-M은 단순한 “완전 분리 Harvard”라기보다 여러 bus를 사용하는 modified Harvard 구조로 이해하는 것이 적절합니다.

---

### 9.2 Flash와 SRAM

Microcontroller에서는 프로그램과 data가 서로 다른 memory technology에 저장되는 경우가 흔합니다.

| Memory           | 일반적인 용도                | 특징                 |
| ---------------- | ---------------------- | ------------------ |
| Flash            | Firmware, constants    | 비휘발성, write가 느림    |
| SRAM             | Stack, heap, variables | 휘발성, 빠른 read/write |
| Peripheral space | GPIO, UART, timer      | MMIO register      |
| EEPROM           | 설정값 저장                 | 비휘발성, 제한된 write 수명 |

부팅 시:

```text
Flash의 .text       → 보통 Flash에서 직접 실행
Flash의 .data 초기값 → SRAM으로 복사
.bss                 → SRAM에서 0으로 초기화
```

예:

```text
Flash
+------------------+
| Vector table     |
| .text            |
| .rodata          |
| .data init image |
+------------------+

SRAM
+------------------+
| .data runtime    |
| .bss             |
| Heap             |
| Stack            |
+------------------+
```

---

### 9.3 왜 `.data`를 SRAM으로 복사하는가?

다음 전역 변수를 생각해 봅시다.

```c
int counter = 10;
```

초기값 10은 전원이 꺼져도 firmware image에 남아 있어야 하므로 Flash에 저장됩니다.

하지만 실행 중에는 `counter`를 변경해야 하므로 SRAM에 있어야 합니다.

```text
Flash:
counter의 초기값 10 보관

Reset handler:
Flash에서 10을 읽어 SRAM의 counter 위치로 복사

Runtime:
CPU가 SRAM의 counter를 읽고 수정
```

---

### 9.4 `const` 데이터와 Flash

```c
static const unsigned int lookup_table[4] = {
    10U, 20U, 30U, 40U
};
```

Embedded linker script는 이 배열을 Flash의 read-only section에 배치할 수 있습니다.

CPU가 table을 읽을 때는 instruction fetch가 아니라 data access를 수행합니다.

즉:

```text
저장 위치는 Flash
접근 목적은 Data load
```

Flash에 있다고 해서 instruction인 것은 아닙니다.

---

### 9.5 DSP와 Harvard Architecture

Digital Signal Processor는 다음 작업을 반복하는 경우가 많습니다.

```c
output[i] += coefficient[j] * input[i - j];
```

한 cycle에 다음이 모두 필요할 수 있습니다.

```text
다음 instruction fetch
coefficient load
input data load
multiply-accumulate
```

따라서 DSP는 여러 독립 memory bank와 bus를 사용할 수 있습니다.

```text
Program memory
Coefficient memory
Sample data memory
```

목표는 계산 장치가 memory를 기다리지 않도록 하는 것입니다.

---

### 9.6 Linux와 x86-64

일반 Linux process는 code와 data가 하나의 virtual address space 안에 존재합니다.

개념적인 process memory map:

```text
High address
+------------------+
| Stack            |
+------------------+
| Shared libraries |
+------------------+
| Heap             |
+------------------+
| .bss             |
| .data            |
| .rodata          |
| .text            |
+------------------+
Low address
```

`.text`와 `.data`는 하나의 주소 공간에 있지만 page permission이 다를 수 있습니다.

```text
.text   → Read + Execute
.rodata → Read
.data   → Read + Write
Stack   → Read + Write
```

현대 보안 정책에서는 writable page와 executable page를 분리하려고 합니다.

```text
W^X:
Writable 또는 Executable 중 하나
동시에 둘 다 허용하지 않는 정책
```

---

### 9.7 JIT Compiler

JavaScript engine, JVM, 일부 emulator는 실행 중 machine code를 생성할 수 있습니다.

개념적인 흐름:

```text
1. Memory page를 writable로 할당
2. Machine instruction bytes를 기록
3. Page를 executable로 변경
4. 필요하면 instruction cache 동기화
5. 해당 주소로 분기
```

Modified Harvard 구조에서는 data path로 쓴 새로운 code가 instruction cache에 즉시 반영되지 않을 수 있습니다.

따라서 cache maintenance가 필요할 수 있습니다.

이 문제는 이후 cache coherence와 memory barrier에서 다시 다룹니다.

---

## 10. 아키텍처 비교

### 10.1 핵심 비교

| 항목                  | Von Neumann           | Harvard        |
| ------------------- | --------------------- | -------------- |
| 명령어·데이터 메모리         | 통합                    | 분리             |
| 주소 공간               | 보통 하나                 | 보통 별도          |
| 접근 경로               | 공유                    | 독립             |
| 동시 접근               | 제한될 수 있음              | 가능             |
| 구조 복잡도              | 비교적 낮음                | 비교적 높음         |
| memory 이용 유연성       | 높음                    | 분할로 인해 낮을 수 있음 |
| self-modifying code | 비교적 자연스러움             | 까다로울 수 있음      |
| 대표 환경               | 범용 컴퓨터의 추상 모델         | DSP, 일부 MCU    |
| 현대적 형태              | Unified address space | Split L1 cache |

---

### 10.2 왜 Von Neumann 구조가 유리한가?

명령어와 data가 동일한 memory를 사용하면 용량을 유연하게 활용할 수 있습니다.

예를 들어 총 1 MB memory가 있을 때:

```text
Code 200 KB
Data 800 KB
```

또는:

```text
Code 700 KB
Data 300 KB
```

처럼 프로그램에 따라 비율을 바꿀 수 있습니다.

완전히 분리된 Harvard 구조에서:

```text
Instruction memory = 512 KB
Data memory        = 512 KB
```

라면 instruction memory가 남더라도 data가 512 KB를 초과하면 사용할 수 없습니다.

---

### 10.3 왜 Harvard 구조가 유리한가?

Instruction과 data가 독립적으로 이동할 수 있습니다.

```text
같은 cycle에:
Instruction fetch
+
Data load
```

또한 서로 다른 특성의 memory를 사용할 수 있습니다.

```text
Instruction memory:
read 중심, sequential access 중심

Data memory:
read/write 모두 필요, random access 가능
```

Embedded에서는 다음 조합이 자연스럽습니다.

```text
Program → Flash
Data    → SRAM
```

---

### 10.4 현대 CPU에서 구분이 단순하지 않은 이유

현대 CPU를 한 단어로 분류하기 어려운 이유는 계층마다 구조가 다르기 때문입니다.

```text
ISA / programmer view:
하나의 주소 공간
→ Von Neumann 특성

L1 cache:
I-cache와 D-cache 분리
→ Harvard 특성

L2/L3/DRAM:
통합
→ Von Neumann 특성
```

따라서 현대 범용 CPU는 흔히 다음처럼 설명합니다.

> 프로그래머에게는 unified address space를 제공하지만, 성능을 위해 내부적으로 instruction과 data 경로를 분리한 modified Harvard architecture다.

---

### 10.5 Cortex-A와 Cortex-M

| 항목              | Cortex-A                  | Cortex-M                           |
| --------------- | ------------------------- | ---------------------------------- |
| 주요 용도           | Linux, 고성능 application    | Microcontroller, real-time control |
| 일반적인 memory 관리  | MMU, virtual memory       | MPU 또는 물리 주소 중심                    |
| Cache           | 보통 I/D cache와 다단계 cache   | 모델에 따라 없거나 소형 cache                |
| Program storage | DRAM, Flash, storage에서 적재 | 주로 on-chip Flash                   |
| Data storage    | DRAM, cache               | 주로 SRAM                            |
| 구조적 성격          | Modified Harvard          | Harvard 특성이 더 명확함                  |
| 복잡도             | 높음                        | 비교적 낮음                             |

모든 Cortex-M이 동일한 memory 구조를 갖는 것은 아닙니다. 실제 bus matrix, cache, tightly coupled memory는 core와 MCU 제품에 따라 달라집니다.

---

## 11. 흔한 오해

### 오해 1: Von Neumann Architecture는 반드시 물리적 bus가 하나다

너무 단순한 설명입니다.

현대 CPU는 unified address space를 제공하면서도 내부에 여러 bus, cache port, memory channel을 가질 수 있습니다.

```text
논리적으로 통합된 memory model
≠
물리적 wire가 하나뿐
```

---

### 오해 2: Harvard Architecture에서는 instruction을 data로 읽을 수 없다

순수 Harvard 구조에서는 어려울 수 있지만, modified Harvard 구조나 별도 instruction을 통해 program memory를 data처럼 읽을 수 있는 시스템도 있습니다.

Microcontroller에 따라 Flash read 방식과 address mapping이 다릅니다.

따라서 실제 ISA와 memory map을 확인해야 합니다.

---

### 오해 3: Flash에 있는 모든 값은 instruction이다

아닙니다.

Flash에는 다음이 모두 들어갈 수 있습니다.

```text
Machine instructions
Constant strings
Lookup tables
Vector table
Initialized data image
Metadata
```

어떤 방식으로 읽는지가 중요합니다.

---

### 오해 4: `const` 변수는 항상 instruction memory에 들어간다

C 언어의 `const`는 수정 가능성에 관한 언어 수준의 제한입니다.

실제 storage location은 다음에 의해 결정됩니다.

```text
Compiler
Linker
Linker script
ABI
Target platform
Optimization
```

---

### 오해 5: Separate I-cache와 D-cache가 있으면 완전한 Harvard Architecture다

반드시 그렇지는 않습니다.

하위 cache와 main memory가 통합되고 하나의 주소 공간을 공유한다면 modified Harvard라고 보는 편이 적절합니다.

---

### 오해 6: 명령어와 데이터의 비트 형식은 본질적으로 다르다

비트는 단지 비트입니다.

```text
같은 32-bit pattern
→ instruction fetch에서는 opcode로 해석
→ data load에서는 integer로 해석
```

의미는 사용 맥락과 ISA 규칙이 부여합니다.

---

### 오해 7: 분리된 cache는 항상 성능을 높인다

대체로 높은 처리량에 유리하지만 비용이 있습니다.

```text
같은 memory 내용의 두 복사본 관리
instruction cache invalidation
cache capacity 분할
하드웨어 면적 증가
전력 소비 증가
```

---

## 12. 실패 사례와 디버깅

### 12.1 Flash의 새로운 코드를 기록했지만 이전 코드가 실행되는 경우

JIT 또는 bootloader가 memory에 새로운 instruction bytes를 기록했다고 합시다.

```text
Data path를 통해 code memory 수정
```

하지만 instruction cache에는 이전 instruction이 남아 있을 수 있습니다.

```text
Memory:
새로운 instruction

I-cache:
이전 instruction
```

CPU가 I-cache를 사용하면 이전 코드가 계속 실행될 수 있습니다.

필요한 작업은 architecture에 따라 다음을 포함할 수 있습니다.

```text
1. Data cache clean
2. Instruction cache invalidate
3. Memory barrier
4. Instruction synchronization barrier
```

구체적인 명령은 ISA와 운영체제 API에 따라 달라집니다.

---

### 12.2 잘못된 linker script

Embedded system에서 `.text`를 SRAM 주소에 배치했지만 startup code가 해당 code를 Flash에서 SRAM으로 복사하지 않았다고 합시다.

결과:

```text
PC가 SRAM의 code address로 이동
SRAM에는 유효한 instruction이 없음
HardFault 또는 잘못된 실행
```

확인할 것:

```text
Linker map file
Section address
Load address와 execution address
Startup copy routine
Vector table address
```

---

### 12.3 `.data` 초기화 실패

전역 변수:

```c
int mode = 3;
```

Linker는 초기값을 Flash에 저장하고 runtime 위치를 SRAM에 배치할 수 있습니다.

Startup code가 복사를 하지 않으면:

```text
예상값: 3
실제 SRAM 값: 초기화되지 않은 값 또는 0
```

확인할 것:

```text
.data load address
.data start/end symbols
Reset handler의 copy loop
Linker script
```

---

### 12.4 Flash wait state 문제

CPU clock을 높였지만 Flash access latency 설정을 바꾸지 않으면 instruction fetch가 정상적으로 이루어지지 않을 수 있습니다.

가능한 증상:

```text
고속 clock 설정 후 HardFault
무작위 instruction 실행
부팅 실패
낮은 clock에서는 정상
```

MCU에서는 system clock을 높이기 전에 Flash wait state를 설정해야 하는 경우가 많습니다.

정확한 값은 해당 MCU의 reference manual을 확인해야 합니다.

---

### 12.5 Code를 data pointer로 잘못 해석

함수 pointer와 object pointer는 C 표준에서 완전히 같은 종류의 pointer로 보장되지 않습니다.

```c
void function(void);

unsigned char *ptr = (unsigned char *)(void *)function;
```

이런 변환과 접근은 portability 문제가 있습니다.

운영체제, compiler extension, ABI에서는 동작할 수 있지만:

```text
플랫폼에서 작동함
≠
ISO C가 모든 구현에서 보장함
```

---

### 12.6 실행 권한 문제

프로그램이 writable data page에 machine code를 작성하고 바로 함수 pointer로 호출하면 다음 문제가 생길 수 있습니다.

```text
Page에 execute permission이 없음
   ↓
Instruction fetch permission fault
   ↓
Segmentation fault 또는 access violation
```

보안 시스템은 data injection 공격을 막기 위해 non-executable memory를 사용합니다.

---

## 13. 확인 문제

정답은 바로 공개하지 않습니다.

### Level 1: 개념 확인

**문제 1**

Von Neumann Architecture와 Harvard Architecture의 핵심 차이를 다음 세 기준으로 설명하세요.

```text
Memory
Address space
Access path
```

---

**문제 2**

다음 문장이 정확한지 판단하고 이유를 설명하세요.

> “현대 x86-64 CPU는 L1 instruction cache와 L1 data cache가 분리되어 있으므로 순수 Harvard Architecture다.”

---

### Level 2: 동작 과정 추적

**문제 3**

다음 AArch64 코드가 있습니다.

```asm
ldr w0, [x1]
add w2, w0, #1
```

초기 상태:

```text
PC = 0x1000
x1 = 0x8000
Memory[0x8000] = 41
```

다음을 구분하여 추적하세요.

1. Instruction path에서 접근하는 주소
2. Data path에서 접근하는 주소
3. 각 명령어 실행 후 `w0`, `w2`, `PC`
4. 어느 시점에 I-cache와 D-cache가 각각 사용되는가

---

**문제 4**

다음 5-stage pipeline에서 single-ported memory를 사용할 때 structural hazard가 발생하는 cycle을 찾으세요.

```text
Cycle       1    2    3    4    5    6
Load       IF   ID   EX   MEM  WB
Add             IF   ID   EX   MEM  WB
Sub                  IF   ID   EX   MEM
```

왜 충돌하는지도 설명하세요.

---

### Level 3: 성능 계산 또는 코드 분석

**문제 5**

다음 조건을 가정합니다.

```text
Base CPI                  = 1.0
I-cache miss rate         = 1.5%
I-cache miss penalty      = 16 cycles
Load/store frequency      = 25%
D-cache miss rate         = 4%
D-cache miss penalty      = 24 cycles
```

다음을 계산하세요.

1. I-cache miss로 인한 추가 CPI
2. D-cache miss로 인한 추가 CPI
3. 전체 평균 CPI

모든 miss penalty가 그대로 노출된다고 단순화합니다.

---

**문제 6**

다음 두 최적화 중 어느 것이 I-cache 성능을 악화시킬 수 있는지 설명하세요.

```text
A. Loop unrolling
B. 작은 helper function을 여러 번 호출
```

둘 다 상황에 따라 장단점이 있으므로 code size와 branch overhead를 기준으로 분석하세요.

---

### Level 4: 설계·디버깅·실무 응용

**문제 7**

Bootloader가 Flash의 application 영역을 새 firmware로 덮어쓴 뒤 application으로 점프했지만 이전 firmware 동작 일부가 계속 나타납니다.

Modified Harvard Architecture와 cache를 기준으로 가능한 원인과 필요한 조치를 설명하세요.

---

**문제 8**

Embedded firmware에서 다음 현상이 발생했습니다.

```text
전역 변수 int mode = 3;
디버거로 확인하면 main() 시작 시 mode == 0
```

Flash, SRAM, `.data` section, startup code를 이용해 가능한 원인을 설명하고 디버깅 순서를 제시하세요.

---

## 14. 실습 과제

### 실습 목표

실행 파일에서 code와 data가 서로 다른 section에 배치되는 것을 확인합니다.

다음 항목을 관찰합니다.

```text
.text
.rodata
.data
.bss
```

---

### 준비물

macOS 또는 Linux의 C compiler와 binary inspection tool이 필요합니다.

Linux:

```text
gcc 또는 clang
readelf
objdump
nm
size
```

macOS:

```text
clang
otool
nm
size
```

---

### 코드

파일 이름: `sections_demo.c`

```c
#include <stdio.h>
#include <stdlib.h>

static const char message[] = "CPU architecture";
static int initialized_counter = 7;
static int zero_initialized_counter;

static int calculate(int value)
{
    return value + initialized_counter;
}

int main(void)
{
    zero_initialized_counter = calculate(5);

    if (printf(
            "%s: initialized=%d, zero_initialized=%d\n",
            message,
            initialized_counter,
            zero_initialized_counter
        ) < 0) {
        fprintf(stderr, "Failed to write output.\n");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

---

### 컴파일 및 실행

```bash
cc -O0 -g -Wall -Wextra -Wpedantic sections_demo.c -o sections_demo
./sections_demo
```

예상 출력:

```text
CPU architecture: initialized=7, zero_initialized=12
```

---

### Linux에서 section 확인

```bash
readelf -S sections_demo
```

symbol 확인:

```bash
nm -S sections_demo | grep -E \
'message|initialized_counter|zero_initialized_counter|calculate'
```

Disassembly:

```bash
objdump -d sections_demo
```

Read-only data:

```bash
objdump -s -j .rodata sections_demo
```

크기 확인:

```bash
size sections_demo
```

---

### macOS에서 section 확인

```bash
otool -l sections_demo
```

symbol 확인:

```bash
nm -m sections_demo | grep -E \
'message|initialized_counter|zero_initialized_counter|calculate'
```

Disassembly:

```bash
otool -tvV sections_demo
```

Data 확인은 Mach-O section 이름이 ELF와 다르므로 다음처럼 찾을 수 있습니다.

```bash
otool -s __TEXT __cstring sections_demo
otool -s __DATA __data sections_demo
```

정확한 section 배치는 compiler와 linker에 따라 달라질 수 있습니다.

---

### 예상 관찰 결과

대체로 다음과 같은 배치를 기대할 수 있습니다.

| 대상                                 | 예상 section               |
| ---------------------------------- | ------------------------ |
| `calculate()` machine instructions | `.text` 또는 `__text`      |
| `"CPU architecture"`               | `.rodata`, `__cstring` 등 |
| `initialized_counter`              | `.data`                  |
| `zero_initialized_counter`         | `.bss`                   |
| `main()` machine instructions      | `.text`                  |

---

### 관찰 결과가 나오는 이유

```text
.text:
실행할 machine instructions

.rodata:
수정하지 않는 상수

.data:
초기값이 있으며 수정 가능한 전역·정적 변수

.bss:
초기값이 0인 전역·정적 변수
```

`.bss`는 실행 파일에 모든 0 바이트를 직접 저장할 필요가 없습니다.

파일에는 크기 정보만 기록하고, loader 또는 startup code가 runtime memory를 0으로 초기화할 수 있습니다.

---

### 잘못된 결과가 나올 수 있는 원인

1. Compiler가 변수를 최적화하여 제거한 경우
2. `static` symbol이라 이름 표시 방식이 다른 경우
3. Position-independent executable로 주소 표현이 복잡한 경우
4. macOS와 Linux의 object format 차이
5. Compiler가 문자열을 다른 section에 병합한 경우
6. `-O2`에서 함수가 inline된 경우
7. Link-time optimization이 적용된 경우

---

### 추가 도전 과제 1

`message`에서 `const`를 제거하세요.

```c
static char message[] = "CPU architecture";
```

다시 컴파일하고 section이 바뀌는지 확인하세요.

---

### 추가 도전 과제 2

다음 큰 배열을 추가하세요.

```c
static unsigned char large_zero_array[1024 * 1024];
```

실행 파일 크기와 `.bss` 크기를 비교하세요.

질문:

```text
1 MB 배열인데 실행 파일 크기가 반드시 1 MB 증가하는가?
```

---

### 추가 도전 과제 3: Embedded 관점

STM32, RP2040 또는 다른 Cortex-M 프로젝트가 있다면 linker map file을 생성하세요.

GCC 예:

```bash
-Wl,-Map=firmware.map
```

찾을 항목:

```text
.text의 load address
.data의 load address
.data의 runtime address
.bss start/end
Stack 위치
Vector table 위치
```

---

## 15. 면접에서 설명하는 방법

### 15.1 30초 설명

논리 구조:

```text
공유 여부
→ 병렬 접근
→ 현대 CPU의 혼합 형태
```

예시:

> Von Neumann Architecture는 instruction과 data가 하나의 주소 공간과 memory system을 공유하는 구조이고, Harvard Architecture는 두 종류의 memory나 접근 경로를 분리합니다. Harvard 구조는 instruction fetch와 data access를 동시에 수행하기 유리하지만 memory 활용이 덜 유연할 수 있습니다. 현대 범용 CPU는 보통 unified address space를 제공하면서 L1 instruction cache와 data cache를 분리하므로 modified Harvard architecture로 설명합니다.

---

### 15.2 2분 설명

답변 구조:

1. 두 구조의 형식적 차이
2. 성능 차이
3. 구현 비용
4. Embedded 사례
5. 현대 CPU의 modified 구조

예시:

> Von Neumann Architecture에서는 instruction과 data가 동일한 memory와 address space에 저장됩니다. 구조가 단순하고 전체 memory 용량을 유연하게 사용할 수 있지만, instruction fetch와 load/store가 같은 경로나 bandwidth를 두고 경쟁할 수 있습니다. 이를 Von Neumann bottleneck의 한 형태로 볼 수 있습니다. Harvard Architecture는 instruction과 data memory 또는 bus를 분리하여 두 접근을 동시에 수행할 수 있고, program에는 Flash, data에는 SRAM처럼 서로 다른 memory technology를 사용할 수 있습니다. 대신 용량이 고정적으로 분리되고 code와 data 사이의 이동이 복잡해질 수 있습니다. 현대 x86이나 Cortex-A CPU는 software 관점에서는 하나의 address space를 사용하지만, core 가까이에서는 L1 I-cache와 D-cache가 분리되고 하위 cache와 DRAM은 통합됩니다. 따라서 순수한 두 모델 중 하나보다는 modified Harvard architecture라고 설명하는 것이 더 정확합니다.

---

### 15.3 심화 꼬리 질문

면접에서 이어질 수 있는 질문:

```text
Von Neumann bottleneck은 단순한 bus 충돌만을 의미하는가?
Separate I-cache와 D-cache 사이에는 coherence 문제가 없는가?
Self-modifying code는 왜 cache flush가 필요한가?
Cortex-M의 I-Code, D-Code, System bus는 어떤 역할을 하는가?
Executable page와 writable page를 분리하는 이유는 무엇인가?
JIT compiler는 memory protection을 어떻게 변경하는가?
Flash에 있는 const data는 instruction인가 data인가?
```

답변 시 다음 계층을 구분해야 합니다.

```text
Programmer-visible address space
Physical memory
Cache hierarchy
Bus structure
Page permissions
Compiler와 linker section
```

---

## 16. 핵심 정리

1. **Von Neumann Architecture는 instruction과 data를 통합된 memory system에 저장합니다.**

2. **Harvard Architecture는 instruction과 data memory 또는 접근 경로를 분리합니다.**

3. Harvard 구조에서는 다음 작업을 동시에 수행하기 쉽습니다.

   ```text
   Instruction fetch
   +
   Data load/store
   ```

4. Von Neumann 구조는 memory 용량을 code와 data 사이에서 유연하게 사용할 수 있습니다.

5. 단일 memory port를 사용하는 pipeline에서는 IF와 MEM 단계가 충돌하여 structural hazard가 발생할 수 있습니다.

6. 현대 CPU는 흔히 다음 구조를 사용합니다.

   ```text
   L1 Instruction Cache → 분리
   L1 Data Cache        → 분리
   L2/L3/DRAM           → 통합
   ```

   이를 Modified Harvard Architecture라고 설명할 수 있습니다.

7. 메모리의 비트 자체가 instruction 또는 data로 고정되는 것은 아닙니다.

   ```text
   PC를 통한 fetch → instruction
   Load를 통한 접근 → data
   ```

8. Embedded system에서는 일반적으로:

   ```text
   Firmware → Flash
   Runtime data → SRAM
   Peripheral → MMIO
   ```

9. `.data`의 초기값은 Flash에 보관되고 reset 과정에서 SRAM으로 복사될 수 있습니다.

10. code를 runtime에 수정하거나 생성한 경우, D-cache와 I-cache가 분리된 시스템에서는 cache synchronization이 필요할 수 있습니다.

11. “Von Neumann 대 Harvard”는 현대 CPU를 완전히 분류하는 이분법이 아닙니다. 어느 계층을 기준으로 보는지 명시해야 합니다.
