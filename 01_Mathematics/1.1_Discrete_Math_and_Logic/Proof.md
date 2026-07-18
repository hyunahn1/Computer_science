# Part B. Proof Techniques 전체 커리큘럼

| Lecture   | 주제                                     | 핵심 목표                                                    |
| --------- | -------------------------------------- | -------------------------------------------------------- |
| Lecture 1 | Proof란 무엇인가                            | example과 proof를 구분하고, 기본 증명 전략을 이해한다                     |
| Lecture 2 | Mathematical Induction                 | base case, inductive hypothesis, inductive step의 구조를 익힌다 |
| Lecture 3 | Induction Example: Sum Formula         | 실제 수식 증명을 induction으로 완성한다                               |
| Lecture 4 | Strong Induction과 Recursive Structures | 재귀, 점화식, DP correctness proof와 induction을 연결한다           |
| Lecture 5 | Pigeonhole Principle                   | 존재 증명의 핵심 원리를 이해한다                                       |
| Lecture 6 | Pigeonhole Principle Applications      | hash collision, birthday paradox, lower bound 직관에 적용한다   |

---

# Lecture 1. Proof란 무엇인가

## 1. 핵심 질문

> **“어떤 명제가 참이라는 것을 어떻게 확실하게 보장할 수 있는가?”**

컴퓨터공학에서 이 질문은 다음 질문으로 바뀐다.

> **“이 알고리즘이 모든 입력에 대해 항상 올바르게 동작한다는 것을 어떻게 보장할 수 있는가?”**

예시 몇 개를 돌려보는 것은 충분하지 않다.

```cpp
int sum_to_n(int n) {
    return n * (n + 1) / 2;
}
```

`n = 1, 2, 3, 10`에서 맞는다고 해서 모든 자연수 `n`에 대해 맞는 것은 아니다.

증명은 **모든 가능한 경우**에 대해 참임을 논리적으로 보장하는 도구이다.

---

## 2. 형식적 정의

### Proposition

**명제 proposition**는 참 또는 거짓이 명확히 정해지는 문장이다.

예:

```text
2 + 3 = 5
```

참인 명제이다.

```text
모든 짝수 n에 대해 n + 2도 짝수이다.
```

참인 명제이다.

```text
이 알고리즘은 빠르다.
```

애매하다. “빠르다”의 기준이 없으므로 형식적 명제로 보기 어렵다.

---

### Theorem, Lemma, Corollary

| 용어        | 의미                          |
| --------- | --------------------------- |
| Theorem   | 중요한 증명된 명제                  |
| Lemma     | theorem을 증명하기 위해 사용하는 보조 정리 |
| Corollary | theorem에서 거의 바로 따라오는 결과     |

예를 들어:

```text
Lemma:
두 짝수의 합은 짝수이다.

Theorem:
짝수 n에 대해 n + 2는 짝수이다.

Corollary:
짝수 n에 대해 n + 4도 짝수이다.
```

---

### Proof

**Proof**란 어떤 명제가 참임을 이미 받아들인 정의, 공리, 이전 정리, 논리 규칙을 사용해 보이는 엄밀한 논증이다.

형식적으로는 보통 다음 구조를 가진다.

```text
Assumption → Conclusion
```

즉,

```text
만약 A가 참이면, B도 참이다.
```

이를 논리 기호로 쓰면:

```text
A ⇒ B
```

---

## 3. 직관적 설명

증명은 “설득력 있는 설명”이 아니다.

증명은 다음을 만족해야 한다.

1. 출발점이 명확해야 한다.
2. 사용하는 정의가 명확해야 한다.
3. 각 단계가 논리적으로 이어져야 한다.
4. 특정 예시에만 의존하면 안 된다.
5. 모든 경우를 덮어야 한다.

예를 보자.

명제:

```text
모든 짝수 n에 대해 n + 2는 짝수이다.
```

예시 확인:

```text
2 + 2 = 4
4 + 2 = 6
10 + 2 = 12
100 + 2 = 102
```

이것은 proof가 아니다.

왜냐하면 아직 확인하지 않은 짝수가 무한히 많기 때문이다.

진짜 증명은 다음처럼 해야 한다.

```text
n이 짝수라고 하자.
짝수의 정의에 의해 어떤 정수 k가 존재해서 n = 2k이다.
그러면 n + 2 = 2k + 2 = 2(k + 1)이다.
k + 1은 정수이므로 n + 2는 2 × 정수 형태이다.
따라서 n + 2는 짝수이다.
```

여기서 중요한 것은 `2, 4, 10` 같은 예시가 아니라, **짝수의 구조 자체**를 사용했다는 점이다.

---

## 4. 왜 필요한지

### 4.1 수학에서 필요한 이유

증명은 무한히 많은 경우를 한 번에 처리한다.

예:

```text
모든 자연수 n에 대해 1 + 2 + ... + n = n(n+1)/2
```

자연수는 무한히 많다.

테스트로는 절대 전부 확인할 수 없다.

그래서 증명이 필요하다.

---

### 4.2 컴퓨터공학에서 필요한 이유

CS에서는 다음을 증명해야 한다.

| 영역       | 증명하고 싶은 것                  |
| -------- | -------------------------- |
| 알고리즘     | 항상 정답을 반환하는가?              |
| 자료구조     | 연산 후에도 구조가 깨지지 않는가?        |
| 해시 테이블   | 충돌은 피할 수 있는가?              |
| 정렬 알고리즘  | 결과가 정말 정렬되어 있는가?           |
| 그래프 알고리즘 | 최단 경로가 진짜 최단인가?            |
| 재귀 함수    | 모든 입력에서 종료하고 올바른 값을 반환하는가? |
| 보안 프로토콜  | 특정 공격이 불가능한가?              |

예를 들어 sorting algorithm을 테스트한다고 해보자.

```cpp
sort([3, 1, 2]) -> [1, 2, 3]
sort([5, 4, 3, 2, 1]) -> [1, 2, 3, 4, 5]
sort([1]) -> [1]
```

이렇게 몇 개 테스트가 통과해도, 모든 배열에 대해 올바른 정렬 결과를 보장하지는 않는다.

증명은 다음을 보여야 한다.

```text
모든 입력 배열 A에 대해,
알고리즘이 종료한 뒤의 배열은
1. 원래 배열과 같은 원소들을 가지고 있고,
2. 오름차순으로 정렬되어 있다.
```

---

## 5. 증명 구조

증명의 가장 기본 형태는 implication이다.

```text
If P, then Q.
```

즉,

```text
P ⇒ Q
```

여기서:

| 구성요소 | 의미                              |
| ---- | ------------------------------- |
| P    | assumption, hypothesis, premise |
| Q    | conclusion                      |

예:

```text
If n is even, then n + 2 is even.
```

분해하면:

| 부분         | 내용            |
| ---------- | ------------- |
| Assumption | n is even     |
| Conclusion | n + 2 is even |

증명은 assumption에서 출발해서 conclusion에 도달해야 한다.

---

# 6. 기본 증명 방식 3가지

이번 Lecture에서는 세 가지를 다룬다.

1. Direct Proof
2. Proof by Contrapositive
3. Proof by Contradiction

---

## 6.1 Direct Proof

### 형식적 정의

명제 `P ⇒ Q`를 증명할 때, `P`를 가정하고 논리적으로 `Q`를 직접 도출하는 방식이다.

```text
Assume P.
...
Therefore Q.
```

---

### 직관적 설명

가장 직선적인 증명이다.

“P가 맞다고 치자. 그러면 정의에 의해 이런 형태이고, 계산해보면 Q가 된다.”

---

### 예시

명제:

```text
모든 정수 n에 대해, n이 짝수이면 n^2도 짝수이다.
```

형식:

```text
P: n is even
Q: n^2 is even
```

증명:

```text
n이 짝수라고 하자.
짝수의 정의에 의해 어떤 정수 k가 존재해서 n = 2k이다.
그러면 n^2 = (2k)^2 = 4k^2 = 2(2k^2)이다.
2k^2는 정수이므로 n^2는 2 × 정수 형태이다.
따라서 n^2는 짝수이다.
```

---

### CS 응용

예를 들어 배열의 모든 원소가 짝수일 때, 각 원소를 제곱해도 모두 짝수임을 보일 수 있다.

```cpp
for (int i = 0; i < n; i++) {
    a[i] = a[i] * a[i];
}
```

증명하고 싶은 명제:

```text
반복문 실행 전 모든 a[i]가 짝수이면,
반복문 실행 후에도 모든 a[i]는 짝수이다.
```

각 원소 `a[i]`에 대해:

```text
a[i]가 짝수이면 a[i]^2도 짝수이다.
```

이 명제를 direct proof로 증명할 수 있다.

---

## 6.2 Proof by Contrapositive

### 형식적 정의

명제:

```text
P ⇒ Q
```

는 다음 명제와 논리적으로 동치이다.

```text
¬Q ⇒ ¬P
```

이를 **contrapositive**라고 한다.

| 원래 명제 | 대우      |
| ----- | ------- |
| P ⇒ Q | ¬Q ⇒ ¬P |

중요하다.

```text
P ⇒ Q
```

와

```text
¬Q ⇒ ¬P
```

는 항상 같은 truth value를 가진다.

---

### 직관적 설명

어떤 명제를 직접 증명하기 어렵다면 방향을 뒤집는다.

예:

```text
n^2이 짝수이면 n도 짝수이다.
```

이 명제를 직접 증명하려 하면 약간 어렵다.

그래서 대우를 증명한다.

원래 명제:

```text
If n^2 is even, then n is even.
```

대우:

```text
If n is not even, then n^2 is not even.
```

즉:

```text
If n is odd, then n^2 is odd.
```

이것은 훨씬 쉽다.

---

### 증명 구조

명제:

```text
P ⇒ Q
```

를 증명하고 싶다.

대우:

```text
¬Q ⇒ ¬P
```

를 증명한다.

구조:

```text
Assume not Q.
...
Therefore not P.
Therefore P implies Q.
```

---

### 단계별 예시

명제:

```text
정수 n에 대해, n^2이 짝수이면 n도 짝수이다.
```

분해:

```text
P: n^2 is even
Q: n is even
```

대우:

```text
¬Q: n is odd
¬P: n^2 is odd
```

증명:

```text
n이 홀수라고 하자.
홀수의 정의에 의해 어떤 정수 k가 존재해서 n = 2k + 1이다.
그러면

n^2 = (2k + 1)^2
    = 4k^2 + 4k + 1
    = 2(2k^2 + 2k) + 1

2k^2 + 2k는 정수이므로 n^2는 홀수이다.

따라서 n이 홀수이면 n^2도 홀수이다.
즉, 대우가 참이다.

그러므로 원래 명제인
n^2이 짝수이면 n도 짝수이다
도 참이다.
```

---

### CS 응용

Hash collision이나 lower bound에서 이런 사고가 자주 나온다.

예:

```text
만약 충돌이 없다면, 서로 다른 입력들이 서로 다른 출력으로 가야 한다.
```

그런데 가능한 입력 수가 가능한 출력 수보다 많다면, 충돌이 없을 수 없다.

즉:

```text
No collision ⇒ number of inputs ≤ number of outputs
```

대우:

```text
number of inputs > number of outputs ⇒ collision exists
```

이것이 pigeonhole principle의 핵심 구조와 연결된다.

---

## 6.3 Proof by Contradiction

### 형식적 정의

명제 `P`를 증명하고 싶을 때, `P`가 거짓이라고 가정한 뒤 모순을 도출한다.

```text
Assume not P.
...
Contradiction.
Therefore P.
```

---

### 직관적 설명

“반대로 생각해보자. 이 명제가 거짓이라고 하면 어떤 일이 벌어지는가?”

그 결과 불가능한 결론이 나오면, 처음의 부정 가정이 틀렸다는 뜻이다.

---

### 증명 구조

```text
To prove P:

Assume ¬P.
Derive something impossible.
Therefore ¬P is false.
Therefore P is true.
```

모순의 형태는 여러 가지가 있다.

```text
0 = 1
x < x
n is both even and odd
A set has both 3 and not 3 elements
```

---

### 단계별 예시

명제:

```text
√2는 유리수가 아니다.
```

즉:

```text
√2 is irrational.
```

증명 아이디어:

```text
√2가 유리수라고 가정한다.
그러면 √2 = a / b로 쓸 수 있다.
단, a와 b는 공약수가 없는 정수라고 하자.
양변을 제곱하면 2 = a^2 / b^2.
따라서 a^2 = 2b^2.
그러므로 a^2는 짝수이고, 따라서 a도 짝수이다.
a = 2k라고 두면,
4k^2 = 2b^2
2k^2 = b^2
따라서 b^2도 짝수이고, b도 짝수이다.
그러면 a와 b가 둘 다 짝수이므로 공약수 2를 가진다.
하지만 처음에 a와 b는 공약수가 없다고 했다.
모순이다.
따라서 √2는 유리수가 아니다.
```

이 증명은 contradiction proof의 대표적인 예시이다.

---

### CS 응용

예를 들어 binary search correctness를 생각해보자.

명제:

```text
Binary search가 target을 찾지 못했다고 반환했다면,
배열 안에 target은 존재하지 않는다.
```

모순으로 증명할 수 있다.

```text
Binary search가 "없다"고 반환했는데,
실제로 target이 배열 안에 있다고 가정하자.

그러면 어떤 index i에 대해 A[i] = target이다.

하지만 binary search는 매 단계마다
target이 존재할 수 있는 구간을 보존한다.

마지막에 구간이 비었는데도 target이 존재한다면,
target이 존재할 수 있는 구간을 보존했다는 loop invariant와 모순이다.

따라서 target은 배열 안에 존재하지 않는다.
```

여기서 핵심은 **loop invariant**이다.

---

# 7. Example과 Proof의 차이

## 7.1 Example

Example은 특정 경우를 확인한다.

명제:

```text
모든 홀수 n에 대해 n^2도 홀수이다.
```

예시:

```text
1^2 = 1
3^2 = 9
5^2 = 25
7^2 = 49
```

이것은 명제를 지지하는 evidence일 수는 있다.

하지만 proof는 아니다.

---

## 7.2 Proof

Proof는 전체 구조를 보인다.

```text
n이 홀수라고 하자.
그러면 어떤 정수 k가 존재해서 n = 2k + 1이다.

n^2 = (2k + 1)^2
    = 4k^2 + 4k + 1
    = 2(2k^2 + 2k) + 1

따라서 n^2는 홀수이다.
```

이 증명은 모든 홀수 `n`에 대해 적용된다.

---

## 7.3 핵심 차이

| 구분     | Example          | Proof             |
| ------ | ---------------- | ----------------- |
| 다루는 범위 | 특정 경우            | 모든 경우             |
| 목적     | 감 잡기, 패턴 발견      | 참임을 보장            |
| 실패 가능성 | 아직 확인 안 한 경우가 남음 | 논리적으로 모든 경우를 커버   |
| CS 대응  | 테스트 케이스          | correctness proof |

---

# 8. Counterexample의 의미

## 형식적 정의

명제:

```text
For all x, P(x)
```

를 반박하려면, `P(x)`가 거짓인 `x` 하나만 찾으면 된다.

그 하나의 예를 **counterexample**이라고 한다.

---

## 예시

명제:

```text
모든 소수는 홀수이다.
```

이 명제는 거짓이다.

반례:

```text
2는 소수이지만 짝수이다.
```

하나의 counterexample만으로도 universal statement는 무너진다.

---

## CS 응용

명제:

```text
이 정렬 함수는 모든 입력 배열을 오름차순으로 정렬한다.
```

반례:

```text
입력: [2, 1, 3]
출력: [1, 3, 2]
```

이 하나의 테스트 케이스만으로도 알고리즘 correctness claim은 거짓이 된다.

---

# 9. Implication 형태의 명제 증명

많은 수학/CS 명제는 다음 형태이다.

```text
If P, then Q.
```

예:

```text
If n is divisible by 4, then n is even.
```

증명:

```text
n이 4로 나누어진다고 하자.
그러면 어떤 정수 k가 존재해서 n = 4k이다.
따라서 n = 2(2k)이다.
2k는 정수이므로 n은 짝수이다.
```

---

## 주의: P가 거짓이면?

논리학에서 `P ⇒ Q`는 `P`가 거짓일 때 자동으로 참이다.

예:

```text
If 3 is even, then 3 is divisible by 2.
```

이 문장은 이상해 보이지만, 논리적으로는 참이다.

왜냐하면 assumption인 `3 is even`이 애초에 거짓이기 때문이다.

CS에서는 이것이 precondition과 관련 있다.

```cpp
// Precondition: n >= 0
int factorial(int n);
```

이 함수의 correctness proof는 보통 `n >= 0`인 입력에 대해서만 한다.

`n = -3` 같은 입력은 precondition 밖이다.

---

# 10. 알고리즘 Correctness Proof와의 연결

알고리즘 correctness proof는 보통 두 가지를 보인다.

## 10.1 Partial Correctness

```text
만약 알고리즘이 종료한다면, 결과는 올바르다.
```

즉:

```text
Termination assumed ⇒ Correct output
```

---

## 10.2 Termination

```text
알고리즘이 실제로 종료한다.
```

---

## 10.3 Total Correctness

```text
알고리즘이 종료하고, 결과도 올바르다.
```

즉:

```text
Partial correctness + Termination = Total correctness
```

---

## 예시: 배열 최댓값 찾기

코드:

```cpp
int max_value(int a[], int n) {
    int m = a[0];

    for (int i = 1; i < n; i++) {
        if (a[i] > m) {
            m = a[i];
        }
    }

    return m;
}
```

전제:

```text
n >= 1
```

증명하고 싶은 것:

```text
반환값은 배열 a[0...n-1]의 최댓값이다.
```

---

### Loop Invariant

반복문 correctness proof에서 핵심 도구는 **loop invariant**이다.

Loop invariant는 반복문이 매번 시작될 때마다 유지되는 성질이다.

이 코드의 loop invariant:

```text
반복문에서 index i에 도달했을 때,
m은 a[0...i-1] 구간의 최댓값이다.
```

---

### 증명 구조

| 단계             | 설명                                    |
| -------------- | ------------------------------------- |
| Initialization | 반복문 시작 전 invariant가 참인지 확인            |
| Maintenance    | 한 반복이 끝나도 invariant가 유지되는지 확인         |
| Termination    | 반복문 종료 시 invariant로부터 원하는 결론이 나오는지 확인 |

---

### 단계별 증명

#### 1. Initialization

반복문 시작 전:

```cpp
int m = a[0];
int i = 1;
```

이때 `m`은 `a[0...0]`의 최댓값이다.

따라서 invariant는 참이다.

---

#### 2. Maintenance

반복문 시작 시점에 다음이 참이라고 가정한다.

```text
m은 a[0...i-1]의 최댓값이다.
```

이제 `a[i]`를 본다.

두 경우가 있다.

| 경우          | 결과                                    |
| ----------- | ------------------------------------- |
| `a[i] > m`  | `m = a[i]`로 갱신하므로 `a[0...i]`의 최댓값이 된다 |
| `a[i] <= m` | 기존 `m`이 여전히 `a[0...i]`의 최댓값이다         |

따라서 다음 반복문 시작 시점에서:

```text
m은 a[0...i]의 최댓값이다.
```

즉 invariant가 유지된다.

---

#### 3. Termination

반복문은 `i == n`일 때 종료한다.

Invariant에 의해:

```text
m은 a[0...n-1]의 최댓값이다.
```

따라서 반환값은 전체 배열의 최댓값이다.

---

### Termination

`i`는 매 반복마다 1씩 증가한다.

```cpp
i++;
```

그리고 `i < n` 조건이 거짓이 되면 반복문이 끝난다.

`n`은 유한하므로 반복문은 종료한다.

따라서 이 알고리즘은 total correctness를 가진다.

---

# 11. 흔한 오해

## 오해 1. “예시를 많이 들면 proof다”

아니다.

예시 1,000개를 확인해도 무한히 많은 경우가 남을 수 있다.

테스트는 버그를 찾을 수 있지만, 일반적으로 버그가 없음을 증명하지는 못한다.

---

## 오해 2. “증명은 수학에서만 필요하다”

아니다.

컴퓨터공학에서 증명은 다음과 직접 연결된다.

```text
algorithm correctness
loop invariant
type safety
protocol safety
data structure invariant
complexity analysis
hash collision inevitability
```

---

## 오해 3. “Contrapositive와 contradiction은 같다”

비슷해 보이지만 다르다.

| 방식             | 구조                       |
| -------------- | ------------------------ |
| Contrapositive | `P ⇒ Q` 대신 `¬Q ⇒ ¬P`를 증명 |
| Contradiction  | 증명하려는 명제의 부정을 가정하고 모순 도출 |

예:

```text
Contrapositive:
n^2이 짝수이면 n도 짝수이다.
→ n이 홀수이면 n^2도 홀수이다.

Contradiction:
n^2이 짝수인데 n이 홀수라고 가정하면 모순이 발생한다.
```

---

## 오해 4. “Proof by contradiction은 아무 때나 쓰면 좋다”

조심해야 한다.

Contradiction proof는 강력하지만, 초보 단계에서는 논리 흐름이 흐려지기 쉽다.

가능하면 먼저 다음 순서로 생각하는 것이 좋다.

```text
1. Direct proof 가능한가?
2. Contrapositive가 더 쉬운가?
3. 그래도 어렵다면 contradiction을 쓸 수 있는가?
```

---

# 12. 반례 또는 실패 사례

## 실패 사례 1. Example을 proof로 착각

명제:

```text
모든 홀수 n에 대해 n + 2는 홀수이다.
```

예시:

```text
1 + 2 = 3
3 + 2 = 5
5 + 2 = 7
```

이것만 쓰면 proof가 아니다.

올바른 proof:

```text
n이 홀수라고 하자.
그러면 어떤 정수 k에 대해 n = 2k + 1이다.
따라서 n + 2 = 2k + 3 = 2(k + 1) + 1이다.
k + 1은 정수이므로 n + 2는 홀수이다.
```

---

## 실패 사례 2. Assumption을 명확히 쓰지 않음

나쁜 증명:

```text
n = 2k이다.
그러므로 n + 2 = 2k + 2 = 2(k + 1)이다.
따라서 짝수이다.
```

문제점:

```text
왜 n = 2k인지 설명하지 않았다.
```

좋은 증명:

```text
n이 짝수라고 가정한다.
짝수의 정의에 의해 어떤 정수 k가 존재해서 n = 2k이다.
...
```

증명에서는 “왜 이 형태로 둘 수 있는지”를 반드시 밝혀야 한다.

---

## 실패 사례 3. Conclusion을 바꿔치기함

증명해야 할 명제:

```text
n이 4의 배수이면 n은 짝수이다.
```

나쁜 증명:

```text
n = 4k이다.
따라서 n은 4의 배수이다.
```

문제점:

```text
이미 가정한 내용을 다시 말했을 뿐,
결론인 “n은 짝수이다”를 보이지 않았다.
```

좋은 증명:

```text
n = 4k = 2(2k)이다.
2k는 정수이므로 n은 짝수이다.
```

---

# 13. 확인 문제

아래 문제는 지금 바로 풀 필요는 없다. 다음에 “문제”라고 하면 같이 풀 수 있다.

## 문제 1

다음 명제는 example인가 proof인가?

```text
3^2 = 9이고 5^2 = 25이고 7^2 = 49이다.
따라서 모든 홀수 n에 대해 n^2는 홀수이다.
```

---

## 문제 2

다음 명제를 direct proof로 증명하라.

```text
모든 정수 n에 대해, n이 짝수이면 3n + 4도 짝수이다.
```

---

## 문제 3

다음 명제를 contrapositive로 증명하라.

```text
정수 n에 대해, n^2이 홀수이면 n도 홀수이다.
```

힌트:

```text
원래 명제: n^2 odd ⇒ n odd
대우: n not odd ⇒ n^2 not odd
즉: n even ⇒ n^2 even
```

---

## 문제 4

다음 주장이 참인지 거짓인지 판단하라. 거짓이면 counterexample을 제시하라.

```text
모든 정수 n에 대해, n^2이 짝수이면 n은 4의 배수이다.
```

---

# 14. 실습 과제

## 실습 1. Proof Template 작성

다음 template을 사용해서 직접 증명을 써라.

```text
Claim:
If __________, then __________.

Proof:
Assume __________.
By definition of __________, there exists __________ such that __________.
Then __________.
Therefore __________.
```

추천 명제:

```text
If n is divisible by 6, then n is divisible by 3.
```

---

## 실습 2. Code Correctness 연결

다음 코드에 대해 loop invariant를 작성해보라.

```cpp
int sum_array(int a[], int n) {
    int s = 0;

    for (int i = 0; i < n; i++) {
        s += a[i];
    }

    return s;
}
```

증명하고 싶은 명제:

```text
함수는 a[0] + a[1] + ... + a[n-1]을 반환한다.
```

힌트:

```text
반복문에서 index i에 도달했을 때,
s는 어느 구간의 합인가?
```

---

# 15. 핵심 정리

이번 Lecture의 핵심은 다음이다.

```text
Example은 특정 경우를 보여준다.
Proof는 모든 경우를 보장한다.
```

증명의 기본 구조는 다음이다.

```text
Assumption → Logical steps → Conclusion
```

가장 중요한 세 가지 증명 방식은 다음이다.

| 증명 방식          | 구조               | 언제 유용한가                       |
| -------------- | ---------------- | ----------------------------- |
| Direct Proof   | P를 가정하고 Q를 직접 도출 | 정의를 바로 사용할 수 있을 때             |
| Contrapositive | ¬Q를 가정하고 ¬P를 도출  | 원래 방향보다 반대 방향이 쉬울 때           |
| Contradiction  | 부정을 가정하고 모순 도출   | 존재성, 불가능성, irrationality 증명 등 |

컴퓨터공학에서는 proof가 다음과 연결된다.

```text
test case ≠ correctness proof
loop invariant = 반복문 증명의 핵심 도구
partial correctness + termination = total correctness
```