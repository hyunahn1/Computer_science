# Lecture 2. Logical Operators

## 1. 핵심 질문

> 명제 여러 개를 어떻게 결합해서 더 복잡한 명제를 만들 수 있는가?

Lecture 1에서 우리는 명제를 다음처럼 정의했다.

```text
명제 = 참 또는 거짓이 명확한 선언문
```

이제 명제들을 서로 연결한다.

예를 들어:

```text
A = "비가 온다"
B = "바람이 분다"
```

그러면 다음처럼 새로운 명제를 만들 수 있다.

```text
A ∧ B  = 비가 오고 바람이 분다
A ∨ B  = 비가 오거나 바람이 분다
¬A     = 비가 오지 않는다
A ⊕ B  = 비가 오거나 바람이 불지만, 둘 다는 아니다
```

이런 연산자를 **logical operator**, 즉 논리 연산자라고 한다.

---

# 2. 형식적 정의

## 2.1 AND, Conjunction

`A ∧ B`는 **A와 B가 모두 참일 때만 참**이다.

```text
A ∧ B
```

읽는 법:

```text
A and B
A 그리고 B
A이면서 B
```

형식적 정의:

| A     | B     | A ∧ B |
| ----- | ----- | ----- |
| True  | True  | True  |
| True  | False | False |
| False | True  | False |
| False | False | False |

즉:

```text
A ∧ B is true iff A is true and B is true.
```

`iff`는 **if and only if**, 즉 “필요충분조건”이라는 뜻이다.

---

## 2.2 OR, Disjunction

`A ∨ B`는 **A와 B 중 적어도 하나가 참이면 참**이다.

```text
A ∨ B
```

읽는 법:

```text
A or B
A 또는 B
A이거나 B
```

형식적 정의:

| A     | B     | A ∨ B |
| ----- | ----- | ----- |
| True  | True  | True  |
| True  | False | True  |
| False | True  | True  |
| False | False | False |

중요하다.

수학적 논리에서 `OR`는 기본적으로 **inclusive OR**다.

즉:

```text
A 또는 B
```

라고 했을 때, 둘 다 참인 경우도 포함한다.

---

## 2.3 NOT, Negation

`¬A`는 A의 truth value를 반대로 바꾼다.

```text
¬A
```

읽는 법:

```text
not A
A가 아니다
A의 부정
```

형식적 정의:

| A     | ¬A    |
| ----- | ----- |
| True  | False |
| False | True  |

---

## 2.4 XOR, Exclusive OR

`A ⊕ B`는 **A와 B 중 정확히 하나만 참일 때 참**이다.

```text
A ⊕ B
```

읽는 법:

```text
A xor B
A exclusive-or B
A 또는 B이지만 둘 다는 아님
```

형식적 정의:

| A     | B     | A ⊕ B |
| ----- | ----- | ----- |
| True  | True  | False |
| True  | False | True  |
| False | True  | True  |
| False | False | False |

XOR는 다음처럼 정의할 수 있다.

```text
A ⊕ B = (A ∨ B) ∧ ¬(A ∧ B)
```

뜻은 이렇다.

```text
A 또는 B는 참이어야 한다.
하지만 A와 B가 동시에 참이면 안 된다.
```

---

# 3. 직관적 설명

## 3.1 AND의 직관

```text
A ∧ B
```

는 조건이 모두 통과해야 한다.

프로그래밍으로 보면:

```cpp
if (isLoggedIn && hasPermission) {
    accessFile();
}
```

여기서 파일에 접근하려면:

```text
A = 로그인되어 있다
B = 권한이 있다
```

둘 다 참이어야 한다.

하나라도 거짓이면 접근하면 안 된다.

---

## 3.2 OR의 직관

```text
A ∨ B
```

는 여러 조건 중 하나만 통과해도 된다.

```cpp
if (isAdmin || isOwner) {
    editPost();
}
```

여기서 글을 수정하려면:

```text
A = 관리자다
B = 글 작성자다
```

둘 중 하나만 참이어도 된다.

그리고 둘 다 참이어도 된다.

관리자이면서 작성자인 경우도 수정 가능하다.

---

## 3.3 NOT의 직관

```text
¬A
```

는 조건을 뒤집는다.

```cpp
if (!isLoggedIn) {
    redirectToLogin();
}
```

```text
A = 로그인되어 있다
¬A = 로그인되어 있지 않다
```

---

## 3.4 XOR의 직관

```text
A ⊕ B
```

는 둘 중 정확히 하나만 허용한다.

예를 들어 어떤 프로그램에서 사용자가 로그인할 수 있는 방식이 둘 중 하나라고 하자.

```text
A = password login을 사용한다
B = OAuth login을 사용한다
```

만약 시스템 정책이 “정확히 하나의 방식만 사용해야 한다”라면:

```text
A ⊕ B
```

이다.

| Password | OAuth | 허용? |
| -------- | ----- | --- |
| 사용       | 사용    | X   |
| 사용       | 미사용   | O   |
| 미사용      | 사용    | O   |
| 미사용      | 미사용   | X   |

---

# 4. 왜 필요한지

논리 연산자는 거의 모든 CS 영역의 기초다.

## 4.1 조건문

```cpp
if ((age >= 18 && hasID) || isAdmin) {
    enter();
}
```

이 코드는 논리식이다.

```text
((A ∧ B) ∨ C)
```

---

## 4.2 Boolean expression simplification

복잡한 조건을 단순화할 때 필요하다.

예:

```cpp
if (!(isAdmin && isActive)) {
    deny();
}
```

나중에 De Morgan’s Laws를 배우면 다음처럼 바꿀 수 있다.

```cpp
if (!isAdmin || !isActive) {
    deny();
}
```

---

## 4.3 Circuit logic

디지털 회로도 논리 연산자로 구성된다.

| 논리 연산   | 회로 게이트   |
| ------- | -------- |
| `A ∧ B` | AND gate |
| `A ∨ B` | OR gate  |
| `¬A`    | NOT gate |
| `A ⊕ B` | XOR gate |

CPU 내부의 연산도 결국 논리 게이트의 조합이다.

---

## 4.4 Algorithm correctness

알고리즘의 조건을 증명할 때도 논리 연산자가 필요하다.

예:

```text
loop invariant:
0 ≤ i ≤ n ∧ sum = a[0] + a[1] + ... + a[i-1]
```

여기서 `∧`는 두 조건이 동시에 유지되어야 한다는 뜻이다.

---

# 5. Truth table / 단계별 계산

이제 복합 명제를 계산해보자.

## 예제

```text
(A ∨ B) ∧ ¬A
```

이 논리식의 truth table을 만든다.

### Step 1. 가능한 A, B 조합 나열

변수가 2개이므로 경우의 수는:

```text
2² = 4
```

| A     | B     |
| ----- | ----- |
| True  | True  |
| True  | False |
| False | True  |
| False | False |

---

### Step 2. `A ∨ B` 계산

| A     | B     | A ∨ B |
| ----- | ----- | ----- |
| True  | True  | True  |
| True  | False | True  |
| False | True  | True  |
| False | False | False |

---

### Step 3. `¬A` 계산

| A     | B     | A ∨ B | ¬A    |
| ----- | ----- | ----- | ----- |
| True  | True  | True  | False |
| True  | False | True  | False |
| False | True  | True  | True  |
| False | False | False | True  |

---

### Step 4. `(A ∨ B) ∧ ¬A` 계산

| A     | B     | A ∨ B | ¬A    | `(A ∨ B) ∧ ¬A` |
| ----- | ----- | ----- | ----- | -------------- |
| True  | True  | True  | False | False          |
| True  | False | True  | False | False          |
| False | True  | True  | True  | True           |
| False | False | False | True  | False          |

최종 결과:

```text
(A ∨ B) ∧ ¬A
```

는 오직 다음 경우에만 참이다.

```text
A = False, B = True
```

직관적으로 해석하면:

```text
A 또는 B 중 하나는 참이어야 하는데, A는 거짓이어야 한다.
그러면 결국 B만 참이어야 한다.
```

즉 이 식은 사실상:

```text
¬A ∧ B
```

와 같다.

---

# 6. 프로그래밍 예시

## 6.1 AND

```cpp
bool canDeleteFile(bool isAdmin, bool fileExists) {
    return isAdmin && fileExists;
}
```

논리식:

```text
A = isAdmin
B = fileExists

A ∧ B
```

파일 삭제 가능 조건:

| isAdmin | fileExists | canDeleteFile |
| ------- | ---------- | ------------- |
| true    | true       | true          |
| true    | false      | false         |
| false   | true       | false         |
| false   | false      | false         |

---

## 6.2 OR

```cpp
bool canEditPost(bool isAdmin, bool isOwner) {
    return isAdmin || isOwner;
}
```

논리식:

```text
A ∨ B
```

| isAdmin | isOwner | canEditPost |
| ------- | ------- | ----------- |
| true    | true    | true        |
| true    | false   | true        |
| false   | true    | true        |
| false   | false   | false       |

---

## 6.3 NOT

```cpp
bool shouldRedirect(bool isLoggedIn) {
    return !isLoggedIn;
}
```

논리식:

```text
¬A
```

| isLoggedIn | shouldRedirect |
| ---------- | -------------- |
| true       | false          |
| false      | true           |

---

## 6.4 XOR

C++에는 논리 XOR 전용 연산자가 따로 자주 쓰이지 않는다.
`bool` 값에 대해서는 다음처럼 표현할 수 있다.

```cpp
bool exactlyOne(bool a, bool b) {
    return (a || b) && !(a && b);
}
```

또는 `bool`에 대해서는 `!=`를 사용할 수도 있다.

```cpp
bool exactlyOne(bool a, bool b) {
    return a != b;
}
```

왜냐하면 bool에서 `a != b`는 두 값이 다를 때 true이기 때문이다.

| a     | b     | a != b | a ⊕ b |
| ----- | ----- | ------ | ----- |
| true  | true  | false  | false |
| true  | false | true   | true  |
| false | true  | true   | true  |
| false | false | false  | false |

따라서 bool에서는:

```cpp
a != b
```

가 XOR처럼 작동한다.

---

# 7. C/C++의 `&&`, `||`, `!`와 수학적 논리의 관계

| 수학 논리   | C/C++    |   |    |
| ------- | -------- | - | -- |
| `A ∧ B` | `A && B` |   |    |
| `A ∨ B` | `A       |   | B` |
| `¬A`    | `!A`     |   |    |

하지만 중요한 차이가 있다.

## 수학적 논리

수학에서는 보통 `A ∧ B`를 평가할 때 A와 B의 truth value가 이미 주어졌다고 본다.

```text
A ∧ B
```

는 단순히 두 truth value의 조합이다.

---

## C/C++ 논리 연산

C/C++에서는 expression을 **실제로 실행하면서 평가**한다.

이때 `&&`, `||`에는 **short-circuit evaluation**이 있다.

---

# 8. Short-circuit evaluation

## 8.1 `&&`의 short-circuit

```cpp
if (ptr != nullptr && ptr->value > 0) {
    // ...
}
```

여기서 `ptr != nullptr`가 false이면?

```cpp
ptr->value > 0
```

는 아예 평가하지 않는다.

왜냐하면:

```text
False && anything = False
```

이므로 뒤를 볼 필요가 없다.

그래서 이 코드는 안전하다.

```cpp
ptr != nullptr && ptr->value > 0
```

하지만 순서를 바꾸면 위험하다.

```cpp
if (ptr->value > 0 && ptr != nullptr) {
    // 위험
}
```

`ptr`이 null이면 첫 번째 조건에서 바로 잘못된 접근이 발생할 수 있다.

---

## 8.2 `||`의 short-circuit

```cpp
if (isAdmin || hasPermission(user)) {
    allow();
}
```

여기서 `isAdmin`이 true이면?

```cpp
hasPermission(user)
```

는 호출하지 않는다.

왜냐하면:

```text
True || anything = True
```

이기 때문이다.

---

## 8.3 수학 논리와 C/C++의 차이

수학에서는:

```text
A ∨ B
```

가 단순히 truth value의 결합이다.

프로그래밍에서는:

```cpp
A || B
```

가 다음까지 포함한다.

```text
1. A를 먼저 평가한다.
2. 필요하면 B를 평가한다.
3. 필요 없으면 B는 실행하지 않는다.
```

따라서 C/C++ 조건문은 논리뿐 아니라 **평가 순서와 side effect**도 고려해야 한다.

---

# 9. Side effect가 있는 조건문

다음 코드를 보자.

```cpp
int x = 0;

if (true || ++x > 0) {
    // ...
}

std::cout << x << "\n";
```

출력은?

```text
0
```

왜냐하면 `true || ...`에서 앞이 이미 true이므로 `++x > 0`은 실행되지 않는다.

반대로:

```cpp
int x = 0;

if (false || ++x > 0) {
    // ...
}

std::cout << x << "\n";
```

출력은?

```text
1
```

이번에는 앞이 false라서 뒤 조건을 평가해야 한다.

이것이 수학적 논리와 프로그래밍 조건문의 중요한 차이다.

---

# 10. 실무적 의미

## 10.1 Null check

```cpp
if (p != nullptr && p->isReady()) {
    run();
}
```

이 패턴은 매우 중요하다.

논리적으로는:

```text
P = pointer is not null
Q = object is ready

P ∧ Q
```

프로그래밍적으로는:

```text
P가 먼저 검사되어야 Q가 안전하게 평가된다.
```

---

## 10.2 Expensive function call 방지

```cpp
if (cached || expensiveCheck()) {
    useResult();
}
```

`cached`가 true이면 `expensiveCheck()`를 호출하지 않는다.

성능 최적화에 사용된다.

---

## 10.3 Guard condition

```cpp
if (!isValidInput(input)) {
    return false;
}

process(input);
```

이것은 논리적으로:

```text
valid input이 아니면 즉시 종료
```

이다.

복잡한 조건을 중첩하지 않고 안전하게 처리하는 방식이다.

---

## 10.4 Security condition

```cpp
if (isAuthenticated && hasRole("admin")) {
    deleteUser();
}
```

이 조건에서 `&&`를 `||`로 잘못 쓰면 치명적인 보안 버그가 된다.

```cpp
if (isAuthenticated || hasRole("admin")) {
    deleteUser();
}
```

이 경우 로그인만 해도 삭제 가능하거나, role 체크만 통과해도 삭제 가능해질 수 있다.

논리 연산자의 의미를 잘못 이해하면 실무에서는 권한 우회 취약점이 된다.

---

# 11. 흔한 오해

## 오해 1. OR는 둘 중 하나만 참이라는 뜻이다

수학 논리에서 기본 OR는 **inclusive OR**다.

즉:

```text
A ∨ B
```

는 다음 세 경우에 참이다.

| A     | B     |
| ----- | ----- |
| True  | True  |
| True  | False |
| False | True  |

“정확히 하나만 참”을 원하면 XOR를 써야 한다.

```text
A ⊕ B
```

---

## 오해 2. `!(A && B)`는 `!A && !B`다

아니다.

정확한 법칙은:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

C/C++로는:

```cpp
!(A && B) == (!A || !B)
```

이것은 Lecture 4에서 엄밀히 증명한다.

---

## 오해 3. C/C++의 `&&`와 수학의 `∧`는 완전히 같다

결과 truth value만 보면 비슷하다.

하지만 C/C++의 `&&`는 평가 순서와 short-circuit을 가진다.

```cpp
A && B
```

는 반드시 `A`를 먼저 평가하고, 필요할 때만 `B`를 평가한다.

수학의 `A ∧ B`에는 이런 실행 개념이 없다.

---

## 오해 4. XOR는 그냥 OR와 같다

아니다.

가장 큰 차이는 둘 다 참일 때다.

| A    | B    | A ∨ B | A ⊕ B |
| ---- | ---- | ----- | ----- |
| True | True | True  | False |

OR는 둘 다 참이면 true.
XOR는 둘 다 참이면 false.

---

# 12. 반례

## 반례 1. OR를 “둘 중 하나만”이라고 착각하는 경우

다음 정책을 생각해보자.

```text
관리자 또는 작성자는 글을 수정할 수 있다.
```

이것은 보통:

```text
isAdmin ∨ isOwner
```

이다.

관리자이면서 작성자인 사람은 수정할 수 있어야 한다.

따라서 이 경우 OR는 inclusive OR가 맞다.

만약 XOR로 쓰면:

```text
isAdmin ⊕ isOwner
```

관리자이면서 작성자인 사람은 오히려 수정할 수 없게 된다.

이것은 명백히 잘못된 정책 구현이다.

---

## 반례 2. `&&`와 `||`를 바꿔 쓰는 경우

```cpp
if (isLoggedIn || hasPermission) {
    accessSecretPage();
}
```

이 조건은 위험하다.

의도는 아마:

```cpp
if (isLoggedIn && hasPermission) {
    accessSecretPage();
}
```

였을 수 있다.

차이를 표로 보자.

| isLoggedIn | hasPermission | `&&` 결과 | `||` 결과 |
|---|---|---|---|
| true | true | true | true |
| true | false | false | true |
| false | true | false | true |
| false | false | false | false |

`||`를 쓰면 네 경우 중 세 경우가 허용된다.
`&&`를 쓰면 둘 다 true인 한 경우만 허용된다.

보안 조건에서는 이 차이가 매우 크다.

---

## 반례 3. Short-circuit을 무시한 경우

```cpp
if (p->ready() && p != nullptr) {
    run();
}
```

이 코드는 논리적으로는 다음과 비슷해 보일 수 있다.

```text
P = p is not null
Q = p is ready
```

하지만 프로그래밍에서는 순서가 중요하다.

`p`가 null이면 `p->ready()`에서 문제가 발생한다.

올바른 순서:

```cpp
if (p != nullptr && p->ready()) {
    run();
}
```

---

# 13. 확인 문제

다음 조건이 언제 true인지 설명하라.

## 문제 1

```text
A ∧ B
```

---

## 문제 2

```text
A ∨ B
```

---

## 문제 3

```text
¬A
```

---

## 문제 4

```text
A ⊕ B
```

---

## 문제 5

다음 C++ 조건문을 논리식으로 바꿔라.

```cpp
if (isLoggedIn && (isAdmin || isOwner)) {
    editPost();
}
```

힌트:

```text
A = isLoggedIn
B = isAdmin
C = isOwner
```

---

## 문제 6

다음 두 조건은 같은가?

```cpp
!(A && B)
```

```cpp
!A && !B
```

같지 않다면 반례를 하나 들어라.

---

## 문제 7

다음 코드의 출력은?

```cpp
int x = 0;

if (false && ++x > 0) {
    std::cout << "inside\n";
}

std::cout << x << "\n";
```

---

# 14. 실습 과제

## 과제 1. Truth table 작성

다음 논리식의 truth table을 직접 만들어라.

```text
(A ∧ B) ∨ ¬A
```

작성 순서:

```text
1. A, B 모든 조합 나열
2. A ∧ B 계산
3. ¬A 계산
4. (A ∧ B) ∨ ¬A 계산
```

---

## 과제 2. C++ 조건 분석

다음 코드를 논리식으로 바꿔라.

```cpp
if ((hasToken && tokenNotExpired) || isAdmin) {
    allowAccess();
}
```

다음 형태로 작성하면 된다.

```text
A = ?
B = ?
C = ?

전체 조건 = ?
```

---

## 과제 3. XOR 구현

다음 조건을 C++로 구현하라.

```text
사용자는 email login 또는 OAuth login 중 정확히 하나만 사용해야 한다.
```

변수:

```cpp
bool usesEmailLogin;
bool usesOAuthLogin;
```

두 방식으로 구현하라.

```cpp
// 방식 1: ||, &&, ! 사용
// 방식 2: != 사용
```

---

# 15. 핵심 정리

이번 Lecture의 핵심은 다음이다.

| 연산자 | 이름                | 의미             | C/C++         |   |   |
| --- | ----------------- | -------------- | ------------- | - | - |
| `∧` | AND, conjunction  | 둘 다 참이면 참      | `&&`          |   |   |
| `∨` | OR, disjunction   | 적어도 하나가 참이면 참  | `             |   | ` |
| `¬` | NOT, negation     | truth value 반전 | `!`           |   |   |
| `⊕` | XOR, exclusive OR | 정확히 하나만 참이면 참  | `!=` 또는 직접 구현 |   |   |

가장 중요한 구분:

```text
OR = 적어도 하나
XOR = 정확히 하나
```

그리고 C/C++에서는 수학 논리와 달리 다음도 중요하다.

```text
short-circuit evaluation
평가 순서
side effect
null check 순서
expensive function 호출 여부
```

즉, 수학 논리에서는 truth value가 중심이고, 프로그래밍에서는 truth value뿐 아니라 **실행 과정**도 중요하다.
