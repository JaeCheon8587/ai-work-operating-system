# BACKEND-FC — 상태 원장 + AI 운영 코어 Feature Catalog

> 본 FC 는 단일 App (`BACKEND`) 의 기능 레지스트리. SYSTEM_CODE SSOT 는 [/CLAUDE.md Backend Services Overview](../../CLAUDE.md). 솔루션 전체 시야는 [솔루션 PRD](../prd.md).

| 항목 | 값 |
|---|---|
| 문서 ID | BACKEND-FC |
| 버전 | 0.1 (Draft) |
| 작성 가정 | Phase 1 MVP 기능만 등재. Runner/Monday/Queue/회고 는 Backlog. |
| 관련 문서 | [BACKEND-PRD](BACKEND-PRD.md) · [BACKEND-ARCHITECTURE](BACKEND-ARCHITECTURE.md) · [BACKEND-ADR-CATALOG](BACKEND-ADR-CATALOG.md) · [FRD 폴더](FRD/) · [솔루션 PRD](../prd.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — Phase 1 기능 10종 등재 + BACKEND-ADR-001 참조 | jaecheon.jeong |

> 정식 기능 F001~F099, Backlog F101~. 미작성 컬럼은 "미작성/추후" 명기.

## App 개요

| 항목 | 요약 |
|---|---|
| App명 | 상태 원장 + AI 운영 코어 |
| 역할 | 도메인 상태 SSOT + AI 코파일럿 + Slack/git 반영 단독 소유 |
| 목적 | PM 설계 문서 안전 생성·확정·전달·기록 |
| 주요 기능 범위 | 인증 · 요구사항 저장 · AI 코파일럿 · planning CRUD · confirm/outbox · slack_notify · repo_reflect · 감사 |
| 범위 밖 | Redis 큐 · Runner Orchestrator · Monday Adapter · Human Check · executor_type · 회고 (Phase 2/3) |

## 기능 레지스트리

### 기본 식별·설명
| 기능 ID | 기능명 | 기능 설명 | 기능 상태 | 구현 상태 | 테스트 상태 | 우선순위 |
|---|---|---|---|---|---|---|
| F001 | 인증 | 서버측 OAuth(PM) + 담당자 공개 토큰 route | Ready | Not Started | 미작성 | P0 |
| F002 | 요구사항 입력 저장 | RequirementInput 원문 저장 | Ready | Not Started | 미작성 | P0 |
| F003 | AI 코파일럿 | `claude -p` subprocess + 스키마 검증 + SSE + timeout/semaphore/실패 처리 | Ready | Not Started | 미작성 | P0 |
| F004 | planning CRUD | SystemPlanning/Business/Domain 개념 CRUD + 섹션 AI 액션 | Ready | Not Started | 미작성 | P0 |
| F005 | 스프린트/역할/배정 CRUD | SprintPlan/AppServiceRole/AssigneeAssignment | Ready | Not Started | 미작성 | P0 |
| F006 | confirm txn | 단일 트랜잭션 + 토큰/문서 + outbox 행 + 미매핑 차단 | Ready | Not Started | 미작성 | P0 |
| F007 | outbox poller | 백그라운드 drain, 멱등키, backoff 재시도, feature-flag | Ready | Not Started | 미작성 | P0 |
| F008 | slack_notify | Bot token send-only, 담당자별, SlackDispatchLog | Ready | Not Started | 미작성 | P0 |
| F009 | repo_reflect | git commit+push, content_hash 멱등, RepositoryReflectionLog | Ready | Not Started | 미작성 | P0 |
| F010 | observability/audit | structlog + 3 감사 로그 테이블 | Ready | Not Started | 미작성 | P0 |

> **우선순위 정의**: P0 = MVP 필수 / 출시 차단. P1 = 이번 릴리즈 권장. P2 = Backlog.

### 문서 연결
| 기능 ID | 관련 App PRD | 관련 FRD | 관련 API Spec | 관련 UI Spec | 관련 Data Spec |
|---|---|---|---|---|---|
| F001~F010 | [BACKEND-PRD §7](BACKEND-PRD.md#7-주요-기능-요약-본-app-한정) | 미작성/추후 | FastAPI OpenAPI (추후) | 없음 | [솔루션 PRD §12](../prd.md) |

### 기능 요구 추적
| 기능 ID | 작업 유형 | 사용자 영향 | 문서 영향 | 완료 기준 |
|---|---|---|---|---|
| F003 | 신규 | AI 응답 품질/지연 | FC / FRD / ADR (ADR-001) | FRD §17 (추후) |
| F006,F007 | 신규 | confirm 신뢰성/가시성 | FC / FRD | FRD §17 (추후) |
| F001,F002,F004,F005,F008,F009,F010 | 신규 | 없음(내부) 또는 담당자 배정 데이터 | FC / FRD (추후) | FRD §17 (추후) |

### 타 App 협력 흐름
| 기능 ID | 협력 App | 협력 기능 ID | 협력 형태 |
|---|---|---|---|
| F003 | FRONTEND | F002 (AI 패널) | REST + SSE 응답 |
| F006 | FRONTEND | F008 (confirm 화면) | REST |
| F001 | FRONTEND | F009 (담당자 뷰) | REST(공개 토큰) |

---

## 별도 문서 미작성 항목 안내
- **API Spec**: FastAPI OpenAPI 스키마 자동 생성 (별도 미작성).
- **UI Spec**: 없음 (Backend).
- **Data Spec**: [솔루션 PRD §12](../prd.md) 인용.
- **Test Case**: pytest 단계에서 별도 작성.

---

## 확장 후보 기능 (Backlog)

| 기능 ID | 기능명 | 설명 | 상태 | 우선순위 | 근거 |
|---|---|---|---|---|---|
| F101 | Redis+Dramatiq 큐 | outbox poller → 정식 큐 전환 | Backlog | P1 | [솔루션 tech_stack §16](../tech_stack.md), Phase 2 |
| F102 | Runner Orchestrator | RunnerJob 생성·상태·결과 수신 | Backlog | P2 | Phase 2 |
| F103 | Monday Adapter | Board/Item/Comment 연동 | Backlog | P2 | Phase 2 |
| F104 | AI 동시성 큐 제어 | subprocess semaphore → 큐 기반 | Backlog | P2 | CEO review Codex #6 |
| F105 | 토큰 링크 생명주기 강화 | 자동 회전 + 접근 감사 + 구 링크 무효화 | Backlog | P2 | CEO review Codex #12 |
| F106 | 완료 요약/회고 생성 | AI Tracker | Backlog | P2 | Phase 3 |
