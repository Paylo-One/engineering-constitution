# Threat Model: <system/flow>

- **Status:** Draft | Reviewed | Superseded by <link>
- **Date:** <YYYY-MM-DD>
- **Author:** <agent/human>
- **Design document:** <link>
- **Scope:** <the flow/components covered; what is explicitly not covered>

## 1. Assets

What an attacker would want, and its classification.

| Asset | Classification (public/internal/PII/financial/secret) | Where it lives / flows |
|---|---|---|

## 2. Actors

Legitimate actors and their intended authority; assumed adversaries and
their capabilities (external attacker, malicious customer, compromised
dependency, insider, compromised agent/tooling).

| Actor | Intended authority | Adversarial capability assumed |
|---|---|---|

## 3. Trust Boundaries and Entry Points

Every place data or control crosses between trust levels.

| # | Boundary / entry point | What crosses | Authn | Authz | Validation |
|---|---|---|---|---|---|

## 4. Threat Enumeration

Per boundary/asset, walk the standard categories — spoofing, tampering,
repudiation, information disclosure, denial of service, elevation of
privilege — plus the domain-specific ones that apply:

- **Financial:** double-spend via replayed/duplicate requests; amount or
  currency tampering; rounding exploitation; reconciliation blind spots;
  refund/reversal abuse.
- **Injection/deserialization** at every parse point.
- **Prompt/data injection** where agents or LLM components consume
  untrusted content: instructions embedded in data must not gain authority.

| # | Threat (concrete: actor + action + asset) | Category | Boundary | Impact | Likelihood | Mitigation | Residual risk |
|---|---|---|---|---|---|---|---|

## 5. Abuse Cases

The system used as designed, but toward harm: enumeration, scraping,
rate/limit abuse, workflow abuse (e.g. creating and cancelling to farm
credits). Required behaviour under each.

## 6. Secrets and Sensitive Data Handling

- Where secrets are stored, how they rotate, what accesses them.
- Where PII/financial data is stored, logged, cached; retention; masking in
  logs and telemetry.
- Card-data never-events: CVV/CVC never stored after authorization; PAN
  stored only tokenized or truncated, and masked everywhere it is displayed
  or logged.
- Erasure obligations: personal data referenced from immutable/append-only
  records by token into an erasable store, never embedded inline.

## 7. Detection and Response

- What telemetry would reveal each high-impact threat being exercised?
- What alert fires, and what is the response runbook?

## 8. Accepted Risks

Residual risks accepted, by whom, with revisit condition.

| Risk | Accepted by | Date | Revisit when |
|---|---|---|---|
