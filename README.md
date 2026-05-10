# security-field-notes

Security research notes, TODOs, references, and detection hypotheses.

이 저장소는 보안 리서치 과정에서 생기는 자료를 정리하기 위한 개인 지식 저장소입니다. 완성된 글보다 **생각의 흔적, 검증해야 할 가설, 나중에 실험할 TODO, 참고 자료**를 우선 보관합니다.

## Purpose

- 보안 기사, 논문, PDF, PoC, 도구를 읽고 핵심을 요약합니다.
- "새로운 주장"과 "원래 있던 기능의 재해석"을 분리합니다.
- 공격 가능성, 방어 적용성, 탐지 포인트를 함께 기록합니다.
- 나중에 블로그 글, 내부 보고서, 탐지 룰, 실험 코드로 확장할 후보를 관리합니다.

## Repository Structure

```text
.
├── AGENTS.md                       # AI agent / 사람이 자료 추가할 때 따르는 규칙
├── TODO.md                         # 전체 backlog와 다음 작업
├── knowledge/                      # 주제별 정리 노트
│   └── windows/                    # Windows internals, ETW, DCOM, logging
├── references/                     # 외부 자료 메타데이터와 링크
├── experiments/                    # 실험 계획, 재현 로그, 결과
├── detections/                     # 탐지 아이디어, Sigma/KQL/YARA 후보
└── templates/                      # 반복 사용 템플릿
```

새 자료를 추가하거나 ChatGPT/Claude에게 commit을 맡길 때는 `AGENTS.md`의 워크플로와 체크리스트를 따릅니다.

## Note Format

각 리서치 노트는 가능하면 다음 구조를 따릅니다.

1. 핵심 요약
2. 기존에 알려진 것과 새롭게 볼 만한 점
3. 공격자가 악용할 수 있는 이유
4. 방어자가 얻을 수 있는 가치
5. 탐지 포인트
6. TODO / 실험 계획
7. 참고 자료

## Current Topics

- Windows ETW / PLA / DCOM 기반 agentless telemetry
- EDR가 없는 환경에서의 보완적 탐지 구조
- Living-off-the-land 방식의 telemetry 생성/조작
- ETW session tampering과 탐지 가능성

## Caution

이 저장소가 public일 경우, 저작권이 있는 PDF 원문, 민감한 내부 정보, 실제 공격 절차를 그대로 올리지 않습니다. 공개 가능한 요약, 링크, 방어 중심 분석, 탐지 아이디어 위주로 정리합니다.
