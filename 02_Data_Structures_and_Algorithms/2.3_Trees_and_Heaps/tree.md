# Part 2.3 — Trees & Heaps 전체 커리큘럼

| 구간        | Lecture | 핵심 내용                                         |
| --------- | ------: | --------------------------------------------- |
| 기초        |     1–2 | Tree의 수학적 정의, 용어, Binary Tree 종류              |
| BST       |     3–7 | BST invariant, C 구현, 순회, 검증, 고전 문제            |
| Heap      |    8–12 | Heap 구조, 배열 표현, 연산, Priority Queue, Heap Sort |
| 균형·외부 메모리 |   13–15 | AVL, Red-Black Tree, Rotation, B-Tree/B+Tree  |
| 특수 트리     |   16–17 | Trie, Segment Tree                            |
| 알고리즘 응용   |   18–19 | 고전 트리 알고리즘, Huffman Coding                    |
| 면접·통합     |   20–21 | 문제 패턴, 자료구조 선택, 시스템 수준 비교                     |

---

# Lecture 1. Tree Fundamentals

## 1. 핵심 질문

이번 강의에서 답해야 할 질문은 다음과 같다.

1. Tree는 정확히 무엇인가?
2. Tree와 Graph는 어떤 관계인가?
3. 왜 트리는 `N`개의 정점을 가지면 정확히 `N - 1`개의 간선을 가지는가?
4. 왜 임의의 두 정점 사이에 경로가 정확히 하나뿐인가?
5. Root, parent, child, leaf, depth, height 같은 용어는 어떻게 정의하는가?
6. 수학적인 tree와 컴퓨터 프로그램에서 사용하는 rooted tree는 무엇이 다른가?
7. 트리는 메모리에서 어떻게 표현되며, 실제 성능은 왜 Big-O만으로 설명되지 않는가?

---

# 2. 형식적 정의

## 2.1 Graph의 정의

그래프는 일반적으로 다음과 같이 정의한다.

[
G = (V, E)
]

* (V): 정점(vertex, node)의 집합
* (E): 간선(edge)의 집합

무방향 그래프에서는 간선을 두 정점의 순서 없는 쌍으로 표현한다.

[
E \subseteq {{u,v} \mid u,v \in V,\ u \neq v}
]

예를 들어:

[
V = {A,B,C,D}
]

[
E = {{A,B}, {A,C}, {C,D}}
]

이를 그림으로 표현하면 다음과 같다.

```text
    A
   / \
  B   C
       \
        D
```

---

## 2.2 Tree의 형식적 정의

무방향 그래프 (T = (V,E))가 다음 두 조건을 만족하면 tree라고 한다.

1. **Connected**

   * 모든 두 정점 사이에 경로가 존재한다.

2. **Acyclic**

   * cycle이 존재하지 않는다.

즉:

[
\boxed{\text{Tree = connected acyclic undirected graph}}
]

여기서 매우 중요한 점은, 수학적 정의의 tree에는 본래 root가 없다는 것이다.

```text
A — B — C
    |
    D
```

이 그래프는 tree이지만, 아직 어느 정점도 root로 지정되지 않았다.

`B`를 root로 선택하면:

```text
      B
    / | \
   A  C  D
```

`A`를 root로 선택하면:

```text
A
|
B
|\
C D
```

같은 무방향 tree라도 root를 어디에 두느냐에 따라 parent-child 관계, depth, subtree가 달라진다.

---

## 2.3 Rooted Tree의 정의

Tree (T = (V,E))에서 특정 정점 (r \in V)를 root로 지정한 구조를 **rooted tree**라고 한다.

[
(T,r)
]

Root가 정해지면 각 정점은 root로부터의 위치에 따라 계층적 관계를 갖는다.

* root를 제외한 모든 정점은 정확히 하나의 parent를 가진다.
* 정점은 0개 이상의 child를 가질 수 있다.
* root에서 각 정점으로 가는 유일한 경로가 존재한다.

```text
        A             depth 0
      /   \
     B     C           depth 1
    / \     \
   D   E     F         depth 2
```

---

# 3. 직관적 설명

## 3.1 Tree는 “갈림길은 있지만 되돌아오는 고리는 없는 구조”

Tree를 직관적으로 이해하면 다음과 같다.

* 한 지점에서 여러 방향으로 갈라질 수 있다.
* 그러나 갈라진 길들이 나중에 다시 합쳐지지 않는다.
* 어떤 두 지점을 연결하는 방법은 정확히 하나뿐이다.

예를 들어 회사 조직도는 다음과 같은 tree로 모델링할 수 있다.

```text
CEO
├── Engineering
│   ├── Backend
│   └── Embedded
└── Business
    ├── Sales
    └── Marketing
```

CEO에서 Embedded 팀으로 가는 경로는 하나뿐이다.

```text
CEO → Engineering → Embedded
```

다른 경로가 하나 더 존재한다면 조직 구조가 단순한 tree가 아니라 일반 graph에 가까워진다.

---

## 3.2 Graph는 관계를 표현하고, Tree는 계층을 표현한다

Graph는 일반적인 관계를 표현한다.

```text
Alice ─ Bob
  │    ╱ │
  │  ╱   │
Carol ─ Dave
```

친구 관계, 도로망, 네트워크 연결은 cycle이나 여러 경로를 가질 수 있다.

Tree는 관계에 제한을 둔다.

```text
       Root
      /    \
     A      B
    / \
   C   D
```

이 제한 덕분에 다음이 쉬워진다.

* 부모와 자식 관계 정의
* 재귀적 분해
* 경로 추적
* 탐색 범위 축소
* 계층적 데이터 표현
* 분할 정복

---

# 4. 왜 필요한가

트리는 단순한 자료구조 하나가 아니라, 컴퓨터과학의 여러 분야를 연결하는 핵심 모델이다.

## 4.1 파일 시스템

```text
/
├── home
│   └── user
│       ├── code
│       └── documents
└── etc
    └── config
```

디렉터리는 하위 디렉터리와 파일을 포함한다.

실제 파일 시스템은 symbolic link, hard link 등의 존재로 완전한 tree가 아닐 수 있지만, 기본 탐색 모델은 tree와 유사하다.

---

## 4.2 프로그램 구조

컴파일러는 코드를 Abstract Syntax Tree, 즉 AST로 표현한다.

```c
x = a + b * c;
```

개념적 AST:

```text
        =
       / \
      x   +
         / \
        a   *
           / \
          b   c
```

연산자 우선순위가 tree의 구조에 반영된다.

---

## 4.3 검색과 정렬

Binary Search Tree는 값의 대소 관계를 계층적으로 저장한다.

```text
        8
       / \
      3   10
     / \    \
    1   6    14
```

구조가 균형을 유지한다면 검색 범위를 매 단계 절반에 가깝게 줄일 수 있다.

---

## 4.4 운영체제와 시스템

* 프로세스 parent-child 관계
* 페이지 테이블의 다단계 구조
* 파일 시스템 디렉터리
* 스케줄링용 heap
* 메모리 allocator의 free block tree
* radix tree 기반 주소 탐색
* device tree
* hierarchical timer wheel과 일부 timer tree
* database index

Embedded 시스템에서도 tree는 중요하다.

* Device Tree를 통한 하드웨어 기술
* 우선순위 기반 이벤트 처리
* 센서 계층 관리
* 상태 머신의 계층적 표현
* 제한된 메모리에서의 검색 구조 설계

---

# 5. 내부 원리

## 5.1 Tree와 Graph의 핵심 관계

모든 tree는 graph이지만, 모든 graph가 tree는 아니다.

```text
Graph
├── Tree
├── Cyclic graph
├── Disconnected graph
├── Directed graph
└── 기타 구조
```

Tree는 graph에 다음 제약을 추가한 구조다.

```text
연결되어 있어야 함
+
cycle이 없어야 함
```

---

## 5.2 Tree의 동치 조건

정점 수가 (N)인 유한 무방향 그래프 (G)에 대해, 다음 조건들은 서로 동치다.

### 조건 A

(G)는 connected이고 acyclic이다.

### 조건 B

임의의 두 정점 사이에 simple path가 정확히 하나 존재한다.

### 조건 C

(G)는 connected이고 간선 수가 (N-1)이다.

### 조건 D

(G)는 acyclic이고 간선 수가 (N-1)이다.

### 조건 E

(G)는 connected이며, 어떤 간선 하나라도 제거하면 disconnected가 된다.

이를 **minimally connected graph**라고 한다.

### 조건 F

(G)는 acyclic이며, 서로 연결되지 않은 두 정점 사이에 간선을 하나 추가하면 정확히 하나의 cycle이 생긴다.

이를 **maximally acyclic graph**라고 볼 수 있다.

따라서 다음은 모두 tree를 설명하는 서로 다른 방식이다.

[
\boxed{
\text{connected + acyclic}
\iff
\text{unique path}
\iff
|E| = |V|-1\text{ with suitable condition}
}
]

주의할 점은 `간선 수가 N-1이다`만으로는 tree라고 결론 내릴 수 없다는 것이다.

---

# 6. 자료구조 invariant

## 6.1 Invariant란 무엇인가

Invariant는 자료구조가 올바른 상태일 때 항상 유지되어야 하는 조건이다.

Tree의 핵심 invariant는 다음과 같다.

### 무방향 tree invariant

1. 모든 정점이 연결되어 있다.
2. cycle이 없다.
3. 정점이 (N)개라면 간선은 (N-1)개다.
4. 임의의 두 정점 사이에 경로가 정확히 하나다.

### Rooted tree invariant

1. root는 parent가 없다.
2. root가 아닌 모든 정점은 parent가 정확히 하나다.
3. parent 관계를 계속 따라가면 언젠가 root에 도달한다.
4. parent 관계에 cycle이 존재하지 않는다.
5. 모든 정점은 root에서 도달 가능하다.

---

## 6.2 잘못된 구조

```text
    A
   / \
  B   C
   \ /
    D
```

여기서 D가 B와 C 모두를 parent로 가진다고 가정하면 일반적인 rooted tree가 아니다.

경로가 두 개 존재한다.

```text
A → B → D
A → C → D
```

또한 무방향으로 보면 cycle이 생긴다.

```text
A → B → D → C → A
```

---

# 7. Tree의 기본 용어

다음 tree를 기준으로 살펴보자.

```text
             A
          /  |  \
         B   C   D
        / \       \
       E   F       G
          / \
         H   I
```

---

## 7.1 Root

계층 구조의 시작점이다.

위 tree에서는:

```text
root = A
```

Root는 parent를 가지지 않는다.

---

## 7.2 Parent

정점 (u)가 정점 (v)보다 root에 한 단계 더 가까우며 직접 연결되어 있다면, (u)는 (v)의 parent다.

```text
parent(E) = B
parent(H) = F
```

---

## 7.3 Child

Parent의 반대 관계다.

```text
children(A) = {B, C, D}
children(F) = {H, I}
```

---

## 7.4 Sibling

같은 parent를 가지는 정점이다.

```text
B, C, D는 sibling
E, F는 sibling
H, I는 sibling
```

Sibling 관계는 일반적으로 자기 자신을 포함하지 않는다.

---

## 7.5 Leaf

Child가 없는 정점이다.

```text
leaves = {C, E, G, H, I}
```

Leaf는 terminal node 또는 external node라고도 부른다.

단, 문헌에 따라 external node의 정의가 다르게 사용될 수 있으므로 주의해야 한다.

---

## 7.6 Internal Node

하나 이상의 child를 가진 정점이다.

```text
internal nodes = {A, B, D, F}
```

어떤 정의에서는 root를 internal node에 포함하고, 어떤 맥락에서는 “root가 아닌 internal node”를 따로 구분한다.

일반적으로 자료구조에서는 child가 있으면 root도 internal node로 본다.

---

## 7.7 Subtree

정점 (v)와 그 정점의 모든 descendant로 구성된 tree다.

F를 root로 하는 subtree:

```text
    F
   / \
  H   I
```

B를 root로 하는 subtree:

```text
    B
   / \
  E   F
     / \
    H   I
```

중요한 재귀적 관점은 다음과 같다.

> Tree는 root와 여러 개의 subtree로 이루어진다.

이 관점이 대부분의 tree 알고리즘을 가능하게 한다.

---

## 7.8 Degree

### 일반 graph에서 정점의 degree

정점에 연결된 간선 수다.

무방향 tree에서 B의 graph degree는 다음과 같다.

```text
A—B
  / \
 E   F
```

B에 연결된 간선은 A, E, F 방향의 총 3개다.

[
\deg(B) = 3
]

### Rooted tree에서 node degree

자료구조 문맥에서는 node가 가진 child 수를 degree라고 부르기도 한다.

```text
children(B) = {E, F}
```

따라서 rooted-tree degree는 2다.

문맥에 따라 결과가 달라질 수 있다.

| 기준                      | B의 degree |
| ----------------------- | --------: |
| Graph degree            |         3 |
| Rooted-tree child count |         2 |

면접에서는 어떤 정의를 사용하는지 명확히 말해야 한다.

### Tree의 degree

Tree 내 모든 node degree 중 최댓값을 tree의 degree라고 부르기도 한다.

위 tree에서는 A의 child 수가 3이므로:

```text
degree of rooted tree = 3
```

---

## 7.9 Path

정점들의 나열:

[
v_0,v_1,\ldots,v_k
]

에서 연속된 정점들이 간선으로 연결되어 있다면 path라고 한다.

A에서 I까지의 경로:

```text
A → B → F → I
```

### Path length

경로에 포함된 간선 수다.

```text
A → B → F → I
```

간선은 3개이므로:

[
\text{path length} = 3
]

정점이 4개라고 해서 길이가 4인 것이 아니다.

---

## 7.10 Ancestor와 Descendant

정점 (u)가 root에서 (v)로 가는 경로 위에 있다면 (u)는 (v)의 ancestor다.

```text
ancestors(I) = {A, B, F}
```

정의에 따라 자기 자신을 포함하는 경우가 있다.

* proper ancestor: 자기 자신 제외
* ancestor-or-self: 자기 자신 포함

Descendant는 반대 관계다.

```text
descendants(B) = {E, F, H, I}
```

---

## 7.11 Depth

정점 (v)의 depth는 root에서 (v)까지의 경로 길이다.

[
\operatorname{depth}(v)
=======================

\text{number of edges from root to }v
]

따라서:

```text
depth(A) = 0
depth(B) = 1
depth(F) = 2
depth(I) = 3
```

Root의 depth는 0이다.

---

## 7.12 Height

정점 (v)의 height는 (v)에서 어떤 leaf까지 내려가는 가장 긴 경로의 길이다.

[
\operatorname{height}(v)
========================

\max_{\ell \text{ is a leaf below }v}
\operatorname{distance}(v,\ell)
]

따라서:

```text
height(H) = 0
height(F) = 1
height(B) = 2
height(A) = 3
```

Leaf의 height는 0이다.

Tree의 height는 root의 height다.

[
\operatorname{height}(T)
========================

\operatorname{height}(\text{root})
]

---

## 7.13 Level

Level은 문헌마다 정의가 다르다.

### 정의 1

```text
level(root) = 0
```

이 경우 level과 depth가 같다.

### 정의 2

```text
level(root) = 1
```

이 경우:

[
\operatorname{level}(v)
=======================

\operatorname{depth}(v)+1
]

따라서 면접이나 문서에서는 다음처럼 정의를 먼저 명시하는 것이 안전하다.

> 여기서는 root의 depth와 level을 모두 0으로 정의하겠습니다.

---

# 8. Depth와 Height의 차이

다음 정점 F를 보자.

```text
             A
            /
           B
          / \
         E   F
            / \
           H   I
```

F에 대해:

* root에서 F까지 올라온 거리: 2
* F에서 가장 깊은 leaf까지 내려갈 거리: 1

따라서:

```text
depth(F)  = 2
height(F) = 1
```

## 직관적 구분

* **Depth**: 위에서부터 얼마나 내려왔는가
* **Height**: 아래로 얼마나 더 내려갈 수 있는가

```text
root
 │
 │ depth: root에서 현재 node까지
 ▼
current node
 │
 │ height: 현재 node에서 가장 먼 leaf까지
 ▼
leaf
```

---

# 9. 단계별 예시

다음 무방향 그래프를 살펴보자.

```text
1 — 2 — 3
    |
    4 — 5
```

## 단계 1: 정점과 간선 수 확인

```text
V = {1, 2, 3, 4, 5}
```

정점 수:

[
|V| = 5
]

간선:

```text
(1,2), (2,3), (2,4), (4,5)
```

간선 수:

[
|E| = 4 = |V|-1
]

---

## 단계 2: Connected 확인

모든 정점이 서로 연결되어 있다.

예:

```text
1 → 2 → 4 → 5
3 → 2 → 4 → 5
```

---

## 단계 3: Cycle 확인

어떤 지점에서 출발해 서로 다른 간선들을 따라 원래 지점으로 돌아오는 경로가 없다.

따라서 acyclic이다.

---

## 단계 4: Tree 결론

Connected이고 acyclic이므로 tree다.

---

## 단계 5: Root 지정

2를 root로 지정하자.

```text
      2
    / | \
   1  3  4
          \
           5
```

관계는 다음과 같다.

```text
root: 2
children(2): 1, 3, 4
parent(5): 4
leaves: 1, 3, 5
internal nodes: 2, 4
depth(5): 2
height(4): 1
height(2): 2
```

---

# 10. 왜 `N`개 정점의 Tree는 `N-1`개 간선을 가지는가

## 10.1 직관적 설명

정점 하나에서 시작한다.

```text
A
```

정점 수 1, 간선 수 0이다.

새 정점을 tree에 추가하려면 기존 tree의 정점 하나와 연결해야 한다.

```text
A — B
```

정점 하나를 추가할 때 간선도 정확히 하나 추가한다.

* 간선을 추가하지 않으면 disconnected
* 두 개 이상의 간선으로 연결하면 cycle 발생 가능

따라서 정점을 총 (N-1)개 추가하는 동안 간선도 (N-1)개 추가된다.

[
|E| = N-1
]

---

## 10.2 귀납법을 이용한 증명 아이디어

### Base case

정점이 하나인 tree:

[
N=1,\qquad |E|=0=N-1
]

성립한다.

### Inductive hypothesis

정점이 (k)개인 모든 tree는 (k-1)개의 간선을 가진다고 가정한다.

### Inductive step

정점이 (k+1)개인 tree에는 leaf가 적어도 하나 존재한다.

Leaf (v)와 그 leaf에 연결된 유일한 간선을 제거한다.

남은 그래프는:

* 정점 (k)개
* 여전히 connected
* 여전히 acyclic

따라서 귀납 가정에 의해 간선은 (k-1)개다.

제거했던 간선 하나를 다시 추가하면:

[
(k-1)+1=k
]

즉:

[
|E|=k=(k+1)-1
]

따라서 모든 유한 tree에서 성립한다.

---

# 11. 왜 두 정점 사이의 경로는 정확히 하나인가

## 존재성

Tree는 connected이므로 임의의 두 정점 (u,v) 사이에 적어도 하나의 경로가 존재한다.

## 유일성

두 개의 서로 다른 simple path (P_1,P_2)가 있다고 가정하자.

두 경로는 어느 지점에서 갈라졌다가 결국 (v)에서 다시 만난다.

```text
      x
     / \
    ... ...
     \ /
      y
```

두 경로를 합치면 cycle이 만들어진다.

하지만 tree는 acyclic이어야 한다.

모순이다.

따라서 두 정점 사이에는 경로가 정확히 하나다.

[
\boxed{\text{connectedness gives existence, acyclicity gives uniqueness}}
]

---

# 12. 반례 또는 실패 사례

## 12.1 `N-1`개의 간선만 있으면 tree인가?

아니다.

```text
1 — 2 — 3
 \_____/

4
```

정점 수:

[
N=4
]

간선 수:

[
3=N-1
]

하지만:

* 1, 2, 3은 cycle을 이룬다.
* 정점 4는 disconnected다.

따라서 tree가 아니다.

즉:

[
|E|=N-1
]

하나만으로는 충분하지 않다.

다음 중 하나를 추가로 확인해야 한다.

* connected
* acyclic

---

## 12.2 Connected이기만 하면 tree인가?

아니다.

```text
A — B
|   |
D — C
```

모든 정점이 연결되어 있지만 cycle이 존재한다.

---

## 12.3 Acyclic이기만 하면 tree인가?

아니다.

```text
A — B     C — D
```

Cycle은 없지만 disconnected다.

이 구조는 하나의 tree가 아니라 여러 tree의 집합인 **forest**다.

---

# 13. Forest

Forest는 cycle이 없는 무방향 그래프다.

즉, 여러 개의 tree가 서로 분리되어 있는 구조일 수 있다.

```text
A — B — C

D — E

F
```

이 graph는 3개의 connected component로 구성된 forest다.

정점 수가 (N), connected component 수가 (K)라면 forest의 간선 수는:

[
|E| = N-K
]

각 component (i)가 (n_i)개의 정점을 가진 tree라면:

[
|E|
===

# \sum_{i=1}^{K}(n_i-1)

# \sum_{i=1}^{K}n_i-K

N-K
]

Tree는 connected component가 하나인 forest다.

[
K=1 \Rightarrow |E|=N-1
]

---

# 14. 실제 C 코드 예시

이번 강의에서는 특정 binary tree가 아니라, child 수가 제한되지 않은 일반 rooted tree를 표현해본다.

## 14.1 표현 방법 1: 고정 크기 child 배열

```c
#define MAX_CHILDREN 4

typedef struct TreeNode {
    int value;
    struct TreeNode *children[MAX_CHILDREN];
    size_t child_count;
} TreeNode;
```

장점:

* 구현이 단순하다.
* child pointer가 node 내부에 연속적으로 존재한다.
* 별도 child 배열 allocation이 필요 없다.

단점:

* 최대 child 수가 고정된다.
* child가 적어도 사용하지 않는 pointer 공간이 낭비된다.
* `MAX_CHILDREN`을 넘는 구조를 표현할 수 없다.

---

## 14.2 표현 방법 2: 동적 child 배열

```c
#include <stddef.h>

typedef struct TreeNode {
    int value;

    struct TreeNode **children;
    size_t child_count;
    size_t child_capacity;
} TreeNode;
```

`children`은 `TreeNode *` 원소들을 저장하는 동적 배열이다.

```text
TreeNode
┌───────────────────┐
│ value             │
│ children ─────────┼──────┐
│ child_count       │      │
│ child_capacity    │      │
└───────────────────┘      │
                           ▼
                    ┌─────────────┐
                    │ child ptr 0 │
                    │ child ptr 1 │
                    │ child ptr 2 │
                    └─────────────┘
```

---

## 14.3 안전한 node 생성

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>
#include <stdint.h>

typedef struct TreeNode {
    int value;
    struct TreeNode **children;
    size_t child_count;
    size_t child_capacity;
} TreeNode;

TreeNode *tree_node_create(int value)
{
    TreeNode *node = malloc(sizeof(*node));

    if (node == NULL) {
        return NULL;
    }

    node->value = value;
    node->children = NULL;
    node->child_count = 0;
    node->child_capacity = 0;

    return node;
}
```

### Ownership

이 함수가 반환한 node의 ownership은 호출자에게 있다.

호출자는 최종적으로 다음 중 하나를 수행해야 한다.

* tree에 연결하고 tree 전체를 파괴할 때 함께 `free`
* 연결하지 않았다면 직접 `free`

---

## 14.4 Child 추가

```c
#include <stdlib.h>
#include <stdint.h>
#include <stddef.h>

int tree_node_add_child(TreeNode *parent, TreeNode *child)
{
    if (parent == NULL || child == NULL) {
        return 0;
    }

    if (parent == child) {
        return 0;
    }

    if (parent->child_count == parent->child_capacity) {
        size_t new_capacity;

        if (parent->child_capacity == 0) {
            new_capacity = 2;
        } else {
            if (parent->child_capacity > SIZE_MAX / 2) {
                return 0;
            }

            new_capacity = parent->child_capacity * 2;
        }

        if (new_capacity > SIZE_MAX / sizeof(*parent->children)) {
            return 0;
        }

        TreeNode **new_children =
            realloc(parent->children,
                    new_capacity * sizeof(*parent->children));

        if (new_children == NULL) {
            return 0;
        }

        parent->children = new_children;
        parent->child_capacity = new_capacity;
    }

    parent->children[parent->child_count] = child;
    parent->child_count++;

    return 1;
}
```

## 중요한 한계

이 함수는 다음을 자동으로 검증하지 않는다.

* `child`가 이미 다른 parent를 가지는가?
* `child`가 `parent`의 ancestor인가?
* 연결 시 cycle이 생기는가?
* 같은 child가 중복 삽입되는가?

따라서 이것은 메모리 배열 관점에서 child를 추가할 뿐, tree invariant 전체를 보장하지 않는다.

완전한 안전성을 원한다면 다음 정보가 필요하다.

```c
struct TreeNode *parent;
```

그리고 삽입 전에 ancestor chain을 검사해야 한다.

---

## 14.5 Parent pointer를 포함한 구조

```c
typedef struct TreeNode {
    int value;

    struct TreeNode *parent;
    struct TreeNode **children;

    size_t child_count;
    size_t child_capacity;
} TreeNode;
```

이 경우 invariant를 명시적으로 유지할 수 있다.

```text
root->parent == NULL
child->parent == parent
```

하지만 parent pointer가 추가되면 다음 문제가 생긴다.

* node당 pointer 하나만큼 메모리 증가
* 이동이나 재연결 시 양방향 관계를 모두 갱신해야 함
* 한쪽만 수정하면 구조가 불일치함
* ownership과 parent 관계가 혼동될 수 있음

`parent`가 반드시 메모리 ownership을 의미하지는 않는다.

---

# 15. Tree 파괴와 메모리 ownership

## 15.1 재귀적 파괴

```c
void tree_destroy_recursive(TreeNode *root)
{
    if (root == NULL) {
        return;
    }

    for (size_t i = 0; i < root->child_count; i++) {
        tree_destroy_recursive(root->children[i]);
    }

    free(root->children);
    root->children = NULL;

    free(root);
}
```

이 방식은 post-order 방식이다.

```text
모든 child 파괴
→ children 배열 해제
→ 현재 node 해제
```

현재 구조에서 ownership 정책은 다음과 같다.

> Parent는 `children` 배열에 연결된 모든 child subtree를 소유한다.

따라서 root를 파괴하면 전체 tree가 파괴된다.

---

## 15.2 호출 후 dangling pointer

```c
TreeNode *root = tree_node_create(10);

tree_destroy_recursive(root);

/* root는 여전히 해제된 주소를 보유한다. */
```

`root`는 dangling pointer가 된다.

호출자가 다음처럼 직접 처리해야 한다.

```c
tree_destroy_recursive(root);
root = NULL;
```

포인터 자체까지 `NULL`로 만드는 API를 만들 수도 있다.

```c
void tree_destroy(TreeNode **root_ptr)
{
    if (root_ptr == NULL || *root_ptr == NULL) {
        return;
    }

    TreeNode *root = *root_ptr;

    for (size_t i = 0; i < root->child_count; i++) {
        tree_destroy(&root->children[i]);
    }

    free(root->children);
    free(root);

    *root_ptr = NULL;
}
```

사용:

```c
tree_destroy(&root);
```

이제 호출 후:

```c
root == NULL
```

이 된다.

---

## 15.3 재귀 깊이 위험

Tree의 높이가 (H)이면 재귀 호출 stack depth는 최대 (O(H))다.

균형 잡힌 tree라면 보통:

[
H = O(\log N)
]

그러나 한쪽으로 길게 늘어진 tree라면:

[
H = O(N)
]

예:

```text
A
|
B
|
C
|
D
|
...
```

수십만 개 node가 연결되어 있다면 재귀적 파괴 중 stack overflow가 발생할 수 있다.

---

# 16. Iterative 파괴의 필요성

재귀를 피하려면 명시적인 stack을 사용할 수 있다.

다만 node를 안전하게 파괴하려면 child를 먼저 처리해야 하므로 post-order 처리가 필요하다.

한 가지 방법은 stack에 방문 여부를 함께 저장하는 것이다.

```c
typedef struct {
    TreeNode *node;
    int visited;
} StackEntry;
```

개념적 흐름:

```text
1. (root, not visited)를 stack에 넣는다.
2. 꺼낸 node가 미방문이라면:
   - (node, visited)를 다시 넣는다.
   - child들을 stack에 넣는다.
3. 방문 상태로 다시 나오면 node를 free한다.
```

장점:

* call stack overflow를 피한다.
* 사용자가 heap에 할당한 stack 크기를 통제할 수 있다.

단점:

* 구현이 길어진다.
* 별도 동적 메모리가 필요하다.
* stack allocation 실패를 처리해야 한다.
* 여전히 최악의 경우 (O(N)) 추가 공간이 필요할 수 있다.

즉, iteration이 자동으로 메모리를 적게 사용하는 것은 아니다.

차이는 주로 다음과 같다.

* recursion: 암시적 call stack
* iteration: 명시적 user-managed stack

---

# 17. 시간 복잡도와 공간 복잡도

## 17.1 전체 node 방문

Tree의 모든 node와 edge를 한 번씩 방문한다면:

[
O(V+E)
]

Tree에서는:

[
E=V-1
]

따라서:

[
O(V+E)=O(V+V-1)=O(V)
]

보통 node 수를 (N)이라 쓰므로:

[
\boxed{O(N)}
]

---

## 17.2 높이 계산

모든 node를 한 번씩 방문해야 하므로:

* 시간: (O(N))
* 재귀 공간: (O(H))

여기서 (H)는 tree 높이다.

| Tree 형태     |          높이 |       재귀 공간 |
| ----------- | ----------: | ----------: |
| 균형 tree     | (O(\log N)) | (O(\log N)) |
| Skewed tree |      (O(N)) |      (O(N)) |

---

## 17.3 Child 배열 추가의 amortized complexity

동적 배열의 capacity를 2배씩 늘리면 child 추가는:

* 대부분: (O(1))
* resize 발생 시: (O(k))
* amortized: (O(1))

여기서 (k)는 기존 child pointer 수다.

### 왜 amortized (O(1))인가

Capacity가 다음처럼 증가한다고 하자.

```text
2 → 4 → 8 → 16 → 32
```

전체 복사 비용은:

[
2+4+8+\cdots+N
]

기하급수 합이므로:

[
<2N
]

총 (N)번 삽입의 전체 비용이 (O(N))이고, 삽입당 평균 분할 비용은 (O(1))이다.

이는 확률적인 average case가 아니라 연산 시퀀스 전체에 대한 amortized bound다.

---

# 18. 정확성 증명 아이디어

Tree 알고리즘의 정확성은 대부분 tree의 재귀적 구조를 이용해 증명한다.

## 18.1 구조적 귀납법

명제:

> `tree_destroy_recursive(root)`는 root를 기준으로 한 subtree의 모든 동적 메모리를 정확히 한 번씩 해제한다.

### Base case

`root == NULL`이면 해제할 node가 없다.

함수는 즉시 반환한다.

### Inductive hypothesis

각 child의 subtree에 대해 함수가 모든 node를 정확히 한 번씩 해제한다고 가정한다.

### Inductive step

현재 root에 대해:

1. 각 child subtree를 재귀적으로 해제한다.
2. 귀납 가정에 따라 모든 descendant가 정확히 한 번씩 해제된다.
3. 현재 root의 child pointer 배열을 해제한다.
4. 현재 root를 해제한다.

Tree invariant에 의해 각 child는 정확히 하나의 parent를 가지므로, 서로 다른 child subtree는 겹치지 않는다.

따라서 같은 node가 두 번 해제되지 않는다.

---

## 18.2 Tree가 아니라 DAG라면 실패할 수 있다

다음 구조를 보자.

```text
    A
   / \
  B   C
   \ /
    D
```

B와 C가 같은 D를 가리키고 있다.

재귀적 destroy가 B subtree를 처리하며 D를 해제한 후, C subtree에서도 D를 다시 해제한다.

결과:

```text
double free
undefined behavior
```

따라서 재귀적 tree destroy의 정확성은 다음 invariant에 의존한다.

> 각 node는 root를 제외하면 정확히 하나의 parent를 가진다.

자료구조 invariant는 단순한 이론이 아니라 메모리 안전성과 직접 연결된다.

---

# 19. 실무·CS·시스템 관점

## 19.1 Pointer-based tree의 메모리 구조

일반적인 pointer-based tree는 node를 개별 `malloc`으로 생성한다.

```text
heap memory

0x1000: Node A
0x8F20: Node B
0x3010: Node C
0xD440: Node D
```

논리적으로 인접한 node가 물리 메모리에서는 멀리 떨어질 수 있다.

탐색 시:

```text
A pointer load
→ B 주소로 이동
→ B pointer load
→ D 주소로 이동
```

이것을 **pointer chasing**이라고 한다.

---

## 19.2 Cache locality

CPU는 메모리를 한 바이트씩 가져오는 것이 아니라 cache line 단위로 읽는다.

일반적인 cache line은 흔히 64바이트지만, 시스템에 따라 다르다.

Array 기반 구조:

```text
[node0][node1][node2][node3]
```

연속된 node를 접근하면 한 cache line에 여러 데이터가 함께 들어올 가능성이 높다.

Pointer 기반 구조:

```text
node0 → 임의 주소
node1 → 다른 임의 주소
node2 → 또 다른 주소
```

Cache miss가 더 자주 발생할 수 있다.

따라서 둘 다 이론적으로 (O(N)) traversal이어도 실제 실행 시간은 크게 다를 수 있다.

---

## 19.3 Allocation overhead

Node마다 `malloc`을 수행하면:

* allocator metadata 비용
* alignment padding
* fragmentation
* system allocator lock 또는 synchronization
* allocation failure 처리
* 해제 비용

등이 발생한다.

예를 들어 node payload가 작아도 실제 메모리 사용량은 더 클 수 있다.

```c
typedef struct Node {
    int value;           /* 4 bytes */
    struct Node *left;   /* 8 bytes */
    struct Node *right;  /* 8 bytes */
} Node;
```

64-bit 환경에서는 alignment 때문에 구조체가 24바이트가 될 수 있다.

여기에 allocator metadata가 추가될 수 있다.

정확한 크기는 ABI와 allocator 구현에 따라 다르다.

---

## 19.4 Memory pool과 arena

Embedded나 고성능 시스템에서는 node마다 일반 `malloc`을 호출하지 않고 pool을 사용할 수 있다.

```text
Node pool:
[node][node][node][node][node]
```

장점:

* allocation 시간이 예측 가능
* fragmentation 감소
* locality 개선
* allocation 실패 시점 통제 가능
* 여러 node를 한 번에 해제 가능

단점:

* 최대 node 수를 미리 정해야 할 수 있음
* 사용하지 않는 capacity가 낭비될 수 있음
* 개별 node 해제가 복잡하거나 불가능할 수 있음

---

## 19.5 Branch prediction

Tree 탐색은 데이터 값에 따라 왼쪽 또는 오른쪽으로 분기한다.

```c
if (key < node->key) {
    node = node->left;
} else {
    node = node->right;
}
```

입력 분포가 불규칙하면 CPU branch predictor가 방향을 잘 예측하지 못할 수 있다.

Branch misprediction은 pipeline flush를 유발할 수 있다.

따라서:

* 배열 binary search
* pointer-based BST search

둘 다 (O(\log N))일 수 있지만, 실제 성능은 다를 수 있다.

배열 binary search도 불규칙한 분기를 가지지만 pointer chasing이 없고 메모리가 더 조밀하다.

---

## 19.6 Embedded 시스템

Embedded 시스템에서는 다음 질문이 중요하다.

* 최대 node 수를 알 수 있는가?
* 동적 allocation을 허용하는가?
* 재귀를 허용하는가?
* stack 크기가 얼마인가?
* worst-case 실행 시간이 예측 가능한가?
* fragmentation을 감수할 수 있는가?
* tree 갱신이 실시간 제약을 만족하는가?

안전성이 중요한 시스템에서는 다음을 선호할 수 있다.

* 정적 배열
* 고정 크기 pool
* iterative traversal
* 명시적인 최대 깊이
* allocation 없는 초기화 이후 동작
* bounded execution time

Big-O가 좋더라도 최악 실행 시간이 예측 불가능하면 real-time 시스템에 부적합할 수 있다.

---

# 20. Array-based 구조와 Pointer-based 구조

| 관점               | Array-based     | Pointer-based      |
| ---------------- | --------------- | ------------------ |
| 메모리 배치           | 연속적             | 분산될 가능성            |
| Cache locality   | 대체로 좋음          | 대체로 낮음             |
| 개별 삽입·삭제         | 이동 또는 재할당 필요 가능 | pointer 수정으로 가능    |
| 구조 유연성           | index 규칙이 필요    | 임의 형태 표현 가능        |
| Allocation       | 한 번 또는 적은 횟수    | node마다 발생 가능       |
| Pointer overhead | 적음              | node마다 pointer 필요  |
| Serialization    | 비교적 쉬움          | 주소를 그대로 저장할 수 없음   |
| Fragmentation    | 낮은 편            | 높은 편               |
| Embedded 적합성     | 용량이 정해지면 유리     | 동적 구조에는 유리하나 관리 복잡 |

Heap은 complete binary tree이므로 array와 특히 잘 맞는다.

반면 일반 tree는 child 수와 형태가 불규칙해 pointer 표현이 자연스러운 경우가 많다.

---

# 21. 흔한 오해

## 오해 1: 모든 tree에는 root가 있다

수학적 무방향 tree에는 root가 필수적이지 않다.

자료구조에서는 대부분 root를 지정한 rooted tree를 사용한다.

---

## 오해 2: Tree는 항상 위에서 아래로 향하는 directed graph다

그림은 위에서 아래로 그리지만, 일반적인 tree의 기본 정의는 무방향 그래프다.

Root를 지정하면 parent-child 방향을 부여해 directed structure처럼 해석할 수 있다.

---

## 오해 3: Node 수가 `N`이면 간선 수가 `N-1`이므로 무조건 tree다

연결성이나 acyclic 조건을 추가로 확인해야 한다.

---

## 오해 4: Depth와 Height는 같은 개념이다

다르다.

* depth: root에서 현재 node까지
* height: 현재 node에서 가장 먼 leaf까지

---

## 오해 5: Path length는 node 개수다

Path length는 일반적으로 edge 개수다.

```text
A → B → C
```

* node 수: 3
* path length: 2

---

## 오해 6: Iterative 구현은 항상 `O(1)` 공간이다

명시적 stack이나 queue를 사용하면 여전히 (O(H)) 또는 (O(N)) 공간이 필요할 수 있다.

---

## 오해 7: Tree node를 `free(root)`만 하면 전체 tree가 해제된다

아니다.

`free`는 전달된 allocation 하나만 해제한다.

Child subtree를 먼저 해제하지 않으면 memory leak이 발생한다.

---

## 오해 8: Parent pointer가 있으면 parent가 child의 메모리를 소유한다

Parent 관계와 ownership은 별개의 개념이다.

API 문서에서 소유권 정책을 명시해야 한다.

---

# 22. Edge Cases

Tree 코드를 작성할 때 최소한 다음을 고려해야 한다.

## 빈 tree

```c
root == NULL
```

많은 함수에서 정상적인 입력으로 처리해야 한다.

예:

```text
empty tree node count = 0
empty tree traversal = 아무것도 출력하지 않음
```

Empty tree의 height는 정의가 다양하다.

* edge 기준: `-1`
* node 기준: `0`

반드시 정의를 명시해야 한다.

이번 강의에서는 non-empty tree에서 leaf height를 0으로 정의했다. 이 정의를 확장하면 empty tree height를 `-1`로 두는 것이 재귀식에 자연스럽다.

---

## Root 하나만 존재

```text
A
```

```text
node count = 1
edge count = 0
depth(A) = 0
height(A) = 0
A는 root이면서 leaf
```

Root가 leaf가 될 수 있다는 점을 기억해야 한다.

---

## 매우 깊은 tree

```text
A
|
B
|
C
|
...
```

재귀로 처리하면 stack overflow가 발생할 수 있다.

---

## 잘못된 cycle

```text
A → B → C
↑       │
└───────┘
```

재귀 traversal이 무한 재귀에 빠진다.

Tree invariant를 외부 입력이 깨뜨릴 가능성이 있다면 visited set 또는 cycle 검증이 필요하다.

---

## Shared child

두 parent가 같은 child를 소유하면:

* double free
* ownership 충돌
* subtree 정의 붕괴

가 발생할 수 있다.

---

# 23. 확인 문제

## 문제 1

다음 그래프는 tree인가?

```text
A — B — C — D
```

<details>
<summary>정답</summary>

Tree다.

* connected
* acyclic
* 정점 4개
* 간선 3개
* 임의의 두 정점 사이 경로가 하나

</details>

---

## 문제 2

다음 그래프는 tree인가?

```text
A — B
|   |
D — C
```

<details>
<summary>정답</summary>

아니다. Cycle `A-B-C-D-A`가 존재한다.

</details>

---

## 문제 3

다음 그래프는 tree인가?

```text
A — B     C — D
```

<details>
<summary>정답</summary>

아니다. Acyclic이지만 disconnected다. 전체 구조는 forest다.

</details>

---

## 문제 4

다음 rooted tree에서 F의 depth와 height를 구하라.

```text
        A
       / \
      B   C
     /
    D
   /
  F
```

<details>
<summary>정답</summary>

* `depth(F) = 3`
* `height(F) = 0`

F는 leaf다.

</details>

---

## 문제 5

정점 12개로 이루어진 tree의 간선 수는?

<details>
<summary>정답</summary>

[
12-1=11
]

</details>

---

## 문제 6

정점 12개, connected component 3개인 forest의 간선 수는?

<details>
<summary>정답</summary>

[
12-3=9
]

</details>

---

## 문제 7

“Graph의 정점 수가 (N)이고 간선 수가 (N-1)이므로 tree다.”

이 주장이 잘못된 이유는?

<details>
<summary>정답</summary>

간선 수만으로는 connected와 acyclic을 보장할 수 없다. 한 component에는 cycle이 있고 다른 정점은 분리된 graph일 수 있다.

</details>

---

## 문제 8

Rooted tree에서 root를 제외한 모든 node가 parent를 정확히 하나 가져야 하는 이유는?

<details>
<summary>정답</summary>

Parent가 없으면 root로부터 도달할 수 없는 disconnected node가 될 수 있다. Parent가 둘 이상이면 root에서 해당 node로 가는 경로가 여러 개 생기며, 무방향으로 보면 cycle이 생길 수 있다.

</details>

---

# 24. 실습 과제

## 실습 1: 용어 계산

다음 tree를 직접 종이에 그리고 모든 node에 대해 다음 값을 작성하라.

```text
        10
      /  |  \
     20  30  40
    /       /  \
   50      60  70
             \
              80
```

각 node별로:

* parent
* child 목록
* sibling 목록
* depth
* height
* leaf 여부
* internal node 여부

---

## 실습 2: Node count 구현

다음 함수의 요구사항을 만족하도록 작성하라.

```c
size_t tree_count_nodes(const TreeNode *root);
```

조건:

* `root == NULL`이면 0
* 모든 node를 정확히 한 번 방문
* 시간 복잡도 분석
* 재귀 depth 분석
* `size_t` overflow 가능성 고려

기본 재귀식:

[
\operatorname{count}(v)
=======================

1+\sum_{c \in children(v)}\operatorname{count}(c)
]

---

## 실습 3: Height 구현

```c
int tree_height(const TreeNode *root);
```

정의:

```text
empty tree height = -1
leaf height = 0
```

재귀식:

[
H(v)
====

1+\max_{c \in children(v)}H(c)
]

단, leaf에는 child가 없으므로 결과가 0이 되어야 한다.

---

## 실습 4: Invariant 검사 설계

Parent pointer가 포함된 tree에 대해 다음을 검사하는 함수를 설계하라.

```c
int tree_validate(const TreeNode *root);
```

검사 대상:

* `root->parent == NULL`
* 각 child의 `parent`가 현재 node인가
* 같은 child pointer가 중복 등록되지 않았는가
* cycle이 존재하지 않는가
* 한 node가 여러 parent 아래에 나타나지 않는가

외부에서 손상된 구조를 검사하려면 visited set이 필요할 수 있다.

---

## 실습 5: Allocation 실패 시나리오

다음 상황을 분석하라.

1. Parent 생성 성공
2. Child 생성 성공
3. `tree_node_add_child()` 내부 `realloc` 실패
4. 호출자가 child를 해제하지 않음

어떤 memory leak이 발생하는가?

호출자는 실패 시 다음과 같이 처리해야 한다.

```c
TreeNode *child = tree_node_create(20);

if (child == NULL) {
    /* allocation failure */
}

if (!tree_node_add_child(parent, child)) {
    tree_destroy(&child);
    /* insertion failure */
}
```

---

# 25. 핵심 정리

## Tree의 본질

[
\boxed{\text{Tree = connected acyclic undirected graph}}
]

Tree는 다음 성질들을 가진다.

* 정점 (N)개이면 간선 (N-1)개
* 임의의 두 정점 사이 simple path가 정확히 하나
* 간선 하나를 제거하면 disconnected
* 새로운 간선 하나를 추가하면 cycle 발생

---

## Rooted tree의 본질

Root가 정해지면 계층 관계가 생긴다.

* root는 parent가 없다.
* 나머지 node는 parent가 정확히 하나다.
* node는 0개 이상의 child를 가진다.
* subtree는 동일한 구조를 재귀적으로 가진다.

---

## Depth와 Height

[
\operatorname{depth}(v)
=======================

\operatorname{distance}(\text{root},v)
]

[
\operatorname{height}(v)
========================

\max_{\ell\text{ leaf}}
\operatorname{distance}(v,\ell)
]

* Depth는 root로부터 내려온 거리
* Height는 leaf까지 내려갈 수 있는 최대 거리

---

## 시스템 관점

Pointer-based tree는 유연하지만 다음 비용이 있다.

* pointer chasing
* cache miss
* 개별 allocation overhead
* fragmentation
* ownership 복잡성
* 재귀 stack overflow 위험

Array-based 구조는 locality와 메모리 효율이 좋지만, 불규칙한 tree 표현에는 제약이 있다.

---

# 26. 면접 대비 핵심 문장

### Tree 정의

> Tree는 connected하면서 cycle이 없는 무방향 graph입니다. 동치적으로, 임의의 두 정점 사이에 simple path가 정확히 하나 존재하는 graph라고 설명할 수 있습니다.

### 간선 수

> 정점이 (N)개인 tree는 정확히 (N-1)개의 간선을 가집니다. 새로운 정점을 연결 상태로 추가하면서 cycle을 만들지 않으려면 기존 tree와 정확히 하나의 간선으로 연결해야 하기 때문입니다.

### Rooted tree

> 수학적 tree는 본래 root가 없는 무방향 graph입니다. 특정 정점을 root로 지정하면 parent, child, depth, subtree 같은 계층적 개념이 정의됩니다.

### Depth와 Height

> Depth는 root에서 현재 node까지의 간선 수이고, height는 현재 node에서 가장 먼 leaf까지의 간선 수입니다. Root의 depth와 leaf의 height는 일반적으로 0입니다.

### Tree와 Graph 차이

> 모든 tree는 graph이지만 모든 graph가 tree는 아닙니다. Graph는 cycle, disconnected component, 여러 경로를 가질 수 있지만 tree는 connected와 acyclic이라는 제약을 가집니다.

### 복잡도

> Tree 전체 traversal은 (O(V+E))이고 tree에서는 (E=V-1)이므로 (O(N))입니다. 재귀적 traversal의 추가 공간은 node 수가 아니라 tree height (H)에 따라 (O(H))입니다.

### 실제 성능

> Pointer-based tree는 이론적인 복잡도가 좋아도 pointer chasing, cache miss, allocation overhead, branch misprediction 때문에 연속 배열 기반 구조보다 실제 성능이 낮을 수 있습니다.

### 메모리 안전성

> Tree 파괴 함수의 정확성은 각 node가 하나의 parent만 갖고 subtree가 겹치지 않는다는 invariant에 의존합니다. Shared child가 존재하면 단순 재귀 파괴에서 double free가 발생할 수 있습니다.
