# References

이 폴더는 외부 자료의 원문 링크, 메타데이터, 읽은 날짜, 관련 노트를 기록합니다.

Public repository에서는 저작권이 있는 PDF 원문을 직접 저장할 때 주의가 필요합니다. 원문 PDF 보관이 필요하면 private storage 또는 private repository를 우선 고려합니다.

## Reference Index

| Date Added | Title | Type | Related Note | Status |
|---|---|---|---|---|
| 2026-05-10 | No Agent, No Problem: Discovering Remote EDR | Article / PDF | `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md` | summarized |
| 2026-05-13 | Defense at AI speed: Microsoft’s new multi-model agentic security system tops leading industry benchmark | Article / PDF | `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md` | summarized |
| 2026-05-13 | Copy Fail / Kernel Live Patching / Rocky Linux Defense Notes | Conversation synthesis / public references | `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md` | summarized |
| 2026-05-13 | Source Code Indexing, LSP, and Sourcegraph for Security Analysis | Conversation synthesis / public references | `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md` | summarized |
| 2026-05-14 | GTIG AI Threat Tracker: Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access | Article / PDF / conversation synthesis | `knowledge/ai-security/2026-05-14-gtig-ai-threat-tracker-ai-enabled-attack-operations.md` | summarized |
| 2026-05-15 | OrBit (Re)turns: Tracking an open-source Linux rootkit across four years of forks and deployments | Article / PDF / conversation synthesis | `knowledge/linux/2026-05-15-orbit-medusa-rootkit-reuse-and-evolution.md` | summarized |
| 2026-05-17 | ExploitGym: Can AI Agents Turn Security Vulnerabilities into Real Attacks? | Paper / PDF / conversation synthesis | `knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md` | summarized |

## ExploitGym: Can AI Agents Turn Security Vulnerabilities into Real Attacks?

- URL: https://arxiv.org/abs/2605.11086
- Author: Zhun Wang, Nico Schiller, Hongwei Li, Srijiith Sesha Narayana, Milad Nasr, Nicholas Carlini, Xiangyu Qi, Eric Wallace, Elie Bursztein, Luca Invernizzi, Kurt Thomas, Yan Shoshitaishvili, Wenbo Guo, Jingxuan He, Thorsten Holz, Dawn Song
- Published: 2026-05-11
- Retrieved / archived locally: 2026-05-17
- Type: arXiv paper / uploaded PDF / conversation synthesis
- Local PDF filename: `2605.11086.pdf`
- SHA-256: `c25dbb381d6385c76f77b2be4b14765a1c58eaf1316b270ffafd68ea25b3582f`
- Related note: `knowledge/ai-security/2026-05-17-exploitgym-benchmark-methodology-harness-control.md`
- Related experiment plan: `experiments/2026-05-17-exploitgym-neutral-harness-comparison.md`
- Why it matters: AI agent exploit capability를 대규모 실제 취약점 benchmark로 측정한 중요한 risk-signal 논문이지만, Table 3의 비교는 model capability와 native agent scaffold, tool interface, process prompt, debugging environment, safety/refusal behavior가 섞인 결과로 해석해야 한다. 후속 연구에는 pre-registered third-party neutral harness track과 native deployed stack track을 분리하는 것이 필요하다.
- Copyright / storage note: PDF 원문은 repository에 commit하지 않음. 공개 가능한 요약, 방법론 비판, 실험 설계, 로컬 파일명과 SHA-256만 기록함.

## OrBit (Re)turns: Tracking an open-source Linux rootkit across four years of forks and deployments

- URL: https://intezer.com/blog/orbit-returns/
- Author: Nicole Fishbein
- Published: 2026-05-14
- Retrieved / archived locally: 2026-05-15
- Type: Intezer blog article / Safari-generated PDF archive / conversation synthesis
- Local PDF filename: `OrBit (Re)turns- Tracking an open-source Linux rootkit across four years of forks and deployments - Intezer.pdf`
- SHA-256: `fe818b672752cace94b4aeb292361869ae74c112e4f218169019584be851c350`
- Related note: `knowledge/linux/2026-05-15-orbit-medusa-rootkit-reuse-and-evolution.md`
- Related detection hypothesis: `detections/linux/orbit-medusa-rootkit-invariants.md`
- Why it matters: OrBit이 독자 개발 malware family라기보다 Medusa 공개 LD_PRELOAD rootkit 코드베이스를 여러 operator가 build option, hook set, install path, credential, loader/dropper 구조 단위로 재조합한 사례라는 점을 보여준다. Hash IOC보다 build pipeline invariant, filesystem skeleton, XOR string table shape, nested ELF loader structure, runtime view divergence를 중심으로 탐지해야 한다는 결론을 제공한다.
- Copyright / storage note: PDF 원문은 repository에 commit하지 않음. 공개 가능한 요약, 해석, 탐지 가설, 로컬 파일명과 SHA-256만 기록함.

## GTIG AI Threat Tracker: Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access

- URL: https://cloud.google.com/blog/topics/threat-intelligence/ai-vulnerability-exploitation-initial-access
- Author: Google Threat Intelligence Group
- Published: 2026-05-12
- Retrieved / archived locally: 2026-05-14
- Type: Google Cloud Blog article / Safari-generated PDF archive / conversation synthesis
- Local PDF filename: `Adversaries Leverage AI for Vulnerability Exploitation, Augmented Operations, and Initial Access | Google Cloud Blog.pdf`
- SHA-256: `840cb22fc3560d85f53dbbf960f130b91f2064dabf441a6990901ccd129c6dff`
- Related note: `knowledge/ai-security/2026-05-14-gtig-ai-threat-tracker-ai-enabled-attack-operations.md`
- Related detection hypothesis: `detections/ai-security/llm-enabled-dynamic-modification.md`
- Why it matters: AI가 취약점 탐색, exploit 개발, 악성코드 Dynamic Modification, 자율 공격 오케스트레이션, LLM 접근 인프라 산업화, AI supply chain 공격에 어떻게 들어오고 있는지 구조적으로 보여준다. 특히 Figure 3은 "AI가 모든 취약점 탐색을 대체한다"는 오해에 대응하고, Dynamic Modification은 변형 로직이 malware 내부에서 외부 model/API/workflow로 이동한다는 탐지 관점의 가설을 만들기 좋다.
- Copyright / storage note: PDF 원문은 repository에 commit하지 않음. 공개 가능한 요약, 해석, 탐지 가설, 로컬 파일명과 SHA-256만 기록함.

## Source Code Indexing, LSP, and Sourcegraph for Security Analysis

- URL: https://microsoft.github.io/language-server-protocol/
- URL: https://sourcegraph.com/
- URL: https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html
- URL: https://clangd.llvm.org/design/indexing
- URL: https://github.com/sourcegraph/scip
- Author: windshock / ChatGPT conversation synthesis with public reference links
- Published: Various
- Retrieved: 2026-05-13
- Type: Conversation synthesis / public reference links / code analysis notes
- Related note: `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`
- Why it matters: 대형 코드베이스 보안 분석에서 LSP를 정의/참조/심볼/호출 후보 수집용 전처리 인덱싱 계층으로 활용하고, Sourcegraph를 다중 저장소 검색 및 LLM/보안 분석용 context 수집 계층으로 활용하는 구조를 정리함.
- Copyright / storage note: 공개 가능한 대화 요약과 공개 reference link만 기록함. 내부 코드, 사내 시스템 식별자, 민감 정보는 기록하지 않음.

## No Agent, No Problem: Discovering Remote EDR

- URL: https://jonny-johnson.medium.com/no-agent-no-problem-discovering-remote-edr-8ca60596559f
- Author: Jonathan Johnson
- Published: 2025-06-06
- Retrieved / archived locally: 2026-05-10
- Type: Medium article / Safari-generated PDF archive
- Local PDF filename: `No Agent, No Problem- Discovering Remote EDR | by Jonathan Johnson | Medium.pdf`
- SHA-256: `238f0735c5fa65994185229d94bcf5abc26191e750df6492f1b85c7d9075da04`
- Related note: `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md`
- Why it matters: PLA/DCOM을 이용한 원격 ETW Data Collector Set 생성과 agentless telemetry 관점의 방어/공격 해석을 다룸.
- PDF metadata check: uploaded PDF metadata contains title, creator `Safari`, producer `iOS Version 26.4.2 (Build 23E261) Quartz PDFContext`, creation/modification date, but no embedded original URL was found in metadata, annotations, attachments, or extracted text.

## Defense at AI speed: Microsoft’s new multi-model agentic security system tops leading industry benchmark

- URL: https://www.microsoft.com/en-us/security/blog/2026/05/12/defense-at-ai-speed-microsofts-new-multi-model-agentic-security-system-tops-leading-industry-benchmark/
- Author: Taesoo Kim
- Published: 2026-05-12
- Retrieved / archived locally: 2026-05-13
- Type: Microsoft Security Blog article / Safari-generated PDF archive
- Local PDF filename: `Defense at AI speed- Microsoft’s new multi-model agentic security system tops leading industry benchmark | Microsoft Security Blog.pdf`
- SHA-256: `ca8f44fd61fa7d503abb7768f343075b04eb1ffe88ccb67122649c506b34c1ea`
- Related note: `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- Why it matters: Microsoft MDASH의 multi-model agentic vulnerability discovery pipeline을 Oh my secuaudit의 reproducible security audit workflow와 비교할 수 있는 기준점을 제공함.
- Copyright / storage note: PDF 원문은 repository에 commit하지 않음. 공개 가능한 요약과 비교 노트만 보관.

## Copy Fail / Kernel Live Patching / Rocky Linux Defense Notes

- URL: https://github.com/rfxn/copyfail
- URL: https://docs.tuxcare.com/live-patching-services/
- URL: https://kernel.org/doc/html/next/livepatch/livepatch.html
- URL: https://forums.rockylinux.org/t/copyfail-cve-2026-31431-patches-now-available-for-rocky-linux/20422
- Author: Multiple public sources; local synthesis by windshock / ChatGPT
- Published: Various
- Retrieved: 2026-05-13
- Type: Conversation synthesis / public reference links / defense notes
- Related note: `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md`
- Related detection hypothesis: `detections/linux/page-cache-integrity-divergence.md`
- Why it matters: Rocky Linux 환경에서 Copy Fail 계열 kernel 취약점 대응을 kernel update + reboot, vendor live patch, page-cache integrity detection, AF_ALG/splice monitoring, fanotify pre-exec validation 관점으로 정리함.
- Copyright / storage note: 내부 위키 내용, 운영 서버 수량, exploit code, 민감한 호스트 정보는 repository에 기록하지 않음. 공개 가능한 방어 중심 분석과 링크만 보관.

## Entry Template

```markdown
### Title

- URL:
- Author:
- Published:
- Retrieved:
- Type:
- Related notes:
- Why it matters:
- Copyright / storage note:
```

## Notes

- 링크가 사라질 수 있는 자료는 별도 private archive에 보관하고, 이곳에는 메타데이터를 남깁니다.
- 분석 노트에는 원문 내용의 장문 복사보다 요약, 비판, 탐지 아이디어를 남깁니다.
