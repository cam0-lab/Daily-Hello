# Talmeez AI-native design (Oct 2026)

MVP scope (my decision, 27 Oct 2026):
- **In:** conversational search (core) · messaging that notices · findable by other AIs · the learning loop (analytics + weekly analyst + proposals)
- **Later:** tutor sign-up by talking · admin/trust copilot (review pre-checks, photo vs ID flag, plain-language admin queries)

```
 web (Next.js) ─┐
 Expo app ──────┼──►  brain (Edge Function, typed tools)  ──► Postgres (+pgvector) / Storage
 admin ─────────┤            │                                   ▲
 MCP for AIs ───┘            ▼                                   │
                        Claude API ──► ai_runs log               │
                                                                 │
 every action ──► events ──► nightly rollups ──► weekly analyst ─┴─► insights + GitHub issues ──► I approve ──► Claude Code ──► PR + evals ──► ship
```

## 1. The brain
One Edge Function `brain` exposing typed tools. Each tool = zod input schema + permission check + implementation. The UI, Ask Talmeez, admin and MCP all call these, so behaviour is identical everywhere.

| Tool | Who can call | Does |
|---|---|---|
| `parse_query` | anyone | Sentence (AR/FR/EN/Arabizi/mixed) → `{subject[], program, class_code, track, exam_code, area, max_price, mode(home/online), language}` + confidence per field |
| `search_tutors` | anyone | Structured filters → exact matches, then near matches, ranked (section 2) |
| `get_tutor` | anyone | Public profile fields only |
| `explain_match` | anyone | One-line reason per tutor, in the visitor's language, says when it's another program |
| `classify_message` | system | Message → `{booking_signal, proposed_time, contact_shared, language}` |
| `suggest_replies` | tutor (own chats) | 2–3 short reply drafts in the parent's language |
| `translate` | system | Text → the 2 other languages, cached in `content_translations` |
| `rewrite_bio` | tutor (own) | Bio rewrite, keeps facts, no invented claims |
| `analyse_week` | cron only | Learning loop (section 5) |

Models: classification/parsing/translation = Haiku; reasons, bio rewrite, weekly analysis = Sonnet. Names in `settings`, swappable without a deploy. Prompt caching on the static parts (class catalogue, equivalence rules).

## 2. Conversational search (core)
1. Hero box submit → `parse_query`. Keyword parser (prototype logic) runs in parallel as the fallback and as a sanity check.
2. Parsed fields show as **editable chips** above results ("Maths · EB9/Brevet · Jounieh"). If the parent edits a chip, log `search.parse_corrected` with before/after → that's a free label for the evals.
3. If nothing parses → open Ask Talmeez with the message already sent.
4. `search_tutors`:
   - **Filter:** exact = same program + stage overlap + track compatible (or same exam code); near = both school programs + stage overlap. Area/distance/price as filters.
   - **Rank** inside each group: `score = w_rel·relevance + w_q·quality + w_new·new_tutor_boost`
     - relevance: subject/class fit + embedding similarity between the query and the tutor's bio/subjects (pgvector, Voyage multilingual)
     - quality: reply rate, median reply time, booking confirmations, profile completeness (from `tutor_stats`, nightly)
     - new_tutor_boost: decays over the first 30 days so new tutors get seen
     - weights in `settings.ranking`; only changed through an approved proposal
5. `explain_match` adds the one-line reason on the top results (lazy, after first paint).
6. Every search writes one `search_logs` row (query text, language, parsed filters, parser used, result ids shown, latency). Clicks and messages link back to it via `search_id`.

## 3. Messaging that notices
On each new message (DB trigger → queue → `classify_message`, async, never blocks sending):
- **Contact shared:** regex first (Lebanese numbers, Arabic-Indic digits, spaced/dotted), AI for written-out numbers ("seb3a ... "). Logs the handoff. Never blocks.
- **Booking signal** ("see you Tuesday at 5", "ok mnbalesh l sabet"): show the "Did you book?" prompt to the parent right away instead of waiting 48h. The 48h prompt stays as the fallback.
- **Translation:** if parent and tutor languages differ, show the other side's messages translated, with "Show original".
- **Reply suggestions (tutor side):** 2–3 drafts under the box, in the parent's language. Tutor taps to edit; never auto-sent.
- Stored result = `message_signals` row (enums + confidence), not the text. Privacy policy must say messages are processed automatically for these features.

## 4. Findable by other AIs
- `/llms.txt`: what Talmeez is, how to search, links to subject × area pages.
- schema.org JSON-LD on profiles (Person + Service + offers price; `aggregateRating` only from real reviews).
- Public read-only MCP server at `/api/mcp` (Vercel MCP adapter → brain): `search_tutors`, `get_tutor`. Public fields only, no contact data, rate-limited per IP, results link to the Talmeez profile to message.
- Log visits/searches from AI agents as `source=ai_agent` (user agent + MCP calls) so we can see if it brings parents.

## 5. The learning loop (how the site and app improve)
**Collect** — our own `events` table (source of truth; trackers in §8 are for ads and marketing reports only), written server-side (and a tiny client helper that posts to our own endpoint). Same events from the Expo app later.
Key events: `page.view`, `search.submitted`, `search.parse_corrected`, `search.zero_results`, `results.filter_changed`, `tutor.card_clicked`, `profile.viewed`, `message.sent`, `lead.counted`, `contact.shared`, `booking.prompt_shown`, `booking.confirmed`, `review.posted`, `signup.step_completed`, `signup.abandoned`, `profile.updated`, `paywall.shown`, `subscription.activated`, `ask.opened`, `ai.fallback_used`, `perf.slow_page`.

**Roll up** — nightly pg_cron:
- `funnel_daily` (visit → search → profile → message → lead → booking → review, by language/device/source)
- `demand_gaps` (subject × class × area searched with < 3 tutors / zero results)
- `tutor_stats` (reply rate, reply time, leads, bookings, completeness)
- `parser_quality` (correction rate, fallback rate, by language)

**Analyse** — weekly (Mon 07:00 Beirut) `analyse_week` with Sonnet. It gets ONLY rollups and anonymised samples (no names, no message text) through read-only SQL tools. It writes an `insights` row shown in admin → Insights and emailed to me:
1. What changed this week (funnel numbers vs last week)
2. Where people drop off, and the likely reason
3. Demand gaps → tutor recruitment targets ("IGCSE Chemistry in Zahle: 14 searches, 0 tutors")
4. AI quality: worst-parsed queries, booking detector misses
5. Up to 3 **proposals**, each with evidence, expected effect, and a "done when"

**Act** — tiered:
| Tier | Examples | How |
|---|---|---|
| Automatic | Add corrected queries to `eval_cases` as candidates · refresh tutor coaching tips · refresh demand-gap list | No approval needed; reversible, user-invisible or suggestion-only |
| Approve in admin | Ranking weight changes · start an A/B test · send recruitment targets | One click in admin → applied via `settings` / `experiments` |
| Approve as code | UI/copy/flow changes | Proposal becomes a GitHub issue (`ai-proposal` label). I approve → Claude Code builds on a branch → Vercel preview + evals → I merge |

**Tutor coaching** (the app improves for tutors too): dashboard tips from real data, e.g. "Parents in Kaslik searched SG Physics 9 times this month. You teach it but your profile doesn't list S3 SG." Tutor chooses; nothing changes on its own.

**Experiments** — `experiments` table + server-side assignment (stable per visitor). Analyst reads results after the minimum sample set in the experiment; says "no clear winner" when that's the truth.

## 6. Evals (quality gate)
- `evals/search.jsonl`: ≥ 50 cases across AR, FR, EN, Arabizi, mixed. Expected fields per case.
- `evals/messages.jsonl`: booking / contact / neither, incl. tricky negatives ("I'll think and tell you Tuesday").
- `pnpm eval` runs them; CI fails a PR if field accuracy drops below the threshold in `evals/README.md` (start: 90% parse, 95% contact detection, 85% booking).
- Real failures from `search.parse_corrected` become new cases weekly (anonymised, reviewed by me before they count).

## 7. Guardrails
- AI never: sends messages, publishes/hides reviews, approves IDs, changes prices/plans, merges code, emails users outside the agreed templates.
- Budget: `settings.ai_monthly_budget_usd`; at 80% email me, at 100% switch to fallbacks.
- Rate limits: per user and per IP on search, Ask Talmeez and MCP.
- Prompt injection: tutor bios and messages are data. Tools that act on them can't call other tools that write.
- Everything the AI did is in `ai_runs`; everything an admin looked at is in `admin_access_log`.

## 8. Third-party trackers (my decision, 27 Oct 2026)
Used for ad measurement and marketing reports. The learning loop does not depend on them.

| Tracker | Why | How |
|---|---|---|
| Google Analytics 4 | Traffic sources, SEO, campaigns | gtag with Google Consent Mode v2, IP not stored, no user-id |
| Meta Pixel + Conversions API | Instagram/Facebook ads: which ads bring sign-ups | Browser pixel after consent; server-side CAPI for sign-up/lead events, deduplicated with event_id |
| TikTok Pixel + Events API | TikTok ads | Same pattern as Meta |

Left out: session-replay tools (Hotjar, Clarity) — they would record parents typing about their children and chat screens.

**Rules**
- **Consent banner** (AR/FR/EN), choices: Accept all / Only necessary / Settings. Nothing loads before "Accept". "Only necessary" must be as easy as "Accept". Choice stored and changeable from the footer.
- **Pages where trackers never load:** messages, admin, sign-up ID step, account settings, dashboard.
- **Allow-listed events only** (anything else is dropped by the `track()` helper):
  - `PageView` (URL without query string)
  - `Search` — no query text, no class, no area; only `{has_results: bool}`
  - `ViewContent` — public tutor profile id
  - `Lead` — parent sent first message (no ids of the parent)
  - `CompleteRegistration` — `{role: parent|tutor}`
  - `Subscribe` — tutor plan activated
- **No advanced matching:** never send hashed email/phone to Meta or TikTok.
- **IDs in `settings`** (`ga4_id`, `meta_pixel_id`, `tiktok_pixel_id`); empty = tracker off.
- **Privacy policy** must list each tracker, what it gets, and how to opt out (lawyer review).
