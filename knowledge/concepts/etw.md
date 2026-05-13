# ETW (Event Tracing for Windows)

Status: evolving
Type: Technical Concept
Last reviewed: 2026-05-14

## Short Definition

ETW는 Windows의 커널/유저 모드 트레이싱 인프라이다. Provider(이벤트 생성), Controller(세션 생성·시작·중지), Consumer(실시간 또는 ETL 파일 소비)로 구성되며, 단순 Event Log 채널보다 한 단계 낮은 수준의 raw telemetry를 다룬다.

## Why It Matters

이 저장소의 첫 출발점은 "원격 이벤트 수집이 새롭다"가 아니라 **Event Log 수집과 ETW trace session 생성의 차이**다 (`2026-05-10-pla-dcom-agentless-edr.md` §3). ETW 세션은 *읽기*가 아니라 *생성/조작*이 가능한 관찰 지점이므로, 공격자에게는 변조 표적이 되고 방어자에게는 agentless 수집 채널이 된다.

## Offensive Use

- **Session tampering**: 보안 솔루션이 의존하는 trace session을 `stop`/`delete`/`update`하여 ETW 기반 탐지 파이프라인을 약화시킨다. EDR 바이너리 자체를 건드리지 않고도 가시성만 깎을 수 있다 (`2026-05-10-pla-dcom-agentless-edr.md` §5.3).
- **Telemetry observation**: 자신의 공격 행위가 어떤 provider에서 어떻게 보이는지 실시간 관찰하여 도구를 조율한다.
- **Namespace 은닉**: `Service\` 또는 `Autosession\` namespace에 collector를 만들면 기본 `logman query -ets` 조회 범위 밖에서 동작할 수 있다 (§4.3).

## Defensive Use

- **Agentless 보완 수집**: EDR 미설치 자산에 대해 PLA를 통한 원격 trace session 생성으로 단기 telemetry를 확보한다 (§6).
- **사고 대응 시 임시 관찰 지점**: 특정 provider 중심으로 짧은 기간 trace session을 운영.

## Detection Surface

대상 세션 (`logman-etw-session-tampering.md` Suspicious Observables에서 가져옴):

- `EventLog-*` 채널 트레이스
- `SYSMON TRACE`, `SysmonDnsEtwSession`
- `Circular Kernel Context Logger`
- 낯선 `Service\` / `Autosession\` collector

행위 (controller 측):

- `logman.exe` 프로세스 + 명령줄 토큰: `stop` / `delete` / `update` / `-ets` / 원격 대상 `-s`
- 비-Logman 경로 (PLA COM API 직접 호출): `pla.dll` 모듈 로드 + RPC/DCOM 연결 조합
- parent process가 PowerShell, cmd, script host, 비표준 관리 도구인 경우 가중치 상승

대상 시스템 측:

- `Microsoft-Windows-Kernel-EventTracing/Analytic` 등 ETW 자체 트레이스 (`logman` 외 경로 탐지 시 핵심)
- `PerfLogs` 하위 ETL/BLG 파일 생성 급증
- 비정상 경로 / UNC path output

## Conflicts / Tensions

- **합법 관리 vs 변조**: 관리자 디버깅이나 백업 도구가 일시적으로 session을 조작하는 케이스와 공격자의 회피 행위가 명령줄 수준에서는 동일하게 보인다. False positive 사례는 `logman-etw-session-tampering.md` §False Positives 참고.
- **`logman.exe` 가시성 vs COM 직접 호출 사각지대**: `logman` 기반 탐지는 쉽지만 PLA COM API를 직접 호출하면 동일 행위가 process artifact 없이 일어난다 (`2026-05-10-pla-dcom-agentless-edr.md` §7.2). 둘은 같은 위협의 다른 가시성 계층이며, 룰을 분리해서 작성해야 한다.

## Open Questions

- 특정 EDR이 사용하는 독자적 ETW provider 이름을 어떻게 효율적으로 inventory할 것인가?
- PLA COM API 직접 호출 시 남는 cross-host 아티팩트(모듈 로드, RPC UUID, 대상측 Diagnosis-PLA 이벤트)의 신뢰도는 어느 정도인가? — `experiments/2026-05-10-pla-dcom-lab-plan.md`에서 검증 예정.

## Related Notes

현재 ETW 관련 노트는 1건. §10 Linking Rule(≥3) escape hatch 적용 (`Related notes: only 1 found yet`).

- [2026-05-10-pla-dcom-agentless-edr.md](../windows/2026-05-10-pla-dcom-agentless-edr.md) — agentless EDR 해석과 session tampering 가설
- _Pending: ETW provider enumeration, kernel-mode patching, Sealighter/Krabs 사례 등이 들어오면 추가._

## Related Detections

- [logman-etw-session-tampering.md](../../detections/windows/logman-etw-session-tampering.md)
