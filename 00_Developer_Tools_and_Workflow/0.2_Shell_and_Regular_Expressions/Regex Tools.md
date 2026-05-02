# Regex Tools

Regex를 사용할 수 있는 툴을 한번 이야기 해보고자 한다.

# grep

## 핵심 질문

grep pattern file을 실행하면 정확히 무슨 일이 일어나는가?

더 구체적으로는 다음을 이해해야 한다.

grep "ERROR" app.log

이 명령에서

| 부분        | 누가 해석하는가                      |
| --------- | ----------------------------- |
| `grep`    | shell이 실행할 프로그램 이름으로 인식       |
| `"ERROR"` | shell이 quote를 제거한 뒤 grep에게 전달 |
| `app.log` | shell이 파일 이름 인자로 grep에게 전달    |
| `ERROR`   | grep이 regex 또는 문자열 패턴으로 해석    |
| 파일 내용     | grep이 한 줄씩 읽으면서 검사            |

가장 중요한 것은 shell이 먼저 해석한다. 그 다음 grep이 regex를 해석한다.

## 개념

### grep이란?

grep은 텍스트에서 특정 패턴이 들어 있는 줄을 찾는 프로그램이다. 

이름은 역사적으로 다음에서 왔다.

g/re/p
global / regular expression / print

전체 텍스트에서 regex에 맞는 줄을 찾아 출력한다.

기본 모델의 구동은 이런식이다.

입력 텍스트
   ↓
한 줄씩 읽음
   ↓
각 줄에 pattern 적용
   ↓
매칭되는 줄을 stdout으로 출력
   ↓
결과에 따라 exit code 반환

### grep의 기본 형태

grep PATTERN FILE

예시:

grep "ERROR" server.log

server.log 파일을 읽어서
ERROR라는 패턴이 포함된 줄을 출력하라.

### grep은 “줄 단위” 도구다

기본적으로 grep은 파일 전체를 하나의 문자열로 보는 게 아니라 한 줄씩 본다.

예를 들어 파일이 이렇게 있다고 하자.

INFO server started
WARN disk almost full
ERROR database connection failed
INFO retrying

grep "ERROR" app.log

ERROR database connection failed

grep의 기본 출력 단위는 매칭된 부분만이 아니라 매칭된 줄 전체이다.

## 왜 필요한지

grep은 거의 모든 터미널 작업의 기본 도구이다.

로그 분석
grep "ERROR" app.log

코드 검색
grep -r "TODO" src/

프로세스 필터링
ps aux | grep nginx

설정 파일 확인
grep "Port" /etc/ssh/sshd_config

## 내부 원리 / 작동 방식

grep pattern file

grep "ERROR" app.log

흐름:

[1] shell이 명령어를 파싱한다.
[2] shell이 "ERROR"에서 quote를 제거한다.
[3] shell이 grep 프로그램을 실행한다.
[4] grep은 argv를 받는다.

    argv[0] = "grep"
    argv[1] = "ERROR"
    argv[2] = "app.log"

[5] grep이 app.log를 연다.
[6] 파일을 한 줄씩 읽는다.
[7] 각 줄에 pattern을 적용한다.
[8] 매칭된 줄을 stdout으로 출력한다.
[9] exit code를 반환한다.

### grep pattern < file

grep "ERROR" < app.log

이건 겉보기에는 비슷하지만 내부적으로 다르다.

shell이 app.log를 열고
그 파일을 grep의 stdin으로 연결한다.

즉,

grep "ERROR" app.log

에서는 grep이 직접 파일을 연다.

grep "ERROR" < app.log

에서는 shell이 파일을 열고, grep은 stdin에서 읽는다.

출력은 보통 같다. 하지만 차이가 있다.

* 파일명을 출력해야 하는 상황

grep "ERROR" app.log

grep은 자신이 읽는 파일 이름을 압니다.

반면:

grep "ERROR" < app.log

grep 입장에서는 그냥 stdin입니다. 파일 이름을 모른다.

### cmd | grep pattern

dmesg | grep "usb"

흐름:

dmesg의 stdout
   ↓ pipe
grep의 stdin
   ↓
매칭된 줄만 stdout

[dmesg] --stdout--> pipe --stdin--> [grep "usb"] --stdout--> terminal

some_command | grep "ERROR"

이 경우 some_command의 stdout만 grep으로 간다.

stderr까지 grep하고 싶으면:

some_command 2>&1 | grep "ERROR"

### 그냥 grep만 치면?

이렇게 하면 보통 멈춘 것처럼 보인다. 하지만 멈춘 게 아니라 grep이 stdin 입력을 기다리는 중이다.

grep "hello"

이제 터미널에서 직접 입력한다.

hello world

그러면 똑같이 출력은 hello world 이렇게 나온다.

## grep, grep -E, grep -F

### 기본 grep

grep PATTERN file

기본 grep은 보통 BRE, 즉 Basic Regular Expression을 사용한다.

BRE에서는 몇몇 metacharacter가 직관과 다르게 동작한다.

### grep -E

grep -E PATTERN file

-E는 Extended Regular Expression이다.

실무에서는 복잡한 regex를 쓸 때 grep -E를 많이 쓴다.

grep -E "ERROR|WARN" app.log

### grep -F

grep -F PATTERN file

-F는 Fixed String입니다.

즉 regex가 아닙니다.

예:

grep -F "user.name@example.com" file.txt

여기서 .은 “아무 문자”가 아니라 진짜 점이다.

* 세가지의 차이

| 명령        | 패턴 해석 방식       | 예시                |
| --------- | -------------- | ----------------- |
| `grep`    | BRE            | 오래된 기본 regex      |
| `grep -E` | ERE            | 현대적으로 쓰기 편한 regex |
| `grep -F` | literal string | regex 해석 없음       |


## grep exit code

| exit code | 의미    |
| --------: | ----- |
|       `0` | 매칭 있음 |
|       `1` | 매칭 없음 |
|       `2` | 에러 발생 |


# ripgrep

## 핵심 질문

ripgrep, 즉 rg는 단순히 “빠른 grep”인가?

반은 맞고, 반은 틀리다.

ripgrep은 기본적으로 다음을 수행하는 도구이다.

rg PATTERN

현재 디렉토리 아래를 재귀적으로 돌면서
PATTERN에 맞는 줄을 찾는다.
단, .gitignore, hidden file, binary file 등을 기본적으로 고려한다.

즉 grep과 가장 큰 차이는 이것이다.

| 도구        | 기본 모델                     |
| --------- | ------------------------- |
| `grep`    | 주어진 파일 또는 stdin에서 검색      |
| `rg`      | 현재 디렉토리 전체를 재귀적으로 검색      |
| `grep -r` | 디렉토리 재귀 검색 가능하지만 필터링이 단순함 |
| `rg`      | 코드베이스 검색을 기본값으로 설계        |


공식 설명에서도 ripgrep은 “line-oriented search tool”이며, 현재 디렉토리를 재귀 검색하고 .gitignore 규칙을 기본적으로 존중한다고 설명한다. 또한 hidden file과 binary file도 기본적으로 건너뛴다.

## 개념

### ripgrep이란 무엇인가

ripgrep의 실행 파일 이름은 보통 rg이다.

rg 'ERROR'

이 명령은 현재 디렉토리부터 시작해서 하위 디렉토리까지 검색한다.

즉 grep에서는 보통 이렇게 해야 한다.

grep -rn 'ERROR' .

하지만 rg에서는 보통 이렇게 한다.

rg 'ERROR'

grep은 파일을 지정하는 느낌이다.
rg는 프로젝트 전체에서 검색하는 느낌이다.

## 기본 출력 형태

예를 들어 이런 파일이 있다고 하자.

src/server.cpp
src/parser.cpp
logs/app.log

rg 'ERROR'

출력 예시:

logs/app.log
3:ERROR database connection failed
src/server.cpp
42:std::cerr << "ERROR: bind failed\n";

터미널이 TTY일 때 rg는 기본적으로 보기 좋게 색상, 줄 번호, 파일 heading 등을 조정한다. 반대로 pipe나 redirect로 연결되면 출력 형식이 달라질 수 있다. Debian manpage도 stdout이 TTY인지에 따라 출력이 달라질 수 있다고 설명한다.

## 왜 필요한지

### grep -r의 실무 문제

프로젝트 루트에서 다음을 실행한다고 하자.

grep -rn 'connect' .

문제:

.git/ 내부까지 뒤질 수 있다.
node_modules/ 같은 거대한 디렉토리를 뒤질 수 있다.
빌드 산출물까지 뒤질 수 있다.
binary file을 건드릴 수 있다.
검색 결과가 너무 많고 느릴 수 있다.

예를 들어 C++ 프로젝트라면 다음 파일까지 검색될 수 있다.

./.git/objects/...
./build/...
./a.out
./compile_commands.json
./third_party/...

반면 rg는 기본적으로 “개발자가 보고 싶어 하는 파일” 위주로 검색하도록 설계되어 있다.

### rg가 실무에서 강한 이유

rg 'Channel'

이 한 줄로 보통 다음이 된다.

현재 프로젝트 전체 검색
.gitignore 반영
숨김 파일 제외
binary 제외
줄 번호 표시
색상 highlight
빠른 검색

그래서 코드베이스 검색에서는 grep -r보다 rg가 훨씬 실용적인 경우가 많다.

## 내부 원리 / 작동 방식

### rg pattern 실행 흐름

rg 'ERROR'

내부 흐름

[1] shell이 명령어를 파싱한다.

    argv[0] = "rg"
    argv[1] = "ERROR"

[2] rg가 현재 디렉토리를 시작점으로 잡는다.

[3] 디렉토리 트리를 순회한다.

[4] .gitignore, .ignore, .rgignore 규칙을 확인한다.

[5] hidden file/directory를 기본적으로 건너뛴다.

[6] binary file을 기본적으로 건너뛴다.

[7] 남은 텍스트 파일을 줄 단위로 읽는다.

[8] regex engine으로 각 줄을 검색한다.

[9] 매칭된 파일명, 줄 번호, 줄 내용을 stdout으로 출력한다.

[10] 결과에 따라 exit code를 반환한다.

도식:

현재 디렉토리
   ↓
directory traversal
   ↓
ignore rule 적용
   ↓
hidden/binary skip
   ↓
텍스트 파일 검색
   ↓
매칭 줄 출력

### rg의 regex engine

rg의 기본 regex engine은 Rust의 regex engine 계열이다.

기본 regex engine은 finite automata 기반이다.
대부분의 검색에서 매우 빠르다.
linear time 검색을 보장한다.
하지만 backreference와 arbitrary look-around는 기본 지원하지 않는다.

Debian manpage도 ripgrep의 기본 regex engine이 finite automata를 사용하고 linear time searching을 보장한다고 설명한다. 
이 때문에 backreference와 임의의 look-around는 기본 엔진에서 지원되지 않는다. PCRE2로 빌드된 경우 -P 또는 --pcre2로 backreference와 look-around를 사용할 수 있다.

## 왜 빠른가?

rg가 빠른 이유는 단순히 “Rust로 만들어져서”가 아니다.

| 이유                   | 설명                              |
| -------------------- | ------------------------------- |
| 빠른 regex engine      | finite automata 기반 검색           |
| smart filtering      | `.gitignore`, hidden, binary 제외 |
| efficient traversal  | 디렉토리 순회를 효율적으로 수행               |
| literal optimization | regex 안의 고정 문자열을 이용해 빠르게 후보를 줄임 |
| parallelism          | 여러 파일을 병렬로 검색 가능                |
| mmap/optimized I/O   | 상황에 따라 효율적인 파일 읽기 사용            |

rg는 “모든 파일을 무식하게 다 빨리 검색”해서만 빠른 게 아니다.
애초에 검색하지 않아도 되는 파일을 많이 제외하기 때문에 빠르다.

## 주요 옵션

### rg pattern

rg 'connect'

현재 디렉토리 아래에서 connect를 검색한다.

grep으로 치면 대략 다음에 가깝다.

grep -rn 'connect' .

하지만 완전히 같지는 않다.

rg는 기본적으로 ignore rule, hidden file, binary file 처리를 한다.

### rg -n

rg -n 'connect'

줄 번호를 표시한다.

사실 터미널 출력에서는 rg가 기본적으로 줄 번호를 보여주는 경우가 많다. 그래도 script나 명시성을 위해 -n을 쓰는 경우가 있다.

### rg -i

rg -i 'error'

대소문자를 무시한다.

ERROR
Error
error
eRrOr

### rg -t

-t는 file type을 지정한다.

rg -t cpp 'Channel'

C++ 관련 파일에서만 검색한다.

rg -t py 'TODO'
rg -t js 'fetch'
rg -t md 'installation'

사용 가능한 type 목록은 다음으로 볼 수 있다.

rg --type-list

### rg --hidden

기본적으로 rg는 hidden file과 hidden directory를 검색하지 않는다.

즉 다음은 기본 검색 대상에서 빠진다.

.env
.github/
.config/
.hidden_file

숨김 파일까지 보고 싶으면:

rg --hidden 'ERROR'

하지만 주의: --hidden을 쓰면 .git/까지 들어갈 수 있다.

그래서 보통은 이런식으로 사용한다.

rg --hidden --glob '!.git/' 'ERROR'

hidden file도 검색한다. 하지만 .git/ 디렉토리는 제외한다.

### rg --glob

--glob은 rg의 검색 대상 파일을 제한하거나 제외하는 옵션이다.

rg 'ERROR' --glob '*.cpp'

*.cpp 파일에서만 ERROR 검색

제외는 !를 쓴다.

rg 'ERROR' --glob '!build/**'

build/ 아래는 제외

복수 사용도 가능하다.

rg 'ERROR' --glob '*.cpp' --glob '*.hpp'

또는 brace 문법도 사용할 수 있다.

rg 'ERROR' --glob '*.{cpp,hpp}'

단, 여기서 quote가 중요하다.

rg 'ERROR' --glob '*.{cpp,hpp}'

### rg -F

rg -F '1.2.3'

grep -F와 같은 의미이다.

regex가 아니라 literal string 검색이다.

### rg -P

rg -P 'PATTERN'

-P는 PCRE2 모드이다.

PCRE2는 Perl Compatible Regular Expression 계열이다.

이 모드에서 가능한 것:

lookahead
lookbehind
backreference
\d, \w 같은 PCRE식 표현

rg -P '\b\w+(?=:)' file.txt

rg의 기본 regex engine과 PCRE2 engine은 다르다.
-P를 쓰면 portable성과 성능 특성이 달라질 수 있다.

또한 모든 rg 빌드가 PCRE2를 지원한다고 가정하면 안 된다. manpage는 PCRE2로 빌드된 경우에 -P/--pcre2를 쓸 수 있다고 설명한다.

확인은 다음으로 한다.

rg --version

# sed

## 핵심 질문

sed 's/foo/bar/' file.txt는 정확히 무엇을 하는가?

sed 's/foo/bar/' file.txt

쉽게 정리하면 이는 

file.txt를 한 줄씩 읽는다.
각 줄에서 foo를 찾아 bar로 바꾼다.
바뀐 결과를 stdout으로 출력한다.
원본 파일은 기본적으로 수정하지 않는다.

이것을 나타낸다.

중요한점은 sed는 기본적으로 “파일을 직접 수정하는 도구”가 아니다. sed는 입력 stream을 읽고, 변환한 결과를 stdout으로 내보내는 도구이다.

그래서 sed의 핵심 모델은 이것이다.

입력
  ↓
한 줄씩 읽음
  ↓
pattern space에 현재 줄 저장
  ↓
sed command 적용
  ↓
결과를 stdout으로 출력

## sed란 무엇인가

sed는 stream editor이다.

stream으로 들어오는 텍스트를 한 줄씩 읽으면서 편집 명령을 적용하는 도구이다.

여기서 stream은 다음 같은 입력을 말한다.

sed 's/foo/bar/' file.txt
cat file.txt | sed 's/foo/bar/'
command | sed 's/foo/bar/'

sed는 vim처럼 파일을 열어서 사용자가 직접 편집하는 도구가 아니다.

vim: interactive editor
sed: non-interactive stream editor

vim은 문서를 사람이 직접 열고 고치는 편집기이다.
sed는 컨베이어 벨트 위로 지나가는 줄들을 자동으로 고치는 기계이다.

## 왜 필요한지

sed는 다음 상황에서 매우 유용하다.

### 단순 치환

sed 's/localhost/127.0.0.1/' config.txt

### 로그에서 prefix 제거

sed 's/^\[[^]]*\] //'

### 특정 줄만 출력

sed -n '10,20p' file.txt

### 특정 줄 삭제

sed '/DEBUG/d' app.log

### pipeline 중간 변환

cat app.log | grep 'ERROR' | sed 's/^/[ERROR LINE] /'

grep이 “찾기” 중심이라면, sed는 “바꾸기/삭제/출력 제어” 중심이다.

## 내부 원리 / 작동 방식

### 기본 처리 흐름

sed는 기본적으로 다음을 반복한다.

[1] 입력에서 한 줄을 읽는다.
[2] 그 줄을 pattern space에 넣는다.
[3] sed script를 적용한다.
[4] pattern space의 내용을 출력한다.
[5] 다음 줄로 넘어간다.

file.txt

line 1 ──> pattern space ──> sed command ──> stdout
line 2 ──> pattern space ──> sed command ──> stdout
line 3 ──> pattern space ──> sed command ──> stdout

### pattern space란 무엇인가

pattern space는 sed가 현재 처리 중인 줄을 담아두는 임시 공간이다.

예를 들어 파일이 이렇다고 하자.

foo one
foo two
bar three

sed 's/foo/hello/' file.txt

1번째 줄:
pattern space = "foo one"
s/foo/hello/ 적용
pattern space = "hello one"
출력

2번째 줄:
pattern space = "foo two"
s/foo/hello/ 적용
pattern space = "hello two"
출력

3번째 줄:
pattern space = "bar three"
s/foo/hello/ 적용
변화 없음
pattern space = "bar three"
출력

## 기본 치환: s/pattern/replacement/

### 가장 기본 형태

sed 's/foo/bar/' file.txt

구조:

s / pattern / replacement /

| 부분    | 의미                |
| ----- | ----------------- |
| `s`   | substitute, 치환 명령 |
| `foo` | 찾을 패턴             |
| `bar` | 바꿀 문자열            |
| `/`   | delimiter, 구분자    |


각 줄에서 첫 번째 foo를 bar로 바꾼다.

### 중요한 점: 한 줄에서 첫 번째 매칭만 바꿈

cat > words.txt <<'EOF'
foo foo foo
bar foo baz
EOF

이때 명령을 내리면

sed 's/foo/XXX/' words.txt

출력:

XXX foo foo
bar XXX baz

각 줄에서 첫 번째 foo만 바뀐다.

### 모든 매칭 바꾸기: g

sed 's/foo/XXX/g' words.txt

XXX XXX XXX
bar XXX baz

여기서 g는 global flag이다.

현재 줄 안에서 모든 매칭을 바꾼다.

g는 파일 전체 global이 아니다.
현재 줄 안에서 global이다.

### delimiter 바꾸기

기본 delimiter는 /이다.

sed 's/foo/bar/'

그런데 path를 바꿀 때는 문제가 생긴다.

sed 's//usr/local/bin//opt/bin/'

이건 읽기 어렵고 깨지기 쉽다.

그래서 delimiter를 바꿀 수 있다.

sed 's#/usr/local/bin#/opt/bin#' file.txt

또는:

sed 's|/usr/local/bin|/opt/bin|' file.txt

즉 s 다음 문자가 delimiter가 된다.

s/pattern/replacement/
s#pattern#replacement#
s|pattern|replacement|
s:pattern:replacement:

| 바꾸는 내용 | 추천 delimiter |   |
| ------ | ------------ | - |
| 일반 단어  | `/`          |   |
| 파일 경로  | `#` 또는 `     | ` |
| URL    | `#` 또는 `     | ` |

## -n과 p

### 기본적으로 sed는 자동 출력한다

sed 's/foo/bar/' file.txt

sed는 각 줄을 처리한 뒤 자동으로 출력한다.

즉 내부적으로는 다음 느낌이다.

read line
apply command
print pattern space

### p는 print 명령이다

sed 'p' file.txt

이렇게 하면 어떻게 될까?

각 줄이 두 번 나온다.

왜냐하면 p 명령으로 한 번 출력
sed의 기본 자동 출력으로 또 한 번 출력

## -n은 자동 출력을 끈다

sed -n 'p' file.txt

이제 각 줄이 한 번만 나온다.

-n: 자동 출력 끄기
p: 내가 원하는 줄만 직접 출력하기

실무에서 -n과 p는 거의 세트로 쓴다.

sed -n '10,20p' file.txt

10번째 줄부터 20번째 줄까지만 출력한다.

## address와 address range

sed 명령 앞에는 “어느 줄에 적용할지”를 붙일 수 있다.

이걸 address라고 한다.

### 특정 줄 번호

sed -n '3p' file.txt

3번째 줄만 출력한다.

### 줄 범위

sed -n '3,5p' file.txt

3번째 줄부터 5번째 줄까지 출력한다.

마지막 줄: $

sed -n '$p' file.txt

마지막 줄만 출력한다.

여기서 $는 sed가 해석하는 마지막 줄 기호이다.
shell 변수 표시가 아니다.

그래서 single quote로 감싼다.

sed -n '$p' file.txt

나쁜 예

sed -n "$p" file.txt

"$p"를 shell 변수로 해석하려 할 수 있다. 

### regex address

특정 패턴이 있는 줄에만 명령을 적용할 수 있다.

sed -n '/ERROR/p' app.log

ERROR가 들어 있는 줄만 출력한다.

grep 'ERROR' app.log

### regex address range

sed -n '/START/,/END/p' file.txt

START가 나오는 줄부터 END가 나오는 줄까지 출력한다.

before
START
line 1
line 2
END
after

sed -n '/START/,/END/p' file.txt

START
line 1
line 2
END

## 특정 줄만 처리하기

### 3번째 줄만 바꾸기

sed '3s/foo/bar/' file.txt

3번째 줄에서만 foo를 bar로 바꾼다.

### 3~5번째 줄만 바꾸기

sed '3,5s/foo/bar/g' file.txt

3번째 줄부터 5번째 줄까지만 각 줄의 모든 foo를 bar로 바꾼다.

## ERROR 줄에서만 치환하기

sed '/ERROR/s/database/DB/' app.log

ERROR가 들어 있는 줄에서만 database를 DB로 바꾼다.

## 삭제: d

### 특정 줄 삭제

sed '3d' file.txt

3번째 줄을 삭제한 결과를 출력한다.

원본 파일을 직접 지우는 것이 아니다.
stdout으로 삭제된 것처럼 보이는 결과를 출력한다.

### 범위 삭제

sed '3,5d' file.txt

3번째 줄부터 5번째 줄까지 삭제한 결과를 출력한다.

### 패턴이 있는 줄 삭제

sed '/DEBUG/d' app.log

DEBUG가 들어 있는 줄은 출력하지 않는다.

grep -v와 비슷하다.

grep -v 'DEBUG' app.log

하지만 sed는 삭제와 치환을 같이 섞을 수 있다.

## ERE로 쓰기: -E

macOS와 GNU sed에서는 보통 -E로 ERE를 쓸 수 있다.

sed -E 's/^([^=]*)=(.*)$/\1: \2/' config.txt