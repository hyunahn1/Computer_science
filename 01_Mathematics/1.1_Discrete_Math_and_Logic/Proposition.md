# Part A. Propositions & Logic 전체 커리큘럼

| Lecture   | 주제                                    | 핵심 목표                                                |
| --------- | ------------------------------------- | ---------------------------------------------------- |
| Lecture 1 | Proposition이란 무엇인가                    | 참/거짓이 명확한 문장을 구분한다                                   |
| Lecture 2 | Logical Operators                     | AND, OR, NOT, XOR의 의미를 정확히 이해한다                      |
| Lecture 3 | Truth Tables                          | 모든 경우를 나열해 논리식을 검증한다                                 |
| Lecture 4 | Logical Equivalence와 De Morgan’s Laws | 논리식이 같은 의미인지 증명한다                                    |
| Lecture 5 | Tautology와 Contradiction              | 항상 참, 항상 거짓, 경우에 따라 다른 명제를 구분한다                      |
| Lecture 6 | Conditional Statements                | implication, contrapositive, converse, inverse를 구분한다 |

---

# Lecture 1. Proposition이란 무엇인가

## 1. 핵심 질문

> 어떤 문장은 왜 논리학에서 다룰 수 있고, 어떤 문장은 왜 다룰 수 없는가?

명제 논리에서 가장 먼저 해야 할 일은 **문장을 논리적으로 계산 가능한 단위로 바꾸는 것**이다.

즉, 우리는 다음을 구분해야 한다.

| 문장                             | 명제인가? | 이유              |
| ------------------------------ | ----: | --------------- |
| `2 + 2 = 4`                    |     O | 참이 명확하다         |
| `5 > 10`                       |     O | 거짓이 명확하다        |
| `x > 3`                        |     X | `x` 값이 정해지지 않았다 |
| `"This sentence is beautiful"` |  보통 X | 아름답다는 기준이 주관적이다 |

---

## 2. 형식적 정의

### Proposition, 명제

**명제 proposition**란 다음 조건을 만족하는 문장이다.

> 정확히 하나의 truth value, 즉 **True 또는 False**를 가지는 선언문.

수학적으로는 이렇게 말할 수 있다.

```text
A proposition is a declarative statement that is either true or false, but not both.
```

한국어로 풀면:

> 명제는 참이거나 거짓인 문장이다.
> 참과 거짓이 동시에 될 수 없고, 참인지 거짓인지 판단 불가능하면 명제가 아니다.

---

## 3. 직관적 설명

명제는 논리학에서의 **계산 가능한 문장 단위**다.

프로그래밍으로 비유하면 명제는 결국 `bool` 값으로 평가될 수 있어야 한다.

```cpp
2 + 2 == 4     // true
5 > 10        // false
```

이런 것은 명제와 비슷하다.

하지만 다음은 다르다.

```cpp
x > 3
```

이 식은 `x`가 정해지기 전에는 true인지 false인지 알 수 없다.

예를 들어:

```cpp
int x = 5;
x > 3;   // true
```

```cpp
int x = 1;
x > 3;   // false
```

따라서 `x > 3` 자체는 **open sentence**, 즉 열린 문장이다.
`x`에 값이 주어져야 명제가 된다.

---

## 4. 왜 필요한지

명제를 정확히 구분하지 못하면 이후 모든 논리 개념이 흔들린다.

논리 연산자는 명제 위에서 작동한다.

```text
A ∧ B
A ∨ B
¬A
A → B
```

여기서 `A`, `B`는 반드시 truth value를 가져야 한다.

즉:

```text
A = "2 + 2 = 4"
B = "5 > 10"
```

이면 가능하다.

하지만:

```text
A = "This sentence is beautiful"
B = "x > 3"
```

이면 엄밀한 명제 논리에서는 바로 사용할 수 없다.

왜냐하면 `A`와 `B`가 참인지 거짓인지 확정되지 않았기 때문이다.

---

## 5. Statement와 Proposition의 차이

### Statement

**statement**는 일반적인 문장이다.

예:

```text
Close the door.
What time is it?
This movie is interesting.
2 + 2 = 4.
```

하지만 모든 statement가 proposition은 아니다.

### Proposition

**proposition**은 statement 중에서 참/거짓이 명확한 것만 말한다.

| 문장                        | Statement? | Proposition? | 이유       |
| ------------------------- | ---------: | -----------: | -------- |
| `2 + 2 = 4`               |          O |            O | 참        |
| `5 > 10`                  |          O |            O | 거짓       |
| `Close the door.`         |          O |            X | 명령문이다    |
| `What time is it?`        |          O |            X | 질문이다     |
| `This food is delicious.` |          O |         보통 X | 주관적 판단이다 |
| `x > 3`                   |          O |            X | 변수 값이 없다 |

---

## 6. Subjective sentence가 proposition이 아닌 이유

다음 문장을 보자.

```text
This sentence is beautiful.
```

이 문장은 왜 명제가 아닌가?

이유는 **truth value가 객관적으로 정해지지 않기 때문**이다.

누군가는 beautiful이라고 느낄 수 있고, 누군가는 그렇지 않다고 느낄 수 있다.

즉, 다음과 같이 판단 기준이 고정되어 있지 않다.

```text
beautiful = ?
```

논리학에서는 이런 문장을 바로 계산할 수 없다.

다만 기준을 명확히 정하면 명제로 만들 수 있다.

예를 들어:

```text
This sentence contains more than 3 words.
```

이 문장은 명제가 될 수 있다.
왜냐하면 단어 수를 세어서 true/false를 결정할 수 있기 때문이다.

---

## 7. Open Sentence와 Proposition의 차이

### Open sentence

**open sentence**는 변수를 포함하고 있어서, 변수 값이 정해지기 전에는 참/거짓을 알 수 없는 문장이다.

예:

```text
x > 3
```

이것은 아직 명제가 아니다.

왜냐하면 `x`가 무엇인지 모르기 때문이다.

하지만 값을 넣으면 명제가 된다.

| x 값 | 문장       | Truth value |
| --: | -------- | ----------- |
|   5 | `5 > 3`  | True        |
|   3 | `3 > 3`  | False       |
|  -1 | `-1 > 3` | False       |

따라서:

```text
x > 3
```

은 open sentence이고,

```text
5 > 3
```

은 proposition이다.

---

## 8. Truth table / 단계별 계산

Lecture 1에서는 아직 AND/OR를 본격적으로 다루지 않지만, 명제가 truth value를 가져야 한다는 감각을 잡기 위해 작은 표를 보자.

### 예제 1

```text
P = "2 + 2 = 4"
```

| P           | Truth value |
| ----------- | ----------- |
| `2 + 2 = 4` | True        |

이것은 명제다.

---

### 예제 2

```text
Q = "5 > 10"
```

| Q        | Truth value |
| -------- | ----------- |
| `5 > 10` | False       |

이것도 명제다.
중요한 점은 **거짓인 문장도 명제**라는 것이다.

명제는 “참인 문장”이 아니다.

명제는:

```text
참 또는 거짓이 명확한 문장
```

이다.

---

### 예제 3

```text
R = "x > 3"
```

|  x | R     |
| -: | ----- |
|  1 | False |
|  3 | False |
|  5 | True  |

`R`은 `x`에 따라 값이 바뀐다.

그래서 `x`가 정해지기 전에는 proposition이 아니다.
정확히는 **predicate** 또는 **open sentence**에 가깝다.

---

## 9. 프로그래밍 예시

C/C++에서 boolean expression은 proposition과 매우 비슷하다.

```cpp
#include <iostream>

int main() {
    bool p = (2 + 2 == 4);
    bool q = (5 > 10);

    std::cout << p << "\n"; // 1
    std::cout << q << "\n"; // 0
}
```

여기서:

```cpp
2 + 2 == 4
```

는 평가되면 `true`.

```cpp
5 > 10
```

는 평가되면 `false`.

따라서 둘 다 명제처럼 볼 수 있다.

---

하지만 다음은 상황이 다르다.

```cpp
int x;
bool r = (x > 3);
```

수학적으로 `x > 3`은 `x`가 정해져야 명제가 된다.

프로그래밍에서도 마찬가지다.

```cpp
int x = 5;
bool r = (x > 3); // true
```

```cpp
int x = 1;
bool r = (x > 3); // false
```

즉, 프로그래밍의 boolean expression은 보통 다음 형태다.

```text
상태 + 표현식 → truth value
```

수학적으로는:

```text
variable assignment + open sentence → proposition
```

---

## 10. 실무적 의미

명제 구분은 단순한 수학 문제가 아니다.

실무에서 조건문을 읽을 때도 매우 중요하다.

예를 들어:

```cpp
if (user.isAdmin() && file.exists()) {
    deleteFile(file);
}
```

여기서 조건은 두 개의 boolean expression으로 구성된다.

```cpp
user.isAdmin()
file.exists()
```

이 각각은 실행 시점에서 true 또는 false가 된다.

따라서 논리적으로는 다음과 같이 볼 수 있다.

```text
A = user is admin
B = file exists

if (A ∧ B) then delete file
```

이런 식으로 조건문을 명제로 바꿔 생각할 수 있어야 한다.

그래야 나중에 다음을 정확히 할 수 있다.

* 복잡한 조건문 단순화
* 잘못된 조건 반례 찾기
* 권한 검사 버그 분석
* 알고리즘 correctness 증명
* circuit logic 이해
* compiler optimization 이해

---

## 11. 흔한 오해

### 오해 1. “명제는 참인 문장이다”

아니다.

명제는 **참 또는 거짓이 명확한 문장**이다.

```text
5 > 10
```

은 거짓이지만 명제다.

---

### 오해 2. “질문도 명제다”

아니다.

```text
What time is it?
```

은 질문이다.
참/거짓으로 평가할 수 없다.

---

### 오해 3. “명령문도 명제다”

아니다.

```text
Close the door.
```

은 명령이다.
참/거짓이 아니다.

---

### 오해 4. `x > 3`은 항상 명제다

엄밀히는 아니다.

`x` 값이 정해지지 않으면 open sentence다.

다만 프로그래밍에서는 실행 시점에 `x`가 어떤 값을 가지고 있기 때문에 boolean expression으로 평가될 수 있다.

---

### 오해 5. 주관적 문장은 절대 명제가 될 수 없다

무조건 그렇지는 않다.

```text
This movie is good.
```

은 기준이 불명확하므로 보통 명제가 아니다.

하지만 기준을 정하면 명제가 될 수 있다.

예:

```text
This movie has a rating above 8.0 on a given database.
```

이제 객관적 기준이 생겼다.
따라서 true/false를 판단할 수 있다.

---

## 12. 반례

명제인지 아닌지 판단할 때 반례를 잘 봐야 한다.

### 문장 1

```text
All humans are immortal.
```

이 문장은 명제인가?

답: **명제다.**

왜냐하면 거짓이지만 참/거짓이 명확하기 때문이다.

중요한 점:

```text
틀린 문장 ≠ 명제가 아닌 문장
```

거짓이어도 명제일 수 있다.

---

### 문장 2

```text
This code is clean.
```

보통 명제가 아니다.

왜냐하면 “clean”의 기준이 불명확하다.

하지만 다음처럼 바꾸면 명제가 될 수 있다.

```text
This code has no function longer than 30 lines.
```

이제 기준이 명확하다.

---

### 문장 3

```text
x is an even number.
```

이것은 open sentence다.

`x = 4`이면 true.
`x = 5`이면 false.

따라서 `x`가 정해지기 전에는 명제가 아니다.

---

## 13. 확인 문제

다음 문장이 명제인지 판단하라.

| 번호 | 문장                                                          | 명제인가? |
| -: | ----------------------------------------------------------- | ----- |
|  1 | `10 is greater than 3`                                      | ?     |
|  2 | `Please open the window`                                    | ?     |
|  3 | `x + 1 = 5`                                                 | ?     |
|  4 | `Paris is the capital of France`                            | ?     |
|  5 | `This song is amazing`                                      | ?     |
|  6 | `Every even number greater than 2 is the sum of two primes` | ?     |
|  7 | `The program terminated with exit code 0`                   | ?     |
|  8 | `This variable has a beautiful name`                        | ?     |

특히 6번을 주의해야 한다.
어떤 문장이 아직 증명되지 않았거나 어렵더라도, 참/거짓이 객관적으로 정해져 있다면 명제다.

---

## 14. 실습 과제

다음 C/C++ 조건문을 보고, 각각의 atomic proposition을 분리하라.

### 예제 코드

```cpp
if (isLoggedIn && hasPermission && fileExists) {
    openFile();
}
```

해야 할 일:

```text
A = ?
B = ?
C = ?
전체 조건 = ?
```

예상 답 형태:

```text
A = user is logged in
B = user has permission
C = file exists

전체 조건 = A ∧ B ∧ C
```

---

### 추가 실습

다음 문장을 명제로 바꿀 수 있는지 판단하라.

```text
This algorithm is fast.
```

그냥 보면 명제가 아니다.
하지만 기준을 넣으면 명제가 될 수 있다.

예:

```text
This algorithm runs in O(n log n) time.
```

또는:

```text
This algorithm processes 1 million inputs in under 1 second on machine M.
```

이제 참/거짓을 판단할 수 있다.

---

## 15. 핵심 정리

Lecture 1의 핵심은 다음이다.

```text
명제 = 참 또는 거짓이 명확하게 결정되는 선언문
```

중요한 구분:

| 대상            | 명제인가? | 이유                 |
| ------------- | ----: | ------------------ |
| 참인 문장         |     O | truth value가 True  |
| 거짓인 문장        |     O | truth value가 False |
| 질문            |     X | 참/거짓이 아니다          |
| 명령문           |     X | 참/거짓이 아니다          |
| 주관적 문장        |  보통 X | 기준이 불명확하다          |
| 변수 포함 문장      |  보통 X | 값이 정해져야 한다         |
| 변수에 값이 대입된 조건 |     O | true/false 평가 가능   |

프로그래밍 관점에서 보면:

```text
boolean expression은 실행 시점의 상태가 주어졌을 때 proposition처럼 평가된다.
```

수학적으로 보면:

```text
open sentence + variable assignment = proposition
```
