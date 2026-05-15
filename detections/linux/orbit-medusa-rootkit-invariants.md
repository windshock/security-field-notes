# OrBit / Medusa Rootkit Invariant Detection

Status: hypothesis
Category: linux / rootkit / detection-engineering
Related notes:
- `knowledge/linux/2026-05-15-orbit-medusa-rootkit-reuse-and-evolution.md`
- `knowledge/linux/2026-05-13-copyfail-livepatch-rocky-defense-notes.md`
- `detections/linux/page-cache-integrity-divergence.md`

## Intent

Detect OrBit / Medusa-like Linux rootkit activity by focusing on structural invariants rather than hashes, paths, credentials, or one-off IOCs.

This detection hypothesis is motivated by the OrBit analysis showing repeated reuse of a stable Medusa-derived codebase across multiple years, operators, paths, XOR keys, and build configurations.

## Data Sources

Host-level telemetry:

- File creation and modification events under `/etc`, `/lib`, `/lib64`, `/usr/lib`, `/usr/local/lib`
- `/etc/ld.so.preload` content and metadata
- Package manager verification output
- PAM configuration and module inventory
- Cron file creation events under `/etc/cron.*`
- Process execution telemetry for shell, loader, package manager, and diagnostic tools
- Network flow logs and DNS logs for suspicious fetch behavior

Binary-level telemetry:

- ELF section and symbol metadata
- Dynamic symbol exports
- Embedded strings after simple decoding attempts
- Nested ELF detection inside loader/dropper binaries
- File hash only as supporting context, not primary detection logic

Runtime consistency checks:

- `/proc` process listing versus direct syscall or trusted sensor output
- `/proc/net/*` versus network sensor / flow log output
- directory enumeration via multiple paths or stat methods
- PAM/auth log behavior versus authentication outcomes

## Suspicious Observables

### 1. LD_PRELOAD persistence and system library placement

- `/etc/ld.so.preload` created or modified unexpectedly
- preload file points to a shared object under suspicious system-looking path
- shared object placed under names that imitate legitimate library/config directories
- dynamic linker or loader-related file metadata changes outside package manager activity

### 2. Medusa / OrBit filesystem skeleton

Suspicious co-occurrence of multiple files in the same hidden-looking or system-looking directory:

- `.boot.sh`
- `.logpam`
- `sshpass.txt`
- `sshpass2.txt`
- `.ports`
- `.udp`
- `.pts`
- credential or capture log files such as `remote.txt`, `local.txt`, `sniff.txt`

The parent path can vary. Do not anchor only on `/lib/libseconf/`, `/lib/locate/`, or `/lib/libntpVnQE6mk/`.

### 3. ELF source and build fingerprints

Potentially suspicious symbols or source filenames in unstripped binaries:

- `rkld.c`
- `rknet.c`
- `rkload.c`
- `rkld_so`
- `rkld_so_len`

These should not be used alone as production alerts. They are best treated as high-value enrichment signals when combined with loader behavior, preload persistence, or hook-related exports.

### 4. Hook and export surface

Suspicious exported symbols or function names associated with LD_PRELOAD rootkit behavior:

- file and directory hiding: `readdir`, `readdir64`, `readdir_r`, `readdir64_r`, `stat`, `lstat`, `open`, `fopen`, `read`
- process and command hiding: `execve`, `system`, `popen`
- PAM: `pam_authenticate`, `pam_acct_mgmt`, `pam_open_session`, `pam_get_item`, `pam_sm_authenticate`
- network or pcap: `pcap_loop`, port hiding helpers, `/proc/net/tcp` filtering
- evasion / compatibility: `xread`, `audit_log_acct_message`, `audit_log_user_message`

A single exported libc-compatible name is not enough. Score the set and combine with location, preload, and string evidence.

### 5. Obfuscated string table shape

Candidate binary characteristic:

- contiguous block of single-byte XOR-obfuscated strings
- key may vary, for example `0xA2` or `0xAA`
- decoded candidate set contains threshold count of known paths, filenames, PAM/logging/network artifacts

Detection should try variable single-byte XOR decoding rather than hard-coding one key.

### 6. Nested ELF loader/dropper shape

Suspicious loader/dropper pattern:

- outer ELF contains a second ELF magic at a non-zero offset
- non-stripped builds expose `rkld_so` and `rkld_so_len`
- inner ELF itself matches LD_PRELOAD rootkit fingerprints
- runtime behavior writes the inner payload to disk and configures preload or startup persistence

Nested ELF alone has many false positives. It must be paired with rootkit-specific inner fingerprint, suspicious filesystem skeleton, or preload mutation.

### 7. Infector / cron-based fetch behavior

Potentially suspicious post-2025 style behavior:

- cron file creation under `/etc/cron.hourly/` or nearby locations
- shell pipeline using `wget`/`curl` and direct pipe to shell
- embedded payload extraction followed by rootkit placement
- repeated infection marker checks
- outbound request to unusual infrastructure shortly after binary execution

## Why It Matters

The OrBit / Medusa case shows why hash-only and path-only detection is fragile.

Operators can rotate:

- credentials
- XOR key
- install path
- stripped/unstripped state
- linked source files
- exported symbol set
- dropper packaging

But they are less likely to fully change:

- the rootkit's hook surface
- the loader's filesystem skeleton
- the binary embedding pattern
- the LD_PRELOAD persistence model
- the observable inconsistencies created by hiding files, processes, ports, and authentication events

## False Positives

- Legitimate software that embeds a secondary ELF using `xxd -i` or equivalent
- Security research tools compiled from public rootkit code in a lab
- Compatibility wrappers exporting libc-like names for benign reasons
- Debug or educational binaries containing source filenames such as `rkld.c`
- Package manager scripts that modify loader configuration during legitimate installs
- Container images or malware analysis sandboxes intentionally containing samples

## Tuning Ideas

- Use a weighted score rather than a single indicator.
- Require at least one persistence signal plus one binary-structure signal for high severity.
- Treat unstripped Medusa source filenames as enrichment unless paired with suspicious execution or placement.
- Suppress known malware lab paths and isolated reverse-engineering workstations.
- Track parent process and package manager context for `/etc/ld.so.preload` changes.
- Combine endpoint signals with network flow logs for cron-based payload fetch behavior.
- Compare runtime views from multiple vantage points to identify userland filtering.

## Candidate Scoring Model

High-confidence candidate when at least two of the following groups are true:

1. `/etc/ld.so.preload` or dynamic linker persistence changed unexpectedly.
2. Suspicious shared object under system library path exports multiple hook-like functions.
3. Medusa/OrBit filesystem skeleton appears in one directory.
4. ELF contains Medusa-like symbol/source fingerprints or nested rootkit ELF.
5. Decoded XOR string table matches a threshold of known artifacts.
6. Runtime views diverge across `/proc`, direct sensor, and network telemetry.
7. Cron/dropper behavior writes or fetches a secondary payload.

## Test Plan

1. Build a benign toy LD_PRELOAD library that exports a small subset of hook-like names without hiding behavior.
2. Compile Medusa source in an isolated lab only for static fingerprint comparison, not deployment.
3. Measure false positives for nested ELF detection across benign Linux binaries and packaging tools.
4. Test variable single-byte XOR string extraction against known benign packed binaries.
5. Compare directory listing and `/proc/net/tcp` visibility through multiple userland and trusted external vantage points.
6. Convert stable structural checks into a YARA hypothesis and keep runtime checks as SIEM/EDR correlation logic.

## Status Notes

- This is a hypothesis note, not a production rule.
- The goal is defense-oriented structural detection.
- Do not include working rootkit code, weaponized build instructions, or operational deployment steps in this repository.
