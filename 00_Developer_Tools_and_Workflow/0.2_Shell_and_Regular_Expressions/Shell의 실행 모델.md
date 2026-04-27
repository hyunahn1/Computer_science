# Shell은 무엇인가: 명령어가 실행되기 전 일어나는 일

## 개요

이번 공부를 하기전에 생각해보자

터미널에 이렇게 입력하면:

ls *.txt | grep report

컴퓨터는 이걸 그대로 ls라는 프로그램에게 전달할까?

아니다.

실제로는 shell이 먼저 해석하고 변형한 뒤, 여러 프로그램을 실행한다.
핵심 질문은 이것이다.

내가 터미널에 입력한 문자열은 언제, 누가, 어떤 규칙으로 해석되는가?

이 질문을 이해해야 glob, quote, pipe, regex, script가 전부 연결된다.

나는 이 대단원을 시작하면서, 하나하나 파고들어서 전부 깨우쳐보고자한다.

| 용어       | 의미                                             |
| -------- | ---------------------------------------------- |
| Terminal | 키보드 입력과 화면 출력을 담당하는 프로그램                       |
| Shell    | 명령어 문자열을 해석하고 프로그램을 실행하는 프로그램                  |
| Program  | 실제로 실행되는 명령어. 예: `ls`, `grep`, `python`, `git` |
| Process  | 실행 중인 프로그램의 인스턴스                               |

예를 들어 macOS에서 Terminal 앱을 열면 보통 내부에서 zsh가 실행된다.

Terminal.app
   ↓
zsh
   ↓
ls, grep, git, python 같은 프로그램 실행

Linux에서는 보통 bash가 기본 shell인 경우가 많다. macOS는 Catalina 이후 기본 shell이 zsh다.

자기가 무슨 shell을 사용하는지를 확인하려면 

echo $SHELL

예상 출력:
/bin/zsh 또는 /bin/bash

자 그럼 왜 필요할까?

grep *.txt

이 명령어를 보았을 때, grep이 *.txt를 이해해서 파일들을 찾겠지? 라고 생각할 수 있다.
하지만 실제로는 대개 그렇지 않다.
대부분의 경우 *.txt를 해석하는 주체는 grep이 아니라 shell이다. 
예를 들어 현재 디렉토리에 이런 파일이 있다고 하자.

a.txt
b.txt
note.md

그리고 다음 명령어를 입력한다.

echo *.txt

출력은 보통 이렇게 된다.

a.txt b.txt

왜냐하면 shell이 echo를 실행하기 전에 *.txt를 실제 파일 이름 목록으로 바꿨기 때문이다.
즉, 사용자가 입력한 명령어는 echo *.txt 이거 이지만, 
shell이 실제로 실행하는 명령어는 echo a.txt b.txt 이다.

다시 말해

grep "*.txt" file
grep *.txt file
grep '.*\.txt' file

이런 말이 전부 다르게 shell이 인식한다는 것을 인지해야한다.

### 내부 원리 / 작동 방식

shell은 명령어를 대충 왼쪽에서 오른쪽으로 실행하는 것이 아니다.
대략적으로 이런식으로 흘러간다.

사용자 입력
   ↓
1. Lexing / Parsing
   ↓
2. Expansion
   ↓
3. Redirection / Pipeline 구성
   ↓
4. Program 실행
   ↓
5. Exit code 수집

예제를 보자

cat *.log | grep "ERROR" > errors.txt

step 1. 명령어를 구조로 나눈다

cat *.log
|
grep "ERROR"
>
errors.txt

shell은 |가 있으므로 이것을 두 개의 명령어로 나눈다.
왼쪽 명령어: cat *.log
오른쪽 명령어: grep "ERROR"
출력 리다이렉션: > errors.txt

step 2. glob expansion을 수행한다

현재 디렉토리에 파일이 이렇게 있다고 하자.

app.log
server.log
memo.txt

*.log 는 shell에 의해 app.log server.log로 바뀐다.
따라서 왼쪽 명령어는 실제로 다음과 비슷하게 된다.

cat app.log server.log

Step 3. quote를 제거하되 의미는 보존한다

grep "ERROR"

여기서 "ERROR"의 따옴표는 grep에게 전달되지 않는다. shell 입장에서는 ERROR 라는 하나의 인자로 유지하라는 뜻이다.
그래서 실제로 grep이 받는 인자는
argv[0] = grep
argv[1] = ERROR

이렇게 된다. 따옴표 자체가 전달되는 것이 아니다.

Step 4. pipe를 만든다

cat app.log server.log | grep ERROR

shell은 두 프로세스를 실행하고, 왼쪽 프로세스의 stdout을 오른쪽 프로세스의 stdin에 연결한다.

cat app.log server.log
        │
        │ stdout
        ▼
      pipe
        │ stdin
        ▼
grep ERROR

Step 5. redirection을 적용한다

> errors.txt

이것은 grep의 stdout을 화면이 아니라 파일로 보내라는 뜻이다.

cat app.log server.log
        │
        ▼
grep ERROR
        │
        ▼
errors.txt

즉, grep이 *.log를 찾은 게 아니다.

shell이 파일 목록을 만든 다음, grep에게 넘긴 것이다.

#### bash 와 zsh의 차이

현재 디렉토리에 .log 파일이 하나도 없다고 하자.

bash에서는 기본적으로:
grep ERROR *.log
가 그대로 실행될 수 있다.

즉, grep이 실제로 받는 인자는:

ERROR
*.log

그래서 이런 에러가 날 수 있다.

grep: *.log: No such file or directory

반면 zsh는 기본 설정에서 매칭되는 파일이 없으면 shell 단계에서 에러를 낸다.

zsh: no matches found: *.log

| 상황            | bash 기본 동작        | zsh 기본 동작  |
| ------------- | ----------------- | ---------- |
| `*.log` 매칭 있음 | 파일 목록으로 확장        | 파일 목록으로 확장 |
| `*.log` 매칭 없음 | `*.log` 문자열 그대로 둠 | 에러 발생      |


그리고 pipe는 왼쪽 명령이 끝난 다음 오른쪽 명령이 실행되는 것이 아니라 보통 두 프로세스는 동시에 실행된다.

cat huge.log | grep ERROR

이 경우 cat이 전체 파일을 다 읽고 끝난 뒤 grep이 시작되는 것이 아니다.

cat이 조금 읽음
   ↓
pipe에 넣음
   ↓
grep이 그걸 읽음
   ↓
cat이 또 읽음
   ↓
grep이 또 처리함

즉, streaming 방식이다. 그래서 큰 로그 파일도 효율적으로 처리할 수 있다.

## Regex와 Glob의 차이

| 개념        | 뜻                        | 주로 누가 해석함?                     |
| --------- | ------------------------ | ------------------------------ |
| **glob**  | 파일 이름을 찾기 위한 간단한 패턴      | shell                          |
| **regex** | 문자열 안에서 패턴을 찾기 위한 정교한 언어 | `grep`, `sed`, `awk`, 언어 런타임 등 |


### Glob이란?

Glob은 shell이 파일 이름을 찾을 때 쓰는 패턴이다.

예를 들어 현재 폴더에 파일이 이렇게 있다고 하자.

a.txt
b.txt
memo.md
main.cpp

이때 echo *.txt 를 하면?
shell이 먼저 *.txt를 보고 .txt로 끝나는 파일들을 찾아라 라고 해석한다.
그래서 실제로는

echo a.txt b.txt 라고 해석한다.

| Glob    | 의미           | 예시              |
| ------- | ------------ | --------------- |
| `*`     | 아무 문자열 0개 이상 | `*.txt`         |
| `?`     | 아무 문자 1개     | `?.txt`         |
| `[abc]` | a, b, c 중 하나 | `[ab].txt`      |
| `[0-9]` | 숫자 하나        | `file[0-9].txt` |

### Regex란?
Regex는 문자열 내부에서 패턴을 찾는 언어다.
예를 들어 파일 내용이 이렇게 있다고 하자.

user: kim, email: kim@example.com
user: lee, email: lee@test.org
error: invalid input

이 안에서 이메일을 찾고 싶다면 regex를 쓴다.

[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}

이건

영문자/숫자/몇몇 기호가 여러 개 나오고
@
도메인 이름이 나오고
.
마지막에 com, org 같은 글자가 나온다

이런 뜻이다.
이런 패턴을 grep, sed, Python, JavaScript, C++ regex 같은 도구가 해석한다.

### 가장 중요한 차이

Glob은 “파일 이름”을 대상으로 한다

ls *.txt

이 명령에서 *.txt는 현재 폴더의 파일 이름들과 비교된다.

a.txt      match
b.txt      match
memo.md    no match
main.cpp   no match


Regex는 “문자열 내용”을 대상으로 한다

grep 'error' app.log

이 명령은 app.log 파일 안의 각 줄에서 error라는 문자열을 찾는다.

INFO server started       no match
ERROR database failed     no match if lowercase error only
error invalid input       match

### Regex 기본 문법 표

| 문법    | 의미            | 예시 패턴       | 매칭 예시      |    |    |
| ----- | ------------- | ----------- | ---------- | -- | -- |
| `abc` | 문자 그대로 `abc`  | `abc`       | `abc`      |    |    |
| `\.`  | 점 문자 `.` 그대로  | `a\.txt`    | `a.txt`    |    |    |
| `\\`  | 역슬래시 `\` 문자   | `C:\\Users` | `C:\Users` |    |    |
| `\|`  | 파이프 문자 `      | ` 그대로       | `A\|B`     | `A | B` |
| `\(`  | 괄호 문자 `(` 그대로 | `func\(`    | `func(`    |    |    |

regex에서 . * + ? ^ $ [ ] ( ) { } | \ 같은 문자는 특별한 의미가 있다.
문자 그대로 찾고 싶으면 보통 \로 escape한다.

| 문법   | 의미          | 예시 패턴  | 매칭 예시                |
| ---- | ----------- | ------ | -------------------- |
| `.`  | 아무 문자 1개    | `a.c`  | `abc`, `a1c`, `a-c`  |
| `.*` | 아무 문자 0개 이상 | `a.*c` | `ac`, `abc`, `axyzc` |
| `.+` | 아무 문자 1개 이상 | `a.+c` | `abc`, `axyzc`       |


. 은 “진짜 점”이 아니다.
진짜 점을 찾으려면 \. 를 써야 한다.

| 문법            | 의미           | 예시 패턴          | 매칭 예시            |
| ------------- | ------------ | -------------- | ---------------- |
| `[abc]`       | a, b, c 중 하나 | `gr[ae]y`      | `gray`, `grey`   |
| `[0-9]`       | 숫자 하나        | `file[0-9]`    | `file1`, `file7` |
| `[a-z]`       | 소문자 하나       | `[a-z]at`      | `cat`, `bat`     |
| `[A-Z]`       | 대문자 하나       | `[A-Z][a-z]+`  | `Kim`, `Park`    |
| `[A-Za-z]`    | 알파벳 하나       | `[A-Za-z]+`    | `Hello`          |
| `[A-Za-z0-9]` | 알파벳 또는 숫자 하나 | `[A-Za-z0-9]+` | `abc123`         |


| 문법       | 의미             | 예시 패턴     | 매칭 예시             |
| -------- | -------------- | --------- | ----------------- |
| `[^0-9]` | 숫자가 아닌 문자 하나   | `[^0-9]+` | `abc`, `hello!`   |
| `[^a-z]` | 소문자가 아닌 문자 하나  | `[^a-z]+` | `ABC`, `123`, `!` |
| `[^,]`   | 쉼표가 아닌 문자 하나   | `[^,]+`   | CSV에서 한 칸 읽기      |
| `[^"]`   | 큰따옴표가 아닌 문자 하나 | `"[^"]*"` | `"hello"`         |

[] 안의 ^는 “부정”이다.
[] 밖의 ^는 “문자열/줄의 시작”이다.

| 패턴       | 의미                |
| -------- | ----------------- |
| `^abc`   | abc로 시작           |
| `[^abc]` | a, b, c가 아닌 문자 하나 |

| 문법      | 의미             | 예시 패턴        | 매칭 예시               |
| ------- | -------------- | ------------ | ------------------- |
| `*`     | 앞 패턴이 0회 이상    | `ab*c`       | `ac`, `abc`, `abbc` |
| `+`     | 앞 패턴이 1회 이상    | `ab+c`       | `abc`, `abbc`       |
| `?`     | 앞 패턴이 0회 또는 1회 | `colou?r`    | `color`, `colour`   |
| `{n}`   | 정확히 n회         | `[0-9]{4}`   | `2026`              |
| `{n,}`  | n회 이상          | `[0-9]{2,}`  | `12`, `123`         |
| `{n,m}` | n회 이상 m회 이하    | `[0-9]{2,4}` | `12`, `123`, `2026` |

| 문법      | 의미               | 예시 패턴      | 매칭 예시          |
| ------- | ---------------- | ---------- | -------------- |
| `^`     | 줄/문자열의 시작        | `^ERROR`   | `ERROR failed` |
| `$`     | 줄/문자열의 끝         | `\.txt$`   | `a.txt`        |
| `^...$` | 전체 줄이 패턴과 정확히 일치 | `^[0-9]+$` | `12345`        |

| 문법   | 의미        | 예시 패턴       | 매칭 예시                  |
| ---- | --------- | ----------- | ---------------------- |
| `\b` | 단어 경계     | `\berror\b` | `error`                |
| `\B` | 단어 경계가 아님 | `\Bcat\B`   | `concatenate` 안의 `cat` |

| 문법        | 의미                  | 예시 패턴      | 매칭 예시        |        |               |
| --------- | ------------------- | ---------- | ------------ | ------ | ------------- |
| `(abc)`   | 그룹                  | `(ab)+`    | `ab`, `abab` |        |               |
| `(cat     | dog)`               | cat 또는 dog | `I like (cat | dog)s` | `I like cats` |
| `(?:abc)` | non-capturing group | `(?:ab)+`  | `ab`, `abab` |        |               |

| 문법    | 의미     | 예시 패턴       | 매칭 예시             |         |              |        |         |
| ----- | ------ | ----------- | ----------------- | ------- | ------------ | ------ | ------- |
| `cat  | dog`   | cat 또는 dog  | `cat              | dog`    | `cat`, `dog` |        |         |
| `(GET | POST)` | GET 또는 POST | `^(GET            | POST) ` | `GET /index` |        |         |
| `(jpg | png    | gif)`       | jpg 또는 png 또는 gif | `.(jpg  | png          | gif)$` | `a.jpg` |

| 문법    | 의미                | 예시      |
| ----- | ----------------- | ------- |
| `.*`  | greedy, 가능한 많이 먹음 | `<.*>`  |
| `.*?` | lazy, 가능한 적게 먹음   | `<.*?>` |

# Paths: 파일 시스템을 탐색하는 언어

터미널에서 다음 명령어를 입력한다고 하자.

cd ../project 또는 ls ~/Desktop

이때 shell은 ../project, ~/Desktop을 어떻게 이해할까?

컴퓨터는 “내가 말하는 위치”를 어떻게 파일 시스템 안의 실제 위치로 해석하는가?
Shell을 잘 쓰려면 먼저 path, 즉 경로를 정확히 이해해야 한다.

## 개념 설명

### Path란?

Path는 파일이나 디렉토리의 위치를 표현하는 문자열이다.

/Users/hyun/Desktop/report.txt

이건 macOS에서 어떤 파일의 위치를 나타낸다.
Linux라면 이런 형태가 흔하다.

/home/hyun/project/main.cpp

파일 시스템은 대략 트리 구조다.

/
├── bin
├── etc
├── home
│   └── hyun
│       └── project
│           └── main.cpp
└── usr

macOS는 보통 이렇게 생겼다.

/
├── Applications
├── System
├── Users
│   └── hyun
│       ├── Desktop
│       ├── Downloads
│       └── project
└── usr

여기서 /는 최상위 디렉토리다.
영어로 root directory라고 한다.

### 왜 필요한지

터미널에서 거의 모든 명령어는 “어디에 있는 파일을 대상으로 할 것인가”를 필요로 한다.

경로를 정확히 이해하지 못하면 이런 문제가 생긴다.

rm -rf build

이 명령이 현재 어디에서 실행되는지 모르고 쓰면 엉뚱한 build 디렉토리를 지울 수 있다.

### 내부 원리 / 작동 방식

현재 작업 디렉토리, CWD

Shell은 항상 하나의 “현재 위치”를 가지고 있다.

이걸 current working directory, 줄여서 CWD라고 한다.

확인하는 법

pwd

pwd는 print working directory다.
지금 shell이 바라보고 있는 현재 디렉토리를 출력하라

### 상대 경로와 절대 경로

| 종류    | 예시                             | 기준            |
| ----- | ------------------------------ | ------------- |
| 절대 경로 | `/Users/hyun/project/main.cpp` | root `/`부터 시작 |
| 상대 경로 | `project/main.cpp`             | 현재 디렉토리부터 시작  |

#### 절대 경로 Absolute Path
절대 경로는 항상 /로 시작한다.

/Users/hyun/Desktop

root /
 → Users
   → hyun
     → Desktop

현재 내가 어디에 있든 같은 위치를 의미한다.

cd /Users/hyun/Desktop

이 명령은 현재 위치가 어디든 항상 /Users/hyun/Desktop으로 간다.

#### 상대 경로 Relative Path

상대 경로는 현재 디렉토리를 기준으로 한다.

현재 위치가

/Users/hyun

cd Desktop 은 cd /Users/hyun/Desktop 를 의미한다.

하지만 현재 위치가 /Users/hyun/project 이라면 cd Desktop 은 /Users/hyun/project/Desktop 으로 가려고 한다.
즉, 같은 명령어라도 현재 위치에 따라 의미가 달라진다.

#### 특별한 경로 기호

1. . 현재 디렉토리
.은 현재 디렉토리를 의미한다.

./program

현재 디렉토리에 있는 program을 실행하라

2. .. 부모 디렉토리

..은 한 단계 위 디렉토리다.

/Users/hyun/project/src 일때, cd .. 하면 /Users/hyun/project 로 간다.

cd ../include은 현재 위치에서 한 단계 위로 간 뒤 include로 들어가라 이다.

3. ~ 홈 디렉토리

~는 사용자의 home directory를 의미한다.

4. / root directory
/는 파일 시스템 최상단이다.

cd /

하면 최상위 디렉토리로 이동한다.
하지만 주의해야한다.

| 명령어    | 의미         |
| ------ | ---------- |
| `cd /` | 파일 시스템 최상단 |
| `cd ~` | 내 홈 디렉토리   |

### PATH 환경변수

명령어는 어디서 찾는가?

다음 명령어를 입력한다고 하자.

ls

shell은 ls라는 프로그램을 어떻게 찾을까? 답은 PATH 환경변수다.

echo $PATH

예시 출력:
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

PATH는 디렉토리 목록이다. :로 구분된다.

/usr/local/bin
/usr/bin
/bin
/usr/sbin
/sbin

shell은 사용자가 입력한 명령어가 /를 포함하지 않으면, PATH 디렉토리들을 순서대로 뒤진다.

사용자 입력: ls

1. /usr/local/bin/ls 있는지 확인
2. /usr/bin/ls 있는지 확인
3. /bin/ls 있는지 확인
4. 찾으면 실행

| 명령어             | 설명                                |
| --------------- | --------------------------------- |
| `which ls`      | PATH에서 찾은 실행 파일 위치 출력             |
| `command -v ls` | shell 기준으로 명령어가 어떻게 해석되는지 출력      |
| `type ls`       | alias, function, builtin 여부까지 알려줌 |

예:

type cd

출력 예:
cd is a shell builtin

왜 cd는 외부 프로그램이 아니라 shell builtin일까?
cd가 별도 프로그램이면, 그 프로그램 자신의 현재 디렉토리만 바꾸고 종료된다. 그러면 부모 shell의 현재 디렉토리는 바뀌지 않는다.
따라서 cd는 shell 자체가 처리해야 한다.

shell process
  └── external cd process가 cwd 변경 후 종료
      → shell의 cwd는 그대로

그래서 cd는 shell builtin이어야 한다.

# Quoting: Shell에서 가장 중요한 생존 기술

## 핵심 질문

다음 명령어들은 비슷해 보인다.

echo *.txt
echo "*.txt"
echo '*.txt'

하지만 결과는 완전히 다르다.

또 이것도 보자.

echo $HOME
echo "$HOME"
echo '$HOME'

그렇다면 Shell은 공백, $, *, \, 따옴표를 언제 특별한 문법으로 해석하고, 언제 그냥 문자로 취급하는가?

## 개념

Shell에서 quote는 그저 문자열을 보기 좋게 묶는 기호가 아니다.

Quote는 shell에게 이렇게 지시한다.

이 안의 문자를 어디까지 특별하게 해석할지 결정해라.

대표적인 quote 방식은 세 가지다.

| 방식           | 예시                    | 핵심 의미                              |
| ------------ | --------------------- | ---------------------------------- |
| Single quote | `'hello $USER *.txt'` | 거의 모든 특수 해석을 막음                    |
| Double quote | `"hello $USER"`       | 변수 확장은 허용, word splitting/glob은 막음 |
| Backslash    | `hello\ world`        | 다음 문자 하나의 특수 의미를 제거                |

## 왜 필요할까?

Shell은 기본적으로 공백을 기준으로 단어를 나눈다.

echo hello world

argv[0] = echo
argv[1] = hello
argv[2] = world

echo "hello world"

argv[0] = echo
argv[1] = hello world

겉보기 출력은 같다.

hello world

하지만 내부적으로는 다르다.

이 차이는 파일 이름에 공백이 있거나, 변수에 공백이 들어 있을 때 치명적이다.

## 내부 원리 / 작동 방식

Shell은 명령어 실행 전에 여러 단계를 거친다.

1. Parsing
2. Expansion
3. Word splitting
4. Globbing
5. Program execution

예를 들어 

cat $file

에서 file 변수 값이 다음과 같다고 하자.

file="my file.txt"

그러면 quote 없이 cat $file은 대략 이렇게 된다.

1. 변수 확장:
   cat my file.txt

2. word splitting:
   cat
   my
   file.txt

3. 실행:
   cat my file.txt

즉 cat은 my file.txt 하나가 아니라 my와 file.txt 두 파일을 열려고 한다.
그렇기 때문에 올바른 방식은 cat "$file" 이다.
이 경우

1. 변수 확장:
   cat "my file.txt"

2. double quote가 word splitting을 막음

3. 실행:
   argv[0] = cat
   argv[1] = my file.txt

그래서 shell script에서는 거의 항상 변수를 quote해야 한다.

"$var"

이 습관은 매우 중요하다.

## Single Quote '...'

Single quote는 내부의 거의 모든 특수 해석을 막는다.

입력

echo '$HOME *.txt'

출력

$HOME *.txt

여기서 $HOME은 변수로 확장되지 않는다.
*.txt도 glob으로 확장되지 않는다.

즉 single quote 안에서는 대부분의 문자가 문자 그대로 유지된다.

예시

name="hyun"
echo '$name'

출력

$name

반면

echo "$name"

hyun

### single quote 안에 single quote를 넣을 수 있는가?

echo 'I'm here'

이건 깨진다. 왜냐하면 shell은 이렇게 본다.

'I'
m
 here'

 해결 방법은 보통 이렇게 쓴다.

 echo 'I'\''m here'

 'I'      → I
\'       → 진짜 작은따옴표 문자
'm here' → m here

출력

I'm here

## Double Quote "..."

Double quote는 일부 확장은 허용하고, 일부는 막는다.

허용되는 것들?

| 기능               | double quote 안에서 동작? |
| ---------------- | -------------------- |
| 변수 확장 `$var`     | 동작                   |
| 명령 치환 `$(cmd)`   | 동작                   |
| 산술 확장 `$((...))` | 동작                   |
| glob `*`         | 막힘                   |
| word splitting   | 막힘                   |

예시

name="hyun"
echo "hello $name"

출력

hello hyun

그 반면

echo "*.txt"

출력

*.txt

*가 glob으로 확장되지 않는다.

### 가장 중요한 사용법

변수를 사용할 때는 기본적으로 double quote를 붙인다.

rm "$file"
cp "$src" "$dst"
grep "$pattern" "$file"

왜냐하면 변수 값에 공백, 줄바꿈, * 같은 문자가 들어 있을 수 있기 때문이다.

## Backslash \

Backslash는 다음 문자 하나의 특수 의미를 제거한다.

echo hello\ world

출력: 

hello world

| 표현   | 의미         |
| ---- | ---------- |
| `\ ` | 공백 문자      |
| `\$` | `$` 문자 그대로 |
| `\*` | `*` 문자 그대로 |
| `\"` | `"` 문자 그대로 |
| `\\` | `\` 문자 그대로 |

## Word Splitting

Shell은 quote 밖에서 변수 확장 결과를 다시 공백 기준으로 쪼갠다.

이걸 word splitting이라고 한다.

예시

items="a b c"
printf '<%s>\n' $items

출력:

<a>
<b>
<c>

왜냐하면 $items가 a b c로 확장된 뒤, shell이 세 단어로 나눴기 때문이다.

printf '<%s>\n' "$items"

출력

<a b c>

하나의 인자로 전달된다.


# stdout, stderr, stdin: 표준 입출력과 File Descriptor

## 핵심질문

grep ERROR app.log > result.txt

이 명령은 화면에 출력하지 않고 result.txt에 결과를 저장한다.

그런데 다음 명령은 어떨까?

grep ERROR missing.log > result.txt

missing.log가 없으면 에러 메시지는 여전히 화면에 나온다.

grep: missing.log: No such file or directory

왜 그럴까?

핵심 질문은 이것이다.

정상 출력과 에러 출력은 왜 분리되어 있고, shell은 그것들을 어떻게 파일이나 다른 명령어로 연결하는가?

## 개념

Unix 계열 시스템에서 프로그램은 기본적으로 세 개의 입출력 통로를 가진다.

| 이름     | 번호 | 의미       | 기본 연결  |
| ------ | -: | -------- | ------ |
| stdin  |  0 | 표준 입력    | 키보드    |
| stdout |  1 | 표준 출력    | 터미널 화면 |
| stderr |  2 | 표준 에러 출력 | 터미널 화면 |

이 번호를 file descriptor, 줄여서 FD라고 한다.

Process
├── fd 0: stdin
├── fd 1: stdout
└── fd 2: stderr

예를 들어:

cat 을 그냥 실행하면 cat은 stdin에서 입력을 기다린다.

keyboard → stdin → cat → stdout → terminal

반면 ls는 결과를 stdout으로 출력한다. 

ls → stdout → terminal

에러가 나면 stderr로 출력한다.

ls missing_file → stderr → terminal

## 왜 필요한지

정상 결과와 에러 메시지를 같은 통로로 보내면 자동화가 어려워진다.

예를 들어 스크립트에서 파일 목록만 저장하고 싶다고 하자.

ls src > files.txt

정상 출력은 files.txt로 들어간다.

하지만 에러는 화면에 남는다.

ls missing_dir > files.txt

ls: missing_dir: No such file or directory

이게 좋은 이유는 files.txt 안에 에러 메시지가 섞이지 않기 때문이다.

정상 데이터 → stdout
문제 보고 → stderr

이 분리는 실무에서 매우 중요하다.

find . -name "*.cpp" > cpp_files.txt
이때 permission denied 같은 메시지가 결과 파일에 섞이면 후속 처리 프로그램이 깨질 수 있다.

그래서 에러는 별도 통로인 stderr로 보낸다.

## 내부 원리 / 작동 방식

프로그램은 파일을 직접 “화면”에 쓰는 게 아니다

프로그램은 보통 이렇게 생각하지 않는다.

'화면에 글자를 출력해야지'

대신 운영체제에게 말한다.

'fd 1번에 bytes를 쓰겠다.'

C 코드로 보면 대략 이런 식이다.

write(1, "hello\n", 6);

여기서 1이 stdout이다.

에러는 보통:

write(2, "error\n", 6);

여기서 2가 stderr다.

즉, shell은 프로그램을 실행하기 전에 fd 1과 fd 2가 어디로 연결될지 바꿀 수 있다.

## 기본 상태

echo hello

echo process
├── fd 0 stdin  ← keyboard
├── fd 1 stdout → terminal
└── fd 2 stderr → terminal

출력:

hello

stdout redirection

echo hello > out.txt

이때 shell은 echo를 실행하기 전에 fd 1을 파일로 바꾼다.

echo process
├── fd 0 stdin  ← keyboard
├── fd 1 stdout → out.txt
└── fd 2 stderr → terminal

그래서 화면에는 아무것도 안 나오고 out.txt에 저장된다.

stderr redirection

ls missing 2> error.txt

여기서 2>는 fd 2, 즉 stderr를 파일로 보내라는 뜻이다.

ls process
├── fd 1 stdout → terminal
└── fd 2 stderr → error.txt

## 기본 리다이렉션 문법

stdout 저장
cmd > file

cmd의 stdout을 file에 저장한다.
기존 file 내용은 덮어쓴다.

echo hello > out.txt

stdout 추가

cmd >> file

cmd의 stdout을 file 뒤에 추가한다.

echo first > log.txt
echo second >> log.txt
cat log.txt

first
second

stderr 저장

cmd 2> error.txt

cmd의 stderr를 error.txt에 저장한다.

ls missing_file 2> error.txt
화면에는 에러가 안 나오고 error.txt에 저장된다.

cmd > all.txt 2>&1

stdout을 all.txt로 보낸다.
stderr도 stdout이 가는 곳으로 보낸다.

### 2>&1의 정확한 의미

2>&1

뜻은 fd 2를 fd 1이 현재 가리키는 곳으로 복사하라.

2>&1은 “2를 1로 보낸다”가 아니라, 2번 통로를 1번 통로의 현재 목적지와 같게 만든다는 뜻이다.

### stdin redirection

stdin은 fd 0이다.

cmd < input.txt

뜻: cmd의 stdin을 input.txt에서 읽게 한다.

예시

cat < input.txt

이는 사실 

cat input.txt 이것과 비슷해보인다. 하지만

| 명령어               | 의미                              |
| ----------------- | ------------------------------- |
| `cat input.txt`   | cat이 직접 input.txt 파일을 연다        |
| `cat < input.txt` | shell이 input.txt를 stdin으로 연결해준다 |

## Pipe와 stdout/stderr

cmd1 | cmd2

pipe는 기본적으로 cmd1의 stdout만 cmd2의 stdin으로 연결한다.

cmd1 stdout → pipe → cmd2 stdin
cmd1 stderr → terminal

예:

grep ERROR app.log missing.log | wc -l

grep의 정상 결과는 wc -l로 들어간다.
하지만 missing.log 에러는 기본적으로 터미널에 그대로 나온다.

stderr까지 pipe에 넣고 싶다면:

grep ERROR app.log missing.log 2>&1 | wc -l

하지만 이 경우 에러 메시지도 wc -l의 입력으로 들어가므로 “정상 결과 줄 수”가 왜곡될 수 있다.

실무에서는 보통 에러는 따로 저장한다.

grep ERROR app.log missing.log 2> errors.log | wc -l

정상 결과만 wc -l로 전달
에러는 errors.log에 저장

# Pipelines: 여러 프로그램을 연결하는 Unix식 사고방식

여러 개의 작은 프로그램을 어떻게 하나의 데이터 처리 흐름으로 연결하는가?

Unix shell의 강력함은 거대한 프로그램 하나를 쓰는 데서 나오지 않는다.
작은 프로그램들을 pipe로 연결하는 방식에서 나온다.

## 개념

Pipeline이란?

Pipeline은 한 명령어의 stdout을 다음 명령어의 stdin으로 연결하는 구조다.

cmd1 | cmd2

cmd1의 stdout → cmd2의 stdin

cmd1
  │ stdout
  ▼
pipe
  │ stdin
  ▼
cmd2

예시:
ls | grep '\.cpp$'

ls의 출력 결과를 grep의 입력으로 넘긴다.
grep은 그중 .cpp로 끝나는 줄만 출력한다.

## 왜 필요할까

파일이나 로그를 분석할 때 한 프로그램이 모든 일을 다 하게 만들면 복잡해진다.

예를 들어 다음 작업을 생각해보자.

1. 로그 파일을 읽는다.
2. ERROR가 있는 줄만 고른다.
3. 중복을 제거한다.
4. 정렬한다.
5. 개수를 센다.

이걸 하나의 프로그램으로 직접 만들 수도 있지만, shell에서는 작은 도구를 연결할 수 있다.

grep 'ERROR' app.log | sort | uniq | wc -l

| 명령어     | 역할           |
| ------- | ------------ |
| `grep`  | 조건에 맞는 줄만 고름 |
| `sort`  | 정렬           |
| `uniq`  | 인접한 중복 줄 제거  |
| `wc -l` | 줄 수 계산       |

이것이 Unix의 기본 철학이다.
작은 도구를 만들고, pipe로 조합한다.

## 내부 원리 / 작동 방식

Pipeline은 순차 실행이 아니다

보통 이렇게 생각할 수 있다.

1. cmd1이 끝까지 실행된다.
2. 그 결과가 저장된다.
3. cmd2가 실행된다.

하지만 실제로는 보통 그렇지 않다.

cat huge.log | grep ERROR
보통 두 프로세스는 거의 동시에 실행된다.

cat process        grep process
    │                   ▲
    │ stdout            │ stdin
    ▼                   │
  pipe buffer ──────────┘

1. shell이 pipe를 만든다.
2. shell이 cat process를 만든다.
3. shell이 grep process를 만든다.
4. cat은 stdout에 데이터를 쓴다.
5. grep은 stdin에서 데이터를 읽는다.
6. pipe buffer가 중간에서 데이터를 전달한다.

### Pipe는 커널이 관리하는 버퍼다

Pipe는 단순한 문자열 치환이 아니다.

운영체제 커널이 관리하는 데이터 통로다.

Process A write()
      │
      ▼
 kernel pipe buffer
      │
      ▼
Process B read()

shell은 연결만 설정한다.
실제 데이터 전달은 kernel이 한다.

### stdout만 pipe로 넘어간다

cmd1 | cmd2

cmd1의 stdout → cmd2의 stdin
cmd1의 stderr → terminal

### Pipeline과 redirection의 차이

cmd > file

cmd의 출력을 파일로 보낸다.

cmd stdout → file


Pipeline

cmd1 | cmd2

cmd1의 출력을 cmd2의 입력으로 보낸다.

cmd1 stdout → cmd2 stdin

### Exit code와 pipeline

pipeline에서 각 명령어는 각자 exit code를 가진다.

grep ERROR app.log | wc -l

여기에는 최소 두 개의 exit code가 있다.

grep의 exit code
wc의 exit code

그런데 shell에서 $?를 보면 기본적으로 마지막 명령어의 exit code를 보여준다.

grep ERROR app.log | wc -l
echo $?

기본적으로는 wc -l의 exit code다.
pipeline의 실패 판정은 생각보다 단순하지 않다.
기본적으로는 마지막 명령어의 exit code가 중요하게 취급된다.


# Exit Codes와 && / ||: Shell에서 성공과 실패를 다루는 방식

## 핵심 질문

make && ./program

이 명령은 무슨 뜻일까?
쉽게 보면 make 하고 program 실행이다. 
정확히는 

make가 성공하면 ./program을 실행하라.
make가 실패하면 ./program은 실행하지 마라.

여기서 나오는 핵심 질문은 이것이다.

* Shell은 어떤 명령이 성공했는지 실패했는지 어떻게 판단하는가?

답은 exit code다.

## 개념

### Exit Code란?

프로그램은 종료될 때 운영체제에게 숫자 하나를 남긴다.

이 숫자를 exit code, 또는 exit status라고 한다.

Unix 계열에서는 관례적으로:

| Exit Code | 의미           |
| --------: | ------------ |
|       `0` | 성공           |
|    `1` 이상 | 실패 또는 특별한 상태 |

C 프로그램으로 보면 이런 식이다.

int main(void)
{
    return 0;
}

이 프로그램은 성공으로 종료된다.

int main(void)
{
    return 1;
}

반면에 이 프로그램은 실패 상태로 종료된다.

## 왜 필요할까?

Shell script나 자동화에서는 “다음 명령을 실행해도 되는가?”를 판단해야 한다.

make
./program

이렇게 쓰면 문제가 있다.

make가 실패해도 ./program이 실행될 수 있다.

더 안전한 방식은:

make && ./program

git pull && make && ./server

git pull 성공
→ make 성공
→ server 실행

중간에 하나라도 실패하면 뒤 명령은 실행되지 않는다.

## 내부 원리 / 작동 방식

Shell은 명령어가 끝나면 그 명령의 exit code를 저장한다.

방금 실행한 명령의 exit code는 $?로 확인할 수 있다.

true
echo $?

0

true는 항상 성공하는 명령이다.

false
echo $?

1

false는 항상 실패하는 명령이다.

### 존재하는 파일 확인

ls
echo $?

보통 0으로 성공 코드를 보낸다. 하지만

ls missing_file
echo $?

이렇게 없는 파일을 보면?

ls: missing_file: No such file or directory
2

여기서 2라는 숫자는 ls가 실패 상태로 종료되었다는 뜻이다.

정확한 숫자는 프로그램마다 다를 수 있다.
하지만 shell 입장에서는 중요한 구분이 이것이다.

0인가?
0이 아닌가?

## &&: 성공하면 다음 명령 실행

cmd1 && cmd2

cmd1이 성공하면 cmd2를 실행한다.
cmd1이 실패하면 cmd2는 실행하지 않는다.

cmd1 실행
   │
   ├── exit code 0     → cmd2 실행
   │
   └── exit code non-0 → cmd2 실행 안 함

mkdir build && cd build

build 디렉토리 생성에 성공하면 build로 이동

## ||: 실패하면 다음 명령 실행

cmd1 || cmd2

cmd1이 실패하면 cmd2를 실행한다.
cmd1이 성공하면 cmd2는 실행하지 않는다.

cmd1 실행
   │
   ├── exit code 0     → cmd2 실행 안 함
   │
   └── exit code non-0 → cmd2 실행

   grep 'ERROR' app.log || echo "No error found"

주의할 점이 있다.

grep은 매칭되는 줄이 없으면 exit code 1을 반환한다.
이건 “프로그램 오류”라기보다는 “찾는 패턴이 없음”이라는 의미다.

grep 'ERROR' app.log || echo "No error found"

는 ERROR가 없을 때 메시지를 출력한다.

## ;와 &&의 차이

cmd1; cmd2

cmd1 결과와 상관없이 cmd2 실행

make; ./program

make가 실패해도 ./program을 실행한다.

| 문법             | 의미                  |       |                 |
| -------------- | ------------------- | ----- | --------------- |
| `cmd1; cmd2`   | 앞 명령 결과와 상관없이 다음 실행 |       |                 |
| `cmd1 && cmd2` | 앞 명령 성공 시 다음 실행     |       |                 |
| `cmd1          |                     | cmd2` | 앞 명령 실패 시 다음 실행 |

## if와 exit code

Shell의 if는 명령어의 출력 내용이 아니라 exit code를 본다.

if grep 'ERROR' app.log; then
  echo "There are errors"
else
  echo "No errors"
fi

grep 명령의 exit code가 0이면 then
아니면 else

여기서 grep은 다음 규칙을 가진다.

| 상황         | grep exit code |
| ---------- | -------------: |
| 매칭되는 줄 있음  |            `0` |
| 매칭되는 줄 없음  |            `1` |
| 파일 없음 등 오류 |            `2` |

그래서 grep은 조건문에서 자주 쓰인다.

## [ ... ]도 명령어다

if [ -f app.log ]; then
  echo "file exists"
fi

여기서 [ ... ]는 문법처럼 보이지만 실제로는 명령어다.

type [

해보면 shell builtin이라고 나올 수 있다.

[ is a shell builtin

즉,

[ -f app.log ]

이 명령어의 뜻은 app.log가 일반 파일이면 exit code 0 아니면 non-zero 를 반환하는 명령어다.

그래서 다음과 같은 경우도 가능하다.

[ -f app.log ] && echo "exists"

app.log 파일이 있으면 exists 출력

자주 쓰는 test 조건

| 표현                 | 의미            |
| ------------------ | ------------- |
| `[ -e file ]`      | file이 존재      |
| `[ -f file ]`      | 일반 파일         |
| `[ -d dir ]`       | 디렉토리          |
| `[ -x file ]`      | 실행 가능         |
| `[ -r file ]`      | 읽기 가능         |
| `[ -w file ]`      | 쓰기 가능         |
| `[ -z "$var" ]`    | 문자열 길이가 0     |
| `[ -n "$var" ]`    | 문자열 길이가 0이 아님 |
| `[ "$a" = "$b" ]`  | 문자열 같음        |
| `[ "$a" != "$b" ]` | 문자열 다름        |

# Shell Scripts: shebang, arguments, functions

## 핵심 질문

지금까지는 터미널에 명령어를 한 줄씩 입력했다.

예:

grep 'ERROR' app.log | sort | uniq -c

그런데 같은 작업을 매번 반복해야 한다면 어떻게 할까?

매번 직접 치는 대신, 파일에 저장해서 실행할 수 있다.

./check-log.sh app.log

핵심 질문은 이것이다.

여러 shell 명령어를 하나의 실행 가능한 프로그램처럼 만들려면 어떻게 해야 하는가?

그 답이 shell script다.

## 개념

Shell script란?

Shell script는 shell 명령어들을 파일에 적어둔 것이다.

예:

#!/usr/bin/env bash

echo "Hello, shell script"

이 파일을 hello.sh로 저장하고 실행 권한을 주면:

chmod +x hello.sh
./hello.sh

Hello, shell script

즉 shell script는 터미널에서 직접 치던 명령어들을 파일로 저장한 것이다.

## 왜 필요할까

Shell script는 반복 작업을 자동화할 때 필요하다.

예를 들어 매번 이렇게 한다고 하자.

make clean
make
./ircserv 6667 password

이걸 매번 손으로 치는 대신:

./run-server.sh

로 만들 수 있다. 또 로그 분석도 마찬가지다.

grep 'ERROR' app.log | sort | uniq -c | sort -nr

이걸 스크립트로 만들면:

./error-summary.sh app.log

처럼 쓸 수 있다.

실무에서 shell script는 이런 용도로 쓴다.

| 용도     | 예시                  |
| ------ | ------------------- |
| 빌드 자동화 | `make && ./program` |
| 로그 분석  | 에러 카운트, IP 빈도 계산    |
| 파일 정리  | 오래된 파일 삭제, 확장자별 분류  |
| 서버 관리  | 프로세스 확인, 재시작        |
| 개발 보조  | 테스트 실행, 포맷팅, 배포 준비  |


## 내부 원리 / 작동 방식

1. 스크립트 실행 방식 1: shell에게 직접 넘기기

bash hello.sh

이 경우 현재 shell이 bash 프로그램을 실행하고, bash가 hello.sh 파일을 읽는다.

현재 shell
   ↓
bash hello.sh
   ↓
hello.sh 안의 명령어 실행

이 방식에서는 파일에 실행 권한이 없어도 된다.

2. 스크립트 실행 방식 2: 직접 실행하기

./hello.sh

이 경우 파일에 실행 권한이 있어야 한다.

chmod +x hello.sh

그리고 파일 첫 줄의 shebang이 중요하다.

#!/usr/bin/env bash

운영체제는 이 줄을 보고 말한다.

이 파일은 bash로 실행해야 하는구나.

## Shebang

### Shebang이란?

스크립트 첫 줄의 이 부분을 shebang이라고 한다.

#!/usr/bin/env bash 또는 #!/bin/bash

의미: 이 스크립트를 어떤 interpreter로 실행할지 지정한다.

### 그렇다면 왜 /usr/bin/env bash를 자주 쓰는가?

두 방식이 있다.

#!/bin/bash

/bin/bash를 직접 실행하라.

하지만 시스템에 따라 bash 위치가 다를 수 있다.

macOS의 기본 bash는 보통 /bin/bash 이지만 Homebrew bash는 예를 들어 /opt/homebrew/bin/bash 에 있을 수 있다.

그래서 더 유연하게 #!/usr/bin/env bash를 쓴다. 뜻은 PATH에서 bash를 찾아 실행하라.

그리고 일반적인 자동화 스크립트는 bash로 쓰는 경우가 많다.

이유는 Linux 서버에서 bash가 널리 쓰이기 때문이다.

## 실행 권한

스크립트를 직접 실행하려면 실행 권한이 필요하다.

chmod +x script.sh

-rwxr-xr-x  1 hyun  staff  50 Apr 27  script.sh

여기서 x가 실행 권한이다.

r = read
w = write
x = execute

## Script arguments: $1, $2, $@

스크립트는 인자를 받을 수 있다.

예를 들어 show.sh:

#!/usr/bin/env bash

echo "first arg: $1"
echo "second arg: $2"

chmod +x show.sh
./show.sh apple banana

출력:
first arg: apple
second arg: banana

| 표현   | 의미                  |
| ---- | ------------------- |
| `$0` | 스크립트 이름             |
| `$1` | 첫 번째 인자             |
| `$2` | 두 번째 인자             |
| `$#` | 인자 개수               |
| `$@` | 전체 인자 목록            |
| `$?` | 직전 명령의 exit code    |
| `$$` | 현재 shell process ID |

단 여기서 $@는 반드시 quote해서 써라

나쁜 예시

for arg in $@; do
  echo "$arg"
done

좋은 예시

for arg in "$@"; do
  echo "$arg"
done

왜냐하면 "$@"는 각각의 인자 경계를 보존한다.

./script.sh "my file.txt" "your file.txt"

"$@"는 다음 두 인자를 그대로 유지한다.

my file.txt
your file.txt

하지만 $@를 quote 없이 쓰면 공백 때문에 깨질 수 있다.

## 인자 검증

스크립트는 인자를 제대로 받았는지 확인해야 한다.

예:

#!/usr/bin/env bash

if [ "$#" -ne 1 ]; then
  echo "Usage: $0 <logfile>" >&2
  exit 1
fi

logfile=$1

grep 'ERROR' "$logfile"

중요한 부분:

if [ "$#" -ne 1 ]; then

뜻은 인자 개수가 1개가 아니면

그리고:

echo "Usage: $0 <logfile>" >&2

>&2는 stderr로 출력하라는 뜻이다.

왜냐하면 사용법 오류는 정상 데이터가 아니라 에러 메시지이기 때문이다.

## 변수

Shell 변수는 이렇게 만든다.

name="hyun"

여기서 주의 할 점은 name = "hyun" 이렇게 shell에서는 = 주변에 공백을 넣으면 안 된다.

사용할 때는 echo "$name" 이런식으로 기본적으로 변수를 사용할 때는 quote한다.

## 함수

반복되는 작업은 함수로 만들 수 있다.

#!/usr/bin/env bash

die() {
  echo "error: $*" >&2
  exit 1
}

[ -f "$1" ] || die "file not found: $1"

grep 'ERROR' "$1"

여기서

die() {
  ...
}

는 함수 정의다.

호출

die "file not found"

### 함수 인자

함수 안에서도 $1, $2, $@를 쓸 수 있다.

say_hello() {
  echo "hello, $1"
}

say_hello "hyun"

## return과 exit

1. exit

exit는 스크립트 전체를 종료한다.

exit 1

스크립트 전체를 실패 상태로 종료

2. return

return은 함수에서 빠져나온다.

check_file() {
  [ -f "$1" ] || return 1
  return 0
}

# Defensive Bash: set -euo pipefail을 언제 쓰고 언제 조심해야 하는가

## 핵심 질문

다음 스크립트를 보자.

#!/usr/bin/env bash

rm -rf "$BUILD_DIR"
mkdir "$BUILD_DIR"
cp "$src" "$BUILD_DIR"

겉보기에는 단순하다. 하지만 위험할 수 있다.
예를 들어 BUILD_DIR 변수가 비어 있다면?

rm -rf "$BUILD_DIR"

이건 상황에 따라 의도와 다른 삭제를 할 수 있다.

또는 cp가 실패했는데도 스크립트가 계속 진행된다면?

빌드는 실패했는데 배포는 계속됨
로그 복사는 실패했는데 성공한 것처럼 보임
테스트 실패 후에도 다음 단계 실행됨

핵심 질문은 이것이다.

* Shell script에서 실수를 빨리 발견하고, 실패를 조용히 무시하지 않게 만들려면 어떻게 해야 하는가?

그 답 중 하나가 Defensive Bash다.

## 개념

Defensive Bash는 말 그대로 방어적으로 shell script를 작성하는 습관이다.

대표적으로 많이 쓰는 설정은 다음 세 가지다.

set -euo pipefail

풀어 쓰면:

set -e
set -u
set -o pipefail

각각의 의미는 다음과 같다.

| 옵션                | 의미                        |
| ----------------- | ------------------------- |
| `set -e`          | 명령이 실패하면 스크립트를 종료하려고 함    |
| `set -u`          | 정의되지 않은 변수를 사용하면 에러       |
| `set -o pipefail` | pipeline 중간 실패도 전체 실패로 취급 |

하지만 이 옵션들은 “무조건 켜면 안전해지는 마법”이 아니다.

## 왜 필요할까

Shell script의 기본 동작은 생각보다 관대하다.

예를 들어:

#!/usr/bin/env bash

cp missing.txt backup/
echo "backup complete"

missing.txt가 없어도 스크립트는 다음 줄을 실행할 수 있다.

cp: missing.txt: No such file or directory
backup complete

이건 위험하다.

사람은 backup complete만 보고 성공했다고 착각할 수 있다.

Defensive Bash는 이런 문제를 줄인다.

#!/usr/bin/env bash
set -e

cp missing.txt backup/
echo "backup complete"

이제 cp가 실패하면 스크립트가 중간에 멈춘다.

### set -e: 실패하면 멈추기

set -e 는 명령이 non-zero exit code로 실패하면 스크립트를 종료하려고 한다.

예

#!/usr/bin/env bash
set -e

echo "before"
false
echo "after"

결과: before
false가 exit code 1을 반환하므로 echo "after"는 실행되지 않는다.

왜 유용한가
다음 스크립트를 보자.

#!/usr/bin/env bash

mkdir build
cd build
cmake ..
make

만약 mkdir build가 실패했는데도 계속 진행되면 문제가 생긴다.

mkdir build 실패
cd build 실패
cmake가 엉뚱한 위치에서 실행
make도 엉뚱한 위치에서 실행

set -e를 켜면 실패 지점에서 멈출 가능성이 높다.

#!/usr/bin/env bash
set -e

mkdir build
cd build
cmake ..
make

* 하지만 set -e는 까다롭다

set -e는 모든 실패에서 항상 직관적으로 동작하지 않는다.

예를 들어 조건문 안에서는 실패가 정상적인 판단 재료일 수 있다.

if grep 'ERROR' app.log; then
  echo "error found"
else
  echo "no error"
fi

여기서 grep은 매칭이 없으면 exit code 1을 반환한다.

그런데 이건 “스크립트 실패”가 아니라 “ERROR가 없음”이라는 조건 결과다.

그래서 set -e는 if, while, &&, || 같은 문맥에서는 다르게 동작한다.

핵심:
실패가 정말 fatal error인지,
아니면 조건 판단용 non-zero인지 구분해야 한다.

### set -u: 정의되지 않은 변수 금지

set -u 또는 set -o nounset은 정의되지 않은 변수를 사용하면 에러를 낸다.

예:

#!/usr/bin/env bash
set -u

echo "$USERNAME"

만약 USERNAME 변수가 정의되어 있지 않다면
USERNAME: unbound variable
같은 에러가 난다.

### 왜 유용한가

오타를 빨리 잡을 수 있다.

#!/usr/bin/env bash

logfile="app.log"

grep ERROR "$logifle"

여기서 logifle은 오타다.

기본 bash에서는 "$logifle"이 빈 문자열로 취급될 수 있다.

그러면 에러가 이상하게 나온다.

grep: : No such file or directory

하지만 set -u를 켜면 바로 잡힌다.

#!/usr/bin/env bash
set -u

logfile="app.log"

grep ERROR "$logifle"

결과:
logifle: unbound variable

### 기본값을 주는 방법

set -u를 쓰면 선택적 변수를 다룰 때 조심해야 한다.

예:
echo "$1"

인자 없이 실행하면 $1이 없어서 에러가 날 수 있다.

그래서 기본값을 줄 수 있다.

name=${1:-guest}
echo "$name"

$1이 있으면 그 값을 사용
없거나 비어 있으면 guest 사용

./hello.sh

guest

./hello.sh hyun

hyun

## set -o pipefail: pipeline 실패 감지

grep ERROR missing.log | wc -l
echo $?

missing.log가 없으면 grep은 실패한다.

하지만 wc -l은 빈 입력을 받아도 정상적으로 실행될 수 있다.

그래서 pipeline 전체 exit code가 0처럼 보일 수 있다.

기본적으로 pipeline의 exit code는 마지막 명령 기준이다.

grep ERROR missing.log | wc -l
        실패          성공

전체는 마지막 wc 기준으로 성공처럼 보일 수 있음

### pipefail 사용하자.

set -o pipefail 을 켜면 pipeline 중간 실패가 전체 실패에 반영된다.

#!/usr/bin/env bash
set -o pipefail

grep ERROR missing.log | wc -l
echo "done"

여기에 set -e까지 같이 있으면:

#!/usr/bin/env bash
set -eo pipefail

grep ERROR missing.log | wc -l
echo "done"

이렇게 하면 grep 실패 때문에 스크립트가 멈출 수 있다.

## 하지만 이 설정들로부터 무조건 정답을 얻는 것은 아니다.

### 경우 1. grep에서 매칭 없음은 실패인가?

set -e

grep 'ERROR' app.log
echo "done"

ERROR가 없으면 grep은 exit code 1을 반환한다.

set -e가 켜져 있으면 스크립트가 멈출 수 있다.

그런데 “ERROR가 없음”은 오히려 정상일 수도 있다.

이럴 때는 의도를 명시한다.

if grep 'ERROR' app.log; then
  echo "error found"
else
  echo "no error"
fi

grep 'ERROR' app.log || true

하지만 || true는 실패를 숨기므로 남용하면 안 된다.

### 경우 2. optional file

set -e

source .env

.env가 없을 수도 있는 구조라면 스크립트가 바로 죽는다.

의도를 명시해야 한다.

if [ -f .env ]; then
  source .env
fi

### 경우 3. unset variable

set -u

echo "mode=$MODE"

MODE가 선택적 환경변수라면 에러가 난다.

기본값을 주는 게 낫다.

mode=${MODE:-dev}
echo "mode=$mode"

## trap: 실패 시 정리 작업

Defensive Bash에서는 trap도 자주 쓴다.

임시 디렉토리를 만들고, 스크립트가 끝날 때 지우고 싶다고 하자.

#!/usr/bin/env bash
set -euo pipefail

tmpdir=$(mktemp -d)

cleanup() {
  rm -rf "$tmpdir"
}

trap cleanup EXIT

echo "temp dir: $tmpdir"

이 의미는 스크립트가 정상 종료되든 실패 종료되든 EXIT 시 cleanup 실행한다는 것이다.

실무에서 임시 파일, lock file, 테스트 산출물 정리에 유용하다.

주의: rm -rf "$tmpdir" 같은 코드는 변수가 비어 있지 않도록 조심해야 한다. mktemp -d 결과처럼 신뢰할 수 있는 값에 쓰는 것이 좋다.
