# SDD Framework Setup Master Template (SDD Commands)

This document serves as the master blueprint for initializing and adapting the **Spec-Driven Development (SDD) Slash Command Suite** to any repository, regardless of language or framework.

> **Execution order:** this template is the **last** setup step. The SDD commands are built on top of the layer skills, so those must already be deployed:
>
> 1. [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) – backend skills (`.claude/skills/fercho-be-*`)
> 2. [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) – frontend skills (`.claude/skills/fercho-fe-*`)
> 3. **This file** – SDD commands (`.claude/commands/sdd-*.md`)
>
> A backend-only or frontend-only project runs just the skill template that applies, then this file. On a new project with no rules file, [init_ia_enviroment_claude.md](./init_ia_enviroment_claude.md) (base project rules) runs before all of them.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Prerequisite Check (Registered Skills)
1. Look for deployed layer skills under `.claude/skills/`: backend `fercho-be-*` and frontend `fercho-fe-*`. A skill with another name counts as a layer skill only if its `SKILL.md` has an `allowed_files_to_modify` section.
2. If **no** layer skills exist, STOP and tell the user to run `init_ia_enviroment_be.md` and/or `init_ia_enviroment_fe.md` first. Do not generate commands.
3. If the repository clearly contains a backend or a frontend whose skills are missing, report it and ask the user whether to continue without that side or to run the missing skill template first.

### Step 2: Skill Inventory & Convention Scanning
1. Read every registered layer skill and build the **Skill Inventory**: skill name, side (backend/frontend), layer, allowed paths, and key constraints.
2. Detect the project conventions the commands depend on:
   - **SDD docs root** (`{{PATH_SDD_DOCS}}`): an existing SDD/specs folder if the project has one; otherwise default to `docs/sdd`.
   - **Ticket-id format** (e.g., `ABC-123`) and how it appears in branch names.
   - **Verification commands:** build, lint, and test commands for each side.
   - **Repository changelog:** whether the project keeps one (e.g., `CHANGELOG.md`) and its format.
3. List any `.claude/commands/sdd-*.md` files that already exist.

### Step 3: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Skill Inventory Table:** Registered Skill vs. Side / Layer / Allowed Paths.
- **Detected Conventions:** SDD docs root, ticket-id format, verification commands, repository changelog.
- **Adjustment Points / Discrepancies:** Missing skills, layers without coverage, or conventions that require custom configuration.
- **Proposed File Generation Plan:** List of commands that will be created in `.claude/commands/`. Mark each existing `sdd-*.md` file as *will be overwritten* so the user can exclude any they have customized.
- **Rules File:** the instruction file where the rules block will be written (section 4). If no earlier run already fixed it, ask the user to choose between `CLAUDE.md` (default), `CLAUDE.local.md`, and `AGENTS.md`. Include any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 4: Deployment
Upon user approval, write the following:
1. `.claude/commands/sdd-*.md` following the file format and specs in sections 2 and 3, with `{{PATH_SDD_DOCS}}`, skill names, and verification commands replaced by the real values. Do not overwrite a command the user excluded.
2. System Instructions patch (section 4): write the rules block between its `fercho-sdd` markers in the rules file resolved in section 4 (`CLAUDE.md` by default).

---

## 2. Shared Conventions

Every generated command must implement these conventions identically.

### 2.1 Command File Format

One file per command under `.claude/commands/`:

```markdown
---
description: <one line shown in the command list>
argument-hint: <arguments, e.g. "<ticket-id>" or "[task-id] [ticket-id]">
---

## Inputs
<how $ARGUMENTS is parsed; how the active ticket is resolved>
## Preconditions
<files and states that must exist; what to tell the user when they don't>
## Steps
## Output
<files written and their structure>
## Stop Conditions
<when to stop and ask instead of continuing>
```

Arguments are read from `$ARGUMENTS`.

### 2.2 Artifacts

All artifacts of a ticket live in `{{PATH_SDD_DOCS}}/{ticket}/`:

| File | Created by | Content |
|---|---|---|
| `requirements.md` | `/sdd-init-feature` | Goal, scope, out of scope, functional requirements (`R01`…), acceptance criteria, non-functional requirements |
| `decisions.md` | `/sdd-init-feature` | Open questions and recorded decisions (section 2.4) |
| `design-architecture.md` | `/sdd-generate-design` | Affected layers and skills (backend and frontend), components to add/change, flow between layers, risks |
| `design-data-api.md` | `/sdd-generate-design` | Data model changes and the API contracts shared by backend and frontend |
| `design-ui.md` | `/sdd-generate-design` | Only when frontend skills are registered: views, components and their props, state, hooks, routes |
| `tasks.md` | `/sdd-generate-tasks` | Task list (section 2.5) |
| `plan-TXX.md` | `/sdd-plan-task` | Execution plan for one task: ordered steps, each naming the layer, the skill to use, and the files it touches |
| `changelog.md` | `/sdd-execute-task` | Per-ticket log: one entry per executed task with files changed and verification result. `/sdd-feature-change` adds one row per change |

`changelog.md` here is the per-ticket log. The repository changelog is updated only if the project keeps one, following its detected format.

### 2.3 Active Ticket Resolution

`/sdd-init-feature` receives the ticket id explicitly. Every other command resolves the ticket in this order:
1. A ticket id passed in the arguments.
2. A ticket id found in the current git branch name that has a folder under `{{PATH_SDD_DOCS}}/`.
3. The only folder under `{{PATH_SDD_DOCS}}/`, if there is exactly one.
4. Otherwise, STOP and ask the user which ticket to use.

Each command states the resolved ticket in its first line of output.

`/sdd-feature-change` uses its argument for something else (a change description) and `/sdd-review-tasks` takes none, so these two start at step 2.

### 2.4 Questions & Decisions

- Ids have no hyphen: requirements are `R01`, `R02`, …; questions are `Q01`, `Q02`, …; decisions are `D01`, `D02`, …. An id is never reused or renumbered.
- Any command that hits missing or ambiguous business information must NOT guess. It records a question in `decisions.md` with the next id, status `open`, the context, the options considered, and the artifacts or tasks it affects.
- Question statuses:
  - `open`: blocks the tasks listed in its "affects" field; work that does not depend on it continues.
  - `answered`: no longer blocks; the answer and its source are recorded.
  - `assumed`: no longer blocks; the working assumption is recorded and must be confirmed before shipping. Plans and reports name the `assumed` questions the work rests on.
- Only `/sdd-answer` and `/sdd-feature-change` change a question's status. An answer the user cannot back with a source is recorded as `assumed`, not `answered`.
- Decisions taken without a question (explicit user instructions) are recorded in the same file with their rationale.

### 2.5 Tasks

- Task ids are `T01`, `T02`, …; the plan file of `T01` is `plan-T01.md`.
- Each task in `tasks.md` has: id, title, the requirement ids it covers, the owning skill(s), dependencies on other tasks, blocking question ids, acceptance criteria, and a status.
- Statuses: `todo` -> `in-progress` -> `review` -> `done`; `blocked` while it has an `open` question or an unfinished dependency.
  - `todo`: not started. There is no separate "planned" status: a planned task is a `todo` task that has a `plan-TXX.md`.
  - `in-progress`: set by `/sdd-execute-task` when it starts the task.
  - `review`: optional, set by hand when the work is implemented and waiting for review.
  - `done`: set by `/sdd-execute-task` only when verification passes.
  - `blocked`: set by `/sdd-generate-tasks` and `/sdd-feature-change`; `/sdd-answer` returns the task to `todo` when nothing else blocks it.
- A task that will not be built is `deferred` or cancelled (struck through in `tasks.md`). Both count as finished, never as remaining work.
- `/sdd-execute-task` owns the `in-progress` and `done` transitions; `/sdd-plan-task` and `/sdd-review-tasks` never change a status.
- When `[task-id]` is omitted, `/sdd-plan-task` uses the first `todo` task in order that has no `plan-TXX.md`, and `/sdd-execute-task` uses the `in-progress` task (a resumed session) or else the first `todo` task that has a `plan-TXX.md`. The command names the task it picked before doing anything else.

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
| `/sdd-feature-change <change description>` | Apply a scope or design change to the existing artifacts in place | Updated `requirements.md`, `decisions.md`, design files, `tasks.md`, `changelog.md` |
| `/sdd-review-tasks` | Read-only status report and next-task recommendation | Report in the chat; no file is written |

The first five commands are the main flow, in order. The other three are supporting commands, run when needed.

### 3.1 `/sdd-init-feature <ticket-id>`
- **Preconditions:** a ticket id is given. If `{{PATH_SDD_DOCS}}/{ticket}/` already exists, STOP and ask whether to update it; never overwrite silently.
- **Steps:**
  1. Gather the feature description from the user's message and any ticket content they provide.
  2. Write `requirements.md` with the structure in section 2.2.
  3. Write `decisions.md`, recording every gap or ambiguity as an `open` question.
- **Output:** the two files, plus a summary listing the open questions.

### 3.2 `/sdd-generate-design [ticket-id]`
- **Preconditions:** `requirements.md` exists.
- **Steps:**
  1. Read `requirements.md`, `decisions.md`, and the Skill Inventory.
  2. Inspect the existing code in the affected layers before designing; reuse existing components and patterns.
  3. Write the design files, organized per registered layer and naming the real skill and paths for each change.
  4. Write `design-ui.md` only when frontend skills are registered and the feature has frontend work.
  5. Record new questions in `decisions.md`; mark the design sections that depend on an `open` or `assumed` question as provisional.
- **Stop Conditions:** an `open` question that makes the overall architecture undecidable.

### 3.3 `/sdd-generate-tasks [ticket-id]`
- **Preconditions:** `design-architecture.md` exists.
- **Steps:**
  1. Split the design into vertical slices: each task delivers a testable increment and may span backend and frontend skills.
  2. Tag every task with the skill(s) that own it, its dependencies, and the requirement ids it covers.
  3. Set every task to `todo`, except the tasks affected by an `open` question, which are `blocked`.
  4. Check that every requirement is covered by at least one task; report any that is not.
- **Output:** `tasks.md` with the structure in section 2.5.

### 3.4 `/sdd-plan-task [task-id] [ticket-id]`
- **Preconditions:** `tasks.md` exists; the task is `todo`. If it already has a `plan-TXX.md`, ask whether to replace it.
- **Steps:**
  1. Read the task, the design sections it refers to, and the `SKILL.md` of each owning skill.
  2. Inspect the files the task will touch.
  3. Write `plan-TXX.md` as ordered implementation steps. Every step names its layer, the one skill to use for it (its real name from the Skill Inventory), and the files it creates or modifies, all within that skill's `allowed_files_to_modify`. A change that needs two skills is two steps. A step that no skill covers (e.g., documentation, configuration) is marked `Skill: none`.
  4. Add to the plan the tests to write (inside the step of the skill that owns them), the verification commands to run, and the `assumed` questions the task relies on.
- **Stop Conditions:** the task needs a file outside every registered skill's allowed paths (report it and ask); the task is `blocked` (name the question or dependency).
- **This command does not modify source code and does not change the task status;** the task stays `todo` until `/sdd-execute-task` starts it.

### 3.5 `/sdd-execute-task [task-id] [ticket-id]`
- **Preconditions:** `plan-TXX.md` exists; the task is `todo` or `in-progress`; none of its questions is `open`; its dependencies are `done`.
- **Steps:**
  1. Set the task status to `in-progress`. If it already is, resume from the first plan step with no entry in `changelog.md`.
  2. Implement the plan steps in order, including the tests. Before each step, load the skill that step names and follow its constraints and workflow; do not choose a different skill than the plan's. For a step marked `Skill: none`, print the fallback notice defined by the layer rules.
  3. Keep each step's edits inside the files the plan lists for it. A needed file outside them is a stop condition, not a reason to widen the step.
  4. Run the verification commands from the plan and fix failures caused by the change.
  5. Append the entry to the per-ticket `changelog.md`; update the repository changelog if the project keeps one.
  6. Set the task status to `done` only when verification passes; otherwise leave it `in-progress` and report what fails.
- **Stop Conditions:** the plan is wrong or incomplete for the real code (report the deviation instead of improvising beyond the plan); a new business question appears (record it and stop the affected part).
- **This command does not commit or push;** the user decides when to do that.

### 3.6 `/sdd-answer <Q-id> <answer>`
- **Preconditions:** the question exists in `decisions.md` and is `open` or `assumed`. If it is already `answered`, STOP and ask whether to revise the decision.
- **Steps:**
  1. Record the answer, its source, and the date. Set the question to `answered`, or to `assumed` when the answer is a working assumption without a confirmed source. Keep the id.
  2. Update every artifact in its "affects" field: requirements, provisional design sections, and tasks.
  3. Remove the question from the blocking list of its tasks; set to `todo` the `blocked` tasks with nothing else blocking them.
  4. If a task that has a `plan-TXX.md`, or is `in-progress`, `review`, or `done`, is affected, report it and say which plan must be regenerated or which code must be revisited; do not change code.
  5. When the answer widens the scope or contradicts work already `done` (including an `assumed` question whose assumption proved wrong), record the answer and hand off to `/sdd-feature-change` with a ready description instead of editing the artifacts.
- **Output:** a summary of the updated files and the tasks that were unblocked.

### 3.7 `/sdd-feature-change <change description>`
- **When to use:** any change to what the ticket builds or how: a scope change or addition, an answer that widens scope (`/sdd-answer` hands off here), review findings, or a design that proves wrong in code.
- **Preconditions:** `requirements.md`, the design files, `tasks.md`, and `decisions.md` exist; a change description is given (ask for it otherwise).
- **Steps:** update the artifacts in this order, editing in place and never regenerating a file:
  1. `requirements.md`: add the new scope and deprecate superseded requirements. Leave it unchanged for a technical-only change.
  2. `decisions.md`: record new ambiguities as `open` questions with the tasks they block; set to `answered` the questions the change settles, with the answer and its source.
  3. Design files: change only the affected sections. Existing section and step numbers never change; new ones take the next free number.
  4. `tasks.md`: never edit a `done` task (rework is a new task). Strike through obsolete tasks as cancelled. Add new tasks with the next free ids, as `todo` or `blocked`. A changed task that already has a `plan-TXX.md` has a stale plan: say so and recommend `/sdd-plan-task TXX`.
  5. `changelog.md`: add one row summarizing the change (requirements, design sections, tasks, questions).
- **Output:** an alignment report listing what changed in each artifact and the first task to work on next.
- **Stop Conditions:** a required artifact is missing; a business rule the change needs is not stated (record an `open` question instead of guessing).
- **This command does not modify source code.**

### 3.8 `/sdd-review-tasks`
- **Preconditions:** `tasks.md` exists. If `decisions.md` is missing, say so and treat every blocking question as `open`.
- **Steps:**
  1. Read each task's id, dependencies, blocking questions, and status from `tasks.md`, and each question's status from `decisions.md`.
  2. Consistency check: report any task whose checkbox and status disagree, any `blocked` task whose questions are all `answered` or `assumed`, and any blocking id missing from `decisions.md`.
  3. Classify the tasks: finished (`done`, `deferred`, or cancelled), blocked (an unfinished dependency or an `open` question), or available.
  4. Recommend the next available task in order, naming any `assumed` question it rests on. If nothing is available, list the `open` questions that would unblock the most tasks and point to `/sdd-answer`.
- **Output:** a report in the chat with completed and remaining counts, the pending tasks with their state, the inconsistencies found, the recommended task, and the command to run next.
- **This command is read-only;** it writes no file and changes no status.

---

## 4. System Prompt Patch (`CLAUDE.md` by default - SDD)

The SDD rules are written to the same instruction file as the layer rules added by the skill templates: `CLAUDE.md` unless the user chose another one.

**Write rules:**
1. **Target file:** resolve it in this order:
   1. If `CLAUDE.md`, `CLAUDE.local.md`, or `AGENTS.md` already contains a `fercho-*` marker block from an earlier run of these templates, use that same file and do not ask again.
   2. Otherwise, ask the user in the confirmation step where the rules should be written, offering these options:
      - **`CLAUDE.md`** (default): project rules loaded by Claude Code and shared with the team through version control.
      - **`CLAUDE.local.md`**: personal rules for this machine only. They are not shared with the team; make sure the file is listed in `.gitignore`.
      - **`AGENTS.md`**: one rules file shared with other AI assistants. Claude Code does not read it directly, so `CLAUDE.md` must import it (see rule 2).
   3. If the user approves without choosing, use `CLAUDE.md`.

   The file lives at the repository root. Create it if it does not exist.
2. **Import (only when the target is `AGENTS.md`):** `CLAUDE.md` must contain the line `@AGENTS.md`. If `CLAUDE.md` does not exist, create it with that line. If it exists without the import, add the line and leave the rest of its content untouched.
3. **Markers:** the block below is delimited by `<!-- fercho-sdd:start -->` and `<!-- fercho-sdd:end -->`. If the target file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
4. **Block order:** when several blocks are present, keep them in the order `fercho-base`, `fercho-be`, `fercho-fe`, `fercho-sdd`.
5. **Legacy rules:** if `CLAUDE.md`, `CLAUDE.local.md`, or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace `{{PATH_SDD_DOCS}}` with the resolved path):

```markdown
<!-- fercho-sdd:start -->
## SDD Workflow Rules

1. Feature work follows the SDD flow: `/sdd-init-feature` -> `/sdd-generate-design` -> `/sdd-generate-tasks` -> `/sdd-plan-task` -> `/sdd-execute-task`. Each step requires the artifacts of the previous one.
2. SDD artifacts live under `{{PATH_SDD_DOCS}}/{ticket}/`. Do not implement a task without its `plan-TXX.md`, and do not execute a task blocked by an `open` question.
3. Missing or ambiguous business information is recorded as a question in `decisions.md` and resolved with `/sdd-answer`. Never guess business logic. Work that rests on an `assumed` question says so, and the assumption is confirmed before shipping.
4. Follow the Information Lookup Hierarchy: SDD Docs -> Project Docs -> Existing Code -> External Specs.
5. **Skills in plans:** `/sdd-plan-task` names the layer skill for every step of `plan-TXX.md` and keeps the step inside that skill's `allowed_files_to_modify`. `/sdd-execute-task` loads the skill each step names before implementing it and does not substitute another.
6. Supporting commands: `/sdd-review-tasks` for the current status and the next task, `/sdd-feature-change` for any change of scope or design (never edit the artifacts by hand for that).
<!-- fercho-sdd:end -->
```
