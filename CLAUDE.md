# 프로젝트: AI Work Operating System (SPRINTOPS)

> 사용자가 나열한 작업 목록을 AI가 스프린트·실행 설계로 변환하고, 각 작업을 **사람 확인 필요 / AI 처리 가능**으로 분류한 뒤, 승인된 실행 작업은 Runner Server가 로컬 Claude Code 슬롯에서 수행하고 Monday/Slack 반영은 Backend가 담당하는 **AI 업무 운영 시스템**(Frontend + Backend + Runner Server). **모든 설계·결정·기능 명세는 아래 문서들이 단일 진실 공급원(SSOT)**. 코드 작성 전 관련 문서를 직접 읽어 최신 정합성을 확보한다.

## 용어 정의

- **SOLUTION_CODE**: `SPRINTOPS` — 솔루션(레포 전체) 식별자. 솔루션 공통 문서(`docs/ARCHITECTURE.md`, `docs/prd.md`)에서 사용.
- **SYSTEM_CODE** ≡ **APP_CODE** ≡ **{App}**: App(S/W 단위) 식별자. 아래 § Backend Services Overview 표가 단일 출처(SSOT). 본 솔루션은 **3개 App**(`FRONTEND` / `BACKEND` / `RUNNER`). App별 문서 ID 패턴 `{App}-PRD`, `{App}-FC`, `{App}-ARCHITECTURE`, `{App}-FRD-{NNN}`, `{App}-TASK-{NNN}`, `{App}-ADR-{NNN}`.

상세 식별자 규약은 문서 작성 가이드(추후 배치, § 진입 순서) §5 참조.

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-02 | 초안 — SPRINTOPS 솔루션 부트스트랩. 솔루션 PRD/tech_stack/ARCHITECTURE 등록, App 3종(FRONTEND/BACKEND/RUNNER) 레지스트리 등재 | jaecheon.jeong |

## 설계 문서 인덱스

| 영역 | 경로 | 역할 |
|---|---|---|
| **AI 진입점 (본 파일)** | `/CLAUDE.md` | SOLUTION_CODE / SYSTEM_CODE SSOT · Backend Services Overview · 라우터 |
| **솔루션 PRD** | [`docs/prd.md`](docs/prd.md) | 시스템 전체 배경/비전/문제/역할 분리/데이터 모델/플로우 SSOT |
| **기술 스택** | [`docs/tech_stack.md`](docs/tech_stack.md) | 서비스별 스택·라이브러리 선택 근거·단계별 도입 계획 SSOT |
| **솔루션 ARCHITECTURE** | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | 시스템 레벨 공통 룰 (서비스 경계·참조 매트릭스·보안 경계·데이터 흐름) |
| **문서 작성 룰** | 추후 배치 (`docs/DOCUMENT_GUIDE.md`) | 문서 작성 SSOT — 식별자/메타/변경 이력/SSOT 인용 패턴. 원본: `reference/slot-runner-main/docs/.templates/DOCUMENT_GUIDE.md` |
| **App: FRONTEND** | `docs/FRONTEND/` | App별 PRD/FC/ARCHITECTURE/FRD/TASK/ADR SSOT 폴더 |
| **App: BACKEND** | `docs/BACKEND/` | App별 PRD/FC/ARCHITECTURE/FRD/TASK/ADR SSOT 폴더 |
| **App: RUNNER** | 추후 생성 (`docs/RUNNER/`) | App별 PRD/FC/ARCHITECTURE/FRD/TASK/ADR SSOT 폴더 |
| **양식 원본 (외부 레퍼런스)** | [`reference/slot-runner-main/docs/.templates/`](reference/slot-runner-main/docs/.templates/) | claudecode-for-me Active 양식. 본 레포 문서 부트스트랩 시 복사 원본 (읽기 전용) |

> 본 솔루션은 **다중 App**이므로 솔루션 단일 PRD(`docs/prd.md`)를 채택한다. App별 상세 문서는 아직 미생성 — § 진입 순서 "신규 App 문서 부트스트랩" 절차로 생성.

## Backend Services Overview

본 솔루션의 App 레지스트리. **SYSTEM_CODE 단일 출처(SSOT)**. 신규 App 도입 시 본 표 행 추가가 모든 다른 작업(PRD/FC/FRD/TASK/ADR 작성)보다 선행. 서비스 경계·참조 룰은 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) SSOT.

| SYSTEM_CODE | 한 줄 설명 | 호스트 종류 | 런타임/스택 | 폴더 |
|---|---|---|---|---|
| FRONTEND | 좌측 작업 목록·상세·승인·Runner 상태를 다루는 작업 운영 콘솔 | 웹 SPA/SSR | Next.js · React · TypeScript · Tailwind · shadcn/ui · Zustand · TanStack Query | `docs/FRONTEND/` |
| BACKEND | 상태 원장 + AI 분석 + Human Check + Monday/Slack 연동 + Runner 오케스트레이션 코어 | REST API 서비스 | Python 3.12+ · FastAPI · Pydantic · SQLAlchemy 2.0 · Alembic · Redis+Dramatiq | `docs/BACKEND/` |
| RUNNER | 승인된 실행 작업을 로컬 Claude Code 슬롯에서 수행하는 Headless 실행 엔진 | 로컬 REST 서버/에이전트 | Rust · Axum · portable-pty · tokio · tracing · Claude Code CLI | 추후 생성 (`docs/RUNNER/`) |

> **불변식**(ARCHITECTURE §2.3 계승): Monday/Slack/AI API Token은 **BACKEND 전유**. RUNNER는 외부 협업 도구를 **절대 호출 금지**하고 실행 결과만 Backend에 반환. FRONTEND는 **BACKEND 하고만** 통신.

## 진입 순서

- 코드 작성 전 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) **§4.2 절대 금지 매트릭스**와 **§7 보안 경계**를 확인한다(서비스 경계 위반 방지).
- 기능 의도·역할 분리·데이터 모델은 [`docs/prd.md`](docs/prd.md), 스택·라이브러리는 [`docs/tech_stack.md`](docs/tech_stack.md)에서 확인한다.
- **신규 기능 작성 흐름**(App 문서 생성 후):
  1. `docs/{App}/{App}-PRD.md` 범위·기능 갱신
  2. `docs/{App}/{App}-FC.md` 5축 표 행 추가
  3. `docs/{App}/FRD/{App}-FRD-{NNN}.md` 신규 (코드 상세 금지 — 목적·흐름·정책·수용 기준)
  4. 필요 시 `docs/{App}/ADR/{App}-ADR-{NNN}.md` 등재 + `{App}-ADR-CATALOG.md` 동기화
  5. 구현 착수 전 최신 코드 기준 세부 설계 판단
- **AI 실행용 작업 지시서(TASK) 흐름** — feature/refactor/maintenance/migration/setup/investigation 통합:
  1. (사전) 영향 영구 SSOT(PRD/FC/FRD/ADR/ARCHITECTURE)를 작성자가 직접 갱신
  2. `docs/{App}/TASK/{App}-TASK-{NNN}.md` 신규 (휘발성·self-contained, 외부 SSOT 인용 금지 양방향)
  3. TASK §6 영향 SSOT 표에 갱신 상태 = "완료" 텍스트 선언
  4. TASK §12 컨텍스트 임베드 — 코드 실행에 필요한 계약·구조·정책 본문 임베드
  5. AI에게 TASK 던져 §8 실행. AI는 코드만 변경
- **신규 App 문서 부트스트랩**(FRONTEND/BACKEND/RUNNER 상세 문서 최초 생성):
  1. 본 파일 § Backend Services Overview 행은 이미 등재됨 — 폴더 경로 확정
  2. `docs/{App}/` + 하위 `FRD/`·`ADR/`·`TASK/` 생성
  3. `reference/slot-runner-main/docs/.templates/App/` 4종 복사·rename:
     - `APP-PRD-TEMPLATE.md` → `docs/{App}/{App}-PRD.md`
     - `APP-FC-TEMPLATE.md` → `docs/{App}/{App}-FC.md`
     - `APP-ARCHITECTURE-TEMPLATE.md` → `docs/{App}/{App}-ARCHITECTURE.md`
     - `APP-ADR-CATALOG-TEMPLATE.md` → `docs/{App}/{App}-ADR-CATALOG.md`
  4. App ARCHITECTURE는 솔루션 ARCHITECTURE(`docs/ARCHITECTURE.md`) 룰 준수 + 호스트 내부 레이어 특이사항만 명시.

## 개발 파이프라인

본 솔루션은 claudecode-for-me 플러그인 파이프라인 참고:
`grill-me → acceptance-design → meta-prompter → docs-add-task → forge-scope → doc-driven-review/ddr-loop → 반영`.

## 절대 변경 금지

- `reference/**` — 외부 레퍼런스(slot-runner-main 및 그 양식). **읽기 전용. 수정·삭제 금지**.
- `docs/prd.md`, `docs/tech_stack.md`, `docs/ARCHITECTURE.md` — 솔루션 SSOT. 사용자 승인 전 수정 금지.
- `/CLAUDE.md`(본 파일), `MEMORY.md` — 사용자 승인 전 수정 금지.
