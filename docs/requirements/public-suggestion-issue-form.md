# Public suggestion issue form

## Classification

**Problem:** Public contributors need a structured way to propose improvements
that maintainers can evaluate against the Engineering Constitution's goals and
principles.

**Classification:** Substantial. The issue form introduces a public
contribution contract that external contributors may depend on.

**Simplest satisfying outcome:** One native GitHub Issue Form. A plain Markdown
template would be simpler, but cannot require the minimum information needed
for evaluation.

**Applicable skills:** `engineering-principles`, `requirements-engineering`,
compressed `architecture-analysis`, compressed `system-design`, compressed
`distributed-systems`, compressed `reliability-engineering`,
`software-design`, `software-lifecycle`, and `engineering-review`.

**Irreversibility:** None. The form can be changed or removed through version
control. Existing issues remain readable after a form change.

**Risks and unknowns:** Existing repository community-file conventions could
conflict with a new form; inspection confirmed none exist. GitHub schema errors
could prevent the form from rendering; validate its YAML and required fields
before release.

## Stakeholders

| Stakeholder | Need | Requirement |
|---|---|---|
| Public contributors | A clear path for proposing improvements | PS-1, PS-2 |
| Maintainers | Enough evidence and trade-off context to evaluate proposals | PS-2, PS-3 |
| Existing issue authors | Continued access to other issue creation paths | PS-4 |

## Requirements

- **PS-1 (firm):** GitHub presents a suggestion-specific issue form with a
  descriptive name, description, and title prefix. Verify by schema inspection.
- **PS-2 (firm):** The form requires the problem, proposed change, affected
  area, evidence, trade-offs, and alternatives. Verify by inspecting required
  form fields.
- **PS-3 (firm):** Contributors must acknowledge that they searched for an
  existing issue and that submission does not guarantee adoption. Verify by
  inspecting required checkboxes.
- **PS-4 (firm):** The change does not disable blank issues or configure
  repository-wide issue routing. Verify that no issue-template `config.yml` is
  introduced.
- **PS-5 (firm):** The form uses only GitHub-native Issue Form schema and adds
  no third-party dependency, automation, permissions, or assumed labels.
  Verify by inspecting the changed files.

## Quality attributes

- **Compatibility:** Existing issues and issue creation paths remain unchanged.
- **Operability:** No runtime service or maintainer-operated automation.
- **Security and privacy:** The form requests no secrets or personal data.
- **Performance, availability, durability, and volume:** Governed entirely by
  GitHub Issues; no additional target is introduced by this repository.

## Invariants and safety properties

- A suggestion can be submitted only after all evaluation-critical fields and
  acknowledgements are completed.
- The form never represents submission as acceptance or commitment.
- The repository does not depend on labels, bots, or external services for the
  form to work.

## Lifecycle and rollback

The change is additive and backward compatible. Existing issues are unaffected,
and blank issue creation remains at GitHub's existing default. Rollback consists
of reverting the issue-form file; there is no migration or point of no return.

## Out of scope

- Automated triage, scoring, labeling, or response workflows
- Bug reports, support requests, and security vulnerability reporting
- A contribution guide or governance policy
- Changing repository visibility or GitHub repository settings
