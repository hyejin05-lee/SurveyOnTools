# Coding Agents 고급 기능 비교 분석

> **작성자:** Antigravity  
> **작성 기준일:** 2026-06-17 (코드 검증 갱신: 2026-06-17)  
> **범례:** ● 1급(native·성숙) · ◐ 부분 지원(제한적·간접) · ○ 미지원/미확인  
> **분석 방법:** Claude Code, Cline, Hermes Agent는 **소스코드 직접 분석** 기반 (AGENTS.md §6 준수)

---

## 한눈에 보는 비교표

| 기능 영역 | Claude Code | Codex CLI | Antigravity | Cline | OpenClaw | oh-my-openagent | Hermes Agent |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **1. Agentic Memory** | | | | | | | |
| ├ 세션 내 대화 기억 | ● | ● | ● | ● | ● | ● | ● |
| ├ 크로스-세션 영속 메모리 | ● [1] | ◐ [7] | ● [13] | ◐ [19] | ● [26] | ◐ [33] | ● [38, 53] |
| ├ 프로젝트 레벨 규칙 파일 | ● [2] | ● [8] | ● [14] | ● [20] | ● [27] | ● [34] | ● [39] |
| └ 사용자 프로필/선호 모델링 | ● [3, 48] | ○ | ◐ [15] | ◐ [21] | ● [28] | ○ | ● [40] |
| **2. Context Compaction (토큰 절감)** | | | | | | | |
| ├ 자동 컨텍스트 압축 | ● [4, 49] | ● [9] | ● [16] | ● [22, 51, 52] | ◐ [29] | ● [35] | ● [41, 61] |
| └ 수동 컴팩션 명령 | ● [4, 50] | ● [9] | ◐ | ● [22, 51, 52, 60] | ○ | ● [35] | ◐ |
| **3. Harness Engineering (Agent Runtime)** | | | | | | | |
| ├ 프로그래머블 Agent SDK | ● [5] | ◐ [10] | ● [17] | ● [23] | ○ | ○ | ○ |
| ├ 라이프사이클 Hooks | ● [5, 49, 50] | ● [10] | ◐ | ● [23] | ○ | ● [36] | ◐ [42] |
| └ Agent 정의 파일(커스텀 에이전트) | ● [6] | ◐ | ● [18] | ● [23, 58] | ○ | ● [36] | ◐ |
| **4. 멀티/서브 에이전트 오케스트레이션** | | | | | | | |
| ├ 네이티브 서브에이전트 생성 | ● [6] | ◐ [11] | ● [17] | ● [23, 57] | ◐ [30] | ● [36] | ● [43, 55] |
| ├ 병렬 에이전트 실행 | ● [6] | ● [11] | ● [17] | ● [23, 57] | ◐ [30] | ● [36] | ● [43, 55] |
| ├ 에이전트 팀 오케스트레이션 | ● [6] | ◐ | ● [17] | ◐ [23, 58] | ○ | ● [36] | ◐ |
| └ 의도 분류/라우팅 레이어 | ○ | ○ | ○ | ○ | ○ | ● [37] | ○ |
| **5. 멀티모델 라우팅** | | | | | | | |
| ├ 복수 LLM 프로바이더 지원 | ◐ [3] | ◐ [10] | ◐ [15] | ● [25] | ● [31] | ● [37] | ● [44] |
| ├ 작업별 모델 라우팅 | ○ | ◐ | ○ | ◐ [25] | ○ | ● [37] | ● [44, 62] |
| ├ Mixture-of-Agents(MoA) 합의 | ○ | ○ | ○ | ○ | ○ | ○ | ● [62] |
| └ 로컬 모델 지원(Ollama 등) | ○ | ○ | ○ | ● [25] | ● [31] | ◐ | ● [44] |
| **6. Skills / 확장** | | | | | | | |
| ├ 모듈식 스킬(SKILL.md 등) | ● [2] | ◐ [8] | ◐ | ● [23, 58] | ● [32] | ◐ | ● [45, 54] |
| ├ 플러그인/확장 마켓플레이스 | ◐ | ○ | ○ | ◐ [24] | ● [32] | ○ | ◐ [45] |
| └ 커뮤니티 스킬 생태계 | ◐ | ○ | ○ | ◐ [24] | ● [32] | ◐ | ● [45] |
| **7. 도구 연결성 (Tool Connectivity)** | | | | | | | |
| ├ MCP (Model Context Protocol) | ● [2] | ◐ | ● [14] | ● [24] | ◐ | ● [34] | ◐ |
| ├ 브라우저 자동화 | ◐ | ○ | ● [17] | ◐ [24] | ◐ [32] | ● [34] | ● [45] |
| ├ 메시징 플랫폼 연동 | ○ | ○ | ○ | ● [59] | ● [26] | ○ | ● [38] |
| └ 외부 API/DB 직접 호출 | ● [2] | ◐ | ● [14] | ● [24] | ● [26] | ● [34] | ● [45] |
| **8. Adaptive / 개인화** | | | | | | | |
| ├ 적응형 추론(Thinking 강도 조절) | ● [4] | ◐ [9] | ● [16] | ◐ [22] | ○ | ○ | ○ |
| ├ Plan/Act 모드 전환 | ◐ | ○ | ● [18] | ● [25] | ○ | ● [36] | ○ |
| └ 자기 학습/스킬 자동 생성 | ○ | ○ | ○ | ○ | ○ | ○ | ● [45, 54] |
| **9. 안전성 / 샌드박싱** | | | | | | | |
| ├ OS 레벨 샌드박스 | ● [5] | ● [10] | ◐ | ○ | ○ | ○ | ○ |
| ├ 권한 모드(Suggest/Auto/Full) | ● [5] | ● [10] | ● [18] | ● [25] | ○ | ○ | ◐ |
| ├ 체크포인트/Rollback | ● [5] | ◐ | ◐ | ● [47] | ○ | ○ | ● [63] |
| └ Git 워크트리 격리 | ● [6] | ◐ | ○ | ○ | ○ | ○ | ○ |
| **10. 배포 환경 / 실행 형태** | | | | | | | |
| ├ CLI (터미널) | ● | ● | ● [17] | ● [23] | ● [26] | ◐ | ● [38] |
| ├ IDE 통합 | ◐ | ◐ | ● [13] | ● [23] | ○ | ○ | ○ |
| ├ 독립 데스크탑 앱 | ○ | ● [11] | ● [17] | ○ | ● [26] | ○ | ● [38] |
| ├ 상시 실행(Always-On / 데몬) | ○ | ○ | ◐ [17] | ◐ [23, 59] | ● [26] | ○ | ● [38] |
| └ 예약 작업(Cron/스케줄링) | ● [46] | ○ | ● [17] | ● [23] | ● [26] | ○ | ● [38, 56] |

---

## 상세 기능 분석

### 1. Agentic Memory

#### Claude Code
`CLAUDE.md`를 프로젝트 루트에 배치하여 세션마다 자동 로드되는 영속 메모리를 제공한다 [1]. 프로젝트 레벨(`CLAUDE.md`), 사용자 레벨(`~/.claude/CLAUDE.md`), 팀/엔터프라이즈 레벨의 다계층 메모리 구조 외에도 메모리 백엔드에서 사용자 상태를 기록하는 `Auto-memory`를 지원한다 [2]. 번들 스킬인 `remember` 스킬 [48]을 통해 자동 메모리 데이터를 검토하고 CLAUDE.md, CLAUDE.local.md 또는 팀 메모리로 승격(Promotion), 중복/충돌 제거(Cleanup & Conflict resolution) 처리를 제안하여 사용자 지침과 환경 지식을 지능적으로 교차 정돈한다.

#### Codex CLI
`~/.codex/settings.json` 및 프로젝트 레벨 `AGENTS.md` 파일을 통해 세션 간 설정을 영속화한다 [7][8]. 그러나 대화 히스토리의 크로스-세션 영속성은 제한적이며, 주로 `resume` 커맨드를 통해 직전 세션을 복원하는 수준이다 [7].

#### Antigravity
Knowledge Items(KI) 시스템을 통해 큐레이션된 리포지토리별 컨텍스트를 자동 제공한다 [13]. 아티팩트 시스템(implementation plan, walkthrough, task 등)을 통해 세션 간 작업 맥락이 영속된다 [14]. 대화 로그는 `transcript.jsonl` 형태로 파일시스템에 저장되어 과거 세션 참조가 가능하다 [13].

#### Cline
기본적으로 과거 대화 이력을 SQLite DB에 보관하여 세션 복원(Resume)을 자체 지원하지만, 크로스-세션 단위의 지식/기억 저장소 연동은 다소 제한적이다. 널리 사용되는 Memory Bank 프레임워크(`MEMORY.md`, `PLAN.md` 등) 방식은 에이전트 시스템 레벨의 자동화된 시맨틱/메모리 엔진이 작동하는 것이 아니라, 사용자가 작성한 지침에 따라 에이전트가 파일 읽기/쓰기 도구를 통해 직접 마크다운 문서를 읽고 지식 구조를 갱신해나가는 수동 워크플로 방법론에 가깝다 [19]. 따라서 전용 DB/임베딩 기반의 메모리 도구나 백엔드가 부재하므로 **부분 지원(◐)**으로 분류하는 것이 타당하다. 다만 `.clinerules` 및 `.cline/` 아래의 다양한 규칙 파일을 자동 로드하여 세션 간 설정/규칙 컨텍스트를 유지한다 [20].

#### OpenClaw
`MEMORY.md`를 큐레이션 메모리로, 일별 로그(`memory/YYYY-MM-DD.md`)를 세션 로그로 활용하는 이중 메모리 구조를 갖는다 [26][27]. 벡터 임베딩과 BM25 전문 검색을 결합한 하이브리드 검색을 지원한다 [26]. 사용자 선호·페르소나 설정이 메모리에 통합되어 개인화된 응답을 생성한다 [28].

#### oh-my-openagent (OmO)
기본적으로 호스트 프레임워크(OpenCode, Codex 등)의 메모리 메커니즘에 의존한다 [33]. Hindsight 등 외부 메모리 프로바이더와의 연동을 통한 자동 회상 기능을 지원한다 [33]. 프로젝트 규칙은 자체 `.opencode/agents.md` 또는 규칙 파일을 통해 관리한다 [34].

#### Hermes Agent
`USER.md`(사용자 프로필, 1,375자 제한)와 `MEMORY.md`(프로젝트/환경 팩트, 2,200자 제한)를 시스템 프롬프트에 주입하는 큐레이션 메모리를 제공한다 [38][39]. 코드로 보면 `tools/memory_tool.py`[53]에서 **Frozen Snapshot Pattern**을 사용하여 세션 시작 시점에만 메모리 파일을 동결 스냅샷으로 시스템 프롬프트에 로드 주입하고, 세션 도중의 수정(add/replace/remove)은 디스크에 즉시 반영하면서도 시스템 프롬프트는 갱신하지 않아 **Prefix Caching**을 깨뜨리지 않게 설계되었다. 또한 외부 도구나 수동 추가로 인한 파일 내용 오염을 막기 위한 **External Drift Detection**과 엄격한 위협 스캔(`scan_for_threats`) 검증이 상시 작동한다. 추가적으로 SQLite 기반 `state.db`에 전체 대화 이력을 저장하고, FTS5 전문 검색을 통해 과거 세션에서 컨텍스트를 검색한다 [38]. 외부 메모리 프로바이더(Honcho 등)와의 통합도 지원한다 [40].

---

### 2. Context Compaction (토큰 절감)

#### Claude Code
토큰 사용량이 임계치를 초과하면 자동으로 요약 프롬프트를 주입하여 대화 히스토리를 압축한다 [4]. 소스코드 상에서 `autoCompact.ts` [49]와 `compact.ts` [50]를 통해 제어되며, 유효 윈도우에서 13,000토큰을 버퍼로 두고 사전 작동한다. 3회 연속 컴팩션 실패 시 API 낭비를 막는 서킷 브레이커가 작동한다. 추가적으로 API의 prompt-too-long 오류에 대응하는 사후 컴팩션 `REACTIVE_COMPACT` 모드와, 90% 커밋 / 95% 블로킹 스폰의 흐름으로 사전 컨텍스트 절감을 가이드하는 `CONTEXT_COLLAPSE` 최적화 아키텍처가 구현되어 있다. 컴팩션 시 대화 내용이 요약 정리되는 과정에서 주요 파일 정보의 상실을 막기 위해 최근 읽은 파일(최대 5개, 파일당 5k 토큰) 및 활성 스킬(최대 25k 토큰 버퍼)을 attachments 형태로 재생성해 재주입하며, 계획 모드 상태도 이력 인젝션으로 유지한다. `/compact` 명령으로 수동 컴팩션도 가능하며, 특정 메시지 지점을 기준으로 양방향(from/up_to) 부분 컴팩션을 수행하는 `partialCompactConversation` 도 네이티브로 지원한다.

#### Codex CLI
동적 컨텍스트 어셈블리를 통해 리포지토리·스킬 번들·프로젝트 지침에서 관련 컨텍스트만 선택적으로 조립한다 [9]. `/compact` 명령으로 수동 정리가 가능하며, `MAX_THINKING_TOKENS` 오버라이드를 통해 추론 비용을 제어한다 [9]. 최대 약 192k–272k 토큰의 컨텍스트 윈도우를 활용한다 [9].

#### Antigravity
장시간 에이전트 작업 시 자동 컨텍스트 컴팩션을 수행한다 [16]. 요약, 도구 결과의 가상 파일시스템 오프로딩, 오래된 정보 필터링 등 다양한 기법을 활용한다 [16]. Gemini 모델의 대형 컨텍스트 윈도우(1M+ 토큰)와 결합하여 효율적으로 운영한다 [16].

#### Cline
Auto Compact 기능을 통해 모델의 컨텍스트 한계 접근 시 자동 요약 프로세스를 트리거한다 [22]. 소스코드로 보면 두 가지 하이브리드 압축 로직을 완비하고 있다. 첫째, `basic-compaction.ts`[51]에서는 최신 턴(protectedTail)을 안전하게 배제하고, 이전 메시지 중 툴 사용(`tool_use`)과 결과(`tool_result`)는 원자적으로 매핑하여 함께 삭제되도록 제어하며, Assistant와 User의 이전 메시지를 우선순위에 맞춰 순차 제거한 뒤 잔여 한도에 맞춰 텍스트를 절단한다. 둘째, `agentic-compaction.ts`[52]를 통해 이전 요약본과 새 대화를 Fold하여 LLM 요약(continuation note)을 동적으로 생성하며, 이때 대화 이력에서 파일 조작 이력(`fileOps`)을 추출하여 문서 내에 파일 조작 세션 정보로 고스란히 보존하도록 보증한다. JIT(Just-in-Time) 로딩으로 필요할 때만 파일/데이터를 컨텍스트에 적재한다 [22].

#### OpenClaw
일별 로그의 자동 요약을 통해 간접적 컴팩션을 수행하나, 범용 코딩 에이전트 대비 컨텍스트 관리가 덜 정교하다 [29]. 주요 효율화는 메모리 파일의 큐레이션과 하이브리드 검색을 통한 선택적 로딩에 의존한다 [29].

#### oh-my-openagent
라이프사이클 훅을 통해 컴팩션 전략을 주입할 수 있다 [35]. 요약 프롬프트 주입이나 히스토리 클리어링을 통해 "컨텍스트 부패"를 방지한다 [35]. 호스트 프레임워크의 컴팩션 기능도 활용 가능하다 [35].

#### Hermes Agent
코드 분석 결과, Hermes Agent는 기존에 알려진 간접적 방식 외에 `trajectory_compressor.py` [61]에서 **본격적인 LLM 기반 trajectory 압축 시스템**을 구현하고 있다. 이 시스템은 ① 첫 번째 system/human/gpt/tool 턴과 마지막 N개 턴을 보호 영역으로 지정(Protected Head/Tail), ② 중간 영역(compressible middle)만을 대상으로 필요한 만큼만 압축, ③ gpt→tool 페어의 원자적 경계를 깨뜨리지 않는 **Boundary Snapping** 알고리즘으로 tool_call-tool_response 쌍이 분리되지 않도록 보장, ④ LLM 요약 API(`gemini-3-flash-preview` 등)로 압축 턴을 `[CONTEXT SUMMARY]` 메시지로 치환, ⑤ 비동기 병렬 압축(`asyncio.gather` + `max_concurrent_requests=50`)으로 대량 trajectory를 일괄 처리하는 성숙한 파이프라인이다. 압축 목표는 15,250토큰이며, 요약 생성은 750토큰 버짓 내에서 수행된다. 압축 실패 시 jittered backoff 재시도와 fallback 메시지를 제공한다. 이에 더해 SQLite 기반 FTS5 전문 검색으로 과거 세션 컨텍스트를 선택 로딩하며 [41], 서브에이전트를 통한 컨텍스트 격리도 메인 윈도우 관리에 기여한다. 따라서 **자동 컨텍스트 압축은 1급 지원(●)**으로 상향 분류한다.

---

### 3. Harness Engineering (Agent Runtime)

#### Claude Code
Claude Agent SDK를 통해 프로그래머블 런타임을 제공한다 [5]. `query()` (단발성)와 `ClaudeSDKClient` (상태 유지 양방향) 두 가지 진입점을 지원한다 [5]. 18개 이상의 라이프사이클 이벤트(`SessionStart`, `PreToolUse`, `PostToolUse`, `SubagentStart`, `PreCompact` 등)에 콜백·셸·HTTP 훅을 등록할 수 있다 [5]. `.claude/agents/` 디렉토리에 에이전트 정의 파일(.md)을 배치하여 커스텀 에이전트를 구성한다 [6]. 컴팩션 전후 및 세션 기동 시점에 동작하는 `executePreCompactHooks`, `executePostCompactHooks`, `processSessionStartHooks` 와 같은 라이프사이클 훅([compact.ts] [50])이 시스템의 핵심 흐름과 긴밀하게 결합되어 있다.

#### Codex CLI
`config.toml` 기반 설정과 프로젝트 레벨 지침 파일을 지원한다 [10]. Hooks & Extensions를 통해 로컬 빌드 파이프라인·CI/CD 체크·외부 프로젝트 관리 도구와 통합이 가능하다 [10]. 단, 전용 SDK로의 프로그래머블 확장은 Claude Code나 Cline 대비 제한적이다 [10].

#### Antigravity
Antigravity SDK(Python)를 통해 에이전트 하네스에 대한 프로그래머블 액세스를 제공한다 [17]. 에이전트의 라이프사이클 이벤트, 훅, 디스패처를 정의할 수 있다 [17]. IDE·CLI·데스크탑 앱·SDK의 4개 표면(surface)을 통합 런타임으로 연결한다 [17]. 아티팩트 시스템(구현 계획, 워크스루, 태스크)이 에이전트 하네스의 핵심 구성 요소로 통합되어 있다 [18].

#### Cline
`@cline/sdk`는 계층형 TypeScript 스택(`@cline/core`, `@cline/agents`, `@cline/llms`, `@cline/shared`)으로 구성된다 [23]. `createTool` 함수로 커스텀 도구를 Zod 스키마 기반으로 정의하고, 플러그인으로 번들링할 수 있다 [23]. 라이프사이클 훅을 통한 정책 관리·관측성·커스텀 에이전트 동작 정의를 지원한다 [23]. VS Code·JetBrains·CLI 모두에서 동일한 SDK 런타임을 공유한다 [23]. 또한 `.cline/agents/` 및 `.cline/skills/` 디렉토리에 마크다운 형식의 에이전트 프리셋 정의 파일(systemPrompt, modelId, providerId, maxIterations 등 포함) 및 스킬 지침 정의 파일을 배치하면 런타임이 이를 동적으로 감지하여 로드한다 [23, 58].

#### OpenClaw
Gateway 아키텍처(Node.js 데몬)를 통해 세션 관리·이벤트 라우팅을 처리하지만 [26], 공식 SDK 수준의 프로그래머블 런타임은 제공하지 않는다 [26]. 확장은 주로 Skills 매니페스트를 통한 선언적 방식이다 [32].

#### oh-my-openagent
25~54개 이상의 라이프사이클 훅(`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `Stop` 등)을 지원한다 [36]. `todo-continuation-enforcer` 등 규율 훅(discipline hooks)을 통해 에이전트 동작을 강제한다 [36]. 다만 독립 SDK가 아닌 OpenCode/Codex 등 호스트 프레임워크의 확장 레이어로 동작한다 [36].

#### Hermes Agent
도구 시스템(40개 이상 빌트인 + 100개 이상 커뮤니티)을 통한 확장이 가능하지만 [42], SDK 수준의 프로그래머블 하네스보다는 플러그인 아키텍처에 가깝다 [42]. 스킬 자동 생성 및 리파인먼트 루프가 런타임 레벨에서 통합되어 있다 [45].

---

### 4. 멀티/서브 에이전트 오케스트레이션

#### Claude Code
`.claude/agents/` 에 정의된 전문 서브에이전트를 생성하여 위임한다 [6]. 각 서브에이전트는 고유 시스템 프롬프트·권한·전용 컨텍스트 윈도우를 가지며, 결과만 메인 세션에 반환하여 컨텍스트 오염을 방지한다 [6]. "리드" 에이전트가 작업을 배분하고 결과를 종합하는 에이전트 팀 패턴을 지원한다 [6]. Git 워크트리를 활용한 파일 충돌 격리도 가능하다 [6].

#### Codex CLI
Codex 데스크탑 앱에서 복수 에이전트의 병렬 실행·스레드 관리·휴먼-인-더-루프 감독을 지원한다 [11]. 병렬 도구 호출(parallel tool calling)로 복잡 작업의 벽시계 시간을 단축한다 [11]. 다만 네이티브 서브에이전트 생성보다는 "듀얼 툴" 전략(외부 에이전트 조합) 의존도가 높다 [11].

#### Antigravity
Dynamic Subagents를 통해 메인 에이전트가 병렬화된 서브에이전트를 생성·위임할 수 있다 [17]. 서브에이전트는 개발자의 주 작업 흐름을 차단하지 않으며, 별도 컨텍스트에서 독립 실행된다 [17]. 데스크탑 앱이 에이전트 상호작용 모니터링·병렬 워크플로 관리·비동기 작업 추적의 허브 역할을 한다 [17]. Browser Subagent를 통해 웹 인터랙션 전담 에이전트도 운영 가능하다 [17].

#### Cline
VS Code 확장 프로그램 및 CLI 환경에서 네이티브 서브에이전트 오케스트레이션을 지원한다 [23, 57]. `use_subagents` 도구와 `SubagentRunner`를 통해 최대 5개까지의 서브에이전트를 동시 생성하여 병렬 실행할 수 있으며 [57], 각 서브에이전트의 실시간 토큰 사용량, 비용, 실행 중인 도구 정보 등을 UI에 표출하고 결과를 취합하여 메인 세션에 반환한다. 또한 SDK 레벨의 `portable-subagents` 플러그인 [58]을 기동하여 `start_subagent`, `message_subagent`, `get_subagent` 도구로 서브에이전트들을 생성 및 위임하고, `save_handoff` 및 `read_handoff` 도구를 활용해 공통 handoffs 디렉토리 내에서 협업용 컨텍스트(handoff) 파일을 유지·공유하는 에이전트 스쿼드 오케스트레이션을 지원한다 [58].

#### OpenClaw
코디네이터 에이전트가 사용자 상호작용을 처리하고 도메인별 하위 작업을 서브에이전트에 위임하는 아키텍처를 지원한다 [30]. 그러나 범용 코딩 에이전트들의 정교한 오케스트레이션에 비해 제한적이다 [30].

#### oh-my-openagent
가장 정교한 멀티에이전트 아키텍처를 제공한다 [36]. Sisyphus(메인 오케스트레이터), Prometheus(전략 기획), Oracle(아키텍처 자문), Atlas(태스크 실행), Metis(계획 검증), Librarian(문서 리서치), Hephaestus(심층 탐색) 등 11개 전문 에이전트로 구성된다 [36]. IntentGate가 모든 사용자 메시지를 사전 분류하여 최적 에이전트·모델로 라우팅한다 [37]. "Ultrawork(`ulw`)" 모드로 자율 실행 루프를 트리거할 수 있다 [36].

#### Hermes Agent
`delegate_task` 도구를 통해 동기 및 비동기(background) 서브에이전트 생성을 지원한다 [43]. 코드를 보면 `tools/async_delegation.py` [55]에서 백그라운드 데몬 스레드 풀(`_DaemonThreadPoolExecutor`, 최대 3개 실행)로 서브에이전트를 위임하여 비동기 실행을 처리한다. 서브에이전트 실행이 완료되면, 메인 세션의 prompt cache 훼손 및 strict message-role alternation(메시지 롤 교차 보존) 규칙 위반을 막기 위해 실행 결과를 즉시 끼워넣지 않고, 메인 에이전트가 유휴 상태(idle)일 때만 completion queue를 거쳐 새로운 유저 턴으로 완료 이벤트를 전달한다. 또한 원래의 goal, context, model, toolsets 정보를 rich task-source block에 온전히 담아 리턴함으로써 컨텍스트 윈도우 이탈 후에도 부모 에이전트가 이전 맥락을 복원할 수 있게 돕는다. SQLite 기반 `state.db`에 전체 대화 이력을 저장하고, FTS5 전문 검색을 통해 과거 세션에서 컨텍스트를 검색한다 [38].

또한 코드 분석에서 발견한 `tools/mixture_of_agents_tool.py` [62]는 멀티에이전트 협업의 다른 차원을 보여준다. 이는 복수의 프론티어 LLM(Claude Opus, Gemini Pro, GPT-5.4 Pro, DeepSeek V3.2)을 reference 모델로 병렬 호출한 뒤, aggregator 모델이 응답들을 종합(synthesis)하는 Mixture-of-Agents(MoA) 아키텍처를 네이티브로 구현한 것이다. 이는 서브에이전트 위임과는 별개의, LLM 앙상블 레이어에 해당하며, 다른 비교 대상 도구에서는 발견되지 않는 고유 기능이다.

다만 oh-my-openagent 수준의 역할 기반 팀 오케스트레이션은 갖추고 있지 않다 [43].

---

### 5. 멀티모델 라우팅

#### Claude Code
기본적으로 Anthropic 모델(Claude Opus/Sonnet) 전용이다 [3]. SDK를 통해 다른 프로바이더를 연결하는 것은 이론적으로 가능하나, 네이티브 환경에서는 Anthropic 생태계에 종속된다 [3].

#### Codex CLI
OpenAI 모델(GPT-5 시리즈) 중심이지만, "빠른(Fast)" 모델과 "표준/딥(Standard/Deep)" 모델의 다계층 전략을 권장한다 [10]. `OPENAI_BASE_URL` 변경으로 다른 엔드포인트를 연결할 수 있으나, 공식적인 멀티프로바이더 지원은 아니다 [10].

#### Antigravity
Google Gemini 모델(Gemini 3.1 Pro, Gemini 3.5 Flash 등)에 최적화되어 있으며 [15], Gemini 생태계 내에서의 모델 선택은 유연하다. 그러나 타 프로바이더(Anthropic, OpenAI 등) 모델의 네이티브 지원은 제한적이다 [15].

#### Cline
완전한 모델 비종속(Model-agnostic) 아키텍처를 제공한다 [25]. Anthropic, OpenAI, Google Gemini, Mistral, 그리고 Ollama/LM Studio를 통한 로컬 모델까지 지원한다 [25]. 플래닝 단계와 실행 단계에 서로 다른 모델을 배정하는 워크플로가 가능하다 [25]. BYOK(Bring Your Own Key) 방식으로 비용·성능 최적화를 사용자에게 맡긴다 [25].

#### OpenClaw
모델 비종속 설계로 Claude, GPT-4, Gemini 및 Ollama를 통한 로컬 모델을 지원한다 [31]. 다만 작업별 모델 라우팅보다는 세션 단위의 모델 선택에 가깝다 [31].

#### oh-my-openagent
가장 적극적인 멀티모델 라우팅을 구현한다 [37]. 에이전트별로 명시적 모델 할당이 가능하여, 고수준 추론 작업에는 Claude Opus를, 단순 작업에는 GPT-mini나 Gemini Flash를 배정할 수 있다 [37]. IntentGate가 의도를 분류한 뒤 최적 모델·에이전트 조합으로 라우팅한다 [37].

#### Hermes Agent
OpenRouter 또는 로컬 엔드포인트를 통해 다양한 모델(Claude, GPT-4, Qwen 3.6 등 오픈 웨이트 모델 포함)을 지원한다 [44]. 특정 프로바이더에 종속되지 않는 유연한 모델 선택이 가능하다 [44]. 코드에서 발견한 가장 주목할 점은 `mixture_of_agents_tool.py` [62]의 네이티브 **Mixture-of-Agents(MoA)** 구현이다. 이는 4개 reference 모델(claude-opus-4.6, gemini-2.5-pro, gpt-5.4-pro, deepseek-v3.2)을 `asyncio.gather`로 병렬 호출하고, aggregator 모델(claude-opus-4.6)이 이들의 응답을 "critically evaluate"하여 단일 고품질 응답으로 종합한다. reference 모델은 temperature=0.6으로 다양성을, aggregator는 temperature=0.4로 일관성을 추구하며, 4개 중 1개만 성공해도 진행 가능한 내결함 설계를 갖는다. 이는 단순한 "모델 선택"을 넘어 **작업별 다중 모델 합의(consensus)** 메커니즘으로, 비교 대상 7종 중 유일하게 네이티브 MoA를 제공하는 도구이다. 또한 크론 잡 단위로 per-job model override(provider + model)를 설정할 수 있어 [56] 작업별 모델 라우팅도 실질적으로 지원한다.

---

### 6. Skills / 확장 시스템

#### Claude Code
`SKILL.md` 파일을 YAML frontmatter와 함께 정의하여 모듈식 지침을 구성한다 [2]. 필요 시에만 컨텍스트에 로드되어 토큰 효율적이다 [2]. 플러그인 시스템을 통해 훅·커맨드·스킬을 설치 가능한 단위로 번들링할 수 있다 [5].

#### Codex CLI
"Skills"을 통해 재사용 가능한 모듈식 지침 세트를 팀 간 공유할 수 있다 [8]. 그러나 Claude Code나 Cline 대비 스킬 생태계의 성숙도는 낮다 [8].

#### Cline
플러그인 시스템을 통해 커스텀 도구·라이프사이클 훅·규칙을 등록하는 명세가 존재하나 [24], Cline SDK 자체적으로 완성도 높은 독립 플러그인 생태계나 자체 마켓플레이스를 지니고 있지는 않다. 주로 VS Code 등 호스트 IDE 확장 프로그램 내의 외부 MCP Marketplace와 외부 API 서버를 연동하여 커뮤니티 도구를 설치·이용하는 간접적인 형태로 부분 지원(`◐`)한다. Skills 개념을 통해 재사용 가능한 작업 로직을 정의하며, `.cline/skills/` 디렉토리에 마크다운 형식의 스킬 정의 파일들을 배치하여 모듈식으로 지침을 동적 로딩하는 스킬 시스템을 지원한다 [23, 58].

#### OpenClaw
메타데이터 기반 스킬 시스템으로 웹 검색·캘린더·이미지 생성 등 기능을 모듈로 추가한다 [32]. 커뮤니티 스킬 생태계가 활발하나, 악성 코드 이슈로 인한 보안 우려가 있다 [32]. `SKILL.md` 매니페스트를 통해 선언적으로 스킬을 정의한다 [32].

#### oh-my-openagent
에이전트 자체가 "스킬"의 역할을 하며, 각 전문 에이전트(Prometheus, Atlas 등)가 특정 도메인의 스킬셋을 담당한다 [36]. 독립적인 스킬 마켓플레이스는 부재하며, 스킬 확장은 에이전트 정의 파일 편집을 통해 이루어진다 [36].

#### Hermes Agent
가장 독특한 자기 진화형 스킬 시스템을 보유한다 [45]. 성공적인 태스크 실행 완료 후 `skill_manager_tool.py` [54]를 통해 구조화된 하위 디렉토리를 갖춘 스킬을 자동 생성한다. 이 과정에서 `skill_usage.py` [54]의 curator 모듈이 사용 이력(use/view/patch 카운터 및 타임스탬프)을 프로세스간 원자적 락 하에 기록하여 스킬의 라이프사이클(active/stale/archived)을 관리하며, built-in 스킬의 경우 프루닝되더라도 동기화 동작 시 자동 복구(re-seeding)되지 않도록 sync를 차단한다. 또한 스킬 패치 시 들여쓰기나 서식 차이에 무관하게 매칭을 보장하는 **Fuzzy Find and Replace Engine**이 탑재되어 안전성과 유연성을 더한다. 100개 이상의 커뮤니티 도구를 플러그인 아키텍처로 지원한다 [45].

---

### 7. 도구 연결성 (Tool Connectivity)

#### Claude Code
MCP(Model Context Protocol) 서버를 `claude mcp add` 명령으로 추가하여 Jira·GitHub·Sentry·DB 등 외부 도구와 직접 통신한다 [2]. 터미널 커맨드·Git 워크플로를 네이티브로 실행한다 [2]. 브라우저 자동화의 경우, 브라우저 연동 패키지를 사용하는 `claude-in-chrome` 스킬 [48]을 통해 Chrome 브라우저 세션 탭 내비게이션, 클릭, 폼 입력, 스크린샷 캡처 및 콘솔 로그 리딩 등을 브라우저 자동화 도구로 제어할 수 있다.

#### Codex CLI
셸 커맨드를 직접 실행하는 것이 MCP 도구 호출보다 10~30배 토큰 효율적이라는 실용 지침을 채택한다 [10]. CI/CD 파이프라인·빌드 시스템과의 통합에 강점이 있다 [10]. 클라우드 샌드박스에서의 비동기 실행을 지원한다 [11].

#### Antigravity
MCP 프로토콜을 통한 외부 도구 연결을 네이티브로 지원한다 [14]. Browser Subagent를 통해 클릭·타이핑·네비게이션·스크린 캡처 등 브라우저 인터랙션을 직접 수행한다 [17]. Google AI Studio·Android·Firebase 등 Google 생태계와의 네이티브 통합이 강점이다 [17]. 이미지 생성 도구도 빌트인으로 포함한다 [17].

#### Cline
MCP 호스트로서 외부 도구·DB·API와의 연결을 표준 프로토콜로 지원한다 [24]. MCP 확장 모듈을 통해 MCP 연동을 처리하며, Puppeteer나 Playwright 등의 네이티브 브라우저 자동화 소스코드는 SDK 내부에 포함되어 있지 않다. 대신 외부 Browser Use나 브라우저 제어 MCP 서버에 명령을 위임하여 작동하므로 브라우저 자동화는 간접적인 수준으로 부분 지원(`◐`)한다. 파일시스템·Git·순차적 사고 프레임워크·문서 검색 등 개발 유틸리티를 기본 제공한다 [24]. 메시징 플랫폼의 경우, CLI의 `connect` 명령을 통해 Telegram 및 Slack 과의 연동 마법사를 제공하며 [59], allowed user ID 설정을 통한 보안 격리 및 원격 대화 턴 기반 자율 작업 수행을 지원한다.

#### OpenClaw
WhatsApp·Telegram·Slack·Discord·iMessage·Signal 등 메시징 플랫폼과의 네이티브 통합이 최대 강점이다 [26]. 로컬 파일시스템·셸 커맨드·API 호출을 직접 수행한다 [26]. 웹 브라우징은 스킬을 통해 지원한다 [32].

#### oh-my-openagent
MCP 빌트인 통합(웹 검색, 문서 검색, GitHub 코드 검색)을 제공한다 [34]. Playwright를 통한 브라우저 자동화를 지원한다 [34]. 호스트 프레임워크의 도구셋에 접근 가능하다 [34].

#### Hermes Agent
40개 이상의 빌트인 도구(웹 검색, 브라우저 자동화, 파일 관리, 터미널 액세스)를 제공한다 [45]. Telegram·Discord·Slack·WhatsApp 등 메시징 플랫폼 게이트웨이를 지원한다 [38]. 100개 이상의 커뮤니티 도구가 플러그인으로 이용 가능하다 [45].

---

### 8. Adaptive / 개인화

#### Claude Code
Thinking 토큰의 강도를 작업 복잡도에 따라 조절하는 적응형 추론을 지원한다 [4]. 사용자 선호는 `CLAUDE.md`의 수동 편집으로 반영되며, 자동 학습 기반 개인화는 제한적이다 [3].

#### Codex CLI
Suggest / Auto-Edit / Full-Auto의 3단계 승인 모드를 제공한다 [10]. 작업 복잡도에 따른 모델 계층 전환(Fast vs Standard)을 권장하지만 자동 전환은 아니다 [9].

#### Antigravity
Planning Mode를 통해 복잡한 작업에는 계획→승인→실행→검증 워크플로를 자동 적용한다 [18]. 아티팩트 기반 피드백 루프(implementation plan 피드백 요청 등)로 사용자와의 반복적 개선을 구현한다 [18]. 스케줄링 기능(Cron/일회성 타이머)을 통한 자동화된 반복 작업도 지원한다 [17].

#### Cline
Plan/Act 모드를 통해 분석 단계와 실행 단계를 명확히 분리한다 [25]. `/deep-planning`으로 복잡한 멀티파일 아키텍처 변경을 위한 심층 기획이 가능하다 [25]. 적응형 추론(모델 의존)과 Human-in-the-Loop 또는 자동 승인의 유연한 전환을 지원한다 [22].

#### oh-my-openagent
Ultrawork(`ulw`) 모드로 자동 기획→리서치→자기 수정 루프를 최소한의 개입으로 실행한다 [36]. Plan/Act에 해당하는 다단계 모드가 에이전트별로 내장되어 있다(Prometheus → Atlas 등) [36].

#### Hermes Agent
**자기 학습(Closed Learning Loop)**이 가장 큰 차별점이다 [45]. 성공적 작업 완료 후 자동으로 패턴을 추출·저장하고, 이후 유사 작업에서 이를 검색·리파인하여 점진적으로 능력이 향상된다 [45]. 사용자 프로필(`USER.md`)을 통한 개인화된 응답이 가능하다 [40].

---

### 9. 안전성 / 샌드박싱

#### Claude Code
OS 레벨 샌드박싱(macOS Seatbelt, Linux bubblewrap)으로 파일시스템·네트워크 액세스를 제한한다 [5]. 권한 모드(Suggest/Auto-Edit/Full-Auto)를 통한 세분화된 안전 제어를 제공한다 [5]. `/rewind` 명령으로 에이전트 실행을 특정 단계로 롤백할 수 있다 [5]. Git 워크트리를 활용한 서브에이전트 격리 패턴도 지원한다 [6].

#### Codex CLI
플랫폼 네이티브 샌드박싱(Seatbelt/bubblewrap/Windows 격리)을 제공한다 [10]. Suggest/Auto-Edit/Full-Auto 3단계 승인 모드를 지원한다 [10]. 클라우드 샌드박스에서의 격리 실행이 가능하다 [11]. `resume` 기능으로 세션 복원이 가능하나, 단계별 롤백은 제한적이다 [10].

#### Antigravity
권한 요청 시스템을 통해 파일 읽기/쓰기·커맨드 실행·URL 접근 등에 대한 세분화된 승인을 요구한다 [18]. 아티팩트(구현 계획, 워크스루)를 통한 사전 검토 메커니즘이 안전 장치로 기능한다 [18]. OS 레벨 샌드박싱은 Claude Code/Codex CLI 대비 덜 명시적이다.

#### Cline
모든 액션에 대한 명시적 사용자 승인을 기본으로 한다 [25]. 소스코드 상에서 세션 버전 관리 서비스 및 `checkpoint-restore.ts` [47]를 활용하여 Git commit 및 stash 기반의 버저닝/롤백 시스템을 지원한다. 특히 `checkpoint-hooks.ts` [47]에서 Git stash create 후 프라이빗 refs 네임스페이스를 관리함으로써 Git GC(가비지 컬렉션)의 소실 위협으로부터 백업을 보호하고, 사용자 로컬의 일반 stash 히스토리를 어지럽히지 않도록 영리하게 격리한다. 반면, 훅 관련 서브프로세스 구현 소스코드를 보면 단순히 로컬 호스트 상의 subprocess를 스폰하여 실행할 뿐이며 자체적인 **OS 레벨 샌드박싱(Seatbelt/Bubblewrap 등) 코드는 내장되어 있지 않다(`○`)**. 따라서 실질적인 보안 격리를 위해서는 외부 가상 환경(Docker, MicroVM 등)을 사용자가 직접 구축해 연동해야 한다. 엔터프라이즈 환경을 위한 역할 기반 접근 제어(RBAC)와 OpenTelemetry 관측성도 함께 지원한다 [25].

#### OpenClaw
자율 실행 특성상 상당한 공격 표면을 가지며, RCE 취약성과 악성 확장 패키지 이슈가 보고되었다 [32]. 보안 전문가들은 격리된 환경에서의 실행을 강력히 권장한다 [32]. `OpenShell` 등 보안 런타임 환경 활용이 추천된다 [32].

#### oh-my-openagent
호스트 프레임워크의 보안 메커니즘에 주로 의존하며, 자체적인 OS 레벨 샌드박싱은 미비하다.

#### Hermes Agent
OS 레벨 샌드박싱은 미비하나, 코드 분석에서 발견한 `checkpoint_manager.py` [63]에서 **본격적인 Git 기반 체크포인트/롤백 시스템**을 구현하고 있다. 이 시스템은:
- **Single Shared Shadow Store**: `~/.hermes/checkpoints/store/`에 단일 bare git 저장소를 운영하여 프로젝트 간 git object를 content-addressable 방식으로 공유·중복 제거한다. 프로젝트별 격리는 `refs/hermes/<hash16>` 브랜치와 `indexes/<hash16>` 인덱스 파일로 달성한다.
- **Transparent Auto-Snapshot**: 파일 변경 도구(`write_file`, `patch`, 파괴적 `terminal` 명령) 호출 시 매 대화 턴마다 자동으로 스냅샷을 생성하며, LLM에는 보이지 않는 투명 인프라로 동작한다.
- **Pre-Rollback Snapshot**: 롤백 실행 전 현재 상태를 자동 스냅샷하여 "undo의 undo"가 가능하다.
- **Auto-Pruning**: 프로젝트당 최대 20개 스냅샷, 전체 저장소 500MB 제한, 스테일/오르판 참조 정리, `git gc --prune=now` 실행 등 자동 유지보수를 수행한다.
- **Security Isolation**: `GIT_CONFIG_GLOBAL=devnull`, `GIT_CONFIG_NOSYSTEM=1`로 사용자 gitconfig를 완전 차단하여 `commit.gpgsign` 등이 pinentry GUI를 트리거하는 문제를 방지한다.

이 시스템은 Claude Code의 `/rewind`나 Cline의 Git stash 체크포인트에 비견되는 수준이므로, 체크포인트/Rollback을 **1급 지원(●)**으로 상향 분류한다. 다만 로컬 실행 특성상 OS 레벨 샌드박싱은 사용자의 자체 보안 관리가 필요하다 [38].

---

### 10. 배포 환경 / 실행 형태

| 특성 | Claude Code | Codex CLI | Antigravity | Cline | OpenClaw | oh-my-openagent | Hermes Agent |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| CLI 지원 | ● | ● | ● | ● | ● | ◐ | ● |
| IDE 통합 | ◐ VS Code 확장 | ◐ | ● VS Code 포크 | ● VS Code+JetBrains | ○ | ○ | ○ |
| 독립 데스크탑 앱 | ○ | ● Codex App | ● Antigravity 2.0 | ○ | ● | ○ | ● |
| 상시 실행(데몬) | ○ | ○ | ◐ 스케줄러 | ◐ [23, 59] | ● | ○ | ● |
| 모바일/메시징 접근 | ○ | ○ | ○ | ● [59] | ● | ○ | ● |
| 예약 작업(Cron/스케줄링) | ● [46] | ○ | ● [17] | ● [23] | ● [26] | ○ | ● [38, 56] |

---

## Cline SDK 기반 개발 시 커버리지 분석

> GOAL.md의 관심사: "Cline SDK 기반으로 Coding Assistant 개발 시 커버되는 것, SDK로도 커버가 안되는 기능들"

### Cline SDK로 커버 가능한 기능
| 기능 | SDK 지원 방식 |
|:---|:---|
| Agentic Memory | Memory Bank (`MEMORY.md` 등) + 세션 영속 |
| Context Compaction | 컨텍스트 확장 패키지의 basic/agentic 컴팩션 빌트인 |
| Agent Runtime / 하네스 | 코어 패키지 — 세션, 영속성, 도구, 자동화, 플러그인 |
| 라이프사이클 Hooks | 정책 관리·관측성·커스텀 동작 훅 지원 |
| 멀티모델 지원 | LLMs 패키지 — 프로바이더 비종속, BYOK |
| MCP 통합 | MCP 확장 모듈의 네이티브 MCP 호스트 |
| 커스텀 도구 정의 | `createTool` + Zod 스키마 |
| 플러그인 시스템 | 플러그인 설치 명세 및 로컬 훅/규칙 번들링 API 지원 |
| Plan/Act 모드 | 빌트인 모드 전환 |
| 권한 제어 | Human-in-the-Loop + 자동 승인 설정 |
| 체크포인트/Rollback | 세션 버전 관리 서비스의 Git stash 및 프라이빗 refs 네임스페이스 기반 복원 |
| 네이티브 서브에이전트 오케스트레이션 | SDK 코어 세션 계층(is_subagent DB 필드) 및 스폰 큐 지원 + portable-subagents 플러그인(squad 오케스트레이션) 제공 |
| 메시징 플랫폼 통합 | CLI의 connect 명령어로 Telegram 및 Slack 커넥터 어댑터 기본 내장 |
| 상시 실행 / 예약 작업 | SDK RPC 사이드카 런타임 및 CLI schedule 명령어 세트로 크론 기반 스케줄링 자율 실행 지원 |

### Cline SDK로 커버가 어려운 기능
| 기능 | 한계 | 대안 |
|:---|:---|:---|
| 의도 분류/라우팅 레이어 | IntentGate 같은 사전 분류 미제공 | OmO Light Edition 적용 또는 자체 구현 |
| 자기 학습/스킬 자동 생성 | 수동 스킬/에이전트 정의만 지원 (자기 학습 부재) | Hermes Agent 스타일의 학습 루프 자체 개발 |
| OS 레벨 샌드박싱 | 로컬 프로세스 실행만 지원하며 자체 샌드박스 미제공 | Docker/MicroVM 등 외부 격리 환경 활용 |
| 브라우저 자동화 | SDK 내부에 자체 웹 자동화 코드(Puppeteer/Playwright 등) 부재 | 외부 Browser Use 또는 브라우저 조작 MCP 서버 연동 |
| 플러그인/확장 마켓플레이스 | 자체적인 마켓플레이스 서버/클라이언트 생태계 부재 | 외부 MCP Marketplace 또는 호스트 IDE 확장 활용 |
| Git 워크트리 격리 | 빌트인 미지원 | 셸 스크립트로 워크트리 관리 |
| 사용자 프로필 모델링 | 단순 규칙 파일 수준 | 외부 사용자 모델링 서비스 연동 |
| 작업별 자동 모델 라우팅 | 수동 전환만 가능 | 라우팅 레이어 자체 구현 (OmO 참고) |

---

## 향후 유사 도구 개발 아이디어

### 아이디어 1: 계층형 메모리 아키텍처 통합
- **문제:** 현재 도구들은 메모리를 Markdown 파일(CLAUDE.md, MEMORY.md) 또는 단순 DB(SQLite)로 관리하며, 의미론적 검색·시간적 감쇠·중요도 가중치가 부족하다.
- **제안:** 3계층 메모리 시스템 구현 — ① 작업 메모리(현재 세션), ② 에피소딕 메모리(과거 세션 검색, 벡터 + 시간 가중), ③ 시맨틱 메모리(프로젝트 지식 그래프). OpenClaw의 하이브리드 검색과 Hermes Agent의 FTS5를 결합하여 시작점으로 삼을 수 있다.

### 아이디어 2: 자기 진화형 스킬 시스템 (Hermes Agent 방식 확장)
- **문제:** 대부분의 도구가 수동으로 스킬을 정의해야 하며, 에이전트가 스스로 학습하지 않는다.
- **제안:** Hermes Agent의 Closed Learning Loop를 확장하여, ① 작업 완료 시 성공/실패를 분석, ② 성공 패턴을 추출하여 자동으로 스킬 생성, ③ 이후 유사 작업에서 스킬을 검색·적용·리파인. 이를 Cline SDK의 플러그인 시스템 위에 구현할 수 있다.

### 아이디어 3: IntentGate 기반 지능형 라우팅
- **문제:** 대부분의 범용 에이전트가 모든 요청을 동일한 파이프라인으로 처리하여 비효율이 발생한다.
- **제안:** oh-my-openagent의 IntentGate 개념을 범용화하여, 사용자 요청을 자동 분류(리서치/구현/디버깅/리팩토링/문서화)한 뒤, 각 유형에 최적화된 에이전트·모델·도구 조합으로 라우팅. Cline SDK의 `@cline/agents` 레이어 위에 라우터 플러그인으로 구현 가능.

### 아이디어 4: 연합 에이전트 프로토콜 (Federated Agent Protocol)
- **문제:** 현재 멀티에이전트 오케스트레이션은 단일 프레임워크 내부에서만 작동하며, 이종(heterogeneous) 에이전트 간의 협업은 지원하지 않는다.
- **제안:** MCP를 확장하여 서로 다른 프레임워크의 에이전트(Claude Code 에이전트 + Hermes Agent + Cline 에이전트)가 표준 프로토콜로 협업할 수 있는 "연합 에이전트 프로토콜" 설계. 각 에이전트가 자체 강점을 유지하면서 작업을 분배받는 구조.

### 아이디어 5: 프로그레시브 신뢰 모델 (Progressive Trust Escalation)
- **문제:** 현재 권한 모드가 정적(Suggest/Auto/Full-Auto)이어서 에이전트의 실적에 따른 동적 신뢰 조절이 없다.
- **제안:** 에이전트의 누적 성공률·작업 유형·리포지토리 친숙도에 따라 자동으로 권한 수준이 에스컬레이션되는 프로그레시브 신뢰 모델. 새 프로젝트에서는 Suggest 모드로 시작하고, 성공적 작업이 쌓이면 자동으로 권한이 확대된다.

### 아이디어 6: 컨텍스트 증류(Context Distillation) 엔진
- **문제:** Context Compaction이 정보 손실을 유발하며, 장기 실행 작업에서 "컨텍스트 부패"가 발생한다.
- **제안:** 단순 요약 대신, 구조화된 "증류(distillation)" 엔진을 구현하여 ① 결정(decisions), ② 의존성(dependencies), ③ 제약 조건(constraints), ④ 다음 단계(next steps)를 별도 구조체로 추출·관리. 컴팩션 이후에도 이 구조체가 완전히 보존되어 추론 품질을 유지한다.

### 아이디어 7: Cline SDK 기반 통합 프레임워크
- **문제:** Cline SDK가 모듈형 아키텍처와 모델 비종속성을 갖추었으나, 멀티에이전트 오케스트레이션·자기 학습·메시징 통합 등이 부족하다.
- **제안:** Cline SDK를 코어 런타임으로, 다음 레이어를 플러그인으로 추가:
  - **오케스트레이션 플러그인:** oh-my-openagent 스타일의 멀티에이전트 팀 관리
  - **학습 플러그인:** Hermes Agent 스타일의 스킬 자동 생성/리파인먼트
  - **채널 플러그인:** OpenClaw 스타일의 메시징 플랫폼 브리지
  - **신뢰 플러그인:** 프로그레시브 신뢰 에스컬레이션
  - **메모리 플러그인:** 3계층 메모리 아키텍처

---

## 주요 인사이트

### 범용 vs 특수목적 에이전트의 수렴
2026년 기준, 범용 코딩 에이전트(Claude Code, Codex CLI, Antigravity, Cline)와 특수목적 에이전트(OpenClaw, oh-my-openagent, Hermes Agent) 사이의 경계가 빠르게 흐려지고 있다. 범용 에이전트들이 멀티에이전트 오케스트레이션과 메시징 통합을 추가하고, 특수목적 에이전트들이 IDE 통합과 코드 편집 기능을 강화하는 양방향 수렴이 관찰된다.

### 핵심 차별화 축
1. **모델 종속성:** Claude Code(Anthropic), Codex CLI(OpenAI), Antigravity(Google) vs Cline·OpenClaw·Hermes(모델 비종속)
2. **오케스트레이션 성숙도:** oh-my-openagent > Claude Code ≈ Antigravity > Hermes Agent > Codex CLI > Cline > OpenClaw
3. **자기 학습:** Hermes Agent >> 기타 전 도구
4. **접근성(메시징/Always-On):** OpenClaw ≈ Hermes Agent >> 기타 전 도구
5. **엔터프라이즈 준비도:** Claude Code ≈ Antigravity > Codex CLI > Cline > 기타
6. **멀티모델 합의(MoA):** Hermes Agent(유일 네이티브 MoA) >> 기타 전 도구
7. **안전성 인프라(체크포인트):** Claude Code ≈ Cline ≈ Hermes Agent > 기타 (코드 분석 결과 Hermes Agent의 shadow-git 체크포인트 시스템이 기존 평가 대비 크게 상향)

---

## References

[1] Anthropic, "Claude Code Memory System — CLAUDE.md," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/memory

[2] Anthropic, "Claude Code Skills and MCP Integration," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/skills

[3] Anthropic, "Claude Code Overview," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code

[4] Anthropic, "Claude Code Context Management," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/context

[5] Anthropic, "Claude Agent SDK — Hooks and Lifecycle Events," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/sdk

[6] Anthropic, "Claude Code Multi-Agent Orchestration," docs.anthropic.com, 2026. https://docs.anthropic.com/en/docs/claude-code/multi-agent

[7] OpenAI, "Codex CLI Configuration," github.com/openai/codex, 2026. https://github.com/openai/codex

[8] OpenAI, "Codex CLI Skills and Instructions," github.com/openai/codex, 2026. https://github.com/openai/codex/blob/main/docs/skills.md

[9] OpenAI, "Codex CLI Context and Token Management," github.com/openai/codex, 2026. https://github.com/openai/codex/blob/main/docs/context.md

[10] OpenAI, "Codex CLI Hooks, Sandboxing, and Configuration," github.com/openai/codex, 2026. https://github.com/openai/codex/blob/main/docs/configuration.md

[11] OpenAI, "Codex Desktop App — Multi-Agent Workflows," openai.com/codex, 2026. https://openai.com/index/introducing-codex/

[13] Google DeepMind, "Antigravity IDE — Knowledge Items System," antigravity.google, 2026. https://antigravity.google/docs/knowledge-items

[14] Google DeepMind, "Antigravity MCP and Artifact System," antigravity.google, 2026. https://antigravity.google/docs/artifacts

[15] Google DeepMind, "Antigravity Gemini Integration," blog.google, 2026. https://blog.google/technology/google-deepmind/antigravity-2/

[16] Google DeepMind, "Antigravity Context Management," antigravity.google, 2026. https://antigravity.google/docs/context

[17] Google DeepMind, "Antigravity 2.0 — Multi-Agent Platform," blog.google, 2026. https://blog.google/technology/google-deepmind/antigravity-2-0/

[18] Google DeepMind, "Antigravity Planning Mode and Permissions," antigravity.google, 2026. https://antigravity.google/docs/planning

[19] Cline, "Memory Bank Framework," docs.cline.bot, 2026. https://docs.cline.bot/improving-your-prompting-skills/cline-memory-bank

[20] Cline, "Cline Rules and Configuration," docs.cline.bot, 2026. https://docs.cline.bot/improving-your-prompting-skills/rules

[21] Cline, "Cline User Preferences," docs.cline.bot, 2026. https://docs.cline.bot/settings

[22] Cline, "Auto Compact and Context Management," docs.cline.bot, 2026. https://docs.cline.bot/features/auto-compact

[23] Cline, "Cline SDK Architecture," github.com/cline/cline, 2026. https://github.com/cline/cline/tree/main/sdk

[24] Cline, "Cline Plugins, Skills, and MCP Marketplace," docs.cline.bot, 2026. https://docs.cline.bot/features/plugins

[25] Cline, "Cline Multi-Model Support and Plan/Act Mode," docs.cline.bot, 2026. https://docs.cline.bot/features/plan-act

[26] OpenClaw, "OpenClaw Architecture — Gateway and Memory," openclaw.ai, 2026. https://openclaw.ai/docs/architecture

[27] OpenClaw, "OpenClaw Memory System — MEMORY.md and Daily Logs," openclaw.ai, 2026. https://openclaw.ai/docs/memory

[28] OpenClaw, "OpenClaw User Persona and Preferences," openclaw.ai, 2026. https://openclaw.ai/docs/personalization

[29] OpenClaw, "OpenClaw Context Management," openclaw.ai, 2026. https://openclaw.ai/docs/context

[30] OpenClaw, "OpenClaw Multi-Agent Workflows," openclaw.ai, 2026. https://openclaw.ai/docs/multi-agent

[31] OpenClaw, "OpenClaw Model Configuration — BYOK," openclaw.ai, 2026. https://openclaw.ai/docs/models

[32] OpenClaw, "OpenClaw Skills System and Security Considerations," openclaw.ai, 2026. https://openclaw.ai/docs/skills

[33] oh-my-openagent, "OmO Memory Integration — Hindsight," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#memory

[34] oh-my-openagent, "OmO MCP and Tool Integrations," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#tools

[35] oh-my-openagent, "OmO Context Compaction Hooks," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#compaction

[36] oh-my-openagent, "OmO Multi-Agent Orchestration — Sisyphus Architecture," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#orchestration

[37] oh-my-openagent, "OmO IntentGate and Multi-Model Routing," github.com/code-yeongyu/oh-my-openagent, 2026. https://github.com/code-yeongyu/oh-my-openagent#intentgate

[38] Nous Research, "Hermes Agent — Always-On Architecture," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent

[39] Nous Research, "Hermes Agent Memory Files — USER.md and MEMORY.md," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#memory

[40] Nous Research, "Hermes Agent User Profiling," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#personalization

[41] Nous Research, "Hermes Agent SQLite Memory and FTS5 Search," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#context

[42] Nous Research, "Hermes Agent Tool System and Plugins," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#tools

[43] Nous Research, "Hermes Agent Sub-Agent Delegation — delegate_task," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#subagents

[44] Nous Research, "Hermes Agent Model Configuration — OpenRouter," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#models

[45] Nous Research, "Hermes Agent Closed Learning Loop and Skill System," github.com/NousResearch/hermes-agent, 2026. https://github.com/NousResearch/hermes-agent#skills

[46] Anthropic, "Claude Code — Remote Triggers & Cloud Container Runtime (CCR)," scheduleRemoteAgents.ts.

[47] Cline, "Cline Core — Git-Stash & Private Ref Session Checkpointing," checkpoint-hooks.ts.

[48] Anthropic, "Claude Code — remember skill & multi-layer memory organizer," remember.ts.

[49] Anthropic, "Claude Code — autoCompact & proactive compaction settings," autoCompact.ts.

[50] Anthropic, "Claude Code — compactConversation & post-compact restorer," compact.ts.

[51] Cline, "Cline Core — basic compaction algorithm & atomic tool removal," basic-compaction.ts.

[52] Cline, "Cline Core — agentic LLM compaction & fileOps history preservation," agentic-compaction.ts.

[53] Nous Research, "Hermes Agent — Frozen system prompt memory snapshot & drift detection," memory_tool.py.

[54] Nous Research, "Hermes Agent — closed-loop skill telemetry & built-in sync suppression," skill_usage.py.

[55] Nous Research, "Hermes Agent — async background daemon delegation & task-source enqueuing," async_delegation.py.

[56] Nous Research, "Hermes Agent — Cronjob creation and local scheduling," cronjob_tools.py.

[57] Cline, "Cline Core — SubagentRunner and parallel subagent execution (use_subagents)," session/team/team-child-session-manager.ts.

[58] Cline, "Cline Core — configured-agent-tool and portable-subagents plugin," extensions/tools/team/configured-agent-tool.ts.

[59] Cline, "Cline CLI — Telegram and Slack connect adapters," runtime/host/local-runtime-host.ts.

[60] Cline, "Cline Core — basic-compaction with atomic tool_use/tool_result pair removal," extensions/context/basic-compaction.ts.

[61] Nous Research, "Hermes Agent — TrajectoryCompressor: LLM-driven protected-head/tail compaction with boundary snapping & async batch summarization," trajectory_compressor.py. 코드 직접 분석: 15,250토큰 목표, 750토큰 요약 버짓, jittered backoff 재시도, 비동기 병렬 처리(max_concurrent_requests=50).

[62] Nous Research, "Hermes Agent — Mixture-of-Agents (MoA) multi-frontier-LLM consensus tool," tools/mixture_of_agents_tool.py. 코드 직접 분석: 4개 reference 모델(claude-opus-4.6, gemini-2.5-pro, gpt-5.4-pro, deepseek-v3.2) + 1개 aggregator(claude-opus-4.6), asyncio.gather 병렬 호출, MIN_SUCCESSFUL_REFERENCES=1 내결함 설계.

[63] Nous Research, "Hermes Agent — CheckpointManager: git shadow-store auto-snapshot, pre-rollback snapshot, auto-pruning & config isolation," tools/checkpoint_manager.py. 코드 직접 분석: 단일 shared bare repo, refs/hermes/<hash16> 프로젝트 격리, 턴당 1회 자동 스냅샷, 프로젝트당 max 20 + 전체 500MB 상한, GIT_CONFIG_GLOBAL=devnull 보안 격리.

---

> **Caveat:** 특수목적 에이전트 3종(OpenClaw, oh-my-openagent, Hermes Agent)에 대한 일부 정보는 공식 문서 및 GitHub README, 커뮤니티 블로그 포스트에 기반하며, 공개 문서가 제한적인 경우 웹 검색 기반 추정이 포함될 수 있다. 각 인용의 정확성은 해당 출처의 접근 가능 여부에 따라 달라질 수 있다. **Claude Code(`claude-code-main/`), Cline(`cline-main/sdk/`), Hermes Agent(`hermes-agent-main/`)는 AGENTS.md §6에 따라 문서가 아닌 소스코드를 직접 분석하여 판정하였다.** 참조 번호 [47]~[63]은 소스 파일 기반 코드 검증 출처이다.

> **참고:** 본 문서는 Cline 열 순서를 AGENTS.md §7의 취합 친화성 규정(Claude Code → Codex CLI → Antigravity → Cline → OpenClaw → oh-my-openagent → Hermes Agent)에 맞추어 작성하였다. 단, AGENTS.md §2 원문 비교 대상에 Cline이 포함되어 있으므로 열에 추가하였다.
