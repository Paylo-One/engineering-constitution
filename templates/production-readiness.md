# Production Readiness Review: <component/service>

- **Date:** <YYYY-MM-DD>
- **Author:** <agent/human>
- **Design document:** <link> · **Threat model:** <link> · **ADRs:** <links>
- **Verdict:** READY | READY WITH FOLLOW-UPS | NOT READY

Each item: **Pass / Fail / N/A (reason)** with evidence (link to test, dashboard,
config, runbook). Unevidenced passes are fails.

## 1. Requirements and Verification

- [ ] Requirements and invariants documented; conformance mapping complete
- [ ] Tests verify stated guarantees incl. failure modes (duplicates, crash
      recovery, concurrency) — not just coverage
- [ ] Adversarial engineering review performed as a fresh-context pass;
      verdict APPROVED
- [ ] Tests deterministic; no production, personal or cardholder data in
      fixtures

## 2. Failure Behaviour

- [ ] Every remote call has timeout + retry policy (idempotent-only), with
      backoff, jitter, cap
- [ ] Dependency down/slow/erroring behaviour designed and tested
- [ ] Money and authorization paths fail closed
- [ ] All queues, pools, buffers bounded with defined at-limit behaviour
- [ ] Fault-injection tests exist for critical dependencies

## 3. State and Data

- [ ] Every store has a single owning writer; invariants enforced at the data
- [ ] Effectful operations idempotent (key, scope, replay semantics defined)
- [ ] Financial records append-only; reconciliation job + divergence alert
      in place
- [ ] Backups/durability configured; restore actually tested
- [ ] Data retention and PII handling match the threat model

## 4. Observability

- [ ] Golden signals (traffic, errors, latency, saturation) emitted per flow
- [ ] Correlation ID propagates across every boundary
- [ ] SLOs defined; symptom-based alerts wired to them; every alert has a
      documented action
- [ ] Dashboards exist for the cause-level diagnosis behind each alert
- [ ] Logs are structured; state transitions and decisions logged with context;
      no secrets/PII in logs

## 5. Capacity

- [ ] Expected load stated with arithmetic (rate × cost vs provisioned, with
      headroom %)
- [ ] Load tested to expected peak × safety factor
- [ ] Scaling mechanism (auto or runbook) defined and exercised
- [ ] Cost at expected load estimated and within budget

## 6. Deployment and Retreat

- [ ] Rollout is gated (flag/canary) with metrics and abort criteria
- [ ] Rollback procedure defined; rehearsed for Critical work
- [ ] Schema/interface changes staged expand→migrate→contract; old/new
      compatibility verified
- [ ] Config versioned, validated at startup, safe defaults

## 7. Security

- [ ] Threat model completed; high-impact mitigations implemented and tested
- [ ] Secrets in a secret store, rotatable; least-privilege service credentials
- [ ] Input validation at every trust boundary; authn/authz tested negatively
      (forbidden actions actually forbidden)
- [ ] Dependencies pinned via committed lockfile; vulnerability scanning in
      place; security-patch cadence stated

## 8. Operations

- [ ] Runbook: what this does, how it fails, top alerts and responses,
      escalation path
- [ ] Ownership assigned (who is paged; who maintains)
- [ ] Dependencies' SLAs/limits documented (rate limits, quotas)

## Follow-ups (for READY WITH FOLLOW-UPS)

| # | Item | Owner | Due |
|---|---|---|---|
