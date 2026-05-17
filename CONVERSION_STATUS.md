# Conversion Status

이 문서는 `security-field-notes`의 각 리서치 주제가 **field note → detection / experiment → blog / tool / checklist** 중 어디까지 전환되었는지 추적한다.

`VALUE_CRITERIA.md`는 무엇이 가치 있는지를 판단한다. 이 문서는 그 가치가 이미 실현되었는지, 아직 남아 있는지를 확인한다.

```text
가치가 높은 주제라도 이미 블로그화되었거나 도구화되었다면,
추가 field note의 한계효용은 낮아진다.

반대로 아직 field note에만 머물러 있고,
실험·탐지·도구·글로 전환되지 않은 주제는
다음 행동의 우선순위가 높아진다.
```

## Status Legend

| Status | Meaning |
|---|---|
| `field-note` | `knowledge/`에 정리 노트가 있음 |
| `detection-hypothesis` | `detections/`에 탐지 가설이 있음 |
| `experiment-planned` | `experiments/`에 실험 계획이 있음 |
| `blogged` | `windshock.github.io`에 공개 글로 전환됨 |
| `toolized` | 별도 GitHub repo / script / skill / workflow로 구현됨 |
| `checklist-ready` | 실무 체크리스트나 schema로 전환 가능 |
| `not-found-in-quick-search` | 접근 가능한 repo에서 빠른 검색으로는 전환 흔적을 찾지 못함. 부재 확정은 아님 |
| `overlap` | 이미 다른 repo나 글의 방향과 강하게 겹침. 새 산출물보다 gap만 추적하는 편이 좋음 |

## How To Use This File

새 작업을 고를 때 다음 순서로 본다.

1. `VALUE_CRITERIA.md`로 가치가 있는지 판단한다.
2. 이 파일에서 이미 blog/tool/checklist로 전환되었는지 확인한다.
3. 이미 전환된 주제라면 새 요약보다 gap, 실험 결과, 탐지 룰, tool issue로 전환한다.
4. 아직 전환되지 않은 주제라면 field note를 다음 산출물로 밀어낸다.

## Conversion Table

| Theme | Field Notes | Detection / Experiment | Blog | Tool / Repo | Current Gap | Next Best Action |
|---|---|---|---|---|---|---|
| AI-assisted vulnerability discovery / benchmark interpretation | GTIG, MDASH, ExploitGym notes | ExploitGym neutral harness experiment plan | `not-found-in-quick-search` | Strong `oh-my-secuaudit` overlap: producer skills, finding contract, architecture synthesis, SPR lifecycle already exist | 새 field memo를 쓰면 기존 tool direction과 중복될 가능성 큼. 다만 benchmark 결과 해석과 조직 흡수 구조를 연결하는 public-facing 글은 아직 미확인 | 새 글보다 `oh-my-secuaudit`의 missing contract / benchmark mode gap만 이관. 필요 시 concept page는 짧게 유지 |
| AI security benchmark harness attribution | ExploitGym methodology critique, MDASH comparison | Neutral harness comparison plan | `not-found-in-quick-search` | `oh-my-secuaudit` has related orchestration and contract model, but no confirmed `AGENT_REGISTRY.yaml` / `MCP_TOOL_REGISTRY.yaml` / benchmark mode in quick search | 연구 논문식 schema보다 실제 repo에 들어갈 registry/manifest gap이 더 가치 있음 | `oh-my-secuaudit` issue or TODO로 `AGENT_REGISTRY.yaml`, `MCP_TOOL_REGISTRY.yaml`, Tool Binding Manifest 초안 작성 |
| AI-enabled Dynamic Modification | GTIG AI Threat Tracker field note | `detections/ai-security/llm-enabled-dynamic-modification.md` | `not-found-in-quick-search` | not found | 탐지 가설은 있으나 harmless toy harness와 telemetry가 없음 | benign toy harness 또는 experiment note로 `external generation service -> local rewrite -> execution` sequence 검증 |
| AI agent / skill / connector supply chain risk | GTIG note, Oh my secuaudit-related TODOs | checklist idea only | `not-found-in-quick-search` | partial overlap with `oh-my-secuaudit` skill system | 구체 체크리스트나 dependency review workflow가 아직 분리되지 않음 | `knowledge/ai-security/` 또는 `templates/`에 AI agent dependency supply chain checklist 초안 작성 |
| PLA / DCOM / ETW agentless telemetry | `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md`, ETW/PLA/DCOM concepts | PLA/DCOM lab plan, logman ETW tampering detection | `not-found-in-quick-search` | not found | Windows lab 관찰값이 아직 없음. `logman.exe` vs PLA COM 직접 호출 차이가 open question으로 남아 있음 | Windows lab telemetry result 작성이 최우선. 이후 detection refinement / blog 가능 |
| ETW session tampering detection | ETW concept, PLA/DCOM note | `detections/windows/logman-etw-session-tampering.md` | `not-found-in-quick-search` | not found | command-line 중심 가설에서 COM/direct API artifact로 확장 필요 | lab result 이후 detection note를 Sigma/KQL 후보로 확장 |
| Copy Fail / page-cache integrity | Linux integrity notes and TODOs | `detections/linux/page-cache-integrity-divergence.md` | `not-found-in-quick-search` | not found | O_DIRECT / cached read divergence의 distro/filesystem별 실험 결과 부족 | Rocky 8/9 ext4/xfs lab note 작성 또는 SIEM sample JSON 생성 |
| OrBit / Medusa rootkit invariant detection | OrBit / Medusa field note | `detections/linux/orbit-medusa-rootkit-invariants.md` | `not-found-in-quick-search` | not found | path/hash보다 invariant scoring이라는 관점은 있으나 샘플 검증과 rule candidate 부족 | YARA/scoring rule 후보 작성 또는 isolated lab static fingerprint 비교 |
| Linux runtime view divergence | Copy Fail/page cache, OrBit/Medusa notes | page-cache divergence and rootkit invariant detections | `not-found-in-quick-search` | not found | page-cache integrity와 rootkit runtime divergence를 같은 concept로 묶을지 미결정 | concept page 승격 전, divergence 유형 taxonomy 초안 작성 |
| Linux kernel maintenance context for AI security analysis | Linux kernel maintenance context note | no direct experiment yet | `not-found-in-quick-search` | strong potential overlap with `oh-my-secuaudit` maintainers/contracts/decisions TODOs | 글보다 `oh-my-secuaudit` repo 구조 개선으로 전환하는 것이 더 직접적 | `MAINTAINERS.md`, `docs/contracts/`, `docs/decisions/`, PR template gap을 tool repo로 이관 |
| Source Code Indexing / LSP / Sourcegraph security analysis | source indexing note | no dedicated detection/experiment | possibly overlaps previous blog/tool direction, not confirmed here | likely overlaps `oh-my-secuaudit` and `fortify_ml` direction | 새 글 가치보다 existing tool integration gap 확인 필요 | `oh-my-secuaudit` candidate reduction / context retrieval 기능 gap으로 재분류 |
| Security testing as code / assessment handoff | multiple notes and `oh-my-secuaudit` README | toolized as skill methodology | likely already public through repo README | `oh-my-secuaudit` has `security-testing-as-code` methodology skill | 이미 toolized. 추가 field note 한계효용 낮음 | 새 요약 금지. 실제 skill validation / examples / fixture gap만 추적 |

## Current Highest-Value Unconverted Items

현재 기준으로 가장 가치가 높은 것은 **이미 글/도구로 전환된 주제를 다시 설명하는 것**이 아니라, 아래처럼 아직 전환이 덜 된 주제를 다음 단계로 밀어내는 것이다.

### 1. PLA / DCOM / ETW Windows lab telemetry result

Why:
- Field note, concept pages, lab plan, detection hypothesis가 이미 있음.
- 그러나 실제 target-side artifact 관찰값이 아직 부족함.
- `logman.exe`와 PLA COM 직접 호출의 차이는 탐지와 블로그 모두에 영향을 주는 핵심 open question.

Next:
```text
experiments/2026-05-XX-pla-dcom-etw-lab-results.md
```

### 2. AI-enabled Dynamic Modification benign toy harness

Why:
- Dynamic Modification은 아직 field note + detection hypothesis 단계.
- 실제 악성 기능 없이도 benign harness로 telemetry sequence를 만들 수 있음.
- 방어 중심 산출물로 public repo에 안전하게 남길 수 있음.

Next:
```text
experiments/2026-05-XX-ai-dynamic-modification-toy-harness.md
```

### 3. OrBit / Medusa invariant scoring rule draft

Why:
- Rootkit 재사용과 build-option 차이는 흥미롭지만 아직 탐지 규칙으로 전환되지 않음.
- path/hash 중심 탐지의 한계를 invariant scoring으로 바꾸는 방법론 가치가 있음.

Next:
```text
detections/linux/orbit-medusa-rootkit-invariants.md 확장
또는
experiments/2026-05-XX-medusa-orbit-static-fingerprint-comparison.md
```

### 4. AI agent supply chain checklist

Why:
- GTIG와 oh-my-secuaudit 방향을 연결할 수 있음.
- 단, broad essay보다 실무 체크리스트가 더 가치 있음.

Next:
```text
templates/ai-agent-supply-chain-checklist.md
```

## Lower-Priority Because Already Converted Or Overlapping

다음 주제들은 중요하지만, 이미 blog/tool 방향과 겹치므로 새 field note보다 구체 gap만 추적한다.

- `oh-my-secuaudit` 전체 방법론: 이미 repo와 README에서 상당 부분 toolized.
- AI-assisted vulnerability discovery 일반론: GTIG/MDASH/ExploitGym 노트가 충분하고, oh-my-secuaudit와 overlap 큼.
- Security testing as code: 이미 `oh-my-secuaudit` skill layer에 존재.
- Architecture-to-product bridge: 이미 `oh-my-secuaudit` README에서 SPR lifecycle로 toolized.

## Maintenance Rule

새로운 field note를 추가할 때는 이 파일의 해당 row를 함께 갱신한다.

- blog로 전환되면 `Blog` 열에 링크를 넣는다.
- tool/repo로 전환되면 `Tool / Repo` 열에 링크를 넣는다.
- experiment 결과가 생기면 `Detection / Experiment` 열과 `Current Gap`을 갱신한다.
- 이미 전환된 주제를 다시 요약하려는 경우, 먼저 `Current Gap`이 남아 있는지 확인한다.
