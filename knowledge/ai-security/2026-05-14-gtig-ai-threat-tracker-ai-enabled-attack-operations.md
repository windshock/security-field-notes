# GTIG AI Threat Tracker: AI 공격 운영화와 Dynamic Modification 관점 정리

Date: 2026-05-14  
Category: AI Security / Threat Intelligence / Malware / Vulnerability Discovery  
Status: draft  
Source: Google Threat Intelligence Group, "GTIG AI Threat Tracker: Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access", Google Cloud Blog, 2026-05-12; windshock / ChatGPT discussion notes, 2026-05-14

Related notes:
- `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`
- Related notes beyond these two: none found yet

Related detection hypothesis:
- `detections/ai-security/llm-enabled-dynamic-modification.md`

## 1. 한 줄 요약

이 GTIG 문서는 "공격자가 AI를 사용한다"는 일반론이 아니라, AI가 취약점 탐색, 악성코드 변형, 자율 공격 오케스트레이션, LLM 접근 인프라, AI 공급망 공격에 어떻게 들어왔는지를 비교적 균형 있게 정리한 자료다.

대화에서의 최종 결론은 다음과 같다.

```text
AI 공격 위협의 중심은 단순한 코드 생성이 아니라,
공격 운영의 자동화와 악성코드 변형 로직의 외부화다.
```

## 2. 배경

자료는 크게 두 축으로 구성된다.

```text
AI as a Tool
→ 취약점 탐색
→ exploit 개발
→ 악성코드 난독화 / Dynamic Modification
→ 자율 공격 오케스트레이션
→ 정찰 / 피싱 / 정보작전
→ LLM 접근 인프라 산업화

AI as a Target
→ agent skill / connector / wrapper library / AI gateway / dependency supply chain 공격
```

따라서 이 문서를 단순한 악성코드 보고서로만 보면 좁다. 더 정확히는 **AI-enabled attack operations** 보고서다. 다만 실제 사례 중 가장 선명하고 실무적으로 중요한 부분은 악성코드와 공격 도구 진화다.

## 3. 기존에 알려진 것

- 공격자는 이미 자동화, 프록시, 계정 풀링, bulletproof infrastructure, exploit kit, malware builder를 사용해왔다.
- LLM 기반 공격 플랫폼화도 이 연장선에 있다.
- 기존 polymorphic / metamorphic malware는 자체 변형 엔진, packer, junk code insertion, encryption routine을 이용해 정적 탐지를 우회해왔다.
- SAST, DAST, fuzzing은 각각 잘하는 영역이 있지만, 비즈니스 로직과 권한 예외 같은 semantic flaw는 전통적으로 사람이 더 잘 찾는 영역이었다.

## 4. 새롭게 볼 만한 점

### 4.1 Figure 3의 실무적 가치

문서 중 Figure 3, "LLM vulnerability discovery capabilities compared with other discovery mechanisms"가 특히 유용하다.

이 그림과 주변 설명은 다음 메시지를 제공한다.

```text
AI는 취약점 탐색을 전부 대체하지 않는다.
AI는 기존 도구가 약한 의미 기반 취약점 탐색 계층을 보완한다.
```

GTIG가 예로 든 2FA bypass 사례는 memory corruption이나 단순 input sanitization 문제가 아니라, 개발자가 코드에 박아둔 잘못된 trust assumption에서 나온 semantic logic flaw였다.

이 점은 임원이나 팀장이 "이제 AI로 다 되는 것 아니냐"고 물을 때 대응하기 좋다.

```text
Fuzzer는 crash와 edge case를 잘 찾는다.
SAST는 known dangerous sink와 반복 패턴을 잘 찾는다.
LLM은 개발 의도와 보안 로직의 모순을 읽는 데 강점이 있다.

따라서 방향은 AI 대체가 아니라,
AI + SAST + fuzzing + threat modeling + human validation이다.
```

### 4.2 Dynamic Modification이 더 흥미로운 이유

AI 공격 플랫폼화는 어느 정도 예측 가능한 방향이다. API gateway, relay, account auto-registration, anti-detect browser, account pooling은 기존 공격 인프라의 LLM 버전으로 볼 수 있다.

반면 Dynamic Modification은 더 본질적인 변화다.

```text
기존 polymorphic malware:
변형 규칙이 악성코드 내부에 있다.

AI 기반 Dynamic Modification:
변형 규칙이 모델, 프롬프트, API, 외부 생성 workflow 쪽으로 이동한다.
```

즉 방어자가 분석해야 할 대상이 고정된 변형 알고리즘이 아니라, **생성-수정-실행이라는 행위 흐름**으로 바뀐다.

이 관점에서 PROMPTFLUX, HONESTCUE, CANFAIL, LONGSTREAM, PROMPTSPY는 각각 다른 레이어를 보여준다.

| 사례 | 관찰 포인트 | 의미 |
|---|---|---|
| PROMPTFLUX | Dynamic Modification | 코드 자체가 계속 변하는 구조 |
| HONESTCUE | AI 기반 난독화 / evasion payload generation | 정적 시그니처 우회 자동화 |
| CANFAIL | LLM-generated decoy logic | 악성 기능 주변에 노이즈 코드 삽입 |
| LONGSTREAM | 대량의 비활성 / 무관 코드 | 코드 리뷰와 정적 분석을 방해하는 camouflage |
| PROMPTSPY | LLM 기반 UI 판단 / 행동 loop | 악성코드 안에 agentic decision loop 삽입 |

### 4.3 PROMPTSPY는 "AI로 만든 악성코드"보다 한 단계 더 나간다

PROMPTSPY의 핵심은 악성코드를 AI가 작성했다는 점이 아니다.

더 중요한 점은 악성코드가 피해자 환경의 UI 상태를 직렬화하고, 모델 응답을 구조화된 action으로 받아 실제 동작으로 연결한다는 점이다.

```text
피해자 환경 상태 수집
→ UI hierarchy serialization
→ LLM 호출
→ JSON action 응답
→ CLICK / SWIPE 등 실제 동작 수행
```

이것은 "악성코드가 명령을 받는다"에서 "악성코드가 상황을 해석하고 행동을 선택한다"로 넘어가는 신호다.

## 5. 공격자 관점

공격자가 AI를 흥미롭게 보는 이유는 단순 생산성 향상만이 아니다.

### 5.1 취약점 탐색 범위 확장

LLM은 다음과 같은 취약점 후보를 더 잘 제기할 수 있다.

- 권한 예외와 인증 흐름의 모순
- hardcoded trust assumption
- 2FA / MFA / role enforcement의 조건 불일치
- 개발자의 의도와 실제 코드 경로 사이의 불일치
- 전통적 scanner가 sink나 crash로 보기 어려운 semantic flaw

### 5.2 변형 로직의 외부화

Dynamic Modification에서 공격자는 변형 엔진을 malware binary 안에 고정하지 않아도 된다.

```text
malware sample
→ prompt / model / API / operator workflow
→ 매번 다른 obfuscation / decoy / code mutation
```

이 구조는 YARA, hash, string, fixed packer signature 중심 탐지를 약화시킬 수 있다.

### 5.3 공격 운영의 자동화

PROMPTSPY 같은 사례는 모델이 command generation, UI navigation, real-time decision support에 들어갈 수 있음을 보여준다.

공격자는 사람 operator가 직접 판단하던 일부 단계를 모델에 넘겨 scaled and adaptive activity를 만들 수 있다.

### 5.4 LLM 접근 인프라의 산업화

공격자는 LLM을 일회성 도구가 아니라 공격 인프라의 일부로 다루기 시작했다.

- API gateway / aggregator
- proxy relay
- premium account auto-registration
- account pooling
- anti-detect browser
- management center

이 흐름은 예전의 proxy, botnet, bulletproof hosting이 LLM 접근권으로 확장된 것으로 볼 수 있다.

## 6. 방어자 관점

### 6.1 AI는 기존 도구의 대체물이 아니라 추가 탐지 계층이다

Figure 3의 메시지를 방어 관점으로 정리하면 다음과 같다.

```text
SAST / DAST / fuzzing / human review / LLM review는 서로 다른 취약점 공간을 본다.
AI는 전체를 대체하는 도구가 아니라 기존 탐지 사각지대를 줄이는 계층이다.
```

따라서 AI 보안 투자는 "진단 인력과 기존 도구를 줄이기 위한 것"이 아니라, 다음 영역을 보완하기 위한 것으로 설명하는 것이 적절하다.

- semantic logic flaw
- authorization exception
- workflow contradiction
- business rule mismatch
- hardcoded trust assumption

### 6.2 탐지는 코드 조각보다 행위 불변성에 집중해야 한다

Dynamic Modification 시대에는 파일 해시와 고정 문자열의 가치가 더 빨리 줄어든다.

대신 다음 불변성이 중요하다.

- 외부 모델/API 호출 이후 코드 또는 스크립트가 변경되는가
- 변경 직후 interpreter / compiler / script host가 실행되는가
- LLM 응답이 command, code, UI action, coordinates, shell fragment로 파싱되는가
- API key, prompt template, model endpoint, JSON mode 같은 AI integration artifact가 있는가
- decoy code 삽입으로 코드량과 의미 없는 branch가 비정상적으로 증가하는가

### 6.3 AI 시스템 자체의 주변부를 보호해야 한다

문서는 frontier model 자체보다 주변부가 먼저 공격받는다고 본다.

- agent skill
- connector
- wrapper library
- LLM gateway
- PyPI / npm / GitHub Actions dependency
- API key and build secret

이는 SAIF 관점에서 Insecure Integrated Component와 Rogue Actions 위험으로 연결된다.

```text
모델이 안전한가?
보다
모델이 연결된 도구, 권한, 공급망이 안전한가?
가 더 현실적인 질문이다.
```

## 7. 탐지 포인트

### 7.1 AI API boundary

- Data source: proxy logs, DNS logs, EDR network telemetry, cloud egress logs
- Observable: model API endpoint, AI gateway, relay service, unusual account pooling infrastructure
- Suspicious condition: 업무상 필요 없는 서버/스크립트/모바일 앱에서 LLM API를 호출하고, 응답 직후 command execution 또는 code generation이 이어짐
- False positive: 정상 AI 기능, 보안 연구 도구, 개발 보조 도구

### 7.2 Dynamic Modification sequence

- Data source: EDR, Sysmon, file integrity monitoring, script block logging, process telemetry
- Observable: API call → file write / self-modification → interpreter execution sequence
- Suspicious condition: 외부 모델 응답 이후 스크립트 본문, payload template, obfuscation layer가 변경되고 바로 실행됨
- False positive: 정상 code generation pipeline, CI helper, AI coding assistant

### 7.3 LLM-generated decoy logic

- Data source: code repository scan, malware sandbox, static analysis output
- Observable: 사용되지 않는 대량의 helper code, 설명적인 LLM-style comments, 기능과 무관한 반복 query, dead branch
- Suspicious condition: primary malicious behavior와 무관한 administrative / environment / time-related code가 과도하게 삽입됨
- False positive: 자동 생성 코드, template-heavy application, legacy utility module

### 7.4 Agentic malware loop

- Data source: mobile telemetry, accessibility API usage, network logs, sandbox behavior
- Observable: UI hierarchy serialization, model request, structured JSON action response, simulated tap/swipe
- Suspicious condition: 접근성 권한을 가진 앱이 화면 상태를 외부 모델에 전송하고, 응답을 사용해 UI action을 수행함
- False positive: legitimate accessibility automation, test automation, RPA app

### 7.5 AI supply chain

- Data source: dependency manifests, package install logs, GitHub Actions audit logs, secret scanning, artifact provenance
- Observable: LLM gateway / agent skill / connector package의 postinstall network call, credential access, suspicious PR or dependency update
- Suspicious condition: AI integration package가 build secret, API key, cloud token에 접근하거나 외부로 전송함
- False positive: 정상 telemetry, package update checker, build integration

## 8. 실험 계획

- [ ] Benign toy sample로 Dynamic Modification telemetry를 만든다. 실제 악성 기능 없이 mock LLM response를 받아 harmless script를 재작성하고 실행하는 흐름만 재현한다.
- [ ] 정적 hash / string 탐지와 행위 sequence 탐지의 차이를 비교한다.
- [ ] `API call → file write → interpreter execution` sequence를 Sigma/KQL 후보로 정리한다.
- [ ] 코드 리뷰 관점에서 hardcoded trust assumption / authentication bypass exception 후보를 찾는 LLM prompt를 방어용으로 설계한다.
- [ ] SAST/fuzzer/LLM가 각각 찾는 취약점 유형을 표로 정리해 임원/팀장 설명 자료로 만든다.
- [ ] AI agent / skill / connector dependency의 supply chain checklist를 별도 문서로 분리한다.

## 9. 결론 강도

- [x] 확인된 사실: GTIG 문서는 AI가 취약점 탐색, 악성코드 변형, 자율 공격 오케스트레이션, LLM 접근 인프라, AI supply chain 공격에 활용되는 사례를 제시한다.
- [x] 확인된 사실: 문서의 Figure 3은 LLM과 기존 취약점 탐색 기법의 강점을 구분해 설명하는 데 유용하다.
- [x] 확인된 사실: PROMPTSPY, PROMPTFLUX, HONESTCUE, CANFAIL, LONGSTREAM은 AI와 악성코드 결합 양상을 보여주는 사례로 제시된다.
- [ ] 가설 수준: Dynamic Modification의 핵심은 변형 로직이 malware 내부에서 외부 모델/API/workflow로 이동하는 것이라는 해석.
- [ ] 추가 검증 필요: 실제 운영 환경에서 AI API boundary 기반 탐지가 false positive를 감당할 수 있을지.
- [ ] 추가 검증 필요: LLM-generated decoy logic을 정적 휴리스틱으로 안정적으로 구분할 수 있을지.

## 10. 대화에서 도출한 결론

이번 대화에서 정리된 핵심 관점은 다음이다.

```text
1. 이 자료는 AI 공격 위협을 잘 구조화한 보고서다.
2. 악성코드 사례가 강하지만, 본질은 AI-enabled attack operations다.
3. AI 공격 플랫폼화는 예상 가능한 흐름이다.
4. 더 흥미로운 지점은 Dynamic Modification이다.
5. Dynamic Modification은 변형 로직의 외부화로 해석할 수 있다.
6. Figure 3은 "AI로 다 되는가?"에 대응하기 좋은 자료다.
7. AI는 기존 도구를 대체하지 않고, 의미 기반 취약점 탐색 계층을 추가한다.
8. 방어 전략은 코드 조각 탐지보다 행위 불변성, AI API boundary, supply chain 보호로 이동해야 한다.
```

블로그나 LinkedIn으로 확장한다면 다음 제목 후보가 적절하다.

```text
AI 악성코드의 진짜 변화: 코드를 생성하는 AI에서, 공격을 운영하는 AI로

폴리모픽 악성코드 이후의 시대: AI가 변형 엔진이 될 때

AI가 취약점을 찾기 시작했다: 문제는 모델이 아니라, 모델이 연결된 세계다
```

## 11. 참고 자료

- Google Threat Intelligence Group, "GTIG AI Threat Tracker: Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access", Google Cloud Blog, 2026-05-12.
- Uploaded local archive: `Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access | Google Cloud Blog.pdf`
- Local archive SHA-256: `840cb22fc3560d85f53dbbf960f130b91f2064dabf441a6990901ccd129c6dff`
- Related internal note: `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- Related internal note: `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`

## 12. Concept page proposal

새 concept page 후보:

- `knowledge/concepts/ai-enabled-dynamic-modification.md`
- `knowledge/concepts/llm-assisted-vulnerability-discovery.md`
- `knowledge/concepts/ai-agent-supply-chain-risk.md`

규칙상 새 concept page는 중복 방지를 위해 별도 판단 후 생성한다.
