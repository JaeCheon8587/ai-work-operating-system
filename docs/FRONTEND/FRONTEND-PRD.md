# FRONTEND-PRD — 작업 운영 콘솔 (PM Planning Console)

> **본 App 의 product 요구사항** (시점 = 솔루션 내 FRONTEND App 포커스). 솔루션 전체 시야 = [솔루션 PRD](../prd.md) · [/CLAUDE.md](../../CLAUDE.md). 기술 시야 = [FRONTEND-ARCHITECTURE](FRONTEND-ARCHITECTURE.md). 기능 정의 SSOT = [FRONTEND-FC](FRONTEND-FC.md).

| 항목 | 값 |
|---|---|
| 문서 ID | FRONTEND-PRD |
| 버전 | 0.1 (Draft) |
| 작성 가정 | Phase 1 MVP = PM Planning & Sprint Assignment. FRONTEND 는 PM 콘솔 + 담당자 읽기전용 뷰만 담당. Runner/Monday/Queue UI 없음. |
| 관련 문서 | [솔루션 PRD](../prd.md) · [FRONTEND-FC](FRONTEND-FC.md) · [FRONTEND-ARCHITECTURE](FRONTEND-ARCHITECTURE.md) · [FRONTEND-ADR-CATALOG](FRONTEND-ADR-CATALOG.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — Phase 1 PM Planning Console 부트스트랩 | jaecheon.jeong |

---

## 1. 배경
- PM이 자유 입력한 요구사항을 AI와 함께 구조화된 PM 설계 문서로 변환·비판·보완하는 웹 콘솔이 필요하다.
- 솔루션 3앱 중 유일한 사용자 직접 대면 표면. Backend 하고만 통신한다(솔루션 ARCHITECTURE §3.1).

## 2. 문제 정의
- PM이 ChatGPT처럼 자유 입력하되, 결과를 섹션별 정해진 포맷 문서로 편집·확정할 수 있어야 하는가?
- AI 비판/누락 탐색/범위 축소 요청을 버튼 한 번으로 트리거하고, 결과 스트림을 실시간으로 볼 수 있어야 하는가?
- 저장/확정 후 Slack 발송·문서 반영 상태를 사람이 silent failure 없이 확인할 수 있어야 하는가?

## 3. 목표
- 섹션별 PM 설계 문서 편집 UI + AI 패널(chat + 섹션별 액션 버튼).
- AI 액션 결과를 SSE 로 실시간 스트리밍.
- confirm 후 side-effect(Slack/문서 반영) 상태를 개별 노출 + 재발송 트리거.
- 담당자는 로그인 없이 토큰 링크로 배정 내용을 읽는다.

### 3.1 릴리즈 범위 (본 App 한정)

| 구분 | 범위 |
|---|---|
| MVP 필수 | 요구사항 입력 · AI 패널(SSE) · 개념/도메인/스프린트/역할/담당자 편집 · confirm 화면(side-effect 상태) · 담당자 읽기전용 뷰 |
| 이번 릴리즈 포함 | 위 MVP 전체 |
| 이번 릴리즈 제외 | Runner 상태·로그 UI · Monday/Slack 상태 패널 · 완료 요약/회고 뷰 · 실시간 WebSocket (Phase 2+) |

## 4. 비목표
- 영속 비즈니스 상태를 Frontend 에 소유하지 않는다(원장은 Backend).
- 승인 게이팅·분류 판정 등 비즈니스 규칙을 Frontend 가 결정하지 않는다.
- Monday/Slack/AI API/Runner 직접 호출 금지(솔루션 ARCHITECTURE §4.2).

## 5. 사용자 / 이해관계자

| 구분 | 역할 | 관심사 |
|---|---|---|
| PM / 프로젝트 매니저 / 서비스 기획자 (1차) | 요구사항을 PM 설계 문서로 정리·확정 | 빠른 자유 입력 → 구조화, AI 비판 품질, 확정 후 상태 가시성 |
| 담당자 (개발자/QA/운영, 2차) | 배정 내용 확인 | Slack 링크로 내 담당 앱/서비스·역할·도메인 개념 읽기 |

## 6. 핵심 시나리오 (본 App 내부)

| # | 시나리오 | 기대 결과 |
|---|---|---|
| S1 | PM이 자유 요구사항 입력 → "AI 정리" | 섹션별 초안이 SSE 로 스트리밍되어 채워진다 |
| S2 | PM이 섹션 편집 후 "비판" 버튼 클릭 | AI 검토 결과(문제점/이유/제안)가 패널에 표시된다 |
| S3 | 앱/서비스 역할·담당자 배정 후 confirm | 미매핑 담당자 있으면 차단, 없으면 확정 + side-effect 상태 노출 |
| S4 | 담당자가 Slack 토큰 링크 오픈 | 로그인 없이 배정 요약 화면 렌더 |

> 기능별 상세 흐름은 [FRD 폴더](FRD/) 참조 (추후 작성).

## 7. 주요 기능 요약 (본 App 한정)
> [FRONTEND-FC](FRONTEND-FC.md) 가 SSOT.

| 기능 ID | 기능명 | 한 줄 설명 | 릴리즈 범위 |
|---|---|---|---|
| F001 | 요구사항 입력 화면 | 자유 입력 + 원문 저장 트리거 | MVP |
| F002 | AI 패널 | chat + 섹션별 액션 버튼, SSE 스트림 소비 | MVP |
| F003 | 비즈니스 개념 편집 | 추가/수정/삭제 | MVP |
| F004 | 도메인 개념 편집 | 비즈니스↔도메인 매핑 편집 | MVP |
| F005 | 스프린트 편집 | 목표·포함/제외 범위·도메인 연결 | MVP |
| F006 | 앱/서비스 역할 정의 | 역할·책임·의존성 후보 | MVP |
| F007 | 담당자 배정 | 앱/서비스별 배정, 미배정·미매핑 표시 | MVP |
| F008 | confirm 화면 | 확정 + side-effect 개별 상태(disabled/pending/succeeded/failed/retrying/skipped) + 재발송 | MVP |
| F009 | 담당자 읽기전용 뷰 | 토큰 링크(로그인X) 배정 요약 | MVP |

## 8. 비기능 요구사항 (App 특화)

| 분류 | 요구사항 |
|---|---|
| 반응성 | AI 액션은 SSE 로 토큰 단위 스트리밍. 액션 실행 중 해당 섹션 버튼 비활성(중복 제출 방지). |
| 상태 커버리지 | 모든 섹션·side-effect 는 loading / empty / error / success / partial 상태를 표시. |
| 타입 안정성 | Backend FastAPI OpenAPI 스키마에서 `openapi-typescript` 로 타입 생성. |

## 9. 제약사항 (App 특화)
- Backend REST + SSE 하고만 통신. 외부 API 직접 호출 절대 금지(솔루션 ARCHITECTURE §4.2).
- 담당자 뷰는 인증 없는 공개 토큰 route — 민감 정보 최소 노출.

## 10. Feature Catalog / FRD 진입점

| Feature Catalog | 주요 FRD |
|---|---|
| [FRONTEND-FC](FRONTEND-FC.md) | 추후 작성 (FRD/FRONTEND-FRD-001.md) |
