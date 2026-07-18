# 전체 강의 커리큘럼

전체 과정은 다음 10개 Part로 진행합니다.

| Part | 핵심 주제                       | 최종적으로 설명할 수 있어야 하는 것                            |
| ---- | --------------------------- | ----------------------------------------------- |
| 1    | CPU와 ISA 기초                 | 프로그램이 명령어로 변환되고 실행되는 과정                         |
| 2    | Pipeline                    | 여러 명령어가 겹쳐 실행되는 원리와 hazard                      |
| 3    | Modern CPU Execution        | Superscalar, Out-of-Order, ROB, Speculation     |
| 4    | Cache와 Memory Hierarchy     | locality, cache mapping, cache miss             |
| 5    | Virtual Memory와 TLB         | 주소 변환과 page-table walk                          |
| 6    | Multicore와 Memory Ordering  | coherence, consistency, atomic, fence           |
| 7    | Interrupt, Exception, DMA   | 장치와 CPU가 상호작용하는 방법                              |
| 8    | ARM Architecture            | Cortex-A/M, register, exception, context switch |
| 9    | Specialized Execution Units | FPU, SIMD, NEON, SSE, AVX                       |
| 10   | Performance Measurement     | CPI, IPC, PMU, `perf`, assembly 분석              |

Part 1은 다음 순서로 시작합니다.

1. **Lecture 1. 컴퓨터가 명령어를 실행한다는 것**
2. Lecture 2. Von Neumann과 Harvard Architecture
3. Lecture 3. ISA, Microarchitecture, RISC와 CISC
4. Lecture 4. Register, ALU, Control Unit, Datapath
5. Lecture 5. Fetch–Decode–Execute Cycle

---

# Lecture 1. 컴퓨터가 명령어를 실행한다는 것

## 1. 오늘의 핵심 질문

오늘 답해야 할 질문은 다음과 같습니다.

> C 프로그램에서 `a + b`라고 작성했을 때, CPU 내부에서는 실제로 무엇이 일어나는가?

그리고 조금 더 세분화하면 다음과 같습니다.

1. CPU는 C나 C++ 코드를 직접 이해하는가?
2. 명령어는 데이터와 무엇이 다른가?
3. CPU는 메모리에 있는 비트가 명령어인지 데이터인지 어떻게 구분하는가?
4. 명령어 실행은 단순한 계산인가, 아니면 상태 변화인가?
5. 프로그램이 실행된다는 것은 정확히 무엇이 계속 반복된다는 뜻인가?

오늘 강의가 끝나면 다음 문장을 자신의 말로 설명할 수 있어야 합니다.

> CPU는 프로그램 카운터가 가리키는 주소에서 명령어를 가져오고, 그 비트 패턴을 ISA 규칙에 따라 해석한 뒤, 레지스터·메모리·제어 상태를 변경한다.

---

## 2. 이전 Lecture와의 연결

첫 번째 강의이므로 이전 Lecture는 없습니다.

대신 앞으로 이어질 흐름을 확인하겠습니다.

```text
소스 코드
   ↓ compiler
Assembly
   ↓ assembler
Machine code
   ↓ loader / operating system
Memory에 배치
   ↓
CPU가 명령어를 반복 실행
```

이번 Lecture는 이 흐름 전체의 지도를 그리는 단계입니다.

아직 pipeline, cache, out-of-order execution 같은 세부 구조를 깊게 다루지는 않습니다. 먼저 CPU가 무엇을 실행하는지 정확히 정의해야 합니다.

---

## 3. 직관적 설명

### 3.1 CPU를 요리사로 비유한다면

다음과 같은 상황을 생각해 봅시다.

```text
요리책의 현재 페이지     → Program Counter
요리책에 적힌 한 단계    → Instruction
조리대 위 재료           → Register
창고                     → Memory
칼과 프라이팬            → Execution unit / ALU
다음 페이지로 이동       → Program Counter update
```

요리사는 다음 과정을 반복합니다.

1. 현재 페이지에 적힌 지시를 읽습니다.
2. 지시가 무엇을 요구하는지 해석합니다.
3. 필요한 재료를 가져옵니다.
4. 작업을 수행합니다.
5. 결과를 조리대나 창고에 저장합니다.
6. 다음 지시로 이동합니다.

CPU의 기본적인 동작도 이와 비슷합니다.

```text
현재 명령어 주소 확인
        ↓
명령어 읽기
        ↓
명령어 해석
        ↓
입력 operand 준비
        ↓
연산 수행
        ↓
결과 저장
        ↓
다음 명령어 주소 결정
```

### 3.2 비유의 한계

CPU는 실제 요리사처럼 문장의 의미를 이해하지 않습니다.

CPU가 보는 것은 다음과 같은 비트 패턴입니다.

```text
10001011000000010000000000000000
```

이 비트들이 특정 ARM 명령어 형식에 따라 해석되면 다음과 같은 의미가 될 수 있습니다.

```asm
add w0, w0, w1
```

CPU는 “두 값을 더해야겠다”고 사고하지 않습니다.

비트 필드 일부가 제어 회로를 활성화하여 다음과 같은 일이 일어납니다.

```text
register 0 읽기
register 1 읽기
ALU의 연산 선택 신호를 ADD로 설정
두 값을 ALU 입력에 전달
ALU 결과를 register 0에 기록
```

즉, 명령어 실행은 **의미 이해**가 아니라 **회로 상태 변화**입니다.

---

## 4. 형식적 정의

### 4.1 프로그램(Program)

**용어:** 프로그램(Program)

**정의:**
컴퓨터가 수행해야 할 동작을 표현한 명령과 데이터의 집합입니다.

**필요한 이유:**
CPU가 어떤 계산을 어떤 순서로 수행해야 하는지 지정하기 위해 필요합니다.

**하드웨어에서의 역할:**
실행 가능한 프로그램의 기계어 명령어와 데이터가 메모리에 배치됩니다.

**관련된 다른 개념:**
소스 코드, executable, process, instruction, data.

중요한 구분이 있습니다.

```text
Program = 디스크 등에 저장된 실행 가능한 코드
Process = 실행 중인 프로그램과 그 실행 상태
```

프로그램 파일 자체는 실행 중이 아닙니다. 운영체제가 프로그램을 메모리에 적재하고 실행 상태를 만들면 process가 됩니다.

---

### 4.2 명령어(Instruction)

**용어:** 명령어(Instruction)

**정의:**
ISA가 정의하는 하나의 연산 단위입니다.

**필요한 이유:**
하드웨어가 수행할 작업을 명확한 이진 형식으로 표현하기 위해 필요합니다.

**하드웨어에서의 역할:**
명령어 비트는 연산 종류, 입력 위치, 출력 위치, 즉시값 등의 정보를 전달합니다.

**관련된 다른 개념:**
Opcode, operand, instruction encoding, machine code, assembly.

예를 들어 다음 AArch64 명령어를 보겠습니다.

```asm
add w0, w1, w2
```

의미는 다음과 같습니다.

```text
w0 = w1 + w2
```

여기서:

```text
add       → Opcode
w1, w2    → Source operands
w0        → Destination operand
```

---

### 4.3 기계어(Machine Code)

**용어:** 기계어(Machine Code)

**정의:**
CPU가 ISA 규칙에 따라 명령어로 해석할 수 있는 이진 인코딩입니다.

**필요한 이유:**
하드웨어는 C 문법이나 assembly 문자열이 아니라 비트 신호를 처리하기 때문입니다.

**하드웨어에서의 역할:**
Instruction decoder가 비트 필드를 분석하여 내부 제어 신호를 생성합니다.

**관련된 다른 개념:**
Assembly, assembler, opcode, ISA.

Assembly는 사람이 읽기 위한 표현입니다.

```asm
add w0, w1, w2
```

CPU가 실제로 읽는 것은 이 명령어에 대응되는 32비트 값입니다.

```text
Assembly text → Assembler → Machine-code bits
```

---

### 4.4 명령어 집합 구조(Instruction Set Architecture, ISA)

**용어:** 명령어 집합 구조(Instruction Set Architecture, ISA)

**정의:**
소프트웨어와 프로세서 사이의 프로그래밍 인터페이스입니다.

ISA는 일반적으로 다음을 정의합니다.

```text
사용 가능한 명령어
레지스터 구조
명령어 인코딩
데이터 타입
주소 지정 방식
예외 처리의 기본 규칙
메모리 접근 방식
```

**필요한 이유:**
동일한 machine code가 어떤 의미를 갖는지 소프트웨어와 하드웨어가 합의해야 하기 때문입니다.

**하드웨어에서의 역할:**
프로세서는 해당 ISA가 정의한 동작을 구현해야 합니다.

**관련된 다른 개념:**
x86-64, AArch64, ARMv7, RISC-V, microarchitecture.

같은 C 코드는 ISA에 따라 서로 다른 machine code로 컴파일됩니다.

```text
C source
 ├─ x86-64 compiler  → x86-64 machine code
 ├─ AArch64 compiler → AArch64 machine code
 └─ RISC-V compiler  → RISC-V machine code
```

---

### 4.5 프로그램 카운터(Program Counter, PC)

**용어:** 프로그램 카운터(Program Counter, PC)

**정의:**
현재 또는 다음에 가져올 명령어의 주소를 나타내는 CPU 상태입니다.

**필요한 이유:**
CPU가 메모리의 어느 위치에서 다음 명령어를 읽을지 알아야 하기 때문입니다.

**하드웨어에서의 역할:**
Instruction fetch를 위한 주소를 제공합니다.

**관련된 다른 개념:**
Branch, jump, function call, return, exception.

일반적인 순차 실행에서는 PC가 다음 명령어를 가리키도록 증가합니다.

AArch64 명령어는 기본적으로 4바이트이므로, 단순한 경우:

```text
PC = 0x1000
명령어 실행
PC = 0x1004
명령어 실행
PC = 0x1008
```

하지만 분기 명령어를 만나면 PC는 연속된 다음 주소가 아닌 다른 주소로 바뀝니다.

```asm
b target
```

```text
PC ← target의 주소
```

---

### 4.6 프로세서 상태(Architectural State)

**용어:** 아키텍처 상태(Architectural State)

**정의:**
ISA 관점에서 프로그램 실행 결과를 결정하는, 소프트웨어에 보이는 CPU 상태입니다.

대표적으로 다음이 포함됩니다.

```text
General-purpose registers
Program Counter
Stack Pointer
Condition flags
일부 system registers
Memory의 관찰 가능한 내용
```

명령어 실행은 본질적으로 다음과 같이 볼 수 있습니다.

```text
현재 상태 + 명령어 → 새로운 상태
```

수식으로 표현하면:

```text
S(t+1) = Execute(S(t), Instruction)
```

예를 들어:

```text
초기 상태:
w1 = 10
w2 = 20
w0 = 0
PC = 0x1000

명령어:
add w0, w1, w2

실행 후:
w1 = 10
w2 = 20
w0 = 30
PC = 0x1004
```

이 관점은 매우 중요합니다.

CPU 실행은 단순히 “계산 결과를 만든다”가 아니라:

> 정의된 규칙에 따라 기계의 상태를 한 단계씩 변화시키는 과정입니다.

---

## 5. 내부 구조와 동작 원리

### 5.1 가장 단순한 CPU 모델

교과서적인 CPU는 다음과 같이 표현할 수 있습니다.

```text
                     address
             +--------------------+
             |                    v
        +---------+          +----------+
        |   PC    |--------->|  Memory  |
        +---------+          +----------+
             ^                    |
             |                    | instruction
             |                    v
             |              +-----------+
             |              |  Decoder  |
             |              +-----------+
             |                    |
             |              control signals
             |                    |
             |          +---------+---------+
             |          |                   |
             |          v                   v
             |    +-----------+        +---------+
             |    | Registers |------->|   ALU   |
             |    +-----------+        +---------+
             |          ^                   |
             |          |                   |
             +----------+-------------------+
                        result / next PC
```

이 그림에서 각 요소의 역할을 살펴보겠습니다.

### 5.2 Program Counter

입력:

```text
이전 명령어에서 계산된 다음 주소
```

출력:

```text
다음 instruction fetch 주소
```

일반적인 다음 주소:

```text
PC + instruction size
```

분기 시:

```text
branch target address
```

---

### 5.3 Instruction memory 또는 cache

입력:

```text
PC가 제공한 주소
```

처리:

```text
해당 주소의 instruction bytes를 읽음
```

출력:

```text
명령어 비트 패턴
```

현대 CPU에서는 일반적으로 주 메모리 DRAM을 매번 직접 읽지 않습니다.

대부분 다음 경로를 사용합니다.

```text
PC
 ↓
L1 Instruction Cache
 ↓ miss
L2 Cache
 ↓ miss
Last-Level Cache
 ↓ miss
DRAM
```

그러나 이것은 이후 cache Lecture에서 자세히 다룹니다.

---

### 5.4 Instruction decoder

입력:

```text
명령어 비트
```

처리:

```text
Opcode 확인
Source register 번호 확인
Destination register 번호 확인
Immediate 값 추출
Instruction 형식 확인
```

출력:

```text
register read 제어 신호
ALU operation 선택
memory read/write 제어
register write-back 제어
PC update 방식
```

예를 들어 개념적으로 다음 명령어가 있다면:

```asm
add w0, w1, w2
```

decoder는 다음과 비슷한 정보를 추출합니다.

```text
Operation       = ADD
Source register = w1
Source register = w2
Destination     = w0
Register write  = enabled
Memory read     = disabled
Memory write    = disabled
```

실제 현대 CPU의 decode 결과는 더 복잡합니다. x86 프로세서는 하나의 복잡한 명령어를 내부의 여러 마이크로 연산(micro-operation, µop)으로 변환하기도 합니다.

---

### 5.5 Register file

레지스터 파일(Register File)은 CPU 내부의 작은 고속 저장소입니다.

입력:

```text
읽을 register 번호
쓸 register 번호
쓸 값
write enable
```

출력:

```text
선택된 source register 값
```

예를 들어:

```text
w1 = 10
w2 = 20
```

명령어가 다음과 같다면:

```asm
add w0, w1, w2
```

레지스터 파일은 ALU에 10과 20을 제공합니다.

---

### 5.6 ALU

산술 논리 장치(Arithmetic Logic Unit, ALU)는 정수 연산을 수행합니다.

대표적인 연산:

```text
Addition
Subtraction
AND
OR
XOR
Shift
Comparison
Address calculation
```

입력:

```text
operand A
operand B
operation selection
```

출력:

```text
연산 결과
condition flags 또는 비교 결과
```

예:

```text
A = 10
B = 20
operation = ADD

result = 30
```

---

### 5.7 결과 저장

연산 결과는 명령어 종류에 따라 다른 곳에 저장됩니다.

```text
산술 명령어 → register
Load 명령어 → memory에서 읽은 값을 register에 저장
Store 명령어 → register 값을 memory에 저장
Branch 명령어 → PC 변경
Compare 명령어 → flags 또는 비교 결과 변경
```

---

### 5.8 기본 명령어 실행 루프

단순화한 모델은 다음과 같습니다.

```text
while (CPU is running) {
    instruction = memory[PC];
    decoded = decode(instruction);
    operands = read_registers(decoded);
    result = execute(decoded, operands);
    update_state(result);
    PC = calculate_next_pc(decoded, result);
}
```

이 코드는 실제 CPU 구현 코드가 아닙니다.

하드웨어에서는 이 동작들이 논리 게이트, 레지스터, 멀티플렉서, 클록 신호로 구현됩니다.

---

### 교과서적 단순화와 실제 프로세서

**교과서적 단순화:**

```text
한 명령어를 가져온다.
완전히 실행한다.
다음 명령어로 넘어간다.
```

**실제 프로세서:**

```text
여러 명령어를 동시에 가져옴
여러 명령어를 동시에 decode
여러 명령어가 서로 다른 실행 장치에서 진행
프로그램 순서와 다른 순서로 실행 가능
분기 결과를 알기 전에 추측 실행 가능
완료 결과는 프로그램 순서대로 확정 가능
```

**이 단순화가 유용한 이유:**

ISA 수준에서 프로그램의 의미를 이해하기 쉽습니다.

**이 단순화가 깨지는 상황:**

다음 현상을 분석할 때는 부족합니다.

```text
Pipeline hazard
Out-of-order execution
Speculation
Cache miss
Memory ordering
Interrupt timing
Performance counter
```

지금은 먼저 단순 모델을 정확히 이해해야 합니다.

---

## 6. 단계별 실행 추적

다음 C 함수를 생각해 봅시다.

```c
int add(int a, int b)
{
    return a + b;
}
```

AArch64 호출 규약에서는 일반적으로 첫 번째와 두 번째 `int` 인수가 `w0`, `w1`에 전달되고 반환값도 `w0`에 저장됩니다.

컴파일러가 개념적으로 다음 assembly를 만들 수 있습니다.

```asm
add:
    add w0, w0, w1
    ret
```

### 초기 상태

```text
w0 = 7
w1 = 5
PC = add 함수의 첫 명령어 주소
```

가상의 주소를 사용하겠습니다.

```text
0x1000: add w0, w0, w1
0x1004: ret
```

### 1단계: Fetch

PC 값:

```text
PC = 0x1000
```

CPU는 `0x1000` 주소의 명령어를 가져옵니다.

```text
Instruction bits ← Memory[0x1000]
```

실제 현대 CPU에서는 instruction cache와 address translation이 개입할 수 있습니다.

---

### 2단계: Decode

Decoder가 명령어 비트를 해석합니다.

```text
Operation   = ADD
Source 1    = w0
Source 2    = w1
Destination = w0
```

---

### 3단계: Operand read

Register file에서 값을 읽습니다.

```text
Read w0 → 7
Read w1 → 5
```

---

### 4단계: Execute

ALU가 두 값을 더합니다.

```text
7 + 5 = 12
```

---

### 5단계: Write-back

ALU 결과를 목적지 레지스터에 기록합니다.

```text
w0 ← 12
```

---

### 6단계: PC update

AArch64 명령어 크기는 4바이트이므로 순차적으로 다음 명령어로 이동합니다.

```text
PC ← 0x1004
```

이제 상태는 다음과 같습니다.

```text
w0 = 12
w1 = 5
PC = 0x1004
```

---

### 7단계: `ret` 실행

AArch64의 `ret`은 일반적으로 link register인 `x30`에 저장된 반환 주소로 분기합니다.

개념적으로:

```text
PC ← x30
```

즉, 호출자에게 돌아갑니다.

`ret`은 단순히 “함수를 끝낸다”는 추상적인 명령이 아닙니다. 실제로는 **반환 주소를 PC에 넣는 제어 흐름 변경 명령어**입니다.

---

## 7. C/C++ 및 Assembly 예제

### 7.1 실행 가능한 C 코드

```c
#include <stdio.h>
#include <stdlib.h>

static int add(int a, int b)
{
    return a + b;
}

int main(void)
{
    const int a = 7;
    const int b = 5;
    const int result = add(a, b);

    if (printf("%d + %d = %d\n", a, b, result) < 0) {
        fprintf(stderr, "Failed to write output.\n");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

파일 이름:

```text
add.c
```

### Linux 또는 macOS에서 컴파일

```bash
cc -O0 -Wall -Wextra -Wpedantic add.c -o add
./add
```

예상 출력:

```text
7 + 5 = 12
```

---

### 7.2 Assembly 생성

#### macOS Apple Silicon 또는 AArch64 Linux

```bash
cc -O0 -S add.c -o add_O0.s
cc -O2 -S add.c -o add_O2.s
```

#### Intel/AMD x86-64 Linux

```bash
gcc -O0 -S -masm=intel add.c -o add_O0.s
gcc -O2 -S -masm=intel add.c -o add_O2.s
```

Assembly 파일 확인:

```bash
less add_O0.s
```

또는:

```bash
grep -A15 "add:" add_O0.s
```

macOS에서는 함수 이름이 `_add`로 표시될 수 있습니다.

```bash
grep -A15 "_add:" add_O0.s
```

---

### 7.3 최적화 수준에 따른 차이

`-O0`에서는 컴파일러가 디버깅 편의를 위해 값을 stack에 저장할 수 있습니다.

개념적인 AArch64 `-O0` 결과:

```asm
add:
    sub sp, sp, #16
    str w0, [sp, #12]
    str w1, [sp, #8]
    ldr w0, [sp, #12]
    ldr w1, [sp, #8]
    add w0, w0, w1
    add sp, sp, #16
    ret
```

`-O2`에서는 불필요한 memory access가 제거될 수 있습니다.

```asm
add:
    add w0, w0, w1
    ret
```

같은 C 함수라도 생성된 명령어 수가 달라집니다.

이유는 C 코드가 하드웨어 명령어를 일대일로 지정하는 언어가 아니기 때문입니다. C 표준이 요구하는 관찰 가능한 동작을 유지하는 한, 컴파일러는 명령어를 자유롭게 바꿀 수 있습니다.

---

### 7.4 함수 자체가 사라질 수도 있다

`add()`가 단순하고 호출 위치가 알려져 있으면 컴파일러가 inline할 수 있습니다.

더 나아가:

```c
const int a = 7;
const int b = 5;
const int result = add(a, b);
```

값이 모두 컴파일 시점에 알려져 있으므로 컴파일러가 결과를 미리 계산할 수 있습니다.

```text
7 + 5 → compile time에 12로 계산
```

따라서 최종 실행 파일에는 실제 `add` instruction이 없을 수도 있습니다.

이는 CPU가 덧셈을 하지 않았다는 뜻입니다. 덧셈을 컴파일러가 미리 수행했기 때문입니다.

---

## 8. 성능과 메모리 관점

### 8.1 Latency

지연 시간(Latency)은 하나의 작업이 시작해서 끝날 때까지 필요한 시간입니다.

예:

```text
정수 ADD latency = 결과가 의존 명령어에 사용 가능해질 때까지의 cycle 수
```

가상의 프로세서에서 ADD latency가 1 cycle이라면:

```text
Cycle 1: r1 + r2 실행
Cycle 2: 결과를 사용하는 다음 연산 가능
```

실제 latency는 CPU 모델과 명령어 종류에 따라 다릅니다.

---

### 8.2 Throughput

처리량(Throughput)은 단위 시간당 몇 개의 작업을 시작하거나 완료할 수 있는지를 나타냅니다.

어떤 CPU가 독립적인 정수 덧셈을 cycle당 4개 처리할 수 있다면:

```text
ADD latency    = 1 cycle
ADD throughput = 4 instructions/cycle
```

Latency와 throughput은 같은 개념이 아닙니다.

예를 들어 나눗셈은 latency가 길더라도 내부가 pipeline되어 있다면 이전 나눗셈이 끝나기 전에 다음 나눗셈을 시작할 수 있습니다.

---

### 8.3 Clock cycle

클록 주기(Clock Cycle)는 CPU의 순차 회로 상태가 갱신되는 기본 시간 단위입니다.

3 GHz CPU의 이상적인 한 cycle 시간은:

```text
1 / 3,000,000,000 second
≈ 0.333 nanoseconds
```

하지만 다음을 주의해야 합니다.

> 명령어 하나가 반드시 한 cycle에 끝나는 것은 아닙니다.

일부 명령어는 여러 cycle이 필요하며, memory access는 cache miss 여부에 따라 수십에서 수백 cycle이 걸릴 수 있습니다.

---

### 8.4 CPI와 IPC

**CPI(Cycles Per Instruction):**

```text
CPI = 총 cycle 수 / 완료된 instruction 수
```

**IPC(Instructions Per Cycle):**

```text
IPC = 완료된 instruction 수 / 총 cycle 수
```

단순한 상황에서는:

```text
IPC = 1 / CPI
```

예:

```text
1,000 instructions
800 cycles
```

```text
CPI = 800 / 1,000 = 0.8
IPC = 1,000 / 800 = 1.25
```

CPI가 1보다 작을 수 있다는 점이 중요합니다.

현대 superscalar CPU는 한 cycle에 여러 명령어를 완료할 수 있기 때문입니다.

---

### 8.5 실행 시간의 기본 공식

CPU 실행 시간은 개념적으로 다음과 같이 표현할 수 있습니다.

```text
CPU time
= Instruction count × CPI × Clock cycle time
```

또는:

```text
CPU time
= Instruction count × CPI / Clock frequency
```

예를 들어:

```text
Instruction count = 1,000,000
Average CPI       = 1.5
Clock frequency   = 3 GHz
```

필요 cycle 수:

```text
1,000,000 × 1.5
= 1,500,000 cycles
```

실행 시간:

```text
1,500,000 / 3,000,000,000
= 0.0005 second
= 0.5 ms
```

이 공식은 전체 프로그램의 현실을 완벽하게 표현하지는 않지만, 성능 분석의 출발점입니다.

---

### 8.6 Dependency chain

다음 코드를 보겠습니다.

```c
x = a + b;
y = x + c;
z = y + d;
```

두 번째 덧셈은 첫 번째 결과가 필요합니다.

```text
a + b → x
        ↓
      x + c → y
              ↓
            y + d → z
```

이를 dependency chain이라고 합니다.

CPU에 여러 ALU가 있어도 이 계산들은 서로 결과를 기다려야 하므로 완전히 동시에 실행할 수 없습니다.

반면:

```c
x = a + b;
y = c + d;
z = e + f;
```

세 연산은 서로 독립적입니다.

```text
a + b → x
c + d → y
e + f → z
```

현대 CPU는 이런 독립 명령어를 동시에 실행할 가능성이 있습니다.

이것이 이후 다룰 명령어 수준 병렬성(Instruction-Level Parallelism, ILP)의 출발점입니다.

---

## 9. 실제 시스템 및 Embedded 응용

### 9.1 Linux에서 프로그램이 실행될 때

터미널에서 다음 명령을 실행했다고 합시다.

```bash
./add
```

개념적으로 다음 과정이 일어납니다.

```text
1. Shell이 실행 파일을 실행해 달라고 kernel에 요청한다.
2. Kernel이 실행 파일 형식을 확인한다.
3. 코드와 데이터를 process의 virtual address space에 매핑한다.
4. Stack과 필요한 runtime 상태를 준비한다.
5. CPU register 상태를 초기화한다.
6. PC를 프로그램 진입점(entry point)으로 설정한다.
7. Scheduler가 해당 process를 CPU에서 실행시킨다.
```

CPU는 `main()`에서 바로 시작하지 않을 수도 있습니다.

일반적인 사용자 프로그램에는 runtime 초기화 코드가 존재합니다.

```text
Entry point
   ↓
C runtime initialization
   ↓
main()
   ↓
exit 처리
```

---

### 9.2 Microcontroller에서는

Bare-metal microcontroller에서는 운영체제가 없을 수 있습니다.

전원이 들어오면 일반적으로 다음과 같은 과정이 시작됩니다.

```text
Power-on / Reset
       ↓
Reset vector 읽기
       ↓
초기 Stack Pointer 설정
       ↓
Reset handler 실행
       ↓
.data 영역 초기화
       ↓
.bss 영역 0으로 초기화
       ↓
System clock 및 peripheral 초기화
       ↓
main() 호출
```

Cortex-M에서는 vector table에 초기 stack pointer와 reset handler 주소가 포함되는 구조가 일반적입니다.

따라서 “CPU가 프로그램을 실행한다”는 말은 embedded 환경에서는 다음과 연결됩니다.

```text
Reset 직후 PC가 어디를 가리키는가?
Stack은 언제 설정되는가?
Global variable은 누가 초기화하는가?
main() 이전에 어떤 코드가 실행되는가?
```

---

### 9.3 Jetson과 같은 시스템에서는

Jetson 계열은 단순한 microcontroller보다 Linux 컴퓨터에 가깝습니다.

개념적인 부팅 흐름은 다음처럼 여러 단계가 있습니다.

```text
Boot ROM
  ↓
Bootloader
  ↓
Firmware / hardware initialization
  ↓
Linux kernel
  ↓
User-space initialization
  ↓
Application process
```

각 단계에서 CPU는 여전히 machine instruction을 실행합니다.

다만 실행 주체와 메모리 환경이 단계마다 달라집니다.

---

## 10. 아키텍처 비교

### 10.1 AArch64와 x86-64의 간단한 덧셈 비교

C 코드:

```c
int add(int a, int b)
{
    return a + b;
}
```

AArch64의 개념적인 결과:

```asm
add w0, w0, w1
ret
```

System V x86-64 호출 규약의 개념적인 결과:

```asm
mov eax, edi
add eax, esi
ret
```

| 항목            | AArch64      | x86-64            |
| ------------- | ------------ | ----------------- |
| 첫 번째 `int` 인수 | `w0`         | `edi`             |
| 두 번째 `int` 인수 | `w1`         | `esi`             |
| 반환값           | `w0`         | `eax`             |
| 덧셈 형태         | 3-operand 가능 | 흔히 2-operand      |
| 일반적 명령어 길이    | 4바이트 고정      | 가변 길이             |
| `ret` 역할      | 반환 주소로 분기    | stack에서 반환 주소를 사용 |

### 왜 이런 차이가 생겼는가?

AArch64는 비교적 규칙적인 instruction encoding과 명확한 register 구조를 사용합니다.

x86-64는 이전 x86 세대와의 하위 호환성을 유지하면서 확장되었기 때문에:

```text
가변 길이 명령어
여러 addressing mode
암시적 operand
복잡한 encoding
```

등을 포함합니다.

### 현대 CPU에서는 왜 단순 비교가 어려운가?

겉으로 보이는 ISA 명령어와 내부 실행 구조는 다를 수 있습니다.

예를 들어 x86 CPU는 복잡한 x86 명령어를 내부적으로 더 단순한 micro-operation으로 변환할 수 있습니다.

```text
x86 instruction
      ↓ decode
여러 개의 internal µops
      ↓
Out-of-order execution engine
```

AArch64 CPU도 ISA가 단순하다고 해서 내부 hardware가 단순한 것은 아닙니다. 고성능 Cortex-A 또는 Apple 계열 CPU에는 복잡한 예측, rename, scheduling, speculation 구조가 들어갑니다.

---

## 11. 흔한 오해

### 오해 1: CPU는 C 코드를 실행한다

틀렸습니다.

CPU는 C 문법을 모릅니다.

```text
C source
  ↓ compiler
Assembly 또는 compiler IR 변환
  ↓ assembler / linker
Machine code
  ↓
CPU 실행
```

---

### 오해 2: Assembly가 CPU가 읽는 언어다

정확하지 않습니다.

Assembly는 사람이 machine instruction을 읽기 쉽게 표현한 문자열입니다.

```asm
add w0, w0, w1
```

CPU는 이 문자열이 아니라 인코딩된 비트를 읽습니다.

---

### 오해 3: 한 줄의 C 코드는 한 개의 CPU 명령어다

그렇지 않습니다.

한 줄의 C 코드가:

* 여러 명령어로 변환될 수 있고,
* 하나의 명령어로 변환될 수 있고,
* 완전히 제거될 수도 있으며,
* 함수 호출이나 library 코드로 변환될 수도 있습니다.

예:

```c
x = a / b;
```

정수 나눗셈 명령어 하나가 될 수도 있지만, 대상 CPU가 hardware division을 지원하지 않으면 여러 명령어나 runtime helper function으로 변환될 수 있습니다.

---

### 오해 4: CPU는 항상 명령어를 순서대로 하나씩 끝낸다

ISA 관점에서는 프로그램 순서와 동일한 결과를 만들어야 합니다.

그러나 실제 현대 CPU는 내부적으로:

```text
여러 명령어를 동시에 진행하고
순서를 바꿔 실행하고
분기 결과를 추측하며
결과를 임시로 보관할 수 있습니다.
```

즉:

```text
Architectural behavior ≠ 내부 실행 시간 순서
```

---

### 오해 5: 메모리에 저장된 비트에는 명령어·데이터 표시가 붙어 있다

일반적으로 비트 자체에는 “나는 명령어다”라는 태그가 없습니다.

같은 비트라도 CPU가 어떤 주소를 instruction fetch 경로로 읽는지, data load 경로로 읽는지에 따라 다르게 취급될 수 있습니다.

운영체제는 메모리 page 권한을 통해 실행 가능 여부를 제어할 수 있습니다.

```text
R = Read
W = Write
X = Execute
```

예를 들어 데이터 page를 non-executable로 설정하면 해당 위치를 명령어로 실행하려는 시도가 예외를 발생시킬 수 있습니다.

---

## 12. 실패 사례와 디버깅

### 12.1 잘못된 함수 포인터 호출

다음 코드는 위험합니다.

```c
#include <stdint.h>

int main(void)
{
    uintptr_t address = 0x12345678;
    void (*function)(void) = (void (*)(void))address;

    function();

    return 0;
}
```

문제:

* 주소가 유효한 executable memory가 아닐 수 있습니다.
* instruction alignment가 맞지 않을 수 있습니다.
* 해당 주소의 비트가 유효한 machine instruction이 아닐 수 있습니다.
* 호출 규약이 맞지 않을 수 있습니다.
* C 표준과 구현 환경에 따라 integer-to-function-pointer 변환의 의미가 달라집니다.

일반 사용자 프로그램에서는 segmentation fault나 illegal instruction이 발생할 수 있습니다.

---

### 12.2 Illegal instruction

CPU가 현재 ISA 또는 활성화된 extension에서 유효하지 않은 instruction encoding을 실행하면 illegal instruction 예외가 발생할 수 있습니다.

Linux에서는 흔히 다음 형태로 보일 수 있습니다.

```text
Illegal instruction
```

가능한 원인:

```text
다른 CPU용 binary 실행
지원되지 않는 SIMD extension 사용
손상된 executable
잘못된 function pointer
코드 영역 overwrite
JIT code generation 오류
```

---

### 12.3 Segmentation fault와 instruction fetch

Segmentation fault는 data access에서만 발생하는 것이 아닙니다.

PC가 다음과 같은 주소를 가리키면 instruction fetch 중에도 문제가 발생할 수 있습니다.

```text
매핑되지 않은 주소
실행 권한이 없는 page
권한이 부족한 주소
손상된 return address
```

예를 들어 stack corruption으로 반환 주소가 손상되면:

```text
ret
 ↓
잘못된 주소가 PC에 들어감
 ↓
instruction fetch 실패
 ↓
exception / signal
```

---

### 12.4 Undefined Behavior와 CPU 동작을 혼동하면 안 된다

다음 C 코드를 보겠습니다.

```c
int x = 2147483647;
x = x + 1;
```

일반적인 32비트 two’s-complement hardware에서는 비트가 다음처럼 wrap될 수 있습니다.

```text
0x7FFFFFFF + 1 = 0x80000000
```

하지만 C에서 signed integer overflow는 Undefined Behavior입니다.

따라서:

```text
하드웨어에서 가능한 동작
≠ C 표준이 보장하는 동작
```

컴파일러는 signed overflow가 발생하지 않는다고 가정하여 코드를 최적화할 수 있습니다.

반면 unsigned integer 연산은 modulo arithmetic으로 정의됩니다.

```c
#include <stdint.h>

uint32_t x = UINT32_MAX;
x = x + 1U;
```

결과는 표준상 다음과 같이 정의됩니다.

```text
x == 0
```

---

### 12.5 디버깅할 때 확인할 것

프로그램이 예상과 다르게 실행된다면 다음 계층을 분리해야 합니다.

```text
1. C/C++ 소스의 의미가 올바른가?
2. Undefined Behavior가 있는가?
3. 컴파일러가 어떤 assembly를 생성했는가?
4. 호출 규약을 지키고 있는가?
5. 메모리 주소와 권한이 올바른가?
6. CPU가 해당 instruction extension을 지원하는가?
7. Exception이나 interrupt가 실행 흐름을 바꾸었는가?
```

---

## 13. 확인 문제

정답은 지금 공개하지 않습니다. 답을 작성하면 문제별로 피드백하겠습니다.

### Level 1: 개념 확인

**문제 1**

CPU가 C 소스 코드를 직접 실행하지 못하는 이유를 설명하세요.

다음 용어를 포함하세요.

```text
Compiler
ISA
Machine code
```

---

**문제 2**

다음 요소의 역할을 각각 한 문장으로 설명하세요.

```text
Program Counter
Instruction decoder
Register file
ALU
```

---

### Level 2: 동작 과정 추적

**문제 3**

다음 AArch64 명령어가 실행되기 전 상태입니다.

```asm
add w3, w1, w2
```

```text
w1 = 18
w2 = 7
w3 = 100
PC = 0x2000
```

AArch64 명령어가 4바이트라고 할 때, 정상적으로 실행한 후 다음 값을 적으세요.

```text
w1 =
w2 =
w3 =
PC =
```

그리고 Fetch부터 PC update까지의 과정을 설명하세요.

---

**문제 4**

다음 명령어의 차이를 architectural state 관점에서 설명하세요.

```asm
add w0, w1, w2
```

```asm
str w0, [x1]
```

두 명령어가 각각 어떤 상태를 변경하는지 구분하세요.

---

### Level 3: 성능 계산 또는 코드 분석

**문제 5**

어떤 프로그램이 다음 조건으로 실행됩니다.

```text
Instruction count = 2,000,000
Average CPI = 1.2
Clock frequency = 2.4 GHz
```

다음을 계산하세요.

1. 총 cycle 수
2. 이상적인 CPU 실행 시간
3. 평균 IPC

단위 변환 과정도 적으세요.

---

**문제 6**

다음 두 코드 중 instruction-level parallelism을 더 많이 활용할 가능성이 있는 코드를 고르고 이유를 설명하세요.

코드 A:

```c
a = a + x;
a = a + y;
a = a + z;
```

코드 B:

```c
a = x + y;
b = z + w;
c = p + q;
```

---

### Level 4: 설계·디버깅·실무 응용

**문제 7**

프로그램에서 함수 호출 후 `ret`을 실행했는데 갑자기 실행 권한이 없는 주소에서 fault가 발생했습니다.

가능한 원인을 최소 3개 제시하고, 디버깅 순서를 설명하세요.

---

**문제 8**

개발자가 다음과 같이 주장했습니다.

> “CPU는 결국 signed integer도 2의 보수로 계산하므로, signed overflow는 항상 wrap-around한다.”

이 주장이 C 언어 관점과 hardware 관점에서 왜 부정확한지 설명하세요.

---

## 14. 실습 과제

### 실습 목표

다음 관계를 직접 확인합니다.

```text
C source
→ 최적화되지 않은 assembly
→ 최적화된 assembly
→ 실제 실행 결과
```

그리고 다음 질문에 답합니다.

> C 함수 하나가 항상 assembly 함수 하나로 남아 있는가?

---

### 준비물

다음 중 하나가 필요합니다.

```text
macOS + Apple Clang
Linux + GCC 또는 Clang
ARM Linux board + GCC
```

컴파일러 확인:

```bash
cc --version
```

---

### 코드

`instruction_demo.c`:

```c
#include <stdio.h>
#include <stdlib.h>

static int add(int a, int b)
{
    return a + b;
}

static int chained_add(int a, int b, int c, int d)
{
    int x = a + b;
    int y = x + c;
    int z = y + d;

    return z;
}

static int independent_add(int a, int b, int c, int d)
{
    int x = a + b;
    int y = c + d;

    return x + y;
}

int main(int argc, char **argv)
{
    if (argc != 5) {
        fprintf(
            stderr,
            "Usage: %s <a> <b> <c> <d>\n",
            argv[0]
        );
        return EXIT_FAILURE;
    }

    char *end = NULL;

    const long a_long = strtol(argv[1], &end, 10);
    if (*argv[1] == '\0' || *end != '\0') {
        fprintf(stderr, "Invalid integer: %s\n", argv[1]);
        return EXIT_FAILURE;
    }

    const long b_long = strtol(argv[2], &end, 10);
    if (*argv[2] == '\0' || *end != '\0') {
        fprintf(stderr, "Invalid integer: %s\n", argv[2]);
        return EXIT_FAILURE;
    }

    const long c_long = strtol(argv[3], &end, 10);
    if (*argv[3] == '\0' || *end != '\0') {
        fprintf(stderr, "Invalid integer: %s\n", argv[3]);
        return EXIT_FAILURE;
    }

    const long d_long = strtol(argv[4], &end, 10);
    if (*argv[4] == '\0' || *end != '\0') {
        fprintf(stderr, "Invalid integer: %s\n", argv[4]);
        return EXIT_FAILURE;
    }

    /*
     * 이 실습에서는 범위를 작게 제한해 int 변환과 signed overflow를
     * 피한다.
     */
    if (a_long < -1000000L || a_long > 1000000L ||
        b_long < -1000000L || b_long > 1000000L ||
        c_long < -1000000L || c_long > 1000000L ||
        d_long < -1000000L || d_long > 1000000L) {
        fprintf(stderr, "Each input must be between -1000000 and 1000000.\n");
        return EXIT_FAILURE;
    }

    const int a = (int)a_long;
    const int b = (int)b_long;
    const int c = (int)c_long;
    const int d = (int)d_long;

    printf("add:             %d\n", add(a, b));
    printf("chained_add:     %d\n", chained_add(a, b, c, d));
    printf("independent_add: %d\n", independent_add(a, b, c, d));

    return EXIT_SUCCESS;
}
```

---

### 컴파일 및 실행 방법

#### 실행 파일 생성

```bash
cc -O0 -Wall -Wextra -Wpedantic instruction_demo.c -o demo_O0
cc -O2 -Wall -Wextra -Wpedantic instruction_demo.c -o demo_O2
```

실행:

```bash
./demo_O0 10 20 30 40
./demo_O2 10 20 30 40
```

예상 출력:

```text
add:             30
chained_add:     100
independent_add: 100
```

---

### Assembly 생성

```bash
cc -O0 -S instruction_demo.c -o demo_O0.s
cc -O2 -S instruction_demo.c -o demo_O2.s
```

함수 이름 찾기:

```bash
grep -n "add" demo_O0.s
grep -n "add" demo_O2.s
```

macOS에서는 symbol 앞에 `_`가 붙을 수 있습니다.

```text
_add
_chained_add
_independent_add
```

---

### 예상 관찰 결과

`-O0`:

* 함수의 지역 변수가 stack에 저장될 수 있습니다.
* `load`와 `store`가 많이 보일 수 있습니다.
* 함수 호출 구조가 비교적 그대로 남을 가능성이 높습니다.

`-O2`:

* 작은 함수가 inline될 수 있습니다.
* 지역 변수가 register에만 존재할 수 있습니다.
* 불필요한 load/store가 제거될 수 있습니다.
* 별도 함수 symbol이 남아 있어도 실제 호출되지 않을 수 있습니다.
* 같은 수학적 결과를 더 적은 명령어로 계산할 수 있습니다.

---

### 관찰 결과가 나오는 이유

C 표준은 “반드시 이 instruction을 사용하라”고 요구하지 않습니다.

컴파일러는 다음 조건만 지키면 됩니다.

> 정의된 프로그램에서 관찰 가능한 동작을 동일하게 유지한다.

따라서 다음 변환이 가능합니다.

```text
함수 inline
상수 미리 계산
불필요한 변수 제거
register allocation
공통 계산 제거
명령어 재배치
```

---

### 잘못된 결과가 나올 수 있는 원인

1. 입력값이 너무 커 signed overflow가 발생한 경우
2. compiler 종류와 버전이 다른 경우
3. CPU ISA가 다른 경우
4. 함수가 완전히 inline되어 symbol을 찾지 못한 경우
5. macOS symbol naming 때문에 `_function`으로 표시된 경우
6. compiler가 dead-code elimination을 수행한 경우
7. debug build와 release build를 혼동한 경우

---

### `volatile`을 사용하지 않은 이유

이번 실습의 결과값은 `printf()`에 전달됩니다.

즉, 결과가 외부에서 관찰되므로 컴파일러가 계산 전체를 무조건 제거할 수 없습니다.

`volatile`은 단순히 benchmark 최적화를 방지하는 만능 도구가 아닙니다.

특히 다음 용도로 사용해서는 안 됩니다.

```text
멀티스레드 동기화
atomicity 보장
memory ordering 보장
race condition 해결
```

---

### 추가 도전 과제

`add`, `chained_add`, `independent_add` 앞에 다음 속성을 붙여 assembly 차이를 관찰하세요.

GCC/Clang:

```c
__attribute__((noinline))
```

예:

```c
__attribute__((noinline))
static int add(int a, int b)
{
    return a + b;
}
```

그리고 다음 두 assembly를 비교하세요.

```bash
cc -O2 -S instruction_demo.c -o inline_allowed.s
cc -O2 -fno-inline -S instruction_demo.c -o inline_disabled.s
```

관찰할 것:

```text
함수 호출 명령어가 존재하는가?
Stack frame이 생성되는가?
인수가 어떤 register에 들어가는가?
반환값은 어떤 register에 들어가는가?
```

---

## 15. 면접에서 설명하는 방법

### 15.1 30초 설명

답변 구조:

```text
입력 언어 구분
→ machine instruction
→ PC
→ fetch/decode/execute
→ architectural state 변화
```

예시:

> CPU는 C 코드를 직접 실행하지 않고, 컴파일된 machine instruction을 실행합니다. Program Counter가 다음 명령어 주소를 제공하면 CPU가 명령어를 fetch하고, decoder가 opcode와 operand를 해석합니다. 이후 register 값을 읽고 ALU나 load/store unit이 연산한 다음, 결과를 register나 memory에 반영하고 PC를 갱신합니다. 따라서 프로그램 실행은 명령어에 따라 architectural state가 연속적으로 변하는 과정이라고 설명할 수 있습니다.

---

### 15.2 2분 설명

논리 구조:

1. 소스 코드와 machine code 구분
2. ISA의 역할
3. 주요 CPU 상태
4. 한 명령어의 실행 흐름
5. 단순 모델과 현대 CPU의 차이

예시:

> C나 C++ 프로그램은 compiler와 assembler를 거쳐 특정 ISA의 machine code로 변환됩니다. ISA는 instruction encoding, register, operand, memory access 규칙처럼 software와 processor 사이의 계약을 정의합니다. 실행 시 Program Counter가 현재 instruction address를 제공하고, CPU는 해당 instruction bits를 instruction cache나 memory hierarchy에서 가져옵니다. Decoder는 opcode와 operand field를 분석하고, register file에서 source operand를 읽습니다. 이후 ALU, branch unit 또는 load/store unit이 작업을 수행하며 결과를 register, memory, condition flag 또는 PC에 반영합니다. 이 과정을 architectural state transition으로 볼 수 있습니다. 교과서에서는 명령어가 하나씩 순서대로 완료된다고 설명하지만, 현대 CPU는 pipeline, superscalar, out-of-order execution과 speculation을 사용합니다. 다만 최종적으로 software에 보이는 결과는 ISA가 규정한 프로그램 의미를 유지해야 합니다.

---

### 15.3 심화 꼬리 질문

면접에서 이어질 수 있는 질문:

```text
ISA와 microarchitecture는 무엇이 다른가?
PC는 항상 다음 instruction을 가리키는가?
x86 instruction은 CPU 내부에서 그대로 실행되는가?
한 C statement가 몇 개의 instruction으로 변환되는가?
Instruction과 data는 memory에서 어떻게 구분되는가?
C signed overflow와 hardware overflow는 어떻게 다른가?
CPU는 out-of-order로 실행하면서 어떻게 정확한 결과를 유지하는가?
```

좋은 답변은 단순 정의보다 계층을 구분해야 합니다.

```text
언어 표준
Compiler
ISA
Microarchitecture
Operating system
Physical hardware
```

---

## 16. 핵심 정리

### 반드시 기억해야 할 내용

1. **CPU는 C나 assembly 문자열을 직접 실행하지 않습니다.**

   ```text
   CPU가 실행하는 것 = ISA에 맞게 인코딩된 machine instruction
   ```

2. **Program Counter는 다음 instruction fetch 위치를 결정합니다.**

3. **명령어는 opcode와 operand 정보를 포함합니다.**

4. **명령어 실행은 architectural state의 변화입니다.**

   ```text
   현재 상태 + instruction → 다음 상태
   ```

5. 가장 단순한 실행 모델은 다음과 같습니다.

   ```text
   Fetch
     ↓
   Decode
     ↓
   Operand read
     ↓
   Execute
     ↓
   Result write
     ↓
   PC update
   ```

6. **한 줄의 C 코드와 하나의 machine instruction은 일대일 관계가 아닙니다.**

7. **Compiler optimization에 따라 함수나 계산 자체가 사라질 수 있습니다.**

8. **ISA가 정의하는 프로그램의 결과와 CPU 내부의 실제 실행 순서는 다를 수 있습니다.**

9. 현대 CPU는 여러 명령어를 겹치고 재배치하여 실행하지만, 최종 결과는 ISA가 규정한 의미를 유지해야 합니다.

10. **하드웨어에서 발생할 법한 동작과 C/C++ 표준이 보장하는 동작을 구분해야 합니다.**

---