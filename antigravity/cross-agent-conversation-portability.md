# Cross-Agent Conversation Portability (대화 상태 이식성) 평가

> **작성자:** Antigravity  
> **작성 기준일:** 2026-06-24  
> **분석 방법:** Claude Code, Cline, Hermes Agent는 **소스코드 직접 분석** 기반
>
> **별점 범례:**
> | 등급 | 의미 |
> |:---:|:---|
> | ★★★★★ | 업계 최고 수준 — 네이티브 성숙, 검증된 강점 |
> | ★★★★ | 1급 — 견고한 기본 지원 |
> | ★★★ | 실용 가능하나 품질/범위에 분명한 한계 |
> | ★★ | 부분 지원 — 제한적, 간접적·외부 보완 필요 |
> | ★ | 미지원/미확인 — 외부 도구로 직접 구축해야 함 |

---

## 평가 개요

### 평가 항목: Cross-Agent Conversation Portability (에이전트 간 대화 상태 이식성)

**정의:** 하나의 AI 에이전트에서 수행한 대화(계획·맥락·결정사항·작업 이력)를 **다른 에이전트로 이전(migration)하여 이어서 작업**할 수 있는 능력.

**왜 중요한가:** 사내 AI 에이전트별 토큰 정책(비용 및 할당량)과 각 에이전트가 잘하는 역할(계획·구현·리뷰)에 맞춰 사용을 분리해야 할 현실적 필요성이 크며, 이를 위해 대화 상태를 끊김 없이 넘겨주어 **비용 효율적이면서도 최적화된 에이전트 파이프라인을 구성**할 수 있어야 하기 때문이다.

**사용 시나리오:**
- Planner(Codex CLI) → Coder(Claude Code) → Reviewer(Cline)
- Codex CLI에서 시작 → Claude Code로 전환 → Cline에서 이어서 구현

### 평가 기준 (5개 하위 축)

| 축 | 설명 |
|:---|:---|
| **A. 대화 상태 영속화** | 대화 이력을 파일/DB에 저장하여 세션 종료 후에도 유지하는가? |
| **B. 표준 포맷 내보내기** | 대화 상태를 JSON/JSONL/Markdown 등 범용 포맷으로 내보낼 수 있는가? |
| **C. 외부 상태 가져오기** | 다른 에이전트가 생성한 대화/맥락을 로드하여 이어서 작업할 수 있는가? |
| **D. 프로토콜 수준 핸드오프** | 에이전트 간 대화를 전달하는 네이티브 핸드오프 메커니즘이 있는가? |
| **E. 프로젝트 규칙 파일 호환성** | AGENTS.md/CLAUDE.md 등 규칙 파일을 통한 간접적 맥락 공유가 가능한가? |

---

## 한눈에 보는 비교표

| 평가 축 | Claude Code | Codex CLI | Antigravity | Cline | OpenClaw | oh-my-openagent | Hermes |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **A. 대화 상태 영속화** | ★★★★★ | ★★★ | ★★★★ | ★★★★★ | ★★★ | ★★ | ★★★★★ |
| **B. 표준 포맷 내보내기** | ★★★ | ★ | ★★★ | ★ | ★★ | ★ | ★★★★★ |
| **C. 외부 상태 가져오기** | ★★ | ★ | ★★ | ★ | ★ | ★★ | ★★★ |
| **D. 프로토콜 수준 핸드오프** | ★★ | ★ | ★ | ★ | ★ | ★★★ | ★★★★★ |
| **E. 규칙 파일 호환성** | ★★★★ | ★★★★ | ★★★★ | ★★★★ | ★★★ | ★★★★ | ★★★★ |
| **종합 (평균)** | **★★★ (3.2)** | **★★ (2.0)** | **★★★ (2.8)** | **★★ (2.4)** | **★★ (2.0)** | **★★ (2.4)** | **★★★★ (4.4)** |

---

## 상세 평가

### Claude Code — ★★★ (3.2/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★★★ | JSONL 기반 transcript 파일에 전체 대화 이력(메시지, 도구 사용, 파일 히스토리, attribution 스냅샷)을 영속화한다 [1]. `sessionStorage.ts`에서 세션별 JSONL 파일을 관리하며, `--resume` 플래그로 세션 복원이 가능하다 [2]. |
| B. 표준 포맷 내보내기 | ★★★ | 세션 데이터가 JSONL 파일로 디스크에 저장되어 외부 도구가 **파싱은** 가능하지만, 명시적인 `/export` 명령이나 표준화된 내보내기 API가 없다. `--fork-session`으로 세션 분기가 가능하나 [2], 이는 자체 프레임워크 내 복제이지 범용 내보내기가 아니다. |
| C. 외부 상태 가져오기 | ★★ | `--resume <session-id>` 또는 `--continue`로 **자기 자신의** 이전 세션만 복원 가능하다 [2]. 다른 에이전트(Codex, Cline 등)가 생성한 대화를 직접 가져오는 기능은 전혀 없다. `CLAUDE.md`를 통한 간접 전달은 가능하나 대화 상태 자체의 이전이 아니다. |
| D. 프로토콜 수준 핸드오프 | ★★ | 서브에이전트 간 핸드오프 분류(handoff classifier)는 있으나 [3], 이는 **동일 프레임워크 내** 서브에이전트 결과 검증 용도에 불과하다. 이종 에이전트 간 대화 전달 프로토콜은 없다. |
| E. 규칙 파일 호환성 | ★★★★ | `CLAUDE.md` (프로젝트·사용자·엔터프라이즈 계층)가 다른 에이전트의 `AGENTS.md`, `.clinerules`와 동일 디렉토리에 공존 가능하나, CLAUDE.md는 Claude Code 전용 포맷이어서 타 에이전트가 의미론적으로 활용하기엔 제한적이다 [4]. |

**시나리오 실현 가능성:** Claude Code에서 작업 → CLAUDE.md에 결정사항 기록 → 다른 에이전트가 CLAUDE.md 로드. 대화 이력 자체의 이전은 불가하나, transcript JSONL 파일을 수동 파싱하여 맥락 문서로 변환하는 워크어라운드가 가능.

---

### Codex CLI — ★★ (2.0/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★ | `resume` 명령으로 직전 세션을 복원할 수 있으나 [5], 체계적 DB 저장이 아닌 직전 세션 상태 캐시 수준이다. 장기 영속성에 분명한 한계가 있다. |
| B. 표준 포맷 내보내기 | ★ | 표준화된 세션 내보내기 기능이 공식적으로 문서화되어 있지 않다. 세션 데이터는 내부 형식으로만 관리된다 [5]. |
| C. 외부 상태 가져오기 | ★ | 자체 세션 `resume`만 지원한다. 외부 에이전트의 대화 상태를 가져오는 메커니즘이 전혀 없다 [5]. |
| D. 프로토콜 수준 핸드오프 | ★ | 이종 에이전트 간 핸드오프 프로토콜이 없다. Codex 데스크탑 앱에서 복수 에이전트 병렬 실행은 가능하나 [6], 동일 프레임워크 내의 기능이며 대화 상태 전달이 아니다. |
| E. 규칙 파일 호환성 | ★★★★ | `AGENTS.md` 파일을 통해 프로젝트 규칙을 다른 에이전트와 공유할 수 있다 [7]. 범용적인 간접 맥락 전달 수단이나 대화 이력 자체는 전달 불가. |

**시나리오 실현 가능성:** 제한적. Codex CLI에서 계획 수립 → AGENTS.md에 계획 기록 → 다른 에이전트가 AGENTS.md 로드. 대화 이력의 직접 이전은 불가.

---

### Antigravity — ★★★ (2.8/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★★ | `transcript.jsonl` 파일에 전체 대화 로그를 영속화하고 [8], KI 시스템으로 큐레이션된 컨텍스트를 자동 제공하며 아티팩트가 파일시스템에 영속된다. 다만 Hermes/Cline 수준의 구조화된 DB 기반 세션 관리에는 미치지 못한다. |
| B. 표준 포맷 내보내기 | ★★★ | 아티팩트(implementation_plan.md, walkthrough.md, task.md)가 Markdown으로 디스크에 저장되어 다른 에이전트가 읽을 수 있다 [8]. 그러나 명시적 `/export` 명령이나 대화 이력 전체를 내보내는 전용 API는 없다. |
| C. 외부 상태 가져오기 | ★★ | KI 시스템으로 과거 세션의 큐레이션된 지식을 자동 로드하지만 [8], 다른 에이전트(Claude Code, Cline 등)의 대화 형식을 직접 가져오는 네이티브 기능은 없다. |
| D. 프로토콜 수준 핸드오프 | ★ | 서브에이전트 시스템(Dynamic Subagents, Browser Subagent)은 동일 프레임워크 내에서만 작동한다 [9]. 이종 에이전트 간 핸드오프 프로토콜이 전혀 없다. |
| E. 규칙 파일 호환성 | ★★★★ | KI, 아티팩트, 프로젝트 규칙 파일이 모두 파일시스템 기반이므로 다른 에이전트가 자연스럽게 접근 가능하다 [8]. AGENTS.md도 자동 로드한다. |

**시나리오 실현 가능성:** Antigravity에서 implementation_plan.md 작성 → 다른 에이전트가 해당 파일을 읽고 구현. 아티팩트 시스템의 구조화된 산출물이 간접 핸드오프 매체로 효과적.

---

### Cline — ★★ (2.4/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★★★ | SQLite DB에 대화 이력을 보관하며, 세션 매니페스트 스토어(`session-manifest-store.ts`)를 통해 메시지를 영속화한다 [10]. 팀 런타임 상태도 `team-persistence-store.ts`를 통해 `exportState()`로 직렬화된다 [10]. |
| B. 표준 포맷 내보내기 | ★ | 팀 런타임 상태의 `exportState()` API가 존재하나 [10], 사용자 대면의 명시적 범용 포맷 내보내기 커맨드는 전무하다. SQLite DB를 직접 해킹하거나 외부 스크립트로 구축해야 하므로 미지원에 가깝다. |
| C. 외부 상태 가져오기 | ★ | SDK의 `portable-subagents` 플러그인이 `read_handoff` 도구를 제공하나 [11], 이는 **동일 Cline 생태계 내** 서브에이전트 파일만 읽을 수 있다. 타 에이전트(Codex 등)의 대화 형식을 가져오는 기능은 전혀 없다. |
| D. 프로토콜 수준 핸드오프 | ★ | `portable-subagents`와 팀 오케스트레이션 메일박스 [12]는 모두 **내부(Intra-agent) 전용**이다. 이종 프레임워크(Claude Code, Codex 등)와 대화를 전달하는 크로스 에이전트 핸드오프 프로토콜은 전무하다. |
| E. 규칙 파일 호환성 | ★★★★ | `.clinerules`, `.cline/agents/`, `.cline/skills/`가 파일시스템 기반이므로 다른 에이전트와 간접 공유가 용이하다 [13]. AGENTS.md도 자동 로드한다. |

**시나리오 실현 가능성:** 이종 에이전트 간 직접 이전 불가. Cline 내부 서브에이전트 간에서만 `save_handoff`/`read_handoff`로 네이티브 동작할 뿐, 타 도구와의 연동은 불가능하다.

---

### OpenClaw — ★★ (2.0/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★ | `MEMORY.md`와 일별 로그(`memory/YYYY-MM-DD.md`)로 세션 데이터를 영속화하지만 [14], 구조화된 DB 기반이 아닌 Markdown 파일 수준이다. |
| B. 표준 포맷 내보내기 | ★★ | Markdown 기반 메모리 파일이 간접적 내보내기 역할을 하나 [14], 구조화된 대화 이력 내보내기 API나 전용 커맨드는 확인되지 않는다. |
| C. 외부 상태 가져오기 | ★ | 자체 메모리 시스템 내 검색만 지원하며, 외부 에이전트의 대화 형식을 가져오는 기능은 확인되지 않는다 [14]. |
| D. 프로토콜 수준 핸드오프 | ★ | 코디네이터 → 서브에이전트 위임이 가능하나 [15], 이는 기본적인 위임이며 대화 상태를 전달하는 핸드오프 프로토콜은 없다. |
| E. 규칙 파일 호환성 | ★★★ | `MEMORY.md`, `SKILL.md` 등이 Markdown 기반이므로 공유 자체는 가능하나 [16], OpenClaw 전용 포맷이어서 타 에이전트의 규칙 파일과의 호환성은 제한적이다. |

**시나리오 실현 가능성:** 제한적. MEMORY.md를 매개로 한 간접 전달만 가능.

> **Caveat:** OpenClaw은 공식 문서가 제한적이며, 내부 세션 관리 구현의 상세 분석이 불가하여 추정이 포함됨.

---

### oh-my-openagent (OmO) — ★★ (2.4/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★ | 호스트 프레임워크(OpenCode, Codex 등)의 세션 관리에 전적으로 의존한다 [17]. OmO 자체의 독자적 대화 영속화 메커니즘이 없어 부분 지원에 해당한다. |
| B. 표준 포맷 내보내기 | ★ | 호스트 프레임워크의 내보내기 기능에 의존하며, OmO 자체의 구조화된 내보내기는 확인되지 않는다 [17]. |
| C. 외부 상태 가져오기 | ★★ | IntentGate의 의도 분류 → 에이전트 라우팅 [18]이 내부 맥락 전달 역할을 하고, Hindsight 외부 메모리와 연동하지만 [17], 이종 에이전트의 대화 형식을 직접 가져오지는 못한다. |
| D. 프로토콜 수준 핸드오프 | ★★★ | 11개 전문 에이전트 간 **IntentGate 기반 자동 라우팅** [18]이 사실상 내부 핸드오프에 해당하며, 사용자 요청이 자동 분류되어 최적 에이전트로 전달되는 구조이다. 실용적이지만 이종 프레임워크 간 핸드오프는 미지원이라는 근본적 한계가 있다. |
| E. 규칙 파일 호환성 | ★★★★ | `.opencode/agents.md` 등 규칙 파일이 파일시스템 기반으로 관리되어 다른 에이전트와의 공존이 가능하다 [19]. AGENTS.md 호환. |

**시나리오 실현 가능성:** OmO 내부에서 Prometheus(계획) → Atlas(실행) 흐름은 IntentGate가 자동 처리. 이종 에이전트 간에는 규칙 파일 기반 간접 전달만 가능.

> **Caveat:** oh-my-openagent은 공개 문서 기반 분석이며, 소스코드 레벨의 상세 검증은 수행하지 않음.

---

### Hermes Agent — ★★★★ (4.4/5)

| 축 | 점수 | 근거 |
|:---|:---:|:---|
| A. 대화 상태 영속화 | ★★★★★ | SQLite `state.db`에 전체 대화 이력을 저장하며, FTS5 전문 검색으로 과거 세션 접근이 가능하다 [20]. 세션별 JSON 스냅샷(`~/.hermes/sessions/session_{sid}.json`)도 턴마다 기록된다 [21]. 비교 대상 중 가장 풍부한 영속화 인프라. |
| B. 표준 포맷 내보내기 | ★★★★★ | **비교 대상 중 유일하게 3가지 내보내기 경로 제공.** ① `/save` 명령(TUI: `session.save`)으로 JSON 저장 [22], ② `hermes sessions export` CLI로 JSONL 일괄 내보내기 [21], ③ 웹 대시보드 `export_session_endpoint` API [23]. |
| C. 외부 상태 가져오기 | ★★★ | `hermes import`로 세션 내보내기/프로필 아카이브를 복원할 수 있고 [21], 세션 resume으로 과거 세션을 재개할 수 있다. 다만 이종 프레임워크(Claude Code, Cline 등)의 대화 형식을 직접 파싱·로드하는 기능은 없어 분명한 한계가 있다. |
| D. 프로토콜 수준 핸드오프 | ★★★★★ | **비교 대상 중 유일한 크로스-플랫폼 핸드오프 프로토콜.** `handoff.request` RPC [22]로 TUI/데스크탑 세션을 **메시징 플랫폼(Telegram, Slack, Discord, WhatsApp 등)으로 핸드오프**할 수 있다. 핸드오프 상태 머신(`pending→running→completed/failed`)이 구현되어 있으며, gateway의 `_handoff_watcher`가 세션을 대상 플랫폼에 재바인딩하고 synthetic turn을 생성한다 [22]. |
| E. 규칙 파일 호환성 | ★★★★ | `USER.md`, `MEMORY.md`, `AGENTS.md`가 Markdown 기반 파일시스템 관리되어 다른 에이전트와의 공존·공유가 자연스럽다 [24]. |

**시나리오 실현 가능성:** Hermes Agent 내에서는 `/save`로 대화를 JSON 내보내기 → `hermes import`로 다른 프로필에서 복원, TUI → Telegram 핸드오프가 **네이티브로 가능.** 이종 에이전트(Claude Code, Cline)와의 직접 대화 이전은 JSON 내보내기 → 수동 변환이 필요하나, 내보내기 포맷이 표준적(JSON, messages 배열)이어서 변환 난이도가 낮다.

---

## 종합 분석

### 핵심 인사이트

1. **이종 에이전트 간 직접 대화 마이그레이션은 어떤 도구도 네이티브 지원하지 않는다.** 모든 도구가 자체 프레임워크 내의 세션 복원/핸드오프만 지원하며, "Claude Code 대화를 Cline에서 이어서" 하는 시나리오는 현재 어떤 단일 도구로도 원클릭 실현이 불가하다.

2. **Hermes Agent가 가장 근접한 솔루션이다.** 3가지 내보내기 경로(JSON/JSONL/API)와 크로스-플랫폼 핸드오프(TUI↔메시징)를 네이티브로 제공하여, 대화 상태의 이식성이 가장 높다.

3. **Cline의 내부 핸드오프가 가장 구조적이다.** `portable-subagents` 플러그인의 `save_handoff`/`read_handoff`와 팀 오케스트레이션의 메일박스 메시징이 **동일 프레임워크 내**에서 가장 정교한 핸드오프를 제공한다.

4. **프로젝트 규칙 파일(AGENTS.md, CLAUDE.md, .clinerules)이 사실상 가장 실용적인 간접 마이그레이션 수단이다.** 모든 도구가 Markdown 기반 규칙 파일을 지원하므로, 이를 통한 결정사항·맥락의 간접 전달이 현실적 워크어라운드이다.

### 시나리오별 실현 가능성 요약

| 시나리오 | 실현 방법 | 난이도 |
|:---|:---|:---:|
| **Codex(계획) → Claude Code(구현)** | AGENTS.md에 계획 기록 → Claude Code가 로드 | 🟢 쉬움 |
| **Claude Code → Cline(리뷰)** | CLAUDE.md + 아티팩트 파일 → Cline이 로드 | 🟢 쉬움 |
| **Hermes TUI → Telegram 이어서** | `/handoff telegram` 네이티브 명령 | 🟢 쉬움 |
| **Cline 내 Planner→Coder** | `save_handoff`/`read_handoff` 네이티브 | 🟢 쉬움 |
| **Claude Code 대화 → Hermes에서 재개** | Claude transcript JSONL → JSON 변환 → `hermes import` | 🟡 보통 |
| **Cline 대화 → Claude Code에서 재개** | SQLite DB 추출 → 맥락 문서 변환 → CLAUDE.md에 주입 | 🔴 어려움 |
| **임의 에이전트 간 완전한 대화 이전** | 표준 프로토콜 부재 — 수동 변환 필수 | 🔴 어려움 |

---

## 향후 개선 아이디어

### Federated Agent Conversation Protocol (FACP)
MCP(Model Context Protocol)를 확장하여 **에이전트 간 대화 상태 교환 프로토콜**을 표준화할 수 있다. 핵심 요소:
- **표준 대화 스냅샷 포맷:** messages + decisions + file_changes + constraints를 구조화된 JSON으로 정의
- **핸드오프 인텐트:** `planning_complete`, `implementation_ready`, `review_requested` 등 역할 전환 시점을 선언
- **맥락 감쇠 정책:** 전체 대화 이력 대신 "결정사항 + 현재 상태 + 다음 단계"만 전달하여 토큰 효율화

이 프로토콜이 표준화되면 `Codex(계획) → Claude Code(구현) → Cline(리뷰)` 파이프라인이 원클릭으로 가능해진다.

---

## References

[1] Anthropic, "Claude Code Memory System — CLAUDE.md," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/memory

[2] Claude Code 소스코드 직접 분석: `sessionRestore.ts` — `processResumedConversation()`, `--resume`, `--fork-session`, `--continue` 세션 복원 파이프라인. `sessionStorage.ts` — JSONL transcript 영속화, `adoptResumedSessionFile()`, `loadConversationForResume()`.

[3] Claude Code 소스코드 직접 분석: `agentToolUtils.ts` — `classifyHandoffIfNeeded()`, 서브에이전트 결과에 대한 handoff classification 및 안전 검증.

[4] Anthropic, "Claude Code Skills and MCP Integration," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/skills

[5] OpenAI, "Codex CLI Configuration," github.com/openai/codex, 2026. https://github.com/openai/codex

[6] OpenAI, "Codex Desktop App — Multi-Agent Workflows," openai.com/codex, 2026. https://openai.com/index/introducing-codex/

[7] OpenAI, "Codex CLI Skills and Instructions," github.com/openai/codex, 2026. https://github.com/openai/codex/blob/main/docs/skills.md

[8] Google DeepMind, "Antigravity IDE — Knowledge Items System," antigravity.google, 2026. https://antigravity.google/docs/knowledge-items

[9] Google DeepMind, "Antigravity 2.0 — Multi-Agent Platform," blog.google, 2026. https://blog.google/technology/google-deepmind/antigravity-2-0/

[10] Cline SDK 소스코드 직접 분석: `session-manifest-store.ts` — `persistSessionMessages()` SQLite 영속화, `team-persistence-store.ts` — `exportState()` 팀 런타임 상태 직렬화, `conversation-store.ts` — 대화 ID 관리.

[11] Cline SDK 소스코드 직접 분석: `sdk/examples/plugins/agents-squad/index.ts` — `portable-subagents` 플러그인, `HANDOFFS_DIR` (`~/.cline/data/plugins/subagents/handoffs/`), `save_handoff` 도구 (conversation-scoped 파일 저장), `read_handoff` 도구 (핸드오프 파일 로드), `resolveHandoffPath()` 경로 안전 검증.

[12] Cline SDK 소스코드 직접 분석: `sdk/packages/shared/src/team/types.ts` — `MissionLogKind` 타입에 `"handoff"` 정의, `sdk/packages/shared/src/team/schema.ts` — 핸드오프 스키마 정의.

[13] Cline, "Cline Rules and Configuration," docs.cline.bot, 2026. https://docs.cline.bot/improving-your-prompting-skills/rules

[14] OpenClaw, "OpenClaw Memory System — MEMORY.md and Daily Logs," openclaw.ai, 2026. https://openclaw.ai/docs/memory

[15] OpenClaw, "OpenClaw Multi-Agent Workflows," openclaw.ai, 2026. https://openclaw.ai/docs/multi-agent

[16] OpenClaw, "OpenClaw Skills System," openclaw.ai, 2026. https://openclaw.ai/docs/skills

[17] oh-my-openagent, "OmO Memory Integration — Hindsight," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#memory

[18] oh-my-openagent, "OmO IntentGate and Multi-Model Routing," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#intentgate

[19] oh-my-openagent, "OmO MCP and Tool Integrations," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#tools

[20] Nous Research, "Hermes Agent — Always-On Architecture," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent

[21] Hermes Agent 소스코드 직접 분석: `hermes_cli/main.py` — `sessions export` CLI 명령 (L12060-12067), `db.export_session()` 호출 (L12238), JSONL 일괄 내보내기; `hermes_cli/tips.py` — `/save`, `hermes import` 팁 텍스트.

[22] Hermes Agent 소스코드 직접 분석: `tui_gateway/server.py` — `handoff.request` RPC (L4823-4908): `handoff_state='pending'` DB 기록 → gateway `_handoff_watcher`가 클레임 → 플랫폼 홈 채널에 세션 재바인딩 → synthetic turn 생성; `handoff.state` (L4911-4935): 폴링 API; `handoff.fail` (L4938-4959): 실패 마킹; `session.save` (L5223-5275): JSON 내보내기 (model, session_id, session_start, system_prompt, messages).

[23] Hermes Agent 소스코드 직접 분석: `hermes_cli/web_server.py` — `export_session_endpoint()` (L6791-6798): 웹 대시보드 단일 세션 JSON 내보내기 API.

[24] Nous Research, "Hermes Agent Memory Files — USER.md and MEMORY.md," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#memory

---

> **Caveat:** 특수목적 에이전트(OpenClaw, oh-my-openagent)에 대한 일부 정보는 공식 문서 기반 추정이 포함될 수 있다. Claude Code, Cline, Hermes Agent는 소스코드를 직접 분석하여 판정하였다. 모든 도구에서 "이종 에이전트 간 직접 대화 마이그레이션"은 현재 네이티브 미지원이므로, 평가는 각 도구의 **대화 이식에 기여하는 인프라의 성숙도**를 기준으로 수행하였다.
