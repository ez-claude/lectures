# AI 규칙 파일 (Claude Code용)

> 이 파일은 Claude Code가 자동으로 읽습니다. 함께 제공된 state/ 폴더의 파일들과 연동됩니다.
>
> ## 설치 방법
>
> 이 파일을 프로젝트의 `.claude/` 폴더 안에 배치하세요:
> ```
> 내 프로젝트/
> ├── .claude/
> │   └── CLAUDE.md          (이 파일)
> ├── state/
> │   ├── versions.json     (라이브러리/모델 버전 잠금)
> │   ├── patterns.json     (실수 패턴 DB)
> │   └── routing.json      (에이전트+스킬 라우팅)
> ├── src/
> └── ...
> ```
>
> ## 스킬 설치 방법
>
> 규칙 파일을 넣은 후, AI에게 다음과 같이 요청하세요:
> "skills.sh에서 아래 스킬들 전부 설치해줘"
>
> | 카테고리 | 스킬 |
> |---------|------|
> | 작업 분배 | dispatching-parallel-agents, subagent-driven-development |
> | 설계 | architecture-patterns, writing-plans |
> | 개발 | frontend-design, backend-architect, vercel-react-best-practices |
> | 검증 | requesting-code-review, systematic-debugging |
> | 시뮬레이션 | sim (장기 영향 시뮬레이션) |
> | 디자인 | teach-impeccable |
> | 완료 | verification-before-completion, finishing-a-development-branch |
>
> AI가 skills.sh에서 알아서 찾아 설치합니다.
>
> ## Antigravity 사용자
>
> RULES.md 파일을 사용하세요. 프로젝트 최상위에 배치하면 됩니다.

---

## 절대 규칙 (최우선)

1. **이모지 금지** - 텍스트로 대체 ([OK], [FAIL], [Step])
2. **한글 응답** - 모든 출력은 한글
3. **버전 잠금** - state/versions.json의 버전을 우선 사용. 학습 데이터의 구버전 금지
4. **보안 필수** - API 키 환경변수만, 하드코딩 금지
5. **EazyCheck 필수** - Build Mode에서 GATE + 토론(3+파일) + Check 강제 실행
6. **기록 전 검증** - 파일에 사실을 기록할 때, 도구로 확인 가능한 것(날짜, 경로, 버전 등)은 반드시 도구로 확인 후 기록. 컨텍스트/이전 세션 데이터는 현재 시점과 다를 수 있으므로 신뢰 금지.

---

## [EazyCheck] 코드 생성 전후 검증 시스템

> 코드 생성 전 GATE, 코드 생성 후 Check. 파일 수에 따라 토론/sim 자동 추가.

### 전체 흐름

```
[사용자 요청]
    |
[GATE] versions.json + patterns.json Read (세션 내 1회)
    - 관련 패턴 경고 출력
    - 버전 확인
    |
[파일 수 판단]
    |
    +-- [1-2 파일] --> [코드 작성] --> [Check] --> [완료]
    |
    +-- [3+ 파일] --> [토론] --> [코드 작성] --> [sim deep] --> [Check] --> [완료]
```

### GATE -1: Pre-Plan Gate

플랜 품질 검증. Plan → Build 전환 시 자동 실행.
5개 체크: 경쟁분석 / 근본원인 / 품질기준 / 도메인규칙 / 검증방법 → 3개 미만 시 차단

### GATE: 버전 + 패턴 통합 Read (필수!)

```
Read: versions.json + patterns.json
→ [GATE 통과] 버전 확인 + 관련 패턴 경고 출력 후 코드 생성 시작
```

세션 내 1회만 실행. 관련 패턴 발견 시 경고 출력 (차단은 안 함)

---

## [DMAD 토론] 3+ 파일 변경 시 설계 검증 (코드 전)

> 신규 기능/복합 수정 시 토론 없이 코드 작성 금지
> DMAD = 추론 방향이 다른 두 역할이 자문자답하는 방식

### 파일 수 + 내용에 따른 분기

```
1-2 파일              → 바로 코드 작성
3-5 파일 + 신규기능   → DMAD 토론 1라운드 → 코드 → sim
3-5 파일 + 설정/배포  → 토론 스킵 → 코드 → sim (토큰 절감)
6+ 파일               → DMAD 토론 2라운드 → 코드 → sim
```

### DMAD 역할 (자문자답)

```
[설계자 - 순방향] 기술적 최선 추구
  Q1. 더 단순한 방법이 있나?
  Q2. 기존 코드 어디와 부딪히나?
  Q3. 가장 위험한 가정 하나는?
  Q4. 기존 코드베이스에 중복되는 로직이 있나?

[사용자 - 역방향] 실제 사용 경험 관점
  Q1. 처음 보는 사람이 이 기능을 찾고 쓸 수 있나?
  Q2. 기대하는 가장 자연스러운 방식으로 동작하나?
  Q3. 실수하거나 실패했을 때 스스로 해결할 수 있나?
  Q4. 실패했을 때 사용자가 알 수 있나? 에러가 조용히 삼켜지는 곳은 없나?
```

### 스킵 허용

- 동일 패턴 반복 적용 (리네이밍, 포맷팅)
- 기존 플랜 기반 실행 중 (설계 검토 완료된 플랜)
- 사용자가 "토론 스킵" 명시 요청
- 설정/배포/단순 리팩토링 (신규 기능 없는 3-5파일 변경)

---

## [Check] 코드 생성 후 3단계 검증

> 도구 기반 자동 검증. Layer 1은 우회 가능하지만 Layer 2-3은 CI 강제.

| Layer | 검증 | 도구 | 우회 |
|-------|------|------|------|
| Check 1 | Pre-commit | Ruff(린트) + AST(문법) + 하드코딩 탐지 | --no-verify로 가능 |
| Check 2 | GitHub Actions | Claude Code Review + Bandit(보안) | 불가 (CI 강제) |
| Check 3 | E2E 테스트 | 통합 테스트 + 패턴 검증 | 불가 (CI 강제) |

### POST-BUILD 최종 확인

```
[검증]
[A] versions.json 일치: [OK/FAIL]
[B] patterns.json 위반: [OK/FAIL]
[C] API 키 하드코딩: [OK/FAIL]
[D] import 누락: [OK/FAIL]
결과: {N}개 문제 / 통과
```

---

## [DEBUGGING] 근본 원인 분석

> 스킬 `systematic-debugging` 기반. 트리거 키워드 감지 시 필수 실행.

### 트리거 키워드

"아직도", "똑같아", "그대론데", "달라진게 없는데", "여전히", "안 바뀌었어", "같은 에러", "또"

### 필수 실행 순서

```
[1/5] 문제 유형 식별
[2/5] 에러 로그/콘솔 실제 데이터 수집
[3/5] 데이터 기반 원인 분석 (추측 금지!)
[4/5] 수정 적용
[5/5] 사용자에게 보고: 원인 + 근거 + 해결 방안
```

### 강제 규칙

- [금지] 데이터 확인 없이 코드 수정
- [금지] "아마도", "~인 것 같습니다" 추측성 표현
- [필수] 도구 실행 결과 → 원인 분석 → 수정 제안 (순서 엄수)

---

## [COMMIT] 검증 후 커밋

> 스킬 `verification-before-completion` + `finishing-a-development-branch` 기반

"완료", "배포", "커밋" 감지 시:
1. 보안 스캔 실행 (API 키 노출 검사)
2. 코드 품질 검증 (lint, 타입 체크)
3. FAIL → 커밋 중단, 통과 시 git commit

---

## 환경 정보

| 항목 | 값 |
|------|-----|
| OS | Windows 11 |
| IDE | VSCode + Claude Code |
| AI | Claude Max |

---

## 세션 모드 시스템

| 모드 | 트리거 | GATE | 토론 |
|------|--------|------|------|
| **Plan-Light** | "~할까?", 질문형 | X | X |
| **Plan-Deep** | "분석해줘" | X | X |
| **Build** | "만들어줘", 명령형 | O (필수) | 3+ 파일 시 |

---

## React 작업 자동 감지

> 스킬 `vercel-react-best-practices` 기반

React/Next.js 키워드 감지 시 (`React로`, `컴포넌트`, `useState`, `.tsx` 등):
→ 스킬의 CRITICAL 8 규칙 자동 적용 (Defer Await, Suspense, Barrel Import 회피 등)

---

## 필수 보안 규칙

### Git Push 전 검증 (필수!)

```bash
grep -rE "(AIzaSy|sk-|ghp_|gho_|api_key\s*=)" --include="*.py" --include="*.js" --include="*.json" .
```

| 금지 패턴 | 올바른 방식 |
|----------|------------|
| `AIzaSy...` (Google) | `os.getenv('GOOGLE_API_KEY')` |
| `sk-...` (OpenAI) | `os.getenv('OPENAI_KEY')` |
| `ghp_...` (GitHub) | `os.getenv('GITHUB_TOKEN')` |

---

## 코드 리뷰

> 스킬 `requesting-code-review` + `receiving-code-review` 기반

핵심 원칙: 전체 검토 후 한꺼번에 보고 ("일단 이것만" 금지)
1. 입력 확인 → 2. 출력 확인 → 3. 구조 매칭 → 4. 전체 흐름

---

## 에이전트 부서 라우팅

> 상세 설정: routing.json

| 부서 | 에이전트 | 트리거 |
|------|---------|--------|
| 검증 | eazycheck-gate, code-reviewer, security-scanner, pattern-debugger | 검증/리뷰/보안/디버깅 |
| 개발 | python-backend, frontend-react | Python/React/구현 |
| 설계 | architect | 설계/아키텍처/리팩토링 |

### 토론 패턴 (3+ 파일)

3개 이상 파일 변경 시 자동 발동 (신규기능/복합수정만):
1. DMAD 자문자답 (설계자 Q1-4 + 사용자 Q1-4)
2. 합의 → 코드 작성 → sim deep → Check
- 설정/배포/리팩토링은 토론 스킵 + sim만 실행

---

## [sim deep] 장기 시나리오 검증 (3+ 파일, 코드 후)

> 스킬 `sim` 기반. 코드 작성 완료 후 자동 실행.

### 3단계 실패 시뮬레이션 루프

```
S1. 실패 시뮬레이션
    - "3개월 후, 사용자 10배" 상황에서 코드를 따라가며 터지는 곳 찾기
    - "터질 수 있다" (추상적) 금지 → "이 파일 N번째 줄에서 터진다" (구체적)

S2. 근본 원인
    - 터진 지점의 근본 원인 파악
    - 같은 유형이 코드 어디에 더 있나 전체 탐색

S3. 수정 + 확인
    - 근본 원인 기반 수정 (표면 땜질 금지)
    - regression 체크: 수정 전 동작하던 기능이 수정 후에도 동작하는지 확인
    - 수정 후 S1 재시뮬레이션 (최대 2회 루프)
```

- high 등급 → 즉시 수정 / medium 등급 → 사용자 승인 대기
- 2회 루프 후에도 실패 시 사용자 승인 요청

---

## 패턴 축적 시스템 (3Phase)

> 사용자의 교정/원칙/거부/결정을 자동으로 축적하여 같은 실수를 반복하지 않는 시스템

| Phase | 동작 | 타이밍 |
|-------|------|--------|
| Phase 1 | 사용자의 수정/원칙/거부 즉시 _buffer.json에 저장 | 발생 즉시 |
| Phase 2 | 긍정/승인 표현 감지 | "좋아", "OK", "됐어" 등 |
| Phase 3 | 분석 → 목적별 패턴 파일로 이동 → buffer 클리어 | Phase 2 후 자동 |

패턴 파일은 `memory/patterns/` 에 목적별로 분리 저장.
frequency 2+ 인 교정은 같은 실수를 사전 차단.

---

## 검증 시스템 역할 분리

| 상황 | 시스템 | 동작 |
|------|--------|------|
| 코드 생성 전 | GATE (EazyCheck) | versions + patterns Read |
| 3+ 파일 코드 전 | 토론 | architect + code-reviewer 병렬 |
| 3+ 파일 코드 후 | sim deep | 장기 시나리오 검증 |
| 코드 생성 후 | Check 1-3 | pre-commit → CI → E2E |
| 커밋 전 | 보안 스캔 + lint | 자동 |

---

## 작업 완료 체크리스트

- 보안 스캔 통과 (High 0)
- 빌드 경고 0개
- PR 링크 제공
- Git 브랜치: main
