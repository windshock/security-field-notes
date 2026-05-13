# Copy Fail, Rocky Linux, and Kernel Live Patching Defense Notes

Date: 2026-05-13
Category: linux / kernel-livepatch / detection-engineering
Status: 정리 노트 / 탐지 가설
Source: Conversation synthesis based on public references: Linux kernel livepatch documentation, Rocky Linux CopyFail advisory discussion, TuxCare KernelCare documentation, and rfxn/copyfail

## 1. 핵심 요약

Copy Fail 계열 Linux kernel 취약점 대응을 두고 처음에는 `kpatch`, rootkit 방어 모듈, eBPF, 가상화 계층 탐지, setuid 파일 page cache 차단, page cache 검증, TuxCare/KernelCare 같은 live patch 서비스를 차례로 검토했다.

최종 결론은 단순하다.

- Rocky Linux의 근본 대응은 kernel update 후 reboot이다.
- 재부팅이 어려운 대규모 운영 환경에서는 TuxCare KernelCare 같은 vendor live patch가 현실적인 임시 대응 후보가 될 수 있다.
- 단, live patch 적용 가능 여부는 제품명으로 판단하지 말고, 운영 커널에서 실제 CVE coverage가 확인되는지 검증해야 한다.
- page cache integrity detection은 의미 있는 탐지 계층이지만, kernel patch를 대체하지는 않는다.
- eBPF, LKRG, hypervisor 관측, auditd, rfxn/copyfail 같은 도구는 모두 보조 방어 레이어로 보아야 한다.

운영 판단의 핵심 문장은 다음과 같다.

```text
kernel update + reboot = final fix
vendor live patch = reboot 전 risk window 축소
page-cache integrity check = compromise signal detection
AF_ALG / splice / setuid monitoring = exploit path visibility
```

## 2. 문제 구조

Copy Fail류 취약점은 디스크 파일을 직접 수정하는 전통적인 파일 변조와 다르게 이해해야 한다. 공격자는 kernel bug를 이용해 privilege-relevant file의 page cache 내용을 바꿀 수 있다. 예를 들어 setuid binary, `/etc/passwd`, PAM 설정, sudoers 같은 파일이 실제 디스크에서는 정상이어도, page cache에 올라온 내용이 변조될 수 있다.

이 특성 때문에 일반적인 file integrity monitoring만으로는 충분하지 않을 수 있다.

```text
on-disk file      = 정상
cached read view  = 변조 가능
privileged reader = 변조된 cache를 볼 가능성
```

따라서 방어자는 `파일이 디스크에서 바뀌었는가?`뿐 아니라 `kernel이 지금 보고 있는 cached view가 디스크 원본과 다른가?`도 물어야 한다.

## 3. kpatch와 Rocky Linux

RHEL 환경에서는 Red Hat `kpatch`가 중요한 대응 수단이 될 수 있다. 하지만 Rocky Linux에서 Red Hat kpatch를 그대로 사용하는 모델은 현실적인 운영 표준으로 보기 어렵다. 이유는 `kpatch` 명령 자체보다, 특정 RHEL kernel release에 맞춰 제공되는 live patch module feed와 vendor support model이 핵심이기 때문이다.

Rocky Linux 계열에서는 기본적으로 다음 구조로 봐야 한다.

```text
Rocky Linux official path:
  kernel update
  reboot
  patched kernel boot 확인

Rocky Linux no-reboot interim path:
  third-party live patch candidate 검토
  AF_ALG / exploit path 제한
  page-cache integrity detection
  audit / eBPF / SIEM alert
```

## 4. TuxCare KernelCare의 의미

TuxCare KernelCare는 Rocky/RHEL 계열 서버에서 재부팅 없이 kernel CVE patch를 적용할 수 있는 상용 후보이다. 기술적으로는 실행 중인 kernel에 live patch module을 올리고, 취약한 kernel function의 실행 흐름을 patched function으로 redirect하는 방식이다.

즉, 원리는 넓은 의미에서 kernel hooking이다. 하지만 rootkit과의 차이는 목적과 신뢰 모델이다.

```text
rootkit-style hooking:
  hidden
  unauthorized
  persistence / evasion
  unverified code

vendor live patching:
  visible
  signed / managed
  CVE-specific
  kernel-version-specific
  tested / staged rollout
  unload / rollback path
```

따라서 KernelCare는 `rootkit과 같은 기술적 뿌리를 가진 후킹`이라기보다, `여러 kernel version에 맞춰 빌드·테스트·서명·배포·롤백 가능하게 만든 managed kernel hook patchset`에 가깝다.

운영 검증은 반드시 실제 서버에서 해야 한다.

```bash
uname -r
sudo kcarectl --update
sudo kcarectl --patch-info | grep CVE-2026-31431
sudo kcarectl --uname
```

`kcarectl --patch-info`에 해당 CVE가 보이면 해당 운영 커널에 대해 live patch coverage가 적용된 것으로 볼 수 있다. 보이지 않으면 아직 patch가 없거나, 적용되지 않았거나, 해당 kernel release가 지원되지 않는 상태일 수 있다.

## 5. rfxn/copyfail과 page-cache integrity detection

Ryan MacDonald가 공개한 `rfxn/copyfail`은 Copy Fail 계열 취약점 대응에서 참고할 만한 공개 방어 툴킷이다. 특히 `copyfail-local-check` auditor는 page-cache integrity probe를 포함하고, 문서상 `O_DIRECT read`와 cached SHA-256 hash divergence를 비교하는 탐지 아이디어를 제시한다.

핵심 개념은 다음과 같다.

```text
cached_hash = normal read path result
odirect_hash = O_DIRECT read path result
baseline_hash = known-good reference

cached_hash != odirect_hash
  => page cache divergence 의심

cached_hash != odirect_hash && odirect_hash == baseline_hash
  => disk는 정상이나 cached view가 이상한 상태일 가능성
```

이 방식은 우리가 논의한 `page cache 전후 검증` 아이디어와 매우 가깝다. 다만 현재 rfxn/copyfail은 주로 auditor / periodic detection / mitigation toolkit에 가깝고, 우리가 추가로 생각한 구조는 더 enforcement-oriented이다.

우리가 생각한 확장 구조:

```text
setuid binary execution attempt
  -> fanotify로 FAN_OPEN_EXEC_PERM 계열 이벤트 수신
  -> 대상 inode가 setuid인지 확인
  -> cached read hash와 O_DIRECT hash 비교
  -> divergence가 있으면 execution deny + alert
```

즉 `rfxn/copyfail`은 검증 로직 측면의 선행 구현에 가깝고, `setuid 실행 직전 차단 gate`는 별도 PoC 가치가 있는 확장 아이디어이다.

## 6. eBPF, LKRG, hypervisor 관측의 위치

### eBPF

eBPF는 kernel 안에서 syscall, kprobe, tracepoint, LSM hook 등을 관측할 수 있어 Copy Fail exploit path에 비교적 가까이 붙을 수 있다. 예를 들어 다음 이벤트는 eBPF/Falco/Tetragon 기반 관측 후보가 될 수 있다.

- `socket(AF_ALG, ...)`
- AEAD/authencesn 계열 algorithm 사용
- `splice()` 호출 패턴
- setuid binary 실행
- setuid 실행 직후 shell 실행
- `drop_caches` 호출

하지만 eBPF도 같은 취약 kernel 안에서 동작한다. 따라서 완전한 외부 신뢰 경계는 아니다. 또한 걸어둔 hook만 볼 수 있으며, page cache 변조 그 자체를 항상 자동으로 증명하지는 못한다.

### LKRG

LKRG 같은 runtime guard는 kernel structure 변조나 일부 exploit behavior 탐지에 의미가 있다. 그러나 Copy Fail처럼 page cache / file-backed page corruption 성격이 강한 취약점은 LKRG만으로 충분히 막는다고 전제하면 안 된다.

### OpenStack / vSphere 계층

가상화 계층은 exploit syscall 자체를 잘 보지는 못하지만, 방어 운영에는 도움이 된다.

- VM network quarantine
- vNIC disconnect
- port mirroring / flow telemetry
- snapshot / disk clone
- incident containment
- 대량 reboot scheduling

즉 hypervisor는 `exploit blocker`라기보다 `containment and forensic operation platform`이다.

## 7. setuid 파일을 cache 안 되게 막는 아이디어

`setuid 파일을 page cache에 안 올라가게 만들면 되지 않나?`라는 생각은 자연스럽지만, 일반 Linux 운영 옵션으로는 깔끔하지 않다.

실행 파일은 ELF loader, mmap, page fault, file-backed page mapping을 거쳐 실행된다. 따라서 특정 setuid inode만 page cache를 쓰지 않게 하는 간단한 mount option이나 sysctl은 없다. 이를 커널 hook으로 구현하려면 VFS/MM/page fault 경로에 깊이 들어가야 하고, 운영 위험이 커진다.

더 현실적인 방향은 다음이다.

```text
cache 금지 시도
  -> 운영 적용 비추천

setuid inventory 축소
  -> 가능하고 효과적

nosuid mount 확대
  -> 사용자 쓰기 가능 영역에 유효

AF_ALG/splice exploit path 제한
  -> 직접적 임시 완화

page-cache integrity detection
  -> 강한 침해 신호 탐지

fanotify pre-exec validation
  -> 별도 PoC 가치 있음
```

## 8. 운영 방어 레이어

Copy Fail 계열 취약점에 대한 운영 방어는 단일 기술보다 layered model로 봐야 한다.

| Layer | Purpose | Notes |
|---|---|---|
| Kernel update + reboot | 근본 해결 | Rocky Linux에서는 최종적으로 필요 |
| Vendor live patch | reboot 전 risk window 축소 | KernelCare, kpatch, ksplice 등. OS/vendor별 제약 |
| AF_ALG restriction | exploit primitive 차단 | systemd `RestrictAddressFamilies`, seccomp, BPF LSM, LD_PRELOAD shim 등 |
| Page-cache integrity detection | cache/disk divergence 탐지 | `O_DIRECT` read vs cached read 비교 |
| setuid surface reduction | privilege escalation target 축소 | 불필요한 setuid bit 제거, `nosuid` mount |
| eBPF/auditd/Falco/Tetragon | exploit path visibility | syscall/exec/drop_caches 관측 |
| Hypervisor/cloud controls | containment / forensics | quarantine, snapshot, traffic mirror |

## 9. Minimal verification checklist

Rocky Linux 서버에서 최소한 아래는 확인할 수 있다.

```bash
# OS / kernel inventory
cat /etc/os-release
uname -r

# official patch path
sudo dnf --refresh update 'kernel*'
# reboot required for official kernel fix

# KernelCare path, if licensed
sudo kcarectl --update
sudo kcarectl --patch-info | grep CVE-2026-31431
sudo kcarectl --uname

# rfxn/copyfail auditor path, if used
sudo copyfail-local-check
sudo copyfail-local-check --json

# setuid surface inventory
find /usr /bin /sbin -xdev -perm -4000 -type f -ls

# filesystem hardening candidates
findmnt -T /tmp
findmnt -T /dev/shm
findmnt -T /home
```

## 10. Detection hypotheses

아래는 production rule이 아니라 hypothesis이다. false positive와 workload impact를 lab에서 확인해야 한다.

- Non-root process creating `AF_ALG` socket on server workloads where AF_ALG is not expected.
- `AF_ALG` + AEAD/authencesn + `splice()` sequence within a short time window.
- User session invoking setuid binary followed immediately by shell execution.
- `drop_caches` invocation after suspicious kernel log or setuid activity.
- Divergence between cached read hash and `O_DIRECT` read hash on privilege-relevant files.
- auditd event for socket family 38 with no matching LD_PRELOAD/systemd block event.

## 11. Experiment ideas

### 11.1 Page-cache integrity checker PoC

Goal: verify whether normal read and `O_DIRECT` read hash comparison works reliably on local ext4/xfs filesystems used by Rocky 8/9 servers.

Questions:

- Which filesystems support the required `O_DIRECT` alignment and read behavior?
- Does the check work on `/usr`, `/etc`, and container overlay environments?
- What is the scan cost for `/etc/passwd`, PAM files, sudoers, `/usr/bin/su`, and similar targets?
- Can this be integrated into SIEM without noisy false positives?

### 11.2 fanotify pre-exec validation PoC

Goal: test whether setuid execution can be gated by fanotify permission events and userspace hash validation.

Questions:

- Does Rocky 8/9 expose suitable fanotify permission events for exec?
- Can the daemon validate setuid files quickly enough without breaking login workflows?
- What happens during package updates or RPM transaction windows?
- Can the daemon fail closed only for high-risk files and fail open elsewhere?

### 11.3 KernelCare operational PoC

Goal: verify whether KernelCare actually covers the target CVE for the exact production kernel releases.

Questions:

- Does `kcarectl --patch-info` show the target CVE?
- Does `kcarectl --uname` reflect an effective patched kernel state?
- Does it coexist with EDR, eBPF, kprobe/ftrace-based observability, and virtualization hosts?
- What is the rollback process if a live patch causes instability?

## 12. Final conclusion

The practical answer is not a single magic control.

For Rocky Linux, the final fix remains kernel update plus reboot. If reboot coordination is operationally hard, a vendor live patch such as TuxCare KernelCare can be a rational interim control, provided the exact kernel release and CVE coverage are verified on production-like hosts.

The interesting defensive idea is page-cache-aware detection. `rfxn/copyfail` already shows that `O_DIRECT` read vs cached hash divergence can be used as a page-cache integrity signal. This can be extended into a stronger pre-execution model using fanotify for setuid binaries, but that remains a hypothesis requiring lab validation.

The safe operating model is therefore:

```text
1. Patch or live-patch the kernel path.
2. Reduce the exploit surface.
3. Detect cache/disk divergence.
4. Watch exploit path syscalls and setuid execution chains.
5. Use virtualization/cloud controls for containment and evidence preservation.
6. Reboot into the fixed kernel when the maintenance window is available.
```

## 13. References

- Linux kernel documentation, `Livepatch`.
- Linux kernel documentation, `Kernel module signing facility`.
- TuxCare documentation, `Live Patching Services` and KernelCare CVE verification guidance.
- Rocky Linux forum discussion, CopyFail / CVE-2026-31431 patch availability.
- rfxn/copyfail GitHub repository and handoff brief.
- Linux man-pages, `fanotify(7)` / `fanotify_mark(2)`.
