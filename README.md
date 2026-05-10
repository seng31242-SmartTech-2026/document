# documents — SmartTech

This repository contains all formal project artefacts produced during the **design phase** of the SmartTech project (SENG 31242 – System Design Project, University of Kelaniya, 2026).

---

## Repository Purpose

The `documents` repository is the **single source of truth** for all written deliverables, diagrams, meeting records, and planning artefacts. Every document submitted to the supervisor panel originates from this repository.

---

## Contents Structure

```
documents/
├── pitch/
│   ├── pitch-deck.pdf              # Idea pitch slide deck (max 10 slides)
│   └── approval-form.pdf           # Signed project approval form
│
├── srs/
│   ├── srs-main.md                 # Software Requirements Specification
│   └── use-cases/
│       ├── UC-01.md                # One file per use case
│       ├── UC-02.md
│       └── ...
│
├── sds/
│   └── sds-main.md                 # Software Design Specification
│
├── architecture/
│   └── decisions/                  # Architectural Decision Records (ADRs)
│       ├── ADR-01-frontend-framework.md
│       ├── ADR-02-backend-framework.md
│       └── ...
│
├── diagrams/
│   ├── *.drawio                    # draw.io source files (REQUIRED)
│   ├── *.puml                      # PlantUML source files (REQUIRED)
│   └── exports/
│       └── *.svg / *.png           # Exported diagram images
│
├── meetings/
│   ├── sprint-1-planning.md
│   ├── sprint-1-retrospective.md
│   └── ...                         # One file per meeting / ceremony
│
└── planning/
    └── gantt.pdf                   # Project Gantt timeline
```

---

## Tooling Required

| File Type | Tool | Notes |
|---|---|---|
| `.md` | Any Markdown viewer / GitHub | Rendered natively on GitHub |
| `.drawio` | [draw.io / diagrams.net](https://app.diagrams.net/) | Free, browser-based |
| `.puml` | [PlantUML](https://plantuml.com/) | VS Code extension available |
| `.pdf` | Any PDF viewer | Adobe Reader, browser, etc. |

> ⚠️ **Diagram Rule:** Both the source file (`.drawio` / `.puml`) **and** the exported image (SVG or high-resolution PNG) must be committed. Committing only an exported image is not acceptable — source files are required for peer review and version diffing.

---

## Branching & Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full branching strategy and commit message convention used in this repository.

---

*© Software Engineering Teaching Unit, University of Kelaniya — SENG 31242, 2026*
