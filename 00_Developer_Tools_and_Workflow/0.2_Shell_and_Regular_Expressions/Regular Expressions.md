# Regex는 무엇인가

앞서서 간단하게 Regex를 설명하였다. 하지만 조금 부족한거 같다는 생각에 확실히 공부를 해보고자 지금 이 문서를 만들게 되었다.

## 핵심 질문

* Regex는 “문자열을 찾는 문법”인가, 아니면 “문자열을 읽는 작은 프로그램”인가?

regex를 보통 이렇게 생각한다.

.*\.txt$

“아, .txt로 끝나는 걸 찾는 거구나.” 라고 생각할 수 있다. 하지만 좀 더 파고들어서 이렇게 생각해보자.

“문자열의 왼쪽부터 시작해서, 임의의 문자를 가능한 많이 소비하다가, 마지막에 literal . + txt가 오고, 그 뒤가 줄 끝인지 확인하는 패턴이다.”

어디서 들어먹은 바로는 regex를 잘하는 사람은 문법을 외우는 사람이 아니다.
regex engine이 문자열을 어떻게 훑고, 어디서 멈추고, 언제 되돌아가는지 이해하는 사람이다.

## 개념

### Regex란?

Regular Expression, 줄여서 regex는 문자열 안에서 특정한 패턴을 찾기 위한 언어다.
예를 들어 이런 문자열이 있다고 하자.

error.log
access.log
main.c
README.md
notes.txt

| 원하는 것           | 예시 regex                           |
| --------------- | ---------------------------------- |
| `.log`로 끝나는 줄   | `\.log$`                           |
| 숫자가 들어간 줄       | `[0-9]`                            |
| `ERROR`로 시작하는 줄 | `^ERROR`                           |
| 이메일처럼 생긴 문자열    | `[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+` |

1. cat

이건 literal matching이다.

즉, 문자 그대로 cat을 찾는다.

하지만 regex는 더 일반적인 규칙을 표현할 수 있다.

2. c.t

이건 다음과 매칭될 수 있다.

cat
cot
cut
c9t
c-t

왜냐하면 regex에서 .은 보통 “아무 문자 하나”라는 의미를 가지기 때문이다.

## 왜 필요한지

regex는 실무에서 거의 매일 쓰인다.

특히 macOS/Linux 터미널에서는 다음 작업에 자주 등장한다.

| 작업           | 예시                                              |              |
| ------------ | ----------------------------------------------- | ------------ |
| 로그 필터링       | `grep '^ERROR' app.log`                         |              |
| 파일 안에서 패턴 검색 | `grep -E '[0-9]{4}-[0-9]{2}-[0-9]{2}' file.txt` |              |
| 문자열 치환       | `sed -E 's/[[:space:]]+/ /g' file.txt`          |              |
| CSV/로그 파싱 보조 | `awk '/^WARN/ { print $0 }' app.log`            |              |
| 코드 검색        | `rg 'TODO                                       | FIXME' src/` |
| 입력 검증        | Python, JavaScript, C++ 등에서 사용                  |              |

ERROR가 포함된 줄만 찾고 싶으면:
grep 'ERROR' app.log

결과는 
ERROR database timeout
ERROR payment failed

이런식으로 나올 것이다. 
여기서 ^는 “줄의 시작”이라는 의미다.
이처럼 regex는 단순 검색보다 훨씬 정밀하다.

## 내부 원리 / 작동 방식

### Regex engine은 문자열을 어떻게 읽는가?

대부분의 regex engine은 기본적으로 문자열을 왼쪽에서 오른쪽으로 읽는다.

예를 들어 다음 regex를 보자.

cat

대상 문자열은 The cat sat.

regex engine은 대략 이렇게 움직인다.

T h e   c a t   s a t .
^

1. T에서 cat이 시작되는지 본다.
2. 아니다.
3. 다음 문자로 이동한다.

T h e   c a t   s a t .
  ^
4. h에서 cat이 시작되는지 본다.
5. 아니다.
6. 계속 이동한다.

T h e   c a t   s a t .
        ^
7. c를 만났다.
8. regex의 첫 글자 c와 맞는다.
9. 다음 문자 a 확인.
10. 다음 문자 t 확인.
11. 전체 cat 매칭 성공.

Pattern: c a t
Text:    T h e   c a t   s a t .
                 c a t
                 ✓ ✓ ✓

## Search, match, full match의 차이

이 차이를 반드시 알아야 한다.

같은 regex라도 도구나 언어에 따라 의미가 달라진다.

대상 문자열: abc123xyz

패턴: [0-9]+

### 1. Search

문자열 어딘가에 패턴이 있으면 성공.

abc123xyz
   ^^^
성공한다.

### 2. Match

문자열의 시작 위치에서 패턴이 맞아야 성공.

abc123xyz
^

패턴 [0-9]+는 숫자로 시작해야 하는데 문자열은 a로 시작한다.

실패한다.

Python의 re.match()가 이런 방식이다.

### 3. Full match

문자열 전체가 패턴과 정확히 맞아야 성공.

abc123xyz
^^^^^^^^^
패턴 [0-9]+는 숫자만 허용한다.
그런데 앞뒤에 문자가 있으므로 실패한다.

Python의 re.fullmatch()가 이런 방식이다.

정리해보면

| 방식         | 의미         | `[0-9]+` vs `abc123xyz` |
| ---------- | ---------- | ----------------------- |
| search     | 어디든 있으면 됨  | 성공                      |
| match      | 시작부터 맞아야 함 | 실패                      |
| full match | 전체가 맞아야 함  | 실패                      |


특히 터미널의 grep은 기본적으로 search에 가깝다.

grep '[0-9]' file.txt

이건 “숫자로만 된 줄”을 찾는 것이 아니다.
숫자가 하나라도 포함된 줄을 찾는다.

숫자로만 된 줄을 찾으려면:

grep -E '^[0-9]+$' file.txt

## shell / glob / regex 차이

이 Lecture에서 가장 중요한 부분이다.

### glob이란?

glob은 shell이 파일 이름을 확장할 때 쓰는 패턴이다.

ls *.txt

여기서 *.txt는 regex가 아니다.

이건 shell glob이다.

shell이 먼저 현재 디렉토리의 파일 이름들을 보고:

a.txt
b.txt
main.c
README.md

*.txt에 맞는 파일을 찾아서 명령어를 이렇게 바꾼다.

ls a.txt b.txt

즉, 실제로 ls 프로그램이 받는 인자는 *.txt가 아니다.
이미 shell이 확장한 결과다.

### regex는 누가 해석하는가?

regex는 shell이 해석하는 것이 아니다.

예:

grep '.*\.txt$' files.txt

여기서 regex를 해석하는 주체는 grep이다.

하지만 주의해야 한다.

shell이 명령어를 먼저 읽는다.

따라서 shell이 특수하게 해석할 수 있는 문자는 quote로 보호해야 한다.

grep '.*\.txt$' files.txt

single quote '...' 안에 넣으면 shell은 내부 내용을 거의 그대로 grep에게 넘긴다.
즉, 
shell 해석 단계:
' .*\.txt$ ' → .*\.txt$ 를 grep에게 그대로 전달

grep regex 해석 단계:
.*\.txt$ → regex로 해석

### glob과 regex의 근본적 차이

| 목적      | glob           | regex                                 |
| ------- | -------------- | ------------------------------------- |
| 주로 쓰는 곳 | shell 파일 이름 확장 | grep/sed/awk/언어 문자열 검색                |
| `*` 의미  | 아무 문자열         | 앞 요소의 0회 이상 반복                        |
| `?` 의미  | 아무 문자 하나       | 앞 요소의 0회 또는 1회                        |
| `.` 의미  | 그냥 점           | 아무 문자 하나                              |
| 누가 해석?  | shell          | grep/sed/awk/Python/JS 등 regex engine |
| 대상      | 보통 파일 이름       | 일반 문자열                                |


### grep / sed / awk / 언어별 regex 차이

regex는 “하나의 절대 표준 문법”이라고 생각하면 안 된다.

도구마다 조금씩 다르다.

#### grep

macOS/Linux에서 기본 grep은 보통 Basic Regular Expression, 즉 BRE를 쓴다.

grep 'a\+' file.txt

BRE에서는 +를 특별한 반복자로 쓰려면 escape가 필요할 수 있다.

하지만 grep -E를 쓰면 Extended Regular Expression, ERE가 된다.

grep -E 'a+' file.txt

ERE에서는 +가 “하나 이상 반복”이라는 뜻이다.

실무에서는 복잡한 regex를 쓸 때 보통 grep -E를 많이 쓴다.

#### sed

sed도 기본적으로 BRE를 많이 쓴다.

sed 's/foo/bar/g' file.txt

이건 foo를 bar로 바꾼다.

그룹을 사용할 때 BRE에서는 보통 이렇게 쓴다.

sed 's/\([0-9][0-9]\)-\([0-9][0-9]\)/\2-\1/'

ERE를 쓰면 더 읽기 쉬워진다.

macOS/BSD sed:

sed -E 's/([0-9][0-9])-([0-9][0-9])/\2-\1/'

GNU sed에서도 보통 -E가 된다.

#### awk

awk는 패턴 매칭 언어이기도 하다.

awk '/^ERROR/ { print $0 }' app.log

이건 ERROR로 시작하는 줄만 출력한다.

awk의 regex는 보통 ERE 계열로 생각하면 된다.

#### ripgrep

rg는 현대적인 코드 검색 도구다.

rg 'TODO|FIXME'

빠르고 사용성이 좋다.

다만 PCRE 고급 기능을 항상 기본으로 쓰는 것은 아니다.
필요할 때 옵션을 켜야 하는 경우가 있다.

rg -P '\d+'

-P는 PCRE2 regex를 사용한다.

#### Python

Python에서는 re 모듈을 쓴다.

import re

re.search(r'\.txt$', 'notes.txt')

Python에서는 raw string을 자주 쓴다.

r'\.txt$'

이렇게 하면 Python 문자열 escape와 regex escape가 덜 충돌한다.

#### JavaScript

JavaScript에서는 보통 이렇게 쓴다.

/\.txt$/.test("notes.txt")

또는 문자열로:

new RegExp("\\.txt$")

문자열로 만들 때는 backslash를 한 번 더 escape해야 해서 헷갈릴 수 있다.

## Shell quote와 regex escape의 차이

두 개의 해석기가 있다

grep '\.txt$' files.txt

1단계: shell이 명령어를 해석한다.
2단계: grep이 regex를 해석한다.

사용자 입력
  |
  v
shell 해석
  - quote 제거
  - variable expansion
  - glob expansion
  - command substitution
  |
  v
grep 실행
  |
  v
regex engine 해석
  - \. = literal dot
  - $ = end of line

### 보통 single quoto를 사용하는데 이때 double quote를 쓰면?

grep "\.txt$" files.txt

대부분의 경우 이것도 동작할 수 있다.

하지만 double quote 안에서는 shell이 여전히 일부 해석을 한다.

echo "$HOME"

$HOME이 변수로 확장된다.

따라서 regex 안에 $, \, backtick, ! 등이 들어가면 상황에 따라 예상과 다를 수 있다.

그래서 regex를 shell에 넘길 때는 기본적으로:

'regex_here'

single quote를 권장한다.

# Regex 기본 메타문자

## 핵심 질문

이번 강의의 핵심 질문은 이것이다.

* regex에서 문자는 언제 “그 문자 그대로”이고, 언제 “특별한 명령”이 되는가?

예를 들어:

a

이건 문자 그대로 a를 찾는다.

그런데:

.

이건 점을 찾는 것이 아니다.
regex에서는 보통 아무 문자 하나를 의미한다.

그래서 regex를 읽을 때는 항상 이렇게 구분해야 한다.

| 종류                | 의미                           |
| ----------------- | ---------------------------- |
| literal character | 문자 그대로 찾는 문자                 |
| metacharacter     | regex engine에게 특별한 명령을 주는 문자 |

## 개념 설명

Literal matching

가장 단순한 regex는 문자 그대로 찾는 것이다.

cat

대상 문자열

The cat is sleeping.

매칭:

The cat is sleeping.
    ^^^

cat에는 특별한 metacharacter가 없다.
따라서 regex engine은 c, a, t를 순서대로 찾는다.

## Metacharacter

metacharacter는 regex engine에게 특별한 의미를 가진 문자다.

| 문자       | 의미                             |                 |
| -------- | ------------------------------ | --------------- |
| `.`      | 아무 문자 하나                       |                 |
| `[]`     | 문자 집합                          |                 |
| `[^...]` | 제외 문자 집합                       |                 |
| `\`      | escape                         |                 |
| `^`      | 줄 시작 또는 character class 안에서 부정 |                 |
| `$`      | 줄 끝                            |                 |
| `*`      | 앞 요소 0회 이상 반복                  |                 |
| `+`      | 앞 요소 1회 이상 반복, ERE/PCRE 기준     |                 |
| `?`      | 앞 요소 0회 또는 1회, ERE/PCRE 기준     |                 |
| `{n,m}`  | 반복 횟수, ERE/PCRE 기준             |                 |
| `()`     | 그룹, ERE/PCRE 기준                |                 |
| `        | `                              | 또는, ERE/PCRE 기준 |

## 왜 필요한지

실무에서 regex를 잘못 쓰는 가장 흔한 이유는 이것이다.

* 문자 그대로 찾고 싶은데 metacharacter로 해석되거나,
* 패턴으로 찾고 싶은데 문자 그대로 해석되는 경우.

예를 들어 IP 주소를 찾고 싶다고 하자.

잘못된 패턴:

192.168.0.1

이건 “192.168.0.1”만 찾는 것 같지만, regex에서는 .이 아무 문자 하나다.

따라서 다음도 매칭될 수 있다.

192x168y0z1

정확히 점을 찾으려면:

192\.168\.0\.1

| 작업           | 위험한 regex | 더 정확한 regex |
| ------------ | --------- | ----------- |
| `.txt` 찾기    | `.txt`    | `\.txt`     |
| IP 일부 찾기     | `192.168` | `192\.168`  |
| 숫자 하나 찾기     | `0-9`     | `[0-9]`     |
| 점이 있는 파일명 찾기 | `.`       | `\.`        |


## 내부 원리 / 작동 방식

### .: 아무 문자 하나

c.t

cat
cot
cut
c-t
ct
cart

매칭 결과:

| 문자열    | 매칭 여부 | 이유                     |
| ------ | ----: | ---------------------- |
| `cat`  |     O | `c` + `a` + `t`        |
| `cot`  |     O | `c` + `o` + `t`        |
| `cut`  |     O | `c` + `u` + `t`        |
| `c-t`  |     O | `c` + `-` + `t`        |
| `ct`   |     X | 중간 문자 하나가 없음           |
| `cart` |     X | `c` 다음 한 문자 뒤가 `t`가 아님 |

. 은 보통 줄바꿈 문자 newline은 매칭하지 않는다.

hello
world

대부분의 regex engine에서 .은 기본적으로 \n을 제외한 한 문자를 의미한다.

다만 Python, PCRE, JavaScript 등에서는 옵션에 따라 .이 newline까지 매칭하게 만들 수 있다.

| 환경         | 옵션                  |
| ---------- | ------------------- |
| Python     | `re.DOTALL`         |
| JavaScript | `/s` flag           |
| PCRE       | `(?s)` 또는 DOTALL 옵션 |

grep은 기본적으로 줄 단위로 처리하기 때문에 이 문제를 보통 덜 마주친다.

### []: character class
[]는 “이 중 한 문자”라는 뜻이다.

c[aeiou]t

c + 모음 하나 + t

| 문자열   | 결과 |
| ----- | -: |
| `cat` |  O |
| `cet` |  O |
| `cit` |  O |
| `cot` |  O |
| `cut` |  O |
| `cxt` |  X |

[abc]

는 abc라는 문자열을 의미하지 않는다.

이 뜻이다.

a 또는 b 또는 c 중 문자 하나

abc 전체를 찾으려면:

abc 라고 써야한다.

### 범위 표현 [0-9], [a-z], [A-Z]

숫자 하나:

[0-9]

소문자 하나:

[a-z]

대문자 하나:

[A-Z]

### [^...]: negated character class

[^...]는 “이 안에 없는 문자 하나”라는 뜻이다.

[^0-9]

숫자가 아닌 문자 하나

또 위치에 따라서 의미가 다르기도 하다.

| 위치                                   | 의미         |
| ------------------------------------ | ---------- |
| character class 밖 `^ERROR`           | 줄 시작       |
| character class 안 첫 위치 `[^0-9]`      | 제외, not    |
| character class 안 첫 위치가 아닐 때 `[a^b]` | 문자 그대로 `^` |

### Escape \

Backslash \는 regex에서 두 가지 역할을 한다.

#### 역할 1: metacharacter를 문자 그대로 만들기

\.

문자 그대로 점

#### 역할 2: 일반 문자에 특별한 의미를 부여하기

PCRE/Python/JavaScript 등에서는:

\d

숫자

\w

word character

\s

공백 문자

하지만 POSIX grep/ERE에서는 \d, \w, \s가 표준이 아니다.
macOS/Linux 터미널에서 이식성 있게 쓰려면:

| 의도                | PCRE/Python/JS | POSIX grep -E            |
| ----------------- | -------------- | ------------------------ |
| 숫자                | `\d`           | `[0-9]` 또는 `[[:digit:]]` |
| 알파벳/숫자/underscore | `\w`           | `[[:alnum:]_]`           |
| 공백                | `\s`           | `[[:space:]]`            |

## POSIX Character Classes

POSIX 계열에서는 이런 표현을 많이 쓴다.

[[:digit:]]

주의:

[:digit:]만 쓰는 게 아니다. 반드시 바깥 []가 필요하다.

[[:digit:]]

이 구조를 분해해보면

[
  [:digit:]
]

즉 character class 안에 POSIX character class가 들어간 형태다.

| POSIX class   | 의미        | 대략 비슷한 표현     |
| ------------- | --------- | ------------- |
| `[[:digit:]]` | 숫자        | `[0-9]`       |
| `[[:alpha:]]` | 알파벳       | `[A-Za-z]`    |
| `[[:alnum:]]` | 알파벳 또는 숫자 | `[A-Za-z0-9]` |
| `[[:space:]]` | 공백류       | space, tab 등  |
| `[[:lower:]]` | 소문자       | `[a-z]`       |
| `[[:upper:]]` | 대문자       | `[A-Z]`       |

## locale에 따라 [a-z]가 달라질 수 있는 이유

대부분 사람은 [a-z]를 이렇게 생각한다.
abcdefghijklmnopqrstuvwxyz

그런데 POSIX regex에서 range는 현재 locale의 문자 정렬 규칙, 즉 collation에 영향을 받을 수 있다.
locale이란 시스템의 언어/지역 설정이다.

locale

LANG=en_US.UTF-8
LC_COLLATE=en_US.UTF-8

이런 설정에서는 문자 범위 [a-z]가 단순 ASCII 순서만을 의미하지 않는 경우가 있다.

그래서 아주 엄격하게 ASCII 소문자만 원하면 실무에서 다음처럼 locale을 고정하기도 한다.

LC_ALL=C grep -E '[a-z]' file.txt

LC_ALL=C는 전통적인 byte/ASCII에 가까운 정렬 및 문자 해석을 사용하게 한다.

또는 의미 중심으로 POSIX class를 쓴다.

# Quantifiers

## 핵심 질문

regex에서 “반복”은 어떻게 표현하고, 왜 .* 같은 패턴은 위험해질 수 있는가?

지금까지는 이런 패턴을 봤다.

[0-9]

이건 숫자 한 글자다.

그런데 실무에서는 보통 숫자 한 글자가 아니라:

123
2026
01012345678

처럼 여러 글자를 찾고 싶다.

이때 사용하는 것이 quantifier, 즉 반복자다.

## 개념

Quantifier는 바로 앞에 있는 요소가 몇 번 반복되는지를 지정한다.

대표 문법:

| Quantifier | 의미          | ERE/PCRE 기준       |
| ---------- | ----------- | ----------------- |
| `*`        | 0회 이상       | 없어도 되고 여러 개 있어도 됨 |
| `+`        | 1회 이상       | 하나는 반드시 있어야 함     |
| `?`        | 0회 또는 1회    | 있어도 되고 없어도 됨      |
| `{n}`      | 정확히 n회      | 정확한 반복            |
| `{n,}`     | n회 이상       | 최소 n회             |
| `{n,m}`    | n회 이상 m회 이하 | 범위 반복             |

* quantifier는 “바로 앞 요소”에 붙는다.

## 왜 필요한지

반복자가 없으면 regex는 너무 길어진다.

예를 들어 날짜를 찾는 패턴을 생각해보자.

[0-9][0-9][0-9][0-9]-[0-9][0-9]-[0-9][0-9]

반복자를 쓰면:

[0-9]{4}-[0-9]{2}-[0-9]{2}

훨씬 읽기 쉽다.

전화번호도 마찬가지다.

[0-9]{3}-[0-9]{4}-[0-9]{4}

로그에서 상태 코드도:

[45][0-9]{2}

4 또는 5로 시작하고, 뒤에 숫자 2개

## 내부 원리 / 작동 방식

### *: 0회 이상

a*

a가 0개 이상

매칭가능:

""
"a"
"aa"
"aaa"

* *는 0회도 허용한다.

### +: 1회 이상

a+

a가 1개 이상

a
aa
aaa

### ?: 0회 또는 1회

colou?r

colo + u가 0개 또는 1개 + r

color
colour

즉, optional이다.

### {n}: 정확히 n회

[0-9]{4}

숫자 정확히 4개

[0-9]{4}-[0-9]{2}-[0-9]{2}

### {n,}: n회 이상

[0-9]{2,}

숫자 2개 이상

12
123
123456

### {n,m}: n회 이상 m회 이하

[0-9]{2,4}

숫자 2개 이상 4개 이하

12
123
1234

주의: 대상 문자열이 12345일 때, 이 regex가 완전히 실패한다는 뜻은 아니다.

grep식 search에서는 그 안의 1234까지 매칭할 수 있다.

## Greedy matching

Quantifier는 기본적으로 greedy, 즉 가능한 많이 먹으려 한다.

a+

aaaa

aaaa
^^^^

a+는 가능한 한 많은 a를 소비한다.

.*의 greedy 동작

<.*>

<a> <b> <c>

greedy matching에서는 보통 이렇게 매칭된다.

<a> <b> <c>
^^^^^^^^^^^

1. <가 첫 <와 매칭된다.
2. .*가 가능한 끝까지 먹는다.
3. 마지막 >를 맞춰야 하므로 engine이 뒤에서부터 되돌아오며 맞는 위치를 찾는다.
4. 결국 마지막 >에서 성공한다.

그래서 전체 <a> <b> <c>가 매칭된다.

## Lazy matching

일부 regex engine에는 lazy quantifier가 있다.

| Greedy  | Lazy     |
| ------- | -------- |
| `*`     | `*?`     |
| `+`     | `+?`     |
| `?`     | `??`     |
| `{n,m}` | `{n,m}?` |


.*?는 가능한 적게 먹으려 한다.

하지만 중요한 사실

POSIX ERE에는 lazy quantifier가 없다.

즉, grep -E '<.*?>'는 PCRE/Python/JavaScript에서 기대하는 lazy 동작을 하지 않는다.

## Possessive quantifier 개념

possessive quantifier는 더 고급 개념이다.

a*+
a++
a?+

가능한 많이 먹고, 절대 되돌려주지 않는다.

예를 들어 PCRE 계열에서:

a*+a

aaaa

동작:

1. a*+가 aaaa를 전부 먹는다.
2. 뒤의 a가 필요하다.
3. 하지만 possessive는 되돌려주지 않는다.
4. 실패.

반면 greedy는 되돌려줄 수 있다.

a*a

aaaa

1. a*가 aaaa를 전부 먹는다.
2. 뒤의 a가 필요하다.
3. a*가 마지막 a 하나를 되돌려준다.
4. 뒤의 a가 매칭된다.
5. 성공.

Possessive quantifier는 catastrophic backtracking을 줄이는 데 쓰이기도 한다.
하지만 POSIX ERE에는 없다.

# Anchors

## 핵심 질문

regex는 “어떤 문자”를 찾는 것뿐 아니라, “어떤 위치”를 찾을 수도 있는가?

답은 그렇다.

지금까지 배운 regex는 대부분 “문자”를 찾았다.

[0-9]

숫자 문자 하나

\.

문자 그대로 점

[[:space:]]+

공백 문자 하나 이상

그런데 이번에 배울 anchor는 문자를 소비하지 않는다.
대신 위치를 검사한다.

| Anchor | 의미                   |
| ------ | -------------------- |
| `^`    | 줄/문자열의 시작            |
| `$`    | 줄/문자열의 끝             |
| `\b`   | word boundary        |
| `\B`   | word boundary가 아닌 위치 |

## 개념

Anchor는 “문자”가 아니라 “위치”다

예를 들어:

^ERROR

이 regex는 ERROR라는 글자 앞에 뭔가를 찾는 게 아니다.

줄 시작 위치 + ERROR를 표현한 것이다.

대상을 

ERROR database failed
2026-05-01 ERROR database failed

이거라고 해보자. 그러면 결과는?

ERROR database failed              O
2026-05-01 ERROR database failed   X

두 번째 줄에도 ERROR는 있다.
하지만 줄의 시작에 있지 않다.

## 왜 필요한지

Anchor는 regex를 정확하게 만든다.

예를 들어 숫자를 찾는다고 하자.

[0-9]+

이건 숫자가 들어간 줄을 찾는다.

123
abc123
123abc
abc

grep -E '[0-9]+' 결과:

123
abc123
123abc

하지만 “전체 줄이 숫자로만 된 경우”를 찾고 싶다면 다르다.

^[0-9]+$

줄 시작
숫자 하나 이상
줄 끝

결과는?

123

즉, anchor는 search를 full line match처럼 제한할 때 자주 쓴다.

## 내부 원리 / 작동 방식

### ^: 시작 위치 검사

^ERROR

ERROR failed
INFO ERROR failed

regex engine은 각 줄에서 이렇게 본다.

첫 번째 줄:

ERROR failed
^

1. 현재 위치가 줄 시작인가?
→ 맞다.
2. 그 다음 문자가 E인가?
→ 맞다.
3. R, R, O, R 순서대로 맞는가?
→ 맞다.
4. 성공.

두 번째 줄:

INFO ERROR failed
^

1. 현재 위치가 줄 시작인가?
→ 맞다.
2. 그 다음 문자가 E인가?
→ 아니다. I다.
3. 실패.

그럼 grep search니까 뒤의 ERROR 위치로 이동해서 다시 시도할까?

이 부분이 중요하다.

INFO ERROR failed
     ^

여기서는 ERROR는 맞지만 ^가 실패한다.
왜냐하면 이 위치는 줄 시작이 아니기 때문이다.

따라서 전체 실패다.

### $: 끝 위치 검사

done$

job done
done job
done

job done   O
done job   X
done       O

job done
        ^
        여기

done 다음 위치가 줄 끝이면 성공한다.

### anchor는 문자를 소비하지 않는다

이걸 zero-width assertion이라고도 한다.

^abc$

abc

매칭구조를 살펴보면?

Text:     a b c
Position: 0 1 2 3

^  → position 0 검사
a  → 문자 a 소비
b  → 문자 b 소비
c  → 문자 c 소비
$  → position 3 검사

^와 $는 실제 문자를 먹지 않는다.

그저 “현재 위치가 시작인가?”, “현재 위치가 끝인가?”를 확인한다.

# Groups와 Alternation

## 핵심 질문

regex에서 여러 문자를 하나의 단위로 묶고, 여러 후보 중 하나를 선택하게 하려면 어떻게 하는가?

지금까지는 이런 패턴을 봤다.

[0-9]+

숫자 하나 이상.

^ERROR

ERROR로 시작하는 줄.

그런데 실무에서는 이런 요구가 많다.

ERROR 또는 WARN으로 시작하는 줄을 찾고 싶다.
cat 또는 dog를 찾고 싶다.
날짜 YYYY-MM-DD에서 연/월/일을 따로 뽑고 싶다.
중복된 단어를 찾고 싶다.

이때 필요한 것이:

(...)
|
\1

즉 group, alternation, backreference다.

## 개념

### Group

괄호 (...)는 여러 regex 요소를 하나의 단위로 묶는다.

(cat)

이건 cat을 하나의 그룹으로 묶은 것이다.

그룹은 주로 세 가지 용도로 쓴다.

| 용도        | 예시                                 |       |
| --------- | ---------------------------------- | ----- |
| 반복 대상 묶기  | `(ab)+`                            |       |
| 선택지 범위 지정 | `(cat                              | dog)` |
| 매칭된 부분 저장 | `([0-9]{4})-([0-9]{2})-([0-9]{2})` |       |


### Alternation |

|는 “또는”이다.

cat|dog

cat 또는 dog

printf 'cat\ndog\nbird\n' | grep -E 'cat|dog'

cat
dog

### Capture group

괄호는 기본적으로 capture group이다.

즉, 매칭된 부분을 기억한다.

([0-9]{4})-([0-9]{2})-([0-9]{2})

예시 2026-05-01

|       그룹 | 매칭           |
| -------: | ------------ |
| 전체 match | `2026-05-01` |
|  group 1 | `2026`       |
|  group 2 | `05`         |
|  group 3 | `01`         |


### Non-capturing group

PCRE, Python, JavaScript에서는 이런 문법이 있다.

(?:cat|dog)

이건 그룹이지만 capture하지 않는다.

하지만 중요하다.

POSIX ERE에는 (?:...)가 없다.

따라서 grep -E, sed -E, awk에서는 일반적으로 사용할 수 없다.

## 왜 필요한지

1. 상황 1: cat 또는 dog 찾기

cat|dog

간단하다.

2. 상황 2: cat 또는 dog 뒤에 s 붙이기

cats
dogs

cat|dogs

cat 또는 dogs

우리가 원하는 것은:

(cat|dog)s

(cat|dog)   cat 또는 dog
s           그 뒤에 s

3. 상황 3: 날짜 순서 바꾸기

2026-05-01

05/01/2026

echo '2026-05-01' | sed -E 's/([0-9]{4})-([0-9]{2})-([0-9]{2})/\2\/\3\/\1/'

05/01/2026

| 표현   | 의미      |
| ---- | ------- |
| `\1` | 첫 번째 그룹 |
| `\2` | 두 번째 그룹 |
| `\3` | 세 번째 그룹 |

## 내부 원리 / 작동 방식

### (cat|dog)의 동작

(cat|dog)

dog

regex engine은 대략 이렇게 본다.

Pattern: ( cat | dog )
Text:    d o g

1. 첫 번째 선택지 cat 시도.
2. 현재 문자는 d, cat의 첫 문자는 c.
3. 실패.
4. 다음 선택지 dog 시도.
5. d, o, g 순서대로 성공.
6. 전체 성공.

### Alternation은 왼쪽부터 시도한다

많은 backtracking engine은 alternation을 왼쪽부터 시도한다.

cat|catalog

catalog

일부 엔진에서 search 결과를 보면 cat만 먼저 매칭될 수 있다.

왜냐하면 cat 선택지가 먼저 성공하기 때문이다.

더 긴 것을 먼저 원하면:

catalog|cat

이런 식으로 순서를 조정해야 할 때가 있다.

다만 grep은 줄 단위로 출력하므로 이 차이가 잘 안 보인다.
매칭 부분만 보려면:

printf 'catalog\n' | grep -Eo 'cat|catalog'
printf 'catalog\n' | grep -Eo 'catalog|cat'

결과 비교:

cat
catalog

이런 식의 차이를 볼 수 있다.

### Precedence: |의 우선순위

|는 생각보다 넓게 적용된다.

cat|dog

이건:

cat 또는 dog

그런데:

^cat|dog$

여기서 줄 전체가 cat 또는 dog 이렇게 생각할 수 있다. 하지만.

(^cat) 또는 (dog$)

즉:

| 부분     | 의미       |
| ------ | -------- |
| `^cat` | cat으로 시작 |
| `dog$` | dog로 끝남  |


전체 줄이 cat 또는 dog인지 확인하려면

^(cat|dog)$

### Quantifier와 group

ab+

a + b가 1개 이상

ab
abb
abbb

하지만

(ab)+

ab라는 묶음이 1회 이상

ab
abab
ababab

# Catastrophic Backtracking

## 핵심 질문

왜 어떤 regex는 짧은 문자열에서는 빠른데, 특정 입력에서는 갑자기 말도 안 되게 느려지는가?

이번 Lecture는 Part B의 마지막이자 가장 중요한 내부 원리 강의다.

지금까지 우리는 regex를 이렇게 배웠다.

[0-9]+
^ERROR
(cat|dog)
.*\.txt$

하지만 regex는 단순히 “문법”만의 문제가 아니다.
어떤 regex는 입력에 따라 실행 시간이 급격히 폭발한다.

대표적으로 위험한 패턴:

(a+)+$

aaaaaaaaaaaaaaaaaaaaX

이런 입력에서 일부 regex engine은 엄청난 backtracking을 하게 된다.

## 개념

Catastrophic Backtracking이란?

Catastrophic Backtracking은 regex engine이 가능한 매칭 조합을 너무 많이 시도하다가 실행 시간이 폭발하는 현상이다.

특히 다음 구조에서 자주 발생한다.

반복자 안에 반복자가 있음

(a+)+

(.+)+

(.*a)* 

* 같은 문자열을 여러 방식으로 나눠서 매칭할 수 있을 때, regex engine이 실패 후 모든 조합을 되돌아가며 다시 시도한다.

## 왜 필요한지

실무에서는 regex가 다음 장소에 들어간다.

| 위치                 | 위험                  |
| ------------------ | ------------------- |
| 웹 서버 입력 검증         | 악의적인 입력으로 서버 CPU 소모 |
| 로그 파서              | 대량 로그 처리 지연         |
| API validation     | 요청 처리 시간 폭증         |
| 보안 필터              | 우회 또는 서비스 지연        |
| CI script          | 빌드/검사 시간 증가         |
| editor/code search | 검색이 멈춘 것처럼 보임       |

특히 보안에서는 이 문제를 ReDoS라고 부른다.

Regular Expression Denial of Service

즉, 정규표현식 때문에 서비스 거부 상태가 되는 것이다.

## 내부 원리 / 작동 방식

### Backtracking engine의 기본 사고방식

많은 regex engine은 backtracking 방식으로 작동한다.

a+a

aaaa

동작을 보자.

패턴:

a+  a

a a a a

a+는 greedy라서 가능한 많이 먹는다.

a+가 전부 먹음
Text: a a a a
      ^^^^^^^

그런데 뒤에 a가 하나 더 필요하다.

현재 남은 문자가 없다.

그래서 engine은 생각한다.

“아, a+가 너무 많이 먹었나? 하나 돌려줘 보자.”

그러면:

a+  a
aaa a
^^^ ^

성공한다.

이것이 backtracking이다.

greedy quantifier가 먼저 많이 먹고, 뒤 조건이 실패하면 조금씩 되돌려주는 방식.

## NFA와 DFA의 직관적 차이

regex engine 이야기를 하면 NFA/DFA가 나온다.

정확한 이론은 깊지만, 지금은 직관만 잡으면 된다.

### DFA 느낌

* 현재 상태 하나만 유지하면서 앞으로 진행

| 특징                 | 설명                          |
| ------------------ | --------------------------- |
| 빠름                 | 보통 입력 길이에 선형적               |
| backtracking 폭발 없음 | 가능한 경로를 되돌아가며 시도하지 않음       |
| 기능 제한              | backreference 같은 기능 구현이 어려움 |

### NFA/backtracking 느낌

가능한 경로를 하나 선택해서 가보고, 실패하면 되돌아가 다른 경로 시도

| 특징       | 설명                                |
| -------- | --------------------------------- |
| 표현력 좋음   | backreference, lookaround 등 지원 가능 |
| 구현 직관적   | 많은 언어 regex engine이 이 방식          |
| 느려질 수 있음 | 특정 패턴에서 조합 폭발                     |

---

| 도구/엔진            | 대략적 성향                              |
| ---------------- | ----------------------------------- |
| POSIX grep       | 구현에 따라 다르지만 단순 검색은 빠름               |
| `grep -E`        | POSIX ERE                           |
| ripgrep 기본       | Rust regex 기반, backtracking 폭발 방지 쪽 |
| `rg -P`          | PCRE2, 고급 기능 가능, backtracking 주의    |
| Python `re`      | backtracking engine                 |
| JavaScript regex | backtracking engine                 |
| Perl/PCRE        | backtracking engine                 |

즉, Python/JavaScript/PCRE에서는 catastrophic backtracking을 특히 조심해야 한다.

위험패턴을 하나 살펴보고 결정해보자

## 위험 패턴: (a+)+$

(a+)+$

대상:

aaaaaaaaaaaaX

이 패턴은 이렇게 생겼다.

(
  a+
)+
$

a 하나 이상으로 이루어진 덩어리가 하나 이상 있고,
그 뒤가 문자열 끝이어야 한다.

예를 들어 aaaa도 이렇게 나눌 수 있다.

aaaa
aaaa
aaa + a
aa + aa
aa + a + a
a + aaa
a + aa + a
a + a + aa
a + a + a + a
...

즉, a+ 덩어리들을 여러 방식으로 나눌 수 있다.

그런데 마지막에 X가 있다.

aaaaaaaaaaaaX
            ^

$ 때문에 전체 끝이 바로 와야 하는데 X가 있어서 실패한다.

그러면 engine은 이렇게 한다.

“다른 방식으로 a들을 나눠보면 성공할까?”

그리고 가능한 조합을 엄청나게 시도한다.

## ReDoS 개념

ReDoS는 regex의 backtracking 폭발을 악용해서 시스템 자원을 소모시키는 공격이다.

예를 들어 자바스크립트로 만들어진 서버 코드가 있다고 하자.

const pattern = /^(a+)+$/;

function validate(input) {
  return pattern.test(input);
}

공격자가 이런 입력을 보낸다.

aaaaaaaaaaaaaaaaaaaaaaaaX

그러면 짧아 보이는 문자열인데도 regex 검사 시간이 비정상적으로 길어질 수 있다.

핵심은 거의 맞는 것처럼 보이다가 마지막에서 실패하는 입력이 가장 위험하다는 것이다.
왜냐하면 engine이 “혹시 다른 조합이면 성공할까?” 하고 많은 경로를 시도하기 때문이다.

## 안전한 regex 작성 원칙

1. 원칙 1. .*를 습관적으로 쓰지 마라

나쁜 예

.*ERROR.*

grep에서는 그냥 이것으로 충분하다.

grep 'ERROR' app.log

나쁜 예

<.*>

더 나은 예:

<[^>]*>

2. 원칙 2. 반복자 안에 반복자를 넣지 마라

위험

(a+)+
(.*)+
([A-Za-z]+)*

가능하면 구조를 단순화한다.

^(a+)+$

^a+$

3. 원칙 3. 선택지는 겹치지 않게 만든다

(a|aa)+$

대상: aaaaaaaaaaaaX

a와 aa가 겹친다.
같은 문자열을 여러 방식으로 만들 수 있다.

더 명확하게 표현할 수 있으면 그렇게 한다.

4. 원칙 4. anchor를 적극적으로 사용한다

입력 검증에서는:

[0-9]+

^[0-9]+$

가 명확하다.

하지만 anchor가 있다고 해서 항상 안전한 것은 아니다.

^(a+)+$

이건 anchor가 있어도 위험하다.

anchor는 정확성을 올려주지만, nested quantifier 문제를 자동으로 해결하지는 않는다.

5. 원칙 5. 문자 범위를 좁혀라

".*"

문자열 literal을 찾는다고 할 때, 더 나은 예:

"[^"]*"

CSV처럼 escape 규칙이 있으면 이것도 부족할 수 있지만, 적어도 .*보다는 범위가 좁다.

6. 원칙 6. 복잡한 검증을 regex 하나에 몰아넣지 마라

이메일 검증 예:

^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$

이 정도는 “이메일처럼 생겼는지” 보는 데는 쓸 수 있다.

하지만 완벽한 이메일 검증은 어렵다.

왜냐하면 이메일 주소 문법은 생각보다 복잡하고, 실제 유효성은 DNS, 도메인, 메일 서버, 정책까지 관련된다.

실무에서는 보통:

1. 간단한 형식 검사
2. 길이 제한
3. 도메인 확인
4. 실제 인증 메일 발송

이렇게 나눌 수 있다.

