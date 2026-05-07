# Example — `acme-ui` (React 디자인시스템 라이브러리)

design-harness 첫 적용 사례 (마스킹·일반화). 외부자에게 "라이브러리에 design-harness가 어떻게 자리잡는지"를 보여주는 자료다.

---

## 1. 적용 프로젝트 개요

| 항목 | 값 |
|---|---|
| **이름** | `acme-ui` (가명. 실제 적용 사례에서 마스킹) |
| **유형** | React 디자인시스템 라이브러리 (단일 패키지, npm publish 대상) |
| **컴포넌트 수** | 22개 (Radix 래핑 14개 + 단순 8개) |
| **빌드** | tsup 8.x — ESM + CJS dual + `dts: true` |
| **타입** | TypeScript 6.x strict |
| **패키지 매니저** | pnpm 9.x |
| **린터·포매터** | biome 2.x |
| **테스트** | vitest 4.x + Testing Library |
| **시각 검증** | Storybook 10.x (react-vite) |
| **버전 관리** | Changesets |
| **이슈 트래커** | 미도입 (이슈 번호 강제 규칙 비활성) |
| **브랜치 전략** | Git Flow (`main`·`develop` 보호) |

---

## 2. 시뮬레이션 흐름

총 3단계, 약 30분 소요.

### 2-1. `install.sh --local` 실행

```bash
cd ~/projects/acme-ui
~/Documents/Dev-Vault/Dev-Vault/design-harness/install.sh --local
```

결과: 프로젝트 루트에 `.claude/skills/`·`.claude/agents/` 복사 + `CLAUDE.md` 미존재 안내.

### 2-2. `/project-init` Q&A

design-harness `skills/project-init/SKILL.md` 가 라이브러리 시그널 자동 감지:

```
[감지] tsup.config.ts 존재 → 라이브러리 빌드
[감지] package.json exports 필드 → 다중 진입점
[감지] peerDependencies: react ^19, react-dom ^19 → React 라이브러리
[감지] .storybook/ 존재 → 시각 검증 = Storybook
[감지] .changeset/ 존재 → Changesets 릴리스
[감지] biome.json 존재 → 포매터 = biome
```

이어서 Q&A:
- Q1 라이브러리명 → `acme-ui`
- Q2 카테고리 → A. 디자인시스템
- Q3 빌드 도구 → tsup (감지 확인)
- Q4 시각 검증 → Storybook (감지 확인)
- Q5 이슈 트래커 → 미도입
- Q6 브랜치 전략 → Git Flow
- Q7 컴포넌트 디렉토리 → `src/components/primitives/`
- Q8 토큰 디렉토리 → `src/styles/tokens/` (core + semantic 2-tier)
- Q9 핵심 규칙 → "5개 산출물 동시 작성 / semantic 토큰만 / 루트 index.ts export"

### 2-3. 컴포넌트 추가 1회 (검증 시나리오)

`Tooltip` 컴포넌트(Radix 래핑) 신규 추가로 워크플로우 검증:

1. `acme-ui-dev` 호출 → agent/architecture·conventions·design-system 읽음
2. 유사 컴포넌트 2개 분석 (`accordion`, `dialog` — 둘 다 Radix 래핑)
3. 구현 계획 제시 → 사용자 확인
4. 5개 산출물 동시 작성 (아래 §5 시연)
5. `pnpm test` · `pnpm build` · `pnpm lint` 통과
6. `pnpm storybook` 으로 사용자 직접 시각 확인
7. `acme-ui-doc-writer` 호출 → HANDOFF 3종 갱신 (architecture.md 컴포넌트 표는 트리거 미충족 → 미갱신)

---

## 3. 산출물 요약

```
acme-ui/
├── CLAUDE.md                              # 219줄 — design-harness CLAUDE.md 템플릿 채움
├── .claude/
│   ├── agents/
│   │   ├── acme-ui-dev.md                 # 유형 4 — 라이브러리 dev
│   │   └── acme-ui-doc-writer.md          # 유형 5 — 라이브러리 doc-writer
│   └── skills/                            # design-harness 스킬 6개 (project-init 등)
└── docs/
    └── acme-ui/
        ├── status/HANDOFF_NOW.md          # 현재 상태·다음 작업 (60줄 이하)
        ├── history/세션_노트.md           # 세션 1~2줄 요약 누적
        ├── plans/HANDOFF.md               # Session Update 누적
        ├── agent/
        │   ├── architecture.md            # 라이브러리 6섹션 (개요·exports·peerDeps·디렉토리·컴포넌트 패턴·빌드)
        │   ├── conventions.md             # 컴포넌트 선언 순서·테스트·스토리 패턴
        │   └── design-system.md           # 2-tier 토큰·variant 패턴·자주 쓰는 토큰 (핵심 문서)
        ├── components/                    # 복잡한 컴포넌트만 (예: dnd-list.md)
        └── git-workflow/branch-commit.md  # Git Flow + 컨벤셔널 커밋
```

CLAUDE.md 분량(~220줄)은 본체 하네스 앱 사례(Acme Platform — 약 290줄)보다 작다. 단일 패키지·페이지 없음·UI 검증 통합으로 인해 자연스럽게 컴팩트.

---

## 4. 발견된 갭과 OSS 보강 결정

본체 하네스(앱 가정)를 라이브러리에 적용하며 갭 **12건** 발견. 분리 정책에 따라 본체 보강 vs design-harness 신설로 나눔.

| # | 갭 | 분류 | 처리 |
|---|---|---|---|
| G1 | install.sh 출력 깨짐 (변수 확장 실패) | 본체 버그 | 본체 v1.1.0 보강 |
| G2 | install.sh `--local` 완료 메시지가 글로벌 경로 안내 | 본체 버그 | 본체 v1.1.0 보강 |
| G3 | `/project-init` Q2 영역 분류에 라이브러리/CLI 누락 | 라이브러리 도메인 | design-harness 신설 |
| G4 | 기존 `docs/*` 사전 자료 처리 가이드 부족 | 보편 | 본체 v1.1.0 보강 |
| G5 | `pnpm-workspace.yaml` 모노레포 미감지 | 보편 | 본체 v1.1.0 보강 |
| G6 | 라이브러리에 `pages/` 폴더 부적합 (`components/` 권장) | 라이브러리 도메인 | design-harness 신설 |
| G7 | CLAUDE.md 템플릿 다중 패키지 가정 | 보편 | 본체 v1.1.0 보강 |
| G8 | `architecture.md` 라이브러리 항목 부재 | 라이브러리 도메인 | design-harness 신설 |
| G9 | Step 7 훅 라이브러리 부적합 | 라이브러리 도메인 | design-harness 신설 (훅 자동 스킵) |
| G10 | UI 검증·FE 기능정리 라이브러리 부적합 (Storybook 갈음) | 라이브러리 도메인 | design-harness 신설 (컴포넌트 정리 템플릿 신규) |
| G11 | branch-commit 이슈 번호 강제 | 보편 | 본체 v1.1.0 보강 (옵션화) |
| G12 | 영역 브래킷 `[FE]/[BE]/[공통]` 단일 패키지 불필요 | 보편 | 본체 v1.1.0 보강 (옵션화) |

**분리 사유**: 도메인이 거의 겹치지 않음 (라이브러리는 페이지 없음·UI 검증 다름·핵심 자산 다름). 본체에 라이브러리 분기를 통합하면 SKILL.md 부풀림 + 외부자가 자기 케이스 외 안내 노이즈를 봐야 함.

---

## 5. 5개 산출물 동시 작성 워크플로우 시연

`Tooltip` 컴포넌트 추가 1회 — 라이브러리 핵심 강제 규칙이 어떻게 작동하는지.

### Step 1. 분석 (acme-ui-dev)

```
[유사 컴포넌트 2개 확인]
- accordion (Radix 래핑) — Radix.Root·Trigger·Content 분리, forwardRef, displayName
- dialog (Radix 래핑) — 동일 패턴

[구현 계획]
- 신규 파일 4개 + 수정 1개
  · src/components/primitives/tooltip/tooltip.tsx
  · src/components/primitives/tooltip/tooltip.test.tsx
  · src/components/primitives/tooltip/index.ts
  · stories/tooltip.stories.tsx
  · src/components/primitives/index.ts (export 추가)
- variant: 단일 (variant 시스템 불필요)
- semantic 토큰: bg-bg-inverse, text-text-inverse, border-border-default
- a11y: Radix Tooltip 기본 ARIA 사용, role="tooltip"
```

사용자 확인 후 진행.

### Step 2. 5개 산출물 동시 작성

```ts
// 1) tooltip.tsx (본체)
import * as TooltipPrimitive from "@radix-ui/react-tooltip";
import { cn } from "../../../utils/cn";
import { forwardRef } from "react";

export const Tooltip = TooltipPrimitive.Root;
export const TooltipTrigger = TooltipPrimitive.Trigger;
export const TooltipProvider = TooltipPrimitive.Provider;

export const TooltipContent = forwardRef<
  React.ElementRef<typeof TooltipPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof TooltipPrimitive.Content>
>(({ className, sideOffset = 4, ...props }, ref) => (
  <TooltipPrimitive.Content
    ref={ref}
    sideOffset={sideOffset}
    className={cn(
      "z-50 rounded-md bg-bg-inverse px-3 py-1.5 text-sm text-text-inverse",
      "border border-border-default shadow-md",
      className
    )}
    {...props}
  />
));
TooltipContent.displayName = TooltipPrimitive.Content.displayName;
```

```ts
// 2) tooltip.test.tsx (vitest + Testing Library)
// 3) index.ts (재export)
// 4) stories/tooltip.stories.tsx (Default + 위치별 스토리)
// 5) src/components/primitives/index.ts (export 추가)
//    export * from "./tooltip";
```

### Step 3. 검증

```bash
pnpm test    # 23 files / 56 tests passed
pnpm build   # dist/ 갱신 — exports 정상
pnpm lint    # biome check passed
pnpm storybook  # 사용자 직접 확인 (localhost:6006)
```

### Step 4. doc-writer 후속 처리 (acme-ui-doc-writer)

- `HANDOFF_NOW.md` §1·§2 갱신
- `세션_노트.md` 최상단 prepend
- `HANDOFF.md` Session Update 추가
- `architecture.md` "5. 컴포넌트 패턴" Radix 래핑 예시에 `Tooltip` 추가 고려 → **트리거 미충족(빌드·peerDeps 변동 없음)으로 갱신 생략**
- `design-system.md` 토큰 사용처 grep — 변경 없음
- `components/tooltip.md` — 단순 컴포넌트라 작성 생략 (Storybook + 테스트로 충분)

---

## 6. 시뮬레이션이 검증한 것

본 사례가 본체 하네스와 design-harness의 분리를 정당화한 핵심 시그널:

| 시그널 | 앱 하네스 | 라이브러리 하네스 (design-harness) |
|---|---|---|
| 작업 영역 단위 | 패키지 | 코드 카테고리 (컴포넌트·토큰·빌드·유틸) |
| 진입 문서 | HANDOFF_NOW만 | HANDOFF_NOW **+ agent/3종** (design-system 핵심) |
| STEP 1 강제 규칙 | API 레이어·이슈 번호·브랜치 보호 | **N개 산출물 + semantic 토큰 + 루트 export** + 브랜치 보호 (이슈 번호 옵션) |
| UI 검증 | Playwright validator 자동 검증 | **Storybook** 시각 확인 (사용자 직접) |
| 핵심 문서 | FE_*.md (페이지·기능 단위) | **design-system.md + components/<...>.md** (복잡한 것만) |
| 갱신 트리거 | 페이지 추가·API 변경 | peerDeps·exports·variant·토큰 추가 |

→ 단일 OSS에 분기로 처리하면 SKILL.md·CLAUDE.md 템플릿이 두 케이스를 모두 책임져야 해서 외부자에게 노이즈. 별도 OSS로 분리해 각자의 도메인 답안만 제공하는 것이 명확하다.

---

## 7. 외부자가 본 사례를 따라할 때

본 OSS를 자기 라이브러리에 적용한다면:

1. design-harness `install.sh` 실행 → `.claude/` 복사
2. `/project-init` Q&A에 답하면서 본 문서 §2-2의 시그널 감지 결과를 비교
3. 산출물 트리(§3)와 비교해 누락된 폴더 채우기
4. 첫 컴포넌트 추가 시 §5 워크플로우를 그대로 따라 실행
5. 이슈 트래커·브랜치 전략·tsup/rollup 등은 본인 환경에 맞춰 CLAUDE.md FILL 주석 부분만 교체

본 사례에서 **하지 않은 것**도 명시:
- ❌ Playwright validator 생성 (Storybook이 갈음)
- ❌ FE_*.md 페이지 문서 (`pages/` 폴더 미사용)
- ❌ 모든 컴포넌트마다 `components/<...>.md` 작성 (복잡한 것만)
- ❌ 이슈 번호 강제 (트래커 미도입 → 비활성)
