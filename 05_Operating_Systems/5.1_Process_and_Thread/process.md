## Part 1. Program에서 Process까지

1. **프로그램, 실행 파일, 프로세스의 차이**
2. Process의 구성 요소와 Virtual Address Space
3. PID, PPID와 Kernel Process Metadata
4. User Mode, Kernel Mode와 System Call
5. Process State와 Scheduler의 기본 구조

## Part 2. Process 생성과 실행

6. `fork()`의 사용자 관점
7. `fork()`의 Kernel 내부 과정
8. Copy-on-Write와 Page Fault
9. `execve()`와 프로그램 Image 교체
10. Shell의 `fork-exec-wait` 구조
11. Windows `CreateProcess()`와 비교

## Part 3. Thread 기초

12. Thread란 무엇인가
13. Process와 Thread가 공유하는 자원
14. Thread Stack과 Register Context
15. User-Level Thread와 Kernel-Level Thread
16. Linux Thread와 Scheduling Entity
17. `pthread_create()` 실행 과정
18. `pthread_join()`, detach와 Thread Lifetime

## Part 4. Context Switch와 Scheduling

19. Context란 무엇인가
20. Context Switch의 전체 과정
21. Timer Interrupt와 Preemption
22. Ready Queue, Wait Queue와 Wake-up
23. Blocking System Call과 Thread Sleep
24. Process Switch와 Thread Switch의 비용
25. TLB와 Cache의 영향

## Part 5. 종료와 자원 회수

26. `return`, `exit()`, `_exit()`
27. Process 종료 시 Kernel Cleanup
28. Zombie Process
29. Orphan Process와 init
30. `wait()`와 `waitpid()`
31. `SIGCHLD`와 여러 Child 관리
32. Thread 종료와 Process 종료

## Part 6. 동시성과 실무 문제

33. 공유 Address Space와 Data Race
34. `counter++`가 안전하지 않은 이유
35. Mutex와 Atomic
36. Deadlock과 Lock Ordering
37. Thread Stack Overflow
38. False Sharing
39. Multithreaded Process에서의 `fork()`
40. Process/Thread Debugging 종합 실습

---

# Lecture 1. 프로그램, 실행 파일, 프로세스의 차이

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답하는 것이 목표입니다.

> 소스 코드를 컴파일해서 생성된 실행 파일은 어떻게 CPU에서 실행되는 프로세스가 되는가?

이를 위해 반드시 다음 개념을 구분해야 합니다.

```text
Source code
Program
Executable file
Process
Process image
Address space
Execution context
Kernel metadata
```

오늘의 핵심 결론은 다음과 같습니다.

> 실행 파일은 디스크에 저장된 수동적인 데이터이고, 프로세스는 CPU 실행 상태, 가상 메모리, 파일, 권한, 스케줄링 정보 등을 포함해 커널이 관리하는 동적인 실행 객체다.

---

## 2. 전체 흐름 먼저 보기

C 프로그램 하나가 실행되는 큰 흐름을 먼저 보겠습니다.

```text
C source code
    |
    | compiler
    v
Object file
    |
    | linker
    v
Executable file
    |
    | shell이 실행 요청
    v
fork() 또는 기존 process
    |
    | execve() system call
    v
Kernel executable loader
    |
    +-- Executable 형식 검사
    +-- Virtual address space 구성
    +-- Code/Data segment mapping
    +-- User stack 구성
    +-- Dynamic linker mapping
    +-- Register 초기화
    |
    v
Runnable task
    |
    | scheduler가 CPU 할당
    v
User mode에서 entry point 실행
    |
    v
C runtime startup code
    |
    v
main(argc, argv)
```

여기서 중요한 것은 다음과 같습니다.

```text
main()이 프로그램 실행의 진짜 시작점은 아니다.
```

커널은 실행 파일 헤더에 기록된 **entry point**로 실행을 시작합니다. 일반적인 C 프로그램에서는 그 entry point가 C 런타임의 시작 코드이고, 런타임 코드가 초기화를 수행한 뒤 `main()`을 호출합니다.

---

## 3. 프로그램이란 무엇인가

### 3.1 일상적인 의미

일상적으로는 소스 코드, 실행 파일, 실행 중인 프로그램을 모두 “프로그램”이라고 부릅니다.

하지만 운영체제를 공부할 때는 이들을 구분해야 합니다.

### 3.2 형식적인 구분

#### Source code

사람이 작성한 코드입니다.

```c
#include <stdio.h>

int main(void)
{
    puts("Hello, process");
    return 0;
}
```

이 파일 자체는 CPU가 직접 실행할 수 있는 명령어가 아닙니다.

#### Object file

컴파일러가 소스 코드를 기계어로 번역한 중간 결과입니다.

```text
hello.c
  |
  | gcc -c
  v
hello.o
```

Object file에는 다음과 같은 정보가 포함될 수 있습니다.

```text
기계어 명령어
심볼 정보
재배치 정보
디버깅 정보
읽기 전용 데이터
초기화된 전역 데이터
```

그러나 아직 외부 함수나 라이브러리 참조가 완전히 연결되지 않았을 수 있습니다.

#### Executable file

링커가 object file과 필요한 라이브러리 정보를 조합해 만든 실행 가능한 파일입니다.

Linux에서는 일반적으로 ELF 형식을 사용합니다.

```text
ELF: Executable and Linkable Format
```

실행 파일은 다음과 같은 정보를 담습니다.

```text
어떤 CPU 아키텍처용인지
어디서부터 실행해야 하는지
어떤 영역을 메모리에 매핑해야 하는지
필요한 동적 라이브러리가 무엇인지
코드와 데이터가 파일의 어디에 있는지
```

하지만 실행 파일 자체는 여전히 디스크에 저장된 바이트 배열입니다.

CPU를 사용하지 않습니다.

스케줄러에도 등록되지 않습니다.

PID도 없습니다.

---

## 4. 실행 파일은 왜 프로세스가 아닌가

다음 실행 파일이 있다고 가정하겠습니다.

```bash
./hello
```

이 파일을 실행하지 않은 상태에서는 다음 특성을 가집니다.

```text
디스크에 존재한다.
inode와 파일 경로가 있다.
코드와 초기 데이터가 저장되어 있다.
실행 권한을 가질 수 있다.
CPU register 상태는 없다.
실행 중인 stack은 없다.
PID는 없다.
scheduler가 관리하지 않는다.
```

이 파일을 동시에 세 번 실행하면 어떻게 될까요?

```bash
./hello &
./hello &
./hello &
```

실행 파일은 하나지만, 프로세스는 세 개가 생성될 수 있습니다.

```text
              /--> Process A, PID 4101
hello ELF file --> Process B, PID 4102
              \--> Process C, PID 4103
```

세 프로세스는 같은 실행 파일의 코드를 사용할 수 있지만 다음 상태는 독립적입니다.

```text
PID
CPU register
Program Counter
Stack
Heap
Scheduling state
Signal state
대부분의 virtual address space
```

따라서 다음 관계가 성립합니다.

```text
하나의 executable file
        ↓
여러 개의 process instance
```

이는 클래스와 객체의 관계와 일부 비슷합니다.

```text
Executable file ≈ 실행을 위한 정적 설계
Process         ≈ 실제로 생성된 실행 인스턴스
```

다만 프로세스는 단순 객체보다 훨씬 많은 커널 자원과 CPU 상태를 포함합니다.

---

## 5. 프로세스의 정확한 의미

프로세스를 흔히 다음처럼 정의합니다.

> 실행 중인 프로그램.

입문 단계에서는 유용하지만 충분하지 않은 정의입니다.

더 정확하게는 다음과 같습니다.

> 프로세스는 하나 이상의 실행 흐름과 가상 주소 공간, 파일·권한·신호 같은 자원, 그리고 운영체제가 실행을 관리하기 위한 커널 메타데이터의 집합이다.

프로세스는 크게 세 부분으로 생각할 수 있습니다.

```text
Process
├── Process image
├── Execution context
└── Kernel metadata and resource references
```

### 5.1 Process image

프로세스가 사용자 영역에서 사용하는 메모리 모습입니다.

```text
Code
Read-only data
Initialized data
BSS
Heap
Memory mappings
User stack
Shared libraries
```

### 5.2 Execution context

CPU가 이 실행 흐름을 이어서 실행하는 데 필요한 상태입니다.

```text
Program Counter
Stack Pointer
General-purpose registers
Status register
FPU/SIMD state
Thread-local state
```

### 5.3 Kernel metadata

커널이 프로세스를 관리하기 위해 보관하는 정보입니다.

```text
PID와 PPID
Scheduling state
Priority
Credentials
Open file references
Signal information
Virtual memory metadata
Accounting information
Kernel stack
Resource limits
```

따라서 다음 문장이 중요합니다.

> 프로세스는 메모리만을 뜻하지 않고, 실행 상태와 운영체제가 관리하는 자원의 집합이다.

---

## 6. 오늘 필요한 세 계층

## 6.1 사용자 영역에서 보이는 것

사용자는 shell에서 실행 파일을 실행합니다.

```bash
./hello
```

또는 C 프로그램에서 다른 프로그램을 실행할 수 있습니다.

```c
execve("./hello", argv, envp);
```

사용자에게 보이는 것은 다음과 같습니다.

```text
프로그램이 실행된다.
출력이 나타난다.
PID를 확인할 수 있다.
종료 코드가 shell로 전달된다.
```

## 6.2 Kernel 내부에서 일어나는 것

커널은 실행 요청을 처리하면서 다음 작업을 수행합니다.

```text
실행 파일을 찾는다.
실행 권한을 검사한다.
파일 형식을 검사한다.
새 virtual address space를 구성한다.
실행 파일의 segment를 메모리에 매핑한다.
stack을 만든다.
register 초기값을 준비한다.
task를 scheduler가 실행할 수 있는 상태로 만든다.
```

이 단계에서 중요한 점은 “실행 파일 전체를 즉시 RAM에 복사한다”가 아니라는 것입니다.

일반적으로 실행 파일의 필요한 영역은 virtual memory에 매핑됩니다. 실제 physical memory page는 해당 주소가 처음 접근될 때 page fault를 통해 들어올 수 있습니다.

이를 demand paging이라고 합니다.

## 6.3 CPU 또는 하드웨어에서 일어나는 것

CPU 관점에서 프로그램 실행이 가능하려면 최소한 다음 상태가 필요합니다.

```text
Program Counter:
다음에 실행할 명령어 주소

Stack Pointer:
현재 stack 위치

General-purpose registers:
연산 중인 값과 함수 인자

Page table base:
virtual address를 physical address로 변환하기 위한 기준

Privilege mode:
user mode인지 kernel mode인지

Status register:
조건 플래그와 CPU 실행 상태
```

CPU는 “프로세스”라는 고수준 개념을 직접 이해하지 않습니다.

CPU가 직접 다루는 것은 다음과 같습니다.

```text
명령어 주소
register 값
virtual address translation
interrupt와 exception
privilege level
```

프로세스라는 추상화는 운영체제가 이러한 하드웨어 상태와 자원을 묶어 관리하기 위해 만든 것입니다.

---

## 7. Executable file의 내부 구조

Linux 실행 파일은 보통 ELF입니다.

다음 명령으로 확인할 수 있습니다.

```bash
file ./hello
```

예상 출력:

```text
hello: ELF 64-bit LSB pie executable, x86-64, dynamically linked, ...
```

ELF 파일에는 두 가지 중요한 관점이 있습니다.

```text
Section
Segment
```

### 7.1 Section

주로 링커와 정적 분석 도구가 사용하는 논리적 구분입니다.

예:

```text
.text     기계어 코드
.rodata   읽기 전용 상수
.data     초기화된 전역 변수
.bss      0으로 초기화될 전역·정적 변수
.symtab   심볼 테이블
.debug_*  디버깅 정보
```

### 7.2 Segment

프로그램을 실행할 때 커널의 loader가 메모리에 매핑하는 단위입니다.

예:

```text
읽기·실행 가능한 segment
읽기 전용 segment
읽기·쓰기 가능한 segment
```

실행 시에는 section보다 program header에 정의된 segment가 더 직접적으로 중요합니다.

이를 다음처럼 생각할 수 있습니다.

```text
Section:
파일을 제작하고 분석하는 관점

Segment:
실행을 위해 메모리에 올리는 관점
```

---

## 8. 실행 파일에서 Process Image로

다음 코드를 사용하겠습니다.

```c
#include <stdio.h>
#include <stdlib.h>

int global_initialized = 42;
int global_uninitialized;

int main(void)
{
    int local = 10;
    int *heap_value = malloc(sizeof(*heap_value));

    if (heap_value == NULL) {
        perror("malloc");
        return EXIT_FAILURE;
    }

    *heap_value = 100;

    printf("global_initialized: %p\n",
           (void *)&global_initialized);
    printf("global_uninitialized: %p\n",
           (void *)&global_uninitialized);
    printf("local: %p\n", (void *)&local);
    printf("heap_value: %p\n", (void *)heap_value);
    printf("main: %p\n", (void *)&main);

    free(heap_value);
    return EXIT_SUCCESS;
}
```

개념적으로 변수들은 다음 영역에 위치합니다.

```text
main()
→ executable code 영역

global_initialized
→ initialized data 영역

global_uninitialized
→ BSS 영역

local
→ user stack

malloc()으로 얻은 객체
→ heap 또는 allocator가 관리하는 memory mapping
```

프로세스의 주소 공간을 단순화하면 다음과 같습니다.

```text
높은 주소
+-----------------------------+
| User stack                  |
| argv, environment           |
| local variables             |
+-----------------------------+
|                             |
| Memory-mapped regions       |
| Shared libraries            |
| mmap() allocations          |
|                             |
+-----------------------------+
| Heap                        |
| malloc() allocations        |
+-----------------------------+
| BSS                         |
| zero-initialized globals    |
+-----------------------------+
| Data                        |
| initialized globals         |
+-----------------------------+
| Read-only data              |
| string literals             |
+-----------------------------+
| Code / Text                 |
| machine instructions        |
+-----------------------------+
낮은 주소
```

실제 배치는 CPU 아키텍처, ABI, 운영체제, linker 설정, PIE, ASLR 등에 따라 달라집니다.

---

## 9. 단계별 실행 추적

Shell에서 다음 명령을 입력했다고 가정하겠습니다.

```bash
./hello
```

### 단계 1. Shell이 명령을 해석한다

Shell은 입력 문자열을 분석합니다.

```text
실행할 파일: ./hello
argument: argv[0] = "./hello"
환경 변수: 현재 shell의 environment
```

Shell 자체도 하나의 프로세스입니다.

### 단계 2. Shell이 자식 실행 환경을 만든다

일반적인 외부 명령 실행에서는 shell이 대략 다음 구조를 사용합니다.

```text
Shell process
   |
   | fork()
   v
Child process
   |
   | execve("./hello", ...)
   v
hello process image
```

이번 Lecture에서는 `fork()`의 세부 구현은 다루지 않습니다.

핵심은 `execve()`가 실행 파일을 현재 프로세스의 새 프로그램 이미지로 교체한다는 점입니다.

### 단계 3. `execve()` system call 요청

사용자 프로그램 또는 shell의 child는 libc wrapper를 통해 `execve()`를 요청합니다.

```text
User-space execve() wrapper
        |
        v
System call argument 준비
        |
        v
syscall instruction
        |
        v
CPU가 kernel mode 진입
```

사용자 영역에서 보이는 것:

```c
execve(path, argv, envp);
```

Kernel 내부에서 일어나는 것:

```text
path 해석
파일 접근
권한 검사
실행 형식 검사
새 process image 준비
```

CPU 또는 하드웨어에서 일어나는 것:

```text
system call instruction 실행
privilege level 변경
kernel entry point로 제어 이동
kernel stack 사용
```

### 단계 4. 커널이 실행 파일을 검사한다

커널은 다음을 확인합니다.

```text
파일이 존재하는가?
일반 파일인가?
실행 권한이 있는가?
지원되는 executable format인가?
현재 CPU 아키텍처에서 실행 가능한가?
```

Linux에서는 executable format에 따라 적절한 loader가 선택됩니다.

예:

```text
ELF binary
Script with #! interpreter
기타 지원 binary format
```

### 단계 5. 새 Virtual Address Space를 구성한다

`execve()`는 성공하면 기존 사용자 주소 공간을 새로운 프로그램 이미지로 교체합니다.

개념적으로 다음이 만들어집니다.

```text
Executable code mappings
Read-only data mappings
Writable data mappings
BSS
Heap 시작점
User stack
Shared library mappings
Dynamic linker mappings
```

여기서 “새 주소 공간”은 단순히 큰 메모리 배열 하나를 할당하는 것이 아닙니다.

커널은 virtual memory area와 page table 관련 구조를 구성합니다.

실제 physical page는 필요할 때 연결될 수 있습니다.

### 단계 6. 초기 User Stack 구성

커널은 새 프로그램이 사용할 stack에 다음 정보를 배치합니다.

```text
argc
argv 문자열과 포인터
environment 문자열과 포인터
auxiliary vector
```

단순화하면 다음과 같습니다.

```text
User stack
+--------------------------+
| argument strings         |
| environment strings      |
+--------------------------+
| auxiliary vector         |
+--------------------------+
| envp pointers            |
+--------------------------+
| argv pointers            |
+--------------------------+
| argc                     |
+--------------------------+
```

Auxiliary vector에는 프로그램과 동적 링커가 초기화 과정에서 사용할 정보가 들어갑니다.

예:

```text
page size
program header 위치
entry point 정보
user/group 관련 값
보안용 random data 위치
```

### 단계 7. Register 초기화

프로그램을 시작하려면 CPU register 초기값이 필요합니다.

대표적으로 다음이 설정됩니다.

```text
Program Counter
→ ELF entry point

Stack Pointer
→ 새 user stack의 시작 위치

General-purpose registers
→ ABI와 아키텍처에 따른 초기 상태

Status register
→ user mode로 실행 가능한 상태
```

### 단계 8. Runnable 상태가 된다

프로그램이 메모리에 구성되었다고 즉시 실행되는 것은 아닙니다.

커널 scheduler가 해당 task를 선택해야 합니다.

```text
프로그램 이미지 구성 완료
        |
        v
Runnable 상태
        |
        | scheduler 선택
        v
CPU에서 실행
```

### 단계 9. User mode로 복귀

커널은 준비된 register 상태를 사용해 user mode로 돌아갑니다.

CPU의 Program Counter는 새로운 실행 파일의 entry point를 가리킵니다.

이 순간부터 기존 shell child의 프로그램 이미지는 더 이상 실행되지 않고, 새로운 `hello` 프로그램이 실행됩니다.

### 단계 10. C Runtime이 `main()`을 호출한다

일반적인 C 프로그램에서는 entry point가 바로 `main()`은 아닙니다.

대략 다음 흐름입니다.

```text
Kernel
  |
  v
ELF entry point
  |
  v
C runtime startup code
  |
  +-- runtime 초기화
  +-- libc 초기화
  +-- constructor 처리
  +-- argc, argv 전달
  |
  v
main(argc, argv)
```

`main()`이 반환하면 런타임은 보통 반환값을 `exit()` 처리로 연결합니다.

이 부분은 Lecture 26에서 자세히 다룹니다.

---

## 10. `main()`이 시작점이라는 오해

다음 코드를 봅시다.

```c
int main(void)
{
    return 0;
}
```

프로그래머 관점에서는 프로그램이 `main()`에서 시작합니다.

그러나 실행 파일 관점에서는 실제 흐름이 대략 다음과 같습니다.

```text
_entry 또는 _start
        |
        v
C runtime startup
        |
        v
main()
        |
        v
exit 처리
```

다음 명령으로 entry point 정보를 확인할 수 있습니다.

```bash
readelf -h ./hello
```

출력 중 다음 필드를 찾습니다.

```text
Entry point address: 0x....
```

또한 심볼을 확인할 수 있습니다.

```bash
objdump -d ./hello | less
```

환경에 따라 `_start` 심볼 주변에서 startup 코드를 확인할 수 있습니다.

---

## 11. 실행 가능한 C 실습

## 실습 목표

다음을 직접 확인합니다.

```text
하나의 실행 파일을 여러 프로세스로 실행할 수 있다.
각 프로세스는 서로 다른 PID를 가진다.
각 프로세스는 독립적인 실행 상태를 가진다.
프로세스의 메모리 영역을 /proc에서 관찰할 수 있다.
```

## 준비물

Linux 또는 Linux virtual machine이 권장됩니다.

필요 도구:

```text
gcc
ps
readelf
pmap
/proc filesystem
```

## 전체 코드

파일 이름:

```text
process_identity.c
```

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int global_initialized = 42;
int global_uninitialized;

int main(void)
{
    int local_variable = 10;
    int *heap_variable = malloc(sizeof(*heap_variable));

    if (heap_variable == NULL) {
        perror("malloc");
        return EXIT_FAILURE;
    }

    *heap_variable = 100;

    printf("PID: %ld\n", (long)getpid());
    printf("PPID: %ld\n", (long)getppid());

    printf("&main                 = %p\n", (void *)&main);
    printf("&global_initialized   = %p\n",
           (void *)&global_initialized);
    printf("&global_uninitialized = %p\n",
           (void *)&global_uninitialized);
    printf("&local_variable       = %p\n",
           (void *)&local_variable);
    printf("heap_variable         = %p\n",
           (void *)heap_variable);

    printf("\n프로세스를 관찰하려면 다음 명령을 실행하세요:\n");
    printf("ps -o pid,ppid,stat,comm -p %ld\n", (long)getpid());
    printf("cat /proc/%ld/status\n", (long)getpid());
    printf("cat /proc/%ld/maps\n", (long)getpid());
    printf("pmap -x %ld\n", (long)getpid());

    printf("\nEnter를 누르면 종료합니다.\n");

    errno = 0;
    int result = getchar();

    if (result == EOF && ferror(stdin)) {
        perror("getchar");
        free(heap_variable);
        return EXIT_FAILURE;
    }

    free(heap_variable);
    return EXIT_SUCCESS;
}
```

## 컴파일 명령

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    process_identity.c -o process_identity
```

## 실행 방법

```bash
./process_identity
```

예상 출력:

```text
PID: 4132
PPID: 3890
&main                 = 0x55c...
&global_initialized   = 0x55c...
&global_uninitialized = 0x55c...
&local_variable       = 0x7ff...
heap_variable         = 0x55c...

프로세스를 관찰하려면 다음 명령을 실행하세요:
ps -o pid,ppid,stat,comm -p 4132
cat /proc/4132/status
cat /proc/4132/maps
pmap -x 4132
```

주소는 시스템과 실행마다 달라질 수 있습니다.

---

## 12. Linux 도구로 관찰하기

프로그램이 Enter 입력을 기다리는 동안 다른 터미널을 엽니다.

## 12.1 `ps`

```bash
ps -o pid,ppid,stat,comm -p <PID>
```

예:

```text
    PID    PPID STAT COMMAND
   4132    3890 S+   process_identity
```

필드 의미:

| 필드        | 의미             |
| --------- | -------------- |
| `PID`     | 프로세스 식별자       |
| `PPID`    | 부모 프로세스의 PID   |
| `STAT`    | 현재 상태와 추가 플래그  |
| `COMMAND` | 명령 또는 실행 파일 이름 |

예시에서 `S`는 interruptible sleep 계열 상태를 나타냅니다.

프로그램은 `getchar()`에서 입력을 기다리고 있으므로, CPU를 계속 사용하지 않고 대기 상태에 들어갈 수 있습니다.

이는 Lecture 5와 Lecture 23에서 자세히 다룹니다.

## 12.2 `/proc/<PID>/status`

```bash
cat /proc/<PID>/status
```

주요 필드:

```text
Name:
State:
Pid:
PPid:
Threads:
VmSize:
VmRSS:
```

의미:

```text
Name
→ 프로세스 이름

State
→ 커널이 보고하는 현재 상태

Pid
→ 프로세스 ID

PPid
→ 부모 프로세스 ID

Threads
→ 해당 프로세스의 thread 수

VmSize
→ 전체 virtual memory 크기

VmRSS
→ 현재 physical memory에 resident한 크기
```

`VmSize`와 `VmRSS`가 다르다는 점이 중요합니다.

```text
Virtual address space 크기
≠
실제 RAM에 올라온 크기
```

## 12.3 `/proc/<PID>/maps`

```bash
cat /proc/<PID>/maps
```

예시 형태:

```text
55c...-55c... r--p ... /path/process_identity
55c...-55c... r-xp ... /path/process_identity
55c...-55c... rw-p ... /path/process_identity
55c...-55c... rw-p ... [heap]
7f...-7f... r-xp ... libc.so.6
7f...-7f... rw-p ... [stack]
```

권한 필드:

```text
r = readable
w = writable
x = executable
p = private mapping
s = shared mapping
```

코드 영역이 일반적으로 `r-x`이고, 데이터 영역이 `rw-`인 이유는 보안과 정확성에 관련됩니다.

코드 영역을 쓰기 가능하게 만들지 않는 것은 잘못된 코드 수정과 공격 가능성을 줄이는 데 도움이 됩니다.

## 12.4 `readelf`

```bash
readelf -h ./process_identity
```

확인할 항목:

```text
Class
Machine
Type
Entry point address
```

Program header 확인:

```bash
readelf -l ./process_identity
```

여기서 `LOAD` 항목들을 찾습니다.

이들은 loader가 virtual address space에 매핑할 실행 파일 segment를 나타냅니다.

## 12.5 `pmap`

```bash
pmap -x <PID>
```

프로세스의 memory mapping과 크기를 보기 쉽게 보여줍니다.

환경에 따라 `/proc/<PID>/maps`가 더 직접적이고 정확한 저수준 정보를 제공합니다.

---

## 13. 같은 실행 파일을 두 번 실행해 보기

터미널 두 개에서 다음을 실행합니다.

```bash
./process_identity
```

각 프로세스는 서로 다른 PID를 출력합니다.

예:

```text
첫 번째 실행: PID 4201
두 번째 실행: PID 4207
```

그러나 virtual address는 같게 보일 수도 있고 다르게 보일 수도 있습니다.

현대 Linux에서는 ASLR과 PIE 때문에 서로 다른 주소가 나타나는 경우가 흔합니다.

```text
ASLR:
Address Space Layout Randomization

PIE:
Position-Independent Executable
```

주소가 같더라도 실제 메모리를 공유한다는 의미는 아닙니다.

두 프로세스가 같은 virtual address를 사용하더라도 서로 다른 page table을 통해 다른 physical page에 매핑될 수 있습니다.

예:

```text
Process A virtual 0x7000
        |
        v
Physical page 0x123000

Process B virtual 0x7000
        |
        v
Physical page 0x9A8000
```

가상 주소는 프로세스마다 독립적으로 해석됩니다.

이 개념은 Lecture 2에서 자세히 다룹니다.

---

## 14. CPU가 Process를 직접 이해하는가

CPU는 보통 다음과 같은 명령을 이해합니다.

```asm
mov
add
sub
call
ret
load
store
syscall
```

그러나 CPU instruction set에 일반적으로 다음과 같은 명령은 없습니다.

```text
run_process
create_process
schedule_process
```

프로세스는 커널이 만드는 추상화입니다.

커널은 각 실행 단위에 대해 register 상태와 자원 정보를 저장하고, scheduler를 사용해 어느 실행 단위를 CPU에서 실행할지 선택합니다.

CPU가 실제로 수행하는 것은 다음과 같습니다.

```text
현재 Program Counter가 가리키는 명령어 fetch
명령어 decode
register와 memory를 사용해 실행
다음 Program Counter 결정
```

운영체제는 context switch를 통해 현재 CPU 상태를 다른 실행 단위의 상태로 바꿉니다.

그 결과 사용자에게 여러 프로세스가 동시에 실행되는 것처럼 보입니다.

---

## 15. Program Counter가 중요한 이유

다음 코드를 생각해 봅시다.

```c
int main(void)
{
    int a = 10;
    int b = 20;
    int c = a + b;

    return c;
}
```

기계어 수준에서는 여러 명령어로 변환됩니다.

개념적 예시:

```asm
mov  $10, ...
mov  $20, ...
add  ...
mov  ...
ret
```

CPU는 Program Counter를 사용해 다음 실행할 명령어를 결정합니다.

```text
PC = instruction 1
PC = instruction 2
PC = instruction 3
...
```

프로세스가 CPU를 빼앗겼다가 다시 실행되려면, 커널은 적어도 “어디까지 실행했는가”를 알아야 합니다.

따라서 Program Counter는 execution context의 핵심입니다.

단, 실제 저장 단위는 Linux에서 일반적으로 프로세스보다 thread에 더 직접적으로 연결됩니다.

같은 프로세스 안에 여러 thread가 있다면 각 thread는 서로 다른 Program Counter를 가질 수 있기 때문입니다.

---

## 16. Stack Pointer가 필요한 이유

함수를 호출하면 일반적으로 다음 정보가 필요합니다.

```text
return address
local variables
saved registers
function arguments 일부
stack frame 정보
```

이 정보는 stack에 배치될 수 있습니다.

CPU는 Stack Pointer를 통해 현재 stack 위치를 추적합니다.

```text
Stack Pointer
      |
      v
+---------------------+
| current stack frame |
+---------------------+
| caller frame        |
+---------------------+
| older frame         |
+---------------------+
```

프로세스 실행을 중단했다가 나중에 재개하려면 Stack Pointer도 복원되어야 합니다.

그렇지 않으면 함수 호출 관계와 지역 변수 접근이 깨집니다.

Thread를 다룰 때 각 thread가 별도의 stack을 가져야 하는 이유도 여기에서 출발합니다.

---

## 17. Process Image와 Process Context의 차이

두 개념을 혼동하기 쉽습니다.

## Process image

주로 사용자 주소 공간에 배치된 실행 프로그램의 모습입니다.

```text
Code
Data
Heap
Stack
Shared libraries
Memory mappings
```

## Execution context

CPU 실행을 이어가기 위한 상태입니다.

```text
Program Counter
Stack Pointer
General-purpose registers
Flags
FPU/SIMD state
```

## Kernel metadata

운영체제가 관리하는 정보입니다.

```text
PID
Scheduling state
File descriptor references
Credentials
Signal configuration
Memory management references
Kernel stack
```

이를 합치면 다음과 같습니다.

```text
Process
├── Memory image
├── Execution context
└── Kernel-managed resources
```

---

## 18. 실행 파일의 Code는 매번 모두 복사되는가

반드시 그렇지는 않습니다.

같은 실행 파일을 여러 프로세스가 실행할 때, 수정되지 않는 코드 page는 physical memory에서 공유될 수 있습니다.

개념적으로:

```text
Process A code virtual page ----+
                                |
                                v
                         Same physical page
                                ^
                                |
Process B code virtual page ----+
```

가능한 이유는 코드 page가 일반적으로 읽기 전용이기 때문입니다.

한 프로세스가 해당 page를 수정하지 않는다면 물리적으로 같은 page를 공유해도 격리가 깨지지 않습니다.

반면 쓰기 가능한 데이터는 프로세스별 독립성이 필요합니다.

```text
Code:
공유 가능성이 높음

Writable data:
일반적으로 프로세스별 독립

Shared memory:
명시적으로 설정하면 프로세스 간 공유 가능
```

이것은 프로세스가 실행 파일 전체를 매번 무조건 독립 복사한다는 설명이 부정확한 이유입니다.

---

## 19. 성능과 자원 비용

실행 파일을 프로세스로 실행하려면 여러 비용이 발생합니다.

### 19.1 실행 파일 검사 비용

```text
경로 탐색
권한 검사
파일 metadata 접근
ELF header 검사
```

### 19.2 Virtual memory 구성 비용

```text
Virtual memory area 생성
Page table 관련 준비
Stack mapping
Library mapping
```

### 19.3 Page fault 비용

실행 파일 page가 아직 RAM에 없다면 첫 접근 시 page fault가 발생할 수 있습니다.

```text
CPU exception
Kernel 진입
Page 확보
파일 내용 읽기 또는 page cache 연결
Page table 갱신
명령 재실행
```

### 19.4 Dynamic linking 비용

동적 연결 프로그램은 시작 과정에서 다음 비용이 추가될 수 있습니다.

```text
Dynamic linker 실행
Shared library 탐색
Symbol relocation
Lazy 또는 immediate binding
```

### 19.5 Cache와 TLB 비용

새 프로그램을 실행하면 다음 상태가 충분히 준비되지 않았을 수 있습니다.

```text
Instruction cache
Data cache
Branch predictor
TLB
```

따라서 첫 실행 구간은 이미 여러 번 실행된 hot code보다 느릴 수 있습니다.

---

## 20. 실패 사례

## 20.1 파일이 존재하지 않음

```bash
./does_not_exist
```

Shell 출력 예:

```text
No such file or directory
```

`execve()` 관점에서는 일반적으로 `ENOENT` 계열 오류가 발생할 수 있습니다.

## 20.2 실행 권한 없음

```bash
chmod -x ./process_identity
./process_identity
```

출력 예:

```text
Permission denied
```

실행 권한을 복원합니다.

```bash
chmod +x ./process_identity
```

## 20.3 CPU 아키텍처 불일치

다른 아키텍처용으로 만들어진 실행 파일을 실행하려 하면 실패할 수 있습니다.

예:

```text
ARM executable을 x86-64 시스템에서 직접 실행
```

에뮬레이터나 호환 계층이 없다면 커널은 해당 명령어를 실행할 수 없습니다.

## 20.4 잘못된 executable format

임의의 텍스트 파일에 실행 권한만 부여했다고 ELF 프로그램이 되는 것은 아닙니다.

```bash
echo "hello" > random_file
chmod +x random_file
./random_file
```

Shell의 fallback 동작 등에 따라 메시지는 달라질 수 있지만, 파일 형식이 실행 가능한 binary 또는 올바른 interpreter script가 아니면 정상 실행되지 않습니다.

## 20.5 동적 라이브러리 누락

실행 파일이 필요로 하는 shared library를 찾을 수 없다면 loader 단계에서 실행이 실패할 수 있습니다.

확인 명령:

```bash
ldd ./process_identity
```

단, 신뢰할 수 없는 실행 파일에 무작정 `ldd`를 사용하는 것은 환경에 따라 안전하지 않을 수 있습니다. 신뢰할 수 없는 binary는 별도 분석 환경에서 다루는 것이 좋습니다.

---

## 21. 동시성 관점의 첫 번째 주의사항

오늘은 thread를 본격적으로 다루지 않지만, 프로세스 개념에서 동시성의 출발점을 볼 수 있습니다.

같은 실행 파일을 여러 번 실행하면 여러 프로세스가 동시에 존재할 수 있습니다.

```text
Executable file
├── Process A
├── Process B
└── Process C
```

각 프로세스의 일반 전역 변수는 서로 독립적입니다.

따라서 한 프로세스가 자신의 전역 변수를 변경해도 다른 프로세스의 전역 변수는 자동으로 변경되지 않습니다.

그러나 다음 자원을 공유하거나 경쟁할 수 있습니다.

```text
같은 파일
같은 network port
같은 device
같은 database
명시적인 shared memory
```

예를 들어 두 프로세스가 같은 파일에 동시에 기록하면 출력 순서가 예상과 다를 수 있습니다.

프로세스의 주소 공간이 분리되어 있다고 해서 모든 race condition이 사라지는 것은 아닙니다.

```text
메모리 격리
≠
외부 자원에 대한 경쟁 없음
```

---

## 22. Linux와 Windows 비교

| 주제            | Linux/POSIX           | Windows                    |
| ------------- | --------------------- | -------------------------- |
| 대표 실행 파일 형식   | ELF                   | PE                         |
| 일반적인 shell 실행 | `fork()` 후 `execve()` | `CreateProcess()`          |
| 프로세스 식별       | PID                   | Process ID와 Process Handle |
| 파일 참조         | File descriptor       | Handle                     |
| 메모리 관찰        | `/proc`, `pmap`       | Process Explorer, WinDbg   |
| 실행 파일 분석      | `readelf`, `objdump`  | `dumpbin`, PE 분석 도구        |

Linux에서는 전통적으로 다음 모델이 중심입니다.

```text
기존 프로세스 복제
        |
        v
새 프로그램 image로 교체
```

즉:

```text
fork() + exec()
```

Windows에서는 새로운 프로세스와 초기 thread를 생성하면서 실행 파일 image를 준비하는 `CreateProcess()` 계열 모델이 중심입니다.

그러나 두 운영체제 모두 내부적으로는 다음 공통 문제를 해결해야 합니다.

```text
실행 파일 형식 해석
가상 주소 공간 구성
초기 thread context 생성
Handle 또는 descriptor 관리
권한과 보안 정보 설정
Scheduler 등록
CPU에서 실행
```

API 설계는 다르지만 하드웨어 수준의 요구사항은 유사합니다.

---

## 23. macOS에서 관찰하는 방법

macOS는 Linux의 `/proc`와 동일한 구조를 기본 제공하지 않습니다.

대신 다음 도구를 사용할 수 있습니다.

```bash
ps -o pid,ppid,state,comm -p <PID>
```

실행 파일 형식 확인:

```bash
file ./process_identity
```

macOS 실행 파일은 일반적으로 Mach-O 형식입니다.

메모리 map 관찰:

```bash
vmmap <PID>
```

디버깅:

```bash
lldb ./process_identity
```

시스템 동작 분석:

```text
Instruments
sample
Activity Monitor
```

Linux 실습을 정확히 따라가려면 Docker만으로 부족할 수 있습니다. Docker 컨테이너도 Linux 커널 기능 일부를 관찰할 수 있지만, host와 namespace를 공유하거나 제한받기 때문입니다.

프로세스와 kernel 동작을 깊게 공부할 때는 Linux virtual machine이나 실제 Linux 환경이 더 적합합니다.

---

## 24. 흔한 오해

### 오해 1. 프로그램과 프로세스는 같은 것이다

아닙니다.

```text
Program:
실행될 명령과 데이터를 표현하는 정적 개념

Executable:
실행 가능한 파일 형식으로 저장된 프로그램

Process:
실행 상태와 자원을 가진 동적 인스턴스
```

### 오해 2. 실행 파일을 열면 프로세스가 된다

단순히 `open()`으로 파일을 여는 것만으로 프로세스가 되지 않습니다.

실행을 위해서는 executable loader가 파일 형식을 해석하고 주소 공간과 CPU context를 구성해야 합니다.

### 오해 3. `main()`이 CPU가 처음 실행하는 함수다

일반적인 C 환경에서는 runtime startup code가 먼저 실행됩니다.

### 오해 4. 실행 파일 전체가 즉시 RAM에 복사된다

일반적으로 virtual memory mapping과 demand paging을 사용합니다.

필요한 page가 접근될 때 실제 RAM에 들어올 수 있습니다.

### 오해 5. 같은 가상 주소는 같은 물리 메모리를 뜻한다

아닙니다.

가상 주소는 각 주소 공간의 page table을 통해 해석됩니다.

### 오해 6. 프로세스는 메모리 덩어리다

프로세스에는 메모리 외에도 다음이 필요합니다.

```text
CPU context
PID
Scheduling state
File references
Credentials
Signals
Kernel stack
```

### 오해 7. 프로세스가 실행 중이면 항상 CPU를 사용한다

아닙니다.

프로세스 또는 그 thread는 다음 상태일 수 있습니다.

```text
Running
Runnable
Sleeping
Stopped
Zombie
```

CPU를 실제로 사용하고 있는 상태는 이 중 제한적입니다.

---

## 25. 실습 과제

## 실습 목표

하나의 executable file에서 여러 독립적인 process가 생성된다는 사실을 확인합니다.

## 과제 1. 두 프로세스 비교

두 터미널에서 각각 다음을 실행합니다.

```bash
./process_identity
```

각 프로세스에 대해 기록하세요.

```text
PID
PPID
main 주소
global variable 주소
stack variable 주소
heap 주소
```

다음 질문에 답해 보세요.

1. PID는 같은가?
2. PPID는 같은가?
3. 주소는 같은가, 다른가?
4. 주소가 같아도 실제 물리 메모리가 같다고 결론 내릴 수 있는가?
5. `/proc/<PID>/maps`에서 code, heap, stack을 찾을 수 있는가?

## 과제 2. ELF 구조 확인

```bash
readelf -h ./process_identity
readelf -l ./process_identity
readelf -S ./process_identity
```

다음을 찾으세요.

```text
Entry point
LOAD segment
.text section
.data section
.bss section
```

## 과제 3. 실행 파일과 프로세스의 수명 비교

프로그램을 종료한 후 다음을 확인합니다.

```bash
ls -l ./process_identity
ps -p <종료한_PID>
```

관찰 결과:

```text
실행 파일은 남아 있다.
프로세스는 종료되어 더 이상 일반 실행 객체로 존재하지 않는다.
```

단, 자식 종료 정보가 부모에게 회수되지 않은 상황에서는 zombie record가 잠시 남을 수 있습니다. 이는 Lecture 28에서 다룹니다.

---

## 26. 예상과 다를 수 있는 이유

실습 결과는 시스템마다 다를 수 있습니다.

### 주소가 매번 달라질 수 있음

원인:

```text
ASLR
PIE
Shared library 배치 변화
환경 변수 크기
Stack randomization
```

### `main` 주소가 매우 높은 값으로 보일 수 있음

PIE executable에서는 프로그램 코드가 고정 주소가 아니라 실행 시 선택된 base address에 매핑될 수 있습니다.

### Heap이 예상한 위치에 없을 수 있음

현대 allocator는 모든 할당을 전통적인 `brk()` 기반 heap에서만 처리하지 않습니다.

큰 할당이나 구현 정책에 따라 `mmap()`을 사용할 수 있습니다.

### 프로세스 상태가 `S`로 보일 수 있음

`getchar()`가 terminal input을 기다리면서 현재 실행 흐름이 sleep 상태로 들어갔기 때문입니다.

---

## 27. 면접에서 설명하는 방법

## 30초 설명

> 프로그램은 실행할 명령과 데이터를 표현하는 정적인 개념이고, 실행 파일은 그것을 ELF나 PE 같은 형식으로 디스크에 저장한 것입니다. 프로세스는 실행 파일이 메모리에 단순히 복사된 것만이 아니라, 가상 주소 공간, CPU register context, PID, 파일 참조, 권한, 신호, 스케줄링 정보 등을 커널이 묶어 관리하는 실행 인스턴스입니다. 하나의 실행 파일에서 여러 독립적인 프로세스가 만들어질 수 있습니다.

## 2분 설명

> 소스 코드는 컴파일과 링크를 거쳐 ELF 같은 실행 파일이 됩니다. 실행 파일은 디스크에 저장된 수동적인 데이터이므로 그 자체로는 PID나 CPU register, stack을 가지지 않습니다. Shell이 프로그램을 실행하면 Linux에서는 일반적으로 child를 만든 뒤 `execve()`를 호출합니다. 커널은 실행 파일 형식을 검사하고, code·data·stack·shared library에 해당하는 virtual memory mapping을 구성합니다. 또한 user stack에 `argc`, `argv`, 환경 변수 등을 배치하고 Program Counter를 executable의 entry point로 설정합니다. 이후 task가 runnable 상태가 되고 scheduler가 CPU를 할당하면 user mode에서 runtime startup code가 실행되고 최종적으로 `main()`이 호출됩니다. 따라서 프로세스는 단순히 메모리 이미지가 아니라 execution context와 커널이 관리하는 자원의 집합입니다.

## 심화 꼬리 질문

다음 질문에 대비해야 합니다.

```text
main() 이전에는 어떤 코드가 실행되는가?

ELF section과 segment는 어떻게 다른가?

같은 executable을 실행한 두 process가 code page를 공유할 수 있는가?

VmSize와 VmRSS의 차이는 무엇인가?

같은 virtual address가 왜 다른 physical address를 가리킬 수 있는가?

execve() 성공 시 왜 원래 함수 호출 위치로 돌아오지 않는가?

프로세스의 register context는 실제로 process 단위인가, thread 단위인가?

실행 파일 전체를 즉시 RAM에 복사하지 않는 이유는 무엇인가?
```

---

## 28. 확인 문제

정답은 아직 공개하지 않습니다.

### Level 1. 개념 확인

**문제 1.**

다음 네 개념을 각각 한 문장으로 구분하세요.

```text
Source code
Executable file
Program
Process
```

**문제 2.**

하나의 executable file에서 여러 process를 만들 수 있는 이유를 설명하세요.

### Level 2. 실행 흐름 추적

**문제 3.**

다음 흐름의 빈칸을 채우세요.

```text
Shell
  |
  v
( A )
  |
  v
execve()
  |
  v
( B )
  |
  +-- Virtual address space 구성
  +-- User stack 구성
  +-- Register 초기화
  |
  v
Runnable task
  |
  v
( C )
  |
  v
User mode에서 entry point 실행
```

**문제 4.**

커널이 executable을 성공적으로 준비했지만 scheduler가 해당 task를 아직 선택하지 않았다면, 프로그램은 CPU에서 명령어를 실행하고 있다고 볼 수 있습니까? 이유도 설명하세요.

### Level 3. C 코드와 메모리 분석

다음 코드를 봅니다.

```c
int global_value = 10;

int main(void)
{
    int local_value = 20;
    int *dynamic_value = malloc(sizeof(*dynamic_value));

    return 0;
}
```

**문제 5.**

각 항목이 일반적으로 어느 메모리 영역에 위치하는지 답하세요.

```text
main의 기계어
global_value
local_value
malloc으로 할당한 객체
```

**문제 6.**

두 프로세스에서 `local_value`의 virtual address가 우연히 같게 출력되었습니다. 두 변수가 같은 physical memory를 사용한다고 결론 내릴 수 있습니까? page table 관점에서 설명하세요.

### Level 4. Kernel·CPU 문제

**문제 7.**

실행 파일을 프로세스로 실행하기 위해 CPU 상태 측면에서 최소한 준비되어야 하는 register 또는 상태 세 가지를 선택하고, 각각 필요한 이유를 설명하세요.

**문제 8.**

다음 주장에 오류가 있는지 분석하세요.

> 프로세스는 실행 파일을 RAM에 통째로 복사한 것이다.

다음 키워드를 사용하세요.

```text
Virtual memory
Memory mapping
Demand paging
CPU context
Kernel metadata
```

---

## 29. 핵심 정리

```text
1. Source code는 사람이 작성한 프로그램 표현이다.

2. Object file은 컴파일된 기계어와 연결 정보를 담는 중간 결과다.

3. Executable file은 실행 형식에 맞게 저장된 정적인 파일이다.

4. 실행 파일 자체에는 PID, 실행 중인 stack, CPU register 상태가 없다.

5. Process는 실행 파일의 단순 복사본이 아니다.

6. Process는 virtual address space, execution context,
   kernel metadata와 자원 참조의 집합이다.

7. Linux에서 프로그램 실행은 일반적으로 shell의 fork-exec 구조와 연결된다.

8. execve() 과정에서 kernel은 executable을 검사하고,
   virtual address space와 user stack, 초기 register 상태를 구성한다.

9. CPU는 process라는 개념을 직접 실행하는 것이 아니라,
   Program Counter가 가리키는 명령어와 register 상태를 실행한다.

10. Kernel scheduler가 runnable task를 선택해야 실제 CPU 실행이 시작된다.

11. 일반적인 C 프로그램에서 main() 이전에 runtime startup code가 실행된다.

12. 같은 executable을 여러 번 실행하면 서로 다른 PID와
    실행 context를 가진 여러 process가 만들어질 수 있다.

13. 같은 virtual address가 같은 physical memory를 의미하지 않는다.

14. 실행 파일 page는 memory mapping과 demand paging을 통해
    필요할 때 physical memory에 들어올 수 있다.
```