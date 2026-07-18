# Lecture 4. Register, ALU, Control Unit, Datapath

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> CPU 내부에서 하나의 명령어가 실행될 때, 데이터는 정확히 어디에서 읽혀 어디를 지나고 어디에 저장되는가?

세부 질문은 다음과 같습니다.

1. 레지스터(Register)는 단순히 빠른 메모리인가?
2. ALU는 덧셈 외에 무엇을 수행하는가?
3. 제어 장치(Control Unit)는 명령어를 어떻게 실제 회로 동작으로 바꾸는가?
4. 데이터패스(Datapath)는 무엇이며, 제어 경로(Control Path)와 어떻게 다른가?
5. `add`, `load`, `store`, `branch` 명령어는 같은 하드웨어를 어떻게 다르게 사용하나?
6. 하나의 ALU를 여러 용도로 공유하면 어떤 장점과 문제가 생기는가?

이번 Lecture는 업로드한 강의 지침의 Part 1 범위에 따라 진행합니다. 

---

## 2. 이전 Lecture와의 연결

Lecture 3에서는 다음 두 계층을 구분했습니다.

```text
ISA
→ CPU가 무엇을 해야 하는가?

Microarchitecture
→ 그것을 하드웨어로 어떻게 구현하는가?
```

예를 들어 AArch64 명령어:

```asm
add w0, w1, w2
```

ISA는 다음 결과를 요구합니다.

```text
w0 ← w1 + w2
```

하지만 실제 CPU가 이 결과를 만들려면 다음 질문에 답해야 합니다.

```text
w1과 w2는 어디에 저장되어 있는가?
어떤 회로가 두 값을 읽는가?
어떤 회로가 덧셈을 수행하는가?
결과를 어디에 기록하는가?
덧셈을 선택하도록 누가 신호를 보내는가?
```

그 답이 오늘 배울 네 요소입니다.

```text
Register
ALU
Control Unit
Datapath
```

---

# 3. 직관적 설명

## 3.1 작은 계산 작업실

CPU를 작은 계산 작업실이라고 생각해 봅시다.

작업실에는 다음 장비가 있습니다.

| 비유          | 실제 CPU 구성 요소 |
| ----------- | ------------ |
| 작업대 위 작은 서랍 | Register     |
| 계산기         | ALU          |
| 작업 지시 담당자   | Control Unit |
| 재료가 이동하는 통로 | Datapath     |
| 통로 선택 스위치   | Multiplexer  |
| 작업 진행 박자    | Clock        |

다음 명령을 받았다고 가정합니다.

```text
서랍 1의 값과 서랍 2의 값을 더해 서랍 0에 넣어라.
```

CPU에서는 다음 instruction에 대응합니다.

```asm
add w0, w1, w2
```

내부 작업은 다음과 같습니다.

```text
Register file에서 w1 읽기
Register file에서 w2 읽기
두 값을 ALU로 전달
ALU 동작을 ADD로 설정
결과를 w0에 기록
```

---

## 3.2 Data와 Control의 구분

CPU 내부를 이해할 때 가장 중요한 구분 중 하나입니다.

### 데이터 경로

실제 값이 이동하는 경로입니다.

```text
10
20
30
주소 0x4000
명령어 비트
```

### 제어 경로

하드웨어가 무엇을 해야 하는지를 지정하는 신호입니다.

```text
ALU는 ADD를 수행하라.
Register write를 허용하라.
Memory를 읽어라.
Memory에 쓰지 마라.
다음 PC는 branch target을 선택하라.
```

즉:

```text
Datapath
→ 값이 흐르는 길

Control path
→ 그 길과 연산을 선택하는 명령 신호
```

---

## 3.3 철도 비유

Datapath를 철도망으로 볼 수도 있습니다.

```text
Register file ──────> ALU ──────> Register file
       \                |
        \               |
         └────> Memory ─┘
```

여러 선로가 있지만 모든 값이 동시에 모든 곳으로 갈 수는 없습니다.

제어 장치는 선로 전환기처럼 multiplexer를 제어합니다.

```text
ALU 두 번째 입력:
- Register 값
- Immediate 값
- 상수 4

이 중 무엇을 선택할 것인가?
```

제어 신호가 이를 결정합니다.

---

## 3.4 비유의 한계

실제 CPU는 중앙의 한 명령 담당자가 순차적으로 모든 부품에 지시하는 구조가 아닐 수 있습니다.

제어 신호는 다음 방식으로 생성될 수 있습니다.

```text
Instruction bit를 조합 논리 회로가 해석
Pipeline stage별 control signal 생성
Decoded µop이 scheduler에 전달
Microcode가 여러 내부 동작 생성
```

현대 out-of-order CPU에서는 하나의 거대한 Control Unit보다 분산된 여러 제어 구조가 동작합니다.

그러나 기본 CPU 모델에서는 하나의 제어 장치로 묶어 이해하는 것이 유용합니다.

---

# 4. 형식적 정의

## 4.1 Register

**용어:** 레지스터(Register)

**정의:**
CPU 내부에서 operand, 주소, 중간 결과, 제어 상태를 저장하는 소형 고속 저장 장치입니다.

**필요한 이유:**
ALU가 매 연산마다 느린 main memory에서 값을 가져오지 않도록 하기 위해 필요합니다.

**하드웨어에서의 역할:**
Clock edge에 값을 저장하고, 조합 논리 회로에 입력값을 제공합니다.

**관련 개념:**

```text
Register file
General-purpose register
Program Counter
Stack Pointer
Status register
Physical register
Architectural register
```

---

## 4.2 Register File

**용어:** 레지스터 파일(Register File)

**정의:**
여러 개의 general-purpose register를 하나의 구조로 묶고, register 번호를 이용해 읽고 쓰게 하는 하드웨어입니다.

**필요한 이유:**
Instruction이 source와 destination register를 번호로 지정할 수 있어야 하기 때문입니다.

**하드웨어에서의 역할:**
일반적으로 여러 read port와 하나 이상의 write port를 제공합니다.

**관련 개념:**

```text
Read port
Write port
Register index
Register renaming
Multi-ported memory
```

가장 단순한 형태:

```text
Read address 1 ──> Register File ──> Read data 1
Read address 2 ──> Register File ──> Read data 2

Write address  ──> Register File
Write data     ──> Register File
Write enable   ──> Register File
```

---

## 4.3 ALU

**용어:** 산술 논리 장치(Arithmetic Logic Unit, ALU)

**정의:**
정수 산술, bitwise logic, shift, comparison, 주소 계산 등을 수행하는 조합 논리 회로입니다.

**필요한 이유:**
Instruction이 요구하는 계산을 수행하기 위해 필요합니다.

**하드웨어에서의 역할:**
두 개 이상의 입력 operand와 operation control을 받아 결과와 상태 정보를 출력합니다.

**관련 개념:**

```text
Adder
Shifter
Comparator
Multiplier
Condition flags
Execution unit
```

대표 연산:

```text
ADD
SUB
AND
OR
XOR
Shift
Compare
Effective-address calculation
```

---

## 4.4 Control Unit

**용어:** 제어 장치(Control Unit)

**정의:**
Instruction encoding과 현재 상태를 해석하여 datapath 구성 요소를 제어하는 신호를 생성하는 논리입니다.

**필요한 이유:**
같은 하드웨어를 instruction 종류에 따라 다르게 사용하기 위해 필요합니다.

**하드웨어에서의 역할:**
다음과 같은 제어 신호를 생성합니다.

```text
Register read/write 선택
ALU operation 선택
ALU input source 선택
Memory read/write
Result write-back source
PC update source
Branch enable
```

**관련 개념:**

```text
Hardwired control
Microcoded control
Instruction decoder
Control signal
Finite-state machine
```

---

## 4.5 Datapath

**용어:** 데이터패스(Datapath)

**정의:**
CPU 내부에서 operand와 결과가 저장되고 이동하며 계산되는 하드웨어 경로의 집합입니다.

**필요한 이유:**
Instruction의 데이터 변환 과정을 실제 회로로 구현하기 위해 필요합니다.

**하드웨어에서의 역할:**
Register, ALU, multiplexer, bus, sign extender, memory interface 등을 연결합니다.

**관련 개념:**

```text
Control path
Bus
Multiplexer
Pipeline register
Execution unit
```

---

## 4.6 Control Path

**용어:** 제어 경로(Control Path)

**정의:**
Datapath의 각 장치가 어떤 동작을 수행할지 결정하는 제어 신호의 생성 및 전달 구조입니다.

예:

```text
ALUControl = ADD
ALUSrc = Immediate
RegWrite = 1
MemRead = 0
MemWrite = 0
ResultSrc = ALU
```

Datapath와 control path의 관계:

```text
Instruction bits
      |
      v
Control Unit
      |
      +── control signals ──> Datapath

Data values
      |
      v
Datapath
      |
      v
Result
```

---

## 4.7 Multiplexer

**용어:** 멀티플렉서(Multiplexer, MUX)

**정의:**
여러 입력 중 하나를 control signal에 따라 선택하여 출력하는 회로입니다.

**필요한 이유:**
하나의 datapath 장치를 여러 종류의 instruction이 공유할 수 있도록 합니다.

예:

```text
             +----------------+
Register ───>|                |
Immediate ──>|      MUX       |──> ALU input B
Constant 4 ─>|                |
             +----------------+
                   ^
                   |
                Select
```

---

## 4.8 Clocked State와 Combinational Logic

CPU 내부 회로는 크게 두 종류로 나눌 수 있습니다.

### 상태 저장 요소

Clock에 맞춰 값을 저장합니다.

```text
Register
Program Counter
Pipeline register
Cache state
```

### 조합 논리

현재 입력으로 즉시 출력을 계산합니다.

```text
ALU
Decoder
Multiplexer
Adder
Comparator
```

기본 관계:

```text
Stored state
    |
    v
Combinational logic
    |
    v
Next state
    |
 clock edge
    v
Stored state 갱신
```

---

# 5. 내부 구조와 동작 원리

## 5.1 기본 Datapath

다음은 단순화된 register-register 연산 datapath입니다.

```text
                   Instruction
                        |
               +--------+--------+
               |                 |
          rs1 field          rs2 field
               |                 |
               v                 v
        +-----------------------------+
        |        Register File        |
        |                             |
        | Read port 1   Read port 2   |
        +-----------------------------+
               |                 |
               | operand A       | operand B
               v                 v
             +-----------------------+
             |          ALU          |
             +-----------------------+
                        |
                        | result
                        v
                +---------------+
                | Register File |
                |  Write port   |
                +---------------+
                        ^
                        |
                  destination rd
```

Control Unit은 다음을 지정합니다.

```text
rs1 번호
rs2 번호
rd 번호
ALU operation
Register write enable
```

---

## 5.2 `add` 명령어의 Datapath

예:

```asm
add w0, w1, w2
```

초기 상태:

```text
w1 = 10
w2 = 20
w0 = 100
```

Instruction decoder가 추출하는 정보:

```text
Operation = ADD
Source 1  = w1
Source 2  = w2
Destination = w0
```

Datapath:

```text
w1 = 10 ─────────────┐
                     v
                   +-----+
                   | ALU |───> 30
                   +-----+
                     ^
w2 = 20 ─────────────┘
```

Write-back:

```text
w0 ← 30
```

필요한 제어 신호의 개념적 값:

```text
ALUSrc     = Register
ALUControl = ADD
RegWrite   = 1
MemRead    = 0
MemWrite   = 0
ResultSrc  = ALU
```

---

## 5.3 Immediate 연산

다음 명령어를 봅시다.

```asm
add w0, w1, #5
```

두 번째 ALU 입력은 register가 아니라 instruction에 포함된 immediate입니다.

```text
Register w1 ──────────────> ALU input A

Register w2 ──┐
              ├─ MUX ─────> ALU input B
Immediate 5 ──┘
```

제어 신호:

```text
ALUSrc = Immediate
```

이때 instruction의 immediate field는 필요한 bit width로 확장되어야 합니다.

```text
Immediate field
      |
      v
Sign extension 또는 Zero extension
      |
      v
ALU input
```

---

## 5.4 Load Datapath

C 코드:

```c
value = *ptr;
```

AArch64:

```asm
ldr w0, [x1]
```

Load에는 두 단계가 필요합니다.

```text
1. Effective address 계산
2. 해당 주소에서 memory read
```

Datapath:

```text
x1 ───────────────┐
                  v
                +-----+
Offset 0 ───────>| ALU |───> Effective address
                +-----+
                              |
                              v
                       +--------------+
                       | Data Memory  |
                       +--------------+
                              |
                              v
                            w0
```

ALU는 덧셈 결과 자체가 최종 데이터가 아니라 memory address를 계산합니다.

제어 신호:

```text
ALUSrc     = Immediate offset
ALUControl = ADD
MemRead    = 1
MemWrite   = 0
RegWrite   = 1
ResultSrc  = Memory
```

---

## 5.5 Store Datapath

C 코드:

```c
*ptr = value;
```

AArch64:

```asm
str w0, [x1]
```

Datapath:

```text
x1 ───────────────┐
                  v
                +-----+
Offset 0 ───────>| ALU |───> Effective address
                +-----+
                              |
                              v
                       +--------------+
w0 ──────────────────>| Data Memory  |
                       |    Write     |
                       +--------------+
```

Store는 register file에 결과를 쓰지 않습니다.

제어 신호:

```text
ALUControl = ADD
MemRead    = 0
MemWrite   = 1
RegWrite   = 0
```

---

## 5.6 Branch Datapath

다음 조건문을 생각해 봅시다.

```c
if (a == b) {
    result = 1;
}
```

개념적인 assembly:

```asm
cmp w0, w1
b.eq target
```

Branch는 두 가지 계산이 필요할 수 있습니다.

```text
1. 조건 비교
2. Branch target address 계산
```

단순 datapath:

```text
Register A ─────┐
                v
              +------------+
Register B ───>| Comparator |──> condition true/false
              +------------+

PC ────────────┐
                v
              +--------+
Offset ───────>| Adder  |──> Branch target
              +--------+

condition
   |
   v
+----------------+
| Next-PC MUX    |
+----------------+
  |            |
PC + 4      Branch target
```

제어 신호는 condition에 따라 next PC를 선택합니다.

---

## 5.7 하나의 ALU를 공유하는 구조

작은 CPU는 하나의 ALU를 여러 목적으로 사용할 수 있습니다.

```text
산술 연산
주소 계산
PC 증가
Branch target 계산
비교
```

장점:

```text
회로 면적 감소
전력 감소
설계 단순화
```

단점:

```text
동시에 여러 계산 불가능
여러 cycle 필요 가능
Structural hazard 발생 가능
처리량 감소
```

고성능 CPU는 목적별로 여러 실행 장치를 둡니다.

```text
Integer ALU
Branch unit
Address-generation unit
Multiplier
Divider
Vector unit
```

---

## 5.8 Hardwired Control

Hardwired control은 instruction bit와 상태를 논리 회로로 직접 해석합니다.

```text
Opcode bits
    |
    v
Combinational decode logic
    |
    +── RegWrite
    +── ALUSrc
    +── MemRead
    +── MemWrite
    +── Branch
```

장점:

```text
빠름
규칙적인 ISA에 적합
```

단점:

```text
복잡한 instruction set에서는 회로가 복잡해짐
수정이 어려움
```

---

## 5.9 Microcoded Control

Microcoded control은 하나의 architectural instruction을 작은 내부 control step으로 나눕니다.

```text
Instruction
    |
    v
Microcode address
    |
    v
Microinstruction sequence
    |
    +── register transfer
    +── ALU operation
    +── memory access
    +── PC update
```

예를 들어 복잡한 instruction이 다음 내부 순서로 구현될 수 있습니다.

```text
Step 1: Source operand 읽기
Step 2: 주소 계산
Step 3: Memory read
Step 4: ALU 연산
Step 5: 결과 기록
```

장점:

```text
복잡한 instruction 구현이 쉬움
동작 수정 가능성이 높음
```

단점:

```text
추가 제어 단계
일부 instruction의 latency 증가
```

---

## 5.10 현대 CPU의 분산된 Datapath

현대 superscalar CPU는 단일 datapath가 아니라 여러 병렬 경로를 가집니다.

```text
                       Scheduler
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   Integer ALU 0      Integer ALU 1      Branch Unit
        |
        +------------------+------------------+
                           |
                        Results
                           |
                     Physical Registers
```

Load/store 쪽은 별도 경로를 갖습니다.

```text
Load Queue
    |
Address Generation Unit
    |
L1 Data Cache
    |
Physical Register
```

기본 모델은 단일 datapath지만 실제 processor에는 여러 datapath가 병렬로 존재할 수 있습니다.

---

# 6. 단계별 실행 추적

다음 C 함수를 사용하겠습니다.

```c
int update(int *ptr, int value)
{
    int old = *ptr;
    int next = old + value;
    *ptr = next;
    return next;
}
```

AArch64의 개념적 assembly:

```asm
update:
    ldr w2, [x0]
    add w1, w2, w1
    str w1, [x0]
    mov w0, w1
    ret
```

호출 시 초기 상태:

```text
x0 = 0x4000
w1 = 7
Memory[0x4000] = 35
```

---

## 6.1 `ldr w2, [x0]`

### Register read

```text
x0 → 0x4000
```

### ALU 입력

```text
Input A = 0x4000
Input B = offset 0
Operation = ADD
```

### 주소 계산

```text
0x4000 + 0 = 0x4000
```

### Memory read

```text
Memory[0x4000] → 35
```

### Write-back

```text
w2 ← 35
```

Control:

```text
ALUSrc     = Immediate
ALUControl = ADD
MemRead    = 1
MemWrite   = 0
ResultSrc  = Memory
RegWrite   = 1
```

---

## 6.2 `add w1, w2, w1`

Register file read:

```text
w2 = 35
w1 = 7
```

ALU:

```text
35 + 7 = 42
```

Write-back:

```text
w1 ← 42
```

Control:

```text
ALUSrc     = Register
ALUControl = ADD
MemRead    = 0
MemWrite   = 0
ResultSrc  = ALU
RegWrite   = 1
```

---

## 6.3 `str w1, [x0]`

Register read:

```text
x0 = 0x4000
w1 = 42
```

Address calculation:

```text
0x4000 + 0 = 0x4000
```

Memory write:

```text
Memory[0x4000] ← 42
```

Control:

```text
ALUControl = ADD
MemWrite   = 1
RegWrite   = 0
```

---

## 6.4 `mov w0, w1`

AArch64에서 `mov`는 실제로 다른 instruction의 alias일 수 있습니다.

개념적 동작:

```text
w0 ← w1
w0 = 42
```

이 과정은 ALU나 단순 register-transfer 경로를 사용할 수 있습니다.

---

## 6.5 최종 상태

```text
w0 = 42
w1 = 42
w2 = 35
Memory[0x4000] = 42
```

C 관점:

```text
old  = 35
value = 7
next = 42
*ptr = 42
return 42
```

---

## 6.6 Datapath 전체 추적

```text
              x0 = 0x4000
                    |
                    v
                +-------+
offset 0 ------>|  ALU  |----> address 0x4000
                +-------+
                    |
                    v
                +--------+
                | Memory |
                +--------+
                    |
                  value 35
                    |
                    v
                  w2 = 35
                    |
                    +---------+
                              |
w1 = 7 -----------------------v
                            +-----+
                            | ALU |----> 42
                            +-----+
                               |
                               v
                             w1 = 42
                               |
                               +------> Memory[0x4000]
                               |
                               +------> w0
```

---

# 7. C/C++ 및 Assembly 예제

## 7.1 실행 가능한 코드

파일 이름: `datapath_demo.c`

```c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

__attribute__((noinline))
static int update(int *ptr, int value)
{
    const int old = *ptr;

    /*
     * 호출자가 overflow가 발생하지 않는 입력을 전달한다고 가정합니다.
     */
    const int next = old + value;

    *ptr = next;
    return next;
}

static int parse_int(const char *text, int *result)
{
    char *end = NULL;

    errno = 0;
    const long value = strtol(text, &end, 10);

    if (errno == ERANGE || value < INT_MIN || value > INT_MAX) {
        return -1;
    }

    if (text[0] == '\0' || *end != '\0') {
        return -1;
    }

    *result = (int)value;
    return 0;
}

int main(int argc, char **argv)
{
    if (argc != 3) {
        fprintf(
            stderr,
            "Usage: %s <initial-value> <increment>\n",
            argv[0]
        );
        return EXIT_FAILURE;
    }

    int initial = 0;
    int increment = 0;

    if (parse_int(argv[1], &initial) != 0) {
        fprintf(stderr, "Invalid initial value: %s\n", argv[1]);
        return EXIT_FAILURE;
    }

    if (parse_int(argv[2], &increment) != 0) {
        fprintf(stderr, "Invalid increment: %s\n", argv[2]);
        return EXIT_FAILURE;
    }

    if ((increment > 0 && initial > INT_MAX - increment) ||
        (increment < 0 && initial < INT_MIN - increment)) {
        fprintf(stderr, "The addition would cause signed overflow.\n");
        return EXIT_FAILURE;
    }

    int value = initial;
    const int result = update(&value, increment);

    if (printf(
            "initial=%d, increment=%d, stored=%d, returned=%d\n",
            initial,
            increment,
            value,
            result
        ) < 0) {
        fprintf(stderr, "Failed to write output.\n");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

---

## 7.2 컴파일 및 실행

```bash
cc -O2 -Wall -Wextra -Wpedantic datapath_demo.c -o datapath_demo
./datapath_demo 35 7
```

예상 출력:

```text
initial=35, increment=7, stored=42, returned=42
```

---

## 7.3 Assembly 생성

macOS 또는 Linux:

```bash
cc -O2 -S datapath_demo.c -o datapath_demo.s
```

함수 확인:

```bash
grep -n -A15 "update" datapath_demo.s
```

Apple Silicon에서는 개념적으로 다음과 비슷할 수 있습니다.

```asm
_update:
    ldr w8, [x0]
    add w1, w8, w1
    str w1, [x0]
    mov w0, w1
    ret
```

x86-64 System V에서는 다음과 비슷할 수 있습니다.

```asm
update:
    add esi, DWORD PTR [rdi]
    mov DWORD PTR [rdi], esi
    mov eax, esi
    ret
```

정확한 assembly는 compiler와 target에 따라 달라집니다.

---

## 7.4 x86 memory operand의 의미

다음 x86 instruction:

```asm
add esi, DWORD PTR [rdi]
```

Architectural 의미:

```text
esi ← esi + Memory[rdi]
```

하지만 내부 datapath 관점에서는 최소한 다음 작업이 필요합니다.

```text
Address calculation
Data-cache read
Integer addition
Destination register result
```

하나의 assembly instruction이라고 datapath 작업도 하나인 것은 아닙니다.

---

## 7.5 `volatile`을 붙이면 어떻게 달라지는가?

다음 함수를 생각해 봅시다.

```c
static int update_volatile(volatile int *ptr, int value)
{
    int old = *ptr;
    int next = old + value;
    *ptr = next;
    return next;
}
```

`volatile`은 compiler에게 해당 object 접근을 관찰 가능한 side effect로 취급하도록 요구합니다.

따라서 load와 store를 함부로 제거하거나 합치지 못하게 할 수 있습니다.

그러나 다음은 보장하지 않습니다.

```text
Atomicity
Thread synchronization
CPU memory ordering 전체
Race-condition 방지
```

MMIO에서는 `volatile`이 필요할 수 있지만, architecture-specific barrier도 추가로 필요할 수 있습니다.

---

# 8. 성능과 메모리 관점

## 8.1 Register가 빠른 이유

Register는 CPU core 내부에 있고 ALU와 직접 연결됩니다.

개념적인 거리:

```text
Register
  ↓
ALU
```

Memory access는 더 긴 경로를 거칩니다.

```text
Virtual address
  ↓
TLB
  ↓
L1 cache
  ↓ miss
L2
  ↓ miss
L3
  ↓ miss
DRAM
```

따라서 compiler는 가능한 값을 register에 유지하려고 합니다.

---

## 8.2 Register Pressure

동시에 살아 있는 값이 너무 많으면 register 수가 부족할 수 있습니다.

예:

```c
int a0, a1, a2, a3, a4, a5, a6, a7;
```

Compiler가 모든 값을 register에 유지하지 못하면 일부를 stack에 저장합니다.

이를 register spill이라고 합니다.

```text
Register 부족
    ↓
Stack memory에 값 저장
    ↓
나중에 다시 load
    ↓
추가 instruction과 memory access
```

성능 비용:

```text
추가 load/store
더 큰 code size
Cache pressure
Dependency 증가
```

---

## 8.3 ALU Latency와 Throughput

ALU 연산마다 latency와 throughput이 다를 수 있습니다.

예시적 모델:

| 연산          |     Latency |     Throughput |
| ----------- | ----------: | -------------: |
| Integer add |     1 cycle | cycle당 여러 개 가능 |
| Multiply    |    3 cycles |   cycle당 1개 가능 |
| Divide      | 10~수십 cycle |             낮음 |
| Shift       |     1 cycle |             높음 |

정확한 값은 CPU microarchitecture에 따라 달라집니다.

---

## 8.4 ALU 하나와 여러 개의 ALU

### ALU 하나

```text
Cycle 1: add A
Cycle 2: add B
Cycle 3: add C
```

### ALU 여러 개

독립 명령어라면:

```text
Cycle 1:
ALU 0 → add A
ALU 1 → add B
ALU 2 → add C
```

하지만 dependency가 있다면 여러 ALU가 있어도 소용이 없습니다.

```c
a = a + x;
a = a + y;
a = a + z;
```

```text
첫 번째 결과
   ↓
두 번째 연산
   ↓
세 번째 연산
```

---

## 8.5 Port Pressure

현대 CPU에서는 instruction이 특정 execution port를 사용할 수 있습니다.

예:

```text
Port 0 → Integer ALU
Port 1 → Integer ALU / multiply
Port 2 → Load
Port 3 → Load
Port 4 → Store data
```

특정 종류의 instruction이 한 port에 몰리면 다른 장치가 비어 있어도 처리량이 제한됩니다.

이를 execution-port pressure라고 합니다.

---

## 8.6 주소 계산도 연산이다

배열 접근:

```c
value = array[index];
```

주소는 다음처럼 계산됩니다.

```text
base + index × element_size
```

이 계산을 위해 Address Generation Unit(AGU)을 사용할 수 있습니다.

Load가 많으면 ALU가 아니라 AGU와 load port가 병목이 될 수 있습니다.

---

## 8.7 Multiplexer와 Clock Frequency

MUX를 많이 거칠수록 조합 논리 경로가 길어집니다.

```text
Register
   ↓
MUX
   ↓
MUX
   ↓
ALU
   ↓
MUX
   ↓
Register
```

한 clock cycle 안에 신호가 이 경로를 모두 통과해야 한다면 clock period를 충분히 길게 잡아야 합니다.

```text
Clock period
≥
가장 긴 combinational path delay
```

이 가장 긴 경로를 critical path라고 합니다.

---

## 8.8 Single-Cycle CPU의 성능 한계

모든 instruction을 한 cycle에 완료하는 CPU를 생각해 봅시다.

```text
Fetch
Decode
Register read
ALU
Memory
Write-back
```

Load instruction은 긴 경로를 가집니다.

```text
PC
→ Instruction memory
→ Decode
→ Register file
→ ALU
→ Data memory
→ Register write-back
```

Clock period는 이 가장 느린 instruction에 맞춰야 합니다.

결과:

```text
간단한 ADD도
느린 LOAD와 같은 긴 cycle을 사용
```

이를 개선하기 위해 multi-cycle 또는 pipeline 구조를 사용합니다.

---

## 8.9 간단한 성능 계산

두 datapath 설계를 비교하겠습니다.

### 설계 A: Single-cycle

```text
모든 instruction = 1 cycle
Clock period = 8 ns
```

100개 instruction:

```text
100 × 8 ns = 800 ns
```

### 설계 B: Multi-cycle

평균:

```text
Average CPI = 3
Clock period = 2 ns
```

100개 instruction:

```text
100 × 3 × 2 ns
= 600 ns
```

CPI는 높아졌지만 clock period가 크게 줄어 전체 실행 시간은 개선되었습니다.

따라서:

```text
낮은 CPI만으로 성능을 판단하면 안 된다.
높은 clock frequency만으로도 판단하면 안 된다.
```

---

# 9. 실제 시스템 및 Embedded 응용

## 9.1 ARM Cortex-M의 Register

Cortex-M에서는 일반적으로 다음 register들이 software에 보입니다.

```text
R0-R12
SP
LR
PC
xPSR
```

대표 용도:

```text
R0-R3   → 인수와 임시 값
R4-R11  → 일반 register
R12     → 임시 register
SP      → Stack Pointer
LR      → Link Register
PC      → Program Counter
xPSR    → 상태와 condition flags
```

정확한 보존 규칙은 ABI에 따라 결정됩니다.

---

## 9.2 Memory-Mapped I/O

Embedded에서는 peripheral register가 memory address에 배치됩니다.

예:

```c
#define GPIO_OUT (*(volatile unsigned int *)0x40020014U)

int main(void)
{
    GPIO_OUT = 1U;
    return 0;
}
```

개념적 datapath:

```text
Immediate address 0x40020014
        |
        v
Address-generation datapath
        |
        v
Bus interconnect
        |
        v
GPIO peripheral register
```

이 store는 SRAM에 데이터를 쓰는 것이 아니라 GPIO hardware state를 바꿉니다.

---

## 9.3 MMIO와 일반 Memory의 차이

CPU instruction은 둘 다 store일 수 있습니다.

```asm
str w0, [x1]
```

하지만 주소가 어디에 mapping되어 있느냐에 따라 목적지가 달라집니다.

```text
0x20000000 → SRAM
0x40020014 → GPIO register
```

Address decoder가 대상 hardware를 선택합니다.

```text
CPU address
   |
   v
Bus interconnect
   |
   +── SRAM
   +── UART
   +── GPIO
   +── Timer
```

---

## 9.4 Device Register의 Read-Modify-Write 위험

다음 코드를 생각해 봅시다.

```c
GPIO_OUT |= (1U << 3);
```

개념적으로:

```text
1. GPIO_OUT 읽기
2. OR 연산
3. GPIO_OUT 쓰기
```

즉 하나의 C statement지만 여러 datapath operation입니다.

Peripheral register에 따라 read 값이나 write 의미가 특수할 수 있습니다.

예:

```text
읽으면 interrupt flag가 clear됨
1을 쓰면 해당 bit만 clear됨
일부 bit는 read-only
```

따라서 일반 RAM처럼 read-modify-write하면 오동작할 수 있습니다.

---

## 9.5 Atomic Set/Clear Register

일부 microcontroller는 GPIO용 set/clear register를 별도로 제공합니다.

```text
GPIO_SET:
1을 쓴 bit만 set

GPIO_CLEAR:
1을 쓴 bit만 clear
```

장점:

```text
Read-modify-write 불필요
Interrupt 경쟁 감소
Atomic bit update 가능
```

Datapath 관점에서는 register read와 ALU 연산 없이 store 하나로 끝날 수 있습니다.

---

## 9.6 Compiler Register Allocation

C 변수와 CPU register는 일대일 관계가 아닙니다.

```c
int x = a + b;
int y = x + c;
```

Compiler는 다음과 같이 처리할 수 있습니다.

```text
x를 별도 memory에 저장하지 않음
ALU 결과가 있는 register를 y 계산에 바로 재사용
```

최적화 후에는 C 변수 `x`가 독립된 저장 공간을 갖지 않을 수 있습니다.

---

## 9.7 RTOS Context Switch

Thread를 전환하려면 현재 register state를 저장해야 합니다.

```text
Running task A
   |
   v
Register 저장
   |
   v
Task B의 register 복원
   |
   v
Running task B
```

일반적으로 저장 대상에는 다음이 포함될 수 있습니다.

```text
General-purpose registers
Stack Pointer
Program Counter 또는 return state
Status register
일부 floating-point register
```

Register가 program execution state의 핵심이기 때문에 context switch에서 반드시 다뤄집니다.

---

# 10. 아키텍처 비교

## 10.1 Register-Memory와 Load/Store 구조

### Load/Store Architecture

AArch64:

```asm
ldr w2, [x0]
add w1, w1, w2
```

산술 연산은 register 사이에서 수행됩니다.

### Register-Memory Architecture

x86-64:

```asm
add esi, DWORD PTR [rdi]
```

산술 instruction이 memory operand를 직접 지정할 수 있습니다.

그러나 내부적으로는 여전히 다음 작업이 필요합니다.

```text
Address generation
Memory load
ALU operation
```

---

## 10.2 Hardwired Control과 Microcoded Control

| 항목     | Hardwired Control   | Microcoded Control        |
| ------ | ------------------- | ------------------------- |
| 제어 생성  | 논리 회로               | Microinstruction sequence |
| 속도     | 일반적으로 빠름            | 추가 단계 가능                  |
| 수정 용이성 | 낮음                  | 상대적으로 높음                  |
| 복잡한 명령 | 구현 어려울 수 있음         | 구현하기 쉬움                   |
| 대표 용도  | 규칙적인 기본 instruction | 복잡한 instruction           |

현대 CPU는 둘 중 하나만 사용하는 것이 아니라 혼합할 수 있습니다.

```text
일반 instruction → fast decoder
복잡한 instruction → microcode sequencer
```

---

## 10.3 Single-Bus와 Multi-Bus Datapath

### Single-Bus

```text
Register ─┐
ALU ──────┼── Shared bus
Memory ───┘
```

장점:

```text
구조 단순
배선 감소
면적 절감
```

단점:

```text
한 번에 하나의 주요 transfer
여러 cycle 필요
병목 발생
```

### Multi-Bus

```text
Register A bus ──> ALU input A
Register B bus ──> ALU input B
Result bus     ──> Register write
```

장점:

```text
동시 operand 전달
높은 throughput
```

단점:

```text
배선과 port 증가
Register file 복잡도 증가
면적·전력 증가
```

---

## 10.4 Cortex-M과 고성능 Cortex-A

| 항목                | Cortex-M 계열      | 고성능 Cortex-A 계열      |
| ----------------- | ---------------- | -------------------- |
| 실행 구조             | 단순한 in-order가 흔함 | Out-of-order 가능      |
| Datapath 수        | 제한적              | 여러 병렬 execution path |
| Register renaming | 일반적으로 없음         | 사용 가능                |
| 실행 폭              | 좁음               | 넓은 superscalar       |
| 목표                | 저전력·예측 가능성       | 높은 throughput        |
| 제어 복잡도            | 낮음               | 높음                   |

---

## 10.5 왜 여러 ALU를 무한히 늘리지 않는가?

ALU를 추가하면 항상 성능이 선형 증가하지 않습니다.

제약:

```text
독립 instruction 부족
Register-file read port 부족
Scheduler 복잡도 증가
결과 bypass network 복잡도 증가
전력 증가
회로 면적 증가
Clock frequency 저하 가능
```

실행 장치가 많아질수록 연결망이 복잡해집니다.

```text
여러 source
×
여러 execution unit
×
여러 destination
```

---

# 11. 흔한 오해

## 오해 1: Register는 CPU 안에 있는 일반적인 RAM이다

비슷한 저장 기능을 하지만 목적과 구조가 다릅니다.

Register file은 매우 적은 entry와 다중 port를 제공하며 ALU에 직접 연결됩니다.

```text
Register file
→ 작고 매우 빠름

Main memory
→ 크지만 훨씬 느림
```

---

## 오해 2: ALU는 덧셈과 뺄셈만 한다

ALU는 다음 작업도 수행할 수 있습니다.

```text
Bitwise logic
Comparison
Shift
Address calculation
Condition generation
```

다만 multiply, divide, floating-point, SIMD는 별도 unit으로 구현될 수 있습니다.

---

## 오해 3: Control Unit은 소프트웨어다

Control Unit은 기본적으로 하드웨어 제어 구조입니다.

Microcode를 사용하더라도 일반 application software와는 다른 CPU 내부 제어 계층입니다.

---

## 오해 4: 하나의 C 변수는 하나의 register에 대응한다

Compiler optimization에 따라:

```text
여러 변수가 하나의 register를 시간별로 공유
변수가 stack으로 spill
변수가 완전히 제거
상수로 대체
```

될 수 있습니다.

---

## 오해 5: 하나의 instruction은 하나의 ALU만 사용한다

Load instruction은 주소 계산과 memory access를 필요로 합니다.

복잡한 instruction은 여러 내부 unit과 여러 cycle을 사용할 수 있습니다.

---

## 오해 6: Register 연산은 항상 0 cycle이다

Register read와 ALU 연산에도 latency가 있습니다.

다만 memory hierarchy보다 훨씬 빠르고 pipeline에 맞게 설계됩니다.

---

## 오해 7: ALU가 많으면 성능이 무조건 비례해 증가한다

Dependency와 front-end 공급, register-file port, memory bandwidth가 충분해야 합니다.

---

## 오해 8: `volatile`은 CPU register 사용을 금지한다

그렇지 않습니다.

`volatile` object에 대한 required access를 유지해야 한다는 의미이지, compiler가 어떤 순간에도 register를 사용하지 못한다는 뜻은 아닙니다.

---

# 12. 실패 사례와 디버깅

## 12.1 Register Clobber

Inline assembly에서 변경하는 register를 compiler에게 알리지 않으면 문제가 생길 수 있습니다.

잘못된 개념적 예:

```c
int value = 10;

__asm__("mov $0, %eax");

return value;
```

Compiler가 `eax`에 중요한 값이 있다고 가정했다면 손상될 수 있습니다.

Inline assembly에는 input, output, clobber 규칙을 정확히 지정해야 합니다.

문법은 compiler와 target ISA에 따라 다릅니다.

---

## 12.2 Calling Convention 위반

함수가 보존해야 하는 callee-saved register를 복구하지 않으면 호출자 상태가 손상됩니다.

```text
Caller:
r4에 중요한 값 보관

Callee:
r4를 임시로 변경
복원하지 않음

Return:
Caller의 값 손상
```

Assembly 함수 작성 시 ABI를 따라야 합니다.

---

## 12.3 Stack Pointer 손상

Stack Pointer가 잘못되면 다음 문제가 발생할 수 있습니다.

```text
지역 변수 접근 오류
Return address 손상
Saved register 복원 실패
Alignment fault
```

함수 prologue와 epilogue가 대칭인지 확인해야 합니다.

---

## 12.4 잘못된 MMIO Read-Modify-Write

다음 코드:

```c
*status_register |= 1U;
```

Status register가 write-one-to-clear 방식이라면 의도와 반대로 여러 flag가 지워질 수 있습니다.

확인할 것:

```text
Register access type
Read side effect
Write-one-to-clear
Reserved bit
Atomic set/clear register
```

---

## 12.5 Signed Overflow

ALU 자체는 일정 bit width로 덧셈을 수행합니다.

```text
0x7FFFFFFF + 1
→ 0x80000000
```

하지만 C의 signed overflow는 Undefined Behavior입니다.

```text
Hardware bit result
≠
C 언어가 보장하는 의미
```

Overflow가 가능하면 범위 검사나 unsigned arithmetic을 사용해야 합니다.

---

## 12.6 Register Spill로 인한 예상 밖 성능 저하

작은 함수에 변수와 temporary를 많이 추가한 뒤 성능이 떨어질 수 있습니다.

Disassembly에서 확인할 것:

```text
Stack frame 크기 증가
반복되는 store
반복되는 load
```

원인:

```text
Register pressure
Compiler의 register allocation 실패
Function call로 인한 register 보존
```

---

## 12.7 잘못된 Operand Width

x86-64에서:

```text
eax = 32-bit
rax = 64-bit
```

AArch64에서:

```text
w0 = 32-bit
x0 = 64-bit
```

Pointer를 32-bit register로 잘못 처리하면 상위 주소 bit가 손실될 수 있습니다.

---

## 12.8 Condition Flag 오염

일부 ISA에서는 arithmetic instruction이 condition flag를 갱신할 수 있습니다.

Branch 전에 다른 flag-setting instruction이 실행되면 조건이 달라질 수 있습니다.

```asm
cmp r0, r1
adds r2, r2, #1
beq target
```

`beq`가 어느 instruction이 만든 flag를 사용하는지 확인해야 합니다.

---

# 13. 확인 문제

정답은 바로 공개하지 않습니다.

## Level 1: 개념 확인

### 문제 1

다음 네 요소를 각각 한 문장으로 설명하세요.

```text
Register file
ALU
Control Unit
Datapath
```

그리고 datapath와 control path의 차이를 설명하세요.

---

### 문제 2

Multiplexer가 CPU datapath에 필요한 이유를 설명하세요.

다음 예를 포함하세요.

```text
ALU의 두 번째 입력으로
register 또는 immediate를 선택하는 경우
```

---

## Level 2: 동작 과정 추적

### 문제 3

다음 AArch64 명령어가 실행됩니다.

```asm
add w3, w1, w2
```

초기 상태:

```text
w1 = 14
w2 = 9
w3 = 100
```

다음을 순서대로 설명하세요.

```text
Register read address
Register read data
ALU control
ALU result
Write address
Write data
RegWrite 값
```

---

### 문제 4

다음 명령어를 datapath 관점에서 추적하세요.

```asm
ldr w2, [x0, #8]
```

초기 상태:

```text
x0 = 0x4000
Memory[0x4008] = 55
```

다음을 구하세요.

```text
ALU input A
ALU input B
Effective address
Memory output
Destination register
필요한 주요 control signal
```

---

## Level 3: 성능 계산 또는 코드 분석

### 문제 5

두 CPU 설계를 비교합니다.

설계 A:

```text
CPI = 1
Clock period = 9 ns
```

설계 B:

```text
Average CPI = 3.2
Clock period = 2 ns
```

프로그램은 500개의 instruction을 실행합니다.

각 실행 시간을 계산하고 더 빠른 설계를 고르세요.

---

### 문제 6

다음 두 코드 중 register pressure가 더 커질 가능성이 높은 코드를 고르고 이유를 설명하세요.

코드 A:

```c
int result = a + b;
return result * 2;
```

코드 B:

```c
int r0 = a + b;
int r1 = c + d;
int r2 = e + f;
int r3 = g + h;
return r0 + r1 + r2 + r3;
```

Live value와 spill 가능성을 포함해 설명하세요.

---

## Level 4: 설계·디버깅·실무 응용

### 문제 7

GPIO output register의 특정 bit를 설정하기 위해 다음 코드를 사용했습니다.

```c
GPIO_OUT |= (1U << 5);
```

하지만 interrupt handler도 같은 register의 다른 bit를 변경합니다.

발생 가능한 read-modify-write race를 설명하고, 가능한 hardware 또는 software 해결 방법을 제시하세요.

---

### 문제 8

직접 작성한 assembly 함수가 가끔 호출자의 변수 값을 손상시킵니다.

Register, ABI, caller-saved, callee-saved 관점에서 가능한 원인과 디버깅 순서를 설명하세요.

---

# 14. 실습 과제

## 실습 목표

C 변수와 CPU register가 일대일로 대응하지 않는다는 것을 assembly로 확인합니다.

또한 변수가 많아질 때 register spill과 stack access가 어떻게 나타나는지 관찰합니다.

---

## 준비물

```text
GCC 또는 Clang
macOS 또는 Linux
Assembly를 확인할 수 있는 text editor
```

---

## 코드

파일 이름: `register_pressure.c`

```c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

__attribute__((noinline))
static int low_pressure(int a, int b)
{
    const int sum = a + b;
    return sum * 2;
}

__attribute__((noinline))
static int higher_pressure(
    int a,
    int b,
    int c,
    int d,
    int e,
    int f,
    int g,
    int h
)
{
    const int r0 = a + b;
    const int r1 = c + d;
    const int r2 = e + f;
    const int r3 = g + h;

    return r0 + r1 + r2 + r3;
}

static int parse_int(const char *text, int *result)
{
    char *end = NULL;

    errno = 0;
    const long value = strtol(text, &end, 10);

    if (errno == ERANGE || value < -1000000L || value > 1000000L) {
        return -1;
    }

    if (text[0] == '\0' || *end != '\0') {
        return -1;
    }

    *result = (int)value;
    return 0;
}

int main(int argc, char **argv)
{
    if (argc != 9) {
        fprintf(
            stderr,
            "Usage: %s a b c d e f g h\n",
            argv[0]
        );
        return EXIT_FAILURE;
    }

    int values[8];

    for (int i = 0; i < 8; ++i) {
        if (parse_int(argv[i + 1], &values[i]) != 0) {
            fprintf(stderr, "Invalid integer: %s\n", argv[i + 1]);
            return EXIT_FAILURE;
        }
    }

    const int low = low_pressure(values[0], values[1]);

    const int high = higher_pressure(
        values[0],
        values[1],
        values[2],
        values[3],
        values[4],
        values[5],
        values[6],
        values[7]
    );

    if (printf("low=%d, high=%d\n", low, high) < 0) {
        fprintf(stderr, "Failed to write output.\n");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

입력 범위를 제한했지만 여러 합의 최종 결과가 `int` 범위를 넘지 않도록 작은 값을 사용하세요.

---

## 컴파일 및 실행

```bash
cc -O0 -Wall -Wextra -Wpedantic register_pressure.c -o pressure_O0
cc -O2 -Wall -Wextra -Wpedantic register_pressure.c -o pressure_O2

./pressure_O0 1 2 3 4 5 6 7 8
./pressure_O2 1 2 3 4 5 6 7 8
```

예상 출력:

```text
low=6, high=36
```

---

## Assembly 생성

```bash
cc -O0 -S register_pressure.c -o pressure_O0.s
cc -O2 -S register_pressure.c -o pressure_O2.s
```

함수 확인:

```bash
grep -n -A30 "low_pressure" pressure_O0.s
grep -n -A40 "higher_pressure" pressure_O0.s

grep -n -A30 "low_pressure" pressure_O2.s
grep -n -A40 "higher_pressure" pressure_O2.s
```

macOS에서는 symbol 앞에 `_`가 붙을 수 있습니다.

---

## 예상 관찰 결과

### `-O0`

```text
함수 인수와 지역 변수를 stack에 자주 저장
많은 load/store
큰 stack frame
C 변수 구조가 상대적으로 잘 보임
```

### `-O2`

```text
값을 register에 유지
불필요한 지역 변수 제거
연산 순서 변경 가능
일부 add를 결합
stack access 감소
```

---

## 관찰할 항목

1. 함수 인수가 어떤 register로 전달되는가?
2. Stack에 저장되는 인수가 있는가?
3. 지역 변수 `r0`, `r1`, `r2`, `r3`가 실제 memory 공간을 갖는가?
4. `-O2`에서 register 이름이 어떻게 재사용되는가?
5. 함수의 stack frame 크기가 달라지는가?

---

## 추가 실습: 강제로 Live Value 늘리기

다음 함수를 추가하세요.

```c
__attribute__((noinline))
static long many_live_values(
    long a0,
    long a1,
    long a2,
    long a3,
    long a4,
    long a5,
    long a6,
    long a7,
    long a8,
    long a9,
    long a10,
    long a11
)
{
    const long r0 = a0 + 1;
    const long r1 = a1 + 2;
    const long r2 = a2 + 3;
    const long r3 = a3 + 4;
    const long r4 = a4 + 5;
    const long r5 = a5 + 6;
    const long r6 = a6 + 7;
    const long r7 = a7 + 8;
    const long r8 = a8 + 9;
    const long r9 = a9 + 10;
    const long r10 = a10 + 11;
    const long r11 = a11 + 12;

    return r0 + r1 + r2 + r3 + r4 + r5
         + r6 + r7 + r8 + r9 + r10 + r11;
}
```

확인할 것:

```text
일부 인수가 stack으로 전달되는가?
중간값이 stack에 spill되는가?
Compiler가 연산 순서를 바꾸어 live range를 줄이는가?
```

---

## 잘못된 결과가 나올 수 있는 원인

```text
Compiler가 예상보다 강하게 최적화
함수가 호출되지 않아 제거됨
값이 상수로 전파됨
ABI 차이
Target architecture 차이
입력값에 따른 signed overflow
```

---

## macOS 대안

Apple Silicon에서 generated assembly를 볼 때:

```bash
clang -O2 -S register_pressure.c -o pressure_arm64.s
```

실행 파일 disassembly:

```bash
otool -tvV pressure_O2
```

Linux:

```bash
objdump -d -Mintel pressure_O2
```

ARM board:

```bash
objdump -d pressure_O2
```

Cross-compiled binary라면 target용 `objdump`를 사용합니다.

---

# 15. 면접에서 설명하는 방법

## 15.1 30초 설명

논리 구조:

```text
Register
→ ALU
→ Datapath
→ Control Unit
```

예시:

> Register file은 CPU 내부의 operand와 중간 결과를 저장하고 여러 read/write port를 제공합니다. ALU는 register에서 읽은 값을 이용해 산술·논리·주소 계산을 수행합니다. Datapath는 register, ALU, memory interface와 multiplexer를 연결해 실제 값이 이동하는 경로이고, Control Unit은 instruction을 decode해 ALU operation, memory access, register write-back, PC 선택 같은 제어 신호를 생성합니다.

---

## 15.2 2분 설명

답변 구조:

1. Register file의 역할
2. ALU의 역할
3. Multiplexer와 datapath
4. Control signal
5. Load와 arithmetic 비교
6. 현대 CPU의 확장

예시:

> CPU의 datapath는 instruction을 실행하기 위해 데이터가 이동하고 변환되는 경로입니다. Register file은 source operand를 여러 read port로 제공하고 destination result를 write port로 저장합니다. ALU는 덧셈이나 bitwise operation뿐 아니라 effective-address 계산과 비교도 수행합니다. 같은 ALU가 register operand, immediate, PC-relative offset 등을 처리하려면 multiplexer가 입력 source를 선택합니다. Control Unit은 opcode를 decode해 ALUControl, RegWrite, MemRead, MemWrite, ResultSrc와 같은 신호를 생성합니다. 예를 들어 add instruction은 두 register 값을 ALU에 넣고 결과를 register에 기록하지만, load instruction은 ALU를 주소 계산에 사용한 뒤 memory output을 register에 기록합니다. 현대 superscalar CPU에서는 이런 datapath가 여러 개 존재하며 scheduler와 register renaming 구조가 execution unit에 작업을 분배합니다.

---

## 15.3 심화 꼬리 질문

```text
Register file에 read/write port가 많아지면 어떤 비용이 생기는가?
Single-cycle datapath의 critical path는 무엇인가?
Load instruction에서 ALU가 필요한 이유는 무엇인가?
Hardwired control과 microcode의 차이는 무엇인가?
Register spill은 왜 발생하는가?
Address Generation Unit은 일반 ALU와 무엇이 다른가?
Result forwarding은 어떤 datapath를 추가하는가?
MMIO store와 SRAM store를 CPU는 어떻게 구분하는가?
```

좋은 답변에서는 다음 계층을 구분해야 합니다.

```text
Architectural register
Physical register
Register file
Datapath
Control signal
Memory-mapped device
Compiler register allocation
```

---

# 16. 핵심 정리

1. **Register는 CPU 내부 상태와 operand를 저장하는 소형 고속 저장 요소입니다.**

2. **Register file은 여러 register를 register 번호로 읽고 쓰게 하는 구조입니다.**

3. **ALU는 산술뿐 아니라 logic, comparison, shift, 주소 계산도 수행합니다.**

4. **Datapath는 값이 저장되고 이동하며 계산되는 하드웨어 경로입니다.**

5. **Control path는 datapath에 어떤 연산과 경로를 선택할지 지시합니다.**

6. Multiplexer는 여러 입력 중 하나를 선택하여 hardware를 공유할 수 있게 합니다.

7. `add` 명령어:

   ```text
   Register → ALU → Register
   ```

8. `load` 명령어:

   ```text
   Register + offset
   → ALU 주소 계산
   → Memory read
   → Register
   ```

9. `store` 명령어:

   ```text
   Register + offset
   → ALU 주소 계산
   → Memory write
   ```

10. Branch는 condition 계산과 next-PC 선택이 필요합니다.

11. ALU 하나를 공유하면 면적과 전력이 줄지만, 병렬 실행 능력이 제한됩니다.

12. 현대 CPU는 여러 ALU, AGU, branch unit, vector unit을 병렬로 사용합니다.

13. Register 수가 부족하면 compiler가 값을 stack으로 spill하여 load/store가 증가합니다.

14. 한 C 변수와 하나의 CPU register는 일대일 대응하지 않습니다.

15. `volatile`은 atomicity나 thread synchronization을 보장하지 않습니다.

16. Single-cycle CPU는 가장 느린 instruction 경로가 clock period를 결정합니다.

17. CPU 성능은 다음을 함께 봐야 합니다.

```text
CPI
Clock period
ALU latency
Throughput
Register pressure
Port pressure
Memory access
```