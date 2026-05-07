---
tags: [design-harness, examples, 1개월]
updated: 2026-05-07
---

# Example 02 — M1: 1개월 운영 후

> **`acme-ui`** 라이브러리에 design-harness 적용 1개월 후 상태.
> 컴포넌트 8개 추가, 첫 함정 3개 발견·기록.
> CLAUDE.md가 150줄 → 210줄로 성장했고, STEP 1에 도메인 강제 규칙이 생기기 시작했다.

---

## 1개월간 무슨 일이 있었나

| 날짜 | 이벤트 | CLAUDE.md 변화 |
|---|---|---|
| M0+1주 | Button에 `asChild` prop 추가 → Radix Slot 충돌 첫 발견 | STEP 1에 "Radix Slot 규칙" 추가 |
| M0+2주 | Select 컴포넌트 — core 토큰 직접 참조 버그 리포트 | design-system.md 토큰 목록 보강 |
| M0+3주 | FormField 컴포넌트 — label + input + error 조합 패턴 정립 | conventions.md에 "복합 컴포넌트" 섹션 추가 |
| M0+4주 | Storybook 시각 검증 중 dark mode 미반영 발견 | STEP 2에 "dark mode Story 확인" 항목 추가 |

---

## CLAUDE.md 발췌 (M1 — 210줄, 변경 부분만)

### STEP 1 — 강제 규칙 (추가된 규칙)

```markdown
## 🔴 STEP 1 — 강제 규칙

| 규칙 | 발견 시점 | 위반 시 |
|---|---|---|
| **5개 산출물 동시 작성** | M0 | 작업 중단 → 누락 파일 추가 |
| **semantic 토큰만 사용** | M0 | 작업 중단 → 토큰 교체 |
| **루트 export 추가** | M0 | 작업 중단 → export 추가 |
| **Radix Slot 컴포넌트에 `asChild` prop 추가 시** — `React.forwardRef` 필수, `displayName` 명시 | M0+1주 | 런타임 에러 발생 이력 — 건너뜀 금지 |
| **복합 컴포넌트** (FormField 등) — 하위 컴포넌트는 `<ParentName>.<ChildName>` 패턴 | M0+3주 | 네이밍 일관성 파괴 |
| **Storybook Story** — Default + 모든 variant + dark mode Story 필수 | M0+4주 | dark mode 미반영 이력 |
```

### STEP 2 — 검증 하네스 (추가된 항목)

```markdown
```
[규칙 준수 체크]
- [ ] 5개 산출물 전부 작성됨
- [ ] semantic 토큰만 사용 (core 토큰 직접 참조 없음)
- [ ] src/index.ts export 추가됨
- [ ] pnpm test 통과
- [ ] pnpm build 통과
- [ ] pnpm lint 통과
- [ ] Storybook Default + variants + dark mode Story 시각 확인   ← M0+4주 추가
- [ ] (Radix 컴포넌트) asChild 구현 시 forwardRef + displayName 확인  ← M0+1주 추가
```
```

---

## design-system.md 발췌 (M1 — 보강됨)

```markdown
# acme-ui — 디자인시스템

## 토큰 2계층 구조

### Core 토큰 (src/tokens/core.ts)
> 절대 직접 참조 금지. Semantic 토큰이 소비한다.

```ts
// core.ts
export const color = {
  neutral: { 0: '#fff', 50: '#f9f9f9', … 900: '#111' },
  brand:   { 50: '#eff6ff', 500: '#3b82f6', 900: '#1e3a8a' },
  error:   { 50: '#fef2f2', 500: '#ef4444' },
}
```

### Semantic 토큰 (src/tokens/semantic.ts)
> 컴포넌트에서 이것만 참조.

```ts
// semantic.ts
export const tokens = {
  color: {
    brand:        color.brand[500],
    brandHover:   color.brand[600],
    surface:      color.neutral[0],
    surfaceSub:   color.neutral[50],
    text:         color.neutral[900],
    textSub:      color.neutral[500],
    border:       color.neutral[200],
    danger:       color.error[500],
  },
}
```

## 컴포넌트별 토큰 사용 현황

| 컴포넌트 | 사용 토큰 | 비고 |
|---|---|---|
| Button | brand, brandHover, surface, text | variant별 override |
| Badge | brand, danger, surface, text | |
| Select | border, surface, surfaceSub, text | Radix |
| FormField | danger, border, text, textSub | 복합 컴포넌트 |
| … | | |

## 자주 쓰는 토큰

```ts
import { tokens } from '../tokens/semantic'

// ✅ 올바른 사용
style={{ color: tokens.color.text }}
style={{ background: tokens.color.surface }}

// ❌ 금지
style={{ color: '#111' }}          // 하드코딩
style={{ color: color.neutral[900] }} // core 직접 참조
```
```

---

## conventions.md 발췌 (M1 — 추가된 섹션)

```markdown
## 복합 컴포넌트 패턴 (M0+3주 추가)

FormField처럼 여러 하위 요소가 조합되는 컴포넌트는 `<Parent>.<Child>` 네임스페이스 패턴을 사용한다.

```tsx
// FormField.tsx
export function FormField({ children }: FormFieldProps) { … }
FormField.Label = FormFieldLabel
FormField.Input = FormFieldInput
FormField.Error = FormFieldError

// 사용
<FormField>
  <FormField.Label>이름</FormField.Label>
  <FormField.Input />
  <FormField.Error />
</FormField>
```

**왜**: 하위 컴포넌트가 독립 export될 경우 네이밍 충돌 + 의존관계 불명확 이력 발생.

## Radix 래핑 컴포넌트 (M0+1주 추가)

`asChild`를 지원하는 컴포넌트는 반드시:
1. `React.forwardRef` 사용
2. `displayName` 명시 (`ComponentName.displayName = 'ComponentName'`)

```tsx
// ✅ 올바른 패턴
const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ asChild, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button'
    return <Comp ref={ref} {...props} />
  }
)
Button.displayName = 'Button'
```

**왜**: `forwardRef` 없으면 Radix Slot이 ref를 통과시키지 못해 런타임 에러 발생 (M0+1주 실제 버그).
```

---

## 1개월 후 자기 점검 (진단 기준)

| 지표 | 정상 | 주의 |
|---|---|---|
| CLAUDE.md 줄수 | 180~220줄 | 150줄 그대로면 함정 미기록 신호 |
| STEP 1 강제 규칙 | 3개 이상 추가됨 | 0개 추가면 운영 누적 안 된 신호 |
| design-system.md | 토큰 표 채워짐 | 여전히 초안이면 컴포넌트 추가 중 토큰 미기록 |
| HANDOFF.md | Session Update 4개 이상 | 없으면 `/session-close` 미실행 |
| 컴포넌트 수 | 초기 대비 3~10개 증가 | 증가 없으면 라이브러리 운영 안 됨 |

**M1 `CLAUDE.md`를 `examples/01-initial.md`와 비교하면 본인 운영 강도가 보인다.**
