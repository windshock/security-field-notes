# Downstream Targets

이 문서는 `security-field-notes`에서 나온 통찰이 실제로 어디로 전환되어야 하는지 판단하기 위한 실행 대상 목록이다.

`VALUE_CRITERIA.md`는 무엇이 가치 있는지 판단하고, `CONVERSION_STATUS.md`는 이미 전환되었는지 확인한다. 이 문서는 **전환 목적지**를 정의한다.

```text
A valuable note should not always become another note.
If there is a living downstream target, route the insight there.
```

한국어로 풀면 다음과 같다.

```text
가치 있는 노트가 항상 또 다른 노트가 되어야 하는 것은 아니다.
살아 있는 실행 대상이 있으면, 통찰은 그곳으로 가야 한다.
```

## Routing Priority

새 field note, concept 후보, detection idea, experiment idea를 볼 때 다음 순서로 판단한다.

1. 이미 살아 있는 tool / repo / blog / checklist 대상이 있는가?
2. 있다면 새 field note나 concept page보다 해당 대상의 issue / PR / checklist / template 개선을 우선한다.
3. 실행 대상이 없고 반복 개념이면 `knowledge/concepts/`로 보낸다.
4. 관찰값이 부족하면 `experiments/`로 보낸다.
5. 탐지 가능성이 중심이면 `detections/`로 보낸다.
6. 대중적 설명과 거버넌스 메시지가 중심이면 `windshock.github.io`로 보낸다.
7. 아직 불확실하면 `security-field-notes`에 유지하고 conversion target을 명시한다.

## Target: windshock/oh-my-secuaudit

Use when the insight concerns:

- security audit workflow
- skill as subsystem
- output contract / schema compatibility
- evidence handoff
- architecture synthesis
- security testing as code
- AI agent maintainability
- audit reproducibility
- finding lifecycle and product/security requirement bridge

Preferred outputs:

- GitHub issue
- PR
- `MAINTAINERS.md`
- `CONTRIBUTING.md`
- `docs/contracts/`
- `docs/decisions/`
- `docs/review-checklists/`
- `.github/ISSUE_TEMPLATE/`
- regression fixture / example project
- validation script

Do not create another field note when:

- the note already proposes concrete repository structure;
- the insight changes how `oh-my-secuaudit` should be maintained;
- the output is a contract, checklist, maintainer map, or workflow rule.

Example:

```yaml
theme: Linux kernel maintenance context for AI security analysis
route: windshock/oh-my-secuaudit
reason: skill system should be treated as subsystem; output schema as ABI; decision log as compressed mailing-list memory
best_next_action: add maintenance skeleton PR
not_recommended: another generic concept page or field memo
```

## Target: windshock.github.io

Use when the insight concerns:

- public narrative
- structural critique
- governance / culture / responsibility
- security as public good
- practitioner-facing explanation
- mature technical argument for broader audience

Preferred outputs:

- Korean blog post
- English blog post
- LinkedIn summary
- visual diagram / explanatory card

Do not route here when:

- the idea is not technically grounded yet;
- the idea still lacks lab evidence;
- a live tool repo exists and needs implementation first;
- the topic has already been blogged and only needs a tool gap closed.

## Target: security-field-notes

Use when the insight is still:

- exploratory;
- evidence-incomplete;
- conceptually unstable;
- useful for later synthesis but not ready for public/tool output;
- dependent on additional references or experiments.

Preferred outputs:

- `knowledge/<platform>/YYYY-MM-DD-topic.md`
- `knowledge/concepts/<concept>.md`
- `detections/<platform>/<hypothesis>.md`
- `experiments/YYYY-MM-DD-topic.md`
- `TODO.md`
- `CONVERSION_STATUS.md` row update

Do not keep it here forever when:

- a downstream repo clearly exists;
- the note already includes concrete file paths for that repo;
- the next action is implementation, not interpretation.

## Target: security-hypothesis-lab

Use when the insight concerns:

- small reproducible security experiment;
- safe toy harness;
- telemetry sequence demonstration;
- non-weaponized validation artifact;
- isolated lab evidence for a detection hypothesis.

Preferred outputs:

- lab fixture
- harmless toy program
- event/log sample
- experiment README
- replayable test case

Example candidates:

- AI-enabled Dynamic Modification benign toy harness
- page-cache / runtime-view divergence experiment
- PLA/DCOM/ETW observation harness, if safely abstracted

## Target: fortify_ml

Use when the insight concerns:

- grouping similar code entry-sink structures;
- machine-learning-assisted triage;
- payload-per-cluster testing strategy;
- code structure embedding;
- security finding deduplication by structural similarity.

Preferred outputs:

- feature idea
- experiment script
- model/data representation note
- issue or PR in `fortify_ml`

Do not route here when:

- the issue is primarily governance, documentation, or workflow contract;
- no code structure clustering or ML triage element exists.

## Target: templates/ inside security-field-notes

Use when the insight should become a reusable practitioner artifact but does not yet belong to a tool repo.

Preferred outputs:

- checklist
- review template
- threat model prompt
- experiment template
- field note intake form

Example candidates:

- AI agent dependency / skill / connector supply chain checklist
- Dynamic Modification triage checklist
- rootkit invariant scoring review checklist

## Target Selection Checklist

Before suggesting the next best action, answer these questions:

1. Does a live downstream repo already exist?
2. Would a PR/issue there change practice more than another field note here?
3. Is the next action implementation, lab evidence, detection logic, public explanation, or concept synthesis?
4. Has this theme already been blogged or toolized?
5. Does the proposed output reduce future repetition or create another duplicate summary?

If the answer to #1 and #2 is yes, route to the downstream repo first.
