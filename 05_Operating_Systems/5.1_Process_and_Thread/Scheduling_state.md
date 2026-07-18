# Lecture 5. Process State와 Scheduler의 기본 구조

## 1. 오늘의 핵심 질문

오늘은 다음 질문에 답합니다.

> 프로세스나 스레드는 왜 `Running`, `Ready`, `Blocked` 같은 상태를 가지며, scheduler는 이 상태들을 이용해 CPU를 어떻게 배분하는가?

핵심 흐름은 다음과 같습니다.

```text
Task 생성
   |
   v
Runnable 상태
   |
   | Scheduler가 CPU 할당
   v
Running
   |
   +-- Time slice 만료 / Preemption
   |          |
   |          v
   |       Runnable
   |
   +-- I/O, Lock, Timer 대기
   |          |
   |          v
   |       Sleeping
   |          |
   |          | Event 발생 / Wake-up
   |          v
   |       Runnable
   |
   +-- 종료
              |
              v
           Zombie 또는 정리 단계
```

오늘 반드시 구분해야 하는 개념은 다음입니다.

```text
Running
Runnable
Ready
Blocked
Sleeping
Stopped
Zombie

Scheduler
Run queue
Wait queue
Preemption
Time slice
Wake-up
Context switch
CPU-bound
I/O-bound
```

이번 강의는 제공된 학습 자료의 Process State, Scheduler, Blocking, Zombie 학습 범위를 기준으로 합니다. 

---

## 2. 이전 Lecture와의 연결

Lecture 4에서는 system call이 다음 두 가지 경로로 끝날 수 있다고 설명했습니다.

### 즉시 완료되는 경우

```text
Thread A user mode
        |
        v
Thread A kernel mode
        |
        v
Thread A user mode
```

이 경우 mode switch는 발생하지만 다른 thread로 바뀌지 않을 수 있습니다.

### Blocking되는 경우

```text
Thread A user mode
        |
        v
Thread A kernel mode
        |
        v
Thread A가 I/O 대기
        |
        v
Scheduler가 Thread B 선택
```

그렇다면 다음 질문이 생깁니다.

```text
Thread A가 기다리는 동안 상태는 무엇인가?

CPU를 기다리는 task는 어디에 저장되는가?

Scheduler는 어떤 task를 선택하는가?

대기하던 I/O가 완료되면 누가 task를 깨우는가?

CPU를 사용 중인 task를 강제로 멈출 수 있는가?
```

이것이 오늘 다룰 내용입니다.

---

# 3. 전체 흐름 먼저 보기

## 3.1 기본 상태 전이

```text
                       Scheduler dispatch
       Runnable --------------------------------> Running
          ^                                          |
          |                                          |
          | Preemption                               | Blocking operation
          | Time slice expiration                    | I/O wait
          | Higher-priority task                     | Lock wait
          |                                          | Timer sleep
          |                                          v
          +-------------------------------------- Sleeping
                           Wake-up
```

조금 더 자세히 그리면 다음과 같습니다.

```text
                     +-------------------+
                     |                   |
                     |     Runnable      |
                     |                   |
                     +---------+---------+
                               |
                               | Scheduler selects task
                               v
                     +-------------------+
                     |                   |
                     |      Running      |
                     |                   |
                     +----+----------+---+
                          |          |
       Preemption         |          | Blocking syscall
       Time slice         |          | I/O wait
       yield              |          | Mutex wait
                          |          | sleep()
                          |          v
                          |   +-------------------+
                          |   |                   |
                          +---|     Sleeping      |
                              |                   |
                              +---------+---------+
                                        |
                                        | Event / interrupt / unlock
                                        v
                                   Runnable
```

중요한 점:

> Sleeping 상태에서 바로 Running으로 가는 것이 아니라, 일반적으로 먼저 Runnable이 된 뒤 scheduler의 선택을 받아야 합니다.

---

# 4. 먼저 용어를 정확히 정리하기

운영체제 교재와 실제 Linux 용어는 완전히 일치하지 않을 수 있습니다.

## 4.1 Ready

교재에서 흔히 사용하는 용어입니다.

```text
Ready
= 실행할 준비는 되었지만 현재 CPU를 받지 못한 상태
```

## 4.2 Runnable

Linux 설명에서 자주 사용하는 표현입니다.

```text
Runnable
= 지금 실행 중이거나, CPU를 받으면 즉시 실행할 수 있는 상태
```

Linux에서는 문맥에 따라 `Running`이라는 커널 상태 값이 실제 CPU에서 실행 중인 task뿐 아니라 run queue에서 기다리는 task까지 포함할 수 있습니다.

따라서 다음을 구분해야 합니다.

```text
교재의 Running
→ 실제 CPU를 사용 중

Linux의 TASK_RUNNING
→ 실제 실행 중이거나 run queue에서 실행 가능
```

이 차이를 모르면 `ps`나 kernel 문서를 볼 때 혼란이 생깁니다.

---

# 5. Process State인가 Thread State인가

엄밀히 말하면 scheduler가 직접 선택하는 대상은 일반적으로 **실행 단위**, 즉 thread 또는 task입니다.

멀티스레드 프로세스를 생각해 보겠습니다.

```text
Process A
├── Thread A1: CPU에서 실행 중
├── Thread A2: Network I/O 대기
└── Thread A3: Mutex 대기
```

하나의 프로세스 안에서도 각 thread의 상태가 다를 수 있습니다.

```text
A1 → Running
A2 → Sleeping
A3 → Sleeping
```

따라서 다음 표현은 단순화입니다.

> 프로세스가 Running 상태다.

더 정확한 표현은 다음입니다.

> 프로세스에 속한 특정 thread 또는 task가 CPU에서 Running 상태다.

Single-threaded process에서는 process와 thread를 거의 같은 실행 단위처럼 이야기해도 큰 문제가 없지만, multithreaded process에서는 반드시 분리해야 합니다.

---

# 6. 주요 상태

## 6.1 New

새 실행 단위가 만들어지는 중인 논리적 상태입니다.

```text
Task 생성 요청
    |
    +-- ID 할당
    +-- Kernel metadata 준비
    +-- Stack 또는 context 준비
    +-- Scheduling 정보 설정
    |
    v
Runnable 등록
```

실제 Linux에서 항상 사용자에게 명확한 `New` 상태가 노출되는 것은 아닙니다.

교재상의 lifecycle 개념으로 이해하는 것이 좋습니다.

---

## 6.2 Runnable 또는 Ready

CPU만 받으면 즉시 실행할 수 있습니다.

```text
필요한 code와 register context 존재

대기 중인 I/O 없음

필요한 event 대기 없음

Scheduler 선택만 기다림
```

Runnable task는 일반적으로 run queue와 연결됩니다.

```text
Run queue
├── Task A
├── Task B
├── Task C
└── Task D
```

CPU가 하나라면 이 중 한 task만 실제로 실행할 수 있습니다.

---

## 6.3 Running

현재 CPU core에서 instruction을 실행하고 있는 상태입니다.

```text
CPU core 0
└── Task A 실행

CPU core 1
└── Task B 실행
```

멀티코어 시스템에서는 여러 task가 동시에 Running일 수 있습니다.

```text
4-core CPU
→ 최대 여러 실행 단위가 동시에 실제 실행 가능
```

다만 SMT 또는 Hyper-Threading이 있으면 논리 CPU와 물리 core의 관계를 추가로 고려해야 합니다.

---

## 6.4 Blocked 또는 Sleeping

어떤 event가 발생하기 전에는 실행을 계속할 수 없는 상태입니다.

예:

```text
Keyboard input 대기

Disk I/O 완료 대기

Network packet 대기

Timer 만료 대기

Mutex unlock 대기

Condition variable signal 대기

Child 종료 대기
```

Sleeping task는 CPU를 기다리는 것이 아닙니다.

이 점이 Runnable과 다릅니다.

```text
Runnable
→ CPU만 주면 실행 가능

Sleeping
→ CPU를 줘도 아직 진행 불가능
```

---

## 6.5 Stopped

의도적으로 실행이 중단된 상태입니다.

예:

```text
SIGSTOP 수신

Debugger가 process 정지

Job control에서 Ctrl+Z
```

Stopped task는 event가 완료되었다고 자동으로 실행되지 않습니다.

계속 실행하려면 `SIGCONT` 같은 명시적인 resume 동작이 필요할 수 있습니다.

---

## 6.6 Zombie

Zombie는 일반적인 실행 대기 상태와 성격이 다릅니다.

```text
Zombie
= 실행은 이미 종료되었지만,
  parent가 exit status를 회수하지 않은 상태
```

Zombie는:

```text
CPU를 사용하지 않는다.

정상적인 user address space는 대부분 정리되었다.

실행 가능한 register context를 유지하는 상태가 아니다.

PID와 exit status 등 최소 record만 남는다.
```

따라서 Zombie를 `Sleeping`이나 `Ready`와 같은 실행 상태로 생각하면 안 됩니다.

Zombie는 **종료 후 회수 대기 상태**입니다.

---

# 7. Scheduler란 무엇인가

Scheduler는 CPU를 사용할 task를 선택하는 kernel subsystem입니다.

```text
Runnable tasks
      |
      v
Scheduler policy
      |
      v
다음 실행할 task 선택
      |
      v
Context switch
      |
      v
Selected task runs
```

Scheduler가 해결해야 할 문제는 다음과 같습니다.

```text
공정성

응답성

처리량

우선순위

실시간 요구

CPU locality

Cache 효율

멀티코어 load balancing

전력 소비
```

하나의 기준만 최적화할 수는 없습니다.

예를 들어 throughput만 높이면 interactive response가 나빠질 수 있습니다.

---

# 8. 운영체제가 유지해야 하는 불변 조건

Scheduler와 상태 관리에서는 다음 조건이 중요합니다.

## 조건 1

```text
실행할 준비가 되지 않은 task는 CPU에서 실행하면 안 된다.
```

I/O가 끝나지 않은 task를 실행해도 진행할 수 없습니다.

## 조건 2

```text
Runnable task는 실행 기회를 받을 수 있어야 한다.
```

무한히 기다리는 starvation을 피해야 합니다.

## 조건 3

```text
하나의 논리 CPU에서는 한 순간에 하나의 task context만 실행한다.
```

## 조건 4

```text
Running task의 CPU context를 잃어버리면 안 된다.
```

전환 시 Program Counter와 Stack Pointer 등을 저장해야 합니다.

## 조건 5

```text
Sleeping task는 올바른 event가 발생했을 때만 깨워야 한다.
```

잘못된 wake-up이나 wake-up 유실은 동시성 버그로 이어질 수 있습니다.

## 조건 6

```text
동일 task가 동시에 잘못 여러 run queue에 들어가면 안 된다.
```

Scheduler 자료구조의 일관성이 깨질 수 있습니다.

---

# 9. Run Queue

Run queue는 CPU를 사용할 준비가 된 task들을 관리하는 scheduler 자료구조입니다.

단순화하면:

```text
CPU 0 Run Queue
├── Task A
├── Task B
└── Task C

CPU 1 Run Queue
├── Task D
└── Task E
```

현대 Linux의 실제 scheduler 자료구조는 정책과 버전에 따라 복잡합니다.

단순 FIFO queue 하나라고 생각하면 안 됩니다.

다음 요소를 고려할 수 있습니다.

```text
Scheduling class

Priority

Virtual runtime

Deadline

CPU affinity

Load balance

NUMA locality
```

---

## 9.1 CPU별 Run Queue

멀티코어 시스템에서는 각 논리 CPU가 자신의 run queue 관련 구조를 가질 수 있습니다.

장점:

```text
Global lock 경쟁 감소

CPU-local scheduling 가능

Cache locality 유지

빠른 task 선택
```

하지만 CPU별 runnable load가 불균형할 수 있습니다.

```text
CPU 0
├── 10 tasks

CPU 1
└── 1 task
```

그래서 kernel은 load balancing을 수행할 수 있습니다.

```text
Task migration
CPU 0 → CPU 1
```

Task migration은 부하를 고르게 하지만 cache locality를 잃을 수 있습니다.

---

# 10. Wait Queue

Sleeping task는 보통 특정 event와 연결된 wait queue에서 기다립니다.

예를 들어 pipe read를 생각해 보겠습니다.

```text
Pipe buffer 비어 있음
        |
        v
read()를 호출한 task
        |
        v
Pipe read wait queue에 등록
        |
        v
Sleeping
```

다른 task가 pipe에 데이터를 쓰면:

```text
write()
  |
  v
Pipe buffer에 data 추가
  |
  v
Read wait queue의 task wake-up
  |
  v
Runnable
```

핵심 구조:

```text
특정 event 또는 resource
          |
          v
Wait queue
├── Task A
├── Task B
└── Task C
```

Run queue와 wait queue의 역할은 다릅니다.

```text
Run queue
→ CPU를 기다리는 task

Wait queue
→ 특정 event를 기다리는 task
```

---

# 11. Blocking System Call의 전체 추적

다음 코드를 사용하겠습니다.

```c
char buffer[128];

ssize_t result =
    read(STDIN_FILENO, buffer, sizeof(buffer));
```

아직 입력이 없다고 가정합니다.

---

## 단계 1. User mode에서 `read()` 호출

### 사용자 영역에서 보이는 것

```text
Program이 terminal input을 요청한다.
```

### CPU에서 일어나는 것

```text
System call number와 argument 준비
syscall instruction 실행
User mode → Kernel mode
```

---

## 단계 2. Kernel이 fd를 확인

```text
Current task
   |
   v
File descriptor table
   |
   v
fd 0
   |
   v
Terminal 또는 pipe object
```

---

## 단계 3. Data 유무 확인

```text
입력 buffer 확인
      |
      +-- Data 있음
      |      |
      |      v
      |   즉시 반환 가능
      |
      +-- Data 없음
             |
             v
          기다려야 함
```

---

## 단계 4. Wait queue 등록

현재 task를 해당 입력 source의 wait queue에 연결합니다.

```text
Terminal input wait queue
└── Current task
```

---

## 단계 5. 상태 변경

```text
Running
   |
   v
Sleeping
```

이제 task는 CPU를 사용할 수 있는 상태가 아닙니다.

---

## 단계 6. Scheduler 호출

현재 task가 실행을 계속할 수 없으므로 scheduler가 다른 runnable task를 선택합니다.

```text
Current task sleeping
        |
        v
Run queue 조회
        |
        v
Next task 선택
        |
        v
Context switch
```

---

## 단계 7. Input 도착

사용자가 keyboard에 입력합니다.

```text
Keyboard / terminal event
        |
        v
Interrupt 또는 input subsystem 처리
        |
        v
Data buffer 갱신
```

---

## 단계 8. Wake-up

Kernel은 input을 기다리던 task를 깨웁니다.

```text
Sleeping
   |
   | wake-up
   v
Runnable
```

주의:

```text
Wake-up
≠
즉시 CPU 실행
```

Wake-up은 task를 실행 가능한 상태로 바꾸는 것입니다.

---

## 단계 9. Scheduler가 다시 선택

```text
Runnable task
      |
      | Scheduler 선택
      v
Running
```

이제 `read()` 처리가 이어집니다.

---

## 단계 10. User mode 복귀

입력을 user buffer로 전달한 뒤 system call이 반환됩니다.

```text
Kernel mode
   |
   v
User register context 복원
   |
   v
User mode
   |
   v
read() 다음 instruction 실행
```

---

# 12. Voluntary Context Switch와 Involuntary Context Switch

## 12.1 Voluntary Context Switch

현재 task가 스스로 더 이상 실행할 수 없거나 CPU를 양보하는 경우입니다.

예:

```text
Blocking read()

sleep()

Mutex wait

Condition variable wait

waitpid()

sched_yield()
```

흐름:

```text
Running task
   |
   | 스스로 block 또는 yield
   v
Scheduler
```

---

## 12.2 Involuntary Context Switch

Task가 계속 실행하고 싶지만 kernel이 강제로 CPU를 회수합니다.

예:

```text
Time slice 만료

더 높은 우선순위 task 등장

실시간 task wake-up

Scheduler preemption
```

흐름:

```text
Task A running
     |
     | Timer interrupt 또는 reschedule 요청
     v
Kernel
     |
     v
Task B 선택
```

---

# 13. Preemption

Preemption은 운영체제가 현재 실행 중인 task로부터 CPU를 회수하는 것입니다.

```text
Task A 실행 중
      |
      v
Timer interrupt
      |
      v
Kernel mode 진입
      |
      v
Scheduler 판단
      |
      +-- A 계속 실행
      |
      +-- B로 전환
```

Preemption이 없다면 CPU-bound task 하나가 자발적으로 CPU를 놓을 때까지 다른 task가 실행되지 않을 수 있습니다.

```c
for (;;) {
    /* 무한 계산 */
}
```

Preemptive multitasking에서는 이런 task도 시스템 전체 CPU를 영구 독점하지 못합니다.

---

# 14. Timer Interrupt와 Scheduler

CPU에는 timer hardware가 있습니다.

개념적인 흐름:

```text
Task A user mode 실행
        |
        v
Timer event
        |
        v
Hardware interrupt
        |
        v
Kernel interrupt handler
        |
        +-- 시간 accounting
        +-- Scheduler 관련 상태 갱신
        +-- Reschedule 필요 여부 판단
        |
        v
Task A 계속 또는 Task B로 전환
```

중요한 점:

> Timer interrupt가 발생할 때마다 반드시 context switch가 일어나는 것은 아닙니다.

Kernel은 현재 task를 계속 실행하도록 결정할 수도 있습니다.

---

# 15. Time Slice란 무엇인가

Time slice 또는 quantum은 task가 CPU를 사용할 수 있는 시간 할당 개념입니다.

단순한 round-robin scheduler를 가정하면:

```text
Task A → 5 ms
Task B → 5 ms
Task C → 5 ms
Task A → 5 ms
...
```

하지만 현대 Linux의 일반 scheduler를 단순히 고정 time slice round-robin으로만 이해하면 부정확합니다.

실제 정책은 다음을 고려합니다.

```text
Task weight

Priority

Runnable task 수

Virtual runtime

Target latency

Wake-up behavior
```

실시간 scheduling class에서는 `SCHED_FIFO`, `SCHED_RR` 등 다른 규칙이 사용됩니다.

---

# 16. CPU-bound와 I/O-bound

## 16.1 CPU-bound Task

대부분의 시간을 계산에 사용합니다.

예:

```text
Video encoding

Scientific computation

Large matrix multiplication

Compression

Busy loop
```

상태 변화:

```text
Running
  |
  | Preemption
  v
Runnable
  |
  | Scheduler 선택
  v
Running
```

Sleeping 비율이 낮습니다.

---

## 16.2 I/O-bound Task

대부분의 시간을 I/O나 event를 기다립니다.

예:

```text
Interactive shell

Web server connection wait

Keyboard input

Disk read

Network response
```

상태 변화:

```text
Running
  |
  | I/O 요청
  v
Sleeping
  |
  | I/O 완료
  v
Runnable
  |
  v
Running
```

I/O-bound task는 짧게 CPU를 사용한 뒤 다시 sleep하는 경우가 많습니다.

---

# 17. Scheduler의 목표 간 충돌

## 목표 1. Throughput

단위 시간당 많은 작업 완료.

## 목표 2. Latency

사용자 입력에 빠르게 반응.

## 목표 3. Fairness

Runnable task가 적절한 CPU 시간을 받음.

## 목표 4. Cache locality

가능하면 이전 CPU에서 다시 실행.

## 목표 5. Energy efficiency

필요 없는 CPU wake-up과 migration 감소.

이 목표들은 충돌할 수 있습니다.

예:

```text
Task migration 증가
→ Load balance 향상
→ Cache locality 악화 가능
```

```text
Long quantum
→ Context switch 감소
→ Interactive latency 악화
```

```text
Short quantum
→ Response 향상
→ Scheduler와 cache overhead 증가
```

---

# 18. Linux의 Scheduling Class

Linux는 하나의 정책만 사용하는 것이 아니라 여러 scheduling class를 지원합니다.

개념적으로:

```text
Scheduling classes
├── Deadline
├── Real-time
├── Fair scheduling
└── Idle
```

대표적인 user-facing 정책:

```text
SCHED_OTHER
→ 일반적인 time-sharing task

SCHED_BATCH
→ Batch 작업

SCHED_IDLE
→ 매우 낮은 우선순위 작업

SCHED_FIFO
→ Real-time FIFO

SCHED_RR
→ Real-time round-robin

SCHED_DEADLINE
→ Deadline 기반 실시간 scheduling
```

일반 사용자 process는 보통 `SCHED_OTHER` 계열 정책을 사용합니다.

정확한 내부 algorithm과 자료구조는 Linux 버전에 따라 발전할 수 있습니다.

---

# 19. Priority와 Nice 값

일반 Linux task에서는 `nice` 값으로 scheduling weight에 영향을 줄 수 있습니다.

확인:

```bash
ps -o pid,ni,pri,stat,comm -p <PID>
```

일반적으로 nice 범위는 다음처럼 사용됩니다.

```text
낮은 nice 값
→ 더 높은 CPU weight 경향

높은 nice 값
→ 더 낮은 CPU weight 경향
```

예:

```bash
nice -n 10 ./cpu_task
```

주의:

> Nice 값이 높다고 task가 절대로 실행되지 않는 것은 아닙니다.

또한 nice와 kernel 내부 priority 표시는 정확히 같은 값이 아닙니다.

---

# 20. CPU Affinity

CPU affinity는 task가 실행될 수 있는 CPU 집합을 제한합니다.

확인:

```bash
taskset -cp <PID>
```

특정 CPU에서 실행:

```bash
taskset -c 0 ./cpu_task
```

개념:

```text
Task A allowed CPUs
→ CPU 0, CPU 1

Task B allowed CPUs
→ CPU 2
```

장점:

```text
Cache locality 향상 가능

Benchmark 조건 통제

특정 workload 격리
```

단점:

```text
Load imbalance

사용 가능한 CPU 제한

성능 저하 가능
```

---

# 21. Linux에서 상태 관찰하기

## 21.1 `ps`

```bash
ps -o pid,ppid,stat,ni,pri,psr,wchan,comm -p <PID>
```

필드:

| 필드      | 의미                     |
| ------- | ---------------------- |
| `PID`   | Process ID             |
| `PPID`  | Parent PID             |
| `STAT`  | 상태와 추가 flag            |
| `NI`    | Nice 값                 |
| `PRI`   | 도구가 표시하는 priority      |
| `PSR`   | 최근 실행된 CPU 번호          |
| `WCHAN` | sleep 중 기다리는 kernel 위치 |
| `COMM`  | task 이름                |

---

## 21.2 대표적인 `STAT` 문자

| 문자  | 일반적 의미                      |
| --- | --------------------------- |
| `R` | Running 또는 runnable         |
| `S` | Interruptible sleep         |
| `D` | Uninterruptible sleep       |
| `T` | Stopped 또는 traced           |
| `Z` | Zombie                      |
| `I` | Idle kernel thread 등 환경별 의미 |

추가 문자가 붙을 수 있습니다.

예:

```text
S+
```

여기서 `+`는 foreground process group 관련 정보를 나타낼 수 있습니다.

---

# 22. Interruptible Sleep과 Uninterruptible Sleep

## 22.1 Interruptible Sleep

Linux에서 흔히 `S`로 표시됩니다.

```text
Event를 기다림

일부 signal에 의해 깨어날 수 있음
```

예:

```text
Terminal input 대기

Timer 대기

일부 pipe/socket 대기
```

---

## 22.2 Uninterruptible Sleep

흔히 `D`로 표시됩니다.

```text
특정 kernel operation 완료를 기다리는 상태

일반 signal로 즉시 중단되지 않을 수 있음
```

주로 특정 I/O 경로와 연관될 수 있습니다.

`D` 상태가 오래 지속되면 storage나 network filesystem 문제를 의심할 수 있지만, `D` 상태 자체가 반드시 kernel bug라는 뜻은 아닙니다.

---

# 23. Wake-up 과정

Sleeping task를 깨우는 주체는 event 종류에 따라 다릅니다.

## Timer 만료

```text
Kernel timer subsystem
→ Sleeping task를 runnable로 변경
```

## Disk I/O 완료

```text
Device interrupt
→ Driver completion 처리
→ Wait queue wake-up
```

## Network packet 도착

```text
NIC interrupt 또는 polling
→ Network stack
→ Socket wait queue wake-up
```

## Mutex unlock

```text
Unlocking thread
→ 대기 중인 thread를 wake-up
```

Wake-up의 핵심은:

```text
Wait queue에서 제거

Task state를 runnable로 변경

적절한 run queue에 enqueue

필요하면 다른 CPU에 reschedule 요청
```

---

# 24. Lost Wake-up 문제

잘못된 대기 구현에서는 wake-up을 놓칠 수 있습니다.

예:

```text
Thread A:
1. 조건 확인: false
2. 아직 wait queue에는 안 들어감

Thread B:
3. 조건을 true로 변경
4. wake-up 실행
   하지만 대기자가 없음

Thread A:
5. wait queue에 들어감
6. 영원히 sleep
```

이것이 lost wake-up입니다.

```text
Condition check
        |
        | 경쟁 구간
        v
Wait registration
```

Kernel과 동기화 library는 조건 확인과 sleep 등록 사이의 race를 방지해야 합니다.

보통 다음 구조를 사용합니다.

```text
Lock 획득

조건 확인

Wait queue 등록

Atomic하게 lock release와 sleep

Wake-up 후 조건 재확인
```

이 내용은 condition variable과 동기화 Lecture에서 더 자세히 다룹니다.

---

# 25. Spurious Wake-up

Thread가 깨었다고 해서 기다리던 조건이 반드시 참이라는 보장은 없을 수 있습니다.

따라서 condition variable은 보통 `if`가 아니라 `while`로 검사합니다.

잘못된 형태:

```c
if (!condition) {
    pthread_cond_wait(&cond, &mutex);
}
```

일반적인 올바른 패턴:

```c
while (!condition) {
    pthread_cond_wait(&cond, &mutex);
}
```

이유:

```text
Spurious wake-up 가능

다른 thread가 먼저 조건 소비

Wake-up 후 scheduling 지연

여러 waiter 동시 wake-up
```

---

# 26. 실행 가능한 실습 1: CPU-bound와 Sleeping 비교

## 실습 목표

두 task의 상태 차이를 관찰합니다.

```text
CPU-bound task
→ 주로 R

Sleeping task
→ 주로 S
```

---

## 전체 코드

파일 이름:

```text
state_demo.c
```

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>

static int run_cpu_mode(void)
{
    volatile unsigned long long counter = 0;

    printf("PID: %ld\n", (long)getpid());
    printf("CPU-bound mode입니다.\n");
    printf("다른 터미널에서 다음을 실행하세요:\n");
    printf("ps -o pid,ppid,stat,ni,pri,psr,wchan,comm "
           "-p %ld\n",
           (long)getpid());

    for (;;) {
        counter++;

        if (counter == 0) {
            /*
             * unsigned overflow 이후 compiler가 loop를
             * 완전히 제거하지 않도록 관찰 가능한 branch를 둡니다.
             */
            fputc('.', stderr);
        }
    }

    return EXIT_SUCCESS;
}

static int run_sleep_mode(void)
{
    struct timespec duration = {
        .tv_sec = 1,
        .tv_nsec = 0
    };

    printf("PID: %ld\n", (long)getpid());
    printf("Sleep mode입니다.\n");
    printf("다른 터미널에서 다음을 실행하세요:\n");
    printf("ps -o pid,ppid,stat,ni,pri,psr,wchan,comm "
           "-p %ld\n",
           (long)getpid());

    for (;;) {
        if (nanosleep(&duration, NULL) == -1) {
            if (errno == EINTR) {
                continue;
            }

            perror("nanosleep");
            return EXIT_FAILURE;
        }
    }
}

int main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stderr,
                "Usage: %s cpu|sleep\n",
                argv[0]);
        return EXIT_FAILURE;
    }

    if (strcmp(argv[1], "cpu") == 0) {
        return run_cpu_mode();
    }

    if (strcmp(argv[1], "sleep") == 0) {
        return run_sleep_mode();
    }

    fprintf(stderr,
            "Unknown mode: %s\n",
            argv[1]);

    return EXIT_FAILURE;
}
```

---

## 컴파일

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    state_demo.c \
    -o state_demo
```

---

## 실행

### CPU-bound

```bash
./state_demo cpu
```

### Sleeping

```bash
./state_demo sleep
```

다른 터미널에서:

```bash
ps -o pid,ppid,stat,ni,pri,psr,wchan,comm \
   -p <PID>
```

---

## 예상 관찰

### CPU mode

```text
STAT
→ R 또는 관찰 순간에 따라 다른 값

WCHAN
→ 보통 sleep 위치 없음
```

### Sleep mode

```text
STAT
→ S

WCHAN
→ nanosleep 또는 timer 관련 kernel wait 위치
```

`ps`는 순간 snapshot이므로 CPU-bound task도 관찰 순간 상태가 달라 보일 수 있습니다.

---

# 27. 실행 가능한 실습 2: Blocking Read 관찰

## 전체 코드

파일 이름:

```text
blocking_state_demo.c
```

```c
#define _POSIX_C_SOURCE 200809L

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(void)
{
    char buffer[128];

    printf("PID: %ld\n", (long)getpid());
    printf("입력을 기다립니다.\n");
    fflush(stdout);

    ssize_t result =
        read(STDIN_FILENO,
             buffer,
             sizeof(buffer));

    if (result == -1) {
        perror("read");
        return EXIT_FAILURE;
    }

    printf("read() returned %ld bytes\n",
           (long)result);

    return EXIT_SUCCESS;
}
```

컴파일:

```bash
gcc -std=c11 -Wall -Wextra -Wpedantic \
    blocking_state_demo.c \
    -o blocking_state_demo
```

실행:

```bash
./blocking_state_demo
```

다른 터미널에서:

```bash
ps -o pid,ppid,stat,wchan,comm -p <PID>
```

관찰:

```text
입력 전
→ S 상태일 가능성

입력 후
→ Wake-up, runnable, running을 거쳐 종료
```

---

# 28. `/proc/<PID>/status` 관찰

```bash
cat /proc/<PID>/status
```

주요 항목:

```text
State:
voluntary_ctxt_switches:
nonvoluntary_ctxt_switches:
```

예:

```text
State:  S (sleeping)
voluntary_ctxt_switches:      12
nonvoluntary_ctxt_switches:   3
```

의미:

```text
voluntary_ctxt_switches
→ Task가 block, sleep, yield 등의 이유로 CPU를 놓은 횟수와 관련

nonvoluntary_ctxt_switches
→ Preemption 등으로 강제 전환된 횟수와 관련
```

정확한 accounting 기준은 kernel 구현과 상황에 따라 세부 차이가 있을 수 있습니다.

---

# 29. `top`과 `htop`

## `top`

```bash
top
```

확인할 항목:

```text
%CPU

State

PR

NI

TIME+

Load average
```

CPU-bound task는 높은 `%CPU`를 보일 수 있습니다.

Sleeping task는 거의 0%에 가까울 수 있습니다.

---

## `htop`

`htop`이 설치되어 있다면 thread 표시와 CPU별 사용량을 시각적으로 확인하기 쉽습니다.

설정에 따라:

```text
Process tree

Thread display

CPU affinity

Priority

State
```

를 확인할 수 있습니다.

---

# 30. `time`으로 Context Switch 관찰

Linux GNU `time`을 사용합니다.

```bash
/usr/bin/time -v ./state_demo sleep
```

무한 loop이므로 종료 신호가 필요합니다.

예:

```bash
timeout 5s /usr/bin/time -v ./state_demo sleep
```

CPU mode:

```bash
timeout 5s /usr/bin/time -v ./state_demo cpu
```

확인 항목:

```text
User time

System time

Percent of CPU

Voluntary context switches

Involuntary context switches
```

일반적인 경향:

```text
CPU-bound
→ 높은 CPU 사용률
→ Involuntary switch가 나타날 수 있음

Sleep-heavy
→ 낮은 CPU 사용률
→ Voluntary switch 비중 증가 가능
```

절대적인 수치는 시스템 load와 kernel 정책에 따라 달라집니다.

---

# 31. `strace`로 Blocking 관찰

```bash
strace -e trace=read ./blocking_state_demo
```

입력 전:

```text
read(0,
```

에서 멈춘 것처럼 보일 수 있습니다.

입력하면:

```text
read(0, "hello\n", 128) = 6
```

처럼 반환됩니다.

이 사이 현재 task는 계속 CPU에서 busy waiting하는 것이 아니라 kernel에서 sleep할 수 있습니다.

---

# 32. Scheduler와 Context Switch

Scheduler가 새로운 task를 선택한 뒤에는 execution context를 바꿔야 합니다.

단순 흐름:

```text
1. 현재 task의 register state 저장

2. 현재 task의 scheduling 상태 갱신

3. 다음 runnable task 선택

4. Address space 변경 필요 여부 확인

5. 다음 task의 kernel stack과 register context로 전환

6. 다음 task 실행 재개
```

오늘은 scheduler 관점만 보고, register와 page table 전환 세부는 Lecture 19~25에서 자세히 다룹니다.

---

# 33. 같은 Process의 Thread 전환

```text
Process A
├── Thread 1
└── Thread 2
```

Thread 1에서 Thread 2로 전환하면 일반적으로 다음은 공유됩니다.

```text
Address space

Code

Heap

Global variables

File descriptor table
```

다음은 바뀝니다.

```text
Program Counter

Stack Pointer

General-purpose registers

Kernel stack

User stack

Scheduling state
```

Address space를 공유하므로 다른 process로 전환할 때와 비교해 page table 전환이 필요하지 않을 수 있습니다.

그러나 cache와 synchronization 비용 때문에 항상 매우 싸다고 단정할 수는 없습니다.

---

# 34. 다른 Process로 전환

```text
Process A Thread
        |
        v
Process B Thread
```

일반적으로 다음이 바뀔 수 있습니다.

```text
Register context

Kernel stack

User stack

Address-space reference

Page-table base 관련 상태

TLB context

Security와 process-related context
```

다른 주소 공간으로 전환하면 TLB와 cache에 추가 영향이 생길 수 있습니다.

현대 CPU는 주소 공간 식별자 등을 사용해 일부 TLB entry를 보존할 수 있으므로, 항상 TLB 전체를 완전히 비운다고 단정해서는 안 됩니다.

---

# 35. Busy Waiting과 Blocking

## Busy Waiting

```c
while (!ready) {
    /* 계속 검사 */
}
```

CPU를 계속 사용합니다.

```text
State
→ Runnable 또는 Running
```

장점:

```text
아주 짧은 대기에서는 context switch 회피 가능
```

단점:

```text
CPU 낭비

전력 소비

다른 task 실행 기회 감소

Memory contention
```

---

## Blocking

```text
조건이 충족될 때까지 wait queue에서 sleep
```

장점:

```text
CPU를 다른 task가 사용 가능

긴 대기에 효율적
```

단점:

```text
Sleep/wake-up 비용

Scheduler 개입

Context switch 가능

Wake-up latency
```

실무에서는 대기 시간이 매우 짧은지 긴지에 따라 spin과 sleep을 조합할 수 있습니다.

---

# 36. Starvation

Runnable task가 오랫동안 CPU를 받지 못하는 현상입니다.

```text
High-priority task 반복 실행
        |
        v
Low-priority task가 계속 밀림
```

Scheduler는 일반적으로 fairness를 고려하지만, 실시간 정책이나 잘못된 priority 설정에서는 starvation이 발생할 수 있습니다.

예:

```text
SCHED_FIFO 고우선순위 task가 CPU를 놓지 않음
→ 낮은 우선순위 task 실행 기회 부족
```

---

# 37. Priority Inversion 미리 보기

낮은 우선순위 task가 lock을 보유하고, 높은 우선순위 task가 그 lock을 기다리는 상황입니다.

```text
Low-priority Task L
→ Mutex 보유

High-priority Task H
→ Mutex 대기

Medium-priority Task M
→ L보다 먼저 계속 실행
```

결과:

```text
H는 높은 priority지만 L이 실행돼 lock을 풀 때까지 기다림

M이 L을 계속 밀어내면 H도 간접적으로 지연
```

이를 priority inversion이라고 합니다.

해결 기법 중 하나는 priority inheritance입니다.

본격적인 내용은 동기화 Lecture에서 다룹니다.

---

# 38. Zombie가 Run Queue에 들어가는가

아닙니다.

Zombie는 이미 실행을 종료했습니다.

```text
Zombie
→ Runnable 아님
→ CPU scheduling 대상 아님
→ User instruction 실행 불가
```

남아 있는 것은 parent가 회수할 종료 정보입니다.

```text
PID

Exit status

일부 accounting 정보

Parent-child 관계 record
```

따라서 Zombie가 많다고 CPU 사용량이 직접 증가하는 것은 아닙니다.

하지만 PID와 kernel table 자원을 소비하므로 누적되면 문제가 됩니다.

---

# 39. Linux와 Windows 비교

| 개념        | Linux                               | Windows                                |
| --------- | ----------------------------------- | -------------------------------------- |
| 실행 선택 단위  | Task/thread 중심                      | Thread 중심                              |
| 일반 상태     | Running, runnable, sleep 등          | Ready, Running, Waiting 등              |
| Run queue | CPU별 scheduler 구조                   | Processor별 dispatcher ready queue 계열   |
| 대기 구조     | Wait queue 등                        | Dispatcher objects와 wait state         |
| Priority  | Nice, scheduler policy, RT priority | Thread priority, priority class        |
| CPU 제한    | `taskset`, affinity API             | Processor affinity                     |
| 관찰        | `ps`, `/proc`, `top`, `perf`        | Task Manager, Process Explorer, WinDbg |

두 운영체제 모두 기본적으로 thread를 실제 CPU scheduling 단위로 사용합니다.

Process는 address space와 resource container 역할이 더 강합니다.

---

# 40. macOS에서 관찰하기

가능한 명령:

```bash
ps -o pid,ppid,state,pri,nice,comm -p <PID>
```

CPU 사용량:

```bash
top -pid <PID>
```

추가 도구:

```text
Activity Monitor

Instruments

sample

lldb
```

Linux의 `/proc/<PID>/status`와 동일한 interface는 기본 제공되지 않습니다.

---

# 41. 흔한 오해

## 오해 1. Ready task는 CPU에서 실행 중이다

아닙니다.

실행할 준비는 되었지만 CPU를 기다리고 있습니다.

---

## 오해 2. Sleeping task도 CPU를 기다린다

아닙니다.

특정 event가 발생하기 전에는 실행할 수 없습니다.

---

## 오해 3. Wake-up되면 즉시 실행된다

아닙니다.

먼저 runnable 상태가 되고 scheduler의 선택을 받아야 합니다.

---

## 오해 4. System call을 호출하면 항상 sleep한다

아닙니다.

즉시 완료되는 system call도 많습니다.

---

## 오해 5. Timer interrupt마다 context switch한다

아닙니다.

현재 task를 계속 실행할 수도 있습니다.

---

## 오해 6. `R`은 반드시 지금 CPU 위에서 실행 중이라는 뜻이다

Linux 도구에서 `R`은 실제 running과 runnable을 함께 나타낼 수 있습니다.

---

## 오해 7. Zombie는 CPU를 사용하는 process다

아닙니다.

이미 종료된 후 최소 record만 남은 상태입니다.

---

## 오해 8. Scheduler는 process를 선택한다

엄밀히는 실행 가능한 thread나 task를 선택합니다.

---

## 오해 9. CPU-bound task는 항상 나쁘다

아닙니다.

연산 중심 workload에서는 정상입니다. 문제는 system 전체 요구와 resource contention입니다.

---

## 오해 10. Context switch 횟수가 적을수록 항상 좋다

아닙니다.

횟수를 줄이면 throughput에 유리할 수 있지만 interactive latency와 fairness가 나빠질 수 있습니다.

---

# 42. 실습 과제

## 실습 1. 상태 변화 기록

### 목표

`blocking_state_demo`의 상태 변화를 관찰합니다.

### 실행

```bash
./blocking_state_demo
```

다른 터미널:

```bash
watch -n 0.2 \
'ps -o pid,ppid,stat,psr,wchan,comm -p <PID>'
```

### 관찰 항목

```text
입력 전 상태

WCHAN 값

입력 후 상태 변화

Process 종료 후 ps 결과
```

---

## 실습 2. CPU-bound task 두 개 실행

```bash
taskset -c 0 ./state_demo cpu &
taskset -c 0 ./state_demo cpu &
```

두 task를 같은 CPU 0에 고정합니다.

관찰:

```bash
top
```

또는:

```bash
ps -o pid,stat,ni,pri,psr,comm \
   -p <PID1>,<PID2>
```

질문:

```text
두 task가 각각 약 절반 수준의 CPU를 사용하는가?

둘 다 R로 보일 수 있는가?

실제로 한 논리 CPU에서 동시에 두 task가 실행되는가?

왜 빠르게 번갈아 실행되는 것처럼 보이는가?
```

종료:

```bash
kill <PID1> <PID2>
```

---

## 실습 3. Nice 값 비교

```bash
taskset -c 0 nice -n 0 ./state_demo cpu &
taskset -c 0 nice -n 10 ./state_demo cpu &
```

관찰:

```bash
top
```

또는:

```bash
ps -o pid,ni,pri,psr,stat,comm \
   -p <PID1>,<PID2>
```

주의:

```text
Nice 차이가 즉시 정확한 비율로 나타나는 것은 아니다.

시스템 load와 scheduler 기간에 따라 결과가 달라진다.
```

---

## 실습 4. Context Switch 비교

```bash
timeout 5s /usr/bin/time -v ./state_demo cpu

timeout 5s /usr/bin/time -v ./state_demo sleep
```

기록:

```text
User time

System time

CPU usage

Voluntary context switches

Involuntary context switches
```

두 mode의 차이를 설명하세요.

---

# 43. 면접에서 설명하는 방법

## 30초 설명

> Scheduler는 CPU를 사용할 runnable thread를 선택하는 kernel subsystem입니다. Running은 현재 CPU에서 실행 중인 상태이고, runnable 또는 ready는 실행할 준비가 되었지만 CPU를 기다리는 상태입니다. Sleeping 또는 blocked 상태는 I/O, lock, timer 같은 event가 필요해 당장 실행할 수 없는 상태입니다. Blocking system call이 발생하면 현재 thread는 wait queue에 들어가 sleep하고 scheduler가 다른 runnable thread를 선택합니다. Event가 발생하면 thread는 wake-up되어 runnable 상태가 되며, 다시 scheduler의 선택을 받아야 실행됩니다.

## 2분 설명

> 운영체제는 thread나 task마다 scheduling state를 관리합니다. Runnable task는 CPU만 받으면 즉시 실행할 수 있으므로 run queue에서 기다리고, scheduler가 이를 선택하면 Running 상태가 됩니다. 현재 task가 time slice 만료나 더 높은 우선순위 task 때문에 CPU를 빼앗기면 다시 runnable 상태가 되는데, 이를 preemption이라고 합니다. 반면 `read()`처럼 data가 없는 blocking system call을 호출하면 thread는 해당 resource의 wait queue에 등록되고 sleeping 상태가 됩니다. 이때 scheduler는 다른 runnable task로 context switch합니다. 이후 device interrupt, timer 만료, mutex unlock 같은 event가 발생하면 kernel이 sleeping task를 wake-up하여 run queue에 넣습니다. Wake-up은 즉시 실행을 의미하지 않으며 scheduler가 다시 선택해야 합니다. Linux에서 실제 scheduling 단위는 process보다는 task 또는 thread에 가깝고, `ps`의 `R`은 실행 중뿐 아니라 runnable 상태를 포함할 수 있습니다.

## 심화 꼬리 질문

```text
Ready와 Running의 차이는 무엇인가?

Linux의 TASK_RUNNING이 왜 혼란을 줄 수 있는가?

Run queue와 wait queue는 어떻게 다른가?

Blocking read의 전체 상태 전이는 무엇인가?

Wake-up된 thread가 즉시 실행되지 않을 수 있는 이유는 무엇인가?

Mode switch와 context switch는 어떻게 다른가?

Voluntary와 involuntary context switch의 차이는 무엇인가?

Timer interrupt마다 context switch가 일어나는가?

CPU-bound와 I/O-bound task의 scheduling 특성은 어떻게 다른가?

Task migration은 어떤 장단점이 있는가?

Zombie가 scheduler 대상이 아닌 이유는 무엇인가?

Lost wake-up은 어떻게 발생하는가?
```

---

# 44. 확인 문제

정답은 아직 공개하지 않습니다.

## Level 1. 개념 확인

### 문제 1

다음 상태를 각각 설명하세요.

```text
Runnable
Running
Sleeping
Stopped
Zombie
```

### 문제 2

Runnable task와 Sleeping task의 가장 중요한 차이는 무엇입니까?

---

## Level 2. 상태 전이

### 문제 3

다음 빈칸을 채우세요.

```text
Running
   |
   | Blocking read
   v
( A )
   |
   | Input 도착
   v
( B )
   |
   | Scheduler 선택
   v
Running
```

### 문제 4

현재 Running인 task가 time slice 만료로 CPU를 빼앗겼습니다.

다음 질문에 답하세요.

```text
어떤 event가 kernel 진입을 일으킬 수 있는가?

Task는 어느 상태로 이동하는가?

Wait queue에 들어가는가?

이 전환은 voluntary인가 involuntary인가?
```

---

## Level 3. Linux 관찰

### 문제 5

다음 `ps` 출력의 의미를 설명하세요.

```text
PID   STAT  PSR  WCHAN     COMMAND
5000  S     2    do_wait   demo
```

다음을 포함하세요.

```text
현재 실제 CPU에서 실행 중인가?

어떤 종류의 상태인가?

PSR은 무엇을 의미하는가?

WCHAN은 무엇을 나타낼 수 있는가?
```

### 문제 6

CPU-bound task 두 개를 하나의 논리 CPU에 고정했습니다.

```bash
taskset -c 0 ./cpu_task &
taskset -c 0 ./cpu_task &
```

두 task가 모두 `R`로 보일 수 있지만, 왜 실제로는 한 순간에 하나만 실행되는지 설명하세요.

---

## Level 4. Kernel과 동시성

### 문제 7

Blocking `read()`의 전체 흐름을 다음 키워드로 설명하세요.

```text
System call

Kernel mode

Data availability check

Wait queue

Sleeping

Scheduler

Context switch

Interrupt

Wake-up

Run queue

User mode
```

### 문제 8

다음 잘못된 대기 구현에서 lost wake-up이 발생할 수 있는 이유를 설명하세요.

```text
Thread A:
1. condition == false 확인
2. wait queue에 들어가기 전 잠시 멈춤

Thread B:
3. condition = true
4. wake-up 호출

Thread A:
5. wait queue에 들어가 sleep
```

어떤 불변 조건이 깨졌는지 설명하세요.

---

# 45. 핵심 정리

```text
1. Scheduler는 runnable thread 또는 task 중
   CPU를 사용할 대상을 선택한다.

2. Single-threaded process에서는 process state처럼 보이지만,
   실제 scheduling state는 thread 또는 task 단위로 이해해야 한다.

3. Runnable은 CPU만 받으면 즉시 실행 가능한 상태다.

4. Running은 현재 CPU에서 instruction을 실행 중인 상태다.

5. Sleeping은 I/O, lock, timer 같은 event가 필요해
   당장 실행할 수 없는 상태다.

6. Runnable task는 run queue와 연결된다.

7. Sleeping task는 특정 event의 wait queue와 연결될 수 있다.

8. Run queue는 CPU를 기다리는 task를 관리한다.

9. Wait queue는 특정 event를 기다리는 task를 관리한다.

10. Blocking system call은 current task를 sleeping 상태로 만들고
    context switch를 일으킬 수 있다.

11. Wake-up은 sleeping task를 runnable로 바꾸는 과정이다.

12. Wake-up되었다고 즉시 실행되는 것은 아니다.

13. Preemption은 kernel이 현재 task로부터 CPU를 회수하는 것이다.

14. Timer interrupt는 preemption 판단의 계기가 될 수 있지만,
    interrupt마다 context switch가 발생하는 것은 아니다.

15. Voluntary context switch는 task가 block, sleep, yield 등으로
    스스로 CPU를 놓는 경우와 관련된다.

16. Involuntary context switch는 time slice 만료나 priority로
    강제 전환되는 경우와 관련된다.

17. CPU-bound task는 주로 Running과 Runnable 사이를 오간다.

18. I/O-bound task는 Running, Sleeping, Runnable 사이를 오간다.

19. Linux의 R 상태는 실제 running과 runnable을 함께
    나타낼 수 있으므로 교재 용어와 구분해야 한다.

20. Zombie는 실행 가능한 상태가 아니라
    종료 후 parent의 회수를 기다리는 kernel record다.

21. Scheduler는 fairness, throughput, latency, cache locality,
    priority, power efficiency 사이에서 균형을 잡아야 한다.

22. CPU별 run queue는 scalability와 locality에 유리하지만,
    load balancing과 task migration이 필요할 수 있다.

23. Busy waiting은 CPU를 계속 사용하고,
    blocking은 CPU를 다른 task에 양보한다.

24. Lost wake-up은 조건 확인과 wait 등록 사이의 race로 발생한다.

25. /proc/<PID>/status, ps, top, time, strace를 이용해
    task 상태와 context switch를 관찰할 수 있다.
```
