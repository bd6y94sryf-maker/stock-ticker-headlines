# stock-ticker-headlines

This repo holds `headlines.json`, fetched over HTTPS by a local macOS menu-bar
app. The app polls the file hourly and renders each headline as a clickable
ticker item that opens its source article.

## Update cadence

A scheduled task ("stock-ticker-headlines") runs hourly, every day, and
regenerates `headlines.json` from scratch each time. It is not just a
pre-market digest anymore — each run should reflect the most current
market-moving news at that hour, favoring new/updated stories over repeating
the same ones run after run (unless a story is still the dominant driver).

## Sourcing

Pull from Wall Street Journal (wsj.com), Yahoo Finance (finance.yahoo.com),
Morning Brew (morningbrew.com), and other major financial news sources.
Prioritize, in roughly this order:
- Fed/central bank commentary or rate decisions
- Major economic data releases (CPI, jobs reports, PMI, GDP, etc.)
- Notable intraday/overnight market moves
- Significant corporate earnings or M&A news
- Major geopolitical events affecting markets

## Output contract

Exactly 5-7 headlines. Each is ONE concise, punchy sentence/title — no
paragraphs, no body text. Each headline needs the direct URL of the specific
source article it was drawn from: a real, specific article URL, never a
homepage/section front page, and never a fabricated URL.

`headlines.json` must always be valid, well-formed JSON in EXACTLY this
shape — no markdown code fences, no extra fields, no trailing commentary in
the file itself:

```json
{
  "date": "Month Day, Year",
  "headlines": [
    {"text": "Headline 1", "url": "https://example.com/real-article-1"}
  ]
}
```

Commit and push straight to the default branch (`main`) with a message like
`Headlines for <date> <HH:MM> UTC`. Since the app only ever reads the latest
commit, there's no need for PRs or review on these routine updates.

## Git safety checklist (always follow, every run)

This repo's working copy sometimes starts in a **detached HEAD** state
rather than on `main` (a past run found 4 hours of commits stranded this
way — committed locally, never actually published, while `git push origin
main` silently "succeeded" by pushing the unchanged, stale local `main`
ref). To prevent silent data loss, every run MUST do this exact sequence:

1. Before committing, check whether HEAD is attached to `main`:
   `git symbolic-ref -q HEAD` — if it does NOT print `refs/heads/main`,
   run `git checkout -B main` first so the branch pointer actually moves
   with your commit (don't just commit on a detached HEAD).
2. If stepping onto `main` reveals it's behind `origin/main`, fast-forward
   first (`git fetch origin main && git merge --ff-only origin/main`)
   before writing the new `headlines.json`.
3. Commit, then push with `git push -u origin main` (not a bare
   `git push`, in case there's no upstream yet).
4. **Verify the push actually landed** — don't trust exit code 0 alone:
   `git fetch origin main && git rev-parse HEAD` and
   `git rev-parse origin/main` must print the same SHA. If they don't
   match, something is wrong (diverged history, rejected push, stale
   local `main`) — investigate and retry rather than ending the run.
5. If step 4 ever reveals pre-existing unpushed commits from earlier runs
   (local `main`/HEAD ahead of `origin/main` by more than your own new
   commit), push those through too rather than leaving them stranded —
   do not rewrite or drop them.
