# Design & Agent Patterns — Principles

## 1) Problem & Scope
- Start with the smallest valuable slice (one user, one trigger, one output).
- Define inputs/outputs upfront (typed schema + examples + edge cases).
- Decide “prototype vs production” on day 0 → sets guardrails and effort.

## 2) Context Is Everything
- Provide **task-specific context** that covers 99% of cases.
- Keep context **minimal and relevant**; prune aggressively to avoid prompt bloat.
- Prefer **retrieval over stuffing** (fetch only what’s needed per request).

## 3) Reasoning & Acting
- Instruct the model *how* to solve: **Reason → Decide → Act → Verify**.
- Encourage chain-of-thought reasoning internally; enforce structured outputs externally.
- Use **tool-first patterns** (call APIs/functions for side-effects or data).

## 4) Determinism & Contracts
- Validate all inputs; fail fast with actionable errors.
- Idempotent operations (safe retries).
- Version schemas and track migrations.

## 5) Model-Agnostic Interface
- Wrap LLM calls behind a clean API wrapper.
- Support swappable backends (OpenAI, Anthropic, local models).
- Pin versions, observe drift, regression-test prompts.

## 6) Prompt Management
- Treat prompts as **first-class artifacts** (version, diff, owner).
- Track prompt → model → config → output in logs.
- Use frameworks (LangChain, LlamaIndex) when they simplify retrieval or orchestration.
- Keep core logic framework-agnostic.

## 7) Privacy & Security
- **Privacy by design**: only collect the data you need; mask/obfuscate early.
- **Secure by default**:
  - Encrypt data in transit (TLS) and at rest.
  - Use least-privilege access (IAM roles, RBAC).
  - Rotate secrets; never hardcode.
- **Zero trust posture**: authenticate and authorize every call, even inside the cluster.
- **Audit trails**: log access and actions (without PHI) for compliance checks.
- **Isolation**: run sensitive components in separate namespaces or VPCs; sandbox LLM calls.

## 8) Data & Compliance (HIPAA-aware)
- **Data minimization**; mask/obfuscate early; never log PHI.
- Prototypes use **synthetic data**; real data only inside company infra.
- Secrets/config separated from code; rotate, audit, and monitor.

## 9) Small, Composable Units
- Separate orchestration (agent flow) from business logic and integrations.
- One concern per module; pure functions where possible.
- Make side effects explicit (network, file, DB).

## 10) Reliability by Default
- Timeouts + retries with backoff; circuit breakers on external calls.
- Dead-letter/quarantine on repeated failure (no silent drops).
- Health checks and graceful degradation.

## 11) Observability & Audit
- **Structured logs** (event, request_id, tool_calls, outcome) — no PHI.
- Metrics: success rate, latency, retries, token/cost usage.
- Traces across steps for debuggability (mask → classify → dispatch).

## 12) Configuration & Flags
- Config via env/ConfigMap; no magic constants.
- Feature flags for risky changes and gradual rollouts.
- Version configuration separately from code.

## 13) Guardrails (Business & Safety)
- **Business rules**: enforce hard constraints pre/post LLM.
- **Rate limits**: per user/service; backoff on spikes.
- **Execution limits**: max runtime, token budgets.
- Sandboxed transformations; no arbitrary code execution.

## 14) Cost Optimization
- Use the **cheapest model** that passes quality thresholds.
- Cap tokens (input/output); compress context via summaries/embeddings.
- **Spending caps & projections**: monitor per-team/env, project monthly usage.

## 15) Benchmarking & Evaluation
- Maintain **benchmark datasets** with expected outputs.
- Automate evals on every model/prompt change.
- Human-in-the-loop spot checks for sensitive flows.

## 16) Path to Production
- Prototype fast; measure adoption.
- Containerize; add probes, limits, structured logs.
- Deploy in company rails (k8s + GitOps) to inherit security/compliance.
