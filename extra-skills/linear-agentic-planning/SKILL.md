---
name: linear-agentic-planning
description: Transforms an engineering objective into a dependency-aware, risk-classified Linear execution plan for human engineers and Cursor, Devin, or other coding agents. Use when planning engineering work, creating Linear projects, milestones, or issues, decomposing work for agents, or preparing Ready tickets.
---

# Linear Agentic Engineering Planning Skill

## Purpose

Use this skill whenever planning engineering work that will be managed through Linear and potentially executed by human engineers, Cursor agents, Devin agents, or other cloud coding agents.

The purpose of this skill is to transform an engineering objective into a **well-structured, dependency-aware, risk-classified execution plan** suitable for parallel agent execution.

The skill must optimise for:

1. Clear human accountability
2. Small, independently executable units of work
3. Safe parallelisation
4. Minimal ambiguity for execution agents
5. Explicit dependencies
6. Controlled architectural change
7. Small and reviewable pull requests
8. Machine-verifiable acceptance criteria
9. Human approval for consequential changes
10. Traceability from business intent to production code

The fundamental operating model is:

```text
Human Intent
    ↓
Project
    ↓
Milestones / Execution Waves
    ↓
Linear Issues
    ↓
Ready Gate
    ↓
Agent or Human Execution
    ↓
Pull Request
    ↓
Automated Verification
    ↓
Independent Review
    ↓
Human Approval where required
    ↓
Merge
    ↓
Release
    ↓
Done
```

---

# 1. Core Principle

Linear is the **source of truth for work**.

Repositories are the **source of truth for implementation and engineering rules**.

Git is the **source of truth for changes**.

CI/CD is the **source of truth for automated verification**.

Humans remain responsible for:

- product intent
- architecture
- risk acceptance
- consequential decisions
- production approval

Agents are execution capacity.

Do not allow an agent's conversational memory to become an authoritative source of project knowledge.

Important decisions must be persisted in:

- Linear
- repository documentation
- ADRs
- specifications
- schemas
- tests
- AGENTS.md
- code

---

# 2. Planning Mode

When this skill is invoked, remain in **planning mode**.

Do not:

- implement production code
- make unrelated repository changes
- silently resolve ambiguous business requirements
- invent architecture merely to make a ticket executable
- combine planning and implementation unless explicitly instructed
- expand the requested scope without recording the additional work

The output of this skill is an **execution plan and Linear work graph**.

If Linear tooling is available and the user requests ticket creation, the plan may be created directly in Linear.

Otherwise, produce Linear-ready content.

---

# 3. Understand the Objective First

Before decomposing work, establish:

## Business Objective

What outcome are we trying to achieve?

## Problem

What problem does this solve?

## Users / Consumers

Who or what consumes the capability?

Examples:

- external client
- internal operations team
- another service
- developer
- administrator
- compliance team

## Success Criteria

How will we know the project has succeeded?

## Constraints

Identify known constraints involving:

- architecture
- security
- compliance
- performance
- compatibility
- infrastructure
- deployment
- data
- external integrations
- deadlines

Do not start decomposition until the objective is sufficiently understood.

If critical information is genuinely missing, identify it explicitly.

---

# 4. Determine the Correct Linear Hierarchy

Use the following hierarchy:

```text
Project
    ↓
Milestone
    ↓
Parent Issue when useful
    ↓
Executable Issue
```

Do not create hierarchy for its own sake.

## Project

A project represents a meaningful business or engineering outcome.

Good:

```text
Request for Pay
KYT v2
Market Data Platform
Billing & Quoting
MS365 Support Integration
```

Bad:

```text
Backend
Go Services
Database Work
Admin Portal
```

Repositories and technologies are not normally projects.

---

# 5. Create Milestones as Execution Waves

Milestones should represent coherent stages of delivery.

Example:

```text
Project: Request for Pay

Milestone 1
Architecture & Contracts

Milestone 2
Core Infrastructure

Milestone 3
Provider Integrations

Milestone 4
Orchestration

Milestone 5
Operations & Administration

Milestone 6
Production Readiness
```

Milestones should help answer:

> What can safely happen in parallel, and what must exist before the next wave starts?

Prefer **wave-based execution** over launching all agents simultaneously.

---

# 6. Build the Dependency Graph Before Dispatching Work

For every issue determine whether it:

- blocks another issue
- is blocked by another issue
- can execute independently
- is related but not dependent
- should be a child of a larger issue

Explicitly model dependencies.

Example:

```text
PAY-101 Define payment contract
    ↓
    ├── PAY-110 Capitec adapter
    ├── PAY-111 Standard Bank adapter
    └── PAY-112 Persistence implementation
             ↓
         PAY-120 Orchestration
             ↓
         PAY-130 API
```

Do not rely only on prose such as:

> "Do this after the API work."

Represent it as a dependency whenever Linear supports it.

---

# 7. Identify the Unit of Agent Execution

The **leaf Linear issue** is normally the unit of execution.

Apply the rule:

> One agent → one Linear issue → one branch → one primary PR.

An executable issue should ideally be completable by one competent engineer or agent without coordinating implementation with another concurrent worker.

Bad:

```text
Build KYT v2
```

Better:

```text
Define screening domain model

Implement screening persistence

Implement provider adapter

Implement screening workflow

Expose screening API

Add screening metrics
```

---

# 8. Optimise Issue Size for Agent Execution

Prefer issues that are:

- bounded
- independently testable
- independently reviewable
- explicit
- reversible
- low in cross-repository contention

Avoid issues that require agents to make broad architectural decisions while implementing.

Split an issue when:

- it touches several unrelated concerns
- multiple independent PRs would make sense
- acceptance criteria contain separate capabilities
- different components could execute concurrently
- different risk levels exist inside the same issue
- different agents would likely edit unrelated areas

Do not split issues so aggressively that coordination overhead becomes larger than the implementation.

---

# 9. Classify Agentability

Every executable engineering issue must receive an Agentability classification.

Use:

## Agent/A — Mechanical

Examples:

- tests
- documentation
- dependency updates
- mocks
- straightforward renaming
- mechanical refactoring

Execution:

**Full agent autonomy within repository guardrails.**

---

## Agent/B — Well Specified

Examples:

- handler implementation
- CRUD endpoint
- provider adapter
- defined UI component
- explicit integration

Execution:

**Agent may implement autonomously and create a PR.**

Review remains required.

---

## Agent/C — Human Design Required

Examples:

- new workflow
- new datastore
- cross-service behaviour
- performance redesign
- state-machine changes

Execution:

**Human defines or approves the design before implementation is dispatched.**

Once design is approved, implementation may be delegated.

---

## Agent/D — Architecture

Examples:

- ledger architecture
- custody architecture
- messaging topology
- new domain boundaries
- major platform decisions

Execution:

**Human-led.**

Agents may:

- research
- analyse
- benchmark
- propose alternatives
- produce design drafts

Agents must not independently establish the architecture.

---

## Agent/E — High Risk / Human Controlled

Examples:

- production credentials
- custody keys
- IAM
- payment routing
- transaction limits
- compliance controls
- destructive production operations
- irreversible migrations

Execution:

**No autonomous consequential action.**

Agents may prepare changes, plans, tests, scripts, or PRs when explicitly permitted, but a human must authorize consequential execution.

---

# 10. Classify Risk Separately

Agentability and risk are separate dimensions.

Every executable issue must receive:

```text
Risk/Low
Risk/Medium
Risk/High
Risk/Critical
```

An issue may therefore be:

```text
Agent/B
Risk/High
```

because implementation is straightforward but consequences are significant.

Consider:

- financial impact
- security impact
- regulatory impact
- customer impact
- data loss
- production availability
- reversibility
- blast radius

When uncertain, classify upward rather than downward and explain why.

---

# 11. Assign Type

Use one primary type:

```text
Type/Bug
Type/Feature
Type/Improvement
Type/Refactor
Type/Infrastructure
Type/Research
Type/Security
Type/Documentation
```

Do not create unnecessary combinations.

---

# 12. Assign Component and Domain

Where useful, classify the affected component.

Examples:

```text
Component/KYT
Component/Transactions
Component/Wallets
Component/FX
Component/Billing
Component/Admin
Component/Infrastructure
Component/Observability
```

Optionally classify business/domain ownership:

```text
Domain/Payments
Domain/Risk
Domain/Treasury
Domain/Compliance
Domain/Developer-Platform
```

Do not create new labels unnecessarily.

Prefer existing taxonomy.

---

# 13. Define Human Ownership

Every meaningful issue must have a human accountable owner.

Agent execution does not remove human accountability.

Use:

```text
Human Owner
    ↓
Delegates execution
    ↓
Agent
```

Do not allow:

```text
Agent owns issue
Nobody accountable
```

For high-risk work, explicitly identify the required human approval point.

---

# 14. Create Every Executable Issue Using This Contract

Use the following structure.

## Objective

State precisely what must be achieved.

Prefer outcome language over implementation activity.

---

## Context

Explain:

- why this work exists
- relevant business context
- architectural context
- important previous decisions

Include only context necessary for execution.

---

## Scope

Explicitly state what must be implemented.

Use concrete requirements.

---

## Out of Scope

Explicitly identify tempting adjacent work that must not be performed.

This is important for preventing agent scope creep.

---

## Repository / Component

Specify when known:

```text
Repository:
Component:
Relevant paths:
Relevant services:
```

Do not guess repository locations when they are unknown.

---

## Dependencies

Identify:

```text
Blocked by:
Blocks:
Related issues:
External dependencies:
```

Use actual Linear relationships where possible.

---

## Technical Constraints

Capture relevant constraints such as:

- API compatibility
- approved libraries
- language/runtime versions
- database restrictions
- infrastructure requirements
- security requirements
- performance expectations
- observability requirements
- architectural patterns
- repository conventions

Reference existing repository documentation rather than duplicating it when possible.

---

## Acceptance Criteria

Acceptance criteria must be objectively verifiable.

Bad:

```text
The service should be robust.
```

Good:

```text
- Failed provider requests are retried according to the approved retry policy.
- Duplicate requests do not create duplicate payment instructions.
- Existing API contracts remain backwards compatible.
- Integration tests cover successful and failed provider responses.
```

Where possible, acceptance criteria should be machine-verifiable.

---

## Verification

Specify expected verification.

Examples:

```text
Build
Lint
Unit tests
Integration tests
Contract tests
Schema validation
Security scanning
Performance benchmark
```

Prefer the repository's canonical verification command where available.

Example:

```text
make verify
```

---

## Definition of Done

Normally include:

- [ ] Implementation complete
- [ ] Acceptance criteria satisfied
- [ ] Required tests added
- [ ] Existing tests pass
- [ ] CI passes
- [ ] PR created and linked
- [ ] Independent review completed
- [ ] Documentation updated where required
- [ ] Observability updated where required
- [ ] No unrelated changes included
- [ ] Required human approval obtained
- [ ] Released to required environment when Done means released

---

# 15. Apply Default Agent Guardrails

Unless explicitly overridden, every agent-executable issue inherits these rules.

## Scope

The agent must not:

- implement unrelated improvements
- refactor unrelated code
- expand scope because adjacent problems were discovered
- silently change business behaviour

## Architecture

The agent must not:

- introduce a new architectural pattern without approval
- replace major technology choices
- change service boundaries
- introduce a new datastore
- materially change public contracts

unless explicitly included in the issue.

## Dependencies

Do not introduce new production dependencies unless:

- explicitly required, or
- clearly necessary and explained in the PR.

Significant dependency changes require review.

## Data

Do not perform destructive migrations without explicit approval.

Schema changes must include appropriate migration and compatibility consideration.

## Security

Never:

- commit secrets
- weaken authentication
- weaken authorization
- bypass security controls to make tests pass
- expose previously private interfaces
- disable security checks

## Production

Agents must not autonomously:

- deploy to production
- modify production credentials
- rotate keys
- alter IAM
- execute destructive production operations
- change payment routing
- change transaction limits
- alter custody mechanisms
- change compliance controls

unless an explicit human-authorized procedure allows the specific action.

---

# 16. Ambiguity Rule

Agents must not invent business behaviour to unblock themselves.

When encountering ambiguity, determine whether it is:

## Minor Implementation Ambiguity

The repository already establishes a clear convention.

The agent may follow the existing convention and document the choice.

## Material Product or Architecture Ambiguity

The decision changes:

- behaviour
- architecture
- security
- money movement
- compliance
- data contracts
- public APIs
- significant operational behaviour

The agent must stop that part of execution and request clarification.

Do not reward agents for guessing.

---

# 17. Discovery Rule

During execution an agent may discover unrelated work.

Do not silently implement it.

Instead:

1. Record the discovery.
2. Determine whether it blocks the current issue.
3. If it does not block execution, propose a new Linear issue.
4. Continue the original scoped task.
5. If it blocks execution, surface the blocker.

Example:

```text
Current:
KYT-184 Implement screening workflow

Discovery:
Mongo connection pool configuration appears incorrect.

Action:
Propose KYT-196 Review Mongo connection pooling.
```

Do not expand KYT-184.

---

# 18. Parallelisation Analysis

Before declaring the project ready for execution, explicitly determine what can safely run concurrently.

Produce execution waves.

Example:

```text
Wave 1
Architecture contract

Wave 2
├── Persistence
├── Provider A adapter
├── Provider B adapter
└── Test infrastructure

Wave 3
├── Orchestration
├── API
└── Reconciliation

Wave 4
├── Administration
└── Observability

Wave 5
Integration and production readiness
```

Parallel issues should ideally:

- edit different areas
- have stable contracts
- not depend on unmerged implementation details
- remain backwards compatible
- be independently testable

Avoid parallelising work that is likely to modify the same core files.

---

# 19. Detect Agent Collision Risk

For every proposed parallel wave, consider:

```text
Repository overlap
File overlap
Schema overlap
API contract overlap
Shared configuration
Infrastructure overlap
Dependency-version overlap
```

If collision risk is high:

- sequence the work
- establish a prerequisite contract first
- combine tightly coupled work
- or assign one worker responsibility for the shared boundary

Do not maximise concurrency at the expense of integration quality.

---

# 20. Research Before Implementation Where Necessary

When the correct solution is genuinely uncertain, create a research issue before implementation.

Research issues should answer:

## Question

What must be determined?

## Options

What credible alternatives exist?

## Evaluation Criteria

Consider:

- correctness
- simplicity
- scalability
- performance
- security
- operational burden
- cost
- maintainability
- vendor dependency

## Required Output

The agent must produce:

1. Findings
2. Evidence
3. Options considered
4. Trade-offs
5. Recommendation
6. Risks
7. Unknowns
8. Proposed implementation approach

Research issues do not silently become implementation issues.

---

# 21. Architecture Decision Rule

When research results in a meaningful architectural choice, create or update an ADR where the repository uses ADRs.

The ADR should capture:

```text
Context
Decision
Alternatives
Rationale
Consequences
Risks
```

Implementation issues should reference the approved decision.

This prevents different agents from repeatedly reconsidering the same architectural question.

---

# 22. Review Independence

The implementing agent must not be considered sufficient final review for its own work.

For Agent/A and Agent/B work:

```text
Worker Agent
    ↓
CI
    ↓
Independent Agent Review where useful
    ↓
Human Review according to risk
```

For higher-risk work:

```text
Worker
    ↓
CI
    ↓
Independent Review
    ↓
Domain Owner
    ↓
Human Approval
```

Review requirements should increase with risk.

---

# 23. Pull Request Expectations

Agent-generated PRs should:

- reference the Linear issue
- explain what changed
- explain why
- identify significant implementation decisions
- list verification performed
- identify remaining risks
- disclose deviations from the original plan
- remain narrowly scoped

Large unrelated PRs are a planning failure.

Prefer multiple dependency-aware issues over one enormous PR.

---

# 24. Definition of Ready

An issue may move to **Ready** only when:

- [ ] Objective is clear
- [ ] Context is sufficient
- [ ] Scope is defined
- [ ] Out-of-scope is defined
- [ ] Acceptance criteria are testable
- [ ] Repository/component is identified where applicable
- [ ] Dependencies are identified
- [ ] Agentability is classified
- [ ] Risk is classified
- [ ] Type is classified
- [ ] Human owner is identified
- [ ] Required design decisions have been made
- [ ] Required guardrails are documented or inherited
- [ ] No unresolved material product ambiguity remains

The meaning of Ready is:

> A competent engineer or agent can execute this issue without needing to invent requirements.

---

# 25. Project Planning Output

When asked to plan a new project, produce the following before creating tickets.

## 1. Project Summary

```text
Project:
Objective:
Problem:
Success criteria:
Key constraints:
Human owner:
```

## 2. Architecture / Design Questions

Identify decisions that must be resolved before implementation.

Classify each as:

```text
Resolved
Research Required
Human Decision Required
```

## 3. Milestones / Execution Waves

Describe the delivery stages.

## 4. Issue Graph

Show dependencies.

Example:

```text
ARCH-101
   ↓
CORE-101 ──────┐
CORE-102 ──────┼──→ API-101
CORE-103 ──────┘
                    ↓
                 OPS-101
```

## 5. Parallelisation Plan

Identify which issues can safely execute simultaneously.

## 6. Risk Summary

Identify high-risk areas before execution.

## 7. Linear Issues

Produce or create each required issue using the standard issue contract.

## 8. Ready Assessment

Separate issues into:

```text
Ready for Agent
Needs Human Design
Needs Research
Blocked
Human Controlled
```

---

# 26. Ticket Quality Test

Before creating or recommending any issue, ask:

### Could an unfamiliar competent engineer understand why this exists?

If no, improve Context.

### Could they determine exactly what they are expected to deliver?

If no, improve Scope.

### Could they determine what they must not change?

If no, improve Out of Scope.

### Could they objectively prove completion?

If no, improve Acceptance Criteria.

### Could they identify relevant dependencies?

If no, improve Dependencies.

### Could an agent execute this without inventing product behaviour?

If no, it is not Ready.

### Could two agents execute this and another issue simultaneously without likely collision?

If no, adjust the execution waves.

### Is the risk classification appropriate to the blast radius?

If uncertain, classify upward.

---

# 27. Avoid These Anti-Patterns

Do not create vague tickets such as:

```text
Improve performance

Fix security

Build payments

Clean up code

Improve architecture
```

Do not create giant implementation issues.

Do not use agents as architects merely because they can generate architecture.

Do not maximise the number of simultaneous agents.

Do not confuse code generation speed with delivery speed.

Do not allow implementation agents to redefine scope.

Do not allow discovered technical debt to contaminate unrelated PRs.

Do not mark work Done simply because code was generated.

Do not let PR creation become the measure of success.

---

# 28. Optimisation Goal

The system is not optimised for:

```text
Maximum number of agents
Maximum code generated
Maximum tickets closed
Maximum PRs opened
```

It is optimised for:

```text
Intent
    ↓
High-quality Ready work
    ↓
Safe parallel execution
    ↓
Small validated changes
    ↓
Fast review
    ↓
Low rework
    ↓
Reliable production outcomes
```

The primary indicator of planning quality is:

> **What percentage of Ready issues can be completed without the executing engineer or agent requiring material clarification?**

If this number is poor, improve planning before increasing agent concurrency.

---

# 29. Default Behaviour When This Skill Is Invoked

When given a new initiative:

1. Understand the objective.
2. Inspect available context and repository documentation where permitted.
3. Identify unknowns.
4. Identify architectural decisions.
5. Determine whether research is required.
6. Define the Linear Project.
7. Define milestones as execution waves.
8. Decompose the work.
9. Build the dependency graph.
10. Analyse parallelisation and collision risk.
11. Classify each issue by Type.
12. Classify each issue by Agentability.
13. Classify each issue by Risk.
14. Assign Component/Domain labels where useful.
15. Identify the human accountable owner.
16. Add acceptance criteria.
17. Add verification requirements.
18. Add guardrails.
19. Apply the Definition of Ready.
20. Separate Ready work from blocked/design/research work.
21. Present the plan for review.
22. Only create Linear tickets when authorized to do so.

The desired result is not simply a collection of tickets.

The desired result is a **safe, dependency-aware execution graph that humans and cloud agents can consume predictably.**
