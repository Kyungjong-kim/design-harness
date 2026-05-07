---
tags: [design-harness, examples, 초기]
updated: 2026-05-07
---

# Example 01 — M0: `/project-init` 직후 골격

> **`acme-ui`** 디자인시스템 라이브러리에 design-harness를 처음 적용한 직후 상태.
> 약 150줄의 `CLAUDE.md`와 기본 구조 문서 3종이 만들어진다.
> 도메인 특정 규칙은 아직 없다 — 이것이 **의도된** 초기 상태다.

---

## 생성된 파일 목록

```
acme-ui/
  CLAUDE.md                              ← 150줄 / STEP 0~3 기본 골격
  docs/acme-ui/
    agent/
      architecture.md                    ← 빌드·exports·peerDeps 분석
      conventions.md                     ← 컴포넌트 파일 순서·네이밍 규칙
      design-system.md                   ← 토큰 2계층 구조 (초안)
  .claude/
    skills/                              ← project-init / project-fix / session-close 등
    agents/
      acme-ui-dev.md
      acme-ui-doc-writer.md
  HANDOFF_NOW.md                         ← §1 현재 상태 · §2 다음 작업 초기화
  HANDOFF.md                             ← 세션 이력 (비어 있음)
  docs/acme-ui/history/세션_노트.md       ← 초기화
```

---

## CLAUDE.md 발췌 (M0 — 150줄)

```markdown
# acme-ui — Claude Code 하네스

> 라이브러리 프로젝트 (npm publish 대상). React 디자인시스템.
> 빌드: tsup 8.x / 테스트: vitest / 시각 검증: Storybook

---

## 🔴 STEP 0 — 작업 시작 전

### 0-A 작업 영역 판별

| 키워드 / 단서 | 영역 |
|---|---|
| 컴포넌트 추가·수정 | components/ |
| 토큰 변경 | tokens/ |
| 빌드·exports | tsup.config.ts |
| 스토리 추가 | stories/ |

불명확 시 → 질문

### 0-B 진입 문서

- `docs/acme-ui/agent/architecture.md`
- `docs/acme-ui/agent/conventions.md`
- `docs/acme-ui/agent/design-system.md`

### 0-C 기존 코드 분석 (필수)

유사 컴포넌트 최소 2개 확인. 파일 순서·variant 패턴·토큰 사용 방식 파악.

### 0-D 구현 계획 수립

계획 제시 → 사용자 확인 후 작업 시작.

---

## 🔴 STEP 1 — 강제 규칙

| 규칙 | 위반 시 |
|---|---|
| **5개 산출물 동시 작성** — `<name>.tsx` + `<name>.test.tsx` + `index.ts` + `<name>.stories.tsx` + `components/<index>.ts` export | 작업 중단 → 누락 파일 추가 |
| **semantic 토큰만 사용** — `color.primary.500` 금지, `color.brand` 사용 | 작업 중단 → 토큰 교체 |
| **루트 export 추가** — `src/index.ts`에 새 컴포넌트 export 추가 | 작업 중단 → export 추가 |

---

## 🔴 STEP 2 — 검증 하네스

```
[규칙 준수 체크]
- [ ] 5개 산출물 전부 작성됨
- [ ] semantic 토큰만 사용 (core 토큰 직접 참조 없음)
- [ ] src/index.ts export 추가됨
- [ ] pnpm test 통과
- [ ] pnpm build 통과
- [ ] pnpm lint 통과
- [ ] Storybook에서 새 Story 시각 확인
```

---

## 🔴 STEP 3 — 세션 종료

1. `HANDOFF_NOW.md` §1·§2 갱신 (60줄 이하 유지)
2. `docs/acme-ui/history/세션_노트.md` 최상단에 1~2줄 prepend
3. `HANDOFF.md` 최상단에 `## Session Update YYYY-MM-DD` 추가

---

## 코딩 4원칙

| # | 원칙 | 핵심 |
|---|------|------|
| 1 | Think Before Coding | 유사 컴포넌트 2개 먼저 확인 |
| 2 | Simplicity First | 요청된 variant만. 미래 확장 금지 |
| 3 | Surgical Changes | 변경된 줄은 요청에 추적 가능해야 함 |
| 4 | Goal-Driven Execution | pnpm test + Storybook 시각 확인 전 완료 처리 금지 |
```

---

## architecture.md 발췌 (M0)

```markdown
# acme-ui — 아키텍처

## 빌드 흐름
tsup 8.x → ESM + CJS dual + `dts: true`
출력: `dist/index.js` (ESM) / `dist/index.cjs` (CJS) / `dist/index.d.ts`

## exports 구조
package.json `exports` 필드:
- `.` → dist/index
- `./tokens` → dist/tokens/index

## peerDependencies
- react ^19
- react-dom ^19

## 컴포넌트 카테고리
- Radix 래핑 (14개): Dialog, Tooltip, Select, Popover …
- 단순 (8개): Button, Badge, Spinner, Avatar …

## 디렉토리
src/
  components/   컴포넌트별 디렉토리
  tokens/       디자인 토큰 (core / semantic)
  index.ts      루트 export
```

---

## 초기 상태의 빈 자리들 (의도적)

M0 CLAUDE.md에 없는 것들 — 운영하면서 채워질 공간:

- STEP 1에 컴포넌트별 함정 규칙 없음 → M1~M6에서 추가됨
- STEP 2 체크리스트에 도메인 항목 없음 → 첫 버그 발견 후 추가됨
- design-system.md 토큰 목록 미완 → 컴포넌트 추가하며 누적됨

**이게 하네스의 정상 초기 상태다. 빈 자리가 많다고 실패한 게 아니다.**
