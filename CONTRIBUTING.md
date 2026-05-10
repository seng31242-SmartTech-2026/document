# Contributing to `documents`

This guide defines the branching strategy and commit conventions for the `documents` repository. All team members must follow these rules before making any contribution.

---

## Branching Strategy

| Branch Pattern | Purpose | Example |
|---|---|---|
| `main` | Latest approved, reviewed work. Merges via PR only. | `main` |
| `draft/<document>` | Active working branch for a document or section in progress. | `draft/srs-chapter3` |
| `fix/<issue-number>` | Corrective changes addressing supervisor or peer review feedback. | `fix/42` |

**Rules:**
- Never commit directly to `main`.
- Branch off the latest `main` when starting new work.
- Delete your branch after it is merged.

```bash
git checkout main
git pull origin main
git checkout -b draft/sds-architecture-section
```

---

## Commit Message Convention

Follow the **Conventional Commits** specification adapted for a documentation project.

```
<type>(<scope>): <short imperative summary>

[Optional body: explain WHY, not WHAT]

[Optional footer: Closes #<issue-number>]
```

### Allowed Types

| Type | When to Use |
|---|---|
| `docs` | Adding or updating an SRS, SDS, meeting note, or any written document |
| `feat` | Adding a new section, use case, diagram, or design element |
| `fix` | Correcting an error or incorporating review feedback |
| `refactor` | Restructuring a document without changing its content |
| `chore` | Repository housekeeping (README, .gitignore, folder structure) |
| `style` | Formatting-only changes (heading levels, spacing, figure captions) |

### Allowed Scopes

`srs` · `sds` · `diagrams` · `adr` · `meeting-notes` · `pitch` · `planning` · `readme`

### Good Examples ✅

```
docs(srs): add functional requirements FR-05 through FR-09

Covers registration, login, password reset, and session management.
Aligned with use cases UC-03 and UC-04.

Closes #15
```

```
feat(diagrams): add activity diagram for UC-04 password reset flow

Source file: diagrams/act-UC-04.drawio
Export: diagrams/exports/act-UC-04.svg

Refs #23
```

### Bad Examples ❌

```
updated srs / fixed stuff / changes / v2 / final / FINAL_FINAL
```

---

## Pull Request Checklist

Before marking a PR as Ready for Review, confirm:

- [ ] Content is accurate and consistent with the approved project scope
- [ ] Diagrams follow UML 2.x notation
- [ ] Both diagram source files AND exports are committed
- [ ] No spelling or grammatical errors
- [ ] Document formatting follows the report specification (Chapter 8 of course guideline)
- [ ] PR description links to the related GitHub Issue (`Closes #N`)
- [ ] At least one team member is assigned as reviewer

---

*© Software Engineering Teaching Unit, University of Kelaniya — SENG 31242, 2026*
