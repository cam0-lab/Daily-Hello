# Talmeez — Architecture Document

Last updated: 19 November 2026. Full AI design: `docs/ai-native.md`. Product rules: `docs/product-rules.md`.

## What the product does
Parents in Lebanon can't easily find a tutor they trust for their child's exact class and exam, so they ask in WhatsApp groups. On Talmeez a parent types one sentence and gets tutors who teach that class, with a reason for each match. The parent messages the tutor on Talmeez. Tutors pay a subscription after 2 free leads.

## Numbers today (from the database, 19 Nov)
- 113 tutors and 11 parents signed up, all for free.
- Nobody pays yet.
- The old site had 3 session requests in total (April, May and September 2026), none since.
- The new site, which counts leads, is not live yet, so counted leads = 0.

## How it is built
```
 web (Next.js) ─┐
 Expo app ──────┼──► brain (Edge Function, typed tools) ──► Postgres (Supabase)
 admin ─────────┤            │
 MCP for AIs ───┘            ▼
                       Claude API ──► ai_runs log
```
- **Brain:** one server function with typed tools (`parse_query`, `search_tutors`, `explain_match`, `classify_message`, `suggest_replies`, `translate`, `rewrite_bio`, `analyse_week`). The site, the app, admin and other AIs all call the same tools, so behaviour is the same everywhere.
- **Models:** Haiku for parsing, classifying and translating; Sonnet for match reasons, bio rewrites and the weekly analysis.
- **Fallback:** a keyword parser and regex rules run on every search and message. If the AI is off, over budget or wrong, the site still works.
- **Database:** Supabase (Postgres) with row-level security on every table.

## The three key processes and their level of AI
I kept the three steps from Daily Hello (call, understand and flag, report) and ran them on Talmeez.

| Step | Daily Hello | Talmeez | Level of AI | Why |
|---|---|---|---|---|
| 1. Listen | AI phoned the parent and talked | Parent types one sentence; Claude reads subject, class, town and price, shown as editable chips | AI does it | Many searches, low risk, the parent can correct it |
| 2. Understand and flag | AI compared answers with the parent's normal: green, amber, red | AI flags shared phone numbers, an agreed booking, a child's name or street (removed), searches with no tutor | AI flags, never blocks | A wrong block loses a lead, so a person can always overrule |
| 3. Report | AI wrote a 3-line summary for the family | Weekly AI report with what changed, where people drop off, missing tutors, and up to 3 proposed changes with evidence | AI proposes, I decide | It touches ranking, money and trust |

Tutor ID checks stay human only.

## Data
- Each search is logged: language, what was understood, which parser was used, results shown. Clicks and messages link back to the search.
- Each message gets a signal row (booking, contact shared, language), not the text.
- Nightly rollups: funnel (visit → search → profile → message → lead → booking → review), demand gaps (searched with fewer than 3 tutors), tutor stats, parser quality.
- The weekly analyst only sees rollups and anonymised samples, never names or message text.

## The number I track: weekly leads
A lead = the parent sends at least 2 messages and the tutor replies, counted once per parent and tutor. It is stored when it happens (`lead_counted_at`) and it is also what uses a tutor's free leads, so fake parents who get no answer can't burn them. It doesn't need a booking. Bookings are tracked separately.

## Evals
- `evals/search.jsonl`: 170 sentences in Arabic, French, English, Arabizi and mixed, with the expected subject, class, area and price.
- `evals/messages.jsonl`: 108 messages, including tricky ones ("I'll think and tell you Tuesday", prices, dates).
- Pass thresholds: 90% parse accuracy, 95% contact detection, 85% booking detection. A change that drops below fails.
- Session 8: 15 cases (10 normal, 5 attacks) through the real code. Before: 9/10 and 4/5. The one real failure broke my red line: a child's full name and street were saved in the search log. I wrote a fix that removes names and streets before anything is logged, plus a database check. After: 5/5 attacks, nothing that worked before broke. The fix is tested but not merged into the live code yet.

## Red line and guardrails
- **Red line:** no child's name or address is stored. The child's name is never asked at sign-up.
- AI never sends messages, publishes or hides reviews, approves IDs, changes prices or merges code.
- Tutor bios and messages are treated as data, never as instructions to the AI.
- Monthly AI budget: warning at 80%, fallbacks at 100%.
- Everything the AI does is logged in `ai_runs`; everything an admin looks at is logged in `admin_access_log`.

## Decision log
| Date | Decision | Why |
|---|---|---|
| End of Oct | Switched the course project from Daily Hello to Talmeez (agreed in class) | Real tutors, real data, can test with real users this term; Daily Hello had 10+ AI call competitors and no real family test |
| 27 Oct | MVP scope: conversational search, messaging that notices, findable by other AIs, learning loop | Search is the core; the rest feeds the learning loop |
| 27 Oct | Keyword parser always runs next to the AI | Site works if the AI fails; also a sanity check |
| 27 Oct | AI flags messages but never blocks | A wrong block loses a lead |
| 27 Oct | No session-replay tools (Hotjar, Clarity) | They would record parents typing about their children |
| Nov | Lead counted only after 2 parent messages and a tutor reply | Fake parents can't use up a tutor's free leads |
| Nov | Remove names and streets before logging (Session 8) | Red line broke in test case 14 |
| Nov | One main judge for the AI grader (Session 9) | Keeps the judging consistent |

## Next
1. Put the new site live so leads are counted automatically.
2. Get real parents searching: recruit through schools and parents' WhatsApp groups.
3. 40 survey answers before Demo Day.
