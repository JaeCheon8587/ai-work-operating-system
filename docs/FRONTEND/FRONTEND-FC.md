# FRONTEND-FC — 작업 운영 콘솔 Feature Catalog

> 본 FC 는 단일 App (`FRONTEND`) 의 기능 레지스트리. SYSTEM_CODE SSOT 는 [/CLAUDE.md Backend Services Overview](../../CLAUDE.md). 솔루션 전체 시야는 [솔루션 PRD](../prd.md).

| 항목 | 값 |
|---|---|
| 문서 ID | FRONTEND-FC |
| 버전 | 0.1 (Draft) |
| 작성 가정 | Phase 1 MVP 기능만 등재. Runner/Monday/회고 UI 는 Backlog. |
| 관련 문서 | [FRONTEND-PRD](FRONTEND-PRD.md) · [FRONTEND-ARCHITECTURE](FRONTEND-ARCHITECTURE.md) · [FRONTEND-ADR-CATALOG](FRONTEND-ADR-CATALOG.md) · [FRD 폴더](FRD/) · [솔루션 PRD](../prd.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — Phase 1 기능 9종 등재 | jaecheon.jeong |

> 정식 기능 F001~F099, Backlog F101~. 미작성 컬럼은 "미작성/추후" 명기.

## App 개요

| 항목 | 요약 |
|---|---|
| App명 | 작업 운영 콘솔 (PM Planning Console) |
| 역할 | PM이 AI와 함께 PM 설계 문서를 작성·확정하는 웹 SPA/SSR |
| 목적 | 자유 입력 → 구조화 문서 + AI 비판 + 담당자 배정/전달 |
| 주요 기능 범위 | 요구사항 입력 · AI 패널 · 개념/도메인/스프린트/역할/담당자 편집 · confirm · 담당자 뷰 |
| 범위 밖 | Runner/Monday 상태 UI · 완료 요약/회고 뷰 · 실시간 WebSocket (Phase 2+) |

## 기능 레지스트리

### 기본 식별·설명
| 기능 ID | 기능명 | 기능 설명 | 기능 상태 | 구현 상태 | 테스트 상태 | 우선순위 |
|---|---|---|---|---|---|---|
| F001 | 요구사항 입력 화면 | 자유 입력 + 원문 저장 트리거 | Ready | Not Started | 미작성 | P0 |
| F002 | AI 패널 | chat + 섹션별 액션 버튼, SSE 스트림 소비, 실행 중 버튼 비활성 | Ready | Not Started | 미작성 | P0 |
| F003 | 비즈니스 개념 편집 | 개념 추가/수정/삭제 | Ready | Not Started | 미작성 | P0 |
| F004 | 도메인 개념 편집 | 비즈니스↔도메인 매핑 편집 | Ready | Not Started | 미작성 | P0 |
| F005 | 스프린트 편집 | 목표·포함/제외 범위·도메인 연결 | Ready | Not Started | 미작성 | P0 |
| F006 | 앱/서비스 역할 정의 | 역할·책임·의존성 후보 | Ready | Not Started | 미작성 | P0 |
| F007 | 담당자 배정 | 앱/서비스별 배정, 미배정·미매핑 표시 | Ready | Not Started | 미작성 | P0 |
| F008 | confirm 화면 | 확정 + side-effect 개별 상태 + 재발송 | Ready | Not Started | 미작성 | P0 |
| F009 | 담당자 읽기전용 뷰 | 토큰 링크(로그인X) 배정 요약 | Ready | Not Started | 미작성 | P0 |

> **우선순위 정의**: P0 = MVP 필수 / 출시 차단. P1 = 이번 릴리즈 권장. P2 = Backlog.

### 문서 연결
| 기능 ID | 관련 App PRD | 관련 FRD | 관련 API Spec | 관련 UI Spec | 관련 Data Spec |
|---|---|---|---|---|---|
| F001~F009 | [FRONTEND-PRD §7](FRONTEND-PRD.md#7-주요-기능-요약-본-app-한정) | 미작성/추후 | Backend OpenAPI (추후) | 미작성/추후 | [솔루션 PRD §12](../prd.md) |

### 기능 요구 추적
| 기능 ID | 작업 유형 | 사용자 영향 | 문서 영향 | 완료 기준 |
|---|---|---|---|---|
| F001~F009 | 신규 | PM/담당자 화면 신규 | FC / FRD (추후) | FRD §17 수용 기준 (추후 작성) |

### 타 App 협력 흐름
| 기능 ID | 협력 App | 협력 기능 ID | 협력 형태 |
|---|---|---|---|
| F002 | BACKEND | F003 (AI 코파일럿) | REST 호출 + SSE 구독 |
| F008 | BACKEND | F006 (confirm txn) | REST 호출 |
| F009 | BACKEND | F001 (담당자 토큰 route) | REST 호출(공개 토큰) |

---

## 별도 문서 미작성 항목 안내
- **API Spec**: Backend FastAPI OpenAPI 스키마 인용 (별도 미작성).
- **UI Spec**: 별도 미작성 — Phase 1 은 `/plan-design-review` 결과로 대체 예정.
- **Data Spec**: [솔루션 PRD §12](../prd.md) 인용.
- **Test Case**: E2E(Playwright) 단계에서 별도 작성.

---

## 확장 후보 기능 (Backlog)

| 기능 ID | 기능명 | 설명 | 상태 | 우선순위 | 근거 |
|---|---|---|---|---|---|
| F101 | Runner 상태·로그 UI | 실행 상태/로그 스트리밍 뷰 | Backlog | P2 | [솔루션 PRD §17](../prd.md), Phase 2 |
| F102 | Monday/Slack 상태 패널 | 연동 결과 표시 | Backlog | P2 | Phase 2 |
| F103 | 완료 요약/회고 뷰 | 스프린트 회고 표시 | Backlog | P2 | Phase 3 |
