# Daily Hello — Architecture Document

Last updated: 25 October 2026 (after Session 6: per-parent baseline built into the app)

## What the product does
An AI agent calls an elderly parent once a day for a short conversation, then sends the adult child (the buyer) a summary with a green, amber or red flag. No hardware: the parent only needs the phone they already have.

## What is built today (index.html, one page)
1. **Setup form + saved parents**: the adult child adds a parent (name, phone, language, call time). Each parent has their own call history.
2. **Check-in call (simulated)**: the call runs in the browser. The agent's voice is the browser's speech synthesis. The person testing types or taps the parent's answers. No real phone call is placed.
3. **Two conversation modes**:
   - *AI conversation*: an LLM runs the call (through the Claude artifact runtime). It gets the parent's history before the call and asks follow-ups.
   - *Fixed script*: five questions, personalised from the history (asks about last call's issue and the parent's usual plan for that weekday).
4. **Summary + flag**: AI summary (or keyword rules as fallback) gives a level per topic: sleep, food, medication, mood, plus the people and activities mentioned.
5. **Baseline comparison (the defence)**: the flag is compared to that parent's own normal before the family sees it. See "Baseline logic".
6. **Family view**: today's flag, "Compared to [name]'s normal", summary, checks, "[name]'s normal" panel, and the last 7 saved calls.
7. **Example history**: two example parents (Margaret, Georges) with 14 days of calls each, marked as example data, so the baseline can be shown without real users.

## Not built yet
- Real phone calls (telephony + realtime voice agent).
- A server and a real database.
- Sending the summary to the family (WhatsApp/SMS).
- Consent flow for the parent, login for the family.

## Data
**Now (demo):** browser localStorage, key `dailyhello.calls.v1`. Data stays in the browser it was created in.
- One entry per parent, keyed by name + phone number: `{ profile, calls[] }`
- One structured record per call:
  - date
  - checks: sleep, food, medication, mood (0 green / 1 amber / 2 red each)
  - notes per topic (short text)
  - mentions (people and activities named, e.g. "lunch with Joan")
  - flag (after baseline) and genericFlag (rules only)
  - example (true for sample history)
- One call per parent per day (a second call the same day replaces the first).

**Planned (first real users):** same record in a SQLite `calls` table on the server, keyed by parent and family. Not built yet.

## Baseline logic
- Built from the parent's last 14 calls, never counting today's call.
- Used only once there are at least 5 calls. Before that the app says "Still learning [name]'s normal" and uses general rules.
- Per topic: share of calls where it was amber or worse.
  - Amber topic that is usual for this parent (amber in at least half of calls) is marked "Normal for them" and does not raise the flag on its own.
  - Amber topic that is rare for this parent (25% of calls or fewer) is marked "Unusual".
- Habits: a person or activity mentioned 2+ times on the same weekday becomes a weekly habit (e.g. lunch with Joan on Sundays). If today is that day and it is not mentioned, the call is flagged amber.
- Red (fall, chest pain, confusion…) is never lowered by the baseline.
- The family view always shows what general rules alone would have said when the baseline changed the flag.
- Before each call, the agent gets: last call summary and issues, the parent's normal per topic, today's habit, people often mentioned.

## Defense: data cycle (chosen in Session 5)
Each call adds to a personal baseline for that parent. The agent compares each new call to the parent's own normal, not a generic rule, so follow-ups get more specific and false alarms drop. A competitor starts from zero for every family. Weakness (from the Session 5 AI review): the baseline is weak in the first weeks, which is exactly when families decide whether to pay.

## Rules
- A parent's baseline is **never shared outside that family**. In the demo, history lives only in the browser where it was created.
- No hardware. Phone call only.
- No real users or real health data in the app yet.
- Before any real parent gets a call: security check (consent from the parent, no exposed keys, data separated per family).

## Decision log
| Date | Decision | Why |
|---|---|---|
| Session 5 | Data cycle as the one defense | Only advantage I can own; the software itself can be copied in an afternoon |
| Session 5 | One structured record per call, per parent | Smallest first step of the baseline |
| Session 5 | Baseline never shared outside the family | Trust, and health-type data |
| Session 6 | No hardware (product choice made earlier, not a sprint cut) | Keeps the product to a phone call |
| Session 6 | Used the sprint to demo the existing MVP, no new feature | Wanted feedback on the full product and the pitch |
| Session 6 | No real users touched the app, so no security check | Demo on sample data only |
| 25 Oct | Built the baseline into the app (save, compare, personalise questions) | First working piece of the Session 5 defence |
| 25 Oct | Browser localStorage instead of SQLite for now | No server yet; keeps data in one family's browser; SQLite when real calls start |
| 25 Oct | Baseline only after 5 calls, window of 14 calls | Too few calls gives a fake "normal"; 14 keeps it recent |
| 25 Oct | Red never lowered by the baseline | A fall is never "normal for them" |
| 25 Oct | Show the rules-only flag next to the baseline flag | Family can see why the flag differs; easier to trust |

## Feedback
- Session 6 demo: make the pitch emotional (the child, the community) and target the right cultures.
- Midterm feedback (Anupam, 21–22 Oct): the room's questions are the riskiest guess. Do parents want to talk to an AI every day? Does it fit their culture? Why a phone call and not a WhatsApp message? Answer these with real parents before building more agents.

## Next
1. One real family for five days: one adult child and their parent, one call a day, summary every evening. On day five, ask the adult child if they would pay to keep it.
2. Ask real parents the three questions above and write down the answers.
3. Before that test: security check (parent consent, keys not exposed, data separated per family).
4. Then: one real phone call through a voice service, using the same call record.
