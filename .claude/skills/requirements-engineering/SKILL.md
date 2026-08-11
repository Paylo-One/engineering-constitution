---
name: requirements-engineering
description: Establish verifiable requirements, invariants, and acceptance criteria before design or implementation. Use when requirements are unclear or implicit, when new behaviour is being introduced, when a task arrives as a solution without a stated problem, or when acceptance criteria are needed.
---

# Requirements Engineering

## Purpose

Turn a need into a set of verifiable statements the system must satisfy —
functional requirements, quality attributes, invariants and constraints —
before any structure is chosen. Requirements are the contract that design is
verified against; work without them can only be judged by whether it "seems
fine".

## Trigger Conditions

- New externally observable behaviour is being introduced. (For
  Routine-classified work, this skill produces only the inline requirements
  statement — see Required Outputs.)
- The task statement is a solution ("use Redis", "add a retry") rather than
  a need.
- Acceptance criteria are absent, ambiguous, or untestable.
- Stakeholders (users, other teams, other systems, regulators) are affected
  and their needs are not written down.
- Mid-implementation, when you discover the requirements are wrong,
  incomplete or contradictory (Principle 25).

## Questions the Agent Must Answer

1. Whose need is this — end user, operator, downstream system, compliance,
   business — and what happens to them if it is not met?
2. What must the system *do* (functional requirements), stated as observable
   behaviour with inputs and outcomes?
3. What must the system *be* (quality attributes): latency, throughput,
   availability, durability, consistency, security, auditability,
   cost — with numbers or explicit "don't care" statements?
4. What must *always* be true (invariants) and what must *never* happen
   (safety properties)? For financial work: what quantity is conserved, and
   what record proves it?
5. What are the fixed constraints — regulatory, contractual, platform,
   deadline, budget, existing data — that no design may violate?
6. What is explicitly out of scope, and who agreed to that?
7. How will each requirement be verified — by what test, measurement or
   inspection?
8. Which requirements conflict with each other, and what is the agreed
   priority between them?

## Required Engineering Process

1. **Separate need from solution.** Record the stakeholder need in the
   stakeholders' terms first; derive system requirements from it. If the
   task arrived as a solution, write the need it implies and confirm the
   solution actually serves it.
2. **Enumerate stakeholders**, including non-human ones: downstream services,
   batch consumers, auditors, support tooling, future maintainers.
3. **Write functional requirements as testable statements** — given/when/then
   or input/behaviour/output. Reject any requirement you cannot state a
   verification for.
4. **Quantify quality attributes.** "Fast" and "reliable" are not
   requirements. Every quality attribute gets a number, a range, or an
   explicit statement that it is unconstrained. Distinguish targets from
   hard limits. "Unconstrained" is a stakeholder decision, not an agent
   inference — record who stated it; absent a source, record the hostile
   assumption you designed against instead.
5. **State invariants separately from features.** Invariants (e.g. "the sum
   of ledger entries for a transfer is zero", "no order ships unpaid")
   survive redesigns and are the primary subject of testing (Principle 23).
6. **Mark stability.** For each requirement, note whether it is firm,
   provisional, or a guess — with the source of that judgment (who confirmed
   a firm one; what makes a guess a guess). Designs should hinge on firm
   ones.
7. **Resolve or record conflicts.** Where requirements tension against each
   other (latency vs consistency, flexibility vs simplicity), record the
   agreed priority; do not leave it to be discovered during implementation.
8. **Produce the requirements document** using
   [templates/requirements.md](../../../templates/requirements.md) at the
   depth the classification demands.

## Decision Rules

- **A requirement that cannot be verified is not a requirement.** Rewrite it
  until a test, measurement or inspection can pass or fail it, or demote it
  to a stated aspiration.
- **Unstated quality attributes default to hostile assumptions.** If no one
  stated a latency budget, assume one exists and ask; do not assume "any
  performance is fine".
- **Invariants outrank features.** When a feature request would break a
  stated invariant, the feature is renegotiated, not the invariant.
- **Scope is closed by default.** Anything not in the requirements is out of
  scope; expanding scope reopens requirements, it does not happen silently
  in code.
- **Ambiguity found late goes back to this skill.** Discovering ambiguity
  during implementation triggers a return here, not an on-the-spot
  interpretation (Principle 25). If a human decision is needed and no human
  is available, and **every candidate interpretation is cheaply
  reversible**, you may choose the most reversible one, mark it
  provisional, and surface it prominently in your final report. If the
  ambiguity touches anything irreversible, financial, or Critical, stop and
  ask — do not proceed provisionally (irreversible beats urgent).

## Anti-Patterns

- **Solution-as-requirement** — "the system shall use a message queue" is a
  design decision wearing a requirements costume.
- **Unfalsifiable requirements** — "shall be user-friendly", "shall be
  scalable", "shall handle errors gracefully".
- **Happy-path-only requirements** — no statement about behaviour under
  invalid input, partial failure, duplicate delivery, or concurrent access.
- **Coverage by example** — a list of examples standing in for a rule, so
  every uncovered case becomes a guess.
- **Gold-plating** — requirements no stakeholder asked for, added because
  they seem professional.
- **Frozen guesses** — treating provisional requirements as firm and
  building irreversible structure on them.

## Required Outputs

- For Substantial work introducing new behaviour, and all Critical work:
  a requirements document per
  [templates/requirements.md](../../../templates/requirements.md).
- For smaller work: an inline requirements statement (functional
  requirements, invariants, out-of-scope) in the task response or PR
  description.
- Every requirement carries an identifier so designs, tests and reviews can
  reference it.

## Exit Criteria

- Every requirement is verifiable and has a stated verification method.
- Quality attributes are quantified or explicitly unconstrained.
- Invariants and never-events are stated separately and completely.
- Constraints and out-of-scope are recorded.
- Conflicts between requirements have an agreed, recorded priority.
- Sufficiency is demonstrated, not asserted: every stakeholder need in the
  stakeholder table traces to at least one requirement that serves it, and
  the trace is recorded.

## Review Checklist

- [ ] Can every requirement fail a test? Which test?
- [ ] Are there numbers on latency, availability, durability, and volume — or explicit "unconstrained" statements?
- [ ] Are failure-mode behaviours specified (invalid input, duplicates, concurrency, partial failure)?
- [ ] Are financial invariants stated as conservation properties with a reconciliation source?
- [ ] Is anything phrased as a solution rather than a need?
- [ ] Is out-of-scope explicit?
- [ ] Would a stranger reading only this document build the right thing?
