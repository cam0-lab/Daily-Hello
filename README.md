# Talmeez (formerly Daily Hello)

ESADE, Launching AI-Native Ventures. Camille-Alexandre Monsef.

Talmeez is a tutoring marketplace for Lebanon. A parent types one sentence ("maths tutor for my son in EB9 in Jounieh"), in Arabic, Arabizi, French or English, and gets tutors who teach that exact class, with a one-line reason for each match. Tutors have their identity checked.

## Why this repo has two projects
Sessions 1 to 6 were on Daily Hello, a daily AI phone call to an elderly parent. After Session 6 I switched my course project to Talmeez, which I had started before the course, because there I have real tutors, real data and can test with real users this term. From Session 7 on, all my work is on Talmeez.

- **Talmeez (current):** the files at the top of this repo.
- **Daily Hello (Sessions 1–6):** the `daily-hello/` folder, unchanged since 25 October.

## Files
- `architecturedoc.md`: how Talmeez works, the AI processes, data, guardrails and decision log.
- `scope.md`: what is built, what is out of scope, the number I track and where it stands.
- `docs/ai-native.md`: the full AI design (the brain, search, messaging, learning loop, evals, guardrails, trackers).
- `docs/product-rules.md`: product rules (accounts, verification, class matching, leads, reviews, paywall).
- `evals/`: the test cases the AI parts must pass (170 search sentences, 108 messages) and the pass thresholds.
- `daily-hello/`: the earlier project.

The app code is in a separate private repository because it holds the live site's configuration. I can give access on request.

## Links
- Live site (old version): https://talmeez.me
- New version: being rebuilt, not live yet.
