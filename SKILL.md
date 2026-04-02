---
name: system-understanding-score
description: Analyzes a codebase to score how deeply the AI agent understands the system — frontend design systems, API integration, backend architecture, business logic, DevOps pipelines, and Figma design mapping. Use when the user asks how much is understood, what can be built autonomously, or what gaps remain before full-confidence delivery.
---

# System Understanding Score

## Goal

Perform a deep, evidence-based analysis of the current codebase and produce:

1. A quantified understanding score across all relevant dimensions.
2. A list of **standard tasks** the agent is confident it can complete right now.
3. A gap analysis showing what the agent does NOT understand and what is needed to close each gap.

## When to Use

Apply this skill when the user asks for:

- "How much do you understand this codebase?"
- "What can you do / build / fix right now?"
- "What's your confidence score?"
- "What are the gaps in your understanding?"
- "Can you work on this project autonomously?"
- "Score this project" or "Analyze this project"
- Any readiness or confidence assessment

## Execution Steps

You MUST follow these steps in order. Do not skip any step. Be thorough — read actual files, do not guess.

### Step 1: Project Type Detection

Scan the project root to determine the project type. Read `package.json`, config files, directory structure, and framework markers.

Classify as one of:
- **Frontend** — Client-side app (React, Vue, Svelte, Angular, Next.js pages/app dir, etc.)
- **Backend** — Server-side (Express, Fastify, NestJS, Django, Rails, Go services, etc.)
- **Full-Stack** — Both frontend and backend in the same repo
- **Monorepo** — Multiple packages/services (detect with workspaces, Turborepo, Nx, Lerna)

### Step 2: Deep Codebase Scan

For each applicable dimension (based on project type), systematically scan and read files. **You must read actual file contents** — do not score based on file names alone.

#### What to scan:
- **Config files**: package.json, tsconfig, eslint, prettier, tailwind, postcss, webpack/vite/next config
- **Entry points**: app/, pages/, src/index, main.ts, server.ts
- **Routing**: file-based routes, router configs, API route definitions
- **State management**: Redux, Zustand, Context, MobX stores
- **API layer**: GraphQL schemas/documents, REST clients, tRPC routers, API hooks
- **Components**: UI primitives, design system components, layout components
- **Styles**: Tailwind config, theme files, CSS modules, styled-components
- **Backend**: Controllers, services, models, middleware, database schemas
- **Tests**: Test files, test config, fixtures, mocks
- **DevOps**: Dockerfile, docker-compose, CI/CD configs (.github/workflows, Jenkinsfile, etc.)
- **Infrastructure**: Terraform, CDK, serverless configs, Cloudflare/Vercel configs
- **Documentation**: README, project instruction files (CLAUDE.md, .cursorrules, .windsurfrules, .github/copilot-instructions.md, .clinerules), API docs, architecture docs
- **Design system**: Component library exports, Storybook, design tokens
- **Figma integration**: Code Connect files, design token bridges, figma plugin configs

### Step 3: Score Each Dimension

Score each applicable dimension using the rubrics below. Every score MUST reference specific files, functions, or patterns found during the scan.

### Step 4: Generate Output

Produce the full output in the required format below.

---

## Scoring Dimensions

### FRONTEND DIMENSIONS (when project has frontend)

#### F1: Design System Mastery (0-25)

Can the agent change UI using the existing design system?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Component library detected but not mapped | Find UI component directory, count primitives |
| 6-10 | Components mapped, variants understood | Read component props/variants, understand composition patterns |
| 11-15 | Theme/token system understood | Read tailwind config, CSS variables, theme providers |
| 16-20 | Layout patterns and responsive system clear | Analyze grid/layout components, breakpoint usage |
| 21-25 | Full design system fluency — can extend consistently | Understand naming conventions, spacing scale, color system, can predict patterns |

**Evidence required**: List specific component files read, theme tokens found, design patterns identified.

**Auto-cap rules**:
- No component library found → cap at 5
- No theme/token system → cap at 15
- No documentation or Storybook → cap at 20

#### F2: API Integration & Middleware Readiness (0-25)

Can the agent read the backend, understand contracts, and build middleware/integration layers?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | API client exists but contracts unclear | Find HTTP/GraphQL clients, check if types exist |
| 6-10 | API contracts partially typed | Read API hooks, GraphQL documents, REST type definitions |
| 11-15 | Full request/response shapes understood | Verify codegen output, schema files, payload types |
| 16-20 | Data flow from API → state → UI is clear | Trace a complete data path from fetch to render |
| 21-25 | Can auto-generate middleware, adapters, or BFF layers | Understand transformation patterns, error handling, caching strategy |

**Evidence required**: List API files read, schemas found, data flows traced.

**Auto-cap rules**:
- No typed API contracts → cap at 10
- No observable data flow pattern → cap at 15
- Backend source not available → cap at 20

#### F3: Build & Component Health (0-25)

Can the agent detect and fix build issues and component problems?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Build toolchain identified | Read build config (next.config, vite.config, webpack, etc.) |
| 6-10 | Dependency graph partially clear | Check imports, barrel files, circular deps |
| 11-15 | TypeScript strictness and type coverage understood | Read tsconfig strict settings, check for any/unknown usage |
| 16-20 | Linting, formatting, and pre-commit pipeline clear | Read eslint, prettier, husky, lint-staged configs |
| 21-25 | Can diagnose build failures, fix type errors, resolve component issues | Understand module resolution, path aliases, env vars, SSR/CSR boundaries |

**Evidence required**: List config files read, build pipeline steps identified, known issues found.

#### F4: Figma Design Bridge (0-25)

Can the agent link Figma designs to code and maintain design fidelity?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | No Figma integration detected | Check for Code Connect, design token files, Figma plugins |
| 6-10 | Design tokens exist but not linked to Figma | Find CSS variables, theme tokens that could map to Figma |
| 11-15 | Component-to-code mapping partially possible | Match component names/variants to potential Figma components |
| 16-20 | Design system rules or Code Connect configured | Find .figma files, code connect maps, design system rule files |
| 21-25 | Full Figma-to-code bridge — can implement from Figma and push back | Code Connect active, design tokens synced, variant mapping complete |

**Evidence required**: List design token files, Code Connect configs, component mapping coverage.

**Auto-cap rules**:
- No design tokens or theme system → cap at 5
- No Code Connect or Figma plugin config → cap at 15
- No Figma integration tools (MCP, plugins, or CLI) available → cap at 20

---

### BACKEND DIMENSIONS (when project has backend)

#### B1: Architecture Understanding (0-25)

Does the agent understand the entire backend architecture?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Entry point and framework identified | Find main server file, framework detection |
| 6-10 | Route/endpoint structure mapped | Read route definitions, controller/handler structure |
| 11-15 | Database schema and ORM/query layer understood | Read models, migrations, schema files |
| 16-20 | Auth, middleware, and cross-cutting concerns clear | Trace auth flow, understand middleware chain, error handling |
| 21-25 | Full service topology understood — microservices, queues, caches, external deps | Map all service boundaries, message flows, infrastructure dependencies |

**Evidence required**: List services found, data models read, integration points identified.

**Auto-cap rules**:
- No database schema access → cap at 15
- No infrastructure config → cap at 20

#### B2: Business Logic Comprehension (0-25)

Does the agent understand WHAT the backend does, not just HOW it's built?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Core domain entities identified | Find primary models/entities and their relationships |
| 6-10 | Key business rules found in code | Read service/domain layer, identify validation rules |
| 11-15 | Data transformation and workflow logic clear | Trace data through business operations |
| 16-20 | Edge cases, guards, and business constraints understood | Find conditional logic, feature flags, role-based behavior |
| 21-25 | Full domain model mapped — can predict behavior for new scenarios | Understand the business problem deeply enough to extend logic correctly |

**Evidence required**: List domain entities, business rules found, workflows traced.

#### B3: Test Case Generation Readiness (0-25)

Can the agent write meaningful test cases from business logic understanding?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Test framework detected | Find test config (jest, vitest, pytest, etc.) |
| 6-10 | Existing test patterns understood | Read existing tests, understand assertion style |
| 11-15 | Critical paths identified for testing | Map which business flows need test coverage |
| 16-20 | Mock/fixture/seed infrastructure available | Find test utilities, factories, fixtures, database seeds |
| 21-25 | Can generate comprehensive test suites from business logic | Understand edge cases, failure modes, integration boundaries |

**Evidence required**: List test files read, test patterns found, coverage gaps identified.

**Auto-cap rules**:
- No existing tests → cap at 10
- No test utilities/fixtures → cap at 15
- Business logic unclear → cap at B2 score

#### B4: DevOps Pipeline Understanding (0-25)

Does the agent understand the deployment and infrastructure pipeline?

| Points | Criteria | What to check |
|--------|----------|---------------|
| 0-5 | Deployment target identified | Find hosting config (Dockerfile, serverless, platform configs) |
| 6-10 | CI/CD pipeline partially mapped | Read workflow files, build scripts |
| 11-15 | Environment management understood | Find env configs, secrets management, stage separation |
| 16-20 | Full pipeline from commit to production clear | Understand build → test → deploy flow, rollback strategy |
| 21-25 | Can modify pipeline and predict cross-system impact | Understand monitoring, alerting, infrastructure dependencies |

**Evidence required**: List CI/CD files read, deployment configs found, pipeline stages identified.

**Auto-cap rules**:
- No CI/CD config found → cap at 5
- No infrastructure config → cap at 15

---

### UNIVERSAL DIMENSIONS (always scored)

#### U1: Platform & Tech Stack Comprehension (0-50)

Does the agent understand the platform holistically?

| Points | Criteria |
|--------|----------|
| 0-10 | Framework and language identified, basic project structure understood |
| 11-20 | All major dependencies mapped, configuration understood |
| 21-30 | Third-party service integrations identified and understood (payments, auth, analytics, etc.) |
| 31-40 | Environment configuration, feature flags, and deployment targets clear |
| 41-50 | Full platform mental model — can predict where any change needs to happen |

#### U2: Business Logic-Code Alignment (0-50)

Is the business logic properly reflected and organized in the code?

| Points | Criteria |
|--------|----------|
| 0-10 | Can identify what the application does at a high level |
| 11-20 | Core user journeys traceable through the code |
| 21-30 | Domain model matches code organization, naming is consistent |
| 31-40 | Business rules are explicit (not buried in UI logic or scattered) |
| 41-50 | Full business domain mastery — can extend features correctly without guidance |

---

## Score Calculation

### Frontend Score (when applicable)
```
Frontend Score = F1 + F2 + F3 + F4  (out of 100)
```

### Backend Score (when applicable)
```
Backend Score = B1 + B2 + B3 + B4  (out of 100)
```

### Universal Score (always)
```
Universal Score = U1 + U2  (out of 100)
```

### Overall Understanding Score
```
If Frontend-only:  Overall = (Frontend * 0.5) + (Universal * 0.5)
If Backend-only:   Overall = (Backend * 0.5) + (Universal * 0.5)
If Full-Stack:     Overall = (Frontend * 0.3) + (Backend * 0.3) + (Universal * 0.4)
```

### Confidence Band
```
0-30%:   Low    — Major gaps, high risk of incorrect changes
31-60%:  Medium — Can handle isolated tasks, needs guidance for cross-cutting work
61-80%:  High   — Can handle most tasks autonomously, some blind spots remain
81-95%:  Very High — Near-full autonomy, minor gaps in edge cases
96-100%: Complete — Full system mastery (never claim without total evidence)
```

---

## Required Output Format

### Header
```
# System Understanding Score: [Project Name]
Project Type: [Frontend | Backend | Full-Stack | Monorepo]
Scanned: [number] files across [number] directories
```

### 1) Overall Score Card

```
Overall Understanding:       XX% — [Confidence Band]
Agentic Delivery Readiness:  XX%
```

Display a visual bar for each:
```
Understanding:  [████████████████████░░░░░] 84%
Delivery Ready: [██████████████████░░░░░░░] 72%
```

### 2) Dimension Breakdown

For EACH scored dimension, show:
```
[Dimension Name]:  XX/25
├─ Evidence: [specific files/patterns found]
├─ Strength: [what is well-understood]
└─ Gap: [what is missing or unclear]
```

### 3) Standard Tasks — Confidence Assessment

List tasks the agent CAN do right now, grouped by confidence:

#### Ready to Execute (90%+ confidence)
- [ ] Task description — *because [evidence]*

#### Can Attempt with Caveats (60-89% confidence)
- [ ] Task description — *caveat: [what's missing]*

#### Blocked or Risky (<60% confidence)
- [ ] Task description — *blocked by: [specific gap]*

**Frontend standard tasks to assess:**
- Change UI components using the design system
- Integrate a new API endpoint with proper typing
- Build middleware/BFF layer from backend contracts
- Fix build errors and type issues
- Fix component rendering or state issues
- Implement a Figma design with design-system fidelity
- Push code changes that maintain Figma design consistency
- Add responsive/adaptive layouts using existing patterns
- Refactor components to follow existing conventions

**Backend standard tasks to assess:**
- Modify API endpoints or add new ones
- Write test cases from business logic understanding
- Change database schema with proper migrations
- Modify auth/authorization logic safely
- Update CI/CD pipeline configuration
- Add monitoring or logging to a service
- Refactor service layer following existing patterns
- Handle cross-service changes understanding dependencies

**Full-Stack standard tasks to assess (in addition to above):**
- End-to-end feature implementation (UI → API → DB)
- Data flow changes spanning frontend and backend
- Authentication/authorization changes across the stack

### 4) Evidence Log

List all files and symbols reviewed, grouped by dimension:
```
F1 Design System:
  - components/ui/ (21 components read)
  - tailwind.config.js (theme tokens extracted)
  - ...

B1 Architecture:
  - src/server.ts (entry point)
  - src/routes/ (14 route files)
  - ...
```

### 5) Gap Analysis & Risk Register

For each gap found:
```
Gap: [description]
├─ Impact: [what tasks are blocked]
├─ Risk Level: Critical / High / Medium / Low
├─ Classification: [Contract | Behavior | Ownership | Validation | Infrastructure]
└─ To close: [what the agent needs — files, docs, access, or clarification]
```

Top 3 regression risks if implementation starts now.

### 6) Path to 100%

Prioritized checklist ordered by impact:
```
1. [Action] — closes [gap], unlocks [tasks], expected impact: +X%
2. [Action] — closes [gap], unlocks [tasks], expected impact: +X%
...
```

---

## Decision Rules

- If critical API contracts are missing → cap Agentic Delivery Readiness at 60%.
- If no tests exist → raise risk severity for all modification tasks.
- If auth/security flow is unclear → mark all auth-related tasks as Blocked.
- If build pipeline is untested by the agent → add "build verification" as prerequisite to all tasks.
- Never claim 100% unless ALL critical flows, contracts, and verification paths are evidenced.
- Separate **observed facts** (from code/docs) from **inferred behavior** (from patterns/naming).
- When in doubt, score lower. Overconfidence is more dangerous than underconfidence.

## Style Rules

- Be concise and evidence-driven.
- Every percentage MUST be justified by concrete file/symbol references.
- Do not present assumptions as facts — clearly label inferences.
- Use tables and visual bars for readability.
- Group related gaps together.
- The output should be actionable — the user should know exactly what to do next.
