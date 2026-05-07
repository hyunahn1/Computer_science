# Profiling Foundations 전체 커리큘럼

| Lecture | 주제                           | 핵심 목표                                  |
| ------: | ---------------------------- | -------------------------------------- |
|      15 | Performance Debugging이란 무엇인가 | “느리다”를 측정 가능한 문제로 바꾸는 법                |
|      16 | Time의 종류                     | wall time, CPU time, user/sys time 구분  |
|      17 | Profiler의 내부 모델              | sampling / instrumentation profiler 이해 |
|      18 | CPU Flame Graphs             | flame graph로 hot path 읽기               |
|      19 | Slow Function Profiling      | 느린 함수의 원인 분류와 개선 보고                    |

Part D 전체의 핵심 흐름은 항상 같다.

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

성능 분석에서 가장 위험한 태도는 이것이다.

> “아마 이 부분이 느릴 것 같다.”

전문가는 이렇게 말한다.

> “이 workload에서 baseline은 3.8초이고, profiler상 전체 CPU sample의 42%가 `parseRecord()` 아래에서 발생했다. 그중 대부분은 문자열 복사와 allocation이다. 따라서 첫 번째 hypothesis는 allocation pressure다.”

---

# Lecture 15. Performance Debugging이란 무엇인가

## 1. 핵심 질문

이번 강의의 핵심 질문은 다음이다.

> **성능 문제는 어떻게 디버깅해야 하는가?**

더 구체적으로는 다음 질문을 다룬다.

1. correctness bug와 performance bug는 무엇이 다른가?
2. “느리다”는 말이 왜 불충분한가?
3. 왜 최적화 전에 반드시 측정해야 하는가?
4. benchmark와 profiling은 어떻게 다른가?
5. baseline은 어떻게 잡는가?
6. 개선 전후 비교는 어떻게 해야 신뢰할 수 있는가?
7. 성능 보고서는 어떻게 작성해야 하는가?

---

## 2. 개념 설명

### 2.1 Correctness bug vs Performance bug

먼저 두 종류의 bug를 구분해야 한다.

| 구분    | Correctness bug                           | Performance bug                            |
| ----- | ----------------------------------------- | ------------------------------------------ |
| 핵심 문제 | 결과가 틀림                                    | 결과는 맞지만 너무 느림                              |
| 예시    | segfault, wrong output, memory corruption | 응답 지연, CPU 과다 사용, 메모리 할당 과다                |
| 판단 기준 | expected output과 actual output 비교         | baseline, latency, throughput, CPU time 비교 |
| 재현 방식 | 특정 입력에서 오류 확인                             | 특정 workload에서 시간/자원 측정                     |
| 도구    | debugger, sanitizer, logs                 | benchmark, profiler, tracing, metrics      |
| 위험    | 프로그램이 잘못 동작                               | 사용자가 기다리거나 시스템 비용 증가                       |

예를 들어 C++ 프로그램에서 정렬 결과가 틀리면 correctness bug다.

```cpp
std::sort(v.begin(), v.end());
// 결과 순서가 틀림
```

반면 결과는 맞지만 10만 개 입력에서는 0.1초, 100만 개 입력에서는 80초가 걸린다면 performance bug다.

```cpp
// 결과는 맞지만 너무 느림
for (size_t i = 0; i < v.size(); ++i) {
    for (size_t j = 0; j < v.size(); ++j) {
        // O(n^2) work
    }
}
```

Correctness bug는 보통 “참/거짓” 성격이 강하다.

```text
정답인가? 아니다.
crash가 나는가? 난다.
메모리 오류가 있는가? 있다.
```

Performance bug는 더 애매하다.

```text
느린가? 어느 정도로?
어떤 입력에서?
비교 대상은?
어느 환경에서?
평균인가? 최악인가?
한 번만 느린가? 계속 느린가?
```

그래서 성능 문제는 반드시 숫자와 조건으로 표현해야 한다.

---

## 3. 왜 필요한지

초보자는 성능 문제를 보면 보통 이렇게 접근한다.

```text
1. 코드가 느린 것 같다.
2. 눈에 보이는 for loop를 고친다.
3. std::vector 대신 raw array를 써본다.
4. 함수 inline을 붙여본다.
5. 별 차이가 없다.
6. 더 복잡한 코드가 된다.
```

이 방식은 위험하다.

성능 최적화는 다음 세 가지를 쉽게 망가뜨린다.

| 위험                | 설명                       |
| ----------------- | ------------------------ |
| correctness 손상    | 빠르게 만들다가 결과가 틀릴 수 있다     |
| readability 저하    | 복잡한 최적화 코드가 유지보수를 어렵게 한다 |
| false improvement | 실제 문제와 무관한 부분을 고칠 수 있다   |

성능 분석의 목적은 무조건 코드를 빠르게 만드는 것이 아니다.

> **어디가, 왜, 얼마나 느린지 증거를 확보하고, 최소한의 변경으로 검증 가능한 개선을 만드는 것**이다.

---

## 4. 내부 원리 / 작동 방식

## 4.1 성능 문제 분석의 전체 모델

항상 다음 흐름으로 접근한다.

```text
[1] symptom
     ↓
[2] baseline
     ↓
[3] measurement
     ↓
[4] hypothesis
     ↓
[5] evidence
     ↓
[6] optimization
     ↓
[7] re-measure
     ↓
[8] prevention
```

각 단계를 하나씩 보자.

---

### 4.2 Symptom: 증상

성능 문제는 보통 모호한 말로 시작한다.

```text
“프로그램이 느려요.”
“서버가 버벅거려요.”
“검색이 오래 걸려요.”
“빌드가 너무 오래 걸려요.”
```

이 상태에서는 아직 분석할 수 없다.

성능 symptom은 다음처럼 바꿔야 한다.

```text
입력 파일 1GB를 처리할 때,
기존 버전에서는 4.2초 걸리던 작업이,
현재 브랜치에서는 11.8초 걸린다.
```

좋은 symptom에는 최소한 다음 정보가 들어간다.

| 항목       | 예시                     |
| -------- | ---------------------- |
| workload | 1GB 로그 파일 파싱           |
| 환경       | macOS M1, clang++, -O2 |
| 기준       | 이전 버전 4.2초             |
| 현재       | 현재 버전 11.8초            |
| 문제       | 약 2.8배 느려짐             |

---

### 4.3 Baseline: 기준선

Baseline은 “개선 전의 측정값”이다.

예를 들어 다음과 같이 측정했다고 하자.

```text
$ ./parser access.log
Processed 10,000,000 lines
Elapsed: 8.42 sec
```

이것이 baseline이다.

하지만 한 번 측정한 값은 신뢰하기 어렵다. OS 스케줄링, 백그라운드 프로세스, 디스크 캐시, CPU 온도, 전원 상태 때문에 흔들릴 수 있다.

따라서 최소한 여러 번 측정해야 한다.

```text
Run 1: 8.42 sec
Run 2: 8.31 sec
Run 3: 8.38 sec
Run 4: 8.35 sec
Run 5: 8.40 sec
```

이 정도면 baseline을 이렇게 보고할 수 있다.

```text
Baseline: 8.37 sec 평균, 대략 ±0.05 sec 변동
```

성능 분석에서 baseline 없이 최적화하는 것은, 디버깅에서 재현 없이 코드를 고치는 것과 비슷하다.

---

### 4.4 Measurement: 측정

성능 측정에는 여러 종류가 있다.

| 측정 대상            | 질문                             |
| ---------------- | ------------------------------ |
| elapsed time     | 전체 작업이 몇 초 걸렸는가?               |
| CPU time         | CPU를 실제로 얼마나 사용했는가?            |
| memory usage     | 메모리를 얼마나 사용했는가?                |
| allocation count | heap allocation이 얼마나 자주 발생했는가? |
| I/O time         | 파일, 네트워크, 디스크 대기 시간이 큰가?       |
| call cost        | 어떤 함수가 많은 시간을 먹는가?             |

Lecture 15에서는 가장 기본적인 elapsed time 중심으로 시작한다. Lecture 16에서 wall time / CPU time을 더 깊게 분해한다.

---

### 4.5 Hypothesis: 가설

측정 후에는 바로 코드를 고치면 안 된다. 먼저 가설을 세워야 한다.

나쁜 가설:

```text
이 함수가 왠지 느릴 것 같다.
```

좋은 가설:

```text
입력 크기가 커질수록 시간이 거의 n²처럼 증가한다.
따라서 nested loop 또는 map/vector 탐색 방식에 알고리즘 문제가 있을 수 있다.
```

또는:

```text
CPU 사용률은 낮은데 wall time이 길다.
따라서 CPU bound가 아니라 I/O wait 또는 blocking 가능성이 있다.
```

또는:

```text
profiler에서 std::string 생성자와 allocator가 많이 보인다.
따라서 반복적인 문자열 복사와 heap allocation이 병목일 수 있다.
```

가설은 반드시 **측정 가능한 주장**이어야 한다.

---

### 4.6 Evidence: 증거

가설은 증거로 확인해야 한다.

예를 들어 “allocation이 문제다”라는 가설이 있다면 다음 증거가 필요하다.

```text
- profiler에서 allocator 관련 함수가 상위권에 있음
- allocation count가 비정상적으로 많음
- reserve() 적용 후 allocation count 감소
- re-measure 결과 elapsed time 감소
```

증거 없이 최적화하면 그건 engineering이 아니라 추측이다.

---

### 4.7 Optimization: 최적화

최적화는 마지막에 한다.

성능 최적화의 원칙은 다음이다.

```text
먼저 가장 큰 병목을 고친다.
작은 병목부터 고치지 않는다.
```

왜냐하면 Amdahl’s Law 때문이다.

간단히 말하면, 전체 시간의 5%를 차지하는 코드를 아무리 빠르게 해도 전체 성능은 크게 좋아지지 않는다.

예를 들어 전체 10초 중 어떤 함수가 0.5초를 차지한다고 하자. 이 함수를 완전히 0초로 만들어도 전체는 9.5초가 된다.

```text
10.0 sec → 9.5 sec
```

반면 전체 10초 중 6초를 차지하는 부분을 절반으로 줄이면 다음과 같다.

```text
10.0 sec → 7.0 sec
```

따라서 profiler는 “어디를 고칠지” 결정하는 데 중요하다.

---

### 4.8 Re-measure: 다시 측정

최적화 후에는 반드시 다시 측정해야 한다.

```text
Before:
  avg 8.37 sec

After:
  avg 5.91 sec

Improvement:
  29.4% faster
```

주의할 점이 있다.

성능 개선은 다음처럼 보고하면 안 된다.

```text
많이 빨라진 것 같다.
체감상 좋아졌다.
```

다음처럼 보고해야 한다.

```text
동일한 workload에서 5회 실행 평균 기준,
8.37초에서 5.91초로 감소했다.
약 29.4% wall time 개선이다.
```

---

### 4.9 Prevention: 재발 방지

성능 문제를 한 번 고쳤다면, 다시 느려지지 않게 해야 한다.

방법은 다음과 같다.

| 방법                   | 설명                                     |
| -------------------- | -------------------------------------- |
| benchmark 추가         | 핵심 workload에 대한 성능 테스트 작성              |
| regression threshold | 특정 시간 이상 느려지면 경고                       |
| CI에서 측정              | 안정적 환경이면 자동 측정                         |
| 코드 리뷰 체크             | O(n²), 과도한 allocation, blocking I/O 확인 |
| 성능 보고서 저장            | 나중에 비교할 기준 유지                          |

다만 CI 성능 측정은 노이즈가 크다. 그래서 절대값 기준보다는 추세나 큰 regression 감지에 더 적합하다.

---

## 5. 쉬운 예시

## 5.1 “방 청소가 느리다”는 말의 문제

누군가 이렇게 말한다고 하자.

```text
방 청소가 너무 느려.
```

이건 성능 분석이 불가능한 문장이다.

다음처럼 바꿔야 한다.

```text
10평 방을 청소하는 데 평소 20분 걸렸는데,
오늘은 45분 걸렸다.
```

이제 분석이 가능하다.

```text
symptom:
  청소 시간이 20분 → 45분으로 증가

baseline:
  평소 20분

measurement:
  오늘 45분

hypothesis:
  물건을 분류하는 시간이 늘었나?
  청소기 배터리가 약했나?
  동선이 비효율적이었나?

evidence:
  실제로 옷 분류에 25분 사용

optimization:
  옷 바구니를 먼저 분리

re-measure:
  다음 청소 27분

prevention:
  옷을 매일 같은 위치에 두기
```

성능 분석도 이와 같다.

“느리다”가 아니라 **무엇이, 어떤 조건에서, 얼마나 느린지**를 말해야 한다.

---

## 6. 실무 예시

## 6.1 C++98 로그 파서 예시

다음과 같은 C++98 로그 파서가 있다고 하자.

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <vector>

struct Record {
    std::string ip;
    std::string path;
    int status;
};

Record parseLine(const std::string& line) {
    Record r;

    // 예시를 단순화한 파싱 로직
    size_t firstSpace = line.find(' ');
    size_t secondSpace = line.find(' ', firstSpace + 1);

    r.ip = line.substr(0, firstSpace);
    r.path = line.substr(firstSpace + 1, secondSpace - firstSpace - 1);
    r.status = 200;

    return r;
}

int main() {
    std::ifstream file("access.log");
    std::string line;
    std::vector<Record> records;

    while (std::getline(file, line)) {
        records.push_back(parseLine(line));
    }

    std::cout << "records: " << records.size() << std::endl;
}
```

이 프로그램이 느리다고 하자.

초보자는 이렇게 생각할 수 있다.

```text
std::vector가 느린가?
std::string이 느린가?
ifstream이 느린가?
parseLine이 느린가?
```

아직 모른다.

전문적으로는 이렇게 접근한다.

---

### 6.2 Symptom

```text
1GB access.log 처리 시간이 22초 걸린다.
기존 Python 스크립트보다도 느리다.
```

아직 부족하다.

더 정확하게 만든다.

```text
Input:
  access.log, 1GB, 약 10 million lines

Environment:
  Linux x86_64, g++ -O2

Observed:
  avg 22.4 sec over 5 runs

Expected:
  10 sec 이하
```

---

### 6.3 Baseline

```text
Run 1: 22.8 sec
Run 2: 22.1 sec
Run 3: 22.5 sec
Run 4: 22.3 sec
Run 5: 22.4 sec

Baseline:
  avg 22.42 sec
```

---

### 6.4 Measurement

먼저 간단히 `/usr/bin/time`으로 측정한다고 하자.

```bash
/usr/bin/time -p ./parser
```

출력 예:

```text
records: 10000000
real 22.42
user 18.80
sys 3.10
```

해석은 Lecture 16에서 자세히 하겠지만, 지금은 대략 이렇게 보면 된다.

| 항목   | 의미                 |
| ---- | ------------------ |
| real | 실제로 기다린 시간         |
| user | 내 코드가 CPU에서 실행된 시간 |
| sys  | 커널 작업에 사용된 CPU 시간  |

여기서는 `user + sys`가 `real`에 가깝다.

```text
user + sys = 21.90 sec
real = 22.42 sec
```

즉, CPU를 꽤 많이 쓰고 있다. 단순히 디스크를 기다리는 문제만은 아닐 수 있다.

---

### 6.5 Hypothesis

가능한 가설:

```text
parseLine에서 substr이 매 줄마다 std::string을 새로 만들고 있다.
10 million lines라면 문자열 allocation과 copy가 매우 많을 수 있다.
```

또 다른 가설:

```text
records vector가 커지면서 capacity 재할당이 반복될 수 있다.
reserve가 없기 때문에 중간중간 큰 복사가 발생할 수 있다.
```

---

### 6.6 Evidence

증거를 얻기 전에는 고치면 안 된다.

다음과 같은 profiler 결과가 있다고 하자.

```text
35% std::string::_M_construct
21% malloc / operator new
18% parseLine
9%  std::vector<Record>::push_back
```

이제 allocation hypothesis가 강해진다.

---

### 6.7 Optimization

첫 번째 최적화는 `records.reserve()`일 수 있다.

```cpp
std::vector<Record> records;
records.reserve(10000000);
```

다시 측정한다.

```text
Before:
  avg 22.42 sec

After reserve:
  avg 18.90 sec
```

개선은 있지만 충분하지 않다.

다음으로 `substr()`로 string을 계속 만드는 부분을 줄일 수 있다. 다만 C++98에는 `std::string_view`가 없다. 그래서 설계를 바꿔야 할 수 있다.

예를 들어 모든 record를 저장할 필요가 없고 count만 필요하다면, 애초에 `Record`를 만들지 않는 것이 더 큰 개선일 수 있다.

```cpp
while (std::getline(file, line)) {
    // 필요한 값만 바로 처리
}
```

즉, 최적화는 작은 문법 변경보다 **데이터 흐름 변경**이 더 클 때가 많다.

---

## 7. 도구 사용 예시

## 7.1 가장 단순한 baseline 측정

Linux/macOS 공통으로 가장 기본적인 측정은 `time`이다.

```bash
time ./program
```

또는 외부 명령으로:

```bash
/usr/bin/time ./program
```

Linux에서 자세히 보고 싶으면:

```bash
/usr/bin/time -v ./program
```

다만 macOS의 `/usr/bin/time`은 Linux GNU time과 옵션이 다르다. macOS에서는 `-l`을 자주 쓴다.

```bash
/usr/bin/time -l ./program
```

---

## 7.2 여러 번 실행해서 평균 보기

단순하게 shell loop를 사용할 수 있다.

```bash
for i in 1 2 3 4 5; do
    /usr/bin/time -p ./program >/dev/null
done
```

출력 예:

```text
real 8.42
user 7.91
sys 0.39

real 8.31
user 7.86
sys 0.38

real 8.38
user 7.90
sys 0.40
```

이때 중요한 점은 `stdout`을 버리는 것이다.

```bash
>/dev/null
```

왜냐하면 터미널 출력 자체가 느릴 수 있기 때문이다. 특히 많은 양을 출력하는 프로그램은 출력 때문에 측정이 왜곡될 수 있다.

---

## 7.3 C++ 코드 내부에서 시간 측정하기

C++98에서는 `std::chrono`가 없다. 따라서 간단히 `clock()`을 사용할 수 있다.

```cpp
#include <ctime>
#include <iostream>

int main() {
    std::clock_t start = std::clock();

    // 측정할 작업
    volatile long sum = 0;
    for (long i = 0; i < 100000000; ++i) {
        sum += i;
    }

    std::clock_t end = std::clock();

    double seconds =
        static_cast<double>(end - start) / CLOCKS_PER_SEC;

    std::cout << "CPU time: " << seconds << " sec\n";
}
```

주의해야 한다.

`clock()`은 보통 wall time이 아니라 CPU time에 가깝다. 즉, `sleep()`이나 I/O wait은 제대로 반영되지 않을 수 있다.

wall time 측정은 플랫폼별 API가 필요하다.

Linux/macOS에서는 C 계열 API로 `gettimeofday()`를 사용할 수 있다.

```cpp
#include <sys/time.h>
#include <iostream>

double now() {
    timeval tv;
    gettimeofday(&tv, 0);
    return tv.tv_sec + tv.tv_usec / 1000000.0;
}

int main() {
    double start = now();

    // 측정할 작업
    volatile long sum = 0;
    for (long i = 0; i < 100000000; ++i) {
        sum += i;
    }

    double end = now();

    std::cout << "Wall time: " << (end - start) << " sec\n";
}
```

단, `gettimeofday()`는 시스템 시간 변경의 영향을 받을 수 있다. 더 현대적인 방식은 `clock_gettime(CLOCK_MONOTONIC, ...)`이지만 macOS/Linux 차이가 있으므로 이후 강의에서 다룬다.

---

## 8. macOS / Linux 차이

## 8.1 `time` 명령 차이

`time`에는 크게 세 종류가 있다.

| 종류                  | 예시                        | 특징                |
| ------------------- | ------------------------- | ----------------- |
| shell built-in time | `time ./program`          | shell 내부 기능       |
| external time       | `/usr/bin/time ./program` | OS의 별도 실행 파일      |
| GNU time            | Linux에서 흔함                | `-v` 같은 자세한 옵션 제공 |

Linux에서는 보통 다음이 가능하다.

```bash
/usr/bin/time -v ./program
```

예상 출력에는 이런 정보가 포함된다.

```text
Elapsed (wall clock) time
User time
System time
Maximum resident set size
File system inputs
File system outputs
```

macOS에서는 GNU time의 `-v`가 기본으로 없다. 대신:

```bash
/usr/bin/time -l ./program
```

macOS의 `-l`은 memory, page faults 같은 정보를 보여준다.

---

## 8.2 profiler 차이

이 강의에서는 profiler를 깊게 쓰지는 않지만, 방향만 잡자.

| 환경    | 대표 도구                                                |
| ----- | ---------------------------------------------------- |
| Linux | `perf`, `gprof`, Valgrind/Callgrind, flamegraph      |
| macOS | Instruments, `sample`, `dtrace` 계열, Activity Monitor |
| 공통    | 언어별 profiler, custom timing, logging                 |

Linux의 `perf`는 매우 강력하지만 macOS에서는 기본적으로 사용할 수 없다.

macOS에서는 Xcode Instruments가 강력하다. 하지만 CLI 기반으로 공부할 때는 `time`, `sample`, 간단한 코드 instrumentation부터 시작하는 것이 좋다.

---

## 9. 흔한 오해

## 오해 1. “빠른 알고리즘이면 항상 빠르다”

아니다.

이론상 O(n log n)이 O(n²)보다 낫지만, 작은 입력에서는 상수 비용 때문에 O(n²)이 더 빠를 수도 있다.

```text
n = 10일 때:
  단순한 O(n²) 코드가 더 빠를 수 있음

n = 10,000,000일 때:
  알고리즘 복잡도가 지배적일 가능성이 큼
```

그래서 입력 크기를 고려해야 한다.

---

## 오해 2. “한 번 측정하면 충분하다”

아니다.

성능 측정은 흔들린다.

원인:

```text
- OS scheduler
- background process
- disk cache
- CPU frequency scaling
- thermal throttling
- memory layout
- network 상태
```

따라서 여러 번 측정하고 평균, 최소, 최대, 변동성을 봐야 한다.

---

## 오해 3. “profiler가 보여준 1위 함수가 반드시 고칠 함수다”

아니다.

예를 들어 profiler에서 `std::string` 생성자가 1위라고 하자.

그렇다고 표준 라이브러리를 고치는 것이 아니다.

진짜 원인은 보통 그 함수를 많이 호출하게 만든 내 코드다.

```text
겉으로 보이는 hot function:
  std::string constructor

실제 원인:
  parseLine에서 substr을 너무 많이 호출
```

즉, profiler 결과는 직접 답이 아니라 **단서**다.

---

## 오해 4. “microbenchmark가 빠르면 실제 프로그램도 빠르다”

아니다.

microbenchmark는 작은 코드 조각만 측정한다.

예를 들어 다음을 비교한다고 하자.

```cpp
// A
x = x * 2;

// B
x = x << 1;
```

microbenchmark에서는 B가 조금 빠를 수도 있다. 하지만 실제 프로그램 전체에서 이 차이는 의미 없을 수 있다.

실제 병목이 파일 I/O나 allocation이라면 이런 최적화는 거의 효과가 없다.

성능 최적화에서 중요한 질문은 이것이다.

```text
이 코드가 실제 workload에서 전체 시간의 몇 %를 차지하는가?
```

---

## 오해 5. “debug build로 측정해도 된다”

대부분 안 된다.

Debug build는 최적화가 꺼져 있고, symbol과 check가 많고, 코드 배치도 다르다.

예:

```bash
g++ -g main.cpp -o app
```

이렇게 만든 프로그램의 성능은 실제 release build와 크게 다를 수 있다.

성능 측정은 보통 최적화 옵션을 켜고 해야 한다.

```bash
g++ -O2 -g main.cpp -o app
```

여기서 `-g`는 debug symbol을 남겨 profiler가 함수 이름을 더 잘 보여주게 한다. `-O2`는 최적화를 적용한다.

즉, profiling용 build는 보통 다음 조합이 좋다.

```bash
-O2 -g
```

다만 optimization 때문에 함수가 inline되거나 사라져 profiler 해석이 어려워질 수 있다. 이것은 Lecture 17에서 다룬다.

---

## 10. Benchmark와 Profiling의 차이

이 둘은 반드시 구분해야 한다.

| 구분    | Benchmark            | Profiling                    |
| ----- | -------------------- | ---------------------------- |
| 질문    | 얼마나 빠른가?             | 어디서 시간이 쓰이는가?                |
| 결과    | 실행 시간, 처리량, 지연시간     | 함수별 비용, call path, sample 비율 |
| 사용 시점 | 개선 전후 비교             | 병목 위치 탐색                     |
| 예시    | 8.4초 → 5.9초          | `parseLine()` 아래에서 45%       |
| 위험    | workload가 비현실적일 수 있음 | overhead와 sampling bias가 있음  |

간단히 말하면:

```text
benchmark:
  속도계

profiling:
  X-ray
```

Benchmark는 프로그램이 얼마나 빠른지 알려준다.

Profiler는 프로그램 안에서 시간이 어디로 가는지 추정한다.

둘 다 필요하다.

---

## 11. Microbenchmark의 위험성

Microbenchmark는 작은 코드 조각을 따로 떼서 측정하는 것이다.

예:

```cpp
for (int i = 0; i < 100000000; ++i) {
    s += std::string("abc");
}
```

위험한 이유는 다음과 같다.

| 위험                    | 설명                            |
| --------------------- | ----------------------------- |
| compiler optimization | 컴파일러가 코드를 제거할 수 있음            |
| unrealistic workload  | 실제 사용 패턴과 다를 수 있음             |
| cache distortion      | 작은 데이터는 cache에 잘 맞음           |
| branch pattern 단순화    | 실제 분기 패턴과 다름                  |
| I/O 제외                | 실제 병목이 I/O인데 CPU 조각만 측정할 수 있음 |

따라서 microbenchmark는 “작은 부품 비교”에는 유용하지만, 전체 프로그램 성능을 대표하지 않는다.

---

## 12. Representative Workload

성능 분석에서 workload는 입력과 사용 패턴을 의미한다.

나쁜 workload:

```text
작은 테스트 파일 10줄
내 로컬에서 대충 실행
```

좋은 workload:

```text
실제 서비스에서 자주 들어오는 요청 1,000개
실제 로그 크기와 비슷한 1GB 파일
평균 케이스와 최악 케이스를 분리
```

대표 workload는 다음을 포함해야 한다.

| 항목                 | 질문                 |
| ------------------ | ------------------ |
| input size         | 실제 입력 크기와 비슷한가?    |
| input distribution | 실제 데이터 분포와 비슷한가?   |
| hot scenario       | 사용자가 자주 실행하는 경로인가? |
| worst case         | 최악 입력도 측정했는가?      |
| environment        | 실제 배포 환경과 비슷한가?    |

성능 최적화에서 workload를 잘못 잡으면 엉뚱한 코드를 최적화하게 된다.

---

## 13. Variance와 Noise

성능 측정값은 매번 같지 않다.

예:

```text
Run 1: 1.02 sec
Run 2: 1.03 sec
Run 3: 1.01 sec
Run 4: 1.80 sec
Run 5: 1.02 sec
```

여기서 1.80초는 outlier일 수 있다. 백그라운드 작업, disk wait, OS scheduler 등의 영향일 수 있다.

그래서 다음을 함께 봐야 한다.

| 값       | 의미       |
| ------- | -------- |
| average | 평균       |
| median  | 중앙값      |
| min     | 가장 좋은 경우 |
| max     | 가장 나쁜 경우 |
| stddev  | 흔들림 정도   |

초기 학습 단계에서는 최소한 다음 정도만 해도 충분하다.

```text
5회 이상 실행
평균과 범위 기록
이상하게 튄 값은 따로 표시
```

---

## 14. Warm-up과 Cache Effect

첫 실행과 두 번째 실행의 시간이 다를 수 있다.

예:

```text
Run 1: 5.8 sec
Run 2: 3.1 sec
Run 3: 3.0 sec
Run 4: 3.1 sec
```

왜 첫 실행이 느릴까?

가능한 이유:

```text
- 디스크에서 처음 읽음
- 파일이 아직 OS page cache에 없음
- dynamic library loading
- CPU cache가 차갑다
- branch predictor가 아직 패턴을 학습하지 못함
```

이를 cold cache / warm cache라고 부른다.

| 상태         | 의미                          |
| ---------- | --------------------------- |
| cold cache | 데이터가 cache에 별로 없는 상태        |
| warm cache | 이전 실행 때문에 데이터가 cache에 남은 상태 |

성능 보고서에는 이 차이를 명확히 해야 한다.

```text
Cold run:
  첫 실행 포함

Warm run:
  첫 실행 제외 후 평균
```

I/O가 중요한 프로그램은 cold cache와 warm cache 차이가 매우 클 수 있다.

---

## 15. 성능 개선 전후 비교법

성능 개선 보고는 다음 형식을 따른다.

```text
Workload:
  1GB access.log, 약 10 million lines

Environment:
  Linux x86_64, g++ 13, -O2 -g

Baseline:
  avg 22.42 sec over 5 runs

Change:
  records.reserve(10000000) 추가

After:
  avg 18.90 sec over 5 runs

Result:
  15.7% wall time 감소

Evidence:
  vector reallocation 비용 감소 추정
  profiler에서 vector growth 관련 cost 감소
```

개선율 계산은 다음과 같다.

```text
improvement = (before - after) / before * 100
```

예:

```text
before = 22.42
after = 18.90

improvement = (22.42 - 18.90) / 22.42 * 100
            = 15.7%
```

주의할 점:

```text
2초 빨라졌다
```

보다

```text
22.42초에서 18.90초로 감소, 약 15.7% 개선
```

이 더 좋다.

비율과 절대 시간을 함께 보고해야 한다.

---

## 16. 성능 보고서 작성법

좋은 성능 보고서는 다음 구조를 가진다.

```text
1. Problem
2. Workload
3. Environment
4. Baseline
5. Measurement method
6. Hypothesis
7. Evidence
8. Change
9. Result
10. Risk / limitation
11. Prevention
```

예시:

```text
Problem:
  로그 파서가 1GB access.log 처리에 22초 이상 소요됨.

Workload:
  access.log 1GB, 약 10 million lines.

Environment:
  Linux x86_64, g++ -O2 -g, local SSD.

Baseline:
  5회 실행 평균 22.42 sec.

Measurement:
  /usr/bin/time -p 사용.
  stdout은 /dev/null로 redirect.

Hypothesis:
  Record 저장 과정에서 vector 재할당과 string allocation이 과도하게 발생한다고 추정.

Evidence:
  profiler에서 std::string construction, operator new, vector push_back 관련 비용이 상위권.

Change:
  records.reserve(10000000) 추가.

Result:
  5회 실행 평균 18.90 sec.
  약 15.7% wall time 감소.

Risk / limitation:
  입력 line 수를 미리 알 수 없는 경우 reserve 크기 추정 필요.
  다른 크기의 입력에서 동일한 개선이 유지되는지 추가 측정 필요.

Prevention:
  대표 workload benchmark 추가.
  큰 입력에서 실행 시간 regression 확인.
```

이런 보고 방식은 실무에서 매우 중요하다.

왜냐하면 성능 개선은 단순히 “코드를 고쳤다”가 아니라, **의사결정의 근거**를 남기는 일이기 때문이다.

---

## 17. 그림으로 보는 Performance Debugging

```text
사용자 말:
  "느려요"
     ↓
성능 분석가:
  "어떤 workload에서 얼마나 느린가?"
     ↓
Baseline:
  "현재 평균 8.4초"
     ↓
Measurement:
  "CPU를 많이 쓰는가? I/O를 기다리는가?"
     ↓
Profiler:
  "시간이 어느 함수/경로에 몰리는가?"
     ↓
Hypothesis:
  "allocation이 많아서 느린 것 같다"
     ↓
Evidence:
  "operator new / string construction이 상위권"
     ↓
Optimization:
  "불필요한 복사 제거, reserve 추가"
     ↓
Re-measure:
  "8.4초 → 5.9초"
     ↓
Prevention:
  "benchmark 추가, regression 감시"
```

---

## 18. 확인 문제

다음 질문에 답해보면 이번 강의의 핵심을 점검할 수 있다.

### 문제 1

다음 문장은 왜 성능 분석에 부족한가?

```text
우리 프로그램이 너무 느립니다.
```

좋은 답변 방향:

```text
어떤 입력인지, 어느 환경인지, 얼마나 느린지, 이전 기준이 무엇인지 없기 때문이다.
```

---

### 문제 2

다음 중 benchmark와 profiling의 차이를 설명하라.

```text
Benchmark: ./program이 8.2초 걸렸다.
Profiling: parseLine 아래에서 전체 sample의 40%가 발생했다.
```

좋은 답변 방향:

```text
benchmark는 전체 성능 수치이고, profiling은 내부 병목 위치를 추정하는 도구다.
```

---

### 문제 3

왜 성능 측정은 여러 번 해야 하는가?

좋은 답변 방향:

```text
OS scheduling, cache, background process, disk 상태, CPU frequency 등으로 측정값에 variance와 noise가 있기 때문이다.
```

---

### 문제 4

다음 결과를 보고 무엇을 의심할 수 있는가?

```text
real 10.0
user 9.7
sys 0.2
```

좋은 답변 방향:

```text
CPU를 대부분 사용하고 있으므로 CPU-bound 가능성이 있다.
```

단, 확정은 아니다. 더 자세한 profiler 분석이 필요하다.

---

### 문제 5

다음 결과를 보고 무엇을 의심할 수 있는가?

```text
real 10.0
user 0.5
sys 0.3
```

좋은 답변 방향:

```text
실제 기다린 시간은 긴데 CPU 사용 시간은 짧다. I/O wait, sleep, blocking, network 대기 가능성이 있다.
```

---

## 19. 실습 과제

## 과제 1. Baseline 측정하기

아래 C++98 코드를 저장하라.

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <ctime>

int main() {
    std::vector<int> v;

    for (int i = 0; i < 5000000; ++i) {
        v.push_back(i);
    }

    long long sum = 0;
    for (size_t i = 0; i < v.size(); ++i) {
        sum += v[i];
    }

    std::cout << sum << std::endl;
}
```

컴파일:

```bash
g++ -O2 -g main.cpp -o test
```

측정:

```bash
for i in 1 2 3 4 5; do
    /usr/bin/time -p ./test >/dev/null
done
```

기록할 것:

```text
Run 1:
Run 2:
Run 3:
Run 4:
Run 5:

Average:
Min:
Max:
```

---

## 과제 2. `reserve()` 추가 후 다시 측정하기

다음 한 줄을 추가하라.

```cpp
v.reserve(5000000);
```

다시 컴파일하고 측정하라.

```bash
g++ -O2 -g main.cpp -o test

for i in 1 2 3 4 5; do
    /usr/bin/time -p ./test >/dev/null
done
```

보고서 형식:

```text
Workload:
  vector에 int 5,000,000개 push_back 후 sum 계산

Environment:
  OS:
  Compiler:
  Flags:

Before:
  avg ___ sec

Change:
  v.reserve(5000000) 추가

After:
  avg ___ sec

Result:
  ___% 개선 또는 변화 없음

Interpretation:
  reserve가 vector 재할당을 줄였을 가능성이 있다.
  단, profiler 없이 원인을 확정할 수는 없다.
```

---

## 과제 3. 출력 비용 확인하기

이번에는 `>/dev/null`을 제거하고 측정하라.

```bash
/usr/bin/time -p ./test
```

그리고 다시 redirect를 넣어라.

```bash
/usr/bin/time -p ./test >/dev/null
```

질문:

```text
출력량이 많지 않은 경우 차이가 거의 없는가?
출력량이 많은 프로그램이라면 터미널 출력이 측정을 왜곡할 수 있는가?
```

---

## 20. 핵심 정리

이번 강의의 핵심은 다음이다.

```text
성능 문제는 correctness bug와 다르다.
결과가 틀린 것이 아니라, 조건에 비해 너무 느린 것이다.
```

```text
“느리다”는 말은 불충분하다.
workload, environment, baseline, expected behavior가 필요하다.
```

```text
최적화 전에는 반드시 측정해야 한다.
profile, don't guess.
```

```text
benchmark는 전체 속도를 측정한다.
profiling은 시간이 어디서 쓰이는지 추정한다.
```

```text
microbenchmark는 유용하지만 위험하다.
실제 workload를 대표하지 못할 수 있다.
```

```text
baseline은 여러 번 측정해야 한다.
성능 값에는 variance와 noise가 있다.
```

```text
최적화 후에는 반드시 re-measure해야 한다.
개선은 감이 아니라 숫자로 보고해야 한다.
```

```text
성능 분석의 표준 흐름은 항상 같다.

symptom
→ baseline
→ measurement
→ hypothesis
→ evidence
→ optimization
→ re-measure
→ prevention
```