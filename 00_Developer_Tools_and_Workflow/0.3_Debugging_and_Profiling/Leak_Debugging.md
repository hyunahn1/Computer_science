# Leak Debugging

이번 강의의 핵심은 이것이다.

> **memory leak은 단순히 `delete`를 까먹은 문제가 아니다.**
> 더 정확히 말하면, **heap allocation을 해제할 책임을 가진 ownership path가 끊어진 상태**다.

---

# 1. 핵심 질문

이번 강의에서 답할 질문은 다음이다.

```text
1. memory leak이 정확히 무엇인가?
2. leak과 still reachable은 무엇이 다른가?
3. Valgrind의 definitely lost / indirectly lost / possibly lost / still reachable은 어떻게 읽는가?
4. allocation site와 deallocation site는 어떻게 추적하는가?
5. ownership graph란 무엇인가?
6. C++ destructor와 RAII는 leak을 어떻게 줄이는가?
7. container가 pointer를 보관할 때 누가 delete해야 하는가?
8. 42 C++98 환경에서 leak을 줄이는 현실적인 설계는 무엇인가?
```

---

# 2. 개념 설명

## 2.1 Memory leak이란 무엇인가

memory leak은 다음 상태다.

> **heap에 할당된 메모리가 아직 해제되지 않았고, 프로그램이 그 메모리에 도달할 수 있는 유효한 경로도 잃어버린 상태**

가장 단순한 예시는 이것이다.

```cpp
int main()
{
    int* p = new int(42);
    return 0;
}
```

여기서 `new int(42)`는 heap에 `int` 객체를 만든다.

```text
stack
+-------------+
| p --------- | ----+
+-------------+     |
                    v
heap
+-------------+
| int 42      |
+-------------+
```

하지만 `main()`이 끝나면 `p`는 사라진다.

```text
stack
+-------------+
| p 사라짐    |
+-------------+

heap
+-------------+
| int 42      |  <-- 아직 delete 안 됨
+-------------+
```

이제 프로그램 안에서 저 heap block에 접근할 방법이 없다.
이것이 leak이다.

---

## 2.2 Leak은 “메모리가 남아 있다”만으로 정의되지 않는다

중요하다.

프로그램 종료 시점에 heap memory가 남아 있다고 해서 항상 심각한 leak은 아니다.

예를 들어:

```cpp
static std::string* cache = new std::string("hello");
```

이런 전역 cache가 프로그램 종료 시까지 살아 있을 수 있다.
Valgrind는 이것을 `still reachable`로 볼 수 있다.

즉, leak report를 볼 때는 다음을 구분해야 한다.

```text
정말 잃어버린 메모리인가?
아직 pointer 경로가 남아 있는 메모리인가?
프로그램 설계상 종료 시까지 유지되는 메모리인가?
반복 실행 중 계속 증가하는 메모리인가?
```

---

# 3. 왜 필요한지

서버 프로그램에서는 leak이 특히 위험하다.

ft_irc 같은 서버를 생각하자.

```text
client 접속
    ↓
User allocation
    ↓
client 종료
    ↓
User delete 누락
    ↓
다음 client 접속
    ↓
또 User allocation
    ↓
누적
```

작은 테스트에서는 문제가 안 보일 수 있다.

```text
1명 접속 → 종료
메모리 200 bytes leak
```

하지만 오래 실행되면 문제가 된다.

```text
10,000명 접속/종료
200 bytes × 10,000 = 2,000,000 bytes leak

더 복잡한 User/Channel/Buffer 구조라면 훨씬 커짐
```

memory leak은 보통 즉시 crash를 만들지 않는다.
그래서 더 위험하다.

```text
segfault:
    바로 드러나는 경우가 많음

memory leak:
    천천히 누적됨
    테스트에서는 통과할 수 있음
    운영 중 서버가 점점 느려짐
    결국 OOM 또는 성능 저하 발생
```

---

# 4. 내부 원리 / 작동 방식

## 4.1 Heap allocation의 생명주기

C++에서 `new`는 대략 두 단계를 수행한다.

```cpp
User* u = new User(fd);
```

내부적으로는 개념상 다음과 같다.

```text
1. operator new가 heap memory를 확보한다.
2. User constructor가 그 memory 위에 객체를 초기화한다.
```

반대로 `delete`는 다음 순서다.

```cpp
delete u;
```

```text
1. User destructor를 호출한다.
2. operator delete가 heap memory를 allocator에게 반환한다.
```

즉:

```text
new
    allocation + constructor

delete
    destructor + deallocation
```

---

## 4.2 Leak은 어디서 발생하는가

leak은 다음 세 지점 사이에서 발생한다.

```text
allocation site
    ↓
ownership transfer
    ↓
deallocation site
```

예를 들어:

```cpp
User* createUser(int fd)
{
    return new User(fd);
}

void acceptClient(int fd)
{
    User* user = createUser(fd);
    _users[fd] = user;
}
```

여기서 allocation site는 `createUser()` 안의 `new User(fd)`다.

하지만 deallocation은 다른 곳에서 해야 한다.

```cpp
void disconnect(int fd)
{
    delete _users[fd];
    _users.erase(fd);
}
```

문제는 이 경로가 끊어질 때 생긴다.

```text
User allocated
    ↓
_users[fd]에 저장됨
    ↓
disconnect에서 erase만 함
    ↓
delete 없음
    ↓
leak
```

잘못된 코드:

```cpp
void disconnect(int fd)
{
    _users.erase(fd); // delete 없이 pointer만 제거
}
```

이 경우 map에 있던 pointer 값이 사라진다.
하지만 heap의 `User` 객체는 그대로 남아 있다.

```text
삭제 전

_users map
+-----+------------+
| fd  | User* -----|----+
+-----+------------+    |
                        v
heap
+----------------+
| User object    |
+----------------+

erase 후

_users map
+-----+------------+
| 비어 있음       |
+-----+------------+

heap
+----------------+
| User object    |  <-- 접근 경로 상실
+----------------+
```

이것이 leak이다.

---

## 4.3 Ownership graph

leak debugging을 제대로 하려면 pointer 하나만 보면 안 된다.
객체들의 ownership 관계를 그래프로 봐야 한다.

예:

```text
Server
  ├── owns User A
  ├── owns User B
  └── owns Channel #lobby
          ├── observes User A
          └── observes User B
```

여기서 중요한 점:

```text
Server → User
    owning edge

Channel → User
    non-owning edge
```

그림으로 보면:

```text
            owns
Server --------------> User A
   |
   | owns
   v
Channel -------------> User A
          observes
```

Channel이 `User*`를 들고 있다고 해서 Channel이 `delete User`를 해야 하는 것은 아니다.

이 구분이 없으면 두 가지 문제가 생긴다.

```text
1. 아무도 delete하지 않음
   → memory leak

2. 둘 이상이 delete함
   → double free
```

---

# 5. 쉬운 예시

## 5.1 가장 단순한 leak

```cpp
int main()
{
    int* p = new int(42);
    return 0;
}
```

문제:

```text
new는 있음
delete는 없음
```

수정:

```cpp
int main()
{
    int* p = new int(42);
    delete p;
    return 0;
}
```

더 좋은 수정:

```cpp
int main()
{
    int x = 42;
    return 0;
}
```

heap이 필요 없으면 stack을 쓰는 것이 가장 좋다.

---

## 5.2 Array leak

```cpp
int main()
{
    int* arr = new int[10];
    delete arr; // 잘못됨
    return 0;
}
```

이것은 leak과 UB를 모두 만들 수 있다.

`new[]`로 할당했으면 `delete[]`로 해제해야 한다.

```cpp
int main()
{
    int* arr = new int[10];
    delete[] arr;
    return 0;
}
```

규칙:

| 할당         | 해제         |
| ---------- | ---------- |
| `new T`    | `delete`   |
| `new T[n]` | `delete[]` |
| `malloc`   | `free`     |

섞으면 안 된다.

```cpp
int* p = new int;
free(p); // 잘못됨
```

```cpp
int* p = (int*)malloc(sizeof(int));
delete p; // 잘못됨
```

---

## 5.3 Early return leak

```cpp
bool process()
{
    int* buffer = new int[100];

    if (some_error())
        return false; // delete[] buffer 누락

    delete[] buffer;
    return true;
}
```

이런 코드는 실무에서 많이 나온다.

수정:

```cpp
bool process()
{
    int* buffer = new int[100];

    if (some_error())
    {
        delete[] buffer;
        return false;
    }

    delete[] buffer;
    return true;
}
```

하지만 이 방식은 error path가 많아지면 위험하다.

```cpp
bool process()
{
    int* buffer = new int[100];

    if (error1())
        return false;

    if (error2())
        return false;

    if (error3())
        return false;

    delete[] buffer;
    return true;
}
```

이런 경우 RAII가 필요하다.

---

## 5.4 C++98 RAII로 해결

C++98에서는 `std::unique_ptr`가 없다.
하지만 직접 RAII class를 만들 수 있다.

```cpp
class IntArray
{
private:
    int* _data;

public:
    IntArray(size_t n)
        : _data(new int[n])
    {
    }

    ~IntArray()
    {
        delete[] _data;
    }

    int& operator[](size_t i)
    {
        return _data[i];
    }

private:
    IntArray(const IntArray&);
    IntArray& operator=(const IntArray&);
};
```

사용:

```cpp
bool process()
{
    IntArray buffer(100);

    if (some_error())
        return false;

    return true;
}
```

`process()`가 어떤 경로로 종료되든 `buffer`의 destructor가 호출된다.

```text
process() 진입
    ↓
IntArray 생성
    ↓
new int[100]
    ↓
중간 return 발생 가능
    ↓
IntArray destructor
    ↓
delete[] 자동 호출
```

---

# 6. Valgrind leak 종류

Valgrind leak report에서 가장 중요한 네 가지가 있다.

```text
definitely lost
indirectly lost
possibly lost
still reachable
```

하나씩 보자.

---

## 6.1 Definitely lost

가장 명확한 leak이다.

```cpp
int main()
{
    int* p = new int(42);
    p = NULL;
    return 0;
}
```

처음에는:

```text
p ----> heap int 42
```

그런데 `p = NULL` 이후:

```text
p ----> NULL

heap int 42에 도달할 pointer 없음
```

Valgrind는 이것을 `definitely lost`로 보고한다.

```text
4 bytes in 1 blocks are definitely lost
```

의미:

> **이 heap block에 도달할 수 있는 pointer를 완전히 잃어버렸다.**

이건 거의 반드시 고쳐야 한다.

---

## 6.2 Indirectly lost

indirectly lost는 “직접 잃어버린 객체가 소유하던 다른 객체도 같이 잃어버렸다”는 뜻이다.

예:

```cpp
struct Node
{
    int* data;
};

int main()
{
    Node* n = new Node;
    n->data = new int(42);

    n = NULL;
    return 0;
}
```

처음:

```text
n ----> Node ----> int 42
```

`n = NULL` 이후:

```text
n ----> NULL

heap
+--------+       +--------+
| Node   | ----> | int 42 |
+--------+       +--------+
```

`Node`를 잃어버렸기 때문에 `Node->data`도 접근할 수 없다.

Valgrind 관점:

```text
Node:
    definitely lost

int 42:
    indirectly lost
```

즉:

> **root object를 잃어버려서, 그 안에서만 접근 가능하던 하위 allocation도 같이 잃어버린 상태**

---

## 6.3 Possibly lost

possibly lost는 애매한 경우다.

Valgrind가 어떤 heap block의 시작 주소를 가리키는 pointer는 못 찾았지만, block 내부를 가리키는 pointer는 발견한 경우가 있다.

예를 들어:

```cpp
int main()
{
    char* p = new char[100];
    char* q = p + 10;

    p = NULL;

    // q는 block 내부를 가리킴
    return 0;
}
```

그림:

```text
heap block
+--------------------------------+
| byte0 byte1 ... byte10 ...     |
+--------------------------------+
^                ^
p 원래 위치       q
```

`p`는 사라졌지만 `q`는 block 내부를 가리킨다.

Valgrind는 고민한다.

```text
q를 통해 원래 block을 해제할 수 있는가?
보통 delete[] q는 불가능하다.
하지만 내부 pointer가 남아 있기는 하다.
```

그래서 `possibly lost`라고 보고할 수 있다.

의미:

> **정확히 leak인지 확신하기 어렵지만, 위험한 pointer 상태다.**

대부분은 점검해야 한다.

---

## 6.4 Still reachable

still reachable은 프로그램 종료 시점에 아직 pointer 경로가 남아 있는 heap memory다.

예:

```cpp
int* g_ptr;

int main()
{
    g_ptr = new int(42);
    return 0;
}
```

종료 시점에도 전역 변수 `g_ptr`이 heap block을 가리킨다.

```text
global/static
+--------+
| g_ptr -|----+
+--------+    |
              v
heap
+--------+
| int 42 |
+--------+
```

Valgrind는 이것을 `still reachable`로 볼 수 있다.

의미:

> **해제되지는 않았지만, 프로그램이 종료 시점에도 여전히 pointer로 접근 가능하다.**

이것은 `definitely lost`보다 덜 심각하다.
하지만 장기 실행 서버에서는 설계상 의도된 것인지 확인해야 한다.

---

# 7. Leak report 읽기

예제 코드:

```cpp
#include <iostream>

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

Valgrind 실행:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./leak
```

예상 report:

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

읽는 순서:

## 1단계: in use at exit

```text
in use at exit: 4 bytes in 1 blocks
```

프로그램 종료 시점에 heap block 1개, 4 bytes가 남아 있다.

---

## 2단계: total heap usage

```text
total heap usage: 1 allocs, 0 frees, 4 bytes allocated
```

총 1번 allocation했고, free/delete는 0번 했다.

---

## 3단계: leak kind

```text
definitely lost
```

명확한 leak이다.

---

## 4단계: allocation site

```text
main (leak.cpp:5)
```

`leak.cpp` 5번째 줄에서 할당되었다.

---

## 5단계: root cause 문장으로 바꾸기

```text
leak.cpp:5에서 new int(42)를 했지만,
프로그램 종료 전까지 해당 allocation에 대해 delete가 호출되지 않았다.
```

---

# 8. 실무 예시: ft_irc User leak

## 8.1 문제 코드

```cpp
void Server::acceptClient(int fd)
{
    User* user = new User(fd);
    _users[fd] = user;
}
```

disconnect:

```cpp
void Server::disconnect(int fd)
{
    _users.erase(fd);
    close(fd);
}
```

이 코드는 leak이다.

왜냐하면 `_users.erase(fd)`는 map entry만 지운다.
그 안에 들어 있던 `User*`가 가리키는 객체를 delete하지 않는다.

---

## 8.2 Symptom

```text
서버는 crash하지 않는다.
하지만 client가 접속/종료할 때마다 heap memory가 증가한다.
Valgrind에서 User allocation이 definitely lost로 나온다.
```

---

## 8.3 Repro

짧고 반복 가능한 시나리오를 만든다.

```text
1. 서버 실행
2. client 1 접속
3. client 1 종료
4. 서버 종료
5. Valgrind leak report 확인
```

명령:

```bash
valgrind \
  --leak-check=full \
  --show-leak-kinds=all \
  ./ircserv 6667 pass
```

---

## 8.4 Hypothesis

```text
Server::_users에서 User*를 erase하기 전에 delete하지 않았다.
```

---

## 8.5 Evidence

Valgrind report에 다음 같은 stack trace가 나온다고 하자.

```text
X bytes in 1 blocks are definitely lost
    at 0x...: operator new(unsigned long)
    by 0x...: Server::acceptClient(int) (Server.cpp:120)
    by 0x...: Server::run() (Server.cpp:300)
```

이것은 말한다.

```text
User는 Server::acceptClient에서 allocation되었다.
하지만 프로그램 종료 전까지 그 block을 해제하는 delete가 없었다.
```

---

## 8.6 Root Cause

```text
_users.erase(fd)는 User 객체를 삭제하지 않는다.
map 안의 pointer 값만 제거한다.
```

C++ container는 raw pointer가 가리키는 객체를 자동으로 delete하지 않는다.

```cpp
std::map<int, User*> users;
```

이 map은 `User*` 값을 저장할 뿐이다.
`User` 객체의 lifetime을 자동 관리하지 않는다.

---

## 8.7 Fix

```cpp
void Server::disconnect(int fd)
{
    std::map<int, User*>::iterator it = _users.find(fd);
    if (it != _users.end())
    {
        delete it->second;
        _users.erase(it);
    }

    close(fd);
}
```

하지만 실제 ft_irc에서는 이것만으로 부족할 수 있다.

Channel이 User pointer를 들고 있다면, 순서가 중요하다.

```cpp
void Server::disconnect(int fd)
{
    std::map<int, User*>::iterator it = _users.find(fd);
    if (it == _users.end())
        return;

    User* user = it->second;

    removeUserFromAllChannels(user);

    delete user;
    _users.erase(it);
    close(fd);
}
```

---

## 8.8 Prevention

```text
1. _users는 owning container라고 명시한다.
2. Channel의 User* vector는 non-owning이라고 명시한다.
3. disconnect 처리는 반드시 한 함수로 모은다.
4. erase 전에 delete할 것인지, delete 전에 observer 제거할 것인지 순서를 문서화한다.
5. Valgrind로 connect/disconnect 반복 시나리오를 검사한다.
```

---

# 9. Container가 메모리를 소유하는 경우

## 9.1 `std::vector<int>`는 안전한 편이다

```cpp
std::vector<int> nums;
nums.push_back(1);
nums.push_back(2);
```

`vector<int>`는 내부 메모리를 직접 관리한다.
`nums`가 destructor를 호출할 때 내부 buffer도 정리된다.

```text
vector<int>
    owns its internal array
```

---

## 9.2 `std::vector<User*>`는 다르다

```cpp
std::vector<User*> users;
users.push_back(new User(1));
users.push_back(new User(2));
```

`vector<User*>`는 pointer 값들을 저장한다.
그러나 pointer가 가리키는 `User` 객체를 자동으로 delete하지 않는다.

```text
vector destructor:
    pointer 값들이 들어 있던 내부 배열은 정리함

하지만:
    각 pointer가 가리키던 User object는 delete하지 않음
```

잘못된 기대:

```text
vector가 사라지면 안에 있던 User*도 알아서 delete하겠지?
```

아니다.

직접 해야 한다.

```cpp
for (size_t i = 0; i < users.size(); ++i)
    delete users[i];

users.clear();
```

---

## 9.3 `std::map<int, User*>`도 마찬가지

```cpp
std::map<int, User*> users;
users[fd] = new User(fd);
```

`users.clear()`는 map node와 pointer 값만 정리한다.

```cpp
users.clear(); // User 객체 delete 안 함
```

올바른 정리:

```cpp
for (std::map<int, User*>::iterator it = users.begin();
     it != users.end();
     ++it)
{
    delete it->second;
}

users.clear();
```

---

# 10. Destructor에서 정리하기

Server가 User를 소유한다면, Server destructor에서 남은 User를 정리해야 한다.

```cpp
class Server
{
private:
    std::map<int, User*> _users;

public:
    ~Server()
    {
        for (std::map<int, User*>::iterator it = _users.begin();
             it != _users.end();
             ++it)
        {
            delete it->second;
        }
        _users.clear();
    }
};
```

이렇게 하면 서버 종료 시점에 아직 남은 User도 정리된다.

하지만 주의해야 한다.

```text
destructor는 마지막 방어선이다.
정상적인 disconnect path에서도 delete가 되어야 한다.
```

왜냐하면 서버가 오래 실행되는 동안에는 destructor가 호출되지 않는다.

```text
장기 실행 서버:
    Server destructor는 프로그램 종료 때만 호출됨

따라서:
    client disconnect 때마다 User를 정리해야 함
```

---

# 11. RAII와 Rule of Three

C++98에서 직접 resource를 소유하는 class를 만들면 Rule of Three를 고려해야 한다.

## 11.1 위험한 class

```cpp
class Buffer
{
private:
    char* _data;

public:
    Buffer(size_t size)
        : _data(new char[size])
    {
    }

    ~Buffer()
    {
        delete[] _data;
    }
};
```

겉보기에는 좋아 보인다.
하지만 복사되면 문제가 생긴다.

```cpp
Buffer a(100);
Buffer b = a;
```

컴파일러가 자동 생성한 copy constructor는 pointer 값만 복사한다.

```text
a._data ----+
            v
          heap buffer
            ^
b._data ----+
```

이제 `a`와 `b`가 같은 heap buffer를 소유한다고 착각한다.

scope 종료 시:

```text
b destructor → delete[] buffer
a destructor → delete[] same buffer
```

결과:

```text
double free
```

---

## 11.2 Rule of Three

C++98에서 resource를 직접 소유하는 class는 보통 다음 셋을 직접 정의해야 한다.

```text
1. destructor
2. copy constructor
3. copy assignment operator
```

이것을 Rule of Three라고 한다.

---

## 11.3 복사를 금지하는 방식

42 과제에서 가장 단순한 방법은 복사를 막는 것이다.

```cpp
class Buffer
{
private:
    char* _data;

public:
    Buffer(size_t size)
        : _data(new char[size])
    {
    }

    ~Buffer()
    {
        delete[] _data;
    }

private:
    Buffer(const Buffer&);
    Buffer& operator=(const Buffer&);
};
```

C++98에서는 `= delete`가 없으므로 private에 선언만 해두는 방식이 흔하다.

이러면 외부에서 복사하려 할 때 컴파일 또는 링크 단계에서 막힌다.

---

## 11.4 깊은 복사를 구현하는 방식

복사가 필요하다면 deep copy를 해야 한다.

```cpp
class Buffer
{
private:
    char* _data;
    size_t _size;

public:
    Buffer(size_t size)
        : _data(new char[size]), _size(size)
    {
    }

    ~Buffer()
    {
        delete[] _data;
    }

    Buffer(const Buffer& other)
        : _data(new char[other._size]), _size(other._size)
    {
        for (size_t i = 0; i < _size; ++i)
            _data[i] = other._data[i];
    }

    Buffer& operator=(const Buffer& other)
    {
        if (this != &other)
        {
            char* newData = new char[other._size];

            for (size_t i = 0; i < other._size; ++i)
                newData[i] = other._data[i];

            delete[] _data;

            _data = newData;
            _size = other._size;
        }

        return *this;
    }
};
```

중요한 점:

```text
assignment operator에서 먼저 새 메모리를 할당한 뒤,
성공하면 기존 메모리를 delete한다.
```

이 방식은 exception safety 측면에서도 더 안전하다.

---

# 12. Leak fix 전략

memory leak을 고치는 순서는 다음과 같다.

## 12.1 1단계: allocation site 찾기

Valgrind report에서 `new`가 일어난 위치를 본다.

```text
by Server::acceptClient(int) (Server.cpp:120)
```

질문:

```text
이 allocation은 어떤 객체를 만든 것인가?
누가 이 객체를 소유해야 하는가?
```

---

## 12.2 2단계: ownership 결정

예:

```text
User는 Server가 소유한다.
Channel은 User를 관찰만 한다.
Parser는 User를 소유하지 않는다.
```

---

## 12.3 3단계: deallocation site 찾기

질문:

```text
이 객체의 lifetime은 언제 끝나야 하는가?
```

User라면:

```text
client disconnect 시
server shutdown 시
accept 실패 시
registration 실패 시
```

즉, 정상 종료 path뿐 아니라 실패 path도 봐야 한다.

---

## 12.4 4단계: 모든 exit path 확인

예:

```cpp
bool Server::registerUser(int fd)
{
    User* user = new User(fd);

    if (!checkPassword(user))
        return false;

    if (!readNick(user))
        return false;

    _users[fd] = user;
    return true;
}
```

문제:

```text
checkPassword 실패 시 leak
readNick 실패 시 leak
```

수정:

```cpp
bool Server::registerUser(int fd)
{
    User* user = new User(fd);

    if (!checkPassword(user))
    {
        delete user;
        return false;
    }

    if (!readNick(user))
    {
        delete user;
        return false;
    }

    _users[fd] = user;
    return true;
}
```

더 좋은 구조:

```cpp
bool Server::registerUser(int fd)
{
    User* user = new User(fd);

    if (!checkPassword(user) || !readNick(user))
    {
        delete user;
        return false;
    }

    _users[fd] = user;
    return true;
}
```

그러나 error path가 많아지면 RAII wrapper를 고려한다.

---

## 12.5 5단계: destructor로 마지막 방어선 만들기

```cpp
Server::~Server()
{
    clearUsers();
    clearChannels();
}
```

```cpp
void Server::clearUsers()
{
    for (std::map<int, User*>::iterator it = _users.begin();
         it != _users.end();
         ++it)
    {
        delete it->second;
    }

    _users.clear();
}
```

---

# 13. False Positive / False Negative

## 13.1 False positive

Valgrind에서 `still reachable`이 보인다고 해서 항상 버그는 아니다.

예:

```cpp
class Logger
{
public:
    static Logger& instance()
    {
        static Logger* logger = new Logger;
        return *logger;
    }
};
```

이런 singleton pattern은 종료 시점에 `still reachable`로 나올 수 있다.

하지만 42 과제에서는 이런 구조를 굳이 만들 필요가 적다.
가능하면 종료 시 명확히 정리하는 편이 좋다.

---

## 13.2 False negative

도구가 모든 leak 설계 문제를 잡아주지는 않는다.

예:

```cpp
std::vector<User*> oldUsers;
std::vector<User*> activeUsers;

User* u = new User(fd);
oldUsers.push_back(u);
activeUsers.push_back(u);
```

disconnect 시:

```cpp
delete u;
```

이건 leak은 아니다.
하지만 `oldUsers`와 `activeUsers`에는 dangling pointer가 남을 수 있다.

Valgrind leak report는 깨끗할 수 있다.
하지만 use-after-free 위험은 남아 있다.

즉:

```text
Leak이 없다는 것 ≠ memory ownership 설계가 안전하다는 것
```

---

# 14. macOS / Linux 차이

## 14.1 Linux

Linux에서는 Valgrind가 leak debugging에 매우 강하다.

추천 명령:

```bash
valgrind \
  --leak-check=full \
  --show-leak-kinds=all \
  --track-origins=yes \
  ./program
```

서버 프로그램이면 client scenario를 짧게 만든 뒤 종료시켜야 한다.

예:

```text
1. 서버 실행
2. client 접속
3. client 종료
4. 서버 종료
5. leak report 확인
```

서버가 계속 실행 중이면 Valgrind가 최종 leak summary를 보여주지 못한다.
따라서 테스트용 종료 조건을 만들거나 `Ctrl+C` 종료 후 destructor가 정상 호출되는지 확인해야 한다.

---

## 14.2 macOS

macOS에서는 Valgrind가 제한적일 수 있다.

대안:

```text
1. AddressSanitizer
2. LeakSanitizer가 가능한 환경이면 LSan
3. leaks 명령
4. Instruments
5. lldb
```

ASan leak detection은 macOS에서 Linux와 다르게 동작할 수 있다.
따라서 42 평가 전에는 Linux에서 Valgrind로 확인하는 것이 좋다.

---

## 14.3 42 환경

42 과제 기준으로는 다음 흐름이 현실적이다.

```text
개발 중 macOS:
    ASan/lldb로 빠르게 잡기

제출 전 Linux:
    Valgrind로 leak 확인

Makefile:
    제출용과 debug/sanitizer용 target 분리
```

추천 Makefile target:

```makefile
leak:
	$(CXX) $(CXXFLAGS) -g -O0 $(SRCS) -o $(NAME)
	valgrind --leak-check=full --show-leak-kinds=all ./$(NAME)
```

서버 프로그램은 인자가 필요하므로:

```makefile
leak:
	$(CXX) $(CXXFLAGS) -g -O0 $(SRCS) -o $(NAME)
	valgrind --leak-check=full --show-leak-kinds=all ./$(NAME) 6667 pass
```

다만 Makefile에서 서버를 실행하면 종료 타이밍 관리가 어렵다.
실무적으로는 빌드 target과 실행 명령을 분리하는 편이 낫다.

```makefile
debug:
	$(CXX) $(CXXFLAGS) -g -O0 $(SRCS) -o $(NAME)
```

그 후 직접 실행:

```bash
make debug
valgrind --leak-check=full --show-leak-kinds=all ./ircserv 6667 pass
```

---

# 15. 실무 Debugging Flow

예제 문제:

```cpp
void Server::acceptClient(int fd)
{
    User* user = new User(fd);

    if (!isAllowed(fd))
        return;

    _users[fd] = user;
}
```

---

## 15.1 Symptom

```text
Valgrind에서 User allocation이 definitely lost로 보고된다.
특히 접속 실패한 client에서 leak이 발생한다.
```

---

## 15.2 Repro

재현 시나리오:

```text
1. 서버 실행
2. 허용되지 않은 client fd 또는 잘못된 password로 접속
3. 서버 종료
4. Valgrind report 확인
```

---

## 15.3 Hypothesis

```text
isAllowed(fd)가 false일 때 user를 delete하지 않고 return한다.
```

---

## 15.4 Evidence

Valgrind report:

```text
X bytes in 1 blocks are definitely lost
    at operator new
    by Server::acceptClient(int) (Server.cpp:42)
```

코드 확인:

```cpp
User* user = new User(fd);

if (!isAllowed(fd))
    return;
```

---

## 15.5 Root Cause

```text
allocation 이후 ownership이 _users map으로 이동하기 전에 early return이 발생했다.
그 결과 user pointer를 잃어버렸다.
```

---

## 15.6 Fix

```cpp
void Server::acceptClient(int fd)
{
    User* user = new User(fd);

    if (!isAllowed(fd))
    {
        delete user;
        return;
    }

    _users[fd] = user;
}
```

더 나은 구조:

```cpp
void Server::acceptClient(int fd)
{
    if (!isAllowed(fd))
        return;

    User* user = new User(fd);
    _users[fd] = user;
}
```

이게 더 좋다.

왜냐하면 allocation을 필요한 시점 이후로 미뤘기 때문이다.

```text
검증 먼저
allocation 나중
```

---

## 15.7 Prevention

```text
1. new를 가능한 늦게 한다.
2. new 직후 early return이 있는지 확인한다.
3. ownership transfer 전 실패 path에는 반드시 delete를 둔다.
4. raw pointer를 container에 넣는 순간부터 owner를 명확히 한다.
5. Valgrind로 실패 시나리오를 따로 테스트한다.
```

---

# 16. 흔한 오해

## 오해 1. “`erase()`는 pointer가 가리키는 객체도 지운다”

아니다.

```cpp
std::map<int, User*> users;
users[1] = new User(1);

users.erase(1);
```

`erase()`는 map entry를 지운다.
`User` 객체는 delete하지 않는다.

올바른 순서:

```cpp
delete users[1];
users.erase(1);
```

더 안전한 코드:

```cpp
std::map<int, User*>::iterator it = users.find(1);
if (it != users.end())
{
    delete it->second;
    users.erase(it);
}
```

---

## 오해 2. “`clear()` 하면 다 정리된다”

부분적으로만 맞다.

```cpp
std::vector<User*> users;
users.push_back(new User(1));

users.clear();
```

`vector` 내부의 pointer 값은 제거된다.
하지만 `User` 객체는 delete되지 않는다.

---

## 오해 3. “프로그램 종료 시 OS가 회수하니 leak은 무시해도 된다”

짧은 CLI 프로그램에서는 심각도가 낮을 수 있다.

하지만 서버에서는 다르다.

```text
IRC server
web server
game server
daemon
database
```

이런 프로그램은 오래 실행된다.
종료 시 OS 회수를 기대하면 안 된다.

---

## 오해 4. “Valgrind에서 still reachable이면 항상 안전하다”

아니다.

`still reachable`은 “접근 경로가 있다”는 뜻이지, “설계가 좋다”는 뜻은 아니다.

특히 42 과제에서는 평가자가 leak summary를 엄격하게 볼 수 있다.
가능하면 종료 시점에 정리하는 것이 좋다.

---

## 오해 5. “RAII를 쓰면 Rule of Three는 신경 안 써도 된다”

아니다.

RAII class가 raw resource를 직접 소유하면 복사 문제가 생길 수 있다.

```cpp
class Buffer
{
    char* _data;
public:
    Buffer() : _data(new char[100]) {}
    ~Buffer() { delete[] _data; }
};
```

이 class는 복사되면 double free 위험이 있다.
C++98에서는 Rule of Three를 반드시 고려해야 한다.

---

# 17. 확인 문제

## 문제 1

```cpp
void f()
{
    int* p = new int(42);
}
```

질문:

```text
1. p는 어디에 저장되는가?
2. new int(42)는 어디에 저장되는가?
3. f()가 끝난 뒤 무엇이 사라지는가?
4. 무엇이 leak되는가?
```

---

## 문제 2

```cpp
std::vector<User*> users;
users.push_back(new User(1));
users.clear();
```

질문:

```text
1. vector 내부 pointer 값은 어떻게 되는가?
2. User 객체는 delete되는가?
3. 올바른 정리 코드는 무엇인가?
```

---

## 문제 3

```cpp
struct Node
{
    int* data;
};

Node* n = new Node;
n->data = new int(42);
n = NULL;
```

질문:

```text
1. definitely lost는 무엇인가?
2. indirectly lost는 무엇인가?
3. 왜 data도 잃어버리는가?
```

---

## 문제 4

```cpp
class Buffer
{
private:
    char* _data;

public:
    Buffer() : _data(new char[100]) {}
    ~Buffer() { delete[] _data; }
};
```

질문:

```text
1. 이 class는 RAII인가?
2. 어떤 문제가 남아 있는가?
3. C++98에서 어떻게 막을 수 있는가?
```

---

# 18. 실습 과제

## 실습 1. definitely lost 확인

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
c++ -Wall -Wextra -g -O0 leak1.cpp -o leak1
```

실행:

```bash
valgrind --leak-check=full --show-leak-kinds=all ./leak1
```

확인:

```text
1. definitely lost가 나오는가?
2. 몇 bytes인가?
3. allocation site는 어디인가?
```

---

## 실습 2. indirectly lost 확인

파일:

```cpp
struct Node
{
    int* data;
};

int main()
{
    Node* n = new Node;
    n->data = new int(42);

    n = 0;
    return 0;
}
```

실행:

```bash
c++ -Wall -Wextra -g -O0 leak2.cpp -o leak2
valgrind --leak-check=full --show-leak-kinds=all ./leak2
```

확인:

```text
1. Node는 어떤 leak kind로 나오는가?
2. data는 어떤 leak kind로 나오는가?
3. n을 잃어버리면 왜 data도 잃어버리는가?
```

---

## 실습 3. vector pointer leak

파일:

```cpp
#include <vector>

class User
{
public:
    int fd;
    User(int f) : fd(f) {}
};

int main()
{
    std::vector<User*> users;

    users.push_back(new User(1));
    users.push_back(new User(2));

    users.clear();

    return 0;
}
```

실행:

```bash
c++ -Wall -Wextra -g -O0 vector_leak.cpp -o vector_leak
valgrind --leak-check=full --show-leak-kinds=all ./vector_leak
```

수정:

```cpp
for (size_t i = 0; i < users.size(); ++i)
    delete users[i];

users.clear();
```

확인:

```text
1. 수정 전 definitely lost가 나오는가?
2. 수정 후 leak summary가 어떻게 바뀌는가?
```

---

## 실습 4. ft_irc ownership audit

프로젝트에서 다음 container를 찾아라.

```text
std::map<int, User*>
std::vector<User*>
std::map<std::string, Channel*>
std::vector<Channel*>
```

각각에 대해 표를 작성해라.

| Container       | 저장 타입      | Owns? | Delete 위치                       | Observer 제거 필요? |
| --------------- | ---------- | ----: | ------------------------------- | --------------- |
| `_users`        | `User*`    |   Yes | `disconnect`, `Server::~Server` | Yes             |
| `_channelUsers` | `User*`    |    No | 없음                              | Yes             |
| `_channels`     | `Channel*` |   Yes | `Server::~Server`               | 상황에 따라          |

그리고 코드에 주석을 달아라.

```cpp
// owns User*: Server is responsible for delete
std::map<int, User*> _users;

// non-owning User*: Channel must not delete User
std::vector<User*> _members;
```

---

# 19. 핵심 정리

## 1. Leak은 ownership path가 끊긴 상태다

```text
heap object는 존재하지만,
그 객체를 delete할 수 있는 유효한 pointer 경로가 사라진 상태
```

---

## 2. Valgrind leak 종류

| 종류              | 의미                        | 심각도      |
| --------------- | ------------------------- | -------- |
| definitely lost | 명확히 잃어버림                  | 매우 높음    |
| indirectly lost | 잃어버린 객체가 소유하던 하위 객체도 잃어버림 | 높음       |
| possibly lost   | 내부 pointer만 남아 있어 애매함     | 점검 필요    |
| still reachable | pointer 경로는 남아 있음         | 설계 확인 필요 |

---

## 3. `erase()`와 `clear()`는 raw pointer 객체를 delete하지 않는다

```cpp
std::vector<User*> users;
users.clear();
```

이것은 `User`를 delete하지 않는다.

직접 해야 한다.

```cpp
for (size_t i = 0; i < users.size(); ++i)
    delete users[i];

users.clear();
```

---

## 4. Destructor는 마지막 방어선이다

```cpp
Server::~Server()
{
    clearUsers();
    clearChannels();
}
```

하지만 장기 실행 서버에서는 disconnect path에서도 반드시 정리해야 한다.

---

## 5. C++98에서는 Rule of Three를 조심해야 한다

raw resource를 소유하는 class는 다음을 고려해야 한다.

```text
1. destructor
2. copy constructor
3. copy assignment operator
```

복사가 필요 없으면 private으로 막아라.

```cpp
private:
    Buffer(const Buffer&);
    Buffer& operator=(const Buffer&);
```

---

## 6. Leak debugging 흐름

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

Leak에서는 특히 다음 세 지점을 추적한다.

```text
allocation site
    ↓
ownership transfer
    ↓
deallocation site
```

---
