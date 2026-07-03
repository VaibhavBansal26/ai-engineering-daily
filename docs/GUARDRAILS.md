# Guardrails & Evaluation Harness

Rules every agent in this automation system must follow.

## Hard rules (never violated)

1. **Human approval before anything public** — no commit, article, post, or video is published without explicit approval from Vaibhav.
2. **No fabricated facts** — every technical claim must be verifiable; benchmarks and stats must cite a source.
3. **No plagiarism** — content is original; quoted material is attributed. Never reproduce copyrighted articles or code without license compliance.
4. **No secrets** — never commit API keys, tokens, or personal data. Scan every commit for credential patterns.
5. **No duplication** — check the Notion pipeline and this repo's learning log before creating content; skip or pivot if the topic was already covered.

## Quality bar (checked before requesting approval)

- Code examples must run (tested in sandbox before inclusion)
- Explanations target a practicing engineer, not fluff
- Each piece teaches at least one concrete, new thing
- LinkedIn/Medium posts include a hook, substance, and a takeaway — no engagement bait
- Videos: accurate captions, no misleading thumbnails

## Evaluation harness (weekly guardrails agent)

Every Sunday the evaluator agent:

1. Pulls the week's content from the Notion pipeline
2. Scores each item 1–10 on: accuracy, originality, clarity, engagement
3. Flags duplicates, factual issues, or guardrail violations
4. Writes an eval report and updates `Eval Score` in the pipeline
5. Suggests corrections for anything scoring below 6

## Incident response

If published content is found to be wrong: correct it publicly within 24h, note the correction in the learning log.
