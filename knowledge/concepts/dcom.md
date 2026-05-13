# DCOM (Distributed COM)

Status: evolving
Type: Technical Concept
Last reviewed: 2026-05-14

## Short Definition

DCOM은 RPC 위에 올라간 분산 COM 호출 인프라다. 이 저장소에서 DCOM은 그 자체로 흥미롭다기보다 **PLA의 원격 인터페이스를 노출시키는 전송 계층**으로 주로 다뤄진다 (`2026-05-10-pla-dcom-agentless-edr.md` §4.2).

## Why It Matters

이 저장소의 핵심 관찰은 "PLA가 *DCOM을 통해* 원격 Data Collector Set 조작을 노출한다"는 점이다. 즉 DCOM은 다음 세 가지 의미를 한 번에 가진다:

1. **합법 관리 채널** — Performance Monitor, MMC, WMI 등의 통신 기반
2. **공격 채널** — PLA 인터페이스를 거쳐 원격 ETW session 변조 가능
3. **WMI 대안** — 많은 조직이 WMI lateral movement는 보지만 PLA/DCOM은 덜 본다 (`2026-05-10-pla-dcom-agentless-edr.md` §5.4)

## Offensive Use

- **Living-off-the-land 원격 telemetry 조작**: PLA COM 객체를 DCOM으로 호출해 ETW session 생성/변조. 별도 바이너리 업로드 없이 가능.
- **WMI 우회**: WMI 중심 탐지 회피 경로.
- **Lateral movement (배경 지식)**: `ShellWindows`, `MMC20.Application`, `Excel.Application` 같은 일반 DCOM lateral move 객체는 이 저장소 범위 밖이지만, 같은 전송 계층이라는 점은 기억해 둘 가치가 있다.

## Defensive Use

- **Hardening**: 불필요한 DCOM 인터페이스 권한 축소, RestrictRemoteClients 등.
- **방향성 기반 모니터링**: 관리 서버 ↔ 단말의 정상 패턴을 베이스라인화하고 역방향(단말 → 다수 서버) DCOM/RPC를 의심.

## Detection Surface

이 저장소의 detection 가설 중 DCOM이 신호인 부분 (`2026-05-10-pla-dcom-agentless-edr.md` Hypothesis 1, `logman-etw-session-tampering.md`):

- 비관리자 단말에서 다수 서버로 DCOM/RPC 아웃바운드 연결 + 직후 대상 시스템에 Data Collector Set 생성 흔적 (cross-host correlation 필요)
- 의심 프로세스의 `pla.dll` 모듈 로드 (logman 외 경로 시그널)
- 비표준 프로세스(`mmc.exe`, `excel.exe` 등)가 DCOM 호출로 실행
- RPC 엔드포인트 / DCOM 인터페이스 UUID 기반 식별 (PLA 관련 UUID inventory는 Open Question)

대상 시스템 측 보강:

- `Microsoft-Windows-Diagnosis-PLA/Operational` 이벤트와 DCOM 인증 이벤트(`4624` 등)의 시간 상관

## Conflicts / Tensions

- **차단 불가능**: 다수 관리 도구가 DCOM 의존 — 전면 차단은 비현실적. Allow-list 기반 정밀 접근만 가능.
- **DCOM 단독 신호의 낮은 변별력**: DCOM 트래픽만으로는 정상/공격 구분이 어렵고, 대상측 PLA 이벤트와의 cross-host 결합이 필수.

## Open Questions

- PLA 관련 DCOM 인터페이스의 RPC UUID를 inventory하면 탐지 정확도가 얼마나 올라가는가?
- DCOM authentication 레벨(Identify/Impersonate/Delegate) 차이가 PLA 원격 조작 시 어떤 흔적 차이로 나타나는가? — `experiments/2026-05-10-pla-dcom-lab-plan.md` 후속 가능.

## Related Notes

현재 DCOM을 다루는 노트는 1건. §10 Linking Rule(≥3) escape hatch 적용.

- [2026-05-10-pla-dcom-agentless-edr.md](../windows/2026-05-10-pla-dcom-agentless-edr.md)
- _Pending: DCOM lateral movement 객체 카탈로그(MMC20, ShellWindows), DCOM authentication, Hardening(RestrictRemoteClients) 정리 후 추가._

## Related Detections

- [logman-etw-session-tampering.md](../../detections/windows/logman-etw-session-tampering.md) — DCOM은 cross-host 보강 신호로 들어감

## Related Experiments

- [2026-05-10-pla-dcom-lab-plan.md](../../experiments/2026-05-10-pla-dcom-lab-plan.md)
