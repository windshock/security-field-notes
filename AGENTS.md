# AGENTS

이 문서는 ChatGPT, Claude 등 AI agent가 이 저장소에 자료를 추가하거나 정리할 때 따라야 할 규칙입니다. 사람이 직접 commit할 때도 같은 규칙을 적용합니다.

저장소의 톤과 구조에 대한 배경은 `README.md`를 먼저 읽고 시작합니다. 이 문서는 그 위에서 "새 자료가 들어왔을 때 무엇을 어디에 어떻게 넣는가"만 정의합니다.

## 1. 표준 워크플로

새 PDF, 기사, 논문, 도구 자료가 들어왔을 때:

1. `references/README.md` Reference Index 테이블에 한 줄 추가 (URL, 저자, retrieved 날짜, type, 관련 노트 경로, status)
2. 자료의 성격에 맞는 폴더에 노트 생성 (아래 §2)
3. 새 가설이나 검증이 필요한 항목이 있으면 `TODO.md`의 적절한 섹션(Inbox / Detection Ideas / Writing Ideas)에 항목 추가
4. 탐지 아이디어가 있으면 `detections/<platform>/` 아래 별도 노트 작성, `Status: hypothesis`로 시작
5. 실험이 필요한 항목이 있으면 `experiments/` 아래 실험 계획 노트 작성, `Status: planned`로 시작
6. 관련 `knowledge/concepts/` 페이지가 있으면 핵심 내용을 반영(Related Notes, Detection Surface, Open Questions). 없고 반복 가능성이 높은 개념이면 새 concept page 생성을 사용자에게 먼저 제안 (§10)
7. commit (§6 스타일)

## 2. 폴더 선택 규칙

| 폴더 | 들어가는 것 |
|---|---|
| `knowledge/<platform>/` | 외부 자료의 요약, 해석, 정리 노트. 1차 자료에 가까운 정리. |
| `knowledge/concepts/` | 반복 등장하는 핵심 기술/프로토콜/공격 기법에 대한 **영구적 컨셉 페이지**. 개별 아티클 요약이 아닌, 여러 노트의 증류물. 구조는 §11 참고. |
| `detections/<platform>/` | 탐지 가설, 룰 후보, Sigma/KQL/YARA 초안. |
| `experiments/` | 실험 계획, 절차, 관찰 결과. 날짜 기반. |
| `references/` | 외부 자료의 메타데이터 인덱스. 원문 PDF는 올리지 않음. |
| `templates/` | 반복 사용 템플릿. 새 노트 작성 시 참고. |
| `plan/` | 저장소 구조/지식 베이스 개선 계획 working doc. 노트가 아니라 메타 작업물. 작업 완료 후에도 history로 남김. |

`<platform>`은 현재 `windows`. 새 플랫폼이 필요하면 같은 레벨에 폴더 추가 (`linux`, `macos`, `network`, `cloud` 등).

분류가 애매하면 `knowledge/` 아래에 두고 `Status: inbox`로 표시합니다. 나중에 옮길 수 있습니다.

## 3. 파일명 규칙

- `knowledge/`, `experiments/`: `YYYY-MM-DD-topic-slug.md`
  - 예: `2026-05-10-pla-dcom-agentless-edr.md`
  - 날짜는 자료를 정리한 날(retrieved date 기준)
- `detections/`: 날짜 없이 주제 슬러그
  - 예: `logman-etw-session-tampering.md`
  - 탐지는 시간이 지나도 같은 주제로 갱신되므로 날짜를 파일명에 박지 않음
- slug는 소문자, 단어 사이는 `-`, 영어로 통일

## 4. 노트 구조

### 4.1 knowledge 노트

상단에 frontmatter 없이 본문 메타 4줄:

```markdown
# 노트 제목

Date: YYYY-MM-DD
Category: <플랫폼> / <서브토픽> / <성격(e.g. Detection Engineering)>
Status: 정리 노트 / 탐지 가설 / draft / reviewed
Source: <저자>, "<제목>", <매체>, <발행일>
```

본문은 `templates/research-note-template.md`의 10개 섹션을 기본으로 하되, 자료 성격에 따라 가감할 수 있습니다. 한국어 서술 + 영어 기술 용어 혼용이 표준입니다.

### 4.2 detection 노트

```markdown
# Detection 제목

Status: hypothesis / lab-tested / production-candidate / deprecated
Category: <플랫폼> / <서브토픽> / <성격>
Related note: <knowledge 노트 경로>

## Intent
## Data Sources
## Suspicious Observables
## Why It Matters
## False Positives
## Tuning Ideas
## Test Plan
## Status Notes
```

검증 전 룰은 반드시 `Status: hypothesis`로 시작하고, production-ready로 단정하지 않습니다.

### 4.3 experiment 노트

```markdown
# 실험 제목

Date: YYYY-MM-DD
Status: planned / running / observed / inconclusive / validated / rejected
Related note: <knowledge 노트 경로>

## Goal
## Scope
## Questions
## Planned Data Sources
## High-Level Steps
## Expected Outputs
## Safety Notes
```

## 5. References Index 갱신

새 1차 자료가 들어올 때마다 `references/README.md`를 갱신합니다.

- Reference Index 테이블에 row 한 줄 추가
- 자료별 상세 블록 (URL, Author, Published, Retrieved, Type, Local PDF filename, SHA-256, Related note, Why it matters)을 같은 파일 하단에 추가
- PDF 자체는 repo에 commit하지 않음. 로컬 파일명과 SHA-256만 기록
- 원문 URL이 사라질 수 있는 자료는 별도 private archive에 보관하고, 이곳에는 메타데이터만 남김

## 6. Commit 메시지 스타일

- 영어, 명사구로 시작 (`Add`, `Record`, `Update`, `Expand`, `Fix`)
- 첫 글자 대문자, 마침표 없음
- 한 줄 요약, 70자 이내
- 한 commit은 한 가지 변경에 집중. 새 자료 추가와 무관한 정리는 별도 commit으로 분리

예시 (기존 history):

```
Add note on PLA DCOM agentless ETW collection
Add logman ETW tampering detection hypothesis
Record original Medium URL and PDF metadata
Expand README with repository structure and purpose
```

## 7. Public Repo 안전 규칙

이 저장소가 public이라는 전제로 항상 작성합니다.

- 저작권이 있는 PDF 원문, 논문 본문, 유료 콘텐츠 본문을 그대로 올리지 않음
- 자료에서 인용은 짧게, 출처를 함께 표기
- 실제 공격 절차(working exploit, weaponized PoC)는 올리지 않음. 방어 중심 분석, 가시성 검증, 탐지 아이디어 중심으로 작성
- 내부 시스템 식별자, 호스트명, IP, 사내 자료를 commit하지 않음
- PDF는 iCloud Drive / Google Drive 등 외부 저장소에 두고, repo에는 SHA-256과 URL만 기록

## 8. AI agent를 위한 체크리스트

새 자료를 받아 commit하기 전에 다음을 확인합니다.

- [ ] 폴더와 파일명이 §2, §3 규칙을 따르는가
- [ ] 노트 상단 메타 4줄(Date / Category / Status / Source)이 있는가
- [ ] `references/README.md`에 인덱스 한 줄과 상세 블록이 추가되었는가
- [ ] PDF 원문이나 저작권 자료가 staging에 포함되지 않았는가
- [ ] 새 가설/실험 후보가 `TODO.md`, `detections/`, `experiments/`에 흘러갔는가
- [ ] **관련 기존 노트를 찾아 연결하고(Linking), 관련 Concept 페이지를 갱신했는가(Distillation)**
- [ ] commit 메시지가 §6 스타일을 따르는가

## 9. 이 문서의 갱신

저장소 구조나 노트 포맷이 바뀌면 이 문서를 먼저 갱신합니다. 규칙이 흔들리면 일관성이 깨지고, AI agent가 매번 다른 결과를 만듭니다.

## 10. Knowledge Synthesis Rules

새 자료를 추가하거나 정리할 때, 단순 요약을 넘어 다음을 수행합니다.

1. **연결 (Linking)**: 관련 있는 기존 노트를 **최소 3개** 찾아 연결합니다.
   - 관련 노트가 3개 미만일 경우, 찾은 것만 연결하거나 `Related notes: none found yet`으로 기록합니다.
   - 억지스럽거나 관련성이 낮은 연결은 지양합니다.
2. **증류 (Distillation)**: 개별 리서치 노트의 핵심 내용을 관련 `knowledge/concepts/` 페이지에 반영합니다.
3. **제안 (Proposal)**: 새 컨셉 페이지가 필요하다고 판단되면, 즉시 생성하기보다 사용자에게 먼저 제안하여 중복을 방지합니다.
4. **상태 관리**: 확실하지 않은 정보는 `hypothesis`, `needs-validation` 등으로 명시하여 지식의 신뢰도를 관리합니다.

## 11. Concept Page 구조

`knowledge/concepts/` 아래의 파일은 다음 구조를 권장합니다.

```markdown
# 개념 이름

Status: evolving / stable
Type: Technical Concept
Last reviewed: YYYY-MM-DD

## Short Definition
## Why It Matters
## Offensive Use
## Defensive Use
## Detection Surface
## Conflicts / Tensions (모순점이나 논쟁 사항)
## Open Questions (남겨진 숙제)
## Related Notes
## Related Detections
```
