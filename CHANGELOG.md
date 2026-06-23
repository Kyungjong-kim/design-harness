---
tags: [design-harness, changelog]
updated: 2026-06-23
---

# CHANGELOG

design-harness 문서·스킬·스크립트의 주요 변경 이력.

---

## 2026-06-23 (v0.3.1 — 성숙 단계 자산: 문서 트리·Storybook 정비)

> 라이브러리 성숙기(컴포넌트 60+·외부 노출)에 필요한 "정보 구조·시각 카탈로그·접근성" 자산을 추가. 실제 적용(`acme-ui` 2차)에서 도출.

### 신규 — `Claude Code/템플릿/디자인시스템_문서트리_템플릿.md`

- `docs/design-system/` 캐노니컬 트리 골격(foundation·tokens·components·patterns) + SoT 규칙 + 작성 원칙 + 마스킹 절차
- `agent/design-system.md`(quickref)와의 분리 기준, CLAUDE.md 하네스 배선 안내

### 신규 — `Claude Code/Storybook_정비_가이드.md`

- 카테고리 택소노미 고정·storySort·autodocs 전수·MDX 랜딩 체크리스트
- 함정: 컴포넌트 `<table>` 렌더 시 Docs 페이지 마크다운 표 CSS 누수 → `.docs-story` 제외 해결
- autodocs = "한눈에 보기" 갤러리 → 별도 인앱 쇼케이스 불필요 명시

### 갱신 — `examples/ui-kit-case.md`

- §8 "2차 적용 — 문서 트리·Storybook 정비" 추가: 성숙 단계 워크플로우 + 자가평가 게이트(a11y 블로커 보강) 교훈

---

## 2026-06-19 (v0.3.0 — Windows 네이티브 설치기 install.ps1)

> claude-code-harness의 검증된 PowerShell 포팅을 design-harness에 적용. install.sh가 동일하므로 같은 ps1을 사용.

### 추가 — `install.ps1` (install.sh의 PS 5.1 1:1 포팅)

- 실제 `python` 탐지(Windows Store `python3` 스텁 회피), `cp`/`rm` 대신 `robocopy /MIR`, UTF-8 BOM, 임시 `.py` + 인자 전달로 한글 경로 보존
- 플래그: `-Local`·`-Personal`·`-Hooks`·`-Force`·`-Yes`·`-Dir`
- 🧪 실험적 — 임시 디렉터리 dry run 확인, 실환경 Windows 검증 진행 중. 완전 검증 경로는 WSL2
- README en/ko에 Windows native 부트스트랩 안내 추가

## 2026-04-30 (2차 — v0.2.0 — 라이브러리 전용 템플릿·가이드·예시 완성)

> 1라운드의 메타·SKILL.md 위에 **라이브러리 전용 답안**을 채운 라운드. 외부자가 design-harness만으로 라이브러리 프로젝트에 하네스를 끝까지 적용 가능한 상태로 도달.

### 변경 — `Claude Code/템플릿/CLAUDE.md_템플릿.md` 라이브러리 전용 재작성

- 헤더에 라이브러리 프로젝트 유형 가정 + 모노레포 안 라이브러리 사용 안내 추가
- 분석 전 필수 확인 → `exports`·`peerDeps`·tsup·rollup·Storybook·Changesets·biome 등 라이브러리 시그널
- 0-A 작업 영역 → "코드 카테고리" 단위 (디자인시스템·SDK 두 가지 FILL 예시)
- 0-B 진입 문서 → HANDOFF_NOW + agent/3종 (architecture·conventions·design-system)
- STEP 1 강제 규칙 → 라이브러리 핵심 3종(N개 산출물·semantic 토큰·루트 export) + 이슈 트래커 옵션 + Git Flow/trunk-based 옵션
- STEP 2 검증 → test/build/lint + Storybook 시각 확인 (Playwright 섹션 제거)
- STEP 3 세션 종료 → 단일 패키지 docs 경로 + 컴포넌트 정리 문서 작성 검토 안내
- 서브에이전트 표 → `<라이브러리>-dev` / `<라이브러리>-doc-writer`
- 팀 프로세스 → 1행 통합 + 릴리스 절차 추가
- 브랜치·커밋 핵심 규칙 → Git Flow + 컨벤셔널 커밋 기본값, 다른 옵션 FILL 주석

### 변경 — `Claude Code/템플릿/agent_문서_템플릿.md` 라이브러리 전용 재작성

- 헤더 표에 작성 대상 컬럼 추가 — design-system.md는 디자인시스템·UI 라이브러리만 작성
- 분석 전 확인 → "라이브러리 시그널" 단일 목록 (FE/BE 분기 제거)
- §1 architecture → 라이브러리 6섹션 표준 구성(개요·exports·peerDeps·디렉토리·컴포넌트 패턴·빌드 흐름)
- §2 conventions → 컴포넌트 파일 내 선언 순서(cva → interface → 함수 → displayName) / 테스트·스토리 패턴 신설
- §3 design-system → **핵심 문서 격상**. 7섹션(2-tier 토큰·core/semantic 카테고리·작업 원칙·variant 패턴·새 토큰 추가 절차·자주 쓰는 토큰)

### 추가 — `Claude Code/템플릿/컴포넌트_정리_템플릿.md` 신규

- 본체 `FE_기능정리_템플릿.md` 라이브러리 대체본
- 작성 정책 — 강제 아님, 1차 문서는 Storybook + 단위 테스트, 본 문서는 보완용
- 9섹션 — 개요·API·variant 매트릭스·사용 예시·의존·접근성·관련 토큰·테스트 시나리오·변경 이력
- 선택 섹션 표시 — variant 매트릭스(*cva 사용 시*) / 합성·제어 사용 예시 / 접근성 / 관련 토큰(*디자인시스템 한정*) / 변경 이력
- 파일 경로 디폴트 — `docs/<라이브러리명>/components/<kebab-case>.md`

### 변경 — `Claude Code/에이전트_정의_가이드.md` 라이브러리 보강

- 헤더 → "라이브러리 전용" 명시 + design-harness 적용 안내
- **유형 4 — 라이브러리 컴포넌트·토큰 dev** 신설 (`<라이브러리명>-dev`)
- **유형 5 — 라이브러리 doc-writer** 신설 (`<라이브러리명>-doc-writer`, agent/3종 + components/ 트리거 처리)
- 최소 필수 에이전트 세트 표 → 라이브러리 행 2개 추가 + Playwright validator는 라이브러리 단독 불필요 안내
- 결정 흐름 → 라이브러리 / FE·풀스택 / BE / 모노레포 4-way 분기로 확장
- 초기 구축 질문 목록 → Q1~Q6 재구성 (라이브러리 vs 앱 분기, 시각 검증 도구 옵션화)

### 추가 — `examples/ui-kit-case.md` 신규

- design-harness 첫 적용 사례 마스킹·일반화 (`ui-kit` → `acme-ui`)
- 7섹션 — 프로젝트 개요 / 시뮬레이션 흐름(install→/project-init→컴포넌트 추가) / 산출물 트리 / 갭 12건 표 / 5개 산출물 동시 작성 시연(`Tooltip` 컴포넌트) / 분리 정당화 시그널 비교표 / 외부자 적용 안내
- 약 220줄

### 추가 — `examples/` 디렉토리 신설

본 OSS의 examples 디렉토리 신설. 향후 3라운드(선택)에서 `01-initial.md`·`02-after-1-month.md`·`03-after-6-months.md` 추가 검토.

### 변경 — `VERSION` 0.1.0 → 0.2.0

semver minor 증가. functional addition + non-breaking. 라이브러리 OSS의 "사용 가능 첫 완성 상태" 도달.

---

## 2026-04-30 (1차 — v0.1.0 — 초기 분기 + 1라운드 골자)

> 본체 `claude-code-harness`에서 **라이브러리·디자인시스템 도메인을 분리**해 신설한 OSS. 본체는 앱(FE/BE/풀스택) 중심으로 유지.

### 결정 — 본체와 분리한 이유

- 앱 하네스가 가정하는 페이지·라우팅·UI 검증(Playwright)·`pages/<도메인>/FE_*.md` 패턴이 라이브러리에 부적합
- 라이브러리는 컴포넌트·토큰·번들 도메인 — 다른 워크플로우 (5개 산출물 동시 작성·semantic 토큰 강제·Storybook 검증)
- 한 SKILL.md에 모두 분기로 처리하면 외부자가 자기 케이스 외의 안내를 계속 봐야 함 → 사용성 저해

### 추가 — 디렉토리 구조

- 본체에서 공용 자산 복사 — `skills/` (6개), `Claude Code/템플릿/` · `Claude Code/` 가이드, `install.sh`, `LICENSE`, `CONTRIBUTING.md`
- 라이브러리 OSS 무관 자산 제거 — `personal-skills/` (개인 노트 영역과 별도), `hooks/` (페이지 훅 패턴 부적합)

### 추가 — 메타

- `VERSION` (0.1.0)
- `README.md` (영어, 짧게) + `README.ko.md` (한국어 본체)
- `CHANGELOG.md` (이 문서)
- `.gitignore`

### 변경 예정 (1라운드-2)

- `skills/project-init/SKILL.md` — 라이브러리 전용으로 재작성. Step 1 자동 감지(tsup·rollup·peerDeps 시그널), Q2 라이브러리 카테고리, Step 3 `components/` 구조, Step 5 architecture(빌드/exports/peerDeps), Step 7 훅 자동 스킵, Step 9 design-system 강화 + Storybook

### 변경 예정 (2라운드 이후 — 별도 세션 가능)

- `Claude Code/템플릿/CLAUDE.md_템플릿.md` 라이브러리 전용 변형
- `Claude Code/템플릿/agent_문서_템플릿.md` architecture를 빌드/exports/peerDeps 중심으로
- `Claude Code/템플릿/컴포넌트_정리_템플릿.md` 신규 — `pages/<도메인>/FE_<기능>.md` 대체
- `Claude Code/에이전트_정의_가이드.md` — 라이브러리 dev / doc-writer 예시 추가
- `examples/ui-kit-case.md` — 본 OSS의 첫 적용 사례(`ui-kit`) 마스킹·일반화

> 2라운드 상세 plan: `<career>/하네스-검증/2026-04-30-design-harness-roadmap.md`
