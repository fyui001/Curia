# Acceptance Testing Skill

## Description
Launch the acceptance testing phase. PdM uses Playwright MCP to actually operate screens, verify the entire business flow, and provide feedback.

## Usage
```
/acceptance <project> <domain>
```

### Examples
```
/acceptance MyProject user
/acceptance MyProject property
```

## Instructions

When this skill is invoked, start the acceptance testing phase.

### Step 0: Prerequisites Check

1. Read `src/{project}/CLAUDE.md` to understand project-specific information
2. Verify all tasks in the task list (`docs/generated/{project}/requirements/{domain}/tasks.md`) are completed
3. If incomplete tasks exist, report to the user and abort
4. Read the requirements document (`docs/generated/{project}/requirements/{domain}/requirements.md`) and review acceptance criteria

### Step 1: PdM Screen Operation Testing

Launch the PdM agent (`.claude/agents/pdm.md`) to perform the following:

#### 1.1 Business Flow Testing

Execute the following scenarios via Playwright MCP according to business manuals:

- **CRUD operations**: List → Create → Detail → Edit → Delete
- **Search & filter**: Filtering with each search condition
- **Pagination**: Page navigation, count display
- **Validation**: Required fields empty, invalid values, character limit exceeded
- **Permissions**: Role-based access control

#### 1.2 Comparison with Existing System

Using Playwright MCP:

1. Take screenshots of frontend screens
2. Compare with the existing system on the following points:
   - **Information volume**: Is the same information displayed?
   - **Layout**: Is information placement appropriate? (Design system-aligned improvements are acceptable)
   - **Usability**: Button placement, navigation flow, confirmation dialogs
   - **Error display**: Error message content and display position

#### 1.3 Edge Case Testing

- Screen display with empty data
- Performance with large datasets
- Display of long strings (truncation)

### Step 2: Feedback Report Creation

Create test results based on the `docs/templates/acceptance-test.md` template:

Save to: `docs/generated/{project}/requirements/{domain}/acceptance-test.md`

### Step 3: Feedback Handling

1. If Critical / High findings exist:
   - Fix via `/develop` skill (code-explorer analysis → fix → plugin review)
   - PdM re-tests after fix completion
2. Medium and below are recorded only (can be addressed later)
3. All Critical / High resolved → **Acceptance approved**

### Step 4: Domain Completion

After acceptance approval:
1. Update the acceptance test report's overall verdict to PASS
2. Append an acceptance summary to `docs/generated/{project}/requirements/{domain}/develop-log.md`:
   - Date, tester, overall verdict
   - Findings count by severity (Critical/High/Medium/Low)
   - Re-test rounds needed
3. Report to the user that the next domain's requirements definition phase can begin

## Notes

- PdM tests **from the user's perspective** (delegate technical issues to Tech Lead)
- Clearly classify differences from the existing system as "bugs" vs "intentional improvements"
- Save all screenshots to `docs/generated/{project}/screenshots/acceptance/{domain}/` — **never to the repository root**
