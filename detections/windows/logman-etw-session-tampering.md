# logman 기반 ETW Session Tampering 탐지 가설

Status: hypothesis  
Category: Windows / ETW / Defense Evasion  
Related notes:
- `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md`
- `knowledge/concepts/etw.md`
- `knowledge/concepts/pla.md`

## Intent

`logman.exe`를 이용해 ETW trace session을 중지, 삭제, 수정하는 행위를 탐지한다. 특히 보안 telemetry와 관련된 session을 조작하는 경우 방어 회피 가능성이 있다.

## Data Sources

- EDR process creation telemetry
- Sysmon Event ID 1, if enabled
- Windows process creation logging, if enabled
- Command line audit logs

## Suspicious Observables

- Image: `logman.exe`
- Command line contains one or more of:
  - `stop`
  - `delete`
  - `update`
  - `-ets`
  - remote target option such as `-s`
- Command line references suspicious or sensitive sessions:
  - `EventLog-*`
  - `SYSMON TRACE`
  - `SysmonDnsEtwSession`
  - `Circular Kernel Context Logger`
  - unfamiliar `Service\\` or `Autosession\\` collector names

## Why It Matters

`logman.exe` is a legitimate Windows administrative tool. That makes it useful for normal operations, but also useful for living-off-the-land style tampering. If an attacker has sufficient privilege, trace sessions used by monitoring or security tooling may be stopped or altered.

## False Positives

- Performance troubleshooting by administrators
- Backup or monitoring tools that temporarily manage trace sessions
- Developer diagnostics
- Incident response activity

## Tuning Ideas

- Allowlist known admin hosts and scheduled maintenance windows
- Prioritize non-admin workstations targeting servers
- Prioritize commands that reference security-related sessions
- Combine with target-side collector/session deletion events

## Test Plan

- [ ] Capture normal `logman query -ets` behavior
- [ ] Capture local `logman stop` behavior in lab
- [ ] Capture remote `logman -s` behavior in lab
- [ ] Compare command-line only detection with target-side artifact detection
- [ ] Review false positives from admin tooling

## Status Notes

This is a detection hypothesis. It should not be treated as production-ready until tested against a Windows lab and normal administrative workflows.
