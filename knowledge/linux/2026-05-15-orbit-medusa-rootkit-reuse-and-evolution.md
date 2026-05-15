# OrBit, Medusa, and Linux Rootkit Reuse

Date: 2026-05-15  
Category: linux / rootkit / detection-engineering  
Status: 정리 노트 / 탐지 가설 / conversation synthesis  
Source: Nicole Fishbein, "OrBit (Re)turns: Tracking an open-source Linux rootkit across four years of forks and deployments", Intezer, 2026-05-14; windshock / ChatGPT discussion notes, 2026-05-15

## 1. 한 줄 요약

OrBit 사례는 Linux rootkit 기술이 매번 새로 개발되는 것이 아니라, 공개 또는 반공개 코드베이스를 build option, install path, credential, hook set, loader architecture 단위로 재조합해 장기간 재사용되는 현실을 보여준다.

핵심 결론은 다음과 같다.

```text
Rootkit primitive는 성숙했고,
platform과 운영 방식은 계속 이동한다.
```

즉, 루트킷 기술이 완전히 정체된 것은 아니다. 다만 실제 공격 현장에서는 검증된 공개 코드베이스를 가져와 환경에 맞게 줄이고, 숨기고, 경로와 credential을 바꾸는 방식이 훨씬 경제적이다.

## 2. 오늘 논의 흐름

1. Intezer의 OrBit 분석 PDF를 읽고 핵심을 요약했다.
2. OrBit이 독자 개발 악성코드라기보다 `ldpreload/Medusa` 공개 rootkit 코드베이스와 강하게 연결된다는 점을 확인했다.
3. Medusa 같은 유명 rootkit repository와 rootkit 모음이 더 있을 것이라는 가설을 검토했다.
4. 공개 rootkit 재사용이 많다면 Linux rootkit 기술 자체가 정체된 것인지 질문했다.
5. 최종적으로 rootkit의 기본 primitive는 오래전부터 성숙했지만, eBPF, io_uring, UEFI, cloud, hypervisor, EDR telemetry 환경으로 은닉 위치가 이동하고 있다는 결론을 냈다.

이 노트는 그 대화의 결과를 저장소 규칙에 맞춰 field note 형태로 정리한 것이다.

## 3. 기존에 알려진 것

Linux rootkit의 기본 목표는 오래전부터 크게 변하지 않았다.

- 파일과 디렉터리 숨기기
- 프로세스 숨기기
- 네트워크 포트와 connection 숨기기
- 인증 credential 탈취 또는 인증 결과 조작
- 로그, audit, forensic trace 약화
- persistence 유지
- 보안 도구와 관리자 명령의 관측 결과 변조

기술 계층은 대략 세 부류로 볼 수 있다.

| 계층 | 대표 방식 | 특징 |
|---|---|---|
| Userland rootkit | `LD_PRELOAD`, `ld.so.preload`, libc/PAM hook | 배포와 수정이 쉽고, 오래된 공개 코드가 많음 |
| Kernel rootkit | LKM, syscall/ftrace/kprobe hook | 강력하지만 커널 버전, 보안 설정, crash risk에 민감 |
| Modern / lower layer | eBPF, io_uring abuse, UEFI bootkit, hypervisor/cloud implant | 탐지면이 더 복잡하지만 운영 난도와 비용도 높음 |

대표적으로 공개 연구나 공개 repository에서 반복적으로 언급되는 계열은 Medusa, Azazel, Jynx/Jynx2, Umbreon, Diamorphine, Reptile, Suterusu, eBPF 기반 rootkit 연구들이다. 이 목록은 공격 절차를 위한 것이 아니라, hook surface와 탐지 primitive를 분류하기 위한 출발점으로 다룬다.

## 4. OrBit / Medusa에서 새롭게 볼 만한 점

### 4.1 OrBit은 단일 operator의 고유 malware로 보기 어렵다

Intezer 분석의 핵심은 OrBit이 여러 해 동안 독자적으로 발전한 단일 악성코드라기보다, Medusa라는 공개 LD_PRELOAD rootkit 코드베이스를 여러 operator가 각자 빌드하고 설정한 결과에 가깝다는 점이다.

운영자가 바꾼 것은 주로 다음이다.

- XOR key
- 설치 경로
- SSH backdoor username / password
- export set
- hook set
- loader / dropper packaging
- 일부 string과 logging behavior

이 관점에서는 hash IOC보다 build pipeline invariant가 훨씬 중요하다.

### 4.2 Lineage A와 Lineage B

OrBit 샘플은 크게 두 계열로 정리된다.

| 계열 | 의미 | 방어 관점 |
|---|---|---|
| Lineage A | full build. PAM, pcap, TCP port hiding, `xread`, auditd evasion, `pam_sm_authenticate` 등 고급 기능 포함 | hook surface가 넓어 탐지 가능성도 넓지만 은닉 능력도 큼 |
| Lineage B | lite build. PAM, pcap, TCP port hiding 등을 제거한 축소형 | footprint를 줄이려는 환경 특화 빌드로 해석 가능 |

Lineage B는 2024년 이후 새 샘플이 확인되지 않은 것으로 정리된다. 따라서 retire 되었거나 full branch로 흡수되었을 가능성이 있다.

### 4.3 새 기능처럼 보였던 것의 상당수는 이미 Medusa source에 있었다

`xread`, `audit_log_*`, `pam_sm_authenticate`, `pcap_loop` 같은 기능은 OrBit 운영자가 새로 작성했다기보다 Medusa의 `src/rknet.c`에 이미 존재하던 기능을 빌드에 포함한 것으로 해석된다.

따라서 OrBit의 연도별 변화는 malware engineering innovation이라기보다 다음에 가깝다.

```text
Makefile / link target change
+ credential rotation
+ install path rotation
+ hook set selection
+ loader/dropper packaging change
```

### 4.4 2025년 infector 구조는 운영상 의미가 크다

기존 OrBit은 passive implant 성격이 강했다. 공격자가 SSH backdoor로 들어오는 방식이 중심이었다.

2025년 계열에서는 `infector -> dropper -> rootkit` 구조와 cron 기반 외부 payload fetch가 관찰된다. 이 변화는 OrBit 계열이 처음으로 직접적인 외부 command/payload channel 성격을 갖기 시작했다는 점에서 중요하다.

방어 관점에서는 rootkit 본체뿐 아니라 다음도 같이 봐야 한다.

- dropper 내부 embedded ELF
- 감염 marker
- cron persistence
- remote payload fetch
- 재감염 구조

### 4.5 attribution은 codebase와 operator를 분리해야 한다

공개 rootkit이 여러 actor에게 재사용되면, 같은 rootkit family 이름만으로 특정 actor를 단정하기 어렵다.

OrBit / Medusa 계열은 문서상 UNC3886, BLOCKADE SPIDER, RHOMBUS 관련 인프라 등 서로 다른 cluster와 연결된다. 따라서 분석 질문을 둘로 분리해야 한다.

```text
이 codebase는 무엇인가?
이 build/configuration/deployment를 수행한 operator는 누구인가?
```

## 5. 공격자 관점

공격자에게 공개 rootkit 재사용은 합리적이다.

- 이미 동작이 검증된 codebase를 쓸 수 있다.
- 커널 crash나 서비스 장애 같은 운영 리스크를 줄일 수 있다.
- path, key, credential, export set만 바꿔도 단순 hash 기반 탐지를 회피할 수 있다.
- 공개 repository 기반이면 attribution을 흐리기 쉽다.
- 필요한 기능만 link하거나 제거해 target environment에 맞는 footprint를 만들 수 있다.

특히 rootkit은 보통 initial access용 exploit이 아니라 post-compromise persistence와 stealth 도구다. 공격자는 새롭고 불안정한 기술보다 오래 버틸 수 있는 안정성을 선호한다.

## 6. 방어자 관점

방어자는 rootkit을 hash IOC 중심으로만 추적하면 안 된다.

더 중요한 것은 다음과 같은 invariant다.

1. build pipeline invariant
2. hook surface invariant
3. filesystem skeleton invariant
4. loader 구조 invariant
5. runtime view divergence

예를 들어 OrBit / Medusa 계열에서는 다음이 hash보다 더 유용하다.

- `/etc/ld.so.preload` 또는 dynamic linker 관련 변경
- 비정상 shared object가 `/lib`, `/lib64` 등 system path에 배치되는지
- `.boot.sh`, `.logpam`, `sshpass.txt`, `sshpass2.txt`, `.ports` 같은 파일 세트가 같은 디렉터리에 공존하는지
- ELF symbol table에 `rkld.c`, `rknet.c`, `rkload.c` 흔적이 남아 있는지
- `xread`, `pam_sm_authenticate`, `pcap_loop`, `audit_log_*` 같은 export/hook 후보가 있는지
- 단일 byte XOR string table 구조가 있는지
- loader 내부에 secondary ELF가 embedded 되어 있는지
- `/proc`, network, process, filesystem view가 서로 불일치하는지

## 7. Rootkit 기술 진전에 대한 결론

오늘 논의의 최종 결론은 다음과 같다.

```text
루트킷 기술은 정체된 것이 아니라,
primitive는 성숙했고 platform은 이동했다.
```

### 7.1 primitive는 오래됐다

파일 숨기기, 프로세스 숨기기, 포트 숨기기, PAM hook, 로그 회피, `/proc` 조작, libc hook, syscall hook 같은 기본 아이디어는 오래됐다. 이 점만 보면 기술 진전이 없어 보일 수 있다.

### 7.2 platform은 바뀌고 있다

하지만 공격 표면은 계속 이동한다.

```text
userland LD_PRELOAD
-> kernel module / syscall / ftrace / kprobe
-> eBPF / io_uring / kernel feature abuse
-> UEFI / boot chain
-> cloud node / container / hypervisor / EDR telemetry layer
```

따라서 최신성은 새로운 hook 이름 하나가 아니라, 같은 은닉 primitive가 어느 platform으로 옮겨졌는지에서 나온다.

### 7.3 운영 자동화가 붙고 있다

OrBit 2025 infector 사례처럼 기존 passive rootkit에 external payload fetch, reinfection, cron persistence가 결합되면 위험도가 달라진다. 기술 primitive는 오래됐어도 운영 구조가 자동화되면 방어 난도는 올라간다.

## 8. 탐지 포인트

| 탐지면 | Observable | 해석 |
|---|---|---|
| Persistence | `/etc/ld.so.preload`, linker patch, suspicious `.so` path | LD_PRELOAD 계열 persistence |
| Filesystem skeleton | `.boot.sh`, `.logpam`, `sshpass.txt`, `sshpass2.txt`, `.ports` co-occurrence | Medusa/OrBit 계열 loader 흔적 후보 |
| ELF metadata | `rkld.c`, `rknet.c`, `rkload.c`, `rkld_so`, `rkld_so_len` | Medusa build pipeline fingerprint 후보 |
| Export / hook surface | `xread`, `pam_sm_authenticate`, `pcap_loop`, `audit_log_*` | advanced hook set 포함 여부 |
| String obfuscation | variable single-byte XOR table + known plaintext threshold | key rotation에 강한 signature 후보 |
| Auth stack | PAM module/config integrity, suspicious auth result behavior | credential harvesting 또는 service-side auth manipulation |
| Runtime divergence | `/proc` view, direct syscall view, network flow view 불일치 | userland/rootkit filtering 가능성 |
| Dropper / infector | embedded ELF, cron fetch, reinfection marker | 2025 이후 운영 자동화 계열 탐지 후보 |

## 9. TODO / 실험 계획

- [ ] 공개 Linux rootkit repository를 기능별 hook surface로 분류한다.
- [ ] Medusa default build와 `rknet.c` 포함 build의 static fingerprint 차이를 방어 연구용 isolated lab에서 비교한다.
- [ ] `LD_PRELOAD` 계열에서 `readdir/read/stat/open/PAM` hook이 만드는 관측 불일치를 안전한 toy binary로 재현한다.
- [ ] Medusa/OrBit 계열의 XOR string table을 key-independent threshold signature로 잡는 YARA 후보를 설계한다.
- [ ] nested ELF + Medusa-specific symbol/inner fingerprint 조합의 false positive를 측정한다.
- [ ] eBPF/io_uring/UEFI/cloud rootkit 계열을 같은 primitive taxonomy에 매핑한다.

실제 공격 절차, weaponized PoC, 운영 가능한 악성 payload는 이 저장소에 기록하지 않는다. 실험은 static fingerprint, toy hook, detection validation 중심으로 제한한다.

## 10. Related notes

- `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md`
- `detections/linux/page-cache-integrity-divergence.md`
- `detections/linux/orbit-medusa-rootkit-invariants.md`

Rootkit 전용 기존 노트가 아직 충분하지 않아 AGENTS §10의 3개 linking rule은 escape hatch를 적용한다. 이 주제는 `linux-rootkit-reuse-and-hook-surface` concept candidate로 유지하고, 관련 노트가 더 쌓이면 concept page 승격을 검토한다.

## 11. 참고 자료

- Nicole Fishbein, "OrBit (Re)turns: Tracking an open-source Linux rootkit across four years of forks and deployments", Intezer, 2026-05-14.
- `ldpreload/Medusa`: https://github.com/ldpreload/Medusa
- `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md`
- `detections/linux/page-cache-integrity-divergence.md`
