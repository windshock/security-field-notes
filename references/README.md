# References

이 폴더는 외부 자료의 원문 링크, 메타데이터, 읽은 날짜, 관련 노트를 기록합니다.

Public repository에서는 저작권이 있는 PDF 원문을 직접 저장할 때 주의가 필요합니다. 원문 PDF 보관이 필요하면 private storage 또는 private repository를 우선 고려합니다.

## Reference Index

| Date Added | Title | Type | Related Note | Status |
|---|---|---|---|---|
| 2026-05-10 | No Agent, No Problem: Discovering Remote EDR | Article / PDF | `knowledge/windows/2026-05-10-pla-dcom-agentless-edr.md` | summarized |
| 2026-05-13 | Defense at AI speed: Microsoft’s new multi-model agentic security system tops leading industry benchmark | Article / PDF | `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md` | summarized |
| 2026-05-13 | Copy Fail / Kernel Live Patching / Rocky Linux Defense Notes | Conversation synthesis / public references | `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md` | summarized |

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

- URL: not recorded in local PDF metadata; source identified as Microsoft Security Blog from document title/content.
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
