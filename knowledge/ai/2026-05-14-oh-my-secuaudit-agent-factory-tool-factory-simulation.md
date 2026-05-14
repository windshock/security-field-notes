# Oh my secuaudit: Agent Factory, Tool Factory, Zero-Config Binding, and Simulation Runs

Date: 2026-05-14  
Category: AI / Agentic Security Audit / Implementation Architecture  
Status: draft / architecture hypothesis / experiment-needed  
Source: ChatGPT 대화 정리, "agent-service-toolkit, production-ready FastAPI LangGraph template, tool factory, Agent-Tool Matching, simulation thread design", 2026-05-14

## 1. 한 줄 요약

`oh-my-secuaudit`를 제품화하려면 LangGraph agent 자체보다 **Agent Factory, Tool Factory, MCP 기반 Agent-Tool Matching, streaming service, simulation run 관리**가 더 중요한 구현 축이 된다.

## 2. 배경

이 노트는 이전 노트 `knowledge/ai/2026-05-13-oh-my-secuaudit-agent-runtime-architecture.md`의 후속 정리다.

이전 결론은 다음이었다.

```text
Skills = 방법론 / 분석 관점 / 프롬프트 자산
MCP/FastMCP = framework 간 연결 가능한 실행 인터페이스
OpenClaw / Claude CLI / OpenCode CLI = specialist worker runtime 또는 batch worker
LangGraph = 필요할 때 workflow orchestration / checkpoint / human gate
Evidence Ledger = 보안 감사 truth store
Verifier = 최종 판정 권한자
```

이번 논의에서는 그 위에 실제 agent service를 만들 때 필요한 구성 요소를 더 구체화했다.

주요 질문은 다음이었다.

- agent가 많아지면 어떻게 관리할 것인가?
- tool을 `tools.py` 하나에 모아도 되는가?
- `skills/*.md`에 이미 있는 설명을 활용해 agent/tool을 자동 매칭할 수 있는가?
- LangGraph 기반 제품화 템플릿을 참고하면 어디까지 커버되는가?
- 실제 자산에서 실험할 수 없다면 simulation lab을 어떻게 설계할 것인가?
- `thread_id`는 시나리오마다 어떻게 부여해야 하는가?

## 3. 기존에 알려진 것

### 3.1 agent-service-toolkit이 보여주는 패턴

`JoshuaC215/agent-service-toolkit`은 LangGraph agent를 FastAPI service, client, Streamlit UI, schema와 함께 운영하는 예제 템플릿이다.

참고할 점은 다음이다.

- `Agent Registry`: 여러 agent를 `agent_id`로 관리
- `LazyLoadingAgent`: MCP client, database, external resource처럼 async initialization이 필요한 agent를 service startup에서 load
- `create_agent` 기반 MCP agent pattern
- `create_supervisor` 기반 supervisor multi-agent pattern
- `/invoke`, `/stream`, `/history` 같은 service endpoint
- `thread_id`, `user_id`, `RunnableConfig` 기반 state/session 관리
- LangGraph streaming event를 SSE로 전달하는 구조

`oh-my-secuaudit` 관점에서는 이 프로젝트가 **agent 개발 패턴과 service wrapper**를 보여준다.

### 3.2 fastapi-langgraph-agent-production-ready-template이 보여주는 패턴

`wassim249/fastapi-langgraph-agent-production-ready-template`은 agent backend 제품화에 필요한 운영 요소를 더 많이 포함한다.

참고할 점은 다음이다.

- FastAPI + LangGraph backend skeleton
- JWT authentication / session management
- rate limiting
- structured logging
- Langfuse tracing
- Prometheus / Grafana metrics
- PostgreSQL + pgvector
- Redis/Valkey cache
- Alembic migration
- LLM retry / fallback / timeout budget
- mem0 기반 long-term memory
- HTTP/SSE interface

이 프로젝트는 `oh-my-secuaudit`의 **제품화 직전 service runtime skeleton**으로 참고할 만하다.

다만 기본 agent graph는 `chat → tool_call → chat` 구조에 가까우므로, `oh-my-secuaudit`의 보안 감사 core를 그대로 대체하지는 않는다.

### 3.3 두 프로젝트의 조합

두 프로젝트를 함께 보면 다음 영역이 어느 정도 커버된다.

| 영역 | 참고 프로젝트 | `oh-my-secuaudit` 적용 |
|---|---|---|
| agent registry / factory | agent-service-toolkit | specialist agent 관리 |
| lazy agent loading | agent-service-toolkit | MCP/OpenClaw/GitHub 도구 초기화 |
| streaming API | agent-service-toolkit | audit 진행 상황 실시간 표시 |
| FastAPI production backend | fastapi-langgraph template | auth, rate limit, logging, session |
| observability | fastapi-langgraph template | Langfuse, Prometheus, structured logs |
| persistence | fastapi-langgraph template | checkpoint, session, memory |
| domain core | 직접 설계 | Evidence Ledger, Verifier, Finding Contract |

결론:

```text
agent-service-toolkit = agent 개발 패턴 참고용
fastapi-langgraph-agent-production-ready-template = production backend 참고용
oh-my-secuaudit = 보안 감사 domain engine
```

## 4. 새롭게 볼 만한 점

### 4.1 Agent Factory

agent가 적을 때는 직접 agent를 만들면 된다.

하지만 agent가 많아지고, 각 agent마다 prompt, toolset, output schema, memory, 권한이 달라지면 `Agent Factory`가 필요하다.

```text
Agent Factory
= agent role, prompt, model, toolset, output contract, policy를 받아 agent graph를 생성하는 계층
```

`oh-my-secuaudit`에서 예상되는 agent role:

```text
hypothesis-planner-agent
static-review-agent
taint-analysis-agent
guard-detection-agent
architecture-review-agent
verifier-agent
reporter-agent
simulation-attacker-agent
simulation-defender-agent
```

agent를 나누는 기준은 이름이나 persona가 아니라 **contract 차이**다.

agent를 분리할 근거:

- 다른 system prompt가 필요함
- 다른 toolset이 필요함
- 다른 output schema가 필요함
- 다른 memory/context가 필요함
- 다른 권한 정책이 필요함
- 다른 human approval gate가 필요함
- 다른 evaluation metric이 필요함

### 4.2 Tool Factory

단순 예제에서는 `tools.py` 하나에 모든 tool을 모아도 된다.

하지만 `oh-my-secuaudit`에서는 tool이 많아지고 agent별 권한이 달라지므로, 단일 `tools.py` 방식은 곧 한계가 온다.

```text
Tool Factory
= agent role, audit mode, policy profile, backend, output contract에 따라 toolset을 조립하는 계층
```

예상 구조:

```text
src/
├── agents/
│   ├── factory.py
│   ├── registry.py
│   ├── static_review.py
│   ├── taint_analysis.py
│   ├── architecture_review.py
│   └── verifier.py
│
├── tools/
│   ├── factory.py
│   ├── registry.py
│   ├── specs.py
│   ├── static_tools.py
│   ├── taint_tools.py
│   ├── guard_tools.py
│   ├── mcp_tools.py
│   ├── openclaw_tools.py
│   └── policy.py
│
├── schemas/
│   ├── evidence.py
│   ├── verification.py
│   └── finding.py
│
└── service/
    └── service.py
```

Tool Factory가 해야 하는 일:

- agent 역할에 맞는 tool 선택
- read/write 권한 제한
- destructive/runtime tool 차단
- human approval 필요 여부 표시
- output schema validator 연결
- evidence ledger writer 연결
- MCP/OpenClaw/CLI worker tool wrapping
- timeout/retry/cost budget 설정

### 4.3 Skills md를 source로 하는 tool manifest 변환

기존 `skills/*.md`에는 이미 다음 정보가 있다.

- skill 목적
- agent 역할 후보
- 분석 절차
- 사용할 도구
- 출력해야 할 산출물
- 주의사항
- 금지해야 할 판단

따라서 AI에게 `skill.md`를 읽혀서 agent별 tool 후보를 추천하게 할 수 있다.

권장 흐름:

```text
skills/*.md
  ↓
AI converter
  ↓
tool manifest YAML 생성
  ↓
policy validator
  ↓
Tool Factory 등록
  ↓
Agent Factory에서 toolset 주입
```

중요한 원칙:

```text
AI가 바로 production tool code를 만드는 것이 아니라,
검토 가능한 tool manifest를 먼저 만든다.
```

예시 manifest:

```yaml
skill: sec-audit-static
agent_role: static-review-agent
description: >
  Produces source/path/sink evidence candidates from source code.

tools:
  - name: extract_source_candidates
    kind: local-python
    purpose: Identify user-controlled input candidates.
    input_schema: SourceCandidateInput
    output_schema: EvidencePacket
    writes:
      - evidence.candidate
      - artifact.code_slice
    cannot_write:
      - verification_result
      - confirmed_finding
    risk_level: low
    requires_human_approval: false

  - name: run_semgrep_scan
    kind: external-cli
    purpose: Run static pattern scan and convert results to evidence candidates.
    input_schema: StaticScanInput
    output_schema: EvidencePacketList
    writes:
      - evidence.candidate
      - artifact.scan_result
    cannot_write:
      - confirmed_finding
    risk_level: medium
    requires_human_approval: false
```

### 4.4 AI 기반 Agent-Tool Matching

정확한 문구는 다음과 같다.

> AGENT_REGISTRY의 각 에이전트 description을 AI(LLM/Embedding)가 분석하여, MCP에 등록된 도구 목록 중 해당 에이전트에 적합한 도구들을 자동으로 추천·연동할 수 있다.

예:

```text
description="이미지 검색/분석 멀티모달 RAG"
  → 이미지 생성, OCR, 차트 분석 도구 추천

description="한국은행 보고서 기반 기본 RAG"
  → 경제 지표 API, 환율 조회 도구 추천
```

이 구조를 `oh-my-secuaudit`에 맞게 바꾸면 다음이다.

```text
AGENT_REGISTRY
  - agent name
  - description
  - role
  - allowed outputs
  - forbidden outputs
  - risk budget

MCP_TOOL_REGISTRY
  - tool name
  - description
  - input schema
  - output schema
  - tags
  - risk level
  - required approval

Agent-Tool Matcher
  - LLM/Embedding 기반 후보 매칭
  - schema compatibility check
  - policy filter
  - binding manifest 생성

Tool Factory
  - binding manifest 기준으로 실제 toolset 생성
```

보안 감사에서는 단순 `Zero-Config Tool Binding`보다 다음 표현이 더 안전하다.

> Controlled Zero-Config Tool Binding

즉 자동 추천은 하되, 실제 binding은 policy layer를 통과해야 한다.

금지 예시:

```text
static-review-agent
  ❌ write_verification_result
  ❌ confirmed_finding_writer
  ❌ runtime_probe_without_approval
```

허용 예시:

```text
static-review-agent
  ✅ read_code
  ✅ run_semgrep_scan
  ✅ extract_source_candidates
  ✅ emit_evidence_candidate
```

### 4.5 Streaming은 UX가 아니라 observability

agent service에서 streaming은 답변을 빨리 읽게 해주는 기능이기도 하지만, `oh-my-secuaudit`에서는 더 중요하다.

보안 감사 workflow는 오래 걸린다.

따라서 사용자는 다음을 실시간으로 볼 수 있어야 한다.

```text
Static review started
Semgrep scan completed
Taint path candidate found
Guard evidence missing
Evidence EVD-001 created
Verifier: not-confirmed because reachability incomplete
Architecture mapping candidate generated
```

권장 event type:

```json
{"type": "status", "message": "Static review started"}
{"type": "tool_start", "tool": "run_semgrep", "hypothesis_id": "HYP-001"}
{"type": "artifact", "artifact_ref": "artifacts/HYP-001/semgrep.json"}
{"type": "evidence", "evidence_id": "EVD-001", "strength": "partial"}
{"type": "gap", "message": "Sink argument unresolved"}
{"type": "verifier", "verdict": "not-confirmed", "reason": "Reachability incomplete"}
{"type": "done"}
```

결론:

```text
streaming = token streaming + audit timeline + tool observability + evidence generation trace
```

### 4.6 agent가 많아지는 경우: character agent와 pentest node

작가가 캐릭터별 agent를 따로 만드는 이유는 각 캐릭터의 말투, 기억, 목표, 금지사항이 다르기 때문이다.

보안에서도 유사한 구조가 가능하다.

Pentest를 graph로 보면 각 node가 다른 specialist agent가 될 수 있다.

```text
recon-agent
web-hacker-agent
server-hacker-agent
cloud-agent
source-code-auditor-agent
taint-agent
architecture-agent
verifier-agent
```

다만 보안에서는 character보다 다음이 중요하다.

```text
toolset
output schema
권한
risk policy
evidence contract
human approval condition
```

따라서 agent를 많이 만드는 기준은 persona가 아니라 contract 차이다.

### 4.7 실제 자산에서 실험할 수 없을 때: simulation lab

실제 자산에서 취약점 실험을 할 수 없다면 simulation lab이 핵심이 된다.

목표는 실제 침투가 아니라 다음이다.

- agent가 올바른 evidence를 생성하는가?
- candidate와 confirmed를 구분하는가?
- guard/contradiction evidence를 놓치지 않는가?
- Verifier가 not-confirmed/refuted를 낼 수 있는가?
- finding이 architecture flow와 연결되는가?
- reporter가 미검증 evidence를 finding처럼 쓰지 않는가?

권장 구조:

```text
sim-labs/
├── toy-java-sqli/
│   ├── vulnerable/
│   ├── fixed/
│   └── expected_findings.yaml
│
├── toy-node-ssrf/
│   ├── vulnerable/
│   ├── guarded/
│   └── expected_findings.yaml
│
├── toy-upload-rce/
│   ├── vulnerable/
│   ├── false-positive/
│   └── expected_findings.yaml
│
└── enterprise-mini/
    ├── web/
    ├── api/
    ├── db/
    ├── object-storage/
    └── dfd.yaml
```

expected finding 예시:

```yaml
hypothesis_id: HYP-SQLI-001
expected:
  verdict: confirmed
  required_evidence:
    - evidence.source
    - evidence.sink
    - evidence.path
  required_negative_checks:
    - no_effective_sanitizer
  architecture_mapping:
    impacted_flow: search-api-to-database
    trust_boundary: public-web-to-internal-db
```

### 4.8 thread_id는 scenario run 단위로 부여

시뮬레이션에서 `thread_id`는 시나리오 자체가 아니라 **시나리오 실행 run**에 부여하는 것이 좋다.

권장 ID 체계:

```text
scenario_id = 시나리오의 정체성
thread_id   = 그 시나리오를 실행한 한 번의 LangGraph 세션
audit_id    = 전체 감사 실행 묶음
```

예:

```text
scenario_id: SIM-SSRF-METADATA-001
audit_id:    AUD-20260514-001
thread_id:   AUD-20260514-001::SIM-SSRF-METADATA-001::run-001
```

같은 시나리오를 다른 조건으로 다시 돌리면:

```text
thread_id: AUD-20260514-001::SIM-SSRF-METADATA-001::run-002
```

원칙:

```text
새로운 실험 run이면 새 thread_id
중단된 분석을 이어가면 같은 thread_id
같은 시나리오에서 agent/tool/prompt 변경 효과를 비교하면 다른 thread_id
Verifier 결과만 재계산하면 같은 evidence set, 새 verification_id
새로운 공격 경로를 추가하면 같은 scenario_id 안의 새 hypothesis_id
```

## 5. 공격자 관점

이 노트는 실제 공격 절차가 아니라 agentic security audit architecture를 다룬다. 다만 설계 실패 시 공격자 또는 오탐/오판이 유리해지는 지점이 있다.

### 5.1 잘못된 tool binding

Agent-Tool Matching이 description 기반으로만 동작하면, 잘못된 tool이 agent에게 붙을 수 있다.

예:

```text
static-review-agent에게 write_verification_result tool이 붙음
reporter-agent에게 raw candidate evidence read 권한이 붙음
runtime probe tool이 approval 없이 붙음
```

방어:

- embedding/LLM 추천은 후보로만 사용
- Tool Policy Layer 필수
- allowed writes / forbidden writes 검사
- risk level / approval 검사
- schema compatibility 검사

### 5.2 simulation contamination

같은 `thread_id`를 여러 실험에 재사용하면 이전 run의 판단이 다음 run에 영향을 줄 수 있다.

방어:

- scenario_id와 thread_id 분리
- run마다 새 thread_id 사용
- resume할 때만 같은 thread_id 사용
- evidence ledger와 message trace 분리

### 5.3 agent over-permission

agent를 많이 만들수록 권한 관리가 어려워진다.

방어:

- agent별 role contract 정의
- Tool Factory에서 toolset 제한
- Verifier 전용 tool은 다른 agent에 노출 금지
- writer는 verified finding만 접근

## 6. 방어자 관점

### 6.1 agent service 제품화의 핵심

`oh-my-secuaudit` 제품화는 LangGraph node를 많이 만드는 문제가 아니다.

핵심은 다음이다.

```text
Agent Registry
Tool Registry
Agent-Tool Matching
Tool Factory
Policy Filter
Streaming Observability
Evidence Ledger
Verifier
Simulation Lab
```

### 6.2 messages는 trace, simulation run은 실험 단위

LangGraph `messages`와 `thread_id`는 실행 추적에 유용하다.

하지만 security finding의 source of truth는 아니다.

```text
messages
  = trace / debug / stream context

thread_id
  = one scenario run execution session

evidence ledger
  = security audit truth store

verification result
  = 판정 기록
```

### 6.3 실제 자산에는 candidate mode로만 적용

실제 자산에서 바로 공격 실험을 하지 않는다.

운영 환경 적용 방식:

```text
real asset
  ↓
read-only analysis
  ↓
evidence candidate 생성
  ↓
human review / verifier gate
  ↓
confirmed finding or not-confirmed
```

simulation lab 적용 방식:

```text
simulated vulnerable asset
  ↓
agent/tool/verifier 실험
  ↓
expected finding과 비교
  ↓
regression test로 저장
```

## 7. 탐지 포인트

- Data source:
  - agent registry change log
  - tool binding manifest
  - MCP tool registry
  - tool invocation log
  - streaming audit events
  - evidence ledger
  - simulation run metadata

- Observable:
  - forbidden tool이 agent에게 binding됨
  - worker agent가 `confirmed_finding` write 시도
  - same scenario에 같은 thread_id를 반복 사용
  - runtime probe가 approval 없이 실행
  - output schema validation 실패 반복
  - reporter가 unverified evidence를 사용

- Suspicious condition:
  - `final_verdict`를 verifier가 아닌 agent가 생성
  - `artifact_ref` 없는 strong evidence
  - `expected_findings.yaml`과 verifier output이 반복적으로 불일치
  - tool matching score만으로 high-risk tool이 자동 binding

- False positive:
  - PoC 단계에서 intentionally permissive policy를 사용하는 경우
  - simulation lab에서 failure case를 일부러 만든 경우
  - manual draft note 작성 중인 경우

## 8. 실험 계획

### 8.1 Agent Registry PoC

- [ ] `AGENT_REGISTRY.yaml` 작성
- [ ] static-review, taint-analysis, guard-detection, verifier, reporter agent 정의
- [ ] 각 agent에 description, allowed outputs, forbidden outputs, risk budget 추가

### 8.2 MCP Tool Registry PoC

- [ ] `MCP_TOOL_REGISTRY.yaml` 작성
- [ ] tool name, description, tags, input_schema, output_schema, risk_level 정의
- [ ] FastMCP tool과 local-python tool을 같은 registry abstraction으로 표현

### 8.3 Agent-Tool Matching PoC

- [ ] agent description과 tool description을 embedding으로 매칭
- [ ] LLM으로 top-k 추천 이유 생성
- [ ] policy filter로 forbidden tools 제거
- [ ] `tool_binding.yaml` 생성
- [ ] 사람이 review 후 Tool Factory에 적용

### 8.4 Tool Factory PoC

- [ ] `create_toolset(agent_role, policy_profile, backends)` 구현
- [ ] static-review-agent에는 evidence candidate tool만 부여
- [ ] verifier-agent에만 verification result writer 부여
- [ ] reporter-agent에는 verified finding reader만 부여

### 8.5 Streaming Audit Event PoC

- [ ] `/audit/stream` SSE endpoint 설계
- [ ] token, status, tool_start, artifact, evidence, verifier, done event 구분
- [ ] UI에서 audit timeline으로 표시

### 8.6 Simulation Lab PoC

- [ ] `toy-java-sqli` lab 생성
- [ ] vulnerable / fixed / guarded 세 variant 생성
- [ ] `expected_findings.yaml` 작성
- [ ] scenario_id / audit_id / thread_id 규칙 적용
- [ ] verifier output과 expected finding 비교

## 9. 결론 강도

- [x] 확인된 사실: agent가 많아지고 tool 권한이 달라지면 Tool Factory가 필요하다.
- [x] 확인된 사실: `skills/*.md`는 Agent/Tool manifest 생성의 source material로 활용 가능하다.
- [x] 확인된 사실: Zero-Config Tool Binding은 보안 도메인에서는 Controlled Zero-Config Tool Binding으로 제한해야 한다.
- [x] 확인된 사실: streaming은 UX뿐 아니라 audit observability 계층이다.
- [x] 확인된 사실: 실제 자산에서 실험할 수 없으면 simulation lab이 verifier 개발의 핵심이 된다.
- [x] 확인된 사실: thread_id는 scenario 자체가 아니라 scenario run 단위로 부여하는 것이 안전하다.
- [ ] 실험실에서만 검증: Agent-Tool Matching 자동화 정확도
- [ ] 실험실에서만 검증: simulation lab 기반 verifier regression test
- [ ] 추가 검증 필요: OpenClaw runtime과 FastAPI/LangGraph service의 운영 경계

## 10. 참고 자료

- JoshuaC215, `agent-service-toolkit`, GitHub, retrieved 2026-05-14
- wassim249, `fastapi-langgraph-agent-production-ready-template`, GitHub, retrieved 2026-05-14
- ChatGPT conversation synthesis, 2026-05-14

## 11. 다음 설계 결론

`oh-my-secuaudit`의 다음 구현 우선순위는 다음이다.

```text
1. EvidencePacket / VerificationResult / FindingContract schema
2. AGENT_REGISTRY.yaml
3. MCP_TOOL_REGISTRY.yaml
4. Agent-Tool Matching prototype
5. Tool Binding Manifest
6. Tool Factory
7. Agent Factory
8. /audit/stream event model
9. simulation lab + expected findings
10. verifier regression test
```

최종 문장:

> `oh-my-secuaudit`는 LangGraph agent를 많이 만드는 프로젝트가 아니라, agent와 tool을 안전하게 매칭하고, simulation에서 검증하며, evidence ledger와 verifier로 finding을 통제하는 보안 감사 runtime이어야 한다.
