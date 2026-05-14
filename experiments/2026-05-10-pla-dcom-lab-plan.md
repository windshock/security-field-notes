# PLA/DCOM 원격 ETW 수집 실험 계획

Date: 2026-05-10  
Status: planned  
Related notes:
- `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md`
- `knowledge/concepts/pla.md`
- `knowledge/concepts/dcom.md`

## Goal

PLA/DCOM 기반 Data Collector Set 생성과 ETW trace session 조작이 실제 Windows 환경에서 어떤 흔적을 남기는지 확인한다.

## Scope

이 실험은 방어적 가시성 확인을 목적으로 한다. 실제 운영망이 아닌 격리된 Windows lab에서 수행한다.

## Questions

1. local collector 생성 시 어떤 이벤트와 파일 흔적이 남는가?
2. remote collector 생성 시 client와 target 양쪽에 어떤 흔적이 남는가?
3. `logman.exe` 사용과 PLA COM 직접 호출은 EDR/Sysmon 관점에서 어떻게 다르게 보이는가?
4. `Session`, `Service`, `Autosession` namespace별로 기본 조회 가시성이 다른가?
5. ETL output path를 UNC로 지정했을 때 어떤 SMB/RPC 흔적이 남는가?

## Planned Data Sources

- Windows Event Log
- Microsoft-Windows-Diagnosis-PLA/Operational
- Sysmon process creation / file creation / network connection
- EDR process and module telemetry, if available
- Network capture, optional
- File system changes under `C:\PerfLogs`

## High-Level Steps

1. Baseline collection
   - 기존 Data Collector Set 목록 저장
   - 기존 ETW trace session 목록 저장
   - `C:\PerfLogs` baseline 저장

2. Local collector test
   - local trace collector 생성
   - 시작/중지/삭제 시 이벤트와 파일 변화 관찰

3. Remote collector test
   - 별도 Windows host를 대상으로 remote 생성
   - client와 target 양쪽 흔적 비교

4. Namespace visibility test
   - `Session`, `Service`, `Autosession` namespace별 조회 결과 비교

5. Detection candidate extraction
   - command line 기반 탐지
   - target-side artifact 기반 탐지
   - network/RPC 기반 탐지

## Expected Outputs

- 관찰 로그 요약
- 탐지 가능한 이벤트 ID 목록
- false positive 후보
- Sigma/KQL 후보 작성 여부 판단

## Safety Notes

- 운영망에서 수행하지 않는다.
- 관리자 권한 사용 내역을 별도로 기록한다.
- 원격 저장 경로에는 테스트용 share만 사용한다.
- offensive automation이 아니라 visibility 검증에 집중한다.
