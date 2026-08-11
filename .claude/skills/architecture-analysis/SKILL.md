---
name: architecture-analysis
description: Analyse significant architectural decisions through explicit trade-off analysis driven by architecture characteristics, and record them as ADRs. Use when choosing between architectural styles, technologies, data stores, or communication patterns, or when any decision would be expensive to reverse.
---

# Architecture Analysis

## Purpose

Make significant structural and technology decisions by explicit trade-off
analysis against the characteristics this system must have — and record
those decisions so future engineers and agents inherit the reasoning, not
just the result. Architecture is the set of decisions that are expensive to
change; this skill exists to make those decisions deliberately.

## Trigger Conditions

- Choosing between architectural styles (monolith vs services, sync vs
  async, layered vs event-driven, etc.).
- Selecting a technology with lock-in: database, queue, framework, cloud
  service, external provider.
- Any decision that would be expensive to reverse — data model shape,
  partitioning scheme, public API style, consistency model.
- An existing architectural decision is being revisited or violated.
- Someone (including you) says "obviously we should use X".

## Questions the Agent Must Answer

1. Which architecture characteristics actually matter for this system, and
   what evidence says so? Pick the few that drive the decision (typically
   3–7: e.g. consistency, latency, availability, auditability, cost,
   evolvability, operability) rather than listing everything.
2. What are the measured or stated constraints — data volume, request rates,
   growth, team size and skills, budget, compliance, existing estate — that
   bound the option space?
3. What are at least two genuinely different options (not one option and a
   straw man)? What is the strongest case *for each*?
4. For each option: what does it make easy, what does it make hard, what
   does it make impossible, and what new failure modes and operational
   burdens does it introduce?
5. How reversible is each option, and what would migration away from it
   cost?
6. What is the simplest option that satisfies the driving characteristics
   (Principle 4)? If it is being rejected, exactly which requirement rejects
   it?
7. How will you know the choice is working — what measurement or fitness
   check would reveal it failing?

## Required Engineering Process

1. **Derive driving characteristics from the requirements** produced by
   `requirements-engineering`. Each driving characteristic must trace to a
   requirement, an invariant or a constraint — not to preference.
2. **Bound the option space with constraints first.** Eliminate options that
   violate hard constraints before comparing merits; do not spend analysis
   on the inadmissible.
3. **Steelman every surviving option.** Write the strongest honest case for
   each, including the ones you dislike. An analysis where one option has
   no real case is an announcement, not an analysis.
4. **Compare against the driving characteristics explicitly** — a short
   table or per-characteristic prose. Include second-order costs:
   operational burden, failure modes, hiring/knowledge, migration cost,
   vendor risk.
5. **Apply the decision rules below** to select.
6. **Record the decision as an ADR** using
   [templates/adr.md](../../../templates/adr.md): context, options,
   decision, consequences (including the negative ones), and the conditions
   under which it should be revisited.
7. **Define the revisit tripwire.** State the observable condition (a
   metric threshold, a scale point, a requirement change) that should
   trigger re-analysis, so the decision ages gracefully instead of
   silently.

## Decision Rules

- **There is no best, only least-worst for these constraints.** Select the
  option that best serves the driving characteristics; never justify a
  choice by its popularity, novelty or familiarity alone.
- **The simplest satisfying option wins by default.** Rejecting it requires
  naming the specific requirement it fails (Principle 4).
- **Irreversible decisions require stronger evidence.** For hard-to-reverse
  choices, prefer measurement and prototyping over estimation; if evidence
  cannot be obtained, prefer the more reversible option even at some cost
  (Principle 7).
- **Buy/adopt beats build for undifferentiated needs**; build only where
  the need is core to the product or no adequate option exists — and record
  the exit cost either way.
- **Adopting a dependency is an architectural decision.** Before adoption,
  check and record: maintenance health, license, security history,
  transitive dependency surface, and provenance (exact name verified
  against the registry — near-miss names are a supply-chain attack vector).
  Pin it in the lockfile. For load-bearing dependencies, this goes in the
  ADR.
- **ADRs are written before the implementation that depends on them.** At
  minimum Context, Driving Characteristics and Options are recorded first;
  the Decision section may be completed after prototyping. An ADR written
  after the fact to fit the code is decision laundering, and ordering is
  visible in history.
- **Don't pay distribution tax without distribution needs.** Splitting into
  separately deployed services is justified by independent scaling,
  independent deployment, fault isolation, or organizational boundaries —
  not by aesthetics.
- **A decision without an owner-visible record does not exist.** If it's
  expensive to reverse and there is no ADR, the work is incomplete.

## Anti-Patterns

- **Resume/fashion-driven architecture** — choosing technology for its
  novelty or career value rather than its fit (violates Principle 3).
- **Straw-man analysis** — one serious option padded with alternatives
  nobody would choose, to make the conclusion look analysed.
- **Characteristic inflation** — claiming every characteristic matters, so
  nothing constrains the choice.
- **Scale cosplay** — designing for hypothetical scale (multi-region,
  infinite horizontal scale) with no requirement or trajectory demanding it.
- **Second-system effect** — using the successor of a working system as the
  vehicle for every idea withheld from the first; replacements are held to
  the same simplicity discipline as new systems.
- **Incumbent inertia / incumbent hatred** — keeping or discarding the
  existing architecture reflexively instead of analysing it as an option
  like any other.
- **Decision laundering** — making the choice first and writing the ADR to
  fit.
- **Ignoring operational cost** — comparing options by development
  ergonomics while ignoring who gets paged and what recovery looks like.

## Required Outputs

- An ADR per [templates/adr.md](../../../templates/adr.md) for every
  decision that is expensive to reverse, including: driving
  characteristics, options with steelman cases, decision, negative
  consequences accepted, and revisit tripwire.
- For Critical work: ADRs are mandatory for all key decisions, not just the
  headline one.

## Exit Criteria

- Driving characteristics are named, few, and traceable to requirements.
- At least two genuine options were analysed with honest cases.
- The selected option's negative consequences are written down and accepted.
- The simplest satisfying option either won or was rejected by a named
  requirement.
- The ADR exists and includes a revisit tripwire.

## Review Checklist

- [ ] Do the driving characteristics trace to actual requirements or constraints?
- [ ] Is any rejected option a straw man? What was its strongest case?
- [ ] Was the simplest satisfying option identified — and if rejected, by which named requirement?
- [ ] Are the negative consequences of the winner stated, or only its virtues?
- [ ] Would this decision survive the disappearance of its advocate? Is the reasoning in the ADR self-sufficient?
- [ ] Is there a revisit tripwire with an observable condition?
- [ ] Does the choice add distribution, infrastructure or vendors that the requirements do not demand?
