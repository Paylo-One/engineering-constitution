---
name: reliability-engineering
description: Design for failure, degradation, capacity, and observability whenever work crosses a process, network, or infrastructure boundary. Use when adding or changing service dependencies, timeouts, retries, queues, or alerting, when preparing anything for production, when assessing production readiness, or when running a post-incident postmortem.
---

# Reliability Engineering

## Purpose

Ensure the system stays correct-enough and interrogable when its parts fail —
because they will. Where `distributed-systems` governs correctness of state
under distribution, this skill governs availability, degradation, capacity,
and observability: how the system behaves when a dependency is down, slow,
or lying, and how operators (human or agent) can see and steer it.

## Trigger Conditions

- Work adds or changes a dependency on another process, service, store, or
  third party.
- Timeouts, retries, circuit breakers, queues, or load shedding are being
  added or tuned.
- A component is being prepared for production, or its traffic profile is
  changing materially.
- New failure modes are introduced (new infrastructure, new deployment
  topology).
- Alerting, dashboards, or logging for a flow is being designed or is found
  missing.
- A production incident has been stabilized and its postmortem is due.

## Questions the Agent Must Answer

1. For each dependency: what happens to this component when that dependency
   is **down**, **slow**, or **returning errors**? Three separate answers —
   "slow" is usually the worst and least designed-for.
2. What is the timeout on every remote call, derived from what caller
   budget? What retry policy (attempts, backoff, jitter, retry budget), and
   is the retried operation idempotent?
3. What is the degraded mode? What can the system still do when each
   dependency is unavailable — and what does it refuse to do rather than do
   wrongly?
4. Where does load shed when demand exceeds capacity? What is bounded —
   every queue, pool, and buffer needs a limit and a defined behaviour at
   the limit.
5. What are the reliability targets (SLOs) for this flow — availability,
   latency percentiles, durability — and what error budget do they imply?
6. How is failure detected before users report it? Which metrics, and what
   alert thresholds tied to the SLOs?
7. Can an operator answer "what is happening and why" from the telemetry
   alone — request rate, error rate, latency distribution, saturation, plus
   per-request tracing/correlation across services?
8. What is the blast radius of this component failing, and what contains it
   (isolation, bulkheads, per-tenant limits)?
9. How does it recover — automatically after the dependency returns? Does
   recovery itself create a surge (thundering herd, replay storm)?

## Required Engineering Process

1. **Enumerate dependencies and do down/slow/erroring analysis** for each.
   Record the answers in the design document's failure section.
2. **Set explicit timeout and retry policy per call**, derived from the
   end-to-end latency budget (callers' timeouts must exceed the sum of
   callees' worst-case, or the retry is a lie). Retries get exponential
   backoff, jitter, a cap, and a budget; they are only applied to
   idempotent operations.
3. **Design degradation deliberately.** Decide per feature: fail closed
   (refuse), fail open with staleness (serve cached/last-known-good), or
   queue for later. Never let the default (hang, then cascade) stand.
   Fail-closed is mandatory for **state-changing value operations and
   authorization decisions**. Advisory money-adjacent checks (fraud
   scoring, loyalty, notifications) get a deliberately designed
   conservative fallback — reduced limits, queue-for-review — with the
   risk acceptance recorded; automatic fail-closed there converts a
   partial outage of an advisory service into a total payment outage.
   Read-only display of financial data may serve stale, labeled as such.
4. **Bound everything.** Every queue, connection pool, in-memory buffer and
   concurrent-request count gets a limit; define what happens at the limit
   (reject, shed lowest priority, backpressure upstream).
5. **Define SLOs and instrument for them.** SLOs come from the
   organization's existing targets where they exist; otherwise propose one
   and mark it provisional-pending-owner — never silently invent a number
   that then reads as agreed. Emit the four golden signals
   (traffic, errors, latency, saturation) per flow; propagate a correlation
   ID across every boundary; log state *transitions* and *decisions* with
   context, structured, not prose.
6. **Design alerts from symptoms, not causes**: alert on SLO burn
   (user-visible symptoms) with cause-based dashboards behind them. Every
   alert must be actionable; if the response is "ignore it", delete it.
7. **Plan recovery and test it.** Verify behaviour under dependency failure
   with fault-injection tests (kill it, delay it, error it) at least at the
   integration-test level. Recovery paths that have never run do not exist.
8. **For production deployment: complete the production readiness review**
   using
   [templates/production-readiness.md](../../../templates/production-readiness.md).
9. **After an incident: run a blameless postmortem** and close the learning
   loop. Establish the timeline and contributing causes (not a guilty
   party); then convert every lesson into an artifact: a new failure-mode
   test that would have caught it, an alert or SLO change that would have
   detected it sooner, and updates to the ADR revisit tripwires it
   implicates. A postmortem that changes no test, alert or decision record
   has not happened.

## Decision Rules

- **Value-changing and authorization paths fail closed.** Degraded modes
  may reduce features, never guarantees. Advisory checks degrade to their
  designed conservative fallback, not to silent approval.
- **No unbounded anything.** An unbounded queue is a slow-motion outage; a
  missing timeout is an infinite one.
- **Slowness is failure.** A dependency exceeding its latency budget is
  treated as failed (timeout + fallback), because waiting consumes the
  caller's capacity and cascades.
- **Retries amplify load; budget them.** Total retry volume is capped
  (e.g. as a fraction of request rate); on breaker-open or shed, back off —
  don't hammer.
- **If it isn't instrumented, it isn't done.** A flow without metrics,
  correlation and alert coverage does not pass review, regardless of tests
  (Principle 22).
- **Alerts page humans only for what humans must do now.** Everything else
  is a ticket or a dashboard.
- **Capacity claims require arithmetic.** Expected load × cost per request
  vs provisioned capacity, with headroom stated — not "should be fine".

## Anti-Patterns

- **Infinite patience** — remote calls with no timeout, or timeouts longer
  than the caller's own deadline.
- **Reflexive retries** — retrying non-idempotent calls, retrying without
  backoff/jitter, retrying into a failing dependency until everything is
  saturated.
- **Cascade by default** — one slow dependency consuming all threads/
  connections so an unrelated feature dies with it (no bulkheads).
- **Fail-open money** — proceeding with a financial or authorization action
  because the checking service was unavailable.
- **Log-and-hope observability** — unstructured prose logs, no correlation
  IDs, metrics added after the incident.
- **Alert spam** — cause-based, non-actionable alerts that train operators
  to ignore the channel.
- **Untested recovery** — failover, replay and restore paths that exist
  only in the design document.
- **Hero capacity planning** — sizing by vibes; discovering the limit in
  production.

## Required Outputs

- Dependency failure analysis (down/slow/erroring per dependency) in the
  design document.
- Timeout/retry/backoff policy table for every remote call.
- SLOs for the flow and the metrics/alerts that watch them.
- Fault-injection tests for critical dependency failures.
- For new services or first production deployments: a completed
  [production readiness review](../../../templates/production-readiness.md).
- After incidents: a postmortem whose action items are concrete artifacts
  (tests, alerts, tripwire updates), each with an owner.
- Where the classification requires no design document, the analyses above
  go in the PR description or task response — the document may be optional;
  the analysis never is.

## Exit Criteria

- Every remote call has a timeout derived from a budget, and a retry policy
  safe for its idempotency.
- Every dependency has designed down/slow/erroring behaviour; money and
  auth paths fail closed.
- Every queue, pool and buffer is bounded with defined at-limit behaviour.
- SLOs exist; golden-signal metrics and symptom-based alerts are wired.
- Correlation IDs propagate across every boundary in the flow.
- Recovery under dependency failure is demonstrated by test, not asserted.

## Review Checklist

- [ ] Pick any remote call: what is its timeout, and whose budget is it derived from?
- [ ] Pick any dependency: what exactly happens when it is slow (not down — slow)?
- [ ] Is anything unbounded — queue, pool, buffer, concurrency, log volume?
- [ ] Do money/auth paths fail closed under every dependency failure?
- [ ] Can you trace one request across all boundaries with a single ID?
- [ ] Does every alert have a documented action? Does every SLO have an alert?
- [ ] Has the failure behaviour been exercised by a test, or only described?
- [ ] Does recovery avoid creating its own surge (herd, replay storm)?
