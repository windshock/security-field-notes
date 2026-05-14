# LLM-Enabled Dynamic Modification Detection Hypothesis

Status: hypothesis
Category: AI Security / Malware / Dynamic Modification / Detection Engineering
Related notes:
- `knowledge/ai-security/2026-05-14-gtig-ai-threat-tracker-ai-enabled-attack-operations.md`
- `knowledge/ai-security/2026-05-13-mdash-oh-my-secuaudit-comparison.md`
- `knowledge/code-analysis/2026-05-13-source-code-indexing-lsp-sourcegraph-security.md`

## Intent

Detect malware or suspicious tooling that uses an LLM, AI gateway, or model-like external service as part of a runtime modification loop.

The core idea is not to detect a specific malware family or generated code style. The goal is to detect the behavior sequence where an external reasoning/generation service influences code, script, command, UI action, or payload mutation shortly before execution.

```text
external model/API call
→ generated or transformed content written locally
→ script/code/payload execution
→ optional persistence, evasion, C2, or UI automation behavior
```

This hypothesis is inspired by GTIG's discussion of PROMPTFLUX, HONESTCUE, CANFAIL, LONGSTREAM, and PROMPTSPY, but it should remain family-agnostic.

## Data Sources

- Proxy / secure web gateway logs
- DNS logs
- EDR network telemetry
- Sysmon or equivalent process and file telemetry
- Script block logging where available
- Cloud egress logs
- Mobile telemetry for accessibility abuse cases
- CI/CD logs for build-time code generation and dependency execution
- Repository / artifact static scan for generated decoy logic and AI integration artifacts

## Suspicious Observables

### AI boundary observables

- Outbound requests to known LLM APIs, AI gateways, relay services, or suspicious OpenAI-compatible proxy endpoints
- API key material, model names, prompt templates, JSON-mode schemas, or tool/action schemas embedded in unexpected binaries or scripts
- Repeated prompts or model calls from hosts that do not normally perform AI-assisted development or automation
- Model responses parsed as commands, code, UI coordinates, or automation actions

### Dynamic modification observables

- Network call followed by local script or payload rewrite
- Generated content written into executable locations, temporary directories, startup paths, profile scripts, scheduled task payloads, browser extension directories, or mobile app private storage
- Execution of newly written or recently modified scripts by interpreters such as Python, PowerShell, wscript/cscript, node, bash, or shell equivalents
- Self-modification behavior where the running process updates its own script body, module, or loader configuration

### Decoy logic observables

- Large volumes of unused helper functions or coherent but irrelevant administrative code
- Repetitive environmental checks unrelated to the program's stated purpose
- Explanatory comments that describe code as filler, inert, placeholder, demo, or unused
- Sudden code-size growth without matching functional change

### Agentic execution observables

- Serialization of UI hierarchy, DOM tree, accessibility tree, screen coordinates, or application state to an external model endpoint
- Structured JSON response consumed as CLICK, SWIPE, COMMAND, EXECUTE, NAVIGATE, WRITE, DOWNLOAD, or similar action types
- Accessibility API abuse followed by model-guided UI automation

## Why It Matters

Traditional malware detection often assumes that the mutation logic is inside the malware sample, packer, builder, or operator toolchain.

LLM-enabled dynamic modification changes this assumption.

```text
Old model:
malware contains the mutation engine

New model:
malware delegates mutation or decision logic to an external model/API/workflow
```

This makes fixed signatures, hashes, and static string matching less durable. The more stable detection surface becomes the behavior around generation, modification, parsing, and execution.

## False Positives

Likely benign cases include:

- AI coding assistants used by developers
- CI jobs that legitimately generate code
- Internal automation that calls LLM APIs for documentation or test generation
- RPA and accessibility automation tools
- Security research sandboxes
- Red team labs with explicit authorization

False positive reduction should use context:

- Is the host expected to call LLM APIs?
- Is the process expected to modify executable content?
- Is the generated content executed immediately?
- Is the execution path user-initiated, CI-controlled, or hidden/persistent?
- Does the process also perform credential access, persistence, C2, or evasion?

## Tuning Ideas

Start with sequence-based scoring rather than one-shot blocking.

Example scoring dimensions:

```text
+ outbound request to LLM or AI gateway
+ response body parsed into code/command/action fields
+ new or modified executable/script file
+ execution within short time window
+ unusual parent process
+ persistence or scheduled execution
+ credential/API key access
+ obfuscation, packed code, or decoy logic indicators
```

Prioritize investigation when multiple signals occur in a short time window.

## Test Plan

- [ ] Build a benign toy harness that mocks an LLM response and rewrites a harmless local script.
- [ ] Capture telemetry for `API-like request → file write → interpreter execution`.
- [ ] Compare hash/string based detection with sequence-based detection.
- [ ] Write a Sigma/KQL draft for Windows process/file/network telemetry.
- [ ] Create a separate mobile-focused hypothesis for accessibility-tree serialization and model-guided UI action.
- [ ] Evaluate false positives from legitimate AI coding assistant and CI code generation workflows.

## Status Notes

This is a hypothesis note, not a production-ready detection rule.

The primary research question is whether the detection surface should be defined around specific AI providers and model endpoints, or around the more general behavioral invariant:

```text
external generation service influences executable behavior at runtime
```

The second approach is likely more durable but requires better context and tuning.
