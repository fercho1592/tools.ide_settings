# SDD Framework Setup Master Template (Backend Skills)

This document serves as the master blueprint for initializing and adapting the **Backend Architectural Skills Framework** to any repository, regardless of language or framework.

> **Execution order:** this template is run **before** [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) (SDD commands). The SDD commands are generated from the skills deployed here (and from the frontend skills in [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md), when the project has a frontend). On a new project with no rules file, [init_ia_enviroment_claude.md](./init_ia_enviroment_claude.md) (base project rules) runs before all of them.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Stack Detection, Layer Mapping & Codebase Scanning
1. Analyze the project folder structure, package configurations, and architectural patterns.
2. Detect the **Stack Profile**; every generated skill must follow it instead of assuming defaults:
   - **Language & framework:** e.g., C# / ASP.NET Core, Java / Spring Boot, Python / FastAPI or Django, TypeScript / NestJS or Express, Go.
   - **Persistence:** ORM or data-access library (EF Core, Hibernate/JPA, SQLAlchemy, Prisma, TypeORM, raw SQL) and database engine.
   - **Schema changes:** migration tool and how migrations are created and applied.
   - **Dependency injection:** container or wiring mechanism (built-in container, Spring context, framework modules, manual composition root).
   - **Async model:** async/await, reactive, threads, or synchronous; how cancellation and timeouts are propagated.
   - **API style:** controllers, minimal/functional endpoints, routers; validation and error-handling conventions; response envelope, if any.
   - **Testing:** test framework, mocking library, unit vs. integration test locations, and the commands to run them.
3. Identify the project's architectural layers (e.g., Data/Entities, Repositories/DataAccess, Domain/Business Rules, Application Services, Controllers/Endpoints, Dependency Injection).
4. Map the generic framework layers to the specific paths in this repository, including where each layer's tests live.

### Step 2: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Stack Profile:** the detected values from Step 1.
- **Layer Mapping Table:** Standard Layer vs. Detected Project Path (source and tests).
- **Adjustment Points / Discrepancies:** Note any layer that doesn't exist, doesn't cleanly align, or requires custom configuration (e.g., no Orchestrators layer, services and domain logic in the same folder).
- **Proposed File Generation Plan:** List of skills that will be created under `.claude/skills/fercho-be-*/`.
- **Existing Skills:** any skill already under `.claude/skills/` that covers one of these layers under another name or in another format. For each one, propose to replace it with the generated skill or to keep it; never leave two skills owning the same layer.
- **Rules File:** the instruction file where the rules block will be written (section 4). If no earlier run already fixed it, ask the user to choose between `CLAUDE.md` (default), `CLAUDE.local.md`, and `AGENTS.md`. Include any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 3: Deployment
Upon user approval, write the following:
1. `.claude/skills/fercho-be-*/SKILL.md` with project-specific scopes and constraints, following the format in section 3. Skip the skill of any layer the project does not have.
2. System Instructions patch (section 4): write the rules block between its `fercho-be` markers in the rules file resolved in section 4 (`CLAUDE.md` by default), with every `{{PATH_*}}` placeholder replaced by the resolved path.

3. **Format check:** re-read every generated `SKILL.md` and confirm that it has the `name` and `description` frontmatter, that the four sections of section 3 are present as `##` headings, in order and not empty, and that no `{{PATH_*}}` placeholder is left. Fix any file that fails before reporting the deployment as complete.

Do NOT generate `.claude/commands/sdd-*.md` in this step. Once the skills are deployed, tell the user that the next step is to run `init_ia_enviroment_fe.md` (if the project has a frontend) and then `init_ia_enviroment_sdd_framework.md` to generate the SDD commands.

---

## 2. Standard Layer Mapping Matrix

| Generic Layer | Standard Role | Skill | Target Project Path Pattern (To be resolved) |
|---|---|---|---|
| **Data Scaffolding** | ORM entities, schemas, DB context, table maps, migrations | `fercho-be-data-scaffolding` | `{{PATH_DATA_SCAFFOLDING}}` |
| **Repositories** | Data access, queries, persistence interfaces | `fercho-be-repositories` | `{{PATH_REPOSITORIES}}` |
| **Domain Logic** | Pure business rules, state machines, specs (No I/O) | `fercho-be-domain-logic` | `{{PATH_DOMAIN_LOGIC}}` |
| **Business Services** | Application use-case orchestrators | `fercho-be-services` | `{{PATH_SERVICES}}` |
| **Orchestrators** | Complex multi-service strategy branching | `fercho-be-orchestrators` | `{{PATH_ORCHESTRATORS}}` |
| **API / DTOs** | Controllers, endpoints, request/response models | `fercho-be-api-dtos` | `{{PATH_API_DTOS}}` |
| **DI Registration** | Container configuration, modules, extensions | `fercho-be-di-registration` | `{{PATH_DI_REGISTRATION}}` |
| **Third-Party Integrations** | Permanent clients for external APIs, SDKs, and services | `fercho-be-integrations` | `{{PATH_INTEGRATIONS}}` |
| **Integration Proxies** | Temporary stand-ins for an external dependency that is not available yet | `fercho-be-integration-proxies` | `{{PATH_INTEGRATION_PROXIES}}` |

A tenth skill, `fercho-be-bugfix-refactor`, is cross-cutting and applies to the whole backend scope rather than to one layer.

---

## 3. Modular Skill Definitions

Each skill must be generated under `.claude/skills/` using the mandatory `fercho-be-` prefix, one folder per skill, with this `SKILL.md` format:

```markdown
---
name: fercho-be-<layer>
description: <what the skill does and when to use it, naming the layer and its real project paths so it triggers on tasks that touch them>
---

## Objective
## allowed_files_to_modify
<resolved path globs for this layer, including the location of its tests>
## Constraints
## Workflow
```

This format is mandatory, whatever format other skills in the repository use:
- The four sections are `##` headings in the body, in this order. None is omitted, renamed, or merged with another.
- Scopes and constraints are never written as frontmatter keys (`allowed_files_to_modify:`, `constraints:`). The SDD commands read the `## allowed_files_to_modify` section to decide which files a task may touch.
- `## allowed_files_to_modify` holds one path glob per line; `## Constraints` and `## Workflow` hold one rule or step per line.

The constraints below are written generically. In each generated skill, rewrite them in the idiom of the detected Stack Profile (real library names, real paths, real commands), and drop any constraint that has no equivalent in the project's stack.

### 3.1 Skill: fercho-be-data-scaffolding
- **Objective:** Create and configure persistence models and schema mapping rules.
- **Target Entity / Layer:** `{{PATH_DATA_SCAFFOLDING}}`
- **Constraints:**
  - Strict mapping to project database conventions (naming, keys, column types, nullability).
  - No business logic or repository calls inside entity definitions.
  - Immutable primary key handling where applicable.
  - Schema changes go through the project's migration tool; never edit an already-applied migration.
- **Workflow:**
  1. Inspect database schema or requirement specification.
  2. Generate the entity model under the target path.
  3. Apply schema mappings using the project's mapping style (configuration classes, annotations, schema files).
  4. Create the migration with the project's migration tool and review the generated change.

### 3.2 Skill: fercho-be-repositories
- **Objective:** Build query and mutation interfaces and data access implementations.
- **Target Entity / Layer:** `{{PATH_REPOSITORIES}}`
- **Constraints:**
  - No business rule validation inside queries.
  - Follow the project's async model; propagate cancellation/timeouts where the stack supports them.
  - Read-only operations must not load entities for update (non-tracking / read-only queries where the persistence library supports it).
  - Parameterized queries only; never build SQL by string concatenation.
  - Return projections instead of full entities when the caller needs a subset.
- **Workflow:**
  1. Define the repository contract (interface, protocol, or abstract type per the stack).
  2. Implement the data layer methods.
  3. Wire projections to avoid returning full entities unnecessarily.
  4. Add integration tests against the project's test database setup, or unit tests where the project mocks persistence.

### 3.3 Skill: fercho-be-domain-logic
- **Objective:** Encapsulate core domain calculations, rule validators, and state machines.
- **Target Entity / Layer:** `{{PATH_DOMAIN_LOGIC}}`
- **Constraints:**
  - ZERO external dependencies (No I/O, DB, HTTP, or logging framework imports).
  - Deterministic inputs and outputs; time, randomness, and identifiers are passed in, not read from the environment.
  - Invalid states are rejected with the project's domain error convention (exceptions or result types).
- **Workflow:**
  1. Extract pure business rules from requirements.
  2. Build the domain class/function structure.
  3. Write unit tests covering each rule, boundary values, and invalid inputs.

### 3.4 Skill: fercho-be-services
- **Objective:** Implement application use cases by coordinating repositories and domain classes.
- **Target Entity / Layer:** `{{PATH_SERVICES}}`
- **Constraints:**
  - No HTTP concerns: no status codes, request/response objects, or transport types.
  - Dependencies are injected through the project's DI mechanism; no direct instantiation of repositories or clients.
  - Delegate pure calculations to `{{PATH_DOMAIN_LOGIC}}`.
  - One use case owns its transactional boundary; do not nest or leak transactions.
  - External systems are reached only through `{{PATH_INTEGRATIONS}}`.
- **Workflow:**
  1. Receive the intent/task plan.
  2. Wire repositories and domain logic.
  3. Handle transactional boundaries and error translation.
  4. Unit test the use case with its dependencies mocked.

### 3.5 Skill: fercho-be-orchestrators
- **Objective:** Coordinate flows that span several business services or select between strategies.
- **Target Entity / Layer:** `{{PATH_ORCHESTRATORS}}`
- **Constraints:**
  - Orchestrators coordinate only: no business rules, no direct repository or database access.
  - Call business services (`{{PATH_SERVICES}}`), never controllers or other orchestrators' internals.
  - Strategy selection is explicit and testable (no hidden branching on global state).
  - Define failure handling for partial completion (compensation, retry, or abort) for every multi-step flow.
- **Workflow:**
  1. Identify the services and the decision points of the flow.
  2. Implement the coordination and strategy selection.
  3. Define behavior for each step's failure.
  4. Unit test every branch and failure path with services mocked.

### 3.6 Skill: fercho-be-api-dtos
- **Objective:** Expose endpoints and define contract request/response models.
- **Target Entity / Layer:** `{{PATH_API_DTOS}}`
- **Constraints:**
  - DTOs are immutable, using the stack's idiom (records, frozen/read-only types).
  - Never expose persistence entities directly in requests or responses.
  - Endpoints are thin: validate, call a service or orchestrator, map the result. No business logic.
  - Use the project's validation mechanism, error-mapping convention, and response envelope (if it has one).
  - Apply the project's authentication/authorization mechanism to every new endpoint, or state explicitly why it is public.
- **Workflow:**
  1. Create the request/response contracts.
  2. Map the endpoint/controller action.
  3. Apply authorization and validation.
  4. Add endpoint tests for success, validation failure, and authorization failure.

### 3.7 Skill: fercho-be-di-registration
- **Objective:** Register new components in the dependency injection container or composition root.
- **Target Entity / Layer:** `{{PATH_DI_REGISTRATION}}`
- **Constraints:**
  - Follow the project's existing registration pattern (extension methods, modules, configuration classes, provider lists).
  - Choose lifetimes/scopes deliberately; never inject a shorter-lived dependency into a longer-lived one.
  - No business logic in registration code.
  - Configuration values and secrets come from the project's configuration system, never hard-coded.
- **Workflow:**
  1. Identify the new contracts and implementations to register.
  2. Add the registrations in the module that owns the layer.
  3. Verify the application starts and resolves the new dependencies (startup or container validation test where the project has one).

### 3.8 Skill: fercho-be-integrations
- **Objective:** Implement the permanent clients that talk to external systems, behind boundary contracts. Use it when the external API, SDK, or service is defined and reachable.
- **Target Entity / Layer:** `{{PATH_INTEGRATIONS}}`
- **Constraints:**
  - Transport and execution only: no business rules, flow decisions, or domain validation inside the client.
  - Each external system is wrapped behind a contract owned by this project; external models never leak into services or domain logic.
  - Use the project's configured HTTP/messaging client setup; endpoints, credentials, and timeouts come from configuration.
  - Every call defines its timeout and its failure behavior (retry policy, fallback, or propagated error) following project conventions.
  - Translate external errors to the project's error convention at the boundary.
  - Never log secrets or full payloads containing personal data.
- **Workflow:**
  1. Define the boundary contract and internal models.
  2. Implement the client and the mapping from external to internal models.
  3. Configure timeouts, retries, and error translation.
  4. Test with the external system stubbed (success, error response, timeout).

### 3.9 Skill: fercho-be-integration-proxies
- **Objective:** Create a temporary stand-in (interface plus stubbed implementation) for an external dependency that is not available yet, so the rest of the feature can be built and tested. Use `fercho-be-integrations` once the real system is available.
- **Target Entity / Layer:** `{{PATH_INTEGRATION_PROXIES}}`
- **Constraints:**
  - If the inputs, return types, or expected behavior of the dependency are missing or ambiguous, stop and ask before writing code.
  - The proxy implements the same boundary contract the real client will implement, so replacing it later changes only the DI registration.
  - Every proxy file starts with this comment, in the comment syntax of the language: `TEMPORARY PROXY: Created for isolated development. Replace or remove once the integration or third-party service is available.`
  - Return simulated data in the expected shape; no business rules, data transformation, or orchestration decisions.
  - Do not modify domain models, repositories, persistence configuration, or endpoints.
- **Workflow:**
  1. Confirm the contract (operations, inputs, outputs, error cases) with the developer; ask about anything unclear.
  2. Define the boundary contract, or reuse the one the real client will implement.
  3. Implement the proxy with the mandatory banner and stubbed responses.
  4. Tell the developer which proxy was created and what must replace it.

### 3.10 Skill: fercho-be-bugfix-refactor
- **Objective:** Surgical backend issue resolution using test-driven methodology, with the minimum impact on existing code.
- **Target Entity / Layer:** Global backend scope
- **Constraints:**
  - Mandatory failing test (Red stage) before making production changes.
  - Apply the fix in the layer that owns the defect and respect that layer skill's constraints.
  - Make the failing test pass with the minimum change; do not refactor surrounding code that the fix does not touch.
  - Never skip, delete, or weaken an existing test to get a passing suite.
  - If the root cause is architectural (the fix would change core contracts, rewrite a whole service, or break layer boundaries), stop and escalate instead of fixing.
- **Workflow:**
  - **Phase 1 – Isolate (Red):**
    1. Read the bug report: reproduction steps, stack trace, expected vs. actual behavior.
    2. Write a unit or integration test in the project's existing test location that reproduces the bug, or update an existing one.
    3. Run it and confirm it fails for the reason the report describes.
  - **Phase 2 – Surgical fix (Green):**
    1. Trace the root cause to the specific method or line.
    2. Complexity check: decide whether the fix is localized (missing check, wrong condition, unhandled edge case) or architectural. If architectural, stop, print this notice, and wait for the developer:
       > *ℹ️ Notice: The root cause of this bug requires broader architectural refactoring that exceeds a surgical fix. Escalating to the standard feature workflow.*
    3. Apply the minimum fix in the layer that owns the defect.
    4. Run the new test (it passes) and the existing suite of the module (no regressions).
  - **Phase 3 – Clean up (optional):** light cleanup of the new test or the touched lines only.

---

## 4. System Prompt Patch (`CLAUDE.md` by default - Backend)

The rules are written to `CLAUDE.md` unless the user chooses another instruction file.

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
3. **Markers:** the block below is delimited by `<!-- fercho-be:start -->` and `<!-- fercho-be:end -->`. If the target file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
4. **Block order:** when several blocks are present, keep them in the order `fercho-base`, `fercho-be`, `fercho-fe`, `fercho-sdd`.
5. **Legacy rules:** if `CLAUDE.md`, `CLAUDE.local.md`, or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace every `{{PATH_*}}` placeholder with the resolved path, and remove from rule 1 any layer whose skill was not generated):

```markdown
<!-- fercho-be:start -->
## Backend Layered Execution & Skill Dispatch Rules

1. **Layer decomposition:** before creating, modifying, or refactoring backend code, split the request into the layers it touches (Data Scaffolding, Repositories, Domain Logic, Business Services, Orchestrators, API / DTOs, DI Registration, Third-Party Integrations, Integration Proxies) and match each part to its registered `fercho-be-*` skill in `.claude/skills/`. A request that spans several layers is done as one part per layer, each under its own skill.
2. For each part, load the matching skill and follow its constraints and workflow before editing.
3. Restrict code edits strictly to the `allowed_files_to_modify` paths defined in the corresponding layer skill. Tests for a layer are covered by that layer's skill.
4. **Dependency direction:** API (`{{PATH_API_DTOS}}`) -> Orchestrators (`{{PATH_ORCHESTRATORS}}`) / Services (`{{PATH_SERVICES}}`) -> Domain Logic (`{{PATH_DOMAIN_LOGIC}}`) and Repositories (`{{PATH_REPOSITORIES}}`). Domain Logic depends on nothing.
5. **Fallback Notice:** If a backend task does not match any registered `fercho-be-*` layer skill (e.g., documentation, build scripts, tooling), print this notice before executing:
   > *ℹ️ Note: No custom backend layer skill applies to this task. Executing under standard project conventions.*
<!-- fercho-be:end -->
```
