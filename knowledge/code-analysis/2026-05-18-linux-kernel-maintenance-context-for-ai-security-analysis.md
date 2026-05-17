# Linux kernel 유지보수 구조가 AI 보안 분석에 주는 교훈

Date: 2026-05-18
Category: Code Analysis / AI Agent Workflow / Security Engineering
Status: draft / conversation synthesis
Source: windshock / ChatGPT conversation synthesis, "Linux kernel maintenance as agent-readable context for security analysis", 2026-05-18; public reference links to Linux kernel process docs and `windshock/oh-my-secuaudit` repository notes

Related notes:
- `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`
- `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- `knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md`

Related repository:
- `windshock/oh-my-secuaudit`

## 1. 한 줄 요약

Linux kernel은 모든 지식을 하나의 완성된 문서에 넣으려 하지 않고, subsystem, maintainer, mailing list, commit history, Documentation을 결합한 **분산된 기억 시스템**으로 유지보수된다. 이 구조는 인간에게는 복잡하지만, 충분한 context를 따라갈 수 있는 AI agent에게는 오히려 분석 가능한 흔적을 많이 제공한다.

## 2. 배경

오늘 논의의 출발점은 "Linux kernel 개발자들은 유지보수를 위해 아키텍처를 특수하게 가져가는가, 아니면 다른 노하우가 있는가"였다.

처음에는 kernel 개발자가 코드를 이해하는 현장감을 가상의 드라이버 코드로 설명했다. 이후 지적에 따라 실제 kernel 개발자는 단순한 `grep + 노트` 방식만 쓰는 것이 아니라 다음 도구와 기록을 함께 사용한다는 방향으로 정정했다.

```text
git grep / rg
cscope / ctags
clangd / LSP
Bootlin 또는 Elixir 같은 코드 브라우저
git log -L / git blame / git show
MAINTAINERS
Documentation/process
mailing list archive
```

핵심 결론은 다음이었다.

```text
커널 코드를 이해한다는 것은 함수 하나를 읽는 일이 아니다.
그 함수가 어떤 subsystem, callback path, object lifetime, lock, userspace ABI, maintainer 관례, 과거 commit 이유 속에 놓여 있는지를 복원하는 일이다.
```

이 관찰은 곧바로 AI 보안 분석 구조로 이어진다.

## 3. 기존에 알려진 것

### 3.1 Kernel 내부 API와 userspace ABI의 분리

Linux kernel은 kernel 내부 interface를 안정적인 public API처럼 고정하지 않는다. 내부 API는 더 나은 구조와 보안 개선을 위해 바뀔 수 있다. 대신 mainline tree 안에 들어온 코드는 API 변경 시 함께 고친다.

반대로 syscall, `/proc`, `/sys`, ioctl 등 userspace가 의존하는 interface는 매우 보수적으로 다룬다.

```text
kernel 내부:
  바꿀 수 있다.
  단, tree 전체의 사용처를 같이 고친다.

kernel ↔ userspace 경계:
  함부로 바꾸면 안 된다.
  이미 외부 프로그램이 의존하는 계약이다.
```

### 3.2 Subsystem과 maintainer 구조

Kernel은 한 사람이 전체를 이해하는 방식으로 유지되지 않는다. `drivers`, `fs`, `net`, `mm`, `arch`, `security` 같은 영역이 있고, 각 영역에는 maintainer, mailing list, tree, 관례가 있다.

개발자는 보통 다음 순서로 움직인다.

```text
수정할 파일 확인
→ 어느 subsystem인지 확인
→ MAINTAINERS에서 담당자와 list 확인
→ subsystem 문서와 기존 코드 스타일 확인
→ 과거 commit과 mailing list 논의 확인
→ 작은 patch 작성
→ self-contained commit message 작성
→ review를 통해 합의
```

### 3.3 Documentation은 전체 지식의 원천이 아니라 압축된 운영 매뉴얼

문서화가 구조를 만든 것이 아니라, 오래된 협업 구조가 먼저 생겼고 문서화는 그 구조를 신규 개발자가 따라갈 수 있게 고정한다.

따라서 kernel의 기억은 한 곳에 있지 않다.

```text
코드
  현재 진실

commit history
  왜 그렇게 바뀌었는지의 시간축

mailing list
  논쟁, 거절된 대안, maintainer 선호

Documentation
  반복되는 규칙과 합의의 압축본

MAINTAINERS
  책임 영역과 사람의 지도
```

이 구조는 단순한 문서화보다 강하다. 문서가 낡아도 코드, history, review record가 남고, 반복되는 합의만 공식 문서로 승격된다.

## 4. 새롭게 볼 만한 점

### 4.1 Kernel은 AI agent에게 "쉬운 코드베이스"가 아니라 "따라갈 흔적이 많은 코드베이스"다

Linux kernel은 거대하고 어렵다. macro, callback, config option, architecture 분기, lock, RCU, memory barrier, lifetime 문제가 많다.

하지만 AI agent 입장에서 보면 다음 장점이 있다.

```text
1. subsystem이 분석 범위를 줄여준다.
2. MAINTAINERS가 책임 영역과 reviewer를 알려준다.
3. Documentation이 공식 규칙을 제공한다.
4. git history가 변경 이유를 제공한다.
5. mailing list가 거절된 대안과 논쟁 맥락을 제공한다.
6. commit message가 reasoning을 강제한다.
7. patch 단위가 작아 review 가능하다.
8. checkpatch, sparse, KUnit, selftests 같은 검증 도구가 있다.
```

즉 AI agent가 무작정 전체 코드를 읽는 것이 아니라, 다음처럼 context를 좁혀 갈 수 있다.

```text
파일
→ subsystem
→ 공식 API 계약
→ 기존 사용 패턴
→ caller/callee path
→ object lifetime
→ 관련 commit history
→ mailing list decision
→ patch candidate
→ self-review checklist
```

### 4.2 모든 것을 문서화하지 못하므로, 지식을 분산 저장한다

오늘 대화의 핵심 문장은 다음에 가깝다.

```text
모든 것을 문서화할 수 없다.
그래서 영역을 나누고,
각 담당 영역마다 스타일과 관례를 만든다.
논의는 mailing list에 남기고,
반복되는 합의는 최종 문서로 정리한다.
```

이것은 문서 부족의 타협이 아니라, 대규모 지식 시스템의 현실적인 운영 방식이다.

### 4.3 AI agent 시대에는 "context가 있는 repository"가 강해진다

AI agent가 잘못된 분석을 하는 이유 중 하나는 context 부족이다. 반대로 repository가 다음 요소를 제공하면 agent가 훨씬 안정적으로 분석한다.

```text
- 영역별 책임과 범위
- output contract
- schema compatibility rule
- decision log
- review checklist
- known false positive pattern
- evidence path
- prior run delta
- accepted-risk reason
```

이것은 단순히 AI를 돕기 위한 장식이 아니다. 사람에게도 유지보수 가능한 구조다.

## 5. 공격자 관점

공격자나 offensive automation도 구조화된 repository에서 이득을 얻을 수 있다. 특히 다음 조건이 있으면 취약점 탐색 속도가 올라간다.

```text
- entrypoint와 sink가 잘 분리되어 있음
- subsystem별 API 계약이 명확함
- 과거 bug fix commit과 regression history가 풍부함
- test와 PoC artifact가 남아 있음
- architecture diagram과 trust boundary가 정리되어 있음
```

따라서 방어자는 "문서화하면 공격자도 좋아진다"는 불편함을 인정해야 한다. 하지만 대안은 지식을 숨기는 것이 아니라, public repo에는 weaponized exploit과 민감 정보를 제외하고, 방어 중심의 context와 검증 가능한 evidence를 남기는 것이다.

## 6. 방어자 관점

보안 분석 repository는 Linux kernel의 구조에서 다음을 빌릴 수 있다.

```text
skills/ 또는 modules/      = subsystem
SKILL.md 또는 PROFILE.md   = subsystem handbook
schemas/                  = 내부 ABI / output contract
references/               = 설계 문서와 운영 문서
templates/                = canonical example
tools/scripts/            = dev tools
reporting_summary.json    = handoff contract
architecture review       = integration maintainer
security-testing-as-code  = reproducible evidence tree
```

특히 `oh-my-secuaudit`에는 다음 요소를 추가하면 좋다.

```text
MAINTAINERS.md
  skill별 owner, scope, downstream consumer, review expectation

CONTRIBUTING.md
  patch/change protocol, one logical change rule, commit message template

docs/contracts/
  finding contract, reporting summary contract, architecture handoff contract

docs/decisions/
  왜 이런 구조를 선택했는지 남기는 ADR-style 기록

docs/review-checklists/
  schema change, skill change, new skill, report output checklist

.github/ISSUE_TEMPLATE/
  mailing-list-like structured discussion
```

핵심 비유는 다음이다.

```text
Skill은 도구가 아니라 subsystem이다.
Output schema는 ABI다.
Finding/evidence/handoff는 commit history다.
Architecture review는 maintainer-level synthesis다.
Decision log는 mailing list의 압축본이다.
```

## 7. 탐지 포인트

이 노트의 탐지 포인트는 malware 탐지가 아니라, **AI-assisted security audit repository의 품질 신호와 위험 신호**에 가깝다.

- Data source:
  - repository layout
  - `README.md`
  - `CONTRIBUTING.md`
  - `MAINTAINERS.md`
  - `schemas/`
  - `docs/decisions/`
  - PR template
  - issue template
  - prior finding artifacts

- Observable:
  - output schema가 있는가
  - finding이 evidence path와 연결되는가
  - high/critical finding이 remediation requirement로 승격되는가
  - false positive / accepted-risk / not-confirmed 상태가 구분되는가
  - schema 변경 시 migration note가 있는가
  - 같은 finding이 다음 cycle에서 delta로 추적되는가

- Suspicious condition:
  - 모든 분석이 단일 report markdown에만 존재함
  - PoC/evidence/request/commit hash가 report와 분리되어 사라짐
  - finding status가 `confirmed`와 `suspected`를 구분하지 않음
  - architecture context 없이 scanner output만 누적됨
  - schema 변경이 downstream consumer를 고려하지 않음

- False positive:
  - 작은 개인 실험 repo는 모든 구조가 필요하지 않다.
  - 초기 prototype 단계에서는 decision log보다 빠른 실험이 우선일 수 있다.
  - 단, 반복 감사와 agent handoff가 필요해지는 순간 contract와 history가 중요해진다.

## 8. 실험 계획

- [ ] `oh-my-secuaudit`에 `MAINTAINERS.md` 초안을 추가하고 skill별 scope / consumer / output contract를 정리한다.
- [ ] `CONTRIBUTING.md`에 kernel-style patch principle을 반영한다.
- [ ] `docs/contracts/finding-contract-v1.md`와 `docs/contracts/reporting-summary-contract-v1.md`를 만든다.
- [ ] `docs/decisions/0001-skill-as-subsystem.md`를 작성한다.
- [ ] `docs/decisions/0002-output-schema-as-abi.md`를 작성한다.
- [ ] PR template에 `Contract impact`, `Affected skills`, `Validation evidence`, `Migration note` 항목을 추가한다.
- [ ] AI agent에게 변경 PR을 리뷰하게 하고, contract break / missing evidence / downstream impact를 얼마나 잘 찾는지 측정한다.

## 9. 결론 강도

- [x] 확인된 사실: Linux kernel은 subsystem, maintainer, MAINTAINERS, Documentation/process, mailing list, patch review, commit history를 결합해 대규모 유지보수를 수행한다.
- [x] 확인된 사실: kernel 내부 API는 안정성을 약속하지 않고, userspace interface는 보수적으로 다루는 구조가 유지보수 철학의 핵심이다.
- [x] 대화 기반 합성: 이 구조는 AI agent가 필요한 context만 따라가며 분석하기 좋은 형태다.
- [ ] 실험 필요: `oh-my-secuaudit`에 MAINTAINERS / contracts / decisions / checklists를 추가했을 때 실제 agent review 품질이 좋아지는지 측정해야 한다.
- [ ] 추가 검증 필요: security audit repo에서 어느 수준의 문서화가 생산성을 올리고, 어느 순간부터 overhead가 되는지 empirical 기준이 필요하다.

## 10. 참고 자료

- Linux kernel, `Documentation/process/submitting-patches.rst`
- Linux kernel, `Documentation/process/stable-api-nonsense.rst`
- Linux kernel, `Documentation/process/maintainer-handbooks.rst`
- Linux kernel, `MAINTAINERS`
- `windshock/oh-my-secuaudit`, README and skill documents
- `security-field-notes`, `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`
- `security-field-notes`, `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- `security-field-notes`, `knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md`
