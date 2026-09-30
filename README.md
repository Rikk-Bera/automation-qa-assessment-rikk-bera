# Automation & QA Developer Assessment — Rikk Bera

Everything here was built and tested against a local n8n instance (`npx n8n`) and the public RealWorld demo app (https://demo.realworld.show).
Walkthrough video: **[ADD LOOM LINK]**

## Files
| File | What it is |
|---|---|
| `Task1_QA_Report_RikkBera.pdf` | Bug table (5 issues) + root-cause analysis |
| `Task2_Workflow_RikkBera.json` | GitHub "morning brief" workflow |
| `Bonus_UptimeMonitor_RikkBera.json` | Uptime monitor for the Task 1 app |
| `screenshots/` | Canvas and execution screenshots |

## Task 1 — QA report
Tested sign-up, login, create/edit/delete article, comments, settings and logout on the RealWorld demo. Found 5 issues (invalid email accepted, no proper 404 page, comment sign-in loses the article, Popular Tags not ranked by popularity, email/password changeable without re-authentication). The root-cause analysis covers the highest-severity one: sensitive credential changes without step-up authentication.

## Task 2 — n8n "Morning Brief"
**Flow:** Schedule (1 hour) → GitHub search → Code (top 5) → GitHub repo details → Code (build digest) → IF → Telegram.
- **APIs:** GitHub Search API (`/search/repositories`, Python repos by stars) and GitHub Repos API (`/repos/{owner}/{repo}`). Chosen because they are free, need no API key for light use, and return rich JSON. Telegram Bot API delivers the digest.
- **Transformation:** `Select Top 5` keeps 5 repos and reshapes them. `Build Digest` sorts the enriched repos by stars, merges them into one message, and counts repos above 1000 stars.
- **IF threshold:** if at least one repo has more than 1000 stars, send the digest with a "popular" footer; otherwise send it with a "low activity" footer.
- **Error path:** both HTTP nodes use *On Error → Continue (using error output)*. A failed call goes to `Telegram - Error Alert`, which sends one alert per run instead of failing silently.
- **Credentials:** the Telegram bot token lives in n8n's credential store; no secrets are in the JSON.

## Bonus — Uptime monitor
**Flow:** Schedule (5 min) → ping the app → record status and response time → IF status is 200. If not, wait 30 seconds, re-check, and only alert (Telegram) if it is still down, so one-off blips don't page anyone. The HTTP nodes also retry 3 times on network errors. A second schedule (9am daily) posts a summary: checks, failures, uptime %, average and slowest response time.

## Setup
1. Run `npx n8n`, open http://localhost:5678, and import each JSON (menu → Import from File).
2. Create a Telegram credential (bot token from @BotFather) and select it in the Telegram nodes. Replace the Chat ID with your own.
3. Click *Execute workflow*. To test the alert path, set the URL in `Config and Timer` to `https://httpbin.org/status/503`.

## Known limitations
- GitHub's unauthenticated API allows 60 requests per hour; a GitHub credential would raise this.
- The daily-summary stats are stored in workflow static data, which only persists for published (active) workflows.
