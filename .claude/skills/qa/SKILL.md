---
name: qa
description: Ad-hoc cross-cutting QA for a portal, outside the main /requirements→/develop→/acceptance process — parallel PdM full-screen verification + plugin/Tech Lead code review, consensus prioritization, and Issue creation. Invoke with /qa <project> <portal>.
---

# QA Skill

## Description
Launch cross-cutting QA. Execute PdM full-screen functional verification and plugin + Tech Lead code quality review in parallel, then determine priority through consensus.

**Note**: This skill is NOT part of the development process (`/requirements → /develop → /acceptance`). It is an ad-hoc skill to be executed as needed.

## Usage
```
/qa <project> <portal>
```

### Examples
```
/qa MyProject admin
/qa MyProject owner
```

## Instructions

When this skill is invoked, start the cross-cutting QA phase.

### Step 0: Prerequisites Check

1. Read `src/{project}/CLAUDE.md` to understand project-specific information
2. Create an Epic Issue on GitHub if one doesn't exist
3. Get the full screen list for the target portal from the project's directory structure
4. Verify Docker containers are running (`docker compose ps`)

### Step 1: Launch PdM QA and Code Review in Parallel

Create **2 Sub-issues** under the Epic and execute in parallel.

---

#### Sub-issue A: PdM Review — Full Feature Verification + UI/UX Refinement

Launch the PdM agent (`.claude/agents/pdm.md`) to perform:

##### A-1. Full Screen Functional Verification (Existing System Comparison)

Check the following for each screen:
1. **Functional operation**: Does CRUD work correctly?
2. **Data consistency**: Is the same data displayed as in the existing system?
3. **Operation flow**: Does the same operation produce the same result?
4. **Validation**: Are errors for required fields and invalid values appropriate?
5. **Permissions**: Is role-based access control correct?
6. **Form input components**: Verify all selects/date inputs/linked selects through actual operation

##### A-2. Cross-screen UI/UX Refinement

Review cross-screen consistency based on the PdM agent's "Cross-cutting QA Review Criteria."

##### A-3. Deliverables

Screenshots: save to `docs/generated/{project}/screenshots/qa/{portal}/`
Report: create based on the `docs/templates/qa-pdm-review.md` template:
Save to: `docs/generated/{project}/qa/{portal}/pdm-review.md`

---

#### Sub-issue B: Code Quality Review — Plugins + Tech Lead

##### B-1. Plugin Review (Sequential Execution)

Launch the following skills:

1. **`/pr-review-toolkit:review-pr`** — All perspectives: test quality / silent failures / type design / code simplification / code quality
2. **`/code-review:code-review`** — CLAUDE.md compliance + git history analysis + bug detection

##### B-2. Supplementary Review by Tech Lead (Based on Plugin Results)

Launch the Tech Lead agent (`.claude/agents/tech-lead.md`) to cover areas plugins don't:

1. **Security** — Authorization, data filtering, transactions
2. **Business logic consistency** — Behavioral consistency with existing systems
3. **API integration verification** — Verify response structure with curl
4. **Technical debt inventory** — TODO/HACK comments

##### B-3. Deliverables

Integrate plugin review results + Tech Lead review results and create based on the `docs/templates/qa-engineer-review.md` template:
Save to: `docs/generated/{project}/qa/{portal}/engineer-review.md`

---

### Step 2: Consensus — Priority Sorting & Issue Creation

After both PdM QA and Code Review are complete:

1. **Report integration**: List all findings from both reports
2. **Deduplication**: Merge when PdM and plugins/Tech Lead flag the same issue
3. **Priority determination**: Critical > High > Medium > Low
4. **Dependency analysis**: Identify fix dependencies
5. **Issue creation**: Create each finding as a GitHub Issue (Sub-issue of Epic)
   - Labels: `qa`, `bug` or `enhancement` or `refactor`
   - Priority labels: `priority:critical`, `priority:high`, `priority:medium`, `priority:low`
6. **GitHub Project management**: Add Issues to the Project and set appropriate statuses

### Step 3: Fix Execution

Execute Issues lined up in Ready from top to bottom via the `/develop` skill.
Follow the normal development process (implementation → review → merge).

## Notes

- PdM should actually operate the existing system for comparison
- Respect plugin review results; Tech Lead should focus on perspectives that don't overlap with plugins
- In consensus, determine priority by "user value" rather than "technical correctness"
- Fix all Critical/High before proceeding to release judgment
