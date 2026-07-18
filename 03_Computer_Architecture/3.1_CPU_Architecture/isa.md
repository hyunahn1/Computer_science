# Lecture 3. ISA, Microarchitecture, RISC와 CISC

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> 같은 프로그램을 실행하는 CPU라도 내부 구조가 서로 다를 수 있는 이유는 무엇인가?

핵심 질문을 더 나누면 다음과 같습니다.

1. ISA와 Microarchitecture는 정확히 무엇이 다른가?
2. 같은 x86-64 프로그램이 Intel CPU와 AMD CPU에서 모두 실행될 수 있는 이유는 무엇인가?
3. RISC는 정말 명령어가 적고, CISC는 정말 명령어가 많은 구조인가?
4. AArch64와 x86-64의 명령어 차이는 실제 CPU 내부 복잡도와 어떤 관계가 있는가?
5. 명령어 하나가 하드웨어 내부에서 반드시 하나의 연산으로 실행되는가?
6. Compiler는 왜 target ISA와 target CPU를 따로 알아야 하는가?

이번 Lecture는 업로드한 학습 지침의 Part 1 범위를 기준으로 진행합니다. 

---

## 2. 이전 Lecture와의 연결

Lecture 1에서는 CPU가 machine instruction을 실행한다는 것을 배웠습니다.

```text
C/C++ source
   ↓ compiler
Machine instructions
   ↓
CPU execution
```

Lecture 2에서는 instruction과 data가 memory system을 통해 CPU로 전달되는 구조를 살펴봤습니다.

```text
Instruction path
Data path
Memory hierarchy
```

하지만 아직 중요한 질문이 남아 있습니다.

다음 두 CPU를 생각해 봅시다.

```text
Intel Core CPU
AMD Ryzen CPU
```

두 프로세서는 내부 설계가 다르지만, 같은 x86-64 프로그램을 실행할 수 있습니다.

반대로 다음 CPU들은 내부적으로 모두 현대적인 고성능 processor일 수 있지만:

```text
Intel Core
Apple Silicon
ARM Cortex-A
```

서로 다른 machine code를 사용합니다.

그 이유를 이해하려면 다음 두 계층을 구분해야 합니다.

```text
ISA
Microarchitecture
```

---

# 3. 직관적 설명

## 3.1 ISA는 규칙서다

ISA를 보드게임의 공식 규칙으로 생각해 봅시다.

규칙서에는 다음이 적혀 있습니다.

```text
말의 종류
말이 이동하는 방법
가능한 행동
점수 계산 방법
게임 종료 조건
```

이 규칙을 지키는 한, 실제 게임판의 재질이나 내부 제작 방식은 달라도 됩니다.

```text
나무 게임판
플라스틱 게임판
전자식 게임판
```

CPU에서도 ISA는 소프트웨어와 하드웨어 사이의 규칙서 역할을 합니다.

ISA는 다음을 정의합니다.

```text
어떤 instruction이 존재하는가?
각 instruction은 어떤 결과를 내야 하는가?
어떤 register가 보이는가?
Memory access는 어떻게 표현하는가?
Function call과 exception은 어떤 상태를 사용하는가?
```

---

## 3.2 Microarchitecture는 규칙을 구현하는 방법이다

같은 규칙의 게임을 구현하더라도 내부 방식은 다를 수 있습니다.

예를 들어 전자 체스판 두 개가 같은 체스 규칙을 지원한다고 합시다.

```text
제품 A:
단순한 순차 처리

제품 B:
여러 수를 미리 계산
가능한 경로를 예측
병렬 처리
```

사용자에게 보이는 체스 규칙은 같습니다.

그러나 내부 구현과 성능은 다릅니다.

CPU에서도 마찬가지입니다.

```text
ISA:
add instruction은 두 값을 더해 결과를 저장해야 한다.

Microarchitecture A:
단순 ALU 하나로 순차 실행

Microarchitecture B:
여러 ALU
Out-of-order scheduler
Register renaming
Speculative execution
```

둘 다 ISA가 요구한 최종 결과를 내면 올바른 구현입니다.

---

## 3.3 RISC와 CISC의 직관

RISC와 CISC를 배송 명령 체계로 비유하겠습니다.

### RISC식 명령

작업이 비교적 단순하고 규칙적입니다.

```text
상자 가져오기
상자 열기
물건 꺼내기
물건 옮기기
```

하나의 명령이 비교적 작은 작업을 합니다.

### CISC식 명령

하나의 명령이 더 복잡한 작업을 표현할 수 있습니다.

```text
창고 3번에서 상자를 가져와 열고,
두 번째 물건을 계산대까지 옮겨라.
```

명령 수는 줄어들 수 있지만, 명령 자체를 해석하고 실행하는 과정은 복잡해질 수 있습니다.

---

## 3.4 비유의 한계

현대 CPU에서는 RISC와 CISC의 경계가 단순하지 않습니다.

x86-64는 CISC 계열 ISA로 분류되지만 내부에서는 명령어를 더 단순한 micro-operation으로 분해할 수 있습니다.

```text
Complex x86 instruction
        ↓ decode
One or more micro-operations
        ↓
RISC-like execution engine
```

반대로 AArch64는 RISC 계열이지만 현대 고성능 AArch64 CPU의 내부 구조는 매우 복잡합니다.

```text
Wide decode
Register renaming
Out-of-order execution
Branch prediction
Multiple cache levels
Speculation
```

따라서 다음은 잘못된 단순화입니다.

```text
RISC = 단순한 CPU
CISC = 복잡한 CPU
```

정확한 구분은 ISA의 설계 철학과 encoding 특성을 중심으로 해야 합니다.

---

# 4. 형식적 정의

## 4.1 ISA

**용어:** 명령어 집합 구조(Instruction Set Architecture, ISA)

**정의:**
소프트웨어가 관찰할 수 있는 processor의 동작과 프로그래밍 모델을 정의하는 명세입니다.

**필요한 이유:**
Compiler, operating system, application과 CPU hardware가 동일한 machine instruction의 의미에 합의해야 하기 때문입니다.

**하드웨어에서의 역할:**
CPU 구현은 ISA에 정의된 instruction과 architectural state transition을 올바르게 수행해야 합니다.

**관련된 다른 개념:**

```text
Machine code
Assembly
Register
Calling convention
Exception
Memory model
Privilege level
ABI
```

ISA가 정의할 수 있는 주요 요소:

```text
Instruction encoding
Register set
Data types
Addressing modes
Memory access instructions
Branch and control flow
Exception behavior
Privilege levels
Atomic operations
Memory ordering
```

---

## 4.2 Microarchitecture

**용어:** 마이크로아키텍처(Microarchitecture)

**정의:**
특정 ISA를 실제 hardware로 구현하는 내부 구조와 동작 방식입니다.

**필요한 이유:**
ISA의 의미를 만족하면서 성능, 전력, 면적, 비용 목표를 달성하기 위해 필요합니다.

**하드웨어에서의 역할:**
Instruction fetch, decode, scheduling, execution, cache, branch prediction, retirement 등을 구체적으로 구현합니다.

**관련된 다른 개념:**

```text
Pipeline
Superscalar
Out-of-order execution
Execution unit
Cache hierarchy
Branch predictor
Register renaming
Reorder buffer
```

예를 들어 x86-64 ISA를 구현하는 microarchitecture는 여러 가지가 있습니다.

```text
Intel Skylake
Intel Golden Cove
AMD Zen
AMD Zen 4
```

이들은 모두 같은 계열의 x86-64 machine code를 실행할 수 있지만 내부 구조와 성능은 다릅니다.

---

## 4.3 Architecture와 Implementation

다음 세 계층을 구분해야 합니다.

| 계층                      | 질문                                | 예                       |
| ----------------------- | --------------------------------- | ----------------------- |
| ISA                     | 소프트웨어가 어떤 instruction을 사용할 수 있는가? | x86-64, AArch64, RISC-V |
| Microarchitecture       | ISA를 내부에서 어떻게 실행하는가?              | Zen, Cortex-A76         |
| Physical implementation | 어떤 공정·주파수·cache 크기로 제작하는가?        | 5 nm, 3 GHz, 32 KB L1   |

같은 microarchitecture도 physical implementation이 달라질 수 있습니다.

```text
동일한 core design
+
다른 clock frequency
다른 cache size
다른 process node
다른 core count
```

---

## 4.4 Instruction Encoding

**용어:** 명령어 인코딩(Instruction Encoding)

**정의:**
Instruction의 opcode, register 번호, immediate 값 등을 bit field로 표현하는 방식입니다.

**필요한 이유:**
CPU decoder가 instruction의 의미를 비트 수준에서 구분해야 하기 때문입니다.

**하드웨어에서의 역할:**
Instruction decoder가 bit field를 읽어 내부 제어 정보 또는 micro-operation을 생성합니다.

**관련된 다른 개념:**

```text
Fixed-length instruction
Variable-length instruction
Opcode
Operand
Immediate
Decoder
```

---

## 4.5 RISC

**용어:** 축소 명령어 집합 컴퓨터(Reduced Instruction Set Computer, RISC)

**정의:**
비교적 규칙적이고 단순한 instruction 형식, load/store 중심의 memory access, 많은 general-purpose register를 강조하는 ISA 설계 철학입니다.

**필요한 이유:**
Instruction decode와 pipeline 구현을 단순화하고 compiler가 명시적인 instruction 조합을 생성하기 쉽게 하기 위해 발전했습니다.

**하드웨어에서의 역할:**
고정 길이 또는 규칙적인 encoding과 단순한 operand 구조를 통해 빠른 decode와 pipeline 설계를 지원할 수 있습니다.

**관련된 다른 개념:**

```text
Load/store architecture
Fixed-length encoding
Register-register operation
ARM
AArch64
RISC-V
MIPS
```

RISC의 일반적인 특징:

```text
Arithmetic은 주로 register 사이에서 수행
Memory 접근은 load/store instruction으로 분리
규칙적인 instruction encoding
상대적으로 단순한 addressing mode
```

---

## 4.6 CISC

**용어:** 복합 명령어 집합 컴퓨터(Complex Instruction Set Computer, CISC)

**정의:**
다양한 instruction 형식, 복잡한 addressing mode, 하나의 instruction에서 여러 작업을 표현할 수 있는 ISA 설계 계열입니다.

**필요한 이유:**
Assembly program의 instruction 수와 code size를 줄이고, 고수준 작업을 instruction 하나로 표현하기 위해 발전했습니다.

**하드웨어에서의 역할:**
복잡한 instruction decoder, microcode 또는 micro-operation 분해가 필요할 수 있습니다.

**관련된 다른 개념:**

```text
Variable-length instruction
Complex addressing mode
Microcode
x86
x86-64
```

---

## 4.7 Micro-operation

**용어:** 마이크로 연산(Micro-operation, µop)

**정의:**
CPU가 architectural instruction을 내부적으로 실행하기 위해 사용하는 더 작은 내부 작업 단위입니다.

**필요한 이유:**
복잡한 ISA instruction을 현대적인 execution engine이 처리하기 쉬운 형태로 변환하기 위해 사용합니다.

**하드웨어에서의 역할:**
Scheduler, execution port, ALU, load/store unit 등에서 실행되는 내부 작업이 됩니다.

**관련된 다른 개념:**

```text
Decode
Microcode
Macro-fusion
Micro-fusion
Instruction retirement
```

Micro-operation은 ISA에 직접 보이지 않습니다.

```text
Application
  ↓
x86 instruction을 봄

CPU 내부
  ↓
µop을 사용하여 실행
```

---

## 4.8 ABI

**용어:** 응용 프로그램 이진 인터페이스(Application Binary Interface, ABI)

**정의:**
같은 ISA 위에서 binary module이 서로 호환되기 위한 호출 규약과 데이터 배치 규칙입니다.

**필요한 이유:**
서로 다른 compiler가 만든 object file, library, operating system code가 함께 동작해야 하기 때문입니다.

**하드웨어에서의 역할:**
직접적인 hardware 명세는 아니지만 register와 stack 사용 방법을 결정하여 생성되는 instruction에 영향을 줍니다.

**관련된 다른 개념:**

```text
Calling convention
Stack layout
Register usage
Object file
Dynamic linking
System call
```

ISA와 ABI는 다릅니다.

```text
ISA:
CPU가 이해하는 instruction과 architectural state

ABI:
소프트웨어가 register와 stack을 사용하는 약속
```

---

# 5. 내부 구조와 동작 원리

## 5.1 ISA 관점의 단순한 명령어

다음 AArch64 instruction을 생각해 봅시다.

```asm
add w0, w1, w2
```

ISA 관점의 의미:

```text
w0 ← lower_32_bits(w1 + w2)
```

ISA는 결과가 어떻게 계산되어야 하는지를 정의합니다.

하지만 다음은 보통 ISA가 강제하지 않습니다.

```text
어떤 ALU를 사용하는가?
몇 cycle이 걸리는가?
Pipeline은 몇 stage인가?
Out-of-order로 실행하는가?
몇 개의 instruction을 동시에 실행하는가?
```

---

## 5.2 단순 Microarchitecture 구현

가장 단순한 구현은 다음과 같을 수 있습니다.

```text
Instruction
    |
    v
+--------+
| Decode |
+--------+
    |
    v
+-----------+
| Register  |
|   File    |
+-----------+
    |
    v
+--------+
|  ALU   |
+--------+
    |
    v
+-----------+
| Writeback |
+-----------+
```

`add` 실행:

```text
1. w1 읽기
2. w2 읽기
3. ALU에 두 값 전달
4. ADD 연산 수행
5. 결과를 w0에 기록
```

---

## 5.3 고성능 Microarchitecture 구현

현대 processor에서는 같은 instruction이 다음 흐름을 통과할 수 있습니다.

```text
Instruction fetch
      |
      v
Branch prediction
      |
      v
Instruction decode
      |
      v
Register renaming
      |
      v
Dispatch
      |
      v
Scheduler / Issue queue
      |
      v
Execution unit
      |
      v
Reorder buffer
      |
      v
Retirement
```

ISA 관점에서는 단순한 덧셈이지만, 내부적으로는 많은 구조를 거칩니다.

---

## 5.4 x86 instruction의 내부 분해

다음 x86-64 instruction을 가정하겠습니다.

```asm
add eax, DWORD PTR [rdi]
```

의미:

```text
eax ← eax + Memory[rdi]
```

이 instruction은 개념적으로 두 작업을 포함합니다.

```text
1. Memory[rdi] load
2. eax와 load된 값을 add
```

내부 µop으로 분해하면 개념적으로 다음과 같을 수 있습니다.

```text
µop 1: load temp, [rdi]
µop 2: add eax, eax, temp
```

실제 µop 수와 fusion 여부는 microarchitecture마다 다를 수 있습니다.

따라서 assembly instruction 수와 내부 작업 수는 동일하지 않을 수 있습니다.

---

## 5.5 Microcode

일부 복잡한 instruction은 일반 decoder만으로 직접 변환하지 않고 microcode engine을 사용할 수 있습니다.

```text
Complex instruction
       |
       v
Microcode sequencer
       |
       v
Sequence of internal µops
```

예를 들어 매우 복잡하거나 드물게 사용되는 instruction은 여러 내부 단계를 거칠 수 있습니다.

Microcode는 다음과 같은 목적으로 사용될 수 있습니다.

```text
복잡한 instruction 구현
예외적인 instruction 처리
CPU 동작 수정
일부 hardware errata 완화
```

Microcode는 일반 application이 직접 보는 instruction set과 다릅니다.

---

## 5.6 Fixed-Length와 Variable-Length Decode

### AArch64

AArch64 instruction은 일반적으로 32비트 고정 길이입니다.

```text
Address 0x1000 → instruction 1
Address 0x1004 → instruction 2
Address 0x1008 → instruction 3
```

Instruction boundary를 찾기 쉽습니다.

```text
PC + 4
```

### x86-64

x86 instruction은 가변 길이입니다.

```text
1 byte ~ 최대 15 bytes
```

Memory byte stream에서 instruction boundary를 찾아야 합니다.

```text
Prefix
Opcode
ModR/M
SIB
Displacement
Immediate
```

개념적으로:

```text
Byte stream
   |
   v
Instruction length decoder
   |
   v
Instruction boundaries
   |
   v
Full decoder
```

이 때문에 x86 front end는 복잡해질 수 있습니다.

---

## 5.7 Decode Width

현대 CPU는 cycle당 여러 instruction을 decode할 수 있습니다.

```text
Cycle N:

Inst 1 ─┐
Inst 2 ─┼─> Multiple decoders
Inst 3 ─┤
Inst 4 ─┘
```

고정 길이 instruction은 boundary를 찾기 상대적으로 쉽습니다.

가변 길이 instruction은 먼저 각 instruction이 어디서 끝나는지 알아야 합니다.

그러나 현대 x86 CPU는 다음 기술로 이를 보완합니다.

```text
Predecoder
µop cache
Decoded instruction cache
Multiple parallel decoders
Instruction queue
```

따라서 ISA가 복잡하다고 해서 반드시 실제 성능이 낮은 것은 아닙니다.

---

## 5.8 Architectural Register와 Physical Register

ISA는 software에 보이는 register를 정의합니다.

AArch64 예:

```text
x0 ~ x30
SP
PC와 system state
```

하지만 실제 out-of-order CPU는 더 많은 내부 physical register를 사용할 수 있습니다.

```text
Architectural register x0
        ↓ rename
Physical register P37
```

이 구조는 이후 register renaming Lecture에서 자세히 다룹니다.

중요한 점은 다음과 같습니다.

```text
ISA register 수
≠
CPU 내부 실제 storage entry 수
```

---

# 6. 단계별 실행 추적

다음 C 함수를 사용하겠습니다.

```c
int sum_element(const int *array, int index)
{
    return array[index] + 5;
}
```

---

## 6.1 AArch64 예시

AArch64 ABI에서 개념적으로:

```text
x0 = array pointer
w1 = index
return value = w0
```

가능한 assembly:

```asm
sum_element:
    ldr w0, [x0, w1, sxtw #2]
    add w0, w0, #5
    ret
```

### 첫 번째 instruction

```asm
ldr w0, [x0, w1, sxtw #2]
```

의미:

```text
sign-extend w1 to 64-bit
index × 4
x0 + index × 4
해당 주소에서 32-bit load
결과를 w0에 저장
```

---

## 6.2 AArch64 내부 추적

초기 상태:

```text
x0 = 0x10000
w1 = 3
Memory[0x1000C] = 37
```

주소 계산:

```text
element size = 4 bytes
offset = 3 × 4 = 12 = 0xC

effective address
= 0x10000 + 0xC
= 0x1000C
```

Load:

```text
w0 ← Memory[0x1000C]
w0 = 37
```

Add:

```text
w0 ← 37 + 5
w0 = 42
```

Return:

```text
PC ← x30
```

---

## 6.3 x86-64 예시

System V x86-64 ABI에서 개념적으로:

```text
rdi = array pointer
esi = index
return value = eax
```

가능한 assembly:

```asm
sum_element:
    movsxd rsi, esi
    mov eax, DWORD PTR [rdi + rsi*4]
    add eax, 5
    ret
```

Compiler에 따라 sign extension과 addressing을 다르게 구성할 수 있습니다.

---

## 6.4 x86-64 내부 µop 관점

다음 instruction:

```asm
mov eax, DWORD PTR [rdi + rsi*4]
```

개념적 내부 작업:

```text
1. Effective address 계산
2. Data cache load
3. 결과를 physical register에 기록
```

다음 instruction:

```asm
add eax, 5
```

개념적 내부 작업:

```text
1. eax에 대응하는 physical register 읽기
2. Immediate 5와 integer ALU 연산
3. 새 physical register에 결과 기록
```

Architectural state에서는 `eax`가 갱신된 것처럼 보이지만, 내부에서는 rename을 통해 새로운 physical register를 사용할 수 있습니다.

---

## 6.5 두 ISA의 결과 비교

AArch64와 x86-64 assembly는 다르지만 최종 C 의미는 같습니다.

```text
return array[index] + 5;
```

공통 동작:

```text
index를 byte offset으로 변환
base address에 더함
memory에서 int load
5를 더함
return register에 저장
```

ISA가 다르면 표현 방식은 달라지지만, compiler는 같은 언어 의미를 각 ISA에 맞게 변환합니다.

---

# 7. C/C++ 및 Assembly 예제

## 7.1 실행 가능한 C 코드

파일 이름: `isa_demo.c`

```c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

__attribute__((noinline))
static int sum_element(const int *array, int index)
{
    return array[index] + 5;
}

static int parse_index(const char *text, int *result)
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
    static const int values[] = {
        10, 20, 30, 37, 50, 60
    };

    const int element_count =
        (int)(sizeof(values) / sizeof(values[0]));

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <index>\n", argv[0]);
        return EXIT_FAILURE;
    }

    int index = 0;

    if (parse_index(argv[1], &index) != 0) {
        fprintf(stderr, "Invalid index: %s\n", argv[1]);
        return EXIT_FAILURE;
    }

    if (index < 0 || index >= element_count) {
        fprintf(
            stderr,
            "Index must be between 0 and %d.\n",
            element_count - 1
        );
        return EXIT_FAILURE;
    }

    const int result = sum_element(values, index);

    if (printf(
            "values[%d] + 5 = %d\n",
            index,
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
cc -O2 -Wall -Wextra -Wpedantic isa_demo.c -o isa_demo
./isa_demo 3
```

예상 출력:

```text
values[3] + 5 = 42
```

---

## 7.3 현재 CPU용 Assembly 생성

macOS 또는 Linux:

```bash
cc -O2 -S isa_demo.c -o isa_demo.s
```

함수 찾기:

```bash
grep -n -A15 "sum_element" isa_demo.s
```

macOS에서는 `_sum_element`로 표시될 수 있습니다.

---

## 7.4 LLVM을 이용한 Cross-Target 예시

Clang이 적절한 target을 지원한다면 assembly만 생성할 수 있습니다.

AArch64:

```bash
clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -S \
  -ffreestanding \
  -fno-stack-protector \
  isa_demo_kernel.c \
  -o isa_demo_aarch64.s
```

x86-64:

```bash
clang \
  --target=x86_64-linux-gnu \
  -O2 \
  -S \
  -ffreestanding \
  -fno-stack-protector \
  isa_demo_kernel.c \
  -o isa_demo_x86_64.s
```

다만 전체 예제에는 standard library가 포함되어 있어 cross-compilation 시 target용 header와 sysroot가 필요할 수 있습니다.

따라서 assembly 비교용으로 다음처럼 작은 파일을 따로 만드는 것이 좋습니다.

`isa_demo_kernel.c`:

```c
__attribute__((noinline))
int sum_element(const int *array, int index)
{
    return array[index] + 5;
}
```

AArch64 assembly 생성:

```bash
clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -S \
  isa_demo_kernel.c \
  -o kernel_aarch64.s
```

x86-64 assembly 생성:

```bash
clang \
  --target=x86_64-linux-gnu \
  -O2 \
  -S \
  isa_demo_kernel.c \
  -o kernel_x86_64.s
```

---

## 7.5 `-march`와 `-mtune`

Compiler option에는 중요한 차이가 있습니다.

### `-march`

사용 가능한 ISA extension을 결정합니다.

예:

```bash
gcc -O2 -march=x86-64-v3 program.c
```

이 옵션은 compiler가 특정 instruction extension을 사용하도록 허용할 수 있습니다.

지원하지 않는 CPU에서 실행하면 illegal instruction이 발생할 수 있습니다.

### `-mtune`

Instruction selection과 scheduling을 특정 microarchitecture에 최적화하지만, 기본적으로 ISA 호환 범위를 반드시 넓히지는 않습니다.

예:

```bash
gcc -O2 -march=x86-64 -mtune=znver4 program.c
```

의미:

```text
사용 instruction:
기본 x86-64 범위

성능 최적화 기준:
AMD Zen 계열 특성 고려
```

정확한 지원 값은 compiler 버전에 따라 달라집니다.

---

# 8. 성능과 메모리 관점

## 8.1 Instruction Count만으로 성능을 판단할 수 없는 이유

다음 두 프로그램을 가정하겠습니다.

```text
Program A:
100 instructions

Program B:
80 instructions
```

B의 instruction 수가 더 적지만 반드시 빠른 것은 아닙니다.

실행 시간은 다음에 영향을 받습니다.

```text
Instruction count
CPI
Clock frequency
Cache miss
Branch miss
Dependency chain
Instruction latency
Execution-port pressure
```

기본 공식:

```text
CPU time
= Instruction count × CPI / Clock frequency
```

예:

```text
A:
100 instructions
CPI 1.0
→ 100 cycles

B:
80 instructions
CPI 2.0
→ 160 cycles
```

B는 instruction 수가 적어도 더 느립니다.

---

## 8.2 CISC instruction 하나가 빠르다고 볼 수 없는 이유

복잡한 instruction 하나가 여러 단순 instruction을 대신할 수 있습니다.

하지만 내부적으로 여러 µop으로 나뉠 수 있습니다.

```text
1 architectural instruction
→ 4 internal µops
```

반대로 RISC instruction 여러 개가 병렬로 빠르게 실행될 수도 있습니다.

따라서 다음 세 수치를 구분해야 합니다.

```text
Source-level operation count
Architectural instruction count
Internal µop count
```

---

## 8.3 Front-End Bottleneck

CPU front end는 다음 작업을 담당합니다.

```text
Instruction fetch
Instruction boundary detection
Decode
µop generation
Branch prediction
```

Front end가 cycle당 충분한 µop을 공급하지 못하면 execution unit이 비어 있을 수 있습니다.

```text
Front end supply < Back end capacity
              ↓
        Front-end bound
```

x86에서는 variable-length decode가 부담이 될 수 있지만, µop cache가 이를 줄일 수 있습니다.

---

## 8.4 Back-End Bottleneck

Decode는 충분하지만 execution unit이나 memory system이 부족할 수도 있습니다.

```text
Decoded µops
    |
    v
Scheduler
    |
    +--> Integer ALU saturated
    +--> Load ports saturated
    +--> Store ports saturated
    +--> Dependency chain
```

이 경우 ISA보다 microarchitecture의 execution resources가 중요합니다.

---

## 8.5 Code Density

Code density는 동일한 작업을 표현하는 데 필요한 machine-code byte 수를 뜻합니다.

높은 code density의 장점:

```text
I-cache에 더 많은 instruction 저장
Memory bandwidth 절약
Flash 용량 절약
```

CISC 또는 압축 instruction extension은 code density에 유리할 수 있습니다.

예:

```text
ARM Thumb
RISC-V Compressed extension
x86 variable-length encoding
```

하지만 decode 복잡성과 trade-off가 생길 수 있습니다.

---

## 8.6 Dependency와 Instruction Selection

다음 두 표현이 동일한 수학적 결과를 낸다고 합시다.

```text
Version A:
여러 독립 instruction

Version B:
복잡하지만 긴 latency를 가진 instruction 하나
```

B의 instruction 수가 적더라도 latency가 길고 dependency chain이 생기면 느릴 수 있습니다.

Compiler는 target microarchitecture의 다음 특성을 고려할 수 있습니다.

```text
Instruction latency
Reciprocal throughput
Execution port
Fusion 가능성
Register pressure
Code size
```

---

## 8.7 성능 계산 예제

두 CPU가 같은 ISA를 구현한다고 가정합니다.

### CPU A

```text
Clock frequency = 3.0 GHz
Instruction count = 1,000,000
Average CPI = 1.2
```

### CPU B

```text
Clock frequency = 2.5 GHz
Instruction count = 1,000,000
Average CPI = 0.8
```

CPU A 실행 시간:

```text
1,000,000 × 1.2 / 3,000,000,000
= 0.0004 seconds
= 0.4 ms
```

CPU B 실행 시간:

```text
1,000,000 × 0.8 / 2,500,000,000
= 0.00032 seconds
= 0.32 ms
```

CPU A의 clock이 더 높지만 CPU B가 더 빠릅니다.

이유는 microarchitecture 차이로 CPI가 더 낮기 때문입니다.

---

# 9. 실제 시스템 및 Embedded 응용

## 9.1 동일 ISA, 다른 Microarchitecture

같은 ARMv8-A 또는 AArch64 ISA를 구현하는 processor라도 내부 구조는 다를 수 있습니다.

```text
단순 in-order core
고성능 out-of-order core
저전력 core
고성능 server core
```

모두 같은 basic instruction을 실행할 수 있지만 다음 특성이 다릅니다.

```text
Pipeline depth
Decode width
Issue width
Cache size
Branch predictor
Execution units
Power consumption
```

---

## 9.2 Cortex-A와 Cortex-M

Cortex-A와 Cortex-M은 모두 ARM 계열이지만 동일한 실행 환경이 아닙니다.

| 항목             | Cortex-A              | Cortex-M         |
| -------------- | --------------------- | ---------------- |
| 주 대상           | Application processor | Microcontroller  |
| OS             | Linux, Android 등      | Bare metal, RTOS |
| ISA 계열         | 주로 AArch32/AArch64    | Thumb/Thumb-2 중심 |
| Virtual memory | MMU 사용 가능             | 보통 MPU 중심        |
| 내부 복잡도         | 높은 편                  | 비교적 단순           |
| 성능 목표          | 처리량                   | 지연 예측성·전력        |
| Exception 구조   | 복잡한 privilege level   | Vector 기반 단순 구조  |

ARM이라는 이름만 같다고 binary가 그대로 호환되는 것은 아닙니다.

---

## 9.3 Jetson

Jetson 계열 시스템은 ARM application CPU와 NVIDIA GPU를 결합한 형태입니다.

CPU application은 보통 AArch64 machine code를 사용합니다.

GPU kernel은 GPU 전용 execution model과 instruction 체계를 사용합니다.

```text
C/C++ host code
   ↓
AArch64 CPU instruction

CUDA kernel
   ↓
GPU용 intermediate / machine instruction
```

하나의 SoC 안에서도 여러 ISA와 execution architecture가 공존할 수 있습니다.

---

## 9.4 Compiler Target Triple

Compiler는 보통 target을 다음과 비슷한 형태로 구분합니다.

```text
architecture-vendor-operating_system-abi
```

예:

```text
x86_64-unknown-linux-gnu
aarch64-unknown-linux-gnu
arm-none-eabi
```

`arm-none-eabi`는 흔히 bare-metal ARM embedded target을 의미합니다.

Target이 달라지면 다음이 달라질 수 있습니다.

```text
ISA
Object file format
Calling convention
Library
System call
Data layout
```

---

## 9.5 Cross-Compilation

x86-64 laptop에서 Cortex-M용 firmware를 빌드할 수 있습니다.

```text
Build host:
x86-64 macOS/Linux

Target:
ARM Cortex-M
```

Compiler는 target ISA에 맞는 machine code를 생성합니다.

```text
arm-none-eabi-gcc
```

생성한 firmware는 x86-64 CPU에서는 직접 실행할 수 없습니다.

---

## 9.6 Illegal Instruction

프로그램이 CPU가 지원하지 않는 ISA extension을 사용하면 exception이 발생할 수 있습니다.

예:

```text
Binary가 AVX-512 instruction 포함
CPU는 AVX-512 미지원
```

실행 결과:

```text
Illegal instruction
```

Embedded에서도 잘못된 architecture 옵션으로 빌드하면 문제가 생길 수 있습니다.

예:

```text
Cortex-M4용 floating-point instruction 생성
실제 target은 FPU 없는 Cortex-M3
```

---

## 9.7 Real-Time System에서의 Microarchitecture 선택

고성능 out-of-order CPU가 항상 embedded real-time system에 유리하지는 않습니다.

고성능 기능:

```text
Cache
Speculation
Out-of-order execution
Branch prediction
Prefetching
```

장점:

```text
평균 성능 향상
높은 throughput
```

대가:

```text
Worst-case execution time 분석 어려움
Memory latency 변동
Branch 결과에 따른 timing 차이
Cache miss 변동
```

Hard real-time system에서는 낮은 평균 latency보다 예측 가능한 최대 latency가 중요할 수 있습니다.

---

# 10. 아키텍처 비교

## 10.1 ISA와 Microarchitecture

| 항목                   | ISA                                 | Microarchitecture          |
| -------------------- | ----------------------------------- | -------------------------- |
| 핵심 질문                | CPU가 무엇을 해야 하는가?                    | 그것을 어떻게 구현하는가?             |
| Software에 보이는가?      | 예                                   | 대부분 아니오                    |
| 예                    | x86-64, AArch64                     | Zen, Cortex-A76            |
| 정의 대상                | Instruction, register, memory model | Pipeline, cache, predictor |
| Binary compatibility | ISA에 크게 의존                          | 같은 ISA면 대체로 유지             |
| 성능 영향                | Instruction 표현 방식                   | 실제 latency·throughput      |

---

## 10.2 RISC와 CISC

| 항목              | RISC 경향         | CISC 경향               |
| --------------- | --------------- | --------------------- |
| Instruction 형식  | 규칙적             | 다양함                   |
| Instruction 길이  | 고정 또는 비교적 규칙적   | 가변 길이 가능              |
| Memory operand  | Load/store로 분리  | 산술 instruction에 포함 가능 |
| Addressing mode | 비교적 단순          | 다양하고 복잡               |
| Decode          | 상대적으로 단순        | 상대적으로 복잡              |
| Code density    | 낮을 수 있음         | 높을 수 있음               |
| 대표 ISA          | AArch64, RISC-V | x86-64                |

---

## 10.3 AArch64와 x86-64

예제 작업:

```text
Memory에서 값을 읽어 5를 더한다.
```

AArch64:

```asm
ldr w0, [x1]
add w0, w0, #5
```

x86-64:

```asm
mov eax, DWORD PTR [rdi]
add eax, 5
```

또는 x86에서는 memory operand를 직접 사용하는 산술 형태도 가능합니다.

```asm
add eax, DWORD PTR [rdi]
```

하지만 이 명령어가 내부적으로 단일 µop이라는 보장은 없습니다.

---

## 10.4 왜 이런 차이가 생겼는가?

역사적 배경이 다릅니다.

CISC 계열은 다음 요구를 중심으로 발전했습니다.

```text
Memory가 비쌌던 환경
높은 code density
Assembly programming 비중
기존 binary compatibility 유지
```

RISC 계열은 다음을 강조했습니다.

```text
Compiler 중심의 code generation
규칙적인 pipeline
간단한 instruction
많은 register
```

그러나 현대 processor는 양쪽의 장점을 흡수했습니다.

```text
x86:
복잡한 ISA
+
내부 µop 기반 execution

AArch64:
규칙적인 ISA
+
매우 복잡한 out-of-order core
```

---

## 10.5 현대 CPU에서 이 구분이 단순하지 않은 이유

현대 CPU의 성능은 ISA 분류 하나로 설명되지 않습니다.

다음 요소가 더 직접적인 영향을 줄 수 있습니다.

```text
Branch predictor quality
Cache hierarchy
Decode width
µop cache
Execution width
Memory latency
Power limit
Compiler optimization
```

따라서 다음 주장은 부정확합니다.

```text
RISC이므로 항상 빠르다.
CISC이므로 항상 느리다.
```

---

# 11. 흔한 오해

## 오해 1: ISA는 CPU 내부 회로도다

아닙니다.

ISA는 software-visible behavior를 정의합니다.

```text
ISA:
무슨 결과를 내야 하는가?

Microarchitecture:
그 결과를 어떻게 만드는가?
```

---

## 오해 2: 같은 ISA를 쓰면 성능도 같다

아닙니다.

같은 x86-64 binary라도 CPU마다 다음이 다릅니다.

```text
Clock
Cache
Branch predictor
Execution width
Instruction latency
Memory system
```

---

## 오해 3: RISC는 instruction 수가 적다는 뜻이다

이름만 보면 그렇게 보이지만 본질은 단순한 instruction 수의 개수가 아닙니다.

RISC ISA도 많은 extension을 포함할 수 있습니다.

중요한 특징은 다음과 같습니다.

```text
규칙적인 encoding
Load/store architecture
단순한 operand 구조
Compiler 친화성
```

---

## 오해 4: CISC instruction 하나는 hardware에서 한 번에 처리된다

그렇지 않을 수 있습니다.

```text
One x86 instruction
→ multiple µops
→ multiple cycles
```

---

## 오해 5: RISC CPU 내부는 단순하다

고성능 RISC processor 내부에는 다음이 들어갈 수 있습니다.

```text
Out-of-order scheduler
Register renaming
ROB
Speculation
Wide superscalar pipeline
Complex prefetcher
```

ISA와 내부 복잡도를 혼동하면 안 됩니다.

---

## 오해 6: Assembly instruction 수가 적으면 무조건 빠르다

Instruction의 latency, throughput, µop 수, memory access를 봐야 합니다.

```text
적은 assembly instruction
≠
적은 cycle
```

---

## 오해 7: Compiler는 ISA만 알면 된다

좋은 성능을 위해 compiler는 microarchitecture 특성도 고려할 수 있습니다.

```text
어떤 instruction이 빠른가?
어떤 instruction이 여러 µop으로 분해되는가?
어떤 execution port를 사용하는가?
```

---

## 오해 8: ARM binary는 모든 ARM CPU에서 실행된다

ARM 계열 안에서도 ISA version과 profile이 다릅니다.

```text
AArch64
AArch32
Thumb
Thumb-2
Cortex-M profile
Cortex-A profile
```

지원 extension과 ABI가 맞아야 합니다.

---

# 12. 실패 사례와 디버깅

## 12.1 잘못된 Target ISA로 빌드

x86-64 프로그램을 AArch64 binary로 빌드하면 x86-64 CPU에서 직접 실행할 수 없습니다.

가능한 오류:

```text
Exec format error
```

확인 방법:

Linux:

```bash
file executable
readelf -h executable
```

macOS:

```bash
file executable
lipo -info executable
```

---

## 12.2 지원되지 않는 Extension 사용

Compiler option:

```bash
-march=native
```

이 옵션은 build machine의 extension을 적극적으로 사용할 수 있습니다.

다른 구형 CPU에 binary를 복사하면:

```text
Illegal instruction
```

이 발생할 수 있습니다.

배포용 binary에서는 target 범위를 명확히 정해야 합니다.

---

## 12.3 ABI 불일치

같은 ISA라도 ABI가 다르면 function call이 깨질 수 있습니다.

예:

```text
호출자는 인수를 register에 전달
피호출자는 stack에 있다고 가정
```

결과:

```text
잘못된 인수
손상된 stack
잘못된 return
```

확인할 것:

```text
Calling convention
Target triple
Compiler flags
Library architecture
Object file format
```

---

## 12.4 32-bit와 64-bit 혼동

AArch64에서는 다음 register가 연결되어 있습니다.

```text
x0 = 64-bit register
w0 = x0의 하위 32-bit view
```

`w0`에 값을 쓰면 일반적으로 `x0`의 상위 32비트가 0으로 정리됩니다.

이 동작을 모르고 assembly를 작성하면 pointer나 64-bit 값을 손상시킬 수 있습니다.

---

## 12.5 Disassembly를 Source와 일대일로 해석

다음과 같은 판단은 위험합니다.

```text
C 한 줄
=
Assembly 한 줄
```

Compiler는 다음을 수행할 수 있습니다.

```text
Inline
Constant folding
Dead-code elimination
Loop transformation
Vectorization
Instruction scheduling
```

Source line 정보는 최적화 후 부정확하게 보일 수 있습니다.

---

## 12.6 Microbenchmark에서 다른 ISA를 비교하는 오류

AArch64와 x86-64에서 같은 C 코드를 실행했더라도 단순히 실행 시간만 비교하면 ISA 자체의 차이를 측정했다고 말하기 어렵습니다.

다음이 함께 달라집니다.

```text
CPU microarchitecture
Clock frequency
Cache
Memory
Compiler
OS
Thermal state
Power limit
```

ISA 효과를 분리하려면 실험 설계가 매우 어렵습니다.

---

## 12.7 Undefined Behavior로 인한 Assembly 오해

다음 함수:

```c
int is_increment_larger(int x)
{
    return x + 1 > x;
}
```

C에서 signed overflow는 Undefined Behavior입니다.

Compiler는 overflow가 없다고 가정하여 다음처럼 최적화할 수 있습니다.

```text
항상 true 반환
```

개발자가 “CPU에서는 wrap되는데 compiler가 잘못했다”고 생각할 수 있지만, 실제 문제는 source code의 Undefined Behavior입니다.

---

# 13. 확인 문제

정답은 바로 공개하지 않습니다.

## Level 1: 개념 확인

### 문제 1

ISA와 Microarchitecture를 다음 형식으로 비교하세요.

```text
ISA가 정의하는 것:
Microarchitecture가 정의하는 것:
두 개념의 관계:
```

---

### 문제 2

Intel CPU와 AMD CPU가 같은 x86-64 프로그램을 실행할 수 있지만 성능은 다를 수 있는 이유를 설명하세요.

다음 용어를 포함하세요.

```text
ISA
Microarchitecture
CPI
Cache
```

---

## Level 2: 동작 과정 추적

### 문제 3

다음 x86-64 instruction이 있습니다.

```asm
add eax, DWORD PTR [rdi]
```

이를 개념적인 내부 작업으로 분해하세요.

다음 항목을 포함해야 합니다.

```text
주소 계산
Memory load
ALU 연산
결과 저장
```

그리고 architectural instruction 수와 내부 µop 수가 반드시 같지 않은 이유를 설명하세요.

---

### 문제 4

다음 AArch64 코드가 있습니다.

```asm
ldr w2, [x0, x1, lsl #2]
add w0, w2, #7
ret
```

초기 상태:

```text
x0 = 0x2000
x1 = 3
Memory[0x200C] = 35
```

실행 후 다음 값을 구하세요.

```text
w2 =
w0 =
effective address =
```

---

## Level 3: 성능 계산 또는 코드 분석

### 문제 5

같은 ISA를 구현하는 두 CPU가 있습니다.

CPU A:

```text
Instruction count = 3,000,000
Average CPI = 1.4
Clock = 3.5 GHz
```

CPU B:

```text
Instruction count = 3,000,000
Average CPI = 0.9
Clock = 2.8 GHz
```

각 CPU의 실행 시간을 계산하고 어느 CPU가 더 빠른지 설명하세요.

---

### 문제 6

다음 두 구현을 비교하세요.

```text
Implementation A:
Architectural instructions = 5
Average 1 µop per instruction

Implementation B:
Architectural instructions = 3
Average 3 µops per instruction
```

Instruction 수만 보고 B가 빠르다고 결론 내릴 수 없는 이유를 설명하세요.

추가로 고려해야 할 성능 요소를 최소 4개 제시하세요.

---

## Level 4: 설계·디버깅·실무 응용

### 문제 7

개발자가 `-march=native`로 빌드한 프로그램을 다른 PC에서 실행했더니 `Illegal instruction`이 발생했습니다.

다음을 설명하세요.

```text
발생 가능한 원인
확인해야 할 사항
배포 binary를 빌드할 때의 해결 방법
```

---

### 문제 8

Cortex-M4F를 대상으로 빌드한 firmware를 FPU가 없는 Cortex-M4 variant에 올렸더니 fault가 발생했습니다.

ISA extension과 compiler target option 관점에서 원인을 설명하고, 확인해야 할 build flag를 제시하세요.

---

# 14. 실습 과제

## 실습 목표

같은 C 함수가 target ISA에 따라 서로 다른 assembly로 변환되는 것을 확인합니다.

또한 다음을 구분합니다.

```text
Source code
ISA
ABI
Compiler optimization
Microarchitecture tuning
```

---

## 준비물

다음 중 하나가 필요합니다.

```text
Clang
GCC cross-compiler
Compiler Explorer 사용 경험
```

로컬 Clang 확인:

```bash
clang --version
```

---

## 코드

파일 이름: `target_compare.c`

```c
#include <stddef.h>

__attribute__((noinline))
int sum_element(const int *array, int index)
{
    return array[index] + 5;
}

__attribute__((noinline))
int sum_four(const int *array)
{
    return array[0]
         + array[1]
         + array[2]
         + array[3];
}
```

이 파일은 standard library를 사용하지 않으므로 cross-target assembly 생성에 적합합니다.

---

## 현재 시스템용 Assembly

```bash
clang \
  -O2 \
  -Wall \
  -Wextra \
  -Wpedantic \
  -S \
  target_compare.c \
  -o target_native.s
```

확인:

```bash
cat target_native.s
```

---

## AArch64용 Assembly

```bash
clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -S \
  target_compare.c \
  -o target_aarch64.s
```

확인:

```bash
cat target_aarch64.s
```

---

## x86-64용 Assembly

```bash
clang \
  --target=x86_64-linux-gnu \
  -O2 \
  -S \
  -masm=intel \
  target_compare.c \
  -o target_x86_64.s
```

확인:

```bash
cat target_x86_64.s
```

---

## 예상 관찰 결과

AArch64에서는 다음과 같은 형태가 나타날 수 있습니다.

```asm
sum_element:
    ldr w0, [x0, w1, sxtw #2]
    add w0, w0, #5
    ret
```

x86-64에서는 다음과 비슷할 수 있습니다.

```asm
sum_element:
    movsxd rsi, esi
    mov eax, DWORD PTR [rdi + rsi*4]
    add eax, 5
    ret
```

정확한 assembly는 compiler 버전에 따라 달라질 수 있습니다.

---

## 관찰할 항목

### 1. 인수 register

AArch64:

```text
x0, w1
```

x86-64 System V:

```text
rdi, esi
```

이는 ISA뿐 아니라 ABI의 영향입니다.

### 2. Addressing mode

배열 index를 다음처럼 처리하는지 확인합니다.

```text
base + index × 4
```

### 3. Load와 add

각 ISA가 memory load와 arithmetic을 어떻게 표현하는지 확인합니다.

### 4. Return register

AArch64:

```text
w0
```

x86-64:

```text
eax
```

---

## `-O0`과 `-O2` 비교

```bash
clang \
  --target=aarch64-linux-gnu \
  -O0 \
  -S \
  target_compare.c \
  -o target_aarch64_O0.s

clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -S \
  target_compare.c \
  -o target_aarch64_O2.s
```

비교:

```bash
diff -u target_aarch64_O0.s target_aarch64_O2.s
```

관찰할 것:

```text
Stack 사용
Register 사용
Instruction 수
함수 prologue/epilogue
불필요한 load/store
```

---

## `-mcpu` 또는 `-mtune` 실험

지원되는 경우:

```bash
clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -mcpu=cortex-a76 \
  -S \
  target_compare.c \
  -o target_cortex_a76.s
```

다른 target:

```bash
clang \
  --target=aarch64-linux-gnu \
  -O2 \
  -mcpu=cortex-a55 \
  -S \
  target_compare.c \
  -o target_cortex_a55.s
```

작은 함수에서는 assembly가 같을 수 있습니다.

이는 compiler가 항상 target별로 다른 instruction을 생성한다는 뜻이 아니기 때문입니다. 차이는 큰 loop, vectorization, scheduling에서 더 잘 나타날 수 있습니다.

---

## 예상 결과가 다를 수 있는 이유

```text
Compiler version
Target backend
Optimization level
ABI
Instruction extension
Function alignment
Vectorization decision
```

---

## 잘못된 결과가 나올 수 있는 원인

1. Clang build가 해당 target을 지원하지 않는 경우
2. Target triple이 잘못된 경우
3. `-masm=intel`이 AArch64에 적용된 경우
4. 함수가 inline되거나 제거된 경우
5. Cross-target header가 필요한 코드를 사용한 경우
6. Compiler가 다른 addressing mode를 선택한 경우
7. macOS ABI와 Linux ABI를 혼동한 경우

---

## 추가 도전 과제

다음 함수를 추가하세요.

```c
__attribute__((noinline))
int dot_product(const int *a, const int *b, size_t count)
{
    int sum = 0;

    for (size_t i = 0; i < count; ++i) {
        sum += a[i] * b[i];
    }

    return sum;
}
```

다음 옵션을 비교하세요.

```bash
-O2
-O3
-O3 -fno-vectorize
-O3 -march=native
```

관찰할 것:

```text
SIMD instruction 사용 여부
Loop unrolling
Instruction count
Code size
Target ISA extension
```

주의:

Signed overflow가 발생하면 C에서 Undefined Behavior가 될 수 있습니다. 실험 입력 범위를 충분히 작게 유지하거나, 명확한 modulo 연산을 원한다면 unsigned type 사용을 고려해야 합니다.

---

# 15. 면접에서 설명하는 방법

## 15.1 30초 설명

답변 구조:

```text
ISA 정의
→ Microarchitecture 정의
→ RISC/CISC
→ 현대 CPU의 혼합 구조
```

예시:

> ISA는 software가 볼 수 있는 instruction, register, memory operation과 같은 processor의 동작 규약입니다. Microarchitecture는 그 ISA를 pipeline, cache, execution unit, branch predictor 등으로 구현하는 내부 방식입니다. 그래서 Intel과 AMD CPU는 같은 x86-64 ISA를 실행하면서도 성능이 다를 수 있습니다. RISC와 CISC는 ISA 설계 철학의 차이지만, 현대 x86은 복잡한 instruction을 내부 µop으로 분해하고 현대 ARM도 복잡한 out-of-order core를 사용하므로 단순한 이분법으로 성능을 판단할 수 없습니다.

---

## 15.2 2분 설명

논리 구조:

1. ISA는 software-hardware contract
2. Microarchitecture는 구현
3. 동일 ISA의 binary compatibility
4. 성능 차이
5. RISC와 CISC의 현대적 해석

예시:

> ISA는 processor가 제공하는 architectural register, instruction encoding, memory access, exception과 같은 software-visible behavior를 정의합니다. 반면 microarchitecture는 fetch, decode, pipeline, cache, out-of-order scheduler와 execution unit을 사용해 해당 ISA를 실제로 구현하는 방법입니다. 같은 x86-64 ISA를 구현하는 Intel과 AMD processor는 같은 binary를 실행할 수 있지만 pipeline depth, cache hierarchy, branch predictor, instruction latency가 다르므로 CPI와 실제 성능이 달라집니다. RISC는 규칙적인 encoding과 load/store 구조를 강조하고, CISC는 복잡한 addressing mode와 variable-length instruction을 허용하는 경향이 있습니다. 하지만 현대 x86 CPU는 instruction을 내부 µop으로 분해하고, AArch64 CPU도 superscalar와 speculation을 사용합니다. 따라서 RISC와 CISC는 ISA 특성을 설명하는 개념이지 CPU 전체의 실제 복잡도나 성능을 직접 결정하는 분류는 아닙니다.

---

## 15.3 심화 꼬리 질문

```text
ISA와 ABI는 무엇이 다른가?
Architectural instruction과 µop은 무엇이 다른가?
x86 variable-length decode는 어떤 비용을 만드는가?
µop cache는 왜 필요한가?
같은 ISA에서 CPU별 instruction latency가 다른 이유는 무엇인가?
-march와 -mtune의 차이는 무엇인가?
RISC CPU도 microcode를 사용할 수 있는가?
왜 instruction count만으로 성능을 판단할 수 없는가?
```

좋은 답변은 다음 계층을 구분해야 합니다.

```text
Language semantics
Compiler
ABI
ISA
Microarchitecture
Physical implementation
```

---

# 16. 핵심 정리

1. **ISA는 software와 hardware 사이의 계약입니다.**

   ```text
   무엇을 실행할 수 있는가?
   실행 결과는 어떻게 보여야 하는가?
   ```

2. **Microarchitecture는 ISA를 실제로 구현하는 내부 방법입니다.**

   ```text
   Pipeline
   Cache
   Decoder
   Execution unit
   Branch predictor
   ```

3. 같은 ISA를 구현하는 CPU는 같은 binary를 실행할 수 있지만 성능은 다를 수 있습니다.

4. ISA는 일반적으로 instruction의 최종 결과를 정의하지만 몇 cycle에 실행할지는 강제하지 않습니다.

5. **Architectural instruction 하나가 내부 µop 하나와 일치한다는 보장은 없습니다.**

6. x86-64는 variable-length instruction과 다양한 addressing mode를 가진 CISC 계열 ISA입니다.

7. AArch64는 규칙적인 32-bit instruction encoding과 load/store 구조를 사용하는 RISC 계열 ISA입니다.

8. 현대 x86 CPU는 복잡한 instruction을 내부 µop으로 변환할 수 있습니다.

9. 현대 AArch64 CPU도 out-of-order execution, speculation, register renaming을 사용하는 복잡한 processor입니다.

10. 따라서 다음 등식은 틀렸습니다.

```text
RISC = 단순하고 항상 빠름
CISC = 복잡하고 항상 느림
```

11. 성능은 다음 요소를 함께 봐야 합니다.

```text
Instruction count
µop count
CPI
Latency
Throughput
Cache miss
Branch miss
Dependency chain
```

12. ISA와 ABI는 다릅니다.

```text
ISA:
CPU instruction과 architectural state

ABI:
Function argument, return value, stack 등의 software 약속
```

13. Compiler의 `-march`는 사용할 ISA extension을 결정하고, `-mtune`은 특정 microarchitecture에 맞춘 성능 선택에 영향을 줄 수 있습니다.

14. 배포 대상보다 새로운 ISA extension을 사용하면 `Illegal instruction`이 발생할 수 있습니다.