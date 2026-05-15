# TODO

이 파일은 저장소 전체의 backlog입니다. 각 항목은 나중에 `knowledge/`, `experiments/`, `detections/` 아래의 구체적인 문서로 분리할 수 있습니다.

## Inbox

- [ ] PLA/DCOM 기반 원격 ETW 수집 구조를 Windows lab에서 재현하기
- [ ] `logman -s` 기반 원격 trace session 생성/조회/중지 행위 정리
- [ ] PLA COM API 직접 호출 방식과 `logman.exe` 방식의 탐지 차이 비교
- [ ] `Microsoft-Windows-Diagnosis-PLA/Operational` 로그에서 확인 가능한 이벤트 조사
- [ ] Data Collector Set 생성/시작/중지 시 파일, 레지스트리, 이벤트 흔적 수집
- [ ] `Session`, `Service`, `Autosession` namespace별 가시성 차이 확인
- [ ] ETW session tampering 탐지 룰 후보 작성
- [ ] EDR 미도입 서버에서 보완적 telemetry 수집 구조로 활용 가능한지 검토
- [ ] Rocky Linux에서 Copy Fail 대응용 KernelCare live patch 적용 여부를 대표 커널별로 검증하기
- [ ] `rfxn/copyfail`의 `copyfail-local-check --json` 결과를 SIEM에 넣을 수 있는 형태로 샘플링하기
- [ ] Rocky 8/9에서 `O_DIRECT` 기반 page-cache integrity check가 ext4/xfs에서 안정적으로 동작하는지 확인하기
- [ ] `fanotify` 기반 setuid 실행 직전 검증/차단 PoC 가능성 조사하기
- [ ] GTIG AI Threat Tracker의 Figure 3을 바탕으로 SAST / fuzzing / LLM / human review의 역할 비교표 작성하기
- [ ] LLM-assisted semantic logic flaw 탐색용 방어 prompt와 검증 workflow 설계하기
- [ ] AI agent / skill / connector dependency supply chain checklist를 별도 문서로 분리하기
- [ ] 공개 Linux rootkit repository를 기능별 hook surface로 분류하기
- [ ] Medusa default build와 `rknet.c` 포함 build의 static fingerprint 차이를 isolated lab에서 비교하기
- [ ] eBPF / io_uring / UEFI / cloud rootkit 계열을 Linux rootkit primitive taxonomy에 매핑하기

## Detection Ideas

- [ ] `logman.exe` command line 기반 탐지
- [ ] 비관리자 단말에서 서버 대상 DCOM/RPC 접근 탐지
- [ ] `PerfLogs` 하위 비정상 ETL/BLG 파일 생성 탐지
- [ ] UNC path로 저장되는 Data Collector Set output 탐지
- [ ] 보안 관련 ETW session stop/delete/update 탐지
- [ ] `Service\\` 또는 `Autosession\\` namespace에 생성되는 의심 collector 탐지
- [ ] `AF_ALG` socket family 생성 탐지
- [ ] `AF_ALG` + AEAD/authencesn + `splice()` 연속 호출 탐지
- [ ] setuid binary 실행 직후 shell 실행 chain 탐지
- [ ] cached read hash와 `O_DIRECT` read hash divergence 탐지
- [ ] `drop_caches` 호출과 privilege escalation 의심 이벤트 상관분석
- [ ] LLM/API call 이후 file write와 interpreter execution이 이어지는 Dynamic Modification sequence 탐지
- [ ] LLM-generated decoy logic 후보를 code-size growth / dead branch / irrelevant helper code 기준으로 탐지 가능성 검토
- [ ] Accessibility tree 또는 UI hierarchy가 외부 모델로 전송되고 JSON action으로 CLICK/SWIPE가 수행되는 agentic malware loop 탐지
- [ ] AI gateway / relay service / account pooling 인프라와 비정상 egress 탐지
- [ ] Medusa/OrBit 계열 XOR string table을 key-independent threshold YARA 후보로 설계하기
- [ ] Nested ELF + rootkit-specific inner fingerprint 조합의 false positive 측정하기
- [ ] `/etc/ld.so.preload` 변경, suspicious `.so`, hook-like export set, filesystem skeleton을 조합한 Linux rootkit scoring rule 설계하기
- [ ] `/proc` view와 network/host telemetry 불일치를 이용한 userland rootkit runtime divergence 탐지 검토하기

## Writing Ideas

- [ ] 블로그: "원격 이벤트 수집은 원래 되는데, 왜 이 글이 흥미로운가?"
- [ ] 블로그: "Event Log 수집과 ETW trace session 생성의 차이"
- [ ] LinkedIn 요약: "Agentless EDR이라는 표현이 오해를 부르는 이유"
- [ ] 내부 보고서 초안: "PLA/DCOM 기반 원격 telemetry와 탐지 사각지대"
- [ ] 블로그: "MDASH와 Oh my secuaudit: 대규모 AI 취약점 발굴 공장과 재현 가능한 보안감사 운영체계"
- [ ] LinkedIn 요약: "모델이 아니라 harness가 제품이다: MDASH가 보안감사 자동화에 주는 교훈"
- [ ] 블로그: "커널 취약점 대응에서 live patch는 합법적 후킹인가?"
- [ ] LinkedIn 요약: "Copy Fail과 page-cache integrity detection: 파일은 정상인데 cache가 거짓말할 때"
- [ ] 블로그: "AI 악성코드의 진짜 변화: 코드를 생성하는 AI에서, 공격을 운영하는 AI로"
- [ ] 블로그: "폴리모픽 악성코드 이후의 시대: AI가 변형 엔진이 될 때"
- [ ] LinkedIn 요약: "AI로 다 되는가? Figure 3이 보여주는 취약점 탐색의 역할 분담"
- [ ] 블로그: "루트킷 기술은 정체됐는가? 오래된 primitive와 새로운 platform의 이동"
- [ ] LinkedIn 요약: "OrBit/Medusa가 보여준 공개 루트킷 재사용의 현실"

## Oh my secuaudit Follow-ups

- [ ] README에 `Finder → Skeptic → Prover → Reporter` 역할 모델 추가
- [ ] `sec-cluster` 문서에 `cluster`, `family`, `root-cause dedup`의 차이 명시
- [ ] finding schema 또는 metadata에 proof status enum 추가
- [ ] 과거 직접 분석한 취약점 case를 regression set으로 정리
- [ ] MDASH 비교 노트를 바탕으로 Oh my secuaudit positioning 문서 작성
- [ ] `AGENT_REGISTRY.yaml` 초안 작성: agent description, role, allowed outputs, forbidden outputs, risk budget 포함
- [ ] `MCP_TOOL_REGISTRY.yaml` 초안 작성: tool description, tags, input/output schema, risk level 포함
- [ ] Agent-Tool Matching PoC 작성: agent description과 MCP tool description을 embedding/LLM으로 매칭
- [ ] Tool Binding Manifest 초안 작성: 추천 tool, rejected tool, policy rejection reason 포함
- [ ] Tool Factory PoC 작성: `create_toolset(agent_role, policy_profile, backends)` 형태
- [ ] `/audit/stream` event model 설계: status/tool_start/artifact/evidence/verifier/done 이벤트 분리
- [ ] simulation lab의 `scenario_id / audit_id / thread_id` 규칙을 별도 설계 문서로 분리
- [ ] `toy-java-sqli` simulation lab과 `expected_findings.yaml` regression fixture 작성

## AI Security Follow-ups

컨셉 페이지 후보는 AGENTS §10 임계점(노트 3+ 또는 노트 2+ + detection/experiment 1)에 도달하면 자동으로 생성 후보가 됩니다. 현재 카운트는 `LLM_CONTEXT.md` Concept page candidates 참조.

- [ ] `detections/ai-security/llm-enabled-dynamic-modification.md`를 Sigma/KQL 초안으로 확장
- [ ] benign toy harness로 `external generation service → local rewrite → execution` telemetry 수집
- [ ] 정상 AI coding assistant / CI code generation과 악성 Dynamic Modification의 false positive 경계 정리

## Repo Maintenance

- [ ] 참고 자료는 `references/README.md`에 먼저 등록
- [ ] 실험 전에는 `experiments/`에 실험 계획부터 작성
- [ ] 탐지 룰은 검증 전에는 `hypothesis` 상태로 표시
- [ ] public repo에 원문 PDF나 민감 자료를 직접 올리지 않기
