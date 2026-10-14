---
name: timetracker-hours
description: Fill your monthly hours in the Mistura Time Tracker (timetracker.misturatechnologies.com) via its API using your logged-in browser session, grounding each day's description in your real GitHub activity. Use when you say "fill/post my hours", "log my timetracker", or similar.
---

# Mistura Time Tracker hours (via API)

Fill the current month's working-day hours in the Mistura Time Tracker by calling its
API directly from your logged-in Chrome tab. No password handling: the call reuses the
session token already in the page. Each day's description is grounded in your real
GitHub activity for that date (verify against reality, never invent work).

## Preconditions

- You must be logged in to https://timetracker.misturatechnologies.com in a Chrome tab
  the browser automation can see. If the MCP tab shows `/login`, log in there first (a
  tab the agent opened does not inherit your other tabs' session).
- Confirm the tab via `tabs_context_mcp`; use that `tabId` for every browser call.

## The API

- Base (dev): `https://ca-timetracker-bff-dev.yellowstone-9a3e246e.centralus.azurecontainerapps.io`
  - This is the dev BFF host and can rotate. If a call 404s, reload `/worker-hours`,
    then `read_network_requests` filtered on `/api/proxy/` to re-discover the base.
- Auth: Bearer token in `localStorage.auth_token` (no cookies). Call from the PAGE context
  with `javascript_tool` so the token never leaves the browser:
  `headers: { Authorization: 'Bearer ' + localStorage.getItem('auth_token'), 'Content-Type': 'application/json' }`
- `GET  /api/proxy/timeentries` -> array of `{id, userId, date, taskName, hours, details, workItemNumber, clientId, clientName, legacyClientText}`.
  `date` is `YYYY-MM-DDT00:00:00`.
- `PUT /api/proxy/timeentries/{id}` with the full entry body updates in place (200); no
  need for delete+repost when enriching existing entries.
- `POST /api/proxy/timeentries` body `{date:'YYYY-MM-DDT00:00:00', taskName, hours, details, clientId}`
  -> `201` with the created entry. Server derives `userId` and `clientName`. `clientId` is
  REQUIRED (a missing one returns `400 "ClientId is required."`); reuse the `clientId` from an
  existing entry of yours (read it from the GET above). `workItemNumber` optional.
- Entries are editable/deletable in the UI, so a wrong entry is recoverable.

## Process

1. `GET` existing entries; build a Set of already-filled dates for the target month.
   Typical pattern: 8h/day, weekdays only, one entry per day.
2. Compute the month's working days: Mon-Fri, minus weekends, minus the holidays that
   apply to you, minus days already filled. Confirm the day-of-week math against a real
   anchor from the existing entries (e.g. a known Monday).
3. Ground each missing day in real activity: pull your PRs created/merged that date, e.g.
   `gh pr list --repo <org>/<repo> --author <your-login> --state all --search "merged:<from>..<to>"`
   (and `created:`), across the repos you work in. Write a `taskName` (~60 chars) + a
   `details` paragraph naming the real PRs / ticket keys.
4. Show the day-by-day plan (date, taskName, 8h) before or alongside posting.
5. POST each missing day with a DEDUP GUARD: re-`GET` and skip any date already present,
   so a partial failure can be safely re-run. Browser tab calls can intermittently fail
   with "Couldn't determine which page this action targets" mid-batch; re-read
   `tabs_context_mcp` and re-run, the guard prevents duplicates.
6. Verify: `GET` again, assert distinct-date count == working days, no duplicates, total
   hours == days x 8.

## Content + safety rules

- INVOICE-GRADE DETAIL: these entries can feed the client invoice. Each day's `details`
  should fully explain the work: what was built or fixed, why it mattered, the concrete
  deliverables (PR numbers, ticket keys), and how it was verified (tests, live validation).
  A substantial paragraph per day, not a summary line.
- PREVIEW FIRST: show the complete day-by-day text and get approval BEFORE posting.
- Keep entries client-facing: describe the work as development and testing, not internal
  tooling/process.
- Never fabricate work. If a day has thin activity, describe it honestly (review
  follow-ups, validation) rather than inflating.
- No em dashes anywhere.
- This is an official employer submission; never invent hours for days not worked.
