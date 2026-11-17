# SDD Framework Setup Master Template (SDD Prompt Files for GitHub Copilot)

This document is the master blueprint for initializing the **Spec-Driven Development (SDD) Slash Command Suite** in any repository using **GitHub Copilot**, regardless of language or framework. Each command is a Copilot prompt file.

> **Execution order:** this template is the **last** setup step. The SDD commands build on the layer instructions, which must already be deployed:
>
> 1. [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) – backend layer instructions (`.github/instructions/fercho-be-*.instructions.md`)
> 2. [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) – frontend layer instructions (`.github/instructions/fercho-fe-*.instructions.md`)
> 3. **This file** – SDD prompt files (`.github/prompts/sdd-*.prompt.md`)
>
> A backend-only or frontend-only project runs just the layer template that applies, then this file. On a new project with no instructions file, [init_ia_enviroment_copilot.md](./init_ia_enviroment_copilot.md) (base project rules) runs first.

---

## 1. Setup Agent Execution Protocol

When this file is provided with a request to initialize or configure a project, execute this workflow.

### Step 1: Prerequisite Check
1. Look for deployed layer instructions under `.github/instructions/`: backend `fercho-be-*.instructions.md` and frontend `fercho-fe-*.instructions.md`. A file with another name counts as a layer instruction only if its `applyTo` is scoped to specific source folders (not `**`).
2. If **no** layer instructions exist, STOP and tell the user to run `init_ia_enviroment_be.md` and/or `init_ia_enviroment_fe.md` first. Do not generate prompt files.
3. If the repository clearly contains a backend or frontend whose layer instructions are missing, report it and ask whether to continue without that side or run the missing layer template first.

### Step 2: Layer Inventory & Convention Scanning
1. Read every registered layer instruction and build the **Layer Inventory**: instruction name, side (backend/frontend), layer, `applyTo` paths, key constraints. Include the `fercho-*-bugfix-refactor` prompt files.
2. Detect the conventions the commands depend on:
   - **SDD docs root** (`{{PATH_SDD_DOCS}}`): an existing SDD/specs folder if present, otherwise `docs/sdd`.
   - **Ticket-id format** (e.g., `ABC-123`) and how it appears in branch names.
   - **Verification commands:** build, lint, and test for each side.
   - **Repository changelog:** whether the project keeps one (e.g., `CHANGELOG.md`) and its format.
3. List any existing `.github/prompts/sdd-*.prompt.md` files.

### Step 3: Confirmation Proposal (Interactive)
Present a report containing:
- **Layer Inventory Table:** instruction vs. side / layer / `applyTo` paths.
- **Detected Conventions:** the four items from Step 2.
- **Discrepancies:** missing layer instructions, layers without coverage, conventions needing custom configuration.
- **File Generation Plan:** the prompt files to create in `.github/prompts/`. Mark each existing `sdd-*.prompt.md` as *will be overwritten* so the user can exclude customized ones.
- **Instruction File Changes:** what will be written to `.github/copilot-instructions.md` (section 4), including any unmarked legacy rules found.

*STOP: wait for user approval before writing files.*

### Step 4: Deployment
After approval, write:
1. `.github/prompts/sdd-*.prompt.md` following sections 2 and 3, with `{{PATH_SDD_DOCS}}`, instruction names, and verification commands replaced by real values. Skip any prompt file the user excluded.
2. The rules block (section 4) in `.github/copilot-instructions.md`.

---

## 2. Shared Conventions

Every generated command implements these conventions identically.

### 2.1 Prompt File Format

One prompt file per command under `.github/prompts/`, named `<command>.prompt.md` (e.g., `sdd-init-feature.prompt.md`). The user runs it in Copilot Chat as `/<command>`; the arguments are the text typed after it.

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

`agent: agent` runs the prompt in agent mode, needed to read and write files. On older VS Code versions the field is named `mode`; use the one the installed version supports.

### 2.2 Artifacts

All artifacts of a ticket live in `{{PATH_SDD_DOCS}}/{ticket}/`:

| File | Created by | Content |
|---|---|---|
| `requirements.md` | `/sdd-init-feature` | Goal, scope, out of scope, functional requirements (`R01`…), acceptance criteria, non-functional requirements |
| `decisions.md` | `/sdd-init-feature` | Questions and decisions (section 2.4) |
| `design-architecture.md` | `/sdd-generate-design` | Affected layers and their instructions (backend and frontend), components to add/change, flow between layers, risks |
| `design-data-api.md` | `/sdd-generate-design` | Data model changes and API contracts shared by backend and frontend |
| `design-ui.md` | `/sdd-generate-design` | Only when frontend layer instructions are registered and the feature has frontend work: views, components and props, state, hooks, routes |
| `tasks.md` | `/sdd-generate-tasks` | Task list (section 2.5) |
| `plan-TXX.md` | `/sdd-plan-task` | Plan for one task: ordered steps, each with its layer, layer instruction, and files touched |
| `changelog.md` | `/sdd-execute-task` | Per-ticket log: one entry per executed task (files changed, verification result); `/sdd-feature-change` adds one row per change |

The per-ticket `changelog.md` is distinct from the repository changelog, which is updated only if the project keeps one, in its detected format.

### 2.3 Active Ticket Resolution

`/sdd-init-feature` receives the ticket id explicitly. Every other command resolves it in this order:
1. A ticket id passed in the arguments.
2. A ticket id in the current git branch name that has a folder under `{{PATH_SDD_DOCS}}/`.
3. The only folder under `{{PATH_SDD_DOCS}}/`, if there is exactly one.
4. Otherwise STOP and ask the user.

The command states the resolved ticket in its first line of output. `/sdd-feature-change` (argument is a change description) and `/sdd-review-tasks` (no argument) start at step 2.

### 2.4 Questions & Decisions

- Ids have no hyphen and are never reused or renumbered: requirements `R01`…, questions `Q01`…, decisions `D01`….
- A command that hits missing or ambiguous business information must NOT guess. It records a question in `decisions.md` with the next id, status `open`, context, options considered, and the artifacts or tasks it affects.
- Statuses:
  - `open`: blocks the tasks in its "affects" field; independent work continues.
  - `answered`: no longer blocks; answer and source recorded.
  - `assumed`: no longer blocks; the working assumption is recorded and must be confirmed before shipping. Plans and reports name the `assumed` questions the work rests on.
- Only `/sdd-answer` and `/sdd-feature-change` change a question's status. An answer without a source is recorded as `assumed`.
- Decisions taken without a question (explicit user instructions) are recorded in the same file with their rationale.

### 2.5 Tasks

- Ids are `T01`, `T02`, …; the plan of `T01` is `plan-T01.md`.
- Each task in `tasks.md` has: checkbox (checked only when `done`, `deferred`, or cancelled), id, title, covered requirement ids, owning layer instruction(s), dependencies, blocking question ids, acceptance criteria, status.
- Statuses and who sets them:

| Status | Meaning | Set by |
|---|---|---|
| `todo` | Not started. A planned task is a `todo` task with a `plan-TXX.md` (there is no "planned" status) | `/sdd-generate-tasks`; `/sdd-answer` when nothing else blocks it |
| `blocked` | Has an `open` question or unfinished dependency | `/sdd-generate-tasks`, `/sdd-feature-change` |
| `in-progress` | Execution started | `/sdd-execute-task` |
| `review` | Optional; implemented and awaiting review | by hand |
| `done` | Verification passed | `/sdd-execute-task` only |
| `deferred` / cancelled (struck through) | Will not be built | `/sdd-feature-change` |

- `deferred` and cancelled count as finished, never as remaining work.
- `/sdd-plan-task` and `/sdd-review-tasks` never change a status.
- When `[task-id]` is omitted: `/sdd-plan-task` takes the first `todo` task without a `plan-TXX.md`; `/sdd-execute-task` takes the `in-progress` task (resumed session) or else the first `todo` task with a `plan-TXX.md`. The command names the task before doing anything else.

---

## 3. SDD Slash Command Suite Spec

| Command | Objective |
|---|---|
| `/sdd-init-feature <ticket-id>` | Initialize requirements and decision registry |
| `/sdd-generate-design [ticket-id]` | Create architecture, data/API, and UI design |
| `/sdd-generate-tasks [ticket-id]` | Break the design into vertical-slice tasks |
| `/sdd-plan-task [task-id] [ticket-id]` | Generate the execution plan for one task |
| `/sdd-execute-task [task-id] [ticket-id]` | Implement the plan; source code + `changelog.md` |
| `/sdd-answer <Q-id> <answer>` | Resolve a question and unblock tasks |
| `/sdd-feature-change <change description>` | Apply a scope or design change to existing artifacts in place |
| `/sdd-review-tasks` | Read-only status report and next-task recommendation (chat only, no file) |

The first five are the main flow, in order; each requires the artifacts of the previous one. The last three are supporting commands, run when needed. Output files are those listed in section 2.2.

**Applies to all commands:** only `/sdd-execute-task` modifies source code, and none of them commits or pushes.

### 3.1 `/sdd-init-feature <ticket-id>`
- **Preconditions:** a ticket id is given. If `{{PATH_SDD_DOCS}}/{ticket}/` exists, STOP and ask whether to update it; never overwrite silently.
- **Steps:**
  1. Gather the feature description from the user's message and any ticket content provided.
  2. Write `requirements.md` (structure in 2.2).
  3. Write `decisions.md`, recording every gap or ambiguity as an `open` question.
- **Output:** both files plus a summary of the open questions.

### 3.2 `/sdd-generate-design [ticket-id]`
- **Preconditions:** `requirements.md` exists.
- **Steps:**
  1. Read `requirements.md`, `decisions.md`, and the Layer Inventory.
  2. Inspect the existing code in the affected layers; reuse existing components and patterns.
  3. Write the design files per layer, naming for each change the real layer instruction and paths.
  4. Record new questions in `decisions.md`; mark design sections that depend on an `open` or `assumed` question as provisional.
- **Stop:** an `open` question makes the overall architecture undecidable.

### 3.3 `/sdd-generate-tasks [ticket-id]`
- **Preconditions:** `design-architecture.md` exists.
- **Steps:**
  1. Split the design into vertical slices: each task delivers a testable increment and may span backend and frontend.
  2. Tag each task with its owning layer instruction(s), dependencies, and covered requirement ids.
  3. Set tasks to `todo`, except those affected by an `open` question, which are `blocked`.
  4. Check every requirement is covered by at least one task; report any that is not.

### 3.4 `/sdd-plan-task [task-id] [ticket-id]`
- **Preconditions:** `tasks.md` exists; the task is `todo`. If it already has a `plan-TXX.md`, ask whether to replace it.
- **Steps:**
  1. Read the task, the design sections it references, and the instruction file of each owning layer.
  2. Inspect the files the task will touch.
  3. Write `plan-TXX.md` as ordered steps. Each step names its layer, exactly one layer instruction (real name from the Layer Inventory), and the files it creates or modifies, all within that instruction's `applyTo` paths. A change needing two instructions is two steps; a step no instruction covers (documentation, configuration) is `Instruction: none`.
  4. Add the tests to write (inside the owning instruction's step), the verification commands, and the `assumed` questions the task relies on.
- **Stop:** a needed file is outside every registered layer instruction's `applyTo` paths (report and ask); the task is `blocked` (name the question or dependency).

### 3.5 `/sdd-execute-task [task-id] [ticket-id]`
- **Preconditions:** `plan-TXX.md` exists; the task is `todo` or `in-progress`; none of its questions is `open`; its dependencies are `done`.
- **Steps:**
  1. Set the task to `in-progress`. If it already is, resume from the first plan step with no `changelog.md` entry.
  2. Implement the steps in order, including tests. Before each step read the instruction file it names and follow its constraints and workflow, without relying only on Copilot attaching it automatically; do not choose a different instruction than the plan's. For `Instruction: none`, print the fallback notice defined by the layer rules.
  3. Keep each step's edits inside the files the plan lists for it; a needed file outside them is a stop condition, not a reason to widen the step.
  4. Run the plan's verification commands and fix failures caused by the change.
  5. Append the entry to the per-ticket `changelog.md`; update the repository changelog if the project keeps one.
  6. Set `done` only if verification passes; otherwise leave `in-progress` and report what fails.
- **Stop:** the plan is wrong or incomplete for the real code (report the deviation, do not improvise); a new business question appears (record it and stop the affected part).

### 3.6 `/sdd-answer <Q-id> <answer>`
- **Preconditions:** the question exists and is `open` or `assumed`. If `answered`, STOP and ask whether to revise it.
- **Steps:**
  1. Record the answer, source, and date; set `answered` (or `assumed` if no confirmed source). Keep the id.
  2. Update every artifact in the question's "affects" field (requirements, provisional design sections, tasks).
  3. Remove the question from its tasks' blocking lists; the tasks it was the last blocker of return to `todo`.
  4. If an affected task has a plan or is `in-progress`, `review`, or `done`, report which plan to regenerate or code to revisit; do not change code.
  5. If the answer widens scope or contradicts `done` work (including a failed `assumed` question), record it and hand off to `/sdd-feature-change` with a ready description instead of editing artifacts.
- **Output:** summary of updated files and unblocked tasks.

### 3.7 `/sdd-feature-change <change description>`
- **When to use:** any change to what the ticket builds or how: scope change, an answer that widens scope, review findings, or a design that proves wrong in code. Never edit the artifacts by hand for this.
- **Preconditions:** `requirements.md`, design files, `tasks.md`, and `decisions.md` exist; a change description is given (ask otherwise).
- **Steps** (edit in place, never regenerate a file), in this order:
  1. `requirements.md`: add new scope, deprecate superseded requirements; unchanged for a technical-only change.
  2. `decisions.md`: record new ambiguities as `open` questions with the tasks they block; set `answered` the questions the change settles, with answer and source.
  3. Design files: change only affected sections. Existing section and step numbers never change; new ones take the next free number.
  4. `tasks.md`: never edit a `done` task (rework is a new task). Strike through obsolete tasks. Add new tasks with the next free ids as `todo` or `blocked`. For a changed task with a `plan-TXX.md`, say the plan is stale and recommend `/sdd-plan-task TXX`.
  5. `changelog.md`: add one row summarizing the change (requirements, design sections, tasks, questions).
- **Output:** an alignment report of what changed per artifact and the first task to work on next.
- **Stop:** a required artifact is missing; a needed business rule is not stated (record an `open` question instead).

### 3.8 `/sdd-review-tasks`
- **Preconditions:** `tasks.md` exists. If `decisions.md` is missing, say so and treat every blocking question as `open`.
- **Steps:**
  1. Read each task's id, dependencies, blocking questions, and status, and each question's status.
  2. Consistency check: report tasks whose checkbox and status disagree, `blocked` tasks whose questions are all `answered` or `assumed`, and blocking ids missing from `decisions.md`.
  3. Classify tasks: finished, blocked (unfinished dependency or `open` question), or available.
  4. Recommend the next available task in order, naming any `assumed` question it rests on. If none is available, list the `open` questions that would unblock the most tasks and point to `/sdd-answer`.
- **Output:** a chat report with completed/remaining counts, pending tasks and their state, inconsistencies, the recommended task, and the next command.

---

## 4. Global Instructions Patch (`.github/copilot-instructions.md` - SDD)

The SDD rules go in `.github/copilot-instructions.md`, next to the layer rules added by the layer instruction templates. Copilot loads this file for every request in the repository.

1. **Target file:** `.github/copilot-instructions.md`. Create it if it does not exist.
2. **Markers:** the block is delimited by `<!-- fercho-sdd:start -->` and `<!-- fercho-sdd:end -->`. If present, replace everything between them; otherwise append at the end. Never write the block twice or modify content outside the markers.
3. **Block order:** `fercho-base`, `fercho-be`, `fercho-fe`, `fercho-sdd`.
4. **Legacy rules:** an unmarked copy of this section in `.github/copilot-instructions.md` or `AGENTS.md` from an earlier setup is reported in Step 3 and replaced with the marked block only after approval.

Block to write (replace `{{PATH_SDD_DOCS}}`):

```markdown
<!-- fercho-sdd:start -->
## SDD Workflow Rules

1. Feature work follows `/sdd-init-feature` -> `/sdd-generate-design` -> `/sdd-generate-tasks` -> `/sdd-plan-task` -> `/sdd-execute-task`, each requiring the previous step's artifacts. Artifacts live under `{{PATH_SDD_DOCS}}/{ticket}/`.
2. Do not implement a task without its `plan-TXX.md`, and do not execute a task blocked by an `open` question.
3. Never guess business logic: record it as a question in `decisions.md` and resolve it with `/sdd-answer`. Work resting on an `assumed` question says so, and the assumption is confirmed before shipping.
4. Follow the Information Lookup Hierarchy: SDD Docs -> Project Docs -> Existing Code -> External Specs.
5. Supporting commands: `/sdd-review-tasks` for status and the next task, `/sdd-feature-change` for any scope or design change.
<!-- fercho-sdd:end -->
```
