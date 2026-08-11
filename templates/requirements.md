# Requirements: <title>

- **Status:** Draft | Agreed | Superseded by <link>
- **Date:** <YYYY-MM-DD>
- **Author:** <agent/human>
- **Work classification:** Routine | Substantial | Critical — triggered by <criterion>

## 1. Problem Statement

<The need in the stakeholders' terms, one paragraph, no solution language.
What happens if this is not built?>

## 2. Stakeholders

| Stakeholder | Type (user/operator/system/compliance/business) | Need |
|---|---|---|
| | | |

Include non-human stakeholders: downstream services, auditors, support
tooling, future maintainers.

## 3. Functional Requirements

Each requirement is testable: observable behaviour with inputs and outcomes.
Stability: **F**irm / **P**rovisional / **G**uess.

| ID | Requirement (given/when/then or input→behaviour→output) | Stability | Verified by |
|---|---|---|---|
| FR-1 | | F/P/G | <test/measurement/inspection> |

## 4. Quality Attributes

Numbers or explicit "unconstrained". Distinguish target from hard limit.

| ID | Attribute | Target | Hard limit | Verified by |
|---|---|---|---|---|
| QA-1 | Latency (p50/p99) | | | |
| QA-2 | Availability | | | |
| QA-3 | Throughput / volume (now → 12mo) | | | |
| QA-4 | Durability | | | |
| QA-5 | Security / data classification | | | |
| QA-6 | Auditability | | | |
| QA-7 | Cost ceiling | | | |

## 5. Invariants and Never-Events

What must always be true; what must never happen. These outrank features.
For financial work: state the conserved quantity and the record that proves
it.

| ID | Invariant / never-event | Enforced by (filled during design) | Verified by |
|---|---|---|---|
| INV-1 | | | |

## 6. Failure-Mode Requirements

Required behaviour under: invalid input, duplicate requests, concurrent
access, partial failure of dependencies, and crash/restart.

| ID | Condition | Required behaviour |
|---|---|---|
| FM-1 | | |

## 7. Constraints

Fixed facts no design may violate: regulatory, contractual, platform,
existing data/estate, deadline, budget.

## 8. Out of Scope

Explicitly excluded, with who agreed. Anything not listed in §3–§6 is also
out of scope by default.

## 9. Requirement Conflicts and Priorities

Where requirements tension against each other, the agreed priority and who
agreed it.

## 10. Open Questions

| # | Question | Blocking? | Owner | Resolution |
|---|---|---|---|---|
