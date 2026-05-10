# Detections

이 폴더는 탐지 아이디어와 룰 후보를 관리합니다.

검증 전 룰은 운영 적용 가능한 탐지로 단정하지 않고 `hypothesis` 상태로 둡니다.

## Detection Note Structure

```markdown
# Detection Title

Status: hypothesis / lab-tested / production-candidate / deprecated

## Intent

무엇을 탐지하려는가?

## Data Sources

- Windows Security Event
- Sysmon
- EDR process telemetry
- Windows Event Log
- ETW / PLA logs
- Network telemetry

## Logic

탐지 조건을 설명합니다.

## False Positives

정상 운영에서 발생 가능한 경우를 기록합니다.

## Test Plan

어떤 실험으로 검증할지 기록합니다.

## References

관련 자료와 노트를 연결합니다.
```

## Initial Detection Backlog

- `logman.exe`를 이용한 ETW session stop/delete/update
- PLA/DCOM 기반 원격 Data Collector Set 생성
- `Service` / `Autosession` namespace의 비정상 collector 생성
- ETL output path가 UNC share로 지정되는 경우
- 관리 서버가 아닌 호스트에서 다수 서버로 RPC/DCOM 접근
