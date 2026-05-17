# Value Criteria

이 문서는 `security-field-notes`에서 무엇을 우선 정리하고, 무엇을 concept page로 승격하고, 어떤 후속 작업을 먼저 할지 판단하기 위한 가치 기준이다.

`AGENTS.md`의 concept page 생성 임계치(노트 3개 이상, 또는 노트 2개 + detection/experiment 1개 이상)는 **준비도(readiness)** 기준이다. 준비도는 "지금 문서화할 재료가 충분한가"를 판단한다. 그러나 이 저장소에서 더 중요한 질문은 "이 일이 나와 세상에 어떤 변화를 남기는가"이다.

이 문서의 기준은 windshock.github.io의 대표 서문인 "코드 이후에도, 구조는 남는다"의 문제의식에서 출발한다.

> 병목은 취약점을 발견하는 데만 있지 않다. 발견은 곧 개선이 아니고, 진단은 곧 변화가 아니다. 남은 문제는 조직이 발견한 것을 어떻게 이해하는지, 구조가 그것을 어떻게 흡수하는지, 제도가 그것을 어떻게 지속시키는지에 있다.

따라서 이 저장소의 우선순위는 단순히 새롭거나 기술적으로 흥미로운 순서가 아니다. 가치 있는 항목은 발견을 구조로 바꾸고, 분석을 방법론으로 남기며, 기술적 사실을 지속 가능한 개선으로 연결하는 항목이다.

## 1. Core Principle

```text
Readiness tells whether a topic is mature enough to document.
Value tells whether documenting it moves security practice forward.
```

한국어로 풀면 다음과 같다.

```text
임계치는 문서화할 준비가 되었는지를 본다.
가치는 그 문서화가 발견 이후의 변화를 만드는지를 본다.
```

## 2. Value Questions

새 작업, concept page, detection hypothesis, experiment, blog seed의 우선순위를 정할 때 다음 질문을 사용한다.

### 2.1 발견 이후 전환력

이 작업은 단순한 발견, 요약, 탐지를 넘어 실제 개선으로 이어지는가?

좋은 작업은 다음 중 하나 이상을 만든다.

- 반복 가능한 실험 절차
- 탐지 가설 또는 룰 후보
- 개발/운영 의사결정에 쓰일 설명 구조
- 조직이 후속 조치를 취할 수 있는 체크리스트
- 보고서가 아니라 코드, 프로세스, 정책으로 남는 결과물

### 2.2 구조적 병목 설명력

이 작업은 개별 사건이 아니라 반복되는 실패 구조를 드러내는가?

예를 들어 단순히 "새 공격 기법이 있다"보다 다음 질문에 답하는 작업이 더 가치 있다.

- 왜 같은 종류의 실패가 반복되는가?
- 왜 기술적으로 알려진 문제가 조직 안에서는 해결되지 않는가?
- 어떤 책임 구조, 계약 구조, 운영 구조가 보안을 약화시키는가?
- 탐지나 진단이 왜 실제 변화로 이어지지 않는가?

### 2.3 기술적 증거성

이 작업은 실제 시스템, 실제 코드, 실제 로그, 실제 실패 패턴에 기반하는가?

철학적 주장만으로는 부족하다. 보안 지식은 가능한 한 다음으로 연결되어야 한다.

- source code path
- telemetry source
- detection surface
- reproducible lab plan
- observable artifact
- false positive boundary
- validation note

### 2.4 방법론화 가능성

이 작업은 다음 사람이 다시 사용할 수 있는 형식으로 남는가?

가치 있는 작업은 일회성 해석을 넘어 다음 중 하나로 변환된다.

- concept page
- experiment plan
- detection hypothesis
- checklist
- scoring model
- audit workflow
- schema / contract
- blog/report outline

### 2.5 거버넌스 연결성

이 작업은 기술 자체를 넘어 책임, 우선순위, 제도, 운영 구조와 연결되는가?

다음 질문에 답할수록 가치가 높다.

- 누가 이 문제를 소유해야 하는가?
- 어떤 evidence가 있어야 조직이 움직이는가?
- 어떤 지표는 문제를 보이게 하지만 흡수하지 못하게 만드는가?
- 어떤 도구 도입은 실제 개선이 아니라 책임 전가로 끝나는가?
- 어떤 정책 또는 프로세스 변화가 필요해지는가?

### 2.6 공유 가능성

이 작업은 개인 메모를 넘어 공개 가능한 지식 자산이 되는가?

좋은 작업은 다음 독자에게 도움이 된다.

- 보안 실무자
- 개발자와 플랫폼 엔지니어
- 보안 자동화 도구를 만드는 사람
- 보안 리더와 의사결정자
- 정책/거버넌스 관점에서 기술 실패를 해석하는 사람

public repository인 만큼, 공격 절차나 민감 정보를 공개하지 않으면서도 방어 중심의 지식, 방법론, 실험 설계를 남기는 것이 중요하다.

## 3. Practical Scoring Rubric

작업 후보를 비교할 때 아래 점수를 사용할 수 있다. 점수는 절대값이 아니라 우선순위 토론을 위한 도구다.

| Criterion | Question | Score |
|---|---|---|
| 전환력 | 발견을 실험, 탐지, 코드, 절차, 의사결정으로 바꾸는가? | 0-3 |
| 구조 설명력 | 반복되는 실패 구조나 병목을 드러내는가? | 0-3 |
| 기술적 증거성 | 실제 artifact, telemetry, source, lab plan으로 뒷받침되는가? | 0-3 |
| 방법론화 | 다음 사람이 재사용할 수 있는 workflow/checklist/schema로 남는가? | 0-3 |
| 거버넌스 연결성 | 책임, 우선순위, 정책, 운영 구조와 연결되는가? | 0-3 |
| 공유 가능성 | public repo/blog/도구로 안전하게 공유 가능한가? | 0-3 |

해석 기준:

- 0-5: 보관은 가능하지만 우선순위 낮음
- 6-10: 노트 또는 TODO로 유지
- 11-14: concept page, experiment, detection 후보
- 15-18: 블로그/보고서/도구화까지 고려할 핵심 주제

## 4. Readiness vs Value

준비도와 가치는 다르다.

| 구분 | 질문 | 예 |
|---|---|---|
| Readiness | 재료가 충분히 쌓였는가? | 노트 3개, detection 1개, experiment 1개 |
| Value | 이것이 발견 이후의 구조를 바꾸는가? | 조직이 AI benchmark를 오해하지 않도록 reporting schema를 제안 |

따라서 어떤 주제는 임계치에 도달했더라도 가치가 낮을 수 있고, 반대로 아직 노트가 적어도 전략적으로 중요한 주제일 수 있다.

작업 우선순위는 다음 순서로 판단한다.

1. 안전성: public repo에 기록 가능한가?
2. 가치: 위 Value Questions에 얼마나 답하는가?
3. 준비도: 문서화할 재료가 충분한가?
4. 연결성: 기존 concept, detection, experiment, TODO와 연결되는가?
5. 다음 행동: 글, 실험, 탐지, 도구, 체크리스트 중 무엇으로 이어지는가?

## 5. Applying This to Current Themes

### AI-assisted vulnerability discovery

단순한 질문은 "LLM이 취약점을 잘 찾는가"이다. 그러나 이 저장소의 가치 기준에서는 더 중요한 질문이 있다.

```text
AI가 발견한 취약점 후보를 조직은 어떻게 검증하고,
어떻게 기존 SAST/fuzzing/human review와 역할을 나누며,
어떻게 보고서가 아니라 반복 가능한 개선 구조로 흡수할 것인가?
```

이 주제는 다음 이유로 가치가 높다.

- 발견 이후 전환력: AI finding을 proof, reproduction, triage, remediation workflow로 연결한다.
- 구조 설명력: model score와 실제 보안 개선 사이의 gap을 드러낸다.
- 방법론화: benchmark reporting schema, neutral harness, validation oracle로 남길 수 있다.
- 거버넌스 연결성: 조직이 AI 보안 도구를 어떻게 평가하고 책임을 배분할지에 영향을 준다.

### AI-enabled dynamic modification

단순한 질문은 "AI가 악성코드를 만든다"이다. 가치 기준에서 더 중요한 질문은 다음이다.

```text
변형 로직이 malware 내부에서 외부 model/API/workflow로 이동할 때,
방어자는 어떤 행위 불변성을 기준으로 탐지하고 검증해야 하는가?
```

이 주제는 detection engineering 가치가 높다. 다만 concept page 승격은 추가 사례와 실험이 쌓인 뒤가 더 적절할 수 있다.

### Linux rootkit invariant detection

단순한 질문은 "이 rootkit family를 어떻게 탐지할 것인가"이다. 더 큰 질문은 다음이다.

```text
path/hash/credential 중심 탐지가 무너질 때,
runtime view divergence와 build invariant는 어떤 방어 방법론이 될 수 있는가?
```

이 주제는 기술적 증거성과 방법론화 가능성이 크다. 거버넌스보다는 탐지/실험 중심 가치가 강하다.

## 6. How Agents Should Use This

AI agent는 새 작업을 제안하거나 우선순위를 매길 때 다음 순서를 따른다.

1. `LLM_CONTEXT.md`에서 현재 후보와 active work를 확인한다.
2. `AGENTS.md`의 readiness threshold를 확인한다.
3. 이 문서의 Value Questions로 실제 우선순위를 다시 평가한다.
4. 임계치에 도달했다는 이유만으로 바로 concept page를 만들지 않는다.
5. 가치가 높지만 준비도가 낮은 항목은 `LLM_CONTEXT.md`의 candidate 또는 `TODO.md`로 남긴다.
6. 가치와 준비도가 모두 높은 항목은 concept page / detection / experiment / blog seed 중 가장 적절한 형태로 승격한다.

## 7. One-line Standard

이 저장소에서 가장 가치 있는 작업은 다음 문장을 만족하는 작업이다.

```text
발견을 구조로 바꾸고, 분석을 방법론으로 남기며, 기술적 사실을 지속 가능한 보안 개선으로 연결하는가?
```
