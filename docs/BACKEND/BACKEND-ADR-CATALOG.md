# BACKEND-ADR-CATALOG — 상태 원장 + AI 운영 코어 ADR Catalog

> ADR 결정 인덱스. 단일 App (`BACKEND`) 의 ADR 인덱스. 새 ADR 등재 시 [ADR 폴더](ADR/) 개별 파일(`BACKEND-ADR-{NNN}.md`) 신규 + 본 카탈로그 행 추가(2곳 동기화).

| 항목 | 값 |
|---|---|
| 문서 ID | BACKEND-ADR-CATALOG |
| 작성 가정 | ADR 본문(개별 파일)과 1:1 동기화. 본 카탈로그가 상태/영향 범위/반영 문서 SSOT. |
| 관련 문서 | [ADR 폴더](ADR/) · [BACKEND-PRD](BACKEND-PRD.md) · [BACKEND-FC](BACKEND-FC.md) · [BACKEND-ARCHITECTURE](BACKEND-ARCHITECTURE.md) · [솔루션 ARCHITECTURE](../ARCHITECTURE.md) · [/CLAUDE.md](../../CLAUDE.md) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — BACKEND-ADR-001(claude -p 실행 경계) Accepted 등재 | jaecheon.jeong |

---

## Accepted
> 채택된 결정. 영향 범위·반영 문서는 본 표가 SSOT.

| ADR | 제목 | 일자 | 영향 범위 | 영향 모듈 | 반영 문서 |
|---|---|---|---|---|---|
| [BACKEND-ADR-001](ADR/BACKEND-ADR-001.md) | AI 코파일럿 실행에 `claude -p`(Claude Code CLI headless) 사용 | 2026-07-04 | AI 코파일럿 실행 경계 (솔루션 ARCHITECTURE §3.3 근접) | `services/ai_copilot/` | [BACKEND-PRD §4·§9](BACKEND-PRD.md) · [BACKEND-ARCHITECTURE §4](BACKEND-ARCHITECTURE.md) · [BACKEND-FC F003](BACKEND-FC.md) |

## Proposed
> 제안 중. 결정 기한 명시로 Open 무한정 방지.

없음.

## Deprecated / Superseded
> 폐기 또는 후속 ADR 로 교체.

없음.
