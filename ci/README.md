# CI / jhms

`jhms` is the **public CI shell**. It holds GitHub Actions workflows only — no
strategy code. All code lives in the private `noveljack/market` repo, which each
workflow checks out using the `MARKET_REPO_PAT` secret.

Workflows are triggered **externally by cron-jobs.org** via `repository_dispatch`
(and manually via `workflow_dispatch`). There are intentionally **no** GitHub
`schedule:` cron triggers.

## IMPORTANT: one-time manual move

The automated agent cannot write into `.github/workflows/` (the GitHub
integration token lacks the `workflows` permission). The ready-to-use YAMLs are
staged in [`ci/workflows/`](./workflows). Move them into place once:

```bash
git mv ci/workflows/daily_eod.yml          .github/workflows/daily_eod.yml
git mv ci/workflows/backfill.yml           .github/workflows/backfill.yml
git mv ci/workflows/monthly_halal_sync.yml .github/workflows/monthly_halal_sync.yml
git commit -m "ci: activate workflows"
git push
```

(Or use the GitHub web UI: open each file, copy its contents into a new file at
`.github/workflows/<name>.yml`.)

## repository_dispatch event types (for cron-jobs.org)

POST to `https://api.github.com/repos/noveljack/jhms/dispatches` with header
`Authorization: Bearer <CRON_DISPATCH_TOKEN>` and body:

| Workflow             | event_type   | cadence (set in cron-jobs.org) |
| -------------------- | ------------ | ------------------------------ |
| daily_eod            | `daily-eod`  | weekdays ~6:00 PM IST          |
| backfill             | `backfill`   | on demand                      |
| monthly_halal_sync   | `halal-sync` | monthly (~11th, 06:00 IST)     |

## Required secrets

`MARKET_REPO_PAT`, `RCLONE_CONFIG`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`,
`HALAL_REPO_PAT`, `GROQ_API_KEY`, `OPENROUTER_API_KEY`, `REDDIT_CLIENT_ID`,
`REDDIT_CLIENT_SECRET`, `DASHBOARD_URL`, `CRON_DISPATCH_TOKEN`.

## Required variables

`RCLONE_REMOTE` (default `gdrive:market_db`), `HALAL_REPO`, `HALAL_FILE`
(default `halal_list.csv`).

## Production note

During the build, `jhms` lives on the primary account. For production it moves to
the secondary account so the primary `market` repo stays isolated.
