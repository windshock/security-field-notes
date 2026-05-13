# Oh my secuaudit: Skills를 넘어 Evidence-first Agent Runtime으로

Date: 2026-05-13  
Category: AI / Agentic Security Audit / Architecture Notes  
Status: draft / architecture hypothesis  
Source: ChatGPT 대화 정리, "LangGraph, Skills, MCP, OpenClaw, oh-my-secuaudit 구조 논의", 2026-05-13

## 1. 한 줄 요약

`oh-my-secuaudit`는 단순한 Skills collection을 넘어, 취약점 가설을 중심으로 evidence/artifact를 생성하고 Verifier가 판정하는 **AI-native security audit runtime**으로 가는 것이 맞다.

## 2. 배경

초기 질문은 LangChain/LangGraph 교육에서 다루는 `messages`, `thread_id`, `handoff`, `swarm`, `skills`, `MCP` 개념이 `oh-my-secuaudit`에 어떻게 연결되는지였다.

논의가 진행되면서 핵심 판단은 바뀌었다.

처음에는 "LangGraph로 multi-agent를 어떻게 구성할까"가 질문이었다. 그러나 최종적으로는 다음 결론에 가까워졌다.

> 보안 감사 자동화의 본질은 agent들이 자유롭게 협업하는 것이 아니라, 취약점 가설이 어떤 evidence로 지지/반박되고, 누가 어떤 조건에서 finding을 확정할 수 있는지를 통제하는 것이다.

따라서 `oh-my-secuaudit`의 중심은 다음 순서가 되어야 한다.

```text
Hypothesis
  ↓
Evidence / Artifact
  ↓
Verification
  ↓
Finding Contract
  ↓
Architecture Mapping
  ↓
SPR / Product Requirement
```

LangGraph, Skills, MCP, OpenClaw, Claude CLI, OpenCode CLI는 이 흐름을 구현하기 위한 선택지이지, 그 자체가 핵심은 아니다.

## 3. 기존에 알려진 것

### 3.1 LangGraph의 장점

LangGraph는 다음이 필요할 때 가치가 있다.

- 상태 기반 workflow
- checkpoint / resume
- thread 단위 short-term memory
- `get_state`, `get_state_history` 기반 디버깅
- human-in-the-loop
- 동적 routing
- handoff / swarm / supervisor pattern
- long-running workflow

하지만 단순한 tool 호출이나 짧은 evidence 생성에는 과하다.

### 3.2 LangChain/LangGraph Skills 패턴의 위치

Skills 패턴은 기본적으로 다음에 적합하다.

- 단일 agent가 control을 유지한다.
- 필요한 전문 prompt/domain knowledge를 on-demand로 로드한다.
- full subagent나 graph node를 만들 정도는 아니다.
- lightweight composition이 필요하다.

즉 Skills는 "실행 가능한 보안 감사 엔진"이라기보다 **방법론/지식/프롬프트 레이어**에 가깝다.

### 3.3 Worker-as-Tool, Handoff, Swarm의 차이

논의 중 정리한 핵심 구분은 다음과 같다.

| 패턴 | 본질 | 적합한 경우 | `oh-my-secuaudit` 적용 |
|---|---|---|---|
| Worker-as-Tool | specialist를 tool처럼 호출 | 짧은 전문 작업, evidence 생성 | 초기 core에 적합 |
| Handoff | active agent를 다른 agent로 넘김 | 사용자와 다음 턴까지 대화가 이어져야 할 때 | human review, architecture 확인에 제한 사용 |
| Swarm | agent들이 handoff tool로 서로 이동 | 상담형/역할 전환형 대화 | core audit에는 부적합, 보조 영역에 제한 사용 |
| Middleware | state에 따라 prompt/tools/model을 바꾸는 interface layer | 하나의 agent 안에서 역할/권한 변경 | phase별 tool 노출 제어에 유용 |

결론적으로 보안 감사 core에서는 자유 handoff보다 Worker-as-Tool 또는 명시적 workflow가 더 안전하다.

## 4. 새롭게 볼 만한 점

### 4.1 `oh-my-secuaudit`는 Skills를 벗어났다

이번 논의의 가장 중요한 결론은 다음이다.

> `oh-my-secuaudit`는 이제 단순 Skills 패턴을 벗어났다.

단, skills가 버려지는 것은 아니다. 역할이 바뀐다.

```text
Before:
Skills = 실행 단위

After:
Skills = 방법론 / 지침 / 분석 관점 / 프롬프트 자산
```

실제 실행은 다음 레이어가 맡는다.

```text
Skills
  ↓
MCP / Tools / CLI Worker / OpenClaw Runtime
  ↓
Evidence Adapter
  ↓
Evidence Ledger
  ↓
Verifier
```

### 4.2 MCP/FastMCP는 Skills를 여러 framework에 연결하는 bridge

여러 agentic framework와 연동해야 한다면 Skills 자체보다 MCP가 유리하다.

예상 연결 대상:

- LangChain / LangGraph
- Google ADK
- AutoGen
- CrewAI
- Semantic Kernel
- Pydantic AI
- Agno
- Claude Desktop / Claude Code 계열
- OpenClaw 계열 runtime

추천 구조:

```text
skills/
  sec-audit-static/
  sec-cluster/
  security-architecture-review/

mcp_servers/
  secuaudit_static.py
  secuaudit_taint.py
  secuaudit_architecture.py

adapters/
  evidence_validator.py
  permission_policy.py
  artifact_store.py

ledger/
  evidence.jsonl
  verification.jsonl
  finding.jsonl
```

FastMCP mapping:

```text
SKILL.md
  → MCP resource 또는 prompt

실행 가능한 분석 기능
  → MCP tool

출력 검증/권한 통제
  → adapter/policy layer
```

### 4.3 OpenClaw의 위치

OpenClaw는 `oh-my-secuaudit`의 core verifier가 아니다.

가장 적절한 표현은 다음이다.

> OpenClaw는 취약점 가설을 입력받아 코드 탐색, 문서 조사, evidence 후보 생성을 수행하는 MCP-capable external specialist agent runtime이다.

즉 위치는 다음이다.

```text
OpenClaw = external specialist worker runtime
Verifier = oh-my-secuaudit 내부 판정자
```

24시간 상시 운영이 필요하면 Claude CLI를 반복 실행하는 방식보다 OpenClaw 같은 runtime/gateway 구조가 더 자연스럽다. Claude CLI나 OpenCode CLI는 batch worker로 적합하고, OpenClaw는 long-running worker runtime으로 더 적합하다.

### 4.4 취약점 분석은 개발 중간 단계에서는 candidate 생성 정도가 현실적

LangGraph로 코드 개발 workflow를 만든다고 해도, 중간 보안 분석 단계가 바로 최종 판정을 내릴 필요는 없다.

현실적인 형태:

```text
code generation / modification
  ↓
test / lint
  ↓
security review artifact generation
  ↓
candidate evidence 저장
  ↓
나중에 verifier 또는 사람 리뷰에서 판정
```

즉 처음에는 다음 정도가 적절하다.

```text
규격에 맞춰 candidate evidence를 생성한다.
나중에 verifier와 사람이 참고한다.
```

이것은 SARIF처럼 분석 결과를 표준 형식으로 남기고, 별도 리뷰/검증 단계에서 해석하는 방식과 유사하다.

## 5. 공격자 관점

이 노트는 직접적인 공격 절차를 다루지는 않는다. 다만 자동화된 보안 감사 시스템이 실패할 때 공격자에게 유리해지는 지점은 있다.

### 5.1 agent 판단 오염

agent들이 raw message를 공유하면 다음 문제가 생긴다.

- 한 agent의 추정이 다른 agent에게 사실처럼 전파됨
- prompt injection이 shared message space를 통해 확산됨
- 미검증 hypothesis가 report writer에게 전달되어 finding처럼 포장됨

따라서 agent 간 공유는 raw messages가 아니라 다음으로 제한해야 한다.

```text
evidence_id
artifact_ref
hypothesis_id
verification_id
```

### 5.2 false confidence

LLM worker가 "confirmed"라고 말하는 순간 시스템이 그것을 받아들이면 위험하다.

정책:

```text
Specialist worker: evidence candidate 생성만 가능
Supervisor: 가설 생성/분배만 가능
Verifier: confirmed / not-confirmed / refuted 판정 가능
Writer: verified finding만 보고서화 가능
```

### 5.3 long-running runtime의 권한 위험

OpenClaw 같은 24시간 runtime은 agent가 실제 파일, tool, credentials, external service에 접근할 수 있다.

따라서 다음 제한이 필요하다.

- 전용 VM/container에서 실행
- 읽기 전용 code snapshot 기본값
- artifact/ledger 디렉터리에만 쓰기 허용
- patch/commit/push는 human approval 뒤에만 허용
- MCP tool input/output schema 검증
- destructive/runtime probe는 별도 approval gate 필요

## 6. 방어자 관점

### 6.1 Evidence-first 구조의 가치

보안 리뷰에서 가장 중요한 것은 "누가 그럴듯하게 말했는가"가 아니라 "어떤 evidence가 어떤 hypothesis를 얼마나 지지/반박하는가"이다.

따라서 `oh-my-secuaudit`는 다음을 기본 구조로 가져가야 한다.

```text
Hypothesis
EvidencePacket
ArtifactRef
VerificationResult
FindingContract
ArchitectureMapping
SPR
```

### 6.2 Git-like memory ledger

메모리 오염과 되돌릴 수 없는 변경을 막으려면 shared memory를 채팅방처럼 만들면 안 된다.

권장 구조:

```text
append-only
versioned
branchable
merge-controlled
revertible
blameable
schema-validated
verifier-gated
```

즉 agent shared memory는 **Git-like Audit Memory Ledger**로 보는 것이 적절하다.

### 6.3 messages는 trace, evidence는 truth

LangGraph의 `MessagesState`, `thread_id`, `checkpoint`는 디버깅과 실행 상태 관리에는 유용하다.

하지만 보안 감사의 source of truth로 삼으면 안 된다.

```text
messages
  = 실행 trace / 디버깅용

evidence.jsonl
  = 감사 증거

verification.jsonl
  = 판정 기록

finding.jsonl
  = 보고 가능한 finding
```

## 7. 탐지 포인트

이 노트의 직접 탐지 대상은 runtime/architecture misuse이다.

- Data source:
  - agent execution logs
  - MCP tool invocation logs
  - evidence ledger
  - artifact write logs
  - Git commit / PR events
  - CI security scan results

- Observable:
  - Verifier를 거치지 않은 `confirmed` finding 생성
  - writer가 candidate evidence를 final report에 사용
  - worker가 허용되지 않은 tool 또는 handoff 대상 호출
  - MCP tool output schema validation 실패
  - runtime worker가 codebase 외부 path에 write 시도
  - 동일 hypothesis에 대한 반복 실패/loop

- Suspicious condition:
  - `final_verdict` 필드를 specialist worker가 직접 채움
  - `evidence_id` 없이 finding 생성
  - `artifact_ref`가 없는 strong evidence
  - `runtime_probe`가 approval 없이 실행

- False positive:
  - 초기 PoC 단계에서 schema가 느슨한 경우
  - 사람이 수동으로 draft note를 작성하는 경우
  - 테스트용 mock evidence

## 8. 실험 계획

### 8.1 MVP: FastMCP 기반 static evidence producer

- [ ] `sec-audit-static` skill을 FastMCP server로 감싼다.
- [ ] `run_static_audit(audit_id, hypothesis_id, artifact_ref)` tool을 만든다.
- [ ] 반환값은 `EvidencePacket` schema로 제한한다.
- [ ] `confirmed` 판정 필드는 금지한다.
- [ ] output validator가 실패하면 ledger에 쓰지 않는다.

### 8.2 Supervisor → MCP specialist team 흐름

- [ ] Supervisor가 `HYP-001` 취약점 가설을 생성한다.
- [ ] static team, taint team, guard team에 같은 hypothesis를 전달한다.
- [ ] 각 team은 evidence candidate와 gaps만 반환한다.
- [ ] Verifier가 evidence ledger를 읽고 판정한다.

### 8.3 OpenClaw runtime 실험

- [ ] OpenClaw를 `external specialist worker runtime`으로 실행한다.
- [ ] `static-review`, `taint-review`, `architecture-review` agent를 분리한다.
- [ ] 각 agent는 동일한 JSON evidence format으로만 출력하게 한다.
- [ ] OpenClaw 출력과 Claude CLI/OpenCode CLI 출력의 안정성을 비교한다.

### 8.4 개발 workflow 중간 보안 산출물 생성

- [ ] 코드 생성/수정 후 security candidate artifact를 자동 생성한다.
- [ ] `candidate` 상태로 저장하고 즉시 blocking하지 않는다.
- [ ] PR 또는 별도 audit run에서 verifier가 candidate를 evidence로 승격할지 판단한다.

## 9. 결론 강도

- [x] 확인된 사실: `oh-my-secuaudit`는 단순 Skills 패턴만으로는 부족하다.
- [x] 확인된 사실: Skills는 방법론/지식 레이어로 남기는 것이 좋다.
- [x] 확인된 사실: 실행 가능한 기능은 Tool/MCP/FastMCP/CLI worker로 분리하는 것이 자연스럽다.
- [x] 확인된 사실: Verifier만 최종 finding 판정 권한을 가져야 한다.
- [ ] 실험실에서만 검증: FastMCP 기반 specialist team 실제 구현
- [ ] 실험실에서만 검증: OpenClaw 24/7 runtime 운영 안정성
- [ ] 추가 검증 필요: LangGraph가 필요한 최소 시점과 직접 Python orchestrator의 경계

## 10. 참고 자료

이번 노트는 외부 1차 자료 요약이 아니라 ChatGPT와의 설계 토론을 정리한 internal field note이다. 이후 실제 구현 시 다음 범주의 공식 문서를 별도 reference로 등록한다.

- LangGraph persistence / checkpoint / handoff / swarm 문서
- LangChain Skills / Subagents / Handoffs / MCP adapter 문서
- FastMCP 공식 문서
- OpenClaw CLI / agent runtime / MCP 문서
- Claude Code headless / MCP 문서
- OpenCode CLI / MCP 문서

## 11. 최종 구조 초안

현재 가장 좋은 구조는 다음과 같다.

```text
oh-my-secuaudit/
├── skills/
│   ├── sec-audit-static/
│   ├── sec-cluster/
│   └── security-architecture-review/
│
├── mcp_servers/
│   ├── static_team.py
│   ├── taint_team.py
│   ├── guard_team.py
│   └── architecture_team.py
│
├── adapters/
│   ├── evidence_validator.py
│   ├── permission_policy.py
│   └── artifact_store.py
│
├── ledger/
│   ├── evidence.jsonl
│   ├── verification.jsonl
│   └── finding.jsonl
│
├── runtime/
│   ├── openclaw_workers/
│   ├── cli_workers/
│   └── langgraph_supervisor/
│
└── reports/
```

역할 분리:

```text
Skills
= 방법론 / 분석 관점 / 프롬프트 자산

MCP/FastMCP
= framework 간 연결 가능한 실행 인터페이스

OpenClaw / Claude CLI / OpenCode CLI
= specialist worker runtime 또는 batch worker

LangGraph
= 필요할 때 workflow orchestration / checkpoint / human gate

Evidence Ledger
= 보안 감사 truth store

Verifier
= 최종 판정 권한자
```

## 12. 다음 판단

지금 단계에서 가장 중요한 다음 작업은 LangGraph multi-agent 구현이 아니다.

우선순위는 다음이다.

1. `EvidencePacket` schema 정의
2. `VerificationResult` schema 정의
3. `FindingContract` schema 정의
4. `sec-audit-static` skill을 FastMCP tool로 감싸는 PoC
5. OpenClaw 또는 Claude CLI를 evidence producer로 실행하는 PoC
6. Verifier가 candidate evidence를 읽고 `confirmed / not-confirmed / refuted`를 판정하는 최소 구현

최종 결론:

> `oh-my-secuaudit`는 Skills-first로 시작했지만, 이제 Evidence-first / Verifier-first / MCP-compatible 구조로 진화해야 한다.
