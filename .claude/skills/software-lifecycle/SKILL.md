---
name: software-lifecycle
description: Manage change over time — API and schema evolution, migrations, versioning, deprecation, releases, rollback, and configuration. Use for any change to an interface others depend on, any data migration, any release or deployment planning, or when compatibility between old and new versions must be preserved.
---

# Software Lifecycle

## Purpose

Govern the dimension of time: systems are changed while running, data
outlives the code that wrote it, and every interface has dependents moving
at their own pace. This skill ensures changes land without breaking old
readers, old writers, in-flight work, or the ability to retreat
(Principles 18–20).

## Trigger Conditions

- Changing anything others depend on: API, schema, message/event format,
  library interface, configuration contract, file format.
- Any data migration: moving, transforming, backfilling or deleting data.
- Planning a release, deployment, or feature rollout.
- Deprecating or removing behaviour.
- Introducing or changing configuration and feature flags.

## Questions the Agent Must Answer

1. Who and what depends on the thing being changed — including consumers
   you cannot see (cached clients, queued messages, stored data written by
   old code, third parties)? Enumerate; do not assume.
2. During rollout there will be a period where old and new code run
   simultaneously against the same data and traffic. Is every combination —
   old-reader/new-data, new-reader/old-data, old-writer, new-writer —
   safe? For queued/stored data the window is as long as the oldest
   surviving message or row, not the deployment window.
3. Is this change backward compatible (new code reads old data/requests)
   and forward compatible (old code tolerates new data/requests)? If not,
   what staging sequence makes it so?
4. How does this change roll back? What is the *point of no return* (e.g.
   destructive migration step, external announcement), and is everything
   before it retreat-safe?
5. For migrations: how is correctness verified (counts, checksums,
   dual-read comparison)? What happens to writes that arrive mid-migration?
6. How will the change be released — behind a flag, canary percentage,
   cohort — and what observation gates each expansion step?
7. What is the deprecation path for the old behaviour: announcement,
   dual-support window, usage measurement, and the criterion for removal?
8. Is any of this irreversible (data deletion, format overwrite, external
   contract)? Irreversible steps happen last and only after verification.

## Required Engineering Process

1. **Inventory dependents and data lifetimes.** List consumers of the
   interface and the age distribution of stored/queued data that the new
   code must still read.
2. **Stage every incompatible change as expand → migrate → contract:**
   - *Expand*: add the new field/endpoint/table alongside the old; new code
     writes both/reads either. Deploy.
   - *Migrate*: backfill and verify; move consumers over; measure that old
     path usage reaches zero.
   - *Contract*: remove the old path — a separate, later change, gated on
     measured zero usage.
   Never combine expand and contract in one release.
3. **Make schema changes additive per step.** Adding optional
   fields/tables/indexes is safe; renames are an add + dual-write +
   remove sequence; type changes are new-column migrations. Message
   formats: readers ignore unknown fields, writers keep emitting required
   old fields until contract.
4. **Design the migration as a restartable, idempotent job** with progress
   tracking, rate limiting (it shares capacity with production), a
   write-handling strategy (dual-write, change-capture, or brief
   freeze — chosen explicitly), and a verification step that compares
   source and destination before any cutover.
5. **Gate rollout on observation.** Flag or canary first; define the
   metrics that must hold (errors, latency, business invariants) and the
   abort criteria before expanding. A rollout without an abort criterion is
   a hope, not a plan.
6. **Rehearse the rollback** for anything Critical: actually run the
   downgrade path (previous binary against new schema; flag off) in a
   pre-production environment. Rollback that has never run is a rumor.
7. **Version deliberately.** Public interfaces carry explicit versions;
   breaking changes get a new version with a dual-support window, a
   migration guide, and telemetry on old-version usage driving removal.
8. **Treat configuration as code**: reviewed, versioned, validated at
   startup (fail fast on invalid config), with safe defaults. Every flag
   has an owner and a removal date — flags are scaffolding, not
   architecture.

## Decision Rules

- **Data outlives code — compatibility is judged against data lifetime,
  not deploy time.** If messages live in a queue for days, "everyone is
  upgraded" takes days, not minutes.
- **Never break the reader.** A change is deployable only if code currently
  running in production can read everything the new code writes, or the
  new data is invisible to it.
- **Contract only on measured zero of legitimate traffic** — or on a passed
  deprecation deadline with every identified consumer notified and the
  residual breakage accepted in an ADR. Scanners, crawlers and health
  checks do not keep an endpoint alive forever; an unreachable "zero" must
  not freeze deprecations indefinitely — but the decision to break a
  laggard is recorded, never implicit.
- **Destructive steps are last and gated.** Data deletion/overwrite happens
  only after verification passes and a retreat is no longer needed;
  prefer soft-delete + delayed purge.
- **Roll back first, debug second.** If a rollout breaches its abort
  criteria, retreat immediately; diagnosis happens off the critical path.
- **One incompatible change at a time.** Do not stack schema change +
  behaviour change + dependency upgrade in a single release; each needs
  its own retreat path.
- **If rollback is impossible, say so in the plan** and compensate with a
  stronger gate (longer canary, dual-run comparison, human approval).
- **Dependencies age like everything else.** Versions are pinned via
  committed lockfiles; security advisories for the dependency set are
  monitored; security patches are applied on a stated cadence, and each
  upgrade is its own change with its own retreat path.

## Anti-Patterns

- **Big-bang migration** — one release that changes schema, code and
  consumers simultaneously, with downtime as the "plan" and no retreat.
- **Rename-in-place** — renaming a column/field/topic in one step,
  breaking every old reader and queued message at once.
- **Contract-with-expand** — removing the old path in the same release
  that introduces the new one, destroying the rollback path.
- **Fire-and-forget backfill** — a migration script with no idempotency,
  no progress tracking, no verification, run once against production.
- **Flag graveyard** — feature flags that never got a removal date,
  compounding into 2^n untested configuration states.
- **Version archaeology** — no explicit versioning, so compatibility is
  discovered by breaking consumers.
- **Deprecation by announcement only** — declaring something deprecated
  and removing it on a date, without measuring who still uses it.
- **Config drift** — production behaviour depending on unversioned,
  hand-edited configuration nobody can reproduce.

## Required Outputs

- For interface/schema changes: the compatibility analysis (old/new
  reader/writer matrix) and staging plan, in the design document or PR
  description.
- For migrations: a migration plan covering idempotency, write handling,
  verification, rate limiting and rollback — in the design document.
- For releases of Substantial+ work: rollout plan with gate metrics and
  abort criteria; rollback procedure. For Critical work: rehearsal
  evidence — where and when the downgrade path actually ran (environment,
  command or log reference, result). Unevidenced rehearsal counts as not
  rehearsed.
- ADRs for versioning strategy changes and any knowingly irreversible step.

## Exit Criteria

- All dependents (including stored/queued data) are enumerated and every
  old/new combination during rollout is safe.
- Incompatible changes are staged expand → migrate → contract across
  separate releases.
- Migrations are idempotent, verified, and have a write-handling strategy.
- The rollout has gate metrics and abort criteria; rollback is defined and
  (for Critical) rehearsed.
- Deprecations have telemetry and a measured-zero removal criterion.
- No unreviewed destructive step precedes verification.

## Review Checklist

- [ ] What old code, old data and in-flight messages will meet the new code? Every combination safe?
- [ ] Is expand separated from contract by at least one release and a zero-usage measurement?
- [ ] Can this deployment be rolled back after each step? Where exactly is the point of no return?
- [ ] Is the migration idempotent and restartable? How are concurrent writes handled? What verifies it?
- [ ] What metric aborts the rollout, and who/what watches it?
- [ ] Does any step delete or overwrite data before verification passes?
- [ ] Does every new flag have an owner and a removal date?
