# Lecture 4. Logical Equivalence와 De Morgan’s Laws

## 1. 핵심 질문

> 두 논리식이 겉모양은 달라도 항상 같은 의미를 가질 수 있는가?

예를 들어 다음 두 조건을 보자.

```cpp
!(A && B)
```

```cpp
!A || !B
```

겉모양은 다르다.
하지만 두 조건은 모든 경우에 같은 결과를 낸다.

수학적으로는 다음과 같이 쓴다.

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

이것이 **logical equivalence**, 즉 논리적 동치다.

이번 강의의 목표는 다음이다.

```text
1. logical equivalence가 무엇인지 이해한다.
2. truth table로 equivalence를 증명한다.
3. De Morgan’s Laws를 정확히 이해한다.
4. C/C++ 조건문 단순화와 연결한다.
5. 흔한 실수인 !(A && B) → !A && !B 오류를 잡는다.
```

---

# 2. 형식적 정의

## 2.1 Logical Equivalence

두 논리식 `P`와 `Q`가 **logically equivalent**하다는 것은 다음 뜻이다.

```text
모든 가능한 truth value 조합에서 P와 Q의 결과가 같다.
```

기호로는 다음처럼 쓴다.

```text
P ≡ Q
```

읽는 법:

```text
P is logically equivalent to Q
P와 Q는 논리적으로 동치다
P와 Q는 항상 같은 truth value를 가진다
```

중요하다.

```text
P ≡ Q
```

는 단순히 “어떤 경우에 우연히 같다”가 아니다.

반드시:

```text
모든 row에서 같다
```

여야 한다.

---

## 2.2 Equality `=`와 Equivalence `≡`의 차이

수학에서 `=`는 보통 값의 같음을 의미한다.

```text
2 + 3 = 5
```

논리에서 `≡`는 두 논리식이 모든 경우에 같은 truth value를 가진다는 뜻이다.

예:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

이 말은 다음 뜻이다.

```text
A, B가 어떤 truth value를 가지든
왼쪽 식과 오른쪽 식의 결과가 항상 같다.
```

---

# 3. 직관적 설명

다음 문장을 보자.

```text
"둘 다 참인 것은 아니다."
```

이 문장은 다음과 같다.

```text
¬(A ∧ B)
```

즉:

```text
A와 B가 동시에 참인 상황은 아니다.
```

그러면 가능한 상황은 무엇인가?

| A | B | “둘 다 참은 아님” |
| - | - | ----------- |
| T | T | 불가능         |
| T | F | 가능          |
| F | T | 가능          |
| F | F | 가능          |

즉, 다음 중 하나다.

```text
A가 거짓이거나, B가 거짓이다.
```

논리식으로는:

```text
¬A ∨ ¬B
```

따라서:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

이다.

이것이 De Morgan’s Law의 첫 번째 법칙이다.

---

# 4. 왜 필요한지

Logical equivalence는 CS에서 매우 중요하다.

## 4.1 조건문 단순화

복잡한 조건문을 더 읽기 쉬운 형태로 바꿀 수 있다.

```cpp
if (!(isAdmin && isActive)) {
    deny();
}
```

동치 변환:

```cpp
if (!isAdmin || !isActive) {
    deny();
}
```

의미:

```text
관리자가 아니거나, 활성 상태가 아니면 거부한다.
```

---

## 4.2 버그 찾기

잘못된 변환은 조건문 버그를 만든다.

틀린 변환:

```cpp
!(A && B)
```

를 다음처럼 바꾸는 것:

```cpp
!A && !B
```

이것은 틀렸다.

왜냐하면 `A = true`, `B = false`일 때 결과가 다르다.

---

## 4.3 회로 최적화

논리 회로에서는 같은 결과를 내는 식 중 더 적은 게이트를 쓰는 식을 선택할 수 있다.

예:

```text
¬(A ∧ B)
```

는 NAND gate 하나로 구현할 수 있다.

반면:

```text
¬A ∨ ¬B
```

는 NOT gate 두 개와 OR gate 하나가 필요하다.

논리적으로는 같지만 물리적 구현 비용이 다를 수 있다.

---

## 4.4 알고리즘 correctness

알고리즘 증명에서 조건을 다른 형태로 바꿀 때 equivalence가 필요하다.

예:

```text
not (x is in range and array is sorted)
```

는 다음과 같다.

```text
x is not in range or array is not sorted
```

즉:

```text
¬(R ∧ S) ≡ ¬R ∨ ¬S
```

이런 변환이 정확해야 proof가 무너지지 않는다.

---

# 5. Truth table / 단계별 계산

## 5.1 De Morgan’s Law 1

첫 번째 법칙:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

왼쪽:

```text
¬(A ∧ B)
```

오른쪽:

```text
¬A ∨ ¬B
```

이 두 식이 항상 같은지 truth table로 증명한다.

---

## Step 1. A, B 조합 나열

| A | B |
| - | - |
| T | T |
| T | F |
| F | T |
| F | F |

---

## Step 2. 왼쪽 식 `¬(A ∧ B)` 계산

| A | B | A ∧ B | ¬(A ∧ B) |
| - | - | ----- | -------- |
| T | T | T     | F        |
| T | F | F     | T        |
| F | T | F     | T        |
| F | F | F     | T        |

---

## Step 3. 오른쪽 식 `¬A ∨ ¬B` 계산

| A | B | ¬A | ¬B | ¬A ∨ ¬B |
| - | - | -- | -- | ------- |
| T | T | F  | F  | F       |
| T | F | F  | T  | T       |
| F | T | T  | F  | T       |
| F | F | T  | T  | T       |

---

## Step 4. 두 결과 비교

| A | B | ¬(A ∧ B) | ¬A ∨ ¬B |
| - | - | -------- | ------- |
| T | T | F        | F       |
| T | F | T        | T       |
| F | T | T        | T       |
| F | F | T        | T       |

모든 row에서 결과가 같다.

따라서:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

---

# 6. De Morgan’s Law 2

두 번째 법칙:

```text
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

왼쪽:

```text
¬(A ∨ B)
```

오른쪽:

```text
¬A ∧ ¬B
```

직관적으로는 다음 뜻이다.

```text
"A 또는 B가 참인 것은 아니다."
```

즉:

```text
A도 거짓이고, B도 거짓이다.
```

따라서:

```text
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

---

## Truth table proof

| A | B | A ∨ B | ¬(A ∨ B) | ¬A | ¬B | ¬A ∧ ¬B |
| - | - | ----- | -------- | -- | -- | ------- |
| T | T | T     | F        | F  | F  | F       |
| T | F | T     | F        | F  | T  | F       |
| F | T | T     | F        | T  | F  | F       |
| F | F | F     | T        | T  | T  | T       |

비교할 column:

| A | B | ¬(A ∨ B) | ¬A ∧ ¬B |
| - | - | -------- | ------- |
| T | T | F        | F       |
| T | F | F        | F       |
| F | T | F        | F       |
| F | F | T        | T       |

모든 row에서 결과가 같다.

따라서:

```text
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

---

# 7. De Morgan’s Laws 전체 정리

두 법칙을 함께 보면 다음과 같다.

| 원래 식       | 동치인 식     |
| ---------- | --------- |
| `¬(A ∧ B)` | `¬A ∨ ¬B` |
| `¬(A ∨ B)` | `¬A ∧ ¬B` |

핵심 패턴:

```text
NOT이 괄호 안으로 들어가면
각 항은 부정되고,
AND와 OR는 서로 바뀐다.
```

즉:

```text
¬ 안으로 들어감
A → ¬A
B → ¬B
∧ → ∨
∨ → ∧
```

---

# 8. C/C++ equivalent

수학 논리를 C/C++로 바꾸면 다음과 같다.

## De Morgan 1

수학:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

C/C++:

```cpp
!(A && B) == (!A || !B)
```

예:

```cpp
if (!(isAdmin && isActive)) {
    deny();
}
```

동치 변환:

```cpp
if (!isAdmin || !isActive) {
    deny();
}
```

---

## De Morgan 2

수학:

```text
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

C/C++:

```cpp
!(A || B) == (!A && !B)
```

예:

```cpp
if (!(isAdmin || isOwner)) {
    deny();
}
```

동치 변환:

```cpp
if (!isAdmin && !isOwner) {
    deny();
}
```

의미:

```text
관리자도 아니고 작성자도 아니면 거부한다.
```

---

# 9. 프로그래밍 예시

## 9.1 예시 1: 권한 검사

```cpp
bool canAccess(bool isLoggedIn, bool hasPermission) {
    return isLoggedIn && hasPermission;
}
```

접근 불가 조건은?

```cpp
!canAccess(...)
```

즉:

```cpp
!(isLoggedIn && hasPermission)
```

De Morgan 적용:

```cpp
!isLoggedIn || !hasPermission
```

전체 코드:

```cpp
bool shouldDeny(bool isLoggedIn, bool hasPermission) {
    return !isLoggedIn || !hasPermission;
}
```

의미:

```text
로그인하지 않았거나,
권한이 없으면 거부한다.
```

---

## 9.2 예시 2: 입력 검증

```cpp
if (!(age >= 18 && hasID)) {
    reject();
}
```

De Morgan 적용:

```cpp
if (age < 18 || !hasID) {
    reject();
}
```

논리식으로 보면:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

여기서:

```text
A = age >= 18
B = hasID
```

따라서:

```text
¬A = age < 18
¬B = !hasID
```

---

## 9.3 예시 3: 범위 검사

다음 조건은 값이 정상 범위인지 확인한다.

```cpp
if (x >= 0 && x <= 100) {
    accept();
}
```

정상 범위가 아닌 경우는?

```cpp
if (!(x >= 0 && x <= 100)) {
    reject();
}
```

De Morgan 적용:

```cpp
if (x < 0 || x > 100) {
    reject();
}
```

수학적으로:

```text
¬(x ≥ 0 ∧ x ≤ 100)
≡ x < 0 ∨ x > 100
```

이 변환은 실무에서 매우 자주 나온다.

---

## 9.4 예시 4: 둘 다 없는 경우

```cpp
if (!(hasEmail || hasPhone)) {
    showError();
}
```

De Morgan 적용:

```cpp
if (!hasEmail && !hasPhone) {
    showError();
}
```

의미:

```text
이메일도 없고 전화번호도 없으면 에러
```

주의해야 한다.

```text
이메일이 없거나 전화번호가 없으면 에러
```

가 아니다.

그 조건은 다음이다.

```cpp
if (!hasEmail || !hasPhone) {
    showError();
}
```

이 둘은 완전히 다르다.

---

# 10. Boolean expression simplification

De Morgan’s Laws는 boolean expression simplification의 기본 도구다.

## 예제

```cpp
if (!(isAdmin || isOwner) && isActive) {
    deny();
}
```

논리식:

```text
¬(A ∨ B) ∧ C
```

De Morgan 적용:

```text
(¬A ∧ ¬B) ∧ C
```

C++:

```cpp
if (!isAdmin && !isOwner && isActive) {
    deny();
}
```

의미:

```text
관리자도 아니고,
작성자도 아니며,
계정이 활성 상태이면 거부한다.
```

---

## 단순화 전후 비교

| 형태         | 장점              |     |                     |
| ---------- | --------------- | --- | ------------------- |
| `!(A       |                 | B)` | 원래 조건의 반대라는 구조가 보인다 |
| `!A && !B` | 각각의 실패 조건이 명확하다 |     |                     |

실무에서는 둘 중 어느 쪽이 더 읽기 쉬운지가 중요하다.

예를 들어:

```cpp
if (!(isValidUser(user) && hasPermission(user, file))) {
    return false;
}
```

이 경우는 원래 형태가 더 읽기 쉬울 수도 있다.

반대로:

```cpp
if (!isValidUser(user) || !hasPermission(user, file)) {
    return false;
}
```

이 형태는 실패 사유를 나누기 좋다.

---

# 11. Circuit optimization

논리식은 회로로 구현할 수 있다.

## 11.1 원래 식

```text
¬(A ∧ B)
```

이것은 NAND다.

회로적으로는:

```text
A ----\
       AND ---- NOT ---- output
B ----/
```

또는 NAND gate 하나:

```text
A ----\
       NAND ---- output
B ----/
```

---

## 11.2 동치식

```text
¬A ∨ ¬B
```

회로적으로는:

```text
A ---- NOT ----\
                OR ---- output
B ---- NOT ----/
```

논리적으로는 같다.

하지만 구현 비용은 다를 수 있다.

이 때문에 digital logic design에서는 logical equivalence를 사용해 더 효율적인 회로를 찾는다.

---

# 12. 실무적 의미

## 12.1 Guard condition 작성

복잡한 조건을 긍정형으로 작성할지, 부정형으로 작성할지 판단할 수 있다.

예:

```cpp
if (!(configLoaded && databaseConnected && cacheReady)) {
    return false;
}
```

De Morgan 적용:

```cpp
if (!configLoaded || !databaseConnected || !cacheReady) {
    return false;
}
```

이 형태는 로그를 나누기 좋다.

```cpp
if (!configLoaded) {
    log("config not loaded");
    return false;
}

if (!databaseConnected) {
    log("database not connected");
    return false;
}

if (!cacheReady) {
    log("cache not ready");
    return false;
}
```

논리 변환이 디버깅 구조까지 바꾼다.

---

## 12.2 테스트 케이스 설계

다음 조건이 있다.

```cpp
if (!isLoggedIn || !hasPermission) {
    deny();
}
```

이 조건이 true가 되는 경우는:

```text
isLoggedIn = false, hasPermission = true
isLoggedIn = true, hasPermission = false
isLoggedIn = false, hasPermission = false
```

즉, 접근 거부 테스트 케이스는 최소한 이 세 범주를 포함해야 한다.

---

## 12.3 보안 정책 검증

다음 조건은 위험할 수 있다.

```cpp
if (!isAdmin && !isOwner) {
    deny();
}
```

이것은 다음과 같다.

```cpp
if (!(isAdmin || isOwner)) {
    deny();
}
```

즉:

```text
관리자도 아니고 작성자도 아니면 거부한다.
```

반대로 다음 조건은 더 강하다.

```cpp
if (!isAdmin || !isOwner) {
    deny();
}
```

이것은:

```text
관리자가 아니거나 작성자가 아니면 거부한다.
```

이 경우 허용되는 사람은 오직:

```text
관리자이면서 작성자인 사람
```

뿐이다.

둘은 완전히 다르다.

---

# 13. 흔한 오해

## 오해 1. `!(A && B)`는 `!A && !B`다

틀렸다.

정확한 법칙은:

```text
!(A && B) == !A || !B
```

즉:

```text
AND를 부정하면 OR가 된다.
```

반례:

| A | B | `!(A && B)` | `!A && !B` |
| - | - | ----------- | ---------- |
| T | F | T           | F          |

`A = true`, `B = false`에서 결과가 다르다.

---

## 오해 2. `!(A || B)`는 `!A || !B`다

틀렸다.

정확한 법칙은:

```text
!(A || B) == !A && !B
```

즉:

```text
OR를 부정하면 AND가 된다.
```

반례:

| A | B | `!(A || B)` | `!A || !B` |
|---|---|---|---|
| T | F | F | T |

---

## 오해 3. NOT을 괄호 안으로 넣을 때 연산자를 안 바꿔도 된다

틀렸다.

다음은 잘못된 변환이다.

```text
¬(A ∧ B) → ¬A ∧ ¬B
```

올바른 변환:

```text
¬(A ∧ B) → ¬A ∨ ¬B
```

NOT이 안으로 들어가면:

```text
각 항은 부정되고,
AND와 OR는 서로 바뀐다.
```

---

## 오해 4. `!`가 많을수록 더 엄밀한 코드다

그렇지 않다.

다음 코드는 읽기 어렵다.

```cpp
if (!(!isDisabled && !isExpired)) {
    reject();
}
```

단순화해보자.

```text
¬(¬D ∧ ¬E)
```

De Morgan 적용:

```text
D ∨ E
```

C++:

```cpp
if (isDisabled || isExpired) {
    reject();
}
```

훨씬 명확하다.

---

# 14. 반례

## 반례 1. 잘못된 De Morgan 적용

주장:

```text
¬(A ∧ B) ≡ ¬A ∧ ¬B
```

반례:

```text
A = True
B = False
```

왼쪽:

```text
¬(A ∧ B)
= ¬(True ∧ False)
= ¬False
= True
```

오른쪽:

```text
¬A ∧ ¬B
= False ∧ True
= False
```

왼쪽과 오른쪽이 다르다.

따라서 equivalent가 아니다.

---

## 반례 2. 보안 조건에서 생기는 오류

의도:

```text
로그인하지 않았거나 권한이 없으면 거부한다.
```

올바른 코드:

```cpp
if (!isLoggedIn || !hasPermission) {
    deny();
}
```

이것은 다음과 같다.

```cpp
if (!(isLoggedIn && hasPermission)) {
    deny();
}
```

그런데 누군가 잘못 바꿨다.

```cpp
if (!isLoggedIn && !hasPermission) {
    deny();
}
```

이 조건은 언제 true인가?

```text
로그인도 안 했고 권한도 없을 때만 true
```

문제는 다음 경우다.

| isLoggedIn | hasPermission | 올바른 deny | 잘못된 deny |
| ---------- | ------------- | -------- | -------- |
| T          | F             | T        | F        |
| F          | T             | T        | F        |

즉, 로그인했지만 권한 없는 사용자가 통과할 수 있다.
또는 이상한 상태에서 권한만 true인 사용자가 통과할 수 있다.

이것은 실제 보안 버그가 될 수 있다.

---

# 15. 확인 문제

## 문제 1

다음을 De Morgan’s Law로 변환하라.

```text
¬(A ∧ B)
```

---

## 문제 2

다음을 De Morgan’s Law로 변환하라.

```text
¬(A ∨ B)
```

---

## 문제 3

다음 C++ 조건을 더 풀어 써라.

```cpp
if (!(isAdmin && isActive)) {
    deny();
}
```

---

## 문제 4

다음 C++ 조건을 더 풀어 써라.

```cpp
if (!(hasEmail || hasPhone)) {
    showError();
}
```

---

## 문제 5

다음 두 식은 equivalent인가?

```text
¬(A ∧ B)
```

```text
¬A ∧ ¬B
```

equivalent가 아니라면 반례를 하나 제시하라.

---

## 문제 6

다음 두 조건의 차이를 설명하라.

```cpp
if (!isAdmin && !isOwner) {
    deny();
}
```

```cpp
if (!isAdmin || !isOwner) {
    deny();
}
```

---

# 16. 실습 과제

## 과제 1. Truth table proof

다음 법칙을 truth table로 증명하라.

```text
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

작성 column:

```text
A
B
A ∨ B
¬(A ∨ B)
¬A
¬B
¬A ∧ ¬B
```

마지막으로 `¬(A ∨ B)` column과 `¬A ∧ ¬B` column을 비교하라.

---

## 과제 2. C++ 조건문 변환

다음 조건을 De Morgan’s Law로 변환하라.

```cpp
if (!(isConnected && response.ok())) {
    retry();
}
```

주의:

```cpp
response.ok()
```

를 안전하게 호출할 수 있는 상황인지도 생각해야 한다.

논리 변환만 하면:

```cpp
if (!isConnected || !response.ok()) {
    retry();
}
```

이지만, 실제 코드에서는 `response` 객체가 유효한지도 고려해야 한다.

---

## 과제 3. 보안 조건 검증

다음 두 코드 중 어느 것이 더 강한 조건인지 분석하라.

```cpp
if (!isAuthenticated || !hasRoleAdmin) {
    deny();
}
```

```cpp
if (!isAuthenticated && !hasRoleAdmin) {
    deny();
}
```

분석할 것:

```text
1. 각각 언제 deny 되는가?
2. 각각 언제 통과되는가?
3. 보안적으로 어떤 조건이 더 엄격한가?
```

---

# 17. 핵심 정리

Logical equivalence는 다음 뜻이다.

```text
두 논리식이 모든 truth value 조합에서 같은 결과를 가진다.
```

기호:

```text
P ≡ Q
```

De Morgan’s Laws는 다음 두 개다.

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
¬(A ∨ B) ≡ ¬A ∧ ¬B
```

C/C++에서는 다음과 같다.

```cpp
!(A && B) == (!A || !B)
!(A || B) == (!A && !B)
```

핵심 패턴:

```text
NOT이 괄호 안으로 들어가면
각 항은 부정되고
AND와 OR가 서로 바뀐다.
```

가장 흔한 오류:

```text
¬(A ∧ B)를 ¬A ∧ ¬B로 바꾸는 것
```

정확한 변환:

```text
¬(A ∧ B) ≡ ¬A ∨ ¬B
```

실무적으로 De Morgan’s Laws는 다음에 중요하다.

```text
1. 조건문 단순화
2. 권한 검사 검증
3. 입력 검증
4. guard condition 작성
5. circuit optimization
6. algorithm correctness proof
```