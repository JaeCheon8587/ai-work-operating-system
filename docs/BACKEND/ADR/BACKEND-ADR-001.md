# BACKEND-ADR-001 — AI 코파일럿 실행에 `claude -p`(Claude Code CLI headless) 사용

> 본 파일은 단일 App (`BACKEND`) 의 개별 ADR 1건이다. 결정 인덱스/메타는 [BACKEND-ADR-CATALOG](../BACKEND-ADR-CATALOG.md) 가 SSOT.

| 항목 | 값 |
|---|---|
| 문서 ID | BACKEND-ADR-001 |
| 버전 | 0.1 (Draft) |
| 상태 | Accepted |
| 작성 가정 | Phase 1 AI 코파일럿은 텍스트 변환·비판만 수행(코드 실행 아님). 사용자가 `/plan-ceo-review` 에서 명시 선택. |
| 관련 문서 | [BACKEND-ADR-CATALOG](../BACKEND-ADR-CATALOG.md) · [BACKEND-PRD](../BACKEND-PRD.md) · [BACKEND-FC](../BACKEND-FC.md) · [BACKEND-ARCHITECTURE](../BACKEND-ARCHITECTURE.md) · [FRD 폴더](../FRD/) |

## 변경 이력
| 버전 | 일자 | 변경 요약 | 작성자 |
|---|---|---|---|
| 0.1 | 2026-07-04 | 초안 — CEO review cross-model 이견 후 사용자 결정 기록 | jaecheon.jeong |

---

## ADR-001: Phase 1 AI 코파일럿을 Anthropic Messages API 가 아닌 `claude -p` subprocess 로 실행한다

- **상태**: Accepted (2026-07-04)
- **우선순위**: P0
- **컨텍스트**:
  - Phase 1 AI 코파일럿은 PM 자유 입력을 구조화 PM 설계 문서로 변환·비판·누락 탐색·Markdown 생성만 수행한다(코드베이스 접근·도구 실행·agentic 동작 없음).
  - 솔루션 ARCHITECTURE §3.3 은 Claude Code CLI/PTY 소유를 Runner Server 책임으로 규정하고, §4.2 각주는 Runner 의 CLI 세션을 "로컬 실행 도구 내부 동작"으로 본다. Backend 가 CLI 를 구동하면 이 경계에 근접한다.
  - CEO review 및 Codex outside-voice 는 둘 다 Anthropic Messages API(스트리밍 + 구조화 출력)를 권고했다(CLI 를 Backend 에 넣으면 CLI 설치·로그인 세션·subprocess 정리·호스트 의존성 유입, 텍스트 변환에 agentic CLI 는 과잉).
  - 반대 맥락: 조직이 이미 Claude Code CLI 를 사용하고, Claude 구독 과금 재사용·팀 익숙도 이점이 있다. 사용자가 이 트레이드오프를 인지한 상태에서 `claude -p` 유지를 명시 결정했다.
- **결정**:
  - Phase 1 AI 코파일럿은 `claude -p "<prompt>" --output-format json` subprocess 로 실행한다.
  - 다음 완화책을 **필수** 적용한다:
    1. stdout JSON 을 `--output-format json` 신뢰만 하지 않고 **Pydantic 스키마 검증**. 파싱실패/빈응답/refusal/절단 각각 실패 처리 + 원문 보존 + 재시도.
    2. Backend Docker 이미지에 Claude Code CLI + 자격증명(auth) 전략 명시(계정·저장 위치·회전·쿼터 귀속).
    3. subprocess **timeout + cleanup + 인-프로세스 semaphore(동시 실행 cap)**.
  - 본 결정은 "텍스트 변환" 용도에 한정한다. **코드 실행형 CLI 사용은 Phase 2 Runner Server 로 이관**한다.
- **결과**:
  - 가능: Claude 구독 과금·팀 CLI 익숙도 재사용, 별도 API 키 관리 없이 착수.
  - 제약/부채: Backend 호스트에 CLI+auth 의존성, 솔루션 ARCHITECTURE §3.3 경계 근접(본 ADR 로 의도적 기록). subprocess 실패 모드를 직접 관리. Phase 2 에서 API 전환 재평가 여지([BACKEND-FC](../BACKEND-FC.md) Backlog).
- **대안 검토**:
  - 옵션 A (Anthropic Messages API): 기각 — CEO/Codex 권고였으나 사용자가 CLI 이점(과금·익숙도) 우선. Phase 2 재평가 대상.
  - 옵션 B (PTY/portable-pty full 세션): 기각 — Runner(Phase 2/3) 영역, 텍스트 변환에 과도.

### 코드 인용
- `backend/app/services/ai_copilot/` — 실행 어댑터 위치 (구현 예정, [BACKEND-ARCHITECTURE §3·§4](../BACKEND-ARCHITECTURE.md) 근거).

### 문서 반영
- [BACKEND-ADR-CATALOG](../BACKEND-ADR-CATALOG.md) — Accepted 행 등재 완료.
- [BACKEND-PRD](../BACKEND-PRD.md) — §4 비목표·§9 제약에 반영 완료.
- [BACKEND-ARCHITECTURE](../BACKEND-ARCHITECTURE.md) — §4 claude -p 경계 결정 반영 완료.
- [BACKEND-FC](../BACKEND-FC.md) — F003 기능 요구 추적에 ADR-001 참조 완료.
