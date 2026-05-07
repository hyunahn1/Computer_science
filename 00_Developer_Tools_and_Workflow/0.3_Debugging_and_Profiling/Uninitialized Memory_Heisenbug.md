# Uninitialized Memory와 Heisenbug

> **초기화되지 않은 메모리는 “랜덤 값이 들어 있다” 정도의 문제가 아니다.**
> C/C++에서는 초기화되지 않은 값을 읽는 순간 프로그램의 의미가 무너질 수 있다.

특히 C/C++ 초보자가 가장 많이 착각하는 부분이 있다.

```text
"초기화 안 된 변수에는 그냥 쓰레기 값이 들어 있겠지."
```

이 말은 반쯤만 맞다.
실제로는 더 위험하다.

```text
초기화되지 않은 값을 읽는 행위는 undefined behavior가 될 수 있다.
undefined behavior가 되면 컴파일러와 실행 결과를 신뢰할 수 없다.
```

---

# 1. 핵심 질문

이번 Lecture 14에서 다룰 질문은 다음이다.

```text
1. uninitialized memory란 정확히 무엇인가?
2. stack garbage와 heap garbage는 어떻게 다른가?
3. 왜 실행할 때마다 결과가 달라질 수 있는가?
4. 왜 debugger를 붙이면 버그가 사라지는가?
5. 왜 optimization 옵션이 버그 양상을 바꾸는가?
6. Heisenbug란 무엇인가?
7. uninitialized memory와 race condition은 어떻게 구분하는가?
8. Valgrind와 sanitizer는 이 문제를 어떻게 감지하는가?
9. deterministic repro는 어떻게 만드는가?
10. 예방하려면 어떤 코딩 습관이 필요한가?
```

---

# 2. 개념 설명

## 2.1 Uninitialized memory란 무엇인가

변수를 선언했지만 값을 넣지 않은 상태를 말한다.

```cpp
int x;
```

이 코드에서 `x`는 존재한다.
하지만 값은 정해져 있지 않다.

```cpp
#include <iostream>

int main()
{
    int x;
    std::cout << x << std::endl;
    return 0;
}
```

여기서 `x`를 읽는 순간 문제가 된다.

```text
int x;
    ↓
stack에 int 크기의 공간 확보
    ↓
하지만 값은 초기화되지 않음
    ↓
std::cout << x
    ↓
초기화되지 않은 값 읽기
```

C++에서 지역 자동 변수는 기본적으로 자동 초기화되지 않는다.

```cpp
void f()
{
    int x;      // 초기화 안 됨
    int y = 0;  // 초기화됨
}
```

---

## 2.2 “메모리 공간이 있다”와 “값이 유효하다”는 다르다

이 차이가 중요하다.

```cpp
int x;
```

이때 `x`의 주소는 존재한다.

```cpp
std::cout << &x << std::endl;
```

주소를 출력하는 것은 보통 문제가 아니다.

하지만:

```cpp
std::cout << x << std::endl;
```

값을 읽는 것은 문제다.

정리하면:

| 상태          | 의미                  |
| ----------- | ------------------- |
| 주소가 있음      | stack에 공간은 잡힘       |
| 값이 초기화됨     | 그 공간 안에 의미 있는 값이 있음 |
| 값이 초기화되지 않음 | 읽으면 위험함             |

Valgrind 관점으로 말하면:

```text
Addressability:
    이 주소에 접근 가능한가?

Validity:
    이 값은 초기화된 값인가?
```

초기화되지 않은 지역 변수는 대체로:

```text
Addressability: OK
Validity: BAD
```

---

# 3. 왜 필요한지

uninitialized memory bug는 디버깅이 어렵다.

이유는 다음과 같다.

```text
1. 항상 crash하지 않는다.
2. 실행할 때마다 값이 바뀔 수 있다.
3. debug build에서는 안 보이다가 release build에서 터질 수 있다.
4. compiler optimization 때문에 코드 흐름이 바뀔 수 있다.
5. debugger를 붙이면 증상이 사라질 수 있다.
6. print 문을 넣으면 증상이 바뀔 수 있다.
```

이런 버그를 **Heisenbug**라고 부르는 경우가 있다.

> 관찰하려고 하면 현상이 바뀌는 버그

물리학의 하이젠베르크 불확정성 원리에서 온 비유다.
정확한 기술 용어라기보다는 실무적인 표현이다.

---

# 4. 내부 원리 / 작동 방식

## 4.1 Stack garbage

다음 코드를 보자.

```cpp
#include <iostream>

void f()
{
    int x;
    std::cout << x << std::endl;
}

int main()
{
    f();
    f();
    f();
    return 0;
}
```

`x`는 stack에 생긴다.

```text
f() 호출

stack frame
+----------------+
| x              |  <-- 초기화 안 됨
+----------------+
| return address |
+----------------+
```

stack memory는 이전 함수 호출에서 사용된 흔적을 가지고 있을 수 있다.

예를 들어 이전에 이런 값이 stack에 있었다고 하자.

```text
이전 함수 호출에서 stack에 남아 있던 값:
0x0000002A
```

그 영역을 다시 `int x`가 사용하면, `x`를 읽었을 때 우연히 `42`처럼 보일 수 있다.

하지만 중요한 점:

```text
그 값은 x의 값이 아니다.
그냥 x가 차지한 메모리 공간에 남아 있던 흔적이다.
```

---

## 4.2 Heap garbage

heap도 비슷하다.

```cpp
#include <iostream>

int main()
{
    int* p = new int;
    std::cout << *p << std::endl;
    delete p;
    return 0;
}
```

`new int;`는 int 객체를 만든다.
하지만 값 초기화는 하지 않는다.

```cpp
int* p = new int;    // 초기화 안 됨
int* q = new int();  // 0으로 초기화됨
```

차이:

| 코드            | 의미                                   |
| ------------- | ------------------------------------ |
| `new int`     | default-initialization, 기본 타입은 값 미정  |
| `new int()`   | value-initialization, `int`는 0으로 초기화 |
| `new int(42)` | 42로 초기화                              |

예:

```cpp
int* a = new int;
int* b = new int();
int* c = new int(42);
```

```text
a ---> heap int, 값 미정
b ---> heap int, 값 0
c ---> heap int, 값 42
```

---

## 4.3 배열 초기화

C++98 기준에서 배열도 주의해야 한다.

```cpp
int arr[5];
```

지역 배열이면 각 원소는 초기화되지 않는다.

```cpp
int arr[5] = {};
```

이렇게 하면 0으로 초기화된다.

동적 배열도 마찬가지다.

```cpp
int* a = new int[5];    // 각 원소 값 미정
int* b = new int[5]();  // 각 원소 0 초기화
```

주의:

```cpp
char buffer[1024];
```

이 buffer를 문자열처럼 쓰려면 반드시 null terminator와 초기화 상태를 신경 써야 한다.

---

# 5. 쉬운 예시

## 5.1 초기화되지 않은 조건문

```cpp
#include <iostream>

int main()
{
    bool enabled;

    if (enabled)
        std::cout << "feature on\n";
    else
        std::cout << "feature off\n";

    return 0;
}
```

문제:

```text
enabled가 true인지 false인지 정해지지 않았다.
그런데 if 조건으로 사용했다.
```

이 코드는 실행할 때마다 결과가 달라질 수 있다.

더 정확히는:

```text
초기화되지 않은 bool 값을 읽었기 때문에 undefined behavior다.
```

수정:

```cpp
bool enabled = false;
```

---

## 5.2 초기화되지 않은 counter

```cpp
#include <iostream>

int main()
{
    int count;

    for (int i = 0; i < 10; ++i)
        count += i;

    std::cout << count << std::endl;
    return 0;
}
```

초보자는 `count`가 0에서 시작한다고 생각할 수 있다.

하지만 아니다.

```text
count의 시작값은 정해져 있지 않다.
```

수정:

```cpp
int count = 0;
```

---

## 5.3 초기화되지 않은 struct field

```cpp
#include <iostream>

struct Packet
{
    int id;
    bool valid;
};

int main()
{
    Packet p;

    p.id = 10;

    if (p.valid)
        std::cout << "valid packet\n";

    return 0;
}
```

문제:

```text
p.id는 초기화됨
p.valid는 초기화 안 됨
```

`struct` 객체를 만들었다고 해서 모든 field가 자동으로 0이 되는 것은 아니다.

수정:

```cpp
Packet p;
p.id = 10;
p.valid = false;
```

또는 constructor 사용:

```cpp
struct Packet
{
    int id;
    bool valid;

    Packet() : id(0), valid(false) {}
};
```

C++에서는 이 방식이 좋다.

---

## 5.4 초기화되지 않은 pointer

```cpp
#include <iostream>

int main()
{
    int* p;

    if (p)
        std::cout << *p << std::endl;

    return 0;
}
```

여기서 `p`는 `NULL`이 아니다.
`p`는 초기화되지 않은 pointer다.

```text
p가 어떤 주소값을 가지고 있는지 모른다.
```

심지어 `if (p)` 자체도 초기화되지 않은 값을 읽는 것이다.

수정:

```cpp
int* p = NULL;
```

C++98에서는 `nullptr`가 없으므로 `NULL`을 사용한다.

---

# 6. 실무 예시

이번에는 ft_irc가 아니라, **파일 파싱 프로그램**을 예로 들자.

## 상황: CSV 파일에서 숫자 합계를 계산하는 프로그램

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>

int main()
{
    std::ifstream file("data.csv");
    std::string line;
    int total;

    while (std::getline(file, line))
    {
        std::stringstream ss(line);
        int value;

        ss >> value;
        total += value;
    }

    std::cout << "total = " << total << std::endl;
    return 0;
}
```

문제는 두 군데다.

```text
1. total이 초기화되지 않았다.
2. ss >> value가 실패했을 때 value가 유효하지 않을 수 있다.
```

예를 들어 `data.csv`가 다음과 같다고 하자.

```text
10
20
abc
30
```

`abc` 줄에서 `ss >> value`는 실패한다.
그런데 코드는 실패 여부를 확인하지 않고 `total += value`를 수행한다.

---

## 6.1 Symptom

```text
프로그램이 어떤 파일에서는 정상처럼 보인다.
어떤 파일에서는 이상한 total을 출력한다.
debug print를 넣으면 값이 달라진다.
```

---

## 6.2 Repro

입력 파일을 고정한다.

```text
data.csv:
10
20
abc
30
```

그리고 매번 같은 명령으로 실행한다.

```bash
c++ -Wall -Wextra -g -O0 sum.cpp -o sum
./sum
```

Valgrind로 실행한다.

```bash
valgrind --track-origins=yes ./sum
```

---

## 6.3 Hypothesis

```text
가설 1. total이 초기화되지 않았다.
가설 2. value가 parsing 실패 후 사용된다.
가설 3. file open 실패를 확인하지 않는다.
```

---

## 6.4 Evidence

Valgrind가 다음과 비슷한 report를 낼 수 있다.

```text
Conditional jump or move depends on uninitialised value(s)
Use of uninitialised value of size 8
```

또는 출력 과정에서 uninitialized value가 사용되었다고 보고할 수 있다.

코드 증거:

```cpp
int total;
```

초기화 없음.

```cpp
int value;
ss >> value;
total += value;
```

`ss >> value` 실패 여부 확인 없음.

---

## 6.5 Root Cause

```text
total의 초기값이 정의되지 않았다.
또한 parsing 실패 시 value가 유효한 값인지 확인하지 않고 사용했다.
```

---

## 6.6 Fix

```cpp
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>

int main()
{
    std::ifstream file("data.csv");
    if (!file)
    {
        std::cerr << "failed to open data.csv\n";
        return 1;
    }

    std::string line;
    int total = 0;

    while (std::getline(file, line))
    {
        std::stringstream ss(line);
        int value = 0;

        if (!(ss >> value))
        {
            std::cerr << "invalid line: " << line << std::endl;
            continue;
        }

        total += value;
    }

    std::cout << "total = " << total << std::endl;
    return 0;
}
```

---

## 6.7 Prevention

```text
1. 모든 지역 변수는 선언과 동시에 초기화한다.
2. parsing 결과를 반드시 확인한다.
3. 실패한 입력을 test case에 포함한다.
4. Valgrind --track-origins=yes를 사용한다.
5. -Wall -Wextra 경고를 무시하지 않는다.
```

---

# 7. 도구 사용 예시

## 7.1 컴파일러 경고

```cpp
int main()
{
    int x;
    return x;
}
```

빌드:

```bash
c++ -Wall -Wextra -Werror -g -O0 main.cpp -o main
```

컴파일러가 경고할 수 있다.

```text
warning: variable 'x' is uninitialized when used here
```

하지만 컴파일러가 항상 잡는 것은 아니다.

예:

```cpp
int maybe(bool flag)
{
    int x;

    if (flag)
        x = 10;

    return x;
}
```

이런 경우 컴파일러가 잡을 수도 있고 못 잡을 수도 있다.

---

## 7.2 Valgrind로 uninitialized read 잡기

예제:

```cpp
#include <iostream>

int main()
{
    int x;

    if (x > 0)
        std::cout << "positive\n";

    return 0;
}
```

빌드:

```bash
c++ -Wall -Wextra -g -O0 uninit.cpp -o uninit
```

실행:

```bash
valgrind --track-origins=yes ./uninit
```

가능한 report:

```text
Conditional jump or move depends on uninitialised value(s)
```

해석:

```text
조건문 if (x > 0)이 초기화되지 않은 값 x에 의존했다.
```

`--track-origins=yes`는 매우 중요하다.

```text
문제가 드러난 위치:
    if (x > 0)

값이 처음 생긴 위치:
    int x;
```

이 둘은 다를 수 있다.

---

## 7.3 AddressSanitizer는 이 문제에 약하다

ASan은 주로 주소 접근 오류를 잡는다.

```text
ASan이 잘 잡는 것:
    buffer overflow
    use-after-free
    double free

ASan이 약한 것:
    uninitialized value read
```

uninitialized value를 전문적으로 잡는 sanitizer는 **MemorySanitizer**다.

하지만 MemorySanitizer는 사용 조건이 까다롭다.

```text
MemorySanitizer:
    clang 기반
    Linux에서 주로 사용
    모든 라이브러리도 instrumented 되어야 정확도 높음
    macOS/일반 42 환경에서는 사용이 불편할 수 있음
```

따라서 일반적으로는:

```text
uninitialized memory 의심:
    Valgrind --track-origins=yes 우선
```

---

# 8. macOS / Linux 차이

## 8.1 Linux

Linux에서는 Valgrind가 매우 강하다.

```bash
valgrind --track-origins=yes ./program
```

특히 uninitialized value 문제에 좋다.

추천 흐름:

```text
1. -g -O0로 빌드
2. Valgrind --track-origins=yes 실행
3. report의 "depends on uninitialised value" 확인
4. origin 위치 추적
5. 변수 초기화 또는 실패 path 수정
```

---

## 8.2 macOS

macOS에서는 Valgrind가 최신 환경에서 불안정할 수 있다.

대안:

```text
1. clang 경고 최대한 활용
2. lldb로 변수 상태 확인
3. AddressSanitizer는 주소 오류용으로 사용
4. 가능하면 Linux 환경에서 Valgrind 재검증
```

macOS에서 일단 할 수 있는 것:

```bash
clang++ -Wall -Wextra -Wuninitialized -g -O0 main.cpp -o main
```

단, `-Wuninitialized`는 optimization 정보에 영향을 받기도 한다.
일부 경우에는 `-O1` 이상에서 더 잘 잡히는 경고도 있다.

---

## 8.3 42 환경

42 과제에서는 보통 다음 전략이 좋다.

```text
개발 중 macOS:
    -Wall -Wextra -Werror
    lldb
    ASan for address bugs

검증용 Linux:
    Valgrind --track-origins=yes
    Valgrind --leak-check=full
```

uninitialized memory는 ASan보다 Valgrind가 더 실용적이다.

---

# 9. Heisenbug

## 9.1 Heisenbug란 무엇인가

Heisenbug는 다음과 같은 버그다.

```text
관찰하려고 하면 증상이 바뀐다.
debugger를 붙이면 사라진다.
print를 넣으면 사라진다.
-O0에서는 괜찮고 -O2에서만 터진다.
실행할 때마다 결과가 다르다.
```

이런 버그의 원인은 여러 가지다.

```text
1. uninitialized memory
2. use-after-free
3. buffer overflow
4. data race
5. timing bug
6. stack layout 변화
7. undefined behavior
```

즉, Heisenbug는 원인 이름이 아니라 **증상 패턴**이다.

---

## 9.2 왜 print를 넣으면 버그가 사라지는가

예를 들어 stack overflow 또는 uninitialized read가 있다고 하자.

```cpp
int main()
{
    int x;
    if (x == 12345)
        crash();
}
```

여기에 print를 넣는다.

```cpp
int main()
{
    int x;
    std::cout << "debug\n";

    if (x == 12345)
        crash();
}
```

`std::cout` 호출이 들어가면 stack layout, register 사용, 함수 호출 순서, timing이 바뀔 수 있다.

```text
print 추가 전 stack:
+---------+
| x       |
+---------+
| old val |
+---------+

print 추가 후 stack:
+---------+
| temp    |
+---------+
| x       |
+---------+
| saved   |
+---------+
```

초기화되지 않은 `x`가 읽는 메모리 흔적이 달라질 수 있다.
그래서 증상이 바뀐다.

---

## 9.3 왜 debugger를 붙이면 버그가 사라지는가

debugger를 붙이면 실행 환경이 바뀐다.

```text
1. 실행 속도가 느려진다.
2. memory layout이 달라질 수 있다.
3. signal 처리 방식이 달라질 수 있다.
4. optimization이 꺼진 binary를 쓰는 경우가 많다.
5. breakpoints 때문에 timing이 바뀐다.
```

특히 race condition은 debugger 때문에 잘 사라진다.

그러나 uninitialized memory도 debugger 환경에서 양상이 달라질 수 있다.

---

## 9.4 왜 optimization이 버그 양상을 바꾸는가

C/C++ 컴파일러는 undefined behavior가 없다고 가정하고 최적화한다.

예를 들어:

```cpp
int f(int* p)
{
    int x = *p;

    if (p == NULL)
        return 0;

    return x;
}
```

이 코드는 이미 첫 줄에서 `*p`를 했다.

```cpp
int x = *p;
```

만약 `p == NULL`이면 첫 줄에서 이미 UB다.

그래서 컴파일러는 이렇게 생각할 수 있다.

```text
*p를 했다는 것은 p가 NULL이 아니라는 뜻이다.
그러면 if (p == NULL)은 불필요하다.
```

최적화 후에는 null check가 사라질 수 있다.

이런 식으로 UB는 컴파일러 최적화와 결합하면 매우 위험해진다.

---

# 10. Uninitialized Memory와 Race Condition의 차이

둘 다 Heisenbug처럼 보일 수 있다.

하지만 원인은 다르다.

| 항목              | Uninitialized Memory               | Race Condition                       |
| --------------- | ---------------------------------- | ------------------------------------ |
| 원인              | 값 초기화 없이 읽음                        | 여러 thread/process가 같은 자원에 동시 접근      |
| 재현성             | stack/heap layout에 민감              | timing/scheduling에 민감                |
| 도구              | Valgrind Memcheck, MemorySanitizer | ThreadSanitizer, helgrind            |
| 단일 thread에서도 발생 | 가능                                 | 일반적으로 동시성 필요                         |
| 대표 증상           | 이상한 값, 분기 오류                       | 가끔 깨지는 상태, deadlock, data corruption |

예:

## Uninitialized memory

```cpp
int x;

if (x > 0)
    doSomething();
```

## Race condition

```cpp
// thread A
counter++;

// thread B
counter++;
```

`counter++`는 실제로는 다음 단계다.

```text
1. counter 읽기
2. 1 더하기
3. counter에 쓰기
```

두 thread가 동시에 하면 update가 사라질 수 있다.

---

# 11. Deterministic Repro 만들기

Heisenbug를 잡으려면 재현성을 높여야 한다.

## 11.1 입력 고정

파일 입력이면 test file을 고정한다.

```text
bad_input.txt
```

랜덤이면 seed를 고정한다.

```cpp
srand(42);
```

---

## 11.2 실행 명령 고정

```bash
./program bad_input.txt
```

항상 같은 command로 실행한다.

---

## 11.3 빌드 옵션 고정

```bash
c++ -Wall -Wextra -g -O0 main.cpp -o program
```

또는 sanitizer용:

```bash
c++ -Wall -Wextra -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer main.cpp -o program
```

빌드 옵션이 바뀌면 증상도 바뀔 수 있다.

---

## 11.4 최소 재현 코드로 줄이기

큰 프로젝트에서 바로 잡으려 하지 말고, 의심 코드를 작은 파일로 분리한다.

```text
large project
    ↓
specific function
    ↓
small input
    ↓
single file repro
```

예:

```cpp
int parseNumber(const std::string& s)
{
    std::stringstream ss(s);
    int value;
    ss >> value;
    return value;
}
```

문제는 `ss >> value` 실패 시 `value`를 반환하는 것이다.

수정:

```cpp
bool parseNumber(const std::string& s, int& out)
{
    std::stringstream ss(s);
    return static_cast<bool>(ss >> out);
}
```

C++98에서는 `explicit operator bool` 같은 최신 문법을 기대하지 말고, stream 상태를 명확히 검사하는 습관을 들이는 것이 좋다.

---

# 12. 실무 Debugging Flow

예제는 **config parser**로 들겠다.

## 문제 코드

```cpp
#include <iostream>
#include <sstream>
#include <string>

struct Config
{
    int port;
    bool verbose;
};

Config parseConfig(const std::string& line)
{
    Config cfg;

    std::stringstream ss(line);
    ss >> cfg.port;

    if (line.find("verbose") != std::string::npos)
        cfg.verbose = true;

    return cfg;
}

int main()
{
    Config cfg = parseConfig("8080");

    if (cfg.verbose)
        std::cout << "verbose mode\n";

    std::cout << "port = " << cfg.port << std::endl;
    return 0;
}
```

문제:

```text
cfg.verbose가 항상 초기화되지 않는다.
```

`line`에 `"verbose"`가 없으면 `cfg.verbose`는 미정 상태다.

---

## 12.1 Symptom

```text
어떤 실행에서는 verbose mode가 출력된다.
어떤 실행에서는 출력되지 않는다.
코드를 조금 바꾸면 증상이 바뀐다.
```

---

## 12.2 Repro

입력을 고정한다.

```cpp
Config cfg = parseConfig("8080");
```

Valgrind로 실행한다.

```bash
c++ -Wall -Wextra -g -O0 config.cpp -o config
valgrind --track-origins=yes ./config
```

---

## 12.3 Hypothesis

```text
Config::verbose가 초기화되지 않은 상태로 if 조건에 사용된다.
```

---

## 12.4 Evidence

문제 코드:

```cpp
Config cfg;
```

이 코드는 `port`, `verbose`를 자동으로 0/false로 만들지 않는다.

```cpp
if (line.find("verbose") != std::string::npos)
    cfg.verbose = true;
```

verbose가 없는 경우 `cfg.verbose`는 초기화되지 않는다.

Valgrind report:

```text
Conditional jump or move depends on uninitialised value(s)
```

---

## 12.5 Root Cause

```text
Config 객체 생성 시 field 기본값을 설정하지 않았다.
특정 입력 경로에서 verbose field가 초기화되지 않은 채 반환되었다.
```

---

## 12.6 Fix

C++98에서 constructor를 둔다.

```cpp
struct Config
{
    int port;
    bool verbose;

    Config() : port(0), verbose(false) {}
};
```

그리고 parsing 실패도 검사한다.

```cpp
Config parseConfig(const std::string& line)
{
    Config cfg;

    std::stringstream ss(line);
    if (!(ss >> cfg.port))
    {
        cfg.port = -1;
        return cfg;
    }

    if (line.find("verbose") != std::string::npos)
        cfg.verbose = true;

    return cfg;
}
```

---

## 12.7 Prevention

```text
1. struct/class field는 constructor에서 모두 초기화한다.
2. parsing 함수는 실패 여부를 반환하게 설계한다.
3. bool, pointer, counter는 선언 즉시 초기화한다.
4. Valgrind --track-origins=yes를 test workflow에 넣는다.
```

---

# 13. 흔한 오해

## 오해 1. “지역 변수는 자동으로 0이다”

아니다.

```cpp
void f()
{
    int x;      // 0 아님
    bool b;     // false 아님
    int* p;     // NULL 아님
}
```

지역 자동 변수는 기본적으로 초기화되지 않는다.

단, 전역/static 변수는 0 초기화된다.

```cpp
int g; // 0으로 초기화됨

void f()
{
    static int s; // 0으로 초기화됨
}
```

---

## 오해 2. “초기화 안 된 값은 그냥 랜덤 값이다”

부정확하다.

초기화 안 된 값은 “랜덤 값”이라기보다 **정의되지 않은 값**이다.

더 중요한 것은:

```text
그 값을 읽는 행위가 undefined behavior가 될 수 있다.
```

따라서 “이번에는 37이 나왔다”가 중요한 게 아니다.
그 값을 읽은 시점부터 프로그램을 신뢰하기 어려워진다.

---

## 오해 3. “debugger에서 정상이라면 문제없다”

아니다.

debugger는 실행 환경을 바꾼다.

```text
debugger에서 정상
    ≠
프로그램이 정상
```

Heisenbug는 특히 debugger에서 사라질 수 있다.

---

## 오해 4. “ASan이 통과했으니 uninitialized 문제는 없다”

아니다.

ASan은 주로 주소 오류를 잡는다.
uninitialized value는 Valgrind Memcheck나 MemorySanitizer 쪽이 더 적합하다.

---

## 오해 5. “struct를 만들면 field가 알아서 초기화된다”

아니다.

```cpp
struct User
{
    int id;
    bool active;
};

User u;
```

`u.id`, `u.active`는 자동으로 0/false가 아니다.

C++98에서는 constructor를 명시하라.

```cpp
struct User
{
    int id;
    bool active;

    User() : id(0), active(false) {}
};
```

---

# 14. 확인 문제

## 문제 1

```cpp
int main()
{
    int x;

    if (x > 0)
        return 1;

    return 0;
}
```

질문:

```text
1. x의 주소는 존재하는가?
2. x의 값은 유효한가?
3. 문제는 주소 접근 오류인가, 값 초기화 문제인가?
4. Valgrind에서는 어떤 종류의 report가 나올 수 있는가?
```

---

## 문제 2

```cpp
struct Packet
{
    int id;
    bool valid;
};

int main()
{
    Packet p;
    p.id = 3;

    if (p.valid)
        return 1;

    return 0;
}
```

질문:

```text
1. 어떤 field가 초기화되지 않았는가?
2. 왜 struct 객체 생성만으로 충분하지 않은가?
3. C++98에서 가장 좋은 수정은 무엇인가?
```

---

## 문제 3

```cpp
int* p = new int;
std::cout << *p << std::endl;
delete p;
```

질문:

```text
1. p 자체는 초기화되었는가?
2. *p는 초기화되었는가?
3. 올바른 수정 코드는?
```

---

## 문제 4

```cpp
int* p;
if (p)
    delete p;
```

질문:

```text
1. p는 NULL인가?
2. if (p)는 안전한가?
3. 어떻게 초기화해야 하는가?
```

---

# 15. 실습 과제

## 실습 1. Local variable uninitialized read

파일:

```cpp
#include <iostream>

int main()
{
    int x;

    if (x > 0)
        std::cout << "positive\n";

    return 0;
}
```

빌드:

```bash
c++ -Wall -Wextra -g -O0 uninit1.cpp -o uninit1
```

실행:

```bash
valgrind --track-origins=yes ./uninit1
```

확인:

```text
1. Conditional jump report가 나오는가?
2. origin은 어디로 나오는가?
3. x = 0으로 수정하면 report가 사라지는가?
```

---

## 실습 2. Struct field 초기화

파일:

```cpp
#include <iostream>

struct Config
{
    int port;
    bool verbose;
};

int main()
{
    Config cfg;
    cfg.port = 8080;

    if (cfg.verbose)
        std::cout << "verbose\n";

    return 0;
}
```

수정 전 Valgrind:

```bash
c++ -Wall -Wextra -g -O0 config.cpp -o config
valgrind --track-origins=yes ./config
```

수정:

```cpp
struct Config
{
    int port;
    bool verbose;

    Config() : port(0), verbose(false) {}
};
```

확인:

```text
1. 수정 전 report가 나오는가?
2. 수정 후 report가 사라지는가?
3. constructor initializer list가 왜 좋은가?
```

---

## 실습 3. Parsing failure

파일:

```cpp
#include <iostream>
#include <sstream>
#include <string>

int parse(const std::string& s)
{
    std::stringstream ss(s);
    int value;
    ss >> value;
    return value;
}

int main()
{
    std::cout << parse("123") << std::endl;
    std::cout << parse("abc") << std::endl;
    return 0;
}
```

문제:

```text
parse("abc")에서 value가 초기화되지 않은 채 반환될 수 있다.
```

수정:

```cpp
bool parse(const std::string& s, int& out)
{
    std::stringstream ss(s);
    return static_cast<bool>(ss >> out);
}
```

사용:

```cpp
int value = 0;

if (parse("abc", value))
    std::cout << value << std::endl;
else
    std::cerr << "invalid number\n";
```

확인:

```text
1. 실패를 값으로 표현하지 말고 bool로 표현하는 이유는?
2. out parameter는 언제 유효한가?
3. value를 호출 전에 초기화하는 이유는?
```

---

# 16. 핵심 정리

## 1. 초기화되지 않은 메모리는 “그냥 랜덤 값”이 아니다

```text
읽는 순간 undefined behavior가 될 수 있다.
```

---

## 2. 주소가 유효한 것과 값이 유효한 것은 다르다

```text
int x;

&x:
    주소는 있음

x:
    값은 초기화되지 않았음
```

---

## 3. Stack과 heap 모두 초기화 문제가 생긴다

```cpp
int x;             // stack uninitialized
int* p = new int;  // heap int uninitialized
```

초기화:

```cpp
int x = 0;
int* p = new int();
```

---

## 4. Struct/class field는 constructor에서 초기화한다

C++98 기준:

```cpp
struct Config
{
    int port;
    bool verbose;

    Config() : port(0), verbose(false) {}
};
```

---

## 5. Heisenbug는 원인이 아니라 증상 패턴이다

대표 원인:

```text
uninitialized memory
use-after-free
buffer overflow
race condition
undefined behavior
```

---

## 6. 도구 선택

| 문제                  | 적합한 도구                         |
| ------------------- | ------------------------------ |
| uninitialized read  | Valgrind `--track-origins=yes` |
| heap-use-after-free | AddressSanitizer               |
| buffer overflow     | AddressSanitizer               |
| leak                | Valgrind / LeakSanitizer       |
| UB 일반               | UBSan                          |
| race condition      | ThreadSanitizer                |

---

## 7. 디버깅 흐름

uninitialized memory도 같은 흐름으로 본다.

```text
symptom
    ↓
repro
    ↓
hypothesis
    ↓
evidence
    ↓
root cause
    ↓
fix
    ↓
prevention
```

예:

```text
symptom:
    결과가 실행마다 다름

repro:
    입력과 빌드 옵션 고정

hypothesis:
    특정 변수가 초기화되지 않은 채 조건문에 사용됨

evidence:
    Valgrind "Conditional jump depends on uninitialised value"

root cause:
    struct field verbose가 일부 path에서 초기화되지 않음

fix:
    constructor에서 모든 field 초기화

prevention:
    선언 즉시 초기화, parsing 실패 검사, Valgrind test 추가
```

---
