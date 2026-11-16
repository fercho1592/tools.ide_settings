# SDD Framework Setup Master Template

This document serves as the master blueprint for initializing and adapting the **Spec-Driven Development (SDD) Commands and Architectural Skills Framework** to any repository, regardless of language or framework.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Layer Mapping & Codebase Scanning
1. Analyze the project folder structure, package configurations, and architectural patterns.
2. Identify the project's architectural layers (e.g., Data/Entities, Repositories/DataAccess, Domain/Business Rules, Application Services, Controllers/Endpoints, Dependency Injection).
3. Map the generic framework layers to the specific paths in this repository.

### Step 2: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Layer Mapping Table:** Standard Layer vs. Detected Project Path.
- **Adjustment Points / Discrepancies:** Note any layer that doesn't cleanly align or requires custom configuration.
- **Proposed File Generation Plan:** List of commands and skills that will be created in `.claude/commands/` and `.claude/skills/`.

*STOP: Wait for user approval before writing files.*

### Step 3: Deployment
Upon user approval, write the following:
1. Updated `.claude/skills/fercho-*/SKILL.md` (or project-equivalent custom skills) with project-specific scopes and constraints.
2. Updated `.claude/commands/sdd-*.md` tailored to project conventions.
3. System Instructions patch for `CLAUDE.md` / `AGENTS.md` (defining local execution policies and layer fallback rules).

---

## 2. Standard Layer Mapping Matrix

| Generic Layer | Standard Role | Target Project Path Pattern (To be resolved) |
|---|---|---|
| **Data Scaffolding** | ORM entities, schemas, DB context, table maps | `{{PATH_DATA_SCAFFOLDING}}` |
| **Repositories** | Data access, queries, persistence interfaces | `{{PATH_REPOSITORIES}}` |
| **Domain Logic** | Pure business rules, state machines, specs (No I/O) | `{{PATH_DOMAIN_LOGIC}}` |
| **Business Services** | Application use-case orchestrators | `{{PATH_SERVICES}}` |
| **Orchestrators** | Complex multi-service strategy branching | `{{PATH_ORCHESTRATORS}}` |
| **API / DTOs** | Controllers, minimal endpoints, request/response models | `{{PATH_API_DTOS}}` |
| **DI Registration** | Container configuration, modules, extensions | `{{PATH_DI_REGISTRATION}}` |
| **Third-Party Integrations**| External APIs, proxies, boundary handlers | `{{PATH_INTEGRATIONS}}` |

---

## 3. Modular Skill Definitions

Each skill must be generated under `.claude/skills/` following this specification:

### 3.1 Skill: Data Scaffolding
- **Objective:** Create and configure ORM models and schema mapping rules.
- **Target Entity / Layer:** `{{PATH_DATA_SCAFFOLDING}}`
- **Constraints:**
  - Strict mapping to project database conventions.
  - No business logic or repository calls inside entity definitions.
  - Immutable primary key handling where applicable.
- **Workflow:**
  1. Inspect database schema or requirement specification.
  2. Generate entity model under target path.
  3. Apply schema bindings/fluent configurations.

### 3.2 Skill: Repositories
- **Objective:** Build query and mutation interfaces and data access implementations.
- **Target Entity / Layer:** `{{PATH_REPOSITORIES}}`
- **Constraints:**
  - No business rule validation inside queries.
  - Enforce asynchronous execution with cancellation token support where applicable.
  - Read-only operations must use non-tracking queries.
- **Workflow:**
  1. Define repository interface.
  2. Implement data layer methods.
  3. Wire projections to avoid returning full entities unnecessarily.

### 3.3 Skill: Business Logic (Pure Domain)
- **Objective:** Encapsulate core domain calculations, rule validators, and state machines.
- **Target Entity / Layer:** `{{PATH_DOMAIN_LOGIC}}`
- **Constraints:**
  - ZERO external dependencies (No I/O, DB, HTTP, or logging framework imports).
  - High testability: deterministic inputs and outputs.
- **Workflow:**
  1. Extract pure business rules from requirements.
  2. Build domain class/method structure.
  3. Include unit test specification outline.

### 3.4 Skill: Business Services
- **Objective:** Implement application use cases by coordinating repositories and domain classes.
- **Target Entity / Layer:** `{{PATH_SERVICES}}`
- **Constraints:**
  - No direct HTTP status code manipulation or request handling.
  - Constructor injection for all dependencies.
  - Delegate pure calculations to `{{PATH_DOMAIN_LOGIC}}`.
- **Workflow:**
  1. Receive intent/task plan.
  2. Wire repositories and domain logic.
  3. Handle transactional boundaries.

### 3.5 Skill: API & DTO Contracts
- **Objective:** Expose endpoints and define contract request/response models.
- **Target Entity / Layer:** `{{PATH_API_DTOS}}`
- **Constraints:**
  - DTOs must be immutable (e.g., `record`s or read-only types).
  - Enforce validation filters and uniform API response envelopes.
- **Workflow:**
  1. Create request/response contracts.
  2. Map endpoint/controller action.
  3. Apply authorization/validation middleware tags.

### 3.6 Skill: Bug Fixing & Refactoring
- **Objective:** Surgical issue resolution using test-driven methodology.
- **Target Entity / Layer:** Global project scope
- **Constraints:**
  - Mandatory failing test (Red stage) before making production changes.
  - Do not introduce structural refactoring unless explicitly authorized.
- **Workflow:**
  1. Reproduce bug with unit/integration test.
  2. Apply minimum fix in the corresponding layer.
  3. Verify fix passes (Green stage).

---

## 4. SDD Slash Command Suite Spec

The following slash commands will be generated under `.claude/commands/`:

| Command | Objective | Output File / Artifact |
|---|---|---|
| `/sdd-init-feature <ticket-id>` | Initialize requirements and decision registry | `docs/sdd/{ticket}/requirements.md`<br>`docs/sdd/{ticket}/decisions.md` |
| `/sdd-generate-design` | Create architecture and API design from specs | `docs/sdd/{ticket}/design-architecture.md`<br>`docs/sdd/{ticket}/design-data-api.md` |
| `/sdd-generate-tasks` | Break down design into vertical-slice task units | `docs/sdd/{ticket}/tasks.md` |
| `/sdd-plan-task [task-id]` | Generate isolated execution plan for a single task | `docs/sdd/{ticket}/plan-TXX.md` |
| `/sdd-execute-task [task-id]`| Implement code changes defined in `plan-TXX.md` | Source code files + `changelog.md` |
| `/sdd-answer <Q-id> <answer>`| Unblock pending task by answering an open decision | Updated `decisions.md` & affected tasks |

---

## 5. System Prompt Patch (`CLAUDE.md` / `AGENTS.md`)

Add the following section to the target project's primary system instruction file:

```markdown
## Layered Execution & Skill Dispatch Rules

1. Before modifying code, analyze the target layer against registered skills in `.claude/skills/`.
2. Restrict code edits strictly to the `allowed_files_to_modify` paths defined in the corresponding layer skill.
3. **Fallback Notice:** If a task does not match any registered layer skill (e.g., documentation, unit tests, script tooling), print this notice before executing:
   > *ℹ️ Note: No custom layer skill applies to this task. Executing under standard project conventions.*
4. Follow the Information Lookup Hierarchy: SDD Docs -> Project Docs -> Existing Code -> External Specs. Never guess business logic.
```