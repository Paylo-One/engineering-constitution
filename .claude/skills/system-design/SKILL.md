---
name: system-design
description: Design component boundaries, responsibilities, interfaces, and explicit data ownership before implementation. Use when responsibilities or boundaries need to be established, when new components or persistent state are introduced, or when it is unclear which part of the system should own a behaviour or a piece of data.
---

# System Design

## Purpose

Decompose the system into components with explicit responsibilities,
interfaces and state ownership, such that the requirements are satisfied by
the structure — not by heroics inside it. This skill governs *what the parts
are and who owns what*; `architecture-analysis` governs *choosing between
structural alternatives*; `software-design` governs *the code inside the
parts*.

## Trigger Conditions

- A new component, service, module or store is being introduced.
- Persistent state is being added, moved or shared.
- It is unclear which existing component should own a new behaviour.
- An interface between components is being created or changed.
- Two components are found writing to the same state, or a component's
  responsibilities have become unstatable in a sentence.

## Questions the Agent Must Answer

1. What are the components, and what is each one's responsibility in a
   single sentence without "and"?
2. For every piece of persistent state: which single component owns it,
   what invariants does the owner enforce, and how do all other components
   get access (owner's interface, replication with stated consistency, or
   not at all)?
3. What are the interfaces between components — operations, data contracts,
   error contracts, and the guarantees each interface makes (idempotency,
   ordering, at-least/at-most-once)?
4. Which design decisions are likely to change, and is each one hidden
   inside exactly one component (information hiding)?
5. Where are the trust boundaries, and what is validated, authenticated and
   authorized at each crossing?
6. What crosses process or network boundaries, and has each such crossing
   been handed to `distributed-systems` and `reliability-engineering`?
7. How does data flow for the main scenarios — and for the failure
   scenarios — end to end?
8. What is deliberately *not* being built, and where would it attach later
   without restructuring?

## Required Engineering Process

1. **Start from requirements and invariants**, not from an existing pattern.
   List the requirements the structure must satisfy; the design is verified
   against this list at the end.
2. **Identify the things that change together.** Group decisions likely to
   change for the same reason behind one boundary; separate things that
   change for different reasons. This — not size, not layer, not team — is
   the primary decomposition criterion.
3. **Assign state ownership first.** Before drawing interfaces, produce the
   state inventory: every store, table, topic, cache and file, each with
   exactly one owner and its invariants. Contention over ownership is a
   design smell to resolve now, not later.
4. **Design interfaces as contracts.** For each interface specify operations,
   inputs/outputs, error behaviour, and guarantees. Prefer few, deep
   interfaces over many shallow ones. Specify what callers may assume — and
   therefore what the implementation must preserve forever.
5. **Walk the failure paths.** Trace each main scenario end to end, then
   re-trace it with each component failing, slow, or returning errors.
   The structure must make the failure behaviour explainable.
6. **Check the design against each requirement and invariant** by stating,
   for each, which component(s) enforce it. A requirement no component
   enforces is unimplemented by construction.
7. **Record the design** using
   [templates/system-design.md](../../../templates/system-design.md), and
   open the threat model
   ([templates/threat-model.md](../../../templates/threat-model.md)) if any
   trust boundary is new or changed.

## Decision Rules

- **One writer per datum.** If two components need to write the same state,
  either one becomes the owner and the other calls it, or the state splits.
  Shared writable state without a single owner is prohibited (Principle 10).
- **Boundaries follow change, not layers.** Do not create a component
  because a diagram convention has a box there; create it because it hides
  a decision that changes independently.
- **Interfaces make guarantees explicit or make none.** If an interface does
  not state idempotency, ordering or delivery semantics, callers must assume
  the worst — so state them.
- **Prefer widening an existing component over adding a new one** unless the
  new responsibility changes for different reasons than the component's
  existing ones. Every new component costs an interface, a deployment, and
  a failure mode.
- **Trust boundary crossings always validate.** Data crossing a trust
  boundary is untrusted regardless of its source's reputation.
- **If a responsibility cannot be assigned in one sentence, the
  decomposition is wrong** — redraw it rather than annotating the exception.

## Anti-Patterns

- **Distributed monolith** — components separated by network calls but
  coupled by shared state, shared schemas, or lockstep deployment.
- **Ownerless state** — a table or topic that several components write "by
  agreement", with invariants enforced nowhere.
- **Anemic boundaries** — components split along technical layers (all
  controllers here, all models there) so every change touches every layer.
- **Interface leakage** — internal representations, database rows, or ORM
  entities exposed through an interface, freezing internals forever.
- **God component** — one component whose responsibility sentence requires
  three "and"s, absorbing every new behaviour by default.
- **Design by diagram** — boxes and arrows with no stated contracts,
  guarantees or failure behaviour.
- **Premature generalization** — abstraction layers and plugin points for
  variations no requirement demands.

## Required Outputs

- For Substantial work with new components or state, and all Critical work:
  a design document per
  [templates/system-design.md](../../../templates/system-design.md),
  including the state ownership inventory and interface contracts.
- A threat model per
  [templates/threat-model.md](../../../templates/threat-model.md) when trust
  boundaries are new or changed, when authentication/authorization changes,
  or for anything financial.
- ADRs (via `architecture-analysis`) for structurally irreversible choices.

## Exit Criteria

- Every component has a one-sentence responsibility.
- Every piece of persistent state has exactly one owner with stated
  invariants; all other access paths are enumerated.
- Every interface states its operations, errors and guarantees.
- Every requirement and invariant maps to the component(s) enforcing it.
- Failure paths for main scenarios are traced and explainable.
- Trust boundaries are identified and each crossing validates.

## Review Checklist

- [ ] Can each component's responsibility be stated in one sentence without "and"?
- [ ] Does any state have zero owners or two writers?
- [ ] Do interfaces state idempotency, ordering and delivery guarantees where relevant?
- [ ] Is anything internal (rows, entities, private enums) leaking through an interface?
- [ ] Which component enforces each invariant? Is any invariant enforced nowhere?
- [ ] What happens to each main scenario when each dependency fails or is slow?
- [ ] Is every new component justified by an independent axis of change?
- [ ] Were the crossings of process/network boundaries handed to `distributed-systems` and `reliability-engineering`?
