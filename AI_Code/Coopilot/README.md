# GitHub Copilot – SDD Environment Setup Templates

This folder contains master templates that you hand to GitHub Copilot Chat (agent mode) to bootstrap a **Spec-Driven Development (SDD)** environment in any repository. Copilot scans the project, maps its architecture to a set of generic layers, and generates path-scoped layer instructions, SDD prompt files, and a rules block in `.github/copilot-instructions.md` tailored to that codebase.

These are the Copilot counterparts of the templates in [../Claude](../Claude/README.md), using Copilot's native customization files:

| Concept | Claude Code | GitHub Copilot |
|---|---|---|
| Layer rules | `.claude/skills/fercho-*/SKILL.md` | `.github/instructions/fercho-*.instructions.md` (applied by `applyTo` path) |
| SDD commands | `.claude/commands/sdd-*.md` | `.github/prompts/sdd-*.prompt.md` |
| Global rules | `AGENTS.md`, imported from `CLAUDE.md` | `.github/copilot-instructions.md` |

## Templates

| Order | File | Generates |
|---|---|---|
| 1 | [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) | Backend layer instructions (`.github/instructions/fercho-be-*.instructions.md`) |
| 2 | [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) | Frontend layer instructions (`.github/instructions/fercho-fe-*.instructions.md`) |
| 3 | [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) | SDD prompt files (`.github/prompts/sdd-*.prompt.md`) |

The SDD prompt files are generated **last** because they are built from the layer instructions already deployed in the repository: tasks, plans, and execution all reference the real backend and frontend instruction names.

---

## init_ia_enviroment_be.md — Backend Layer Instructions

Master blueprint for the layer rules of a backend project (any language or framework). It contains:

1. **Setup Agent Execution Protocol** – detect the stack profile (language and framework, persistence, migrations, DI, async model, API style, testing) and map layers → present a confirmation report and **stop for approval** → deploy the files.
2. **Standard Layer Mapping Matrix** – generic layers with `{{PATH_*}}` placeholders that Copilot resolves to real paths:
   Data Scaffolding, Repositories, Domain Logic, Business Services, Orchestrators, API / DTOs, DI Registration, Third-Party Integrations.
3. **Layer Instruction Definitions** – the instruction file format and the eight `fercho-be-*` layer instructions:
   `data-scaffolding`, `repositories`, `domain-logic`, `services`, `orchestrators`, `api-dtos`, `di-registration`, `integrations`; plus the `/fercho-be-bugfix-refactor` prompt.
4. **Global Instructions Patch** – a "Backend Layered Execution & Instruction Dispatch Rules" section written to `.github/copilot-instructions.md`, including the fallback notice printed when no layer instruction applies.

## init_ia_enviroment_fe.md — Frontend Layer Instructions

Master blueprint for the layer rules of a React JS frontend. It contains:

1. **Setup Agent Execution Protocol** – detect the stack profile (TypeScript or JavaScript, data-fetching and state libraries, HTTP client, styling, router, testing) and map layers → present a confirmation report and **stop for approval** → deploy the files.
2. **Standard Frontend Layer Mapping Matrix** – generic layers with `{{PATH_*}}` placeholders:
   UI Components, Features / Containers, State Management, Custom Hooks, API Services, Routing & Guards.
3. **Frontend Layer Instruction Definitions** – the instruction file format and the six `fercho-fe-*` layer instructions:
   `ui-components`, `features`, `state-management`, `custom-hooks`, `api-services`, `routing`; plus the `/fercho-fe-bugfix-refactor` prompt.
4. **Global Instructions Patch** – a "Frontend Layered Execution & Separation Rules" section written to `.github/copilot-instructions.md`.

## init_ia_enviroment_sdd_framework.md — SDD Prompt Files

Master blueprint for the SDD slash command suite. It contains:

1. **Setup Agent Execution Protocol** – check that layer instructions are already deployed (stops if none exist) → build a layer inventory and detect conventions (SDD docs folder, ticket-id format, verification commands, changelog) → present a confirmation report and **stop for approval** → deploy the prompt files.
2. **Shared Conventions** – the prompt file format, the artifacts of a ticket, how the active ticket is resolved, the question/decision lifecycle (`Q-01`…), and task ids and statuses (`T01`…).
3. **SDD Slash Command Suite** – preconditions, steps, and stop conditions for each command:

   | Command | Purpose | Output |
   |---|---|---|
   | `/sdd-init-feature <ticket-id>` | Initialize requirements and decision registry | `requirements.md`, `decisions.md` |
   | `/sdd-generate-design [ticket-id]` | Create architecture, data/API, and UI design from specs | `design-architecture.md`, `design-data-api.md`, `design-ui.md` (frontend only) |
   | `/sdd-generate-tasks [ticket-id]` | Break the design into vertical-slice tasks | `tasks.md` |
   | `/sdd-plan-task [task-id] [ticket-id]` | Plan a single task in isolation (no code changes) | `plan-TXX.md` |
   | `/sdd-execute-task [task-id] [ticket-id]` | Implement the plan and verify it (no commit) | Source code + `changelog.md` |
   | `/sdd-answer <Q-id> <answer>` | Answer an open decision and unblock tasks | Updated `decisions.md` and affected artifacts |

4. **Global Instructions Patch** – an "SDD Workflow Rules" section written to `.github/copilot-instructions.md`, including the information lookup hierarchy.

---

## How Layer Rules Work in Copilot

- Each layer is one file in `.github/instructions/` whose `applyTo` globs point to that layer's folders (and its tests). Copilot attaches the file automatically when it works on matching files.
- `applyTo` is also the layer's edit scope: a task executed under a layer instruction may only modify files matching it.
- Bug fixing is a prompt, not an instruction file, so its test-first workflow applies only when you run `/fercho-be-bugfix-refactor` or `/fercho-fe-bugfix-refactor`.

## Where the Rules Are Written

Each template writes its own rules block to `.github/copilot-instructions.md`:

| Template | Markers |
|---|---|
| Backend layer instructions | `<!-- fercho-be:start -->` … `<!-- fercho-be:end -->` |
| Frontend layer instructions | `<!-- fercho-fe:start -->` … `<!-- fercho-fe:end -->` |
| SDD prompt files | `<!-- fercho-sdd:start -->` … `<!-- fercho-sdd:end -->` |

Re-running a template replaces the content between its markers instead of appending a duplicate, and nothing outside the markers is touched. Do not edit inside the markers by hand; project-specific rules go outside them.

---

## How to Set Up a Development Environment

Run the templates in order, one step at a time. A backend-only or frontend-only project skips the layer template that does not apply; the SDD template always goes last.

1. **Copy the templates** into the target repository (or keep them somewhere you can attach from):

   ```bash
   cp init_ia_enviroment_*.md /path/to/your/project/
   ```

2. **Open the repository in VS Code** and open Copilot Chat in **agent mode**, so Copilot can read the codebase and create files.

3. **Generate the backend layer instructions.** Attach `init_ia_enviroment_be.md` to the chat (`#file` or drag and drop) and send:

   ```text
   Read this master setup template and initialize this project following the Setup Agent Execution Protocol.
   ```

   Copilot stops after scanning and shows the stack profile, the layer mapping table (generic layer → detected project path), any discrepancies, and the files it plans to generate. Correct any wrong path, then approve. Result: `.github/instructions/fercho-be-*.instructions.md`, the `/fercho-be-bugfix-refactor` prompt, and the backend rules block in `.github/copilot-instructions.md`.

4. **Generate the frontend layer instructions** (if the project has a frontend), the same way with `init_ia_enviroment_fe.md`. Result: `.github/instructions/fercho-fe-*.instructions.md` and the `/fercho-fe-bugfix-refactor` prompt.

5. **Generate the SDD prompt files**, the same way with `init_ia_enviroment_sdd_framework.md`. Copilot verifies the layer instructions exist, shows the layer inventory and the prompt files it plans to create, and waits for approval. Result: `.github/prompts/sdd-*.prompt.md` and the SDD rules block in `.github/copilot-instructions.md`.

6. **Review and commit** the generated files so the whole team shares the same setup. The templates themselves can be removed from the repo afterwards.

## Daily SDD Workflow (after setup)

Run the commands in Copilot Chat:

```text
/sdd-init-feature ABC-123     → requirements.md + decisions.md
/sdd-generate-design          → architecture, data/API and UI design
/sdd-generate-tasks           → tasks.md (vertical slices)
/sdd-plan-task T01            → plan-T01.md
/sdd-execute-task T01         → code changes + tests + changelog.md
/sdd-answer Q-01 "<answer>"   → resolves an open question blocking a task
```

Artifacts are stored under `docs/sdd/{ticket}/` (or the project's existing specs folder, if the setup detected one). After `/sdd-init-feature`, the other commands find the ticket from the branch name; pass the ticket id explicitly when the branch does not contain it. When no task id is given, plan and execute pick the next task that is not blocked.
