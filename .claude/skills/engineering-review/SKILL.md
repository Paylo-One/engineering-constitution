---
name: engineering-review
description: Perform an adversarial engineering review before declaring work complete — attacking requirements conformance, invariants, failure handling, state ownership, security, and test strength. Use before completing any Routine-or-larger work, when reviewing others' changes, or when asked to assess whether work is actually done.
---

# Engineering Review

## Purpose

Subject finished-looking work to an adversarial examination whose goal is to
**find the reasons it is not done** — not to confirm that it is. Work is
complete only when it demonstrably satisfies its requirements and invariants
and has survived this review (Principle 24). The reviewer's stance is that
the work contains at least one material defect and the job is to find it.

## Trigger Conditions

- Before declaring any Routine-or-larger work complete. For Routine work a
  self-review with the checklist answered in writing suffices.
- For Substantial and Critical work: mandatory as a **separate pass with
  fresh context** — a subagent or new session that has not seen the
  author's reasoning (see Required Engineering Process step 2 for the
  protocol).
- When reviewing another agent's or engineer's change.
- When work has been through significant rework since its last review.
- When the user asks "is this done?" or "is this correct?".

## Questions the Agent Must Answer

1. **Conformance:** For each requirement and invariant in the requirements
   document — where exactly is it satisfied, and which test proves it?
   Produce the mapping; every unmapped requirement is a finding.
2. **Interpretation drift:** Does the implementation solve the stated
   problem, or a nearby easier one? Compare behaviour against the
   requirement's words, not the author's summary of them.
3. **Failure behaviour:** For each external interaction — what happens on
   timeout, duplicate, partial failure, crash between steps? Were the
   `distributed-systems` and `reliability-engineering` exit criteria
   actually met, or asserted?
4. **State and invariants:** Can any sequence of operations — including
   concurrent and repeated ones — reach a state that violates a stated
   invariant? Actively construct such sequences; do not wait for them to
   suggest themselves.
5. **Security:** At each trust boundary — what does a malicious, malformed
   or oversized input do? Can any actor reach data or effects beyond their
   authorization? Were secrets, PII and financial data handled per the
   threat model?
6. **Test strength:** Would these tests fail if the code were subtly wrong?
   Pick 2–3 plausible bugs (off-by-one, inverted condition, dropped error,
   wrong idempotency scope) and check whether any existing test would catch
   each. Coverage numbers answer nothing here.
7. **Simplicity and integrity:** Is there structure no requirement demands?
   Does the change fork an existing idiom? Is complexity spent where the
   design documents said it would be?
8. **Operability:** Could an on-call operator, seeing this misbehave at
   3am, diagnose it from the telemetry this change ships?
9. **Honesty of the completion claim:** Was anything in the engineering
   sequence skipped silently? Do the documents match what was actually
   built?

## Required Engineering Process

1. **Gather the contract first**: requirements document, invariants, design
   document, ADRs, threat model. Review against those — not against the
   code's own apparent intent. If they don't exist for work that required
   them, that is itself a Critical finding; stop and route back.
2. **Read the diff cold, as a skeptic.** For Substantial and Critical work,
   the pass runs in fresh context so the author's framing cannot launder
   the review. The protocol: spawn a subagent (or new session) whose input
   contains **only** the diff, the requirements/design/ADR/threat-model
   documents, and the instruction to review per this skill. It must **not**
   contain the author's plan, summary, claims, or characterization of the
   change ("review this code, which correctly handles X" is contamination).
   If the environment cannot provide fresh context, say so in the report
   and compensate with a written attack log held to the same standard.
3. **Attack in order of consequence:** invariant violations and financial
   correctness first; then failure/concurrency behaviour; then security;
   then compatibility/rollback; then design quality; style last (and only
   where it obscures meaning).
4. **Construct concrete failure scenarios.** Every finding must name the
   sequence — inputs, timing, state — that produces the defect. "This looks
   risky" is not a finding; "two concurrent POSTs with the same key both
   pass the check at line N because the read and write are not atomic" is.
5. **Verify, don't pattern-match.** Before reporting, re-read the actual
   code paths to confirm the scenario is real in *this* code. Discard
   findings you cannot substantiate; a review that cries wolf gets ignored.
6. **Run the verification yourself** where possible: execute the tests,
   run the failure-mode tests, try the malformed input. Reviewed ≠ read.
7. **Classify findings and issue a verdict** (see Decision Rules), then
   **route fixes back through the sequence**: a requirements defect goes to
   `requirements-engineering`, not to a code patch that entrenches the
   misunderstanding.
8. **Re-review after fixes.** Material fixes get their findings re-verified;
   a fix can introduce the next defect.

## Decision Rules

- **Severity classification:**
  - *Critical* — violates an invariant, a Non-Negotiable, or loses/creates
    money or data; security breach; unrecoverable failure mode. **Blocks
    completion. No exceptions, including deadline pressure.**
  - *Major* — a requirement unmet or unverified; failure/concurrency path
    with wrong behaviour; missing rollback for an incompatible change;
    test suite that would not catch the plausible bugs. **Blocks
    completion unless the requirement itself is formally changed.**
  - *Minor* — design-quality or operability weaknesses that don't break
    guarantees. May be accepted with a recorded follow-up; silent
    acceptance is prohibited.
- **The verdict is one of:** `APPROVED`, `APPROVED WITH FOLLOW-UPS` (minors
  recorded), or `REJECTED` (any Critical/Major open). There is no "approved
  but concerned".
- **Absence of evidence blocks.** If a guarantee has no test and no
  demonstration, it is unverified — a Major finding — even if the code
  looks right.
- **Execution evidence is output, not assertion.** "Ran the tests; all
  pass" is a claim; the command and result summary (test names, counts,
  failures) is evidence. If execution was not possible, the report states
  why — an unexplained non-execution caps the verdict at APPROVED WITH
  FOLLOW-UPS.
- **The author's assurances carry zero evidentiary weight.** Only code,
  tests, documents and executed results count.
- **Findings the reviewer cannot substantiate are dropped**, not hedged
  into the report as vague concerns.
- **Review effort scales with blast radius**, not with diff size. A
  ten-line change to ledger posting gets more scrutiny than a
  thousand-line UI change.
- **Self-review is a floor, not a substitute.** For Substantial and
  Critical work, the fresh-context separate pass is mandatory even when
  self-review found nothing — the author's context is where the author's
  blind spots live.

## Anti-Patterns

- **Confirmation review** — reading the code looking for reasons it works,
  narrating what it does, and concluding "LGTM".
- **Checklist theater** — ticking boxes without constructing a single
  concrete failure scenario.
- **Style capture** — twenty naming comments and no examination of the
  idempotency logic.
- **Author-context contamination** — reviewing inside the author's
  reasoning (same conversation, same plan) so the review inherits the
  author's blind spots.
- **Fix-it-in-review** — patching defects inline during review and
  approving in the same breath, so the fix itself is never reviewed.
- **Severity haggling** — downgrading a Critical to Major because the fix
  is expensive or the deadline is near.
- **Vague-concern padding** — unsubstantiated "might be an issue" items
  that shift the verification burden back to the author.
- **Declaring done by exhaustion** — approving because the review is long,
  not because the findings ran out.

## Required Outputs

A review report containing:

- The requirement/invariant → implementation → test conformance mapping.
- Findings, each with: severity, the concrete failure scenario (inputs,
  sequence, state), the location, and the routed destination (which skill/
  stage the fix belongs to).
- **The attack log**: the concrete failure scenarios and plausible bugs
  constructed during the review — at minimum the 2–3 plausible bugs from
  Question 6 and one invariant-violation sequence from Question 4 — each
  with what was checked and why it did or did not become a finding. A
  zero-finding review with an empty attack log is not a review.
- What was executed during review (commands and result summaries, per the
  evidence rule above).
- For Substantial and Critical work: the review mechanism (subagent / new
  session) and, for Critical, the exact input given to the reviewer.
- The verdict: `APPROVED` / `APPROVED WITH FOLLOW-UPS` / `REJECTED`.
- For `APPROVED WITH FOLLOW-UPS`: the recorded follow-up list.

For Routine work, the report may be a compact version of the above in the
task response; the conformance check and verdict are never omitted.

## Exit Criteria

- Every requirement and invariant is mapped to implementation and
  verification, or reported as a finding.
- Every finding has a concrete, substantiated failure scenario and a
  severity.
- All Critical and Major findings are resolved (or requirements formally
  changed) and re-verified.
- The verification evidence (executed tests, tried inputs) is recorded.
- A verdict is issued. Work with any open Critical/Major is not complete,
  whatever the schedule says.

## Review Checklist

*(This checklist reviews the review itself.)*

- [ ] Was the review done against the requirements documents, or against the code's apparent intent?
- [ ] Does every finding name a concrete sequence that produces the defect?
- [ ] Were failure, concurrency and duplicate-delivery paths attacked — or only the happy path read?
- [ ] Were 2–3 plausible bugs tested against the test suite's ability to catch them?
- [ ] Was anything executed (tests, malformed inputs), or was this a reading?
- [ ] For Substantial+ work: was the review a genuinely separate pass with fresh context, given only the diff and documents — not the author's framing?
- [ ] Is the verdict one of the three allowed values, with no hedging?
- [ ] Did any Critical/Major finding get talked down in severity? By what argument?
