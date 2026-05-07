# Valgrind / Sanitizers 개념

이번 강의의 핵심은 이것이다.

> **메모리 디버깅 도구는 마법이 아니다.**
> 도구는 프로그램 실행을 관찰하면서 “정상적인 메모리 접근 규칙”을 위반하는 순간을 잡아낸다.

Valgrind와 Sanitizer는 둘 다 메모리 버그를 찾는 도구지만, 작동 방식이 상당히 다르다.

---

# 1. 핵심 질문

이번 Lecture 12에서 답할 질문은 다음이다.

```text
1. Valgrind는 프로그램을 어떻게 감시하는가?
2. AddressSanitizer는 왜 빠른가?
3. Valgrind와 Sanitizer는 무엇이 다른가?
4. debug symbol은 왜 중요한가?
5. -fno-omit-frame-pointer는 왜 넣는가?
6. LeakSanitizer와 Valgrind leak report는 어떻게 다른가?
7. macOS와 Linux에서 어떤 도구를 써야 하는가?
8. 42 과제에서는 Makefile에 어떻게 안전하게 넣어야 하는가?
```

---

# 2. 개념 설명

C/C++ 메모리 디버깅 도구는 크게 두 계열로 나눌 수 있다.

| 계열         | 대표 도구                                  | 핵심 방식                    |
| ---------- | -------------------------------------- | ------------------------ |
| 동적 실행 감시   | Valgrind Memcheck                      | 프로그램을 가상 CPU 위에서 실행하며 감시 |
| 컴파일러 삽입 감시 | AddressSanitizer, UBSan, LeakSanitizer | 컴파일 시 검사 코드를 프로그램 안에 삽입  |

쉽게 말하면:

```text
Valgrind:
    "내가 네 프로그램을 대신 실행하면서 모든 메모리 접근을 감시하겠다."

Sanitizer:
    "컴파일할 때 네 코드 곳곳에 감시 장치를 심어두겠다."
```

---

# 3. 왜 필요한지

C/C++ 메모리 버그는 다음 특징을 가진다.

```text
1. 항상 crash하지 않는다.
2. crash 위치가 root cause 위치와 다를 수 있다.
3. debug build에서는 멀쩡하고 release build에서 터질 수 있다.
4. Linux에서는 터지고 macOS에서는 안 터질 수 있다.
5. 작은 입력에서는 괜찮고 큰 입력에서만 터질 수 있다.
```

예를 들어:

```cpp
int* p = new int(42);
delete p;

std::cout << *p << std::endl;
```

이 코드는 명백한 use-after-free다.
하지만 항상 segfault가 나는 것은 아니다.

왜냐하면 `delete p` 이후에도 그 주소가 당장 OS에 의해 접근 금지되는 것은 아닐 수 있기 때문이다.

```text
delete 전:
p ----> [ int 42 ]

delete 후:
p ----> [ allocator에게 반환된 heap block ]

문제:
그 block이 아직 물리적으로 남아 있을 수도 있음.
그래서 읽으면 "우연히" 42가 보일 수도 있음.
```

이런 상황에서 도구가 필요하다.

도구는 다음 정보를 제공한다.

```text
1. 어떤 주소를 잘못 읽거나 썼는가?
2. 그 주소는 heap인가 stack인가?
3. 언제 할당되었는가?
4. 언제 해제되었는가?
5. 어느 함수 호출 경로에서 문제가 발생했는가?
6. leak이라면 누가 allocation했고 누가 delete하지 않았는가?
```

---

# 4. 내부 원리 / 작동 방식

## 4.1 Valgrind Memcheck의 기본 원리

Valgrind는 프로그램을 직접 CPU에서 그대로 실행시키지 않는다.

대략적으로는 다음처럼 작동한다.

```text
원래 실행:
    내 프로그램 ----> CPU

Valgrind 실행:
    내 프로그램 ----> Valgrind 가상 실행 엔진 ----> CPU
```

Valgrind는 프로그램의 기계어 명령을 내부 표현으로 변환한 뒤, 각 메모리 접근을 감시한다.

```text
프로그램이 하는 일:
    "주소 0x1234에서 4바이트 읽기"

Valgrind가 확인하는 것:
    1. 이 주소는 접근 가능한가?
    2. 이 메모리는 초기화되었는가?
    3. 이 메모리는 이미 free된 영역인가?
    4. 이 접근이 할당 범위를 넘어서는가?
```

Valgrind Memcheck는 크게 두 가지 정보를 추적한다.

```text
A-bit: Addressability
    이 메모리 주소에 접근해도 되는가?

V-bit: Validity
    이 메모리 값은 초기화된 값인가?
```

간단히 말하면:

| 추적 정보          | 의미                   |
| -------------- | -------------------- |
| Addressability | 이 주소가 유효한 메모리 영역인가   |
| Validity       | 이 메모리 안의 값이 초기화된 값인가 |

예를 들어:

```cpp
int x;
std::cout << x << std::endl;
```

`x`는 stack에 있으므로 주소 자체는 접근 가능하다.

하지만 값은 초기화되지 않았다.

```text
Addressability: OK
Validity: BAD
```

그래서 Valgrind는 이런 식으로 보고할 수 있다.

```text
Conditional jump or move depends on uninitialised value(s)
```

---

## 4.2 AddressSanitizer의 기본 원리

AddressSanitizer, 줄여서 ASan은 컴파일러가 코드에 검사 로직을 삽입한다.

```text
원래 코드:
    arr[i] = 10;

ASan 빌드:
    if (arr+i 주소가 위험한지 검사)
        report error
    arr[i] = 10;
```

ASan의 핵심 아이디어는 **shadow memory**다.

ASan은 프로그램 메모리와 별도로 “이 주소가 안전한지”를 기록하는 shadow memory를 둔다.

```text
Application memory:
+---------+---------+---------+---------+
| byte 0  | byte 1  | byte 2  | byte 3  |
+---------+---------+---------+---------+

Shadow memory:
+-----------------------------+
| 이 구간은 접근 가능 / 불가능 |
+-----------------------------+
```

ASan은 heap allocation 주변에 **redzone**을 만든다.

```cpp
int* p = new int[3];
```

실제 배치는 개념적으로 다음과 비슷하다.

```text
heap
+-----------+----------------+-----------+
| redzone   | int[3]         | redzone   |
+-----------+----------------+-----------+
            ^
            p
```

`p[0]`, `p[1]`, `p[2]`는 정상이다.

하지만:

```cpp
p[3] = 10;
```

이것은 오른쪽 redzone을 침범한다.

```text
heap
+-----------+----------------+-----------+
| redzone   | int[3]         | redzone   |
+-----------+----------------+-----------+
                             ^
                             p[3] 접근
```

ASan은 이 redzone 접근을 감지한다.

---

## 4.3 Use-after-free를 ASan이 잡는 방식

다음 코드가 있다고 하자.

```cpp
int* p = new int(42);
delete p;

std::cout << *p << std::endl;
```

ASan은 `delete p` 이후 해당 heap block을 즉시 완전히 재사용하지 않고, 일정 시간 **quarantine** 영역에 둔다.

개념적으로는 다음과 같다.

```text
new int(42):
+----------------+
| valid int      |
+----------------+

delete p 이후:
+----------------+
| poisoned block |
+----------------+
```

`poisoned`라는 말은 “이 주소에 접근하면 오류로 보고하겠다”는 뜻이다.

그래서 `*p`를 읽는 순간 ASan이 보고한다.

```text
ERROR: AddressSanitizer: heap-use-after-free
```

---

## 4.4 UndefinedBehaviorSanitizer의 기본 원리

UBSan은 메모리 접근만 보는 것이 아니다.
C/C++의 undefined behavior가 될 수 있는 연산을 검사한다.

예:

```cpp
int x = 2147483647;
x = x + 1; // signed integer overflow
```

C++에서 signed integer overflow는 undefined behavior다.

UBSan을 켜면 이런 문제를 보고할 수 있다.

```bash
c++ -g -O1 -fsanitize=undefined main.cpp -o main
```

대표적으로 UBSan이 잡는 것:

```text
1. signed integer overflow
2. null pointer dereference
3. misaligned pointer access
4. out-of-bounds array access 일부
5. invalid enum value
6. invalid downcast 일부
```

단, UBSan은 모든 UB를 잡지 못한다.

---

## 4.5 LeakSanitizer의 기본 원리

LeakSanitizer, 줄여서 LSan은 프로그램 종료 시점에 heap allocation을 검사한다.

개념적으로 다음 질문을 던진다.

> **아직 free/delete되지 않은 heap block 중에서, 더 이상 도달할 수 없는 것은 무엇인가?**

예:

```cpp
int main()
{
    int* p = new int(42);
    return 0;
}
```

프로그램 종료 시 `p`는 stack에서 사라진다.
하지만 heap block은 delete되지 않았다.

```text
stack:
p 사라짐

heap:
+---------+
| int 42  |  <-- 아직 delete 안 됨
+---------+
```

LSan은 이를 leak으로 보고한다.

---

# 5. 쉬운 예시

## 5.1 Heap buffer overflow

```cpp
#include <iostream>

int main()
{
    int* arr = new int[3];

    arr[0] = 1;
    arr[1] = 2;
    arr[2] = 3;
    arr[3] = 4; // overflow

    delete[] arr;
    return 0;
}
```

ASan 빌드:

```bash
c++ -Wall -Wextra -g -O1 -fsanitize=address -fno-omit-frame-pointer overflow.cpp -o overflow
./overflow
```

예상 report 핵심:

```text
ERROR: AddressSanitizer: heap-buffer-overflow
WRITE of size 4
```

읽는 법:

```text
heap-buffer-overflow
    heap에 할당된 배열 범위 밖에 접근했다.

WRITE of size 4
    4바이트 int 값을 썼다.

allocated by thread T0 here
    이 배열이 어디서 new 되었는지 보여준다.
```

---

## 5.2 Stack buffer overflow

```cpp
int main()
{
    int arr[3];
    arr[3] = 10;
    return 0;
}
```

ASan report:

```text
ERROR: AddressSanitizer: stack-buffer-overflow
```

heap과 stack을 구분해야 한다.

| Report                 | 의미                                 |
| ---------------------- | ---------------------------------- |
| heap-buffer-overflow   | `new`, `malloc`으로 할당된 heap 범위 밖 접근 |
| stack-buffer-overflow  | 지역 변수 배열 범위 밖 접근                   |
| global-buffer-overflow | 전역/static 배열 범위 밖 접근               |

---

## 5.3 Use-after-free

```cpp
#include <iostream>

int main()
{
    int* p = new int(42);
    delete p;

    std::cout << *p << std::endl;
    return 0;
}
```

ASan report 핵심:

```text
ERROR: AddressSanitizer: heap-use-after-free
READ of size 4
```

읽는 순서:

```text
1. 에러 종류:
   heap-use-after-free

2. 접근 종류:
   READ of size 4

3. 잘못 접근한 위치:
   main.cpp:8

4. free된 위치:
   main.cpp:6

5. allocation된 위치:
   main.cpp:5
```

이 세 위치를 연결해야 한다.

```text
allocated here
      ↓
freed here
      ↓
used here
```

---

## 5.4 Double free

```cpp
int main()
{
    int* p = new int(42);
    delete p;
    delete p;
    return 0;
}
```

ASan report:

```text
ERROR: AddressSanitizer: attempting double-free
```

이 경우 핵심은:

```text
1. 두 번째 delete 위치
2. 첫 번째 delete 위치
3. 원래 allocation 위치
```

Double free는 allocator의 내부 free list를 망가뜨릴 수 있다.

---

## 5.5 Uninitialized memory

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

Valgrind 실행:

```bash
c++ -Wall -Wextra -g -O0 uninit.cpp -o uninit
valgrind --track-origins=yes ./uninit
```

가능한 report:

```text
Conditional jump or move depends on uninitialised value(s)
```

의미:

```text
조건문 판단에 초기화되지 않은 값이 사용되었다.
```

`--track-origins=yes`를 넣으면 그 uninitialized value가 어디서 생겼는지 더 추적해준다.

---

# 6. 실무 예시

## 상황: ft_irc에서 disconnect 후 broadcast crash

가정:

```cpp
class Channel
{
private:
    std::vector<User*> _users; // non-owning
};
```

```cpp
class Server
{
private:
    std::map<int, User*> _users; // owning
};
```

disconnect 처리:

```cpp
void Server::disconnect(int fd)
{
    delete _users[fd];
    _users.erase(fd);
}
```

문제:

Channel의 `_users` vector에는 여전히 같은 User pointer가 남아 있다.

이후:

```cpp
void Channel::broadcast(const std::string& msg)
{
    for (size_t i = 0; i < _users.size(); ++i)
        _users[i]->send(msg);
}
```

ASan report가 다음처럼 나올 수 있다.

```text
ERROR: AddressSanitizer: heap-use-after-free
READ of size 8

#0 User::send(std::string const&)
#1 Channel::broadcast(std::string const&)
#2 Server::handlePrivmsg(...)
#3 Server::run()

freed by thread T0 here:
#0 operator delete
#1 Server::disconnect(int)

previously allocated by thread T0 here:
#0 operator new
#1 Server::acceptClient()
```

이 report를 디버깅 흐름으로 읽자.

---

## 6.1 Symptom

```text
client A disconnect 후
client B가 메시지를 보내면
서버가 segfault 또는 ASan error 발생
```

---

## 6.2 Repro

재현 절차를 고정한다.

```text
1. 서버 실행
2. A 접속
3. B 접속
4. A, B 모두 #lobby join
5. A 종료
6. B가 PRIVMSG #lobby :hello
7. crash 또는 ASan report
```

---

## 6.3 Hypothesis

```text
가설 1. Channel에 dangling User*가 남아 있다.
가설 2. poll fd 배열에는 fd가 남아 있는데 User는 제거되었다.
가설 3. map iterator를 erase 후 계속 사용했다.
가설 4. output buffer가 삭제된 User 내부 string을 참조한다.
```

---

## 6.4 Evidence

ASan report의 세 위치를 본다.

```text
allocated:
    Server::acceptClient()

freed:
    Server::disconnect()

used:
    Channel::broadcast()
```

이 세 위치가 말하는 사실:

```text
User 객체는 Server::disconnect에서 이미 죽었다.
하지만 Channel::broadcast가 그 객체를 계속 사용했다.
```

---

## 6.5 Root Cause

ownership과 observer 관계가 불명확했다.

```text
Server owns User.
Channel observes User.

그런데 Server가 User를 delete할 때,
Channel의 observer pointer를 제거하지 않았다.
```

---

## 6.6 Fix

disconnect 시 모든 Channel에서 User pointer를 제거한다.

```cpp
void Server::disconnect(int fd)
{
    User* user = _users[fd];

    for (std::map<std::string, Channel*>::iterator it = _channels.begin();
         it != _channels.end();
         ++it)
    {
        it->second->removeUser(user);
    }

    delete user;
    _users.erase(fd);
    close(fd);
}
```

Channel:

```cpp
void Channel::removeUser(User* user)
{
    for (std::vector<User*>::iterator it = _users.begin();
         it != _users.end(); )
    {
        if (*it == user)
            it = _users.erase(it);
        else
            ++it;
    }
}
```

---

## 6.7 Prevention

```text
1. owning container와 non-owning container를 구분한다.
2. delete 전에 모든 observer에서 제거한다.
3. disconnect path를 하나의 함수로 통일한다.
4. fd close, poll removal, User removal, Channel removal 순서를 문서화한다.
5. ASan target으로 disconnect 시나리오를 반복 테스트한다.
```

---

# 7. 도구 사용 예시

## 7.1 기본 디버그 빌드

```bash
c++ -Wall -Wextra -Werror -g -O0 main.cpp -o main
```

| 옵션                      | 의미              |
| ----------------------- | --------------- |
| `-g`                    | debug symbol 포함 |
| `-O0`                   | 최적화 끔           |
| `-Wall -Wextra -Werror` | 경고를 엄격히 처리      |

---

## 7.2 AddressSanitizer

```bash
c++ -Wall -Wextra -g -O1 \
    -fsanitize=address \
    -fno-omit-frame-pointer \
    main.cpp -o main
```

실행:

```bash
./main
```

추천 조합:

```bash
-fsanitize=address -fno-omit-frame-pointer -g -O1
```

왜 `-O1`인가?

```text
-O0:
    디버깅은 쉽지만 sanitizer가 잡는 양상이 약간 다를 수 있다.

-O1:
    sanitizer와 stack trace 품질 사이에서 실무적으로 자주 쓰는 균형점.

-O2 이상:
    최적화로 인해 stack trace가 복잡해질 수 있다.
```

---

## 7.3 AddressSanitizer + UndefinedBehaviorSanitizer

```bash
c++ -Wall -Wextra -g -O1 \
    -fsanitize=address,undefined \
    -fno-omit-frame-pointer \
    main.cpp -o main
```

이 조합은 매우 유용하다.

```text
ASan:
    heap-use-after-free
    heap-buffer-overflow
    stack-buffer-overflow
    double-free
    invalid free 일부

UBSan:
    signed integer overflow
    null pointer dereference
    misaligned access
    invalid casts 일부
```

---

## 7.4 LeakSanitizer

Linux에서는 ASan과 함께 leak detection이 켜지는 경우가 많다.

```bash
c++ -Wall -Wextra -g -O1 \
    -fsanitize=address \
    -fno-omit-frame-pointer \
    leak.cpp -o leak

./leak
```

필요하면 환경변수:

```bash
ASAN_OPTIONS=detect_leaks=1 ./leak
```

macOS에서는 LeakSanitizer 지원이 제한적이거나 동작이 다를 수 있다.

---

## 7.5 Valgrind Memcheck

Linux:

```bash
valgrind ./main
```

더 자세히:

```bash
valgrind \
    --leak-check=full \
    --show-leak-kinds=all \
    --track-origins=yes \
    ./main
```

각 옵션 의미:

| 옵션                                 | 의미                                                   |
| ---------------------------------- | ---------------------------------------------------- |
| `--leak-check=full`                | leak의 allocation stack trace를 자세히 보여줌                |
| `--show-leak-kinds=all`            | definitely/indirectly/possibly/still reachable 모두 표시 |
| `--track-origins=yes`              | uninitialized value의 기원을 추적                          |
| `--errors-for-leak-kinds=definite` | 특정 leak을 error로 취급 가능                                |
| `--error-exitcode=1`               | CI나 script에서 실패 처리 가능                                |

---

# 8. Valgrind vs Sanitizer 차이

## 8.1 큰 차이

| 항목                   | Valgrind Memcheck              | Sanitizers                      |
| -------------------- | ------------------------------ | ------------------------------- |
| 방식                   | 실행 중 동적 binary instrumentation | 컴파일 시 검사 코드 삽입                  |
| 재컴파일 필요              | 보통 필요 없음                       | 필요                              |
| 속도                   | 매우 느림                          | 비교적 빠름                          |
| uninitialized memory | 강함                             | MemorySanitizer 필요, 일반 ASan은 약함 |
| use-after-free       | 잘 잡음                           | 매우 잘 잡음                         |
| buffer overflow      | 잘 잡음                           | 매우 잘 잡음                         |
| leak report          | 자세함                            | 빠르고 실용적                         |
| macOS                | 최신 환경에서 제한적                    | clang ASan 사용 가능                |
| Linux                | 매우 강력                          | 매우 강력                           |
| 42 환경                | Linux면 매우 유용                   | Makefile target으로 추가 추천         |

---

## 8.2 속도 차이

대략적인 감각:

```text
일반 실행:
    1x

ASan:
    약 2x 정도 느려질 수 있음

Valgrind:
    약 10x ~ 50x 이상 느려질 수 있음
```

정확한 수치는 프로그램에 따라 다르다.

서버 프로그램에서는 Valgrind 실행 시 타이밍이 크게 바뀔 수 있다.
그래서 network server에서는 ASan이 더 실용적인 경우가 많다.

---

## 8.3 어떤 상황에 무엇을 먼저 쓸까?

| 상황                      | 우선 도구                          |
| ----------------------- | ------------------------------ |
| use-after-free 의심       | ASan                           |
| buffer overflow 의심      | ASan                           |
| double free 의심          | ASan                           |
| memory leak 정밀 분류       | Valgrind                       |
| uninitialized read 의심   | Valgrind `--track-origins=yes` |
| signed overflow / UB 의심 | UBSan                          |
| Linux에서 과제 평가 전 leak 확인 | Valgrind                       |
| macOS 개발 중 빠른 확인        | ASan                           |
| 오래 도는 서버 시나리오           | ASan 우선, Valgrind는 짧은 repro로   |

---

# 9. Runtime Overhead

## 9.1 Valgrind overhead

Valgrind는 프로그램 실행을 거의 “해석”하듯이 감시한다.

```text
장점:
    매우 상세한 메모리 추적 가능

단점:
    느림
    timing이 바뀜
    network program에서 timeout이나 race 양상이 달라질 수 있음
```

---

## 9.2 Sanitizer overhead

Sanitizer는 컴파일된 프로그램 안에 검사 코드가 들어간다.

```text
장점:
    빠름
    실제 실행 환경과 비교적 비슷함
    큰 테스트에도 적용 가능

단점:
    재컴파일 필요
    모든 버그를 잡지는 못함
    sanitizer가 켜진 binary는 제출용 binary와 다름
```

---

# 10. False Positive / False Negative

## 10.1 False Positive

False positive는 실제 버그가 아닌데 도구가 버그처럼 보고하는 것이다.

메모리 도구에서는 생각보다 흔하지 않지만, 가능하다.

예:

```text
1. custom allocator
2. assembly 코드
3. system library와 sanitizer runtime 충돌
4. intentionally unusual memory layout
5. third-party library 내부 문제
```

42 과제의 일반 C/C++ 코드에서는 false positive보다 실제 버그일 가능성이 높다.

---

## 10.2 False Negative

False negative는 실제 버그가 있는데 도구가 못 잡는 것이다.

예:

```cpp
int* p = new int[3];
p[3] = 10;
```

ASan은 대부분 잡는다.

하지만 다음처럼 범위 안의 잘못된 logical access는 못 잡을 수 있다.

```cpp
int arr[10];

int logical_size = 3;
arr[5] = 10; // 물리적으로는 arr 안, 논리적으로는 잘못
```

도구 입장에서는 `arr[5]`가 배열 범위 안이다.

```text
물리적 메모리 관점:
    OK

프로그램 논리 관점:
    BUG
```

이런 버그는 도구가 자동으로 잡기 어렵다.

---

# 11. Debug Symbol과 Stack Trace

## 11.1 debug symbol이 없으면?

`-g` 없이 빌드하면 report가 이렇게 나올 수 있다.

```text
#0 0x00000040123a in ??
#1 0x0000004011f0 in ??
```

이러면 어디서 문제가 났는지 보기 어렵다.

---

## 11.2 debug symbol이 있으면?

`-g`를 넣으면:

```text
#0 User::send(std::string const&) User.cpp:42
#1 Channel::broadcast(...) Channel.cpp:87
#2 Server::handlePrivmsg(...) Server.cpp:210
#3 Server::run() Server.cpp:340
#4 main main.cpp:18
```

이제 다음을 알 수 있다.

```text
1. 함수 이름
2. 파일 이름
3. 줄 번호
4. 호출 경로
```

메모리 디버깅에서 stack trace는 “범죄 현장 지도”다.

---

## 11.3 `-fno-omit-frame-pointer`

컴파일러는 최적화할 때 frame pointer를 생략할 수 있다.

그러면 stack trace가 부정확하거나 짧아질 수 있다.

```bash
-fno-omit-frame-pointer
```

이 옵션은 frame pointer를 유지하게 해서 sanitizer report의 stack trace 품질을 높인다.

간단히 말하면:

```text
-g:
    소스 파일/줄 번호 정보를 넣는다.

-fno-omit-frame-pointer:
    함수 호출 체인을 더 잘 복원하게 한다.
```

둘은 역할이 다르다.

---

# 12. Sanitizer Report 읽는 법

다음 report를 보자.

```text
ERROR: AddressSanitizer: heap-use-after-free on address 0x602000000010
READ of size 4 at 0x602000000010 thread T0
    #0 0x4012ab in main uaf.cpp:8

0x602000000010 is located 0 bytes inside of 4-byte region
freed by thread T0 here:
    #0 0x7f... in operator delete
    #1 0x40127c in main uaf.cpp:6

previously allocated by thread T0 here:
    #0 0x7f... in operator new
    #1 0x40125a in main uaf.cpp:5
```

읽는 순서:

## 1단계: 에러 종류

```text
heap-use-after-free
```

이미 free된 heap memory를 사용했다.

---

## 2단계: 읽기인가 쓰기인가

```text
READ of size 4
```

4바이트를 읽었다.
`int`일 가능성이 높다.

---

## 3단계: 잘못 접근한 위치

```text
main uaf.cpp:8
```

여기가 symptom 위치다.

---

## 4단계: free 위치

```text
main uaf.cpp:6
```

여기서 object lifetime이 끝났다.

---

## 5단계: allocation 위치

```text
main uaf.cpp:5
```

여기서 object lifetime이 시작됐다.

---

## 6단계: root cause 문장으로 변환

```text
uaf.cpp:5에서 new된 int가
uaf.cpp:6에서 delete되었고,
uaf.cpp:8에서 다시 읽혔다.
따라서 root cause는 delete 이후 pointer 사용이다.
```

---

# 13. Valgrind Report 읽는 법

예제:

```cpp
#include <iostream>

int main()
{
    int* p = new int(42);
    return 0;
}
```

실행:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./leak
```

Report 예시:

```text
HEAP SUMMARY:
    in use at exit: 4 bytes in 1 blocks
  total heap usage: 1 allocs, 0 frees, 4 bytes allocated

4 bytes in 1 blocks are definitely lost in loss record 1 of 1
    at 0x...: operator new(unsigned long)
    by 0x...: main (leak.cpp:5)

LEAK SUMMARY:
   definitely lost: 4 bytes in 1 blocks
   indirectly lost: 0 bytes in 0 blocks
   possibly lost: 0 bytes in 0 blocks
   still reachable: 0 bytes in 0 blocks
```

핵심:

```text
total heap usage:
    allocation과 free 총량

in use at exit:
    프로그램 종료 시 아직 해제되지 않은 heap

definitely lost:
    접근 가능한 pointer가 없어서 명백한 leak

main leak.cpp:5:
    allocation site
```

---

# 14. macOS / Linux 차이

## 14.1 Linux

Linux에서는 다음 조합이 가장 실용적이다.

```text
1. ASan + UBSan
2. Valgrind Memcheck
3. gdb
```

추천 흐름:

```text
빠른 테스트:
    ASan + UBSan

leak 정밀 확인:
    Valgrind --leak-check=full

초기화 문제:
    Valgrind --track-origins=yes

stack trace 분석:
    gdb
```

---

## 14.2 macOS

macOS에서는 일반적으로:

```text
1. clang ASan
2. lldb
3. leaks
4. Instruments
```

Valgrind는 최신 macOS에서 설치나 동작이 까다로운 경우가 많다.

macOS에서 sanitizer 빌드:

```bash
clang++ -Wall -Wextra -g -O1 \
    -fsanitize=address,undefined \
    -fno-omit-frame-pointer \
    main.cpp -o main
```

실행:

```bash
./main
```

LeakSanitizer는 macOS에서 Linux만큼 안정적이지 않을 수 있다.

---

## 14.3 42 환경

42에서는 보통 Linux 기반 평가 또는 Linux-like 환경을 기준으로 leak을 본다.

권장:

```text
개발 중:
    macOS라면 ASan/lldb 사용

제출 전:
    Linux 환경에서 Valgrind 확인

Makefile:
    기본 빌드는 subject 조건 준수
    sanitizer는 별도 target으로 분리
```

---

# 15. 42 과제용 Makefile 예시

C++98 기준 프로젝트라고 하자.

```makefile
NAME = ircserv

CXX = c++
CXXFLAGS = -Wall -Wextra -Werror -std=c++98

SRCS = main.cpp Server.cpp User.cpp Channel.cpp
OBJS = $(SRCS:.cpp=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(CXX) $(CXXFLAGS) $(OBJS) -o $(NAME)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

debug: CXXFLAGS += -g -O0
debug: re

asan: CXXFLAGS += -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer
asan: re

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME)

re: fclean all

.PHONY: all clean fclean re debug asan
```

하지만 이 Makefile에는 한 가지 문제가 있다.

`asan: re`는 `re`를 타고 다시 `all`로 가면서 flags 전달이 잘 되는 구조는 Make 버전에 따라 기대와 다르게 느껴질 수 있다.
더 명시적으로는 sanitizer 전용 binary를 따로 만드는 방식이 좋다.

---

## 15.1 더 안전한 sanitizer target

```makefile
NAME = ircserv
ASAN_NAME = ircserv_asan

CXX = c++
CXXFLAGS = -Wall -Wextra -Werror -std=c++98
DEBUGFLAGS = -g -O0
ASANFLAGS = -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer

SRCS = main.cpp Server.cpp User.cpp Channel.cpp
OBJS = $(SRCS:.cpp=.o)

all: $(NAME)

$(NAME): $(OBJS)
	$(CXX) $(CXXFLAGS) $(OBJS) -o $(NAME)

asan:
	$(CXX) $(CXXFLAGS) $(ASANFLAGS) $(SRCS) -o $(ASAN_NAME)

debug:
	$(CXX) $(CXXFLAGS) $(DEBUGFLAGS) $(SRCS) -o $(NAME)

clean:
	rm -f $(OBJS)

fclean: clean
	rm -f $(NAME) $(ASAN_NAME)

re: fclean all

.PHONY: all clean fclean re debug asan
```

사용:

```bash
make asan
./ircserv_asan 6667 pass
```

이 방식의 장점:

```text
1. 제출용 binary와 sanitizer binary를 분리한다.
2. 기본 NAME은 subject 조건을 유지한다.
3. ASan 테스트 후 실수로 sanitizer binary를 제출할 가능성이 줄어든다.
```

---

# 16. 디버깅 흐름 적용 예시

## 문제 코드

```cpp
#include <iostream>

class User
{
public:
    int fd;

    User(int f) : fd(f) {}
    void sendMsg()
    {
        std::cout << "send to " << fd << std::endl;
    }
};

int main()
{
    User* u = new User(3);
    User* channelUser = u;

    delete u;

    channelUser->sendMsg();
    return 0;
}
```

---

## 16.1 Symptom

```text
프로그램이 segfault가 날 수도 있고,
그냥 "send to 3"이 출력될 수도 있다.
```

---

## 16.2 Repro

ASan으로 빌드한다.

```bash
c++ -Wall -Wextra -g -O1 \
    -fsanitize=address \
    -fno-omit-frame-pointer \
    uaf_user.cpp -o uaf_user

./uaf_user
```

---

## 16.3 Hypothesis

```text
channelUser가 이미 delete된 User 객체를 가리킨다.
```

---

## 16.4 Evidence

ASan report:

```text
ERROR: AddressSanitizer: heap-use-after-free

READ of size 4
    #0 User::sendMsg() uaf_user.cpp:10
    #1 main uaf_user.cpp:21

freed by thread T0 here:
    #0 operator delete
    #1 main uaf_user.cpp:19

previously allocated by thread T0 here:
    #0 operator new
    #1 main uaf_user.cpp:16
```

해석:

```text
User는 line 16에서 new됨.
line 19에서 delete됨.
line 21에서 channelUser를 통해 다시 사용됨.
```

---

## 16.5 Root Cause

```text
User 객체의 owner는 main의 u였지만,
channelUser라는 non-owning alias가 남아 있었다.
delete 이후 alias를 정리하지 않았다.
```

---

## 16.6 Fix

잘못된 순서를 고친다.

```cpp
int main()
{
    User* u = new User(3);
    User* channelUser = u;

    channelUser = NULL;
    delete u;

    return 0;
}
```

하지만 이것은 toy example에서만 충분하다.

실무적으로는 container에서 제거해야 한다.

```cpp
channel.removeUser(u);
delete u;
```

---

## 16.7 Prevention

```text
1. owner를 하나로 정한다.
2. non-owning pointer는 delete 전에 제거한다.
3. raw pointer alias를 줄인다.
4. ASan으로 disconnect/reconnect/broadcast 시나리오를 테스트한다.
```

---

# 17. 흔한 오해

## 오해 1. “Valgrind만 돌리면 모든 메모리 버그를 잡는다”

아니다.

Valgrind는 강력하지만 모든 버그를 잡지 못한다.

예:

```cpp
int arr[10];
int logical_size = 3;

arr[5] = 100;
```

메모리 관점에서는 `arr[5]`가 유효하다.
하지만 프로그램 논리상 `logical_size`를 넘어섰다.

이런 것은 Valgrind도 ASan도 보통 잡지 못한다.

---

## 오해 2. “ASan이 통과하면 메모리 문제는 없다”

아니다.

ASan은 매우 강력하지만 한계가 있다.

```text
1. uninitialized read는 일반 ASan이 잘 못 잡는다.
2. 논리적 범위 오류는 못 잡을 수 있다.
3. race condition은 ThreadSanitizer 영역이다.
4. 모든 UB를 잡지는 못한다.
5. sanitizer build와 release build는 다르다.
```

---

## 오해 3. “Valgrind와 ASan은 동시에 쓰면 더 좋다”

보통 동시에 쓰지 않는다.

```text
Valgrind ./asan_binary
```

이런 방식은 sanitizer runtime과 Valgrind가 충돌하거나 report가 이상해질 수 있다.

권장:

```text
1. 일반 debug binary를 Valgrind로 실행
2. sanitizer binary는 그냥 직접 실행
```

---

## 오해 4. “-g는 프로그램을 느리게 만든다”

`-g`는 debug symbol을 추가한다.
일반적으로 실행 성능에 직접 큰 영향을 주지 않는다.

성능에 더 큰 영향을 주는 것은:

```text
-O0
-fsanitize=address
Valgrind 실행
```

---

## 오해 5. “-O0가 항상 디버깅에 가장 좋다”

초기 gdb 디버깅에는 `-O0`가 좋다.

하지만 sanitizer는 `-O1`이 더 실용적인 경우가 많다.

```text
gdb로 변수 하나하나 보기:
    -g -O0

ASan으로 memory bug 찾기:
    -g -O1 -fsanitize=address -fno-omit-frame-pointer
```

---

# 18. 확인 문제

## 문제 1

다음 report가 있다.

```text
ERROR: AddressSanitizer: heap-buffer-overflow
WRITE of size 4
```

질문:

```text
1. heap에서 어떤 일이 벌어진 것인가?
2. READ와 WRITE 중 어느 쪽인가?
3. size 4는 어떤 타입일 가능성이 큰가?
```

---

## 문제 2

다음 report가 있다.

```text
ERROR: AddressSanitizer: heap-use-after-free

freed by thread T0 here:
    Server::disconnect(int)

previously allocated by thread T0 here:
    Server::acceptClient()
```

질문:

```text
1. 객체의 lifetime은 어디서 시작되었는가?
2. 객체의 lifetime은 어디서 끝났는가?
3. root cause를 한 문장으로 쓰면?
```

---

## 문제 3

다음 Valgrind report가 있다.

```text
Conditional jump or move depends on uninitialised value(s)
```

질문:

```text
1. 주소 접근 자체가 잘못된 것인가?
2. 값의 어떤 상태가 문제인가?
3. 어떤 옵션을 추가하면 origin 추적이 쉬워지는가?
```

---

## 문제 4

다음 두 명령의 차이를 설명해라.

```bash
c++ -g -O0 main.cpp -o main
```

```bash
c++ -g -O1 -fsanitize=address -fno-omit-frame-pointer main.cpp -o main
```

질문:

```text
1. 첫 번째는 어떤 상황에 좋은가?
2. 두 번째는 어떤 상황에 좋은가?
3. -fno-omit-frame-pointer는 왜 들어가는가?
```

---

# 19. 실습 과제

## 실습 1. ASan으로 heap-use-after-free 잡기

파일:

```cpp
#include <iostream>

int main()
{
    int* p = new int(42);
    delete p;

    std::cout << *p << std::endl;
    return 0;
}
```

빌드:

```bash
c++ -Wall -Wextra -g -O1 \
    -fsanitize=address \
    -fno-omit-frame-pointer \
    uaf.cpp -o uaf
```

실행:

```bash
./uaf
```

확인:

```text
1. 에러 종류가 무엇인가?
2. invalid access line은 어디인가?
3. freed by line은 어디인가?
4. allocated by line은 어디인가?
```

---

## 실습 2. Valgrind로 leak 잡기

파일:

```cpp
int main()
{
    int* p = new int(42);
    return 0;
}
```

빌드:

```bash
c++ -Wall -Wextra -g -O0 leak.cpp -o leak
```

실행:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./leak
```

확인:

```text
1. definitely lost가 나오는가?
2. 몇 bytes가 lost인가?
3. allocation site는 몇 번째 줄인가?
4. delete p를 추가하면 report가 어떻게 바뀌는가?
```

---

## 실습 3. Uninitialized value 잡기

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
c++ -Wall -Wextra -g -O0 uninit.cpp -o uninit
```

실행:

```bash
valgrind --track-origins=yes ./uninit
```

확인:

```text
1. 어떤 종류의 warning이 나오는가?
2. x의 주소는 접근 가능한가?
3. 문제는 주소인가, 값인가?
```

---

## 실습 4. ft_irc에 sanitizer target 추가하기

Makefile에 다음 target을 추가해라.

```makefile
ASAN_NAME = ircserv_asan
ASANFLAGS = -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer

asan:
	$(CXX) $(CXXFLAGS) $(ASANFLAGS) $(SRCS) -o $(ASAN_NAME)
```

실행:

```bash
make asan
./ircserv_asan 6667 pass
```

테스트 시나리오:

```text
1. client A 접속
2. client B 접속
3. 둘 다 같은 channel join
4. A disconnect
5. B가 channel에 message 전송
6. ASan report 확인
```

관찰:

```text
1. heap-use-after-free가 발생하는가?
2. poll fd 제거 문제인가?
3. User 객체 lifetime 문제인가?
4. Channel에 dangling pointer가 남는가?
```

---

# 20. 핵심 정리

## 1. Valgrind와 Sanitizer는 작동 방식이 다르다

```text
Valgrind:
    프로그램을 감시 환경에서 실행한다.

Sanitizer:
    컴파일 시 검사 코드를 삽입한다.
```

---

## 2. ASan은 메모리 접근 오류에 강하다

잘 잡는 것:

```text
heap-use-after-free
heap-buffer-overflow
stack-buffer-overflow
double-free
invalid free 일부
```

상대적으로 약한 것:

```text
uninitialized memory
논리적 범위 오류
race condition
모든 종류의 UB
```

---

## 3. Valgrind는 uninitialized memory와 leak 분석에 강하다

특히:

```bash
valgrind --leak-check=full --show-leak-kinds=all --track-origins=yes ./program
```

이 조합은 Linux에서 매우 강력하다.

---

## 4. Report는 세 위치를 연결해서 읽는다

ASan report는 보통 다음 세 위치가 핵심이다.

```text
1. invalid access site
2. free/delete site
3. allocation site
```

이것을 문장으로 바꾸면 root cause가 보인다.

```text
어디서 할당되었고,
어디서 해제되었고,
어디서 다시 사용되었는가?
```

---

## 5. 42 과제에서는 기본 빌드와 디버그 빌드를 분리한다

```text
제출용:
    subject 요구 flags 유지

디버깅용:
    -g -O0

sanitizer용:
    -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer
```

---

## 6. 메모리 디버깅 흐름은 항상 같다

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

도구는 이 흐름에서 **evidence**를 제공한다.
도구가 root cause를 자동으로 이해해주는 것은 아니다.

---

# 다음 Lecture 예고

다음은 **Lecture 13. Leak Debugging**이다.

핵심은 다음이다.

```text
memory leak은 단순히 delete를 까먹은 문제가 아니다.
진짜 핵심은 ownership graph가 끊어진 것이다.
```

다음 강의에서는 Valgrind leak report의:

```text
definitely lost
indirectly lost
possibly lost
still reachable
```

를 하나씩 해부하고, C++98 환경에서 RAII와 destructor로 leak을 줄이는 방식을 다룬다.
