# Batch 05 — Risk/Bug Hotspots (with exact file+line references)

This batch lists code-level hotspots that can affect security, reliability, or maintainability.

## 1) PII/token-adjacent console behavior in frontend bootstrap

- File: `claude_bank/app/frontend/src/index.tsx`
- Exact refs: lines **28-56** iterate token entries from `localStorage`; lines **48-49** log user/roles.
- Why risky: startup path inspects token cache and logs identity claims to browser console. This can leak sensitive metadata in shared/dev environments.

## 2) Verbose production debug prints with message payloads in chat routing

- File: `claude_bank/app/copilot/app/api/chat_routers.py`
- Exact refs: lines **200-207**, **236-256**, **260-279** print full/partial message history and A2A payloads.
- Why risky: contains user prompts and possibly personal finance details in plain logs; can violate least-logging principle.

## 3) Feature-flag inconsistency between app startup and container mode selection

- File A: `claude_bank/app/copilot/app/main.py`
- Exact refs: lines **68-70** and **105-107** use `AND` for `_a2a_mode_enabled`.
- File B: `claude_bank/app/copilot/app/config/container_foundry.py`
- Exact refs: line **302** uses `OR` for selecting A2A supervisor path.
- Why risky: partial flag enablement can produce mixed mode behavior where startup/prewarm assumptions differ from runtime supervisor wiring.

## 4) Supervisor import gate checks only one feature flag

- File: `claude_bank/app/copilot/app/config/container_foundry.py`
- Exact refs: lines **30-36** import A2A supervisor based solely on `USE_A2A_FOR_ACCOUNT_AGENT`.
- Why risky: configuration intent for transaction/payment-only A2A enablement may not match imported supervisor implementation.

## 5) Authentication setup endpoint logs tenant/client identifiers

- File: `claude_bank/app/copilot/app/api/auth_routers.py`
- Exact refs: lines **19-20** print tenant/client values; line **24** logs missing-config path.
- Why risky: even non-secret identifiers can become noisy leakage and aid reconnaissance in logs.

## 6) Hardcoded cloud URL fallbacks for MCP endpoints in cache client

- File: `claude_bank/app/copilot/app/cache/mcp_client.py`
- Exact refs: lines **324**, **333**, **343**, **353** fallback to external Azure Container Apps URLs.
- Why risky: implicit fallback can mask misconfiguration and cause unexpected cross-environment calls.

## 7) CORS policy allows all methods/headers but only localhost origins

- File: `claude_bank/app/copilot/app/main.py`
- Exact refs: lines **47-55** allow localhost origins and wildcard methods/headers.
- Why risky: secure for local dev, but deployment portability requires explicit environment-bound origin management to avoid drift.

## 8) Legacy + v2 + v3 payment agent variants can drift

- File tree: `claude_bank/app/agents/payment-agent`, `payment-agent-a2a`, `payment-agent-v2-a2a`, `payment-agent-v3-a2a`.
- Exact refs example: `claude_bank/app/agents/payment-agent-v3-a2a/main.py` lines **1-5** document 2-tool flow; other variants retain alternate execution models.
- Why risky: behavior divergence across versions increases support/test burden and can create inconsistent transfer semantics.

## 9) Startup warmup invokes private/internal methods

- File: `claude_bank/app/copilot/app/main.py`
- Exact refs: line **117** calls `supervisor._build_af_agent(...)`; lines **122-129** invoke agent build on startup.
- Why risky: reliance on internal method contracts increases fragility when agent-framework versions evolve.

## 10) Large instruction prompts embedded directly in code

- File: `claude_bank/app/copilot/app/agents/foundry/supervisor_agent_foundry.py`
- Exact refs: lines **72+** begin long in-code instructions/routing rules.
- Why risky: prompt/routing drift is harder to review/test than versioned external policy artifacts.

---

## Prioritization suggestion

1. Stop logging payload/token-adjacent data (`index.tsx`, `chat_routers.py`, `auth_routers.py`).
2. Unify A2A flag logic (`AND` vs `OR`) across startup/container paths.
3. Externalize long prompt policy and add tests for routing and confirmation behavior.
4. Replace hardcoded MCP fallbacks with explicit environment validation/fail-fast rules.
