# Lecture 2. Mathematical Induction

## 1. 핵심 질문

> **무한히 많은 자연수 명제 `P(1), P(2), P(3), ...`를 어떻게 한 번에 증명할 수 있는가?**

예를 들어 이런 명제를 생각해보자.

```text
모든 자연수 n ≥ 1에 대해,
1 + 2 + ... + n = n(n + 1) / 2 이다.
```

자연수는 무한히 많다.

```text
n = 1 확인
n = 2 확인
n = 3 확인
n = 100 확인
```

이렇게 해도 proof가 아니다.

그래서 필요한 도구가 **mathematical induction**, 즉 **수학적 귀납법**이다.

---

# 2. 형식적 정의

## Mathematical Induction

자연수 `n`에 대한 명제 `P(n)`이 있다고 하자.

다음 두 가지를 증명하면:

```text
1. Base Case:
   P(1)이 참이다.

2. Inductive Step:
   모든 k ≥ 1에 대해,
   P(k)가 참이라고 가정하면 P(k + 1)도 참이다.
```

그러면 결론적으로:

```text
모든 자연수 n ≥ 1에 대해 P(n)이 참이다.
```

즉:

```text
P(1)
P(1) ⇒ P(2)
P(2) ⇒ P(3)
P(3) ⇒ P(4)
...
```

이 사슬이 끊기지 않으면 모든 자연수에 대해 참이다.

---

## 수식 구조

```text
To prove: ∀n ≥ 1, P(n)

1. Base Case:
   Prove P(1).

2. Inductive Hypothesis:
   Assume P(k) is true for some arbitrary k ≥ 1.

3. Inductive Step:
   Prove P(k + 1).

4. Conclusion:
   Therefore, by mathematical induction,
   P(n) is true for all n ≥ 1.
```

---

# 3. 직관적 설명

수학적 귀납법은 흔히 **도미노**에 비유된다.

```text
[1] → [2] → [3] → [4] → [5] → ...
```

모든 도미노가 쓰러지려면 두 가지가 필요하다.

| 귀납법 요소         | 도미노 비유                  |
| -------------- | ----------------------- |
| Base Case      | 첫 번째 도미노를 직접 밀기         |
| Inductive Step | 하나가 쓰러지면 다음 것도 쓰러진다는 규칙 |
| Conclusion     | 따라서 모든 도미노가 쓰러진다        |

중요한 점:

```text
첫 번째 도미노만 밀어도 부족하다.
```

왜냐하면 두 번째, 세 번째로 전달되는 규칙이 없으면 첫 번째만 쓰러질 수 있다.

반대로:

```text
하나가 쓰러지면 다음 것도 쓰러진다
```

만 보여도 부족하다.

왜냐하면 첫 번째 도미노가 애초에 쓰러지지 않으면 아무 일도 시작되지 않기 때문이다.

따라서 귀납법에는 반드시 두 가지가 모두 필요하다.

```text
Base Case + Inductive Step
```

---

# 4. 왜 필요한지

## 4.1 무한한 경우를 다루기 위해

자연수 전체에 대한 명제는 무한하다.

```text
P(1), P(2), P(3), P(4), ...
```

각각을 따로 증명할 수 없다.

귀납법은 다음 구조로 무한한 경우를 압축한다.

```text
시작점 하나 증명
+
다음으로 넘어가는 규칙 증명
=
전체 증명
```

---

## 4.2 반복문과 재귀를 증명하기 위해

컴퓨터공학에서 induction은 다음과 직접 연결된다.

| CS 개념            | induction과의 관계                      |
| ---------------- | ----------------------------------- |
| 반복문              | `i = 0, 1, 2, ...` 단계별 상태 증명        |
| 재귀 함수            | 작은 입력이 맞으면 큰 입력도 맞다는 구조             |
| 자료구조             | 크기 `n`인 구조에 대한 성질 증명                |
| 알고리즘 correctness | 모든 입력 크기에 대해 올바름 증명                 |
| DP               | 작은 subproblem의 정답으로 큰 subproblem 증명 |
| Tree traversal   | 작은 subtree가 맞으면 전체 tree도 맞음         |

예를 들어 반복문은 자연스럽게 귀납 구조를 가진다.

```cpp
for (int i = 0; i < n; i++) {
    // do something
}
```

반복문 correctness proof는 보통 이런 식이다.

```text
i = 0에서 성질이 참이다.
i = k에서 성질이 참이면,
i = k + 1에서도 성질이 참이다.
따라서 모든 i에 대해 성질이 유지된다.
```

이것은 induction과 같은 구조이다.

---

# 5. 증명 구조

수학적 귀납법의 표준 template은 다음과 같다.

```text
Claim:
For all integers n ≥ 1, P(n) is true.

Proof by induction on n.

Base Case:
Show P(1).

Inductive Hypothesis:
Assume P(k) is true for an arbitrary integer k ≥ 1.

Inductive Step:
Using the inductive hypothesis, prove P(k + 1).

Conclusion:
Therefore, by mathematical induction, P(n) is true for all integers n ≥ 1.
```

여기서 매우 중요한 부분은:

```text
Using the inductive hypothesis
```

이다.

귀납 가정을 사용하지 않으면 induction proof가 아니다.

---

# 6. 핵심 구성 요소

## 6.1 Base Case

Base case는 증명의 시작점이다.

보통 자연수 명제에서는 다음 중 하나가 된다.

```text
P(0)
P(1)
```

문제에서 `n ≥ 0`이면 `P(0)`부터 시작한다.

문제에서 `n ≥ 1`이면 `P(1)`부터 시작한다.

---

### 예시

명제:

```text
모든 n ≥ 1에 대해 2^n ≥ n + 1
```

Base case:

```text
n = 1일 때,

2^1 = 2
1 + 1 = 2

따라서 2^1 ≥ 1 + 1 이므로 P(1)은 참이다.
```

---

## 6.2 Inductive Hypothesis

Inductive hypothesis는 다음 형태이다.

```text
어떤 임의의 k ≥ 1에 대해 P(k)가 참이라고 가정한다.
```

중요한 점:

```text
P(k)를 증명하는 것이 아니다.
P(k)가 이미 참이라고 가정하고 P(k + 1)을 증명하는 것이다.
```

초보자가 여기서 자주 헷갈린다.

Induction에서는 `P(k)`를 가정해도 된다.

왜냐하면 증명의 논리 구조가:

```text
P(k) ⇒ P(k + 1)
```

을 보이는 것이기 때문이다.

---

## 6.3 Inductive Step

Inductive step은 다음을 증명한다.

```text
P(k)가 참이면 P(k + 1)도 참이다.
```

즉:

```text
P(k) ⇒ P(k + 1)
```

이 부분에서 반드시 inductive hypothesis를 사용해야 한다.

---

# 7. 단계별 예시 1

## 명제

```text
모든 자연수 n ≥ 1에 대해 2^n ≥ n + 1이다.
```

즉:

```text
P(n): 2^n ≥ n + 1
```

---

## Proof by Induction

### Step 1. Base Case

`n = 1`일 때:

```text
2^1 = 2
1 + 1 = 2
```

따라서:

```text
2^1 ≥ 1 + 1
```

즉 `P(1)`은 참이다.

---

### Step 2. Inductive Hypothesis

어떤 임의의 정수 `k ≥ 1`에 대해 `P(k)`가 참이라고 가정한다.

즉:

```text
2^k ≥ k + 1
```

이라고 가정한다.

이것이 inductive hypothesis이다.

---

### Step 3. Inductive Step

증명해야 할 것은 `P(k + 1)`이다.

즉:

```text
2^(k + 1) ≥ (k + 1) + 1
```

다시 쓰면:

```text
2^(k + 1) ≥ k + 2
```

이제 시작한다.

```text
2^(k + 1) = 2 · 2^k
```

inductive hypothesis에 의해:

```text
2^k ≥ k + 1
```

따라서 양변에 2를 곱하면:

```text
2 · 2^k ≥ 2(k + 1)
```

즉:

```text
2^(k + 1) ≥ 2k + 2
```

그리고 `k ≥ 1`이므로:

```text
2k + 2 ≥ k + 2
```

따라서:

```text
2^(k + 1) ≥ k + 2
```

즉 `P(k + 1)`이 참이다.

---

### Step 4. Conclusion

Base case `P(1)`이 참이고, `P(k) ⇒ P(k + 1)`도 참이다.

따라서 수학적 귀납법에 의해:

```text
모든 자연수 n ≥ 1에 대해 2^n ≥ n + 1
```

이다.

---

# 8. 단계별 예시 2

## 명제

```text
모든 자연수 n ≥ 1에 대해 1 + 3 + 5 + ... + (2n - 1) = n^2 이다.
```

즉, 처음 `n`개의 홀수 합은 `n^2`이다.

```text
1 = 1^2
1 + 3 = 4 = 2^2
1 + 3 + 5 = 9 = 3^2
1 + 3 + 5 + 7 = 16 = 4^2
```

예시는 패턴을 보여줄 뿐이다.

이제 proof를 한다.

---

## Proof by Induction

### Step 1. Base Case

`n = 1`일 때:

```text
Left side = 1
Right side = 1^2 = 1
```

따라서 `P(1)`은 참이다.

---

### Step 2. Inductive Hypothesis

어떤 임의의 `k ≥ 1`에 대해 `P(k)`가 참이라고 가정한다.

즉:

```text
1 + 3 + 5 + ... + (2k - 1) = k^2
```

이라고 가정한다.

---

### Step 3. Inductive Step

증명해야 할 것은 `P(k + 1)`이다.

즉:

```text
1 + 3 + 5 + ... + (2k - 1) + (2(k + 1) - 1) = (k + 1)^2
```

마지막 항을 정리하면:

```text
2(k + 1) - 1 = 2k + 1
```

따라서 증명 목표는:

```text
1 + 3 + 5 + ... + (2k - 1) + (2k + 1) = (k + 1)^2
```

이제 왼쪽을 본다.

```text
1 + 3 + 5 + ... + (2k - 1) + (2k + 1)
```

inductive hypothesis에 의해:

```text
1 + 3 + 5 + ... + (2k - 1) = k^2
```

따라서:

```text
1 + 3 + 5 + ... + (2k - 1) + (2k + 1)
= k^2 + (2k + 1)
= k^2 + 2k + 1
= (k + 1)^2
```

따라서 `P(k + 1)`이 참이다.

---

### Step 4. Conclusion

Base case와 inductive step이 모두 성립하므로:

```text
모든 자연수 n ≥ 1에 대해
1 + 3 + 5 + ... + (2n - 1) = n^2
```

이다.

---

# 9. CS 응용

## 9.1 반복문 correctness와 induction

다음 코드를 보자.

```cpp
int sum_array(int a[], int n) {
    int s = 0;

    for (int i = 0; i < n; i++) {
        s += a[i];
    }

    return s;
}
```

증명하고 싶은 것:

```text
함수는 a[0] + a[1] + ... + a[n - 1]을 반환한다.
```

---

## Loop Invariant

반복문에서 index `i`에 도달했을 때:

```text
s = a[0] + a[1] + ... + a[i - 1]
```

즉:

```text
s는 처음 i개 원소의 합이다.
```

---

## Induction 구조로 보기

### Base Case

반복문 시작 전 `i = 0`이다.

이때:

```text
s = 0
```

처음 0개 원소의 합은 0이다.

따라서 invariant는 참이다.

---

### Inductive Hypothesis

반복문이 어떤 `i = k`에 도달했을 때 다음이 참이라고 가정한다.

```text
s = a[0] + a[1] + ... + a[k - 1]
```

---

### Inductive Step

반복문 body가 실행된다.

```cpp
s += a[i];
```

즉 `i = k`일 때:

```text
s = a[0] + a[1] + ... + a[k - 1] + a[k]
```

따라서 다음 반복 시작 시점 `i = k + 1`에서는:

```text
s = a[0] + a[1] + ... + a[k]
```

즉 처음 `k + 1`개 원소의 합이다.

따라서 invariant가 유지된다.

---

### Termination

반복문은 `i = n`일 때 종료한다.

Invariant에 의해:

```text
s = a[0] + a[1] + ... + a[n - 1]
```

따라서 함수는 전체 배열의 합을 반환한다.

---

## 9.2 재귀 함수 correctness와 induction

다음 재귀 함수를 보자.

```cpp
int factorial(int n) {
    if (n == 0) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

증명하고 싶은 것:

```text
모든 n ≥ 0에 대해 factorial(n)은 n!을 반환한다.
```

---

## Induction 구조

### Base Case

`n = 0`일 때:

```cpp
factorial(0) returns 1
```

그리고:

```text
0! = 1
```

따라서 맞다.

---

### Inductive Hypothesis

어떤 `k ≥ 0`에 대해:

```text
factorial(k) = k!
```

라고 가정한다.

---

### Inductive Step

`n = k + 1`일 때:

```cpp
factorial(k + 1)
= (k + 1) * factorial(k)
```

inductive hypothesis에 의해:

```text
factorial(k) = k!
```

따라서:

```text
factorial(k + 1)
= (k + 1) * k!
= (k + 1)!
```

따라서 `P(k + 1)`도 참이다.

---

# 10. 흔한 오해

## 오해 1. Base case를 빼먹어도 된다

안 된다.

다음 구조만 보이면 부족하다.

```text
P(k) ⇒ P(k + 1)
```

왜냐하면 시작점이 없기 때문이다.

도미노 규칙은 있는데 첫 도미노를 밀지 않은 상태와 같다.

---

### 실패 예시

명제:

```text
모든 자연수 n ≥ 1에 대해 n = n + 1이다.
```

이 명제는 거짓이다.

하지만 이상한 방식으로 inductive step만 쓰면 이런 오류가 생긴다.

```text
P(k): k = k + 1 이라고 가정한다.

양변에 1을 더하면:

k + 1 = k + 2

따라서 P(k + 1)이 참이다.
```

Inductive step 자체는 `P(k)`가 참이라는 가정하에서는 형식적으로 이어진다.

하지만 base case가 없다.

`P(1)`은:

```text
1 = 2
```

거짓이다.

따라서 전체 명제는 증명되지 않는다.

---

## 오해 2. Inductive hypothesis를 사용하지 않아도 된다

Induction proof에서 inductive step은 반드시 다음 구조여야 한다.

```text
P(k)를 가정하고 P(k + 1)을 증명한다.
```

그런데 `P(k)`를 전혀 쓰지 않으면 induction의 핵심을 사용하지 않은 것이다.

물론 어떤 명제는 induction 없이도 direct proof로 증명될 수 있다.

하지만 induction proof라고 쓰려면 `P(k)`가 실제로 역할을 해야 한다.

---

## 오해 3. `P(k)`를 “모든 k에 대해 참”이라고 가정하면 안 된다

Inductive hypothesis는:

```text
임의의 고정된 k에 대해 P(k)가 참이라고 가정한다.
```

이지,

```text
모든 k에 대해 P(k)가 이미 참이라고 가정한다.
```

가 아니다.

후자는 증명하려는 결론을 미리 가정하는 circular reasoning이다.

---

## 오해 4. `P(k)`에서 `P(k + 1)`로 넘어갈 때 목표식을 모른다

귀납법에서 가장 중요한 습관은 이것이다.

```text
먼저 P(k + 1)이 무엇인지 정확히 써라.
```

예를 들어:

```text
P(n): 1 + 3 + ... + (2n - 1) = n^2
```

이면:

```text
P(k): 1 + 3 + ... + (2k - 1) = k^2
```

그리고:

```text
P(k + 1):
1 + 3 + ... + (2k - 1) + (2k + 1) = (k + 1)^2
```

이렇게 목표를 명확히 써야 한다.

---

# 11. 반례 또는 실패 사례

## 실패 사례 1. Base case 없음

잘못된 증명:

```text
Claim:
모든 n ≥ 1에 대해 2^n > 1000이다.

Inductive Step:
2^k > 1000이라고 가정하면,
2^(k + 1) = 2 · 2^k > 2000 > 1000이다.

따라서 모든 n ≥ 1에 대해 2^n > 1000이다.
```

문제점:

```text
P(1)이 거짓이다.
```

실제로:

```text
2^1 = 2
```

이고:

```text
2 > 1000
```

은 거짓이다.

Base case가 없으면 명제가 시작되지 않는다.

---

## 실패 사례 2. Inductive hypothesis를 안 씀

명제:

```text
모든 n ≥ 1에 대해 1 + 2 + ... + n = n(n + 1)/2
```

잘못된 inductive step:

```text
P(k + 1)을 보이면 된다.
1 + 2 + ... + k + (k + 1) = (k + 1)(k + 2)/2이다.
따라서 참이다.
```

문제점:

```text
왜 왼쪽 합이 오른쪽과 같은지 보이지 않았다.
```

`P(k)`를 사용해서 다음처럼 해야 한다.

```text
1 + 2 + ... + k + (k + 1)
= k(k + 1)/2 + (k + 1)
= (k + 1)(k + 2)/2
```

여기서 첫 등호가 inductive hypothesis를 사용하는 부분이다.

---

## 실패 사례 3. `k`를 특정 숫자로 착각

잘못된 생각:

```text
P(k)를 가정한다.
예를 들어 k = 5라고 하자.
P(6)을 보이면 된다.
```

문제점:

```text
k는 특정 숫자가 아니라 임의의 자연수이다.
```

귀납 단계는 모든 `k ≥ 1`에 대해 작동해야 한다.

정확한 표현:

```text
Let k ≥ 1 be arbitrary.
Assume P(k) is true.
Then prove P(k + 1).
```

---

# 12. 확인 문제

## 문제 1

다음 증명은 왜 틀렸는가?

```text
Claim:
모든 n ≥ 1에 대해 n^2 ≥ 10이다.

Inductive Step:
n = k에서 k^2 ≥ 10이라고 가정한다.
그러면 (k + 1)^2 > k^2 ≥ 10이다.
따라서 P(k + 1)이 참이다.
그러므로 모든 n ≥ 1에 대해 n^2 ≥ 10이다.
```

힌트:

```text
Base case를 확인하라.
```

---

## 문제 2

다음 명제를 induction으로 증명하라.

```text
모든 n ≥ 1에 대해 1 + 2 + ... + n = n(n + 1)/2
```

이 문제는 Lecture 3에서 정식으로 깊게 다룬다.

---

## 문제 3

다음 명제를 induction으로 증명하라.

```text
모든 n ≥ 1에 대해 2^n ≥ n
```

---

## 문제 4

다음 코드의 correctness를 induction 구조로 설명하라.

```cpp
int power2(int n) {
    int result = 1;

    for (int i = 0; i < n; i++) {
        result *= 2;
    }

    return result;
}
```

증명하고 싶은 것:

```text
power2(n)은 2^n을 반환한다.
```

힌트:

```text
반복문에서 i에 도달했을 때 result는 무엇인가?
```

---

# 13. 실습 과제

## 실습 1. Induction Proof Template 채우기

다음 명제를 증명하라.

```text
모든 n ≥ 1에 대해 3^n ≥ 2n + 1이다.
```

Template:

```text
Claim:
For all n ≥ 1, 3^n ≥ 2n + 1.

Proof by induction on n.

Base Case:
n = 1일 때, __________.
따라서 P(1)은 참이다.

Inductive Hypothesis:
어떤 임의의 k ≥ 1에 대해 __________ 라고 가정한다.

Inductive Step:
증명해야 할 것은 __________ 이다.

3^(k + 1)
= 3 · 3^k
≥ __________
= __________

이 값이 2(k + 1) + 1 이상임을 보이면 된다.

Conclusion:
따라서 수학적 귀납법에 의해 __________.
```

---

## 실습 2. Loop Invariant 작성

다음 코드에 대해 loop invariant를 작성하라.

```cpp
int product_array(int a[], int n) {
    int p = 1;

    for (int i = 0; i < n; i++) {
        p *= a[i];
    }

    return p;
}
```

증명하고 싶은 것:

```text
product_array는 a[0] · a[1] · ... · a[n - 1]을 반환한다.
```

질문:

```text
반복문에서 i에 도달했을 때 p는 무엇인가?
```

---

## 실습 3. 잘못된 induction 찾기

다음 증명이 왜 틀렸는지 설명하라.

```text
Claim:
모든 n ≥ 1에 대해 n = 1이다.

Base Case:
n = 1이면 참이다.

Inductive Step:
P(k)가 참이라고 가정한다.
즉 k = 1이다.
그러면 k + 1 = 2이다.
따라서 P(k + 1)도 참이다.
```

힌트:

```text
P(k + 1)은 무엇이어야 하는가?
```

---

# 14. 핵심 정리

Mathematical induction은 자연수 전체에 대한 명제를 증명하는 기본 도구이다.

핵심 구조는 다음이다.

```text
1. Base Case:
   P(1)을 증명한다.

2. Inductive Hypothesis:
   P(k)가 참이라고 가정한다.

3. Inductive Step:
   P(k + 1)을 증명한다.

4. Conclusion:
   모든 n ≥ 1에 대해 P(n)이 참이다.
```

가장 중요한 감각은 이것이다.

```text
P(k)를 이미 참이라고 믿고,
그 힘을 사용해서 P(k + 1)을 증명한다.
```

CS에서는 induction이 다음과 연결된다.

```text
반복문 correctness proof
재귀 함수 correctness proof
자료구조 invariant
dynamic programming correctness
입력 크기 n에 대한 알고리즘 증명
```

# Lecture 3. Induction Example: Sum Formula

## 1. 핵심 질문

> **왜 `1 + 2 + ... + n = n(n+1)/2`라는 공식이 모든 자연수 `n`에 대해 참인가?**

예시를 보면 맞아 보인다.

```text
n = 1: 1 = 1(2)/2 = 1

n = 2: 1 + 2 = 3 = 2(3)/2

n = 3: 1 + 2 + 3 = 6 = 3(4)/2

n = 4: 1 + 2 + 3 + 4 = 10 = 4(5)/2
```

하지만 이것은 **패턴 관찰**이지 **proof**가 아니다.

이번 강의의 목표는 이 공식을 induction으로 완전히 증명하는 것이다.

---

# 2. 형식적 정의

우리가 증명하려는 명제는 다음이다.

```text
For all integers n ≥ 1,

1 + 2 + ... + n = n(n + 1) / 2
```

이를 `P(n)`이라고 하자.

```text
P(n): 1 + 2 + ... + n = n(n + 1) / 2
```

귀납법의 구조는 다음이다.

```text
1. Base Case:
   P(1)이 참임을 보인다.

2. Inductive Hypothesis:
   어떤 임의의 k ≥ 1에 대해 P(k)가 참이라고 가정한다.

3. Inductive Step:
   P(k)가 참이면 P(k + 1)도 참임을 보인다.

4. Conclusion:
   따라서 모든 n ≥ 1에 대해 P(n)이 참이다.
```

---

# 3. 직관적 설명

공식은 다음 합을 압축해서 표현한다.

```text
1 + 2 + 3 + ... + n
```

예를 들어 `n = 100`이면:

```text
1 + 2 + 3 + ... + 100
```

을 직접 더하지 않고:

```text
100(101) / 2 = 5050
```

으로 계산할 수 있다.

직관적으로는 양 끝을 묶으면 된다.

```text
1   + 2   + 3   + ... + 98  + 99  + 100
100 + 99  + 98  + ... + 3   + 2   + 1
```

각 쌍의 합은 `101`이다.

```text
1 + 100 = 101
2 + 99  = 101
3 + 98  = 101
...
```

총 `100`개의 항이 있고, 두 줄을 더했으므로:

```text
2S = 100 · 101
S = 100 · 101 / 2
```

일반화하면:

```text
S = n(n + 1) / 2
```

하지만 이 설명은 이번 강의의 메인 proof가 아니다.

이번에는 **induction proof**를 배운다.

---

# 4. 왜 필요한지

이 공식은 단순한 산수 공식이 아니다.

CS에서는 다음과 직접 연결된다.

## 4.1 반복문의 총 실행 횟수

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        // O(1) work
    }
}
```

내부 반복문의 실행 횟수는:

```text
1 + 2 + 3 + ... + n
```

따라서 전체 시간 복잡도는:

```text
n(n + 1) / 2
```

즉:

```text
O(n^2)
```

---

## 4.2 알고리즘 분석

예를 들어 insertion sort의 최악의 경우 비교 횟수도 비슷한 형태가 나온다.

```text
1 + 2 + 3 + ... + (n - 1)
```

이 합은:

```text
(n - 1)n / 2
```

따라서 insertion sort의 worst-case time complexity는:

```text
O(n^2)
```

---

## 4.3 Loop invariant와 induction

반복문이 매 단계에서 누적합을 계산할 때, 그 correctness proof는 induction과 거의 같다.

```cpp
int sum_to_n(int n) {
    int s = 0;

    for (int i = 1; i <= n; i++) {
        s += i;
    }

    return s;
}
```

증명하고 싶은 것:

```text
함수는 1 + 2 + ... + n을 반환한다.
```

반복문 invariant:

```text
i번째 반복이 끝난 후,
s = 1 + 2 + ... + i
```

이 구조는 정확히 induction이다.

---

# 5. 증명 구조

우리는 다음 명제를 증명한다.

```text
Claim:
For all integers n ≥ 1,

1 + 2 + ... + n = n(n + 1) / 2
```

귀납 증명의 skeleton은 다음과 같다.

```text
Proof by induction on n.

Base Case:
n = 1일 때 P(1)이 참임을 보인다.

Inductive Hypothesis:
어떤 임의의 k ≥ 1에 대해 P(k)가 참이라고 가정한다.
즉,
1 + 2 + ... + k = k(k + 1) / 2

Inductive Step:
P(k + 1)을 증명한다.
즉,
1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2

Conclusion:
따라서 모든 n ≥ 1에 대해 공식이 참이다.
```

여기서 가장 중요한 것은 다음이다.

```text
P(k)를 정확히 쓰고,
P(k + 1)을 정확히 쓴다.
```

많은 초보자가 여기서 틀린다.

---

# 6. 단계별 예시

## 6.1 Claim

```text
Claim:
For all n ≥ 1,

1 + 2 + ... + n = n(n + 1) / 2
```

---

## 6.2 Base Case

`n = 1`일 때를 확인한다.

왼쪽:

```text
1
```

오른쪽:

```text
1(1 + 1) / 2 = 1 · 2 / 2 = 1
```

따라서:

```text
1 = 1(2) / 2
```

즉 `P(1)`은 참이다.

---

## 6.3 Inductive Hypothesis

어떤 임의의 정수 `k ≥ 1`에 대해 `P(k)`가 참이라고 가정한다.

즉:

```text
1 + 2 + ... + k = k(k + 1) / 2
```

이것이 inductive hypothesis이다.

중요하다.

여기서 우리는 모든 경우를 이미 참이라고 가정하는 것이 아니다.

오직 다음 implication을 증명하려고 한다.

```text
P(k) ⇒ P(k + 1)
```

즉:

```text
만약 k까지의 공식이 맞다면,
k + 1까지의 공식도 맞다.
```

---

## 6.4 Inductive Step

이제 증명해야 할 것은 `P(k + 1)`이다.

먼저 `P(k + 1)`을 정확히 써보자.

`P(n)`은:

```text
1 + 2 + ... + n = n(n + 1) / 2
```

이다.

그러므로 `n = k + 1`을 대입하면:

```text
1 + 2 + ... + k + (k + 1)
= (k + 1)((k + 1) + 1) / 2
```

정리하면:

```text
1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2
```

이것이 우리가 증명해야 하는 목표다.

---

## 6.5 왼쪽에서 시작하기

왼쪽을 보자.

```text
1 + 2 + ... + k + (k + 1)
```

앞부분 `1 + 2 + ... + k`는 inductive hypothesis에서 이미 알고 있다.

inductive hypothesis:

```text
1 + 2 + ... + k = k(k + 1) / 2
```

따라서:

```text
1 + 2 + ... + k + (k + 1)
= k(k + 1) / 2 + (k + 1)
```

이제 대수적으로 정리한다.

```text
k(k + 1) / 2 + (k + 1)
= k(k + 1) / 2 + 2(k + 1) / 2
= [k(k + 1) + 2(k + 1)] / 2
= (k + 1)(k + 2) / 2
```

따라서:

```text
1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2
```

즉 `P(k + 1)`이 참이다.

---

## 6.6 Conclusion

Base case `P(1)`이 참이고, 모든 `k ≥ 1`에 대해:

```text
P(k) ⇒ P(k + 1)
```

가 참이다.

따라서 수학적 귀납법에 의해:

```text
모든 n ≥ 1에 대해

1 + 2 + ... + n = n(n + 1) / 2
```

이다.

---

# 7. 전체 증명 완성본

아래가 제출 가능한 형태의 완전한 proof이다.

```text
Claim:
For all integers n ≥ 1,

1 + 2 + ... + n = n(n + 1) / 2.

Proof:
We prove the claim by induction on n.

Base Case:
When n = 1, the left-hand side is 1.
The right-hand side is 1(1 + 1) / 2 = 1.
Therefore P(1) is true.

Inductive Hypothesis:
Assume that for some arbitrary integer k ≥ 1,

1 + 2 + ... + k = k(k + 1) / 2.

Inductive Step:
We need to prove that

1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2.

Starting from the left-hand side,

1 + 2 + ... + k + (k + 1)
= k(k + 1) / 2 + (k + 1)
= k(k + 1) / 2 + 2(k + 1) / 2
= [k(k + 1) + 2(k + 1)] / 2
= (k + 1)(k + 2) / 2.

Thus P(k + 1) is true.

Therefore, by mathematical induction,

1 + 2 + ... + n = n(n + 1) / 2

for all integers n ≥ 1.
```

---

# 8. Formal Proof와 직관 구분

## 직관

```text
1부터 n까지 더하면 대략 n개의 항이 있고,
평균값이 대략 n/2 정도이므로 전체 합은 대략 n²/2이다.
```

또는:

```text
처음과 끝을 묶으면 n + 1이 반복되므로 n(n + 1)/2가 된다.
```

이것은 좋은 intuition이다.

---

## Formal Proof

```text
Base case를 보인다.
P(k)를 가정한다.
P(k + 1)을 도출한다.
따라서 모든 n에 대해 참이다.
```

Formal proof는 모든 자연수 `n`에 대해 논리적으로 보장한다.

---

# 9. Algebraic Manipulation 자세히 보기

Inductive step에서 가장 중요한 계산은 이것이다.

```text
k(k + 1) / 2 + (k + 1)
```

이것을 목표 형태로 바꿔야 한다.

목표:

```text
(k + 1)(k + 2) / 2
```

중간 과정:

```text
k(k + 1) / 2 + (k + 1)
```

`(k + 1)`을 분모 2로 맞춘다.

```text
= k(k + 1) / 2 + 2(k + 1) / 2
```

분자를 합친다.

```text
= [k(k + 1) + 2(k + 1)] / 2
```

공통인수 `(k + 1)`을 묶는다.

```text
= [(k + 1)(k + 2)] / 2
```

따라서:

```text
= (k + 1)(k + 2) / 2
```

여기서 핵심은 **목표식을 알고 변형하는 것**이다.

---

# 10. 왜 몇 개 예시 확인은 proof가 아닌가

다음은 proof가 아니다.

```text
1 = 1(2)/2
1 + 2 = 2(3)/2
1 + 2 + 3 = 3(4)/2
1 + 2 + 3 + 4 = 4(5)/2

따라서 모든 n에 대해 참이다.
```

문제는 명제가 무한한 범위를 갖는다는 것이다.

```text
n = 1, 2, 3, 4
```

를 확인해도 아직 다음이 남아 있다.

```text
n = 5, 6, 7, ...
```

심지어 처음 몇 개는 맞지만 나중에 틀리는 명제도 만들 수 있다.

예를 들어 다음 명제를 보자.

```text
P(n): n^2 - n + 41은 소수이다.
```

몇 개를 확인하면 참처럼 보인다.

```text
n = 1: 41
n = 2: 43
n = 3: 47
n = 4: 53
```

하지만 `n = 41`이면:

```text
41^2 - 41 + 41 = 41^2
```

이므로 소수가 아니다.

즉 예시 확인은 proof가 아니다.

---

# 11. Loop Invariant와 Induction의 관계

Induction은 반복문 correctness proof의 기반이다.

다시 이 코드를 보자.

```cpp
int sum_to_n(int n) {
    int s = 0;

    for (int i = 1; i <= n; i++) {
        s += i;
    }

    return s;
}
```

증명하고 싶은 것:

```text
sum_to_n(n)은 1 + 2 + ... + n을 반환한다.
```

---

## Loop Invariant

반복문에서 `i`번째 반복이 끝난 후:

```text
s = 1 + 2 + ... + i
```

또는 반복문 시작 시점 기준으로 쓰면:

```text
반복문이 i에 도달하기 직전,
s = 1 + 2 + ... + (i - 1)
```

두 표현 모두 가능하다.

다만 기준 시점을 명확히 해야 한다.

---

## Induction과 비교

| Induction            | Loop Invariant                   |
| -------------------- | -------------------------------- |
| Base Case            | 반복문 시작 전 invariant 확인            |
| Inductive Hypothesis | 어떤 반복 시점에서 invariant가 참이라고 가정    |
| Inductive Step       | loop body 실행 후에도 invariant 유지 증명 |
| Conclusion           | 반복문 종료 시 원하는 결과 도출               |

---

## Correctness Proof

### Initialization

반복문 시작 전:

```cpp
int s = 0;
int i = 1;
```

이때 `s = 0`이다.

그리고 `1`부터 `i - 1 = 0`까지의 합은 empty sum이므로 0이다.

따라서 invariant는 참이다.

```text
s = 1 + 2 + ... + (i - 1)
```

---

### Maintenance

반복문 시작 시점에:

```text
s = 1 + 2 + ... + (i - 1)
```

이라고 가정한다.

loop body:

```cpp
s += i;
```

실행 후:

```text
s = 1 + 2 + ... + (i - 1) + i
```

즉:

```text
s = 1 + 2 + ... + i
```

그 다음 `i`가 1 증가하므로, 다음 반복 시작 시점에서는 다시:

```text
s = 1 + 2 + ... + (i - 1)
```

형태가 유지된다.

---

### Termination

반복문은 `i = n + 1`일 때 종료한다.

Invariant에 의해:

```text
s = 1 + 2 + ... + n
```

따라서 함수는 원하는 합을 반환한다.

---

# 12. CS 응용

## 12.1 시간 복잡도 분석

다음 코드를 보자.

```cpp
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        count++;
    }
}
```

`count++`가 실행되는 횟수는:

```text
1 + 2 + ... + n
```

방금 증명한 공식에 의해:

```text
n(n + 1) / 2
```

따라서:

```text
n(n + 1) / 2 = (n^2 + n) / 2
```

Big-O로는:

```text
O(n^2)
```

---

## 12.2 배열 prefix sum

다음 코드도 같은 induction 구조를 가진다.

```cpp
void prefix_sum(int a[], int prefix[], int n) {
    prefix[0] = a[0];

    for (int i = 1; i < n; i++) {
        prefix[i] = prefix[i - 1] + a[i];
    }
}
```

증명하고 싶은 것:

```text
prefix[i] = a[0] + a[1] + ... + a[i]
```

Induction 구조:

```text
Base:
prefix[0] = a[0]

Inductive Hypothesis:
prefix[k - 1] = a[0] + ... + a[k - 1]

Step:
prefix[k] = prefix[k - 1] + a[k]
          = a[0] + ... + a[k - 1] + a[k]
          = a[0] + ... + a[k]
```

이것이 배열 알고리즘에서 induction이 쓰이는 전형적인 방식이다.

---

## 12.3 Insertion Sort와 합 공식

Insertion sort의 worst-case에서는 새 원소를 앞쪽으로 계속 밀어야 한다.

대략 비교 횟수는:

```text
1 + 2 + ... + (n - 1)
```

공식에 의해:

```text
(n - 1)n / 2
```

따라서 worst-case time complexity는:

```text
O(n^2)
```

여기서 sum formula는 단순한 수학 공식이 아니라, 알고리즘 비용 분석의 도구이다.

---

# 13. 흔한 오해

## 오해 1. “P(k)를 가정하는 것은 cheating 아닌가?”

아니다.

우리는 `P(k)`가 실제로 참이라고 무작정 믿는 것이 아니다.

우리가 증명하는 것은 다음 implication이다.

```text
P(k) ⇒ P(k + 1)
```

즉:

```text
만약 k번째까지 공식이 맞다면,
그 다음 k + 1번째도 맞다.
```

그리고 base case가 있으므로 chain이 시작된다.

```text
P(1)
P(1) ⇒ P(2)
P(2) ⇒ P(3)
P(3) ⇒ P(4)
...
```

따라서 모든 자연수에 대해 참이다.

---

## 오해 2. “Base case는 대충 확인하면 된다”

아니다.

Base case는 귀납 사슬의 시작점이다.

만약 명제가 `n ≥ 1`에 대한 것이라면 보통 `P(1)`을 확인해야 한다.

명제가 `n ≥ 0`에 대한 것이라면 `P(0)`을 확인해야 한다.

예:

```text
모든 n ≥ 0에 대해 1 + 2 + ... + n = n(n + 1)/2
```

이 경우 `n = 0`에서 왼쪽은 empty sum으로 0이다.

```text
0 = 0(1)/2
```

그래서 참이다.

---

## 오해 3. “Inductive step에서 목표식을 안 써도 된다”

초보자는 반드시 써야 한다.

예를 들어 `P(k + 1)`은 다음이다.

```text
1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2
```

이것을 안 쓰면 어디로 가야 하는지 모르게 된다.

증명은 방향감각이 중요하다.

---

## 오해 4. “등식 변형만 하면 proof다”

Induction proof에서 등식 변형은 일부일 뿐이다.

반드시 다음 구조가 있어야 한다.

```text
Base Case
Inductive Hypothesis
Inductive Step
Conclusion
```

등식 변형만 있고 inductive hypothesis를 사용하지 않으면 귀납 증명이 아니다.

---

# 14. 반례 또는 실패 사례

## 실패 사례 1. Inductive hypothesis 없이 목표만 씀

잘못된 증명:

```text
P(k + 1)을 보이면 된다.

1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2) / 2

따라서 참이다.
```

문제:

```text
가장 중요한 등식을 그냥 주장했다.
```

왜 왼쪽이 오른쪽과 같은지 설명하지 않았다.

올바른 방식:

```text
1 + 2 + ... + k + (k + 1)
= k(k + 1)/2 + (k + 1)
= (k + 1)(k + 2)/2
```

첫 번째 등호에서 inductive hypothesis를 사용해야 한다.

---

## 실패 사례 2. `P(k + 1)`을 잘못 씀

원래 명제:

```text
P(n): 1 + 2 + ... + n = n(n + 1)/2
```

잘못된 `P(k + 1)`:

```text
1 + 2 + ... + k + 1 = k(k + 1)/2
```

문제는 두 가지다.

첫째, 왼쪽에서 마지막 항이 애매하다.

```text
... + k + 1
```

은 `k + 1`이라는 한 항인지, `k`와 `1`을 따로 더한 것인지 헷갈린다.

둘째, 오른쪽에 `k + 1`을 대입하지 않았다.

올바른 `P(k + 1)`:

```text
1 + 2 + ... + k + (k + 1)
= (k + 1)(k + 2)/2
```

---

## 실패 사례 3. 예시 확인을 proof라고 착각

잘못된 증명:

```text
n = 1, 2, 3, 4에서 모두 맞다.
따라서 모든 n에 대해 맞다.
```

문제:

```text
무한히 많은 n을 다루지 못했다.
```

이것은 테스트 케이스와 같다.

테스트는 반례를 찾는 데 유용하지만, 전체 correctness proof는 아니다.

---

# 15. 확인 문제

## 문제 1

다음 induction proof에서 inductive hypothesis가 사용된 줄을 찾아라.

```text
1 + 2 + ... + k + (k + 1)
= k(k + 1)/2 + (k + 1)
= [k(k + 1) + 2(k + 1)]/2
= (k + 1)(k + 2)/2
```

---

## 문제 2

다음 명제를 induction으로 증명하라.

```text
모든 n ≥ 1에 대해

2 + 4 + 6 + ... + 2n = n(n + 1)
```

힌트:

```text
P(k): 2 + 4 + ... + 2k = k(k + 1)

P(k + 1):
2 + 4 + ... + 2k + 2(k + 1)
= (k + 1)(k + 2)
```

---

## 문제 3

다음 코드는 몇 번 `count++`를 실행하는가?

```cpp
int count = 0;

for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) {
        count++;
    }
}
```

정확한 식과 Big-O를 모두 구하라.

---

## 문제 4

다음 loop invariant를 완성하라.

```cpp
int sum_to_n(int n) {
    int s = 0;

    for (int i = 1; i <= n; i++) {
        s += i;
    }

    return s;
}
```

반복문 시작 시점에서의 invariant:

```text
At the start of iteration i, s = __________.
```

---

# 16. 실습 과제

## 실습 1. 완전한 induction proof 작성

다음 명제를 증명하라.

```text
모든 n ≥ 1에 대해

2 + 4 + 6 + ... + 2n = n(n + 1)
```

제출 형식:

```text
Claim:

Proof by induction on n.

Base Case:

Inductive Hypothesis:

Inductive Step:

Conclusion:
```

---

## 실습 2. 코드와 수식 연결

다음 코드의 실행 횟수를 induction으로 설명하라.

```cpp
int count = 0;

for (int i = 1; i <= n; i++) {
    count += i;
}
```

증명하고 싶은 것:

```text
반복문이 끝난 후 count = n(n + 1)/2
```

Loop invariant:

```text
i번째 반복이 끝난 후,
count = 1 + 2 + ... + i
```

이 invariant를 induction 구조로 증명해보라.

---

## 실습 3. 잘못된 proof 고치기

다음 proof를 고쳐라.

```text
Claim:
1 + 2 + ... + n = n(n + 1)/2.

Proof:
n = 1이면 참이다.
P(k)가 참이라고 하자.
그러면 P(k + 1)도 참이다.
따라서 모든 n에 대해 참이다.
```

문제점:

```text
P(k + 1)이 왜 참인지 증명하지 않았다.
```

고칠 때는 반드시 다음 줄이 들어가야 한다.

```text
1 + 2 + ... + k + (k + 1)
= k(k + 1)/2 + (k + 1)
```

---

# 17. 핵심 정리

이번 강의의 핵심은 다음이다.

```text
1 + 2 + ... + n = n(n + 1)/2
```

이 공식은 induction으로 다음처럼 증명된다.

```text
Base:
n = 1에서 참이다.

Hypothesis:
1 + 2 + ... + k = k(k + 1)/2 라고 가정한다.

Step:
1 + 2 + ... + k + (k + 1)
= k(k + 1)/2 + (k + 1)
= (k + 1)(k + 2)/2

Conclusion:
모든 n ≥ 1에 대해 참이다.
```

CS에서 이 공식은 특히 중요하다.

```text
중첩 반복문 실행 횟수
insertion sort worst-case 분석
prefix sum correctness
loop invariant proof
O(n²) 분석
```

가장 중요한 습관은 이것이다.

```text
P(k)를 정확히 쓰고,
P(k + 1)을 정확히 쓴 다음,
P(k)를 이용해서 P(k + 1)을 만든다.
```
