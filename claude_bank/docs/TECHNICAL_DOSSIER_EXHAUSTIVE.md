# BankX Exhaustive Technical Dossier (Batch-Oriented)

This dossier supersedes the previous high-level memory by providing a **batch-based exhaustive inventory** and deep technical artifacts.

## Deliverables map

1. **Batch 01 — Copilot Backend Domain**
   - File: `claude_bank/docs/dossier/BATCH_01_COPILOT.md`
   - Scope: `claude_bank/app/copilot/app/*`
   - Content: file-by-file summaries (backend APIs, agents, cache, auth, telemetry, config).

2. **Batch 02 — Specialist Agents Domain**
   - File: `claude_bank/docs/dossier/BATCH_02_AGENTS.md`
   - Scope: `claude_bank/app/agents/*`
   - Content: file-by-file summaries for standalone and A2A agent services/handlers.

3. **Batch 03 — Platform Domain (Registry + SDK + Frontend + Infra)**
   - File: `claude_bank/docs/dossier/BATCH_03_PLATFORM.md`
   - Scope: `claude_bank/app/agent-registry/*`, `claude_bank/app/a2a-sdk/*`, `claude_bank/app/frontend/src/*`, `claude_bank/infrastructure/bicep/*`
   - Content: file-by-file summaries for registry, SDK reliability layer, UI, and IaC modules.

4. **Batch 04 — Call-flow diagrams + Config Dependency Matrix**
   - File: `claude_bank/docs/dossier/BATCH_04_USECASE_CALLFLOWS_AND_CONFIG.md`
   - Content:
     - Mermaid call-flow diagrams per use case (UC1/UC2/UC3, continuation fast-path, A2A topology).
     - Config dependency matrix mapping critical runtime env keys to consuming modules.

5. **Batch 05 — Risk/Bug Hotspots with exact file+line references**
   - File: `claude_bank/docs/dossier/BATCH_05_RISK_BUG_HOTSPOTS.md`
   - Content: prioritized hotspot list with exact path+line references.

---

## How this dossier should be used

- For code ownership/onboarding: start at Batch 01/02/03 by folder ownership.
- For runtime troubleshooting: use Batch 04 for flow tracing and config impact analysis.
- For hardening/refactoring backlog: use Batch 05 priority ordering.

---

## Generation notes

- File-by-file summaries were generated from repository source inventory and symbol extraction patterns (Python AST for `.py`, heuristic symbol extraction for `.ts/.tsx/.js`), then curated into batch markdowns.
- This keeps coverage broad while preserving navigability per domain.
