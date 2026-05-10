# PLA/DCOM 기반 원격 ETW 수집과 Agentless EDR 해석

Date: 2026-05-10  
Category: Windows / ETW / DCOM / Detection Engineering  
Status: 정리 노트 / 탐지 가설  
Source: Jonathan Johnson, "No Agent, No Problem: Discovering Remote EDR", Medium, 2025-06-06

## 1. 한 줄 요약

이 자료의 핵심은 "원격 Windows 이벤트 수집이 가능하다"가 아니다. 그것은 원래 가능했다. 핵심은 **PLA(Performance Logs and Alerts)의 DCOM 인터페이스를 이용해 원격 시스템에 ETW trace session과 Data Collector Set을 만들고, 조작하고, 그 결과를 수집할 수 있다**는 점이다.

즉, 단순 로그 조회가 아니라 **원격 telemetry instrumentation**에 가깝다.

## 2. 왜 처음에는 당연해 보이는가

Windows 운영을 해본 사람에게 다음 기능은 익숙하다.

- Windows Event Forwarding(WEF)
- Event Viewer 원격 조회
- WinRM / PowerShell Remoting
- Performance Monitor 원격 성능 카운터 수집
- `logman`을 이용한 trace/session 관리

그래서 "원격에서 이벤트를 수집할 수 있다"는 설명만 들으면 새롭지 않다. 실제 차이는 Event Log 수집과 ETW trace session 생성의 차이에 있다.

## 3. 일반 이벤트 수집과 이 글의 차이

| 구분 | 일반적인 Windows 이벤트 수집 | PLA/DCOM 기반 원격 ETW 수집 |
|---|---|---|
| 주 대상 | 이미 기록된 Windows Event Log | 임의의 ETW provider / trace session |
| 방식 | WEF, Event Viewer, WinRM, SIEM agent | PLA DCOM interface, Data Collector Set |
| 결과 | Event Log channel 중심 | ETL trace file, raw ETW telemetry |
| 성격 | 로그 조회/전달 | 원격 관찰 지점 생성/조작 |
| 방어 가치 | 표준 운영 로그 수집 | agentless telemetry 보완 수단 |
| 공격 가치 | 기존 로그 열람 | trace session 생성, 중지, 변조, 은닉 가능성 |

## 4. 문서의 핵심 기술 요소

### 4.1 PLA / Data Collector Set

PLA는 Performance Logs and Alerts의 약자로, Windows Performance Monitor와 `logman` 등이 사용하는 기반 기능이다. Data Collector Set은 여러 collector를 묶는 단위이며, ETW trace session도 collector의 한 형태로 다룰 수 있다.

주요 collector 유형은 다음과 같다.

- Performance Counter
- Trace
- Configuration
- Alert
- API Trace

### 4.2 DCOM을 통한 원격 조작

자료의 핵심 포인트는 PLA가 COM/DCOM 인터페이스를 통해 로컬뿐 아니라 원격 대상에서도 data collector set을 열거, 생성, 수정, 시작, 중지할 수 있다는 것이다.

즉, 별도 에이전트를 설치하지 않고도 원격 시스템에 ETW 기반 수집 지점을 만들 수 있다.

### 4.3 Namespace 문제

문서에서 흥미로운 부분은 namespace이다. 일반적인 `logman query -ets`는 기본적으로 보이는 범위가 제한적일 수 있으며, `Session` 이외의 namespace에 생성된 collector는 기본 조회에서 덜 드러날 수 있다.

관찰해야 할 namespace 후보:

- `Session`
- `Service`
- `Autosession`

이 차이는 공격자에게는 은닉성, 방어자에게는 가시성 사각지대가 될 수 있다.

## 5. 공격자가 악용할 수 있는 이유

이 기술은 초기 침투용이라기보다 **post-exploitation** 단계에서 가치가 있다.

### 5.1 에이전트 없이 원격 정찰

공격자는 이미 권한을 얻은 뒤 다음 정보를 알고 싶어 한다.

- 어떤 보안 제품이 설치되어 있는가
- 어떤 서비스와 프로세스가 동작 중인가
- 어떤 registry key와 configuration이 존재하는가
- 특정 공격 도구 실행 시 어떤 telemetry가 남는가

PLA/ETW 기반 수집은 별도 바이너리를 올리지 않고도 일부 관찰을 가능하게 한다.

### 5.2 ETW provider를 이용한 낮은 수준의 telemetry 관찰

ETW provider는 일반 이벤트 로그보다 더 세밀한 정보를 제공할 수 있다. 공격자는 이를 이용해 방어자가 어떤 telemetry를 볼 수 있는지 추정하거나, 자신의 행위가 어떤 흔적을 남기는지 관찰할 수 있다.

### 5.3 보안 trace session 중지 또는 변조

더 위험한 지점은 trace session 조작이다. 공격자가 충분한 권한을 가진 경우 다음과 같은 시도가 가능할 수 있다.

- 기존 trace session 중지
- provider 제거
- security descriptor 변경
- output path 변경
- collector set 삭제 또는 수정

이는 EDR 자체를 직접 무력화한다기보다, **ETW 기반 탐지 파이프라인을 약화시키는 행위**로 볼 수 있다.

### 5.4 WMI가 아닌 경로

많은 조직은 WMI 악용을 탐지한다. 반면 PLA/DCOM 기반 data collector 조작은 상대적으로 덜 익숙할 수 있다. 따라서 공격자는 WMI 대신 PLA/DCOM을 이용해 living-off-the-land 방식의 원격 telemetry 조작을 시도할 수 있다.

## 6. 방어자가 얻을 수 있는 가치

이 기술은 공격자에게만 유용한 것이 아니다. EDR agent 설치가 어렵거나 부담스러운 서버에서 보완적 telemetry 수집 구조로 사용할 수 있다.

가능한 방어 활용:

- EDR 미설치 서버에 대한 제한적 ETW 수집
- 특정 provider 중심의 임시 telemetry 수집
- 사고 대응 시 단기 원격 관찰 지점 구성
- WEF/SIEM/Sentinel과 결합한 보완적 탐지

다만 운영 환경에서는 권한, 네트워크, 파일 저장 경로, 로그 보존, 성능 영향을 함께 검토해야 한다.

## 7. EDR/SIEM 탐지 관점

### 7.1 쉽게 보이는 경우

`logman.exe`를 사용하는 경우는 비교적 탐지하기 쉽다.

관찰 포인트:

- `logman.exe` 프로세스 실행
- 원격 호스트 지정
- trace session create/start/stop/delete/update 계열 명령
- `-ets` 사용
- `Service\\`, `Autosession\\` 문자열
- parent process가 PowerShell, cmd, script host, 비표준 관리 도구인 경우

### 7.2 어려운 경우

공격자가 C++/C#/.NET 코드에서 PLA COM 인터페이스를 직접 호출하면 `logman.exe`가 나타나지 않는다. 이 경우 EDR은 다음 행위를 조합해서 봐야 한다.

- 의심 프로세스의 COM/DCOM 사용
- `pla.dll` 관련 모듈 로드
- RPC/DCOM 네트워크 연결
- 대상 시스템의 Data Collector Set 생성/시작 흔적
- ETL/BLG 파일 생성
- PLA Operational 로그 변화

### 7.3 대상 시스템 측면 탐지

- 새 Data Collector Set 생성
- 평소 없던 ETW trace session 시작
- output path가 비정상 경로 또는 UNC path
- `PerfLogs` 하위 ETL/BLG 파일 생성 급증
- 보안 관련 trace session stop/delete/update
- `Service` 또는 `Autosession` namespace에 낯선 collector 생성

## 8. 탐지 가설

### Hypothesis 1

관리 서버가 아닌 일반 사용자 단말에서 다수 서버로 DCOM/RPC 연결이 발생하고, 직후 대상 서버에 Data Collector Set이 생성되면 의심할 수 있다.

### Hypothesis 2

`logman.exe`가 보안 관련 ETW session을 stop/delete/update하면 방어 회피 가능성이 있다.

### Hypothesis 3

`Service` 또는 `Autosession` namespace에 생성된 낯선 collector는 기본 조회에서 누락될 수 있으므로 별도 inventory가 필요하다.

### Hypothesis 4

ETL output path가 UNC share를 가리키면 원격 수집 또는 데이터 반출 시나리오를 의심할 수 있다.

## 9. 지금 단계의 결론

이 자료는 "Windows 원격 이벤트 수집이 새롭다"는 글이 아니다. 이미 존재하던 `logman`, PerfMon, PLA, ETW 기능을 **agentless EDR / 원격 telemetry / 공격 표면** 관점으로 다시 해석한 글이다.

실무적으로 중요한 포인트는 다음이다.

- Event Log 수집과 ETW trace session 생성은 다르다.
- PLA/DCOM은 원격에서 trace session을 만들거나 조작할 수 있다.
- `logman.exe` 기반 탐지는 쉽지만, PLA COM 직접 호출은 탐지 난이도가 높다.
- 공격자는 정찰, telemetry 관찰, trace session tampering에 활용할 수 있다.
- 방어자는 EDR 미설치 자산에 대한 보완적 telemetry 수집에 활용할 수 있다.

## 10. 다음 TODO

- [ ] Windows lab에서 local collector 생성 흔적 수집
- [ ] 원격 collector 생성 시 대상 시스템 이벤트 로그 확인
- [ ] `Microsoft-Windows-Diagnosis-PLA/Operational` 이벤트 조사
- [ ] `logman.exe` 기반 탐지 룰 초안 작성
- [ ] PLA COM API 직접 호출 시 EDR telemetry 비교
- [ ] Data Collector Set inventory 스크립트 작성 여부 검토
- [ ] public repo에 올려도 되는 방어 중심 실험 코드 범위 결정

## 11. 참고 자료

- Jonathan Johnson, "No Agent, No Problem: Discovering Remote EDR", Medium, 2025-06-06
- Microsoft Docs: Performance Logs and Alerts
- Microsoft Docs: logman
- SigmaHQ rules related to logman and ETW session tampering
