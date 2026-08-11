# Engineering Principles

These are the durable principles of this constitution. Every skill, template and
review practice in this repository is an application of one or more of them.
They change rarely and only by deliberate amendment.

Each principle is stated as an obligation on the engineering agent.

## Reasoning before building

1. **Engineering precedes implementation.** Understand the problem, the
   requirements and the constraints before writing code. Implementation is the
   last stage of engineering, not the first.

2. **Requirements precede architecture; architecture precedes code.** Never
   select a structure before you can state what it must satisfy, and never
   write code before you can state where it belongs.

3. **Architecture follows constraints, not fashion.** Derive structural
   decisions from measured or stated constraints — load, latency, consistency,
   compliance, team shape, cost — never from what is currently popular.

4. **Prefer the simplest system that satisfies the requirements.** When two
   designs meet the requirements, the simpler one is correct. Justify every
   element that is not strictly required.

5. **Complexity is a cost paid on every future change.** Complexity is
   anything that makes the system harder to understand or modify. Spend it
   only where it buys a required property, and record where you spent it.

6. **Every significant decision is a trade-off analysis.** There are no best
   answers in architecture, only least-worst ones for the stated constraints.
   Record the alternatives considered, the criteria, and the consequences.

7. **Decide reversibly when you can; slowly when you cannot.** Classify
   decisions as reversible or irreversible. Take reversible decisions cheaply;
   subject irreversible ones to explicit analysis and record them.

## Structure and state

8. **Design deep modules.** A module's interface should be far simpler than
   its implementation. Pull complexity downward into implementations rather
   than pushing it up onto callers.

9. **Information hiding is the primary decomposition criterion.** Draw
   boundaries so that each design decision likely to change is hidden inside
   exactly one module.

10. **State ownership must be explicit.** Every piece of persistent state has
    exactly one owning component that writes it and defines its invariants.
    All other access is via the owner's interface or explicit replication with
    a stated consistency model.

11. **Make invalid states unrepresentable where practical.** Prefer types,
    schemas and state machines that cannot express illegal states. Where not
    practical, validate at the boundary and reject early.

12. **Conceptual integrity outranks local optimization.** One coherent design
    consistently applied beats a collection of locally clever fragments.
    New code follows the system's existing concepts or changes them
    deliberately — never silently forks them.

## Failure and distribution

13. **The network is not reliable.** Every remote interaction can fail, time
    out, duplicate, reorder or delay. Any design that assumes otherwise is
    wrong by construction.

14. **Partial failure must be explicitly designed.** Timeouts, retries,
    idempotency, backpressure and degraded modes are design decisions made
    before implementation, not patches applied after incidents.

15. **Everything fails; reliability comes from designed recovery.** Assume
    crashes at every step. Correctness must survive a crash between any two
    operations, and recovery paths are part of the design, not an afterthought.

16. **Financial effects must be idempotent and reconcilable.** Any operation
    that moves or commits value must be safely retryable, exactly-once in
    effect, attributable to a durable record, and independently reconcilable
    against an external source of truth.

17. **Security and privacy are architectural characteristics.** Trust
    boundaries, authentication, authorization, data classification,
    retention and abuse cases are established at design time, not
    retrofitted.

## Time and change

18. **Software engineering is programming integrated over time.** Optimize for
    reading, operating and changing the system over its expected life, not for
    the speed of the first implementation.

19. **Data outlives code; compatibility over time is a requirement.** Every
    schema, API or message format change must account for old readers and old
    writers coexisting with new ones during and after rollout.

20. **There are no purely internal changes.** Every observable behaviour of an
    interface has or will have dependents. Change interfaces deliberately,
    with versioning, migration and deprecation as first-class work.

21. **Partition work along architectural boundaries.** Adding effort to late
    work makes it later unless the work is divisible along clean interfaces.
    Structure systems so work can be divided; do not divide work where no
    boundary exists.

## Verification and honesty

22. **Observability is part of the design.** A system that cannot be
    interrogated in production — its state, its decisions, its failures — is
    incomplete regardless of test results.

23. **Tests verify guarantees and invariants, not coverage.** Every stated
    requirement, invariant and failure behaviour has a test or a documented
    reason why it cannot have one. Coverage without asserted guarantees is
    not verification.

24. **Completion requires verification and adversarial review.** Work is
    complete when it demonstrably satisfies its requirements and invariants
    and has survived an adversarial engineering review — not when the code
    compiles, and not when the author believes it is done.

25. **When reality and the plan conflict, stop and re-engineer — never
    silently reinterpret.** If implementation reveals that requirements are
    wrong, infeasible or ambiguous, surface the conflict and revise the
    requirements explicitly. Do not quietly build something else.
