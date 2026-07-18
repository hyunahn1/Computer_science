# Lecture 2. Process의 구성 요소와 Virtual Address Space

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> 프로세스가 생성되었다면, 그 내부에는 정확히 무엇이 들어 있으며 CPU는 그 메모리에 어떻게 접근하는가?

Lecture 1에서는 실행 파일과 프로세스를 구분했습니다.

```text
Executable file
→ 디스크에 저장된 정적인 명령어와 데이터

Process
→ 실행 상태, 주소 공간, 커널 자원을 가진 동적인 실행 객체
```

이번 Lecture에서는 그중 **주소 공간과 메모리 구조**를 집중적으로 다룹니다. 

오늘의 핵심 질문은 다음과 같습니다.

```text
가상 주소란 무엇인가?

프로세스마다 왜 독립적인 주소 공간이 필요한가?

Code, Data, BSS, Heap, Stack은 왜 나뉘는가?

malloc()으로 받은 주소는 어디에 속하는가?

Page table은 무엇을 기록하는가?

MMU와 TLB는 주소 변환에 어떻게 관여하는가?

서로 다른 프로세스가 같은 가상 주소를 사용해도 되는 이유는 무엇인가?

Segmentation fault는 어떤 상황에서 발생하는가?
```

---

## 2. 이전 Lecture와의 연결

Lecture 1의 전체 흐름은 다음과 같았습니다.

```text
Executable file
      |
      v
execve()
      |
      v
Kernel loader
      |
      v
Virtual address space 구성
      |
      v
Register와 stack 초기화
      |
      v
Scheduler가 task 선택
      |
      v
User mode 실행
```

여기서 아직 자세히 설명하지 않은 부분이 있습니다.

```text
Virtual address space 구성
```

커널은 실행 파일을 단순히 RAM의 빈 공간에 복사하지 않습니다.

대신 프로세스가 사용할 **가상 주소 공간**을 만들고, 각 가상 주소 영역이 어떤 파일이나 물리 메모리와 연결되는지를 관리합니다.

---

# 3. 전체 흐름 먼저 보기

CPU가 C 변수에 접근하는 전체 흐름을 먼저 보겠습니다.

```text
C expression
    |
    v
Compiler가 load/store instruction 생성
    |
    v
CPU가 virtual address 생성
    |
    v
MMU가 page table을 이용해 주소 변환
    |
    +-- TLB hit
    |      |
    |      v
    |   즉시 physical address 획득
    |
    +-- TLB miss
           |
           v
       Page table walk
           |
           +-- Mapping 존재
           |      |
           |      v
           |   TLB 갱신 후 접근
           |
           +-- Mapping 없음 또는 권한 위반
                  |
                  v
              Page fault exception
                  |
                  v
              Kernel page-fault handler
```

프로세스의 주소 공간은 개념적으로 다음과 같습니다.

```text
높은 가상 주소
+-----------------------------------+
| Kernel 영역 또는 접근 불가 영역   |
+-----------------------------------+
| User stack                        |
| 지역 변수, 함수 호출 frame        |
|                ↓ 성장 가능        |
+-----------------------------------+
|                                   |
| Memory-mapped area                |
| shared libraries                  |
| mmap() mappings                   |
| large malloc allocations          |
|                                   |
+-----------------------------------+
|                ↑ 성장 가능        |
| Heap                              |
| 동적 할당 객체                    |
+-----------------------------------+
| BSS                               |
| 초기값 없는 전역·정적 변수        |
+-----------------------------------+
| Data                              |
| 초기화된 전역·정적 변수           |
+-----------------------------------+
| Read-only data                    |
| 문자열 리터럴, 상수               |
+-----------------------------------+
| Text / Code                       |
| 실행 가능한 기계어                |
+-----------------------------------+
낮은 가상 주소
```

이 그림은 단순화된 모델입니다.

실제 배치는 다음 요인에 따라 달라집니다.

```text
CPU architecture
Operating system
ABI
PIE 여부
ASLR
Linker 설정
Dynamic libraries
malloc 구현
Thread 수
```

---

# 4. 직관적 설명

## 4.1 가상 주소 공간은 프로세스 전용 지도다

각 프로세스는 자신만의 메모리 지도를 가진다고 생각할 수 있습니다.

```text
Process A의 주소 지도
0x1000 → A의 코드
0x2000 → A의 데이터
0x3000 → A의 heap

Process B의 주소 지도
0x1000 → B의 코드
0x2000 → B의 데이터
0x3000 → B의 heap
```

두 프로세스가 모두 `0x3000`이라는 주소를 사용하더라도, 실제로는 서로 다른 물리 메모리에 연결될 수 있습니다.

```text
Process A virtual 0x3000
        |
        v
Physical page 0x8A000

Process B virtual 0x3000
        |
        v
Physical page 0xC4000
```

CPU가 사용하는 가상 주소는 현재 실행 중인 주소 공간의 page table을 기준으로 해석됩니다.

즉, 가상 주소만으로는 실제 RAM 위치를 알 수 없습니다.

---

## 4.2 왜 가상 주소가 필요한가

물리 주소를 프로그램이 직접 사용한다고 가정해 보겠습니다.

```text
Program A:
나는 RAM 0x100000부터 사용한다.

Program B:
나도 RAM 0x100000부터 사용한다.
```

두 프로그램이 같은 물리 메모리를 덮어쓸 수 있습니다.

이런 구조에서는 다음 문제가 발생합니다.

```text
프로세스 간 메모리 훼손

보안 경계 붕괴

프로그램을 어느 주소에 배치할지 사전 결정 필요

물리 RAM 배치 변화에 프로그램이 의존

연속된 큰 물리 메모리가 필요

메모리 공유와 보호 정책 구현이 어려움
```

가상 메모리는 프로그램에게 다음과 같은 환상을 제공합니다.

> 자신이 독립적이고 연속적인 큰 메모리 공간을 사용하는 것처럼 보인다.

커널과 MMU는 이 가상 공간을 실제 물리 메모리와 연결합니다.

---

# 5. 형식적 정의

## 5.1 Virtual Address

프로그램의 명령어가 사용하는 주소입니다.

예:

```c
int value = 10;
printf("%p\n", (void *)&value);
```

여기서 출력되는 주소는 일반적으로 **가상 주소**입니다.

프로그램은 보통 자신의 변수에 대응하는 실제 물리 RAM 주소를 직접 알지 못합니다.

---

## 5.2 Physical Address

실제 RAM 또는 시스템 메모리 버스에서 식별되는 주소입니다.

CPU의 메모리 관리 장치가 가상 주소를 물리 주소로 변환합니다.

```text
Virtual address
       |
       v
      MMU
       |
       v
Physical address
```

---

## 5.3 Virtual Address Space

하나의 프로세스가 사용할 수 있도록 정의된 전체 가상 주소 범위와 mapping의 집합입니다.

주소 공간은 단순한 숫자 범위만을 의미하지 않습니다.

각 영역에는 다음 정보가 연결됩니다.

```text
주소 시작과 끝

읽기·쓰기·실행 권한

Private 또는 shared 여부

연결된 파일

Anonymous memory 여부

Page 상태

Copy-on-Write 여부
```

---

## 5.4 Page

가상 메모리와 물리 메모리를 일정 크기의 블록으로 나눈 단위입니다.

일반적인 Linux 시스템에서는 기본 page 크기가 4 KiB인 경우가 많지만, 시스템과 설정에 따라 다를 수 있습니다.

```text
Virtual address space
+----------+
| Page 0   |
+----------+
| Page 1   |
+----------+
| Page 2   |
+----------+
```

물리 메모리도 page frame 단위로 관리됩니다.

```text
Virtual page
      |
      v
Physical page frame
```

---

## 5.5 Page Table

가상 page가 어떤 physical page frame에 대응하는지 기록하는 자료구조입니다.

개념적인 page table entry는 다음 정보를 포함할 수 있습니다.

```text
Physical page frame number

Present 여부

Readable 여부

Writable 여부

Executable 여부

User mode 접근 허용 여부

Accessed 여부

Dirty 여부

Copy-on-Write 관련 상태

Cache 정책
```

실제 비트 구성은 CPU architecture와 운영체제 구현에 따라 다릅니다.

---

# 6. Process의 구성 요소

프로세스는 크게 다음과 같이 볼 수 있습니다.

```text
Process
├── Virtual address space
├── Execution context
├── Kernel metadata
└── Resource references
```

## 6.1 Virtual Address Space

```text
Code
Read-only data
Writable data
BSS
Heap
Memory mappings
Stack
Shared libraries
```

## 6.2 Execution Context

실제로는 각 thread에 더 직접적으로 속하는 요소가 많습니다.

```text
Program Counter
Stack Pointer
General-purpose registers
Status register
FPU/SIMD state
```

## 6.3 Kernel Metadata

```text
PID
PPID
Process state
Scheduling information
Credentials
Signal information
Resource limits
Accounting information
```

## 6.4 Resource References

```text
File descriptor table
Current working directory
Root directory reference
Open file descriptions
Sockets
Pipes
Device references
```

오늘은 첫 번째인 virtual address space에 집중합니다.

---

# 7. 메모리 영역별 구조

## 7.1 Text 또는 Code 영역

컴파일된 기계어 명령어가 배치됩니다.

```c
int add(int a, int b)
{
    return a + b;
}
```

이 함수는 컴파일되면 기계어 명령어가 됩니다.

개념적 assembly:

```asm
mov eax, edi
add eax, esi
ret
```

이 명령어들이 text 영역에 매핑됩니다.

일반적인 권한:

```text
Read:    가능
Write:   불가능
Execute: 가능
```

표기:

```text
r-x
```

왜 쓰기 권한을 제거할까요?

```text
실수로 코드가 변경되는 것 방지

악성 코드 주입 난이도 증가

여러 프로세스가 같은 읽기 전용 code page 공유 가능

W^X 정책 적용 가능
```

W^X는 한 page가 동시에 writable하고 executable하지 않도록 제한하는 보안 원칙입니다.

---

## 7.2 Read-only Data 영역

다음과 같은 데이터가 배치될 수 있습니다.

```c
const char *message = "Hello";
```

문자열 리터럴 `"Hello"`는 일반적으로 읽기 전용 mapping에 들어갑니다.

잘못된 코드:

```c
char *message = "Hello";
message[0] = 'Y';
```

문자열 리터럴을 수정하려는 동작은 C에서 정의되지 않은 동작이며, 실제 환경에서는 segmentation fault가 발생할 수 있습니다.

수정 가능한 배열이 필요하다면:

```c
char message[] = "Hello";
message[0] = 'Y';
```

이 경우 배열 내용은 쓰기 가능한 저장 영역에 배치됩니다.

---

## 7.3 Data 영역

명시적인 초기값을 가진 전역 변수와 static 변수가 들어갑니다.

```c
int global_value = 42;

static int static_value = 100;
```

실행 파일에는 이 초기값이 저장되어 있어야 합니다.

```text
global_value의 초기값 42
static_value의 초기값 100
```

일반적으로 쓰기 가능한 영역입니다.

```text
rw-
```

---

## 7.4 BSS 영역

초기값이 없거나 0으로 초기화되는 전역·static 변수가 들어갑니다.

```c
int global_uninitialized;
static int static_zero;
int explicit_zero = 0;
```

개념적으로 초기값은 모두 0입니다.

```text
global_uninitialized = 0
static_zero = 0
explicit_zero = 0
```

BSS의 핵심 장점은 실행 파일에 수많은 0을 직접 저장할 필요가 없다는 점입니다.

예:

```c
static char buffer[100 * 1024 * 1024];
```

100 MiB 배열이 있다고 해서 실행 파일에 100 MiB의 0을 저장할 필요는 없습니다.

실행 파일에는 대략 다음 정보만 기록하면 됩니다.

```text
이 위치에 100 MiB 크기의 zero-initialized 영역이 필요하다.
```

커널과 loader는 실행 시 해당 범위를 0으로 초기화된 memory로 제공합니다.

---

## 7.5 Heap

동적 메모리 할당에 사용되는 영역입니다.

```c
int *value = malloc(sizeof(*value));
```

`malloc()`은 C library 함수입니다.

반드시 매번 직접 system call을 수행하는 것은 아닙니다.

흐름을 단순화하면:

```text
사용자 코드
    |
    v
malloc()
    |
    v
User-space allocator
    |
    +-- 이미 확보한 영역이 있으면 내부에서 반환
    |
    +-- 메모리가 부족하면
           |
           +-- brk() 계열
           |
           +-- mmap() 계열
```

따라서 다음 문장은 부정확합니다.

> `malloc()`을 호출하면 커널이 정확히 요청한 크기의 물리 메모리를 즉시 준다.

더 정확한 설명:

> `malloc()`은 사용자 공간 allocator가 가상 주소 범위를 관리해 반환하며, 필요할 때 커널로부터 더 큰 mapping을 확보한다. 실제 physical page는 최초 접근 시 할당될 수도 있다.

---

## 7.6 Memory-mapped Area

다음이 배치될 수 있습니다.

```text
Shared libraries

mmap()으로 매핑한 파일

Anonymous mmap

Shared memory

Large malloc allocations

Thread stacks
```

예를 들어 동적 연결된 `libc.so`는 프로세스 주소 공간에 mapping됩니다.

```text
Executable
libc.so
dynamic linker
other shared libraries
```

파일을 `mmap()`하면 파일 내용이 프로세스의 가상 주소 공간 일부로 연결됩니다.

```text
File offset
    |
    v
Virtual memory mapping
    |
    v
Program load/store instruction
```

---

## 7.7 User Stack

함수 호출과 지역 실행 상태에 사용됩니다.

다음 항목이 stack에 놓일 수 있습니다.

```text
지역 변수

함수 인자 일부

Return address

Saved registers

Stack frame information

임시 값
```

예:

```c
void function(int argument)
{
    int local = 10;
}
```

단순화된 stack frame:

```text
높은 주소
+-------------------+
| caller frame      |
+-------------------+
| return address    |
+-------------------+
| saved registers   |
+-------------------+
| argument 일부     |
+-------------------+
| local             |
+-------------------+
낮은 주소
```

실제 배치는 ABI와 compiler optimization에 따라 달라집니다.

지역 변수가 반드시 stack에 존재한다고 단정해서는 안 됩니다.

컴파일러는 지역 변수를 register에만 보관할 수도 있고, 최적화로 제거할 수도 있습니다.

---

# 8. 사용자 영역과 Kernel 경계

프로그램이 자신의 메모리에 접근할 때마다 system call을 호출하는 것은 아닙니다.

다음 코드를 봅시다.

```c
int value = 10;
value++;
```

일반적으로 이 코드는 user mode에서 직접 실행됩니다.

```text
사용자 영역에서 보이는 것:
변수를 읽고 수정한다.

Kernel 내부에서 일어나는 것:
정상 mapping이 이미 존재한다면 아무 일도 하지 않을 수 있다.

CPU 또는 하드웨어에서 일어나는 것:
load/store instruction 실행
MMU로 virtual address translation
권한 검사
cache와 memory 접근
```

반대로 mapping이 아직 준비되지 않았거나 권한이 없으면 exception이 발생합니다.

```text
CPU
 |
 | page fault exception
 v
Kernel
```

즉, 일반 메모리 접근은 system call이 아니지만, 실패하거나 지연된 처리가 필요하면 exception을 통해 커널에 진입할 수 있습니다.

---

# 9. System Call과 Page Fault의 차이

둘 다 kernel mode로 진입할 수 있지만 원인이 다릅니다.

## System Call

사용자 프로그램이 의도적으로 커널 서비스를 요청합니다.

```c
read(fd, buffer, size);
```

```text
User program의 명시적 요청
        |
        v
syscall instruction
        |
        v
Kernel
```

## Page Fault

CPU가 메모리 접근을 수행하다가 translation이나 permission 문제를 발견합니다.

```c
value = *pointer;
```

```text
Memory access
     |
     v
Mapping 없음 또는 권한 위반
     |
     v
CPU exception
     |
     v
Kernel page-fault handler
```

Page fault가 반드시 프로그램 오류라는 뜻은 아닙니다.

정상적인 page fault도 많습니다.

예:

```text
Demand paging

Copy-on-Write

Stack growth

File-backed page의 최초 접근
```

비정상적인 page fault도 있습니다.

```text
NULL pointer dereference

Unmapped address 접근

Read-only page에 write

Executable 권한 없는 page에서 실행
```

커널이 문제를 해결할 수 없으면 프로세스에 `SIGSEGV` 같은 signal을 전달할 수 있습니다.

---

# 10. MMU와 Page Table

## 10.1 MMU

MMU는 Memory Management Unit입니다.

CPU가 생성한 가상 주소를 물리 주소로 변환하고 접근 권한을 검사합니다.

```text
CPU core
   |
   | virtual address
   v
MMU
   |
   | physical address
   v
Cache / RAM
```

---

## 10.2 Virtual Address 분해

단순화된 page 구조에서는 virtual address를 두 부분으로 나눌 수 있습니다.

```text
Virtual address
+----------------------+-------------+
| Virtual page number  | Page offset |
+----------------------+-------------+
```

Page size가 4 KiB라면:

```text
4 KiB = 4096 bytes = 2^12
```

따라서 하위 12 bit가 page 내부 offset으로 사용될 수 있습니다.

```text
Virtual page number
→ 어느 page인가?

Page offset
→ 그 page 내부에서 몇 번째 byte인가?
```

Page table은 virtual page number를 physical page frame number로 변환합니다.

```text
Virtual page number
        |
        v
Page table
        |
        v
Physical frame number
```

최종 physical address:

```text
Physical frame number + 동일한 page offset
```

---

## 10.3 왜 다단계 Page Table을 사용하는가

64-bit 가상 주소 공간 전체에 대해 단일 거대한 배열을 만들면 page table 자체가 지나치게 커질 수 있습니다.

따라서 현대 CPU는 일반적으로 다단계 page table을 사용합니다.

개념적으로:

```text
Virtual address
   |
   +-- Level 1 index
   +-- Level 2 index
   +-- Level 3 index
   +-- Level 4 index
   +-- Offset
```

주소 공간에서 실제 사용하는 영역에 대해서만 하위 page table을 만들 수 있으므로 메모리를 절약합니다.

구체적인 단계 수는 architecture와 paging mode에 따라 달라집니다.

---

# 11. TLB

Page table을 매번 RAM에서 여러 단계로 읽으면 모든 메모리 접근이 매우 느려집니다.

이를 줄이기 위해 CPU는 최근 주소 변환 결과를 TLB에 캐시합니다.

```text
TLB
= Translation Lookaside Buffer
```

전체 흐름:

```text
Virtual address
      |
      v
TLB lookup
      |
      +-- Hit
      |    |
      |    v
      |  즉시 physical frame 확인
      |
      +-- Miss
           |
           v
       Page table walk
           |
           v
       TLB entry 저장
```

중요한 구분:

```text
TLB miss
≠
Page fault
```

TLB miss는 변환 정보가 TLB에 없다는 뜻입니다.

Page table에 유효한 mapping이 존재하면 hardware page-table walk로 해결할 수 있습니다.

Page fault는 page table상에서 현재 접근을 바로 완료할 수 없다는 뜻입니다.

---

# 12. 단계별 실행 추적

다음 코드를 생각해 봅시다.

```c
int global_value = 42;

int main(void)
{
    global_value++;
    return 0;
}
```

## 단계 1. Compiler가 주소 접근 명령 생성

개념적 assembly:

```asm
mov global_value(%rip), %eax
add $1, %eax
mov %eax, global_value(%rip)
```

실제 assembly는 architecture와 optimization에 따라 다릅니다.

---

## 단계 2. CPU가 virtual address 계산

CPU는 `global_value`가 위치한 가상 주소를 계산합니다.

```text
현재 instruction address
+
instruction에 포함된 offset
=
global_value의 virtual address
```

---

## 단계 3. TLB 조회

```text
global_value virtual page
        |
        v
TLB lookup
```

### TLB hit

물리 page frame이 즉시 확인됩니다.

### TLB miss

CPU가 page table walk를 수행합니다.

---

## 단계 4. 권한 검사

해당 page가 writable한지 확인합니다.

```text
Present: yes
User accessible: yes
Writable: yes
```

권한이 맞으면 store가 진행됩니다.

---

## 단계 5. Cache 접근

물리 주소를 기준으로 cache hierarchy를 탐색합니다.

```text
L1 cache
  |
  v
L2 cache
  |
  v
Last-level cache
  |
  v
RAM
```

---

## 단계 6. Dirty 상태

page가 수정되면 page table entry 또는 관련 하드웨어 상태에 dirty 정보가 표시될 수 있습니다.

파일-backed private mapping인지 anonymous page인지에 따라 이후 처리 방식이 달라집니다.

---

# 13. Demand Paging 실행 추적

프로그램이 처음 code page에 접근한다고 가정합니다.

```text
CPU가 instruction fetch 시도
        |
        v
Page table에서 page가 아직 resident하지 않음
        |
        v
Page fault exception
        |
        v
Kernel mode 진입
        |
        v
Page-fault handler
        |
        +-- 접근이 유효한 mapping인지 확인
        +-- Page cache 확인
        +-- 필요하면 storage에서 데이터 읽기
        +-- Physical page 연결
        +-- Page table entry 갱신
        +-- TLB 관련 처리
        |
        v
Fault를 일으킨 instruction 재실행
```

사용자 프로그램 입장에서는 해당 load나 instruction fetch가 그냥 완료된 것처럼 보입니다.

Page fault 처리 중 시간이 걸렸다는 사실은 일반 코드 흐름에서는 명시적으로 드러나지 않을 수 있습니다.

---

# 14. 실행 가능한 C 실습

## 실습 목표

다음 항목을 관찰합니다.

```text
함수 주소

문자열 리터럴 주소

초기화된 전역 변수 주소

BSS 변수 주소

Heap 객체 주소

Stack 변수 주소

Memory mapping 권한
```

## 전체 코드

파일 이름:

```text
address_space_demo.c
```

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_initialized = 42;
int global_uninitialized;
static int static_initialized = 100;
static int static_uninitialized;

static void print_addresses(void)
{
    const char *string_literal = "read-only string";
    char writable_array[] = "writable string";
    int local_variable = 10;
    static int function_static = 77;

    int *heap_value = malloc(sizeof(*heap_value));
    if (heap_value == NULL) {
        perror("malloc");
        exit(EXIT_FAILURE);
    }

    *heap_value = 1234;

    printf("PID: %ld\n\n", (long)getpid());

    printf("[Code]\n");
    printf("print_addresses          : %p\n",
           (void *)(uintptr_t)print_addresses);
    printf("main                     : %p\n",
           (void *)(uintptr_t)main);

    printf("\n[Read-only data candidate]\n");
    printf("string literal           : %p\n",
           (const void *)string_literal);

    printf("\n[Initialized data]\n");
    printf("global_initialized       : %p\n",
           (void *)&global_initialized);
    printf("static_initialized       : %p\n",
           (void *)&static_initialized);
    printf("function_static          : %p\n",
           (void *)&function_static);

    printf("\n[BSS / zero-initialized]\n");
    printf("global_uninitialized     : %p\n",
           (void *)&global_uninitialized);
    printf("static_uninitialized     : %p\n",
           (void *)&static_uninitialized);

    printf("\n[Heap]\n");
    printf("heap object              : %p\n",
           (void *)heap_value);

    printf("\n[Stack candidates]\n");
    printf("local_variable           : %p\n",
           (void *)&local_variable);
    printf("writable_array           : %p\n",
           (void *)writable_array);
    printf("heap pointer variable    : %p\n",
           (void *)&heap_value);

    printf("\nCommands to run in another terminal:\n");
    printf("cat /proc/%ld/maps\n", (long)getpid());
    printf("cat /proc/%ld/smaps_rollup\n", (long)getpid());
    printf("pmap -x %ld\n", (long)getpid());
    printf("readelf -l ./address_space_demo\n");

    printf("\nPress Enter to exit.\n");

    errno = 0;
    int result = getchar();

    if (result == EOF && ferror(stdin)) {
        perror("getchar");
        free(heap_value);
        exit(EXIT_FAILURE);
    }

    free(heap_value);
}

int main(void)
{
    print_addresses();
    return EXIT_SUCCESS;
}
```

위 코드에서는 `uintptr_t`를 사용하므로 다음 header가 필요합니다.

```c
#include <stdint.h>
```

따라서 완전한 header 부분은 다음과 같아야 합니다.

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
```

---

## 컴파일

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    address_space_demo.c -o address_space_demo
```

일부 compiler는 함수 포인터와 `void *` 변환에 대해 경고할 수 있습니다. 함수 주소 출력은 엄밀한 ISO C 이식성보다 POSIX/Linux 관찰 목적에 가깝습니다.

---

## 실행

```bash
./address_space_demo
```

예상되는 주소 경향:

```text
Code:
0x55...

Read-only data:
0x55...

Data / BSS:
0x55...

Heap:
0x55...

Stack:
0x7fff...
```

정확한 값과 순서는 보장되지 않습니다.

---

# 15. `/proc/<PID>/maps` 읽는 방법

실행 중인 PID가 5000이라면:

```bash
cat /proc/5000/maps
```

예시:

```text
55a10000-55a11000 r--p 00000000 ... /path/address_space_demo
55a11000-55a12000 r-xp 00001000 ... /path/address_space_demo
55a12000-55a13000 r--p 00002000 ... /path/address_space_demo
55a13000-55a14000 rw-p 00002000 ... /path/address_space_demo
55a30000-55a51000 rw-p 00000000 ... [heap]
7f120000-7f140000 r-xp 00000000 ... libc.so.6
7fff1000-7fff3000 rw-p 00000000 ... [stack]
```

각 필드의 개념:

```text
주소 범위
권한
파일 offset
device
inode
mapping 이름
```

권한:

| 문자  | 의미              |
| --- | --------------- |
| `r` | 읽기 가능           |
| `w` | 쓰기 가능           |
| `x` | 실행 가능           |
| `p` | Private mapping |
| `s` | Shared mapping  |

예:

```text
r-xp
```

의미:

```text
읽기 가능
쓰기 불가능
실행 가능
private mapping
```

---

# 16. ELF Segment와 Memory Mapping 연결

다음 명령을 실행합니다.

```bash
readelf -l ./address_space_demo
```

`LOAD` 항목을 확인합니다.

예상 형태:

```text
Type   Offset   VirtAddr   FileSiz   MemSiz   Flags
LOAD   ...      ...        ...       ...       R
LOAD   ...      ...        ...       ...       R E
LOAD   ...      ...        ...       ...       RW
```

대략적인 연결:

```text
R
→ 읽기 전용 metadata 또는 data

R E
→ 실행 가능한 code

RW
→ writable data와 BSS
```

`FileSiz`보다 `MemSiz`가 더 클 수 있습니다.

그 차이는 BSS처럼 파일에는 실제 byte가 없지만 메모리에서는 0으로 초기화되어야 하는 영역일 수 있습니다.

```text
MemSiz > FileSiz
        |
        v
파일에 저장되지 않은 zero-filled 영역 존재 가능
```

---

# 17. `malloc()`은 정확히 무엇을 반환하는가

다음 코드:

```c
int *pointer = malloc(sizeof(*pointer));
```

`malloc()`이 반환하는 것은 다음입니다.

> 최소 요청 크기 이상을 사용할 수 있는 정렬된 저장 영역의 시작 주소.

`malloc()`은 다음을 보장하지 않습니다.

```text
정확히 4 byte만 kernel에서 받았다는 것

새 physical page가 즉시 할당되었다는 것

반드시 전통적인 heap segment에 있다는 것

주소가 연속된 physical memory라는 것

메모리가 0으로 초기화되었다는 것
```

0으로 초기화된 동적 메모리가 필요하면:

```c
int *pointer = calloc(1, sizeof(*pointer));
```

그러나 `calloc()` 역시 physical page가 즉시 모두 준비되었다고 단정할 수는 없습니다.

---

# 18. Stack은 왜 제한되어 있는가

Stack이 무한정 커질 수 있다면 다른 mapping과 충돌할 수 있습니다.

운영체제는 일반적으로 stack 크기에 제한을 둡니다.

확인:

```bash
ulimit -s
```

출력 단위는 shell과 시스템에 따라 보통 KiB입니다.

예:

```text
8192
```

이는 약 8 MiB stack 제한을 의미할 수 있습니다.

재귀 호출이 지나치게 깊거나 큰 지역 배열을 만들면 stack overflow가 발생할 수 있습니다.

```c
void recurse(void)
{
    char buffer[1024 * 1024];
    buffer[0] = 1;
    recurse();
}
```

이 코드는 안전하지 않으며 결국 stack limit를 넘어설 수 있습니다.

커널이 인접한 유효 mapping으로 stack을 확장할 수 없는 경우 page fault를 해결하지 못하고 `SIGSEGV`가 발생할 수 있습니다.

---

# 19. Segmentation Fault의 실제 의미

Segmentation fault는 단순히 “잘못된 포인터 오류”라는 언어 수준 개념이 아닙니다.

일반적으로 다음 흐름으로 이해할 수 있습니다.

```text
User instruction이 memory access 수행
        |
        v
CPU가 page translation 또는 permission 검사
        |
        v
Page fault exception
        |
        v
Kernel이 주소와 접근 유형 검사
        |
        +-- 합법적 접근
        |      |
        |      v
        |   Mapping 또는 page 준비
        |
        +-- 불법 접근
               |
               v
           SIGSEGV 전달
               |
               v
           기본 동작으로 process 종료 가능
```

대표 원인:

```text
NULL pointer dereference

해제된 메모리 접근

배열 범위 초과

Read-only memory에 write

Stack overflow

잘못된 함수 포인터 실행
```

---

# 20. 실패 사례

## 20.1 NULL pointer 접근

```c
int *pointer = NULL;
*pointer = 10;
```

일반적으로 낮은 주소는 mapping되지 않으므로 page fault가 발생하고 `SIGSEGV`로 이어질 수 있습니다.

하지만 C 언어 관점에서는 NULL pointer 역참조 자체가 undefined behavior입니다.

---

## 20.2 문자열 리터럴 수정

```c
char *text = "hello";
text[0] = 'H';
```

문자열 리터럴 수정은 undefined behavior입니다.

read-only mapping에 배치된 환경에서는 write permission fault가 발생할 수 있습니다.

---

## 20.3 Use-after-free

```c
int *pointer = malloc(sizeof(*pointer));

if (pointer == NULL) {
    return EXIT_FAILURE;
}

free(pointer);
*pointer = 42;
```

`free()` 이후 포인터가 가리키던 객체의 lifetime은 종료됩니다.

그 주소가 여전히 mapping되어 있을 수 있기 때문에 즉시 segmentation fault가 발생하지 않을 수도 있습니다.

이 점이 use-after-free를 특히 위험하게 만듭니다.

```text
잘못된 접근
≠
항상 즉시 crash
```

---

## 20.4 큰 지역 배열

```c
int main(void)
{
    char buffer[100 * 1024 * 1024];
    buffer[0] = 1;
    return 0;
}
```

기본 stack limit보다 크면 stack overflow가 발생할 수 있습니다.

큰 buffer가 필요하면 heap이나 적절한 memory mapping을 검토합니다.

---

# 21. 동시성과 Address Space

같은 프로세스의 thread들은 일반적으로 같은 virtual address space를 공유합니다.

```text
Process
├── Thread A
├── Thread B
└── Thread C

공유:
Code
Global variables
Heap
Memory mappings
```

따라서 한 thread가 heap 객체를 수정하면 다른 thread가 같은 주소로 그 변경을 볼 수 있습니다.

```c
int counter = 0;
```

두 thread가 다음을 실행한다고 가정합니다.

```c
counter++;
```

이 연산은 하나의 불가분 명령이 아닐 수 있습니다.

```text
Load counter
Add 1
Store counter
```

가능한 interleaving:

```text
Thread A: load 0
Thread B: load 0
Thread A: add → 1
Thread B: add → 1
Thread A: store 1
Thread B: store 1
```

기대값은 2였지만 결과는 1이 될 수 있습니다.

C/C++ 언어 모델에서는 비원자적 공유 객체를 동기화 없이 읽고 쓰는 data race가 undefined behavior가 될 수 있습니다.

이 문제는 Lecture 33 이후에 자세히 다룹니다.

---

# 22. Process 격리와 공유

서로 다른 프로세스는 기본적으로 독립 주소 공간을 가집니다.

```text
Process A heap
≠
Process B heap
```

따라서 Process A가 자신의 전역 변수를 수정해도 Process B의 전역 변수는 자동으로 변경되지 않습니다.

하지만 명시적인 공유는 가능합니다.

```text
Shared memory

Shared file mapping

File

Pipe

Socket
```

예를 들어 `mmap()`의 shared mapping을 사용하면 두 프로세스의 서로 다른 virtual address가 같은 physical page를 가리킬 수 있습니다.

```text
Process A virtual address ----+
                              |
                              v
                        Shared physical page
                              ^
                              |
Process B virtual address ----+
```

가상 주소가 서로 같을 필요도 없습니다.

---

# 23. 성능과 자원 비용

## 23.1 Page Table 비용

프로세스마다 독립 주소 공간을 제공하려면 page table 관련 메모리가 필요합니다.

주소 공간이 클수록 항상 page table이 동일 비율로 커지는 것은 아니지만, 실제로 mapping된 page가 많아질수록 관리 비용이 증가할 수 있습니다.

---

## 23.2 Page Fault 비용

정상적인 메모리 접근보다 page fault 처리는 훨씬 복잡합니다.

```text
CPU exception
Kernel 진입
VMA 탐색
권한 확인
Page 준비
Page table 갱신
TLB 처리
명령 재시작
```

파일 I/O까지 필요하면 비용이 더 커집니다.

---

## 23.3 TLB 비용

프로세스가 많은 page를 무작위로 접근하면 TLB locality가 나빠질 수 있습니다.

```text
TLB miss 증가
→ Page-table walk 증가
→ Memory access latency 증가
```

---

## 23.4 Cache 비용

가상 주소 translation이 성공해도 data가 CPU cache에 없으면 RAM 접근이 필요할 수 있습니다.

가상 메모리 비용과 cache 비용은 서로 다른 계층의 문제입니다.

```text
TLB
→ 주소 변환 cache

L1/L2/L3 cache
→ 명령어와 데이터 cache
```

---

## 23.5 Overcommit과 실제 메모리

큰 가상 주소 공간을 예약했다고 해서 같은 크기의 RAM이 즉시 소비되는 것은 아닙니다.

```text
Virtual memory reserved
≠
Physical memory resident
```

따라서 다음 값을 구분해야 합니다.

```text
VmSize
→ 전체 virtual memory 범위

VmRSS
→ 현재 resident physical memory 규모
```

정확한 의미와 집계 방식은 운영체제와 도구에 따라 세부 차이가 있습니다.

---

# 24. Linux와 다른 운영체제 비교

| 개념           | Linux              | macOS                     | Windows                      |
| ------------ | ------------------ | ------------------------- | ---------------------------- |
| 실행 파일 형식     | ELF                | Mach-O                    | PE                           |
| Mapping 관찰   | `/proc/PID/maps`   | `vmmap`                   | VMMap, Process Explorer      |
| 주소 공간 API    | `mmap`, `brk`      | `mmap`, Mach VM           | `VirtualAlloc`, file mapping |
| 보호 변경        | `mprotect`         | `mprotect`, Mach VM       | `VirtualProtect`             |
| Page size 확인 | `getconf PAGESIZE` | `sysctl`, `getpagesize()` | `GetSystemInfo`              |

세 운영체제 모두 다음 핵심 원리는 공유합니다.

```text
Process별 virtual address space

Page 단위 관리

Access permission

Demand paging

Shared mapping

CPU MMU 사용
```

API와 내부 자료구조는 다르지만 해결하려는 하드웨어 문제는 유사합니다.

---

# 25. 흔한 오해

## 오해 1. 포인터는 물리 주소다

일반적인 user-space 포인터는 가상 주소입니다.

---

## 오해 2. Heap은 항상 하나의 연속된 영역이다

Allocator는 `brk()`와 `mmap()` 등 여러 방식으로 메모리를 확보할 수 있습니다.

큰 allocation은 별도 mapping에 놓일 수 있습니다.

---

## 오해 3. 지역 변수는 항상 stack에 있다

Compiler optimization에 따라 register에만 존재하거나 제거될 수 있습니다.

---

## 오해 4. `malloc()`은 system call이다

`malloc()`은 library function입니다.

필요할 때 `brk()` 또는 `mmap()` 같은 kernel interface를 사용할 수 있지만, 호출마다 system call을 한다고 단정할 수 없습니다.

---

## 오해 5. Page fault는 항상 오류다

Demand paging과 Copy-on-Write에서도 정상적으로 발생합니다.

---

## 오해 6. TLB miss는 page fault다

TLB miss는 translation cache miss입니다.

Page table에 mapping이 있으면 page fault 없이 해결될 수 있습니다.

---

## 오해 7. 같은 virtual address는 같은 메모리다

각 프로세스의 page table이 다르면 다른 physical page를 가리킬 수 있습니다.

---

## 오해 8. 가상 메모리 크기는 실제 RAM 사용량이다

`VmSize`와 `VmRSS`는 다릅니다.

---

# 26. 실습 과제

## 실습 1. 주소와 Mapping 연결하기

### 목표

프로그램이 출력한 주소가 `/proc/<PID>/maps`의 어느 영역에 포함되는지 찾습니다.

### 수행

```bash
./address_space_demo
```

다른 terminal에서:

```bash
cat /proc/<PID>/maps
```

다음 주소를 mapping과 연결하세요.

```text
main
string literal
global_initialized
global_uninitialized
heap object
local_variable
```

### 관찰 질문

```text
main이 포함된 mapping의 권한은 무엇인가?

Global variable mapping에는 write 권한이 있는가?

Heap에 [heap] 이름이 표시되는가?

Stack에 [stack] 이름이 표시되는가?

Shared library는 어느 주소대에 위치하는가?
```

---

## 실습 2. BSS 크기 관찰

다음 전역 배열을 추가합니다.

```c
static char large_zero_buffer[64 * 1024 * 1024];
```

컴파일 후 파일 크기를 확인합니다.

```bash
ls -lh ./address_space_demo
size ./address_space_demo
readelf -S ./address_space_demo
```

질문:

```text
64 MiB 배열을 추가했는데 executable 크기도 64 MiB 증가했는가?

.bss 크기는 어떻게 변했는가?

실행 직후 VmRSS도 즉시 64 MiB 증가했는가?
```

배열이 최적화되거나 실제로 참조되지 않는 문제를 피하려면 다음처럼 사용합니다.

```c
large_zero_buffer[0] = 1;
large_zero_buffer[sizeof(large_zero_buffer) - 1] = 2;
```

---

## 실습 3. Minor Page Fault 관찰

Linux에서 다음 명령을 사용할 수 있습니다.

```bash
/usr/bin/time -v ./address_space_demo
```

출력 중 확인:

```text
Minor page faults
Major page faults
Maximum resident set size
```

일반적 의미:

```text
Minor fault
→ Disk I/O 없이 해결할 수 있었던 page fault

Major fault
→ Storage I/O가 필요했던 page fault
```

정확한 수치는 page cache 상태와 시스템 환경에 따라 크게 달라집니다.

---

# 27. 면접에서 설명하는 방법

## 30초 설명

> 프로세스는 자신만의 virtual address space를 가지며, 그 안에는 code, data, BSS, heap, memory-mapped area, stack 등이 배치됩니다. 프로그램이 사용하는 포인터는 일반적으로 가상 주소이고, CPU의 MMU가 page table을 이용해 물리 주소로 변환합니다. 최근 변환은 TLB에 캐시됩니다. Mapping이 아직 준비되지 않았거나 권한이 맞지 않으면 page fault가 발생하며, 커널이 demand paging 같은 정상 상황을 처리하거나 잘못된 접근에 대해 `SIGSEGV`를 전달할 수 있습니다.

## 2분 설명

> Virtual address space는 프로세스마다 독립된 메모리 지도를 제공하는 추상화입니다. Code 영역에는 기계어가 들어가며 보통 읽기와 실행 권한만 가지고, data에는 초기화된 전역·static 변수, BSS에는 0으로 초기화되는 전역·static 변수가 들어갑니다. Heap은 동적 할당에 사용되고, stack은 함수 호출과 지역 실행 상태를 관리합니다. CPU가 memory instruction을 실행하면 virtual address를 만들고 MMU가 TLB와 page table을 통해 physical address로 변환합니다. TLB에 변환이 없으면 page-table walk를 수행하고, page가 아직 resident하지 않거나 권한 문제가 있으면 page fault exception으로 kernel에 진입합니다. 정상적인 demand paging이면 kernel이 page를 준비한 뒤 instruction을 재실행하고, 불법 접근이면 `SIGSEGV`가 발생할 수 있습니다.

## 심화 꼬리 질문

```text
BSS가 executable file 크기를 크게 늘리지 않는 이유는 무엇인가?

TLB miss와 page fault는 어떻게 다른가?

malloc()은 왜 항상 system call을 호출하지 않는가?

VmSize와 VmRSS는 어떻게 다른가?

같은 virtual address가 다른 physical address를 가리킬 수 있는 이유는 무엇인가?

String literal 수정이 왜 위험한가?

Heap allocation이 반드시 [heap] 영역에 존재하지 않는 이유는 무엇인가?

Page table entry에는 어떤 permission 정보가 들어가는가?

Demand paging에서 fault instruction은 어떻게 처리되는가?
```

---

# 28. 확인 문제

정답은 바로 공개하지 않습니다.

## Level 1. 개념 확인

### 문제 1

다음 개념을 각각 구분하세요.

```text
Virtual address
Physical address
Virtual address space
Page
Page table
```

### 문제 2

다음 객체가 일반적으로 어느 영역에 위치하는지 답하세요.

```c
int initialized = 10;
int uninitialized;
static int static_value;
const char *text = "hello";

int main(void)
{
    int local = 20;
    int *dynamic = malloc(sizeof(*dynamic));
}
```

대상:

```text
initialized
uninitialized
static_value
문자열 리터럴 "hello"
local
dynamic 변수 자체
malloc으로 할당된 객체
main의 기계어
```

---

## Level 2. 주소 변환 추적

### 문제 3

CPU가 가상 주소를 사용해 memory load를 수행할 때 다음 단계를 올바른 순서로 배열하세요.

```text
A. Physical address로 cache 또는 memory 접근
B. TLB 조회
C. CPU가 virtual address 생성
D. TLB miss 시 page-table walk
E. Permission 확인
```

### 문제 4

다음 두 상황의 차이를 설명하세요.

```text
TLB miss
Page fault
```

---

## Level 3. 코드와 시스템 분석

### 문제 5

다음 코드가 있습니다.

```c
static char buffer[100 * 1024 * 1024];

int main(void)
{
    buffer[0] = 1;
    return 0;
}
```

실행 파일 크기가 반드시 100 MiB 이상이 되지 않는 이유를 BSS 관점에서 설명하세요.

### 문제 6

두 프로세스에서 지역 변수 주소가 모두 `0x7fffffffe000`으로 출력되었습니다.

다음 주장에 답하세요.

> 두 지역 변수는 같은 RAM 위치를 사용한다.

이 주장이 성립하지 않는 이유를 page table과 address space 관점에서 설명하세요.

---

## Level 4. Kernel·실패 분석

### 문제 7

프로그램이 아직 physical memory에 올라오지 않은 정상적인 executable code page를 처음 실행하려 합니다.

다음 항목을 사용해 전체 흐름을 설명하세요.

```text
Instruction fetch
Page fault
Kernel
File-backed mapping
Physical page
Page table update
Instruction retry
```

### 문제 8

다음 코드가 반드시 즉시 segmentation fault를 발생시키는지 분석하세요.

```c
int *pointer = malloc(sizeof(*pointer));

if (pointer == NULL) {
    return 1;
}

free(pointer);
*pointer = 42;
```

다음 관점을 포함하세요.

```text
C object lifetime
Undefined behavior
Virtual mapping
Allocator
즉시 crash 여부
```

---

# 29. 핵심 정리

```text
1. 프로세스는 독립적인 virtual address space를 가진다.

2. 프로그램이 출력하는 포인터는 일반적으로 virtual address다.

3. Virtual address는 MMU와 page table을 통해 physical address로 변환된다.

4. TLB는 최근 주소 변환을 저장하는 CPU cache다.

5. TLB miss와 page fault는 서로 다른 사건이다.

6. Text 영역에는 기계어가 배치되며 보통 read-execute 권한을 가진다.

7. Data에는 초기화된 전역·static 변수가 들어간다.

8. BSS에는 0으로 초기화되는 전역·static 변수가 들어간다.

9. BSS는 0의 전체 내용을 executable file에 저장할 필요가 없다.

10. Heap은 동적 할당에 사용되지만 모든 allocation이 하나의
    전통적인 heap mapping에 존재한다고 단정할 수 없다.

11. malloc()은 library function이며 매 호출마다 system call을
    실행하는 것은 아니다.

12. Stack은 함수 호출과 지역 실행 상태에 사용되지만,
    지역 변수가 항상 실제 stack memory에 존재하는 것은 아니다.

13. Page fault는 demand paging처럼 정상적인 상황에서도 발생한다.

14. Kernel이 page fault를 해결할 수 없으면 SIGSEGV가 전달될 수 있다.

15. 서로 다른 프로세스의 동일한 virtual address는 서로 다른
    physical page에 대응할 수 있다.

16. 같은 프로세스의 thread들은 일반적으로 같은 address space를
    공유하므로 공유 데이터에 race condition이 발생할 수 있다.

17. VmSize는 virtual memory 규모이고 VmRSS는 resident physical
    memory와 관련된 값이다.

18. 가상 메모리는 격리, 보호, 공유, demand paging을 가능하게 하는
    운영체제의 핵심 추상화다.
```