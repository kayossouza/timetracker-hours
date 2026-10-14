# timetracker-hours

A Kiro / Claude **skill** that fills your monthly hours in the Mistura Time Tracker for you, by
calling the time tracker's own API from your already-logged-in browser tab, and grounding each
day's description in your **real GitHub activity** for that date.

No passwords. No hardcoded tokens. It reuses the session token already sitting in your logged-in
tab, so nothing secret ever leaves your browser.

## What it does

- Reads the month's existing entries and figures out which working days are still missing.
- For each missing weekday, pulls the PRs you created/merged that day from GitHub and writes an
  honest, invoice-grade description (what you built, why, the PRs/ticket keys, how it was verified).
- Shows you the full day-by-day plan to approve, then posts the missing days (8h/day by default).
- Dedup-guarded and idempotent: safe to re-run, it skips days already filled.

## Why it's safe

- Auth is a Bearer token read at runtime from `localStorage.auth_token` in **your** logged-in page
  context, never stored or committed.
- It never invents hours or work. Thin day? It says so honestly instead of padding.
- Preview-before-post: you see every day's text before anything is written.

## Install

Drop the skill where your agent looks for skills:

- Kiro CLI: `~/.kiro/skills/timetracker-hours/SKILL.md`
- Claude: `~/.claude/skills/timetracker-hours/SKILL.md`

```bash
git clone https://github.com/kayossouza/timetracker-hours.git
mkdir -p ~/.kiro/skills/timetracker-hours
cp timetracker-hours/SKILL.md ~/.kiro/skills/timetracker-hours/SKILL.md
```

## Use

1. Log in to https://timetracker.misturatechnologies.com in a Chrome tab the browser automation
   can see.
2. Tell your agent: **"fill my timetracker hours"** (or "log my hours", "post my hours").
3. Review the day-by-day plan it shows you, then let it post.

## Requirements

- An agent with browser automation (Claude-in-Chrome / the MCP browser tools) and the `gh` CLI
  authenticated for your GitHub.
- You logged in to the time tracker in a visible tab.

## Notes

- The dev BFF host can rotate; if a call 404s, reload `/worker-hours` and re-discover the base from
  the network requests (the skill explains how).
- `clientId` is required on POST; the skill reads yours from your existing entries rather than
  assuming one.

## License

MIT
