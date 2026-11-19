# Evals
- `search.jsonl` — one JSON per line: {"q": "...", "lang": "ar|fr|en|arabizi|mixed", "expect": {"subject": [...], "program": "LB", "class_code": "LB-EB9", "area": "jounieh-keserwan", ...}}. Only the fields listed in `expect` are scored; use `null` / `[]` to assert "nothing found". 130+ cases across AR, FR, EN, Arabizi and mixed, including negatives (a subject we do not list, a sentence with no filters).
- `messages.jsonl` — {"text": "...", "lang": "...", "expect": {"booking_signal": true|false, "contact_shared": true|false}, "tricky": true?}. `tricky` marks negatives and look-alikes ("I'll think and tell you Tuesday", dates, prices, order numbers, spoken numbers).
- Thresholds (also in `settings.eval_thresholds`): parse field accuracy ≥ 90%, contact detection ≥ 95%, booking detection ≥ 85%.
- Codes come from `packages/catalog/catalog.json` (classes, exams, subjects, areas). `pnpm eval` rejects a case whose expected codes do not exist, or whose class and program disagree.
- New cases from real corrections are added weekly as candidates and count only once I approve them.

## Running
`pnpm eval` runs offline with no API key:
1. **AI off** — the keyword parser and the regex message classifier, called through the same tool registry the site uses. This is the gate that must pass in CI.
2. **Oracle model** — a stand-in model that answers with the expected values. It proves the AI path (JSON tool schema, zod validation, merge with the keyword parser, contact redaction) accepts a perfect answer. It says nothing about model quality.
3. **Failing model** — a model that always errors. Scores must equal the fallback, and every failure must be logged.

To score the real model:
- `ANTHROPIC_API_KEY=... pnpm eval --live --record evals/recordings/<date>.json` runs against Claude and saves every request/response.
- `pnpm eval --replay evals/recordings/<date>.json` re-scores that run offline (CI), and fails if the prompts changed so the recording no longer matches.
- Never commit a key; recordings contain only eval sentences (no user data).
