# Lecture 2. Array — 연속 메모리와 Random Access

## 1. 핵심 질문

**왜 배열은 인덱스로 접근할 때 O(1)인가?**

그리고 더 중요한 질문:

> “왜 배열은 이론적으로도 빠르고, 실제 컴퓨터에서도 빠른가?”

배열의 핵심은 두 가지다.

```text
1. 같은 타입의 데이터를
2. 메모리에 연속적으로 배치한다
```

이 때문에 배열은 다음 장점을 가진다.

| 장점                 | 이유                      |
| ------------------ | ----------------------- |
| 인덱스 접근이 빠름         | 주소 계산이 가능함              |
| 순회가 빠름             | 메모리가 연속되어 있음            |
| cache locality가 좋음 | CPU가 다음 데이터를 미리 가져오기 쉬움 |

올린 자료에서도 array는 **contiguous memory**, **O(1) index access**, **excellent cache performance**를 가진다고 정리되어 있습니다. 

---

## 2. 형식적 정의

### Array의 정의

**Formal definition**

배열은 다음 조건을 만족하는 선형 자료구조다.

> 동일한 타입의 원소들이 메모리상 연속된 위치에 저장되고, 각 원소는 정수 인덱스를 통해 접근된다.

수학적으로는 이렇게 볼 수 있다.

```text
Array A of length n

A[0], A[1], A[2], ..., A[n-1]
```

각 원소의 주소는 다음 식으로 계산된다.

```text
address(A[i]) = base_address + i × sizeof(element)
```

예를 들어:

```c
int arr[5] = {10, 20, 30, 40, 50};
```

`int`가 4바이트이고 `arr[0]`의 주소가 1000이라면:

| 원소     |   주소 |
| ------ | ---: |
| arr[0] | 1000 |
| arr[1] | 1004 |
| arr[2] | 1008 |
| arr[3] | 1012 |
| arr[4] | 1016 |

그래서 `arr[3]`은 이렇게 계산된다.

```text
address(arr[3]) = 1000 + 3 × 4 = 1012
```

반복문으로 앞에서부터 찾는 게 아니다.

주소를 바로 계산한다.

그래서:

```text
arr[i] access = O(1)
```

---

## 3. 직관적 설명

배열은 **호텔 복도에 번호가 붙은 방들이 일렬로 있는 구조**다.

```text
Room 0 | Room 1 | Room 2 | Room 3 | Room 4
  10   |   20   |   30   |   40   |   50
```

3번 방에 가고 싶으면?

```text
시작 위치 + 방 크기 × 3
```

바로 간다.

반대로 linked list는 이런 느낌이다.

```text
Room A says: next room is 8th floor
Room B says: next room is basement
Room C says: next room is another building
```

다음 위치를 계속 따라가야 한다.

배열은 “번호로 바로 찾기”가 가능하고, linked list는 “길을 따라가기”가 필요하다.

---

## 4. 왜 필요한지

배열은 컴퓨터 과학에서 가장 기본적인 자료구조다.

이유는 단순하다.

대부분의 고급 자료구조가 내부적으로 배열을 사용한다.

| 자료구조 / 시스템            | 내부에서 배열이 쓰이는 방식           |
| --------------------- | ------------------------- |
| `std::vector`         | 동적 배열                     |
| string buffer         | 문자 배열                     |
| hash table            | bucket array              |
| heap / priority queue | 배열 기반 binary heap         |
| matrix                | 2D array                  |
| CPU cache line        | 연속 메모리 접근 최적화             |
| ring buffer           | 고정 크기 배열 + circular index |

즉 배열을 모르면 `vector`, `string`, `hash table`, `heap`, `matrix`, `ring buffer`를 제대로 이해하기 어렵다.

---

## 5. 내부 원리

배열의 내부 원리는 크게 세 가지다.

```text
1. Contiguous memory
2. Index arithmetic
3. Cache locality
```

---

### 5.1 Contiguous Memory

배열은 메모리에 붙어서 저장된다.

```text
int arr[5] = {10, 20, 30, 40, 50};

Memory:
[10][20][30][40][50]
```

각 원소가 서로 바로 옆에 있다.

이 구조 덕분에 CPU는 다음 원소가 어디 있는지 예측하기 쉽다.

---

### 5.2 Index Arithmetic

배열 접근은 다음 계산으로 이루어진다.

```text
base + index × element_size
```

C에서:

```c
arr[i]
```

는 내부적으로 거의 다음과 같다.

```c
*(arr + i)
```

예:

```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};

    printf("%d\n", arr[3]);
    printf("%d\n", *(arr + 3));

    return 0;
}
```

둘 다 `40`을 출력한다.

즉 배열 이름 `arr`은 많은 상황에서 첫 번째 원소의 주소처럼 동작한다.

---

### 5.3 Cache Locality

여기가 중요하다.

Big-O만 보면 array traversal과 linked list traversal은 둘 다 O(N)이다.

하지만 실제 성능은 다르다.

#### Array traversal

```text
[1][2][3][4][5][6][7][8]
```

CPU가 `[1]`을 가져올 때, 보통 주변 데이터도 cache line 단위로 같이 가져온다.

그러면 다음 원소 `[2]`, `[3]`, `[4]` 접근이 빠르다.

#### Linked list traversal

```text
[1] -> memory somewhere -> [2] -> memory elsewhere -> [3]
```

다음 노드가 메모리 어디에 있는지 예측하기 어렵다.

그래서 cache miss가 자주 발생한다.

자료에서도 array의 cache locality는 **spatial locality**, **CPU prefetch**, **cache line utilization** 때문에 좋다고 설명합니다. 

---

## 6. 단계별 예시

### 예시 1. 배열 접근

```c
int arr[5] = {3, 7, 11, 15, 19};
```

`arr[2]`를 읽는 과정:

```text
1. arr의 시작 주소를 안다.
2. int 크기가 4바이트임을 안다.
3. index 2를 받는다.
4. 주소 = base + 2 × 4
5. 해당 주소에서 값을 읽는다.
```

결과:

```text
arr[2] = 11
```

복잡도:

```text
O(1)
```

---

### 예시 2. 배열 전체 순회

```c
#include <stdio.h>

void print_all(int arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d\n", arr[i]);
    }
}
```

이 함수는 원소 n개를 모두 방문한다.

```text
Time Complexity: O(N)
Space Complexity: O(1)
```

입력 배열 자체는 이미 존재하는 데이터이므로, 추가 공간은 거의 쓰지 않는다.

---

### 예시 3. 중간 삽입

배열이 다음과 같다고 하자.

```text
index:  0   1   2   3   4
value: [10][20][30][40][50]
```

index 2에 `99`를 넣고 싶다.

하지만 이미 index 2에는 `30`이 있다.

따라서 뒤 원소들을 오른쪽으로 밀어야 한다.

```text
Before:
[10][20][30][40][50][_]

Step 1:
[10][20][30][40][_][50]

Step 2:
[10][20][30][_][40][50]

Step 3:
[10][20][_][30][40][50]

Insert:
[10][20][99][30][40][50]
```

삽입 위치 뒤의 원소를 전부 움직여야 한다.

최악의 경우:

```text
O(N)
```

---

### 예시 4. 중간 삭제

자료의 예시와 같은 구조다. 

```text
[1, 2, 3, 4, 5]
```

`3`을 삭제한다.

```text
[1, 2, _, 4, 5]
```

뒤 원소를 앞으로 당긴다.

```text
[1, 2, 4, 5]
```

삭제 자체는 간단해 보이지만, 실제로는 shifting이 필요하다.

```text
Time Complexity: O(N)
```

---

## 7. 실제 응용

### 7.1 `std::vector`

C++에서 가장 자주 쓰는 배열 기반 컨테이너다.

```cpp
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v;

    v.push_back(10);
    v.push_back(20);
    v.push_back(30);

    std::cout << v[1] << "\n"; // 20
}
```

`std::vector`의 특징:

| 연산          |                  평균 복잡도 |
| ----------- | ----------------------: |
| `v[i]`      |                    O(1) |
| `push_back` |          amortized O(1) |
| 중간 삽입       |                    O(N) |
| 중간 삭제       |                    O(N) |
| 순회          | O(N), 매우 cache-friendly |

---

### 7.2 문자열

C 문자열도 배열이다.

```c
char s[] = "hello";
```

메모리에는 대략 이렇게 저장된다.

```text
['h']['e']['l']['l']['o']['\0']
```

마지막의 `'\0'`은 null terminator다.

C에서 문자열 길이를 구하는 `strlen`은 O(N)이다.

왜냐하면 `'\0'`을 만날 때까지 앞에서부터 순회해야 하기 때문이다.

```c
size_t my_strlen(const char *s) {
    size_t len = 0;

    while (s[len] != '\0') {
        len++;
    }

    return len;
}
```

---

### 7.3 2D Array와 Matrix

2차원 배열도 실제 메모리에서는 보통 일렬로 저장된다.

```c
int mat[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};
```

논리적으로는:

```text
[1 2 3]
[4 5 6]
```

하지만 메모리에는 대체로:

```text
[1][2][3][4][5][6]
```

처럼 저장된다.

C의 row-major order에서는 `mat[i][j]` 주소가 다음처럼 계산된다.

```text
base + (i × column_count + j) × sizeof(int)
```

예:

```text
mat[1][2]
= base + (1 × 3 + 2) × 4
= base + 5 × 4
```

---

### 7.4 실무적 성능 차이

다음 두 코드는 Big-O만 보면 둘 다 O(N²)이다.

```c
for (int i = 0; i < rows; i++) {
    for (int j = 0; j < cols; j++) {
        sum += mat[i][j];
    }
}
```

```c
for (int j = 0; j < cols; j++) {
    for (int i = 0; i < rows; i++) {
        sum += mat[i][j];
    }
}
```

하지만 C의 row-major layout에서는 첫 번째 코드가 더 cache-friendly하다.

왜냐하면 메모리 저장 순서대로 접근하기 때문이다.

```text
좋은 접근:
mat[0][0] -> mat[0][1] -> mat[0][2]

덜 좋은 접근:
mat[0][0] -> mat[1][0] -> mat[2][0]
```

---

## 8. 흔한 오해

### 오해 1. “Array는 항상 크기를 바꿀 수 없다”

정확히는 **고정 크기 배열**은 크기를 바꿀 수 없다.

```c
int arr[10];
```

이 배열은 10칸이다.

하지만 dynamic array는 내부적으로 새 배열을 만들어 크기를 늘린다.

```cpp
std::vector<int> v;
```

즉 vector가 기존 배열의 크기를 마법처럼 늘리는 게 아니다.

실제로는:

```text
1. 더 큰 메모리 확보
2. 기존 원소 복사/이동
3. 기존 메모리 해제
```

---

### 오해 2. “arr[i]는 무조건 안전하다”

아니다.

C/C++에서는 bounds checking을 자동으로 해주지 않는다.

```c
int arr[3] = {1, 2, 3};

printf("%d\n", arr[10]); // undefined behavior
```

`arr[10]`은 존재하지 않는 메모리를 읽는다.

이건 단순한 에러가 아니라 **undefined behavior**다.

프로그램이 죽을 수도 있고, 이상한 값을 출력할 수도 있고, 겉보기엔 정상처럼 보일 수도 있다.

---

### 오해 3. “O(1)이면 비용이 없다”

`arr[i]`는 O(1)이지만, 실제로는 다음 비용이 있다.

```text
- 주소 계산
- 메모리 접근
- cache hit 또는 cache miss
```

O(1)은 “입력 크기 N에 따라 증가하지 않는다”는 뜻이지, “0초”라는 뜻이 아니다.

---

## 9. 반례 또는 실패 사례

### 실패 사례 1. 배열 범위 초과

```c
#include <stdio.h>

int main(void) {
    int arr[3] = {10, 20, 30};

    for (int i = 0; i <= 3; i++) {
        printf("%d\n", arr[i]);
    }

    return 0;
}
```

문제는 여기다.

```c
i <= 3
```

배열 index는 `0, 1, 2`만 유효하다.

`arr[3]`은 범위 밖이다.

수정:

```c
for (int i = 0; i < 3; i++) {
    printf("%d\n", arr[i]);
}
```

---

### 실패 사례 2. 중간 삭제가 많은데 array만 사용하는 경우

다음 상황을 보자.

```text
- 원소 100,000개
- 매번 중간에서 삭제
- 삭제가 수천 번 반복됨
```

배열에서 중간 삭제는 O(N)이다.

수천 번 반복하면 전체 비용이 커진다.

```text
O(N × number_of_deletions)
```

이런 경우에는 다른 구조를 고려해야 한다.

예:

```text
- linked list
- deque
- balanced tree
- gap buffer
- lazy deletion
```

단, linked list가 무조건 답은 아니다.

삭제할 위치를 찾는 비용이 여전히 O(N)이기 때문이다.

---

## 10. 확인 문제

### 문제 1

다음 코드의 시간 복잡도는?

```c
int x = arr[n - 1];
```

정답:

```text
O(1)
```

이유:

```text
마지막 원소라도 index로 주소를 바로 계산한다.
```

---

### 문제 2

다음 코드의 시간 복잡도는?

```c
for (int i = 0; i < n; i++) {
    sum += arr[i];
}
```

정답:

```text
O(N)
```

이유:

```text
n개의 원소를 모두 한 번씩 읽는다.
```

---

### 문제 3

배열에서 index 0의 원소를 삭제하면 왜 O(N)인가?

정답:

```text
나머지 모든 원소를 한 칸씩 왼쪽으로 이동해야 하기 때문이다.
```

예:

```text
[10][20][30][40]

delete 10

[_][20][30][40]
[20][30][40]
```

---

### 문제 4

C에서 `arr[i]`와 `*(arr + i)`는 어떤 관계인가?

정답:

```text
대부분의 배열 접근 상황에서 arr[i]는 *(arr + i)와 같은 의미다.
```

---

## 11. 실습 과제

### 과제 1. 배열 최댓값 찾기

다음 함수를 구현해라.

```c
int max_value(int arr[], int n);
```

조건:

```text
- n이 0 이하이면 어떻게 처리할지 정해야 한다.
- 배열 전체를 순회한다.
```

예시 구현:

```c
#include <limits.h>

int max_value(int arr[], int n) {
    if (n <= 0) {
        return INT_MIN;
    }

    int max = arr[0];

    for (int i = 1; i < n; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }

    return max;
}
```

복잡도:

```text
Time: O(N)
Space: O(1)
```

---

### 과제 2. 배열 중간 삽입 구현

다음 함수를 구현해라.

```c
int insert_at(int arr[], int *n, int capacity, int index, int value);
```

조건:

```text
- capacity가 꽉 차 있으면 실패
- index가 0보다 작거나 *n보다 크면 실패
- 성공하면 1 반환
- 실패하면 0 반환
```

예시 구현:

```c
int insert_at(int arr[], int *n, int capacity, int index, int value) {
    if (*n >= capacity) {
        return 0;
    }

    if (index < 0 || index > *n) {
        return 0;
    }

    for (int i = *n; i > index; i--) {
        arr[i] = arr[i - 1];
    }

    arr[index] = value;
    (*n)++;

    return 1;
}
```

핵심은 뒤에서부터 밀어야 한다는 점이다.

앞에서부터 밀면 값이 덮어씌워진다.

---

### 과제 3. 주소 계산 직접 확인하기

아래 코드를 실행해서 배열 주소가 어떻게 변하는지 확인해라.

```c
#include <stdio.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};

    for (int i = 0; i < 5; i++) {
        printf("&arr[%d] = %p, value = %d\n",
               i, (void *)&arr[i], arr[i]);
    }

    return 0;
}
```

예상 관찰:

```text
int가 4바이트인 환경에서는 주소가 보통 4씩 증가한다.
```

---

## 12. 핵심 정리

```text
Array의 본질은 “연속 메모리 + 인덱스 기반 주소 계산”이다.
```

| 항목     | 배열의 특징                         |
| ------ | ------------------------------ |
| 메모리 배치 | 연속적                            |
| 인덱스 접근 | O(1)                           |
| 순회     | O(N), cache-friendly           |
| 중간 삽입  | O(N)                           |
| 중간 삭제  | O(N)                           |
| 장점     | 단순함, 빠른 접근, 좋은 cache locality  |
| 단점     | 크기 변경 어려움, 삽입/삭제 비용, bounds 위험 |

배열을 배울 때 가장 중요한 문장은 이것이다.

> 배열은 단순한 자료구조가 아니라, 컴퓨터 메모리 모델과 가장 직접적으로 연결된 자료구조다.
