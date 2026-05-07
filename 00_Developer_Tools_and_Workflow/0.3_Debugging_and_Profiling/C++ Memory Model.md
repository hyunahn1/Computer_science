# C/C++ Memory Model 기초

## 1. 핵심 질문

이번 강의의 핵심 질문은 이것이다.

> **C/C++ 프로그램에서 “메모리 문제”는 왜 발생하는가?**

조금 더 구체적으로 말하면 다음 질문들이다.

1. 변수는 메모리 어디에 저장되는가?
2. stack과 heap은 무엇이 다른가?
3. pointer는 무엇을 “소유”하는가?
4. object lifetime은 언제 시작되고 언제 끝나는가?
5. `delete` 이후 pointer를 쓰면 왜 위험한가?
6. memory leak과 dangling pointer는 어떻게 다른가?
7. C++에서 RAII가 왜 중요한가?
8. undefined behavior는 왜 디버깅을 어렵게 만드는가?

---

# 2. 개념 설명

C/C++ 프로그램은 실행될 때 하나의 **process**가 된다.

process는 운영체제로부터 독립된 가상 메모리 공간을 받는다.
그 안에는 대략 다음 영역들이 있다.

```text
높은 주소
+-----------------------------+
| stack                       |
| 함수 호출, 지역 변수        |
| 아래 방향으로 자라는 경우 많음 |
+-----------------------------+
|                             |
| unused / mmap area          |
|                             |
+-----------------------------+
| heap                        |
| new / malloc 동적 할당      |
| 위 방향으로 자라는 경우 많음 |
+-----------------------------+
| global/static memory        |
| 전역 변수, static 변수      |
+-----------------------------+
| text segment                |
| 실행 코드, 함수 기계어      |
+-----------------------------+
낮은 주소
```

정확한 배치는 OS, ABI, 컴파일러, 보안 옵션에 따라 다르다.
하지만 개념적으로는 위 구조로 이해하면 된다.

---

# 3. 왜 필요한지

메모리 디버깅은 단순히 “segfault가 났다”를 보는 것이 아니다.

진짜 질문은 다음이다.

> **이 주소는 어느 메모리 영역의 주소인가?**
> **그 객체는 아직 살아 있는가?**
> **누가 이 메모리를 해제해야 하는가?**
> **이 pointer는 유효한 객체를 가리키는가?**

예를 들어 다음 두 코드는 둘 다 위험하지만 성격이 다르다.

```cpp
int* p = new int(42);
delete p;
std::cout << *p << std::endl; // use-after-free
```

```cpp
int* p = new int(42);
// delete p 없음
```

첫 번째는 **죽은 객체를 사용**한다.
두 번째는 **살아 있던 heap memory를 잃어버린다**.

즉,

| 문제                   | 본질                                |
| -------------------- | --------------------------------- |
| use-after-free       | lifetime이 끝난 객체를 사용               |
| double free          | 이미 끝난 lifetime을 다시 끝내려 함          |
| memory leak          | 더 이상 접근할 수 없는 heap allocation이 남음 |
| buffer overflow      | 할당된 범위 밖을 침범                      |
| uninitialized memory | 값이 정해지지 않은 메모리를 읽음                |
| dangling pointer     | 이미 죽은 객체를 가리키는 pointer            |
| undefined behavior   | C/C++ 표준이 결과를 보장하지 않음             |

---

# 4. 내부 원리 / 작동 방식

## 4.1 Text Segment

text segment에는 실행 코드가 들어간다.

```cpp
void hello()
{
    std::cout << "hello\n";
}
```

컴파일 후 `hello()` 함수의 기계어 명령어들은 text segment 쪽에 놓인다.

일반적으로 이 영역은 읽기/실행 가능하고, 쓰기는 제한된다.

```text
text segment
+----------------------+
| hello 함수의 기계어  |
| main 함수의 기계어   |
| std::cout 호출 코드  |
+----------------------+
```

그래서 함수 pointer는 “코드 위치”를 가리키는 pointer다.

---

## 4.2 Global / Static Memory

전역 변수와 static 변수는 프로그램 시작부터 종료까지 살아 있다.

```cpp
int g_count = 0;

void f()
{
    static int local_static = 0;
    local_static++;
}
```

`g_count`와 `local_static`은 stack에 생기는 것이 아니다.
이들은 프로그램 전체 lifetime을 가진다.

```text
global/static memory
+----------------------+
| g_count              |
| local_static         |
+----------------------+
```

중요한 점:

> `static` 지역 변수는 scope는 함수 안이지만, lifetime은 프로그램 전체다.

즉, **scope와 lifetime은 다르다.**

---

## 4.3 Stack

stack은 함수 호출과 지역 변수를 관리한다.

```cpp
void foo()
{
    int x = 10;
    int y = 20;
}
```

`foo()`가 호출되면 stack frame이 생긴다.

```text
main stack frame
+----------------------+
| main의 지역 변수     |
+----------------------+

foo 호출 후

foo stack frame
+----------------------+
| x = 10               |
| y = 20               |
| return address       |
| saved registers      |
+----------------------+
main stack frame
+----------------------+
| main의 지역 변수     |
+----------------------+
```

`foo()`가 끝나면 `foo`의 stack frame은 사라진다.

그래서 다음 코드는 위험하다.

```cpp
int* bad()
{
    int x = 42;
    return &x; // 매우 위험
}
```

`x`는 `bad()` 함수가 끝나면 lifetime이 종료된다.

```text
bad() 실행 중

bad stack frame
+----------------------+
| x = 42               |  <-- &x 반환
+----------------------+

bad() 종료 후

bad stack frame 사라짐
반환된 pointer는 죽은 stack 영역을 가리킴
```

이런 pointer를 **dangling pointer**라고 한다.

---

## 4.4 Heap

heap은 동적으로 메모리를 할당하는 영역이다.

C++:

```cpp
int* p = new int(42);
delete p;
```

C:

```c
int* p = (int*)malloc(sizeof(int));
free(p);
```

heap allocation은 함수가 끝나도 자동으로 사라지지 않는다.

```cpp
int* make_number()
{
    int* p = new int(42);
    return p;
}
```

이 코드는 stack 변수 `p`는 사라지지만, `p`가 가리키던 heap memory는 살아 있다.

```text
make_number stack frame
+----------------------+
| p -------------------|----+
+----------------------+    |
                            v
heap
+----------------------+
| int 42               |
+----------------------+
```

함수가 끝난 뒤:

```text
stack frame 사라짐

heap
+----------------------+
| int 42               |  <-- 여전히 살아 있음
+----------------------+
```

따라서 누군가는 반드시 `delete` 해야 한다.

```cpp
int* p = make_number();
delete p;
```

---

# 5. 쉬운 예시

## 예시 1. Stack 변수

```cpp
void f()
{
    int x = 10;
}
```

`x`는 `f()`가 호출될 때 생기고, `f()`가 끝나면 사라진다.

```text
lifetime:
f() 시작 -------- x 살아 있음 -------- f() 종료
```

---

## 예시 2. Heap 변수

```cpp
void f()
{
    int* p = new int(10);
}
```

여기서 `p`는 stack 변수다.
하지만 `new int(10)`으로 만든 `int`는 heap에 있다.

```text
stack
+----------------+
| p ------------ | ----+
+----------------+     |
                       v
heap
+----------------+
| int 10         |
+----------------+
```

`f()`가 끝나면 `p`는 사라진다.
하지만 heap의 `int 10`은 사라지지 않는다.

결과:

```text
memory leak
```

왜냐하면 heap allocation에 접근할 pointer를 잃어버렸기 때문이다.

---

## 예시 3. Use-after-free

```cpp
int* p = new int(10);
delete p;

std::cout << *p << std::endl; // 위험
```

`delete p` 이후 heap object의 lifetime은 끝났다.

중요한 점:

```cpp
delete p;
```

이 코드는 `p` 변수 자체를 없애는 것이 아니다.
`p`가 가리키는 heap object를 해제한다.

즉,

```text
delete 전

p ----> heap int 10

delete 후

p ----> 해제된 heap 영역
```

`p`에는 여전히 주소값이 남아 있을 수 있다.
하지만 그 주소는 더 이상 유효한 객체를 의미하지 않는다.

---

## 예시 4. Double free

```cpp
int* p = new int(10);
delete p;
delete p; // double free
```

첫 번째 `delete`로 object lifetime은 끝났다.
두 번째 `delete`는 이미 해제된 메모리를 다시 해제하려고 한다.

이것은 allocator 내부 구조를 망가뜨릴 수 있다.

---

## 예시 5. Buffer overflow

```cpp
int arr[3];

arr[0] = 1;
arr[1] = 2;
arr[2] = 3;
arr[3] = 4; // buffer overflow
```

`arr`의 유효 index는 `0, 1, 2`다.

```text
stack
+---------+
| arr[0]  |
+---------+
| arr[1]  |
+---------+
| arr[2]  |
+---------+
| 다른 값 |  <-- arr[3]이 이 영역을 침범할 수 있음
+---------+
```

C/C++은 기본적으로 배열 경계 검사를 자동으로 하지 않는다.

그래서 `arr[3]`은 컴파일은 될 수 있지만, 실행 결과는 보장되지 않는다.

---

# 6. 실무 예시

## 상황: ft_irc 서버에서 User 객체 관리

예를 들어 IRC 서버에서 client fd마다 `User` 객체를 heap에 만든다고 하자.

```cpp
User* user = new User(fd);
_users[fd] = user;
```

연결이 끊기면 다음과 같이 정리할 수 있다.

```cpp
delete _users[fd];
_users.erase(fd);
close(fd);
```

문제는 ownership이 불명확할 때 발생한다.

예를 들어 Channel도 같은 User pointer를 들고 있다.

```cpp
_users[fd] = user;
_channelUsers.push_back(user);
```

연결 종료 시:

```cpp
delete _users[fd];
_users.erase(fd);
```

하지만 channel 쪽에는 아직 pointer가 남아 있다.

```text
_users map
fd ----> User object

channel vector
[ User* ] ----> 같은 User object
```

삭제 후:

```text
_users map
fd 제거됨

channel vector
[ User* ] ----> 이미 delete된 User object
```

이후 channel broadcast에서:

```cpp
_channelUsers[i]->sendMessage(msg);
```

이러면 use-after-free가 된다.

---

## 이 문제를 디버깅 흐름으로 보면

### 1. Symptom

서버가 특정 클라이언트 disconnect 이후 broadcast 시 segfault 발생.

### 2. Repro

재현 절차를 고정한다.

```text
1. 서버 실행
2. client A 접속
3. client B 접속
4. 둘 다 #lobby 입장
5. client A 종료
6. client B가 메시지 전송
7. 서버 segfault
```

### 3. Hypothesis

가능한 가설:

```text
가설 1. fd가 닫혔는데 poll 배열에 남아 있다.
가설 2. User 객체가 delete됐는데 Channel에 pointer가 남아 있다.
가설 3. send buffer가 이미 해제된 객체를 참조한다.
가설 4. map erase 후 iterator를 계속 사용한다.
```

### 4. Evidence

gdb에서 stack trace 확인.

```bash
gdb ./ircserv
run 6667 pass
bt
```

예상 stack trace:

```text
#0 User::sendMessage(...)
#1 Channel::broadcast(...)
#2 Server::handlePrivmsg(...)
#3 Server::run()
#4 main()
```

그리고 `User*` 주소 확인.

```gdb
frame 1
print _channelUsers[i]
print *_channelUsers[i]
```

만약 접근 시 오류가 나거나 이상한 값이 보이면 dangling pointer 가능성이 있다.

### 5. Root Cause

`User` object의 실제 lifetime은 끝났지만, Channel이 그 pointer를 계속 보관했다.

### 6. Fix

연결 종료 시 모든 channel에서 해당 user를 제거한다.

```cpp
void Server::removeUser(int fd)
{
    User* user = _users[fd];

    for (size_t i = 0; i < _channels.size(); ++i)
        _channels[i].removeUser(user);

    delete user;
    _users.erase(fd);
}
```

또는 ownership을 명확히 한다.

```text
Server owns User
Channel does not own User
Channel only references User
```

### 7. Prevention

C++98에서는 `std::unique_ptr`가 없으므로 다음 규칙을 세운다.

```text
1. User를 delete할 권한은 Server만 가진다.
2. Channel은 User를 delete하지 않는다.
3. User 삭제 전 반드시 모든 Channel에서 제거한다.
4. raw pointer를 저장하는 container에는 "소유 여부"를 주석으로 명시한다.
5. destructor에서 owned resource를 정리한다.
```

---

# 7. 도구 사용 예시

Lecture 12에서 깊게 다루겠지만, 여기서는 기본만 본다.

## 7.1 컴파일 옵션

디버깅용으로는 보통 다음을 사용한다.

```bash
c++ -Wall -Wextra -Werror -g -O0 main.cpp -o main
```

의미:

| 옵션              | 의미              |
| --------------- | --------------- |
| `-g`            | debug symbol 포함 |
| `-O0`           | 최적화 끔           |
| `-Wall -Wextra` | 경고 많이 켬         |
| `-Werror`       | 경고를 에러로 처리      |

42 과제에서는 보통 `-Wall -Wextra -Werror`가 요구된다.
디버깅할 때는 `-g`를 추가하는 것이 좋다.

---

## 7.2 AddressSanitizer

Linux 또는 clang/gcc 환경:

```bash
c++ -g -O1 -fsanitize=address -fno-omit-frame-pointer main.cpp -o main
./main
```

use-after-free 예시:

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

ASan은 대략 이런 식으로 보고한다.

```text
ERROR: AddressSanitizer: heap-use-after-free
READ of size 4
    #0 main main.cpp:8

freed by thread T0 here:
    #0 operator delete
    #1 main main.cpp:6

previously allocated by thread T0 here:
    #0 operator new
    #1 main main.cpp:5
```

이 report에서 중요한 것은 세 군데다.

```text
1. 어디서 잘못 읽었는가?
2. 어디서 free/delete 되었는가?
3. 어디서 allocation 되었는가?
```

memory debugging은 이 세 점을 연결하는 작업이다.

```text
allocation site
      |
      v
lifetime begins
      |
      v
free/delete site
      |
      v
lifetime ends
      |
      v
invalid access site
```

---

## 7.3 Valgrind

Linux:

```bash
valgrind --leak-check=full --track-origins=yes ./main
```

주의:

* Valgrind는 Linux에서 강력하다.
* macOS 최신 버전에서는 지원이 불안정하거나 제한적일 수 있다.
* 42 Linux 환경에서는 Valgrind가 매우 유용하다.
* macOS에서는 Sanitizer 쪽이 더 현실적인 경우가 많다.

---

# 8. macOS / Linux 차이

## 8.1 Linux

Linux에서는 보통 다음 조합이 강력하다.

```text
gdb
valgrind
AddressSanitizer
UndefinedBehaviorSanitizer
LeakSanitizer
perf / strace
```

C/C++ memory debugging에서는 특히:

```bash
valgrind --leak-check=full --track-origins=yes ./program
```

그리고:

```bash
c++ -g -O1 -fsanitize=address,undefined -fno-omit-frame-pointer ...
```

이 조합이 실무적으로 강하다.

---

## 8.2 macOS

macOS에서는 일반적으로:

```text
lldb
clang sanitizer
leaks
Instruments
```

를 더 많이 쓴다.

예:

```bash
clang++ -g -O1 -fsanitize=address -fno-omit-frame-pointer main.cpp -o main
./main
```

macOS의 Valgrind는 버전과 환경에 따라 불안정할 수 있다.
따라서 macOS에서는 ASan을 먼저 쓰는 것이 현실적이다.

---

## 8.3 42 환경 기준

42 과제에서는 보통 다음을 구분해야 한다.

| 상황              | 추천                                 |
| --------------- | ---------------------------------- |
| 일반 제출용 빌드       | subject 요구 flags만 사용               |
| 로컬 디버깅          | `-g -O0` 추가                        |
| memory error 추적 | sanitizer build 따로 만들기             |
| leak 확인         | Linux라면 Valgrind                   |
| macOS에서 개발      | ASan/lldb 우선                       |
| 평가 전            | sanitizer flag 제거 후 clean build 확인 |

중요:

> sanitizer flag를 제출용 Makefile 기본 빌드에 항상 넣는 것은 위험할 수 있다.

따라서 Makefile에는 보통 별도 target을 둔다.

```makefile
debug:
	c++ $(CXXFLAGS) -g -O0 $(SRCS) -o $(NAME)

asan:
	c++ $(CXXFLAGS) -g -O1 -fsanitize=address -fno-omit-frame-pointer $(SRCS) -o $(NAME)
```

---

# 9. 흔한 오해

## 오해 1. “pointer를 delete하면 pointer가 NULL이 된다”

아니다.

```cpp
int* p = new int(42);
delete p;
```

`delete p`는 `p`가 가리키는 객체를 해제한다.
`p` 변수 자체의 값이 자동으로 `NULL`이 되지는 않는다.

그래서 방어적으로는 다음처럼 쓰기도 한다.

```cpp
delete p;
p = NULL;
```

다만 이것도 모든 문제를 해결하지 않는다.

왜냐하면 같은 객체를 가리키는 다른 pointer가 있을 수 있기 때문이다.

```cpp
int* p = new int(42);
int* q = p;

delete p;
p = NULL;

std::cout << *q << std::endl; // q는 여전히 dangling pointer
```

---

## 오해 2. “segfault가 안 나면 정상이다”

아니다.

C/C++에서는 잘못된 메모리 접근이 항상 즉시 crash를 내지 않는다.

```cpp
int arr[3];
arr[3] = 10;
```

이 코드가 어떤 환경에서는 조용히 실행될 수 있다.
하지만 이미 undefined behavior다.

---

## 오해 3. “memory leak은 프로그램이 종료되면 OS가 회수하니까 무시해도 된다”

부분적으로만 맞다.

프로그램 종료 시 OS가 process memory를 회수하는 것은 맞다.

하지만 서버처럼 오래 실행되는 프로그램에서는 leak이 누적된다.

IRC 서버 같은 프로그램에서는 특히 치명적이다.

```text
client 접속 → User allocation
client 종료 → delete 누락
반복
메모리 사용량 계속 증가
```

---

## 오해 4. “reference는 pointer보다 항상 안전하다”

reference는 null이 될 수 없다는 점에서 더 제한적이다.
하지만 lifetime 문제를 자동으로 해결하지는 않는다.

```cpp
int& bad()
{
    int x = 42;
    return x; // 죽을 stack 객체에 대한 reference 반환
}
```

이것도 dangling reference다.

---

## 오해 5. “RAII를 쓰면 메모리 문제는 전부 사라진다”

아니다.

RAII는 resource lifetime을 object lifetime에 묶어 많은 문제를 줄인다.
하지만 다음 문제들은 여전히 가능하다.

```text
1. 잘못된 raw pointer aliasing
2. container에 남은 dangling pointer
3. 순환 참조
4. 배열 경계 초과
5. uninitialized read
6. 잘못된 복사 생성자 / 대입 연산자
```

C++98에서는 특히 Rule of Three를 조심해야 한다.

---

# 10. Object Lifetime과 Scope Lifetime

이 부분이 중요하다.

## Scope

scope는 이름을 사용할 수 있는 코드 범위다.

```cpp
void f()
{
    int x = 10;
    // 여기서 x 사용 가능
}
// 여기서 x 이름 사용 불가
```

## Lifetime

lifetime은 객체가 실제로 살아 있는 기간이다.

```cpp
int* p = new int(10);
delete p;
```

`p`라는 이름의 scope와 `new int(10)`으로 만들어진 객체의 lifetime은 다르다.

---

## Scope와 Lifetime 비교

```cpp
int* make()
{
    int* p = new int(42);
    return p;
}
```

`p`의 scope:

```text
make() 함수 내부
```

heap object의 lifetime:

```text
new 실행 시 시작
delete 실행 시 종료
```

즉:

```text
p 변수는 사라져도 heap object는 살아 있을 수 있다.
heap object가 죽어도 pointer 값은 남아 있을 수 있다.
```

이 두 문장을 반드시 기억해야 한다.

---

# 11. Ownership

ownership은 C/C++ memory debugging의 핵심 개념이다.

> **ownership이란 “누가 이 resource를 해제할 책임이 있는가”이다.**

예를 들어:

```cpp
User* user = new User(fd);
```

여기서 질문은 이것이다.

```text
누가 delete user를 호출해야 하는가?
```

가능한 설계:

```text
Server owns User
Channel observes User
Parser does not own User
```

이 경우:

| 객체           | User에 대한 관계                |
| ------------ | -------------------------- |
| Server       | owns                       |
| Channel      | observes / references      |
| Parser       | temporary access           |
| UserDatabase | owns or indexes, 설계에 따라 다름 |

소유자가 여러 명이면 위험하다.

```cpp
delete user; // Server
delete user; // Channel
```

이러면 double free다.

소유자가 아무도 없으면 위험하다.

```cpp
new User(fd);
// 아무도 delete하지 않음
```

이러면 leak이다.

---

# 12. C++ Destructor와 Lifetime

C++에서 destructor는 객체 lifetime이 끝날 때 호출된다.

```cpp
class User
{
public:
    User()
    {
        std::cout << "User created\n";
    }

    ~User()
    {
        std::cout << "User destroyed\n";
    }
};
```

stack object:

```cpp
void f()
{
    User u;
}
```

`f()`가 끝나면 `u`의 destructor가 자동 호출된다.

heap object:

```cpp
void f()
{
    User* u = new User();
    delete u;
}
```

heap object는 `delete`를 호출해야 destructor가 실행된다.

```text
new User()
1. heap memory allocation
2. constructor call

delete u
1. destructor call
2. heap memory deallocation
```

이 순서가 중요하다.

---

# 13. RAII 기본 개념

RAII는 다음 원칙이다.

> **Resource Acquisition Is Initialization**
> resource의 획득과 해제를 객체 lifetime에 묶는다.

예를 들어 file descriptor를 직접 관리하면 위험하다.

```cpp
void f()
{
    int fd = open("file.txt", O_RDONLY);

    if (some_error)
        return; // close(fd) 누락 가능

    close(fd);
}
```

RAII 스타일에서는 객체 destructor가 정리한다.

```cpp
class FdGuard
{
private:
    int _fd;

public:
    FdGuard(int fd) : _fd(fd) {}

    ~FdGuard()
    {
        if (_fd >= 0)
            close(_fd);
    }

private:
    FdGuard(const FdGuard&);
    FdGuard& operator=(const FdGuard&);
};
```

사용:

```cpp
void f()
{
    FdGuard guard(open("file.txt", O_RDONLY));

    if (some_error)
        return;

    // return해도 guard destructor가 close 수행
}
```

C++98에서는 `std::unique_ptr`가 없지만, RAII class를 직접 만들 수 있다.

---

# 14. Debugging Flow: symptom → repro → hypothesis → evidence → root cause → fix → prevention

메모리 디버깅은 항상 이 흐름으로 간다.

## 예제 문제

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

---

## 1. Symptom

프로그램이 가끔 이상한 값을 출력하거나 crash한다.

```text
42가 나올 수도 있고
이상한 값이 나올 수도 있고
segfault가 날 수도 있다.
```

---

## 2. Repro

최소 재현 케이스를 만든다.

```bash
c++ -g -O0 uaf.cpp -o uaf
./uaf
```

더 강하게 재현:

```bash
c++ -g -O1 -fsanitize=address -fno-omit-frame-pointer uaf.cpp -o uaf
./uaf
```

---

## 3. Hypothesis

가설:

```text
p가 delete된 뒤에도 사용되고 있다.
```

---

## 4. Evidence

ASan report:

```text
heap-use-after-free
READ of size 4
freed by thread T0 here
previously allocated by thread T0 here
```

---

## 5. Root Cause

`delete p` 이후 `*p`를 읽었다.

즉, object lifetime이 끝난 후 접근했다.

---

## 6. Fix

사용 후 delete하거나, delete 이후 접근하지 않는다.

```cpp
int* p = new int(42);
std::cout << *p << std::endl;
delete p;
```

더 좋은 설계는 heap allocation 자체가 필요 없는 경우 stack을 쓴다.

```cpp
int x = 42;
std::cout << x << std::endl;
```

---

## 7. Prevention

```text
1. new/delete를 최소화한다.
2. ownership rule을 명확히 한다.
3. delete 이후 pointer를 쓰지 않는다.
4. container에 raw pointer를 저장할 때 소유권을 주석으로 명시한다.
5. destructor에서 resource를 정리한다.
6. sanitizer/valgrind target을 Makefile에 둔다.
```

---

# 15. 확인 문제

다음 코드를 보고 문제를 판단해라.

## 문제 1

```cpp
int* f()
{
    int x = 10;
    return &x;
}
```

질문:

```text
1. 이 코드는 왜 위험한가?
2. x는 어느 영역에 있는가?
3. 반환된 pointer는 어떤 상태인가?
```

---

## 문제 2

```cpp
void f()
{
    int* p = new int(10);
}
```

질문:

```text
1. p는 어느 영역에 있는가?
2. new int(10)은 어느 영역에 있는가?
3. f()가 끝난 뒤 무엇이 leak되는가?
```

---

## 문제 3

```cpp
int* p = new int(10);
int* q = p;

delete p;

std::cout << *q << std::endl;
```

질문:

```text
1. p만 위험한가, q도 위험한가?
2. delete p 이후 heap object의 lifetime은 어떤 상태인가?
3. 이 문제의 이름은 무엇인가?
```

---

## 문제 4

```cpp
int arr[3];
arr[3] = 100;
```

질문:

```text
1. 왜 컴파일될 수 있는가?
2. 왜 실행 결과가 보장되지 않는가?
3. 이것은 어떤 종류의 메모리 문제인가?
```

---

# 16. 실습 과제

## 실습 1. Stack dangling pointer 만들기

```cpp
#include <iostream>

int* bad()
{
    int x = 42;
    return &x;
}

int main()
{
    int* p = bad();
    std::cout << *p << std::endl;
    return 0;
}
```

다음으로 실행해라.

```bash
c++ -Wall -Wextra -g -O0 bad.cpp -o bad
./bad
```

그리고 sanitizer로 실행해라.

```bash
c++ -Wall -Wextra -g -O1 -fsanitize=address -fno-omit-frame-pointer bad.cpp -o bad
./bad
```

관찰할 것:

```text
1. 컴파일러 경고가 나오는가?
2. 실행 결과가 매번 같은가?
3. sanitizer가 무엇을 보고하는가?
```

---

## 실습 2. Heap use-after-free 만들기

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

실행:

```bash
c++ -Wall -Wextra -g -O1 -fsanitize=address -fno-omit-frame-pointer uaf.cpp -o uaf
./uaf
```

ASan report에서 다음 세 줄을 찾아라.

```text
1. invalid access site
2. freed by
3. allocated by
```

---

## 실습 3. Leak 만들기

```cpp
int main()
{
    int* p = new int(42);
    return 0;
}
```

Linux에서 Valgrind:

```bash
c++ -Wall -Wextra -g -O0 leak.cpp -o leak
valgrind --leak-check=full ./leak
```

확인할 것:

```text
1. definitely lost가 나오는가?
2. allocation site는 어디인가?
3. delete를 추가하면 report가 어떻게 바뀌는가?
```

---

## 실습 4. Ownership 주석 달기

ft_irc 같은 프로젝트에서 raw pointer가 들어간 container를 찾아라.

예:

```cpp
std::map<int, User*> _users;
std::vector<User*> _channelUsers;
```

각각에 주석을 달아라.

```cpp
// owns User objects: Server deletes them on disconnect
std::map<int, User*> _users;

// non-owning pointers: Channel must not delete User
std::vector<User*> _channelUsers;
```

이 작업은 단순 주석이 아니다.
나중에 double free와 use-after-free를 막는 설계 문서 역할을 한다.

---

# 17. 핵심 정리

이번 Lecture 11의 핵심은 다음이다.

## 1. C/C++ 메모리 영역

```text
text      : 실행 코드
static    : 전역/static 변수
heap      : new/malloc으로 동적 할당
stack     : 함수 호출, 지역 변수
```

## 2. Stack과 Heap의 차이

| 구분       | Stack                  | Heap                              |
| -------- | ---------------------- | --------------------------------- |
| 생성       | 함수 호출 시 자동             | `new`, `malloc`                   |
| 해제       | scope 종료 시 자동          | `delete`, `free`                  |
| 속도       | 일반적으로 빠름               | 상대적으로 비용 있음                       |
| 위험       | dangling stack address | leak, use-after-free, double free |
| lifetime | 함수 실행 범위와 강하게 연결       | 명시적 해제 전까지 유지                     |

## 3. Scope와 Lifetime은 다르다

```text
scope    = 이름을 사용할 수 있는 코드 범위
lifetime = 객체가 실제로 살아 있는 기간
```

## 4. Ownership은 핵심이다

```text
누가 delete/free/close 해야 하는가?
```

이 질문이 불명확하면 memory leak, double free, use-after-free가 발생한다.

## 5. `delete`는 pointer를 없애지 않는다

```cpp
delete p;
```

이것은 `p`가 가리키는 object의 lifetime을 끝낸다.
`p` 변수 자체를 `NULL`로 만들지는 않는다.

## 6. Undefined Behavior는 디버깅을 어렵게 만든다

UB는 “항상 crash”가 아니다.

```text
정상처럼 보일 수 있음
가끔 crash할 수 있음
최적화 옵션에 따라 달라질 수 있음
debugger 붙이면 사라질 수 있음
다른 컴퓨터에서 다르게 보일 수 있음
```

## 7. RAII는 C++ memory debugging의 중심 철학이다

RAII는 resource lifetime을 object lifetime에 묶는다.

```text
생성자에서 resource 획득
소멸자에서 resource 해제
```

C++98에서도 직접 RAII class를 만들 수 있다.

---