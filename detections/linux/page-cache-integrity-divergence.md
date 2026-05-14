# Page Cache Integrity Divergence Detection

Status: hypothesis
Category: linux / kernel / detection-engineering
Related notes:
- `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md`

## Intent

Detect possible page cache corruption by comparing cached read results with an `O_DIRECT` read path for privilege-relevant files.

This hypothesis is motivated by Copy Fail-like bugs where on-disk files remain unchanged while file-backed pages in memory can be corrupted. In such a case, a normal cached read and a disk-oriented direct read may disagree.

## Data Sources

- Local file read via normal POSIX I/O
- Local file read via `O_DIRECT`, where supported by filesystem and alignment constraints
- Known-good baseline hash
- File metadata: path, device, inode, size, mtime, ctime
- Package metadata, such as RPM ownership and verification output
- Optional event triggers:
  - auditd syscall events
  - eBPF/Falco/Tetragon events
  - kernel log messages
  - fanotify execution permission events

## Suspicious Observables

- `cached_hash != odirect_hash` for privilege-relevant files
- `cached_hash != odirect_hash && odirect_hash == baseline_hash`
- Divergence shortly after suspicious AF_ALG or splice activity
- Divergence shortly before or after setuid binary execution
- Divergence followed by `drop_caches` activity

Candidate files:

- `/etc/passwd`
- `/etc/group`
- `/etc/shadow` where root access is available
- `/etc/sudoers`
- `/etc/pam.d/su`
- `/etc/pam.d/sshd`
- `/etc/pam.d/login`
- `/etc/pam.d/system-auth`
- `/etc/pam.d/password-auth`
- `/etc/nsswitch.conf`
- `/etc/ssh/sshd_config`
- `/usr/bin/su`
- `/usr/bin/passwd`
- selected setuid-root binaries discovered by inventory

## Why It Matters

Traditional file integrity monitoring usually answers whether the file on disk changed. Copy Fail-like bugs make a different question important: whether the page cache view used by privileged code differs from the disk-backed view.

A divergence does not by itself prove root compromise, but it is a high-signal condition because normal production systems should rarely show mismatched cached and direct read hashes for stable privilege-relevant files.

## False Positives

- Package update or RPM transaction in progress
- File replacement race during configuration management
- Filesystem-specific `O_DIRECT` behavior or alignment failure
- Overlayfs, NFS, FUSE, or container rootfs differences
- Legitimate administrative changes not yet reflected in baseline
- Reading through different mount namespaces

## Tuning Ideas

- Start with read-only monitoring on a small set of stable files.
- Store `(device, inode)` in addition to path to handle hardlinks and file replacement.
- Suppress alerts during known package update windows.
- Compare metadata before and after hashing to detect race conditions.
- Treat `cached_hash != odirect_hash` as high severity only when file metadata is stable.
- Use audit/eBPF AF_ALG and splice events as triggers for immediate focused scans.

## Test Plan

1. Test `O_DIRECT` read hashing on Rocky 8/9 ext4 and xfs filesystems.
2. Validate behavior on `/etc` and `/usr` paths.
3. Measure scan cost for the minimal privilege-relevant file set.
4. Compare behavior under RPM update, normal configuration management, and reboot.
5. Verify whether rfxn/copyfail `copyfail-local-check` emits useful JSON for SIEM ingestion.
6. Build a small userspace prototype that reports:
   - `cached_hash`
   - `odirect_hash`
   - `baseline_hash`
   - file metadata
   - result classification
7. Optional: test fanotify pre-exec gating for setuid binaries in a lab.

## Status Notes

This is a detection hypothesis, not a production-ready rule. It should be validated on representative Rocky/RHEL kernels and filesystems before any enforcement action is considered.

A possible later extension is a `fanotify`-based execution gate:

```text
FAN_OPEN_EXEC_PERM event
  -> target is setuid-root binary
  -> run cached/O_DIRECT comparison
  -> allow if consistent
  -> deny + alert if divergent
```

This extension is higher risk because it may affect login, sudo, password change, package update, and recovery workflows.
