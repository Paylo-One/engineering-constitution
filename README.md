# Engineering Constitution

A practical engineering doctrine for software-engineering agents.

The constitution is designed to make agents reason like strong senior
engineers: understand the problem before implementing, choose architecture
from constraints, make state ownership explicit, design for failure, preserve
compatibility, and verify claims with evidence.

It is intentionally opinionated. Its goal is not to produce more process or
more code, but systems that remain correct, understandable, reliable,
maintainable, operable, secure, and evolvable.

## What's included

- [`AGENTS.md`](AGENTS.md) — orchestration, work classification, mandatory
  engineering sequence, and non-negotiables
- [`PRINCIPLES.md`](PRINCIPLES.md) — 25 durable engineering principles
- [`.claude/skills/`](.claude/skills/) — operational skills for each
  engineering discipline
- [`templates/`](templates/) — lightweight templates for requirements,
  system design, ADRs, threat models, and production readiness
- [`CLAUDE.md`](CLAUDE.md) — directs Claude Code to the constitution

These files form one unit. The skills depend on `AGENTS.md`, `PRINCIPLES.md`,
and the templates being available at the repository root.

## How it works

Every task begins with classification:

- **Trivial** — no behaviour change and fully verifiable by inspection
- **Routine** — a narrow change within one component, with no new state,
  contract, or boundary crossing
- **Substantial** — new state, contracts, boundaries, dependencies,
  concurrency, shared code, or financial/security relevance
- **Critical** — irreversible, financial, security-sensitive, or migration
  work

The classification determines how much engineering is required. Substantial
and Critical work follows the full sequence:

1. Understand the problem
2. Establish requirements and invariants
3. Analyse architectural trade-offs
4. Design boundaries and state ownership
5. Design for distribution, failure, and recovery
6. Implement with disciplined software design
7. Plan compatibility, release, and rollback
8. Verify requirements and perform adversarial review

Smaller work uses a compressed process. The checking gets shorter; it does
not disappear.

## Using the constitution

Add the repository's contents to the root of the project or agent workspace
you want it to govern. The agent must be able to discover `AGENTS.md` before
starting work and resolve the relative links from the skills back to the
repository root.

For Claude Code, keep `CLAUDE.md` and `.claude/skills/` in place. For other
agents that support repository instructions or reusable skills, point their
equivalent mechanisms at `AGENTS.md` and the skill directories.

Start a task by applying `engineering-principles`. It classifies the work and
routes the remaining disciplines. Do not invoke individual skills in
isolation from the constitution: their requirements and authority come from
`AGENTS.md` and `PRINCIPLES.md`.

## Core non-negotiables

1. Financial effects are idempotent and reconcilable.
2. State ownership is explicit.
3. Remote interactions handle timeout, duplication, and failure.
4. Schema and interface changes preserve compatibility during rollout.
5. Substantial work receives adversarial review.
6. Requirements conflicts are surfaced, never silently reinterpreted.

Read [`PRINCIPLES.md`](PRINCIPLES.md) for the reasoning behind these rules.

## Contributing

Changes to engineering doctrine affect every project that adopts it. Keep
proposals small, explain the problem they solve, identify the trade-offs, and
update principles, skills, templates, and orchestration together when their
contracts change.

When contributing, apply the constitution to itself.

Have an idea but not a finished change? Open a
[suggestion](https://github.com/Paylo-One/engineering-constitution/issues/new?template=suggestion.yml)
with the problem, evidence, trade-offs, and alternatives so it can be
evaluated on its merits.

## License

Licensed under the [Apache License 2.0](LICENSE).
