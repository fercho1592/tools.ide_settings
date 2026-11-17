# SDD Framework Setup Master Template (Backend Layer Instructions for GitHub Copilot)

This document serves as the master blueprint for initializing and adapting the **Backend Layer Instructions Framework** to any repository using **GitHub Copilot**, regardless of language or framework.

> **Execution order:** this template is run **before** [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) (SDD prompt files). The SDD prompt files are generated from the layer instructions deployed here (and from the frontend layer instructions in [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md), when the project has a frontend). On a new project with no instructions file, [init_ia_enviroment_copilot.md](./init_ia_enviroment_copilot.md) (base project rules) runs before all of them.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Stack Detection, Layer Mapping & Codebase Scanning
1. Analyze the project folder structure, package configurations, and architectural patterns.
2. Detect the **Stack Profile**; every generated instruction file must follow it instead of assuming defaults:
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
- **Proposed File Generation Plan:** List of files that will be created: `.github/instructions/fercho-be-*.instructions.md` and `.github/prompts/fercho-be-bugfix-refactor.prompt.md`, each instruction file with its `applyTo` globs.
- **Instruction File Changes:** what will be written to `.github/copilot-instructions.md` (section 4), including any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 3: Deployment
Upon user approval, write the following:
1. `.github/instructions/fercho-be-*.instructions.md` with project-specific scopes and constraints, following the format in section 3. Skip the file of any layer the project does not have.
2. System Instructions patch (section 4): write the rules block to `.github/copilot-instructions.md` between its `fercho-be` markers, with every `{{PATH_*}}` placeholder replaced by the resolved path.

3. **Format check:** re-read every generated `fercho-be-*.instructions.md` and confirm that it has the `description` and `applyTo` frontmatter, that the three sections of section 3 are present as `##` headings, in order and not empty, that its `applyTo` globs do not overlap those of another layer, and that no `{{PATH_*}}` placeholder is left. Do the same for the bugfix prompt file (`description` and `agent` frontmatter). Fix any file that fails before reporting the deployment as complete.

Do NOT generate `.github/prompts/sdd-*.prompt.md` in this step. Once the layer instructions are deployed, tell the user that the next step is to run `init_ia_enviroment_fe.md` (if the project has a frontend) and then `init_ia_enviroment_sdd_framework.md` to generate the SDD prompt files.

---

## 2. Standard Layer Mapping Matrix

| Generic Layer | Standard Role | Instruction | Target Project Path Pattern (To be resolved) |
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

A tenth unit, `fercho-be-bugfix-refactor`, is cross-cutting and applies to the whole backend scope rather than to one layer; it is generated as a prompt file (see section 3).

---

## 3. Layer Instruction Definitions

Each layer must be generated as one path-specific instruction file under `.github/instructions/`, named `fercho-be-<layer>.instructions.md`, with this format:

```markdown
---
description: <what the instruction covers, naming the layer>
applyTo: "<comma-separated resolved globs for this layer, including the location of its tests>"
---
# <Layer name> (fercho-be-<layer>)

## Objective
## Constraints
## Workflow
```

This format is mandatory, whatever format other instruction files in the repository use:
- The three sections are `##` headings in the body, in this order. None is omitted, renamed, or merged with another.
- Scopes are never written as extra frontmatter keys or inside the body: the layer's edit scope is its `applyTo`. The SDD prompt files read `applyTo` to decide which files a task may touch.
- `## Constraints` and `## Workflow` hold one rule or step per line.

- `applyTo` is what makes Copilot load the file automatically when it works on matching files, and it is also the layer's **edit scope**: a task executed under a layer instruction may only modify files matching its `applyTo` globs.
- Globs of different layers must not overlap. If two layers share a folder, narrow the globs by file-name pattern or report it as a discrepancy in the confirmation step.
- The cross-cutting `fercho-be-bugfix-refactor` is NOT an instruction file: a rule applied to every backend file would force its test-first workflow on all work. Generate it as a prompt file, `.github/prompts/fercho-be-bugfix-refactor.prompt.md` (frontmatter: `description`, `agent: agent`), which the user runs on demand as `/fercho-be-bugfix-refactor`.

The constraints below are written generically. In each generated file, rewrite them in the idiom of the detected Stack Profile (real library names, real paths, real commands), and drop any constraint that has no equivalent in the project's stack.

### 3.1 Instruction: fercho-be-data-scaffolding
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

### 3.2 Instruction: fercho-be-repositories
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

### 3.3 Instruction: fercho-be-domain-logic
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

### 3.4 Instruction: fercho-be-services
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

### 3.5 Instruction: fercho-be-orchestrators
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

### 3.6 Instruction: fercho-be-api-dtos
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

### 3.7 Instruction: fercho-be-di-registration
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

### 3.8 Instruction: fercho-be-integrations
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

### 3.9 Instruction: fercho-be-integration-proxies
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

### 3.10 Prompt: fercho-be-bugfix-refactor
- **Objective:** Surgical backend issue resolution using test-driven methodology, with the minimum impact on existing code.
- **Target Entity / Layer:** Global backend scope
- **Constraints:**
  - Mandatory failing test (Red stage) before making production changes.
  - Apply the fix in the layer that owns the defect and respect that layer instruction's constraints.
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

## 4. Global Instructions Patch (`.github/copilot-instructions.md` - Backend)

The rules are written to `.github/copilot-instructions.md`, the file GitHub Copilot loads for every request in the repository.

**Write rules:**
1. **Target file:** `.github/copilot-instructions.md`. Create it if it does not exist.
2. **Markers:** the block below is delimited by `<!-- fercho-be:start -->` and `<!-- fercho-be:end -->`. If the file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
3. **Block order:** when several blocks are present, keep them in the order `fercho-base`, `fercho-be`, `fercho-fe`, `fercho-sdd`.
4. **Legacy rules:** if `.github/copilot-instructions.md` or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace every `{{PATH_*}}` placeholder with the resolved path, and remove from rule 1 any layer whose instruction was not generated):

```markdown
<!-- fercho-be:start -->
## Backend Layered Execution & Instruction Dispatch Rules

1. **Layer decomposition:** before creating, modifying, or refactoring backend code, split the request into the layers it touches (Data Scaffolding, Repositories, Domain Logic, Business Services, Orchestrators, API / DTOs, DI Registration, Third-Party Integrations, Integration Proxies) and match each part to its `fercho-be-*` instruction file in `.github/instructions/`. A request that spans several layers is done as one part per layer, each under its own instruction.
2. For each part, read the matching instruction file and follow its constraints and workflow before editing; do not rely only on Copilot attaching it automatically.
3. Restrict code edits strictly to the `applyTo` paths of the corresponding layer instruction. Tests for a layer are covered by that layer's instruction.
4. **Dependency direction:** API (`{{PATH_API_DTOS}}`) -> Orchestrators (`{{PATH_ORCHESTRATORS}}`) / Services (`{{PATH_SERVICES}}`) -> Domain Logic (`{{PATH_DOMAIN_LOGIC}}`) and Repositories (`{{PATH_REPOSITORIES}}`). Domain Logic depends on nothing.
5. Bugs are fixed with the `/fercho-be-bugfix-refactor` prompt (failing test first).
6. **Fallback Notice:** If a backend task does not match any registered `fercho-be-*` layer instruction (e.g., documentation, build scripts, tooling), print this notice before executing:
   > *ℹ️ Note: No custom backend layer instruction applies to this task. Executing under standard project conventions.*
<!-- fercho-be:end -->
```
