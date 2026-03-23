# CGS Team Claude Setup

A complete AI-assisted development workflow powered by [Claude Code](https://claude.com/claude-code), [Spec Kit](https://github.com/lsendel/spec-kit-mcp), and the Model Context Protocol (MCP).

This repo provides the full infrastructure for spec-driven development: from writing feature specs, through planning and implementation, to visual validation against Figma designs.

---

## Table of Contents

- [Quick Start](#quick-start)
- [MCP Servers](#mcp-servers)
- [Agents](#agents)
- [Skills / Commands](#skills--commands)
  - [Setup & Bootstrap](#setup--bootstrap)
  - [Specification Workflow](#specification-workflow)
  - [BA Workflow](#ba-workflow)
  - [QA Workflow](#qa-workflow)
  - [Figma Workflow](#figma-workflow)
  - [Code Review](#code-review)
- [Workflow Overview](#workflow-overview)
- [Real-World Examples](#real-world-examples)
  - [Example 1: Full Feature Lifecycle — Spec to Release](#example-1-full-feature-lifecycle--spec-to-release)
  - [Example 2: Setting Up Spec Kit on an Existing Project](#example-2-setting-up-spec-kit-on-an-existing-project)
  - [Example 3: Converting Existing Docs and Code into Specs](#example-3-converting-existing-docs-and-code-into-specs)
  - [Example 4: Capturing App Screens into Figma](#example-4-capturing-app-screens-into-figma)
- [Project Structure](#project-structure)

---

## Quick Start

1. **Install Claude Code** — follow the [official guide](https://docs.anthropic.com/en/docs/claude-code)
2. **Clone this repo** and open it in your terminal
3. **Start Claude Code** — MCP servers defined in `.mcp.json` will be loaded automatically

```bash
git clone <repo-url>
cd cgs-team-claude-setup
claude
```

All skills, agents, and MCP servers are configured at the project level and available immediately.

---

## MCP Servers

Configured in `.mcp.json`. These extend Claude Code with external tool capabilities.

| Server | Package | Purpose |
|--------|---------|---------|
| **spec-kit** | `@lsendel/spec-kit-mcp` | Spec Kit workflow — templates, scripts, and feature scaffolding |
| **playwright** | `@anthropic-ai/mcp-server-playwright` | Browser automation — navigate pages, fill forms, take screenshots, run accessibility checks |
| **figma** | `@anthropic-ai/mcp-server-figma` | Figma integration — read designs, extract metadata, manage Code Connect mappings |
| **chrome-devtools** | `@anthropic-ai/mcp-server-chrome-devtools` | Chrome DevTools — inspect network requests, console errors, evaluate JS, performance tracing |

### Usage

MCP tools are available to Claude Code and agents automatically. You don't invoke them directly — they're used behind the scenes when agents need to interact with browsers, Figma, or the spec kit system.

To verify servers are loaded, check the status bar in Claude Code or run `/mcp` in the CLI.

---

## Agents

Located in `.claude/agents/`. These are specialized AI personas that Claude Code can delegate work to.

### Backend Developer

| | |
|---|---|
| **File** | `.claude/agents/backend-developer.md` |
| **Focus** | NestJS API development — controllers, services, Prisma schema/migrations, DTOs, guards, interceptors |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, Task + Chrome DevTools MCP |
| **Modes** | **tasks.md-driven** (processes backend tasks in order) or **ad-hoc** (direct instructions) |

Invoke directly:

```
@backend-developer add the projects CRUD endpoints
```

Or let `/speckit.implement` delegate backend tasks to it automatically.

### Web Developer

| | |
|---|---|
| **File** | `.claude/agents/web-developer.md` |
| **Focus** | Frontend UI — implement pages and components from Figma designs, write unit tests, validate visually |
| **Tools** | Read, Write, Edit, Bash, Grep, Glob, Task + Figma MCP + Playwright MCP |
| **Modes** | **tasks.md-driven** (processes frontend tasks in order) or **ad-hoc** (direct instructions) |

Invoke directly:

```
@web-developer implement the users page from this Figma link: https://figma.com/design/...
```

---

## Skills / Commands

Located in `.claude/commands/` and `.claude/skills/`. Invoke them by typing the command name in Claude Code (e.g., `/speckit.specify`).

### Setup & Bootstrap

Skills for bootstrapping new projects with the CGS toolchain. Located in `.claude/skills/`. These are also installed at the **user level** (`~/.claude/skills/`) so they work from any project.

#### `/cgs.claude.setup` — Bootstrap Project with CGS Toolchain

Copies all CGS skills, speckit commands, agents, MCP config, and constitution from the latest version of this repo into the current project. Asks whether the project uses the same tech stack as the CGS default — if not, delegates to `/cgs.claude.constitution` to generate a tailored constitution.

```
/cgs.claude.setup
```

**What it copies:**
- All `cgs.*` skills (BA, QA, Figma, review, setup, constitution)
- All `speckit.*` commands (specify, clarify, plan, tasks, implement, analyze, checklist, constitution, taskstoissues)
- Agent definitions (backend-developer, web-developer)
- MCP config (`.mcp.json`), skills lock, settings
- Speckit templates and scripts (`.specify/`)

**Tech stack gate (mandatory):** The skill asks whether the project uses the CGS default stack (NestJS/React/React Native/Turborepo). If yes, the CGS constitution is copied as-is. If no, the user must describe their stack, and `/cgs.claude.constitution` generates a tailored constitution.

#### `/cgs.claude.constitution` — Generate Tailored Constitution

Generates a project constitution adapted to the project's specific tech stack while preserving all stack-agnostic code quality principles from the CGS reference constitution. Use this when the project uses a different stack than the CGS default.

```
/cgs.claude.constitution Frontend: Next.js + TypeScript + Tailwind, no backend, Vercel deployment
/cgs.claude.constitution Backend: Django + Python, Frontend: Vue 3 + Pinia, DB: PostgreSQL
```

**Preserved principles** (adapted to your stack, never removed):
Clean Architecture & SOLID, Modular Architecture, Strict Type Safety, Security by Design, Testing Discipline (80%+ coverage), Independent Deployability, Observability-First, Shared-Before-Custom, Code Quality Standards, Naming Conventions, Error Handling, Design Token Management, PR & Review Workflow, Governance.

**Stack-specific sections** are adapted: technology stack tables, folder structures, behavioral contracts, tool references, and package layouts are all rewritten to match your actual tech stack. Sections that don't apply (e.g., mobile if you have none) are removed.

### Specification Workflow

These skills follow a sequential pipeline. Each builds on the output of the previous step.

#### `/speckit.specify` — Create Feature Spec

Creates a new feature branch and generates `spec.md` from a natural language description.

```
/speckit.specify User authentication with email/password and OAuth
```

**Produces:** `spec.md`, `checklists/requirements.md`, new git branch

#### `/speckit.clarify` — Resolve Ambiguities

Identifies underspecified areas in the current spec and asks up to 5 targeted questions. Answers are encoded back into `spec.md`.

```
/speckit.clarify
```

#### `/speckit.plan` — Create Implementation Plan

Generates the technical architecture from the spec: data models, API contracts, and dev environment setup.

```
/speckit.plan
```

**Produces:** `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

#### `/speckit.tasks` — Generate Task Breakdown

Creates a dependency-ordered, actionable task list from plan and spec artifacts.

```
/speckit.tasks
```

**Produces:** `tasks.md` with phased tasks, dependency graph, and parallel execution markers

#### `/speckit.analyze` — Cross-Artifact Analysis

Read-only consistency check across `spec.md`, `plan.md`, and `tasks.md`. Finds gaps, duplications, ambiguities, and constitution violations.

```
/speckit.analyze
```

#### `/speckit.implement` — Execute Tasks

Processes all tasks from `tasks.md` in order, delegating to backend and web developer agents as appropriate. Uses TDD — tests are written before implementation.

```
/speckit.implement
```

#### `/cgs.git.commit` — Logical Git Commits

Creates logical, feature-grouped git commits from uncommitted changes. Runs lint/build pre-flight checks, groups changes by User Story, updates documentation incrementally per commit, and enforces a docs-first staging rule.

```
/cgs.git.commit
```

#### `/speckit.checklist` — Generate Quality Checklist

Creates requirement quality checklists that validate whether specs are complete, clear, and testable (not implementation checklists).

```
/speckit.checklist
```

#### `/speckit.constitution` — Project Governance

Creates or updates the project constitution — the non-negotiable rules governing architecture, tech stack, code quality, security, and testing.

```
/speckit.constitution
```

**Produces:** `.specify/memory/constitution.md`

#### `/speckit.taskstoissues` — Export Tasks to GitHub Issues

Converts tasks from `tasks.md` into dependency-ordered GitHub issues.

```
/speckit.taskstoissues
```

### BA Workflow

Business analysis skills for scope and requirements validation. Located in `.claude/skills/`.

#### `/cgs.ba.inscope` — Validate Requirement Scope

Analyzes whether new requirements fit within the project's defined boundaries by evaluating them against all available Speckit artifacts (constitution, existing specs, plans, tasks).

```
/cgs.ba.inscope Add a user notifications feature with a notification bell and notifications page
/cgs.ba.inscope path/to/requirements.md
/cgs.ba.inscope We need a Python ML microservice with MongoDB
```

**Input:** Plain text requirements, a file path, or multiple requirements separated by line breaks

**Produces:** A structured Scope Analysis Report with:
- **Overall verdict**: IN SCOPE / PARTIALLY IN SCOPE / OUT OF SCOPE
- **Confidence level**: High / Medium / Low
- **Dimension breakdown table**: Architecture Alignment, Tech Stack Compliance, Business Domain Fit, Principle Compliance, Feasibility
- **Detailed findings** with evidence from specific Speckit artifacts
- **Recommendations**: next steps for in-scope items, adjustment suggestions for partial, alternatives for out-of-scope

**What it checks against:**
1. **Constitution** (`.specify/memory/constitution.md`) — architecture, tech stack, principles, naming conventions
2. **Existing feature specs** — business domains already covered
3. **Implementation plans** — architectural decisions, API contracts, data models
4. **Task breakdowns** — implementation patterns and work granularity
5. **Project structure** — existing modules, packages, and code organization

**Key behaviors:**
- Read-only — never modifies project files
- Evidence-based — every finding cites a specific Speckit artifact
- Charitable interpretation — flags ambiguity rather than assuming the worst
- Project-agnostic — works with any project that uses Speckit

### QA Workflow

These skills handle test generation, visual analysis, and QA reporting. Located in `.claude/skills/`.

They use internal Playwright guide skills (`_guides.playwright-core`, `_guides.playwright-pom`, `_guides.playwright-cli`) for best practices — you don't invoke the guides directly.

#### `/cgs.qa.e2e` — Generate E2E Tests from Spec

Generates Playwright E2E test suites from speckit `spec.md` acceptance scenarios. Each user story gets its own test file with tests derived from Given/When/Then scenarios.

```
/cgs.qa.e2e
/cgs.qa.e2e US1,US2
/cgs.qa.e2e --update
```

**Options:** `--update` (overwrite existing test files), `--dry-run` (plan only), story filter

**Produces:** `tests/e2e/<feature>/<story-slug>.spec.ts`, shared helpers/fixtures/page objects as needed

**What it does:**
1. Reads `spec.md` user stories and acceptance scenarios
2. Reads `plan.md` for technical context (routes, auth, data model)
3. Discovers existing test infrastructure (helpers, fixtures, config)
4. Decides test pattern (POM vs fixtures vs helpers) based on feature complexity
5. Outputs a test plan for approval, then generates test files

#### `/cgs.qa.visual` — Visual Mismatch Investigation

Deep LLM-powered analysis of screens where the rendered UI diverges from the Figma design. While `/cgs.figma.visual` tells you **how much** differs (pixel percentage), this skill tells you **what exactly** differs.

```
/cgs.qa.visual
/cgs.qa.visual "Dashboard"
/cgs.qa.visual --threshold=0.05 --save-report
```

**Options:** `--threshold=0.02` (override diff threshold), `--save-report` (save to `specs/<feature>/visual-analysis.md`), screen filter

**Requires:** `FIGMA_TOKEN` environment variable, prior run of `/cgs.figma.visual`

**What it does:**
1. Reads pixel diff results from `/cgs.figma.visual`
2. For screens exceeding the threshold, captures three images: rendered UI, Figma export, diff overlay
3. Uses LLM vision to systematically compare: layout, spacing, typography, colors, components
4. Reports each finding with severity (Critical/Major/Minor/Cosmetic), expected vs actual values, and likely cause

#### `/cgs.qa.report` — Full QA Report

Runs a complete QA pass — E2E tests + visual comparison + visual investigation — and produces a single timestamped markdown report.

```
/cgs.qa.report
/cgs.qa.report --skip-visual
/cgs.qa.report --story-filter=US1
```

**Options:** `--skip-visual` (E2E only), `--story-filter=US1,US2` (specific stories), `--threshold=0.02`

**Produces:** `reports/<feature>-qa-<YYYY-MM-DD_HH-mm>.md`

**Visual tests are optional** — if `figma-visual.spec.ts` doesn't exist or `FIGMA_TOKEN` isn't set, the report gracefully skips visual testing and notes it.

**Report includes:**
- Executive summary with release recommendation (READY / CONDITIONAL / NOT READY / BLOCKED)
- Test environment details
- Per-story E2E results with failed test details and suggested fixes
- Visual comparison results with LLM-analyzed findings
- Coverage analysis against `spec.md` acceptance scenarios
- Risk assessment and prioritized recommendations

**The Figma cache is cleared before visual tests** so designs are re-fetched, catching any Figma changes since the last run.

### Figma Workflow

These skills handle the Figma design integration pipeline. Located in `.claude/skills/`.

#### `/cgs.figma.capture` — Capture Screens to Figma

Captures the app's screen states from e2e tests and pushes them into a Figma file as design frames.

```
/cgs.figma.capture https://figma.com/design/abc123/file-name
/cgs.figma.capture https://figma.com/design/abc123/file-name --new
/cgs.figma.capture https://figma.com/design/abc123/file-name "Add Task Modal" --viewport=D,M
```

**Options:** `--new` (only new screens), `--viewport=D,M,T` (filter viewports), screen filter (substring match)

**Produces:** `tests/e2e/<feature>/capture-screens.spec.ts`, Figma frames

#### `/cgs.figma.link` — Link Figma to Spec

Generates the `## Screens` section in `spec.md` and auto-matches Figma frames to each screen by name similarity.

```
/cgs.figma.link https://figma.com/design/abc123/file-name
/cgs.figma.link https://figma.com/design/abc123/file-name US1,US3
/cgs.figma.link https://figma.com/design/abc123/file-name --force
```

**Options:** `--force` (overwrite existing links), story filter

**Produces:** Updated `spec.md` with `## Screens` table and `**Design**:` links on User Stories

#### `/cgs.figma.visual` — Visual Comparison

Runs pixelmatch-based visual comparison of rendered UI against Figma designs. Each screen with a Figma Node value gets a pixel-level diff test.

```
/cgs.figma.visual
/cgs.figma.visual US1
/cgs.figma.visual --threshold=0.05
```

**Options:** `--threshold=0.05` (override default 2% diff threshold), screen/story filter

**Produces:** `tests/e2e/<feature>/figma-visual.spec.ts`, `tests/e2e/helpers/figma-compare.ts`

**Pipeline:**
```
spec.md → e2e tests → /cgs.figma.capture → /cgs.figma.link → /cgs.figma.visual
```

### Code Review

#### `/cgs.review` — Interactive Code Review & Fix

Multi-stage interactive code review that analyzes your changes, presents issues one by one, and offers auto-fix for each.

```
/cgs.review
/cgs.review src/features/auth/
```

**Stages:**
1. **Scope** — Detects uncommitted changes, last commit, or branch diff and lets you choose what to review
2. **Review** — Launches a subagent to find bugs, security issues, performance problems, race conditions, and edge cases
3. **Resolution** — Walks through each issue with options: auto-fix, fix yourself, reject (false positive), or skip
4. **Wrap up** — Summary and optional re-review of new changes

**Issue severities:** Critical, Warning, Suggestion

---

## Workflow Overview

### Full Feature Lifecycle (Spec → Ship)

This is the recommended end-to-end workflow for every feature you release. Each stage produces a commit, creating a clean audit trail.

```
# 1. SPECIFY — define what to build
/speckit.specify "feature description"     → spec.md + branch          → commit
/speckit.clarify                           → refined spec.md           → commit
/speckit.plan                              → plan.md + contracts       → commit
/speckit.tasks                             → tasks.md                  → commit
/speckit.analyze                           → consistency check

# 2. IMPLEMENT — build it (parallel agents)
/speckit.implement                         → working code + tests      → commit

# 3. TEST — generate E2E coverage
/cgs.qa.e2e                                → Playwright test suites    → commit

# 4. DESIGN SYNC — capture screens to Figma
/cgs.figma.capture <figma-url>             → app screens in Figma      → commit

# 5. LINK — connect Figma frames to spec
/cgs.figma.link <figma-url>                → spec.md with design links → commit

# 6. VISUAL — compare implementation vs design
/cgs.figma.visual                          → pixel-level diff tests    → commit

# 7. QA REPORT — full QA pass with release recommendation
/cgs.qa.report                             → timestamped QA report     → commit
```

### Standard Feature Development

```
 /speckit.specify "feature description"     → spec.md + branch
 /speckit.clarify                           → refined spec.md
 /speckit.plan                              → plan.md + contracts + data model
 /speckit.tasks                             → tasks.md
 /speckit.implement                         → working code + tests
 /cgs.git.commit                            → logical feature-grouped commits
 /qa.fullpass                               → test results + bug reports
```

### Design-First Development

```
/speckit.specify "feature description"                         → spec.md
/cgs.figma.link <figma-url>                        → spec.md with Figma screen links
/speckit.plan                                                  → plan.md
/speckit.tasks                                                 → tasks.md
@web-developer implement from Figma                            → UI code
/cgs.figma.visual                                  → pixel-level design comparison
```

### Figma Capture & Validation Pipeline

```
/cgs.figma.capture <figma-url>                     → capture app screens to Figma
/cgs.figma.link <figma-url>                        → link Figma frames to spec
/cgs.figma.visual                                  → pixel-level comparison report
```

---

## Real-World Examples

### Example 1: Full Feature Lifecycle — Spec to Release

The complete workflow for shipping a feature from idea to QA-verified release. This is the standard process for every feature in your project.

**Scenario:** You're building a task management feature for a project management app. The designer has prepared Figma mockups. You need to specify, implement, test, validate against designs, and produce a QA report.

---

**Stage 1 — Specify: Define what to build**

Start by generating the feature specification from a natural language description:

```
/speckit.specify Task management — users can create, edit, delete, and
reorder tasks within projects. Supports due dates, assignees, priority
levels, and status transitions (todo → in progress → done). Includes
kanban board and list views.
```

Claude creates a feature branch (`feature/task-management`), generates `spec.md` with user stories, acceptance scenarios, edge cases, and functional requirements.

```
/commit
```

Clarify any ambiguities — Claude asks up to 5 targeted questions and encodes answers back into the spec:

```
/speckit.clarify
```

> **Claude:** "Should task reordering be drag-and-drop only, or also support keyboard shortcuts for accessibility?"
> **You:** "Both — drag-and-drop with keyboard arrow keys as alternative"

```
/commit
```

Generate the technical plan — architecture, data model, API contracts, and dev environment setup:

```
/speckit.plan
/commit
```

Break the plan into dependency-ordered tasks ready for implementation:

```
/speckit.tasks
/commit
```

Run cross-artifact analysis to catch gaps before implementation:

```
/speckit.analyze
```

This is read-only — it flags issues like "US3 acceptance scenario #2 references a `priority` field not in the data model" so you can fix them before writing code.

---

**Stage 2 — Implement: Build it with parallel agents**

Execute the task list. Claude delegates work to specialized agents that can run in parallel:

```
/speckit.implement
```

The orchestrator reads `tasks.md` and dispatches work:

- **`@backend-developer`** — NestJS controllers, services, Prisma schema, DTOs, guards
- **`@web-developer`** — React components, pages, hooks, unit tests, Figma-matched UI

Backend and frontend tasks on independent branches of the dependency graph run in parallel. Tasks with dependencies wait for their prerequisites to complete.

Once implementation is done:

```
/commit
```

---

**Stage 3 — Test: Generate E2E coverage from the spec**

Generate Playwright E2E tests directly from the acceptance scenarios in `spec.md`:

```
/cgs.qa.e2e
```

Claude reads your spec's Given/When/Then scenarios and produces one test file per user story:

```
tests/e2e/task-management/
├── task-crud.spec.ts          # US1: Create, edit, delete tasks
├── task-reordering.spec.ts    # US2: Drag-and-drop + keyboard reorder
├── kanban-board.spec.ts       # US3: Kanban view with status transitions
└── list-view.spec.ts          # US4: List view with sorting and filtering
```

Tests follow best practices from the internal Playwright guide skills — role-based locators, web-first assertions, no `waitForTimeout`, proper test isolation.

```
/commit
```

---

**Stage 4 — Design sync: Capture app screens to Figma**

Now that the implementation is running, capture every screen state into your Figma file so the design team can see what was built:

```
/cgs.figma.capture https://figma.com/design/abc123/TaskApp-Designs
```

Claude discovers screens from your e2e tests, generates a capture script, and pushes frames to Figma at multiple viewports (Desktop, Tablet, Mobile).

```
/commit
```

---

**Stage 5 — Link: Connect Figma frames to the spec**

Link the Figma frames back to your `spec.md` so each user story has a direct reference to its design:

```
/cgs.figma.link https://figma.com/design/abc123/TaskApp-Designs
```

Claude auto-matches Figma frame names to screens in the spec and updates `spec.md` with the `## Screens` table and `**Design**:` links on each User Story.

```
/commit
```

---

**Stage 6 — Visual: Compare implementation accuracy against design**

Run pixel-level comparison to check how closely the implementation matches the Figma designs:

```
/cgs.figma.visual
```

This generates `figma-visual.spec.ts` which uses pixelmatch to diff each screen against its Figma frame. Output shows per-screen diff percentages:

```
✓ Empty State — Desktop: 0.3% diff (threshold: 2%) — PASS
✗ Kanban Board — Desktop: 4.2% diff (threshold: 2%) — FAIL
✗ Kanban Board — Mobile: 8.1% diff (threshold: 2%) — FAIL
✓ Task Detail — Desktop: 1.1% diff (threshold: 2%) — PASS
```

```
/commit
```

---

**Stage 7 — QA Report: Full QA pass with release recommendation**

Run the complete QA pass — this executes E2E tests, re-runs visual comparison (with fresh Figma cache), performs LLM visual investigation on any mismatches, and compiles everything into a single report:

```
/cgs.qa.report
```

The report is saved to `reports/task-management-qa-2026-03-12_15-30.md` and includes:

- **Executive summary** — overall PASS/FAIL/CONDITIONAL status
- **Release recommendation** — READY, CONDITIONAL, NOT READY, or BLOCKED
- **E2E results** — per-story pass/fail with error details and suggested fixes
- **Visual comparison** — per-screen diff results
- **Visual analysis** — for failed screens, LLM-identified findings with severity, expected vs actual, and likely cause
- **Coverage analysis** — which acceptance scenarios have test coverage
- **Risk assessment** — flagged risks with impact and mitigation
- **Recommendations** — prioritized list of actions

```
/commit
```

---

**What to do with the report:**

| Recommendation | Meaning | Action |
|----------------|---------|--------|
| **READY** | All tests pass, visuals match | Merge the PR and ship |
| **CONDITIONAL** | E2E pass, but visual discrepancies exist | Review visual findings with designer — may be intentional |
| **NOT READY** | E2E failures or critical visual issues | Fix the issues, then re-run `/cgs.qa.report` |
| **BLOCKED** | Tests can't execute | Fix environment or config issue first |

---

**Repeat for every feature.** Each feature gets its own branch, spec directory, and QA report. The full lifecycle creates a clean audit trail:

```
feature/task-management
├── specs/task-management/
│   ├── spec.md              # What to build
│   ├── plan.md              # How to build it
│   ├── tasks.md             # Work breakdown
│   └── data-model.md        # Schema
├── tests/e2e/task-management/
│   ├── task-crud.spec.ts    # E2E tests
│   ├── kanban-board.spec.ts
│   └── figma-visual.spec.ts # Visual comparison
└── reports/
    └── task-management-qa-2026-03-12_15-30.md  # QA report
```

---

### Example 2: Setting Up Spec Kit on an Existing Project

You have a project with code but no Spec Kit infrastructure. Here's how to bootstrap it.

**Step 1 — Bootstrap with `/cgs.claude.setup`**

The fastest way to set up a project. Start Claude Code in your project directory and run:

```
/cgs.claude.setup
```

This fetches the latest CGS toolchain from GitHub and copies all skills, commands, agents, MCP config, and speckit templates into your project. It then asks about your tech stack:

- **Same stack as CGS default** (NestJS/React/React Native/Turborepo) → copies the constitution as-is
- **Different stack** → asks you to describe your stack, then generates a tailored constitution via `/cgs.claude.constitution`

**Alternative: Manual setup**

If you prefer manual control:

```bash
# From your existing project root
cp -r /path/to/cgs-team-claude-setup/.mcp.json .
cp -r /path/to/cgs-team-claude-setup/.claude .
cp -r /path/to/cgs-team-claude-setup/.specify .
```

Then create the constitution:

```
/speckit.constitution
```

The constitution is written to `.specify/memory/constitution.md`. All future specs, plans, and implementations will follow these rules.

**Step 3 — Create specs for existing and new features**

Since this is an existing project, start by asking Claude to analyze what's already built. Point it at the source code and any existing documentation:

```
Analyze the current codebase and docs/architecture.md to identify
the main feature areas already implemented in this project.
List each feature with a short summary of what it does.
```

Claude will scan the codebase structure, routes, modules, and docs to produce a feature inventory. Use this as your starting point.

Then create specs for each feature area, referencing the existing code and docs so the spec captures what's already built:

```
/speckit.specify User authentication — already implemented in src/modules/auth/.
See docs/authentication.md for the original PRD.
Supports email/password, Google OAuth, and magic links.
```

Claude reads the referenced files, analyzes the existing implementation, and generates a `spec.md` that reflects the current state — not a greenfield design.

```
/speckit.clarify
```

Resolve any ambiguities — Claude asks up to 5 targeted questions and encodes the answers back into the spec.

```
/speckit.plan
```

Generates the technical plan: architecture, data model, API contracts, and dev environment setup — all aligned with your constitution.

```
/speckit.tasks
```

Breaks the plan into dependency-ordered, actionable tasks ready for implementation.

**Step 4 — Implement**

```
/speckit.implement
```

Claude processes tasks in order, delegating to the backend-developer and web-developer agents. Tests are written before code. Completed tasks are checked off in `tasks.md`.

**Step 5 (optional) — Capture existing screens into Figma**

If you have e2e tests, capture your app's screens into Figma. This gives your design team a baseline to work from for future redesigns:

```
/cgs.figma.capture https://figma.com/design/abc123/MyApp-Current-UI
```

Then link the Figma file to your specs:

```
/cgs.figma.link https://figma.com/design/abc123/MyApp-Current-UI
```

Now your design team can duplicate the Figma file, create redesigned versions, and developers can use `/cgs.figma.visual` to validate implementations against the new designs.

**What you get:**
- MCP servers (Playwright, Figma, Chrome DevTools) for browser automation and design integration
- 2 specialized agents (backend and web developer) that understand your project's conventions
- 20+ skills covering the full specify → plan → implement → test → Figma validation → code review lifecycle
- A constitution (default or tailored to your stack) that enforces consistency across all future work
- (Optional) Figma baseline for your design team to iterate on

---

### Example 3: Converting Existing Docs and Code into Specs

You have a running project with existing documentation (PRDs, READMEs, wiki pages, Confluence docs) and working source code, but no formal specs. Here's how to retroactively create specs for each feature.

**Step 1 — Set up constitution first (if not done)**

```
/speckit.constitution
```

This ensures the generated specs follow your project's actual conventions.

**Step 2 — Create a spec from existing documentation**

Feed your existing docs directly to the specify command. Claude will read them, extract requirements, and produce a structured spec:

```
/speckit.specify Based on our existing user auth system:
- See docs/authentication.md for the original PRD
- The implementation is in src/modules/auth/
- We support email/password, Google OAuth, and magic links
- JWT tokens with refresh rotation
- Rate limiting on login attempts
```

Claude will read the referenced files, analyze the existing implementation, and generate a `spec.md` that captures what's already built — not what needs to be built.

**Step 3 — Clarify gaps between docs and implementation**

Documentation often drifts from reality. Use clarify to surface the differences:

```
/speckit.clarify
```

Claude will ask targeted questions like:
> "The docs mention 2FA support but the codebase has no 2FA implementation. Should the spec reflect current state (no 2FA) or intended state (with 2FA)?"

Answer based on what you want the spec to represent.

**Step 4 — Generate the technical plan from existing code**

```
/speckit.plan
```

Since the code already exists, the plan will document the current architecture rather than propose a new one: data models, API contracts, project structure, and dev environment setup.

**Step 5 — Generate tasks for remaining work**

```
/speckit.tasks
```

If the spec captures only what exists, the task list will be minimal (mostly test coverage). If you included planned features, tasks will cover the delta between current state and target state.

**Step 6 — Validate with analysis**

```
/speckit.analyze
```

This cross-checks `spec.md`, `plan.md`, and `tasks.md` for consistency. It catches things like:
- Requirements in the spec with no corresponding tasks
- Tasks that reference contracts not defined in the plan
- Terminology drift between documents

**Step 7 (optional) — Link Figma designs to specs**

If you have Figma designs for this feature, link them to the spec:

```
/cgs.figma.link https://figma.com/design/abc123/MyDesign
```

Claude reads the Figma file, finds all frames, and auto-matches them to screens in your `spec.md`. Each User Story gets a `**Design**:` link to the relevant Figma frame. This enables `/cgs.figma.visual` pixel-level comparisons later.

**Step 8 — Implement remaining work**

With specs, plans, tasks, and (optionally) Figma links in place, execute the implementation:

```
/speckit.implement
```

Claude processes tasks from `tasks.md` in dependency order — delegating backend work to the backend-developer agent and frontend work to the web-developer agent. Tests are written before implementation (TDD). Each completed task is checked off in `tasks.md`.

If Figma screens are linked, the web-developer agent will pull design context directly from Figma to match the implementation to the designs.

**Repeat for each major feature area.** A typical project might produce specs for: authentication, user management, billing, notifications, etc. — one spec per feature branch.

**Pro tip:** For large codebases, work feature-by-feature rather than trying to spec the entire app at once. Each `/speckit.specify` creates its own branch and isolated spec directory.

---

### Example 4: Capturing App Screens into Figma

You have a running application with e2e tests but no Figma designs. You want to capture every screen into Figma so you can use the visual validation workflow and link designs to specs.

**Prerequisites:**
- E2e tests must exist for the feature (in `tests/e2e/<feature>/`)
- You need a Figma account with edit access
- Playwright must be configured with a `webServer` entry (no manual dev server needed)

**Step 1 — Capture screens into Figma**

The capture skill discovers screens from your e2e tests, generates a Playwright capture script, and pushes frames to Figma:

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens
```

Claude will:
1. Analyze your e2e test files to discover all screen states
2. Determine viewports per screen type (Desktop/Tablet/Mobile for fundamental states, Desktop/Mobile for modals)
3. Check which screens already exist in the Figma file
4. Present a capture plan for your approval
5. Generate `capture-screens.spec.ts` and run it

**Step 2 — Capture only new screens**

After adding new e2e tests, capture only the screens not yet in Figma:

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens --new
```

**Step 3 — Capture specific screens or viewports**

```
/cgs.figma.capture https://figma.com/design/abc123/My-App-Screens "Add Task Modal" --viewport=D,M
```

**Step 4 — Link Figma frames to specs**

```
/cgs.figma.link https://figma.com/design/abc123/My-App-Screens
```

Claude reads the Figma file structure, auto-matches frames to screens by name similarity, and updates `spec.md` with the `## Screens` table and `**Design**:` links on each User Story.

**Step 5 — Run visual comparison**

With screens linked, run pixel-level comparison anytime:

```
/cgs.figma.visual
```

This generates a Playwright test that compares live rendered pages against Figma frames using pixelmatch with a 2% default threshold. As the app evolves, re-capture updated screens and re-run to catch visual regressions.

**Full pipeline in action:**

```
# 1. Capture all screens into Figma from e2e tests
/cgs.figma.capture https://figma.com/design/abc123/...

# 2. Link Figma frames to spec
/cgs.figma.link https://figma.com/design/abc123/...

# 3. Pixel-level visual comparison
/cgs.figma.visual

# 4. After code changes, re-capture and re-validate
/cgs.figma.capture https://figma.com/design/abc123/... "Dashboard"
/cgs.figma.visual
```

---

## Project Structure

```
.
├── .mcp.json                          # MCP server configuration
├── .claude/
│   ├── agents/
│   │   ├── backend-developer.md       # Backend agent definition
│   │   └── web-developer.md           # Frontend agent definition
│   ├── commands/
│   │   ├── speckit.specify.md         # Feature spec generation
│   │   ├── speckit.clarify.md         # Spec clarification
│   │   ├── speckit.plan.md            # Implementation planning
│   │   ├── speckit.tasks.md           # Task breakdown
│   │   ├── speckit.analyze.md         # Cross-artifact analysis
│   │   ├── speckit.implement.md       # Task execution
│   │   ├── cgs.git.commit.md          # Logical git commits
│   │   ├── speckit.checklist.md       # Quality checklists
│   │   ├── speckit.constitution.md    # Project governance
│   │   └── speckit.taskstoissues.md   # GitHub issue export
│   ├── skills/
│   │   ├── _guides.playwright-cli/       # Playwright CLI guide (internal)
│   │   ├── _guides.playwright-core/      # Playwright testing guide (internal)
│   │   ├── _guides.playwright-pom/       # Page Object Model guide (internal)
│   │   ├── cgs.ba.inscope/              # Validate requirement scope
│   │   ├── cgs.claude.constitution/     # Generate tailored constitution
│   │   ├── cgs.claude.setup/           # Bootstrap project with CGS toolchain
│   │   ├── cgs.figma.capture/          # Capture app screens to Figma
│   │   ├── cgs.figma.link/            # Link Figma frames to spec
│   │   ├── cgs.figma.visual/          # Visual comparison against Figma
│   │   ├── cgs.qa.e2e/                # Generate E2E tests from spec
│   │   ├── cgs.qa.report/             # Full QA pass with report
│   │   ├── cgs.qa.visual/             # LLM visual mismatch analysis
│   │   └── cgs.review/                # Interactive code review & fix
│   └── settings.local.json            # Local permissions
├── .specify/
│   ├── memory/
│   │   └── constitution.md            # Project constitution
│   ├── templates/
│   │   ├── spec-template.md           # Spec scaffold
│   │   ├── plan-template.md           # Plan scaffold
│   │   ├── tasks-template.md          # Tasks scaffold
│   │   ├── checklist-template.md      # Checklist scaffold
│   │   ├── constitution-template.md   # Constitution scaffold
│   │   └── agent-file-template.md     # Agent config scaffold
│   └── scripts/bash/
│       ├── create-new-feature.sh      # Feature branch + spec init
│       ├── check-prerequisites.sh     # Feature context discovery
│       ├── setup-plan.sh              # Plan initialization
│       ├── update-agent-context.sh    # Agent context refresh
│       └── common.sh                  # Shared utilities
└── README.md
```
