# LLM Context & Master Index

이 파일은 AI agent가 저장소의 구조와 상태를 빠르게 파악하고 다음 작업을 수행하기 위한 **Master Map**입니다.

## Repository Purpose
보안 리서치 과정에서 발생하는 파편화된 정보를 **증류(Distillation)**하고 **연결(Linking)**하여, 검증 가능한 탐지 가설과 실험으로 발전시키는 지식 엔진입니다.

## Value Criteria

우선순위 판단은 [`VALUE_CRITERIA.md`](VALUE_CRITERIA.md)와 [`CONVERSION_STATUS.md`](CONVERSION_STATUS.md)를 함께 참고합니다.

`AGENTS.md`의 concept page 생성 임계치(노트 3개 이상, 또는 노트 2개 + detection/experiment 1개 이상)는 **준비도(readiness)** 기준입니다. 최종 우선순위는 다음 질문으로 판단합니다.

```text
발견을 구조로 바꾸고, 분석을 방법론으로 남기며, 기술적 사실을 지속 가능한 보안 개선으로 연결하는가?
```

단, 이미 블로그화되었거나 도구화된 주제는 추가 field note의 한계효용이 낮습니다. 따라서 AI agent는 새 concept page, detection, experiment, blog seed를 제안할 때 다음 두 단계를 모두 확인합니다.

1. `VALUE_CRITERIA.md`: 이 주제가 본질적으로 가치 있는가?
2. `CONVERSION_STATUS.md`: 이미 blog / tool / checklist / experiment로 전환되었는가, 아니면 아직 field note에 머물러 있는가?

## Directory Schema
- `knowledge/`: 주제별 리서치 노트 및 정리 자료
  - `knowledge/concepts/`: **[중요]** 핵심 기술/개념에 대한 영구적인 백과사전 레이어
- `detections/`: 탐지 가설 (`Status: hypothesis` 등) 및 룰 초안
- `experiments/`: 가설 검증을 위한 실험 계획 및 결과 로그
- `references/`: 외부 자료의 메타데이터 및 인덱스 (`README.md` 참고)
- `templates/`: 표준화된 노트 작성을 위한 템플릿

## Core Knowledge Map (Concept Hubs)
현재 저장소의 핵심 리서치 주제와 연결된 Concept 페이지들입니다.

### 1. Windows Agentless Telemetry
- [ETW](knowledge/concepts/etw.md): 핵심 텔레메트리 소스
- [PLA](knowledge/concepts/pla.md): 원격 수집 및 제어 메커니즘
- [DCOM](knowledge/concepts/dcom.md): 원격 제어를 위한 통신 프로토콜

### 2. Linux Integrity, Rootkit, and EDR
- [Page Cache Integrity](detections/linux/page-cache-integrity-divergence.md): (Concept 페이지 승격 예정)
- [OrBit / Medusa rootkit reuse field note](knowledge/linux/2026-05-15-orbit-medusa-rootkit-reuse-and-evolution.md): 공개/반공개 Linux rootkit 코드베이스 재사용, Lineage A/B, build option 기반 기능 변화, rootkit primitive 성숙과 platform 이동
- [OrBit / Medusa invariant detection hypothesis](detections/linux/orbit-medusa-rootkit-invariants.md): hash/path/credential보다 build pipeline invariant, filesystem skeleton, XOR string table, nested ELF, runtime divergence 중심 탐지 가설

### 3. AI Security / AI-Enabled Attack Operations
- [GTIG AI Threat Tracker field note](knowledge/ai-security/2026-05-14-gtig-ai-threat-tracker-ai-enabled-attack-operations.md): AI-enabled attack operations, Dynamic Modification, Figure 3의 취약점 탐색 역할 분담, PROMPTSPY/PROMPTFLUX/HONESTCUE/CANFAIL/LONGSTREAM 해석
- [MDASH and Oh my secuaudit comparison](knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md): AI vulnerability discovery pipeline, agentic harness, evidence promotion, proof workflow
- [ExploitGym methodology critique](knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md): AI agent exploit benchmark에서 model capability와 native agent scaffold, tool interface, process prompt, debugging environment, safety/refusal behavior가 섞이는 문제와 neutral harness track 제안
- [ExploitGym neutral harness experiment plan](experiments/2026-05-17-exploitgym-neutral-harness-comparison.md): third-party pre-registered agent stack, MCP/tool registry, debugging/logging policy, process prompt를 고정한 comparison track 설계
- [LLM-enabled Dynamic Modification detection hypothesis](detections/ai-security/llm-enabled-dynamic-modification.md): 외부 generation service가 runtime executable behavior에 영향을 주는 sequence 탐지 가설

### 4. Code Analysis / Agent-Readable Repository Operations
- [Source Code Indexing, LSP, and Sourcegraph for Security Analysis](knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md): LSP/Sourcegraph를 후보 축소와 context retrieval 계층으로 쓰는 대형 코드베이스 보안 분석 구조
- [Linux kernel maintenance context for AI security analysis](knowledge/code-analysis/2026-05-18-linux-kernel-maintenance-context-for-ai-security-analysis.md): Linux kernel의 subsystem, maintainer, mailing list, commit history, Documentation 구조를 AI agent가 따라갈 수 있는 repository memory model로 재해석하고 `oh-my-secuaudit`에 적용할 MAINTAINERS/contracts/decisions/checklists 방향 정리

Concept page candidates (AGENTS §10 임계점 도달 시 생성. 임계점: 노트 3+ 또는 노트 2+ + detection/experiment 1):
- `llm-assisted-vulnerability-discovery` — 현재: note 3 (GTIG, MDASH, ExploitGym) + experiment 1. **임계점 도달. Concept page 생성 검토 필요.**
- `ai-enabled-dynamic-modification` — 현재: note 1 (GTIG) + detection 1. **1 note 부족.**
- `ai-agent-supply-chain-risk` — 현재: note 1~2 (GTIG, oh-my-secuaudit 부분 포함). **note 1~2개 부족.**
- `ai-security-benchmark-harness` — 현재: note 2 (MDASH, ExploitGym) + experiment 1. **임계점 도달 후보. llm-assisted-vulnerability-discovery와 분리할지 검토 필요.**
- `agent-readable-repository-operations` — 현재: note 2 (LSP/Sourcegraph, Linux kernel maintenance context). **1 note 또는 detection/experiment 1개 부족.**
- `linux-rootkit-reuse-and-hook-surface` — 현재: note 1 (OrBit/Medusa) + detection 1. **1 note 부족.**
- `linux-runtime-view-divergence` — 현재: note 2 (Copy Fail/page cache, OrBit/Medusa) + detection 2. Concept 후보이나 page-cache와 rootkit을 함께 묶을지 분리할지 추가 검토 필요.

## Active Work & High-Priority TODOs
- **Research**: PLA COM API 직접 호출과 `logman.exe` 호출의 아티팩트 차이 분석
- **Detection**: ETW session tampering 탐지 룰 고도화
- **Experiment**: PLA/DCOM 원격 수집 실험 수행 (Windows Lab)
- **AI Security Research**: GTIG Figure 3을 바탕으로 SAST / fuzzing / LLM / human review의 역할 비교표 작성
- **AI Security Research**: ExploitGym / MDASH / GTIG를 연결해 `llm-assisted-vulnerability-discovery` 또는 `ai-security-benchmark-harness` concept page 생성 여부 검토
- **AI Security Benchmarking**: model / agent / skill prompt / MCP tool binding / debugging environment / validation oracle를 분리하는 reporting schema 초안 작성
- **AI Security Detection**: LLM/API call 이후 file write와 interpreter execution이 이어지는 Dynamic Modification sequence 탐지 가설 검증
- **Code Analysis / Repository Operations**: Linux kernel식 subsystem / maintainer / ABI / mailing-list memory model을 `oh-my-secuaudit`의 MAINTAINERS, contracts, decisions, PR checklist 구조로 이식하는 방안 설계
- **Linux Rootkit Research**: 공개 Linux rootkit repository를 기능별 hook surface로 분류하고 Medusa/OrBit 계열 invariant를 정적 탐지 가설로 발전
- **Linux Rootkit Detection**: `/etc/ld.so.preload`, suspicious `.so`, hook-like export set, filesystem skeleton, nested ELF, runtime view divergence를 조합한 scoring rule 설계

## Active Open Questions

각 컨셉 페이지 `Open Questions` 섹션을 한곳에서 색인합니다. 답을 채울 때는 원본 컨셉 페이지를 갱신하고, 검증된 항목은 여기서 제거합니다.

- [ETW](knowledge/concepts/etw.md#open-questions)
  - 특정 EDR이 사용하는 독자적 ETW provider 이름을 어떻게 효율적으로 inventory할 것인가?
  - PLA COM API 직접 호출 시 남는 cross-host 아티팩트의 신뢰도는 어느 정도인가?
- [PLA](knowledge/concepts/pla.md#open-questions)
  - PLA COM API 직접 호출 vs `logman.exe` 호출의 target-side 아티팩트 차이
  - `Service\` / `Autosession\` namespace collector를 정상 inventory에 포함시키려면 어떤 조회 방법이 필요한가?
- [DCOM](knowledge/concepts/dcom.md#open-questions)
  - PLA 관련 DCOM 인터페이스의 RPC UUID inventory가 탐지 정확도를 얼마나 높이는가?
  - DCOM authentication 레벨 차이가 PLA 원격 조작 시 어떤 흔적 차이로 나타나는가?
- [AI-enabled Dynamic Modification](detections/ai-security/llm-enabled-dynamic-modification.md#status-notes)
  - 탐지면을 특정 AI provider/model endpoint 기준으로 잡을 것인가, 아니면 `external generation service influences executable behavior at runtime`이라는 행위 불변성 기준으로 잡을 것인가?
  - 정상 AI coding assistant / CI code generation과 악성 Dynamic Modification의 false positive 경계를 어떻게 줄일 것인가?
- [ExploitGym / benchmark harness attribution](knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md#12-todo)
  - AI security benchmark에서 model capability와 agent scaffold/tool/process/environment 효과를 어떻게 분리해 보고할 것인가?
  - native deployed stack track과 common neutral harness track을 어떤 reporting schema로 병렬 제시할 것인가?
- [Linux kernel maintenance context / agent-readable repository operations](knowledge/code-analysis/2026-05-18-linux-kernel-maintenance-context-for-ai-security-analysis.md#8-실험-계획)
  - security audit repo에서 어느 수준의 MAINTAINERS/contracts/decision log가 AI agent 분석 품질을 실제로 높이는가?
  - output schema를 ABI처럼 관리할 때 생산성 증가와 문서화 overhead의 균형점은 어디인가?
- [OrBit / Medusa invariant detection](detections/linux/orbit-medusa-rootkit-invariants.md#status-notes)
  - Medusa/OrBit 계열 탐지를 path/hash 중심이 아닌 invariant scoring model로 만들 때 false positive 기준을 어떻게 잡을 것인가?
  - `linux-rootkit-reuse-and-hook-surface`를 별도 concept page로 승격할지, `linux-runtime-view-divergence`와 통합할지 결정 필요.

## Maintenance Rules (Quick Ref)
- **Value**: 우선순위 판단은 `VALUE_CRITERIA.md`와 `CONVERSION_STATUS.md`를 함께 참고. 임계치는 준비도 기준이고, 이미 전환된 주제는 추가 요약보다 gap / 실험 / tool issue로 전환
- **Linking**: 새 노드 추가 시 최소 3개의 기존 노드 연결 (`AGENTS.md §10`)
- **Distillation**: 개별 리서치 결과는 반드시 관련 Concept 페이지에 반영
- **Conversion Tracking**: field note가 blog / tool / checklist / experiment로 전환되면 `CONVERSION_STATUS.md`를 갱신
- **Safety**: PDF 원문이나 민감한 내부 정보는 절대 commit 금지