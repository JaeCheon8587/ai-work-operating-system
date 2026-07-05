# BACKEND-PRD — 상태 원장 + AI 운영 코어

> **본 App 의 product 요구사항** (시점 = 솔루션 내 BACKEND App 포커스). 솔루션 전체 시야 = [솔루션 PRD](../prd.md) · [/CLAUDE.md](../../CLAUDE.md). 기술 시야 = [BACKEND-ARCHITECTURE](BACKEND-ARCHITECTURE.md). 기능 정의 SSOT = [BACKEND-FC](BACKEND-FC.md).

| 항목 | 값 |
|---|---|
| 문서 ID | BACKEND-PRD |
| 버전 | 0.1 (Draft) |
| 작성 가정 | Phase 1 MVP. BACKEND 는 상태 원장 + AI 코파일럿 + confirm/outbox + Slack/git 반영. Runner/Monday/Queue(Redis) 제외 — sync-first + DB outbox poller. |
| 관련 문서 | [솔루션 PRD](../prd.md) · [BACKEND-FC](BACKEND-FC.md) · [BACKEND-ARCHITECTURE](BACKEND-ARCHITECTURE.md) · [BACKEND-ADR-CATALOG](BACKEND-ADR-CATALOG.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — Phase 1 상태 원장 + AI 운영 코어 부트스트랩 | jaecheon.jeong |

---

## 1. 배경
- 솔루션의 상태 원장(SSOT)이자 업무 운영 코어. Frontend 조회/명령을 받아 도메인 상태를 관리하고, AI 코파일럿·Slack 발송·문서 반영을 단독 소유한다.
- Phase 1 은 Runner/Monday/Queue 없이 FastAPI + Postgres 단일 프로세스 + 백그라운드 outbox poller 로 동작.

## 2. 문제 정의
- PM 자유 입력을 AI가 구조화 PM 설계 문서로 변환·비판하도록 어떻게 안전하게(스키마 검증·실패 처리) 실행하는가?
- 6개 planning 엔티티 확정과 외부 side-effect(Slack/git)를 어떻게 원자성·멱등성·가시성을 지키며 분리하는가?
- 담당자 링크·Slack·git 자격증명을 어떻게 Backend 단독 소유·보호하는가?

## 3. 목표
- PRD §12 전 엔티티 CRUD + 상태 원장.
- AI 코파일럿(`claude -p` subprocess, JSON 스키마 검증, SSE 스트림, 전 실패모드 처리).
- confirm = 단일 트랜잭션(6엔티티 + 토큰 + 문서 + outbox 행) + 빠른 반환.
- outbox poller = Slack 발송 · git 반영을 멱등키로 재시도, 가시 상태.

### 3.1 릴리즈 범위 (본 App 한정)

| 구분 | 범위 |
|---|---|
| MVP 필수 | 인증(OAuth+토큰) · 요구사항 저장 · AI 코파일럿(SSE) · planning CRUD · confirm txn · outbox poller · slack_notify · repo_reflect · 감사 로그 |
| 이번 릴리즈 포함 | 위 MVP 전체 |
| 이번 릴리즈 제외 | Redis+Dramatiq 큐 · Runner Orchestrator · Monday Adapter · Human Check 게이팅 · executor_type 분류 · 완료 요약/회고 (Phase 2/3) |

## 4. 비목표
- 로컬 코드/파일 실행(=Runner 책임). Phase 1 은 `claude -p` 로 텍스트 변환만(코드 실행 아님, [BACKEND-ADR-001](ADR/BACKEND-ADR-001.md) 참조).
- Monday/Slack inbound 액션 처리(Phase 2).
- 사용자 승인 전 외부 시스템 반영.

## 5. 사용자 / 이해관계자

| 구분 | 역할 | 관심사 |
|---|---|---|
| PM (via FRONTEND) | 요구사항 정리·확정 | AI 응답 품질/지연, confirm 신뢰성 |
| 담당자 (via 토큰 링크) | 배정 확인 | 배정 데이터 정확·최신 |
| 운영자 | 시스템 운영 | Slack/git 실패 가시성, 감사 추적 |

## 6. 핵심 시나리오 (본 App 내부)

| # | 시나리오 | 기대 결과 |
|---|---|---|
| S1 | POST /ai-actions | AIActionRun(pending→running), `claude -p` 실행, JSON 스키마 검증 후 SSE 스트림, 원문 보존 |
| S2 | POST /confirm | 6엔티티+토큰+문서+outbox 행 단일 txn, 미매핑 차단, 즉시 200 |
| S3 | outbox poller tick | slack_notify/repo_reflect 를 멱등키로 처리, backoff 재시도, 로그 갱신 |
| S4 | GET /assignments/{token} | 인증 없이 배정 요약 반환(공개 토큰) |

> 기능별 상세 흐름은 [FRD 폴더](FRD/) 참조 (추후 작성).

## 7. 주요 기능 요약 (본 App 한정)
> [BACKEND-FC](BACKEND-FC.md) 가 SSOT.

| 기능 ID | 기능명 | 한 줄 설명 | 릴리즈 범위 |
|---|---|---|---|
| F001 | 인증 | 서버측 OAuth(PM) + 담당자 공개 토큰 route | MVP |
| F002 | 요구사항 입력 저장 | RequirementInput 원문 저장 | MVP |
| F003 | AI 코파일럿 | `claude -p` subprocess + 스키마 검증 + SSE + 실패 처리 | MVP |
| F004 | planning CRUD | SystemPlanning/Business/Domain 개념 CRUD + 섹션 AI 액션 | MVP |
| F005 | 스프린트/역할/배정 CRUD | SprintPlan/AppServiceRole/AssigneeAssignment | MVP |
| F006 | confirm txn | 단일 트랜잭션 + 토큰/문서 생성 + outbox 행 + 미매핑 차단 | MVP |
| F007 | outbox poller | 백그라운드 drain, 멱등키, backoff 재시도, feature-flag | MVP |
| F008 | slack_notify | Bot token send-only, 담당자별, SlackDispatchLog | MVP |
| F009 | repo_reflect | Markdown git commit+push, content_hash 멱등, RepositoryReflectionLog | MVP |
| F010 | observability/audit | structlog + AIActionRun/SlackDispatchLog/RepositoryReflectionLog 감사 추적 | MVP |

## 8. 비기능 요구사항 (App 특화)

| 분류 | 요구사항 |
|---|---|
| AI 실행 안전성 | CLI stdout JSON 스키마 검증. 파싱실패/빈응답/refusal/timeout 각각 처리, 원문 보존, 재시도. |
| 동시성 | AI subprocess 인-프로세스 semaphore cap. 초과 시 429/대기. |
| 멱등성 | Slack = `assignment_id + notification_version`, repo = `document_id + content_hash`. side-effect 전 결정. |
| 원자성 | confirm 도메인 쓰기 + outbox 행은 단일 트랜잭션. |

## 9. 제약사항 (App 특화)
- Monday/Slack/AI/git 자격증명은 Backend 단독 보유(솔루션 ARCHITECTURE §7). 로그 금지.
- Backend 가 Claude Code CLI 를 구동하는 것은 솔루션 ARCHITECTURE §3.3 경계에 근접 — [BACKEND-ADR-001](ADR/BACKEND-ADR-001.md) 로 명시 결정.
- Human Check Required 미승인 작업의 Runner Job 생성 없음(Phase 1 은 Runner 자체 없음).

## 10. Feature Catalog / FRD 진입점

| Feature Catalog | 주요 FRD |
|---|---|
| [BACKEND-FC](BACKEND-FC.md) | 추후 작성 (FRD/BACKEND-FRD-001.md) |

---

## 부록 B — App 사용 errorCode (subset 인덱스)
> Phase 1 신규 errorCode 초안. 솔루션 부록 B 미배치 상태 — 확정 시 이관.

| errorCode | 발생 기능 | 의미 |
|---|---|---|
| `AI_EXEC_FAILED` | F003 | `claude -p` 비정상 종료/크래시 |
| `AI_OUTPUT_INVALID` | F003 | JSON 파싱/스키마 검증 실패 |
| `AI_EMPTY_OR_REFUSAL` | F003 | 빈 응답 또는 모델 거부 |
| `CONFIRM_UNMAPPED_ASSIGNEE` | F006 | 미매핑 slack_user_id 로 확정 차단 |
| `SLACK_DISPATCH_FAILED` | F008 | Slack 발송 실패 |
| `REPO_REFLECT_FAILED` | F009 | git commit/push 실패 |
