---
name: develop
description: Run the development phase for a domain — orchestrate feature-dev/frontend-design plugins and code-explorer/code-reviewer for implementation, then PR review (plugins + Tech Lead) and merge. Invoke with /develop <project> <domain> [--task <taskID>].
---

# Development Skill

## Description
Launch the development phase. New features use the `feature-dev:feature-dev` plugin for guided development; bug fixes follow the flow of `code-explorer` analysis → fix → review. **This skill acts as the team leader role**, orchestrating plugins and agents.

## Usage
```
/develop <project> <domain> [--task <taskID>]
```

### Examples
```
/develop MyProject user                           # All tasks for user domain
/develop MyProject property --task A-property-001  # Specific task
/develop MyProject qa-billing-fix                  # QA bug fix
```

## Instructions

When this skill is invoked, start the development phase.

### Step 0: Prerequisites Check

1. Read `src/{project}/CLAUDE.md` — and `src/{project}/CLAUDE.local.md` if it exists (authoritative when both exist) — to understand project-specific information
2. Review the requirements document (`docs/generated/{project}/requirements/{domain}/`) or GitHub Issue content
3. If a task list exists, analyze dependencies and identify executable tasks
4. Check `gh pr list --state open` for unmerged PRs. If any exist, prioritize their merge first

### Step 1: Task Analysis & Strategy Decision (Team Leader Role)

As team leader, determine each task's type and implementation strategy.

| Task Type | Strategy |
|---|---|
| **New feature** (new domain implementation) | → Step 2A: Guided development with `/feature-dev:feature-dev` |
| **New frontend UI** (standalone page/component) | → Step 2A-F: `/frontend-design:frontend-design` for design-quality implementation |
| **Bug fix** (QA Issues, etc.) | → Step 2B: `code-explorer` analysis → fix → review |
| **Refactoring** | → Same flow as Step 2B |

### Step 2A: New Feature Implementation — feature-dev:feature-dev

Launch the `/feature-dev:feature-dev` skill. This plugin executes the following in sequence:

1. **Discovery**: Requirements clarification
2. **Codebase Exploration**: Launch code-explorer in parallel to analyze existing patterns
3. **Clarifying Questions**: Ask about ambiguities
4. **Architecture Design**: Launch code-architect in parallel to present multiple design proposals, user selects
5. **Implementation**: Implement based on design
6. **Quality Review**: Launch code-reviewer in parallel for quality check
7. **Summary**: Document implementation

**Team leader role**: When judgment is requested during each feature-dev phase, respond based on the project CLAUDE.md and requirements document.

**Information to pass to the plugin**:
- Project CLAUDE.md content (tech stack, architecture, reference projects, naming conventions)
- Requirements document and task definitions
- Docker configuration (all commands run inside containers)
- TDD required (test-first)

### Step 2A-F: Frontend UI Implementation — frontend-design:frontend-design

When the task involves building a new page or standalone UI component (not wired into existing backend logic), launch `/frontend-design:frontend-design`. This plugin generates production-grade, design-quality frontend code.

**When to use instead of 2A**:
- New dashboard pages, landing pages, or self-contained UI components
- Significant visual redesigns of existing pages

**When to use 2A instead**: Feature implementations that involve backend integration, API wiring, and CRUD logic should go through `feature-dev:feature-dev` (Step 2A).

After generation, the output goes through the same review pipeline (Step 3) as any other implementation.

### Step 2B: Bug Fix / Refactoring

#### 2B-1: Root Cause Analysis (code-explorer)

Launch a `feature-dev:code-explorer` SubAgent via the Agent tool to analyze the problematic code:

```
Agent(subagent_type="feature-dev:code-explorer", prompt="Issue content + relevant files + analysis instructions")
```

#### 2B-2: Fix Implementation

Implement the fix based on code-explorer analysis results.

- Write tests before fixing (TDD)
- Follow the project CLAUDE.md architecture conventions
- Run all commands inside Docker containers

#### 2B-3: Quality Check

After fixing, launch a `feature-dev:code-reviewer` SubAgent via the Agent tool for quality check:

```
Agent(subagent_type="feature-dev:code-reviewer", prompt="Fix content + review scope")
```

### Step 2.5: External Review Comment Handling

When external review comments (from humans or other AI tools) arrive on the PR:

1. Review all comments
2. **Actionable** → Code fix
3. **Ignorable** → Resolve with reason in PR comment
4. Never silently ignore. Explicitly state judgment for all items

### Step 3: PR Review (Plugins + Tech Lead)

After PR creation, execute the following.

#### Step 3a: Plugin Review (Sequential Execution)

Launch the following skills in order:

1. **`/code-review:code-review`** — CLAUDE.md compliance check + git history analysis + bug detection (confidence ≥ 80 only)
2. **`/pr-review-toolkit:review-pr`** — Expert review on test quality / silent failures / type design / code simplification

#### Step 3b: Tech Lead Review (Perspectives Not Covered by Plugins)

After plugin review results, launch the Tech Lead agent (`.claude/agents/tech-lead.md`):

Tech Lead-specific review items:
1. **Security review** (all items in tech-lead.md checklist)
2. **Business logic consistency** — Behavioral consistency with existing systems
3. **API integration verification**: Hit actual APIs with curl and verify response structure
4. Verify frontend display with Playwright MCP

#### Step 3c: Integrated Verdict (Team Leader Role = This Skill)

Integrate plugin review results + Tech Lead review results for final verdict.
**Tech Lead only makes verdicts on their 3 areas (security/API integration/business logic). This skill handles the integrated verdict.**

- **REQUEST_CHANGES**: Security NG / business logic inconsistency / plugin confidence ≥ 80 Critical
- **APPROVE (conditional)**: Important only → re-approve after fix
- **APPROVE**: All checks passed
- Fix → re-review → approve

### Step 4: Final Approval & Merge

After integrated verdict passes:
1. Final check on design principles, conventions, and quality
2. Commit and merge

### Step 5: Task Completion

After approval:
1. If a task list exists, mark the relevant task as completed
2. Close the GitHub Issue
3. Write a brief execution summary to `docs/generated/{project}/requirements/{domain}/develop-log.md` (append, do not overwrite):
   - Date, task/PR, what was implemented
   - Review findings count: plugin (Critical/High/Medium), Tech Lead (pass/fail per area)
   - Final verdict and merge status
4. Proceed to the next task
5. All tasks complete → Move to acceptance testing phase

## Required Checklist Before Completion

Before marking a PR as complete/mergeable, **all** of the following must be satisfied.

### CI Verification
- [ ] All commands listed in the project CLAUDE.md "CI Commands" section pass
- [ ] GitHub Actions CI is **green** (verify with `gh run list`)

### Testing (TDD)
- [ ] Tests written first for new implementations (TDD)
- [ ] When business rules exist (BR-IDs in requirements or business-rules.md), test `describe`/`it` blocks include the corresponding BR-ID (e.g., `describe('BR-V-001: Email validation')`)
- [ ] All CRITICAL / HIGH review findings are **fixed**

### Review
- [ ] Plugin review (`/pr-review-toolkit:review-pr` + `/code-review:code-review`) complete
- [ ] Tech Lead review complete
- [ ] All findings with confidence ≥ 80 addressed

### API ↔ Frontend Type Consistency (When Frontend Changes Exist)
- [ ] Hit actual APIs with curl and obtained response JSON
- [ ] Confirmed frontend type definitions exactly match API responses

### UI Quality (When Frontend Changes Exist)
- [ ] Screenshots taken with Playwright MCP and saved to `docs/generated/{project}/screenshots/develop/{domain}/`
- [ ] Visibility confirmed in both light mode and dark mode

### Walk the Screen as a Person (When Frontend Changes Exist) — **Gate**

Measuring that the change works is not enough. **Before calling anything done**, open the
screen on the real machine and use it the way a person would. A report that does not include
this walk is not accepted, from an agent or from this skill.

- [ ] Opened the screen in its **first state** — nothing selected, no data, empty list — not only
      the state where data is present
- [ ] **Reached the screen the way a user reaches it**, and left it the way a user leaves it:
      followed the link in, pressed **Back**, pressed Back again, reloaded, opened it directly by URL
- [ ] **Pressed the controls** — every button that changed, plus the keys the control implies
- [ ] Did the **whole errand end to end**, not just the changed step (e.g. list → open → act → back to list)
- [ ] **Opened the screenshots and looked at them**, and asked: does this look like a real one of
      these? Compare against a real product of the same kind
- [ ] Anything odd found on the way is **reported even if it is outside the task** — do not fix it
      silently, and do not leave it unsaid

### Comments and the Language of Git Text — **Gate**

Applies to every PR, from this skill and from every agent it delegates to. **Say both of these
in the agent's instructions**; a soft phrasing like "no redundant comments" comes back as prose.

- [ ] **Comments default to zero.** `//` and `/* */` inside code survive in two cases only: a
      `TODO`/`FIXME` for work someone will really do, and **one line** naming a trap a reader
      would fall into (a browser or library bug, a spec-mandated workaround). Design rationale,
      measurements, requirement ids and issue numbers never belong in code — they go in the
      requirements and the design canon
- [ ] **Documentation comments follow the language's own convention**, not an essay. In .NET,
      `///` carries a short `<summary>` of what the member does or represents, and `<param>`,
      `<returns>`, `<exception>` or `<remarks>` only where the signature does not already say it.
      JSDoc/TSDoc likewise. A doc comment that explains why the code was built this way is design
      rationale and moves out of the code like any other
- [ ] **Counted before merge**, not judged by eye (documentation comments are read, not counted):
      `gh pr diff <n> | grep -E '^\+' | grep -vE '^\+\s*(///|/\*\*)' | grep -cE '^\+\s*(//|/\*)'`
      Compare against the repository's own baseline (Carina master is about 0.3%). Generated
      files are excluded: Vela `repository/client/schema.ts`, EF `*.Designer.cs`, model snapshots
      and **the migration `.cs` itself** — `dotnet ef` writes `/// <inheritdoc />` on the class
      and on `Up`/`Down`, and every migration on master carries them. **Never let an agent strip
      those to make the count pass**; a hand-edited generated file comes back as a diff the next
      time it is regenerated. Fix the exclusion, not the file
- [ ] **PR title, PR body and commit messages are Japanese.** Identifiers, test names and
      branch names stay English. A branch already pushed cannot have its commits reworded
      (force-push is refused), so cut a fresh branch rather than leaving English commits
- [ ] **The PR body is bullets and nothing else.** Every line starts with `- `, except the
      trailing generated-with line. No heading, no opening paragraph, no closing summary, no
      test counts and no CI result — the checks report those. Plain form, not polite form.
      Adding a test is one bullet about what it pins, never how many passed. Before posting,
      read the body and delete every line that does not begin with `- `. **Copy this rule
      verbatim into an agent's instructions whenever an agent writes the PR** — an agent that
      is not told will write a preface and a summary every time
- [ ] **The pull request has an assignee before it is announced.** Pass `--assignee fyui001` to
      `gh pr create`. If that is not accepted, assign with the GraphQL mutation
      `addAssigneesToAssignable` (the REST assignee endpoint sits under a repository's issue
      path, so the issue-write hook refuses the whole command). `gh pr edit --add-assignee`
      fails silently, so never trust it. Read the PR back and confirm the field yourself:
      finding a field already correct after the user has fixed it is not a check.
      **Copy this rule verbatim into an agent's instructions whenever an agent opens the PR.**
- [ ] The three content checks in each repository's `CLAUDE.local.md` still apply: the
      implementation only, the standalone-clone test, the leak grep

### Merge
- [ ] **Merge when all checks pass** (CI green + all reviews fixed + all external reviews addressed → autonomous merge)
- [ ] PR always created on a branch

### Command Execution
- [ ] All commands executed inside Docker containers

## PR Lifecycle Management (Strictly Enforced)

**One-domain-at-a-time rule**: Do not proceed to the next domain's PR until the current domain's PR is **merged**.

```
Implementation → CI green check → Plugin review → Tech Lead review → Fix findings
→ CI re-check → External review handling → Final approval → Merge → Next
```

## Notes

- Maximize use of `/feature-dev:feature-dev` for new feature development
- Even for bug fixes, analyze root cause with `code-explorer` before fixing
- Never complete a frontend PR without UI verification
- Never merge with outstanding MUST items
