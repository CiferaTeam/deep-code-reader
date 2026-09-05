# Deep Code Reader

[中文版](README_CN.md)

**Turn codebases into reusable AI knowledge skills, grounded in source evidence and tested for querying and reference.**

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
              Regression pass → fresh final exam → next module
              Either exam fails → repair → cumulative re-test
                          ↓
              Global index + Q&A acceptance with you
```

The tool scans the repo structure and dependencies, then lets you choose modules to read. Each module produces skills tested through **cumulative closed-book regression and a fresh final exam**. Source evidence is checked before grading. Passing records coverage of the tested questions at a specific source version; remaining gaps are reported after at most three rounds.

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
- **Agent B** (model selected for accurate code reading): reads source code WITHOUT seeing skills, generates questions with answer keys and source evidence for every required fact
- **Agent C** (primary model): reads ONLY skill files, takes the exam without source code access

The coordinator checks answer keys against cited source code before grading. Each round tests the **entire accumulated question bank**, including earlier successes, plus new questions. Answers must cover the required facts without materially false or contradictory claims.

After regression passes, the documents stay frozen while a fresh B generates an **independent final exam** about mechanisms, limitations, and practical reference scenarios. A fresh C receives only the skills and question text/IDs. Both exams must pass for the same document version. Failed final exams join the regression bank before repairs; the next final exam uses new questions.

Each module has at most 3 rounds, including the initial candidate. Unresolved gaps or evidence disputes remain visible and prevent verification. Records include the source commit, document hashes, source evidence, separate exam scores, and the isolation level. Runs relying solely on prompt-based isolation are marked `partial_validated`; `verified` requires scoped access or checked access logs. Passing establishes the tested coverage, rather than exhaustive understanding.

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
