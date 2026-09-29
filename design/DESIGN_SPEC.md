# Ready Land Buyers Curative CRM: design HYPOTHESES (v1, unverified)

> **Status: not a spec.** Written from public HTML and screenshots before any logged-in or server recon. It has known gaps: it assumes fields (next action, due, call log) that may not exist, and it ignores the dossier/ingest pipeline and the `localhost:3000` iteration. Replace it after `design/RECON.md` exists. Keep only what recon confirms.

Author roles: product designer, interaction designer, information architect, distressed-real-estate operator, front-end engineer.

**Evidence base.** Public, unauthenticated view of https://5-78-191-172.sslip.io/ (served HTML/JS only). The logged-in screens were NOT seen. Anything marked *verify* must be checked in the live app before building.

## 1. What exists today (from the shipped front end)

- Single-page vanilla JS app, one ~1 MB HTML file (about 645 KB inline CSS with embedded fonts, about 98 KB JS). Python `BaseHTTP` server behind Caddy. Every response has `Cache-Control: no-store`, so the whole 1 MB is re-downloaded on each load.
- Left nav: **Pipeline / Mine / Killed Archived Leads / Archived**. "Mine" is really the title-exam queue (steps: need-exam, verifying, needs-judge, no-go). The name doesn't say that.
- Home is a deal **board / table / cards** with filters, sort and drag to change stage.
- Clicking a deal opens a full-height **drawer** (with "← Back to pipeline"). Tabs: Call dossier, per-engine reports, Parcel map / flood / wetlands, Action Notes.
- Deals carry verdict (ADVANCE / KILL), urgency, stage, attorney, county, value, spread.
- Endpoints seen: `/api/login`, `/api/me`, `/api/pdfmeta`, `/api/prompts`. *verify:* whether next-action, due date, blocker and call outcome exist as data fields.

## 2. Diagnosis

| # | Problem | Why it costs time |
|---|---|---|
| 1 | The home screen is the pipeline (a filing cabinet), not a to-do list | Stan has to decide what to work on every morning |
| 2 | The drawer hides the list | Every deal is open, read, back, find the next one |
| 3 | Deal tabs are organised by data source (engine reports, map) instead of by question | Understanding a deal means clicking through tabs |
| 4 | "Mine", "Killed Archived Leads" and "Archived" are unclear or overlapping | Navigation costs thought |
| 5 | Nothing on screen ties an action to its result | The loop breaks at "record the result, get the next action" |
| 6 | No-cache 1 MB page | Slow load, especially on phone |

## 3. Design principle

The CRM is a to-do list of deals that opens the right context beside it. Screen 1 shows what to do. The next screen shows why, and lets you do it and log it.

## 4. Information architecture

Replace the 4-item nav:

| New | Replaces | Purpose |
|---|---|---|
| **Today** (default) | new | Ranked work queue |
| **Pipeline** | Pipeline | Board / table for browsing. Secondary. Cards show next action and due only |
| **Title exam** | Mine | Same content, honest name |
| **Archive** | Killed + Archived | One list with a Killed / Archived filter |

Global search stays. Add match context: address, owner, phone, parcel, case number.

## 5. Today (home)

- Split view on desktop: queue on the left, selected deal on the right. No drawer for the main flow.
- Row: rank, name, stage tag, next action, why-now line, due (red if late, "None" in red if unset).
- Rank uses real data only: overdue, due today, promised callbacks, contract or deadline risk, then deals with no next action. Never invent urgency. Show what's missing.
- Filter chips: All, Calls, Overdue, No next action, Blocked. These are filters on one list, not separate pages.
- `J` / `K` moves through the queue, `Enter` opens.

## 6. Deal pane

Top to bottom, in this order:
1. **Header:** name, address, county, ask / value / spread, verdict.
2. **Five answer cells:** what is this (distress), what is wrong (blocker), waiting for, talk to (person, role, phone), prior attempts.
3. **Sticky Next Action bar:** the action, due, and one-tap result buttons (Reached, Voicemail, No answer, Wrong number, Callback, Price objection, Wants offer, Not interested).
4. **Tabs:** Brief (default, the current Call dossier), Title (exam and engine reports), Map, Notes and timeline.

Rules:
- Anything that doesn't affect a decision, action, blocker or next step goes in a tab.
- Engine reports move under Title.
- There is no state where a deal has a status but no next action.

## 7. Log a result

A result button opens an inline follow-up, in the same place, with:
- a pre-filled next action based on the result (voicemail becomes "call again", wants offer becomes "send offer" and so on),
- date presets (Tomorrow, 3 days, Next week),
- **Save and go to next**, which saves, re-ranks and opens the next deal.

Two taps plus one edit. No modal, no navigation. Not-interested offers "archive or re-check in 90 days".

## 8. Mobile

- Queue as full-width rows. Tap opens the deal full-screen with a "← Today" back.
- Bottom tab bar (Today, Pipeline, Title, Archive).
- The Next Action bar stays visible while scrolling. The phone number is a tap-to-call link.
- Touch targets are at least 44 px.

## 9. Visual and accessibility

- Keep the existing brand (Ready Land Buyers, Inter, light and dark themes).
- One accent colour for action. Red is only for late or missing. Amber is for due today.
- Visible focus rings, keyboard reachable, colour never the only signal (due text is always written), AA contrast in both themes.

## 10. Engineering (small, no rewrite)

- Move inline CSS and JS into static files. Serve with hashed names and long cache. Keep HTML `no-store`.
- Add `GET /api/today` only if the ranking can't be computed from the existing deal list on the client.
- Reuse existing fields. Add next-action, due, blocker and outcome columns only if *verify* shows they don't exist, and add them as additive migrations with a backup first.
- Feature flag or separate route (`/beta`) so the current UI keeps working during rollout.

## 11. Acceptance test

On the live app, on 3 different deals, in the browser: open Today, open the top deal, understand it (30 s), act, log the result, save and go to next. Count clicks (target: 3 from result to next deal). Phone and desktop. Test data only: no real calls, SMS or emails.

## 12. Out of scope for this pass

AI features, command palette, title graphs, new dashboards, new data entities.

See `mockup.html` for the interaction (invented placeholder data).
