---
tags: [design-harness, examples, 6개월]
updated: 2026-05-07
---

# Example 03 — M6: 6개월 운영 후

> **`acme-ui`** 라이브러리에 design-harness 적용 6개월 후 상태.
> 컴포넌트 35개로 증가, 함정 14개 누적, CLAUDE.md 150줄 → 285줄로 성장.
> design-system.md가 실질적 "작업 전 필독" 문서가 되었고,
> 릴리스 절차·peer 의존성 관리가 STEP에 흡수되었다.

---

## 6개월간 무슨 일이 있었나

| 시점 | 이벤트 | CLAUDE.md 변화 |
|---|---|---|
| M0 | `/project-init` — 초기 골격 (~150줄) | 5개 산출물·semantic 토큰·루트 export 3개 규칙 |
| M0+1주 | `asChild` prop 추가 → Radix Slot `ref` 통과 실패 | STEP 1: `forwardRef + displayName` 강제 |
| M0+2주 | Select에서 `color.brand[500]` 직접 참조 발견 | STEP 1: core 토큰 직접 참조 금지 재명시 |
| M0+3주 | FormField 하위 컴포넌트 네이밍 충돌 | STEP 1: `<Parent>.<Child>` 복합 컴포넌트 패턴 강제 |
| M1 | Storybook dark mode 미반영 bug report | STEP 2: dark mode Story 필수 체크 추가 |
| M2 | DnD 컴포넌트 추가 — `react-dnd` vs `@dnd-kit` 결정 필요 | STEP 0: "DnD 라이브러리 선택 가이드" 0-D 추가 |
| M2 | `Popover` + `Command` 조합 — z-index 충돌 | STEP 1: Radix 오버레이 컴포넌트 z-index 규칙 |
| M3 | Changesets 릴리스 중 `peerDependencies` 버전 범위 오기입 | STEP 3에 릴리스 전 체크 추가 |
| M3 | 컴포넌트 삭제 후 루트 export 미정리 → 빌드 오류 | STEP 1: 삭제 시 `src/index.ts` 제거 강제 |
| M4 | 새 semantic 토큰 추가 절차 미흡 → 컴포넌트마다 따로 정의 | design-system.md: 토큰 추가 절차 섹션 신설 |
| M4 | a11y 감사 — 포커스 링 미정의 | STEP 2: `focus-visible` 토큰 체크 추가 |
| M5 | 팀원 B가 `tsup.config.ts` 수정 → CJS 출력 깨짐 | STEP 1: 빌드 설정 수정 시 dual output 검증 강제 |
| M5 | Storybook 업그레이드(9 → 10) — Story 포맷 변경 | STEP 0-B에 "Storybook 버전 확인" 추가 |
| M6 | npm publish 자동화 CI 도입 | STEP 3: CI publish 체크리스트 흡수 |

---

## CLAUDE.md 발췌 (M6 — 285줄, 주요 변경 부분)

### STEP 1 — 강제 규칙 (M6 전체)

```markdown
## 🔴 STEP 1 — 강제 규칙

| 규칙 | 발견 시점 | 위반 시 |
|---|---|---|
| **5개 산출물 동시 작성** — `.tsx` + `.test.tsx` + `index.ts` + `.stories.tsx` + `components/<index>.ts` export | M0 | 작업 중단 → 누락 파일 추가 |
| **semantic 토큰만 사용** — `color.brand[500]` 직접 참조 금지 | M0 | 작업 중단 → 토큰 교체 |
| **루트 export 추가** — `src/index.ts`에 새 컴포넌트 export 추가 / 삭제 시 제거 | M0 + M3 | 작업 중단 → export 동기화 |
| **Radix 컴포넌트에 `asChild` 지원 시** — `React.forwardRef` 필수, `displayName` 명시 | M0+1주 | 런타임 ref 에러 이력 — 건너뜀 금지 |
| **복합 컴포넌트** (FormField·Combobox 등) — 하위는 `<Parent>.<Child>` 네임스페이스 | M0+3주 | 네이밍 충돌 이력 |
| **Radix 오버레이 컴포넌트** (Popover·Dialog·DropdownMenu 등) — z-index `tokens.zIndex.overlay` 고정 | M2 | 레이어 충돌 이력 |
| **컴포넌트 삭제 시** — `src/index.ts` 동시 제거 필수 | M3 | 빌드 오류 이력 |
| **빌드 설정(`tsup.config.ts`) 수정 시** — ESM + CJS dual output 로컬 확인 필수 | M5 | CJS 출력 깨짐 이력 |
| **새 semantic 토큰 추가 시** — `design-system.md §토큰 추가 절차` 먼저 실행 | M4 | 토큰 분산 이력 |
```

### STEP 2 — 검증 하네스 (M6 전체)

```markdown
[규칙 준수 체크]
- [ ] 5개 산출물 전부 작성됨
- [ ] semantic 토큰만 사용 (core 토큰 직접 참조 없음)
- [ ] src/index.ts export 추가·제거 동기화됨
- [ ] pnpm test 통과
- [ ] pnpm build 통과 (ESM + CJS dual 확인)
- [ ] pnpm lint 통과
- [ ] Storybook Default + variants + dark mode Story 시각 확인
- [ ] (Radix 컴포넌트) asChild 구현 시 forwardRef + displayName 확인
- [ ] (Radix 오버레이) z-index tokens.zIndex.overlay 사용 확인
- [ ] (복합 컴포넌트) <Parent>.<Child> 패턴 확인
- [ ] (새 semantic 토큰) design-system.md 반영 완료
- [ ] focus-visible 스타일 정의됨 (접근성)
```

### STEP 3 — 세션 종료 (M6, 릴리스 항목 추가)

```markdown
1. `HANDOFF_NOW.md` §1·§2 갱신 (60줄 이하 유지)
2. `docs/acme-ui/history/세션_노트.md` 최상단에 1~2줄 prepend
3. `HANDOFF.md` 최상단에 `## Session Update YYYY-MM-DD` 추가
4. `design-system.md` 토큰 사용처 표 갱신 (신규 컴포넌트 추가 시)
5. (릴리스 시) changeset 생성 → `pnpm changeset` → PR merge → CI publish 확인
```

---

## design-system.md 발췌 (M6 — 성숙 상태)

```markdown
# acme-ui — 디자인시스템

## 토큰 2계층 구조

### Core 토큰 (`src/tokens/core.ts`)
> 절대 직접 참조 금지. Semantic 토큰이 소비한다.

### Semantic 토큰 (`src/tokens/semantic.ts`)
> 컴포넌트에서 이것만 참조.

export const tokens = {
  color: {
    brand:          color.brand[500],
    brandHover:     color.brand[600],
    brandSubtle:    color.brand[50],       // M4 추가 — Badge 배경용
    surface:        color.neutral[0],
    surfaceSub:     color.neutral[50],
    surfaceOverlay: color.neutral[950],    // M2 추가 — 모달 딤
    text:           color.neutral[900],
    textSub:        color.neutral[500],
    textDisabled:   color.neutral[300],    // M4 추가
    textInverse:    color.neutral[0],      // M2 추가 — Tooltip
    border:         color.neutral[200],
    borderFocus:    color.brand[500],      // M4 추가 — focus-visible
    danger:         color.error[500],
    dangerSubtle:   color.error[50],       // M3 추가
    success:        color.green[500],      // M3 추가
  },
  zIndex: {
    overlay:   200,    // M2 추가 — Radix 오버레이 통일
    tooltip:   300,    // M2 추가
    modal:     400,    // M2 추가
  },
  radius: {
    sm: '4px',
    md: '8px',
    lg: '12px',
    full: '9999px',
  },
}

## 컴포넌트별 토큰 사용 현황 (M6 기준 — 35개)

| 컴포넌트 | 사용 토큰 | 비고 |
|---|---|---|
| Button | brand, brandHover, surface, text, textDisabled, borderFocus | variant별 override |
| Badge | brand, brandSubtle, danger, dangerSubtle, text | |
| Select | border, surface, surfaceSub, text, overlay | Radix |
| Popover | surface, border, overlay, zIndex.overlay | Radix |
| Dialog | surface, surfaceOverlay, border, zIndex.modal | Radix |
| Tooltip | surfaceOverlay, textInverse, zIndex.tooltip | Radix |
| FormField | danger, dangerSubtle, border, borderFocus, text, textSub | 복합 컴포넌트 |
| DnD List | surface, border, brand | @dnd-kit |
| … | | |

## 토큰 추가 절차 (M4 신설)

새 semantic 토큰이 필요할 때 — **컴포넌트 코드에 직접 추가 금지**.

1. `design-system.md` "토큰 추가 절차" 섹션 확인
2. 유사한 기존 토큰이 없는지 검색 (`grep -r "tokens.color" src/`)
3. `src/tokens/semantic.ts`에 토큰 추가 + 주석 (추가 사유·사용 컴포넌트)
4. `design-system.md` "컴포넌트별 토큰 사용 현황" 표 갱신
5. PR에 "새 semantic 토큰 추가" 명시
```

---

## conventions.md 발췌 (M6 — 누적된 섹션들)

```markdown
## Radix 오버레이 z-index 규칙 (M2 추가)

Popover·Dialog·DropdownMenu·Tooltip 등 오버레이 컴포넌트는 반드시
`tokens.zIndex.overlay / .modal / .tooltip` 중 하나를 사용한다.

임의 숫자(`z-index: 9999`) 금지.

**왜**: Popover + Dialog 동시 열림 시 z-index 충돌 이력 발생 (M2 실제 버그).

## DnD 컴포넌트 선택 기준 (M2 추가)

신규 DnD 컴포넌트 추가 전 0-D(계획 수립)에서 반드시 확인:

| 조건 | 선택 |
|---|---|
| 리스트 정렬만 필요 | `@dnd-kit/core` (이미 사용 중 — 일관성 우선) |
| 칸반·트리·복잡 DnD | 기존 라이브러리 확장 검토 후 사용자 확인 |
| 새 DnD 라이브러리 도입 | **반드시 사용자 결정 후 진행** |

**왜**: DnD 라이브러리는 번들 크기 + peerDeps 충돌 위험이 크다.

## 릴리스 워크플로우 (M3 추가, M6 CI 자동화)

컴포넌트 추가·수정 완료 후 릴리스가 필요한 경우:

1. `pnpm changeset` — 변경 유형 선택 (patch/minor/major)
2. changeset 파일 커밋 + PR 생성
3. PR merge → CI가 `pnpm changeset version` + `pnpm publish` 자동 실행
4. GitHub Releases 확인

**수동 publish 금지** — M6 이후 CI 자동화로 전환됨.

## 컴포넌트 삭제 절차 (M3 추가)

컴포넌트 삭제 시 반드시:
1. `src/components/primitives/<name>/` 디렉토리 삭제
2. `src/components/primitives/index.ts` export 제거
3. `src/index.ts` export 제거
4. `design-system.md` 컴포넌트 표에서 제거
5. 해당 컴포넌트를 사용하는 stories 삭제

**왜**: export만 남기고 파일 삭제 시 `pnpm build` 오류 발생 이력 (M3).
```

---

## HANDOFF_NOW.md 패턴 (M6 — 안정화 상태)

M0에는 HANDOFF_NOW.md가 임시 메모 수준이었다. M6에서는 세션 시작 전 필독 문서로 자리잡았다.

```markdown
# acme-ui — HANDOFF_NOW

## §1 현재 상태

- **라이브러리 버전**: 2.4.1 (npm published)
- **컴포넌트 수**: 35개 (Radix 래핑 20개 + 단순 15개)
- **미완료 작업**: Combobox 키보드 네비게이션 개선 (M6 스프린트)
- **마지막 릴리스**: 2.4.1 — DatePicker 추가 (2026-04-28)
- **다음 릴리스 예정**: 2.5.0 — Combobox 개선 + 새 토큰 2개

## §2 다음 작업

1. Combobox `arrow key` 네비게이션 — M2 이후 미완 (#021)
2. `color.success` semantic 토큰 컴포넌트 적용 확인 (StatusBadge에 누락)
3. Storybook 10.x 마이그레이션 완료 후 CSF3 포맷 일괄 전환
```

---

## 6개월 후 자기 점검 (진단 기준)

| 지표 | 정상 | 주의 |
|---|---|---|
| CLAUDE.md 줄수 | 250~300줄 | 210줄 그대로면 M1 이후 함정 미기록 |
| STEP 1 강제 규칙 | 8개 이상 | 3개 이하면 운영 누적 안 된 신호 |
| design-system.md | 토큰 30개 이상·절차 섹션 있음 | 초안 수준이면 토큰 분산 위험 |
| HANDOFF.md | Session Update 20개 이상 | 없으면 `/session-close` 미실행 |
| 컴포넌트 수 | 초기 대비 10개 이상 증가 | 증가 없으면 라이브러리 운영 안 됨 |
| conventions.md | 10개 이상 섹션 | 3개 이하면 도메인 규칙 미기록 |
| 릴리스 수 | 10회 이상 | 1회 이하면 Changesets 미운영 |

**외부 사용자에게 가장 중요한 한 줄:**

> **이 문서가 보여주는 것은 6개월 운영의 결과물이지, 부트스트랩 직후 얻을 수 있는 것이 아니다.**
> M0 초기 상태(`examples/01-initial.md`)와 비교하면 — 모든 규칙은 실제 버그·충돌·비용에서 왔다.
> design-harness가 주는 것은 *발견을 기록하는 구조*이지 *발견된 내용 자체*가 아니다.
