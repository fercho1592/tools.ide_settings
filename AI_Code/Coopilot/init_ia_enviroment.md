# GitHub Copilot SDD & Modular Framework - Master Setup Template

This master document contains the instructions for initializing and deploying the Spec-Driven Development (SDD) framework and layer-scoped rules for GitHub Copilot (`.github/copilot-instructions.md`, path-specific instructions, and prompt files).

---

## Part 1: Execution Protocol for Copilot Initialization

When this file is placed in a new repository, run a chat session with GitHub Copilot and provide the prompt:

> *"Read this master setup template, analyze the repository structure, map the architecture layers, present a comparison table, and wait for my confirmation before creating any files."*

### Step 1: Structure Analysis & Layer Mapping

Copilot will scan the workspace to detect the equivalences for Clean Architecture layers (Entities/Models, Repositories, Domain/Business Logic, Services, API/Controllers, Dependency Injection).

It must output a **Comparison Matrix Table**:

| Standard Layer | Expected Path / Pattern | Detected Project Equivalent Path | Status / Action |
| --- | --- | --- | --- |
| Scaffolding / Models | `src/**/Infrastructure/Data/Models/` | *[Detected or Default]* | Match / Needs adjustment |
| Repositories | `src/**/Repositories/` | *[Detected or Default]* | Match / Needs adjustment |
| Domain Classes (Pure) | `src/**/Services/Rules/` | *[Detected or Default]* | Match / Needs adjustment |
| Business Services | `src/**/Services/` | *[Detected or Default]* | Match / Needs adjustment |
| API / Endpoints / DTOs | `src/Contracts/` or Controllers | *[Detected or Default]* | Match / Needs adjustment |
| DI Registration | `src/**/Extensions/` | *[Detected or Default]* | Match / Needs adjustment |

---

## Part 2: Copilot Artifact Generation Structure

Once the user approves or adjusts the mapping table, Copilot will automatically generate the following file tree:

```text
.github/
├── copilot-instructions.md            # Global system prompt & fallback rules
├── instructions/
│   ├── db-scaffolding.instructions.md   # Layer: Scaffolding (applyTo: Models/Entities paths)
│   ├── repositories.instructions.md     # Layer: Repositories (applyTo: Repositories paths)
│   ├── domain-rules.instructions.md     # Layer: Domain/Pure logic (applyTo: Rules/Strategy paths)
│   ├── business-services.instructions.md# Layer: Services (applyTo: Services paths)
│   ├── api-dtos.instructions.md         # Layer: API & Contracts (applyTo: Controllers/Contracts paths)
│   └── di-registration.instructions.md  # Layer: DI Extension (applyTo: Extensions paths)
└── prompts/
    ├── sdd-init-feature.prompt.md       # Slash command for step 1
    ├── sdd-generate-design.prompt.md    # Slash command for step 2
    ├── sdd-generate-tasks.prompt.md     # Slash command for step 3
    ├── sdd-plan-task.prompt.md          # Slash command for step 4a
    └── sdd-execute-task.prompt.md       # Slash command for step 4b

```

---

## Part 3: Templates Content to be Generated

### 3.1 Global Instructions (`.github/copilot-instructions.md`)

```markdown
# Project SDD & Architecture Instructions

## Fallback & Transparency Rule
- Before executing any task, evaluate whether it matches a specialized layer instruction.
- If a task falls outside those scopes (unit tests, documentation, generic fixes), print a brief notice before starting and proceed under standard project conventions:
  > *ℹ️ Notice: No specialized layer instruction applies to this task. Executing under standard project conventions.*

## Information Lookup Hierarchy (Docs First)
1. Ticket SDD docs (`docs/sdd/{ticket}/`)
2. Repository docs & READMEs
3. Current codebase implementations

```

### 3.2 Path-Specific Instruction Example (`.github/instructions/domain-rules.instructions.md`)

```yaml
---
applyTo: "src/**/Services/Rules/**"
---
# Domain Business Logic Rules

## Objective & Scope
- Create pure domain classes, entities, and state-machine processors with zero I/O, database, or external dependencies.

## Constraints
- Never inject DbContext, repositories, HTTP clients, or logging libraries.
- Must be fully deterministic; edge cases must be handled internally.

## Workflow
1. Write pure calculation or validation rules.
2. Ensure zero framework coupling to make unit testing direct.

```

### 3.3 Prompt File Example (`.github/prompts/sdd-init-feature.prompt.md`)

```markdown
---
description: "Step 1: Initialize SDD feature requirements, decisions, and changelog"
---
# Initialize SDD Feature

You are acting as the SDD Feature Manager.
1. Read the ticket ID provided in the argument.
2. Generate `docs/sdd/{ticket}/requirements.md`, `decisions.md` (seeded with questions), and `changelog.md`.
3. Confirm branch setup with the user before finalizing.

```