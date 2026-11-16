# SDD Framework Setup Master Template (SDD Prompt Files for GitHub Copilot)

This document serves as the master blueprint for initializing and adapting the **Spec-Driven Development (SDD) Slash Command Suite** to any repository using **GitHub Copilot**, regardless of language or framework. Each command is a Copilot prompt file.

> **Execution order:** this template is the **last** setup step. The SDD commands are built on top of the layer instructions, so those must already be deployed:
>
> 1. [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) – backend layer instructions (`.github/instructions/fercho-be-*.instructions.md`)
> 2. [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) – frontend layer instructions (`.github/instructions/fercho-fe-*.instructions.md`)
> 3. **This file** – SDD prompt files (`.github/prompts/sdd-*.prompt.md`)
>
> A backend-only or frontend-only project runs just the layer template that applies, then this file.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Prerequisite Check (Registered Layer Instructions)
1. Look for deployed layer instructions under `.github/instructions/`: backend `fercho-be-*.instructions.md` and frontend `fercho-fe-*.instructions.md`. An instruction file with another name counts as a layer instruction only if its `applyTo` is scoped to specific source folders (not `**`).
2. If **no** layer instructions exist, STOP and tell the user to run `init_ia_enviroment_be.md` and/or `init_ia_enviroment_fe.md` first. Do not generate prompt files.
3. If the repository clearly contains a backend or a frontend whose layer instructions are missing, report it and ask the user whether to continue without that side or to run the missing layer template first.

### Step 2: Layer Inventory & Convention Scanning
1. Read every registered layer instruction and build the **Layer Inventory**: instruction name, side (backend/frontend), layer, `applyTo` paths, and key constraints. Include the `fercho-*-bugfix-refactor` prompt files.
2. Detect the project conventions the commands depend on:
   - **SDD docs root** (`{{PATH_SDD_DOCS}}`): an existing SDD/specs folder if the project has one; otherwise default to `docs/sdd`.
   - **Ticket-id format** (e.g., `ABC-123`) and how it appears in branch names.
   - **Verification commands:** build, lint, and test commands for each side.
   - **Repository changelog:** whether the project keeps one (e.g., `CHANGELOG.md`) and its format.
3. List any `.github/prompts/sdd-*.prompt.md` files that already exist.

### Step 3: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Layer Inventory Table:** Registered Instruction vs. Side / Layer / `applyTo` Paths.
- **Detected Conventions:** SDD docs root, ticket-id format, verification commands, repository changelog.
- **Adjustment Points / Discrepancies:** Missing layer instructions, layers without coverage, or conventions that require custom configuration.
- **Proposed File Generation Plan:** List of prompt files that will be created in `.github/prompts/`. Mark each existing `sdd-*.prompt.md` file as *will be overwritten* so the user can exclude any they have customized.
- **Instruction File Changes:** what will be written to `.github/copilot-instructions.md` (section 4), including any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 4: Deployment
Upon user approval, write the following:
1. `.github/prompts/sdd-*.prompt.md` following the file format and specs in sections 2 and 3, with `{{PATH_SDD_DOCS}}`, instruction names, and verification commands replaced by the real values. Do not overwrite a prompt file the user excluded.
2. System Instructions patch (section 4): write the rules block to `.github/copilot-instructions.md` between its `fercho-sdd` markers.

---

## 2. Shared Conventions

Every generated command must implement these conventions identically.

### 2.1 Prompt File Format

One prompt file per command under `.github/prompts/`, named `<command>.prompt.md` (e.g., `sdd-init-feature.prompt.md`). The user runs it in Copilot Chat as `/<command>`:

```markdown
---
description: <one line shown in the prompt list>
argument-hint: <arguments, e.g. "<ticket-id>" or "[task-id] [ticket-id]">
agent: agent
---

## Inputs
<how the arguments are parsed; how the active ticket is resolved>
## Preconditions
<files and states that must exist; what to tell the user when they don't>
## Steps
## Output
<files written and their structure>
## Stop Conditions
<when to stop and ask instead of continuing>
```

- The arguments are the text the user types after the command in the chat.
- `agent: agent` runs the prompt in agent mode, which the commands need in order to read and write files. On older VS Code versions the same field is named `mode`; use the one the installed version supports.

### 2.2 Artifacts

All artifacts of a ticket live in `{{PATH_SDD_DOCS}}/{ticket}/`:

| File | Created by | Content |
|---|---|---|
| `requirements.md` | `/sdd-init-feature` | Goal, scope, out of scope, functional requirements (`R-01`…), acceptance criteria, non-functional requirements |
| `decisions.md` | `/sdd-init-feature` | Open questions and recorded decisions (section 2.4) |
| `design-architecture.md` | `/sdd-generate-design` | Affected layers and their instructions (backend and frontend), components to add/change, flow between layers, risks |
| `design-data-api.md` | `/sdd-generate-design` | Data model changes and the API contracts shared by backend and frontend |
| `design-ui.md` | `/sdd-generate-design` | Only when frontend layer instructions are registered: views, components and their props, state, hooks, routes |
| `tasks.md` | `/sdd-generate-tasks` | Task list (section 2.5) |
| `plan-TXX.md` | `/sdd-plan-task` | Execution plan for one task |
| `changelog.md` | `/sdd-execute-task` | Per-ticket log: one entry per executed task with files changed and verification result |

`changelog.md` here is the per-ticket log. The repository changelog is updated only if the project keeps one, following its detected format.

### 2.3 Active Ticket Resolution

`/sdd-init-feature` receives the ticket id explicitly. Every other command resolves the ticket in this order:
1. A ticket id passed in the arguments.
2. A ticket id found in the current git branch name that has a folder under `{{PATH_SDD_DOCS}}/`.
3. The only folder under `{{PATH_SDD_DOCS}}/`, if there is exactly one.
4. Otherwise, STOP and ask the user which ticket to use.

Each command states the resolved ticket in its first line of output.

### 2.4 Questions & Decisions

- Any command that hits missing or ambiguous business information must NOT guess. It records a question in `decisions.md` with the next id (`Q-01`, `Q-02`, …), status `Open`, the context, the options considered, and the artifacts or tasks it affects.
- A question blocks only the tasks listed in its "affects" field; work that does not depend on it continues.
- `/sdd-answer` is the only command that changes a question to `Answered`.
- Decisions taken without a question (explicit user instructions) are recorded in the same file as `D-01`, `D-02`, … with their rationale.

### 2.5 Tasks

- Task ids are `T01`, `T02`, …; the plan file of `T01` is `plan-T01.md`.
- Each task in `tasks.md` has: id, title, the requirement ids it covers, the owning layer instruction(s), dependencies on other tasks, blocking question ids, acceptance criteria, and a status.
- Statuses: `Pending` -> `Planned` -> `In Progress` -> `Done`; `Blocked` while it has an `Open` question or an unfinished dependency.
- When `[task-id]` is omitted, the command uses the first task in order that is not `Done` and not `Blocked` (and, for `/sdd-execute-task`, that is `Planned`). It names the task it picked before doing anything else.

---

## 3. SDD Slash Command Suite Spec

| Command | Objective | Output File / Artifact |
|---|---|---|
| `/sdd-init-feature <ticket-id>` | Initialize requirements and decision registry | `requirements.md`, `decisions.md` |
| `/sdd-generate-design [ticket-id]` | Create architecture, data/API, and UI design from specs | `design-architecture.md`, `design-data-api.md`, `design-ui.md` |
| `/sdd-generate-tasks [ticket-id]` | Break down design into vertical-slice task units | `tasks.md` |
| `/sdd-plan-task [task-id] [ticket-id]` | Generate isolated execution plan for a single task | `plan-TXX.md` |
| `/sdd-execute-task [task-id] [ticket-id]` | Implement code changes defined in `plan-TXX.md` | Source code files + `changelog.md` |
| `/sdd-answer <Q-id> <answer>` | Unblock pending tasks by answering an open decision | Updated `decisions.md` & affected artifacts |

### 3.1 `/sdd-init-feature <ticket-id>`
- **Preconditions:** a ticket id is given. If `{{PATH_SDD_DOCS}}/{ticket}/` already exists, STOP and ask whether to update it; never overwrite silently.
- **Steps:**
  1. Gather the feature description from the user's message and any ticket content they provide.
  2. Write `requirements.md` with the structure in section 2.2.
  3. Write `decisions.md`, recording every gap or ambiguity as an `Open` question.
- **Output:** the two files, plus a summary listing the open questions.

### 3.2 `/sdd-generate-design [ticket-id]`
- **Preconditions:** `requirements.md` exists.
- **Steps:**
  1. Read `requirements.md`, `decisions.md`, and the Layer Inventory.
  2. Inspect the existing code in the affected layers before designing; reuse existing components and patterns.
  3. Write the design files, organized per registered layer and naming the real layer instruction and paths for each change.
  4. Write `design-ui.md` only when frontend layer instructions are registered and the feature has frontend work.
  5. Record new questions in `decisions.md`; mark the design sections that depend on an `Open` question as provisional.
- **Stop Conditions:** an `Open` question that makes the overall architecture undecidable.

### 3.3 `/sdd-generate-tasks [ticket-id]`
- **Preconditions:** `design-architecture.md` exists.
- **Steps:**
  1. Split the design into vertical slices: each task delivers a testable increment and may span backend and frontend layers.
  2. Tag every task with the layer instruction(s) that own it, its dependencies, and the requirement ids it covers.
  3. Mark as `Blocked` every task affected by an `Open` question.
  4. Check that every requirement is covered by at least one task; report any that is not.
- **Output:** `tasks.md` with the structure in section 2.5.

### 3.4 `/sdd-plan-task [task-id] [ticket-id]`
- **Preconditions:** `tasks.md` exists; the task is not `Blocked` or `Done`.
- **Steps:**
  1. Read the task, the design sections it refers to, and the instruction file of each owning layer.
  2. Inspect the files the task will touch.
  3. Write `plan-TXX.md`: layer instruction(s) to apply, files to create/modify (all within each instruction's `applyTo` paths), ordered implementation steps, tests to write, and the verification commands to run.
  4. Set the task status to `Planned`.
- **Stop Conditions:** the task needs a file outside every registered layer instruction's `applyTo` paths (report it and ask); the task is `Blocked` (name the question or dependency).
- **This command does not modify source code.**

### 3.5 `/sdd-execute-task [task-id] [ticket-id]`
- **Preconditions:** `plan-TXX.md` exists; the task is `Planned`; none of its questions is `Open`; its dependencies are `Done`.
- **Steps:**
  1. Set the task status to `In Progress`.
  2. Read the instruction file(s) named in the plan before editing and follow their constraints and workflow; do not rely only on Copilot attaching them automatically. When a step matches no layer instruction, print the fallback notice defined by the layer rules.
  3. Implement the plan steps in order, including the tests.
  4. Run the verification commands from the plan and fix failures caused by the change.
  5. Append the entry to the per-ticket `changelog.md`; update the repository changelog if the project keeps one.
  6. Set the task status to `Done` only when verification passes; otherwise leave it `In Progress` and report what fails.
- **Stop Conditions:** the plan is wrong or incomplete for the real code (report the deviation instead of improvising beyond the plan); a new business question appears (record it and stop the affected part).
- **This command does not commit or push;** the user decides when to do that.

### 3.6 `/sdd-answer <Q-id> <answer>`
- **Preconditions:** the question exists in `decisions.md` and is `Open`. If it is already `Answered`, STOP and ask whether to revise the decision.
- **Steps:**
  1. Record the answer and set the question to `Answered`.
  2. Update every artifact in its "affects" field: requirements, provisional design sections, and tasks.
  3. Remove the question from the blocking list of its tasks; set to `Pending` the tasks with nothing else blocking them.
  4. If a task already `Planned` or `Done` is affected, report it and say which plan must be regenerated or which code must be revisited; do not change code.
- **Output:** a summary of the updated files and the tasks that were unblocked.

---

## 4. Global Instructions Patch (`.github/copilot-instructions.md` - SDD)

The SDD rules are written to `.github/copilot-instructions.md`, next to the layer rules added by the layer instruction templates. GitHub Copilot loads this file for every request in the repository.

**Write rules:**
1. **Target file:** `.github/copilot-instructions.md`. Create it if it does not exist.
2. **Markers:** the block below is delimited by `<!-- fercho-sdd:start -->` and `<!-- fercho-sdd:end -->`. If the file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
3. **Block order:** when several blocks are present, keep them in the order `fercho-be`, `fercho-fe`, `fercho-sdd`.
4. **Legacy rules:** if `.github/copilot-instructions.md` or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace `{{PATH_SDD_DOCS}}` with the resolved path):

```markdown
<!-- fercho-sdd:start -->
## SDD Workflow Rules

1. Feature work follows the SDD flow: `/sdd-init-feature` -> `/sdd-generate-design` -> `/sdd-generate-tasks` -> `/sdd-plan-task` -> `/sdd-execute-task`. Each step requires the artifacts of the previous one.
2. SDD artifacts live under `{{PATH_SDD_DOCS}}/{ticket}/`. Do not implement a task without its `plan-TXX.md`, and do not execute a task blocked by an `Open` question.
3. Missing or ambiguous business information is recorded as a question in `decisions.md` and resolved with `/sdd-answer`. Never guess business logic.
4. Follow the Information Lookup Hierarchy: SDD Docs -> Project Docs -> Existing Code -> External Specs.
<!-- fercho-sdd:end -->
```
