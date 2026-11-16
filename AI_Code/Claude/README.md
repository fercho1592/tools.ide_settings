# Claude Code – SDD Environment Setup Templates

This folder contains master templates that you hand to Claude Code to bootstrap a **Spec-Driven Development (SDD)** environment in any repository. Claude scans the project, maps its architecture to a set of generic layers, and generates layer-scoped skills, SDD slash commands, and a rules block in `CLAUDE.md` (or another instruction file you choose) tailored to that codebase.

| Order | File | Generates | Status |
|---|---|---|---|
| 1 | [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) | Backend layer skills (`.claude/skills/fercho-be-*`) | Ready |
| 2 | [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) | Frontend layer skills (`.claude/skills/fercho-fe-*`) | Ready |
| 3 | [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) | SDD slash commands (`.claude/commands/sdd-*.md`) | Ready |

The SDD commands are generated **last** because they are built from the skills already deployed in the repository: tasks, plans, and execution all reference the real backend and frontend skill names.

---

## init_ia_enviroment_be.md — Backend Skills

Master blueprint for the layer skills of a backend project (any language or framework). It contains:

1. **Setup Agent Execution Protocol** – detect the stack profile (language and framework, persistence, migrations, DI, async model, API style, testing) and map layers → present a confirmation report and **stop for approval** → deploy the skills.
2. **Standard Layer Mapping Matrix** – generic layers with `{{PATH_*}}` placeholders that Claude resolves to real paths:
   Data Scaffolding, Repositories, Domain Logic, Business Services, Orchestrators, API / DTOs, DI Registration, Third-Party Integrations.
3. **Modular Skill Definitions** – the `SKILL.md` format and the nine `fercho-be-*` skills:
   `data-scaffolding`, `repositories`, `domain-logic`, `services`, `orchestrators`, `api-dtos`, `di-registration`, `integrations`, `bugfix-refactor`.
4. **System Prompt Patch** – a "Backend Layered Execution & Skill Dispatch Rules" section written to the rules file, including the fallback notice printed when no layer skill applies.

## init_ia_enviroment_fe.md — Frontend Skills

Master blueprint for the layer skills of a React JS frontend. It contains:

1. **Setup Agent Execution Protocol** – detect the stack profile (TypeScript or JavaScript, data-fetching and state libraries, HTTP client, styling, router, testing) and map layers → present a confirmation report and **stop for approval** → deploy the skills.
2. **Standard Frontend Layer Mapping Matrix** – generic layers with `{{PATH_*}}` placeholders:
   UI Components, Features / Containers, State Management, Custom Hooks, API Services, Routing & Guards.
3. **Modular Frontend Skill Definitions** – the `SKILL.md` format and the seven `fercho-fe-*` skills:
   `ui-components`, `features`, `state-management`, `custom-hooks`, `api-services`, `routing`, `bugfix-refactor`.
4. **System Prompt Patch** – a "Frontend Layered Execution & Separation Rules" section written to the rules file.

## init_ia_enviroment_sdd_framework.md — SDD Commands

Master blueprint for the SDD slash command suite. It contains:

1. **Setup Agent Execution Protocol** – check that layer skills are already deployed (stops if none exist) → build a skill inventory and detect conventions (SDD docs folder, ticket-id format, verification commands, changelog) → present a confirmation report and **stop for approval** → deploy the commands.
2. **Shared Conventions** – the command file format, the artifacts of a ticket, how the active ticket is resolved, the question/decision lifecycle (`Q-01`…), and task ids and statuses (`T01`…).
3. **SDD Slash Command Suite** – preconditions, steps, and stop conditions for each command:

   | Command | Purpose | Output |
   |---|---|---|
   | `/sdd-init-feature <ticket-id>` | Initialize requirements and decision registry | `requirements.md`, `decisions.md` |
   | `/sdd-generate-design [ticket-id]` | Create architecture, data/API, and UI design from specs | `design-architecture.md`, `design-data-api.md`, `design-ui.md` (frontend only) |
   | `/sdd-generate-tasks [ticket-id]` | Break the design into vertical-slice tasks | `tasks.md` |
   | `/sdd-plan-task [task-id] [ticket-id]` | Plan a single task in isolation (no code changes) | `plan-TXX.md` |
   | `/sdd-execute-task [task-id] [ticket-id]` | Implement the plan and verify it (no commit) | Source code + `changelog.md` |
   | `/sdd-answer <Q-id> <answer>` | Answer an open decision and unblock tasks | Updated `decisions.md` and affected artifacts |

4. **System Prompt Patch** – an "SDD Workflow Rules" section written to the rules file, including the information lookup hierarchy.

---

## Where the Rules Are Written

Each template writes its own rules block to one instruction file at the repository root. The first template you run asks where to put the rules, as part of its confirmation report:

| Option | Use it when |
|---|---|
| `CLAUDE.md` (default) | The rules are for the whole team and the project uses Claude Code. |
| `CLAUDE.local.md` | You want the rules only on your machine. The file must be in `.gitignore` and is not shared. |
| `AGENTS.md` | Other AI assistants should read the same rules. `CLAUDE.md` then gets an `@AGENTS.md` import line so Claude Code loads them. |

Later templates find the existing blocks and reuse the same file without asking again.

| Template | Markers |
|---|---|
| Backend skills | `<!-- fercho-be:start -->` … `<!-- fercho-be:end -->` |
| Frontend skills | `<!-- fercho-fe:start -->` … `<!-- fercho-fe:end -->` |
| SDD commands | `<!-- fercho-sdd:start -->` … `<!-- fercho-sdd:end -->` |

Re-running a template replaces the content between its markers instead of appending a duplicate, and nothing outside the markers is touched. Do not edit inside the markers by hand; project-specific rules go outside them.

---

## How to Set Up a Development Environment

Run the templates in order, one Claude Code session step at a time. A backend-only or frontend-only project skips the skill template that does not apply; the SDD commands template always goes last.

1. **Copy the templates** into the target repository (or reference them by path):

   ```bash
   cp init_ia_enviroment*.md /path/to/your/project/
   ```

2. **Open Claude Code** in the root of the target repository:

   ```bash
   cd /path/to/your/project
   claude
   ```

3. **Generate the backend skills:**

   ```text
   Read @init_ia_enviroment_be.md and initialize this project following the Setup Agent Execution Protocol.
   ```

   Claude stops after scanning and shows the layer mapping table (generic layer → detected project path), any discrepancies, and the files it plans to generate. Correct any wrong path, then approve. Result: `.claude/skills/fercho-be-*/SKILL.md` and the backend rules block in the rules file you chose.

4. **Generate the frontend skills** (if the project has a frontend), the same way with `@init_ia_enviroment_fe.md`. Result: `.claude/skills/fercho-fe-*/SKILL.md`.

5. **Generate the SDD commands:**

   ```text
   Read @init_ia_enviroment_sdd_framework.md and initialize this project following the Setup Agent Execution Protocol.
   ```

   Claude verifies the skills exist, shows the skill inventory and the commands it plans to create, and waits for approval. Result: `.claude/commands/sdd-*.md` and the SDD rules block in the same rules file.

6. **Review and commit** the generated files so the whole team shares the same setup. The templates themselves can be removed from the repo afterwards.

## Daily SDD Workflow (after setup)

```text
/sdd-init-feature ABC-123     → requirements.md + decisions.md
/sdd-generate-design          → architecture, data/API and UI design
/sdd-generate-tasks           → tasks.md (vertical slices)
/sdd-plan-task T01            → plan-T01.md
/sdd-execute-task T01         → code changes + tests + changelog.md
/sdd-answer Q-01 "<answer>"   → resolves an open question blocking a task
```

Artifacts are stored under `docs/sdd/{ticket}/` (or the project's existing specs folder, if the setup detected one). After `/sdd-init-feature`, the other commands find the ticket from the branch name; pass the ticket id explicitly when the branch does not contain it. When no task id is given, plan and execute pick the next task that is not blocked.
