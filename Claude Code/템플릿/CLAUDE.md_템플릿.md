---
tags:
  - 하네스
  - 템플릿
  - 라이브러리
updated: 2026-04-30
---

# CLAUDE.md 템플릿 — 라이브러리 전용

라이브러리 프로젝트 루트에 위치하는 Claude Code 하네스 제어 파일 템플릿.

> **프로젝트 유형 가정**
> - 단일 패키지 라이브러리 (디자인시스템·UI 키트·유틸·SDK·CLI 등)
> - 빌드 산출물(`dist/`)을 외부 앱이 소비
> - 페이지(routes)가 없음. UI 검증은 Storybook·Playground·시각 회귀 테스트로 갈음
>
> 모노레포 안의 라이브러리 패키지(예: `packages/ui-kit/`)에 적용 시: 본 템플릿을 패키지 루트에 두고, 모노레포 루트의 CLAUDE.md(본체 하네스 템플릿)와 함께 운용한다. 충돌 시 더 제한적인 것 적용.

> **에이전트 사용 안내**
> 1. 이 템플릿을 라이브러리 루트에 `CLAUDE.md`로 복사한다.
> 2. `<!-- FILL: ... -->` 주석 안의 내용을 프로젝트 분석 결과로 채운다.
> 3. 채운 뒤 주석과 이 안내 섹션을 삭제한다.
>
> **분석 전 필수 확인 목록 (라이브러리 시그널)**
> - `package.json` → `exports` / `main` / `module` / `types` / `files` / `peerDependencies` / `sideEffects`
> - 빌드 도구 — `tsup.config.ts` · `rollup.config.*` · `vite.config.*` (라이브러리 모드)
> - 시각 검증 도구 — `.storybook/` · `playground/` · `examples/` 등
> - 테스트 — `vitest.config.*` · `jest.config.*` · `*.test.tsx` 위치
> - 컴포넌트 디렉토리 — `src/components/` · `src/primitives/` · `lib/` 등
> - 디자인 토큰 — `src/styles/tokens/` · `tailwind.config.*` · `tokens.json` 등
> - 릴리스 도구 — `.changeset/` · `release-please-config.*` · `semantic-release` 설정
> - 코드 스타일 — `biome.json` · `.eslintrc.*` · `.prettierrc.*`
> - `git log --oneline -10` → 컨벤셔널 커밋 사용 여부, 브랜치 전략 시그널
> - 이슈 트래커 — GitHub Issues / Linear / Jira / **(없음)**

---

<!-- 아래부터 실제 CLAUDE.md 내용 시작 — 라이브러리에 맞게 수정 후 사용 -->

# [라이브러리명] — Claude Code 하네스

<!-- FILL: 라이브러리명을 실제 이름으로 교체 (예: `acme-ui`, `@acme/components`) -->

> 이 문서는 단순 가이드가 아닌 **작업을 통제하는 하네스**다.
> 아래 규칙을 따르지 않으면 작업을 진행하지 않는다.

> **프로젝트 유형**: <!-- FILL: 라이브러리 유형 --> (예: React 디자인시스템 라이브러리 / TypeScript SDK / CLI 도구)
> 빌드 산출물(`dist/`)을 외부 앱이 소비.

---

## 🔴 STEP 0 — 작업 시작 전 필수 (건너뜀 금지)

### 0-A. 작업 영역 판별

요청을 받으면 **코드를 건드리기 전에** 작업 대상 영역을 판별한다.
불명확하면 **반드시 사용자에게 먼저 질문한다.** 임의 추정 금지.

라이브러리는 패키지가 1개라서 영역 판별이 **"코드 카테고리"** 단위가 된다.

<!-- FILL: 라이브러리 실제 디렉토리 구조에 맞게 행 추가·수정·삭제

  [작성 방법]
  1. `find src -maxdepth 2 -type d` 로 실제 구조 확인
  2. 한 행 = 한 카테고리. 키워드는 사용자가 자연어로 말할 때 쓸 법한 단어
  3. 사용하지 않는 카테고리(예: 토큰이 없는 SDK)는 행 삭제

  [디자인시스템 예시]
  | 컴포넌트 추가·수정·variant·prop | `src/components/<카테고리>/<컴포넌트>/` |
  | 색상·spacing·radius 등 디자인 값 | `src/styles/tokens/` |
  | 스토리북·예시·시각 검증 | `stories/` 또는 `.storybook/` |
  | 단위 테스트 | `<컴포넌트경로>/<이름>.test.tsx` |
  | 빌드·exports·릴리스 | `tsup.config.ts` · `package.json` · `.changeset/` |
  | 유틸 함수 (cn 등) | `src/utils/` |

  [SDK 예시]
  | API 클라이언트 추가·메서드 | `src/clients/` |
  | 타입 정의·스키마 | `src/types/` · `src/schemas/` |
  | 빌드·exports·릴리스 | `tsup.config.ts` · `package.json` · `.changeset/` |
  | 유틸·헬퍼 | `src/utils/` |
-->

| 키워드 / 단서 | 작업 영역 |
|---|---|
| `[키워드1]`, `[키워드2]` | `[경로1]` |
| `[키워드3]`, `[키워드4]` | `[경로2]` |
| 빌드·exports·릴리스 | `[빌드도구설정]` · `package.json` · `.changeset/` |

불명확 시 질문:
> "이 작업은 [카테고리1]인가요, [카테고리2]인가요, 빌드 설정인가요?"

---

### 0-B. 진입 문서 로드

작업 영역 확정 후 진입 문서를 읽고 작업을 시작한다.

라이브러리는 **agent/ 3종 문서**가 핵심 컨텍스트다. 컴포넌트·토큰·exports 작업 모두 이 문서들에 의존한다.

<!-- FILL: 실제 docs 경로로 수정. design-system.md는 디자인시스템·UI 키트가 아닌 SDK·CLI에서는 행 삭제 가능.

  기본 경로 패턴:
  - HANDOFF_NOW: `docs/<라이브러리명>/status/HANDOFF_NOW.md`
  - agent/architecture: `docs/<라이브러리명>/agent/architecture.md`
  - agent/conventions: `docs/<라이브러리명>/agent/conventions.md`
  - agent/design-system: `docs/<라이브러리명>/agent/design-system.md`
-->

| 문서 | 경로 |
|---|---|
| **HANDOFF_NOW** (현재 상태·다음 작업) | `docs/[라이브러리명]/status/HANDOFF_NOW.md` |
| **agent/architecture.md** (라이브러리 구조·빌드·exports) | `docs/[라이브러리명]/agent/architecture.md` |
| **agent/conventions.md** (코딩 규칙) | `docs/[라이브러리명]/agent/conventions.md` |
| **agent/design-system.md** (토큰·variant 패턴) | `docs/[라이브러리명]/agent/design-system.md` |

읽은 후 §1(현재 상태)·§2(다음 작업)를 사용자에게 요약 출력한다.

---

### 0-C. 기존 코드 분석 (코드 작성 전 필수)

**코드 추가·수정 시 유사 사례 최소 2개를 먼저 읽는다.**

<!-- FILL: 라이브러리 유형에 맞춰 분기 가이드 작성

  [디자인시스템 예시]
  - 새 컴포넌트가 Radix 기반이면 → `accordion`, `dialog`, `tabs` 중 2개
  - 새 컴포넌트가 단순(non-Radix)이면 → `badge`, `spinner`, `skeleton` 중 2개
  - variant 시스템이 필요하면 → `button`, `badge` 의 `cva` 패턴 확인

  [SDK 예시]
  - 새 API 메서드 추가면 → 같은 도메인 메서드 2개 (요청·응답·에러 핸들링 패턴)
  - 새 클라이언트 클래스면 → 기존 클라이언트 2개 (생성자·인증·재시도 패턴)
-->

확인 항목 (디자인시스템 기준): 디렉토리 구조 / `displayName` / `forwardRef` 사용 여부 / `cva` variants / 테스트 형태 / 스토리 형태.

### 0-D. 구현 계획 수립 및 확인

분석 결과를 바탕으로 단계별 구현 계획을 수립한 뒤 **사용자에게 제시하고 확인을 받는다.**

```
[구현 계획]
- 신규/수정 파일: <파일 목록>
- 변경 내용: <단계별 작업>
- 영향 범위: <export·index.ts 갱신·기존 컴포넌트/메서드 영향 여부·peerDeps 변동>
```

**사용자 확인 전까지 코드 작성·파일 수정 금지.**
계획 변경 요청 시 계획을 수정한 뒤 재확인한다.

---

## 에이전트 작업 방식 (필수)

**모든 작업에서 아래 루프를 따른다. 임의 실행 금지.**

```
1. 분석 — 유사 사례 2개 이상 확인 (구조·variant·테스트·스토리)
2. 제안 — "다음 행동: [구체적 행동]을 하겠습니다. 진행할까요?" 사용자에게 먼저 물어봄
3. 대기 — 사용자 승인(예/진행/ok 등) 확인
4. 실행 — 승인 후에만 코드 작성·파일 수정·명령 실행
5. 보고 — 완료 결과 요약 후 다음 행동 제안으로 돌아감
```

**예외 — 승인 없이 바로 실행 가능한 것:**
- 파일 읽기·검색 등 조회성 작업
- 사용자가 "바로 해줘" / "한 번에 다 해줘" 등으로 명시적으로 허가한 경우

---

## 횡단규칙 — 모든 작업에 적용

| # | 규칙 | 설명 | 위반 예 |
|---|---|---|---|
| 1 | **Think Before Coding** | → STEP 0-D 참조. 분석·계획·확인 전 코드 작성 금지 | 추정으로 코드 작성 시작 |
| 2 | **Simplicity First** | 요청된 만큼만. 단발성 코드에 추상화 금지. 발생 불가능한 시나리오 에러 처리 금지 | "나중에 쓸지도"로 옵션 추가 |
| 3 | **Surgical Changes** | 변경된 모든 줄은 요청에 추적 가능해야 함. 인접 코드 임의 정리 금지 | 버그 수정 중 주변 포맷팅 정리 |
| 4 | **Goal-Driven Execution** | 작업을 검증 가능한 목표로 변환. 검증 전 완료 처리 금지 | "되는 것 같다"로 마무리 |

---

## 🔴 STEP 1 — 작업 중 강제 규칙

> **규칙 위반 발견 시: 즉시 중단 → 사용자에게 위반 내용 보고 → 지시 후 재개. 임의 수정 후 계속 진행 금지.**

라이브러리 핵심 강제 규칙 3종은 다음과 같다. 라이브러리 유형에 맞게 조정한다.

<!-- FILL: 라이브러리 유형에 따라 핵심 3종을 조정

  [디자인시스템 — 그대로 사용 권장]
  1. 신규 컴포넌트 N개 산출물 동시 작성 (구현·테스트·index·스토리·루트 export)
  2. 디자인 토큰 직접 hex 사용 금지 — semantic 토큰 변수 경유
  3. 루트 components index.ts export 갱신 누락 금지

  [SDK — 예시 치환]
  1. 신규 메서드 N개 산출물 동시 작성 (구현·타입·테스트·index export)
  2. 외부 노출 타입은 별도 export 필수 (소비자 d.ts 누락 방지)
  3. 루트 index.ts export 갱신 누락 금지
-->

| 규칙 | 위반 시 |
|---|---|
| **신규 컴포넌트는 [N]개 산출물 동시 작성** (`<name>.tsx` / `<name>.test.tsx` / `index.ts` / `stories/<name>.stories.tsx` / `[루트 export 파일]` export 추가) | 즉시 중단 → 누락 항목 보고 후 보완 |
| **디자인 토큰 직접 hex 사용 금지** — 컴포넌트 className에 `#fabc37` 같은 raw값 작성 금지. semantic 토큰 변수(`bg-bg-brand-default` 등) 경유 | 즉시 중단 → semantic 토큰으로 교체 |
| **`[루트 export 파일]` 갱신 누락 금지** — 신규 export 추가 후 빌드 실행 | 즉시 중단 → export 추가 후 빌드 재실행 |

이슈·브랜치·커밋 규칙:

<!-- FILL: 이슈 트래커 사용 여부에 따라 첫 행을 활성/삭제

  [트래커 사용 시]
  - 행 그대로 유지

  [트래커 미사용 시]
  - "(이슈 트래커 도입 후)" 행을 통째로 삭제하거나
  - "트래커 미도입 — 본 규칙 비활성" 으로 명시

  [브랜치 보호 행]
  - Git Flow: `main`·`develop` 양쪽 직접 커밋 금지
  - trunk-based: `main` 직접 커밋 금지
  - 단일 long-lived: `main`만 사용
-->

| 규칙 | 위반 시 |
|---|---|
| **(이슈 트래커 도입 후) 이슈 번호 없이 커밋 금지** — 트래커 미도입이면 본 행 삭제 | 즉시 중단 → 이슈 생성 후 재개 |
| **`[보호 브랜치]` 직접 커밋 금지** — Git Flow면 `main`·`develop` / trunk-based면 `main` | 즉시 중단 → 브랜치 생성 |
| **커밋은 명시적 요청 시에만** | 사용자가 "커밋해줘" 전까지 커밋 불가 |

> **규칙 우선순위**: 모노레포 안의 라이브러리라면 모노레포 루트 CLAUDE.md가 함께 적용된다. 충돌 시 **더 제한적인 것** 적용.

---

## 🔴 STEP 2 — 검증 하네스 (코드 작성 후 필수)

```
[규칙 준수 체크]
- [ ] [N]개 산출물 모두 작성 (신규 컴포넌트/메서드 시)
- [ ] 디자인 토큰 직접 hex 사용 없음 (디자인시스템)
- [ ] forwardRef 사용 패턴 일관 (Radix 래퍼는 forwardRef, 단순 컴포넌트는 함수)
- [ ] cva variants 네이밍 기존 패턴 따름 (variant·size 키)
- [ ] 변경된 모든 줄이 사용자 요청에 추적 가능 (Surgical Changes)
- [ ] 테스트 통과 (`pnpm test` 등)
- [ ] 빌드 통과 (`pnpm build` — `dist/` 갱신·exports 정상)
- [ ] 린트 통과 (`pnpm lint`)
- [ ] (디자인시스템) Storybook 시각 확인 — 사용자가 직접
```

<!-- FILL: 라이브러리 유형에 맞춰 체크 항목 조정. SDK·CLI는 forwardRef·cva·Storybook 항목 삭제. -->

**체크 실패 시:** 위반 항목을 사용자에게 보고한 뒤 수정 방향을 확인받는다. 스스로 판단해 수정 후 완료 처리 금지.

**테스트 작성 기준:**
- 신규 컴포넌트·메서드 → **필수**
- variant·prop·옵션 추가 → **필수**
- 토큰만 변경 → 시각 회귀(스토리북) 확인
- 문서·설정만 변경 → 생략

**시각 검증 방법 (디자인시스템·UI 라이브러리):**
- 컴포넌트 추가·수정 시 `pnpm storybook` 으로 해당 스토리 확인 (사용자가 직접)
- 자동 검증은 단위 테스트로 충분 — Playwright 별도 도입은 라이브러리 단계에서는 과함
- 시각 회귀가 필요하면 Chromatic·Loki 등 별도 도입 검토

**빌드·타입·린트 오류 시:** ① 내 코드 문제 → 즉시 수정 ② 기존 호환성 문제(peerDeps 충돌·exports 깨짐) → `debugger` 에이전트 ③ 2회 이상 반복 실패 → **즉시 중단·사용자 보고**

---

## agent/ 문서 갱신 트리거

매 세션마다 갱신하지 않고 아래 조건 충족 시에만 갱신한다. 갱신은 doc-writer 에이전트에게 위임한다.

<!-- FILL: 라이브러리 유형에 맞춰 트리거 조정. design-system 행은 디자인시스템 외에는 삭제 가능. -->

| 문서 | 갱신 트리거 |
|------|------------|
| `agent/architecture.md` | 빌드 시스템 변경 (tsup→rollup 등), 새 디렉토리 구조 추가, `peerDependencies` 변경, `exports` 패턴 변경 |
| `agent/conventions.md` | 린터·포매터 설정 변경, 파일 네이밍 규칙 변경, 새 컨벤션 결정 |
| `agent/design-system.md` | 디자인 토큰 추가·변경, 새 variant 패턴 도입, semantic 토큰 재정의, 컴포넌트 라이브러리(Radix 등) 업그레이드 |

---

## 🔴 STEP 3 — 세션 종료 강제 절차

코드·문서 변경이 있었던 세션은 사용자 지시 없이도 자동 수행한다.

<!-- FILL: 실제 docs 경로로 수정
  기본 패턴 (단일 패키지 라이브러리):
  - HANDOFF_NOW: `docs/<라이브러리명>/status/HANDOFF_NOW.md`
  - 세션_노트: `docs/<라이브러리명>/history/세션_노트.md`
  - HANDOFF: `docs/<라이브러리명>/plans/HANDOFF.md`
-->

**갱신 순서 (순서 바꾸지 말 것):**
1. `docs/[라이브러리명]/status/HANDOFF_NOW.md` — §1 현재 상태·§2 다음 작업 갱신 (60줄 이하 유지)
2. `docs/[라이브러리명]/history/세션_노트.md` — 최상단 prepend (`> Session note YYYY-MM-DD: [요약]`)
3. `docs/[라이브러리명]/plans/HANDOFF.md` — `## Session Update YYYY-MM-DD` 섹션 최상단 추가

**갱신 후 일관성 검증:** HANDOFF_NOW.md §2 첫 항목이 최신 다음 작업인지 / 60줄 이하인지 확인.

**컴포넌트 추가·수정 시 추가 산출물:** 복잡한 컴포넌트(DnD·차트·복합 폼)는 `docs/[라이브러리명]/components/<컴포넌트>.md` 작성 검토. 단순 컴포넌트는 Storybook 스토리·테스트 코드가 1차 문서.

**변경 결과 요약 출력 (필수):**
```
[작업 결과]
- 변경 파일:
- 주요 변경:
- 영향 범위:
```

위 결과 요약 없으면 작업 완료로 간주하지 않음.

---

## 협의 대기 상태 처리

협의가 필요해 진행 불가 시: ① 즉시 중단 ② HANDOFF_NOW.md §2에 `⏸ [협의 대기] <작업명> — <협의 내용> / 담당: <팀원명>` 추가 ③ 사용자에게 보고 ④ 독립적인 다른 태스크로 전환.

---

## 작업 범위 초과 시

요청 범위 밖 변경(파일·디렉토리 추가, 의존성 설치, 설정 수정, 요청하지 않은 파일 수정 등)이 필요하다고 판단될 때:
① 즉시 중단 ② 사용자에게 필요한 추가 변경과 이유를 보고 ③ 허가 후 진행.
거부 시 해당 변경 없이 가능한 범위만 구현 후 보고.

---

## 서브에이전트 호출 규칙

<!-- FILL: 라이브러리 dev 에이전트 이름은 보통 `<라이브러리명>-dev`. doc-writer는 design-system 트리거 처리를 명시적으로 담당. -->

| 작업 유형 | 호출 에이전트 |
|---|---|
| 컴포넌트·메서드 추가·수정·variant·prop | `[라이브러리명]-dev` |
| 디자인 토큰 변경·확장 (디자인시스템) | `[라이브러리명]-dev` (design-system.md 자동 갱신 트리거) |
| 문서·HANDOFF 갱신 | `[라이브러리명]-doc-writer` |
| 코드 리뷰 | `code-reviewer` |
| 버그 원인 추적 | `debugger` |
| 리팩토링 | `refactor` |
| 테스트 작성 | `test-writer` (**프롬프트에 "기존 소스 파일 수정 금지, 테스트 파일만 작성" 명시 필수**) |

---

## 팀 프로세스

<!-- FILL: 실제 문서 경로로 수정. 라이브러리는 보통 git-workflow 1행으로 통합 관리. -->

| 작업 | 참조 문서 |
|------|-----------|
| 커밋·브랜치·PR 규칙 (브랜치 전략·네이밍·컨벤셔널 커밋) | `docs/[라이브러리명]/git-workflow/branch-commit.md` |
| PR 템플릿 | `.github/PULL_REQUEST_TEMPLATE.md` (없으면 생성 검토) |
| 릴리스 절차 | `docs/[라이브러리명]/git-workflow/release.md` 또는 `.changeset/README.md` |

> 브랜치 분기·머지·PR 대상 매트릭스는 위 문서를 참조한다. 브랜치 생성·커밋·PR 생성 시점에만 읽으면 된다.

### 브랜치·커밋 핵심 규칙

<!-- FILL: 실제 팀 컨벤션으로 수정
  분석 기준: git log --oneline -20 으로 기존 커밋 패턴 파악

  [브랜치 전략 옵션]
  - Git Flow: main(릴리스)·develop(개발) + feature/*·release/*·hotfix/*
  - trunk-based: main + short-lived feature/*
  - 단일 long-lived: main 만 사용

  본 템플릿 기본값은 Git Flow 옵션. 다른 전략이면 표를 교체.

  [커밋 규칙 옵션]
  - 컨벤셔널 커밋 (Changesets·semantic-release 와 호환) — 라이브러리 권장
  - 사내 포맷 — 회사 정책이 있으면 그대로 따름
-->

| 항목 | 규칙 (Git Flow + 컨벤셔널 커밋 기준) | 예시 |
|------|---|------|
| **브랜치 전략** | `main`(릴리스 태그)·`develop`(통합) + `feature/*`·`release/*`·`hotfix/*` | `feature/button-loading-state` |
| **브랜치명** | (트래커 사용) `<타입>/<이슈번호>` / (미사용) `<타입>/<짧은-설명>` | `feature/1234` 또는 `feature/add-tooltip` |
| **브랜치 기준점** | `feature/*`·`release/*` → `develop` / `hotfix/*` → `main` | `git checkout develop && git pull` 후 분기 |
| **커밋 메시지** | 컨벤셔널 커밋 — `<타입>(<scope>): <설명>` | `feat(button): add loading variant` |
| **이슈·PR 제목** | `<타입>: <설명>` (스코프 선택) | `feat: add button loading variant` |
| **릴리스** | `pnpm changeset` → `pnpm version` → `pnpm release` (Changesets 기준) | 변경마다 `.changeset/*.md` 추가 |

---

## 공통 규칙

<!-- FILL: 실제 패키지 매니저·빌드·테스트·릴리스 명령으로 수정 -->

- 언어: 한국어로 응답.
- 패키지 매니저: `[pnpm/npm/yarn]`
- 빌드: `[빌드 명령]` (예: `pnpm build` — tsup, ESM+CJS dual)
- 개발: `[개발 명령]` (예: `pnpm dev` — tsup --watch)
- 시각 확인: `[시각 확인 명령]` (예: `pnpm storybook`)
- 테스트: `[테스트 명령]` (예: `pnpm test` — vitest)
- 린트·포맷: `[린트 명령]` / `[포맷 명령]` (예: `pnpm lint` / `pnpm format` — biome)
- 릴리스: `[릴리스 명령]` (예: `pnpm changeset → pnpm version → pnpm release`)

---

## 스킬

<!-- FILL: 실제 설치된 스킬 확인 후 목록 교체
  확인 명령: ls ~/.claude/skills/ .claude/skills/ 2>/dev/null | sort -u
  설치 안 된 스킬은 행 삭제. 추가 설치된 스킬(개인 스킬 등)은 행 추가.
-->

| 스킬 | 용도 |
|------|------|
| `/session-close` | 세션 종료 — HANDOFF 3종 갱신 |
| `/project-fix` | QA·버그 이슈 → 서브이슈 생성 + 브랜치 준비 (이슈 트래커 사용 시) |
| `/project-pr` | PR 생성 — 이슈 연결 |
| `/project-issue` | GitHub 이슈 인터랙티브 생성 (이슈 트래커 사용 시) |
| `/document-review` | 문서 세트 시나리오 검증·이슈 수정 |
