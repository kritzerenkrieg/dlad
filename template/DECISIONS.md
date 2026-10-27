# Decisions

Working list for deciding features, approach and technical details. This file overrules `ARCHITECTURE.md` and `ROADMAP.md` wherever they disagree.

Each item has a code: the letter is the `ROADMAP.md` phase, the number is its sequence within that phase. Codes are stable and never reused. A new feature takes the next unused letter.

Status:
- **Decided**: settled and binding. State the choice and the reason in a few words.
- **Inspect**: needs a look first (spike, library, benchmark, test). Research only.
- **TBD**: read, but not discussed yet. Ask, don't guess.
- **Question**: raised (usually by the agent) and not yet read by the human. Triage it by moving it to TBD, Inspect or Decided. Treat it like TBD until then.

A phase starts only when its rows are Decided.

| Letter | Roadmap phase |
|---|---|
| A | Phase A: Decisions and spikes |
| B | Phase B: <name> |
| C | Phase C: <name> |
| ... | ... |

| Code | Topic | Status | Decision / Question |
|---|---|---|---|
| A1 | <topic> | TBD | |
| A2 | <topic> | TBD | |
| B1 | <topic> | TBD | |
| ... | ... | ... | ... |
