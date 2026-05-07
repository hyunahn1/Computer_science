# Incident-Style Note 작성

## 1. 핵심 질문

오늘의 핵심 질문은 이것이다.

> “문제를 해결한 뒤, 팀원이 읽고 신뢰할 수 있는 debugging report를 어떻게 작성할 것인가?”

디버깅을 잘하는 사람과 초보자의 차이는 단순히 문제를 빨리 고치는 능력만이 아니다.

실무에서는 다음 능력이 중요하다.

```text
문제를 관찰 가능한 증상으로 설명한다.
추측과 사실을 분리한다.
어떤 명령으로 무엇을 확인했는지 남긴다.
root cause와 fix를 구분한다.
재발 방지책을 구체적으로 적는다.
팀원이 재현할 수 있게 쓴다.
```

이것이 **incident-style note**다.

---

# 2. 개념 설명

## Incident-style note란 무엇인가?

**Incident-style note**는 문제 발생부터 해결까지의 과정을 구조화한 기록이다.

단순한 메모가 아니다.

```text
나쁜 메모:
서버가 안 켜져서 확인해보니 포트 문제였음. 고침.

좋은 incident-style note:
Symptom:
  ./server --port 8080 실행 시 exit code 1로 종료됨.

Hypothesis:
  port 8080이 이미 사용 중일 가능성이 있음.

Command Evidence:
  lsof -i :8080 실행 결과 node process가 LISTEN 상태였음.
  ./server stderr에 "bind: Address already in use" 출력됨.

Root Cause:
  기존 node process가 port 8080을 점유하고 있어 server bind가 실패함.

Fix:
  개발 환경에서 기존 node process 종료 후 server 재실행.

Prevention:
  server startup 실패 시 port number와 recovery hint를 stderr에 출력하도록 개선.
```

핵심은 이것이다.

> incident note는 “내가 뭘 생각했는지”보다 “무엇을 관찰했고, 어떤 증거로 결론을 냈는지”를 기록한다.

---

# 3. 왜 필요한지

## 3.1 문제 해결은 개인 작업이 아니라 팀 자산이다

혼자 공부할 때는 “고쳤다”로 끝날 수 있다.

하지만 팀에서는 다르다.

팀원은 다음을 알고 싶어 한다.

```text
같은 문제가 다시 생기면 어떻게 확인하지?
정말 root cause가 맞나?
임시방편으로 고친 건가, 근본 수정인가?
배포해도 안전한가?
rollback이 필요한 상황인가?
누가 어떤 명령을 실행했나?
```

incident-style note는 이 질문들에 답하는 문서다.

---

## 3.2 디버깅 과정은 시간이 지나면 왜곡된다

사람의 기억은 신뢰하기 어렵다.

디버깅 중에는 특히 더 그렇다.

```text
처음에는 A가 원인 같았다.
중간에 B도 의심했다.
나중에 C가 진짜 원인이었다.
그런데 보고할 때는 “처음부터 C인 줄 알았다”처럼 기억한다.
```

이것을 **hindsight bias**, 즉 사후 확신 편향이라고 볼 수 있다.

incident note는 이 편향을 줄인다.

```text
당시 어떤 가설을 세웠는가?
그 가설은 어떤 증거로 기각되었는가?
최종 root cause는 어떤 증거로 확인되었는가?
```

이 흐름을 남겨야 한다.

---

## 3.3 팀 리스크를 낮춘다

좋은 incident note는 다음 리스크를 줄인다.

| 리스크       | incident note가 줄이는 방식     |
| --------- | ------------------------- |
| 같은 버그 반복  | prevention과 follow-up을 남김 |
| 잘못된 원인 판단 | command evidence로 검증      |
| 팀원 간 오해   | 사실과 추측을 분리                |
| 배포 위험     | fix, rollback, impact를 명시 |
| 지식 손실     | 문제 해결 과정을 문서화             |

디버깅은 기술 문제이기도 하지만, 팀 리스크 관리 문제이기도 하다.

---

# 4. 내부 원리 / 작동 방식

## 4.1 Incident note의 기본 흐름

이번 Part에서 계속 사용할 구조는 이것이다.

```text
Symptom
→ Impact
→ Timeline
→ Hypothesis
→ Command Evidence
→ Root Cause
→ Fix
→ Prevention
→ Follow-up
```

그림으로 보면 다음과 같다.

```text
┌────────────┐
│  Symptom   │  관찰된 문제
└─────┬──────┘
      ▼
┌────────────┐
│  Impact    │  사용자/시스템 영향
└─────┬──────┘
      ▼
┌────────────┐
│  Timeline  │  언제 무엇이 일어났는가
└─────┬──────┘
      ▼
┌────────────┐
│ Hypothesis │  가능한 원인들
└─────┬──────┘
      ▼
┌──────────────────┐
│ Command Evidence │  명령, 출력, exit code
└─────┬────────────┘
      ▼
┌────────────┐
│ Root Cause │  증거로 확인된 원인
└─────┬──────┘
      ▼
┌────────────┐
│    Fix     │  적용한 수정
└─────┬──────┘
      ▼
┌────────────┐
│ Prevention │  재발 방지
└─────┬──────┘
      ▼
┌────────────┐
│ Follow-up  │  추가 작업
└────────────┘
```

중요한 점은 순서다.

**root cause를 먼저 쓰고 증거를 끼워 맞추면 안 된다.**

좋은 note는 이렇게 간다.

```text
관찰 → 가설 → 검증 → 결론
```

나쁜 note는 이렇게 간다.

```text
느낌 → 결론 → 그럴듯한 설명
```

---

# 5. Incident Note의 각 구성 요소

## 5.1 Symptom

**Symptom은 관찰된 증상이다.**

추측을 넣으면 안 된다.

나쁜 예:

```text
DB 연결 문제 때문에 서버가 죽었다.
```

이 문장은 root cause를 이미 단정하고 있다.

좋은 예:

```text
`./api-server` 실행 후 3초 이내에 exit code 1로 종료되었다.
stderr에는 `connection refused`가 출력되었다.
```

좋은 symptom은 다음을 포함한다.

```text
- 어떤 명령 또는 기능에서 발생했는가
- 기대 결과는 무엇이었는가
- 실제 결과는 무엇이었는가
- exit code 또는 HTTP status가 무엇이었는가
- 관련 stderr/log excerpt가 있는가
```

---

## 5.2 Impact

**Impact는 문제가 누구에게 어떤 영향을 주었는지**다.

개발 중 문제라면 이렇게 쓴다.

```text
Impact:
로컬 개발 환경에서 backend server 실행이 실패하여 frontend-backend integration test를 진행할 수 없었다.
production user impact는 없음.
```

production 문제라면 더 구체적으로 써야 한다.

```text
Impact:
2026-05-07 13:10~13:25 UTC 동안 일부 사용자의 login request가 HTTP 500으로 실패했다.
영향 범위는 login endpoint에 한정되며, 데이터 손실 증거는 발견되지 않았다.
```

impact에서 중요한 것은 과장하지 않는 것이다.

나쁜 예:

```text
전체 시스템이 망가졌다.
```

좋은 예:

```text
login endpoint에서 HTTP 500 비율이 증가했다.
다른 endpoint의 실패 증가는 확인되지 않았다.
```

---

## 5.3 Timeline

**Timeline은 시간순 기록이다.**

예:

```text
Timeline:
- 13:05 UTC: v1.8.2 배포 시작.
- 13:10 UTC: login API 500 응답 증가 관찰.
- 13:12 UTC: nginx error log에서 upstream timeout 확인.
- 13:15 UTC: v1.8.1 rollback 결정.
- 13:20 UTC: rollback 완료.
- 13:25 UTC: login API error rate 정상화.
```

timeline은 blame을 찾기 위한 것이 아니다.

timeline의 목적은 다음이다.

```text
문제가 언제 시작되었는가?
어떤 조치가 언제 적용되었는가?
조치 후 증상이 어떻게 변했는가?
```

---

## 5.4 Hypothesis

**Hypothesis는 가능한 원인이다.**

여기서는 확정적으로 쓰지 않는다.

나쁜 예:

```text
원인은 nginx 설정이다.
```

좋은 예:

```text
Hypothesis 1:
nginx upstream timeout 설정이 너무 짧아 login request가 중간에 끊길 가능성이 있다.

Hypothesis 2:
backend server가 DB query에서 지연되어 response를 제때 반환하지 못했을 가능성이 있다.

Hypothesis 3:
최근 frontend 변경으로 login request payload가 잘못 전송되었을 가능성이 있다.
```

좋은 hypothesis는 검증 가능해야 한다.

즉, 다음 질문에 답할 수 있어야 한다.

```text
이 가설이 맞다면 어떤 evidence가 보여야 하는가?
이 가설이 틀렸다면 어떤 evidence가 보여야 하는가?
```

---

## 5.5 Command Evidence

이 부분이 incident note의 핵심이다.

**Command Evidence는 실제 실행한 명령과 결과다.**

최소한 다음을 포함해야 한다.

```text
- command
- stdout excerpt
- stderr excerpt
- exit code
- timestamp
- environment
```

예:

````markdown
### Evidence 1: port check

Command:

```bash
lsof -i :8080
````

stdout:

```text
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
node     4210 dev    22u  IPv4  ...    TCP *:http-alt (LISTEN)
```

stderr:

```text
<empty>
```

exit code:

```text
0
```

Interpretation:

```text
port 8080은 기존 node process가 LISTEN 상태로 점유하고 있었다.
```

````

command evidence에서는 **출력 전체를 무조건 다 넣지 않는다.**

중요한 부분만 excerpt로 넣는다.

단, 원본 로그 파일 경로는 남긴다.

```text
Full log:
debug-bundle-20260507T132000Z/server.stderr.log
````

---

## 5.6 Root Cause

**Root Cause는 증거로 확인된 원인이다.**

root cause는 symptom보다 깊어야 한다.

예를 들어:

```text
Symptom:
server가 실행 직후 종료됨.

얕은 원인:
bind 실패.

더 깊은 root cause:
port 8080을 기존 node process가 점유하고 있었고,
server startup code가 bind 실패 시 명확한 에러 메시지를 남기지 않아
초기 분석이 지연되었다.
```

root cause는 보통 두 층으로 나눌 수 있다.

| 종류             | 의미                         |
| -------------- | -------------------------- |
| Direct cause   | 바로 실패를 일으킨 원인              |
| Systemic cause | 왜 그 문제가 쉽게 발생하거나 늦게 발견되었는가 |

예:

```text
Direct Cause:
port 8080이 이미 사용 중이어서 bind가 실패했다.

Systemic Cause:
server startup script가 preflight port check를 하지 않았고,
bind 실패 메시지에 port number가 포함되지 않아 원인 확인이 지연되었다.
```

전문적인 incident note는 direct cause만 쓰지 않는다.

**prevention을 위해 systemic cause까지 본다.**

---

## 5.7 Fix

**Fix는 실제 적용한 수정이다.**

예:

```text
Fix:
- 기존 node process를 종료했다.
- server를 재실행했다.
- server startup error log에 bind 실패 port를 출력하도록 수정했다.
```

여기서 주의할 점:

```text
fix와 prevention을 구분해야 한다.
```

예를 들어:

```text
Fix:
기존 process 종료.

Prevention:
startup script에 port availability check 추가.
```

기존 process를 종료하는 것은 현재 문제를 해결하는 조치다.

port check 추가는 재발 방지다.

---

## 5.8 Prevention

**Prevention은 재발 방지책이다.**

좋은 prevention은 구체적이어야 한다.

나쁜 예:

```text
다음부터 조심한다.
```

좋은 예:

```text
Prevention:
- server startup 전에 `lsof -i :$PORT`로 port 점유 여부를 확인하는 preflight check 추가.
- bind 실패 시 stderr에 port number와 process check hint 출력.
- README의 local development troubleshooting section에 recovery command 추가.
```

prevention은 보통 다음 범주로 나뉜다.

| 범주       | 예시                                    |
| -------- | ------------------------------------- |
| 코드 개선    | 에러 메시지 개선, validation 추가              |
| 테스트 추가   | regression test, integration test     |
| 운영 절차 개선 | preflight check, deployment checklist |
| 관측성 개선   | log, metric, trace 추가                 |
| 문서화      | runbook, README, incident note 링크     |

---

## 5.9 Follow-up

**Follow-up은 지금 당장 끝내지 못한 추가 작업이다.**

예:

```text
Follow-up:
- [ ] startup error message 개선 PR 생성
- [ ] local dev README에 port conflict troubleshooting 추가
- [ ] CI에서 invalid config case test 추가
```

좋은 follow-up은 담당자와 기한이 있으면 더 좋다.

```text
- [ ] Hyunjun: startup preflight check PR 작성, 2026-05-10까지
```

---

# 6. 쉬운 예시

## 상황

프로그램을 실행했는데 입력 파일이 없어서 실패했다.

```bash
./fake_app.sh missing.txt
```

나쁜 note:

```text
파일 문제였음.
missing.txt 없어서 에러남.
고침.
```

좋은 note:

````markdown
# Incident Note: fake_app input file failure

## Symptom

`./fake_app.sh missing.txt` 실행 시 프로그램이 exit code 1로 종료되었다.

Expected:
프로그램이 입력 파일을 읽고 `processing missing.txt`를 출력해야 했다.

Actual:
stderr에 `error: input file not found: missing.txt`가 출력되었고, stdout은 비어 있었다.

## Impact

로컬 실습 환경에서 실패했다.
production user impact는 없다.

## Timeline

- 2026-05-07 13:20 UTC: `./fake_app.sh missing.txt` 실행.
- 2026-05-07 13:20 UTC: exit code 1 확인.
- 2026-05-07 13:22 UTC: `ls -l missing.txt`로 파일 부재 확인.
- 2026-05-07 13:23 UTC: root cause 확정.

## Hypothesis

입력 파일 `missing.txt`가 존재하지 않아 실패했을 가능성이 있다.

## Command Evidence

### Evidence 1: app execution

Command:

```bash
./fake_app.sh missing.txt
````

stdout:

```text
<empty>
```

stderr:

```text
error: input file not found: missing.txt
```

exit code:

```text
1
```

### Evidence 2: file existence check

Command:

```bash
ls -l missing.txt
```

stdout:

```text
<empty>
```

stderr:

```text
ls: cannot access 'missing.txt': No such file or directory
```

exit code:

```text
2
```

## Root Cause

`missing.txt` 파일이 존재하지 않았고, `fake_app.sh`는 입력 파일이 없을 경우 stderr에 에러 메시지를 출력한 뒤 exit code 1로 종료하도록 작성되어 있었다.

## Fix

올바른 입력 파일을 생성하거나 기존 파일 경로를 인자로 전달한다.

```bash
echo "hello" > input.txt
./fake_app.sh input.txt
```

## Prevention

실행 전에 input file 존재 여부를 확인하는 preflight check를 추가한다.

```bash
if [[ ! -f "$input" ]]; then
  echo "error: input file not found: $input" >&2
  exit 1
fi
```

## Follow-up

* [ ] README에 input file requirement 추가.
* [ ] missing input case를 테스트에 추가.

````

이 note는 짧지만 구조가 명확하다.

---

# 7. 실무 예시

## 상황: Frontend와 Backend 연결 실패

예를 들어 frontend에서 login 버튼을 눌렀는데 backend API 호출이 실패한다고 하자.

초보자는 이렇게 말한다.

```text
프론트랑 백엔드 연결이 안 되는 것 같아요.
````

이건 너무 넓다.

incident-style note로 바꾸면 다음과 같다.

````markdown
# Incident Note: Frontend login request fails against local backend

## Symptom

Frontend login button 클릭 시 browser console에 API request failure가 표시되었다.

Expected:
`POST /api/login` 요청이 HTTP 200 또는 인증 실패 시 HTTP 401을 반환해야 한다.

Actual:
`POST /api/login` 요청이 HTTP 404를 반환했다.

## Impact

로컬 개발 환경에서 frontend-backend integration test가 막혔다.
production user impact는 없다.

## Timeline

- 14:05 UTC: frontend dev server 실행.
- 14:07 UTC: backend server 실행.
- 14:10 UTC: login button 클릭 시 HTTP 404 확인.
- 14:13 UTC: frontend API base URL 확인.
- 14:18 UTC: backend route list 확인.
- 14:25 UTC: endpoint mismatch 확인.

## Hypothesis

1. Frontend가 잘못된 API path로 요청하고 있을 가능성이 있다.
2. Backend에 login route가 등록되지 않았을 가능성이 있다.
3. Frontend의 API base URL이 잘못 설정되었을 가능성이 있다.

## Command Evidence

### Evidence 1: frontend request

Browser Network tab:

```text
POST http://localhost:8000/api/login
Status: 404 Not Found
````

### Evidence 2: backend route check

Command:

```bash
curl -i http://localhost:8000/api/login
```

stdout excerpt:

```text
HTTP/1.1 404 Not Found
```

stderr:

```text
<empty>
```

exit code:

```text
0
```

Important:
`curl`의 exit code 0은 HTTP 404가 없다는 뜻이 아니다.
HTTP response를 정상적으로 받았다는 뜻이다.

### Evidence 3: backend expected route

Command:

```bash
rg "login" backend/
```

stdout excerpt:

```text
backend/routes/auth.py: @app.post("/auth/login")
```

stderr:

```text
<empty>
```

exit code:

```text
0
```

## Root Cause

Frontend는 `POST /api/login`으로 요청하고 있었지만, backend에 등록된 login endpoint는 `POST /auth/login`이었다.
즉, frontend와 backend 사이의 endpoint path contract가 일치하지 않았다.

## Fix

Frontend API client의 login path를 `/api/login`에서 `/auth/login`으로 수정했다.

## Prevention

* frontend와 backend가 공유하는 API contract 문서를 작성한다.
* login endpoint에 대한 integration test를 추가한다.
* API path를 문자열로 흩뿌리지 않고 central config로 관리한다.

## Follow-up

* [ ] API endpoint table 작성
* [ ] login integration test 추가
* [ ] frontend API client path 상수화

````

여기서 중요한 포인트가 있다.

`curl`은 HTTP 404를 받아도 exit code 0일 수 있다.

왜냐하면 curl 입장에서는 “서버와 통신해서 응답을 받는 데 성공”했기 때문이다.

HTTP status code와 process exit code는 다르다.

---

# 8. 도구 사용 예시

## 8.1 incident note 템플릿 만들기

프로젝트에 다음 파일을 둘 수 있다.

```bash
mkdir -p docs/incidents
cat > docs/incidents/template.md <<'EOF'
# Incident Note: <title>

## Symptom

<관찰된 증상. 추측 금지.>

Expected:

```text
<기대 결과>
````

Actual:

```text
<실제 결과>
```

## Impact

<사용자, 개발, 테스트, 배포에 준 영향>

## Timeline

* <time>: <event>
* <time>: <event>

## Hypothesis

1. <검증 가능한 가설 1>
2. <검증 가능한 가설 2>
3. <검증 가능한 가설 3>

## Command Evidence

### Evidence 1: <name>

Command:

```bash
<command>
```

stdout excerpt:

```text
<stdout excerpt or <empty>>
```

stderr excerpt:

```text
<stderr excerpt or <empty>>
```

exit code:

```text
<exit code>
```

Interpretation:

```text
<이 증거가 무엇을 의미하는지>
```

## Root Cause

<증거로 확인된 실제 원인>

## Fix

<적용한 수정>

## Prevention

<재발 방지책>

## Follow-up

* [ ] <task>
  EOF

````

주의:

```text
template에는 민감정보를 넣지 않는다.
실제 incident note에도 token, password, personal data를 넣지 않는다.
````

---

## 8.2 command evidence를 note에 자동으로 연결하기

Lecture 25에서 만든 debug bundle과 연결하면 좋다.

예:

```text
debug-bundle-20260507T132000Z/
├── environment.txt
├── failure_case.command.txt
├── failure_case.stdout.log
├── failure_case.stderr.log
├── failure_case.exit_code.txt
└── incident_note.md
```

incident note에는 이렇게 적는다.

````markdown
## Command Evidence

Full debug bundle:

```text
debug-bundle-20260507T132000Z/
````

### Evidence 1: failing command

Command file:

```text
failure_case.command.txt
```

stdout:

```text
failure_case.stdout.log
```

stderr:

```text
failure_case.stderr.log
```

exit code:

```text
failure_case.exit_code.txt
```

````

즉, note에는 핵심 excerpt를 넣고, 원본은 bundle에 둔다.

---

## 8.3 로그 excerpt를 넣는 법

로그 전체를 붙여넣으면 note가 망가진다.

나쁜 예:

```text
로그 3,000줄 전체 붙여넣기
````

좋은 예:

````markdown
stderr excerpt:

```text
[2026-05-07T13:10:03Z] ERROR bind failed: Address already in use
[2026-05-07T13:10:03Z] ERROR port=8080
````

Full log:

```text
debug-bundle-20260507T131003Z/server.stderr.log
```

````

좋은 excerpt는 다음 조건을 만족한다.

```text
- 짧다
- timestamp가 있다
- error keyword가 있다
- 원인 판단에 필요한 주변 문맥이 있다
- 원본 파일 위치가 있다
````

---

# 9. 추측과 사실을 구분하는 법

incident note에서 가장 흔한 문제는 추측과 사실이 섞이는 것이다.

## 나쁜 문장

```text
DB가 느려져서 login이 실패했다.
```

이 문장은 사실처럼 보이지만, 실제로는 추측일 수 있다.

## 좋은 문장

```text
Observed:
login request가 HTTP 500을 반환했다.

Hypothesis:
DB query latency 증가로 backend response가 timeout 되었을 가능성이 있다.

Evidence:
현재까지 DB query latency metric은 확인하지 못했다.

Next Step:
DB slow query log와 backend request duration log를 확인한다.
```

즉, 아직 확인하지 못한 것은 root cause에 쓰면 안 된다.

---

## Fact / Hypothesis / Interpretation 구분

| 종류             | 의미        | 예시                                              |
| -------------- | --------- | ----------------------------------------------- |
| Fact           | 관찰된 사실    | `POST /api/login`이 HTTP 404를 반환했다               |
| Hypothesis     | 가능한 설명    | frontend endpoint path가 틀렸을 수 있다                |
| Evidence       | 검증 자료     | `rg "login"` 결과 backend route는 `/auth/login`이었다 |
| Interpretation | 증거 해석     | frontend와 backend endpoint contract가 불일치한다      |
| Root Cause     | 최종 확인된 원인 | frontend가 존재하지 않는 `/api/login`으로 요청했다           |

이 구분을 지켜야 note가 신뢰된다.

---

# 10. 좋은 Incident Note와 나쁜 Incident Note 비교

## 나쁜 note

```markdown
# Login bug

로그인이 안 됐다.
처음에는 백엔드 문제인 줄 알았는데 프론트에서 잘못 보내고 있었다.
고쳤다.
다음부터 조심하자.
```

문제점:

```text
- symptom이 모호하다
- HTTP status가 없다
- 명령 evidence가 없다
- 어떤 파일을 수정했는지 없다
- prevention이 추상적이다
- 팀원이 재현할 수 없다
```

---

## 좋은 note

````markdown
# Incident Note: Login endpoint mismatch in local integration test

## Symptom

Frontend login button 클릭 시 `POST /api/login` 요청이 HTTP 404를 반환했다.

## Impact

로컬 개발 환경에서 login integration test가 실패했다.
production user impact는 없다.

## Hypothesis

Frontend API path와 backend route가 일치하지 않을 가능성이 있다.

## Command Evidence

Command:

```bash
rg "login" backend/ frontend/
````

stdout excerpt:

```text
backend/routes/auth.py: @app.post("/auth/login")
frontend/src/api/auth.ts: path: "/api/login"
```

stderr:

```text
<empty>
```

exit code:

```text
0
```

## Root Cause

Backend login route는 `/auth/login`이지만 frontend API client는 `/api/login`으로 요청하고 있었다.
API endpoint contract가 frontend와 backend 사이에서 일치하지 않았다.

## Fix

`frontend/src/api/auth.ts`의 login path를 `/auth/login`으로 수정했다.

## Prevention

* API endpoint table을 문서화한다.
* login integration test를 추가한다.
* API path를 central config에서 관리한다.

````

좋은 note는 읽는 사람이 바로 판단할 수 있다.

```text
무슨 문제가 있었는가?
어떤 evidence가 있는가?
왜 그게 root cause인가?
어떻게 고쳤는가?
다시 막는 방법은 무엇인가?
````

---

# 11. Postmortem과 Blame-free Culture

## Postmortem이란?

**Postmortem**은 incident가 끝난 뒤 작성하는 회고 문서다.

목표는 다음이다.

```text
누가 잘못했는지 찾는 것 X
시스템이 왜 이 문제를 허용했는지 찾는 것 O
```

예를 들어 나쁜 접근은 이렇다.

```text
누가 API path 잘못 썼어?
```

좋은 접근은 이렇다.

```text
왜 frontend와 backend 사이에 API contract check가 없었는가?
왜 integration test가 endpoint mismatch를 잡지 못했는가?
왜 route 변경이 문서화되지 않았는가?
```

---

## Blame-free culture

**Blame-free culture**는 사람을 비난하지 않고 시스템 개선에 집중하는 문화다.

나쁜 문장:

```text
현준이 endpoint를 잘못 써서 문제가 생겼다.
```

좋은 문장:

```text
Frontend API client와 backend route 사이의 contract를 검증하는 automated test가 없어 endpoint mismatch가 PR 단계에서 발견되지 않았다.
```

이 차이는 중요하다.

사람을 탓하면 팀원들은 문제를 숨긴다.

시스템을 개선하면 같은 문제가 줄어든다.

---

# 12. 흔한 오해

## 오해 1. “Incident note는 production 장애 때만 쓴다”

아니다.

작은 개발 문제에도 쓸 수 있다.

다만 길이는 조절한다.

| 상황            | note 길이     |
| ------------- | ----------- |
| 로컬 실습 문제      | 짧은 note     |
| 팀 개발 버그       | 중간 길이       |
| production 장애 | 자세한 note    |
| 보안/데이터 이슈     | 매우 엄격한 note |

프론트엔드-백엔드 연결 문제, CI 실패, build failure, test flake 같은 것도 incident-style로 기록할 수 있다.

---

## 오해 2. “Root cause는 하나만 있다”

항상 그렇지 않다.

많은 문제는 direct cause와 systemic cause가 함께 있다.

예:

```text
Direct Cause:
frontend가 잘못된 endpoint로 요청했다.

Systemic Cause:
API contract 문서와 integration test가 없었다.
```

direct cause만 고치면 같은 유형의 문제가 반복된다.

systemic cause까지 봐야 prevention이 나온다.

---

## 오해 3. “로그가 있으면 evidence는 충분하다”

아니다.

로그만으로는 부족할 수 있다.

보통 다음이 함께 필요하다.

```text
- 실행 명령
- 환경 정보
- git commit
- config
- stdout/stderr
- exit code
- timestamp
- expected vs actual
```

로그는 evidence의 한 종류일 뿐이다.

---

## 오해 4. “추측도 일단 root cause에 써도 된다”

안 된다.

아직 검증되지 않은 것은 hypothesis에 둬야 한다.

root cause에는 증거로 확인된 것만 쓴다.

```text
Hypothesis:
DB timeout 가능성 있음.

Evidence:
아직 DB log 확인 전.

Root Cause:
작성하면 안 됨.
```

---

# 13. 확인 문제

## 문제 1

다음 문장은 symptom인가, hypothesis인가?

```text
서버가 DB 연결 실패 때문에 종료되었다.
```

정답:

```text
hypothesis 또는 root cause 후보
```

이유:

```text
DB 연결 실패가 확인되기 전이라면 추측이다.
symptom은 “서버가 exit code 1로 종료되었고 stderr에 X가 출력되었다”처럼 관찰된 사실이어야 한다.
```

---

## 문제 2

다음 중 command evidence에 반드시 들어가는 것이 아닌 것은?

```text
A. 실행한 명령
B. stdout/stderr
C. exit code
D. 작성자의 기분
```

정답은 D다.

incident note에는 감정이 아니라 evidence가 들어간다.

---

## 문제 3

다음 prevention 중 더 좋은 것은?

A:

```text
다음부터 더 조심한다.
```

B:

```text
server startup 전에 port availability를 확인하는 preflight check를 추가하고, bind 실패 시 port number를 stderr에 출력한다.
```

정답은 B다.

이유:

```text
구체적이고 실행 가능하며 검증 가능하다.
```

---

## 문제 4

다음 문장을 더 좋은 incident note 문장으로 바꿔라.

```text
프론트가 이상해서 API가 안 됐다.
```

좋은 답안 예:

```text
Frontend login button 클릭 시 `POST /api/login` 요청이 HTTP 404를 반환했다.
Browser Network tab에서 request URL은 `http://localhost:8000/api/login`으로 확인되었다.
```

---

# 14. 실습 과제

## 실습 1. Incident note 템플릿 만들기

안전한 실습 디렉토리에서 진행한다.

```bash
mkdir -p ~/debugging-lab/lecture26
cd ~/debugging-lab/lecture26
```

템플릿 파일을 만든다.

````bash
cat > incident_template.md <<'EOF'
# Incident Note: <title>

## Symptom

Expected:

```text
<expected behavior>
````

Actual:

```text
<actual behavior>
```

## Impact

<who/what was affected?>

## Timeline

* <time>: <event>

## Hypothesis

1. <hypothesis 1>
2. <hypothesis 2>

## Command Evidence

### Evidence 1: <name>

Command:

```bash
<command>
```

stdout excerpt:

```text
<stdout or <empty>>
```

stderr excerpt:

```text
<stderr or <empty>>
```

exit code:

```text
<exit code>
```

Interpretation:

```text
<what this evidence means>
```

## Root Cause

<confirmed cause based on evidence>

## Fix

<what changed?>

## Prevention

<how to prevent recurrence?>

## Follow-up

* [ ] <task>
  EOF

````

---

## 실습 2. 아래 상황을 note로 작성하기

상황:

```text
`./fake_app.sh missing.txt` 실행 시 실패했다.
stderr에는 `error: input file not found: missing.txt`가 출력되었다.
exit code는 1이었다.
`ls -l missing.txt` 실행 결과 파일이 존재하지 않았다.
````

반드시 다음 구조로 작성한다.

```text
symptom
→ hypothesis
→ command evidence
→ root cause
→ fix
→ prevention
```

---

## 실습 3. 나쁜 문장을 좋은 문장으로 바꾸기

다음 문장을 incident-style로 바꿔라.

```text
백엔드가 이상해서 안 됐다.
```

좋은 방향:

```text
어떤 명령 또는 요청이 실패했는가?
HTTP status는 무엇인가?
stderr 또는 log에는 무엇이 있었는가?
exit code는 무엇인가?
```

예상 답안:

```text
`curl -i http://localhost:8000/api/health` 실행 시 HTTP 500 응답을 받았다.
backend stderr에는 `database connection refused`가 출력되었다.
curl process exit code는 0이었다.
```

여기서도 주의해야 한다.

```text
HTTP 500과 curl exit code 0은 동시에 가능하다.
```

---

# 15. 핵심 정리

Lecture 26의 핵심은 다음이다.

```text
incident note는 “내 생각”이 아니라 “검증 가능한 증거의 흐름”이다.
```

반드시 이 구조를 유지한다.

```text
Symptom
→ Hypothesis
→ Command Evidence
→ Root Cause
→ Fix
→ Prevention
```

조금 더 실무적으로는 다음까지 포함한다.

```text
Impact
Timeline
Follow-up
```

가장 중요한 구분은 이것이다.

| 항목               | 의미         |
| ---------------- | ---------- |
| Symptom          | 관찰된 문제     |
| Hypothesis       | 가능한 원인     |
| Command Evidence | 검증 자료      |
| Root Cause       | 증거로 확인된 원인 |
| Fix              | 적용한 수정     |
| Prevention       | 재발 방지책     |

마지막으로, 좋은 incident note는 사람을 비난하지 않는다.

```text
나쁜 방향:
누가 실수했는가?

좋은 방향:
왜 시스템이 이 실수를 막지 못했는가?
어떤 evidence로 확인했는가?
다음에는 어떻게 자동으로 막을 것인가?
```