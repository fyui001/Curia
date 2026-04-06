# Requirements Definition Skill

## Description
Launch the requirements definition phase. Integrate existing system analysis by plugins (code-explorer / code-architect) with PdM's business judgment to produce requirements documents.

## Usage
```
/requirements <project> <domain>
```

### Examples
```
/requirements MyProject user
/requirements MyProject billing
```

## Instructions

When this skill is invoked, start the requirements definition phase.

### Step 0: Load Project Information

1. Read `src/{project}/CLAUDE.md` to understand the tech stack, directory structure, reference projects, and existing system information
2. Check the target domain's position and dependencies from the "Implementation Phases" section in the project CLAUDE.md
3. Verify whether prerequisite domains have been defined in `docs/generated/{project}/requirements/`
4. Create the `docs/generated/{project}/requirements/{domain}/` directory

### Step 1: Existing System Analysis via code-explorer (Parallel)

Launch 2-3 `feature-dev:code-explorer` SubAgents in parallel via the Agent tool, analyzing the existing system from different perspectives:

```
Agent(subagent_type="feature-dev:code-explorer", prompt="...")  # Explorer A: Business logic & data flow
Agent(subagent_type="feature-dev:code-explorer", prompt="...")  # Explorer B: Related domains, screen structure, Views
Agent(subagent_type="feature-dev:code-explorer", prompt="...")  # Explorer C: Auth & permissions (when needed)
```

Each Explorer returns a list of important files (5-10) and an analysis summary.

### Step 2: Architecture Analysis via code-architect

Launch a `feature-dev:code-architect` SubAgent via the Agent tool to extract:

```
Agent(subagent_type="feature-dev:code-architect", prompt="...")
```

- Architecture blueprint for the implementation target
- List of files to create/modify
- Data flow
- Pattern mapping with reference projects

### Step 3: Supplementation & Verification by Tech Lead

After receiving Explorer / Architect results, launch the Tech Lead agent (`.claude/agents/tech-lead.md`) to supplement:

- Comprehensively extract business rule edge cases (month-end dates, null tolerance, permission boundaries)
- External dependencies (relationships with other domains)
- Business-specific constraints that plugins miss

### Step 4: PdM — Business Judgment (Parallel with Steps 1-2)

In parallel with Steps 1-2, launch the PdM agent (`.claude/agents/pdm.md`) (Step 3 depends on Steps 1-2 results, so it runs after):

- Read the relevant domain information from business manuals
- Organize business value and purpose
- Draft user story outlines

### Step 5: Consensus & Requirements Document Creation

Integrate all results from Steps 1-2, Step 3 (TechLead), and Step 4 (PdM), then create the following documents based on `docs/templates/` templates:

- `docs/generated/{project}/requirements/{domain}/requirements.md` — Requirements document
- `docs/generated/{project}/requirements/{domain}/tasks.md` — Task list
- `docs/generated/{project}/requirements/{domain}/business-rules.md` — Business rules (**optional**: only create when the domain has complex rules such as calculations, state machines, or permission matrices. For simple CRUD domains, include rules inline in requirements.md)

### Step 6: User Approval

Present the requirements documents to the user and await approval.
Address any revisions, then mark as complete upon approval.

## Notes

- Plugin analysis results are **fact-based**. Do not write requirements based on speculation
- Tech Lead should focus on supplementing edge cases that plugins miss
- PdM determines scope and priority from a business value perspective
