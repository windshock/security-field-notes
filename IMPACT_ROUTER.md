# Impact Router

이 문서는 `security-field-notes`의 field note, concept candidate, detection idea, experiment idea가 어떤 산출물로 전환되어야 하는지 판단하는 라우팅 규칙이다.

`VALUE_CRITERIA.md`가 가치 판단을 담당하고, `CONVERSION_STATUS.md`가 이미 전환되었는지 확인하며, `DOWNSTREAM_TARGETS.md`가 실행 대상을 정의한다면, 이 문서는 그 셋을 연결해 **다음 행동**을 고른다.

## Core Rule

```text
A valuable note should not always become another note.
Route it to the place where it changes practice.
```

한국어로 풀면 다음과 같다.

```text
가치 있는 노트가 항상 또 다른 노트가 되어야 하는 것은 아니다.
실천을 바꾸는 위치로 보내야 한다.
```

## Routing Order

새 작업을 제안하기 전에 다음 순서로 판단한다.

1. `VALUE_CRITERIA.md`를 보고 이 주제가 가치 있는지 판단한다.
2. `CONVERSION_STATUS.md`를 보고 이미 blog / tool / checklist / experiment로 전환되었는지 확인한다.
3. `DOWNSTREAM_TARGETS.md`를 보고 살아 있는 실행 대상이 있는지 확인한다.
4. 실행 대상이 있으면 새 field note보다 해당 대상의 issue / PR / checklist / template을 우선한다.
5. 실행 대상이 없고 반복 개념이면 `knowledge/concepts/`로 보낸다.
6. 관찰값이 부족하면 `experiments/`로 보낸다.
7. 탐지 규칙으로 만들 수 있으면 `detections/`로 보낸다.
8. 사회적/조직적 메시지가 강하고 기술 근거가 충분하면 `windshock.github.io`로 보낸다.
9. 아직 불확실하면 `security-field-notes`에 유지하고 conversion target을 명시한다.

## Routing Table

| Signal in Field Note | Best Route | Why | Example |
|---|---|---|---|
| 반복 개념이 여러 노트에 등장 | `knowledge/concepts/` | 장기적으로 재사용할 개념 허브가 필요함 | ETW, PLA, DCOM |
| 관찰값이 부족한 탐지 가설 | `experiments/` | 추론보다 telemetry/evidence가 먼저 필요함 | PLA/DCOM/ETW lab result |
| 행위 기반 탐지 가능성이 중심 | `detections/` | hypothesis를 detection surface로 정리해야 함 | Dynamic Modification sequence |
| 이미 구현 대상 repo가 있음 | downstream repo issue/PR | 실천 변화가 field note 내부가 아니라 repo 구조 변경에서 발생함 | `oh-my-secuaudit` maintenance skeleton |
| 대중적 문제 제기 가치가 큼 | `windshock.github.io` | 구조적 비판이나 거버넌스 메시지로 확장 가능 | 발견 이후 조직 흡수 문제 |
| 실무자가 바로 쓸 절차 | `templates/` 또는 checklist | 반복 사용 가능한 practitioner artifact가 됨 | AI agent supply chain checklist |
| 기존 도구와 강하게 overlap | 새 field note 금지, gap만 이관 | 중복 요약보다 missing contract/fixture/validation을 보강해야 함 | `oh-my-secuaudit` skill system |
| 안전한 toy validation 가능 | `security-hypothesis-lab` 또는 `experiments/` | public repo에 안전한 증거를 남길 수 있음 | benign Dynamic Modification harness |
| 코드 구조 clustering/ML triage와 연결 | `fortify_ml` | field note보다 모델/데이터/실험으로 전환 가치가 큼 | entry-sink cluster testing |

## Decision Template

작업 후보를 평가할 때 아래 블록을 채운다.

```yaml
theme:
value:
readiness:
conversion_status:
downstream_target:
best_next_action:
not_recommended:
reason:
```

예시:

```yaml
theme: Linux kernel maintenance context for AI security analysis
value: high
readiness: enough
conversion_status: field-note only
downstream_target: windshock/oh-my-secuaudit
best_next_action: add maintenance skeleton PR
not_recommended:
  - another generic field memo
  - blog-first conversion
  - concept page before implementation
reason:
  - the note already proposes concrete repo structure
  - oh-my-secuaudit is a live implementation target
  - value is realized by changing the tool, not by summarizing again
```

## Anti-Patterns

Avoid these mistakes:

1. **Concept-page reflex**
   - 노트가 여러 개 쌓였다는 이유만으로 concept page를 만들지 않는다.
   - 먼저 실행 대상이 있는지 확인한다.

2. **Blog-first reflex**
   - 메시지가 흥미롭다는 이유만으로 바로 블로그화하지 않는다.
   - 기술 근거나 구현 대상이 있으면 먼저 그쪽을 강화한다.

3. **Experiment reflex**
   - 실험이 가능하다는 이유만으로 항상 최우선으로 두지 않는다.
   - 구조 개선이 더 큰 영향을 주면 downstream repo를 먼저 고친다.

4. **Duplicate-summary reflex**
   - 이미 toolized/blogged된 주제를 다시 요약하지 않는다.
   - 남은 gap을 issue, PR, fixture, checklist로 전환한다.

5. **Security-field-notes gravity**
   - 모든 통찰을 이 저장소 안에 붙잡아두지 않는다.
   - 이 저장소는 통찰의 종착지가 아니라 routing hub다.

## Current High-Value Routes

현재 가장 중요한 routing 후보는 다음과 같다.

| Theme | Route | Next Action |
|---|---|---|
| Linux kernel maintenance context for AI security analysis | `windshock/oh-my-secuaudit` | Add Linux-kernel-style maintenance skeleton: `MAINTAINERS.md`, `CONTRIBUTING.md`, `docs/contracts/`, `docs/decisions/`, `docs/review-checklists/` |
| PLA / DCOM / ETW agentless telemetry | `experiments/` | Write Windows lab telemetry result after observation |
| AI-enabled Dynamic Modification | `experiments/` or `security-hypothesis-lab` | Build harmless toy harness and telemetry sequence |
| OrBit / Medusa invariant detection | `detections/` or `experiments/` | Add invariant scoring rule draft or static fingerprint comparison |
| AI agent supply chain risk | `templates/` or `oh-my-secuaudit` | Draft checklist or route to skill/dependency governance docs |

## Maintenance Rule

When adding a new field note:

- Add or update its row in `CONVERSION_STATUS.md`.
- Identify its downstream target using `DOWNSTREAM_TARGETS.md`.
- If a live target exists, write `Best next action` as an issue/PR/checklist/tool change, not another note.
- If no live target exists, keep it in `security-field-notes` and mark what evidence is missing.
