# System Design: <title>

- **Status:** Draft | Reviewed | Superseded by <link>
- **Date:** <YYYY-MM-DD>
- **Author:** <agent/human>
- **Requirements:** <link to requirements document>
- **Related ADRs:** <links>
- **Threat model:** <link | "not required: no new/changed trust boundary,
  no authn/authz change, not financial" — all three must hold>

## 1. Overview

<One paragraph: the shape of the solution and why this shape. A reader
should be able to stop here and know what is being built.>

## 2. Components and Responsibilities

One sentence per component, no "and". New components justified by an
independent axis of change.

| Component | Responsibility (one sentence) | New/Changed/Existing |
|---|---|---|

## 3. State Ownership Inventory

Every store, table, topic, cache, file. Exactly one owner each.

| State | Owner (single writer) | Invariants enforced by owner | Other access (owner API / replica+consistency / none) |
|---|---|---|---|

## 4. Interface Contracts

Per new/changed interface:

### <interface name>
- **Operations:** <signatures / endpoints / message types>
- **Guarantees:** idempotency (key + scope), ordering, delivery semantics,
  atomicity
- **Errors:** what callers can receive and what they can do about each
- **Compatibility:** versioning; what old consumers see during rollout

## 5. Main Scenarios

End-to-end data flow for each primary scenario (messages and state changes,
not function calls).

## 6. Failure Analysis

### 6.1 Message faults
For each boundary-crossing interaction: behaviour when lost / duplicated /
delayed / reordered.

| Interaction | Lost | Duplicated | Delayed | Reordered |
|---|---|---|---|---|

### 6.2 Crash points
Numbered steps; for each gap, the intermediate state and its recovery path.

### 6.3 Dependency failure
Per dependency: behaviour when down / slow / erroring. Fail-closed paths
identified (money, auth).

| Dependency | Down | Slow | Erroring |
|---|---|---|---|

## 7. Requirements Conformance

Every requirement and invariant maps to the component(s) enforcing it.

| Req/Inv ID | Enforced by | Verified by |
|---|---|---|

## 8. Observability

Metrics, correlation propagation, key log events, alerts, and the SLOs they
serve.

## 9. Rollout and Compatibility

Expand/migrate/contract staging, rollback plan and point of no return
(or link to migration plan). Old/new reader/writer matrix if any format
changes.

## 10. Complexity Spent

Where the design is deliberately more complex than the simplest imaginable
version, and the requirement or invariant that bought each instance
(Principle 5). The adversarial review checks complexity against this list.

| Complexity introduced | Bought by (Req/Inv ID) |
|---|---|

## 11. Rejected Alternatives

Briefly: the simpler designs considered and the named requirement that
rejected each. (Full analysis lives in ADRs.)

## 12. Deliberately Not Built

What is out of scope structurally, and where it would attach later without
restructuring.
