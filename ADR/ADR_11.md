# Title: Verifying GenAI Functionality and Managing Non‑Determinism

## Status
Accepted

## Context
GenAI outputs vary run‑to‑run and can drift over time. We need reliable ways to prove it works before rollout, detect regressions in production, and fail safely—across Demand Prediction, Battery Optimization, Vision Returns, and Personalization.

## Decision Drivers
- Objective pre‑prod evaluation with reproducibility
- Safe rollouts with quick rollback on SLO breaches
- Strong observability for quality/cost/safety/drift
- Guardrails and human‑in‑loop for critical flows
- Auditability and compliance

## Considered Options
- A) Manual spot‑checks and basic logs
- B) Full evaluation pipeline (golden datasets, CI gates), canary/A‑B, rich telemetry, drift monitors, and human‑review workflow
- C) Disable GenAI in critical paths; use rules only

## Decision 
Chosen option: "B) Evaluation pipeline + safe rollouts + observability + guardrails", because it provides measurable quality control and rapid mitigation while preserving GenAI value.

### Consequences
- Positive:
  - Quantified quality before release; automated rollback in prod
  - Live visibility into accuracy, cost, latency, safety, grounding coverage
  - Human review for low‑confidence/edge cases; audit trail
- Negative:
  - Additional infra and process overhead (datasets, eval harness, dashboards)

