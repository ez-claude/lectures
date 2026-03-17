# Claude Code 마스터 설정

> 이 파일은 항상 로드됨. 세부 규칙은 rules/ 폴더 참조.
>
> ## 저장 위치 (도구별)
>
> | AI 도구 | 파일명 | 저장 위치 |
> |---------|--------|----------|
> | **Claude Code** | `CLAUDE.md` | 프로젝트 폴더/.claude/ 안에 |
> | **Antigravity** | `RULES.md` | 프로젝트 폴더 최상위 |
> | **Cursor** | `.cursorrules` | 프로젝트 폴더 최상위 |

---

## 절대 규칙 (최우선)

1. **이모지 금지** - 텍스트로 대체 ([OK], [FAIL], [Step])
2. **한글 응답** - 모든 출력은 한글
3. **버전 잠금** - state/versions.json 우선
4. **보안 필수** - API 키 환경변수만, 하드코딩 금지
5. **Pillar 시스템 필수** - Build Mode에서 Step 0~5 강제 실행
6. **기록 전 검증** - 파일에 사실을 기록할 때, 도구로 확인 가능한 것(날짜, 경로, 버전 등)은 반드시 도구로 확인 후 기록. 컨텍스트/이전 세션 데이터는 현재 시점과 다를 수 있으므로 신뢰 금지.

---

## [GATE SYSTEM] 코드 생성 전 필수 관문

> **"만들어줘", "구현해줘", "코드" 키워드 감지 시 자동 실행. 관문 통과 전 코드 생성 금지**

### GATE -1: Pre-Plan Gate

플랜 품질 검증. ExitPlanMode 전 자동 실행.
5개 체크: 경쟁분석 / 근본원인 / 품질기준 / 도메인규칙 / 검증방법 → 3개 미만 시 차단

### GATE 0: 버전 파일 Read (필수!)

```
Read: versions.json
→ [GATE 0 통과] 라이브러리/AI 모델 정확한 버전 확인
```

### GATE 1: 패턴 파일 Read (필수!)

```
Read: patterns.json
→ [GATE 1 통과] 이전 실수 패턴 확인 + 관련 패턴 나열
```

### GATE 2: 관련 패턴 경고 → [GATE 통과] 출력 후 코드 생성 시작

---

## [POST-BUILD] 코드 생성 후 필수 검증

코드 생성 완료 후 반드시:
```
[/v 검증]
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
| IDE | VSCode / Antigravity |
| AI | Claude Max, Gemini, GPT |

---

## 세션 모드 시스템

| 모드 | 트리거 | 동작 |
|------|--------|------|
| **Plan-Light** | "~할까?", 질문형 | 가벼운 분석, 선택지 제시 |
| **Plan-Deep** | "분석해줘" | 심층 분석 + 경쟁 제품 비교 |
| **Build** | "만들어줘", 명령형 | GATE 시스템 발동 → 코드 생성 |

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

> 스킬 `dispatching-parallel-agents` + `subagent-driven-development` 기반
> 상세 설정: routing.json

| 부서 | 에이전트 | 트리거 |
|------|---------|--------|
| 검증 | pillar-gate, code-reviewer, security-scanner, pattern-debugger | 검증/리뷰/보안/디버깅 |
| 개발 | python-backend, frontend-react, n8n-builder | Python/React/n8n/구현 |
| 설계 | architect | 설계/아키텍처/리팩토링 |

3개 이상 파일 변경 시 토론 패턴 자동 발동 (architect + code-reviewer 병렬)

---

## 검증 시스템 역할 분리

| 상황 | 시스템 | 명령 |
|------|--------|------|
| 코드 생성 후 자동 검증 | Pillar (GATE) | `/v` 또는 자동 |
| 커밋 전 전체 검증 | 보안 스캔 + lint | 자동 |

---

## 작업 완료 체크리스트

- 보안 스캔 통과 (High 0)
- 빌드 경고 0개
- PR 링크 제공
- Git 브랜치: main

