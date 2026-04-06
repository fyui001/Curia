# Curia

Multi-project development platform powered by Claude Code.

## Concept

- Place applications under `src/` and operate them with shared processes, agents, and plugins
- `src/` is gitignored — anyone can clone and place their own applications
- Generated documents are also gitignored (`docs/generated/`). Only templates are committed

## Setup

```bash
git clone <repository-url>
cd Curia

# Place your application
cp -r /path/to/my-project src/MyProject

# Create a project-specific CLAUDE.md
# Describe tech stack, architecture, etc. in src/MyProject/CLAUDE.md

# Link generated docs (optional)
ln -s ../../src/MyProject/docs docs/generated/MyProject
```

## Directory Structure

```
Curia/
├── .claude/
│   ├── settings.json          # Shared settings (permissions + plugins + hooks)
│   ├── settings.local.json    # Local overrides (gitignored)
│   ├── agents/                # Custom agent definitions
│   │   ├── pdm.md             # PdM (Product Manager)
│   │   └── tech-lead.md       # Tech Lead
│   └── skills/                # Process definitions (slash commands)
│       ├── requirements/      # /requirements — Requirements definition
│       ├── develop/           # /develop — Development
│       ├── acceptance/        # /acceptance — Acceptance testing
│       └── qa/                # /qa — Cross-cutting QA (ad-hoc)
├── docs/
│   ├── templates/             # Document templates (committed)
│   └── generated/             # Generated documents (gitignored)
├── src/                       # Applications (gitignored)
├── CLAUDE.md                  # Platform design philosophy & process definitions
└── README.md
```

## Plugin-First Architecture

Plugins are the primary actors. Custom agents only cover areas that plugins cannot.

### Plugins (Primary)

| Plugin | Type | Role | Phase |
|---|---|---|---|
| `feature-dev:feature-dev` | Skill | Guided development — analysis → design → implementation → quality review | Development |
| `feature-dev:code-explorer` | SubAgent | Codebase exploration — execution path tracing, architecture mapping (read-only) | Requirements, QA |
| `feature-dev:code-architect` | SubAgent | Architecture design — pattern analysis → implementation blueprints (read-only) | Requirements |
| `feature-dev:code-reviewer` | SubAgent | Code review — bugs, logic errors, quality checks, confidence ≥ 80 only (read-only) | Review, QA |
| `pr-review-toolkit:review-pr` | Skill | Comprehensive PR review — 6 specialized agents in parallel | Review, QA |
| `code-review:code-review` | Skill | GitHub PR auto-review — confidence ≥ 80 comments only | Review |
| `frontend-design:frontend-design` | Skill | Design-quality frontend implementation for new pages/components | Development (frontend) |
| `security-guidance` | Hook | Auto-detect XSS, command injection, etc. on Edit/Write | All phases (automatic) |

### Custom Agents (Complementary)

| Agent | Role | Why plugins can't cover this |
|---|---|---|
| **PdM** | Business value judgment, acceptance testing (Playwright MCP), UI/UX review | Business domain knowledge and user value assessment |
| **Tech Lead** | Security review (OWASP), API integration verification (curl), business logic consistency | Security judgment, real API execution, domain-specific logic |

**Note**: The team leader role (process control, task assignment, integrated verdict) is embedded in each skill (`/develop`, etc.). No dedicated agent is defined.

## Development Process

3-phase structure for all domains:

```
/requirements <project> <domain>  →  Requirements definition
/develop <project> <domain>       →  Development + review + merge
/acceptance <project> <domain>    →  Acceptance testing by PdM
```

Ad-hoc QA (outside the main process):

```
/qa <project> <portal>            →  Full-screen verification + code quality review
```

**Phase skipping is prohibited. Merging before acceptance testing is prohibited.**

## Workflow

```
/requirements:
  code-explorer (parallel) + code-architect (design) // PdM (business) → TechLead (supplement) → consensus

/develop:
  [New feature] feature-dev:feature-dev (analysis → design → implementation → quality review)
  [New UI]      frontend-design:frontend-design (design-quality frontend)
  [Bug fix]     code-explorer (root cause) → fix → code-reviewer (quality)
  → pr-review-toolkit:review-pr + code-review:code-review (PR review)
  → TechLead (security / API / business logic)
  → Skill integrates verdict and merges

/acceptance:
  PdM (Playwright MCP screen testing + UI/UX review)

/qa (ad-hoc):
  PdM (full-screen verification) + pr-review-toolkit (code quality) + TechLead (security/business)
```

## Project CLAUDE.md

Each project declares its specifics in `src/{project}/CLAUDE.md`:

| Section | Content |
|---|---|
| **Overview** | Project purpose and background |
| **Tech Stack** | Languages, frameworks, databases |
| **Architecture** | Layer structure, directory layout, naming conventions |
| **Reference Projects** | Pattern sources (paths and reference points) |
| **CI Commands** | Build, test, lint commands |
| **Docker Config** | Container names, execution methods |
| **UI Hostname** | Frontend access URL (direct localhost access is blocked by hooks) |
| **Implementation Phases** | Domain order and phase plan |

## Hooks

| Hook | Matcher | Behavior |
|---|---|---|
| localhost block | `Bash`, `Playwright navigate` | Block direct `localhost:<port>` access. Use project CLAUDE.md UI hostname |
| Pre-commit UI check | `Bash` (git commit) | Remind to verify UI with Playwright MCP before committing |
| Secret file guard | `Bash` (git add) | Block staging of `.env`, `credentials`, `secrets` files |
| Force push block | `Bash` (git push) | Block `git push --force` / `-f` |
| Main branch guard | `Bash` (git push) | Block direct push to `main` / `master` |
| Screenshot path guard | `Playwright screenshot` | Block saving screenshots outside `docs/generated/` |
| Security patterns | `Edit`, `Write` | Auto-detect XSS, command injection, unsafe deserialization (via `security-guidance` plugin) |
