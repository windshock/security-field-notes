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

## Writing Ideas

- [ ] 블로그: "원격 이벤트 수집은 원래 되는데, 왜 이 글이 흥미로운가?"
- [ ] 블로그: "Event Log 수집과 ETW trace session 생성의 차이"
- [ ] LinkedIn 요약: "Agentless EDR이라는 표현이 오해를 부르는 이유"
- [ ] 내부 보고서 초안: "PLA/DCOM 기반 원격 telemetry와 탐지 사각지대"
- [ ] 블로그: "MDASH와 Oh my secuaudit: 대규모 AI 취약점 발굴 공장과 재현 가능한 보안감사 운영체계"
- [ ] LinkedIn 요약: "모델이 아니라 harness가 제품이다: MDASH가 보안감사 자동화에 주는 교훈"
- [ ] 블로그: "커널 취약점 대응에서 live patch는 합법적 후킹인가?"
- [ ] LinkedIn 요약: "Copy Fail과 page-cache integrity detection: 파일은 정상인데 cache가 거짓말할 때"

## Oh my secuaudit Follow-ups

- [ ] README에 `Finder → Skeptic → Prover → Reporter` 역할 모델 추가
- [ ] `sec-cluster` 문서에 `cluster`, `family`, `root-cause dedup`의 차이 명시
- [ ] finding schema 또는 metadata에 proof status enum 추가
- [ ] 과거 직접 분석한 취약점 case를 regression set으로 정리
- [ ] MDASH 비교 노트를 바탕으로 Oh my secuaudit positioning 문서 작성

## Repo Maintenance

- [ ] 참고 자료는 `references/README.md`에 먼저 등록
- [ ] 실험 전에는 `experiments/`에 실험 계획부터 작성
- [ ] 탐지 룰은 검증 전에는 `hypothesis` 상태로 표시
- [ ] public repo에 원문 PDF나 민감 자료를 직접 올리지 않기
