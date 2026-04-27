# retrogames1337 status

Public uptime / status page for the **retrogames1337** services
(currently TypeSnake, with more games to come).

- **Live page:** https://status.retrogames1337.com
- **Project being monitored:** https://github.com/valikor/typesnake
- **Tech:** [upptime](https://upptime.js.org) — a fully open-source,
  GitHub-Actions-powered status page. No third-party SaaS.

## How it works

A scheduled GitHub Action runs every **5 minutes** and curls each
endpoint listed in [`.upptimerc.yml`](./.upptimerc.yml). Results are
committed to this repo's `master` branch as JSON in `history/`, and a
Jekyll-based static site is rebuilt nightly to `gh-pages` and served via
GitHub Pages at the custom domain above.

If an endpoint goes down (non-200 or times out beyond `maxResponseTime`),
upptime auto-opens a GitHub issue here labelled `down` and — if the
`DISCORD_WEBHOOK_URL` repo secret is set — fires a Discord notification.
The issue is auto-closed when the endpoint comes back.

## Endpoints monitored

| Endpoint | What it checks |
| --- | --- |
| `https://snake.retrogames1337.com/` | Game loads (HTTP 200, < 4 s) |
| `https://snake.retrogames1337.com/healthz` | App-level health (HTTP 200, < 1.5 s) |
| `https://snake.retrogames1337.com/metrics` | Prometheus metrics scrape (HTTP 200, < 2 s) |

## One-time setup (operator)

After cloning this repo onto GitHub, the operator needs to do a few
manual steps the GitHub CLI can't do. Full checklist in
[`docs/SETUP.md`](./docs/SETUP.md), summary:

1. Enable GitHub Pages on the `gh-pages` branch (created by the first
   workflow run).
2. Set custom domain `status.retrogames1337.com` and add the matching
   `CNAME` record in Route 53.
3. (Optional) Add `DISCORD_WEBHOOK_URL` repo secret for incident pings.
4. Manually trigger the **Uptime CI** workflow once to seed the first
   data point.

## License

MIT — same as upptime.
