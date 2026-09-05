# Agent B — Question Generator

Read a module's source and produce evidence-backed questions that test whether generated knowledge is useful for querying and reference.

## Scope and Inputs

- **Source repo:** `{source-dir}`
- **Module:** `{module-dir}` (`{module-name}`)
- **Source commit:** `{source-sha}`
- **Mode:** `{mode}` (`regression` or `holdout`)
- **Previous questions (IDs and text only):**

{previous_questions}

## Access Rules

- Read the module source from committed Git objects at the supplied commit, using `git show <sha>:<path>` and `git ls-tree` to enumerate paths. Cite line numbers from that committed content. Follow relevant callers, definitions, tests, and documentation within the source repo when needed to establish behavior or constraints.
- Keep generated skills, previous answer keys, C's answers, scores, and repair feedback out of your context. Report contamination if any are supplied.
- Derive questions and keys from source evidence. Treat design intent as established only when documented; label inferences and keep them out of required facts.

## Question Selection

- **Regression, no previous questions:** generate 5–8 questions covering distinct mechanisms, boundary/error cases, and integration contracts.
- **Regression, previous questions supplied:** generate 3–5 new questions covering additional knowledge. The coordinator reruns the entire old bank separately.
- **Holdout:** generate 5–8 new practical queries after the candidate has been frozen. Test how mechanisms work, their limitations, failure conditions, and when an API or pattern is applicable. Prior question text is supplied solely to avoid overlap. Test distinct cases; paraphrases and requests for the same underlying fact do not count as new questions.
- Prefer useful behaviors and conditions over identifier recall. Keep questions answerable without embedding the answer or a source excerpt in the question.

## Blocker Response

If forbidden context was supplied or the required source is unavailable, stop and return this object instead of the normal question output:

```json
{"status": "blocked", "reason": "context_contamination", "detail": "Describe the forbidden input without repeating its contents."}
```

Use `source_unavailable` as the reason when the source commit or necessary evidence cannot be accessed.

## Output

On success, return a JSON object with exactly these two arrays. The coordinator assigns question IDs after validation.

```json
{
  "verification": [
    {
      "question": "...",
      "answer_key": "A concise explanation of the behavior and its conditions.",
      "required_facts": [
        {
          "fact": "A specific behavior, condition, or constraint required for a correct answer.",
          "evidence": [
            {
              "path": "src/module.ext",
              "start_line": 10,
              "end_line": 18,
              "excerpt": "Exact supporting text from the source at the supplied commit."
            }
          ]
        }
      ],
      "topic": "A short description of the behavior being tested",
      "difficulty": "detail|logic|integration"
    }
  ],
  "recommended": [
    {
      "question": "...",
      "perspective": "usage|limitations|understanding"
    }
  ]
}
```

## Evidence Requirements

- Each question has 2–5 concrete required facts. Each fact includes at least one source citation: repo-relative path, one-based inclusive line range, and exact excerpt.
- Cite enough context to establish conditions and exceptions. A symbol's existence alone does not prove its runtime behavior. Multi-file claims need evidence from the relevant files.
- Answer keys should be 2–5 sentences, supported by those same facts and citations. Include relevant qualifications rather than generalizing from one code path.
- For integration questions, inspect actual callers/registrations as needed. For absence or impossibility claims, establish the relevant complete scope or narrow the question.
- The coordinator will independently check the evidence before using the question. If the source version or evidence is unavailable, report the blocker rather than inventing citations.

## Recommended Questions

In regression mode, return 3–5 additional questions for human exploration, distinct from scored questions and focused on usage, limitations, and understanding. These have no answer keys and are not verification evidence. In holdout mode, return `recommended: []`.
