# SDD Framework Setup Master Template (Frontend Skills - React JS)

This document serves as the master blueprint for initializing and adapting the **Frontend Architectural Skills Framework** to any React JS repository.

> **Execution order:** this template is run **after** [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) (backend skills, when the project has a backend) and **before** [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) (SDD commands). The SDD commands are generated from the skills deployed here. On a new project with no rules file, [init_ia_enviroment_claude.md](./init_ia_enviroment_claude.md) (base project rules) runs before all of them.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a React project, the assistant MUST execute the following workflow:

### Step 1: Stack Detection, Layer Mapping & Codebase Scanning
1. Analyze the project folder structure, package configurations (Vite, Webpack, Next.js, etc.), and architectural patterns.
2. Detect the **Stack Profile**; every generated skill must follow it instead of assuming defaults:
   - **Language:** TypeScript or JavaScript (and, for JavaScript, whether PropTypes or JSDoc types are used).
   - **Server state / data fetching:** TanStack Query, RTK Query, SWR, or hand-written hooks.
   - **Global state:** Zustand, Redux Toolkit, Context, other, or none.
   - **HTTP client:** configured Axios instance, fetch wrapper, generated client.
   - **Styling:** CSS Modules, Tailwind, styled-components, design-system library, etc.
   - **Routing:** React Router, TanStack Router, Next.js file-based routing.
   - **Testing:** test runner and libraries (Vitest/Jest, React Testing Library, Playwright/Cypress) and the commands to run them.
3. Identify the project's frontend architectural layers (e.g., UI components, feature modules, custom hooks, state management, API services, routing).
4. Map the generic framework layers to the specific paths in this repository.

### Step 2: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Stack Profile:** the detected values from Step 1.
- **Layer Mapping Table:** Standard Layer vs. Detected Project Path.
- **Adjustment Points / Discrepancies:** Note any layer that doesn't exist, doesn't cleanly align, or requires custom configuration (e.g., no global state library, file-based routing).
- **Proposed File Generation Plan:** List of skills that will be created under `.claude/skills/fercho-fe-*/`.
- **Existing Skills:** any skill already under `.claude/skills/` that covers one of these layers under another name or in another format. For each one, propose to replace it with the generated skill or to keep it; never leave two skills owning the same layer.
- **Rules File:** the instruction file where the rules block will be written (section 4). If no earlier run already fixed it, ask the user to choose between `CLAUDE.md` (default), `CLAUDE.local.md`, and `AGENTS.md`. Include any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 3: Deployment
Upon user approval, write the following:
1. `.claude/skills/fercho-fe-*/SKILL.md` with project-specific scopes and constraints, following the format in section 3. Skip the skill of any layer the project does not have.
2. System Instructions patch (section 4): write the rules block between its `fercho-fe` markers in the rules file resolved in section 4 (`CLAUDE.md` by default), with every `{{PATH_*}}` placeholder replaced by the resolved path.

3. **Format check:** re-read every generated `SKILL.md` and confirm that it has the `name` and `description` frontmatter, that the four sections of section 3 are present as `##` headings, in order and not empty, and that no `{{PATH_*}}` placeholder is left. Fix any file that fails before reporting the deployment as complete.

Do NOT generate `.claude/commands/sdd-*.md` in this step. Once the skills are deployed, tell the user that the next step is to run `init_ia_enviroment_sdd_framework.md` to generate the SDD commands.

---

## 2. Standard Frontend Layer Mapping Matrix

| Generic Frontend Layer | Standard Role | Skill | Target Project Path Pattern (To be resolved) |
|---|---|---|---|
| **1. UI Components** | Presentational, props-driven components | `fercho-fe-ui-components` | `{{PATH_UI_COMPONENTS}}` |
| **2. Features / Containers** | View assemblers, page containers | `fercho-fe-features` | `{{PATH_FEATURES}}` |
| **3. State Management** | Global client state (Zustand, Redux Toolkit, Context) | `fercho-fe-state-management` | `{{PATH_STATE}}` |
| **4. Custom Hooks** | Reusable business logic, local state orchestrators | `fercho-fe-custom-hooks` | `{{PATH_HOOKS}}` |
| **5. API Services** | HTTP clients, endpoint definitions, response contracts | `fercho-fe-api-services` | `{{PATH_FE_API_SERVICES}}` |
| **6. Routing & Guards** | App routing, layouts, and route protection | `fercho-fe-routing` | `{{PATH_ROUTING}}` |

A seventh skill, `fercho-fe-bugfix-refactor`, is cross-cutting and applies to the whole frontend scope rather than to one layer.

---

## 3. Modular Frontend Skill Definitions

Each skill must be generated under `.claude/skills/` using the mandatory `fercho-fe-` prefix, one folder per skill, with this `SKILL.md` format:

```markdown
---
name: fercho-fe-<layer>
description: <what the skill does and when to use it, naming the layer and its real project paths so it triggers on tasks that touch them>
---

## Objective
## allowed_files_to_modify
<resolved path globs for this layer, including its co-located test files>
## Constraints
## Workflow
```

This format is mandatory, whatever format other skills in the repository use:
- The four sections are `##` headings in the body, in this order. None is omitted, renamed, or merged with another.
- Scopes and constraints are never written as frontmatter keys (`allowed_files_to_modify:`, `constraints:`). The SDD commands read the `## allowed_files_to_modify` section to decide which files a task may touch.
- `## allowed_files_to_modify` holds one path glob per line; `## Constraints` and `## Workflow` hold one rule or step per line.

Constraints must be written against the detected Stack Profile (real library names, real paths), not against the generic wording below.

### 3.1 Skill: fercho-fe-ui-components
- **Objective:** Build presentational, props-driven visual components.
- **Target Path:** `{{PATH_UI_COMPONENTS}}`
- **Constraints:**
  - **Zero Business Logic:** No direct API calls, no global state selectors (`useStore`, `useSelector`), no data-fetching hooks, and no complex conditional data processing.
  - Local **UI-only** state is allowed (open/closed, hover, focus, input drafts); business and server state arrive through props.
  - Props are typed with TypeScript interfaces/types in TypeScript projects; in JavaScript projects use the project's convention (PropTypes or JSDoc).
  - Style only with the project's detected styling approach; reuse existing design tokens and components before creating new ones.
  - Accessible by default: semantic HTML elements, labelled controls, keyboard operability, visible focus.
- **Workflow:**
  1. Define the component prop contract.
  2. Implement clean JSX structure and styling.
  3. Emit user actions entirely through callback props.
  4. Add a render/interaction test following the project's testing setup.

### 3.2 Skill: fercho-fe-features
- **Objective:** Assemble pages and feature views by connecting hooks and state to presentational components.
- **Target Path:** `{{PATH_FEATURES}}`
- **Constraints:**
  - Containers orchestrate only: they call hooks, read state, and pass data and callbacks down as props.
  - No direct HTTP calls; data access goes through custom hooks or API services.
  - No business rules or data transformation inline in JSX; extract them to hooks or pure functions.
  - Handle the loading, error, and empty states of every async dependency.
  - Keep layout-only markup here; reusable visual pieces belong in `{{PATH_UI_COMPONENTS}}`.
- **Workflow:**
  1. Identify the data and actions the view needs from the task spec.
  2. Wire the corresponding hooks and state selectors.
  3. Compose UI components, passing data and callbacks as props.
  4. Cover loading, error, empty, and success states.

### 3.3 Skill: fercho-fe-state-management
- **Objective:** Manage shared global client state across multiple domain views.
- **Target Path:** `{{PATH_STATE}}`
- **Constraints:**
  - Use the project's detected state library; do not introduce a second one.
  - Avoid global state for strictly local component UI state.
  - Do not duplicate server data in the global store when the project uses a server-state library (TanStack Query, RTK Query, SWR).
  - Keep state updates predictable, immutable, and typed per the project's language convention.
  - No JSX and no direct HTTP calls inside stores.
- **Workflow:**
  1. Define the store, slice, or context structure.
  2. Implement pure actions and mutators.
  3. Expose selectors or selector hooks for consumers.
  4. Unit test actions and selectors.

### 3.4 Skill: fercho-fe-custom-hooks
- **Objective:** Isolate business logic, side effects, and complex state management away from views.
- **Target Path:** `{{PATH_HOOKS}}`
- **Constraints:**
  - Mandatory `use` prefix naming convention; follow the Rules of Hooks.
  - Contains NO JSX markup.
  - Server data: when the project uses a server-state library, wrap its primitives (`useQuery`, `useMutation`, etc.) and follow its existing query-key and caching conventions. Only when the project has none, manage `loading`, `error`, and `data` manually.
  - HTTP access goes through `{{PATH_FE_API_SERVICES}}`, never through inline `fetch`/Axios calls.
  - Correct effect dependencies and cleanup (abort or ignore stale requests, remove listeners and timers).
- **Workflow:**
  1. Extract business rules and state requirements from the task spec.
  2. Implement the hook handling async lifecycle and state transitions.
  3. Return a clean, typed API contract for container components.
  4. Unit test the hook with the project's hook-testing utilities.

### 3.5 Skill: fercho-fe-api-services
- **Objective:** Centralize HTTP communication and map backend data contracts.
- **Target Path:** `{{PATH_FE_API_SERVICES}}`
- **Constraints:**
  - Exclusive use of the project's configured HTTP client instance; no ad-hoc clients, base URLs, or hard-coded hosts.
  - Request and response contracts are typed per the project's language convention (TypeScript types, or JSDoc in JavaScript projects).
  - Zero React imports, hooks, JSX, or DOM access inside this layer.
  - Auth headers and error normalization live in the shared client/interceptors, not in individual endpoint functions.
- **Workflow:**
  1. Define the request/response data contracts.
  2. Implement asynchronous endpoint functions.
  3. Map transport errors to the project's error shape.
  4. Unit test with the HTTP layer mocked.

### 3.6 Skill: fercho-fe-routing
- **Objective:** Define routes, layouts, and route protection.
- **Target Path:** `{{PATH_ROUTING}}`
- **Constraints:**
  - Use the project's detected router and its existing conventions (central route config or file-based routing).
  - Route paths are defined once as constants or helpers; no hard-coded path strings scattered across components.
  - Guards read authentication/authorization state from hooks or state; they contain no HTTP calls or business rules.
  - Follow the project's existing code-splitting (lazy loading) pattern for route-level views.
  - Route elements render feature containers from `{{PATH_FEATURES}}`, not ad-hoc markup.
- **Workflow:**
  1. Register the route and its layout.
  2. Apply guards and redirects.
  3. Wire lazy loading and fallback UI where the project uses it.
  4. Test guarded and unguarded navigation.

### 3.7 Skill: fercho-fe-bugfix-refactor
- **Objective:** Surgical resolution of UI and state-flow issues using test-driven methodology, with the minimum impact on existing code.
- **Target Path:** Global frontend scope
- **Constraints:**
  - Mandatory failing test (Red stage) reproducing the bug before making production changes.
  - Apply the fix in the layer that owns the defect and respect that layer skill's constraints.
  - Make the failing test pass with the minimum change; do not refactor surrounding code that the fix does not touch.
  - Never skip, delete, or weaken an existing test to get a passing suite.
  - If the root cause is architectural (the fix would change a shared component's prop contract, restructure global state, rewrite a whole feature, or break layer boundaries), stop and escalate instead of fixing.
- **Workflow:**
  - **Phase 1 – Isolate (Red):**
    1. Read the bug report: reproduction steps, console or network errors, expected vs. actual behavior.
    2. Write a component, hook, or integration test in the project's existing test location that reproduces the bug, or update an existing one.
    3. Run it and confirm it fails for the reason the report describes.
  - **Phase 2 – Surgical fix (Green):**
    1. Trace the root cause to the specific component, hook, store, or service.
    2. Complexity check: decide whether the fix is localized (missing guard, wrong condition, stale effect dependency, unhandled state) or architectural. If architectural, stop, print this notice, and wait for the developer:
       > *ℹ️ Notice: The root cause of this bug requires broader architectural refactoring that exceeds a surgical fix. Escalating to the standard feature workflow.*
    3. Apply the minimum fix in the layer that owns the defect.
    4. Run the new test (it passes) and the existing suite of the affected area (no regressions).
  - **Phase 3 – Clean up (optional):** light cleanup of the new test or the touched lines only.

---

## 4. System Prompt Patch (`CLAUDE.md` by default - Frontend)

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
3. **Markers:** the block below is delimited by `<!-- fercho-fe:start -->` and `<!-- fercho-fe:end -->`. If the target file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
4. **Block order:** when several blocks are present, keep them in the order `fercho-base`, `fercho-be`, `fercho-fe`, `fercho-sdd`.
5. **Legacy rules:** if `CLAUDE.md`, `CLAUDE.local.md`, or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace every `{{PATH_*}}` placeholder with the resolved path, and remove from rule 2 any layer whose skill was not generated):

```markdown
<!-- fercho-fe:start -->
## Frontend Layered Execution & Separation Rules

1. **Strict Separation of Concerns:**
   - UI Components (`{{PATH_UI_COMPONENTS}}`) must remain **presentational** (props in, JSX out, UI-only local state, zero API/business logic).
   - Business logic must reside entirely inside Custom Hooks (`{{PATH_HOOKS}}`) or API Services (`{{PATH_FE_API_SERVICES}}`).
   - Never mix business logic directly inside render blocks.
2. **Layer decomposition:** before creating, modifying, or refactoring frontend code, split the request into the layers it touches (UI Components, Features / Containers, State Management, Custom Hooks, API Services, Routing & Guards) and match each part to its registered `fercho-fe-*` skill in `.claude/skills/`. A request that spans several layers is done as one part per layer, each under its own skill.
3. For each part, load the matching skill and follow its constraints and workflow before editing.
4. Restrict code edits strictly to the `allowed_files_to_modify` paths defined in the corresponding layer skill.
5. **Fallback Notice:** If a frontend task does not match any registered `fercho-fe-*` layer skill, print this notice before executing:
   > *ℹ️ Note: No custom frontend layer skill applies to this task. Executing under standard React project conventions.*
6. **Frontend Lookup Order:** when looking for existing frontend code to reuse, check Project UI Guidelines -> Existing Hooks -> Existing Components and their props before creating anything new.
<!-- fercho-fe:end -->
```
