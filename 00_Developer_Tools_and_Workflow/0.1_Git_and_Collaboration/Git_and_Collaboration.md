# 1. Git & Collaboration

## Core Objects

먼저 Git은 명령어를 외우는 도구가 아니다.
프로젝트의 스냅샷과 그 스냅샷 사이의 관계를 저장하는 그래프 데이터베이스라고 생각하길 바란다.

이제 거기서 우리가 확실히 Git을 사용할 줄 안다고 말하기 위해서는 다음을 이야기할 수 있어야한다.

1. Git이 내부적으로 무엇을 저장하는가?
2. 커밋들이 어떤 그래프 구조를 이루는가?
3. branch,HEAD,tag가 실제로 무엇을 가르키는가?
4. merge 와 rebase가 그래프를 어떻게 바꾸는가?
5. 협업 중 실수를 했을 때 왜 revert, reset, reflog를 구분해서 사용해야 하는가?

### Git Mental Model

자 먼저, Git 을 이해하기 위해서는 세 공간을 알아야 한다.

Working Directory  →  Staging Area  →  Repository
     작업 공간             대기실              저장소

쉽게 이야기하면

Working Directory = 내가 실제로 파일을 수정하는 곳
Staging Area      = 다음 커밋에 넣을 변경사항을 고르는 곳
Repository        = 커밋들이 영구 저장되는 곳

이런 식으로도 이해할 수 있다.

예를 들어 main.cpp를 수정했다고 치자.
이 시점에서는 main.cpp는 Working Directory 에만 있다. 그렇기 때문에 깃은 '수정된 것은 보이는데, 다음 커밋에 넣겠다고 확정한 것은 아니다.' 이렇게 이야기한다.

그 이후에 
git add main.cpp

라는 명령어를 사용하였을 때, 변경사항이 Staging Area에 올라가게 된다.

또 그 이후에

git commit -m "Implement socket set up"

이라고 하는 명령어를 사용하게 되면 Staging Area에 있던 내용이 Repository에 commit으로 저장되게 된다.

#### git status는 세 공간의 차이를 보여준다.

git status의 경우 정확하게 세 공간을 비교한다.

Working Directory vs Staging Area
Staging Area vs HEAD commit

즉, git status는 내부적으로 이런 질문을 한다.
1. 현재 작업 파일이 staging area와 다른가?
2. staging area가 마지막 commit과 다른가?
3. 현재 branch는 remote branch와 차이가 있는가?

다시 예를 들어, 

Changes not staged for commit

이런 말이 나오게 되면 'Working Directory에는 변경이 있는데, Staging Area에는 아직 반영되지 않았다.' 라는 뜻이다.
그리고, 

Changes to be committed

이는 'Staging Area에는 올라갔고, 다음 commit에 들어갈 준비가 되었다.' 는 뜻이다.

#### Commit '변경사항'이 아니라 '스냅샷'이다.

많은 사람들은 commit을 변경사항 저장이라고 생각한다. 이것은 반쯤 맞는 말이지만 한번 파고들어서 생각을 해보면, 기본적으로 프로젝트 전체의 스냅샷이라고 생각할 수 있다.

예를 들어

project/
├── main.cpp
├── server.cpp
└── user.cpp

이런 구조의 파일이 있다고 해보자.

여기서 A이라는 커밋이 있다.
A = main.cpp, server.cpp, user.cpp의 특정 상태

그다음 server.cpp만 수정해서 커밋 B를 만든다.

B = main.cpp, server.cpp, user.cpp의 새로운 전체 상태

이러한 상황에서 Git은 내부적으로 효율성을 위해 동일한 파일 내용은 재사용하지만, 개념적으로 commit은 전체 프로젝트의 스냅샷이다.
이 관점이 매우 중요하다. 

왜냐하면 이후에 나올 branch, merge,rebase를 이해할 때, commit을 "패치 조각"으로 보면 헷갈리고, "스냅샷 노드"로 보면 훨씬 명확해보이기 때문이다.

#### Commit은 그래프를 이룬다.
Git commit들은 일렬로만 존재하지 않는다.
커밋은 부모 commit을 가리킨다.

예를 들어,
A --- B --- C

B의 parent = A
C의 parent = B

이러한 형태의 커밋이 있다고 치자.
그러면 commit은 이러한 정보를 가진다.

commit C:
  tree: 프로젝트 스냅샷
  parent: B
  author: 작성자
  message: 커밋 메시지

그래서 Git history는 사실상 그래프다.

정확히는 DAG.
Directed Acyclic Graph
방향성 있고, 순환 없는 그래프이다.

#### Branch는 커밋을 가리키는 포인터이다.
또 많은 사람들은 Branch를 "작업 공간의 복사본" 처럼 생각한다.
하지만 Git에서 branch는 매우 가볍다.

사실 branch는 그냥 특정 commit을 가리키는 이름표일 뿐이다.

예를 들어,

A --- B --- C
          ↑
        main

이러한 커밋이 있다고 치자.
여기서 main은 C 커밋을 가리키는 포인터이다.

이 상태에서 새 브랜치를 만든다.
git branch feature

그러면 

A --- B --- C
          ↑
        main
          ↑
       feature

이러한 형태가 되는 것을 알 수 있다. 
여기서 중요한 점은 아직 커밋이 새로 생긴 게 아니라는 것이다.
그냥 feature라는 이름표가 C를 가리키게 된 것이다.

#### HEAD는 "내가 현재 보고 있는 위치"다.

HEAD는 현재 체크아웃된 위치를 가리킨다.
보통은 branch를 가리킨다.

A --- B --- C
          ↑
        main
          ↑
        HEAD

정확하게 말하면, HEAD → main → C 이렇게 되어져 있다.
즉 HEAD가 직접 C를 가리키는 게 아니라, 보통은 branch 이름을 가리킨다.

이 상태에서 커밋을 하나 더 하면

git commit -m "Add parser"

그 결과는

A --- B --- C --- D
                ↑
              main
                ↑
              HEAD

commit을 하면 HEAD가 가리키는 branch가 앞으로 이동한다는 것을 알 수 있다.
즉, main 포인터가 C에서 D로 이동한다.

#### Branch를 이동하면 HEAD가 바뀐다.

A --- B --- C
          ↑
        main
          ↑
       feature
          ↑
        HEAD

현재 HEAD가 feature에 있다고 하자. 이 상태에서

git checkout main 또는 git switch main

를 사용하여 branch를 바꿔다고 해보자.

A --- B --- C
          ↑
        main
          ↑
        HEAD

feature도 C를 가리키고 있음

그러면 결국 HEAD는 feature에서 main으로 이동한다.
파일 내용도 main이 가리키는 커밋 상태로 바뀌게 된다.

#### 새 브랜치에서 커밋하면 그래프가 갈라진다

A --- B --- C
          ↑
        main
          ↑
       feature

git switch feature로 feature로 이동하고, 그리고 git commit -m "Implement channel broadcast" 커밋을 했을 때 

A --- B --- C --- D
          ↑       ↑
        main   feature
                  ↑
                HEAD

여기서 main은 아직 C에 있다.
feature만 D로 이동한다.

이게 branch의 본질이다.

* branch는 독립된 폴더 복사본이 아니라, 서로 다른 commit을 가리키는 이름표다.

#### 중요한 명령어를 내부 동작으로 보기

1. git add

git add main.cpp

Working Directory의 main.cpp 상태를 Staging Area에 반영한다.

2. git commit

git commit -m "Implement server socket"

Staging Area의 상태를 기준으로 새 commit object를 만든다.
현재 branch 포인터를 새 commit으로 이동시킨다.

3. git branch feature

git branch feature

현재 commit을 가리키는 feature라는 ref를 만든다.
HEAD는 이동하지 않는다.

git branch feature는 브랜치를 만들기만 한다. 이동하는 것은 따로 이동해주어야 함.

4. git switch feature

git switch feature

HEAD가 feature branch를 가리키게 한다.
Working Directory를 feature가 가리키는 commit 상태로 바꾼다.

git switch -c feature

현재 commit에서 feature branch를 만들고,
HEAD를 feature로 이동시킨다.

### Core Objects

* Git은 파일을 저장하는 것이 아니라, 객체 object를 저장한다.

blob   = 파일 내용
tree   = 디렉토리 구조
commit = 스냅샷 + 부모 커밋 + 메타데이터
tag    = 특정 커밋에 붙이는 고정 이름표

그리고 객체는 아니지만 매우 중요한 것이 있다.

ref    = 커밋을 가리키는 포인터
HEAD   = 현재 위치
branch = ref의 한 종류

#### Blob: 파일 내용

blob은 파일의 내용을 의미한다.

echo "hello" > a.txt

이 파일을 Git에 추가하면 Git은  "hello"라는 내용을 객체로 저장한다.
이 객체가 바로 blob이다.

개념적으로는 

a.txt
내용: hello

Git 내부에서는

blob object
내용: hello

즉, 파일 이름 a.txt는 blob에 저장되지 않는다.
파일 이름은 나중에 tree가 관리한다.

예를 들어보자

mkdir git-object-lab
cd git-object-lab
git init

echo "hello git" > hello.txt
git add hello.txt

이제 hello.txt의 내용은 Git object database에 들어간다.

git hash-object hello.txt

이라는 명령어를 사용하면, 

5f8f... 이런식으로 해쉬값이 출력이 된다. 이 해시는 파일 내용의 ID이다. 
더 정확히 말하면 Git은 파일 내용을 그대로 해시하지 않고, 내부적으로 이런 형태를 해시한다.

blob <크기>\0<내용>

예를 들어 hello라는 내용을 넣었다고 하면?

blob 10\0hello git\n

이 전체를 SHA 해시로 계산한다.

#### git cat-file로 객체 보기

Git 내부 객체를 볼 때는 이 명령어가 자주 쓰인다.

git cat-file -t <hash>

이 객체의 타입을 보여주라는 뜻이다.

git cat-file -t 5f8f...

이러한 해쉬를 뒤에 집어넣으면,

blob

이렇게 그 타입을 알려준다.
그리고 -p 옵션을 사용할 경우

git cat-file -p 5f8f...

hello git
이러한 출력을 보여준다.

정리하면

git cat-file -t <hash>  # 객체 타입 확인
git cat-file -p <hash>  # 객체 내용 출력

#### Tree: 디렉토리 구조

blob은 파일 내용만 알고있다. 그렇다면 파일 이름은 어디에 저장될까
바로 tree이다. tree는 보통

파일 이름
파일 모드
해당 파일 내용 blob의 hash
하위 디렉토리 tree의 hash

이러한 정보를 저장하고 있다.

예를 들어

project/
├── main.cpp
└── server.cpp

이러한 프로젝트 구조가 있다고 했을 때, Git은 대략 이렇게 저장한다.

tree object
├── main.cpp   -> blob hash 1111...
└── server.cpp -> blob hash 2222...

즉,
blob = 파일 내용
tree = 파일 이름과 디렉토리 구조 라는 것이다.

#### Commit: 스냅샷 + 부모 정보

commit은 프로젝트 전체 상태를 나타낸다.

commit object는 대략 이런 정보를 가진다.

tree <tree-hash>
parent <parent-commit-hash>
author <author>
committer <committer>
message

예를 들어

commit C
├── tree: 프로젝트 루트 디렉토리의 tree hash
├── parent: B
├── author: 현준
├── message: Implement parser

구조는 이런식으로 만들어 진다.
여기서 중요한 점은 commit은 파일을 직접 들고 있는 게 아니라, 루트 tree를 가리킨다.

즉 전체적인 구조는

commit
  ↓
root tree
  ↓
file blobs / subdirectory trees

이러하다.

#### 실제 commit 내부 보기

그럼 돌아와서 실제 commit의 내부를 둘러보자.

git commit -m "Initial commit"

이렇게 커밋을 시켰을 때,

git rev-parse HEAD

이러한 명령어를 통해서 현재 커밋 해시를 확인할 수 있다.
그때 예상출력은

a1b2c3...

이런식일 것이다.

이제 커밋 객체를 살펴본다

git cat-file -t HEAD

출력은

commit

이렇게 나올 것이며, 객체 속을 보기 위해 -p 옵션을 넣으면?

git cat-file -p HEAD

tree 9f3a...
author Your Name <you@example.com> 1710000000 +0900
committer Your Name <you@example.com> 1710000000 +0900

Initial commit

이런식으로 나올 것이다. 물론 첫 커밋이라면 첫 커밋이면 parent 줄이 없다. 왜냐하면 부모 커밋이 없기 때문.

#### Branch는 commit object가 아니다

정말 정말 중요한 부분인데, main 브랜치는 commit object가 아니다.
main은 그냥 파일일 뿐이다.

실제로 Git repo 안에는 이런 파일이 있다.

cat .git/refs/heads/main

출력은 대략 abc123... 이러한 해쉬값이 나올 것이다. 그리고 이 해시가 현재 main이 가리키는 commit이다.

즉, 

main branch = .git/refs/heads/main 파일
그 안의 내용 = commit hash

브랜치가 포인터라는 말은 여기서부터 나온다.

#### HEAD 또한 그저 파일이다. 

cat .git/HEAD

이러면 보통 ref: refs/heads/main 이런식으로 나온다.

HEAD는 main 브랜치를 가리킨다.
main 브랜치는 어떤 commit hash를 가리킨다.

즉, HEAD -> refs/heads/main -> commit hash

실제로는 이런식으로 저장되어져 있다는 뜻이다.

HEAD
 ↓
.git/refs/heads/main
 ↓
commit object
 ↓
tree object
 ↓
blob object

#### 브랜치를 만들면?

git branch feature

이 명령어를 사용해서 feature 브랜치를 만들면 Git 내부에는 .git/refs/heads/feature 이런식의 파일이 생긴다.그 파일 안에 commit hash가 들어가는데 그 파일안을 확인해보면?

A --- B
      ↑
    main
      ↑
    HEAD

이 구조 에서는

main    -> B
feature -> B

이렇게 되어져 있다.
그러나 git switch feature는 HEAD를 바꾼다.

git switch feature

이후 cat .git/HEAD를 확인해보면?
ref: refs/heads/feature

즉, HEAD가 바뀐 것이다.
HEAD -> feature

하지만 그렇다고 아직 커밋이 새로 생긴 것은 아니다.

#### feature에서 커밋하면 feature 파일이 바뀐다

다시 돌아와서 상황을 이어나가보자.

HEAD -> feature -> B
main -> B

이렇게 되어져 있다고 하자.

echo "feature work" > feature.txt
git add feature.txt
git commit -m "Add feature work"

이런식으로 파일을 수정하고 커밋 했을 시, 
cat .git/refs/heads/feature 를 살펴보면 새 커밋 해시가 나온다.
반면 cat .git/refs/heads/main 를 하면 아직 이전 커밋 해시를 가지고있다.

즉 여기서 분기점이 갈리게 되는 것이다.

main    -> B
feature -> C
HEAD    -> feature

A --- B
      ↑
    main
       \
        C
        ↑
     feature
        ↑
      HEAD

#### Tag: 움직이지 않는 이름표

branch는 보통 계속 움직인다. 커밋을 만들면 현재 branch가 앞으로 간다.

하지만 tag는 보통 움직이지 않는다.

git tag v1.0

이런식으로 현재 커밋에 tag를 붙이면 

A --- B --- C
          ↑
        main
          ↑
        v1.0
  
이후 main에 새 커밋을 할 시, 

A --- B --- C --- D
          ↑       ↑
        v1.0    main

main은 D로 이동했지만, v1.0은 C에 남아 있다.
그래서 주로 tag는 릴리즈 지점에 씁니다.

##### Branch와 Tag 차이?

branch = 움직이는 포인터
tag    = 고정된 포인터

main = 현재 개발 최신 지점
v1.0 = 1.0 릴리즈 당시의 고정 지점

git tag v1.0.0
git push origin v1.0.0

이렇게 볼 수 있는 것이다.

#### Ref란 무엇인가?

ref는 reference의 줄임말이다. 
쉽게 이야기해서
* 커밋 해시를 사람이 읽을 수 있는 이름으로 가리키는 것

refs/heads/main       = 로컬 브랜치
refs/heads/feature    = 로컬 브랜치
refs/tags/v1.0        = 태그
refs/remotes/origin/main = 원격 추적 브랜치

그래서 Git에서 이런 이름들은 결국 commit hash를 가리킨다.
git rev-parse main
git rev-parse HEAD
git rev-parse v1.0

이 명령어들은 해당 이름이 실제로 어떤 해시를 가리키는지 보여준다.

여기서 헷갈릴 수 있는데, 브랜치는 추상적인 마법이 아니라, 커밋 해시가 적힌 ref다.

#### 그럼 전부 포인터같은데 실제로 어디에 저장이 될까요?

* 실제 파일 내용은 .git/objects/ 안에 저장된다.

예를 들어.

echo "hello" > a.txt
git add a.txt

라고 했을 때, Git은 a.txt의 내용을 읽는다.
hello

그리고 이 내용을 Git object database에 저장한다.
대략 위치는
  .git/objects/해시앞2글자/나머지해시

그럼 여기서 blob hash가 이렇다고 치자.
ce013625030ba8dba906f756967f9e9ca394464a

그러면 실제 저장 위치는 대략
  .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a
가 되는 것이다. 

즉 요약하면

파일 내용 "hello"
        ↓
blob object 생성
        ↓
.git/objects/ce/013625...

* 그런데 여기서 그냥 텍스트로 저장되는 건 아니다.

.git/objects/ 안에 들어가서 파일을 직접 열어보면 사람이 읽을 수 있는 텍스트가 아니다.

Git은 객체를 저장할 때 대략 이렇게 한다.

원본 내용
↓
Git object header 추가
↓
SHA 해시 계산
↓
압축
↓
.git/objects/ 안에 저장

예를 들어 a.txt 내용이:
  hello
라면 Git은 단순히 hello만 저장하지 않는다.
내부적으로는 이런 데이터를 만든다.

blob 6\0hello\n

blob = 객체 타입
6    = 내용 크기
\0   = header와 content를 나누는 null byte
hello\n = 실제 파일 내용

그리고 이것을 해시로 식별하고 압축해서 .git/objects/에 저장하게 되는 것이다. 앞서 말한 내용과 거의 같지만, 헷갈리기에 한번 더 정리했다.
하지만 여기서도 한가지 재미있는 점이 있는데,

* 같은 내용의 파일은 같은 blob을 공유한다.

echo "hello" > a.txt
echo "hello" > b.txt
git add a.txt b.txt

이렇다고 할 때, 두 파일의 내용이 완전히 같으면 Git은 같은 blob을 사용한다.
a.txt -> blob ce013625...
b.txt -> blob ce013625...

왜냐하면 blob hash는 파일 이름이 아니라 파일 내용으로 결정되기 때문이다.

파일 이름이 달라도 내용이 같으면 같은 blob
파일 이름이 같아도 내용이 바뀌면 다른 blob

#### 그렇다면 Git은 변경분만 저장할까, 전체 파일을 저장할까?
개념적으로는 Git은 스냅샷 시스템이다.

각 커밋은 전체 프로젝트 상태를 나타낸다. 하지만 실제 저장은 효율적으로 한다.

처음에는 파일 내용을 blob으로 저장한다. 그리고 나중에 Git이 packfile이라는 형태로 압축할 때, 내부적으로 유사한 객체들을 묶고 델타 압축을 할 수 있다.
스냅샷 기반이라는 개념적 특징을 가지고 있지만, 실제 저장의 최적화는 압축 + 중복 제거 + packfile을 사용한다고 볼 수 있다.

### Git의 압축방식

Git의 압축 방식은 크게 두 단계로 이해할 수 있다.

1단계: loose object 저장
2단계: packfile 압축

처음 git add, git commit을 할 때는 객체들이 보통 loose object 형태로 저장된다.
나중에 Git이 정리할 때는 여러 객체를 묶어서 packfile로 압축하게 된다.

#### 먼저 loose object란?

echo "hello" > a.txt
git add a.txt

이때 Git은 a.txt의 내용을 blob object로 저장한다.
대력적으로 .git/objects/ce/013625030ba8dba906f756967f9e9ca394464a 이러한 위치에 있을 것이다.

이런 식으로 .git/objects/ 안에 객체 하나하나가 따로 저장된 상태를 loose object라고 한다.

.git/objects/
├── ce/013625...
├── 4b/825dc...
└── ab/c123...

각 파일은 Git object 하나이다.
이때 Git은 객체를 그냥 원문 그대로 저장하지 않는다.

원본 내용
↓
Git object header 추가
↓
zlib 압축
↓
.git/objects/ 안에 저장

#### loose object의 한계

loose object는 이해하기 쉽고 빠르게 만들 수 있다. 하지만 문제가 있다. 

예를 들어 큰 파일을 조금씩 수정하면서 여러 번 커밋했다고 하자.

commit A: report.txt 1000줄
commit B: report.txt 1001줄
commit C: report.txt 1002줄
commit D: report.txt 1003줄

개념적으로 Git은 각 버전의 파일 내용을 blob으로 저장한다.

A의 report.txt -> blob A
B의 report.txt -> blob B
C의 report.txt -> blob C
D의 report.txt -> blob D

문제는 이 blob들이 대부분 비슷하다는 것이다.

blob A: 거의 1000줄
blob B: blob A와 거의 같고 1줄만 다름
blob C: blob B와 거의 같고 1줄만 다름
blob D: blob C와 거의 같고 1줄만 다름

이걸 매번 독립 객체로 압축해서 저장하면 공간이 낭비가 되기 때문에, 그래서 Git은 나중에 객체들을 묶어서 더 효율적으로 저장한다. 그게 packfile이다.

#### packfile

packfile은 여러 Git object를 하나의 큰 파일에 모아서 저장한 것이다. 

보통 위치는 

.git/objects/pack/

예시

.git/objects/pack/
├── pack-a1b2c3.pack
└── pack-a1b2c3.idx

.pack = 실제 압축된 객체 데이터
.idx  = packfile 안에서 특정 객체를 빨리 찾기 위한 인덱스

loose object
= 객체 하나당 파일 하나

packfile
= 여러 객체를 하나의 pack 파일에 묶음

#### 그렇다면 Git은 언제 packfile을 만들까?

git gc 또는 git repack 또는 원격 저장소와 통신할 때, 
git clone
git fetch
git push

Git은 네트워크로 객체를 주고받을 때도 pack 형태로 묶어서 효율적으로 전송한다.
즉, Git은 처음에는 객체를 loose하게 저장하다가, 시간이 지나면 packfile로 정리한다.

#### Git 압축의 핵심: delta compression

Git packfile의 핵심은 delta compression이다.

* 비슷한 객체를 여러 개 저장할 때, 하나는 온전하게 저장하고 나머지는 차이점만 저장한다.

예를 들어 report.txt가 이렇게 바뀌었다고 하자.

line 1
line 2
line 3
line 4

line 1
line 2
line 3 changed
line 4

그러면 Git 은 이 둘을 따로 저장하는 대신, 대략 이런 식으로 저장할 수 있다.

blob A = 전체 내용 저장
blob B = blob A를 기준으로 "3번째 줄이 바뀌었다"만 저장

즉, 
A 전체 저장
B는 A와의 차이만 저장

하지만 여기서 가장 중요한 점은 Git은 커밋을 diff로 저장하는 게 아니다라는 것이다.
Git의 개념 모델은 여전히 commit = 전체 프로젝트 스냅샷 이고, 저장 최적화 단계에서는 비슷한 객체끼리 delta로 압축이라고 할 수 있다.
논리 모델: 스냅샷
물리 저장: 압축 + delta

### Git LFS

Git LFS는 Large File Storage이다.

개념은 단순하다.
일반 Git에는 진짜 큰 파일을 직접 넣지 않는다. 대신 Git에는 작은 포인터 파일만 저장한다.
Git repo
└── model.bin pointer

LFS storage
└── 실제 model.bin

Git은 큰 파일의 주소만 관리
실제 큰 파일은 별도 LFS 서버가 관리

### Daily Commands

Working Directory  →  Staging Area(Index)  →  Repository
     작업 파일              커밋 대기실             커밋 저장소

Git의 핵심은 이 세가지 공간을 이해하는 것이다.

Working Directory
= 실제 파일을 수정하는 곳

Staging Area / Index
= 다음 커밋에 들어갈 내용을 고르는 곳

Repository
= commit object들이 저장되는 곳

다시 말해서

Working Directory
      |
      | git add
      v
Staging Area / Index
      |
      | git commit
      v
Repository

#### Git status

git status는 무엇을 보여주는가?

git status는 단순하게 현재 상태를 보여주는 명령어가 아니다.

1. HEAD commit vs Staging Area
2. Staging Area vs Working Directory
3. Local branch vs Remote-tracking branch

즉, 이 내부적으로 이러한 질문들을 한다
Staging Area가 HEAD와 다른가?
Working Directory가 Staging Area와 다른가?
현재 브랜치가 원격 추적 브랜치보다 앞서거나 뒤처졌는가?

#### Git add

git add a.txt

Working Directory의 현재 파일 내용을 Staging Area에 복사한다.

HEAD commit
└── a.txt = "hello"

Staging Area
└── a.txt = "hello world"

Working Directory
└── a.txt = "hello world"

이때 git status 를 하면

Changes to be committed:
  modified: a.txt

Staging Area가 HEAD와 다르다.
이 차이가 다음 커밋에 들어갈 예정이다.

Working Directory = Staging Area
Staging Area ≠ HEAD

#### git commit

git commit -m "Update a"

를 실행하면 Git은 Staging Area를 기준으로 새 commit object를 만든다.

Before commit:

HEAD commit
└── a.txt = "hello"

Staging Area
└── a.txt = "hello world"

Working Directory
└── a.txt = "hello world"

after

New HEAD commit
└── a.txt = "hello world"

Staging Area
└── a.txt = "hello world"

Working Directory
└── a.txt = "hello world"

즉, HEAD = Staging Area = Working Directory

* git commit -a

git commit -a -m "Update tracked files"

이 명령은 tracked file의 수정 사항을 자동으로 stage하고 commit한다.
하지만 한가지 주의해야할 점이 있다.

새 파일 untracked file은 자동으로 추가하지 않는다.
예를들어, 

echo "new" > new.txt
git commit -a -m "Add new"

이렇게 되면 new.txt는 커밋되지 않는다.
왜냐하면 Git이 아직 추적하지 않는 파일이기 때문이다.
새 파일은 반드시 git add를 통해 처음에 staging area에 올려줘야한다.


#### git diff

이건 그리 어렵지 않다.

Working Directory와 Staging Area의 차이를 보여줘.

아직 git add하지 않은 변경을 보여준다.

Working Directory와 Staging Area가 같아지면 변경이 없다고 나온다.

* git diff --staged

git diff --staged 또는 git diff --cached

이는 Staging Area와 HEAD commit의 차이를 보여주라고 하는 것과 같다.

** 정리 **
git diff
= Working Directory vs Staging Area

git diff --staged
= Staging Area vs HEAD

#### git push

A --- B --- C
          ↑
        main

현재 이렇게 로컬이 형성되어져 있다고 하고,

원격 origin/main은 아직 B에 있다고 가정하자.

A --- B
      ↑
 origin/main

그러면

 A --- B --- C
      ↑     ↑
origin/main main

이때

git push origin main

push를 하게되면 로컬 main의 커밋 C를 원격으로 보낸다.
그 결과,

A --- B --- C
          ↑
        main
          ↑
    origin/main

정확히 말하면 로컬의 remote-tracking ref인 origin/main도 fetch/push 이후 업데이트되어 C를 가리키게 된다.

#### origin/main은 무엇인가?

origin/main는 내 로컬 Git이 마지막으로 확인한 origin 서버의 main 상태를 말한다. 
다시 말해서

main        = 내 로컬 브랜치
origin/main = 원격 main의 로컬 추적 사본

실제 원격 서버에 있는 브랜치는 GitHub/GitLab 같은 서버에 있다.
내 로컬의 origin/main은 그것을 기억하는 ref인 것이다.


#### git fetch

원격 저장소의 최신 커밋과 브랜치 정보를 가져와서 origin/main 같은 remote-tracking ref를 업데이트한다.

주의: git fetch는 내 작업 브랜치 main을 직접 바꾸지 않는다.

원격에 새 커밋 D가 생겼다고 가정하자.

Remote origin/main:
A --- B --- C --- D

하지만 내 로컬은 아직 C에 있을 때, 

Local:
A --- B --- C
          ↑
        main
          ↑
    origin/main

git fetch를 하게 되면?

A --- B --- C --- D
          ↑       ↑
        main  origin/main

origin/main은 D로 이동
main은 아직 C

#### git pull

git pull은 보통 다음 두 명령의 조합이다.

git fetch
git merge

즉, git pull origin main 을 하게 되면
git fetch origin
git merge origin/main

이라는 뜻이고,

A --- B --- C --- D
          ↑       ↑
        main  origin/main

여기서 git pull을 하면 내 main에 origin/main을 합치게 되는 것이다.

* git pull --rebase

팀단위에서는 이렇게 사용하기도 한다.

git pull --rebase origin main

git fetch origin
git rebase origin/main

      C --- E
     /      ↑
A --- B     main
     \
      D
      ↑
 origin/main

내 로컬에는 E가 있고, 원격에는 D가 있다고 하자.
git pull --rebase를 하면 내 E를 D 뒤에 다시 붙인다.

A --- B --- D --- E'
                  ↑
                main

여기서 E'는 기존 E와 내용은 비슷하지만 새 커밋이다.

#### git stash
git stash는 현재 Working Directory와 Staging Area의 변경사항을 임시 저장하는 기능이다.

상황을 이렇게 가정하자.

HEAD commit = 깨끗한 기준점
Working Directory = 작업 중 변경 있음

그때 갑자기 다른 브랜치로 이동해야 하는 상황이 생길 수 있다.
그런데 아직 커밋하기는 애매하다면?

git stash push -m "WIP: parser refactor"

-> 현재 변경사항을 임시 저장하고, Working Directory를 깨끗하게 되돌린다.

#### stash 확인하기

git stash list

출력 예시
  stash@{0}: On main: WIP: parser refactor
  stash@{1}: On feature: WIP: socket setup

stash 내용을 다시 적용:
git stash apply stash@{0}

적용하고 stash 목록에는 남긴다.

적용하면서 목록에서 제거를 하려면?
  git stash pop

stash 적용 + stash 목록에서 제거

* stash에서 중요한 습관

나쁜 습관:
  git stash
좋은 습관:
  git stash push -m "WIP: fix poll event handling"

stash가 여러 개 쌓이면 나중에 구분이 어렵다.

stash@{0}: WIP on main: 39af2c...
stash@{1}: WIP on main: 12ab9d...
stash@{2}: WIP on feature: 3c5d1e...

이렇게 되면 무엇이 무엇인지 구분할 수 없다.

* 그런데 여기서 untracked file도 stash하려면?

  기본 git stash는 tracked file의 변경을 저장한다.

그런데 새로 만든 untracked file까지 stash하려면:
  git stash push -u -m "WIP: include new files"

여기서 -u는:
  --include-untracked 를 의미한다.

무시된 파일까지 포함하려면? :
  git stash push -a

하지만 -a는 .gitignore에 들어간 파일까지 포함하므로 신중하게 써야 한다.

### Branching

Git 협업을 하기위해선 가장 중요한 파트다.

merge
fast-forward
3-way merge
merge commit
rebase
interactive rebase

목표는 이 말들을 모두 이해할 수 있게 되어야한다.

#### Branching의 기본 상황

처음에는 main 하나만 있다고 가정하자.
A --- B --- C
          ↑
        main
          ↑
        HEAD

여기서 새 기능을 만들기 위해 브랜치를 만든다.

git switch -c feature

A --- B --- C
          ↑
        main
          ↑
      feature
          ↑
        HEAD

아직 main과 feature는 같은 커밋 C를 가리키고 있다. 이제 feature에서 커밋을 만든다.

git commit -m "Add login form"
git commit -m "Add login validation"

A --- B --- C
          ↑
        main
           \
            D --- E
                  ↑
              feature
                  ↑
                HEAD

이제 여기서 
A, B, C = 기존 main의 커밋
D, E    = feature 브랜치에서 새로 만든 커밋

이 상황을 기본 상황으로 전제하고 다음 상황들을 생각해보자.

#### merge란 무엇인가?

merge는 한 브랜치의 변경을 현재 브랜치에 합치는 명령이다.
예를 들어 현재 main으로 이동한다.

git switch main

A --- B --- C
          ↑
        main
           \
            D --- E
                  ↑
              feature

이제 여기서 feature를 main에 합쳐보자

git merge feature

이때 상황에 따라 두 가지 결과가 나온다.

1. fast-forward merge
2. 3-way merge with merge commit

#### Fast-forward merge

자 다시 현재 상황은 

A --- B --- C
          ↑
        main
           \
            D --- E
                  ↑
              feature

이거다. 그리고 이 그림은 사실

A --- B --- C --- D --- E
          ↑           ↑
        main      feature

이렇게도 볼 수 있다. 왜냐하면 main의 C가 feature의 조상이기 때문이다. 
즉, main은 단순히 feature보다 뒤처져 있을 뿐이라는거다.

이때 git merge feature를 사용하면?

Git은 새 커밋을 만들 필요가 없다. 그냥 main 포인터를 C에서 E로 앞으로 이동시키면 되는 것이다.

그 결과

A --- B --- C --- D --- E
                      ↑
                    main
                      ↑
                  feature
                      ↑
                    HEAD

fast-forward merge
= 현재 브랜치가 병합 대상 브랜치의 조상일 때,
  새 merge commit 없이 현재 브랜치 포인터만 앞으로 이동시키는 merge

* Fast-forward의 핵심

fast-forward는 새로운 커밋을 만들지 않는다. fast-forward는 “진짜 병합”이라기보다 “포인터 이동”에 가깝다.

그렇다면 Fast-forward가 불가능한 경우는 뭐가 있을까?

예를 들어보자
feature에서 작업하는 동안 main에도 새 커밋이 생겼다.

          D --- E
         /       ↑
A --- B --- C     feature
          \
           F
           ↑
         main

여기서 공통 조상은 C다.

common ancestor = C

feature는 C에서 갈라져서 D, E를 만들었다.

main도 C에서 갈라진 뒤 F를 만들었다.

이제 main에서 

git merge feature

를 했을 때, Git은 단순히 포인터만 이동할 수 없다.
왜냐하면 main의 F도 보존해야 하고, feature의 E도 가져와야 하기 때문이다.
그래서 Git은 새 커밋을 만든다.

          D --- E
         /       \
A --- B --- C --- F --- M
                    ↑
                  main

이것이 바로 3-way merge이다.

#### 3-way merge란?

위 상황에서 Git은 세 지점을 본다.

1. 공통 조상 C
2. 현재 브랜치 main의 끝 F
3. 병합 대상 feature의 끝 E

          D --- E   ← feature side
         /
A --- B --- C       ← common ancestor
          \
           F        ← current branch side

C와 F의 차이 = main에서 바뀐 내용
C와 E의 차이 = feature에서 바뀐 내용
두 변경을 합쳐서 M을 만든다

          D --- E
         /       \
A --- B --- C --- F --- M
                    ↑
                  main

여기서 M은 부모가 2명이다.
merge commit M
├── parent 1: F
└── parent 2: E

일반 커밋의 경우 보통 부모가 하나지만 이러한 경우에는 다르다. 

* --no-ff 옵션?

가끔 팀에서는 fast-forward가 가능해도 일부러 merge commit을 만들기도 한다.

git merge --no-ff feature

예를 들어 fast-forward 가능한 상태라면?

A --- B --- C --- D --- E
          ↑           ↑
        main      feature

일반 merge라면

A --- B --- C --- D --- E
                      ↑
                    main

하지만 --no-ff를 쓰면

A --- B --- C --------- M
          \           /
           D --- E ---

merge commit M이 생긴다.

이는 feature 브랜치에서 진행된 작업 단위를 히스토리에 명확히 남기기 위해서라고 볼 수 있다.
팀단위의 대규모 프로젝트일 경우에는 각자의 의견들을 모두 반영해야하는 경우가 있기 때문에 이렇게 일부로 히스토리를 남기기도 한다.

#### merge의 장점과 단점

장점으로는 

1. 기존 커밋을 그대로 보존한다.
2. 협업 중 안전하다.
3. 히스토리 조작이 적다.
4. 브랜치가 언제 갈라지고 언제 합쳐졌는지 명확하다.\

단점으로는 

1. 히스토리가 복잡해질 수 있다.
2. merge commit이 많아질 수 있다.
3. 그래프가 지저분해 보일 수 있다.

A --- B --- C ------- M1 -------- M2
     \       \       /           /
      D --- E \     /           /
               F---G           /
                    \         /
                     H --- I -

이러한 히스토리가 있다고 쳤을 때, 작은 프로젝트조차도 이런식의 복잡한 그래프를 구성할 수도 있다.

#### rebase란 무엇인가?

말 그대로 base를 다시 정한다 라는 뜻을 가지고 있는 rebase를 살펴보자.

          D --- E
         /       ↑
A --- B --- C     feature
          \
           F
           ↑
         main

feature는 C에서 갈라졌다. 
그런데 이제 main이 F까지 진행 되었고, 이때 우리는 feature를 최신 main 위에 다시 얹고 싶다.

git switch feature
git rebase main

그러면 Git은 feature의 커밋 D, E를 main의 F 뒤에 다시 적용한다.

A --- B --- C --- F --- D' --- E'
                              ↑
                          feature

D'와 E'는 기존 D, E와 다른 새 커밋이다. 내용은 비슷할 수 있지만 commit hash가 다르다. 왜냐하면 parent가 바뀌었기 때문.

기존

D의 parent = C
E의 parent = D

rebase 후

D'의 parent = F
E'의 parent = D'

parent 정보가 바뀌면 commit object 내용이 바뀌고, hash도 바뀐다.

example

          D --- E
         /
A --- B --- C --- F
              ↑   ↑
            base main

git switch feature
git rebase main

1. feature에서 main에 없는 커밋 D, E를 찾는다.
2. feature를 main의 끝 F로 옮긴다.
3. D, E의 변경 내용을 F 위에 다시 적용한다.

A --- B --- C --- F --- D' --- E'
                              ↑
                          feature

기존 D, E는 더 이상 feature가 가리키지 않습니다.

          D --- E    ← dangling/unreachable이 될 수 있음
         /
A --- B --- C --- F --- D' --- E'
                              ↑
                          feature

나중에 reflog로 찾을 수는 있지만, 일반 히스토리에서는 사라진 것처럼 보인다.

#### merge와 rebase 비교

같은 출발 상태:

          D --- E
         /
A --- B --- C --- F
                  ↑
                main

merge 결과

          D --- E ----
         /             \
A --- B --- C --- F --- M
                      ↑
                    main

특징:

D, E를 그대로 보존한다.
M이라는 merge commit이 생긴다.
브랜치가 갈라졌다가 합쳐진 흔적이 남는다.

rebase 결과

A --- B --- C --- F --- D' --- E'
                              ↑
                          feature

특징:

히스토리가 일직선처럼 보인다.
merge commit이 없다.
하지만 D, E가 D', E'로 새로 만들어진다.

#### rebase의 장점과 단점

장점

히스토리가 선형으로 깔끔해진다.
PR 리뷰 전에 커밋 정리에 좋다.
main 위에서 내 변경을 다시 테스트할 수 있다.

단점

커밋 해시가 바뀐다.
공유된 브랜치에서 쓰면 다른 사람 히스토리를 망가뜨릴 수 있다.
충돌이 여러 커밋에서 반복적으로 날 수 있다.

* 절대 규칙: 공유된 브랜치를 함부로 rebase하지 말 것

이미 다른 사람이 가져간 브랜치는 rebase하지 마라.

예를 들어 내가 feature를 GitHub에 push를 했다고 치자.

origin/feature:
C --- D --- E

동료가 이걸 pull해서 작업하고 있다.
그런데 내가 로컬에서:

git rebase main
git push --force

를 해버리면 원격은 이렇게 바뀐다.

C --- F --- D' --- E'

즉, 기존 D, E가 사라지고 D', E'가 생긴다. 동료의 로컬에는 아직 D, E 기반 작업이 있을 수 있다. 그러면 동료 입장에서는 히스토리가 꼬인다.
내 개인 feature 브랜치에서, 아직 공유 전이거나 팀 규칙상 허용된 경우에만 rebase
main, develop, shared feature 브랜치는 rebase 금지

#### git pull --rebase

앞에서 본 pull은 보통:

git fetch
git merge

그런데

git pull --rebase

git fetch
git rebase origin/main

상황;

      D
      ↑
     main
    /
A --- B --- C
          ↑
    origin/main

내 로컬 main에 D가 있고, 원격 origin/main에 C가 있다.

일반적인 pull merge라면

      D ----- M
     /       /
A --- B --- C

하지만 pull rebase라면?

A --- B --- C --- D'

그래서 pull --rebase는 로컬 커밋을 원격 최신 커밋 뒤로 다시 붙인다.

#### interactive rebase
interactive rebase는 커밋 히스토리를 정리하는 도구다.

예를 들어 feature 브랜치에 커밋이 지저분하게 쌓였을 때,

A --- B --- C --- D --- E
                  ↑   ↑
              typo fix debug commit

PR 올리기 전에 정리하고 싶으면

git rebase -i HEAD~3

그러면 최근 3개 커밋을 편집할 수 있다.
대략 이런 화면이 나오고

pick abc123 Add login form
pick def456 Fix typo
pick 789abc Add validation

여기서 할 수 있는 작업은

pick   = 그대로 사용
reword = 커밋 메시지만 수정
squash = 이전 커밋과 합치기
fixup  = 이전 커밋과 합치되 메시지는 버리기
drop   = 커밋 제거
edit   = 해당 커밋에서 멈춰서 수정

이러한 것들이 있다.

#### merge conflict

충돌은 두 브랜치가 같은 부분을 다르게 수정했을 때 생긴다.

예를 들어 공통 조상 C의 파일이 있고,

int port = 6667;

main에서는:

int port = 8080;

feature에서는:

int port = 4242;

이 상태에서 merge하거나 rebase하면 Git은 자동 판단을 못 한다.
그래서 파일에 충돌 표시를 남긴다.

<<<<<<< HEAD
int port = 8080;
=======
int port = 4242;
>>>>>>> feature

HEAD 쪽 변경 = 현재 브랜치 변경
feature 쪽 변경 = 병합하려는 브랜치 변경

그럴때는 직접 원하는 코드로 고쳐야 한다.

### 그렇다면 merge와 rebase는 언제 쓸까?

merge가 적합한 경우

공유 브랜치에서 안전하게 합칠 때
브랜치의 실제 분기/합류 기록을 남기고 싶을 때
main, develop 같은 공식 브랜치에 통합할 때

rebase가 적합한 경우

내 개인 feature 브랜치를 최신 main 위에 올리고 싶을 때
PR 올리기 전에 커밋 히스토리를 정리하고 싶을 때
아직 남들이 내 브랜치를 기반으로 작업하지 않을 때

#### 팀단위의 Git 작업 시 생각할 것.

main에서는 직접 작업하지 않는다.
feature 브랜치에서 작업한다.
작업 전 git fetch --prune으로 최신 원격 상태를 확인한다.
feature를 최신 main에 맞추고 싶으면 rebase 또는 merge를 팀 규칙에 맞춰 사용한다.
공유된 main/develop은 rebase하지 않는다.
PR 전에는 interactive rebase로 커밋을 정리할 수 있다.

### History Repair

이번 파트의 핵심 명령어는

git revert
git reset --soft
git reset --mixed
git reset --hard
git reflog

이러한 것들이 있다.
목표는 단순하다.

실수했을 때 무엇을 써야 안전하고, 무엇을 쓰면 위험한지 구분하는 것.

먼저 우리는 큰 원칙을 한가지 기억해둬야한다.
Git에서 히스토리 복구 명령은 크게 두 종류이다.

1. 새 커밋을 만들어서 실수를 취소한다
   → git revert

2. 브랜치 포인터를 과거로 이동시킨다
   → git reset

#### git revert

git revert는 기존 커밋을 삭제하지 않는다. 대신 특정 커밋의 변경사항을 반대로 적용하는 새 커밋을 만든다.
예를 들어 현재 히스토리가 이렇다고 하자.

A --- B --- C
          ↑
        main

여기서 C가 잘못된 커밋이라고 했을 때,

git revert C

를 실행하면 C가 사라지는 게 아니다.

A --- B --- C --- D
                ↑
              main

여기서 D는 이런 커밋이다.

D = C의 변경사항을 되돌리는 커밋이다. 즉, C는 히스토리에 남아 있다. 대신 D가 C를 취소한다.

#### revert는 왜 안전한가?

공유 브랜치에서는 revert가 안전하다. 
예를 들어 main에 이미 C가 push 되었다고 치자.

GitHub main:

A --- B --- C
          ↑
        main

팀원들이 이미 C를 pull했을 수 있다. 이때 C를 없애려고 reset 후 강제 push를 하면 팀원들의 히스토리와 충돌할 수 있기 때문에 revert를 사용하면

A --- B --- C --- D
                ↑
              main

revert를 하면 모든 팀원이 같은 히스토리를 계속 따라갈 수 있다.

#### git reset

이제 revert보다 좀 더 위험한 방식을 생각해보자.

현재 브랜치가 가리키는 커밋을 다른 커밋으로 옮긴다.

A --- B --- C
          ↑
        main
        HEAD

여기서

git reset B

를 하면 main이 B로 이동한다.

A --- B --- C
      ↑
    main
    HEAD

C는 브랜치에서 떨어져 나간다. 일반 git log에서는 C가 보이지 않을 수 있다.

#### reset의 세 가지 모드

git reset은 세 공간을 어떻게 바꾸느냐에 따라 달라진다.

Working Directory
Staging Area
HEAD / branch

git reset --soft
git reset --mixed
git reset --hard

차이는

| 명령어             | branch/HEAD | Staging Area | Working Directory |
| --------------- | ----------- | ------------ | ----------------- |
| `reset --soft`  | 이동          | 유지           | 유지                |
| `reset --mixed` | 이동          | 변경           | 유지                |
| `reset --hard`  | 이동          | 변경           | 변경                |

기본값은 --mixed.

#### git reset --soft의 경우

A --- B --- C
          ↑
        main
        HEAD

C 커밋을 만들었는데, 커밋 메시지를 잘못 썼거나 커밋을 다시 만들고 싶다.

git reset --soft HEAD~1

A --- B --- C
      ↑
    main
    HEAD

하지만 C의 변경사항은 staging area에 남아 있다.

branch는 B로 돌아감
stage에는 C의 변경사항이 남음
working directory에도 C의 변경사항이 남음

그래서 바로 다시 커밋할 수 있다.

용도는 보통

방금 만든 로컬 커밋을 다시 만들고 싶을 때
커밋 메시지를 바꾸거나 커밋을 합치고 싶을 때

단, 단순히 마지막 커밋 메시지만 바꾸려면 보통 이게 더 간단하다.

git commit --amend

#### git reset --mixed의 경우

git reset --mixed HEAD~1

결과

branch는 이전 커밋으로 이동
staging area도 이전 커밋 상태로 돌아감
working directory는 그대로 유지

A --- B --- C
          ↑
        main

git reset HEAD~1

A --- B --- C
      ↑
    main

C에서 했던 파일 변경은 working directory에는 남아 있다.
하지만 staging은 해제된다.

즉 git status를 보면:

Changes not staged for commit

용도

커밋은 취소하고, 수정한 파일은 작업 공간에 남겨두고 싶을 때

#### git reset --hard의 경우

git reset --hard HEAD~1

결과:
branch는 이전 커밋으로 이동
staging area도 이전 커밋으로 돌아감
working directory도 이전 커밋으로 돌아감

즉, C의 변경사항이 작업 폴더에서도 사라진다.

A --- B --- C
          ↑
        main

실행 후,

A --- B --- C
      ↑
    main

파일 상태도 B 시점으로 돌아간다.

C에서 수정했던 내용은 현재 작업 폴더에서 사라진다.

단, 커밋 C 자체는 즉시 완전히 삭제된 것은 아닐 수 있습니다. 보통 reflog로 당분간 찾을 수 있다.

하지만 uncommitted changes는 상황에 따라 복구가 어려울 수 있다.

#### reset을 쓸 때 가장 중요한 질문

reset을 쓰기 전에 반드시 이 질문을 해야 한다.

이 커밋이 이미 push되어서 다른 사람이 봤는가?

아직 push하지 않은 내 로컬 커밋의 경우 reset 가능성이 있습니다.

내 로컬에서만 존재하는 커밋
→ reset으로 정리 가능

하지만 이미 push된 공유 커밋이라면? reset은 위험하다.

이미 공유된 커밋
→ revert 우선

* git revert가 선호되는 경우

이미 공유된 커밋을 취소할 때
main/develop 같은 공용 브랜치에서 실수를 되돌릴 때
히스토리를 보존해야 할 때
팀원이 이미 pull했을 가능성이 있을 때

#### git reflog

이제 복구 도구이다.
git reflog는 HEAD와 브랜치가 과거에 어디를 가리켰는지 기록한다.

예를 들어 실수로:
  git reset --hard HEAD~1

원래 상태:

A --- B --- C
          ↑
        main

reset 이후

A --- B
      ↑
    main

C는 보이지 않는다. 이때 git reflog 를 보면 이런 식으로 나올 수 있다.

b123456 HEAD@{0}: reset: moving to HEAD~1
c789abc HEAD@{1}: commit: Add important feature
a111111 HEAD@{2}: commit: Previous commit

여기서 c789abc가 잃어버린 것처럼 보이는 C 커밋이다.

복구:

git reset --hard c789abc

그러면 main이 다시 C로 돌아간다.

A --- B --- C
          ↑
        main

다시 말해서 reflog는 “시간 여행 기록”이다.
일반 git log는 현재 브랜치에서 도달 가능한 커밋만 보여준다.
반면 reflog는 HEAD가 어디 있었는지의 이동 기록을 보여주는 것이다.

예를 들어 이런 행동들이 reflog에 남는다.
commit
reset
checkout/switch
rebase
merge
pull

즉, 실수했을 때 가장 먼저 확인할 명령은 git reflog 이다.

브랜치를 그 커밋으로 되돌리기
git reset --hard abc1234

안전하게 새 브랜치로 보존하기
git branch rescue abc1234
git switch rescue

#### uncommitted change는 더 위험하다

Git은 커밋된 것은 강하게 보호한다. 하지만 아직 커밋도, stash도, stage도 안 된 작업은 훨씬 위험하다.

echo "important work" > file.txt
git reset --hard

이렇게 해서 working directory 변경사항을 날리면 복구가 어려울 수 있다.
그래서 위험한 명령 전에는 습관적으로 확인해야 한다.

git status
git diff

그리고 애매하면 stash한다.
git stash push -u -m "backup before reset"

#### git restore

최근 Git에서는 작업 파일 복구에 restore를 많이 쓴다.

git restore file.txt

file.txt의 working directory 변경을 버리고, staging area 또는 HEAD 상태로 되돌린다.

Staging 취소

git restore --staged file.txt

file.txt를 staging area에서 내린다.
working directory의 수정 내용은 유지한다.


예전에는 이렇게 많이 했다.

git reset HEAD file.txt
요즘은 의미가 더 명확한 restore --staged를 쓰는 것이 좋다는게 주류이다.

#### reset과 restore 차이?

reset
= 브랜치/HEAD 이동 또는 stage 조작

restore
= 파일 내용을 특정 상태로 복구

커밋 히스토리나 브랜치 포인터를 움직인다
→ reset

파일 하나의 수정/스테이징을 되돌린다
→ restore

#### git clean

이건 untracked file을 삭제한다.

git clean -fd

Git이 추적하지 않는 파일과 디렉토리를 삭제한다. 매우 위험할 수 있다.

먼저 항상 dry-run을 쓴다.

git clean -fdn

여기서 -n은 실제 삭제하지 않고 무엇을 삭제할지 보여준다.
그다음 확신이 있으면

git clean -fd

#### 상황별 명령어 선택

1. 상황 1: 마지막 커밋 메시지만 바꾸고 싶다

git commit --amend

단, 이미 push한 커밋이면 신중해야 한다.

2. 상황 2: 방금 만든 로컬 커밋을 취소하고, 변경은 staged 상태로 두고 싶다

git reset --soft HEAD~1

3. 상황 3: 방금 만든 로컬 커밋을 취소하고, 변경은 파일에 남기고 싶다

git reset HEAD~1

또는

git reset --mixed HEAD~1

4. 상황 4: 방금 만든 로컬 커밋과 작업 내용을 완전히 버리고 싶다

git reset --hard HEAD~1

5. 상황 5: 이미 push된 커밋을 취소하고 싶다

git revert <commit>
git push

6. 상황 6: reset으로 커밋이 사라진 것 같다

git reflog
git branch rescue <commit>

7. 상황 7: stage에 올린 파일을 내리고 싶다

git restore --staged <file>

8. 상황 8: working directory 수정 내용을 버리고 싶다

git restore <file>

#### 마지막 생각할 안전 규칙

1. push된 커밋을 취소할 때는 revert.
2. push 전 로컬 커밋 정리는 reset 가능.
3. reset --hard 전에는 반드시 git status와 git diff.
4. 중요한 작업은 커밋하거나 stash한 뒤 위험한 명령 실행.
5. 커밋이 사라진 것 같으면 git reflog.
6. 복구할 때는 바로 reset하지 말고 rescue 브랜치를 먼저 만드는 것이 안전.

### Collaboration

Git을 혼자 잘 쓰는 것과 팀에서 안전하게 쓰는 것은 무엇이 다른가?

혼자 쓸 때 Git의 목적은 주로 이것이다.

내 작업을 저장하고 되돌리는 것

팀에서 Git의 목적은 더 넓다.

1. 팀원이 이해할 수 있는 변경 이력을 만든다.
2. 리뷰 가능한 단위로 작업을 나눈다.
3. 공유 브랜치의 히스토리를 안전하게 유지한다.
4. 실수가 생겼을 때 팀 전체를 덜 흔들리게 복구한다.

협업에서 가장 중요한 원칙

공유된 히스토리는 함부로 다시 쓰지 않는다.

git reset --hard <old-commit>
git rebase
git commit --amend
git push --force

이러한 것들이 공유된 히스토리를 함부로 바꾸는 행위들이다.

#### 브랜치 종류

1. main

배포 가능하거나 가장 안정적인 기준 브랜치

좋지않은 흐름

git switch main
git add .
git commit -m "fix stuff"
git push origin main

좋은 흐름

git switch main
git pull --ff-only
git switch -c feature/login

2. feature/*

새 기능 개발용 브랜치

feature/login
feature/chat
feature/payment
feature/irc-channel-broadcast

보통 main에서 갈라져 나온다.

git switch main
git pull --ff-only
git switch -c feature/irc-channel-broadcast

작업 후

git push -u origin feature/irc-channel-broadcast

-u는 upstream 설정이다.

3. bugfix/*

버그 수정용 브랜치

bugfix/socket-close-error
bugfix/parser-prefix-crash
bugfix/pollout-infinite-loop

4. hotfix/*

긴급 수정용 브랜치

hotfix/production-crash
hotfix/security-token-leak

보통 main에서 바로 갈라져서 빠르게 수정 후 merge한다.

#### Pull Request란 무엇인가?

Pull Request, 줄여서 PR은 단순히 “merge 요청”이 아니다.
PR은 세 가지 역할을 한다.

1. 코드 리뷰 단위
2. 변경사항 설명 문서
3. 히스토리 통합 지점

좋은 PR은 작아야 한다.

#### PR 설명을 어떻게 써야 하는가?

Summary
무엇을 변경했는가?

Why
왜 이 변경이 필요한가?

Changes
구체적으로 어떤 부분이 바뀌었는가?

Test
어떻게 테스트했는가?

Notes
리뷰어가 특히 봐야 할 부분은 무엇인가?

#### 커밋 메시지

커밋 메시지는 미래의 팀원에게 남기는 설명이다.

나쁜 예시

fix
update
asdf
final
real final
bug fixed

좋은 예시

Fix poll loop disconnect handling
Add channel broadcast helper
Refactor parser command dispatch
Validate nickname before registration

보통 명령형으로 쓰는 것이 좋다. 이유는 Git 자체가 기본 메시지에서 이런 스타일을 쓴다.

#### Conventional Commits

팀에서 원하면 Conventional Commits를 쓸 수 있다.

<type>(optional scope): <description>

예시

feat(channel): add broadcast helper
fix(server): handle closed socket in poll loop
refactor(parser): split prefix validation
test(irc): add PRIVMSG channel scenario
docs(readme): document local run command

자주 쓰는 type:

feat      = 기능 추가
fix       = 버그 수정
refactor  = 동작 변화 없는 구조 개선
test      = 테스트 추가/수정
docs      = 문서 수정
style     = 포맷팅, 세미콜론 등
chore     = 빌드/설정/잡무
perf      = 성능 개선

#### force push 규칙

rebase나 amend를 하면 커밋 해시가 바뀐다. 그러면 원격 브랜치와 내 로컬 브랜치가 달라진다.
이때 push하면 거절될 수 있다.

git push

non-fast-forward

이때 강제 push가 필요할 수 있다.

하지만 안전한 방식은

git push --force-with-lease

#### --force와 --force-with-lease 차이

1. --force

git push --force

원격이 현재 어떤 상태든 내 로컬 상태로 덮어써라.

2. --force-with-lease

git push --force-with-lease

내가 마지막으로 알고 있던 원격 상태와 지금 원격 상태가 같을 때만 덮어써라.
즉, 동료가 그 사이에 새 커밋을 push했다면 실패한다. 그래서 force push가 필요한 경우에는 보통 이걸 쓴다.

#### 코드 리뷰에서 봐야할 것

1. 변경 목적이 명확한가?
2. PR 크기가 적당한가?
3. 코드가 기존 구조와 맞는가?
4. 사이드 이펙트가 없는가?
5. 에러 처리와 예외 상황이 고려됐는가?
6. 테스트 방법이 적혀 있는가?
7. 커밋/PR 설명이 나중에 추적 가능하게 되어 있는가?

#### 팀에서 정해야 할 Git 규칙

1. main에 직접 push 가능한가?
2. PR은 필수인가?
3. PR approve 몇 명이 필요한가?
4. merge 방식은 무엇인가?
   - merge commit
   - squash merge
   - rebase merge
5. commit message 형식은 자유인가, conventional commits인가?
6. force push는 어디까지 허용되는가?
7. 브랜치 이름 규칙은 무엇인가?
8. CI 통과 전 merge 가능한가?