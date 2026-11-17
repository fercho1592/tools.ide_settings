# SDD Framework Setup Master Template (Base Project Rules)

This document serves as the master blueprint for creating the **base project rules** of a repository: the instruction file (`CLAUDE.md` by default) that tells an AI assistant how the project is built, run, tested, and kept secure. It applies to any language or framework.

> **Execution order:** this template is the **first** setup step, and it is **optional**. It runs only for a new project or a project that has no rules file yet. A project that already has its own rules skips it and starts at step 2.
>
> 1. **This file** – base project rules (`fercho-base` block)
> 2. [init_ia_enviroment_be.md](./init_ia_enviroment_be.md) – backend skills (`.claude/skills/fercho-be-*`)
> 3. [init_ia_enviroment_fe.md](./init_ia_enviroment_fe.md) – frontend skills (`.claude/skills/fercho-fe-*`)
> 4. [init_ia_enviroment_sdd_framework.md](./init_ia_enviroment_sdd_framework.md) – SDD commands (`.claude/commands/sdd-*.md`)
>
> It goes first because it creates the rules file the other three templates append their blocks to, and it fixes which file that is.

---

## 1. Setup Agent Execution Protocol

When this file is provided to an AI assistant with the request to initialize or configure a project, the assistant MUST execute the following workflow:

### Step 1: Applicability Check
1. Look for existing rules files at the repository root: `CLAUDE.md`, `CLAUDE.local.md`, `AGENTS.md` (and `.claude/CLAUDE.md`).
2. Decide which case applies:

   | Case | What exists | Action |
   |---|---|---|
   | **New** | No rules file, or only empty files | Continue with Step 2. |
   | **Framework only** | A rules file whose only content is `fercho-*` marker blocks | Continue with Step 2; the base block is added above the existing blocks. |
   | **Re-run** | A rules file that already contains the `fercho-base` block | Continue with Step 2; the block is replaced, nothing else is touched. |
   | **Already initialized** | A rules file with any other project rules, written by hand or by another tool | STOP. Write nothing. |

3. In the **Already initialized** case, tell the user that this template does not apply because the project already has its own rules, and that the next step is `init_ia_enviroment_be.md`. Do not merge, rewrite, or reorder the existing rules. On request only, list the sections of section 2 that the existing file does not cover, as suggestions for the user to add by hand.

### Step 2: Project Scan
1. Analyze the repository to fill the sections of section 2. For each section, collect the evidence named in its "Detect" line: configuration files, scripts, pipelines, and patterns repeated in the code.
2. Detect the **Stack Profile** for each side the project has (backend, frontend): language and framework, persistence, API style, UI library, test tooling.
3. **Evidence rule:** a rule is written only when the repository shows it (a configuration file, a script, or a pattern used consistently in the code). A section with no evidence is omitted or raised as a question in Step 3. Never invent a convention, a command, or a path.
4. **No duplication rule:** do not write the rules that the layer skills carry (section 2.2). Those are set up by the skill templates.

### Step 3: Confirmation Proposal (Interactive Step)
Generate a confirmation report for the user containing:
- **Applicability:** the case found in Step 1.
- **Stack Profile:** the detected values per side.
- **Section Plan:** for each section of section 2, the detected content in a few lines, or *omitted* with the reason.
- **Questions:** conventions the scan could not settle (e.g., two commit formats in the history, no visible branch pattern).
- **Rules File:** ask the user to choose between `CLAUDE.md` (default), `CLAUDE.local.md`, and `AGENTS.md` (section 4). The later templates reuse this choice without asking again.

*STOP: Wait for user approval before writing files.*

### Step 4: Deployment
Upon user approval:
1. Write the base rules block between its `fercho-base` markers in the chosen rules file, following section 3, with every `{{...}}` placeholder replaced by the detected value and every omitted section removed.
2. **Format check:** re-read the written block and confirm that no `{{...}}` placeholder is left, that no section is empty, that every path and command in it exists in the repository, and that no secret or credential was copied into it. Fix any failure before reporting the deployment as complete.
3. Tell the user that the next step is `init_ia_enviroment_be.md` (or `init_ia_enviroment_fe.md` for a frontend-only project).

---

## 2. Section Catalogue

### 2.1 Sections of the Base Rules

| # | Section | Kind | Detect | Omit when |
|---|---|---|---|---|
| 1 | Proactive contract | Generic | Nothing; written as given | Never |
| 2 | Enforcement layers | Detected | Hooks (`.claude/hooks`, git hooks), formatters, linters, analyzers, build-time checks, CI gates | The project has none |
| 3 | Mandatory backend rules | Detected | The backend conventions the code follows without exception and a reviewer would reject | No backend |
| 4 | Mandatory frontend rules | Detected | The frontend conventions the code follows without exception and a reviewer would reject | No frontend |
| 5 | Git and ticket conventions | Detected | Branch names and commit subjects in the history, commit hooks, CI title checks | No visible pattern and the user gives none |
| 6 | Architecture | Detected | Applications, services, and shared projects; how a request travels between them; architecture docs | Single-project repository with nothing to explain |
| 7 | Read-first docs index | Detected | Docs under the project's docs folder that are the contract for an area | The project has no such docs |
| 8 | Running the project | Detected | Run modes, first-time setup, required environment files | Never |
| 9 | Common commands | Detected | Build, test, lint, format, and other routine commands per side | Never |
| 10 | Backend standards | Detected + generic | Language standards, comment rule, error-response convention | No backend |
| 11 | Backend testing standards | Detected | Test frameworks, banned libraries, test structure, shared test helpers | No backend tests |
| 12 | Security standards | Generic + detected | Authentication mechanism, trust boundary, ownership checks, secrets handling, file handling | Never |
| 13 | Frontend standards | Detected | Folder structure, forms, internationalization, environment variables | No frontend |

"Generic" text is written as given in section 3. "Detected" text is written from the scan.

### 2.2 Rules Left to the Layer Skills

These rules are set up by `init_ia_enviroment_be.md` and `init_ia_enviroment_fe.md` inside the layer skills and their rules blocks. Do not restate them in the base rules:

| Area | Carried by |
|---|---|
| Typed ORM projections, async and cancellation, read-only queries, parameterized queries | `fercho-be-repositories` |
| Thin endpoints, immutable DTOs, validation, error mapping, response envelope, endpoint authorization | `fercho-be-api-dtos` |
| Configuration and secrets never hard-coded; lifetimes | `fercho-be-di-registration` |
| Timeouts, retries, error translation, no secrets in logs for external calls | `fercho-be-integrations` |
| A test step per layer with the project's test tooling | Every `fercho-be-*` and `fercho-fe-*` skill |
| Layer dependency direction | `fercho-be` rules block |
| Presentational components, typed props, styling approach, hooks, global state, HTTP client instance, routing and guards | The `fercho-fe-*` skills and the `fercho-fe` rules block |
| SDD flow, questions and decisions, information lookup hierarchy | `fercho-sdd` rules block |

---

## 3. Section Definitions

For each section: what to write and how. Rules are written as short imperative bullets, one rule per bullet, each naming the real library, path, or command.

### 3.1 Proactive Contract
Written as given in section 4. It states that the mandatory sections apply when code is written, not only at review time.

### 3.2 Enforcement Layers
List each mechanism that enforces the rules, from the most to the least reliable, with what it catches: write-time hooks, formatters and linters, build-time checks, CI gates. State which rules are **not** enforced mechanically and stay with review. Omit the section when the project has no enforcement; do not propose new tooling here.

### 3.3 Mandatory Backend Rules
A checklist of at most ten rules. Each rule is one sentence, says what to do and what is forbidden, and links to the detailed section (3.10–3.12) that defines it. Candidates to look for:
- Code style that is not left to the formatter (bracing, naming).
- Libraries that are mandatory or banned (e.g., a banned mocking library).
- Persistence rules beyond the repository skill: fields every entity must carry (audit or tenancy columns), queries that must stay index-friendly, collections that are never `null`.
- Caching and concurrency patterns the project repeats.
- How asynchronous side effects are published (events, queues) and when not to use them.

Write only the rules the code follows without exception.

### 3.4 Mandatory Frontend Rules
Same format as 3.3. Candidates to look for:
- Mandatory internationalization: no hard-coded user-visible text, how keys are named, the command to regenerate keys or types.
- The form library and the schema-validation library, and any banned alternative.
- Resilience attached to the HTTP client (retry, circuit breaker) and how responses are unwrapped.
- How dynamic or server-driven forms bind their fields.
- The rule for test files (e.g., one sibling test per source file) and where exclusions are declared.

### 3.5 Git and Ticket Conventions
- **Branch naming:** the pattern with a real example.
- **Commit message format:** the pattern, which part is mandatory, and a real example.
- **Enforcement:** the hooks and CI scripts that enforce them, by path, and the sentence: *"Keep this section in sync with those files; they are the actual enforcement."*

If the history shows more than one format, ask in Step 3 instead of choosing.

### 3.6 Architecture
- The request flow as a short diagram (client → gateway or API → services → data).
- One bullet per application, service, or shared project: its name and its role in one line.
- Where the trust boundary is, when the project has one (see 3.12).
- Links to the architecture docs.

Do not repeat the layer dependency direction; the `fercho-be` block states it.

### 3.7 Read-First Docs Index
A table of the docs that are the contract for an area, so the assistant reads the doc before changing that area:

| Area | Read first | Rules to preserve |
|---|---|---|
| `<area>` | `<path to doc>` | `<one or two rules from the doc that must never be broken>` |

Include a doc only when it describes how an area must behave. Take the "rules to preserve" from the doc itself; when the doc states none, leave the cell empty instead of inventing one.

### 3.8 Running the Project
The commands for each run mode (e.g., from source, in containers) and the first-time setup, in the order a new developer runs them. Name the environment file to copy from its example; never copy its values.

### 3.9 Common Commands
One code block per side with the routine commands: build, test (all and a single test), lint, format, and any generation step (e.g., translation keys, API clients). The SDD commands detect their own verification commands; this section is the full list for a developer.

### 3.10 Backend Standards
Three parts:
1. **Language standards:** the rules detected for bracing, collections, strings, enums, caching, exceptions, and code documentation, each as one bullet.
2. **Docs and comments (generic, written as given):** describe what the code does now and its contract; do not narrate change history, the bug that prompted the code, or the alternative not taken.
3. **Error responses:** the response envelope type and where it is defined, the helper methods to use per status code, where error codes are declared (constants, never string literals), how framework-level errors are wrapped, and a short review checklist.

Omit a part that has no evidence.

### 3.11 Backend Testing Standards
The test framework and assertion library, the only mocking library allowed and any banned one, the test structure the project follows (e.g., Arrange-Act-Assert), and the shared test-helper project or folder to check before writing new factories.

### 3.12 Security Standards
Written from the generic rules in section 4, each completed with the project's real mechanism:

| Rule | Complete it with |
|---|---|
| Object-level authorization | The service, guard, or policy that checks that the caller owns a resource |
| Input and path safety | Where file names, ids, and slugs are validated; the allowlists in use |
| Cryptography and secrets | The token validation settings; where secrets are loaded and how a missing one fails |
| Fail closed | The guards, filters, and validators that must reject when misconfigured |
| Output and link safety | How user-influenced or rich content is rendered and sanitized |
| Logging | What is redacted and where |
| Trust boundary | The component that authenticates the caller and what the services behind it trust |
| Framing | Where the allowed embedding origins are configured |

Keep a rule in its generic form when the project has no specific mechanism for it. The rule about deferred changes is always written as given.

### 3.13 Frontend Standards
- **Folder structure:** what each page or feature folder owns and where shared code lives.
- **Data mapping:** where backend-to-frontend mapping code lives.
- **Forms:** the form and schema libraries and how types are derived.
- **Internationalization:** how text is translated, how keys are named, the command to run after a key changes.
- **Environment variables:** the required prefix, the files to update, and where types are declared.
- **Testing:** the test runner and library, what tests assert (behavior, not implementation), and where shared fixtures live.

---

## 4. System Prompt Patch (`CLAUDE.md` by default - Base Rules)

The base rules are written to `CLAUDE.md` unless the user chooses another instruction file.

**Write rules:**
1. **Target file:** ask the user in the confirmation step where the rules should be written, offering these options:
   - **`CLAUDE.md`** (default): project rules loaded by Claude Code and shared with the team through version control.
   - **`CLAUDE.local.md`**: personal rules for this machine only. They are not shared with the team; make sure the file is listed in `.gitignore`.
   - **`AGENTS.md`**: one rules file shared with other AI assistants. Claude Code does not read it directly, so `CLAUDE.md` must import it (see rule 2).

   If a rules file already contains a `fercho-*` marker block, use that same file and do not ask. If the user approves without choosing, use `CLAUDE.md`. The file lives at the repository root; create it if it does not exist.
2. **Import (only when the target is `AGENTS.md`):** `CLAUDE.md` must contain the line `@AGENTS.md`. If `CLAUDE.md` does not exist, create it with that line.
3. **Markers:** the block is delimited by `<!-- fercho-base:start -->` and `<!-- fercho-base:end -->`. If the target file already contains these markers, replace everything between them. Never write the block twice and never modify content outside the markers.
4. **Block order:** the base block is the first block of the file, followed by `fercho-be`, `fercho-fe`, `fercho-sdd`.
5. **Hand edits:** rules a developer adds later go outside the markers, so a re-run does not remove them.

Block to write (replace every `{{...}}` placeholder with the detected content, and remove every section that Step 3 marked as omitted):

```markdown
<!-- fercho-base:start -->
# {{PROJECT_NAME}} – Project Rules

This file provides guidance to AI assistants when working with code in this repository.

> The mandatory sections below are a **proactive** contract: new code must satisfy them the moment it is written, not only at review time. Each rule links to the detailed section that defines it; the detail is the source of truth. Never generate code that violates them.

> **Enforcement:** {{ENFORCEMENT_LAYERS}}

## Mandatory Backend Rules ({{BACKEND_STACK}})

{{MANDATORY_BACKEND_RULES}}

## Mandatory Frontend Rules ({{FRONTEND_STACK}})

{{MANDATORY_FRONTEND_RULES}}

## Git & Ticket Conventions

- **Branch naming:** {{BRANCH_PATTERN}}
- **Commit message format:** {{COMMIT_FORMAT}}
- **Enforcement:** {{GIT_ENFORCEMENT}} Keep this section in sync with those files; they are the actual enforcement.

## Architecture

{{REQUEST_FLOW_DIAGRAM}}

{{PROJECT_LIST}}

## Read First

Read the doc for an area before changing that area.

| Area | Read first | Rules to preserve |
|---|---|---|
{{READ_FIRST_ROWS}}

## Running the Project

{{RUN_AND_SETUP_COMMANDS}}

## Common Commands

{{COMMON_COMMANDS}}

## Backend Standards

{{BACKEND_LANGUAGE_STANDARDS}}
- **Docs and comments:** describe what the code does now and its contract. Do not narrate change history, the bug or incident that prompted the code, or the alternative not taken. A reader seeing the code for the first time should not need the backstory.

### Error Responses

{{ERROR_RESPONSE_RULES}}

## Backend Testing Standards

{{BACKEND_TESTING_RULES}}

## Security Standards

Apply these when writing or changing code, not only at review time. New code that handles authentication, user input, resources, or external boundaries must be secure by default.

- **Object-level authorization:** authenticating the caller is not enough. Every endpoint that reads or changes a resource verifies that the caller owns it. {{OWNERSHIP_CHECK}}
- **Input and path safety:** never use a client-supplied value (file name, id, slug) directly in a storage key, file path, or query without validating it. Prefer allowlists over blocklists, and never trust a client-declared content type as the only gate. {{INPUT_VALIDATION}}
- **Cryptography and secrets:** pin token validation to the expected algorithm. Load secrets from the project's configuration and fail at startup when one is missing; never ship a hard-coded fallback secret, and never commit a real secret. {{SECRETS_HANDLING}}
- **Fail closed:** guards, filters, and validators reject when they are misconfigured; they never skip silently. Default to deny. {{FAIL_CLOSED_MECHANISMS}}
- **Output and link safety:** when rendering user-influenced or rich content, allow only safe URL schemes and never inject unsanitized HTML. {{OUTPUT_SANITIZATION}}
- **Logging:** never log secrets, tokens, cookies, passwords, or personal data. Log the event, not the credential. {{LOG_REDACTION}}
- **Trust boundary:** {{TRUST_BOUNDARY}} Do not add endpoints or paths that bypass it or introduce new implicit trust.
- **Framing:** pages that may be embedded use an allowlist of origins taken from configuration per environment, never a hard-coded or wildcard origin. {{FRAMING_CONFIG}}
- **Deferred changes:** when a change waits for an external confirmation, leave the code secure by default and document the follow-up; never ship the insecure interim state.

## Frontend Standards

{{FRONTEND_STANDARDS}}
<!-- fercho-base:end -->
```
