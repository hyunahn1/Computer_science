# Debugging이란 무엇인가

## 핵심 질문

이번 강의의 핵심 질문은 다음이다.

문제가 생겼을 때, 우리는 무엇을 먼저 해야 하는가?

우리는 디버깅을 왜하는 것일까?

컴퓨터 속에서 생각을 해보면 한정된 자원과 시간 속에서 최선의 효율을 내어야한다.
그런 것들을 생각해보았을 때, 디버깅은 그 자원과 시간을 아낄 수 있는 선택인 것이다.
디버깅은 감으로 코드를 고치는 행위가 아니다.

디버깅은 그렇게 다음 과정을 거친다.

관찰 → 재현 → 가설 → 실험 → 증거 → 원인 → 수정 → 예방

## 개념

### Debugging의 목적

러를 없애는 것만을 목적으로 하진 않는다. 

* 프로그램이 왜 기대와 다르게 행동하는지 설명 가능한 원인을 찾고, 그 원인을 제거하며, 같은 문제가 다시 생기지 않도록 만드는 것.

여기서 가장 중요한 키워드는 설명 가능한 원인이다. 

C++에 

std::vector<int> v;
std::cout << v[0] << std::endl;

이런 코드가 있다고 하자. 그때 실행을 하려고 봤더니 crash가 난다. 
그러면 우리에게는 두가지 선택지가 있다.
1. 추측하고 추측가는 부분을 고치고 테스트해본다.

2. 직접 파일들을 하나하나 뜯어보면서 뭐가 문제인지 하나씩 찾아보고 고쳐서 테스트해본다.

예를 들어

v는 empty vector다.
v[0]은 bounds check를 하지 않는다.
존재하지 않는 메모리를 읽는다.
undefined behavior다.
따라서 crash가 날 수도 있고, 이상한 값이 나올 수도 있다.
root cause는 empty vector에 대한 인덱스 접근이다.

이런식으로 말이다.
즉, 디버깅은 GDB 돌리기 LLDB 돌리기를 넘어서서 디버깅은 현상을 설명하는 모델을 세우는 작업인 것이다.

## Bug, Defect, Failure의 차이

이 세 단어는 비슷해 보이지만, 구분하면 디버깅 사고가 훨씬 정밀해질 수 있다.

| 용어      | 의미                 | 예시                              |
| ------- | ------------------ | ------------------------------- |
| Defect  | 코드나 설계 안에 숨어 있는 결함 | `vector[0]`을 empty 상태에서 접근하는 코드 |
| Bug     | 일반적으로 결함을 부르는 말    | “여기 버그 있다”                      |
| Failure | 실제 실행 중 관찰된 실패     | 프로그램이 segmentation fault로 종료됨   |

흐름은 보통 이렇게 된다.

Defect가 코드 안에 존재함
        ↓
특정 입력/환경에서 실행됨
        ↓
Failure로 드러남

[코드 안의 결함]
      defect
        │
        │ 특정 입력, 특정 환경, 특정 순서
        ▼
[실행 중 실패]
      failure
        │
        ▼
[사용자가 보는 현상]
      symptom

예시를 들어보자.

int divide(int a, int b)
{
    return a / b;
}

이런 C++ 코드가 있다고 했을 때, 여기서 b == 0을 처리하지 않는 것은 defect다.

하지만 프로그램을 항상 b = 2로만 실행하면 실패하지 않는다.

divide(10, 2); // 정상

그러다가 어느 날 이런 입력이 들어온다.

divide(10, 0); // failure 발생 가능

그러다보니 프로그램이 갑자기 죽었다.

이게 symptom, 즉 증상이다.

## Symptom, Cause, Fix의 차이

### Symptom

Symptom은 겉으로 보이는 현상이다. 

서버가 죽었다.
테스트가 실패했다.
출력이 비어 있다.
채팅 메시지가 상대방에게 안 간다.
프로그램이 무한 루프에 빠진다.
Git push가 rejected 된다.

증상은 중요하지만, 증상이 곧 원인은 아니다.

### Cause

Cause는 그 증상을 만든 실제 이유다.

서버가 죽었다.

이 Symptom의 cause는 여러가지가 될 수 있다.

포트가 이미 사용 중이다.
bind() 실패 후에도 listen()을 호출했다.
NULL pointer를 역참조했다.
vector index가 잘못됐다.
SIGPIPE가 발생했다.
파일 descriptor가 닫힌 뒤 재사용됐다.

같은 증상이라도 원인은 완전히 다를 수 있다.

### Fix

Fix는 원인을 제거하는 수정이다.

if (b == 0)
{
    std::cerr << "division by zero" << std::endl;
    return 0;
}
return a / b;

하지만 여기서 중요한 점이 있다.

Fix는 root cause를 겨냥해야 한다.

## Root Cause란 무엇인가

Root cause는 “가장 깊은 원인”이라고 대충 말하면 부족하다.

Root cause는 그 문제를 다시 발생하지 않게 만들기 위해 반드시 고쳐야 하는 근본 조건이다.

다시 예를 들어보자.

IRC 서버에서 클라이언트가 메시지를 보냈는데 상대방에게 broadcast가 안 된다.

symptom

클라이언트 A가 PRIVMSG를 보냈지만 클라이언트 B가 받지 못한다.

abailable cause

Parser가 PRIVMSG를 잘못 파싱했다.
Channel에 user가 등록되지 않았다.
broadcast 함수가 자기 자신에게만 보냈다.
outbox queue에 메시지는 들어갔지만 POLLOUT이 켜지지 않았다.
send/write가 호출되지 않았다.
write는 됐지만 partial write 처리를 안 했다.

여기서 실제 원인이 다음이라고 해보자

std::queue<std::string> User::getOutbox()
{
    return _outbox;
}

그리고 flush 코드가 이렇게 되어 있다.

std::queue<std::string> q = user.getOutbox();
q.pop();

이 경우 getOutbox()가 복사본을 반환한다.

따라서 pop()을 해도 실제 user의 _outbox는 변하지 않는다.

root cause:

User의 outbox를 수정해야 하는 코드가 실제 queue가 아니라 복사본을 수정하고 있었다.

fix:

std::queue<std::string>& User::getOutboxRef()
{
    return _outbox;
}

prevention:

상태를 수정하는 getter는 reference를 반환한다는 naming convention을 만든다.
예: getOutboxRef()
복사 반환 getter와 수정용 getter를 구분한다.
flush 동작에 대한 단위 테스트를 만든다.

여기서 중요한 것은 root cause가 단순히 이것이 아니라는 점이다.

메시지가 안 갔다. 

이건 그저 증상이다. root cause는 "상태를 변경한다고 생각한 코드가 실제 상태가 아닌 복사본을 변경했다는 것"이다

## 왜 필요한지

디버깅 사고방식이 필요한 이유는 간단하다.

프로그램은 작아 보이지만 실제로는 여러 층으로 이루어져 있다.

사용자 입력
  ↓
shell
  ↓
program arguments
  ↓
parser
  ↓
business logic
  ↓
data structure
  ↓
system call
  ↓
kernel
  ↓
filesystem / network / terminal

문제가 생겼을 때 겉으로는 하나의 증상만 보인다.

안됨.

하지만 실제 원인은 어느 층에도 있을 수 있다.


예를 들어

./server 6667 password

서버가 실행되지 않는다고 할 때, 가능한 원인은 매우 많다.

| 층           | 가능한 원인                 |
| ----------- | ---------------------- |
| Shell       | 인자를 잘못 넘김              |
| Program     | argv parsing 실패        |
| Network     | port already in use    |
| Permission  | 낮은 포트 사용               |
| Code        | bind 실패를 처리하지 않음       |
| OS          | firewall, interface 문제 |
| Test client | nc가 바로 EOF 보냄          |

이렇게 계층별로 다양한 원인이 만들어질 수 있다.그러므로 디버깅은 “코드 보기”가 아니라 전체 시스템의 증거를 좁혀가는 작업인 것이다.

## 내부 원리 / 작동 방식

### 프로그램 실패는 보통 관찰 지점보다 앞에서 발생한다

이는 어떻게 생각하면 쉽게 생각할 수 있는 법칙인데 중요한 원칙이다.

증상이 관찰된 위치는 원인이 발생한 위치보다 뒤일 가능성이 높다.

자 다시 예를 들어보자

int *p = NULL;

// 훨씬 앞쪽 코드
p = get_pointer();

// 훨씬 뒤쪽 코드
*p = 42; // crash

Crash는 *p = 42에서 발생한다.

하지만 진짜 원인은 앞에 있을 수 있다.

p = get_pointer();

get_pointer()가 왜 NULL을 반환했는지 봐야 한다.

즉, crash line은 시체가 발견된 장소라고 표현할 수 있다.

범죄가 일어난 장소가 아닐 수도 있다는 것.

### 디버깅은 탐색 공간을 줄이는 과정이다

처음에는 가능한 원인이 많다.

원인 후보:
- 입력 문제
- 환경 문제
- 빌드 문제
- parser 문제
- 자료구조 문제
- memory 문제
- OS syscall 문제
- network 문제
- race condition 문제

디버깅은 이 후보들을 증거로 제거하는 과정입니다.

가설 A: 입력 문제인가?
증거: 같은 입력에서 항상 실패한다.
결론: 입력과 관련 있음.

가설 B: macOS에서만 실패하는가?
증거: Linux에서도 실패한다.
결론: macOS 전용 문제는 아님.

가설 C: parser 문제인가?
증거: parser output은 정상이다.
결론: parser 이후 단계 문제일 가능성.

이렇게 범위를 좁힌다.

전체 시스템
   ↓
입력 처리 이후
   ↓
Channel broadcast
   ↓
User outbox
   ↓
flush / POLLOUT
   ↓
queue reference 문제

### 디버깅의 기본 루프

전문적인 디버깅 루프는 다음과 같다.

1. Observe
   증상을 정확히 관찰한다.

2. Reproduce
   같은 실패를 다시 만들 수 있게 한다.

3. Hypothesize
   가능한 원인을 세운다.

4. Test
   가설을 검증할 실험을 한다.

5. Narrow
   원인 범위를 줄인다.

6. Fix
   root cause를 수정한다.

7. Verify
   같은 재현 케이스가 통과하는지 확인한다.

8. Prevent
   테스트, 로그, 문서, assert 등으로 재발을 막는다.

이걸 그림으로 보면:

[Symptom]
    ↓
[Reproduce]
    ↓
[Hypothesis]
    ↓
[Evidence]
    ↓
[Root Cause]
    ↓
[Fix]
    ↓
[Verification]
    ↓
[Prevention]

## 쉬운 예시를 한번 들어보자.

### 예시 1: Shell script가 이상하게 실패한다

#!/bin/bash

name=$1

if [ $name = "admin" ]; then
    echo "hello admin"
else
    echo "hello user"
fi

이를 ./hello.sh 실행시켰을 때, 

./hello.sh: line 5: [: =: unary operator expected
hello user

이러한 증상이 나왔다고 생각을 하면?

Symptom

인자를 주지 않았을 때 test 명령에서 unary operator expected가 발생한다.

Cause 후보

$name이 비어 있어서 [ = "admin" ] 형태가 되었을 가능성

Evidence

디버깅 출력 추가:

set -x

+ name=
+ '[' = admin ']'

실제 shell이 실행한 것은 다음과 같다.

[ = admin ]

문법이 깨졌다.

Root cause는 변수를 quote하지 않아 빈 문자열일 때 test 명령의 인자 구조가 깨졌다.

Fix

if [ "$name" = "admin" ]; then
    echo "hello admin"
else
    echo "hello user"
fi

Prevention

set -u
인자 개수 검사
shellcheck 사용
항상 변수는 quote하기

개선된 코드:

#!/bin/bash
set -u

if [ "$#" -lt 1 ]; then
    echo "usage: $0 NAME" >&2
    exit 1
fi

name=$1

if [ "$name" = "admin" ]; then
    echo "hello admin"
else
    echo "hello user"
fi

## 좋은 디버깅 노트 작성법

# Debugging Note

## 1. Symptom
무엇이 관찰되었는가?

## 2. Expected
원래 무엇이 일어나야 하는가?

## 3. Actual
실제로 무엇이 일어났는가?

## 4. Reproduction
어떤 명령, 입력, 환경에서 재현되는가?

## 5. Environment
OS, compiler, shell, dependency, commit hash는 무엇인가?

## 6. Hypotheses
가능한 원인 후보는 무엇인가?

## 7. Evidence
각 가설을 지지하거나 반박하는 증거는 무엇인가?

## 8. Root Cause
최종 원인은 무엇인가?

## 9. Fix
무엇을 수정했는가?

## 10. Verification
수정 후 어떤 명령으로 확인했는가?

## 11. Prevention
다시 발생하지 않게 무엇을 추가했는가?

# Reproduce First

지난 Lecture 1에서 우리는 디버깅의 전체 흐름을 봤다.

이번 Lecture 2의 주제는 두 번째 단계이다.
* 먼저 재현하라.

디버깅에서 가장 위험한 상태는 바로

“가끔 안 돼요.”
“어제는 됐는데 오늘은 안 돼요.”
“내 컴퓨터에서는 되는데 팀원 컴퓨터에서는 안 돼요.”
“고친 것 같은데 다시 깨질 수도 있어요.”

이 상태에서는 원인 분석도, fix 검증도, Git bisect도 어렵다.

## 핵심 질문

* 실패를 다시 만들 수 없는 상태에서, 우리는 정말로 버그를 고칠 수 있는가?

답은 엄밀히 말하면:

운 좋게 고칠 수는 있다.
하지만 전문적으로 고쳤다고 말하기는 어렵다.

왜냐하면 재현이 안 되면 다음을 알 수 없기 때문이다.

1. 내가 본 증상이 진짜 버그인지
2. 어떤 조건에서 발생하는지
3. 내 수정이 실제로 문제를 해결했는지
4. 같은 문제가 다시 생겼는지
5. 다른 사람이 같은 문제를 확인할 수 있는지

* Fix before reproduction is guessing.

## 개념

### Reproduction이란 무엇인가

Reproduction, 즉 재현이란 다음을 의미한다.

* 같은 조건에서 같은 실패를 다시 발생시킬 수 있는 상태를 만드는 것.

좋은 reproduction은 보통 다음 형태를 가진다.

1. 어떤 commit에서
2. 어떤 OS / compiler / dependency로
3. 어떤 명령을 실행했을 때
4. 어떤 입력을 넣으면
5. 어떤 출력 / crash / failure가 발생한다

어려우니 예를 들어보자.

Commit:
abc1234

Environment:
macOS 14.x
clang++ -std=c++98

Command:
./ircserv 6667 pass

Client input:
PASS pass
NICK a
USER a 0 * :A
JOIN #lobby
PRIVMSG #lobby :hello

Actual:
Client B receives nothing.

Expected:
Client B should receive message from Client A.

이 정도가 있으면 팀원이 같은 문제를 따라 할 수 있다.

### “재현 가능하다”의 의미

재현 가능하다는 말은 한 번 다시 됐다가 아닙니다.

좋은 재현은 반복 가능해야 한다.

한 번 실패했다 = observation
여러 번 같은 조건에서 실패한다 = reproducible

예:

for i in $(seq 1 10); do
    ./test_privmsg.sh || echo "failed at $i"
done

결과:

failed at 1
failed at 2
failed at 3
...
failed at 10

이런 경우는 매우 좋은 재현이다 반면에 10번 중 1번만 실패하는 것은 불안정한 재현이다. 
이걸 intermittent bug 또는 flaky failure라고 부른다.

## 왜 필요한지

### 재현은 디버깅의 기준점이다

재현 가능한 실패 케이스가 있으면 디버깅이 과학 실험처럼 됩니다.

현재 상태:
실패함

수정 후:
같은 명령을 실행함

결과:
성공함

즉, 비교가 가능하다.

### 재현이 있어야 가설을 검증할 수 있다

디버깅은 가설 검증이다.

예를 들어 ft_irc에서 이런 증상이 있다고 하자.

가설:

H1. Parser가 PRIVMSG를 잘못 파싱한다.
H2. Channel에 Client B가 없다.
H3. Outbox에는 들어갔지만 flush가 안 된다.
H4. POLLOUT event가 켜지지 않았다.

이 가설들을 검증하려면 같은 실패를 반복해서 만들어야 한다.

같은 입력으로 실패를 재현
  ↓
parser log 확인
  ↓
channel members 확인
  ↓
outbox size 확인
  ↓
poll events 확인

재현이 없으면 매번 다른 상황을 보고 있을 수 있다.

### 재현은 fix 검증의 도구다

재현이 있으면 fix 검증이 명확하다.

./reproduce_bug.sh
echo $?

수정전:

FAIL
exit code 1

수정후:

PASS
exit code 0

이 fix는 최소한 내가 기록한 실패 케이스를 해결했다.

하나의 reproduction을 통과했다고 모든 문제가 해결된 것은 아니다. 하지만 reproduction도 없이 고쳤다고 말하는 것보다는 훨씬 낫다.

## 내부 원리 / 작동 방식

### 프로그램 실행은 조건들의 조합이다

프로그램의 실행 결과는 코드만으로 결정되지 않는다.

다음 요소들이 함께 결과를 만든다.

Program behavior
=
Code
+ Input
+ Arguments
+ Environment variables
+ Config
+ Filesystem state
+ Network state
+ Time
+ Randomness
+ OS
+ Compiler
+ Library versions
+ Hardware
+ Scheduling

                [ source code ]
                      │
                      ▼
[ argv ] → [ process execution ] ← [ env ]
                      ▲
                      │
[ files ] → [ OS / kernel / libc ] ← [ network ]
                      ▲
                      │
             [ compiler / build flags ]

코드는 같은가?
commit hash는 같은가?
build flag는 같은가?
입력은 같은가?
환경 변수는 같은가?
파일 상태는 같은가?
OS와 compiler는 같은가?

### Deterministic bug

Deterministic bug는 같은 조건에서 항상 같은 방식으로 실패하는 버그다.

int divide(int a, int b)
{
    return a / b;
}

int main()
{
    return divide(10, 0);
}

실행하면 거의 항상 실패합니다.

./a.out

이런 종류는 비교적 디버깅하기 쉽습니다.

항상 segmentation fault
항상 wrong output
항상 test failure
항상 compile error

Deterministic bug는 재현이 명확하다.

### Intermittent bug

Intermittent bug는 가끔만 발생하는 버그다.

10번 중 1번 실패
특정 타이밍에만 실패
네트워크 상태에 따라 실패
파일 순서에 따라 실패
thread scheduling에 따라 실패

예를 들어 C/C++에서 초기화되지 않은 변수를 사용하면:

#include <iostream>

int main()
{
    int x;

    if (x == 42)
        std::cout << "lucky" << std::endl;
    else
        std::cout << "normal" << std::endl;

    return 0;
}

x의 값은 초기화되지 않았다.

결과는 컴파일러, 최적화, stack 상태에 따라 달라질 수 있다.

이런 문제는 재현이 어렵다.

### Flaky test

Flaky test는 코드가 바뀌지 않았는데도 어떤 때는 pass, 어떤 때는 fail하는 테스트다.

예:

Run 1: PASS
Run 2: PASS
Run 3: FAIL
Run 4: PASS
Run 5: FAIL

Flaky test의 흔한 원인:

시간 의존성
랜덤 값
네트워크 의존성
외부 API 의존성
파일 순서 의존성
race condition
테스트 간 공유 상태
cleanup 누락

예를 들어 이런 테스트는 위험하다.

#!/bin/bash

./server &
sleep 1
./client_test

sleep 1 동안 서버가 항상 준비된다는 보장이 없다.
느린 환경에서는 서버가 아직 listen 상태가 아닐 수 있다.
빠른 환경에서는 통과하고, 느린 환경에서는 실패한다.

더 좋은 방식은 서버가 준비될 때까지 확인하는 것이다.

#!/bin/bash

./server 6667 pass >server.out 2>server.err &
SERVER_PID=$!

for i in $(seq 1 50); do
    nc -z 127.0.0.1 6667 && break
    sleep 0.1
done

./client_test
STATUS=$?

kill "$SERVER_PID"
exit "$STATUS"

nc -z는 연결 가능 여부를 확인하는 방식, 환경에 따라 옵션 동작이 조금 다를 수 있으므로 macOS/Linux에서 확인이 필요하다.

## 재현 가능한 버그와 재현 어려운 버그

### 재현 가능한 버그

좋은 상황이다.

이 입력을 넣으면 항상 crash 난다.
이 명령을 실행하면 항상 exit code 1이 나온다.
이 commit에서 항상 test가 실패한다.

이 경우는 다음이 명확하다.

입력 파일: invalid_input.txt
명령: ./btc invalid_input.txt
실패: bad input 처리 문제

### 재현 어려운 버그

사용자는 가끔 안 된다고 함
서버 로그에는 명확한 에러가 없음
개발자 컴퓨터에서는 재현 안 됨
CI에서는 가끔 실패함

이때 목표는 바로 고치는 것이 아니다.

목표는 먼저 이것이다.

재현 확률을 높이는 것.

10번 중 1번 실패 → 100번 반복해서 잡는다.
특정 시간에 실패 → 시간을 고정하거나 mock한다.
랜덤하게 실패 → seed를 기록한다.
환경에 따라 실패 → 환경 차이를 비교한다.

## 환경 차이 문제

### 내 컴퓨터에서는 되는데요

개발자 세계에서 유명한 말이 있다.

It works on my machine.

코드만 문제가 아닐 수 있다. 환경 차이가 behavior를 바꾸고 있다.

비교해야 할 것:

| 항목                | 확인 명령                                |
| ----------------- | ------------------------------------ |
| OS                | `uname -a`                           |
| Shell             | `echo "$SHELL"`                      |
| Bash version      | `bash --version`                     |
| Compiler          | `g++ --version`, `clang++ --version` |
| Make version      | `make --version`                     |
| Git commit        | `git rev-parse HEAD`                 |
| Git status        | `git status --short`                 |
| Build flags       | Makefile 확인                          |
| Environment       | `env`                                |
| Locale            | `locale`                             |
| Path              | `echo "$PATH"`                       |
| Open ports        | `lsof -i :PORT`, `ss -ltnp`          |
| File permissions  | `ls -l`                              |
| Current directory | `pwd`                                |

