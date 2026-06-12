# jhms

Public **CI shell** for the `noveljack/market` screener. This repository contains **only GitHub Actions workflow definitions** — no application code, no data.

## How it works

- Workflows here check out the private `noveljack/market` repo (via a deploy token) and run its entrypoints.
- Workflows are **triggered exclusively by [cron-jobs.org](https://cron-jobs.org)**, not by GitHub's built-in `schedule:` cron. Each workflow exposes `workflow_dispatch` + `repository_dispatch` so cron-jobs.org can fire it over the GitHub API.
- Heavy backfill jobs fan out using a build matrix of **16–18 parallel shards** (kept below the 20-job free-tier ceiling).

## Lifecycle

- During development this repo lives on the **primary** account alongside `market`.
- Before production it will be **moved to a secondary account**, so that if anything happens to the public automation account, the primary account holding `market` stays safe.

## Secrets required

`MARKET_REPO_PAT`, `RCLONE_CONFIG`, `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`, `HALAL_REPO_PAT`, `GROQ_API_KEY`, `OPENROUTER_API_KEY`, `REDDIT_CLIENT_ID`, `REDDIT_CLIENT_SECRET`, `DASHBOARD_URL`, and a `CRON_DISPATCH_TOKEN` used by cron-jobs.org.
