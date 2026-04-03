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

### Step 0: Discover & Respect Project Rules (MANDATORY — Run First)

Before any analysis begins, you MUST find and read ALL existing project instruction and rule files. These rules are set by the project owner and **OVERRIDE any default behavior** of this skill or its subagents.

**Search for and read these files (if they exist):**
- `CLAUDE.md` — Claude Code project instructions
- `.cursorrules` — Cursor IDE rules
- `.windsurfrules` — Windsurf rules
- `.github/copilot-instructions.md` — GitHub Copilot instructions
- `.clinerules` — Cline rules
- `.roo/rules/` — Roo Code rules directory
- `.continue/rules/` — Continue rules directory
- `AGENTS.md` — Codex CLI / Zed instructions
- `.amazonq/rules/` — Amazon Q rules
- `.augment/rules/` — Augment Code rules
- `.tabnine/guidelines/` — Tabnine guidelines
- Any other `*.rules`, `*.md` files in `.cursor/`, `.claude/`, `.github/` directories

**What to extract from project rules:**
1. **Build restrictions** — Can the agent run build commands? (e.g., "Never build the repository")
2. **Deployment restrictions** — Can the agent deploy or run deployment commands?
3. **Code style mandates** — Required formatting, naming conventions, import order, etc.
4. **Framework mandates** — Required libraries, UI primitives, styling approach (e.g., "use Tailwind + Radix UI")
5. **Testing mandates** — Required test frameworks, patterns, coverage thresholds
6. **File naming conventions** — PascalCase components, kebab-case folders, etc.
7. **Commit conventions** — Conventional commits, signed commits, etc.
8. **Any explicit prohibitions** — Things the agent must NOT do

**Critical rule**: Pass all discovered project rules to every subagent as context. Subagents MUST NOT violate project rules during analysis. For example:
- If rules say "never build" → subagents MUST NOT run build commands; score Build Health (F3) by reading configs only
- If rules mandate specific frameworks → factor this into Design System scoring
- If rules restrict deployment → DevOps scoring should note this constraint

Store the discovered rules as `PROJECT_RULES` context and include them in every subagent prompt.

### Step 0.5: Detect Available Tools & Plugins

Check what tools, MCP servers, and plugins are available to the agent. This directly affects scoring caps and what the agent can actually do.

**Check for:**
- **Figma tools**: Figma MCP server, Code Connect MCP, Pencil MCP, Figma plugin — lifts F4 cap above 20
- **Design tools**: Stitch, Pencil, UI/UX design MCPs — boosts F1 scoring potential
- **Database tools**: Database MCP servers, query tools — boosts B1 scoring potential
- **CI/CD tools**: GitHub Actions MCP, deployment MCPs — boosts B4 scoring potential
- **Browser/preview tools**: Preview MCP, Chrome control — enables runtime verification
- **Documentation tools**: Notion MCP, Confluence MCP — may provide architecture docs
- **Communication tools**: Slack MCP — may have context about ongoing work

**How to detect (by platform):**
- **Claude Code**: Check system prompt for MCP tool listings, check `settings.json` for `enabledPlugins` and permissions
- **Cursor**: Check for enabled extensions and composer capabilities
- **Other tools**: Check for available tool/function definitions

Store discovered tools as `AVAILABLE_TOOLS` context and pass to relevant subagents.

---

## Execution Strategy

### Subagent Architecture

This skill performs a **deep, comprehensive** codebase analysis. To do this efficiently, you MUST spawn parallel subagents to analyze different dimensions concurrently. Do NOT attempt to scan the entire codebase sequentially — it is too slow and risks incomplete coverage.

**How parallelism works across tools:**
- **Claude Code**: Use the `Agent` tool to spawn multiple subagents in a single message. Each subagent gets its own context window and can read hundreds of files independently. Use `subagent_type: "Explore"` for read-only scanning agents. Launch them in parallel (multiple Agent calls in one response).
- **Cursor**: Use parallel tool calls to read and search files concurrently. If Composer mode is available, spawn multiple composer agents for independent dimension analysis. Use background agents where supported.
- **Other tools**: Use whatever parallelism the tool supports — concurrent file reads, background tasks, or sequential scan if no parallelism is available.

### Subagent Definitions

After Step 1 (Project Type Detection), spawn the following subagents **in parallel** based on the detected project type.

**CRITICAL — Every subagent prompt MUST be prefixed with:**
```
CONTEXT — Respect these project rules during analysis:
{PROJECT_RULES}  ← insert the rules discovered in Step 0

CONTEXT — Available tools and plugins:
{AVAILABLE_TOOLS}  ← insert the tools discovered in Step 0.5

IMPORTANT: Do NOT run any commands that violate the project rules above.
If rules say "never build" — do NOT run build commands; analyze by reading configs only.
If rules mandate specific frameworks — note compliance/deviation in your findings.
```

#### For Frontend projects — spawn these 4 agents simultaneously:

**Agent 1: Design System Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the design system and UI component architecture of this project.
Read and report on:
- All files in the UI/component library directories (components/ui/, components/common/, etc.)
- Component props, variants, composition patterns
- Tailwind/CSS config, theme tokens, CSS variables, design tokens
- Layout patterns, grid systems, responsive breakpoints, spacing scale
- Storybook config if present
- Any design token files (.json, .ts, .css) used for theming
Return: list of every component file read, theme tokens found, design patterns identified,
and any gaps (missing variants, undocumented components, inconsistent patterns).
```

**Agent 2: API & Data Flow Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the API integration layer and data flow architecture of this project.
Read and report on:
- All API client files (GraphQL documents, REST clients, tRPC routers, fetch wrappers)
- API hooks, query/mutation definitions, codegen output
- Schema files, type definitions for request/response shapes
- State management setup (Redux, Zustand, Context, etc.) and how API data flows into state
- Data transformation layers, middleware, BFF patterns, error handling
- Trace at least 2 complete data paths: API call → state update → UI render
Return: list of every API file read, schemas found, data flows traced with specific file:line references,
and any gaps (untyped endpoints, missing error handling, unclear data flows).
```

**Agent 3: Build & Toolchain Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the build toolchain, developer experience, and code health of this project.
IMPORTANT: Do NOT run build commands — analyze by reading config files only.
Read and report on:
- Build config (next.config, vite.config, webpack, turbopack, etc.)
- TypeScript config (tsconfig) — strict mode, path aliases, module resolution
- Linting config (eslint, biome) and formatting (prettier)
- Pre-commit hooks (husky, lint-staged, lefthook)
- CI/CD pipeline files (.github/workflows, Jenkinsfile, etc.)
- Module resolution: barrel files, import patterns, circular dependency risks
- Environment variable handling (.env files, env validation schemas)
- SSR/CSR/ISR boundaries if applicable
Return: list of every config file read, build pipeline steps identified,
type safety assessment, and any issues found.
```

**Agent 4: Figma & Design Bridge Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the Figma-to-code bridge and design integration of this project.
Read and report on:
- Figma Code Connect config files (.figma, codeconnect.config.ts, etc.)
- Design token bridge files (Style Dictionary, Tokens Studio, etc.)
- Component-to-Figma mapping (naming alignment between code components and potential Figma components)
- Design system rule files, if any
- Any Figma plugin configs, MCP tool configs, or design tool integrations
- CSS custom properties / Tailwind theme that could serve as design tokens
Return: list of every design-related file found, mapping coverage assessment,
and specific gaps blocking Figma integration.
```

#### For Backend projects — spawn these 4 agents simultaneously:

**Agent 5: Architecture Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the complete backend architecture of this project.
Read and report on:
- Entry point and server setup (main server file, framework bootstrap)
- All route/endpoint definitions — list every API endpoint with its HTTP method and path
- Database schema: models, migrations, schema files, ORM config
- Authentication and authorization flow — trace the full auth chain
- Middleware chain — list all middleware in order
- External service integrations (message queues, caches, third-party APIs, webhooks)
- Microservice boundaries if applicable (service discovery, API gateways, inter-service communication)
Return: complete architecture map with file references, service topology diagram (text-based),
and gaps (undocumented services, unclear auth flow, missing schema access).
```

**Agent 6: Business Logic Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the business logic and domain model of this project.
Read and report on:
- Core domain entities and their relationships (models, types, interfaces)
- Service/domain layer: business rules, validation logic, workflow orchestration
- Data transformation and processing pipelines
- Feature flags, role-based behavior, conditional business logic
- Edge cases and guards: error conditions, boundary checks, business constraints
- Trace at least 3 core business operations end-to-end
Return: domain entity map, business rules catalog with file:line references,
workflow descriptions, and gaps (implicit rules, scattered logic, undocumented behavior).
```

**Agent 7: Test & Quality Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the testing infrastructure and quality assurance setup of this project.
Read and report on:
- Test framework and config (jest, vitest, pytest, mocha, etc.)
- Existing test files — read at least 5 representative tests to understand patterns
- Test utilities: factories, fixtures, mocks, seed data, test helpers
- Coverage config and reports if available
- Integration vs unit vs e2e test distribution
- Which critical business paths have test coverage and which don't
Return: test framework details, test pattern examples, coverage assessment,
list of tested vs untested critical paths, and gaps (missing fixtures, no integration tests, etc.).
```

**Agent 8: DevOps & Infrastructure Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the DevOps pipeline and infrastructure of this project.
IMPORTANT: Do NOT run deployment or infrastructure commands — analyze by reading config files only.
Read and report on:
- CI/CD pipeline config (.github/workflows, Jenkinsfile, GitLab CI, CircleCI, etc.)
- Dockerfile and docker-compose — build stages, multi-stage builds, compose services
- Infrastructure as code (Terraform, CDK, Pulumi, CloudFormation, serverless.yml)
- Deployment config (Vercel, Netlify, Cloudflare, AWS, GCP, platform configs)
- Environment management: staging/production separation, env var handling, secrets
- Monitoring and observability: logging config, APM, alerting rules, health checks
- Database migrations strategy and rollback procedures
Return: complete pipeline map from commit to production, environment topology,
infrastructure components identified, and gaps (no rollback strategy, missing monitoring, etc.).
```

#### For Full-Stack / Monorepo — spawn ALL 8 agents simultaneously.

#### Universal agents — ALWAYS spawn these 2 agents (in parallel with dimension agents):

**Agent 9: Platform & Integration Scanner**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the overall platform, tech stack, and third-party integrations of this project.
Read and report on:
- package.json (or equivalent) — ALL dependencies, categorized by purpose
- All configuration files at the project root
- Third-party service integrations: payments (Stripe, etc.), auth (Auth0, Clerk, etc.),
  analytics (PostHog, Segment, etc.), messaging (Twilio, SendGrid, etc.), storage (S3, etc.)
- Environment configuration and deployment targets
- Monorepo structure if applicable (workspaces, packages, shared libs)
- Project documentation: README, architecture docs, API docs, onboarding guides
- Project instruction files: CLAUDE.md, .cursorrules, .windsurfrules, .github/copilot-instructions.md
Return: complete tech stack inventory, dependency map, integration catalog with config locations,
and gaps (undocumented integrations, unclear environment setup).
```

**Agent 10: Business Domain & User Journey Mapper**
```
{INSERT PROJECT_RULES AND AVAILABLE_TOOLS CONTEXT HERE}

Analyze the business domain and how user journeys map to code in this project.
Read and report on:
- What does this application DO? Identify the core product/business purpose.
- Map at least 5 core user journeys through the code (e.g., sign up, search, purchase, manage account)
- For each journey: trace the route → component → state → API → (backend if available) path
- Domain model alignment: do code names match business concepts?
- Are business rules centralized or scattered across UI/API/backend?
- Feature completeness: are there partial implementations, TODOs, or placeholder code?
Return: business domain summary, user journey traces with file:line references,
domain-code alignment assessment, and gaps (broken journeys, scattered logic, orphaned code).
```

### Subagent Merge Strategy

After ALL subagents return their results:

1. **Collect** all evidence logs from each agent
2. **Cross-reference** findings — one agent's output may affect another's score (e.g., if Agent 6 finds unclear business logic, Agent 7's test generation cap applies)
3. **Score** each dimension using the rubrics below, citing evidence from the relevant agent
4. **Identify** cross-cutting gaps that span multiple dimensions
5. **Generate** the final output in the required format

If any subagent fails or returns incomplete results, note this in the evidence log and apply the relevant auto-cap rule for that dimension.

---

## Execution Steps (Summary)

```
Step 0:   Discover & Respect Project Rules     ← MANDATORY, run first
Step 0.5: Detect Available Tools & Plugins      ← run immediately after Step 0
Step 1:   Project Type Detection                ← classify as FE / BE / Full-Stack / Monorepo
Step 2:   Spawn Subagents for Deep Scan         ← parallel, with PROJECT_RULES + AVAILABLE_TOOLS context
Step 3:   Merge Results & Score Dimensions      ← cross-reference, apply caps and modifiers
Step 4:   Generate Output                       ← full report in required format
```

### Step 1: Project Type Detection

Scan the project root to determine the project type. Read `package.json`, config files, directory structure, and framework markers.

Classify as one of:
- **Frontend** — Client-side app (React, Vue, Svelte, Angular, Next.js pages/app dir, etc.)
- **Backend** — Server-side (Express, Fastify, NestJS, Django, Rails, Go services, etc.)
- **Full-Stack** — Both frontend and backend in the same repo
- **Monorepo** — Multiple packages/services (detect with workspaces, Turborepo, Nx, Lerna)

### Step 2: Spawn Subagents for Deep Codebase Scan

Based on the project type detected in Step 1, launch the relevant subagents **in parallel** as defined in the Subagent Architecture section above. Each subagent independently scans its assigned dimension and returns structured findings.

**IMPORTANT**: Inject `PROJECT_RULES` and `AVAILABLE_TOOLS` context (from Steps 0 and 0.5) into every subagent prompt.

**You must read actual file contents** — do not score based on file names alone. Instruct each subagent to read files, not just list them.

#### What the subagents collectively cover:
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

### Step 3: Merge Results & Score Each Dimension

Once all subagents return, merge their findings and:
1. Score each applicable dimension using the rubrics below
2. Evaluate cross-cutting modifiers (C1-C4)
3. Apply auto-cap rules and cross-dimension dependencies
4. Every score MUST reference specific files, functions, or patterns found by the subagents

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

### CROSS-CUTTING CONCERNS (bonus modifiers — always assessed)

These are not standalone dimensions but **modify the Agentic Delivery Readiness** score. They represent risks the agent should flag even if the primary dimensions score well.

#### C1: Security Posture (modifier: -5 to +5)

| Modifier | Criteria |
|----------|----------|
| -5 | Secrets found in code, no auth checks, SQL injection / XSS risks detected |
| -3 | Auth exists but not fully traced, env vars in code, no input validation pattern |
| 0 | Standard security patterns in place, auth traced, no obvious vulnerabilities |
| +3 | Security middleware well-structured, input validation consistent, CORS/CSP configured |
| +5 | Full security posture understood — OWASP patterns, rate limiting, secrets management, audit logging |

**What to check**: .env files in repo (should be gitignored), auth middleware, input validation, CORS config, CSP headers, dependency audit (known vulnerabilities).

#### C2: Accessibility (modifier: -3 to +3)

| Modifier | Criteria |
|----------|----------|
| -3 | No a11y consideration found — no aria attributes, no semantic HTML patterns |
| 0 | Basic a11y patterns present (semantic elements, some aria labels) |
| +3 | Full a11y system — proper ARIA roles, keyboard navigation, screen reader support, a11y testing |

**What to check**: Semantic HTML usage, ARIA attributes in components, keyboard event handlers, focus management, a11y testing tools (axe, pa11y), eslint-plugin-jsx-a11y.

#### C3: Internationalization / i18n (modifier: -2 to +2)

| Modifier | Criteria |
|----------|----------|
| -2 | Hardcoded user-facing strings everywhere, no i18n setup |
| 0 | i18n library present (next-intl, react-i18next, etc.) but partial coverage |
| +2 | Full i18n system — translation files, locale detection, RTL support if applicable |

**What to check**: i18n library in dependencies, translation files, locale configs, string extraction patterns.

#### C4: Project Rules Compliance (modifier: -5 to 0)

| Modifier | Criteria |
|----------|----------|
| -5 | Project rules found but code violates them significantly (e.g., wrong frameworks, ignored conventions) |
| -3 | Partial compliance — some rules followed, some ignored |
| 0 | Full compliance with all discovered project rules, or no rules found |

**What to check**: Compare `PROJECT_RULES` from Step 0 against actual code patterns.

**Apply modifiers to Agentic Delivery Readiness:**
```
Agentic Delivery Readiness = base_readiness + C1 + C2 + C3 + C4
(clamp to 0-100 range)
```

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
If Monorepo:       Score each package/service separately, then:
                   Overall = weighted average by package complexity
                   (report per-package scores AND aggregate)
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

### 4) Cross-Cutting Modifiers

```
Security Posture (C1):           [+X / -X]  — [brief justification]
Accessibility (C2):              [+X / -X]  — [brief justification]
Internationalization (C3):       [+X / -X]  — [brief justification]
Project Rules Compliance (C4):   [+X / -X]  — [brief justification]
```

### 5) Project Rules Detected

List all rule files found and key constraints:
```
Rules found:
  - CLAUDE.md: code style (Prettier, Tailwind, Radix UI), build commands listed
  - .cursorrules: NEVER run build or deploy commands
  - settings.json: Figma plugin enabled, Pencil MCP allowed

Key constraints applied:
  - Build Health (F3): scored by config reading only (no build execution)
  - Figma Bridge (F4): cap lifted to 25 (Figma MCP available)
```

### 6) Evidence Log

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

Available Tools:
  - Figma MCP (design access)
  - Pencil MCP (design editor)
  - Preview MCP (browser preview)
  - ...
```

### 7) Gap Analysis & Risk Register

For each gap found:
```
Gap: [description]
├─ Impact: [what tasks are blocked]
├─ Risk Level: Critical / High / Medium / Low
├─ Classification: [Contract | Behavior | Ownership | Validation | Infrastructure]
└─ To close: [what the agent needs — files, docs, access, or clarification]
```

Top 3 regression risks if implementation starts now.

### 8) Path to 100%

Prioritized checklist ordered by impact:
```
1. [Action] — closes [gap], unlocks [tasks], expected impact: +X%
2. [Action] — closes [gap], unlocks [tasks], expected impact: +X%
...
```

---

## Decision Rules

### Core Rules
- If critical API contracts are missing → cap Agentic Delivery Readiness at 60%.
- If no tests exist → raise risk severity for all modification tasks.
- If auth/security flow is unclear → mark all auth-related tasks as Blocked.
- If build pipeline is untested by the agent → add "build verification" as prerequisite to all tasks.
- Never claim 100% unless ALL critical flows, contracts, and verification paths are evidenced.
- Separate **observed facts** (from code/docs) from **inferred behavior** (from patterns/naming).
- When in doubt, score lower. Overconfidence is more dangerous than underconfidence.

### Project Rules Integration
- If project rules say "never build" → F3 (Build Health) must be scored by reading configs only. Note this constraint in the evidence log. Do NOT penalize the score for being unable to run builds — score based on config readability.
- If project rules mandate specific frameworks → verify code compliance. Non-compliance lowers C4 modifier AND the relevant dimension score.
- If project rules specify code style → subagents must note any deviations found, but do NOT auto-fix during analysis.
- Any project rule violation found during scanning → add to Gap Analysis as a "Compliance" classification gap.

### Tool & Plugin Awareness
- If Figma MCP/plugin is available → lift F4 (Figma Bridge) cap from 20 to 25. The agent CAN access Figma directly.
- If design tools (Pencil MCP, Stitch, UI/UX Pro Max) are available → note in F1 evidence as extended capability.
- If database MCP tools are available → B1 (Architecture) can verify schemas directly, lift cap if applicable.
- If browser/preview tools are available → agent can verify runtime behavior, note in Agentic Delivery Readiness.
- List all available tools in the Evidence Log under a "Available Tools" section.

## Style Rules

- Be concise and evidence-driven.
- Every percentage MUST be justified by concrete file/symbol references.
- Do not present assumptions as facts — clearly label inferences.
- Use tables and visual bars for readability.
- Group related gaps together.
- The output should be actionable — the user should know exactly what to do next.
