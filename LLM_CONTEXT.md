# LLM Context & Master Index

이 파일은 AI agent가 저장소의 구조와 상태를 빠르게 파악하고 다음 작업을 수행하기 위한 **Master Map**입니다.

## Repository Purpose
보안 리서치 과정에서 발생하는 파편화된 정보를 **증류(Distillation)**하고 **연결(Linking)**하여, 검증 가능한 탐지 가설과 실험으로 발전시키는 지식 엔진입니다.

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

### 2. Linux Integrity & EDR
- [Page Cache Integrity](detections/linux/page-cache-integrity-divergence.md): (Concept 페이지 승격 예정)

### 3. AI Security / AI-Enabled Attack Operations
- [GTIG AI Threat Tracker field note](knowledge/ai-security/2026-05-14-gtig-ai-threat-tracker-ai-enabled-attack-operations.md): AI-enabled attack operations, Dynamic Modification, Figure 3의 취약점 탐색 역할 분담, PROMPTSPY/PROMPTFLUX/HONESTCUE/CANFAIL/LONGSTREAM 해석
- [MDASH and Oh my secuaudit comparison](knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md): AI vulnerability discovery pipeline, agentic harness, evidence promotion, proof workflow
- [LLM-enabled Dynamic Modification detection hypothesis](detections/ai-security/llm-enabled-dynamic-modification.md): 외부 generation service가 runtime executable behavior에 영향을 주는 sequence 탐지 가설

Concept page candidates (AGENTS §10 임계점 도달 시 생성. 임계점: 노트 3+ 또는 노트 2+ + detection/experiment 1):
- `ai-enabled-dynamic-modification` — 현재: note 1 (GTIG) + detection 1. **1 note 부족.**
- `llm-assisted-vulnerability-discovery` — 현재: note 2 (GTIG, MDASH). **detection/experiment 1개 또는 note 1개 부족.**
- `ai-agent-supply-chain-risk` — 현재: note 1~2 (GTIG, oh-my-secuaudit 부분 포함). **note 1~2개 부족.**

## Active Work & High-Priority TODOs
- **Research**: PLA COM API 직접 호출과 `logman.exe` 호출의 아티팩트 차이 분석
- **Detection**: ETW session tampering 탐지 룰 고도화
- **Experiment**: PLA/DCOM 원격 수집 실험 수행 (Windows Lab)
- **AI Security Research**: GTIG Figure 3을 바탕으로 SAST / fuzzing / LLM / human review의 역할 비교표 작성
- **AI Security Detection**: LLM/API call 이후 file write와 interpreter execution이 이어지는 Dynamic Modification sequence 탐지 가설 검증

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

## Maintenance Rules (Quick Ref)
- **Linking**: 새 노드 추가 시 최소 3개의 기존 노드 연결 (`AGENTS.md §10`)
- **Distillation**: 개별 리서치 결과는 반드시 관련 Concept 페이지에 반영
- **Safety**: PDF 원문이나 민감한 내부 정보는 절대 commit 금지
