# Lecture 5. Tautology와 Contradiction

## 1. 핵심 질문

> 어떤 논리식은 왜 항상 참이고, 어떤 논리식은 왜 절대로 참이 될 수 없는가?

지금까지 우리는 논리식이 특정 입력 조합에서 true 또는 false가 되는 것을 보았다.

예:

```text
A ∧ B
```

이 식은 `A = true`, `B = true`일 때만 true다.

하지만 어떤 논리식은 입력값과 무관하게 **항상 true**다.

예:

```text
A ∨ ¬A
```

`A`가 true이든 false이든 결과는 항상 true다.

반대로 어떤 논리식은 입력값과 무관하게 **항상 false**다.

예:

```text
A ∧ ¬A
```

`A`가 true이든 false이든 결과는 항상 false다.

이번 Lecture의 목표는 다음이다.

```text
1. tautology를 이해한다.
2. contradiction을 이해한다.
3. contingency를 이해한다.
4. truth table로 세 개를 구분한다.
5. proof, SAT/SMT solver, compiler optimization과 연결한다.
```

---

# 2. 형식적 정의

## 2.1 Tautology

**Tautology**, 항진명제란 모든 truth value 조합에서 항상 참인 논리식이다.

형식적으로:

```text
논리식 P가 모든 가능한 변수 할당에서 true이면,
P는 tautology다.
```

기호로는 보통 다음처럼 표현한다.

```text
⊨ P
```

읽는 법:

```text
P is valid.
P는 모든 경우에 참이다.
```

대표 예:

```text
A ∨ ¬A
```

이것은 **Law of Excluded Middle**, 배중률이라고 부른다.

의미:

```text
A이거나 A가 아니다.
```

고전 논리에서는 반드시 참이다.

---

## 2.2 Contradiction

**Contradiction**, 모순명제란 모든 truth value 조합에서 항상 거짓인 논리식이다.

형식적으로:

```text
논리식 P가 모든 가능한 변수 할당에서 false이면,
P는 contradiction이다.
```

대표 예:

```text
A ∧ ¬A
```

의미:

```text
A이면서 동시에 A가 아니다.
```

고전 논리에서는 불가능하다.

---

## 2.3 Contingency

**Contingency**, 우연명제 또는 상황명제란 어떤 경우에는 참이고 어떤 경우에는 거짓인 논리식이다.

예:

```text
A ∧ B
```

이 식은 `A = true`, `B = true`이면 true지만, 다른 경우에는 false다.

따라서 tautology도 아니고 contradiction도 아니다.

---

# 3. 직관적 설명

세 개를 직관적으로 구분하면 다음과 같다.

| 종류            | 의미           | 예        |
| ------------- | ------------ | -------- |
| Tautology     | 무슨 일이 있어도 참  | `A ∨ ¬A` |
| Contradiction | 무슨 일이 있어도 거짓 | `A ∧ ¬A` |
| Contingency   | 상황에 따라 참/거짓  | `A ∧ B`  |

프로그래밍으로 비유하면 다음과 같다.

```cpp
if (x > 0 || x <= 0) {
    // 항상 실행됨
}
```

정수 `x`에 대해 `x > 0 || x <= 0`은 항상 true다.

반대로:

```cpp
if (x > 0 && x <= 0) {
    // 절대 실행되지 않음
}
```

정수 `x`가 동시에 `0보다 크고`, `0 이하`일 수는 없다.
따라서 항상 false다.

---

# 4. 왜 필요한지

## 4.1 Proof에서 중요하다

수학적 증명에서는 어떤 명제가 항상 참인지 확인하는 것이 중요하다.

예:

```text
A → A
```

이것은 항상 참이다.

왜냐하면 어떤 명제가 참이면 자기 자신은 참이고, 거짓이면 implication 규칙상 전체가 참이기 때문이다.
Implication은 다음 Lecture에서 자세히 다룬다.

---

## 4.2 Proof by contradiction에서 중요하다

**Proof by contradiction**, 귀류법은 어떤 가정을 했을 때 contradiction이 나오면 그 가정이 틀렸다고 결론 내리는 방식이다.

구조는 다음과 같다.

```text
1. 증명하고 싶은 명제 P가 있다고 하자.
2. 반대로 ¬P를 가정한다.
3. 그 가정에서 contradiction을 도출한다.
4. 따라서 ¬P는 불가능하다.
5. 그러므로 P가 참이다.
```

여기서 contradiction은 보통 이런 형태다.

```text
Q ∧ ¬Q
```

즉, 어떤 명제 `Q`와 그 부정 `¬Q`가 동시에 참이라는 불가능한 결론이 나온다.

---

## 4.3 SAT solver와 연결된다

SAT는 **satisfiability**, 즉 어떤 논리식이 true가 될 수 있는지 판단하는 문제다.

예:

```text
A ∧ B
```

이 식은 satisfiable이다.

왜냐하면:

```text
A = true, B = true
```

이면 true가 되기 때문이다.

반면:

```text
A ∧ ¬A
```

는 unsatisfiable이다.

어떤 `A` 값에서도 true가 될 수 없다.

SAT solver는 이런 문제를 자동으로 푸는 도구다.

---

## 4.4 Compiler optimization에서 중요하다

컴파일러는 항상 true이거나 항상 false인 조건을 최적화할 수 있다.

예:

```cpp
if (true) {
    run();
}
```

는 사실상:

```cpp
run();
```

과 같다.

반대로:

```cpp
if (false) {
    run();
}
```

는 제거될 수 있다.

더 현실적인 예:

```cpp
if (x == x) {
    run();
}
```

일반적인 정수에서는 항상 true다.
다만 부동소수점 `NaN` 같은 특수한 경우는 조심해야 한다.

논리적으로는 tautology처럼 보이는 조건도 실제 프로그래밍 언어의 타입과 값 체계에 따라 달라질 수 있다.

---

# 5. Truth table / 단계별 계산

## 5.1 Tautology 예: `A ∨ ¬A`

```text
A ∨ ¬A
```

Truth table:

| A | ¬A | A ∨ ¬A |
| - | -- | ------ |
| T | F  | T      |
| F | T  | T      |

마지막 column이 모두 true다.

따라서:

```text
A ∨ ¬A
```

는 tautology다.

---

## 5.2 Contradiction 예: `A ∧ ¬A`

```text
A ∧ ¬A
```

Truth table:

| A | ¬A | A ∧ ¬A |
| - | -- | ------ |
| T | F  | F      |
| F | T  | F      |

마지막 column이 모두 false다.

따라서:

```text
A ∧ ¬A
```

는 contradiction이다.

---

## 5.3 Contingency 예: `A ∧ B`

```text
A ∧ B
```

Truth table:

| A | B | A ∧ B |
| - | - | ----- |
| T | T | T     |
| T | F | F     |
| F | T | F     |
| F | F | F     |

마지막 column에 true도 있고 false도 있다.

따라서:

```text
A ∧ B
```

는 contingency다.

---

# 6. 더 복잡한 예제

## 예제 1

다음 식을 분류하라.

```text
(A ∧ B) → A
```

아직 implication을 정식으로 배우지는 않았지만, 의미만 먼저 보자.

```text
A와 B가 둘 다 참이면, A는 참이다.
```

이것은 항상 참이어야 한다.

왜냐하면 `A ∧ B`가 참이라는 말 안에 이미 `A`가 참이라는 정보가 들어 있기 때문이다.

따라서 이 식은 tautology다.

다음 Lecture에서 truth table로 엄밀히 증명한다.

---

## 예제 2

다음 식을 분류하라.

```text
(A ∨ B) ∧ ¬A ∧ ¬B
```

직관적으로 해석하자.

```text
A 또는 B 중 적어도 하나는 참이다.
그런데 A는 거짓이다.
그리고 B도 거짓이다.
```

앞부분은 “둘 중 하나는 참”이라고 말하고, 뒷부분은 “둘 다 거짓”이라고 말한다.

동시에 만족할 수 없다.

따라서 contradiction이다.

Truth table로 확인해보자.

| A | B | A ∨ B | ¬A | ¬B | `(A ∨ B) ∧ ¬A ∧ ¬B` |
| - | - | ----- | -- | -- | ------------------- |
| T | T | T     | F  | F  | F                   |
| T | F | T     | F  | T  | F                   |
| F | T | T     | T  | F  | F                   |
| F | F | F     | T  | T  | F                   |

마지막 column이 모두 false다.

따라서 contradiction이다.

---

## 예제 3

다음 식을 분류하라.

```text
(A ∧ B) ∨ (A ∧ ¬B)
```

이 식은 이전 Lecture에서 봤다.

직관적으로:

```text
A이면서 B이거나,
A이면서 B가 아니다.
```

즉:

```text
A가 참이면 B가 어떤 값이든 된다.
```

따라서 이 식은 `A`와 equivalent다.

Truth table:

| A | B | ¬B | A ∧ B | A ∧ ¬B | `(A ∧ B) ∨ (A ∧ ¬B)` |
| - | - | -- | ----- | ------ | -------------------- |
| T | T | F  | T     | F      | T                    |
| T | F | T  | F     | T      | T                    |
| F | T | F  | F     | F      | F                    |
| F | F | T  | F     | F      | F                    |

마지막 column은 `A`와 같다.

| A | 결과 |
| - | -- |
| T | T  |
| T | T  |
| F | F  |
| F | F  |

따라서 이 식은 contingency다.
항상 true도 아니고, 항상 false도 아니다.

---

# 7. 프로그래밍 예시

## 7.1 항상 true인 조건

```cpp
bool check(int x) {
    return x > 0 || x <= 0;
}
```

정수 `x` 기준으로 이 조건은 항상 true다.

논리식으로 보면:

```text
A ∨ ¬A
```

여기서:

```text
A = x > 0
¬A = x <= 0
```

따라서 `check`는 사실상 다음과 같다.

```cpp
bool check(int x) {
    return true;
}
```

---

## 7.2 항상 false인 조건

```cpp
bool impossible(int x) {
    return x > 0 && x <= 0;
}
```

정수 `x`는 동시에 `0보다 크고`, `0 이하`일 수 없다.

논리식으로 보면:

```text
A ∧ ¬A
```

따라서 이 함수는 사실상 다음과 같다.

```cpp
bool impossible(int x) {
    return false;
}
```

---

## 7.3 Dead code

```cpp
if (x > 0 && x <= 0) {
    expensiveOperation();
}
```

이 블록은 실행될 수 없다.

이런 코드를 **dead code**라고 부를 수 있다.

컴파일러나 정적 분석 도구는 이런 조건을 감지할 수 있다.

---

## 7.4 조건이 너무 약한 경우

```cpp
if (isAdmin || !isAdmin) {
    deleteFile();
}
```

이 조건은 항상 true다.

즉, 사실상 다음과 같다.

```cpp
deleteFile();
```

만약 개발자가 권한 검사를 의도했다면 심각한 보안 버그다.

---

## 7.5 조건이 너무 강한 경우

```cpp
if (isAdmin && !isAdmin) {
    deleteFile();
}
```

이 조건은 항상 false다.

즉, `deleteFile()`은 절대 실행되지 않는다.

이 경우 기능이 작동하지 않는 버그가 된다.

---

# 8. 실무적 의미

## 8.1 조건문 리뷰

실무에서 다음과 같은 조건은 반드시 의심해야 한다.

```cpp
if (A || !A) {
    ...
}
```

이것은 항상 true다.

또한 다음도 의심해야 한다.

```cpp
if (A && !A) {
    ...
}
```

이것은 항상 false다.

---

## 8.2 테스트 설계

어떤 조건이 tautology라면 테스트가 의미 없어질 수 있다.

예:

```cpp
if (inputValid || !inputValid) {
    process(input);
}
```

이 코드는 `inputValid` 값과 상관없이 항상 실행된다.

즉, `inputValid` 체크가 아무 역할을 하지 않는다.

---

## 8.3 SAT/SMT solver

SAT solver는 명제 논리식이 satisfiable인지 확인한다.

| 논리식            | SAT 여부        |
| -------------- | ------------- |
| `A ∧ B`        | satisfiable   |
| `A ∨ B`        | satisfiable   |
| `A ∧ ¬A`       | unsatisfiable |
| `(A ∨ B) ∧ ¬A` | satisfiable   |

예:

```text
(A ∨ B) ∧ ¬A
```

이 식은 satisfiable이다.

왜냐하면:

```text
A = false
B = true
```

이면 전체가 true다.

SAT solver는 이런 할당을 찾아낸다.

---

## 8.4 SMT solver

SMT는 SAT보다 더 확장된 문제를 푼다.

SAT가 순수 boolean logic을 다룬다면, SMT는 다음 같은 이론을 함께 다룬다.

```text
정수 산술
배열
비트벡터
문자열
포인터 모델
```

예:

```text
x > 0 ∧ x <= 0
```

이것은 단순히 `A ∧ ¬A`처럼 보일 수도 있지만, 실제로는 정수 산술 이론을 사용해서 unsatisfiable임을 판단한다.

이런 도구는 프로그램 검증, 정적 분석, 보안 분석에 사용된다.

---

## 8.5 Compiler optimization

컴파일러는 다음과 같은 최적화를 할 수 있다.

```cpp
if (condition || true) {
    run();
}
```

이 조건은 항상 true다.

```cpp
if (condition && false) {
    run();
}
```

이 조건은 항상 false다.

논리 법칙으로 보면:

```text
A ∨ true ≡ true
A ∧ false ≡ false
```

이런 단순화는 constant folding, dead code elimination과 연결된다.

---

# 9. 흔한 오해

## 오해 1. Tautology는 “중요하지 않은 당연한 말”이다

항상 그렇지 않다.

논리학에서 tautology는 proof rule의 기반이다.

예를 들어:

```text
A → A
```

는 단순해 보이지만, 형식 체계에서는 “항상 참인 식”이라는 사실이 중요하다.

또한 다음 같은 법칙도 tautology다.

```text
(A ∧ B) → A
```

이것은 conjunction elimination의 기반이다.

---

## 오해 2. Contradiction은 그냥 “틀린 문장”이다

정확히 말하면 contradiction은 단순히 false인 문장이 아니다.

예:

```text
5 > 10
```

이 문장은 false이지만 contradiction이라고 하지는 않는다.

왜냐하면 변수 조합에 따라 변하는 논리식이 아니라, 단일 false proposition이기 때문이다.

논리식 수준에서 contradiction은 보통 모든 변수 할당에서 false인 식을 말한다.

예:

```text
A ∧ ¬A
```

---

## 오해 3. Contingency는 애매한 문장이다

아니다.

Contingency는 truth value가 애매한 문장이 아니다.

각 입력 조합에서는 명확히 true 또는 false다.
다만 입력 조합에 따라 결과가 달라질 뿐이다.

예:

```text
A ∧ B
```

는 contingency지만 애매하지 않다.

각 row에서는 결과가 분명하다.

---

## 오해 4. `x == x`는 항상 true다

대부분의 정수 타입에서는 맞다.

하지만 부동소수점에서는 조심해야 한다.

C/C++에서 `NaN`은 자기 자신과 같지 않다.

```cpp
double x = std::numeric_limits<double>::quiet_NaN();

std::cout << (x == x) << "\n"; // false
```

따라서 프로그래밍에서는 “수학적으로 당연한 것”이 언어의 값 체계에서는 예외를 가질 수 있다.

---

# 10. 반례

## 반례 1. Tautology라고 착각한 식

주장:

```text
A ∧ B는 tautology다.
```

반례:

```text
A = true
B = false
```

계산:

```text
A ∧ B = true ∧ false = false
```

한 row라도 false가 있으므로 tautology가 아니다.

---

## 반례 2. Contradiction이라고 착각한 식

주장:

```text
A ∨ B는 contradiction이다.
```

반례:

```text
A = true
B = false
```

계산:

```text
A ∨ B = true ∨ false = true
```

한 row라도 true가 있으므로 contradiction이 아니다.

---

## 반례 3. 항상 false라고 착각한 조건

```cpp
if (x < 0 || x >= 0) {
    run();
}
```

이 조건은 항상 true다.

왜냐하면 정수 `x`는 `0보다 작거나`, `0 이상`이다.

즉:

```text
A ∨ ¬A
```

형태다.

---

## 반례 4. 항상 true라고 착각한 조건

```cpp
if (x < 0 && x >= 0) {
    run();
}
```

이 조건은 항상 false다.

왜냐하면 정수 `x`가 동시에 `0보다 작고`, `0 이상`일 수 없기 때문이다.

---

# 11. 확인 문제

## 문제 1

다음 논리식은 tautology, contradiction, contingency 중 무엇인가?

```text
A ∨ ¬A
```

---

## 문제 2

다음 논리식은 무엇인가?

```text
A ∧ ¬A
```

---

## 문제 3

다음 논리식은 무엇인가?

```text
A ∨ B
```

---

## 문제 4

다음 논리식은 무엇인가?

```text
(A ∧ B) ∨ (A ∧ ¬B)
```

힌트:

```text
이 식은 A와 equivalent다.
```

---

## 문제 5

다음 C++ 조건은 항상 true인가, 항상 false인가, 아니면 입력에 따라 달라지는가?

```cpp
if (isLoggedIn || !isLoggedIn) {
    access();
}
```

---

## 문제 6

다음 C++ 조건은 어떤 문제가 있는가?

```cpp
if (hasPermission && !hasPermission) {
    allowAccess();
}
```

---

# 12. 실습 과제

## 과제 1. Truth table로 분류하기

다음 식을 truth table로 분석하고, tautology / contradiction / contingency 중 하나로 분류하라.

```text
(A ∨ B) ∨ ¬A
```

작성 column:

```text
A
B
A ∨ B
¬A
(A ∨ B) ∨ ¬A
```

힌트: 마지막 column이 모두 true인지 확인하라.

---

## 과제 2. Contradiction 찾기

다음 식이 contradiction인지 truth table로 확인하라.

```text
(A ∧ B) ∧ ¬A
```

작성 column:

```text
A
B
A ∧ B
¬A
(A ∧ B) ∧ ¬A
```

---

## 과제 3. C++ 코드 분석

다음 코드를 보고 조건이 의미 있는지 분석하라.

```cpp
bool canProceed(bool ready, bool failed) {
    return (ready || !ready) && !failed;
}
```

분석할 것:

```text
1. ready || !ready는 무엇인가?
2. 전체 조건은 무엇과 equivalent인가?
3. 이 코드는 어떻게 단순화할 수 있는가?
```

---

## 과제 4. 보안 조건 검토

다음 조건을 검토하라.

```cpp
if (isAdmin || !isAdmin) {
    deleteUser();
}
```

분석할 것:

```text
1. 이 조건은 tautology인가?
2. deleteUser()는 언제 실행되는가?
3. 보안상 왜 위험한가?
```

---

# 13. 핵심 정리

이번 Lecture의 핵심은 세 가지다.

| 개념            | 정의                | 예        |
| ------------- | ----------------- | -------- |
| Tautology     | 모든 경우에 true       | `A ∨ ¬A` |
| Contradiction | 모든 경우에 false      | `A ∧ ¬A` |
| Contingency   | 경우에 따라 true/false | `A ∧ B`  |

Truth table에서 판단하는 방법:

```text
마지막 column이 모두 true  → tautology
마지막 column이 모두 false → contradiction
true와 false가 섞임       → contingency
```

CS에서의 의미:

```text
1. proof의 기반
2. proof by contradiction
3. SAT/SMT solver
4. compiler optimization
5. dead code detection
6. security condition review
```

중요한 공식:

```text
A ∨ ¬A = 항상 true
A ∧ ¬A = 항상 false
```

프로그래밍에서는 추가로 주의해야 한다.

```text
수학적으로 항상 참처럼 보이는 조건도
언어의 타입, NaN, undefined behavior, side effect 때문에
실제 실행에서는 조심해야 한다.
```