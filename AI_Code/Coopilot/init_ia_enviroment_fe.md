# SDD Framework Setup Master Template (Frontend Layer Instructions - React JS for GitHub Copilot)

This document serves as the master blueprint for initializing and adapting the **Frontend Layer Instructions Framework** to any React JS repository using **GitHub Copilot**.

> **Execution order:** this template is run **after** [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) (backend layer instructions, when the project has a backend) and **before** [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) (SDD prompt files). The SDD prompt files are generated from the layer instructions deployed here.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a React project, the assistant MUST execute the following workflow:

### Step 1: Stack Detection, Layer Mapping & Codebase Scanning
1. Analyze the project folder structure, package configurations (Vite, Webpack, Next.js, etc.), and architectural patterns.
2. Detect the **Stack Profile**; every generated instruction file must follow it instead of assuming defaults:
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
- **Proposed File Generation Plan:** List of files that will be created: `.github/instructions/fercho-fe-*.instructions.md` and `.github/prompts/fercho-fe-bugfix-refactor.prompt.md`, each instruction file with its `applyTo` globs.
- **Instruction File Changes:** what will be written to `.github/copilot-instructions.md` (section 4), including any unmarked legacy rules found.

*STOP: Wait for user approval before writing files.*

### Step 3: Deployment
Upon user approval, write the following:
1. `.github/instructions/fercho-fe-*.instructions.md` with project-specific scopes and constraints, following the format in section 3. Skip the file of any layer the project does not have.
2. System Instructions patch (section 4): write the rules block to `.github/copilot-instructions.md` between its `fercho-fe` markers, with every `{{PATH_*}}` placeholder replaced by the resolved path.

Do NOT generate `.github/prompts/sdd-*.prompt.md` in this step. Once the layer instructions are deployed, tell the user that the next step is to run `init_ia_enviroment_sdd_framework.md` to generate the SDD prompt files.

---

## 2. Standard Frontend Layer Mapping Matrix

| Generic Frontend Layer | Standard Role | Instruction | Target Project Path Pattern (To be resolved) |
|---|---|---|---|
| **1. UI Components** | Presentational, props-driven components | `fercho-fe-ui-components` | `{{PATH_UI_COMPONENTS}}` |
| **2. Features / Containers** | View assemblers, page containers | `fercho-fe-features` | `{{PATH_FEATURES}}` |
| **3. State Management** | Global client state (Zustand, Redux Toolkit, Context) | `fercho-fe-state-management` | `{{PATH_STATE}}` |
| **4. Custom Hooks** | Reusable business logic, local state orchestrators | `fercho-fe-custom-hooks` | `{{PATH_HOOKS}}` |
| **5. API Services** | HTTP clients, endpoint definitions, response contracts | `fercho-fe-api-services` | `{{PATH_FE_API_SERVICES}}` |
| **6. Routing & Guards** | App routing, layouts, and route protection | `fercho-fe-routing` | `{{PATH_ROUTING}}` |

A seventh unit, `fercho-fe-bugfix-refactor`, is cross-cutting and applies to the whole frontend scope rather than to one layer; it is generated as a prompt file (see section 3).

---

## 3. Frontend Layer Instruction Definitions

Each layer must be generated as one path-specific instruction file under `.github/instructions/`, named `fercho-fe-<layer>.instructions.md`, with this format:

```markdown
---
description: <what the instruction covers, naming the layer>
applyTo: "<comma-separated resolved globs for this layer, including the location of its tests>"
---
# <Layer name> (fercho-fe-<layer>)

## Objective
## Constraints
## Workflow
```

- `applyTo` is what makes Copilot load the file automatically when it works on matching files, and it is also the layer's **edit scope**: a task executed under a layer instruction may only modify files matching its `applyTo` globs.
- Globs of different layers must not overlap. If two layers share a folder, narrow the globs by file-name pattern or report it as a discrepancy in the confirmation step.
- The cross-cutting `fercho-fe-bugfix-refactor` is NOT an instruction file: a rule applied to every frontend file would force its test-first workflow on all work. Generate it as a prompt file, `.github/prompts/fercho-fe-bugfix-refactor.prompt.md` (frontmatter: `description`, `agent: agent`), which the user runs on demand as `/fercho-fe-bugfix-refactor`.

Constraints must be written against the detected Stack Profile (real library names, real paths), not against the generic wording below.

### 3.1 Instruction: fercho-fe-ui-components
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

### 3.2 Instruction: fercho-fe-features
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

### 3.3 Instruction: fercho-fe-state-management
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

### 3.4 Instruction: fercho-fe-custom-hooks
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

### 3.5 Instruction: fercho-fe-api-services
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

### 3.6 Instruction: fercho-fe-routing
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

### 3.7 Prompt: fercho-fe-bugfix-refactor
- **Objective:** Surgical resolution of UI and state-flow issues using test-driven methodology.
- **Target Path:** Global frontend scope
- **Constraints:**
  - Mandatory failing test (Red stage) reproducing the bug before making production changes.
  - Apply the fix in the layer that owns the defect and respect that layer instruction's constraints.
  - Do not introduce structural refactoring unless explicitly authorized.
- **Workflow:**
  1. Reproduce the bug with a component, hook, or integration test.
  2. Apply the minimum fix in the corresponding layer.
  3. Verify the fix passes (Green stage) and the existing suite still passes.

---

## 4. Global Instructions Patch (`.github/copilot-instructions.md` - Frontend)

The rules are written to `.github/copilot-instructions.md`, the file GitHub Copilot loads for every request in the repository.

**Write rules:**
1. **Target file:** `.github/copilot-instructions.md`. Create it if it does not exist.
2. **Markers:** the block below is delimited by `<!-- fercho-fe:start -->` and `<!-- fercho-fe:end -->`. If the file already contains these markers, replace everything between them; otherwise append the block at the end of the file. Never write the block twice and never modify content outside the markers.
3. **Block order:** when several blocks are present, keep them in the order `fercho-be`, `fercho-fe`, `fercho-sdd`.
4. **Legacy rules:** if `.github/copilot-instructions.md` or `AGENTS.md` already contains an unmarked copy of this section from an earlier setup, report it in the confirmation step and replace it with the marked block only after the user approves.

Block to write (replace every `{{PATH_*}}` placeholder with the resolved path):

```markdown
<!-- fercho-fe:start -->
## Frontend Layered Execution & Separation Rules

1. **Strict Separation of Concerns:**
   - UI Components (`{{PATH_UI_COMPONENTS}}`) must remain **presentational** (props in, JSX out, UI-only local state, zero API/business logic).
   - Business logic must reside entirely inside Custom Hooks (`{{PATH_HOOKS}}`) or API Services (`{{PATH_FE_API_SERVICES}}`).
   - Never mix business logic directly inside render blocks.
2. Before modifying frontend code, identify the target layer and follow its `fercho-fe-*` instruction file in `.github/instructions/`.
3. Restrict code edits strictly to the `applyTo` paths of the corresponding layer instruction.
4. **Fallback Notice:** If a frontend task does not match any registered `fercho-fe-*` layer instruction, print this notice before executing:
   > *ℹ️ Note: No custom frontend layer instruction applies to this task. Executing under standard React project conventions.*
5. Bugs are fixed with the `/fercho-fe-bugfix-refactor` prompt (failing test first).
6. **Frontend Lookup Order:** when looking for existing frontend code to reuse, check Project UI Guidelines -> Existing Hooks -> Existing Components and their props before creating anything new.
<!-- fercho-fe:end -->
```
