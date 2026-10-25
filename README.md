# Daily Hello

An AI agent calls an elderly parent once a day for a short chat, then tells the adult child how they are with a green, amber or red flag. No hardware, just the phone the parent already has.

ESADE, Launching AI-Native Ventures. Cam (Camille-Alexandre Monsef).

## Files
- `index.html` — the app (one page). Open it in a browser.
- `architecturedoc.md` — what is built, how the data works, decision log.

## What the app does today
- Add a parent, run a check-in call (simulated in the browser, you answer as the parent).
- Every call is saved to that parent's history.
- The flag is compared to that parent's own normal (last 14 calls, from 5 calls on). Example: "slept badly" is normal for Margaret (green), unusual for Georges (amber).
- Follow-up questions use the history ("Today is Sunday. Usually that means lunch with Joan. Is that still the plan?").
- Two example parents with 14 days of example history are loaded on first open.

## Not built yet
Real phone calls, a server, sending summaries to the family. No real users or real health data.

## Try it
1. Open `index.html`.
2. Pick Georges, press the green button, answer "Not great, I was up a lot" to the sleep question. Flag: amber, "unusual for Georges".
3. Pick Margaret, give the same answer. Flag: green, "usual for Margaret".

The AI conversation mode needs the Claude artifact runtime. Opened as a plain file, the app uses the fixed script.
