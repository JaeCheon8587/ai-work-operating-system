# BACKEND-ARCHITECTURE — 상태 원장 + AI 운영 코어 호스트 아키텍처

> App별 ARCHITECTURE. 솔루션 공통 룰은 [솔루션 ARCHITECTURE](../ARCHITECTURE.md) SSOT 우선. 본 문서는 호스트 특이 사항만 보유.

| 항목 | 값 |
|---|---|
| 문서 ID | BACKEND-ARCHITECTURE |
| 버전 | 0.1 (Draft) |
| App 코드 | BACKEND |
| 작성 가정 | 솔루션 공통 룰([../ARCHITECTURE.md](../ARCHITECTURE.md)) 준수. Phase 1 = sync-first + DB outbox poller, Redis 없음. |
| 관련 문서 | [솔루션 ARCHITECTURE](../ARCHITECTURE.md) · [BACKEND-PRD](BACKEND-PRD.md) · [BACKEND-FC](BACKEND-FC.md) · [BACKEND-ADR-CATALOG](BACKEND-ADR-CATALOG.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — FastAPI 호스트 아키텍처 + ADR-001(claude -p 경계) 부트스트랩 | jaecheon.jeong |

## 1. App 개요

| 항목 | 값 |
|---|---|
| App 코드 | BACKEND |
| 한 줄 설명 | 도메인 상태 SSOT + AI 코파일럿 + Slack/git 반영 단독 소유 REST 서비스 |
| 런타임 | Python 3.12+ · FastAPI · Pydantic · SQLAlchemy 2.0 · Alembic |
| 진입점 경로 | `backend/app/main.py` |
| 호스트 종류 | REST API 서비스 + 인-프로세스 백그라운드 poller |

내부 레이어 (tech_stack §4.3 기준): `api/routes/`(Presentation) · `services/`(Application) · `repositories/`·`db/`·adapters(Infrastructure) · `schemas/`(Pydantic 계약).

## 2. 핵심 책임 (4단계 마커)

- **반드시** PRD §12 전 도메인 상태 CRUD 와 상태 원장(SSOT) 유지.
- **반드시** AI 코파일럿(AI Planner), confirm 트랜잭션, outbox poller, slack_notify, repo_reflect, 감사 로그 소유.
- **허용** PostgreSQL read/write, Anthropic(`claude -p`) 실행, Slack Bot API 호출, docs repo git commit+push.
- **금지** 외부 API 를 요청 스레드에서 동기 처리(Slack/git 은 outbox poller 로 위임). 사용자 승인 전 외부 반영.
- **절대 금지** Monday API 호출(Phase 2), 외부 Token 을 Backend 밖 노출, DB 를 Frontend/Runner 가 직접 접속하게 허용.

## 3. 외부 IO Adapter 위치 정책
- DB 접근은 `repositories/` + `db/session.py`(Infrastructure)에 집중. services 는 repository 경유.
- AI 실행(`ai_copilot/`)·Slack(`slack_notify/`)·git(`repo_reflect/`) 은 각각 서비스 인터페이스 뒤로 격리 → Phase 2 Dramatiq worker 가 동일 인터페이스로 감쌀 수 있게(재작성 방지).
- outbox poller(`workers/outbox_poller.py`)는 인-프로세스 백그라운드(APScheduler/스레드). Phase 2 에서 큐 소비자로 승격.

## 4. App 특이 도메인/패턴 룰

- **confirm 원자성**: 6 planning 엔티티 + assignment 토큰 + PlanningMarkdownDocument + outbox 행을 단일 트랜잭션. side-effect(Slack/git)는 트랜잭션 밖, poller 가 멱등키로 처리.
- **멱등키**: Slack = `assignment_id + notification_version`, repo = `document_id + content_hash`. side-effect 실행 전 결정(사후 로깅 아님).
- **AI 실행 계약**: `claude -p --output-format json` stdout 을 Pydantic 스키마 검증. timeout·subprocess cleanup·인-프로세스 semaphore cap. 파싱실패/빈/refusal 각각 실패 처리 + 원문 보존.
- **claude -p 경계 결정**: Backend 가 Claude Code CLI 를 구동 — 솔루션 ARCHITECTURE §3.3("Runner 가 Claude Code CLI 소유") 경계에 근접. Phase 1 은 텍스트 변환(코드 실행 아님)에 한정하며, 본 결정을 [BACKEND-ADR-001](ADR/BACKEND-ADR-001.md) 로 명시 기록. Phase 2 Runner 도입 시 코드 실행형 CLI 사용은 Runner 로 이관.

## 5. 솔루션 SSOT 인용

본 App 작업 시 솔루션 공통 룰 **반드시** 준수:
- [§2 시스템 아키텍처](../ARCHITECTURE.md#2-시스템-아키텍처)
- [§3.2 Backend 책임](../ARCHITECTURE.md#32-backend)
- [§4.2 절대 금지 매트릭스](../ARCHITECTURE.md#42-절대-금지-매트릭스-시스템-레벨)
- [§5 데이터 흐름 & 비동기 경계](../ARCHITECTURE.md#5-데이터-흐름--비동기-경계)
- [§7 보안 경계](../ARCHITECTURE.md#7-보안-경계)

솔루션 룰과 본 App 룰 충돌 시 **솔루션 ARCHITECTURE 우선**.
