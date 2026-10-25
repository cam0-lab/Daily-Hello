# Daily Hello — Scope

Last updated: 25 October 2026

## The product in one line
A daily AI phone call to an elderly parent, with a green / amber / red summary for the adult child who pays. No hardware.

## Riskiest guess
Families will sign up for a daily AI call to their parent and keep using it after the first weeks.

## In scope now (built in index.html)
- Add a parent: name, phone, language, call time.
- Simulated check-in call in the browser (AI conversation or fixed script), in English, Spanish or French.
- Summary for the family with a flag per topic: sleep, food, medication, mood.
- Per-parent baseline (Session 5 defence): every call saved under that parent, flag compared to their own normal, questions personalised from their history.
- Last 7 calls trend from saved calls.
- Two example parents with 14 days of example history, labelled as examples.

## Out of scope for now
- Real phone calls (telephony and realtime voice).
- Server, real database (SQLite planned), login.
- Sending summaries by WhatsApp or SMS.
- Hardware of any kind (product choice: phone call only).

## Definition of done: per-parent baseline (25 Oct)
- [x] Each call is saved as one record under its parent and survives a page reload.
- [x] The baseline is only used from 5 past calls; before that the app says it is still learning.
- [x] The same answer gives a different flag for two parents with different histories (Margaret green, Georges amber for poor sleep).
- [x] A red flag is never lowered by the baseline.
- [x] The family view shows why: "normal for them" or "unusual", and what general rules alone would have said.
- [x] The call asks about last call's issue and the parent's usual plan for that weekday.

## Next scope (before Demo Day)
1. Talk to real parents: an AI call every day? fits their culture? why a call and not WhatsApp?
2. One real family, five days, one call a day, summary each evening; ask the adult child on day 5 if they would pay.
3. Security check before that test: parent consent, no exposed keys, data separated per family.
