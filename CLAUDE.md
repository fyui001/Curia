# Curia — Multi-Project Development Platform

## Overview

Curia is a multi-project development platform powered by Claude Code.
Applications are placed under `src/` and operated with shared processes, agents, and plugins.

## Directory Structure

```
Curia/
├── .claude/
│   ├── settings.json          # Shared settings (permissions + plugins + hooks)
│   ├── settings.local.json    # Local overrides (gitignored)
│   ├── agents/                # Custom agent definitions
│   │   ├── pdm.md             # PdM (business judgment, acceptance testing)
│   │   └── tech-lead.md       # Tech Lead (security, API, business logic)
│   └── skills/                # Process definitions
│       ├── requirements/SKILL.md
│       ├── develop/SKILL.md
│       ├── acceptance/SKILL.md
│       └── qa/SKILL.md        # Ad-hoc execution (outside main process)
├── docs/
│   ├── templates/             # Document templates (committed)
│   └── generated/             # Generated documents (gitignored)
│       └── {project}/         # Per-project
├── src/                       # Applications (gitignored)
│   └── {project}/
│       └── CLAUDE.md          # Project-specific settings
└── README.md
```

## Project Placement Convention

Projects can be placed at any depth under `src/`.

```
src/
├── ProjectA/                  # Direct placement
├── ProjectB/                  # Direct placement
└── group/
    └── ProjectC/              # Subdirectory placement
```

### Document Linking

Project documents are referenced via symlinks from `docs/generated/{project}`.
Skills access documents through the `docs/generated/{project}/` path.

```bash
# Example: Link ProjectC documents
ln -s ../../src/group/ProjectC/docs docs/generated/ProjectC
```

### Project Configuration Convention

Each project declares its specifics in `CLAUDE.md`.
Curia's skills and agents read this CLAUDE.md and dynamically adapt their behavior.

### Registered Projects

| Project | Path | Status |
|---|---|---|
| (none registered) | | |

### Required Sections in Project CLAUDE.md

| Section | Content |
|---|---|
| **Overview** | Project purpose and background |
| **Tech Stack** | Languages, frameworks, databases |
| **Architecture** | Layer structure, directory layout, naming conventions |
| **Reference Projects** | Pattern sources (paths and reference points) |
| **CI Commands** | Build, test, lint commands |
| **Docker Config** | Container names, execution methods |
| **UI Hostname** | Frontend access URL (direct localhost access blocked by hooks) |
| **Implementation Phases** | Domain order and phase plan |

## Plugin-First Architecture

Plugins are the primary actors in Curia's development. Custom agents only cover areas that plugins cannot.

### Plugins (Primary)

| Plugin | Type | Role | Phase |
|---|---|---|---|
| `feature-dev:feature-dev` | Skill | **Guided development** — analysis → design → implementation → quality review. Automatically launches code-explorer/code-architect/code-reviewer | Development |
| `feature-dev:code-explorer` | SubAgent | Codebase exploration — execution path tracing, architecture layer mapping, dependency analysis (read-only) | Requirements, QA |
| `feature-dev:code-architect` | SubAgent | Architecture design — existing pattern analysis → implementation blueprints (read-only) | Requirements |
| `feature-dev:code-reviewer` | SubAgent | Code review — bugs, logic errors, quality checks, confidence ≥ 80 only (read-only) | Review, QA |
| `pr-review-toolkit:review-pr` | Skill | **Comprehensive PR review** — 6 specialized agents (test quality/silent failures/type design/comments/code quality/simplification) in parallel | Review, QA |
| `code-review:code-review` | Skill | **GitHub PR auto-review** — 5 parallel agents, confidence ≥ 80 comments only | Review |
| `frontend-design:frontend-design` | Skill | **Design-quality frontend implementation** — production-grade pages and UI components | Development (frontend) |
| `security-guidance` | Hook | **Auto security detection** — blocks XSS, command injection, etc. on Edit/Write. Supplements Tech Lead's manual review | All phases (automatic) |

### Custom Agents (Complementary)

Only areas **not covered** by plugins are handled by custom agents.

| Agent | Role | Why plugins can't cover this |
|---|---|---|
| **PdM** | Business value judgment, acceptance testing (Playwright MCP), UI/UX review | Business domain knowledge and user value assessment are not in plugins |
| **Tech Lead** | Security review (OWASP), API integration verification (curl), business logic consistency | Security judgment, real API execution, and domain-specific logic verification are not in plugins |

**Note**: The team leader role (process control, task assignment, integrated verdict) is embedded in each skill (`/develop`, etc.). No dedicated agent is defined.

### Workflow

```
/requirements:
  code-explorer (parallel) + code-architect (design) // PdM (business) → TechLead (supplement) → consensus

/develop:
  [New feature]  feature-dev:feature-dev (analysis → design → implementation → quality review)
  [New UI]       frontend-design:frontend-design (design-quality frontend)
  [Bug fix]      code-explorer (root cause) → fix → code-reviewer (quality)
  → pr-review-toolkit:review-pr + code-review:code-review (PR review)
  → TechLead (security / API / business logic)
  → Skill integrates verdict and merges

/acceptance:
  PdM (Playwright MCP screen testing + UI/UX review)

/qa (ad-hoc):
  PdM (full-screen verification) + pr-review-toolkit (code quality) + TechLead (security/business)
```

## Team Composition

| Role | Type | Responsibility |
|---|---|---|
| PdM | Agent | Stakeholder. Final judgment on business requirements and acceptance criteria |
| Tech Lead | Agent | Security, API integration, business logic review. Supplements plugin results |
| Team Leader | Embedded in Skills | Process control and integrated verdict. Handled by skills (`/develop`, etc.) |

Implementation is handled by the `feature-dev:feature-dev` plugin. No custom "engineer agent" is defined.

### Agent Definitions

- `.claude/agents/pdm.md` — PdM
- `.claude/agents/tech-lead.md` — Tech Lead

## Development Process (3-Phase Structure)

```
/requirements → /develop → /acceptance
```

**Phase skipping is prohibited. Merging before acceptance testing is prohibited.**

### Critical Rules

- **Skills do not implement or investigate themselves.** Delegate to plugins and agents
- **Do not proceed to the next domain until PR merge is complete** (one-domain-at-a-time rule)
- **Do not leave CI red.** Fix immediately
- **Merge after acceptance testing.** Merging before acceptance is prohibited
- **Commit frequently.** One feature per commit. No large commits
- **Run all commands inside Docker containers** (follow Docker config in project CLAUDE.md)
- **TDD required.** No implementation without tests

## Docs Operation

- `docs/templates/` — Templates for requirements, task lists, acceptance tests, etc. Committed to git
- `docs/generated/{project}/` — Documents generated by skills. Gitignored
  - Each skill outputs documents under `docs/generated/{project}/`
  - Project name is auto-detected from skill arguments or current directory

### Screenshot Save Convention

**Saving Playwright MCP screenshots to the repository root is prohibited.** Always save to the following paths:

| Phase | Path |
|---|---|
| Development visual check | `docs/generated/{project}/screenshots/develop/{domain}/` |
| Acceptance testing | `docs/generated/{project}/screenshots/acceptance/{domain}/` |
| Cross-cutting QA | `docs/generated/{project}/screenshots/qa/{portal}/` |

Filename format: `{screen}-{state}.png` (e.g., `billing-list-dark.png`, `user-form-validation-error.png`)
