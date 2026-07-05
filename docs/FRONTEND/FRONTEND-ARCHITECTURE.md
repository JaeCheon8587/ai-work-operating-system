# FRONTEND-ARCHITECTURE — 작업 운영 콘솔 호스트 아키텍처

> App별 ARCHITECTURE. 솔루션 공통 룰은 [솔루션 ARCHITECTURE](../ARCHITECTURE.md) SSOT 우선. 본 문서는 호스트 특이 사항만 보유.

| 항목 | 값 |
|---|---|
| 문서 ID | FRONTEND-ARCHITECTURE |
| 버전 | 0.1 (Draft) |
| App 코드 | FRONTEND |
| 작성 가정 | 솔루션 공통 룰([../ARCHITECTURE.md](../ARCHITECTURE.md)) 준수. Phase 1 = PM 콘솔 + 담당자 뷰. |
| 관련 문서 | [솔루션 ARCHITECTURE](../ARCHITECTURE.md) · [FRONTEND-PRD](FRONTEND-PRD.md) · [FRONTEND-FC](FRONTEND-FC.md) · [FRONTEND-ADR-CATALOG](FRONTEND-ADR-CATALOG.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — Next.js 호스트 아키텍처 부트스트랩 | jaecheon.jeong |

## 1. App 개요

| 항목 | 값 |
|---|---|
| App 코드 | FRONTEND |
| 한 줄 설명 | PM이 AI와 함께 PM 설계 문서를 작성·확정하는 작업 운영 콘솔 |
| 런타임 | Next.js (App Router) · React · TypeScript |
| 진입점 경로 | `frontend/app/layout.tsx` |
| 호스트 종류 | 웹 SPA/SSR |

내부 레이어 (tech_stack §3.4 기준): `app/`(라우팅=Presentation) · `features/`(도메인별 상태·로직=Application) · `components/`(UI) · `lib/api`(Backend 클라이언트=Infrastructure) · `store/`(Zustand UI 상태).

## 2. 핵심 책임 (4단계 마커)

- **반드시** 작업 목록/섹션별 PM 설계 편집 UI, AI 패널(chat + 섹션 액션), confirm 화면(side-effect 상태), 담당자 읽기전용 뷰 제공.
- **반드시** 서버 데이터는 TanStack Query 로 조회/캐시, UI 선택 상태는 Zustand.
- **허용** Backend REST 호출 + SSE 구독. FastAPI OpenAPI → `openapi-typescript` 타입 사용.
- **금지** 영속 비즈니스 상태 소유, 승인 게이팅·분류 판정 자체 결정.
- **절대 금지** Monday / Slack / AI API / Runner Server 직접 호출, AI API Key·Token 보유(솔루션 ARCHITECTURE §4.2).

## 3. 외부 IO Adapter 위치 정책
- Backend API 호출은 `lib/api/`(Infrastructure) 에 집중. feature 컴포넌트는 TanStack Query 훅 경유로만 접근.
- SSE 구독 로직은 `lib/api/sse.ts`(가칭) 단일 위치. AI 액션 스트림 파싱·재연결은 여기서 처리, feature 는 상태만 소비.

## 4. App 특이 도메인/패턴 룰 (있을 때만)
- 담당자 뷰(`app/assignments/[token]`)는 인증 없는 공개 route — PM 콘솔 route 와 layout·권한 분리.
- AI 액션 실행 중 해당 섹션 액션 버튼 비활성(중복 제출 방지) — `AIActionRun` 상태를 Backend 에서 구독해 판정.

## 5. 솔루션 SSOT 인용

본 App 작업 시 솔루션 공통 룰 **반드시** 준수:
- [§2 시스템 아키텍처](../ARCHITECTURE.md#2-시스템-아키텍처)
- [§3.1 Frontend 책임](../ARCHITECTURE.md#31-frontend)
- [§4.2 절대 금지 매트릭스](../ARCHITECTURE.md#42-절대-금지-매트릭스-시스템-레벨)
- [§7 보안 경계](../ARCHITECTURE.md#7-보안-경계)

솔루션 룰과 본 App 룰 충돌 시 **솔루션 ARCHITECTURE 우선**.
