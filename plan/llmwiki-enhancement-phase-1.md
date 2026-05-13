# Plan: Enhancing Security Field Notes to llmwiki (Phase 1)

This plan outlines the first phase of transforming the `security-field-notes` repository into a structured, LLM-friendly knowledge base (llmwiki), focusing on core concepts, synthesis rules, and an AI-optimized entry point.

## Objective
- Improve knowledge distillation and cross-referencing between research notes, detections, and experiments.
- Provide a clear 'map' for LLMs to maintain context across sessions.
- Establish a permanent 'encyclopedia' layer for core security concepts.

## Key Changes

### 1. Foundation: LLM_CONTEXT.md
Create a root-level `LLM_CONTEXT.md` to serve as the primary entry point for AI agents.
- **Content**: Repository purpose, directory schema, core concept map, and high-priority TODOs.
- **Benefit**: Reduces the need for the LLM to scan the entire tree every time a new session starts.

### 2. Protocol: Updating AGENTS.md
Refactor `AGENTS.md` to include explicit rules for knowledge synthesis.
- **Changes**:
    - Update "Folder Selection Rules" to include `knowledge/concepts/` and clarify it is for **durable concept pages**, not one-off article summaries.
    - Add a new section: `## Knowledge Synthesis Rules`.
    - **Linking Rule**: Minimum 3 existing notes. If fewer exist, record `Related notes: none found yet` or link only those found. Avoid weak/artificial links.
    - **Concept Creation**: Propose new concepts before creating them to avoid duplication.
    - Update the AI agent checklist to include synthesis steps.

### 3. Knowledge Layer: Concept Pages
Establish `knowledge/concepts/` as a permanent layer for technical maturity.
- **Initial Files**:
    - `knowledge/concepts/README.md`: Directory index and purpose.
    - `knowledge/concepts/etw.md`: Event Tracing for Windows.
    - `knowledge/concepts/pla.md`: Performance Logs and Alerts.
    - `knowledge/concepts/dcom.md`: Distributed COM.
- **Structure**: Each page will include definition, offense/defense use, detection surface, and open questions.

## Implementation Steps

1. **Update AGENTS.md**: Refactor with new synthesis and linking rules.
2. **Initialize Concepts Directory**: Create `knowledge/concepts/README.md`.
3. **Create Initial Concept Pages**: Draft `etw.md`, `pla.md`, and `dcom.md`.
4. **Draft LLM_CONTEXT.md**: Create the master index for AI agents.
5. **Cross-Link**: Update existing research notes (e.g., `2026-05-10-pla-dcom-agentless-edr.md`) with links to new concept pages.

## Verification & Testing
- **Structure Check**: Verify the new directory and files follow the defined naming conventions.
- **Link Check**: Ensure `LLM_CONTEXT.md` and `AGENTS.md` correctly point to each other and the new folders.
- **Process Check**: Simulate an ingestion task to ensure the new "Knowledge Synthesis Rules" in `AGENTS.md` are clear and actionable.

## Future Considerations (Phase 2)
- `CHANGELOG.md` or a lightweight `LOG.md` for tracking knowledge evolution.
- `knowledge/research-questions.md` for capturing cross-document contradictions.
- Automated linting scripts to find unlinked notes.
