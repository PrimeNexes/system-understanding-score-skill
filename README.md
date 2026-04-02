# System Understanding Score Skill

An AI coding agent skill that deeply analyzes a codebase and produces a quantified understanding score — telling the user exactly how much the agent understands, what it can build right now, and what gaps remain.

Works with **any AI coding tool** that supports markdown instructions — Claude Code, Cursor, Windsurf, Zed, GitHub Copilot, Cline, Roo Code, Continue, Codex CLI, Amazon Q, Augment Code, Tabnine, Aider, Devin, Bolt, v0, Lovable, and more.

## What It Does

Performs a comprehensive, evidence-based codebase scan and produces:

- **Overall Understanding %** — how deeply the agent comprehends the system
- **Agentic Delivery Readiness %** — how much can be built autonomously right now
- **Dimension-level scores** — granular breakdowns for frontend, backend, and universal concerns
- **Standard task confidence list** — specific tasks rated by execution confidence
- **Gap analysis** — what the agent doesn't understand and what's needed to close each gap
- **Path to 100%** — prioritized checklist to reach full understanding

## Scored Dimensions

### Frontend (when applicable)
| Dimension | Max | What it measures |
|-----------|-----|-----------------|
| Design System Mastery | 25 | Can the agent change UI using the existing design system? |
| API Integration & Middleware | 25 | Can it read backend contracts and build integration layers? |
| Build & Component Health | 25 | Can it detect and fix build/component issues? |
| Figma Design Bridge | 25 | Can it link Figma designs to code and maintain fidelity? |

### Backend (when applicable)
| Dimension | Max | What it measures |
|-----------|-----|-----------------|
| Architecture Understanding | 25 | Does it understand the entire backend architecture? |
| Business Logic Comprehension | 25 | Does it know WHAT the backend does, not just how? |
| Test Case Generation | 25 | Can it write meaningful tests from business logic? |
| DevOps Pipeline Understanding | 25 | Does it understand the deployment and infra pipeline? |

### Universal (always scored)
| Dimension | Max | What it measures |
|-----------|-----|-----------------|
| Platform & Tech Stack | 50 | Holistic platform comprehension |
| Business Logic-Code Alignment | 50 | Is business logic properly reflected in code? |

## Install

### Quick Reference

| Tool | Method | Install path |
|------|--------|-------------|
| Claude Code | File | `.claude/skills/system-understanding-score/SKILL.md` |
| Cursor | File | `.cursor/skills/system-understanding-score/SKILL.md` or `.cursorrules` |
| Windsurf | File | `.windsurfrules` or `~/.windsurf/rules/` |
| Zed | File | `.rules` or `AGENTS.md` |
| GitHub Copilot | File | `.github/copilot-instructions.md` or `.github/prompts/` |
| Cline | File | `.clinerules` |
| Roo Code | File | `.roo/rules/system-understanding-score.md` |
| Continue | File | `.continue/rules/system-understanding-score.md` |
| Codex CLI | File | `AGENTS.md` |
| Amazon Q | File | `.amazonq/rules/system-understanding-score.md` |
| Augment Code | File | `.augment/rules/system-understanding-score.md` |
| Tabnine | File | `.tabnine/guidelines/system-understanding-score.md` |
| Aider | Config | `--read SKILL.md` or `.aider.conf.yml` |
| Devin | Web UI | Settings > Knowledge |
| Bolt.new | Web UI | Project Settings > Knowledge |
| v0 | Web UI | Settings > Preferences |
| Lovable | Web UI | Project Settings > Knowledge |

---

### Claude Code

**Project-level** (scoped to a single project):
```bash
mkdir -p .claude/skills/system-understanding-score
cp SKILL.md .claude/skills/system-understanding-score/SKILL.md
```

**User-level** (available across all projects):
```bash
mkdir -p ~/.claude/skills/system-understanding-score
cp SKILL.md ~/.claude/skills/system-understanding-score/SKILL.md
```

**Via skills CLI:**
```bash
npx skills add PrimeNexes/system-understanding-score-skill
```

---

### Cursor

**Project-level (skills directory):**
```bash
mkdir -p .cursor/skills/system-understanding-score
cp SKILL.md .cursor/skills/system-understanding-score/SKILL.md
```

**User-level:**
```bash
mkdir -p ~/.cursor/skills/system-understanding-score
cp SKILL.md ~/.cursor/skills/system-understanding-score/SKILL.md
```

**Alternative — append to .cursorrules:**
```bash
echo "" >> .cursorrules
cat SKILL.md >> .cursorrules
```

---

### Windsurf

**Project-level:**
```bash
echo "" >> .windsurfrules
cat SKILL.md >> .windsurfrules
```

**User-level (global rules):**
```bash
mkdir -p ~/.windsurf/rules
cp SKILL.md ~/.windsurf/rules/system-understanding-score.md
```

---

### Zed

**Project-level — `.rules` file:**
```bash
echo "" >> .rules
cat SKILL.md >> .rules
```

**Alternative — `AGENTS.md` (also read by Codex CLI):**
```bash
echo "" >> AGENTS.md
cat SKILL.md >> AGENTS.md
```

Zed also reads `.cursorrules` and `.windsurfrules` for compatibility.

---

### GitHub Copilot

**Instructions file:**
```bash
mkdir -p .github
echo "" >> .github/copilot-instructions.md
cat SKILL.md >> .github/copilot-instructions.md
```

**Prompt file (Copilot Chat):**
```bash
mkdir -p .github/prompts
cp SKILL.md .github/prompts/system-understanding-score.prompt.md
```

---

### Cline

```bash
echo "" >> .clinerules
cat SKILL.md >> .clinerules
```

---

### Roo Code

**Project-level:**
```bash
mkdir -p .roo/rules
cp SKILL.md .roo/rules/system-understanding-score.md
```

**User-level (global):**
```bash
mkdir -p ~/.roo/rules
cp SKILL.md ~/.roo/rules/system-understanding-score.md
```

Also reads `.clinerules` for legacy compatibility.

---

### Continue

```bash
mkdir -p .continue/rules
cp SKILL.md .continue/rules/system-understanding-score.md
```

---

### Codex CLI (OpenAI)

**Project-level — append to AGENTS.md:**
```bash
echo "" >> AGENTS.md
cat SKILL.md >> AGENTS.md
```

**User-level (global):**
```bash
mkdir -p ~/.codex
echo "" >> ~/.codex/AGENTS.md
cat SKILL.md >> ~/.codex/AGENTS.md
```

---

### Amazon Q Developer

```bash
mkdir -p .amazonq/rules
cp SKILL.md .amazonq/rules/system-understanding-score.md
```

---

### Augment Code

```bash
mkdir -p .augment/rules
cp SKILL.md .augment/rules/system-understanding-score.md
```

---

### Tabnine

```bash
mkdir -p .tabnine/guidelines
cp SKILL.md .tabnine/guidelines/system-understanding-score.md
```

---

### Aider

**Via config file** (`.aider.conf.yml`):
```yaml
read:
  - SKILL.md
```

**Via CLI:**
```bash
aider --read SKILL.md
```

---

### Devin

1. Open Devin web interface
2. Go to **Settings & Library > Knowledge**
3. Create a new knowledge entry
4. Paste the contents of `SKILL.md`

---

### Bolt.new

1. Click the **gear icon** in your project
2. Go to **All project settings > Knowledge**
3. Paste the contents of `SKILL.md`

For global rules: **Personal Settings > Personal Knowledge**

---

### v0 by Vercel

1. Go to **Settings > Preferences**
2. Paste the contents of `SKILL.md` as custom instructions

Also supports project-level and chat-level instructions.

---

### Lovable

1. Click the **gear icon** in your project
2. Go to **Project Settings > Knowledge**
3. Paste the contents of `SKILL.md`

For workspace-wide rules: use **Workspace Knowledge** instead.

---

### Any Other AI Coding Tool

The skill is a standard markdown file. To use it with any tool:

1. **If the tool supports a rules/instructions file** — append or include `SKILL.md` content
2. **If the tool supports a rules directory** — copy `SKILL.md` into it
3. **If the tool supports system prompts** — paste the content as a system prompt
4. **Manual** — paste the content into the chat when you want the agent to score the project

## Usage

Once installed, trigger the skill by asking:

- "How much do you understand this codebase?"
- "What's your confidence score for this project?"
- "What can you build right now?"
- "Score this project"
- "What gaps do you have in understanding?"
- "Can you work on this project autonomously?"

## Example Output (abbreviated)

```
# System Understanding Score: StayNow Web
Project Type: Frontend (Next.js 14)
Scanned: 342 files across 48 directories

Overall Understanding:       78% — High
Agentic Delivery Readiness:  65%

Understanding:  [████████████████████░░░░░] 78%
Delivery Ready: [████████████████░░░░░░░░░] 65%

Design System Mastery:           21/25
API Integration & Middleware:    18/25
Build & Component Health:        22/25
Figma Design Bridge:             12/25
Platform Comprehension:          38/50
Business Logic-Code Alignment:   34/50

Ready to Execute (90%+ confidence):
- [x] Change UI components using Radix + Tailwind design system
- [x] Fix TypeScript build errors and type issues
- [x] Add responsive layouts using existing grid patterns

Can Attempt with Caveats (60-89%):
- [ ] Integrate new API endpoint — caveat: GraphQL schema partially typed
- [ ] Implement Figma design — caveat: no Code Connect configured

Blocked (<60%):
- [ ] Push code maintaining Figma design — blocked by: no Figma bridge
```

## Confidence Bands

| Range | Band | Meaning |
|-------|------|---------|
| 0-30% | Low | Major gaps, high risk of incorrect changes |
| 31-60% | Medium | Can handle isolated tasks, needs guidance |
| 61-80% | High | Most tasks autonomous, some blind spots |
| 81-95% | Very High | Near-full autonomy, minor edge-case gaps |
| 96-100% | Complete | Full mastery (never claimed without total evidence) |

## Key Design Principles

- **Tool-agnostic** — works with any AI coding agent that can read markdown instructions
- **Evidence-driven** — every score references specific files and symbols
- **Conservative scoring** — overconfidence is more dangerous than underconfidence
- **Facts vs. inferences** — observed code is separated from pattern-based guesses
- **Actionable output** — gap analysis tells you exactly what's needed to close each gap
- **Auto-cap rules** — missing design tokens, schemas, or tests automatically cap relevant scores

## License

MIT
