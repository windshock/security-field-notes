# PLA (Performance Logs and Alerts)

Status: evolving
Type: Technical Concept
Last reviewed: 2026-05-14

## Short Definition

PLA는 Performance Monitor와 `logman.exe`가 의존하는 Windows 데이터 수집 인프라다. 단위는 **Data Collector Set**이고, 그 안에 Performance Counter / **Trace** / Configuration / Alert / API Trace collector가 들어간다 (`2026-05-10-pla-dcom-agentless-edr.md` §4.1). 이 저장소 관점에서 가장 중요한 사실은 **Trace collector가 ETW trace session과 동일한 객체**라는 점 — 즉 PLA는 "성능 카운터 도구"가 아니라 **ETW의 관리 평면**이다.

## Why It Matters

PLA는 **COM/DCOM 인터페이스를 통해 원격 대상에 Data Collector Set을 생성·수정·시작·중지할 수 있다** (§4.2). 이는 `logman.exe -s <host>` 같은 wrapper로도 노출되고, C++/C#/.NET에서 PLA COM 객체를 직접 호출해도 노출된다. 후자는 process artifact 없이 동일 결과를 만든다.

결과적으로 PLA는 두 가지 의미를 가진다:

1. **공격 표면**: 원격에서 ETW 세션을 만들거나 변조할 수 있는 living-off-the-land 경로 (WMI 대안)
2. **방어 자산**: agent 미설치 환경에서 단기/임시 telemetry 수집 채널

## Offensive Use

- **Agentless 원격 정찰**: 권한 확보 후 대상의 보안 제품, 서비스, registry, 자기 행위의 ETW 흔적을 수집 (`2026-05-10-pla-dcom-agentless-edr.md` §5.1–5.2).
- **WMI 대신 PLA/DCOM**: 많은 조직이 WMI 악용은 본다. PLA/DCOM 기반 collector 조작은 상대적으로 덜 익숙한 경로 (§5.4).
- **Namespace 은닉**: `Session` 기본 범위 대신 `Service\`, `Autosession\`에 collector를 만들면 `logman query -ets`의 기본 조회에서 누락 가능 (§4.3, Hypothesis 3).

## Defensive Use

- **EDR 미도입 자산 보완**: 단기 ETW 수집 지점 생성 (§6).
- **사고 대응 시 원격 관찰**: 단기 trace session 운영 후 회수.
- 운영 시 권한·네트워크·output path·보존·성능 영향 함께 검토 필요 (§6).

## Detection Surface

대상 시스템 측 (가장 강한 신호):

- `Microsoft-Windows-Diagnosis-PLA/Operational` 로그의 Data Collector Set 생성/시작/중지/수정 이벤트 — `logman` 우회 시에도 남는다.
- 새 Data Collector Set 생성, 평소 없던 trace session 시작
- `PerfLogs` 하위 ETL/BLG 파일 생성 급증
- output path가 UNC share (`2026-05-10-pla-dcom-agentless-edr.md` Hypothesis 4 — 원격 수집/반출 의심)
- `Service` 또는 `Autosession` namespace에 낯선 collector

Controller 측:

- `logman.exe` + `stop`/`delete`/`update` (`logman-etw-session-tampering.md`)
- 비-logman 경로: `pla.dll` 모듈 로드 + RPC/DCOM 아웃바운드 연결 조합
- 관리 서버가 아닌 단말이 다수 서버로 DCOM/RPC + 직후 대상측 Data Collector Set 생성 (Hypothesis 1)

## Conflicts / Tensions

- **정상 관리 vs 정찰**: PLA 원격 수집은 정식 관리 기능이므로 베이스라인 없이 단발 행위만으로 구분 불가. "관리 서버 ↔ 단말" 방향성 + 빈도 + 대상 session 민감도의 조합 필요.
- **명령줄 가시성 차이**: `logman.exe` 호출과 PLA COM 직접 호출은 같은 변조의 두 계층이며, 한쪽만 보면 사각지대가 생긴다.

## Open Questions

- **PLA COM API 직접 호출 vs `logman.exe` 호출의 아티팩트 차이**: 대상측 Diagnosis-PLA 이벤트는 동일한가, 차이가 있는가? `experiments/2026-05-10-pla-dcom-lab-plan.md`에서 검증 예정.
- `Service\` / `Autosession\` namespace에 들어간 collector를 정상 inventory에 포함시키려면 어떤 조회 방법이 필요한가? (TODO §Detection Ideas).

## Related Notes

현재 PLA를 다루는 노트는 1건. §10 Linking Rule(≥3) escape hatch 적용.

- [2026-05-10-pla-dcom-agentless-edr.md](../windows/2026-05-10-pla-dcom-agentless-edr.md)
- _Pending: PLA COM API 직접 호출 사례, Data Collector Set XML 스키마, Diagnosis-PLA 이벤트 카탈로그 정리 후 추가._

## Related Detections

- [logman-etw-session-tampering.md](../../detections/windows/logman-etw-session-tampering.md)

## Related Experiments

- [2026-05-10-pla-dcom-lab-plan.md](../../experiments/2026-05-10-pla-dcom-lab-plan.md)
