<!-- agent:brain -->
<!-- Commit convention: Conventional Commits with agent scope -->
<!-- Format: type(agent:brain): description -->
<!-- Tags: semantic versioning (v0.1.0, v1.0.0) -->
<!-- Branch: main (direct commit) -->

# Claude Code Memory Setup Guide

> A reference for setting up persistent memory, context, and feedback loops when starting a new project with Claude Code.

Claude Code starts every session blank. These patterns fix that. This guide covers two approaches (pick the one that fits your workflow) plus a lightweight baseline pattern you can adopt immediately.

**IDE:** VS Code  
**OS:** Windows  
**Shell:** PowerShell (default) or Git Bash  
**Agent:** `agent:brain`  
**Commit format:** `type(agent:brain): message`

---

## Minimum Configuration (Must-Haves)

Before choosing an approach, every new project needs these. Copy-paste and go.

| # | File | One-liner | Why it's non-negotiable |
|---|------|-----------|-------------------------|
| 1 | `.claude/CLAUDE.md` | Master instructions: tech stack, build commands, conventions | Without this, Claude asks "what framework?" every session |
| 2 | `.claude/memory/feedback.md` | Running log of corrections and anti-patterns | Stops Claude from repeating the same mistakes |
| 3 | `.claude/memory/architecture.md` | Active design decisions and system state | Prevents contradictory suggestions across sessions |
| 4 | `.gitignore` entry | `# .claude/memory/` (optional, your call) | Decide once whether memory is committed or local-only |

### 30-Second Setup (PowerShell)

```powershell
# From your project root in VS Code terminal (Ctrl+`)
New-Item -ItemType Directory -Force -Path .claude\memory

# 1. CLAUDE.md -- your project's identity card
@"
# Project: <name>

## Stack
- <language/framework>
- <database>
- <key dependencies>

## Commands
- ``<dev command>`` -- run locally
- ``<test command>`` -- run tests
- ``<build command>`` -- production build

## Rules
- <your top 3-5 coding conventions>

## Memory
Always read ``.claude/memory/feedback.md`` and ``.claude/memory/architecture.md`` at session start.
"@ | Set-Content .claude\CLAUDE.md -Encoding UTF8

# 2. feedback.md -- corrections accumulate here
@"
# Corrections
<!-- One line per lesson. Date it. -->
"@ | Set-Content .claude\memory\feedback.md -Encoding UTF8

# 3. architecture.md -- decisions live here
@"
# Decisions
<!-- One line per decision. Date it. -->
"@ | Set-Content .claude\memory\architecture.md -Encoding UTF8
```

### 30-Second Setup (Git Bash alternative)

```bash
# From your project root in VS Code terminal (select Git Bash from dropdown)
mkdir -p .claude/memory

cat > .claude/CLAUDE.md << 'EOF'
# Project: <name>

## Stack
- <language/framework>
- <database>
- <key dependencies>

## Commands
- `<dev command>` -- run locally
- `<test command>` -- run tests
- `<build command>` -- production build

## Rules
- <your top 3-5 coding conventions>

## Memory
Always read `.claude/memory/feedback.md` and `.claude/memory/architecture.md` at session start.
EOF

cat > .claude/memory/feedback.md << 'EOF'
# Corrections
<!-- One line per lesson. Date it. -->
EOF

cat > .claude/memory/architecture.md << 'EOF'
# Decisions
<!-- One line per decision. Date it. -->
EOF
```

That's it. Claude now starts every session knowing your stack, your rules, and your past corrections. Everything below is an upgrade path.

---

## Prerequisites

| Requirement | Version | Install on Windows | Notes |
|-------------|---------|-------------------|-------|
| Claude CLI (`claude`) | Latest | `npm install -g @anthropic-ai/claude-code` | [Install docs](https://docs.anthropic.com/en/docs/claude-code) |
| Python | 3.9+ | [python.org](https://www.python.org/downloads/windows/) or `winget install Python.Python.3.11` | Add to PATH during install |
| Node.js | 22+ | [nodejs.org](https://nodejs.org/) or `winget install OpenJS.NodeJS` | Required for learning-loop only |
| Git | 2.x | [git-scm.com](https://git-scm.com/download/win) or `winget install Git.Git` | Ships Git Bash |
| VS Code | Latest | [code.visualstudio.com](https://code.visualstudio.com/) | Terminal: `Ctrl+\`` |

### VS Code Extensions (Recommended)

```powershell
# Install from VS Code terminal
code --install-extension yzhang.markdown-all-in-one
code --install-extension bierner.markdown-preview-github-styles
```

These give you live preview of your memory files (`Ctrl+Shift+V` to preview any .md file).

---

## Approach 1: claude-code-memory (Zero-Dependency Markdown Templates)

**Best for:** Solo devs, small projects, anyone who wants deterministic recall with zero moving parts.

**Philosophy:** Curated markdown files that Claude reads on startup. No daemon, no vector DB, no cloud. You control exactly what gets recalled.

**Source:** [LuciferForge/claude-code-memory](https://github.com/LuciferForge/claude-code-memory)

### Setup Steps (Windows / VS Code Terminal)

```powershell
# 1. Clone the template repo
git clone https://github.com/LuciferForge/claude-code-memory.git
cd claude-code-memory

# 2. Run setup for your project
python setup.py --project C:\Users\<you>\projects\your-project

# Or set up just the global config
python setup.py --global-only

# 3. (Alternative) Manual copy if you prefer
Copy-Item templates\CLAUDE.md $env:USERPROFILE\.claude\CLAUDE.md
```

### Resulting Structure

```
%USERPROFILE%\.claude\
  CLAUDE.md                          # Global custom instructions (loaded every session)
  projects\<project>\
    CLAUDE.md                        # Project-specific instructions
    memory\
      MEMORY.md                      # Index file (always loaded, MUST stay under 200 lines)
      user_profile.md                # Who you are, skills, preferences
      feedback_style.md              # How you want Claude to behave
      project_overview.md            # What you're building, architecture decisions
      reference_links.md             # External resources, API docs, dashboards
```

### Memory Types

| Type | What to Store | Example |
|------|--------------|----------|
| `user` | Your role, skills, preferences | "Senior backend dev, prefers Go, hates verbose output" |
| `feedback` | Corrections you've given Claude | "Don't mock the database in tests, use real DB" |
| `project` | Ongoing work context, deadlines, decisions | "Auth rewrite driven by compliance, not tech debt" |
| `reference` | Pointers to external resources | "Bug tracker is Linear project INGEST" |

### Key Rules

1. **MEMORY.md must stay under 200 lines.** Truncation is silent. Check: `(Get-Content .claude\memory\MEMORY.md).Count`
2. **One topic per file.** Small focused files let Claude load only what's relevant.
3. **Feedback memories are highest value.** Every correction should become a memory.
4. **Don't store what Claude can read.** Code conventions, file paths, git history: Claude can look those up.
5. **Review monthly.** Delete outdated memories. Lean beats bloated.

### Frontmatter Format

Each topic file uses this schema:

```markdown
---
name: Short descriptive name
description: One line, used to decide relevance in future sessions
type: user|feedback|project|reference
---

Content here.
```

### When to Choose This Approach

- You want full control over what Claude sees
- You prefer deterministic recall (same facts load every session)
- You don't want background processes or API costs
- Your memory needs are human-scale (< 200 indexed items)

---

## Approach 2: learning-loop (Knowledge Vault with Quality Gates)

**Best for:** Researchers, knowledge workers, anyone building a compounding knowledge base across sessions.

**Philosophy:** A self-improving memory plugin that creates a knowledge vault from sessions. Quality gates, source verification, and explicit reflection triggers turn corrections into structured rules.

**Source:** [robinslange/learning-loop](https://github.com/robinslange/learning-loop)

### Setup Steps (Windows)

> **Note:** The one-line installer supports WSL x86_64. On native Windows, use the manual install path.

#### Option A: WSL (recommended for full compatibility)

```bash
# From WSL terminal in VS Code (select WSL from terminal dropdown)
curl -fsSL https://raw.githubusercontent.com/robinslange/learning-loop/main/install.sh | bash

# After install, open Claude Code and run:
/learning-loop:init
```

#### Option B: Native Windows (manual)

```powershell
# 1. Ensure Claude Code is installed
npm install -g @anthropic-ai/claude-code

# 2. Add marketplaces
claude plugin marketplace add obra/superpowers-marketplace
claude plugin marketplace add robinslange/learning-loop

# 3. Install plugins
claude plugin install episodic-memory@superpowers-marketplace
claude plugin install learning-loop@learning-loop-marketplace

# 4. Restart Claude Code, then run:
/learning-loop:init
```

### Vault Structure

```
your-vault\
  0-inbox\          # Rough captures, new ideas
  1-fleeting\       # Developing notes, partially sourced
  2-literature\     # External source captures
  3-permanent\      # Complete, sourced, linked, voiced
  4-projects\       # Project index notes
  5-maps\           # Synthesis and discovery maps
  _system\          # Persona and capture rules
  Excalidraw\       # Diagrams
```

### Key Commands

| Command | What it does |
|---------|-------------|
| `/learning-loop:discovery "topic"` | Research with web search + vault context |
| `/learning-loop:quick-note "insight"` | Capture to inbox without breaking flow |
| `/learning-loop:reflect` | End-of-session consolidation |
| `/learning-loop:verify` | Check note quality and source integrity |
| `/learning-loop:refresh "topic"` | See what you already know (no web research) |
| `/learning-loop:rewrite "old" "new"` | Retract a belief across vault + episodic history |
| `/learning-loop:gaps "topic"` | Surface thin ice, tensions, and blindspots |
| `/learning-loop:doctor` | Diagnose and fix your install |
| `/learning-loop:inbox` | Batch triage inbox notes, promote mature ones |
| `/learning-loop:help` | Show all commands |

### How It Works

1. **Retrieval fires before you ask.** Every session starts by recalling what you already know.
2. **Quality gates fire before promotion.** Captures earn their place, not just pile up.
3. **Source verification catches fabricated sources.** `/verify` checks every citation mechanically.
4. **Corrections propagate.** When a belief changes, `/rewrite` traces everything that depends on it.
5. **Episodic memory provides semantic recall.** Past conversations are searchable, not just stored.

### Lifecycle Hooks

The plugin wires into Claude Code's lifecycle:

- `SessionStart`: Loads retrieval context
- `PostToolUse (Write|Edit)`: Full vault enrichment (autolink, edge inference, reflect tracking)
- `PostToolUse (Task|Skill)`: Lightweight provenance tracking

### Dependencies

- **episodic-memory** (required): Provides semantic recall over past conversations
- **Node.js 22+**: For the plugin runtime
- **ll-search binary**: Installed automatically (WSL/Linux); may need source build on native Windows

### When to Choose This Approach

- You want a compounding knowledge system, not just memory
- You value source verification and quality gates
- You want semantic search over past sessions
- You're comfortable with a plugin that hooks into the runtime
- You want structured research workflows (`/discovery`, `/deep-research`)

---

## Comparison

| Dimension | claude-code-memory | learning-loop |
|-----------|--------------------|---------------|
| **Setup time** | 2 minutes | 3-5 minutes |
| **Dependencies** | Zero | episodic-memory plugin, Node.js 22+ |
| **Recall type** | Deterministic (full index loads every session) | Semantic (similarity search + quality scoring) |
| **Scale** | Human-scale (< 200 indexed items) | Vault-scale (thousands of notes) |
| **Automation** | Manual curation | Automated hooks + quality gates |
| **Source verification** | None (you verify) | Built-in (checks PMIDs, DOIs, citations) |
| **Cost** | Free | Free (local) or embedding API costs |
| **Inspect/edit** | Open in VS Code (`Ctrl+Click` the path) | Commands + vault files |
| **Best mental model** | A briefing doc you maintain | A second brain that compounds |
| **Correction propagation** | Manual update | Automated via `/rewrite` |
| **Offline capable** | Yes (always) | Yes (`LL_OFFLINE=1`) |
| **Windows support** | Full (just markdown files) | Full via WSL; native Windows needs manual install |

---

## Dual-File Feedback Loop (Lightweight Baseline)

This pattern works standalone or alongside either approach above. It's the minimum viable memory you can set up in 30 seconds for any new project.

### Structure

```
.claude\
+-- CLAUDE.md              # Master rules, build commands, and tech stack
+-- memory\
    +-- feedback.md        # Distilled user corrections and anti-patterns
    +-- architecture.md    # Active architectural decisions and state
```

### Setup (PowerShell in VS Code)

```powershell
# From your project root (Ctrl+` to open terminal)
New-Item -ItemType Directory -Force -Path .claude\memory

# Create the master instructions file
@"
# Project: <your-project-name>

## Tech Stack
- <list your stack here>

## Build Commands
- ``npm run dev`` -- start dev server
- ``npm test`` -- run tests
- ``npm run build`` -- production build

## Conventions
- <your coding conventions>

## Memory
On session start, read ``.claude/memory/feedback.md`` and ``.claude/memory/architecture.md`` for accumulated context.
"@ | Set-Content .claude\CLAUDE.md -Encoding UTF8

# Create the feedback file
@"
# Feedback and Corrections

Distilled user corrections, anti-patterns, and preferences learned across sessions.

---

<!-- Add entries below as they come up -->
"@ | Set-Content .claude\memory\feedback.md -Encoding UTF8

# Create the architecture decisions file
@"
# Architecture Decisions

Active architectural decisions, patterns in use, and current system state.

---

<!-- Add entries below as decisions are made -->
"@ | Set-Content .claude\memory\architecture.md -Encoding UTF8
```

### End-of-Session Distillation

Run this at the end of heavy sessions to auto-capture learnings:

```powershell
# From VS Code terminal
claude -p "Review our recent conversation, extract any newly established debugging patterns or corrected user preferences, and append them concisely to .claude/memory/feedback.md."
```

For architecture updates:

```powershell
claude -p "Review our recent conversation, extract any new architectural decisions or state changes, and append them concisely to .claude/memory/architecture.md."
```

### Tips for the Feedback Loop

1. **Run distillation before ending any session where you corrected Claude.** Those corrections are the highest-value memories.
2. **Keep entries concise.** One line per pattern or correction. Don't write paragraphs.
3. **Date your entries.** Prefix with `YYYY-MM-DD` so stale ones are easy to spot.
4. **Prune quarterly.** If a pattern hasn't been relevant in 3 months, archive or delete it.
5. **Version control these files.** They're small, diffable, and your future self will thank you.
6. **Use VS Code's timeline view** (`Right-click file > Open Timeline`) to see how your memory evolves over time.

### Example feedback.md After a Few Sessions

```markdown
# Feedback and Corrections

2026-08-21 -- Don't use `any` types in TypeScript, always define interfaces
2026-08-21 -- Prefer named exports over default exports
2026-08-22 -- When fixing tests, run the full suite, not just the failing one
2026-08-23 -- Don't suggest database migrations without checking current schema first
2026-08-25 -- Error messages should include the operation that failed, not just the error
```

---

## Starting a New Project Checklist

```powershell
# 1. Create project (VS Code terminal, PowerShell)
mkdir my-new-project
cd my-new-project
git init

# 2. Open in VS Code
code .

# 3. Set up the dual-file baseline (always, takes 30 seconds)
New-Item -ItemType Directory -Force -Path .claude\memory
# Create CLAUDE.md, feedback.md, architecture.md (see commands above)

# 4. Choose your memory approach:

# Option A: Keep it simple (claude-code-memory templates)
git clone https://github.com/LuciferForge/claude-code-memory.git $env:TEMP\ccm
python $env:TEMP\ccm\setup.py --project .
Remove-Item -Recurse -Force $env:TEMP\ccm

# Option B: Full knowledge vault (learning-loop)
# Use WSL terminal in VS Code, run the one-liner from Approach 2, then /learning-loop:init

# 5. Add to .gitignore (if using local memory files you don't want committed)
Add-Content .gitignore ".claude/memory/"

# 6. Start Claude Code
claude
```

---

## VS Code Tips for Working with Memory Files

| Action | Shortcut / Command |
|--------|-------------------|
| Open terminal | `Ctrl+\`` |
| Switch terminal shell (PowerShell/Git Bash/WSL) | Click dropdown arrow in terminal panel |
| Quick-open any memory file | `Ctrl+P` then type `feedback.md` |
| Preview markdown | `Ctrl+Shift+V` |
| Side-by-side preview | `Ctrl+K V` |
| View file history (Git) | Right-click file > `Open Timeline` |
| Search across memory files | `Ctrl+Shift+F` then scope to `.claude/` folder |
| Multi-cursor edit (bulk date prefixes) | `Ctrl+Alt+Down` to add cursors |
| Fold sections in large files | `Ctrl+Shift+[` |

### Recommended VS Code Settings for Memory Files

Add to `.vscode/settings.json` in your project:

```json
{
  "files.associations": {
    ".claude/**/*.md": "markdown"
  },
  "[markdown]": {
    "editor.wordWrap": "on",
    "editor.quickSuggestions": false
  },
  "search.exclude": {
    ".claude/memory/**": false
  }
}
```

This ensures memory files get proper markdown highlighting and word wrap, and are always searchable.

---

## References

| Repo | What it provides |
|------|------------------|
| [LuciferForge/claude-code-memory](https://github.com/LuciferForge/claude-code-memory) | Zero-dep markdown memory templates, setup script, frontmatter schema |
| [robinslange/learning-loop](https://github.com/robinslange/learning-loop) | Knowledge vault plugin with quality gates, source verification, episodic recall |
| [Digital-Process-Tools/claude-remember](https://github.com/Digital-Process-Tools/claude-remember) | Automated memory plugin (hooks + Haiku compression, continuous recall). Not covered in this guide but worth evaluating for automated session persistence. |
| [davila7/claude-code-templates (git worktrees skill)](https://github.com/davila7/claude-code-templates/blob/main/cli-tool/components/skills/development/using-git-worktrees/SKILL.md) | Git worktree isolation for parallel feature work. Not covered here but recommended for multi-branch workflows. |

---

## License

MIT
