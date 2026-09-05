# Deep Code Reader

[中文版](README_CN.md)

**Turn any codebase into verified, reusable AI skills — not summaries, not RAG, but genuine comprehension.**

---

## The Problem

When LLMs read code, they default to skimming and summarizing. Ask them to "understand" a repo, and you'll get a high-level overview that falls apart the moment you ask a specific question. Next time you ask, they go back to searching from scratch.

## The Solution

Deep Code Reader produces **verified cognitive skills** — structured knowledge documents that an AI can load and immediately operate at the level of someone who has actually read the code.

### How it works

```
Scan repo → identify modules & dependencies → you pick what to read
                          ↓
         For each module: deep read → generate skill
                          ↓
              Closed-book exam verification (ABC loop)
                          ↓
         Agent B (reads code, no skill) → exam questions + answer keys
         Agent C (reads skill, no code) → takes the exam
                          ↓
              Pass? → Next module / Fail? → improve skill → re-exam
                          ↓
              Global index + Q&A acceptance with you
```

The tool first scans the repo structure to map out modules and their dependencies, then lets you choose which modules to deep-read. Each module goes through a thorough reading phase followed by a **closed-book exam** — if Agent C can answer detailed questions using ONLY the generated skills, without touching source code, the skills are genuinely comprehensive. If not, they get improved until they are.

## Let Your Tokens Learn While You Sleep

Most subscription plans include ~5 hours of daily AI compute. Much of it goes unused overnight. Deep Code Reader turns that idle quota into accumulated knowledge.

Fire it off before bed, wake up to a fully analyzed repo with verified skills ready to load. The more repos you run, the more your AI knows — compounding overnight, zero extra cost.

## Quick Start

### Install

Add `deep-code-read` to your agent's skills directory:

```bash
git clone https://github.com/CiferaTeam/deep-code-reader.git
cp -r deep-code-reader/deep-code-read ~/.claude/skills/
```

**Skill authoring:** Uses the host agent's built-in skill creator or authoring guidance when available; otherwise, the included templates provide the format for direct generation.

### Use

```bash
# From a GitHub URL
/deep-code-read https://github.com/example/project ~/.claude/skills/

# From a local repo
/deep-code-read ./path/to/project ~/.claude/skills/
```

That's it. The tool handles everything automatically, pausing only twice for your input:

1. **Confirm version** — which tag/branch to analyze
2. **Select modules** — which parts to deep-read

## What You Get

```
~/.claude/skills/
  project/                      # Cloned source (URL only)
  project-dr/                   # Global index skill
    SKILL.md
  project-dr-auth/              # Module skill
    SKILL.md
    reference.md                # Optional for complex modules
  project-dr-routing/
    SKILL.md
  ...
```

### Each module skill covers 5 dimensions:

| Dimension | What it captures |
|---|---|
| **Purpose & Capabilities** | What the module does, its public API, function signatures |
| **Core Design Logic** | WHY it's built this way, key architectural decisions |
| **Data Structures** | Key types, interfaces, and their relationships |
| **State Flow** | How data flows, entry points, error paths |
| **Modification Guide** | "To change X, modify these files" |

### The global index skill includes:

- Repo source, version, tracked branch
- All modules with one-line descriptions
- Inter-module dependency map
- Cross-module scenario guides

## The ABC Verification Loop

This is what makes deep-code-reader different from "just another code summarizer":

- **Agent A** (primary model): reads source code, generates skill files
- **Agent B** (lightweight model): reads source code WITHOUT seeing skills, generates exam questions with answer keys and required facts
- **Agent C** (primary model): reads ONLY skill files, takes the exam without source code access

Each iteration, B adds **new questions** covering untested areas — so A can't just "teach to the test". Max 3 rounds per module; unresolved gaps are surfaced to you for judgment.

## After Generation

The tool enters a **Q&A acceptance phase**:

- Ask anything about the codebase — AI answers using ONLY the generated skills
- Recommended deep questions from Agent B are provided if you're not sure what to ask
- If the AI can't answer from skills alone, that's an honest signal of a gap

## Platform Support

Deep Code Reader is platform-agnostic. It works with any AI coding agent that supports:

- Skill/instruction file loading
- Subagent dispatching
- File system read/write

Tested with Claude Code. Should work with Codex, Gemini CLI, and other skill-compatible agents.

## License

MIT
