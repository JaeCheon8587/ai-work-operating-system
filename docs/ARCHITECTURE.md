# 시스템 아키텍처 (AI Work Operating System)

가칭: **SprintOps AI**

> 본 파일은 **솔루션(시스템) 레벨** 아키텍처 SSOT다. 서비스 경계·책임·참조 방향·보안 경계·데이터 흐름의 단일 규칙 문서이며, 충돌 시 본 문서가 우선한다.
> 각 App(Frontend / Backend / Runner Server) **내부 레이어 규칙**은 본 문서 범위 밖이며, 향후 `docs/{App}/{App}-ARCHITECTURE.md` 로 위임한다(§3.6).
> 기준 문서: [PRD](prd.md) · [기술 스택](tech_stack.md).

| 항목 | 값 |
|---|---|
| 문서 ID | ARCHITECTURE (솔루션 단일 파일) |
| 버전 | 0.1 (Draft) |
| SOLUTION_CODE | SPRINTOPS |
| 작성 가정 | 3-서비스 분산 시스템(Frontend / Backend / Runner Server) + 공통 인프라(DB / Queue / Storage). 폴리글랏(TS / Python / Rust). 진입점 호스트 3개. |
| 관련 문서 | [PRD](prd.md) · [tech_stack](tech_stack.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-02 | 초안 — 시스템 레벨 아키텍처 부트스트랩(서비스 경계·참조 매트릭스·보안 경계) | jaecheon.jeong |

---

## 1. 문서 목적과 범위

### 1.1 목적

이 문서는 AI 코딩 에이전트와 신규 개발자가 SprintOps AI 시스템에 코드를 추가할 때, **서비스 경계를 위반하지 않도록** 반드시 지켜야 하는 시스템 레벨 규칙 문서다.

- 모든 규칙은 **반드시** / **허용** / **금지** / **절대 금지** 중 하나로 판정한다.
- 이 문서만 읽고 "어떤 서비스가 무엇을 담당하고, 무엇을 담당하면 안 되는지"를 결정한다.
- 서비스 **내부** 레이어(Domain / Application / Infrastructure / Presentation) 배치 판정은 각 App ARCHITECTURE로 위임한다(§3.6).

### 1.2 적용 범위

- 3개 실행 서비스: **Frontend**, **Backend**, **Runner Server**.
- 3개 공통 인프라: **Database**, **Queue**, **Storage**.
- 서비스 간 통신 경로, 데이터 소유권, 통합(Monday/Slack) 소유권, 실행 주체 라우팅, 보안 경계.

### 1.3 제외 범위

- 각 App 내부 코드 레이어 규칙 → `docs/{App}/{App}-ARCHITECTURE.md`.
- 기능 명세 → [PRD](prd.md). 라이브러리 선택 근거 → [tech_stack](tech_stack.md).
- 빌드/배포/CI(`docker-compose`, `Dockerfile`, `Cargo.toml`, `pyproject.toml`, `next.config`) 설정.

---

## 2. 시스템 아키텍처

### 2.1 시스템 토폴로지

```text
                        ┌─────────────────────────┐
                        │        Frontend         │  Next.js / React / TS
                        │  (작업 운영 콘솔)         │  Zustand · TanStack Query
                        └────────────┬────────────┘
                                     │ HTTPS REST + SSE
                                     │ (Frontend → Backend 만)
                                     ▼
        ┌──────────────────────────────────────────────────────────┐
        │                         Backend                           │  Python / FastAPI
        │              상태 원장 + 업무 운영 코어                     │  Pydantic · SQLAlchemy 2.0
        │                                                            │
        │  Workspace · AI Planner · Execution Classifier ·          │
        │  Human Check · Runner Orchestrator · AI Tracker ·         │
        │  Monday Adapter · Slack Adapter                           │
        └──┬─────────┬───────────┬──────────────┬─────────────┬─────┘
           │         │           │              │             │
   REST/Pull│   async │      httpx│         httpx│       env/secret
           │         │           │              │             │
           ▼         ▼           ▼              ▼             ▼
   ┌───────────┐ ┌────────┐ ┌─────────┐  ┌─────────┐   (AI API Key,
   │  Runner   │ │ Queue  │ │ Monday  │  │  Slack  │    Monday/Slack
   │  Server   │ │ Redis+ │ │  API    │  │   API   │    Token 은
   │  (Rust)   │ │Dramatiq│ └─────────┘  └─────────┘    Backend 전유)
   │           │ └────────┘
   │ Slot Pool │      │
   │ PTY(Claude│      ▼
   │  Code CLI)│ ┌──────────┐     ┌──────────┐
   │ Artifact  │ │ Database │     │ Storage  │
   │ Collector │ │PostgreSQL│     │ 파일/로그 │
   └───────────┘ └──────────┘     └──────────┘
```

핵심: **Frontend 는 Backend 하고만 통신**한다. **Monday / Slack / AI API 는 Backend 만** 호출한다. **Runner Server 는 Backend 하고만 통신**하며 외부 협업 도구를 절대 건드리지 않는다.

### 2.2 구성요소 매핑

| 구성요소 | 역할 | 스택 | 소유 데이터/자원 |
|---|---|---|---|
| **Frontend** | 사용자 작업 운영 콘솔 (표시·입력·승인 트리거) | Next.js, React, TS, Tailwind, shadcn/ui, Zustand, TanStack Query | 세션 UI 상태만. 영속 데이터 없음 |
| **Backend** | 상태 원장 + AI 분석 + Human Check + 통합 소유 + Runner 오케스트레이션 | Python 3.12+, FastAPI, Pydantic, SQLAlchemy 2.0, Alembic, httpx | 전체 도메인 상태(SSOT), AI/Monday/Slack Token |
| **Runner Server** | 승인된 실행 작업을 로컬 Claude Code 슬롯에서 수행 | Rust, Axum, portable-pty, tokio, tracing, Claude Code CLI | 로컬 슬롯 상태, 실행 중 Job, 로컬 프로젝트 경로 |
| **Database** | 관계형 상태 + AI 원문(JSONB) + 이벤트/감사 로그 | PostgreSQL | 영속 상태 원장 |
| **Queue** | 비동기 작업 처리(AI 분석·Runner dispatch·통합·요약·회고) | Redis + Dramatiq(우선) / Celery | 인플라이트 작업 |
| **Storage** | Runner 산출물·로그·생성 문서 | Local File Storage → 추후 S3/MinIO/R2 | 산출물 파일 |

### 2.3 핵심 아키텍처 원칙 (PRD §7 계승)

1. **통합 단일 소유(Integration Ownership).** Monday / Slack / AI API 호출은 **반드시** Backend가 단독 수행한다. 다른 서비스는 통합 Token을 **절대** 보유·호출 **금지**.
2. **Runner 실행 전용(Execution-Only Runner).** Runner Server는 승인된 실행 작업만 수행하고, 결과(로그·산출물·성공/실패)만 Backend에 반환한다. 비즈니스 판단·외부 시스템 변경·완료 판정 **절대 금지**.
3. **Human Check 게이팅.** `Human Check Required` 미승인 작업은 Runner Job 생성·진행 전환으로 **절대** 넘어가지 **금지**.
4. **Backend = 상태 원장.** 전역 상태(Project/Sprint/Task/RunnerJob/WorkEvent…)의 최종 진실은 **반드시** Backend DB다. Runner 내부 큐/슬롯 상태는 로컬 부분 상태일 뿐이다(§5.2).
5. **감시가 아니라 작업 흐름 추적.** 근태·응답속도·개인 생산성 점수화는 **절대 금지**. 추적 대상은 작업 증거(상태 변화·이벤트·산출물)로 **제한**한다.

---

## 3. 컴포넌트별 책임

각 서비스 책임을 **반드시** / **허용** / **금지** / **절대 금지** 로 판정한다. 서비스 **내부** 레이어 배치는 §3.6 위임.

### 3.1 Frontend

- **반드시** 둘 것: 작업 목록 사이드바, 작업/스프린트 상세 표시, 실행 설계·완료 조건 표시/편집 UI, Human Check 승인/보류/거절 액션, Runner 상태·로그 표시, AI 패널. 서버 데이터는 TanStack Query로 조회, UI 선택 상태는 Zustand.
- **허용**: Backend REST/SSE 호출. FastAPI OpenAPI 스키마에서 생성한 타입 사용(`openapi-typescript`).
- **금지**: 영속 비즈니스 상태를 Frontend에 소유. 비즈니스 규칙(승인 게이팅·분류 판정) 자체 결정.
- **절대 금지**: Monday / Slack / AI API / Runner Server **직접 호출**. AI API Key·Token 보유.

### 3.2 Backend

- **반드시** 둘 것: 전체 도메인 상태의 CRUD와 상태 원장, AI 분석(AI Planner) / 실행 유형·주체 분류(Execution Classifier) / Human Check 관리, Runner Job 생성·상태 관리·결과 수신(Runner Orchestrator), Monday/Slack 연동(Adapter), WorkEvent 수집·완료 요약·회고(AI Tracker), AuditLog. 무거운 작업은 **반드시** Queue로 위임(§5.1).
- **허용**: PostgreSQL 읽기/쓰기, Redis 큐 발행/소비, Monday/Slack/AI API 호출(httpx), Runner Server REST 호출 또는 Pull 응답.
- **금지**: 외부 API를 API 요청 스레드 안에서 동기 처리(§5.1 큐 대상 작업). 사용자 승인 전 외부 시스템 반영.
- **절대 금지**: 로컬 코드/파일 실행(=Runner 책임). Human Check Required 미승인 작업의 Runner Job 생성.

### 3.3 Runner Server

- **반드시** 둘 것: Backend에서 받은 Runner Job 검증, 빈 Slot 배정, Claude Code PTY 세션 실행, 실행 로그·산출물 수집, 성공/실패 결과 Backend 반환, 로컬 Slot/Runner 상태 관리 및 heartbeat.
- **허용**: 등록된 프로젝트 경로(`cwd`) 안에서만 로컬 파일/코드/문서 작업. runner_secret 또는 단기 토큰으로 Backend 인증. MVP-1 내부 로컬 큐 보유(단 최종 원장은 Backend).
- **금지**: 미검증 `cwd`·미등록 경로 실행. 허용되지 않은 stage/명령 실행.
- **절대 금지**: Monday/Slack API 호출, 외부 시스템 상태 직접 변경, 작업 최종 완료 판정, Human Check 승인, 비즈니스 의사결정, 운영 DB 변경, 시크릿/외부 Token 보유.

### 3.4 Database (PostgreSQL)

- **반드시**: 관계형 도메인 상태 저장(§ PRD 16 데이터 모델), AI 응답 원문 JSONB 보존, WorkEvent/AuditLog append, Monday/Slack/Runner ID ↔ 내부 엔티티 매핑, 트랜잭션 보장.
- **허용**: 추후 pgvector 확장.
- **절대 금지**: Frontend/Runner의 직접 접속(반드시 Backend 경유).

### 3.5 Queue (Redis + Dramatiq) / Storage

- Queue **반드시**: 비동기 작업 파이프라인(§5.1). idempotency로 중복 처리 방지.
- Storage **반드시**: Runner 산출물·로그·생성 문서 보관. MVP는 DB 텍스트 + 파일 경로. 확장 시 S3-compatible.
- **절대 금지**: Queue/Storage에 최종 도메인 진실 저장(원장은 DB).

### 3.6 App 내부 레이어 위임

각 서비스 내부의 레이어 모델(Domain / Application / Infrastructure / Presentation), 폴더→레이어 매핑, 절대 금지 매트릭스는 **본 문서가 아니라** 각 App ARCHITECTURE가 SSOT다. 신규 App 착수 시 `docs/{App}/{App}-ARCHITECTURE.md`를 작성한다(claudecode-for-me `ARCHITECTURE-TEMPLATE.md` 기반).

| App | 내부 레이어 문서(예정) | 예상 스택 적응 |
|---|---|---|
| Frontend | `docs/FRONTEND/FRONTEND-ARCHITECTURE.md` | Next.js App Router + feature 폴더, store=Application 상태 |
| Backend | `docs/BACKEND/BACKEND-ARCHITECTURE.md` | FastAPI 라우트=Presentation, services=Application, repositories/adapters=Infrastructure |
| Runner Server | `docs/RUNNER/RUNNER-ARCHITECTURE.md` | Rust: api=Presentation, slot/pipeline=Application, pty/artifacts=Infrastructure |

> **충돌 시 우선순위**: 본 시스템 ARCHITECTURE(서비스 경계·참조 매트릭스·보안 경계)가 App ARCHITECTURE보다 **우선**. App은 시스템 룰 준수 + 내부 레이어 특이사항만 명시.

---

## 4. 컴포넌트 간 참조 / 호출 방향

### 4.1 허용 통신 경로

| From | To | 프로토콜 | 용도 |
|---|---|---|---|
| Frontend | Backend | HTTPS REST + SSE | 조회/명령/승인 트리거, 실시간 상태 스트림 |
| Backend | Database | SQLAlchemy | 상태 원장 read/write |
| Backend | Queue | Redis(Dramatiq) | 비동기 작업 발행/소비 |
| Backend | Runner Server | REST(Push) 또는 Pull | Runner Job 전달·상태 조회 |
| Backend | Monday API | HTTPS(httpx) | Board/Group/Item/Subitem/Comment, webhook 수신 |
| Backend | Slack API | HTTPS(httpx) | 메시지 전송, 승인/액션 수신 |
| Backend | AI API | HTTPS | 분석·분류·요약·회고 |
| Runner Server | Backend | REST | 상태/로그/결과 보고, (Pull 모드) Job 조회 |
| Runner Server | Storage(로컬) | 파일 IO | 산출물·로그 수집 |

Runner 통신은 MVP-1 `Frontend → Backend → Runner Server`(Push). Backend가 외부, Runner가 사내/개인 PC일 때는 Runner가 **Pull**로 Job을 조회한다(PRD §19.2).

### 4.2 절대 금지 매트릭스 (시스템 레벨)

명시된 **금지** / **절대 금지** 조합만 위반으로 판정한다. 나머지는 §4.1 허용 경로를 따른다.

| From \ To | Frontend | Backend | Runner Server | Database | Monday/Slack API | AI API |
|---|---|---|---|---|---|---|
| **Frontend** | — | **허용** | **절대 금지** | **절대 금지** | **절대 금지** | **절대 금지** |
| **Backend** | (SSE 응답) | — | **허용** | **허용** | **허용** | **허용** |
| **Runner Server** | **절대 금지** | **허용** | — | **절대 금지** | **절대 금지** | **금지**\* |
| **Database** | **절대 금지** | — | **절대 금지** | — | — | — |

\* Runner의 Claude Code CLI 세션 자체는 Anthropic과 통신하지만, 이는 로컬 실행 도구의 내부 동작이며 **시스템의 AI 분석 API 호출(Backend 전유)과는 별개**다. Runner는 시스템 AI API Key를 보유·호출하지 않는다.

**핵심 위반 판정**:
- Frontend → Monday/Slack/Runner/DB 직접 접근 = **절대 금지**.
- Runner → Monday/Slack/DB/Frontend 접근 = **절대 금지**.
- 외부 시스템 쓰기(Monday 댓글·Slack 알림·상태 업데이트)는 오직 Backend.

---

## 5. 데이터 흐름 & 비동기 경계

### 5.1 Queue 대상 작업 (동기 API 금지)

아래 작업은 **반드시** Queue로 위임한다. API 요청 스레드에서 동기 처리 **금지**.

| 큐 | 처리 작업 |
|---|---|
| `ai_analysis_queue` | 작업 분석, 스프린트 분리, 실행 설계·완료 조건 생성 |
| `runner_job_queue` | Runner Job dispatch, 결과 처리 |
| `monday_integration_queue` | Board/Group/Item 생성, 댓글 작성 |
| `slack_integration_queue` | 메시지 전송, 액션 처리 |
| `summary_generation_queue` | 완료 요약 생성 |
| `retrospective_queue` | 스프린트 회고 생성 |

### 5.2 상태 원장 vs 로컬 상태

```text
Backend DB   = 전역 RunnerJob Queue·전체 도메인 상태의 최종 원장(SSOT)
Runner Server = 로컬 Slot 상태 + 현재 실행 중 Job 관리(부분·휘발 상태)
```

MVP-1은 Runner 내부 큐를 **허용**하되, 불일치 시 **반드시** Backend DB를 진실로 본다.

### 5.3 실시간(Realtime)

MVP는 **SSE**(Backend → Frontend) 우선: RunnerJob 상태 변경, Runner 로그 스트리밍, Slot 상태, AI 분석 진행. 확장 시 WebSocket 검토.

---

## 6. 실행 주체 분리 → 컴포넌트 라우팅

작업의 `execution_type`(사람/AI 판단 수준)과 `executor_type`(실행 담당)이 곧 **어느 컴포넌트가 실행하는가**를 결정한다.

| execution_type | executor_type | 실행 컴포넌트 | 승인 필요 |
|---|---|---|---|
| Human Check Required | Human | 사람 (Frontend에서 승인/판단) | O |
| AI Draft + Human Approval | Backend AI | Backend AI Planner/Tracker | O (반영 전) |
| AI Draft + Human Approval | Runner Server | Runner Server (승인 후 Job) | O (Job 생성 전) |
| AI Auto Executable | Backend AI | Backend AI | X |
| AI Auto Executable | Backend Integration | Backend Monday/Slack Adapter | X |

**게이트 규칙**: `executor_type = Runner Server` 작업은 (1) 연결된 Human Check Required 항목이 모두 승인, (2) 승인된 stage·프로젝트 경로일 때만 Runner Job으로 생성된다(PRD §9.3).

---

## 7. 보안 경계

| 경계 | 인증/검증 | 원칙 |
|---|---|---|
| Frontend → Backend | Bearer Token / 세션 (확장: OAuth/SSO) | — |
| Backend → Runner Server | `runner_secret` 또는 단기 토큰 | Runner는 승인된 Job만 실행 |
| Slack → Backend | Slack signing secret 검증 | webhook 위조 방지 |
| Monday → Backend | webhook verification token / signature | webhook 위조 방지 |

**토큰 소유 불변식** (PRD §12.5):
- Monday API Token · Slack Bot Token · AI API Key 는 **반드시** Backend만 보유(환경변수/Secret Manager).
- Runner Server는 외부 서비스 Token을 **절대** 보유 **금지**.
- Runner Server는 **반드시** 등록된 프로젝트 경로에서만 실행.
- Human Check Required 미승인 작업은 Runner Server로 **절대** 전달 **금지**.
- Human Check 승인자는 **반드시** AuditLog에 기록.

---

## 8. 상태·이벤트 & 신뢰성 경계

### 8.1 이벤트 원장

- 모든 외부/내부 이벤트(Monday 상태 변경·댓글, Slack 보고, RunnerJob 전이, Human Check 결정)는 **반드시** `WorkEvent`로 Backend에 수집한다.
- 상태 변경·승인·통합 호출은 **반드시** `AuditLog`에 남긴다(PRD §24 추적성).

### 8.2 멱등성 & 재시도

- Monday/Slack webhook 중복 수신 대비 **반드시** idempotency key 사용.
- Monday 등록·Slack 전송 실패는 **반드시** 재시도 가능해야 하며, AI 파싱 실패 시 **반드시** 원문 보존 후 재시도.

### 8.3 Runner 장애 경계

- Runner offline / 슬롯 없음 → Job을 `queued` 유지 또는 실패 처리(Backend 판정).
- heartbeat 유실 → `timeout`/`unknown` 전이.
- 실패 시 **자동 재시도하지 않고** 사람 확인 대상으로 전환 가능. 실패 통지는 **반드시** Backend가 Monday/Slack에 전달하고 WorkEvent/AuditLog에 저장.

---

## 9. 단계별 도입 (아키텍처 관점)

| 단계 | 활성 컴포넌트 | 아키텍처 초점 |
|---|---|---|
| MVP-0 | Frontend + Backend + DB | 작업 워크스페이스·AI 분석 검증(동기 우선) |
| MVP-1 | + Queue + Runner Server + Monday/Slack Adapter | 비동기 경계, Runner 오케스트레이션, 통합 소유 확립 |
| MVP-2 | + SSE + AI Tracker | 실시간 상태, WorkEvent 히스토리, 완료 요약·회고 |
| MVP-3 | 다중 Runner + Pull + S3 | Runner 수평 확장, 원격 Pull 모델, Storage 외부화 |
