---
name: distributed-systems
description: Reason correctly about distributed state, consistency, ordering, idempotency, and exactly-once effects. Use whenever work involves databases, queues, caches, asynchronous processing, remote APIs, concurrency, replication, transactions across boundaries, or any operation with financial effect.
---

# Distributed Systems

## Purpose

Ensure that any work touching more than one process, store or machine is
designed against the actual behaviour of distributed systems — messages that
duplicate and reorder, clocks that disagree, nodes that crash mid-operation,
reads that race writes — rather than against the convenient fiction of a
single reliable computer. This skill governs correctness under distribution;
`reliability-engineering` governs staying up and observable.

## Trigger Conditions

- Work involves any of: a database, a queue/stream, a cache, an external
  API, a background job, another service — anything reached over a
  connection is a distributed interaction, including a single application
  talking to a single database.
- Any state is replicated, cached, or derived from other state.
- Any operation must happen "exactly once" — especially payments, ledger
  writes, notifications, order state changes.
- Concurrent writers can touch the same entity.
- A transaction's steps span more than one store or system.
- Anything reads its own recent writes, or ordering between events matters.

## Questions the Agent Must Answer

1. Enumerate every message/request in the flow: what happens if it is
   **lost**, **duplicated**, **delayed**, or **reordered**? "It can't be" is
   not an answer unless the transport contractually guarantees it.
2. What happens if the process **crashes between any two steps**? Walk each
   gap: what state is left, and how is it detected and repaired?
3. For each operation with effects: what is the **idempotency key**, where
   is it stored, what is its scope and retention, and what does the
   operation return when replayed?
4. What is the **consistency model** of every read: read-your-writes needed?
   monotonic? bounded staleness acceptable? Which reads may be stale and
   which must not?
5. Where two systems must both change (DB + queue, DB + external API), what
   mechanism makes the pair atomic-in-effect — transactional outbox, saga
   with compensation, reconciliation — and what is the compensation when the
   second leg fails?
6. What orders events, given that wall clocks disagree? Sequence numbers,
   versions, or single-writer serialization — never timestamps alone for
   correctness.
7. Under concurrent access, which invariants must hold, and what enforces
   them — unique constraints, optimistic version checks, row locks,
   single-writer queues? For **in-process** concurrency (threads, shared
   memory), analyse the actual risks directly — atomicity of compound
   operations, lock ordering and deadlock, memory visibility — rather than
   producing message-fault tables that don't apply.
8. For financial effects: what durable record is written first, what is the
   external source of truth, and how does reconciliation detect and surface
   divergence (Principle 16)?

## Required Engineering Process

1. **Draw the flow as messages and state changes**, not as function calls.
   Every arrow crossing a process boundary gets the four-fault treatment
   (lost/duplicated/delayed/reordered).
2. **Do crash-point analysis.** Number the steps; for each gap between
   steps, state the intermediate state and its recovery path. Any gap whose
   answer is "inconsistent until someone notices" is a defect in the design.
3. **Design idempotency in, end to end.** Choose the key (caller-supplied
   for external effects), the dedup store, and the replay response. Verify
   the key survives the retry path actually used (client retries, queue
   redelivery, job re-runs).
4. **Choose consistency deliberately per read-path**, not globally. State
   which reads tolerate staleness and which require the owner's
   authoritative state; route them accordingly.
5. **Make cross-system writes atomic-in-effect.** Default order: single
   store transaction if possible > transactional outbox for DB+message >
   saga with explicit compensations for multi-service flows. Two-phase
   commit across heterogeneous systems requires an ADR.
6. **Enforce invariants where the data lives.** Uniqueness by unique index,
   conservation by constraint or single-writer ledger, state transitions by
   compare-and-swap on version — not by application-level check-then-act.
7. **Design reconciliation for anything financial or externally-effectful:**
   a periodic job that compares internal records against the external
   source of truth and surfaces divergence as an alert, not a log line.
8. **Write the tests that attack the design**: duplicate delivery, retry
   after timeout-but-success, crash between steps (kill points), concurrent
   writers on one entity, stale read after write.

## Decision Rules

- **At-least-once + idempotent effects is the default architecture.** Any
  claim of exactly-once delivery from infrastructure is treated as
  at-least-once; exactly-once is achieved in *effect*, at the receiver, via
  idempotency.
- **A timeout is an unknown outcome, not a failure.** After a timeout the
  operation may have succeeded; the caller must either retry idempotently
  or query for the outcome — never assume failure and re-execute a
  non-idempotent effect.
- **Never check-then-act across a boundary.** Any read-decide-write on
  shared state uses an atomic primitive (constraint, CAS, transaction) at
  the data.
- **Wall-clock time never establishes order or uniqueness.** Timestamps are
  for humans and metrics; ordering uses sequences, versions, or a single
  writer.
- **Money moves append-only.** Financial state changes are recorded as
  immutable entries (ledger style) with derived balances; corrections are
  new compensating entries, never updates in place.
- **Immutable records must not embed erasable data.** Personal data subject
  to erasure obligations is referenced from append-only records by token or
  pointer into an erasable store — never stored inline, or erasure requests
  become unsatisfiable.
- **External effect before durable intent is prohibited.** Record the
  durable intent (with idempotency key) first, then perform the external
  call, then record the outcome. The recovery path replays from intent.
- **Caches may be wrong; design for it.** Anything served from a cache must
  either tolerate staleness or be validated against the owner on the paths
  that cannot.

## Anti-Patterns

- **The reliable-network assumption** — no timeout, no retry policy, or
  retries on non-idempotent operations.
- **Exactly-once by hope** — deduplication nowhere, "the queue guarantees
  it" as a correctness argument.
- **Dual writes** — writing DB and publishing an event as two independent
  operations with no outbox or reconciliation, guaranteeing eventual
  divergence.
- **Distributed monolith transactions** — synchronous call chains that
  pretend to be one transaction and leave orphans when a middle link dies.
- **Clock-ordered correctness** — using timestamps to decide winner, order
  or uniqueness across nodes.
- **Balance as a mutable cell** — storing money as an updatable number with
  no entry history, making reconciliation and audit impossible.
- **Retry storms** — retries without caps, jitter, or budgets, converting a
  partial failure into a total one (see `reliability-engineering`).
- **Testing only the sunny day** — no test for duplicate delivery, crash
  recovery, or concurrent modification despite those being the actual risk.

## Required Outputs

- Crash-point and message-fault analysis recorded in the design document's
  failure section
  ([templates/system-design.md](../../../templates/system-design.md)).
- Idempotency design: keys, scope, storage, replay semantics — in the
  design document or interface contract.
- For financial flows: the ledger/record schema, the external source of
  truth, and the reconciliation mechanism, recorded in the design document
  and threat model.
- Tests covering duplicate delivery, crash recovery and concurrency for
  each critical flow.
- Where the classification requires no design document, the analyses above
  go in the PR description or task response — the document may be optional;
  the analysis never is.

## Exit Criteria

- Every boundary-crossing message has an answer for lost, duplicated,
  delayed and reordered.
- Every inter-step crash has a detected recovery path.
- Every effectful operation is idempotent with a designed key, or is
  explicitly single-shot with human-visible failure handling.
- Consistency needs are stated per read-path and satisfied by routing.
- Cross-system writes are atomic-in-effect via a named mechanism.
- Financial flows are append-only, idempotent, and reconciled against an
  external source of truth.
- The failure-mode tests exist and pass.

## Review Checklist

- [ ] Point to the idempotency key for each effectful operation. What happens on replay?
- [ ] Pick any two adjacent steps: crash between them — what repairs the state, and how soon?
- [ ] Is there any dual write (two systems, no outbox/saga/reconciliation)?
- [ ] Does anything use wall-clock time for ordering, uniqueness or expiry-based correctness?
- [ ] Is any check-then-act racing a concurrent writer?
- [ ] For money: append-only entries? External reconciliation? Alert on divergence?
- [ ] Do the tests include duplicate delivery, kill points, and concurrent writers — and do they assert invariants, not just status codes?
