# 기술 스택 정리: AI Work Operating System

가칭: **SprintOps AI**

---

## 1. 전체 시스템 구성

AI Work Operating System은 다음 3개 앱/서비스와 공통 인프라로 구성한다.

```text
AI Work Operating System
├─ Frontend
├─ Backend
├─ Runner Server
├─ Database
├─ Queue
└─ Storage
```

각 역할은 다음과 같다.

```text
Frontend
= 사용자가 작업을 만들고, 스프린트를 보고, 승인/수정하는 작업 운영 콘솔

Backend
= 상태 원장, AI 분석, Human Check, Monday/Slack 연동, Runner Job 관리

Runner Server
= 승인된 실행 작업을 로컬 Claude Code 슬롯에서 수행하는 실행 엔진

Database
= 작업, 스프린트, 승인, Runner Job, 이벤트, 회고 데이터 저장소

Queue
= AI 분석, Runner Job, Monday/Slack 알림, 완료 요약 등 비동기 작업 처리

Storage
= Runner 산출물, 로그, 생성 문서, 첨부 파일 저장소
```

---

## 2. 최종 추천 기술 스택

| 영역 | 추천 스택 |
|---|---|
| Frontend | Next.js, React, TypeScript |
| UI | Tailwind CSS, shadcn/ui |
| Client State | Zustand |
| Server State | TanStack Query |
| Backend | Python, FastAPI |
| Backend Schema | Pydantic |
| ORM | SQLAlchemy 2.0 |
| Migration | Alembic |
| DB | PostgreSQL |
| Queue | Redis + Dramatiq 또는 Celery |
| Realtime | SSE 우선, 추후 WebSocket |
| Runner Server | Rust, Axum, portable-pty |
| Local AI Execution | Claude Code CLI |
| AI API | OpenAI API 또는 Anthropic API |
| Storage | Local File Storage, 추후 S3/MinIO/R2 |
| Infra | Docker Compose |
| Logging | structlog/loguru, Runner는 tracing |
| Testing | pytest, Playwright |
| API 문서 | FastAPI OpenAPI |
| Frontend API Types | openapi-typescript |

---

## 3. Frontend 기술 스택

### 3.1 추천 스택

```text
Next.js
React
TypeScript
Tailwind CSS
shadcn/ui
TanStack Query
Zustand
```

### 3.2 역할

Frontend는 사용자가 실제로 보는 작업 운영 콘솔이다.

주요 기능:

```text
- 좌측 작업 목록
- 새 작업 만들기
- 작업 검색/필터
- 작업 상세 화면
- 스프린트 상세 화면
- 실행 설계 확인/수정
- Human Check 승인/보류/거절
- AI 패널
- Runner 실행 상태 확인
- Runner 실행 로그 확인
- Runner 산출물 확인
- Monday/Slack 상태 확인
- 완료 요약/회고 확인
```

### 3.3 추천 라이브러리

```text
Next.js
- App Router 기반 라우팅

React
- 컴포넌트 기반 UI

TypeScript
- 타입 안정성 확보

Tailwind CSS
- 빠른 스타일링

shadcn/ui
- 기본 UI 컴포넌트

TanStack Query
- Backend API 호출, 캐싱, 서버 상태 관리

Zustand
- 좌측 사이드바 상태, 선택된 작업/스프린트, UI 상태 관리

openapi-typescript
- FastAPI OpenAPI 스키마에서 Frontend 타입 자동 생성
```

### 3.4 추천 디렉터리 구조

```text
frontend/
├─ app/
│  ├─ workspace/
│  ├─ projects/[projectId]/
│  ├─ settings/
│  └─ layout.tsx
├─ components/
│  ├─ ui/
│  ├─ layout/
│  ├─ workspace/
│  ├─ sprint/
│  ├─ human-check/
│  ├─ runner/
│  └─ ai-panel/
├─ features/
│  ├─ workspace/
│  ├─ project/
│  ├─ sprint/
│  ├─ task/
│  ├─ human-check/
│  ├─ runner/
│  └─ ai-panel/
├─ lib/
│  ├─ api/
│  ├─ types/
│  └─ utils/
└─ store/
```

---

## 4. Backend 기술 스택

### 4.1 추천 스택

```text
Python 3.12+
FastAPI
Pydantic
SQLAlchemy 2.0
Alembic
PostgreSQL
Redis
Dramatiq 또는 Celery
httpx
structlog 또는 loguru
pytest
```

### 4.2 Backend를 Python으로 선택하는 이유

이 시스템의 Backend는 단순 CRUD 서버가 아니라 AI 중심의 업무 운영 코어다.

Backend가 담당하는 주요 기능:

```text
- AI 작업 분석
- 스프린트 생성
- 실행 설계 생성
- Human Check 분류
- executor_type 분류
- Runner Job 생성
- Runner 결과 요약
- 완료 요약 생성
- 스프린트 회고 생성
- WorkEvent 분석
- 블로커 감지
```

Python은 AI 연동, 텍스트 처리, 분류, 요약, 데이터 후처리에 적합하다.

### 4.3 Backend 주요 모듈

```text
backend/
├─ app/
│  ├─ main.py
│  ├─ api/
│  │  └─ routes/
│  │     ├─ projects.py
│  │     ├─ sprints.py
│  │     ├─ tasks.py
│  │     ├─ human_checks.py
│  │     ├─ runner_jobs.py
│  │     ├─ runners.py
│  │     ├─ monday.py
│  │     ├─ slack.py
│  │     └─ webhooks.py
│  ├─ core/
│  │  ├─ config.py
│  │  ├─ security.py
│  │  └─ logging.py
│  ├─ db/
│  │  ├─ session.py
│  │  ├─ models/
│  │  └─ migrations/
│  ├─ schemas/
│  ├─ services/
│  │  ├─ ai_planner/
│  │  ├─ execution_classifier/
│  │  ├─ human_check/
│  │  ├─ runner_orchestrator/
│  │  ├─ monday/
│  │  ├─ slack/
│  │  ├─ work_event/
│  │  └─ retrospective/
│  ├─ workers/
│  └─ repositories/
```

### 4.4 Backend 주요 기능

```text
- Project / Sprint / Task / Subtask CRUD
- 작업 워크스페이스 상태 관리
- AI 분석 요청
- 스프린트 생성
- 실행 설계 생성
- Human Check Required 분류
- AI Draft + Human Approval 분류
- AI Auto Executable 분류
- executor_type 분류
- Human Check 승인/보류/거절 처리
- RunnerJob 생성
- Runner 상태 관리
- Runner 실행 결과 수신
- Monday Board/Item/Comment 연동
- Slack 알림/승인 연동
- WorkEvent 저장
- 완료 요약 생성
- 스프린트 회고 생성
- AuditLog 저장
```

---

## 5. Database 기술 스택

### 5.1 추천 DB

```text
PostgreSQL
```

### 5.2 PostgreSQL을 선택하는 이유

이 시스템은 상태와 관계가 많은 시스템이다.

저장해야 할 주요 데이터:

```text
- Project
- Sprint
- Task
- Subtask
- SprintDesign
- HumanCheckItem
- RunnerJob
- Runner
- RunnerSlot
- WorkEvent
- WorkLogSummary
- SprintRetrospective
- IntegrationEvent
- AuditLog
```

PostgreSQL이 적합한 이유:

```text
- 관계형 데이터 모델에 적합
- 트랜잭션 처리 가능
- JSONB로 AI 응답 원문 저장 가능
- AuditLog / WorkEvent 저장에 적합
- 향후 pgvector 확장 가능
- Monday/Slack/Runner ID 매핑에 적합
```

### 5.3 주요 테이블

```text
projects
user_workspace_states
raw_task_inputs
sprints
tasks
subtasks
sprint_designs
human_check_items
runner_jobs
runners
runner_slots
assignees
work_events
work_log_summaries
blockers
sprint_retrospectives
integration_events
audit_logs
```

### 5.4 ORM / Migration

```text
ORM: SQLAlchemy 2.0
Migration: Alembic
```

---

## 6. Queue 기술 스택

### 6.1 추천 Queue

MVP에서는 다음 중 하나를 선택한다.

```text
Dramatiq + Redis
또는
Celery + Redis
```

### 6.2 추천

초기 MVP에서는 단순성을 고려해 **Dramatiq + Redis**를 추천한다.

운영 안정성과 고급 재시도/스케줄링이 더 중요해지면 **Celery + Redis**도 가능하다.

### 6.3 Queue가 필요한 이유

아래 작업들은 API 요청 안에서 바로 처리하면 안 된다.

```text
- AI 작업 분석
- 스프린트 생성
- 실행 설계 생성
- 완료 조건 생성
- Runner Job dispatch
- Runner 결과 처리
- Monday 댓글 작성
- Slack 알림 전송
- 완료 요약 생성
- 스프린트 회고 생성
```

### 6.4 Queue 사용처

```text
ai_analysis_queue
runner_job_queue
monday_integration_queue
slack_integration_queue
summary_generation_queue
retrospective_queue
```

---

## 7. Runner Server 기술 스택

### 7.1 추천 스택

```text
Rust
Axum
portable-pty
tokio
serde
tracing
Claude Code CLI
```

### 7.2 Runner Server 역할

Runner Server는 승인된 실행 작업만 수행하는 로컬 실행 엔진이다.

주요 기능:

```text
- Backend에서 RunnerJob 수신
- RunnerJob 검증
- 빈 Slot에 Job 배정
- Claude Code PTY 세션 실행
- 실행 로그 수집
- 산출물 수집
- 성공/실패 결과 Backend에 반환
- Runner 상태/Slot 상태 관리
```

### 7.3 Rust를 선택하는 이유

Runner Server는 로컬 프로세스와 PTY를 안정적으로 관리해야 한다.

Rust가 적합한 이유:

```text
- 프로세스 제어에 강함
- PTY 관리에 적합
- 장시간 실행 안정성
- Windows / macOS / Linux 대응 가능
- 기존 Slot Runner의 Rust 기반 구조를 참고 가능
```

### 7.4 기존 Slot Runner 참고 방향

기존 Slot Runner의 다음 요소를 참고한다.

```text
- 슬롯 풀
- Claude Code PTY 세션
- Job Queue
- 파일 게이트 기반 단계 판정
- 실행 로그 수집
- 완료/실패 상태 반환
```

단, 다음은 신규 Runner Server에서 제외하거나 Backend로 이동한다.

```text
monday-notify
→ Backend Monday Adapter로 이동

Slack/Monday Token 보유
→ Runner Server에서는 금지

외부 협업 도구 업데이트
→ Backend에서만 수행
```

### 7.5 Runner Server 추천 구조

```text
runner-server/
├─ src/
│  ├─ main.rs
│  ├─ api/
│  │  ├─ jobs.rs
│  │  ├─ runners.rs
│  │  └─ health.rs
│  ├─ slot/
│  │  ├─ pool.rs
│  │  ├─ slot.rs
│  │  └─ state.rs
│  ├─ pty/
│  │  ├─ claude.rs
│  │  └─ session.rs
│  ├─ artifacts/
│  │  └─ collector.rs
│  ├─ security/
│  │  └─ validator.rs
│  └─ config.rs
└─ Cargo.toml
```

### 7.6 Runner Server MVP 범위

```text
- 단일 Runner Server
- 로컬 실행 우선
- 슬롯 수 설정 가능
- Claude Code PTY 세션 실행
- RunnerJob 수신
- 실행 로그 수집
- 완료/실패 결과 반환
- Backend에 결과 보고
- Monday/Slack 직접 연동 없음
```

---

## 8. Storage 기술 스택

### 8.1 MVP

```text
Local File Storage
```

### 8.2 확장

```text
MinIO
AWS S3
Cloudflare R2
```

### 8.3 저장 대상

```text
- Runner 실행 로그
- Runner 산출물
- 생성된 Markdown 문서
- 변경 파일 목록
- AI 응답 원문
- 완료 요약
- 스프린트 회고
```

MVP에서는 DB에 텍스트 저장 + 파일 경로 저장으로 시작한다.

---

## 9. Realtime 기술 스택

### 9.1 추천

```text
MVP: Server-Sent Events
확장: WebSocket
```

### 9.2 사용처

```text
- RunnerJob 상태 변경
- Runner 로그 스트리밍
- Slot 상태 변경
- AI 분석 진행 상태
- Monday/Slack 연동 결과
```

MVP에서는 SSE가 단순하고 충분하다.

---

## 10. AI / LLM 기술 스택

### 10.1 선택 가능 모델

```text
OpenAI API
Anthropic API
```

### 10.2 사용 영역

```text
- 작업 분석
- 스프린트 분리
- 실행 설계 생성
- 완료 조건 생성
- Human Check Required 분류
- executor_type 분류
- Runner Prompt 생성
- 작업 히스토리 요약
- 완료 요약 생성
- 스프린트 회고 생성
```

### 10.3 권장 방식

```text
- JSON Schema 기반 구조화 출력 사용
- AI 응답 원문 저장
- 파싱 실패 시 재시도
- 사람이 승인하기 전 외부 시스템 반영 금지
```

---

## 11. 외부 연동 스택

### 11.1 Monday.com

담당 앱:

```text
Backend
```

사용 기능:

```text
- Board 조회
- Group 생성
- Item 생성
- Subitem 생성
- Comment / Update 작성
- Status 변경 이벤트 수신
- Webhook 수신
```

주의:

```text
Runner Server는 Monday API를 호출하지 않는다.
```

### 11.2 Slack

담당 앱:

```text
Backend
```

사용 기능:

```text
- Slack 메시지 전송
- 승인 버튼 처리
- Human Check 승인 요청
- 수정 요청 수신
- 완료 보고 수신
- 블로커 보고 수신
```

주의:

```text
Runner Server는 Slack API를 호출하지 않는다.
```

---

## 12. 인증 / 보안 스택

### 12.1 Frontend → Backend

```text
MVP:
- Bearer Token 또는 세션 기반 인증

확장:
- OAuth / SSO
```

### 12.2 Backend → Runner Server

```text
runner_secret
또는
short-lived token
```

### 12.3 Slack → Backend

```text
Slack signing secret 검증
```

### 12.4 Monday → Backend

```text
Webhook verification token 또는 signature 검증
```

### 12.5 중요 원칙

```text
- Monday API Token은 Backend만 보유한다.
- Slack Bot Token은 Backend만 보유한다.
- AI API Key는 Backend만 보유한다.
- Runner Server는 외부 서비스 Token을 보유하지 않는다.
- Runner Server는 등록된 프로젝트 경로에서만 실행한다.
- Human Check Required 미승인 작업은 Runner Server로 전달하지 않는다.
```

---

## 13. Infra / DevOps

### 13.1 MVP 인프라

```text
Docker Compose
PostgreSQL
Redis
Backend FastAPI
Frontend Next.js
Runner Server local process
```

### 13.2 추천 docker-compose 구성

```text
services:
  frontend
  backend
  postgres
  redis
```

Runner Server는 초기에는 로컬 프로세스로 실행한다.

확장 시 Runner Server도 별도 서비스 또는 데스크톱 에이전트로 분리한다.

### 13.3 환경 변수 예시

```text
DATABASE_URL
REDIS_URL
OPENAI_API_KEY
ANTHROPIC_API_KEY
MONDAY_API_TOKEN
SLACK_BOT_TOKEN
SLACK_SIGNING_SECRET
RUNNER_SECRET
RUNNER_ENDPOINT
```

---

## 14. 테스트 기술 스택

### 14.1 Frontend

```text
Vitest
React Testing Library
Playwright
```

### 14.2 Backend

```text
pytest
pytest-asyncio
httpx test client
factory-boy 또는 polyfactory
```

### 14.3 Runner Server

```text
cargo test
integration test
mock Claude Code command
```

### 14.4 E2E 테스트

```text
Playwright
Docker Compose 기반 테스트 환경
```

---

## 15. Logging / Observability

### 15.1 Backend

```text
structlog 또는 loguru
```

저장/추적 대상:

```text
- API 요청
- AI 분석 요청
- AI 응답
- RunnerJob 상태
- Monday/Slack API 호출
- Webhook 수신
- WorkEvent 생성
```

### 15.2 Runner Server

```text
tracing
tracing-subscriber
```

저장/추적 대상:

```text
- Runner 상태
- Slot 상태
- Claude Code 실행 로그
- Job 시작/완료/실패
- 산출물 수집 결과
```

---

## 16. 단계별 기술 도입 계획

### MVP-0

목표: 작업 워크스페이스와 AI 분석 검증

```text
Frontend:
- Next.js
- 작업 목록
- 작업 상세
- 스프린트 상세

Backend:
- FastAPI
- PostgreSQL
- SQLAlchemy
- AI 분석 API

Infra:
- Docker Compose
- PostgreSQL
```

### MVP-1

목표: Runner Server와 외부 연동 추가

```text
Backend:
- Redis
- Dramatiq 또는 Celery
- Monday Adapter
- Slack Adapter
- Runner Orchestrator

Runner Server:
- Rust
- Axum
- portable-pty
- Claude Code CLI 실행

Frontend:
- Runner 상태 표시
- Human Check 승인
- Monday/Slack 상태 표시
```

### MVP-2

목표: 운영 자동화와 회고 강화

```text
- SSE 기반 실시간 상태
- Runner 로그 스트리밍
- WorkEvent 히스토리
- 완료 요약 자동 생성
- 스프린트 회고 자동 생성
```

### MVP-3

목표: 운영 인사이트와 확장

```text
- 다중 Runner 지원
- Runner Pull 방식
- S3-compatible storage
- 작업 간 인사이트 분석
- Human Check 패턴 분석
- Runner 실패 패턴 분석
```

---

## 17. 최종 추천안

최종 추천 스택은 다음과 같다.

```text
Frontend:
- Next.js
- React
- TypeScript
- Tailwind CSS
- shadcn/ui
- TanStack Query
- Zustand

Backend:
- Python 3.12+
- FastAPI
- Pydantic
- SQLAlchemy 2.0
- Alembic
- PostgreSQL
- Redis
- Dramatiq 또는 Celery
- httpx
- structlog 또는 loguru

Runner Server:
- Rust
- Axum
- portable-pty
- tokio
- serde
- tracing
- Claude Code CLI

Infra:
- Docker Compose
- PostgreSQL
- Redis
- Local File Storage

Realtime:
- SSE 우선
- WebSocket 추후 검토

External Integrations:
- Monday.com API
- Slack API

AI:
- OpenAI API 또는 Anthropic API
```

---

## 18. 결론

Backend를 Python으로 구성하는 것은 문제없다.

오히려 이 시스템은 AI 분석, 분류, 요약, 회고 생성 등 AI 중심 로직이 많기 때문에 Python Backend가 잘 맞는다.

단, 다음 원칙은 유지해야 한다.

```text
1. Backend는 FastAPI로 구성한다.
2. DB는 PostgreSQL을 사용한다.
3. Queue는 Redis + Dramatiq 또는 Celery를 사용한다.
4. Frontend 타입은 FastAPI OpenAPI 스키마에서 생성한다.
5. Runner Server는 Rust 기반 Headless Runner로 구성한다.
6. Monday/Slack 연동은 Backend가 단일 소유한다.
7. Runner Server는 승인된 실행 작업만 처리한다.
8. Runner Server는 Monday/Slack/API Key를 보유하지 않는다.
```
