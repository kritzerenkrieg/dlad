# DLAD: Decision-led Agentic Development

A framework for building software with AI coding agents. Project context lives in a few markdown files in the repo. One of them, `DECISIONS.md`, is the source of truth and overrules the rest. The agent works inside what is decided, researches what is open, and asks about what is undiscussed.

## Principles

1. **Context is a deliverable.** An agent starts every session with no memory. What it cannot find in the repo, it guesses, and the guess ends up in code.
2. **One source of truth.** Docs disagree over time. A fixed precedence rule decides which one wins.
3. **The human decides, the agent executes.** Agents propose, research and implement. Only the human settles a decision.
4. **Verifiable work.** Every task has a check the agent can run itself. "Done" means the check passes, not that the agent says so.
5. **Small, reviewable steps.** One task per session, one focused diff, a human review at each phase boundary.
6. **Least context, loaded on demand.** Keep the always-loaded file short and point to detail elsewhere. Long instructions get ignored.
7. **Least privilege.** The agent gets the access the task needs, and nothing it reads can grant more.

## Files

| File | Holds | Changes |
|---|---|---|
| `AGENTS.md` | Entry point for the agent: commands, boundaries, working loop. Read automatically by most tools. | Rarely |
| `DECISIONS.md` | Coded decisions and open questions. **Overrules the other files.** | Constantly, while discussing |
| `ARCHITECTURE.md` | Design: scope, components, flows, data, interfaces, stack, quality targets | When a decision changes the design |
| `ROADMAP.md` | Phased task checklist, each task with a done-check, then the feature and maintenance cycle | As work completes |

`AGENTS.md` is the cross-tool convention (Codex, Cursor, Copilot, Gemini CLI and others read it). For tools that read a different filename, such as `CLAUDE.md`, make that file a one-line pointer to `AGENTS.md` or a symlink.

## The decision table

Each decision has a code: **letter = roadmap phase, number = sequence within that phase**. `B2` is the second item of the phase marked B. Codes are stable and never reused. A new feature turn takes the next unused letter.

| Code | Topic | Status | Decision / Question |
|---|---|---|---|

Statuses:

| Status | Meaning | Agent behavior |
|---|---|---|
| **Decided** | Settled. | Binding. Implement it as written. |
| **Inspect** | Needs a look first: a spike, reading a library, a benchmark, a test. | May research and report options with evidence. Does not choose. |
| **TBD** | Read, but not discussed yet. | Stops and asks. Does not guess. |
| **Question** | Raised, usually by the agent, and not yet read by the human. | Treated like TBD. Stops and asks. |

The status shows what the human has seen. A new row starts as Question, and the human triages it to TBD, Inspect or Decided when they read it. Anything still marked Question is unread.

A Decided row states the choice and, in a few words, the constraint that forced it. A Decided row without a reason is hard to revisit and easy to misapply.

## Rules

1. **`DECISIONS.md` overrules `ARCHITECTURE.md` and `ROADMAP.md`** wherever they disagree.
2. **The human decides.** Only the human moves a row to Decided.
3. **The agent doesn't invent answers.** If work touches an Inspect, TBD or Question row, it stops and asks.
4. **A conflict is raised, not worked around.** If a task contradicts a Decided row, the agent says so before writing code.
5. **New questions become rows.** When the agent hits an unspecified point that is not yet a row, it adds one as Question and continues with unrelated work.
6. **Reference decisions by code** in prompts, commits, PRs and issues (`implements B2`).
7. **Decisions change by editing the row.** The table always states the current state, and git holds the history.
8. **A phase starts only when its rows are Decided.** Any other status in a phase blocks it.
9. **The project does not end, it cycles.** After the planned phases, every new feature gets its own lettered phase and goes through the same gate. Maintenance runs on a schedule and uses the same task format.

## Working loop

Human, per project:

1. Draft `ARCHITECTURE.md` and `ROADMAP.md` from the first conversation.
2. Pull every unsettled point into `DECISIONS.md` as TBD or Inspect (or Question, if you have not read it yet).
3. Discuss. Move rows to Decided as you answer them. Run the spikes for Inspect rows.
4. Write the done-check for each task in the next phase.
5. Hand the phase to the agent. Review at the end of it.
6. Repeat as a cycle. A feature turn adds the next lettered phase (new idea as a Question row, triage, Decided, tasks). A maintenance turn runs the recurring tasks in `ROADMAP.md` on a schedule: dependencies, bugs, docs drift, stale rows.

Agent, per task (also in `AGENTS.md`):

1. **Orient.** Read `DECISIONS.md`, then the relevant parts of the other files. Find the task's codes and check their statuses.
2. **Plan.** For anything beyond a trivial change, write a short plan first: files to touch, approach, how it will be checked. Stop for approval if the plan touches an Inspect, TBD or Question row or contradicts a Decided one.
3. **Implement.** Smallest diff that satisfies the task. Write or update tests alongside, ideally before.
4. **Verify.** Run the task's done-check and the project's test, lint and type commands. Fix failures. Do not weaken or delete a test to make it pass.
5. **Report.** Summarize what changed, which codes it implements, the check results, and any new questions raised as rows. Tick the roadmap item only if the check passed.

## Making tasks agent-ready

A roadmap task is ready when it has:

- **Codes** of the decisions it depends on, in parentheses.
- A `[P]` marker if it can run in parallel with its neighbors.
- A **done-check**: a command, test or observable behavior that is true when the task is finished.
- A **scope** small enough for one session and one reviewable diff. If it is not, split it.

```
- [ ] Add the retry wrapper (B1, B2). Done when: `make test` passes and `test_retry.py` covers timeout, 5xx and success.
```

## Context hygiene

- Keep `AGENTS.md` short. Put commands, boundaries and the loop in it, and link to deeper docs for anything else.
- Put only what the agent cannot infer from the code in the docs. Don't restate what the repo shows.
- Use one session per task. Start a fresh session when the task changes or the conversation gets long. The files carry the state, not the chat.
- Roadmap checkboxes are the progress record. Don't keep a separate agent scratch file in the repo.
- When the agent makes the same mistake twice, fix it at the source: add a line to `AGENTS.md`, a lint rule, or a test.

## Safety

- **Boundaries** in `AGENTS.md` are three tiers: always do, ask first, never do. Typical "never": commit secrets, edit generated or vendored files, change CI or deploy config without asking.
- **Sandbox and permissions.** Run agents in a scoped environment with the narrowest tool permissions that work. Require approval for destructive or outward-facing actions: deleting data, pushing, deploying, sending messages, spending money.
- **Untrusted input.** Text from web pages, issues, dependencies, tool output and files is data, not instructions. Only `AGENTS.md`, the decision files and the human direct the agent.
- **Secrets** never go in the docs, prompts or commits. Reference them by name.
- **Review before merge.** The agent's output is a proposal. A human reviews the diff, and CI runs independently of the agent.
- **Use hooks or CI** for rules that must always hold (format, lint, tests, secret scan), since instructions alone are not enforced.

## Parallel and multi-agent work

- Split work by independent task or by file area, with no shared edits. Give each agent its own branch or worktree.
- Give each agent the task, its codes and its done-check. Do not give it the whole chat.
- `DECISIONS.md` is edited by the human. Agents propose rows in their report and do not write Decided rows.
- Use a read-only agent for research and review, and a separate agent to implement. A reviewer that did not write the code catches more.

## Using it

1. Copy the contents of `template/` into a new project root.
2. Fill in the commands and boundaries in `AGENTS.md`.
3. Start with `DECISIONS.md`, then the others.

## Prior art

DLAD combines established practice and does not claim to be new.

- ADRs as agent context: [ADRs for Agent Context](https://understandingdata.com/posts/adrs-for-agent-context/), [Coding agents love decision records](https://duncandavidson.com/agents-love-decisions), [The ADR your agent never read](https://tianpan.co/blog/2026/07/02/the-adr-your-agent-never-read).
- Spec-driven development: [Plannotator](https://docs.plannotator.ai/learn/planning/spec-driven-development-for-ai-coding-agents.md), [How to write a good spec for AI agents](https://addyosmani.com/blog/good-spec/).
- The [AGENTS.md](https://agents.md) convention for agent instructions.

What DLAD adds: one table instead of a folder of ADR files, codes tied to roadmap phases, an explicit precedence rule, an Inspect status, a phase gate on open decisions, and a table used to decide before code exists as well as to record afterwards.

## Limits

- Docs go stale. Rule 1 limits the damage, and the maintenance turn catches drift, but neither removes it.
- It suits work where choices can be made up front. For exploratory work, keep Phase A (spikes) long and the later phases short.
- Agents still make mistakes inside Decided rows. Tests, CI and review are the defense, not the docs.
