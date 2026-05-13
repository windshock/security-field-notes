# MDASH와 Oh my secuaudit 비교: 대규모 AI 취약점 발굴 공장과 재현 가능한 보안감사 운영체계

Date: 2026-05-13
Category: AI Security / AppSec Automation / Security Audit Workflow
Status: draft
Source: Taesoo Kim, "Defense at AI speed: Microsoft’s new multi-model agentic security system tops leading industry benchmark", Microsoft Security Blog, 2026-05-12; discussion notes from ChatGPT session

## 1. 핵심 요약

Microsoft의 MDASH는 단순히 "LLM으로 취약점을 찾는 도구"가 아니다. 글의 핵심은 모델 자체보다 **모델을 둘러싼 agentic harness**에 있다.

MDASH가 제시한 구조는 다음과 같이 요약된다.

```text
Prepare
→ Scan
→ Validate
→ Dedup
→ Prove
```

우리 대화에서는 이 구조를 다음과 같은 일반화된 보안감사 pipeline으로 재해석했다.

```text
코드 구조화
→ 후보 경로 추출
→ 취약점 가설 생성
→ 반박 agent 검증
→ 중복 제거
→ 실제 trigger 가능성 확인
→ evidence 중심 보고서
```

초기 판단에서는 Oh my secuaudit가 이 방향으로 "가야 한다"고 보았지만, 실제 `windshock/oh-my-secuaudit` 저장소를 확인한 뒤 결론을 수정했다.

**Oh my secuaudit는 이미 이 흐름의 상당 부분을 구현하고 있다.**

차이는 기능의 유무가 아니라 규모, 데이터, 조직 인프라, proving depth에 있다.

- MDASH: Microsoft 내부 코드, MSRC 이력, Patch Tuesday, WARP/ACS 조직, 대규모 agent/model 운영이 결합된 **취약점 발굴 공장**
- Oh my secuaudit: static/runtime/external/architecture/security-testing-as-code를 연결해 보안감사를 재현 가능한 구조로 남기는 **보안감사 운영체계**

## 2. MDASH에서 중요한 기술 아이디어

Microsoft 글에서 중요한 것은 개별 취약점보다 운영 구조다.

MDASH는 다음을 강조한다.

1. 단일 모델보다 harness가 중요하다.
2. auditor, debater, prover처럼 역할이 다른 agent가 필요하다.
3. candidate finding은 곧 finding이 아니다.
4. validation, deduplication, proof construction을 거쳐야 한다.
5. domain plugin이 모델이 모르는 시스템 내부 invariant를 보강한다.
6. 새 모델이 나와도 pipeline과 plugin 투자분은 유지된다.

이 구조는 보안감사 자동화에도 그대로 유효하다. 다만 Microsoft 규모의 인프라를 그대로 복제하려고 하면 실패한다.

## 3. Oh my secuaudit가 이미 갖고 있는 대응 구조

Oh my secuaudit의 README는 여러 skill을 producer/synthesis 구조로 나눈다.

- `sec-audit-static`: source repo 기반 SAST/SCA/secret/reporting
- `sec-cluster`: dataflow 기반 security code clustering
- `sec-audit-dast`: runtime/API assessment
- `external-software-analysis`: third-party binary/package analysis
- `security-architecture-review`: DFD, trust boundary, Attack Flow, SPR backlog
- `security-testing-as-code`: PoC/evidence/handoff를 project로 보존

이 구조는 MDASH의 단일 pipeline과 다르게 보이지만, 실제로는 더 넓은 감사 lifecycle을 다룬다.

### 3.1 코드 구조화

`sec-audit-static`은 asset identification, API inventory, global filter/interceptor 확인, Joern CPG/call graph snapshot, slicing, facet tagging, state store, source/decompiled 비교를 포함한다.

즉 단순 rule scan이 아니라, 후보를 구조화해서 이후 단계에서 재사용할 수 있게 만드는 흐름이다.

### 3.2 후보 경로 추출

`run_static_audit.sh`는 다음 단계를 orchestration한다.

```text
version check
→ state store init
→ candidate scoping
→ Semgrep / Joern
→ auth/key exposure scan
→ decompiled pass
→ enrichment
→ slicing
→ ranking
→ fuzz queue
→ validation
→ report
```

이것은 MDASH식 Prepare/Scan/Validate/Prove를 local audit runner로 현실화한 형태다.

### 3.3 취약점 가설 생성

`vuln_automation_principles.md`는 다음 loop를 제시한다.

```text
Discovery → Analysis
Hypothesis → Evidence → PoC → Re-check
```

여기서 Discovery는 저비용 signal 수집이고, Analysis는 축소된 candidate set에 대해 고정밀 reasoning을 수행하는 단계다.

이것은 LLM을 모든 코드에 무차별 적용하지 않고, token cost와 false positive를 줄이기 위한 실용적 구조다.

### 3.4 Source → Sink 검증

`taint_tracking.md`는 confirmed finding으로 승격하는 조건을 명시한다.

```text
Source: user-controlled input
Transform: validation/sanitization/intermediate processing
Sink: execution point
```

그리고 Source → Sink path가 입증될 때만 confirmed finding으로 올리도록 한다.

이 점은 매우 중요하다. Oh my secuaudit의 finding은 "LLM이 취약하다고 말했다"가 아니라 **flow evidence 기반으로 승격되는 구조**다.

### 3.5 중복 제거와 grouping

`sec-cluster`는 `(Endpoint, Sink)` 단위로 code path를 묶고, 같은 review strategy를 공유하는 cluster를 만든다.

이때 cluster의 의미는 "동일한 취약점"이 아니다.

```text
cluster = 같은 review strategy를 적용할 가능성
```

따라서 cluster는 신뢰하는 대상이 아니라, 사용하면서 검증해야 하는 scoping 도구다.

MDASH의 dedup은 동일하거나 의미적으로 같은 finding을 병합하는 기능에 가깝다. Oh my secuaudit의 cluster는 그보다 앞단에서 reviewer의 작업 전략을 줄이는 기능이다.

향후에는 다음 세 개를 명확히 분리하면 좋다.

```text
cluster = 같은 검토 전략
family = 같은 sink anchor 계열
root-cause dedup = 같은 취약점 / 같은 원인
```

### 3.6 실제 trigger 가능성 확인

`poc_policy.md`는 PoC 생성을 best-effort로 정의하면서도 우선순위를 둔다.

1. JUnit5 integration test
2. Playwright
3. Jazzer fuzzing
4. ZAP

`security-testing-as-code`는 여기서 한 단계 더 나아간다. 보안 진단 결과를 문서가 아니라 version-controlled project로 보존한다.

핵심 명제는 다음과 같다.

```text
A diagnosis should be a project.
```

즉 PoC code, saved HTTP request/response, commit hash, runtime evidence, handoff plan이 보고서와 함께 남는다.

이것은 MDASH의 Prove stage를 실무 감사 방식으로 재해석한 좋은 방향이다.

## 4. Microsoft 인프라 때문에 그대로 따라가기 어려운 것

MDASH에서 인상적인 부분 중 일부는 Microsoft이기 때문에 가능하다. Oh my secuaudit가 그대로 복제하려고 하면 안 되는 항목은 다음과 같다.

| MDASH 요소 | Microsoft 의존성 | Oh my secuaudit 판단 |
|---|---|---|
| Windows 내부 코드 전체 분석 | proprietary code, symbol, build, owner context | 일반화 불가. 대상 repo 단위로 축소 |
| MSRC 5년치 ground truth | 실제 보안 대응/패치 이력 DB | 공개 CVE, 과거 개인 분석 case, regression set으로 대체 |
| Patch Tuesday 연동 | 제품 owner, triage, release process | 이슈/보고서/SPR backlog까지가 현실적 |
| WARP/ACS 전문 조직 | kernel/network offensive research team | 개인/소규모 AppSec workflow로 포지셔닝 |
| 100개+ specialized agents | agent 관리, prompt/versioning, 평가셋, 비용 | 3~4개 role agent로 충분 |
| multi-model ensemble/A-B testing | 여러 SOTA/distilled 모델 비용과 telemetry | 기본 모델 + optional reviewer 모델 정도 |
| Windows kernel proving plugin | CLFS/tcpip/ikeext 내부 invariant | Spring/JVM/API/FileUpload/Deserialization recipe로 대체 |
| 대규모 sanitizer/fuzz/build farm | VM, crash triage, coverage infra | prove-lite, short fuzz gate, local reproducible PoC |
| CyberGym leaderboard 최적화 | benchmark 반복 실행과 leaderboard 운영 | 실제 감사 case regression으로 충분 |

## 5. Oh my secuaudit가 계속 밀어야 할 방향

Oh my secuaudit는 MDASH의 축소판이 아니다. 더 정확한 정체성은 다음과 같다.

```text
재현 가능한 보안감사 운영체계
```

MDASH가 "AI 취약점 발굴 공장"이라면, Oh my secuaudit는 "보안감사자가 판단한 근거를 구조화하고 다음 감사자가 이어받을 수 있게 만드는 체계"다.

따라서 강화해야 할 축은 다음이다.

1. State Store 중심의 reproducibility
2. Source/Sink/Transform evidence 기반 finding promotion
3. cluster를 통한 review strategy 재사용
4. PoC/evidence artifact packaging
5. DFD/Attack Flow/Trust Boundary로 architecture synthesis
6. SPR backlog로 remediation lifecycle 연결
7. 이전 감사 결과와 다음 감사 결과의 delta 관리

## 6. MDASH에서 추가로 빌릴 수 있는 것

현재 Oh my secuaudit는 큰 방향이 이미 맞다. 추가로 빌릴 수 있는 것은 다음 세 가지다.

### 6.1 명시적인 Skeptic/Debater role

현재 Oh my secuaudit에는 validation gate, unknown taxonomy, rule validation, low-confidence gate가 있다. 그러나 MDASH처럼 반박 agent 역할을 이름 붙여 명확히 분리하면 설명력이 좋아진다.

제안 구조:

```text
Finder
→ Skeptic
→ Prover
→ Reporter
```

- Finder: 후보 경로와 취약점 가설 생성
- Skeptic: reachability, sanitizer, auth boundary, false positive 가능성 반박
- Prover: PoC, test, fuzz gate, runtime evidence 확인
- Reporter: finding JSON, markdown, architecture mapping, SPR 변환

### 6.2 dedup layer 명시

`sec-cluster`는 review strategy grouping이다. 이것과 별개로 root-cause dedup을 명시하면 MDASH와 비교 가능한 구조가 된다.

제안 taxonomy:

```text
candidate_id: 특정 sink callsite 중심 후보
family_id: 같은 sink anchor 계열
cluster_id: 같은 review strategy 그룹
root_cause_id: 같은 취약점 원인
finding_id: 보고서에 올라가는 최종 finding
```

### 6.3 Prove status 세분화

PoC policy는 이미 있으므로, proof 상태를 더 세분화하면 좋다.

제안 상태:

```text
not_reachable
source_confirmed
sink_confirmed
flow_confirmed
runtime_triggered
poc_reproducible
fixed_and_rechecked
```

이 상태를 finding metadata에 넣으면, "확인됨"이라는 단어의 모호성이 줄어든다.

## 7. 최종 결론

초기에는 Oh my secuaudit를 MDASH와 비교하면서 "앞으로 이런 구조를 가져야 한다"고 보았다. 그러나 실제 repo를 확인한 뒤 결론은 바뀌었다.

```text
Oh my secuaudit는 이미 MDASH와 유사한 pipeline 사고를 갖고 있다.
```

다만 두 시스템의 목표는 다르다.

```text
MDASH = Microsoft 인프라 위에서 작동하는 대규모 취약점 발굴 공장
Oh my secuaudit = 개인/소규모 팀이 운영 가능한 재현 가능한 보안감사 체계
```

따라서 Oh my secuaudit의 다음 과제는 MDASH를 따라잡는 것이 아니다.

다음 과제는 이미 존재하는 구조를 더 명확한 언어로 정리하는 것이다.

```text
Pipeline
Role separation
Evidence promotion
Cluster vs dedup separation
Proof status
Architecture synthesis
Assessment-as-code lifecycle
```

이렇게 정리하면 Oh my secuaudit는 "LLM 보안도구"가 아니라, **보안감사자가 생각하고 검증하고 보고하는 방식을 실행 가능한 구조로 만든 프로젝트**로 설명될 수 있다.

## 8. 후속 TODO

- [ ] Oh my secuaudit README에 `Finder → Skeptic → Prover → Reporter` 역할 모델을 추가한다.
- [ ] `sec-cluster` 문서에 `cluster`, `family`, `root-cause dedup`의 차이를 명시한다.
- [ ] finding schema 또는 metadata에 proof status enum을 추가한다.
- [ ] 과거 직접 분석한 취약점 case를 regression set으로 정리한다.
- [ ] MDASH 비교 글을 블로그 초안으로 확장한다.

## 9. 참고 자료

- Microsoft Security Blog, "Defense at AI speed: Microsoft’s new multi-model agentic security system tops leading industry benchmark", 2026-05-12.
- `windshock/oh-my-secuaudit`, `README.md`.
- `windshock/oh-my-secuaudit`, `skills/static/sec-audit-static/SKILL.md`.
- `windshock/oh-my-secuaudit`, `skills/static/sec-cluster/SKILL.md`.
- `windshock/oh-my-secuaudit`, `skills/methodology/security-testing-as-code/SKILL.md`.
- `windshock/oh-my-secuaudit`, `skills/architect/security-architecture-review/SKILL.md`.
