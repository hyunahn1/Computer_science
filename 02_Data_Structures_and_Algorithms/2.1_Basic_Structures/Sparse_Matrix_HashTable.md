# Lecture 7. Sparse Matrix & Hash Table Intro

## 희소성, Key-Value Mapping, Load Factor

## 1. 핵심 질문

이번 강의의 핵심 질문은 두 개다.

> **데이터 대부분이 비어 있거나 0이면, 그대로 저장해야 하는가?**

그리고:

> **값을 찾기 위해 매번 처음부터 탐색하지 않고, key로 바로 접근할 수는 없는가?**

이 두 질문이 각각 다음 자료구조로 이어진다.

| 문제             | 해결 아이디어         | 자료구조          |
| -------------- | --------------- | ------------- |
| 대부분의 값이 0이다    | 0이 아닌 값만 저장한다   | Sparse Matrix |
| key로 빠르게 찾고 싶다 | key를 index로 바꾼다 | Hash Table    |

자료에서도 Sparse Matrix는 대부분이 0인 행렬이며, 2D array로 저장하면 비효율적이므로 non-zero element만 저장하는 방식을 사용한다고 정리되어 있습니다. Hash Table은 key-value mapping과 hash function을 통해 key를 index로 바꾸는 구조로 설명되어 있습니다. 

---

# 2. 형식적 정의

## 2.1 Sparse Matrix

**Formal definition**

Sparse Matrix는 대부분의 원소가 `0`인 행렬이다.

일반적인 행렬:

```text
M ∈ R^(m×n)
```

즉 `m × n`개의 원소를 가진다.

일반 2D array 저장 방식은 모든 값을 저장한다.

```text
M[i][j]
```

하지만 sparse matrix에서는 대부분의 값이 `0`이므로, 보통 **0이 아닌 값만 저장**한다.

예:

```text
[
  [0, 0, 0, 5],
  [0, 0, 0, 0],
  [0, 7, 0, 0],
  [0, 0, 0, 9]
]
```

0이 아닌 값은 3개뿐이다.

```text
(0, 3) = 5
(2, 1) = 7
(3, 3) = 9
```

이렇게 저장할 수 있다.

---

## 2.2 Hash Table

**Formal definition**

Hash Table은 key-value pair를 저장하는 자료구조다.

```text
key -> value
```

핵심은 hash function이다.

```text
h(key) = index
```

즉 key를 배열 index로 변환한다.

예:

```text
h("hyunjun") = 3
```

그러면 내부 배열의 3번 bucket에 값을 저장한다.

```text
table[3] = value
```

Hash table의 기본 연산:

```text
insert(key, value)
search(key)
delete(key)
```

평균적으로:

```text
insert: O(1)
search: O(1)
delete: O(1)
```

하지만 collision이 심하면 최악의 경우 O(N)이 될 수 있다. 자료에서도 hash table lookup은 average O(1)이지만, collision이 많으면 worst case O(N)이라고 설명합니다. 

---

# 3. 직관적 설명

## 3.1 Sparse Matrix 직관

일반 matrix 저장은 큰 창고의 모든 칸을 기록하는 것이다.

```text
0 0 0 5
0 0 0 0
0 7 0 0
0 0 0 9
```

그런데 대부분이 0이면 이렇게 묻는 게 자연스럽다.

> “0은 굳이 저장하지 않아도 되지 않나?”

그래서 sparse matrix는 실제 값이 있는 위치만 기록한다.

```text
row col value
0   3   5
2   1   7
3   3   9
```

즉 sparse matrix는:

```text
빈 공간을 저장하지 않는 행렬 표현 방식
```

이다.

---

## 3.2 Hash Table 직관

배열에서 값을 찾으려면 보통 탐색이 필요하다.

```text
users = [Alice, Bob, Charlie, David]
```

`"Charlie"`를 찾으려면 앞에서부터 볼 수 있다.

```text
Alice? no
Bob? no
Charlie? yes
```

O(N).

Hash table은 다르게 한다.

```text
"Charlie" -> hash function -> index 2
```

바로 2번 위치로 간다.

```text
table[2]
```

그래서 평균 O(1)에 가깝다.

직관적으로 hash function은:

```text
이름표를 사물함 번호로 바꿔주는 함수
```

다.

---

# 4. 왜 필요한지

## 4.1 Sparse Matrix가 필요한 이유

일반 행렬을 그대로 저장하면 메모리가 낭비된다.

예를 들어 `10000 × 10000` 행렬이 있다고 하자.

전체 원소 수:

```text
10000 × 10000 = 100,000,000
```

`int` 하나가 4 bytes면:

```text
400,000,000 bytes ≈ 400 MB
```

그런데 non-zero 값이 10,000개뿐이라면?

일반 2D array는 400MB를 쓴다.

Sparse representation은 대략:

```text
(row, col, value) × 10000
```

정도만 저장하면 된다.

각각 int 3개라고 단순 계산하면:

```text
3 × 4 bytes × 10000 = 120,000 bytes ≈ 120 KB
```

400MB와 120KB는 차이가 매우 크다.

---

## 4.2 Hash Table이 필요한 이유

검색은 프로그램에서 매우 자주 나온다.

예:

```text
user_id로 User 찾기
문자열이 이미 등장했는지 확인
파일 경로로 metadata 찾기
변수 이름으로 symbol 찾기
cache key로 결과 찾기
HTTP header name으로 value 찾기
```

매번 list를 순회하면 O(N)이다.

Hash table을 쓰면 평균적으로 O(1)에 가까워진다.

예:

```cpp
std::unordered_map<std::string, int> score;

score["Alice"] = 90;
score["Bob"] = 85;

std::cout << score["Alice"] << "\n";
```

내부적으로는 `"Alice"`를 hash해서 bucket 위치를 찾는다.

---

# 5. 내부 원리

## 5.1 Sparse Matrix 저장 방식

대표적인 방식 세 가지가 있다.

자료에서도 Coordinate list, CSR, Hash table 방식을 제시합니다. 

| 방식         | 핵심 아이디어                        |
| ---------- | ------------------------------ |
| COO        | `(row, col, value)` triples 저장 |
| CSR        | row 기준으로 압축 저장                 |
| Hash table | `(row, col)`을 key로 저장          |

---

## 5.2 COO: Coordinate List

가장 단순한 sparse matrix 표현이다.

행렬:

```text
[
  [0, 0, 0, 5],
  [0, 0, 0, 0],
  [0, 7, 0, 0],
  [0, 0, 0, 9]
]
```

COO 표현:

```text
rows   = [0, 2, 3]
cols   = [3, 1, 3]
values = [5, 7, 9]
```

또는 struct 배열:

```c
typedef struct {
    int row;
    int col;
    int value;
} Entry;
```

```c
Entry entries[] = {
    {0, 3, 5},
    {2, 1, 7},
    {3, 3, 9}
};
```

장점:

```text
- 구현이 쉽다
- non-zero 값만 저장한다
- sparse matrix를 처음 이해하기 좋다
```

단점:

```text
- 특정 위치 M[i][j]를 찾으려면 entries를 탐색해야 할 수 있다
- row별 연산이 최적화되어 있지 않다
```

---

## 5.3 CSR: Compressed Sparse Row

CSR은 row 기준 연산에 강하다.

세 배열을 사용한다.

```text
values
col_indices
row_ptr
```

예시 행렬:

```text
[
  [0, 0, 0, 5],
  [0, 0, 0, 0],
  [0, 7, 0, 0],
  [0, 0, 0, 9]
]
```

non-zero 값:

```text
row 0: col 3 -> 5
row 1: none
row 2: col 1 -> 7
row 3: col 3 -> 9
```

CSR:

```text
values      = [5, 7, 9]
col_indices = [3, 1, 3]
row_ptr     = [0, 1, 1, 2, 3]
```

`row_ptr`의 의미:

```text
row i의 non-zero 원소는
values[row_ptr[i] ... row_ptr[i+1]-1]
```

예:

```text
row 2:
row_ptr[2] = 1
row_ptr[3] = 2

따라서 values[1 ... 1] = [7]
col_indices[1] = 1
```

즉 `(2, 1) = 7`.

---

## 5.4 Hash Table 내부 구조

Hash table은 내부적으로 배열을 가진다.

```text
bucket array
```

예:

```text
table size = 8
```

```text
index:  0   1   2   3   4   5   6   7
table: [_] [_] [_] [_] [_] [_] [_] [_]
```

key `"Alice"`를 넣는다.

```text
hash("Alice") = 123456
index = 123456 % 8 = 0
```

그러면 table[0]에 저장한다.

```text
table[0] = ("Alice", 90)
```

key `"Bob"`:

```text
hash("Bob") = 98123
index = 98123 % 8 = 3
```

```text
table[3] = ("Bob", 85)
```

---

# 6. Collision

Hash table의 핵심 문제는 collision이다.

서로 다른 key가 같은 index로 가는 상황이다.

```text
hash("Alice") % 8 = 3
hash("David") % 8 = 3
```

둘 다 bucket 3으로 가면 충돌이 생긴다.

```text
table[3]에 두 값을 어떻게 저장할 것인가?
```

대표 해결 방식:

| 방식                | 아이디어                               |
| ----------------- | ---------------------------------- |
| Separate chaining | bucket마다 linked list 또는 vector를 둔다 |
| Open addressing   | 빈 bucket을 찾아 다른 위치에 저장한다           |

이번 Lecture에서는 intro만 한다. Collision은 더 깊은 section에서 다시 다루는 주제다. 자료에서도 hash collision은 별도 section에서 다룬다고 되어 있습니다. 

---

# 7. Load Factor

Hash table 성능에서 중요한 값이 있다.

```text
load factor = n / m
```

| 기호  | 의미        |
| --- | --------- |
| `n` | 저장된 원소 개수 |
| `m` | bucket 개수 |

예:

```text
n = 6
m = 8

load factor = 6 / 8 = 0.75
```

자료에서도 load factor를 `α = n / m`으로 정의하고, 성능을 위해 보통 `α < 0.75`로 유지한다고 설명합니다. 

왜 중요할까?

bucket이 너무 꽉 차면 collision이 많아진다.

collision이 많아지면 평균 O(1)이 무너진다.

그래서 hash table은 일정 load factor를 넘으면 resize한다.

```text
bucket 수 증가
기존 key들을 다시 hash
새 table에 재배치
```

이를 rehashing이라고 한다.

---

# 8. 단계별 예시

## 예시 1. Sparse Matrix COO 저장

행렬:

```text
[
  [0, 0, 8],
  [0, 0, 0],
  [5, 0, 0]
]
```

일반 2D array 저장:

```text
9개 원소 저장
```

COO 저장:

```text
(0, 2, 8)
(2, 0, 5)
```

2개 entry만 저장한다.

C 표현:

```c
typedef struct {
    int row;
    int col;
    int value;
} Entry;

Entry sparse[] = {
    {0, 2, 8},
    {2, 0, 5}
};
```

특정 위치 찾기:

```c
int get_value(Entry entries[], int nnz, int row, int col) {
    for (int i = 0; i < nnz; i++) {
        if (entries[i].row == row && entries[i].col == col) {
            return entries[i].value;
        }
    }
    return 0;
}
```

복잡도:

```text
Time: O(nnz)
Space: O(nnz)
```

`nnz`는 number of non-zero elements다.

---

## 예시 2. 간단한 Hash Table

문자열까지 구현하면 복잡하므로, 정수 key로 단순화한다.

```c
#define TABLE_SIZE 8

typedef struct {
    int used;
    int key;
    int value;
} Entry;

Entry table[TABLE_SIZE];
```

Hash function:

```c
int hash(int key) {
    return key % TABLE_SIZE;
}
```

삽입:

```c
void insert(int key, int value) {
    int idx = hash(key);

    table[idx].used = 1;
    table[idx].key = key;
    table[idx].value = value;
}
```

검색:

```c
int search(int key, int *out) {
    int idx = hash(key);

    if (table[idx].used && table[idx].key == key) {
        *out = table[idx].value;
        return 1;
    }

    return 0;
}
```

하지만 이 코드는 collision을 처리하지 않는다.

예:

```text
key = 1  -> index 1
key = 9  -> index 1
```

`9`를 넣으면 `1`이 덮어씌워질 수 있다.

이게 실패 사례다.

---

# 9. 실제 응용

## 9.1 Sparse Matrix 실제 응용

Sparse matrix는 다음 분야에서 중요하다.

| 분야                    | 이유                            |
| --------------------- | ----------------------------- |
| Graph                 | adjacency matrix가 대부분 0일 수 있음 |
| Machine Learning      | feature vector가 sparse한 경우 많음 |
| Search engine         | document-term matrix가 sparse함 |
| Recommendation system | user-item matrix가 sparse함     |
| Scientific computing  | 대형 선형 시스템에서 sparse matrix 사용  |
| Robotics / SLAM       | sparse optimization 문제 등장     |

예를 들어 추천 시스템을 보자.

사용자가 1,000,000명이고 영화가 100,000개라고 하자.

전체 user-item matrix는:

```text
1,000,000 × 100,000 = 100,000,000,000
```

하지만 각 사용자가 실제 평가한 영화는 수십 개일 수 있다.

그러면 대부분의 칸은 비어 있다.

이런 경우 dense matrix로 저장하면 메모리가 폭발한다.

---

## 9.2 Hash Table 실제 응용

Hash table은 거의 모든 소프트웨어에 나온다.

| 사용처               | 예                                       |
| ----------------- | --------------------------------------- |
| Compiler          | symbol table                            |
| Interpreter       | variable environment                    |
| Database          | hash index                              |
| Web server        | header lookup                           |
| Cache             | key-value cache                         |
| OS                | process table, file descriptor table    |
| Algorithm problem | frequency counting, duplicate detection |

예: 단어 빈도 세기.

```cpp
#include <unordered_map>
#include <string>
#include <vector>

std::unordered_map<std::string, int> count_words(
    const std::vector<std::string>& words
) {
    std::unordered_map<std::string, int> freq;

    for (const auto& word : words) {
        freq[word]++;
    }

    return freq;
}
```

입력:

```text
["apple", "banana", "apple", "orange", "banana", "apple"]
```

결과:

```text
apple: 3
banana: 2
orange: 1
```

배열/list로 매번 찾으면 비효율적이다.

Hash table은 평균적으로 빠르다.

---

# 10. 흔한 오해

## 오해 1. “Sparse Matrix는 무조건 빠르다”

아니다.

Sparse matrix는 메모리를 아끼지만, 접근 방식에 따라 느릴 수 있다.

예를 들어 COO에서 `M[i][j]`를 찾으려면 entries를 순회해야 할 수 있다.

```text
get(i, j): O(nnz)
```

Dense array에서는:

```text
M[i][j]: O(1)
```

즉 sparse matrix는:

```text
메모리 절약 ↔ 접근 비용 증가
```

라는 trade-off가 있다.

---

## 오해 2. “0이 많으면 무조건 sparse matrix를 써야 한다”

대부분 0이어도 연산 패턴에 따라 다르다.

Sparse matrix가 좋은 경우:

```text
- non-zero 값만 순회한다
- 행 단위 연산이 많다
- 메모리가 중요한 병목이다
```

Dense matrix가 좋은 경우:

```text
- 랜덤 접근이 매우 많다
- 크기가 작다
- SIMD/GPU dense 연산이 유리하다
```

---

## 오해 3. “Hash table은 항상 O(1)이다”

정확히는:

```text
Average: O(1)
Worst case: O(N)
```

collision이 심하면 한 bucket에 많은 값이 몰릴 수 있다.

특히 나쁜 hash function을 쓰면 성능이 크게 무너진다.

---

## 오해 4. “Hash function은 아무렇게나 만들면 된다”

아니다.

좋은 hash function은 다음 성질이 필요하다.

```text
- 같은 key는 항상 같은 hash
- 다른 key들은 가능하면 고르게 분포
- 계산이 너무 비싸지 않음
```

나쁜 예:

```c
int hash(int key) {
    return 0;
}
```

모든 key가 index 0으로 간다.

그러면 hash table은 사실상 linked list처럼 변한다.

검색은 O(N)이 된다.

---

# 11. 반례 또는 실패 사례

## 실패 사례 1. Dense matrix로 거대한 sparse graph 저장

노드가 100,000개인 그래프가 있다고 하자.

Adjacency matrix를 만들면:

```text
100,000 × 100,000 = 10,000,000,000 entries
```

bool 하나를 1 byte로 잡아도:

```text
10 GB
```

실제로 edge가 300,000개뿐이라면 대부분 0이다.

이런 경우 adjacency list나 sparse representation이 적합하다.

---

## 실패 사례 2. Collision을 처리하지 않는 hash table

앞에서 본 코드:

```c
void insert(int key, int value) {
    int idx = key % TABLE_SIZE;

    table[idx].used = 1;
    table[idx].key = key;
    table[idx].value = value;
}
```

문제:

```text
key 1과 key 9는 TABLE_SIZE가 8일 때 같은 index 1로 간다.
```

```text
1 % 8 = 1
9 % 8 = 1
```

나중 값이 이전 값을 덮어쓴다.

Hash table에서 collision 처리는 선택 사항이 아니라 필수다.

---

## 실패 사례 3. Load factor를 관리하지 않음

bucket이 8개인데 1000개를 넣는다고 하자.

```text
load factor = 1000 / 8 = 125
```

bucket 하나당 평균 125개가 들어간다.

Separate chaining이라면 각 bucket의 list가 길어진다.

검색은 평균 O(1)에 가까워지기 어렵다.

따라서 rehashing이 필요하다.

---

# 12. 확인 문제

## 문제 1

다음 행렬은 sparse matrix인가?

```text
[
  [0, 0, 0, 0],
  [0, 5, 0, 0],
  [0, 0, 0, 0],
  [7, 0, 0, 0]
]
```

정답:

```text
그렇다. 대부분의 원소가 0이다.
```

---

## 문제 2

COO 방식으로 위 행렬을 표현해라.

정답:

```text
(1, 1, 5)
(3, 0, 7)
```

---

## 문제 3

Hash table에서 hash function의 역할은?

정답:

```text
key를 bucket index로 변환하는 것이다.
```

---

## 문제 4

Hash table의 평균 search 복잡도와 최악 search 복잡도는?

정답:

```text
Average: O(1)
Worst case: O(N)
```

---

## 문제 5

Load factor가 너무 커지면 어떤 문제가 생기는가?

정답:

```text
collision이 증가하고, hash table의 평균 성능이 나빠질 수 있다.
```

---

# 13. 실습 과제

## 과제 1. COO Sparse Matrix 구현

다음 구조체를 사용해라.

```c
typedef struct {
    int row;
    int col;
    int value;
} Entry;
```

구현할 함수:

```c
int get_value(Entry entries[], int nnz, int row, int col);
```

조건:

```text
- 해당 위치에 non-zero entry가 있으면 value 반환
- 없으면 0 반환
```

---

## 과제 2. Sparse Matrix Memory 계산

다음 조건에서 dense와 sparse 저장 방식의 메모리를 비교해라.

```text
matrix size: 10,000 × 10,000
non-zero elements: 20,000
int size: 4 bytes
COO entry: row, col, value = int 3개
```

계산할 것:

```text
1. dense matrix memory
2. COO memory
3. 몇 배 차이인지
```

---

## 과제 3. Collision 없는 단순 Hash Table 구현

일단 collision이 없다고 가정하고 정수 key-value table을 구현해라.

```c
#define TABLE_SIZE 16

typedef struct {
    int used;
    int key;
    int value;
} Entry;
```

구현할 함수:

```c
void init_table(Entry table[]);
int insert(Entry table[], int key, int value);
int search(Entry table[], int key, int *out);
```

단, collision이 발생하면 insert는 실패하도록 만들어라.

---

## 과제 4. Collision 사례 만들기

`TABLE_SIZE = 8`일 때, 다음 key들의 hash index를 계산해라.

```text
1, 9, 17, 25
```

hash function:

```c
int hash(int key) {
    return key % 8;
}
```

질문:

```text
왜 이 hash function은 이 입력에서 문제가 되는가?
```

---

# 14. 핵심 정리

```text
Sparse Matrix는 “0을 저장하지 않는 전략”이고,
Hash Table은 “key를 index로 바꿔 빠르게 찾는 전략”이다.
```

| 개념                | 핵심                       |
| ----------------- | ------------------------ |
| Sparse Matrix     | 대부분 0인 행렬                |
| COO               | `(row, col, value)` 저장   |
| CSR               | row 기준 압축 저장             |
| Hash Table        | key-value mapping        |
| Hash Function     | key → index              |
| Collision         | 서로 다른 key가 같은 bucket으로 감 |
| Load Factor       | `n / m`                  |
| Hash Table 평균 복잡도 | O(1)                     |
| Hash Table 최악 복잡도 | O(N)                     |

## 반드시 기억할 문장

> Sparse Matrix와 Hash Table은 모두 “모든 것을 그대로 저장하거나 순회하지 말자”는 생각에서 출발한다.