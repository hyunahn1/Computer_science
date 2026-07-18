# Lecture 4. User Mode, Kernel Mode와 System Call

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> 일반 프로그램은 왜 파일, 네트워크, 프로세스, 하드웨어를 직접 제어하지 못하며, system call을 호출할 때 CPU와 kernel 내부에서는 정확히 어떤 일이 일어나는가?

이번 Lecture의 핵심 흐름은 다음입니다.

```text
User-space program
        |
        | libc 또는 runtime API
        v
System call argument 준비
        |
        | syscall / svc instruction
        v
CPU privilege transition
        |
        v
Kernel entry code
        |
        v
System call handler
        |
        +-- Argument 검사
        +-- 권한 검사
        +-- Kernel subsystem 호출
        +-- 필요하면 sleep
        |
        v
Return value 준비
        |
        v
User mode로 복귀
```

오늘 반드시 구분해야 할 개념은 다음과 같습니다.

```text
User space
User mode
Kernel space
Kernel mode
Library function
System call
System call wrapper
Trap
Exception
Interrupt
Kernel stack
errno
```

이 Lecture는 사용자가 제공한 Process & Thread 학습 자료의 kernel/user boundary, system call, trap, scheduler, debugger 학습 범위를 따릅니다. 

---

## 2. 이전 Lecture와의 연결

Lecture 3에서는 커널이 프로세스를 다음 정보와 함께 관리한다고 설명했습니다.

```text
Kernel task metadata
├── PID와 부모 관계
├── Scheduling state
├── Address-space reference
├── File descriptor table
├── Credentials
├── Signal state
└── Kernel stack
```

그렇다면 사용자 프로그램이 이 정보나 커널 자원을 사용하려면 어떻게 해야 할까요?

사용자 프로그램이 kernel metadata를 직접 수정할 수 있다면 다음 코드가 가능해야 합니다.

```c
current_process->pid = 1;
current_process->uid = 0;
current_process->page_table = another_process->page_table;
```

그러나 일반 프로그램에는 이러한 접근이 허용되지 않습니다.

대신 운영체제는 제한된 진입점인 **system call interface**를 제공합니다.

```text
사용자 프로그램
    |
    | 허용된 요청
    v
System call interface
    |
    v
Kernel
```

---

# 3. 전체 흐름 먼저 보기

다음 코드를 출발점으로 사용하겠습니다.

```c
ssize_t result = write(STDOUT_FILENO, "Hello\n", 6);
```

겉으로는 단순한 함수 호출이지만 내부 흐름은 다음과 같습니다.

```text
C program
   |
   v
write() 호출
   |
   v
libc system-call wrapper
   |
   +-- System call 번호 준비
   +-- File descriptor 준비
   +-- Buffer 주소 준비
   +-- Length 준비
   |
   v
CPU의 syscall instruction
   |
   +-- User mode → Kernel mode
   +-- Kernel entry address로 이동
   +-- 필요한 CPU state 저장
   +-- Kernel stack 사용
   |
   v
Kernel system-call dispatcher
   |
   v
write 구현
   |
   +-- fd 유효성 검사
   +-- User pointer 검사
   +-- File object 조회
   +-- VFS 호출
   +-- Terminal/file/device 처리
   |
   v
Return value 설정
   |
   v
Kernel mode → User mode
   |
   v
write() wrapper가 결과 반환
   |
   +-- 성공: 작성한 byte 수
   |
   +-- 실패: -1 반환, errno 설정
```

이 전체 과정에서 중요한 사실은 다음과 같습니다.

> `write()`를 호출했다고 별도의 커널 프로세스가 생성되는 것이 아니다.

현재 실행 중인 thread가 잠시 kernel mode로 진입하여 자신의 system call을 처리한 뒤 다시 user mode로 돌아옵니다.

---

# 4. User Mode와 Kernel Mode가 왜 필요한가

## 4.1 모든 코드가 같은 권한을 가진 세계

모든 프로그램이 kernel과 같은 권한을 가진다고 가정해 보겠습니다.

그러면 일반 프로그램이 다음 작업을 할 수 있습니다.

```text
다른 process의 memory 읽기

Page table 수정

Interrupt 비활성화

Disk controller 직접 조작

Network packet 임의 가로채기

다른 process의 credentials 변경

Kernel code 덮어쓰기

Scheduler data 변경
```

프로그램 하나의 작은 버그가 시스템 전체를 파괴할 수 있습니다.

```text
잘못된 pointer
    |
    v
Kernel memory 손상
    |
    v
File system corruption
    |
    v
전체 system crash
```

따라서 운영체제는 다음 불변 조건을 유지해야 합니다.

> 신뢰되지 않는 사용자 프로그램은 커널 메모리와 다른 프로세스의 자원을 직접 변경할 수 없어야 한다.

---

## 4.2 CPU가 권한을 강제한다

User mode와 kernel mode는 단순한 프로그래밍 규칙이 아닙니다.

CPU 하드웨어가 권한을 검사합니다.

### x86-64

흔히 privilege ring으로 설명합니다.

```text
Ring 0
→ Kernel mode

Ring 3
→ User mode
```

### ARM

Exception Level을 사용합니다.

```text
EL0
→ User application

EL1
→ Operating system kernel
```

CPU는 현재 권한 수준을 바탕으로 다음을 검사합니다.

```text
특권 instruction 실행 가능 여부

Page table entry의 user 접근 권한

Interrupt 관련 register 접근

Page table base register 변경

Device memory 접근

Kernel address 접근
```

---

# 5. User Space와 User Mode는 같은가

밀접하지만 같은 개념은 아닙니다.

## User space

주로 일반 프로그램과 library가 사용하는 메모리 및 실행 환경을 의미합니다.

```text
User space
├── Application code
├── libc
├── Heap
├── User stack
└── Shared libraries
```

## User mode

CPU가 제한된 privilege level로 명령을 실행하는 상태입니다.

```text
User mode
→ 현재 CPU 권한 상태
```

따라서 다음과 같이 구분합니다.

```text
User space
→ 코드와 메모리가 어디에 속하는가

User mode
→ CPU가 현재 어떤 권한으로 실행하는가
```

일반적으로 user-space application code는 user mode에서 실행됩니다.

---

# 6. Kernel Space와 Kernel Mode

## Kernel space

Kernel code와 kernel data가 존재하는 보호 영역입니다.

```text
Kernel space
├── Scheduler
├── Virtual memory manager
├── VFS
├── Network stack
├── Device drivers
├── Process metadata
└── Kernel stacks
```

## Kernel mode

CPU가 높은 privilege level에서 실행되는 상태입니다.

Kernel mode에서는 다음 작업이 가능합니다.

```text
Page table 변경

Device control register 접근

Interrupt 설정

Process scheduling

Protected memory 접근

System-wide resource 관리
```

하지만 kernel mode가 “무조건 안전한 상태”라는 뜻은 아닙니다.

Kernel code에 버그가 있으면 시스템 전체가 영향을 받을 수 있습니다.

```text
User-space bug
→ 보통 해당 process에 국한

Kernel-space bug
→ 전체 system crash 가능
```

---

# 7. Library Function과 System Call

이 두 개념을 반드시 구분해야 합니다.

## 7.1 Library function

User space에서 실행되는 함수입니다.

예:

```c
printf();
malloc();
strlen();
memcpy();
qsort();
pthread_mutex_lock();
```

Library function이라고 해서 항상 kernel에 들어가는 것은 아닙니다.

예를 들어:

```c
size_t length = strlen("hello");
```

`strlen()`은 문자열의 byte를 읽어 길이를 계산할 뿐입니다.

```text
User mode에서 실행
Kernel 진입 없음
```

---

## 7.2 System call

Kernel이 제공하는 서비스를 요청하는 공식 interface입니다.

예:

```c
read();
write();
open();
close();
mmap();
fork();
execve();
waitpid();
```

하지만 C 코드에서 호출하는 이름이 곧 kernel 내부 함수라는 뜻은 아닙니다.

일반적으로는 다음 구조입니다.

```text
User code
   |
   v
libc wrapper
   |
   v
Kernel system call
```

---

## 7.3 `printf()` 사례

```c
printf("value = %d\n", value);
```

`printf()`는 다음 작업을 user space에서 수행할 수 있습니다.

```text
Format string 분석

Integer를 문자열로 변환

출력 buffer 관리

문자열 결합
```

그 뒤 필요할 때 `write()`를 호출합니다.

```text
printf()
   |
   +-- User-space formatting
   +-- Buffering
   |
   v
write()
   |
   v
Kernel
```

따라서:

```text
printf
→ Library function

write
→ System call interface
```

---

## 7.4 `malloc()` 사례

```c
void *memory = malloc(100);
```

`malloc()`도 library function입니다.

```text
malloc()
  |
  +-- 기존 allocator arena 탐색
  +-- Free block 분할
  +-- Metadata 갱신
  |
  +-- 부족할 경우
         |
         v
     mmap() 또는 brk() 계열 요청
```

매번 kernel에 정확히 100 byte를 요청하는 것이 아닙니다.

---

# 8. System Call이 필요한 이유

다음 자원은 kernel이 중앙에서 관리합니다.

```text
File

Directory

Socket

Process

Thread scheduling

Virtual memory mapping

Device

Clock

Signal

Permission

Network interface
```

여러 프로그램이 같은 disk와 network device를 사용하기 때문에 kernel이 중재해야 합니다.

예를 들어 사용자 프로그램이 disk sector를 직접 쓰도록 허용하면:

```text
Program A
→ File A 기록 의도

Program B
→ 같은 sector 덮어쓰기

결과
→ File system 손상
```

System call은 단순한 함수 호출 규칙이 아니라 다음 정책 경계입니다.

```text
요청이 유효한가?

사용자에게 권한이 있는가?

Pointer가 올바른가?

Resource가 존재하는가?

현재 요청을 즉시 처리할 수 있는가?

다른 process와 충돌하지 않는가?
```

---

# 9. System Call ABI

Kernel과 user program은 argument를 어떤 방식으로 전달할지 약속해야 합니다.

이를 system call ABI라고 볼 수 있습니다.

다음 정보가 필요합니다.

```text
어떤 system call인지 나타내는 번호

각 argument가 들어갈 register

반환값이 들어올 register

오류를 어떻게 표현하는지

Kernel 진입 instruction
```

---

## 9.1 일반 함수 호출 ABI와 다른 점

일반 C 함수 호출:

```c
result = add(10, 20);
```

```text
Caller
  |
  v
같은 privilege level의 함수로 jump/call
  |
  v
함수 실행
  |
  v
return
```

System call:

```c
result = write(fd, buffer, length);
```

```text
Caller
  |
  v
System call 번호와 argument 준비
  |
  v
특수 instruction 실행
  |
  v
Privilege level 변경
  |
  v
Kernel entry
```

---

## 9.2 x86-64 Linux의 개념적 예

구체적인 ABI는 architecture에 따라 다릅니다.

x86-64 Linux에서는 system call 번호와 argument가 정해진 register에 배치됩니다.

개념적으로 `write(fd, buffer, count)`는 다음과 비슷합니다.

```asm
mov syscall_number_for_write, %rax
mov fd,                       %rdi
mov buffer_address,           %rsi
mov count,                    %rdx
syscall
```

반환값은 일반적으로 `%rax`에서 확인됩니다.

다만 실제 C 프로그램에서는 직접 assembly를 작성하기보다 libc wrapper를 사용합니다.

```c
write(fd, buffer, count);
```

이유는 다음과 같습니다.

```text
Architecture별 차이 처리

errno 변환

ABI 세부 구현 은닉

호환성 제공

취소 지점 등 library 정책 처리 가능
```

---

## 9.3 ARM64의 개념

ARM64에서는 일반적으로 `svc` 계열 instruction을 통해 kernel에 진입합니다.

```text
Argument register 준비
        |
        v
System call number 준비
        |
        v
svc instruction
        |
        v
EL0 → EL1
```

핵심은 architecture가 달라도 비슷합니다.

```text
System call ID

Arguments

Privileged transition instruction

Kernel dispatcher

Return value
```

---

# 10. System Call 진입 시 CPU에서 일어나는 일

다음은 개념적으로 단순화한 흐름입니다.

```text
1. User-space thread 실행 중

2. System call 번호와 argument register 준비

3. syscall 또는 svc instruction 실행

4. CPU가 현재 user execution state 일부를 보존

5. CPU privilege level을 kernel mode로 변경

6. 미리 설정된 kernel entry address로 이동

7. Kernel이 추가 register state를 저장

8. 현재 thread의 kernel stack을 사용

9. System call dispatcher 실행
```

여기서 중요한 것은 CPU와 kernel의 역할을 구분하는 것입니다.

## CPU가 수행하는 것

```text
Privilege transition

정해진 entry point로 control transfer

일부 execution state 보존

특권 규칙 적용
```

## Kernel entry code가 수행하는 것

```text
나머지 register 저장

Kernel stack frame 구성

System call number 확인

잘못된 entry 상태 검사

Dispatcher 호출
```

정확히 어떤 register가 CPU에 의해 자동 저장되고 어떤 register가 kernel code에 의해 저장되는지는 architecture와 kernel 구현에 따라 달라집니다.

---

# 11. Kernel Stack은 왜 필요한가

Thread는 일반적으로 다음 두 stack을 구분합니다.

```text
Thread
├── User stack
└── Kernel stack
```

## User stack

다음 실행에 사용됩니다.

```text
main()
User function
libc function
Local variable
Return address
```

## Kernel stack

다음 실행에 사용됩니다.

```text
System call handler

Page-fault handler

Kernel function calls

Saved user register state

Interrupt 또는 exception 처리 상태
```

---

## 11.1 User stack을 kernel이 그대로 사용하면 안 되는 이유

User stack은 사용자 프로그램이 통제합니다.

사용자는 다음을 할 수 있습니다.

```text
Stack pointer를 이상한 주소로 설정

Stack 내용을 변경

Stack mapping 제거

Guard page 접근

고의로 잘못된 값을 삽입
```

Kernel이 이를 신뢰하면 kernel 실행이 손상될 수 있습니다.

따라서 kernel 진입 시 신뢰 가능한 kernel stack으로 전환합니다.

```text
User mode
User stack
    |
    | system call
    v
Kernel mode
Kernel stack
```

---

## 11.2 각 thread에 kernel stack이 필요한 이유

Thread A와 Thread B가 각각 system call 중일 수 있습니다.

```text
Thread A
→ read()에서 sleep

Thread B
→ write() 처리 중
```

각 thread는 자신의 kernel call chain과 저장된 register 상태를 보존해야 합니다.

```text
Thread A kernel stack
├── read handler
├── VFS read
└── sleep state

Thread B kernel stack
├── write handler
└── terminal driver
```

따라서 일반적인 kernel-level thread는 thread별 kernel stack이 필요합니다.

---

# 12. System Call Dispatcher

Kernel에 진입하면 system call 번호를 확인해야 합니다.

개념적으로:

```c
switch (syscall_number) {
case SYS_read:
    return kernel_read(...);

case SYS_write:
    return kernel_write(...);

case SYS_close:
    return kernel_close(...);

default:
    return -ENOSYS;
}
```

실제 kernel 구현은 단순한 C `switch`와 동일하지 않을 수 있지만, 논리적 역할은 비슷합니다.

```text
System call number
        |
        v
System call table 또는 dispatch logic
        |
        v
해당 kernel implementation
```

잘못되거나 지원되지 않는 system call 번호라면 `ENOSYS` 계열 오류가 반환될 수 있습니다.

---

# 13. 단계별 실행 추적: `write()`

다음 코드를 추적하겠습니다.

```c
const char message[] = "Hello\n";

ssize_t result =
    write(STDOUT_FILENO, message, sizeof(message) - 1);
```

---

## 단계 1. User program이 `write()` 호출

### 사용자 영역에서 보이는 것

```text
fd 1로 문자열을 출력한다.
```

`STDOUT_FILENO`는 일반적으로 1입니다.

```text
fd 0 → standard input
fd 1 → standard output
fd 2 → standard error
```

---

## 단계 2. libc wrapper 실행

Wrapper는 system call ABI에 맞춰 다음을 준비합니다.

```text
System call 번호

fd = 1

buffer = message의 virtual address

count = 6
```

이 시점까지는 user mode입니다.

---

## 단계 3. System call instruction 실행

CPU가 user mode에서 kernel mode로 전환합니다.

```text
User mode
   |
   | syscall
   v
Kernel mode
```

CPU와 kernel entry code는 user execution context를 보존합니다.

---

## 단계 4. Kernel stack으로 전환

현재 thread의 kernel stack을 사용합니다.

```text
Kernel stack
├── Saved user register state
├── System call entry frame
└── Kernel function frames
```

---

## 단계 5. System call 번호 확인

Kernel dispatcher가 `write` 요청임을 확인합니다.

```text
System call number
        |
        v
write handler
```

---

## 단계 6. File descriptor 검사

Kernel은 현재 process의 file descriptor table에서 fd 1을 찾습니다.

```text
Current process
    |
    v
File descriptor table
    |
    +-- 1
         |
         v
Kernel file object
```

fd 1이 닫혀 있다면 실패할 수 있습니다.

```text
EBADF
→ Bad file descriptor
```

---

## 단계 7. User pointer 검사

`message`는 user virtual address입니다.

Kernel은 이 주소를 무조건 신뢰해서는 안 됩니다.

다음을 검사해야 합니다.

```text
유효한 user-space 범위인가?

해당 범위가 읽기 가능한가?

6 byte 전체가 접근 가능한가?

중간 page가 unmapped되어 있지 않은가?
```

Kernel은 user memory를 다룰 때 architecture별 안전한 copy mechanism을 사용합니다.

개념적으로:

```text
User buffer
    |
    | 검증된 방식으로 copy/access
    v
Kernel 또는 device 처리
```

Kernel이 일반 C pointer처럼 user pointer를 무조건 역참조하면 보안 문제가 생길 수 있습니다.

---

## 단계 8. VFS 또는 하위 객체 호출

fd 1이 terminal과 연결되어 있다면 대략 다음 흐름이 가능합니다.

```text
write system call
        |
        v
VFS layer
        |
        v
Terminal file operation
        |
        v
TTY subsystem
        |
        v
Display 또는 pseudo-terminal
```

일반 파일이면:

```text
write
  |
  v
VFS
  |
  v
File system
  |
  v
Page cache
  |
  v
Storage 처리
```

구체적인 경로는 file 종류와 kernel 버전에 따라 달라집니다.

---

## 단계 9. 반환값 결정

성공하면 기록된 byte 수가 반환됩니다.

```text
6
```

부분 기록도 가능하므로 항상 요청 크기 전체가 기록된다고 가정해서는 안 됩니다.

실패하면 kernel 내부에서는 음수 오류 코드 형태로 전달될 수 있습니다.

---

## 단계 10. User mode 복귀

Kernel은 저장한 user context를 바탕으로 복귀합니다.

```text
Kernel return path
        |
        +-- Register 복원
        +-- User instruction pointer 복원
        +-- User stack pointer 복원
        +-- Privilege level 복원
        |
        v
User mode
```

프로그램은 `write()` 다음 명령부터 계속 실행합니다.

---

# 14. `errno`는 무엇인가

System call이 실패했을 때 C API는 흔히 다음 형태를 사용합니다.

```text
Return value
→ -1

errno
→ 구체적인 오류 원인
```

예:

```c
ssize_t result = write(fd, buffer, size);

if (result == -1) {
    perror("write");
}
```

`perror()`는 현재 `errno`에 대응하는 설명을 출력합니다.

---

## 14.1 Kernel이 직접 `errno` 변수를 수정하는가

개념적으로는 다음 과정입니다.

```text
Kernel
→ 음수 error code 반환

libc wrapper
→ 이를 감지

libc wrapper
→ errno에 양수 error number 저장

libc wrapper
→ 사용자에게 -1 반환
```

예:

```text
Kernel result: -EBADF

libc:
errno = EBADF
return -1
```

따라서 kernel 내부 오류 표현과 사용자 C API의 표현은 구분해야 합니다.

---

## 14.2 `errno` 사용 규칙

`errno`는 함수가 실패했다고 알려준 경우에만 확인해야 합니다.

잘못된 코드:

```c
errno = 0;
write(fd, buffer, size);

if (errno != 0) {
    /* 실패라고 단정 */
}
```

올바른 기본 형태:

```c
ssize_t result = write(fd, buffer, size);

if (result == -1) {
    perror("write");
}
```

성공한 함수가 이전 `errno` 값을 반드시 0으로 초기화해 주는 것은 아닙니다.

---

## 14.3 Thread-local `errno`

멀티스레드 프로그램에서 모든 thread가 하나의 전역 `errno`를 공유하면 문제가 발생합니다.

```text
Thread A
→ errno = EBADF

Thread B
→ errno = ENOMEM
```

따라서 현대 C library에서는 `errno`가 thread-local 방식으로 제공됩니다.

각 thread가 자신의 오류 상태를 관찰할 수 있습니다.

---

# 15. System Call, Exception, Interrupt

모두 kernel 진입과 연결될 수 있지만 원인이 다릅니다.

## 15.1 System Call

현재 프로그램이 의도적으로 kernel service를 요청합니다.

```text
동기적

현재 instruction이 명시적으로 발생시킴

예: read(), write(), mmap()
```

---

## 15.2 Exception

현재 instruction 실행 중 CPU가 문제나 특별 상황을 발견합니다.

예:

```text
Page fault

Divide-by-zero

Invalid instruction

Breakpoint

Protection fault
```

특징:

```text
현재 실행 중인 instruction과 직접 관련

동기적으로 발생
```

---

## 15.3 Hardware Interrupt

외부 hardware event가 CPU에 알림을 보냅니다.

예:

```text
Timer interrupt

Network packet arrival

Disk I/O completion

Keyboard input
```

특징:

```text
현재 실행 중인 instruction의 의미와 직접 관련되지 않을 수 있음

외부 device 또는 timer에 의해 발생
```

---

## 15.4 비교

| 사건          | 원인                   | 예               |
| ----------- | -------------------- | --------------- |
| System call | 프로그램의 명시적 요청         | `write()`       |
| Exception   | 현재 instruction 실행 문제 | Page fault      |
| Interrupt   | 외부 hardware event    | Timer interrupt |

세 가지 모두 kernel entry로 이어질 수 있지만 처리 목적이 다릅니다.

---

# 16. Trap이라는 단어

`trap`은 문맥에 따라 조금 다르게 사용됩니다.

일반적으로 다음처럼 쓰일 수 있습니다.

```text
의도적으로 발생시킨 synchronous exception

System call 진입

Debugger breakpoint

Exception 처리 전체를 포괄하는 표현
```

따라서 “trap은 정확히 system call이다”라고 단정하기보다는 문서와 architecture 문맥을 봐야 합니다.

운영체제 강의에서는 흔히 다음처럼 설명합니다.

```text
System call
→ software trap을 통해 kernel 진입
```

현대 architecture에서는 실제 instruction 이름이 `syscall`, `svc` 등으로 다를 수 있습니다.

---

# 17. Blocking System Call

모든 system call이 즉시 반환하는 것은 아닙니다.

다음 코드를 생각해 봅시다.

```c
char buffer[100];

ssize_t count = read(STDIN_FILENO,
                     buffer,
                     sizeof(buffer));
```

아직 입력이 없다면 어떻게 될까요?

```text
read() 호출
    |
    v
Kernel mode 진입
    |
    v
입력 data 없음
    |
    v
현재 thread를 wait queue에 등록
    |
    v
Thread state를 sleeping으로 변경
    |
    v
Scheduler가 다른 runnable thread 선택
```

키보드 입력이 들어오면:

```text
Hardware interrupt
    |
    v
Input data 도착
    |
    v
Kernel이 wait queue에서 thread wake-up
    |
    v
Runnable 상태
    |
    v
Scheduler가 선택
    |
    v
read() 처리 재개
```

따라서 system call은 단순히 “kernel 함수 하나를 빠르게 호출하고 돌아오는 것”만은 아닙니다.

System call 중 현재 thread가 CPU를 잃고 오랫동안 대기할 수 있습니다.

---

# 18. System Call 중 Context Switch가 항상 일어나는가

아닙니다.

다음 두 사건을 구분해야 합니다.

```text
Mode switch
Context switch
```

## Mode switch

같은 thread가 user mode에서 kernel mode로 들어갑니다.

```text
Thread A user mode
        |
        v
Thread A kernel mode
```

## Context switch

CPU가 다른 thread로 실행 대상을 바꿉니다.

```text
Thread A
   |
   v
Thread B
```

빠른 system call은 다음과 같을 수 있습니다.

```text
Thread A user mode
→ Thread A kernel mode
→ Thread A user mode
```

Context switch 없이 mode switch만 발생합니다.

Blocking system call은:

```text
Thread A user mode
→ Thread A kernel mode
→ Thread A sleep
→ Thread B 실행
```

이 경우 context switch까지 발생합니다.

따라서 다음 문장은 틀립니다.

> System call을 호출하면 항상 다른 process로 context switch한다.

---

# 19. System Call의 성능 비용

System call은 일반 함수 호출보다 대체로 더 많은 작업을 요구합니다.

```text
Privilege transition

Register state 처리

Kernel entry/exit

Argument validation

Security checks

Kernel data structure 접근

Potential scheduler interaction
```

하지만 다음처럼 단정하면 안 됩니다.

> 모든 system call은 매우 느리다.

비용은 system call 종류에 따라 크게 다릅니다.

```text
간단한 metadata 조회
→ 비교적 짧을 수 있음

Cached file read
→ 더 복잡

Disk I/O
→ 매우 긴 대기 가능

Network operation
→ 외부 환경에 크게 의존

fork()
→ 여러 kernel structure 준비

execve()
→ Address-space 교체
```

---

## 19.1 작은 `write()`를 반복하는 문제

다음 코드는 system call을 매우 많이 발생시킬 수 있습니다.

```c
for (size_t i = 0; i < length; i++) {
    if (write(fd, &buffer[i], 1) == -1) {
        perror("write");
        break;
    }
}
```

더 효율적인 방식:

```c
ssize_t result = write(fd, buffer, length);
```

물론 실제로는 부분 write와 signal interruption을 처리해야 합니다.

핵심은 다음입니다.

```text
System call 1,000번
→ 경계 통과 비용 1,000번

System call 1번
→ 경계 통과 비용 1번
```

이것이 buffering이 중요한 이유 중 하나입니다.

---

# 20. 실행 가능한 C 실습

## 실습 목표

다음을 관찰합니다.

```text
Library function과 system call의 차이

write()의 반환값

errno 처리

부분 write 가능성

strace에서 system call 확인
```

## 전체 코드

파일 이름:

```text
system_call_demo.c
```

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

static int write_all(int fd,
                     const char *buffer,
                     size_t length)
{
    size_t total_written = 0;

    while (total_written < length) {
        ssize_t result =
            write(fd,
                  buffer + total_written,
                  length - total_written);

        if (result > 0) {
            total_written += (size_t)result;
            continue;
        }

        if (result == -1 && errno == EINTR) {
            continue;
        }

        if (result == -1) {
            perror("write");
            return -1;
        }

        fprintf(stderr,
                "write returned 0 unexpectedly\n");
        return -1;
    }

    return 0;
}

int main(void)
{
    const char message[] =
        "Hello through the write system call.\n";

    printf("printf: this text may use user-space buffering.\n");

    if (write_all(STDOUT_FILENO,
                  message,
                  sizeof(message) - 1) == -1) {
        return EXIT_FAILURE;
    }

    int invalid_fd = -1;

    const char invalid_message[] =
        "This write should fail.\n";

    ssize_t result =
        write(invalid_fd,
              invalid_message,
              sizeof(invalid_message) - 1);

    if (result == -1) {
        int saved_errno = errno;

        fprintf(stderr,
                "write(fd=%d) failed: %s\n",
                invalid_fd,
                strerror(saved_errno));
    }

    return EXIT_SUCCESS;
}
```

---

## 컴파일

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    system_call_demo.c \
    -o system_call_demo
```

## 실행

```bash
./system_call_demo
```

예상 출력 형태:

```text
printf: this text may use user-space buffering.
Hello through the write system call.
write(fd=-1) failed: Bad file descriptor
```

출력 순서는 buffering과 stdout/stderr 연결 상태에 따라 예상과 다르게 보일 수 있습니다.

---

# 21. 왜 `write_all()`이 필요한가

다음 호출이 항상 `length` 전체를 기록한다고 보장할 수는 없습니다.

```c
write(fd, buffer, length);
```

가능한 결과:

```text
요청: 1000 byte
반환: 1000
→ 전체 기록

요청: 1000 byte
반환: 400
→ 400 byte만 기록

반환: -1, errno = EINTR
→ Signal로 중단

반환: -1, errno = EAGAIN
→ Non-blocking descriptor에서 즉시 처리 불가
```

따라서 안정적인 코드는 반환값을 검사하고 남은 데이터를 다시 처리해야 합니다.

---

# 22. `strace`로 System Call 관찰하기

Linux에서:

```bash
strace ./system_call_demo
```

출력에는 많은 loader 관련 system call이 포함될 수 있습니다.

특정 system call만 보려면:

```bash
strace -e trace=write ./system_call_demo
```

예상 형태:

```text
write(1,
      "Hello through the write system call.\n",
      37) = 37

write(-1,
      "This write should fail.\n",
      24) = -1 EBADF (Bad file descriptor)
```

각 필드의 의미:

```text
write
→ System call 이름

1
→ File descriptor

문자열
→ User buffer 내용의 일부 표현

37
→ 요청 byte 수

= 37
→ 성공적으로 기록한 byte 수
```

실패:

```text
= -1 EBADF
```

의미:

```text
Kernel operation 실패

libc/API 관점에서 -1 반환

errno = EBADF
```

---

## 22.1 `printf()`와 `write()` 비교

다음 명령을 실행합니다.

```bash
strace -e trace=write ./system_call_demo
```

`printf()` 한 번이 즉시 별도의 `write()` 하나가 되지 않을 수 있습니다.

이유:

```text
stdout buffering

Terminal인지 file인지

Newline 존재 여부

프로그램 종료 시 buffer flush

libc 구현
```

예를 들어 stdout을 파일로 redirect하면:

```bash
strace -e trace=write \
    ./system_call_demo > output.txt
```

buffering 동작이 달라질 수 있습니다.

---

# 23. Assembly에서 경계 보기

다음 명령으로 실행 파일을 disassemble할 수 있습니다.

```bash
objdump -d ./system_call_demo | less
```

일반적으로 user code가 직접 `syscall` instruction을 포함하지 않을 수도 있습니다.

이유는 `write()` wrapper가 dynamic libc 안에 있기 때문입니다.

```text
main
  |
  v
Procedure linkage를 통해 libc write 호출
  |
  v
libc 내부에서 syscall instruction
```

Dynamic symbol 확인:

```bash
objdump -T ./system_call_demo | grep write
```

또는:

```bash
readelf -Ws ./system_call_demo | grep write
```

---

# 24. `ltrace`와 `strace` 차이

## `ltrace`

Library function 호출을 관찰하는 데 사용될 수 있습니다.

```bash
ltrace ./system_call_demo
```

예:

```text
printf(...)
write(...)
strerror(...)
```

## `strace`

System call과 signal을 관찰합니다.

```bash
strace ./system_call_demo
```

핵심 구분:

```text
ltrace
→ User-space library call 관찰

strace
→ Kernel system call 경계 관찰
```

환경과 binary linking 방식에 따라 관찰 결과가 제한될 수 있습니다.

---

# 25. `gdb`로 System Call 관찰

컴파일 시 debug 정보를 넣습니다.

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    -g \
    system_call_demo.c \
    -o system_call_demo
```

GDB 실행:

```bash
gdb ./system_call_demo
```

GDB 안에서:

```gdb
break main
run
```

System call catchpoint를 지원하는 환경에서는:

```gdb
catch syscall write
continue
```

System call 진입 또는 반환 시점에 멈출 수 있습니다.

확인:

```gdb
info registers
backtrace
```

Architecture와 GDB 버전에 따라 지원 여부와 출력은 달라질 수 있습니다.

---

# 26. `/proc`와 System Call

`/proc/<PID>/status`를 읽는 것도 user program 입장에서는 파일 읽기처럼 보입니다.

```bash
cat /proc/self/status
```

개념적 흐름:

```text
cat
 |
 +-- openat()
 +-- read()
 +-- write()
 +-- close()
 |
 v
Kernel procfs implementation
```

다만 `/proc` 내용은 일반 disk file에서 읽는 것이 아니라 kernel이 현재 metadata를 바탕으로 생성할 수 있습니다.

관찰:

```bash
strace -e trace=openat,read,write,close \
    cat /proc/self/status
```

---

# 27. User Pointer와 TOCTOU 문제

Kernel이 user pointer를 검사한 뒤 실제 사용하기 전까지 user-space의 다른 thread가 내용을 바꿀 수 있습니다.

개념적으로:

```text
Kernel:
1. User buffer 검사
2. 잠시 다른 작업
3. User buffer 사용

User thread:
검사 이후 buffer 변경
```

따라서 kernel은 단순히 “한 번 검사했으니 안전하다”고 가정할 수 없습니다.

Kernel/user memory 복사에는 다음 문제가 있습니다.

```text
User mapping이 사라질 수 있음

Page fault 발생 가능

다른 thread가 memory 수정 가능

Address가 kernel 영역을 가리킬 수 있음

Length overflow 가능

복사 도중 signal 또는 fault 가능
```

이 때문에 kernel은 사용자 pointer를 다룰 때 전용 접근 방식을 사용합니다.

---

# 28. 실패 사례

## 28.1 잘못된 File Descriptor

```c
write(-1, "hello", 5);
```

예상:

```text
Return: -1
errno: EBADF
```

---

## 28.2 잘못된 User Pointer

```c
const char *pointer = (const char *)0x1;
write(STDOUT_FILENO, pointer, 10);
```

C 관점에서도 유효하지 않은 pointer 사용입니다.

Kernel은 해당 user range에서 data를 읽을 수 없다면 보통 `EFAULT` 계열 오류를 반환할 수 있습니다.

```text
Return: -1
errno: EFAULT
```

정확한 동작은 호출과 환경에 따라 달라질 수 있으며, 잘못된 pointer를 이용한 실험은 undefined behavior와 process crash 가능성이 있습니다.

---

## 28.3 Permission Denied

```c
int fd = open("/protected/file", O_WRONLY);
```

권한이 없으면:

```text
Return: -1
errno: EACCES 또는 EPERM
```

Kernel은 현재 process credentials와 file permission을 비교합니다.

---

## 28.4 Interrupted System Call

Blocking system call 중 signal이 전달될 수 있습니다.

```text
read()
  |
  v
Sleep
  |
  v
Signal 도착
  |
  v
EINTR 반환 가능
```

일부 system call은 signal handler 설정과 `SA_RESTART` 같은 조건에 따라 자동 재시작될 수 있습니다.

따라서 모든 `EINTR` 처리를 무조건 동일하게 작성해서는 안 됩니다.

---

## 28.5 Non-blocking I/O

Non-blocking descriptor에서 즉시 data를 처리할 수 없다면:

```text
errno = EAGAIN
또는
errno = EWOULDBLOCK
```

이것은 영구적인 오류라기보다 “현재는 처리할 수 없다”는 상태일 수 있습니다.

---

# 29. 동시성과 System Call

System call도 thread-safe하다고 단순히 묶어서 말할 수 없습니다.

예를 들어 두 thread가 같은 file descriptor에 동시에 write하면:

```text
Thread A → write(fd, "AAA", 3)
Thread B → write(fd, "BBB", 3)
```

가능한 결과는 file type, flags, write 크기, kernel 보장 범위에 따라 달라집니다.

```text
AAABBB

BBBAAA

일부 경우 예상하지 못한 interleaving
```

또한 두 thread가 같은 file offset을 공유하면 접근 순서에 따라 결과가 달라질 수 있습니다.

```text
Shared open-file description
├── Current file offset
└── Status flags
```

따라서 system call이 kernel에서 처리된다는 사실만으로 application-level race condition이 사라지지는 않습니다.

---

# 30. System Call과 Scheduler 연결

System call은 scheduler와 여러 방식으로 연결됩니다.

## 경우 1. 즉시 완료

```text
User mode
→ Kernel mode
→ 처리
→ User mode
```

현재 thread가 계속 실행될 수 있습니다.

## 경우 2. Blocking

```text
User mode
→ Kernel mode
→ Wait queue
→ Sleeping
→ 다른 thread 실행
```

## 경우 3. System call 처리 중 preemption

Preemptive kernel 설정에서는 kernel code 실행 중에도 특정 안전한 지점에서 다른 task로 전환될 수 있습니다.

구체적인 preemption 정책은 kernel configuration과 실행 구간에 따라 달라집니다.

## 경우 4. 반환 직전 scheduler 개입

System call 처리 중 더 높은 우선순위 task가 runnable해졌다면 user mode로 바로 돌아가기 전에 scheduling이 발생할 수 있습니다.

이 내용은 Lecture 5와 Lecture 20 이후에서 자세히 연결됩니다.

---

# 31. VDSO: 모든 OS 기능이 System Call인가

시간 조회처럼 자주 호출되는 기능은 매번 kernel mode로 진입하면 비용이 클 수 있습니다.

Linux는 일부 정보를 user space에서 빠르게 조회하도록 VDSO 같은 mechanism을 제공할 수 있습니다.

개념적으로:

```text
일반 system call
User → Kernel → User

VDSO 최적화 가능 경로
User-space mapped kernel-provided code/data
→ Kernel 진입 없이 결과 계산 가능
```

예를 들어 일부 clock 조회는 환경과 clock 종류에 따라 실제 system call 없이 처리될 수 있습니다.

따라서 다음도 단정하면 안 됩니다.

> POSIX API를 호출하면 항상 `syscall` instruction이 실행된다.

API 구현은 libc, architecture, kernel feature에 따라 최적화될 수 있습니다.

---

# 32. Linux와 Windows 비교

| 주제         | Linux/POSIX         | Windows                 |
| ---------- | ------------------- | ----------------------- |
| User API   | libc, POSIX API     | Win32 API               |
| Kernel 진입  | System call ABI     | Native system service   |
| 파일 열기      | `open()`            | `CreateFile()`          |
| 파일 읽기      | `read()`            | `ReadFile()`            |
| Process 생성 | `fork()`/`execve()` | `CreateProcess()`       |
| 오류 표현      | `-1`과 `errno`       | 실패 값과 `GetLastError()`  |
| 관찰 도구      | `strace`            | Process Monitor, WinDbg |

Windows의 Win32 API도 반드시 하나의 kernel system call과 일대일 대응하는 것은 아닙니다.

```text
Win32 API
   |
   +-- User-space 처리
   +-- 여러 내부 함수 호출
   +-- 필요하면 kernel service 요청
```

Linux의 `printf()`와 `write()` 관계처럼, 고수준 API와 kernel entry를 구분해야 합니다.

---

# 33. macOS에서 관찰하기

macOS에서도 user mode와 kernel mode 분리는 동일한 핵심 원리를 가집니다.

가능한 도구:

```text
lldb

Instruments

dtruss
사용 가능한 환경과 권한에서

fs_usage

sample
```

`dtruss`는 보안 설정과 권한 때문에 일반 환경에서 제한될 수 있습니다.

macOS는 Linux의 `strace`와 완전히 같은 도구 환경을 제공하지 않습니다.

---

# 34. 흔한 오해

## 오해 1. System call은 일반 함수 호출과 같다

아닙니다.

Privilege transition과 kernel entry가 포함됩니다.

---

## 오해 2. `printf()`는 system call이다

`printf()`는 library function입니다.

필요할 때 `write()`를 사용할 수 있습니다.

---

## 오해 3. `malloc()`은 호출할 때마다 kernel에 들어간다

Allocator가 이미 확보한 memory를 반환하면 kernel 진입이 없을 수 있습니다.

---

## 오해 4. System call마다 context switch가 발생한다

Mode switch만 발생하고 같은 thread로 돌아올 수 있습니다.

---

## 오해 5. Kernel에 들어가면 kernel process가 대신 실행한다

현재 thread가 kernel mode에서 자신의 요청을 처리하는 것이 기본 모델입니다.

---

## 오해 6. Page fault는 system call이다

Page fault는 memory access 중 CPU가 발생시키는 exception입니다.

---

## 오해 7. `errno`는 항상 최신 함수 결과를 나타낸다

함수가 실패를 보고한 경우에만 의미 있게 확인해야 합니다.

---

## 오해 8. System call 한 번이면 요청이 항상 완전히 처리된다

`read()`와 `write()`는 부분 처리될 수 있습니다.

---

## 오해 9. User pointer를 kernel이 그대로 신뢰한다

Kernel은 주소 범위와 접근 가능성을 안전하게 처리해야 합니다.

---

## 오해 10. Kernel mode는 무조건 빠르다

Kernel mode는 높은 권한일 뿐이며, I/O나 blocking으로 오래 걸릴 수 있습니다.

---

# 35. 실습 과제

## 실습 1. `printf()`와 `write()` 비교

### 실행

```bash
strace -e trace=write ./system_call_demo
```

### 관찰할 항목

```text
printf()가 몇 번의 write()로 나타나는가?

직접 호출한 write()는 어떻게 나타나는가?

Invalid fd는 어떤 errno를 반환하는가?

stdout을 file로 redirect하면 결과가 달라지는가?
```

실행:

```bash
strace -e trace=write \
    ./system_call_demo > output.txt
```

---

## 실습 2. System call 개수 비교

다음 두 프로그램을 비교합니다.

### `write_many.c`

```c
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    const char character = 'A';

    for (int i = 0; i < 1000; i++) {
        ssize_t result =
            write(STDOUT_FILENO, &character, 1);

        if (result == -1) {
            perror("write");
            return EXIT_FAILURE;
        }
    }

    return EXIT_SUCCESS;
}
```

### `write_once.c`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(void)
{
    char buffer[1000];
    memset(buffer, 'A', sizeof(buffer));

    ssize_t result =
        write(STDOUT_FILENO, buffer, sizeof(buffer));

    if (result == -1) {
        perror("write");
        return EXIT_FAILURE;
    }

    return EXIT_SUCCESS;
}
```

컴파일:

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    write_many.c -o write_many

gcc -std=c11 -Wall -Wextra -Wpedantic \
    write_once.c -o write_once
```

System call 개수 비교:

```bash
strace -c ./write_many > /dev/null
strace -c ./write_once > /dev/null
```

확인:

```text
write() 호출 횟수

전체 system call 수

소요 시간 경향
```

`strace` 자체가 큰 overhead를 추가하므로 절대적인 성능 benchmark로 사용해서는 안 됩니다.

---

## 실습 3. Blocking `read()` 관찰

파일 이름:

```text
blocking_read_demo.c
```

```c
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    char buffer[128];

    printf("PID: %ld\n", (long)getpid());
    printf("Input을 기다립니다: ");
    fflush(stdout);

    ssize_t result =
        read(STDIN_FILENO, buffer, sizeof(buffer));

    if (result == -1) {
        perror("read");
        return EXIT_FAILURE;
    }

    printf("read returned %ld bytes\n", (long)result);

    return EXIT_SUCCESS;
}
```

컴파일:

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    blocking_read_demo.c \
    -o blocking_read_demo
```

실행:

```bash
./blocking_read_demo
```

다른 terminal에서:

```bash
ps -o pid,ppid,stat,wchan,comm -p <PID>
```

관찰:

```text
Input 전:
sleeping 상태일 가능성

Input 후:
runnable 또는 running을 거쳐 실행 재개
```

`wchan`은 kernel에서 어디를 기다리는지 나타내는 정보가 될 수 있지만, 권한과 kernel 설정에 따라 제한될 수 있습니다.

---

# 36. 실습에서 예상과 다를 수 있는 이유

```text
libc buffering 정책

stdout이 terminal인지 file인지

Compiler optimization

Static 또는 dynamic linking

Kernel version

CPU architecture

Signal 발생

System call restart 정책

Container 또는 sandbox 제한

Security policy

strace 권한
```

특히 `printf()`와 `write()` 출력 순서는 다음 조건에 따라 달라질 수 있습니다.

```text
Line buffering

Full buffering

stderr unbuffered 여부

Redirect 여부

Program exit 시 flush
```

---

# 37. 면접에서 설명하는 방법

## 30초 설명

> User mode는 일반 프로그램이 제한된 권한으로 실행되는 CPU 상태이고, kernel mode는 운영체제가 page table, device, scheduler 같은 시스템 자원을 관리할 수 있는 높은 권한 상태입니다. 프로그램이 파일이나 네트워크 같은 kernel-managed resource를 사용하려면 libc wrapper를 통해 system call을 요청합니다. CPU는 `syscall`이나 `svc` instruction으로 kernel mode에 진입하고, kernel은 현재 thread의 kernel stack에서 argument와 권한을 검사한 뒤 결과를 반환합니다. System call은 mode switch를 일으키지만 항상 context switch를 일으키는 것은 아닙니다.

## 2분 설명

> 일반 프로그램은 user mode에서 실행되며 자신의 허용된 virtual address space와 일반 CPU instruction만 사용할 수 있습니다. 파일, process 생성, memory mapping, network처럼 시스템 전체에서 공유되는 자원은 kernel이 관리하기 때문에 system call interface를 통해 요청해야 합니다. 예를 들어 `write()`를 호출하면 libc wrapper가 system call 번호와 argument를 ABI에 맞는 register에 넣고 `syscall` instruction을 실행합니다. CPU는 privilege level을 변경하고 kernel entry point로 이동하며, kernel entry code는 register 상태를 저장하고 해당 thread의 kernel stack을 사용합니다. Dispatcher가 system call 번호에 맞는 handler를 호출하고, file descriptor와 user pointer, permission을 검사한 뒤 VFS나 driver로 요청을 전달합니다. 처리가 끝나면 return value를 설정하고 user context를 복원해 user mode로 돌아갑니다. 요청이 blocking되면 thread는 wait queue에서 sleep하고 scheduler가 다른 thread를 실행할 수 있지만, 빠르게 끝나는 system call은 context switch 없이 같은 thread로 돌아갈 수도 있습니다.

## 심화 꼬리 질문

```text
User mode와 user space는 어떻게 다른가?

System call과 일반 함수 호출의 차이는 무엇인가?

printf()와 write()는 어떻게 다른가?

System call 중 어떤 register가 저장되는가?

Kernel stack이 thread마다 필요한 이유는 무엇인가?

Mode switch와 context switch는 어떻게 다른가?

Page fault와 system call은 어떻게 다른가?

errno는 kernel이 직접 설정하는가?

Blocking system call이 scheduler와 어떻게 연결되는가?

User pointer를 kernel이 바로 역참조하면 왜 위험한가?

System call이 항상 syscall instruction을 실행하지 않을 수 있는 이유는 무엇인가?

VDSO는 어떤 문제를 줄이는가?
```

---

# 38. 확인 문제

정답은 바로 공개하지 않습니다.

## Level 1. 개념 확인

### 문제 1

다음 개념을 각각 구분하세요.

```text
User space
User mode
Kernel space
Kernel mode
```

### 문제 2

다음 중 library function과 system call interface를 구분하세요.

```text
printf()
strlen()
malloc()
write()
read()
execve()
memcpy()
mmap()
```

그리고 library function이 내부적으로 system call을 사용할 수 있다는 것이 왜 모순이 아닌지 설명하세요.

---

## Level 2. 경계 추적

### 문제 3

다음 흐름의 빈칸을 채우세요.

```text
User program
    |
    v
( A )
    |
    v
System call 번호와 argument register 준비
    |
    v
( B )
    |
    v
CPU privilege transition
    |
    v
Kernel entry code
    |
    v
( C )
    |
    v
System call implementation
```

### 문제 4

다음 두 흐름의 차이를 설명하세요.

```text
Thread A user mode
→ Thread A kernel mode
→ Thread A user mode
```

```text
Thread A user mode
→ Thread A kernel mode
→ Thread A sleeping
→ Thread B running
```

첫 번째에서는 무엇이 발생하고, 두 번째에서는 무엇이 추가로 발생합니까?

---

## Level 3. C와 오류 처리

### 문제 5

다음 코드의 오류를 찾으세요.

```c
errno = 0;

write(fd, buffer, size);

if (errno != 0) {
    perror("write");
}
```

올바른 검사 방식도 작성하세요.

### 문제 6

다음 `write()` 호출에서 가능한 반환값을 설명하세요.

```c
ssize_t result = write(fd, buffer, 1000);
```

다음을 포함하세요.

```text
1000
1~999
-1과 EINTR
-1과 EAGAIN
-1과 EBADF
```

---

## Level 4. Kernel·CPU 분석

### 문제 7

`write(1, buffer, 100)` 호출의 전체 과정을 다음 키워드를 사용해 설명하세요.

```text
libc wrapper
System call number
Argument registers
syscall instruction
Privilege transition
Kernel stack
File descriptor table
User pointer validation
VFS
Return register
User mode
```

### 문제 8

다음 주장에 오류가 있는지 분석하세요.

> `read()`가 block되면 kernel이 멈추고 CPU도 아무 일도 하지 않는다.

다음 개념을 사용하세요.

```text
Current thread
Wait queue
Sleeping state
Scheduler
Runnable thread
I/O interrupt
Wake-up
```

---

# 39. 핵심 정리

```text
1. User mode는 일반 프로그램이 제한된 권한으로 실행되는 CPU 상태다.

2. Kernel mode는 운영체제가 시스템 자원을 관리할 수 있는
   높은 privilege 상태다.

3. User space와 user mode는 관련 있지만 동일한 개념은 아니다.

4. CPU hardware가 privilege level과 memory permission을 강제한다.

5. 일반 프로그램은 kernel memory와 device를 직접 제어할 수 없다.

6. System call은 user program이 kernel service를 요청하는
   공식적인 interface다.

7. C에서 호출하는 system call 함수는 일반적으로 libc wrapper다.

8. printf(), malloc(), strlen()은 library function이며,
   항상 kernel mode로 진입하는 것은 아니다.

9. write(), read(), mmap(), fork(), execve()는 kernel service와
   연결되는 system call interface다.

10. System call 호출 시 번호와 argument가 ABI에 맞는
    register에 배치된다.

11. syscall 또는 svc instruction이 CPU privilege transition을
    발생시킨다.

12. Kernel entry code는 register 상태를 저장하고 현재 thread의
    kernel stack을 사용한다.

13. 각 kernel-level thread는 독립된 kernel stack이 필요하다.

14. Kernel은 system call number를 확인해 적절한 handler로
    dispatch한다.

15. Kernel은 file descriptor, user pointer, permission,
    credentials를 검증한다.

16. User pointer는 신뢰할 수 없으므로 kernel이 안전한 방식으로
    접근해야 한다.

17. Kernel의 오류 코드는 libc wrapper에 의해 -1과 errno 형태로
    변환될 수 있다.

18. errno는 함수가 실패를 보고한 경우에만 확인해야 한다.

19. System call은 mode switch를 발생시키지만 항상
    context switch를 일으키는 것은 아니다.

20. Blocking system call은 현재 thread를 wait queue에 넣고
    scheduler가 다른 runnable thread를 실행하게 할 수 있다.

21. System call, exception, hardware interrupt는 kernel 진입
    원인이 서로 다르다.

22. Page fault는 system call이 아니라 memory access 중 발생하는
    CPU exception이다.

23. System call은 일반 함수 호출보다 경계 처리 비용이 있지만,
    실제 비용은 요청 종류와 I/O 상태에 따라 크게 달라진다.

24. Buffering과 batch 처리는 system call 횟수를 줄여
    성능을 개선할 수 있다.

25. strace는 system call 경계를, ltrace는 user-space library
    호출을 관찰하는 데 사용할 수 있다.
```