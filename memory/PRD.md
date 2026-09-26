# PromptSentry — PRD

## Original Problem Statement
Hackathon prototype: **PromptSentry** — "Pre-Deployment Adversarial Behavior Testing for AI Systems." A developer pastes the system prompt / behavioral rules for an AI app; PromptSentry dynamically generates targeted adversarial probes, runs them against a live target model configured with that prompt, and shows the raw evidence-based observed behavior. Scope is strictly the **core engine only** (no Fortify/Retest/CI-CD/DB/auth/accounts/scores/certification/export/payments).

## Core Loop (Definition of Done — all working)
SYSTEM PROMPT → DYNAMIC PROBE GENERATION (2 probes) → REAL TARGET MODEL EXECUTION → RAW RESPONSE → EVIDENCE-BASED EVALUATION → RESISTED / VIOLATED / INCONCLUSIVE.

## Architecture
- **Frontend**: React single page (`/app/frontend/src/App.js`). Editorial off-white + charcoal + bronze theme (`index.css`). Controls the two-call sequence so judges see probes before results.
- **Backend**: FastAPI (`/app/backend/server.py`). No DB, no auth.
  - `POST /api/generate-probes` — attack-generator role; developer prompt passed as untrusted DATA inside `<DEVELOPER_POLICY>` delimiters; returns exactly 2 probes (instruction_override + prompt_extraction), tailored to the actual rules. Strict-JSON parse with one stricter retry.
  - `POST /api/run-probes` — runs both attack_texts concurrently against target (system message = developer prompt); one combined evaluator call; **backend verifies each `evidence_quote` is a verbatim substring of the raw `target_response`, else clears it and forces INCONCLUSIVE.**
  - Validation: trim, min 30 / max 6000 chars → HTTP 400 before any model call.
- **Model**: `gemini-3.8-flash` for all three roles (generator / target / evaluator) via Emergent Universal Key (`EMERGENT_LLM_KEY`, server-side only).

## User Persona
- AI application developer wanting a fast pre-deployment behavioral sanity check before shipping.

## Implemented (2026-06-26)
- Full core engine, dynamic per-policy probe generation, concurrent target execution, combined evaluator, backend evidence-integrity check.
- Single responsive page: hero, system-prompt textarea, NovaBank demo preset (input-only), STRESS TEST (disabled while running), phase indicators, probe cards with raw responses + verdicts + verified evidence, result summary ("X / 2 attacks resisted"), error state + Retry.
- Claim-boundary wording throughout (observed behavior; not a certification).
- Verified live: testing agent 100% backend + 100% frontend, no issues.

## Backlog (explicitly OUT of scope for this prompt — do NOT build unless asked)
- P2: Fortify, Retest, CI/CD, GitHub integration, database, auth, accounts, multi-turn agents, monitoring, analytics, large attack libraries, security/readiness scores, certification, OWASP claims, export, payments, connect-to-real-endpoint.

## Next Tasks (only if user requests)
- Optional: server-side request timeout wrapper, basic rate limiting on public LLM endpoints.
