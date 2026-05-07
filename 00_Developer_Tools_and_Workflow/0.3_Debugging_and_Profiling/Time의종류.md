# Time의 종류

## 1. 핵심 질문

이번 강의의 핵심 질문은 다음이다.

> **프로그램이 “10초 걸렸다”라고 할 때, 그 10초는 정확히 무엇을 의미하는가?**

성능 분석에서 시간은 하나가 아니다.

```text
wall time
CPU time
user time
system time
latency
throughput
cold cache time
warm cache time
```

이것들을 구분하지 못하면 다음과 같은 오류를 낸다.

```text
“10초 걸렸으니까 CPU가 10초 동안 바빴겠지.”
```

항상 그런 것은 아니다.

프로그램이 10초 걸렸지만 CPU는 0.1초만 사용했을 수도 있다. 나머지 9.9초는 파일, 네트워크, sleep, lock, 다른 프로세스 대기였을 수 있다.

---

# 2. 개념 설명

## 2.1 wall time

**wall time**은 사람이 시계를 보고 기다린 시간이다.

```text
프로그램 시작: 10:00:00
프로그램 종료: 10:00:05

wall time = 5초
```

즉, 현실 세계에서 실제로 흐른 시간이다.

다른 이름으로는 다음이 있다.

```text
elapsed time
real time
clock time
```

`time` 명령에서 보통 `real`로 보이는 값이 wall time이다.

```bash
time ./program
```

출력 예:

```text
real    0m5.000s
user    0m0.020s
sys     0m0.010s
```

여기서 `real`이 wall time이다.

---

## 2.2 CPU time

**CPU time**은 CPU가 실제로 그 프로그램을 실행하는 데 쓴 시간이다.

예를 들어 프로그램이 이런 일을 한다고 하자.

```cpp
#include <unistd.h>

int main() {
    sleep(5);
    return 0;
}
```

실행하면 사람은 5초 기다린다.

```text
wall time ≈ 5초
```

하지만 CPU는 거의 일하지 않는다. 프로그램은 대부분 자고 있다.

```text
CPU time ≈ 0초
```

따라서 결과는 대략 이렇게 나온다.

```text
real 5.00
user 0.00
sys  0.00
```

이 차이를 이해하는 것이 performance debugging의 출발점이다.

---

## 2.3 user time

**user time**은 프로그램의 사용자 공간 코드가 CPU에서 실행된 시간이다.

예를 들어 다음 코드는 대부분 user time을 만든다.

```cpp
#include <iostream>

int main() {
    volatile long long sum = 0;

    for (long long i = 0; i < 1000000000LL; ++i) {
        sum += i;
    }

    std::cout << sum << std::endl;
}
```

여기서 CPU는 대부분 내 프로그램의 loop를 실행한다.

```text
real 2.50
user 2.48
sys  0.01
```

대략 이런 식으로 나온다.

의미:

| 항목     | 의미                  |
| ------ | ------------------- |
| `real` | 사람이 기다린 전체 시간       |
| `user` | 내 코드가 CPU에서 실행된 시간  |
| `sys`  | 커널 코드가 CPU에서 실행된 시간 |

---

## 2.4 system time

**system time**은 내 프로그램 때문에 커널이 CPU를 사용한 시간이다.

예를 들어 다음 작업은 system time을 증가시킬 수 있다.

```text
read()
write()
open()
close()
stat()
send()
recv()
mmap()
fork()
exec()
```

사용자 코드가 직접 계산하는 것이 아니라, 운영체제 커널에게 일을 요청하는 경우다.

예:

```cpp
#include <unistd.h>
#include <fcntl.h>

int main() {
    char buf[4096];

    int fd = open("bigfile.txt", O_RDONLY);
    while (read(fd, buf, sizeof(buf)) > 0) {
        // 읽기만 하고 별도 계산은 거의 안 함
    }
    close(fd);
}
```

이 프로그램은 파일을 계속 읽는다.

예상 출력:

```text
real 1.20
user 0.03
sys  0.80
```

여기서 `sys`가 크다면, 커널 작업이 많다는 신호일 수 있다.

---

# 3. 왜 필요한지

성능 문제를 분석할 때 가장 먼저 해야 할 일은 이것이다.

> **느린 이유가 CPU 계산 때문인지, 기다림 때문인지 구분한다.**

같은 10초라도 완전히 다른 문제일 수 있다.

## 3.1 CPU-bound 프로그램

```text
real 10.0
user 9.8
sys  0.1
```

해석:

```text
거의 10초 동안 CPU가 바빴다.
계산이 많을 가능성이 크다.
```

관심 대상:

```text
알고리즘
loop
함수 호출
branch
cache miss
data structure
string processing
numeric computation
```

---

## 3.2 I/O-bound 프로그램

```text
real 10.0
user 0.3
sys  0.2
```

해석:

```text
사람은 10초 기다렸지만 CPU는 0.5초 정도만 일했다.
나머지는 기다림일 가능성이 크다.
```

관심 대상:

```text
disk I/O
network I/O
database query
blocking read
sleep
lock wait
external process wait
```

---

## 3.3 system-call-heavy 프로그램

```text
real 10.0
user 1.0
sys  7.5
```

해석:

```text
커널 작업이 매우 많다.
read/write/stat/open 같은 system call이 과도할 수 있다.
```

관심 대상:

```text
작은 read/write 반복
파일 stat 반복
너무 많은 open/close
많은 process fork/exec
socket syscall 과다
```

---

# 4. 내부 원리 / 작동 방식

## 4.1 프로그램은 CPU를 계속 쓰지 않는다

프로그램이 실행된다는 말은 단순하지 않다.

프로그램은 다음 상태를 오간다.

```text
running
  CPU에서 실제 실행 중

ready
  실행 가능하지만 CPU 차례를 기다림

blocked
  I/O, sleep, lock 등을 기다림
```

그림으로 보면 다음과 같다.

```text
시간 흐름 →
┌─────────┬───────────────┬─────────┬───────────────┐
│ running │ blocked       │ running │ blocked       │
│ CPU 사용│ disk read 대기 │ CPU 사용│ network 대기  │
└─────────┴───────────────┴─────────┴───────────────┘

wall time:
  전체 구간

CPU time:
  running 구간만
```

즉:

```text
wall time = running + waiting + scheduling delay
CPU time  = running
```

---

## 4.2 user mode와 kernel mode

현대 OS는 보통 CPU 실행을 두 영역으로 나눈다.

| 영역          | 설명            |
| ----------- | ------------- |
| user mode   | 일반 프로그램 코드 실행 |
| kernel mode | OS 커널 코드 실행   |

내 C++ 코드가 계산할 때는 user mode다.

```cpp
sum += data[i];
```

하지만 파일을 읽을 때는 커널에게 요청해야 한다.

```cpp
read(fd, buf, 4096);
```

이때 흐름은 대략 이렇다.

```text
내 프로그램
  ↓
read() 호출
  ↓
system call
  ↓
kernel mode 진입
  ↓
커널이 파일 시스템 / 디스크 / page cache 처리
  ↓
user mode 복귀
  ↓
내 프로그램 계속 실행
```

따라서 `sys` time은 다음을 의미한다.

```text
내 프로그램이 요청한 작업 때문에
커널이 CPU를 사용한 시간
```

주의해야 한다.

`sys` time이 크다고 무조건 나쁜 것은 아니다. 파일 처리 프로그램이라면 어느 정도 `sys` time은 정상이다. 하지만 너무 크면 system call 횟수나 방식이 비효율적일 수 있다.

---

## 4.3 blocking I/O

blocking I/O는 요청이 완료될 때까지 프로그램이 멈춰 기다리는 방식이다.

예:

```cpp
int n = read(fd, buf, sizeof(buf));
```

파일이나 socket에서 데이터가 아직 준비되지 않았다면, 프로그램은 기다릴 수 있다.

```text
read() 호출
  ↓
데이터 없음
  ↓
프로세스 blocked
  ↓
CPU는 다른 프로세스 실행
  ↓
데이터 준비
  ↓
프로세스 다시 ready
  ↓
CPU 배정
  ↓
read() 반환
```

이때 프로그램의 wall time은 증가한다.

하지만 CPU time은 많이 증가하지 않을 수 있다.

---

## 4.4 sleep이 wall time과 CPU time에 다르게 보이는 이유

다음 프로그램을 보자.

```cpp
#include <unistd.h>

int main() {
    sleep(3);
    return 0;
}
```

측정:

```bash
/usr/bin/time -p ./sleep_test
```

예상 출력:

```text
real 3.00
user 0.00
sys  0.00
```

왜 그런가?

`sleep(3)`은 CPU에게 “3초 동안 계속 반복문을 돌라”는 뜻이 아니다.

운영체제에게 다음처럼 요청하는 것이다.

```text
이 프로세스를 3초 동안 깨우지 마라.
```

따라서 그 3초 동안 CPU는 다른 일을 한다.

그 결과:

```text
wall time은 증가
CPU time은 거의 증가하지 않음
```

---

# 5. 쉬운 예시

## 5.1 식당 대기 비유

음식을 주문했다고 하자.

```text
주문 후 음식이 나오기까지 20분 걸림
```

이 20분은 wall time이다.

그런데 요리사가 실제로 내 음식에 손을 댄 시간은 5분일 수 있다.

```text
wall time = 20분
actual cooking time = 5분
waiting time = 15분
```

프로그램도 같다.

```text
프로그램 완료까지 20초
CPU가 실제 실행한 시간 5초
나머지 15초는 I/O, lock, network, sleep 대기
```

성능 분석가는 이렇게 질문해야 한다.

> “이 프로그램은 정말 계산하느라 느린가, 아니면 기다리느라 느린가?”

---

## 5.2 공부 시간 비유

책상에 앉아 있던 시간이 5시간이라고 하자.

```text
wall time = 5시간
```

하지만 실제 집중해서 공부한 시간은 2시간일 수 있다.

```text
CPU time = 2시간
```

나머지는 다음일 수 있다.

```text
휴대폰 봄
자료 다운로드 기다림
딴생각
커피 사러 감
```

프로그램도 마찬가지다.

```text
프로그램이 살아 있던 시간 ≠ CPU가 일한 시간
```

---

# 6. 실무 예시

## 6.1 CPU-bound 예시: 비효율적인 계산

```cpp
#include <iostream>

bool isPrime(int n) {
    if (n < 2) return false;

    for (int i = 2; i < n; ++i) {
        if (n % i == 0)
            return false;
    }
    return true;
}

int main() {
    int count = 0;

    for (int n = 2; n < 200000; ++n) {
        if (isPrime(n))
            ++count;
    }

    std::cout << count << std::endl;
}
```

컴파일:

```bash
g++ -O2 -g prime.cpp -o prime
```

측정:

```bash
/usr/bin/time -p ./prime >/dev/null
```

예상:

```text
real 3.20
user 3.18
sys  0.01
```

해석:

```text
real ≈ user
```

즉, 대부분 CPU 계산이다.

분석 흐름:

```text
symptom:
  prime 계산이 오래 걸림

baseline:
  real 3.20 sec

measurement:
  user 3.18, sys 0.01

hypothesis:
  isPrime의 O(n) loop가 너무 비싸다

evidence:
  profiler에서 isPrime이 hot function으로 나타날 가능성 높음

optimization:
  i < n 대신 i * i <= n까지만 검사

re-measure:
  real 감소 확인

prevention:
  입력 크기별 benchmark 추가
```

개선:

```cpp
bool isPrime(int n) {
    if (n < 2) return false;

    for (int i = 2; i * i <= n; ++i) {
        if (n % i == 0)
            return false;
    }
    return true;
}
```

이 경우는 명백히 알고리즘 개선이다.

---

## 6.2 I/O-bound 예시: 네트워크 또는 파일 대기

다음 프로그램은 실제로는 `sleep`으로 I/O 대기를 흉내 낸다.

```cpp
#include <unistd.h>
#include <iostream>

int main() {
    for (int i = 0; i < 5; ++i) {
        std::cout << "request " << i << std::endl;
        sleep(1); // network response wait처럼 생각
    }
}
```

측정:

```bash
/usr/bin/time -p ./wait_test >/dev/null
```

예상:

```text
real 5.00
user 0.00
sys  0.00
```

해석:

```text
CPU가 느린 것이 아니다.
프로그램이 기다리는 구조다.
```

이런 프로그램을 최적화할 때 `for` loop의 미세 최적화는 의미가 없다.

진짜 해결책은 다음일 수 있다.

```text
비동기 I/O
concurrency
batching
timeout 조정
network request 수 감소
cache 사용
```

---

## 6.3 syscall-heavy 예시: 작은 write를 너무 많이 호출

다음 코드를 보자.

```cpp
#include <unistd.h>

int main() {
    const char c = 'x';

    for (int i = 0; i < 1000000; ++i) {
        write(1, &c, 1);
    }

    return 0;
}
```

측정:

```bash
g++ -O2 -g write_many.cpp -o write_many
/usr/bin/time -p ./write_many >/dev/null
```

예상:

```text
real 0.80
user 0.10
sys  0.65
```

해석:

```text
system time이 크다.
write system call을 너무 많이 호출하고 있다.
```

개선 방향:

```text
작은 write 1,000,000번
→ 큰 buffer에 모아서 write 몇 번
```

개선 예:

```cpp
#include <unistd.h>
#include <string>

int main() {
    std::string s;
    s.reserve(1000000);

    for (int i = 0; i < 1000000; ++i) {
        s += 'x';
    }

    write(1, s.data(), s.size());
    return 0;
}
```

C++98에서는 `std::string::data()`가 null-terminated 보장을 C++11처럼 명확히 제공하지 않으므로, 더 보수적으로는 `s.c_str()`를 쓸 수 있다.

```cpp
write(1, s.c_str(), s.size());
```

분석 흐름:

```text
symptom:
  출력 프로그램이 느림

baseline:
  real 0.80 sec

measurement:
  sys 0.65 sec

hypothesis:
  작은 write system call이 너무 많다

evidence:
  strace/dtruss로 write 호출 수 확인 가능

optimization:
  buffering

re-measure:
  real/sys time 감소 확인

prevention:
  반복문 안에서 작은 I/O 금지 규칙화
```

---

# 7. 도구 사용 예시

## 7.1 shell built-in `time`

가장 간단한 사용:

```bash
time ./program
```

bash/zsh에서 `time`은 shell built-in 또는 shell keyword일 수 있다.

출력 예:

```text
./program  2.41s user 0.05s system 98% cpu 2.505 total
```

zsh에서는 이런 형식으로 나올 수 있다.

```text
2.41s user 0.05s system 98% cpu 2.505 total
```

해석:

| 항목       | 의미              |
| -------- | --------------- |
| `user`   | user CPU time   |
| `system` | kernel CPU time |
| `% cpu`  | CPU 사용률 비슷한 지표  |
| `total`  | wall time       |

---

## 7.2 `/usr/bin/time -p`

POSIX 스타일에 가까운 간단한 형식이다.

```bash
/usr/bin/time -p ./program
```

출력:

```text
real 2.50
user 2.41
sys 0.05
```

초보자에게는 이 형식이 가장 읽기 좋다.

---

## 7.3 Linux GNU time

Linux에서는 보통 자세한 정보를 볼 수 있다.

```bash
/usr/bin/time -v ./program
```

출력에는 다음 같은 정보가 포함될 수 있다.

```text
User time (seconds)
System time (seconds)
Percent of CPU this job got
Elapsed (wall clock) time
Maximum resident set size
Major page faults
Minor page faults
File system inputs
File system outputs
Voluntary context switches
Involuntary context switches
```

이 정보는 성능 원인 추정에 유용하다.

예:

| 지표                   | 의심 가능한 문제                        |
| -------------------- | -------------------------------- |
| user time 큼          | CPU 계산 문제                        |
| system time 큼        | system call / kernel 작업 많음       |
| file system inputs 큼 | disk read 영향                     |
| major page faults 큼  | disk-backed memory access        |
| context switches 많음  | blocking, lock, scheduling 영향 가능 |
| max RSS 큼            | memory footprint 문제              |

---

## 7.4 macOS `/usr/bin/time -l`

macOS에서는 다음을 사용한다.

```bash
/usr/bin/time -l ./program
```

출력에는 다음 같은 정보가 포함될 수 있다.

```text
real
user
sys
maximum resident set size
page reclaims
page faults
voluntary context switches
involuntary context switches
```

Linux의 `-v`와 형식이 다르므로 스크립트를 그대로 공유하면 깨질 수 있다.

---

## 7.5 stdout redirect

성능 측정할 때 출력이 많으면 반드시 redirect를 고려해야 한다.

```bash
/usr/bin/time -p ./program >/dev/null
```

왜냐하면 terminal 출력은 느릴 수 있다.

예:

```cpp
#include <iostream>

int main() {
    for (int i = 0; i < 1000000; ++i) {
        std::cout << i << "\n";
    }
}
```

이 프로그램을 그냥 실행하면, 실제 계산보다 터미널 출력이 병목일 수 있다.

측정할 때는 목적을 분리해야 한다.

```text
계산 성능을 보고 싶은가?
출력까지 포함한 end-to-end 성능을 보고 싶은가?
```

두 경우는 다르다.

---

# 8. macOS / Linux 차이

## 8.1 `time` 명령

| 항목             | Linux              | macOS              |
| -------------- | ------------------ | ------------------ |
| 간단 측정          | `/usr/bin/time -p` | `/usr/bin/time -p` |
| 자세한 측정         | `/usr/bin/time -v` | 기본 제공 안 됨          |
| macOS 상세       | 해당 없음              | `/usr/bin/time -l` |
| shell built-in | bash/zsh마다 다름      | zsh 기본 출력 형식 다름    |

macOS에서 GNU time이 필요하면 보통 Homebrew로 `gtime`을 설치해서 쓰기도 한다.

```bash
gtime -v ./program
```

다만 이 강의에서는 기본 도구 기준으로 설명한다.

---

## 8.2 system call 추적 도구

system time이 너무 크면 system call을 보고 싶을 수 있다.

Linux:

```bash
strace -c ./program
```

macOS:

```bash
sudo dtruss ./program
```

주의:

```text
strace/dtruss는 overhead가 크다.
그 자체로 프로그램을 느리게 만들 수 있다.
```

따라서 실행 시간 자체보다는 system call 패턴 확인에 더 적합하다.

예:

```text
write가 1,000,000번 호출됨
open/stat가 너무 많이 호출됨
read buffer가 너무 작음
```

이런 단서를 얻는 용도다.

---

# 9. 흔한 오해

## 오해 1. `real = user + sys`라고 생각하기

단일 CPU에서 CPU-bound 단일 스레드 프로그램이라면 대략 비슷할 수 있다.

```text
real ≈ user + sys
```

하지만 항상 그렇지는 않다.

멀티스레드 프로그램에서는 `user + sys`가 `real`보다 클 수 있다.

예:

```text
real 2.0
user 7.5
sys  0.3
```

왜 가능한가?

4개 CPU core에서 동시에 계산하면, 2초 동안 여러 thread가 CPU를 쓴다.

```text
wall time:
  2초

CPU time:
  thread별 CPU 사용 시간을 합산
  총 7.8초 가능
```

따라서 멀티스레드에서는 다음이 가능하다.

```text
CPU time > wall time
```

---

## 오해 2. `user`가 크면 무조건 나쁘다

아니다.

계산 프로그램이라면 user time이 큰 것은 정상이다.

문제는 다음이다.

```text
필요한 계산인가?
알고리즘이 적절한가?
같은 결과를 더 적은 계산으로 만들 수 있는가?
```

예를 들어 영상 인코딩, 압축, 암호화, 머신러닝 학습은 user time이 큰 것이 자연스럽다.

---

## 오해 3. `sys`가 크면 OS가 나쁜 것이다

대부분 아니다.

`sys` time은 내 프로그램이 커널에게 일을 많이 시킨 결과다.

원인은 보통 다음이다.

```text
작은 read/write 반복
너무 많은 파일 metadata 조회
과도한 process 생성
socket syscall 과다
memory allocation 관련 kernel 개입
page fault
```

즉, OS 탓이 아니라 프로그램 구조 문제일 수 있다.

---

## 오해 4. `real`만 보면 충분하다

아니다.

`real`은 사용자가 체감하는 시간이라 중요하다. 하지만 원인 분석에는 부족하다.

예:

```text
Case A:
real 10
user 9.8
sys 0.1

Case B:
real 10
user 0.2
sys 0.1
```

둘 다 사용자는 10초 기다린다.

하지만 원인은 완전히 다르다.

| Case | 원인 후보  | 해결 방향                               |
| ---- | ------ | ----------------------------------- |
| A    | CPU 계산 | 알고리즘, data structure, profiler      |
| B    | 대기     | I/O, network, lock, async, batching |

---

## 오해 5. 첫 실행 시간이 항상 대표값이다

아니다.

첫 실행은 cold cache일 수 있다.

```text
Run 1: 8.0 sec
Run 2: 3.5 sec
Run 3: 3.4 sec
```

이런 경우 첫 실행과 이후 실행을 분리해서 보고해야 한다.

보고 예:

```text
Cold run:
  8.0 sec

Warm runs:
  avg 3.45 sec over 2 runs
```

파일 처리 프로그램에서는 이 차이가 매우 중요하다.

---

# 10. Latency와 Throughput

성능 분석에서는 시간만 보는 것이 아니라, 어떤 성능 지표를 볼지도 정해야 한다.

## 10.1 latency

**latency**는 하나의 작업이 완료되는 데 걸리는 시간이다.

```text
요청 하나가 들어와서 응답이 나갈 때까지 120ms
```

예:

```text
HTTP request latency
DB query latency
파일 하나 처리 시간
검색 한 번의 응답 시간
```

사용자 경험과 직접 연결된다.

---

## 10.2 throughput

**throughput**은 단위 시간당 처리량이다.

```text
초당 10,000 requests
초당 500MB 처리
분당 1,000 jobs
```

예:

```text
log parser: 250 MB/s
server: 3,000 requests/sec
batch system: 10,000 records/sec
```

---

## 10.3 latency와 throughput은 다를 수 있다

예를 들어 식당을 생각하자.

| 상황                    | latency | throughput |
| --------------------- | ------: | ---------: |
| 요리사 1명, 주문 하나씩 빠르게 처리 | 낮을 수 있음 |         낮음 |
| 대형 주방, 주문을 batch로 처리  | 높을 수 있음 |         높음 |

프로그램도 같다.

batching은 throughput을 높이지만 개별 latency를 늘릴 수 있다.

예:

```text
로그를 1줄마다 즉시 전송:
  latency 낮음
  throughput 낮음

로그를 1000줄 모아서 전송:
  latency 증가
  throughput 증가
```

따라서 성능 목표를 명확히 해야 한다.

```text
사용자 클릭 반응을 줄이고 싶은가?
전체 batch 처리량을 높이고 싶은가?
서버가 초당 더 많은 요청을 처리하게 하고 싶은가?
```

---

# 11. Cold Cache vs Warm Cache

## 11.1 cache는 여러 층이 있다

성능에서 cache라고 하면 여러 가지가 있다.

```text
CPU L1/L2/L3 cache
OS page cache
database cache
application cache
browser cache
CDN cache
```

Lecture 16에서는 주로 OS page cache와 CPU cache를 생각하면 된다.

---

## 11.2 OS page cache

파일을 처음 읽으면 디스크에서 데이터를 가져와야 한다.

```text
Program
  ↓
read file
  ↓
OS page cache에 없음
  ↓
disk에서 읽음
  ↓
느림
```

두 번째 읽으면 이미 메모리에 있을 수 있다.

```text
Program
  ↓
read file
  ↓
OS page cache에 있음
  ↓
memory에서 읽음
  ↓
빠름
```

그래서 파일 처리 프로그램은 첫 실행과 두 번째 실행 시간이 달라질 수 있다.

```text
Run 1: 12.0 sec
Run 2: 4.0 sec
Run 3: 4.1 sec
```

---

## 11.3 CPU cache

CPU는 메모리보다 훨씬 빠른 cache를 사용한다.

연속적인 vector 접근은 cache에 유리하다.

```cpp
for (size_t i = 0; i < v.size(); ++i) {
    sum += v[i];
}
```

반면 pointer를 따라 여기저기 이동하는 구조는 cache에 불리할 수 있다.

```cpp
Node* p = head;
while (p) {
    sum += p->value;
    p = p->next;
}
```

같은 개수의 원소를 처리해도 `vector`와 linked list의 성능이 크게 다를 수 있다.

이것은 Lecture 19에서 algorithm/data structure 병목을 볼 때 다시 중요해진다.

---

# 12. 단계별 분석 흐름

성능 문제가 들어오면 `time` 결과를 먼저 본다.

## Case 1

```text
real 10.0
user 9.8
sys  0.1
```

분석:

```text
symptom:
  실행 시간이 10초

baseline:
  real 10.0

measurement:
  user가 대부분

hypothesis:
  CPU-bound 계산 문제

evidence:
  sampling profiler로 hot function 확인

optimization:
  알고리즘, 자료구조, 불필요한 연산 제거

re-measure:
  real/user 감소 확인

prevention:
  대표 입력 benchmark 추가
```

---

## Case 2

```text
real 10.0
user 0.4
sys  0.2
```

분석:

```text
symptom:
  실행 시간이 10초

baseline:
  real 10.0

measurement:
  CPU time은 작음

hypothesis:
  I/O wait, sleep, lock, network wait 가능성

evidence:
  logs, tracing, strace/dtruss, request timing

optimization:
  batching, async, caching, timeout 조정, I/O 횟수 감소

re-measure:
  real 감소 확인

prevention:
  slow I/O metrics 추가
```

---

## Case 3

```text
real 10.0
user 1.0
sys  8.0
```

분석:

```text
symptom:
  실행 시간이 10초

baseline:
  real 10.0

measurement:
  sys time이 큼

hypothesis:
  system call 과다 또는 kernel 작업 과다

evidence:
  strace -c, dtruss, 파일 I/O 패턴 확인

optimization:
  buffering, syscall 횟수 감소, batch 처리

re-measure:
  sys time 감소 확인

prevention:
  반복문 내부 작은 I/O 방지
```

---

## Case 4

```text
real 2.0
user 7.5
sys  0.3
```

분석:

```text
symptom:
  2초 동안 끝났지만 CPU total은 7.8초

measurement:
  user + sys > real

hypothesis:
  멀티스레드 CPU-bound 작업

evidence:
  thread 수, CPU core 사용률, profiler thread view

optimization:
  thread 간 work balance, lock contention, algorithm 개선

re-measure:
  wall time과 CPU time 둘 다 확인

prevention:
  throughput benchmark와 CPU efficiency 지표 추가
```

---

# 13. 확인 문제

## 문제 1

다음 결과를 해석하라.

```text
real 5.00
user 0.01
sys  0.00
```

좋은 답변:

```text
프로그램은 5초 걸렸지만 CPU는 거의 쓰지 않았다.
sleep, blocking I/O, network wait, lock wait 가능성이 있다.
```

---

## 문제 2

다음 결과를 해석하라.

```text
real 3.20
user 3.15
sys  0.02
```

좋은 답변:

```text
대부분 CPU user code에서 시간을 썼다.
CPU-bound 계산 문제일 가능성이 있다.
다음 단계는 sampling profiler로 hot function을 찾는 것이다.
```

---

## 문제 3

다음 결과를 해석하라.

```text
real 1.00
user 3.70
sys  0.10
```

좋은 답변:

```text
CPU time이 wall time보다 크다.
멀티스레드 프로그램이 여러 core에서 병렬로 실행되었을 가능성이 있다.
```

---

## 문제 4

`stdout`이 많은 프로그램을 측정할 때 왜 `>/dev/null`을 사용할 수 있는가?

좋은 답변:

```text
터미널 출력 비용이 계산 시간 측정을 왜곡할 수 있기 때문이다.
계산 성능을 분리해서 보고 싶다면 출력 비용을 제거할 수 있다.
단, end-to-end 성능을 측정하려면 출력 포함 측정도 따로 해야 한다.
```

---

## 문제 5

cold cache와 warm cache의 차이를 설명하라.

좋은 답변:

```text
cold cache는 필요한 데이터가 cache에 없는 상태이고, warm cache는 이전 실행이나 접근으로 데이터가 cache에 남아 있는 상태다.
파일 처리 프로그램에서는 첫 실행이 느리고 두 번째 실행이 빨라질 수 있다.
```

---

# 14. 실습 과제

## 과제 1. sleep 프로그램 측정

파일명: `sleep_test.cpp`

```cpp
#include <unistd.h>

int main() {
    sleep(3);
    return 0;
}
```

컴파일:

```bash
g++ -O2 -g sleep_test.cpp -o sleep_test
```

측정:

```bash
/usr/bin/time -p ./sleep_test
```

예상:

```text
real ≈ 3
user ≈ 0
sys  ≈ 0
```

보고:

```text
Symptom:
  프로그램이 3초 걸림

Measurement:
  real:
  user:
  sys:

Interpretation:
  CPU 계산이 아니라 sleep 때문에 wall time이 증가했다.
```

---

## 과제 2. CPU-bound 프로그램 측정

파일명: `cpu_test.cpp`

```cpp
#include <iostream>

int main() {
    volatile long long sum = 0;

    for (long long i = 0; i < 1000000000LL; ++i) {
        sum += i;
    }

    std::cout << sum << std::endl;
}
```

컴파일:

```bash
g++ -O2 -g cpu_test.cpp -o cpu_test
```

측정:

```bash
/usr/bin/time -p ./cpu_test >/dev/null
```

예상:

```text
real ≈ user
sys 작음
```

보고:

```text
Symptom:
  계산 loop가 오래 걸림

Measurement:
  real:
  user:
  sys:

Interpretation:
  CPU-bound 가능성이 높다.
```

---

## 과제 3. 작은 write 반복 측정

파일명: `write_many.cpp`

```cpp
#include <unistd.h>

int main() {
    const char c = 'x';

    for (int i = 0; i < 1000000; ++i) {
        write(1, &c, 1);
    }

    return 0;
}
```

컴파일:

```bash
g++ -O2 -g write_many.cpp -o write_many
```

측정:

```bash
/usr/bin/time -p ./write_many >/dev/null
```

보고:

```text
Measurement:
  real:
  user:
  sys:

Interpretation:
  sys time이 상대적으로 크다면 작은 write system call 반복이 원인일 수 있다.
```

추가 실험:

```cpp
#include <unistd.h>
#include <string>

int main() {
    std::string s;
    s.reserve(1000000);

    for (int i = 0; i < 1000000; ++i) {
        s += 'x';
    }

    write(1, s.c_str(), s.size());
    return 0;
}
```

다시 측정하고 비교하라.

---

## 과제 4. 성능 보고서 작성

아래 형식으로 세 프로그램을 비교하라.

```text
Program:
  sleep_test / cpu_test / write_many

Workload:
  무엇을 수행했는가?

Environment:
  OS:
  Compiler:
  Flags:

Measurement:
  real:
  user:
  sys:

Classification:
  CPU-bound / I/O-bound / syscall-heavy / sleep-wait

Evidence:
  왜 그렇게 판단했는가?

Next step:
  profiler가 필요한가?
  strace/dtruss가 필요한가?
  코드 변경이 필요한가?
```

---

# 15. 핵심 정리

이번 강의의 핵심은 다음이다.

```text
wall time은 사람이 기다린 전체 시간이다.
```

```text
CPU time은 CPU가 실제로 프로그램을 실행한 시간이다.
```

```text
user time은 사용자 공간 코드 실행 시간이다.
```

```text
system time은 커널이 내 프로그램을 위해 실행한 시간이다.
```

```text
real이 크고 user/sys가 작으면 CPU가 아니라 대기 문제일 수 있다.
```

```text
real과 user가 비슷하게 크면 CPU-bound 가능성이 높다.
```

```text
sys가 크면 system call 또는 kernel 작업이 많은지 의심해야 한다.
```

```text
멀티스레드 프로그램에서는 user + sys가 real보다 클 수 있다.
```

```text
latency는 한 작업의 응답 시간이고, throughput은 단위 시간당 처리량이다.
```

```text
cold cache와 warm cache는 측정값을 크게 바꿀 수 있다.
```

성능 분석은 항상 다음 흐름을 따른다.

```text
symptom
→ baseline
→ measurement
→ hypothesis
→ evidence
→ optimization
→ re-measure
→ prevention
```

이번 Lecture의 핵심 판단법은 이것이다.

```text
먼저 real/user/sys를 보고,
CPU 계산 문제인지,
I/O 대기 문제인지,
kernel/syscall 문제인지
대략 분류한다.
```