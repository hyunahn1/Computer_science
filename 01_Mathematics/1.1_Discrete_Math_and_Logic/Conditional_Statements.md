# Lecture 6. Conditional Statements

## 1. 핵심 질문

> “if p, then q”는 왜 `p`가 false일 때도 true로 취급되는가?

이번 Lecture는 Part A에서 가장 헷갈리기 쉬운 부분이다.

우리가 배울 핵심 표현은 다음이다.

```text
p → q
```

읽는 법:

```text
if p, then q
p이면 q이다
p가 참이면 q도 참이다
```

예:

```text
p = 비가 온다
q = 땅이 젖는다

p → q = 비가 오면 땅이 젖는다
```

중요한 점은 implication `p → q`가 **인과관계**를 뜻하는 것이 아니라는 점이다.
명제논리에서 `p → q`는 truth table로 정의되는 논리 연산자다.

---

# 2. 형식적 정의

## 2.1 Conditional Statement

**Conditional statement**, 조건명제는 다음 형태의 명제다.

```text
p → q
```

여기서:

| 기호      | 이름          | 의미     |
| ------- | ----------- | ------ |
| `p`     | antecedent  | 조건, 전건 |
| `q`     | consequent  | 결론, 후건 |
| `p → q` | implication | p이면 q  |

형식적 정의는 truth table로 주어진다.

| p | q | p → q |
| - | - | ----- |
| T | T | T     |
| T | F | F     |
| F | T | T     |
| F | F | T     |

가장 중요한 row는 이것이다.

| p | q | p → q |
| - | - | ----- |
| T | F | F     |

즉:

```text
p → q는 p가 true인데 q가 false일 때만 false다.
```

나머지 경우는 모두 true다.

---

# 3. 직관적 설명

## 3.1 약속으로 이해하기

Implication은 “약속”처럼 이해하면 좋다.

```text
p → q
```

를 다음 약속이라고 하자.

```text
p가 일어나면 q를 보장하겠다.
```

예:

```text
p = 네가 숙제를 제출한다
q = 내가 피드백을 준다

p → q = 네가 숙제를 제출하면 내가 피드백을 준다
```

이 약속이 깨지는 경우는 언제인가?

| p: 숙제 제출 | q: 피드백 제공 | 약속 위반? |
| -------- | --------- | ------ |
| 제출함      | 피드백 줌     | 위반 아님  |
| 제출함      | 피드백 안 줌   | 위반     |
| 제출 안 함   | 피드백 줌     | 위반 아님  |
| 제출 안 함   | 피드백 안 줌   | 위반 아님  |

즉, 약속이 깨지는 경우는 오직 하나다.

```text
p가 true인데 q가 false인 경우
```

그래서 `p → q`의 truth table은 다음이 된다.

| p | q | p → q |
| - | - | ----- |
| T | T | T     |
| T | F | F     |
| F | T | T     |
| F | F | T     |

---

# 4. 왜 필요한지

Implication은 컴퓨터과학에서 매우 중요하다.

## 4.1 프로그램 조건

```cpp
if (x > 0) {
    assert(result > 0);
}
```

이 코드는 논리적으로 다음과 비슷하다.

```text
x > 0 → result > 0
```

즉:

```text
x가 양수이면 result도 양수여야 한다.
```

---

## 4.2 Precondition / Postcondition

함수 명세에서 자주 사용된다.

```text
Precondition: array is sorted
Postcondition: binarySearch returns correct index
```

논리식으로:

```text
array is sorted → binarySearch returns correct index
```

즉:

```text
배열이 정렬되어 있다면 binarySearch는 올바른 결과를 반환해야 한다.
```

여기서 중요한 점:

```text
배열이 정렬되어 있지 않은 경우에는 이 명세가 함수의 결과를 보장하지 않는다.
```

이것이 implication의 핵심이다.

---

## 4.3 알고리즘 correctness

알고리즘 증명은 보통 이런 형태다.

```text
Precondition → Postcondition
```

예:

```text
input satisfies constraints → algorithm returns correct output
```

즉:

```text
입력이 조건을 만족하면 알고리즘은 올바른 결과를 반환한다.
```

---

## 4.4 Type system / static analysis

정적 분석에서도 implication이 나온다.

```text
pointer is not null → pointer can be dereferenced safely
```

C/C++로 보면:

```cpp
if (p != nullptr) {
    use(*p);
}
```

여기서 `p != nullptr`가 true일 때만 `*p`를 안전하게 사용할 수 있다.

---

# 5. Truth table / 단계별 계산

## 5.1 Implication truth table

다시 표를 보자.

| p | q | p → q |
| - | - | ----- |
| T | T | T     |
| T | F | F     |
| F | T | T     |
| F | F | T     |

각 row를 문장으로 해석한다.

예:

```text
p = 비가 온다
q = 땅이 젖는다
```

| p | q | 해석              | p → q |
| - | - | --------------- | ----- |
| T | T | 비가 오고 땅이 젖음     | T     |
| T | F | 비가 오는데 땅이 안 젖음  | F     |
| F | T | 비는 안 오는데 땅이 젖음  | T     |
| F | F | 비도 안 오고 땅도 안 젖음 | T     |

`p → q`는 “비가 오면 땅이 젖는다”라는 주장이다.

이 주장을 깨는 경우는:

```text
비가 오는데 땅이 젖지 않는 경우
```

뿐이다.

비가 오지 않은 경우에는 이 명제가 깨졌다고 말할 수 없다.
그래서 `p = false`인 두 row는 true로 처리된다.

---

# 6. Vacuous Truth

## 6.1 정의

**Vacuous truth**는 전건 `p`가 false이기 때문에 implication `p → q`가 true가 되는 현상이다.

```text
p = false이면 p → q는 q와 관계없이 true
```

Truth table에서 이 두 row다.

| p | q | p → q |
| - | - | ----- |
| F | T | T     |
| F | F | T     |

---

## 6.2 직관적 예시

문장:

```text
내가 대통령이면 너에게 달을 사주겠다.
```

논리식:

```text
p = 나는 대통령이다
q = 너에게 달을 사준다

p → q
```

내가 대통령이 아니라면 `p`가 false다.
그러면 이 implication은 논리적으로 true로 취급된다.

왜냐하면 이 약속은 **p가 발생했을 때만 검사되는 약속**이기 때문이다.

---

## 6.3 프로그래밍 예시

함수 명세:

```text
if input is valid, function returns sorted output
```

논리식:

```text
valid(input) → sorted(output)
```

만약 input이 invalid라면?

```text
valid(input) = false
```

그러면 이 명세는 output에 대해 아무것도 보장하지 않는다.

이것을 잘못 이해하면 다음 오해가 생긴다.

```text
invalid input이어도 sorted output을 반환해야 한다.
```

아니다.
명세는 valid input에 대해서만 보장한다.

---

# 7. Implication의 동치식

Implication은 `OR`와 `NOT`으로 표현할 수 있다.

```text
p → q ≡ ¬p ∨ q
```

이것은 매우 중요하다.

## Truth table proof

| p | q | ¬p | ¬p ∨ q | p → q |
| - | - | -- | ------ | ----- |
| T | T | F  | T      | T     |
| T | F | F  | F      | F     |
| F | T | T  | T      | T     |
| F | F | T  | T      | T     |

`¬p ∨ q`와 `p → q`의 column이 같다.

따라서:

```text
p → q ≡ ¬p ∨ q
```

직관적으로:

```text
p이면 q
```

는 다음과 같다.

```text
p가 아니거나, q이다.
```

즉:

```text
조건 p가 발생하지 않았거나,
발생했다면 q가 성립한다.
```

---

# 8. Contrapositive, Converse, Inverse

이제 네 가지 형태를 구분해야 한다.

원래 명제:

```text
p → q
```

## 8.1 Contrapositive

```text
¬q → ¬p
```

읽는 법:

```text
q가 아니면 p가 아니다
```

원래 명제와 **항상 동치**다.

```text
p → q ≡ ¬q → ¬p
```

---

## 8.2 Converse

```text
q → p
```

읽는 법:

```text
q이면 p이다
```

원래 명제와 일반적으로 동치가 아니다.

---

## 8.3 Inverse

```text
¬p → ¬q
```

읽는 법:

```text
p가 아니면 q가 아니다
```

원래 명제와 일반적으로 동치가 아니다.

---

## 8.4 네 형태 정리

| 이름             | 형태        | 원래 명제와 동치인가? |
| -------------- | --------- | ------------ |
| Original       | `p → q`   | 자기 자신        |
| Contrapositive | `¬q → ¬p` | O            |
| Converse       | `q → p`   | X            |
| Inverse        | `¬p → ¬q` | X            |

중요한 관계:

```text
p → q ≡ ¬q → ¬p
q → p ≡ ¬p → ¬q
```

즉, converse와 inverse는 서로 동치지만, 원래 명제와는 일반적으로 동치가 아니다.

---

# 9. Contrapositive가 동치인 이유

## 9.1 Truth table proof

원래 명제:

```text
p → q
```

대우:

```text
¬q → ¬p
```

Truth table:

| p | q | ¬p | ¬q | p → q | ¬q → ¬p |
| - | - | -- | -- | ----- | ------- |
| T | T | F  | F  | T     | T       |
| T | F | F  | T  | F     | F       |
| F | T | T  | F  | T     | T       |
| F | F | T  | T  | T     | T       |

비교:

| p | q | p → q | ¬q → ¬p |
| - | - | ----- | ------- |
| T | T | T     | T       |
| T | F | F     | F       |
| F | T | T     | T       |
| F | F | T     | T       |

모든 row에서 같다.

따라서:

```text
p → q ≡ ¬q → ¬p
```

---

## 9.2 직관적 설명

원래 명제:

```text
p → q
```

뜻:

```text
p가 참인 모든 경우에는 q도 참이다.
```

그러면 q가 거짓인 경우에 p가 참일 수 있을까?

안 된다.

왜냐하면 p가 참이면 q도 참이어야 하기 때문이다.

따라서:

```text
q가 거짓이면 p도 거짓이다.
```

즉:

```text
¬q → ¬p
```

이다.

---

# 10. Converse가 동치가 아닌 이유

원래 명제:

```text
p → q
```

Converse:

```text
q → p
```

예를 들어:

```text
p = x는 4의 배수다
q = x는 2의 배수다
```

원래 명제:

```text
x가 4의 배수이면 x는 2의 배수다.
```

이것은 참이다.

Converse:

```text
x가 2의 배수이면 x는 4의 배수다.
```

이것은 거짓이다.

반례:

```text
x = 6
```

6은 2의 배수지만 4의 배수는 아니다.

따라서:

```text
p → q
```

가 참이어도

```text
q → p
```

가 참이라는 보장은 없다.

---

# 11. Inverse가 동치가 아닌 이유

원래 명제:

```text
p → q
```

Inverse:

```text
¬p → ¬q
```

같은 예를 사용하자.

```text
p = x는 4의 배수다
q = x는 2의 배수다
```

원래 명제:

```text
x가 4의 배수이면 x는 2의 배수다.
```

참이다.

Inverse:

```text
x가 4의 배수가 아니면 x는 2의 배수가 아니다.
```

거짓이다.

반례:

```text
x = 6
```

6은 4의 배수가 아니지만 2의 배수다.

따라서 inverse는 original과 일반적으로 동치가 아니다.

---

# 12. 프로그래밍 예시

## 12.1 Precondition / Postcondition

함수 명세:

```cpp
int binarySearch(const std::vector<int>& a, int target);
```

명세:

```text
If array a is sorted, then binarySearch returns a correct index or not found.
```

논리식:

```text
S → C
```

여기서:

```text
S = array is sorted
C = binarySearch returns correct result
```

이 명세는 다음을 뜻한다.

| S | C | S → C | 해석                    |
| - | - | ----- | --------------------- |
| T | T | T     | 정렬된 배열에서 올바른 결과       |
| T | F | F     | 정렬된 배열인데 틀린 결과: 버그    |
| F | T | T     | 정렬 안 됐는데 우연히 맞음       |
| F | F | T     | 정렬 안 됐고 틀림: 명세 위반은 아님 |

중요하다.

```text
Precondition이 false이면 postcondition을 보장하지 않는다.
```

즉, 정렬되지 않은 배열에서 binary search가 틀려도 이 명세 자체를 위반한 것은 아니다.

---

## 12.2 Assertion

```cpp
if (x > 0) {
    assert(y > 0);
}
```

논리적으로는:

```text
x > 0 → y > 0
```

이 assertion이 실패하는 경우는 오직 하나다.

```text
x > 0은 true인데 y > 0은 false인 경우
```

C++ 코드로 보면:

```cpp
if (x > 0 && !(y > 0)) {
    // implication violation
}
```

즉:

```text
p → q가 false인 조건 = p ∧ ¬q
```

이것도 중요하다.

---

## 12.3 Guard condition

```cpp
if (p == nullptr) {
    return;
}

p->run();
```

이 코드는 암묵적으로 다음 논리를 만든다.

```text
p->run()이 실행된다 → p != nullptr
```

즉, `p->run()`이 실행되는 시점에서는 `p`가 null이 아니어야 한다.

이런 식으로 guard condition은 안전한 후속 연산의 전제를 만든다.

---

# 13. Implication 위반 조건

`p → q`가 false가 되는 조건은 정확히 하나다.

```text
p ∧ ¬q
```

Truth table:

| p | q | ¬q | p ∧ ¬q | p → q |
| - | - | -- | ------ | ----- |
| T | T | F  | F      | T     |
| T | F | T  | T      | F     |
| F | T | F  | F      | T     |
| F | F | T  | F      | T     |

따라서 다음이 성립한다.

```text
¬(p → q) ≡ p ∧ ¬q
```

직관적으로:

```text
"p이면 q"가 틀렸다는 것은
p가 실제로 일어났는데 q가 일어나지 않았다는 뜻이다.
```

이것은 버그 리포트 작성에도 중요하다.

예:

```text
Spec: valid input → returns sorted output
Violation: valid input ∧ output is not sorted
```

---

# 14. 실무적 의미

## 14.1 버그를 재현할 때

명세가 다음이라고 하자.

```text
valid token → access allowed
```

이 명세가 깨지는 반례는:

```text
valid token ∧ access denied
```

반대로 보안 정책이 다음이라고 하자.

```text
invalid token → access denied
```

이 명세가 깨지는 반례는:

```text
invalid token ∧ access allowed
```

즉, implication을 이해하면 “어떤 입력이 버그를 증명하는가”를 정확히 찾을 수 있다.

---

## 14.2 테스트 케이스 설계

명세:

```text
isAdmin → canDeleteUser
```

테스트에서 가장 중요한 케이스는:

```text
isAdmin = true, canDeleteUser = false
```

왜냐하면 이 경우가 명세를 깨는 유일한 경우이기 때문이다.

하지만 보안 정책에서는 반대 방향도 중요할 수 있다.

```text
canDeleteUser → isAdmin
```

이 명세는 의미가 다르다.

```text
삭제할 수 있으면 관리자여야 한다.
```

이것의 위반은:

```text
canDeleteUser = true, isAdmin = false
```

즉, 비관리자가 삭제 가능한 경우다.

실무에서는 둘을 구분해야 한다.

---

## 14.3 권한 정책에서 방향이 중요하다

다음 두 명제는 다르다.

```text
isAdmin → canDeleteUser
```

```text
canDeleteUser → isAdmin
```

첫 번째:

```text
관리자라면 삭제할 수 있다.
```

두 번째:

```text
삭제할 수 있다면 관리자다.
```

보안상 더 중요한 것은 보통 두 번째다.

왜냐하면 “비관리자가 삭제 가능하면 안 된다”를 표현하기 때문이다.

```text
canDeleteUser → isAdmin
```

이것의 contrapositive는:

```text
¬isAdmin → ¬canDeleteUser
```

즉:

```text
관리자가 아니면 삭제할 수 없다.
```

이 형태가 보안 정책에서 매우 자연스럽다.

---

# 15. 흔한 오해

## 오해 1. `p → q`는 p와 q 사이의 인과관계다

아니다.

명제논리에서 implication은 truth table로 정의되는 논리 연산자다.

예:

```text
2 + 2 = 5 → Paris is in France
```

전건이 false이므로 전체 implication은 true다.

이 문장이 자연어로 의미 있어 보이지 않아도, 명제논리에서는 truth table에 따라 true다.

---

## 오해 2. `p → q`와 `q → p`는 같다

아니다.

```text
x가 4의 배수이면 x는 2의 배수다.
```

는 참이지만,

```text
x가 2의 배수이면 x는 4의 배수다.
```

는 거짓이다.

즉:

```text
p → q ≠ q → p
```

일반적으로 동치가 아니다.

---

## 오해 3. `p → q`와 `¬p → ¬q`는 같다

아니다.

원래 명제와 같은 것은 inverse가 아니라 contrapositive다.

```text
p → q ≡ ¬q → ¬p
```

헷갈리지 말아야 한다.

---

## 오해 4. p가 false인데 `p → q`가 true인 것은 이상하다

자연어 관점에서는 이상하게 느껴질 수 있다.

하지만 형식 논리에서는 implication을 다음처럼 정의한다.

```text
p → q ≡ ¬p ∨ q
```

만약 `p`가 false이면 `¬p`가 true다.

따라서:

```text
¬p ∨ q = true ∨ q = true
```

그래서 `p → q`는 true다.

---

## 오해 5. Precondition이 false인데 결과가 틀리면 함수가 틀린 것이다

명세가 다음이라면:

```text
precondition → postcondition
```

precondition이 false인 입력에서는 postcondition을 보장하지 않는다.

예:

```text
array sorted → binarySearch correct
```

배열이 정렬되어 있지 않다면, binary search가 틀려도 이 명세 자체를 위반한 것은 아니다.

물론 실제 소프트웨어에서는 invalid input을 따로 처리해야 할 수 있다.
하지만 그것은 다른 명세다.

---

# 16. 반례

## 반례 1. Converse가 틀리는 경우

원래 명제:

```text
x는 4의 배수다 → x는 2의 배수다
```

참이다.

Converse:

```text
x는 2의 배수다 → x는 4의 배수다
```

반례:

```text
x = 6
```

6은 2의 배수지만 4의 배수가 아니다.

따라서 converse는 틀린다.

---

## 반례 2. Inverse가 틀리는 경우

원래 명제:

```text
x는 4의 배수다 → x는 2의 배수다
```

Inverse:

```text
x는 4의 배수가 아니다 → x는 2의 배수가 아니다
```

반례:

```text
x = 6
```

6은 4의 배수는 아니지만 2의 배수다.

따라서 inverse는 틀린다.

---

## 반례 3. 권한 정책 방향 오류

다음 명세를 보자.

```text
isAdmin → canDeleteUser
```

이 명세는 다음을 보장한다.

```text
관리자는 삭제할 수 있다.
```

하지만 이것만으로는 다음을 막지 못한다.

```text
일반 사용자도 삭제할 수 있다.
```

왜냐하면 `isAdmin = false`, `canDeleteUser = true`인 경우:

| isAdmin | canDeleteUser | isAdmin → canDeleteUser |
| ------- | ------------- | ----------------------- |
| F       | T             | T                       |

이다.

보안적으로 일반 사용자를 막으려면 다음이 필요하다.

```text
canDeleteUser → isAdmin
```

또는 그 contrapositive:

```text
¬isAdmin → ¬canDeleteUser
```

---

# 17. 확인 문제

## 문제 1

다음 implication의 truth table을 작성하라.

```text
p → q
```

---

## 문제 2

`p → q`가 false가 되는 유일한 경우는 무엇인가?

---

## 문제 3

다음의 contrapositive, converse, inverse를 각각 작성하라.

```text
p → q
```

---

## 문제 4

다음 명제의 contrapositive를 작성하라.

```text
x가 4의 배수이면 x는 2의 배수다.
```

---

## 문제 5

다음 두 명제는 같은가?

```text
isAdmin → canDeleteUser
```

```text
canDeleteUser → isAdmin
```

같지 않다면 차이를 설명하라.

---

## 문제 6

다음 명세가 깨지는 입력 조건을 논리식으로 쓰라.

```text
validInput → returnsSuccess
```

힌트:

```text
p → q가 깨지는 조건은 p ∧ ¬q
```

---

# 18. 실습 과제

## 과제 1. Truth table proof

다음을 truth table로 증명하라.

```text
p → q ≡ ¬p ∨ q
```

작성 column:

```text
p
q
¬p
¬p ∨ q
p → q
```

마지막 두 column을 비교하라.

---

## 과제 2. Contrapositive 증명

다음을 truth table로 증명하라.

```text
p → q ≡ ¬q → ¬p
```

작성 column:

```text
p
q
¬p
¬q
p → q
¬q → ¬p
```

---

## 과제 3. Converse 반례 찾기

다음 명제의 converse를 작성하고, 반례를 찾아라.

```text
x가 6의 배수이면 x는 3의 배수다.
```

해야 할 것:

```text
1. p 정의
2. q 정의
3. original 작성
4. converse 작성
5. converse의 반례 제시
```

---

## 과제 4. 프로그래밍 명세 분석

다음 명세를 분석하라.

```text
isAuthenticated → canViewDashboard
```

질문:

```text
1. 이 명세는 무엇을 보장하는가?
2. 이 명세만으로 비인증 사용자의 접근을 막을 수 있는가?
3. 비인증 사용자의 접근을 막으려면 어떤 implication이 필요한가?
4. 그 implication의 contrapositive는 무엇인가?
```

힌트:

```text
비인증 사용자가 볼 수 없게 하려면:
canViewDashboard → isAuthenticated
```

---

# 19. 핵심 정리

Implication의 기본 형태:

```text
p → q
```

의미:

```text
p이면 q이다.
```

Truth table:

| p | q | p → q |
| - | - | ----- |
| T | T | T     |
| T | F | F     |
| F | T | T     |
| F | F | T     |

핵심:

```text
p → q는 p가 true이고 q가 false일 때만 false다.
```

동치식:

```text
p → q ≡ ¬p ∨ q
```

위반 조건:

```text
¬(p → q) ≡ p ∧ ¬q
```

네 가지 형태:

| 이름             | 형태        | original과 동치? |
| -------------- | --------- | ------------- |
| Original       | `p → q`   | O             |
| Contrapositive | `¬q → ¬p` | O             |
| Converse       | `q → p`   | X             |
| Inverse        | `¬p → ¬q` | X             |

가장 중요한 동치:

```text
p → q ≡ ¬q → ¬p
```

프로그래밍에서 implication은 다음과 연결된다.

```text
1. precondition / postcondition
2. assertion
3. guard condition
4. algorithm correctness
5. security policy
6. bug reproduction condition
```

특히 보안 정책에서는 방향이 중요하다.

```text
isAdmin → canDeleteUser
```

와

```text
canDeleteUser → isAdmin
```

는 다르다.

첫 번째는 “관리자는 삭제 가능하다.”
두 번째는 “삭제 가능하면 관리자여야 한다.”

비관리자의 삭제를 막으려면 보통 두 번째가 필요하다.

---

# Part A 마무리

Part A에서 배운 핵심은 다음이다.

| 주제                  | 핵심                                           |
| ------------------- | -------------------------------------------- |
| Proposition         | 참/거짓이 명확한 선언문                                |
| Logical Operators   | `∧`, `∨`, `¬`, `⊕`                           |
| Truth Table         | 모든 truth value 조합을 나열                        |
| Logical Equivalence | 모든 row에서 결과가 같음                              |
| De Morgan’s Laws    | NOT이 들어가면 AND/OR가 바뀜                         |
| Tautology           | 항상 true                                      |
| Contradiction       | 항상 false                                     |
| Implication         | `p → q`, false only when `p=true`, `q=false` |
| Contrapositive      | `p → q ≡ ¬q → ¬p`                            |


```text
1. 명제와 명제가 아닌 문장을 구분한다.
2. AND, OR, NOT, XOR을 truth table로 설명한다.
3. 복합 논리식의 truth table을 만든다.
4. logical equivalence를 truth table로 검증한다.
5. De Morgan’s Laws를 정확히 적용한다.
6. tautology, contradiction, contingency를 구분한다.
7. implication, contrapositive, converse, inverse를 구분한다.
8. C/C++ 조건문을 논리식으로 분석한다.
```