---
name: deep-code-read
description: Use when you want to deeply understand an unfamiliar codebase and generate reusable cognitive skills from it, by providing a local path or GitHub URL
version: 1.0.0
homepage: https://github.com/CiferaTeam/deep-code-reader
---

# Deep Code Reader

Systematically read and understand a codebase, producing a set of verified cognitive skills that capture deep knowledge — module capabilities, design logic, data structures, state flow, and modification guides.

The core mechanism: a closed-book exam verification loop ensures generated skills are genuinely comprehensive, not shallow summaries.

**Skill authoring:** Use the host agent's built-in skill creator or authoring guidance when available. Otherwise, create skills directly using the templates in this workflow: a `SKILL.md` with YAML `name` and `description`, a focused Markdown body, and relative links to supporting files. Apply this to both module skills and the global index. The ABC loop below validates the generated knowledge.

## Usage

```
/deep-code-read <source> <output-dir>
```

- **source**: local path (e.g., `./path/to/repo`) or GitHub URL (e.g., `https://github.com/org/repo`)
- **output-dir**: where generated skills are written (e.g., your platform's skills directory)

## Full Flow

You MUST follow these phases in order. Track progress across modules using your platform's task/todo tracking mechanism.

### Phase 1: Prepare

1. Determine the project name:
   - Local path → directory name
   - GitHub URL → repo name
2. If source is a URL:
   - Clone to `{output-dir}/{project-name}/`
   - If the directory already exists, skip cloning and use it
3. If source is a local path:
   - Verify the path exists and is a git repo
   - Use it directly (read-only — do NOT modify any files in the source repo)
4. Detect version:
   - Run `git tag --list` in the source repo
   - If tags exist, sort with semver-aware ordering (handle `v` prefix), recommend the latest
   - If no reasonable tags, recommend `main` or `master` branch
5. **PAUSE — present recommendation to user:**
   > "Detected the following tags/branches: [list]. I recommend tracking `{recommended}`. Confirm or specify a different target."
6. Checkout the confirmed ref

### Phase 2: Scan

1. Scan the source repo directory structure
2. Identify module boundaries using heuristics:
   - Top-level directories under `src/`, `lib/`, `pkg/`, `packages/`, or project root
   - Language-specific patterns: Python packages (`__init__.py`), Go packages, Node packages (`package.json`), etc.
   - Look for existing module documentation or manifest files
3. Analyze import/dependency relationships between modules
4. **PAUSE — present module list and dependency graph to user:**
   > "Found the following modules: [list with one-line descriptions]. Select which modules to deep-read (or 'all')."
5. Record the user's selection — one task per selected module

### Phase 3: Deep Read (Agent A)

For each selected module, dispatch a subagent with the prompt template from `agent-a-prompt.md`.

**Subagent dispatch parameters:**
- `prompt`: rendered `agent-a-prompt.md` with variables filled in
- `description`: "Deep read {module-name}"

**Variables to fill in the prompt:**
- `{source-dir}`: path to the source repo
- `{module-dir}`: path to the specific module within the source repo
- `{output-dir}`: the skill output directory
- `{project-name}`: extracted project name
- `{module-name}`: the module name
- `{ref}`: the tracked tag/branch

After Agent A completes, verify the skill files were written to `{output-dir}/{project-name}-dr-{module-name}/`. Update the module's task status.

### Phase 4: Verify (ABC Loop)

For each module that has generated skills, run the verification cycle:

**Step 1 — Agent B (question generation):**

Dispatch a subagent with `agent-b-prompt.md`, using a lightweight/smaller model (e.g., Haiku-class).

**Subagent dispatch parameters:**
- `prompt`: rendered `agent-b-prompt.md`
- `model`: a smaller, cheaper model — the weaker the better (if it catches gaps, those gaps are real)
- `description`: "Generate questions for {module-name}"

**Variables:**
- `{source-dir}`, `{module-dir}`, `{module-name}`
- `{previous_questions}`: empty string for the first round

Agent B returns two sets:
- Verification questions with answer keys (JSON array)
- Recommended questions for user (JSON array)

Save the recommended questions (keep in context for Phase 6). Accumulate all verification questions asked so far across rounds.

**Step 2 — Agent C (closed-book answer):**

Dispatch a subagent with `agent-c-prompt.md`.

**Subagent dispatch parameters:**
- `prompt`: rendered `agent-c-prompt.md` with verification questions embedded
- `description`: "Verify skills for {module-name}"

**Variables:**
- `{skill-dir}`: `{output-dir}/{project-name}-dr-{module-name}/`
- `{questions}`: the verification questions from Agent B (without answer keys)

Agent C returns answers to each question.

**Step 3 — Evaluate:**

For each question, check Agent C's answer against Agent B's `required_facts` list:
- An answer PASSES if it covers ALL required facts (exact match or semantic equivalent)
- An answer FAILS if it misses any required fact
- This is an objective check, not a subjective judgment

**Step 4 — Loop or proceed:**

**HARD RULE: You MUST continue looping until 100% of verification questions pass OR you have completed exactly 3 rounds. There is NO early exit. A pass rate of 99% is still a failure — loop again.**

- 100% pass → module verified, update task, move to next module
- ANY question fails (even one) → you MUST continue to the next round:
  1. Collect failed questions with: the question, B's answer key, C's failed answer
  2. Feed these back to Agent A: dispatch again with supplementary instructions to improve the skill based on the gaps
  3. Re-run Agent B and Agent C, passing ALL previous questions (from all rounds) as `{previous_questions}` so B generates new questions instead of repeating old ones
  4. Evaluate again — repeat until 100% or 3 rounds completed
- **After exactly 3 rounds with failures remaining** → show the unresolved questions and pass rates to the user for judgment. Do NOT silently move on.

**Do NOT rationalize stopping early.** "Good enough", "most questions passed", "diminishing returns" are not valid reasons to skip a round. The loop exists to catch gaps — use all 3 rounds if needed.

### Phase 5: Generate Global Index

After all modules are verified, generate `{output-dir}/{project-name}-dr/SKILL.md`:

```yaml
---
name: {project-name}-dr
description: Use when working with {project-name} codebase — provides comprehensive module knowledge, design logic, and modification guides (generated from {ref})
---
```

Content must include:
- Repo source (GitHub URL if applicable, or local path)
- Version: tag or commit hash
- Tracked branch
- Generation timestamp
- Each module's one-line purpose (from the module skills)
- Inter-module dependency relationships (from Phase 2 scan)
- Cross-module scenario entry guides: for common operations that span multiple modules, describe which modules are involved and in what order

To generate cross-module scenarios, read ALL the module skills and synthesize typical user workflows.

### Phase 6: User Acceptance

Present the recommended questions collected from Phase 4:

> "Skills generated and verified. Here are some questions you might want to test:
> [list recommended questions]
>
> Feel free to ask any question about {project-name}. I'll answer using ONLY the generated skills."

When answering user questions in this phase:
- Read ONLY the generated skill files in `{output-dir}/{project-name}-dr*/`
- Do NOT read source code
- If you cannot answer a question from the skills alone, say so honestly — this indicates a gap

Continue until the user is satisfied or decides to end the session.

### Phase 7: Cleanup

If the source was cloned from a URL (i.e., `{output-dir}/{project-name}/` was created in Phase 1):

> "Skills are ready. The cloned source code is at `{output-dir}/{project-name}/`. Want me to delete it to save disk space, or keep it for reference?"

- User says delete → remove the cloned directory
- User says keep → leave it as is

Skip this phase if the source was a local path (we never cloned anything).

## Key Rules

- **Never modify source code** — the source repo is read-only throughout
- **Agent isolation is critical** — each agent's prompt strictly defines what it can read
- **Skills must be self-sufficient** — the verification loop exists to ensure this
- **Track progress** — every module is a task, updated as it progresses through phases
- **Skill formatting** — follow available native authoring guidance or the included templates; check frontmatter and supporting-file links before verification
