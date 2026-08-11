---
name: software-design
description: Apply disciplined design while writing or refactoring code — deep modules, information hiding, error design, and complexity control. Use during implementation of any behaviour change, when designing module or function interfaces, when refactoring, or when code exhibits change amplification or high cognitive load.
---

# Software Design

## Purpose

Govern the design of the code itself: modules, interfaces, abstractions,
errors, naming and comments. The unit of quality here is *complexity borne
by the reader and the caller* — every design choice is judged by whether it
makes the system easier or harder to understand and change (Principles 5, 8,
18). This skill applies while implementing what `system-design` structured.

## Trigger Conditions

- Implementing any behaviour change (Routine and above).
- Creating or changing a module, class or function interface.
- Refactoring, or deciding whether to refactor before a change.
- Code review comments indicate confusion, surprise, or repeated
  explanation.
- You notice change amplification (one logical change, many edit sites),
  high cognitive load, or unknown unknowns (unclear which code must change).

## Questions the Agent Must Answer

1. What does the caller of each new/changed interface need to know? Is that
   less — much less — than what the implementation knows?
2. What information does this module hide? If the answer is "none", why
   does it exist?
3. What is the most common use of this interface, and is it the simplest
   possible call for that use? Are rare-case parameters polluting the
   common case?
4. Which errors can occur, and for each: can it be **defined out of
   existence** (make the operation total), handled locally, or must it
   propagate — and in the last case, what can the caller actually do with
   it?
5. Can invalid states be made unrepresentable here — by types, enums, state
   machines, non-null constraints — instead of validated repeatedly
   (Principle 11)?
6. Does this change follow the codebase's existing concepts and idioms, or
   does it introduce a competing way to do the same thing (Principle 12)?
7. What would a reader six months from now misunderstand first, and what —
   structure first, then names, then comments — prevents that?

## Required Engineering Process

1. **Design the interface before the implementation.** Write the signature,
   the contract (inputs, outputs, errors, invariants preserved), and the
   caller's code *first*. If the caller's code is awkward, fix the
   interface now — it is the cheapest moment.
2. **Make modules deep.** Provide substantial functionality behind a small
   interface. If a module is an interface with barely any implementation
   (a pass-through, a thin wrapper), collapse it unless it hides a genuine
   decision.
3. **Pull complexity downward.** Given a choice between a simple
   implementation with a complicated interface and the reverse, complicate
   the implementation. Handle the awkward cases inside so callers don't.
4. **Design errors deliberately.** Prefer, in order: define the error out
   of existence (make the operation succeed by definition — e.g. deleting a
   missing item is success); handle it fully at the point of detection;
   aggregate and propagate with enough context to act on. Never swallow,
   never catch-and-log-and-continue past a broken invariant.
5. **Encode invariants in the type system and schema** where practical:
   sum types over boolean flags, parsed values over raw strings validated
   everywhere ("parse, don't validate"), constructors that cannot produce
   invalid objects.
6. **Match the codebase.** Follow existing naming, error handling, layering
   and idioms; if an existing idiom is wrong, change it deliberately and
   consistently, not by introducing a second idiom alongside it.
7. **Comment what the code cannot say**: units, ranges, invariants,
   ownership, why this way and not the obvious way. Interface comments
   describe the contract, not the implementation. Never narrate what the
   next line does.
8. **Refactor strategically.** When a change fights the current design,
   first reshape the design so the change becomes natural, then make the
   change — as two separately reviewable steps where feasible. Budget for
   this; working "tactically" (fastest local edit, design debt every time)
   is how systems rot.
9. **Test the contract.** Tests assert the interface's stated guarantees
   and invariants under normal, boundary and failure inputs — not the
   implementation's internal steps (Principle 23). Tests are deterministic:
   no real clocks, real networks, or unseeded randomness. A flaky test is
   quarantined and fixed as its own task — never deleted or retried into
   silence, because a flaky failure-mode test silently voids a verified
   guarantee. Test fixtures never contain production, personal or
   cardholder data.

## Decision Rules

- **Interface simplicity outranks implementation simplicity.** Complexity
  in one implementation is paid once; complexity in an interface is paid by
  every caller forever.
- **Duplicate once, abstract on the third occurrence — but only if the
  duplication is essential**, i.e. the copies must change together. Two
  things that merely look alike today must not be fused.
- **Special cases are a design smell.** Each `if` handling a special case
  at the call site is a candidate for redefinition inside the module.
- **Exceptions to invariants don't get code paths.** If an invariant has
  exceptions, it isn't the invariant — restate it until it holds
  universally, then enforce it.
- **A function should be one level of abstraction.** Mixed altitude (three
  lines of business logic, then socket handling) forces the reader to
  context-switch; split by level, not by length.
- **If it's hard to name, it's mis-designed.** A module or function that
  resists a precise short name has the wrong boundary; fix the boundary,
  not the thesaurus.
- **Leave it better or leave it alone — visibly.** Opportunistic
  improvements within scope are encouraged; unrelated drive-by rewrites in
  the same change are prohibited (they bloat review and risk).

## Anti-Patterns

- **Shallow module farms** — many tiny classes/functions that each add an
  interface but hide nothing, moving complexity into the wiring.
- **Pass-through layers** — methods that only forward to another method
  with the same signature.
- **Configuration escapism** — exposing a knob instead of making a
  decision, exporting the design problem to the operator.
- **Boolean blindness & flag arguments** — `process(true, false)`;
  behaviour switched by opaque flags instead of distinct operations or sum
  types.
- **Validate everywhere, guarantee nowhere** — the same raw input
  re-checked in five places because it was never parsed into a trustworthy
  type at the boundary.
- **Catch-log-continue** — handling an exception by logging it and
  proceeding with a violated invariant.
- **Comment compensation** — long comments explaining confusing code that
  should have been restructured instead.
- **Tactical tornado** — a stream of locally-fastest edits, each leaving
  the design slightly worse, praised for speed.
- **Test-shaped tests** — tests that mirror the implementation
  line-by-line (mock every internal call, assert every step) and therefore
  verify nothing but the code's own structure.

## Required Outputs

- Interface contracts (signature + documented contract) for new or changed
  public interfaces.
- Tests asserting the stated guarantees, including boundary and failure
  behaviour.
- Where a strategic refactor preceded the change: the refactor identified
  as such (separate commit/PR where feasible).

## Exit Criteria

- Every new/changed interface is simpler than its implementation and
  documented as a contract.
- The common case for each interface is the simplest call.
- Errors are defined away, handled, or propagated with actionable context —
  none swallowed.
- Invalid states are unrepresentable where practical; otherwise parsed and
  rejected at the boundary.
- The change follows or deliberately amends existing idioms — it does not
  fork them.
- Tests verify the contract and invariants, and fail if the guarantee is
  broken.

## Review Checklist

- [ ] Read each new interface as its caller: is the common case one obvious call?
- [ ] What does each new module hide? (If nothing — why does it exist?)
- [ ] Any pass-through layers, flag arguments, or knobs standing in for decisions?
- [ ] Trace each error path: defined away, handled, or propagated — is any swallowed?
- [ ] Could a type/enum/state machine make any of the runtime validation impossible instead?
- [ ] Does anything now exist in two idioms where the codebase had one?
- [ ] Do the tests assert guarantees (would they catch a correct-looking wrong implementation)?
- [ ] Do comments state contracts and non-obvious why — or narrate the obvious?
