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
- **Build 1 (core engine)**: dynamic per-policy probe generation (2 probes), concurrent target execution, combined evaluator, backend evidence-integrity check. Single responsive page (off-white/bronze), NovaBank demo preset (input-only), audit button, phase indicators, probe cards with raw responses + verdicts + verified evidence, result summary, error + Retry. Testing agent: 100% backend + 100% frontend.
- **Build 2 (Fortify + same-attack Retest)**:
  - `POST /api/fortify-retest` — gated on verified VIOLATED (original evidence quote must be supported by the original response, else 400). Fortifier LLM proposes `fortified_prompt` + `change_summary` (forbids trivial/useless fixes). Retest replays the EXACT original attack_text (byte-for-byte equality invariant gate) against the fortified prompt in a fresh execution, then dynamic evaluation.
  - Retest evidence rules (normalized: trim, strip outer quotes, case-insensitive): VIOLATED requires verified quote else INCONCLUSIVE ("Evaluator failed to isolate verifiable violation evidence."); RESISTED never demoted for missing quote; INCONCLUSIVE preserved.
  - UI: gray metadata bar ("Behavior Contract / System Prompt" / "NovaBank Support Bot — latest developer rules"), renamed button "RUN BEHAVIORAL REGRESSION AUDIT", "Load conflicting-change demo" preset (input-only), FORTIFY & RETEST button only on verified-VIOLATED cards, Before/After result panel with fortified prompt, change summary, Original Attack vs Retest Attack — Exact Replay (identical), fresh retest response/evidence/explanation, fortify error + Retry.
  - Verified live: VIOLATED → fortify → exact replay → RESISTED with verified evidence; all negative guards (400s) pass. Testing agent: 100% backend + 100% frontend, no issues.

## Backlog (explicitly OUT of scope — do NOT build unless asked)
- P2: Fortify, Retest, CI/CD, GitHub integration, database, auth, accounts, multi-turn agents, monitoring, analytics, large attack libraries, security/readiness scores, certification, OWASP claims, export, payments, connect-to-real-endpoint.

## Next Tasks (only if user requests)
- Optional: server-side request timeout wrapper, basic rate limiting on public LLM endpoints.
