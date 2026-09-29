# CRM OPERATOR PROMPT (v3)

Run this in Claude Code on the machine that has VS Code connected to the Hetzner server, with the live CRM open in the browser.

## Known facts (from the public front end; verify on the live system)

- Live app: https://5-78-191-172.sslip.io/ ("Ready Land Buyers, Curative CRM"). Single-page vanilla JS, Python `BaseHTTP` server behind Caddy.
- It is a curative-title workroom, not a stock Pipedrive clone: Pipeline / Mine (title exam) / Killed / Archived, deals with ADVANCE/KILL verdicts, a drawer with Call dossier, engine reports, parcel map and Action Notes.
- The design to implement is in `design/DESIGN_SPEC.md`, with an interaction mockup in `design/mockup.html`. Follow it. Deviate only where the live app proves it wrong, and say why.
- Do not assume the data model matches section 2 constraint 4's "Deal → Person → Organization → Activity". Read the real schema first.

## 0. The job

Make this CRM the fastest possible environment for one operator (Stan) to advance real distressed-real-estate deals.

Understand the existing system. Change only what materially improves the daily loop below. Prove it by using the application yourself.

This is not an architecture, audit or redesign exercise. A better-designed CRM that is still annoying to use is a failure.

## 1. The four daily actions (the whole product)

| # | Question | What the system must do |
|---|---|---|
| A | What should I work on? | Home screen is a ranked work queue |
| B | Understand this deal | Deal page answers the 7 questions in section 4 in under 30 seconds |
| C | Do the next action | Call, email, research and so on happen from the deal, with context in view |
| D | Record the result | One quick capture that changes state and creates or confirms the next action |

Everything else is subordinate. If a feature does not serve A–D, do not build it.

## 2. Hard constraints

1. **Usable first.** Do not redesign anything unless the current implementation prevents Stan from completing real work.
2. **Smallest change that materially improves daily deal execution.** Prefer editing, hiding or removing over adding.
3. **No new entity, table, abstraction, workflow engine or service** unless the existing model demonstrably cannot support the workflow. A cleaner model is not sufficient justification for a migration.
   - If a migration is truly needed: additive only, data-preserving, backed up first, tested on a copy, and reversible where practical.
4. **Keep working Pipedrive-derived structures** (Deal → Person → Organization → Activity, stages, and so on) unless you can name the specific workflow friction they cause. Do not replace them on theoretical grounds.
5. **No AI features in the first pass.** Only add one if it removes a bottleneck you observed while doing the section-6 loop. Do the CRM well without AI first.
6. **If information does not affect a current decision, action, blocker or next step, it must not compete for primary attention.** Move it behind a tab or expander, or remove it.
7. **Levels.** Do not spend effort on a lower level while a higher one is not excellent.
   - L1 must work: today's queue, deal understanding, next action, action → outcome → next action.
   - L2 must be supported: property, people, distress, title, documents, economics, timeline. Use existing fields and structured fields where they exist. Do not invent a title graph or evidence model.
   - L3 automate later: stale detection, missing info, follow-up, summaries.
   - L4 optional: AI copilot, command palette, graphs, extra dashboards, analytics.

## 3. Phase 0: Recon (read-only, time-boxed)

Before changing anything, determine what is actually running on the Hetzner server, reached through the existing VS Code environment. Reconcile it with the repository and the live browser app. The running system is the one you are improving, not just the repo.

Establish, and write down in about one page (no essay):
- Stack, how it is deployed, and where it runs (process manager, containers, reverse proxy).
- Which database, where it lives, and how it is backed up.
- Whether next action, due date, blocker and call outcome already exist as data fields. This decides whether any migration is needed.
- Where the deployed code differs from the repo (uncommitted changes, different branch, hand-edited files).
- Logs and any current errors.
- What in the UI is real, mocked or broken. Click through the main screens: dashboard, pipeline, deal, person, tasks, calls and activities.

Safety rules for the server:
- Read-only until a backup exists. Take a database dump and note the git commit or a copy of the deployed code before the first change.
- Make changes in the repo on a branch. Deploy the smallest change, verify, and keep a rollback path.
- Never print, log or commit secrets or credentials. Never run destructive commands (drop, truncate, rm -rf, force-push) without asking me.
- Do not change server, firewall, DNS or SSL configuration unless it is required by a fix and I have approved it.

## 4. Deal page: the 7 questions

Answerable at a glance, in this order:
1. What is this? (property, address, county, asset)
2. Is it worth pursuing? (asking, offer or value, expected spread, priority)
3. What is wrong? (distress type, title or lien problems)
4. What are we waiting for? (blocker, waiting-on)
5. Who do I talk to? (key person, role, phone, prior attempts)
6. What do I do next? (next action)
7. When? (due date and owner)

A deal must never sit with only a status. It needs next action + due + owner, and a blocker or waiting-on where one applies. Everything else (documents, full timeline, economics detail, title detail) sits below or behind tabs.

## 5. Home screen = the work queue

The home screen is one ranked list:

`Priority | Deal | Why now | Blocker | Next action | Person | Desired outcome`

- Clicking a row opens the deal directly. Working the queue replaces navigating the CRM.
- Ranking uses real data: due or overdue actions, hot or money deals, callbacks, and stale deals with no next action. Never invent urgency. If data is missing, show what is missing.
- Deals with no next action must be visible in the queue, since that is where deals die.
- Extra queues (Blocked, Waiting, Stale and so on) are filters on this list, not separate pages.

## 6. The mandatory operator loop (the definition of done)

Do this against the running system (desktop and phone width), and repeat it at least 3 times on 3 different deals:

Open CRM → find today's work → open a real deal → understand it → perform an action → record the result → create or confirm the next action → return to today's work → repeat.

Rules for the loop:
- Do it in the real browser, not by reading code or hitting APIs.
- Do not send real SMS, email or calls, and do not damage real deal data. Use a clearly labelled TEST deal, or make reversible edits and revert them. Delete test records afterward.
- Record each friction point (extra clicks, hunting, unclear next step, a field asked for twice) with the screen and number of clicks.
- Fix the top friction points and re-run the loop. The work is not finished while the loop is not smooth.

Call disposition: after a call, one step records the outcome (reached, no answer, voicemail, wrong number, callback, interested, not interested, price objection, wants offer, other) and prompts the next action. Add it only if it is missing or clumsy.

## 7. Working method

1. Recon (section 3).
2. Run the section-6 loop once as-is. List friction, ranked by how much it slows the loop. This is your real backlog.
3. Fix P0 and P1 only. P0 blocks deal work; P1 is recurring friction. Batch small, verifiable changes. Deploy each and re-test.
4. Re-run the loop. Regression-check adjacent screens.
5. Stop when the loop is smooth on 3 deals. Do not go looking for more to build.

Do not declare success because the code compiles or the tests pass.

## 8. Success criteria

1. **Can Stan use the system for a full real work session without thinking about the CRM itself?** (The CRM should disappear; the thought should be "I need to call Sarah," not "where do I put this?")
2. Home screen shows what needs attention in 5 seconds.
3. A deal is understood in 30 seconds.
4. Call → outcome → next action happens in one flow.
5. Another competent person could pick up a deal without Stan's memory.

North star: more valuable deals advanced per unit of human attention. Do not confuse activity with progress. More screens, fields, dashboards, automations or AI are not progress.

## 9. Final report (short)

1. What was running on Hetzner, and any repo/server drift found.
2. Friction found in the loop (top 5–10, with click counts).
3. What you changed (files, migrations, deploy steps) and how to roll back.
4. What you deliberately did not change, and why.
5. Loop results: before vs after, on which deals.
6. Known gaps and the next 3 highest-value changes.
7. Today's queue from the live CRM, using real records.
